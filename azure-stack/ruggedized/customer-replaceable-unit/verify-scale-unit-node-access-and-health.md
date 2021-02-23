---
title: スケール ユニット ノードのアクセスと正常性の確認
description: スケール ユニット ノードのアクセスと正常性の確認方法について説明します
author: PatAltimore
ms.topic: how-to
ms.date: 11/13/2020
ms.author: patricka
ms.reviewer: ''
ms.lastreviewed: ''
ms.openlocfilehash: d636b5ebb0d07c2711a17b2b2f10651480c38068
ms.sourcegitcommit: 5ea0e915f24c8bcddbcaf8268e3c963aa8877c9d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2021
ms.locfileid: "100488022"
---
# <a name="verifying-scale-unit-node-access-and-health"></a>スケール ユニット ノードのアクセスと正常性の確認

特権アクセス ワークステーションにログインし、管理者ポータルを起動して、システム正常性を確認し、特権エンドポイントの IP アドレスを取得して、ドレインまたは再開する必要があるノードを特定します。

1.  リモート デスクトップを使用して、特権アクセス ワークステーションに接続します。

2.  Azure Stack Hub 管理者ポータルにアクセスします。

    顧客から取得した資格情報を使用して、Azure Stack Hub 管理者ポータルにログインします。
        
3.  特権エンドポイントの IP アドレスを取得します。


    **[Region Management]\(リージョン管理\)** タイルで、 **[プロパティ]** を選択します。 ペインの一番下までスクロールし、 **[Privileged endpoint IP addresses]\(特権エンドポイントの IP アドレス\)** フィールドで IP アドレスを見つけます。 これらは、この手順で後ほど必要になる場合や、問題が発生した場合にサポートで必要になることもあるため、メモしておきます。

    [![[Privileged endpoint IP addresses]\(特権エンドポイントの IP アドレス\) セクションが強調表示されている [管理] ページを示すスクリーンショット。](media/image-18-inline.png)](media/image-18-expanded.png#lightbox)
    
4.  現在のアラートを確認します。

    **[Region Management]\(リージョン管理\)** で、 **[アラート]** を選択して、現在のアラートを確認します。 予期しないアラートが存在する場合は、それらをクリアできるかまたは無視しても問題ないのかを Microsoft サポートに確認してください。
    
    [![[名前] セクションが強調表示されている [プロパティ] ページを示すスクリーンショット。](media/image-19-inline.png)](media/image-19-expanded.png#lightbox)
    
5.  スケール ユニット ノードを特定します。

    サービス タグのみが提供され、Azure Stack Hub 管理ポータルから問題が発生しているノードを特定できない場合 (つまり、ノードの電源状態が既に停止している場合) は、次の手順を使用して、スケール ユニット ノードをサービス タグに関連付けます。
    
    1.  **[Region Management]\(リージョン管理\)** で、 **[スケール ユニット]** を選択し、クラスター **s-cluster** を選択します。 **[ノード]** を選択します。
    
    1.  ノード サービス タグを取得するには、 **[BMC]** IP アドレス リンクを選択します。これにより、サーバーの iDRAC Web インターフェイスが新しいタブまたはウィンドウに表示されます。

        [![[BMC] 列が強調表示されている [ノード] ページを示すスクリーンショット。](media/image-20-inline.png)](media/image-20-expanded.png#lightbox) 
    
    1.  iDRAC インターフェイスにログインし、 **[システム情報]** ペインでノードのサービス タグを確認します。
    
    1.  すべてのノードに対してこの手順を繰り返し、これらのサービス タグと予定されているハードウェアの交換を関連付けて、処理する必要があるノードを特定します。

        [![[サービス タグ] が強調表示されている [ダッシュボード] を示すスクリーンショット。](media/image-21-inline.png)](media/image-21-expanded.png#lightbox)
    
