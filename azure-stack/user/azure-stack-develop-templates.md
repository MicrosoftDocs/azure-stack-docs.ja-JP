---
title: Azure Stack のテンプレートの開発 | Microsoft Docs
description: Azure Stack テンプレートのベスト プラクティスを説明します
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: 8a5bc713-6f51-49c8-aeed-6ced0145e07b
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/21/2019
ms.author: sethm
ms.reviewer: unknown
ms.lastreviewed: 05/21/2019
ms.openlocfilehash: 9967da0434be577e3db8586f28e3078658623e9b
ms.sourcegitcommit: 6fcd5df8b77e782ef72f0e1419f1f75ec8c16c04
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/21/2019
ms.locfileid: "65991331"
---
# <a name="azure-resource-manager-template-considerations"></a>Azure Resource Manager テンプレートに関する考慮事項

*適用対象:Azure Stack 統合システムと Azure Stack Development Kit*

アプリケーションを開発するときは、Azure および Azure Stack 間のテンプレートの移植性を確保する必要があります。 この記事では、[Azure Resource Manager テンプレート](https://download.microsoft.com/download/E/A/4/EA4017B5-F2ED-449A-897E-BD92E42479CE/Getting_Started_With_Azure_Resource_Manager_white_paper_EN_US.pdf)を開発するための考慮事項を説明して、アプリケーションのプロトタイプ作成とデプロイのテストを、Azure Stack 環境にアクセスせずに Azure で実行できるようにします。

## <a name="resource-provider-availability"></a>リソース プロバイダーの可用性

デプロイする予定のテンプレートは、Azure Stack で既に使用できるか、またはプレビューの段階にある Microsoft Azure サービスのみを使用する必要があります。

## <a name="public-namespaces"></a>パブリック名前空間

Azure Stack はデータセンターでホストされるため、そのサービス エンドポイントの名前空間は、Azure パブリック クラウドとは異なります。 その結果、Azure Resource Manager テンプレートでハードコーディングされたパブリック エンドポイントは、Azure Stack にデプロイしようとすると失敗します。 デプロイ中にリソース プロバイダーから値を取得するために、`reference` および `concatenate` 関数を使用してサービス エンドポイントを動的に構築できます。 たとえば、テンプレートに `blob.core.windows.net` をハードコーディングする代わりに、[primaryEndpoints.blob](https://github.com/Azure/AzureStack-QuickStart-Templates/blob/master/101-vm-windows-create/azuredeploy.json#L175) を取得して *osDisk.URI* エンドポイントを動的に設定します。

```json
"osDisk": {"name": "osdisk","vhd": {"uri":
"[concat(reference(concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName')), '2015-06-15').primaryEndpoints.blob, variables('vmStorageAccountContainerName'),
 '/',variables('OSDiskName'),'.vhd')]"}}
```

## <a name="api-versioning"></a>API のバージョン管理

Azure のサービス バージョンが Azure と Azure Stack で異なる場合があります。 各リソースには、提供される機能を定義する **apiVersion** 属性が必要です。 Azure Stack での API のバージョンの互換性を確保するための各リソースプロバイダーで有効な API のバージョンは次のとおりです。

| リソース プロバイダー | apiVersion |
| --- | --- |
| Compute |**2015-06-15** |
| ネットワーク |**2015-06-15**、**2015-05-01-preview** |
| Storage |**2016-01-01**、**2015-06-15**、**2015-05-01-preview** |
| KeyVault | **2015-06-01** |
| App Service |**2015-08-01** |

## <a name="template-functions"></a>テンプレート関数

Azure Resource Manager の[関数](/azure/azure-resource-manager/resource-group-template-functions)は、動的テンプレートを構築するために必要な機能を提供します。 たとえば、次のようなタスクのために関数を使用できます。

* 文字列の連結やトリミング。
* その他のリソースからの値の参照。
* 複数のインスタンスをデプロイするためのリソースの反復処理。

次の関数は、Azure Stack では使用できません。

* Skip
* Take

## <a name="resource-location"></a>リソースの場所

Azure Resource Manager テンプレートは、デプロイ時にリソースを配置するのに `location` 属性を使用します。 Azure では、場所は、米国西部や南アメリカなどのリージョンを指します。 Azure Stack では、Azure Stack はユーザーのデータセンター内にあるため、場所の意味が異なります。 Azure と Azure Stack 間でテンプレートを確実に転送するには、個々のリソースをデプロイするときにリソース グループの場所を参照する必要があります。 そのためには、`[resourceGroup().Location]` を使用して、すべてのリソースがリソース グループの場所を継承するようにします。 次のコードは、ストレージ アカウントのデプロイ中にこの関数を使用する例を示しています。

```json
"resources": [
{
  "name": "[variables('storageAccountName')]",
  "type": "Microsoft.Storage/storageAccounts",
  "apiVersion": "[variables('apiVersionStorage')]",
  "location": "[resourceGroup().location]",
  "comments": "This storage account is used to store the VM disks",
  "properties": {
  "accountType": "Standard_GRS"
  }
}
]
```

## <a name="next-steps"></a>次の手順

* [PowerShell を使用したテンプレートのデプロイ](azure-stack-deploy-template-powershell.md)
* [Azure CLI を使用したテンプレートのデプロイ](azure-stack-deploy-template-command-line.md)
* [Visual Studio を使用したテンプレートのデプロイ](azure-stack-deploy-template-visual-studio.md)
