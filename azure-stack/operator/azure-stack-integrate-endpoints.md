---
title: Azure Stack とデータセンターの統合 - エンドポイントの公開 | Microsoft Docs
description: データセンターで Azure Stack のエンドポイントを公開する方法を学習します
services: azure-stack
author: mattbriggs
manager: femila
ms.service: azure-stack
ms.topic: article
ms.date: 05/02/2019
ms.author: mabrigg
ms.reviewer: wamota
ms.lastreviewed: 02/06/2019
ms.openlocfilehash: 7ee47a5dc7344628561521f067a8310a0c8d3347
ms.sourcegitcommit: 23816ec68f67f3ac51f78de925b7631590743a29
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/11/2019
ms.locfileid: "66835084"
---
# <a name="azure-stack-datacenter-integration---publish-endpoints"></a>Azure Stack とデータセンターの統合 - エンドポイントの公開

Azure Stack は、そのインフラストラクチャ ロールのために仮想 IP アドレス (VIP) を設定します。 この VIP はパブリック IP アドレス プールから割り当てられます。 各 VIP は、ソフトウェア定義のネットワーク レイヤーで、アクセス制御リスト (ACL) で保護されます。 ACL は、ソリューションをさらに強化するために、さまざまな物理スイッチ (TOR や BMC) でも使われます。 デプロイ時に指定された外部 DNS ゾーン内のエンドポイントごとに DNS エントリが作成されます。


次のアーキテクチャ図は、さまざまなネットワーク レイヤーと ACL を示しています。

![構造の画像](media/azure-stack-integrate-endpoints/Integrate-Endpoints-01.png)

## <a name="ports-and-protocols-inbound"></a>ポートとプロトコル (受信)

Azure Stack エンドポイントを外部ネットワークに公開するには、一連のインフラストラクチャ VIP が必要です。 "*エンドポイント (VIP)* " の表は、各エンドポイント、必要なポート、およびプロトコルを示しています。 SQL のリソース プロバイダーなど、追加のリソース プロバイダーを必要とするエンドポイントについては、特定のリソース プロバイダーの展開に関するドキュメントを参照してください。

社内インフラストラクチャの VIP は Azure Stack の発行には不要なため、記載されていません。

> [!Note]  
> ユーザーの VIP は動的で、Azure Stack オペレーターによって管理されず、ユーザー自身が定義します。

> [!Note]  
> IKEv2 VPN。 IKEv2 VPN は、標準ベースの IPsec VPN ソリューションであり、UDP ポート 500 と 4500 のほか、IP プロトコル番号 50 を使用しています。 ファイアウォールでここに挙げたポートが必ずしも開いているとは限りません。このため、IKEv2 VPN ではプロキシとファイアウォールを通過できないことがあります。

> [!Note]  
> [拡張機能ホスト](azure-stack-extension-host-prepare.md)の追加により、1811 更新プログラム以降、12495 から 30015 の範囲のポートを開く必要はなくなりました。

|エンドポイント (VIP)|DNS ホスト A レコード|Protocol|Port|
|---------|---------|---------|---------|
|AD FS|Adfs. *&lt;region>.&lt;fqdn>*|HTTPS|443|
|ポータル (管理者)|Adminportal. *&lt;region>.&lt;fqdn>*|HTTPS|443|
|AdminHosting | *.adminhosting.\<region>.\<fqdn> | HTTPS | 443 |
|Azure Resource Manager (管理者)|Adminmanagement. *&lt;region>.&lt;fqdn>*|HTTPS|443|
|ポータル (ユーザー)|Portal. *&lt;region>.&lt;fqdn>*|HTTPS|443|
|Azure Resource Manager (ユーザー)|Management. *&lt;region>.&lt;fqdn>*|HTTPS|443|
|Graph|Graph. *&lt;region>.&lt;fqdn>*|HTTPS|443|
|証明書の失効リスト|Crl. *&lt;region>.&lt;fqdn>*|HTTP|80|
|DNS|&#42;. *&lt;region>.&lt;fqdn>*|TCP と UDP|53|
|Hosting | *.hosting.\<region>.\<fqdn> | HTTPS | 443 |
|Key Vault (ユーザー)|&#42;.vault. *&lt;region>.&lt;fqdn>*|HTTPS|443|
|Key Vault (管理者)|&#42;.adminvault. *&lt;region>.&lt;fqdn>*|HTTPS|443|
|ストレージ キュー|&#42;.queue. *&lt;region>.&lt;fqdn>*|HTTP<br>HTTPS|80<br>443|
|ストレージ テーブル|&#42;.table. *&lt;region>.&lt;fqdn>*|HTTP<br>HTTPS|80<br>443|
|ストレージ BLOB|&#42;.blob. *&lt;region>.&lt;fqdn>*|HTTP<br>HTTPS|80<br>443|
|SQL リソース プロバイダー|sqladapter.dbadapter. *&lt;region>.&lt;fqdn>*|HTTPS|44300-44304|
|MySQL リソース プロバイダー|mysqladapter.dbadapter. *&lt;region>.&lt;fqdn>*|HTTPS|44300-44304|
|App Service|&#42;.appservice. *&lt;region>.&lt;fqdn>*|TCP|80 (HTTP)<br>443 (HTTPS)<br>8172 (MSDeploy)|
|  |&#42;.scm.appservice. *&lt;region>.&lt;fqdn>*|TCP|443 (HTTPS)|
|  |api.appservice. *&lt;region>.&lt;fqdn>*|TCP|443 (HTTPS)<br>44300 (Azure Resource Manager)|
|  |ftp.appservice. *&lt;region>.&lt;fqdn>*|TCP、UDP|21、1021、10001-10100 (FTP)<br>990 (FTPS)|
|VPN ゲートウェイ|     |     |[VPN Gateway に関する FAQ を参照してください](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-vpn-faq#can-i-traverse-proxies-and-firewalls-using-point-to-site-capability)。|
|     |     |     |     |

## <a name="ports-and-urls-outbound"></a>ポートと URL (送信)

Azure Stack は、透過的なプロキシ サーバーのみをサポートします。 透過プロキシから従来のプロキシ サーバーへのアップリンクが存在するデプロイ環境では、次のポートと URL に外部への通信を許可する必要があります。

> [!Note]  
> Azure Stack では、ExpressRoute を利用して､次の表に示す Azure サービスに到達することはできません。

|目的|接続先 URL|Protocol|Port|ソース ネットワーク|
|---------|---------|---------|---------|---------|
|ID|login.windows.net<br>login.microsoftonline.com<br>graph.windows.net<br>https:\//secure.aadcdn.microsoftonline-p.com<br>office.com|HTTP<br>HTTPS|80<br>443|パブリック VIP - /27<br>パブリック インフラストラクチャ ネットワーク|
|Marketplace シンジケーション|https:\//management.azure.com<br>https://&#42;.blob.core.windows.net<br>https://*.azureedge.net<br>https://&#42;.microsoftazurestack.com|HTTPS|443|パブリック VIP - /27|
|パッチと更新プログラム|https://&#42;.azureedge.net<br>https:\//aka.ms/azurestackautomaticupdate|HTTPS|443|パブリック VIP - /27|
|登録|https:\//management.azure.com|HTTPS|443|パブリック VIP - /27|
|使用法|https://&#42;.microsoftazurestack.com<br>https://*.trafficmanager.net |HTTPS|443|パブリック VIP - /27|
|Windows Defender|\*.wdcp.microsoft.com<br>\*.wdcpalt.microsoft.com<br>\*.wd.microsoft.com<br>\*.update.microsoft.com<br>\*.download.microsoft.com<br>https:\//www.microsoft.com/pkiops/crl<br>https:\//www.microsoft.com/pkiops/certs<br>https:\//crl.microsoft.com/pki/crl/products<br>https:\//www.microsoft.com/pki/certs<br>https:\//secure.aadcdn.microsoftonline-p.com<br>|HTTPS|80<br>443|パブリック VIP - /27<br>パブリック インフラストラクチャ ネットワーク|
|NTP|(デプロイに提供される NTP サーバーの IP)|UDP|123|パブリック VIP - /27|
|DNS|(デプロイに提供される DNS サーバーの IP)|TCP<br>UDP|53|パブリック VIP - /27|
|CRL|(証明書上で CRL 配布ポイントの下にある URL)|HTTP|80|パブリック VIP - /27|
|LDAP|Graph 統合のために用意されている Active Directory フォレスト|TCP<br>UDP|389|パブリック VIP - /27|
|LDAP SSL|Graph 統合のために用意されている Active Directory フォレスト|TCP|636|パブリック VIP - /27|
|LDAP GC|Graph 統合のために用意されている Active Directory フォレスト|TCP|3268|パブリック VIP - /27|
|LDAP GC SSL|Graph 統合のために用意されている Active Directory フォレスト|TCP|3269|パブリック VIP - /27|
|AD FS|AD FS 統合のために用意されている AD FS メタデータ エンドポイント|TCP|443|パブリック VIP - /27|
|     |     |     |     |     |

送信 URL は Azure Traffic Manager を使用して負荷分散され、地理的な場所に基づいて可能な限り最適な接続が提供されます。 URL を負荷分散することで、Microsoft は、顧客に影響を与えることなくバックエンド エンドポイントを更新および変更することができます。 Microsoft では、負荷分散される URL の IP アドレスのリストを共有していません。 IP ではなく URL に基づくフィルターをサポートするデバイスを使用する必要があります。

送信 DNS は常に必要です。違っているのは、外部 DNS のクエリを実行しているソースと、選択された ID 統合の種類です。 これが接続されたシナリオの場合は、デプロイ時には BMC ネットワーク上に配置された DVM でその送信アクセスが必要ですが、デプロイ後は、DNS サービスはパブリック VIP を使用してクエリを送信する内部コンポーネントに移動します。 その時点で、BMC ネットワーク経由での送信 DNS アクセスは削除できますが、その DNS サーバーへのパブリック VIP アクセスは残しておく必要があり、そうしないと認証は失敗します。

## <a name="next-steps"></a>次の手順

[Azure Stack PKI の要件](azure-stack-pki-certs.md)
