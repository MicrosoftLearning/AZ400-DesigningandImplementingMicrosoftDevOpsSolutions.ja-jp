---
lab:
  title: ラボ 12:Azure Key Vault と Azure DevOps の統合
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
ms.openlocfilehash: ee482422f21a674e4a91b7cd7af048fbd2bfbfbb
ms.sourcegitcommit: f72fcf5ee578f465b3495f3cf789b06c530e88a4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2022
ms.locfileid: "139262605"
---
# <a name="lab-12-integrating-azure-key-vault-with-azure-devops"></a>ラボ 12:Azure Key Vault と Azure DevOps の統合
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-overview"></a>ラボの概要

Azure Key Vault は、キー、パスワード、証明書などの機密データの安全な保管と管理を行います。 Azure Key Vault には、ハードウェア セキュリティ モジュールのサポートに加えて、さまざまな暗号化アルゴリズムとキーの長さが含まれています。 Azure Key Vault を使用することで、開発者がよく犯す間違いであるソース コードを介して機密データを開示する可能性を最小限に抑えることができます。 Azure Key Vault にアクセスするには、コンテンツに対するきめ細かいアクセス許可をサポートする適切な認証と承認が必要です。

このラボでは、次の手順を使用して、Azure KeyVault を Azure DevOps パイプラインと統合する方法を説明します。

- MySQL サーバーのパスワードをシークレットとして保存する Azure Key Vault を作成します。
- Azure Key Vault 内のシークレットへのアクセスを提供する Azure サービス プリンシパルを作成します。
- サービス プリンシパルがシークレットを読み取れるようにする権限を構成します。
- Azure Key Vault からパスワードを取得し、それを後続のタスクに渡すようにパイプラインを構成します。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

-   Azure Active Directory (Azure AD) サービス プリンシパルを作成します。
-   Azure キー コンテナーを作成します。 
-   Azure DevOps パイプラインを介してプル要求を追跡する。

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

#### <a name="prepare-an-azure-subscription"></a>Azure サブスクリプションの準備

-   既存の Azure サブスクリプションを識別するか、新しいものを作成します。
-   Azure サブスクリプションでは所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントではグローバル管理者のロールで Microsoft アカウントまたは Azure AD アカウントがあることを確認します。 詳細については、[「Azure portal を使用して Azure ロールの割り当てを一覧表示する」](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal)および[「Azure Active Directory で管理者ロールを表示して割当てる」](https://docs.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal#view-my-roles)を参照してください。

#### <a name="set-up-an-azure-devops-organization"></a>Azure DevOps 組織を設定する

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)で利用できる手順に従って作成してください。

### <a name="exercise-0-configure-the-lab-prerequisites"></a>演習 0:ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。Azure DevOps Demo Generator テンプレートに基づいて事前構成された Parts Unlimited チームプロジェクトで構成されます。

#### <a name="task-1-configure-the-team-project"></a>タスク 1:チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用して、**Azure Key Vault** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。 このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**:サイトの詳細については、 https://docs.microsoft.com/en-us/azure/devops/demo-gen を参照してください。

1.  「**サインイン**」をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。
1.  必要な場合は、「**Azure DevOps Demo Generator**」ページで「**承諾する**」をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。
1.  **[新しいプロジェクトの作成]** ページの **[新しいプロジェクト名]** テキストボックスに、「**Azure KeyVault と Azure DevOps の統合**」と入力し、 **[組織の選択]** ドロップダウンリストで、Azure DevOps 組織を選択して、 **[テンプレートの選択]** をクリックします。
1.  **[テンプレートの選択]** ページのヘッダー メニューで、 **[DevOps ラボ]** をクリックし、テンプレートのリストで **[Azure Key Vault テンプレート]** をクリックしてから、 **[テンプレートの選択]** をクリックします。
1.  **[新しいプロジェクトの作成]** ページに戻り、 **[ARM 出力]** ラベルの下のチェックボックスを選択して、 **[プロジェクトの作成]** をクリックします。

    > **注**:プロセスが完了するまでお待ちください。 これには 2 分ほどかかります。 プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

### <a name="exercise-1-integrate-azure-key-vault-with-azure-devops"></a>演習 1:Azure Key Vault を Azure DevOps と統合する

- Azure Key Vault 内のシークレットへのアクセスを提供する Azure サービス プリンシパルを作成します。
- MySQL サーバーのパスワードをシークレットとして保存する AzureKey Vault を作成します。
- サービス プリンシパルがシークレットを読み取れるようにする権限を構成します。
- Azure Key Vault からパスワードを取得し、それを後続のタスクに渡すようにパイプラインを構成します。

#### <a name="task-1-create-a-service-principal"></a>タスク 1:サービス プリンシパルを作成する 

このタスクでは、Azure CLI を使用してサービス プリンシパルを作成します。 

> **注**:サービス プリンシパルが既にある場合は、次のタスクに直接進むことができます。

Azure Pipelines からAzure リソースにアプリをデプロイするには、サービス プリンシパルが必要です。 パイプラインでシークレットを取得するため、Azure Key Vault を作成するときにサービスにアクセス許可を付与する必要があります。 

サービス プリンシパルは、パイプライン定義内から Azure サブスクリプションに接続するとき、またはプロジェクト設定ページから新しいサービス接続を作成するときに、Azure Pipeline によって自動的に作成されます。 ポータルから、または Azure CLI を使用してサービス プリンシパルを手動で作成し、プロジェクト間で再利用することもできます。 事前定義された権限のセットが必要な場合は、既存のサービス プリンシパルを使用することをお勧めします。

1.  ラボのコンピューターで Web ブラウザーを起動し、[**Azure portal**](https://portal.azure.com) に移動します。このラボで使用する Azure サブスクリプションで、所有者のロールがあり、このサブスクリプションに関連付けられている Azure AD テナントでグローバル管理者のロールがあるユーザー アカウントを使ってサインインします。
1.  Azure portal で、ページ上部の検索テキスト ボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。 
1.  **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、**[Bash]** を選択します。 

   >**注**:**Cloud Shell** を初めて起動し、「**ストレージがマウントされていません**」というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、 **[ストレージの作成]** を選択します。 

1.  **Bash** プロンプトの **Cloud Shell** ペインで、次のコマンドを実行してサービス プリンシパルを作成します (`<service-principal-name>` を文字と数字で構成される一意の文字列に置き換えます)。

    ```
    az ad sp create-for-rbac --name <service-principal-name> --role Contributor
    ```

    > **注**:このコマンドは JSON 出力を生成します。 出力をテキスト ファイルにコピーします。 このラボで後ほど必要になります。

1.  **Bash** プロンプトの **[Cloud Shell]** ペインで、次のコマンドを実行して、Azure サブスクリプション ID とサブスクリプション名の属性の値を取得します。 

    ```
    az account show --query id --output tsv
    az account show --query name --output tsv
    ```

    > **注**:両方の値をテキスト ファイルにコピーします。 これらは、このラボの後半で必要になります。


#### <a name="task-2-create-an-azure-key-vault"></a>タスク 2:Azure Key Vault を作成する

このタスクでは、Azure portal を使用して Azure Key Vault を作成します。

このラボ シナリオでは、MySQL データベースに接続するアプリがあります。 MySQL データベースのパスワードをシークレットとして Key Vault に保存します。

1.  Azure portal の **[リソース、サービス、ドキュメントの検索]** テキスト ボックスに「**Key Vault**」と入力し、**Enter** キーを押します。 
1.  **Key Vault** ブレードで、 **[+ 作成]** をクリックします。 
1.  **[キー コンテナーの作成]** ブレードの **[基本]** タブで、次の設定を指定し、 **[次へ: アクセス ポリシー]** をクリックします。

    | 設定 | 値 |
    | --- | --- |
    | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
    | リソース グループ | 新しいリソース グループ **az400m07l01-RG** の名前 |
    | キー コンテナー名 | 任意の一意の有効な名前 |
    | リージョン | ラボ環境の場所に近い Azure リージョン |
    | Pricing tier | **Standard** |
    | 削除されたボールトを保持する日数 | **7** |
    | 消去保護 | **消去保護を無効にする** |

1.  **[キーボールトの作成]** ブレードの **[アクセス ポリシー]** タブで、 **[+ アクセスポリシーの追加]** をクリックして、新しいポリシーを設定します。

    > **注**:許可されたアプリケーションとユーザーのみを許可することにより、Key Vault のアクセスを保護する必要があります。 ボールトからデータにアクセスするには、パイプラインでの認証に使用するサービス プリンシパルに読み取り (取得) アクセス許可を提供する必要があります。 

1.  **[アクセスポリシーの追加]** ブレードで、 **[プリンシパルの選択]** ラベルのすぐ下にある **[選択されていない]** リンクをクリックします。 
1.  **プリンシパル** ブレードで、前の演習で作成したセキュリティ プリンシパルを検索して選択し、 **[選択]** をクリックします。 

    > **注**:プリンシパルの名前または ID で検索できます。

1.  **[アクセス ポリシーの追加]** ブレードに戻り、 **[シークレット アクセス許可]** ドロップダウン リストで、 **[取得]** および **[リスト]** アクセス許可の横にあるチェックボックスを選択し、 **[追加]** をクリックします。 
1.  **[Key Vault の作成]** ブレードの **[アクセス ポリシー]** タブに戻り、 **[レビュー+作成]** をクリックし、 **[レビュー+作成]** ブレードで **[作成]** をクリックします。 

    > **注**:Azure Key Vault がプロビジョニングされるのを待ちます。 これにかかる時間は 1 分未満です。

1.  **[デプロイが完了しました]** ブレードで、 **[リソースに移動]** をクリックします。
1.  Azure Key Vault ブレードのブレードの左側にある垂直メニューの **[設定]** セクションで、 **[シークレット]** をクリックします。 
1.  **[シークレット]** ブレードで、 **[生成/インポート]** をクリックします。
1.  **[シークレット ブレードの作成]** で、次の設定を指定し、 **[作成]** をクリックします (他の設定はデフォルト値のままにします)。

    | 設定 | 値 |
    | --- | --- |
    | Upload options | **[手動]** |
    | Name | **sqldbpassword** |
    | 値 | 任意の有効な MySQL のパスワード値 |


#### <a name="task-3-check-the-azure-pipeline"></a>タスク 3:Azure Pipelines を確認する

このタスクでは、Azure Key Vault からシークレットを取得するように Azure Pipelines を構成します。

1.  ラボ コンピューターで、Web ブラウザーを起動し、前の演習で作成した **Azure KeyVault と Azure DevOps の統合** プロジェクトに移動します。
1.  Azure DevOps ポータルの垂直ナビゲーション ペインで、 **[パイプライン]** を選択し、 **[パイプライン]** ペインが表示されていることを確認します。
1.  **[パイプライン]** ペインで、**SmartHotel-CouponManagement-CI** パイプラインを表すエントリをクリックします。 **[Edit]\(編集\)** をクリックします。
2.   パイプラインの定義で、 **[パイプライン]**  >  **[エージェントの指定]** が **ubuntu 18.04** であることを確認します。 **[保存してキューに登録]**  >  **[キュー]**  >  **[実行]** をクリックして、ビルドをトリガーします。
3.  Azure DevOps ポータルの垂直ナビゲーション ペインの **[パイプライン]** セクションで、 **[リリース]** を選択します。 
4.  **SmartHotel-CouponManagement-CD** ペインで、右上隅にある **[編集]** をクリックします。
5.  **[すべてのパイプライン] > [SmartHotel-CouponManagement-CD]** ペインで、 **[タスク]** タブを選択し、ドロップダウン メニューで **[開発]** を選択します。

    > **注**:**開発** ステージのリリース定義には、**Azure Key Vault** タスクがあります。 このタスクは、Azure KeyVault から *シークレット* をダウンロードします。 ラボで以前に作成したサブスクリプションと Azure Key Vault リソースを指定する必要があります。

    > **注**:Azure にデプロイするには、パイプラインを承認する必要があります。 Azure Pipelines は、新しいサービス プリンシパルとのサービス接続を自動的に作成できます。**ただし、ここでは以前に作成したものを使います**。これは、シークレットの読み取りが認可されているからです。 

1.  **[エージェントで実行]** を選び、 **[エージェント プール]** フィールドを **[Azure Pipelines]** に、[エージェントの指定] を **ubuntu 18.04** を変更します。
1.  **Azure Key Vault** タスクを選択し、右側の **Azure Key Vault** タスクのプロパティで、**Azure サブスクリプション** ラベルの横にある **[管理]** をクリックします。 これにより、Azure DevOps ポータルの **[サービス接続]** ペインを表示する別のブラウザー タブが開きます。
1.  **[サービス接続]** ペインで、 **[新しいサービス接続]** をクリックします。 
1.  **[新しいサービス接続]** ウィンドウで、 **[Azure Resource Manager]** オプションを選択し、 **[次へ]** をクリックして、 **[サービス プリンシパル (手動)]** を選択し、もう一度 **[次へ]** をクリックします。
1.  **[新しいサービス接続]** ペインで、Azure CLI を使用してサービス プリンシパルを作成した後、この演習の最初のタスクでテキスト ファイルにコピーした情報を使用して、次の設定を指定します。

    - サブスクリプション ID: `az account show --query id --output tsv` を実行して取得した値
    - サブスクリプション名: `az account show --query name --output tsv` を実行して取得した値
    - サービス プリンシパル ID: `az ad sp create-for-rbac ` を実行して生成された出力の **appId** というラベルの付いた値。
    - サービス プリンシパル キー: `az ad sp create-for-rbac ` を実行して生成された出力の **password** というラベルの付いた値。
    - TenantId: `az ad sp create-for-rbac ` を実行して生成された出力の **tenant** というラベルの付いた値。

1.  **[新しいサービスの接続]** ペインで、 **[確認]** をクリックして、指定した情報が有効かどうかを確認します。 
1.  「**検証に成功しました**」という応答を受け取ったら、 **[サービス接続名]** テキストボックスに「**kv-service-connection**」と入力し、 **[検証して保存]** をクリックします。
1.  パイプラインの定義と **Azure Key Vault** タスクを表示している Web ブラウザー タブに戻ります。
1.  **Azure Key Vault** タスクを選択した状態で、 **[Azure Key Vault]** ペインの **[更新]** ボタンをクリックし、 **[Azure サブスクリプション]** ドロップダウン リストで **kv-service-connection** エントリを選択し、 **[Key Vault]** ドロップダウン リストで、最初のタスクで作成した Azure Key Vault を表すエントリを選択し、 **[シークレット フィルター]** テキストボックスに「**sqldbpassword**」と入力します。 最後に、 **[出力変数]** セクションを展開し、 **[参照名]** テキストボックスに「**sqldbpassword**」と入力します。 

    > **注**:実行時に、Azure Pipelines はシークレットの最新の値をフェッチし、それをタスク変数 **$(sqldbpassword)** として設定します。 タスクは、その変数を参照することにより、後続のタスクによって消費される可能性があります。  

1.  これを確認するには、次のタスクである **Azure Deployment** を選択します。このタスクは、ARM テンプレートをデプロイし、 **[テンプレート パラメーターのオーバーライド]** テキスト ボックスの内容を確認します。 

    ```
    -webAppName $(webappName) -mySQLAdminLoginName "azureuser" -mySQLAdminLoginPassword $(sqldbpassword)
    ```

    > **注**:**テンプレート パラメーターのオーバーライド** のコンテンツは、**sqldbpassword** 変数を参照して mySQL 管理者パスワードを設定します。 これにより、キーボールトで指定したパスワードを使用して、ARM テンプレートで定義された MySQL データベースがプロビジョニングされます。 

1.  パイプライン定義を完了するには、サブスクリプション (プロジェクトで新しいサブスクリプションを使用する場合は、 **[承認]** をクリックします) とタスクの場所を指定します。 パイプライン **Azure App Service Deploy** の最後のタスクについても同じことを **繰り返します** (ドロップダウンの **[利用可能な Azure サービス接続]** セクションからサブスクリプションを選択します)。 

    > **注**:[Azure サブスクリプション] ドロップダウン リストに、Azure への接続が既に承認されているサブスクリプションで **使用可能な Azure サービス接続** が表示されます。 (**使用可能な Azure サブスクリプション** リストから) 承認済みサブスクリプションを再度選択して **承認** しようとすると、プロセスは失敗します。

1.  **[変数]** タブの **resourcegroup** 変数をプレーンテキストに変更し (ロックをクリックします)、値フィールドに **az400m07l01-RG** と入力します。
1.  最後に **[保存]** して **[新しいリリースの作成]**  >  **[作成]** をクリックし (既定のままにします)、デプロイを始めます。

1. パイプラインが正常に実行されていることを確認し、終了したら、Azure portal でリソース グループ **az400m07l01-RG** を開いて、作成されたリソースを確認します。 **App Service** を開いて参照し **([概要] -> [参照])** 、公開された Web サイトを表示します。

### <a name="exercise-2-remove-the-azure-lab-resources"></a>演習 2:Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

>**注**:新規に作成し、使用しなくなったすべての Azure リソースを削除することを忘れないでください。 使用していないリソースを削除することで、予期しない料金が発生しなくなります。

#### <a name="task-1-remove-the-azure-lab-resources"></a>タスク 1:Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'az400m07l01-RG')].name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体を通して作成したすべてのリソース グループを削除します。

    ```sh
    az group list --query "[?starts_with(name,'az400m07l01-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注**:コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

#### <a name="review"></a>確認

このラボでは、次の手順を使用して、Azure KeyVault を Azure DevOps パイプラインと統合しました。

- MySQL サーバーのパスワードをシークレットとして格納する Azure Key Vaultを作成しました。
- Azure Key Vault 内のシークレットへのアクセスを提供する Azure サービス プリンシパルを作成しました。
- サービス プリンシパルがシークレットを読み取れるようにアクセス許可を構成しました。
- Azure Key Vault からパスワードを取得し、それを後続のタスクに渡すようにパイプラインを構成しました。
