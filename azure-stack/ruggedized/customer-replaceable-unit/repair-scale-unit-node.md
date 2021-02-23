---
title: スケール ユニット ノードの電源オンと修復
description: スケール ユニット ノードを電源オンにして修復する方法について説明します
author: PatAltimore
ms.topic: how-to
ms.date: 02/05/2021
ms.author: patricka
ms.reviewer: ''
ms.lastreviewed: ''
ms.openlocfilehash: 0a2aa54234b5dccc8f1ce3425906425064463910
ms.sourcegitcommit: 5ea0e915f24c8bcddbcaf8268e3c963aa8877c9d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2021
ms.locfileid: "100488124"
---
# <a name="powering-on-and-repairing-a-scale-unit-node"></a>スケール ユニット ノードの電源オンと修復

**手順**

スケール ユニット ノードを修復して運用環境に戻すには、Azure Stack Hub の修復手順を実行する必要があります。

> [!NOTE]
> この修復手順の所要時間は約 3 時間です。

1.  **管理ポータル** で、ノードを選択し、 **[修復]** を選択します。

    ![ノードと [修復] 操作が選択されている [管理] - [ノード] ページを示すスクリーンショット。](media/image-52.png)

1.  修復するノードに対応する **BMC IP アドレス** を指定して、 **[修復]** を選択します。

    ![ノードが選択され、IP アドレスが強調表示され、[Repair node]\(ノードの修復\) ダイアログが表示されている [管理] - [ノード] ページを示すスクリーンショット。](media/image-53.png)

1.  通知ウィンドウで進行状況を監視します。

    ![[通知] ペインと [Repairing node]\(ノードの修復\) - [実行中] が表示されているスクリーンショット。](media/image-54.png)
    
    
    > [!NOTE]
    > 修復手順が 3 時間以内に完了しなかった場合、または問題が発生した場合は、Microsoft サポートでケースを開き、さらなるトラブルシューティングを行います。
    
    修復処理が完了すると、**実行中の動作状態** に戻ります。
    
