---
title: Azure Stack Hub 用の Red Hat ベースの仮想マシンを提供する
titleSuffix: Azure Stack Hub
description: Red Hat Linux オペレーティング システムを格納した Azure 仮想ハード ディスク (VHD) を作成してアップロードする方法について説明します。
author: sethmanheim
ms.topic: article
ms.date: 3/3/2021
ms.author: sethm
ms.reviewer: kivenkat
ms.lastreviewed: 3/3/2021
ms.openlocfilehash: 92623d76de9f3358edc1d1ffb1cace199f2f6148
ms.sourcegitcommit: f194f9ca4297864500e62d8658674a0625b29d1d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102187284"
---
# <a name="offer-a-red-hat-based-virtual-machine-for-azure-stack-hub"></a>Azure Stack Hub 用の Red Hat ベースの仮想マシンを提供する

この記事では、Red Hat Enterprise Linux の仮想マシン (VM) を Azure Stack Hub で使用できるように準備する方法について説明します。 

## <a name="offer-a-red-hat-based-vm"></a>Red Hat ベースの VM を提供する

Azure Stack Hub 内で Red Hat ベースの VM を提供する方法は 2 つあります。

- Azure Stack Hub Marketplace を介してマシンを追加できます。
    - [Red Hat Cloud Access プログラム](https://www.redhat.com/en/technologies/cloud-computing/cloud-access)の使用条件について理解を深めます。 [Red Hat サブスクリプション マネージャー](https://access.redhat.com/management/cloud)で、Cloud Access の Red Hat サブスクリプションを有効にします。 Cloud Access に登録するには、Azure Stack Hub が登録されている Azure サブスクリプションを手元に用意しておく必要があります。
    - Red Hat Enterprise Linux イメージは Azure Stack Hub 上のプライベート オファリングです。 このオファリングをご自身の **[Marketplace Management]\(Marketplace の管理\)** タブで使用できるようにするには、[アンケートを完了](https://forms.office.com/pages/responsepage.aspx?id=v4j5cvGGr0GRqy180BHbR_e32WQju3tMrgXNcUR94AVUNkJTWjdQRjc3TzFLREdGU0dIVFRUQ1JCSi4u)する必要があります。 [Marketplace Management]\(Marketplace の管理\) の **[Add from Azure]\(Azure から追加\)** タブに表示されるようになるには、アンケートを投稿してから 7 営業日かかります。
    - 詳細については、「[Azure Stack Hub Marketplace の概要](azure-stack-marketplace.md)」をご覧ください。
- 独自のカスタムを Azure Stack Hub に追加してから、Marketplace 内でイメージを提供することができます。 
    1. Red Hat クラウド アクセスが必要になります。
    2. Azure および Azure Stack Hub 用にイメージを準備する手順については、「[Azure 用の Red Hat ベースの仮想マシンの準備](/azure/virtual-machines/linux/redhat-create-upload-vhd)」をご覧ください。
    3. Azure Stack Hub Marketplace 内でご自身のカスタム イメージを提供する手順については、[Marketplace 項目の作成と発行](azure-stack-create-and-publish-marketplace-item.md)に関するページをご覧ください。

## <a name="next-steps"></a>次のステップ

Red Hat Enterprise Linux の実行が認定されているハイパーバイザーの詳細については、[Red Hat の Web サイト](https://access.redhat.com/certified-hypervisors)を参照してください。