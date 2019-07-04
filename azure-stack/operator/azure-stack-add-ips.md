---
title: Azure Stack で Public IP Addresses を追加する | Microsoft Docs
description: Azure Stack に Public IP Addresses をさらに追加する方法について説明します。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/13/2019
ms.author: mabrigg
ms.reviewer: scottnap
ms.lastreviewed: 02/28/2019
ms.openlocfilehash: 54caca10d729968f4c7f05dea456ee13056e4e7e
ms.sourcegitcommit: b79a6ec12641d258b9f199da0a35365898ae55ff
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/13/2019
ms.locfileid: "67131046"
---
# <a name="add-public-ip-addresses"></a>Public IP Addresses を追加する
*適用対象:Azure Stack 統合システムと Azure Stack Development Kit*  

Azure Stack に Public IP Addresses をさらに追加する方法について説明します。  この記事では、外部アドレスを Public IP Addresses と呼びますが、Azure Stack では、これはご利用の外部ネットワークに IP アドレス ブロックを追加することを指します。  この記事では、外部ネットワークがルーティング可能なパブリック インターネットであるか、イントラネット上にあるか、またプライベート アドレス空間が使用されるかはあまり重要ではありません。  手順は同じです。 

## <a name="add-a-public-ip-address-pool"></a>パブリック IP アドレス プールを追加する
Azure Stack システムの初期デプロイ後にいつでも Azure Stack システムに Public IP Addresses を追加することができます。 [Public IP Addresses の使用量を表示する](azure-stack-viewing-public-ip-address-consumption.md)方法を参照し、ご利用の Azure Stack での現在の使用量と Public IP Addresses の可用性を確認してください。

Azure Stack に新しい Public IP Addresses ブロックを追加するプロセスの概要を以下に示します。

 ![IP の追加フロー](media/azure-stack-add-ips/flow.PNG)

## <a name="obtain-the-address-block-from-your-provider"></a>プロバイダーからアドレス ブロックを取得する
まず、Azure Stack に追加するアドレス ブロックを取得する必要があります。  どこからアドレス ブロックを取得するかに応じて、リード タイムを考慮し、Azure Stack でのパブリック IP アドレスの使用率と照らし合わせて管理する必要があります。  

> [!IMPORTANT]
> Azure Stack では指定されるすべてのアドレス ブロックが受け入れられます。ただし、それはそのアドレス ブロックが有効であり、Azure Stack の既存のアドレス範囲と重複していない場合に限られます。  Azure Stack が接続されている外部ネットワークと重複しておらず、ルーティングできる有効なアドレス ブロックを取得するようにしてください。  Azure Stack に範囲を追加した後で、それを削除することはできません。

## <a name="add-the-ip-address-range-to-azure-stack"></a>Azure Stack に IP アドレス範囲を追加する

1. インターネット ブラウザーで、管理ポータル ダッシュボードに移動します。  この例では、 https://adminportal.local.azurestack.external を使用します。  
2.  Azure Stack 管理ポータルにクラウド オペレーターとしてサインインします。
3.  既定のダッシュボードで、[リージョンの管理] リストを見つけて、管理するリージョン (この例では、[local]) を選択します。
4.  [リソース プロバイダー] タイルを見つけて、[ネットワーク リソース プロバイダー] をクリックします。
5.  [パブリック IP プールの使用量] タイルをクリックします。
6.  [Add IP pool]\(IP プールの追加\) ボタンをクリックします。
7.  IP プールの名前を指定します。  選択する名前は、単に IP プールを簡単に識別し、自由に呼び出せるようにするためのものです。  必須ではありませんが、アドレス範囲と同じ名前にすることをお勧めします。
8.   CIDR 表記で追加するアドレス ブロックを入力します。  例: 192.168.203.0/24
9.  [アドレス範囲 (CIDR ブロック)] フィールドに有効な CIDR 範囲を指定すると、[開始 IP アドレス]、[終了 IP アドレス] および [利用可能な IP アドレス] の各フィールドが自動的に設定されます。  これらは読み取り専用で、自動的に生成されるため、[アドレス範囲] フィールドで値を変更せずにこれらを変更することはできません。
10. ブレード上の情報を確認し、すべて正しいことがわかったら、[OK] をクリックして変更をコミットし、アドレス範囲を Azure Stack に追加します。


## <a name="next-steps"></a>次の手順 
[スケール ユニットのノード操作を確認する](azure-stack-node-actions.md) 
