---
lab:
  title: ラボ 15:Terraform と Azure Pipelines を使用したクラウドでのインフラストラクチャのデプロイの自動化
  module: 'Module 06: Manage infrastructure as code using Azure, DSC, and third-party tools'
ms.openlocfilehash: e98e6235c4b2e3604390f66109dd395ae7c0d2b9
ms.sourcegitcommit: f72fcf5ee578f465b3495f3cf789b06c530e88a4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2022
ms.locfileid: "139262572"
---
# <a name="lab-15-automating-infrastructure-deployments-in-the-cloud-with-terraform-and-azure-pipelines"></a>ラボ 15:Terraform と Azure Pipelines を使用したクラウドでのインフラストラクチャのデプロイの自動化
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-overview"></a>ラボの概要

[Terraform](https://www.terraform.io/intro/index.html) は、インフラストラクチャを構築、変更、およびバージョン管理するためのツールです。 Terraform は、既存および一般的なクラウド サービス プロバイダーだけでなく、カスタムの社内ソリューションも管理できます。

Terraform 構成ファイルは、単一のアプリケーションまたはデータセンター全体を実行するために必要なコンポーネントを記述します。 Terraform は、目的の状態に到達するために何をするかを記述する実行プランを生成し、それを実行して説明されたインフラストラクチャを構築します。 構成が変更されると、Terraform は変更内容を判断し、増分実行プランを作成できます。 

このラボでは、Terraform を Azure Pipeline に取り入れてインフラストラクチャをコードとして実装する方法を学びます。 

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- Terraform を使用してインフラストラクチャをコードとして実装する
- Terraform および Azure Pipelines を使用して Azure でインフラストラクチャのデプロイを自動化する

## <a name="lab-duration"></a>ラボの所要時間

-   予想所要時間: **60 分**

## <a name="instructions"></a>Instructions

### <a name="before-you-start"></a>開始する前に

#### <a name="sign-in-to-the-lab-virtual-machine"></a>ラボ仮想マシンにサインインする

次の認証情報を使用して、Windows 10 仮想マシンにサインインしていることを確認します。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### <a name="review-applications-required-for-this-lab"></a>このラボで必要なアプリケーションのレビュー

このラボで使用するアプリケーションを特定:
    
-   Microsoft Edge

#### <a name="set-up-an-azure-devops-organization"></a>Azure DevOps 組織を設定します。 

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)で利用できる手順に従って作成してください。

#### <a name="prepare-an-azure-subscription"></a>Azure サブスクリプションの準備

-   既存の Azure サブスクリプションを識別するか、新しいものを作成します。
-   Azure サブスクリプションでは所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントではグローバル管理者のロールで Microsoft アカウントまたは Azure AD アカウントがあることを確認します。

### <a name="exercise-0-configure-the-lab-prerequisites"></a>演習 0:ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。これは、Azure DevOps Demo Generator テンプレートに基づいて事前に構成された Parts Unlimited チーム プロジェクトで構成されます。 

#### <a name="task-1-configure-the-team-project"></a>タスク 1:チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用し、**Terraform** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボのコンピューターで Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。 このユーティリティ サイトは、ラボで必要なコンテンツ (作業項目、リポジトリなど) が事前設定されている新しい Azure DevOps プロジェクトをアカウント内で作成するプロセスを自動化します。 

    > **注**:サイトの詳細については、 https://docs.microsoft.com/en-us/azure/devops/demo-gen を参照してください。

1.  「**サインイン**」をクリックし、Azure DevOps サブスクリプションに関連のある Microsoft アカウントを使用してサインインします。
1.  必要な場合は、「**Azure DevOps Demo Generator**」ページで「**承諾する**」をクリックし、Azure DevOps サブスクリプションへのアクセス許可要求を承諾します。
1.  **新しいプロジェクトの作成** ページで **[新しいプロジェクト名]** テキストボックスに「**Automating infrastructure deployments with Terraform**」と入力します。 **[組織の選択]** ドロップダウン リストで Azure DevOps 組織を選択し、 **[テンプレートの選択]** をクリックします。
1.  テンプレートのリストのツールバーで「**DevOps ラボ**」をクリックし、「**Terraform**」テンプレートを選択して「**テンプレートの選択**」をクリックします。
1.  再び「**新しいプロジェクトの作成**」ページで、欠落している拡張機能をインストールするよう指示されたら「**トークンの置換**」と「**Terraform**」の下にあるチェックボックスを選択し、「**プロジェクトの作成**」をクリックします。

    > **注**:プロセスが完了するまでお待ちください。 これには 2 分ほどかかります。 プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新しいプロジェクトの作成**」ページで「**プロジェクトに移動**」をクリックします。

### <a name="exercise-1-automate-infrastructure-deployments-in-the-cloud-with-terraform-and-azure-pipelines"></a>演習 1:Terraform および Azure Pipelines を使用してクラウドでのインフラストラクチャのデプロイを自動化

この演習では、Terraform と Azure Pipelines を使用して、コードとしてのインフラストラクチャを Azure にデプロイします

#### <a name="task-1-examine-the-terraform-configuration-files"></a>タスク 1:Terraform 構成ファイルを検証する

このタスクでは、PartsUnlimited Web サイトのデプロイに必要な Azure リソースのプロビジョニングにおける Terraform の使用を検証します。

1.  ラボのコンピューターで、**Terraform を使用したインフラストラクチャのデプロイの自動化** プロジェクトが開かれている Azure DevOps ポータルが表示された Web ブラウザーで、Azure DevOps ポータルの一番左側にある垂直メニュー バーの **[リポジトリ]** をクリックします。
1.  「**ファイル**」ペインで最上部にある **マスター** エントリの隣にある下向きキャレット記号をクリックします。ブランチのドロップダウン リストで **terraform** ブランチを示しているエントリをクリックします。

    > **注**:**Terraform** ブランチが表示され、リポジトリのコンテンツ内で **Terraform** フォルダーが表示されていることを確認します。 

1.  **Terraform** リポジトリのフォルダー階層で **Terraform** フォルダーを拡張し、**webapp.tf** をクリックします。
1.  **webapp.tf** で、**webapp.tf** ファイルの内容を確認し、「**編集**」をクリックします。
1.  **[provider]** セクションに新しい行を追加します。ファイルは次の例のようになります。

    ```
     terraform {
      required_version = ">= 0.11" 
      backend "azurerm" {
      storage_account_name = "__terraformstorageaccount__"
        container_name       = "terraform"
        key                  = "terraform.tfstate"
        access_key  ="__storagekey__"
        }
    }
    provider "azurerm" {
        features {} 
      }

    resource "azurerm_resource_group" "dev" {
      name     = "PULTerraform"
      location = "West Europe"
    }

    resource "azurerm_app_service_plan" "dev" {
      name                = "__appserviceplan__"
      location            = "${azurerm_resource_group.dev.location}"
      resource_group_name = "${azurerm_resource_group.dev.name}"

      sku {
        tier = "Free"
        size = "F1"
      }
    }

    resource "azurerm_app_service" "dev" {
      name                = "__appservicename__"
      location            = "${azurerm_resource_group.dev.location}"
      resource_group_name = "${azurerm_resource_group.dev.name}"
      app_service_plan_id = "${azurerm_app_service_plan.dev.id}"

    }
    ```
1.  **[コミット]** をクリックし、 **[コミット]** ペインでもう一度 **[コミット]** をクリックします。

    > **注**:**webapp.tf** は terraform 構成ファイルです。 Terraform は、HCL (Hashicorp Configuration Language) と呼ばれる、YAML に類似した独自のファイル形式を使用します。

    > **注**:この例では、Azure リソース グループ、アプリ サービス プラン、および Web サイトのデプロイで必要なアプリ サービスを作成します。 Terraform ファイルは、Azure DevOps プロジェクトのソース コントロール リポジトリに追加されているので、これを使って必要な Azure リソースをデプロイできます。 
    
#### <a name="task-2-build-your-application-using-azure-ci-pipeline"></a>タスク 2:Azure CI Pipeline を使用してアプリケーションを構築する

このタスクでは、アプリケーションを構築し、ドロップと呼ばれる成果物として必要なファイルを公開します。

1.  Azure DevOps ポータルで、Azure DevOps ポータルの左側にある垂直メニュー バーで「**パイプライン**」をクリックします。 その下で「**パイプライン**」を選択します。
1.  「**パイプライン**」ペインで「**Terraform-CI**」をクリックして選択します。「**Terraform-CI**」ペインで「**編集**」をクリックします。

    > **注**:この CI パイプラインには .NET Core プロジェクトをコンパイルするタスクがあります。 パイプラインの DotNet タスクは依存関係を復元し、ビルド出力を構築してテストし、zip ファイル (パッケージ) に公開します。これを Web アプリケーションにデプロイできます。 アプリケーション ビルドのほか、terraform ファイルを公開して成果物を構築し、CD パイプラインで利用できるようにする必要があります。 **ファイルのコピー** タスクで Terraform ファイルを成果物ディレクトリにコピーするのはそのためです。

1.  **Terraform-CI** ペインの「**タスク**」タブを確認したら、「**キュー**」をクリックします (ドロップダウン メニューを表示するには、最初にペインの右上隅にある省略記号をクリックする必要がある場合があります)。
1.  「**パイプラインの実行**」ペインで「**実行**」をクリックし、ビルドを開始します。 
1.  ビルド実行ペインの「**サマリー**」タブの「**ジョブ**」セクションで、「**エージェント ジョブ 1**」をクリックし、ビルド プロセスの進捗状況を監視します。
1.  ビルドに成功した後、ビルド実行ペインの「**サマリー**」タブに戻り、「**関連**」セクションで「**公開済み 1、消費済み 1**」リンクをクリックします。 「**成果物**」ペインが表示されます。
1.  「**成果物**」ペインの「**公開済み**」タブで **ドロップ** フォルダーを拡張します。**PartsUnlimitedwebsite.zip** ファイルが含まれていることを確認してから、その **Terraform** サブフォルダーを拡張し、**webapp.tf** ファイルが含まれていることを確認します。 

#### <a name="task-3-deploy-resources-using-terraform-iac-in-azure-cd-pipeline"></a>タスク 3:Azure CD パイプラインで Terraform (IaC) を使用してリソースをデプロイする

このタスクでは、デプロイパイプラインの一環として Terraform を使用して Azure リソースを作成し、Terraform でプロビジョニングされた Azure アプリ サービス Web アプリに PartsUnlimited アプリケーションをデプロイします。

1.  Azure DevOps ポータルで、Azure DevOps ポータルの左側にある垂直メニューの「**パイプライン**」セクションで「**リリース**」をクリックします。**Terraform-CD** エントリが選択されていることを確認し、「**編集**」をクリックします。
1.  **「すべてのパイプライン」>「Terraform-CD」** ペインの「**Dev**」ステージを示す長方形で「**1 ジョブ、8 タスク**」リンクをクリックします。
1.  「**Dev**」ステージのタスク リストで「**必要な Azure リソースをデプロイする Azure CLI**」タスクを選択します。 
1.  「**Azure CLI**」ペインの「**Azure サブスクリプション**」ドロップダウン リストで、Azure サブスクリプションを示すエントリを選択してから「**承認**」をクリックし、該当するサービス接続を構成します。 指示されたら、Azure サブスクリプションで所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントでグローバル管理者のロールがあるアカウントを使用してサインインします。

    > **注**:既定で、Terraform は「terraform.tfstate」という名前のファイルに状態をローカルで格納します。 チームで Terraform を使用する際は、ローカル ファイルを使用すると Terraform の使用が複雑になります。 Terraform は状態の共有を容易にするリモート データ ストアに対応しています。 この場合は、**Azure CLI** タスクを使用して Azure ストレージ アカウントと BLOB コンテナーを作成して Terraform の状態を格納します。 Terraform リモート状態の詳細については、[Terraform ドキュメント](https://www.terraform.io/docs/state/remote.html)を参照してください

1.  「**Dev**」ステージのタスク リストで「**Azure PowerShell**」タスクを選択します。 
1.  「**Azure PowerShell**」ペインの「**Azure 接続タイプ**」ドロップダウン リストで「**Azure Resource Manager**」を選択します。「**Azure サブスクリプション**」ドロップダウン リストで、新しく作成された Azure サービス接続 (「使用可能な Azure サービス接続」の下にある接続) を選択します。

    > **注**:Terraform [バックエンド](https://www.terraform.io/docs/backends/)を構成するには、Terraform の状態をホストする Azure ストレージ アカウントへのアクセス キーが必要です。 この場合、Azure PowerShell タスクを使用して、前のタスクでプロビジョニングされた Azure ストレージ アカウントのアクセス キーを取得します。 `Write-Host "##vso[task.setvariable variable=storagekey]$key"` を使用して、後のタスクで使用できるパイプライン変数を作成します。

1.  「**Dev**」ステージのタスク リストで「**Terraform ファイルでトークンを置き換える**」タスクを選択します。

    > **注**:**webapp.tf** ファイルをよく確認すると、`__terraformstorageaccount__` のように、いくつかの値に **__** のサフィックスとプレフィックスが付いているのがわかります。 「**トークンの置換**」タスクは、このような値を、リリース パイプラインで定義した変数値に置き換えます。
     
1.  Terraform ツール インストーラー タスクを使用して、Terraform の指定されたバージョンをインターネットまたはツール キャッシュからインストールし、Azure パイプライン エージェント (ホステッドまたはプライベート) の PATH の先頭に追加します。
1.  「**Dev**」ステージのタスク リストで「**Terraform のインストール**」タスクを選択して確認します。 このタスクでは、指定したバージョンの Terraform がインストールされます。

    > **注**:Terraform を自動で実行する場合は通常、以下の 3 つのステージで構成されたコア プラン / 適用サイクルに焦点が当てられます:

    1.  Terraform 作業ディレクトリの初期化。
    1.  希望する構成に合わせるために現在の構成を変更する計画の策定。
    1.  計画ステージで判別された変更の適用。
    
    > **注**:リリース パイプラインで残りの Terraform タスクは、このワークフローを実装します。

1.  「**Dev**」ステージのタスク リストで「**Terraform: init**」タスクを選択します。 
1.  「**Terraform**」ペインの「**Azure サブスクリプション**」ドロップダウン リストで、以前に使用したものと同じ Azure サービス接続を選択します。 
1.  「**Terraform**」ペインで「**コンテナー**」ドロップダウン リストに「**terraform**」と入力し、**Key** パラメーターが **terraform.tfstate** に設定されていることを確認します。 

    > **注**:`terraform init` コマンドは現在の作業ディレクトリで *.tf ファイルをすべて解析し、その処理に必要なプロバイダーを自動的にダウンロードします。 この例では、Azure リソースをデプロイしているので、[Azure プロバイダー](https://www.terraform.io/docs/providers/azurerm/)をダウンロードします。 `terraform init` コマンドの詳細については、[Terraform のドキュメント](https://www.terraform.io/docs/commands/init.html)を参照してください

1.  「**Dev**」ステージのタスク リストで「**Terraform: plan**」タスクを選択します。
1.  「**Terraform**」ペインの「**Azure サブスクリプション**」ドロップダウン リストで、以前に使用したものと同じ Azure サービス接続を選択します。 
1.  **[Terraform]** ペインの **追加のコマンド引数** テキスト ボックスに、`-out=tfplan` と入力します。

    > **注**:`terraform plan` コマンドは、実行プランを作成するために使用します。 Terraform は、構成ファイルで指定された希望する状態を実現するために必要なアクションを決定します。 これにより、実際に変更を適用しなくても、どの変更が範囲内なのか確認できます。 `terraform plan` コマンドの詳細については、[Terraform のドキュメント](https://www.terraform.io/docs/commands/plan.html)を参照してください

1.  「**Dev**」ステージのタスク リストで「**Terraform: apply -auto-approve**」タスクを選択します。 
1.  「**Terraform**」ペインの「**Azure サブスクリプション**」ドロップダウン リストで、以前に使用したものと同じ Azure サービス接続を選択します。 
1.  **Terraform** ペインの **[追加のコマンド引数]** テキスト ボックスで、現在のエントリを `-auto-approve tfplan` に置き換えます。

    > **注**:このタスクは `terraform apply` コマンドを実行してリソースをデプロイします。 既定により、確認して続行するよう指示されます。 ここではデプロイを自動化しているため、タスクには確認の必要がない `auto-approve` パラメーターが含まれています。 

1.  「**Dev**」ステージのタスク リストで「**Azure App Service のデプロイ**」タスクを選択します。 ドロップダウン リストから Azure サービス接続を選択します。

    > **注**:このタスクでは、前のステップの「**Terraform: apply -auto-approve**」タスクでプロビジョニングされた Azure App Service に PartsUnlimited パッケージをデプロイします。

1.  「**Dev**」ステージで、「**エージェント ジョブ**」をクリックし、「エージェント プール」ドロップダウンリストで、「**Azure Pipelines > windows-2019**」を選択します。
1.  **「すべてのパイプライン」> 「Terraform-CD」** ペインで「**保存**」をクリックします。「**保存**」ダイアログ ボックスで「**OK**」をクリックし、右上コーナーで「**リリースの作成**」をクリックします。 
1.  「**新しいリリースの作成**」ペインの「**自動から手動への変更トリガーのステージ**」ドロップダウン リストで「**Dev**」をクリックします。「**成果物**」セクションの「**バージョン**」ドロップダウン リストでこのリリースの成果物のバージョンを示すエントリを選択して「**作成**」をクリックします。
1.  Azure DevOps ポータルで「**Terraform-CD**」ペインに戻り、新しく作成されたリリースを示す「**Release-1**」エントリをクリックします。 
1.  **「Terraform-CD」 > 「Release-1」** ブレードで、「**Dev**」ステージを示す長方形をクリックします。「**Dev**」ペインで「**デプロイ**」をクリックしてから再び「**デプロイ**」をクリックします。
1.  再び **「Terraform-CD」 > 「Release-1」** ブレードで、「**Dev**」ステージを示す長方形をクリックし、デプロイ プロセスを監視します。 
1.  リリースが正常に完了したら、ラボのコンピューターで別の Web ブラウザー ウィンドウを開き、[**Azure portal**](https://portal.azure.com) に移動します。このラボで使用する Azure サブスクリプションで、少なくとも共同作成者のロールがあるユーザー アカウントを使ってサインインします。 
1.  Azure portal で **App Services** のリソースを検索して選択します。「**App Services**」ブレードから、**pulterraformweb** で始まる名前の Web アプリに移動します。 
1.  Web アプリ ブレードで「**参照**」をクリックします。 別の Web ブラウザー タブが開き、新しくデプロイされた Web アプリケーションが表示されます。

### <a name="exercise-2-remove-the-azure-lab-resources"></a>演習 2:Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

>**注**:新規に作成し、使用しなくなったすべての Azure リソースを削除することを忘れないでください。 使用していないリソースを削除することで、予期しない料金が発生しなくなります。

#### <a name="task-1-remove-the-azure-lab-resources"></a>タスク 1:Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query '[?contains(`["terraformrg", "PULTerraform"]`, name)].name' --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体を通して作成したすべてのリソース グループを削除します。

    ```sh
    az group list --query '[?contains(`["terraformrg", "PULTerraform"]`, name)].name' --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注**:コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。


## <a name="review"></a>確認

このラボでは、Azure Pipelines を使用して Azure 上で Terraform を使い、反復可能なデプロイを自動化する方法を学習しました。
