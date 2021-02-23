---
title: Azure Stack Hub に Event Hubs をインストールするための前提条件
description: Azure Stack Hub に Event Hubs リソース プロバイダーをインストールするにあたり、必要な前提条件について説明します。
author: BryanLa
ms.author: bryanla
ms.service: azure-stack
ms.topic: how-to
ms.date: 12/09/2019
ms.reviewer: jfggdl
ms.lastreviewed: 12/09/2019
ms.openlocfilehash: b4286074828a0cea911ed6ccd977b6b0690e7eb1
ms.sourcegitcommit: 34babe5abf1bceee718011b5c5c25f75e1b03b0c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/17/2021
ms.locfileid: "100562987"
---
# <a name="prerequisites-for-installing-event-hubs-on-azure-stack-hub"></a>Azure Stack Hub に Event Hubs をインストールするための前提条件

Azure Stack Hub に Event Hubs をインストールするには、次の前提条件を満たしておく必要があります。 すべての手順を完了するには、**数日から数週間のリード タイムが必要になる場合があります**。

> [!IMPORTANT]
> これらの前提条件は、少なくとも 4 ノードの Azure Stack Hub 統合システムが既にデプロイされていることを前提としています。 Event Hubs リソース プロバイダーは、Azure Stack Development Kit (ASDK) ではサポートされていません。

> [!IMPORTANT]
> Event Hubs には、Azure Stack Hub 2005 ビルド バージョン以上が必要です。 Azure Stack Hub のビルドが増分であることに注意してください。 たとえば、バージョン 1910 がインストールされている場合は、まず [2002](./release-notes.md?view=azs-2002&preserve-view=true#2002-build-reference) にアップグレードしてから、2005 にアップグレードする必要があります。 つまり、途中のビルドをスキップすることはできません。

## <a name="common-prerequisites"></a>一般的な前提条件

[!INCLUDE [Common RP prerequisites](../includes/resource-provider-prerequisites.md)]

## <a name="event-hubs-prerequisites"></a>Event Hubs の前提条件

1. Event Hubs 用の公開キー基盤 (PKI) SSL 証明書を入手します。 サブジェクトの別名 (SAN) は、`CN=*.eventhub.<region>.<fqdn>` の名前付けパターンに従っている必要があります。 サブジェクト名を指定することもできますが、証明書を処理する際に Event Hubs によってこれが使用されることはありません。 サブジェクトの別名のみが使用されます。 詳細な要件の完全な一覧については、[PKI 証明書の要件](azure-stack-pki-certs.md)に関するページをご覧ください。  

   ![証明書の例](media/event-hubs-rp-prerequisites/certificate-example.png)

   > [!NOTE]
   > **PFX ファイルはパスワードで保護する必要があります**。 後のインストール時に、パスワードを要求されます。

2. 必ず、[証明書の検証](azure-stack-validate-pki-certs.md)に関する記事に目を通してください。 この記事には、Event Hubs リソース プロバイダーに使用する証明書を準備して検証する方法が示されています。 

## <a name="next-steps"></a>次のステップ

次に、[Event Hubs リソース プロバイダーをインストールします](event-hubs-rp-install.md)。
