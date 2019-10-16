---
title: Azure Stack の DNS | Microsoft Docs
description: Azure Stack の DNS についてと DNS ゾーンを作成および管理する方法を説明します。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/03/2019
ms.author: sethm
ms.lastreviewed: 01/05/2019
ms.openlocfilehash: 5600dd6537df35e703e0ac7a08ad4a61f976e489
ms.sourcegitcommit: b2d19e12a50195bb8925879ee75c186c9604f313
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/04/2019
ms.locfileid: "71961478"
---
# <a name="use-dns-in-azure-stack"></a>Azure Stack での DNS の使用

*適用対象:Azure Stack 統合システムと Azure Stack Development Kit*

Azure Stack では、次の Azure DNS 機能がサポートされます。

* DNS ホスト名の解決。
* API を使用して DNS のゾーンとレコードを作成し、管理します。

## <a name="support-for-dns-hostname-resolution"></a>DNS ホスト名解決のサポート

パブリック IP リソースの DNS ドメイン名ラベルを指定できます。 Azure Stack では、ラベル名に **domainnamelabel.location**.cloudapp.azurestack.external を使用し、Azure Stack によって管理される DNS サーバーでそれをパブリック IP アドレスにマップします。

たとえば、ドメイン名ラベルとして **contoso** を使用したパブリック IP リソースをローカルの Azure Stack の場所に作成した場合、[完全修飾ドメイン名 (FQDN)](https://en.wikipedia.org/wiki/Fully_qualified_domain_name) である **contoso.local.cloudapp.azurestack.external** は、このリソースのパブリック IP アドレスに解決されます。 この FQDN を使用して、Azure Stack 内のパブリック IP アドレスをポイントするカスタム ドメイン CNAME レコードを作成できます。

名前解決の詳細については、[DNS 解決](/azure/dns/dns-for-azure-services?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)に関する記事を参照してください。

> [!IMPORTANT]
> 作成した各ドメイン名ラベルは、Azure Stack の場所内で一意である必要があります。

次のスクリーンショットは、ポータルを利用してパブリック IP アドレスを作成するための **[パブリック IP アドレスの作成]** ダイアログのものです。

![パブリック IP アドレスの作成](media/azure-stack-dns/image01.png)

### <a name="example-scenario"></a>サンプル シナリオ

Web アプリからの要求を処理するロード バランサーがあります。 その背後では、ある Web サイトが 1 つ以上の仮想マシン上で実行されているとします。 この負荷分散された Web サイトには、IP アドレスではなく、DNS 名でアクセスできます。

## <a name="create-and-manage-dns-zones-and-records-using-the-apis"></a>API を使用して DNS のゾーンとレコードを作成し、管理する

Azure Stack 内に DNS ゾーンとレコードを作成して管理することができます。

Azure Stack は、Azure の DNS API と同じ API を使用して、Azure と同じような DNS サービスを提供します。  Azure Stack DNS でドメインをホストすることで、同じ資格情報、API、ツールを使用して DNS レコードを管理できます。 他の Azure サービスと同じ請求とサポートも使用できます。

Azure Stack DNS のインフラストラクチャは Azure のインフラストラクチャよりもコンパクトです。 Azure Stack デプロイのサイズと場所は、DNS のスコープ、規模、パフォーマンスに影響を与えます。 また、パフォーマンス、可用性、グローバル配布、高可用性などは、デプロイに応じて変わってきます。

## <a name="comparison-with-azure-dns"></a>Azure DNS との比較

Azure Stack の DNS は Azure の DNS に似ていますが、重要な例外がいくつかあります。

* **AAAA レコードのサポートがない**:Azure Stack では IPv6 アドレスがサポートされないので、AAAA レコードもサポートされません。 これは、Azure と Azure Stack の DNS の大きな違いのひとつです。

* **マルチテナントではない**:Azure Stack の DNS サービスはマルチテナントではありません。 テナントで同じ DNS ゾーンを作成することはできません。 ゾーンの作成を試みた最初のサブスクリプションだけが成功し、後の要求は失敗します。 これは、Azure と Azure Stack DNS のもう 1 つの大きな違いです。

* **タグ、メタデータ、Etag**:Azure Stack でのタグ、メタデータ、Etag、制限の処理方法にも多少の違いがあります。

Azure DNS の詳細については、[DNS のゾーンとレコード](/azure/dns/dns-zones-records)に関するページを参照してください。

### <a name="tags"></a>Tags

Azure Stack DNS では、DNS ゾーン リソースに対して Azure Resource Manager のタグを使用できます。 DNS レコード セットへのタグ設定はサポートされていません。 その代わりとして、DNS レコード セットでは、次のセクションで説明するように**メタデータ**がサポートされます。

### <a name="metadata"></a>Metadata

レコード セットのタグの代わりに、Azure Stack DNS では*メタデータ*を使用してレコード セットに注釈を付けることができます。 メタデータを使用すると、タグと同じように、各レコード セットに名前と値のペアを関連付けることができます。 たとえば、メタデータは、各レコード セットの用途を記録する場合に便利です。 タグと異なる点として、メタデータは、Azure の課金内容に関するフィルター ビューを提供するためには使用できず、Azure Resource Manager のポリシーで指定することもできません。

### <a name="etags"></a>Etag

たとえば、2 人のユーザーまたは 2 つのプロセスが同時に DNS レコードを変更しようとするとします。 どちらの変更が優先されるでしょうか。 また、優先された側は、他のユーザーまたはプロセスによって行われた変更を上書きしたことに気付くのでしょうか。

Azure Stack DNS は、*Etag* を使用して同じリソースへの同時変更を安全に処理します。 Etag と Azure Resource Manager の*タグ*は異なります。 各 DNS リソース (ゾーンまたはレコード セット) には、関連付けられている Etag があります。 リソースが取得されるときは、常に Etag も取得されます。 リソースを更新する場合は、Azure Stack DNS がサーバー上の Etag の一致を確認できるように Etag を返すこともできます。 リソースを更新するたびに Etag が再生成されるため、Etag の不一致は同時変更が発生していることを示します。 Etag は、既存のリソースがないことを確認するために、新しいリソースの作成時にも使用できます。

Azure Stack DNS PowerShell コマンドレットは、既定で、ゾーンとレコード セットへの同時変更のブロックに Etag を使用します。 オプションの `-Overwrite` スイッチを使用すると、Etag チェックを抑制できます。 Etag チェックを行わないと、発生した同時変更がすべて上書きされます。

Azure Stack DNS REST API のレベルでは、HTTP ヘッダーを使用して Etag を指定します。 次の表に各ヘッダーの動作を示します。

| ヘッダー | 動作|
|--------|---------|
| なし   | PUT は常に成功します (Etag チェックなし)。|
| If-match| PUT は、リソースが存在し、Etag が一致する場合にのみ成功します。|
| If-match *| PUT はリソースが存在する場合にのみ成功します。|
| If-none-match *| PUT はリソースが存在しない場合にのみ成功します。|

### <a name="limits"></a>制限

Azure Stack DNS を使用する際は、次の制限が既定で適用されます。

| リソース| 既定の制限|
|---------|--------------|
| サブスクリプションあたりのゾーン数| 100|
| ゾーンあたりのレコード セット数| 5000|
| レコード セットあたりのレコード数| 20|

## <a name="next-steps"></a>次の手順

* [Azure Stack の iDNS の概要](azure-stack-understanding-dns.md)
