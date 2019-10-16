---
title: Azure Stack の VPN ゲートウェイ設定の構成 | Microsoft Docs
description: Azure Stack の VPN ゲートウェイ設定についてと、その構成について説明します。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: fa8d3adc-8f5a-4b4f-8227-4381cf952c56
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 10/03/2019
ms.author: sethm
ms.lastreviewed: 12/27/2018
ms.openlocfilehash: 650257a0bfe94741d00345f98b40fddd8d00cb44
ms.sourcegitcommit: b2d19e12a50195bb8925879ee75c186c9604f313
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/04/2019
ms.locfileid: "71961456"
---
# <a name="configure-vpn-gateway-settings-for-azure-stack"></a>Azure Stack の VPN ゲートウェイ設定の構成

*適用対象:Azure Stack 統合システムと Azure Stack Development Kit*

VPN ゲートウェイは、Azure Stack の仮想ネットワークとリモート VPN ゲートウェイの間で暗号化されたトラフィックを送信する、仮想ネットワーク ゲートウェイの一種です。 リモート VPN ゲートウェイは、Azure、データセンター内のデバイス、または別のサイト内のデバイスに配置することができます。 2 つのエンドポイント間にネットワーク接続がある場合は、2 つのネットワーク間にセキュリティで保護されたサイト間 (S2S) VPN 接続を確立できます。

VPN Gateway の接続は複数のリソースの構成に依存し、それぞれに構成可能な設定が含まれます。 この記事では、Resource Manager デプロイ モデルに作成される仮想ネットワークの VPN ゲートウェイに関するリソースと設定について説明します。 各接続ソリューションの説明とトポロジ ダイアグラムについては、「[About VPN Gateway for Azure Stack (Azure Stack の VPN ゲートウェイについて)](azure-stack-vpn-gateway-about-vpn-gateways.md)」を参照してください。

## <a name="vpn-gateway-settings"></a>VPN ゲートウェイの設定

### <a name="gateway-types"></a>ゲートウェイの種類

各 Azure Stack 仮想ネットワークでは、単一の仮想ネットワーク ゲートウェイがサポートされています。ゲートウェイの種類は、**Vpn** である必要があります。  このサポートは Azure とは異なります。Azure では、その他の種類もサポートされています。

仮想ネットワーク ゲートウェイを作成するときは、ゲートウェイの種類が構成に対して適切であることを確認する必要があります。 VPN ゲートウェイでは、次の例のように `-GatewayType Vpn` フラグが必要です。

```powershell
New-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg `
-Location 'West US' -IpConfigurations $gwipconfig -GatewayType Vpn `
-VpnType RouteBased
```

### <a name="gateway-skus"></a>ゲートウェイの SKU

仮想ネットワーク ゲートウェイを作成するときは、使用するゲートウェイの SKU を指定する必要があります。 ワークロード、スループット、機能、および SLA の種類に基づく要件を満たす SKU を選択します。

Azure Stack には、次の表に示す VPN ゲートウェイ SKU が用意されています。

| | VPN ゲートウェイのスループット |VPN ゲートウェイの IPsec トンネルの最大数 |
|-------|-------|-------|
|**Basic SKU**  | 100 Mbps  | 20    |
|**Standard SKU**   | 100 Mbps  | 20 |
|**高性能 SKU** | 200 Mbps | 10 |

### <a name="resizing-gateway-skus"></a>ゲートウェイ SKU のサイズ変更

Azure Stack は、サポートされているレガシ SKU 間での SKU のサイズ変更をサポートしていません。

同様に、Azure Stack では、サポートされているレガシ SKU (**Basic**、**Standard**、**HighPerformance**) から Azure でサポートされている新しい SKU (**VpnGw1**、**VpnGw2**、**VpnGw3**) へのサイズ変更もサポートされません。

### <a name="configure-the-gateway-sku"></a>ゲートウェイ SKU の構成

#### <a name="azure-stack-portal"></a>Azure Stack ポータル

Azure Stack ポータルを使用して Resource Manager の仮想ネットワーク ゲートウェイを作成する場合は、ドロップダウン リストを使用してゲートウェイ SKU を選択することができます。 オプションは、選択したゲートウェイの種類と VPN の種類に対応します。

#### <a name="powershell"></a>PowerShell

次の PowerShell の例では、`-GatewaySku` パラメーターが **Standard** として指定されています。

```powershell
New-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg `
-Location 'West US' -IpConfigurations $gwipconfig -GatewaySku Standard `
-GatewayType Vpn -VpnType RouteBased
```

### <a name="connection-types"></a>接続の種類

Resource Manager デプロイ モデルの各構成では、仮想ネットワーク ゲートウェイの接続の種類を指定する必要があります。 `-ConnectionType` に使用できる Resource Manager PowerShell 値は **IPsec** です。

次の PowerShell の例では、IPsec の接続の種類を必要とするサイト間接続が作成されます。

```powershell
New-AzureRmVirtualNetworkGatewayConnection -Name localtovon -ResourceGroupName testrg `
-Location 'West US' -VirtualNetworkGateway1 $gateway1 -LocalNetworkGateway2 $local `
-ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'
```

### <a name="vpn-types"></a>VPN の種類

VPN Gateway 構成に対して仮想ネットワーク ゲートウェイを作成する場合は、VPN の種類を指定する必要があります。 選択する VPN の種類は、作成する接続トポロジによって異なります。 VPN の種類は、使用しているハードウェアによっても異なる場合があります。 S2S 構成では、VPN デバイスが必要です。 一部の VPN デバイスでは、特定の VPN の種類のみがサポートされます。

> [!IMPORTANT]  
> 現時点の Azure Stack では、ルート ベースの VPN の種類のみがサポートされます。 お使いのデバイスがポリシー ベースの VPN のみに対応している場合、Azure Stack からこれらのデバイスへの接続はサポートされません。  
>
> さらに、IPSec/IKE ポリシーのカスタム構成はサポートされていないため、現時点で Azure Stack はルート ベース ゲートウェイに対するポリシー ベース トラフィック セレクターの使用をサポートしていません。

* **PolicyBased**: PolicyBased VPN では、パケットを暗号化し、オンプレミス ネットワークと Azure Stack VNet の間でアドレスのプレフィックスの組み合わせで構成された IPsec ポリシーに基づいて、IPsec トンネル経由でそのパケットを送信します。 VPN デバイスの構成では、通常このポリシー (またはトラフィック セレクター) がアクセス リストになります。

  >[!NOTE]
  >**PolicyBased** は Azure ではサポートされていますが、Azure Stack ではサポートされていません。

* **RouteBased**: ルートベースの VPN は、IP 転送やルーティング テーブルに構成されているルートを使用して、対応するトンネル インターフェイスにパケットを転送します。 その後、トンネル インターフェイスではトンネルの内部または外部でパケットを暗号化または復号します。 **RouteBased** VPN のポリシー (またはトラフィック セレクター) は、Any-to-Any (またはワイルドカードを使用して) 構成できます。 既定では変更できません。 **RouteBased** VPN の種類の値は **RouteBased** です。

次の PowerShell の例では、 `-VpnType` を **RouteBased**に指定しています。 ゲートウェイを作成する場合、`-VpnType` が自分の構成に対して適切であることを確認する必要があります。

```powershell
New-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg `
-Location 'West US' -IpConfigurations $gwipconfig `
-GatewayType Vpn -VpnType RouteBased
```

### <a name="gateway-requirements"></a>ゲートウェイの要件

次の表に、VPN ゲートウェイの要件を示します。

| |ポリシーベースの Basic VPN Gateway | ルートベースの Basic VPN Gateway | ルートベースの Standard VPN Gateway | ルートベースの High Performance VPN Gateway|
|--|--|--|--|--|
| **サイト間接続 (S2S 接続)** | サポートされていません | ルートベースの VPN 構成 | ルートベースの VPN 構成 | ルートベースの VPN 構成 |
| **認証方法**  | サポートされていません | S2S 接続用の事前共有キー  | S2S 接続用の事前共有キー  | S2S 接続用の事前共有キー  |
| **S2S 接続の最大数**  | サポートされていません | 20 | 20| 10|
|**アクティブ ルーティングのサポート (BGP)** | サポートされていません | サポートされていません | サポートされています | サポートされています |

### <a name="gateway-subnet"></a>ゲートウェイ サブネット

VPN ゲートウェイを作成する前に、ゲートウェイ サブネットを作成する必要があります。 ゲートウェイ サブネットには、仮想ネットワーク ゲートウェイの VM とサービスが使用する IP アドレスが含まれます。 仮想ネットワーク ゲートウェイを作成すると、ゲートウェイ VM はゲートウェイ サブネットにデプロイされ、必要な VPN ゲートウェイ設定で構成されます。 ゲートウェイ サブネットには、何も (追加の VM など) をデプロイしないでください。

>[!IMPORTANT]
>ゲートウェイ サブネットを正常に動作させるには、 **GatewaySubnet** という名前を付ける必要があります。 Azure Stack は、この名前を使用して、仮想ネットワーク ゲートウェイ VM およびサービスをデプロイするサブネットを識別します。

ゲートウェイ サブネットを作成するときに、サブネットに含まれる IP アドレスの数を指定します。 ゲートウェイ サブネット内の IP アドレスは、ゲートウェイ VM とゲートウェイ サービスに割り当てられます。 一部の構成では、他の構成よりも多くの IP アドレスを割り当てる必要があります。 作成する構成の指示を調べて、作成するゲートウェイ サブネットがその要件を満たしていることを確認してください。

さらに、今後の追加構成を処理できる十分な IP アドレスがゲートウェイ サブネットにあることを確認する必要があります。 /29 のような小さいゲートウェイ サブネットを作成できますが、/28 以上 (/28、/27、/26 など) のゲートウェイ サブネットを作成することをお勧めします。そうすれば、将来的に機能を追加する場合に、ゲートウェイを破棄し、ゲートウェイ サブネットを削除してから再作成して、より多くの IP アドレスを割り当てられるようにする必要がなくなります。

次の Resource Manager PowerShell の例では、**GatewaySubnet** という名前のゲートウェイ サブネットを示しています。 CIDR 表記で /27 を指定しています。これで既存のほとんどの構成で IP アドレスに十分対応できます。

```powershell
Add-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.0.3.0/27
```

> [!IMPORTANT]
> ゲートウェイ サブネットを使用する場合は、ゲートウェイ サブネットにネットワーク セキュリティ グループ (NSG) を関連付けないようにしてください。 このサブネットにネットワーク セキュリティ グループを関連付けると、VPN ゲートウェイが正常に動作しなくなることがあります。 ネットワーク セキュリティ グループの詳細については、[ネットワーク セキュリティ グループの概要](/azure/virtual-network/virtual-networks-nsg)に関するページを参照してください。

### <a name="local-network-gateways"></a>ローカル ネットワーク ゲートウェイ

Azure で VPN ゲートウェイ構成を作成する場合、多くのローカル ネットワーク ゲートウェイはオンプレミスの場所を表します。 Azure Stack では、Azure Stack の外にある任意のリモート VPN デバイスを表します。 このデバイスは、データセンター (またはリモート データセンター) 内の VPN デバイス、または Azure 内の VPN ゲートウェイなどになります。

ローカル ネットワーク ゲートウェイに名前と VPN デバイスのパブリック IP アドレスを指定し、オンプレミスの場所を示すアドレスのプレフィックスを指定します。 Azure によって、ネットワーク トラフィックの宛先アドレスのプレフィックスが参照され、ローカル ネットワーク ゲートウェイに指定した構成が確認されて、それに応じてパケットがルーティングされます。

この PowerShell の例では、新しいローカル ネットワーク ゲートウェイを作成します。

```powershell
New-AzureRmLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg `
-Location 'West US' -GatewayIpAddress '23.99.221.164' -AddressPrefix '10.5.51.0/24'
```

場合によっては、ローカル ネットワーク ゲートウェイの設定を変更する必要があります。たとえば、アドレスの範囲を追加または変更した場合や、VPN デバイスの IP アドレスが変更された場合などです。 詳細については、「[PowerShell を使用したローカル ネットワーク ゲートウェイの設定の変更](/azure/vpn-gateway/vpn-gateway-modify-local-network-gateway)」を参照してください。

## <a name="ipsecike-parameters"></a>IPsec/IKE パラメーター

Azure Stack で VPN 接続を設定する場合、両端で接続を構成する必要があります。 VPN ゲートウェイとして動作するスイッチやルーターなどのハードウェア デバイスと Azure Stack との間で VPN 接続を構成している場合、そのデバイスには追加の設定が必要になることがあります。

イニシエーターとレスポンダーの両方として複数のオファーをサポートしている Azure とは異なり、Azure Stack は、既定では 1 つのオファーだけをサポートします。 VPN デバイスを操作するために異なる IPSec/IKE 設定を使用する必要がある場合は、接続を手動で構成するために使用できる設定が他にもあります。 詳細については、「[サイト間 VPN 接続の IPsec/IKE ポリシーを構成する](azure-stack-vpn-s2s.md)」を参照してください。

### <a name="ike-phase-1-main-mode-parameters"></a>IKE フェーズ 1 (メイン モード) のパラメーター

| プロパティ              | 値|
|-|-|
| IKE のバージョン           | IKEv2 |
|Diffie-hellman グループ   | グループ 2 (1024 ビット) |
| 認証方法 | 事前共有キー |
|暗号化とハッシュ アルゴリズム | AES256、SHA256 |
|SA の有効期間 (時間)     | 28,800 秒|

### <a name="ike-phase-2-quick-mode-parameters"></a>IKE フェーズ 2 (クイック モード) のパラメーター

| プロパティ| 値|
|-|-|
|IKE のバージョン |IKEv2 |
|暗号化とハッシュ アルゴリズム (暗号化)     | GCMAES256|
|暗号化とハッシュ アルゴリズム (認証) | GCMAES256|
|SA の有効期間 (時間)  | 27,000 秒  |
|SA の有効期間 (キロバイト単位) | 33,553,408     |
|Perfect Forward Secrecy (PFS) |なし (**注 1** を参照) |
|Dead Peer Detection | サポートされています|  

"**注 1:** " 1807 より前のバージョンの Azure Stack では、Perfect Forward Secrecy (PFS) の値として PFS2048 を使用しています。

## <a name="next-steps"></a>次の手順

* [ExpressRoute を使用して接続する](../operator/azure-stack-connect-expressroute.md)
