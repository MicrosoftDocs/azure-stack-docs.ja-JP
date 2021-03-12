---
title: Uninstall-AksHci
author: jessicaguan
description: Uninstall-AksHci PowerShell コマンドは、AKS on Azure Stack HCI を削除します。
ms.topic: reference
ms.date: 2/12/2021
ms.author: jeguan
ms.openlocfilehash: 89cd1bd48518f90dd4ce850e35e42c3106681823
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101874603"
---
# <a name="uninstall-akshci"></a>Uninstall-AksHci

## <a name="synopsis"></a>構文
Azure Kubernetes Service on Azure Stack HCI を削除します。

## <a name="syntax"></a>構文

```powershell
Uninstall-AksHci [-skipConfigCleanup]
```

## <a name="description"></a>説明
Azure Kubernetes Service on Azure Stack HCI を削除します。 

以前に Windows Admin Center を使用してデプロイしたクラスターで PowerShell コマンドを実行すると、PowerShell モジュールによって Windows Admin Center 構成ファイルが存在するかどうかがチェックされます。 Windows Admin Center によって、すべてのノードに Windows Admin Center 構成ファイルが配置されます。 

## <a name="example"></a>例

### <a name="example"></a>例
```powershell
PS C:\> Uninstall-AksHci
```

## <a name="parameters"></a>パラメーター

### <a name="-skipconfigcleanup"></a>-skipConfigCleanup
アンインストール後、構成の削除をスキップします。 このフラグを使用すると、ご自身の古い構成が保持されます。

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
