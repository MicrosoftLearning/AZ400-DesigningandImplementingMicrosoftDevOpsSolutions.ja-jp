---
lab:
    title: 'ラボ 16: Azure Kubernetes サービスへのマルチコンテナー アプリケーションのデプロイ'
    module: 'モジュール 16: Kubernetes サービス インフラストラクチャの作成と管理'
---

# ラボ 16: Azure Kubernetes Services へのマルチコンテナー アプリケーションのデプロイ
# 受講生用ラボ マニュアル

## ラボの概要

[**Azure Kubernetes Service (AKS)**](https://azure.microsoft.com/ja-jp/services/kubernetes-service/) は、Azure で Kubernetes を使用する最も簡単な方法です。**Azure Kubernetes Service (AKS)** を使用すると、ホストされている Kubernetes 環境を管理できます。これによって、コンテナー オーケストレーションの知識がなくてもコンテナー化されたアプリケーションを簡単にデプロイして管理できるようになります。また、コンテナー化されたワークロードの機敏性、スケーラビリティ、可用性が向上します。Azure DevOps は、継続的なビルドおよびデプロイ機能により、AKS 操作をさらに簡素化します。 

このラボでは、Azure DevOps を使用して、コンテナー化された ASP.NET Core Web アプリケーション **MyHealthClinic** (MHC) を AKS クラスターにデプロイします。 

## 目標

このラボを完了すると、次のことができるようになります。

- Azure DevOps Demo Generator ツールを使用して、.NET Core アプリケーションで Azure DevOps チーム プロジェクトを作成する
- Azure CLI を使用して、Azure コンテナー レジストリ (ACR)、AKS クラスター、Azure SQL Database を作成する
- Azure DevOps を使用して、コンテナー化されたアプリケーションとデータベースのデプロイを構成する
- Azure DevOps パイプラインを使用して、コンテナー化されたアプリケーションを自動的にデプロイするようビルドする

## ラボの所要時間

-   推定時間: **60 分**

## 手順

### 開始する前に

#### ラボの仮想マシンにログインする

必ず次の資格情報を使用して Windows 10 コンピューターにサインインしてください。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### このラボで必要なアプリケーションのレビュー

このラボで使用するアプリケーションを特定:
    
-   Microsoft Edge

#### Azure DevOps 組織を設定します。 

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/ja-jp/azure/devops/organizations/accounts/create-organization?view=azure-devops) で利用できる手順に従って作成してください。

#### Azure サブスクリプションの準備

-   既存の Azure サブスクリプションを識別するか、新しいものを作成します。
-   Azure サブスクリプションでは所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントではグローバル管理者のロールで Microsoft アカウントまたは Azure AD アカウントがあることを確認します。詳細については、[Azure portal を使用して Azure ロールの割り当てを一覧表示する](https://docs.microsoft.com/ja-jp/azure/role-based-access-control/role-assignments-list-portal) および [Azure Active Directory で管理者ロールを表示して割当てる](https://docs.microsoft.com/ja-jp/azure/active-directory/roles/manage-roles-portal#view-my-roles) を参照してください。

### 演習 0: ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、Azure DevOps Demo Generator テンプレートに基づいて事前に構成されたチーム プロジェクトで構成されます。 

#### タスク 1: チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用し、**Azure Kubernetes Service** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**: サイトの詳細については、[https://docs.microsoft.com/ja-jp/azure/devops/demo-gen](https://docs.microsoft.com/ja-jp/azure/devops/demo-gen) をご覧ください。

1.  「**サインイン**」をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。
1.  必要な場合は、「**Azure DevOps Demo Generator**」ページで「**承諾する**」をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。
1.  「**新しいプロジェクトの作成**」ページで「**新しいプロジェクト名**」テキストボックスに「**マルチコンテナー アプリケーションを AKS にデプロイ**」と入力します。「**組織の選択**」ドロップダウン リストで Azure DevOps 組織を選択し、「**テンプレートの選択**」をクリックします。
1.  テンプレートのリストのツールバーで「**DevOps ラボ**」をクリックし、「**Azure Kubernetes Service**」テンプレートを選択して「**テンプレートの選択**」をクリックします。
1.  再び「**新しいプロジェクトの作成**」ページで、欠落している拡張機能をインストールするよう指示されたら「**トークンの置換**」と「**Kubernetes 拡張機能**」の下にあるチェックボックスを選択し、「**プロジェクトの作成**」をクリックします。

    > **注**: プロセスが完了するまでお待ちください。通常は 2 分ほどかかります。プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

### 演習 1: Azure DevOps を使用して、コンテナー化された ASP.NET Core Web アプリケーションを AKS クラスターにデプロイする

この演習では、Azure DevOps を使用して、コンテナー化された ASP.NET Core Web アプリケーションを AKS クラスターにデプロイします。

#### タスク 1: ラボの Azure リソースをデプロイする

このタスクでは、Azure CLI を使用して、このラボで必要な以下の Azure リソースのデプロイを行います:

| Azure リソース | 説明 |
| --------------- | ----------- |
| Azure Container Registry | Docker イメージのプライベート ストとして機能します |
| AKS | Docker イメージを実行しているコンテナーのオーケストレーターとして機能します |
| Azure SQL Database | AKS で実行中のコンテナー化されたワークロードの固定ストアを提供します |

1.  ラボのコンピューターから Web ブラウザーを起動し、[**Azure Portal**](https://portal.azure.com) に移動します。このラボで使用している Azure サブスクリプションで少なくとも共同作成者のロールがあるユーザー アカウントを使ってサインインします。
1.  Azure portal のツールバーで、検索テキスト ボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。 
1.  **Bash** または **PowerShell** のいずれかを選択するためのプロンプトが表示されたら、**「Bash」** を選択します。 

    >**注**: **Cloud Shell** を初めて起動し、「**ストレージがマウントされていません**」というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、「**ストレージの作成**」を選択します。 

1.  Cloud Shell ペインで **Bash** セッションから以下を実行し、このラボで使用する Azure リージョンで利用可能な Kubernetes の最新バージョンを特定します (**`<Azure_region>` プレースホルダー**は、このラボでリソースをデプロイしたい Azure リージョンの名前に置き換えます):

    ```bash
    LOCATION='<Azure_region>'
    ```

    > **注**: 次のコマンド `<Azure_region>` : `az account list-locations -o table` を実行すると可能な場所を見つけることができます。**Name** プロパティにスペースを入れずに値を使用します。

    ```bash
    VERSION=$(az aks get-versions --location $LOCATION --query 'orchestrators[-1].orchestratorVersion' --output tsv); echo $VERSION
    ```

1.  Cloud Shell ペインで **Bash** セッションから以下を実行し、AKS デプロイをホストするリソース グループを作成します:

    ```bash
    RGNAME=az400m16l01a-RG
    az group create --name $RGNAME --location $LOCATION
    ```

1.  Cloud Shell ペインの **Bash** セッションから以下を実行し、利用可能な最新バージョンを使用して AKS クラスターを作成します:
    
    ```bash
    AKSNAME='az400m16aks'$RANDOM$RANDOM
    az aks create --location $LOCATION --resource-group $RGNAME --name $AKSNAME --enable-addons monitoring --kubernetes-version $VERSION --generate-ssh-keys
    ```
    
    > **注**: 次のタスクに進む前に、デプロイが完了するのを待ちます。AKS のデプロイには約 5 分間かかります。 

1.  Cloud Shell ペインの **Bash** セッションから以下を実行し、このラボで使用する Azure SQL データベースをホストする論理サーバーを作成します:
  
    ```bash
    SQLNAME='az400m16sql'$RANDOM$RANDOM
    az sql server create --location $LOCATION --resource-group $RGNAME --name $SQLNAME --admin-user sqladmin --admin-password P2ssw0rd1234
    ```

1.  Cloud Shell ペインで **Bash** セッションから以下を実行し、新しくプロビジョニングされた論理サーバーに Azure からアクセスできるようにします:

    ```bash
    az sql server firewall-rule create --resource-group $RGNAME --server $SQLNAME --name allowAzure --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
    ```

1.  Cloud Shell ペインの **Bash** セッションから以下を実行し、このラボで使用する Azure SQL データベースを作成します:


    ```bash
    az sql db create --resource-group $RGNAME --server $SQLNAME --name mhcdb --service-objective S0 --no-wait
    ```

1.  Cloud Shell ペインの **Bash** セッションから以下を実行し、このラボで使用する Azure Container レジストリを作成します:

    ```bash
    ACRNAME='az400m16acr'$RANDOM$RANDOM
    az acr create --location $LOCATION --resource-group $RGNAME --name $ACRNAME --sku Standard
    ```

1.  Cloud Shell ペインの **Bash** セッションから以下を実行し、AKS で生成されたマネージド ID が新しく作成された ACR にアクセスできるようにします:

    ```bash
    # Retrieve the id of the service principal configured for AKS
    CLIENT_ID=$(az aks show --resource-group $RGNAME --name $AKSNAME --query "identityProfile.kubeletidentity.clientId" --output tsv)

    # Retrieve the ACR registry resource id
    ACR_ID=$(az acr show --name $ACRNAME --resource-group $RGNAME --query "id" --output tsv)

    # Create role assignment
    az role assignment create --assignee $CLIENT_ID --role acrpull --scope $ACR_ID
    ```

    > **注**: この課題の詳細については、[Azure Container Registry を使用した Azure Kubernetes Service からの承認](https://docs.microsoft.com/ja-jp/azure/container-registry/container-registry-auth-aks) を参照してください

1.  Cloud Shell ペインの **Bash** セッションから以下を実行し、このタスクで以前に作成した Azure SQL データベースをホストする論理サーバーの名前を表示します:

    ```bash
    echo $(az sql server list --resource-group $RGNAME --query '[].name' --output tsv)'.database.windows.net'
    ```

1.  Cloud Shell ペインの **Bash** セッションから以下を実行し、このタスクで先ほど作成した Azure Container レジストリのログイン サーバーの名前を表示します:

    ```bash
    az acr show --name $ACRNAME --resource-group $RGNAME --query "loginServer" --output tsv
    ```

    > **注**: 両方の値を記録します。これは、次のタスクで必要になります。

1.  Cloud Shell ペインを閉じます。

#### タスク 2: ビルド パイプラインとリリース パイプラインを構成する

このタスクでは、Azure リソースをマッピングすることにより、このラボで先ほど生成した Azure DevOps プロジェクトでビルド パイプラインとリリース パイプラインを構成します (AKS クラスターと、ビルドおよびリリース定義への Azure Container レジストリが含まれます)。

1.  ラボのコンピューターで、Azure DevOps ポータルが表示され、「**AKS へのマルチコンテナー アプリケーションのデプロイ**」プロジェクトが開いている Web ブラウザー ウィンドウに切り替え、Azure DevOps ポータルの一番左側にある垂直メニュー バーで「**リポジトリ**」をクリックします。

    >**注**: まず、Docker イメージへの参照を変更します。

1.  **AKS** リポジトリ ペインのファイルのリストで、**docker-compose.ci.build.yml** を選択します。
1.  **docker-compose.ci.build.yml** ペインで、**「編集」** をクリックして、Docker イメージを参照する **5** 行目を `image: az400mp/aspnetcore-build:1.0-2.0` で置き換え、**「コミット」** を選択し、確認を求められたら、もう一度 **「コミット」** をクリックします。 
1.  **AKS** リポジトリ ペインで、ファイルのリストで、**MyHealth.Web** フォルダーに移動して、**Dockerfile** を選択します。
1.  「**Dockerfile**」ペインで、「**編集**」をクリックし、ベース Docker イメージを参照する **1** 行目を `FROM az400mp/aspnetcore1.0:1.0.4` に置き換え、「**コミット**」を選択し、確認を求められたら、もう一度「**コミット**」をクリックします。 
1.  Azure DevOps ポータルが表示され、「**AKS へのマルチコンテナー アプリケーションのデプロイ**」プロジェクトが開いている Web ブラウザー ウィンドウに切り替え、Azure DevOps ポータルの一番左側にある垂直メニュー バーで「**パイプライン**」をクリックします。

    >**注**: 次に、ビルド パイプラインを変更します。

1.  **パイプライン** ペインの **「すべて」** オプションの下で、**MyHealth.AKS.build** パイプラインを示しているエントリをクリックして、**MyHealth.AKS.build** ペインで、**「編集」** をクリックします。

1.  「**MyHealth.AKS.build** パイプライン」ペインで、**パイプライン** エントリが選択されていることを確認し、「**エージェント仕様**」ドロップダウン リストで「**ubuntu-18.04**」を選択します。
1.  パイプラインのタスクのリストで、「**appsettings.json のトークンの置換**」タスクをクリックし、「**トークン パターン**」ドロップダウンリストで ```__...__``` を選択します。 
1.  パイプラインのタスクのリストで、「**サービスの実行**」タスクをクリックし、右側の「**Docker 作成**」ペインの「**Azure サブスクリプション**」ドロップダウン リストで、使用している Azure サブスクリプションを表すエントリを選択します。このラボで、「**承認**」をクリックして、対応するサービス接続を作成します。指示されたら、Azure サブスクリプションで所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントでグローバル管理者のロールがあるアカウントを使用してサインインします。

    >**注**: 認可プロセスが完了するまで待ちます。この手順では、Azure サービス接続が作成され、サービス プリンシパル認証 (SPA) を使用してターゲット Azure サブスクリプションへの接続が定義・確立されます。 

1.  パイプラインのタスク一覧で「**サービスの実行**」タスクを選択し、右側の 「**Docker Compose**」 ペインにある 「**Azure Container Registry**」 ドロップダウン リストから、このラボで先ほど作成した ACR インタンスを示しているエントリを選択します。 
     >**注**: 必要に応じて、リストを**更新**して、前に作成した ACR を選択します。**オプションが表示されない場合は、ACR の完全名前: ACRNAME.azurecr.io**) を入力します。
1.  前の 2 つの手順を繰り返し、**Azure サブスクリプション** (次回は再び承認せず、作成済みの「**利用可能な Azure サービス接続**」を使用) と「**Azure Container Registry**」の設定を「**サービスのビルド**」、「**サービスのプッシ**ュ」、「**サービスのロック**」タスクで構成します。ただし、この場合は Azure サブスクリプションを選択する代わりに、新しく作成されたサービス接続を選択します。 

    >**注**: パイプラインは以下のタスクで構成されます

    | タスク | 使用状況 |
    | ----- | ----- |
    | **トークンの置換** | プレースホルダーを、**appsettings.json** ファイルおよび **mhc-aks.yaml** マニフェスト ファイルのデータベース接続文字列の ACR 名に置き換えます |
    | **サービスの実行** |  必要なイメージ (aspnetcore-build:1.0-2.0 など) をプルし、**「.csproj」** で参照されるパッケージを復元して環境を準備します |
    | **サービスのビルド** |  **docker-compose.yml** ファイルで指定されている Docker イメージと、**$(Build.BuildId)** および **latest** タグの付いているタグ イメージをビルドします |
    | **サービスのプッシュ** |  Azure Container Registry に Docker イメージ「**myhealth.web**」をプッシュします |
    | **ビルド成果物の公開** |  **mhc-aks.yaml** および **myhealth.dacpac** ファイルを Azure DevOps の成果物格納場所に公開し、その後のリリースで利用できるようにします |

    >**注**: **appsettings.json** ファイルには、このラボで先ほど作成した Azure SQL データベースへの接続で使われるデータベース接続文字列の詳細が含まれています。**mhc-aks.yaml** マニフェスト ファイルには、**デプロイ**、**サービス**、Azure Kubernetes Service でデプロイされる**ポッド**の構成情報が含まれています。デプロイ マニフェストの詳細については、[AKS デプロイと YAML マニフェスト](https://docs.microsoft.com/ja-jp/azure/aks/concepts-clusters-workloads#deployments-and-yaml-manifests)を参照してください
1.  **パイプライン変数**のリストで、**ACR** および **SQLserver** 変数の値を前の宅すくの終了時に記録した値 (SQLPassword は **P2ssw0rd1234**、SQLuser は **sqladmin**、SQLdatabase は **mhcdb**) で更新してから、**「保存してキュー」** ボタンの横の下向きキャレット記号をクリックして、**「保存」** をクリックして、変更を保存します。再度プロンプトが表示されたら、**「保存」** をクリックします。
1.  Azure DevOps ポータルが表示されている Web ブラウザー ウィンドウで、Azure DevOps ポータルの一番左にある垂直メニュー バーの「**パイプライン**」セクションで「**リリース**」をクリックします。 
1.  「**パイプライン / リリース**」ペインで「**MyHealth.AKS.Release**」エントリを選択し、「**編集**」をクリックします。
1.  「**すべてのパイプライン / MyHealth.AKS.Release**」ペインで、デプロイの **Dev** ステージを示す長方形の中にある「**2 ジョブ、3 タスク**」リンクをクリックします。
1.  「**DB のデプロイ**」ジョブと「**AKS のデプロイ**」ジョブ (その名前をクリック) では、「Agent Pool」 (**Azure Pipelines --> windows-2019**) を選択します。
1.  **Dev** ステージのタスク一覧の「**DB デプロイ**」ジョブ セクション内で「**Azure SQL の実行: DacpacTask**」タスクを選択します。右側の「**Azure SQL Database デプロイ**」ペインで「**Azure サブスクリプション**」ドロップダウン リストから、このタスクで先ほど作成した Azure サービス接続を示すエントリを選択します。
1.  **Dev** ステージのタスク一覧の「**AKS デプロイ**」ジョブセクションで、「**AKS でデプロイとサービスを作成**」タスクを選択します。 
1.  右側の「**Kubectl**」ペインで「**Azure サブスクリプション**」ドロップダウン リストから、同じ Azure サービス接続を示すエントリを選択します。「**リソース グループ**」ドロップダウン リストから「**az400m16l01a-RG**」エントリを選択し、「**Kubernetes クラスター**」ドロップダウン リストからこのラボで先ほどデプロイした AKS クラスターを示すエントリを選択します。
1.  **Dev** ステージのタスク一覧の「**AKS デプロイ**」ジョブ セクションで、「**AKS でデプロイとサービスを作成**」タスクを選択します。右側の「**Kubectl**」ペインでスクロールダウンし、「**シークレット**」セクションを拡張します。「**Azure サブスクリプション**」ドロップダウン リストから同じ Azure サービス接続を示すエントリを選択します。「**Azure Container レジストリ**」ドロップダウン リストから、このラボで先ほど作成した Azure Container レジストリを示すエントリを選択します。
1.  「**AKS でイメージを更新**」タスクでも前の 2 つの手順を繰り返します。

    >**注**: 「**AKS でデプロイとサービスを作成**」タスクでは、**mhc-aks.yaml** ファイルで指定された構成に従い、AKS で必要なデプロイとサービスを作成します。ポッドは最新の Docker イメージをプルします。

    >**注**: 「**AKS でイメージを更新**」タスクでは、指定されたリポジトリから BuildID に対応する必要なイメージをプルし、そのイメージを AKS で実行中の **mhc-front pod** にデプロイします。

    >**注**: `kubectl create secret` コマンドをバックグラウンドで使用することで、Azure DevOps により「**mysecretkey**」と呼ばれるシークレットが AKS クラスターで作成されます。このシークレットを使用して、myhealth.web イメージをプルするために Azure Container Registry へのアクセスが承認されます。

1.  **MyHealth.AKS.Release** パイプラインの **Dev** ステージの「**タスク**」ペインで、「**変数**」タブをクリックします。 
1.  「**パイプライン変数**」リストで、前のタスクの最後に記録した Azure Container Registry 名に **ACR** 変数の値を更新します。 
1.  **パイプライン変数**リストで、**SQLserver** 変数の値を前のタスクの終了時に記録した論理サーバーの名前 (SQLPassword は **P2ssw0rd1234**、SQLuser は **sqladmin**、SQLdatabase は **mhcdb**) に更新します。 
1.  「**すべてのパイプライン / MyHealth.AKS.Release**」ペインで「**保存**」をクリックします。指示されたら再び「**保存**」をクリックして変更を保存します。

    >**注**: パイプライン変数のリストで **DatabaseName** は **mhcdb** に設定されており、**SQLuser** は **sqladmin**、**SQLpassword** は **P2ssw0rd1234** に設定されています。このラボで以前に Azure SQL データベースを作成した際に異なる値を入力していた場合は、それに従って変数の値を更新します。

#### タスク 3: ビルド パイプラインとリリース パイプラインをトリガーする

このタスクでは、ビルド パイプラインとリリース パイプラインをトリガーして、その完了を検証します。

1.  Azure DevOps ポータルが表示されている Web ブラウザー ウィンドウで、Azure DevOps ポータルの一番左にある垂直メニュー バーの「**パイプライン**」セクションで「**パイプライン**」をクリックします。 
1.  **Pipelines** ペインの **「すべて」** オプションで、**MyHealth.AKS.build** ペインの **MyHealth.AKS.build** パイプラインを選択して、**「パイプラインの実行」** をクリックして、**パイプラインの実行**ペインで、**「実行」** をクリックします。
1.  ビルド パイプライン実行ペインの「**ジョブ**」セクションで「**フェーズ 1**」をクリックし、ビルド プロセスの進捗状況を監視します。

    >**注**: ビルドでは、Docker イメージを生成して ACR をプッシュします。ビルドの完了後、ビルドの概要をレビューできます。 

1.  生成されたイメージをレビューするには、Azure ポータルを表示している Web ブラウザー ウィンドウに切り替えます。
1.  Azure ポータルで「**コンテナー レジストリ**」リソース タイプを検索して選択します。「**コンテナー レジストリ**」ブレードで、このラボで先ほど作成した Azure Container レジストリを選択します。
1.  Azure Container レジストリ ブレードの「**サービス**」セクションで「**リポジトリ**」をクリックし、リポジトリのリストに **myhealth.web** エントリが含まれていることを確認します。
1.  Azure DevOps ポータルが表示されている Web ブラウザー ウィンドウに戻ります。
1.  Azure DevOps ポータルの一番左にある垂直メニュー バーの「**パイプライン**」セクションで「**リリース**」をクリックします。「**MyHealth.AKS.Release**」ブレードで最新のリリースをクリックし、「**進行中**」リンクを選択してリリースの進捗状況を監視します。
1.  リリースが完了したら、Azure ポータルを表示している Web ブラウザー ウィンドウに切り替えます。
1.  Azure portal のツールバーで、検索テキスト ボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。 
1.  Cloud Shell ペインのバッシュ セッションから以下を実行し、このラボで先ほどデプロイした AKS クラスターへのアクセスを取得します:

    ```bash
    RGNAME=az400m16l01a-RG
    AKSNAME=$(az aks list --resource-group $RGNAME --query '[].name' --output tsv)
    az aks get-credentials --resource-group $RGNAME --name $AKSNAME
    ```

1.  Cloud Shell 画面の Bash セッションで、次を実行してリリース パイプラインを使用してデプロイされた AKS で実行中のポッドを列挙する

    ```bash
    kubectl get pods
    ```

1.  Cloud Shell ペインのバッシュ セッションから以下を実行し、コンテナー化されたアプリケーションへのアクセスで使用できる外部 IP アドレスを提供するロード バランサー サービスを一覧表示します:

    ```bash
    kubectl get service mhc-front --watch
    ```

    > **注**: このアプリケーションは、外部接続を提供するロード バランサー サービスのあるポッドでデプロイされるよう設計されています。 

1.  コマンド出力の「**External-IP**」列で IP アドレスの値をメモし、新しい Web ブラウザー タブを開きます。その IP アドレスを参照し、**MyHealthClinic** アプリケーションが実行中であることを確認します。

    >**注**: Kubernetes には、基本的な管理操作に使用できる Web ダッシュボードが含まれています。このダッシュボードでは、アプリケーションの基本的な正常性状態とメトリックの表示、サービスの作成とデプロイ、既存のアプリケーションの編集を行うことができます。[Microsoft ドキュメント](https://docs.microsoft.com/ja-jp/azure/aks/kubernetes-dashboard)に従い、Azure Kubernetes Service (AKS) で Kubernetes Web ダッシュボードにアクセスします。

### 演習 2: Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

>**注**: 新しく作成した Azure リソースのうち、使用しないリソースは必ず削除してください。使用しないリソースを削除しないと、予期しないコストが発生する場合があります。

#### タスク 1: Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'az400m16l01a-RG')].name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを削除します。

    ```sh
    az group list --query "[?starts_with(name,'az400m16l01a-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    > **注**: コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## 確認

このラボでは、Azure DevOps を使用して、コンテナー化された ASP.NET Core Web アプリケーション **MyHealthClinic** (MHC) を AKS クラスターにデプロイする方法を学習しました。 
