---
lab:
  title: ラボ 04:エージェント プールの構成とパイプライン スタイルの把握
  module: 'Module 3: Implement CI with Azure Pipelines and GitHub Actions'
ms.openlocfilehash: 2aff45194cb8c16e1d19be44af175bb937873f60
ms.sourcegitcommit: f72fcf5ee578f465b3495f3cf789b06c530e88a4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2022
ms.locfileid: "139262527"
---
# <a name="lab-04-configuring-agent-pools-and-understanding-pipeline-styles"></a>ラボ 04:エージェント プールの構成とパイプライン スタイルの把握
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-overview"></a>ラボの概要

YAML ベースのパイプラインを使用すると、CI/CD をコードとして完全に実装できます。パイプライン定義は、Azure DevOps プロジェクトの一部であるコードと同じリポジトリにあります。 YAML ベースのパイプラインは、プルリクエスト、コードレビュー、履歴、分岐、テンプレートなど、従来のパイプラインの一部である幅広い機能をサポートします。 

パイプライン スタイルの選択に関係なく、Azure Pipelines を使用してコードをビルドしたりソリューションをデプロイしたりするには、エージェントが必要です。 エージェントは、一度に 1 つのジョブを実行するコンピューティング リソースをホストします。 ジョブは、エージェントのホスト マシンで直接実行することも、コンテナーで実行することもできます。 管理されている Microsoft がホストするエージェントを使用してジョブを実行するか、自分で設定および管理するセルフホステッド エージェントを実装するかを選択できます。 

このラボでは、従来のパイプラインを YAML ベースのパイプラインに変換し、最初に Microsoft がホストするエージェントを使用して実行し、次にセルフホステッド エージェントを使用して同等のタスクを実行するプロセスを順を追って説明します。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- YAML ベースのパイプラインを実装する
- セルフホステッド エージェントを実装する

## <a name="lab-duration"></a>ラボの所要時間

-   推定時間:**45 分**

## <a name="instructions"></a>Instructions

### <a name="before-you-start"></a>開始する前に

#### <a name="sign-in-to-the-lab-virtual-machine"></a>ラボ仮想マシンにサインインする

必ず次の資格情報を使用して Windows 10 コンピューターにサインインしてください。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### <a name="review-the-installed-applications"></a>インストールされているアプリケーションを確認する

Windows 10 デスクトップでタスク バーを見つけます。 タスク バーには、このラボで使用するアプリケーションのアイコンが含まれています。
    
-   Microsoft Edge
-   [Visual Studio Code](https://code.visualstudio.com/)。 このラボでは前提条件の一部としてインストールされます。

#### <a name="set-up-an-azure-devops-organization"></a>Azure DevOps 組織を設定する

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)で利用できる手順に従って作成してください。

### <a name="exercise-0-configure-the-lab-prerequisites"></a>演習 0:ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。Azure DevOps Demo Generator テンプレートに基づいて事前構成された Parts Unlimited チームプロジェクトで構成されます。

#### <a name="task-1-configure-the-team-project"></a>タスク 1:チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用して、**PartsUnlimited** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。 このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**:サイトの詳細については、https://docs.microsoft.com/en-us/azure/devops/demo-gen を参照してください。

1.  「**サインイン**」をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。
1.  必要な場合は、「**Azure DevOps Demo Generator**」ページで「**承諾する**」をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。
1.  [**新しいプロジェクトの作成**] ページの [**新しいプロジェクト名**] テキスト ボックスに「**エージェント プールの構成とパイプライン スタイルの理解**」と入力し、[**組織の選択**] ドロップダウン リストで、Azure DevOps 組織を選択して、[**テンプレートの選択**] をクリックします。
1.  [**テンプレートの選択**] ページで、**PartsUnlimited** テンプレートをクリックし、[**テンプレートの選択**] をクリックします。
1.  [**プロジェクトの作成**] をクリックします。

    > **注**:プロセスが完了するまでお待ちください。 これには 2 分ほどかかります。 プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

### <a name="exercise-1-author-yaml-based-azure-devops-pipelines"></a>演習 1:YAML ベースの Azure DevOps パイプラインを作成する

この演習では、従来の Azure DevOps パイプラインを YAML ベースのパイプラインに変換します。 

#### <a name="task-1-create-an-azure-devops-yaml-pipeline"></a>タスク 1:Azure DevOps YAML パイプラインを作成する

このタスクでは、テンプレートベースの Azure DevOps YAML パイプラインを作成します。

1.  
          **エージェント プールの構成とパイプライン スタイルの理解** プロジェクトが開いた状態で Azure DevOps ポータルを表示している Web ブラウザーから、左側の垂直ナビゲーション ペインで [**パイプライン**] をクリックします。 
1.  [**パイプライン**] ペインの [**最近**] タブで、[**新しいパイプライン**] をクリックします。
1.  「**コードはどこにありますか?**」ペインで「**Azure Repos Git**」をクリックします。 
1.  「**リポジトリの選択**」ペインで「**PartsUnlimited**」をクリックします。
1.  **[パイプライン YAML をレビューする]** ペインでサンプル パイプラインを確認し、**[実行]** ボタンの横にある下向きのキャレット記号をクリックして、**[保存]** をクリックします。

### <a name="exercise-2-manage-azure-devops-agent-pools"></a>演習 2:Azure DevOps エージェント プールを管理する

この演習では、セルフホスティド Azure DevOps エージェントを実装します。

#### <a name="task-1-configure-an-azure-devops-self-hosting-agent"></a>タスク 1:Azure DevOps セルフホスティングエージェントを構成する

このタスクでは、LOD VM を Azure DevOps セルフホスティング エージェントとして構成し、それを使用してビルド パイプラインを実行します。

1.  ラボの仮想マシン (ラボ VM) またはお使いになっているコンピュータ内で、Web ブラウザーを起動し、[Azure DevOps ポータル](https://dev.azure.com)に移動し、Azure DevOps 組織に関連付けられている Microsoft アカウントを使用してサインインします。
1.  Azure DevOps ポータルで、[Azure DevOps] ページの右上隅にある **[ユーザー設定]** アイコンをクリックします。プレビュー機能を有効にしているかどうかに応じて、メニューに **[セキュリティ]** または **[個人用アクセス トークン]** のどちらかの項目が表示されます。**[セキュリティ]** が表示される場合は、それをクリックしてから、**[個人用アクセス トークン]** をクリックします。 **[個人用アクセス トークン]** ペインで、**[+ 新しいトークン]** をクリックします。
1.  「**新しい個人用アクセス トークンの作成**」ペインで「**すべてのスコープを表示**」リンクをクリックし、以下の設定を指定します。「**作成**」をクリックします (他はすべて、既定値のままにします):

    | 設定 | 値 |
    | --- | --- |
    | Name | **エージェント プールの構成とパイプライン スタイルの理解ラボ** |
    | スコープ (カスタム定義) | **エージェント プール** (必要に応じて以下のスコープ オプションをさらに表示)|
    | アクセス許可 | **読み取りと管理** |

1.  「**成功**」ペインで個人用アクセス トークンの値をクリップボードにコピーします。

    > **注**:必ずトークンをコピーしてください。 このペインを閉じると、取得できなくなります。 

1.  「**成功**」ペインで「**閉じる**」をクリックします。
1.  Azure DevOps ポータルの [**パーソナル アクセス トークン**] ペインで、左上隅にある [**Azure DevOps**] 記号をクリックしてから、左下隅にある [**組織の設定**] ラベルをクリックします。
1.  [**概要**] ペインの左側の垂直メニューの [**パイプライン**] セクションで、[**エージェント プール**] をクリックします。
1.  [**エージェント プール**] ペインの右上隅にある [**プールの追加**] をクリックします。 
1.  [**エージェント プールの追加**] ペインの [**プールの種類**] ドロップダウン リストで、[**セルフホスティド**] を選択し、[**名前**] テキスト ボックスに「**az400m05l05a-pool**」と入力して、[**作成**] をクリックします。
1.  [**エージェント プール**] ペインに戻り、新しく作成された **az400m05l05a-pool** を表すエントリをクリックします。 
1.  **az400m05l05a-pool** ペインの **[ジョブ]** タブで、**[新しいエージェント]** ボタンをクリックします。
1.  [**エージェントの取得**] ペインで、[**Windows**] タブと [**x64**] タブが選択されていることを確認し、[**ダウンロード**] をクリックしてエージェント バイナリを含む zip アーカイブをダウンロードし、ユーザー プロファイル内のローカルの **ダウンロード** フォルダーにダウンロードします。

    > **注**:この時点で、現在のシステム設定でファイルのダウンロードができないことを示すエラーメッセージが表示された場合は、[インターネット エクスプローラー] ウィンドウの右上隅にある、ドロップダウン メニューの [**設定**] メニュー ヘッダーを示す歯車の記号をクリックします。[**インターネット オプション**] を選択し、[**インターネット オプション**] ダイアログ ボックスで [**詳細設定**] をクリックし、[**詳細設定**] タブで [**リセット**] をクリックし、[**インターネット エクスプローラー設定のリセット**] ダイアログ ボックスで [**リセット**] をクリックし、[**閉じる**] をクリックして、ダウンロードを再試行します。 

1.  管理者として Windows PowerShell を起動し、**[管理者: Windows PowerShell]** コンソールで次の行を実行して、**C:\\agent** ディレクトリを作成し、ダウンロードしたアーカイブの内容をそこに抽出します。 

    ```powershell
    cd \
    mkdir agent ; cd agent
    $TARGET = Get-ChildItem "$Home\Downloads\vsts-agent-win-x64-*.zip"
    Add-Type -AssemblyName System.IO.Compression.FileSystem
    [System.IO.Compression.ZipFile]::ExtractToDirectory($TARGET, "$PWD")
    ```

1.  同じ **[管理者: Windows PowerShell]** コンソールで、以下を実行してエージェントを構成します。

    ```powershell
    .\config.cmd
    ```

1.  プロンプトが表示されたら、次の設定の値を指定します。

    | 設定 | 値 |
    | ------- | ----- |
    | サーバーの URL を入力する | **https://dev.azure.com/`<organization_name>`** の形式での Azure DevOps 組織の URL。`<organization_name>` は、Azure DevOps 組織の名前を表します |
    | 認証タイプを入力する (PAT の場合は Enter 押します) | **Enter** |
    | パーソナル アクセス トークンを入力する | このタスクの前半で記録したアクセス トークン |
    | エージェント・プールを入力する (デフォルトで Enter を押します) | **az400m05l05a-pool** |
    | エージェント名を入力する | **az400m05-vm0** |
    | 作業フォルダーを入力する (_work の場合は Enter 押します) | **Enter** |
    | 「各ステップのタスクの解凍を実行する」と入力します。 (N の場合は Enter を押します) | **Enter** |
    | 「エージェントをサービスとして実行しますか?」入力します。 (Y/N) (N の場合は Enter を押します) | **Y** |
    | サービスで使用するユーザー アカウントを入力します (NT AUTHORITY\NETWORK SERVICE で Enter を押します) | **Enter** |
    | Enter whether to prevent service starting immediately after configuration is finished? (構成が完了した直後にサービスが開始されないようにするかどうかを入力します。) (Y/N) (N の場合は Enter を押します) | **Enter** |

    > **注**:セルフホスティド エージェントは、サービスまたは対話型プロセスとして実行できます。 エージェントの機能の検証が簡単になるため、対話型モードから始めることをお勧めします。 本番環境で使用する場合は、エージェントをサービスとして実行するか、自動ログオンを有効にした対話型プロセスとして実行することを検討する必要があります。どちらも実行状態を維持し、オペレーティング システムが再起動された場合にエージェントが自動的に起動するようにするためです。

1.  Azure DevOps ポータルを表示しているブラウザー ウィンドウに切り替えて、[**エージェントの取得**] ペインを閉じます。
1.  
          **az400m05l05a-pool** ペインの [**エージェント**] タブに戻り、新しく構成されたエージェントが [**オンライン**] ステータスで一覧表示されていることに注意してください。
1.  Azure DevOps ポータルを表示している Web ブラウザー ウィンドウの左上隅にある、**Azure DevOps** ラベルをクリックします。
1.  プロジェクトのリストを表示しているブラウザー ウィンドウで、**エージェント プールの構成とパイプライン スタイルの理解** プロジェクトを表すタイルをクリックします。 
1.  [**エージェント プールの構成とパイプライン スタイルの理解**] ペインの左側の垂直ナビゲーション ペインの [**パイプライン**] セクションで、[**パイプライン**] をクリックします。 
1.  [**パイプライン**] ペインの [**最近**] タブで [**PartsUnlimited**] を選択し、[**PartsUnlimited**] ペインで [**編集**] を選択します。
1.  **PartsUnlimited** 編集ペインの既存の YAML ベースのパイプラインで、ターゲット エージェント プールを指定している `vmImage: windows-2019` の行を、新しく作成されたセルフホスティド エージェント プールを指定する次の内容に置き換えます。

    ```yaml
    name: az400m05l05a-pool
    demands:
    - agent.name -equals az400m05-vm0
    ```
1. `Task: NugetInstaller@0` で **[設定] (グレーでタスクの上に表示されているリンク)** をクリックし、**[インストールする NuGet.exe のバージョン]** > **[4.0.0]** に変更して、**[追加]** をクリックします。 
1.  
          **PartsUnlimited** 編集ペインのペインの右上隅にある [**保存**] をクリックし、[**保存**] ペインでもう一度 [**保存**] をクリックします。 これにより、このパイプラインに基づいてビルドが自動的にトリガーされます。 
1.  Azure DevOps ポータルの左側の垂直ナビゲーション ペインの [**パイプライン**] セクションで、[**パイプライン**] をクリックします。
1.  [**パイプライン**] ペインの [**最近**] タブで、[**PartsUnlimited**] エントリをクリックし、[**PartsUnlimited**] ペインの [**実行**] タブで最新の実行を選択し、実行の [**概要**] ペインで一番下までスクロールし、[**ジョブ**] セクションで [**フェーズ 1**] をクリックし、正常に完了するまでジョブを監視します。 

#### <a name="review"></a>確認

このラボでは、従来のパイプラインを YAML ベースのパイプラインに変換する方法と、セルフホスティド エージェントを実装して使用する方法を学習しました。
