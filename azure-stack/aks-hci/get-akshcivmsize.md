---
title: Get-AksHciVmSize
author: jessicaguan
description: Get-AksHciVmSize PowerShell コマンドは、サポートされている VM サイズを一覧表示します。
ms.topic: reference
ms.date: 2/12/2021
ms.author: jeguan
ms.openlocfilehash: 874da946a58ce8b6b8ec28edf298ab608ca6fda0
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101874556"
---
# <a name="get-akshcivmsize"></a>Get-AksHciVmSize

## <a name="synopsis"></a>構文
サポートされる VM のサイズを一覧表示します。

## <a name="syntax"></a>構文

```powershell
Get-AksHciVmSize
```

## <a name="description"></a>説明
サポートされる VM のサイズを一覧表示します。

## <a name="examples"></a>例

### <a name="example"></a>例
```powershell
PS C:\> Get-AksHciVmSize
```

**出力:**
```
VmSize  CPU MemoryGB
------  --- --------
Default 4   4
Standard_A2_v2 2 4
Standard_A4_v2 4 8
Standard_D2s_v3 2 8
Standard_D4s_v3 4 16
Standard_D8s_v3 8 32
Standard_D16s_v3 16 64
Standard_D32s_v3 32 128
Standard_DS2_v2 2 7
Standard_DS3_v2 2 14
Standard_DS4_v2 8 28
Standard_DS5_v2 16 56
Standard_DS13_v2 8 56
Standard_K8S_v1 4 2
Standard_K8S2_v1 2 2
Standard_K8S3_v1 4 6
``` 
