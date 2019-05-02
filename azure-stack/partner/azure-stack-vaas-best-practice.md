---
title: Azure Stack の検証のベスト プラクティス | Microsoft Docs
description: この記事では、サービスとしての検証のベスト プラクティスについて説明します。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/11/2019
ms.author: mabrigg
ms.reviewer: johnhas
ms.lastreviewed: 03/11/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 30b7a5327a709fb35c3c3360f4bb0246e9a5f75f
ms.sourcegitcommit: 0973dddb81db03cf07c8966ad66526d775ced8b9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/23/2019
ms.locfileid: "64310015"
---
# <a name="best-practices-for-validation-as-a-service"></a>サービスとしての検証のベスト プラクティス

[!INCLUDE [Azure_Stack_Partner](./includes/azure-stack-partner-appliesto.md)]

この記事では、サービスとしての検証 (VaaS) でリソースを管理するためのベスト プラクティスについて説明します。 VaaS リソースの概要については、「[サービスとしての検証の主要概念](azure-stack-vaas-key-concepts.md)」をご覧ください。

## <a name="solution-management"></a>ソリューション管理

### <a name="naming-convention-for-vaas-solutions"></a>VaaS ソリューションの名前付け規則

VaaS に登録するすべてのソリューションに、一貫性のある名前付け規則を使用します。 たとえば、次のようにハードウェアのプロパティに基づいてソリューション名を作成できます。

|製品名 | 一意のハードウェア要素 1 | 一意のハードウェア要素 2 | ソリューション名
|---|---|---|---|
My Solution XYZ |  All Flash | My Switch X01 | MySolutionXYZ_AllFlash_MySwitchX01

### <a name="when-to-create-a-new-vaas-solution"></a>新しい VaaS ソリューションを作成するタイミング

同じハードウェア SKU に対してワークフローを実行する場合は、同じ VaaS ソリューションを使用します。 新しい VaaS ソリューションを作成するのは、ハードウェア SKU に変更が発生した場合だけです。

## <a name="workflow-management"></a>ワークフロー管理

### <a name="naming-convention-for-vaas-workflows"></a>VaaS ワークフローの名前付け規則

すべての VaaS ワークフロー実行に、一貫性のある名前付け規則を使用します。 たとえば、次のようにビルドのプロパティに基づいてワークフロー名を作成します。

|ビルド番号 (メジャー) | Date | ソリューションのサイズ | ワークフロー名
|---|---|---| ---|
1808 | 081518 | 4NODE | 1808_081518_4NODE

## <a name="next-steps"></a>次の手順

- [サービスとしての検証の主要概念](azure-stack-vaas-key-concepts.md)を確認する
