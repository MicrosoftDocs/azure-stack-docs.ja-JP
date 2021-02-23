---
title: Modular Data Center (MDC) のデプロイの概要と、Azure Stack Hub ハードウェア ライフサイクル ホスト (HLH) 管理サーバー用の設定 | Microsoft Docs
description: 計画からデプロイ後まで、Modular Data Center (MDC) のオンサイトのデプロイを成功させるために必要なことについて説明します。
services: azure-stack
documentationcenter: ''
author: asganesh
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/17/2021
ms.author: patricka
ms.reviewer: asganesh
ms.lastreviewed: 02/17/2021
ms.openlocfilehash: ad0a80f28b26e3c7da71860670feefd7efc15a0f
ms.sourcegitcommit: 4c97ed2caf054ebeefa94da1f07cfb6be5929aac
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/18/2021
ms.locfileid: "100647795"
---
# <a name="mdc-requirements-overview"></a>MDC の要件の概要

このガイドでは、Modular Data Center (MDC) をインストールして構成するために必要な要件について説明します。 

このガイドの目的は次のとおりです。

- コンポーネントをインストールする前に、すべての前提条件が満たされていることを確認するための、デプロイ前チェックリストを提供します。
- MDC の主要コンポーネントについて説明します。
- お客様によるデプロイを検証します。

このガイドの内容を完全に理解するには、仮想化、サーバー、オペレーティング システム、ネットワーク、ストレージ ソリューションについての技術的な経験が必要です。 

このガイドの中心的なテーマは、Microsoft Azure Stack Hub のコア コンポーネントのデプロイと、MDC ソリューションの詳細です。 このガイドでは、Azure Stack Hub の操作手順と、Azure Stack Hub で利用できるすべての機能については説明しません。 

## <a name="introduction"></a>はじめに

MDC は、標準の 40 フィート金属製出荷コンテナーに収められている Azure Stack Hub 用の統合オファリングです。 コンテナーには、環境制御ユニット、照明、および警告システムが含まれています。 Azure Stack Hub のコア コンポーネントは、3 つの独立したポッドとしてインストールされます: ポッド 1 とラック 1 とラック 2、ポッド 2 とラック 1 とラック 2、ポッド 3 とラック 1 とラック 2。

各ポッドは、2 つの 42U ラックで構成されます。 ポッドには、Top-of-Rack (ToR) スイッチ、エッジ スイッチ、およびベースボード管理コントローラー (BMC) スイッチが含まれています。 また、各ポッドには、ハードウェア ライフサイクル ホスト (HLH) とシリアル ポート コンセントレーターが含まれます。 コンピューティングとストレージのコア容量は、8 つの Rugged Edge Appliance (REA) R840 サーバーで構成される Azure Stack Hub スケール ユニット (SU) によって提供されます。 48 個の Isilon ストレージ ノードによって、追加のストレージ容量が提供されます。 物理的な構成はすべてのポッドで同じです。

## <a name="terminology"></a>用語

次の表に、このガイドで使用されている用語の一部を示します。

|期間    |定義 |
|-------|-----------|
|ハードウェア ライフサイクル ホスト (HLH)|    HLH は、最初のデプロイ ブートストラップと、Azure Stack Hub インフラストラクチャの継続的なハードウェア管理、サポート、バックアップに使用される、物理サーバーです。 HLH においては、デスクトップ エクスペリエンスと Hyper-V ロールを備えた Windows Server 2019 が実行されます。 サーバーは、ハードウェア管理ツール、スイッチ管理ツール、Azure Stack Hub パートナー ツールキット、デプロイ仮想マシンをホストするために使用されます。 |
|デプロイ仮想マシン (DVM)|    DVM は、Azure Stack Hub ソフトウェアのデプロイの間だけ HLH 上に作成される仮想マシンです。 DVM により、ネットワーク経由ですべての Azure Stack Hub スケール ユニット サーバーに Azure Stack Hub ファブリック インフラストラクチャ ソフトウェアをインストールして構成するための、エンタープライズ クラウド エンジン (ECE) と呼ばれる Azure Stack Hub ソフトウェア オーケストレーション エンジンが実行されます。|
|Azure Stack Hub パートナー ツールキット|    顧客固有の入力パラメーターをキャプチャし、Azure Stack Hub インストールと構成を開始するために使用される、ソフトウェア ツールのコレクション。 デプロイ ワークシートが含まれています。これは、Azure Stack Hub のインストールの構成可能なパラメーターをキャプチャして格納するために使用されるグラフィカル ユーザー インターフェイス (GUI) ツールです。 また、デプロイ ワークシートの入力を使用して、ソリューション内のすべての物理ネットワーク デバイスに対するネットワーク構成ファイルを生成する、ネットワーク構成ジェネレーター ツールも含まれています。|
|OEM 拡張機能パッケージ    |初期のデプロイと更新の間に Azure Stack Hub によって使用される特殊な形式の、ファームウェア、デバイス ドライバー、ハードウェア管理ツールのパッケージ。|
|シリアル ポート コンセントレーター    |各ポッドにインストールされ、デプロイと管理のためにネットワーク スイッチのシリアル ポートへのネットワーク アクセスを提供する物理デバイス。|
|スケール ユニット    |Azure Stack Hub のファブリック インフラストラクチャとワークロードに対してコンピューティング リソースとストレージ リソースを提供する、Azure Stack Hub のコア コンポーネント。 各ポッドには、ノードとも呼ばれる 8 個の MDC R840 サーバーが含まれています。|
|Isilon ストレージ |    MDC ソリューションに固有の Azure Stack Hub コンポーネント。 Isilon により、Azure Stack Hub ワークロード用の追加の BLOB およびファイル ストレージが提供されます。 各ポッドには、48 個の Isilon ストレージ ノードが含まれます。|
|Pod    |MDC のコンテキストの場合、ポッドは 2 つの相互接続された物理ラックで構成される独立した論理ユニットです。 完全なソリューションには、1 つのコンテナーに設置された 3 つのポッドが含まれます。|

## <a name="deployment-workflow"></a>デプロイのワークフロー

大まかに言えば、MDC のデプロイ プロセスは次のフェーズで構成されます。

### <a name="planning-phase"></a>計画フェーズ
1. データセンターの電力の計画。
1. Azure Stack Hub の論理ネットワーク構成の計画。
1. [データセンターのネットワーク統合](../operator/azure-stack-network.md)の計画。
1. [ID](../operator/azure-stack-identity-overview.md) の統合の計画。
1. [セキュリティ](../operator/azure-stack-security-foundations.md)の統合の計画。
1. [PKI 証明書](../operator/azure-stack-pki-certs.md)の計画。

### <a name="preparation-phase"></a>準備フェーズ
1. インベントリの収集。
1. 電源の接続と、ソリューションの電源オン。
1. HVAC システムの正常性の検証。
1. 火災監視と警告システムの正常性の検証。
1. 物理ハードウェアの正常性の検証。

### <a name="execution-phase--separately-for-each-of-the-three-pods"></a>実行フェーズ – 3 つのポッドに対して個別に実行
1. ハードウェア ライフサイクル ホストの構成。
1. ネットワーク スイッチの構成。
1. データセンターのネットワークの統合。
1. 物理ハードウェアの設定の構成。
1. Isilon ストレージの構成。
1. Azure Stack Hub ファブリック インフラストラクチャのデプロイ。
1. データセンターの ID の統合。
1. 拡張機能用のアドオンのインストール。

### <a name="validation-phase--separately-for-each-of-the-three-pods"></a>検証フェーズ – 3 つのポッドに対して個別に実行
1. デプロイ後の正常性の検証。
1. Microsoft への Azure Stack Hub の登録。
1. Azure Stack Hub のお客様への引き継ぎ。