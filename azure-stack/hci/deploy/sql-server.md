---
title: Azure Stack HCI 上で SQL Server をデプロイする (AKS-HCI)
description: このトピックでは、Azure Stack HCI 上で SQL Server をデプロイする方法のガイダンスを提供します。
author: JohnCobb1
ms.author: v-johcob
ms.topic: how-to
ms.date: 01/11/2021
ms.openlocfilehash: 433a9b8b7bf6f57cac936b24d7654b94b8e8c050
ms.sourcegitcommit: f194f9ca4297864500e62d8658674a0625b29d1d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102186808"
---
# <a name="deploy-sql-server-on-azure-stack-hci"></a>Azure Stack HCI 上で SQL Server をデプロイする

>適用対象:Azure Stack HCI、バージョン 20H2、SQL Server (サポートされているすべてのバージョン)

このトピックでは、Azure Stack HCI オペレーティング システム上で SQL Server を計画、構成、およびデプロイする方法に関するガイダンスを提供します。 このオペレーティング システムは、ハイブリッドのオンプレミス環境で仮想化された Windows および Linux のワークロードとそのストレージをホストする、ハイパーコンバージド インフラストラクチャ (HCI) クラスター ソリューションです。

## <a name="solution-overview"></a>ソリューションの概要
Azure Stack HCI では、SQL Server と記憶域スペース ダイレクトを実行するための高可用性で、コスト効率に優れ、柔軟なプラットフォームを提供しています。 Azure Stack HCI は、オンライン トランザクション処理 (OLTP) ワークロード、データ ウェアハウスと BI、および AI とビッグデータの高度な分析を実行できます。

プラットフォームの柔軟性は、ミッション クリティカルなデータベースでは特に重要です。 SQL Server は、Windows Server または Linux のいずれかを使用する仮想マシン (VM) で実行できます。このため、複数のデータベース ワークロードを統合し、必要に応じて Azure Stack HCI 環境に VM を追加できます。 また Azure Stack HCI を使用することで、SQL Server を Azure Site Recovery と統合して、組織のデータについて、信頼性とセキュリティに優れたクラウドベースの移行、復元、および保護ソリューションを提供できます。

## <a name="deploy-sql-server"></a>SQL Server を展開する
このセクションでは、Azure Stack HCI 上で SQL Server 用のハードウェアを取得する方法と、Windows Admin Center を使用してサーバー上でオペレーティング システムを管理する方法について説明します。 SQL Server の設定、監視とパフォーマンスのチューニング、高可用性 (HA) と Azure ハイブリッド サービスの使用に関する情報を取り扱います。

### <a name="step-1-acquire-hardware-from-the-azure-stack-hci-catalog"></a>手順 1:Azure Stack HCI カタログからハードウェアを取得する
最初に、ハードウェアを調達する必要があります。 最も簡単な方法としては、[Azure Stack HCI カタログ](https://hcicatalog.azurewebsites.net)で適切な Microsoft ハードウェア パートナーを見つけて、Azure Stack HCI オペレーティング システムがプレインストールされている統合システムを購入します。 カタログでは、フィルターを適用して、この種類のワークロードに最適化されたベンダーのハードウェアを表示できます。

それ以外の場合は、Azure Stack HCI オペレーティング システムを独自のハードウェアにデプロイする必要があります。 Azure Stack HCI デプロイ オプションの詳細および Windows Admin Center のインストールについては、「[Azure Stack HCI オペレーティング システムのデプロイ](./operating-system.md)」を参照してください。

次に、Windows Admin Center を使用して、[Azure Stack HCI クラスターを作成します](./create-cluster.md)。

### <a name="step-2-install-sql-server-on-azure-stack-hci"></a>手順 2:Azure Stack HCI に SQL Server をインストールする
要件に応じて、Windows Server または Linux のいずれかを実行している VM に SQL Server をインストールできます。

SQL Server のインストール手順については、以下を参照してください。
- [Windows 用の SQL Server インストール ガイド](/sql/database-engine/install-windows/install-sql-server?preserve-view=true&view=sql-server-ver15)。
- [SQL Server on Linux のインストール ガイド](/sql/linux/sql-server-linux-setup?preserve-view=true&view=sql-server-ver15)。

### <a name="step-3-monitor-and-performance-tune-sql-server"></a>手順 3: SQL Server の監視およびパフォーマンス チューニングを行う
Microsoft では、SQL Server のイベントを監視したり、物理データベース デザインをチューニングしたりするための広範なツール セットを用意しています。 ツールは、実行する監視またはチューニングの種類に応じて選択します。

Azure Stack HCI の SQL Server インスタンスのパフォーマンスと正常性を確保するには、「[パフォーマンス監視およびチューニング ツール](/sql/relational-databases/performance/performance-monitoring-and-tuning-tools?preserve-view=true&view=sql-server-ver15)」を参照してください。

SQL Server 2017 と SQL Server 2016 のチューニングについては、[SQL Server 2017 と 2016 でパフォーマンス ワークロードの高いときに推奨される更新プログラムと構成オプション](https://support.microsoft.com/help/4465518/recommended-updates-and-configurations-for-sql-server)に関するページを参照してください。

### <a name="step-4-use-sql-server-high-availability-features"></a>手順 4:SQL Server の高可用性機能を使用する
Azure Stack HCI では、ハードウェア障害の発生時に VM で実行中の SQL Server をサポートするために、[Windows Server フェールオーバー クラスタリングと SQL Server (WSFC)](/sql/sql-server/failover-clusters/windows/windows-server-failover-clustering-wsfc-with-sql-server) を活用します。 また SQL Server では [Always On 可用性グループ](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server) (AG) が提供されており、アプリケーションやソフトウェアの障害に対応するように設計されたデータベースレベルの高可用性を実現します。 WSFC と AG に加えて、Azure Stack HCI では [Always On フェールオーバー クラスター インスタンス](/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server) (FCI) を使用できます。これは、共有記憶域の[記憶域スペース ダイレクト](/windows-server/storage/storage-spaces/storage-spaces-direct-overview) テクノロジに基づいた機能です。

これらのオプションはすべて、クォーラム制御のために Microsoft Azure [クラウド監視](/windows-server/failover-clustering/deploy-cloud-witness)と連動します。 異なる物理ノードに配置されている VM に対しては、WSFC のクラスター [AntiAffinity](/windows-server/failover-clustering/cluster-affinity) を使用することで、Always On 可用性グループを構成するときにホストで障害が発生した場合に、SQL Server のアップタイムを維持することをお勧めします。

### <a name="step-5-set-up-azure-hybrid-services"></a>手順 5:Azure ハイブリッド サービスを設定する
SQL Server のデータとアプリケーションのセキュリティを確保するために使用できる Azure ハイブリッド サービスがいくつかあります。 [Azure Site Recovery](https://azure.microsoft.com/services/site-recovery/) は、サービスとしてのディザスター リカバリー (DRaaS) です。 このサービスを使用して、ワークロードをオンラインで維持するためにアプリケーションの SQL Server バックエンドを保護する方法の詳細については、「[SQL Server のためにディザスター リカバリーを設定する](/azure/site-recovery/site-recovery-sql)」を参照してください。

[Azure Backup](https://azure.microsoft.com/services/backup/) を使用すると、SQL Server の整合性をバックアップおよび復元するエンタープライズ ワークロードおよびサポートを保護するようにバックアップ ポリシーを定義できます。 オンプレミスの SQL データをバックアップする方法の詳細については、[Azure Backup Server のインストール](/azure/backup/backup-azure-microsoft-azure-backup)に関するページを参照してください。

または、SQL Server の [SQL Server マネージド バックアップ](/sql/relational-databases/backup-restore/sql-server-managed-backup-to-microsoft-azure?preserve-view=true&view=sql-server-ver15)機能を使用して、Azure Blob Storage のバックアップを管理することもできます。

オフサイトのアーカイブに適しているこのオプションの使用の詳細については、以下を参照してください。 

- [チュートリアル:Azure Blob Storage サービスと SQL Server 2016 の使用](/sql/relational-databases/tutorial-use-azure-blob-storage-service-with-sql-server-2016?preserve-view=true&view=sql-server-ver15)
- [クイック スタート:Azure Blob Storage サービスへの SQL のバックアップと復元](/sql/relational-databases/tutorial-sql-server-backup-and-restore-to-azure-blob-storage-service?preserve-view=true&tabs=SSMS&view=sql-server-ver15)

これらのバックアップ シナリオに加えて、[Azure Data Factory](/azure/machine-learning/team-data-science-process/move-sql-azure-adf) や [Integration Services (SSIS) 用の Azure Feature Pack](/sql/integration-services/azure-feature-pack-for-integration-services-ssis?preserve-view=true&view=sql-server-ver15) など、SQL Server が提供するその他のデータベース サービスをセットアップすることもできます。

## <a name="next-steps"></a>次のステップ
SQL Server で使用する方法の詳細については、以下を参照してください。
- [チュートリアル:データベース エンジンの概要](/sql/relational-databases/tutorial-getting-started-with-the-database-engine?preserve-view=true&view=sql-server-ver15)