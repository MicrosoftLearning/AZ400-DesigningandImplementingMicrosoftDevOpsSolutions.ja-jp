---
lab:
  title: Azure Bicep テンプレートを使用したデプロイ
  module: 'Module 06: Manage infrastructure as code using Azure and DSC'
---

# Azure Bicep テンプレートを使用したデプロイ

## 受講生用ラボ マニュアル

## ラボの要件

- このラボには、**Microsoft Edge** または [Azure DevOps 対応ブラウザー](https://docs.microsoft.com/azure/devops/server/compatibility)が必要です。

- **Azure DevOps 組織を設定する:** このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization)に関するページの手順に従って作成してください。

- 既存の Azure サブスクリプションを識別するか、新しいものを作成します。

- Azure サブスクリプションでは所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントではグローバル管理者のロールで Microsoft アカウントまたは Azure AD アカウントがあることを確認します。

- [Visual Studio Code](https://code.visualstudio.com/)。 このラボでは前提条件の一部としてインストールされます。

## ラボの概要

このラボでは、Azure Bicep テンプレートを作成し、Azure Bicep モジュールの概念を使ってそれをモジュール化します。 次に、モジュールを使うようにメイン デプロイ テンプレートを変更して、最後にすべてのリソースを Azure にデプロイします。

## 目標

このラボを完了すると、次のことができるようになります。

- Azure Bicep テンプレートを理解して作成する。
- ストレージ リソース用の再利用可能な Bicep モジュールを作成する。
- リンク済みテンプレートを Azure Blob Storage にアップロードして SAS トークンを生成する。
- モジュールを使用するようにメイン テンプレートを変更する。
- メイン テンプレートを修正して、依存関係を更新する。
- Azure Bicep テンプレートを使用して、すべてのリソースを Azure にデプロイする。

## 推定時間:60 分

## Instructions

### 演習 0:ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。この中には、Visual Studio Code が含まれます。

#### タスク 1:Git と Visual Studio Code をインストールして構成する

このタスクでは、Visual Studio Code をインストールします。 この前提条件をすでに実装している場合は、直接、次のタスクに進むことができます。

1. まだ Visual Studio Code がインストールされていない場合は、ラボのコンピューターから Web ブラウザーを起動し、[Visual Studio Code ダウンロード ページ](https://code.visualstudio.com/) に移動し、これをダウンロードしてインストールします。

### 演習 1: Azure Bicep テンプレートを作成してデプロイする

このラボでは、Azure Bicep テンプレートとテンプレート モジュールを作成します。 その後、テンプレート モジュールを使うようにメイン デプロイ テンプレートを変更し、依存関係を更新して、最後にテンプレートを Azure にデプロイします。

#### タスク 1: Azure Bicep テンプレートを作成する

このタスクでは、Visual Studio Code を使って Azure Bicep テンプレートを作成します

1. ラボのコンピューターから Visual Studio Code を起動し、Visual Studio Code で **[ファイル]** トップ レベル メニューをクリックし、ドロップダウン メニューで **[基本設定]** を選び、カスケード メニューで **[拡張機能]** を選び、 **[拡張機能の検索]** テキストボックスに「**Bicep**」と入力し、Microsoft によって発行されたものを選び、 **[インストール]** をクリックして Azure Bicep 言語サポートをインストールします。
2. Web ブラウザーで、 **<https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vm-simple-windows/main.bicep>** に接続します。 ファイルの **[Raw]** オプションをクリックします。 コード ウィンドウの内容をコピーして、Visual Studio コード エディターに貼り付けます。

   > **注**:テンプレートを最初から作成するよりも、[Azure クイックスタート テンプレート](https://azure.microsoft.com/resources/templates/) のひとつ (**シンプルな Windows テンプレート VM のデプロイ**) を使用します。 テンプレートは GitHub - [vm-simple-windows](https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vm-simple-windows) からダウンロードできます。

3. ラボのコンピューターでエクスプローラーを開き、テンプレートの格納に使う次のローカル フォルダーを作成します。

   - **C:\\templates**

4. main.bicep テンプレートのある Visual Studio Code ウィンドウに戻り、 **[ファイル]** トップ レベル メニューをクリックし、ドロップダウン メニューで **[名前を付けて保存]** をクリックし、新しく作成したローカル フォルダー **C:\\templates** にテンプレートを **main.bicep** として保存します。
5. テンプレートをレビューし、その構造をよりよく把握します。 テンプレートには 5 種類のリソースが含まれています。

   - Microsoft.Storage/storageAccounts
   - Microsoft.Network/publicIPAddresses
   - Microsoft.Network/virtualNetworks
   - Microsoft.Network/networkInterfaces
   - Microsoft.Compute/virtualMachines

6. Visual Studio Code でファイルを再び保存しますが、今回は保存先として **C:\\templates**、ファイル名として **storage.bicep** を選びます。

   > **注**: これで 2 つの同一の JSON ファイルができました: **C:\\templates\\main.bicep** と **C:\\templates\\storage.bicep**。

#### タスク 2: ストレージ リソース用のテンプレート モジュールを作成する

このタスクでは、前のタスクで保存したテンプレートを変更して、ストレージ テンプレート モジュール **storage.bicep** によってストレージ アカウントのみが作成され、それが最初のテンプレートによってインポートされるようにします。 ストレージ テンプレート モジュールは、メイン テンプレート **main.bicep** に値を戻す必要があります。この値は、ストレージ テンプレート モジュールの出力要素で定義されます。

1. Visual Studio Code ウィンドウに表示されている **storage.bicep** ファイルの**リソース セクション**で、**storageAccounts** リソース以外のすべてのリソース要素を削除します。 これによりリソース セクションは以下のようになるはずです。

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

2. 次に、すべての変数定義を削除します。

   ```bicep
   var storageAccountName = 'bootdiags${uniqueString(resourceGroup().id)}'
   var nicName = 'myVMNic'
   var addressPrefix = '10.0.0.0/16'
   var subnetName = 'Subnet'
   var subnetPrefix = '10.0.0.0/24'
   var virtualNetworkName = 'MyVNET'
   var networkSecurityGroupName = 'default-NSG'
   var securityProfileJson = {
     uefiSettings: {
       secureBootEnabled: true
       vTpmEnabled: true
     }
     securityType: securityType
   }
   var extensionName = 'GuestAttestation'
   var extensionPublisher = 'Microsoft.Azure.Security.WindowsAttestation'
   var extensionVersion = '1.0'
   var maaTenantName = 'GuestAttestation'
   var maaEndpoint = substring('emptyString', 0, 0)
   ```

3. 次に、場所以外のあらゆるパラメーター値を削除し、以下のパラメーター コードを追加すると、次のような結果になります:

   ```bicep
   @description('Location for all resources.')
   param location string = resourceGroup().location

   @description('Name for the storage account.')
   param storageAccountName string
   ```

4. 次に、ファイルの最後で、現在の出力を削除し、storageURI 出力値と呼ばれる新しい出力を追加します。 以下のようになるように出力を変更します。

   ```bicep
   output storageURI string = storageAccount.properties.primaryEndpoints.blob
   ```

5. storage.bicep テンプレート モジュールを保存します。 ストレージ テンプレートは次のようになります。

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

#### タスク 3: テンプレート モジュールを使うようにメイン テンプレートを変更する

このタスクでは、前のタスクで作成したテンプレート モジュールを参照するように、メイン テンプレートを変更します。

1. Visual Studio Code で **[ファイル]** トップ レベル メニューをクリックし、ドロップダウン メニューで **[ファイルを開く]** を選び、[ファイルを開く] ダイアログ ボックスで **C:\\templates\\main.bicep** に移動してそれを選び、 **[開く]** をクリックします。
2. **main.bicep** ファイルのリソース セクションで、ストレージ リソース要素を削除します

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

3. 次に、新しく削除されたストレージ リソース要素があった場所に直接、以下のコードを追加します:

   ```bicep
   module storageModule './storage.bicep' = {
     name: 'linkedTemplate'
     params: {
       location: location
       storageAccountName: storageAccountName
     }
   }
   ```

4. また、代わりにモジュールの出力を使うように、仮想マシン リソースのストレージ アカウント BLOB URI への参照を変更する必要があります。 仮想マシン リソースを見つけて、diagnosticsProfile セクションを次のように置き換えます。

   ```bicep
   diagnosticsProfile: {
     bootDiagnostics: {
       enabled: true
       storageUri: storageModule.outputs.storageURI
     }
   }
   ```

5. メイン テンプレートで以下の詳細をレビューします:

   - メイン テンプレートのモジュールは、別のテンプレートにリンクするために使われます。
   - モジュールのシンボル名は storageModule です。 依存関係を構成する場合はこの名前を使います。
   - テンプレート モジュールを使うときは、増分デプロイ モードのみを使用できます。
   - テンプレート モジュールには相対パスを使います。
   - パラメーターを使って、メイン テンプレートからテンプレート モジュールに値を渡します。

   > **注**: Azure ARM テンプレートでは、ストレージ アカウントを使ってリンクされたテンプレートをアップロードし、他のユーザーが簡単に使用できるようしました。 Azure Bicep モジュールでは、パブリックとプライベート両方のレジストリ オプションを備えた Azure Bicep モジュール レジストリに、モジュールをアップロードするオプションがあります。 詳しくは、[Azure Bicep に関するドキュメント](https://learn.microsoft.com/azure/azure-resource-manager/bicep/modules)をご覧ください。

6. テンプレートを保存します。

#### タスク 4: テンプレート モジュールを使用して Azure にリソースをデプロイする

> **注**: テンプレートのデプロイは、ローカル環境にインストールされた Azure CLI の使用、Azure Cloud Shell、CI/CD パイプラインなど、複数の方法で行うことができます。 このラボでは、Azure Cloud Shell から Azure CLI を使用します。

> **注**: ARM テンプレートとは対照的に、Azure portal を使って Bicep テンプレートを直接デプロイすることはできません。

> **注**: Azure Cloud Shell を使うには、main.bicep と storage.bicep の両方のファイルを、Cloud Shell のホーム ディレクトリにアップロードします。

> **注**: 現在、Azure CLI ではリモート Bicep ファイルのデプロイはサポートされていません。 Bicep ファイルをビルドして ARM テンプレートの JSON を取得し、それをストレージ アカウントにアップロードしてから、リモートでデプロイできます。

1. ラボのコンピューターで、Azure Portal が表示されている Web ブラウザーで **[Cloud Shell]** アイコンをクリックして Cloud Shell を開きます。
   > **注**: この演習で前に使った PowerShell セッションがまだアクティブなままの場合は、Bash に切り替えます (次のステップ)。
2. Cloud Shell ペインで **[PowerShell]** をクリックします。ドロップダウン メニューで **[バッシュ]** をクリックし、指示されたら **[確認]** をクリックします。
3. Cloud Shell ペインで、 **[ファイルのアップロード / ダウンロード]** アイコンをクリックし、ドロップダウン メニューで **[アップロード]** をクリックします。
4. **[開く]** ダイアログ ボックスで、**C:\\templates\\main.bicep** に移動して選び、 **[開く]** をクリックします。
5. 同じ手順で、**C:\\templates\\storage.bicep** ファイルもアップロードします。
6. Cloud Shell ペインの **Bash** セッションから以下を実行し、新しくアップロードされたテンプレートを使用してデプロイを実行します。

   ```bash
   LOCATION='<region>'
   ```

   > **注**: リージョンの名前を、自分の場所に近いリージョンに置き換えます。 使用できる場所がわからない場合は、`az account list-locations -o table` コマンドを実行してください。

   ```bash
   az group create --name az400m06l15-RG --location $LOCATION
   ```

   ```bash
   az deployment group what-if --name az400m06l15deployment --resource-group az400m06l15-RG --template-file main.bicep
   ```

7. 'adminUsername' の値を提供するよう指示されたら、「**Student**」と入力して **Enter** キーを押します。
8. 'adminPassword' の値を提供するよう指示されたら、「**Pa55w.rd1234**」と入力して **Enter** キーを押します。 (パスワードの入力は表示されません)
9. このコマンドの結果を調べると、デプロイを検証し、テンプレートにエラーがあるかどうかを確認できます。 これは特に、多くのリソースを含むテンプレートをビジネス クリティカルなクラウド環境でデプロイする場合に非常に便利です。

10. Cloud Shell ペインの **Bash** セッションから以下を実行し、新しくアップロードされたテンプレートを使用してデプロイを実行します。

    ```bash
    az deployment group create --name az400m06l15deployment --resource-group az400m06l15-RG --template-file main.bicep
    ```

11. 'adminUsername' の値を提供するよう指示されたら、「**Student**」と入力して **Enter** キーを押します。
12. 'adminPassword' の値を提供するよう指示されたら、「**Pa55w.rd1234**」と入力して **Enter** キーを押します。 (パスワードの入力は表示されません)

13. テンプレートをデプロイする上記のコマンドの実行時にエラーが出た場合は以下を試してください。

- 複数の Azure サブスクリプションがある場合は、リソース グループがデプロイされている適切な場所にサブスクリプションのコンテキストが設定されていることを確認してください。
- 指定した URI を介してリンク済みテンプレートにアクセスできることを確認します。

> **注**:次のステップでは、メイン デプロイ テンプレートで残りのリソース定義 (ネットワークや仮想マシンのリソース定義など) をモジュラー化できます。

> **注**:デプロイされたリソースを使用する予定がない場合は、関連した料金が発生しないようにリソースを削除してください。 リソース グループ **az400m06l15-RG** を削除するだけで、簡単にそれを行うことができます。

### 演習 2:Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。

> **注**:新規に作成し、使用しなくなったすべての Azure リソースを削除することを忘れないでください。 使用していないリソースを削除することで、予期しない料金が発生しなくなります。

#### タスク 1:Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。

1. Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
2. 次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

   ```bash
   az group list --query "[?starts_with(name,'az400m06l15-RG')].name" --output tsv
   ```

3. 次のコマンドを実行して、このモジュールのラボ全体を通して作成したすべてのリソース グループを削除します。

   ```bash
   az group list --query "[?starts_with(name,'az400m06l15-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

   > **注**:コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## 確認

このラボでは、Azure Resource Manager テンプレートを作成し、リンク済みテンプレートを使用してこれをモジュラー化し、メイン デプロイ テンプレートを変更してリンク済みテンプレートと更新済みの依存関係を呼び出し、最後にテンプレートを Azure にデプロイする方法を学びました。
