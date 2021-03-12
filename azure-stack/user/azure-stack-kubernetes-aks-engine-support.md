---
title: Azure Stack Hub の AKS エンジンのサポート ポリシー
description: このトピックでは Azure Stack Hub の AKS エンジンのサポート ポリシーについて説明します。
author: mattbriggs
ms.topic: article
ms.date: 2/1/2021
ms.author: mabrigg
ms.reviewer: waltero
ms.lastreviewed: 09/0102020
ms.openlocfilehash: 9969447ab737c6913576c73b2e53453a6cb8e1f6
ms.sourcegitcommit: ccc4ee05d71496653b6e27de1bb12e4347e20ba4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102231763"
---
# <a name="support-policies-for-aks-engine-on-azure-stack-hub"></a>Azure Stack Hub の AKS エンジンのサポート ポリシー

この記事では、Azure Stack Hub の AKS エンジンのテクニカル サポート ポリシーおよび制限事項の詳細について説明します。 また、Kubernetes Marketplace の項目、サード パーティ製オープン ソース コンポーネント、およびセキュリティまたは修正プログラムの管理についても説明します。 

## <a name="self-managed-kubernetes-clusters-on-azure-stack-hub-with-aks-engine"></a>AKS エンジンを使用した Azure Stack Hub の自己管理型 Kubernetes クラスター

コンピューティング コンポーネントやネットワーク コンポーネントなどのサービスとしてのインフラストラクチャ (IaaS) クラウド コンポーネントでは、低レベルの制御とカスタマイズのオプションへのアクセス権がユーザーに付与されます。 AKS エンジンを使用すると、ユーザーはこれらの IaaS コンポーネントを使用して Kubernetes クラスターを透過的に展開できます。そのため、ユーザーはデプロイのすべての側面にアクセスして影響を与えることができます。

クラスターが作成されると、お客様は AKS エンジンで作成される Kubernetes マスターおよびワーカー ノードを定義します。 お客様のワークロードは、これらのノードで実行されます。 お客様はこれらのマスターおよびワーカー ノードを所有し、表示または変更できます。 ノードが不注意に変更された場合、データとワークロードの損失が発生し、クラスターが機能しない状態になる可能性があります。 また、アップグレードやスケールなどの AKS エンジン操作により、バインドされていない変更が上書きされます。 たとえば、クラスターに静的ポッドがある場合、これらは AKS エンジンのアップグレード操作後は保持されません。

お客様のクラスター ノードではプライベート コードが実行され、機密データが格納されるため、Microsoft サポートはこれらのノードへのアクセスを制限されています。 Microsoft サポートは、お客様による明確な許可または支援なしに、これらのノードにサインインしたり、これらのノードでコマンドを実行したり、これらのノードのログを表示することができません。

## <a name="version-support"></a>バージョンのサポート

AKS エンジンのバージョン サポートは、残りの Azure Stack Hub のサポート ポリシーで確立されているのと同じパターンに従います。つまり、Azure Stack Hub での AKS エンジンのバージョンのサポートは、n-2 式に基づきます。 たとえば、AKS エンジンの最新バージョンが v0.55.0 の場合、サポートされるバージョンのセットは次のようになります。0.48.0、0.51.0、0.55.0。 また、Azure Stack Hub の更新バージョンおよび AKS エンジンでサポートされるバージョンへの対応するマッピングに従うことも重要です。これは、[AKS エンジンのリリース ノート](kubernetes-aks-engine-release-notes.md#aks-engine-and-azure-stack-version-mapping)に記載されています。

## <a name="aks-engine-supported-areas"></a>AKS エンジンのサポートされている領域

Microsoft は、以下に関するテクニカル サポートを提供します。

-  AKS エンジン コマンドに関する問題: デプロイ、生成、アップグレード、およびスケール。 このツールは、Azure での動作と一致している必要があります。
-  「[AKS エンジンの概要](azure-stack-kubernetes-aks-engine-overview.md)」に従ってデプロイされた Kubernetes クラスターに関する問題。
-  他の Azure Stack Hub サービスへの接続に関する問題。 
-  Kubernetes API 接続に関する問題。
-  Azure Resource Manager を使用した Azure Stack Hub Kubernetes プロバイダーの機能と接続に関する問題。
-  ロード バランサー、ネットワーク セキュリティ グループ、VNet、サブネット、ネットワーク インターフェイス、ルート テーブル、可用性セット、パブリック IP アドレス、ストレージ アカウント、VM マシンなどの Azure Stack Hub ネイティブな成果物の、AKS エンジンによって生成された構成に関する問題。 
-  ネットワーク パフォーマンスと待ち時間に関する問題。
-  切断されたデプロイで AKS エンジンによって使用される AKS 基本イメージに関する問題 

## <a name="aks-engine-areas-not-supported"></a>AKS エンジンのサポートされていない領域

以下に関するテクニカル サポートは、Microsoft からは提供されません。

-  Azure での AKS エンジンの使用。
-  Azure Stack Hub Kubernetes Marketplace の項目。
-  次の AKS エンジン クラスター定義オプションとアドオンの使用。
    -  サポートされていないアドオン:  
            -  Azure AD ポッド ID  
            -  ACI Connector  
            -  Blobfuse Flex Volume  
            -  Cluster Autoscaler  
            -  Container Monitoring  
            -  KeyVault Flex Volume  
            -  NVIDIA Device Plugin  
            -  Rescheduler  
            -  SMB Flex Volume  
        
    -  サポートされていないクラスター定義オプション:  
            -  KubernetesConfig の下:  
                    -  cloudControllerManagerConfig  
                    -  enableDataEncryptionAtRest  
                    -  enableEncryptionWithExternalKms  
                    -  enablePodSecurityPolicy  
                    -  etcdEncryptionKey  
                    -  useInstanceMetadata  
                    -  useManagedIdentity  
                    -  azureCNIURLLinux  
                    -  azureCNIURLWindows  
            -  masterProfile の下:  
                    -  availabilityZones  
            -  agentPoolProfiles の下:  
                    -  availabilityZones  
                    -  singlePlacementGroup  
                    -  scaleSetPriority  
                    -  scaleSetEvictionPolicy  
                    -  acceleratedNetworkingEnabled  
                    -  acceleratedNetworkingEnabledWindows

-  Kubernetes 構成の変更は、Kubernetes 構成ストアの etcd の外部で保持されます。 たとえば、クラスターのノードで実行されている静的ポッドなどです。
-  Kubernetes の使用方法に関する質問。 たとえば、Microsoft サポートでは、カスタムのイングレス コント ローラーの作成方法、アプリケーションのワークロードの使用方法、サード パーティまたはオープン ソースのソフトウェア パッケージやツールの適用方法に対するアドバイスは提供していません。
-  AKS エンジンによってデプロイされた Kubernetes クラスターの一部として提供されない、サードパーティのオープンソース プロジェクト。 これらのプロジェクトには、Kubeadm、Kubespray、Native、Istio、Helm、Envoy などが含まれる場合があります。
-  サードパーティ製ソフトウェア。 このソフトウェアには、セキュリティ スキャン ツール、およびネットワーク デバイスまたはソフトウェアが含まれる場合があります。
-  マルチクラウドまたはマルチベンダーの構築に関する問題。 たとえば、Microsoft では、フェデレーション マルチパブリック クラウド ベンダーのソリューションの実行に関連する問題はサポートしていません。
-  「[AKS エンジンのサポートされている領域](#aks-engine-supported-areas)」に記載されている以外のネットワークのカスタマイズ。
-  運用環境では、可用性の高い Kubernetes クラスター、つまり、少なくとも 3 つのマスター ノードと 3 つのエージェント ノードを使用してデプロイされたクラスターのみを使用する必要があります。 運用環境のデプロイでは、これより少ないものはサポートされません。

##  <a name="security-issues-and-patching"></a>セキュリティの問題と修正プログラムの適用

AKS エンジンまたは Azure Stack Hub 用 Kubernetes プロバイダーの 1 つ以上のコンポーネントでセキュリティ上の欠陥が見つかった場合、Microsoft は問題を軽減するために、影響を受けるクラスターを修正するための修正プログラムを提供します。 または、チームがユーザーにアップグレード ガイダンスを示します。 修正プログラムではクラスターのダウンタイムが必要になる場合があることに注意してください。 再起動が必要な場合、Microsoft はこの要件をお客様に通知します。 ユーザーが Microsoft のガイダンスに従って修正プログラムを適用しない場合、クラスターはセキュリティ問題に対して脆弱なままになります。

## <a name="kubernetes-marketplace-item"></a>Kubernetes Marketplace 項目

ユーザーは、Kubernetes Marketplace 項目をダウンロードできます。これにより、Azure Stack Hub ユーザー ポータルのテンプレートを介して AKS エンジンを間接的に使用することで Kubernetes クラスターをデプロイできます。 この方法を使用すると、AKS エンジンを直接使用するよりも簡単になります。 Kubernetes Marketplace 項目は、デモンストレーション、テスト、および開発のためにクラスターをすばやく設定するのに便利なツールです。 これは運用環境向けではないため、Microsoft によってサポートされる項目のセットには含まれていません。

## <a name="preview-features"></a>プレビュー機能

拡張テストとユーザー フィードバックを必要とする機能の場合、Microsoft では、新しいプレビュー機能または機能フラグの背後にある機能をリリースします。 これらの機能は、プレリリースまたはベータ機能としてお考えください。 プレビュー機能または機能フラグは、運用環境向けではありません。 機能の継続的な変更と動作、バグの修正、その他の変更により、クラスターが不安定になったり、ダウンタイムが生じたりする可能性があります。 これらの機能は Microsoft ではサポートされていません。

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub 上の AKS エンジン](azure-stack-kubernetes-aks-engine-overview.md)を確認してください