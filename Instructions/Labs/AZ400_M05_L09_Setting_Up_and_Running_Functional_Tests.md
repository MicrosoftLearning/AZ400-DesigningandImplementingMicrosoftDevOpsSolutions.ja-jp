---
lab:
  title: 機能テストの設定と実行
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
---

# 機能テストの設定と実行

## 受講生用ラボ マニュアル

## ラボの要件

- このラボには、**Microsoft Edge** または [Azure DevOps 対応ブラウザー](https://docs.microsoft.com/azure/devops/server/compatibility?view=azure-devops)が必要です。

- **Azure DevOps 組織を設定する:** このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization)に関するページの手順に従って作成してください。

- 既存の Azure サブスクリプションを識別するか、新しいものを作成します。

- Azure サブスクリプションでは所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントではグローバル管理者のロールで Microsoft アカウントまたは Azure AD アカウントがあることを確認します。 詳細については、[「Azure portal を使用して Azure ロールの割り当てを一覧表示する」](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal)および[「Azure Active Directory で管理者ロールを表示して割当てる」](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal)を参照してください。

## ラボの概要

[Selenium](http://www.seleniumhq.org/) は、Web アプリケーション用のポータブル オープン ソース ソフトウェア テスト フレームワークです。 ほとんどすべてのオペレーティング システムで動作できます。 これは、すべての最新のブラウザー、.NET (C#) や Java などの複数の言語をサポートしています。

このラボでは、Azure DevOps リリース パイプラインの一部として、C# Web アプリケーションで Selenium テスト ケースを実行する方法を学習します。

## 目標

このラボを完了すると、次のことができるようになります。

- セルフホステッド Azure DevOps エージェントを構成する。
- リリース パイプラインを構成する。
- ビルドとリリースをトリガーする。
- Chrome と Firefox でテストを実行する。

## 推定時間:60 分

## Instructions

### 演習 0:ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。Azure DevOps Demo Generator テンプレートと Azure リソースに基づいて事前に構成された Parts Unlimited チーム プロジェクトが含まれます。

#### タスク 1:チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用して、**Selenium** テンプレートに基づいて新しいプロジェクトを生成します。

1. ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。 このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。

    > **注**: このサイトの詳細については、「[Azure DevOps Services Demo Generator とは](https://docs.microsoft.com/azure/devops/demo-gen)」を参照してください。

2. **[サインイン]** をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。
3. 必要な場合は、**[Azure DevOps Demo Generator]** ページで **[承諾する]** をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。
4. **新しいプロジェクトの作成**ページで **[新しいプロジェクト名]** テキストボックスに「**機能テストの設定と実行**」と入力します。**[組織の選択]** ドロップダウン リストで Azure DevOps 組織を選択し、**[テンプレートの選択]** をクリックします。
5. テンプレートの一覧のツールバーで、**[DevOps ラボ]** をクリックし、**[Selenium]** テンプレートを選択して、**[テンプレートの選択]** をクリックします。
6. もう一度、**[新しいプロジェクトの作成]** ページで **[プロジェクトの作成]** をクリックします

    > **注**:プロセスが完了するまでお待ちください。 これには 2 分ほどかかります。 プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

7. 「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

#### タスク 2: Azure リソースを作成する

このタスクでは、Windows Server 2016 のほか、SQL Express 2017、Chrome、Firefox を実行している Azure VM をプロビジョニングします。

1. **[[Azure にデプロイ]](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2Falmvm%2Fmaster%2Flabs%2Fvstsextend%2Fselenium%2Farmtemplate%2Fazuredeploy.json)** リンクをクリックします。 これで、自動的に Azure portal の **[カスタム デプロイ]** ブレードにリダイレクトされます。
2. 指示されたら、このラボで使用する Azure サブスクリプションで所有者のロールがあり、このサブスクリプションに関連のある Azure AD テナントでグローバル管理者のロールがあるユーザー アカウントを使用してサインインします。
3. **[カスタム デプロイ]** ブレードで、**[パラメーターの編集]** を選択します。
4. **[テンプレートの編集]** ブレードで、行 `"https://raw.githubusercontent.com/microsoft/azuredevopslabs/master/labs/vstsextend/selenium/armtemplate/chrome_firefox_VSTSagent_IIS.ps1"` を探して `"https://raw.githubusercontent.com/MicrosoftLearning/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/master/Allfiles/Labs/11b/chrome_firefox_VSTSagent_IIS.ps1"` に置き換え、 **[保存]** をクリックします。
5. **[カスタム デプロイ]** ブレードに戻って、次の設定を指定します。

    | 設定 | 値 |
    | --- | --- |
    | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
    | リソース グループ | 新しいリソース グループの名前 **az400m11l02-RG** |
    | リージョン | このラボで Azure リソースをデプロイする Azure リージョンの名前 |
    | 仮想マシン名 | **az40011bvm** |

6. **[Review + create](レビュー + 作成)** をクリックし、 **[作成]** をクリックします。

    > **注**:プロセスが完了するまでお待ちください。 通常は 15 分ほどかかります。

### 演習 1:セルフホステッド Azure DevOps エージェントを使用して Selenium テストを実装する

この演習では、セルフホステッド Azure DevOps エージェントを使用して Selenium テストを実装します。

#### タスク 1:セルフホステッド Azure DevOps エージェントを構成する

このタスクでは、以前の演習でデプロイした VM を使用してセルフホステッド エージェントを構成します。 Selenium では、インタラクティブ モードでエージェントを実行し、UI テストを実行する必要があります。

1. Azure portal が表示されている Web ブラウザー ウィンドウで「**仮想マシン**」を検索して選択し、**[仮想マシン]** ブレードで **[az40011bvm]** を選択します。
2. **[az40011bvm]** ブレードで、 **[接続]** を選択し、ドロップダウン メニューの **[RDP]** を選択します。 **[az40011bvm] \| [接続]** ブレードの **[RDP]** タブで、 **[RDP ファイルのダウンロード]** を選択し、ダウンロードしたファイルを開きます。
3. プロンプトが表示されたら、次の認証情報を入力します。

    | 設定 | 値 |
    | --- | --- |
    | [ユーザー名] | **vmadmin** |
    | Password | **P2ssw0rd@123** |

4. **az40011bvm** へのリモート デスクトップ セッション内で Chrome Web ブラウザー ウィンドウを開き、 **<https://dev.azure.com>** に移動して Azure DevOps 組織にサインインします。
5. **Azure DevOps** ポータルの左下隅で、**[組織の設定]** をクリックします。
6. ページの左側にある垂直メニューの **[パイプライン]** セクションで、**[エージェント プール]** をクリックします。
7. **[エージェント プール]** ペインで **[既定]** をクリックします。
8. **[既定]** ペインで **[新しいエージェント]** をクリックします。
9. **[エージェントの取得]** パネルで、**[Windows]** タブと **[x64]** セクションが選択されていることを確認してから、**[ダウンロード]** をクリックします。
10. エクスプローラーを起動して **C:\\AzAgent** ディレクトリを作成し、**ダウンロード** フォルダーにあるダウンロード済みのエージェント Zip ファイルの内容を展開します。
11. **[az40011bvm]** へのリモート デスクトップ セッション内で、**[起動]** メニューを右クリックし、**[コマンド プロンプト (管理者)]** をクリックします。
12. **管理者:コマンド プロンプト**ウィンドウで、以下を実行してエージェント バイナリのインストールを開始します。

    ```cmd
    cd C:\AzAgent
    Config.cmd
    ```

13. **[管理者:コマンド プロンプト]** ウィンドウで、 **[Enter server URL]** (サーバーの URL を入力します) と表示されたら **https://dev.azure.com/\<your-DevOps-organization-name\>** と入力します。ここで **\<your-DevOps-organization-name\>** は、Azure DevOps 組織を表します。次に **Enter** キーを押します。
14. **[管理者:コマンド プロンプト]** ウィンドウで、 **[Enter Authentication type (press enter for PAT)]** (認証タイプを入力します (PAT の場合は Enter を押します)) と表示されたら**Enter キー**を押します。
15. **[管理者:コマンド プロンプト]** ウィンドウで、 **[Enter personal access token]** (個人用アクセス トークンを入力します) と表示されたら、Azure DevOps ポータルに切り替え、 **[エージェントの取得]** パネルを閉じ、Azure DevOps ページの右上隅で **[ユーザー設定]** アイコンをクリックします。ドロップダウン メニューで **[個人用アクセス トークン]** をクリックし、 **[個人用アクセス トークン]** ペインで **[+ 新しいトークン]** をクリックします。
16. **[新しい個人用アクセス トークンの作成]** ペインで次の設定を指定して、**[作成]** をクリックします (他の設定はすべて既定値のままにします)。

    | 設定 | 値 |
    | --- | --- |
    | 名前 | **機能テストの設定と実行ラボ** |
    | スコープ | **カスタム定義済み** |
    | スコープ | ウィンドウの下部にある **[すべてのスコープを表示]** をクリックします |
    | スコープ | **エージェント プール** - **読み取りと管理** |

17. **[成功]** ペインで、個人用アクセス トークンの値をクリップボードにコピーします。

    > **注**:必ずトークンをコピーしてください。 このペインを閉じると、取得できなくなります。

18. **[成功]** ペインで **[閉じる]** をクリックします。
19. **管理者:コマンド プロンプト** ウィンドウに戻り、クリップボードの内容を貼り付けてから **Enter** キーを押します。
20. **[管理者:コマンド プロンプト]** ウィンドウで、 **[Enter agent pool (press enter for default)]** (エージェント プールを入力します (既定を使用する場合は Enter キーを押します)) と表示されたら **Enter キー**を押します。
21. **[管理者:コマンド プロンプト]** ウィンドウで、 **[Enter agent name (press enter for az40011bvm)]** (エージェント名を入力します(az40011bvm の場合は Enter キーを押します)) と表示されたら **Enter キー**を押します。
22. **[管理者:コマンド プロンプト]** ウィンドウで、 **[Enter work folder (press enter for _work)]** (作業フォルダーを入力します(_work の場合は Enter キーを押します)) と表示されたら **Enter キー**を押します。
23. **[管理者:コマンド プロンプト]** ウィンドウで、 **[Enter run agent as service (Y/N) (press enter for N)]** (サービスとしてのエージェントの実行を入力します(はい/いいえ) いいえの場合は Enter キーを押します)) と表示されたら **Enter キー**を押します。
24. **[管理者:コマンド プロンプト]** ウィンドウで、 **[Enter configure autologon and run agent on startup (Y/N) (press enter for N)]** (起動時の自動ログオンの構成とエージェントの実行を入力します(はい/いいえ) いいえの場合は Enter キーを押します)) と表示されたら **Enter キー**を押します。
25. エージェントが登録されたら **[管理者: コマンド プロンプト]** ウィンドウに、「**run.cmd**」と入力し、**Enter** キーを押して、エージェントを起動します。

    > **注**:Dac Framework もインストールする必要があります。ラボで後ほどデプロイするアプリケーションで使用されます。

26. **az40011bvm** へのリモート デスクトップ セッション内で、Web ブラウザーの別のインスタンスを起動し、[Microsoft SQL Server データ層アプリケーションフレームワーク (18.2) ダウンロード ページ](https://www.microsoft.com/download/details.aspx?id=58207&WT.mc_id=rss_alldownloads_extensions)に移動して、**[ダウンロード]** をクリックします。
27. **[ダウンロード先フォルダーの選択]** で、**[EN\x64\DacFramework.msi]** チェックボックスを選択し、**[次へ]** をクリックします。 これにより、**DacFramework.msi** ファイルの自動ダウンロードがトリガーされます。
28. **DacFramework.msi** ファイルのダウンロードが完了したら、これを使用し、既定の設定で Microsoft SQL サーバー データ層アプリケーション フレームワークのインストールを実行します。

#### タスク 2:リリース パイプラインを構成する

このタスクでは、リリース パイプラインを構成します。

> **注**: Azure VM には、アプリケーションをデプロイして Selenium テスト ケースを実行するよう構成されたエージェントが含まれています。 リリース定義は **[フレーズ](https://docs.microsoft.com/vsts/build-release/concepts/process/phases)** を使用してターゲット サーバーにデプロイします。

1. **az40011bvm** へのリモート デスクトップ セッション内の **Azure DevOps** ポータルを表示しているブラウザー ウィンドウで、左上コーナーにある **Azure DevOps** のシンボルをクリックします。
2. 組織のプロジェクトを表示しているペインで、 **[機能テストの設定と実行]** プロジェクトを示しているタイルをクリックします。
3. **[機能テストの設定と実行]** ペインの垂直ナビゲーション ペインで **[Pipelines]** を選択します。 **[Pipelines]** セクション内で **[リリース]** をクリックし、 **[Selenium]** ペインで **[編集]** をクリックします。
4. **[すべてのパイプライン] > [Selenium]** ペインで **[タスク]** タブのヘッダーをクリックし、ドロップダウン メニューで **[Dev]** をクリックします。
5. **[Dev]** ステージのタスクの一覧内で、**[IIS デプロイ]**、**[SQL デプロイ]**、**[Selenium のテストの実行]** デプロイ フェーズを確認します。

   - **IIS デプロイ フェーズ**:このフェーズでは、以下のタスクを使用してアプリケーションを VM にデプロイします:

     - **IIS Web アプリ管理**:このタスクは、エージェントを登録したターゲット マシンで実行します。 これにより、"*Web サイト*" と "*アプリケーション プール*" がローカルで作成されます。**PartsUnlimited** という名前で、ポート **82**、[ **http://localhost:82** ](http://localhost:82) で実行されます。
     - **IIS Web アプリ デプロイ**:このタスクでは、**Web Deploy** を使用してアプリケーションを ISS サーバーにデプロイします。

   - **データベース デプロイ フェーズ**:このフェーズでは、[**SQL Server データベースのデプロイ**](https://github.com/Microsoft/vsts-tasks/blob/master/Tasks/SqlDacpacDeploymentOnMachineGroup/README.md) タスクを使用して、[**dacpac**](https://docs.microsoft.com/sql/relational-databases/data-tier-applications/data-tier-applications) ファイルを DB サーバーにデプロイします。

   - **Selenium のテストの実行**:リリース プロセスの一環として **UI テスト** を実行すると、予期しない変化を検出できます。 自動化されたブラウザー ベースのテストを設定すると、手動でなくてもアプリケーションの品質を高められます。 このフェーズでは、デプロイされた Web アプリケーションで Selenium テストを実行します。 その後のタスクでは、Selenium を使用したリリース パイプライン での Web サイトのテストについて説明します。

     - **Visual Studio テスト プラットフォーム インストーラー**:[Visual Studio テスト プラットフォーム インストーラー](https://docs.microsoft.com/azure/devops/pipelines/tasks/tool/vstest-platform-tool-installer?view=vsts) タスクでは、nuget.org または指定されたフィードから Microsoft テスト プラットフォームを取得し、ツール キャッシュに追加します。 これは **vstest** 要件を満たすため、ビルドまたはリリース パイプラインにおけるその後の Visual Studio テスト タスクは、エージェント マシンに完全な Visual Studio がインストールされていなくても実行できます。
     - **Selenium UI テストの実行**: この[タスク](https://github.com/Microsoft/azure-pipelines-tasks/blob/master/Tasks/VsTestV2/README.md)では、**vstest.console.exe** を使ってエージェント マシンで Selenium テスト ケースを実行します。

6. **[すべてのパイプライン] > [Selenium]** ペインで **[IIS デプロイ]** フェーズをクリックし、**[エージェント ジョブ]** ペインで **[既定]** エージェント プールが選択されていることを確認します。
7. **SQL デプロイ**と **Selenium テスト実行**フェーズでも以前のステップを繰り返します。 必要であれば、**[保存]** をクリックして変更内容を保存します。

#### タスク 3:ビルドとリリースをトリガーする

このタスクでは、**ビルド**をトリガーして Selenium C# スクリプトと Web 亜プいrケーションをコンパイルします。 この結果生じるバイナリはセルフホステッド エージェントにコピーされ、Selenium スクリプトは自動**リリース**の一般として実行されます。

1. **az40011bvm** へのリモート デスクトップ セッション内で、**Azure DevOps** ポータルを表示しているブラウザー ウィンドウの垂直のナビゲーション ウィンドウにある **[パイプライン]** セクションで **[パイプライン]** をクリックした後、**[パイプライン]** ペインで **[Selenium]** をクリックします。
2. **[Selenium]** ペインで **[パイプラインの実行]** をクリックし、**[パイプラインの実行]** で **[実行]** をクリックします。

    > **注**:このビルドは、テスト成果物を Azure DevOps に公開します。これはリリースで使用されます。

    > **注**:ビルドが成功すると、リリースがトリガーされます。

3. [パイプライン実行] ペインの **[ジョブ]** セクションで、**[フェーズ 1]** をクリックし、完了するまでビルドの進捗状況を監視します。
4. **Azure DevOps** が表示されているブラウザー ウィンドウの垂直のナビゲーション ウィンドウの **[パイプライン]** セクションで、**[リリース]** をクリックし、リリースを示すエントリーをクリックして、**[Selenium] > [Release-1]** ペインで **[Dev]** をクリックします。
5. **[Selenium] > [Release-1] > [Dev]** ペインで、該当するデプロイを監視します。
6. **Selenium テスト実行**フェーズが始まったら、Web ブラウザー テストを監視します。
7. リリースの完了後、**[Selenium] > [Release-1] > [Dev]** ペインで **[テスト]** タブをクリックして、テスト結果を分析します。 **[結果]** セクションのドロップダウンから必要なフィルターを選択し、テストとその状態を表示します。

### 演習 2:Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。

>**注**:新規に作成し、使用しなくなったすべての Azure リソースを削除することを忘れないでください。 使用していないリソースを削除することで、予期しない料金が発生しなくなります。

#### タスク 1:Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。

1. Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
2. 次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'az400m11l02-RG')].name" --output tsv
    ```

3. 次のコマンドを実行して、このモジュールのラボ全体を通して作成したすべてのリソース グループを削除します。

    ```sh
    az group list --query "[?starts_with(name,'az400m11l02-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注**:コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## 確認

このラボでは、Azure DevOps リリース パイプラインの一部として、C# Web アプリケーションで Selenium テスト ケースを実行する方法を学習しました。
