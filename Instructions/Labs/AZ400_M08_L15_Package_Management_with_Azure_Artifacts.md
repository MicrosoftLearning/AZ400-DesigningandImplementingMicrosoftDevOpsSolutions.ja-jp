---
lab:
  title: Azure Artifacts によるパッケージ管理
  module: 'Module 08: Design and implement a dependency management strategy'
---

# Azure Artifacts によるパッケージ管理

## 受講生用ラボ マニュアル

## ラボの要件

- このラボには、**Microsoft Edge** または [Azure DevOps 対応ブラウザー](https://docs.microsoft.com/azure/devops/server/compatibility)が必要です。

- **Azure DevOps 組織を設定する:** このラボで使える Azure DevOps 組織がまだない場合は、[AZ-400 ラボの前提条件](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html)に関するページの手順に従って作成してください。

- **サンプルの EShopOnWeb プロジェクトをセットアップする:** このラボに使えるサンプルの EShopOnWeb プロジェクトがまだない場合は、[AZ-400 ラボの前提条件](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html)に関するページの手順に従って作成してください。

- Visual Studio 2022 Community Edition ([Visual Studio のダウンロード ページ](https://visualstudio.microsoft.com/downloads/)から入手可能)。 Visual Studio 2022 のインストールには、**ASP<nolink>.NET および Web 開発**、**Azure 開発**、 **.NET Core クロスプラットフォーム開発**のワークロードを含める必要があります。

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

この演習では、ラボの前提条件の検証、および Azure DevOps 組織の準備と EShopOnWeb プロジェクトの作成の両方について、復習します。 詳細については上記の手順を参照してください。

#### タスク 1: Visual Studio で EShopOnWeb ソリューションを構成する

このタスクでは、ラボの準備をするために Visual Studio を構成します。

1. Azure DevOps ポータルで **EShopOnWeb** チーム プロジェクトを表示していることを確認します。

    > **注**: [https://dev.azure.com/`<your-Azure-DevOps-account-name>` /EShopOnWeb](https://dev.azure.com/`<your-Azure-DevOps-account-name>`/EShopOnWeb) という URL に移動することで、プロジェクト ページに直接アクセスできます。`<your-Azure-DevOps-account-name>` プレースホルダーは、自分の Azure DevOps 組織名を表します。

2. **EShopOnWeb** ペインの左側にある垂直メニューで、 **[リポジトリ]** をクリックします。
3. **[ファイル]** ペインで、 **[複製]** をクリックし、 **[VS Code で複製]** の横にあるドロップダウン矢印を選択し、ドロップダウン メニューで **[Visual Studio]** を選択します。
4. 続行するかどうかを確認するメッセージが表示されたら、 **[開く]** をクリックします。
5. プロンプトが表示されたら、Azure DevOps 組織のセットアップに使用したユーザー アカウントでサインインします。
6. Visual Studio インターフェイス内の **Azure DevOps** ポップアップ ウィンドウで、既定のローカル パス (C:\EShopOnWeb) をそのままにして、 **[クローン]** をクリックします。 これにより、プロジェクトが Visual Studio に自動的にインポートされます。
7. ラボで使用するために、Visual Studio ウィンドウを開いたままにします。

### 演習 1:Azure Artifacts の操作

この演習では、次の手順を使用して Azure Artifacts を操作する方法を学習します。

- フィードを作成して接続する。
- NuGet パッケージを作成して公開する。
- NuGet パッケージをインポートする。
- NuGet パッケージを更新する。

#### タスク 1:フィードを作成して接続する

このタスクでは、フィードを作成して接続します。

1. Azure DevOps ポータルでプロジェクト設定を表示している Web ブラウザー ウィンドウの垂直ナビゲーション ペインで、 **[アーティファクト]** を選択します。
2. **アーティファクト** ハブが表示された状態で、ペインの上部にある **[+ フィードの作成]** をクリックします。

    > **注**:このフィードは、組織内のユーザーが利用できる NuGet パッケージのコレクションであり、ピアとしてパブリック NuGet フィードと並んで配置されます。 このラボのシナリオでは、Azure Artifacts を使用するためのワークフローに焦点を当てているため、実際のアーキテクチャと開発の決定は純粋に例示的なものです。  このフィードには、この組織のプロジェクト間で共有できる共通の機能が含まれます。

3. **[新しいフィードの作成]** ペインの **[名前]** テキストボックスに「**EShopOnWebShared**」と入力し、 **[スコープ]** セクションで **[組織]** オプションを選び、他の設定を既定値のままにして、 **[作成]** をクリックします。

    > **注**:このNuGet フィードに接続するユーザーは、環境を構成する必要があります。

4. **アーティファクト** ハブに戻り、 **[フィードに接続]** をクリックします。
5. **[フィードに接続]** ペインの **[NuGet]** セクションで **[Visual Studio]** を選択し、 **[Visual Studio]** ペインで**ソース** URL をコピーします。 (https://pkgs.dev.azure.com/<Azure-DevOps-Org-Name>_packaging/EShopOnWebShared/nuget/v3/index.json)
6. **Visual Studio** ウィンドウに戻ります。
7. Visual Studio ウィンドウで、 **[ツール]** メニュー ヘッダーをクリックし、ドロップダウン メニューで **[NuGet Package Manager]** を選択し、カスケード メニューで **[パッケージ マネージャー設定]** を選択します。
8. **[オプション]** ダイアログ ボックスで、 **[パッケー ジソース]** をクリックし、プラス記号をクリックして新しいパッケージ ソースを追加します。
9. ダイアログ ボックスの下部にある **[名前]** テキストボックスで、 **[パッケージ ソース]** を **EShopOnWebShared** に置き換え、 **[ソース]** テキストボックスに、Azure DevOps ポータルでコピーした URL を貼り付けます。
10. **[更新]** をクリックし、 **[OK]** をクリックして追加を完了します。

    > **注**:Visual Studio が新しいフィードに接続されました。

#### タスク 2: 社内で開発された NuGet パッケージを作成して発行する

このタスクでは、社内で開発したカスタム NuGet パッケージを作成して発行します。

1. 新しいパッケージソースの構成に使用した Visual Studio ウィンドウで、メイン メニューの **[ファイル]** をクリックし、ドロップダウン メニューの **[新規]** をクリックしてから、カスケード メニューの **[プロジェクト]** をクリックします。

    > **注**:NuGet パッケージとして公開される共有アセンブリを作成して、他のチームがプロジェクト ソースを直接操作しなくても、アセンブリを統合して最新の状態に保つことができるようにします。

2. **[新しいプロジェクトの作成]** ペインの **[最近のプロジェクト テンプレート]** ページで、検索テキスト ボックスを使って**クラス ライブラリ** テンプレートを見つけ、C# 用のテンプレートを選んで **[次へ]** をクリックします。
3. **[新しいプロジェクトの作成]** ペインの **[クラス ライブラリ]** ページで、次の設定を指定して **[作成]** をクリックします。

    | 設定 | [値] |
    | --- | --- |
    | プロジェクト名 | **EShopOnWeb.Shared** |
    | 場所 | 既定値を受け入れる |
    | ソリューション | **新しいソリューションの作成** |
    | ソリューション名 | **EShopOnWeb.Shared** |

    **[ソリューションとプロジェクトを同じディレクトリに配置する]** の設定は有効のままにします。

4. [次へ] をクリックします。 **.NET 6.0 (長期サポート)** をフレームワーク オプションとして受け入れます。
5. **[作成]** ボタンを選んで、プロジェクトの作成を確認します。
6. Visual Studio インターフェイス内の **[ソリューション エクスプローラー]** ウィンドウで、 **[Class1.cs]** を右クリックし、右クリック メニューで **[削除]** を選択し、確認を求められたら **[OK]** をクリックします。
7. **Ctrl + Shift + B** キーを押すか、**EShopOnWeb.Shared プロジェクトを右クリック**して **[ビルド]** を選び、プロジェクトをビルドします。

    > **注**:次のタスクでは、**NuGet.exe** を使用して、ビルドされたプロジェクトから直接 NuGet パッケージを生成しますが、最初にプロジェクトをビルドする必要があります。

8. Azure DevOps ポータルを表示している Web ブラウザーに切り替えます。
9. **[フィードに接続]** ペインの **[NuGet]** セクションに移動し、 **[NuGet.exe]** を選択します。 これにより、 **[NuGet.exe]** ペインが表示されます。
10. **[NuGet.exe]** ペインで、 **[ツールの取得]** をクリックします。
11. **[ツールの取得]** ペインで、 **[最新の NuGet をダウンロードする]** リンクをクリックします。 これにより、 **[利用可能な NuGet 配布バージョン]** ページを表示する別のブラウザー タブが自動的に開きます。
12. **[Available NuGet Distribution Versions] (利用可能な NuGet 配布バージョン )** ページで **[nuget.exe - recommended latest v6.x] (nuget.exe - 推奨される最新の v6.x)** を選んで、実行可能ファイルをローカル環境の **EShopOnWeb.Shared プロジェクト** フォルダーにダウンロードします (既定のフォルダーの場所のままにしている場合、これは C:\EShopOnWeb\EShopOnWeb.Shared です)。
13. **nuget.exe** ファイルを選び、ファイルを右クリックしてコンテキスト メニューから **[プロパティ]** を選んで、そのプロパティを開きます。
14. [プロパティ] コンテキスト ウィンドウの **[全般]** タブで、[セキュリティ] セクションの **[ブロック解除]** を選びます。 **[適用]** 、 **[OK]** の順に選んで確定します。
15. ラボ ワークステーションから [スタート] メニューを開き、**Windows PowerShell** を見つけます。 次に、カスケード メニューの **[Windows PowerShell を管理者として開く]** をクリックします。
16. **[管理者: Windows PowerShell]** ウィンドウで、次のコマンドを実行して EShopOnWeb.Shared フォルダーに移動します。

    ```text
    cd c:\EShopOnWeb\EShopOnWeb.Shared
    ```

    次を実行して、プロジェクトから **.nupkg** ファイルを作成します。

    > **注**:これは、デプロイ用に NuGet ビットをパッケージ化するためのショートカットです。 NuGet は高度にカスタマイズ可能です。 詳細については、[NuGet パッケージの作成ページ](https://docs.microsoft.com/nuget/create-packages/overview-and-workflow)を参照してください。

    ```text
    .\nuget.exe pack ./EShopOnWeb.Shared.csproj
    ```

    > **注**: **[管理者:Windows PowerShell]** ウィンドウに表示される警告はすべて無視します。

    > **注**:NuGet は、プロジェクトから識別できる情報に基づいて最小限のパッケージを構築します。 **EShopOnWeb.Shared.1.0.0.nupkg** のような名前であることに注意してください。 そのバージョン番号は、アセンブリから取得されました。

17. パッケージが正常に作成されたら、次を実行してパッケージを **EShopOnWebShared** フィードに発行します。

    > **注**:**API キー**を指定する必要があります。これは、空でない文字列にすることができます。 ここでは **AzDO** を使用しています。 プロンプトが表示されたら、Azure DevOps 組織にサインインします。

    ```text
    .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO EShopOnWeb.Shared.1.0.0.nupkg
    ```

18. パッケージ プッシュ操作の成功の確認を待ちます。
19. Azure DevOps ポータルを表示する Web ブラウザー ウィンドウに切り替え、垂直ナビゲーションペインで **[アーティファクト]** を選択します。
20. **[成果物]** ハブ ペインで、左上隅のドロップダウン リストをクリックし、フィードのリストで **EShopOnWebShared** エントリを選びます。

    > **注**: **EShopOnWebShared** フィードには、新しく発行された NuGet パッケージが含まれているはずです。

21. NuGet パッケージをクリックして、詳細を表示します。

#### タスク 3: オープンソース NuGet パッケージを Azure DevOps パッケージ フィードにインポートする

独自のパッケージを開発するだけでなく、オープンソースの Nuget (https://www.nuget.org) DotNet パッケージ ライブラリを使うこともできます。 数百万個のパッケージを利用できるので、必ず何かアプリケーションに役立つものがあります。

このタスクでは、汎用の "Hello World" サンプル パッケージを使いますが、ライブラリの他のパッケージにも同じ方法を使用できます。

1. 同じ PowerShell ウィンドウから、次の **nuget** コマンドを実行してサンプル パッケージをインストールします。

    ```text
    .\nuget install HelloWorld -ExcludeVersion
    ```

2. インストール プロセスの出力を確認します。 1 行目に、パッケージのダウンロードを試みるさまざまなフィードが示されています。

    ```text
    Feeds used:
      https://api.nuget.org/v3/index.json
      https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/eShopOnWeb/_packaging/EShopOnWebPFeed/nuget/v3/index.json
    ```

3. その次に、実際のインストール プロセス自体に関する追加の出力が示されています。

    ```text
    Installing package 'Helloworld' to 'C:\eShopOnWeb\EShopOnWeb.Shared'.
      GET https://api.nuget.org/v3/registration5-gz-semver2/helloworld/index.json
      OK https://api.nuget.org/v3/registration5-gz-semver2/helloworld/index.json 114ms
    MSBuild auto-detection: using msbuild version '17.5.0.10706' from 'C:\Program Files\Microsoft Visual Studio\2022\Professional\MSBuild\Current\bin'.
      GET https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v3/registrations2-semver2/helloworld/index.json
      OK https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v3/registrations2-semver2/helloworld/index.json 698ms
    
    Attempting to gather dependency information for package 'Helloworld.1.3.0.17' with respect to project 'C:\eShopOnWeb\EShopOnWeb.Shared', targeting 'Any,Version=v0.0'
    Gathering dependency information took 21 ms
    Attempting to resolve dependencies for package 'Helloworld.1.3.0.17' with DependencyBehavior 'Lowest'
    Resolving dependency information took 0 ms
    Resolving actions to install package 'Helloworld.1.3.0.17'
    Resolved actions to install package 'Helloworld.1.3.0.17'
    Retrieving package 'HelloWorld 1.3.0.17' from 'nuget.org'.
      GET https://api.nuget.org/v3-flatcontainer/helloworld/1.3.0.17/helloworld.1.3.0.17.nupkg
      OK https://api.nuget.org/v3-flatcontainer/helloworld/1.3.0.17/helloworld.1.3.0.17.nupkg 133ms
    Installed HelloWorld 1.3.0.17 from https://api.nuget.org/v3/index.json with content hash 1Pbk5sGihV5JCE5hPLC0DirUypeW8hwSzfhD0x0InqpLRSvTEas7sPCVSylJ/KBzoxbGt2Iapg72WPbEYxLX9g==.
    Adding package 'HelloWorld.1.3.0.17' to folder 'C:\eShopOnWeb\EShopOnWeb.Shared'
    Added package 'HelloWorld.1.3.0.17' to folder 'C:\eShopOnWeb\EShopOnWeb.Shared'
    Successfully installed 'HelloWorld 1.3.0.17' to C:\eShopOnWeb\EShopOnWeb.Shared
    Executing nuget actions took 686 ms
    ```

4. HelloWorld パッケージは、EShopOnWeb.Shared フォルダーの下のサブフォルダー **HelloWorld** にインストールされます。 Visual Studio の**ソリューション エクスプローラー**で **EShopOnWeb.Shared** プロジェクトに移動し、**HelloWorld** サブフォルダーに注目します。 サブフォルダーの左側にある小さな矢印をクリックして、フォルダーとファイルの一覧を開きます。
5. **lib** サブフォルダーにある **signature.p7s** 署名ファイルによってパッケージの配信元が証明されていることに注意してください。 次に、**HelloWorld.nupkg** パッケージ ファイル自体に注目してください。

#### タスク 4: オープンソースの NuGet パッケージを Azure Artifacts にアップロードする

このパッケージを、前に作成した Azure Artifacts パッケージ フィードにアップロードすることで、DevOps チームが再利用するための "承認済み" のパッケージであると考えてみましょう。

1. PowerShell ウィンドウで、次のコマンドを実行します。

    ```text
    .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO c:\EShopOnWeb\EShopOnWeb.Shared\HelloWorld\HelloWorld.nupkg
    ```

    > **注**: これにより、エラー メッセージが表示されます。

    ```text
    Response status code does not indicate success: 409 (Conflict - 'HelloWorld 1.3.0.17' cannot be published to the feed because it exists in at least one of the feed's upstream sources. Publishing this copy would prevent you from using 'HelloWorld 1.3.0.17' from 'NuGet Gallery'. For more information, see https://go.microsoft.com/fwlink/?linkid=864880 (DevOps Activity ID: AE08BE89-C2FA-4FF7-89B7-90805C88972C)).
    ```

Azure DevOps Artifacts パッケージ フィードを作成すると、設計上、dotnet の例の nuget.org などの**アップストリーム ソース**に対応します。 ただし、DevOps チームが **"内部専用"** パッケージ フィードを作成することを妨げるものはありません。

1. Azure DevOps ポータルに移動し、 **[成果物]** を参照して、**EShopOnWebShared** フィードを選びます。
2. **[アップストリーム ソースの検索]** をクリックします
3. **[アップストリーム パッケージに移動]** ウィンドウで、パッケージの種類として **Nuget** を選び、検索フィールドに「**HelloWorld**」と入力します。
4. **[検索]** ボタンを選んで確定します。
5. これにより、HelloWorld のパッケージの使用できるさまざまなバージョンの一覧が表示されます。
6. **左方向キー**を押して **EShopOnWebShared** フィードに戻ります。
7. 歯車をクリックして **[フィードの設定]** を開きます。 [フィードの設定] ページで、 **[アップストリーム ソース]** を選びます。
8. さまざまな開発言語用にさまざまなアップストリーム パッケージ マネージャーがあることに注意してください。 一覧から **Nuget.org** を選びます。 **[削除]** ボタン、 **[保存]** ボタンの順に選びます。

9. これらの保存した変更を使い、次のコマンドをもう一度実行することで、PowerShell ウィンドウから Nuget.exe を使って **HelloWorld** パッケージをアップロードできます。

    ```text
     .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO c:\EShopOnWeb\EShopOnWeb.Shared\HelloWorld\HelloWorld.nupkg
    ```

    > **注**: これでアップロードが成功します 

    ```text
    Pushing HelloWorld.nupkg to 'https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/'...
      PUT https://pkgs.dev.azure.com/<AZUREDEVOPSORGANIZATION>/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/
    MSBuild auto-detection: using msbuild version '17.5.0.10706' from 'C:\Program Files\Microsoft Visual Studio\2022\Professional\MSBuild\Current\bin'.
      Accepted https://pkgs.dev.azure.com/pdtdemoworld<AZUREDEVOPSORGANIZATION>/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/ 1645ms
    Your package was pushed.
    PS C:\eShopOnWeb\EShopOnWeb.Shared>
    ```

10. Azure DevOps ポータルで、成果物パッケージ フィード ページを**更新**します。 パッケージの一覧に、**EShopOnWeb.Shared** カスタム開発パッケージと **HelloWorld** パブリック ソース パッケージの両方が表示されます。
11. Visual Studio の **EShopOnWeb.Shared** ソリューションで **EShopOnWeb.Shared** プロジェクトを右クリックし、コンテキスト メニューから **[Nuget パッケージの管理]** を選びます。
12. Nuget パッケージ マネージャー ウィンドウで、 **[パッケージ ソース]** が **EShopOnWebShared** に設定されていることを確認します。
13. **[参照]** をクリックして、Nuget パッケージの一覧が読み込まれるまで待ちます。
14. この一覧には、**EShopOnWeb.Shared** カスタム開発パッケージと **HelloWorld** パブリック ソース パッケージの両方も表示されます。

## 確認

このラボでは、次の手順を使用して Azure Artifacts を操作する方法を学習しました：

- 作成してフィードに接続しました。
- NuGet パッケージを作成して公開しました。
- カスタム開発 NuGet パッケージをインポートしました。
- パブリック ソース NuGet パッケージをインポートしました。
