---
title: SQL Server リソース プロバイダーのデプロイ
titleSuffix: Azure Stack Hub
description: Azure Stack Hub に SQL Server リソース プロバイダーをデプロイする方法について説明します。
author: bryanla
ms.topic: article
ms.date: 12/07/2020
ms.lastreviewed: 12/07/2020
ms.author: bryanla
ms.reviewer: xiao
ms.openlocfilehash: 7f3813d241b01e46f4a0eac639e3e6c78d74e50f
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101840322"
---
# <a name="deploy-the-sql-server-resource-provider-on-azure-stack-hub"></a>Azure Stack Hub への SQL Server リソース プロバイダーのデプロイ

Azure Stack Hub SQL Server リソース プロバイダーを使用して、SQL データベースを Azure Stack Hub サービスとして公開します。 SQL リソース プロバイダーは、Windows Server 2016 Server Core 仮想マシン (アダプター バージョン 1.1.47.0 以降) または特別なアドオン RP Windows Server (アダプター バージョン 1.1.93.0 以前) 上でサービスとして実行されます。

> [!IMPORTANT]
> SQL または MySQL をホストするサーバー上に項目を作成するのは、リソース プロバイダーのみが実行する必要があります。 リソース プロバイダーによって作成されていないホスト サーバー上の項目はサポートされません。これによって、不一致状態になる可能性があります。

## <a name="prerequisites"></a>前提条件

Azure Stack Hub SQL リソース プロバイダーをデプロイする前に、いくつかの前提条件を満たす必要があります。

- 以下にアクセスできるコンピューターとアカウントが必要です。
   - [Azure Stack Hub 管理者ポータル](azure-stack-manage-portals.md)。
   - [特権エンドポイント](azure-stack-privileged-endpoint.md)。
   - Azure Resource Manager 管理エンドポイント `https://management.region.<fqdn>`。`<fqdn>` は完全修飾ドメイン名 (ASDK を使用する場合は `https://management.local.azurestack.external`) です。
   - Azure Stack Hub が ID プロバイダーとして Azure Active Directory (AD) を使用するようにデプロイされた場合は、インターネット。

- まだ実行していない場合は、Azure Marketplace アイテムをダウンロードできるよう、Azure に [Azure Stack Hub を登録](azure-stack-registration.md)します。

- 必要な Windows Server VM を Azure Stack Hub Marketplace に追加します。
  - SQL RP バージョン <= 1.1.47.0 の場合は、**Windows Server 2016 Datacenter - Server Core** イメージをダウンロードします。
  - 1\.1.93.0 以降の SQL RP バージョンの場合は、**Microsoft AzureStack Add-On RP Windows Server** イメージをダウンロードします。 この Windows Server バージョンは Azure Stack Add-on RP インフラストラクチャ専用であり、テナント マーケットプレースには表示されません。

- 次のバージョン マッピングの表に従って、サポートされているバージョンの SQL リソース プロバイダー バイナリをダウンロードします。 自己解凍ツールを実行して、ダウンロードした内容を一時ディレクトリに抽出します。 

  |サポートされる Azure Stack Hub のバージョン|SQL RP バージョン|RP サービスが実行されている Windows Server
  |-----|-----|-----|
  |2008、2005|[SQL RP バージョン 1.1.93.1](https://aka.ms/azshsqlrp11931)|Microsoft AzureStack Add-on RP Windows Server
  |2005、2002、1910|[SQL RP バージョン 1.1.47.0](https://aka.ms/azurestacksqlrp11470)|Windows Server 2016 Datacenter - Server Core|
  |1908|[SQL RP バージョン 1.1.33.0](https://aka.ms/azurestacksqlrp11330)|Windows Server 2016 Datacenter - Server Core|
  |     |     |     |

- データセンターの統合の前提条件を満たしていることを確認します。

    |前提条件|リファレンス|
    |-----|-----|
    |条件付き DNS フォワーダーが正しく設定されている。|[Azure Stack Hub とデータセンターの統合 - DNS](azure-stack-integrate-dns.md)|
    |リソース プロバイダー用の受信ポートが開いている。|[Azure Stack Hub データセンターの統合 - ポートとプロトコル (受信)](azure-stack-integrate-endpoints.md#ports-and-protocols-inbound)|
    |PKI 証明書のサブジェクトと SAN が正しく設定されている。|[Azure Stack Hub デプロイの必須 PKI 前提条件](azure-stack-pki-certs.md)<br>[Azure Stack Hub デプロイの PaaS 証明書の前提条件](azure-stack-pki-certs.md)|
    |     |     |

接続が切断された場合のシナリオでは、次の手順を実行して、必要な PowerShell モジュールのダウンロードと手動でのリポジトリ登録を行います。

1. インターネット接続が確立されたコンピューターにサインインし、次のスクリプトを使用して、PowerShell モジュールをダウンロードします。

```powershell
Import-Module -Name PowerShellGet -ErrorAction Stop
Import-Module -Name PackageManagement -ErrorAction Stop

# path to save the packages, c:\temp\azs1.6.0 as an example here
$Path = "c:\temp\azs1.6.0"
```

2. デプロイするリソース プロバイダーのバージョンに応じて、いずれかのスクリプトを実行します。

```powershell
# for resource provider version >= 1.1.93.0
Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name AzureRM -Path $Path -Force -RequiredVersion 2.5.0
Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name AzureStack -Path $Path -Force -RequiredVersion 1.8.2
```
```powershell
# for resource provider version <= 1.1.47.0
Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name AzureRM -Path $Path -Force -RequiredVersion 2.3.0
Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name AzureStack -Path $Path -Force -RequiredVersion 1.6.0
```

3. その後、ダウンロードしたパッケージを USB デバイスにコピーします。

4. 接続が切断されたワークステーションにサインインし、パッケージを USB デバイスからワークステーション上の場所にコピーします。

5. この場所をローカル リポジトリとして登録します。

```powershell
# requires -Version 5
# requires -RunAsAdministrator
# requires -Module PowerShellGet
# requires -Module PackageManagement

$SourceLocation = "C:\temp\azs1.6.0"
$RepoName = "azs1.6.0"

Register-PSRepository -Name $RepoName -SourceLocation $SourceLocation -InstallationPolicy Trusted

New-Item -Path $env:ProgramFiles -name "SqlMySqlPsh" -ItemType "Directory"
```

### <a name="certificates"></a>証明書

_統合システムのインストールのみを対象_。 [Azure Stack Hub のデプロイの PKI の要件](./azure-stack-pki-certs.md)に関するページの「オプションの PaaS 証明書」セクションで説明されている SQL PaaS PKI 証明書を指定する必要があります。 **DependencyFilesLocalPath** パラメーターで指定された場所に .pfx ファイルを配置します。 ASDK システムの証明書は提供しないでください。

## <a name="deploy-the-sql-resource-provider"></a>SQL リソース プロバイダーをデプロイする

すべての前提条件を完了したら、Azure Stack Hub の Azure Resource Manager 管理エンドポイントと特権エンドポイントの両方にアクセスできるコンピューターから **DeploySqlProvider.ps1** スクリプトを実行して、SQL リソース プロバイダーをデプロイします。 DeploySqlProvider.ps1 スクリプトは、Azure Stack Hub のバージョンに応じてダウンロードした SQL リソース プロバイダーのバイナリの一部として抽出されます。

 > [!IMPORTANT]
 > リソース プロバイダーをデプロイする前に、新しい機能、修正、デプロイに影響を与える可能性のある既知の問題に関する詳細については、リリース ノートを確認してください。

SQL リソース プロバイダーをデプロイするには、管理者特権で **新しい** PowerShell ウィンドウ (PowerShell ISE ではない) を開き、SQL リソース プロバイダーのバイナリ ファイルを抽出したディレクトリに変更します。 

> [!IMPORTANT]
> デプロイまたは更新スクリプトを実行する前に、**Clear-AzureRmContext -Scope CurrentUser** および **Clear-AzureRmContext -Scope Process** を使用してキャッシュをクリアすることを強くお勧めします。

DeploySqlProvider.ps1 スクリプトを実行すると、次のタスクが完了します。

- Azure Stack Hub のストレージ アカウントに、証明書とその他のアーティファクトをアップロードします。
- ギャラリー パッケージを発行して、ギャラリーを使用して SQL データベースをデプロイできるようにします。
- ホスティング サーバーをデプロイするためのギャラリー パッケージを発行します。
- ダウンロードした Windows Server 2016 Core イメージまたは Microsoft AzureStack Add-on RP Windows Server イメージを使用して VM をデプロイした後、SQL リソース プロバイダーをインストールします。
- リソース プロバイダー VM にマップされるローカル DNS レコードを登録します。
- リソース プロバイダーをオペレーター アカウントのローカルの Azure Resource Manager に登録します。

> [!NOTE]
> SQL リソース プロバイダーのデプロイが開始されると、**system.local.sqladapter** リソース グループが作成されます。 このリソース グループに対して必要なデプロイが完了するまで最大で 75 分かかる可能性があります。 **system.local.sqladapter** リソース グループ内に他のリソースを配置しないでください。

### <a name="deploysqlproviderps1-parameters"></a>DeploySqlProvider.ps1 パラメーター

コマンド ラインから次のパラメーターを指定できます。 必須パラメーターの指定がない場合、またはいずれかのパラメーターの検証が失敗した場合は、指定することを求められます。

| パラメーター名 | 説明 | コメントまたは既定値 |
| --- | --- | --- |
| **CloudAdminCredential** | 特権エンドポイントへのアクセスに必要な、クラウド管理者の資格情報。 | _必須_ |
| **AzCredential** | Azure Stack Hub サービス管理者アカウントの資格情報。 Azure Stack Hub のデプロイに使用したのと同じ資格情報を使用します。 AzCredential で使用するアカウントが多要素認証 (MFA) を必要とする場合、スクリプトは失敗します。| _必須_ |
| **VMLocalCredential** | SQL リソース プロバイダー VM のローカル管理者アカウントの資格情報。 | _必須_ |
| **PrivilegedEndpoint** | 特権エンドポイントの IP アドレスまたは DNS 名。 |  _必須_ |
| **AzureEnvironment** | Azure Stack Hub のデプロイに使用するサービス管理者アカウントの Azure 環境。 Azure AD のデプロイでのみ必須です。 サポートされている環境名は **AzureCloud**、**AzureUSGovernment**、または中国の Azure AD を使用している場合は **AzureChinaCloud** です。 | AzureCloud |
| **DependencyFilesLocalPath** | 統合システムの場合のみ、証明書 .pfx ファイルはこのディレクトリにも配置する必要があります。 必要に応じて、ここで 1 つの Windows Update MSU パッケージをコピーできます。 | _省略可能_ (統合システムでは _必須_) |
| **DefaultSSLCertificatePassword** | .pfx 証明書のパスワード。 | _必須_ |
| **MaxRetryCount** | エラーが 発生した場合に各操作を再試行する回数。| 2 |
| **RetryDuration** | 再試行間のタイムアウト間隔 (秒単位)。 | 120 |
| **アンインストール** | リソース プロバイダーと関連付けられているすべてのリソースを削除します (以下のメモを参照してください)。 | いいえ |
| **DebugMode** | 障害発生時に自動クリーンアップが行われないようにします。 | いいえ |

## <a name="deploy-the-sql-resource-provider-using-a-custom-script"></a>カスタム スクリプトを使用して SQL リソース プロバイダーをデプロイする

SQL リソース プロバイダーのバージョン 1.1.33.0 以前のバージョンをデプロイする場合は、PowerShell で特定のバージョンの AzureRm.BootStrapper と Azure Stack Hub モジュールをインストールする必要があります。 SQL リソース プロバイダーのバージョン 1.1.47.0 以降をデプロイする場合は、デプロイ スクリプトにより自動的に必要な PowerShell モジュールがダウンロードされ、パス C:\Program Files\SqlMySqlPsh にインストールされます。

```powershell
# Install the AzureRM.Bootstrapper module, set the profile, and install the AzureStack module
# Note that this might not be the most currently available version of Azure Stack Hub PowerShell
Install-Module -Name AzureRm.BootStrapper -RequiredVersion 0.5.0 -Force
Use-AzureRmProfile -Profile 2018-03-01-hybrid -Force
Install-Module -Name AzureStack -RequiredVersion 1.6.0
```

> [!NOTE]
> 接続が切断された場合のシナリオでは、必要な PowerShell モジュールのダウンロードと手動でのリポジトリ登録が前提条件として必要です。

リソース プロバイダーをデプロイする際の手動による構成を除外するには、次のスクリプトをカスタマイズできます。 必要に応じて、Azure Stack Hub のデプロイ用の既定のアカウントの情報とパスワードを変更します。

```powershell
# Use the NetBIOS name for the Azure Stack Hub domain. On the Azure Stack Hub SDK, the default is AzureStack but could have been changed at install time.
$domain = "AzureStack"

# For integrated systems, use the IP address of one of the ERCS VMs
$privilegedEndpoint = "AzS-ERCS01"

# Provide the Azure environment used for deploying Azure Stack Hub. Required only for Azure AD deployments. Supported values for the <environment name> parameter are AzureCloud, AzureChinaCloud, or AzureUSGovernment depending which Azure subscription you're using.
$AzureEnvironment = "<EnvironmentName>"

# Point to the directory where the resource provider installation files were extracted.
$tempDir = 'C:\TEMP\SQLRP'

# The service admin account can be Azure Active Directory or Active Directory Federation Services.
$serviceAdmin = "admin@mydomain.onmicrosoft.com"
$AdminPass = ConvertTo-SecureString 'P@ssw0rd1' -AsPlainText -Force
$AdminCreds = New-Object System.Management.Automation.PSCredential ($serviceAdmin, $AdminPass)

# Set credentials for the new resource provider VM local admin account.
$vmLocalAdminPass = ConvertTo-SecureString 'P@ssw0rd1' -AsPlainText -Force
$vmLocalAdminCreds = New-Object System.Management.Automation.PSCredential ("sqlrpadmin", $vmLocalAdminPass)

# Add the cloudadmin credential that's required for privileged endpoint access.
$CloudAdminPass = ConvertTo-SecureString 'P@ssw0rd1' -AsPlainText -Force
$CloudAdminCreds = New-Object System.Management.Automation.PSCredential ("$domain\cloudadmin", $CloudAdminPass)

# Change the following as appropriate.
$PfxPass = ConvertTo-SecureString 'P@ssw0rd1' -AsPlainText -Force

# For version 1.1.47.0 or later, the PowerShell modules used by the RP deployment are placed in C:\Program Files\SqlMySqlPsh
# The deployment script adds this path to the system $env:PSModulePath to ensure correct modules are used.
$rpModulePath = Join-Path -Path $env:ProgramFiles -ChildPath 'SqlMySqlPsh'
$env:PSModulePath = $env:PSModulePath + ";" + $rpModulePath 

# Change to the directory folder where you extracted the installation files. Don't provide a certificate on ASDK!
. $tempDir\DeploySQLProvider.ps1 `
    -AzCredential $AdminCreds `
    -VMLocalCredential $vmLocalAdminCreds `
    -CloudAdminCredential $cloudAdminCreds `
    -PrivilegedEndpoint $privilegedEndpoint `
    -AzureEnvironment $AzureEnvironment `
    -DefaultSSLCertificatePassword $PfxPass `
    -DependencyFilesLocalPath $tempDir\cert

 ```

リソース プロバイダーのインストール スクリプトが終了したら、ブラウザーを最新の情報に更新します。これによって、最新の更新プログラムが表示され、現在の PowerShell セッションを終了できることを確認します。

## <a name="verify-the-deployment-using-the-azure-stack-hub-portal"></a>Azure Stack Hub ポータルを使用してデプロイを確認する

1. 管理ポータルにサービス管理者としてサインインします。
2. **[リソース グループ]** を選択します。
3. **system.\<location\>.sqladapter** リソース グループを選択します。
4. リソース グループの概要の概要ページで、失敗したデプロイは表示されていないはずです。
5. 最後に、管理者ポータルで **[仮想マシン]** を選択して、SQL リソース プロバイダー VM が正常に作成され、実行されていることを確認します。

## <a name="next-steps"></a>次のステップ

[ホスティング サーバーを追加する](azure-stack-sql-resource-provider-hosting-servers.md)
