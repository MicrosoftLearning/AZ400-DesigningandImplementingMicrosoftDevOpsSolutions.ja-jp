---
lab:
  title: Azure Load Testing を使ってアプリケーション パフォーマンスを監視する
  module: 'Module 08: Implement continuous feedback'
---

# Azure Load Testing を使ってアプリケーション パフォーマンスを監視する

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

この演習では、ラボの前提条件を設定します。

#### タスク 1: (完了している場合はスキップしてください) チーム プロジェクトを作成して構成する

このタスクでは、複数のラボで使用される **eShopOnWeb** Azure DevOps プロジェクトを作成します。

1. ラボ コンピューターのブラウザー ウィンドウで、Azure DevOps 組織を開きます。 **[新しいプロジェクト]** をクリックします。 プロジェクトに **eShopOnWeb** という名前を付け、 **[作業項目プロセス]** ドロップダウンで **[スクラム]** を選びます。 **[作成]** をクリックします。

   ![[新しいプロジェクトの作成] パネルのスクリーンショット。](images/create-project.png)

#### タスク 2: (完了している場合はスキップしてください) eShopOnWeb Git リポジトリをインポートする

このタスクでは、複数のラボで使用される eShopOnWeb Git リポジトリをインポートします。

1. ラボ コンピューターのブラウザー ウィンドウで、Azure DevOps 組織と、前に作成した **eShopOnWeb** プロジェクトを開きます。 **[リポジトリ]、[ファイル]**、**[インポート]** の順にクリックします。 **[Git リポジトリをインポートする]** ウィンドウで、URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> を貼り付けて、 **[インポート]** をクリックします。

   ![リポジトリのインポート パネルのスクリーンショット。](images/import-repo.png)

1. リポジトリは次のように編成されています。
   - **.ado** フォルダーには、Azure DevOps の YAML パイプラインが含まれています
   - **.devcontainer** フォルダーには、コンテナーを使って開発するためのセットアップが含まれています (VS Code でローカルに、または GitHub Codespaces で)
   - **infra** フォルダーには、一部のラボ シナリオで使用される Bicep&ARM コードとしてのインフラストラクチャ テンプレートが含まれています。
   - **.github** フォルダーには、YAML GitHub ワークフローの定義が含まれています。
   - **src** フォルダーには、ラボ シナリオで使用される .NET 8 Web サイトが含まれています。

#### タスク 3: (完了している場合はスキップしてください) メイン ブランチを既定のブランチとして設定する

1. **[リポジトリ]、[ブランチ]** の順に移動します。
1. **main** ブランチをポイントし、列の右側に表示される省略記号をクリックします。
1. **[既定のブランチとして設定]** をクリックします。

#### タスク 4: Azure リソースを作成する

このタスクでは、Azure portal でクラウド シェルを使って Azure Web アプリを作成します。

1. ラボ コンピューターから Web ブラウザーを起動して、[**Azure portal**](https://portal.azure.com) に移動し、サインインします。
1. Azure portal のツールバーで、検索テキスト ボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。
1. **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、**[Bash]** を選択します。

   > **注**: **Cloud Shell** を初めて起動し、[**ストレージがマウントされていません**] というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、**[ストレージの作成]** を選択します。

1. **Bash** プロンプトの **[Cloud Shell]** ペインで、次のコマンドを実行してリソース グループを作成します (`<region>` プレースホルダーを 'eastus' などのご自分の場所に最も近い Azure リージョンの名前に置き換えます)。

   ```bash
   RESOURCEGROUPNAME='az400m08l14-RG'
   LOCATION='<region>'
   az group create --name $RESOURCEGROUPNAME --location $LOCATION
   ```

1. 次のコマンドを実行して Windows App Service プランを作成するには、次のようにします。

   ```bash
   SERVICEPLANNAME='az400l14-sp'
   az appservice plan create --resource-group $RESOURCEGROUPNAME \
       --name $SERVICEPLANNAME --sku B3
   ```

1. 一意の名前を指定して Web アプリを作成します。

   ```bash
   WEBAPPNAME=az400eshoponweb$RANDOM$RANDOM
   az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME
   ```

   > **注**:Web アプリの名前を記録します。 このラボで後ほど必要になります。

### 演習 1:Azure DevOps で YAML を使用して CI/CD パイプラインをコードとして構成する

この演習では、Azure DevOps で YAML を使用して CI/CD パイプラインをコードとして構成します。

#### タスク 1: YAML ビルドとデプロイ定義を追加する

このタスクでは、既存のプロジェクトに YAML ビルドの定義を追加します。

1. **[パイプライン]** ハブで **[パイプライン]** に戻ります。
1. **[新しいパイプライン]** (または、初めて作成する場合は [パイプラインの作成]) をクリックします。

   > **注**: ウィザードを使い、プロジェクトに基づいて新しい YAML パイプラインの定義を作成します。

1. **[コードはどこにありますか?]** ペインで **[Azure Repos Git (YAML)]** オプションをクリックします。
1. **[リポジトリの選択]** ペインで **eShopOnWeb** をクリックします。
1. **[パイプラインの構成]** ペインで下にスクロールして **[スタート パイプライン]** を選びます。
1. [スタート パイプライン] のすべての行を**選択**し、削除します。
1. 以下から完全なテンプレート パイプラインを**コピー**します。ただし、変更を**保存する前**に、パラメーターを変更する必要があります。

   ```yml
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
                 command: "restore"
                 projects: "**/*.sln"
                 feedsToUse: "select"

             - task: DotNetCoreCLI@2
               displayName: Build
               inputs:
                 command: "build"
                 projects: "**/*.sln"

             - task: DotNetCoreCLI@2
               displayName: Publish
               inputs:
                 command: "publish"
                 publishWebProjects: true
                 arguments: "-o $(Build.ArtifactStagingDirectory)"

             - task: PublishBuildArtifacts@1
               displayName: Publish Artifacts ADO - Website
               inputs:
                 pathToPublish: "$(Build.ArtifactStagingDirectory)"
                 artifactName: Website

     - stage: Deploy
       displayName: Deploy to an Azure Web App
       jobs:
         - job: Deploy
           pool:
             vmImage: "windows-2019"
           steps:
             - task: DownloadBuildArtifacts@1
               inputs:
                 buildType: "current"
                 downloadType: "single"
                 artifactName: "Website"
                 downloadPath: "$(Build.ArtifactStagingDirectory)"
   ```

1. YAML 定義の最後で新しいライン上にカーソルを配置します。 **必ず前のタスク レベルのインデントにカーソルを置いてください**。

   > **注**:これは、新しいタスクが追加される場所になります。

1. ポータルの右側にある **[アシスタントを表示する]** をクリックします。 タスクの一覧で **[Azure App Service デプロイ]** タスクを見つけて選びます。
1. **[Azure App Service のデプロイ]** ペインで次の設定を指定して、**[追加]** をクリックします。

   - **[Azure サブスクリプション]** ドロップダウン リストで、先ほど作成したサービス接続を選択します。
   - **[App Service の種類]** で Windows 上の Web アプリが指定されていることを確認します。
   - **[App Service の名前]** ドロップダウン リストで、ラボで以前にデプロイした Web アプリの名前を選びます (\*\*az400eshoponweb...)。
   - **[パッケージまたはフォルダー]** テキスト ボックスで、既定値を `$(Build.ArtifactStagingDirectory)/**/Web.zip` に**更新**します。
   - **[アプリケーションと構成の設定]** を展開し、[アプリの設定] テキスト ボックスにキーと値のペア `-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development` を追加します。

1. **[追加]** ボタンをクリックし、[アシスタント] ペインで設定を確認します。

   > **注**:これにより、デプロイ タスクが YAML パイプラインの定義に自動的に追加されます。

1. エディターには次のようなコード スニペットが追加されていて、azureSubscription と WebappName パラメーターには実際の名前が反映されている必要があります。

   ```yml
   - task: AzureRmWebAppDeployment@4
     inputs:
       ConnectionType: "AzureRM"
       azureSubscription: "SERVICE CONNECTION NAME"
       appType: "webApp"
       WebAppName: "az400eshoponWeb369825031"
       packageForLinux: "$(Build.ArtifactStagingDirectory)/**/Web.zip"
       AppSettings: "-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development"
   ```

   > **注**:**packageForLinux** パラメーターは、このラボの状況では不適切ですが、Windows または Linux では有効です。

1. 更新を yml ファイルに保存する前に、もっとわかりやすい名前を付けます。 yaml エディター ウィンドウの上部に、**EShopOnweb/azure-pipelines-#.yml** と表示されます。 (# は数字で、通常は 1 ですが、設定によっては異なる場合があります。) **そのファイル名**を選択し、名前を **m08l14-pipeline.yml** に変更します

1. **[保存]** をクリックし、 **[保存]** ペインでもう一度 **[保存]** をクリックして、メイン ブランチに変更を直接コミットします。

   > **注**: 元の CI-YAML は新しいビルドを自動的にトリガーするように構成されていなかったため、手動で開始する必要があります。

1. Azure DevOps の左側のメニューから **[パイプライン]** に移動し、もう一度 **[パイプライン]** を選びます。 次に、**[すべて]** を選んで、最近使ったものだけでなく、すべてのパイプライン定義を開きます。

   > **注**: 前のラボ演習のパイプラインをすべて残している場合、次のスクリーンショットで示すように、パイプラインの既定の **eShopOnWeb (#)** シーケンス名がこの新しいパイプラインで再利用されている可能性があります。 パイプラインを選択します (おそらくシーケンス番号が最も大きいものを選択し、[編集] を選択して、m08l14-pipeline.yml コード ファイルを指していることを確認します)。

   ![eShopOnWeb の実行を示す Azure Pipelines のスクリーンショット。](images/m3/eshoponweb-m9l16-pipeline.png)

1. 表示されるペインで **[実行]** をクリックしてこのパイプラインが実行することを確認し、もう一度 **[実行]** をクリックして確認します。
1. **[Build .Net Core Solution] (.Net Core ソリューションをビルドする)** と **[Deploy to Azure Web App] (Azure Web アプリにデプロイする)** という 2 つの異なるステージが表示されることに注意してください。
1. パイプラインが開始するまで待ちます。

1. ビルド ステージの間に表示される警告はすべて**無視します**。 ビルド ステージが正常に完了するまで待ちます。 (実際のビルド ステージを選んで、ログからさらに詳しく確認できます)。

1. デプロイ ステージが開始できる状態になると、 **[アクセス許可が必要です]** というプロンプトとオレンジ色のバーが表示されます。

   ```text
   This pipeline needs permission to access a resource before this run can continue to Deploy to an Azure Web App
   ```

1. **[表示]** をクリックします
1. **[レビューを待機しています]** ペインで、 **[許可]** をクリックします。
1. **[許可]** ポップアップ ウィンドウでメッセージを確認し、 **[許可]** をクリックして確認します。
1. これにより、デプロイ ステージが開始します。 これが正常に完了するまで待ちます。

#### タスク 2: デプロイされたサイトを確認する

1. Azure portal を表示している Web ブラウザー ウィンドウに戻り、Azure Web アプリのプロパティが表示されているブレードに移動します。
1. [Azure Web アプリ] ブレードで **[概要]** をクリックします。[概要] ブレードで **[参照]** をクリックし、新しい Web ブラウザー タブでサイトを開きます。
1. デプロイされたサイトが新しいブラウザー タブに期待どおりに読み込まれ、eShopOnWeb eコマース Web サイトが表示されることを確認します。

### 演習 2: Azure Load Testing をデプロイして設定する

この演習では、Azure Load Testing リソースを Azure にデプロイし、現在実行されている Azure App Service に対してさまざまな Load Testing シナリオを構成します。

> **重要**: Azure Load Testing は、**有料サービスです**。 ロード テストを実行するためのコストが発生します。 追加コストが発生しないように、ラボの完了後にリソースをクリーンアップしてください。 1 か月の任意の部分でアクティブな 'ロード テスト リソース' ごとに、月額料金が課金され、含まれている 50 VUH にアクセスできます。 詳細については、[Azure Storage の価格に関するページ](https://azure.microsoft.com/pricing/details/load-testing)を参照してください。

#### タスク 1: Azure Load Testing をデプロイする

このタスクでは、Azure Load Testing リソースを Azure サブスクリプションにデプロイします。

1. Azure Portal (<https://portal.azure.com>) から、**[Azure リソースの作成]** に移動します。
1. [サービスとマーケットプレースの検索] 検索フィールドに「**`Azure Load Testing`**」と入力します。
1. 検索結果から (Microsoft が公開した) **[Azure Load Testing]** を選びます。
1. [Azure Load Testing] ページの **[作成]** をクリックしてデプロイ プロセスを開始します。
1. [Create a Load Testing Resource] (Load Testing リソースの作成) ページで、リソースのデプロイに必要な詳細を指定します。

   - **サブスクリプション**: Azure サブスクリプションを選びます
   - **リソース グループ**: 前の演習で Web App Service のデプロイに使ったリソース グループを選びます
   - **名前**: `eShopOnWebLoadTesting`
   - **リージョン**: 自分の地域に近いリージョンを選びます

   > **注**: Azure Load Testing サービスは、すべての Azure リージョンで使用できるわけではありません。

1. **[確認および作成]** をクリックして設定を検証します。
1. **[作成]** をクリックして確定し、Azure Load Testing リソースをデプロイします。
1. [デプロイが進行中です] ページに切り替わります。 デプロイが正常に完了するまで数分待ちます。
1. デプロイの進行状況ページから **[リソースに移動]** をクリックして、**eShopOnWebLoadTesting** Azure Load Testing リソースに移動します。

   > **注**: Azure Load Testing リソースのデプロイ中にブレードを閉じたり、Azure Portal を閉じたりした場合は、Azure Portal の検索フィールド、リソース、リソースの最近のリストからもう一度見つけることができます。

#### タスク 2: Azure Load Testing テストを作成する

このタスクでは、さまざまな負荷構成設定を使って、さまざまな Azure Load Testing テストを作成します。

1. **eShopOnWebLoadTesting** Azure Load Testing リソース ブレード内から、**[テスト]** の下の **[テスト]** に移動します。 **[+ 作成]** メニュー オプションをクリックして、**[URL ベースのテストの作成]** を選択します。
1. **[詳細設定を有効にする]** チェックボックスをオフにして、詳細設定を表示します。
1. ロード テストを作成するには、次のパラメーターと設定を完了します。

   - **[Test URL] (テストの URL)**: 前の演習でデプロイした Azure App Service の URL を **https:// を含めて**入力します (az400eshoponweb...azurewebsites.net)
   - **Specify Load (負荷の指定)** : 仮想ユーザー
   - **Number of Virtual Users (仮想ユーザーの数)** : 50
   - **[Test Duration (minutes)] (テストの継続時間 (分))**: 5
   - **[Ramp-up time (minutes)] (ランプアップ時間 (分))**: 1

1. **[確認と作成]** をクリックして、テストの構成を確認します (他のタブは変更しないでください)。 もう一度 **[作成]** をクリックします。
1. これにより、Load Testing のテストが開始され、テストが 5 分間実行されます。
1. テストを実行した状態で **eShopOnWebLoadTesting** Azure Load Testing リソース ページに戻り、**[テスト]** に移動して **[テスト]** を選び、テスト **Get_eshoponweb...** を確認します
1. 上部のメニューから **[作成]**、**[Create a URL-based test] (URL ベースのテストの作成)** の順にクリックして、2 つ目のロード テストを作成します。
1. もう 1 つのロード テストを作成するには、次のパラメーターと設定を完了します。

   - **[Test URL] (テストの URL)**: 前の演習でデプロイした Azure App Service の URL を **https:// を含めて**入力します (eShopOnWeb...azurewebsites.net)
   - **Specify Load (負荷の指定)** : 1 秒あたりの要求数 (RPS)
   - **Requests per second (RPS) (1 秒あたりの要求数 (RPS))** : 100
   - **Response time (milliseconds) (応答時間 (ミリ秒))** : 500
   - **[Test Duration (minutes)] (テストの継続時間 (分))**: 5
   - **[Ramp-up time (minutes)] (ランプアップ時間 (分))**: 1

1. **[確認と作成]** をクリックしてテストの構成を確認し、**[作成]** をもう一度クリックします。
1. テストが約 5 分間実行されます。

#### タスク 3: Azure Load Testing の結果を検証する

このタスクでは、Azure Load Testing TestRun の結果を検証します。

両方のクイック テストが完了したら、それらにいくつか変更を加え、結果を検証しましょう。

1. **Azure Load Testing** から **[テスト]** に移動します。 いずれかのテスト定義を選び、テストの 1 つを**クリック**して、より詳細なビューを開きます。 これにより、より詳細なテスト ページにリダイレクトされます。 ここで、結果の一覧から **TestRun_mm/dd/yyh-hh:hh** を選ぶことで、実際の実行の詳細を確認できます。
1. 詳細な **TestRun** ページから、Azure Load Testing シミュレーションの実際の結果を特定します。 値の一部を次に示します。

   - 負荷と合計要求数
   - Duration
   - 応答時間 (90 パーセンタイルの応答時間を反映した結果を秒単位で示します。これは、要求の 90% について、応答時間が所定の結果の範囲内になることを意味します)
   - 1 秒あたりの要求数のスループット

1. 以下に、これらの値のいくつかをダッシュボードのグラフの折れ線とグラフ ビューを使って表します。
1. 数分かけて両方のシミュレート テストの**結果を相互に比較します**。また、より多くのユーザーが App Service のパフォーマンスに与える**影響を特定します**。

### 演習 3: Azure Pipelines で CI/CD を使用してロード テストを自動化する

CI/CD パイプラインにロード テスト追加して、Azure Load Testing でのロード テストの自動化を開始します。 Azure portal でロード テストを実行した後、構成ファイルをエクスポートし、Azure Pipelines で CI/CD パイプラインを構成します (GitHub Actions 用に対して機能があります)。

この演習を完了すると、Azure Load Testing でロード テストを実行するように構成された CI/CD ワークフローが作成されます。

#### タスク 1: Azure DevOps サービス接続の詳細を特定する

このタスクでは、Azure DevOps サービス接続に必要なアクセス許可を付与します。

1. **Azure DevOps Portal** (<https://aex.dev.azure.com>) から **eShopOnWeb** プロジェクトに移動します。
1. 左下隅にある **[プロジェクトの設定]** を選びます。
1. **[パイプライン]** セクションで、 **[サービス接続]** を選びます。
1. [サービス接続] には、このラボ演習の開始時に Azure リソースのデプロイに使った Azure サブスクリプションの名前が含まれていることに注目してください。
1. **[サービス接続] を選びます**。 **[概要]** タブから **[詳細]** に移動し、**[サービス接続のロールの管理]** を選択します。
1. これで Azure portal にリダイレクトされ、そこからアクセス制御 (IAM) ブレードにリソース グループの詳細が開きます。

#### タスク 2: Azure Load Testing リソースへのアクセス許可を付与する

Azure Load Testing では、Azure RBAC を使用して、ロード テスト リソースで特定のアクティビティを実行するためのアクセス許可を付与します。 CI/CD パイプラインからロード テストを実行するには、**ロード テスト共同作成者**ロールを Azure DevOps サービス接続に付与します。

1. **[+ 追加]** を選択し、**[ロールの割り当ての追加]** を選択します。
1. **[ロール] タブ**のジョブ関数ロールの一覧で **[ロード テスト共同作成者]** を選びます。
1. **[メンバー]** タブで、**[メンバーの選択]** を選択して、自分のユーザー アカウントを見つけて選択し、**[選択]** をクリックします。
1. **[確認と 割り当て]** タブで、**[確認と割り当て]** を選択して ロールの割り当てを追加します。

Azure Pipelines ワークフロー定義でサービス接続を使用して、Azure ロード テスト リソースにアクセスできるようになりました。

#### タスク 3:ロード テスト入力ファイルをエクスポートし、Azure Repos にインポートする

CI/CD ワークフローで Azure Load Testing を使用してロード テストを実行するには、ロード テスト構成設定と入力ファイルをソース管理リポジトリに追加する必要があります。 既存のロード テストがある場合は、構成設定とすべての入力ファイルを Azure portal からダウンロードできます。

次の手順を実行して、Azure portal の既存のロード テスト用の入力ファイルをダウンロードします。

1. **Azure portal** で **Azure Load Testing** リソースに移動します。
1. 左側のペインで **[テスト]** を選んでロード テストの一覧を表示し、**対象のテスト**を選びます。
1. 使っているテストの実行の横にある省略記号 ( **...** ) を選び、 **[Download input file] (入力ファイルのダウンロード)** を選びます。
1. ブラウザーは、ロード テスト入力ファイルを含む zip 形式のフォルダーをダウンロードします。
1. 任意の zip ツールを使用して、入力ファイルを抽出します。 このフォルダーには次のファイルが格納されています。

   - _config.yaml_: ロード テストの YAML 構成ファイル。 このファイルは、CI/CD ワークフロー定義で参照します。
   - _quick_test.jmx_: JMeter テスト スクリプト

1. 抽出されたすべての入力ファイルをソース管理リポジトリにコミットします。 これを行うには、**Azure DevOps Portal** (<https://aex.dev.azure.com/>) に移動し、**eShopOnWeb** DevOps プロジェクトに移動します。
1. **[リポジトリ]** を選びます。 ソース コード フォルダー構造の **tests** サブフォルダーを選びます。 省略記号 (...) を選び、 **[新規作成] > [フォルダー]** を選びます。
1. フォルダー名として **jmeter**、ファイル名として **placeholder.txt** を指定します (注: フォルダーを空として作成することはできません)
1. **[コミット]** をクリックして、プレースホルダー ファイルと jmeter フォルダーの作成を確認します。
1. **[フォルダー構造]** から、新しく作成された **jmeter** サブフォルダーに移動します。 **省略記号 (...)** をクリックし、 **[ファイルのアップロード]** を選びます。
1. **[参照]** オプションを使って、抽出した zip ファイルの場所に移動し、**config.yaml** と **quick_test.jmx** の両方を選びます。
1. **[コミット]** をクリックして、ソース管理へのファイルのアップロードを確認します。
1. [リポジトリ] 内で、先ほど作成した **/tests/jmeter** サブフォルダーに移動します。
1. Load Testing の **config.yaml** ファイルを開きます。 **[編集]** をクリックすると、ファイルを編集できます。
1. **displayName** 属性と **testId** 属性を **ado_load_test** という値に置き換えます

  ![編集した構成ファイルのスクリーンショット。](images/config_edit.png)

#### タスク 4: CI/CD ワークフロー YAML 定義ファイルを更新する

1. ロード テストを作成して実行するために、Azure Pipelines ワークフロー定義には Azure DevOps Marketplace の **Azure Load Testing タスク**拡張機能が使われます。 Azure DevOps Marketplace で [Azure Load Testing タスク拡張機能](https://marketplace.visualstudio.com/items?itemName=AzloadTest.AzloadTesting)を開き、**[Get it free] (無料で入手)** を選択します。
1. Azure DevOps 組織を選択し、**[インストール]** を選択して拡張機能をインストールします。
1. Azure DevOps Portal と [プロジェクト] 内から **[パイプライン]** に移動し、この演習の開始時に作成したパイプラインを選びます。 **編集** をクリックします。
1. YAML スクリプトで**行 56** に移動し、Enter/Return キーを押して新しい空行を追加します (これは YAML ファイルのデプロイ ステージの直前です)。
1. 57 行目で、右側にある [タスク アシスタント] を選び、**Azure Load Testing** を検索します。
1. シナリオの正しい設定を使ってグラフィカル ペインを完了します。

   - Azure サブスクリプション: Azure リソースを実行するサブスクリプションを選びます
   - ロード テスト ファイル: '$(Build.SourcesDirectory)/tests/jmeter/config.yaml'
   - ロード テスト リソース グループ: Azure Load Testing リソースを保持するリソース グループ
   - ロード テスト リソース名: `eShopOnWebLoadTesting`
   - ロード テストの実行名: ado_run
   - ロード テストの実行の説明: ADO からのロード テスト

1. **[追加]** をクリックして YAML のスニペットとしてパラメーターの挿入を確認します
1. YAML スニペットのインデントでエラー (赤い波線) が発生している場合は、スニペットを正しく配置するために 2 つのスペースまたはタブを追加して修正します。
1. 次のサンプル スニペットは、YAML コードがどのようになるかを示しています

   ```yml
        - task: AzureLoadTest@1
         inputs:
           azureSubscription: 'AZURE DEMO SUBSCRIPTION'
           loadTestConfigFile: '$(Build.SourcesDirectory)/tests/jmeter/config.yaml'
           resourceGroup: 'az400m08l14-RG'
           loadTestResource: 'eShopOnWebLoadTesting'
           loadTestRunName: 'ado_run'
           loadTestRunDescription: 'load testing from ADO'
   ```

1. 挿入された YAML スニペットの下に、Enter/Return キーを押して新しい空行を追加します。
1. この空行の下に、パイプライン実行中の Azure Load Testing タスクの結果を表示する発行タスクのスニペットを追加します。

   ```yml
   - publish: $(System.DefaultWorkingDirectory)/loadTest
     artifact: loadTestResults
   ```

1. YAML スニペットのインデントでエラー (赤い波線) が発生している場合は、スニペットを正しく配置するために 2 つのスペースまたはタブを追加して修正します。
1. 両方のスニペットが CI/CD パイプラインに追加されたら、変更を **[保存]** します。
1. 保存したら、 **[実行]** をクリックしてパイプラインをトリガーします。
1. ブランチ (main) を確認し、 **[実行]** ボタンをクリックしてパイプラインの実行を開始します。
1. パイプラインの状態ページで **[ビルド]** ステージをクリックし、パイプライン内の異なるタスクの詳細ログを開きます。
1. パイプラインがビルド ステージをキックオフし、パイプラインのフロー内の **AzureLoadTest** タスクに到達するまで待ちます。
1. タスクが実行されている間に、Azure Portal で **Azure Load Testing** を参照して、パイプラインによって **adoloadtest1** という新しい RunTest がどのように作成されるかを確認します。 これを選ぶと、TestRun ジョブの結果値が表示されます。
1. Azure DevOps CI/CD のパイプライン実行ビューに戻ります。**AzureLoadTest タスク**は正常に完了しています。 詳細ログの出力から、ロード テストの結果の値も表示されます。

   ```text
   Task         : Azure Load Testing
   Description  : Automate performance regression testing with Azure Load Testing
   Version      : 1.2.30
   Author       : Microsoft Corporation
   Help         : https://docs.microsoft.com/azure/load-testing/tutorial-cicd-azure-pipelines#azure-load-testing-task
   ==============================================================================
   Test '0d295119-12d0-482d-94be-a7b84787c004' already exists
   Uploaded test plan for the test
   Creating and running a testRun for the test
   View the load test run in progress at: https://portal.azure.com/#blade/Microsoft_Azure_CloudNativeTesting/NewReport//resourceId/%2fsubscriptions%4b75-a1e0-27fb2ea7f9f4%2fresourcegroups%2faz400m08l14-RG%2fproviders%2fmicrosoft.loadtestservice%2floadtests%2feshoponwebloadtesting/testId/0d295119-12d0-787c004/testRunId/161046f1-d2d3-46f7-9d2b-c8a09478ce4c
   TestRun completed

   -------------------Summary ---------------
   TestRun start time: Mon Jul 24 2023 21:46:26 GMT+0000 (Coordinated Universal Time)
   TestRun end time: Mon Jul 24 2023 21:51:50 GMT+0000 (Coordinated Universal Time)
   Virtual Users: 50
   TestStatus: DONE

   ------------------Client-side metrics------------

   Homepage
   response time         : avg=1359ms min=59ms med=539ms max=16629ms p(90)=3127ms p(95)=5478ms p(99)=13878ms
   requests per sec      : avg=37
   total requests        : 4500
   total errors          : 0
   total error rate      : 0
   Finishing: AzureLoadTest

   ```

1. パイプライン実行の一環として自動ロード テストが実行されました。 最後のタスクでは、失敗の条件を指定します。つまり、Web アプリのパフォーマンスが一定のしきい値を下回っている場合、デプロイ ステージの開始は許可されません。

#### タスク 5: ロード テスト パイプラインに失敗と成功の条件を追加する

このタスクでは、ロード テストの失敗条件を使って、アプリケーションが品質要件を満たさない場合にアラートを受け取ります (結果として失敗したパイプラインが実行されます)。

1. Azure DevOps から eShopOnWeb プロジェクトに移動し、**[リポジトリ]** を開きます。
1. [リポジトリ] 内で、先ほど作成し、使った **/tests/jmeter** サブフォルダーに移動します。
1. Load Testing の \*config.yaml** ファイルを開きます。 **[編集]\*\* をクリックすると、ファイルを編集できます。
1. `failureCriteria: []` が存在する場合は置き換え、それ以外の場合は次のコード スニペットを追加します。

   ```text
   failureCriteria:
     - avg(response_time_ms) > 300
     - percentage(error) > 50
   ```

1. **[コミット]** と [コミット] をもう一度クリックして、config.yaml への変更を保存します。
1. **[パイプライン]** に戻り、**eShopOnWeb** パイプラインをもう一度実行します。 数分後、**AzureLoadTest** タスクの**失敗した**状態で実行が完了します。
1. パイプラインの詳細ログ ビューを開き、**AzureLoadtest** の詳細を検証します。 同様の出力例を次に示します。

   ```text
   Creating and running a testRun for the test
   View the load test run in progress at: https://portal.azure.com/#blade/Microsoft_Azure_CloudNativeTesting/NewReport//resourceId/%2fsubscriptions%2fb86d9ae1-7552-47fb2ea7f9f4%2fresourcegroups%2faz400m08l14-RG%2fproviders%2fmicrosoft.loadtestservice%2floadtests%2feshoponwebloadtesting/testId/0d295119-12d0-a7b84787c004/testRunId/f4bec76a-8b49-44ee-a388-12af34f0d4ec
   TestRun completed

   -------------------Summary ---------------
   TestRun start time: Mon Jul 24 2023 23:00:31 GMT+0000 (Coordinated Universal Time)
   TestRun end time: Mon Jul 24 2023 23:06:02 GMT+0000 (Coordinated Universal Time)
   Virtual Users: 50
   TestStatus: DONE

   -------------------Test Criteria ---------------
   Results           :1 Pass 1 Fail

   Criteria                  :Actual Value        Result
   avg(response_time_ms) > 300                       1355.29               FAILED
   percentage(error) > 50                                                  PASSED


   ------------------Client-side metrics------------

   Homepage
   response time         : avg=1355ms min=58ms med=666ms max=16524ms p(90)=2472ms p(95)=5819ms p(99)=13657ms
   requests per sec      : avg=37
   total requests        : 4531
   total errors          : 0
   total error rate      : 0
   ##[error]TestResult: FAILED
   Finishing: AzureLoadTest

   ```

1. ロード テスト出力の最後の行に **##[error]TestResult: FAILED** という出力があることに注目してください。平均応答時間が > 300 であるか、エラーの割合が > 20 という **FailCriteria** を定義したので、300 を超える平均応答時間が表示されると、タスクは失敗としてフラグが付けられます。

   > **注:** App Service のパフォーマンスを検証するという実際のシナリオを想像してください。パフォーマンスが一定のしきい値を下回る場合 (通常は Web アプリの負荷が高いことを意味します)、追加の Azure App Service への新しいデプロイをトリガーできます。 Azure ラボ環境では応答時間を制御できないため、失敗を保証するロジックに元に戻すことにしました。

1. パイプライン タスクの失敗状態は、実際には Azure Load Testing の要件条件の検証の成功を反映しています。

   > [!IMPORTANT]
   > 不要な料金が発生しないように、Azure portal で作成されたリソースを必ず削除してください。

## 確認

この演習では、Azure Pipelines を使って Azure App Service に Web アプリをデプロイし、TestRuns を使って Azure Load Testing Resource をデプロイしました。 次に、JMeter ロード テストの config.yaml ファイルを Azure Repos ソース管理に統合し、Azure Load Testing を使って CI/CD パイプラインを拡張しました。 最後の演習では、LoadTest の成功条件を定義する方法を学習しました。
