---
lab:
  title: ラボ 13:機能テストの設定と実行
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
ms.openlocfilehash: 364154c285807d14dcbe9bbc511ede835930a350
ms.sourcegitcommit: f72fcf5ee578f465b3495f3cf789b06c530e88a4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2022
ms.locfileid: "139262608"
---
# <a name="lab-13-setting-up-and-running-functional-tests"></a>ラボ 13:機能テストの設定と実行
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-overview"></a>ラボの概要

[Selenium](http://www.seleniumhq.org/) は、Web アプリケーション用のポータブル オープン ソース ソフトウェア テスト フレームワークです。 ほとんどすべてのオペレーティングシステムで動作する機能があります。 これは、すべての最新のブラウザと .NET (C#)、Java を含む複数の言語をサポートしています。

このラボでは、Azure DevOps リリース パイプラインの一部として、C# Web アプリケーションで Selenium テスト ケースを実行する方法について説明します。 

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- セルフホステッド Azure DevOps エージェントを構成する
- リリース パイプラインを構成する
- ビルドとリリースをトリガーする
- Chrome と Firefox でテストを実行する

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

この演習では、ラボの前提条件を設定します。Azure DevOps Demo Generator テンプレートと Azure リソースに基づいて事前に構成された Parts Unlimited チーム プロジェクトが含まれます。 

#### <a name="task-1-configure-the-team-project"></a>タスク 1:チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用して、**Selenium** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。 このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**:このサイトの詳細については、「[Azure DevOps Services Demo Generator とは](https://docs.microsoft.com/en-us/azure/devops/demo-gen)」を参照してください。

1.  「**サインイン**」をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。
1.  必要な場合は、「**Azure DevOps Demo Generator**」ページで「**承諾する**」をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。
1.  **新しいプロジェクトの作成** ページで **[新しいプロジェクト名]** テキストボックスに「**機能テストの設定と実行**」と入力します。 **[組織の選択]** ドロップダウン リストで Azure DevOps 組織を選択し、 **[テンプレートの選択]** をクリックします。
1.  テンプレートのリストのツールバーで「**DevOps ラボ**」をクリックし、「**Selenium**」テンプレートを選択して「**テンプレートの選択**」をクリックします。
1.  再び「**新しいプロジェクトの作成**」ページで「**プロジェクトの作成**」をクリックします。

    > **注**:プロセスが完了するまでお待ちください。 これには 2 分ほどかかります。 プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

#### <a name="task-2-create-azure-resources"></a>タスク 2: Azure リソースを作成する

このタスクでは、Windows Server 2016 のほか、SQL Express 2017、Chrome、Firefox を実行している Azure VM をプロビジョニングします。

1.  **[[Azure にデプロイ]](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2Falmvm%2Fmaster%2Flabs%2Fvstsextend%2Fselenium%2Farmtemplate%2Fazuredeploy.json)** リンクをクリックします。 これで自動的に Azure portal の「**カスタム デプロイ**」ブレードにリダイレクトされます。
1.  指示されたら、このラボで使用する Azure サブスクリプションで所有者のロールがあり、このサブスクリプションに関連のある Azure AD テナントでグローバル管理者のロールがあるユーザー アカウントを使用してサインインします。
1.  「**カスタム デプロイ**」ブレードで、「**パラメーターの編集**」を選択します。
1.  **[テンプレートの編集]** ブレードで、行 `"https://raw.githubusercontent.com/microsoft/azuredevopslabs/master/labs/vstsextend/selenium/armtemplate/chrome_firefox_VSTSagent_IIS.ps1"` を探して `https://raw.githubusercontent.com/MicrosoftLearning/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/master/Allfiles/Labs/11b/chrome_firefox_VSTSagent_IIS.ps1` に置き換え、 **[保存]** をクリックします。
1.  「**カスタム デプロイ**」ブレードに戻って、次の設定を指定します。

    | 設定 | 値 |
    | --- | --- |
    | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
    | リソース グループ | 新しいリソース グループの名前 **az400m11l02-RG** |
    | リージョン | このラボで Azure リソースをデプロイする Azure リージョンの名前 |
    | 仮想マシン名 | **az40011bvm** |

1.  **[Review + create]\(レビュー + 作成\)** をクリックし、 **[作成]** をクリックします。 

    > **注**:プロセスが完了するまでお待ちください。 通常は 15 分ほどかかります。 

### <a name="exercise-1-implement-selenium-tests-by-using-a-self-hosted-azure-devops-agent"></a>演習 1:セルフホステッド Azure DevOps エージェントを使用して Selenium テストを実装する

この演習では、セルフホステッド Azure DevOps エージェントを使用して Selenium テストを実装します。

#### <a name="task-1-configure-a-self-hosted-azure-devops-agent"></a>タスク 1:セルフホステッド Azure DevOps エージェントを構成する

このタスクでは、以前の演習でデプロイした VM を使用してセルフホステッド エージェントを構成します。 Selenium では、インタラクティブ モードでエージェントを実行し、UI テストを実行する必要があります。

1.  Azure portal が表示されている Web ブラウザー ウィンドウで「**仮想マシン**」を検索して選択し、「**仮想マシン**」ブレードで「**az40011bvm**」を選択します。
1.  **[az40011bvm]** ブレードで、 **[接続]** を選択し、ドロップダウン メニューの **[RDP]** を選択します。 **[az40011bvm] \| [接続]** ブレードの **[RDP]** タブで、 **[RDP ファイルのダウンロード]** を選択し、ダウンロードしたファイルを開きます。
1.  プロンプトが表示されたら、次の認証情報を入力します。

    | 設定 | 値 |
    | --- | --- |
    | [ユーザー名] | **vmadmin** |
    | Password | **P2ssw0rd@123** |

1.  **az40011bvm** へのリモート デスクトップ セッション内で Chrome Web ブラウザー ウィンドウを開き、 **https://dev.azure.com** に移動して Azure DevOps 組織にサインインします。 
1.  **Azure DevOps** ポータルの左下コーナーで、「**組織の設定**」をクリックします。
1.  ページの左側にある垂直メニューの「**パイプライン**」セクションで「**エージェント プール**」をクリックします。
1.  「**エージェント プール**」ペインで「**既定**」をクリックします。 
1.  「**既定**」ペインで「**新しいエージェント**」をクリックします。 
1.  「**エージェントの取得**」パネルで、「**Windows**」タブと「**x64**」セクションが選択されていることを確認してから、「**ダウンロード**」をクリックします。
1.  エクスプローラーを起動して **C:\\AzAgent** ディレクトリを作成し、**ダウンロード** フォルダーにあるダウンロード済みのエージェント Zip ファイルの内容を展開します。
1.  「**az40011bvm**」へのリモート デスクトップ セッション内で、「**起動**」メニューを右クリックし、「**コマンド プロンプト (管理者)** 」をクリックします。
1.  **管理者:コマンド プロンプト** ウィンドウで、以下を実行してエージェント バイナリのインストールを開始します。

    ```cmd
    cd C:\AzAgent
    Config.cmd
    ```
1.  **[管理者:コマンド プロンプト]** ウィンドウで、 **[Enter server URL]** \(サーバーの URL を入力します\) と表示されたら **https://dev.azure.com/\<your-DevOps-organization-name\>** と入力します。ここで **\<your-DevOps-organization-name\>** は、Azure DevOps 組織を表します。次に **Enter** キーを押します。
1.  **[管理者:コマンド プロンプト]** ウィンドウで、 **[Enter Authentication type (press enter for PAT)]** \(認証タイプを入力します \(PAT の場合は Enter を押します\)\) と表示されたら **Enter キー** を押します。
1.  **[管理者:コマンド プロンプト]** ウィンドウで、 **[Enter personal access token]** \(個人用アクセス トークンを入力します\) と表示されたら、Azure DevOps ポータルに切り替え、 **[エージェントの取得]** パネルを閉じ、Azure DevOps ページの右上隅で **[ユーザー設定]** アイコンをクリックします。ドロップダウン メニューで **[個人用アクセス トークン]** をクリックし、 **[個人用アクセス トークン]** ペインで **[+ 新しいトークン]** をクリックします。
1.  「**新しい個人用アクセス トークンの作成**」ペインで以下の設定を指定し、「**作成**」をクリックします (他はすべて規定値のままにします):

    | 設定 | 値 |
    | --- | --- |
    | Name | **機能テストの設定と実行ラボ** |
    | スコープ | **カスタム定義済み** |
    | スコープ | ウィンドウの下部にある「**すべてのスコープを表示**」をクリックする |
    | スコープ | **エージェント プール** - **読み取りと管理** |

1.  「**成功**」ペインで個人用アクセス トークンの値をクリップボードにコピーします。

    > **注**:必ずトークンをコピーしてください。 このペインを閉じると、取得できなくなります。 

1.  「**成功**」ペインで「**閉じる**」をクリックします。
1.  **管理者:コマンド プロンプト** ウィンドウに戻り、クリップボードの内容を貼り付けてから **Enter** キーを押します。
1.  **[管理者:コマンド プロンプト]** ウィンドウで、 **[Enter agent pool (press enter for default)]** \(エージェント プールを入力します \(既定を使用する場合は Enter キーを押します\)\) と表示されたら **Enter キー** を押します。
1.  **[管理者:コマンド プロンプト]** ウィンドウで、 **[Enter agent name (press enter for az40011bvm)]** \(エージェント名を入力します\(az40011bvm の場合は Enter キーを押します\)\) と表示されたら **Enter キー** を押します。
1.  **[管理者:コマンド プロンプト]** ウィンドウで、 **[Enter work folder (press enter for _work)]** \(作業フォルダーを入力します\(_work の場合は Enter キーを押します\)\) と表示されたら **Enter キー** を押します。
1.  **[管理者:コマンド プロンプト]** ウィンドウで、 **[Enter run agent as service (Y/N) (press enter for N)]** \(サービスとしてのエージェントの実行を入力します\(はい/いいえ\) いいえの場合は Enter キーを押します\)\) と表示されたら **Enter キー** を押します。
1.  **[管理者:コマンド プロンプト]** ウィンドウで、 **[Enter configure autologon and run agent on startup (Y/N) (press enter for N)]** \(起動時の自動ログオンの構成とエージェントの実行を入力します\(はい/いいえ\) いいえの場合は Enter キーを押します\)\) と表示されたら **Enter キー** を押します。
1.  エージェントが登録されたら **[管理者:コマンド プロンプト]** ウィンドウに、「**run.cmd**」と入力して **Enter** キーを押しエージェントを起動します。

    > **注**:Dac Framework もインストールする必要があります。ラボで後ほどデプロイするアプリケーションで使用されます。

1.  **az40011bvm** へのリモート デスクトップ セッション内で、Web ブラウザーの別のインスタンスを起動し、[Microsoft SQL Serverデータ層アプリケーションフレームワーク (18.2) ダウンロード ページ](https://www.microsoft.com/en-us/download/details.aspx?id=58207&WT.mc_id=rss_alldownloads_extensions)に移動して、「**ダウンロード**」をクリックします。 
1.  「**必要なダウンロードの選択**」で、「**EN\x64\DacFramework.msi**」チェックボックスを選択し、「**次へ**」をクリックします。 これにより、**DacFramework.msi** ファイルの自動ダウンロードがトリガーされます。
1.  **DacFramework.msi** ファイルのダウンロードが完了したら、これを使用し、既定の設定で Microsoft SQL サーバー データ層アプリケーション フレームワークのインストールを実行します。

#### <a name="task-2-configure-a-release-pipeline"></a>タスク 2:リリース パイプラインを構成する

このタスクでは、リリース パイプラインを構成します。

> **注**:Azure VM には、アプリケーションをデプロイして Selenium テストケースを実行するよう構成されたエージェントが含まれています。 リリース定義は **[フレーズ](https://docs.microsoft.com/en-us/vsts/build-release/concepts/process/phases)** を使用してターゲット サーバーにデプロイします。

1.  **az40011bvm** へのリモート デスクトップ セッション内の **Azure DevOps** ポータルを表示しているブラウザー ウィンドウで、左上コーナーにある **Azure DevOps** のシンボルをクリックします。
1.  組織のプロジェクトを表示しているペインで、 **[機能テストの設定と実行]** プロジェクトを示しているタイルをクリックします。
1.  **[機能テストの設定と実行]** ペインの垂直ナビゲーション ペインで **[Pipelines]** を選択します。 **[Pipelines]** セクション内で **[リリース]** をクリックし、 **[Selenium]** ペインで **[編集]** をクリックします。
1.  **「すべてのパイプライン」 > 「Selenium」** ペインで「**タスク**」タブのヘッダーをクリックし、ドロップダウン メニューで「**Dev**」をクリックします。
1.  「**Dev**」ステージのタスク リスト内で「**IIS デプロイ**」、「**SQL デプロイ**」、「**Selenium テスト実行**」デプロイ フェーズをレビューします。 

- **IIS デプロイ フェーズ**:このフェーズでは、以下のタスクを使用してアプリケーションを VM にデプロイします:

   - **IIS Web アプリ管理**:このタスクは、エージェントを登録したターゲット マシンで実行します。 これにより、"*Web サイト*" と "*アプリケーション プール*" がローカルで作成されます。**PartsUnlimited** という名前で、ポート **82**、[ **http://localhost:82** ](http://localhost:82) で実行されます。
   - **IIS Web アプリ デプロイ**:このタスクでは、**Web Deploy** を使用してアプリケーションを ISS サーバーにデプロイします。

- **データベース デプロイ フェーズ**:このフェーズでは、[**SQL Server データベースのデプロイ**](https://github.com/Microsoft/vsts-tasks/blob/master/Tasks/SqlDacpacDeploymentOnMachineGroup/README.md) タスクを使用して、[**dacpac**](https://docs.microsoft.com/en-us/sql/relational-databases/data-tier-applications/data-tier-applications) ファイルを DB サーバーにデプロイします。

- **Selenium のテストの実行**:リリース プロセスの一環として **UI テスト** を実行すると、予期しない変化を検出できます。 自動化されたブラウザー ベースのテストを設定すると、手動でなくてもアプリケーションの品質を高められます。 このフェーズでは、デプロイされた Web アプリケーションで Selenium テストを実行します。 その後のタスクでは、Selenium を使用したリリース パイプライン での Web サイトのテストについて説明します。

  - **Visual Studio テスト プラットフォーム インストーラー**:[Visual Studio テスト プラットフォーム インストーラー](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/tool/vstest-platform-tool-installer?view=vsts) タスクでは、nuget.org または指定されたフィードから Microsoft テスト プラットフォームを取得し、ツール キャッシュに追加します。 これは **vstest** 要件を満たすため、ビルドまたはリリース パイプラインにおけるその後の Visual Studio テスト タスクは、エージェント マシンに完全な Visual Studio がインストールされていなくても実行できます。
  - **Selenium UI テストの実行**:この [タスク](https://github.com/Microsoft/azure-pipelines-tasks/blob/master/Tasks/VsTestV2/README.md)では、**vstest.console.exe** を使用してエージェント マシンで Selenium テストケースを実行します。

1.  **「すべてのパイプライン」 > 「Selenium」** ペインで **「IIS デプロイ**」フェーズをクリックし、「**エージェント ジョブ**」ペインで「**既定**」エージェント プールが選択されていることを確認します。 
1.  **SQL デプロイ** と **Selenium テスト実行** フェーズでも以前のステップを繰り返します。 必要であれば、「**保存**」をクリックして変更内容を保存します。

#### <a name="task-3-trigger-build-and-release"></a>タスク 3:ビルドとリリースをトリガーする

このタスクでは、**ビルド** をトリガーして Selenium C# スクリプトと Web 亜プいrケーションをコンパイルします。 この結果生じるバイナリはセルフホステッド エージェントにコピーされ、Selenium スクリプトは自動 **リリース** の一般として実行されます。

1.  **az40011bvm** へのリモート デスクトップ セッション内で、**Azure DevOps** ポータルを表示しているブラウザー ウィンドウの垂直ナビゲーション ペインにある「**パイプライン**」セクションで「**パイプライン**」をクリックした後、「**パイプライン**」ペインで「**Selenium**」をクリックします。
1.  「**Selenium**」ペインで「**パイプラインの実行**」をクリックし、「**パイプラインの実行**」で「**実行**」をクリックします。

    > **注**:このビルドは、テスト成果物を Azure DevOps に公開します。これはリリースで使用されます。 

    > **注**:ビルドが成功すると、リリースがトリガーされます。 

1.  パイプライン実行ペインの「**ジョブ**」セクションで「**フェーズ 1**」をクリックし、完了するまでビルドの進捗状況を監視します。
1.  **Azure DevOps** ポータルを表示しているブラウザー ウィンドウの垂直ナビゲーション ペインにある「**パイプライン**」セクションで「**リリース**」をクリックします。リリースを示すエントリーをクリックし、 **「Selenium」> 「Release-1」** ペインで「**Dev**」をクリックします。
1.  **「Selenium」 > 「Release-1」 > 「Dev」** ペインで、該当するデプロイを監視します。
1.  **Selenium テスト実行** フェーズが始まったら、Web ブラウザー テストを監視します。 
1.  リリースの完了後、 **「Selenium」 > 「Release-1」 > 「Dev」** ペインで「**テスト**」タブをクリックしてテスト結果を分析します。 「**結果**」セクションのドロップダウンから必要なフィルターを選択し、テストとそのステータスを表示します。

### <a name="exercise-2-remove-the-azure-lab-resources"></a>演習 2:Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

>**注**:新規に作成し、使用しなくなったすべての Azure リソースを削除することを忘れないでください。 使用していないリソースを削除することで、予期しない料金が発生しなくなります。

#### <a name="task-1-remove-the-azure-lab-resources"></a>タスク 1:Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'az400m11l02-RG')].name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体を通して作成したすべてのリソース グループを削除します。

    ```sh
    az group list --query "[?starts_with(name,'az400m11l02-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注**:コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。


## <a name="review"></a>確認

このラボでは、Azure DevOps リリース パイプラインの一部として、C# Web アプリケーションで Selenium テスト ケースを実行する方法を学習しました。
