---
lab:
  title: ラボ 16:Resource Manager テンプレートを使用した Azure デプロイ
  module: 'Module 06: Manage infrastructure as code using Azure, DSC, and third-party tools'
ms.openlocfilehash: cf3f2907ec5e7ddc6f0742e92328d4003cb44500
ms.sourcegitcommit: f72fcf5ee578f465b3495f3cf789b06c530e88a4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2022
ms.locfileid: "139262538"
---
# <a name="lab-16-azure-deployments-using-resource-manager-templates"></a>ラボ 16:Resource Manager テンプレートを使用した Azure デプロイ
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-overview"></a>ラボの概要

このラボでは、Azure Resource Manager (ARM) テンプレートを作成し、リンクされたテンプレートのコンセプトを使ってこれをモジュラー化します。 その後、主要なデプロイ テンプレートを修正し、リンクされたテンプレートと更新された依存関係を呼び出し、最終的にテンプレートを Azure にデプロイします。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- Resource Manager テンプレートの作成
- ストレージ リソース向けのリンク済みテンプレートを作成する
- リンク済みテンプレートを Azure Blob Storage にアップロードして SAS トークンを生成する
- メイン テンプレートを変更して、リンク済みテンプレートを呼び出す
- メイン テンプレートを変更して依存関係を更新する
- リンク済みテンプレートを使用してリソースを Azure にデプロイする

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
-   [Visual Studio Code](https://code.visualstudio.com/)。 このラボでは前提条件の一部としてインストールされます。 

#### <a name="prepare-an-azure-subscription"></a>Azure サブスクリプションの準備

-   既存の Azure サブスクリプションを識別するか、新しいものを作成します。
-   Azure サブスクリプションでは所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントではグローバル管理者のロールで Microsoft アカウントまたは Azure AD アカウントがあることを確認します。

### <a name="exercise-0-configure-the-lab-prerequisites"></a>演習 0:ラボの前提条件の構成

この演習では、ラボの前提条件を設定します。この中には、Visual Studio Code が含まれます。

#### <a name="task-1-install-and-configure-git-and-visual-studio-code"></a>タスク 1:Git と Visual Studio Code をインストールして構成する

このタスクでは、Visual Studio Code をインストールします。 この前提条件をすでに実装している場合は、直接、次のタスクに進むことができます。

1.  まだ Visual Studio Code がインストールされていない場合は、ラボのコンピューターから Web ブラウザーを起動し、[Visual Studio Code ダウンロード ページ](https://code.visualstudio.com/) に移動し、これをダウンロードしてインストールします。 

### <a name="exercise-1-author-and-deploy-azure-resource-manager-templates"></a>演習 1:Azure Resource Manager テンプレートを作成してデプロイする

このラボでは、Azure Resource Manager テンプレートを作成し、リンクされたテンプレートを使ってこれをモジュラー化します。 その後、主要なデプロイ テンプレートを修正し、リンクされたテンプレートと更新された依存関係を呼び出し、最終的にテンプレートを Azure にデプロイします。

#### <a name="task-1-create-resource-manager-template"></a>タスク 1:Resource Manager テンプレートを作成する

このタスクでは、Visual Studio Code を使用して Resource Manager テンプレートを作成します

1.  ラボのコンピューターから Visual Studio Code を起動し、Visual Studio Code で **[ファイル]** トップ レベル メニューをクリックします。ドロップダウン メニューで **[基本設定]** を選択します。カスケード メニューで **[拡張機能]** を選択し、 **[拡張機能の選択]** テキストボックスに「**Azure Resource Manager (ARM) ツール**」と入力します。該当する検索結果を選択し、 **[インストール]** をクリックして Azure Resource Manager ツールをインストールします。
1.  Web ブラウザーで、 **https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vm-simple-windows/azuredeploy.json** に接続します。 ファイルの **[Raw]** オプションをクリックします。 コード ウィンドウの内容をコピーして、Visual Studio コード エディターに貼り付けます。

    > **注**:テンプレートを最初から作成するよりも、[Azure クイックスタート テンプレート](https://azure.microsoft.com/en-us/resources/templates/) のひとつ (**シンプルな Windows テンプレート VM のデプロイ**) を使用します。 テンプレートは GitHub - [vm-simple-windows](https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vm-simple-windows) からダウンロードできます。

1.  ラボのコンピューターでエクスプローラーを開き、テンプレートの格納で使用する以下のローカル フォルダーを作成します。

    - **C:\\templates** 
    - **C:\\templates\\storage** 

1.  azuredeploy.json テンプレートのある Visual Studio Code ウィンドウに戻り、 **[ファイル]** トップ レベル メニューをクリックします。ドロップダウン メニューで **[名前を付けて保存]** をクリックし、新しく作成されたローカル フォルダー **C:\\templates** でテンプレートを **azuredeploy.json** として保存します。
1.  テンプレートをレビューし、その構造をよりよく把握します。 テンプレートには 5 種類のリソースが含まれています。

    - Microsoft.Storage/storageAccounts
    - Microsoft.Network/publicIPAddresses
    - Microsoft.Network/virtualNetworks
    - Microsoft.Network/networkInterfaces
    - Microsoft.Compute/virtualMachines

1.  Visual Studio Code でファイルを再び保存しますが、今回は保存先として **C:\\templates\\storage**、ファイル名として **storage.json** を選択します。

    > **注**:これで 2 つの同一の JSON ファイルができました。**C:\\templates\\azuredeploy.json** と **C:\\templates\\storage\\storage.json** です。

#### <a name="task-2-create-a-linked-template-for-storage-resources"></a>タスク 2:ストレージ リソース向けのリンク済みテンプレートを作成する

このタスクでは、前のタスクで保存したテンプレートを変更し、リンク済みのストレージ テンプレート「**storage.json**」がストレージ アカウントのみを作成し、最初のテンプレートで実行が起動されるようにします。 リンク済みのストレージ テンプレートは、メイン テンプレート「**azuredeploy.json**」に値を戻す必要があります。この値は、リンク済みストレージ テンプレートの出力要素で定義されます。

1.  Visual Studio Code ウィンドウに表示されている **storage.json** ファイルの **[リソース セクション]** で、**storageAccounts** リソース以外のすべてのリソース要素を削除します。 これによりリソース セクションは以下のようになるはずです。

    ```json
    "resources": [
      {
        "type": "Microsoft.Storage/storageAccounts",
        "name": "[variables('storageAccountName')]",
        "location": "[parameters('location')]",
        "apiVersion": "2021-04-01",
        "sku": {
           "name": "Standard_LRS"
        },
        "kind": "Storage"
      }
    ],
    ```

1.  storageAccount の名前要素を変数からパラメーターに変更します

    ```json
    "resources": [
      {
        "type": "Microsoft.Storage/storageAccounts",
        "name": "[parameters('storageAccountName')]",
        "location": "[parameters('location')]",
        "apiVersion": "2021-04-01",
        "sku": {
           "name": "Standard_LRS"
        },
        "kind": "Storage",
        "properties": {}
      }
    ],
    ```

1.  次に、変数セクション全体とあらゆる変数の定義を削除します:

    ```json
    "variables": {
      "storageAccountName": "[concat('bootdiags', uniquestring(resourceGroup().id))]",
      "nicName": "myVMNic",
      "addressPrefix": "10.0.0.0/16",
      "subnetName": "Subnet",
      "subnetPrefix": "10.0.0.0/24",
      "virtualNetworkName": "MyVNET",
      "networkSecurityGroupName": "default-NSG"
    },
    ```

1.  次に、場所以外のあらゆるパラメーター値を削除し、以下のパラメーター コードを追加すると、次のような結果になります:

    ```json
    "parameters": {
      "location": {
        "type": "string",
        "defaultValue": "[resourceGroup().location]",
        "metadata": {
          "description": "Location for all resources."
        }
      },
        "storageAccountName":{
        "type": "string",
        "metadata": {
          "description": "Azure Storage account name."
        }
      }
    },
    ```

1.  次に、出力セクションを更新して、storageURI 出力値を定義します。 メイン テンプレートの仮想マシン リソース定義では、storageUri 値が必要です。 この値を、出力値としてメイン テンプレートに値を渡し返します。 以下のようになるように出力を変更します。

    ```json
    "outputs": {
      "storageUri": {
        "type": "string",
        "value": "[reference(parameters('storageAccountName')).primaryEndpoints.blob]"
      }
    }
    ```

1. 最後に、スキーマのバージョンが 2019-04-01 であることを確認してください (VS Code に表示されている場合は警告/エラーを無視してください):

    ```json
        {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "storageAccountName":{
              "type": "string",
              "metadata": {
                "description": "Azure Storage account name."
              }
    ```

1. Storage.json テンプレートを保存します。 リンク済みストレージ テンプレートは次のようになります:

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "metadata": {
        "_generator": {
          "name": "bicep",
          "version": "0.4.1.14562",
          "templateHash": "8381960602397537918"
        }
      },
      "parameters": {
        "location": {
          "type": "string",
          "defaultValue": "[resourceGroup().location]",
          "metadata": {
            "description": "Location for all resources."
          }
        },
        "storageAccountName": {
          "type": "string",
          "metadata": {
            "description": "Azure Storage account name."
          }
        }
    
      },
      "functions": [],
      "variables": {
      },
      "resources": [
        {
          "type": "Microsoft.Storage/storageAccounts",
          "apiVersion": "2021-04-01",
          "name": "[parameters('storageAccountName')]",
          "location": "[parameters('location')]",
          "sku": {
            "name": "Standard_LRS"
          },
          "kind": "Storage"
        }
      ],
      "outputs": {
        "storageUri": {
          "type": "string",
          "value": "[reference(parameters('storageAccountName')).primaryEndpoints.blob]"
        }
      }
    }
    ```

#### <a name="task-3-upload-linked-template-to-azure-blob-storage-and-generate-sas-token"></a>タスク 3:リンク済みテンプレートを Azure Blob Storage にアップロードして SAS トークンを生成する

このタスクでは、前のタスクで作成したリンク済みテンプレートを Azure Blob Storage にアップロードし、SAS トークンを生成して、その後のデプロイ中のアクセスを提供します。

> **注**:テンプレートにリンクする際、Azure Resource Manager サービスは http または https のいずれかを介してこれにアクセスできなくてはなりません。 これを実行するため、リンク済みストレージ テンプレート「**storage.json**」を Azure の Blob ストレージにアップロードします。 その後、デジタル署名された URL を生成します。これは、該当する BLOB への限定的なアクセスを提供します。 これらのステップは、Azure Cloud Shell で Azure CLI を使用して実行します。 また、Azure Portal を介して BLOB コンテナーを手動で作成し、ファイルをアップロードして URL を生成するか、ラボのコンピューターにインストールされている Azure CLI または Azure PowerShell のいずれかのモジュールを使用します。

1.  ラボのコンピューターで、Web ブラウザーを起動し、[**Azure Portal**](https://portal.azure.com) に移動します。このラボで使用する Azure サブスクリプションで少なくとも共同作成者のロールがあるユーザー アカウントを使ってサインインします。 

1.  Azure portal のツールバーで、検索テキスト ボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。 

    > **注**:また、[Azure Cloud Shell](http://shell.azure.com) に直接移動することもできます。

1.  **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、 **[PowerShell]** を選択します。 

    >**注**:**Cloud Shell** を初めて起動し、「**ストレージがマウントされていません**」というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、 **[ストレージの作成]** を選択します。 

1.  Cloud Shell ペインの **PowerShell** セッションから、以下を実行して BLOB ストレージ コンテナーを作成し、前のタスクで作成したテンプレート ファイルをアップロードします。その後、メイン テンプレートで参照してリンク済みテンプレートにアクセスできるように SAS トークンを生成します。
1.  まず、以下のコードのラインをコピーして貼り付け、デプロイ先の Azure リージョンの値を設定します。 プロンプトに示されているように、コマンドは入力を待ちます。

    ```powershell
    # Provide the name of the closest Azure region in which you can provision Azure VMs
    $location = Read-Host -Prompt 'Enter the name of Azure region (i.e. centralus)'
    ```
1. 次に、次のコードをコピーして同じ Cloud Shell セッションに貼り付け、BLOB ストレージ コンテナーを作成します。

    ```powershell
    # This is a random string used to assign the name to the Azure storage account
    $suffix = Get-Random
    $resourceGroupName = 'az400m13l01-RG'
    $storageAccountName = 'az400m13blob' + $suffix

    # The name of the Blob container to be created
    $containerName = 'linktempblobcntr' 

    # A file name used for downloading and uploading the linked template
    $fileName = 'storage.json' 
    
    # Create a resource group
    New-AzResourceGroup -Name $resourceGroupName -Location $location 
    
    # Create a storage account
    $storageAccount = New-AzStorageAccount `
      -ResourceGroupName $resourceGroupName `
      -Name $storageAccountName `
      -Location $location `
      -SkuName 'Standard_LRS'
    
    $context = $storageAccount.Context
    
    # Create a container
    New-AzureStorageContainer -Name $containerName -Context $context
    ```
  1. Cloud Shell ペインで、 **[ファイルのアップロード / ダウンロード]** アイコンをクリックし、ドロップダウン メニューで **[アップロード]** をクリックします。 **[開く]** ダイアログ ボックスで、**C:\\templates\\storage\\storage.json** に移動してこれを選択し、 **[開く]** をクリックします。

      ```powershell
      # Upload the linked template
      Set-AzureStorageBlobContent `
        -Container $containerName `
        -File "$home/$fileName" `
        -Blob $fileName `
        -Context $context

      # Generate a SAS token. We set an expiry time of 24 hours, but you could have shorter values for increased security.
      $templateURI = New-AzureStorageBlobSASToken `
        -Context $context `
        -Container $containerName `
        -Blob $fileName `
        -Permission r `
        -ExpiryTime (Get-Date).AddHours(24.0) `
        -FullUri

      "Resource Group Name: $resourceGroupName"
      "Linked template URI with SAS token: $templateURI"
      ```

  >**注**:スクリプトで生成された最終的な出力を必ず記録してください。 これは、ラボの後半で必要になります。
  
  >**注**:出力値は以下のようになるはずです:

  ```
      Resource Group Name: az400m13l01-RG
      Linked template URI with SAS token: https://az400m13blob1677205310.blob.core.windows.net/linktempblobcntr/storage.json?sv=2018-03-28&sr=b&sig=B4hDLt9rFaWHZXToJlMwMjejAQGT7x0INdDR9bHBQnI%3D&se=2020-11-23T21%3A54%3A53Z&sp=r
  ```

  >**注**:セキュリティのレベルを強化する必要がある場合は、メイン テンプレートのデプロイ中に SAS トークンをダイナミックに生成し、より短い有効期間を SAS トークンに割り当てることができます。

1.  [Cloud Shell] ペインを閉じます。

#### <a name="task-4-modify-the-main-template-to-call-the-linked-template"></a>タスク 4:メイン テンプレートを変更して、リンク済みテンプレートを呼び出す

このタスクでは、メイン テンプレートを変更し、前のタスクで Azure Blob Storage にアップロードされたリンク済みテンプレートを参照します。

> **注**:あらゆるストレージ要素をモジュラー化してテンプレート構造に加えた変更を説明するため、メイン テンプレートを変更して新しいストレージ リソース定義を呼び出す必要があります。

1.  Visual Studio Code で **[ファイル]** トップ レベル メニューをクリックし、ドロップダウン メニューで **[ファイルを開く]** を選択します。[ファイルを開く] ダイアログ ボックスで **C:\\templates\\azuredeploy.json** に移動して選択し、 **[開く]** をクリックします。
1.  **Azuredeploy.json** ファイルのリソース セクションで、ストレージ リソース要素を削除します。

    ```json
    {
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[variables('storageAccountName')]",
      "location": "[parameters('location')]",
      "apiVersion": "2021-04-01",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage"
    },
    ```

1.  次に、新しく削除されたストレージ リソース要素があった場所に直接、以下のコードを追加します:

    > **注**:`<linked_template_URI_with_SAS_token>` プレースホルダーは、前のタスクの最後で記録した実際の値に必ず置き換えてください。

    ```json
    {
      "name": "linkedTemplate",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2018-05-01",
      "properties": {
          "mode": "Incremental",
          "templateLink": {
              "uri":"<linked_template_URI_with_SAS_token>"
          },
          "parameters": {
              "storageAccountName":{"value": "[variables('storageAccountName')]"},
              "location":{"value": "[parameters('location')]"}
          }
       }
    },
    ```

1.  メイン テンプレートで以下の詳細をレビューします:

    - メイン テンプレートの Microsoft.Resources/deployments リソースを使用して、別のテンプレートにリンクします。
    - デプロイ リソースには、linkedTemplate という名前が付いています。 この名前は、依存関係を構成する場合に使用されます。
    - リンクされたテンプレートを呼び出すときは、増分デプロイ モードのみを使用できます。
    - templateLink/uri には、リンク済みテンプレートの URI が含まれています。
    - パラメーターを使用して、メイン テンプレートからリンク済みテンプレートに値を渡します。

1.  テンプレートを保存します。


#### <a name="task-5-modify-main-template-to-update-dependencies"></a>タスク 5:メイン テンプレートを変更して依存関係を更新する

このタスクでは、メイン テンプレートを変更して、更新する必要のある残りの依存関係を説明します。

> **注**:ストレージ アカウントはリンク済みストレージ テンプレートで定義されているため、**Microsoft.Compute/virtualMachines** リソース定義を更新する必要があります。 

1.  仮想マシン要素のリソース セクションで、以下を置き換えることによって **dependsOn** 要素を更新します:

    ```json
    "dependsOn": [
      "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]",
      "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]"
    ]
    ```

    代入

    ```json
    "dependsOn": [
      
      "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]",
      "linkedTemplate"
    ]
      ```

1.  **Microsoft.Compute/virtualMachines** 要素のリソース セクションで、**properties/diagnosticsProfile/bootDiagnostics/storageUri** 要素を再構成し、以下を置き換えることによってリンク済みストレージ テンプレートで定義された出力値を反映させます:

    ```json
    "diagnosticsProfile": {
      "bootDiagnostics": {
        "enabled": true,
        "storageUri": "[reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))).primaryEndpoints.blob]"
      }
    ```

    代入

    ```json
    "diagnosticsProfile": {
      "bootDiagnostics": {
        "enabled": true,
        "storageUri": "[reference('linkedtemplate').outputs.storageUri.value]"
      }
    ```

1.  更新されたメイン デプロイ テンプレートを保存します。

#### <a name="task-6-deploy-resources-to-azure-by-using-linked-templates"></a>タスク 6:リンク済みテンプレートを使用してリソースを Azure にデプロイする

> **注**:テンプレートは複数の方法でデプロイできます。Azure Portal から直接デプロイしたり、ローカルで、または Azure Cloud Shell からインストールされた Azure CLI や PowerShell を使用したりできます。 このラボでは、Azure Cloud Shell から Azure CLI を使用します。  

> **注**:Azure Cloud Shell を使用するには、メイン デプロイ テンプレート「azuredeploy.json」を Cloud Shell のホーム ディレクトリにアップロードします。 また、リンク済みテンプレートをアップロードした場合と同様に、Azure Blob Storage にアップロードし、ローカル ファイルのシステム パスではなく URI を使用して参照することもできます。

1.  ラボのコンピューターで、Azure Portal が表示されている Web ブラウザーで **[Cloud Shell]** アイコンをクリックして Cloud Shell を開きます。 
    > **注**:この演習で以前に使用した PowerShell セッションがまだアクティブな場合は、Bash に切り替えなくてもこれを使用できます (次のステップ)。 Cloud Shell の PowerShell と Bash セッションの両方で以下のステップを実行できます。 新しい Cloud Shell セッションを開く場合は手順に従ってください。 
1.  Cloud Shell ペインで **[PowerShell]** をクリックします。ドロップダウン メニューで **[バッシュ]** をクリックし、指示されたら **[確認]** をクリックします。 
1.  Cloud Shell ペインで、 **[ファイルのアップロード / ダウンロード]** アイコンをクリックし、ドロップダウン メニューで **[アップロード]** をクリックします。 
1.  **[開く]** ダイアログ ボックスで、**C:\\templates\\azuredeploy.json** に移動してこれを選択し、 **[開く]** をクリックします。
1.  Cloud Shell ペインの **Bash** セッションから以下を実行し、新しくアップロードされたテンプレートを使用してデプロイを実行します。

    ```bash
    az deployment group create --name az400m13l01deployment --resource-group az400m13l01-RG --template-file azuredeploy.json
    ```

1.  'adminUsername’ の値を提供するよう指示されたら、「**Student**」と入力して **Enter** キーを押します。
1.  'adminPassword' の値を提供するよう指示されたら、「**Pa55w.rd1234**」と入力して **Enter** キーを押します。 (パスワードの入力は表示されません)

1.  テンプレートをデプロイする上記のコマンドの実行時にエラーが出た場合は以下を試してください。

    - 複数の Azure サブスクリプションがある場合は、リソース グループがデプロイされている適切な場所にサブスクリプションのコンテキストが設定されていることを確認してください。
    - 指定した URI を介してリンク済みテンプレートにアクセスできることを確認します。

> **注**:次のステップでは、メイン デプロイ テンプレートで残りのリソース定義 (ネットワークや仮想マシンのリソース定義など) をモジュラー化できます。 

> **注**:デプロイされたリソースを使用する予定がない場合は、関連した料金が発生しないようにリソースを削除してください。 リソース グループ「**az400m13l01-RG**」を削除するだけです。

### <a name="exercise-2-remove-the-azure-lab-resources"></a>演習 2:Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

>**注**:新規に作成し、使用しなくなったすべての Azure リソースを削除することを忘れないでください。 使用していないリソースを削除することで、予期しない料金が発生しなくなります。

#### <a name="task-1-remove-the-azure-lab-resources"></a>タスク 1:Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```bash
    az group list --query "[?starts_with(name,'az400m13l01-RG')].name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体を通して作成したすべてのリソース グループを削除します。

    ```bash
    az group list --query "[?starts_with(name,'az400m13l01-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注**:コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## <a name="review"></a>確認

このラボでは、Azure Resource Manager テンプレートを作成し、リンク済みテンプレートを使用してこれをモジュラー化し、メイン デプロイ テンプレートを変更してリンク済みテンプレートと更新済みの依存関係を呼び出し、最後にテンプレートを Azure にデプロイする方法を学びました。
