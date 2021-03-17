---
title: Azure Stack Hub の AKS エンジンの前提条件を設定する
description: Azure Stack Hub で AKS エンジンを実行するための前提条件を確立します。
author: mattbriggs
ms.topic: article
ms.date: 3/4/2021
ms.author: mabrigg
ms.reviewer: waltero
ms.lastreviewed: 3/4/2021
ms.openlocfilehash: be20bb06ce5b8b87bc6099e823ec1a1eb9aec01c
ms.sourcegitcommit: ccc4ee05d71496653b6e27de1bb12e4347e20ba4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102231610"
---
# <a name="set-up-the-prerequisites-for-the-aks-engine-on-azure-stack-hub"></a>Azure Stack Hub の AKS エンジンの前提条件を設定する

AKS エンジンは、ご自分の環境内の仮想マシン (VM)、または Azure Stack Hub Resource Manager エンドポイントにアクセスできる任意のクライアント マシンにインストールできます。 エンジンを実行する前に、次のものを用意する必要があります: サブスクリプションで使用できる AKS ベースの Ubuntu サーバーと Linux カスタム スクリプト拡張機能、共同作成者ロールに割り当てられているサービス プリンシパル ID、ご自分の Ubuntu サーバーにアクセスするための SSH 用秘密/公開キー。 また、Azure Stack Development Kit を使用している場合は、ご自分のマシンで適切な証明書を信頼する必要があります。

前提条件がある場合は、[クラスターの定義](azure-stack-kubernetes-aks-engine-deploy-cluster.md)を開始することができます。

Azure Stack Hub のクラウド オペレーターとして AKS エンジンを提供する場合は、[Azure Stack Hub Marketplace への AKS エンジンの追加](../operator/azure-stack-aks-engine.md)に関するページに記載されている手順に従ってください。

## <a name="prerequisites-for-the-aks-engine"></a>AKS エンジンの前提条件

AKS エンジンを使用するには、次のリソースが使用可能である必要があります。 AKS エンジンは、テナント サブスクリプションに Kubernetes クラスターをデプロイするために Azure Stack Hub のテナントによって使用されることを念頭に置いてください。 Azure Stack Hub オペレーターの関与が必要になるのは、マーケットプレースの項目をダウンロードし、サービス プリンシパル ID を作成するところのみです。 詳細については、次の表を参照してください。

クラウド オペレーターは次の項目を配置する必要があります。

| 前提条件 | 説明 | 必須 | Instructions |
| --- | --- | --- | --- |
| Azure Stack Hub 1910 以降 | AKS エンジンを使用するには、Azure Stack Hub 1910 以降が必要です。 | 必須 | Azure Stack Hub のバージョンがわからない場合は、クラウド オペレーターに問い合わせてください。 |
| Linux カスタム スクリプト拡張機能 | Linux カスタム スクリプト拡張機能 2.0<br>オファー: Linux 2.0 用のカスタム スクリプト<br>バージョン:2.0.6 (または最新バージョン)<br>発行元: Microsoft Corp | 必須 | サブスクリプションにこの項目がない場合は、クラウド オペレーターに問い合わせてください。 |
| AKS Base Image | AKS Base Ubuntu と Windows Image<br>バージョンの依存関係の詳細については、「[エンジンと基本イメージ バージョンの照合](#matching-engine-to-base-image-version)」をご覧ください。 | 必須 | サブスクリプションにこの項目がない場合は、クラウド オペレーターに問い合わせてください。<br> Azure Stack Hub のクラウド オペレーターとして AKS エンジンを提供する場合は、[Azure Stack Hub Marketplace への AKS エンジンの追加](../operator/azure-stack-aks-engine.md)に関するページに記載されている手順に従ってください。 |
| サービス プリンシパル ID (SPN) |  アプリケーションのリソースのデプロイや構成を Azure Resource Manager を通じて行う必要がある場合は、そのアプリケーションをサービス プリンシパルで表す必要があります。 | 必須 | この項目については、Azure Stack Hub オペレーターへの問い合わせが必要な場合があります。<br>Azure Active Directory (Azure AD) サービス プリンシパル ID を使用する場合は、サービス プリンシパルを Azure AD で認証できるように、Kubernetes クラスター内の VM からインターネットにアクセスする必要があります。 インターネットにアクセスできない場合、Kubernetes クラスターは機能しません。<br>手順については、「[アプリ ID を使用してリソースにアクセスする](../operator/azure-stack-create-service-principals.md)」を参照してください |
| (SPN) 割り当て済み **共同作成者** ロール | 自分のサブスクリプションに含まれるリソースに、アプリケーションからサービス プリンシパルを使用してアクセスできるようにするには、そのサービス プリンシパルを特定のリソースに対するロールに割り当てる必要があります。 | 必須 | 手順については、「[ロールの割り当て](../operator/azure-stack-create-service-principals.md#assign-a-role)」を参照してください |


以下の項目を設定できます。

| 前提条件 | 説明 | 必須 | Instructions |
| --- | --- | --- | --- |
| Azure Stack Hub サブスクリプション | サブスクリプションを使用して、Azure Stack Hub 内のオファーにアクセスします。 オファーには、利用可能なサービスが含まれています。 | 必須 | Azure Stack Hub でテナントのワークロードをデプロイするには、最初に [Azure Stack Hub サブスクリプション](./azure-stack-subscribe-services.md)を取得する必要があります。 |
| Resource group | リソース グループは、Azure ソリューションの関連するリソースを保持するコンテナーです。 既存のリソース グループを指定しない場合は、ツールによって作成されます。 | 省略可能 | [Azure portal を使用して Azure Resource Manager リソース グループを管理する](/azure/azure-resource-manager/manage-resource-groups-portal) |
| 秘密/公開キー | 開発用マシンから、ご自分の Web アプリをホストする Azure Stack Hub インスタンス内のサーバー VM へのオープン SSH 接続を使用するには、SSH (Secure Shell) 公開キーと秘密キーのペアを作成する必要があります。 | 必須 | キーを生成する手順については、[SSH キーの生成](./azure-stack-dev-start-howto-ssh-public-key.md)に関するページを参照してください。|


> [!Note]  
> [Azure Stack Hub 用の Azure CLI](./azure-stack-version-profiles-azurecli2.md) または [Azure Stack Hub PowerShell](../operator/powershell-install-az-module.md) で AKS エンジンを使用するための前提条件を作成することもできます。

## <a name="matching-engine-to-base-image-version"></a>エンジンと基本イメージバージョンの照合

AKS エンジンによって、カスタマイズされた Ubuntu Server OS が各クラスター ノード イメージである **AKS Base Ubuntu と Windows Image ディストリビューション** にデプロイされます。 すべての AKS エンジンのバージョンは、Azure Stack Hub オペレーターが Azure Stack Hub で使用できる特定のイメージ バージョンに依存します。 AKS エンジンのバージョンと、それに対してサポートされている Kubernetes のバージョンが一覧表示された表は、「[Supported Kubernetes Versions](kubernetes-aks-engine-release-notes.md#aks-engine-and-azure-stack-version-mapping)」 (サポートされている Kubernetes バージョン) に記載されています。 たとえば、AKS エンジンのバージョン `v0.55.0` は、AKS Base Ubuntu と Windows Image ディストリビューションのバージョン `2020.08.24` によって変わります。 Azure Stack Hub のオペレーターに、Azure Marketplace から Azure Stack Hub Marketplace に特定のイメージ バージョンをダウンロードするよう依頼してください。

Azure Stack Hub Marketplace にイメージがない場合は、エラーがトリガーされます。 たとえば、AKS エンジンのバージョン v0.55.0 を現在使用していて、AKS Base Ubuntu および Windows Image ディストリビューション バージョン `2020.08.24` を使用できない場合は、AKS エンジンの実行時に次のエラーが表示されます。 

```Text  
The platform image 'microsoft-aks:aks:aks-ubuntu-1604-202003:2020.08.24' is not available. 
Verify that all fields in the storage profile are correct.
```

次のコマンドを実行すると、ご自分の AKS エンジンのバージョンを確認できます。

```bash  
$ aks-engine version
Version: v0.55.0
GitCommit: 44a35c00c
GitTreeState: clean
```

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [Azure Stack Hub の Windows に AKS エンジンをデプロイする](azure-stack-kubernetes-aks-engine-deploy-windows.md)  
> [Azure Stack Hub の Linux に AKS エンジンをデプロイする](azure-stack-kubernetes-aks-engine-deploy-linux.md)
