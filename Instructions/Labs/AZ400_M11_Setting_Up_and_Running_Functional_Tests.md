---
lab:
    title: 'ラボ 11b: 機能テストの設定と実行'
    module: 'モジュール 11: Azure Pipelines を使用した継続的デプロイの実装'
---

# ラボ 11b: 機能テストの設定と実行
# 受講生用ラボ マニュアル

## ラボの概要

[Selenium](http://www.seleniumhq.org/) は、Web アプリケーション用のポータブル オープン ソース ソフトウェア テスト フレームワークです。ほとんどすべてのオペレーティングシステムで動作する機能があります。これは、すべての最新のブラウザと .NET (C#)、Java を含む複数の言語をサポートしています。

このラボでは、Azure DevOps リリース パイプラインの一部として、C# Web アプリケーションで Selenium テスト ケースを実行する方法について説明します。 

## 目標

このラボを完了すると、次のことができるようになります。

- セルフホステッド Azure DevOps エージェントを構成する
- リリース パイプラインを構成する
- ビルドとリリースをトリガーする
- Chrome と Firefox でテストを実行する

## ラボの所要時間

-   推定時間: **60 分**

## 手順

### 開始する前に

#### ラボの仮想マシンにログインする

次の認証情報を使用して、Windows 10 仮想マシンにサインインしていることを確認します。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### このラボで必要なアプリケーションのレビュー

このラボで使用するアプリケーションを特定:
    
-   Microsoft Edge

#### Azure DevOps 組織を設定します。 

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/ja-jp/azure/devops/organizations/accounts/create-organization?view=azure-devops)で利用できる手順に従って作成してください。

#### Azure サブスクリプションの準備

-   既存の Azure サブスクリプションを識別するか、新しいものを作成します。
-   Azure サブスクリプションでは所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントではグローバル管理者のロールで Microsoft アカウントまたは Azure AD アカウントがあることを確認します。詳細については、[Azure portal を使用して Azure ロールの割り当てを一覧表示する](https://docs.microsoft.com/ja-jp/azure/role-based-access-control/role-assignments-list-portal) および [Azure Active Directory で管理者ロールを表示して割当てる](https://docs.microsoft.com/ja-jp/azure/active-directory/roles/manage-roles-portal#view-my-roles) を参照してください。

### 演習 0: ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。Azure DevOps Demo Generator テンプレートと Azure リソースに基づいて事前に構成された Parts Unlimited チーム プロジェクトが含まれます。 

#### タスク 1: チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用して、**Selenium** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**: このサイトの詳細については、「[Azure DevOps Services Demo Generator とは](https://docs.microsoft.com/ja-jp/azure/devops/demo-gen)」を参照してください。

1.  「**サインイン**」をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。
1.  必要な場合は、「**Azure DevOps Demo Generator**」ページで「**承諾する**」をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。
1.  「**新しいプロジェクトの作成**」 ページで 「**新しいプロジェクト名**」 テキストボックスに「**Setting Up and Running Functional Tests**」と入力します。「**組織の選択**」 ドロップダウン リストで Azure DevOps 組織を選択し、「**テンプレートの選択**」 をクリックします。
1.  テンプレートのリストのツールバーで 「**DevOps Labs**」 をクリックし、「**Selenium**」 テンプレートを選択して 「**テンプレートの選択**」 をクリックします。
1.  再び「**新しいプロジェクトの作成**」ページで「**プロジェクトの作成**」をクリックします。

    > **注**: プロセスが完了するまでお待ちください。これにはおよそ 2 分かかる場合があります。プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

#### タスク 2: Azure リソースを作成する

このタスクでは、Windows Server 2016 のほか、SQL Express 2017、Chrome、Firefox を実行している Azure VM をプロビジョニングします。

1.  「**[Azure にデプロイ](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2Falmvm%2Fmaster%2Flabs%2Fvstsextend%2Fselenium%2Farmtemplate%2Fazuredeploy.json)**」リンクをクリックします。これで自動的に Azure portal の「**カスタム デプロイ**」ブレードにリダイレクトされます。
1.  指示されたら、このラボで使用する Azure サブスクリプションで所有者のロールがあり、このサブスクリプションに関連のある Azure AD テナントでグローバル管理者のロールがあるユーザー アカウントを使用してサインインします。
1.  「**カスタム デプロイ**」ブレードで、「**パラメーターの編集**」を選択します。
1.  「**テンプレートの編集**」ブレードで、行 `"https://raw.githubusercontent.com/microsoft/azuredevopslabs/master/labs/vstsextend/selenium/armtemplate/chrome_firefox_VSTSagent_IIS.ps1"` を見つけ、`https://raw.githubusercontent.com/MicrosoftLearning/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/master/Allfiles/Labs/11b/chrome_firefox_VSTSagent_IIS.ps1` に置き換え、「**保存**」をクリックします。
1.  「**カスタム デプロイ**」ブレードに戻って、次の設定を指定します。

    | 設定 | 値 |
    | --- | --- |
    | サブスクリプション | このラボで使用する Azure サブスクリプションの名前 |
    | リソース グループ | 新しいリソース グループ「**az400m11l02-RG**」の名前 |
    | リージョン | このラボで Azure リソースをデプロイする Azure リージョンの名前 |
    | 仮想マシン名 | **az40011bvm** |

1.  **「確認と作成」** をクリックし、**「作成」** をクリックします。 

    > **注**: プロセスが完了するまでお待ちください。これにはおよそ 15 分かかる場合があります。 

### 演習 1: セルフホステッド Azure DevOps エージェントを使用して Selenium テストを実装する

この演習では、セルフホステッド Azure DevOps エージェントを使用して Selenium テストを実装します。

#### タスク 1: セルフホステッド Azure DevOps エージェントを構成する

このタスクでは、以前の演習でデプロイした VM を使用してセルフホステッド エージェントを構成します。Selenium では、インタラクティブ モードでエージェントを実行し、UI テストを実行する必要があります。

1.  Azure portal が表示されている Web ブラウザー ウィンドウで 「**仮想マシン**」 を検索して選択し、「**仮想マシン**」 ブレードで 「**az40011bvm**」 を選択します。
1.  「**az40011bvm**」 ブレードで 「**接続**」 を選択し、「**az40011bvm**」 の 「**RDP**」 タブのドロップダウン メニューで 「**RDP**」 を選択します。「**接続**」 ブレードで 「**RDP ファイルのダウンロード**」 を選択し、ダウンロードされたファイルを開きます。
1.  プロンプトが表示されたら、次の認証情報を入力します。

    | 設定 | 値 |
    | --- | --- |
    | ユーザー名 | **vmadmin** |
    | パスワード | **P2ssw0rd@123** |

1.  **az40011bvm** へのリモート デスクロップ セッション内で Chrome Web ブラウザー ウィンドウを開き、**https://dev.azure.com** に移動して Azure DevOps 組織にサインインします。ログインしている Azure AD ディレクトリに注意してください。 
1.  **Azure DevOps** ポータルの左下コーナーで、「**組織の設定**」 をクリックします。
1.  ページの左側にある垂直メニューの 「**パイプライン**」 セクションで 「**エージェント プール**」 をクリックします。
1.  「**エージェント プール**」 ペインで 「**Default**」 をクリックします。 
1.  「**既定**」 ペインで 「**New Agent**」 をクリックします。 
1.  「**エージェントの取得**」 パネルで、「**Windows**」 タブと 「**x64**」 セクションが選択されていることを確認してから、「**ダウンロード**」 をクリックします。
1.  エクスプローラーを起動して「**C:\\AzAgent**」ディレクトリを作成し、「**ダウンロード**」 フォルダーにあるダウンロード済みのエージェント zip ファイルの内容を展開します。
1.  「**az40011bvm**」 へのリモート デスクトップ セッション内で、「**コマンド プロンプト (管理者)**」 をクリックします。
1.  「**管理者: コマンド プロンプト**」 ウィンドウ内で、以下を実行してエージェント バイナリのインストールを起動します。

    ```cmd
    cd C:\AzAgent
    Config.cmd
    ```
1.  「**管理者: コマンド プロンプト**」ウィンドウで、「**サーバーの URL を入力**」するよう指示されたら「**https://dev.azure.com/\<your-DevOps-organization-name\>**」を入力します。ここで、「**\<your-DevOps-organization-name\>**」は Azure DevOps 組織の名前を示します。**Enter** キーを押します。
1.  「**管理者: コマンド プロンプト**」 ウィンドウで、「**認証タイプを入力する (PAT の場合は Enter を押す)**」 よう指示されたら **Enter キー**を押します。
1.  「**管理者: コマンド プロンプト**」 ウィンドウで、「**個人用アクセス トークンを入力**」 するよう指示されたら Azure DevOps ポータルに切り替えます。「**エージェントの取得**」 パネルを閉じ、Azure DevOps ページの右上コーナーで 「**ユーザー設定**」 アイコンをクリックします。ドロップダウン メニューで 「**個人アクセス トークン**」 、「**個人用アクセス トークン**」 ペインで 「**+ 新しいトークン**」 をクリックします。
1.  「**新しい個人用アクセス トークンの作成**」 ペインで以下の設定を指定し、「**作成**」 をクリックします (他はすべて規定値のままにします):

    | 設定 | 値 |
    | --- | --- |
    | 名前 | **機能テストの設定と実行ラボ** |
    | スコープ | **Custom Defined** |
    | スコープ | ウィンドウの下部にある「**Show all scopes**」をクリックする |
    | スコープ | **エージェント プール** - **読み取りと管理** |

1.  「**成功**」ペインで個人用アクセス トークンの値をクリップボードにコピーします。

    > **注**: 必ずトークンをコピーしてください。このペインを閉じると、取得できなくなります。 

1.  「**成功**」ペインで「**閉じる**」をクリックします。
1.  「**管理者: コマンド プロンプト**」 ウィンドウに戻り、クリップボードの内容を貼り付けてから **Enter キー**を押します。
1.  「**管理者: コマンド プロンプト**」 ウィンドウで、「**Enter Agent Pool (既定の場合は Enter を押す)**」 よう指示されたら **Enter キー**を押します。
1.  「**管理者: コマンド プロンプト**」 ウィンドウで、「**Enter Agent Name (az40011bvm の場合は Enter を押す)**」 よう指示されたら **Enter キー**を押します。
1.  「**管理者: コマンド プロンプト**」 ウィンドウで、「**Enter Work Folder (_work の場合は Enter を押す)**」 よう指示されたら **Enter キー**を押します。
1.  「**管理者: コマンド プロンプト**」 ウィンドウで、「**Enter run agent as service (Y/N) (N の場合は Enter を押す)**」 よう指示されたら **Enter キー**を押します。
1.  「**管理者: コマンド プロンプト**」 ウィンドウで、「**Enter configure autologon and run agent on startup? (Y/N) (N の場合は Enter を押す)**」 よう指示されたら **Enter キー**を押します。
1.  エージェントが登録された後、「**管理者: コマンド プロンプト**」 ウィンドウで「**run.cmd**」と入力し、**Enter** を押してエージェントを起動します。

    > **注**: Dac Framework もインストールする必要があります。ラボで後ほどデプロイするアプリケーションで使用されます。

1.  **az40011bvm** へのリモート デスクトップ セッション内で、Web ブラウザーの別のインスタンスを起動し、[Microsoft SQL Serverデータ層アプリケーションフレームワーク (18.2) ダウンロード ページ](https://www.microsoft.com/ja-jp/download/details.aspx?id=58207&WT.mc_id=rss_alldownloads_extensions)に移動して、「**ダウンロード**」をクリックします。 
1.  「**必要なダウンロードの選択**」で、「**EN\x64\DacFramework.msi**」チェックボックスを選択し、「**次へ**」をクリックします。これにより、**DacFramework.msi** ファイルの自動ダウンロードがトリガーされます。
1.  **DacFramework.msi** ファイルのダウンロードが完了したら、これを使用し、既定の設定で Microsoft SQL サーバー データ層アプリケーション フレームワークのインストールを実行します。

#### タスク 2: リリース パイプラインを構成する

このタスクでは、リリース パイプラインを構成します。

> **注**: Azure VM には、アプリケーションをデプロイして Selenium テストケースを実行するよう構成されたエージェントが含まれています。リリース定義は **[フレーズ](https://docs.microsoft.com/ja-jp/vsts/build-release/concepts/process/phases)** を使用してターゲット サーバーにデプロイします。

1.  **az40011bvm** へのリモート デスクトップ セッション内の **Azure DevOps** ポータルを表示しているブラウザー ウィンドウで、左上コーナーにある **Azure DevOps** のシンボルをクリックします。
1.  組織のプロジェクトを表示しているペインで、「**Setting Up and Running Functional Tests**」 プロジェクトを示しているタイルをクリックします。
1.  「**パイプライン**」 を選択します。「**パイプライン**」 セクション内で 「**リリース**」 をクリックし、「**Selenium**」 ペインで 「**Edit**」 をクリックします。
1.  「**Tasks**」 タブをクリックし、ドロップダウン メニューで 「**Dev**」 をクリックします。
1.  「**Dev**」 ステージのタスク リスト内で 「**IIS Deployment**」、「**SQL Deployment**」、「**Selenium tests execution**」 デプロイ フェーズをレビューします。 

- **IIS Deployment フェーズ**: このフェーズでは、以下のタスクを使用してアプリケーションを VM にデプロイします:

   - **IIS Web App Manage**: このタスクは、エージェントを登録したターゲット マシンで実行します。*Web サイト*と*アプリケーション プール*がローカルで作成されます。「**PartsUnlimited**」という名前で、ポート **82**、[**http://localhost:82**](http://localhost:82) で実行されます。
   - **Deploy IIS Website/App**: このタスクでは、**Web Deploy** を使用してアプリケーションを ISS サーバーにデプロイします。

- **DB Deployment フェーズ**: このフェーズでは、[**SQL サーバー データベースのデプロイ**](https://github.com/Microsoft/vsts-tasks/blob/master/Tasks/SqlDacpacDeploymentOnMachineGroup/README.md) タスクを使用して、[**dacpac**](https://docs.microsoft.com/ja-jp/sql/relational-databases/data-tier-applications/data-tier-applications) ファイルを DB サーバーにデプロイします。

- **Selenium tests execution**: リリース プロセスの一環として **UI テスト** を実行すると、予期しない変化を検出できます。自動化されたブラウザー ベースのテストを設定すると、手動でなくてもアプリケーションの品質を高められます。このフェーズでは、デプロイされた Web アプリケーションで Selenium テストを実行します。その後のタスクでは、Selenium を使用したリリース パイプライン での Web サイトのテストについて説明します。

  - **Visual Studio Test Platform Installer**: [Visual Studio テスト プラットフォーム インストーラー](https://docs.microsoft.com/ja-jp/azure/devops/pipelines/tasks/tool/vstest-platform-tool-installer?view=vsts) タスクでは、nuget.org または指定されたフィードから Microsoft テスト プラットフォームを取得し、ツール キャッシュに追加します。これは **vstest** 要件を満たすため、ビルドまたはリリース パイプラインにおけるその後の Visual Studio テスト タスクは、エージェント マシンに完全な Visual Studio がインストールされていなくても実行できます。
  - **Run Selenium UI tests**: この[タスク](https://github.com/Microsoft/azure-pipelines-tasks/blob/master/Tasks/VsTestV2/README.md)では、**vstest.console.exe** を使用してエージェント マシンで Selenium テストケースを実行します。

1. **「IIS Deployment**」 フェーズをクリックし、「**Agent pool**」 ペインで 「**Default**」 エージェント プールが選択されていることを確認します。 
1.  **SQL Deployment**と **Selenium tests execution**フェーズでも以前のステップを繰り返します。必要であれば、「**保存**」 をクリックして変更内容を保存します。

#### タスク 3: ビルドとリリースをトリガーする

このタスクでは、**ビルド**をトリガーして Selenium C# スクリプトと Web アプリケーションをコンパイルします。この結果生じるバイナリはセルフホステッド エージェントにコピーされ、Selenium スクリプトは自動 **リリース** プロセスの中で実行されます。

1.  **az40011bvm** へのリモート デスクトップ セッション内で、**Azure DevOps** ポータルを表示しているブラウザー ウィンドウの垂直ナビゲーション ペインにある 「**パイプライン**」 セクションで 「**パイプライン**」 をクリックした後、「**パイプライン**」 ペインで 「**Selenium**」 をクリックします。
1.  「**Selenium**」 ペインで 「**パイプラインの実行**」 をクリックし、「**パイプラインの実行**」 で 「**実行**」 をクリックします。

    > **注**: このビルドは、テスト成果物を Azure DevOps に公開します。これはリリースで使用されます。 

    > **注**: ビルドが成功すると、リリースがトリガーされます。 

1.  パイプライン実行ペインの 「**ジョブ**」 セクションで 「**Phase 1**」 をクリックし、完了するまでビルドの進捗状況を監視します。
1.  **Azure DevOps** ポータルを表示しているブラウザー ウィンドウの垂直ナビゲーション ペインにある 「**パイプライン**」 セクションで 「**リリース**」 をクリックします。リリースを示すエントリーをクリックし、**Selenium > Release-1** ペインで 「**Dev**」 「**View Logs**」をクリックします。
1.  **Selenium > Release-1 > Dev** ペインで、該当するデプロイを監視します。
1.  **Selenium テスト実行**フェーズが始まったら、Web ブラウザー テストを監視します。 
1.  リリースの完了後、**「Selenium」 > 「Release-1」 > 「Dev」** ペインで 「**Test**」 タブをクリックしてテスト結果を分析します。「**Filter by test or run name**」 セクションの右側のドロップダウンでフィルターを選択し、テストとそのステータスを表示します。

### 演習 2: Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

>**注**: Azure リソースのうち、使用しないリソースは必ず削除してください。使用しないリソースを削除しないと、予期しないコストが発生する場合があります。

#### タスク 1: Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'az400m11l02-RG')].name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを削除します。

    ```sh
    az group list --query "[?starts_with(name,'az400m11l02-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注**: コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。


## レビュー

このラボでは、Azure DevOps リリース パイプラインの一部として、C# Web アプリケーションで Selenium テスト ケースを実行する方法を学習しました。
