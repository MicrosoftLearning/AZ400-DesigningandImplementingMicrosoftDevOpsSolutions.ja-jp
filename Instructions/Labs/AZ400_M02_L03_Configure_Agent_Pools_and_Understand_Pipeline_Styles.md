---
lab:
  title: エージェント プールを構成してパイプライン スタイルを把握する
  module: 'Module 02: Implement CI with Azure Pipelines and GitHub Actions'
---

# エージェント プールを構成してパイプライン スタイルを把握する

## 受講生用ラボ マニュアル

## ラボの要件

- このラボには、**Microsoft Edge** または [Azure DevOps 対応ブラウザー](https://docs.microsoft.com/azure/devops/server/compatibility)が必要です。

- **Azure DevOps 組織を設定する:** このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization)に関するページの手順に従って作成してください。

- [Git for Windows ダウンロード ページ](https://gitforwindows.org/)。 このラボでは前提条件の一部としてインストールされます。

- [Visual Studio Code](https://code.visualstudio.com/)。 このラボでは前提条件の一部としてインストールされます。

## ラボの概要

YAML ベースのパイプラインを使用すると、CI/CD をコードとして完全に実装できます。パイプライン定義は、Azure DevOps プロジェクトの一部であるコードと同じリポジトリにあります。 YAML ベースのパイプラインは、プルリクエスト、コードレビュー、履歴、分岐、テンプレートなど、従来のパイプラインの一部である幅広い機能をサポートします。

パイプライン スタイルの選択に関係なく、Azure Pipelines を使用してコードをビルドしたりソリューションをデプロイしたりするには、エージェントが必要です。 エージェントは、一度に 1 つのジョブを実行するコンピューティング リソースをホストします。 ジョブは、エージェントのホスト マシンで直接実行することも、コンテナーで実行することもできます。 管理されている Microsoft がホストするエージェントを使用してジョブを実行するか、自分で設定および管理するセルフホステッド エージェントを実装するかを選択できます。

このラボでは、YAML パイプラインでセルフホステッド エージェントを実装して使用する方法について学習します。

## 目標

このラボを完了すると、次のことができるようになります。

- YAML ベースのパイプラインを実装する
- セルフホステッド エージェントを実装する

## 推定時間:45 分

## Instructions

### 演習 0:ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、[eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) に基づくリポジトリを含む新しい Azure DevOps プロジェクトで構成されます。

#### タスク 1: (完了している場合はスキップしてください) チーム プロジェクトを作成して構成する

このタスクでは、複数のラボで使用される **eShopOnWeb** Azure DevOps プロジェクトを作成します。

1. ラボ コンピューターのブラウザー ウィンドウで、Azure DevOps 組織を開きます。 **[新しいプロジェクト]** をクリックします。 プロジェクトに「**eShopOnWeb**」という名前を付け、他のフィールドは既定値のままにします。 **[作成]** をクリックします。

#### タスク 2: (完了している場合はスキップしてください) eShopOnWeb Git リポジトリをインポートする

このタスクでは、複数のラボで使用される eShopOnWeb Git リポジトリをインポートします。

1. ラボ コンピューターのブラウザー ウィンドウで、Azure DevOps 組織と、前に作成した **eShopOnWeb** プロジェクトを開きます。 **[リポジトリ] > [ファイル]** 、 **[リポジトリをインポートする]** をクリックします。 **[インポート]** を選択します。 **[Git リポジトリをインポートする]** ウィンドウで、URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> を貼り付けて、 **[インポート]** をクリックします。

1. リポジトリは次のように編成されています。
    - **.ado** フォルダーには、Azure DevOps の YAML パイプラインが含まれています。
    - **.devcontainer** フォルダーには、コンテナーを使って開発するためのセットアップが含まれています (VS Code でローカルに、または GitHub Codespaces で)。
    - **infra** フォルダーには、一部のラボ シナリオで使用される Bicep および ARM のコードとしてのインフラストラクチャ テンプレートが含まれています。
    - **.github** フォルダーには、YAML GitHub ワークフローの定義が含まれています。
    - **src** フォルダーには、ラボ シナリオで使用される .NET 8 Web サイトが含まれています。

#### タスク 3: (完了している場合はスキップしてください) メイン ブランチを既定のブランチとして設定する

1. **[リポジトリ] > [ブランチ]** に移動します。
1. **main** ブランチをポイントし、列の右側に表示される省略記号をクリックします。
1. **[既定のブランチとして設定]** をクリックします。

### 演習 1:YAML ベースの Azure Pipelines を作成する

この演習では、YAML ベースのテンプレートを使用して、アプリケーション ライフサイクル ビルド パイプラインを作成します。

#### タスク 1:Azure DevOps YAML パイプラインを作成する

このタスクでは、テンプレートベースの Azure DevOps YAML パイプラインを作成します。

1. **eShopOnWeb** プロジェクトが開いた状態で Azure DevOps ポータルを表示している Web ブラウザーから、左側の垂直ナビゲーション ウィンドウで、**[パイプライン]** をクリックします。
1. **[パイプラインの作成]** ボタンをクリックします。まだ他のパイプラインを作成していない場合は、**[新しいパイプライン]** をクリックして新しいパイプラインを作成します。

1. **[コードはどこにありますか?]** ペインで **[Azure Repos Git]** をクリックします。
1. **[リポジトリの選択]** ペインで **eShopOnWeb** をクリックします。
1. **[パイプラインの構成]** ペインで、 **[既存の Azure Pipelines YAML ファイル]** を選択します。
1. **[既存の YAML ファイルの選択]** で、[ブランチ] では **[メイン]** を、[パス] では **[/.ado/eshoponweb-ci-pr.yml]** を選択します。
1. **[続行]** をクリックします。
1. **[パイプライン YAML の確認]** ペインで、サンプル パイプラインを確認します。 これは、次の処理を行う、かなり簡単な .NET アプリケーション ビルド パイプラインです。

   - 1 つのステージ: Build
   - 1 つのジョブ: Build
   - ビルド ジョブ内の 4 つのタスク:
   - Dotnet Restore
   - Dotnet Build
   - Dotnet Test
   - Dotnet Publish

1. **[パイプライン YAML の確認]** ペインで、 **[実行]** ボタンの横にある下向きのキャレット記号をクリックして、 **[保存]** をクリックします。

    > 注: ここではパイプライン定義を作成しているだけであり、それを実行はしません。 最初に Azure DevOps エージェント プールを設定し、後の演習でパイプラインを実行します。 

### 演習 2:Azure DevOps エージェント プールを管理する

この演習では、セルフホステッド Azure DevOps エージェントを実装します。

#### タスク 1:Azure DevOps セルフホスティングエージェントを構成する

このタスクでは、ラボの仮想マシンを Azure DevOps セルフホスティング エージェントとして構成し、それを使用してビルド パイプラインを実行します。

1. ラボの仮想マシン (ラボ VM) またはお使いになっているコンピュータ内で、Web ブラウザーを起動し、[Azure DevOps ポータル](https://dev.azure.com)に移動し、Azure DevOps 組織に関連付けられている Microsoft アカウントを使用してサインインします。

  > **注**: ラボ仮想マシンには、必要なすべての前提条件ソフトウェアがインストールされている必要があります。 自身のコンピューターにインストールする場合は、デモ プロジェクトをビルドするために必要な .NET 8 SDK 以上をインストールする必要があります。 「[.NET のダウンロード](https://dotnet.microsoft.com/download/dotnet)」を参照してください。

1. Azure DevOps ポータルで、[Azure DevOps] ページの右上隅にある **[ユーザー設定]** アイコンをクリックします。プレビュー機能を有効にしているかどうかに応じて、メニューに **[セキュリティ]** または **[個人用アクセス トークン]** のどちらかの項目が表示されます。**[セキュリティ]** が表示される場合は、それをクリックしてから、**[個人用アクセス トークン]** をクリックします。 **[個人用アクセス トークン]** ペインで、**[+ 新しいトークン]** をクリックします。
1. **[新しい個人用アクセス トークンの作成]** ペインで **[すべてのスコープを表示]** リンクをクリックし、以下の設定を指定し、**[作成]** をクリックします (他はすべて、既定値のままにします):

    | 設定 | 値 |
    | --- | --- |
    | 名前 | **eShopOnWeb** |
    | スコープ (カスタム定義) | **エージェント プール** (必要に応じて以下のスコープ オプションをさらに表示)|
    | アクセス許可 | **読み取りと管理** |

1. **[成功]** ペインで個人用アクセス トークンの値をクリップボードにコピーします。

    > **注**:必ずトークンをコピーしてください。 このペインを閉じると、取得できなくなります。

1. **[成功]** ペインで **[閉じる]** をクリックします。
1. Azure DevOps ポータルの [**パーソナル アクセス トークン**] ペインで、左上隅にある [**Azure DevOps**] 記号をクリックしてから、左下隅にある [**組織の設定**] ラベルをクリックします。
1. [**概要**] ペインの左側の垂直メニューの [**パイプライン**] セクションで、[**エージェント プール**] をクリックします。
1. [**エージェント プール**] ペインの右上隅にある [**プールの追加**] をクリックします。
1. **[エージェント プールの追加]** ペインの **[プールの種類]** ドロップダウン リストで、 **[セルフホステッド]** を選択し、 **[名前]** テキスト ボックスに「**az400m03l03a-pool**」と入力してから、 **[作成]** をクリックします。
1. **[エージェント プール]** ペインに戻り、新しく作成された **[az400m03l03a-pool]** を表すエントリをクリックします。
1. **[az400m03l03a-pool]** ペインの **[ジョブ]** タブで、 **[新しいエージェント]** ボタンをクリックします。
1. [**エージェントの取得**] ペインで、[**Windows**] タブと [**x64**] タブが選択されていることを確認し、[**ダウンロード**] をクリックしてエージェント バイナリを含む zip アーカイブをダウンロードし、ユーザー プロファイル内のローカルの**ダウンロード** フォルダーにダウンロードします。

    > **注**: この時点で、現在のシステム設定でファイルのダウンロードができないことを示すエラー メッセージが表示された場合は、[ブラウザー] ウィンドウの右上隅にある、ドロップダウン メニューの **[設定]** メニュー ヘッダーを示す歯車の記号をクリックします。ドロップダウン メニューで **[インターネット オプション]** を選択し、 **[インターネット オプション]** ダイアログ ボックスで **[詳細]** をクリックし、 **[詳細]** タブで **[リセット]** をクリックし、 **[ブラウザー設定のリセット]** ダイアログ ボックスで **[リセット]** をクリックし、 **[閉じる]** をクリックし、ダウンロードを再度試します。

1. 管理者として Windows PowerShell を起動し、**[管理者: Windows PowerShell]** コンソールで次の行を実行して、**C:\\agent** ディレクトリを作成し、ダウンロードしたアーカイブの内容をそこに抽出します。

    ```powershell
    cd \
    mkdir agent ; cd agent
    $TARGET = Get-ChildItem "$Home\Downloads\vsts-agent-win-x64-*.zip"
    Add-Type -AssemblyName System.IO.Compression.FileSystem
    [System.IO.Compression.ZipFile]::ExtractToDirectory($TARGET, "$PWD")
    ```

1. 同じ **[管理者: Windows PowerShell]** コンソールで、以下を実行してエージェントを構成します。

    ```powershell
    .\config.cmd
    ```

1. プロンプトが表示されたら、次の設定の値を指定します。

    | 設定 | 値 |
    | ------- | ----- |
    | サーバーの URL を入力する | **<https://dev.azure.com/>`<organization_name>`** の形式での Azure DevOps 組織の URL。`<organization_name>` は、Azure DevOps 組織の名前を表します |
    | 認証タイプを入力する (PAT の場合は Enter 押します) | **Enter** |
    | パーソナル アクセス トークンを入力する | このタスクの前半で記録したアクセス トークン |
    | エージェント・プールを入力する (デフォルトで Enter を押します) | **az400m03l03a-pool** |
    | エージェント名を入力する | **az400m03-vm0** |
    | 作業フォルダーを入力する (_work の場合は Enter 押します) | **Enter** |
    | **(表示された場合のみ)** 「各ステップのタスクの解凍を実行する」と入力します。 (N の場合は Enter を押します) | **警告**: メッセージが表示される場合は、**Enter** キーを押すだけにしてください|
    | 「エージェントをサービスとして実行しますか?」と入力します。 (Y/N) (N の場合は Enter を押します) | **Y** |
    | SERVICE_SID_TYPE_UNRESTRICTED の有効化を入力する (Y/N) (N の場合は Enter キーを押す) | **Y** |
    | サービスで使用するユーザー アカウントを入力します (NT AUTHORITY\NETWORK SERVICE で Enter を押します) | **Enter** |
    | Enter whether to prevent service starting immediately after configuration is finished? (構成が完了した直後にサービスが開始されないようにするかどうかを入力します。) (Y/N) (N の場合は Enter を押します) | **Enter** |

    > **注**:セルフホスティド エージェントは、サービスまたは対話型プロセスとして実行できます。 エージェントの機能の検証が簡単になるため、対話型モードから始めることをお勧めします。 本番環境で使用する場合は、エージェントをサービスとして実行するか、自動ログオンを有効にした対話型プロセスとして実行することを検討する必要があります。どちらも実行状態を維持し、オペレーティング システムが再起動された場合にエージェントが自動的に起動するようにするためです。

1. Azure DevOps ポータルを表示しているブラウザー ウィンドウに切り替えて、[**エージェントの取得**] ペインを閉じます。
1. **[az400m03l03a-pool]** ペインの **[エージェント]** タブに戻り、新しく構成したエージェントが **[オンライン]** 状態で一覧に表示されていることを確認してください。
1. Azure DevOps ポータルを表示している Web ブラウザー ウィンドウの左上隅にある、**Azure DevOps** ラベルをクリックします。
1. プロジェクトの一覧から、**eShopOnWeb** プロジェクトを表すタイルをクリックします。
1. **[eShopOnWeb]** ペインの左側の垂直ナビゲーション ウィンドウの **[パイプライン]** セクションで、**[パイプライン]** をクリックします。
1. **[パイプライン]** ペインの **[最近]** タブで、**[eShopOnWeb]** を選択し、**[eShopOnWeb]** ペインで **[編集]** を選択します。
1. **[eShopOnWeb]** 編集ペインの既存の YAML ベースのパイプラインで、ターゲット エージェント プールを指定している `vmImage: ubuntu-latest` という 13 行目の行を以下の内容に置き換えて、新しく作成したセルフホステッド エージェント プールを指定します。

    ```yaml
    name: az400m03l03a-pool
    demands:
    - Agent.Name -equals az400m03-vm0
    ```

    > **警告**:コピーと貼り付けを行うときは、上記と同じインデントになるように注意してください。

    ![YAML プールの構文](images/m3/eshoponweb-ci-pr-pool_v1.png)

1. **[eShopOnWeb]** 編集ペインで、ペインの右上隅にある **[検証して保存]** をクリックします。 これにより、このパイプラインに基づいてビルドが自動的にトリガーされます。
1. Azure DevOps ポータルの左側の垂直ナビゲーション ペインの [**パイプライン**] セクションで、[**パイプライン**] をクリックします。 ラボのセットアップによっては、パイプラインでアクセス許可の入力が求められる場合があります。 **[許可]** をクリックして、パイプラインの実行を許可します。 
1. **[パイプライン]** ペインの **[最近]** タブで **[eShopOnWeb]** エントリをクリックし、**[eShopOnWeb]** ペインの **[実行]** タブで最も新しい実行を選択し、実行の **[概要]** ペインで下部までスクロール ダウンし、**[ジョブ]** セクションで **[フェーズ 1]** をクリックして実行が成功するまでジョブを監視します。

### 演習 3: ラボで使用されているリソースを削除する

1. コマンド プロンプトから `.\config.cmd remove` を実行して、エージェント サービスを停止して削除します。
   - お使いのエージェントを組織から削除するために、もう一度個人用アクセス トークンを入力するように求められます。
   - 個人用アクセス トークンがなくなった場合は、演習 2、タスク 1、手順 2 で最初に作成したトークンの再生成に進むことができます。
1. エージェント プールを削除します。
1. PAT トークンを取り消します。
1. Repos/.ado/eshoponweb-ci-pr.yml ファイルに移動し、**[編集]** を選び、13 から 15 行目 (エージェント プール スニペット) を削除して元の `vmImage: ubuntu-latest` に戻すことで、**eshoponweb-ci-pr.yml** ファイルの変更を元に戻します。 (これは、この後のラボ演習で同じサンプル パイプライン ファイルを使うためです)。

![パイプライン プールを vmImage の設定に戻す](images/m3/eshoponweb-ci-pr-vmimage_v2.png)

## 確認

このラボでは、YAML パイプラインでセルフホステッド エージェントを実装して使用する方法について学習しました。
