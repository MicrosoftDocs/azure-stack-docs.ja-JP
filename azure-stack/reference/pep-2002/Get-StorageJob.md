---
title: Azure Stack Hub の Get-StorageJob 特権エンドポイント
description: PowerShell Azure Stack 特権エンドポイントのリファレンス - Get-StorageJob
author: mattbriggs
ms.topic: reference
ms.date: 04/27/2020
ms.author: mabrigg
ms.reviewer: fiseraci
ms.lastreviewed: 04/27/2020
ms.openlocfilehash: fad5223778bb49b20d3af86f781c6864c9bdc8c9
ms.sourcegitcommit: 54f98b666bea9226c78f26dc255ddbdda539565f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/29/2020
ms.locfileid: "82563537"
---
# <a name="get-storagejob"></a>Get-StorageJob

## <a name="syntax"></a>構文

```
Get-StorageJob [-CimSession <Object>] [-Name <Object>] [-VirtualDisk <Object>] [-JobState <Object>]
 [-UniqueId <Object>] [-ThrottleLimit <Object>] [-AsJob] [-StoragePool <Object>] [-StorageSubsystem <Object>]
 [-Volume <Object>] [-Disk <Object>]
```

## <a name="parameters"></a>パラメーター

### <a name="-asjob"></a>-AsJob


```yaml
Type: SwitchParameter
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-cimsession"></a>-CimSession
 

```yaml
Type: Object
Parameter Sets: (All)
Aliases: Session

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-disk"></a>-Disk
 

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-jobstate"></a>-JobState
 

```yaml
Type: Object
Parameter Sets: (All)
Aliases:
Accepted values: New, Starting, Running, Suspended, ShuttingDown, Completed, Terminated, Killed, Exception, Service, QueryPending

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-name"></a>-Name
 

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-storagepool"></a>-StoragePool
 

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-storagesubsystem"></a>-StorageSubsystem
 

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-throttlelimit"></a>-ThrottleLimit
 

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-uniqueid"></a>-UniqueId
 

```yaml
Type: Object
Parameter Sets: (All)
Aliases: Id

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-virtualdisk"></a>-VirtualDisk
 

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-volume"></a>-Volume
 

```yaml
Type: Object
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

## <a name="inputs"></a>入力

### <a name="systemstring"></a>System.String[]

### <a name="microsoftmanagementinfrastructureciminstance"></a>Microsoft.Management.Infrastructure.CimInstance

## <a name="outputs"></a>出力

### <a name="microsoftmanagementinfrastructureciminstance"></a>Microsoft.Management.Infrastructure.CimInstance

### <a name="microsoftmanagementinfrastructureciminstancerootmicrosoftwindowsstoragemsft_storagejob"></a>Microsoft.Management.Infrastructure.CimInstance#ROOT/Microsoft/Windows/Storage/MSFT_StorageJob

## <a name="next-steps"></a>次のステップ

特権エンドポイントにアクセスして使用する方法については、「[Azure Stack Hub で特権エンドポイントを使用する](https://docs.microsoft.com/azure-stack/operator/azure-stack-privileged-endpoint)」を参照してください。