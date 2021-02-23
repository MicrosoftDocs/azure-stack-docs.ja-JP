---
title: Azure Stack Hub ラグド内でスケール ユニット ノードを使用するために必要な知識
description: Azure Stack Hub ラグド内でスケール ユニット ノードを使用するために必要な知識について説明します
author: PatAltimore
ms.topic: how-to
ms.date: 11/13/2020
ms.author: patricka
ms.reviewer: ''
ms.lastreviewed: ''
ms.openlocfilehash: b96bf9ecd1f46495da2aa4c2d49d66cc0aff1b53
ms.sourcegitcommit: 5ea0e915f24c8bcddbcaf8268e3c963aa8877c9d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2021
ms.locfileid: "100487801"
---
# <a name="required-knowledge-for-working-with-scale-unit-nodes-in-azure-stack-hub-ruggedized"></a>Azure Stack Hub ラグド内でスケール ユニット ノードを使用するために必要な知識

FRU の手順を完了するには、次の概念、ガイド、および Web サイトに習熟し、アクセスできる必要があります。

## <a name="privileged-access-workstation-and-the-privileged-endpoint"></a>特権アクセス ワークステーションと特権エンドポイント

特権アクセス ワークステーション (PAW) は、インターネットおよび外部の脅威ベクトルから保護された専用のワークステーションです。 これは、ハードウェア ライフサイクル ホスト (HLH) に配置することも、Azure Stack Hub スケール ユニット ノードへのルーティング可能なアクセスを使用して、外部の顧客ネットワークに配置することもできます。

PAW にアクセスするには、リモート デスクトップを使用してログインする必要があります。 顧客から資格情報と IP アドレスを取得します。

このコンピューターから、特権エンドポイント (PEP) にアクセスすることもできます。

## <a name="azure-stack-hub-administrator-portal"></a>Azure Stack Hub 管理者ポータル

顧客から管理者ポータルの資格情報と URL を取得します。
詳細については、「[Azure Stack Hub で](../../operator/azure-stack-manage-portals.md)
[管理者ポータルを使用する](../../operator/azure-stack-manage-portals.md)」を参照してください。


