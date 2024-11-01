---
lab:
  title: 動的構成と機能フラグを有効にする
  module: 'Module 04: Implement a secure continuous deployment using Azure Pipelines'
---

# 動的構成と機能フラグを有効にする

## ラボの要件

- このラボには、**Microsoft Edge** または [Azure DevOps 対応ブラウザー](https://learn.microsoft.com/azure/devops/server/compatibility)が必要です。

- **Azure DevOps 組織を設定する:** このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization?view=azure-devops)に関するページの手順に従って作成してください。

- 既存の Azure サブスクリプションを識別するか、新しいものを作成します。

- Azure サブスクリプションで共同作成者または所有者のロールを持つ Microsoft アカウントまたは Microsoft Entra アカウントを持っていることを確認します。 詳細については、[「Azure portal を使用して Azure ロールの割り当てを一覧表示する」](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal)および[「Azure Active Directory で管理者ロールを表示して割当てる」](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal)を参照してください。

## ラボの概要

[Azure App Configuration](https://learn.microsoft.com/azure/azure-app-configuration/overview) は、アプリケーションの設定と機能フラグを一元管理するサービスを提供します。 近年のプログラム、特にクラウドで実行されるものには、分散されたコンポーネントが多数存在するのが一般的です。 これらのコンポーネント全体に構成設定を分散させることは、トラブルシューティングすることの難しいエラーがアプリケーションのデプロイ中に発生する原因となります。 App Configuration を使用すると、アプリケーションのすべての設定を 1 か所に格納して、そのアクセスをセキュリティで保護することができます。

## 目標

このラボを完了すると、次のことができるようになります。

- 動的な構成を有効にする。
- 機能フラグを管理する。

## 推定時間:45 分

## 手順

### 演習 0: ラボの前提条件を構成する (完了している場合はスキップしてください)

この演習では、ラボの前提条件を設定します。これは、[eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) に基づくリポジトリを含む新しい Azure DevOps プロジェクトで構成されます。

#### タスク 1: (完了している場合はスキップしてください) チーム プロジェクトを作成して構成する

このタスクでは、複数のラボで使用される **eShopOnWeb** Azure DevOps プロジェクトを作成します。

1. ラボ コンピューターのブラウザー ウィンドウで、Azure DevOps 組織を開きます。 **[新しいプロジェクト]** をクリックします。 プロジェクトに **eShopOnWeb** という名前を付け、 **[作業項目プロセス]** ドロップダウンで **[スクラム]** を選びます。 **[作成]** をクリックします。

#### タスク 2: (完了している場合はスキップしてください) eShopOnWeb Git リポジトリをインポートする

このタスクでは、複数のラボで使用される eShopOnWeb Git リポジトリをインポートします。

1. ラボ コンピューターのブラウザー ウィンドウで、Azure DevOps 組織と、前に作成した **eShopOnWeb** プロジェクトを開きます。 **[リポジトリ]、[ファイル]**、**[インポート]** の順にクリックします。 **[Git リポジトリをインポートする]** ウィンドウで、URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> を貼り付けて、 **[インポート]** をクリックします。

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

### 演習 1: (完了している場合はスキップしてください) CI/CD パイプラインをインポートして実行する

この演習では、CI/CD パイプラインをインポートして、eShopOnWeb アプリケーションをビルドおよびデプロイします。 CI パイプラインは既に、アプリケーションをビルドしてテストを実行する準備が整っています。 CD パイプラインは、アプリケーションを Azure Web アプリにデプロイします。

#### タスク 1: CI パイプラインをインポートして実行する

まず、[eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml) という CI パイプラインをインポートします。

1. **[パイプライン] > [パイプライン]** に移動します。
1. **[パイプラインの作成]** ボタン (パイプラインがない場合) または **[新しいパイプライン]** ボタン (既に作成されたパイプラインがある場合) をクリックします。
1. **[Azure Repos Git (Yaml)]** を選びます。
1. **eShopOnWeb** リポジトリを選びます。
1. **[既存の Azure Pipelines YAML ファイル]** を選びます。
1. **メイン** ブランチと **/.ado/eshoponweb-ci.yml** ファイルを選択し、**[続行]** をクリックします。
1. **[実行]** ボタンをクリックしてパイプラインを実行します。
1. パイプラインには、プロジェクト名に基づく名前が付けられます。 パイプラインを識別しやすくするために、**名前を変更**しましょう。 **[パイプライン]、[パイプライン]** の順に移動し、先ほど作成したパイプラインをクリックします。 省略記号と **[名前の変更]/[削除]** オプションをクリックします。 **eshoponweb-ci** という名前を付け、 **[保存]** をクリックします。

#### タスク 2: CD パイプラインをインポートして実行する

[eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml) という名前の CD パイプラインをインポートしてみましょう。

1. **[パイプライン] > [パイプライン]** に移動します。
1. **[新しいパイプライン]** ボタンをクリックします。
1. **[Azure Repos Git (Yaml)]** を選びます。
1. **eShopOnWeb** リポジトリを選びます。
1. **[既存の Azure Pipelines YAML ファイル]** を選びます。
1. **メイン** ブランチと **/.ado/eshoponweb-cd-webapp-code.yml** ファイルを選択し、**[続行]** をクリックします
1. YAML パイプライン定義で、次をカスタマイズします。
   - **YOUR-SUBSCRIPTION-ID** を使用する Azure サブスクリプション ID にします。
   - **az400eshop-NAME** の NAME を置き換えて、グローバルに一意にします。
   - **AZ400-EWebShop-NAME** をラボで前に定義したリソース グループ名にします。

1. **[保存および実行]** をクリックし、パイプラインが正常に実行されるまで待ちます。

    > **注**: デプロイが完了するまでに数分かかる場合があります。

    CD の定義は以下のタスクで構成されます。
    - **リソース**: CI パイプラインの完了に基づいて自動的にトリガーされるように準備されています。 また、bicep ファイルのリポジトリもダウンロードします。
    - **AzureResourceManagerTemplateDeployment**: bicep テンプレートを使用して Azure Web Apps をデプロイします。

1. パイプラインには、プロジェクト名に基づく名前が付けられます。 パイプラインを識別しやすくするために、**名前を変更**しましょう。 **[パイプライン]、[パイプライン]** の順に移動し、先ほど作成したパイプラインをクリックします。 省略記号と **[名前の変更]/[削除]** オプションをクリックします。 **eshoponweb-cd-webapp-code** という名前を付け、 **[保存]** をクリックします。

### 演習 2: Azure App Configuration を管理する

この演習では、Azure で App Configuration リソースを作成し、マネージド ID を有効にしてから、ソリューション全体をテストします。

> **注**: この演習にコーディングのスキルは必要ありません。 この Web サイトのコードは、Azure App Configuration の機能を既に実装しています。

アプリケーションにこの機能を実装する方法については、「[ASP.NET Core アプリで動的な構成を使用する](https://learn.microsoft.com/azure/azure-app-configuration/enable-dynamic-configuration-aspnet-core)」と「[Azure App Configuration で機能フラグを管理する](https://learn.microsoft.com/azure/azure-app-configuration/manage-feature-flags)」のチュートリアルを参照してください。

#### タスク 1: App Configuration リソースを作成する

1. Azure Portal で **App Configuration** サービスを検索します
1. **[Create app configuration] (アプリ構成の作成)** をクリックして、次を選びます。
    - 自分の Azure サブスクリプション。
    - 前に作成したリソース グループ (**AZ400-EWebShop-NAME** という名前になっているはずです)。
    - 場所。
    - たとえば、**appcs-NAME-REGION** のような一意の名前。
    - **Free** 価格レベルを選びます。
1. **[確認と作成]** 、 **[作成]** の順にクリックします。
1. App Configuration サービスを作成したら、 **[概要]** に移動し、 **[エンドポイント]** の値をコピーまたは保存します。

#### タスク 2: マネージド ID を有効にする

1. パイプラインを使ってデプロイされた Web アプリに移動します (名前はおそらく **az400-webapp-NAME** です)。
1. **[設定]** セクションで **[ID]** をクリックし、 **[システム割り当て]** セクションで状態を **[オン]** に切り替え、 **[保存] > [はい]** をクリックして、操作が完了するまで数秒待ちます。
1. App Configuration サービスに戻り、 **[アクセス制御]** 、 **[ロールの割り当てを追加]** の順にクリックします。
1. **[ロール]** セクションで **[App Configuration データ閲覧者]** を選びます。
1. **[メンバー]** セクションで **[マネージド ID]** を確認し、Web アプリのマネージド ID (同じ名前である必要があります) を選びます。
1. **[Review and assign] (確認と割り当て)** をクリックします。

#### タスク 3: Web アプリを構成する

Web サイトが App Configuration にアクセスしていることを確認するには、その構成を更新する必要があります。

1. Web アプリに戻ります。
1. **[設定]** セクションで、**[環境変数]** をクリックします。
1. 2 つの新しいアプリケーション設定を追加します。
    - 1 つ目のアプリ設定
        - **名前:** UseAppConfig
        - **値:** true
    - 2 つ目のアプリ設定
        - **名前:** AppConfigEndpoint
        - **値:** <App Configuration Endpoint から以前に保存またはコピーした値。 <https://appcs-NAME-REGION.azconfig.io> のような値です>**

1. **[適用]**、**[確認]** の順に選択し、設定が更新されるまで待ちます。
1. **[概要]** に移動して **[参照]** をクリックします
1. この手順では、App Configuration にデータが含まれていないため、Web サイトに変更はありません。 これは、次のタスクで行うことです。

#### タスク 4: 構成管理をテストする

1. Web サイトで、 **[ブランド]** ドロップダウン リストから **[Visual Studio]** を選び、矢印ボタン ( **>** ) をクリックします。
1. "THERE ARE NO RESULTS THAT MATCH YOUR SEARCH" (検索と一致する結果がありません) というメッセージが表示されます。** このラボの目標は、Web サイトのコードを更新したり、再デプロイしたりすることなく、この値を更新できるようにすることです。
1. このことを試すために、App Configuration に戻ります。
1. **[操作]** セクションで **[構成エクスプローラー]** を選びます。
1. **[作成] > [キー値]** の順にクリックして、次を追加します。
    - **キー:** eShopWeb:Settings:NoResultsMessage
    - **値**: <カスタム メッセージを入力してください>**
1. **[適用]** をクリックしてから Web サイトに戻り、ページを更新します。
1. これで、古い既定値ではなく、新しいメッセージが表示されます。

#### タスク 5: 機能フラグをテストする

引き続き機能マネージャーをテストしましょう。

1. このことを試すために、App Configuration に戻ります。
1. **[操作]** セクションで、 **[機能マネージャー]** を選びます。
1. **[作成]** をクリックし、次を追加します。
    - **機能フラグを有効にする:** オン
    - **機能フラグ名:** SalesWeekend
1. **[適用]** をクリックしてから Web サイトに戻り、ページを更新します。
1. 画像と "ALL T-SHIRTS ON SALE THIS WEEKEND" (今週末、すべての T シャツが特価に) というテキストが表示されます。
1. App Configuration でこの機能を無効にすると、画像が表示されなくなることがわかります。

   > [!IMPORTANT]
   > 不要な料金が発生しないように、Azure portal で作成されたリソースを必ず削除してください。 **eshoponweb-cd-webapp-code** パイプラインを必ず無効にしてください。そうしないと、削除したリソース グループと関連リソースが **eshoponweb-ci** の次回の実行後に再作成されます。

## 確認

このラボでは、構成を動的に有効にし、機能フラグを管理する方法を学びました。
