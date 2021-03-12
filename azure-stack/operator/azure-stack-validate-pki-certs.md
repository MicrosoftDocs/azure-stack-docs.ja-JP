---
title: Azure Stack Hub PKI 証明書の検証
titleSuffix: Azure Stack Hub
description: Azure Stack Hub 適合性チェック ツールを使用して、Azure Stack Hub 統合システムの PKI 証明書を検証する方法について説明します。
services: azure-stack
documentationcenter: ''
author: BryanLa
ms.topic: how-to
ms.date: 10/19/2020
ms.author: bryanla
ms.reviewer: ppacent
ms.lastreviewed: 10/19/2020
ms.openlocfilehash: b03bd67f774c8034ad68490393a29fc99c118b54
ms.sourcegitcommit: ccc4ee05d71496653b6e27de1bb12e4347e20ba4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102231151"
---
# <a name="validate-azure-stack-hub-pki-certificates"></a>Azure Stack Hub PKI 証明書の検証

この記事で説明されている Azure Stack Hub 適合性チェッカー ツールは、[PowerShell ギャラリーから](https://aka.ms/AzsReadinessChecker)入手可能です。 デプロイ前にこのツールを使用して、[生成されたパブリック キー インフラストラクチャ (PKI) 証明書](azure-stack-get-pki-certs.md)が適切であるかどうかを検証します。 十分な時間をかけて証明書を検証し、必要であれば証明書をテストして再発行します。

適合性チェッカー ツールでは、次の証明書の検証を実行します。

- **PFX の解析**  
    有効な PFX ファイル、正しいパスワード、およびパブリック情報がパスワードによって保護されているかどうかを確認します。
- **有効期限**  
    7 日間の最小限の有効期間があるかどうかを確認します。
- **署名アルゴリズム**  
    署名アルゴリズムが SHA1 ではないことを確認します。
- **秘密キー**  
    秘密キーが存在し、ローカル コンピューター属性でエクスポートされることを確認します。 
- **証明書チェーン**  
    自己署名証明書の場合も含めて証明書チェーンが損なわれていないことを確認します。
- **DNS 名**  
    SAN に各エンドポイントの関連する DNS 名が含まれていること、またはサポートするワイルドカードが存在するかどうかを確認します。
- **キー使用法**  
    キー使用法にデジタル署名とキーの暗号化が含まれていること、および拡張キー使用法にサーバー認証とクライアント認証が含まれていることを確認します。
- **キーサイズ**  
    キー サイズが 2048 以上であることを確認します。
- **チェーンの順序**  
    他の証明書の順序を確認して、順序が正しいことを検証します。
- **他の証明書**  
    関連するリーフ証明書とそのチェーン以外の他の証明書が PFX にパッケージ化されていないことを確認します。

> [!IMPORTANT]  
> PKI 証明書が PFX ファイルです。パスワードは機密情報として扱われる必要があります。

## <a name="prerequisites"></a>前提条件

Azure Stack Hub のデプロイに対して PKI 証明書を検証する前に、システムは次の前提条件を満たしている必要があります。

- Microsoft Azure Stack Hub 適合性チェッカー。
- [準備手順](azure-stack-prepare-pki-certs.md)に従ってエクスポートされた SSL 証明書。
- DeploymentData.json。
- Windows 10 または Windows Server 2016。

## <a name="perform-core-services-certificate-validation"></a>コア サービス証明書の検証を実行する

次の手順を使って、デプロイとシークレット ローテーションに使用する Azure Stack Hub PKI 証明書を検証します。

1. 次のコマンドレットを実行して、PowerShell プロンプト (5.1 以降) から **AzsReadinessChecker** をインストールします。

    ```powershell  
        Install-Module Microsoft.AzureStack.ReadinessChecker -Force -AllowPrerelease
    ```

2. 証明書ディレクトリ構造を作成します。 次の例では、`<C:\Certificates\Deployment>` を、選択した新規ディレクトリ パスに変更できます。
    ```powershell  
    New-Item C:\Certificates\Deployment -ItemType Directory
    
    $directories = 'ACSBlob', 'ACSQueue', 'ACSTable', 'Admin Extension Host', 'Admin Portal', 'ARM Admin', 'ARM Public', 'KeyVault', 'KeyVaultInternal', 'Public Extension Host', 'Public Portal'
    
    $destination = 'C:\Certificates\Deployment'
    
    $directories | % { New-Item -Path (Join-Path $destination $PSITEM) -ItemType Directory -Force}
    ```
    
    > [!Note]  
    > AD FS を ID システムとして使用する場合は、AD FS と Graph が必要です。 次に例を示します。
    >
    > ```powershell  
    > $directories = 'ACSBlob', 'ACSQueue', 'ACSTable', 'ADFS', 'Admin Extension Host', 'Admin Portal', 'ARM Admin', 'ARM Public', 'Graph', 'KeyVault', 'KeyVaultInternal', 'Public Extension Host', 'Public Portal'
    > ```
    
     - 前の手順で作成された適切なディレクトリに証明書を配置します。 次に例を示します。  
        - `C:\Certificates\Deployment\ACSBlob\CustomerCertificate.pfx`
        - `C:\Certificates\Deployment\Admin Portal\CustomerCertificate.pfx`
        - `C:\Certificates\Deployment\ARM Admin\CustomerCertificate.pfx`

3. PowerShell ウィンドウで、Azure Stack Hub 環境に合わせて `RegionName`、`FQDN`、および `IdentitySystem` の値を変更し、次のコマンドレットを実行します。

    ```powershell  
    $pfxPassword = Read-Host -Prompt "Enter PFX Password" -AsSecureString 
    Invoke-AzsHubDeploymentCertificateValidation -CertificatePath C:\Certificates\Deployment -pfxPassword $pfxPassword -RegionName east -FQDN azurestack.contoso.com -IdentitySystem AAD  
    ```

4. 出力を確認し、すべての証明書がすべてのテストに合格していることを確認します。 次に例を示します。

    ```powershell
    Invoke-AzsHubDeploymentCertificateValidation v1.2005.1286.272 started.
    Testing: KeyVaultInternal\KeyVaultInternal.pfx
    Thumbprint: E86699****************************4617D6
        PFX Encryption: OK
        Expiry Date: OK
        Signature Algorithm: OK
        DNS Names: OK
        Key Usage: OK
        Key Length: OK
        Parse PFX: OK
        Private Key: OK
        Cert Chain: OK
        Chain Order: OK
        Other Certificates: OK
    Testing: ARM Public\ARMPublic.pfx
    Thumbprint: 8DC4D9****************************69DBAA
        PFX Encryption: OK
        Expiry Date: OK
        Signature Algorithm: OK
        DNS Names: OK
        Key Usage: OK
        Key Length: OK
        Parse PFX: OK
        Private Key: OK
        Cert Chain: OK
        Chain Order: OK
        Other Certificates: OK
    Testing: Admin Portal\AdminPortal.pfx
    Thumbprint: 6F9055****************************4AC0EA
        PFX Encryption: OK
        Expiry Date: OK
        Signature Algorithm: OK
        DNS Names: OK
        Key Usage: OK
        Key Length: OK
        Parse PFX: OK
        Private Key: OK
        Cert Chain: OK
        Chain Order: OK
        Other Certificates: OK
    Testing: Public Portal\PublicPortal.pfx


    Log location (contains PII): C:\Users\[*redacted*]\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessChecker.log
    Report location (contains PII): C:\Users\[*redacted*]\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessCheckerReport.json
    Invoke-AzsHubDeploymentCertificateValidation Completed

    ```

    他の Azure Stack Hub サービスの証明書を検証するには、```-CertificateType``` の値を変更します。 次に例を示します。

    ```powershell  
    # App Services
    Invoke-AzsHubAppServicesCertificateValidation -CertificatePath C:\Certificates\AppServices -pfxPassword $pfxPassword -RegionName east -FQDN azurestack.contoso.com

    # DBAdapter
    Invoke-AzsHubDBAdapterCertificateValidation -CertificatePath C:\Certificates\DBAdapter -pfxPassword $pfxPassword -RegionName east -FQDN azurestack.contoso.com

    # EventHubs
    Invoke-AzsHubEventHubsCertificateValidation -CertificatePath C:\Certificates\EventHubs -pfxPassword $pfxPassword -RegionName east -FQDN azurestack.contoso.com

    # IoTHub
    Invoke-AzsHubIoTHubCertificateValidation -CertificatePath C:\Certificates\IoTHub -pfxPassword $pfxPassword -RegionName east -FQDN azurestack.contoso.com
    ```
    各フォルダーには、証明書の種類に対応する単一の PFX ファイルが含まれている必要があります。 ある証明書の種類で複数の証明書が必要な場合は、個々の証明書ごとに入れ子になったフォルダーが必要で、名前で区別されます。 次のコードは、すべての証明書の種類に関するフォルダー/証明書構造の例と、```-CertificateType``` および ```-CertificatePath```の適切な値を示しています。
    
    ```powershell  
    C:\>tree c:\SecretStore /A /F
        Folder PATH listing
        Volume serial number is 85AE-DF2E
        C:\SECRETSTORE
        \---AzureStack
            +---CertificateRequests
            \---Certificates
                +---AppServices         # Invoke-AzsCertificateValidation `
                |   +---API             #     -CertificateType AppServices `
                |   |       api.pfx     #     -CertificatePath C:\Certificates\AppServices
                |   |
                |   +---DefaultDomain
                |   |       wappsvc.pfx
                |   |
                |   +---Identity
                |   |       sso.pfx
                |   |
                |   \---Publishing
                |           ftp.pfx
                |
                +---DBAdapter           # Invoke-AzsCertificateValidation `
                |       dbadapter.pfx   #   -CertificateType DBAdapter `
                |                       #   -CertificatePath C:\Certificates\DBAdapter
                |
                +---Deployment          # Invoke-AzsCertificateValidation `
                |   +---ACSBlob         #   -CertificateType Deployment `
                |   |       acsblob.pfx #   -CertificatePath C:\Certificates\Deployment
                |   |
                |   +---ACSQueue
                |   |       acsqueue.pfx
               ./. ./. ./. ./. ./. ./. ./.    <- Deployment certificate tree trimmed.
                |   \---Public Portal
                |           portal.pfx
                |
                +---EventHubs           # Invoke-AzsCertificateValidation `
                |       eventhubs.pfx   #   -CertificateType EventHubs `
                |                       #   -CertificatePath C:\Certificates\EventHubs
                |
                \---IoTHub              # Invoke-AzsCertificateValidation `
                        iothub.pfx      #   -CertificateType IoTHub `
                                        #   -CertificatePath C:\Certificates\IoTHub
    ```

### <a name="known-issues"></a>既知の問題

**現象**:テストがスキップされる

**原因**:AzsReadinessChecker は、依存関係が満たされていない場合、特定のテストをスキップします。

 - 証明書チェーンに問題がある場合、他の証明書はスキップされます。

    ```powershell  
    Testing: ACSBlob\singlewildcard.pfx
        Read PFX: OK
        Signature Algorithm: OK
        Private Key: OK
        Cert Chain: OK
        DNS Names: Fail
        Key Usage: OK
        Key Size: OK
        Chain Order: OK
        Other Certificates: Skipped
    Details:
    The certificate records '*.east.azurestack.contoso.com' do not contain a record that is valid for '*.blob.east.azurestack.contoso.com'. Please refer to the documentation for how to create the required certificate file.
    The Other Certificates check was skipped because Cert Chain and/or DNS Names failed. Follow the guidance to remediate those issues and recheck. 

    Log location (contains PII): C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessChecker.log
    Report location (contains PII): C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessCheckerReport.json
    Invoke-AzsCertificateValidation Completed
    ```

**解決方法**:ツール ガイダンスの詳細セクションにある各証明書テスト設定に従います。

**現象**:HTTP CDP が x509 拡張機能に書き込まれているにもかかわらず、HTTP CRL チェックが失敗する。

**原因**:現時点では、AzsReadinessChecker によって HTTP CDP を確認できない言語がいくつかあります。

**解決方法**:OS 言語を EN-US に設定して検証を実行します。

## <a name="certificates"></a>証明書

| ディレクトリ | Certificate |
| ---    | ----        |
| ACSBlob | `wildcard_blob_<region>_<externalFQDN>` |
| ACSQueue  |  `wildcard_queue_<region>_<externalFQDN>` |
| ACSTable  |  `wildcard_table_<region>_<externalFQDN>` |
| 管理者拡張機能ホスト  |  `wildcard_adminhosting_<region>_<externalFQDN>` |
| 管理ポータル  |  `adminportal_<region>_<externalFQDN>` |
| ARM 管理  |  `adminmanagement_<region>_<externalFQDN>` |
| ARM パブリック  |  `management_<region>_<externalFQDN>` |
| KeyVault  |  `wildcard_vault_<region>_<externalFQDN>` |
| KeyVaultInternal  |  `wildcard_adminvault_<region>_<externalFQDN>` |
| パブリック拡張機能ホスト  |  `wildcard_hosting_<region>_<externalFQDN>` |
| パブリック ポータル  |  `portal_<region>_<externalFQDN>` |

## <a name="using-validated-certificates"></a>検証済み証明書の使用

証明書が AzsReadinessChecker によって検証されたら、Azure Stack Hub デプロイまたは Azure Stack Hub シークレット ローテーションで使用できます。

 - デプロイの場合、[Azure Stack Hub PKI の要件ドキュメント](azure-stack-pki-certs.md)で指定されているように、デプロイ エンジニアが証明書を配置ホストにコピーできるように、セキュリティを保護して証明書をデプロイ エンジニアに送付します。
 - シークレット ローテーションの場合、[Azure Stack Hub シークレット ローテーションのドキュメント](azure-stack-rotate-secrets.md)の説明に従い、この証明書を使用して、お使いの Azure Stack Hub 環境のパブリック インフラストラクチャ エンドポイントの古い証明書を更新できます。
 - PaaS サービスの場合、「[Azure Stack Hub でのサービスの提供の概要](service-plan-offer-subscription-overview.md)」の説明に従い、証明書を使用して、Azure Stack Hub に SQL、MySQL、App Services リソース プロバイダーをインストールできます。

## <a name="next-steps"></a>次のステップ

[データセンターの ID の統合](azure-stack-integrate-identity.md)
