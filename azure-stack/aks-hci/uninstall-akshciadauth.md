---
title: Uninstall-AksHciAdAuth
author: jessicaguan
description: Uninstall-AksHciAdAuth PowerShell コマンドは、AD 認証がアンインストールします。
ms.topic: reference
ms.date: 2/12/2021
ms.author: jeguan
ms.openlocfilehash: 4239d9c268d57315584381e02fa121848395e683
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101874496"
---
# <a name="uninstall-akshciadauth"></a>Uninstall-AksHciAdAuth

## <a name="synopsis"></a>構文
Active Directory 認証をアンインストールします。

## <a name="syntax"></a>構文

```powershell
Uninstall-AksHciAdAuth -name <String>
```

## <a name="description"></a>説明
Active Directory 認証をアンインストールします。

## <a name="examples"></a>例

### <a name="example"></a>例
```powershell
PS C:\> Uninstall-AksHciAksHciAdAuth -name myCluster
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
