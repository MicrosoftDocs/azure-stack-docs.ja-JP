---
title: Azure Stack Hub ユーザー サブスクリプションの課金の所有者を変更する
description: Azure Stack Hub ユーザー サブスクリプションの課金の所有者を変更する方法について学習します。
author: justinha
ms.topic: conceptual
ms.date: 09/17/2019
ms.author: justinha
ms.reviewer: shnatara
ms.lastreviewed: 10/19/2018
ms.openlocfilehash: e6404ad425f75dfdbf714e9b4efbf81f4bfdddcb
ms.sourcegitcommit: fd5d217d3a8adeec2f04b74d4728e709a4a95790
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/29/2020
ms.locfileid: "76878949"
---
# <a name="change-the-billing-owner-for-an-azure-stack-hub-user-subscription"></a>Azure Stack Hub ユーザー サブスクリプションの課金の所有者を変更する

Azure Stack Hub オペレーターは、PowerShell を使用して、ユーザー サブスクリプションの課金の所有者を変更することができます。 所有者を変更する理由の 1 つに、組織を離れるユーザーの後任を指定することなどがあります。

サブスクリプションに割り当てられる "*所有者*" には、次の 2 種類があります。

- **課金の所有者**: 既定では、課金の所有者は、オファーからサブスクリプションを取得し、そのサブスクリプションの請求関係を所有するユーザー アカウントです。 このアカウントはサブスクリプションの管理者でもあります。 サブスクリプションでこの所有者に指定できるユーザー アカウントは 1 つだけです。 課金の所有者は、多くの場合、組織またはチームのリーダーです。

  課金の所有者を変更するには、PowerShell コマンドレット [Set-AzsUserSubscription](/powershell/module/azs.subscriptions.admin/set-azsusersubscription) を使用します。  

- **RBAC ロールを介して追加された所有者** - [ロールベースのアクセス制御](azure-stack-manage-permissions.md) (RBAC) を使用して、追加のユーザーに**所有者**ロールを付与できます。 課金の所有者を補完するために、任意の数の追加ユーザー アカウントを所有者として追加できます。 追加の所有者は、サブスクリプションの管理者でもあり、課金の所有者を削除するアクセス許可を除き、サブスクリプションのすべての特権を保有します。

  追加の所有者を管理するために、PowerShell を使用できます。 詳細については、 [こちらの記事](/azure/role-based-access-control/role-assignments-powershell)を参照してください。

## <a name="change-the-billing-owner"></a>課金の所有者の変更

ユーザー サブスクリプションの課金の所有者を変更するには、次のスクリプトを実行します。 スクリプトを実行するために使用するコンピューターでは、Azure Stack Hub に接続し、Azure Stack Hub PowerShell モジュール 1.3.0 以降を実行する必要があります。 詳細については、[Azure Stack Hub PowerShell のインストール](azure-stack-powershell-install.md)に関するページを参照してください。

>[!NOTE]
>マルチテナントの Azure Stack Hub では、新しい所有者を既存の所有者と同じディレクトリに配置する必要があります。 別のディレクトリに配置されているユーザーにサブスクリプションの所有権を与えるには、最初に[そのユーザーをゲストとして自分のディレクトリに招待する](/azure/active-directory/b2b/add-users-administrator)必要があります。

スクリプトを実行する前に、次の値を置き換えてください。

- **$ArmEndpoint**: ご使用の環境用の Resource Manager エンドポイント。
- **$TenantId**: テナント ID。
- **$SubscriptionId**: サブスクリプション ID。
- **$OwnerUpn**: **user\@example.com** など、新しい課金の所有者として追加するアカウント。

```powershell
# Set up Azure Stack Hub admin environment
Add-AzureRmEnvironment -ARMEndpoint $ArmEndpoint -Name AzureStack-admin
Add-AzureRmAccount -Environment AzureStack-admin -TenantId $TenantId

# Select admin subscription
$providerSubscriptionId = (Get-AzureRmSubscription -SubscriptionName "Default Provider Subscription").Id
Write-Output "Setting context to the Default Provider Subscription: $providerSubscriptionId"
Set-AzureRmContext -Subscription $providerSubscriptionId

# Change user subscription owner
$subscription = Get-AzsUserSubscription -SubscriptionId $SubscriptionId
$Subscription.Owner = $OwnerUpn
Set-AzsUserSubscription -InputObject $subscription
```

[!include[Remove Account](../../includes/remove-account.md)]

## <a name="next-steps"></a>次のステップ

- [ロールベースのアクセス制御の管理](azure-stack-manage-permissions.md)
