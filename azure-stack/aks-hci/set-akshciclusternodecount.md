---
title: Set-AksHciClusterNodeCount
author: jessicaguan
description: Set-AksHciClusterNodeCount PowerShell コマンドは、クラスター内のコントロール プレーン ノードまたはワーカー ノードの数をスケーリングします。
ms.topic: reference
ms.date: 2/12/2021
ms.author: jeguan
ms.openlocfilehash: 1dff2962ee1997d6eaa696f6e0eeadf7ef2d59e1
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101874588"
---
# <a name="set-akshciclusternodecount"></a>Set-AksHciClusterNodeCount

## <a name="synopsis"></a>構文
クラスター内のコントロール プレーン ノードまたはワーカー ノードの数をスケーリングします。

## <a name="syntax"></a>構文

### <a name="scale-control-plane-nodes"></a>コントロール プレーン ノードをスケーリングします
```powershell
Set-AksHciClusterNodeCount -name <String>
                           -controlPlaneNodeCount <int> 
```

### <a name="scale-worker-nodes"></a>ワーカー ノードをスケーリングします
```powershell
Set-AksHciClusterNodeCount -name <String>
                           -linuxNodeCount <int>
                           -windowsNodeCount <int>
```

## <a name="description"></a>説明
クラスター内のコントロール プレーン ノードまたはワーカー ノードの数をスケーリングします。 コントロール プレーン ノードとワーカー ノードは、個別にスケーリングする必要があります。

## <a name="examples"></a>例

### <a name="scale-control-plane-nodes"></a>コントロール プレーン ノードをスケーリングします
```powershell
PS C:\> Set-AksHciClusterNodeCount -name myCluster -controlPlaneNodeCount 3
```

### <a name="scale-worker-nodes"></a>ワーカー ノードをスケーリングします
```powershell
PS C:\> Set-AksHciClusterNodeCount -name myCluster -linuxNodeCount 2 -windowsNodeCount 2
```

## <a name="parameters"></a>パラメーター

### <a name="-name"></a>-name
Kubernetes クラスターの英数字名。

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

### <a name="-controlplanenodecount"></a>-controlPlaneNodeCount
コントロール プレーン内のノード数。 既定値はありません。

```yaml
Type: System.Int32
Parameter Sets: (All)
Aliases:

Required: True
Position: Named
Default value: none
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-linuxnodecount"></a>-linuxNodeCount
Kubernetes クラスター内の Linux ノードの数。 既定値は 1 です。

```yaml
Type: System.Int32
Parameter Sets: (All)
Aliases:

Required: True
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-proxyservercertfile"></a>-proxyServerCertFile
プロキシ サーバーに対する認証に使用される証明書。
 
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

### <a name="-windowsnodecount"></a>-windowsNodeCount
Kubernetes クラスター内の Windows ノードの数。 既定値は 1 です。

```yaml
Type: System.Int32
Parameter Sets: (All)
Aliases:

Required: True
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```
