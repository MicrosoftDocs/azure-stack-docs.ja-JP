---
title: Azure Stack デプロイ ネットワーク トラフィック | Microsoft Docs
description: この記事では、Azure Stack デプロイ ネットワーキング プロセスで予想されることについて説明します。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/12/2019
ms.author: mabrigg
ms.reviewer: wamota
ms.lastreviewed: 08/30/2018
ms.openlocfilehash: 20ec9e6ea8831e6667a282074ba28b16da3180f9
ms.sourcegitcommit: 2a4321a9cf7bef2955610230f7e057e0163de779
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/14/2019
ms.locfileid: "65618981"
---
# <a name="about-deployment-network-traffic"></a>デプロイ ネットワーク トラフィックについて
デプロイを成功させるためには、Azure Stack デプロイ中のネットワーク トラフィック フローのしくみを理解することが重要です。 この記事では、デプロイ プロセス中に予想されるネットワーク トラフィックについて理解できるよう、段階的に説明します。

この図には、デプロイ プロセスに関連するすべてのコンポーネントと接続が示されています。

![Azure Stack デプロイ ネットワーク トポロジ](media/deployment-networking/figure1.png)

> [!NOTE]
> この記事では、接続デプロイの要件について説明します。他のデプロイ方法については、[Azure Stack デプロイ接続モデル](azure-stack-connection-models.md)に関するページを参照してください。

### <a name="the-deployment-vm"></a>デプロイ VM
Azure Stack ソリューションには、Azure Stack コンポーネントをホストするためのサーバーのグループと、Hardware Lifecycle Host (HLH) という名前の特別なサーバーが含まれています。 このサーバーは、ソリューションをデプロイし、そのライフサイクルを管理するために使用されます。デプロイ中、デプロイ VM (DVM) をホストします。

## <a name="deployment-requirements"></a>デプロイ要件
デプロイを正常に完了するための最小要件があります。デプロイを開始する前に、その要件を OEM で検証できます。 次の要件を理解することで、環境を整え、検証に合格できます。

-   [証明書](azure-stack-pki-certs.md)
-   [Azure サブスクリプション](https://azure.microsoft.com/free/?b=17.06)
-   インターネットへのアクセス
-   DNS
-   NTP

> [!NOTE]
> この記事では、最後の 3 つの要件について説明します。 最初の 2 つの詳細については、リンク先のページをご覧ください。

## <a name="deployment-network-traffic"></a>デプロイ ネットワーク トラフィック
DVM は BMC ネットワークからの IP で構成され、インターネットへのネットワーク アクセスを必要とします。 BMC ネットワークのすべてのコンポーネントで外部ルーティングまたはインターネットへのアクセスが必要になるわけではありませんが、このネットワークからの IP を利用する OEM 固有のコンポーネントでも必要になることがあります。

デプロイ中、DVM はお使いのサブスクリプションの Azure アカウントを利用し、Azure Active Directory (Azure AD) に対して認証を行います。 そのために、DVM は一覧にある特定のポートや URL にインターネット アクセスする必要があります。 完全な一覧は、[エンドポイントの公開](azure-stack-integrate-endpoints.md)に関する文書にあります。 DVM は DNS サーバーを利用し、内部コンポーネントによって行われた DNS 要求を外部 URL に転送します。 内部 DNS は転送されてきた要求を DNS フォワーダー アドレスに転送します。このアドレスは、デプロイ前に OEM に指定します。 同じことは NTP サーバーにも当てはまります。このサーバーは、すべての Azure Stack コンポーネントで一貫性を維持し、時刻を同期するために必要な、信頼性のあるタイム サーバーです。

デプロイ中、DVM が必要とするインターネット アクセスは外向きのみです。デプロイ中、内向きの呼び出しは行われません。 DVM はその IP をソースとして利用すること、Azure Stack はプロキシ構成に対応していないことにご注意ください。 そのため、必要であれば、インターネットにアクセスするために透過型プロキシまたは NAT を指定する必要があります。 デプロイ時に、一部の内部コンポーネントは、パブリック VIP を使用して外部ネットワーク経由でインターネットにアクセスし始めます。 デプロイの完了後、Azure と Azure Stack の間の全通信はパブリック VIP を利用した外部ネットワーク経由で行われます。

Azure Stack スイッチのネットワーク構成には、特定のネットワーク ソースと接続先の間のトラフィックを制限するアクセス制御リスト (ACL) が含まれます。 DVM は、アクセスに制限がない唯一のコンポーネントです。HLH でさえ、厳しい制限があります。 ご利用のネットワークからの管理とアクセスを簡単にするためのカスタマイズ オプションについては、OEM にお問い合わせください。 このような ACL が存在するため、デプロイ時には、DNS サーバーと NTP サーバーのアドレスを変更しないことが重要です。 変更する場合、このソリューションのすべてのスイッチを再構成する必要があります。

デプロイの完了後、指定した DNS サーバー アドレスと NTP サーバー アドレスは、システムのコンポーネントによって直接、引き続き使用されます。 たとえば、デプロイの完了後に DNS 要求を確認する場合、DVM IP から外部ネットワーク範囲のアドレスにソースが変わります。

デプロイの完了後、指定した DNS サーバー アドレスと NTP サーバー アドレスは、システムのコンポーネントによって、外部ネットワークを使用して SDN 経由で引き続き使用されます。 たとえば、デプロイの完了後に DNS 要求を確認する場合、DVM IP からパブリック VIP にソースが変わります。

## <a name="next-steps"></a>次の手順
[Azure の登録を検証する](azure-stack-validate-registration.md)
