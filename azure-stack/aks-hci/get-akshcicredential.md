---
title: Get-AksHciCredential
author: jessicaguan
description: Get-AksHciCredential PowerShell コマンドは、kubectl を使用してご自身のクラスターにアクセスします。
ms.topic: reference
ms.date: 2/12/2021
ms.author: jeguan
ms.openlocfilehash: 50f99f6e5a64a1a0b687c0dfdc42b407b4106500
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101874568"
---
# <a name="get-akshcicredential"></a>Get-AksHciCredential

## <a name="synopsis"></a>構文
`kubectl` を使用してご自身のクラスターにアクセスします。 これにより、指定したクラスターの _kubeconfig_ ファイルが `kubectl` の既定の _kubeconfig_ ファイルとして使用されます。

## <a name="syntax"></a>構文

```powershell
Get-AksHciCredential -name <String>
                    [-outputLocation <String>]
                    [-adAuth]
```

## <a name="description"></a>説明
kubectl を使用してご自身のクラスターにアクセスします。

## <a name="examples"></a>例

### <a name="access-your-cluster-using-kubectl"></a>kubectl を使用してご自身のクラスターにアクセスします。
```powershell
PS C:\> Get-AksHciCredential -name myCluster
```

## <a name="parameters"></a>パラメーター

### <a name="-name"></a>-name
クラスターの名前です。

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

### <a name="-outputlocation"></a>-outputLocation
kubeconfig をダウンロードする場所。 既定値は `%USERPROFILE%\.kube` です。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: %USERPROFILE%\.kube
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-adauth"></a>-adAuth
このフラグを使用して、kubeconfig の Active Directory SSO バージョンを取得します。

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
