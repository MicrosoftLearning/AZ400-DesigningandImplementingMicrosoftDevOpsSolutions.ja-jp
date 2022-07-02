---
lab:
  title: ラボ 24:SonarCloud および Azure DevOps による技術的負債の管理
  module: 'Module 10: Implement security and validate code bases for compliance'
ms.openlocfilehash: 6d21a232131d1e5eb7a59c48cea8a6737a9f8dcf
ms.sourcegitcommit: ad90d44825eb733a12cbf9ce1e987948530e09db
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/04/2022
ms.locfileid: "144761312"
---
# <a name="lab-24-managing-technical-debt-with-sonarcloud-and-azure-devops"></a>ラボ 24:SonarCloud および Azure DevOps による技術的負債の管理
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-overview"></a>ラボの概要

Azure DevOps では、"*技術的負債*" という言葉は戦術的な目標を達成する最適以下の手段を意味します。これは、ソフトウェアの開発とデプロイ分野で戦略的な目標能力に悪影響を与えます。 技術的負債は、コードを理解しにくくなる、失敗しやすくなる、変更に時間がかかる、検証が難しいといった点で生産性に影響します。 適切な監視と管理を行わなければ、技術的負債は時間とともに蓄積し、長期的にはソフトウェアの全体的な品質と開発チームの生産性に重大なインパクトを与えます。

[SonarCloud](https://sonarcloud.io/){:target="\_blank"} は、クラウドベースのコード品質およびセキュリティ サービスです。 SonarCloud のおもな特徴は以下の通りです:

- 23 のプログラミング言語とスクリプト言語に対応 (Java、JS、C#、C/C++、Objective-C、TypeScript、Python、ABAP、PLSQL、T-SQL など)。
- 強力な静的コード アナライザーに基づき数千のルールで見つけにくいバグと品質の問題を検出。
- 人気の高い CI サービスを使用したクラウド ベースのインテグレーション （Travis、Azure DevOps、BitBucket、AppVeyor など）。
- ブランチとプル要求のあらゆるソース ファイルを調べる詳細なコード分析で、緑のクオリティ ゲートを達成してビルドを促進。
- スピードとスケーラビリティ。

このラボでは、Azure DevOps Services を SonarCloud に統合する方法を学びます。

> **注**:このラボを実行する前に、Azure DevOps パイプラインを実行できることを確認してください。 2021 年 2 月に行われたパブリック プロジェクトの変更により、パイプラインへのアクセスを要求する必要があります: https://devblogs.microsoft.com/devops/change-in-azure-pipelines-grant-for-public-projects/

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- Azure DevOps プロジェクトと CI ビルドを設定して SonarCloud と統合する
- SonarCloud レポートの分析
- 静的分析を Azure DevOps プル要求プロセスに統合する

## <a name="lab-duration"></a>ラボの所要時間

-   予想所要時間: **60 分**

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

この演習では、ラボの前提条件を設定します。これは、[Sonar Scanning Examples リポジトリ](https://github.com/SonarSource/sonar-scanning-examples.git)に基づくチーム プロジェクトで構成されています。

#### <a name="task-1-create-the-team-project"></a>タスク 1:チーム プロジェクトを作成する

このタスクでは、[Sonar Scanning Examples リポジトリ](https://github.com/SonarSource/sonar-scanning-examples.git)に基づいて新しい Azure DevOps プロジェクトを作成します。

1.  ラボのコンピューターで Web ブラウザーを起動して [**Azure DevOps ポータル**](https://dev.azure.com)に移動し、Azure DevOps 組織にサインインします。
1.  **Azure DevOps ポータル** の右上コーナーで **[+ 新しいプロジェクト]** をクリックします。 
1.  **[新しいプロジェクトの作成]** ペインで **[プロジェクト名]** テキストボックスに「**SonarExamples**」と入力します。**[可視性]** セクションで **[パブリック]** をクリックしてから **[作成]** をクリックします。 
   
    > **注**:SonarCloud の有料プランにサインアップするのでない限り、Azure DevOps プロジェクトがパブリックに設定されていることを確認してください。 有料プランに *サインアップ* するのであれば、プライベート プロジェクトを作成できます。

1.  **[SonarExamples]** ペインで Azure DevOps ポータルの一番左にある垂直メニュー バーで **[SonarExamples は空です。コードを追加してください。]** の上で、 **[リポジトリ]** をクリックします。 **[リポジトリのインポート]** セクションで **[インポート]** をクリックします。
1.  **[Git リポジトリのインポート]** ペインで、**Git** が **[リポジトリの種類]** ドロップダウン リストに表示されていることを確認し、**[URL の複製]** に「**https://github.com/SonarSource/sonar-scanning-examples.git**」と入力して、**[インポート]** をクリックします。 

    > **注**:スキャン例のリポジトリには、MSBuild を使用する C#、Maven、Java を使用した Gradle など、多数のビルド システムと言語のサンプル プロジェクトが含まれています。

#### <a name="task-2-generate-an-azure-devops-personal-access-token"></a>タスク 2:Azure DevOps 個人用アクセス トークンを生成する

このタスクでは、Azure DevOps 個人用アクセス トークンを生成します。これは、この演習の次のタスクでインストールする Postman アプリからの認証に使用されます。

1.   ラボのコンピューターを使い、Azure DevOps ポータルが表示されている Web ブラウザー ウィンドウで Azure DevOps ページの右上コーナーにある **[ユーザー設定]** アイコンをクリックします。ドロップダウン メニューで **[個人用アクセス トークン]** をクリックし、**[個人用アクセス トークン]** ペインで **[+ 新しいトークン]** をクリックします。
1.   **[新しい個人用アクセス トークンの作成]** ペインで **[すべてのスコープを表示]** リンクをクリックし、以下の設定を指定します。**[作成]** をクリックします (他はすべて、既定値のままにします):

     | 設定 | 値 |
     | --- | --- |
     | Name | **SonarCloud および Azure DevOps を使用した技術的負債の管理ラボ** |
     | スコープ | **カスタム定義済み** |
     | スコープ | **コード** |
     | アクセス許可 | **読み取り、書き込み** |

1.   **[成功]** ペインで個人用アクセス トークンの値をクリップボードにコピーします。

     > **注**:必ずトークンの値を記録してください。 このペインを閉じると、取得できなくなります。 

1.   **[成功]** ペインで **[閉じる]** をクリックします。

#### <a name="task-3-install-and-configure-the-sonarcloud-azure-devops-extension"></a>タスク 3:SonarCloud Azure DevOps 拡張機能をインストールして構成する

このタスクでは、SonarCloud Azure DevOps 拡張機能を Azure DevOps プロジェクトでインストールして構成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、Visual Studio Marketplace で [SonarCloud 拡張機能](https://marketplace.visualstudio.com/items?itemName=SonarSource.sonarcloud)に関するページに移動し、**[無料で取得]** をクリックします。ご自分の Azure DevOps 組織の名前が **[Azure Devops 組織の選択]** ドロップダウン リストに表示されていることを確認し、**[インストール]** をクリックします。
1.  インストールが完了したら、**[組織に進む]** をクリックします。 組織のホーム ページを表示している Azure DevOps ポータルにブラウザーがリダイレクトされます。 

    > **注**:マーケットプレースから拡張機能をインストールする適切な許可がない場合は、インストールをっ承認するよう求めるリクエストがアカウント管理者に送られます。

    > **注**:SonarCloud 拡張機能にはビルド タスク、ビルド テンプレート、カスタム ダッシュボード ウィジェットが含まれています。

1.  Web ブラウザー ウィンドウで **SonarCloud ホーム ページ** [https://sonarcloud.io/](https://sonarcloud.io/) に移動します。
1.  SonarCloud ホーム ページで **[ログイン]** をクリックします。
1.  **[SonarCloud にログインまたはサインアップ]** で **[Azure DevOps を使用]** をクリックします。 
1. **[このアプリがあなたの情報にアクセスすることを許可しますか?]** と聞かれたら **[はい]** をクリックします。 ダイアログが表示されたら、 **[組織の代理の同意]** と **[同意]** を選択します。

    > **注**:SonarCloud で組織を作成し、その内部で新しいプロジェクトを作成します。 SonarCloud で設定した組織とプロジェクトは、Azure DevOps で設定した組織とプロジェクトと同じになります。

1.  **[Azure から組織をインポートする]** をクリックします。
1.  **[組織の作成]** ページの **[Azure DevOps 組織名]** テキストボックスにご自分の Azure DevOps 組織の名前を入力します。**[個人用アクセス トークン]** テキストボックスに、前の演習で記録しておいたトークンの値を貼り付け、**[続行]** をクリックします。 
1.  **[組織の詳細をインポート]** セクションの **[キー]** テキストボックスに、ご自分の組織を示す文字列を入力し、**[続行]** をクリックします。

    > **注**:キーは SonarCloud システム内で一意のものでなくてはなりません。 **[キー]** テキストボックスの右側に緑色のチェックマークが表示されていることを確認してください。 これは、キーが一意性の前提条件を満たしていることを示唆します。

1.  **[プランの選択]** セクションで、このラボで使用したいプラン ( **[無料]** を推奨します) を選択し、 **[組織の作成]** をクリックします。

    > **注**:これで Azure DevOps 組織と同一の SonarCloud 組織が作成されました。

    > **注**: 次に、新しく作成された組織内で、Azure DevOps プロジェクト **SonarExamples** と同一の SonarCloud プロジェクトを作成します。 

1.  **[プロジェクトの分析 - リポジトリの選択]** ページの Azure DevOps プロジェクトの一覧で、**[SonarExamples / SonarExamples]** エントリの横にあるチェックボックスを選択し、**[セットアップ]** をクリックします。
1.  **[分析方法の選択]** ページで、 **[Azure DevOps パイプラインを使用]** タイルを選択します。 
1.  **[Azure Pipelines を使用して分析]** ページの **[拡張機能をインストール]** セクションで、**[続行]** をクリックします。

    > **注**:すでにインストールしている場合は拡張機能の作成をスキップできます。 
1.  **[新しい Sonarcloud サービス エンドポイントの追加]** で、Azure DevOps プロジェクトで説明されている手順に従って、サービス接続に **SonarSC** という名前を付け、すべてのパイプラインへのアクセスを許可するボックスを **オン** にして、 **[確認して保存]** をクリックします。 Sonarcloud Web サイトに戻り、 **[続行]** をクリックします。
1.  **Azure Pipelines を使用して分析** ページの **[Azure Pipelines を構成]** セクションで **[.NET]** をクリックします。 **[分析構成の準備]**、**[コード分析の実行]**、**[クオリティ ゲート結果の発行]** で必要な一連の手順が表示されます。 パイプラインの定義には、次の手順が必要です。 

    > **注**:各目標を達成する手順の一覧をレビューします。 これは、後続のタスクで実装します。

    > **注**: SonarCloud サービス エンドポイントのセットアップに必要なトークンの値と、**[プロジェクト キー]** および **[プロジェクト名]** の値を記録しておきます。


### <a name="exercise-1-set-up-an-azure-devops-pipeline-that-integrates-with-sonarcloud"></a>演習 1:SonarCloud と統合する Azure DevOps パイプラインを設定する

この演習では、SonarCloud と統合する Azure DevOps パイプラインを設定します。

> **注**:SonarCloud と統合して **SonarExamples** コードを分析する新しいビルド パイプラインを設定します。 

#### <a name="task-1-initiate-creation-of-the-project-build-pipeline"></a>タスク 1:プロジェクト ビルド パイプラインの作成を開始する

このタスクでは、プロジェクト用のビルド パイプラインの作成を開始します。

1. Azure DevOps ポータルで **[SonarExamples]** ペインが表示されている Web ブラウザー ウィンドウに切り替えます。 **[プロジェクトの設定]** に移動し、 **[可視性]** を [プライベート] に変更して、 **[保存]** を選択します。

    > **注**:これは、Mod 00 の手順に従ってプライベート プロジェクト専用の並列ジョブをセットアップし、組織が現在パブリック プロジェクトで使用できるジョブがない場合に行う必要があります。

#### <a name="task-2-create-a-pipeline-by-using-the-yaml-editor"></a>タスク 2:YAML エディターを使用してパイプラインを作成する

このタスクでは、YAML エディターを使用してパイプラインを作成します。

> **注**:YAML パイプラインの構成を進める前に、まず、SonarCloud のサービス接続を作成します。

1.  Azure DevOps ポータルの左端にある垂直メニュー バーで、 **[パイプライン]** をクリックし、 **[パイプラインの作成]** をクリックします。
1.  **[コードはどこにありますか?]** ペインで **[Azure Repos Git]** をクリックします。
1.  **[リポジトリの選択]** ペインで **[SonarExamples]** をクリックします。 
1.  **[パイプラインの構成]** ペインで **[.NET Desktop]** YAML テンプレートをクリックします。

    > **注**:テンプレート YAML ファイルが開いている状態で YAML エディターが自動的に表示されます。 適正に構成するには、以下のファイルに一致するように調整 (または置換) が必要になります:

    ```yaml
    trigger:
    - master

    pool:
      vmImage: 'windows-latest'

    variables:
      buildConfiguration: 'Release'
      buildPlatform: 'any cpu'

    steps:
    - task: NuGetToolInstaller@0
      displayName: 'Use NuGet 4.4.1'
      inputs:
        versionSpec: 4.4.1

    - task: NuGetCommand@2
      displayName: 'NuGet restore'
      inputs:
        restoreSolution: 'SomeConsoleApplication.sln'

    - task: SonarCloudPrepare@1
      displayName: 'Prepare analysis configuration'
      inputs:
        SonarCloud: 'SC'
        organization: 'myorga'
        scannerMode: 'MSBuild'
        projectKey: 'dotnet-framework-on-azdo'
        projectName: 'Sample .NET Framework project with Azure DevOps'


    - task: VSBuild@1
      displayName: 'Build solution **\*.sln'
      inputs:
        solution: 'SomeConsoleApplication.sln'
        platform: '$(BuildPlatform)'
        configuration: '$(BuildConfiguration)'

    - task: VSTest@2
      displayName: 'VsTest - testAssemblies'
      inputs:
        testAssemblyVer2: |
         **\$(BuildConfiguration)\*Test*.dll
         !**\obj\**
        codeCoverageEnabled: true
        platform: '$(BuildPlatform)'
        configuration: '$(BuildConfiguration)'

    - task: SonarCloudAnalyze@1
      displayName: 'Run SonarCloud analysis'

    - task: SonarCloudPublish@1
      displayName: 'Publish results on build summary'
    ```

    > **注**:**Net-desktop-sonarcloud.yml** ファイルを [SonarSource GitHub リポジトリ](https://github.com/SonarSource/sonar-scanner-vsts/blob/master/yaml-pipeline-templates/net-desktop-sonarcloud.yml)からダウンロードできます。

    > **注**:YAML パイプラインは、このタスクの残りの手順に従って変更する必要があります。 

1. **NuGetCommand@2** タスクで `restoreSolution: 'SomeConsoleApplication.sln'` を `restoreSolution: '**\SomeConsoleApplication.sln'` に置き換え、このソリューションがリポジトリのルートにないことを考慮します。
1.  **SonarCloudPrepare@1** タスクで、 **[設定]** オプションをクリックしてビジュアル ヘルパーを開き、ドロップダウンから、作成した **[sonarSC]** サービスを選択し、**Sonarcloud Web サイト > [Azure パイプラインの構成]** セクションで提案されているフィールドの値を置き換えます。 **[追加]** をクリックして、パイプラインの変更を含めます。
1.  **VSBuild@1** タスクで `solution: 'SomeConsoleApplication.sln'` を `solution: '**\SomeConsoleApplication.sln'` に置き換え、このソリューションがリポジトリのルートにないことを考慮します。
1.  **[パイプライン YAML のレビュー]** ペインで **[保存して実行]** をクリックし、**[保存して実行]** ペインで **[保存して実行]** をクリックします。

    > **注**:YAML エディターでこのタスクを完了した場合は、次のタスクをスキップしてください。

1. [Azure パイプライン] > [パイプライン] の順に移動し、 **[Sonarexamples]** パイプラインをクリックして、パイプラインが完了するまで待ちます。

#### <a name="task-3-check-pipeline-results"></a>タスク 3:パイプラインの結果を確認する

このタスクでは、パイプラインの結果を確認します。

1.  ビルド実行ペイン (以前に作成された YAML または Classic のいずれか) で、**[概要]** タブの内容をレビューしてから **[拡張機能]** タブのヘッダーをクリックします。

    > **注**: **[クオリティ ゲート結果の発行]** タスクを有効なままにしておいた場合は、**[拡張機能]** タブに SonarCloud 分析レポートの概要が含まれています。

1.  **[拡張機能]** タブで **[詳細な SonarCloud レポート]** をクリックします。 新しいブラウザー タブが自動的に開き、SonarCloud プロジェクト ページのレポートが表示されます。

    > **注**:SonarCloud プロジェクトを参照することもできます。 

1.  レポートにクオリティ ゲート結果が含まれていないことを確認し、その理由をメモしておきます。

    > **注**: クオリティ ゲート結果を表示できるようにするには、最初のレポートの実行後、**[新しいコード定義]** を設定する必要があります。 これにより、その後のパイプライン実行にはクオリティ ゲート結果が含まれるようになります。

1.  SonarCloud プロジェクト (Sonarcloud Web サイト) の **[概要]** で、 **[管理]** アイコン (左側の列) と **[新しいコード]** をクリックします。 
1.  SonarCloud プロジェクトの **[新しいコード]** タブで、 **[以前のバージョン]** をクリックします。
1.  **Azure DevOps ポータル** で最も直近のビルド実行と共に **[SonarExamples]** プロジェクト ペインが表示されている Web ブラウザー ウィンドウに切り替えて、 **[新しい実行]** をクリックし、 **[パイプラインの実行]** ペインで、 **[実行]** をクリックします。
1.  ビルド実行ペインで、**[概要]** タブの内容をレビューしてから **[拡張機能]** タブのヘッダーをクリックします。
1.  **[拡張機能]** タブで **[詳細な SonarCloud レポート]** をクリックします。 新しいブラウザー タブが自動的に開き、SonarCloud プロジェクト ページのレポートが表示されます。
1.  レポートと、Azure DevOps の **[拡張機能]** タブに、 **[品質ゲートの結果]** が含まれるようになったことを確認します。

    > **注**:これで SonarCloud で新しい組織が作成され、Azure DevOps ビルドを構成して、分析を行い、ビルドの結果を SonarCloud にプッシュできるようになりました。

### <a name="exercise-2-analyze-sonarcloud-reports"></a>演習 2:SonarCloud レポートの分析

この演習では、SonarCloud レポートを分析します。

#### <a name="task-1-analyze-sonarcloud-reports"></a>タスク 1:SonarCloud レポートの分析

このタスクでは、SonarCloud レポートを分析します。

1.  SonarCloud プロジェクトの **[概要]** タブに、**メイン** ブランチに関するレポートの概要が表示されます。 **[メイン ブランチ]** アイコン (左側の列) をクリックし、 **[コード全体]** をクリックすると、より詳細なレポートが表示されます。

    > **注**:このページには、 **[コード スメル]** 、 **[カバレッジ]** 、 **[重複]** 、 **[サイズ]** (コードの行数) などのメトリクスが含まれています。 以下の表はそれぞれの用語について簡単に説明したものです。

    | 用語 | 説明 |
    | --- | --- |
    | **バグ** | コードのエラーを示す問題。 バグでまだ不具合が生じていない場合は、おそらく最悪のタイミングで発生するでしょう。 これは修正しておく必要があります。 |
    | **脆弱性** | 攻撃者に潜在的なバックドアを提供するセキュリティ関連の問題 |
    | **コードの臭い** | コードのメンテナンス性関連の問題。 そのままにしておくと、メンテナンス担当者が変更を加える際、通常よりも作業が困難になります。 最悪の場合はコードの状態が混乱し、変更を加えるとさらにエラーが発生する可能性があります |
    | **Coverage** | 単体テストなどのテストで検証中のコードの割合を示します。 バグから効果的に防御するには、このようなテストをコードの大部分で行うかカバーする必要があります |
    | **重複** | 重複修飾は、ソース コードのどの部分が重複しているのかを示します |
    | **[サイズ]** | プロジェクト内のコードのライン数です。ステートメント、関数、クラス、ファイル、ディレクトリの数が含まれます |

    > **注**:バグ カウントの隣に表示されている文字は **信頼性の評価** を示します。 特に "**C**" という文字は、コードに少なくとも 1 つの重大なバグがあることを示唆します。 信頼性評価の詳細については、[SonarQube ドキュメント](https://docs.sonarqube.org/display/SONAR/Metric+Definitions#MetricDefinitions-Reliability)を参照してください。 [ルールの種類](https://docs.sonarqube.org/latest/user-guide/rules/)に関する詳細も記載されています。[重要度](https://docs.sonarqube.org/latest/user-guide/issues/)を参照してください。

1.  **バグ** の数を示す数字をクリックします。 **[問題]** タブの内容が自動的に表示されます。 
1.  **[問題]** タブの右側で、バグを示す大きな長方形をクリックすると該当するコードが表示されます。 

    > **注**: **Program.cs** ファイルのライン番号 9 でエラーの詳細を確認します。"**常に 'true' にならないようこの条件を変更してください。後続のコードには実行されないものもあります**" という推奨も含まれます。

1.  コードとライン番号の間の垂直線上にマウスのポインターを配置すると、コードのカバレッジのギャップを特定できます。

    > **注**:サンプル プロジェクトは非常に小規模なので履歴データは含まれていません。 ただし、[SonarCloud のパブリック プロジェクト](https://sonarcloud.io/explore/projects)は多数あり、より興味深く現実的な結果を閲覧できます。

### <a name="exercise-3-implement-azure-devops-pull-request-integration-with-sonarcloud"></a>演習 3:Azure DevOps プル要求を SonarCloud と統合する

この演習では、Azure DevOps と SonarCloud 間のプル要求の統合を設定します。

> **注**:Azure DevOps プル要求に含まれているコードの分析を行うよう SonarCloud 分析を構成するには、以下のタスクを実行する必要があります:

- Azure DevOps 個人用アクセス トークンを SonarCloud プロジェクトに追加します (これによりプル要求へのアクセスが承認されます)。
- プル要求でトリガーされるビルドを管理する Azure DevOps ブランチ ポリシーを構成します。

#### <a name="task-1-create-an-azure-devops-personal-access-token-for-pull-request-integration-with-sonarcloud"></a>タスク 1:SonarCloud とのプル要求統合向けに Azure DevOps 個人用アクセス トークンを作成する

このタスクでは、Azure DevOps プル要求を SonarCloud プロジェクトと統合するための個人用アクセス トークン要件をレビューします。

1.  Azure DevOps ポータルで **SonarExamples** プロジェクトが表示されている Web ブラウザー ウィンドウに切り替えます。
1.  このラボで以前に生成した Azure DevOps 個人用アクセス トークンを **再使用** します。または、 **[コード]** スコープと、**SonarExamples** への **読み取りおよび書き込み** アクセス許可を持つ個人用アクセス トークンを生成するために、このラボで以前に説明した手順を繰り返します。 

    > **注**:このラボで以前に生成した個人用アクセス トークンを再使用することもできます。

    > **注**:プル要求への SonarCloud コメントは、個人用アクセス トークンを作成したユーザーのセキュリティを考慮して追加されます。 この目的で別個の "ボット" Azure DevOps ユーザーを作成し、SonarCloud からのコメントを明確に識別できるようお勧めします。

#### <a name="task-2-configure-pull-request-integration-in-sonarcloud"></a>タスク 2:SonarCloud でプル要求統合を構成する

このタスクでは、Azure DevOps 個人用アクセス トークンを SonarCloud プロジェクトに割り当てて、SonarCloud でプル要求統合を構成します。

1.  **SonarCloud ポータル** で **[SonarExamples]** プロジェクトが表示されている Web ブラウザー ウィンドウに切り替えます。 
1.  プロジェクトのダッシュボード ページで、 **[管理]** タブのアイコンをクリックし、ドロップダウン メニューで、 **[全般設定]** をクリックします。
1.  **[全般設定]** ページで **[プル要求]** をクリックします。
1.  **[プル要求]** 設定の **[全般]** セクションで **[プロバイダー]** ドロップダウン リストから **[Azure DevOps Services]** を選択して **[保存]** をクリックします。
1.  **[プル要求]** 設定の **[Azure DevOps Services との統合]** セクションで、**[個人用アクセス トークン]** テキストボックスに、以前に生成した Azure DevOps 個人用アクセス トークンを貼り付けて、**[保存]** をクリックします。

#### <a name="task-3-configure-a-branch-policy-for-integration-with-sonarcloud"></a>タスク 3:SonarCloud との統合向けのブランチ ポリシーを構成する

このタスクでは、SonarCloudとの統合向けに Azure DevOps ブランチ ポリシーを構成します。

1.  **Azure DevOps ポータル** で **[SonarExamples]** プロジェクトが表示されている Web ブラウザー ウィンドウに切り替えます。
1.  Azure DevOps ポータルの一番左にある垂直メニュー バーで **[リポジトリ]** をクリックし、**[リポジトリ]** セクションで **[ブランチ]** をクリックします。 
1.  **[ブランチ]** ペインのブランチ一覧で、マウスのポインターを **[マスター]** ブランチ エントリの右端に配置すると、垂直の省略記号が現れ、**[その他のオプション]** メニューが表示されます。これをクリックし、ポップアップ メニューで **[ブランチ ポリシー]** をクリックします。
1.  **[マスター]** ペインの [**ビルドの検証]** セクションの右側で **+** をクリックします。
1.  **[ビルド ポリシーの追加]** ペインの **[ビルド パイプライン]** ドロップダウン リストで、このラボで先ほど作成したパイプラインを選択します。**[表示名]** テキストボックスに「**SonarCloud 分析**」と入力し、**[保存]** をクリックします。

    > **注**:これで Azure DevOps は **マスター** ブランチを対象にしたプル要求が作成されると、 SonarCloud 分析をトリガーするよう構成されました。

#### <a name="task-4-validate-pull-request-integration"></a>タスク 4:プル要求の統合を検証する

このタスクでは、プル要求を作成し、その結果をレビューすることにより、Azure DevOps と SonarCloud 間のプル要求統合を検証します。

> **注**:リポジトリのファイルに変更を加え、SonarCloud 分析をトリガーする要求を作成します。

1.  Azure DevOps ポータルの左側にある垂直メニュー バーで **[リポジトリ]** をクリックします。 **[ファイル]** ペインが表示されます。 
1.  中央のペインのフォルダー階層で **sonarqube-scanner-msbuild\\CSharpProject\\SomeConsoleApplication** フォルダーの **Program.cs** ファイルに移動し、 **[編集]** をクリックします。
1.  **[Program.cs]** ペインで、`public static bool AlwaysReturnsTrue()` 行のすぐ上にあるコードに以下の空のメソッドを追加します 

    ```csharp
    public void Unused(){

    }
    ```

1.  **[Program.cs]** ペインで、**[コミット]** をクリックします。
1.  **[コミット]** ペインで **[ブランチ名]** テキストボックスに **[branch1]** と入力します。**[プル要求の作成]** チェックボックスを選択し、**[コミット]** をクリックします。
1.  **[新しいプル リクエスト]** ペインで **[作成]** を選択します。 
1.  **[Updated Program.cs]** ペインの **[概要]** タブで、ビルド プロセスの完了までの進捗状況を監視します。 
1.  **[Updated Program.cs]** ペインの **[概要]** タブで、必要なチェックを行う一方、オプションのチェックが失敗したことを認識し、**[3 チェックを表示]** リンクをクリックします。
1.  **[チェック]** ペインで SonarCloud チェックの結果をレビューし、ペインを閉じます。

    > **注**:結果は、分析ビルドが完了したことを示していますが、PR の新しいコードはコード品質チェックに失敗しています。 発見された新しい問題について、コメントが PR に投稿されています。

1.  **[Updated Program.cs]** ペインの **[概要]** タブに戻り、コメントが含まれているセクションまでスクロールダウンし、新しく追加されたクラスに関する SonarCloud からのコメントをレビューします。 

    > **注**:報告された問題には、プル要求に対応するコードの変更のみが含まれています。 **Program.cs** と他のファイルの既存の問題は無視されます。

#### <a name="task-4-block-pull-requests-in-response-to-failing-code-quality-checks"></a>タスク 4:コード品質チェックの失敗に対応するプル要求をブロックする

このタスクでは、コード品質チェックの失敗に対応するプル要求のブロックを構成します。 

> **注**:この時点では、コード品質チェックが失敗しても、まだプル要求を完了して、該当する変更をコミットできます。 適切なコード品質チェックに合格しなければ、Azure DevOps 構成を変更してコミットをブロックします。

1.  Azure DevOps ポータルの **[Updated Programs.cs]** ペインの左下コーナーで **[プロジェクトの設定]** をクリックします。
1.  **[プロジェクトの設定]** 垂直メニューの **[リポジトリ]** セクションで **[リポジトリ]** をクリックします。
1.  **[すべてのリポジトリ]** ペインで **[SonarExamples]** をクリックします。
1.  **[SonarExamples]** ペインで **[ポリシー]** タブのヘッダーをクリックします。
1.  **[ポリシー]** 一覧でブランチのリストまでスクロールダウンし、**[マスター]** ブランチを示すエントリをクリックします。
1.  **[マスター]** ペインで **[状態の確認]** セクションまでスクロールダウンし、 **+** をクリックします。
1.  **[ステータス ポリシーの追加]** ペインの **[確認する状態]** ドロップダウン リストで **[SonarCloud / クオリティ ゲート]** エントリを選択し、**[ポリシー要件]** オプションが **[必須**] に設定されていることを確認して **[保存]** をクリックします。

    > **注**:この時点でユーザーは、コード品質チェックに成功するまでプル要求をマージできなくなります。 また、SonarCloud で識別された問題はすべて修正済みか、該当する SonarCloud プロジェクトで **確認済み** または **解決済み** としてマークされていなくてはなりません。

## <a name="review"></a>確認

このラボでは、Azure DevOps Services を SonarCloud に統合する方法を学習しました。
