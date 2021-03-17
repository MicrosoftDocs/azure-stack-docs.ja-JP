---
title: Azure Kubernetes Service on Azure Stack HCI クラスターを監視する
description: Azure Kubernetes Service on Azure Stack HCI クラスターに対するログ データを監視および表示します
author: v-susbo
ms.topic: how-to
ms.date: 01/26/2021
ms.author: v-susbo
ms.reviewer: ''
ms.openlocfilehash: 7bed9a767728525ae1e40ce4f0e607cc6f034055
ms.sourcegitcommit: 2c6418ee465e67edd417961b1f5211b2e09dbd5f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102116803"
---
# <a name="monitor-aks-on-azure-stack-hci-clusters"></a>AKS on Azure Stack HCI クラスター を監視する

AKS on Azure Stack HCI で使用できる監視およびログ ソリューションは 2 種類あります。 この 2 つのソリューションの比較を次に示します。 

| 解決策  | Azure 接続  | サポートとサービス  | コスト | デプロイ |
| ------- |  ------------  | ---------  | --------------  | ---------------- |
| Azure Monitor | Azure Arc for Kubernetes を使用して AKS on Azure Stack HCI クラスターを Azure に接続する必要がある | Microsoft による完全なサポートと保守 | Azure Monitor サービスへのサインアップが必要 |  [クラスターの監視](https://docs.microsoft.com/azure/azure-monitor/containers/container-insights-overview)に Azure Arc を使用 |
| オンプレミスの監視とログ | Azure 接続は不要 | Microsoft (サポート契約または SLA なし)、コミュニティおよび外部ベンダーがオープンソース ソフトウェアとしてサポート  | ベンダー依存 | 顧客のニーズに基づく。[オンプレミスの監視を使用したクラスター監視](#use-on-premises-monitoring)に関するトピックをご覧ください |

Azure Stack HCI クラスターで AKS と共に Azure Monitor を使用するには、[Azure Monitor の概要](https://docs.microsoft.com/azure/azure-monitor/containers/container-insights-overview)に関するページを参照してください。 

## <a name="use-on-premises-monitoring"></a>オンプレミス監視を使用する

運用環境でアプリを実行する場合、クラスター上で、コントロール プレーン ノードとワークロードの正常性、パフォーマンス、およびリソースの利用状況を監視することが非常に重要です。 オンプレミス監視ソリューションを設定する方法については、[Prometheus と Grafana のインストール](https://github.com/microsoft/AKS-HCI-Apps/tree/main/Monitoring)に関するページをご覧ください。 この監視ソリューションには、次の 2 つのツールが含まれています。 

- **Prometheus** は、コンテナ化されたワークロードの監視に使用できる監視およびアラート ツールキットです。 Prometheus は、さまざまな種類のコレクターおよびエージェントと連携することで、メトリックを収集し、クエリの実行やレポート表示が可能なデータベースに格納します。 

- **Grafana** は、Grafana ダッシュボード上でのメトリックの表示、クエリの実行、および視覚化に使用されるツールです。 Prometheus をデータ ソースとして使用するように事前構成されています。 

### <a name="install-monitoring-tools"></a>監視ツールをインストールする

ご利用の環境内で簡単な監視ソリューションを迅速に設定するには、[Setup-Monitoring.ps1](https://github.com/microsoft/AKS-HCI-Apps/tree/main/Monitoring#easy-steps-to-setup-monitoring-to-use-local-port-forward-to-access-grafana) PowerShell スクリプトを実行します。 このスクリプトには、監視ソリューションを迅速に稼働させるのに必要なすべての構成手順が含まれています。 

Grafana にイングレス コントローラーを含めるには、[イングレス コントローラーを使用して Grafana にアクセス](https://github.com/microsoft/AKS-HCI-Apps/tree/main/Monitoring#detailed-steps-to-setup-monitoring-to-use-ingress-controller-to-access-grafana)する手順に従います。 イングレス コントローラーは、リバース プロキシ、構成可能なトラフィック ルーティング、およびトランスポート層セキュリティ (TLS) 終端を、Kubernetes サービスに提供するソフトウェアです。

### <a name="on-premises-logging"></a>オンプレミスのログ

トラブルシューティングと診断にはログが不可欠です。 AKS on Azure Stack HCI 内のログ ソリューションは、Elasticsearch、Fluent Bit、および Kibana (EFK) に基づいています。 これらのコンポーネントはすべて、コンテナーとしてデプロイされます。 

- Fluent Bit は、さまざまなソースからデータとログを収集してから、フォーマット、統合し、Elasticsearch に格納するログ プロセッサおよびフォワーダーです。 
- Elasticsearch は、高速な検索および Data Analytics 用にログを一元的に保存できる分散検索および分析エンジンです。  
- Kibana は、Web ダッシュボード内に対話型の視覚化を提供します。 このツールを使用すると、Elasticsearch 内に格納されているログを表示し、クエリを実行して、グラフやダッシュボードを使って視覚化することができます。

オンプレミスのログ ソリューションを設定するには、[Kibana にアクセスするためにログを設定](https://github.com/microsoft/AKS-HCI-Apps/tree/main/Logging#easy-steps-to-setup-logging-to-use-local-port-forward-to-access-kibana)する手順をご覧ください。 この記事には、クラスター全体でのコンテナー ログの収集、集計、およびクエリの実行に必要なすべてのコンポーネントが含まれています。 

詳細な構成手順については、[Windows ログ](https://github.com/microsoft/AKS-HCI-Apps/tree/main/Logging#detailed-steps-to-setup-logging)に関するトピックをご覧ください。

## <a name="next-steps"></a>次のステップ

- [Kubernetes クラスターに Linux アプリケーションをデプロイします](./deploy-linux-application.md)。
- [Kubernetes クラスターに Windows Server アプリケーションをデプロイします](./deploy-windows-application.md)。
- [Kubernetes の主要な概念](kubernetes-concepts.md)。
