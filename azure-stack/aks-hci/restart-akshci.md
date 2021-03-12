---
title: Restart-AksHci
author: jessicaguan
description: Restart-AksHci PowerShell コマンドは AKS on Azure Stack HCI を再起動し、デプロイされているすべての Kubernetes クラスターを削除します。
ms.topic: reference
ms.date: 2/12/2021
ms.author: jeguan
ms.openlocfilehash: 358ca36b085b21c6ecdbea7a024d9e72e0ca0597
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101874591"
---
# <a name="restart-akshci"></a>Restart-AksHci

## <a name="synopsis"></a>構文
Azure Kubernetes Service on Azure Stack HCI を再起動し、デプロイされているすべての Kubernetes クラスターを削除します。

## <a name="syntax"></a>構文

### <a name="restart-aks-hci"></a>Restart AKS-HCI
```powershell
Restart-AksHci
```

## <a name="description"></a>説明
Azure Kubernetes Service on Azure Stack HCI を再起動すると、すべての Kubernetes クラスター (ある場合)、および Azure Kubernetes Service ホストが削除されます。 また、ノードから Azure Stack HCI エージェントおよびサービス上の Azure Kubernetes Service もアンインストールされます。 その後、ホストが再作成されるまで、元のインストール プロセスのステップが再実行されます。 Set-AksHciConfig を使用して構成した Azure Kubernetes Service on Azure Stack HCI と、ダウンロードした VHDX イメージは保持されます。

## <a name="examples"></a>例

### <a name="example"></a>例
```powershell
PS C:\> Restart-AksHci
```
