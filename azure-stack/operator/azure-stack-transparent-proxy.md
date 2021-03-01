---
title: Azure Stack Hub 統合システムの透過プロキシ
description: Azure Stack Hub 統合システムの透過プロパティの概要。
author: PatAltimore
ms.topic: conceptual
ms.date: 01/25/2021
ms.author: patricka
ms.reviewer: sranthar
ms.lastreviewed: 01/25/2021
ms.openlocfilehash: 974f40364b4eed13bd7440b35596597312c98624
ms.sourcegitcommit: 283b1308142e668749345bf24b63d40172559509
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/05/2021
ms.locfileid: "99577373"
---
# <a name="transparent-proxy-for-azure-stack-hub"></a>Azure Stack Hub の透過プロキシ

透過プロキシ (インターセプト、インライン、または強制プロキシとも呼ばれます) は、特殊なクライアント構成を必要とすることなく、通常の通信をネットワーク レイヤーでインターセプトします。 クライアントがプロキシの存在を意識する必要はありません。

データセンターですべてのトラフィックがプロキシを使用する必要がある場合は、ネットワーク上のゾーン間でトラフィックを分離することにより、ポリシーに従ってすべてのトラフィックを処理するように透過プロキシを構成します。

## <a name="traffic-types"></a>トラフィックの種類

Azure Stack Hub からの送信トラフィックは、テナント トラフィックまたはインフラストラクチャ トラフィックのいずれかに分類されます。

テナント トラフィックは、仮想マシン、ロード バランサー、VPN ゲートウェイ、アプリ サービスなどを介してテナントによって生成されます。

インフラストラクチャ トラフィックは、ID、パッチと更新プログラム、使用状況メトリック、Marketplace シンジケーション、登録、ログ収集、Windows Defender などのインフラストラクチャ サービスに割り当てられたパブリック仮想 IP プールの最初の `/27` 範囲から生成されます。これらのサービスからのトラフィックは、[Azure エンドポイント](azure-stack-integrate-endpoints.md#ports-and-urls-outbound)にルーティングされます。 プロキシによって変更されたトラフィックまたは TLS/SSL で傍受されたトラフィックは、Azure に受け入れられません。 これが、Azure Stack Hub がネイティブ プロキシ構成をサポートしていない理由です。

透過プロキシを構成する場合、プロキシを介してすべての送信トラフィックを送信するか、インフラストラクチャ トラフィックのみを送信するかを選択できます。

## <a name="partner-integration"></a>パートナー統合

Microsoft は、業界の主要なプロキシ ベンダーと提携し、透過プロキシ構成を使用した Azure Stack Hub のユース ケース シナリオを検証しています。 HA プロキシを使用した Azure Stack Hub ネットワーク構成の例を次の図に示します。 外部プロキシ デバイスは、境界デバイスの前面に配置する必要があります。

![境界デバイスの前面にプロキシがあるネットワーク図](./media/azure-stack-transparent-proxy/proxy.svg)

 また、境界デバイスは、次のいずれかの方法で Azure Stack Hub からのトラフィックをルーティングするように構成する必要があります。
- Azure Stack Hub からのすべての送信トラフィックをプロキシ デバイスにルーティングする
- Azure Stack Hub 仮想 IP プールの最初の `/27` 範囲からのすべての送信トラフィックを、ポリシーベースのルーティングを介してプロキシ デバイスにルーティングします。  

境界構成のサンプルについては、この記事の「[境界構成の例](#example-border-configuration)」セクションを参照してください。

Azure Stack Hub で検証された透過プロキシ構成については、次のドキュメントを参照してください。 

- [Check Point Security Gateway の透過プロキシを構成する](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk171559)
- [Sophos XG ファイアウォールの透過プロキシを構成する](https://community.sophos.com/xg-firewall/f/recommended-reads/124106/xg-firewall-integration-with-azure-stack-hub)

Azure Stack Hub からの送信トラフィックが明示的なプロキシを経由する必要があるシナリオの場合、Sophos および Checkpoint デバイスにはデュアル モード機能が用意されています。これにより、特定の範囲のトラフィックには透過モードを使用し、他の範囲には明示モードを使用するように構成できます。 この機能を使用すると、インフラストラクチャ トラフィックのみを透過プロキシを経由して送信し、すべてのテナント トラフィックを明示モードを使用して送信するようにこれらのプロキシ デバイスを構成できます。

> [!IMPORTANT]
> SSL トラフィックのインターセプトはサポートされていないため、エンドポイントにアクセスするとサービス エラーが発生する場合があります。 ID に必要なエンドポイントとの通信に対してサポートされる最大タイムアウトは 60 秒で、再試行は 3 回です。 詳細については、「[Azure Stack Hub ファイアウォールの統合](azure-stack-firewall.md#ssl-interception)」を参照してください。

## <a name="example-border-configuration"></a>境界構成の例

このソリューションは、アクセス制御リスト (ACL) によって実装された管理者定義の一連の条件を使用するポリシー ベース ルーティング (PBR) に基づいています。 ACL により、宛先 IP アドレスのみに基づく通常のルーティングではなく、ルートマップに実装されたプロキシ デバイスの次ホップ IP に送信されたトラフィックが分類されます。 ポート 80 および 443 の特定のインフラストラクチャ ネットワーク トラフィックは、境界デバイスから透過プロキシ展開にルーティングされます。 透過プロキシにより URL フィルター処理が実行され、"*許可されていない*" トラフィックはドロップされます。

Cisco Nexus 9508 シャーシ用の構成サンプルを次に示します。 

このシナリオの場合、インターネットへのアクセスを必要とするソース インフラストラクチャ ネットワークは次のとおりです。

- パブリック VIP - 最初の /27
- インフラストラクチャ ネットワーク - 最後の /27
- BMC ネットワーク - 最後の /27

このシナリオの場合、次のサブネットはポリシー ベース ルーティング (PBR) 処理を受けます。

| ネットワーク | IP 範囲 | PBR 処理を受けるサブネット
|---------|----------|-------------------------------
| パブリック仮想 IP プール | 172.21.107.0/27 の最初の /27 | 172.21.107.0/27 = 172.21.107.1 から 172.21.107.30
| インフラストラクチャ ネットワーク | 172.21.7.0/24 の最後の /27 | 172.21.7.224/27 = 172.21.7.225 から 172.21.7.254
| BMC ネットワーク | 10.60.32.128/26 の最後の /27 | 10.60.32.160/27 = 10.60.32.161 から 10.60.32.190

### <a name="configure-border-device"></a>境界デバイスを構成する

`feature pbr` コマンドを入力して PBR を有効にします。 

```
****************************************************************************
PBR Configuration for Cisco Nexus 9508 Chassis
PBR Enivronment configured to use VRF08
The test rack has is a 4-node Azure Stack stamp with 2x TOR switches and 1x BMC switch. Each TOR switch 
has a single uplink to the Nexus 9508 chassis using BGP for routing. In this example the test rack 
is in it's own VRF (VRF08)
****************************************************************************
!
feature pbr
!

<Create VLANs that the proxy devices will use for inside and outside connectivity>

!
VLAN 801
name PBR_Proxy_VRF08_Inside
VLAN 802
name PBR_Proxy_VRF08_Outside
!
interface vlan 801
description PBR_Proxy_VRF08_Inside
no shutdown
mtu 9216
vrf member VRF08
no ip redirects
ip address 10.60.3.1/29
!
interface vlan 802
description PBR_Proxy_VRF08_Outside
no shutdown
mtu 9216
vrf member VRF08
no ip redirects
ip address 10.60.3.33/28
!
!
ip access-list PERMITTED_TO_PROXY_ENV1
100 permit tcp 172.21.107.0/27 any eq www
110 permit tcp 172.21.107.0/27 any eq 443
120 permit tcp 172.21.7.224/27 any eq www
130 permit tcp 172.21.7.224/27 any eq 443
140 permit tcp 10.60.32.160/27 any eq www
150 permit tcp 10.60.32.160/27 any eq 443
!
!
route-map TRAFFIC_TO_PROXY_ENV1 pbr-statistics
route-map TRAFFIC_TO_PROXY_ENV1 permit 10
  match ip address PERMITTED_TO_PROXY_ENV1
  set ip next-hop 10.60.3.34 10.60.3.35
!
!
interface Ethernet1/1
  description DownLink to TOR-1:TeGig1/0/47
  mtu 9100
  logging event port link-status
  vrf member VRF08
  ip address 192.168.32.193/30
  ip policy route-map TRAFFIC_TO_PROXY_ENV1
  no shutdown
!
interface Ethernet2/1
  description DownLink to TOR-2:TeGig1/0/48
  mtu 9100
  logging event port link-status
  vrf member VRF08
  ip address 192.168.32.205/30
  ip policy route-map TRAFFIC_TO_PROXY_ENV1
  no shutdown
!

<Interface configuration for inside/outside connections to proxy devices. In this example there are 2 firewalls>

!
interface Ethernet1/41
  description management interface for Firewall-1
  switchport
  switchport access vlan 801
  no shutdown
!
interface Ethernet1/42
  description Proxy interface for Firewall-1
  switchport
  switchport access vlan 802
  no shutdown
!
interface Ethernet2/41
  description management interface for Firewall-2
  switchport
  switchport access vlan 801
  no shutdown
!
interface Ethernet2/42
  description Proxy interface for Firewall-2
  switchport
  switchport access vlan 802
  no shutdown
!

<BGP network statements for VLAN 801-802 subnets and neighbor statements for R023171A-TOR-1/R023171A-TOR-2> 

!
router bgp 65000
!
vrf VRF08
address-family ipv4 unicast
network 10.60.3.0/29
network 10.60.3.32/28
!
neighbor 192.168.32.194
  remote-as 65001
  description LinkTo 65001:R023171A-TOR-1:TeGig1/0/47
  address-family ipv4 unicast
    maximum-prefix 12000 warning-only
neighbor 192.168.32.206
  remote-as 65001
  description LinkTo 65001:R023171A-TOR-2:TeGig1/0/48
  address-family ipv4 unicast
    maximum-prefix 12000 warning-only
!
!
```

PBR 処理を受けるトラフィックを識別するために使用される新しい ACL を作成します。 このトラフィックは、この例で詳しく説明されているように、プロキシ サービスを取得するテスト ラック内のホストおよびサブネットからの Web トラフィック (HTTP ポート 80 および HTTPS ポート 443) です。 たとえば、ACL 名は **PERMITTED_TO_PROXY_ENV1** です。

```
ip access-list PERMITTED_TO_PROXY_ENV1
100 permit tcp 172.21.107.0/27 any eq www <<HTTP traffic from CL04 Public Admin VIPs leaving test rack>>
110 permit tcp 172.21.107.0/27 any eq 443 <<HTTPS traffic from CL04 Public Admin VIPs leaving test rack>>
120 permit tcp 172.21.7.224/27 any eq www <<HTTP traffic from CL04 INF-pub-adm leaving test rack>>
130 permit tcp 172.21.7.224/27 any eq 443 <<HTTPS traffic from CL04 INF-pub-adm leaving test rack>>
140 permit tcp 10.60.32.160/27 any eq www <<HTTP traffic from DVM and HLH leaving test rack>>
150 permit tcp 10.60.32.160/27 any eq 443 <<HTTPS traffic from DVM and HLH leaving test rack>>
```

PBR 機能の中核は、**TRAFFIC_TO_PROXY_ENV1** ルートマップによって実装されます。 **pbr-statistics** オプションが追加され、ポリシー一致統計情報を表示して、PBR 転送を実行するパケットとしないパケットの数を確認できるようになりました。 ルートマップ シーケンス 10 を使用すると、ACL **PERMITTED_TO_PROXY_ENV1** 条件を満たすトラフィックへの PBR 処理が許可されます。 そのトラフィックは、`10.60.3.34` および `10.60.3.35` の次ホップ IP アドレスに転送されます。これらは、この構成例のプライマリおよびセカンダリ プロキシ デバイスの VIP です。

```
!
route-map TRAFFIC_TO_PROXY_ENV1 pbr-statistics
route-map TRAFFIC_TO_PROXY_ENV1 permit 10
  match ip address PERMITTED_TO_PROXY_ENV1
  set ip next-hop 10.60.3.34 10.60.3.35
```

ACL は、**TRAFFIC_TO_PROXY_ENV1** ルートマップの一致条件として使用されます。 トラフィックが **PERMITTED_TO_PROXY_ENV1** ACL に一致すると、PBR により通常のルーティング テーブルが上書きされ、代わりに一連にある IP 次ホップにトラフィックが転送されます。
 
**TRAFFIC_TO_PROXY_ENV1** PBR ポリシーは、CL04 ホストとパブリック VIP から、またテスト ラック内の HLH と DVM から境界デバイスに送信されるトラフィックに適用されます。

## <a name="next-steps"></a>次のステップ

ファイアウォールの統合の詳細については、「[Azure Stack Hub ファイアウォールの統合](azure-stack-firewall.md)」を参照してください。
