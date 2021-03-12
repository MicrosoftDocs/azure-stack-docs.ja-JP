---
title: Get-AksHciClusterUpgrades
author: jessicaguan
description: Get-AksHciClusterUpgrades PowerShell コマンドは、Azure Kubernetes Service クラスターの利用可能な Kubernetes アップグレードを取得します。
ms.topic: reference
ms.date: 2/12/2021
ms.author: jeguan
ms.openlocfilehash: 3d5f4a6b67345baaa8df9b9eccc26d0a0503dafd
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101874527"
---
# <a name="get-akshciclusterupgrades"></a>Get-AksHciClusterUpgrades

## <a name="synopsis"></a>構文
Azure Kubernetes Service クラスターの利用可能な Kubernetes アップグレードを取得します。

## <a name="syntax"></a>構文

```powershell
Get-AksHciClusterUpgrades -name <String>
                          
```

## <a name="description"></a>説明
Azure Kubernetes Service クラスターの利用可能な Kubernetes アップグレードを取得します。

## <a name="examples"></a>例

```powershell
PS C:\> Get-AksHciClusterUpgrades -name mycluster
```

## <a name="parameters"></a>パラメーター

### <a name="-name"></a>-name
お使いのクラスターの英数字名。

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
