---
lab:
  title: ラボ 18:Docker コンテナーを Azure App Service Web アプリにデプロイする
  module: 'Module 08: Create and manage containers using Docker and Kubernetes'
ms.openlocfilehash: 6204f1a93e4b1d0da770d4de3bcfe6a367b5d5ef
ms.sourcegitcommit: d54d32ce867046c2c038c1a16d757e9e66b26fc5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/14/2022
ms.locfileid: "139840793"
---
# <a name="lab-18-deploying-docker-containers-to-azure-app-service-web-apps"></a>ラボ 18:Docker コンテナーを Azure App Service Web アプリにデプロイする
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-overview"></a>ラボの概要

このラボでは、Azure DevOps CI/CD パイプラインを使用してカスタム Docker イメージを構築し、それを Azure Container Registry にプッシュして、コンテナーとして Azure App Service にデプロイする方法を学習します。 

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- Microsoft がホストする Linux エージェントを使用して、カスタム Docker イメージを構築する
- Azure Container Registry へのイメージのプッシュ
- Azure DevOps を使用して Docker イメージをコンテナーとして、Azure App Service にデプロイする

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

#### <a name="prepare-an-azure-subscription"></a>Azure サブスクリプションの準備

-   既存の Azure サブスクリプションを識別するか、新しいものを作成します。
-   Azure サブスクリプションで共同作成者または所有者のロールを持つ Microsoft アカウントまたは Azure AD アカウントを持っていることを確認します。 詳細については、[「Azure portal を使用して Azure ロールの割り当てを一覧表示する」](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal)および[「Azure Active Directory で管理者ロールを表示して割当てる」](https://docs.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal#view-my-roles)を参照してください。

#### <a name="set-up-an-azure-devops-organization"></a>Azure DevOps 組織を設定する

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)で利用できる手順に従って作成してください。

### <a name="exercise-1-configure-the-lab-prerequisites"></a>演習 1:ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、Azure DevOps Demo Generator テンプレートと Azure リソース (Azure App Service Web アプリ、Azure Container Registry インスタンス、Azure SQL データベースなど) に基づくチーム プロジェクトで構成されています。 

#### <a name="task-1-configure-the-team-project"></a>タスク 1:チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用して、Docker テンプレートに基づいて新しいプロジェクトを生成します。

> **注**:Docker テンプレートベースのプロジェクトは、コンテナー化された ASP.NET Core アプリをビルドして Azure App Service にデプロイします

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。 このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**:サイトの詳細については、[https://docs.microsoft.com/en-us/azure/devops/demo-gen](https://docs.microsoft.com/en-us/azure/devops/demo-gen) を参照してください。

1.  「**サインイン**」をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。
1.  必要な場合は、「**Azure DevOps Demo Generator**」ページで「**承諾する**」をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。
1.  「**新しいプロジェクトの作成**」ページの「**新しいプロジェクト名**」テキストボックスに「**Docker コンテナーを Azure App Service Web アプリにデプロイする**」と入力し、「**組織の選択**」ドロップダウン リストで、Azure DevOps 組織を選択して、「**テンプレートの選択**」をクリックします。
1.  テンプレートのリストのツールバーで、「**DevOps ラボ**」をクリックし、「**DevOps ラボ**」ヘッダーを選択し、「**Docker**」テンプレートをクリックして、「**テンプレートの選択**」をクリックします。
1.  再び「**新しいプロジェクトの作成**」ページで、欠落している拡張機能をインストールするよう指示されたら「**Docker Integration**」ラベルの下にあるチェックボックスを選択し、「**プロジェクトの作成**」をクリックします。

    > **注**:プロセスが完了するまでお待ちください。 これには 2 分ほどかかります。 プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

#### <a name="task-2-create-azure-resources"></a>タスク 2: Azure リソースを作成する

このタスクでは、Azure Cloud Shell を使用して、このラボで必要な Azure リソースを作成します。

- Azure Container Registry
- Azure Web App for Containers
- Azure SQL データベース

1.  ラボのコンピューターで Web ブラウザーを起動し、[**Azure portal**](https://portal.azure.com) に移動します。このラボで使用する Azure サブスクリプションで、所有者のロールがあり、このサブスクリプションに関連付けられている Azure AD テナントでグローバル管理者のロールがあるユーザー アカウントを使ってサインインします。
1.  Azure portal を表示している Web ブラウザーのツールバーで、検索テキスト ボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。 
1.  **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、**[Bash]** を選択します。 

    >**注**:**Cloud Shell** を初めて起動し、「**ストレージがマウントされていません**」というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、 **[ストレージの作成]** を選択します。 

1.  Cloud Shell ペインの **Bash** セッションから、以下を実行して、このラボでリソースをデプロイする Azure リージョンを表す変数を作成します。これらのリソースを含むリソース グループ、これらのリソースの名前 (Azure Container Registry インスタンス、Azure App Service プラン名、Azure Web アプリ名、Azure SQL Database 論理サーバー名、Azure SQL データベース名など)。

1.  Cloud Shell ペインの Bash セッションから、以下を実行して、このラボでデプロイする Azure リソースをホストするリソースグループを作成します (`<Azure_region>` プレースホルダーを、これらのリソースをデプロイする予定の 'eastus' などの Azure リージョンの名前に置き換えます)。

    ```bash
    LOCATION='<Azure_region>'
    ```

    >**注**:`az account list-locations -o table` を実行すると、Azure リージョンの名前を識別することができます。

1.  次のコマンドを実行して、Azure Container Registry インスタンス、Azure App Servic eプラン名、Azure Web アプリ名、Azure SQL データベース論理サーバー名、Azure SQL Database 名などの Azure リソースの名前を表す変数を作成します。

    ```bash
    RG_NAME='az400m1501a-RG'
    ACR_NAME=az400m151acr$RANDOM$RANDOM
    APP_SVC_PLAN='az400m1501a-app-svc-plan'
    WEB_APP_NAME=az400m151web$RANDOM$RANDOM
    SQLDB_SRV_NAME=az400m15sqlsrv$RANDOM$RANDOM
    SQLDB_NAME='az400m15sqldb'
    ```

    >**注**:`az account list-locations -o table` を実行すると、Azure リージョンの名前を識別することができます。

1.  以下を実行して、このラボに必要なすべてのリソースに必要な Azure リソースを作成します。

    ```bash
    az group create --name $RG_NAME --location $LOCATION
    az acr create --name $ACR_NAME --resource-group $RG_NAME --location $LOCATION --sku Standard --admin-enabled true
    az appservice plan create --name 'az400m1501a-app-svc-plan' --location $LOCATION --resource-group $RG_NAME --is-linux
    az webapp create --name $WEB_APP_NAME --resource-group $RG_NAME --plan $APP_SVC_PLAN --deployment-container-image-name elnably/dockerimagetest
    IMAGE_NAME=myhealth.web
    az webapp config container set --name $WEB_APP_NAME --resource-group $RG_NAME --docker-custom-image-name $IMAGE_NAME --docker-registry-server-url $ACR_NAME.azurecr.io/$IMAGE_NAME:latest --docker-registry-server-url https://$ACR_NAME.azurecr.io
    az sql server create --name $SQLDB_SRV_NAME --resource-group $RG_NAME --location $LOCATION --admin-user sqladmin --admin-password Pa55w.rd1234
    az sql db create --name $SQLDB_NAME --resource-group $RG_NAME --server $SQLDB_SRV_NAME --service-objective S0 --no-wait 
    az sql server firewall-rule create --name AllowAllAzure --resource-group $RG_NAME --server $SQLDB_SRV_NAME --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
    ```

    >**注**:プロビジョニング プロセスが完了するまで待ちます。 これには 5 分ほどかかる場合があります。

1.  以下を実行して、新しく作成された Azure Web アプリの接続文字列を構成します ($SQLDB_SRV_NAME および $SQLDB_NAME プレースホルダーをそれぞれ Azure SQL Database 論理サーバーとそのデータベース インスタンスの名前の値に置き換えます)。
   
    ```bash
    CONNECTION_STRING="Data Source=tcp:$SQLDB_SRV_NAME.database.windows.net,1433;Initial Catalog=$SQLDB_NAME;User Id=sqladmin;Password=Pa55w.rd1234;"
    az webapp config connection-string set --name $WEB_APP_NAME --resource-group $RG_NAME --connection-string-type SQLAzure --settings defaultConnection="$CONNECTION_STRING"
    ```

1.  Azure portal を表示している Web ブラウザーで、「Cloud Shell」ペインを閉じ、「**リソース グループ**」ブレードに移動し、「**リソース グループ**」ブレードで **az400m1501a-RG** エントリを選択します。
1.  「**az400m1501a-RG**」リソース グループ ブレードで、リソースのリストを確認します。 

    >**注**:論理 Azure SQL Database サーバーの名前を記録します。 このラボで後ほど必要になります。

1.  **az400m1501a** リソース グループ ブレードのリソース リストで、コンテナー レジストリ インスタンスを表すエントリをクリックします。
1.  「コンテナー レジストリ」ブレードの左側の垂直メニューの「**設定**」セクションで、「**アクセス キー**」をクリックします。
1.  コンテナー レジストリ インスタンスの「**アクセス キー**」ブレードで、**レジストリ名**、**ログイン サーバー**、**管理者ユーザー**、および **パスワード** エントリの値を特定します。

    >**注**:**レジストリ名** と **ログイン サーバー** の値を記録します (レジストリ名と管理者ユーザー名は一致している必要があります)。 これらは、このラボの後半で必要になります。


### <a name="exercise-2-deploy-a-docker-container-to-azure-app-service-web-app-by-using-azure-devops"></a>演習 2:Azure DevOps を使用して、Docker コンテナーを Azure App Service Web アプリにデプロイする

この演習では、Azure DevOps を使用して、Docker コンテナーを Azure App Service Web アプリにデプロイします。

#### <a name="task-1-configure-continuous-integration-ci-and-continuous-delivery-cd"></a>タスク 1:継続的インテグレーション (CI) と継続的デリバリー (CD) を構成する

このタスクでは、前の演習で生成した Azure DevOps プロジェクトを使用して、Docker コンテナーをビルドして Azure App Service Web アプリにデプロイする CI/CD パイプラインを実装します。

1.  ラボのコンピューターで、Azure DevOps ポータルを表示する Web ブラウザー ウィンドウに切り替え、**Docker コンテナーを Azure App Service Web アプリ プロジェクトにデプロイします**。Azure DevOps ポータルの左端にある垂直メニューバーで、「**リポジトリ**」をクリックします。

    >**注**:まず、Docker イメージへの参照を変更します。

1.  **[Docker]** リポジトリ ペインのファイル リストで、**docker-compose.ci.build.yml** を選択します。
1.  **[docker-compose.ci.build.yml]** ペインで、 **[編集]** をクリックし、ターゲット Docker イメージを参照する **5** 行目を `image: az400mp/aspnetcore-build:1.0-2.0` に置き換え、 **[コミット]** を選択し、確認のダイアログが表示されたら、もう一度 **[コミット]** をクリックします。 
1.  **[Docker]** リポジトリ ペインのファイルのリストで、**src/MyHealth.Web** フォルダーに移動し、 **[Dockerfile]** を選択します。
1.  **[Dockerfile]** ペインで、 **[編集]** をクリックし、ベース Docker イメージを参照する **1** 行目を `FROM az400mp/aspnetcore1.0:1.0.4` に置き換え、 **[コミット]** を選択し、確認のダイアログが表示されたら、もう一度 **[コミット]** をクリックします。 
1.  Azure DevOps ポータルを表示する Web ブラウザー ウィンドウに切り替え、**Docker コンテナーを Azure App Service Web アプリ プロジェクトにデプロイします**。Azure DevOps ポータルの左端にある垂直メニューバーで、「**パイプライン**」をクリックします。

    >**注**:次に、ビルド パイプラインを変更します。

1.  「**パイプライン**」ペインで、**MHCDocker.build** パイプラインを示しているエントリをクリックし、「**MHCDocker.Build**」ペインで「**編集**」をクリックします。

    >**注**:ビルド パイプラインは以下のタスクで構成されます

    | タスク | 使用 |
    | ----- | ----- |
    | **サービスの実行** | 必要なパッケージを復元してビルド環境を準備します |
    | **サービスのビルド** | **myhealth.web** イメージを構築します |
    | **サービスのプッシュ** | **$(Build.BuildId)** でタグ付けされ **myhealth.web** イメージをコンテナー レジストリにプッシュします |
    | **成果物の公開** | Azure DevOps の成果物を介してデータベース デプロイ用の dacpac を共有できます |

1.  「**MHCDocker.build** パイプライン」ペインで、**パイプライン** エントリが選択されていることを確認し、「**エージェント仕様**」ドロップダウン　リストで「**ubuntu-18.04**」を選択します。
1.  **MHCDocker.build** パイプライン ペインのパイプラインのタスクのリストで、「**サービスの実行**」タスクをクリックし、右側の「**Docker 作成**」ペインの「**Azure サブスクリプション**」ドロップダウン リストで、使用している Azure サブスクリプションを表すエントリを選択します。このラボで、「**承認**」をクリックして、対応するサービス接続を作成します。 指示されたら、Azure サブスクリプションで所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントでグローバル管理者のロールがあるアカウントを使用してサインインします。

    >**注**:この手順では、Azure サービス接続が作成され、サービス プリンシパル認証 (SPA) を使用してターゲット Azure サブスクリプションへの接続が定義・確立されます。 

1.  **[サービスの実行]** タスクが選択されているパイプラインのタスク一覧で、右側の **[Docker Compose]** ペインにある **[Azure Container Registry]** ドロップダウン リストから、このラボで先ほど作成した ACR インタンスを示しているエントリを選択します (**必要に応じてリストを更新する** か、ログイン サーバーの名前を入力します)。
1.  前の 2 つの手順を繰り返して、**ビルド サービス** タスクと **プッシュ サービス** タスクで **Azure サブスクリプション** と **Azure Container Registry** の設定を構成しますが、今回は、Azure サブスクリプションを選択する代わりに、新しく作成したサービス接続を選択します。
1.  同じ「**MHCDocker.build** パイプライン」ペインのペインの上部で、「**保存してキュー**」ボタンの横にある下向きのキャレットをクリックし、「**保存**」をクリックして変更を保存し、もう一度プロンプトが表示されたら「**保存**」をクリックします。

    >**注**:次に、リリース パイプラインを変更します。

1.  Azure DevOps ポータルが表示されている Web ブラウザー ウィンドウで、Azure DevOps ポータルの一番左にある垂直メニュー バーの「**パイプライン**」セクションで「**リリース**」をクリックします。 
1.  「**パイプライン/リリース**」ペインで、**MHCDocker.release** エントリが選択されていることを確認し、「**編集**」をクリックします。
1.  「**すべてのパイプライン/MHCDocker.Release**」ペインで、デプロイの **Dev** ステージを示す長方形の中にある「**2 ジョブ、2 タスク**」リンクをクリックします。

    >**注**:リリース パイプラインは以下のタスクで構成されます

    | タスク | 使用 |
    | ----- | ----- |
    | **Azure SQL:DacpacTask の実行** | ターゲット スキーマとデータを含む dacpac 成果物を Azure SQL データベースにデプロイします |
    | **Azure App Service のデプロイ** | ビルド段階で生成された Docker イメージを、指定されたコンテナー レジストリからプルし、そのイメージを Azure App Service Web アプリにデプロイします |

1.  パイプラインのタスクのリストで、 **[Azure SQL:DacpacTask の実行]** タスクをクリックし、右側の **[Azure SQL Database のデプロイ]** ペインの **[Azure サブスクリプション]** ドロップダウン リストで、このタスクの前半で作成した Azure サービス接続を示すエントリを選択します。
1.  パイプラインのタスクのリストで、「**Azure App Service デプロイ**」タスクをクリックし、右側の「**Azure App Service デプロイ**」ペインで、**Azure サブスクリプション** ドロップダウン リストで、このタスクの前半で作成した Azure サービス接続を表すエントリを選択します。また、「**App Service 名**」ドロップダウンリストで、このラボの前半でデプロイした Azure App Service Web アプリを表すエントリを選択します。

    >**注**:次に、デプロイに必要なエージェント プール情報を構成する必要があります。

1.  右側の **[エージェント ジョブ]** ペインで **[DB デプロイ ジョブ]** を選択します。 **[エージェント プール]** ドロップダウン リストで **[Azure Pipelines]** を選択し、次に **[エージェント仕様]** ドロップダウン リストで **[windows-2019]** を選択します。
1.  右側の **[エージェント ジョブ]** ペインで **[Web アプリ デプロイ]** ジョブを選択します。 **[エージェント プール]** ドロップダウン リストで **[Azure Pipelines]** を選択し、次に **[エージェント仕様]** ドロップダウン リストで **[ubuntu-18.04]** を選択します。
1.  ペインの上部にある「**変数フィルター」** ヘッダーをクリックします。
1.  パイプライン変数のリストで、次の変数の値を設定します。

    | 変数 | 値 |
    | -------- | ----- |
    | ACR | このラボの前回の演習で記録した Azure Container Registry のログイン名 (**azurecr.io** サフィックスを含む) |
    | DatabaseName | **az400m15sqldb** |
    | パスワード | **Pa55w.rd1234** |
    | SQLadmin | **sqladmin** |
    | SQLServer | このラボの前回の演習で記録した Azure SQL Database 論理サーバー名 (**database.windows.net** サフィックスを含む) |

1.  ペインの右上隅にある「**保存**」ボタンをクリックして変更を保存し、再度プロンプトが表示されたら「**OK**」をクリックします。

#### <a name="task-2-trigger-build-and-release-pipelines-by-using-code-commit"></a>タスク 2:コード コミットを使用して、ビルド パイプラインとリリース パイプラインをトリガーする 

この演習では、コード コミットを使用してビルド パイプラインとリリース パイプラインをトリガーします。

1.  Azure DevOps ポータルが表示されている Web ブラウザー ウィンドウで、Azure DevOps ポータルの一番左にある垂直メニュー バーの「**リポジトリ**」をクリックします。 

    >**注**:「**ファイル**」ペインが自動的に表示されます。 

1.  「**ファイル**」ペインで、**src/MyHealth.Web/Views/Home** フォルダーに移動し、**Index.cshtml** ファイルを表すエントリをクリックしてから、「**編集**」をクリックして編集用に開きます。
1.  「**Index.cshtml**」ペインの **28** 行目で、「**JOIN US**」を「**CONTACT US**」に変更し、ペインの右上隅にある「**コミット**」をクリックし、確認を求められたら、もう一度「**コミット**」をクリックします。

    >**注**:このアクションにより、ソース コードの自動ビルドが開始されます。

1.  Azure DevOps ポータルが表示されている Web ブラウザー ウィンドウで、Azure DevOps ポータルの一番左にある垂直メニュー バーの「**パイプライン**」をクリックします。 
1.  「**パイプライン**」ペインで、コミットによってトリガーされたパイプライン実行を表すエントリをクリックします。
1.  「**MHCDocker.build**」ペインで、パイプラインの実行を表すエントリをクリックします。
1.  パイプライン実行の「**概要**」タブの「**ジョブ**」セクションで、**Docker** エントリをクリックし、表示されるペインで、ジョブが正常に完了するまで個々のタスクの進行状況を監視します。 

    >**注**:ビルドは Docker イメージを生成し、それを Azure Container Registry にプッシュします。 ビルドが完了すると、その概要を確認できるようになります。

1.  Azure DevOps ポータルが表示されている Web ブラウザー ウィンドウで、Azure DevOps ポータルの一番左にある垂直メニュー バーの「**パイプライン**」セクションで「**リリース**」をクリックします。 
1.  「**リリース**」ペインで、ビルドの成功によってトリガーされた最新のリリースを表すエントリをクリックします。
1.  「**MHCDocker.release > Release-1**」ペインで、**Dev** ステージを表す長方形を選択します。
1.  「**MHCDocker.release > Release-1 > Dev**」ペインで、正常に完了するまでリリース タスクの進行状況を監視します。 

    >**注**:このリリースでは、ビルド プロセスによって生成された Docker イメージがA pp Service Web アプリにデプロイされます。 リリースが完了すると、その概要とログを確認できます。 

1.  リリース パイプラインが完了したら、[Azure portal](https://portal.azure.com) を表示している Web ブラウザー ウィンドウに切り替えて、このラボで以前にプロビジョニングした Azure App Service Web アプリのブレードに移動します。
1.  App Service Web アプリで、ターゲット Web アプリを表す **URL** リンク エントリをクリックします。

    >**注**:これにより、ターゲット Web サイトを表示する新しい Web ブラウザー タブが自動的に開きます。

1.  CI/CD パイプラインをトリガーするために適用した変更を含め、ターゲット Web アプリに HealthClinic.biz Web サイトが表示されることを確認します。

### <a name="exercise-3-remove-the-azure-lab-resources"></a>演習 3:Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

>**注**:新規に作成し、使用しなくなったすべての Azure リソースを削除することを忘れないでください。 使用していないリソースを削除することで、予期しない料金が発生しなくなります。

#### <a name="task-1-remove-the-azure-lab-resources"></a>タスク 1:Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'az400m1501')].name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体を通して作成したすべてのリソース グループを削除します。

    ```sh
    az group list --query "[?starts_with(name,'az400m1501')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注**:コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## <a name="review"></a>確認

このラボでは、Azure DevOps CI/CD パイプラインを使用してカスタム Docker イメージを構築し、それを Azure Container Registry にプッシュし、Azure DevOps を使用して Azure App Service にコンテナーとしてデプロイしました。
