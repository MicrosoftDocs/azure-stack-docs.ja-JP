---
title: Azure Stack ADFS のユーザーを追加する | Microsoft Docs
description: Azure Stack の ADFS デプロイ用にユーザーを追加する方法について説明します
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
ms.openlocfilehash: d50eb52de39c789498928a7b5e2227998872b937
ms.sourcegitcommit: 80775f5c5235147ae730dfc7e896675a9a79cdbe
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/03/2019
ms.locfileid: "66458972"
---
# <a name="add-azure-stack-users-in-ad-fs"></a>AD FS の Azure Stack ユーザーを追加する
**[Active Directory ユーザーとコンピューター]** スナップインを使用して、AD FS を ID プロバイダーとして利用している Azure Stack 環境にユーザーを追加できます。

## <a name="add-windows-server-active-directory-users"></a>Windows Server Active Directory ユーザーを追加する
> [!TIP]
> この例では、既定の azurestack.local ASDK アクティブ ディレクトリを使用します。 

1. Windows 管理ツールにアクセスできるアカウントを使用してコンピューターにログインし、新しい Microsoft 管理コンソール (MMC) を開きます。
2. **[ファイル]、[スナップインの追加と削除]** の順に選択します。
3. **[Active Directory ユーザーとコンピューター]**  >  **[AzureStack.local]**  >  **[Users]** を選択します。
4. **[操作]**  >  **[新規作成]**  >  **[ユーザー]** の順に選択します。
5. [新しいオブジェクト - ユーザー] で、ユーザー情報の詳細を入力します。 **[次へ]** を選択します。
6. パスワードの指定と確認入力を行います。
7. **[次へ]** を選択して、値を確定します。 **[完了]** を選択して、ユーザーを作成します。


## <a name="next-steps"></a>次の手順
[サービス プリンシパルの作成](azure-stack-create-service-principals.md)