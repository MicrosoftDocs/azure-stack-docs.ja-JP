---
title: Azure Stack 1907 リリース ノート | Microsoft Docs
description: Azure Stack 統合システム 1907 の更新プログラムについて説明します
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
ms.openlocfilehash: 5ac273ef07747ffa0f5f32e681afd25ecc60bad6
ms.sourcegitcommit: f194f9ca4297864500e62d8658674a0625b29d1d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102187216"
---
# <a name="azure-stack-updates-1907-release-notes"></a>Azure Stack の更新プログラム: 1907 リリース ノート

この記事では、Azure Stack 更新プログラム パッケージの内容について説明します。 この更新プログラムには、このリリースの Azure Stack に対する新機能、機能強化、および修正が含まれています。

別のバージョンのリリース ノートにアクセスするには、左側の目次の上部にあるバージョン セレクターのドロップダウンを使用します。

> [!IMPORTANT]  
> 更新プログラム パッケージは、Azure Stack 統合システム専用です。 Azure Stack Development Kit にこの更新プログラム パッケージは適用しないでください。

> [!IMPORTANT]  
> ご利用の Azure Stack インスタンスが 2 つ前の更新プログラムより遅れている場合、コンプライアンスに対応していないとみなされます。 [サポートを受けるためには、少なくともサポートされる最小バージョンまで更新する](../azure-stack-servicing-policy.md#keep-your-system-under-support)必要があります。

## <a name="update-planning"></a>計画の更新

更新プログラムを適用する前に、必ず次の情報を確認してください。

- [既知の問題](known-issues-1907.md)
- [セキュリティ更新プログラム](../release-notes-security-updates.md)
- [更新プログラム適用前後のアクティビティのチェックリスト](../release-notes-checklist.md)

更新プログラムのトラブルシューティングと更新プロセスの詳細については、[Azure Stack の修正プログラムと更新プログラムに関する問題のトラブルシューティング](../azure-stack-troubleshooting.md)に関するページをご確認ください。

## <a name="1907-build-reference"></a>1907 ビルドのリファレンス

Azure Stack 1907 更新プログラムのビルド番号は **1.1907.0.20** です。

### <a name="update-type"></a>更新の種類

Azure Stack 1907 更新プログラムのビルドの種類は **高速** です。 更新プログラムのビルドの種類については、「[Azure Stack での更新プログラムの管理概要](../azure-stack-updates.md)」を参照してください。 内部テストに基づいて、1907 更新プログラムが完了するまでの予測所要時間は約 13 時間です。

- 更新プログラムの正確な実行時間は一般に、ご使用のシステムでテナント ワークロードによって使用されている容量、システム ネットワーク接続 (インターネットに接続されている場合)、およびシステム ハードウェアの構成に左右されます。
- 実行時間が予測よりも長くなることは一般的ではなく、更新が失敗した場合を除き、Azure Stack オペレーターによるアクションは不要です。
- このおおよその実行時間は、1907 更新プログラムに固有であり、他の Azure Stack 更新プログラムと比較することはできません。

## <a name="whats-in-this-update"></a>この更新プログラムの新機能

<!-- The current theme (if any) of this release. -->

### <a name="whats-new"></a>新機能

<!-- What's new, also net new experiences and features. -->

- 診断ログの収集を容易にし、改善するための Azure Stack 診断ログ収集サービスの一般公開リリース。 Azure Stack 診断ログ収集サービスは、診断ログを収集して、Microsoft カスタマー サポート サービス (CSS) と共有するための簡単な方法を提供します。 この診断ログ収集サービスは、Azure Stack 管理者ポータルでの新しいユーザー エクスペリエンスを提供します。これにより、オペレーターは、特定の重要なアラートが発生したときに、ストレージ BLOB への診断ログの自動アップロードを設定したり、オンデマンドで同じ操作を実行したりすることができます。 詳細については、[診断ログの収集](../diagnostic-log-collection.md)に関する記事を参照してください。

- Azure Stack 検証ツール **Test-AzureStack** の一部としての Azure Stack ネットワーク インフラストラクチャ検証の一般公開リリース。 Azure Stack ネットワーク インフラストラクチャは、**Test-AzureStack** の一部になり、Azure Stack のネットワーク インフラストラクチャで障害が発生したかどうかを特定します。 テストでは、Azure Stack ソフトウェアで定義されたネットワークをバイパスすることによって、ネットワーク インフラストラクチャの接続が確認されます。 パブリック VIP から構成済みの DNS フォワーダー、NTP サーバー、および ID エンドポイントへの接続が示されます。 さらに、ID プロバイダーとして Azure AD を使用する場合に Azure への、または ADFS を使用する場合にフェデレーション サーバーへの接続が確認されます。 詳細については、[Azure Stack 検証ツール](../azure-stack-diagnostic-test.md)に関する記事を参照してください。

- システムの更新中に、必要に応じて、内部の SQL TLS 証明書をローテーションする、内部シークレットのローテーション プロシージャが追加されました。

### <a name="improvements"></a>機能強化

<!-- Changes and product improvements with tangible customer-facing value. -->

- Azure Stack の更新ブレードに、アクティブな更新の **最後のステップが完了した** 時刻が表示されるようになりました。 これは、更新ブレードに移動し、実行中の更新をクリックすると表示されます。 **[Last Step Completed]\(最後のステップの完了\)** は、 **[Update run details]\(更新実行の詳細\)** セクションにあります。

- **Start-AzureStack** と **Stop-AzureStack** オペレーター アクションの改善。 Azure Stack を起動する時間が、平均で 50% 短縮されました。 Azure Stack をシャットダウンする時間が、平均で 30% 短縮されました。 平均の起動とシャットダウンの時間は、スケールユニットのノード数が増加しても変わりません。

- 切断された Marketplace ツールのエラー処理が改善されました。 **Export-AzSOfflineMarketplaceItem** を使用した場合に、ダウンロードが失敗するか、部分的に成功した場合、エラーと軽減手順 (存在する場合) に関する詳細を示す詳細なエラーメッセージが表示されます。

- 大きなページ BLOB/スナップショットからのマネージド ディスク作成のパフォーマンスが向上しました。 以前は、大容量ディスクの作成時にタイムアウトが発生しました。  

<!-- https://icm.ad.msft.net/imp/v3/incidents/details/127669774/home -->
- 予期しない仮想ディスクのデタッチを避けるために、ノードをシャットダウンする前の仮想ディスクの正常性チェックが改善されました。

- 管理者操作のための内部ログのストレージが改善されました。 その結果、内部ログ プロセスのメモリとストレージの消費量を最小限に抑えることで、管理者の操作中にパフォーマンスと信頼性が向上します。 また、管理者ポータルの更新ブレードのページ読み込み時間が短縮される可能性もあります。 この改善の一環として、6 か月を経過した更新ログはシステムで使用できなくなります。 これらの更新のログが必要な場合は、1907 更新を実行する前に、6 か月より前のすべての更新の実行の[概要をダウンロード](../azure-stack-apply-updates.md)してください。

### <a name="changes"></a>[変更点]

- Azure Stack バージョン 1907 には、バージョン 1908 に更新する前に、システムの OEM パッケージをバージョン 2.1 以降に更新するようにオペレーターに指示する警告アラートが含まれています。 Azure Stack の OEM 更新プログラムの適用方法の詳細については、「[Azure Stack に OEM (相手先ブランド供給) 更新プログラムを適用する](../azure-stack-update-oem.md)」を参照してください。

- Azure Stack 診断ログ収集サービスの通信を有効にするための新しいアウトバウンド規則 (HTTPS) が追加されました。 詳細については、[Azure Stack データセンター統合 - エンドポイントの発行](../azure-stack-integrate-endpoints.md#ports-and-urls-outbound)に関するページをご覧ください。

- 外部ストレージの場所の容量が不足している場合、インフラストラクチャ バックアップ サービスによって、部分的にアップロードされたバックアップが削除されるようになりました。

- インフラストラクチャのバックアップに、ドメイン サービス データのバックアップが含まれなくなりました。 これは、Azure Active Directory を ID プロバイダーとして使用するシステムにのみ適用されます。

- **[コンピューティング] > [VM イメージ]** ブレードに取り込まれたイメージが、ページ BLOB の種類であることを検証するようになりました。

### <a name="fixes"></a>修正

<!-- Product fixes that came up from customer deployments worth highlighting, especially if there is an SR/ICM associated to it. -->
- Resource Manager テンプレートで、発行元、オファー、SKU が大文字と小文字を区別して扱われていた問題を修正しました (イメージ パラメーターが発行元、オファー、および SKU と同じ大文字または小文字でない場合、イメージがデプロイ用にフェッチされませんでした)。

<!-- https://icm.ad.msft.net/imp/v3/incidents/details/129536438/home -->
- ストレージ サービス メタデータのバックアップ中のタイムアウトのため、**PartialSucceeded** エラー メッセージでバックアップが失敗する問題を修正しました。  

- ユーザー サブスクリプションを削除すると、リソースが孤立する問題を修正しました。

- オファーの作成時に説明フィールドが保存されなかった問題を修正しました。

- **読み取り専用** のアクセス許可を持つユーザーが、リソースの作成、編集、および削除を行うことができる問題を修正しました。 これで、**共同作成者** のアクセス許可が割り当てられている場合にのみ、ユーザーはリソースを作成できるようになりました。 

<!-- https://icm.ad.msft.net/imp/v3/incidents/details/127772311/home -->
- WMI プロバイダー ホストによって DLL ファイルがロックされるために更新が失敗する問題を修正しました。

- 更新サービスで、更新タイルまたはリソース プロバイダーで、使用可能な更新が表示されない問題を修正しました。 この問題は 1906 で見つかり、修正プログラム [KB4511282](https://support.microsoft.com/help/4511282/) で修正されました。

- 不正な構成によって、管理プレーンが異常になるために、更新が失敗する可能性のある問題を修正しました。 この問題は 1906 で見つかり、修正プログラム [KB4512794](https://support.microsoft.com/help/4512794/) で修正されました。

- ユーザーがマーケットプレースからサードパーティのイメージのデプロイを実行できない問題を修正しました。 この問題は 1906 で見つかり、修正プログラム [KB4511259](https://support.microsoft.com/help/4511259/) で修正されました。

- ユーザー イメージ マネージャー サービスのクラッシュのため、管理対象イメージからの VM の作成が失敗する可能性がある問題を修正しました。 この問題は 1906 で見つかり、修正プログラム [KB4512794](https://support.microsoft.com/help/4512794/) で修正されました

- アプリ ゲートウェイ キャッシュが想定どおりに更新されないために VM CRUD 操作が失敗することがある問題を修正しました。 この問題は 1906 で見つかり、修正プログラム [KB4513119](https://support.microsoft.com/en-us/help/4513119/) で修正されました

- 管理者ポータルでリージョンおよびアラート ブレードの可用性に影響を与えていた正常性リソース プロバイダーの問題を修正しました。 この問題は 1906 で見つかり、修正プログラム [KB4512794](https://support.microsoft.com/help/4512794) で修正されました。

## <a name="security-updates"></a>セキュリティ更新プログラム

Azure Stack のこの更新でのセキュリティ更新プログラムについては、「[Azure Stack security updates](../release-notes-security-updates.md)」 (Azure Stack のセキュリティ更新プログラム) をご覧ください。

## <a name="download-the-update"></a>更新プログラムをダウンロードする

Azure Stack 1907 更新プログラム パッケージは、[Azure Stack ダウンロード ページ](https://aka.ms/azurestackupdatedownload)からダウンロードできます。

## <a name="hotfixes"></a>修正プログラム

Azure Stack では、修正プログラムが定期的にリリースされます。 Azure Stack を 1907 に更新する前に、必ず 1906 用の最新の Azure Stack 修正プログラムをインストールしてください。

Azure Stack 修正プログラムを適用できるのは Azure Stack 統合システムのみです。ASDK には修正プログラムをインストールしないでください。

### <a name="before-applying-the-1907-update"></a>1907 更新プログラムを適用する前

Azure Stack の 1907 リリースは、次の修正プログラムが適用された 1906 リリースに適用する必要があります。

<!-- One of these. Either no updates at all, nothing is required, or the LATEST hotfix that is required-->
- [Azure Stack 修正プログラム 1.1906.15.60](https://support.microsoft.com/help/4524559)

### <a name="after-successfully-applying-the-1907-update"></a>1907 更新プログラムの適用に成功した後

この更新プログラムをインストールした後、適用可能な修正プログラムがあればインストールします。 詳細については、[サービス ポリシー](../azure-stack-servicing-policy.md)に関する記事を参照してください。

<!-- One of these. Either no updates at all, nothing is required, or the LATEST hotfix that is required-->
- [Azure Stack 修正プログラム 1.1907.17.54](https://support.microsoft.com/help/4523826)