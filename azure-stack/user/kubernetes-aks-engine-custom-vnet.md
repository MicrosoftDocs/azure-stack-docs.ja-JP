---
title: Azure Stack Hub のカスタム仮想ネットワークに Kubernetes クラスターをデプロイする
description: Azure Stack Hub のカスタム仮想ネットワークに Kubernetes クラスターをデプロイする方法について説明します。
author: mattbriggs
ms.topic: article
ms.date: 2/1/2021
ms.author: mabrigg
ms.reviewer: waltero
ms.lastreviewed: 9/2/2020
ms.openlocfilehash: 2b958dad58b04d27037f7110e2350e73ad6cd880
ms.sourcegitcommit: ccc4ee05d71496653b6e27de1bb12e4347e20ba4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102231508"
---
# <a name="deploy-a-kubernetes-cluster-to-a-custom-virtual-network-on-azure-stack-hub"></a>Azure Stack Hub のカスタム仮想ネットワークに Kubernetes クラスターをデプロイする 

カスタム仮想ネットワークで Azure Kubernetes Service (AKS) エンジンを使用して Kubernetes クラスターをデプロイできます。 この記事では、仮想ネットワークで必要な情報を見つける方法について説明します。 クラスターで使用されている IP アドレスの計算、API モデルでの値の設定、ルート テーブルとネットワーク セキュリティ グループの設定の手順を確認できます。

AKS エンジンを使用した Azure Stack Hub の Kubernetes クラスターでは、kubernet ネットワーク プラグインを使用します。 Azure での kubernet ネットワーク プラグイン ネットワークの詳細については、「[Azure Kubernetes Service (AKS) の独自の IP アドレス範囲で kubenet ネットワークを使用する](/azure/aks/configure-kubenet)」を参照してください。

## <a name="constraints-when-creating-a-custom-virtual-network"></a>カスタム仮想ネットワークを作成する際の制約

-  カスタム VNET は、Kubernetes クラスターのその他すべてのコンポーネントと同じサブスクリプションに存在する必要があります。
-  マスター ノードのプールとエージェント ノードのプールは、同じ仮想ネットワークに存在する必要があります。 同じ仮想ネットワーク内の異なるサブネットにノードをデプロイすることはできます。
-  Kubernetes クラスター サブネットに使用する IP 範囲は、カスタム仮想ネットワークの IP 範囲の空間内に存在する必要があります。「[IP アドレス ブロックを取得する](#get-the-ip-address-block)」を参照してください。

## <a name="create-custom-virtual-network"></a>カスタム仮想ネットワークを作成する

Azure Stack Hub インスタンスにカスタム仮想ネットワークが必要です。 詳細については、「[クイック スタート: Azure portal を使用した仮想ネットワークの作成](/azure/virtual-network/quick-create-portal)」を参照してください。

仮想ネットワークの新しいサブネットを作成します。 サブネットのリソース ID と IP アドレス範囲を取得する必要があります。 クラスターをデプロイするときに、API モデル内でリソース ID と範囲を使用します。

1. Azure Stack Hub インスタンスで Azure Stack Hub ユーザー ポータルを開きます。
2. **[すべてのリソース]** を選択します。
3. 検索ボックスに仮想ネットワークの名前を入力します。
4. サブネットを追加するには **[サブネット]**  >  **[+ サブネット]** を選択します。
5. CIDR 表記を使用して **[名前]** と **[アドレス範囲]** を追加します。 **[OK]** を選択します。
4. **[仮想ネットワーク]** ブレードで、 **[プロパティ]** を選択します。 **[リソース ID]** フィールドをコピーし、`/subnets/<nameofyoursubnect>` を追加します。 この値は、クラスターの API モデルの `vnetSubnetId` キーの値として使用します。 サブネットのリソース ID は、次の形式を使用します。<br>`/subscriptions/SUB_ID/resourceGroups/RG_NAME/providers/Microsoft.Network/virtualNetworks/VNET_NAME/subnets/SUBNET_NAME`

    ![仮想ネットワークのリソース ID](media/kubernetes-aks-engine-custom-vnet/virtual-network-id.png)

5. **[仮想ネットワーク]** ブレードで、 **[サブネット]** を選択します。 サブネット名を選択します (default など)。
    
    ![仮想ネットワークの CIDR ブロック](media/kubernetes-aks-engine-custom-vnet/virtual-network-cidr-block.png)
    
6. サブネット ブレードで、アドレス範囲と仮想ネットワークの CIDR ブロック (たとえば、`10.1.0.0 - 10.1.0.255 (256 addresses)` と `10.1.0.0/24`) を書き留めておきます。

## <a name="considerations-for-selecting-an-address-space"></a>アドレス空間を選択する場合の考慮事項

カスタム仮想ネットワークを作成するときに、ネットワークの IP アドレス空間と、すべてのサブネットの IP アドレス範囲を指定します。 Kubernetes クラスターで使用するアドレス空間と範囲を選択するときは、次の点を考慮してください。
-  アドレス空間が重複していると、IP アドレスの競合や通信エラーが発生するおそれがあります。 IP アドレスの重複のリスクを軽減するには、新しい仮想ネットワーク用に一意のアドレス空間を選択します。
-  `10.` と `172.` の範囲内のアドレス空間は、プライベート ネットワークによく使用されているため、既存のデータセンター インフラストラクチャで使用されている可能性があります。 Kubernetes アプリケーションでデータセンターのリソースが使用されている場合は、データセンターのアドレス空間とは異なるカスタム仮想ネットワークのアドレス空間を選択することで、競合のリスクを軽減します。
-  Kubernetes クラスターには専用サブネットを使用することをお勧めします。

## <a name="get-the-ip-address-block"></a>IP アドレス ブロックを取得する

AKS エンジンは、既存の仮想ネットワークへのデプロイをサポートしています。 既存のサブネットにデプロイする場合、クラスターでは連続する IP のブロックをエージェントに使用し、別のブロックをマスターに使用します。

2 つの値を設定する必要があります。 クラスターに予約する必要がある IP アドレスの数と、サブネットの IP 空間内の最初の連続する静的 IP を把握しておく必要があります。

複数のマスター ノードを使用する場合、AKS エンジンでは未使用の最大16個の IP アドレスの範囲が必要です。 クラスターは最大 5 つのマスターまで、マスターごとに 1 つの IP アドレスを使用します。 AKS エンジンでは、IP アドレス予約の余分として、最後のマスターの後の 10 個の IP アドレスが必要になります。 最後に、マスターと余分の予約の後に、ロード バランサーによって 1 つの IP アドレスが使用され、合計 16 になります。

IP アドレスのブロックを配置する場合、サブネットでは既存の IP アドレスの次の割り当てが必要になります。
 - 最初の 4つの IP アドレスと最後の IP アドレスは予約されており、どの Azure サブネットでも使用できません
 - 16 個の IP アドレスのバッファーをオープンにしたままにする必要があります。
 - クラスターの最初の IP の値は、IP の競合を避けるためにアドレス空間の最後の方にある必要があります。 可能な場合は、`firstConsecutiveStaticIP` プロパティに、サブネット内の使用可能な IP アドレス空間の *終わり* の近くにある IP アドレスを割り当てます。

次の例では、これらのさまざまな考慮事項によってサブネットの IP 範囲がどのように決まるかを確認できます。 これは 3 つのマスターを対象としています。 10.1.0.0/24 などの 256 個のアドレスを持つサブネットを使用している場合は、連続する最初の静的 IP アドレスを 207 に設定する必要があります。 次の表に、アドレスと考慮事項を示します。

| /24 サブネットの範囲 | Number | 注意 |
| --- | --- | --- |
| 10.1.0.0 - 10.1.0.3 | 4 | Azure サブネットで予約済み。 |
| **10.1.0.224** - 10.1.0.238 | 14 | AKS エンジンで定義されたクラスターの IP アドレスの数。<br><br> マスター 3 つにつき IP アドレス 3 個<br>余分の IP アドレス 10 個<br>ロード バランサー用の IP アドレス 1 個 |
| 10.1.0.239 - 10.1.0.255 | 16 | 16 個の IP アドレス バッファー。 |
| 10.1.0.256 | 1 | Azure サブネットで予約済み。 |

この例では、`firstConsecutiveStaticIP` プロパティは `10.1.0.224` になります。

より大きいサブネット (たとえば、/16 でアドレスが 6 万を超える) では、ネットワーク領域の最後に静的 IP の割り当てを設定することが実用的でない場合があります。 クラスターの静的 IP アドレス範囲を IP 空間の最初の 24 個のアドレスから離して設定し、アドレスを要求するときにクラスターが回復力を持てるようにします。

## <a name="update-the-api-model"></a>API モデルの更新

AKS エンジンからカスタム仮想ネットワークにクラスターをデプロイするために使用する API モデルを更新します。

**masterProfile** で、次の値を設定します。

| フィールド | 例 | 説明 |
| --- | --- | --- |
| vnetSubnetId | `/subscriptions/77e28b6a-582f-42b0-94d2-93b9eca60845/resourceGroups/MDBN-K8S/providers/Microsoft.Network/virtualNetworks/MDBN-K8S/subnets/default` | サブネットのリソース ID を指定します。  |
| firstConsecutiveStaticIP | 10.1.0.224 | `firstConsecutiveStaticIP` 構成プロパティに、サブネット内の使用可能な IP アドレス空間の *終わり* の近くにある IP アドレスを割り当てます。 `firstConsecutiveStaticIP` はマスター プールにのみ適用されます。 |

**agentPoolProfiles** で、次の値を設定します。

| フィールド | 例 | 説明 |
| --- | --- | --- |
| vnetSubnetId | `/subscriptions/77e28b6a-582f-42b0-94d2-93b9eca60845/resourceGroups/MDBN-K8S/providers/Microsoft.Network/virtualNetworks/MDBN-K8S/subnets/default` | サブネットの Azure Resource Manager パス ID を指定します。  |

**orchestratorProfile** から **kubernetesConfig** を見つけ、次の値を設定します。

| フィールド | 例 | 説明 |
| --- | --- | --- |
| clusterSubnet | `10.244.0.0/16` | ポッド ネットワーク インターフェイスに IP アドレスを割り当てるために使用される IP サブネット。 サブネットは、VNET アドレス空間にある必要があります。 Azure CNI が有効になっている場合、既定値は 10.240.0.0/12 です。 Azure CNI を使用しない場合、既定値は 10.244.0.0/16 です。  クラスター サブネット (ポッド ネットワーク) ネットワークで、VNET と同じ IP 空間を使用することはできません。 /24 ではなく /16 サブネットを使用します。 /24 を使用する場合、このサブネットは 1 つのノードにのみ割り当てられます。 IP 空間が不足するため、他のノードにポッド ネットワークが割り当てられません。そのため、これらをクラスターで準備することができなくなります。 |

次に例を示します。

```json
"masterProfile": {
  ...
  "vnetSubnetId": "/subscriptions/77e28b6a-582f-42b0-94d2-93b9eca60845/resourceGroups/MDBN-K8S/providers/Microsoft.Network/virtualNetworks/MDBN-K8S/subnets/default",
  "firstConsecutiveStaticIP": "10.1.0.224",
  ...
},
...
"agentPoolProfiles": [
  {
    ...
    "vnetSubnetId": "/subscriptions/77e28b6a-582f-42b0-94d2-93b9eca60845/resourceGroups/MDBN-K8S/providers/Microsoft.Network/virtualNetworks/MDBN-K8S/subnets/default",
    ...
  },
    ...
"kubernetesConfig": [
  {
    ...
    "clusterSubnet": "172.16.244.0/16",
    ...
  },

```

## <a name="deploy-your-cluster"></a>クラスターをデプロイする

API モデルに値を追加した後、AKS エンジンを使用して、`deploy` コマンドを使ってクライアント マシンからクラスターをデプロイできます。 説明については、「[Kubernetes クラスターのデプロイ](azure-stack-kubernetes-aks-engine-deploy-cluster.md#deploy-a-kubernetes-cluster)」を参照してください。

## <a name="set-the-route-table"></a>ルート テーブルを設定する

クラスターをデプロイした後、Azure Stack ユーザー ポータルで仮想ネットワークに戻ります。 サブネット ブレードで、ルート テーブルとネットワーク セキュリティ グループ (NSG) の両方を設定します。 たとえば、Azure CNI を使用していない場合は、`kubernetesConfig` API モデル構成オブジェクトの `networkPlugin`: `kubenet` です。 クラスターをカスタム仮想ネットワークに正常にデプロイしたら、クラスターのリソース グループにある **[ネットワーク]** ブレードからルート テーブル リソースの ID を取得します。

1. Azure Stack Hub インスタンスで Azure Stack Hub ユーザー ポータルを開きます。
2. **[すべてのリソース]** を選択します。
3. 検索ボックスに仮想ネットワークの名前を入力します。
4. **[サブネット]** を選択し、クラスターを含むサブネットの名前を選択します。
    
    ![ルート テーブルとネットワーク セキュリティ グループ](media/kubernetes-aks-engine-custom-vnet/virtual-network-route-table.png)
    
5. **[ルート テーブル]** を選択し、クラスターのルート テーブルを選択します。

> [!NOTE]  
> Kubernetes Windows クラスターのカスタム仮想ネットワークには [既知の問題](https://github.com/Azure/aks-engine/issues/371)があります。

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub 上の AKS エンジン](azure-stack-kubernetes-aks-engine-overview.md)を確認してください  
- 「[コンテナーに対する Azure Monitor の概要](/azure/azure-monitor/insights/container-insights-overview)」を読む
