---
title: Azure Stack 統合システム デプロイのための Azure Stack 公開キー インフラストラクチャ証明書を生成する | Microsoft Docs
description: Azure Stack 統合システムの Azure Stack PKI 証明書のデプロイ プロセスについて説明します。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/16/2019
ms.author: mabrigg
ms.reviewer: ppacent
ms.lastreviewed: 01/25/2019
ms.openlocfilehash: 1c342b1edb86629fff95dc04735fd5b6d98fc70a
ms.sourcegitcommit: 889fd09e0ab51ad0e43552a800bbe39dc9429579
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/16/2019
ms.locfileid: "65782269"
---
# <a name="azure-stack-certificates-signing-request-generation"></a>Azure Stack 証明書署名要求の生成

Azure Stack 適合性チェッカー ツールを使用して、Azure Stack デプロイに適した証明書署名要求 (CSR) を作成できます。 証明書は、デプロイ前のテストに十分な時間を確保した上で、要求、生成、検証する必要があります。 このツールは [PowerShell ギャラリーから](https://aka.ms/AzsReadinessChecker)取得できます。

Azure Stack 適合性チェッカー ツール (AzsReadinessChecker) を使用すると、次の証明書を要求できます。

- [Azure Stack デプロイのための PKI 証明書の生成](azure-stack-get-pki-certs.md)に関する説明に従った**標準の証明書要求**。
- **サービスとしてのプラットフォーム**:「Azure Stack 公開キー インフラストラクチャ証明書の要件」の「[オプションの PaaS 証明書](azure-stack-pki-certs.md#optional-paas-certificates)」で指定されているように、証明書に対するサービスとしてのプラットフォーム (PaaS) 名を要求できます。

## <a name="prerequisites"></a>前提条件

Azure Stack デプロイのための PKI 証明書に対する CSR を生成する前に、システムは次の前提条件を満たしている必要があります。

- Microsoft Azure Stack 適合性チェッカー
- 証明書の属性:
  - リージョン名
  - 外部完全修飾ドメイン名 (FQDN)
  - サブジェクト
- Windows 10 または Windows Server 2016

  > [!NOTE]  
  > 証明機関から証明書が送り返されたら、「[Azure Stack PKI 証明書の準備](azure-stack-prepare-pki-certs.md)」の手順を同じシステムで完了する必要があります。

## <a name="generate-certificate-signing-requests"></a>証明書の署名要求を生成する

次の手順を使って、Azure Stack PKI 証明書を準備し、検証します。

1. 次のコマンドレットを実行して、PowerShell プロンプト (5.1 以上) から AzsReadinessChecker をインストールします。

    ```powershell  
        Install-Module Microsoft.AzureStack.ReadinessChecker
    ```

2. 順序指定されたディクショナリとして**サブジェクト**を宣言します。 例: 

    ```powershell  
    $subjectHash = [ordered]@{"OU"="AzureStack";"O"="Microsoft";"L"="Redmond";"ST"="Washington";"C"="US"}
    ```

    > [!note]  
    > 共通名 (CN) が指定されている場合、証明書要求の最初の DNS 名によって上書きされます。

3. 既に存在する出力ディレクトリを宣言します。 例: 

    ```powershell  
    $outputDirectory = "$ENV:USERPROFILE\Documents\AzureStackCSR"
    ```

4. ID システムを宣言します

    Azure Active Directory

    ```powershell
    $IdentitySystem = "AAD"
    ```

    Active Directory フェデレーション サービス (AD FS)

    ```powershell
    $IdentitySystem = "ADFS"
    ```

5. Azure Stack デプロイのために**リージョン名**と**外部 FQDN** を宣言します。

    ```powershell
    $regionName = 'east'
    $externalFQDN = 'azurestack.contoso.com'
    ```

    > [!note]  
    > `<regionName>.<externalFQDN>` は、Azure Stack のすべての外部 DNS が作成されるベースを形成します。この例では、ポータルは `portal.east.azurestack.contoso.com` となります。  

6. DNS 名ごとに証明書署名要求を生成する場合は、次のように実行します。

    ```powershell  
    New-AzsCertificateSigningRequest -RegionName $regionName -FQDN $externalFQDN -subject $subjectHash -OutputRequestPath $OutputDirectory -IdentitySystem $IdentitySystem
    ```

    PaaS サービスを含めるには、スイッチ ```-IncludePaaS``` を指定します

7. または、開発/テスト環境の場合は、複数のサブジェクトの別名 (SAN) を含む 1 つの証明書要求を生成するには、**-RequestType SingleCSR** パラメーターと値を追加します (運用環境では、推奨**されません**)。

    ```powershell  
    New-AzsCertificateSigningRequest -RegionName $regionName -FQDN $externalFQDN -subject $subjectHash -RequestType SingleCSR -OutputRequestPath $OutputDirectory -IdentitySystem $IdentitySystem
    ```

    PaaS サービスを含めるには、スイッチ ```-IncludePaaS``` を指定します

8. 出力を確認します。

    ```powershell  
    New-AzsCertificateSigningRequest v1.1809.1005.1 started.

    CSR generating for following SAN(s): dns=*.east.azurestack.contoso.com&dns=*.blob.east.azurestack.contoso.com&dns=*.queue.east.azurestack.contoso.com&dns=*.table.east.azurestack.cont
    oso.com&dns=*.vault.east.azurestack.contoso.com&dns=*.adminvault.east.azurestack.contoso.com&dns=portal.east.azurestack.contoso.com&dns=adminportal.east.azurestack.contoso.com&dns=ma
    nagement.east.azurestack.contoso.com&dns=adminmanagement.east.azurestack.contoso.com*dn2=*.adminhosting.east.azurestack.contoso.com@dns=*.hosting.east.azurestack.contoso.com
    Present this CSR to your Certificate Authority for Certificate Generation: C:\Users\username\Documents\AzureStackCSR\wildcard_east_azurestack_contoso_com_CertRequest_20180405233530.req
    Certreq.exe output: CertReq: Request Created

    Log location (contains PII): C:\Users\username\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessChecker.log
    New-AzsCertificateSigningRequest Completed
    ```

9. 生成された **.REQ** ファイルを CA (内部またはパブリック) に送信します。  **New-AzsCertificateSigningRequest** の出力ディレクトリには、証明機関への送信に必要な CSR が含まれています。  また、このディレクトリには、証明書要求の生成中に使用される INF ファイルが含まれる子ディレクトリも参照用として含まれています。 CA が生成された要求を使用して、[Azure Stack PKI の要件](azure-stack-pki-certs.md)を満たす証明書を生成することを確認してください。

## <a name="next-steps"></a>次の手順

[Azure Stack PKI 証明書の準備](azure-stack-prepare-pki-certs.md)
