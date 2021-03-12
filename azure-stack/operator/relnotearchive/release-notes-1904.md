---
title: Azure Stack 1904 リリース ノート | Microsoft Docs
description: Azure Stack 統合システムの 1904 更新プログラムについて説明します
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
ms.date: 05/30/2019
ms.author: sethm
ms.reviewer: hectorl
ms.lastreviewed: 05/30/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 9ca014878021d0fbe02a8997e1436910557e93f5
ms.sourcegitcommit: e432e7f0a790bd6419987cbb5c5f3811e2e7a4a2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/09/2021
ms.locfileid: "102515925"
---
# <a name="azure-stack-1904-update"></a>Azure Stack 1904 更新プログラム

*適用対象:Azure Stack 統合システム*

この記事では、1904 更新プログラム パッケージの内容について説明します。 この更新プログラムには、このリリースの Azure Stack に対する新機能、機能強化、および修正が含まれています。 この記事には、次の情報が含まれています。

> [!IMPORTANT]  
> 更新プログラム パッケージは、Azure Stack 統合システム専用です。 Azure Stack Development Kit にこの更新プログラム パッケージは適用しないでください。

## <a name="build-reference"></a>ビルドのリファレンス

Azure Stack 1904 更新プログラムのビルド番号は **1.1904.0.36** です。

### <a name="update-type"></a>更新の種類

Azure Stack 1904 更新プログラムのビルドの種類は **高速** です。 更新プログラムのビルドの種類については、「[Azure Stack での更新プログラムの管理概要](../azure-stack-updates.md)」を参照してください。 1904 更新プログラムが完了するまでの予測所要時間は約 16 時間ですが、正確な時間は変わる可能性があります。 このおおよその実行時間は、1904 更新プログラムに固有であり、他の Azure Stack 更新プログラムと比較することはできません。

## <a name="whats-in-this-update"></a>この更新プログラムの新機能

<!-- The current theme (if any) of this release. -->

<!-- What's new, also net new experiences and features. -->

### <a name="improvements"></a>機能強化

<!-- Changes and product improvements with tangible customer-facing value. -->

- 1904 では、ソフトウェア定義ネットワーク (SDN) スタックが大幅に強化されました。 これらの機能強化により、Azure Stack の SDN スタックの全体的なサービスと信頼性が向上します。

- 現在ログインしているユーザーに必要なアクセス許可がない場合の通知を管理者ポータルに追加しました。これで、ダッシュボードを適切に読み込むことができるようになります。 また、デプロイ中に使用された ID プロバイダーに応じて、適切なアクセス許可を持つアカウントについて説明したドキュメントのリンクも含まれています。

- VM の回復性と稼働時間の機能強化が追加されました。これにより、VM の構成ファイルを含むストレージ ボリュームがオフラインになった場合にすべての VM がオフラインになるシナリオが解決されます。

<!-- 1901,2,3 related hotfix -->
- ネットワークに大きな負荷がかかっている場合の VM の電圧低下または停電に対処するために、同時に退避させる VM の数の最適化を追加し、消費される帯域幅に上限を設定しました。 この変更により、システム更新時の VM の稼働時間が長くなります。

<!-- 1901,2,3 related hotfix -->
- 内部プロセスによってプラットフォーム リソースがすべて使用され、その結果、ポータルでの操作が失敗することを防ぐために、システムが大規模に実行されているときのリソースの調整機能を強化しました。

- フィルター機能が強化され、オペレーターは同時に複数のフィルターを適用できるようになりました。 新しいユーザー インターフェイスでは **[名前]** 列のみで並べ替えることができます。

- オファー、プラン、クォータ、およびサブスクリプションの削除プロセスの改良。 削除するオブジェクトに依存関係がない場合は、管理者ポータルからオファー、クォータ、プラン、およびサブスクリプションを正常に削除できるようになりました。 詳細については、 [こちらの記事](../azure-stack-delete-offer.md)を参照してください。  

<!-- this applies to bug 3725384 and bug #4225643 -->
- 不要なイベントを除外し、転送されるメッセージに目的の重大度を選択する構成パラメーターを用意することで、syslog メッセージの量を改良しました。 重大度の構成方法の詳細については、「[Azure Stack datacenter integration - syslog forwarding](../azure-stack-integrate-security.md)」(Azure Stack データセンターの統合 - syslog 転送) を参照してください。

<!--this applied to Bug 1473487 -->
- 追加のパラメーター `-OutputSASUri` を組み込むことで、**Get-AzureStackLog** コマンドレットに新しい機能を追加しました。 これで、環境から Azure Stack ログを収集し、指定した Azure Storage Blob コンテナーに保存できるようになりました。 詳細については、「[Azure Stack の診断](/azure-stack/operator/diagnostic-log-collection)」を参照してください。

- **Test-AzureStack** `UpdateReadiness` グループに新しいメモリ チェックを追加しました。これは、更新プログラムを正常に完了できる十分なメモリがスタック上にあるかどうかを確認する機能です。

<!-- Bug/Task 4311058 -->
- Service Fabric の正常性を評価するための **Test-AzureStack** の機能強化。

<!-- feature: 2976966 -->
- ハードウェア更新プログラムの機能強化。ドライブ ファームウェアの更新プログラムを完了するまでにかかる時間が 2 - 4 時間に短縮されました。 更新プログラム エンジンでは、パッケージの内容に基づいて、更新プログラムのどの部分を実行する必要があるかが動的に判断されます。

<!-- Feature 3906611 -->
- 可用性に影響を与える破壊的なインフラストラクチャ ロール インスタンス操作を防ぐために、堅牢な操作の事前チェックを追加しました。

<!-- Feature 3780326 -->
- インフラストラクチャのバックアップ アクション プランのべき等性の改良。

<!--Bug/Task 3139609 -->
- Azure Stack のログ収集の機能強化。 これらの機能強化により、一連のログの取得にかかる時間が短縮されます。 また、[Get-AzureStackLog](/azure-stack/operator/relnotearchive/azure-stack-configure-on-demand-diagnostic-log-collection-portal#run-get-azurestacklog-on-an-azure-stack-development-kit-asdk-system) コマンドレットから OEM ロールの既定のログが生成されなくなりました。 OEM ログを取得するロールを指定して、[Invoke-AzureStackOnDemandLog](/azure-stack/operator/relnotearchive/azure-stack-configure-on-demand-diagnostic-log-collection-portal#invoke-azurestackondemandlog) コマンドレットを実行する必要があります。 詳細については、「[Azure Stack の診断](/azure-stack/operator/relnotearchive/azure-stack-configure-on-demand-diagnostic-log-collection-portal#run-get-azurestacklog-on-an-azure-stack-development-kit-asdk-system)」を参照してください。

- Azure Stack では、データセンターと ADFS の統合のために提供されているフェデレーション データ URL が監視されるようになりました。 これにより、お客様の ADFS インスタンスまたはファームのシークレット ローテーション中の信頼性が向上します。

### <a name="changes"></a>[変更点]

<!-- Feature 3906611 -->
- Azure Stack オペレーターが管理者ポータルのインフラストラクチャ ロール インスタンスをシャットダウンするオプションを削除しました。 再起動機能により、インフラストラクチャ ロール インスタンスを再起動する前にクリーン シャットダウンが確実に試行されます。 高度なシナリオでは、API と PowerShell の機能を引き続き使用できます。

<!-- Feature ## 4199257 -->
- Marketplace 管理エクスペリエンスが新しくなり、Marketplace イメージ用とリソース プロバイダー用に個別の画面が表示されるようになりました。 現在のところ、**[リソース プロバイダー]** ウィンドウは空ですが、今後のリリースでは新しい PaaS サービス オファリングが表示され、**[リソース プロバイダー]** ウィンドウで管理されるようになる予定です。

<!-- Feature ## 4199257 -->
- オペレーター ポータルでの更新プログラム エクスペリエンスの変更。 リソース プロバイダーの更新プログラム用の新しいグリッドがあります。 リソース プロバイダーを更新する機能はまだ利用できません。

<!-- Task ## 3748423  -->
- オペレーター ポータルでの更新プログラム インストール エクスペリエンスの変更。 Azure Stack オペレーターが更新プログラムの問題に適切に対応できるように、**Test-AzureStack** を実行して結果を解析することで自動的に導き出されるスケール ユニットの正常性に基づいて、より具体的な推奨事項がポータルに表示されるようになりました。 結果に基づいて、2 つのアクションのうちいずれかを実行するようにオペレーターに通知されます。

  - "ソフト" な警告アラートの場合、ポータルに "The most recent update needs attention. Microsoft recommends opening a service request during normal business hours. As part of the update process, Test-AzureStack is performed, and based on the output we generate the most appropriate alert. In this case, Test-AzureStack passed." (最新の更新プログラムには注意が必要です。マイクロソフトは通常の営業時間中にサポート リクエストを開くことをお勧めします。更新プロセスの一環として Test-AzureStack が実行され、その出力に基づいて最も適切なアラートが生成されます。このケースでは、Test-AzureStack は合格しています。)

  - "ハード" で重大なアラートの場合、ポータルに "The most recent update failed. Microsoft recommends opening a service request as soon as possible. As part of the update process, Test-AzureStack is performed, and based on the output we generate the most appropriate alert. In this case, Test-AzureStack also failed." (最新の更新プログラムは失敗しました。マイクロソフトはできるだけ早くサポート リクエストを開くことをお勧めします。更新プロセスの一環として Test-AzureStack が実行され、その出力に基づいて最も適切なアラートが生成されます。このケースでは、Test-AzureStack も失敗しています。)

- Azure Linux Agent バージョン 2.2.38.0 を更新しました。 このサポートにより、お客様は Azure と Azure Stack との間で一貫性のある Linux イメージを保持できるようになります。

- オペレーター ポータルでの更新ログの変更。 成功した更新ログを取得する要求は利用できなくなりました。 失敗した更新ログは、診断のためのアクションにつながるため、引き続きダウンロードできます。

### <a name="fixes"></a>修正

<!-- Product fixes that came up from customer deployments worth highlighting, especially if there is an SR/ICM associated to it. -->

- syslog の構成が更新サイクル全体で保持されず、その結果、syslog クライアントがその構成を失い、syslog メッセージが転送されなくなる問題を修正しました。 syslog の構成が保持されるようになりました。

- VM の割り当て解除がブロックされる CRP の問題を修正しました。 以前は、VM に複数の大きなマネージド ディスクが含まれている場合、VM の割り当て解除がタイムアウト エラーで失敗することがありました。

- スケール ユニット ストレージへのアクセスに影響を与える Windows Defender エンジンに関する問題を修正しました。

- BLOB Storage アカウントの [アクセス ポリシー] ウィンドウの読み込みに失敗するユーザー ポータルの問題を修正しました。

- グローバル Azure portal について不適切な通知が表示されるという、管理者ポータルとユーザー ポータルの両方の問題を修正しました。

- **[フィードバック]** タイルを選択すると空のブラウザー タブが開くユーザー ポータルの問題を修正しました。

- VM インスタンスにアタッチされたネットワーク アダプターにバインドされている IP 構成の静的 IP アドレスを変更すると、エラー メッセージが表示されるというポータルの問題を修正しました。

- **[ネットワーク]** ウィンドウを介して既存の VM に対して **[ネットワーク インターフェイスの接続]** を実行しようとすると、エラー メッセージが表示されて操作が失敗するというユーザー ポータルの問題を修正しました。

- Azure Stack が VM インスタンスへの 4 つを超えるネットワーク インターフェイス (NIC) のアタッチをサポートしていなかった問題を修正しました。

- 受信セキュリティ規則を追加し、ソースとして **[Service Tag]\(サービス タグ\)** を選択すると、Azure Stack には使用できないいくつかのオプションが表示されるというポータルの問題を修正しました。

- ネットワーク セキュリティ グループ (NSG) がグローバル Azure と同様に Azure Stack で機能しなかった問題を修正しました。

- 登録の期限が切れた場合、または削除された場合に、ダウンロードされたすべての製品が非表示になる Marketplace 管理の問題を修正しました。

- 既存の仮想ネットワーク ゲートウェイ接続に対して PowerShell で **Set-AzureRmVirtualNetworkGatewayConnection** コマンドを発行すると、エラー メッセージ "**無効な共有キーが構成されています...**" が表示されて失敗する問題を修正しました。

- ネットワーク リソース プロバイダー (NRP) がネットワーク コントローラーと同期しなくなり、その結果、重複するリソースが要求される問題を修正しました。 場合によっては、結果として親リソースがエラー状態のままになります。

- サブスクリプションに共同作成者ロールが割り当てられていても読み取りアクセス許可が明示的に与えられていないユーザーが、リソースの変更を保存しようとすると、"**オブジェクト ID 'GUID' のクライアント 'somelogonaccount@domain.com' は、... アクション '...' の実行を承認されていません...**" というエラーが生成されていた問題を修正しました。

- オフライン シンジケーション ツールを使用してイメージをアップロードしたときに、マーケットプレース管理画面に何も表示されず、いずれにもアイコンの URI が表示されない問題を修正しました。

- ダウンロードに失敗した製品が Marketplace Management から削除されるのを妨げる問題を修正しました。

### <a name="security-updates"></a>セキュリティ更新プログラム

Azure Stack のこの更新プログラムには、Azure Stack をホストする、基になるオペレーティング システムに対するセキュリティ更新プログラムは含まれていません。

## <a name="update-planning"></a>計画の更新

更新プログラムを適用する前に、必ず次の情報を確認してください。

- [既知の問題](known-issues-1904.md)
- [セキュリティ更新プログラム](../release-notes-security-updates.md)
- [更新プログラム適用前後のアクティビティのチェックリスト](../release-notes-checklist.md)

> [!NOTE]
> ワークロードの計画とサイズ設定を行うには、最新バージョンの [Azure Stack Capacity Planner](https://aka.ms/azstackcapacityplanner) ツールを使用します。 最新バージョンではバグの修正が含まれ、Azure Stack の各更新プログラムでリリースされる新機能が提供されています。

## <a name="download-the-update"></a>更新プログラムをダウンロードする

Azure Stack 1904 更新プログラム パッケージは [Azure Stack ダウンロード ページ](https://aka.ms/azurestackupdatedownload)からダウンロードできます。

## <a name="hotfixes"></a>修正プログラム

Azure Stack では、修正プログラムが定期的にリリースされます。 Azure Stack を 1904 に更新する前に、必ず 1903 用の最新の Azure Stack 修正プログラムをインストールしてください。

Azure Stack 修正プログラムを適用できるのは Azure Stack 統合システムのみです。ASDK には修正プログラムをインストールしないでください。

### <a name="before-applying-the-1904-update"></a>1904 更新プログラムを適用する前

次の修正プログラムが適用された 1903 リリースに、Azure Stack の 1904 リリースを適用する必要があります。

<!-- One of these. Either no updates at all, nothing is required, or the LATEST hotfix that is required-->
- [Azure Stack 修正プログラム 1.1903.2.39](https://support.microsoft.com/help/4500638)

### <a name="after-successfully-applying-the-1904-update"></a>1904 更新プログラムの適用に成功した後

この更新プログラムをインストールした後、適用可能な修正プログラムがあればインストールします。 詳細については、[サービス ポリシー](../azure-stack-servicing-policy.md)に関する記事を参照してください。

- [Azure Stack 修正プログラム 1.1904.4.45](https://support.microsoft.com/help/4505688)

## <a name="automatic-update-notifications"></a>自動更新通知

インフラストラクチャ ネットワークからインターネットにアクセスできるシステムをご利用の場合は、オペレーター ポータルに "**利用可能な更新プログラムがあります**" というメッセージが表示されます。 インターネットにアクセスできないシステムでは、対応する .xml を含む .zip ファイルをダウンロードしてインポートできます。

> [!TIP]  
> 以下の *RSS* または *Atom* フィードに登録して、Azure Stack 修正プログラムの最新情報を入手してください。
>
> - [RSS](https://support.microsoft.com/app/content/api/content/feeds/sap/en-us/32d322a8-acae-202d-e9a9-7371dccf381b/rss)
> - [Atom](https://support.microsoft.com/app/content/api/content/feeds/sap/en-us/32d322a8-acae-202d-e9a9-7371dccf381b/atom)

## <a name="next-steps"></a>次のステップ

- Azure Stack での更新プログラム管理の概要については、「[Azure Stack での更新プログラムの管理概要](../azure-stack-updates.md)」を参照してください。  
- Azure Stack に更新プログラムを適用する方法については、「[Azure Stack で更新を適用する](../azure-stack-apply-updates.md)」を参照してください。
- Azure Stack 統合システムのサービス ポリシーについて、およびサポートを受けられる状態にシステムを維持するために必要な作業について確認するには、「[Azure Stack サービス ポリシー](../azure-stack-servicing-policy.md)」を参照してください。  
- 特権エンドポイント (PEP) を使用して更新プログラムを監視および再開するには、「[特権エンドポイントを使用して Azure Stack での更新プログラムをモニターする](../azure-stack-monitor-update.md)」をご覧ください。
