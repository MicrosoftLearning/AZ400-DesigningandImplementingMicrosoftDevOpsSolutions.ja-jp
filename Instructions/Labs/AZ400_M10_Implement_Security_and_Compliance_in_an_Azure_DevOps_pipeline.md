---
lab:
  title: 'ラボ 23: Azure DevOps パイプラインにセキュリティとコンプライアンスを実装する'
  module: 'Module 10: Implement security and validate code bases for compliance'
ms.openlocfilehash: 3cf0a6e4824c90766f95c3cfe051993231c8d567
ms.sourcegitcommit: f72fcf5ee578f465b3495f3cf789b06c530e88a4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2022
ms.locfileid: "139262525"
---
# <a name="lab-23-implement-security-and-compliance-in-an-azure-devops-pipeline"></a>ラボ 23: Azure DevOps パイプラインにセキュリティとコンプライアンスを実装する
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-overview"></a>ラボの概要

このラボでは、**WhiteSource Bolt と Azure DevOps** を使用して、脆弱なオープン ソース コンポーネント、古いライブラリ、コードのライセンス コンプライアンスの問題を自動的に検出します。 WebGoat を使用しますが、これは一般的な Web アプリケーションのセキュリティの問題を説明するために設計された OWASP によって維持される、意図的に安全でない Web アプリケーションです。

[WhiteSource](https://www.whitesourcesoftware.com/) は、トップクラスの継続的オープン ソース ソフトウェア セキュリティおよびコンプライアンス管理のリーダーです。 WhiteSource は、プログラミング言語、ビルド ツール、または開発環境に関係なく、ビルド プロセスに統合します。 自動的かつ継続的にバックグラウンドでサイレントに機能し、WhiteSource の常に更新されているオープン ソース リポジトリの最終データベースに対してオープン ソース コンポーネントのセキュリティ、ライセンス、品質を確認します。

WhiteSource の提供する WhiteSource Bolt は、特に Azure DevOps および Azure DevOps Server との統合向けに開発されたライトウェイト オープン ソース セキュリティおよび管理ソリューションです。 WhiteSource Bolt はプロジェクトに基づいて機能し、リアルタイムのアラート機能が備えられていないため、**フル プラットフォーム** が必要です。これは通常、ソフトウェア開発サイクル全体 (リポジトリから開発後段階) を通して、あらゆるプロジェクトと製品でオープン ソース管理の自動化を希望する、より大きな開発チームで推奨されるものです。

Azure DevOps を WhiteSource Bolt に同等すると以下が可能になります:

- 脆弱なオープンソースコンポーネントを検出して修正します。
- プロジェクトまたはビルドごとに包括的なオープン ソース インベントリ レポートを生成します。
- 依存関係のライセンスを含む、オープン ソース ライセンスのコンプライアンスを実施します。
- 更新の推奨事項がある古いオープン ソース ライブラリを特定します。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- WhiteSource Bolt をアクティブにする
- ビルド パイプラインを実行し、WhiteSource のセキュリティおよびコンプライアンス レポートをレビューする

## <a name="lab-duration"></a>ラボの所要時間

-   推定時間:**45 分**

## <a name="instructions"></a>Instructions

### <a name="before-you-start"></a>開始する前に

#### <a name="sign-in-to-the-lab-virtual-machine"></a>ラボ仮想マシンにサインインする

必ず次の資格情報を使用して Windows 10 コンピューターにサインインしてください。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### <a name="review-applications-required-for-this-lab"></a>このラボで必要なアプリケーションのレビュー

このラボで使用するアプリケーションを特定:
    
-   Microsoft Edge

#### <a name="set-up-an-azure-devops-organization"></a>Azure DevOps 組織を設定します。 

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)で利用できる手順に従って作成してください。

### <a name="exercise-0-configure-the-lab-prerequisites"></a>演習 0:ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、[Parts Unlimited MRP GitHub リポジトリ](https://www.github.com/microsoft/partsunlimitedmrp)に基づくリポジトリを含む新しい Azure DevOps プロジェクトで構成されます。

#### <a name="task-1-create-and-configure-the-team-project"></a>タスク 1:チーム プロジェクトを作成して構成する

このタスクでは、Azure DevOps Demo Generator を使用し、[WhiteSource-Bolt テンプレート](https://azuredevopsdemogenerator.azurewebsites.net/?name=WhiteSource-Bolt&templateid=77362)に基づいて新しいプロジェクトを生成します

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。 このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**:サイトの詳細については、 https://docs.microsoft.com/en-us/azure/devops/demo-gen を参照してください。

1.  「**サインイン**」をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。
1.  必要な場合は、「**Azure DevOps Demo Generator**」ページで「**承諾する**」をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。
1.  「**新しいプロジェクトの作成**」ページで「**新しいプロジェクト名**」テキストボックスに「**WhiteSource Bolt**」と入力します。「**組織の選択**」ドロップダウン リストで Azure DevOps 組織を選択し、「**テンプレートの選択**」をクリックします。
1.  テンプレートのリストのツールバーで「**DevOps ラボ**」をクリックし、「**WhiteSource Bolt**」テンプレートを選択して「**テンプレートの選択**」をクリックします。
1.  再び「**新しいプロジェクトの作成**」ページで、欠落している拡張機能をインストールするよう指示されたら「**WhiteSource Bolt**」の下にあるチェックボックスを選択し、「**プロジェクトの作成**」をクリックします。

    > **注**:プロセスが完了するまでお待ちください。 これには 2 分ほどかかります。 プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

### <a name="exercise-1-implement-security-and-compliance-in-an-azure-devops-pipeline-by-using-whitesource-bolt"></a>演習 1:WhiteSource Bolt を使用して Azure DevOps パイプラインにセキュリティとコンプライアンスを実装する

この演習では、WhiteSource Bolt を利用し、セキュリティの脆弱性とライセンスのコンプライアンスの問題に関してプロジェクト コードをスキャンし、その結果作成されるレポートを表示します。

#### <a name="task-1-activate-whitesource-bolt"></a>タスク 1:WhiteSource Bolt をアクティブにする

このタスクでは、新しい生成された Azure Devops プロジェクトで WhiteSource Bolt をアクティブにします。

1.  ラボ コンピューターで、**White Source Bolt** プロジェクトを開いた状態で Azure DevOps ポータルを表示している Web ブラウザー ウィンドウで、Azure DevOps ポータルの左端にある **垂直メニュー バーで**、「**パイプライン**」セクションと「**WhiteSource Bolt**」オプション (「デプロイ グループ」オプションの下の垂直メニューバー) をクリックします。
1.  「**You're almost there（あと一歩）** 」ペインで、「**勤務先の電子メール**」と「**会社名**」を入力し、「**国**」ドロップダウン リストで居住国を示すエントリを選択します。「*作業の開始*」ボタンをクリックして、WhiteSource Bolt の *無料* バージョンの使用を開始します。 新しいブラウザー タブが自動的に開き、「**Bolt の使用開始**」ページが表示されます。 
1.  Azure DevOps ポータルが表示されている Web ブラウザー タブに戻り、「**WhiteSource Bolt の無料バージョンを使用しています**」と表示されていることを確認します。

#### <a name="task-2-trigger-a-build"></a>タスク 2:ビルドをトリガーする

このタスクでは、Java コード ベースの Azure DevOps プロジェクト内でビルドをトリガーします。 **WhiteSource Bolt** 拡張機能を使い、このコードに含まれている脆弱なコンポーネントを特定します。

1.  ラボ コンピューターの左側の垂直メニューバーで、「**パイプライン**」セクションに移動し、「**WhileSourceBolt**」をクリックして、「**パイプラインの実行**」をクリックし、「**パイプラインの実行**」ペインで「**実行**」をクリックします。
1.  「ビルド」ペインの「**サマリー**」タブの「**ジョブ**」セクションで、「**フェーズ 1**」をクリックし、ビルド プロセスの進捗状況を監視します。

    > **注**:このビルドは、完了するまで数分かかる場合があります。 ビルドの定義は以下のタスクで構成されます。

    | タスク | 使用方法 |
    | ---- | ------ |
    | ![npm](images/m19/npm.png) **npm** |  ビルドに必要な npm パッケージをインストールして発行します |
    | ![maven](images/m19/maven.png) **Maven** |  提供された pom xml ファイルを使って Java コードをビルドします |
    | ![whitesourcebolt](images/m19/whitesourcebolt.png) **WhiteSource Bolt** |  提供された作業ディレクトリ / ルート ディレクトリでコードをスキャンし、セキュリティの脆弱性、問題のあるオープン ソース ライセンスを検出します |
    | ![copy-files](images/m19/copy-files.png) **ファイルのコピー** |  一致パターンを使い、結果的な JAR ファイルをソースからコピー先フォルダーにコピーします |
    | ![publish-build-artifacts](images/m19/publish-build-artifacts.png) **ビルド成果物の発行** |  ビルドによって生じた成果物を発行します |
    
1.  ビルドが完了したら、「**サマリー**」タブに戻り、「**テストとカバレッジ**」セクションをレビューします。 

#### <a name="task-3-analyze-reports"></a>タスク 3:レポートを分析する

このタスクでは、WhiteSource Bolt ビルド レポートをレビューします。 

1.  ビルド ペインで「**WhiteSource Bolt ビルド レポート**」タブの見出しをクリックし、レポートが完全に表示されるのを待ちます。 
1.  「**WhiteSource Bolt ビルド レポート**」タブで、WhiteSource Bolt がソフトウェアのオープン ソース コンポーネント (推移的な依存関係とそのライセンスを含む) を自動的に検出したことを確認します。
1.  「**WhiteSource Bolt ビルド レポート**」タブで、ビルド中に発見された脆弱性が表示されたセキュリティ ダッシュボードをレビューします。

    > **注**:レポートには、あらゆる脆弱なオープン ソース コンポーネントのリストが表示されます。この中には、**脆弱性スコア**、**脆弱なライブラリ**、**重大度配分** が含まれます。 あらゆるコンポーネントおよびメタデータとライセンス済み参照へのリンクの詳細なビューを利用して、オープンソース ライセンス配布を特定します。

1.  「**WhiteSource Bolt ビルド レポート**」タブで「**古いライブラリ**」セクションまでスクロールダウンして、その内容をレビューします。

    > **注**:WhiteSource Bolt はプロジェクトの古いライブラリを追跡し、ライブラリの詳細、より新しいバージョンへのリンク、修復の推奨を提供します。

## <a name="review"></a>確認

このラボでは、**WhiteSource Bolt と Azure DevOps** を使用して、脆弱なオープン ソース コンポーネント、古いライブラリ、コードのライセンス コンプライアンスの問題を自動的に検出します。
