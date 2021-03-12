---
title: Get-AksHciKubernetesVersion
author: jessicaguan
description: Get-AksHciKubernetesVersion PowerShell コマンドは、マネージド Kubernetes クラスターを作成するための利用可能なバージョンを一覧表示します。
ms.topic: reference
ms.date: 2/12/2021
ms.author: jeguan
ms.openlocfilehash: 76b87c1ddef2943181d30d43cf0bc06da644b110
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101874567"
---
# <a name="get-akshcikubernetesversion"></a>Get-AksHciKubernetesVersion

## <a name="synopsis"></a>構文
マネージド Kubernetes クラスターを作成するための利用可能なバージョンを一覧表示します。

## <a name="syntax"></a>構文

```powershell
Get-AksHciKubernetesVersion
```

## <a name="description"></a>説明
マネージド Kubernetes クラスターを作成するための利用可能なバージョンを一覧表示します。

## <a name="examples"></a>例

### <a name="example"></a>例 
```powershell
PS C:\> Get-AksHciKubernetesVersion
```

```Output
Linux {v1.16.10, v1.16.15, v1.17.11, v1.17.13...}
Windows {v1.18.8, v1.18.10}
```
