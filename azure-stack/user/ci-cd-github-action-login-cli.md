---
title: Azure Stack Hub 上で Azure CLI と PowerShell による Azure ログイン アクションを使用する
description: Azure CLI と PowerShell での Azure ログイン アクションを使用して、Azure Stack Hub 上に継続的インテグレーション/継続的デプロイ (CI/CD) ワークフローを作成する
author: mattbriggs
ms.topic: how-to
ms.date: 1/11/2021
ms.author: mabrigg
ms.reviewer: gara
ms.lastreviewed: 1/11/2021
ms.openlocfilehash: 1421917f870d09d61f665a2cee6eb9b617ae75f3
ms.sourcegitcommit: f194f9ca4297864500e62d8658674a0625b29d1d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102187352"
---
# <a name="use-the-azure-login-action-with-azure-cli-and-powershell-on-azure-stack-hub"></a>Azure Stack Hub 上で Azure CLI と PowerShell による Azure ログイン アクションを使用する

Azure Stack Hub インスタンスにサインインし、PowerShell を実行してから、Azure CLI スクリプトを実行するように、GitHub Actions を設定できます。 これは、Azure Stack Hub を使用したソリューションの継続的インテグレーション/継続的デプロイ (CI/CD) ワークフローの基盤として使用できます。 このワークフローを使用すると、ソリューションのビルド、テスト、およびデプロイを自動化でき、コードの記述に専念できるようになります。 たとえば、他のアクションを追加して、このワークフローを Azure Resource Manager テンプレートと共に使用すると、VM をプロビジョニングし、アプリケーション リポジトリを検証して、GitHub 内の特定のブランチにマージするたびに、その VM にアプリケーションをデプロイすることができます。 ここで、この記事によって GitHub Actions と Azure Stack Hub に順応できるようになります。

GitHub Actions は、コード リポジトリ内で自動化を有効にするアクションで構成されるワークフローです。 GitHub 開発プロセスにおいてイベントを含むワークフローをトリガーできます。 テスト、デプロイ、継続的インテグレーションなど、一般的な DevOps 自動化タスクを実行できます。

Azure Stack Hub で GitHub Actions を使用するには、特定の要件を持つサービス プリンシパル (SPN) を使用する必要があります。 この記事では、"*セルフホステッド ランナー*" を作成します。 GitHub では、GitHub がアクセスできる任意のコンピューターを GitHub Actions で使用できます。 Azure、Azure Stack Hub、またはその他の場所で、仮想マシン (VM) をランナーとして作成できます。

このワークフロー例には以下が含まれます。
- SPN の作成と検証の手順。
- Azure Stack Hub と一緒に動作する GitHub Actions セルフホステッド ランナーとしての Windows 2016 Server コア マシンの構成。
- 以下を使用するワークフロー:
    - Azure ログイン アクション
    - PowerShell スクリプト アクション

### <a name="azure-stack-hub-github-actions"></a>Azure Stack Hub の GitHub Actions

次の図は、さまざまな環境とそれらの関係を示しています。

![Azure Stack Hub の Github アクション](.\media\ci-cd-github-action-login-cli\ash-github-actions-v1d1.svg) セルフホステッド ランナーを使用する部分:

- GitHub でホストされている GitHub Actions
- Azure でホストされているセルフホステッド ランナー
- Azure Stack Hub

Azure Stack Hub で GitHub Actions を使用する際の制約は、このプロセスでは Web に接続されている Azure Stack Hub を使用する必要があるということです。 ワークフローは、GitHub リポジトリ内でトリガーされます。 ID プロバイダーとして、Azure Active Directory (Azure AD) と Active Directory フェデレーション サービス (AD FS) のどちらも使用できます。

これはこの記事の範囲外ですが、セルフホステッド ランナーでは仮想プライベート ネットワークを使用してファイアウォールの内側にある Azure Stack Hub に接続することもできます。

## <a name="get-service-principal"></a>サービス プリンシパルの取得

Azure 外部のプロセスでリソースに接続したり、リソースとやり取りしたりするために、SPN によってロールベースの資格情報が提供されます。 GitHub Actions で使用するために、共同作成者アクセス権を含む SPN と、以下の手順で指定される属性が必要になります。

Azure Stack Hub のユーザーには、SPN を作成するアクセス許可はありません。 このプリンシパルはクラウド オペレーターに要求する必要があります。 ここで提供する説明を使用することで、クラウド オペレーターである場合は SPN を作成できます。あるいは、クラウド オペレーターから提供された SPN をワークフローで使用する開発者である場合は、SPN を検証できます。

クラウド オペレーターは Azure CLI を使用して SPN を作成する必要があります。

次のコード スニペットは、Azure CLI で PowerShell プロンプトを使用する Windows マシン用に作成されています。 Linux コンピューターと bash で CLI を使用している場合は、行継続を削除するか、`\` に置き換えます。

1. SPN の作成に使用する次のパラメーターの値を準備します。

    | パラメーター | 例 | 説明 |
    | --- | --- | --- |
    endpoint-resource-manager | "https://management.orlando.azurestack.corp.microsoft.com"  | リソース管理エンドポイント。 |
    suffix-storage-endpoint | "orlando.azurestack.corp.microsoft.com"  | ストレージ アカウントのエンドポイント サフィックス。 |
    suffix-keyvault-dns | ".vault.orlando.azurestack.corp.microsoft.com"  | Key Vault サービス dns サフィックス。 |
    endpoint-active-directory-graph-resource-id | "https://graph.windows.net/"  | Active Directory リソース ID。 |
    endpoint-sql-management | https://notsupported  | SQL サーバー管理エンドポイント。 これを `https://notsupported` |
    プロファイル | 2019-03-01-hybrid | このクラウドで使用するプロファイル。 |

2. Windows PowerShell や Bash などのコマンドライン ツールを開き、サインインします。 次のコマンドを使用します。

    ```azurecli  
    az login
    ```

3. 新しい環境の場合は `register` コマンドを使用し、既存の環境を使用する場合は `update` コマンドを使用します。 次のコマンドを使用します。

    ```azurecli  
    az cloud register `
        -n "AzureStackUser" `
        --endpoint-resource-manager "https://management.<local>.<FQDN>" `
        --suffix-storage-endpoint ".<local>.<FQDN>" `
        --suffix-keyvault-dns ".vault.<local>.<FQDN>" `
        --endpoint-active-directory-graph-resource-id "https://graph.windows.net/" `
        --endpoint-sql-management https://notsupported  `
        --profile 2019-03-01-hybrid
    ```

4. SPN に使用するサブスクリプション ID とリソース グループを取得します。

5. サブスクリプション ID とリソース グループを使用して、次のコマンドで SPN を作成します。

    ```azurecli  
    az ad sp create-for-rbac --name "myApp" --role contributor `
        --scopes /subscriptions/{subscription-id}/resourceGroups/{resource-group} `
        --sdk-auth
    ```

6. 結果の JSON オブジェクトを確認します。 JSON オブジェクトを使用して、アクションが含まれる GitHub リポジトリにシークレットを作成します。 JSON オブジェクトには、次の属性が必要です。

    ```json
    {
      "clientId": <Application ID for the SPN>,
      "clientSecret": <Client secret for the SPN>,
      "subscriptionId": <Subscription ID for the SPN>,
      "tenantId": <Tenant ID for the SPN>,
      "activeDirectoryEndpointUrl": "https://login.microsoftonline.com/",
      "resourceManagerEndpointUrl": "https://management.<FQDN>",
      "activeDirectoryGraphResourceId": "https://graph.windows.net/",
      "sqlManagementEndpointUrl": "https://notsupported",
      "galleryEndpointUrl": "https://providers.<FQDN>:30016/",
      "managementEndpointUrl": "https://management.<FQDN>"
    }
    ```

## <a name="add-service-principal-to-repository"></a>リポジトリへのサービス プリンシパルの追加

GitHub シークレットを使用して、アクションで使用する機密情報を暗号化できます。 SPN を格納するシークレットを作成して、アクションが Azure Stack Hub インスタンスにサインインできるようにします。

> [!WARNING]  
> GitHub では、セルフホステッド ランナーをパブリック リポジトリと共に使用しないことを推奨しています。パブリック リポジトリのフォークでは、ワークフローのコードを実行するプル要求リクエストを作成することで、セルフホステッド ランナー上で危険なコードが実行される可能性があります。 詳細については、[セルフホステッド ランナー](https://docs.github.com/en/free-pro-team@latest/github/automating-your-workflow-with-github-actions/about-self-hosted-runners#self-hosted-runner-security-with-public-repositories)に関する記述をご覧ください。

1. GitHub リポジトリを開くか作成します。 GitHub でリポジトリを作成するためのガイダンスが必要な場合は、[GitHub ドキュメントの手順](https://docs.github.com/en/free-pro-team@latest/github/getting-started-with-github/create-a-repo)を参照してください。
1. リポジトリをプライベートに設定します。
    1. **[Settings]\(設定\)**  >  **[Change repository visibility]\(リポジトリ表示の変更\)** を選択します。
    1. **[Make private]\(プライベートにする\)** を選択します。
    1. リポジトリの名前を入力します。
    1. **[I understand, change the repository visibility]\(理解して、リポジトリの表示を変更します\)** を選択します。
1. **[設定]** を選択します。
1. **[Secrets]\(シークレット\)** を選択します。
1. **[New repository secret]\(新しいリポジトリ シークレット\)** を選択します。
    ![GitHub Actions シークレットの追加](.\media\ci-cd-github-action-login-cli\github-action-secret.png)
1. シークレットに `AZURE_CREDENTIALS` という名前を付けます。
1. SPN を表す JSON オブジェクトを貼り付けます。
1. **[Add secret]\(シークレットの追加\)** を選択します。

## <a name="create-your-vm-and-install-prerequisites"></a>VM の作成と前提条件のインストール

1. セルフホステッド ランナーを作成します。 

    次の手順では、Azure の Windows VM としてランナーを作成します。 データセンターでホストされている Azure Stack Hub に接続する必要がある場合は、VPN 接続が必要になることがあります。 接続を有効にする方法については、VPN 接続が必要な[セルフホステッド ランナーへの Azure Stack Hub ツールのインストール](#optional-install-azure-stack-hub-tools-on-your-self-hosted-runner)のセクションを参照してください。
    - Azure で Windows VM を作成する方法については、「[クイックスタート:Azure portal で Windows 仮想マシンを作成する](/azure/virtual-machines/windows/quick-create-portal)」を参照してください。 この手順に従う場合は、Windows Server 2016 Core をインストールしてください。
    - Azure Stack Hub で Windows VM を作成する方法については、「[クイックスタート:Azure Stack Hub ポータルを使用して Windows サーバー VM を作成します](./azure-stack-quick-windows-portal.md)。 この手順に従う場合は、Windows Server 2016 Core をインストールしてください。
1. リモート接続を使用して、サーバーの IP アドレス、ユーザー名およびパスワード (マシンの作成時に定義) を使用して、Windows 2016 サーバーに接続します。
1. Chocolatey をインストールします。 Chocolatey は、コマンド ラインからの依存関係のインストールと管理に使用できる Windows のパッケージ マネージャーです。 管理者特権の PowerShell プロンプトで、次のように入力します。
    ```powershell
    Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
    ```
1. PowerShell Core をインストールします。 管理者特権の PowerShell プロンプトで、次のように入力します。
    ```powershell  
    choco install powershell-core
    ```
1. Azure CLI をインストールします。 管理者特権の PowerShell プロンプトで、次のように入力します。
    ```powershell  
    choco install azure-cli
    ```
1. Azure Stack Hub PowerShell をインストールします。 管理者特権の PowerShell プロンプトで、次のように入力します。
    ```powershell  
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

    Install-Module -Name Az.BootStrapper -Force -AllowPrerelease
    Install-AzProfile -Profile 2019-03-01-hybrid -Force
    Install-Module -Name AzureStack -RequiredVersion 2.0.2-preview -AllowPrerelease
    ```
    Azure Stack Hub の Az モジュールの使用の詳細については、「[Azure Stack Hub 用の PowerShell Az モジュールをインストールする](../operator/powershell-install-az-module.md)」を参照してください。
7. マシンを再起動します。 管理者特権の PowerShell プロンプトで、次のように入力します。
    ```powershell  
    shutdown /r
    ```
8. コンピューターをセルフホステッド ランナーとして GitHub リポジトリに追加します。 セルフホステッド ランナーを追加する方法については、GitHub のドキュメントを参照してください。詳細については、「[セルフホステッド ランナーの追加](https://docs.github.com/en/free-pro-team@latest/actions/hosting-your-own-runners/adding-self-hosted-runners)」をご覧ください。

    ![ランナーがリッスンしています](.\media\ci-cd-github-action-login-cli\github-action-runner-listen.png)

9. 完了したら、サービスが実行されており、サービスをリッスンしていることを確認します。 ランナーのディレクトリから `/run.cmd` を実行して、もう一度確認します。

### <a name="optional-install-azure-stack-hub-tools-on-your-self-hosted-runner"></a>省略可能:セルフホステッド ランナーへの Azure Stack Hub ツールのインストール

この記事の手順では、[Azure Stack Hub ツール](../operator/azure-stack-powershell-download.md?tabs=az)へのアクセスは必要ありませんが、独自のワークフローを開発する際には、ツールの使用が必要になる場合があります。 次の手順は、Windows セルフホステッド ランナーにツールをインストールするのに役立ちます。 Azure Stack Hub ツールの詳細については、「[GitHub からの Azure Stack Hub ツールのダウンロード](../operator/azure-stack-powershell-download.md?tabs=az)」を参照してください。 この手順では、パッケージ マネージャー Chocolatey がインストールされていることを前提としています。

1. GIT をインストールします。
    ```powershell  
    choco install git
    ```

2. 管理者特権の PowerShell プロンプトで、次のように入力します。
    ```powershell
    # Change directory to the root directory.
    cd \
    
    # Download the tools archive.
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12 
    invoke-webrequest `
      https://github.com/Azure/AzureStack-Tools/archive/az.zip `
      -OutFile az.zip
    
    # Expand the downloaded files.
    expand-archive az.zip `
      -DestinationPath . `
      -Force
    
    # Change to the tools directory.
    cd AzureStack-Tools-az
    ```

3. ランナーを Azure Stack Hub インスタンスに接続する必要がある場合は、PowerShell を使用できます。 この手順については、記事「[PowerShell を使用して Azure Stack Hub に接続する](../operator/azure-stack-powershell-configure-admin.md?tabs=az1%2Caz2%2Caz3)」を参照してください。

## <a name="create-a-self-hosted-runner"></a>セルフホステッド ランナーの作成

セルフホステッド ランナーは、GitHub ドキュメントで設定できます。セルフホステッド ランナーは、GitHub に接続できる任意のマシンで実行できます。 ワークフローに自動化タスクがあり、広範な依存関係、特定のライセンス要件 (ソフトウェア ライセンス用の USB ドングルなど)、あるいは他のマシンまたはソフトウェア固有のニーズが必要な場合には、セルフホステッド ランナーの使用を選択できます。 マシンとして、物理マシン、VM、またはコンテナーを使用できます。 ランナーは、データセンターまたはクラウドに配置できます。

この記事では、Azure でホストされている Windows VM を使用します。これが Azure Stack Hub 固有の PowerShell 要件で構成されます。

セルフホステッド ランナーの設定、構成、リポジトリへの接続の手順は、GitHub ドキュメント「[セルフホステッド ランナーについて](https://docs.github.com/en/free-pro-team@latest/actions/hosting-your-own-runners/about-self-hosted-runners)」を参照してください。

![接続されているセルフホステッド ランナー](.\media\ci-cd-github-action-login-cli\github-actions-self-hosted-runner.png)

セルフホステッド ランナーの名前とタグをメモしておきます。 この記事のワークフローでは、タグ `self-hosted` を使用して呼び出します。

## <a name="add-the-workflow-to-your-repository"></a>ワークフローのリポジトリへの追加

ワークフローを作成するには、このセクションの yaml を使用して新しいワークフローを作成します。

1. GitHub リポジトリを開きます。
2. **[Actions]\(アクション\)** を選択します。
3. 新しいワークフローを作成します。
    - これが最初のワークフローである場合、 **[Choose a workflow template]\(ワークフロー テンプレートの選択\)** の下で **[set up a workflow yourself]\(自分でワークフローを設定\)** を選択します。
    - 既存のワークフローがある場合は、 **[New workflow]|\(新しいワークフロー\)**  >  **[Set up a workflow yourself]\(自分でワークフローを設定\)** を選択します。

4. パスで、ファイルに `workflow.yml` という名前を指定します。
5. ワークフロー yml をコピーして貼り付けます。
    ```yaml  
    on: [push]
    
    env:
      ACTIONS_ALLOW_UNSECURE_COMMANDS: 'true'
     
    jobs: 
      azurestack-test:
        runs-on: self-hosted
        steps:
    
          - name: Login to AzureStack with Az Powershell
            uses: azure/login@releases/v1
            with:
              creds: ${{ secrets.AZURE_CREDENTIALS }}
              environment: 'AzureStack'
              enable-AzPSSession: true
          
          - name: Run Az PowerShell Script Against AzureStack
            uses: azure/powershell@v1
            with:
              azPSVersion: '3.1.0'
              inlineScript: |
                hostname
                Get-AzContext
                Get-AzResourceGroup
          
          - name: Login to AzureStack with CLI
            uses: azure/login@releases/v1
            with:
              creds: ${{ secrets.AZURE_CREDENTIALS }}
              environment: 'AzureStack'
              enable-AzPSSession: false
          
          - name: Run Azure CLI Script Against AzureStack
            run: |
              hostname
              az group list --output table
    ```
6. **[Start commit]\(コミットの開始\)** を選択します。
7. コミットのタイトルと省略可能な詳細を追加し、 **[Commit new file]\(新しいファイルのコミット\)** を選択します。

アクションが実行されたら、正常に実行されたことを確認します。

1. GitHub リポジトリを開きます。 リポジトリにプッシュすることで、ワークフローをトリガーできます。
1. **[Actions]\(アクション\)** を選択します。
1. **[All workflows]\(すべてのワークフロー\)** の下でコミット名を選択します。

    ![コミットの概要の確認](.\media\ci-cd-github-action-login-cli\github-actions-review-log-summary.png)
1. ジョブの名前 **azurestack-test** を選択します。

    ![コミットの詳細の確認](.\media\ci-cd-github-action-login-cli\github-action-success-screen.png)
1. セクションを展開して、PowerShell および CLI コマンドの戻り値を確認します。

ワークフロー ファイルとアクションに関する注意事項:

- ワークフローには、`azurestack-test` という名前のジョブが 1 つ含まれています。
- プッシュ イベントによってワークフローがトリガーされます。
- アクションでは、リポジトリ内に設定されたセルフホステッド ランナーを使用し、`runs on: self-hosted` という行により、ワークフロー内でランナーのラベルによって呼び出されます。
- ワークフローには 3 つのアクションが含まれています。
- 最初のアクションでは、Azure ログイン アクションを呼び出して、PowerShell を使用してサインインします。Azure 向けの GitHub Actions を使用すると、ビルド、テスト、パッケージ化、リリース、および Azure へのデプロイを行うために設定可能なワークフローをリポジトリ内に作成できます。 このアクションでは、Azure Stack SPN 資格情報を使用して、セッションを開き Azure Stack Hub 環境に接続します。 このアクションの使用方法の詳細については、GitHub の「[Azure ログイン アクション](https://github.com/marketplace/actions/azure-login)」を参照してください。
- 2 番目のアクションでは、Azure PowerShell を使用します。 このアクションでは Az PowerShell モジュールを使用し、このアクションは Government クラウドおよび Azure Stack Hub クラウドの両方で動作します。 このワークフローを実行した後、ジョブを確認して、Azure Stack Hub 環境でスクリプトによってリソース グループが収集されたことを検証します。 詳細については、「[Azure PowerShell アクション](https://github.com/marketplace/actions/azure-powershell-action)」を参照してください。
- 3 番目のアクションでは、Azure CLI を使用してサインインし、Azure Stack Hub に接続してリソース グループを収集します。 詳細については、「[Azure CLI アクション](https://github.com/marketplace/actions/azure-cli-action)」を参照してください。
- GitHub Actions とセルフホステッド ランナーの使用方法の詳細については、[GitHub Actions](https://github.com/features/actions) のドキュメントを参照してください。

## <a name="next-steps"></a>次のステップ

- その他のアクションについては、[GitHub Marketplace](https://github.com/marketplace) を参照してください。
- 「[Common deployments for Azure Stack Hub 向けの一般的なデプロイ](azure-stack-dev-start-deploy-app.md)」について学習する  
- 「[Azure Stack Hub で Azure Resource Manager テンプレートを使用する](azure-stack-arm-templates.md)」について学習する  
- DevOps ハイブリッド クラウド パターン、[DevOps パターン](/hybrid/app-solutions/pattern-cicd-pipeline)を確認する
