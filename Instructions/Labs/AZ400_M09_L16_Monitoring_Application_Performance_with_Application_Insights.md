---
lab:
  title: Azure Load Testing を使ったアプリケーション パフォーマンスの監視
  module: 'Module 09: Implement continuous feedback'
---

# Application Insights と Azure Load Testing を使ったアプリケーション パフォーマンスの監視

## 受講生用ラボ マニュアル

## ラボの要件

- このラボには、**Microsoft Edge** または [Azure DevOps 対応ブラウザー](https://docs.microsoft.com/azure/devops/server/compatibility)が必要です。

- **Azure DevOps 組織を設定する:** このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization)に関するページの手順に従って作成してください。

- 既存の Azure サブスクリプションを識別するか、新しいものを作成します。

- Azure サブスクリプションの所有者ロールと、Azure サブスクリプションに関連付けられた Microsoft Entra テナントの全体管理者ロールを持つ Microsoft アカウントまたは Microsoft Entra アカウントがあることを確認します。 詳細については、[「Azure portal を使用して Azure ロールの割り当てを一覧表示する」](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal)および[「Azure Active Directory で管理者ロールを表示して割当てる」](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal#view-my-roles)を参照してください。

## ラボの概要

**Azure Load Testing** は、大規模な負荷を生成できるフル マネージドのロード テスト サービスです。 サービスは、アプリケーションがどこにホストされているかにかかわらず、そのトラフィックをシミュレートします。 開発者、テスト担当者、品質保証 (QA) エンジニアは、それを使って、アプリケーションのパフォーマンス、スケーラビリティ、または容量を最適化できます。
Web アプリケーションのロード テストは、URL を使用してすぐに作成できます。テスト ツールの事前の知識は必要ありません。 ロード テストを大規模に実行するための複雑さとインフラストラクチャは、Azure Load Testing によって抽象化されます。
さらに高度なロード テスト シナリオでは、広く使われているオープンソースのロード パフォーマンス ツールである既存の Apache JMeter テスト スクリプトを再利用してロード テストを作成できます。 たとえば、テスト計画が複数のアプリケーション要求から成る可能性がある、HTTP 以外のエンドポイントを呼び出す必要がある、テストをより動的に行うために入力データとパラメーターを使用している、などの場合です。

このラボでは、Azure Load Testing を使って、現在実行されている Web アプリケーションに対するパフォーマンス テストをさまざまな負荷シナリオでシミュレートする方法について説明します。 最後に、Azure Load Testing を CI/CD パイプラインに統合する方法について説明します。 

## 目標

このラボを完了すると、次のことができるようになります。

- Azure App Service Web アプリをデプロイする。
- YAML ベースの CI/CD パイプラインを構成して実行します。
- Azure Load Testing をデプロイします。
- Azure Load Testing を使って Azure Web アプリのパフォーマンスを調べます。
- Azure Load Testing を CI/CD パイプラインに統合します。

## 推定時間:60 分

## Instructions

### 演習 0:ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、[eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) に基づくリポジトリを含む新しい Azure DevOps プロジェクトで構成されます。

#### タスク 1: (完了している場合はスキップしてください) チーム プロジェクトを作成して構成する

このタスクでは、複数のラボで使用される **eShopOnWeb** Azure DevOps プロジェクトを作成します。

1. ラボ コンピューターのブラウザー ウィンドウで、Azure DevOps 組織を開きます。 **[新しいプロジェクト]** をクリックします。 プロジェクトに **eShopOnWeb** という名前を付け、 **[作業項目プロセス]** ドロップダウンで **[スクラム]** を選びます。 **[作成]** をクリックします。

    ![Create Project](images/create-project.png)

#### タスク 2: (完了している場合はスキップしてください) eShopOnWeb Git リポジトリをインポートする

このタスクでは、複数のラボで使用される eShopOnWeb Git リポジトリをインポートします。

1. ラボ コンピューターのブラウザー ウィンドウで、Azure DevOps 組織と、前に作成した **eShopOnWeb** プロジェクトを開きます。 **[リポジトリ] > [ファイル]** 、 **[インポート]** をクリックします。 **[Git リポジトリをインポートする]** ウィンドウで、URL https://github.com/MicrosoftLearning/eShopOnWeb.git を貼り付けて、 **[インポート]** をクリックします。

    ![インポートリポジトリ](images/import-repo.png)

2. リポジトリは次のように編成されています。
    - **.ado** フォルダーには、Azure DevOps の YAML パイプラインが含まれています
    - **.devcontainer** フォルダーには、コンテナーを使って開発するためのセットアップが含まれています (VS Code でローカルに、または GitHub Codespaces で)
    - **.azure** フォルダーには、一部のラボ シナリオで使用される Bicep&ARM コードとしてのインフラストラクチャ テンプレートが含まれています。
    - **.github** フォルダーには、YAML GitHub ワークフローの定義が含まれています。
    - **src** フォルダーには、ラボ シナリオで使用される .NET 6 Web サイトが含まれています。

#### タスク 2: Azure リソースを作成する

このタスクでは、Azure portal でクラウド シェルを使って Azure Web アプリを作成します。

1. ラボのコンピューターで Web ブラウザーを起動し、[**Azure portal**](https://portal.azure.com) に移動します。このラボで使用する Azure サブスクリプションで、所有者のロールがあり、このサブスクリプションに関連付けられている Microsoft Entra テナントで全体管理者のロールがあるユーザー アカウントを使ってサインインします。
2. Azure portal のツールバーで、検索テキスト ボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。
3. **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、**[Bash]** を選択します。
    >**注**: **Cloud Shell** を初めて起動し、[**ストレージがマウントされていません**] というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、**[ストレージの作成]** を選択します。

4. **Bash** プロンプトの **[Cloud Shell]** ペインで、次のコマンドを実行してリソース グループを作成します (`<region>` プレースホルダーを 'eastus' などのご自分の場所に最も近い Azure リージョンの名前に置き換えます)。

    ```bash
    RESOURCEGROUPNAME='az400m09l16-RG'
    LOCATION='<region>'
    az group create --name $RESOURCEGROUPNAME --location $LOCATION
    ```

5. 次のコマンドを実行して Windows App Service プランを作成するには、次のようにします。

    ```bash
    SERVICEPLANNAME='az400l16-sp'
    az appservice plan create --resource-group $RESOURCEGROUPNAME \
        --name $SERVICEPLANNAME --sku B3 
    ```

6. 一意の名前を指定して Web アプリを作成します。

    ```bash
    WEBAPPNAME=partsunlimited$RANDOM$RANDOM
    az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME 
    ```

    > **注**:Web アプリの名前を記録します。 このラボで後ほど必要になります。

### 演習 1:Azure DevOps で YAML を使用して CI/CD パイプラインをコードとして構成する

この演習では、Azure DevOps で YAML を使用して CI/CD パイプラインをコードとして構成します。

#### タスク 1: YAML ビルドとデプロイ定義を追加する

このタスクでは、既存のプロジェクトに YAML ビルドの定義を追加します。

1. **[パイプライン]** ハブで **[パイプライン]** に戻ります。
2. **[最初のパイプラインを作成]** ウィンドウで、**[パイプラインの作成]** をクリックします。

    > **注**: ウィザードを使い、プロジェクトに基づいて新しい YAML パイプラインの定義を作成します。

3. **[コードはどこにありますか?]** ペインで **[Azure Repos Git (YAML)]** オプションをクリックします。
4. **[リポジトリの選択]** ペインで **eShopOnWeb** をクリックします。
5. **[パイプラインの構成]** ペインで下にスクロールして **[スタート パイプライン]** を選びます。
6. [スタート パイプライン] のすべての行を**選択**し、削除します。
7. 以下から完全なテンプレート パイプラインを**コピー**します。ただし、変更を**保存する前**に、パラメーターを変更する必要があります。

```
#Template Pipeline for CI/CD 
# trigger:
# - main

resources:
  repositories:
    - repository: self
      trigger: none

stages:
- stage: Build
  displayName: Build .Net Core Solution
  jobs:
  - job: Build
    pool:
      vmImage: ubuntu-latest
    steps:
    - task: DotNetCoreCLI@2
      displayName: Restore
      inputs:
        command: 'restore'
        projects: '**/*.sln'
        feedsToUse: 'select'

    - task: DotNetCoreCLI@2
      displayName: Build
      inputs:
        command: 'build'
        projects: '**/*.sln'
    
    - task: DotNetCoreCLI@2
      displayName: Publish
      inputs:
        command: 'publish'
        publishWebProjects: true
        arguments: '-o $(Build.ArtifactStagingDirectory)'
    
    - task: PublishBuildArtifacts@1
      displayName: Publish Artifacts ADO - Website
      inputs:
        pathToPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: Website
    
- stage: Deploy
  displayName: Deploy to an Azure Web App
  jobs:
  - job: Deploy
    pool:
      vmImage: 'windows-2019'
    steps:
    - task: DownloadBuildArtifacts@1
      inputs:
        buildType: 'current'
        downloadType: 'single'
        artifactName: 'Website'
        downloadPath: '$(Build.ArtifactStagingDirectory)'

```
4. YAML 定義の最後の新しい行にカーソルを置きます (行 69)。

    > **注**:これは、新しいタスクが追加される場所になります。

5. [コード] ペインの右側のタスクの一覧で、**[Azure App Service のデプロイ]** タスクを検索して選択します。
6. **[Azure App Service のデプロイ]** ペインで次の設定を指定して、**[追加]** をクリックします。

    - **[Azure サブスクリプション]** ドロップダウン リストで、このラボの前半に Azure リソースをデプロイした Azure サブスクリプションを選択し、**[承認]** をクリックします。プロンプトが表示されたら、Azure リソース デプロイで使用したものと同じユーザー アカウントを使用して認証します。
    - **[App Service の名前]** ドロップダウン リストで、このラボの前半にデプロイした Web アプリの名前を選択します。
    - **[パッケージまたはフォルダー]** テキスト ボックスで、既定値を `$(Build.ArtifactStagingDirectory)/**/Web.zip` に**更新**します。
7. **[追加]** ボタンをクリックし、[アシスタント] ペインで設定を確認します。

    > **注**:これにより、デプロイ タスクが YAML パイプラインの定義に自動的に追加されます。

8. エディターには次のようなコード スニペットが追加されていて、azureSubscription と WebappName パラメーターには実際の名前が反映されている必要があります。

> **注**:**packageForLinux** パラメーターは、このラボの状況では不適切ですが、Windows または Linux では有効です。

    ```yaml
        - task: AzureRmWebAppDeployment@4
          inputs:
            ConnectionType: 'AzureRM'
            azureSubscription: 'AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)'
            appType: 'webApp'
            WebAppName: 'eshoponWebYAML369825031'
            packageForLinux: '$(Build.ArtifactStagingDirectory)/**/Web.zip'
    ```
9. **[保存]** をクリックし、**[保存]** ペインでもう一度 **[保存]** をクリックして、master ブランチに変更を直接コミットします。

    > **注**: 元の CI-YAML は新しいビルドを自動的にトリガーするように構成されていなかったため、手動で開始する必要があります。

10. Azure DevOps の左側のメニューから **[パイプライン]** に移動し、もう一度 **[パイプライン]** を選びます。
11. **EShopOnWeb_MultiStageYAML** パイプラインを開いて、 **[パイプラインの実行]** をクリックします。
12. 表示されるペインで **[実行]** を確認します。
13. **[Build .Net Core Solution] (.Net Core ソリューションをビルドする)** と **[Deploy to Azure Web App] (Azure Web アプリにデプロイする)** という 2 つの異なるステージが表示されることに注意してください。
14. パイプラインが開始され、ビルド ステージが正常に完了するまで待ちます。
15. デプロイ ステージが開始できる状態になると、 **[アクセス許可が必要です]** というプロンプトとオレンジ色のバーが表示されます。

    ```text
    This pipeline needs permission to access a resource before this run can continue to Deploy to an Azure Web App
    ```

16. **[表示]** をクリックします
17. **[レビューを待機しています]** ペインで、 **[許可]** をクリックします。
18. **[許可]** ポップアップ ウィンドウでメッセージを確認し、 **[許可]** をクリックして確認します。
19. これにより、デプロイ ステージが開始します。 これが正常に完了するまで待ちます。

#### タスク 2: デプロイされたサイトを確認する

1. Azure portal を表示している Web ブラウザー ウィンドウに戻り、Azure Web アプリのプロパティが表示されているブレードに移動します。
2. [Azure Web アプリ] ブレードで **[概要]** をクリックします。[概要] ブレードで **[参照]** をクリックし、新しい Web ブラウザー タブでサイトを開きます。
3. デプロイされたサイトが新しいブラウザー タブに期待どおりに読み込まれ、EShopOnWeb eコマース Web サイトが表示されることを確認します。

### 演習 2: Azure Load Testing をデプロイして設定する

この演習では、Azure Load Testing リソースを Azure にデプロイし、現在実行されている Azure App Service に対してさまざまな Load Testing シナリオを構成します。

#### タスク 1: Azure Load Testing をデプロイする

このタスクでは、Azure Load Testing リソースを Azure サブスクリプションにデプロイします。

1. Azure Portal (https://portal.azure.com) から **[Azure リソースの作成]** に移動します。
2. [Search Services and marketplace] (サービスとマーケットプレースの検索) 検索フィールドに「**Azure Load Testing**」と入力します。
3. 検索結果から (Microsoft が公開した) **[Azure Load Testing]** を選びます。
4. [Azure Load Testing] ページの **[作成]** をクリックしてデプロイ プロセスを開始します。
5. [Create a Load Testing Resource] (Load Testing リソースの作成) ページで、リソースのデプロイに必要な詳細を指定します。
- **サブスクリプション**: Azure サブスクリプションを選びます
- **リソース グループ**: 前の演習で Web App Service のデプロイに使ったリソース グループを選びます
- **名前**: EShopOnWebLoadTesting
- **リージョン**: 自分の地域に近いリージョンを選びます

    > **注**: Azure Load Testing サービスは、すべての Azure リージョンで使用できるわけではありません。

6. **[確認および作成]** をクリックして設定を検証します。
7. **[作成]** をクリックして確定し、Azure Load Testing リソースをデプロイします。
8. [デプロイが進行中です] ページに切り替わります。 デプロイが正常に完了するまで数分待ちます。
9. デプロイの進行状況ページから **[Go to Resource] (リソースに移動)** をクリックして、**EShopOnWebLoadTesting** Azure Load Testing リソースに移動します。 

    > **注**: Azure Load Testing リソースのデプロイ中にブレードを閉じたり、Azure Portal を閉じたりした場合は、Azure Portal の検索フィールド、リソース、リソースの最近のリストからもう一度見つけることができます。 

#### タスク 2: Azure Load Testing テストを作成する

このタスクでは、さまざまな負荷構成設定を使って、さまざまな Azure Load Testing テストを作成します。 

10. **EShopOnWebLoadTesting** Azure Load Testing リソース ブレード内で、 **[Get Started with a quick test] (クイック テストを開始する)** を見つけて **[クイック テスト]** ボタンをクリックします。
11. ロード テストを作成するには、次のパラメーターと設定を完了します。
- **テスト用 URL**: 前の演習でデプロイした Azure App Service の URL を **https:// を含めて**入力します (EShopOnWeb...azurewebsites.net)
- **Specify Load (負荷の指定)** : 仮想ユーザー
- **Number of Virtual Users (仮想ユーザーの数)** : 50
- **Test Duration (Seconds) (テスト継続時間 (秒))** : 120
- **Ramp-up time (Seconds) (ランプアップ時間 (秒))** : 0
12. **[テストの実行]** をクリックして、テストの構成を確認します。
13. テストは約 2 分間実行されます。 
14. テストを実行した状態で **EShopOnWebLoadTesting** Azure Load Testing リソース ページに戻り、 **[テスト]** に移動して **[テスト]** を選び、テスト **Get_eshoponweb...** を確認します
15. 上部のメニューから **[作成]** 、 **[クイック テストの作成]** をクリックし、2 つ目のロード テストを作成します。
16. もう 1 つのロード テストを作成するには、次のパラメーターと設定を完了します。
- **テスト用 URL**: 前の演習でデプロイした Azure App Service の URL を **https:// を含めて**入力します (EShopOnWeb...azurewebsites.net)
- **Specify Load (負荷の指定)** : 1 秒あたりの要求数 (RPS)
- **Requests per second (RPS) (1 秒あたりの要求数 (RPS))** : 100
- **Response time (milliseconds) (応答時間 (ミリ秒))** : 500
- **Test Duration (seconds) (テスト継続時間 (秒))** : 120
- **Ramp-up time (Seconds) (ランプアップ時間 (秒))** : 0
17. **[テストの実行]** をクリックして、テストの構成を確認します。
18. テストは約 2 分間実行されます。

#### タスク 3: Azure Load Testing の結果を検証する

このタスクでは、Azure Load Testing TestRun の結果を検証します。 

両方のクイック テストが完了したら、それらにいくつか変更を加え、結果を検証しましょう。

19. **EShopOnWebLoadTesting** リソース ブレードで **[テスト]** に移動し、最初の Get_eshoponwebyaml... を選びます。 上部のメニューから **[編集]** をクリックします。
20. ここで、ポータルを使って **[テスト名]** を既定で生成されたものから、よりわかりやすいものに変更できます。 また、先ほど定義した任意のパラメーターを変更することもできます。
21. **[テストの編集]** ブレードから **[テスト計画]** タブに移動します。 
22. ここで **Apache JMeter** ロード テスト スクリプト ファイルを管理できます。これは Azure Load Testing がフレームワークとして使っているものです。 ファイル **quick_test.jmx** に注目してください。 ラボ仮想マシン上でこのファイル選んで **[開く]** を選びます。 ポップアップ ウィンドウで、エディターとして **Visual Studio Code** を選んでファイルを開きます。
23. ファイルの XML 言語構造に注目してください。

    > 注: Apache JMeter のさらに高度な構文の詳細と説明については、次の [Azure Load Testing - Jmeter](https://learn.microsoft.com/en-us/azure/load-testing/how-to-create-and-run-load-test-with-jmeter-script) リンクを参照してください。

24. 両方のテストが表示されている **[テスト]** ビューに戻り、いずれかを選び、いずれかのテストを**クリック**すると、より詳細なビューが開きます。 これにより、より詳細なテスト ページにリダイレクトされます。 ここで、結果の一覧から **TestRun_mm/dd/yyh-hh:hh** を選ぶことで、実際の実行の詳細を確認できます。
25. 詳細な **TestRun** ページから、Azure Load Testing シミュレーションの実際の結果を特定します。 値の一部を次に示します。
- 負荷と合計要求数
- Duration
- 応答時間 (90 パーセンタイルの応答時間を反映した結果を秒単位で示します。これは、要求の 90% について、応答時間が所定の結果の範囲内になることを意味します)
- 1 秒あたりの要求数のスループット
26. 以下に、これらの値のいくつかをダッシュボードのグラフの折れ線とグラフ ビューを使って表します。
27. 数分かけて両方のシミュレート テストの**結果を相互に比較します**。また、より多くのユーザーが App Service のパフォーマンスに与える**影響を特定します**。

### 演習 2: Azure DevOps Pipelines で CI/CD を使ってロード テストを自動化する

CI/CD パイプラインにロード テスト追加して、Azure Load Testing でのロード テストの自動化を開始します。 Azure portal でロード テストを実行した後、構成ファイルをエクスポートし、Azure Pipelines で CI/CD パイプラインを構成します (GitHub Actions 用に対して機能があります)。

この演習を完了すると、Azure Load Testing でロード テストを実行するように構成された CI/CD ワークフローが作成されます。

#### タスク 1: ADO サービス接続の詳細を特定する

このタスクでは、Azure DevOps Service Connection のサービス プリンシパルに必要なアクセス許可を付与します。

1. **Azure DevOps Portal** (https://dev.azure.com) から **EShopOnWeb** プロジェクトに移動します。
2. 左下隅にある **[プロジェクトの設定]** を選びます。
3. **[パイプライン]** セクションで、 **[サービス接続]** を選びます。
4. [サービス接続] には、このラボ演習の開始時に Azure リソースのデプロイに使った Azure サブスクリプションの名前が含まれていることに注目してください。
5. **[サービス接続] を選びます**。 **[概要]** タブから **[詳細]** に移動し、 **[サービス プリンシパルの管理]** を選びます。
6. これで Azure Portal にリダイレクトされ、ID オブジェクトの**サービス プリンシパル**の詳細が開きます。
7. **[表示名]** の値 (Name_of_ADO_Organization_EShopOnWeb_-b86d9ae1-7552-4b75-a1e0-27fb2ea7f9f4 のような形式です) をコピーしておきます。これは次の手順で必要になります。

#### タスク 2: サービス プリンシパルのアクセス許可を付与する

Azure Load Testing では、Azure RBAC を使用して、ロード テスト リソースで特定のアクティビティを実行するためのアクセス許可を付与します。 CI/CD パイプラインからロード テストを実行するには、**ロード テスト共同作成者**ロールをサービス プリンシパルに付与します。

1. **Azure portal** で **Azure Load Testing** リソースに移動します。
2. **[アクセス制御 (IAM)]** > [追加] > [ロールの割り当ての追加] の順に選びます。
3. **[ロール] タブ**のジョブ関数ロールの一覧で **[ロード テスト共同作成者]** を選びます。
4. **[メンバー]** タブで **[メンバーの選択]** を選び、先ほどコピーした**表示名**を使ってサービス プリンシパルを検索します。
5. **サービス プリンシパル**を選び、 **[選択]** を選びます。
6. **[確認と 割り当て]** タブで、**[確認と割り当て]** を選択して ロールの割り当てを追加します。

Azure Pipelines ワークフロー定義でサービス接続を使用して、Azure ロード テスト リソースにアクセスできるようになりました。

#### タスク 3: ロード テスト入力ファイルをエクスポートし、Azure DevOps ソース管理にインポートする

CI/CD ワークフローで Azure Load Testing を使用してロード テストを実行するには、ロード テスト構成設定と入力ファイルをソース管理リポジトリに追加する必要があります。 既存のロード テストがある場合は、構成設定とすべての入力ファイルを Azure portal からダウンロードできます。

次の手順を実行して、Azure portal の既存のロード テスト用の入力ファイルをダウンロードします。

1. **Azure portal** で **Azure Load Testing** リソースに移動します。
2. 左側のペインで **[テスト]** を選んでロード テストの一覧を表示し、**対象のテスト**を選びます。
3. 使っているテストの実行の横にある省略記号 ( **...** ) を選び、 **[Download input file] (入力ファイルのダウンロード)** を選びます。
4. ブラウザーは、ロード テスト入力ファイルを含む zip 形式のフォルダーをダウンロードします。
5. 任意の zip ツールを使用して、入力ファイルを抽出します。 このフォルダーには次のファイルが格納されています。

- *config.yaml*: ロード テストの YAML 構成ファイル。 このファイルは、CI/CD ワークフロー定義で参照します。
- *quick_test.jmx*: JMeter テスト スクリプト

6. 抽出されたすべての入力ファイルをソース管理リポジトリにコミットします。 これを行うには、**Azure DevOps Portal** (https://dev.azure.com) に移動し、**EShopOnWeb** DevOps プロジェクトに移動します。 
7. **[リポジトリ]** を選びます。 ソース コード フォルダー構造の **tests** サブフォルダーを選びます。 省略記号 (...) を選び、 **[新規作成] > [フォルダー]** を選びます。
8. フォルダー名として **jmeter**、ファイル名として **placeholder.txt** を指定します (注: フォルダーを空として作成することはできません)
9. **[コミット]** をクリックして、プレースホルダー ファイルと jmeter フォルダーの作成を確認します。
10. **[フォルダー構造]** から、新しく作成された **jmeter** サブフォルダーに移動します。 **省略記号 (...)** をクリックし、 **[ファイルのアップロード]** を選びます。
11. **[参照]** オプションを使って、抽出した zip ファイルの場所に移動し、**config.yaml** と **quick_test.jmx** の両方を選びます。
12. **[コミット]** をクリックして、ソース管理へのファイルのアップロードを確認します。

#### タスク 4: CI/CD ワークフロー YAML 定義ファイルを更新する

このタスクでは、Azure Load Testing - Azure DevOps Marketplace 拡張機能をインポートし、AzureLoadTest タスクを使って既存の CI/CD パイプラインを更新します。

1. ロード テストを作成して実行するために、Azure Pipelines ワークフロー定義には Azure DevOps Marketplace の **Azure Load Testing タスク**拡張機能が使われます。 Azure DevOps Marketplace で [Azure Load Testing タスク拡張機能](https://marketplace.visualstudio.com/items?itemName=AzloadTest.AzloadTesting)を開き、**[Get it free] (無料で入手)** を選択します。
2. Azure DevOps 組織を選択し、**[インストール]** を選択して拡張機能をインストールします。
3. Azure DevOps Portal と [プロジェクト] 内から **[パイプライン]** に移動し、この演習の開始時に作成したパイプラインを選びます。 **[編集]** をクリックします。
4. YAML スクリプトで**行 56** に移動し、Enter/Return キーを押して新しい空行を追加します (これは YAML ファイルのデプロイ ステージの直前です)。
5. 57 行目で、右側にある [タスク アシスタント] を選び、**Azure Load Testing** を検索します。
6. シナリオの正しい設定を使ってグラフィカル ペインを完了します。
- Azure サブスクリプション: Azure リソースを実行するサブスクリプションを選びます
- ロード テスト ファイル: '$(Build.SourcesDirectory)/tests/jmeter/config.yaml' 
- ロード テスト リソース グループ: Azure Load Testing リソースを保持するリソース グループ
- ロード テスト リソース名: ESHopOnWebLoadTesting
- ロード テストの実行名: ado_run
- ロード テストの実行の説明: ADO からのロード テスト
7. **[追加]** をクリックして YAML のスニペットとしてパラメーターの挿入を確認します
8. YAML スニペットのインデントでエラー (赤い波線) が発生している場合は、スニペットを正しく配置するために 2 つのスペースまたはタブを追加して修正します。  
9. 次のサンプル スニペットは、YAML コードがどのようになるかを示しています
```
     - task: AzureLoadTest@1
      inputs:
        azureSubscription: 'AZURE DEMO SUBSCRIPTION(b86d9ae1-1234-4b75-a8e7-27fb2ea7f9f4)'
        loadTestConfigFile: '$(Build.SourcesDirectory)/tests/jmeter/config.yaml'
        resourceGroup: 'az400m05l11-RG'
        loadTestResource: 'EShopOnWebLoadTesting'
        loadTestRunName: 'ado_run'
        loadTestRunDescription: 'load testing from ADO'
```
10. 挿入された YAML スニペットの下に、Enter/Return キーを押して新しい空行を追加します。 
11. この空行の下に、パイプライン実行中の Azure Load Testing タスクの結果を表示する発行タスクのスニペットを追加します。

```
    - publish: $(System.DefaultWorkingDirectory)/loadTest
      artifact: loadTestResults
```
12.  YAML スニペットのインデントでエラー (赤い波線) が発生している場合は、スニペットを正しく配置するために 2 つのスペースまたはタブを追加して修正します。  
13. 両方のスニペットが CI/CD パイプラインに追加されたら、変更を **[保存]** します。 
14. 保存したら、 **[実行]** をクリックしてパイプラインをトリガーします。
15. ブランチ (main) を確認し、 **[実行]** ボタンをクリックしてパイプラインの実行を開始します。
16. パイプラインの状態ページで **[ビルド]** ステージをクリックし、パイプライン内の異なるタスクの詳細ログを開きます。
17. パイプラインがビルド ステージをキックオフし、パイプラインのフロー内の **AzureLoadTest** タスクに到達するまで待ちます。 
18. タスクが実行されている間に、Azure Portal で **Azure Load Testing** を参照して、パイプラインによって **adoloadtest1** という新しい RunTest がどのように作成されるかを確認します。 これを選ぶと、TestRun ジョブの結果値が表示されます。
19. Azure DevOps CI/CD のパイプライン実行ビューに戻ります。**AzureLoadTest タスク**は正常に完了しています。 詳細ログの出力から、ロード テストの結果の値も表示されます。

```
Task         : Azure Load Testing
Description  : Automate performance regression testing with Azure Load Testing
Version      : 1.2.30
Author       : Microsoft Corporation
Help         : https://docs.microsoft.com/azure/load-testing/tutorial-cicd-azure-pipelines#azure-load-testing-task
==============================================================================
Test '0d295119-12d0-482d-94be-a7b84787c004' already exists
Uploaded test plan for the test
Creating and running a testRun for the test
View the load test run in progress at: https://portal.azure.com/#blade/Microsoft_Azure_CloudNativeTesting/NewReport//resourceId/%2fsubscriptions%4b75-a1e0-27fb2ea7f9f4%2fresourcegroups%2faz400m05l11-rg%2fproviders%2fmicrosoft.loadtestservice%2floadtests%2feshoponwebloadtesting/testId/0d295119-12d0-787c004/testRunId/161046f1-d2d3-46f7-9d2b-c8a09478ce4c
TestRun completed

-------------------Summary ---------------
TestRun start time: Mon Jul 24 2023 21:46:26 GMT+0000 (Coordinated Universal Time)
TestRun end time: Mon Jul 24 2023 21:51:50 GMT+0000 (Coordinated Universal Time)
Virtual Users: 50
TestStatus: DONE

------------------Client-side metrics------------

Homepage
response time        : avg=1359ms min=59ms med=539ms max=16629ms p(90)=3127ms p(95)=5478ms p(99)=13878ms
requests per sec     : avg=37
total requests       : 4500
total errors         : 0
total error rate     : 0
Finishing: AzureLoadTest

```
20. パイプライン実行の一環として自動ロード テストが実行されました。 最後のタスクでは、失敗の条件を指定します。つまり、Web アプリのパフォーマンスが一定のしきい値を下回っている場合、デプロイ ステージの開始は許可されません。 

#### タスク 5: ロード テスト パイプラインに失敗と成功の条件を追加する

このタスクでは、ロード テストの失敗条件を使って、アプリケーションが品質要件を満たさない場合にアラートを受け取ります (結果として失敗したパイプラインが実行されます)。

1. Azure DevOps から EShopOnWeb プロジェクトに移動し、 **[リポジトリ]** を開きます。
2. [リポジトリ] 内で、先ほど作成し、使った **/tests/jmeter** サブフォルダーに移動します。
3. Load Testing の *config.yaml** ファイルを開きます。 **[編集]** をクリックすると、ファイルを編集できます。
4. ファイルの最後に、次のコード スニペットを追加します。

```
failureCriteria:
  - avg(response_time_ms) > 300
  - percentage(error) > 50
```
5. **[コミット]** と [コミット] をもう一度クリックして、config.yaml への変更を保存します。
6. **[パイプライン]** に戻り、**EShopOnWeb** パイプラインをもう一度実行します。 数分後、**AzureLoadTest** タスクの**失敗した**状態で実行が完了します。 
7. パイプラインの詳細ログ ビューを開き、**AzureLoadtest** の詳細を検証します。 同様の出力例を次に示します。

```
Creating and running a testRun for the test
View the load test run in progress at: https://portal.azure.com/#blade/Microsoft_Azure_CloudNativeTesting/NewReport//resourceId/%2fsubscriptions%2fb86d9ae1-7552-47fb2ea7f9f4%2fresourcegroups%2faz400m05l11-rg%2fproviders%2fmicrosoft.loadtestservice%2floadtests%2feshoponwebloadtesting/testId/0d295119-12d0-a7b84787c004/testRunId/f4bec76a-8b49-44ee-a388-12af34f0d4ec
TestRun completed

-------------------Summary ---------------
TestRun start time: Mon Jul 24 2023 23:00:31 GMT+0000 (Coordinated Universal Time)
TestRun end time: Mon Jul 24 2023 23:06:02 GMT+0000 (Coordinated Universal Time)
Virtual Users: 50
TestStatus: DONE

-------------------Test Criteria ---------------
Results          :1 Pass 1 Fail

Criteria                     :Actual Value        Result
avg(response_time_ms) > 300                       1355.29               FAILED
percentage(error) > 50                                                  PASSED


------------------Client-side metrics------------

Homepage
response time        : avg=1355ms min=58ms med=666ms max=16524ms p(90)=2472ms p(95)=5819ms p(99)=13657ms
requests per sec     : avg=37
total requests       : 4531
total errors         : 0
total error rate     : 0
##[error]TestResult: FAILED
Finishing: AzureLoadTest
```

8. ロード テスト出力の最後の行に **##[error]TestResult: FAILED** という出力があることに注目してください。平均応答時間が > 300 であるか、エラーの割合が > 20 という **FailCriteria** を定義したので、300 を超える平均応答時間が表示されると、タスクは失敗としてフラグが付けられます。 

    > 注: App Service のパフォーマンスを検証するという実際のシナリオを想像してください。パフォーマンスが一定のしきい値を下回る場合 (通常は Web アプリの負荷が高いことを意味します)、追加の Azure App Service への新しいデプロイをトリガーできます。 Azure ラボ環境では応答時間を制御できないため、失敗を保証するロジックに元に戻すことにしました。

9.  パイプライン タスクの失敗状態は、実際には Azure Load Testing の要件条件の検証の成功を反映しています。

### 演習 3:Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。

>**注**:新規に作成し、使用しなくなったすべての Azure リソースを削除することを忘れないでください。 使用していないリソースを削除することで、予期しない料金が発生しなくなります。

#### タスク 1:Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。

1. Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
2. 次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'az400m16l01')].name" --output tsv
    ```

3. 次のコマンドを実行して、このモジュールのラボ全体を通して作成したすべてのリソース グループを削除します。

    ```sh
    az group list --query "[?starts_with(name,'az400m16l01')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注**:コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## 確認

この演習では、Azure Pipelines を使って Azure App Service に Web アプリをデプロイし、TestRuns を使って Azure Load Testing Resource をデプロイしました。 次に、Jmeter ロード テストの config.yaml ファイルを Azure DevOps リポジトリ ソース管理に統合し、Azure Load Testing を使って CI/CD パイプラインを拡張しました。 最後の演習では、LoadTest の成功条件を定義する方法を学習しました。
