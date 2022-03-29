---
lab:
  title: 'ラボ 10: LaunchDarkly と Azure DevOps を使用した機能フラグ管理'
  module: 'Module 04: Design and implement a release strategy'
ms.openlocfilehash: f2b752ebb825fea24c5eb1ba74a12929e5191022
ms.sourcegitcommit: f72fcf5ee578f465b3495f3cf789b06c530e88a4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2022
ms.locfileid: "139262604"
---
# <a name="lab-10-feature-flag-management-with-launchdarkly-and-azure-devops"></a>ラボ 10: LaunchDarkly と Azure DevOps を使用した機能フラグ管理
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-overview"></a>ラボの概要

[**LaunchDarkly**](https://launchdarkly.com/) は、サービスとして機能フラグを提供する継続的デリバリーのプラットフォームです。 LaunchDarkly を使用すると、機能のロールアウトをコードのデプロイから分離し、機能フラグを大規模に管理できます。 LaunchDarkly と Azure DevOps を統合すると、頻繁なリリースに関連した潜在的なリスクを最小限に抑えられます。 リリースと開発プロセスをさらに統合するため、機能フラグロールアウトを Azure DevOps 作業項目にリンクすることができます。 

このラボでは、LaunchDarkly を活用して Azure DevOps で機能フラグの管理を最適化する方法を学びます。 

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- LaunchDarkly で機能フラグを作成する
- LaunchDarkly と Web アプリケーションを統合する
- Azure DevOps リリース パイプラインで LaunchDarkly 機能フラグを自動的にロールアウトする

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
-   Visual Studio 2019 Community Edition ([Visual Studio Downloads page](https://visualstudio.microsoft.com/vs) から入手可能) Visual Studio 2019 のインストールには、**ASP.NET および Web 開発**、**Azure 開発**、**.NET Core クロスプラットフォーム開発** のワークロードを含める必要があります。 これはすでにラボのコンピューターに事前インストールされています。

#### <a name="set-up-a-launchdarkly-trial-account"></a>LaunchDarkly トライアル アカウントを設定する

Web ブラウザーを使用して [LaunchDarkly Web サイト](https://launchdarkly.com/) に移動し、トライアル アカウントを作成します。 

#### <a name="set-up-an-azure-devops-organization"></a>Azure DevOps 組織を設定します。


          [組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops) の手順に従います。

#### <a name="prepare-an-azure-subscription"></a>Azure サブスクリプションの準備

-   既存の Azure サブスクリプションを識別するか、新しいものを作成します。
-   Azure サブスクリプションでは所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントではグローバル管理者のロールで Microsoft アカウントまたは Azure AD アカウントがあることを確認します。 詳細については、[「Azure portal を使用して Azure ロールの割り当てを一覧表示する」](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal)および[「Azure Active Directory で管理者ロールを表示して割当てる」](https://docs.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal#view-my-roles)を参照してください。

### <a name="exercise-0-configure-the-lab-prerequisites"></a>演習 0:ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、Azure DevOps Demo Generator テンプレートに基づいて事前に構成された Parts Unlimited チーム プロジェクトで構成されます。 

#### <a name="task-1-configure-the-team-project"></a>タスク 1:チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用し、**Parts Unlimited** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。 このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**:サイトの詳細については、https://docs.microsoft.com/en-us/azure/devops/demo-gen を参照してください。

1.  「**サインイン**」をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。
1.  必要な場合は、「**Azure DevOps Demo Generator**」ページで「**承諾する**」をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。
1.  「**新しいプロジェクトの作成**」ページで「**新しいプロジェクト名**」テキストボックスに「**LaunchDarkly**」と入力します。「**組織の選択**」ドロップダウン リストで Azure DevOps 組織を選択し、「**テンプレートの選択**」をクリックします。
1.  テンプレートのリストのツールバーで「**DevOps ラボ**」をクリックし、「**LaunchDarkly**」テンプレートを選択して「**テンプレートの選択**」をクリックします。
1.  再び「**新しいプロジェクトの作成**」ページで、欠落している拡張機能をインストールするよう指示されたら「**LaunchDarkly Integration V2**」ラベルの下にあるチェックボックスを選択し、「**プロジェクトの作成**」をクリックします。

    > **注**:プロセスが完了するまでお待ちください。 これには 2 分ほどかかります。 プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

### <a name="exercise-1-configure-feature-flags-in-azure-devops-by-using-launchdarkly"></a>演習 1:LaunchDarkly を使用して Azure DevOps で機能フラグを構成する

この演習では、LaunchDarkly を使用して Azure DevOps で機能フラグを構成します。 

#### <a name="task-1-create-a-feature-flag-in-launchdarkly"></a>タスク 1:LaunchDarkly で機能フラグを作成する

このタスクでは、LaunchDarkly で機能フラグを作成します

1.  ラボのコンピューターで Web ブラウザーを起動して [LaunchDarkly Web サイト](https://app.launchdarkly.com/)に移動し、ユーザー アカウントを作成します。 ブラウザーのセッションは **既定のプロジェクト** ポータルにリダイレクトされ、ここで機能フラグを作成できます。 
1.  LaunchDarkly ポータルの左側にある垂直メニュー バーで、「**機能フラグ**」をクリックします。
1. **[機能フラグ]** ペインの **[フラグの作成]** をクリックします。
1.  「**機能フラグの作成**」ペインで「**名前**」テキスト ボックスに「**メンバー ポータル**」と入力し、「**フラグの保存**」ボタンをクリックします。

    > **注**:「**メンバー ポータル**」という名前のフラグが作成されました。 このフラグを使用して、ASP.NET MVC Web アプリで「**メンバー ポータル**」機能の可視性を判断するものと想定します。 

    > **注**:LaunchDarkly を使用中のアプリケーションに統合するには、SDK キーが必要です。 

1.  LaunchDarkly ポータルの左側の垂直メニューバーで、「**アカウント設定**」をクリックし、「**プロジェクト**」タブをクリックします。

    > **注**:「**アカウント設定**」ペインの「**プロジェクト**」タブには、2 つの事前定義された環境(**実稼働** と **テスト**) があります。  このプロジェクトでは、運用環境の SDK キーを使用できます。 

1.  運用環境の SDK キーを記録します。 このラボで後ほど必要になります。

#### <a name="task-2-integrate-launchdarkly-in-your-web-application"></a>タスク 2:LaunchDarkly を使用中の アプリケーションに統合する

このタスクでは、LaunchDarkly を使用中の Web アプリケーションに統合します。

1.  ラボのコンピューターで、Azure DevOps ポータルが表示され、**LaunchDarkly** プロジェクトが開いている Web ブラウザー ウィンドウに切り替えます。プロジェクト ペインの左下コーナーで「**プロジェクトの設定**」をクリックします。
1.  「**プロジェクトの設定**」という垂直メニュー バーの「**リポジトリ**」セクションで「**リポジトリ**」を選択します。
1.  「**すべてのリポジトリ**」ペインで「**LaunchDarkly**」をクリックします。「**リポジトリの設定**」ペインで「**コミット メンション リンク**」と「**コミット メンション作業項目ソリューション**」を「**オン**」に設定していることを確認します。
1.  Azure DevOps ポータルの一番左にある垂直メニュー バーで「**リポジトリ**」をクリックし、「**ファイル**」ペインが表示されていることを確認します。

    > **注**:リポジトリが空の場合は、**[リポジトリのインポート]** セクションで **[インポート]** を選択し、**[Git リポジトリのインポート]** ペインの **[URL の複製]** テキスト ボックスに「`https://github.com/hsachinraj/PartsUnlimited.git`」と入力して、**[インポート]** をクリックします。

1.  「**ファイル**」ペインで「**複製**」をクリックします。
1. 「**リポジトリの複製**」ペインの「**IDE**」ドロップダウン リストで、「**Visual Studio**」を選択します。 プロンプトが表示されたら、「**Microsoft Visual Studio Web プロトコル ハンドラー セレクターを開く**」を選択します。 これにより、Visual Studio が自動的に起動します。 Azure DevOps サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。 
1.  Visual Studio ウィンドウ内の「**Azure DevOps**」ダイアログ ボックスで、「**複製**」をクリックし、プロンプトが表示されたら、Azure DevOps サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。 
1.  Visual Studio ウィンドウ内で、最上位の **Git** メニューをクリックし、「**ローカル リポジトリ**」をクリックし、「**フォルダー**」をクリックし、「**フォルダーの選択**」で、LaunchDarkly リポジトリを複製したローカル フォルダーに移動して、「**フォルダーの選択**」をクリックします。
1.  Visual Studio ウィンドウ内で最上部の「**表示**」メニューをクリックし、ドロップダウン メニューで「**Git 変更**」をクリックします。 
1. Visual Studio ウィンドウ内の「**Git の変更**」ペインの最上部にある **マスター** ドロップダウン リストで、下向き矢印をクリックします。ドロップダウン ダイアログ ボックスで「**リモート**」をクリックします。リモート ブランチのリストで「**origin/launch-darkly**」を選択します。 **[チェックアウト]** を選びます。 

    > **注**:これにより、**launch-darkly** ブランチが自動的にチェックアウトされます。 

1.  Visual Studio ウィンドウ内で、**ソリューション エクスプローラー** ウィンドウに切り替え、**PartsUnlimited.sln** ファイルをダブルクリックしてソリューションを開きます。 

    > **注**:**LaunchDarkly** と .NET アプリケーションを統合するには、**LaunchDarkly クライアント** を使用して NuGet パッケージをインストールする必要があります。 現在のプロジェクトでは、使いやすいようにこのパッケージはすでに追加されています。

1.  「**ソリューション エクスプローラー**」ペインで **PartsUnlimitedWebsite** ノードを拡張し、**Dependencies** サブノードを右クリックします。右クリック メニューで「**NuGet パッケージの管理**」エントリを選択します。
1.  **[NuGet: PartsUnlimitedWebsite]** ペインで、**LaunchDarkly.Client** が既にインストールされていることを確認します。
1.  Visual Studio インターフェイスのトップ メニューで「**IIS Express**」をクリックし、アプリケーションをローカルに起動します。 
1.  ローカル ブラウザー セッションでアプリケーションが起動し、「**メンバー ポータル**」セクションが Web インターフェイスの右上コーナーに表示されていることを確認します。

    > **注**:この **メンバー ポータル** モジュールは新しい機能で、**LaunchDarkly 機能フラグ** を使用してこれをコントロールするものと想定します。 これにより、LaunchDarkly をオンにすると、この機能がユーザーに表示されるはずです。

1.  Web アプリケーション インターフェイスの表示されている Web ブラウザー ウィンドウを閉じます。
1.  Visual Studio ウィンドウの **ソリューション エクスプローラー** で、**PartsUnlimitedWebsite\\Controllers\\HomeController.cs** に移動してこれを開きます。その内容を[最新の HomeController.cs コード スニペット](https://raw.githubusercontent.com/Microsoft/azuredevopslabs/master/labs/vstsextend/launchdarkly/codesnippet/HomeController.cs)のページで入手できるコードに置き換え、変更を保存します。 
1.  Visual Studio ウィンドウの **ソリューション エクスプローラー** で、**PartsUnlimitedWebsite\\Controllers\\AccountController.cs** に移動してこれを開きます。その内容を[最新の AccountController.cs コード スニペット](https://raw.githubusercontent.com/Microsoft/azuredevopslabs/master/labs/vstsextend/launchdarkly/codesnippet/AccountController.cs)のページで入手できるコードに置き換え、変更を保存します。 
1.  Visual Studio ウィンドウの **ソリューション エクスプローラー** で、**PartsUnlimitedWebsite\\Views\Shared\\_Layout.chtml** に移動してこれを開き、55 行目の `@await Html.PartialAsync("_Login")` を以下のコードに置き換えます。

    ```cshtml
    @if (ViewBag.togglevalue == true)
    {
      @await Html.PartialAsync("_Login")
    }
    ```

1.  **Home Controller.cs** ファイルの内容を表示しているタブに切り替え、`static LdClient client = new LdClient("__YourLaunchDarklySDKKey__");` 行目のプレースホルダー `__YourLaunchDarklySDKKey__` を LaunchDarkly アカウントの SDK キー (この演習の以前のタスクでコピーしたもの) に置き換えます。 

    > **注**:これにより環境固有の SDK キーで LdClient オブジェクトが作成されます。

1.  **AccountController.cs** ファイルの内容を表示しているタブに切り替え、`static LdClient client = new LdClient("__YourLaunchDarklySDKKey__");` 行目のプレースホルダー `__YourLaunchDarklySDKKey__` を LaunchDarkly アカウントの SDK キー (この演習の以前のタスクでコピーしたもの) に置き換えます。 

    > **注**:蒸気の変更により、静的 LaunchDarkly クライアントを初期化すると **HomeController** が起動するようになります。 
          **MemberPortal** の表示方法は、機能フラグが LaunchDarkly がオンかオフかを確認できるよう修正されています。 
          **_Layout.cshtml** ページはトグル値をチェックし、フラグがオンの場合は MemberPortal リンクをレンダリングします。 

1.  Visual Studio ウィンドウで、「**HomeController.cs**」タブに戻り、内容を確認します。ライン 57 とライン 74 の間のコードに注目してください:

    ```
        //LaunchDarkly start
        User user = LaunchDarkly.Client.User.WithKey("administrator@test.com");
        bool value = client.BoolVariation("member-portal", user, false);
        if (value)
        {
          ViewBag.Message = "Your application description page.";
          ViewData["togglevalue"] = value;
          return View(viewModel);
        }
        else
        {
          return View(viewModel);
        }
        // return View(viewModel);

    }
    //LaunchDarkly End
    ```

    > **注**:機能フラグを要求する場合は、ユーザー オブジェクトでパスする必要があります。 このため、最初にユーザー オブジェクトを初期化します。 これは、指定されたキーのユーザーが LaunchDarkly にいるかどうかを確認するために使われます。 このサンプルでは、ユーザー値をハードコードしています。 実際のシナリオでは、ログインしているユーザーを特定するか、データベース検索を行うと、この値を取得できます。

    > **注**:その後、**BoolVariation** メソッドを呼び出して LaunchDarkly で機能フラグの値をチェックします。 フラグが true の場合は、**[ViewData["togglevalue"]** が true に設定されます。これは **_Layout.chtml** でメンバー ポータル モジュールを表示するために使われます。 false の場合、メンバー ポータル モジュールは表示されません。

    > **注**:**AccountController.cs** の場合と同様に、LaunchDarkly コードはすでに **Login()** メソッドに追加されています。このため、**メンバー ポータル** アイコンをクリックすると、ログイン ページが表示されます。 LaunchDarkly のフラグが false の場合は、ログイン ページで HttpNotFound エラーが返されます。

1. Visual Studio ウィンドウで「**すべて保存**」をクリックし、「**IIS Express**」をクリックしてアプリケーションをローカルで起動します。 **[IIS Express]** オプションを使用できない場合は、ツール バーから **[Browser Link]** オプションを選び、**[Browser link ダッシュボード]** を選びます。 ダッシュボードから **[ブラウザーで表示]** を選びます。 
1.  Parts Unlimited Web サイトを表示している Web ブラウザーにもう **メンバー ポータル** リンクが表示されていないことを確認します。この時点で **MemberPortal** フラグはオフになっています。

    > **注**:これにより、LaunchDarkly を使用して機能フラグのコントロールが実装されます。 これで LaunchDarkly ポータルからのトグルを手動で有効にできます。 ただし、このラボでは、LaunchDarkly 拡張機能を使用して Azure DevOps リリース パイプラインでこれを構成します。 リリース プロセスの一部として機能フラグを含めるには、この変更を Azure DevOps 作業項目に関連付けます。 

1.  Web アプリケーション インターフェイスの表示されている Web ブラウザー ウィンドウを閉じます。
1.  Visual Studio ウィンドウ内で、必要に応じて、最上部の「**表示**」メニューをクリックし、ドロップダウン メニューで「**チーム エクスプローラー**」をクリックします。
1. 
          **チーム エクスプローラー** ペインの **ホーム** ページで、「**作業項目**」をクリックします。 プロンプトが表示されたら、リンクを選んで新しいハブを表示します。
1.  
          **チーム エクスプローラー** ペインの「**作業項目**」ページで、作業項目 ID を含め、このブランチに関連付けられている単一の作業項目をメモします。 
1.  作業項目を表すエントリをダブルクリックします。 これにより、別の Web ブラウザー タブが自動的に開き、Azure DevOps ポータルに作業項目が表示されます。 
1.  作業項目ペインで、作業項目が自分に割り当てられていることを確認します。 
1. Visual Studio ウィンドウに切り替え、**[Git 変更]** ペインに移動し、**[メッセージの入力]** テキストボックスに「**Integated LaunchDarkly #<work\_item\_ID>**」と入力します。この **<work\_item\_ID>** は先ほどメモした ID をです。**[すべてコミット]** ボタンの隣にある下向き矢印をクリックし、ドロップダウン リストの **[すべてをコミットしてプッシュ]** をクリックします。 **[すべてコミット]** が淡色表示されている場合は、上矢印を選んでプッシュを実行できます。 

#### <a name="task-3-automatically-rollout-launchdarkly-feature-flags-during-release"></a>タスク 3:リリース中に自動的に LaunchDarkly 機能フラグをロールアウトする

このタスクでは、Azure DevOps リリース中の LaunchDarkly 機能フラグの自動ロールアウトを構成します。

> **注**:Azure DevOps サービスと統合するには、LaunchDarkly アクセス トークンが必要です。 

1.  LaunchDarkly アクセス トークンを取得するには、LaunchDarkly ポータルを表示しているブラウザー ウィンドウに戻ります。左側の垂直メニューで「**アカウント設定**」をクリックし、「**アカウント設定**」ペインで「**承認**」タブをクリックします。「**アクセス トークン**」セクションで「**トークンの作成**」をクリックします。
1.  「**アクセス トークンの作成**」ペインで「**名前**」テキストボックスに「**メンバー ポータル**」と入力します。「**ロール**」ドロップダウン リストで「**ライター**」を選択し、「**トークンの保存**」をクリックします。
1.  再び「**アカウント設定**」ペインの「**承認**」タブでアクセス トークンをコピーし、メモ帳に貼り付けます。 

    > **注**:必ずトークンをコピーしてください。 現在のページを閉じると、取得できなくなります。

1.  Azure DevOps ポータルを表示しているブラウザー ウィンドウに戻り、「**LaunchDarkly**」プロジェクト ペインで「**プロジェクトの設定**」をクリックします。 
1. [**プロジェクトの詳細**] ペインの [**パイプライン**] セクションで [**サービス接続**] をクリックしてから [**サービス接続の作成**] をクリックします。
1.  「**新しいサービス接続**」ペインで「**LaunchDarkly**」をクリックしてから「**次へ**」をクリックします。
1.  「**新しい LaunchDarkly サービス接続**」ペインで「**アクセス トークン**」テキスト ボックスに、このタスクで以前に取得しておいたアクセス トークンを貼り付けます。「**サービス接続名**」テキスト ボックスに「**az-400 m12l01 LaunchDarkly**」と入力して「**保存**」をクリックします。
1.  Azure DevOps ポータルの左側の垂直メニューで「**Boards**」をクリックします。「**作業項目**」ペインで、以前のタスクで自分に割り当てておいた作業項目をクリックします。
1.  「**LaunchDarkly を使用して機能フラグ管理を実装**」作業項目で、右上コーナーの「**Launch Darkly**」タブを選択します。 
1.  「**LauchDarkly**」タブの「**機能フラグの選択**」セクションで、「**メンバー ポータル**」機能フラグを選択します。
1.  Azure DevOps ポータルの左側にある垂直メニュー バーで「**パイプライン**」をクリックし、「**パイプライン**」セクションで「**リリース**」を選択します。 
1.  「**パイプライン / リリース**」ペインで「**LaunchDarkly_CD**」パイプラインを選択し、「**編集**」をクリックします。
1.  「**すべてのパイプライン / LaunchDarkly_CD**」ペインで「**ステージ 1**」段階の「**1 ジョブ、3 タスク**」リンクをクリックし、パイプラインのタスクを表示します。
1.  ステージに次のタスクが含まれていることを確認します。

    - **Azure リソース グループのデプロイ**:このタスクでは、ARM テンプレートを使用して PartsUnlimited Web サイト向けに Azure アプリ サービスをデプロイします。
    - **LaunchDarkly のロールアウト**:このタスクでは、LaunchDarkly サブスクリプションで機能フラグを有効にします。
    - **Azure App Service のデプロイ**:このタスクでは、最初のタスクで作成した Azure アプリ サービスに PartsUnlimited Web アプリをデプロイします。

1.  タスクのリストで「**Azure リソース グループのデプロイ**」タスクを選択し、「**Azure サブスクリプション**」ドロップダウン リストで下向きキャレット記号をクリックします。エントリのリストで、このタスクで使用したい Azure サブスクリプションを選択し、「**承認**」をクリックして Azure サービス接続を構成します。 指示されたら、Azure サブスクリプションで所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントでグローバル管理者のロールがあるアカウントを使用してサインインします。
1.  タタスクのリストで「**LaunchDarkly のロールアウト**」タスクを選択します。「**アカウント**」ドロップダウン リストで、このタスクで以前に作成した LaunchDarkly サービス接続を示しているエントリを選択します。

    > **注**:「**フラグの状態**」が「**オン**」になっていることを確認します。 この設定により、リリース中のフラグの状態が管理されます。

1.  タスクのリストで「**Azure App Service のデプロイ**」タスクを選択します。「**Azure サブスクリプション**」ドロップダウン リストで、以前に作成した Azure サブスクリプション サービス接続を選択し、「**保存**」をクリックします。
1.  Azure DevOps ポータルの Azure DevOps ページの右上コーナーで「**ユーザー設定**」アイコンをクリックします。ドロップダウン メニューで「**個人用アクセス トークン**」をクリックし、「**個人用アクセス トークン**」ペインで「**+ 新しいトークン**」をクリックします。
1.  「**新しい個人用アクセス トークンの作成**」ペインで以下の設定を指定し、「**作成**」をクリックします (他はすべて規定値のままにします):

    | 設定 | 値 |
    | --- | --- |
    | Name | **LaunchDarkly および Azure DevOps ラボ** |
    | スコープ | **フル アクセス** |

1.  「**成功**」ペインで個人用アクセス トークンの値をクリップボードにコピーします。

    > **注**:必ずトークンをコピーしてください。 このペインを閉じると、取得できなくなります。 

1.  「**成功**」ペインで「**閉じる**」をクリックします。
1.  「**すべてのパイプライン / LaunchDarkly_CD**」ペインに戻り、「**編集**」をクリックし、「**変数**」タブをクリックします。 
1.  変数リストで **launchdarkly-pat** 変数の値を、新しく生成した個人用アクセス トークンに設定します。
1.  変数リストで **launchdarkly-project-name** 変数の値を **LaunchDarkly** に設定し、「**保存**」をクリックします。

    > **注**:これによりリリース パイプラインの構成が完了します。 

1.  Azure DevOps ポータルの左側にある垂直メニュー バーの「**パイプライン**」セクションで、「**パイプライン**」を選択します。
1. 「**パイプライン**」ペインで、**LaunchDarkly-CI** ビルド パイプラインを示すエントリをクリックします。「**LaunchDarkly-CI**」ペインで「**パイプラインの実行**」をクリックし、「**パイプラインの実行**」ペインで「**実行**」をクリックします。

    > **注**:この CI パイプラインは .Net Core プロジェクトをコンパイルします。 ビルドが完了すると、リリースがトリガーされ、アプリがデプロイされて LaunchDarkly で機能フラグがロールアウトされます。

1.  ビルド パイプライン実行ペインの「**ジョブ**」セクションで「**エージェント ジョブ 1**」エントリをクリックし、ビルド プロセスの進捗状況を監視します。
1.  ビルドの完了後、Azure DevOps ポータルの左側にある垂直メニュー バーの「**パイプライン**」セクションで、「**リリース**」を選択します。
1.  「**パイプライン / リリース**」ペインの「**LaunchDarkly_CD**」ペインで「**Release-1**」エントリをクリックし、デプロイ プロセスの進捗状況を監視します。

    > **注**:デプロイの完了後は、LaunchDarkly ダッシュボードで **MemberPortal** 機能フラグがオンになっていることを確認できるはずです。

1.  ラボのコンピューターから Web ブラウザーを起動し、[**Azure Portal**](https://portal.azure.com) に移動します。このラボで使用している Azure サブスクリプションで所有者のロールがあるユーザー アカウントを使ってサインインします。
1.  Azure portal で **App Services** のリソースを検索して選択します。「**App Services**」ブレードから、Azure DevOps リリース パイプラインを介してアプリケーションをデプロイした Web アプリに移動します。
1.  Web アプリ ブレードで「**参照**」をクリックします。 別の Web ブラウザー タブが開き、新しくデプロイされた Web アプリケーションが表示されます。
1.  Parts Unlimited の Web ページで、「**メンバー ポータル**」機能が有効になっていることを確認します。

> **注**:LaunchDarkly には、このほかにも以下のような多数の機能が含まれています。
    
- **ユーザーのターゲット**:LaunchDarkly ターゲットでは、各ユーザーやユーザー グループの機能をオンまたはオフにできます。 これを使用すると、より広範なロールアウトを行う前に内部テストやプライベート ベータ、または使用可能性テストで機能をロールアウトできます。 
- **カスタム ターゲット規則**:LaunchDarkly では個々のユーザーをターゲットにできるだけでなく、カスタム規則を構築してユーザーのセグメントをターゲットにすることも可能です。 つまり、カスタム規則を作成し、指定した属性に基づいてユーザーをターゲットにできます。 
- **開発プロセスを管理するためのプロジェクトと環境**:[プロジェクト](https://docs.launchdarkly.com/docs/projects) では、ひとつの LaunchDarkly アカウントで複数の様々なソフトウェア プロジェクトを管理できます。 [環境](https://docs.launchdarkly.com/docs/environments) では、ローカル開発から QA、ステージング、運用にいたるまで開発サイクル全体で機能フラグを管理できます。 

### <a name="exercise-2-remove-the-azure-lab-resources"></a>演習 2:Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

>**注**:新規に作成し、使用しなくなったすべての Azure リソースを削除することを忘れないでください。 使用していないリソースを削除することで、予期しない料金が発生しなくなります。

#### <a name="task-1-remove-the-azure-lab-resources"></a>タスク 1:Azure ラボ リソースを削除する

このタスクでは、Azure portal を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、このラボで以前にデプロイした App Service のインスタンスまでナビゲートします。 
1.  App Service インスタンス ブレードで、App Service インスタンスを含むリソース グループの名前を示すリンクをクリックします。
1.  App Service インスタンスが含まれているリソース グループのブレードで、「**リソース グループの削除**」をクリックします。指示されたら、リソース グループの名前を提供し、「**削除**」をクリックします。

## <a name="review"></a>確認

このラボでは、LaunchDarklyを活用して Azure DevOps で機能フラグの管理を最適化する方法を学びました。 
