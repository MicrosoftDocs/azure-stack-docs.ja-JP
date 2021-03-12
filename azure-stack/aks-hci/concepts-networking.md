---
title: 概念 - Azure Kubernetes Service (AKS) on Azure Stack HCI 内のネットワーク
description: 静的 IP、DHCP ネットワーク、ロードバランサーなど、Azure Kubernetes Service (AKS) on Azure Stack HCI におけるネットワークについて説明します。
author: abha
ms.topic: conceptual
ms.date: 02/17/2021
ms.author: abha
ms.custom: fasttrack-edit
ms.openlocfilehash: a8e96783be3228aae525027855c1498b0269bd57
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101874519"
---
# <a name="network-concepts-for-deploying-azure-kubernetes-service-aks-on-azure-stack-hci"></a>Azure Kubernetes Service (AKS) on Azure Stack HCI をデプロイするためのネットワークの概念

アプリケーション開発に対するコンテナー ベースのマイクロサービス アプローチでは、アプリケーション コンポーネントは、連携して各自のタスクを処理する必要があります。 Kubernetes には、このアプリケーションの通信を可能にするさまざまなリソースが提供されています。 アプリケーションへの接続と内部または外部への公開を実行できます。 高可用性アプリケーションをビルドするために、アプリケーションの負荷を分散させることができます。 

この記事では、AKS on Azure Stack HCI 内でアプリケーションにネットワークを提供する主要な概念について説明します。 その後、さまざまなデプロイ モデルと、ご自身のオンプレミスの AKS デプロイに最適なネットワーク インフラストラクチャを設定する例を紹介します。

## <a name="kubernetes-on-azure-stack-hci-basics"></a>Azure Stack HCI 上の Kubernetes の基本

アプリケーションへのアクセスまたはアプリケーション コンポーネントの相互通信を可能にするため、Kubernetes には、仮想ネットワークに対する抽象化レイヤーが提供されています。 仮想ネットワークに接続された Kubernetes ノードは、ポッドに対して受信接続と送信接続を実行できます。 これらのネットワーク機能を提供するために、各ノードで *kube-proxy* コンポーネントが実行されます。

Kubernetes 内では、特定のポートで IP アドレスまたは DNS 名を使用した直接アクセスを許可するように、"*サービス*" によってポッドが論理的にグループ化されます。 "*ロード バランサー*" を使用してトラフィックを分散させることもできます。 

Azure Stack HCI プラットフォームは、AKS on Azure Stack HCI クラスターの仮想ネットワークの簡略化にも役立ちます。 AKS on Azure Stack HCI 上にクラスターを作成すると、基礎となる `HAProxy` ロード バランサーのリソースが作成され、構成されます。 お使いの Kubernetes クラスター上にアプリケーションを構築すると、ご自身の Kubernetes サービスに対して IP アドレスが構成されます。

## <a name="aks-on-azure-stack-hci-resources"></a>AKS on Azure Stack HCI のリソース 

アプリケーション ワークロードのネットワーク構成を簡素化するため、AKS on Azure Stack HCI によって、IP アドレスがデプロイ内の次のオブジェクトに割り当てられます。 

- **Kubernetes クラスター API サーバー** - API サーバーは、Kubernetes API を公開する Kubernetes コントロール プレーンのコンポーネントです。 API サーバーは、Kubernetes コントロール プレーンのフロントエンドです。 API サーバーには、基礎となるネットワーク モデルに関係なく、常に静的 IP アドレスが割り当てられます。

- **Kubernetes ノード (仮想マシン上)** - Kubernetes クラスターは、コンテナ化されたアプリケーションを実行する一連のワーカー マシン (ノードと呼ばれます) で構成されています。 各クラスターには、少なくとも 1 つのワーカー ノードがあります。 AKS on Azure Stack HCI の場合、Kubernetes ノードは仮想マシン内で実行されます。 これらの仮想マシンは、Azure Stack HCI クラスター上に作成されます。 

- **Kubernetes services** - Kubernetes 内では、特定のポートで IP アドレスまたは DNS 名を使用した直接アクセスを許可するように、"*サービス*" によってポッドが論理的にグループ化されます。 "*ロード バランサー*" を使用してトラフィックを分散させることもできます。 Kubernetes サービスには、基礎となるネットワーク モデルに関係なく、常に静的 IP アドレスが割り当てられます。

- **HAProxy ロード バランサー** - HAProxy (高可用性プロキシ) は、Web サーバーが受信要求を複数のエンドポイントに分散できるようにする、TCP/HTTP ロード バランサーとプロキシ サーバーです。 AKS-HCI 内のすべてのワークロード クラスターに、独自の特殊な仮想マシン内で実行されている HAProxy ロード バランサーがあります。

- **Microsoft オンプレミス クラウド サービス** - これは、オンプレミスの Azure Stack HCI クラスター上で Kubernetes を実行するための Azure Stack HCI クラウド プロバイダーです。 このオブジェクトの IP アドレス割り当て方法は、Azure Stack HCI クラスターが従うネットワーク モデルによって決まります。


## <a name="virtual-networks"></a>仮想ネットワーク

AKS on Azure Stack HCI 内では、仮想ネットワークを使用して、IP アドレスが、そのアドレスを必要とする上記の Kubernetes リソースに割り当てられます。 必要な AKS on Azure Stack HCI ネットワーク アーキテクチャに応じて、2 つのネットワーク モデルから選択できます。 ここで AKS on Azure Stack HCI デプロイに対して定義した仮想ネットワーク アーキテクチャは、データセンター内の基礎となる物理ネットワーク アーキテクチャとは異なることに注意してください。

- 静的 IP ネットワーク - 仮想ネットワークによって、静的 IP アドレスが、Kubernetes クラスター API サーバー、Kubernetes ノード、基礎となる VM、ロードバランサー、およびお使いのクラスター上で実行するすべての Kubernetes サービスに割り当てられます。

- DHCP ネットワーク - 仮想ネットワークによって、動的 IP アドレスが、DHCP サーバーを使用して、Kubernetes ノード、基礎となる VM、およびロード バランサーに割り当てられます。 Kubernetes クラスター API サーバーと、お使いのクラスター上で実行するすべての Kubernetes サービスには、まだ静的 IP アドレスが割り当てられています。

### <a name="virtual-ip-pool"></a>仮想 IP プール
仮想 IP (VIP) プールは AKS on Azure Stack HCI デプロイに必須です。 VIP プールは、お使いのアプリケーションが常に到達可能であることを保証するために、IP アドレスを Kubernetes クラスター API サーバーおよび Kubernetes サービス用の外部 IP に割り当てる際に使用される、予約済み静的 IP アドレス範囲です。 仮想ネットワーク モデルに関係なく、お使いの AKS ホスト デプロイに対して VIP プールをプロビジョニングする必要があります。 VIP プール内の IP アドレスの数は、静的ネットワーク モデルと DHCP ネットワーク モデルの両方について、ご自身のデプロイ内で実行されているワークロード クラスターと Kubernetes サービスの数によって異なります。 ただし、VIP プールは、ご自身のネットワーク モデルに応じて、次の点で異なります。

- 静的 IP - 静的 IP を使用している場合は、ご自身の仮想 IP が同じサブネットに含まれることを確認します。 
- DHCP - ネットワークが DHCP を使用して構成されている場合は、ご自身のネットワーク管理者と連携して、VIP プールに使用する IP 範囲を予約する必要があります。

### <a name="kubernetes-node-ip-pool"></a>Kubernetes ノード IP プール
前述のように、Kubernetes ノードは、AKS on Azure Stack HCI デプロイ内の特殊な仮想マシン上で実行されます。 これらの仮想マシンに、Kubernetes ノード間での通信を目的として IP アドレスが AKS on Azure Stack HCI によって割り当てられます。 DHCP ネットワーク モデルの場合、Kubernetes ノードへの IP アドレスは、ご自身のネットワーク上の DHCP サーバーによって動的に割り当てられるため、Kubernetes ノード VM プールを指定する必要はありません。 静的 IP ベースのネットワーク モデルの場合は、Kubernetes ノード IP プールの範囲を指定する必要があります。 この範囲内の IP アドレスの数は、AKS ホストおよびワークロード Kubernetes クラスター全体にわたる Kubernetes ノードの合計数によって異なります。

### <a name="virtual-network-with-static-ip-networking"></a>静的 IP ネットワークを使用した仮想ネットワーク

このネットワーク モデルにより、ご自身のデプロイ内のすべてのオブジェクトに静的 IP アドレスを割り当てる仮想ネットワークが作成されます。 静的 IP ネットワークを使用する付加的な利点として、長期間維持されるデプロイとアプリケーション ワークロードが常に到達可能であることが保証されます。 ご自身の AKS on Azure Stack HCI デプロイに対して、静的 IP 仮想ネットワーク モデルを作成することをお勧めします。

静的 IP 構成を使用して仮想ネットワークを定義する場合は、次のパラメーターを指定する必要があります。

- 名前: 仮想ネットワークの名前
- AddressPrefix: お使いのサブネットに使用する IP アドレスのプレフィックス。
- ゲートウェイ: サブネットの既定のゲートウェイの IP アドレス。
- DNS サーバー: サブネットに使用する DNS サーバーを指す IP アドレスの配列。 最小 1 台、最大 3 台のサーバーを指定できます。
- Kubernetes ノード IP プール: お使いの Kubernetes ノード VM に使用する連続した IP アドレス範囲。
- 仮想 IP プール: 外部 IP アドレスを必要とするお使いの Kubernetes クラスター API サーバーと Kubernetes サービスに使用する連続した IP アドレス範囲。
- vLAN ID: 仮想ネットワークの vLAN ID。 省略した場合、仮想ネットワークはタグ付けされません。

### <a name="virtual-network-with-dhcp-networking"></a>DHCP ネットワークを使用した仮想ネットワーク

このネットワーク モデルにより、ご自身のデプロイ内のすべてのオブジェクトに動的 IP アドレスを割り当てる仮想ネットワークが作成されます。  

静的 IP 構成を使用して仮想ネットワークを定義する場合は、次のパラメーターを指定する必要があります。

- 名前: 仮想ネットワークの名前
- 仮想 IP プール: 外部 IP アドレスを必要とするお使いの Kubernetes クラスター API サーバーと Kubernetes サービスに使用する連続した IP アドレス範囲。
- vLAN ID: 仮想ネットワークの vLAN ID。 省略した場合、仮想ネットワークはタグ付けされません。

## <a name="microsoft-on-premises-cloud-service"></a>Microsoft オンプレミス クラウド サービス

Microsoft オンプレミス クラウド (MOC) は、Azure Stack HCI および Windows Server ベースの SDDC クラウド上で仮想マシンを管理できるようにする新しい管理スタックです。 MOC の構成は次のとおりです。

- クラスター上にデプロイされている可用性の高い `cloud agent` サービスの 1 つのインスタンス。 このエージェントは、Azure Stack HCI クラスター内の任意の 1 つのノードで実行されます。 
- すべての Azure Stack HCI 物理ノードで実行されている `node agent`。 

`-cloudserviceCIDR` は、可用性の高いクラウド エージェント サービスを確保するために仮想 IP を割り当てる際に使用される `Set-AksHciConfig` コマンド内のパラメーターです。

MOC サービス用の IP アドレスの割り当ては、ご自身の Azure Stack HCI クラスター デプロイが従う基礎となるネットワーク モデルによって異なります。 **MOC サービス用の IP アドレスの割り当ては、仮想ネットワーク モデルには依存しません。この割り当ては、データセンター内の Azure Stack HCI クラスター ノードに IP アドレスを提供する、基礎となる物理ネットワークにのみ依存します。**

### <a name="azure-stack-hci-cluster-nodes-with-a-dhcp-based-ip-address-allocation-model"></a>DHCP ベースの IP アドレス割り当てモデルを使用した Azure Stack HCI クラスター ノード

ご自身の Azure Stack HCI ノードに、物理ネットワーク上に存在する DHCP サーバーから IP アドレスが割り当てられている場合、IP アドレスを MOC サービスに明示的に指定する必要はありません。

### <a name="azure-stack-hci-cluster-nodes-with-a-static-ip-allocation-model"></a>静的 IP 割り当てモデルを使用した Azure Stack HCI クラスター ノード 

ご自身の Azure Stack HCI クラスター ノードに静的 IP アドレスが割り当てられている場合は、MOC クラウド サービスに対して IP アドレスを明示的に指定する必要があります。 MOC サービスのこの IP アドレスは、Azure Stack HCI クラスターノードと同じサブネット内に存在する必要があります。 MOC サービスに対して IP アドレスを明示的に割り当てるには、`Set-AksHciConfig` コマンドを `-cloudserviceCIDR` パラメーターを指定して使用します。 IP アドレスは必ず CIDR 形式で入力します (例: "10.11.23.45/16")。

## <a name="compare-network-models"></a>ネットワーク モデルを比較する

DHCP と静的 IP は両方とも、AKS on Azure Stack HCI デプロイにネットワーク接続を提供します。 ただし、それぞれに長所と短所があります。 大まかに言えば、次の考慮事項が適用されます。

* **DHCP**
    - AKS on Azure Stack HCI デプロイ内の一部のリソースの種類については、長時間維持される IP アドレスが保証されません。
    - ご自身のデプロイが最初に予想したよりも大きくなった場合の予約済み DHCP IP アドレスの拡張がサポートされます。
    
* **[静的 IP]**
    - AKS on Azure Stack HCI デプロイ内のすべてのリソースの種類について、長時間維持される IP アドレスが保証されます。
    - Kubernetes ノード IP プールの自動拡張はサポートされていないため、Kubernetes ノード IP プールを使い果たしたときに、新しいクラスターを作成できないことがあります。


次の表は、静的 IP ネットワーク モデルと DHCP ネットワーク モデルのリソースに対する IP アドレスの割り当てを比較したものです。

| 機能                                                                                   | 静的 IP   | DHCP |
|----------------------------------------------------------------------------------------------|-----------|-----------|
| Kubernetes クラスター API サーバー                                             | VIP プールを使用して静的に割り当て済み | VIP プールを使用して静的に割り当て済み |
| Kubernetes ノード (仮想マシン上)                                                                         | Kubernetes ノード IP プールを使用して割り当て済み | 動的に割り当て済み |
| Kubernetes サービス                                          | VIP プールを使用して静的に割り当て済み | VIP プールを使用して静的に割り当て済み |
| HAProxy ロード バランサー VM                                          | Kubernetes ノード IP プールを使用して割り当て済み | 動的に割り当て済み |
| Microsoft オンプレミス クラウド サービス                                               | Azure Stack HCI クラスター ノードの物理ネットワーク構成によって異なる | Azure Stack HCI クラスター ノードの物理ネットワーク構成によって異なる |
| VIP プール                                             | Mandatory | Mandatory |
| Kubernetes ノード IP プール | Mandatory | サポートされていません |


## <a name="minimum-ip-address-reservations-for-an-aks-on-azure-stack-hci-deployment"></a>AKS on Azure Stack HCI デプロイに対する最小 IP アドレス予約
予約済み IP アドレスの数は、デプロイ モデルに関係なく同じです。 このセクションでは AKS on Azure Stack HCI デプロイ モデルに基づいて予約する IP アドレスの数について説明します。

### <a name="minimum-ip-address-reservation"></a>最小 IP アドレス予約

少なくとも、次の数の IP アドレスをご自身のデプロイに対して予約する必要があります。

| クラスターの種類  | コントロール プレーン ノード | ワーカー ノード | 更新操作 | Load Balancer  |
| ------------- | ------------------ | ---------- | ----------| -------------|
| AKS ホスト |  1 IP |  NA  |  2 IP |  NA  |
| ワークロード クラスター  |  ノードあたり 1 IP  | ノードあたり 1 IP |  5 IP  |  1 IP |

さらに、次の数の IP アドレスをご自身の VIP プールに対して予約する必要があります。

| リソースの種類  | IP アドレスの数 
| ------------- | ------------------
| クラスター API サーバー |  クラスターあたり 1 
| Kubernetes サービス  |  サービスあたり 1  

ご覧のように、必要な IP アドレスの数は、AKS on Azure Stack HCI アーキテクチャと Kubernetes クラスター上で実行するサービスの数によって異なります。 お使いのデプロイに対して合計 256 IP アドレス (/24 サブネット) を予約することをお勧めします。

### <a name="walking-through-an-example-deployment"></a>デプロイ例の手順

Jane は、AKS on Azure Stack HCI を始めたばかりの IT 管理者です。 彼女は、Kubernetes クラスター A と Kubernetes クラスター B の 2 つの Kubernetes クラスターを、自分の Azure Stack HCI クラスターにデプロイしたいと考えています。 また、自分のクラスター上で投票アプリケーションを実行したいとも考えています。 このアプリケーションには、2 つのクラスターと、バックエンド データベースの 1 つのインスタンスにわたって実行されている、フロントエンド UI の 3 つのインスタンスがあります。

- Kubernetes クラスター A には 3 つのコントロール プレーン ノードと 5 つのワーカー ノードがあります
- Kubernetes クラスター B には 1 つのコントロール プレーン ノードと 3 つのワーカー ノードがあります
- フロントエンド UI の 3 つのインスタンス (ポート 443)
- バックエンド データベースの 1 つのインスタンス (ポート 80)

上の表に基づいて、次を予約する必要があります。

- 自分の AKS ホストに対して 3 つの IP アドレス (コントロール プレーン ノード用に 1 つの IP、更新操作を実行するために 2 つの IP)
- クラスター A 内の自分のコントロール プレーン ノードに対して 3 つの IP アドレス (コントロール プレーン ノードごとに 1 つの IP)
- クラスター A 内の自分のワーカー ノードに対して 5 つの IP アドレス (コントロール プレーン ノードごとに 1 つの IP)
- クラスター A に対して追加で 6 つの IP アドレス (更新操作を実行するために 5 つの IP、ロード バランサー用に 1 つのIP)
- クラスター B 内の自分のコントロール プレーン ノードに対して 1 つの IP アドレス (コントロール プレーン ノードごとに 1 つの IP)
- クラスター B 内の自分のワーカー ノードに対して 3 つの IP アドレス (コントロール プレーン ノードごとに 1 つの IP)
- クラスター B に対して追加で 6 つの IP アドレス (更新操作を実行するために 5 つの IP、ロード バランサー用に 1 つのIP)
- 自分の Kubernetes クラスター API サーバー用に 2 つの IP アドレス (Kubernetes クラスターごとに 1 つの IP)
- 自分の Kubernetes サービス用に 3 つの IP アドレス (すべてが同じポートを使用しているため、フロントエンド UI のインスタンスごとに 1 つの IP アドレス)。 バックエンド データベースは、別のポートにあるため、3 つの IP アドレスのいずれかを使用できます

前述したように、Jane は合計 32 の IP アドレスを必要としています。 そのため、Jane は、自分の仮想ネットワーク用に /26 サブネットを予約する必要があります。 

### <a name="splitting-reserved-ip-addresses-based-on-a-static-ip-network-model"></a>静的 IP ネットワーク モデルに基づく予約済み IP アドレスの分割

予約済み IP アドレスは、合計数は同じですが、IP グループ間でどのように分割されるかはデプロイ モデルによって決まります。 前述したように、静的 IP ネットワーク モデルには次の 2 つの IP プールがあります。

- Kubernetes ノード IP プール - Kubernetes ノード VM およびロード バランサー VM 用。 この IP プールには、更新操作の実行に必要な IP アドレスも含まれています。
- 仮想 IP プール - Kubernetes API サーバーおよび Kubernetes サービス用

上記の例を使用して、Jane は次の IP アドレスを、さらに複数の VIP プールと Kubernetes ノード IP プールに分割する必要があります。
- 32 個の IP アドレスのうち、自分の VIP プール用に 5 つ (Kubernetes クラスター API サーバー用に 2 つ、Kubernetes サービス用に 3 つ)。
- Kubernetes ノード IP プール用に 27 個 (すべての IP アドレスが、Kubernetes ノード IP プールと基礎となる VM、ロード バランサー VM、および更新操作用)。

### <a name="splitting-reserved-ip-addresses-based-on-a-dhcp-network-model"></a>DHCP ネットワーク モデルに基づく予約済み IP アドレスの分割

予約済み IP アドレスは、合計数は同じですが、IP グループ間でどのように分割されるかはデプロイ モデルによって決まります。 前述したように、DHCP ネットワーク モデルには次の 1 つの IP プールがあります。

- 仮想 IP プール - Kubernetes API サーバーおよび Kubernetes サービス用

上記の例を使用します。
- Jane は、自分の DHCP サーバー上で合計 32 個の IP アドレスまたは /26 サブネットを予約する必要があります。 
- 彼女は 32 個の IP アドレスのうち、自分の VIP プール用に 5 つ (Kubernetes クラスター API サーバー用に 2 つ、Kubernetes サービス用に 3 つ) を予約する必要があります 

## <a name="next-steps"></a>次のステップ

この記事では、AKS on Azure Stack HCI 内でアプリケーションにネットワークを提供する主要な概念について学習しました。 次に、以下を実行できます。

- [AKS on Azure Stack HCI 上でプロキシ設定を構成する](./proxy.md)
- [システム要件](./system-requirements.md)

