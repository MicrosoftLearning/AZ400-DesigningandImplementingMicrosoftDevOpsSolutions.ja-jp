---
lab:
  title: ラボ 21:Application Insights を使用したアプリケーション パフォーマンスの監視
  module: 'Module 09: Implement continuous feedback'
ms.openlocfilehash: f6bd76e03bef3782b5aabf447711974f11bf823a
ms.sourcegitcommit: f72fcf5ee578f465b3495f3cf789b06c530e88a4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2022
ms.locfileid: "139262611"
---
# <a name="lab-21-monitoring-application-performance-with-application-insights"></a>ラボ 21:Application Insights を使用したアプリケーション パフォーマンスの監視
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-overview"></a>ラボの概要

Application Insights は、複数のプラットフォームで使用できる Web 開発者向けの拡張可能なアプリケーション パフォーマンス管理 (APM) サービスです。 実行中の Web アプリケーションを監視するために使用できます。 パフォーマンスの異常を自動的に検出するほか、問題の診断に役立つ強力な分析ツールを備え、継続的なパフォーマンスと使用可能性を向上させます。 オンプレミス、ハイブリッド、または任意のパブリック クラウドでホストされている .NET、Node.js、Java EE などのさまざまなプラットフォーム上のアプリで機能します。 さまざまな開発ツールで接続ポイントを利用でき、DevOps プロセスと統合できます。 また、Visual Studio App Center と統合することにより、モバイル アプリからの製品利用統計情報を監視して分析できます。

このラボでは、Application Insights を既存の Web アプリケーションに追加する方法と、Azure portal を介してアプリケーションを監視する方法について学習します。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- Azure App Service Web アプリをデプロイする
- Application Insights を使用して Azure Web アプリのアプリケーション トラフィックを生成して監視する
- Application Insights を使用して Azure Web アプリのパフォーマンスを調べる
- Application Insights を使用して Azure Web アプリの使用状況を追跡する
- Application Insights を使用して Azure Web アプリのアラートを作成する

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

このタスクでは、Azure DevOps Demo Generator を使用し、**Parts Unlimited** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。 このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**:サイトの詳細については、 https://docs.microsoft.com/en-us/azure/devops/demo-gen を参照してください。

1.  「**サインイン**」をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。
1.  必要な場合は、「**Azure DevOps Demo Generator**」ページで「**承諾する**」をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。
1.  **[新しいプロジェクトの作成]** ページで **[新しいプロジェクト名]** テキストボックスに「**アプリケーションのパフォーマンスの監視**」と入力します。 **[組織の選択]** ドロップダウン リストで Azure DevOps 組織を選択し、 **[テンプレートの選択]** をクリックします。
1.  テンプレートの一覧 で **[PartsUnlimited]** テンプレートを選択 し、 **[テンプレートの選択]** をクリック します。
1.  再び「**新しいプロジェクトの作成**」ページで「**プロジェクトの作成**」をクリックします。

    > **注**:プロセスが完了するまでお待ちください。 これには 2 分ほどかかります。 プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

#### <a name="task-2-create-azure-resources"></a>タスク 2: Azure リソースを作成する

このタスクでは、Azure portal の Cloud Shell を使用して、Azure Web アプリと Azure SQL データベースを作成します。

> **注**:このラボでは、Parts Unlimited サイトを Azure アプリ サービスにデプロイします。 この要件を達成するため、必要なインフラストラクチャをスピンアップする必要があります。 

1.  ラボのコンピューターで Web ブラウザーを起動し、[**Azure Portal**](https://portal.azure.com) に移動します。このラボで使用する Azure サブスクリプションで所有者ロールがあり、このサブスクリプションに関連のある Azure AD テナントでグローバル管理者ロールがあるユーザー アカウントを使ってサインインします。
1. Azure portal のツールバーで、検索テキスト ボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。
1. **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、**[Bash]** を選択します。
    >**注**:**Cloud Shell** を初めて起動し、「**ストレージがマウントされていません**」というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、 **[ストレージの作成]** を選択します。

1.  **Bash** プロンプトの **[Cloud Shell]** ペインで、次のコマンドを実行してリソース グループを作成します (`<region>` プレースホルダーを 'eastus' などのご自分の場所に最も近い Azure リージョンの名前に置き換えます)。

    ```bash
    RESOURCEGROUPNAME='az400m17l01a-RG'
    LOCATION='<region>'
    az group create --name $RESOURCEGROUPNAME --location $LOCATION
    ```

1.  次のコマンドを実行して Windows App Service プランを作成するには、次のようにします。

    ```bash
    SERVICEPLANNAME='az400l17-sp'
    az appservice plan create --resource-group $RESOURCEGROUPNAME \
        --name $SERVICEPLANNAME --sku B3 
    ```
    > **注**:`ModuleNotFoundError: No module named 'vsts_cd_manager'` で始まるエラー メッセージで `az appservice plan create` コマンドが失敗した場合は、次のコマンドを実行してから、失敗したコマンドを再び実行します。

    ```bash
    az extension remove --name appservice-kube
    az extension add --yes --source "https://aka.ms/appsvc/appservice_kube-latest-py2.py3-none-any.whl"
    ```
1.  一意の名前を指定して Web アプリを作成します。

    ```bash
    WEBAPPNAME=partsunlimited$RANDOM$RANDOM
    az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME 
    ```

    > **注**:Web アプリの名前を記録します。 このラボで後ほど必要になります。

1. ここで、Application Insights インスタンスを作成します。

    ```bash
    az monitor app-insights component create --app $WEBAPPNAME \
        --location $LOCATION \
        --kind web --application-type web \
        --resource-group $RESOURCEGROUPNAME
    ```

    > **注**:"コマンドには、拡張機能の application-insights が必要です。 今すぐインストールしますか?" というプロンプトが表示されたら、「Y」と入力し、Enter キーを押します。

1. Application Insights を Web アプリケーションに接続してみましょう。

    ```bash
    az monitor app-insights component connect-webapp --app $WEBAPPNAME \
        --resource-group $RESOURCEGROUPNAME --web-app $WEBAPPNAME
    ```

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

### <a name="exercise-1-monitor-an-azure-app-service-web-app-by-using-azure-application-insights"></a>演習 1:Azure Application Insights を使用して Azure App Service Web アプリを監視する

この演習では、Azure DevOps パイプラインを使用した Web アプリの Azure App Service へのデプロイ、その Web アプリを対象にしたトラフィックの生成、Application Insights を使用した Web トラフィックのレビュー、アプリケーションのパフォーマンスの調査、アプリケーションの使用状況の追跡、アラートの構成を行います。

#### <a name="task-1-deploy-a-web-app-to-azure-app-service-by-using-azure-devops"></a>タスク 1:Azure DevOps を使用して Web アプリを Azure App Service にデプロイする

このタスクでは、Azure DevOps パイプラインを使用して Web アプリを Azure にデプロイします。

> **注**:このラボで使用するサンプル プロジェクトには、継続的インテグレーションのビルドが含まれており、これを変更せずに使用します。 また、継続的デリバリー リリース パイプラインもあり、前のタスクで実装した Azure リソースにデプロイする前に少々変更を加える必要があります。 

1.  Azure DevOps ポータルで **[アプリケーションのパフォーマンスの監視]** プロジェクトを表示している Web ブラウザー ウィンドウに切り替えます。垂直ナビゲーション ペインで **[パイプライン]** を選択し、 **[パイプライン]** セクションで **[リリース]** を選択します。
1.  リリース パイプライン一覧の **[PartsUnlimitedE2E]** ペインで **[編集]** をクリックします。 
1.  **[すべてのパイプライン] > [PartsUnlimitedE2E]** ペインで、**Dev** ステージを示している長方形をクリックします。 **[Dev]** ペインで **[削除]** をクリックし、 **[削除ステージ]** ダイアログ ボックスで **[確定]** をクリックします。
1.  **[すべてのパイプライン] > [PartsUnlimitedE2E]** ペインに戻り、**QA** ステージを示している長方形をクリックします。 **[QA]** ペインで **[削除]** をクリックし、 **[削除ステージ]** ダイアログ ボックスで **[確定]** をクリックします。
1.  **[すべてのパイプライン] > [PartsUnlimitedE2E]** ペインに戻り、**運用** ステージを示す長方形で **[1 ジョブ、1 タスク]** リンクをクリックします。
1.  **運用** _ ステージのタスク一覧が表示されているペインで、_ *[Azure App Service のデプロイ]* * タスクを示すエントリをクリックします。
1.  **[Azure App Service のデプロイ]** ペインの **[Azure サブスクリプション]** ドロップダウン リストで、このラボで使用している Azure サブスクリプションを示すエントリを選択してから **[承認]** をクリックし、該当するサービス接続を作成します。 指示されたら、Azure サブスクリプションで所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントでグローバル管理者のロールがあるアカウントを使用してサインインします。
1.  **[すべてのパイプライン] > [PartsUnlimitedE2E]** ペインの **[タスク]** タブを有効にし、 **[パイプライン]** タブのヘッダーをクリックしてパイプラインの図に戻ります。 
1.  この図で、**運用** ステージを示す長方形の左側にある **[デプロイ前の条件]** の楕円形記号をクリックします。
1.  **[デプロイ前の条件]** ペインの **[トリガーの選択]** セクションで **[リリース後]** を選択します。

    > **注**:プロジェクトのビルド パイプラインに成功すると、リリース パイプラインが呼び出されます。

1.  **[すべてのパイプライン] > [PartsUnlimitedE2E]** ペインの **[パイプライン]** タブを有効にし、 **[変数]** タブのヘッダーをクリックします。
1.  変数リストで、このラボで先ほど作成した Azure App Service Web アプリの名前に一致するように **WebsiteName** 変数の値を設定します。
1.  ペインの右上コーナーで **[保存]** をクリックします。指示されたら、 **[保存]** ダイアログ ボックスで再び **[OK]** をクリックします。

    > **注**:リリース パイプラインが配置されたので、マスター ブランチへのコミットによりビルド パイプラインとリリース パイプラインがトリガーされます。

1.  Azure DevOps ポータルが表示されている Web ブラウザー ウィンドウの垂直ナビゲーション ペインで、 **[リポジトリ]** をクリックします。 
1.  **[ファイル]** ペインで **PartsUnlimited-aspnet45/src/PartsUnlimitedWebsite/Web.config** ファイルに移動して選択します。

    > **注**:このアプリケーションには、Application Insights キーと SQL 接続の構成設定がすでにあります。 

1. **[Web.config]** ペインで、Application Insights キーと SQL 接続を参照するラインをレビューします:

    ```xml
    <add key="Keys:ApplicationInsights:InstrumentationKey" value="0839cc6f-b99b-44b1-9d74-4e408b7aee29" />
    ```

    ```xml
    <connectionStrings>
       <add name="DefaultConnectionString" connectionString="Server=(localdb)\mssqllocaldb;Database=PartsUnlimitedWebsite;Integrated Security=True;" providerName="System.Data.SqlClient" />
    </connectionStrings>
    ```

    > **注**:デプロイ後、Azure portal でこれらの設定の値を変更し、このラボで先ほどデプロイした Azure Application Insights と Azure SQL Database を示すようにします。

    > **注**:ここで、空のラインをファイルの最後に追加してビルド プロセスとリリース プロセスをトリガーします。関連のあるコードを変更する必要はありません。

1. **[Web.config]** ペインで **[編集]** をクリックし、ファイルの最後に空のラインを追加します。 **[コミット]** をクリックして、 **[コミット]** ペインで再び **[コミット]** をクリックします。

    > **注**:新しいビルドが始まり、最終的に Azure にデプロイされます。 デプロイが完了するのを待たず、次の手順に進んでください。

1.  Azure portal を表示している Web ブラウザーに切り替え、このラブで先ほどプロビジョニングした App Service Web アプリに移動します。 
1.  App Service Web アプリ ブレードで左側の垂直メニューをクリックし、 **[設定]** セクションで **[構成]** タブをクリックします。
1. **[アプリケーション設定]** リストで **[APPINSIGHTS_INSTRUMENTATIONKEY]** エントリをクリックします。 (このエントリが表示されない場合は、[設定] の下の **[Application Insights]** を選択し、[Application Insights] を **[有効]** にして **[適用]** を選択します。) 
1.  **[アプリケーション設定の追加 / 編集]** ブレードで **[値]** テキストボックスのテキストをコピーして、 **[キャンセル]** をクリックします。

    > **注**:これは、App Service Web アプリのデプロイ中に追加された既定の設定で、すでに Application Insights ID が含まれています。 このアプリで予想される新しい設定を追加する必要があります。名前は変更しますが、値は同じものを維持します。 これは、このサンプルの特別な要件です。

1.  **[アプリケーション設定]** セクションで **[+ 新しいアプリケーション設定]** をクリックします。
1. **[アプリケーション設定の追加 / 編集]** ブレードで **[名前]** テキストボックスに「**Keys:ApplicationInsights:InstrumentationKey**」と入力します。 **[値]** テキストボックスに、クリップボードにコピーした文字列を入力し、 **[OK]** をクリックします。 **[保存]** を選択します。

    > **注**:アプリケーション設定と接続文字列を変更すると、Web アプリの再起動がトリガーされます。

1.  Azure DevOps ポータルの Web ブラウザー ウィンドウに戻り、垂直ナビゲーション ペインで **[パイプライン]** を選択します。 **[パイプライン]** セクションで、最も直近に実行したビルド パイプラインを示すエントリをクリックします。
1.  ビルドがまだ完了していない場合は、完了するまで追跡します。その後、垂直ナビゲーション ペインの **[パイプライン]** セクションで **[リリース]** をクリックし、 **[PartsUnlimiteE2E]** ペインで **[Release-1]** をクリックしてリリース パイプラインが完了するまでフォローします。
1.  Azure portal が表示されている Web ブラウザー ウィンドウに切り替え、 **[App Service Web アプリ]** ブレードの左側にある垂直メニュー バーで **[概要]** をクリックします。 
1.  右側の **[要点]** セクションで **[URL]** リンクをクリックします。 別の Web ブラウザー タブが自動的に開き、**Parts Unlimited** Web サイトが表示されます。
1.  **Parts Unlimited** Web サイトが期待通りに読み込まれることを確認します。 

#### <a name="task-2-generate-and-review-application-traffic"></a>タスク 2:アプリケーション トラフィックを生成してレビューする

このタスクでは、前のタスクでデプロイした App Service Web アプリを対象にトラフィックを生成し、この Web アプリに関連のある Application Insights リソースが収集したデータをレビューします。

1.  **Parts Unlimited** Web サイトが表示されている Web ブラウザー ウィンドウで、該当するページに移動してトラフィックを生成します。
1.  **Parts Unlimited** Web サイトで **[ブレーキ]** メニュー項目をクリックします。
1.  ブラウザー ウィンドウの最上部にある URL テキストボックスで、URL 文字列の最後に **1** を追加して **Enter** を押します。これにより、**CategoryId** パラメーターは **11** に設定されます。 

    > **注**:そのカテゴリーは存在しないのでサーバー エラーが生じます。 ページを数回更新すると、さらにエラーが生成されます。

1.  Azure portal が表示されている Web ブラウザー タブに戻ります。
1.  Azure portal を表示している Web ブラウザー タブで、 **[App Service]** Web アプリ ブレードの左側にある垂直メニュー バーの **[設定]** セクションで **[Application Insights]** エントリをクリックして **Application Insights** 構成ブレードを表示します。

    > **注**:このブレードには、Application Insights をさまざまなタイプのアプリに統合できる設定が含まれています。 既定の動作によりアプリを追跡して監視するための豊富なデータを得られますが、API はもっと専門的なシナリオのサポートとカスタム イベント追跡機能を提供します。

1.  **[Application Insights]** 構成ブレードで **[Application Insights データの表示]** リンクをクリックします。
1.  その結果、 **[Application Insights]** ブレードには収集されたデータのさまざまな特徴を示す図が表示されるので、これを確認します。この中には、生成されたトラフィックや、このタスクで先ほどトリガーされたものの失敗した要求が含まれます。

    > **注**:何も表示されなかった場合は、数分待って、概要セクションにログが表示され始めるまでページを更新してください。

#### <a name="task-3-investigate-application-performance"></a>タスク 3:アプリケーションのパフォーマンスを調べる

このタスクでは、Application Insights を使用して App Service Web アプリのパフォーマンスを調べます。

1.  **[Application Insights]** ブレードの左側にある垂直メニューの **[調査]** セクションで **[アプリケーション マップ]** をクリックします。

    > **注**:アプリケーション マップは、分散アプリケーションのすべてのコンポーネントでパフォーマンスのボトルネックや障害ホットスポットを見つけることができます。 マップ上の各ノードは、アプリケーション コンポーネントまたはその依存関係、正常性 KPI およびアラートステータスを表します。 任意のコンポーネントをクリックして、さらに詳しい診断結果 (Application Insights イベントなど) にアクセスすることができます。 アプリで Azure サービスを使用している場合は、SQL データベース アドバイザーの推奨事項など、サービスに関連した Azure 診断をクリックスルーできます。

1.  **[Application Insights]** ブレードの左側にある垂直メニューの **[調査]** セクションで **[スマート検出]** をクリックします。

    > **注**:スマート検出により、Web アプリケーションの潜在的なパフォーマンスの問題について警告を自動的に受け取ることができます。 スマート検出では、アプリから Application Insights に送信されるテレメトリがプロアクティブに分析されます。 障害発生率が急激に上昇したり、クライアントまたはサーバーのパフォーマンスに異常なパターンが発生したりした場合に、アラートが表示されます。 この機能には構成は不要です。 アプリケーションから適切なテレメトリが送信されていれば動作します。 ただし、アプリはデプロイしたばかりなので、まだデータはありません。

1.  **[Application Insights]** ブレードの左側にある垂直メニューの **[調査]** セクションで **[ライブ メトリック]** をクリックします。

    > **注**:ライブ メトリック ストリームでは、運用中の Web アプリケーションという中核的な要素を調べられます。 メトリックとパフォーマンス カウンターを選択してフィルタリングし、サービスに影響を与えることなくリアルタイムで監視できます。 失敗した要求と例外のサンプルからスタック トレースを確認することもできます。

1.  **Parts Unlimited** Web サイトが表示されている Web ブラウザー ウィンドウに戻り、該当するページに移動してトラフィックを生成します (サーバー エラーも含まれます)。 
1.  Azure portal を表示している Web ブラウザーに戻り、受信するライブ トラフィックを監視します。 
1.  **[Application Insights]** ブレードの左側にある垂直メニューの **[調査]** セクションで、 **[トランザクション検索]** をクリックします。

    > **注**:トランザクション検索は柔軟なインターフェイスで、質問に答えるために必要となる正確なテレメトリを見つけられます。 

1.  **[トランザクション検索]** ブレードで **[直近 24 時間のデータをすべて表示する]** をクリックします。

    > **注**:この結果にはあらゆるテレメトリ データが含まれ、複数のプロパティで絞り込めます。

1.  **[トランザクション検索]** ブレードの中央、結果リストのすぐ上で **[グループ化された結果]** をクリックします。 

    > **注**:これらの結果は一般的なプロパティに基づいてグループ化されます。

1.  **[トランザクション検索]** ブレードの中央、結果リストのすぐ上で **[結果]** をクリックすると、あらゆる結果が一覧表示された元のビューに戻れます。
1.  **[トランザクション検索]** ブレードの最上部で **[イベントの種類 = すべて選択済み]** をクリックします。ドロップダウン リストで **[すべて選択]** チェックボックスをクリアし、イベントの種類リストで **[例外]** チェックボックスを選択します。

    > **注**:先ほど生成したエラーを示す例外がいくつかあるはずです。 

1.  結果リストで、そのひとつをクリックします。 これにより **[エンド ツー エンド トランザクションの詳細]** ブレードが表示され、要求のコンテキスト内の詳細な例外のタイムラインが表示されます。 
1.  **[エンド ツー エンド トランザクションの詳細]** ブレードの最下部で **[すべてのテレメトリを表示]** をクリックします。

    > **注**: **[テレメトリ]** にも同じデータが表示されますが、形式が異なります。 **[エンド ツー エンド トランザクションの詳細]** ブレードの右側で、例外の詳細 (プロパティやコール スタックなど) もレビューできます。

1.  **[エンド ツー エンド トランザクションの詳細]** ブレードを閉じて **[トランザクション検索]** ブレードに戻り、左側の垂直メニューの **[調査]** セクションで **[可用性]** をクリックします。

    > **注**:Web アプリまたは Web サイトを任意のサーバーに展開したら、その可用性と応答性を監視するテストを設定できます。 Application Insights は、世界各地の複数のポイントから定期的にアプリケーションに Web 要求を送信します。 アプリケーションが応答しないか、応答が遅い場合に警告します。 

1.  **[可用性]** ブレードのツールバーで **[+ テストの追加]** をクリックします。
1.  **[テストの作成]** ブレードで **[テスト名]** テキストボックスに「**ホーム ページ**」と入力します。**URL** を App Service Web アプリのルートに設定し、 **[作成]** をクリックします。

    > **注**:テストはすぐに実行されないため、データは表示されません。 後ほど確認すると、実際のサイトに対してテストを反映するよう可用性データが更新されているはずです。 今はこの更新を待たないでください。

1.  **[可用性]** ブレードの左側にある垂直メニューの **[調査]** セクションで **[障害]** をクリックします。

    > **注**:[障害] ビューでは、あらゆる例外レポートを単一のダッシュボードに集計します。 ここから、依存関係や例外などのフィルターに基づいて関連のあるデータを容易に見つけられます。 

1.  **[障害]** ブレードの右上コーナーにある **[トップ 3 応答コード]** リストで、**500** 件のエラーの数を示すリンクをクリックします。

    > **注**:これは、この HTTP 応答コードに一致する例外のリストを表します。 提案されている例外を選択すると、以前にレビューしたものと同じ例外ビューが表示されます。

1.  **[障害]** ブレードの左側にある垂直メニューの **[調査]** セクションで、 **[パフォーマンス]** をクリックします。

    > **注**:[パフォーマンス] ビューには、収集されたテレメトリに基づくアプリケーション パフォーマンスの詳細を簡素化したダッシュボードが表示されます。

1.  **[パフォーマンス]** ブレードの左側にある垂直メニューの **[監視]** セクションで、 **[メトリック]** をクリックします。

    > **注**:Application Insights のメトリックとは、アプリケーションからのテレメトリとして送信される測定値とイベントの数を表します。 メトリックは、パフォーマンスの問題を検出し、アプリケーションの利用に関する傾向を把握するのに役立ちます。 さまざまな標準メトリックが用意されているほか、独自にカスタムのメトリックとイベントを作成することもできます。 

1.  **[メトリック]** ブレードのフィルター セクションで **[メトリックの選択]** をクリックし、ドロップダウン リストで **[サーバー要求]** を選択します。

    > **注**:分割を使用してデータをセグメント化することもできます。 

1.  新しく表示されたチャートの最上部で **[分割の適用]** をクリックします。その結果生じるフィルターの **[値の選択]** ドロップダウン リストで **[操作名]** を選択します。 

    > **注**:これでサーバー要求は、参照するページに基づいて分割され、チャート上では異なる色で示されます。

#### <a name="task-4-track-application-usage"></a>タスク 4:アプリケーションの使用状況を追跡する

> **注**:Application Insights には、アプリケーションの使用状況を追跡するための広範な機能も備えられています。 

1.  **[メトリック]** ブレードの左側にある垂直メニューの **[使用状況]** セクションで、 **[ユーザー]** をクリックします。

    > **注**:このアプリケーションのユーザーはまだ多くありませんが、一部のデータを利用できます。 

1.  **[ユーザー]** ブレードのメイン チャートの下で **[その他の分析情報を表示]** をクリックします。 さらにデータが表示され、ブレードが下向きに拡張します。
1.  スクロールダウンして、地理やオペレーティング システム、ブラウザーに関する詳細をレビューします。

    > **注**:ユーザー固有の使用状況パターンをよりよく理解するため、ユーザー固有のデータに絞り込むこともできます。

1.  **[ユーザー]** ブレードの左側にある垂直メニューの **[使用状況]** セクションで、 **[イベント]** をクリックします。
1.  **[イベント]** ブレードのメイン チャートの下で **[その他の分析情報を表示]** をクリックします。

    > **注**:サイトの使用状況に基づいてこれまでに発生した広範な組み込みイベントも表示されます。 ニーズに合わせてカスタム データとともにカスタム イベントをプログラムで追加できます。

1.  **[イベント]** ブレードの左側にある垂直メニューの **[使用状況]** セクションで、 **[ファネル]** をクリックします。

    > **注**:カスタマー エクスペリエンスの理解は、ビジネスにとって極めて重要です。 アプリケーションの使用状況に複数の手順が含まれている場合は、ほとんどの顧客が意図されたプロセスに従っているか把握する必要があります。 Web アプリケーション内の一連の手順の進行は、ファネルと呼ばれています。 Azure Application Insights のファネルを使用して、ユーザーに対する洞察を取得し、手順ごとのコンバージョン レートを監視できます。

1.  **[ファネル]** ブレードの左側にある垂直メニューの **[使用状況]** セクションで、 **[ユーザー フロー]** をクリックします。

    > **注**:ユーザー フロー ツールは、指定した最初のページ ビュー、カスタム イベント、または例外から開始します。 この最初のイベントを前提に、ユーザー フローには、該当するユーザー セッション中、その前後に発生したイベントが表示されます。 太さが異なる線は、ユーザーが各パスを通過した回数を示します。 **[セッションを開始しました]** ノードは、セッションの開始を示します。 **[セッションが終了しました]** ノードは、前のノードの後でページ ビューまたはカスタム イベントを生成しなかったユーザーの数を示します (サイトを離れたユーザーに該当する可能性があります)。

1.  **[ユーザー フロー]** ブレードで **[イベントの選択]** をクリックします。 **[編集]** ペインにある **[最初のイベント]** ドロップダウン リストの **[ページ ビュー]** セクションで、 **[ホーム ページ - Parts Unlimited]** エントリを選択してから **[グラフの作成]** をクリックします。

1.  **[ユーザー フロー]** ブレードの左側にある垂直メニューの **[使用状況]** セクションで、 **[リテンション期間]** をクリックします。

    > **注**:Application Insights のリテンション期間機能を使用すると、アプリに戻るユーザーの数、およびそのユーザーが特定のタスクを実行したり目標を達成したりする頻度を分析できます。 たとえば、ゲームのサイトを運営する場合は、ゲームに負けた後にサイトに戻るユーザーの数と、勝った後に戻るユーザーの数を比較できます。 このデータによって、ユーザー エクスペリエンスとビジネス戦略の両方の強化に役立ちます。

    > **注**:ユーザー リテンション期間分析エクスペリエンスは Azure Workbooks に移行されました

1.  **[リテンション期間]** ブレードで **[リテンション期間分析ワークブック]** をクリックし、 **[保有期間全体]** チャートをレビューしてから、このブレードを閉じます。
1.  **[リテンション期間]** ブレードの左側にある垂直メニューの **[使用状況]** セクションで、 **[影響]** をクリックします。

    > **注**:[影響] では、Web サイトのプロパティ (読み込み時間など) がアプリのさまざまな部分の変換率に及ぼす影響について分析します。 より正確に言えば、ページ ビュー、カスタム イベント、または要求のディメンションが、ページ ビューまたはカスタム イベントにどのように影響するのかを明らかにします。

    > **注**:影響分析エクスペリエンスは Azure Workbooks に移行されました

1.  **[影響]** ブレードで **[影響分析ワークブック]** をクリックします。 **[影響]** ブレードの **[選択されたイベント]** ドロップダウン リストにある **[ページ ビュー]** セクションで **[ホーム ページ - Parts Unlimited]** を選択します。 **[影響するイベント]** で **[製品の参照 - Parts Unlimited]** を選択し、その結果をレビューしてブレードを閉じます。
1.  **[影響]** ブレードの左側にある垂直メニューの **[使用状況]** セクションで、 **[コーホート]** をクリックします。

    > **注**:コーホートとは、共通点を持つユーザー セット、セッション セット、イベント セット、または操作セットのことです。 Application Insights では、コーホートは分析クエリによって定義されます。 特定のユーザー セットまたはイベント セットを繰り返し分析する必要がある場合、コーホートを使用することで、より柔軟性の高い方法で目的のセットを正確に表現することができます。 コーホートはフィルターと同様の方法で使用されますが、コーホート定義はカスタム分析クエリから構築されます。そのため、より適応性が高く複雑になります。 フィルターとは異なり、コーホートの場合はチームの他のメンバーも再利用できるように保存することができます。

1.  **[コーホート]** ブレードの左側にある垂直メニューの **[使用状況]** セクションで、 **[その他]** をクリックします。

    > **注**:このブレードには、レビュー用のさまざまなレポートとテンプレートが含まれています。

1.  **[詳細 \| ギャラリー]** ブレードの **[使用状況]** セクションで、 **[ページ ビューの分析]** をクリックし、該当するブレードの内容を確認します。

    > **注**:この特定のレポートには、ページ ビューに関する分析情報が含まれています。 既定で利用できる他のレポートも多数あり、新しいレポートをカスタマイズして保存することもできます。

#### <a name="task-5-configure-web-app-alerts"></a>タスク 5:Web アプリの アラート を構成する

1.  **[詳細 \| ギャラリー]** ブレードで、左側にある垂直メニューの **[監視]** セクションで **[アラート]** をクリックします。 

    > **注**:アラートを使うと、Application Insights の測定が指定された条件に達するとアクションを実行するトリガーを設定できます。

1.  **[アラート]** ブレードのツールバーで **[+ 新しいアラート ルール]** をクリックします。
1.  **[アラート ルールの作成]** ブレードの **[スコープ]** セクションでは、既定で現在の Application Insights リソースが選択されます。 
1.  **[アラート ルールの作成]** ブレードの **[条件]** セクションで、 **[条件の追加]** をクリックします。
1.  **[信号論理の構成]** ブレードで **[失敗した要求]** メトリックを検索して選択します。
1.  **[信号論理の構成]** ブレードで **[アラート論理]** セクションまでスクロールダウンします。 **[しきい値]** が **[静的]** に設定されていることを確認し、 **[しきい値]** を **1** に設定します。 

    > **注**:2 回目の要求失敗が報告されると、アラートがトリガーされます。 既定により、直近 5 分間の測定集計に基づき、毎分、条件が評価されます。 

1.  **[信号論理の構成]** ブレードで **[完了]** をクリックします。

    > **注**:条件が作成されたので、実行する **アクション グループ** を定義する必要があります。 

1.  **[アラート ルールの作成]** ブレードに戻り、 **[アクション]** セクションで **[アクション グループの選択]** をクリックします。その後、**[このアラート ルールに添付するアクション グループを選択] で **[+ アクション グループの作成]** をクリックします。
1.  **[アクション グループの作成]** ブレードの **[基本]** タブで、次の設定を指定して **[次へ:通知 >]** をクリックします。

    | 設定 | 値 |
    | --- | --- |
    | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
    | リソース グループ | 新しいリソース グループ **az400m17l01b-RG** の名前 |
    | アクション グループ名 | **az400m17-action-group** |
    | Display name | **az400m17-ag** |

1.  **[アクション　グループの作成]** ブレードの **[通知]** タブにある **[通知の種類]** ドロップダウン リストで、 **[電子メール / SMS メッセージ/ プッシュ / 音声]** を選択します。 **[電子メール / SMS メッセージ / プッシュ / 音声]** ブレードが開きます。 
1.  **[電子メール / SMS メッセージ / プッシュ / 音声]** ブレードで **[電子メール]** チェックボックスを選択し、 **[電子メール]** テキストボックスにご自分のメール アドレスを入力して **[OK]** をクリックします。
1.  **[アクション グループの作成]** ブレードの **[通知]** タブに戻り、 **[名前]** テキストボックスに「**email**」と入力して **[次へ:アクション >]** をクリックします。
1.  **[アクション グループの作成]** ブレードの **[アクション]** タブで、 **[アクションの種類]** ドロップダウン リストを選択し、変更しなくても利用できるオプションを確認して **[レビューと作成]** をクリックします。
1.  **[アクション グループの作成]** ブレードの **[レビューと作成]** タブで、 **[作成]** をクリックします
1.  **[アラート ルールの作成]** ブレードに戻り、 **[アラート ルールの詳細]** セクションで **[アラート ルール名]** テキストボックスに「**az400m17 lab alert rule**」と入力します。残りのアラート ルール設定を変更することなく確認し、 **[アラート ルールの作成]** をクリックします。
1.  **Parts Unlimited** Web サイトを表示している Web ブラウザー ウィンドウに切り替え、**Parts Unlimited** Web サイトで **[ブレーキ]** メニュー項目をクリックします。
1.  ブラウザー ウィンドウの最上部にある URL テキストボックスで、URL 文字列の最後に **1** を追加して **Enter** を押します。これにより、**CategoryId** パラメーターは **11** に設定されます。 

    > **注**:そのカテゴリーは存在しないのでサーバー エラーが生じます。 ページを数回更新すると、さらにエラーが生成されます。

1.  5 分ほど経ったら、メール アカウントをチェックし、定義したアラートがトリガーされたことを知らせるメールを受け取っているか確認します。

### <a name="exercise-2-remove-the-azure-lab-resources"></a>演習 2:Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

>**注**:新規に作成し、使用しなくなったすべての Azure リソースを削除することを忘れないでください。 使用していないリソースを削除することで、予期しない料金が発生しなくなります。

#### <a name="task-1-remove-the-azure-lab-resources"></a>タスク 1:Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'az400m17l01')].name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体を通して作成したすべてのリソース グループを削除します。

    ```sh
    az group list --query "[?starts_with(name,'az400m17l01')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注**:コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## <a name="review"></a>確認

この演習では、Azure DevOps パイプラインを使用した Web アプリの Azure App Service へのデプロイ、その Web アプリを対象にしたトラフィックの生成、Application Insights を使用した Web トラフィックのレビュー、アプリケーションのパフォーマンスの調査、アプリケーションの使用状況の追跡、アラートの構成を行いました。
