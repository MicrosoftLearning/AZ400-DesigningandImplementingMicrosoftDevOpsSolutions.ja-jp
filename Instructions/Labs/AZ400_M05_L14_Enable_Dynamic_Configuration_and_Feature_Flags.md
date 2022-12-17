---
lab:
  title: 動的構成と機能フラグを有効にする
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
---

# <a name="enable-dynamic-configuration-and-feature-flags"></a>動的構成と機能フラグを有効にする

# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-requirements"></a>ラボの要件

- このラボには、**Microsoft Edge** または [Azure DevOps 対応ブラウザー](https://learn.microsoft.com/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)が必要です。

- **Azure DevOps 組織を設定する:** このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization?view=azure-devops)に関するページの手順に従って作成してください。

- 既存の Azure サブスクリプションを識別するか、新しいものを作成します。

- Azure サブスクリプションで共同作成者または所有者のロールを持つ Microsoft アカウントまたは Azure AD アカウントを持っていることを確認します。 詳細については、[「Azure portal を使用して Azure ロールの割り当てを一覧表示する」](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal)および[「Azure Active Directory で管理者ロールを表示して割当てる」](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal)を参照してください。

## <a name="lab-overview"></a>ラボの概要

[Azure App Configuration](https://learn.microsoft.com/azure/azure-app-configuration/overview) は、アプリケーションの設定と機能フラグを一元管理するサービスを提供します。 近年のプログラム、特にクラウドで実行されるものには、分散されたコンポーネントが多数存在するのが一般的です。 これらのコンポーネント全体に構成設定を分散させることは、トラブルシューティングすることの難しいエラーがアプリケーションのデプロイ中に発生する原因となります。 App Configuration を使用すると、アプリケーションのすべての設定を 1 か所に格納して、そのアクセスをセキュリティで保護することができます。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- 動的な構成の有効化
- 機能フラグを管理する

## <a name="estimated-timing-60-minutes"></a>推定時間:60 分

## <a name="instructions"></a>Instructions

### <a name="exercise-0-configure-the-lab-prerequisites"></a>演習 0:ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、[eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) に基づくリポジトリを含む新しい Azure DevOps プロジェクトで構成されます。

#### <a name="task-1-skip-if-done-create-and-configure-the-team-project"></a>タスク 1: (完了している場合はスキップしてください) チーム プロジェクトを作成して構成する

このタスクでは、複数のラボで使用される **eShopOnWeb** Azure DevOps プロジェクトを作成します。

1.  ラボ コンピューターのブラウザー ウィンドウで、Azure DevOps 組織を開きます。 **[新しいプロジェクト]** をクリックします。 プロジェクトに **eShopOnWeb** という名前を付け、 **[作業項目プロセス]** ドロップダウンで **[スクラム]** を選びます。 **[作成]** をクリックします。

#### <a name="task-2-skip-if-done-import-eshoponweb-git-repository"></a>タスク 2: (完了している場合はスキップしてください) eShopOnWeb Git リポジトリをインポートする

このタスクでは、複数のラボで使用される eShopOnWeb Git リポジトリをインポートします。

1.  ラボ コンピューターのブラウザー ウィンドウで、Azure DevOps 組織と、前に作成した **eShopOnWeb** プロジェクトを開きます。 **[リポジトリ] > [ファイル]** 、 **[インポート]** をクリックします。 **[Git リポジトリをインポートする]** ウィンドウで、URL https://github.com/MicrosoftLearning/eShopOnWeb.git を貼り付けて、 **[インポート]** をクリックします。

1.  リポジトリは次のように編成されています。
    - **.ado** フォルダーには、Azure DevOps の YAML パイプラインが含まれています
    - **.devcontainer** フォルダーには、コンテナーを使って開発するためのセットアップが含まれています (VS Code でローカルに、または GitHub Codespaces で)
    - **.azure** フォルダーには、一部のラボ シナリオで使用される Bicep&ARM コードとしてのインフラストラクチャ テンプレートが含まれています。
    - **.github** フォルダーには、YAML GitHub ワークフローの定義が含まれています。
    - **src** フォルダーには、ラボ シナリオで使用される .NET 6 Web サイトが含まれています。

#### <a name="task-3-skip-if-done-set-main-branch-as-default-branch"></a>タスク 3: (完了している場合はスキップしてください) メイン ブランチを既定のブランチとして設定する

1. **[リポジトリ] > [ブランチ]** に移動します
1. **main** ブランチにカーソルを合わせ、列の右側にある省略記号をクリックします
1. **[既定のブランチとして設定]** をクリックします

### <a name="exercise-1-skip-if-done-import-and-run-cicd-pipelines"></a>演習 1: (完了している場合はスキップしてください) CI/CD パイプラインをインポートして実行する

この演習では、CI パイプラインをインポートして実行し、サービスの Azure サブスクリプションとの接続を構成し、CD パイプラインをインポートして実行します。

#### <a name="task-1-import-and-run-the-ci-pipeline"></a>タスク 1: CI パイプラインをインポートして実行する

まず、[eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml) という CI パイプラインをインポートします。

1. **[パイプライン] > [パイプライン]** に移動します

1. **[パイプラインを作成]** ボタンをクリックします

1. **[Azure Repos Git (Yaml)]** を選びます

1. **eShopOnWeb** リポジトリを選びます

1. **[既存の Azure Pipelines の YAML ファイル]** を選びます

1. **/.ado/eshoponweb-ci.yml** ファイルを選び、 **[続行]** をクリックします

1. **[実行]** ボタンをクリックしてパイプラインを実行します

1. パイプラインには、プロジェクト名に基づく名前が付けられます。 パイプラインを識別しやすくするために、**名前を変更**しましょう。 **[パイプライン] > [パイプライン]** に移動し、先ほど作成したパイプラインをクリックします。 省略記号と **[名前の変更]/[削除]** オプションをクリックします。 **eshoponweb-ci** という名前を付け、 **[保存]** をクリックします。

#### <a name="task-2-manage-the-service-connection"></a>タスク 2: サービス接続を管理する

Azure Pipelines から外部およびリモート サービスへの接続を作成し、ジョブのタスクを実行できます。

このタスクでは、Azure CLI を使ってサービス プリンシパルを作成します。これにより、Azure DevOps で次のことができるようになります。
- Azure サブスクリプションでリソースをデプロイする
- eShopOnWeb アプリケーションをデプロイする

> **注**:サービス プリンシパルが既にある場合は、次のタスクに直接進むことができます。

Azure Pipelines から Azure リソースをデプロイするには、サービス プリンシパルが必要です。

サービス プリンシパルは、パイプライン定義内から Azure サブスクリプションに接続するとき、またはプロジェクト設定ページから新しいサービス接続を作成するときに (自動オプション)、Azure パイプラインによって自動的に作成されます。 ポータルから、または Azure CLI を使用してサービス プリンシパルを手動で作成し、プロジェクト間で再利用することもできます。

1.  ラボのコンピューターで Web ブラウザーを起動し、[**Azure Portal**](https://portal.azure.com) に移動します。このラボで使用する Azure サブスクリプションで所有者ロールがあり、このサブスクリプションに関連のある Azure AD テナントでグローバル管理者ロールがあるユーザー アカウントを使ってサインインします。
1.  Azure portal で、ページ上部の検索テキストボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。
1.  **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、**[Bash]** を選択します。 

   >**注**: **Cloud Shell** を初めて起動し、[**ストレージがマウントされていません**] というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、**[ストレージの作成]** を選択します。 

1.  **Bash** プロンプトの **[Cloud Shell]** ペインで、次のコマンドを実行して、Azure サブスクリプション ID の属性の値を取得します。 

    ```sh
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > **注**:両方の値をテキスト ファイルにコピーします。 これらは、このラボの後半で必要になります。

1.  **Bash** プロンプトの **[Cloud Shell]** ペインで、次のコマンドを実行してサービス プリンシパルを作成します。

    ```sh
    az ad sp create-for-rbac --name sp-az400-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > **注**:このコマンドは JSON 出力を生成します。 出力をテキスト ファイルにコピーします。 このラボで後ほど必要になります。

1. 次に、ラボ コンピューターから Web ブラウザーを起動し、Azure DevOps **eShopOnWeb** プロジェクトに移動します。 **[プロジェクトの設定] > [サービス接続] ([パイプライン] の下)** 、 **[新しいサービス接続]** の順にクリックします。

1. **[新しいサービス接続]** ブレードで、 **[Azure Resource Manager]** と **[次へ]** を選択します (必要に応じて下にスクロールします)。

1. **[サービス プリンシパル (手動)]** を選択し、 **[次へ]** をクリックします。

1. 前の手順で収集した情報を使って、空のフィールドに入力します。
    - サブスクリプション ID と名前
    - サービス プリンシパル ID (または clientId)、Key (または Password)、TenantId。
    - **[サービス接続名]** に「**azure subs**」と入力します。 この名前は、Azure サブスクリプションと通信するために Azure DevOps サービス接続が必要になるときに、YAML パイプラインで参照されます。

1. **[確認して保存]** をクリックします。

#### <a name="task-3-import-and-run-the-cd-pipeline"></a>タスク 3: CD パイプラインをインポートして実行する

1. **[パイプライン] > [パイプライン]** に移動します

1. **[新しいパイプライン]** ボタンをクリックします

1. **[Azure Repos Git (Yaml)]** を選びます

1. **eShopOnWeb** リポジトリを選びます

1. **[既存の Azure Pipelines の YAML ファイル]** を選びます

1. **/.ado/eshoponweb-ci.yml** ファイルを選び、 **[続行]** をクリックします

1. YAML パイプライン定義で、次のようにカスタマイズします。
- **YOUR-SUBSCRIPTION-ID** を使用する Azure サブスクリプション ID にします。
- **az400eshop-NAME** の NAME を置き換えて、グローバルに一意にします。
- **AZ400-EWebShop-NAME** をラボで前に定義したリソース グループ名にします。

1. **[保存および実行]** をクリックし、パイプラインが正常に実行されるまで待ちます。

    > **注**: デプロイが完了するまでに数分かかる場合があります。

    CD の定義は以下のタスクで構成されます。
    - **リソース**: CI パイプラインの完了に基づいて自動的にトリガーされるように準備されています。 また、bicep ファイルのリポジトリもダウンロードします。
    - **AzureResourceManagerTemplateDeployment**: bicep テンプレートを使用して Azure Web Apps をデプロイします。

1. パイプラインには、プロジェクト名に基づく名前が付けられます。 パイプラインを識別しやすくするために、**名前を変更**しましょう。 **[パイプライン] > [パイプライン]** に移動し、先ほど作成したパイプラインをクリックします。 省略記号と **[名前の変更]/[削除]** オプションをクリックします。 **eshoponweb-cd-webapp-code** という名前を付け、 **[保存]** をクリックします。

### <a name="exercise-2-manage-azure-app-configuration"></a>演習 2: Azure App Configuration を管理する

この演習では、Azure で App Configuration リソースを作成し、マネージド ID を有効にしてから、ソリューション全体をテストします。

>注: この演習にコーディングのスキルは必要ありません。 この Web サイトのコードは、Azure App Configuration の機能を既に実装しています。
アプリケーションにこの機能を実装する方法については、「[ASP.NET Core アプリで動的な構成を使用する](https://learn.microsoft.com/azure/azure-app-configuration/enable-dynamic-configuration-aspnet-core)」と「[Azure App Configuration で機能フラグを管理する](https://learn.microsoft.com/azure/azure-app-configuration/manage-feature-flags)」のチュートリアルを参照してください。

#### <a name="task-1-create-the-app-configuration-resource"></a>タスク 1: App Configuration リソースを作成する

1. Azure Portal で **App Configuration** サービスを検索します
1. **[Create app configuration] (アプリ構成の作成)** をクリックして、次を選びます。
    - 自分の Azure サブスクリプション
    - 以前に作成したリソース グループ (名前はおそらく **AZ400-EWebShop-NAME** です)
    - 場所
    - たとえば、**appcs-NAME-REGION** のような一意の値です
    - **Free** の価格レベルを選びます
1. **[確認および作成]** 、 **[作成]** の順にクリックします
1. App Configuration サービスを作成したら、 **[概要]** に移動し、 **[エンドポイント]** の値をコピーまたは保存します。

#### <a name="task-2-enable-managed-identity"></a>タスク 2: マネージド ID を有効にする

1. パイプラインを使ってデプロイされた Web アプリに移動します (名前はおそらく **az400-webapp-NAME** です)。
1. **[設定]** セクションで **[ID]** をクリックし、 **[システム割り当て]** セクションで状態を **[オン]** に切り替え、 **[保存] > [はい]** をクリックして、操作が完了するまで数秒待ちます。
1. App Configuration サービスに戻り、 **[アクセス制御]** 、 **[ロールの割り当てを追加]** の順にクリックします。
1. **[ロール]** セクションで **[App Configuration データ閲覧者]** を選びます
1. **[メンバー]** セクションで **[マネージド ID]** を確認し、Web アプリのマネージド ID (同じ名前である必要があります) を選びます。
1. **[Review and assign] (確認と割り当て)** をクリックします。

#### <a name="task-3-configure-the-web-app"></a>タスク 3: Web アプリを構成する

Web サイトが App Configuration にアクセスしていることを確認するには、その構成を更新する必要があります。
1. Web アプリに戻ります。
1. **[設定]** セクションで **[構成]** をクリックします。
1. 2 つの新しいアプリケーション設定を追加します。
    - 1 つ目のアプリ設定
        - **名前:** UseAppConfig
        - **値:** true
    - 2 つ目のアプリ設定
        - **名前:** AppConfigEndpoint
        - **値:** <App Configuration Endpoint から以前に保存またはコピーした値。 https://appcs-NAME-REGION.azconfig.io のような値です>**
1. **[OK]** 、 **[保存]** の順にクリックし、設定が更新されるまで待ちます。
1. **[概要]** に移動して **[参照]** をクリックします
1. この手順では、App Configuration にデータが含まれていないため、Web サイトに変更はありません。 これは、次のタスクで行うことです。

#### <a name="task-4-test-the-configuration-management"></a>タスク 4: 構成管理をテストする

1. Web サイトで、 **[ブランド]** ドロップダウン リストから **[Visual Studio]** を選び、矢印ボタン ( **>** ) をクリックします。
1. "THERE ARE NO RESULTS THAT MATCH YOUR SEARCH" (検索と一致する結果がありません) というメッセージが表示されます。**
このラボの目標は、Web サイトのコードを更新したり、再デプロイしたりすることなく、この値を更新できるようにすることです。
1. このことを試すために、App Configuration に戻ります。
1. **[操作]** セクションで **[構成エクスプローラー]** を選びます。
1. **[作成] > [キー値]** の順にクリックして、次を追加します。
    - **キー:** eShopWeb:Settings:NoResultsMessage
    - **値**: <カスタム メッセージを入力してください>**
1. **[適用]** をクリックしてから Web サイトに戻り、ページを更新します。
1. これで、古い既定値ではなく、新しいメッセージが表示されます。

お疲れさまでした。 このタスクでは、Azure App Configuration で **Configuration エクスプローラー**をテストしました。

#### <a name="task-5-test-the-feature-flag"></a>タスク 5: 機能フラグをテストする

引き続き機能マネージャーをテストしましょう。
1. このことを試すために、App Configuration に戻ります。
1. **[操作]** セクションで、 **[機能マネージャー]** を選びます。
1. **[作成]** をクリックし、次を追加します。
    - **機能フラグを有効にする:** オン
    - **機能フラグ名:** SalesWeekend
1. **[適用]** をクリックしてから Web サイトに戻り、ページを更新します。
1. 画像と "ALL T-SHIRTS ON SALE THIS WEEKEND" (今週末、すべての T シャツが特価に) というテキストが表示されます。
1. App Configuration でこの機能を無効にすると、画像が表示されなくなることがわかります。

お疲れさまでした。 このタスクでは、Azure App Configuration で**機能マネージャー**をテストしました。

### <a name="exercise-3-remove-the-azure-lab-resources"></a>演習 3:Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。

>**注**:新規に作成し、使用しなくなったすべての Azure リソースを削除することを忘れないでください。 使用していないリソースを削除することで、予期しない料金が発生しなくなります。

#### <a name="task-1-remove-the-azure-lab-resources"></a>タスク 1:Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。

1. Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1. 次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'AZ400-EWebShop-')].name" --output tsv
    ```

1. 次のコマンドを実行して、このモジュールのラボ全体を通して作成したすべてのリソース グループを削除します。

    ```sh
    az group list --query "[?starts_with(name,'AZ400-EWebShop-')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注**:コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## <a name="review"></a>確認

このラボでは、構成を動的に有効にし、機能フラグを管理する方法を学びました。
