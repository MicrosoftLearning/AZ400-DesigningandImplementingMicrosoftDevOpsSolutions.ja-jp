---
lab:
  title: Azure Bicep テンプレートを使用したデプロイ
  module: 'Module 05: Manage infrastructure as code using Azure and DSC'
---

# Azure Bicep テンプレートを使用したデプロイ

## ラボの要件

- このラボには、**Microsoft Edge** または [Azure DevOps 対応ブラウザー](https://docs.microsoft.com/azure/devops/server/compatibility)が必要です。

- **Azure DevOps 組織を設定する:** このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization)に関するページの手順に従って作成してください。

- 既存の Azure サブスクリプションを識別するか、新しいものを作成します。

- Azure サブスクリプションの所有者ロールと、Azure サブスクリプションに関連付けられた Microsoft Entra テナントの全体管理者ロールを持つ Microsoft アカウントまたは Microsoft Entra アカウントがあることを確認します。 詳細については、[「Azure portal を使用して Azure ロールの割り当てを一覧表示する」](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal)および[「Azure Active Directory で管理者ロールを表示して割当てる」](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal)を参照してください。

## ラボの概要

このラボでは、Azure Bicep テンプレートを作成し、Azure Bicep モジュールの概念を使ってそれをモジュール化します。 次に、モジュールを使用するようにメイン デプロイ テンプレートを変更して、最後にすべてのリソースを Azure にデプロイします。

## 目標

このラボを完了すると、次のことができるようになります。

- Azure Bicep テンプレートの構造を理解します。
- 再利用可能な Bicep モジュールを作成します。
- モジュールを使うようにメイン テンプレートを変更する
- Azure YAML パイプラインを使って、すべてのリソースを Azure にデプロイします。

## 推定時間:45 分

## 手順

### 演習 0: ラボの前提条件を構成する (完了している場合はスキップしてください)

この演習では、ラボの前提条件を設定します。これは、[eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) に基づくリポジトリを含む新しい Azure DevOps プロジェクトで構成されます。

#### タスク 1: (完了している場合はスキップしてください) チーム プロジェクトを作成して構成する

このタスクでは、複数のラボで使用される **eShopOnWeb** Azure DevOps プロジェクトを作成します。

1. ラボ コンピューターのブラウザー ウィンドウで、Azure DevOps 組織を開きます。 **[新しいプロジェクト]** をクリックします。 プロジェクトに「**eShopOnWeb**」という名前を付け、他のフィールドは既定値のままにします。 **[作成]** をクリックします。

    ![[新しいプロジェクトの作成] パネルのスクリーンショット。](images/create-project.png)

#### タスク 2: (完了している場合はスキップしてください) eShopOnWeb Git リポジトリをインポートする

このタスクでは、複数のラボで使用される eShopOnWeb Git リポジトリをインポートします。

1. ラボ コンピューターのブラウザー ウィンドウで、Azure DevOps 組織と、前に作成した **eShopOnWeb** プロジェクトを開きます。 **[リポジトリ]、[ファイル]**、**[リポジトリをインポートする]** の順にクリックします。 **インポート** を選択します。 **[Git リポジトリをインポートする]** ウィンドウで、URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> を貼り付けて、 **[インポート]** をクリックします。

    ![[リポジトリのインポート] パネルのスクリーンショット。](images/import-repo.png)

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

### 演習 1: Azure Bicep テンプレートを理解し、再利用可能なモジュールを使って簡略化する

このラボでは、Azure Bicep テンプレートを確認し、再利用可能なモジュールを使って簡略化します。

#### タスク 1: Azure Bicep テンプレートを作成する

このタスクでは、Visual Studio Code を使って Azure Bicep テンプレートを作成します

1. Azure DevOps プロジェクトを開いているブラウザーのタブで、 **[リポジトリ]** と **[ファイル]** に移動します。 `infra` フォルダーを開き、`simple-windows-vm.bicep` ファイルを見つけます。

   ![simple-windows-vm.bicep ファイル パスのスクリーンショット。](./images/m06/browsebicepfile.png)

1. テンプレートをレビューし、その構造をよりよく把握します。 型、既定値、検証を含むいくつかのパラメーター、いくつかの変数、以下の型を持つかなりの数のリソースがあります。

   - Microsoft.Storage/storageAccounts
   - Microsoft.Network/publicIPAddresses
   - Microsoft.Network/virtualNetworks
   - Microsoft.Network/networkInterfaces
   - Microsoft.Compute/virtualMachines

1. リソース定義のシンプルさと、テンプレート全体を通して明示的な `dependsOn` ではなく暗黙的にシンボリック名を参照できることに注意してください。

#### タスク 2: ストレージ リソースの Bicep モジュールを作成する

このタスクでは、ストレージ テンプレート モジュール **storage.bicep** を作成します。これにより、ストレージ アカウントのみが作成され、メイン テンプレートによってインポートされます。 ストレージ テンプレート モジュールは、メイン テンプレート **main.bicep** に値を戻す必要があります。この値は、ストレージ テンプレート モジュールの出力要素で定義されます。

1. まず、メイン テンプレートからストレージ リソースを削除する必要があります。 ブラウザー ウィンドウの右上隅から **[編集]** ボタンをクリックします。

   ![[パイプラインの編集] ボタンのスクリーンショット。](./images/m06/edit.png)

1. 次にストレージ リソースを削除します。

   ```bicep
   resource storageAccount 'Microsoft.Storage/storageAccounts@2022-05-01' = {
     name: storageAccountName
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'Storage'
   }
   ```

1. 20 行目の `publicIPAllocationMethod` パラメーターの既定値を `Dynamic` から `Static` に変更します

1. 27 行目の `publicIpSku` パラメーターの既定値を `Basic` から `Standard` に変更します

1. ファイルをコミットしますが、まだこれで終わりではありません。

   ![[ファイルのコミット] ボタンのスクリーンショット。](./images/m06/commit.png)

1. 次に、`Infra` フォルダーをマウスでポイントして省略記号アイコンをクリックした後、**[新規作成]**、**[ファイル]** の順に選択します。 名前に「**`storage.bicep`**」と入力し、**[作成]** をクリックします。

   ![[新しいファイル] メニューのスクリーンショット。](./images/m06/newfile.png)

1. 次に、次のコード スニペットを ファイルにコピーし、変更をコミットします。

   ```bicep
   @description('Location for all resources.')
   param location string = resourceGroup().location

   @description('Name for the storage account.')
   param storageAccountName string

   resource storageAccount 'Microsoft.Storage/storageAccounts@2022-05-01' = {
     name: storageAccountName
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'Storage'
   }

   output storageURI string = storageAccount.properties.primaryEndpoints.blob
   ```

#### タスク 3:テンプレート モジュールを使用するように simple-windows-vm テンプレートを変更する

このタスクでは、前のタスクで作成したテンプレート モジュールを参照するように、`simple-windows-vm.bicep` テンプレートを変更します。

1. `simple-windows-vm.bicep` ファイルに戻り、 **[編集]** ボタンをもう一度クリックします。

1. 次に、変数の後に次のコードを追加します。

   ```bicep
   module storageModule './storage.bicep' = {
     name: 'linkedTemplate'
     params: {
       location: location
       storageAccountName: storageAccountName
     }
   }
   ```

1. また、代わりにモジュールの出力を使うように、仮想マシン リソースのストレージ アカウント BLOB URI への参照を変更する必要があります。 仮想マシン リソースを見つけて、diagnosticsProfile セクションを次のように置き換えます。

   ```bicep
   diagnosticsProfile: {
     bootDiagnostics: {
       enabled: true
       storageUri: storageModule.outputs.storageURI
     }
   }
   ```

1. メイン テンプレートで以下の詳細をレビューします:

   - メイン テンプレートのモジュールは、別のテンプレートにリンクするために使われます。
   - モジュールのシンボル名は `storageModule` です。 依存関係を構成する場合はこの名前を使います。
   - テンプレート モジュールを使うときは、**増分**デプロイ モードのみを使用できます。
   - テンプレート モジュールには相対パスを使います。
   - パラメーターを使って、メイン テンプレートからテンプレート モジュールに値を渡します。

1. テンプレートをコミットします。

### 演習 2: YAML パイプラインを使って Azure にテンプレートをデプロイする

このラボでは、Azure DevOps YAML パイプラインを使用して、テンプレートを Azure 環境にデプロイします。

#### タスク 1: YAML パイプラインによって Azure にリソースをデプロイする

1. **[パイプライン]** ハブで **[パイプライン]** に戻ります。
1. **[最初のパイプラインを作成]** ウィンドウで、**[パイプラインの作成]** をクリックします。

    > **注**: ウィザードを使い、プロジェクトに基づいて新しい YAML パイプラインの定義を作成します。

1. **[コードはどこにありますか?]** ペインで **[Azure Repos Git (YAML)]** オプションをクリックします。
1. **[リポジトリの選択]** ペインで **eShopOnWeb** をクリックします。
1. **[パイプラインを構成する]** ペインで、 **[既存の Azure Pipelines YAML ファイル]** を選びます。
1. **[既存の YAML ファイルを選択する]** ペインで、次のパラメーターを指定します。
   - [ブランチ]: **main**
   - パス: **.ado/eshoponweb-cd-windows-cm.yml**
1. **[続行]** をクリックして、これらの設定を保存します。
1. 変数セクションで、リソース グループの名前を選び、目的の場所を設定し、サービス接続の値を先ほど作成した既存のサービス接続のいずれかに置き換えます。
1. 右上隅のコーダーから **[保存して実行]** ボタンをクリックし、コミット ダイアログが表示されたら、もう一度 **[保存して実行]** をクリックします。

   ![[保存して実行] ボタンのスクリーンショット。](./images/m06/saveandrun.png)

1. デプロイが完了するまで待ってから、結果を確認します。
   ![YAML パイプラインを使った Azure へのリソース デプロイに成功したことを示すスクリーンショット。](./images/m06/deploy.png)

   > **注**:前に作成したサービス接続を使用するためのアクセス許可をパイプラインに付与することを忘れないでください。

   > [!IMPORTANT]
   > 不要な料金が発生しないように、Azure portal で作成されたリソースを必ず削除してください。

## 確認

このラボでは、Azure Bicep テンプレートを作成し、テンプレート モジュールを使ってこれをモジュラー化し、メイン デプロイ テンプレートを変更してモジュールと更新済みの依存関係を呼び出し、最後に YAML パイプラインを使ってテンプレートを Azure にデプロイする方法を学びました。
