---
title: Azure Stack Hub でのログおよび顧客データの処理
description: Azure Stack Hub がどのように顧客データと情報を収集するかについて学習します。
author: BryanLa
ms.topic: article
ms.date: 02/24/2020
ms.author: bryanla
ms.reviewer: chengwei
ms.lastreviewed: 02/24/2020
ms.openlocfilehash: 2b0194ba2955ee0c04c574571cbf16ec27edeef7
ms.sourcegitcommit: ccc4ee05d71496653b6e27de1bb12e4347e20ba4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102231746"
---
# <a name="azure-stack-hub-log-and-customer-data-handling"></a>Azure Stack Hub でのログおよび顧客データの処理 

Microsoft は、Azure Stack Hub に関係する個人データのプロセッサーまたはサブプロセッサーである範囲において、2018 年 5 月 25 日、すべてのお客様に対して次の確約を有効にします。

- [オンライン サービス使用条件](http://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=31)の "データ保護条件" セクションの "個人データの処理: GDPR" 条項。
- [オンライン サービス使用条件](http://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=31)の特則 4 の EU 一般データ保護規則の条件。

Azure Stack Hub がお客様のデータセンター内に存在している間、Microsoft は、[診断](./diagnostic-log-collection.md)、[テレメトリ](azure-stack-telemetry.md)、[課金](azure-stack-usage-reporting.md)を通じて Microsoft と共有されるデータのみを制御します。  

## <a name="data-access-controls"></a>データ アクセスの制御 
特定のサポート ケースを調査するために割り当てられた Microsoft の従業員には、暗号化されたデータを読み取り専用でアクセスする権限が付与されます。 また、Microsoft の従業員は、必要に応じてデータの削除に使用するツールにアクセスできます。 顧客データへのアクセスは、すべて監査および記録されます。  

データ アクセスの制御:
- データは、ケースの終了後、最大でも 90 日間しか保持されません。
- お客様は、その 90 日の期間中、いつでもデータを削除することを選択できます。
- Microsoft の従業員は、サポート問題を解決するために、 ケースバイケースなおかつ必要な場合にのみ、データにアクセスできます。
- Microsoft が顧客データを OEM パートナーと共有する必要がある場合は、必ずお客様の同意を得る必要があります。  

### <a name="what-data-subject-requests-dsr-controls-do-customers-have"></a>お客様が持つデータ主体の要求 (DSR) コントロール
Microsoft は、お客様の要求に応じたオンデマンドでのデータ削除をサポートしています。 お客様は、データが完全に消去される前に、任意の時点で Microsoft サポート エンジニアの 1 人に特定のケースのすべてのログを削除してもらうことを要求できます。  

### <a name="does-microsoft-notify-customers-when-the-data-is-deleted"></a>Microsoft は、データの削除時にお客様に通知するか
(ケースが終了した 90 日後の) 自動データ削除アクションについては、Microsoft は、その削除に関して事前にお客様に連絡をとって通知することはありません。

オンデマンドでのデータ削除アクションを実行する場合、Microsoft サポート エンジニアはオンデマンドでデータを削除できるようにするツールにアクセスできます。 完了したら、その旨を電話でお客様に確認します。

## <a name="diagnostic-data"></a>診断データ
サポート プロセスの一環として、Azure Stack Hub オペレーターは、トラブルシューティングに役立てるため、Azure Stack Hub のサポート チームおよびエンジニアリング チームと[診断ログを共有](./diagnostic-log-collection.md)できます。

Microsoft は、お客様が、要求された診断ログ ファイルを収集してアップロードするためのツールおよびスクリプトを提供します。 収集されたログ ファイルは、HTTPS で保護された暗号化接続を通じて Microsoft に転送されます。 HTTPS では、ネットワーク データが暗号化されるため、転送時に暗号化のためのパスワードは不要です。 受信されたログは、サポート ケースが終了して 90 日後に自動削除されるまで、暗号化された状態で保管されます。

## <a name="telemetry-data"></a>テレメトリ データ
[Azure Stack Hub のテレメトリ](azure-stack-telemetry.md)では、接続ユーザー エクスペリエンスを介してシステム データが自動的に Microsoft にアップロードされます。 テレメトリの機能やプライバシー設定は、Azure Stack Hub オペレーターの権限でいつでもカスタマイズできます。

Microsoft が意図して機密データ (クレジット カード番号、ユーザー名とパスワード、メール アドレスなど) を収集することはありません。 意図せず機密情報を受け取ったと Microsoft が判断した場合には、削除します。

## <a name="billing-data"></a>課金データ
[Azure Stack Hub の課金](azure-stack-usage-reporting.md)では、グローバルな Azure の課金および使用パイプラインが活用されます。このため、Microsoft のコンプライアンス ガイドラインに合致しています。

Azure Stack Hub オペレーターは、課金のための使用状況情報を Azure に転送するように Azure Stack Hub を構成できます。 従量制課金モデルを選択した Azure Stack Hub 統合システムのお客様には、この構成が必須となります。 使用状況のレポートは、テレメトリとは別に管理されます。容量モデルを選択した統合システムのお客様や、Azure Stack Development Kit ユーザーの場合、使用状況のレポートは不要です。 そのような場合は、[登録スクリプト](azure-stack-usage-reporting.md)を使って、使用状況のレポートを無効にすることができます。


## <a name="next-steps"></a>次のステップ 
[Azure Stack Hub のセキュリティについて詳しく学習する](azure-stack-security-foundations.md)