---
title: Azure および Azure Stack アプリケーションでハイブリッド クラウド ID を構成する | Microsoft Docs
description: Azure および Azure Stack アプリケーションでハイブリッド クラウド ID を構成する方法について説明します。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/14/2019
ms.author: mabrigg
ms.reviewer: anajod
ms.lastreviewed: 01/14/2019
ms.openlocfilehash: a05f6fe12347e04d0329e2a0c81c1d4429af237d
ms.sourcegitcommit: 85c3acd316fd61b4e94c991a9cd68aa97702073b
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/01/2019
ms.locfileid: "64985807"
---
# <a name="tutorial-configure-hybrid-cloud-identity-for-azure-and-azure-stack-applications"></a>チュートリアル:Azure および Azure Stack アプリケーションのハイブリッド クラウド ID を構成する

*適用対象:Azure Stack 統合システムと Azure Stack Development Kit*

Azure および Azure Stack アプリケーションのハイブリッド クラウド ID を構成する方法について説明します。

グローバル Azure および Azure Stack の両方でアプリケーションへのアクセスを許可するには、2 つのオプションがあります。

 * Azure Stack がインターネットに継続的に接続している場合、Azure Active Directory (Azure AD) を使用できます。
 * Azure Stack がインターネットから切断されているときは、Active Directory フェデレーション サービス (AD FS) を使用できます。

Azure Stack 内の Azure Resource Manager を使用したデプロイまたは構成のために、サービス プリンシパルを使用して、Azure Stack アプリケーションへのアクセスを許可します。

このチュートリアルでは、以下を実現するためのサンプル環境を構築します。

> [!div class="checklist"]
> - グローバル Azure および Azure Stack でハイブリッド ID を確立する
> - Azure Stack API にアクセスするためのトークンを取得します。

このチュートリアルの手順を行うには、Azure Stack オペレーターのアクセス許可が必要です。

> [!Tip]  
> ![hybrid-pillars.png](./media/azure-stack-solution-cloud-burst/hybrid-pillars.png)  
> Microsoft Azure Stack は Azure の拡張機能です。 Azure Stack は、オンプレミスの環境にクラウド コンピューティングの俊敏性とイノベーションを提供します。これにより、どこでもハイブリッド アプリをビルドしてデプロイできる唯一のハイブリッド クラウドが実現されます。  
> 
> ホワイト ペーパー「[Design Considerations for Hybrid Applications](https://aka.ms/hybrid-cloud-applications-pillars)」では、ハイブリッドアプリケーションの設計、デプロイ、および操作に関するソフトウェア品質の重要な要素 (配置、スケーラビリティ、可用性、回復性、管理の容易性、セキュリティ) について概説しています。 これらの設計の考慮事項は、ハイブリッド アプリケーションの設計を最適化したり、運用環境での課題を最小限に抑えたりするのに役立ちます。


## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a>ポータルで Azure AD のサービス プリンシパルを作成する

Azure AD を ID ストアとして使用して Azure Stack をデプロイした場合は、Azure での手順と同様の方法でサービス プリンシパルを作成できます。 [サービス プリンシパルの作成](azure-stack-create-service-principals.md#create-service-principal-for-azure-ad)に関する記事では、ポータル経由でそれらの手順を実行する方法が示されています。 始める前に、[Azure AD で必要なアクセス許可](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions)があることを確認してください。

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a>PowerShell を使用して AD FS のサービス プリンシパルを作成する

AD FS を使用して Azure Stack をデプロイした場合は、PowerShell を使ってサービス プリンシパルを作成し、アクセスのロールを割り当てて、その ID を使用して PowerShell からサインインできます。 「[AD FS のサービス プリンシパルを作成する](azure-stack-create-service-principals.md#create-service-principal-for-ad-fs)」では、PowerShell を使って必要な手順を実行する方法が示されています。

## <a name="using-the-azure-stack-api"></a>Azure Stack API を使用する

Azure Stack API にアクセスするためのトークンを取得するプロセスについては、[Azure Stack API](azure-stack-rest-api-use.md) に関するチュートリアルで説明されています。

## <a name="connect-to-azure-stack-using-powershell"></a>PowerShell を使用して Azure Stack に接続する

[Azure Stack での PowerShell の設定と実行](../operator/azure-stack-powershell-install.md)に関するクイック スタートでは、Azure PowerShell をインストールして、Azure Stack のインストールに接続するために必要な手順が説明されています。

### <a name="prerequisites"></a>前提条件

アクセスできるサブスクリプションの Azure Active Directory に接続された Azure Stack インストール。 Azure Stack のインストールがない場合は、この手順に従って [Azure Stack Development Kit](../asdk/asdk-install.md) を設定できます。

#### <a name="connect-to-azure-stack-using-code"></a>コードを使用して Azure Stack に接続する

コードを使用して Azure Stack に接続するには、Azure Resource Manager エンドポイント API を使用して、Azure Stack インストールの認証およびグラフ エンドポイントを取得し、REST 要求を使用して認証します。 サンプル クライアント アプリケーションは [GitHub](https://github.com/shriramnat/HybridARMApplication) で見つかります。

>[!Note]
>選択した言語の Azure SDK で Azure API プロファイルがサポートされていない限り、SDK は Azure Stack で動作しません。 Azure API プロファイルについて詳しくは、「[Azure Stack での API バージョンのプロファイルの管理](azure-stack-version-profiles.md)」をご覧ください。

## <a name="next-steps"></a>次の手順

 - Azure Stack での ID の処理方法の詳細については、「[Azure Stack の ID アーキテクチャ](../operator/azure-stack-identity-architecture.md)」を参照してください。
 - Azure のクラウド パターンの詳細については、「[クラウド設計パターン](https://docs.microsoft.com/azure/architecture/patterns)」を参照してください。
