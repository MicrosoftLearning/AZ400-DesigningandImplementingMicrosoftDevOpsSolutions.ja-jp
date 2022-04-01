---
lab:
  title: ラボ 00:ラボ環境を検証する
  module: 'Module 0: Welcome'
ms.openlocfilehash: 082d0853be7780d0a49c949d9c60851a7f30b7a8
ms.sourcegitcommit: 7001996e0238a23639a4d9ce16e68a201393abfb
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/22/2022
ms.locfileid: "140748969"
---
# <a name="lab-00-validate-lab-environment"></a>ラボ 00:ラボ環境を検証する
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="instructions"></a>Instructions

1. 講師またはその他のソースから、新しい **Azure Pass プロモーションコード** (30 日間有効) を入手します。
2. プライベート ブラウザー セッションを使用して、[Microsoft アカウント で](https://account.microsoft.com)新しい **Microsoft アカウント (MSA)** を取得するか、既存のアカウントを使用します。
3. 同じブラウザー セッションを使用して、[Microsoft Azure Pass](https://www.microsoftazurepass.com) にアクセスし、Microsoft アカウント (MSA) を使用して Azure Pass を利用します。 詳しくは、[Microsoft Azure Pass の引き換え](https://www.microsoftazurepass.com/Home/HowTo?Length=5)に関するページをご覧ください。 引き換えの手順に従います。 
4. 同じブラウザー セッションを使用して、[Microsoft Azure](https://portal.azure.com) にアクセスし、ポータル画面の上部で **Azure DevOps** を検索します。 表示されたページで、「**Azure DevOps 組織**」をクリックします。 
5. 次に、**My Azure DevOps Organizations** というラベルの付いたリンクをクリックします（または [自分の情報](https://aex.dev.azure.com)に直接移動します)。
6. **[We need a few more details]\(詳細情報をいくつか入力する必要があります\)** ページで、 **[続行]** を選びます。
7. 左側のドロップダウン ボックスで、"Microsoft アカウント" の代わりに「**既定のディレクトリ**」を選択します。
8. プロンプト ( *[We need a few more details]\(詳細情報をいくつか入力する必要があります\)* ) が表示されたら、名前、メールアドレス、場所を入力して、 **[続行]** をクリックします。
9. **[既定のディレクトリ]** を選択した状態で [自分の情報](https://aex.dev.azure.com)に戻り、青いボタン **[新しい組織の作成]** をクリックします。
10. **[続行]** をクリックして *利用規約* に同意します。
11. プロンプト ( *[Almost done]\(ほぼ完了\)* ) が表示されたら、Azure DevOps 組織の名前を既定のままにし (グローバルに一意の名前である必要があります)、一覧から最寄りのホスティング場所を選びます。
12. 新しく作成した組織が **Azure DevOps** で開いたら、左下隅にある **[組織の設定]** をクリックします。
13. **[組織の設定]** 画面で、 **[課金]** をクリックします (この画面を開くには数秒かかります)。
14. **[課金の設定]** をクリックし、画面の右側で **[Azure Pass] - [スポンサーシップ** サブスクリプション] を選択し、 **[保存]** をクリックしてサブスクリプションを組織にリンクします。
15. 画面の上部にリンクされた Azure サブスクリプション ID が表示されたら、**MS Hosted CI/CD** の **有料並列ジョブ** の数を 0 から **1** に変更します。 次に、下部にある **[保存]** をクリックします。 
16. 新しい設定がバックエンドに反映されるように、**CI/CD 機能を使用する前に少なくとも 3 時間待ちます**。 それ以外の場合は、 *"リクエストの最大数に達したため、このエージェントは実行されていません…"* というメッセージが表示されます。
17. [組織の設定] で、[セキュリティ] -> **[ポリシー]** に移動します
18. [OAuth を使用したサード パーティ アプリケーションのアクセス] を **オンにします**
    > 注: OAuth 設定は、DemoDevOpsGenerator などのツールで拡張機能を登録できるようにするのに役立ちます。 これがないと、必要な拡張機能が不足して、いくつかのラボが失敗するおそれがあります。
19. [パブリック プロジェクトを許可します] を **オンにします**
    > 注: パブリック プロジェクト設定では、一部のラボで、いくつかの拡張機能の無料バージョンを使用できます。
20. オプション: ビルド パイプラインを作成してトリガーすることで、この新しい設定が正常に適用されたことを検証できます。 これを行うには、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) を使用して、講師に相談するか、課金を有効にして新しく作成した組織にデモプロジェクトを作成します。
