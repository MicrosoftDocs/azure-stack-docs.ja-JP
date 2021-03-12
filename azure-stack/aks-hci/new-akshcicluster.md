---
title: New-AksHciCluster
author: jessicaguan
description: New-AksHciCluster PowerShell コマンドは、新しい マネージド Kubernetes クラスターを作成します。
ms.topic: reference
ms.date: 2/12/2021
ms.author: jeguan
ms.openlocfilehash: d4b374ca093c60018d1dc68c3d5a39db68456298
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101874536"
---
# <a name="new-akshcicluster"></a>New-AksHciCluster

## <a name="synopsis"></a>構文
新しいマネージド Kubernetes クラスターを作成します。

## <a name="syntax"></a>構文

```powershell
New-AksHciCluster -name <String>
                 [-kubernetesVersion <String>]
                 [-controlPlaneNodeCount <int>]
                 [-linuxNodeCount <int>]
                 [-windowsNodeCount <int>]
                 [-controlPlaneVmSize <VmSize>]
                 [-loadBalancerVmSize <VmSize>]
                 [-linuxNodeVmSize <VmSize>]
                 [-windowsNodeVmSize <VmSize>]
                 [-vnet <Virtual Network>]
                 [-primaryNetworkPlugin <Network Plugin>]   
                 [-enableAdAuth]
                 [-enableSecretsEncryption]          
```

## <a name="description"></a>説明

新しい Azure Kubernetes Service on Azure Stack HCI クラスターを作成します。

## <a name="examples"></a>例

### <a name="new-aks-hci-cluster-with-required-params"></a>必要なパラメーターが指定された新しい AKS-HCI クラスター。

```powershell
PS C:\> New-AksHciCluster -name myCluster
```

## <a name="parameters"></a>パラメーター

### <a name="-name"></a>-name
Kubernetes クラスターの名前。

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

### <a name="-kubernetesversion"></a>-kubernetesVersion
デプロイする Kubernetes のバージョン。 既定値は最新バージョンです。 使用できるバージョンの一覧を取得するには、`Get-AksHciKubernetesVersion` を実行します。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: v1.18.8 or later
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-controlplanenodecount"></a>-controlPlaneNodeCount
コントロール プレーン内のノード数。 既定値は 1 です。

```yaml
Type: System.Int32
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: 1
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-linuxnodecount"></a>-linuxNodeCount
Kubernetes クラスター内の Linux ノードの数。 既定値は 1 です。

```yaml
Type: System.Int32
Parameter Sets: (All)
Aliases: 

Required: False
Position: Named
Default value: 1
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-windowsnodecount"></a>-windowsNodeCount
Kubernetes クラスター内の Windows ノードの数。 既定値は 0 です。 Windows ノードをデプロイできるのは、Kubernetes v1.18.8 以上を実行している場合のみです。

```yaml
Type: System.Int32
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: 0
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-controlplanevmsize"></a>-controlPlaneVmSize
コントロール プレーン VM のサイズ。 既定値は Standard_A4_V2 です。 使用できる VM サイズの一覧を取得するには、`Get-AksHciVmSize` を実行します。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: Standard_A4_V2
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-loadbalancervmsize"></a>-loadBalancerVmSize
ロード バランサー VM のサイズ。 既定値は Standard_A4_V2 です。 使用できる VM サイズの一覧を取得するには、`Get-AksHciVmSize` を実行します。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: Standard_A4_V2
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-linuxnodevmsize"></a>-linuxNodeVmSize
Linux ノード VM のサイズ。 既定値は Standard_K8S3_v1 です。 使用できる VM サイズの一覧を取得するには、`Get-AksHciVmSize` を実行します。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: Standard_K8S3_v1
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-windowsvmsize"></a>-windowsVmSize
Windows ノード VM のサイズ。 既定値は Standard_K8S3_v1 です。 使用できる VM サイズの一覧を取得するには、`Get-AksHciVmSize` を実行します。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: Standard_K8S3_v1
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-vnet"></a>-vnet
New-AksHciNetworkSetting コマンドを使用して作成された AksHciNetworkSetting オブジェクトの名前。

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

### <a name="-primarynetworkplugin"></a>-primaryNetworkPlugin
デプロイで使用されるネットワーク プラグイン。 このパラメーターは、`flannel` または `calico` のいずれかを受け入れます。 Calico は Linux 専用クラスターでのみ使用できます。 お使いのクラスターに Calico を選択した場合、Windows ノードの追加は許可されません。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: flannel
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-enableadauth"></a>-enableADAuth
このフラグを使用して、お使いの Kubernetes クラスター内で Active Directory を有効にします。

```yaml
Type: System.Management.Automation.SwitchParameter
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-enablesecretsencryption"></a>-enableSecretsEncryption
このフラグを使用して、お使いの Kubernetes シークレット内で暗号化を有効にします。

```yaml
Type: System.Management.Automation.SwitchParameter
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```
