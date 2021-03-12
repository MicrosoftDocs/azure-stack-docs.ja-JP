---
title: Azure Stack Hub SQL リソース プロバイダーの更新
titleSuffix: Azure Stack Hub
description: Azure Stack Hub SQL リソース プロバイダーの更新方法について説明します。
author: bryanla
ms.topic: article
ms.date: 8/19/2020
ms.author: bryanla
ms.reviewer: xiaofmao
ms.lastreviewed: 11/11/2019
ms.openlocfilehash: 27f7725ac5b650c201935b86e823b2a8618dcbc4
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101840271"
---
# <a name="update-the-sql-resource-provider"></a>SQL リソース プロバイダーの更新

> [!IMPORTANT]
> リソース プロバイダーを更新する前に、新しい機能、修正、およびデプロイに影響を与える可能性のある既知の問題に関する詳細については、リリース ノートを確認してください。 リリース ノートには、リソース プロバイダーに必要な Azure Stack Hub の最小バージョンも指定されています。

Azure Stack Hub が新しいビルドに更新される際に、新しい SQL リソース プロバイダーがリリースされることがあります。 既存のリソース プロバイダーが動作し続けている場合でも、できるだけ早く最新のビルドに更新することをお勧めします。

|サポートされる Azure Stack Hub のバージョン|SQL RP バージョン|RP サービスが実行されている Windows Server
  |-----|-----|-----|
  |2008、2005|[SQL RP バージョン 1.1.93.1](https://aka.ms/azshsqlrp11931)|Microsoft AzureStack Add-on RP Windows Server
  |2005、2002、1910|[SQL RP バージョン 1.1.47.0](https://aka.ms/azurestacksqlrp11470)|Windows Server 2016 Datacenter - Server Core|
  |1908|[SQL RP バージョン 1.1.33.0](https://aka.ms/azurestacksqlrp11330)|Windows Server 2016 Datacenter - Server Core|
  |     |     |     |

SQL リソース プロバイダーの更新プログラムは累積的です。 古いバージョンから更新する場合は、最新バージョンに直接更新できます。 

リソース プロバイダーの更新には **UpdateSQLProvider.ps1** スクリプトを使用します。 ローカル管理権限を持ち、サブスクリプションの **所有者** であるサービス アカウントを使用します。 この更新スクリプトは、リソース プロバイダーのダウンロードに含まれています。 

更新プロセスは、[リソース プロバイダーを展開する](./azure-stack-sql-resource-provider-deploy.md)ために使用されるプロセスに似ています。 更新スクリプトは DeploySqlProvider.ps1 スクリプトと同じ引数を使用し、証明書情報を提供する必要があります。

## <a name="update-script-processes"></a>更新スクリプトのプロセス

**UpdateSQLProvider.ps1** スクリプトにより、最新の OS イメージを使用して新しい仮想マシン (VM) が作成され、最新のリソース プロバイダーのコードがデプロイされて、古いリソース プロバイダーから新しいリソース プロバイダーに設定が移行されます。 

> [!NOTE]
>Marketplace の管理から最新の Windows Server 2016 Core イメージまたは Microsoft AzureStack Add-on RP Windows Server イメージをダウンロードすることをお勧めします。 更新プログラムをインストールする必要がある場合は、ローカルの依存関係のパスに MSU パッケージを **1 つ** 配置できます。 この場所に複数の MSU ファイルがある場合、スクリプトは失敗します。

*UpdateSQLProvider.ps1* スクリプトにより、新しい VM が作成された後、このスクリプトによって古いリソース プロバイダー VM から次の設定が移行されます。

* データベース情報
* ホスティング サーバー情報
* 必要な DNS レコード

> [!IMPORTANT]
> デプロイまたは更新スクリプトを実行する前に、**Clear-AzureRmContext -Scope CurrentUser** および **Clear-AzureRmContext -Scope Process** を使用してキャッシュをクリアすることを強くお勧めします。

## <a name="update-script-parameters"></a>更新スクリプトのパラメーター

**UpdateSQLProvider.ps1** PowerShell スクリプトを実行するときに、コマンド ラインから以下のパラメーターを指定できます。 必須パラメーターの指定がない場合、またはいずれかのパラメーターの検証が失敗した場合は、指定することを求められます。

| パラメーター名 | 説明 | コメントまたは既定値 |
| --- | --- | --- |
| **CloudAdminCredential** | 特権エンドポイントへのアクセスに必要な、クラウド管理者の資格情報。 | _必須_ |
| **AzCredential** | Azure Stack Hub サービス管理者アカウントの資格情報。 Azure Stack Hub のデプロイに使用したのと同じ資格情報を使用します。 AzCredential で使用するアカウントが多要素認証 (MFA) を必要とする場合、スクリプトは失敗します。 | _必須_ |
| **VMLocalCredential** | SQL リソース プロバイダー VM のローカル管理者アカウントの資格情報。 | _必須_ |
| **PrivilegedEndpoint** | 特権エンドポイントの IP アドレスまたは DNS 名。 |  _必須_ |
| **AzureEnvironment** | Azure Stack Hub のデプロイに使用したサービス管理者アカウントの Azure 環境。 Azure AD のデプロイでのみ必須です。 サポートされている環境名は **AzureCloud**、**AzureUSGovernment**、または中国の Azure AD を使用している場合は **AzureChinaCloud** です。 | AzureCloud |
| **DependencyFilesLocalPath** | このディレクトリには、証明書 .pfx ファイルも配置する必要があります。 | "_1 つのノードの場合は省略可能。複数のノードの場合は必須_" |
| **DefaultSSLCertificatePassword** | .pfx 証明書のパスワード。 | _必須_ |
| **MaxRetryCount** | エラーが 発生した場合に各操作を再試行する回数。| 2 |
| **RetryDuration** |再試行間のタイムアウト間隔 (秒単位)。 | 120 |
| **アンインストール** | リソース プロバイダーと関連付けられているすべてのリソースを削除します。 | いいえ |
| **DebugMode** | 障害発生時に自動クリーンアップが行われないようにします。 | いいえ |

## <a name="update-script-powershell-example"></a>PowerShell 更新スクリプトの例

SQL リソース プロバイダーのバージョンを 1.1.33.0 以前のバージョンに更新する場合は、PowerShell で特定のバージョンの AzureRm.BootStrapper および Azure Stack Hub モジュールをインストールする必要があります。 

SQL リソース プロバイダーを 1.1.47.0 以降のバージョンに更新する場合は、この手順をスキップできます。 デプロイ スクリプトにより、必要な PowerShell モジュールが自動的にダウンロードされ、パス C:\Program Files\SqlMySqlPsh にインストールされます。

>[!NOTE]
>PowerShell モジュールがダウンロードされるフォルダー C:\Program Files\SqlMySqlPsh が既に存在する場合は、更新スクリプトを実行する前にこのフォルダーをクリーンアップすることをお勧めします。 これは、適切なバージョンの PowerShell モジュールがダウンロードされて使用されるようにするためです。

```powershell
# Run the following scripts when updating to version 1.1.33.0 only.
# Install the AzureRM.Bootstrapper module, set the profile, and install the AzureStack module.
# Note that this might not be the most currently available version of Azure Stack Hub PowerShell.
Install-Module -Name AzureRm.BootStrapper -Force
Use-AzureRmProfile -Profile 2018-03-01-hybrid -Force
Install-Module -Name AzureStack -RequiredVersion 1.6.0
```

> [!NOTE]
> 接続が切断された場合のシナリオでは、必要な PowerShell モジュールのダウンロードと手動でのリポジトリ登録が前提条件として必要です。 詳細については、[SQL リソース プロバイダーのデプロイ](azure-stack-sql-resource-provider-deploy.md)に関する記事を参照してください。

次に、管理者特権の PowerShell コンソールから実行できる *UpdateSQLProvider.ps1* スクリプトの使用例を示します。 変数情報とパスワードは、必要に応じて変更してください。  

```powershell
# Use the NetBIOS name for the Azure Stack Hub domain. On the Azure Stack Hub SDK, the default is AzureStack but this might have been changed at installation.
$domain = "AzureStack"

# For integrated systems, use the IP address of one of the ERCS VMs.
$privilegedEndpoint = "AzS-ERCS01"

# Provide the Azure environment used for deploying Azure Stack Hub. Required only for Azure AD deployments. Supported values for the <environment name> parameter are AzureCloud, AzureChinaCloud, or AzureUSGovernment depending which Azure subscription you're using.
$AzureEnvironment = "<EnvironmentName>"

# Point to the directory where the resource provider installation files were extracted.
$tempDir = 'C:\TEMP\SQLRP'

# The service admin account (this can be Azure AD or AD FS).
$serviceAdmin = "admin@mydomain.onmicrosoft.com"
$AdminPass = ConvertTo-SecureString 'P@ssw0rd1' -AsPlainText -Force
$AdminCreds = New-Object System.Management.Automation.PSCredential ($serviceAdmin, $AdminPass)

# Set the credentials for the new resource provider VM.
$vmLocalAdminPass = ConvertTo-SecureString 'P@ssw0rd1' -AsPlainText -Force
$vmLocalAdminCreds = New-Object System.Management.Automation.PSCredential ("sqlrpadmin", $vmLocalAdminPass)

# Add the cloudadmin credential required for privileged endpoint access.
$CloudAdminPass = ConvertTo-SecureString 'P@ssw0rd1' -AsPlainText -Force
$CloudAdminCreds = New-Object System.Management.Automation.PSCredential ("$domain\cloudadmin", $CloudAdminPass)

# Change the following as appropriate.
$PfxPass = ConvertTo-SecureString 'P@ssw0rd1' -AsPlainText -Force

# For version 1.1.47.0 or later, the PowerShell modules used by the RP deployment are placed in C:\Program Files\SqlMySqlPsh
# The deployment script adds this path to the system $env:PSModulePath to ensure correct modules are used.
$rpModulePath = Join-Path -Path $env:ProgramFiles -ChildPath 'SqlMySqlPsh'
$env:PSModulePath = $env:PSModulePath + ";" + $rpModulePath

# Change directory to the folder where you extracted the installation files.
# Then adjust the endpoints.
. $tempDir\UpdateSQLProvider.ps1 -AzCredential $AdminCreds `
  -VMLocalCredential $vmLocalAdminCreds `
  -CloudAdminCredential $cloudAdminCreds `
  -PrivilegedEndpoint $privilegedEndpoint `
  -AzureEnvironment $AzureEnvironment `
  -DefaultSSLCertificatePassword $PfxPass `
  -DependencyFilesLocalPath $tempDir\cert
 ```

リソース プロバイダーの更新スクリプトが終了したら、現在の PowerShell セッションを終了します。

## <a name="next-steps"></a>次のステップ

[SQL リソース プロバイダーの維持](azure-stack-sql-resource-provider-maintain.md)
