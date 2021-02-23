---
title: Azure Stack Hub 上の Event Hubs 1.2012.1.0 リリースノート
description: バグの修正、機能、更新プログラムのインストール方法など、Azure Stack Hub 上の Event Hubs の1.2012.1.0 リリースについて説明します。
author: BryanLa
ms.author: bryanla
ms.service: azure-stack
ms.topic: article
ms.date: 02/15/2021
ms.reviewer: kalkeea
ms.lastreviewed: 02/15/2021
ms.openlocfilehash: ef28447b87c0647b65c996112c5739bf7c6caccf
ms.sourcegitcommit: 34babe5abf1bceee718011b5c5c25f75e1b03b0c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/17/2021
ms.locfileid: "100563532"
---
# <a name="event-hubs-on-azure-stack-hub-1201210-release-notes"></a>Azure Stack Hub 上の Event Hubs 1.2012.1.0 リリースノート

これらのリリースノートでは、Azure Stack Hub 上の Event Hubs バージョン 1.2012.1.0 での機能強化と修正、および既知の問題について説明します。 

[!INCLUDE [Azure Stack Hub update reminder](../includes/event-hubs-hub-update-banner.md)]

## <a name="updates-in-this-release"></a>このリリースでの更新

このリリースには、次の更新プログラムが含まれています。

- Azure portal SDK 開発者向けに、ポータル バージョン5.0.303.7361 がサポートされるようになりました。
- Event Hubs クラスター用の内部ログ機能の強化。

## <a name="issues-fixed-in-this-release"></a>このリリースで修正された問題

このリリースでは、次の問題点を修正しました。

- アップグレードの問題に対処するための、Event Hubs クラスターへのアップグレード順序の修正。
- クラスターが "アップグレード中" または "アップグレードに失敗しました" 状態のときに、Event Hubs クラスターのクラスター正常性とバックアップ正常性のチェックが実行されていませんでした。 この問題は今回のリリースで修正されています。
- 使用状況レコードに間違った数量が含まれる原因のバグを修正しました。 コア数ではなく、容量ユニット (CU) を出力していました。 以前は、1 CU クラスターが 1 時間の使用量で 1 コアと表示されていました。 ユーザーには、1 CU クラスターの 1 時間の使用量に対して 10 コアという正しい数量が表示されるようになりました。

## <a name="known-issues"></a>既知の問題 

このリリースに既知の問題はありません。

## <a name="next-steps"></a>次のステップ

- 詳細については、「[Azure Stack Hub 上の Event Hubs オペレーターの概要](event-hubs-rp-overview.md)」を参照してください。

