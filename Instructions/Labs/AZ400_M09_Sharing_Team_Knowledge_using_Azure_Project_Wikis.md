---
lab:
  title: ラボ 22:Azure プロジェクトの Wiki を使用してチームの知識を共有
  module: 'Module 09: Implement continuous feedback'
ms.openlocfilehash: f553c6451b6a152d4ddb8e81407d12bb045abeb9
ms.sourcegitcommit: f72fcf5ee578f465b3495f3cf789b06c530e88a4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2022
ms.locfileid: "139262521"
---
# <a name="lab-22-sharing-team-knowledge-using-azure-project-wikis"></a>ラボ 22:Azure プロジェクトの Wiki を使用してチームの知識を共有
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-overview"></a>ラボの概要

このラボでは、マークダウンのコンテンツの管理やマーメイド ダイアグラムの作成など、Azure DevOps で Wiki を作成および構成します。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- Azure プロジェクトで Wiki を作成する
- Markdown を追加および編集する
- マーメイド ダイアグラムを作成する

## <a name="lab-duration"></a>ラボの所要時間

-   推定時間:**45 分**

## <a name="instructions"></a>Instructions

### <a name="before-you-start"></a>開始する前に

#### <a name="sign-in-to-the-lab-virtual-machine"></a>ラボ仮想マシンにサインインする

必ず次の資格情報を使用して Windows 10 コンピューターにサインインしてください。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### <a name="review-the-installed-applications"></a>インストールされているアプリケーションを確認する

Windows デスクトップでタスクバーを見つけます。 タスク バーには、このラボで使用するアプリケーションのアイコンが含まれています。
    
-   Microsoft Edge

#### <a name="set-up-an-azure-devops-organization"></a>Azure DevOps 組織を設定します。 

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)で利用できる手順に従って作成してください。

### <a name="exercise-0-configure-the-lab-prerequisites"></a>演習 0:ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、Azure DevOps Demo Generator テンプレートおよび Microsoft Teams で作成したチームに基づいて事前に構成された **Tailwind Traders** チーム プロジェクトから成っています。

#### <a name="task-1-configure-the-team-project"></a>タスク 1:チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用し、**Tailwind Traders** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。 このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**:サイトの詳細については、 https://docs.microsoft.com/en-us/azure/devops/demo-gen を参照してください。

1.  「**サインイン**」をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。
1.  必要な場合は、「**Azure DevOps Demo Generator**」ページで「**承諾する**」をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。
1.  「**新しいプロジェクトの作成**」ページの 「**新しいプロジェクト名**」テキストボックスに、「**Azure プロジェクト Wiki を使用してチームの知識を共有する**」と入力し、「**組織の選択**」ドロップダウン リストで、Azure DevOps 組織を選択して、「**テンプレートの選択**」をクリックします。
1.  テンプレートの一覧 で 「**Tailwind Traders**」テンプレートを選択 し、「**テンプレートの選択**」をクリックします。
1.  再び 「**新しいプロジェクトの作成**」ページで、欠落している拡張機能をインストールするよう指示されたら 「**ARM 出力**」の下にあるチェックボックスを選択し、「**プロジェクトの作成**」をクリックします。

    > **注**:プロセスが完了するまでお待ちください。 これには 2 分ほどかかります。 プロセスが失敗した場合は、Azure DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

### <a name="exercise-1-publish-code-as-wiki"></a>演習 1:コードを Wiki として公開する

この演習では、Azure DevOps リポジトリを Wiki として公開し、公開された Wiki を管理する手順を説明します。

> **注**:Git リポジトリに保持しているコンテンツは、Azure DevOps wiki に公開できます。 たとえば、ソフトウェア開発キット、製品ドキュメント、またはREADME ファイルをサポートするために作成されたコンテンツは、Wiki に直接公開できます。 同じ Azure DevOps チーム プロジェクト内で複数の Wiki を公開するオプションがあります。

#### <a name="task-1-publish-a-branch-of-an-azure-devops-repo-as-wiki"></a>タスク 1:Azure DevOps リポジトリのブランチを wiki として公開する

このタスクでは、Azure DevOps リポジトリのブランチを wiki として公開します。

> **注**:公開された Wiki が製品バージョンに対応している場合は、製品の新しいバージョンをリリースするときに新しいブランチを公開できます。 

1.  左側の垂直メニューで 「**リポジトリ**」をクリックし、「**ファイル**」ペインの上部で **TailwindTraders-Website** リポジトリが選択されていることを確認します (上部のドロップダウンから Git アイコンで選択します)。 ブランチのドロップダウン リスト (ブランチ アイコンのある "ファイル" の上) で、「**メイン**」を選択し、メイン ブランチのコンテンツを確認します。
1.  「**ファイル**」ペインの左側にあるリポジトリフォルダーとファイル階層のリストで、「**ドキュメント」** フォルダーとその 「**画像**」サブフォルダーを展開し、「**画像**」サブフォルダーで、**Website.png** エントリを見つけ、マウス ポインターを右端に置いて 「**その他**」メニューを表す縦の省略記号 (3 つのドット) を表示し、「**その他**」をクリックし、ドロップダウン メニューで 「**ダウンロード**」をクリックして、**Website.png** ファイルをラボ コンピューターのローカルの **ダウンロード** フォルダーにダウンロードします。 

    >**注**:このラボの次の演習では、この画像を使用します。

1.  左側の垂直メニューで **[概要]** をクリックし、 **[概要]** セクションで **[Wiki]** を選択して、 *[コードを wiki として発行]* を選択します。
1.  「**コードを Wiki として公開**」ペインで、次の設定を指定して 「**公開**」をクリックします。

    | 設定 | 値 |
    | ------- | ----- |
    | リポジトリ | **TailwindTraders-Website** |
    | [Branch]\(ブランチ) | **main** |
    | Folder | **/Documents** |
    | Wiki 名 | **Tailwind Traders (Documents)** |

    >**注**:これは、自動的に **GitHubAction.md** ファイルの内容を表示します。

1.  **GitHubActions** ファイルのコンテンツを確認し、基になるリポジトリの構造と一致する wiki の全体的な構造に注意してください。

#### <a name="task-2-manage-content-of-a-published-wiki"></a>タスク 2:公開された Wiki のコンテンツを管理する

このタスクでは、前のタスクで公開した Wiki のコンテンツを管理します。

1.  左側の垂直メニューで 「**リポジトリ**」をクリックし、「**ファイル**」ペインの上部にあるドロップダウンメ ニューに **TailwindTraders-Website** リポジトリと **メイン** ブランチが表示されていることを確認します。リポジトリ フォルダー階層で「**ドキュメント**」フォルダーを選択し、右上隅にある 「**新規**」をクリックして、ドロップダウン メニューで、「**ファイル**」をクリックします。 
1.  **[新しいファイル]** パネルの **[新しいファイル名]** で、 **/Documents/** プレフィックスの後に「 **.order**」と入力し、 **[作成]** をクリックします。
1.  **[.order]** ペインの **[コンテンツ]** タブで、次のように入力し、 **[コミット]** をクリックします。

    ```text
    GitHubActions
    Images
    ```

1.  「**コミット** ペインで、「**コミット**」をクリックします。
1.  左側の垂直メニューで 「**概要**」をクリックし、「**概要**」セクションで 「**Wiki**」を選択し、ペインの上部に **Tailwind Traders (ドキュメント)** が表示されることを確認して、Wiki コンテンツの順序を確認します。 

    >**注**:Wiki コンテンツの順序は、 **.order** ファイルに一覧表示されているファイルとフォルダーの順序と一致している必要があります。

1.  左側の垂直メニューで 「**リポジトリ**」をクリックし、「**ファイル**」ペインの上部にあるドロップダウン メニューに **TailwindTraders-Website** リポジトリと **メイン** ブランチが表示されていることを確認し、ファイルのリストで 「**ドキュメント**」の下の 「**GitHubActions.md**」を選択し、「**GitHubActions.md**」ペインで 「**編集**」をクリックします。 
1.  **[GitHubActions.md]** ペインの `#GitHub Actions` ヘッダーのすぐ下に、**Documents** フォルダー内の画像の 1 つを参照する次のマークダウン要素を追加します。

    ```
    ![Tailwind Traders Website](Images/Website.png)
    ```    

1.  「**GitHubActions.md**」ペインで、「**コミット**」をクリックし、「**コミット**」ペインで 「**コミット**」をクリックします。
1.  「**GitHubActions.md**」ペインの 「**プレビュー**」タブで、画像が表示されていることを確認します。
1.  左側の垂直メニューで 「**概要**」をクリックし、「**概要**」セクションで 「**Wiki**」を選択し、ペインの上部に **Tailwind Traders (ドキュメント)** が表示されていること、および 「**GitHubActions**」ペインのコンテンツに新しく参照された画像が含まれていることを確認します。 

### <a name="exercise-2-create-and-manage-a-project-wiki"></a>演習 2:プロジェクト Wiki の作成と管理

この演習では、プロジェクト Wiki の作成と管理について説明します。

> **注**:既存のリポジトリとは別に、Wiki を作成および管理できます。 

#### <a name="task-1-create-a-project-wiki-including-a-mermaid-diagram-and-an-image"></a>タスク 1:マーメイド図と画像を含むプロジェクト Wiki を作成する

このタスクでは、プロジェクト Wiki を作成し、それにマーメイド図と画像を追加します。

1.  ラボのコンピューターで、**Tailwind Traders (Documents)** wiki のコンテンツが選択されている **Azure プロジェクトの Wiki を使用してチームの知識を共有する** プロジェクトの **Wiki のペイン** を表示している Azure DevOps ポータルの、ペイン上部の **[Tailwind Traders (Documents)]** ドロップダウン リスト ヘッダーをクリックし、 **[新しいプロジェクト Wiki を作成する]** を選択します。 
1.  「**ページ タイトル**」テキスト ボックスに、「**プロジェクト設計**」と入力します。
1.  ページの本文にカーソルを置き、ヘッダー設定を表すツールバーの左端のアイコンをクリックし、ドロップダウン リストで 「**ヘッダー 1**」をクリックします。 これにより、行の先頭にハッシュ文字 ( **#** ) が自動的に追加されます。
1.  新しく追加された **#** 文字の直後に、「**認証と認可**」と入力し、**Enter** キーを押します。
1.  ヘッダー設定を表すツールバーの左端のアイコンをクリックし、ドロップダウン リストで 「**ヘッダー 2**」をクリックします。 これにより、行の先頭にハッシュ文字 ( **##** ) が自動的に追加されます。
1.  新しく追加された **##** 文字の直後に、「**Azure DevOps OAuth 2.0 認証フロー**」と入力し、**Enter** キーを押します。

1.  次のコードを **コピーして貼り付け**、マーメイド図を Wiki に挿入します。

    ```
    ::: mermaid
    sequenceDiagram
     participant U as User
     participant A as Your app
     participant D as Azure DevOps
     U->>A: Use your app
     A->>D: Request authorization for user
     D-->>U: Request authorization
     U->>D: Grant authorization
     D-->>A: Send authorization code
     A->>D: Get access token
     D-->>A: Send access token
     A->>D: Call REST API with access token
     D-->>A: Respond to REST API
     A-->>U: Relay REST API response
    :::
    ```

    >**注**:マーメイド構文の詳細については、「[マーメイドについて](https://mermaid-js.github.io/mermaid/#/)」を参照してください。

1.  エディター ペインの右側にある、プレビュー ペインで、「**ダイアグラムのロード**」をクリックして、結果を確認します。 

    >**注**:出力は、[OAuth 2.0を使用してREST API へのアクセスを承認する](https://docs.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/oauth?view=azure-devops)方法を示すフローチャートのようになります。

1.  ます。エディター ペインの右上隅にある 「**保存**」ボタンの横にある下向きのキャレットをクリックし、ドロップダウン メニューで、「**リビジョン メッセージとともに保存**」をクリックします。 
1.  「**保存ページ**」ダイアログ ボックスで、「**OAuth 2.0 マーメイド図を使用した認証と承認セクション**」と入力し、「**保存**」をクリックします。
1.  **プロジェクト設計** エディター ペインで、このタスクの前半で追加したマーメイド要素の最後にカーソルを置き、**Enter** キーを押して行を追加し、ヘッダー設定を表すツールバーの左端のアイコンをクリックし、ドロップダウン リストで、「**ヘッダー 2**」をクリックします。 これにより、行の先頭にダブルハッシュ文字 ( **##** ) が自動的に追加されます。
1.  新しく追加された **##** 文字の直後に、「**ユーザー インターフェイス**」と入力し、**Enter** キーを押します。
1.  **プロジェクト設計** エディター ペインのツールバーで、「**ファイルの挿入**」アクションを表すペーパー クリップ アイコンをクリックし、「**開く**」ダイアログ ボックスで 「**ダウンロード**」フォルダーに移動し、前の演習でダウンロードした **Website.png** ファイルを選択して 「**開く**」をクリックします。
1.  **プロジェクト設計** エディター ペインに戻り、プレビュー ペインを確認して、画像が正しく表示されていることを確認します。
1.  ます。エディター ペインの右上隅にある 「**保存**」ボタンの横にある下向きのキャレットをクリックし、ドロップダウン メニューで、「**リビジョン メッセージとともに保存**」をクリックします。 
1.  「**ページの保存**」ダイアログ ボックスで、「**Tailwind Traders イメージを含むユーザー インターフェイス セクション**」と入力し、「**保存**」をクリックします。
1.  エディター ペインに戻り、右上隅にある 「**閉じる**」をクリックします。 

#### <a name="task-2-manage-a-project-wiki"></a>タスク 2:プロジェクト Wiki の管理

このタスクでは、新しく作成されたプロジェクト Wiki を管理します。

>**注**:まず、Wiki ページへの最新の変更を元に戻します。

1.  ラボのコンピューターで、**プロジェクト設計** wiki のコンテンツが選択されている **Azure プロジェクトの Wiki を使用してチームの知識を共有する** プロジェクトの **Wiki のペイン** を表示している Azure DevOps ポータルの、右上隅で、縦の省略記号をクリックし、ドロップダウン メニューで **[リビジョンを表示]** をクリックします。
1.  「**リビジョン**」ペインで、最新の変更を表すエントリをクリックします。 
1.  表示されたペインで、ドキュメントの以前のバージョンと現在のバージョンの比較を確認し、「**元に戻す**」をクリックし、確認を求められたら、もう一度 「**元に戻す**」をクリックして、「**ページの参照**」をクリックします。 
1.  「**プロジェクト設計**」ペインに戻り、変更が正常に元に戻されたことを確認します。

    >**注**:ここで、プロジェクト Wiki に別のページを追加し、それを Wiki ホーム ページとして設定します。

1.  「**プロジェク設計**」ペインの左下隅にある 「 **+ 新しいページ**」をクリックします。
1.  ページ エディター ペインの 「**ページ タイトル**」テキストボックスに「**プロジェクト設計の概要**」と入力し、「**保存」** をクリックして、「**閉じる**」をクリックします。
1.  **プロジェクト設計** プロジェクト Wiki 内のページを一覧表示するペインに戻り、**プロジェクト設計概要** エントリを見つけ、マウス ポインターで選択し、**プロジェクト設計** ページ エントリの上にドラッグ アンド ドロップします。 
1.  **プロジェクト設計概要** エントリがトップレベル ページとして一覧表示され、ホーム アイコンがそれを Wiki ホーム ページとして指定していることを確認します。

## <a name="review"></a>確認

このラボでは、Markdown コンテンツの管理やマーメイド図の作成など、Azure DevOps で Wiki を作成および構成しました。