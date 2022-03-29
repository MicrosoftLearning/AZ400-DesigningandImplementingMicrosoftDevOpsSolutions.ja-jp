---
lab:
  title: ラボ 11:YAML を使用したコードとしてのパイプラインの構築
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
ms.openlocfilehash: 6f6c4d98338022a305fb3fd05f0d1efc7a4b9c00
ms.sourcegitcommit: f72fcf5ee578f465b3495f3cf789b06c530e88a4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2022
ms.locfileid: "139262577"
---
# <a name="lab-11-configuring-pipelines-as-code-with-yaml"></a>ラボ 11:YAML を使用したコードとしてのパイプラインの構築
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-overview"></a>ラボの概要

多くのチームは YAML を使用してビルドを定義し、パイプラインをリリースすることを好みます。 これにより、ビジュアル デザイナーを使用した場合と同じパイプラインの機能にアクセスできます。ただし、マークアップ ファイルは他のソース ファイルと同様に管理できます。 YAML ビルドの定義は、該当するファイルをリポジトリのルートに加えるだけでプロジェクトに追加できます。 Azure DevOps は、人気の高いプロジェクトの種類の既定テンプレートのほか、YAML デザイナーを提供して、ビルドおよびリリース タスクの定義プロセスを簡素化します。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- Azure DevOps で YAML を使用して CI/CD パイプラインをコードとして構成する

## <a name="lab-duration"></a>ラボの所要時間

-   予想所要時間: **60 分**

## <a name="instructions"></a>Instructions

### <a name="before-you-start"></a>開始する前に

#### <a name="sign-in-to-the-lab-virtual-machine"></a>ラボ仮想マシンにサインインする

次の認証情報を使用して、Windows 10 仮想マシンにサインインしていることを確認します。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### <a name="review-applications-required-for-this-lab"></a>このラボで必要なアプリケーションのレビュー

このラボで使用するアプリケーションを特定:
    
-   Microsoft Edge

#### <a name="set-up-an-azure-devops-organization"></a>Azure DevOps 組織を設定します。 

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)で利用できる手順に従って作成してください。

#### <a name="prepare-an-azure-subscription"></a>Azure サブスクリプションの準備

-   既存の Azure サブスクリプションを識別するか、新しいものを作成します。
-   Azure サブスクリプションでは所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントではグローバル管理者のロールで Microsoft アカウントまたは Azure AD アカウントがあることを確認します。 詳細については、[「Azure portal を使用して Azure ロールの割り当てを一覧表示する」](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal)および[「Azure Active Directory で管理者ロールを表示して割当てる」](https://docs.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal#view-my-roles)を参照してください。

### <a name="exercise-0-configure-the-lab-prerequisites"></a>演習 0:ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、Azure DevOps Demo Generator テンプレートと Azure リソースに基づいて事前に構成された Parts Unlimited チーム プロジェクトで構成され、Azure Web アプリと Azure SQL データベースが含まれます。 

#### <a name="task-1-configure-the-team-project"></a>タスク 1:チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用して、**PartsUnlimited-YAML** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。 このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**:このサイトの詳細については、「[Azure DevOps Services Demo Generator とは](https://docs.microsoft.com/en-us/azure/devops/demo-gen)」を参照してください。

1.  「**サインイン**」をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。
1.  必要な場合は、「**Azure DevOps Demo Generator**」ページで「**承諾する**」をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。
1.  「**新しいプロジェクトの作成**」ページで「**新しいプロジェクト名**」テキストボックスに「**YAML を使用したパイプラインのコードとしての構成**」と入力します。「**組織の選択**」ドロップダウン リストで Azure DevOps 組織を選択し、「**テンプレートの選択**」をクリックします。
1.  テンプレートのリストのツールバーで 「**全般**」をクリックし、「**PartsUnlimited-YAML**」テンプレートを選択して 「**テンプレートの選択**」をクリックします。
1.  再び「**新しいプロジェクトの作成**」ページで「**プロジェクトの作成**」をクリックします。

    > **注**:プロセスが完了するまでお待ちください。 これには 2 分ほどかかります。 プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

#### <a name="task-2-create-azure-resources"></a>タスク 2: Azure リソースを作成する

このタスクでは、Azure portal を使用して Azure Web アプリと Azure SQL データベースを作成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[**Azure portal**](https://portal.azure.com) に移動します。このラボで使用する Azure サブスクリプションで、所有者のロールがあり、このサブスクリプションに関連付けられている Azure AD テナントでグローバル管理者のロールがあるユーザー アカウントを使ってサインインします。
1.  Azure portal のツールバーで、検索テキスト ボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。
1.  **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、**[Bash]** を選択します。

    >**注**:**Cloud Shell** を初めて起動し、「**ストレージがマウントされていません**」というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、 **[ストレージの作成]** を選択します。 

1.  **Bash** プロンプトの **[Cloud Shell]** ペインで、次のコマンドを実行してリソース グループを作成します (`<region>` プレースホルダーを 'eastus' などのご自分の場所に最も近い Azure リージョンの名前に置き換えます)。
    
    ```bash
    LOCATION='<region>'
    ```
    ```bash
    RESOURCEGROUPNAME='az400m11l01-RG'
    az group create --name $RESOURCEGROUPNAME --location $LOCATION
    ```

1.  次のコマンドを実行して Windows App Service プランを作成するには:

    ```bash
    SERVICEPLANNAME='az400l11a-sp1'
    az appservice plan create --resource-group $RESOURCEGROUPNAME --name $SERVICEPLANNAME --sku B3
    ```
    
    > **注**:`ModuleNotFoundError: No module named 'vsts_cd_manager'` で始まるエラー メッセージで `az appservice plan create` コマンドが失敗した場合は、次のコマンドを実行してから、失敗したコマンドを再び実行します。

    ```bash
    az extension remove -n appservice-kube
    az extension add --yes --source "https://aka.ms/appsvc/appservice_kube-latest-py2.py3-none-any.whl"
    ```

1.  一意の名前を指定して Web アプリを作成します。

    ```bash
    WEBAPPNAME=partsunlimited$RANDOM$RANDOM
    az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME
    ```

    > **注**:Web アプリの名前を記録します。 このラボで後ほど必要になります。

1.  次に、Azure SQL Server を作成します。

    ```bash
    USERNAME="Student"
    SQLSERVERPASSWORD="Pa55w.rd1234"
    SERVERNAME="partsunlimitedserver$RANDOM"

    az sql server create --name $SERVERNAME --resource-group $RESOURCEGROUPNAME \
    --location $LOCATION --admin-user $USERNAME --admin-password $SQLSERVERPASSWORD
    ```

1.  Web アプリは、SQL サーバーにアクセスできる必要があります。そのため、SQL Server ファイアウォール規則で Azure リソースへのアクセスを許可する必要があります。

    ```bash
    STARTIP="0.0.0.0"
    ENDIP="0.0.0.0"
    az sql server firewall-rule create --server $SERVERNAME --resource-group $RESOURCEGROUPNAME \
    --name AllowAzureResources --start-ip-address $STARTIP --end-ip-address $ENDIP
    ```

1.  次に、そのサーバー内にデータベースを作成します。

    ```bash
    az sql db create --server $SERVERNAME --resource-group $RESOURCEGROUPNAME --name PartsUnlimited \
    --service-objective S0
    ```

1.  作成した Web アプリの構成にはデータベース接続文字列が必要です。そのため、次のコマンドを実行してそれを準備し、Web アプリのアプリ設定に追加します。

    ```bash
    CONNSTRING=$(az sql db show-connection-string --name PartsUnlimited --server $SERVERNAME \
    --client ado.net --output tsv)

    CONNSTRING=${CONNSTRING//<username>/$USERNAME}
    CONNSTRING=${CONNSTRING//<password>/$SQLSERVERPASSWORD}

    az webapp config connection-string set --name $WEBAPPNAME --resource-group $RESOURCEGROUPNAME \
    -t SQLAzure --settings "DefaultConnectionString=$CONNSTRING" 
    ```

### <a name="exercise-1-configure-cicd-pipelines-as-code-with-yaml-in-azure-devops"></a>演習 1:Azure DevOps で YAML を使用して CI/CD パイプラインをコードとして構成する

この演習では、Azure DevOps で YAML を使用して CI/CD パイプラインをコードとして構成します。

#### <a name="task-1-delete-the-existing-pipeline"></a>タスク 1:既存のパイプラインを削除する

このタスクでは、既存のパイプラインを削除します。

1.  ラボのコンピューターで、Azure DevOps ポータルに「**YAML を使用してパイプラインをコードとして構成する**」プロジェクトが表示されているブラウザー ウィンドウに切り替え、垂直のナビゲーション ペインで「**パイプライン**」を選択します。

    > **注**:YAML パイプラインを構成する前に、既存のビルド パイプラインを無効にします。

1.  「**パイプライン**」ペインで「**PartsUnlimited**」エントリを選択します。 
1.  「**PartsUnlimited**」ブレードの右上コーナーで、垂直の省略記号をクリックし、ドロップダウン メニューで 「**削除**」を選択します。
1.  **PartsUnlimited** と記述し、 **「削除**」をクリックします。
1.  垂直ナビゲーションペインで、「**リポジトリ > ファイル**」を選択します。 **マスター** ブランチ (「**ファイル**」ウィンドウの上部にあるドロップダウン) にいることを確認し、**azure-pipelines.yml** ファイルで、縦の省略記号をクリックして、ドロップダウン メニューで 「**削除**」を選択します。 「**コミット**」をクリックして、マスター ブランチで変更をコミットします (既定のオプションのままにします)。

#### <a name="task-2-add-a-yaml-build-definition"></a>タスク 2:YAML ビルド定義を追加する

このタスクでは、既存のプロジェクトに YAML ビルドの定義を追加します。

1.  「**パイプライン**」ハブで「**パイプライン**」に戻ります。 
1.  「**最初のパイプラインの作成**」ウィンドウで、「**パイプラインの作成**」をクリックします。 

    > **注**:ウィザードを使い、プロジェクトに基づいて YAML の定義を自動的に作成します。

1.  「**コードはどこにありますか?** 」ペインで「**Azure Repos Git (YAML)** 」オプションをクリックします。
1.  「**リポジトリの選択**」ペインで「**PartsUnlimited**」をクリックします。
1.  **[パイプラインの構成]** ペインで **[ASP.<nolink>NET]** をクリックし、このテンプレートをパイプラインの起点として利用します。 これにより、「**パイプライン YAML のレビュー**」ペインが開きます。

    > **注**:パイプラインの定義は、リポジトリのルートで「**azure-pipelines.yml**」という名前のファイルとして保存されます。 このファイルには、典型的な ASP<nolink>.NET ソリューションを構築してテストするために必要なステップが含まれます。 また、必要に応じてビルドをカスタマイズすることもできます。 このシナリオでは、**プール** を更新し、Windows 2019 を実行している VM を使用できるようにします。

1.  `trigger` が **master** である必要があります。

    > **注**:リポジトリに **マスター** ブランチまたは **メイン** ブランチがある場合は、リポジトリで確認します。組織は、新しいリポジトリに既定のブランチ名を選択できます。[既定のブランチを変更する](https://docs.microsoft.com/en-us/azure/devops/repos/git/change-default-branch?view=azure-devops#choosing-a-name)。 

1.  **[パイプライン YAML のレビュー]** ペインの **10** 行目で、`vmImage: 'windows-latest'` を `vmImage: 'windows-2019'` に置き換えます。
1.  **VSTest@2** タスクを削除します。
    ```yaml
    - task: VSTest@2
      inputs:
        platform: '$(buildPlatform)'
        configuration: '$(buildConfiguration)'
    ```
1.  「**パイプライン YAML のレビュー**」ペインで「**保存して実行**」をクリックします。
1.  「**保存して実行**」ペインで既定の設定を承諾し、「**保存して実行**」をクリックします。
1.  パイプライン実行ペインの「**ジョブ**」セクションで「**ジョブ**」をクリックし、進捗状況を監視して、完了したことを確認します。 

    > **注**:警告やエラーなど、YAML ファイルの各タスクをレビューできます。

#### <a name="task-3-add-continuous-delivery-to-the-yaml-definition"></a>タスク 3:YAML 定義に継続的デリバリーを追加する

このタスクでは、以前のタスクで作成したパイプラインの YAML ベースの定義に継続的デリバリーを追加します。

> **注**:ビルドとテストのプロセスに成功すると、YAML 定義にデリバリーを追加できます。 

1.  パイプライン実行ペインで、右上コーナーにある省略記号をクリックし、ドロップダウン メニューで「**パイプラインの編集**」をクリックします。
1.  **azure-pipelines.yaml** ファイルのコンテンツが表示されているペインで **8** 行目の `trigger` セクションの後に以下のコンテンツを追加し、YAML パイプラインの **ビルド** ステージを定義します。 

    > **注**:パイプラインの進捗状況をよりよく整理して追跡するために必要なステージを定義できます。

    ```yaml
    stages:
    - stage: Build
      jobs:
      - job: Build
    ```

1.  YAML ファイルの残りのコンテンツを選択し、**Tab** キーを 2 回押してスペース 4 つ分をインデントします (```job: Build``` と同じインデントに配置される必要があります)。 

    > **注**:これにより、`pool` セクションで始まるものはすべて `job: Build` の一部になります。 

1.  ファイルの最下部で、以下の項性を追加して 2 番目のステージを定義します。

    ```yaml
    - stage: Deploy
      jobs:
      - job: Deploy
        pool:
          vmImage: 'windows-2019'
        steps:
    ```

1.  YAML 定義の最後で新しいライン上にカーソルを配置します。 

    > **注**:これは、新しいタスクが追加される場所になります。

1.  コード ペインの右側のタスク一覧で、「**Azure App Service のデプロイ**」タスクを検索して選択します。
1.  「**Azure App Service のデプロイ**」ペインで以下の設定を指定し、「**追加**」をクリックします。

    - 「**Azure サブスクリプション**」ドロップダウン リストで、ラボで以前に Azure リソースをデプロイした Azure サブスクリプションを選択し、「**承認**」をクリックします。指示されたら、Azure リソース デプロイの際に使用したものと同じユーザー アカウントを使用して認証します。
    - 「**App Service の名前**」ドロップダウン リストで、ラボで以前にデプロイした Web アプリの名前を選択します。 
    - **[パッケージまたはフォルダー]** テキスト ボックスに「`$(System.ArtifactsDirectory)/drop/*.zip`」と入力します。 

    > **注**:これにより、デプロイ タスクが YAML パイプラインの定義に自動的に追加されます。

1.  エディターで追加されたタスクを選択したままの状態にして、**Tab** キーを 2 回押し、スペース 4 つ分をインデントします。これにより、**steps** タスクの子としてリストされます。

    > **注**:**packageForLinux** パラメーターは、このラボの状況では不適切ですが、Windows または Linux では有効です。 

    > **注**:既定により、2 つのステージは独立して実行されます。 このため、最初のステージのビルド出力は、さらに変更を加えなければ 2 番目のステージで利用できない可能性があります。 このような変更を実装するには、ひとつのタスクを使用してビルド出力をビルド ステージの最後に公開し、別のタスクを使ってデプロイ ステージの最初にこれをダウンロードします。 

1.  ビルド ステージの最後に空白のライン上にカーソルを配置して、別のタスクを追加します。 (`task: VSBuild@1` のすぐ下)
1.  「**タスク**」ペインで「**ビルド成果物の公開**」タスクを検索して選択します。 
1.  「**ビルド成果物の公開**」ペインで既定の設定を承諾し、「**追加**」をクリックします。 

    > **注**:これにより、「**drop**」というエイリアスでダウンロードできる場所にビルド成果物が公開されます。

1.  追加したタスクをエディターで選択したまま、**Tab** キーを 2 回押して、4 つのスペースをインデントします (または、タスクが上記のようにインデントされるまで **Tab** キーを押します)。

    > **注**:また、前後に空白のラインを追加して読みやすくすることをお勧めします。

1.  **デプロイ** ステージの **steps** ノードで最初のラインにカーソルを配置します。
1.  「**タスク**」ペインで「**ビルド成果物のダウンロード**」タスクを検索して選択します。
1.  **[追加]** をクリックします。
1.  エディターで追加されたタスクを選択したままの状態にして、**Tab** キーを 2 回押し、スペース 4 つ分をインデントします。 

    > **注**:ここでも、前後に空白のラインを追加して読みやすくすることをお勧めします。

1.  ダウンロード タスクにプロパティを追加し、`drop` の `artifactName` を指定します (必ずスペースを一致させます):

    ```
    artifactName: 'drop'
    ```

1.  「**保存**」をクリックし、「**保存**」ペインで再び「**保存**」をクリックして変更を直接、マスター ブランチにコミットします。

    > **注**:これにより、新しいビルドが自動的にトリガーされます。

1.  パイプラインは次の例のようになります (**最後のタスクで独自のサブスクリプションと Web アプリを参照します**)。
    
     > **注**:これを機能させるには、正しくインデントする必要があります。コピーして貼り付けると、変更される場合があります。

    ```
    trigger:
    - master

    stages:
    - stage: Build
      jobs:
      - job: Build
        pool:
            vmImage: 'windows-2019'

        variables:
            solution: '**/*.sln'
            buildPlatform: 'Any CPU'
            buildConfiguration: 'Release'

        steps:
        - task: NuGetToolInstaller@1

        - task: NuGetCommand@2
          inputs:
            restoreSolution: '$(solution)'

        - task: VSBuild@1
          inputs:
            solution: '$(solution)'
            msbuildArgs: '/p:DeployOnBuild=true /p:WebPublishMethod=Package /p:PackageAsSingleFile=true /p:SkipInvalidConfigurations=true /p:PackageLocation="$(build.artifactStagingDirectory)"'
            platform: '$(buildPlatform)'
            configuration: '$(buildConfiguration)'

        - task: PublishBuildArtifacts@1
          inputs:
            PathtoPublish: '$(Build.ArtifactStagingDirectory)'
            ArtifactName: 'drop'
            publishLocation: 'Container'

    - stage: Deploy
      jobs:
      - job: Deploy
        pool:
            vmImage: 'windows-2019'
        steps:
        - task: DownloadBuildArtifacts@0
          inputs:
            buildType: 'current'
            downloadType: 'single'
            downloadPath: '$(System.ArtifactsDirectory)'
            artifactName: 'drop'
        - task: AzureRmWebAppDeployment@4
          inputs:
            ConnectionType: 'AzureRM'
            azureSubscription: 'YOUR-AZURE-SUBSCRIPTION'
            appType: 'webApp'
            WebAppName: 'YOUR-WEBAPP-NAME'
            packageForLinux: '$(System.ArtifactsDirectory)/drop/*.zip'
    ```

1.  Azure DevOps ポータルが表示されている Web ブラウザー ウィンドウの垂直ナビゲーション ペインで、「**パイプライン**」を選択します。
1.  「**パイプライン**」ペインで、新しく構成されたパイプラインを示すエントリをクリックします。
1. 最新の実行 (自動的に開始) をクリックします。
1.  「**概要**」ペインで、パイプライン実行の進捗状況を監視します。
1.  「*このパイプラインは、この実行のデプロイを継続する前にリソースのアクセス許可が必要です*」というメッセージが表示されたら、「**表示**」をクリックします。「**レビューの待機中**」ダイアログ ボックスで「**許可**」をクリックします。「**アクセス許可?** 」ペインで再び「**許可**」をクリックします。
1.  「**概要**」ペインの最下部で「**デプロイ**」ステージをクリックすると、デプロイの詳細が表示されます。

    > **注**:タスクが完了すると、アプリが Azure Web アプリにデプロイされます。

#### <a name="task-4-review-the-deployed-site"></a>タスク 4:デプロイされたサイトをレビューする

1.  Azure portal を表示している Web ブラウザー ウィンドウに戻り、Azure Web アプリのプロパティが表示されているブレードに移動します。
1.  Azure Web アプリ ブレードで「**概要**」をクリックします。概要ブレードで「**参照**」をクリックし、新しい Web ブラウザー タブでサイトを開きます。
1.  デプロイされたサイトが新しいブラウザー タブで期待されているように読み込まれていることを確認します。

### <a name="exercise-2-remove-the-azure-lab-resources"></a>演習 2:Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

>**注**:新規に作成し、使用しなくなったすべての Azure リソースを削除することを忘れないでください。 使用していないリソースを削除することで、予期しない料金が発生しなくなります。

#### <a name="task-1-remove-the-azure-lab-resources"></a>タスク 1:Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'az400m11l01-RG')].name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体を通して作成したすべてのリソース グループを削除します。

    ```sh
    az group list --query "[?starts_with(name,'az400m11l01-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注**:コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## <a name="review"></a>確認

このラボでは、Azure DevOps で YAML を使用して CI/CD パイプラインをコードとして構成しました。
