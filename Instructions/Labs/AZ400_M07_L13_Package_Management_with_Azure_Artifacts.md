---
lab:
  title: Azure Artifacts によるパッケージ管理
  module: 'Module 07: Design and implement a dependency management strategy'
---

# Azure Artifacts によるパッケージ管理

## ラボの要件

- このラボには、**Microsoft Edge** または [Azure DevOps 対応ブラウザー](https://docs.microsoft.com/azure/devops/server/compatibility)が必要です。

- **Azure DevOps 組織を設定する:** このラボで使える Azure DevOps 組織がまだない場合は、[AZ-400 ラボの前提条件](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html)に関するページの手順に従って作成してください。

- **サンプルの eShopOnWeb プロジェクトを設定する:** このラボで使用できるサンプルの eShopOnWeb プロジェクトがまだない場合は、[AZ-400 ラボの前提条件](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html)に関するページに記載されている手順に従って作成してください。

- Visual Studio 2022 Community Edition ([Visual Studio のダウンロード ページ](https://visualstudio.microsoft.com/downloads/)から入手可能)。 Visual Studio 2022 のインストールには、**ASP<nolink>.NET および Web 開発**、**Azure 開発**、 **.NET Core クロスプラットフォーム開発**のワークロードを含める必要があります。

- **.NET Core SDK:**[.NET Core SDK (2.1.400 以降) をダウンロードしてインストールします](https://go.microsoft.com/fwlink/?linkid=2103972)

- **Azure Artifacts 資格情報プロバイダー:**[資格情報プロバイダーをダウンロードしてインストールします](https://go.microsoft.com/fwlink/?linkid=2099625)。

## ラボの概要

Azure Artifacts は、Azure DevOps での NuGet、npm、Maven パッケージの検出、インストール、公開を容易にします。 ビルドなどの他の Azure DevOps 機能と緊密に統合されているため、パッケージ管理は既存のワークフローのシームレスな部分になります。

## 目標

このラボを完了すると、次のことができるようになります。

- フィードを作成して接続する。
- NuGet パッケージを作成して公開する。
- NuGet パッケージをインポートする。
- NuGet パッケージを更新する。

## 推定時間:40 分

## Instructions

### 演習 0:ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。

#### タスク 1: (完了している場合はスキップしてください) チーム プロジェクトを作成して構成する

このタスクでは、複数のラボで使用される **eShopOnWeb** Azure DevOps プロジェクトを作成します。

1. ラボ コンピューターのブラウザー ウィンドウで、Azure DevOps 組織を開きます。 **[新しいプロジェクト]** をクリックします。 プロジェクトに「**eShopOnWeb**」という名前を付け、他のフィールドは既定値のままにします。 **[作成]** をクリックします。

    ![[新しいプロジェクトの作成] パネルのスクリーンショット。](images/create-project.png)

#### タスク 2: (完了している場合はスキップしてください) eShopOnWeb Git リポジトリをインポートする

このタスクでは、複数のラボで使用される eShopOnWeb Git リポジトリをインポートします。

1. ラボ コンピューターのブラウザー ウィンドウで、Azure DevOps 組織と、前に作成した **eShopOnWeb** プロジェクトを開きます。 **[リポジトリ]、[ファイル]**、**[リポジトリをインポートする]** の順にクリックします。 **インポート** を選択します。 **[Git リポジトリをインポートする]** ウィンドウで、URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> を貼り付けて、 **[インポート]** をクリックします。

    ![[リポジトリのインポート] パネルのスクリーンショット。](images/import-repo.png)

1. リポジトリは次のように編成されています。
    - **.ado** フォルダーには、Azure DevOps の YAML パイプラインが含まれています。
    - **.devcontainer** フォルダーには、コンテナーを使って開発するためのセットアップが含まれています (VS Code でローカルに、または GitHub Codespaces で)。
    - **infra** フォルダーには、一部のラボ シナリオで使用される Bicep および ARM のコードとしてのインフラストラクチャ テンプレートが含まれています。
    - **.github** フォルダーには、YAML GitHub ワークフローの定義が含まれています。
    - **src** フォルダーには、ラボ シナリオで使用される .NET 8 Web サイトが含まれています。

#### タスク 3: (完了している場合はスキップしてください) メイン ブランチを既定のブランチとして設定する

1. **[リポジトリ]、[ブランチ]** の順に移動します。
1. **main** ブランチをポイントし、列の右側に表示される省略記号をクリックします。
1. **[既定のブランチとして設定]** をクリックします。

#### タスク 4:Visual Studio で eShopOnWeb ソリューションを構成する

このタスクでは、ラボの準備をするために Visual Studio を構成します。

1. Azure DevOps ポータルで **eShopOnWeb** チーム プロジェクトを表示していることを確認します。

    > **注**:[https://dev.azure.com/`<your-Azure-DevOps-account-name>`/eShopOnWeb](https://dev.azure.com/`<your-Azure-DevOps-account-name>`/eShopOnWeb) という URL に移動することで、プロジェクト ページに直接アクセスできます。`<your-Azure-DevOps-account-name>` プレースホルダーは、ご自分の Azure DevOps 組織名を表します。

1. **eShopOnWeb** ペインの左側にある垂直メニューで、**[リポジトリ]** をクリックします。
1. **[ファイル]** ペインで、 **[複製]** をクリックし、 **[VS Code で複製]** の横にあるドロップダウン矢印を選択し、ドロップダウン メニューで **[Visual Studio]** を選択します。
1. 続行するかどうかを確認するメッセージが表示されたら、 **[開く]** をクリックします。
1. プロンプトが表示されたら、Azure DevOps 組織のセットアップに使用したユーザー アカウントでサインインします。
1. Visual Studio インターフェイス内の **Azure DevOps** ポップアップ ウィンドウで、既定のローカル パス (C:\eShopOnWeb) をそのままにして、**[クローン]** をクリックします。 これにより、プロジェクトが Visual Studio に自動的にインポートされます。
1. ラボで使用するために、Visual Studio ウィンドウを開いたままにします。

### 演習 1:Azure Artifacts の操作

この演習では、次の手順を使用して Azure Artifacts を操作する方法を学習します。

- フィードを作成して接続する。
- NuGet パッケージを作成して公開する。
- NuGet パッケージをインポートする。
- NuGet パッケージを更新する。

#### タスク 1:フィードを作成して接続する

このタスクでは、フィードを作成して接続します。

1. Azure DevOps ポータルでプロジェクト設定を表示している Web ブラウザー ウィンドウの垂直ナビゲーション ペインで、 **[アーティファクト]** を選択します。
1. **アーティファクト** ハブが表示された状態で、ペインの上部にある **[+ フィードの作成]** をクリックします。

    > **注**:このフィードは、組織内のユーザーが利用できる NuGet パッケージのコレクションであり、ピアとしてパブリック NuGet フィードと並んで配置されます。 このラボのシナリオでは、Azure Artifacts を使用するためのワークフローに焦点を当てているため、実際のアーキテクチャと開発の決定は純粋に例示的なものです。  このフィードには、この組織のプロジェクト間で共有できる共通の機能が含まれます。

1. **[新しいフィードの作成]** ペインの **[名前]** テキストボックスに、「**`eShopOnWebShared`**」と入力し、**[スコープ]** セクションで **[組織]** オプションを選択し、他の設定を既定値のままにして、**[作成]** をクリックします。

    > **注**:このNuGet フィードに接続するユーザーは、環境を構成する必要があります。

1. **アーティファクト** ハブに戻り、 **[フィードに接続]** をクリックします。
1. **[フィードに接続]** ペインの **[NuGet]** セクションで **[Visual Studio]** を選択し、 **[Visual Studio]** ペインで**ソース** URL をコピーします。 `https://pkgs.dev.azure.com/Azure-DevOps-Org-Name/_packaging/eShopOnWebShared/nuget/v3/index.json`
1. **Visual Studio** ウィンドウに戻ります。
1. Visual Studio ウィンドウで、 **[ツール]** メニュー ヘッダーをクリックし、ドロップダウン メニューで **[NuGet Package Manager]** を選択し、カスケード メニューで **[パッケージ マネージャー設定]** を選択します。
1. **[オプション]** ダイアログ ボックスで、 **[パッケー ジソース]** をクリックし、プラス記号をクリックして新しいパッケージ ソースを追加します。
1. ダイアログ ボックスの下部にある **[名前]** テキストボックスで、**[パッケージ ソース]** を **eShopOnWebShared** に置き換え、**[ソース]** テキストボックスに、Azure DevOps ポータルでコピーした URL を貼り付けます。
1. **[更新]** をクリックし、 **[OK]** をクリックして追加を完了します。

    > **注**:Visual Studio が新しいフィードに接続されました。

#### タスク 2: 社内で開発された NuGet パッケージを作成して発行する

このタスクでは、社内で開発したカスタム NuGet パッケージを作成して発行します。

1. 新しいパッケージソースの構成に使用した Visual Studio ウィンドウで、メイン メニューの **[ファイル]** をクリックし、ドロップダウン メニューの **[新規]** をクリックしてから、カスケード メニューの **[プロジェクト]** をクリックします。

    > **注**:NuGet パッケージとして公開される共有アセンブリを作成して、他のチームがプロジェクト ソースを直接操作しなくても、アセンブリを統合して最新の状態に保つことができるようにします。

1. **[新しいプロジェクトの作成]** ペインの **[最近のプロジェクト テンプレート]** ページで、検索テキスト ボックスを使って**クラス ライブラリ** テンプレートを見つけ、C# 用のテンプレートを選んで **[次へ]** をクリックします。
1. **[新しいプロジェクトの作成]** ペインの **[クラス ライブラリ]** ページで、次の設定を指定して **[作成]** をクリックします。

    | 設定 | [値] |
    | --- | --- |
    | プロジェクト名 | **eShopOnWeb.Shared** |
    | 場所 | 既定値を受け入れる |
    | ソリューション | **新しいソリューションの作成** |
    | ソリューション名 | **eShopOnWeb.Shared** |

    **[ソリューションとプロジェクトを同じディレクトリに配置する]** のチェックボックスをオンにします。

1. [次へ] をクリックします。 **.NET 8** をフレームワーク オプションとして受け入れます。
1. **[作成]** ボタンを選んで、プロジェクトの作成を確認します。
1. Visual Studio インターフェイス内の **[ソリューション エクスプローラー]** ウィンドウで、 **[Class1.cs]** を右クリックし、右クリック メニューで **[削除]** を選択し、確認を求められたら **[OK]** をクリックします。
1. **Ctrl + Shift + B** キーを押すか、**EShopOnWeb.Shared プロジェクトを右クリック**して **[ビルド]** を選び、プロジェクトをビルドします。
1. ラボ ワークステーションから [スタート] メニューを開き、**Windows PowerShell** を見つけます。 次に、カスケード メニューの **[Windows PowerShell を管理者として開く]** をクリックします。
1. 
          **[管理者:Windows PowerShell]** ウィンドウで、次のコマンドを実行して eShopOnWeb.Shared フォルダーに移動します。

    ```text
    cd c:\eShopOnWeb\eShopOnWeb.Shared
    ```

    > **注**:**eShopOnWeb.Shared** フォルダーは、**eShopOnWeb.Shared.csproj** ファイルの場所です。 別の場所を選択した場合は、代わりにその場所に移動します。

1. 次を実行して、プロジェクトから **.nupkg** ファイルを作成します。

    ```powershell
    dotnet pack .\eShopOnWeb.Shared.csproj
    ```

    > **注**:**dotnet pack** コマンドは、プロジェクトをビルドし、NuGet パッケージを **bin\Release** フォルダーに作成します。 **[リリース]** フォルダーがない場合は、代わりに **[デバッグ]** フォルダーを使用できます。

    > **注**: **[管理者:Windows PowerShell]** ウィンドウに表示される警告はすべて無視します。

    > **注意**: dotnet pack は、プロジェクトから識別できる情報に基づいて最小限のパッケージをビルドします。 たとえば、名前が **eShopOnWeb.Shared.1.0.0.nupkg** となることに注意してください。 そのバージョン番号は、アセンブリから取得されました。

1. PowerShell ウィンドウで、次のコマンドを実行して、**bin\Release** フォルダーを開きます。

    ```powershell
    cd .\bin\Release
    ```

1. 次を実行して、パッケージを **eShopOnWebShared** フィードに発行します。

    > **重要**: Azure DevOps で認証できるようにするには、オペレーティング システムの資格情報プロバイダーをインストールする必要があります。 インストール手順については、「[Azure Artifacts 資格情報プロバイダー](https://go.microsoft.com/fwlink/?linkid=2099625)」を参照してください。 PowerShell ウィンドウで次のコマンドを実行することでインストールできます: `iex "& { $(irm https://aka.ms/install-artifacts-credprovider.ps1) } -AddNetfx"`

    > **注**:**API キー**を指定する必要があります。これは、空でない文字列にすることができます。 ここでは **az** を使用しています。 プロンプトが表示されたら、Azure DevOps 組織にサインインします。

    ```powershell
    dotnet nuget push --source "eShopOnWebShared" --api-key az eShopOnWeb.Shared.1.0.0.nupkg
    ```

    > **注**:プロンプトが表示されない場合は、**--interactive** パラメータをコマンドに追加できます。

1. パッケージ プッシュ操作の成功の確認を待ちます。
1. Azure DevOps ポータルを表示する Web ブラウザー ウィンドウに切り替え、垂直ナビゲーションペインで **[アーティファクト]** を選択します。
1. **[成果物]** ハブ ペインで、左上隅のドロップダウン リストをクリックし、フィードのリストで **eShopOnWebShared** エントリを選びます。

    > **注**:**eShopOnWebShared** フィードには、新しく発行された NuGet パッケージが含まれているはずです。

1. NuGet パッケージをクリックして、詳細を表示します。

#### タスク 3: オープンソース NuGet パッケージを Azure DevOps パッケージ フィードにインポートする

独自のパッケージを開発するだけでなく、オープンソースの NuGet (<https://www.nuget.org>) DotNet パッケージ ライブラリを使用することもできます。 数百万個のパッケージを利用できるので、必ず何かアプリケーションに役立つものがあります。

このタスクでは、汎用の "Newtonsoft.Json" サンプル パッケージを使用しますが、ライブラリ内の他のパッケージにも同じ方法を使用できます。

1. 同じ PowerShell ウィンドウで、**eShopOnWeb.Shared** フォルダーに移動し、次の **dotnet** コマンドを実行して、サンプル パッケージをインストールします。

    ```powershell
    dotnet add package Newtonsoft.Json
    ```

1. インストール プロセスの出力を確認します。 これには、パッケージのダウンロードを試みるさまざまなフィードが示されています。

    ```powershell
    Feeds used:
      https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json
      https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/eShopOnWeb/_packaging/eShopOnWebShared/nuget/v3/index.json
    ```

1. その次に、実際のインストール プロセス自体に関する追加の出力が示されています。

    ```powershell
    Determining projects to restore...
    Writing C:\Users\AppData\Local\Temp\tmpxnq5ql.tmp
    info : X.509 certificate chain validation will use the default trust store selected by .NET for code signing.
    info : X.509 certificate chain validation will use the default trust store selected by .NET for timestamping.
    info : Adding PackageReference for package 'Newtonsoft.Json' into project 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
    info :   GET https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json
    info :   OK https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json 124ms
    info : Restoring packages for c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj...
    info :   GET https://api.nuget.org/v3/vulnerabilities/index.json
    info :   OK https://api.nuget.org/v3/vulnerabilities/index.json 84ms
    info :   GET https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/vulnerability.base.json
    info :   GET https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/2024.02.17.11.23.35/vulnerability.update.json
    info :   OK https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/vulnerability.base.json 14ms
    info :   OK https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/2024.02.17.11.23.35/vulnerability.update.json 30ms
    info : Package 'Newtonsoft.Json' is compatible with all the specified frameworks in project 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
    info : PackageReference for package 'Newtonsoft.Json' version '13.0.3' added to file 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
    info : Writing assets file to disk. Path: c:\eShopOnWeb\eShopOnWeb.Shared\obj\project.assets.json
    log  : Restored c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj (in 294 ms).
    ```

1. Newtonsoft.Json パッケージが、**Newtonsoft.Json** として [パッケージ] にインストールされました。 Visual Studio **ソリューション エクスプローラー**で **eShopOnWeb.Shared** プロジェクトに移動し、[依存関係] を展開して、[パッケージ] の下にある **Newtonsoft.Json** に注目します。 [パッケージ] の左側にある小さな矢印をクリックして、フォルダーとファイルの一覧を開きます。

#### タスク 4: オープンソースの NuGet パッケージを Azure Artifacts にアップロードする

このパッケージを、前に作成した Azure Artifacts パッケージ フィードにアップロードすることで、DevOps チームが再利用するための "承認済み" のパッケージであると考えてみましょう。

1. Visual Studio で新しい **Newtonsoft.Json** パッケージを右クリックし、コンテキスト メニューから **[エクスプローラーでフォルダーを開く]** を選択します。 拡張機能 **.nupkg** を含む新しい **Newtonsoft.Json** パッケージが表示されます。
2. エクスプローラー ウィンドウのアドレス バーから完全なパスをコピーします。
3. PowerShell ウィンドウで、パスをコピーしたものに置き換えて、次のコマンドを実行します。

    ```powershell
    dotnet nuget push --source "eShopOnWebShared" --api-key az C:\eShopOnWeb\eShopOnWeb.Shared\Newtonsoft.Json\newtonsoft.json.13.0.3.nupkg
    ```

    > **注**: これにより、エラー メッセージが表示されます。

    ```text
    Response status code does not indicate success: 409 (Conflict - 'Newtonsoft.Json 1.3.0.17' cannot be published to the feed because it exists in at least one of the feed's upstream sources. Publishing this copy would prevent you from using 'Newtonsoft.Json 1.3.0.17' from 'NuGet Gallery'. For more information, see https://go.microsoft.com/fwlink/?linkid=864880 (DevOps Activity ID: AE08BE89-C2FA-4FF7-89B7-90805C88972C)).
    ```

Azure DevOps Artifacts パッケージ フィードを作成すると、設計上、dotnet の例の nuget.org などの**アップストリーム ソース**に対応します。 ただし、DevOps チームが **"内部専用"** パッケージ フィードを作成することを妨げるものはありません。

1. Azure DevOps ポータルに移動し、**[成果物]** を参照して、**eShopOnWebShared** フィードを選択します。
1. **[アップストリーム ソースの検索]** をクリックします
1. **[アップストリーム パッケージに移動]** ウィンドウで、パッケージの種類として **[NuGet]** を選択し、検索フィールドに「**`Newtonsoft.Json`**」と入力します。
1. **[検索]** ボタンを選んで確定します。
1. これにより、すべての Newtonsoft.Json パッケージと使用可能なさまざまなバージョンの一覧が表示されます。
1. **左方向キー**を押して **eShopOnWebShared** フィードに戻ります。
1. 歯車をクリックして **[フィードの設定]** を開きます。 [フィードの設定] ページで、 **[アップストリーム ソース]** を選びます。
1. さまざまな開発言語用にさまざまなアップストリーム パッケージ マネージャーがあることに注意してください。 一覧から **[NuGet ギャラリー]** を選択します。 **[削除]** ボタン、 **[保存]** ボタンの順に選択します。

1. これらの保存した変更を使用して、次のコマンドをもう一度実行することで、PowerShell ウィンドウから NuGet.exe を使用して **Newtonsoft.Json** パッケージをアップロードできます。

    ```text
     dotnet nuget push --source "eShopOnWebShared" --api-key az C:\eShopOnWeb\eShopOnWeb.Shared\Newtonsoft.Json\newtonsoft.json.13.0.3.nupkg
    ```

    > **注**:これでアップロードが成功します。

    ```text
    Pushing newtonsoft.json.13.0.3.nupkg to 'https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/_packaging/5faffb6c-018b-4452-a4d6-72c6bffe79db/nuget/v2/'...
    PUT https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/_packaging/5faffb6c-018b-4452-a4d6-72c6bffe79db/nuget/v2/
    Accepted https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/_packaging/5faffb6c-018b-4452-a4d6-72c6bffe79db/nuget/v2/ 3160ms
    Your package was pushed.
    ```

1. Azure DevOps ポータルで、成果物パッケージ フィード ページを**更新**します。 パッケージの一覧に、**eShopOnWeb.Shared** カスタム開発パッケージと **Newtonsoft.Json** パブリック ソース パッケージの両方が表示されます。
1. Visual Studio の **eShopOnWeb.Shared** ソリューションで **eShopOnWeb.Shared** プロジェクトを右クリックし、コンテキスト メニューから **[NuGet パッケージの管理]** を選択します。
1. NuGet パッケージ マネージャー ウィンドウで、**[パッケージ ソース]** が **eShopOnWebShared** に設定されていることを確認します。
1. **[参照]** をクリックして、NuGet パッケージの一覧が読み込まれるまで待ちます。
1. この一覧には、**eShopOnWeb.Shared** カスタム開発パッケージと **Newtonsoft.Json** パブリック ソース パッケージの両方も表示されます。

## 確認

このラボでは、次の手順を使用して Azure Artifacts を操作する方法を学習しました：

- 作成してフィードに接続しました。
- NuGet パッケージを作成して公開しました。
- カスタム開発 NuGet パッケージをインポートしました。
- パブリック ソース NuGet パッケージをインポートしました。
