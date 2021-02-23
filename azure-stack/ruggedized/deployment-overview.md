---
title: Azure Stack Hub ハードウェア ライフサイクル ホスト (HLH) 管理サーバー向けの Azure Stack Hub ラグドの配置と設定 | Microsoft Docs
description: 計画から配置後まで、Azure Stack Hub ラグドの適切なオンサイト配置のために想定されることについて説明します。
services: azure-stack
documentationcenter: ''
author: ashika789
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
ms.openlocfilehash: 4d61fc75cd7db67f113369981fe0fa4742178dd5
ms.sourcegitcommit: 4c97ed2caf054ebeefa94da1f07cfb6be5929aac
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/18/2021
ms.locfileid: "100648067"
---
# <a name="azure-stack-hub-ruggedized-deployment-overview"></a>Azure Stack Hub ラグドの配置の概要

この配置ガイドでは、Azure Stack Hub ラグドを設置し、構成する手順について説明します。 

このガイドの目的は次のとおりです。

- コンポーネントをインストールする前に、すべての前提条件が満たされていることを確認するための、デプロイ前チェックリストを提供します。
- Azure Stack Hub ラグドの主要コンポーネントを紹介します。
- 主要なコンポーネントをインストールして構成する方法について説明します。
- お客様によるデプロイを検証します。

この配置ガイドは、顧客サイトで Azure Stack Hub ラグドの配置を担当する Microsoft フィールド プロフェッショナル チームを対象としています。

このガイドの内容を完全に理解するには、仮想化、サーバー、オペレーティング システム、ネットワーク、ストレージ ソリューションについての技術的な経験が必要です。 デプロイ エンジニアは、Hyper-V、Azure Stack Hub、Azure、および Microsoft PowerShell を使用する Microsoft Windows Server 2019 に関する知識を持っている必要があります。

このガイドの中心的なテーマは、Microsoft Azure Stack Hub のコア コンポーネントのデプロイと、Azure Stack Hub ラグド ソリューションの詳細です。 このガイドでは、Azure Stack Hub の操作手順と、Azure Stack Hub で利用できるすべての機能については説明しません。 詳細については、[Azure Stack Hub オペレーター ガイド](../operator/index.yml)に関するページを参照してください。

## <a name="introduction"></a>はじめに

Azure Stack Hub ラグドは、Microsoft Azure Stack Hub 向けの耐久性が高くフィールド配置可能なオファリングです。 サーバーやスイッチなどのコア コンポーネントは、ポッドと呼ばれる運搬用ケースに格納されています。

ポッドは、通常の 4U ラックよりも小さい寸法の 4U ラック コンテナーです。 1 つの管理ポッドと 2 つのスケール ユニット (SU) ポッドがあります。 管理ポッドには、ハードウェア ライフサイクル ホスト (HLH)、2 つの 25 GbE トップオブラック (ToR) スイッチ、1 つのベースボード管理コントローラー (BMC) スイッチが含まれています。

各 SU ポッドには、2 つの Azure Stack Hub ラグドの R640 SU サーバーが含まれています。 1 台の R640 サーバーは、ポッド内の 2U ラック スペースを占有します。 配置時に、SU ポッド内のサーバーは管理ポッド内の BMC および ToR スイッチに接続されます。

## <a name="terminology"></a>用語

次の表に、このガイドで使用されている用語の一部を示します。

|期間   | 定義 |
|-------|------------|
|ハードウェア ライフサイクル ホスト (HLH)| HLH は、最初のデプロイ ブートストラップと、Azure Stack Hub インフラストラクチャの継続的なハードウェア管理、サポート、バックアップに使用される、物理サーバーです。 HLH においては、デスクトップ エクスペリエンスと Hyper-V ロールを備えた Windows Server 2019 が実行されます。 サーバーは、ハードウェア管理ツール、スイッチ管理ツール、Azure Stack Hub パートナー ツールキット、デプロイ仮想マシンをホストするために使用されます。 |
|デプロイ仮想マシン (DVM)|  DVM は、Azure Stack Hub ソフトウェアのデプロイの間だけ HLH 上に作成される仮想マシンです。 DVM により、ネットワーク経由ですべての Azure Stack Hub スケール ユニット サーバーに Azure Stack Hub ファブリック インフラストラクチャ ソフトウェアをインストールして構成するための、エンタープライズ クラウド エンジン (ECE) と呼ばれる Azure Stack Hub ソフトウェア オーケストレーション エンジンが実行されます。|
|Azure Stack Hub パートナー ツールキット|   顧客固有の入力パラメーターをキャプチャし、Azure Stack Hub インストールと構成を開始するために使用される、ソフトウェア ツールのコレクション。 デプロイ ワークシートが含まれています。これは、Azure Stack Hub のインストールの構成可能なパラメーターをキャプチャして格納するために使用されるグラフィカル ユーザー インターフェイス (GUI) ツールです。 また、デプロイ ワークシートの入力を使用して、ソリューション内のすべての物理ネットワーク デバイスに対するネットワーク構成ファイルを生成する、ネットワーク構成ジェネレーター ツールも含まれています。|
|OEM 拡張機能パッケージ  |初期のデプロイと更新の間に Azure Stack Hub によって使用される特殊な形式の、ファームウェア、デバイス ドライバー、ハードウェア管理ツールのパッケージ。|
|Integrated Dell Remote Access Controller (iDRAC)|  ライフサイクル コントローラー搭載の iDRAC は、すべての Azure Stack Hub ラグド R640 サーバーに組み込まれているベースボード管理コントローラーです。 iDRAC には、Azure Stack Hub サーバーの配置、更新、監視、および保守を支援する帯域外の管理機能があります。|
|スケール ユニット |Azure Stack Hub のファブリック インフラストラクチャとワークロードに対してコンピューティング リソースとストレージ リソースを提供する、Azure Stack Hub のコア コンポーネント。 4 台の Azure Stack Hub ラグド R640 サーバー (ノードとも呼ばれます) で構成され、最大 16 ノードまで動的にスケーリングすることができます。|
|Pod    |Azure Stack Hub ラグドのコンテキストでは、ポッドは 2 人で運ぶように設計された耐久性の高い物理コンテナーであり、Azure Stack Hub ラグド ハードウェアを物理的な環境ストレスから保護するためのラック マウント ブラケットとショック アブソーバーが含まれています。 ハードウェアを輸送する場合に取り付けて密封するできる前面および背面の運送用ケース カバーが含まれています。 最小構成のソリューション一式には、3 つのポッドが含まれています。|


## <a name="deployment-workflow"></a>デプロイのワークフロー

大まかに言えば、MDC のデプロイ プロセスは以下のセクションで説明するフェーズで構成されています。

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
