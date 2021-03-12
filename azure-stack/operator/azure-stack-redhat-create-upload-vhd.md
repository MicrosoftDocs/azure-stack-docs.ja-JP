---
title: Azure Stack Hub 用の Red Hat ベースの仮想マシンを提供する
titleSuffix: Azure Stack Hub
description: Red Hat Linux オペレーティング システムを格納した Azure 仮想ハード ディスク (VHD) を作成してアップロードする方法について説明します。
author: sethmanheim
ms.topic: article
ms.date: 2/18/2021
ms.author: sethm
ms.reviewer: kivenkat
ms.lastreviewed: 2/18/2021
ms.openlocfilehash: 4dc1280b0120c5c64dff6a273ce13ae97dbe980c
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101840305"
---
# <a name="offer-a-red-hat-based-virtual-machine-for-azure-stack-hub"></a>Azure Stack Hub 用の Red Hat ベースの仮想マシンを提供する

この記事では、Red Hat Enterprise Linux の仮想マシン (VM) を Azure Stack Hub で使用できるように準備する方法について説明します。 

## <a name="offer-a-red-hat-based-vm"></a>Red Hat ベースの VM を提供する

Azure Stack Hub 内で Red Hat ベースの VM を提供する方法は 2 つあります。

- Azure Stack Hub Marketplace を介してマシンを追加できます。
    - Red Hat Enterprise Linux イメージは Azure Stack Hub 上のプライベート オファリングです。 このオファリングをご自身の **[Marketplace Management]\(Marketplace の管理\)** タブで使用できるようにするには、[アンケートを完了](https://forms.office.com/pages/responsepage.aspx?id=v4j5cvGGr0GRqy180BHbR_e32WQju3tMrgXNcUR94AVUNkJTWjdQRjc3TzFLREdGU0dIVFRUQ1JCSi4u)する必要があります。
    - 詳細については、「[Azure Stack Hub Marketplace の概要](azure-stack-marketplace.md)」をご覧ください。
- 独自のカスタムを Azure Stack Hub に追加してから、Marketplace 内でイメージを提供することができます。 
    1. Red Hat クラウド アクセスが必要になります。
    2. Azure および Azure Stack Hub 用にイメージを準備する手順については、「[Azure 用の Red Hat ベースの仮想マシンの準備](/azure/virtual-machines/linux/redhat-create-upload-vhd)」をご覧ください。
    3. Azure Stack Hub Marketplace 内でご自身のカスタム イメージを提供する手順については、[Marketplace 項目の作成と発行](azure-stack-create-and-publish-marketplace-item.md)に関するページをご覧ください。

## <a name="next-steps"></a>次のステップ

Red Hat Enterprise Linux の実行が認定されているハイパーバイザーの詳細については、[Red Hat の Web サイト](https://access.redhat.com/certified-hypervisors)を参照してください。