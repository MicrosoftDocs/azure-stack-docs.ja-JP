---
title: New-AksHciNetworkSetting
author: jessicaguan
description: New-AksHciNetworkSetting PowerShell コマンドは、新しい仮想ネットワークに対してオブジェクトを作成します。
ms.topic: reference
ms.date: 2/12/2021
ms.author: jeguan
ms.openlocfilehash: 5babbe99254b23d642696b7ce7c5b865105bd906
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101874503"
---
# <a name="new-akshcinetworksetting"></a>New-AksHciNetworkSetting

## <a name="synopsis"></a>構文
新しい仮想ネットワークに対してオブジェクトを作成します。

## <a name="syntax"></a>構文
```powershell
New-AksHciNetworkSetting -vnetName <String>
                         -gateway <String>
                         -dnsServers <String[]>
                         -ipAddressPrefix <String>
                         -vipPoolStart <IP address>
                         -vipPoolEnd <IP address>
                         -k8sNodeIpPoolStart <IP address>
                         -k8sNodeIpPoolEnd <IP address>
                        [-vlanID <int>]
                    
```

## <a name="description"></a>説明
仮想ネットワークを作成し、DHCP または静的 IP アドレスを、すべての Kubernetes クラスター内のノード用のコントロール プレーン、ロード バランサー、エージェント エンドポイント、および静的 IP 範囲に対して設定します。 このコマンドレットは、構成手順の後半で使用できる VirtualNetwork オブジェクトを返します。

## <a name="examples"></a>例

### <a name="deploy-with-a-static-ip-environment"></a>静的 IP 環境でデプロイする

```powershell
PS C:\> $vnet = New-AksHciNetworkSetting -vnetName "External" -k8sNodeIpPoolStart "172.16.10.0" -k8sNodeIpPoolEnd "172.16.10.255" -vipPoolStart "172.16.255.0" -vipPoolEnd "172.16.255.254" -ipAddressPrefix "172.16.0.0/16" -gateway "172.16.0.1" -dnsServers "172.16.0.1" 
PS C:\> Set-AksHciConfig -imageDir c:\clusterstorage\volume1\Images -cloudConfigLocation c:\clusterstorage\volume1\Config -vnet $vnet -enableDiagnosticData -cloudservicecidr "172.16.10.10/16"
```

> [!NOTE]
> この例のコマンドで指定した値は、ご利用の環境に合わせてカスタマイズする必要があります。

### <a name="deploy-with-a-dhcp-environment"></a>DHCP 環境でデプロイする

```powershell
PS C:\> $vnet = New-AksHciNetworkSetting -vnetName "External" -vipPoolStart "172.16.255.0" -vipPoolEnd "172.16.255.254" 
```

```powershell
PS C:\> Set-AksHciConfig -imageDir c:\clusterstorage\volume1\Images -cloudConfigLocation c:\clusterstorage\volume1\Config -vnet $vnet -enableDiagnosticData"
```

## <a name="parameters"></a>パラメーター

### <a name="-vnetname"></a>-vnetName
お使いの外部スイッチの名前。 使用可能なスイッチの名前の一覧を取得するには、コマンド `Get-VMSwitch` を実行します。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: True
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-gateway"></a>-gateway
サブネットの既定のゲートウェイの IP アドレス。

```yaml
Type: System.String
Parameter Sets: (StaticIP)
Aliases:

Required: False (This is required when creating a network with a static IP.)
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-dnsservers"></a>-dnsServers
静的 IP を使用してネットワークを作成するときに必要です。 サブネットに使用する DNS サーバーを指す IP アドレスの配列。 最小 1 台、最大 3 台のサーバーを指定できます。 例: "8.8.8.8"、"192.168.1.1"

```yaml
Type: System.String[]
Parameter Sets: (StaticIP)
Aliases:

Required: False (This is required when creating a network with a static IP.)
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-ipaddressprefix"></a>-ipAddressPrefix
静的 IP の割り当てに使用するアドレス プレフィックス。

```yaml
Type: System.String
Parameter Sets: (StaticIP)
Aliases:

Required: True
Position: Named
Default value: external
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-vippoolstart"></a>-vipPoolStart
VIP プールの開始 IP アドレス。 アドレスは、DHCP サーバーによって提供される範囲内、またはサブネット CIDR 内で提供される範囲内である必要があります。 VIP プール内の IP アドレスは、API サーバーおよび Kubernetes サービスに使用されます。 DHCP を使用している場合は、ご自身の仮想 IP アドレスが DHCP IP 予約に含まれることを確認します。 静的 IP を使用している場合は、ご自身の仮想 IP が同じサブネットに含まれることを確認します。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: True
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-vippoolend"></a>-vipPoolEnd
VIP プールの終了 IP アドレス。 アドレスは、DHCP サーバーによって提供される範囲内、またはサブネット CIDR 内で提供される範囲内である必要があります。 VIP プール内の IP アドレスは、API サーバーおよび Kubernetes サービスに使用されます。 DHCP を使用している場合は、ご自身の仮想 IP アドレスが DHCP IP 予約に含まれることを確認します。 静的 IP を使用している場合は、ご自身の仮想 IP が同じサブネットに含まれることを確認します。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: True
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-k8snodeippoolstart"></a>-k8sNodeIpPoolStart
VM プールの開始 IP アドレス。 アドレスは、サブネットの範囲内である必要があります。 これは、静的 IP デプロイに必要です。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-k8snodeippoolend"></a>-k8sNodeIpPoolEnd
VM プールの終了 IP アドレス。 アドレスは、サブネットの範囲内である必要があります。 これは、静的 IP デプロイに必要です。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-vlanid"></a>-vlanID
指定されたネットワークに対する vLAN ID。 省略した場合、ネットワークはタグ付けされません。

```yaml
Type: System.Int32
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

<!--- ### -macPoolName
The name of the MAC address pool that you wish to use for the Azure Kubernetes Service host VM. The pool will be created with the New-AksHciMacPoolSetting command.

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```
--->
