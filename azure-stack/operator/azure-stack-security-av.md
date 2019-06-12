---
title: Azure Stack 上で Windows Defender ウイルス対策を更新する
description: Azure Stack でウイルス対策を最新の状態に保つ方法の詳細
services: azure-stack
author: PatAltimore
manager: femila
ms.service: azure-stack
ms.topic: article
ms.date: 06/10/2019
ms.author: patricka
ms.reviewer: fiseraci
ms.lastreviewed: 06/10/2019
ms.openlocfilehash: 6e9c4a5334ce44da777126f09ca0ecf578ccfcbd
ms.sourcegitcommit: af63214919e798901399fdffef09650de4176956
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/11/2019
ms.locfileid: "66828111"
---
# <a name="update-windows-defender-antivirus-on-azure-stack"></a>Azure Stack 上で Windows Defender ウイルス対策を更新する

[Windows Defender ウイルス対策](https://docs.microsoft.com/windows/security/threat-protection/windows-defender-antivirus/windows-defender-antivirus-in-windows-10)はセキュリティとウイルス対策を提供するマルウェア対策ソリューションです。 すべての Azure Stack インフラストラクチャ コンポーネント (Hyper-V ホストと仮想マシン) は、Windows Defender ウイルス対策によって保護されています。 保護を最新に保つために、Windows Defender ウイルス対策の定義、エンジン、プラットフォームには、定期的な更新が必要です。 更新を適用する方法は、構成によって変わります。

## <a name="connected-scenario"></a>接続されているシナリオ

マルウェア対策の定義とエンジンの更新では、Azure Stack の[更新リソース プロバイダー](azure-stack-updates.md#the-update-resource-provider)が、1 日に複数回、マルウェア対策の定義とエンジンの更新をダウンロードします。 各 Azure Stack インフラストラクチャ コンポーネントは、更新リソース プロバイダーから更新を取得し、それを自動的に適用します。

マルウェア対策プラットフォームの更新では、[Azure Stack の月次更新](azure-stack-apply-updates.md)が適用されます。 Azure Stack の月次更新には、月ごとの Windows Defender ウイルス対策プラットフォームの更新が含まれています。

## <a name="disconnected-scenario"></a>切断されたシナリオ

 [Azure Stack の月次更新](azure-stack-apply-updates.md)を適用します。 Azure Stack の月次更新には、月ごとの Windows Defender ウイルス対策の定義、エンジン、プラットフォームの更新が含まれています。

## <a name="next-steps"></a>次の手順

[Azure Stack のセキュリティについて詳しく学習する](azure-stack-security-foundations.md)
