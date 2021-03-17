---
title: シークレットの切り替え
titleSuffix: Azure Stack Hub
description: Azure Stack Hub でシークレットをローテーションする方法について説明します。
author: BryanLa
ms.topic: how-to
ms.date: 03/09/2021
ms.reviewer: fiseraci
ms.author: bryanla
ms.lastreviewed: 01/19/2021
monikerRange: '>=azs-1803'
ms.openlocfilehash: d473301265c8b7c1c3b48386cfbc8f4986283a8e
ms.sourcegitcommit: e7d6f953e7014900b4e7d710340cfa98d253fce9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/11/2021
ms.locfileid: "102637597"
---
# <a name="rotate-secrets-in-azure-stack-hub"></a>Azure Stack Hub でシークレットをローテーションする

この記事では、Azure Stack Hub インフラストラクチャのリソースおよびサービスとのセキュリティで保護された通信の維持に役立つ、シークレットのローテーション実行に関するガイダンスを提供します。

## <a name="overview"></a>概要

Azure Stack Hub はシークレットを使用して、インフラストラクチャのリソースおよびサービスとのセキュリティで保護された通信を維持します。 Azure Stack Hub インフラストラクチャの整合性を維持するために、オペレーターは、組織のセキュリティ要件と一致する頻度でシークレットをローテーションできる必要があります。

シークレットの有効期限が近づくと、管理者ポータルで次のアラートが生成されます。 シークレットのローテーションを完了すると、次のアラートが解決されます。

- 保留中のサービス アカウントのパスワードの有効期限
- 保留中の内部証明書の有効期限
- 保留中の外部証明書の有効期限

> [!WARNING]
> 有効期限が切れる前に管理者ポータルでトリガーされるアラートには、次の 2 つのフェーズがあります。
> - 有効期限の 90 日前になると、警告アラートが生成されます。
> - 有効期限の 30 日前になると、重要なアラートが生成されます。 
>
> **これらの通知を受け取った場合は、シークレットのローテーションを完了することが "*重要*" です。これを行わないと、ワークロードが失われたり、Azure Stack Hub の再デプロイを自分の費用で行うことになる可能性があります。**

アラートの監視と修復の詳細については、「[Azure Stack Hub での正常性およびアラートの監視](azure-stack-monitor-health.md)」を参照してください。

::: moniker range="<azs-1811"  
> [!NOTE]
> 1811 より前の Azure Stack Hub 環境では、保留中の内部証明書やシークレットの有効期限のアラートが表示される場合があります。 これらのアラートは正確ではないため、内部シークレットのローテーションを実行せずに無視する必要があります。 正しくない内部シークレットの有効期限切れアラートは、1811 で解決される既知の問題です。 環境が 2 年間アクティブになっていない限り、内部シークレットの有効期限が切れることはありません。
::: moniker-end

## <a name="prerequisites"></a>前提条件

1. Azure Stack Hub のサポートされているバージョンを実行することを強くお勧めします。また、インスタンスが実行されている Azure Stack Hub バージョンに使用可能な最新の修正プログラムが適用されていることを確認してください。 たとえば、2008 を実行している場合は、2008 で使用可能な最新の修正プログラムがインストールされていることを確認します。

    ::: moniker range="<azs-1811"  
    >[!IMPORTANT]
    > 1811 より前のバージョンの場合:
    > - シークレットのローテーションが既に実行されている場合、バージョン 1811 以降に更新してから、シークレットのローテーションを再び実行する必要があります。 シークレットのローテーションは、[特権エンドポイント](azure-stack-privileged-endpoint.md)を使用して実行する必要があり、Azure Stack Hub オペレーターの資格情報が必要です。 ご使用の環境でシークレットのローテーションが実行されているかどうかがわからない場合は、シークレットのローテーションを実行する前に 1811 に更新してください。
    > - 拡張機能ホストの証明書を追加するためにシークレットのローテーションを行う必要はありません。 拡張機能ホストの証明書を追加するには、「[Azure Stack Hub の拡張機能ホストを準備する](azure-stack-extension-host-prepare.md)」の手順に従ってください。
    ::: moniker-end

2. 計画メンテナンス操作をユーザーに通知します。 可能な限り非営業時間中に、通常のメンテナンス期間をスケジュールします。 メンテナンス操作は、ユーザーのワークロードとポータル操作の両方に影響を及ぼす可能性があります。

3. シークレットのローテーション中に、アラートが開いて自動的に閉じることにオペレーターが気付くことがあります。 これは予期される動作であり、アラートは無視してもかまいません。 オペレーターは、[PowerShell コマンドレットの Test-AzureStack](azure-stack-diagnostic-test.md) を使用して、これらのアラートの有効性を確認できます。 System Center Operations Manager を使用して Azure Stack Hub システムを監視しているオペレーターの場合、システムをメンテナンス モードにすると、これらのアラートは ITSM システムに到達しなくなりますが、Azure Stack Hub システムに到達できなくなっても引き続きアラートは生成されます。

::: moniker range=">=azs-1811"
## <a name="rotate-external-secrets"></a>外部シークレットをローテーションする

> [!Important]
> 外部シークレットのローテーション:
> - **セキュリティで保護されたキーや文字列などの証明書ではないシークレット** は、管理者が手動で実行する必要があります。 これには、ユーザーと管理者のアカウントのパスワード、[ネットワーク スイッチのパスワード](azure-stack-customer-defined.md)などがあります。
> - **付加価値リソース プロバイダー (RP) のシークレット** については、個別のガイダンスで説明されています。
>    - [Azure Stack Hub 上の App Service](app-service-rotate-certificates.md)
>    - [Azure Stack Hub 上の Event Hubs](event-hubs-rp-rotate-secrets.md)
>    - [Azure Stack Hub 上の IoT Hub](iot-hub-rp-rotate-secrets.md)
>    - [Azure Stack Hub 上の MySQL](azure-stack-mysql-resource-provider-maintain.md#secrets-rotation)
>    - [Azure Stack Hub 上の SQL](azure-stack-sql-resource-provider-maintain.md#secrets-rotation)
> - **ベースボード管理コントローラー (BMC) の資格情報** も手動プロセスであり、[この記事の後半で説明します](#update-the-bmc-credential)。 

このセクションでは、外部向けサービスをセキュリティで保護するために使用される証明書のローテーションについて説明します。 これらの証明書は、次のサービスのために Azure Stack Hub Operator によって提供されます。

- 管理者ポータル
- パブリック ポータル
- 管理者の Azure Resource Manager
- グローバルな Azure Resource Manager
- 管理者の Key Vault
- Key Vault
- 管理者拡張機能ホスト
- ACS (Blob、Table、Queue Storage を含みます)
- ADFS<sup>*</sup>
- Graph<sup>*</sup>

<sup>*</sup>Active Directory フェデレーション サービス (AD FS) の使用時に適用されます。

### <a name="preparation"></a>準備

外部シークレットのローテーション前:

1. シークレットをローテーションする前に、`-group SecretRotationReadiness` パラメーターを使用して **[`Test-AzureStack`](azure-stack-diagnostic-test.md)** PowerShell コマンドレットを実行し、すべてのテスト出力が正常であることを確認します。
2. 代わりの外部証明書の新しいセットを準備します。
   - 新しいセットは、[Azure Stack Hub PKI 証明書の要件](azure-stack-pki-certs.md)に関する記事に記載されている証明書の仕様に一致している必要があります。 
   - 証明機関 (CA) に送信する証明書署名要求 (CSR) を生成します。 [証明書署名要求の生成](azure-stack-get-pki-certs.md)に関する記事に記載された手順を使用します。また、[PKI 証明書の準備](azure-stack-prepare-pki-certs.md)に関する記事の手順を使用して、これらの要求を Azure Stack Hub 環境で使用するために準備します。 Azure Stack Hub では、次のようなコンテキストで、新しい証明書機関 (CA) からの外部証明書を使用したシークレットのローテーションがサポートされます。

     |CA からのローテーション|CA へのローテーション|Azure Stack Hub バージョンのサポート|
     |-----|-----|-----|-----|
     |自己署名|Enterprise| 1903 以降|
     |自己署名|自己署名|サポートされていません|
     |自己署名|パブリック<sup>*</sup>|1803 以降|
     |Enterprise|Enterprise|1803 以降。デプロイ時に使用されるものと同じエンタープライズ CA の場合は 1803-1903|
     |Enterprise|自己署名|サポートされていません|
     |Enterprise|パブリック<sup>*</sup>|1803 以降|
     |パブリック<sup>*</sup>|Enterprise|1903 以降|
     |パブリック<sup>*</sup>|自己署名|サポートされていません|
     |パブリック<sup>*</sup>|パブリック<sup>*</sup>|1803 以降|

     <sup>*</sup>[Windows の信頼されたルート プログラム](/security/trusted-root/participants-list)の一部。

   - 準備する証明書は、[PKI 証明書の検証](azure-stack-validate-pki-certs.md)に関する記事に記載された手順を使用して必ず検証してください。
   - パスワードに `*` や `)` などの特殊文字が含まれないことを確認します。
   - PFX 暗号化が **TripleDES-SHA1** であることを確認します。 問題が発生した場合は、「[Azure Stack Hub PKI 証明書に関する一般的な問題を修復する](azure-stack-remediate-certs.md#pfx-encryption)」を参照してください。

3. ローテーションに使われる証明書のバックアップを安全なバックアップ場所に格納します。 ローテーションを実行して失敗した場合は、ローテーションを再実行する前に、ファイル共有内の証明書をバックアップ コピーに置き換えます。 バックアップ コピーはセキュリティで保護されたバックアップ場所に保存してください。
4. ERCS VM からアクセスできるファイル共有を作成します。 ファイル共有は、**CloudAdmin** ID で読み書きできる必要があります。
5. ファイル共有にアクセスできるコンピューターから PowerShell ISE コンソールを開きます。 ファイル共有に移動します。ここで、外部証明書を配置するディレクトリを作成します。
6. **[CertDirectoryMaker.ps1](https://www.aka.ms/azssecretrotationhelper)** をネットワーク ファイル共有にダウンロードし、スクリプトを実行します。 このスクリプトにより、ID プロバイダーに応じて **_.\Certificates\AAD_ *_ または _* _.\Certificates\ADFS_ *_ に準拠するフォルダー構造が作成されます。フォルダー構造は、_* \\Certificates** フォルダーで始まり、その後に、 **\\AAD** または **\\ADFS** フォルダーのみが続く必要があります。 残りのすべてのサブディレクトリは、上記の構造内に含まれます。 次に例を示します。
    - ファイル共有 = **\\\\\<IPAddress>\\\<ShareName>**
    - Azure AD プロバイダーの証明書ルート フォルダー = **\\Certificates\AAD**
    - 完全なパス = **\\\\\<IPAddress>\\\<ShareName>\Certificates\AAD**

    > [!IMPORTANT]
    > 後で `Start-SecretRotation` を実行すると、フォルダー構造が検証されます。 準拠していないフォルダー構造の場合、次のエラーがスローされます。
    >
    > ```powershell
    > Cannot bind argument to parameter 'Path' because it is null.
    > + CategoryInfo          : InvalidData: (:) [Test-Certificate], ParameterBindingValidationException
    > + FullyQualifiedErrorId : ParameterArgumentValidationErrorNullNotAllowed,Test-Certificate
    > + PSComputerName        : xxx.xxx.xxx.xxx
    > ```

7. 手順 2 で作成した代替の外部証明書の新しいセットを、手順 6 で作成した **\Certificates\\\<IdentityProvider>** ディレクトリにコピーします。 \<CertName\> の `cert.<regionName>.<externalFQDN>` 形式に必ず従ってください。 

    Azure AD ID プロバイダーのフォルダー構造の例を次に示します。
    ```powershell
        <ShareName>
            │
            └───Certificates
                  └───AAD
                      ├───ACSBlob
                      │       <CertName>.pfx
                      │
                      ├───ACSQueue
                      │       <CertName>.pfx
                      │
                      ├───ACSTable
                      │       <CertName>.pfx
                      │
                      ├───Admin Extension Host
                      │       <CertName>.pfx
                      │
                      ├───Admin Portal
                      │       <CertName>.pfx
                      │
                      ├───ARM Admin
                      │       <CertName>.pfx
                      │
                      ├───ARM Public
                      │       <CertName>.pfx
                      │
                      ├───KeyVault
                      │       <CertName>.pfx
                      │
                      ├───KeyVaultInternal
                      │       <CertName>.pfx
                      │
                      ├───Public Extension Host
                      │       <CertName>.pfx
                      │
                      └───Public Portal
                              <CertName>.pfx

    ```

### <a name="rotation"></a>回転

外部シークレットをローテーションするには、次の手順を実行します。

1. 次の PowerShell スクリプトを使用して、シークレットをローテーションします。 このスクリプトには、特権エンドポイント (PEP) セッションへのアクセスが必要です。 PEP には、PEP をホストする仮想マシン (VM) 上のリモート PowerShell セッションを介してアクセスします。 統合システムを使用している場合、PEP の 3 つのインスタンスがあり、それぞれ異なるホスト上の VM (Prefix-ERCS01、Prefix-ERCS02、または Prefix-ERCS03) 内で動作しています。 ASDK を使用している場合、この VM は AzS-ERCS01 という名前になります。 次を実行する前に、`<placeholder>` の値を更新します。

    ```powershell
    # Create a PEP Session
    winrm s winrm/config/client '@{TrustedHosts= "<IP_address_of_ERCS>"}'
    $PEPCreds = Get-Credential
    $PEPSession = New-PSSession -ComputerName <IP_address_of_ERCS_Machine> -Credential $PEPCreds -ConfigurationName "PrivilegedEndpoint"

    # Run Secret Rotation
    $CertPassword = ConvertTo-SecureString "<Cert_Password>" -AsPlainText -Force
    $CertShareCreds = Get-Credential
    $CertSharePath = "<Network_Path_Of_CertShare>"
    Invoke-Command -Session $PEPSession -ScriptBlock {
        Start-SecretRotation -PfxFilesPath $using:CertSharePath -PathAccessCredential $using:CertShareCreds -CertificatePassword $using:CertPassword
    }
    Remove-PSSession -Session $PEPSession
    ```

    このスクリプトは、次のステップを実行します。

    - **CloudAdmin** アカウントを使用して [特権エンドポイント](azure-stack-privileged-endpoint.md)で PowerShell セッションを作成し、セッションを変数として格納します。 この変数は、次の手順でパラメーターとして使用されます。 

    - [Invoke-Command](/powershell/module/microsoft.powershell.core/Invoke-Command) を実行し、PEP セッション変数を `-Session` パラメーターとして渡します。

    - 次のパラメーターを使用して、PEP セッションで `Start-SecretRotation` を実行します。
        - `-PfxFilesPath`:前に作成した Certificates ディレクトリへのネットワーク パス。  
        - `-PathAccessCredential`:共有に対する資格情報の PSCredential オブジェクト。
        - `-CertificatePassword`:作成されるすべての pfx 証明書ファイルに使われるパスワードのセキュリティで保護された文字列です。

2. 外部シークレットのローテーションには、約 1 時間かかります。 正常に完了すると、コンソールに `ActionPlanInstanceID ... CurrentStatus: Completed` メッセージが表示され、その後に `DONE` が表示されます。 「準備」セクションで作成した共有から証明書を削除し、セキュリティで保護されたバックアップ場所に保存します。

    > [!Note]
    > シークレットのローテーションが失敗した場合は、エラー メッセージの指示に従い、 `-ReRun` パラメーターを指定して `Start-SecretRotation` を再実行します。
    >
    >```powershell
    >Start-SecretRotation -ReRun
    >```  
    >
    >シークレットのローテーションの失敗が繰り返される場合は、サポートにお問い合わせください。
::: moniker-end

## <a name="rotate-internal-secrets"></a>内部シークレットをローテーションする

内部シークレットには、Azure Stack Hub オペレーターの介入なしに Azure Stack Hub インフラストラクチャによって使用されるすべての証明書、パスワード、セキュリティで保護された文字列、キーなどがあります。 内部シークレットのローテーションは、それが侵害された可能性がある場合、または有効期限アラートを受け取った場合にのみ必要です。 
::: moniker range="<azs-1811"  
1811 より前のデプロイでは、保留中の内部証明書やシークレットの有効期限に関するアラートが表示される場合があります。 これらのアラートは正確ではないため、無視してください。これらは既知の問題であり、1811 で解決されています。
::: moniker-end

内部シークレットをローテーションするには、次の手順を実行します。

1. 次の PowerShell スクリプトを実行します。 内部シークレットのローテーションの場合、シークレットのローテーション実行に関するセクションで [Start-SecretRotation コマンドレット](../reference/pep-2002/start-secretrotation.md)の `-Internal` パラメーターのみを使用します。

    ```powershell
    # Create a PEP Session
    winrm s winrm/config/client '@{TrustedHosts= "<IP_address_of_ERCS>"}'
    $PEPCreds = Get-Credential
    $PEPSession = New-PSSession -ComputerName <IP_address_of_ERCS_Machine> -Credential $PEPCreds -ConfigurationName "PrivilegedEndpoint"

    # Run Secret Rotation
    Invoke-Command -Session $PEPSession -ScriptBlock {
        Start-SecretRotation -Internal
    }
    Remove-PSSession -Session $PEPSession
    ```

    ::: moniker range="<azs-1811"
    > [!Note]
    > 1811 より前のバージョンでは、`-Internal` フラグは必要ありません。 
    ::: moniker-end


2. 正常に完了すると、コンソールに `ActionPlanInstanceID ... CurrentStatus: Completed` メッセージが表示され、その後に `DONE` が表示されます。

    > [!Note]
    > シークレット ローテーションが失敗した場合は、エラー メッセージ内の指示に従い、 `-Internal` パラメーターと `-ReRun` パラメーターを付けて `Start-SecretRotation` を再実行します。  
    >
    >```powershell
    >Start-SecretRotation -Internal -ReRun
    >```
    >
    > シークレットのローテーションの失敗が繰り返される場合は、サポートにお問い合わせください。

## <a name="update-the-bmc-credential"></a>BMC 資格情報を更新する

ベースボード管理コントローラーは、サーバーの物理的な状態を監視します。 BMC のユーザー アカウント名とパスワードを更新する手順については、OEM (Original Equipment Manufacturer) ハードウェア ベンダーを参照してください。

>[!NOTE]
> OEM から追加の管理アプリが提供されている場合があります。 他の管理アプリのユーザー名またはパスワードを更新しても、BMC のユーザー名またはパスワードには影響しません。 

::: moniker range="<azs-1910"
1. OEM の指示に従って、Azure Stack Hub の物理サーバー上で BMC を更新します。 環境内の各 BMC のユーザー名とパスワードは同じである必要があります。 BMC のユーザー名は、16 文字を超えることはできません。
::: moniker-end

::: moniker range=">=azs-1910"
1. OEM の指示に従って Azure Stack Hub 物理サーバーの BMC 資格情報を最初に更新する必要がなくなりました。 環境内の各 BMC のユーザー名とパスワードは同じである必要があり、16 文字以内で指定する必要があります。 
::: moniker-end

2. Azure Stack Hub セッションで特権エンドポイントを開きます。 手順については、「[Azure Stack Hub で特権エンドポイントを使用する](azure-stack-privileged-endpoint.md)」を参照してください。 

3. 特権エンドポイント セッションを開いたら、次の PowerShell スクリプトのいずれかを実行します。これは、Invoke-Command を使用して、Set-BmcCredential を実行します。 省略可能な -BypassBMCUpdate パラメーターを Set-BMCCredential と共に使用すると、BMC の資格情報は更新されません。 Azure Stack Hub の内部データストアだけが更新されます。特権エンドポイントのセッション変数をパラメーターとして渡します。

    ユーザー名とパスワードの入力を求める PowerShell スクリプトの例を次に示します。 

    ```powershell
    # Interactive Version
    $PEPIp = "<Privileged Endpoint IP or Name>" # You can also use the machine name instead of IP here.
    $PEPCreds = Get-Credential "<Domain>\CloudAdmin" -Message "PEP Credentials"
    $NewBmcPwd = Read-Host -Prompt "Enter New BMC password" -AsSecureString
    $NewBmcUser = Read-Host -Prompt "Enter New BMC user name"

    $PEPSession = New-PSSession -ComputerName $PEPIp -Credential $PEPCreds -ConfigurationName "PrivilegedEndpoint"

    Invoke-Command -Session $PEPSession -ScriptBlock {
        # Parameter BmcPassword is mandatory, while the BmcUser parameter is optional.
        Set-BmcCredential -BmcPassword $using:NewBmcPwd -BmcUser $using:NewBmcUser
    }
    Remove-PSSession -Session $PEPSession
    ```

    ユーザー名とパスワードを変数にエンコードすることもできますが、これは安全性が低くなる可能性があります。

    ```powershell
    # Static Version
    $PEPIp = "<Privileged Endpoint IP or Name>" # You can also use the machine name instead of IP here.
    $PEPUser = "<Privileged Endpoint user for example Domain\CloudAdmin>"
    $PEPPwd = ConvertTo-SecureString "<Privileged Endpoint Password>" -AsPlainText -Force
    $PEPCreds = New-Object System.Management.Automation.PSCredential ($PEPUser, $PEPPwd)
    $NewBmcPwd = ConvertTo-SecureString "<New BMC Password>" -AsPlainText -Force
    $NewBmcUser = "<New BMC User name>"

    $PEPSession = New-PSSession -ComputerName $PEPIp -Credential $PEPCreds -ConfigurationName "PrivilegedEndpoint"

    Invoke-Command -Session $PEPSession -ScriptBlock {
        # Parameter BmcPassword is mandatory, while the BmcUser parameter is optional.
        Set-BmcCredential -BmcPassword $using:NewBmcPwd -BmcUser $using:NewBmcUser
    }
    Remove-PSSession -Session $PEPSession
    ```

## <a name="reference-start-secretrotation-cmdlet"></a>リファレンス: Start-SecretRotation コマンドレット

[Start-SecretRotation コマンドレット](../reference/pep-2002/start-secretrotation.md)は、Azure Stack Hub システムのインフラストラクチャ シークレットのローテーションを行います。 このコマンドレットは、`-Session` パラメーターで PEP セッションを渡す `Invoke-Command` スクリプト ブロックを使用することによって、Azure Stack Hub の特権エンドポイントに対してのみ実行できます。 既定では、すべての外部ネットワーク インフラストラクチャ エンドポイントの証明書のみのローテーションを行います。

| パラメーター | Type | 必須 | [位置] | Default | 説明 |
|--|--|--|--|--|--|
| `PfxFilesPath` | String  | False  | named  | なし  | すべての外部ネットワーク エンドポイント証明書を含む **\Certificates** ディレクトリへのファイル共有パスです。 外部シークレットのローテーションを行う場合にのみ必要です。 最後のディレクトリは **\Certificates** にする必要があります。 |
| `CertificatePassword` | SecureString | False  | named  | なし  | -PfXFilesPath で提供されているすべての証明書のパスワード。 外部のシークレットのローテーションを行うときに PfxFilesPath を指定する場合は、必須の値です。 |
| `Internal` | String | False | named | なし | Azure Stack Hub オペレーターが内部インフラストラクチャ シークレットのローテーションを行うときは、常に Internal フラグを使用する必要があります。 |
| `PathAccessCredential` | PSCredential | False  | named  | なし  | すべての外部ネットワーク エンドポイント証明書を含む **\Certificates** ディレクトリへのファイル共有の PowerShell 資格情報。 外部シークレットのローテーションを行う場合にのみ必要です。  |
| `ReRun` | SwitchParameter | False  | named  | なし  | 試行が失敗した後でシークレットのローテーションが再試行されるときに、常に使用する必要があります。 |

### <a name="syntax"></a>構文

#### <a name="for-external-secret-rotation"></a>外部シークレットのローテーションの場合

```powershell
Start-SecretRotation [-PfxFilesPath <string>] [-PathAccessCredential <PSCredential>] [-CertificatePassword <SecureString>]  
```

#### <a name="for-internal-secret-rotation"></a>内部シークレットのローテーションの場合

```powershell
Start-SecretRotation [-Internal]  
```

#### <a name="for-external-secret-rotation-rerun"></a>外部シークレットのローテーションの再実行の場合

```powershell
Start-SecretRotation [-ReRun]
```

#### <a name="for-internal-secret-rotation-rerun"></a>内部シークレットのローテーションの再実行の場合

```powershell
Start-SecretRotation [-ReRun] [-Internal]
```

### <a name="examples"></a>例

#### <a name="rotate-only-internal-infrastructure-secrets"></a>内部インフラストラクチャ シークレットのみのローテーション

このコマンドは、Azure Stack Hub [環境の特権エンドポイント](azure-stack-privileged-endpoint.md)を使用して実行する必要があります。

```powershell
PS C:\> Start-SecretRotation -Internal
```

このコマンドを使用すると、Azure Stack Hub 内部ネットワークに公開されているすべてのインフラストラクチャ シークレットのローテーションが行われます。

#### <a name="rotate-only-external-infrastructure-secrets"></a>外部インフラストラクチャ シークレットのみのローテーション  

```powershell
# Create a PEP Session
winrm s winrm/config/client '@{TrustedHosts= "<IP_address_of_ERCS>"}'
$PEPCreds = Get-Credential
$PEPSession = New-PSSession -ComputerName <IP_address_of_ERCS> -Credential $PEPCreds -ConfigurationName "PrivilegedEndpoint"

# Create Credentials for the fileshare
$CertPassword = ConvertTo-SecureString "<CertPasswordHere>" -AsPlainText -Force
$CertShareCreds = Get-Credential
$CertSharePath = "<NetworkPathOfCertShare>"
# Run Secret Rotation
Invoke-Command -Session $PEPSession -ScriptBlock {  
    Start-SecretRotation -PfxFilesPath $using:CertSharePath -PathAccessCredential $using:CertShareCreds -CertificatePassword $using:CertPassword
}
Remove-PSSession -Session $PEPSession
```

このコマンドは、Azure Stack Hub の外部ネットワーク インフラストラクチャ エンドポイントで使用する TLS 証明書のローテーションを行います。

::: moniker range="<azs-1811"
#### <a name="rotate-internal-and-external-infrastructure-secrets-pre-1811-only"></a>内部と外部のインフラストラクチャ シークレットのローテーション (**1811 より前** のみ)

> [!IMPORTANT]
> ローテーションは内部証明書と外部証明書に分けられたので、このコマンドは **1811 より前** の Azure Stack Hub にのみ適用されます。
>
> **"*1811 以降*" 、内部証明書と外部証明書の両方をローテーションすることはできなくなりました。**

```powershell
# Create a PEP Session
winrm s winrm/config/client '@{TrustedHosts= "<IP_address_of_ERCS>"}'
$PEPCreds = Get-Credential
$PEPSession = New-PSSession -ComputerName <IP_address_of_ERCS> -Credential $PEPCreds -ConfigurationName "PrivilegedEndpoint"

# Create Credentials for the fileshare
$CertPassword = ConvertTo-SecureString "<CertPasswordHere>" -AsPlainText -Force
$CertShareCreds = Get-Credential
$CertSharePath = "<NetworkPathOfCertShare>"
# Run Secret Rotation
Invoke-Command -Session $PEPSession -ScriptBlock {
    Start-SecretRotation -PfxFilesPath $using:CertSharePath -PathAccessCredential $using:CertShareCreds -CertificatePassword $using:CertPassword
}
Remove-PSSession -Session $PEPSession
```

このコマンドは、Azure Stack Hub 内部ネットワークに公開されているインフラストラクチャ シークレットと、Azure Stack Hub の外部ネットワーク インフラストラクチャ エンドポイントに使用される TLS 証明書をローテーションします。 Start-SecretRotation は、スタックで生成されたすべてのシークレットのローテーションを行い、証明書が指定されているため、外部エンドポイント証明書のローテーションも行われます。  
::: moniker-end

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub のセキュリティについて詳しく学習する](azure-stack-security-foundations.md)
