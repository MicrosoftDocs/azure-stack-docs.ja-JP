---
title: Azure Stack Hub でのお客様への請求と配賦
description: Azure Stack Hub ユーザーにどのようにリソース使用量が請求されるか、および分析と配賦のために請求情報にアクセスする方法を学習します。
author: sethmanheim
ms.topic: article
ms.date: 02/16/2021
ms.author: sethm
ms.reviewer: alfredop
ms.lastreviewed: 03/21/2019
ms.openlocfilehash: 478014ac48ccc060281f8feab2271f1c5af1ed79
ms.sourcegitcommit: 34babe5abf1bceee718011b5c5c25f75e1b03b0c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/17/2021
ms.locfileid: "100562558"
---
# <a name="usage-and-billing-in-azure-stack-hub"></a>Azure Stack Hub での使用量と請求

この記事では、Azure Stack Hub ユーザーにどのようにリソース使用量が請求されるか、および分析と配賦のために請求情報にアクセスする方法を説明します。

Azure Stack Hub では、使用されたリソースの使用量データが収集されてグループ化され、このデータが Azure Commerce に転送されます。 Azure Commerce では、Azure の使用量に対する課金と同様に、Azure Stack Hub の使用量に対して課金が行われます。

また、請求アダプターを使用して、使用量データを取得し、独自の請求システムや配賦システムにエクスポートしたり、Microsoft Power BI などのビジネス インテリジェンス ツールにエクスポートしたりすることもできます。

## <a name="usage-pipeline"></a>使用量パイプライン

Azure Stack Hub 内の各リソース プロバイダーは、リソース使用率に従って使用量データを投稿します。 使用状況サービスにより、使用量データが定期的に (時間単位および日単位で) 集計され、使用量データベースに保存されます。 Azure Stack Hub のオペレーターとユーザーは、Azure Stack Hub リソース使用量 API を介して、保存されている使用量データにアクセスできます。

[Azure Stack Hub インスタンスを Azure に登録](azure-stack-registration.md)している場合、Azure Commerce に使用量データを送信するように Azure Stack Hub が構成されます。 データが Azure にアップロードされたら、課金ポータルまたは Azure リソース使用量 API を使用してデータにアクセスできます。 Azure にレポートされる使用量データの詳細については、[使用量データのレポート](azure-stack-usage-reporting.md)に関する記事をご覧ください。  

次の図は、使用量パイプラインの主要なコンポーネントを示しています。

![使用量パイプライン](media/azure-stack-billing-and-chargeback/usagepipeline.svg)

## <a name="what-usage-information-can-i-find-and-how"></a>確認できる使用量情報とその確認方法

コンピューティング、ストレージ、ネットワークなど、Azure Stack Hub のリソース プロバイダーでは、サブスクリプションごとに 1 時間間隔で使用量データが生成されます。 使用量データには、リソース名、使用されたサブスクリプション、使用された量など、使用されたリソースに関する情報が含まれます。 測定 ID リソースについては、[Usage API の FAQ](azure-stack-usage-related-faq.md) に関するページをご覧ください。

使用量データが収集されると、そのデータが [Azure に報告され](azure-stack-usage-reporting.md)、請求書が生成されます。請求書は、Azure Billing Portal から表示できます。

> [!NOTE]  
> 使用量データのレポートは、Azure Stack Development Kit (ASDK) および容量モデルのライセンスを持つ Azure Stack Hub 統合システムのユーザーにとっては必須ではありません。 Azure Stack Hub のライセンスの詳細については、[パッケージと価格に関するデータ シート](https://azure.microsoft.com/mediahandler/files/resourcefiles/5bc3f30c-cd57-4513-989e-056325eb95e1/Azure-Stack-packaging-and-pricing-datasheet.pdf)をご覧ください。

Azure 課金ポータルには、課金対象のリソースの使用量データが表示されます。 Azure Stack Hub では、課金対象のリソースに加え、幅広い一連のリソースについて使用量情報が取得されます。この情報には、Azure Stack Hub 環境で REST API または PowerShell コマンドレットを使ってアクセスできます。 Azure Stack Hub のオペレーターは、すべてのユーザー サブスクリプションの使用量データを取得できます。 個々のユーザーは、自分自身の使用量の詳細のみを取得できます。

## <a name="usage-reporting-for-multi-tenant-cloud-solution-providers"></a>マルチテナント クラウド ソリューション プロバイダーの使用状況レポート

Azure Stack Hub を使用しているマルチテナント クラウド サービス プロバイダー (CSP) は、各顧客の使用状況を個別に報告できるため、プロバイダーはさまざまな Azure サブスクリプションに使用料金を課金できます。

各顧客の ID は、異なる Azure Active Directory (Azure AD) テナントによって表されます。 Azure Stack Hub では、Azure AD テナントごとに 1 つの CSP サブスクリプションを割り当てることができます。 テナントとそのサブスクリプションを Azure Stack Hub の基本登録に追加できます。 基本登録はすべての Azure Stack Hub インスタンスに対して行われます。 サブスクリプションがテナントに登録されていなくても、ユーザーは引き続き Azure Stack Hub を使用することができ、使用状況は基本登録に使用されたサブスクリプションに送信されます。

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub を登録する](azure-stack-registration.md)
- [Azure Stack Hub 使用状況データの Azure への報告](azure-stack-usage-reporting.md)
- [プロバイダーリソース使用量 API](azure-stack-provider-resource-api.md)
- [テナント リソース使用量 API](azure-stack-tenant-resource-usage-api.md)
- [使用量に関するよくあるご質問 (FAQ)](azure-stack-usage-related-faq.md)
