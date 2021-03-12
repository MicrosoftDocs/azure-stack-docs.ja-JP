---
title: Remove-AksHciCluster
description: Remove-AksHciCluster PowerShell コマンドは、マネージド Kubernetes クラスターを削除します。
author: jessicaguan
ms.topic: reference
ms.date: 2/12/2021
ms.author: jeguan
ms.openlocfilehash: 868aa8a739c69ad6e29d73192f91c9bda73a3195
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101874595"
---
# <a name="remove-akshcicluster"></a>Remove-AksHciCluster

## <a name="synopsis"></a>構文
マネージド Kubernetes クラスターを削除します。

## <a name="syntax"></a>構文

### <a name="delete-a-managed-kubernetes-cluster"></a>マネージド Kubernetes クラスターを削除します
```powershell
Remove-AksHciCluster -name 
                    [-force]   
```

## <a name="description"></a>説明
マネージド Kubernetes クラスターを削除します。

## <a name="examples"></a>例

### <a name="delete-an-existing-managed-kubernetes-cluster"></a>既存のマネージド Kubernetes クラスターを削除します
```powershell
PS C:\> Remove-AksHciCluster -name myCluster
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
