---
title: データセンターに Azure Stack Hub サービスを発行する
description: データセンターに Azure Stack Hub サービスを発行する方法を学習します。
author: myoung
ms.topic: article
ms.date: 09/24/2020
ms.author: patricka
ms.reviewer: wamota
ms.lastreviewed: 09/24/2020
ms.openlocfilehash: 23e97f1e91b9ee9a6a76ee6037514bb9c17636b4
ms.sourcegitcommit: 283b1308142e668749345bf24b63d40172559509
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/05/2021
ms.locfileid: "99570618"
---
# <a name="publish-azure-stack-hub-services-in-your-datacenter"></a>データセンターに Azure Stack Hub サービスを発行する

Azure Stack Hub では、そのインフラストラクチャ ロールのために仮想 IP アドレス (VIP) を設定します。 この VIP はパブリック IP アドレス プールから割り当てられます。 各 VIP は、ソフトウェア定義のネットワーク レイヤーで、アクセス制御リスト (ACL) で保護されます。 ACL は、ソリューションをさらに強化するために、さまざまな物理スイッチ (TOR や BMC) でも使われます。 デプロイ時に指定された外部 DNS ゾーン内のエンドポイントごとに、DNS エントリが作成されます。 たとえば、ユーザー ポータルに portal. *&lt;region>.&lt;fqdn>* の DNS ホスト エントリが割り当てられます。

次のアーキテクチャ図は、さまざまなネットワーク レイヤーと ACL を示しています。

![さまざまなネットワーク レイヤーと ACL を示した図](media/azure-stack-integrate-endpoints/integrate-endpoints-01.svg)

## <a name="ports-and-urls"></a>ポートと URL

Azure Stack Hub サービス (ポータル、Azure Resource Manager、DNS など) を外部ネットワークで使用できるようにするには、特定の URL、ポート、プロトコルに対して、これらのエンドポイントへの受信トラフィックを許可する必要があります。

透過プロキシから従来のプロキシ サーバーへのアップリンクが存在するか、ファイアウォールでソリューションを保護しているデプロイでは、特定のポートと URL に[受信](azure-stack-integrate-endpoints.md#ports-and-protocols-inbound)および[送信](azure-stack-integrate-endpoints.md#ports-and-urls-outbound)の両方の通信を許可する必要があります。 これには、ID、マーケットプレース、パッチと更新プログラム、登録、使用状況データに使用するポートと URL が該当します。

SSL トラフィックのインターセプトは[サポートされておらず](azure-stack-firewall.md#ssl-interception)、エンドポイントへのアクセスでサービス エラーが発生する可能性があります。 

## <a name="ports-and-protocols-inbound"></a>ポートとプロトコル (受信)

Azure Stack Hub エンドポイントを外部ネットワークに発行するには、一連のインフラストラクチャ VIP が必要です。 "*エンドポイント (VIP)* " の表は、各エンドポイント、必要なポート、およびプロトコルを示しています。 SQL リソース プロバイダーなど、追加のリソース プロバイダーを必要とするエンドポイントについては、特定のリソース プロバイダーのデプロイに関するドキュメントを参照してください。

内部インフラストラクチャの VIP は Azure Stack Hub の発行には必要ないため、リストされていません。 ユーザーの VIP は動的であり、ユーザー自身によって定義され、Azure Stack Hub オペレーターによって制御されません。

> [!Note]  
> IKEv2 VPN は、標準ベースの IPsec VPN ソリューションであり、UDP ポート 500 と 4500、および TCP ポート 50 を使用します。 ファイアウォールでは、これらのポートが必ずしも開いているとは限りません。そのため、IKEv2 VPN ではプロキシとファイアウォールを通過できないことがあります。

[拡張機能ホスト](azure-stack-extension-host-prepare.md)の追加により、12495 から 30015 の範囲のポートは必要ありません。

|エンドポイント (VIP)|DNS ホスト A レコード|Protocol|Port|
|---------|---------|---------|---------|
|AD FS|Adfs. *&lt;region>.&lt;fqdn>*|HTTPS|443|
|ポータル (管理者)|Adminportal. *&lt;region>.&lt;fqdn>*|HTTPS|443|
|AdminHosting | *.adminhosting.\<region>.\<fqdn> | HTTPS | 443 |
|Azure Resource Manager (管理者)|Adminmanagement. *&lt;region>.&lt;fqdn>*|HTTPS|443|
|ポータル (ユーザー)|Portal. *&lt;region>.&lt;fqdn>*|HTTPS|443|
|Azure Resource Manager (ユーザー)|Management. *&lt;region>.&lt;fqdn>*|HTTPS|443|
|グラフ|Graph. *&lt;region>.&lt;fqdn>*|HTTPS|443|
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
|VPN ゲートウェイ|     |     |[VPN Gateway に関する FAQ を参照してください](/azure/vpn-gateway/vpn-gateway-vpn-faq#can-i-traverse-proxies-and-firewalls-using-point-to-site-capability)。|
|     |     |     |     |

## <a name="ports-and-urls-outbound"></a>ポートと URL (送信)

Azure Stack Hub では、透過プロキシ サーバーのみがサポートされます。 透過プロキシから従来のプロキシ サーバーへのアップリンクが存在するデプロイ環境では、次の表のポートと URL に外部への通信を許可する必要があります。 透過プロキシ サーバーの構成の詳細については、「[Azure Stack Hub の透過プロキシ](azure-stack-transparent-proxy.md)」を参照してください。

SSL トラフィックのインターセプトは[サポートされておらず](azure-stack-firewall.md#ssl-interception)、エンドポイントへのアクセスでサービス エラーが発生する可能性があります。 ID に必要なエンドポイントとの通信に対してサポートされる最大タイムアウトは、60 秒です。

> [!Note]  
> ExpressRoute ではすべてのエンドポイントにトラフィックをルーティングできない場合があるため、Azure Stack Hub では、ExpressRoute を使用して、次の表にリストされている Azure サービスに到達することはサポートされていません。

|目的|接続先 URL|Protocol|Port|ソース ネットワーク|
|---------|---------|---------|---------|---------|
|ID|**Azure**<br>login.windows.net<br>login.microsoftonline.com<br>graph.windows.net<br>https:\//secure.aadcdn.microsoftonline-p.com<br>www.office.com<br>ManagementServiceUri = https:\//management.core.windows.net<br>ARMUri = https:\//management.azure.com<br>https:\//\*.msftauth.net<br>https:\//\*.msauth.net<br>https:\//\*.msocdn.com<br>**Azure Government**<br>https:\//login.microsoftonline.us/<br>https:\//graph.windows.net/<br>**Azure China 21Vianet**<br>https:\//login.chinacloudapi.cn/<br>https:\//graph.chinacloudapi.cn/<br>**Azure Germany**<br>https:\//login.microsoftonline.de/<br>https:\//graph.cloudapi.de/|HTTP<br>HTTPS|80<br>443|パブリック VIP - /27<br>パブリック インフラストラクチャ ネットワーク|
|Marketplace シンジケーション|**Azure**<br>https:\//management.azure.com<br>https://&#42;.blob.core.windows.net<br>https://&#42;.azureedge.net<br>**Azure Government**<br>https:\//management.usgovcloudapi.net/<br>https://&#42;.blob.core.usgovcloudapi.net/<br>**Azure China 21Vianet**<br>https:\//management.chinacloudapi.cn/<br>http://&#42;.blob.core.chinacloudapi.cn|HTTPS|443|パブリック VIP - /27|
|パッチと更新プログラム|https://&#42;.azureedge.net<br>https:\//aka.ms/azurestackautomaticupdate|HTTPS|443|パブリック VIP - /27|
|登録|**Azure**<br>https:\//management.azure.com<br>**Azure Government**<br>https:\//management.usgovcloudapi.net/<br>**Azure China 21Vianet**<br>https:\//management.chinacloudapi.cn|HTTPS|443|パブリック VIP - /27|
|使用法|**Azure**<br>https://&#42;.trafficmanager.net<br>**Azure Government**<br>https://&#42;.usgovtrafficmanager.net<br>**Azure China 21Vianet**<br>https://&#42;.trafficmanager.cn|HTTPS|443|パブリック VIP - /27|
|Windows Defender|&#42;.wdcp.microsoft.com<br>&#42;.wdcpalt.microsoft.com<br>&#42;.wd.microsoft.com<br>&#42;.update.microsoft.com<br>&#42;.download.microsoft.com<br><br>https:\//secure.aadcdn.microsoftonline-p.com<br>|HTTPS|80<br>443|パブリック VIP - /27<br>パブリック インフラストラクチャ ネットワーク|
|NTP|(デプロイに提供される NTP サーバーの IP)|UDP|123|パブリック VIP - /27|
|DNS|(デプロイに提供される DNS サーバーの IP)|TCP<br>UDP|53|パブリック VIP - /27|
|SYSLOG|(デプロイに提供される SYSLOG サーバーの IP)|TCP<br>UDP|6514<br>514|パブリック VIP - /27|
|CRL|(証明書上で CRL 配布ポイントの下にある URL)<br>http://crl.microsoft.com/pki/crl/products<br>http://mscrl.microsoft.com/pki/mscorp<br>http://www.microsoft.com/pki/certs<br>http://www.microsoft.com/pki/mscorp<br>http://www.microsoft.com/pkiops/crl<br>http://www.microsoft.com/pkiops/certs<br>|HTTP|80|パブリック VIP - /27|
|LDAP|Graph 統合のために用意されている Active Directory フォレスト|TCP<br>UDP|389|パブリック VIP - /27|
|LDAP SSL|Graph 統合のために用意されている Active Directory フォレスト|TCP|636|パブリック VIP - /27|
|LDAP GC|Graph 統合のために用意されている Active Directory フォレスト|TCP|3268|パブリック VIP - /27|
|LDAP GC SSL|Graph 統合のために用意されている Active Directory フォレスト|TCP|3269|パブリック VIP - /27|
|AD FS|AD FS 統合のために用意されている AD FS メタデータ エンドポイント|TCP|443|パブリック VIP - /27|
| 診断ログの収集 |https://*.blob.core.windows.net<br>https://azsdiagprdlocalwestus02.blob.core.windows.net<br>https://azsdiagprdwestusfrontend.westus.cloudapp.azure.com<br>https://azsdiagprdwestusfrontend.westus.cloudapp.azure.com | HTTPS | 443 | パブリック VIP - /27 |
|     |     |     |     |     |

送信 URL は Azure Traffic Manager を使用して負荷分散され、地理的な場所に基づいて可能な限り最適な接続が提供されます。 URL を負荷分散することで、Microsoft は、顧客に影響を与えることなくバックエンド エンドポイントを更新および変更することができます。 Microsoft では、負荷分散される URL の IP アドレスのリストを共有していません。 IP ではなく URL によるフィルター処理をサポートするデバイスを使用してください。

送信 DNS は常に必要です。違っているのは、外部 DNS のクエリを実行しているソースと、選択された ID 統合の種類です。 接続されたシナリオの場合は、デプロイ時には BMC ネットワーク上に配置された DVM で送信アクセスが必要です。 しかし、デプロイ後は、DNS サービスはパブリック VIP を使用してクエリを送信する内部コンポーネントに移動します。 その時点で、BMC ネットワーク経由での送信 DNS アクセスは削除できますが、その DNS サーバーへのパブリック VIP アクセスは残しておく必要があり、そうしないと認証は失敗します。

## <a name="next-steps"></a>次のステップ

[Azure Stack Hub PKI の要件](azure-stack-pki-certs.md)
