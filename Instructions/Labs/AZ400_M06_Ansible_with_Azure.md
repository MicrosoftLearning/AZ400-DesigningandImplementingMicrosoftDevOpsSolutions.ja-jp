---
lab:
  title: ラボ 14:Ansible と Azure
  module: 'Module 06: Manage infrastructure as code using Azure, DSC, and third-party tools'
ms.openlocfilehash: dab37a62b873ca7949ae39adaa870b5db7f3e113
ms.sourcegitcommit: f72fcf5ee578f465b3495f3cf789b06c530e88a4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2022
ms.locfileid: "139262609"
---
# <a name="lab-14-ansible-with-azure"></a>ラボ 14:Ansible と Azure
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-overview"></a>ラボの概要

このラボでは、Ansible を使用して Azure リソースをデプロイ、構成、管理します。 

Ansible は宣言型構成管理ソフトウェアです。 プレイブックという形式で管理対象コンピューター向けに意図された構成の説明に依存します。 Ansible は自動的にその構成を適用し、その後はこれを維持して、潜在的な不一致に対応します。 プレイブックは通常、YAML を使用して書式化されます。

Puppet や Chef といった他の大半の構成管理ツールとは異なり、Ansible はエージェントレスなので、管理対象マシンでソフトウェアをインストールする必要はありません。 Ansible は SSH を使用して Linux サーバーを管理し、Powershell Remoting を使用して Windows サーバーとクライアントを管理します。

オペレーティング システム以外のリソース (たとえば、Azure Resource Manager を介してアクセスできる Azure リソース) を操作するため、Ansible はモジュールと呼ばれる拡張機能に対応します。 Ansible は Python で書き込まれるため、実質的にモジュールは Python ライブラリとして実装されます。 Ansible は [GitHub ホステッド モジュール](https://github.com/ansible-collections/azure) を利用して、Azure リソースを管理します。

Ansible では、管理対象リソースをホスト インベントリで指定する必要があります。 Ansible は、Azure など一部のシステムで動的インベントリに対応するため、ホスト インベントリはランタイムで動的に生成されます。

ラボは以下の高レベルの手順で構成されます。

- Ansible を Azure VM にインストールして構成する
- Ansible の構成とサンプル プレイブック ファイルをダウンロードする
- Azure AD でマネージド ID を作成して構成する
- Ansible で使用できるように Azure AD 資格情報と SSH を構成する
- Ansible プレイブックを使用して Azure VM をデプロイする
- Ansible プレイブックを使用して Azure VM を構成する

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- Ansible を Azure VM にインストールして構成する
- Ansible の構成とサンプル プレイブック ファイルをダウンロードする
- Azure Active Directory のマネージド ID を作成して構成する
- Ansible で使用できるように Azure AD 資格情報と SSH を構成する
- Ansible プレイブックを使用して Azure VM をデプロイする
- Ansible プレイブックを使用して Azure VM を構成する

## <a name="lab-duration"></a>ラボの所要時間

-   推定時間:**90 分**

## <a name="instructions"></a>Instructions

### <a name="before-you-start"></a>開始する前に

#### <a name="sign-in-to-the-lab-virtual-machine"></a>ラボ仮想マシンにサインインする

必ず次の資格情報を使用して Windows 10 コンピューターにサインインしてください。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### <a name="review-applications-required-for-this-lab"></a>このラボで必要なアプリケーションのレビュー

このラボで使用するアプリケーションを特定:
    
-   Microsoft Edge

#### <a name="prepare-an-azure-subscription"></a>Azure サブスクリプションの準備

-   既存の Azure サブスクリプションを識別するか、新しいものを作成します。
-   Azure サブスクリプションでは所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントではグローバル管理者のロールで Microsoft アカウントまたは Azure AD アカウントがあることを確認します。 詳細については、[「Azure portal を使用して Azure ロールの割り当てを一覧表示する」](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal)および[「Azure Active Directory で管理者ロールを表示して割当てる」](https://docs.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal#view-my-roles)を参照してください。

### <a name="exercise-1-deploy-configure-and-manage-azure-vms-by-using-ansible"></a>演習 1:Ansible を使用して Azure VM をデプロイ、構成、管理する

この演習では、Ansible を使用して Azure リソースをデプロイ、構成、管理します。

#### <a name="task-1-provision-an-azure-vm-serving-as-the-ansible-control-node"></a>タスク 1:Ansible コントロール ノードとして機能する Azure VM のプロビジョニングを行う

このタスクでは、Azure CLI を使用してAzure VM をデプロイし、Ansible 環境を管理する Ansible コントロール ノードとして構成します。

>**注**:Ansible コントロール ノードとして構成された Azure VM を使用して、このラボの前のタスクで実行したタスクを含む、Ansible 管理タスクを実行します。 

1.  Azure portal のツールバーで、検索テキスト ボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。 

    >**注**:または、[https://shell.azure.com](https://shell.azure.com) に移動して Cloud Shell に直接アクセスすることもできます。

1.  **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、**[Bash]** を選択します。 

    >**注**:**Cloud Shell** を初めて起動し、「**ストレージがマウントされていません**」というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、 **[ストレージの作成]** を選択します。 

1.  Cloud Shell ペインの Bash セッションから、以下を実行して、このラボでデプロイするリソースをホストする Azure リージョンの名前を指定します (`<Azure_region>` プレースホルダーを、リソースをデプロイする予定の Azure リージョンの名前に置き換えます。 名前にスペースが含まれていないことを確認してください (例: `westeurope`)。

    ```bash
    LOCATION=<Azure_region>
    ```

1.  Cloud Shell ペインの Bash セッションから、以下を実行して、このラボでデプロイする Azure VM をホストするリソースグループを作成します。

    ```bash
    RG1NAME=az400m14l03rg
    az group create --name $RG1NAME --location $LOCATION
    RG2NAME=az400m14l03arg
    az group create --name $RG2NAME --location $LOCATION
    ```

1.  以下を実行し、Ubuntu を実行している最初の Azure VM を、前のステップで作成したリソース グループにデプロイします:

    ```bash
    VM1NAME=az400m1403vm1
    az vm create \
    --resource-group $RG1NAME \
    --name $VM1NAME \
    --image UbuntuLTS \
    --authentication-type password \
    --admin-username azureuser \
    --admin-password Pa55w.rd1234
    ```

    >**注**:デプロイが完了するのを待ってから、次の手順に進みます。 これには 2 分ほどかかる場合があります。

    >**注**:プロビジョニングの完了後、JSON ベースの出力で、出力に含まれている **"publicIpAddress"** プロパティの値を識別します。 

1.  以下を実行して、SSH を使用して新しくデプロイされた Azure VM に接続します。

    ```bash
    PIP=$(az vm show --show-details --resource-group $RG1NAME --name $VM1NAME --query publicIps --output tsv)
    ssh azureuser@$PIP
    ```

1.  確認して続行するよう指示されたら、「**yes**」と入力して **Enter** キーを押します。パスワードを入力するよう指示されたら「**Pa55w.rd1234**」と入力します。


#### <a name="task-2-install-and-configure-ansible-on-an-azure-vm"></a>タスク 2:Ansible を Azure VM にインストールして構成する

このタスクでは、前のタスクでデプロイした Azure VM で Ansible をインストールして構成します。

1.  Cloud Shell ペインのバッシュ セッションで、新しくデプロイされた Azure VM へのSSH セッション内で以下を実行し、最新バージョンとパッケージの詳細が含まれるように Advanced Packaging Tool (apt) パッケージのリストを更新します:

    ```bash
    sudo apt-get update
    ```
1.  次のコマンドを実行し、VSCode を追加してインストールします (確認を求められたら、**y** と入力し、**Enter** キーを押します)。
    ```
    sudo apt-get install wget
    ```
    ```
    wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
    ```
    ```
    sudo install -o root -g root -m 644 packages.microsoft.gpg /etc/apt/trusted.gpg.d/
    sudo sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/trusted.gpg.d/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'
    ```
    ```
    rm -f packages.microsoft.gpg
    ```
    ```
    sudo apt update
    sudo apt install apt-transport-https
    sudo apt install code
    ```

1.  以下を実行して、Ansible と必要な Azure モジュールをインストールします (**コマンドを 1 行ずつ個別に実行し**、確認を求められたら、**y** と入力して **Enter** キーを押します)。

    ```bash
    sudo apt install python3-pip
    sudo -H pip3 install --upgrade pip
    sudo -H pip3 install ansible[azure]
    sudo apt-add-repository --yes --update ppa:ansible/ansible
    sudo apt install ansible
    sudo ansible-galaxy collection install azure.azcollection
    curl -O https://raw.githubusercontent.com/ansible-collections/azure/dev/requirements-azure.txt
    sudo pip3 install -r requirements-azure.txt
    rm requirements-azure.txt
    ```

    >**注**:警告は無視してください。 エラーが発生した場合は、コマンドを再実行してください。

1.  以下を実行して dnspython パッケージをインストールし、Ansible プレイブックがデプロイ前に DNS 名を確認できるようにします。

    ```bash
    sudo -H pip3 install dnspython
    ```

1.  Cloud Shell ペインのバッシュ セッションで、新しくデプロイされた Azure VM へのSSH セッション内で以下を実行し、**jq** JSON 解析ツールをインストールします (プロンプトが表示されたら、**y** と入力し、**Enter** キーを押します):

    ```bash
    sudo apt install jq
    ```

1.  以下を実行して、Azure CLI をインストールしてください。

    ```bash
    curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
    ```

#### <a name="task-3-download-ansible-configuration-and-sample-playbook-files"></a>タスク 3:Ansible の構成とサンプル プレイブック ファイルをダウンロードする

このタスクでは、サンプル ラボ ファイルとともに GitHub Ansible 構成リポジトリからダウンロードします。 

1.  Cloud Shell ペインのバッシュ セッションで、新しくデプロイされた Azure VM へのSSH セッション内で以下を実行し、**git** がインストールされていることを確認します:

    ```bash
    sudo apt install git
    ```

1.  以下を実行して、GitHub から PartsUnlimitedMRP リポジトリを複製します。

    ```bash
    git clone https://github.com/Microsoft/PartsUnlimitedMRP.git
    ```

    >**注**:このリポジトリには、広範なリソースを作成するためのプレイブックが含まれており、その一部をこのラボで使用します。


#### <a name="task-4-create-and-configure-azure-active-directory-managed-identity"></a>タスク 4:Azure Active Directory のマネージド ID を作成して構成する

このタスクでは、Azure AD マネージド ID を生成して、Azure リソースへのアクセスに必要な Ansible の非対話型認証を促します。 また、前のタスクで作成したリソース グループでマネージド ID に共同作成者のロールを割り当てます。

1.  Cloud Shell ペインのバッシュ セッションで、新しくデプロイされた Azure VM への SSH セッション内で以下を実行し、お使いになっている Azure サブスクリプションに関連付けられている Azure AD テナントにサインインします:

    ```bash
    az login
    ```

    >**注**:コマンドが失敗した場合は、Azure CLI のインストールを再実行してください。

1.  以前のコマンドの出力として表示されているコードを確認し、ラボのコンピューターに切り替えます。 ラボのコンピューターから Azure portal が表示されているブラウザー ウィンドウで別のタブを開き、[Microsoft デバイス ログイン ページ](https://microsoft.com/devicelogin) に移動します。指示されたらコードを入力し、「**次へ**」を選択します。

1.  指示されたら、このラボで使用している資格情報を使ってサインインし、ブラウザー タブを閉じます。

1.  Cloud Shell で Bash セッションに切り替えます。 Ansible コントロール ノードとして構成された Azure VM への SSH セッション内で、以下を実行して、システムに割り当てられたマネージ ID を生成します。


    ```bash
    RG1NAME=az400m14l03rg
    VM1NAME=az400m1403vm1
    az vm identity assign --resource-group $RG1NAME --name $VM1NAME
    ```

1.  次のコマンドを実行して、サブスクリプションの値を確認します。

    ```bash
    SUBSCRIPTIONID=$(az account show --query id --output tsv)
    ```

1.  次のコマンドを実行して、組み込みの Azure のロールベースのアクセス制御共同作成者ロールの **ID** プロパティの値を取得します。 

    ```bash
    CONTRIBUTORID=$(az role definition list --name "Contributor" --query "[].id" --output tsv)
    ```

1.  以下を実行して、このラボで以前に作成したリソース グループに共同作成者のロールを割り当てます。 

    ```bash
    MIID=$(az resource list --name $VM1NAME --query [*].identity.principalId --out tsv)

    RG2NAME=az400m14l03arg
    az role assignment create --assignee "$MIID" \
    --role "$CONTRIBUTORID" \
    --scope /subscriptions/$SUBSCRIPTIONID/resourceGroups/$RG2NAME
    ```

#### <a name="task-5-configure-ssh-for-use-with-ansible"></a>タスク 5:Ansible で使用するために SSH を構成する

このタスクでは、Ansible で使用するために SSH を構成します。

1.  Cloud Shell ペインの Bash セッションで、新しくデプロイされた Azure VM への SSH セッション内で、次のコマンドを実行してキー ペアを生成します (プロンプトが表示されたら、**Enter** キーを 3 回押して、ファイルの場所のデフォルト値を受け入れます。パスフレーズを設定しないでください)。

    ```bash
    ssh-keygen -t rsa
    ```

1.  以下を実行して、秘密鍵をホストしている **.ssh** フォルダーに対する読み取り、書き込み、および実行のアクセス許可を付与します。

    ```bash
    chmod 755 ~/.ssh
    ```

1.  以下を実行して、**authorized_keys** ファイルの読み取りおよび書き込み権限を作成および設定します。

    ```bash
    touch ~/.ssh/authorized_keys
    chmod 644 ~/.ssh/authorized_keys
    ```

    >**注**:このファイルに含まれているキーを提供すると、パスワードがなくてもアクセスが許可されます。

1.  以下を実行して、**authorized_keys** ファイルにパスワードを追加します。

    ```bash
    ssh-copy-id azureuser@127.0.0.1
    ```

1. 指示されたら「**yes**」と入力し、このラボで以前に 3 番目の Azure VM をデプロイした際に指定した **azureuser** ユーザー アカウントのパスワード「**Pa55w.rd1234**」を入力します。 
1.  以下を実行して、パスワードの入力を求められていないことを確認します。

    ```bash
    ssh 127.0.0.1
    ```

1.  **「exit」** と入力し、**Enter** キーを押して、確立したループバック接続を終了します。 

>**注**:パスワードのない SSH 認証を設定することは、Ansible 環境を設定するための重要なステップです。 


#### <a name="task-6-create-a-web-server-azure-vm-by-using-an-ansible-playbook"></a>タスク 6:Ansible プレイブックを使用して Web サーバー Azure VM を作成する

このタスクでは、Ansible プレイブックを使用して Web サーバーをホストする Azure VM を作成します。 

>**注**:Ansible がコントロール Azure VM で実行中になったので、最初のプレイブックをデプロイして管理対象 Azure VM を作成して構成することができます。 サンプル プレイブックをデプロイするっ前に、コンテンツに含まれているパブリック SSH キーを、以前のタスクで生成したキーに置き換える必要があります。 

1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール ノードとして構成されている Azure VM への SSH セッション内で以下を実行し、以前のタスクで生成され、ローカルに保管されているパブリック キーを識別します:

    ```bash
    cat ~/.ssh/id_rsa.pub
    ```

1.  出力を記録します (出力文字列の最後のユーザー名を含む)。 
1.  以下を実行して、Code テキスト エディターで **new_vm_web.yml** ファイルを開きます。

    ```bash
    code ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_vm_web.yml
    ```

1.  必要に応じて、Code エディターで `dnsname: '{{ vmname }}.westeurope.cloudapp.azure.com'` エントリのリージョン名をデプロイのターゲットである Azure リージョン名に変更します。 

    >**注**:このリージョンが、**az400m14l03rg** リソース グループを作成した Azure リージョンと一致していることを確認してください。

1.  Code エディターで、`vm_size` エントリの値を `Standard_A0` から `Standard_DS1_v2` に変更します。
1.  Code エディターで、ファイルの最後の方にある SSH 文字列を見つけ (`key_data` エントリ)、既存のキー値を削除して、このタスクで以前に記録しておいたキー値に置き換えます。 

    >**注**:ファイルに含まれている `admin_username` エントリの値が、Ansible コントロール システムをホストしている Azure VM へのサインインで使われたユーザー名に一致することを確認します (**azureuser**)。 `ssh_public_keys` セクションの `path` エントリで同じユーザー名を使う必要があります。

1.  Code エディター インターフェイス内で、右上隅の **[...]** をクリックし、 **[保存]** を選びます。

    >**注**:次に、ラボの最初に作成したリソース グループに Azure VM をデプロイします。 デプロイでは以下の値を使用してください: 

    | 設定 | 値 |
    | --- | --- |
    | Resource group | **az400m14l03arg** |
    | 仮想ネットワーク | **az400m1403aVNET** |
    | Subnet | **az400m1403aSubnet** |

    >**注**:変数は、プレイブック内で定義できます。また、`ansible-playbook` コマンドを呼び出すときに `--extra-vars` オプションを含めることで、実行時に入力することもできます。 VM 名には、最高 15 字の小文字と数字のみを使用できます (ハイフン、下線、大文字は使用不可)。同じ名前を使用して、該当する Azure VM に関連のあるパブリック IP アドレスのストレージ アカウントと DNS 名を生成するため、グローバルに一意であることを確認してください。 

1.  以下を実行して、Ansible プレイブックを使用して Azure VM をデプロイする仮想ネットワークとそのサブネットを作成します。

    ```bash
    RG1NAME=az400m14l03arg
    LOCATION=$(az group show --resource-group $RG1NAME --query location --output tsv)
    RG2NAME=az400m14l03arg
    VNETNAME=az400m1403aVNET
    SUBNETNAME=az400m1403aSubnet
    az network vnet create \
    --name $VNETNAME \
    --resource-group $RG2NAME \
    --location $LOCATION \
    --address-prefixes 192.168.0.0/16 \
    --subnet-name $SUBNETNAME \
    --subnet-prefix 192.168.1.0/24
    ```

1.  次のコマンドを実行して、Azure VM をプロビジョニングするサンプルの Ansible プレイブックをデプロイします (**必ず `<VM_name>` を選択した一意の VM 名に置き換えてください**)。

    ```bash
    sudo ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_vm_web.yml --extra-vars "vmname=<VM_name> resgrp=az400m14l03arg vnet=az400m1403aVNET subnet=az400m1403aSubnet"
    ```

    >**注**:ip_configuration の設定に関する非推奨の警告は無視してください。

    >**注**:既存または無効な VM 名を入力すると、以下のエラーが返される可能性があります:

    - `fatal: [localhost]: FAILED! => {"changed": false, "failed": true, "msg": "The storage account named storageaccountname is already taken. - Reason.already_exists"}`. これを解決するには、Azure VM の別の名前を使ってください。使用された名前はグローバルに一意ではありません。
    - `fatal: [localhost]: FAILED! => {"changed": false, "failed": true, "msg": "Error creating or updating your-vm-name - Azure Error: InvalidDomainNameLabel\nMessage: The domain name label for your VM is invalid. It must conform to the following regular expression: ^[a-z][a-z0-9-]{1,61}[a-z0-9]$.”}`. この問題を解決するには、必要な命名規則に従って、Azure VM に別の名前を使用してください。 

    >**注**: デプロイが完了するまで待ちます。 これには 3 分ほどかかる場合があります。 

1.  以下を実行し、「**myazure_rm.yml**」という名前の新しいファイルを作成し、Code テキスト エディターでこれを開きます。

    ```bash
    code ./myazure_rm.yml
    ```

1.  Code エディターのインターフェイス内で以下の内容を貼り付けます。

    ```bash
    plugin: azure_rm
    include_vm_resource_groups:
    - az400m14l03arg
    auth_source: msi

    keyed_groups:
    - prefix: tag
      key: tags
    ```

1.  Code エディター インターフェイス内で、右上隅の **[...]** をクリックし、 **[保存]** を選びます。
1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール ノードとして構成されている Azure VM への SSH セッション内で以下を実行し、ping テストを行って、動的インベントリ ファイルに新しくデプロイされた Azure VM が含まれていることを確認します:

    ```bash
    sudo ansible --user azureuser --private-key=/home/azureuser/.ssh/id_rsa all -m ping -i ./myazure_rm.yml
    ```

1.  接続を続行するかどうかを確認するメッセージが表示されたら、 **「yes」** と入力し、**Enter** キーを押します。

    >**注**:出力は次のようになります。

    ```bash
    az400m1403vm2_5444 | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python&quot;: &quot;/usr/bin/python"
        },
        "changed": false,
        "ping&quot;: &quot;pong"
    }
    ```

    >**注**:コマンドを初めて実行する際は、ターゲット VM の信頼性を認識する必要があります。「**yes**」と入力してから **Enter** キーを押してください。 


#### <a name="task-7-configure-an-azure-vm-by-using-an-ansible-playbook"></a>タスク 7:Ansible プレイブックを使用して Azure VM を構成する

このタスクでは、別の Ansible プレイブックを実行し、新しくデプロイされた Azure VM を構成します。 ソフトウェア パッケージ httpd をインストールし、HTML ページを GitHub リポジトリからダウンロードするプレイブックを使用します。 これが完了すると、Web サーバーは完全に機能するようになります。

>**注**:サンプル プレイブック「 **~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/httpd.yml**」を使用します。 プレイブックのホスト パラメーターを変更するために **vmname** 変数を使用します。これは、プレイブックが (動的インベントリのスクリプトから返されたホストのうち) どのホストをターゲットにするのかを定義するものです。 

1.  Cloud Shell ペインの Bash セッションで、Ansible コントロール ノードとして構成された Azure VM への SSH セッション内で、以下を実行して、新しくデプロイされた Azure VM のパブリック IP アドレスを特定します (**必ず `<VM_name>` プレースホルダーを新しくプロビジョニングされた Azure VM に割り当てた名前に置き換えてください**)。

    ```bash
    RGNAME='az400m14l03arg'
    VMNAME='<VM_name>'
    PIP=$(az vm show --show-details --resource-group $RGNAME --name $VMNAME --query publicIps --output tsv)
    ```

1.  次の手順を実行して、新しくデプロイされた Azure VM が現在 Web サービスを実行していないことを確認します (この `<IP_address>` プレースホルダーは、前のタスクでプロビジョニングした Azure VM のネットワーク アダプターに割り当てられたパブリック IP アドレスを表します)。

    ```bash
    curl http://$PIP
    ```

    >**注**:応答の形式が `curl: (7) Failed to connect to 52.186.157.26 port 80: Connection refused` であることを確認してください。

1.  以下を実行して、Ansible プレイブックを使用して HTTP サービスをインストールします (この `<VM_name>` プレースホルダーは前のタスクでプロビジョニングした VM の名前を表します)。 

    ```bash
    sudo ansible-playbook --user azureuser --private-key=/home/azureuser/.ssh/id_rsa -i ./myazure_rm.yml ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/httpd.yml --extra-vars "vmname=<VM_name>*"
    ```

    >**注**:Azure VM 名の後に必ず末尾のアスタリスク ( **\*** ) を含めてください。

    >**注**: インストールが完了するまで待ちます。 これに要する時間は 1 分未満です。 

1.  インストールが完了したら、以下を実行して、新しくデプロイされた Azure VM が Web サービスを実行していることを確認します (この `<IP_address>` プレースホルダーは、前のタスクでプロビジョニングした Azure VM のネットワーク アダプターに割り当てられたパブリック IP アドレスを表します)。

    ```bash
    curl http://$PIP
    ```

    >**注**:出力には以下の内容が必要です: 

    ```html
     <!DOCTYPE html>
     <html lang="en">
         <head>
             <meta charset="utf-8">
             <title>Hello World</title>
         </head>
         <body>
             <h1>Hello World</h1>
             <p>
                 <br>This is a test page
                 <br>This is a test page
                 <br>This is a test page
             </p>
         </body>
     </html>
    ```

### <a name="exercise-2-remove-the-azure-lab-resources"></a>演習 2:Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

>**注**:新規に作成し、使用しなくなったすべての Azure リソースを削除することを忘れないでください。 使用していないリソースを削除することで、予期しない料金が発生しなくなります。

#### <a name="task-1-remove-the-azure-lab-resources"></a>タスク 1:Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'az400m14l03')].name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体を通して作成したすべてのリソース グループを削除します。

    ```sh
    az group list --query "[?starts_with(name,'az400m14l03')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注**:コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## <a name="review"></a>確認

このラボでは、Ansible を使用して Azure リソースをデプロイ、構成、管理する方法を学習しました。 

