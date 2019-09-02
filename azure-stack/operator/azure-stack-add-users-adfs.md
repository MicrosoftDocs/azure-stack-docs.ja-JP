---
title: AD FS の Azure Stack ユーザーを追加する | Microsoft Docs
description: Active Directory フェデレーション サービス (AD FS) デプロイに Azure Stack ユーザーを追加する方法について説明します。
services: azure-stack
documentationcenter: ''
author: PatAltimore
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/03/2019
ms.author: patricka
ms.reviewer: unknown
ms.lastreviewed: 06/03/2019
ms.openlocfilehash: 4411290b075e105a827de8fb2c8295dfd84e3b50
ms.sourcegitcommit: e8f7fe07b32be33ef621915089344caf1fdca3fd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/28/2019
ms.locfileid: "70118647"
---
# <a name="add-azure-stack-users-in-ad-fs"></a>AD FS の Azure Stack ユーザーを追加する
**[Active Directory ユーザーとコンピューター]** スナップインを使用して、Active Directory フェデレーション サービス (AD FS) を ID プロバイダーとして利用している Azure Stack 環境にユーザーを追加できます。

## <a name="add-windows-server-active-directory-users"></a>Windows Server Active Directory ユーザーを追加する
> [!TIP]
> この例では、既定の azurestack.local ASDK アクティブ ディレクトリを使用します。 

1. Windows 管理ツールにアクセスできるアカウントを使用してコンピューターにサインインし、新しい Microsoft 管理コンソール (MMC) を開きます。
2. **[ファイル]、[スナップインの追加と削除]** の順に選択します。
3. **[Active Directory ユーザーとコンピューター]**  >  **[AzureStack.local]**  >  **[Users]** を選択します。
4. **[操作]**  >  **[新規作成]**  >  **[ユーザー]** の順に選択します。
5. [新しいオブジェクト - ユーザー] で、ユーザーの詳細を入力します。 **[次へ]** を選択します。
6. パスワードの指定と確認入力を行います。
7. **[次へ]** を選択して、値を入力します。 **[完了]** を選択して、ユーザーを作成します。


## <a name="next-steps"></a>次の手順
[サービス プリンシパルの作成](azure-stack-create-service-principals.md)