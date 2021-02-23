---
title: オペレーティング システム ディスクの交換
description: オペレーティング システム ディスクを交換する方法について学習します
author: PatAltimore
ms.topic: how-to
ms.date: 02/05/2021
ms.author: patricka
ms.reviewer: ''
ms.lastreviewed: ''
ms.openlocfilehash: 24447611a4b3c775b92f4d163bb250060e29417e
ms.sourcegitcommit: 5ea0e915f24c8bcddbcaf8268e3c963aa8877c9d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2021
ms.locfileid: "100487835"
---
# <a name="replacing-an-operating-system-disk"></a>オペレーティング システム ディスクの交換

スケール ユニット ノードで障害が発生したオペレーティング システム ディスクを交換するには、次の手順に従います。

## <a name="prerequisites"></a>前提条件

1.  このガイドの冒頭にある "*メモ、注意、警告*" を確認します。

2.  使用上の注意事項を確認します。

3.  スケール ユニット ノードを使用している場合は、Azure Stack Hub ラグド内でスケール ユニット ノードを使用するために必要な知識。

4.  スケール ユニット ノードのアクセスと正常性の検証を完了します。

5.  スケール ユニット ノードの電源オフを完了します。

    Azure Stack Hub のスケール ユニット ノードの場合、オペレーティング システムは Boot Optimized Storage Solution (BOSS) カード上にあるミラー化された M.2 SSD モジュールの組から実行されます。 オペレーティング システム ディスクを交換するには、システムの電源をオフにしておく必要があります。
    
## <a name="steps"></a>手順

1.  ラック内の物理ノードを見つけます。

2.  ノードの電源がオフになっていることを確認します。 電源の LED は、オレンジ色になっているはずです。

    > [!CAUTION]
    > 使用中のサーバーのケーブルを抜く前に、確実に、各ケーブルに適切なラベルを付けます。 ケーブルは、同じポートに再接続する必要があります。
    
3.  障害が発生した M.2 SSD モジュールを交換します。

    スケール ユニット ノードについては、[M.2 SSD モジュールの交換プロセス](https://www.dell.com/support/manuals/us/en/04/poweredge-r640/per640_ism_pub/dell-emc-poweredge-r640-overview?guid=guid-f39be9ba-158c-45e3-b8b1-f07bb750d6d4)に従います。
    
4.  ノードの電源をオンにします。

    電源に再接続した後で、サーバーが自動的に再起動しない場合は、電源ボタンを押して、ノードを再びオンにします。
    
## <a name="next-steps"></a>次の手順

1.  スケール ユニット ノードの電源オンと修復を完了します。

2.  スケール ユニット ノードの正常性の検証を完了します。

