---
title: スケール ユニット ノードのディスク正常性の確認
description: スケール ユニット ノードのディスク正常性を確認する方法について説明します
author: PatAltimore
ms.topic: how-to
ms.date: 11/13/2020
ms.author: patricka
ms.reviewer: ''
ms.lastreviewed: ''
ms.openlocfilehash: 8d872a11ce0312e0e1624a310606190da90c48a2
ms.sourcegitcommit: 5ea0e915f24c8bcddbcaf8268e3c963aa8877c9d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2021
ms.locfileid: "100487971"
---
# <a name="verifying-scale-unit-node-disk-health"></a>スケール ユニット ノードのディスク正常性の確認

1.  特権エンドポイント (PEP) に接続します。

    1.  PEP に接続する方法については、「特権アクセス ワークステーションと特権エンドポイント アクセス」を参照してください。

    1.  接続したら、PEP セッション `Enter-PSSession -Session $pep` を入力します。

2.  仮想ディスクの正常性を取得します。

    1.  `Get-VirtualDisk -cimsession "S-Cluster"` を実行して、仮想ディスクの正常性を確認します。

        システムで **OperationalStatus** の **OK** と **HealthStatus** の **Healthy** が返されない場合は、数分待ってからコマンドを再実行します。
        
        !["OperationsStatus" と "HealthStatus" 列が強調表示されている Windows PowerShell を示すスクリーンショット。](media/image-57.png)
        
    1.  `Get-VirtualDisk -cimsession "S-Cluster" | Get-StorageJob` を実行して、実行中のすべての記憶域ジョブが完了していることを確認します。
    
    1.  結果が返されないことを確認します。 **JobState** で示されているようにジョブがまだ実行中の場合、またはすべてのジョブが完了とマークされている場合は、10 分待ってから、同じコマンドをもう一度実行します。 最終的な状態は、ジョブが一覧表示されないことです。
    
    1.  必要な場合は、「[Azure Stack Hub PowerShell を使用した仮想ディスクの修復状態の確認](../../operator/azure-stack-replace-disk.md#check-the-status-of-virtual-disk-repair-using-azure-stack-hub-powershell)」で記憶域の正常性確認の追加手順を確認できます。
        
