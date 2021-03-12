---
title: デプロイまたはローテーションのために Azure Stack Hub PKI 証明書を準備する
titleSuffix: Azure Stack Hub
description: Azure Stack Hub のデプロイ、またはシークレットのローテーションのために PKI 証明書を準備する方法について説明します。
author: BryanLa
ms.topic: how-to
ms.date: 10/19/2020
ms.author: bryanla
ms.reviewer: ppacent
ms.lastreviewed: 10/19/2020
ms.openlocfilehash: 863a10ced2bf4c9ee4efef29472aab595ece4aa4
ms.sourcegitcommit: ccc4ee05d71496653b6e27de1bb12e4347e20ba4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102231253"
---
# <a name="prepare-azure-stack-hub-pki-certificates-for-deployment-or-rotation"></a>デプロイまたはローテーションのために Azure Stack Hub PKI 証明書を準備する

> [!NOTE]
> この記事では、外部インフラストラクチャとサービスのエンドポイントをセキュリティで保護するために使用される外部証明書の準備についてのみ説明します。 内部証明書は、[証明書のローテーション プロセス](azure-stack-rotate-secrets.md)中に個別に管理されます。

[証明機関 (CA) から取得した](azure-stack-get-pki-certs.md)証明書ファイルは、Azure Stack Hub の証明書要件に一致するプロパティを使用してインポートおよびエクスポートする必要があります。

この記事では、Azure Stack Hub のデプロイまたはシークレットのローテーションを準備するために、外部証明書をインポート、パッケージ化、検証する方法について説明します。 

## <a name="prerequisites"></a>前提条件

Azure Stack Hub のデプロイに対して PKI 証明書をパッケージ化する前に、システムが次の前提条件を満たしている必要があります。

- 証明機関から返された証明書は、.cer 形式 (その他の構成可能な形式は cert、.sst、.pfx など) で1 つのディレクトリに格納されます。
- Windows 10、または Windows Server 2016 以降
- 証明書署名要求を生成したのと同じシステムを使用します (PFX に事前パッケージ化されている証明書を対象としている場合を除く)。

適切なセクション「[証明書を準備する (Azure Stack 適合性チェッカー)](#prepare-certificates-azure-stack-readiness-checker)」または「[証明書を準備する (手動手順)](#prepare-certificates-manual-steps)」に進みます。

## <a name="prepare-certificates-azure-stack-readiness-checker"></a>証明書を準備する (Azure Stack 適合性チェッカー)

Azure Stack 適合性チェッカーの PowerShell コマンドレットを使用して証明書をパッケージ化するには、次の手順を使用します。

1. 次のコマンドレットを実行して、PowerShell プロンプト (5.1 以上) から Azure Stack 適合性チェッカー モジュールをインストールします。

    ```powershell  
        Install-Module Microsoft.AzureStack.ReadinessChecker -Force -AllowPrerelease
    ```
2. 証明書ファイルの **パス** を指定します。 次に例を示します。

    ```powershell  
        $Path = "$env:USERPROFILE\Documents\AzureStack"
    ```

3. **pfxPassword** を宣言します。 次に例を示します。

    ```powershell  
        $pfxPassword = Read-Host -AsSecureString -Prompt "PFX Password"
    ```
4. 結果の PFX のエクスポート先となる **ExportPath** を宣言します。 次に例を示します。

    ```powershell  
        $ExportPath = "$env:USERPROFILE\Documents\AzureStack"
    ```

5. 証明書を Azure Stack Hub 証明書に変換します。 次に例を示します。

    ```powershell  
        ConvertTo-AzsPFX -Path $Path -pfxPassword $pfxPassword -ExportPath $ExportPath
    ```
8.  出力を確認します。

    ```powershell  
    ConvertTo-AzsPFX v1.2005.1286.272 started.

    Stage 1: Scanning Certificates
        Path: C:\Users\[*redacted*]\Documents\AzureStack Filter: CER Certificate count: 11
        adminmanagement_east_azurestack_contoso_com_CertRequest_20200710235648.cer
        adminportal_east_azurestack_contoso_com_CertRequest_20200710235645.cer
        management_east_azurestack_contoso_com_CertRequest_20200710235644.cer
        portal_east_azurestack_contoso_com_CertRequest_20200710235646.cer
        wildcard_adminhosting_east_azurestack_contoso_com_CertRequest_20200710235649.cer
        wildcard_adminvault_east_azurestack_contoso_com_CertRequest_20200710235642.cer
        wildcard_blob_east_azurestack_contoso_com_CertRequest_20200710235653.cer
        wildcard_hosting_east_azurestack_contoso_com_CertRequest_20200710235652.cer
        wildcard_queue_east_azurestack_contoso_com_CertRequest_20200710235654.cer
        wildcard_table_east_azurestack_contoso_com_CertRequest_20200710235650.cer
        wildcard_vault_east_azurestack_contoso_com_CertRequest_20200710235647.cer

    Detected ExternalFQDN: east.azurestack.contoso.com

    Stage 2: Exporting Certificates
        east.azurestack.contoso.com\Deployment\ARM Admin\ARMAdmin.pfx
        east.azurestack.contoso.com\Deployment\Admin Portal\AdminPortal.pfx
        east.azurestack.contoso.com\Deployment\ARM Public\ARMPublic.pfx
        east.azurestack.contoso.com\Deployment\Public Portal\PublicPortal.pfx
        east.azurestack.contoso.com\Deployment\Admin Extension Host\AdminExtensionHost.pfx
        east.azurestack.contoso.com\Deployment\KeyVaultInternal\KeyVaultInternal.pfx
        east.azurestack.contoso.com\Deployment\ACSBlob\ACSBlob.pfx
        east.azurestack.contoso.com\Deployment\Public Extension Host\PublicExtensionHost.pfx
        east.azurestack.contoso.com\Deployment\ACSQueue\ACSQueue.pfx
        east.azurestack.contoso.com\Deployment\ACSTable\ACSTable.pfx
        east.azurestack.contoso.com\Deployment\KeyVault\KeyVault.pfx

    Stage 3: Validating Certificates.

    Validating east.azurestack.contoso.com-Deployment-AAD certificates in C:\Users\[*redacted*]\Documents\AzureStack\east.azurestack.contoso.com\Deployment 

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
        ...
    Log location (contains PII): C:\Users\[*redacted*]\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessChecker.log
    ConvertTo-AzsPFX Completed
    ```
    > [!NOTE]
    > 検証を無効にする、または、さまざまな証明書形式をフィルター処理する場合などのその他の使用方法については、Get-help ConvertTo-AzsPFX -Full を使用します。

    検証が成功すると、追加の手順を行わなくても、デプロイまたはローテーション用の証明書を提示できます。

## <a name="prepare-certificates-manual-steps"></a>証明書を準備する (手動の手順)

手動の手順を使用して新しい Azure Stack Hub PKI 証明書の証明書をパッケージ化するには、次の手順を使用します。

### <a name="import-the-certificate"></a>証明書のインポート

1. [任意の CA から取得した](azure-stack-get-pki-certs.md)オリジナル バージョンの証明書を、デプロイ ホスト上のディレクトリにコピーします。 
   > [!WARNING]
   > CA によって直接提供されたファイルから何らかの方法で既にインポート、エクスポート、または変更されているファイルをコピーしないでください。

1. 証明書を右クリックして、CA からどのように証明書が配信されたかによって、 **[証明書のインストール]** または **[PFX のインストール]** を選択します。

1. **証明書のインポート ウィザード** で、インポートの場所として **[ローカル コンピューター]** を選択します。 **[次へ]** を選択します。 次の画面で、もう一度 [次へ] を選択します。

    ![ローカル コンピューターの証明書のインポート場所](./media/prepare-pki-certs/1.png)

1. **[Place all certificate in the following store]\(すべての証明書を次のストアに配置する\)** を選択し、場所として **[エンタープライズの信頼]** を選択します。 **[OK]** を選択して証明書ストアの選択ダイアログ ボックスを閉じ、 **[次へ]** をクリックします。

   ![証明書のインポート用に証明書ストアを構成する](./media/prepare-pki-certs/3.png)

   a. PFX をインポートする場合は、追加のダイアログが表示されます。 **[秘密キーの保護]** ページで、証明書ファイルのパスワードを入力してから、 **[このキーをエクスポート可能にする]** オプションを有効にします。これにより、後でキーをバックアップしたり転送したりすることができます。 **[次へ]** を選択します。

   ![キーをエクスポート可能としてマークする](./media/prepare-pki-certs/2.png)

1. **[完了]** を選択してインポートを完了します。

> [!NOTE]
> Azure Stack Hub 用に証明書をインポートすると、証明書の秘密キーがクラスター記憶域に PKCS 12 ファイル (PFX) として保存されます。

### <a name="export-the-certificate"></a>証明書をエクスポートする

証明書管理者 MMC コンソールを開き、ローカル コンピューターの証明書ストアに接続します。

1. Microsoft 管理コンソールを開きます。 Windows 10 でコンソールを開くには、 **[スタート] メニュー** を右クリックし、 **[実行]** を選択してから、「**mmc**」と入力して Enter キーを押します。

2. **[ファイル]**  >  **[スナップインの追加と削除]** の順に選択してから、 **[証明書]** を選択して **[追加]** を選択します。

    ![Microsoft 管理コンソールで証明書スナップインを追加する](./media/prepare-pki-certs/mmc-2.png)

3. **[コンピューター アカウント]** を選択し、 **[次へ]** を選択します。 **[ローカル コンピューター]** を選択し、 **[完了]** を選択します。 **[OK]** を選択して [スナップインの追加と削除] ページを閉じます。

    ![Microsoft 管理コンソールで証明書スナップインを追加するためのアカウントを選択する](./media/prepare-pki-certs/mmc-3.png)

4. **[証明書]**  >  **[エンタープライズの信頼]**  >  **[証明書の場所]** の順に進みます。 右側に証明書が表示されることを確認します。

5. 証明書管理者コンソールのタスク バーから、 **[アクション]**  >  **[すべてのタスク]**  >  **[エクスポート]** の順に選択します。 **[次へ]** を選択します。

   > [!NOTE]
   > Azure Stack Hub の証明書の数によっては、このプロセスを複数回完了する必要があります。

6. **[はい、秘密キーをエクスポートします]** を選択し、 **[次へ]** を選択します。

7. [エクスポート ファイルの形式] セクションで、次の操作を行います。
    
   - **[Include all certificates in the certificate if possible]\(証明書にすべての証明書を含める (可能な場合)\)** を選択します。  
   - **[Export all Extended Properties]\(すべての拡張プロパティをエクスポートする\)** を選択します。  
   - **[証明書のプライバシーを有効にする ]** を選択します。  
   - **[次へ]** を選択します。  
    
     ![いくつかのオプションが選択されている [Certificate export wizard]\(証明書のエクスポート ウィザード\)](./media/prepare-pki-certs/azure-stack-save-cert.png)

8. **[パスワード]** を選択し、証明書のパスワードを指定します。 次のパスワードの複雑さの要件を満たしているパスワードを作成します。

    * 8 文字の最小長。
    * 大文字、小文字、0 から 9 までの数字、特殊文字、および大文字でも小文字でもない英字のうちの少なくとも 3 種類。

    このパスワードを書き留めておいてください。 デプロイ パラメーターとして使用します。

9. **[次へ]** を選択します。

10. エクスポートする PFX ファイルのファイル名と場所を選択します。 **[次へ]** を選択します。

11. **[完了]** を選択します。

## <a name="next-steps"></a>次のステップ

[PKI 証明書の検証](azure-stack-validate-pki-certs.md)
