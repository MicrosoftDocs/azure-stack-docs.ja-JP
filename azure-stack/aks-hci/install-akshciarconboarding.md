---
title: Install-AksHciArcOnboarding
author: jessicaguan
description: Install-AksHciArcOnboarding PowerShell コマンドは、AKS on Azure Stack HCI ワークロード クラスターを Azure Arc for Kubernetes に接続します。
ms.topic: reference
ms.date: 2/12/2021
ms.author: jeguan
ms.openlocfilehash: a366e553ac76882a7f45a9f25424cabbb4402319
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101874540"
---
# <a name="install-akshciarconboarding"></a>Install-AksHciArcOnboarding

## <a name="synopsis"></a>構文
AKS on Azure Stack HCI ワークロード クラスターを Azure Arc for Kubernetes に接続します。

## <a name="syntax"></a>構文

```powershell
Install-AksHciArcOnboarding -Name <String> 
                            -tenantId <String>
                            -subscriptionId <String> 
                            -resourceGroup <String>
                            -clientId <String>
                            -clientSecret <String>
                            [-location <String>]
```

## <a name="description"></a>説明
AKS on Azure Stack HCI ワークロード クラスターを Azure Arc for Kubernetes に接続します。

## <a name="examples"></a>例

### <a name="connect-an-aks-on-azure-stack-hci-cluster-to-azure-arc-for-kubernetes-with-required-parameters"></a>AKS on Azure Stack HCI クラスターを、必要なパラメーターが指定された Azure Arc for Kubernetes に接続します

```PowerShell
Install-AksHciArcOnboarding -name "myCluster" -resourcegroup "myResourceGroup" -subscriptionid "57ac26cf-a9f0-4908-b300-9a4e9a0fb205"  -clientid "22cc2695-54b9-49c1-9a73-2269592103d8" -clientsecret "09d3a928-b223-4dfe-80e8-fed13baa3b3d" -tenantid "72f988bf-86f1-41af-91ab-2d7cd011db47"
```

### <a name="connect-an-aks-on-azure-stack-hci-cluster-to-azure-arc-for-kubernetes-with-all-parameters"></a>AKS on Azure Stack HCI クラスターを、すべてのパラメーターが指定された Azure Arc for Kubernetes に接続します

```PowerShell
Install-AksHciArcOnboarding -name "myCluster" -resourcegroup "myResourceGroup" -location "eastus" -subscriptionid "57ac26cf-a9f0-4908-b300-9a4e9a0fb205"  -clientid "22cc2695-54b9-49c1-9a73-2269592103d8" -clientsecret "09d3a928-b223-4dfe-80e8-fed13baa3b3d" -tenantid "72f988bf-86f1-41af-91ab-2d7cd011db47"
```

## <a name="parameters"></a>パラメーター

### <a name="-name"></a>-Name
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

### <a name="-tenantid"></a>-tenantId
ご自身の Azure サービス プリンシパルのテナント ID。

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

### <a name="-subscriptionid"></a>-subscriptionId
ご自身の Azure アカウントのサブスクリプション ID。

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

### <a name="-resourcegroup"></a>-resourceGroup
Azure リソース グループの名前

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

### <a name="-clientid"></a>-clientId
ご自身の Azure サービス プリンシパルのクライアント ID またはアプリ ID

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

### <a name="-clientsecret"></a>-clientSecret
ご自身の Azure サービス プリンシパルのクライアント シークレットまたはパスワード

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

### <a name="-location"></a>-location
ご自身の Azure リソースの場所または Azure リージョン。 既定値は、ご自身の Azure リソース グループの場所です。 Azure Arc for Kubernetes の場合、場所として指定できるのは eastus または westeurope のみです。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: Azure resource group's location
Accept pipeline input: False
Accept wildcard characters: False
```
