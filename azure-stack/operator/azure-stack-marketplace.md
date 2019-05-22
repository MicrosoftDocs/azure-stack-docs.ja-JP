---
title: Azure Stack でのカスタム Marketplace アイテムの発行 (クラウド オペレーター) | Microsoft Docs
description: Azure Stack オペレーターとして、Azure Stack でカスタム Marketplace アイテムを発行する方法について説明します。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: 60871cbb-eed2-433c-a76d-d605c7aec06c
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/07/2019
ms.author: sethm
ms.reviewer: unknown
ms.lastreviewed: 05/07/2019
ms.openlocfilehash: 70a1e1223accda5dfb273372da486588fb030afc
ms.sourcegitcommit: ccd86bd0862c45de1f6a4993f783ea2e186c187a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/06/2019
ms.locfileid: "65172744"
---
# <a name="azure-stack-marketplace-overview"></a>Azure Stack Marketplace の概要

*適用対象:Azure Stack 統合システムと Azure Stack Development Kit*

Azure Stack Marketplace は、Azure Stack 向けにカスタマイズされたサービス、アプリケーション、およびリソースのコレクションです。 リソースには、ネットワーク、仮想マシン、ストレージなどが含まれます。 Marketplace を使用して、新しいリソースを作成したり、新しいアプリケーションをデプロイしたり、使用するアイテムを参照して選択したりします。 ユーザーが Marketplace アイテムを使用するには、アイテムへのアクセスが許可されるサービスにサブスクライブする必要があります。

Azure Stack オペレーターとして、Marketplace に追加 (公開) するアイテムを決定します。 データベース、App Services などのアイテムを公開できます。 公開すると、アイテムはすべてのユーザーに表示されます。 作成したカスタム アイテムを公開することも、増え続けている [Azure Marketplace アイテムの一覧](azure-stack-marketplace-azure-items.md)からアイテムを公開することもできます。 Marketplace にアイテムを公開すると、5 分以内にユーザーがそれらを表示できます。

> [!CAUTION]  
> ギャラリー アイテム アーティファクト (イメージや JSON ファイルを含む) はすべて、Azure Stack Marketplace で使用可能にした後、認証なしでアクセスできます。 Marketplace アイテムを発行するときの考慮事項については、「[Marketplace アイテムを作成および発行する](azure-stack-create-and-publish-marketplace-item.md)」を参照してください。

Marketplace を開くには、管理者ポータルで **[+ リソースの作成]** を選択します。

![マーケットプレース](media/azure-stack-marketplace/marketplace1.png)

## <a name="marketplace-items"></a>Marketplace アイテム

Azure Stack Marketplace の項目は、ユーザーがダウンロードおよび使用できるサービス、アプリケーション、またはリソースです。 プランやオファーなどの管理アイテムを含むすべての Azure Stack Marketplace アイテムがすべてのユーザーに表示されます。 これらのアイテムは、表示するためのサブスクリプションは必要ありませんが、ユーザーは使用できません。

各 Marketplace アイテムには以下のものが含まれます。

* リソース プロビジョニングのための Azure Resource Manager テンプレート。
* 文字列、アイコン、その他のマーケティング資料などのメタデータ。
* ポータルでアイテムを表示するための書式設定情報。

Marketplace に公開されるすべてのアイテムは、Azure ギャラリー パッケージ (azpkg) 形式を使用します。 デプロイまたはランタイム リソース (コード、ソフトウェアの .zip ファイル、または仮想マシン イメージ) を Marketplace アイテムの一部としてではなく、Azure Stack に個別に追加します。

Azure Stack バージョン 1803 以降では、Azure からダウンロードするときや、カスタム画像をアップロードするときに画像はスパース ファイルに変換されます。 このプロセスは画像を追加するときに時間がかかりますが、領域を節約し、画像の展開時間を短縮することができます。 変換は新しい画像にのみ適用されます。 既存の画像は変更されません。

## <a name="next-steps"></a>次の手順

* [Marketplace アイテムのダウンロード](azure-stack-download-azure-marketplace-item.md)  
* [Marketplace アイテムの作成と発行](azure-stack-create-and-publish-marketplace-item.md)
