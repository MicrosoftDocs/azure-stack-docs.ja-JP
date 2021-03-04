---
title: SQL リソース プロバイダーの保守操作
titleSuffix: Azure Stack Hub
description: Azure Stack Hub での SQL リソース プロバイダーのメンテナンス操作について説明します。
author: bryanla
ms.topic: article
ms.date: 9/22/2020
ms.author: bryanla
ms.reviewer: jiahan
ms.lastreviewed: 01/11/2020
ms.openlocfilehash: 05ff121ccba8076276c8dd7106d56992500b57d8
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101840288"
---
# <a name="sql-resource-provider-maintenance-operations"></a>SQL リソース プロバイダーの保守操作

SQL リソース プロバイダーは、ロックダウンされた仮想マシン (VM) 上で実行されます。 メンテナンス操作を有効にするには、VM のセキュリティを更新する必要があります。 最小限の特権の原則を使用してそれを行うには、[PowerShell Just Enough Administration (JEA)](/powershell/scripting/learn/remoting/jea/overview) エンドポイント *DBAdapterMaintenance* を使用します。 リソース プロバイダーのインストール パッケージには、この操作のためのスクリプトが含まれています。

## <a name="patching-and-updating"></a>修正プログラム適用と更新

SQL リソース プロバイダーはアドオン コンポーネントであるため、Azure Stack Hub の一部としては提供されません。 必要に応じて、SQL リソース プロバイダーの更新プログラムが提供されます。 更新された SQL アダプターがリリースされると、更新プログラムを適用するためのスクリプトが提供されます。 このスクリプトによって新しいリソース プロバイダー VM が作成され、古い方のプロバイダーの状態が新しい方の VM に移行されます。 詳細については、「[SQL リソース プロバイダーの更新](azure-stack-sql-resource-provider-update.md)」を参照してください。

### <a name="provider-vm"></a>プロバイダー VM

リソース プロバイダーは "*ユーザー*" VM 上で実行されているので、必要な修正プログラムと更新プログラムがリリースされたら、それらを適用する必要があります。 修正プログラム適用と更新のサイクルの一環で提供される Windows 更新プログラム パッケージを使用して、VM に更新プログラムを適用します。

## <a name="updating-sql-credentials"></a>SQL 資格情報の更新

管理者は、SQL Server の sysadmin アカウントの作成と保守を担当します。 リソース プロバイダーには、ユーザーに代わってデータベースを管理するためにこれらの権限があるアカウントが必要となりますが、ユーザーのデータにアクセスする必要はありません。 SQL Server で sysadmin パスワードを更新する必要がある場合、リソース プロバイダーの管理者インターフェイスを使用して、保存済みパスワードを変更できます。 これらのパスワードは、Azure Stack Hub インスタンス上の Key Vault に格納されています。

設定を変更するには、 **[参照]** &gt; **[管理リソース]** &gt; **[SQL ホスティング サーバー]** &gt; **[SQL ログイン]** の順に選択し、ユーザー名を選択します。 変更は、最初に SQL インスタンス (および必要な場合はレプリカ) で行う必要があります。 **[設定]** で **[パスワード]** を選択します。

![SQL 管理者パスワードの更新](./media/azure-stack-sql-rp-deploy/sql-rp-update-password.png)

## <a name="secrets-rotation"></a>シークレットのローテーション

"*この説明は、Azure Stack Hub 統合システムに対してのみ適用されます。* "

Azure Stack Hub 統合システムで SQL および MySQL リソース プロバイダーを使用する場合、以下のリソース プロバイダーのインフラストラクチャ シークレットが確実に期限切れにならないようにローテーションする責任は Azure Stack Hub オペレーターにあります。

- [デプロイ時に提供](azure-stack-pki-certs.md)された外部 SSL 証明書。
- デプロイ時に提供された、リソース プロバイダー VM のローカル管理者アカウントのパスワード。
- リソース プロバイダーの診断ユーザー (dbadapterdiag) のパスワード。
- (バージョン 1.1.47.0 以上) デプロイ時に生成された Key Vault 証明書。

### <a name="powershell-examples-for-rotating-secrets"></a>PowerShell のシークレットのローテーション例

**すべてのシークレットを同時に変更する。**

```powershell
.\SecretRotationSQLProvider.ps1 `
    -Privilegedendpoint $Privilegedendpoint `
    -CloudAdminCredential $cloudCreds `
    -AzCredential $adminCreds `
    -DiagnosticsUserPassword $passwd `
    -DependencyFilesLocalPath $certPath `
    -DefaultSSLCertificatePassword $certPasswd  `
    -VMLocalCredential $localCreds `
    -KeyVaultPfxPassword $keyvaultCertPasswd
```

**診断ユーザーのパスワードを変更する。**

```powershell
.\SecretRotationSQLProvider.ps1 `
    -Privilegedendpoint $Privilegedendpoint `
    -CloudAdminCredential $cloudCreds `
    -AzCredential $adminCreds `
    -DiagnosticsUserPassword  $passwd
```

**VM ローカル管理者アカウントのパスワードを変更する。**

```powershell
.\SecretRotationSQLProvider.ps1 `
    -Privilegedendpoint $Privilegedendpoint `
    -CloudAdminCredential $cloudCreds `
    -AzCredential $adminCreds `
    -VMLocalCredential $localCreds
```

**SSL 証明書のパスワードを変更する。**

```powershell
.\SecretRotationSQLProvider.ps1 `
    -Privilegedendpoint $Privilegedendpoint `
    -CloudAdminCredential $cloudCreds `
    -AzCredential $adminCreds `
    -DependencyFilesLocalPath $certPath `
    -DefaultSSLCertificatePassword $certPasswd
```

**Key Vault 証明書のパスワードを変更する。**

```powershell
.\SecretRotationSQLProvider.ps1 `
    -Privilegedendpoint $Privilegedendpoint `
    -CloudAdminCredential $cloudCreds `
    -AzCredential $adminCreds `
    -KeyVaultPfxPassword $keyvaultCertPasswd
```

### <a name="secretrotationsqlproviderps1-parameters"></a>SecretRotationSQLProvider.ps1 のパラメーター

|パラメーター|説明|解説|
|-----|-----|-----|
|AzureEnvironment|Azure Stack Hub のデプロイに使用するサービス管理者アカウントの Azure 環境。 Azure AD のデプロイでのみ必須です。 サポートされている環境名は **AzureCloud**、**AzureUSGovernment**、または中国の Azure Active Directory を使用している場合は **AzureChinaCloud** です。|省略可能|
|AzCredential|Azure Stack Hub サービス管理者アカウントの資格情報。 AzCredential で使用するアカウントが多要素認証 (MFA) を必要とする場合、スクリプトは失敗します。|Mandatory|
|CloudAdminCredential|Azure Stack Hub クラウド管理者ドメイン アカウントの資格情報。|Mandatory|
|PrivilegedEndpoint|Get-AzureStackStampInformation にアクセスするための特権エンドポイント。|Mandatory|
|DiagnosticsUserPassword|診断ユーザー アカウントのパスワード。|省略可能|
|VMLocalCredential|MySQLAdapter VM のローカル管理者アカウント。|省略可能|
|DefaultSSLCertificatePassword|既定の SSL 証明書 (*pfx) のパスワード。|省略可能|
|DependencyFilesLocalPath|依存関係ファイルのローカル パス。|省略可能|
|KeyVaultPfxPassword|データベース アダプターの Key Vault 証明書の生成に使用されるパスワード。|省略可能|
|     |     |     |

### <a name="known-issues"></a>既知の問題

**問題**:<br>
シークレット ローテーション ログ。 シークレット ローテーションのカスタム スクリプトが実行され、失敗した場合、シークレット ローテーションのログは自動的に収集されません。

**回避策**:<br>
Get-AzsDBAdapterLogs コマンドレットを使用して、C:\Logs に保存されているすべてのリソース プロバイダーのログ (AzureStack.DatabaseAdapter.SecretRotation.ps1_*.log など) を収集します。

## <a name="update-the-vm-operating-system"></a>VM のオペレーティング システムを更新する

VM のオペレーティング システムを更新するには、次のいずれかの方法を使用します。

- 現在パッチが適用されている VM イメージを使用して最新のリソース プロバイダーのパッケージをインストールする。
- リソース プロバイダーのインストールまたは更新中に Windows 更新プログラム パッケージをインストールする。

## <a name="update-the-vm-windows-defender-definitions"></a>VM の Windows Defender の定義を更新する

Windows Defender の定義を更新するには: 

1. [Windows Defender のセキュリティ インテリジェンスの更新](https://www.microsoft.com/wdsi/definitions)に関するページから Windows Defender 定義の更新プログラムをダウンロードします。

   定義の更新に関するページ上で下方にスクロールして「Manually download the update」 (更新プログラムを手動でダウンロードする) を見つけます。 "Windows Defender Antivirus for Windows 10 および Windows 8.1" 64 ビット ファイルをダウンロードします。

   または、[こちらのダイレクト リンク](https://go.microsoft.com/fwlink/?LinkID=121721&arch=x64)を使用して、fpam-fe.exe ファイルをダウンロードして実行することもできます。

2. SQL リソース プロバイダー アダプター VM のメンテナンス エンドポイントへの PowerShell セッションを作成します。

3. メンテナンス エンドポイントのセッションを使用して、VM に定義更新ファイルをコピーします。

4. メンテナンス PowerShell セッションで *Update-DBAdapterWindowsDefenderDefinitions* コマンドを実行します。

5. 定義をインストールしたら、*Remove-ItemOnUserDrive* コマンドを使用して、定義更新ファイルを削除することをお勧めします。

**定義を更新するための PowerShell スクリプトの例**

次のスクリプトを編集して実行することにより、Defender の定義を更新することができます。 スクリプト内の値を、実際の環境の値に置き換えます。

```powershell
# Set credentials for local admin on the resource provider VM.
$vmLocalAdminPass = ConvertTo-SecureString '<local admin user password>' -AsPlainText -Force
$vmLocalAdminUser = "<local admin user name>"
$vmLocalAdminCreds = New-Object System.Management.Automation.PSCredential `
    ($vmLocalAdminUser, $vmLocalAdminPass)

# Provide the public IP address for the adapter VM.
$databaseRPMachine  = "<RP VM IP address>"
$localPathToDefenderUpdate = "C:\DefenderUpdates\mpam-fe.exe"

# Download the Windows Defender update definitions file from https://www.microsoft.com/wdsi/definitions.
Invoke-WebRequest -Uri 'https://go.microsoft.com/fwlink/?LinkID=121721&arch=x64' `
    -Outfile $localPathToDefenderUpdate

# Create a session to the maintenance endpoint.
$session = New-PSSession -ComputerName $databaseRPMachine `
    -Credential $vmLocalAdminCreds -ConfigurationName DBAdapterMaintenance
# Copy the defender update file to the adapter VM.
Copy-Item -ToSession $session -Path $localPathToDefenderUpdate `
     -Destination "User:\"
# Install the update definitions.
Invoke-Command -Session $session -ScriptBlock `
    {Update-AzSDBAdapterWindowsDefenderDefinition -DefinitionsUpdatePackageFile "User:\mpam-fe.exe"}
# Cleanup the definitions package file and session.
Invoke-Command -Session $session -ScriptBlock `
    {Remove-AzSItemOnUserDrive -ItemPath "User:\mpam-fe.exe"}
$session | Remove-PSSession
```

## <a name="collect-diagnostic-logs"></a>診断ログの収集

::: moniker range=">= azs-2008"

Azure Stack Hub には、診断ログを収集、保存して Microsoft サポートに送信するための方法が複数用意されています。 バージョン 1.1.93 から、SQL リソース プロバイダーは、Azure Stack Hub 環境からログを収集するための標準的な方法をサポートしています。 詳細については、「[診断ログの収集](diagnostic-log-collection.md)」を参照してください。

::: moniker-end

::: moniker range="< azs-2008"

バージョン 1.1.93 から、SQL リソース プロバイダーは、Azure Stack Hub 環境からログを収集するための標準的な方法をサポートしています。 以前のバージョンを使用している場合は、SQL リソース プロバイダーを最新バージョンに更新することをお勧めします。

ロックダウンされた VM からログを収集するには、PowerShell Just Enough Administration (JEA) エンドポイント *DBAdapterDiagnostics* を使用します。 このエンドポイントでは、次のコマンドが提供されます。

- **Get-AzsDBAdapterLog**。 このコマンドは、リソース プロバイダー診断ログの zip パッケージを作成し、そのファイルをセッションのユーザー ドライブ上に保存します。 このコマンドはパラメーターなしで実行することができ、過去 4 時間のログが収集されます。
- **Remove-AzsDBAdapterLog**。 このコマンドは、リソース プロバイダー VM に対する既存のログ パッケージを削除します。

### <a name="endpoint-requirements-and-process"></a>エンドポイントの要件とプロセス

リソース プロバイダーをインストールまたは更新すると、**dbadapterdiag** ユーザー アカウントが作成されます。 このアカウントを使用することで、診断ログを収集できます。

>[!NOTE]
>dbadapterdiag アカウント パスワードは、プロバイダーのデプロイ中または更新中に作成される VM のローカル管理者用に使用されるパスワードと同じです。

*DBAdapterDiagnostics* コマンドを使用するには、リソース プロバイダー VM に対するリモート PowerShell セッションを作成し、**Get-AzsDBAdapterLog** コマンドを実行します。

**FromDate** および **ToDate** パラメーターを使用して、ログ収集の期間を設定します。 これらのパラメーターの一方または両方を指定しない場合は、次の既定値が使用されます。

- FromDate は現在の時刻の 4 時間前です。
- ToDate は現在の時刻です。

**ログを収集する PowerShell スクリプトの例**

次のスクリプトでは、リソース プロバイダー VM から診断ログを収集する方法を示します。

```powershell
# Create a new diagnostics endpoint session.
$databaseRPMachineIP = '<RP VM IP address>'
$diagnosticsUserName = 'dbadapterdiag'
$diagnosticsUserPassword = '<Enter Diagnostic password>'

$diagCreds = New-Object System.Management.Automation.PSCredential `
        ($diagnosticsUserName, (ConvertTo-SecureString -String $diagnosticsUserPassword -AsPlainText -Force))
$session = New-PSSession -ComputerName $databaseRPMachineIP -Credential $diagCreds `
        -ConfigurationName DBAdapterDiagnostics

# Sample that captures logs from the previous hour.
$fromDate = (Get-Date).AddHours(-1)
$dateNow = Get-Date
$sb = {param($d1,$d2) Get-AzSDBAdapterLog -FromDate $d1 -ToDate $d2}
$logs = Invoke-Command -Session $session -ScriptBlock $sb -ArgumentList $fromDate,$dateNow

# Copy the logs to the user drive.
$sourcePath = "User:\{0}" -f $logs
$destinationPackage = Join-Path -Path (Convert-Path '.') -ChildPath $logs
Copy-Item -FromSession $session -Path $sourcePath -Destination $destinationPackage

# Clean up the logs.
$cleanup = Invoke-Command -Session $session -ScriptBlock {Remove-AzsDBAdapterLog}
# Close the session.
$session | Remove-PSSession
```

::: moniker-end

## <a name="configure-azure-diagnostics-extension-for-sql-resource-provider"></a>SQL リソース プロバイダーの Azure Diagnostics 拡張機能の構成
Azure Diagnostics 拡張機能は、既定で SQL リソース プロバイダー アダプター VM にインストールされます。 次の手順では、トラブルシューティングと監査の目的で SQL リソース プロバイダーの操作イベント ログと IIS ログを収集するために拡張機能をカスタマイズする方法について説明します。

1. Azure Stack Hub 管理者ポータルにサインインします。

2. 左側のペインで **[仮想マシン]** を選択し、SQL リソース プロバイダー アダプター VM を検索して、その VM を選択します。

3. VM の **[診断設定]** で、**[ログ]** タブにアクセスし、**[カスタム]** を選択して、収集するイベント ログをカスタマイズします。
![診断設定への移動](media/azure-stack-sql-resource-provider-maintain/sqlrp-diagnostics-settings.png)

4. SQL リソース プロバイダーの操作イベント ログを収集するために、**Microsoft-AzureStack-DatabaseAdapter/Operational!\*** を追加します。
![イベント ログの追加](media/azure-stack-sql-resource-provider-maintain/sqlrp-event-logs.png)

5. IIS ログの収集を有効にするには、**[IIS ログ]** と **[失敗した要求のログ]** をオンにします。
![IIS ログの追加](media/azure-stack-sql-resource-provider-maintain/sqlrp-iis-logs.png)

6. 最後に、 **[保存]** を選択して、すべての診断設定を保存します。

イベント ログと IIS ログの収集が SQL リソース プロバイダーに対して構成されると、**sqladapterdiagaccount** という名前のシステム ストレージ アカウント内にログが見つかります。

Azure Diagnostics 拡張機能の詳細については、「[Azure Diagnostics 拡張機能とは何か](/azure/azure-monitor/platform/diagnostics-extension-overview)」を参照してください。

## <a name="next-steps"></a>次のステップ

[サーバーをホストする SQL Server を追加する](azure-stack-sql-resource-provider-hosting-servers.md)
