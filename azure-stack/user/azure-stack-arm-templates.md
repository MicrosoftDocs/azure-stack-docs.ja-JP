---
title: Azure Stack で Azure Resource Manager テンプレートを使用する | Microsoft Docs
description: Azure Stack で Azure Resource Manager テンプレートを使用してリソースをプロビジョニングする方法を説明します。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: 2022dbe5-47fd-457d-9af3-6c01688171d7
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/04/2019
ms.author: sethm
ms.reviewer: justini
ms.lastreviewed: 11/14/2018
ms.openlocfilehash: 0b9aedb4a6b046755192b84e18e8c75a8d015c8f
ms.sourcegitcommit: f91979c1613ea1aa0e223c818fc208d902b81299
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/05/2019
ms.locfileid: "71974066"
---
# <a name="use-azure-resource-manager-templates-in-azure-stack"></a>Azure Stack で Azure リソース マネージャー テンプレートを使用する

*適用対象:Azure Stack 統合システムと Azure Stack Development Kit*

Azure Resource Manager テンプレートを使用して、お使いのアプリケーションのすべてのリソースを、単一の連携した操作でデプロイしてプロビジョニングできます。 テンプレートを再デプロイして、リソース グループ内のリソースを変更することもできます。

これらのテンプレートは、Microsoft Azure Stack ポータル、PowerShell、コマンド ライン、および Visual Studio を使用してデプロイできます。

以下のクイックスタート テンプレートを [GitHub](https://aka.ms/azurestackgithub) で入手できます。

## <a name="deploy-sharepoint-server-non-high-availability-deployment"></a>SharePoint サーバーのデプロイ (非高可用性デプロイ)

PowerShell [Desired State Configuration](/powershell/dsc/overview/overview) (DSC) 拡張機能を使用して、以下のリソースを含む [SharePoint Server 2013 ファームを作成](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/sharepoint-2013-non-ha)します。

* 仮想ネットワーク
* 3 つのストレージ アカウント
* 2 つの外部ロード バランサー
* 単一のドメインを使用する新しいフォレスト用のドメイン コントローラーとして構成された 1 台の仮想マシン (VM)
* SQL Server 2014 のスタンドアロン サーバーとして構成された 1 台の VM
* ワンマシン SharePoint Server 2013 ファームとして構成された 1 台の VM

## <a name="deploy-ad-non-high-availability-deployment"></a>AD のデプロイ (非高可用性デプロイ)

PowerShell DSC 拡張機能を使用して、以下のリソースを含む [AD ドメイン コントローラーを作成](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/ad-non-ha)します。

* 仮想ネットワーク
* 1 つのストレージ アカウント
* 1 つの外部ロード バランサー
* 単一のドメインを使用する新しいフォレスト用のドメイン コントローラーとして構成された 1 台の仮想マシン (VM)

## <a name="deploy-adsql-non-high-availability-deployment"></a>AD/SQL のデプロイ (非高可用性デプロイ)

PowerShell DSC 拡張機能を使用して、以下のリソースを含む [SQL Server 2014 スタンドアロン サーバーを作成](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/sql-2014-non-ha)します。

* 仮想ネットワーク
* 2 つのストレージ アカウント
* 1 つの外部ロード バランサー
* 単一のドメインを使用する新しいフォレスト用のドメイン コントローラーとして構成された 1 台の仮想マシン (VM)
* SQL Server 2014 のスタンドアロン サーバーとして構成された 1 台の VM

## <a name="vm-dsc-extension-azure-automation-pull-server"></a>VM-DSC-Extension-Azure-Automation-Pull-Server

PowerShell DSC 拡張機能を使用して、既存の仮想マシンのローカル構成マネージャー (LCM) を構成し、それを Azure Automation Account DSC Pull Server に登録します。

## <a name="create-a-virtual-machine-from-a-user-image"></a>ユーザー イメージからの仮想マシンの作成

[カスタム ユーザー イメージから仮想マシンを作成](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/101-vm-create-from-customimage)します。 このテンプレートは、仮想ネットワーク (DNS 付き)、パブリック IP アドレス、およびネットワーク インターフェイスもデプロイします。

## <a name="basic-virtual-machine"></a>基本的な仮想マシン

仮想ネットワーク (DNS 付き)、パブリック IP アドレス、およびネットワーク インターフェイスを含む [Windows VMをデプロイ](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/101-simple-windows-vm)します。

## <a name="cancel-a-running-template-deployment"></a>実行中のテンプレートのデプロイのキャンセル

実行中のテンプレートのデプロイをキャンセルするには、[Stop-AzureRmResourceGroupDeployment](/powershell/module/azurerm.resources/stop-azurermresourcegroupdeployment) PowerShell [コマンドレット](/powershell/developer/cmdlet/cmdlet-overview)を使用します。

## <a name="next-steps"></a>次の手順

* [ポータルを使用してテンプレートをデプロイする](azure-stack-deploy-template-portal.md)
* [PowerShell を使用したテンプレートのデプロイ](azure-stack-deploy-template-powershell.md)
* [Visual Studio を使用したテンプレートのデプロイ](azure-stack-deploy-template-visual-studio.md)
* [Azure リソース マネージャーの概要](/azure/azure-resource-manager/resource-group-overview)
