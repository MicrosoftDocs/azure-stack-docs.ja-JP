---
title: Azure Stack Hub の Kubernetes にコンテナー ストレージを追加する
description: Azure Stack Hub の Kubernetes にコンテナー ストレージを追加する方法について説明します。
author: mattbriggs
ms.topic: how-to
ms.date: 3/4/2021
ms.author: mabrigg
ms.reviewer: waltero
ms.lastreviewed: 3/4/2021
ms.openlocfilehash: ee176f8e111c6e10831124b78afd99535d2fe7ee
ms.sourcegitcommit: ccc4ee05d71496653b6e27de1bb12e4347e20ba4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102233894"
---
# <a name="add-container-storage-to-kubernetes-in-azure-stack-hub"></a>Azure Stack Hub の Kubernetes にコンテナー ストレージを追加する

> [!IMPORTANT]  
> 現在、この機能はパブリック プレビュー段階にあります。
> このプレビュー バージョンはサービス レベル アグリーメントなしで提供されています。運用環境のワークロードに使用することはお勧めできません。 特定の機能はサポート対象ではなく、機能が制限されることがあります。 詳しくは、[Microsoft Azure プレビューの追加使用条件](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)に関するページをご覧ください。

ツリー内のボリューム プロバイダーを Container Storage Interface ([CSI](https://kubernetes.io/blog/2019/01/15/container-storage-interface-ga/)) に移動する Kubernetes コミュニティの取り組み ([ツリー内の Kubernetes から CSI ボリュームへの移行](https://kubernetes.io/blog/2019/12/09/kubernetes-1-17-feature-csi-migration-beta/)) の一環として、Azure Stack で次の 2 つの CSI ドライバー (Azure Disk と NFS) を見つけることができます。

|                       | **Azure Disk CSI ドライバー**                                                                                                    | **NFS CSI ドライバー**                                                       |
|-----------------------|------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------|
| プロジェクト リポジトリ    | [azuredisk-csi-driver](https://github.com/kubernetes-sigs/azuredisk-csi-driver)                                              | [csi-driver-nfs](https://github.com/kubernetes-csi/csi-driver-nfs)       |
| CSI ドライバー バージョン    | v1.0.0 以降                                                                                                                      | v3.0.0 以降                                                                  |
| アクセス モード           | ReadWriteOnce                                                                                                                | ReadWriteOnce ReadOnlyMany ReadWriteMany                                 |
| Windows エージェント ノード    | サポート                                                                                                                      | サポートなし                                                              |
| 動的プロビジョニング  | サポート                                                                                                                      | サポート                                                                  |
| 考慮事項        | [Azure Disk CSI ドライバーの制限事項](https://github.com/kubernetes-sigs/azuredisk-csi-driver/blob/master/docs/limitations.md) | NFS サーバーの設定と管理は、ユーザーが行う必要があります。 |
| Slack のサポート窓口 | [\#provider-azure](https://kubernetes.slack.com/archives/C5HJXTT9Q)                                                          | [\#sig-storage](https://kubernetes.slack.com/archives/C09QZFCE5)         |

現時点では、接続されていない環境では CSI ドライバーはサポートされていません。

## <a name="requirements"></a>必要条件

-   Azure Stack ビルド 2011 以降。
-   AKS エンジン バージョン v 0.60.1 以降。
-   Kubernetes バージョン 1.18 以降。
-   CSI ドライバーのコントローラー サーバーには 2 つのレプリカが必要であるため、単一ノードのマスター プールはお勧めできません。
-   [Helm 3](https://helm.sh/docs/intro/install/)

## <a name="install-and-uninstall-csi-drivers"></a>CSI ドライバーのインストールとアンインストール

このセクションでは、サンプル コマンドに従って、CSI ドライバーを使用する statefulset アプリケーションをデプロイします。

## <a name="azure-disk-csi-driver"></a>Azure Disk CSI ドライバー

### <a name="install-csi-driver"></a>CSI ドライバーのインストール

```bash  
helm repo add azuredisk-csi-driver https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/charts
helm install azuredisk-csi-driver azuredisk-csi-driver/azuredisk-csi-driver --namespace kube-system --set cloud=AzureStackCloud --set controller.runOnMaster=true --version v1.0.0

```
### <a name="deploy-storage-class"></a>ストレージ クラスのデプロイ

```bash  
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/storageclass-azuredisk-csi-azurestack.yaml
```

### <a name="deploy-example-statefulset-application"></a>サンプル statefulset アプリケーションのデプロイ

```bash  
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/statefulset.yaml
```

### <a name="validate-volumes-and-applications"></a>ボリュームとアプリケーションの検証

ボリュームに保持されている一連のタイムスタンプが表示されます。

```bash  
kubectl exec statefulset-azuredisk-0 -- tail /mnt/azuredisk/outfile
```

### <a name="delete-example-statefulset-application"></a>サンプル statefulset アプリケーションの削除

```bash  
kubectl delete -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/statefulset.yaml
```

### <a name="delete-storage-class"></a>ストレージ クラスの削除

ストレージ クラスを削除する前に、そのストレージ クラスを使用しているポッドが終了していることを確認してください。

```bash  
kubectl delete -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/storageclass-azuredisk-csi-azurestack.yaml
```

### <a name="uninstall-csi-driver"></a>CSI ドライバーのアンインストール

```bash  
helm uninstall azuredisk-csi-driver --namespace kube-system
helm repo remove azuredisk-csi-driver
```

## <a name="nfs-csi-driver"></a>NFS CSI ドライバー

### <a name="install-csi-driver"></a>CSI ドライバーのインストール

```bash  
helm repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
helm install csi-driver-nfs csi-driver-nfs/csi-driver-nfs --namespace kube-system --set controller.runOnMaster=true --version v3.0.0
```

### <a name="deploy-nfs-server"></a>NFS サーバーのデプロイ

> [!NOTE]  
> NFS サーバーは、運用環境用に NFS サーバーを適切に検証、セットアップ、および管理するためのものです。

NFS サーバーを運用環境用に適切にセットアップして管理します。

```bash  
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/deploy/example/nfs-provisioner/nfs-server.yaml
```

### <a name="deploy-storage-class"></a>ストレージ クラスのデプロイ

```bash  
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/deploy/example/storageclass-nfs.yaml
```

### <a name="deploy-example-statefulset-application"></a>サンプル statefulset アプリケーションのデプロイ

```bash  
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/deploy/example/statefulset.yaml
```

### <a name="validate-volumes-and-applications"></a>ボリュームとアプリケーションの検証

ボリュームに保持されている一連のタイムスタンプが表示されます。

```bash  
kubectl exec statefulset-nfs-0 -- tail /mnt/nfs/outfile
```

### <a name="delete-example-statefulset-application"></a>サンプル statefulset アプリケーションの削除

```bash  
kubectl delete -f https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/deploy/example/nfs-provisioner/nfs-server.yaml
```

### <a name="delete-storage-class"></a>ストレージ クラスの削除

ストレージ クラスを削除する前に、そのストレージ クラスを使用しているポッドが終了していることを確認してください。

```bash  
kubectl delete -f https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/deploy/example/storageclass-nfs.yaml
```
### <a name="delete-example-nfs-server"></a>サンプル NFS サーバーの削除

```bash  
kubectl delete -f https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/deploy/example/nfs-provisioner/nfs-server.yaml
```

### <a name="uninstall-csi-driver"></a>CSI ドライバーのアンインストール

```bash  
helm uninstall csi-driver-nfs --namespace kube-system
helm repo remove csi-driver-nfs
```
## <a name="azure-disk-csi-driver-limitations-on-azure-stack-hub"></a>Azure Stack Hub での Azure Disk CSI ドライバーの制限事項

-   Azure Disk IOPS の上限は 2300 です。詳細については、「[Azure Stack Hub でサポートされている VM サイズ](azure-stack-vm-sizes.md)」をお読みください。
-   Azure Stack Hub では共有ディスクがサポートされていないため、1 より大きい `maxShares` パラメーターは StorageClass では無効です。
-   Azure Stack Hub でサポートされているのは、Standard ローカル冗長 (Standard_LRS) と Premium ローカル冗長 (Premium_LRS) のストレージ アカウントの種類のみであるため、`StorageClass` のパラメーター `skuName` に対して有効なのは Standard_LRS と Premium_LRS だけです。
-   Azure Stack Hub では増分ディスク スナップショットがサポートされていないため、`VolumeSnapshotClass` のパラメーター `incremental` に対して有効なのは false のみです。
-   Windows エージェント ノードの場合は、Windows CSI プロキシをインストールする必要があります。[Windows CSI プロキシ](https://github.com/kubernetes-csi/csi-proxy)に関するページを参照してください。 AKS エンジン API モデルを使用してプロキシを有効にするには、「[CSI Proxy for Windows](https://github.com/Azure/aks-engine/blob/master/docs/topics/csi-proxy-windows.md)」(Windows の CSI プロキシ) を参照してください。

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub 上の AKS エンジン](azure-stack-kubernetes-aks-engine-overview.md)を確認してください  
