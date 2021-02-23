---
title: Azure Stack Hub のリリース ノート
description: 更新プログラムやバグ修正プログラムを含む、Azure Stack Hub 統合システムのリリース ノート。
author: sethmanheim
ms.topic: article
ms.date: 02/17/2021
ms.author: sethm
ms.reviewer: sranthar
ms.lastreviewed: 09/09/2020
ms.openlocfilehash: 441e764231e9ce85de69d7cd4020325883fde7ec
ms.sourcegitcommit: 4c97ed2caf054ebeefa94da1f07cfb6be5929aac
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/18/2021
ms.locfileid: "100648084"
---
# <a name="azure-stack-hub-release-notes"></a>Azure Stack Hub のリリース ノート

この記事では、Azure Stack Hub 更新プログラム パッケージの内容について説明します。 更新プログラムには、Azure Stack Hub の最新のリリースに対する機能強化と修正が含まれています。

別のバージョンのリリース ノートにアクセスするには、左側の目次の上部にあるバージョン セレクターのドロップダウンを使用します。

::: moniker range=">=azs-2002"
> [!IMPORTANT]  
> この更新プログラム パッケージは、Azure Stack Hub 統合システム専用です。 Azure Stack Development Kit (ASDK) にこの更新プログラム パッケージを適用しないでください。
::: moniker-end
::: moniker range="<azs-2002"
> [!IMPORTANT]  
> お使いの Azure Stack Hub インスタンスが 2 つ前の更新プログラムより古い場合、コンプライアンスに対応していないとみなされます。 [サポートを受けるためには、少なくともサポートされる最小バージョンまで更新する](azure-stack-servicing-policy.md#keep-your-system-under-support)必要があります。
::: moniker-end

## <a name="update-planning"></a>計画の更新

更新プログラムを適用する前に、必ず次の情報を確認してください。

- [更新プログラム適用前後のアクティビティのチェックリスト](release-notes-checklist.md)
- [既知の問題](known-issues.md)
- [修正プログラム](#hotfixes)
- [セキュリティ更新プログラム](release-notes-security-updates.md)

更新プログラムのトラブルシューティングと更新プロセスの詳細については、[Azure Stack Hub の修正プログラムと更新プログラムについての問題のトラブルシューティング](azure-stack-troubleshooting.md)に関するページを参照してください。

## <a name="download-the-update"></a>更新プログラムをダウンロードする

Azure Stack Hub 更新プログラム パッケージは、[Azure Stack Hub 更新プログラム ダウンローダー ツール](https://aka.ms/azurestackupdatedownload)を使用してダウンロードできます。

<!---------------------------------------------------------->
<!------------------- SUPPORTED VERSIONS ------------------->
<!---------------------------------------------------------->
::: moniker range="azs-2008"
## <a name="2008-build-reference"></a>2008 ビルドのリファレンス

Azure Stack Hub 2008 更新プログラムのビルド番号は **1.2008.13.88** です。

### <a name="update-type"></a>更新の種類

Azure Stack Hub 2008 更新プログラムのビルドの種類は **完全** です。

2008 更新プログラム パッケージは、以前の更新プログラムと比較してサイズが大きくなっています。 サイズが大きいため、ダウンロード時間が長くなります。 この更新プログラムは **準備中** 状態の時間が長くなります。オペレーターは以前の更新プログラムよりもこのプロセスの時間が長くなることを想定してください。 内部テストでは、2008 更新プログラムの実行時間が次のように予測されました。4 ノード: 13 から 20 時間、8 ノード: 16 から 26 時間、12 ノード: 19 から 32 時間、16 ノード: 22 から 38 時間。 更新プログラムの正確なランタイムは一般的に、ご使用のシステムでテナント ワークロードによって使用されている容量、システム ネットワーク接続 (インターネットに接続されている場合)、およびシステム ハードウェアの仕様に左右されます。 実行時間がこの予測値よりも短くなったり長くなったりすることは一般的ではなく、更新が失敗した場合を除き、Azure Stack Hub オペレーターによるアクションは不要です。 この実行時間の概算は 2008 更新プログラムに固有であるため、他の Azure Stack Hub 更新プログラムと比較しないでください。

更新プログラムのビルドの種類については、「[Azure Stack Hub での更新プログラム管理の概要](azure-stack-updates.md)」を参照してください。

<!-- ## What's in this update -->

<!-- The current theme (if any) of this release. -->

### <a name="whats-new"></a>新機能

<!-- What's new, also net new experiences and features. -->
- Azure Stack Hub で VNET ピアリングがサポートされるようになりました。これにより、ネットワーク仮想アプライアンス (NVA) を使用せずに VNET に接続できるようになります。 詳細については、[新しい VNET ピアリングのドキュメント](../user/virtual-network-peering.md)を参照してください。
- Azure Stack Hub の Blob Storage で、ユーザーが不変 BLOB を使用できるようになりました。 コンテナーで不変ポリシーを設定することにより、ビジネス クリティカルなデータ オブジェクトを WORM (Write Once, Read Many) 状態で格納することができます。 このリリースでは、不変ポリシーは REST API またはクライアント SDK によってのみ設定できます。 このリリースでは、追加 BLOB の書き込みもできません。 不変 BLOB の詳細については、「[不変ストレージを使用してビジネスに不可欠な BLOB データを保存する](/azure/storage/blobs/storage-blob-immutable-storage)」を参照してください。
- Azure Stack Hub ストレージで、Azure Storage サービスの API バージョン **2019-07-07** がサポートされるようになりました。 新しい REST API のバージョンと互換性がある Azure クライアント ライブラリについては、[Azure Stack Hub ストレージの開発ツール](../user/azure-stack-storage-dev.md#azure-client-libraries)に関するページを参照してください。 Azure Storage サービスの管理 API の場合、利用可能な全機能のサブセットが含まれる **2018-02-01** がサポートに追加されています。
- Azure Stack Hub コンピューティングで、Azure Compute API バージョン **2020-06-01** がサポートされるようになりました。これには、利用可能な全機能のサブセットが含まれています。
- Azure Stack Hub マネージド ディスクで、Azure Disk API バージョン **2019-03-01** がサポートされるようになりました。これには、利用可能な機能のサブセットが含まれます。
- サポート操作中に Azure Stack Hub に接続してインフラストラクチャに関する詳細な分析情報を提供できるようになった、Windows Admin Center のプレビュー (緊急用アカウントが必要)。
- デプロイ時に特権エンドポイント (PEP) にログイン バナーを追加する機能。
- より多くの **排他的操作** バナーがリリースされました。これにより、システムで現在行われている操作の可視性が向上し、ユーザーは他の排他的な操作を開始することが (そして後で失敗することが) なくなります。
- Azure Stack Hub Marketplace の各項目の製品ページに、2 つの新しいバナーが導入されました。 Marketplace のダウンロードに失敗した場合、オペレーターはエラーの詳細を表示し、問題を解決するために推奨される手順を試行できます。
- お客様がフィードバックを提供するための評価ツールがリリースされました。 これにより、Azure Stack Hub でカスタマー エクスペリエンスを測定して最適化できます。
- このリリースの Azure Stack Hub には、Azure Kubernetes Service (AKS) と Azure Container Registry (ACR) のプライベート プレビューが含まれています。 このプライベート プレビューの目的は、Azure Stack Hub の AKS と ACR の品質、機能、ユーザー エクスペリエンスに関するフィードバックを収集することです。
- このリリースには、[AKS Engine v 0.55.4](../user/kubernetes-aks-engine-release-notes.md) を使用する Azure CNI と Windows コンテナーのパブリック プレビューが含まれています。 API モデルでこれらを使用する方法の例については、[GitHub のこちらの例を参照](https://raw.githubusercontent.com/Azure/aks-engine/master/examples/azure-stack/kubernetes-windows.json)してください。
- [AKS Engine v 0.55.4](../user/kubernetes-aks-engine-release-notes.md) によってデプロイされたクラスターでの [Istio 1.3 デプロイ](https://github.com/Azure/aks-engine/tree/master/examples/service-mesh)がサポートされるようになりました。 詳細については、[こちらの手順を参照してください](../user/kubernetes-aks-engine-service-account.md)。
- [AKS Engine v 0.55.4](../user/kubernetes-aks-engine-release-notes.md) を使用した[プライベート クラスター](https://github.com/Azure/aks-engine/blob/master/docs/topics/features.md#private-cluster)のデプロイがサポートされるようになりました。
- このリリースでは、Azure および Azure Stack Hub Key Vault インスタンスからの [Kubernetes の構成シークレットのソーシング](https://github.com/Azure/aks-engine/blob/master/docs/topics/keyvault-secrets.md#use-key-vault-as-the-source-of-cluster-configuration-secrets)がサポートされています。

### <a name="improvements"></a>機能強化

<!-- Changes and product improvements with tangible customer-facing value. -->
- ネットワーク コントローラーと SLB ホスト エージェントの内部監視が実装されたため、停止状態になったサービスは自動的に修復されます。
- お客様が独自の Active Directory フェデレーション サービス (AD FS) サーバーで新しいトークン署名証明書をローテーションした後、AD FS によってそれが取得されるようになりました。 既に構成されているシステムでこの新機能を利用するには、AD FS 統合を再度構成する必要があります。 詳細については、「[AD FS ID を Azure Stack Hub データセンターに統合する](azure-stack-integrate-identity.md)」を参照してください。
- インフラストラクチャ ロール インスタンスおよびスケール ユニット ノードでのそれらの依存関係における、スタートアップ プロセスとシャットダウン プロセスの変更。 これにより、Azure Stack Hub の起動とシャットダウンの信頼性が向上します。
- **Test-AzureStack** 検証ツールの **AzSScenarios** スイートが、すべての顧客アカウントに多要素認証が適用されている場合にクラウド サービス プロバイダーがこのスイートを正常に実行できるように更新されました。
- ライフサイクル操作中の 29 の顧客向けアラートの抑制ロジックを追加することで、アラートの信頼性が向上しました。
- ログ コレクションの役割、期間、およびステータスの詳細を提供する詳細なログ コレクション HTML レポートを表示できるようになりました。 このレポートは、収集したログの概要をユーザーがまとめやすくなるよう提供されています。 その後、Microsoft カスタマー サポート サービスがレポートを迅速に査定してログ データを評価し、システムの問題をトラブルシューティングして軽減するサポートを行います。
- CPU 使用率やメモリ消費量などのユーザー シナリオにまたがる、障害検知の信頼性向上に貢献する 7 つのモニターが新たに追加されたことで、インフラストラクチャの障害検知範囲が拡張されました。

### <a name="changes"></a>[変更点]

- SRP API バージョン **2016-01-01** および **2016-05-01** で **supportHttpsTrafficOnly** ストレージ アカウント リソースの種類プロパティが有効にされましたが、このプロパティは Azure Stack Hub ではサポートされていません。
- ボリューム容量使用率アラートのしきい値が、80% (警告) と 90% (重大) から、90% (警告) と 95% (重大) に引き上げられました。 詳細については、「[記憶域スペースのアラート](azure-stack-manage-storage-shares.md#storage-space-alerts)」を参照してください
- AD Graph の構成手順が、このリリースで変更されます。 詳細については、「[AD FS ID を Azure Stack Hub データセンターに統合する](azure-stack-integrate-identity.md)」を参照してください。
- Windows Server 2019 用に定義されている現在のベストプラクティスに合わせるために、Azure Stack Hub は、追加のトラフィック クラスまたは優先順位を使用して、フェールオーバー クラスタリング制御通信をサポートするサーバー間通信をさらに分離するように変更されています。 これらの変更により、フェールオーバー クラスターの通信の回復性が向上します。 このトラフィック クラスと帯域幅の予約の構成は、Azure Stack Hub ソリューションの Top-of-Rack (ToR) スイッチ上と、Azure Stack Hub のホストまたはサーバー上での変更によって実現されます。

  これらの変更は、Azure Stack Hub システムのホスト レベルで追加されていることに注意してください。 OEM に連絡して、Top-of-Rack (ToR) ネットワーク スイッチで必要な変更を行うように手配してください。 この ToR の変更は、2008 リリースに更新する前に実行することも、2008 に更新した後に実行することもできます。 詳細については、[ネットワーク統合に関するドキュメント](azure-stack-network.md)を参照してください。

- このビルドでは、GPU 対応の VM サイズ **NCas_v4 (NVIDIA T4)** が、Azure に合わせて VM サイズ **NCasT4_v3** に置き換えられました。 それらはポータルにはまだ表示されておらず、Azure Resource Manager テンプレートでのみ使用できることに注意してください。

### <a name="fixes"></a>修正

<!-- Product fixes that came up from customer deployments worth highlighting, especially if there is an SR/ICM associated to it. -->
- 実行中の VM にアタッチされていない NIC の NSG の削除が失敗する問題を修正しました。
- ロード バランサーに関連付けられているパブリック IP の **IdleTimeoutInMinutes** の値を変更すると、パブリック IP がエラー状態になるという問題を修正しました。
- アタッチされたマネージド ディスクに対して **OnlineMigration** ではなく、正しく **Attached** 状態が返されるように、**Get-AzsDisk** コマンドレットを修正しました。

## <a name="security-updates"></a>セキュリティ更新プログラム

Azure Stack Hub のこの更新でのセキュリティ更新プログラムについては、「[Azure Stack Hub のセキュリティ更新プログラム](release-notes-security-updates.md)」を参照してください。

## <a name="hotfixes"></a>修正プログラム

Azure Stack Hub では、修正プログラムが定期的にリリースされます。 2008 に更新する前に、最新の 2005 修正プログラムをインストールしてください。 また、2005 リリース以降、新しいメジャー バージョンに更新すると (たとえば、1.2005.x から 1.2008.x)、その新しいメジャー バージョン内の最新の修正プログラム (パッケージのダウンロード時に使用可能なものがある場合) が自動的にインストールされます。 これにより 2008 インストールにすべての修正プログラムが適用され、最新の状態になります。 それ以降は、2008 の修正プログラムがリリースされたら、それをインストールする必要があります。

> [!NOTE]
> Azure Stack Hub 修正プログラムのリリースは累積的です。そのバージョンに対する以前の修正プログラムのリリースに含まれるすべての修正を取得するには、最新の修正プログラムをインストールするだけで済みます。

詳細については、[サービス ポリシー](azure-stack-servicing-policy.md)に関する記事を参照してください。

Azure Stack Hub 修正プログラムを適用できるのは Azure Stack Hub 統合システムのみです。ASDK には修正プログラムをインストールしないでください。

> [!TIP]
> 各修正プログラムのリリースについて通知を受け取る場合は、[**RSS フィード**](https://azurestackhubdocs.azurewebsites.net/xml/hotfixes.rss)をサブスクライブして、各修正プログラムのリリースについての通知を受け取ります。

### <a name="after-successfully-applying-the-2008-update"></a>2008 更新プログラムが正常に適用された後

Azure Stack Hub の修正プログラムは累積されるため、ベスト プラクティスとして、お使いのビルド用にリリースされたすべての修正プログラムをインストールして、メジャー リリース間の更新エクスペリエンスを最適にする必要があります。 新しいメジャー バージョンに更新すると (たとえば、1.2005.x から 1.2008.x)、その新しいメジャー バージョン内の最新の修正プログラム (パッケージのダウンロード時に使用可能なものがある場合) が自動的にインストールされます。

2008 のインストール後に、2008 修正プログラムがリリースされた場合は、それらをインストールする必要があります。

- [Azure Stack Hub 修正プログラム 1.2008.25.114](hotfix-1-2008-25-114.md)
::: moniker-end

::: moniker range="azs-2005"
## <a name="2005-build-reference"></a>2005 ビルドのリファレンス

Azure Stack Hub 2005 更新プログラムのビルド番号は **1.2005.6.53** です。

### <a name="update-type"></a>更新の種類

Azure Stack Hub 2005 更新プログラムのビルドの種類は **完全** です。

2005 更新プログラム パッケージは、以前の更新プログラムと比較してサイズが大きくなっています。 サイズが大きいため、ダウンロード時間が長くなります。 この更新プログラムは **準備中** 状態の時間が長くなります。オペレーターは以前の更新プログラムよりもこのプロセスの時間が長くなることを想定してください。 内部テストでは、2005 更新プログラムの実行時間が 4 ノードの場合は 13 から 20 時間、8 ノードの場合は 16 から 26 時間、12 ノードの場合は 19 から 32 時間、16 ノードの場合は 22 から 38 時間と予測されました。 更新プログラムの正確なランタイムは一般的に、ご使用のシステムでテナント ワークロードによって使用されている容量、システム ネットワーク接続 (インターネットに接続されている場合)、およびシステム ハードウェアの仕様に左右されます。 実行時間がこの予測値よりも短くなったり長くなったりすることは一般的ではなく、更新が失敗した場合を除き、Azure Stack Hub オペレーターによるアクションは不要です。 この実行時間の概算は 2005 更新プログラムに固有であるため、他の Azure Stack Hub 更新プログラムと比較しないでください。

更新プログラムのビルドの種類については、「[Azure Stack Hub での更新プログラム管理の概要](azure-stack-updates.md)」を参照してください。

<!-- ## What's in this update -->

<!-- The current theme (if any) of this release. -->

### <a name="whats-new"></a>新機能

<!-- What's new, also net new experiences and features. -->
- このビルドでは、3 つの新しい GPU VM の種類がサポートされています。NCv3 (Nvidia V100)、NVv4 (AMD MI25)、NCas_v4 (NVIDIA T4) の各 VM サイズです。 適切なハードウェアを使用し、Azure Stack Hub GPU プレビュー プログラムにオンボードされていれば、VM のデプロイは成功します。 関心がある場合は、 https://aka.ms/azurestackhubgpupreview で GPU プレビュー プログラムにサインアップしてください。 詳細については、[こちら](../user/gpu-vms-about.md)を参照してください。
- このリリースには、エラーを検出し、影響を評価して、システムの問題を安全に軽減する自律復旧機能を有効にする新機能が用意されています。 この機能により、Microsoft は手動による介入なしでシステムの可用性を向上させることを目指しています。 リリース 2005 以降では、アラートの数が減少します。 このパイプラインでエラーが発生しても、通知されない限り、Azure Stack Hub オペレーターによるアクションは不要です。
- Azure Stack Hub 管理ポータルに、エアギャップまたは切断された Azure Stack Hub のお客様がログをローカルに保存するための新しいオプションが追加されました。 Azure Stack Hub が Azure から切断されている場合、ログをローカル SMB 共有に保存できます。
- システム操作が既に進行中の場合に Azure Stack Hub 管理ポータルで特定の操作がブロックされるようになりました。 たとえば、更新が進行中の場合、新しいスケール ユニット ノードを追加することはできません。
- このリリースでは、1910 より前に作成された VM で、Azure とのファブリックの一貫性が向上します。 1910 で、Microsoft は、新しく作成されたすべての VM で wireserver プロトコルが使用されることを発表しました。これにより、お客様は Azure と同じ WALA エージェントおよび Windows ゲスト エージェントを使用できるようになり、Azure Stack Hub でより簡単に Azure イメージを使用できるようになりました。 今回のリリースでは、1910 より前に作成されたすべての VM が、wireserver プロトコルを使用するよう自動的に移行されます。 これにより、より信頼性の高い VM の作成、VM 拡張機能のデプロイ、安定状態のアップタイムの向上も実現します。
- Azure Stack Hub ストレージで、Azure Storage サービスの API バージョン 2019-02-02 がサポートされるようになりました。 Azure クライアント ライブラリでは、これは新しい REST API バージョンと互換性があります。 詳細については、[Azure Stack Hub ストレージ開発ツール](../user/azure-stack-storage-dev.md#azure-client-libraries)に関する記事をご覧ください。
- Azure Stack Hub で、[CreateUiDefinition (バージョン 2)](/azure/azure-resource-manager/managed-applications/create-uidefinition-overview) の最新バージョンがサポートされるようになりました。
- バッチ処理された VM デプロイに関する新しいガイダンス。 詳細については、[こちらの記事](../operator/azure-stack-capacity-planning-compute.md)をご覧ください。
- Azure Stack Hub Marketplace CoreOS Container Linux 項目は、[まもなくサポート終了になります](https://azure.microsoft.com/updates/flatcar-in-azure/)。 詳細については、[CoreOS Container Linux からの移行](https://docs.flatcar-linux.org/os/migrate-from-container-linux/)に関するページをご覧ください。

### <a name="improvements"></a>改善

<!-- Changes and product improvements with tangible customer-facing value. -->

- ストレージ インフラストラクチャ クラスター サービスのログとイベントの機能強化。 ストレージ インフラストラクチャ クラスター サービスのログとイベントは、診断とトラブルシューティングの向上のために最大 14 日間保持されます。
- Azure Stack Hub の開始と停止の信頼性を向上させる機能強化。
- 分散を使用し、依存関係を削除することで、更新プログラムの実行時間を短縮する機能強化。 2002 更新プログラムと比較すると、4 ノードのスタンプ更新時間が、15 から 42 時間から、13 から 20 時間に短縮されます。 8 ノードでは、20 から 50 時間から、16 から 26 時間に短縮されます。 12 ノードでは、20 から 60 時間から、19 から 32 時間に短縮されます。 16 ノードでは、25 から 70 時間から、22 から 38 時間に短縮されます。 更新プログラムの正確なランタイムは一般的に、ご使用のシステムでテナント ワークロードによって使用されている容量、システム ネットワーク接続 (インターネットに接続されている場合)、およびシステム ハードウェアの仕様に左右されます。
- 特定の回復不能なエラーが発生した場合、更新が早期に失敗するようになりました。
- インターネットからのダウンロード中に更新プログラム パッケージの回復性が向上しました。
- VM の停止 - 割り当て解除中の回復性が向上しました。
- ネットワーク コントローラーのホスト エージェントの回復性が向上しました。
- 特権エンドポイントと復旧エンドポイントへの接続に使用されるソース IP とアカウントを報告するために、syslog メッセージの CEF ペイロードにフィールドが追加されました。 詳細については、「[Syslog 転送を使用して Azure Stack Hub と監視ソリューションを統合する](azure-stack-integrate-security.md)」を参照してください。
- syslog クライアントによって生成されるイベントのリストに Windows Defender イベント (イベント ID 5001、5010、5012) が追加されました。
- Defender プラットフォームとシグネチャのバージョンの不一致、および検出されたマルウェアに対するアクションの失敗を報告するために、Azure Stack 管理者ポータルに Windows Defender 関連のイベントのアラートが追加されました。
- Azure Stack Hub をデータセンターに統合するときに、4 つの境界デバイスのサポートが追加されました。

### <a name="changes"></a>[変更点]

- インフラストラクチャ ロール インスタンスを停止、シャットダウン、再起動するアクションが、管理ポータルから削除されました。 また、ファブリック リソース プロバイダーの対応する API も削除されました。 Azure Stack Hub 用の管理 RM モジュールと AZ プレビューの次の PowerShell コマンドレットは機能しなくなりました: **Stop-AzsInfrastructureRoleInstance**、**Disable-InfrastructureRoleInstance**、**Restart-InfrastructureRoleInstance**。 これらのコマンドレットは、Azure Stack Hub 用管理 AZ モジュールの次回リリースから削除される予定です。
- Azure Stack Hub 2005 では、[App Service on Azure Stack Hub 2020 (バージョン 87.x)](app-service-release-notes-2020-Q2.md) のみがサポートされるようになりました。
- セキュリティを強化するために、ハードウェア監視に必要なユーザー暗号化設定が DES から AES に変更されました。 ベースボード管理コントローラー (BMC) の設定を変更する方法については、ハードウェア パートナーにお問い合わせください。 場合によっては、BMC で変更を行った後、特権エンドポイントを使用して **Set-BmcCredential** コマンドを再実行する必要があります。 詳細については、「[Azure Stack Hub でシークレットをローテーションする](azure-stack-rotate-secrets.md)」を参照してください。

### <a name="fixes"></a>修正

<!-- Product fixes that came up from customer deployments worth highlighting, especially if there is an SR/ICM associated to it. -->

- ベース OS イメージへのパスが見つからないためスケール ユニット ノードの修復が失敗する原因となる問題を修正しました。
- スケール ユニット ノードの修復に連鎖的な影響を及ぼすサポート インフラストラクチャ ロールのスケールインとスケールアウトに関する問題を修正しました。
- オペレーターが **[すべてのサービス] > [コンピューティング] > [VM イメージ] > [追加]** で Azure Stack Hub 管理者ポータルに独自のイメージを追加するときに (.vhd ではなく) .VHD 拡張子が許可されない問題を修正しました。
- 以前の VM 再起動操作によって、他の VM 更新操作 (ディスク、タグなどの追加) 後に予期しない再起動が発生する問題を修正しました。
- 重複する DNS ゾーンを作成するとポータルが応答しなくなる問題を修正しました。 適切なエラーが表示されるようになりました。
- **Get-AzureStackLogs** で、ネットワークの問題のトラブルシューティングに必要なログが収集されていない問題を修正しました。 
- ポータルで接続が許可される NIC の数が、実際に許可されている数よりも少ないという問題を修正しました。 
- 特定の内部ソフトウェアに対して違反イベントが生成されないようにコードの整合性ポリシーを修正しました。 これにより、syslog クライアントによって生成されるコードの整合性違反イベントのノイズが減少します。
- https サービスの再起動またはホストの再起動を求めることなく新しいポリシーを適用するように **Set-TLSPolicy** コマンドレットを修正しました。
- Linux NTP サーバーを使用したときに管理ポータルで誤ってアラートが生成される問題を修正しました。  
- バックアップ コントローラー サービス インスタンスのフェールオーバーによって自動バックアップが無効になる問題を修正しました。
- インフラストラクチャ サービスがインターネットに接続されていないときに内部シークレットのローテーションが失敗する問題を修正しました。
- ユーザーが、Azure Stack Hub ポータルを使用して、サブスクリプションのアクセス許可を表示できない問題を修正しました。

## <a name="security-updates"></a>セキュリティ更新プログラム

Azure Stack Hub のこの更新でのセキュリティ更新プログラムについては、「[Azure Stack Hub のセキュリティ更新プログラム](release-notes-security-updates.md)」を参照してください。

## <a name="hotfixes"></a>修正プログラム

Azure Stack Hub では、修正プログラムが定期的にリリースされます。 2005 リリース以降では、新しいメジャー バージョンに更新すると (たとえば、1.2002.x から 1.2005.x)、その新しいメジャー バージョン内の最新の修正プログラム (存在する場合) が自動的にインストールされます。 それ以降は、ビルドの修正プログラムがリリースされたら、それをインストールする必要があります。

> [!NOTE]
> Azure Stack Hub 修正プログラムのリリースは累積的です。そのバージョンに対する以前の修正プログラムのリリースに含まれるすべての修正を取得するには、最新の修正プログラムをインストールするだけで済みます。

詳細については、[サービス ポリシー](azure-stack-servicing-policy.md)に関する記事を参照してください。

Azure Stack Hub 修正プログラムを適用できるのは Azure Stack Hub 統合システムのみです。ASDK には修正プログラムをインストールしないでください。

### <a name="prerequisites-before-applying-the-2005-update"></a>前提条件:2005 更新プログラムを適用する前

Azure Stack Hub の 2005 リリースは、次の修正プログラムが適用された 2002 リリースに適用する必要があります。

- [Azure Stack Hub 修正プログラム 1.2002.65.171](https://support.microsoft.com/topic/d743db84-df31-496b-b37c-6e5618b4cc8f)

### <a name="after-successfully-applying-the-2005-update"></a>2005 更新プログラムが正常に適用された後

2005 リリース以降では、新しいメジャー バージョンに更新すると (たとえば、1.2002.x から 1.2005.x)、その新しいメジャー バージョン内の最新の修正プログラム (存在する場合) が自動的にインストールされます。

2005 のインストール後に、2005 修正プログラムがリリースされた場合は、それらをインストールする必要があります。

- [Azure Stack Hub 修正プログラム 1.2005.29.100](hotfix-1-2005-29-100.md)
::: moniker-end

::: moniker range="azs-2002"
## <a name="2002-build-reference"></a>2002 ビルドのリファレンス

Azure Stack Hub 2002 更新プログラムのビルド番号は **1.2002.0.35** です。

> [!IMPORTANT]  
> Azure Stack Hub 2002 更新プログラムでは、Microsoft は [Azure Stack Hub サポート ポリシー ステートメント](azure-stack-servicing-policy.md)を一時的に延長しています。  Microsoft は現在、COVID-19 に対応中のユーザーで、Azure Stack Hub システムとその更新および管理の方法に関する重要な意思決定を行う可能性がある世界中のお客様と連携しています。その結果、お客様のデータセンターのビジネス運営が引き続き通常通り運営できるようにしています。 Microsoft では、お客様をサポートするために、3 つ前までの更新プログラムのバージョンを含めるように一時的なサポート ポリシー変更の延長を行っています。  その結果、新しくリリースされた 2002 更新プログラムと、3 つ前までの更新プログラムのバージョン (1910、1908、1907 など) のいずれもがサポートされるようになります。

### <a name="update-type"></a>更新の種類

Azure Stack Hub 2002 更新プログラムのビルドの種類は **完全** です。

2002 更新プログラム パッケージは、以前の更新プログラムよりも大きなサイズです。 サイズが大きいため、ダウンロード時間が長くなります。 この更新プログラムは **準備中** 状態の時間が長くなります。オペレーターは以前の更新プログラムよりもこのプロセスの時間が長くなることを想定してください。 内部テストでは、2002 更新プログラムの実行時間が次のように予測されました。4 ノード:15 から 42 時間、8 ノード:20 から 50 時間、12 ノード:20 から 60 時間、16 ノード:25 から 70 時間。 更新プログラムの正確なランタイムは一般的に、ご使用のシステムでテナント ワークロードによって使用されている容量、システム ネットワーク接続 (インターネットに接続されている場合)、およびシステム ハードウェアの仕様に左右されます。 実行時間がこの予測値よりも短くなったり長くなったりすることは一般的ではなく、更新が失敗した場合を除き、Azure Stack Hub オペレーターによるアクションは不要です。 このおおよその実行時間は 2002 更新プログラムに固有であり、他の Azure Stack Hub 更新プログラムと比較することはできません。

更新プログラムのビルドの種類については、「[Azure Stack Hub での更新プログラム管理の概要](azure-stack-updates.md)」を参照してください。

<!-- ## What's in this update -->

<!-- The current theme (if any) of this release. -->

### <a name="whats-new"></a>新機能

<!-- What's new, also net new experiences and features. -->

- AzureRM に基づく Azure Stack Hub 管理 PowerShell モジュールの新しいバージョン (1.8.1) がリリースされました。
- Azure Stack Hub 管理 REST API の新しいバージョンがリリースされました。 エンドポイントと破壊的変更の詳細については、[API リファレンス](/rest/api/azure-stack/)でご確認ください。
- 新しい Azure PowerShell テナント モジュールは、2020 年 4 月 15 日に Azure Stack Hub 用にリリースされます。 現在使用されている Azure RM モジュールは引き続き機能しますが、ビルド 2002 の後は更新されません。
- 構成された syslog サーバーの接続の問題について報告するために Azure Stack Hub 管理者ポータルに新しい警告アラートが追加されました。 アラートのタイトルは、**The Syslog client encountered a networking issue while sending a Syslog message** (syslog クライアントは syslog メッセージの送信中にネットワークの問題を検出しました) です。
- ネットワーク タイム プロトコル (NTP) サーバーの接続の問題について報告するために Azure Stack Hub 管理者ポータルに新しい警告アラートが追加されました。 アラートのタイトルは、**Invalid Time Source on [node name]** ([ノード名] の時間ソースが無効です) です。
- 2002 での TLS 制限に関連する破壊的変更により、[Java SDK](https://azure.microsoft.com/develop/java/)の新しいパッケージがリリースされました。 新しい Java SDK 依存関係をインストールする必要があります。 手順については、「[Java と API バージョンのプロファイル](../user/azure-stack-version-profiles-java.md?view=azs-2002&preserve-view=true#java-and-api-version-profiles)」を参照してください。
- System Center Operations Manager - Azure Stack Hub MP の新しいバージョン (1.0.5.10) が利用できます。これは、API の破壊的変更により、2002 を実行しているすべてのシステムで必要となります。 この API の変更は、バックアップとストレージのパフォーマンス ダッシュボードに影響します。最初にすべてのシステムを 2002 に更新してから MP を更新することをお勧めします。

### <a name="improvements"></a>機能強化

<!-- Changes and product improvements with tangible customer-facing value. -->

- この更新プログラムには、今後の完全な更新のパフォーマンスを大幅に向上させる更新プロセスの変更が含まれています。 これらの変更は、2002 リリース後の次の完全な更新で有効になり、特にホスト オペレーティング システムが更新される完全な更新のフェーズのパフォーマンスを向上させます。 ホスト オペレーティング システムの更新のパフォーマンスを向上させると、完全な更新中にテナントのワークロードが影響を受ける時間が大幅に短縮されます。
- Azure Stack Hub 適合性チェッカー ツールは、AD Graph に割り当てられているすべての TCP IP ポートを使用して AD Graph の統合を検証するようになりました。
- オフライン シンジケーション ツールは、信頼性に関する機能強化によって更新されました。 このツールは GitHub では入手できなくなり、[PowerShell ギャラリーに移動されました](https://www.powershellgallery.com/packages/Azs.Syndication.Admin/)。 詳細については、「[Azure Stack Hub に Marketplace の項目をダウンロードする](azure-stack-download-azure-marketplace-item.md)」を参照してください。
- 新しい監視機能が導入されています。 物理ホストとインフラストラクチャ VM のディスク領域不足のアラートは、プラットフォームによって自動的に修復されます。この操作が失敗した場合にのみ、オペレーターがアクションを実行するために、Azure Stack Hub 管理者ポータルにアラートが表示されます。
- [診断ログの収集](./diagnostic-log-collection.md?preserve-view=true&view=azs-2002)に対する改善。 新しいエクスペリエンスでは、BLOB ストレージ アカウントを事前に構成する必要がなくなるため、診断ログの収集が合理化されて簡素化されます。 ストレージ環境が事前構成されるため、サポート ケースを開く前にログを送信でき、サポート コールにかかる時間が短縮されます。
- [事前ログ収集とオンデマンドのログ収集](./diagnostic-log-collection.md?preserve-view=true&view=azs-2002)の両方にかかる時間が 80% 削減されました。 ログ収集時間は、この予想値より長くなることがありますが、ログ収集が失敗しない限り、Azure Stack Hub オペレーターによる操作は必要ありません。
- 更新が開始された後、Azure Stack Hub 更新パッケージのダウンロードの進行状況が更新ブレードに表示されるようになりました。 これは、[自動ダウンロードを使用して更新パッケージを準備する](azure-stack-update-prepare-package.md#automatic-download-and-preparation-for-update-packages)ことを選択した、接続済みの Azure Stack Hub システムに対してのみ適用されます。
- ネットワーク コントローラーのホスト エージェントの信頼性に関する機能強化。
- 修正プログラムや更新プログラムを適用中の内部 DNS サービスの回復性ロジックを向上させる、DNS Orchestrator と呼ばれる新しいマイクロサービスが導入されました。
- VM の作成中にブート診断ストレージ アカウントパラメーターの無効な BLOB URI をエラーとする新しい要求検証を追加しました。
- VM の CRUD 操作を容易にするホスト上の 2 つのサービスである Rdagent とホスト エージェントの自動修復とログ作成の機能強化が追加されました。
- Azure Stack のバージョンや課金モデルなどのさまざまなプロパティにより、利用している Azure Stack と互換性のないマーケットプレース製品を管理者にダウンロードさせないようにする属性を Microsoft が追加できるようにする新しい機能がマーケットプレース管理に追加されました。 これらの属性を追加できるのは Microsoft だけです。 詳細については、「[ポータルを使用して Marketplace 項目をダウンロードする](azure-stack-download-azure-marketplace-item.md#use-the-portal-to-download-marketplace-items)」を参照してください。

### <a name="changes"></a>[変更点]

- 管理者ポータルで、操作が進行中かどうかが Azure Stack 領域の横にあるアイコンで表示されるようになりました。 アイコンの上にマウスポインターを置くと、操作の名前が表示されます。 これにより、何時間も実行されることがあるバックアップ ジョブやストレージの拡張など、実行中のシステム バックグラウンド操作を識別することができます。

- 次の管理者 API が非推奨となりました。

  | リソース プロバイダー       | リソース              | Version            |
  |-------------------------|-----------------------|--------------------|
  | Microsoft.Storage.Admin | farms                 | 2015-12-01-preview |
  | Microsoft.Storage.Admin | farms/acquisitions    | 2015-12-01-preview |
  | Microsoft.Storage.Admin | farms/shares          | 2015-12-01-preview |
  | Microsoft.Storage.Admin | farms/storageaccounts | 2015-12-01-preview |

- 次の管理者 API は、新しいバージョン (2018-09-01) に置き換えられました。

  | リソース プロバイダー      | リソース              | Version    |
  |------------------------|-----------------------|------------|
  | Microsoft.Backup.Admin | backupLocation         | 2016-05-01 |
  | Microsoft.Backup.Admin | backups                | 2016-05-01 |
  | Microsoft.Backup.Admin | operations             | 2016-05-01 |

- PowerShell を使用して Windows VM を作成するときに、VM で拡張機能をデプロイする場合は、必ず `provisionvmagent` フラグを追加してください。 このフラグがない場合、VM はゲスト エージェントなしで作成され、VM 拡張機能をデプロイする機能が削除されます。

   ```powershell
   $VirtualMachine = Set-AzureRmVMOperatingSystem `
     -VM $VirtualMachine `
     -Windows `
     -ComputerName "MainComputer" `
     -Credential $Credential -ProvisionVMAgent
  ```

### <a name="fixes"></a>修正

<!-- Product fixes that came up from customer deployments worth highlighting, especially if there is an SR/ICM associated to it. -->

- 仮想マシンの同じ NIC に複数のパブリック IP を追加すると、インターネット接続の問題が発生する問題を修正しました。 これで、2 つのパブリック IP を持つ NIC は正常に動作するようになります。
- Azure AD のホーム ディレクトリを構成する必要があることを示すアラートがシステムによって生成される原因となった問題を修正しました。
- アラートが自動的に閉じない原因となった問題を修正しました。 このアラートは、Azure AD のホーム ディレクトリが構成されている必要があることを示していましたが、問題が軽減された後も閉じませんでした。
- 更新リソース プロバイダーの内部エラーの結果として、更新準備フェーズ中に更新が失敗する原因となった問題を修正しました。
- Azure Stack Hub のシークレット ローテーションの実行後にアドオン リソース プロバイダーの操作が失敗する原因となる問題を修正しました。
- ERCS ロールのメモリ不足のために Azure Stack Hub の更新エラーの一般的な原因となった問題を修正しました。
- Azure Stack Hub の更新の準備フェーズ中に更新ステータスが **[準備中]** ではなく **[インストール中]** と表示されていた、更新ブレードのバグを修正しました。
- 仮想スイッチ上の RSC 機能で不整合が発生し、ロード バランサーを通過するトラフィックが破棄される問題を修正しました。 RSC 機能は既定で無効化されるようになりました。
- NIC の複数の IP 構成が原因でトラフィックが誤ってルーティングされ、送信接続が妨げられていた問題を修正しました。 
- NIC の MAC アドレスがキャッシュされているときに、そのアドレスを別のリソースに割り当てると VM のデプロイ エラーが発生する問題が修正されました。
- RETAIL チャネルからの Windows VM イメージが、AVMA によってライセンス認証を行うことができない問題を修正しました。
- VM によって要求された仮想コアの数がノードの物理コアと等しい場合に VM が作成されないという問題を修正しました。 VM の仮想コアがノードの物理コアの数以下でも許可されるようになりました。
- ライセンスの種類を "null" に設定して、従量課金制イメージを BYOL に切り替えることができない問題を修正しました。
- VM スケール セットに拡張機能を追加できるようにするために問題を修正しました。

## <a name="security-updates"></a>セキュリティ更新プログラム

Azure Stack Hub のこの更新でのセキュリティ更新プログラムについては、「[Azure Stack Hub のセキュリティ更新プログラム](release-notes-security-updates.md)」を参照してください。

## <a name="hotfixes"></a>修正プログラム

Azure Stack Hub では、修正プログラムが定期的にリリースされます。 Azure Stack Hub を 2002 に更新する前に、必ず 1910 用の最新の Azure Stack Hub 修正プログラムをインストールしてください。

> [!NOTE]
> Azure Stack Hub 修正プログラムのリリースは累積的です。そのバージョンに対する以前の修正プログラムのリリースに含まれるすべての修正を取得するには、最新の修正プログラムをインストールするだけで済みます。

Azure Stack Hub 修正プログラムを適用できるのは Azure Stack Hub 統合システムのみです。ASDK には修正プログラムをインストールしないでください。

修正プログラムの詳細については、「[Azure Stack Hub サービス ポリシー](azure-stack-servicing-policy.md#hotfixes)」を参照してください。

### <a name="prerequisites-before-applying-the-2002-update"></a>前提条件:2002 更新プログラムを適用する前

Azure Stack Hub の 2002 リリースは、以下の修正プログラムが適用された 1910 リリースに適用する必要があります。

<!-- One of these. Either no updates at all, nothing is required, or the LATEST hotfix that is required-->
- [Azure Stack Hub 修正プログラム 1.1910.84.230](https://support.microsoft.com/help/4592243)

### <a name="after-successfully-applying-the-2002-update"></a>2002 更新プログラムの適用に成功した後

この更新プログラムをインストールした後、適用可能な修正プログラムがあればインストールします。

<!-- One of these. Either no updates at all, nothing is required, or the LATEST hotfix that is required-->
- [Azure Stack Hub 修正プログラム 1.2002.65.171](https://support.microsoft.com/topic/d743db84-df31-496b-b37c-6e5618b4cc8f)
::: moniker-end

<!------------------------------------------------------------>
<!------------------- UNSUPPORTED VERSIONS ------------------->
<!------------------------------------------------------------>
::: moniker range="azs-1910"
## <a name="1910-archived-release-notes"></a>1910 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1908"
## <a name="1908-archived-release-notes"></a>1908 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1907"
## <a name="1907-archived-release-notes"></a>1907 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1906"
## <a name="1906-archived-release-notes"></a>1906 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1905"
## <a name="1905-archived-release-notes"></a>1905 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1904"
## <a name="1904-archived-release-notes"></a>1904 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1903"
## <a name="1903-archived-release-notes"></a>1903 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1902"
## <a name="1902-archived-release-notes"></a>1902 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1901"
## <a name="1901-archived-release-notes"></a>1901 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1811"
## <a name="1811-archived-release-notes"></a>1811 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1809"
## <a name="1809-archived-release-notes"></a>1809 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1808"
## <a name="1808-archived-release-notes"></a>1808 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1807"
## <a name="1807-archived-release-notes"></a>1807 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1805"
## <a name="1805-archived-release-notes"></a>1805 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1804"
## <a name="1804-archived-release-notes"></a>1804 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1803"
## <a name="1803-archived-release-notes"></a>1803 アーカイブされたリリース ノート
::: moniker-end
::: moniker range="azs-1802"
## <a name="1802-archived-release-notes"></a>1802 アーカイブされたリリース ノート
::: moniker-end

::: moniker range="<azs-2002"
以前のバージョンの Azure Stack Hub リリースノートには、左側の目次の [ **[リソース] > [リリース ノートのアーカイブ]**](./relnotearchive/release-notes-1910.md) からアクセスできます。 これらのアーカイブされた記事は、参照のみを目的に提供されており、これらのバージョンのサポートを意味しているわけではありません。 Azure Stack のサポートについては、「[Azure Stack Hub サービス ポリシー](azure-stack-servicing-policy.md)」を参照してください。 さらにサポートが必要な場合は、Microsoft カスタマー サポート サービスにお問い合わせください。
::: moniker-end