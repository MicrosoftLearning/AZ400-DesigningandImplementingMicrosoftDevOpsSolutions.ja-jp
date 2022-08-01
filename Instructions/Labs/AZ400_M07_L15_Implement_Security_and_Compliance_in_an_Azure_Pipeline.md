---
lab:
  title: ラボ 15:Azure Pipeline でのセキュリティとコンプライアンスの実装
  module: 'Module 07: Implement security and validate code bases for compliance'
ms.openlocfilehash: 4c067d30c00b9a07342df3b8390534ba10c4ecf1
ms.sourcegitcommit: 8dc3c31d31d99fd0a2bdd3fb1c7f8f2f04d39778
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/27/2022
ms.locfileid: "147422484"
---
# <a name="lab-15-implement-security-and-compliance-in-an-azure-pipeline"></a>ラボ 15:Azure Pipeline でのセキュリティとコンプライアンスの実装

# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-requirements"></a>ラボの要件

- このラボには、**Microsoft Edge** または [Azure DevOps 対応ブラウザー](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)が必要です。

- **Azure DevOps 組織を設定する:** このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)に関するページの手順に従って作成してください。

## <a name="lab-overview"></a>ラボの概要

このラボでは、**Mend Bolt と Azure DevOps** を使って、脆弱なオープン ソース コンポーネント、古いライブラリ、コードのライセンス コンプライアンスの問題を自動的に検出します。 WebGoat を使いますが、これは一般的な Web アプリケーションのセキュリティの問題を説明するために設計された OWASP によって維持される、意図的に安全でない Web アプリケーションです。

[Mend](https://www.mend.io/) は、継続的オープン ソース ソフトウェア セキュリティおよびコンプライアンス管理のリーダーです。 Mend は、プログラミング言語、ビルド ツール、または開発環境に関係なく、ビルド プロセスに統合します。 自動的かつ継続的にバックグラウンドでサイレントに機能し、Mend の常に更新されているオープン ソース リポジトリの最終データベースに対してオープン ソース コンポーネントのセキュリティ、ライセンス、品質を確認します。

Mend の提供する Mend Bolt は、特に Azure DevOps の統合向けに開発されたライトウェイト オープン ソース セキュリティおよび管理ソリューションです。

> **注**: Mend Bolt はプロジェクトごとに機能し、リアルタイムのアラート機能が備えられていないため、**フル プラットフォーム** が必要です。

Mend Bolt は通常、ソフトウェア開発ライフサイクル全体 (リポジトリから開発後段階) を通して、あらゆるプロジェクトと製品でオープン ソース管理の自動化を希望する、より大きな開発チームで推奨されるものです。

Azure DevOps を Mend Bolt に統合すると以下が可能になります:

- 脆弱なオープンソースコンポーネントを検出して修正します。
- プロジェクトまたはビルドごとに包括的なオープン ソース インベントリ レポートを生成します。
- 依存関係のライセンスを含む、オープン ソース ライセンスのコンプライアンスを実施します。
- 更新の推奨事項がある古いオープン ソース ライブラリを特定します。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- Mend Bolt をアクティブにする。
- ビルド パイプラインを実行し、Mend のセキュリティとコンプライアンス レポートをレビューする。

## <a name="estimated-timing-45-minutes"></a>推定時間:45 分

## <a name="instructions"></a>Instructions

### <a name="exercise-0-configure-the-lab-prerequisites"></a>演習 0:ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、[Parts Unlimited MRP GitHub リポジトリ](https://www.github.com/microsoft/partsunlimitedmrp)に基づくリポジトリを含む新しい Azure DevOps プロジェクトで構成されます。

#### <a name="task-1-create-and-configure-the-team-project"></a>タスク 1:チーム プロジェクトを作成して構成する

このタスクでは、Azure DevOps Demo Generator を使用し、[WhiteSource-Bolt テンプレート (Mend Bolt に改称予定)](https://azuredevopsdemogenerator.azurewebsites.net/?name=WhiteSource-Bolt&templateid=77362) に基づいて新しいプロジェクトを生成します

1. ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。 このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。

    > **注**:サイトの詳細については、 <https://docs.microsoft.com/en-us/azure/devops/demo-gen> を参照してください。

1. **[サインイン]** をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。
1. 必要な場合は、**[Azure DevOps Demo Generator]** ページで **[承諾する]** をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。
1. **[新しいプロジェクトの作成]** ページで **[新しいプロジェクト名]** テキストボックスに「**Mend Bolt**」と入力します。 **[組織の選択]** ドロップダウン リストで Azure DevOps 組織を選択し、 **[テンプレートの選択]** をクリックします。
1. テンプレートのリストのツールバーで **[DevOps ラボ]** をクリックし、 **[WhiteSource Bolt]** (Mend Bolt に改称予定) テンプレートを選択して **[テンプレートの選択]** をクリックします。
1. 再び **[新しいプロジェクトの作成]** ページで、欠落している拡張機能をインストールするダイアログが表示されたら **[Mend Bolt]** の下にあるチェックボックスを選択し、 **[プロジェクトの作成]** をクリックします。

    > **注**:プロセスが完了するまでお待ちください。 これには 2 分ほどかかります。 プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1. **[新しいプロジェクトの作成]** ページで **[プロジェクトに移動]** をクリックします。

### <a name="exercise-1-implement-security-and-compliance-in-an-azure-pipeline-using-mend-bolt"></a>演習 1: Mend Bolt を使用して Azure パイプラインにセキュリティとコンプライアンスを実装する

この演習では、Mend Bolt を利用し、セキュリティの脆弱性とライセンスのコンプライアンスの問題に関してプロジェクト コードをスキャンし、その結果作成されるレポートを表示します。

#### <a name="task-1-activate-mend-bolt"></a>タスク 1: Mend Bolt をアクティブにする

このタスクでは、新しく生成された Azure Devops プロジェクトで Mend Bolt をアクティブにします。

1. ラボ コンピューターで、**Mend Bolt** プロジェクトを開いた状態で Azure DevOps ポータルを表示している Web ブラウザー ウィンドウで、Azure DevOps ポータルの左端にある **垂直メニュー バー** で、 **[パイプライン]** セクションと **[Mend Bolt]** オプション ([デプロイ グループ] オプションの下の垂直メニューバー) をクリックします。
1. **[もう少しです]** ペインで、 **[勤務先の電子メール]** と **[会社名]** を入力し、 **[国]** ドロップダウン リストで居住国を示すエントリを選択します。 *[作業の開始]* ボタンをクリックして、Mend Bolt の *無料* バージョンの使用を開始します。 新しいブラウザー タブが自動的に開き、**[Bolt の使用を開始する]** ページが表示されます。
1. Azure DevOps ポータルが表示されている Web ブラウザー タブに戻り、「**Mend Bolt の無料バージョンを使用しています**」と表示されていることを確認します。

#### <a name="task-2-trigger-a-build"></a>タスク 2:ビルドをトリガーする

このタスクでは、Java コード ベースの Azure DevOps プロジェクト内でビルドをトリガーします。 **Mend Bolt** 拡張機能を使い、このコードに含まれている脆弱なコンポーネントを特定します。

1. ラボ コンピューターの左側の垂直メニューバーで、**[パイプライン]** セクションに移動し、**[WhileSourceBolt]** をクリックして、**[パイプライン実行]** をクリックし、**[パイプライン実行]** ペインで **[実行]** をクリックします。
1. [ビルド] ペインの **[概要]** タブの **[ジョブ]** セクションで、**[フェーズ 1]** をクリックし、ビルド プロセスの進捗状況を監視します。

    > **注**:このビルドは、完了するまで数分かかる場合があります。 ビルドの定義は以下のタスクで構成されます。

    | タスク | 使用方法 |
    | ---- | ------ |
    | ![npm](images/m07/npm.png) **npm** |  ビルドに必要な npm パッケージをインストールして発行します |
    | ![maven](images/m07/maven.png) **Maven** |  提供された pom xml ファイルを使って Java コードをビルドします |
    | ![Mendbolt](images/m07/whitesourcebolt.png) **Mend Bolt** |  提供された作業ディレクトリ / ルート ディレクトリでコードをスキャンし、セキュリティの脆弱性、問題のあるオープン ソース ライセンスを検出します |
    | ![copy-files](images/m07/copy-files.png) **ファイルのコピー** |  一致パターンを使い、結果的な JAR ファイルをソースからコピー先フォルダーにコピーします |
    | ![publish-build-artifacts](images/m07/publish-build-artifacts.png) **ビルド成果物の発行** |  ビルドによって生じた成果物を発行します |

1. ビルドが完了したら、**[概要]** タブに戻り、**[テストとカバレッジ]** セクションをレビューします。

#### <a name="task-3-analyze-reports"></a>タスク 3:レポートを分析する

このタスクでは、Mend Bolt ビルド レポートをレビューします。

1. ビルド ペインで **[Mend Bolt ビルド レポート]** タブのヘッダーをクリックし、レポートが完全に表示されるのを待ちます。
1. **[Mend Bolt ビルド レポート]** タブで、Mend Bolt がソフトウェアのオープン ソース コンポーネント (推移的な依存関係とそのライセンスを含む) を自動的に検出したことを確認します。
1. **[Mend Bolt ビルド レポート]** タブで、ビルド中に発見された脆弱性が表示されたセキュリティ ダッシュボードをレビューします。

    > **注**:レポートには、あらゆる脆弱なオープン ソース コンポーネントのリストが表示されます。この中には、**脆弱性スコア**、**脆弱なライブラリ**、**重大度配分** が含まれます。 あらゆるコンポーネントおよびメタデータとライセンス済み参照へのリンクの詳細なビューを利用して、オープンソース ライセンス配布を特定します。

1. **[Mend Bolt ビルド レポート]** タブで **[古いライブラリ]** セクションまでスクロールダウンして、その内容をレビューします。

    > **注**: Mend Bolt はプロジェクトの古いライブラリを追跡し、ライブラリの詳細、より新しいバージョンへのリンク、修復の推奨を提供します。

## <a name="review"></a>確認

このラボでは、**Mend Bolt と Azure DevOps** を使って、脆弱なオープン ソース コンポーネント、古いライブラリ、コードのライセンス コンプライアンスの問題を自動的に検出します。
