---
lab:
  title: Azure プロジェクトの Wiki を使用してチームの知識を共有する
  module: 'Module 08: Implement continuous feedback'
---

# Azure プロジェクトの Wiki を使用してチームの知識を共有する

## 受講生用ラボ マニュアル

## ラボの要件

- このラボには、**Microsoft Edge** または [Azure DevOps 対応ブラウザー](https://docs.microsoft.com/azure/devops/server/compatibility)が必要です。

- **Azure DevOps 組織を設定する:** このラボで使える Azure DevOps 組織がまだない場合は、[AZ-400 ラボの前提条件](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html)に関するページの手順に従って作成してください。

- **サンプルの eShopOnWeb プロジェクトを設定する:** このラボで使用できるサンプルの eShopOnWeb プロジェクトがまだない場合は、[AZ-400 ラボの前提条件](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html)に関するページに記載されている手順に従って作成してください。

## ラボの概要

このラボでは、マークダウンのコンテンツの管理やマーメイド ダイアグラムの作成など、Azure DevOps で Wiki を作成および構成します。

## 目標

このラボを完了すると、次のことができるようになります。

- Azure プロジェクトで Wiki を作成する。
- マークダウンを追加および編集する。
- マーメイド ダイアグラムを作成する。

## 推定時間:45 分

## Instructions

### 演習 0:ラボの前提条件の構成

この演習では、ラボの前提条件の検証、Azure DevOps 組織の準備が整っていること、および eShopOnWeb プロジェクトを作成したことについてもう一度確認してください。 詳細については上記の手順を参照してください。

### 演習 1:コードを Wiki として公開する

この演習では、Azure DevOps リポジトリを Wiki として公開し、公開された Wiki を管理する手順を説明します。

> **注**:Git リポジトリに保持しているコンテンツは、Azure DevOps wiki に公開できます。 たとえば、ソフトウェア開発キット、製品ドキュメント、またはREADME ファイルをサポートするために作成されたコンテンツは、Wiki に直接公開できます。 同じ Azure DevOps チーム プロジェクト内で複数の Wiki を公開するオプションがあります。

#### タスク 1:Azure DevOps リポジトリのブランチを wiki として公開する

このタスクでは、Azure DevOps リポジトリのブランチを wiki として公開します。

> **注**:公開された Wiki が製品バージョンに対応している場合は、製品の新しいバージョンをリリースするときに新しいブランチを公開できます。

1. 左側の垂直メニューで **[リポジトリ]** をクリックし、**[ファイル]** ペインの上部セクションで **[eShopOnWeb]** リポジトリが選択されていることを確認します (上部のドロップダウンから Git アイコンで選択します)。 ブランチのドロップダウン リスト (ブランチ アイコンのある "ファイル" の上) で、**[メイン]** を選択し、メイン ブランチのコンテンツを確認します。
1. **[ファイル]** ペインの左側にあるリポジトリ フォルダーとファイル階層の一覧で、 **[src]** フォルダーを展開して、 **[Web] > [wwwroot] > [images]** サブフォルダーを参照します。 **[Images]** サブフォルダーで **brand.png** エントリを見つけ、その右端をポイントして **[その他]** メニューを表す縦の省略記号 (3 つのドット) を表示し、 **[ダウンロード]** をクリックして、**brand.png** ファイルをラボ コンピューターのローカルの **[ダウンロード]** フォルダーにダウンロードします。

    >**注**:このラボの次の演習では、この画像を使用します。

1. Wiki ソース ファイルは、Repos の現在のフォルダー構造内の別のフォルダーに格納します。 **Repos** 内から、 **[ファイル]** を選びます。 フォルダー構造の上部に **eShopOnWeb** というリポジトリ タイトルがあることに注意してください。 **省略記号 (3 つのドット) を選び**、 **[新規] > [フォルダー]** を選んで、[新しいフォルダー名] のタイトルとして「**ドキュメント**」を指定します。 リポジトリでは空のフォルダーを作成できないため、[新しいファイル名] として「**READ.ME**」を指定します。
1. **[作成] ボタンを選んで**、フォルダーとファイル の作成を確認します。
1. READ.ME ファイルが組み込みのビュー モードで開きます。 これは "コードとして" 格納されるため、 **[コミット]** ボタンをクリックして変更を**コミットする**必要があります。 [コミット] ウィンドウで、もう一度 **[コミット]** を選んで確認します。
1. Azure DevOps の左側の垂直メニューで **[概要]** をクリックし、 **[概要]** セクションで **[Wiki]** を選んで、 *[Wiki としてコードを公開する]* を選びます。
1. **[Wiki としてコードを公開する]** ペインで、次の設定を指定して **[公開]** をクリックします。

    | 設定 | 値 |
    | ------- | ----- |
    | リポジトリ | **eShopOnWeb** |
    | [Branch]\(ブランチ) | **main** |
    | Folder | **/Documents** |
    | Wiki 名 | **eShopOnWeb (ドキュメント)** |

    >**注**: これにより、Wiki セクションが自動的に開き、**エディター**が発行されます。ここで、Wiki ページのタイトルを指定したり、実際のコンテンツを追加したりできます。 MarkDown 形式を使うことが推奨されることに注意してください。リボンを使うといくつかの MarkDown レイアウト構文に役立ちます。

1. Wiki ページの **[タイトル]** フィールドに、「オンライン小売店へようこそ!」と入力します。

1. Wiki ページの本文に、次のテキストを貼り付けます。

    ```text
    ##Welcome to Our Online Retail Store!
    At our online retail store, we offer a **wide range of products** to meet the **needs of our customers**. Our selection includes everything from *clothing and accessories to electronics, home decor, and more*.
    
    We pride ourselves on providing a seamless shopping experience for our customers. Our website offers the following benefits:
    1. user-friendly,
    1. and easy to navigate, 
    1. allowing you to find what you're looking for,
    1. quickly and easily. 
    
    We also offer a range of **_payment and shipping options_** to make your shopping experience as convenient as possible.
    
    ### about the team
    Our team is dedicated to providing exceptional customer service. If you have any questions or concerns, our knowledgeable and friendly support team is always available to assist you. We also offer a hassle-free return policy, so if you're not completely satisfied with your purchase, you can easily return it for a refund or exchange.
    
    ### Physical Stores
    |Location|Area|Hours|
    |--|--|--|
    | New Orleans | Home and DIY  |07.30am-09.30pm  |
    | Seattle | Gardening | 10.00am-08.30pm  |
    | New York | Furniture Specialists  | 10.00am-09.00pm |
    
    ## Our Store Qualities
    - We're committed to providing high-quality products
    - Our products are offered at affordable prices 
    - We work with reputable suppliers and manufacturers 
    - We ensure that our products meet our strict standards for quality and durability. 
    - Plus, we regularly offer sales and discounts to help you save even more.
    
    #Summary
    Thank you for choosing our online retail store for your shopping needs. We look forward to serving you!
    ```

1. このサンプル テキストを見ると、タイトルとサブタイトル (##)、太字 (* *)、斜体 (* )、テーブルの作成方法など、MarkDown 構文のいくつかの一般的な機能の概要がわかります。

1. 終わったら、右上隅にある **[保存]** ボタンを選びます。

1. ブラウザーを**更新**するか、他の DevOps ポータル オプションを選んで Wiki セクションに戻ります。 **EshopOnWeb (ドキュメント)** Wiki が表示され、Wiki の **HomePage** として **[オンライン小売店へようこそ!]** と表示されていることに注意してください。

#### タスク 2:公開された Wiki のコンテンツを管理する

このタスクでは、前のタスクで公開した Wiki のコンテンツを管理します。

1. 左側の縦型メニューで **[リポジトリ]** をクリックし、**[ファイル]** ペインの上部のドロップダウン メニューに **eShopOnWeb** リポジトリと**メイン** ブランチが表示されていることを確認します。リポジトリ フォルダー階層で **[ドキュメント]** フォルダーを選び、**Welcome-to-our-Online-Retail-Store!.md** ファイルを選びます。
1. ここでは MarkDown 形式が生のテキスト形式として表示され、ここからもファイルの内容の編集を続けられることに注意してください。

> **注**: Wiki ソース ファイルはソース コードとして処理されるため、従来のソース管理からのすべての操作 (クローン、pull request、承認など) を Wiki ページにも適用できることを覚えておいてください。

### 演習 2:プロジェクト Wiki の作成と管理

この演習では、プロジェクト Wiki の作成と管理について説明します。

> **注**:既存のリポジトリとは別に、Wiki を作成および管理できます。

#### タスク 1:マーメイド図と画像を含むプロジェクト Wiki を作成する

このタスクでは、プロジェクト Wiki を作成し、それにマーメイド図と画像を追加します。

1. ラボ コンピューターの、**EShopOnweb** プロジェクトの **Wiki ペイン**が表示されている Azure DevOps ポータルで (**eShopOnWeb (ドキュメント)** Wiki のコンテンツが選ばれています)、ペイン上部の **eShopOnWeb (ドキュメント)** ドロップダウン リスト ヘッダー (下矢印アイコン) をクリックして、ドロップダウン リストで **[新しいプロジェクト Wiki を作成する]** を選びます。
1. **[ページ タイトル]** テキスト ボックスに、「**プロジェクト設計**」と入力します。
1. ページの本文にカーソルを置き、ヘッダー設定を表すツールバーの左端のアイコンをクリックし、ドロップダウン リストで **[ヘッダー 1]** をクリックします。 これにより、行の先頭にハッシュ文字 ( **#** ) が自動的に追加されます。
1. 新しく追加された **#** 文字の直後に、「**認証と認可**」と入力し、**Enter** キーを押します。
1. ヘッダー設定を表すツールバーの左端のアイコンをクリックし、ドロップダウン リストで **[ヘッダー 2]** をクリックします。 これにより、行の先頭にハッシュ文字 ( **##** ) が自動的に追加されます。
1. 新しく追加された **##** 文字の直後に、「**Azure DevOps OAuth 2.0 認証フロー**」と入力し、**Enter** キーを押します。
1. 次のコードを**コピーして貼り付け**、マーメイド図を Wiki に挿入します。

    ```text
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

    >**注**: マーメイド構文の詳細については、「[マーメイドについて](https://mermaid-js.github.io/mermaid/#/)」を参照してください。

1. エディター ペインの右側にある、プレビュー ペインで、**[ダイアグラムのロード]** をクリックして、結果を確認します。

    >**注**:出力は、[OAuth 2.0を使用してREST API へのアクセスを承認する](https://docs.microsoft.com/azure/devops/integrate/get-started/authentication/oauth)方法を示すフローチャートのようになります。

1. エディター ペインの右上隅にある **[保存]** ボタンの横にある下向きのキャレットをクリックし、ドロップダウン メニューで、**[リビジョン メッセージと一緒に保存する]** をクリックします。
1. **[ページの保存]** ダイアログ ボックスで、「**OAuth 2.0 マーメイド図を使用した認証と承認セクション**」と入力し、**[保存]** をクリックします。
1. **[プロジェクト設計]** エディター ペインで、このタスクの前半で追加したマーメイド要素の最後にカーソルを置き、**Enter** キーを押して行を追加し、ヘッダー設定を表すツールバーの左端のアイコンをクリックし、ドロップダウン リストで **[ヘッダー 2]** をクリックします。 これにより、行の先頭にダブルハッシュ文字 ( **##** ) が自動的に追加されます。
1. 新しく追加された **##** 文字の直後に、「**ユーザー インターフェイス**」と入力し、**Enter** キーを押します。
1. **[プロジェクト設計]** エディター ペインのツールバーで、 **[ファイルの挿入]** アクションを表すペーパー クリップ アイコンをクリックし、 **[開く]** ダイアログ ボックスで **[ダウンロード]** フォルダーに移動し、前の演習でダウンロードした **Brand.png** ファイルを選んで **[開く]** をクリックします。
1. **プロジェクト設計**エディター ペインに戻り、プレビュー ペインを確認して、画像が正しく表示されていることを確認します。
1. エディター ペインの右上隅にある **[保存]** ボタンの横にある下向きのキャレットをクリックし、ドロップダウン メニューで、**[リビジョン メッセージと一緒に保存する]** をクリックします。
1. **[ページの保存]** ダイアログ ボックスで、「**eShopOnWeb 画像を含むユーザー インターフェイス セクション**」と入力し **[保存]** をクリックします。
1. エディター ペインに戻り、右上隅にある **[閉じる]** をクリックします。

#### タスク 2:プロジェクト Wiki の管理

このタスクでは、新しく作成されたプロジェクト Wiki を管理します。

>**注**:まず、Wiki ページへの最新の変更を元に戻します。

1. ラボ コンピューターの、**eShopOnWeb** プロジェクトの **Wiki ペイン**が表示されている Azure DevOps ポータルで (**プロジェクト設計** Wiki のコンテンツが選択されています)、右上隅の縦の省略記号をクリックし、ドロップダウン メニューで **[リビジョンを表示する]** をクリックします。
1. **[リビジョン]** ペインで、最新の変更を表すエントリをクリックします。
1. 表示されたペインで、ドキュメントの以前のバージョンと現在のバージョンの比較を確認し、**[元に戻す]** をクリックし、確認を求められたら、もう一度 **[元に戻す]** をクリックして、**[ページの参照]** をクリックします。
1. **[プロジェクト設計]** ペインに戻り、変更が正常に元に戻されたことを確認します。

    >**注**:ここで、プロジェクト Wiki に別のページを追加し、それを Wiki ホーム ページとして設定します。

1. **[プロジェクト設計]** ペインの左下隅にある **[+ 新しいページ]** をクリックします。
1. ページ エディター ペインの **[ページ タイトル]** テキスト ボックスに「**プロジェクト設計の概要**」と入力し、**[保存]** をクリックして、**[閉じる]** をクリックします。
1. **プロジェクト設計**プロジェクト Wiki 内のページを一覧表示するペインに戻り、**プロジェクト設計概要**エントリを見つけ、マウス ポインターで選択し、**プロジェクト設計**ページ エントリの上にドラッグ アンド ドロップします。
1. 表示されるウィンドウの **[移動]** ボタンを選んで、変更を確認します。
1. **プロジェクト設計概要**エントリがトップレベル ページとして一覧表示され、ホーム アイコンがそれを Wiki ホーム ページとして指定していることを確認します。

## 確認

このラボでは、Markdown コンテンツの管理やマーメイド図の作成など、Azure DevOps で Wiki を作成および構成しました。
