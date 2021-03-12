---
title: Azure Stack 1906 リリース ノート | Microsoft Docs
description: Azure Stack 統合システム 1906 の更新プログラムについて説明します
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/29/2019
ms.author: sethm
ms.reviewer: prchint
ms.lastreviewed: 10/29/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 9a51d7c7b1470e95fd9c8b6905c2ac31f59bf7b9
ms.sourcegitcommit: f194f9ca4297864500e62d8658674a0625b29d1d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102187250"
---
# <a name="azure-stack-updates-1906-release-notes"></a>Azure Stack の更新プログラム: 1906 リリース ノート

この記事では、Azure Stack 更新プログラム パッケージの内容について説明します。 この更新プログラムには、このリリースの Azure Stack に対する新機能、機能強化、および修正が含まれています。

別のバージョンのリリース ノートにアクセスするには、左側の目次の上部にあるバージョン セレクターのドロップダウンを使用します。

> [!IMPORTANT]  
> 更新プログラム パッケージは、Azure Stack 統合システム専用です。 Azure Stack Development Kit にこの更新プログラム パッケージは適用しないでください。

> [!IMPORTANT]  
> ご利用の Azure Stack インスタンスが 2 つ前の更新プログラムより遅れている場合、コンプライアンスに対応していないとみなされます。 [サポートを受けるためには、少なくともサポートされる最小バージョンまで更新する](../azure-stack-servicing-policy.md#keep-your-system-under-support)必要があります。

## <a name="update-planning"></a>計画の更新

更新プログラムを適用する前に、必ず次の情報を確認してください。

- [既知の問題](known-issues-1906.md)
- [セキュリティ更新プログラム](../release-notes-security-updates.md)
- [更新プログラム適用前後のアクティビティのチェックリスト](../release-notes-checklist.md)

更新プログラムのトラブルシューティングと更新プロセスの詳細については、[Azure Stack の修正プログラムと更新プログラムに関する問題のトラブルシューティング](../azure-stack-troubleshooting.md)に関するページをご確認ください。

## <a name="1906-build-reference"></a>1906 ビルドのリファレンス

Azure Stack 1906 更新プログラムのビルド番号は **1.1906.0.30** です。

### <a name="update-type"></a>更新の種類

Azure Stack 1906 更新プログラムのビルドの種類は **高速** です。 更新プログラムのビルドの種類については、「[Azure Stack での更新プログラムの管理概要](../azure-stack-updates.md)」を参照してください。 1906 更新プログラムが完了するまでの予測所要時間は、ご使用の Azure Stack 環境内の物理ノード数に関係なく、約 10 時間です。 更新プログラムの正確なランタイムは一般的に、ご使用のシステムでテナント ワークロードによって使用されている容量、システム ネットワーク接続 (インターネットに接続されている場合)、およびシステム ハードウェアの仕様に左右されます。 ランタイムがこの予測値よりも長くなることは一般的ではなく、更新が失敗した場合を除き、Azure Stack オペレーターによるアクションは不要です。 このおおよその実行時間は、1906 更新プログラムに固有であり、他の Azure Stack 更新プログラムと比較することはできません。

## <a name="whats-in-this-update"></a>この更新プログラムの新機能

<!-- The current theme (if any) of this release. -->

<!-- What's new, also net new experiences and features. -->

- すべてのエンドポイントで TLS 1.2 を強制するため、特権エンドポイント (PEP) に **Set-TLSPolicy** コマンドレットが追加されました。 詳しくは、[Azure Stack のセキュリティ コントロール](../azure-stack-security-configuration.md)に関するページを参照してください。

- 適用されている TLS ポリシーを取得するため、特権エンドポイント (PEP) に **Get-TLSPolicy** コマンドレットが追加されました。 詳しくは、[Azure Stack のセキュリティ コントロール](../azure-stack-security-configuration.md)に関するページを参照してください。

- システムの更新中に、必要に応じて、内部の TLS 証明書をローテーションする、内部シークレットのローテーション プロシージャが追加されました。

- 期限切れ間近のシークレットに関する重大なアラートが無視された場合に、内部シークレットのローテーションを強制することで、内部シークレットの有効期限切れを回避するための保護が追加されました。 これを通常の運用手順として依存しないでください。 シークレットのローテーションは、メンテナンス期間中に計画する必要があります。 詳しくは、[Azure Stack シークレットのローテーション](../azure-stack-rotate-secrets.md)に関するページを参照してください。

- AD FS を使用した Azure Stack のデプロイで Visual Studio Code がサポートされるようになりました。

### <a name="improvements"></a>機能強化

<!-- Changes and product improvements with tangible customer-facing value. -->

- 特権エンドポイントの **Get-GraphApplication** コマンドレットで、現在使用されている証明書の拇印が表示されるようになりました。 これにより、AD FS で Azure Stack がデプロイされるときのサービス プリンシパルの証明書の管理が改善されます。

- AD Graph と AD FS の可用性を検証するため、アラートを生成する機能を含む、新しい正常性の監視ルールが追加されました。

- インフラストラクチャ バックアップ サービスを別のインスタンスに移動するときに、バックアップ リソース プロバイダーの信頼性が向上しました。

- メンテナンス期間のスケジュール設定を円滑にするため、均一な実行時間を提供する外部シークレットのローテーション プロシージャのパフォーマンスが最適化されました。

- **Test-AzureStack** コマンドレットで、有効期限が近づいている (重大なアラート) 内部シークレットについて報告されるようになりました。

- 特権エンドポイントの **Register-CustomAdfs** コマンドレットで、AD FS に対してフェデレーションの信頼を構成するときに、証明書失効リストのチェックをスキップできるようにする新しいパラメーターが使用できるようになりました。

- 1906 リリースでは、更新プログラムが一時停止していないことが確認できるように、更新プログラムの進行状況の可視性が向上しています。 これにより、**[更新]** ブレードでオペレーターに表示される更新手順の合計数が増えました。 また、以前の更新プログラムよりも並列で行われる更新手順が増えています。

#### <a name="networking-updates"></a>ネットワークの更新

- DHCP レスポンダーに設定されているリース期間が、Azure と一致するように更新されました。

- リソースのデプロイに失敗したシナリオで、リソース プロバイダーへの再試行回数が改善されました。

- **Standard** SKU オプションは、現在サポートされていないため、ロード バランサーとパブリック IP の両方から削除されました。

### <a name="changes"></a>[変更点]

- ストレージ アカウントのエクスペリエンスの作成が、Azure と一致するようになりました。

- 内部シークレットの有効期限のアラート トリガーが変更されました。
  - 警告アラートが、シークレットの有効期限の 90 日前に表示されるようになりました。
  - 重大なアラートが、シークレットの有効期限の 30 日前に表示されるようになりました。

- 用語の一貫性のため、インフラストラクチャ バックアップ リソース プロバイダー内の文字列が更新されました。

### <a name="fixes"></a>修正

<!-- Product fixes that came up from customer deployments worth highlighting, especially if there is an SR/ICM associated to it. -->

- **内部操作エラー** でマネージド ディスク VM のサイズ変更が失敗する問題を修正しました。

- ユーザー イメージの作成の失敗により、イメージを管理するサービスが不適切な状態になり、これにより失敗したイメージの削除と新しいイメージの作成がブロックされる問題を修正しました。 これは 1905 修正プログラムでも修正されます。

- 期限切れ間近の内部シークレットのアクティブなアラートが、内部シークレットのローテーションが正常に実行された後に自動的に終了されるようになりました。

- 更新プログラムが 99 時間を超えて実行されていた場合、[更新履歴] タブの更新期間の最初の桁がトリミングされる問題を修正しました。

- **[更新]** ブレードに、失敗した更新プログラムの **[再開]** オプションが含まれるようになりました。

- 管理者ポータルとユーザー ポータルで、検索で Docker 拡張機能が不正に返されるが、Azure Stack では使用できないため、それ以上の操作を実行できないマーケットプレースでの問題を修正しました。

- テンプレートの名前がアンダー スコア ('_') で始まる場合、テンプレートのデプロイ UI でパラメーターが設定されない問題を修正しました。

- 仮想マシン スケール セットの作成エクスペリエンスで、デプロイのオプションとして CentOS-based 7.2 が提供される問題を修正しました。 CentOS 7.2 は Azure Stack では利用できません。 Centos 7.5 をデプロイのオプションとして提供するようになりました

- **[仮想マシン スケール セット]** ブレードからスケール セットを削除できるようになりました。

## <a name="security-updates"></a>セキュリティ更新プログラム

Azure Stack のこの更新でのセキュリティ更新プログラムについては、「[Azure Stack security updates](../release-notes-security-updates.md)」 (Azure Stack のセキュリティ更新プログラム) をご覧ください。

## <a name="download-the-update"></a>更新プログラムをダウンロードする

Azure Stack 1906 更新プログラム パッケージは、[Azure Stack ダウンロード ページ](https://aka.ms/azurestackupdatedownload)からダウンロードできます。

## <a name="hotfixes"></a>修正プログラム

Azure Stack では、修正プログラムが定期的にリリースされます。 Azure Stack を 1906 に更新する前に、必ず 1905 用の最新の Azure Stack 修正プログラムをインストールしてください。 更新後、[1906 に対して利用可能な修正プログラム](#after-successfully-applying-the-1906-update)があればインストールします。

Azure Stack 修正プログラムを適用できるのは Azure Stack 統合システムのみです。ASDK には修正プログラムをインストールしないでください。

### <a name="before-applying-the-1906-update"></a>1906 更新プログラムを適用する前

Azure Stack の 1906 リリースは、次の修正プログラムが適用された 1905 リリースに適用する必要があります。

<!-- One of these. Either no updates at all, nothing is required, or the LATEST hotfix that is required-->
- [Azure Stack 修正プログラム 1.1905.3.48](https://support.microsoft.com/help/4510078)

### <a name="after-successfully-applying-the-1906-update"></a>1906 更新プログラムの適用に成功した後

この更新プログラムをインストールした後、適用可能な修正プログラムがあればインストールします。 詳細については、[サービス ポリシー](../azure-stack-servicing-policy.md)に関する記事を参照してください。

<!-- One of these. Either no updates at all, nothing is required, or the LATEST hotfix that is required-->
- [Azure Stack 修正プログラム 1.1906.15.60](https://support.microsoft.com/help/4524559)

## <a name="next-steps"></a>次のステップ

- Azure Stack での更新プログラム管理の概要については、「[Azure Stack での更新プログラムの管理概要](../azure-stack-updates.md)」を参照してください。  
- Azure Stack に更新プログラムを適用する方法については、「[Azure Stack で更新を適用する](../azure-stack-apply-updates.md)」を参照してください。
- Azure Stack 統合システムのサービス ポリシーについて、およびサポートを受けられる状態にシステムを維持するために必要な作業について確認するには、「[Azure Stack サービス ポリシー](../azure-stack-servicing-policy.md)」を参照してください。  
- 特権エンドポイント (PEP) を使用して更新プログラムを監視および再開するには、「[特権エンドポイントを使用して Azure Stack での更新プログラムをモニターする](../azure-stack-monitor-update.md)」をご覧ください。