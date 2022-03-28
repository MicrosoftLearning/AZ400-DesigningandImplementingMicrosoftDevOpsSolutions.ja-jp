---
lab:
  title: DevOps Starter を使用して GitHub Actions を実装する
  module: 'Module 3: Implement CI with Azure Pipelines and GitHub Actions'
ms.openlocfilehash: 9a57b5948b959773dcda04d48f802bcf5a516a9f
ms.sourcegitcommit: f72fcf5ee578f465b3495f3cf789b06c530e88a4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2022
ms.locfileid: "139262522"
---
# <a name="lab-06-implementing-github-actions-by-using-devops-starter"></a>DevOps Starter を使用して GitHub Actions を実装する
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-overview"></a>ラボの概要

このラボでは、DevOps Starter を使用して Azure Web アプリ をデプロイする GitHub Actions を実装する方法を学習します。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- DevOps Starter を使用して GitHub Actions ワークフローを実装する
- GitHub Actions ワークフローの基本特性を説明する

## <a name="lab-duration"></a>ラボの所要時間

-   推定時間:**30 分**

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
-   Azure サブスクリプションで共同作成者または所有者のロールを持つ Microsoft アカウントまたは Azure AD アカウントを持っていることを確認します。 詳細については、[「Azure portal を使用して Azure ロールの割り当てを一覧表示する」](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal)および[「Azure Active Directory で管理者ロールを表示して割当てる」](https://docs.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal#view-my-roles)を参照してください。

#### <a name="prepare-a-github-account"></a>GitHub アカウントを準備する

このラボで使用できる GitHub アカウントをまだお持ちでない場合は、[新しい GitHub アカウントのサインアップ](https://github.com/join)にある手順に従ってアカウントを作成してください。

### <a name="exercise-1--create-a-devops-starter-project"></a>演習 1:DevOps Starter プロジェクトを作成する

この演習では、DevOps Starter を使用して、次のような多くのリソースのプロビジョニングを容易にします。 

-  GitHub リポジトリ ホスティング:

    -  サンプルの .NET Core Web サイトのコード。
    -  Web サイトコードをホストする Azure Web アプリをデプロイする Azure Resource Manager テンプレート。
    -  Web サイトを構築、デプロイ、およびテストするワークフロー。

-  GitHub ワークフローを使用して自動的にデプロイされる Azure Web アプリ。

#### <a name="task-1-create-devops-starter-project"></a>タスク 1:DevOps Starter プロジェクトを作成する

このタスクでは、GitHub リポジトリを自動的にセットアップする Azure DevOps Starter プロジェクトを作成し、GitHub リポジトリのコンテンツに基づいて Azure Web アプリをデプロイする GitHub ワークフローを作成してトリガーします。

1.  ラボのコンピューターから Web ブラウザーを起動し、[**Azure Portal**](https://portal.azure.com) に移動します。このラボで使用している Azure サブスクリプションで少なくとも共同作成者のロールがあるユーザー アカウントを使ってサインインします。
1.  Azure portal で、**DevOps スターター** リソースの種類を検索して選択し、**DevOps スターター** ブレードで 「 **+ 追加**、「 **+ 新規**」または 「 **+ 作成**」をクリックします。
1.  「**DevOps Starter**」ブレードの 「**新しいアプリケーションで新しく開始する**」ページで、ここをクリックして、「**GitHub を使用して DevOps Starter をセットアップする**」テキストで、**ここの** リンクをクリックします。 

    > **注**:これにより、**DevOps Starter 設定** ブレードが表示されます。 

1.  「**DevOps Starter 設定**」ブレードで、**GitHub** タイルが選択されていることを確認し、「**完了**」をクリックします。
1.  「**DevOps Starter**」ブレードに戻り、 **[次へ:フレームワーク >]** をクリックします。
1.  「**DevOps Starter**」ブレードの **[アプリケーション フレームワークの選択]** ページで、 **[ASP.NET Core]** タイルを選択し、 **[次へ:サービス >]** をクリックします。
1.  「**DevOps Starter**」ブレードの **[アプリケーションをデプロイする Azure サービスの選択]** ページで、 **[Windows Web アプリ]** タイルが選択されていることを確認し、 **[次へ:作成 >]** をクリックします。
1.  「**DevOps Starter**」ブレードの 「**リポジトリとサブスクリプションの選択**」ページで、「**承認**」をクリックします。 

    > **注**:これにより、「**Azure GitHub Actions の承認**」ポップアップ Web ブラウザー ウィンドウが表示されます。

1.  「**Azure GitHub Actions の承認**」ポップアップ ウィンドウで、必要なアクセス許可を確認し、「**Azure Github Actions の承認**」をクリックします。 

    > **注**:これにより、Web ポップアップ ブラウザー ウィンドウが Azure DevOps サイトにリダイレクトされ、Azure DevOps 情報の入力を求められます。

1.  プロンプトが表示されたら、ポップアップ Web ブラウザー ウィンドウで 「**続行**」をクリックします。
1.  「**DevOps Starter**」ブレードの 「**リポジトリとサブスクリプションの選択**」ページに戻り、次の設定を指定して、「**Review + create** をクリックします。

    | 設定 | 値 |
    | ------- | ----- |
    | Organization | GitHub アカウントの名前 |
    | リポジトリ | **az400m08l01** |
    | サブスクリプション | このラボに使用する Azure サブスクリプションの名前 |
    | Web アプリの名前 | **azurewebsites.net** DNS 名前空間内の有効でグローバルに一意のホスト名 |
    | 場所 | Azure Web アプリをプロビジョニングできる Azure リージョンの名前 |

    > **注**:プロビジョニングが完了するまで待ちます。 これには 1 分ほどかかります。

1.  「**Deploy_DevOps_Project_az400m08l01\| の概要**」ブレードで、 **[リソースに移動]** をクリックします。
1.  「**az400m08l01**」ブレードの **GitHub ワークフロー** タイルで、「**承認**」をクリックします。 
1.  「**GitHub 認可**」ブレードで、もう一度 「**承認**」をクリックします。
1.  「**az400m08l01**」ブレードに戻り、**GitHub ワークフロー** タイルでのアクションの進行状況を監視します。 

    > **注**:GitHub ワークフローのビルド、デプロイ、機能テストのジョブが完了するのを待ちます。 これには 5 分ほどかかります。

#### <a name="task-2-review-the-results-of-creating-the-devops-starter-project"></a>タスク 2:DevOps Starter プロジェクトの作成結果を確認する

このタスクでは、DevOps Starter プロジェクトの作成結果を確認します。

1.  Azure portal を表示している Web ブラウザー ウィンドウの 「**az400m08l01**」ブレードで、**GitHub ワークフロー** のセクションを確認し、**ビルド**、**デプロイ**、および **機能テスト** のジョブが正常に完了したことを確認します。
1.  「**az400m08l01**」ブレードで、**Azure リソース** のセクションを確認し、App Service Webア プリインスタンスと対応する Application Insights リソースが含まれていることを確認します。
1.  「**az400m08l01**」ブレードの上部で、前のタスクで作成した **ワークフロー ファイル** と GitHub リポジトリへのリンクをメモします。
1.  「**az400m08l01**」ブレードの上部で、GitHub リポジトリへのリンクをクリックします。 
1.  「GitHub リポジトリ」ページで、次のラベルが付いた 3 つのフォルダーに注意してください。

    - **.github\workflows** - YAML 形式のワークフロー ファイルが含まれています
    - **Application** - サンプル Web サイトのコードが含まれています
    - **ArmTemplates** ‐ ワークフローが Azure リソースのプロビジョニングに使用する Azure Resource Manager テンプレートが含まれています

1.  [GitHub リポジトリ] ページで、「 **.github/workflows**」をクリックしてから、「**devops-starter-workflow.yml**」エントリをクリックします。
1.  **devops-starter-workflow.yml** のコンテンツを表示する 「GitHub リポジトリ」ページで、そのコンテンツを確認し、**ビルド**、**デプロイ**、および **機能テスト** のジョブ定義が含まれていることに注意してください。
1.  「GitHub リポジトリ」ページのツールバーで、「**アクション**」をクリックします。
1.  「GitHub リポジトリ」ページの 「**アクション**」タブの 「**すべてのワークフロー**」セクションで、最新のワークフロー実行を表すエントリをクリックします。
1.  「ワークフロー実行」ページで、ワークフロース テータス、および **注釈** と **成果物** のリストを確認します。
1.  「GitHub リポジトリ」ページのツールバーで 「**設定**」をクリックし、「**設定**」タブで 「**シークレット**」をクリックします。
1.  「**アクション シークレット**」ペインで、ターゲットの Azure サブスクリプションにアクセスするために必要な資格情報を表す **AZURE_CREDENTIALS** エントリに注意してください。 
1.  **az400m08l01/Application/aspnet-core-dotnet-core/Pages/Index.cshtml** GitHub リポジトリページに移動し、右上隅にある鉛筆アイコンをクリックして編集モードに切り替えます。
1.  行 19 を `<div class="description line-1"> GitHub Workflow has been successfully updated</div>` に変更します。
1.  ページの下部までスクロールし、「**変更のコミット**」をクリックします。
1.  「GitHub リポジトリ」ページのツールバーで、「**アクション**」をクリックします。
1.  「**すべてのワークフロー**」セクションで、「**Index.cshtml の更新**」エントリをクリックします。
1.  **devops-starter-workflow.yml** セクションで、デプロイの進行状況を監視し、正常に完了したことを確認します。
     > **注**: **"azure/CLI@1" を使用するアクションが失敗** した場合は、**devops-starter-workflow.yml** ファイルに次の変更をコミットし (既定の azure cli バージョンを変更)、正常に完了したことを確認します。
       ```
       - name: Deploy ARM Template
          uses: azure/CLI@v1
          continue-on-error: false
          with:
            azcliversion: 2.29.2
            inlineScript: |
              az group create --name "${{ env.RESOURCEGROUPNAME }}" --location "${{ env.LOCATION }}"
              az deployment group create --resource-group "${{ env.RESOURCEGROUPNAME }}" --template-file ./ArmTemplates/windows-webapp-     template.json --parameters webAppName="${{ env.AZURE_WEBAPP_NAME }}" hostingPlanName="${{ env.HOSTINGPLANNAME }}"   appInsightsLocation="${{ env.APPINSIGHTLOCATION }}" sku="${{ env.SKU }}"
       ```
1.  Azure portal に 「DevOps Starter」ブレードを表示しているブラウザー ウィンドウに切り替え、**アプリケーション エンドポイント** エントリの横にある 「**参照**」リンクをクリックします。
1.  新しく開いた Web ブラウザー ウィンドウで、GitHub リポジトリでコミットした変更を表す更新されたテキストが Web アプリのホームページに表示されていることを確認します。

### <a name="exercise-2-remove-the-azure-lab-resources"></a>演習 2:Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

>**注**:新規に作成し、使用しなくなったすべての Azure リソースを削除することを忘れないでください。 使用していないリソースを削除することで、予期しない料金が発生しなくなります。

#### <a name="task-1-remove-the-azure-lab-resources"></a>タスク 1:Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'az400m08l01')].name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体を通して作成したすべてのリソース グループを削除します。

    ```sh
    az group list --query "[?starts_with(name,'az400m08l01')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注**:コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## <a name="review"></a>確認

このラボでは、DevOps Starter を使用してAzure Web アプリをデプロイする GitHub アクション ワークフローを実装しました。
