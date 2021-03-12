---
title: Get-AksHciCluster
author: jessicaguan
description: Get-AksHciCluster PowerShell コマンドは、Azure Kubernetes Service ホストを含む Kubernetes マネージド クラスターを一覧表示します。
ms.topic: reference
ms.date: 2/12/2021
ms.author: jeguan
ms.openlocfilehash: 14d7f807ef04432e7ea8c3c57dbb4bb80c64731b
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101874512"
---
# <a name="get-akshcicluster"></a>Get-AksHciCluster

## <a name="synopsis"></a>構文
Azure Kubernetes Service ホストを含む Kubernetes マネージド クラスターを一覧表示します。

## <a name="syntax"></a>構文

```powershell
Get-AksHciCluster [-name <String>]
```

## <a name="description"></a>説明
Azure Kubernetes Service ホストを含む Kubernetes マネージド クラスターを一覧表示します。

## <a name="examples"></a>例

### <a name="list-all-kubernetes-clusters"></a>すべての Kubernetes クラスターを一覧表示します
```powershell
PS C:\> Get-AksHciCluster
```

## <a name="parameters"></a>パラメーター

### <a name="-name"></a>-name
ご利用のクラスターの名前。

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
