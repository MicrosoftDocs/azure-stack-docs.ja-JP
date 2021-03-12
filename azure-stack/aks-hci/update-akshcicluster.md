---
title: Update-AksHciCluster
author: jessicaguan
description: Update-AksHciCluster PowerShell コマンドは、マネージド Kubernetes クラスターを新しい Kubernetes または OS バージョンに更新します。
ms.topic: reference
ms.date: 2/12/2021
ms.author: jeguan
ms.openlocfilehash: a8fc5c1e87ed78891333af7a3a998767110428ea
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101874499"
---
# <a name="update-akshcicluster"></a>Update-AksHciCluster

## <a name="synopsis"></a>構文
マネージド Kubernetes クラスターを新しい Kubernetes または OS バージョンに更新します。

## <a name="syntax"></a>構文

```powershell
Update-AksHciCluster -name <String>
                    [-kubernetesVersion <String>]
                    [-operatingSystem]
```

## <a name="description"></a>説明
マネージド Kubernetes クラスターを新しい Kubernetes または OS バージョンに更新します。

## <a name="examples"></a>例

### <a name="example"></a>例
```powershell
PS C:\> Update-AksHciCluster -clusterName myCluster -kubernetesVersion v1.18.8 -operatingSystem
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
アップグレード先の Kubernetes のバージョン。

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

### <a name="-operatingsystem"></a>-operatingSystem
次のバージョンのオペレーティング システムに更新する場合は、このフラグを使用します。

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

