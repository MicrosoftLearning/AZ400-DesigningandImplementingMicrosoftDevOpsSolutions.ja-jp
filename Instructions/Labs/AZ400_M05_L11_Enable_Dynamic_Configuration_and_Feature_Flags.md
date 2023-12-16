---
lab:
  title: 動的構成と機能フラグを有効にする
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
---

# 動的構成と機能フラグを有効にする

## 受講生用ラボ マニュアル

## ラボの要件

- このラボには、**Microsoft Edge** または [Azure DevOps 対応ブラウザー](https://learn.microsoft.com/azure/devops/server/compatibility)が必要です。

- **Azure DevOps 組織を設定する:** このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization?view=azure-devops)に関するページの手順に従って作成してください。

- 既存の Azure サブスクリプションを識別するか、新しいものを作成します。

- Azure サブスクリプションで共同作成者または所有者のロールを持つ Microsoft アカウントまたは Microsoft Entra アカウントを持っていることを確認します。 詳細については、[「Azure portal を使用して Azure ロールの割り当てを一覧表示する」](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal)および[「Azure Active Directory で管理者ロールを表示して割当てる」](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal)を参照してください。

## ラボの概要

[Azure App Configuration](https://learn.microsoft.com/azure/azure-app-configuration/overview) は、アプリケーションの設定と機能フラグを一元管理するサービスを提供します。 近年のプログラム、特にクラウドで実行されるものには、分散されたコンポーネントが多数存在するのが一般的です。 これらのコンポーネント全体に構成設定を分散させることは、トラブルシューティングすることの難しいエラーがアプリケーションのデプロイ中に発生する原因となります。 App Configuration を使用すると、アプリケーションのすべての設定を 1 か所に格納して、そのアクセスをセキュリティで保護することができます。

## 目標

このラボを完了すると、次のことができるようになります。

- 動的な構成を有効にする。
- 機能フラグを管理する。

## 推定時間:60 分

## Instructions

### 演習 0:ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、[eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) に基づくリポジトリを含む新しい Azure DevOps プロジェクトで構成されます。

#### タスク 1: (完了している場合はスキップしてください) チーム プロジェクトを作成して構成する

このタスクでは、複数のラボで使用される **eShopOnWeb** Azure DevOps プロジェクトを作成します。

1. ラボ コンピューターのブラウザー ウィンドウで、Azure DevOps 組織を開きます。 **[新しいプロジェクト]** をクリックします。 プロジェクトに **eShopOnWeb** という名前を付け、 **[作業項目プロセス]** ドロップダウンで **[スクラム]** を選びます。 **[作成]** をクリックします。

#### タスク 2: (完了している場合はスキップしてください) eShopOnWeb Git リポジトリをインポートする

このタスクでは、複数のラボで使用される eShopOnWeb Git リポジトリをインポートします。

1. ラボ コンピューターのブラウザー ウィンドウで、Azure DevOps 組織と、前に作成した **eShopOnWeb** プロジェクトを開きます。 **[リポジトリ] > [ファイル]** 、 **[インポート]** をクリックします。 **[Git リポジトリをインポートする]** ウィンドウで、URL https://github.com/MicrosoftLearning/eShopOnWeb.git を貼り付けて、 **[インポート]** をクリックします。

2. リポジトリは次のように編成されています。
    - **.ado** フォルダーには、Azure DevOps の YAML パイプラインが含まれています。
    - **.devcontainer** フォルダーには、コンテナーを使って開発するためのセットアップが含まれています (VS Code でローカルに、または GitHub Codespaces で)。
    - **.azure** フォルダーには、一部のラボ シナリオで使用される Bicep&ARM コードとしてのインフラストラクチャ テンプレートが含まれています。
    - **.github** フォルダーには、YAML GitHub ワークフローの定義が含まれています。
    - **src** フォルダーには、ラボ シナリオで使用される .NET 6 Web サイトが含まれています。

#### タスク 3: (完了している場合はスキップしてください) メイン ブランチを既定のブランチとして設定する

1. **[リポジトリ] > [ブランチ]** に移動します。
2. **main** ブランチをポイントし、列の右側に表示される省略記号をクリックします。
3. **[既定のブランチとして設定]** をクリックします。

### 演習 1: (完了している場合はスキップしてください) CI/CD パイプラインをインポートして実行する

この演習では、CI パイプラインをインポートして実行し、サービスの Azure サブスクリプションとの接続を構成し、CD パイプラインをインポートして実行します。

#### タスク 1: CI パイプラインをインポートして実行する

まず、[eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml) という CI パイプラインをインポートします。

1. **[パイプライン] > [パイプライン]** に移動します。
2. **[パイプラインの作成]** ボタン (パイプラインがない場合) または **[新しいパイプライン]** ボタン (既に作成されたパイプラインがある場合) をクリックします。
3. **[Azure Repos Git (Yaml)]** を選びます。
4. **eShopOnWeb** リポジトリを選びます。
5. **[既存の Azure Pipelines YAML ファイル]** を選びます。
6. **/.ado/eshoponweb-ci.yml** ファイルを選び、 **[続行]** をクリックします。
7. **[実行]** ボタンをクリックしてパイプラインを実行します。
8. パイプラインには、プロジェクト名に基づく名前が付けられます。 パイプラインを識別しやすくするために、**名前を変更**しましょう。 **[パイプライン] > [パイプライン]** に移動し、先ほど作成したパイプラインをクリックします。 省略記号と **[名前の変更]/[削除]** オプションをクリックします。 **eshoponweb-ci** という名前を付け、 **[保存]** をクリックします。

#### タスク 2: サービス接続を管理する

Azure Pipelines から外部およびリモート サービスへの接続を作成し、ジョブのタスクを実行できます。

このタスクでは、Azure CLI を使ってサービス プリンシパルを作成します。これにより、Azure DevOps で次のことができるようになります。

- Azure サブスクリプションでリソースをデプロイする
- eShopOnWeb アプリケーションをデプロイする

> **注**:サービス プリンシパルが既にある場合は、次のタスクに直接進むことができます。

Azure Pipelines から Azure リソースをデプロイするには、サービス プリンシパルが必要です。

サービス プリンシパルは、パイプライン定義内から Azure サブスクリプションに接続するとき、またはプロジェクト設定ページから新しいサービス接続を作成するときに (自動オプション)、Azure パイプラインによって自動的に作成されます。 ポータルから、または Azure CLI を使用してサービス プリンシパルを手動で作成し、プロジェクト間で再利用することもできます。

1. ラボのコンピューターで Web ブラウザーを起動し、[**Azure portal**](https://portal.azure.com) に移動します。このラボで使用する Azure サブスクリプションで、所有者のロールがあり、このサブスクリプションに関連付けられている Microsoft Entra テナントで全体管理者のロールがあるユーザー アカウントを使ってサインインします。
2. Azure portal で、ページ上部の検索テキストボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。
3. **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、**[Bash]** を選択します。

   >**注**: **Cloud Shell** を初めて起動し、[**ストレージがマウントされていません**] というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、**[ストレージの作成]** を選択します。

4. **Bash** プロンプトの **[Cloud Shell]** ペインで、次のコマンドを実行して、Azure サブスクリプション ID の属性の値を取得します。

    ```sh
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > **注**:両方の値をテキスト ファイルにコピーします。 これらは、このラボの後半で必要になります。

5. **Bash** プロンプトの **[Cloud Shell]** ペインで、次のコマンドを実行してサービス プリンシパルを作成します。

    ```sh
    az ad sp create-for-rbac --name sp-az400-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > **注**:このコマンドは JSON 出力を生成します。 出力をテキスト ファイルにコピーします。 このラボで後ほど必要になります。

6. 次に、ラボ コンピューターから Web ブラウザーを起動し、Azure DevOps **eShopOnWeb** プロジェクトに移動します。 **[プロジェクトの設定] > [サービス接続] ([パイプライン] の下)** 、 **[新しいサービス接続]** の順にクリックします。

7. **[新しいサービス接続]** ブレードで、 **[Azure Resource Manager]** と **[次へ]** を選択します (必要に応じて下にスクロールします)。

8. **[サービス プリンシパル (手動)]** を選択し、 **[次へ]** をクリックします。

9. 前の手順で収集した情報を使って、空のフィールドに入力します。
    - サブスクリプション ID と名前
    - サービス プリンシパル ID (または clientId)、Key (または Password)、TenantId。
    - **[サービス接続名]** に「**azure subs**」と入力します。 この名前は、Azure サブスクリプションと通信するために Azure DevOps サービス接続が必要になるときに、YAML パイプラインで参照されます。

10. **[確認して保存]** をクリックします。

#### タスク 3: CD パイプラインをインポートして実行する

[eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml) という名前の CD パイプラインをインポートしてみましょう。

1. **[パイプライン] > [パイプライン]** に移動します。
2. **[新しいパイプライン]** ボタンをクリックします。
3. **[Azure Repos Git (Yaml)]** を選びます。
4. **eShopOnWeb** リポジトリを選びます。
5. **[既存の Azure Pipelines YAML ファイル]** を選びます。
6. **/.ado/eshoponweb-cd-webapp-code.yml** ファイルを選んで、 **[続行]** をクリックします。
7. YAML パイプライン定義で、次をカスタマイズします。
   - **YOUR-SUBSCRIPTION-ID** を使用する Azure サブスクリプション ID にします。
   - **az400eshop-NAME** の NAME を置き換えて、グローバルに一意にします。
   - **AZ400-EWebShop-NAME** をラボで前に定義したリソース グループ名にします。

8. **[保存および実行]** をクリックし、パイプラインが正常に実行されるまで待ちます。

    > **注**: デプロイが完了するまでに数分かかる場合があります。

    CD の定義は以下のタスクで構成されます。
    - **リソース**: CI パイプラインの完了に基づいて自動的にトリガーされるように準備されています。 また、bicep ファイルのリポジトリもダウンロードします。
    - **AzureResourceManagerTemplateDeployment**: bicep テンプレートを使用して Azure Web Apps をデプロイします。

9. パイプラインには、プロジェクト名に基づく名前が付けられます。 パイプラインを識別しやすくするために、**名前を変更**しましょう。 **[パイプライン] > [パイプライン]** に移動し、先ほど作成したパイプラインをクリックします。 省略記号と **[名前の変更]/[削除]** オプションをクリックします。 **eshoponweb-cd-webapp-code** という名前を付け、 **[保存]** をクリックします。

### 演習 2: Azure App Configuration を管理する

この演習では、Azure で App Configuration リソースを作成し、マネージド ID を有効にしてから、ソリューション全体をテストします。

> **注**: この演習にコーディングのスキルは必要ありません。 この Web サイトのコードは、Azure App Configuration の機能を既に実装しています。

アプリケーションにこの機能を実装する方法については、「[ASP.NET Core アプリで動的な構成を使用する](https://learn.microsoft.com/azure/azure-app-configuration/enable-dynamic-configuration-aspnet-core)」と「[Azure App Configuration で機能フラグを管理する](https://learn.microsoft.com/azure/azure-app-configuration/manage-feature-flags)」のチュートリアルを参照してください。

#### タスク 1: App Configuration リソースを作成する

1. Azure Portal で **App Configuration** サービスを検索します
2. **[Create app configuration] (アプリ構成の作成)** をクリックして、次を選びます。
    - 自分の Azure サブスクリプション。
    - 前に作成したリソース グループ (**AZ400-EWebShop-NAME** という名前になっているはずです)。
    - 場所。
    - たとえば、**appcs-NAME-REGION** のような一意の名前。
    - **Free** 価格レベルを選びます。
3. **[確認と作成]** 、 **[作成]** の順にクリックします。
4. App Configuration サービスを作成したら、 **[概要]** に移動し、 **[エンドポイント]** の値をコピーまたは保存します。

#### タスク 2: マネージド ID を有効にする

1. パイプラインを使ってデプロイされた Web アプリに移動します (名前はおそらく **az400-webapp-NAME** です)。
2. **[設定]** セクションで **[ID]** をクリックし、 **[システム割り当て]** セクションで状態を **[オン]** に切り替え、 **[保存] > [はい]** をクリックして、操作が完了するまで数秒待ちます。
3. App Configuration サービスに戻り、 **[アクセス制御]** 、 **[ロールの割り当てを追加]** の順にクリックします。
4. **[ロール]** セクションで **[App Configuration データ閲覧者]** を選びます。
5. **[メンバー]** セクションで **[マネージド ID]** を確認し、Web アプリのマネージド ID (同じ名前である必要があります) を選びます。
6. **[Review and assign] (確認と割り当て)** をクリックします。

#### タスク 3: Web アプリを構成する

Web サイトが App Configuration にアクセスしていることを確認するには、その構成を更新する必要があります。

1. Web アプリに戻ります。
2. **[設定]** セクションで **[構成]** をクリックします。
3. 2 つの新しいアプリケーション設定を追加します。
    - 1 つ目のアプリ設定
        - **名前:** UseAppConfig
        - **値:** true
    - 2 つ目のアプリ設定
        - **名前:** AppConfigEndpoint
        - **値:** <App Configuration Endpoint から以前に保存またはコピーした値。 https://appcs-NAME-REGION.azconfig.io のような値です>**

4. **[OK]** 、 **[保存]** の順にクリックし、設定が更新されるまで待ちます。
5. **[概要]** に移動して **[参照]** をクリックします
6. この手順では、App Configuration にデータが含まれていないため、Web サイトに変更はありません。 これは、次のタスクで行うことです。

#### タスク 4: 構成管理をテストする

1. Web サイトで、 **[ブランド]** ドロップダウン リストから **[Visual Studio]** を選び、矢印ボタン ( **>** ) をクリックします。
2. "THERE ARE NO RESULTS THAT MATCH YOUR SEARCH" (検索と一致する結果がありません) というメッセージが表示されます。** このラボの目標は、Web サイトのコードを更新したり、再デプロイしたりすることなく、この値を更新できるようにすることです。
3. このことを試すために、App Configuration に戻ります。
4. **[操作]** セクションで **[構成エクスプローラー]** を選びます。
5. **[作成] > [キー値]** の順にクリックして、次を追加します。
    - **キー:** eShopWeb:Settings:NoResultsMessage
    - **値**: <カスタム メッセージを入力してください>**
6. **[適用]** をクリックしてから Web サイトに戻り、ページを更新します。
7. これで、古い既定値ではなく、新しいメッセージが表示されます。

お疲れさまでした。 このタスクでは、Azure App Configuration で **Configuration エクスプローラー**をテストしました。

#### タスク 5: 機能フラグをテストする

引き続き機能マネージャーをテストしましょう。

1. このことを試すために、App Configuration に戻ります。
2. **[操作]** セクションで、 **[機能マネージャー]** を選びます。
3. **[作成]** をクリックし、次を追加します。
    - **機能フラグを有効にする:** オン
    - **機能フラグ名:** SalesWeekend
4. **[適用]** をクリックしてから Web サイトに戻り、ページを更新します。
5. 画像と "ALL T-SHIRTS ON SALE THIS WEEKEND" (今週末、すべての T シャツが特価に) というテキストが表示されます。
6. App Configuration でこの機能を無効にすると、画像が表示されなくなることがわかります。

お疲れさまでした。 このタスクでは、Azure App Configuration で**機能マネージャー**をテストしました。

### 演習 3:Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。

>**注**:新規に作成し、使用しなくなったすべての Azure リソースを削除することを忘れないでください。 使用していないリソースを削除することで、予期しない料金が発生しなくなります。

#### タスク 1:Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。

1. Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
2. 次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'AZ400-EWebShop-')].name" --output tsv
    ```

3. 次のコマンドを実行して、このモジュールのラボ全体を通して作成したすべてのリソース グループを削除します。

    ```sh
    az group list --query "[?starts_with(name,'AZ400-EWebShop-')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注**:コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## 確認

このラボでは、構成を動的に有効にし、機能フラグを管理する方法を学びました。
