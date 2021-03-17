---
title: Azure Kubernetes Services (AKS) エンジンの前提条件を Azure Stack Hub Marketplace に追加する
description: AKS エンジンの前提条件を Azure Stack Hub Marketplace に追加する方法について説明します。
author: mattbriggs
ms.topic: article
ms.date: 3/1/2021
ms.author: mabrigg
ms.reviewer: waltero
ms.lastreviewed: 3/1/2021
ms.openlocfilehash: ceb51221d03f9c037ac3af2304b55d691f17f021
ms.sourcegitcommit: ccc4ee05d71496653b6e27de1bb12e4347e20ba4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102231236"
---
# <a name="add-the-azure-kubernetes-services-aks-engine-prerequisites-to-the-azure-stack-hub-marketplace"></a>Azure Kubernetes Services (AKS) エンジンの前提条件を Azure Stack Hub Marketplace に追加する

ユーザー向けに Azure Kubernetes Services (AKS) エンジンを設定することができます。 この記事で説明する項目を Azure Stack ハブに追加します。 ユーザーはその後、1 回の連携した操作で Kubernetes クラスターをデプロイできます。 この記事では、接続されている環境と接続されていない環境の両方で、ユーザーが AKS エンジンを使用できるようにするために必要な手順について説明します。 AKS エンジンは、サービス プリンシパル ID と、Marketplace のカスタム スクリプト拡張機能および AKS 基本イメージに依存します。 AKS エンジンを使用するには、[Azure Stack Hub 1910](release-notes.md?view=azs-1910&preserve-view=true) 以降を実行している必要があります。

> [!NOTE]  
> Azure Stack Hub と AKS エンジンのバージョン番号のマッピングについては、[AKS エンジンのリリース ノート](../user/kubernetes-aks-engine-release-notes.md#aks-engine-and-azure-stack-version-mapping)をご覧ください。

## <a name="check-your-users-service-offering"></a>ユーザーのサービス オファリングを確認する

ユーザーには、十分な領域がある Azure Stack Hub のプラン、オファー、およびサブスクリプションが必要です。 多くの場合、ユーザーは、3 つのマスター ノードと 3 つのワーカー ノードで構成される最大 6 台の仮想マシンによるクラスターをデプロイする必要があります。 ユーザーにとって十分なクォータがあることを確認する必要があります。

サービス オファリングの計画と設定に関する詳細については、「[Azure Stack Hub でのサービスの提供の概要](service-plan-offer-subscription-overview.md)」を参照してください

## <a name="create-a-service-principal-and-credentials"></a>サービス プリンシパルと資格情報を作成する

Kubernetes クラスターには、Azure Stack Hub でのサービス プリンシパル (SPN) とロールベースのアクセス許可が必要です。

### <a name="create-an-spn-in-azure-ad"></a>Azure AD で SPN を作成する

ID 管理サービスのために Azure Active Directory (Azure AD) を使用する場合は、Kubernetes クラスターをデプロイするユーザーのサービス プリンシパルを作成する必要があります。 クライアント シークレットを使用してサービス プリンシパルを作成します。 手順については、「[クライアント シークレット資格情報を使用するサービス プリンシパルを作成する](azure-stack-create-service-principals.md#create-a-service-principal-that-uses-a-client-secret-credential)」を参照してください。

### <a name="create-an-spn-in-ad-fs"></a>AD FS で SPN を作成する

ID 管理サービスのために Active Directory Federated Services (AD FS) を使用する場合は、Kubernetes クラスターをデプロイするユーザーのサービス プリンシパルを作成する必要があります。 クライアント シークレットを使用してサービス プリンシパルを作成します。 手順については、[クライアント シークレットを使用したサービス プリンシパルの作成](azure-stack-create-service-principals.md#create-a-service-principal-that-uses-client-secret-credentials)に関するセクションを参照してください。

## <a name="add-an-aks-base-image"></a>AKS Base Image を追加する

Azure から項目を取得して、AKS Base Image を Marketplace に追加できます。 ただし、Azure Stack Hub が切断されている場合は、次の手順に従って、[Azure から Marketplace の項目をダウンロード](azure-stack-download-azure-marketplace-item.md?pivots=state-disconnected)し、項目を追加します。 手順 5 で指定した項目を追加します。

次の項目を Marketplace に追加します。

1. 管理ポータル `https://adminportal.local.azurestack.external` にサインインします。

1. **[すべてのサービス]** を選択し、 **[管理]** カテゴリで **[Marketplace management] (Marketplace 管理)** を選択します。

1. **+ Add from Azure**(+ Azure から追加) を選択します。

1. 「`AKS Base`」と入力します。

1. AKS エンジンのバージョンと一致するイメージ バージョンを選択します。 AKS エンジンと AKS 基本イメージの対応一覧については、「[Supported Kubernetes Versions (サポートされている Kubernetes バージョン)](..\user\kubernetes-aks-engine-release-notes.md#aks-engine-and-azure-stack-version-mapping)」を参照してください。 

1. **[ダウンロード]** を選択します。

## <a name="add-a-custom-script-extension"></a>カスタム スクリプト拡張機能を追加する

Azure から項目を取得して、カスタム スクリプトを Marketplace に追加できます。 ただし、Azure Stack Hub が切断されている場合は、次の手順に従って、[Azure から Marketplace の項目をダウンロード](azure-stack-download-azure-marketplace-item.md?pivots=state-disconnected)し、項目を追加します。  手順 5 で指定した項目を追加します。

1. 管理ポータル `https://adminportal.local.azurestack.external` を開きます。

1. **[すべてのサービス]** を選択し、 **[管理]** カテゴリで **[Marketplace management] (Marketplace 管理)** を選択します。

1. **+ Add from Azure**(+ Azure から追加) を選択します。

1. 「`Custom Script for Linux`」と入力します。

1. 次のプロファイルを持つスクリプトを選択します。
   - **[プラン]** : Linux 2.0 用のカスタム スクリプト
   - **バージョン**:2.0.6 (または最新バージョン)
   - **[発行者]** : Microsoft Corp

     > [!Note]  
     > 複数のバージョンの Linux 用カスタム スクリプトが表示されることがあります。 最新バージョンの項目を追加する必要があります。

1. **[ダウンロード]** を選択します。

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub の AKS エンジンとは](../user/azure-stack-kubernetes-aks-engine-overview.md)

[Azure Stack Hub でのサービスの提供の概要](service-plan-offer-subscription-overview.md)