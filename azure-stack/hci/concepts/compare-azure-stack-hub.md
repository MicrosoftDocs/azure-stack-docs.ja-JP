---
title: Azure Stack HCI と Azure Stack Hub の比較
description: このトピックは、Azure Stack HCI または Azure Stack Hub が組織に適しているかどうかを判断するのに役立ちます。
ms.topic: conceptual
author: khdownie
ms.author: v-kedow
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 02/04/2021
ms.openlocfilehash: 7c501650d5c9a4d2d6df66269f689d80ffd3f8e0
ms.sourcegitcommit: 283b1308142e668749345bf24b63d40172559509
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/05/2021
ms.locfileid: "99570686"
---
# <a name="compare-azure-stack-hci-to-azure-stack-hub"></a>Azure Stack HCI と Azure Stack Hub の比較

> 適用対象:Azure Stack HCI バージョン 20H2、Azure Stack Hub

お客様の組織がデジタル変革を進める中で、パブリック クラウド サービスを使用して、最新のアーキテクチャ上に構築し、従来のアプリを更新することで、より迅速に前進できることに気付くことがあります。 ただし、技術上の障害や規制の問題などの理由により、多くのワークロードをオンプレミスに残す必要があります。 次の表を使用すると、必要なところに必要なものを提供し、ワークロードの場所に関係なくそのクラウド イノベーションを実現する Microsoft ハイブリッド クラウド戦略を判断するのに役立ちます。

| Azure Stack HCI | Azure Stack Hub |
| --------------- | --------------- |
| 同じスキル、使い慣れたプロセス | 新しいスキル、革新的なプロセス |
| お客様のデータセンターを Azure サービスに接続する | お客様のデータセンター内での Azure サービス |

## <a name="when-to-use-azure-stack-hci"></a>Azure Stack HCI を使用する局面

次の表は、Azure Stack Hub より Azure Stack HCI の方が適しているユース ケースを比較したものです。

| Azure Stack HCI                                                                 | Azure Stack Hub                                                                         |
| ------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| リモート オフィスやブランチの場合に占有領域を最小限に抑えるには Azure Stack HCI を使用します。 たった 2 台のサーバーとスイッチレスのバックツーバック ネットワークから開始でき、ピークがわかりやすく、価格が手頃です。 | Azure Stack Hub には、少なくとも 4 台のサーバーとそれ専用のネットワーク スイッチが必要です。 |
| Exchange、SharePoint、SQL Server などの従来のエンタープライズ アプリを仮想化する場合、またはファイル サーバー、DNS、DHCP、IIS、Active Directory などの Windows Server の役割を仮想化する場合は、Azure Stack HCI を使用します。 Hyper-V 機能への無制限のアクセスを提供します。| Azure Stack Hub を使用すると、Azure との整合性を確保するため、Hyper-V の構成機能と機能セットが制限されます。 | 
| 大規模な再設計なしで、古いストレージ アレイやネットワーク アプライアンスの代わりにソフトウェアによるインフラストラクチャを使用する場合に、Azure Stack HCI を使用します。 組み込みの Hyper-V、記憶域スペース ダイレクト、およびソフトウェアによるネットワーク (SDN) は直接アクセスして管理できます。 | Azure Stack Hub では、これらのインフラストラクチャ テクノロジは公開されません。 |

## <a name="when-to-use-azure-stack-hub"></a>Azure Stack Hub を使用する局面

次の表は、Azure Stack HCI より Azure Stack Hub の方が適しているユース ケースを比較したものです。

| Azure Stack HCI                                                                 | Azure Stack Hub                                                                          |
| ------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Azure Stack HCI では、マルチテナントをネイティブに適用したり、提供したりすることはありません。 | 併置された複数のテナントの強力な分離および正確な使用状況の追跡とチャージバックを実現するセルフサービスのインフラストラクチャとしてのサービス (IaaS) に Azure Stack Hub を使用します。 サービス プロバイダーやエンタープライズ プライベート クラウドに最適です。 テンプレートは Azure Marketplace にあります。 | 
| Azure Stack HCI は、オンプレミスでサービスとしてのプラットフォーム (PaaS) サービスを実行するためのものではありませんが、[Azure Kubernetes Service](../../aks-hci/overview.md) をホストする機能は含まれています。 | オンプレミスで Web Apps、Functions、Event Hubs などの PaaS サービスを利用するアプリを開発して実行するには、Azure Stack Hub を使用します。 これらのサービスは、Azure 内とまったく同じように Azure Stack Hub 上で実行され、一貫性のあるハイブリッドの開発およびランタイム環境を提供します。 |
| Azure Stack HCI には、DevOps ツールがネイティブに含まれていません。 | DevOps プラクティス (コードとしてのインフラストラクチャ、継続的インテグレーションと継続的デプロイ (CI/CD) など) と便利な機能 (Azure と整合性のある VM 拡張機能など) を利用したアプリのデプロイと操作の現代化に Azure Stack Hub を使用します。 開発チームや DevOps チームに最適です。 |

## <a name="next-steps"></a>次のステップ

- [Azure Stack HCI と Windows Server 2019 を比較する](compare-windows-server.md)
