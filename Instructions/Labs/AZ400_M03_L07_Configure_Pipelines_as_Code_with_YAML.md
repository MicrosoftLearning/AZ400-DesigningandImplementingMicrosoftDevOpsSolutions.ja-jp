---
lab:
  title: YAML を使用してパイプラインをコードとして構成する
  module: 'Module 03: Design and implement a release strategy'
---

# YAML を使用してパイプラインをコードとして構成する

## ラボの要件

- このラボには、**Microsoft Edge** または [Azure DevOps 対応ブラウザー](https://docs.microsoft.com/azure/devops/server/compatibility)が必要です。

- **Azure DevOps 組織を設定する:** このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization)に関するページの手順に従って作成してください。

- 既存の Azure サブスクリプションを識別するか、新しいものを作成します。

- Azure サブスクリプションの所有者ロールと、Azure サブスクリプションに関連付けられた Microsoft Entra テナントの全体管理者ロールを持つ Microsoft アカウントまたは Microsoft Entra アカウントがあることを確認します。 詳細については、[「Azure portal を使用して Azure ロールの割り当てを一覧表示する」](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal)および[「Azure Active Directory で管理者ロールを表示して割当てる」](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal)を参照してください。

## ラボの概要

多くのチームは YAML を使用してビルドを定義し、パイプラインをリリースすることを好みます。 これにより、ビジュアル デザイナーを使用した場合と同じパイプラインの機能にアクセスできます。ただし、マークアップ ファイルは他のソース ファイルと同様に管理できます。 YAML ビルドの定義は、該当するファイルをリポジトリのルートに加えるだけでプロジェクトに追加できます。 Azure DevOps はまた、人気の高いプロジェクトの種類のための既定テンプレート、および YAML デザイナーを提供して、ビルドおよびリリース タスクの定義プロセスを簡素化します。

## 目標

このラボを完了すると、次のことができるようになります。

- Azure DevOps で YAML を使用して CI/CD パイプラインをコードとして構成する。

## 推定時間:45 分

## Instructions

### 演習 0:ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。

#### タスク 1: (完了している場合はスキップしてください) チーム プロジェクトを作成して構成する

このタスクでは、複数のラボで使用される **eShopOnWeb_MultiStageYAML** Azure DevOps プロジェクトを作成します。

1. ラボ コンピューターのブラウザー ウィンドウで、Azure DevOps 組織を開きます。 **[新しいプロジェクト]** をクリックします。 プロジェクトに「**eShopOnWeb_MultiStageYAML**」という名前を付け、他のフィールドは既定値のままにします。 **[作成]** をクリックします。

   ![[新しいプロジェクトの作成] パネルのスクリーンショット。](images/create-project.png)

#### タスク 2: (完了している場合はスキップしてください) eShopOnWeb Git リポジトリをインポートする

このタスクでは、複数のラボで使用される eShopOnWeb Git リポジトリをインポートします。

1. ラボ コンピューターのブラウザー ウィンドウで、Azure DevOps 組織と、前に作成した **eShopOnWeb_MultiStageYAML** プロジェクトを開きます。 **[リポジトリ]、[ファイル]**、**[リポジトリをインポートする]** の順にクリックします。 **インポート** を選択します。 **[Git リポジトリをインポートする]** ウィンドウで、URL https://github.com/MicrosoftLearning/eShopOnWeb.git を貼り付けて、 **[インポート]** をクリックします。

   ![[リポジトリのインポート] パネルのスクリーンショット。](images/import-repo.png)

1. リポジトリは次のように編成されています。
   - **.ado** フォルダーには、Azure DevOps の YAML パイプラインが含まれています。
   - **.devcontainer** フォルダーには、コンテナーを使って開発するためのセットアップが含まれています (VS Code でローカルに、または GitHub Codespaces で)。
   - **infra** フォルダーには、一部のラボ シナリオで使用される Bicep および ARM のコードとしてのインフラストラクチャ テンプレートが含まれています。
   - **.github** フォルダーには、YAML GitHub ワークフローの定義が含まれています。
   - **src** フォルダーには、ラボ シナリオで使用される .NET 8 Web サイトが含まれています。

1. **[リポジトリ]、[ブランチ]** の順に移動します。
1. **main** ブランチをポイントし、列の右側に表示される省略記号をクリックします。
1. **[既定のブランチとして設定]** をクリックします。

    > **注**: メイン ブランチが既に既定のブランチである場合、**[既定のブランチとして設定する]** オプションは淡色表示されます。この場合は、指示に沿って続けます。

#### タスク 3:Azure リソースを作成します

このタスクでは、Azure portal を使って Azure Web アプリを作成します。

1. ラボのコンピューターで Web ブラウザーを起動し、[**Azure portal**](https://portal.azure.com) に移動します。このラボで使用する Azure サブスクリプションで、所有者のロールがあり、このサブスクリプションに関連付けられている Microsoft Entra テナントで全体管理者のロールがあるユーザー アカウントを使ってサインインします。
1. Azure portal のツールバーで、検索テキスト ボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。
1. **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、**[Bash]** を選択します。

   > **注**:**Cloud Shell** の初回起動時に、**[はじめに]** というポップアップが表示される場合は、**[ストレージ アカウントは必要ありません]** と、このラボで使用しているサブスクリプションを選択して、**[適用]** をクリックします。

1. **[Cloud Shell]** ペインの **Bash** プロンプトから、次のコマンドを実行してリソース グループを作成します (`<region>` プレースホルダーを、"centralus"、"westeurope" など、自分の場所に最も近い Azure リージョンの名前に置き換えます)。

   ```bash
   LOCATION='<region>'
   ```

   ```bash
   RESOURCEGROUPNAME='az400m03l07-RG'
   az group create --name $RESOURCEGROUPNAME --location $LOCATION
   ```

1. 次のコマンドを実行して Windows App Service プランを作成するには、次のようにします。

   ```bash
   SERVICEPLANNAME='az400m03l07-sp1'
   az appservice plan create --resource-group $RESOURCEGROUPNAME --name $SERVICEPLANNAME --sku B3
   ```

    > **注**:前のコマンドを実行したときに "サブスクリプションは名前空間 'Microsoft.Web' を使用するように登録されていません" のようなエラーが発生した場合は、次の `az provider register --namespace Microsoft.Web` を実行し、エラーを生成したコマンドをもう一度実行します。

1. 一意の名前を指定して Web アプリを作成します。

   ```bash
   WEBAPPNAME=eshoponWebYAML$RANDOM$RANDOM
   az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME
   ```

   > **注**:Web アプリの名前を記録します。 このラボで後ほど必要になります。

1. Azure Cloud Shell は閉じますが、Azure Portal はブラウザーで開いたままにします。

### 演習 1:Azure DevOps で YAML を使用して CI/CD パイプラインをコードとして構成する

この演習では、Azure DevOps で YAML を使用して CI/CD パイプラインをコードとして構成します。

#### タスク 1: YAML ビルド定義を追加する

このタスクでは、既存のプロジェクトに YAML ビルドの定義を追加します。

1. **[パイプライン]** ハブで **[パイプライン]** に戻ります。
1. **[最初のパイプラインを作成]** ウィンドウで、**[パイプラインの作成]** をクリックします。

   > **注**: ウィザードを使い、プロジェクトに基づいて新しい YAML パイプラインの定義を作成します。

1. **[コードはどこにありますか?]** ペインで **[Azure Repos Git (YAML)]** オプションをクリックします。
1. **[リポジトリの選択]** ペインで **eShopOnWeb_MultiStageYAML** をクリックします。
1. **[パイプラインを構成する]** ペインで、 **[既存の Azure Pipelines YAML ファイル]** を選びます。
1. **[既存の YAML ファイルを選択する]** ペインで、次のパラメーターを指定します。
   - [ブランチ]: **main**
   - パス: **.ado/eshoponweb-ci.yml**
1. **[続行]** をクリックして、これらの設定を保存します。
1. **[パイプライン YAML をレビューする]** 画面で **[実行]** をクリックして、ビルド パイプライン プロセスを開始します。
1. ビルド パイプラインが正常に完了するまで待ちます。 ソース コード自体に関する警告は無視します。これらは、このラボ演習には関係ありません。

   > **注**:警告やエラーなど、YAML ファイルの各タスクをレビューできます。

#### タスク 2: YAML の定義に継続的デリバリーを追加する

このタスクでは、以前のタスクで作成したパイプラインの YAML ベースの定義に継続的デリバリーを追加します。

> **注**:ビルドとテストのプロセスに成功すると、YAML 定義にデリバリーを追加できます。

1. [パイプライン実行] ペインで、右上隅にある省略記号をクリックし、ドロップダウン メニューで **[パイプラインの編集]** をクリックします。
1. **eShopOnWeb_MultiStageYAML/.ado/eshoponweb-ci.yml** ファイルの内容が表示されているペインで、ファイルの末尾 (56 行目) に移動し、**Enter/Return** キーを押して新しい空の行を追加します。
1. **57** 行目に移動し、YAML パイプラインの**リリース** ステージを定義する次の内容を追加します。

   > **注**:パイプラインの進捗状況をよりよく整理して追跡するために必要なステージを定義できます。

   ```yaml
   - stage: Deploy
     displayName: Deploy to an Azure Web App
     jobs:
       - job: Deploy
         pool:
           vmImage: "windows-latest"
         steps:
   ```

1. YAML 定義の最後で新しいライン上にカーソルを配置します。

   > **注**:これは、新しいタスクが追加される場所になります。

1. [コード] ペインの右側のタスクの一覧で、**[Azure App Service のデプロイ]** タスクを検索して選択します。
1. **[Azure App Service のデプロイ]** ペインで次の設定を指定して、**[追加]** をクリックします。

   - **[Azure サブスクリプション]** ドロップダウン リストで、このラボの前半に Azure リソースをデプロイした Azure サブスクリプションを選択し、**[承認]** をクリックします。プロンプトが表示されたら、Azure リソース デプロイで使用したものと同じユーザー アカウントを使用して認証します。
   - **[App Service の名前]** ドロップダウン リストで、このラボの前半にデプロイした Web アプリの名前を選択します。
   - **[パッケージまたはフォルダー]** テキスト ボックスで、既定値を `$(Build.ArtifactStagingDirectory)/**/Web.zip` に**更新**します。
   - **[アプリケーションと構成の設定]** セクションを開き、**[アプリケーション設定]** テキストボックスに `-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development` を追加します

1. **[追加]** ボタンをクリックし、[アシスタント] ペインで設定を確認します。

   > **注**:これにより、デプロイ タスクが YAML パイプラインの定義に自動的に追加されます。

1. エディターには次のようなコード スニペットが追加されていて、azureSubscription と WebappName パラメーターには実際の名前が反映されている必要があります。

   ```yaml
   - task: AzureRmWebAppDeployment@4
     inputs:
       ConnectionType: "AzureRM"
       azureSubscription: "AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)"
       appType: "webApp"
       WebAppName: "eshoponWebYAML369825031"
       packageForLinux: "$(Build.ArtifactStagingDirectory)/**/Web.zip"
       AppSettings: "-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development"
   ```

1. タスクが **steps** タスクの子として一覧に表示されていることを確認します。 そうでない場合は、追加したタスクのすべての行を選び、**Tab** キーを 2 回押してスペース 4 つ分だけインデントして、**steps** タスクの子として一覧に表示されるようにします。

   > **注**:**packageForLinux** パラメーターは、このラボの状況では不適切ですが、Windows または Linux では有効です。

   > **注**:既定により、2 つのステージは独立して実行されます。 このため、最初のステージのビルド出力は、さらに変更を加えなければ 2 番目のステージで利用できない可能性があります。 これらの変更を実装するため、デプロイ ステージの先頭でデプロイ成果物をダウンロードする新しいタスクを追加します。

1. **deploy** ステージの **steps** ノードの下の最初の行にカーソルを置き、Enter/Return キーを押して新しい空の行 (64 行目) を追加します。
1. **[タスク]** ペインで **[ビルド成果物のダウンロード]** タスクを検索して選択します。
1. このタスクに対する次のパラメーターを指定します。
   - ダウンロードする成果物の生成元: **現在のビルド**
   - ダウンロードの種類: **特定の成果物**
   - 成果物名: **一覧から [Web サイト] を選択します** (一覧に自動的に表示されない場合は、**「`Website`」と直接入力します**)
   - ダウンロード先ディレクトリ: **$(Build.ArtifactStagingDirectory)**
1. **[追加]** をクリックします。
1. 追加されたコードのスニペットは、次のようになります。

   ```yaml
   - task: DownloadBuildArtifacts@1
     inputs:
       buildType: "current"
       downloadType: "single"
       artifactName: "Website"
       downloadPath: "$(Build.ArtifactStagingDirectory)"
   ```

1. YAML のインデントがオフになっている場合は、エディターで追加されたタスクを選んだまま、**Tab** キーを 2 回押してスペース 4 つ分インデントします。

   > **注**:ここでも、前後に空白のラインを追加して読みやすくすることをお勧めします。

1. **[検証と保存]** をクリックし、**[検証と保存]** ペインでもう一度 **[保存]** をクリックして、変更をメイン ブランチに直接コミットします。

   > **注**: 元の CI-YAML は新しいビルドを自動的にトリガーするように構成されていなかったため、手動で開始する必要があります。

1. Azure DevOps の左側のメニューから **[パイプライン]** に移動し、もう一度 **[パイプライン]** を選びます。
1. **eShopOnWeb_MultiStageYAML** パイプラインを開いて、**[パイプラインの実行]** をクリックします。
1. 表示されるペインで **[実行]** を確認します。
1. **[Build .Net Core Solution] (.Net Core ソリューションをビルドする)** と **[Deploy to Azure Web App] (Azure Web アプリにデプロイする)** という 2 つの異なるステージが表示されることに注意してください。
1. パイプラインが開始され、ビルド ステージが正常に完了するまで待ちます。
1. デプロイ ステージが開始できる状態になると、 **[アクセス許可が必要です]** というプロンプトとオレンジ色のバーが表示されます。

   ```text
   This pipeline needs permission to access a resource before this run can continue to Deploy to an Azure Web App
   ```

1. **[表示]** をクリックします
1. **[レビューを待機しています]** ペインで、 **[許可]** をクリックします。
1. **[アクセスを許可しますか?]** ウィンドウでメッセージを検証し、**[許可]** をクリックして確認します。
1. これにより、デプロイ ステージが開始します。 これが正常に完了するまで待ちます。

   > **注**: YAML パイプライン構文に問題があるためにデプロイが失敗する場合は、以下を参照として使用します。

   ```yaml
   #NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml")
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
         displayName: Test
         inputs:
           command: 'test'
           projects: 'tests/UnitTests/*.csproj'

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

       - task: PublishBuildArtifacts@1
         displayName: Publish Artifacts ADO - Bicep
         inputs:
           PathtoPublish: '$(Build.SourcesDirectory)/infra/webapp.bicep'
           ArtifactName: 'Bicep'
           publishLocation: 'Container'

    - stage: Deploy
      displayName: Deploy to an Azure Web App
      jobs:
      - job: Deploy
        pool:
          vmImage: 'windows-latest'
        steps:
        - task: DownloadBuildArtifacts@1
          inputs:
            buildType: 'current'
            downloadType: 'single'
            artifactName: 'Website'
            downloadPath: '$(Build.ArtifactStagingDirectory)'
        - task: AzureRmWebAppDeployment@4
          inputs:
            ConnectionType: 'AzureRM'
            azureSubscription: 'AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)'
            appType: 'webApp'
            WebAppName: 'eshoponWebYAML369825031'
            packageForLinux: '$(Build.ArtifactStagingDirectory)/**/Web.zip'
            AppSettings: '-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development'
   ```

#### タスク 3: デプロイされたサイトを確認する

1. Azure portal を表示している Web ブラウザー ウィンドウに戻り、Azure Web アプリのプロパティが表示されているブレードに移動します。
1. [Azure Web アプリ] ブレードで **[概要]** をクリックします。[概要] ブレードで **[参照]** をクリックし、新しい Web ブラウザー タブでサイトを開きます。
1. デプロイされたサイトが新しいブラウザー タブに期待どおりに読み込まれ、eShopOnWeb eコマース Web サイトが表示されることを確認します。

### 演習 2: Azure DevOps で YAML を使用してコードとしての CI/CD パイプラインの環境設定を構成する

この演習では、Azure DevOps の YAML ベースのパイプラインに承認を追加します。

#### タスク 1: パイプラインの環境を設定する

コードとしての YAML パイプラインには、Azure DevOps クラシック リリース パイプラインのようなリリースおよび品質ゲートがありません。 ただし、 **[環境]** を使ってコードとしての YAML パイプライン用に同様のものを構成できます。 このタスクでは、このメカニズムを使ってビルド ステージ用に承認を構成します。

1. Azure DevOps プロジェクト **eShopOnWeb_MultiStageYAML** から、**[パイプライン]** に移動します。
1. 左側の [パイプライン] メニューで、 **[環境]** を選びます。
1. **[環境の作成]** をクリックします。
1. **[新しい環境]** ペインで、環境に **`approvals`** という名前を追加します。
1. **[リソース]** で **[なし]** を選びます。
1. **[作成]** ボタンを選んで設定を確定します。
1. 環境を作成したら、新しい **approvals** 環境から **[承認とチェック]** タブを選択します。
1. **[最初のチェックを追加]** で、 **[承認]** を選びます。
1. 自分の Azure DevOps ユーザー アカウント名を **[承認者]** フィールドに追加します。

   > **注:** 実際のシナリオでは、このプロジェクトで作業する DevOps チームの名前にします。

1. **[作成]** ボタンを選んで、定義された承認設定を確定します。
1. 最後に、デプロイ ステージの YAML パイプライン コードに必要な "environment: approvals" 設定を追加する必要があります。 これを行うには、 **[リポジトリ]** に移動し、 **.ado** フォルダーを参照して、コードとしてのパイプラインのファイル **eshoponweb-ci.yml** を選びます。
1. [コンテンツ] ビューで、 **[編集]** ボタンをクリックして編集モードに切り替えます。
1. **デプロイ ジョブ**の開始に移動します (60 行目の -job: Deploy)
1. すぐ下に新しい空の行を追加して、次のスニペットを追加します。

   ```yaml
   environment: approvals
   ```

   結果のコード スニペットは次のようになります。

   ```yaml
   jobs:
     - job: Deploy
       environment: approvals
       pool:
         vmImage: "windows-latest"
   ```

1. 環境はデプロイ ステージに固有の設定なので、"jobs" では使用できません。 したがって、現在のジョブ定義にいくつか追加の変更を行う必要があります。
1. **60** 行目で、名前を "- job: Deploy" から **- deployment: Deploy** に変更します
1. 次に、**63** 行目 (vmImage: windows-latest) で、新しい空の行を追加します。
1. 次の Yaml スニペットを貼り付けます。

   ```yaml
   strategy:
     runOnce:
       deploy:
   ```

1. 残りのスニペット (**67** 行目から最後まで) を選び、**Tab** キーを使用して YAML のインデントを修正します。

   結果の YAML スニペットは、**デプロイ ステージ**を反映した次のようなものになります。

   ```yaml
   - stage: Deploy
     displayName: Deploy to an Azure Web App
     jobs:
       - deployment: Deploy
         environment: approvals
         pool:
           vmImage: "windows-latest"
         strategy:
           runOnce:
             deploy:
               steps:
                 - task: DownloadBuildArtifacts@1
                   inputs:
                     buildType: "current"
                     downloadType: "single"
                     artifactName: "Website"
                     downloadPath: "$(Build.ArtifactStagingDirectory)"
                 - task: AzureRmWebAppDeployment@4
                   inputs:
                     ConnectionType: "AzureRM"
                     azureSubscription: "AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)"
                     appType: "webApp"
                     WebAppName: "eshoponWebYAML369825031"
                     packageForLinux: "$(Build.ArtifactStagingDirectory)/**/Web.zip"
                     AppSettings: "-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development"
   ```

1. **[コミット]** をクリックし、表示される [コミット] ペインで **[コミット]** をもう一度クリックして、コードの YAML ファイルに対する変更を確定します。
1. 左側の Azure DevOps プロジェクト メニューに移動し、 **[パイプライン]** を選び、 **[パイプライン]** を選んで、前に使った **EshopOnWeb_MultiStageYAML** パイプラインを見つけます。
1. そのパイプラインを開きます。
1. **[パイプラインの実行]** をクリックして、新しいパイプラインの実行をトリガーします。 **[実行]** をクリックして確認します。
1. 前と同じように、ビルド ステージが想定どおりに開始されます。 それが正常に完了するまで待ちます。
1. 次に、デプロイ ステージに対して _environment:approvals_ を構成してあるため、開始する前に承認の確認を求められます。
1. これは [パイプライン] ビューから確認でき、**[待機中 (1 件のチェックが処理中)]** と表示されます。 **この実行を続行して Azure Web アプリへのデプロイに進む前に 1 件の承認のレビューが必要である**ことを示す通知メッセージも表示されます。
1. このメッセージの隣の **[レビュー]** ボタンをクリックします。
1. 表示されるペインの **[レビューを待機しています]** で、**[承認する]** ボタンをクリックします。
1. これにより、デプロイ ステージは開始して、Azure Web アプリのソース コードを正常にデプロイできます。

   > **注:** この例では承認のみを使っていますが、Azure Monitor や REST API などの他のチェックも同様の方法で使用できます

   > [!IMPORTANT]
   > 不要な料金が発生しないように、Azure portal で作成されたリソースを必ず削除してください。

## 確認

このラボでは、Azure DevOps で YAML を使用して CI/CD パイプラインをコードとして構成しました。
