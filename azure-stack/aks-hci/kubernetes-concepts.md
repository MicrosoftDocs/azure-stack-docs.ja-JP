---
title: Azure Stack HCI 上の Azure Kubernetes Services (AKS) における Kubernetes の基本概念
description: Kubernetes のクラスターおよびワークロードの基本コンポーネントと、それらが Azure Stack HCI 上の Azure Kubernetes Service (AKS) の機能にどのように関連しているかを説明します。
author: daschott
ms.author: daschott
ms.topic: conceptual
ms.date: 12/02/2020
ms.openlocfilehash: 93c0103cc13fdffbee7ba0d16aea9c33474565ba
ms.sourcegitcommit: 2c6418ee465e67edd417961b1f5211b2e09dbd5f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102116769"
---
# <a name="clusters-and-workloads-for-azure-kubernetes-service-on-azure-stack-hci"></a>Azure Kubernetes Service on Azure Stack HCI のクラスターとワークロード

> 適用対象:Azure Stack HCI 上の AKS、Windows Server 2019 Datacenter 上の AKS ランタイム

Azure Stack HCI 上の Azure Kubernetes Service は、Azure Stack HCI を利用するエンタープライズ グレードの Kubernetes コンテナー プラットフォームです。 これには、Microsoft がサポートするコア Kubernetes、アドオン、専用 Windows コンテナー ホスト、および Microsoft がサポートする Linux コンテナー ホストが含まれており、 **デプロイとライフ サイクル管理のシンプルなエクスペリエンス** を用意することを目的としています。

この記事では、"コントロール プレーン"、"ノード"、および "ノード プール" などの主要な Kubernetes インフラストラクチャ コンポーネントを紹介します。 また、プール、デプロイ、およびセットなどのワークロード リソースと、リソースを名前空間にグループ化する方法も紹介します。

## <a name="kubernetes-cluster-architecture"></a>Kubernetes クラスターのアーキテクチャ
Kubernetes は、Azure Stack HCI 上の Azure Kubernetes Service のコア コンポーネントです。 Azure Stack HCI 上の Azure Kubernetes Service では、事前に定義された構成のセットを使用し、スケーラビリティを念頭に置いて効果的に Kubernetes クラスターがデプロイされます。
 
デプロイ操作では、Linux または Windows の仮想マシンを複数作成し、それらを結合して Kubernetes クラスターを作成します。 
 
デプロイされたシステムは、標準の Kubernetes ワークロードを受け取り、これらのワークロードをスケーリングしたり、必要に応じて仮想マシンの数やクラスターの数を上下にスケーリングする準備ができています。

Azure Kubernetes Service クラスターは、Azure Stack HCI 上で 2 つの主要コンポーネントに分割されています。

- *管理* クラスターでは、1 つ以上のターゲット クラスターをデプロイおよび管理するためのコア オーケストレーション メカニズムとインターフェイスが提供されます。
- *ターゲット* クラスター (ワークロード クラスターとも呼ばれます) は、アプリケーションのワークロードが実行され、管理クラスターによって管理される場所です。

![Azure Stack HCI 上の Azure Kubernetes Service の技術的アーキテクチャを示す図です。](.\media\concepts\architecture.png)

## <a name="management-cluster"></a>管理クラスター
Azure Stack HCI 上に Azure Kubernetes Service クラスターを作成すると、管理クラスターが自動的に作成され、構成されます。 この管理クラスターは、ワークロードが実行されるターゲット クラスターのプロビジョニングと管理を担います。 管理クラスターには、以下の Kubernetes コア コンポーネントが含まれています。
- *API サーバー* - API サーバーは、基になる Kubernetes API が公開される方法となっています。 このコンポーネントは、Windows Admin Center、PowerShell モジュール、`kubectl` などの管理ツールに対して相互作用を提供します。
- *ロード バランサー* - ロード バランサーは、管理クラスターの API サーバーのための負荷分散規則を備えた 1 つの専用 Linux VM です。

### <a name="windows-admin-center"></a>Windows Admin Center
Windows Admin Center では、Kubernetes オペレーターに向けて、Azure Stack HCI 上の Azure Kubernetes Service クラスターのライフサイクルを管理するための直感的な UI が提供されています。

### <a name="powershell-module"></a>PowerShell モジュール
PowerShell モジュールは、Azure Stack HCI 上の Azure Kubernetes Service をダウンロード、構成、デプロイする簡単な方法です。 PowerShell モジュールでは、追加のターゲット クラスターのデプロイと構成のほか、既存のクラスターの再構成がサポートされています。

## <a name="target-cluster"></a>ターゲット クラスター
ターゲット (ワークロード) クラスターは、Kubernetes コントロール プレーン コンポーネントおよび Linux ワーカー ノードを実行するために Linux VM を使用する、Kubernetes の高可用性デプロイです。 Windows ワーカー ノードを確立するために、Windows Server Core ベースの VM が使用されます。 1 つの管理クラスターによって管理されるターゲット クラスターが 1 つ以上存在する場合があります。

### <a name="worker-nodes"></a>ワーカー ノード
アプリケーションとサポート対象のサービスを実行するには、Kubernetes のノードが必要です。 Azure Stack HCI 上の Azure Kubernetes Service ターゲット クラスターには、ワーカー ノードが 1 つ以上あります。これは、Kubernetes ノード コンポーネントを実行する仮想マシン (VM) であり、アプリケーション ワークロードを構成するポッドやサービスをホストします。

### <a name="load-balancer"></a>Load Balancer
ロード バランサーは、Linux と、HAProxy および KeepAlive を実行する仮想マシンであり、管理クラスターによってデプロイされたターゲット クラスターに、負荷分散されたサービスを提供します。

Azure Stack HCI 上の Azure Kubernetes Service により、ターゲット クラスターごとに、少なくとも 1 つのロード バランサー仮想マシン (LB VM) が追加されます。 これに加えて、ターゲット クラスターでの API サーバーの高可用性のために、別のロード バランサーを作成できます。 ターゲット クラスターで作成された `LoadBalancer` 型の Kubernetes サービスはすべて、LB VM 内に負荷分散規則を作成することになります。

### <a name="add-on-components"></a>アドオン コンポーネント
任意のクラスターに展開できるオプションのアドオン コンポーネントがいくつかあります。特に重要なのは、Azure Arc、Prometheus、Grafana、Kubernetes ダッシュボードです。

## <a name="kubernetes-components"></a>Kubernetes コンポーネント
このセクションでは、ポッド、デプロイ、セットなど、Azure Stack HCI ターゲット クラスター上の Azure Kubernetes Service にデプロイできる Kubernetes のコア ワークロード コンポーネントについて紹介します。また、リソースを名前空間にグループ化する方法について説明します。

### <a name="pods"></a>ポッド

Kubernetes は *ポッド* を使用して、お使いのアプリケーションのインスタンスを実行します。 ポッドは、アプリケーションの単一のインスタンスを表します。 ポッドには、通常、コンテナーとの 1 対 1 のマッピングがありますが、1 つのポッドに複数のコンテナーが含まれる場合がある高度なシナリオが存在します。 これらのマルチコンテナー ポッドは、同じノード上にまとめてスケジュールされ、コンテナーが関連するリソースを共有できるようにします。

ポッドを作成する場合、"*リソース要求*" を定義して、一定量の CPU やメモリ リソースを要求できます。 Kubernetes スケジューラは、要件を満たす利用可能なリソースを備えたノード上で実行されるように、ポッドのスケジュール設定を試みます。 また、指定されたポッドでの基本ノードからのコンピューティング リソースの消費量が過大にならないように、リソースの上限を指定することも可能です。 ベスト プラクティスは、必要かつ許可されているリソースがどれかを Kubernetes スケジューラが認識できるように、すべてのポッドにリソース制限を組み入れることです。

詳細については、[Kubernetes ポッド][kubernetes-pods]と [Kubernetes ポッドのライフサイクル][kubernetes-pod-lifecycle]に関するページをご覧ください。

ポッドは論理リソースですが、コンテナーは、アプリケーション ワークロードが実行される場所です。 ポッドは通常、短期間の破棄可能なリソースであり、個々にスケジュール設定されたポッドには、Kubernetes が提供する高可用性と冗長性の機能の一部がありません。 代わりに、ポッドは、デプロイ コントローラーなどの Kubernetes の *コントローラー* によってデプロイされ管理されます。

### <a name="deployments-and-yaml-manifests"></a>デプロイと YAML マニフェスト

1 つの *デプロイ* は 1 つ以上の同一ポッドに相当し、Kubernetes デプロイ コントローラーによって管理されます。 デプロイでは、作成する *レプリカ* (ポッド) の数を定義し、Kubernetes スケジューラは、ポッドまたはノードで問題が発生した場合に、追加のポッドが正常なノード上にスケジュールされることを保証します。

デプロイを更新して、ポッドの構成、使用されるコンテナー イメージの構成、またはアタッチされたストレージの構成を変更できます。 デプロイ コントローラーは、指定された数のレプリカをドレインして終了し、新しいデプロイ定義からレプリカを作成して、デプロイ内のすべてのレプリカが更新されるまで、プロセスを継続します。

最もステートレスなアプリケーションでは、個々のポッドをスケジュール設定するのではなく、デプロイ モデルを使用する必要があります。 Kubernetes では、必要な数のレプリカがクラスター内で確実に実行されるように、デプロイの正常性と状態を監視できます。 個々のポッドのスケジュール設定だけを行うと、ポッドで問題が発生した場合、そのポッドは再起動されません。また、ポッドの現在のノードで問題が発生した場合、ポッドは正常なノード上に再スケジュールされません。

アプリケーションが、管理上の判断に常に利用できるインスタンスのクォーラムを必要としている場合、更新プロセスによってその機能が中断されることは望ましくありません。 *ポッドの中断バジェット* を使用すると、更新やノードのアップグレードの期間中に、デプロイ内でいくつのレプリカを停止できるかを定義できます。 たとえば、ご自分のデプロイ内に *5* 個のレプリカがある場合、ポッド中断数を *4* と定義して、一度に削除および再スケジュールできるレプリカ数を 1 個のみにすることが可能です。 ポッドのリソース制限と同様に、ベスト プラクティスは、レプリカの最小数を常に提示する必要があるアプリケーション上に、ポッドの中断バジェットを定義することです。

デプロイは通常、`kubectl create` または `kubectl apply` で作成および管理されます。 デプロイを作成するには、YAML (YAML ではマークアップ言語を使用しない) 形式のマニフェスト ファイルを定義します。 次の例では、NGINX Web サーバーの基本のデプロイを作成します。 デプロイでは、*3* 個のレプリカを作成すること、またポート *80* をコンテナー上に開くことを指定します。 また、CPU とメモリに対しては、リソース要求と制限が定義されます。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.2
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 250m
            memory: 64Mi
          limits:
            cpu: 500m
            memory: 256Mi
```

YAML マニフェスト内にロード バランサーなどのサービスを含めても、より複雑なアプリケーションを作成できます。

詳細については、[Kubernetes のデプロイ][kubernetes-deployments]に関するページを参照してください。

##### <a name="mixed-os-deployments"></a>混合 OS デプロイ

Azure Stack HCI ターゲット クラスター上の特定の Azure Kubernetes Service が、Linux と Windows の両方のワーカー ノードで構成されている場合、ワークロードのプロビジョニングをサポートできる OS にワークロードをスケジュールする必要があります。 Kubernetes には、ターゲット オペレーティング システムが稼働するノードにワークロードを確実に配置するため、2 つのメカニズムが用意されています。

- *ノード セレクター* は、ポッド仕様内の簡単なフィールドであり、オペレーティング システムが一致する正常なノードにのみスケジュールされるように、ポッドが制約されます。 
- *テイントと容認* は、連携動作して、ポッドが意図せずノードにスケジュールされないようにします。 ポッド仕様の "toleration" を通して明示的にテイントを容認していないポッドを受け入れないように、ノードを "テイント" することができます。

詳細については、[ノード セレクター][node-selectors]に関するページと、[テイントおよび容認][taints-tolerations]に関するページを参照してください。

### <a name="statefulsets-and-daemonsets"></a>StatefulSet と DaemonSet

デプロイ コントローラーでは、Kubernetes スケジューラを使って、使用できるリソースがある任意の利用可能なノード上で、指定された数のレプリカを実行します。 デプロイを使用するこの方法は、ステートレス アプリケーションでは十分なかもしれませんが、永続的な名前規則やストレージを必要とするアプリケーションでは、不十分です。 1 つのクラスター内で、各ノード上または選択されたノード上にレプリカが存在することが必要なアプリケーションの場合、デプロイ コントローラーでは、ノード間にレプリカが配信される方法を確認しません。

これらの種類のアプリケーションを管理できる Kubernetes リソースとして、次の 2 つがあります。

- *StatefulSet* - ストレージなど、個々のポッド ライフサイクルを超えて、アプリケーションの状態を保守します。
- *DaemonSet* - Kubernetes ブートストラップ プロセスの初期段階で、各ノード上で実行されるインスタンスを確保します。

### <a name="statefulsets"></a>StatefulSet

最新のアプリケーション開発では、通常、ステートレス アプリケーションを目指しますが、*StatefulSet* はデータベース コンポーネントを含むアプリケーションなど、ステートフル アプリケーションに使用できます。 StatefulSet は、1 つ以上のポッドが作成され管理されるデプロイとほぼ同じです。 デプロイ、スケーリング、アップグレード、および強制終了において、StatefulSet 内のレプリカは、適切な順序付けの手法に従います。 StatefulSet では (レプリカが再スケジュールされるときに)、名前規則、ネットワーク名、およびストレージが永続化されます。

`kind: StatefulSet` を使用して YAML 形式でアプリケーションを定義すると、StatefulSet コントローラーは要求されたレプリカのデプロイと管理を処理します。 データは永続的ストレージに書き込まれ、StatefulSet が削除された後でも基になっているストレージが保持されます。

詳細については、[Kubernetes の StatefulSet][kubernetes-statefulsets] に関するページを参照してください。

StatefulSet 内のレプリカは、Azure Stack HCI クラスター上の Azure Kubernetes Service 内にあるすべての利用可能なノード全体にスケジュールされ、実行されます。 セット内の 1 つ以上のポッドが、ノード上で実行されることを保証する必要がある場合は、代わりに、DaemonSet を使用できます。

### <a name="daemonsets"></a>DaemonSet

特定のログ コレクションや監視のニーズのために、すべてのノード上または選択されたノード上で、指定されたポッドを実行する必要が生じる場合があります。 この場合も、1 つ以上の同一ポッドをデプロイするために *DaemonSet* が使用されますが、DaemonSet コントローラーが、指定された各ノードでポッドのインスタンスが実行されることを保証します。

DaemonSet コントローラーでは、既定の Kubernetes スケジューラが起動済みになる前に、クラスターのブート プロセスの初期段階でノード上にポッドをスケジュール設定できます。 この機能によって、1 つのデプロイまたは StatefulSet 内の従来のポッドがスケジュールされる前に、確実に DaemonSet 内のポッドが開始されます。

StatefulSet と同様に、DaemonSet は `kind: DaemonSet` を使用して、YAML 定義の一部として定義されています。

詳細については、[Kubernetes の DaemonSet][kubernetes-daemonset] に関するページを参照してください。

### <a name="namespaces"></a>名前空間

ポッドやデプロイなどの Kubernetes リソースは、論理的に 1 つの *名前空間* にグループ化されます。 このグループ化により、Azure Stack HCI ターゲット クラスター上の Azure Kubernetes Service を論理的に分割し、リソースを作成、表示、または管理するためのアクセスを制限する方法が提供されます。 たとえば、名前空間を作成して業務グループを分けることができます。 ユーザーは、割り当てられている名前空間内のリソースのみを操作できます。

Azure Stack HCI 上に Azure Kubernetes Service クラスターを作成するときには、以下の名前空間を使用できます。

- *既定* - この名前空間は、何も指定されない場合に、既定でポッドおよびデプロイが作成される場所です。 より小規模な環境では、追加の論理分割を作成せずに、既定の名前空間にアプリケーションを直接デプロイできます。 `kubectl get pods` などの Kubernetes API を操作する場合、何も指定しないと、既定の名前空間が使用されます。
- *kube-system* - この名前空間は、DNS とプロキシ、または Kubernetes ダッシュボードのようなネットワーク機能など、重要なリソースが置かれている場所です。 通常は、この名前空間に独自のアプリケーションをデプロイしません。
- *kube-public* - この名前空間は、通常は使用されませんが、クラスター全体でリソースを表示可能にして、ユーザーが確認できるようにするために使用できます。

詳細については、[Kubernetes の名前空間][kubernetes-namespaces] に関するページを参照してください。

[kubernetes-pods]: https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/
[kubernetes-pod-lifecycle]: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/
[kubernetes-deployments]: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
[kubernetes-statefulsets]: https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/
[kubernetes-daemonset]: https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/
[kubernetes-namespaces]: https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
[node-selectors]: https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/
[taints-tolerations]: https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/
