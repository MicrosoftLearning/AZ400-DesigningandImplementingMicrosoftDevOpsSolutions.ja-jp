---
lab:
  title: ラボ 20:Azure DevOps とチーム間の統合
  module: 'Module 09: Implement continuous feedback'
ms.openlocfilehash: 948b36e939e869f553106a104b652a73d26f37e0
ms.sourcegitcommit: f72fcf5ee578f465b3495f3cf789b06c530e88a4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2022
ms.locfileid: "139262528"
---
# <a name="lab-20-integration-between-azure-devops-and-teams"></a>ラボ 20:Azure DevOps とチーム間の統合
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-overview"></a>ラボの概要

**[Microsoft Teams](https://teams.microsoft.com/start)** は、Office 365 でのチームワークのためのハブです。 チームのあらゆるチャットやミーティング、ファイル、アプリを 1 箇所で管理して使用できます。 Office 365 と Azure DevOps 全体でチーム、会話、コンテンツ、ツールのハブをソフトウェア開発チームに提供します。

このラボでは、Azure DevOps サービスと Microsoft Teams 間の統合シナリオを実装します。

> **注**:**Azure DevOps Services** を Microsoft Teams と統合すると、開発サイクルを通して包括的なチャットとコラボレーションを体験できます。 チームは Azure DevOps チーム プロジェクトで、作業項目、プル要求、コードのコミット、ビルドおよびリリース イベントに関する通知とアラートにより、重要なアクティビティについて容易に情報を得られます。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- Microsoft Teams と Azure DevOps を統合する
- Azure DevOps Kanban ボードと Teams のダッシュボードを統合する
- Azure Pipelines と Microsoft Teams を統合する
- Azure Pipelines アプリを Microsoft Teams にインストールする
- Azure Pipelines 通知をサブスクライブする

## <a name="lab-duration"></a>ラボの所要時間

-   予想所要時間: **60 分**

## <a name="instructions"></a>Instructions

### <a name="before-you-start"></a>開始する前に

#### <a name="sign-in-to-the-lab-virtual-machine"></a>ラボ仮想マシンにサインインする

必ず次の資格情報を使用して Windows 10 コンピューターにサインインしてください。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### <a name="review-applications-required-for-this-lab"></a>このラボで必要なアプリケーションのレビュー

このラボで使用するアプリケーションを特定:
    
-   Microsoft Edge
-   Microsoft Teams。 このラボでは前提条件の一部としてインストールされます。

#### <a name="set-up-an-office-365-subscription"></a>Office 365 サブスクリプションを設定 

[Microsoft Teams サインアップ ページ](https://teams.microsoft.com/start) から無料のトライアル サブスクリプションを作成します。 

#### <a name="set-up-an-azure-devops-organization"></a>Azure DevOps 組織を設定します。 

[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops) の手順に従います。 Azure DevOps 組織を作成する際は、Office 365 サブスクリプションの設定で使用したものと同じユーザー アカウントを使ってサインインします。

> **注**:Office 365 サブスクリプションと Azure DevOps 組織は、同じ Azure Active Directory (Azure AD) テナントを共有する必要があります。

### <a name="exercise-0-configure-the-lab-prerequisites"></a>演習 0:ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、Azure DevOps Demo Generator テンプレートおよび Microsoft Teams で作成したチームに基づいて事前に構成された **Tailwind Traders** チーム プロジェクトから成っています。

#### <a name="task-1-configure-the-team-project"></a>タスク 1:チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用し、**Tailwind Traders** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。 このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**:サイトの詳細については、 https://docs.microsoft.com/en-us/azure/devops/demo-gen を参照してください。

1.  「**サインイン**」をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。
1.  必要な場合は、「**Azure DevOps Demo Generator**」ページで「**承諾する**」をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。
1.  「**新しいプロジェクトの作成**」ページで「**テンプレートの選択**」をクリックします。
1.  テンプレート リストのツール バーで、 **[全般]** をクリックし、 **[Tailwind Traders]** を選択して、 **[テンプレートの選択]** をクリックします。
1.  「**新しいプロジェクトの作成**」ページで「**新しいプロジェクト名**」テキストボックスに「**Tailwind Traders**」と入力します。「**組織の選択**」ドロップダウン リストで Azure DevOps 組織を選択します。
1.  **[新しいプロジェクトの作成]** ページで、欠落している拡張機能のインストールを求められたら、 **[ARM 出力]** の下にあるチェックボックスをオンにし、 **[プロジェクトの作成]** をクリックします (GitHub フォークは無視)。

    > **注**:プロセスが完了するまでお待ちください。 これには 2 分ほどかかります。 プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

#### <a name="task-2-create-a-team-in-microsoft-teams"></a>タスク 2:Microsoft Teams でチームを作成する

このタスクでは、Microsoft Teams でチームを作成します。

1.  ラボのコンピューターで Web ブラウザーを起動して [Microsoft Teams ダウンロード ページ](https://www.microsoft.com/en-us/microsoft-365/microsoft-teams/download-app) に移動します。そこから既定の設定で Microsoft Teams をダウンロードしてインストールします。 
1.  ラボのコンピューターでデスクトップ アプリを使い、**Microsoft Teams** を起動します。

    > **注**:Web ブラウザーを使用して [Microsoft Teams 起動ページ](https://teams.microsoft.com/dl/launcher/launcher.html?url=/_%23/l/home/0/0&type=home) に移動することもできます

1.  サインインするよう指示されたら、Office 365 サブスクリプションの一部であり、Azure DevOps 組織へのアクセス権限があるユーザー アカウントを使ってサインインします。
1.  Microsoft Teams のページ左側のツールバーで「**Teams**」をクリックし、チーム一覧の最下部で「**チームに参加、またはチームを作成**」をクリックします。

    >**注**:チームは、共通の目標を掲げて集まる人々のグループです。 

1.  「**チームに参加、またはチームを作成**」ペインで「**チームを作成**」をクリックします。
1.  **[チームの作成]** パネルで、 *[初めから作成]* * をクリックし、 **[チームの種類は何でしょうか?]** パネルで、 **[プライベート]** をクリックします
1.  「**プライベート チームに関する簡単な詳細**」パネルで「**チームに名前を付ける**」を「**Tailwind Traders**」に置き換えて「**作成**」をクリックします。
1.  「**Tailwind Traders にメンバーを追加**」パネルで「**スキップ**」をクリックします。

### <a name="exercise-1-integrate-azure-boards-with-microsoft-teams"></a>演習 1:Azure Boards に Microsoft Teams を統合する

この演習では、Azure Boards と Microsoft Teams 間で統合を実装します。

#### <a name="task-1-install-and-configure-azure-boards-app-in-microsoft-teams"></a>タスク 1:Microsoft Teams で Azure Boards アプリをインストールして構成する

このタスクでは、Microsoft Teams で新しく作成されたチームで Azure Boards アプリをインストールして構成します。

1.  Microsoft Teams ウィンドウの左下コーナーで「**アプリ**」アイコンをクリックします。 「**アプリ**」ペインが開きます。
1.  「**アプリ**」ペインの「**すべてのアプリを検索**」テキストボックスで「**Azure Boards**」と入力し、アプリ一覧で「**Azure Boards**」をクリックします。
**[開く]** ボタンのすぐ右にある下向き矢印をクリックし、ドロップダウンで **[チームに追加]** エントリを選択します。
1.  **[チームのために Azure ボードを設定]** パネルの **[検索]** テキスト ボックスに、「**Tailwind Traders**」と入力します。 検索結果で **[Tailwind Traders] > [全般]** エントリを選択し、 **[ボットを設定]** をクリックします。

1.  **Tailwind Traders** チームの **[全般]** チャネルの投稿リストから、**Azure Boards** というタイトルの投稿を選択し、**Enter** キーを押して、以下のようなボットで投稿された追加メッセージを確認します。

    ```
    Here are some of the things you can do:
    link [project url] - Link to a project to create work items and receive notifications
    subscriptions - Add or remove subscriptions for this channel
    addAreapath [area path] - Add an area path from your project to this channel
    signin - Sign in to your Azure Boards account
    signout - Sign out from your Azure Boards account
    unlink - Unlink a project from this channel
    feedback - Report a problem or suggest a feature
    To know more see documentation.
    ```
1.  **[新しい会話]** を開き、「 **@** 」と入力します。その後、「**Azure**」と入力すると、結果のプロンプトが表示されます。コマンド パネルで、後に **signin** が続く **Azure Boards** を選択します。 手順に従って、Azure DevOps 組織にアクセスできることを確認します。
1.  Azure DevOps の **Tailwind Traders** プロジェクトの URL をコピーします。 例: https://dev.azure.com/myorg/myproject。 Teams チャネルで **[新しい会話]** を開き、「 **@** 」と入力します。その後、「**Azure**」と入力すると、結果のプロンプトが表示されます。コマンド パネルで、後に **link** が続く **Azure Boards** を選択します。 この時点で、コピーしたプロジェクト URL を貼り付けます。 投稿は、'**Azure Boards** link https://dev.azure.com/myorg/myproject ' のようになるはずです。 **Enter** キーを押します。

    受け取ったメッセージを確認します。

    ```
    NAME has linked this channel to project  Tailwind Traders. Create work items using @Azure         Boards create.
    To monitor work items, please add subscription
    Add subscription
    ```

1. **[サブスクリプションの追加]** をクリックします。 ドロップダウンで、イベントのリストから **作成された作業項目** を選択し、 **[次へ]** をクリックします。 既定値のままにして、 **[送信]** をクリックします。 **[OK]** をクリックして閉じます。 新しく追加されたサブスクリプションの詳細と共に、メッセージがチャネルに投稿されます。
1.  Azure DevOps ポータルで **Tailwind Traders** プロジェクトが表示されている Web ブラウザーに切り替え、 **[ボード] > [作業項目]** をクリックします。 **[新しい作業項目]** をクリックし、ドロップダウンで **[ユーザー ストーリー]** を選択します。 ユーザー ストーリーにタイトル (**Teams を使って Azure Boards 統合をテストする** など) を付けて、**保存** します。 最近設定された Teams チャネルでは、作成されたユーザー ストーリーに関する情報を含む通知やカードが投稿されます。

#### <a name="task-2-add-azure-boards-kanban-boards-to-microsoft-teams"></a>タスク 2:Azure Boards Kanban ボードを Microsoft Teams に追加する

このタスクでは、Azure Boards Kanban ボードを Microsoft Teams のタブに追加します。

> **注**:かんばんボードは、バックログをインタラクティブな看板に変え、作業の視覚的なフローを提供します。 作業が構想から完了まで進むにつれて、ボード上の項目を更新します。 各列は作業段階を示し、各カードはその作業段階のユーザーのストーリー (青いカード) またはバグ (赤いカード) を表します。 タブを使用することで、チームのかんばんボードまたはお気に入りのダッシュボードを直接、Microsoft Teams に取り込むことができます。 **タブ** を使うと、チーム メンバーは専用のキャンバスで、チャネル内で、またはユーザーの個人的なアプリ スペースでサービスにアクセスできます。 既存の Web アプリを利用して、Teams 内ですばらしいタブ経験を創造できます。

1.  ラボのコンピューターで、Azure DevOps ポータルで **Tailwind Traders** プロジェクトを表示している Web ブラウザーに切り替えます。Azure DevOps ポータルの一番左にある垂直メニュー バーで「**ボード**」をクリックし、「**ボード**」セクションで「**ボード**」をクリックします。
1.  「**ボード**」パネルの「**マイ チーム ボード (1)** 」セクションで、「**Tailwind Traders Team ボード**」エントリをクリックします。 
1.  「**Tailwind Traders Team**」ペインが表示されている間に、Web ブラウザー ウィンドウでその URL をクリップボードにコピーします。
1.  Microsoft Teams ウィンドウに切り替え、新しく作成されたチーム **Tailwind Traders** の **[全般]** チャネルが選択されていることを確かめ、 **[全般]** ペインの上部の **[投稿]、[ファイル] および [Wiki] タブ** の横にあるプラス記号をクリックします。 「**タブの追加**」パネルが表示されます。
1.  「**タブの追加**」パネルで「**Web サイト**」をクリックします。「**Web サイト**」パネルで「**タブ名**」を「**Tailwind Traders Team ボード**」に設定し、「**URL**」をクリップボードにコピーしたばかりの URL に設定してから「**保存**」をクリックします。
1.  **Tailwind Traders** チームの **[全般]** チャネルが選択されている状態の Microsoft Teams ウィンドウにあるトップ メニューのタブ リストで、新しく追加された **[Tailwind Traders Team ボード]** タブをクリックし、Azure DevOps ポータルで利用できる **Tailwind Traders Team** ボードに一致する内容がそれに含まれていることを確めます (サインインが必要な場合があります)。

> **注**:毎日の立ち上げ中にすべての作業を監視できます。該当する作業項目の状態が変化した場合、更新はリアルタイムで反映されます。 かんばんボードを Microsoft Teams から変更するオプションもあります。

### <a name="exercise-2-integrate-azure-pipelines-with-microsoft-teams"></a>演習 2:Azure Pipelines と Microsoft Teams を統合する

この演習では、Azure Pipelines と Microsoft Teams の統合を実装します。

#### <a name="task-1-install-and-configure-azure-pipelines-app-in-microsoft-teams"></a>タスク 1:Microsoft Teams で Azure Pipelines アプリをインストールして構成する

このタスクでは、Microsoft Teams で指定されたチームで Azure Pipelines アプリをインストールして構成します。

> **注**:Microsoft Teams の Azure Pipelines アプリを使うと、パイプラインのイベントを監視できます。 通知を直接 Teams チャネルに投稿することで、リリースや保留中の承認、完了したビルドといったイベントのサブスクリプションを設定して管理できます。 また、Teams チャネル内からリリースを承認することも可能です。

1.  Microsoft Teams ウィンドウに移動し、左下隅にある **[アプリ]** アイコンをクリックします。 「**アプリ**」ペインが開きます。
1.  「**アプリ**」ペインの「**すべてのアプリを検索**」テキストボックスで「**Azure Pipelines**」と入力し、アプリ一覧で「**Azure Pipelines**」をクリックします。
1.  「**Azure Pipelines**」パネルで、「**開く**」ボタンのすぐ右にある下向き矢印をクリックします。ドロップダウン リストで「**チームに追加**」エントリをクリックします。
1.  **[チームのために Azure Pipelines を設定]** パネルの **[検索]** テキスト ボックスに、「**Tailwind Traders**」と入力し、結果のリストで、 **[Tailwind Traders] > [全般]** エントリを選択し、 **[ボットを設定]** をクリックします。 **Tailwind Traders** チームの **[全般]** チャネルの投稿ビューに自動的にリダイレクトされます。
1.  **Tailwind Traders** チームの **[全般]** チャネルの投稿リストで、 **[新しい会話]** を開き、「 **@** 」と入力します。その後、「**Azure**」と入力すると、結果のプロンプトが表示されます。**Azure Pipelines** を選択し、**Enter** キーを押して、以下のような、ボットで投稿された追加メッセージを確認します。

    ```
    Here are some of the things you can do:
    subscribe [pipeline url/ project url] - Subscribe to a pipeline or all pipelines in a project to receive notifications
    subscriptions - Add or remove subscriptions for this channel
    feedback - Report a problem or suggest a feature
    signin - Sign in to your Azure Pipelines account
    signout - Sign out from your Azure Pipelines account
    To know more see documentation.
    ```
   
#### <a name="task-2-subscribe-to-the-azure-pipeline-notifications-in-microsoft-teams"></a>タスク 2:Microsoft Teams で Azure Pipelines 通知にサブスクライブする

このタスクで、Microsoft Teams で Azure Pipelines 通知にサブスクライブします

> **注**:`@Azure Pipelines` ハンドルを使用して、アプリの操作を開始できます。

1.  **[投稿]** タブが選択された状態で、**Tailwind Traders** チームの **[全般]** チャネルで、「 **@** 」と入力します。その後、「**Azure**」と入力すると、結果のプロンプトが表示されます。コマンド パネルで、後に **signin** が続く **Azure Pipelines** を選択して、**Enter** キーを押します。
1.  「**Azure Pipelines サインイン**」ペインで「**サインイン**」をクリックします。
1.  **サービス フック (読み取りと書き込み)** 、**ビルド (実行のビルド)** 、**リリース (読み取り、書き込み、実行、管理)** 、**プロジェクトおよびチーム (読み取り)** 、**ID ピッカー (読み取り)** 、**Teams 統合** 許可を与えるよう指示されたら、「**承諾**」をクリックしてから「**閉じる**」をクリックします。

    >**注**:これで、`@azure pipelines subscribe [pipeline url]` コマンドを実行して、Azure DevOps パイプラインをサブスクライブできるようになりました。

1.  Azure DevOps ポータルの **Tailwind Traders** プロジェクトが表示されている Web ブラウザーに切り替え、Azure DevOps ポータルの一番左側にある垂直メニュー バーで **[パイプライン]** をクリックし、 **[パイプライン]** ペインで、 **[Website-CI]** エントリをクリックします。その後、 **[Website-CI]** ペインの使用中に、Web ブラウザー ウィンドウでその URL をクリップボードにコピーします。

    >**注**:URL は `https://dev.azure.com/<organization_name>/Tailwind%20Traders/_build?definitionId=<number>` の形式になります。ここで、`<organization_name>` は DevOps 組織の名前を表すプレースホルダーです。

    >**注**:パイプライン URL は、*definitionId* または *buildId/releaseId* が URL に含まれているパイプライン内のどのページでも構いません。

1.  Teams に戻り、 **[投稿]** タブが選択された状態で、**Tailwind Traders** チームの **[全般]** チャネルで、「 **@** 」と入力します。その後、「**Azure**」と入力すると、結果のプロンプトが表示されます。コマンド パネルで、後に **subscribe** が続く **Azure Pipelines** を選択します。 この時点で、コピーしたパイプライン URL を貼り付けます。 投稿は `@Azure Pipelines subscribe https://dev.azure.com/<organization_name>/Tailwind%20Traders/_build?definitionId=<number>` のようになるはずです (必ず、`<organization_name>` プレースホルダーをご自分の DevOps 組織の名前に置き換えてください)。 **Enter** キーを押します。
1.  サブスクリプションが作成されたという通知を待ちます。
 
    >**注**:ビルド パイプラインで、チャネルは「**実行ステージの状態変更**」と「**実行ステージは承認待ち**」という通知にサブクライブします。

1.  Azure DevOps ポータルの **Tailwind Traders** プロジェクトが表示されている Web ブラウザーに切り替え、Azure DevOps ポータルの一番左側にある垂直メニュー バーで **[パイプライン]** をクリックします。 **[パイプライン]** セクションでは、 **[リリース]** をクリックし、リリースのリストで、* *[Website-CD]* エントリをクリックします。その後、 **[Website-CD]** エントリが選択された状態で、Web ブラウザー ウィンドウでその URL をクリップボードにコピーします。

    >**注**:URL は `https://dev.azure.com/<organization_name>/Tailwind%20Traders/_release?_a=releases&view=mine&definitionId=2` の形式になります。ここで、`<organization_name>` は DevOps 組織の名前を表すプレースホルダーです。

1.  **[投稿]** タブが選択された状態で、**Tailwind Traders** チームの **[全般]** チャネルで、「 **@** 」と入力します。その後、「**Azure**」と入力すると、結果のプロンプトが表示されます。コマンド パネルで、後に **subscribe** が続く **Azure Pipelines** を選択します。 この時点で、コピーしたリリース URL を貼り付けます。 投稿は `@Azure Pipelines subscribe https://dev.azure.com/<organization_name>/Tailwind%20Traders/_release?_a=releases&view=mine&definitionId=2` のようになるはずです (必ず、`<organization_name>` プレースホルダーをご自分の DevOps 組織の名前に置き換えてください)。これは、リリース パイプラインをサブスクライブするためのものです。 **Enter** キーを押します。

    >**注**:リリース パイプラインで、チャネルは「**デプロイ開始**」、「**デプロイ完了**」、「**デプロイ承認待ち**」通知にサブスクライブします。
 
#### <a name="task-3-using-filters-to-customize-subscriptions-to-azure-pipelines-in-microsoft-teams"></a>タスク 3:フィルターを使用し、Microsoft Teams での Azure Pipelines へのサブスクリプションをカスタマイズする

このタスクでは、フィルターを使い、Microsoft Teams での Azure Pipelines へのサブスクリプションをカスタマイズします。

>**注**:ユーザーがいずれかのパイプラインにサブスクライブすると、既定によりいくつかのサブスクリプションが作成され、フィルターは適用されません。 ユーザーは頻繁にサブスクリプションをカスタマイズする必要があります。 たとえば、ユーザーはビルドが失敗した場合のみ、またはデプロイが運用環境にプッシュされる場合にのみ通知を希望する可能性があります。 Azure Pipelines アプリは、チャネルの表示内容をカスタマイズするフィルターをサポートします。

>**注**:`@Azure Pipelines subscriptions` コマンドを使用して、サブスクリプションを一覧表示および管理できます。 このコマンドでは、当該チャネルの現在のサブスクリプションすべてを一覧表示します。

1.  **[投稿]** タブが選択された状態で、**Tailwind Traders** チームの **[全般]** チャネルで、「 **@** 」と入力します。その後、「**Azure**」と入力すると、結果のプロンプトが表示されます。後に **subscriptions** が続く **Azure Pipelines** を選択し、**Enter** キーを押します。 **Azure Pipelines** ボットからの応答で、 **[サブスクリプションの追加]** を選択します。
1.  **Azure Pipelines** の **[サブスクリプションの追加]** パネルにある **[イベントの選択]** ドロップダウン リストで **[ビルド完了]** が選択されていることを確かめて、 **[次へ]** をクリックします。
1.  **Azure Pipelines** の **[サブスクリプションの追加]** パネルにある **[パイプラインの選択]** ドロップダウン リストで **[Website-CI]** が選択されていることを確かめて、 **[次へ]** をクリックします。
1.  **Azure Pipelines** の **[サブスクリプションの追加]** パネルにある **[ビルドの状態]** ドロップダウン リストで **[任意]** が選択されていることを確かめて、 **[送信]** をクリックします。
1.  **Azure Pipelines** の **[サブスクリプションの追加]** パネルで、 **[OK]** をクリックして確認メッセージを受け入れます。
1.  **Azure Pipelines** の **[サブスクリプションの表示]** パネルで、サブスクリプションのリストを確認し、パネルを閉じます。
1.  Azure DevOps ポータルの **Tailwind Traders** プロジェクトが表示されている Web ブラウザーに切り替え、Azure DevOps ポータルの一番左側にある垂直メニュー バーで **[パイプライン]** をクリックし、[パイプライン] ペインで、 **[Website-CI]** エントリをクリックします。その後、[Website-CI] ペインの使用中に、 **[パイプラインの実行] > [実行]** をクリックします。 
1. Teams チャネルでは、想定される動作として、パイプラインの失敗した実行に関する通知が投稿されます (パイプラインがセットアップされていない)。


## <a name="review"></a>確認

このラボでは、Azure DevOps サービスと Microsoft Teams 間の統合シナリオを実装しました。
