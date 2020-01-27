---
title: デプロイまたはローテーションのために Azure Stack Hub PKI 証明書を準備する | Microsoft Docs
titleSuffix: Azure Stack Hub
description: Azure Stack Hub 統合システムのデプロイ、または既存の Azure Stack Hub 環境でのシークレットのローテーションのために PKI 証明書を準備する方法について説明します。
services: azure-stack
documentationcenter: ''
author: justinha
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/16/2019
ms.author: justinha
ms.reviewer: ppacent
ms.lastreviewed: 09/16/2019
ms.openlocfilehash: 9031f81909a5bfe44882aabce47488ba6ca40edf
ms.sourcegitcommit: 1185b66f69f28e44481ce96a315ea285ed404b66
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75810352"
---
# <a name="prepare-azure-stack-hub-pki-certificates-for-deployment-or-rotation"></a>デプロイまたはローテーションのために Azure Stack Hub PKI 証明書を準備する

[任意の証明機関 (CA) から取得した](azure-stack-get-pki-certs.md)証明書ファイルは、Azure Stack Hub の証明書要件に一致するプロパティを使用してインポートおよびエクスポートする必要があります。

## <a name="prepare-certificates-for-deployment"></a>デプロイ用の証明書を準備する

次の手順を使用して、新しい Azure Stack Hub 環境のデプロイまたは既存の Azure Stack Hub 環境でのシークレットのローテーションに使用される Azure Stack Hub PKI 証明書を準備および検証します。

### <a name="import-the-certificate"></a>証明書のインポート

1. [任意の CA から取得した](azure-stack-get-pki-certs.md)オリジナル バージョンの証明書を、デプロイ ホスト上のディレクトリにコピーします。 
   > [!WARNING]
   > CA によって直接提供されたファイルから何らかの方法で既にインポート、エクスポート、または変更されているファイルをコピーしないでください。

1. 証明書を右クリックして、CA からどのように証明書が配信されたかによって、 **[証明書のインストール]** または **[PFX のインストール]** を選択します。

1. **証明書のインポート ウィザード**で、インポートの場所として **[ローカル コンピューター]** を選択します。 **[次へ]** を選択します。 次の画面で、もう一度 [次へ] を選択します。

    ![ローカル コンピューターの証明書のインポート場所](./media/prepare-pki-certs/1.png)

1. **[Place all certificate in the following store]\(すべての証明書を次のストアに配置する\)** を選択し、場所として **[エンタープライズの信頼]** を選択します。 **[OK]** を選択して証明書ストアの選択ダイアログ ボックスを閉じ、 **[次へ]** をクリックします。

   ![証明書のインポート用に証明書ストアを構成する](./media/prepare-pki-certs/3.png)

   a. PFX をインポートする場合は、追加のダイアログが表示されます。 **[秘密キーの保護]** ページで、証明書ファイルのパスワードを入力し、 **[Mark this key as exportable. This allows you to back up or transport your keys at a later time]\(このキーをエクスポート可能としてマークします。これにより、後でキーをバックアップまたは転送できるようになります\)** オプションを有効にします。 **[次へ]** を選択します。

   ![キーをエクスポート可能としてマークする](./media/prepare-pki-certs/2.png)

1. **[完了]** を選択してインポートを完了します。

> [!NOTE]
> Azure Stack Hub 用に証明書をインポートすると、証明書の秘密キーがクラスター記憶域に PKCS 12 ファイル (PFX) として保存されます。

### <a name="export-the-certificate"></a>証明書をエクスポートする

証明書管理者 MMC コンソールを開き、ローカル コンピューターの証明書ストアに接続します。

1. Microsoft 管理コンソールを開きます。 Windows 10 でコンソールを開くには、 **[スタート] メニュー**を右クリックし、 **[実行]** を選択してから、「**mmc**」と入力して Enter キーを押します。

2. **[ファイル]**  >  **[スナップインの追加と削除]** の順に選択してから、 **[証明書]** を選択して **[追加]** を選択します。

    ![Microsoft 管理コンソールで証明書スナップインを追加する](./media/prepare-pki-certs/mmc-2.png)

3. **[コンピューター アカウント]** を選択し、 **[次へ]** を選択します。 **[ローカル コンピューター]** を選択し、 **[完了]** を選択します。 **[OK]** を選択して [スナップインの追加と削除] ページを閉じます。

    ![Microsoft 管理コンソールで証明書スナップインを追加するためのアカウントを選択する](./media/prepare-pki-certs/mmc-3.png)

4. **[証明書]**  >  **[エンタープライズの信頼]**  >  **[証明書の場所]** の順に進みます。 右側に証明書が表示されることを確認します。

5. 証明書管理者コンソールのタスク バーから、 **[アクション]**  >  **[すべてのタスク]**  >  **[エクスポート]** の順に選択します。 **[次へ]** を選択します。

   > [!NOTE]
   > Azure Stack Hub の証明書の数によっては、このプロセスを複数回完了する必要があります。

6. **[はい、秘密キーをエクスポートします]** を選択し、 **[次へ]** をクリックします。

7. [エクスポート ファイルの形式] セクションで、次の操作を行います。
    
   - **[Include all certificates in the certificate if possible]\(証明書にすべての証明書を含める (可能な場合)\)** を選択します。  
   - **[Export all Extended Properties]\(すべての拡張プロパティをエクスポートする\)** を選択します。  
   - **[証明書のプライバシーを有効にする ]** を選択します。  
   - **[次へ]** をクリックします。  
    
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