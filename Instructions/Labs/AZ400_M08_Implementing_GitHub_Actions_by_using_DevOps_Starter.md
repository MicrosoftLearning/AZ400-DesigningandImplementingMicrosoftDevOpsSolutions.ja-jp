---
lab:
    title: 'ラボ 08: DevOps Starter を使用した GitHub アクションの実装'
    module: 'モジュール 8: GitHub アクションとの継続的インテグレーションを実装する'
---

# ラボ 08: DevOps Starter を使用した GitHub アクションの実装
# 受講生用ラボ マニュアル

## ラボの概要

このラボでは、DevOps Starter を使用してAzure Web アプリをデプロイする GitHub アクション ワークフローを実装する方法を学習します。

## 目標

このラボを完了すると、次のことができるようになります。

- DevOps Starter を使用して GitHub アクション ワークフローを実装する
- GitHub アクション ワークフローの基本的な特徴を説明する

## ラボの所要時間

-   推定時間: **30 分**

## 手順

### 開始する前に

#### ラボの仮想マシンにログインする

次の認証情報を使用して、Windows 10 仮想マシンにサインインしていることを確認します。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### このラボで必要なアプリケーションのレビュー

このラボで使用するアプリケーションを特定:
  
-   Microsoft Edge

#### Azure サブスクリプションの準備

-   既存の Azure サブスクリプションを識別するか、新しいものを作成します。
-   Azure サブスクリプションで共同作成者または所有者のロールを持つ Microsoft アカウントまたは Azure AD アカウントを持っていることを確認します。詳細については、[Azure portal を使用して Azure ロールの割り当てを一覧表示する](https://docs.microsoft.com/ja-jp/azure/role-based-access-control/role-assignments-list-portal) および [Azure Active Directory で管理者ロールを表示して割当てる](https://docs.microsoft.com/ja-jp/azure/active-directory/roles/manage-roles-portal#view-my-roles) を参照してください。

#### GitHub アカウントを準備する

このラボで使用できる GitHub アカウントをまだお持ちでない場合は、[新しい GitHub アカウントのサインアップ](https://github.com/join)にある手順に従ってアカウントを作成してください。

### 演習 1: DevOps Starter プロジェクトを作成する

この演習では、DevOps Starter を使用して、次のような多くのリソースのプロビジョニングを容易にします。 

-  GitHub リポジトリ ホスティング:

    -  サンプルの .NET Core Web サイトのコード。
    -  Web サイトコードをホストする Azure Web アプリをデプロイする Azure Resource Manager テンプレート。
    -  Web サイトを構築、デプロイ、およびテストするワークフロー。

-  GitHub ワークフローを使用して自動的にデプロイされる Azure Web アプリ。

#### タスク 1: DevOps Starter プロジェクトを作成する

このタスクでは、GitHub リポジトリを自動的にセットアップする Azure DevOps Starter プロジェクトを作成し、GitHub リポジトリのコンテンツに基づいて Azure Web アプリをデプロイする GitHub ワークフローを作成してトリガーします。

1.  ラボのコンピューターから Web ブラウザーを起動し、[**Azure Portal**](https://portal.azure.com) に移動します。このラボで使用している Azure サブスクリプションで少なくとも共同作成者のロールがあるユーザー アカウントを使ってサインインします。
1.  Azure portal で、**DevOps Starter** リソースの種類を検索して選択し、**DevOps Starter** ブレードで 「**+ 作成**]をクリックします。
1.  「**DevOps Starter**」ブレードの 「**Start fresh with a new application**」ページで、「**Setting up DevOps starter with GitHub, change settings**」の横にある **Here**リンクをクリックします。 

    > **注**: これにより、**DevOps Starter 設定**ブレードが表示されます。 

1.  「**DevOps Starter 設定**」ブレードで、**GitHub** タイルが選択されていることを確認し、「**Done**」をクリックします。
1.  「**DevOps Starter**」ブレードに戻り 、「**次へ: Framework >**」を選択します。
1.  「**DevOps Starter**」ブレードの 「**Choose an application framework**」ページで、**ASP.NET Core** タイルを選択し、「**Next: Service >**」をクリックします。
1.  「**DevOps Starter**」ブレードの 「**Select an Azure service to deploy the application**」ページで、**Windows Web App** タイルを選択して、「**Next Create >**」をクリックします。
1.  「**DevOps Starter**」ブレードの 「**Select Repository and Subscription**」ページで、「**Authorize**」をクリックします。 

    > **注**: これにより、「**Authorize Azure Github Actions**」ポップアップ Web ブラウザー ウィンドウが表示されます。

1.  「**Authorize Azure Github Actions**」ポップアップ ウィンドウで、必要なアクセス許可を確認し、「**Authorize AzureGithubActions**」をクリックします。 

    > **注**: これにより、Web ポップアップ ブラウザー ウィンドウが Azure DevOps サイトにリダイレクトされ、Azure DevOps 情報の入力を求められます。

1.  プロンプトが表示されたら、ポップアップ Web ブラウザー ウィンドウで 「**続行**」をクリックします。
1.  「**DevOps Starter**」ブレードの 「**Select Repository and Subscription**」ページが表示されたら、以下の設定を指定して、「**Review + create**]をクリックします。

    | 設定 | 値 |
    | ------- | ----- |
    | 組織 | GitHub アカウントの名前 |
    | Repository | **az400m08l01** |
    | Subscription | このラボに使用する Azure サブスクリプションの名前 |
    | Web App Name | DNS 名前空間内の有効でグローバルに一意のホスト名. **<指定したアプリ名>.azurewebsites.net** というURLになる |
    | 場所 | Azure Web アプリをプロビジョニングできる Azure リージョンの名前 |

    > **注**: プロビジョニングが完了するのを待ってください。通常は 1 分ほどかかります。

1.  「**Deploy_DevOps_Project_az400m08l01 \| 概要**」ブレードで、「**リソースに移動**」をクリックします。
1.  「**az400m08l01**」ブレードの **GitHub Workflow** タイルで、「**Authorize**」をクリックします。 
1.  「**GitHub Authorizetion**」ブレードで、もう一度 「**Authorize**」をクリックします。
1.  「**az400m08l01**」ブレードに戻り、**GitHub Workflow** タイルでのアクションの進行状況を監視します。 

    > **注**: GitHub ワークフローのビルド、デプロイ、機能テストのジョブが完了するのを待ちます。これにはおよそ 5 分かかる場合があります。

#### タスク 2: DevOps Starter プロジェクトの作成結果を確認する

このタスクでは、DevOps Starter プロジェクトの作成結果を確認します。

1.  Azure portal を表示している Web ブラウザー ウィンドウの 「**az400m08l01**」ブレードで、**GitHub Workflow** のセクションを確認し、**Build**、**Deploy**、および**Functional tests**のジョブが正常に完了したことを確認します。
1.  「**az400m08l01**」ブレードで、**Azure Resources**のセクションを確認し、App Service Webアプリインスタンスと対応する **Application Insights** リソースが含まれていることを確認します。
1.  「**az400m08l01**」ブレードの上部で、**Workflow file** が保存されている GitHub リポジトリへのリンクをクリックします。 
1.  「**az400m08l01**」ブレードの上部で、GitHub リポジトリへのリンクをクリックします。 
1.  「GitHub リポジトリ」ページで、次のラベルが付いた 3 つのフォルダーに注意してください。

    - **.github\workflows** -  YAML 形式のワークフロー ファイルを含みます
    - **Application** -  サンプル Web サイトのコードが含まれています
    - **ArmTemplates** -  ワークフローが Azure リソースのプロビジョニングに使用する Azure Resource Manager テンプレートが含まれています

1.  「GitHub リポジトリ」ページで、**「github/Workflows」** をクリックしてから、**「devops-starter-workflow.yml」** エントリをクリックします。
1.  **devops-starter-workflow.yml** のコンテンツを表示するページで、**Build**、**Deploy**、および**FunctionalTests**のジョブ定義が含まれていることに注意してください。
1.  「GitHub リポジトリ」ページの上のツールバーで、「**Actions**」をクリックします。
1.  「**All workflows**」セクションで、最新のワークフロー実行を表すエントリをクリックします。
1.  「ワークフロー実行」ページで、ワークフロース テータス、および**Annotations（注釈）** と **Artifacts（成果物）** のリストを確認します。
1.  「GitHub リポジトリ」ページの上のツールバーで 「**Settings**」をクリックし、左側のメニューから「**Secrets**」をクリックします。
1.  「**Actions secrets**」ペインで、ターゲットの Azure サブスクリプションにアクセスするために必要な資格情報を表す **AZURE_CREDENTIALS** エントリに注意してください。 
1.  **Application/aspnet-core-dotnet-core/Pages/Index.cshtml** GitHub リポジトリページに移動し、右上隅にある鉛筆アイコンをクリックして編集モードに切り替えます。
1.  19行目を `<div class="description line-1"> GitHub Workflow has been successfully updated</div>` に変更します。
1.  ページの下部までスクロールし、「**Commit changes**」をクリックします。
1.  「GitHub リポジトリ」ページのツールバーで、「**Actions**」をクリックします。
1.  「**すべてのワークフロー**」セクションで、「**Update Index.cshtml**」エントリをクリックします。
1.  **devops-starter-workflow.yml** セクションで、デプロイの進行状況を監視し、正常に完了したことを確認します。
     >**注**: **"azure/CLI@1" を使用する操作に失敗した場合は**、次の変更を **devops-starter-workflow.yml** ファイルに対してコミットし (既定の Azure CLI のバージョンを変更します)、正常に完了することを確認します。
       ```
       - name: ARM テンプレートのデプロイ
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

### 演習 2: Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

>**注**: Azure リソースのうち、使用しないリソースは必ず削除してください。使用しないリソースを削除しないと、予期しないコストが発生する場合があります。

#### タスク 1: Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'az400m08l01')].name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを削除します。

    ```sh
    az group list --query "[?starts_with(name,'az400m08l01')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注**: コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## レビュー

このラボでは、DevOps Starter を使用してAzure Web アプリをデプロイする GitHub アクション ワークフローを実装しました。
