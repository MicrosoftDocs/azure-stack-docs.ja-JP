---
title: Azure Stack でのリージョン管理 | Microsoft Docs
titleSuffix: Azure Stack
description: Azure Stack でのリージョン管理の概要。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: e94775d5-d473-4c03-9f4e-ae2eada67c6c
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/13/2019
ms.author: sethm
ms.reviewer: efemmano
ms.lastreviewed: 11/27/2018
ms.openlocfilehash: 38004b88f43ef59448ca99c3eb2762e5ca63e89c
ms.sourcegitcommit: ca358ea5c91a0441e1d33f540f6dbb5b4d3c92c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/08/2019
ms.locfileid: "73802274"
---
# <a name="region-management-in-azure-stack"></a>Azure Stack でのリージョン管理

*適用対象:Azure Stack 統合システムと Azure Stack Development Kit*

Azure Stack では、"*リージョン*" という概念が使用されています。リージョンとは、Azure Stack インフラストラクチャを構成するハードウェア リソースから成る論理エンティティです。 リージョン管理では、Azure Stack インフラストラクチャを正常に運用するために必要なすべてのリソースを見つけることができます。

1 つの統合システム デプロイ (*Azure Stack クラウド*  と呼ばれます) が 1 つのリージョンを構成します。 各 Azure Stack Development Kit (ASDK) には、**local** という名前の 1 つのリージョンがあります。 2 番目の Azure Stack 統合システムをデプロイするか、または別のハードウェア上に ASDK の別のインスタンスを設定すると、この Azure Stack クラウドは別のリージョンになります。

## <a name="information-available-through-the-region-management-tile"></a>[リージョンの管理] タイルから使用可能な情報

Azure Stack には、 **[Region management]** (リージョン管理) タイルで使用できる一連のリージョン管理機能があります。 このタイルは、管理者ポータルの既定のダッシュボードで Azure Stack オペレーターが使用できます。 このタイルを使用して、Azure Stack リージョンと、リージョン固有のそのコンポーネントを監視および更新できます。

![Azure Stack 管理者ポータルの [region management]\(リージョン管理\) タイル](media/azure-stack-region-management/image1.png)

**[Region management]\(リージョン管理\)** タイル内のリージョンをクリックすると、次の情報にアクセスできます。

[![Azure Stack 管理者ポータルの [Region management]\(リージョン管理\) ブレードにあるペインの説明](media/azure-stack-region-management/regionssm.png "Azure Stack 管理者ポータルの [Region management]\(リージョン管理\) ブレード")](media/azure-stack-region-management/regions.png#lightbox)

1. **リソース メニュー**:さまざまなインフラストラクチャ管理領域にアクセスして、ストレージ アカウントや仮想ネットワークなどのユーザー リソースを表示および管理します。

2. **アラート**:システム全体のアラートの一覧と、各アラートの詳細が表示されます。

3. **更新プログラム**:Azure Stack インフラストラクチャの現在のバージョン、利用可能な更新プログラム、および更新履歴が表示されます。 統合システムを更新することもできます。

4. **リソース プロバイダー**:Azure Stack を実行するために必要なコンポーネントによって提供されるユーザー機能を管理します。 リソース プロバイダーごとに管理エクスペリエンスが異なります。 このエクスペリエンスには、プロバイダー固有のアラート、メトリック、およびリソース プロバイダー固有のその他の管理機能が含まれます。

5. **インフラストラクチャ ロール**:Azure Stack を実行するために必要なコンポーネントです。 アラートをレポートするインフラストラクチャ ロールのみが一覧表示されます。 ロールを選択すると、ロールおよびそのロールが実行されているロール インスタンスに関連付けられているアラートを表示できます。

6. **[プロパティ]** :リージョン管理ブレードへの環境の登録状態と詳細。 状態には、 **[登録済み]** 、 **[未登録]** または **[有効期限切れ]** があります。 登録済みである場合は、Azure Stack の登録に使用した Azure サブスクリプション ID が、登録のリソース グループおよび名前と共に表示されます。

## <a name="next-steps"></a>次の手順

- [Azure Stack での正常性およびアラートの監視](azure-stack-monitor-health.md)
- [Azure Stack での更新の管理](azure-stack-updates.md)
