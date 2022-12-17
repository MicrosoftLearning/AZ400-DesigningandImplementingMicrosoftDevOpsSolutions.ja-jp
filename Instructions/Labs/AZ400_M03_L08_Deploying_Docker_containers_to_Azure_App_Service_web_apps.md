---
lab:
  title: Docker コンテナーを Azure App Service Web アプリにデプロイする
  module: 'Module 03: Implement CI with Azure Pipelines and GitHub Actions'
---

# <a name="deploying-docker-containers-to-azure-app-service-web-apps"></a>Docker コンテナーを Azure App Service Web アプリにデプロイする

# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-requirements"></a>ラボの要件

- このラボには、**Microsoft Edge** または [Azure DevOps 対応ブラウザー](https://learn.microsoft.com/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)が必要です。

- **Azure DevOps 組織を設定する:** このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization?view=azure-devops)に関するページの手順に従って作成してください。

- 既存の Azure サブスクリプションを識別するか、新しいものを作成します。

- Azure サブスクリプションで共同作成者または所有者のロールを持つ Microsoft アカウントまたは Azure AD アカウントを持っていることを確認します。 詳細については、[「Azure portal を使用して Azure ロールの割り当てを一覧表示する」](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal)および[「Azure Active Directory で管理者ロールを表示して割当てる」](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal)を参照してください。

## <a name="lab-overview"></a>ラボの概要

このラボでは、Azure DevOps CI/CD パイプラインを使用してカスタム Docker イメージを構築し、それを Azure Container Registry にプッシュして、コンテナーとして Azure App Service にデプロイする方法を学習します。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- Microsoft がホストする Linux エージェントを使用して、カスタム Docker イメージを構築する
- Azure Container Registry にイメージをプッシュする
- Azure DevOps を使用して、Docker イメージをコンテナーとして Azure App Service にデプロイする

## <a name="estimated-timing-30-minutes"></a>推定時間:30 分

## <a name="instructions"></a>Instructions

### <a name="exercise-0-configure-the-lab-prerequisites"></a>演習 0:ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、[eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) に基づくリポジトリを含む新しい Azure DevOps プロジェクトで構成されます。

#### <a name="task-1-skip-if-done-create-and-configure-the-team-project"></a>タスク 1: (完了している場合はスキップしてください) チーム プロジェクトを作成して構成する

このタスクでは、複数のラボで使用される **eShopOnWeb** Azure DevOps プロジェクトを作成します。

1.  ラボ コンピューターのブラウザー ウィンドウで、Azure DevOps 組織を開きます。 **[新しいプロジェクト]** をクリックします。 プロジェクトに **eShopOnWeb** という名前を付け、 **[作業項目プロセス]** ドロップダウンで **[スクラム]** を選びます。 **[作成]** をクリックします。

#### <a name="task-2-skip-if-done-import-eshoponweb-git-repository"></a>タスク 2: (完了している場合はスキップしてください) eShopOnWeb Git リポジトリをインポートする

このタスクでは、複数のラボで使用される eShopOnWeb Git リポジトリをインポートします。

1.  ラボ コンピューターのブラウザー ウィンドウで、Azure DevOps 組織と、以前に作成した **eShopOnWeb** プロジェクトを開きます。 **[リポジトリ] > [ファイル]** 、 **[インポート]** をクリックします。 **[Git リポジトリをインポートする]** ウィンドウで、URL https://github.com/MicrosoftLearning/eShopOnWeb.git を貼り付けて、 **[インポート]** をクリックします。 

1.  リポジトリは次のように編成されています。
    - **.ado** フォルダーには Azure DevOps YAML パイプラインが含まれています
    - **.devcontainer** フォルダーには、コンテナーを使って開発するためのセットアップが含まれています (VS Code でローカルに、または GitHub Codespaces で)
    - **.azure** フォルダーには、一部のラボ シナリオで使用される Bicep&ARM コードとしてのインフラストラクチャ テンプレートが含まれています。
    - **.github** フォルダーには YAML GitHub ワークフロー定義が含まれています。
    - **src** フォルダーには、ラボ シナリオで使用される .NET 6 Web サイトが含まれています。

#### <a name="task-3-skip-if-done-set-main-branch-as-default-branch"></a>タスク 3: (完了している場合はスキップしてください) メイン ブランチを既定のブランチとして設定する

1. **[リポジトリ] > [ブランチ]** に移動します
1. **メイン** ブランチにカーソルを合わせ、列の右側にある省略記号をクリックします
1. **[既定のブランチとして設定]** をクリックします

### <a name="exercise-1-manage-the-service-connection"></a>演習 1: サービス接続を管理する

この演習では、Azure サブスクリプションとのサービス接続を構成し、CI パイプラインをインポートして実行します。

#### <a name="task-1-skip-if-done-manage-the-service-connection"></a>タスク 1: (完了している場合はスキップしてください) サービス接続を管理する

Azure Pipelines から外部およびリモート サービスへの接続を作成し、ジョブのタスクを実行できます。

このタスクでは、Azure CLI を使ってサービス プリンシパルを作成します。これにより、Azure DevOps で次のことができるようになります。
- Azure サブスクリプションでリソースをデプロイする
- Docker イメージを Azure Container Registry にプッシュする
- ロールの割り当てを追加して、Azure App Service が Azure Container Registry から Docker イメージをプルできるようにする

> **注**:サービス プリンシパルが既にある場合は、次のタスクに直接進むことができます。

Azure Pipelines から Azure リソースをデプロイするには、サービス プリンシパルが必要です。

サービス プリンシパルは、パイプライン定義内から Azure サブスクリプションに接続するとき、またはプロジェクト設定ページから新しいサービス接続を作成するときに (自動オプション)、Azure パイプラインによって自動的に作成されます。 ポータルから、または Azure CLI を使用してサービス プリンシパルを手動で作成し、プロジェクト間で再利用することもできます。 

1.  ラボのコンピューターで Web ブラウザーを起動し、[**Azure Portal**](https://portal.azure.com) に移動します。このラボで使用する Azure サブスクリプションで所有者ロールがあり、このサブスクリプションに関連のある Azure AD テナントでグローバル管理者ロールがあるユーザー アカウントを使ってサインインします。
1.  Azure portal で、ページ上部の検索テキストボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。 
1.  **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、**[Bash]** を選択します。 

   >**注**: **Cloud Shell** を初めて起動し、[**ストレージがマウントされていません**] というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、**[ストレージの作成]** を選択します。 

1.  **Bash** プロンプトの **[Cloud Shell]** ペインで、次のコマンドを実行して、Azure サブスクリプション ID の属性の値を取得します。 

    ```
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > **注**:両方の値をテキスト ファイルにコピーします。 これらは、このラボの後半で必要になります。

1.  **Bash** プロンプトの **[Cloud Shell]** ペインで、次のコマンドを実行してサービス プリンシパルを作成します。

    ```
    az ad sp create-for-rbac --name sp-az400-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > **注**:このコマンドは JSON 出力を生成します。 出力をテキスト ファイルにコピーします。 このラボで後ほど必要になります。

1. 次に、ラボ コンピューターから Web ブラウザーを起動し、Azure DevOps **eShopOnWeb** プロジェクトに移動します。 **[プロジェクトの設定] > [サービス接続] ([パイプライン] の下)** 、 **[新しいサービス接続]** の順にクリックします。

1. **[新しいサービス接続]** ブレードで、 **[Azure Resource Manager]** と **[次へ]** を選択します (下にスクロールする必要がある場合があります)。

1. **[サービス プリンシパル (手動)]** を選択し、 **[次へ]** をクリックします。

1. 前の手順で収集した情報を使って、空のフィールドに入力します。
    - サブスクリプション ID と名前
    - サービス プリンシパル ID (または clientId)、Key (または Password)、TenantId。
    - **[サービス接続名]** に「**azure-connection**」と入力します。 この名前は、Azure サブスクリプションと通信するために Azure DevOps サービス接続が必要になるときに、YAML パイプラインで参照されます。

1. **[確認して保存]** をクリックします。

### <a name="exercise-2-import-and-run-the-ci-pipeline"></a>演習 2: CI パイプラインをインポートして実行する

この演習では、CI パイプラインをインポートして実行します。

#### <a name="task-1-import-and-run-the-ci-pipeline"></a>タスク 1: CI パイプラインをインポートして実行する

1. **[パイプライン] > [パイプライン]** に移動します

1. **[新しいパイプライン]** ボタンをクリックします

1. **[Azure Repos Git (Yaml)]** を選択します

1. **eShopOnWeb** リポジトリを選択します

1. **[既存の Azure Pipelines の YAML ファイル]** を選択します

1. **/.ado/eshoponweb-ci-docker.yml** ファイルを選択し、 **[続行]** をクリックします

1. YAML パイプライン定義で、次をカスタマイズします。
- **YOUR-SUBSCRIPTION-ID** を使用する Azure サブスクリプション ID にします。
- **rg-az400-container-NAME** をラボで前に定義したリソース グループ名にします。

1. **[保存および実行]** をクリックし、パイプラインが正常に実行されるまで待ちます。

    > **注**: デプロイが完了するまでに数分かかる場合があります。

    CI の定義は以下のタスクで構成されます。
    - **Resources**: 以下のタスクで使用されるリポジトリ ファイルをダウンロードします。
    - **AzureResourceManagerTemplateDeployment**: bicep テンプレートを使用して Azure Container Registry をデプロイします。
    - **PowerShell**: 前のタスクの出力から **ACR ログイン サーバー**の値を取得し、新しいパラメーター **acrLoginServer** を作成します
    - [**Docker**](https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/docker-v0?view=azure-pipelines) **- Build**: Docker イメージをビルドし、2 つのタグを作成します (Latest と現在の BuildID)
    - **Docker - Push**: Azure Container Registry にイメージをプッシュします

1. パイプラインには、プロジェクト名に基づく名前が付けられます。 パイプラインを識別しやすくするために、**名前を変更**しましょう。 **[パイプライン] > [パイプライン]** に移動し、作成したばかりのパイプラインをクリックします。 省略記号と **[名前の変更]/[削除]** オプションをクリックします。 **eshoponweb-ci-docker** という名前を付け、 **[保存]** をクリックします。

1. [**Azure Portal**](https://portal.azure.com) に移動し、最近作成したリソース グループから Azure Container Registry を検索します (**rg-az400-container-NAME** という名前であるはずです)。 **eshoponweb/web** が作成され、2 つのタグ (そのうちの 1 つは **Latest**) が含まれていることを確認します。

### <a name="exercise-3-import-and-run-the-cd-pipeline"></a>演習 3: CD パイプラインをインポートして実行する

この演習では、Azure サブスクリプションとのサービス接続を構成した後、CD パイプラインをインポートして実行します。

#### <a name="task-1-add-a-new-role-assignment"></a>タスク 1: 新しいロールの割り当てを追加する

このタスクでは、新しいロールの割り当てを追加して、Azure App Service が Azure Container Registry から Docker イメージをプルできるようにします。

1. [**Azure Portal**](https://portal.azure.com) に移動します。
1. Azure portal で、ページ上部の検索テキストボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。 
1. **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、**[Bash]** を選択します。 

1. **Bash** プロンプトの **[Cloud Shell]** ペインで、次のコマンドを実行して、Azure サブスクリプション ID の属性の値を取得します。 

    ```sh
    spId=$(az ad sp list --display-name sp-az400-azdo --query "[].id" --output tsv)
    echo $spId
    roleName=$(az role definition list --name "User Access Administrator" --query [0].name --output tsv)
    echo $roleName
    ```

1. サービス プリンシパル ID とロール名を取得したら、次のコマンドを実行してロールの割り当てを作成しましょう (**rg-az400-container-NAME** はリソース グループ名に置き換えてください)

    ```sh
    az role assignment create --assignee $spId --role $roleName --resource-group "rg-az400-container-NAME"
    ```

これで、コマンド実行の成功を確認する JSON 出力が表示されるはずです。

#### <a name="task-2-import-and-run-the-cd-pipeline"></a>タスク 2: CD パイプラインをインポートして実行する

このタスクでは、CI パイプラインをインポートして実行します。

1. **[パイプライン] > [パイプライン]** に移動します

1. **[新しいパイプライン]** ボタンをクリックします

1. **[Azure Repos Git (Yaml)]** を選択します

1. **eShopOnWeb** リポジトリを選択します

1. **[既存の Azure Pipelines の YAML ファイル]** を選択します

1. **/.ado/eshoponweb-cd-webapp-docker.yml** ファイルを選択し、 **[続行]** をクリックします

1. YAML パイプライン定義で、次をカスタマイズします。
- **YOUR-SUBSCRIPTION-ID** を使用する Azure サブスクリプション ID にします。
- **rg-az400-container-NAME** をラボで前に定義したリソース グループ名にします。

1. **[保存および実行]** をクリックし、パイプラインが正常に実行されるまで待ちます。

    > **注**: デプロイが完了するまでに数分かかる場合があります。

    CI の定義は以下のタスクで構成されます。
    - **Resources**: 以下のタスクで使用されるリポジトリ ファイルをダウンロードします。
    - **AzureResourceManagerTemplateDeployment**: bicep テンプレートを使用して Azure App Service をデプロイします。
    - **AzureResourceManagerTemplateDeployment**: Bicep を使用してロールの割り当てを追加します

1. パイプラインには、プロジェクト名に基づく名前が付けられます。 パイプラインを識別しやすくするために、**名前を変更**しましょう。 **[パイプライン] > [パイプライン]** に移動し、作成したばかりのパイプラインをクリックします。 省略記号と **[名前の変更]/[削除]** オプションをクリックします。 **eshoponweb-cd-webapp-docker** という名前を付け、 **[保存]** をクリックします。

    > **注 1**: **/.azure/bicep/webapp-docker.bicep** テンプレートを使用すると、アプリ サービス プラン、システム割り当てマネージド ID が有効な Web アプリが作成され、前にプッシュされた Docker イメージ **${acr.properties.loginServer}/eshoponweb/web:latest** が参照されます。

    > **注 2**: **/.azure/bicep/webapp-to-acr-roleassignment.bicep** テンプレートを使用すると、Docker イメージを取得できるように、AcrPull ロールを使用して Web アプリの新しいロールの割り当てが作成されます。 これは最初のテンプレートで実行される可能性がありますが、ロールの割り当てが反映されるまでには時間がかかることがあるため、両方のタスクを個別に実行することをお勧めします。

    > **注 3**: 

#### <a name="task-3-test-the-solution"></a>タスク 3: ソリューションをテストする

1. Azure Portal で、最近作成したリソース グループに移動すると、3 つのリソース (Ap Service、App Service プラン、Container Registry) が表示されるはずです。

1. App Service に移動し、 **[参照]** をクリックすると、Web サイトに移動します。

お疲れさまでした。 この演習では、カスタム Docker イメージを使って Web サイトをデプロイしました。

### <a name="exercise-4-remove-the-azure-lab-resources"></a>演習 4:Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。

>**注**:新規に作成し、使用しなくなったすべての Azure リソースを削除することを忘れないでください。 使用していないリソースを削除することで、予期しない料金が発生しなくなります。

#### <a name="task-1-remove-the-azure-lab-resources"></a>タスク 1:Azure ラボ リソースを削除する

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

## <a name="review"></a>確認

このラボでは、Azure DevOps CI/CD パイプラインを使用してカスタム Docker イメージを構築し、それを Azure Container Registry にプッシュして、コンテナーとして Azure App Service にデプロイする方法を学習しました。
