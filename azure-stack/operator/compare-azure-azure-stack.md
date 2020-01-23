---
title: Azure Stack Hub とグローバル Azure の比較 | Microsoft Docs
description: マイクロソフトによって 1 つの Azure エコシステムで Azure と Azure Stack Hub サービス ファミリがどのように提供されているかを説明します
services: azure-stack
documentationcenter: ''
author: justinha
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.date: 05/03/2019
ms.author: justinha
ms.reviewer: unknown
ms.custom: ''
ms.lastreviewed: 03/29/2019
ms.openlocfilehash: 2d350bedd88ba5c3d04b29f833a17bdd19705f05
ms.sourcegitcommit: 320eddb281a36d066ec80d67b103efad7d4f33c8
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/16/2020
ms.locfileid: "76145854"
---
# <a name="differences-between-global-azure-azure-stack-hub-and-azure-stack-hci"></a>グローバル Azure、Azure Stack Hub、および Azure Stack HCI の違い

Microsoft では 1 つの Azure エコシステムで Azure と Azure Stack Hub サービス ファミリを提供しています。 業務でグローバル Azure とオンプレミス リソースのどちらを使用しているかに関係なく、Azure Resource Manager で同じアプリケーション モデル、セルフサービス ポータル、API を使用して、クラウドベースの機能を提供します。

この記事では、グローバル Azure、Azure Stack Hub、および Azure Stack HCI の機能について説明し、組織で Microsoft クラウドベースのサービスを実現するための最適な選択を行うときに役立つ一般的なシナリオの推奨事項を示します。

![Azure エコシステムの概要](./media/compare-azure-azure-stack/azure-family.png)

## <a name="global-azure"></a>グローバル Azure

Microsoft Azure は、ビジネス上の課題への対応を支援するために絶えず拡大を続けるクラウド サービスの集合体です。 世界規模の巨大なネットワークに対し、お気に入りのツールやフレームワークを使ってアプリケーションを自在に構築、管理、デプロイすることができます。

グローバル Azure では、世界 54 の地域で 100 を超えるサービスが提供されています。 グローバル Azure サービスの最新の一覧については、「[*リージョン別の利用可能な製品*](https://azure.microsoft.com/regions/services)」を参照してください。 Azure で利用可能なサービスがカテゴリ別に一覧表示され、一般提供かプレビューかも示されます。

グローバル Azure サービスについて詳しくは、[Azure の作業開始](https://docs.microsoft.com/azure/#pivot=get-started&panel=get-started1)に関するページをご覧ください。

## <a name="azure-stack-hub"></a>Azure Stack Hub

Azure Stack Hub は Azure を拡張したもので、クラウド コンピューティングの俊敏性やイノベーションをオンプレミス環境にもたらします。 オンプレミスに展開した Azure Stack Hub を使用すると、インターネット (および Azure) に接続した環境でも、インターネット接続のない切断された環境でも、一貫性がある Azure のサービスを提供できます。 Azure Stack Hub はグローバル Azure と同じ基盤テクノロジを使用します。これには、コア コンポーネントである、サービスとしてのインフラストラクチャ (IaaS)、サービスとしてのソフトウェア (SaaS)、およびオプションのサービスとしてのプラットフォーム (PaaS) の機能が含まれます。

- Azure VM (Windows および Linux 用)
- Azure Web Apps および Azure Functions
- Azure Key Vault
- Azure Resource Manager
- Azure Marketplace
- Containers
- 管理ツール (プラン、オファー、RBAC など)

Azure Stack Hub は Microsoft ではなくお客様によって運用されるため、Azure Stack Hub の PaaS 機能はオプションです。 つまり、基礎となるインフラストラクチャおよびプロセスをエンド ユーザーから離して抽象化する準備ができている場合は、必要な PaaS サービスをエンド ユーザーに提供できます。 ただし、Azure Stack Hub には、App Service、SQL データベース、MySQL データベースなど、いくつかのオプションの PaaS サービス プロバイダーが含まれています。 これらはリソース プロバイダーとして提供されているため、マルチテナントに対応し、標準の Azure Stack Hub 更新プログラムによって時間経過に応じて更新され、Azure Stack Hub ポータルに表示され、Azure Stack Hub と一体化しています。

上記のリソース プロバイダーに加えて、その他の PaaS サービスが IaaS で実行する [Azure Resource Manager テンプレートベースのソリューション](https://github.com/Azure/AzureStack-QuickStart-Templates)として提供されテストされています。ただし、Azure Stack Hub オペレーターはこれらを次のような PaaS サービスとしてユーザーに提供できます。

- Service Fabric
- Kubernetes コンテナー サービス
- Ethereum ブロックチェーン
- Cloud Foundry

### <a name="example-use-cases-for-azure-stack-hub"></a>Azure Stack Hub のユース ケース例:

- 財務モデリング
- 臨床データおよび診療報酬データ
- IoT デバイス分析
- 小売の品揃えの最適化
- サプライチェーンの最適化
- 産業用 IoT
- 予測的なメンテナンス
- スマート シティ
- 市民エンゲージメント

Azure Stack Hub についての詳細は、「[Azure Stack Hub とは](azure-stack-overview.md)」で確認してください。

## <a name="azure-stack-hci"></a>Azure Stack HCI

[Azure Stack HCI](azure-stack-hci-overview.md) ソリューションでは、ハイパーコンバージド インフラストラクチャ (HCI) ソリューションを使用して、仮想マシンをオンプレミスで実行し、Azure に簡単に接続できます。 規制要件または技術要件を満たすために、一貫性のあるオンプレミスの Azure サービスを使用してクラウド アプリケーションをビルドおよび実行します。 仮想化されたアプリケーションをオンプレミスで実行するだけでなく、Azure Stack HCI では、経年劣化したサーバー インフラストラクチャを交換および整理し、クラウド サービスを利用するために Windows Admin Center を使用して Azure に接続できます。

Azure Stack HCI では、Hyper-V および Windows Server 2019 Software-Defined Datacenter (SDDC) の記憶域スペース ダイレクトによって強化された検証済みの HCI ソリューションが提供されます。 Windows Admin Center は、次のような Azure サービスの管理と統合アクセスのために使用されます。

- Azure Backup
- Azure Site Recovery
- Azure Monitor および更新プログラム

Azure Stack HCI を接続できる Azure サービスの最新一覧については、「[Connecting Windows Server to Azure hybrid services (Windows Server と Azure ハイブリッド サービスの接続)](https://docs.microsoft.com/windows-server/azure-hybrid-services/index)」をご覧ください。

### <a name="example-use-cases-for-azure-stack-hci"></a>Azure Stack HCI のユース ケース例
- リモート オフィスまたはブランチ オフィスのシステム
- データセンターの統合
- 仮想デスクトップ インフラストラクチャ
- ビジネスに不可欠なインフラストラクチャ
- 記憶域のコスト削減
- クラウドの高可用性と障害復旧
- SQL Server などのエンタープライズ アプリ

[Azure Stack HCI Web サイト](https://azure.microsoft.com/overview/azure-stack/hci/)にアクセスして、現在 Microsoft パートナーから提供されている 70 を超える Azure Stack HCI ソリューションを確認してください。

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub の管理の基本](azure-stack-manage-basics.md)

[クイック スタート: Azure Stack Hub 管理ポータルを使用する](azure-stack-manage-portals.md)
