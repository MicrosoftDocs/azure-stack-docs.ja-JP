---
title: Azure Stack HCI ソリューションの概要
description: Azure Stack HCI は、オンプレミスのハイブリッド環境で仮想化された Windows および Linux のワークロードとそのストレージをホストするハイパーコンバージド インフラストラクチャ (HCI) クラスター ソリューションです。 Azure のハイブリッド サービスでは、クラウドベースの監視、Site Recovery、VM バックアップなどの機能、および Azure portal 内のすべての Azure Stack HCI デプロイの一元的なビューにより、クラスターが強化されます。
ms.topic: overview
author: khdownie
ms.author: v-kedow
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 02/10/2021
ms.openlocfilehash: 66156b01796c483f509b5b0787ca947f85fa6a1a
ms.sourcegitcommit: 5ea0e915f24c8bcddbcaf8268e3c963aa8877c9d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2021
ms.locfileid: "100487869"
---
# <a name="azure-stack-hci-solution-overview"></a>Azure Stack HCI ソリューションの概要

> 適用対象:Azure Stack HCI バージョン 20H2

Azure Stack HCI は、オンプレミスのハイブリッド環境で仮想化された Windows および Linux のワークロードとそのストレージをホストするハイパーコンバージド インフラストラクチャ (HCI) クラスター ソリューションです。 Azure のハイブリッド サービスでは、クラウドベースの監視、Site Recovery、VM バックアップなどの機能、および Azure portal 内のすべての Azure Stack HCI デプロイの一元的なビューにより、クラスターが強化されます。 Windows Admin Center や PowerShell などの既存のツールを使用して、クラスターを管理できます。

Azure Stack HCI では、バージョン 20H2 を[ダウンロードできるようになりました](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)。 その対象は、ハイブリッド クラウド接続が組み込まれ、仮想化されたワークロードが実行されている、オンプレミスのクラスターです。 そのため、Azure Stack HCI は Azure サービスとして提供され、Azure サブスクリプションで課金されます。 Azure Stack HCI には、Azure Kubernetes サービスをホストする機能も含まれるようになりました。詳細については、「[Azure Stack HCI の Azure Kubernetes Service](../aks-hci/overview.md)」を参照してください。

Azure Stack HCI の高レベルの機能については、数分時間を取ってビデオをご覧ください。

> [!VIDEO https://www.youtube.com/embed/fw8RVqo9dcs]

その中核である Azure Stack HCI は、次のものが組み合わされたソリューションです。

- OEM パートナーからの検証済みハードウェア
- Azure Stack HCI オペレーティング システム
- Windows Admin Center
- Azure サービス

:::image type="content" source="media/overview/azure-stack-hci-solution.png" alt-text="Azure Stack HCI OS は、検証済みハードウェア上で実行され、Windows Admin Center によって管理され、Azure に接続される" border="false":::

Azure Stack HCI バージョン 20H2 では、ストレージ コスト パフォーマンスが優れた記憶域スペース ダイレクトを使用するハイパーコンバージド クラスターを Windows Admin Center で作成する機能など、Windows Server には存在しない新機能が提供されています。 これには、サイト間の自動フェールオーバーのために、サイトをまたいでクラスターをストレッチするオプションが含まれます。 詳細については、「[Azure Stack HCI の新機能](#whats-new-in-azure-stack-hci)」を参照してください。

## <a name="use-cases-for-azure-stack-hci"></a>Azure Stack HCI のユース ケース

Azure Stack HCI には多くのユース ケースがありますが、仮想化されていないワークロードのためのものではありません。 多くの場合、お客様は次のようなシナリオで Azure Stack HCI を選択します。

### <a name="data-center-consolidation-and-modernization"></a>データ センターの統合と近代化

Azure Stack HCI を使用して老朽化した仮想化ホストを更新して統合すると、スケーラビリティが向上し、環境の管理とセキュリティ保護が容易になります。 また、レガシ SAN ストレージを退役させて、占有領域と総保有コストを削減するのにもよい機会です。 運用とシステムの管理は、統合されたツールとインターフェイス、および単一のサポートにより簡素化されます。

### <a name="remote-and-branch-offices"></a>リモート オフィスとブランチ オフィス

Azure Stack HCI では、1 箇所あたり 2 万ドル未満で 2 サーバーのクラスター ソリューションを始めることができ、リモート オフィスとブランチ オフィス、小売店、現場サイトの最新化を低コストで行うのに最適です。 革新的な入れ子になった回復性を使用すると、複数のハードウェアで同時に障害が発生した場合でも、ボリュームをオンラインでアクセス可能な状態に維持できます。 クラウド監視テクノロジを使用すると、クラスター クォーラムに対する軽量の決定機構として Azure を使用でき、3 番目のホストにコストをかけることなくスプリット ブレイン状況を防止できます。 また、お客様は、リモート環境の Azure Stack HCI のデプロイを Azure portal で一元的に見ることもできます。

### <a name="virtual-desktops"></a>仮想デスクトップ

多くの組織は、低待機時間とデータ主権のため、仮想デスクトップをオンプレミスでホストすることを望んでいます。 Azure Stack HCI では、ローカル環境と同様のパフォーマンスを提供できます。

### <a name="high-performance-virtualized-workloads"></a>高パフォーマンスの仮想化ワークロード

Azure Stack HCI では、百万単位のストレージ IOPS や 1 秒あたりデータベース トランザクション数を必要とする SQL Server データベースや他のパフォーマンス重視の仮想化ワークロードに対して、業界最高レベルのパフォーマンスを提供することができ、NVMe SSD、堅牢な RDMA スタック、および永続メモリにより一貫した低待機時間が提供されます。

## <a name="azure-integration-benefits"></a>Azure 統合の利点

ハイブリッド インフラストラクチャに関して Azure Stack HCI と同じような製品は他にはなく、クラウドとオンプレミスのリソースを連携させて利用し、監視、セキュリティ保護、クラウドに対するバックアップをネイティブに行うことができます。

Azure Stack HCI クラスターを Azure に登録した後は、最初に Azure portal を使用して以下のことができます。

- **監視:** すべての Azure Stack HCI クラスターを 1 つのグローバルなビューで表示し、リソース グループ別にグループ化したり、タグを付けたりします。
- **課金**: Azure サブスクリプションを通じて Azure Stack HCI の料金を支払います。

追加機能の作成に取り組んでいますので、注目していてください。

また、追加の Azure ハイブリッド サービスをサブスクライブすることもできます。

- **Azure Site Recovery**: 高可用性およびサービスとしてのディザスター リカバリー (DRaaS)。
- **Azure Monitor**: AI による高度な分析を使用して、お客様のアプリ、ネットワーク、インフラストラクチャ全体で起こっていることを追跡するための集中ハブ。
- **クラウド監視**: Azure をクラスター クォーラム用の軽量な決定機構として使用。
- **Azure Backup**: オフサイト データ保護およびランサムウェアに対する保護を提供。
- **Azure Update Management**: Azure およびオンプレミスで実行されている Windows VM に対する更新プログラムの評価と更新プログラムのデプロイ。
- **Azure Network Adapter**: ポイント対サイト VPN 経由で、オンプレミスのリソースを Azure 内のお客様の VM に接続。
- **Azure File Sync**: ファイル サーバーとクラウドを同期。

詳細については、「[Azure ハイブリッド サービスへの Windows Server の接続](/windows-server/manage/windows-admin-center/azure/index)」を参照してください。

## <a name="why-azure-stack-hci"></a>Azure Stack HCI にする理由

Azure Stack HCI は、既に大規模にデプロイされている実証済みのテクノロジ (Hyper-V、記憶域スペース ダイレクト、Azure によって触発されたソフトウェアによるネットワーク (SDN) など) に基づいて構築された、世界クラスの統合された仮想化スタックです。 お客様は以下のようなさまざまな理由で Azure Stack HCI を選択します。

- Hyper-V やサーバーの管理者にとっては使い慣れたものであり、既存の仮想化およびストレージの概念とスキルを活用できます。
- Microsoft System Center、Active Directory、グループ ポリシー、PowerShell スクリプトなど、データ センターの既存のプロセスとツールで動作します。
- バックアップ、セキュリティ、監視に関する一般的なサードパーティ製ツールで動作します。
- ハードウェアを柔軟に選択できるので、お客様は、最寄りの地域でサービスとサポートが最も優れたベンダーを選択できます。
- Microsoft とハードウェア ベンダーの共同サポートにより、カスタマー エクスペリエンスが向上します。
- シームレスで完全なスタックの更新により、最新の状態を簡単に維持できます。
- 柔軟で広範なエコシステムにより、IT プロフェッショナルはニーズに最適なソリューションを柔軟に構築できます。

## <a name="what-you-need-for-azure-stack-hci"></a>Azure Stack HCI に必要なもの

使い始めるには、次のものが必要です。

- 好みの Microsoft ハードウェア パートナーから購入した、[Azure Stack HCI カタログ](https://hcicatalog.azurewebsites.net)に掲載されている 2 台以上のサーバーのクラスター
- [Azure サブスクリプション](https://azure.microsoft.com/)
- クラスター内の各サーバーのインターネット接続。少なくとも 30 日ごとに HTTPS 送信トラフィック経由で既知の Azure エンドポイントに接続できること
- 複数のサイトに拡張されるクラスターの場合は、サイト間に 1 Gb 以上の接続が必要であり (25 Gb の RDMA 接続を推奨)、両方のサイトで書き込みが同時に発生する同期レプリケーションを行う場合は、平均待機時間が 5 ミリ秒のラウンドトリップが必要です
- ソフトウェアによるネットワーク制御 (SDN) を使用する計画の場合は、ネットワーク コントローラー VM を作成するために Azure Stack HCI オペレーティング システム用の仮想ハード ディスク (VHD) が必要です (「[ネットワーク コントローラーのデプロイを計画する](concepts/network-controller.md)」をご覧ください)

詳細については、[システム要件](concepts/system-requirements.md)に関するページを参照してください。 Azure Stack HCI での Azure Kubernetes Service の要件については、[Azure Stack HCI での AKS の要件](../aks-hci/overview.md#what-you-need-to-get-started)に関するページを参照してください。

## <a name="availability"></a>可用性

|側面|詳細|
|----|:----|
|リリース状態:|一般提供 (GA)|
|価格:|[Azure サブスクリプションに対して課金](concepts/billing.md)|
|必要なロールとアクセス許可:|**Azure Active Directory 管理者** または委任されたアクセス許可|
|||

## <a name="hardware-partners"></a>ハードウェア パートナー

好みの Microsoft パートナーから検証済みの Azure Stack HCI ソリューションを購入し、時間のかかる設計とビルド時間なしで稼働させることができます。 また、Microsoft パートナーは、単一窓口で実装およびサポート サービスの問い合わせに対応します。 検証済みのノードまたは統合システムを購入できます。これには、プレインストールされている Azure Stack HCI オペレーティング システムと共に、ドライバーとファームウェアの更新プログラムに対するパートナー拡張機能も含まれます。

[Azure Stack HCI ソリューション](https://azure.microsoft.com/overview/azure-stack/hci)のページにアクセスするか、[Azure Stack HCI カタログ](https://hcicatalog.azurewebsites.net)を参照してください。現在、ASUS、Axellio、Blue Chip、DataON、Dell EMC、富士通、HPE、日立、Huawei、Lenovo、NEC、primeLine Solutions、QCT、SecureGUARD、Supermicro などの Microsoft パートナーの、70 以上の Azure Stack HCI ソリューションを利用できます。

## <a name="software-partners"></a>ソフトウェア パートナー

さまざまな Microsoft パートナーが、IT 管理者が使い慣れたツールを使用できるように、Azure Stack HCI の機能を拡張するソフトウェアの作業を行っています。 詳細については、「[Azure Stack HCI のユーティリティ アプリケーション](concepts/utility-applications.md)」を参照してください。

## <a name="licensing-billing-and-pricing"></a>ライセンス、請求、価格

Azure Stack HCI の請求は、永続的ライセンスではなく、物理プロセッサ コアあたりの月額サブスクリプション料金に基づきます。 お客様が Azure に接続すると、使用されているコアの数が自動的にアップロードされ、請求のために評価されます。 コストは、物理プロセッサ コア以外の消費量では変わりません。つまり、VM の数が増えてもコストが高くなることはなく、より高密度の仮想環境を実行できるお客様ほど得をします。

お客様は、Azure Stack HCI オペレーティング システムがプレインストールされている検証済みサーバーをハードウェア パートナーから購入できます。または、OEM から検証済みベア メタル サーバーを購入した後、Azure Stack HCI サービスをサブスクライブし、[Azure Stack HCI オペレーティング システムをダウンロードする](https://azure.microsoft.com/products/azure-stack/hci/)ことができます。

## <a name="the-azure-stack-family"></a>Azure Stack ファミリ

Azure Stack HCI は、Azure と Azure Stack ファミリの一員であり、Azure Stack Hub と同じ、ソフトウェアによるコンピューティング、ストレージ、およびネットワーク ソフトウェアを利用します。 さまざまなソリューションの簡単な概要を次に示します。 詳細については、[Azure Stack エコシステムの比較](../operator/compare-azure-azure-stack.md)に関するページを参照してください。

- [Azure](https://azure.microsoft.com) - オンデマンドのセルフサービス コンピューティング リソースのためのパブリック クラウド サービスを利用して、既存のアプリを移行して最新化し、新しいクラウドネイティブ アプリを構築します。
- [Azure Stack Edge](/azure/databox-online/data-box-edge-overview) - 機械学習ワークロードを加速し、オンプレミスのコンテナー化されたアプリや仮想化されたワークロードをクラウド管理アプライアンスで実行します。
- [Azure Stack HCI](https://azure.microsoft.com/overview/azure-stack/hci) - 仮想化されたアプリをオンプレミスで実行し、古くなったサーバー インフラストラクチャを交換および整理し、クラウド サービスを利用するために Azure に接続します。
- [Azure Stack Hub](../operator/azure-stack-overview.md) - 接続していない場合や、規制要件を満たすために、一貫性のある Azure サービスを使用し、クラウド アプリをオンプレミスで実行します。

:::image type="content" source="media/overview/azure-family-updated.png" alt-text="Azure Stack ファミリのソリューションの図" border="false":::

## <a name="whats-new-in-azure-stack-hci"></a>Azure Stack HCI の新機能

Windows Admin Center バージョン 2009 では、次のような多数の機能が Azure Stack HCI に追加されています。

- **Azure Kubernetes Service のホスティング機能**:[Azure Stack HCI 上の Azure Kubernetes Service](https://azure.microsoft.com/products/azure-stack/hci/hci-download/) のプレビュー バージョンをインストールできるようになりました。

- **クラスター作成ウィザードでのリモート ダイレクト メモリ アクセス (RDMA) の機能強化**: クラスター作成ウィザードで、データ センター ブリッジング (DCB) を含む iWARP および RoCE ネットワーク アダプターの RDMA を構成できるようになりました。

新機能の詳細については、「[Windows Admin Center のクラスター作成拡張機能の一般提供に関する告知](https://techcommunity.microsoft.com/t5/windows-admin-center-blog/announcing-general-availability-of-the-cluster-creation/ba-p/1978332)」を参照してください。

Windows Server 2019 ベースのソリューションと比較して、Azure Stack HCI バージョン 20H2 を実行するクラスターには、次の新機能があります。

- **Windows Admin Center の新機能**: 直感的な UI を使用してハイパーコンバージド クラスターを作成および更新する機能により、Azure Stack HCI はこれまで以上に簡単に使用できるようになりました。
- **自動フェールオーバーのためのストレッチ クラスター**: 記憶域レプリカ レプリケーションと自動 VM フェールオーバーを使用するマルチサイト クラスタリングによって、ネイティブのディザスター リカバリーとビジネス継続性が提供されます。
- **アフィニティと非アフィニティのルール**: これらは、ストレッチ クラスターなどの複数の障害ドメインを持つクラスターで、VM とストレージをまとめて、または分離して維持するために Azure で Availability Zones が使用される方法と同様に使用できます。
- **Azure portal の統合**: Azure Stack HCI に対する Azure portal のエクスペリエンスは、開発中の新機能を使用して、世界中のすべての Azure Stack HCI クラスターを表示するように設計されています。
- **高パフォーマンス ワークロード用の GPU アクセラレーション**: AI/ML アプリケーションでは、GPU を使用したパフォーマンスの向上によってメリットがあります。
- **BitLocker 暗号化**: BitLocker を使用して Azure Stack HCI 上のデータ ボリュームの内容を暗号化できるようになりました。これにより、政府や他のお客様は FIPS 140-2 や HIPAA などの標準に準拠した状態を維持できます。
- **記憶域スペース ダイレクト ボリュームの修復速度の向上**: すばやくシームレスにボリュームを修復します。

Azure Stack HCI 20H2 の新機能の詳細については、Microsoft Inspire の[こちらの動画をご覧ください](https://www.youtube.com/watch?v=DPG7wGhh3sAa)。

## <a name="roles-you-can-run-without-virtualizing"></a>仮想化なしで実行できる役割

Azure Stack HCI は、仮想マシンですべてのワークロードを実行する仮想化ホストとして意図されているため、Azure Stack HCI の条件では、仮想マシンをホストするために必要なものしか実行できません。

これは、次のサーバーの役割を実行できることを意味します。

- Hyper-V
- ソフトウェアによるネットワーク (SDN) に必要なネットワーク コントローラーおよびその他のコンポーネント

一方、その他の役割とアプリは VM 内で実行する必要があります。 ホストされている VM の管理と正常性に必要なユーティリティ、アプリ、サービスを実行できることに注意してください。

## <a name="video-based-learning"></a>ビデオでの学習

Azure 拡張ネットワーク デオは次のとおりです。

- [Windows Server とハイブリッド ットワークを使用した Azure へのシームレスな接続](https://www.youtube.com/watch?v=do2_4Y2p9dk)

Azure Stack HCI の元の Windows Server 2019 ベースのバージョンに関するビデオ:

- [Azure Stack および Azure Stack HCI の概要ビュー](https://aka.ms/AzureStackOverviewVideo)
- [Microsoft Ignite Live 2019 - Azure Stack HCI の概要](https://www.youtube.com/watch?v=vueHIBqNIEU)
- [Azure Stack HCI の探索](https://www.youtube.com/watch?v=4aGZK0Ndmh8&list=PLQXpv_NQsPICdXZoH-EzlIFa4P6VS5m11&index=13&t=0s)
- [Azure Stack HCI による小売店やブランチ オフィスの最新化](https://www.youtube.com/watch?v=-JzLhjfkhmM&list=PLQXpv_NQsPICdXZoH-EzlIFa4P6VS5m11&index=9&t=0s)
- [Azure Stack HCI の新機能: 45 分で 45 の機能をご紹介](https://www.youtube.com/watch?v=C5J4IEnlS_E&list=PLQXpv_NQsPICdXZoH-EzlIFa4P6VS5m11&index=12&t=0s)
- [Azure Stack HCI のデプロイをすぐに開始する](https://www.youtube.com/watch?v=gxaPJLrWy5w&list=PLQXpv_NQsPICdXZoH-EzlIFa4P6VS5m11&index=11&t=0s)
- [データ圧縮のケース: Azure Stack HCI でのデータ重複除去](https://www.youtube.com/watch?v=fmm4iDbDiY4&list=PLQXpv_NQsPICdXZoH-EzlIFa4P6VS5m11&index=23&t=0s)
- [Azure Stack HCI のフィールドからの Dave Kawula のメモ](https://www.youtube.com/watch?v=OXv7fLlz0ew&list=PLQXpv_NQsPICdXZoH-EzlIFa4P6VS5m11&index=2&t=0s)

ハイブリッド クラウド仮想イベントのビデオをご紹介します。

- [Azure Stack HCI | ハイブリッド クラウド仮想イベント](https://www.youtube.com/watch?v=nxpoEva-R2Y)

## <a name="next-steps"></a>次のステップ

- [Azure Stack HCI をダウンロードする](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)
- [Azure Stack HCI クラスターを作成して Azure に登録する](deploy/deployment-quickstart.md)
- [Windows Admin Center で Azure Stack HCI を使用する](get-started.md)
