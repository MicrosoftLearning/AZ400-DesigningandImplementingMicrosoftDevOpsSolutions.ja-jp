---
lab:
    title: 'ラボ 18: Azure DevOps とチーム間の統合'
    module: 'モジュール 18: システム フィードバックのメカニズムの実装'
---

# ラボ 18: Azure DevOps とチーム間の統合
# 受講生用ラボ マニュアル

## ラボの概要

**[Microsoft Teams](https://teams.microsoft.com/start)** は、Office 365 でのチームワークのためのハブです。チームのあらゆるチャットやミーティング、ファイル、アプリを 1 箇所で管理して使用できます。Office 365 と Azure DevOps 全体でチーム、会話、コンテンツ、ツールのハブをソフトウェア開発チームに提供します。

このラボでは、Azure DevOps サービスと Microsoft Teams 間の統合シナリオを実装します。

> **注**: **Azure DevOps Services** を Microsoft Teams と統合すると、開発サイクルを通して包括的なチャットとコラボレーションを体験できます。チームは Azure DevOps チーム プロジェクトで、作業項目、プル要求、コードのコミット、ビルドおよびリリース イベントに関する通知とアラートにより、重要なアクティビティについて容易に情報を得られます。

## 目標

このラボを完了すると、次のことができるようになります。

- Microsoft Teams と Azure DevOps を統合する
- Azure DevOps Kanban ボードと Teams のダッシュボードを統合する
- Azure Pipelines と Microsoft Teams を統合する
- Azure Pipelines アプリを Microsoft Teams にインストールする
- Azure Pipelines 通知をサブスクライブする

## ラボの所要時間

-   推定時間: **60 分**

## 手順

### 開始する前に

#### ラボの仮想マシンにログインする

必ず次の資格情報を使用して Windows 10 コンピューターにサインインしてください。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### このラボで必要なアプリケーションのレビュー

このラボで使用するアプリケーションを特定:
    
-   Microsoft Edge
-   Microsoft Teams。このラボでは前提条件の一部としてインストールされます。

#### Office 365 サブスクリプションを設定 

[Microsoft Teams サインアップ ページ](https://teams.microsoft.com/start) から無料のトライアル サブスクリプションを作成します。 

#### Azure DevOps 組織を設定します。 

[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/ja-jp/azure/devops/organizations/accounts/create-organization?view=azure-devops) の手順に従います。Azure DevOps 組織を作成する際は、Office 365 サブスクリプションの設定で使用したものと同じユーザー アカウントを使ってサインインします。

> **注**: Office 365 サブスクリプションと Azure DevOps 組織は、同じ Azure Active Directory (Azure AD) テナントを共有する必要があります。

### 演習 0: ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、Azure DevOps Demo Generator テンプレートおよび Microsoft Teams で作成したチームに基づいて事前に構成された **Tailwind Traders** チーム プロジェクトから成っています。

#### タスク 1: チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用し、**Tailwind Traders** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**: サイトの詳細については、https://docs.microsoft.com/ja-jp/azure/devops/demo-gen をご覧ください。

1.  「**サインイン**」をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。
1.  必要な場合は、「**Azure DevOps Demo Generator**」ページで「**承諾する**」をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。
1.  「**新しいプロジェクトの作成**」ページで「**テンプレートの選択**」をクリックします。
1.  テンプレートのリストのツールバーで「**全般**」をクリックし、「**Tailwind Traders**」を選択して「**テンプレートの選択**」をクリックします。
1.  「**新しいプロジェクトの作成**」ページで「**新しいプロジェクト名**」テキストボックスに「**Tailwind Traders**」と入力します。「**組織の選択**」ドロップダウン リストで Azure DevOps 組織を選択します。
1.  **新しいプロジェクトの作成**ページで、欠落している拡張機能をインストールするよう指示されたら、**「ARM 出力」** の下のチェックボックスを選択して、**「プロジェクトの作成」** をクリックします (GitHub フォークを無視します)。

    > **注**: プロセスが完了するまでお待ちください。通常は 2 分ほどかかります。プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

#### タスク 2: Microsoft Teams でチームを作成する

このタスクでは、Microsoft Teams でチームを作成します。

1.  ラボのコンピューターで Web ブラウザーを起動して [Microsoft Teams ダウンロード ページ](https://www.microsoft.com/ja-jp/microsoft-365/microsoft-teams/download-app) に移動します。そこから既定の設定で Microsoft Teams をダウンロードしてインストールします。 
1.  ラボのコンピューターでデスクトップ アプリを使い、**Microsoft Teams** を起動します。

    > **注**: Web ブラウザーを使用して [Microsoft Teams 起動ページ](https://teams.microsoft.com/dl/launcher/launcher.html?url=/_%23/l/home/0/0&type=home) に移動することもできます

1.  サインインするよう指示されたら、Office 365 サブスクリプションの一部であり、Azure DevOps 組織へのアクセス権限があるユーザー アカウントを使ってサインインします。
1.  Microsoft Teams のページ左側のツールバーで「**Teams**」をクリックし、チーム一覧の最下部で「**チームに参加、またはチームを作成**」をクリックします。

    >**注**: チームは、共通の目標を掲げて集まる人々のグループです。 

1.  「**チームに参加、またはチームを作成**」ペインで「**チームを作成**」をクリックします。
1.  「**チームを作成**」パネルで「**初めから作成**」をクリックします。「**チームの種類は何でしょうか?**」パネルで「**プライベート**」をクリックします。
1.  「**プライベート チームに関する簡単な詳細**」パネルで「**チームに名前を付ける**」を「**Tailwind Traders**」に置き換えて「**作成**」をクリックします。
1.  「**Tailwind Traders にメンバーを追加**」パネルで「**スキップ**」をクリックします。

### 演習 1: Azure Boards に Microsoft Teams を統合する

この演習では、Azure Boards と Microsoft Teams 間で統合を実装します。

#### タスク 1: Microsoft Teams で Azure Boards アプリをインストールして構成する

このタスクでは、Microsoft Teams で新しく作成されたチームで Azure Boards アプリをインストールして構成します。

1.  Microsoft Teams ウィンドウの左下コーナーで「**アプリ**」アイコンをクリックします。「**アプリ**」ペインが開きます。
1.  「**アプリ**」ペインの「**すべてのアプリを検索**」テキストボックスで「**Azure Boards**」と入力し、アプリ一覧で「**Azure Boards**」をクリックします。
**「開く」** ボタンの右側の下向き矢印をクリックして、ドロップダウンで **「チームに追加」** エントリを選択します。
1.  **チーム用 Azure ボードのセットアップ** パネルの**検索**テキスト ボックスに **Tailwind Traders** と入力します。検索結果から **Tailwind Traders > 全般** エントリを選択して、**「ボットのセットアップ」** をクリックします。

1.  **Tailwind Traders** チームの**全般**チャンネルの投稿のリストから、**Azure Boards** というタイトルの付いた投稿を選択し、**Enter** キーを押して、ボットにより投稿された以下のような追加メッセージを確認します。

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
1.  **新しい会話**を開き、**@** と入力します。**Azure** と入力すると、結果がプロンプト表示されます。**Azure Boards** を選択し、コマンド パネルで**サインイン**します。手順に従い、Azure DevOps 組織にアクセスできることを確認します。
1.  Azure DevOps **Tailwind Traders** プロジェクトの URL をコピーします。例: https://dev.azure.com/myorg/myproject。 **新しい会話**を開き、Teams チャンネルに **@** と入力します。**Azure** と入力すると、結果がプロンプト表示されます。**Azure Boards** を選択し、コマンド パネルで**リンク**を選択します。この時点で、前にコピーしたプロジェクトの URL を貼り付けます。投稿は次のようになります。 
´**Azure Boards** link https://dev.azure.com/myorg/myproject ´。**Enter** キーを押します。

    次のような受け取ったメッセージを確認します。

    ```
    NAME has linked this channel to project  Tailwind Traders. Create work items using @Azure         Boards create.
    To monitor work items, please add subscription
    Add subscription
    ```

1.  **「サブスクリプションの追加」** をクリックします。ドロップダウンで、イベントのリストから、**作成した作業項目**を選択し、**「次へ」** をクリックします。既定値のままにして、**「送信」** をクリックします。**「OK」** をクリックして閉じます。新しく追加したサブスクリプションに関する詳細と共に、メッセージがチャンネルに投稿されます。
1.  Azure DevOps portal の **Tailwind Traders** プロジェクトを表示している Web ブラウザーに切り替え、**「ボード」 > 「作業項目」** をクリックします。**「新しい作業項目」** をクリックして、ドロップダウンで **「ユーザー ストーリー」** を選択します。**Teams を使って Azure Boards 統合をテストする** などのようなユース ストーリーにタイトルを付けて、**保存**します。当社が最近開発した Teams は、作成されたユーザー ストーリーに関する情報を含む通知/カードを投稿します。

#### タスク 2: Azure Boards Kanban ボードを Microsoft Teams に追加する

このタスクでは、Azure Boards Kanban ボードを Microsoft Teams のタブに追加します。

> **注**: かんばんボードは、バックログをインタラクティブな看板に変え、作業の視覚的なフローを提供します。作業が構想から完了まで進むにつれて、ボード上の項目を更新します。各列は作業段階を示し、各カードはその作業段階のユーザーのストーリー (青いカード) またはバグ (赤いカード) を表します。タブを使用することで、チームのかんばんボードまたはお気に入りのダッシュボードを直接、Microsoft Teams に取り込むことができます。**タブ** を使うと、チーム メンバーは専用のキャンバスで、チャネル内で、またはユーザーの個人的なアプリ スペースでサービスにアクセスできます。既存の Web アプリを利用して、Teams 内ですばらしいタブ経験を創造できます。

1.  ラボのコンピューターで、Azure DevOps ポータルで **Tailwind Traders** プロジェクトを表示している Web ブラウザーに切り替えます。Azure DevOps ポータルの一番左にある垂直メニュー バーで「**ボード**」をクリックし、「**ボード**」セクションで「**ボード**」をクリックします。
1.  「**ボード**」パネルの「**マイ チーム ボード (1)**」セクションで、「**Tailwind Traders Team ボード**」エントリをクリックします。 
1.  「**Tailwind Traders Team**」ペインが表示されている間に、Web ブラウザー ウィンドウでその URL をクリップボードにコピーします。
1.  Microsoft Teams ウィンドウに切り替え、新しい作成した **Tailwind Traders** チームの**全般**チャンネルが選択されていることを確認します。**全般**ペインの上部セクションで、**投稿、ファイル、Wiki タブ**の横のプラス記号をクリックします。「**タブの追加**」パネルが表示されます。
1.  「**タブの追加**」パネルで「**Web サイト**」をクリックします。「**Web サイト**」パネルで「**タブ名**」を「**Tailwind Traders Team ボード**」に設定し、「**URL**」をクリップボードにコピーしたばかりの URL に設定してから「**保存**」をクリックします。
1.  **Tailwind Traders** チームの **全般** チャンネルが選択された状態の Microsoft Teams ウィンドウの トップ メニューのタブのリストで、新しい追加された **Tailwind Traders Team ボード** タブをクリックして、**Tailwind Traders Team** ボードが Azure DevOps portal で利用可能になったことを示すコンテンツが含まれていることを確認します (サインインする必要があります)。

> **注**: 毎日の立ち上げ中にすべての作業を監視できます。該当する作業項目の状態が変化した場合、更新はリアルタイムで反映されます。かんばんボードを Microsoft Teams から変更するオプションもあります。

### 演習 2: Azure Pipelines と Microsoft Teams を統合する

この演習では、Azure Pipelines と Microsoft Teams の統合を実装します。

#### タスク 1: Microsoft Teams で Azure Pipelines アプリをインストールして構成する

このタスクでは、Microsoft Teams で指定されたチームで Azure Pipelines アプリをインストールして構成します。

> **注**: Microsoft Teams の Azure Pipelines アプリを使うと、パイプラインのイベントを監視できます。リリース、保留中の承認、完了したビルドなどのイベントに対するサブスクリプションを Teams のチャンネルに通知を直接投稿するなどで設定し、管理できます。また、Teams チャネル内からリリースを承認することも可能です。

1.  Microsoft Teams ウィンドウに移動し、左下隅で「**アプリ**」アイコンをクリックします。「**アプリ**」ペインが開きます。
1.  「**アプリ**」ペインの「**すべてのアプリを検索**」テキストボックスで「**Azure Pipelines**」と入力し、アプリ一覧で「**Azure Pipelines**」をクリックします。
1.  「**Azure Pipelines**」パネルで、「**開く**」ボタンのすぐ右にある下向き矢印をクリックします。ドロップダウン リストで「**チームに追加**」エントリをクリックします。
1.  「**チームのために Azure Pipelines を設定**」パネルの「**検索**」テキスト ボックスに「**Tailwind Traders**」と入力します。結果リストで **「Tailwind Traders」 > 「全般」** エントリを選択し、「**ボットを設定**」をクリックします。自動的に **Tailwind Traders** チームの **全般** チャネルの投稿ビューにリダイレクトされます。
1.  **Tailwind Traders** チームの**全般**チャンネルの投稿のリストで、**新しい会話**を開いて、**@** を入力します。**Azure** と入力すると、結果がプロンプト表示されます。**Azure Pipelines** を選択し、**Enter** キーを押して、ボットにより投稿された以下のような追加メッセージを確認します。

    ```
    Here are some of the things you can do:
    subscribe [pipeline url/ project url] - Subscribe to a pipeline or all pipelines in a project to receive notifications
    subscriptions - Add or remove subscriptions for this channel
    feedback - Report a problem or suggest a feature
    signin - Sign in to your Azure Pipelines account
    signout - Sign out from your Azure Pipelines account
    To know more see documentation.
    ```
   
#### タスク 2: Microsoft Teams で Azure Pipelines 通知にサブスクライブする

このタスクで、Microsoft Teams で Azure Pipelines 通知にサブスクライブします

> **注**: `@Azure Pipelines` ハンドルを使用して、アプリの操作を開始できます。

1.  **投稿**タブが選択された状態で、**Tailwind Traders** チームの**全般**チャンネルで、**@** と入力します。**Azure** と入力すると、結果がプロンプト表示されます。**Azure Pipelines** を選択して、コマンド パネルで**サインイン**して、**Enter** キーを押します。
1.  「**Azure Pipelines サインイン**」ペインで「**サインイン**」をクリックします。
1.  **サービス フック (読み取りと書き込み)**、**ビルド (実行のビルド)**、**リリース (読み取り、書き込み、実行、管理)**、**プロジェクトおよびチーム (読み取り)**、**ID ピッカー (読み取り)**、**Teams 統合**許可を与えるよう指示されたら、「**承諾**」をクリックしてから「**閉じる**」をクリックします。

    >**注**: これで `@azure pipelines subscribe [pipeline url]` コマンドを実行して Azure DevOps パイプラインにサブスクライブできます。

1.  Azure DevOps portal で **Tailwind Traders** プロジェクトを表示している Web ブラウザーに切り替え、Azure DevOps portal の一番左側の垂直メニュー バーで、**「パイプライン」** をクリックして、**パイプライン** ペインで、**Website-CI** エントリをクリックします。Web ブラウザー ウィンドウでの **Website-CI** ペインで、その URL をクリップボードにコピーします。

    > **注**: URL は、`https://dev.azure.com/<organization_name>/Tailwind%20Traders/_build?definitionId=<number>` の形式になります。`<organization_name>` は、DevOps 組織の名前を示すプレースホルダーです。

    >**注**: パイプライン URL は、*definitionId* または *buildId/releaseId* が URL に含まれているパイプライン内のどのページでも構いません。

1.  Teams に戻り、**投稿**タブが選択された状態で、**Tailwind Traders** チームの**全般**チャンネルで、**@** と入力します。**Azure** と入力すると、結果がプロンプト表示されます。**Azure Pipelines** を選択して、コマンド パネルで**サブスクライブ**します。この時点で、前にコピーしたパイプラインの URL を貼り付けます。投稿は次のようになります、`@Azure Pipelines subscribe https://dev.azure.com/<organization_name>/Tailwind%20Traders/_build?definitionId=<number>` (`<organization_name>` プレースホルダーを DevOps 組織の名前に置き換えます)。**Enter** キーを押します。
1.  サブスクリプションが作成されたという通知を待ちます。
 
    >**注**: ビルド パイプラインで、チャネルは「**実行ステージの状態変更**」と「**実行ステージは承認待ち**」という通知にサブクライブします。

1.  Azure DevOps portal で **Tailwind Traders** プロジェクトを表示している Web ブラウザーに切り替え、Azure DevOps portal の一番左側の垂直メニュー バーで、**「パイプライン」** をクリックして、**パイプライン** セクションで、リリースのリストで、「**リリース**」をクリックして、**Website-CD** エントリをクリックします。**Website-CD** エントリが選択された状態で、Web ブラウザー ウィンドウで、その URL をクリップボードにコピーします。

    >**注**: URL は、`https://dev.azure.com/<organization_name>/Tailwind%20Traders/_release?_a=releases&view=mine&definitionId=2` の形式になります。`<organization_name>` は、DevOps 組織の名前を示すプレースホルダーです。

1.  **投稿**タブが選択された状態で、**Tailwind Traders** チームの**全般**チャンネルで、**@** と入力します。**Azure** と入力すると、結果がプロンプト表示されます。**Azure Pipelines** を選択して、コマンド パネルで**サブスクライブ**します。この時点で、前にコピーしたリリースの URL を貼り付けます。投稿は次のようになり、`@Azure Pipelines subscribe https://dev.azure.com/<organization_name>/Tailwind%20Traders/_release?_a=releases&view=mine&definitionId=2`、サブスクライブして、パイプラインをリリースします (`<organization_name>` プレースホルダーを DevOps 組織の名前に置き換えてください)。**Enter** キーを押します。

    >**注**: リリース パイプラインで、チャネルは「**デプロイ開始**」、「**デプロイ完了**」、「**デプロイ承認待ち**」通知にサブスクライブします。
 
#### タスク 3: フィルターを使用し、Microsoft Teams での Azure Pipelines へのサブスクリプションをカスタマイズする

このタスクでは、フィルターを使い、Microsoft Teams での Azure Pipelines へのサブスクリプションをカスタマイズします。

>**注**: ユーザーがいずれかのパイプラインにサブスクライブすると、既定によりいくつかのサブスクリプションが作成され、フィルターは適用されません。ユーザーは頻繁にサブスクリプションをカスタマイズする必要があります。たとえば、ユーザーはビルドが失敗した場合のみ、またはデプロイが運用環境にプッシュされる場合にのみ通知を希望する可能性があります。Azure Pipelines アプリは、チャネルの表示内容をカスタマイズするフィルターをサポートします。

>**注**: `@Azure Pipelines subscriptions` コマンドを使用すると、サブスクリプションを一覧表示した管理できます。このコマンドでは、当該チャネルの現在のサブスクリプションすべてを一覧表示します。

1.  **投稿**タブが選択された状態で、**Tailwind Traders** チームの**全般**チャンネルで、**@** と入力します。**Azure** と入力すると、結果がプロンプト表示されます。**Azure Pipelines** を選択して、**「サブスクリプション」** を選択して、**Enter** キーを押します。**Azure Pipelines** ボットからの応答で、**「サブスクリプションの追加」** を選択します。
1.  「**Azure Pipelines** **サブスクリプションの追加**」パネルの「**イベントの選択**」ドロップダウン リストで「**ビルド完了**」が選択されていることを確認し、「**次へ**」をクリックします。
1.  「**Azure Pipelines** **サブスクリプションの追加**」パネルの「**パイプラインの選択**」ドロップダウン リストで「**Website-CI**」が選択されていることを確認し、「**次へ**」をクリックします。
1.  「**Azure Pipelines** **サブスクリプションの追加**」パネルの「**ビルドの状態**」ドロップダウン リストで「**任意**」が選択されていることを確認し、「**送信**」をクリックします。
1.  「**Azure Pipelines** **サブスクリプションの追加**」パネルで「**OK**」をクリックし、確定メッセージを認識します。
1.  「**Azure Pipelines** **サブスクリプションの表示**」パネルで、サブスクリプションのリストをレビューし、パネルを閉じます。
1.  Azure DevOps portal で **Tailwind Traders** プロジェクトを表示している Web ブラウザーに切り替え、Azure DevOps portal の一番左側の垂直メニュー バーで、**「パイプライン」** をクリックして、パイプライン ペインで、**Website-CI** エントリをクリックします。Web ブラウザー ウィンドウでの Website-CI ペインで、**「パイプラインの実行」 > 「実行」** をクリックします。 
1.  Teams チャンネルは、予測された動作として、パイプラインの失敗した実行に関する通知を投稿します (パイプラインの設定が欠如しているため)。


## 確認

このラボでは、Azure DevOps サービスと Microsoft Teams 間の統合シナリオを実装しました。
