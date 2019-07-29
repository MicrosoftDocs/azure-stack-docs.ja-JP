---
title: Azure Stack ポータルの使用 | Microsoft Docs
description: Azure Stack でユーザー ポータルにアクセスして使用する方法を説明します。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/13/2019
ms.author: mabrigg
ms.reviewer: efemmano
ms.lastreviewed: 01/25/2019
ms.openlocfilehash: 629056556b04a7b5d19c2463b619f3da3a70f7e0
ms.sourcegitcommit: 72d45bb935db0db172d4d7c37d8e48e79e25af64
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/22/2019
ms.locfileid: "68376900"
---
# <a name="use-the-azure-stack-portal"></a>Azure Stack ポータルの使用

*適用対象:Azure Stack 統合システムと Azure Stack Development Kit*

Azure Stack ポータルを使用して、パブリック オファーをサブスクライブしたり、これらのオファーで提供されるサービスを使用したりします。 グローバル Azure portal を使用したことがあれば、サイトのしくみには既になじみがあります。

## <a name="access-the-portal"></a>ポータルへのアクセス

Azure Stack オペレーター (サービス プロバイダーまたは組織内の管理者のどちらか) から、ポータルにアクセスするための正しい URL が通知されます。

- 統合システムの場合、URL はオペレーターのリージョンと外部ドメイン名によって異なり、 https://portal.&lt;*リージョン*&gt;.&lt;*FQDN*&gt; の形式になります。
- Azure Stack Development Kit (ASDK) を使用している場合、ポータル アドレスは https://portal.local.azurestack.external です。
- すべての Azure Stack デプロイの既定のタイム ゾーンは、協定世界時 (UTC) に設定されます。 Azure Stack のインストール時にタイム ゾーンを選択できますが、インストール中に自動的に既定として UTC に戻されます。

## <a name="customize-the-dashboard"></a>ダッシュボードのカスタマイズ

ダッシュボードには、タイルの既定のセットが含まれています。 **[ダッシュボードの編集]** を選択して既定のダッシュボードを変更するか、 **[新しいダッシュボード]** を選択してカスタムのダッシュボードを作成します。 タイルを追加または削除して、ダッシュボードを簡単にカスタマイズすることができます。 たとえば、[コンピューティング] タイルを追加するには、 **[+ リソースの作成]** を選択します。 **[コンピューティング]** を右クリックし、 **[ダッシュボードにピン留め]** を選択します。

![Azure Stack ユーザー ポータルの画面キャプチャ](media/azure-stack-use-portal/userportal.png)

ダッシュボードを元の設定に復元するには:
1.  **[ダッシュボードの編集]** を選択します。 
2.  右クリックし、 **[既定の状態にリセット]** を選択します。

## <a name="create-subscription-and-browse-available-resources"></a>サブスクリプションの作成と使用可能リソースの参照

まだサブスクリプションがない場合、オファーをサブスクライブする必要があります。 その後、利用可能なリソースを参照することができます。 リソースを参照したり作成したりするには、次のいずれかの方法を使用します。

- ダッシュボードで **[Marketplace]** タイルを選択します。
- **[すべてのリソース]** タイルで、 **[リソースの作成]** を選択します。
- 左側のナビゲーション ウィンドウで、 **[+ リソースの作成]** を選択します。

## <a name="learn-how-to-use-available-services"></a>使用可能なサービスの使用方法を学ぶ

使用可能なサービスの使用方法についてガイダンスが必要な場合に、利用できるオプションが異なる可能性があります。

- 組織またはサービス プロバイダーによって独自のドキュメントが提供されることがあります。これは通常、カスタマイズされたサービスまたはアプリが提供される場合です。
- サード パーティのアプリには、独自のドキュメントがあります。
- Azure 互換サービスについては、まず Azure Stack のドキュメントを確認することを強くお勧めします。 Azure Stack ユーザー ドキュメントにアクセスするには、ヘルプ アイコン ( **?** )、 **[ヘルプとサポート]** の順に選択します。

    ![UI の [ヘルプとサポート] オプション](media/azure-stack-use-portal/HelpAndSupport.png)

    特に、作業を開始するにあたって次の記事を確認することをお勧めします。

    - [重要な考慮事項: Azure Stack でのサービスの使用またはアプリの作成](azure-stack-considerations.md)。
    - ドキュメントの**サービスの使用**に関するセクションには、サービスごとの考慮事項の記事があります。 考慮事項に関するページでは、Azure で提供されるサービスと Azure Stack で提供される同じサービス間の相違点を説明しています。 例については、「[VM に関する考慮事項](azure-stack-vm-considerations.md)」を参照してください。 **サービスの使用**に関するセクションには、Azure Stack に固有であるその他の情報が含まれている場合があります。

      サービスの一般的なリファレンスとして Azure のドキュメントを使用できますが、これらの相違点を把握しておく必要があります。 **[Quickstart tutorials] (クイックスタート チュートリアル)** タイル上のドキュメント リンクが Azure のドキュメントを指していることを理解します。

## <a name="get-support"></a>サポートを受ける

サポートが必要な場合は、所属組織またはサービス プロバイダーにお問い合わせください。

Azure Stack Development Kit (ASDK) を使用している場合、[Azure Stack フォーラム](https://social.msdn.microsoft.com/Forums/azure/home?forum=azurestack)が唯一のサポート情報源です。

## <a name="next-steps"></a>次の手順

[重要な考慮事項: Azure Stack でのサービスの使用またはアプリの作成](azure-stack-considerations.md)
