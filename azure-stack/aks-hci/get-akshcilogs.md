---
title: Get-AksHciLogs
author: jessicaguan
description: Get-AksHciLogs PowerShell コマンドは、お使いのすべてのポッドからログを含む zip 形式のフォルダーを作成します。
ms.topic: reference
ms.date: 2/12/2021
ms.author: jeguan
ms.openlocfilehash: ef78efb91a6a8b5c0c91e815cb8aedd0a57adfe8
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101874564"
---
# <a name="get-akshcilogs"></a>Get-AksHciLogs

## <a name="synopsis"></a>構文
お使いのすべてのポッドからログを含む zip 形式のフォルダーを作成します。 

## <a name="syntax"></a>構文

```powershell
Get-AksHciLogs
```

## <a name="description"></a>説明
お使いのすべてのポッドからログを含む zip 形式のフォルダーを作成します。 このコマンドにより、AKS on Azure Stack HCI 作業ディレクトリ内に、`akshcilogs.zip` と呼ばれる出力 zip 形式のフォルダーが作成されます。 `akshcilogs.zip` ファイルへの完全なパスは、`Get-AksHciLogs` の実行後の出力になり ます (例: `C:\AksHci\0.9.6.3\akshcilogs.zip`。ここで `0.9.6.3` は AKS on Azure Stack HCI リリース番号です)。

## <a name="examples"></a>例

### <a name="example"></a>例
```powershell
PS C:\> Get-AksHciLogs
```
