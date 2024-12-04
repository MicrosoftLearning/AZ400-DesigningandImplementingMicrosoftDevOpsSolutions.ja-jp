---
lab:
  title: Docker コンテナーを Azure App Service Web アプリにデプロイする
  module: 'Module 02: Implement CI with Azure Pipelines and GitHub Actions'
---

# Docker コンテナーを Azure App Service Web アプリにデプロイする

## 受講生用ラボ マニュアル

## ラボの要件

- このラボには、**Microsoft Edge** または [Azure DevOps 対応ブラウザー](https://learn.microsoft.com/azure/devops/server/compatibility)が必要です。

- **Azure DevOps 組織を設定する:** このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization)に関するページの手順に従って作成してください。

- 既存の Azure サブスクリプションを識別するか、新しいものを作成します。

- Azure サブスクリプションで共同作成者または所有者のロールを持つ Microsoft アカウントまたは Microsoft Entra アカウントを持っていることを確認します。 詳細については、[「Azure portal を使用して Azure ロールの割り当てを一覧表示する」](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal)および[「Azure Active Directory で管理者ロールを表示して割当てる」](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal)を参照してください。

## ラボの概要

このラボでは、Azure DevOps CI/CD パイプラインを使用してカスタム Docker イメージを構築し、それを Azure Container Registry にプッシュして、コンテナーとして Azure App Service にデプロイする方法を学習します。

## 目標

このラボを完了すると、次のことができるようになります。

- Microsoft がホストする Linux エージェントを使用して、カスタム Docker イメージを構築する
- Azure Container Registry にイメージをプッシュする
- Azure DevOps を使用して、Docker イメージをコンテナーとして Azure App Service にデプロイする

## 推定時間:30 分

## Instructions

### 演習 0:ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、[eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) に基づくリポジトリを含む新しい Azure DevOps プロジェクトで構成されます。

#### タスク 1: (完了している場合はスキップしてください) チーム プロジェクトを作成して構成する

このタスクでは、複数のラボで使用される **eShopOnWeb** Azure DevOps プロジェクトを作成します。

1. ラボ コンピューターのブラウザー ウィンドウで、Azure DevOps 組織を開きます。 **[新しいプロジェクト]** をクリックします。 プロジェクトに **eShopOnWeb** という名前を付け、 **[作業項目プロセス]** ドロップダウンで **[スクラム]** を選びます。 **[作成]** をクリックします。

#### タスク 2: (完了している場合はスキップしてください) eShopOnWeb Git リポジトリをインポートする

このタスクでは、複数のラボで使用される eShopOnWeb Git リポジトリをインポートします。

1. ラボ コンピューターのブラウザー ウィンドウで、Azure DevOps 組織と、前に作成した **eShopOnWeb** プロジェクトを開きます。 **[リポジトリ] > [ファイル]** 、 **[インポート]** をクリックします。 **[Git リポジトリをインポートする]** ウィンドウで、URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> を貼り付けて、 **[インポート]** をクリックします。

1. リポジトリは次のように編成されています。
    - **.ado** フォルダーには、Azure DevOps の YAML パイプラインが含まれています。
    - **.devcontainer** フォルダーには、コンテナーを使って開発するためのセットアップが含まれています (VS Code でローカルに、または GitHub Codespaces で)。
    - **infra** フォルダーには、一部のラボ シナリオで使用される Bicep および ARM のコードとしてのインフラストラクチャ テンプレートが含まれています。
    - **.github** フォルダーには、YAML GitHub ワークフローの定義が含まれています。
    - **src** フォルダーには、ラボ シナリオで使用される .NET 8 Web サイトが含まれています。

#### タスク 3: (完了している場合はスキップしてください) メイン ブランチを既定のブランチとして設定する

1. **[リポジトリ] > [ブランチ]** に移動します。
1. **main** ブランチをポイントし、列の右側に表示される省略記号をクリックします。
1. **[既定のブランチとして設定]** をクリックします。

### 演習 1: サービス接続を管理する

この演習では、Azure サブスクリプションとのサービス接続を構成し、CI パイプラインをインポートして実行します。

#### タスク 1: (完了している場合はスキップしてください) サービス接続を管理する

Azure Pipelines から外部およびリモート サービスへの接続を作成し、ジョブのタスクを実行できます。

このタスクでは、Azure CLI を使ってサービス プリンシパルを作成します。これにより、Azure DevOps で次のことができるようになります。

- Azure サブスクリプションでリソースをデプロイします。
- Docker イメージを Azure Container Registry にプッシュします。
- ロールの割り当てを追加して、Azure App Service で Azure Container Registry から Docker イメージをプルできるようにします。

> **注**:サービス プリンシパルが既にある場合は、次のタスクに直接進むことができます。

Azure Pipelines から Azure リソースをデプロイするには、サービス プリンシパルが必要です。

サービス プリンシパルは、パイプライン定義内から Azure サブスクリプションに接続するとき、またはプロジェクト設定ページから新しいサービス接続を作成するときに (自動オプション)、Azure パイプラインによって自動的に作成されます。 ポータルから、または Azure CLI を使用してサービス プリンシパルを手動で作成し、プロジェクト間で再利用することもできます。

1. ラボのコンピューターで Web ブラウザーを起動し、[**Azure portal**](https://portal.azure.com) に移動します。このラボで使用する Azure サブスクリプションで、所有者のロールがあり、このサブスクリプションに関連付けられている Microsoft Entra テナントで全体管理者のロールがあるユーザー アカウントを使ってサインインします。
1. Azure portal で、ページ上部の検索テキストボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。
1. **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、**[Bash]** を選択します。

   >**注**: **Cloud Shell** を初めて起動し、[**ストレージがマウントされていません**] というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、**[ストレージの作成]** を選択します。

1. **Bash** プロンプトの **[Cloud Shell]** ペインで、次のコマンドを実行して、Azure サブスクリプション ID の属性の値を取得します。

    ```bash
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > **注**:両方の値をテキスト ファイルにコピーします。 これらは、このラボの後半で必要になります。

1. **Bash** プロンプトの **[Cloud Shell]** ペインで、次のコマンドを実行してサービス プリンシパルを作成します。

    ```bash
    az ad sp create-for-rbac --name sp-az400-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > **注**:このコマンドは JSON 出力を生成します。 出力をテキスト ファイルにコピーします。 このラボで後ほど必要になります。

1. 次に、ラボ コンピューターから Web ブラウザーを起動し、Azure DevOps **eShopOnWeb** プロジェクトに移動します。 **[プロジェクトの設定] > [サービス接続] ([パイプライン] の下)** 、 **[新しいサービス接続]** の順にクリックします。
1. **[新しいサービス接続]** ブレードで、 **[Azure Resource Manager]** と **[次へ]** を選択します (必要に応じて下にスクロールします)。
1. **[サービス プリンシパル (手動)]** を選択し、 **[次へ]** をクリックします。
1. 前の手順で収集した情報を使って、空のフィールドに入力します。
    - サブスクリプション ID と名前。
    - サービス プリンシパル ID (appId)、サービス プリンシパル キー (パスワード)、テナント ID (テナント)。
    - **[サービス接続名]** に「**azure-connection**」と入力します。 この名前は、Azure サブスクリプションと通信するために Azure DevOps サービス接続が必要になるときに、YAML パイプラインで参照されます。

1. **[確認して保存]** をクリックします。

### 演習 2: CI パイプラインをインポートして実行する

この演習では、CI パイプラインをインポートして実行します。

#### タスク 1: CI パイプラインをインポートして実行する

1. **[パイプライン] > [パイプライン]** に移動します
1. **[新しいパイプライン]** ボタン (または、以前に作成したパイプラインがない場合は、**[パイプラインの作成]**) をクリックします
1. **[Azure Repos Git (YAML)]** を選びます
1. **eShopOnWeb** リポジトリを選びます
1. **[既存の Azure Pipelines YAML ファイル]** を選択します。
1. **メイン** ブランチと **/.ado/eshoponweb-ci-docker.yml** ファイルを選択し、**[続行]** をクリックします
1. YAML パイプライン定義で、次をカスタマイズします。
   - **YOUR-SUBSCRIPTION-ID** を、お使いの Azure サブスクリプション ID に置き換えます。
   - **rg-az400-container-NAME** と、パイプラインによって作成されるリソース グループ名 (既存のリソース グループでもかまいません)。

1. **[保存および実行]** をクリックし、パイプラインが正常に実行されるまで待ちます。

    > **注**: デプロイが完了するまでに数分かかる場合があります。

    CI の定義は以下のタスクで構成されます。
    - **リソース**: 以下のタスクで使われるリポジトリ ファイルをダウンロードします。
    - **AzureResourceManagerTemplateDeployment**: bicep テンプレートを使用して Azure Container Registry をデプロイします。
    - **PowerShell**: 前のタスクの出力から **ACR ログイン サーバー**の値を取得し、新しいパラメーター **acrLoginServer** を作成します
    - [**Docker**](https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/docker-v0?view=azure-pipelines) **- Build**: Docker イメージをビルドし、2 つのタグを作成します (最新と現在の BuildID)
    - **Docker - Push**: Azure Container Registry にイメージをプッシュします

1. パイプラインには、プロジェクト名に基づく名前が付けられます。 パイプラインを識別しやすくするために、**名前を変更**しましょう。 **[パイプライン] > [パイプライン]** に移動し、先ほど作成したパイプラインをクリックします。 省略記号と **[名前の変更または移動]** オプションをクリックします。 **eshoponweb-ci-docker** という名前を付け、 **[保存]** をクリックします。

1. [**Azure Portal**](https://portal.azure.com) に移動し、最近作成したリソース グループから Azure Container Registry を検索します (**rg-az400-container-NAME** という名前であるはずです)。 左側の **[サービス]** の下にある **[リポジトリ]** をクリックし、リポジトリ **eshoponweb/web** が作成されていることを確認します。 リポジトリ リンクをクリックすると、2 つのタグ (そのうちの 1 つが **最新**) が表示されます。これらはプッシュされたイメージです。 これが表示されない場合は、パイプラインの状態を調べます。

### 演習 3: CD パイプラインをインポートして実行する

この演習では、Azure サブスクリプションとのサービス接続を構成した後、CD パイプラインをインポートして実行します。

#### タスク 1: 新しいロールの割り当てを追加する

このタスクでは、新しいロールの割り当てを追加して、Azure App Service で Azure Container Registry から Docker イメージをプルできるようにします。

1. [**Azure Portal**](https://portal.azure.com) に移動します。
1. Azure portal で、ページ上部の検索テキストボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。
1. **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、**[Bash]** を選択します。
1. **Bash** プロンプトの **[Cloud Shell]** ペインで、次のコマンドを実行して、Azure サブスクリプション ID の属性の値を取得します。

    ```sh
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionId
    spId=$(az ad sp list --display-name sp-az400-azdo --query "[].id" --output tsv)
    echo $spId
    roleName=$(az role definition list --name "User Access Administrator" --query "[0].name" --output tsv)
    echo $roleName
    ```

1. サービス プリンシパル ID とロール名を取得したら、次のコマンドを実行してロールの割り当てを作成しましょう (**&lt;rg-az400-container-NAME&gt;** はリソース グループ名に置き換えてください)

    ```sh
    az role assignment create --assignee $spId --role $roleName --scope /subscriptions/$subscriptionId/resourceGroups/<rg-az400-container-NAME>
    ```

これで、コマンド実行の成功を確認する JSON 出力が表示されるはずです。

#### タスク 2: CD パイプラインをインポートして実行する

このタスクでは、CD パイプラインをインポートして実行します。

1. **[パイプライン] > [パイプライン]** に移動します
1. **[新しいパイプライン]** ボタンをクリックします
1. **[Azure Repos Git (YAML)]** を選びます
1. **eShopOnWeb** リポジトリを選びます
1. **[既存の Azure Pipelines の YAML ファイル]** を選択します
1. **メイン** ブランチと **/.ado/eshoponweb-cd-webapp-docker.yml** ファイルを選択し、**[続行]** をクリックします
1. YAML パイプライン定義で、次をカスタマイズします。
   - **YOUR-SUBSCRIPTION-ID** を、お使いの Azure サブスクリプション ID に置き換えます。
   - **rg-az400-container-NAME** をラボで前に定義したリソース グループ名にします。

1. **[保存および実行]** をクリックし、パイプラインが正常に実行されるまで待ちます。

    > **注**: デプロイが完了するまでに数分かかる場合があります。

    > **重要**: "TF402455: このブランチに対するプッシュは許可されていません。このブランチを更新するには pull request を使用する必要があります" というエラー メッセージが表示された場合は、前のラボで有効にした [レビュー担当者の最少数が必要です] というブランチ保護ルールをオフにする必要があります。

    CD の定義は以下のタスクで構成されます。
    - **リソース**: 以下のタスクで使われるリポジトリ ファイルをダウンロードします。
    - **AzureResourceManagerTemplateDeployment**: bicep テンプレートを使用して Azure App Service をデプロイします。
    - **AzureResourceManagerTemplateDeployment**: Bicep を使用してロールの割り当てを追加します

1. パイプラインには、プロジェクト名に基づく名前が付けられます。 パイプラインを識別しやすくするために、**名前を変更**しましょう。 **[パイプライン] > [パイプライン]** に移動し、先ほど作成したパイプラインをポイントします。 省略記号と **[名前の変更または移動]** オプションをクリックします。 **eshoponweb-cd-webapp-docker** という名前を付け、 **[保存]** をクリックします。

    > **注 1**: **/infra/webapp-docker.bicep** テンプレートを使用すると、アプリ サービス プラン、システム割り当てマネージド ID が有効な Web アプリが作成され、前にプッシュされた Docker イメージ **${acr.properties.loginServer}/eshoponweb/web:latest** が作成されます。

    > **注 2**:**/infra/webapp-to-acr-roleassignment.bicep** テンプレートを使用すると、Docker イメージを取得できるように、AcrPull ロールを使用して Web アプリの新しいロールの割り当てが作成されます。 これは最初のテンプレートで実行される可能性がありますが、ロールの割り当てが反映されるまでには時間がかかることがあるため、両方のタスクを個別に実行することをお勧めします。

#### タスク 3: ソリューションをテストする

1. Azure Portal で、最近作成したリソース グループに移動すると、3 つのリソース (Ap Service、App Service プラン、Container Registry) が表示されます。

1. App Service に移動し、 **[参照]** をクリックすると、Web サイトに移動します。

お疲れさまでした。 この演習では、カスタム Docker イメージを使って Web サイトをデプロイしました。

### 演習 4:Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。

>**注**:新規に作成し、使用しなくなったすべての Azure リソースを削除することを忘れないでください。 使用していないリソースを削除することで、予期しない料金が発生しなくなります。

#### タスク 1:Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。

1. Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1. 次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'rg-az400-container-')].name" --output tsv
    ```

1. 次のコマンドを実行して、このモジュールのラボ全体を通して作成したすべてのリソース グループを削除します。

    ```sh
    az group list --query "[?starts_with(name,'rg-az400-container-')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注**:コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## 確認

このラボでは、Azure DevOps CI/CD パイプラインを使用してカスタム Docker イメージを構築し、それを Azure Container Registry にプッシュして、コンテナーとして Azure App Service にデプロイする方法を学習しました。
