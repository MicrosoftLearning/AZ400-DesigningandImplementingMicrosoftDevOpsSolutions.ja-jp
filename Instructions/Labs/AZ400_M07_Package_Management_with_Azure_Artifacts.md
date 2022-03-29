---
lab:
  title: ラボ 17:Azure Artifacts によるパッケージ管理
  module: 'Module 07: Design and implement a dependency management strategy'
ms.openlocfilehash: e4172b81e228750517d6b7dc93e2a16755d031c1
ms.sourcegitcommit: f72fcf5ee578f465b3495f3cf789b06c530e88a4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2022
ms.locfileid: "139262537"
---
# <a name="lab-17-package-management-with-azure-artifacts"></a>ラボ 17:Azure Artifacts によるパッケージ管理
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-overview"></a>ラボの概要

Azure Artifacts は、Azure DevOps での NuGet、npm、Maven パッケージの検出、インストール、公開を容易にします。 ビルドなどの他の Azure DevOps 機能と緊密に統合されているため、パッケージ管理は既存のワークフローのシームレスな部分になります。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

-  フィードを作成して接続する。
-  NuGet パッケージを作成して公開する。
-  NuGet パッケージをインポートする。
-  NuGet パッケージを更新する。

## <a name="lab-duration"></a>ラボの所要時間

-   予想所要時間: **40 分**

## <a name="instructions"></a>Instructions

### <a name="before-you-start"></a>開始する前に

#### <a name="sign-in-to-the-lab-virtual-machine"></a>ラボ仮想マシンにサインインする

次の認証情報を使用して、Windows 10 仮想マシンにサインインしていることを確認します。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### <a name="review-applications-required-for-this-lab"></a>このラボで必要なアプリケーションのレビュー

このラボで使用するアプリケーションを特定:
    
-   Microsoft Edge
-   Visual Studio 2019 Community Edition ([Visual Studio Downloads page](https://visualstudio.microsoft.com/downloads/) から入手可能) Visual Studio 2019 のインストールには、**ASP<nolink>.NET および Web 開発**、**Azure 開発**、 **.NET Core クロスプラットフォーム開発** ワークロードを含める必要があります。 これはすでにラボのコンピューターに事前インストールされています。

#### <a name="set-up-an-azure-devops-organization"></a>Azure DevOps 組織を設定する

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)で利用できる手順に従って作成してください。

### <a name="exercise-0-configure-the-lab-prerequisites"></a>演習 0:ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これには、Azure DevOps Demo Generator テンプレートと Visual Studio 構成に基づいて事前構成された Parts Unlimited チーム プロジェクトが含まれます。

#### <a name="task-1-configure-the-team-project"></a>タスク 1:チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用して、**PartsUnlimited** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。 このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**:サイトの詳細については、 https://docs.microsoft.com/en-us/azure/devops/demo-gen を参照してください。

1.  「**サインイン**」をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。
1.  必要な場合は、「**Azure DevOps Demo Generator**」ページで「**承諾する**」をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。
1.  **[新しいプロジェクトの作成]** ページの **[新しいプロジェクト名]** テキストボックスに「A **zure Artifacts を使用したパッケージ管理**」と入力し、 **[組織の選択]** ドロップダウン リストで、Azure DevOps 組織を選択して、 **[テンプレートの選択]** をクリックします。
1.  **[テンプレートの選択]** ページのテンプレートのリストで、 **[PartsUnlimited]** テンプレートをクリックし、 **[テンプレートの選択]** をクリックします。
1.  再び「**新しいプロジェクトの作成**」ページで「**プロジェクトの作成**」をクリックします。

    > **注**:プロセスが完了するまでお待ちください。 これには 2 分ほどかかります。 プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

#### <a name="task-2-configuring-the-parts-unlimited-solution-in-visual-studio"></a>タスク 2:Visual Studio で Parts Unlimited ソリューションを構成する

このタスクでは、ラボの準備をするために Visual Studio を構成します。

1.  Azure DevOps ポータルで **Azure Artifacts を使用したパッケージ管理** チーム プロジェクトを表示していることを確認します。 

    > **注**:プロジェクト ページには、URL [https://dev.azure.com/`<your-Azure-DevOps-account-name>` /Package%20Management%20with%20Azure%20Artifacts](https://dev.azure.com/`<your-Azure-DevOps-account-name>`/Package%20Management%20with%20Azure%20Artifacts) から直接アクセスできます。ここで、`<your-Azure-DevOps-account-name>` プレースホルダーは、ご自分のアカウント名を表します。 

1.  **[Azure Artifacts を使用したパッケージ管理]** ペインの左側にある垂直メニューで、 **[リポジトリ]** をクリックします。
1. **[ファイル]** ペインで、 **[複製]** をクリックし、 **[VS Code で複製]** の横にあるドロップダウン矢印を選択し、ドロップダウン メニューで **[Visual Studio]** を選択します。
1.  続行するかどうかを確認するメッセージが表示されたら、 **[開く]** をクリックします。 
1.  プロンプトが表示されたら、Azure DevOps 組織のセットアップに使用したユーザー アカウントでサインインします。
1.  Visual Studio インターフェイス内の **Azure DevOps** ポップアップ ウィンドウで、デフォルトのローカル パスを受け入れ、 **[複製]** をクリックします。 これにより、プロジェクトが Visual Studio に自動的にインポートされ、移行レポート ページを表示する新しい Web ブラウザー タブが開きます。

    > **注**: **[プロジェクトとソリューションの変更の確認]** ダイアログ ボックスで、サポートされていないプロジェクトの種類に関する警告を確認し、 **[OK]** をクリックします。 

1.  移行レポート ページを表示している Web ブラウザー タブを閉じます。
1.  ラボで使用するために、Visual Studio ウィンドウを開いたままにします。

### <a name="exercise-1-working-with-azure-artifacts"></a>演習 1:Azure Artifacts の操作

この演習では、次の手順を使用して Azure Artifacts を操作する方法を学習します。

- フィードを作成して接続する。
- NuGet パッケージを作成して公開する。
- NuGet パッケージをインポートする。
- NuGet パッケージを更新する。

#### <a name="task-1-creating-and-connecting-to-a-feed"></a>タスク 1:フィードを作成して接続する

このタスクでは、フィードを作成して接続します。

1.  Azure DevOps ポータルでプロジェクト設定を表示している Web ブラウザー ウィンドウの垂直ナビゲーション ペインで、 **[アーティファクト]** を選択します。
1.  **アーティファクト** ハブが表示された状態で、ペインの上部にある **[+ フィードの作成]** をクリックします。 

    > **注**:このフィードは、組織内のユーザーが利用できる NuGet パッケージのコレクションであり、ピアとしてパブリック NuGet フィードと並んで配置されます。 このラボのシナリオでは、Azure Artifacts を使用するためのワークフローに焦点を当てているため、実際のアーキテクチャと開発の決定は純粋に例示的なものです。  このフィードには、この組織のプロジェクト間で共有できる共通の機能が含まれます。 

1.  **[新しいフィードの作成]** ペインの **[名前]** テキストボックスに「**PartsUnlimitedShared**」と入力し、 **[スコープ]** セクションで **[組織]** オプションを選択し、他の設定をデフォルト値のままにして、 **[作成]** をクリックします。 

    > **注**:このNuGet フィードに接続するユーザーは、環境を構成する必要があります。 

1.  **アーティファクト** ハブに戻り、 **[フィードに接続]** をクリックします。
1.  **[フィードに接続]** ペインの **[NuGet]** セクションで **[Visual Studio]** を選択し、 **[Visual Studio]** ペインで **ソース** URL をコピーします。 
1.  **Visual Studio** ウィンドウに戻ります。
1.  Visual Studio ウィンドウで、 **[ツール]** メニュー ヘッダーをクリックし、ドロップダウン メニューで **[NuGet Package Manager]** を選択し、カスケード メニューで **[パッケージ マネージャー設定]** を選択します。
1.  **[オプション]** ダイアログ ボックスで、 **[パッケー ジソース]** をクリックし、プラス記号をクリックして新しいパッケージ ソースを追加します。
1.  ダイアログ ボックスの下部にある **[名前]** テキストボックスで、 **[パッケージ ソース]** を **PartsUnlimitedShared** に置き換え、 **[ソース]** テキストボックスで、コピーした URL を Azure DevOps ポータルに貼り付けます。 
1.  **[更新]** をクリックし、 **[OK]** をクリックして追加を完了します。

    > **注**:Visual Studio が新しいフィードに接続されました。

1.  PartsUnlimited リポジトリの複製に使用した他の Visual Studio インスタンスを閉じて再度開き、アーティファクト ソースの更新を考慮して、**PartsUnlimited** ソリューションを開きます。 この演習の 3 番目のタスクで必要になります。

#### <a name="task-2-creating-and-publishing-a-nuget-package"></a>タスク 2:NuGet パッケージの作成と公開

このタスクでは、NuGet パッケージを作成して公開します。

1.  新しいパッケージソースの構成に使用した Visual Studio ウィンドウで、メイン メニューの **[ファイル]** をクリックし、ドロップダウン メニューの **[新規]** をクリックしてから、カスケード メニューの **[プロジェクト]** をクリックします。 

    > **注**:NuGet パッケージとして公開される共有アセンブリを作成して、他のチームがプロジェクト ソースを直接操作しなくても、アセンブリを統合して最新の状態に保つことができるようにします。

1. **[新しいプロジェクトの作成]** ペインの **[最近のプロジェクト テンプレート]** ページで、検索テキスト ボックスを使用して **クラス ライブラリ (.NET Framework)** テンプレートを見つけて、C# 用のテンプレートを選択し、 **[次へ]** をクリックします。
1.  **[新しいプロジェクトの作成]** ペインの **[クラス ライブラリ (.NET Framework)]** ページで、次の設定を指定し、 **[作成]** をクリックします。

    | 設定 | [値] |
    | --- | --- |
    | プロジェクト名 | **PartsUnlimited.Shared** |
    | 場所 | 既定値を受け入れる |
    | ソリューション | **新しいソリューションの作成** |
    | ソリューション名 | **PartsUnlimited.Shared** |
    | フレームワーク | **.NET Framework 4.5.1** |

    > **注**: **[.NET Standard]** を選択しないようにしてください。

1.  Visual Studio インターフェイス内の **[ソリューション エクスプローラー]** ウィンドウで、 **[Class1.cs]** を右クリックし、右クリック メニューで **[削除]** を選択し、確認を求められたら **[OK]** をクリックします。
1.  Visual Studio インターフェイス内の **[ソリューション エクスプローラー]** ペインで、**PartsUnlimited.Shared** プロジェクト ノードを右クリックし、 **[プロパティ]** を選択します。
1.  **PartsUnlimited.Shared** プロパティ ペイン内で、 **[ターゲット フレームワーク]** が **[.NET Framework 4.5.1]** に設定されていることを確認します。
1.  **Ctrl + Shift + B** キーを押して、プロジェクトをビルドします。 

    > **注**:次のタスクでは、**NuGet.exe** を使用して、ビルドされたプロジェクトから直接 NuGet パッケージを生成しますが、最初にプロジェクトをビルドする必要があります。

1.  Azure DevOps ポータルを表示している Web ブラウザーに切り替えます。 
1.  **[フィードに接続]** ペインの **[NuGet]** セクションに移動し、 **[NuGet.exe]** を選択します。 これにより、 **[NuGet.exe]** ペインが表示されます。
1.  **[NuGet.exe]** ペインで、 **[ツールの取得]** をクリックします。
1.  **[ツールの取得]** ペインで、 **[最新の NuGet をダウンロードする]** リンクをクリックします。 これにより、 **[利用可能な NuGet 配布バージョン]** ページを表示する別のブラウザー タブが自動的に開きます。
1.  **[利用可能な NuGet 配布バージョン]** ページで、nuget.exe のバージョン **v5.5.1** を選択し、実行可能ファイルをローカルの **ダウンロード** フォルダーにダウンロードします。
1.  **Visual Studio** ウィンドウに切り替えます。 **[ソリューション エクスプローラー]** ペインで、**PartsUnlimited.Shared** プロジェクト ノードを右クリックし、右クリック メニューで **[ファイル エクスプローラーでフォルダーを開く]** を選択します。
1.  **[エクスプローラー]** ウィンドウ内で、ダウンロードした **nuget.exe** ファイルをダウンロード フォルダーから **.csproj** ファイルを含むフォルダーに移動します。
1.  同じファイル エクスプローラー ウィンドウで、 **[ファイル]** メニュー ヘッダーを選択し、ドロップダウン メニューで **[Windows PowerShell を開く]** を選択し、カスケード メニューで **[管理者として Windows PowerShell を開く]** をクリックします。 
1.  **管理者:Windows PowerShell** ウィンドウで、次を実行して、プロジェクトから **.nupkg** ファイルを作成します。 

    > **注**:これは、デプロイ用に NuGet ビットをパッケージ化するためのショートカットです。 NuGet は高度にカスタマイズ可能です。 詳細については、[NuGet パッケージの作成ページ](https://docs.microsoft.com/en-us/nuget/create-packages/overview-and-workflowhttps:/docs.microsoft.com/en-us/nuget/create-packages/overview-and-workflow)を参照してください。

    ```
    ./nuget.exe pack ./PartsUnlimited.Shared.csproj
    ```

    > **注**: **[管理者:Windows PowerShell]** ウィンドウに表示される警告はすべて無視します。

    > **注**:NuGet は、プロジェクトから識別できる情報に基づいて最小限のパッケージを構築します。 たとえば、名前が **PartsUnlimited.Shared.1.0.0.nupkg** であることに注意してください。 そのバージョン番号は、アセンブリから取得されました。

1.  **Visual Studio** ウィンドウに戻り、 **[ソリューション エクスプローラー]** ウィンドウで **[PartsUnlimited.Shared \ Properties]** ノードを展開し、 **[AssemblyInfo.cs]** をクリックしてウィンドウの中央ペインで開き、コンテンツを確認します。

    > **注**:**AssemblyVersion** 属性は、アセンブリに組み込むバージョン番号を指定します。 NuGet の各リリースには一意のバージョン番号が必要なため、パッケージの作成にこのメソッドを引き続き使用する場合は、ビルドの前にこれをインクリメントする必要があります。

1.  **[管理者:Windows PowerShell]** ウィンドウに切り替え、次のコマンドを実行して、パッケージを **PartsUnlimitedShared** フィードに公開します。

    > **注**:**API キー** を指定する必要があります。これは、空でない文字列にすることができます。 ここでは **AzDO** を使用しています。 プロンプトが表示されたら、Azure DevOps 組織にサインインします。 

    ```
    ./nuget.exe push -source "PartsUnlimitedShared" -ApiKey AzDO PartsUnlimited.Shared.1.0.0.nupkg
    ```
1.  Azure DevOps ポータルを表示する Web ブラウザー ウィンドウに切り替え、垂直ナビゲーションペインで **[アーティファクト]** を選択します。
1.  **アーティファクト** ハブ ペインで、左上隅のドロップダウンリストをクリックし、フィードのリストで、**PartsUnlimitedShared** エントリを選択します。

    > **注**:**PartsUnlimitedShared** フィードには、新しく公開された NuGet パッケージが含まれている必要があります。 

1.  NuGet パッケージをクリックして、詳細を表示します。

#### <a name="task-3-importing-a-nuget-package"></a>タスク 3:NuGet パッケージのインポート

このタスクでは、NuGet パッケージをインポートします。

1.  **Parts Unlimited** ソリューションを表示している **Visual Studio** ウィンドウに切り替えます。
1.  **[ソリューション エクスプローラー]** ペインで、**PartsUnlimitedWebsite** プロジェクトの下の **[参照]** ノードを右クリックし、右クリックメニューで **[NuGet パッケージの管理]** を選択します。 これにより、ウィンドウの中央のペインに **[NuGet:PartsUnlimitedWebsite]** タブが開きます。
1.  **[NuGet: PartsUnlimitedWebsite]** ペインで、 **[参照]** タブをクリックし、ペインの右上隅にある **[パッケージ ソース]** ドロップダウン リストで、 **[PartsUnlimitedShared]** を選択します。 

    > **注**:パッケージのリストは、追加した単一のパッケージのみで構成されます。

1.  パッケージを選択し、**PartsUnlimited.Shared** ペインで、 **[インストール]** をクリックしてプロジェクトに追加します。
1.  プロンプトが表示されたら、 **[変更のプレビュー]** ダイアログ ボックスで **[OK]** をクリックします。
1.  **Ctrl+Shift+B** を押してプロジェクトをビルドし、ビルドが正常に完了したことを確認します。 

    > **注**:NuGet パッケージはまだ値を追加していませんが、ワークフローが意図したとおりに機能することを確認できました。 

#### <a name="task-4-updating-a-nuget-package"></a>タスク 4:NuGet パッケージの更新

このタスクでは、NuGet パッケージを更新します。

1.  **PartsUnlimited.Shared** プロジェクト (NuGet ソース プロジェクトを含む) が開いている **Visual Studio** ウィンドウに切り替えます。
1.  **[ソリューション エクスプローラー]** ペインで、**PartsUnlimited.Shared** プロジェクト ノードを右クリックし、右クリック メニューで **[追加]** を選択し、カスケード メニューで **[新しい項目]** を選択します。
1. **[新しい項目の追加 - PartsUnlimitedShared]** ダイアログ ボックスの **Visual C# 項目** のリストで、 **[クラス]** テンプレートが選択されていることを確認し、ダイアログ ボックスの下部にある **[名前]** テキストボックスに「**TaxService.cs**」と入力し、 **[追加]** をクリックしてクラスを追加します。 

    > **注**:他のチームが NuGet パッケージを簡単に操作できるように、税計算がこの共有クラスに統合され、一元管理されるように見せかけます。

1.  中央ペインの **TaxService.cs** クラスのコードで、クラスの既存の定義を次のコードに置き換え、ファイルを保存します。

    ```c#
    namespace PartsUnlimited.Shared
    {
        public class TaxService
        {
            static public decimal CalculateTax(decimal taxable, string postalCode)
            {
                return taxable * (decimal).1;
            }
        }
    }
    ```

    > **注**:アセンブリ (およびパッケージ) を更新しているため、アセンブリのバージョンを更新する必要があります。

1.  Visual Studio ウィンドウの中央のペインで、 **[AssemblyInfo.cs]** タブをクリックして、対応するファイルのコンテンツを表示します。 
1.  **AssemblyInfo.cs** ファイルで、`[assembly: AssemblyVersion("1.0.0.0")]` を `[assembly: AssemblyVersion("1.1.0.0")]` に変更し、ファイルを保存します。
1.  **Ctrl + Shift + B** キーを押して、プロジェクトをビルドします。
1.  **[管理者:Windows PowerShell]** ウィンドウに切り替え、次のコマンドを実行して NuGet パッケージを再パッケージ化します。 

    > **注**:新しいパッケージのバージョン番号は更新されます。

    ```
    ./nuget.exe pack PartsUnlimited.Shared.csproj
    ```
1.  **[管理者:Windows PowerShell]** ウィンドウで、次のコマンドを実行して、更新されたパッケージを公開します。 

    > **注**:公開されたアーティファクトのバージョン番号は、パッケージのバージョン更新を反映するように変更されています。

    ```
    ./nuget.exe push -source "PartsUnlimitedShared" -ApiKey AzDO PartsUnlimited.Shared.1.1.0.nupkg
    ```

1.  **PartsUnlimitedShared1.0.0** アーティファクト ペインを備えた Azure DevOps ポータルを表示する Web ブラウザー ウィンドウに切り替えます。
1.  **PartsUnlimitedShared 1.0.0** アーティファクトペインで、 **[バージョン]** タブをクリックし、バージョン **1.0.0** および **1.1.0** が含まれていることを確認します。 
1.  **PartsUnlimited** プロジェクトを表示している **Visual Studio** ウィンドウに戻ります。
1.  **[ソリューション エクスプローラー]** ペインで、**PartsUnlimitedWebsite\Utils\DefaultShippingTaxCalculator.cs** に移動して選択します。 これにより、ウィンドウの中央ペインでファイルが自動的に開きます。
1.  **DefaultShippingTaxCalculator.cs** ファイルのコードで、**20** 行目の **CalculateTax** への呼び出しを見つけて、`tax = CalculateTax(subTotal + shipping, postalCode);` を `tax = PartsUnlimited.Shared.TaxService.CalculateTax(subTotal + shipping, postalCode);` に置き換えます。

    > **注**:元のコードはこのクラスの内部のメソッドを呼び出したため、行の先頭に追加するコードは、NuGet アセンブリからのコードにリダイレクトしています。 ただし、このプロジェクトはまだ NuGet パッケージを更新していないため、1.0.0.0 を参照しており、これらの新しい変更を利用できないため、コードは正しくビルドされません。

1.  **[ソリューション エクスプローラー]** ペインで、 **[参照]** ノードを右クリックし、右クリック メニューで **[NuGet パッケージの管理]** を選択します。

    > **注**: **[更新]** タブの内容で示されているように、NuGet は更新を認識しています。 

1.  **[NuGet: PartsUnlimitedWebsite]** ペインで、 **[更新]** タブをクリックし、検索テキストボックスに「**PartsUnlimited.Shared**」と入力し、ペインの右側の **[バージョン:最新の安定版 1.1.0]** ドロップダウン リストの横にある **[更新]** をクリックして、新しいバージョンをインストールします。 

    > **注**:利用可能な NuGet の更新が多数ある場合がありますが、更新する必要があるのは **PartsUnlimited.Shared** のみです。 パッケージが完全に更新できるようになるまで、少し時間がかかる場合があることに注意してください。 エラーが発生した場合は、しばらく待ってから再試行してください。

1.  プロンプトが表示されたら、 **[変更のプレビュー]** ダイアログ ボックスで **[OK]** をクリックします。
1.  **F5** キーを押して、サイトを構築して実行します。 期待どおりに機能することを確認します。

#### <a name="review"></a>確認

このラボでは、次の手順を使用して Azure Artifacts を操作する方法を学習しました：

- 作成してフィードに接続しました。
- NuGet パッケージを作成して公開しました。
- NuGet パッケージをインポートしました。
- NuGet パッケージを更新しました。
