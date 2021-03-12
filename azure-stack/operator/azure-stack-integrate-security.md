---
title: Syslog 転送を使用して Azure Stack Hub と監視ソリューションを統合する
description: Syslog 転送を使用して、Azure Stack Hub と監視ソリューションを統合する方法を説明します。
author: BryanLa
ms.topic: article
ms.date: 01/10/2020
ms.author: bryanla
ms.reviewer: fiseraci
ms.lastreviewed: 06/15/2020
ms.openlocfilehash: 6b1b4a206813178686b924b1de71ad957b4feb18
ms.sourcegitcommit: ccc4ee05d71496653b6e27de1bb12e4347e20ba4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102231287"
---
# <a name="integrate-azure-stack-hub-with-monitoring-solutions-using-syslog-forwarding"></a>Syslog 転送を使用して Azure Stack Hub と監視ソリューションを統合する

この記事では、Syslog を使用して Azure Stack Hub インフラストラクチャを、ご自分のデータセンターに既にデプロイされている外部セキュリティ ソリューションと統合する方法を説明します。 たとえば、セキュリティ情報イベント管理 (SIEM) システムなどです。 Syslog チャネルは、Azure Stack Hub インフラストラクチャのすべてのコンポーネントの監査、アラート、セキュリティ ログを公開します。 セキュリティ監視ソリューションと統合するため、および保存して保持できるようすべての監査、アラート、セキュリティ ログを取得するため、Syslog 転送を使用します。

Azure Stack Hub 更新プログラム 1809 以降には、構成すると、CEF (Common Event Format) のペイロードで Syslog メッセージが送信される Syslog クライアントが統合されています。

次の図は、Azure Stack Hub と外部の SIEM との統合について説明しています。 考慮する必要がある統合のパターンは 2 つあります。最初の 1 つ (青色のもの) は、インフラストラクチャの仮想マシンと HYPER-V ノードを含む Azure Stack Hub インフラストラクチャです。 すべての監査、セキュリティ ログ、およびそれらのコンポーネントからのアラートは、一元的に収集されて、CEF ペイロードを持つ Syslog を介して公開されます。 この統合パターンについては、このドキュメントのページで説明します。
2 つ目の統合パターンは、オレンジ色で示されているもので、その範囲には、ベースボード管理コントローラー (BMC)、ハードウェア ライフサイクル ホスト (HLH)、仮想マシンやハードウェア パートナーの監視および管理ソフトウェアを実行する仮想アプライアンス、トップ オブ ラック (TOR) スイッチが含まれています。 これらのコンポーネントはハードウェア パートナー固有のものであるため、それらを外部 SIEM に統合する方法に関するドキュメントについては、ハードウェア パートナーにお問い合わせください。

![Syslog 転送の図](media/azure-stack-integrate-security/azure-stack-hub-syslog-forwarding-diagram_bg.svg)

## <a name="configuring-syslog-forwarding"></a>Syslog 転送の構成

Azure Stack Hub の Syslog クライアントでは、次の構成をサポートしています。

1. **相互認証 (クライアントとサーバー) と TLS 1.2 暗号化を使用した TCP 経由の Syslog:** この構成では、Syslog サーバーと Syslog クライアントの両方が証明書を介して相互に ID を確認できます。 メッセージは、TLS 1.2 暗号化チャネル経由で送信されます。

2. **サーバー認証と TLS 1.2 暗号化を使用した TCP 経由の Syslog:** この構成では、Syslog クライアントは証明書を介して Syslog サーバーの ID を確認できます。 メッセージは、TLS 1.2 暗号化チャネル経由で送信されます。

3. **暗号化なしの TCP 経由の Syslog:** この構成では、Syslog クライアントと Syslog サーバーの ID は確認されません。 メッセージは、TCP を介してクリア テキストで送信されます。

4. **暗号化なしの UDP 経由の Syslog:** この構成では、Syslog クライアントと Syslog サーバーの ID は確認されません。 メッセージは、UDP を介してクリア テキストで送信されます。

> [!IMPORTANT]
> Microsoft は、中間者攻撃やメッセージの傍受からの保護のため、運用環境では、認証と暗号化を使用して TCP を使用する (構成 1 か最低でも 2) よう強くお勧めします。

### <a name="cmdlets-to-configure-syslog-forwarding"></a>Syslog 転送を構成するためのコマンドレット
Syslog 転送を構成するには、特権エンドポイント (PEP) へのアクセスが必要です。 Syslog 転送を構成するために、次の 2 つの PowerShell コマンドレットが PEP に追加されています。


```powershell
### cmdlet to pass the syslog server information to the client and to configure the transport protocol, the encryption and the authentication between the client and the server

Set-SyslogServer [-ServerName <String>] [-ServerPort <UInt16>] [-NoEncryption] [-SkipCertificateCheck] [-SkipCNCheck] [-UseUDP] [-Remove]

### cmdlet to configure the certificate for the syslog client to authenticate with the server

Set-SyslogClient [-pfxBinary <Byte[]>] [-CertPassword <SecureString>] [-RemoveCertificate] [-OutputSeverity]
```
#### <a name="cmdlets-parameters"></a>コマンドレットのパラメーター

*Set-SyslogServer* コマンドレットのパラメーター:

| パラメーター | 説明 | Type | 必須 |
|---------|---------|---------|---------|
|*ServerName* | Syslog サーバーの FQDN または IP アドレス。 | String | はい|
|*ServerPort* | Syslog サーバーがリッスンしているポート番号。 | UInt16 | はい|
|*NoEncryption*| クライアントに Syslog メッセージを強制的にクリア テキストで送信させます。 | フラグ | no|
|*SkipCertificateCheck*| 初期 TLS ハンドシェイク時に Syslog サーバーによって提供された証明書の検証をスキップします。 | フラグ | no|
|*SkipCNCheck*| 初期 TLS ハンドシェイク時に Syslog サーバーによって提供された証明書の共通名値の検証をスキップします。 | フラグ | no|
|*UseUDP*| トランスポート プロトコルとして UDP を使用して、Syslog を使用します。 |フラグ | no|
|*Remove*| クライアントからサーバーの構成を削除し、Syslog 転送を停止します。| フラグ | no|

*Set-SyslogClient* コマンドレットのパラメーター:

| パラメーター | 説明 | Type |
|---------|---------| ---------|
| *pfxBinary* | Syslog サーバーへの認証でクライアントが ID として使用する証明書が入った Byte[] にパイプする pfx ファイルの内容。  | Byte[] |
| *CertPassword* |  pfx ファイルに関連付けられている秘密キーをインポートするためのパスワード。 | SecureString |
|*RemoveCertificate* | クライアントから証明書を削除します。 | フラグ|
| *OutputSeverity* | 出力ログのレベル。 値は **[既定値]** または **[詳細]** です。 規定値には、重大度レベルの警告、クリティカル、またはエラーが含まれています。 詳細には、すべての重大度レベル (詳細、情報、警告、クリティカル、またはエラー) が含まれています。  | String |
### <a name="configuring-syslog-forwarding-with-tcp-mutual-authentication-and-tls-12-encryption"></a>TCP、相互認証、TLS 1.2 暗号化を使用した Syslog 転送の構成

この構成では、Azure Stack Hub の Syslog クライアントは、TLS 1.2 暗号化を使用して TCP 経由で Syslog サーバーにメッセージを転送します。 初期ハンドシェイク時には、有効で信頼された証明書がサーバーから提供されていることが、クライアントによって確認されます。 また、クライアントでは、ID の証明としてサーバーに証明書も提供されます。 この構成は、クライアントとサーバーの ID がどちらも完全に検証され、メッセージが暗号化されたチャネルで送信されるため、最も安全です。

> [!IMPORTANT]
> Microsoft は、運用環境ではこの構成を使用するよう強くお勧めします。 

TCP、相互認証、TLS 1.2 暗号化を使用した Syslog 転送を構成するには、PEP セッションで次の両方のコマンドレットを実行します。

```powershell
# Configure the server
Set-SyslogServer -ServerName <FQDN or ip address of syslog server> -ServerPort <Port number on which the syslog server is listening on>

# Provide certificate to the client to authenticate against the server
Set-SyslogClient -pfxBinary <Byte[] of pfx file> -CertPassword <SecureString, password for accessing the pfx file>
```

クライアント証明書には、Azure Stack Hub のデプロイ中に提供されたものと同じルートが必要です。 これには秘密キーを含める必要もあります。

```powershell
##Example on how to set your syslog client with the certificate for mutual authentication.
##This example script must be run from your hardware lifecycle host or privileged access workstation.

$ErcsNodeName = "<yourPEP>"
$password = ConvertTo-SecureString -String "<your cloudAdmin account password" -AsPlainText -Force
 
$cloudAdmin = "<your cloudAdmin account name>"
$CloudAdminCred = New-Object System.Management.Automation.PSCredential ($cloudAdmin, $password)
 
$certPassword = $password
$certContent = Get-Content -Path C:\cert\<yourClientCertificate>.pfx -Encoding Byte
 
$params = @{ 
    ComputerName = $ErcsNodeName 
    Credential = $CloudAdminCred 
    ConfigurationName = "PrivilegedEndpoint" 
}

$session = New-PSSession @params
 
$params = @{ 
    Session = $session 
    ArgumentList = @($certContent, $certPassword) 
}
Write-Verbose "Invoking cmdlet to set syslog client certificate..." -Verbose 
Invoke-Command @params -ScriptBlock { 
    param($CertContent, $CertPassword) 
    Set-SyslogClient -PfxBinary $CertContent -CertPassword $CertPassword }
```

### <a name="configuring-syslog-forwarding-with-tcp-server-authentication-and-tls-12-encryption"></a>TCP、サーバー認証、TLS 1.2 暗号化を使用した Syslog 転送の構成

この構成では、Azure Stack Hub の Syslog クライアントは、TLS 1.2 暗号化を使用して TCP 経由で Syslog サーバーにメッセージを転送します。 初期ハンドシェイク時には、有効で信頼された証明書がサーバーから提供されていることも、クライアントによって確認されます。 この構成により、信頼されていない宛先にクライアントからメッセージを送信することができなくなります。
認証と暗号化を使用する TCP は既定の構成であり、Microsoft が運用環境にお勧めする最小限のセキュリティのレベルです。

```powershell
Set-SyslogServer -ServerName <FQDN or ip address of syslog server> -ServerPort <Port number on which the syslog server is listening on>
```

お使いの Syslog サーバーと Azure Stack Hub クライアントの統合をテストするときに、自己署名証明書や信頼されていない証明書を使用する場合は、次のフラグを使用すると、最初のハンドシェイク時にクライアントが行うサーバー検証をスキップできます。

```powershell
 #Skip validation of the Common Name value in the server certificate. Use this flag if you provide an IP address for your syslog server
 Set-SyslogServer -ServerName <FQDN or ip address of syslog server> -ServerPort <Port number on which the syslog server is listening on>
 -SkipCNCheck

 #Skip entirely the server certificate validation
 Set-SyslogServer -ServerName <FQDN or ip address of syslog server> -ServerPort <Port number on which the syslog server is listening on>
 -SkipCertificateCheck
```

> [!IMPORTANT]
> Microsoft は、運用環境には -SkipCertificateCheck フラグを使用しないようお勧めします。 

### <a name="configuring-syslog-forwarding-with-tcp-and-no-encryption"></a>TCP を使用した暗号化なしの Syslog 転送の構成

この構成では、Azure Stack Hub の Syslog クライアントから Syslog サーバーに暗号化せずに TCP 経由でメッセージが転送されます。 クライアントによってサーバーの ID が確認されることはなく、クライアントから検証用に自分の ID がサーバーに提供されることもありません。

```powershell
Set-SyslogServer -ServerName <FQDN or ip address of syslog server> -ServerPort <Port number on which the syslog server is listening on> -NoEncryption
```

> [!IMPORTANT]
> Microsoft は、運用環境ではこの構成を使用しないようお勧めします。


### <a name="configuring-syslog-forwarding-with-udp-and-no-encryption"></a>UDP を使用した暗号化なしの Syslog 転送の構成

この構成では、Azure Stack Hub の Syslog クライアントから Syslog サーバーに暗号化せずに UDP 経由でメッセージが転送されます。 クライアントによってサーバーの ID が確認されることはなく、クライアントから検証用に自分の ID がサーバーに提供されることもありません。

```powershell
Set-SyslogServer -ServerName <FQDN or ip address of syslog server> -ServerPort <Port number on which the syslog server is listening on> -UseUDP
```

暗号化を使用しない UDP は構成が最も簡単ですが、中間者攻撃やメッセージの傍受に対する保護機能はありません。

> [!IMPORTANT]
> Microsoft は、運用環境ではこの構成を使用しないようお勧めします。


## <a name="removing-syslog-forwarding-configuration"></a>Syslog 転送構成の削除

Syslog サーバー構成を完全に削除し、Syslog 転送を停止するには、次を実行します。

**クライアントから Syslog サーバー構成を削除する**

```powershell  
Set-SyslogServer -Remove
```

**クライアントからクライアント証明書を削除する**

```powershell  
Set-SyslogClient -RemoveCertificate
```

## <a name="verifying-the-syslog-setup"></a>Syslog のセットアップの確認

Syslog クライアントを Syslog サーバーに接続することに成功したら、すぐにイベントの受信を開始しなければなりません。 イベントが何も表示されない場合は、次のコマンドレットを実行して、Syslog クライアントの構成を確認します。

**Syslog クライアントのサーバー構成を確認する**

```powershell  
Get-SyslogServer
```

**Syslog クライアントの証明書のセットアップを確認する**

```powershell  
Get-SyslogClient
```

## <a name="syslog-message-schema"></a>Syslog メッセージ スキーマ

Azure Stack Hub インフラストラクチャの Syslog は、CEF (Common Event Format) で書式設定され転送されます。
各 Syslog メッセージの構造は、次のスキーマに基づいています。

```Syslog
<Time> <Host> <CEF payload>
```

CEF ペイロードは、以下の構造に基づいていますが、各フィールドのマッピングは、メッセージの種類 (Windows イベント、作成されたアラート、解決されたアラート) によって異なります。

```CEF
# Common Event Format schema
CEF: <Version>|<Device Vendor>|<Device Product>|<Device Version>|<Signature ID>|<Name>|<Severity>|<Extensions>
* Version: 0.0
* Device Vendor: Microsoft
* Device Product: Microsoft Azure Stack Hub
* Device Version: 1.0
```

### <a name="cef-mapping-for-privileged-endpoint-events"></a>特権エンドポイント イベントの CEF マッピング

```
Prefix fields
* Signature ID: Microsoft-AzureStack-PrivilegedEndpoint: <PEP Event ID>
* Name: <PEP Task Name>
* Severity: mapped from PEP Level (details see the PEP Severity table below)
* Who: account used to connect to the PEP
* WhichIP: IP address of the device used to connect to the PEP
```

特権エンドポイントのイベントの表:

| Event | PEP イベント ID | PEP タスク名 | 重大度 |
|-------|--------------| --------------|----------|
|PrivilegedEndpointAccessed|1000|PrivilegedEndpointAccessedEvent|5|
|SupportSessionTokenRequested |1001|SupportSessionTokenRequestedEvent|5|
|SupportSessionDevelopmentTokenRequested |1002|SupportSessionDevelopmentTokenRequestedEvent|5|
|SupportSessionUnlocked |1003|SupportSessionUnlockedEvent|10|
|SupportSessionFailedToUnlock |1004|SupportSessionFailedToUnlockEvent|10|
|PrivilegedEndpointClosed |1005|PrivilegedEndpointClosedEvent|5|
|NewCloudAdminUser |1006|NewCloudAdminUserEvent|10|
|RemoveCloudAdminUser |1007|RemoveCloudAdminUserEvent|10|
|SetCloudAdminUserPassword |1008|SetCloudAdminUserPasswordEvent|5|
|GetCloudAdminPasswordRecoveryToken |1009|GetCloudAdminPasswordRecoveryTokenEvent|10|
|ResetCloudAdminPassword |1010|ResetCloudAdminPasswordEvent|10|
|PrivilegedEndpointSessionTimedOut |1017|PrivilegedEndpointSessionTimedOutEvent|5|

PEP 重大度の表:

| 重大度 | Level | 数値 |
|----------|-------| ----------------|
|0|未定義。|値:0。 すべてのレベルのログを示します|
|10|Critical|値:1. 重大なアラートのログを示します|
|8|エラー| 値:2. エラーのログを示します|
|5|警告|値:3. 警告のログを示します|
|2|Information|値:4. 情報メッセージのログを示します|
|0|"詳細"|値:5. すべてのレベルのログを示します|

### <a name="cef-mapping-for-recovery-endpoint-events"></a>復旧エンドポイント イベントの CEF マッピング

```
Prefix fields
* Signature ID: Microsoft-AzureStack-PrivilegedEndpoint: <REP Event ID>
* Name: <REP Task Name>
* Severity: mapped from REP Level (details see the REP Severity table below)
* Who: account used to connect to the REP
* WhichIP: IP address of the device used to connect to the REP
```

復旧エンドポイントのイベントの表:

| Event | REP イベント ID | REP タスク名 | 重大度 |
|-------|--------------| --------------|----------|
|RecoveryEndpointAccessed |1011|RecoveryEndpointAccessedEvent|5|
|RecoverySessionTokenRequested |1012|RecoverySessionTokenRequestedEvent |5|
|RecoverySessionDevelopmentTokenRequested |1013|RecoverySessionDevelopmentTokenRequestedEvent|5|
|RecoverySessionUnlocked |1014|RecoverySessionUnlockedEvent |10|
|RecoverySessionFailedToUnlock |1015|RecoverySessionFailedToUnlockEvent|10|
|RecoveryEndpointClosed |1016|RecoveryEndpointClosedEvent|5|

REP 重大度の表:

| 重大度 | Level | 数値 |
|----------|-------| ----------------|
|0|未定義。|値:0。 すべてのレベルのログを示します|
|10|Critical|値:1. 重大なアラートのログを示します|
|8|エラー| 値:2. エラーのログを示します|
|5|警告|値:3. 警告のログを示します|
|2|Information|値:4. 情報メッセージのログを示します|
|0|"詳細"|値:5. すべてのレベルのログを示します|

### <a name="cef-mapping-for-windows-events"></a>Windows イベントの CEF マッピング

```
* Signature ID: ProviderName:EventID
* Name: TaskName
* Severity: Level (for details, see the severity table below)
* Extension: Custom Extension Name (for details, see the Custom Extension table below)
```

Windows イベントの重大度の表:

| CEF 重大度の値 | Windows イベント レベル | 数値 |
|--------------------|---------------------| ----------------|
|0|未定義。|値:0。 すべてのレベルのログを示します|
|10|Critical|値:1. 重大なアラートのログを示します|
|8|エラー| 値:2. エラーのログを示します|
|5|警告|値:3. 警告のログを示します|
|2|Information|値:4. 情報メッセージのログを示します|
|0|"詳細"|値:5. すべてのレベルのログを示します|

Azure Stack Hub の Windows イベントのカスタム拡張機能の表:

| カスタム拡張機能名 | Windows イベントの例 | 
|-----------------------|---------|
|MasChannel | システム|
|MasComputer | test.azurestack.contoso.com|
|MasCorrelationActivityID| C8F40D7C-3764-423B-A4FA-C994442238AF|
|MasCorrelationRelatedActivityID| C8F40D7C-3764-423B-A4FA-C994442238AF|
|MasEventData| svchost!!4132,G,0!!!!EseDiskFlushConsistency!!ESENT!!0x800000|
|MasEventDescription| ユーザーのグループ ポリシー設定は正しく処理されました。 前回グループ ポリシーが正しく処理されてからの変更は検出されませんでした。|
|MasEventID|1501|
|MasEventRecordID|26637|
|MasExecutionProcessID | 29380|
|MasExecutionThreadID |25480|
|MasKeywords |0x8000000000000000|
|MasKeywordName |成功の監査|
|MasLevel |4|
|MasOpcode |1|
|MasOpcodeName |info|
|MasProviderEventSourceName ||
|MasProviderGuid |AEA1B4FA-97D1-45F2-A64C-4D69FFFD92C9|
|MasProviderName |Microsoft-Windows-GroupPolicy|
|MasSecurityUserId |\<Windows SID\> |
|MasTask |0|
|MasTaskCategory| プロセス作成|
|MasUserData|KB4093112!!5112!!Installed!!0x0!!WindowsUpdateAgent Xpath: /Event/UserData/*|
|MasVersion|0|

### <a name="cef-mapping-for-alerts-created"></a>作成されたアラートの CEF マッピング

```
* Signature ID: Microsoft Azure Stack Hub Alert Creation : FaultTypeId
* Name: FaultTypeId : AlertId
* Severity: Alert Severity (for details, see alerts severity table below)
* Extension: Custom Extension Name (for details, see the Custom Extension table below)
```

アラートの重大度の表:

| 重大度 | Level |
|----------|-------|
|0|未定義。|
|10|Critical|
|5|警告|

Azure Stack Hub の作成されたアラートのカスタム拡張機能の表:

| カスタム拡張機能名 | 例 | 
|-----------------------|---------|
|MasEventDescription|説明: \<TestDomain\> のユーザー アカウント \<TestUser\> が作成されました。 これは潜在的なセキュリティ リスクです。 -- 解決策: サポートにお問い合わせください。 この問題を解決するにはカスタマー サポートが必要です。 サポートを受けずに、この問題を解決しようとしないでください。 サポート リクエストを開く前に、https://aka.ms/azurestacklogfiles のガイダンスを使用してログ ファイルの収集プロセスを開始してください。

### <a name="cef-mapping-for-alerts-closed"></a>解決されたアラートの CEF マッピング

```
* Signature ID: Microsoft Azure Stack Hub Alert Creation : FaultTypeId
* Name: FaultTypeId : AlertId
* Severity: Information
```

次の例は、CEF ペイロードによる Syslog メッセージを示しています。
```
2018:05:17:-23:59:28 -07:00 TestHost CEF:0.0|Microsoft|Microsoft Azure Stack Hub|1.0|3|TITLE: User Account Created -- DESCRIPTION: A user account \<TestUser\> was created for \<TestDomain\>. It's a potential security risk. -- REMEDIATION: Please contact Support. Customer Assistance is required to resolve this issue. Do not try to resolve this issue without their assistance. Before you open a support request, start the log file collection process using the guidance from https://aka.ms/azurestacklogfiles|10
```

## <a name="syslog-event-types"></a>Syslog イベントの種類  

Syslog チャネルを介して送信されるすべてのイベントの種類、イベント、メッセージ スキーマ、またはプロパティを次の表に示します。 設定の verbose スイッチは、SIEM 統合に Windows 情報イベントが必要な場合にのみ使用してください。 

| イベントの種類                                 | イベントまたはメッセージ スキーマ                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | verbose 設定が必要         | イベントの説明 (省略可能)                                                                                                                                   |
|--------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Azure Stack Hub のアラート                     | アラート メッセージ スキーマについては、「[解決されたアラートの CEF マッピング](#cef-mapping-for-alerts-closed)」を参照してください。 <br> <br>別のドキュメントに、すべてのアラートの一覧がまとめられています。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | いいえ                               | システムの正常性のアラート                                                                                                                                           |
| 特権エンドポイント イベント                 | 特権エンドポイントのメッセージ スキーマについては、「[特権エンドポイント イベントの CEF マッピング](#cef-mapping-for-privileged-endpoint-events)」を参照してください。<br> <br>PrivilegedEndpointAccessed <br>SupportSessionTokenRequested <br>SupportSessionDevelopmentTokenRequested <br>SupportSessionUnlocked <br>SupportSessionFailedToUnlock <br>PrivilegedEndpointClosed <br>NewCloudAdminUser <br>RemoveCloudAdminUser <br>SetCloudAdminUserPassword <br>GetCloudAdminPasswordRecoveryToken <br>ResetCloudAdminPassword <br>PrivilegedEndpointSessionTimedOut                                                                                                                                                                                                                                                                                                      | いいえ                               |                                                                                                                                                                |
| 復旧エンドポイント イベント                   | 復旧エンドポイントのメッセージ スキーマについては、「[復旧エンドポイント イベントの CEF マッピング](#cef-mapping-for-recovery-endpoint-events)」を参照してください。 <br>RecoveryEndpointAccessed <br>RecoverySessionTokenRequested <br>RecoverySessionDevelopmentTokenRequested <br>RecoverySessionUnlocked <br>RecoverySessionFailedToUnlock <br>Recovand RecoveryEndpointClosed                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | いいえ                               |                                                                                                                                                                |
| Windows セキュリティ イベント                    |   <br>Windows イベントのメッセージ スキーマについては、「[Windows イベントの CEF マッピング](#cef-mapping-for-windows-events)」を参照してください。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | はい (情報イベントを取得するため)  | 種類: <br>- 情報 <br>- 警告 <br>- エラー <br>- Critical (重大)                                                                                               |
| ARM のイベント                                 | メッセージのプロパティ: <br> <br>AzsSubscriptionId <br>AzsCorrelationId <br>AzsPrincipalOid <br>AzsPrincipalPuid <br>AzsTenantId <br>AzsOperationName <br>AzsOperationId <br>AzsEventSource <br>AzsDescription <br>AzsResourceProvider <br>AzsResourceUri <br>AzsEventName <br>AzsEventInstanceId <br>AzsChannels <br>AzsEventLevel <br>AzsStatus <br>AzsSubStatus <br>AzsClaims <br>AzsAuthorization <br>AzsHttpRequest <br>AzsProperties <br>AzsEventTimestamp <br>AzsAudience <br>AzsIssuer <br>AzsIssuedAt <br>AzsApplicationId <br>AzsUniqueTokenId <br>AzsArmServiceRequestId <br>AzsEventCategory <br> <br>                                                                                                                                                                                                                                | いいえ <br>                          | 登録されている各 ARM リソースによってイベントを発生させることができます。                                                                                                               |
| BCDR イベント                                | メッセージ スキーマ: <br> <br>AuditingManualBackup { <br>} <br>AuditingConfig <br>{ <br>Interval <br>Retention <br>IsSchedulerEnabled <br>BackupPath <br>} <br>AuditingPruneBackupStore { <br>IsInternalStore <br>} <br>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | いいえ                               | これらのイベントでは、ユーザーが手動で実行したインフラのバックアップ管理操作 (トリガーのバックアップ、バックアップ構成の変更、バックアップ データの排除など) が追跡されます。       |
| インフラ障害によるイベントの作成と終了    | メッセージ スキーマ: <br> <br>InfrastructureFaultOpen { <br>AzsFaultId, <br>AzsFaultTypeName, <br>AzsComponentType, <br>AzsComponentName, <br>AzsFaultHash, <br>AzsCreatedTimeUtc, <br>AzsSource <br>} <br>  <br>InfrastructureFaultClose {  <br>AzsFaultId, <br>AzsFaultTypeName, <br>AzsComponentType, <br>AzsComponentName, <br>AzsFaultHash, <br>AzsLastUpdatedTimeUtc, <br>AzsSource <br>}                                                                                                                                                                                                                                                                                                                                                                                                                                                    | いいえ                               | 障害によって、アラートにつながる可能性があるエラーの修復を試みるワークフローがトリガーされます。 障害に修復策がない場合は、そのままアラートが発生します。            |
| サービス障害によるイベントの作成と終了  | メッセージ スキーマ: <br> <br>ServiceFaultOpen { <br>AzsFaultId, <br>AzsFaultTypeName, <br>AzsSubscriptionId, <br>AzsResourceGroup, <br>AzsServiceName, <br>AzsResourceId <br>AzsFaultHash, <br>AzsCreatedTimeUtc, <br>AzsSource <br>} <br>  <br>ServiceFaultClose { <br>AzsFaultId, <br>AzsFaultTypeName, <br>AzsSubscriptionId, <br>AzsResourceGroup, <br>AzsServiceName, <br>AzsResourceId <br>AzsFaultHash, <br>AzsLastUpdatedTimeUtc, <br>AzsSource <br>}                                                                                                                                                                                                                                                                                                                                                                                     | いいえ                               | 障害によって、アラートにつながる可能性があるエラーの修復を試みるワークフローがトリガーされます。  <br>障害に修復策がない場合は、そのままアラートが発生します。 <br>  |
| PEP WAC イベント                             | メッセージ スキーマ: <br> <br>プレフィックス フィールド  <br>* Signature ID: Microsoft-AzureStack-PrivilegedEndpoint: <PEP Event ID>  <br>* Name: <PEP Task Name>  <br>* Severity: PEP レベルからマップされます (詳細については、次の PEP 重大度テーブルを参照してください)  <br>* Who: PEP への接続に使用されるアカウント  <br>* WhichIP: PEP への接続に使用されるデバイスの IP アドレス <br><br>WACServiceStartFailedEvent <br>WACConnectedUserNotRetrievedEvent <br>WACEnableExceptionEvent  <br>WACUserAddedEvent <br>WACAddUserToLocalGroupFailedEvent <br>WACIsUserInLocalGroupFailedEvent  <br>WACServiceStartTimeoutEvent  <br>WACServiceStartInvalidOperationEvent <br>WACGetSidFromUserFailedEvent <br>WACDisableFirewallFailedEvent <br>WACCreateLocalGroupIfNotExistFailedEvent <br>WACEnableFlagIsTrueEvent <br>WACEnableFlagIsFalseEvent <br>WACServiceStartedEvent | いいえ                               |                                                                                                                                                                |

## <a name="next-steps"></a>次のステップ

[サービス ポリシー](azure-stack-servicing-policy.md)
