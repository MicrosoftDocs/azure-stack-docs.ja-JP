---
title: Azure Stack Hub 公開キー インフラストラクチャ証明書の要件
description: Azure Stack Hub 統合システムの Azure Stack Hub PKI 証明書の要件について説明します。
author: BryanLa
ms.topic: conceptual
ms.date: 08/19/2020
ms.author: bryanla
ms.reviewer: ppacent
ms.lastreviewed: 12/16/2019
ms.openlocfilehash: 33beba13cf9d71cf2b591f6b3ee87a8076a8172d
ms.sourcegitcommit: ccc4ee05d71496653b6e27de1bb12e4347e20ba4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102231219"
---
# <a name="azure-stack-hub-public-key-infrastructure-pki-certificate-requirements"></a>Azure Stack Hub 公開キー インフラストラクチャ (PKI) 証明書の要件

Azure Stack Hub には、少数の Azure Stack Hub サービスやテナント VM に割り当てられた外部からアクセス可能なパブリック IP アドレスを使用するパブリック インフラストラクチャ ネットワークが存在します。 Azure Stack Hub のデプロイ中に、これらの Azure Stack Hub パブリック インフラストラクチャ エンドポイントの適切な DNS 名を持つ PKI 証明書が必要です。 この記事では、次の項目に関する情報を提供します。

- Azure Stack Hub の証明書要件。
- Azure Stack Hub のデプロイに必須の証明書。
- 付加価値リソース プロバイダーをデプロイするときに必要なオプションの証明書。

> [!NOTE]
> 既定で Azure Stack Hub では、ノード間の認証に、内部の Active Directory 統合証明機関 (CA) から発行された証明書も使用されます。 証明書を検証するには、すべての Azure Stack Hub インフラストラクチャ マシンで、その証明書をローカルの証明書ストアに追加することによって、内部 CA のルート証明書を信頼します。 Azure Stack Hub には、証明書のピン留めもフィルター処理もありません。 各サーバー証明書の SAN は、ターゲットの FQDN に対して検証されます。 証明書の有効期限 (証明書のピン留めがない標準 TLS サーバー認証) と共に、信頼チェーン全体も検証されます。

## <a name="certificate-requirements"></a>証明書の要件
次の一覧では、一般的な証明書の発行、セキュリティ、および書式設定の要件について説明します。

- 証明書は、内部の証明機関または公的証明機関のどちらかから発行されている必要があります。 公的証明機関が使用されている場合は、Microsoft の信頼されたルート機関プログラムの一部として基本オペレーティング システム イメージに含まれている必要があります。 完全な一覧は、「[参加者の一覧 - Microsoft の信頼されたルート プログラム](/security/trusted-root/participants-list)」を参照してください。
- お使いの Azure Stack Hub インフラストラクチャは、証明書において公開されている証明機関の証明書失効リスト (CRL) の場所にネットワークでアクセスできる必要があります。 この CRL は、http エンドポイントである必要があります。
::: moniker range="< azs-1903"
- 1903 より前のビルドで証明書を交換する場合、証明書は、デプロイ時に指定された証明書の署名に使用したのと同じ内部の証明機関、または上記の公的パブリック証明機関のいずれかから発行されたものである必要があります。
::: moniker-end
::: moniker range=">= azs-1903"
- 1903 以降のビルドで証明書を交換する場合、証明書は、企業や公共の証明機関によって発行できます。
::: moniker-end
- 自己署名証明書の使用はサポートされていません。
- デプロイおよびローテーションの場合は、証明書のサブジェクト名フィールドとサブジェクトの別名 (SAN) フィールドのすべての名前空間をカバーする 1 つの証明書を使用するか、各名前空間で利用する予定の Azure Stack Hub サービスで必要な個別の証明書を使用することができます。 どちらの方法でも、それらが必要とされるエンドポイントに対してワイルド カードを使用する必要があります (例: **KeyVault**､**KeyVaultInternal**)。
- 証明書の署名アルゴリズムを SHA1 にしないでください。
- Azure Stack Hub のインストールには公開キーと秘密キーの両方が必要なため、証明書の形式は PFX である必要があります。 秘密キーにはローカル コンピューターのキー属性が設定されている必要があります。
- PFX 暗号化は、3DES (この暗号化は、Windows 10 クライアントまたは Windows Server 2016 証明書ストアからエクスポートする場合の既定です) である必要があります。
- 証明書 pfx ファイルの "Key Usage" フィールドには、"Digital Signature" と "KeyEncipherment" の値が含まれている必要があります。
- 証明書の pfx ファイルの "Enhanced Key Usage" フィールドには、"Server Authentication (1.3.6.1.5.5.7.3.1)" と "Client Authentication (1.3.6.1.5.5.7.3.2)" の値が含まれている必要があります。
- 証明書の "Issued to:" フィールドは "Issued by:" フィールドと同じにしないでください。
- デプロイの時点で、すべての証明書 pfx ファイルのパスワードが同じである必要があります。
- 証明書 pfx に対するパスワードは、複雑なパスワードである必要があります。 このパスワードはデプロイ パラメーターとして使用するため、メモしておいてください。 パスワードは、次のパスワードの複雑さ要件を満たしている必要があります。
    - 8 文字の最小長。
    - 大文字、小文字、0 から 9 までの数字、特殊文字、および大文字でも小文字でもない英字のうちの少なくとも 3 種類。
- サブジェクト名と、サブジェクトの別名の拡張子 (x509v3_config) のサブジェクトの別名が一致するようにします。 サブジェクトの別名フィールドでは、単一の SSL 証明書によって保護される追加のホスト名 (Web サイト、IP アドレス、共通名) を指定できます。

> [!NOTE]  
> 自己署名証明書はサポートされていません。  
> Azure Stack Hub を切断モードでデプロイする場合は、エンタープライズ証明機関によって発行された証明書を使用することをお勧めします。 Azure Stack Hub エンドポイントにアクセスするクライアントは、証明書失効リスト (CRL) にアクセスできる必要があるため、これは重要です。

> [!NOTE]  
> 証明書の信頼チェーン内での中間証明機関の存在が *サポートされています*。

## <a name="mandatory-certificates"></a>必須の証明書
このセクションの表では、Azure AD と AD FS Azure Stack Hub の両方のデプロイに必要な Azure Stack Hub パブリック エンドポイント PKI 証明書について説明します。 証明書の要件が領域のほか、使用される名前空間や、名前空間ごとに必要な証明書でグループ化されています。 この表ではまた、ソリューション プロバイダーがパブリック エンドポイントごとに異なる証明書をコピーするフォルダーについても説明します。

各 Azure Stack Hub パブリック インフラストラクチャ エンドポイントの適切な DNS 名を持つ証明書が必要です。 各エンドポイントの DNS 名は、 *&lt;prefix>.&lt;region>.&lt;fqdn>* の形式で表されます。

デプロイでは、[region] と [externalfqdn] の値が Azure Stack Hub システム用に選択したリージョン名と外部ドメイン名に一致している必要があります。 例として、リージョン名が *Redmond* で、外部ドメイン名が *contoso.com* である場合、DNS 名は *&lt;prefix>.redmond.contoso.com* の形式になります。 *&lt;prefix>* 値は、その証明書によってセキュリティ保護されるエンドポイントを指定するようにマイクロソフトによって事前に設計されています。 さらに、外部インフラストラクチャ エンドポイントの *&lt;prefix>* 値は、特定のエンドポイントを使用する Azure Stack Hub サービスによって異なります。

運用環境では、各証明書がエンドポイントごとに生成され、対応するディレクトリにコピーされることをお勧めします。 開発環境では、証明書は、すべてのディレクトリにコピーされるサブジェクト フィールドおよびサブジェクトの別名 (SAN) フィールドのすべての名前空間をカバーする 1 つのワイルドカード証明書として提供することができます。 すべてのエンドポイントとサービスを対象とする単一の証明書は安全ではないため、開発専用です。 どちらのオプションでも、証明書を必要とする **acs** や Key Vault などのエンドポイントにはワイルドカード証明書を使う必要があることに注意してください。

> [!Note]  
> デプロイ時には、その対象となる ID プロバイダー (Azure AD または AD FS) に合ったデプロイ フォルダーに証明書をコピーする必要があります。 すべてのエンドポイントについて 1 つの証明書を使用する場合は、次の表に記載した各デプロイ フォルダーに、その証明書ファイルをコピーする必要があります。フォルダー構造は、デプロイ仮想マシンにあらかじめ作成されており、C:\CloudDeployment\Setup\Certificates で確認できます。

| デプロイ フォルダー | 必要な証明書のサブジェクト名とサブジェクトの別名 (SAN) | スコープ (リージョンごと) | サブドメインの名前空間 |
|-------------------------------|------------------------------------------------------------------|----------------------------------|-----------------------------|
| パブリック ポータル | portal.&lt;region>.&lt;fqdn> | ポータル | &lt;region>.&lt;fqdn> |
| 管理ポータル | adminportal.&lt;region>.&lt;fqdn> | ポータル | &lt;region>.&lt;fqdn> |
| Azure Resource Manager パブリック | management.&lt;region>.&lt;fqdn> | Azure Resource Manager | &lt;region>.&lt;fqdn> |
| Azure Resource Manager 管理 | adminmanagement.&lt;region>.&lt;fqdn> | Azure Resource Manager | &lt;region>.&lt;fqdn> |
| ACSBlob | *.blob.&lt;region>.&lt;fqdn><br>(ワイルドカード SSL 証明書) | Blob Storage | blob.&lt;region>.&lt;fqdn> |
| ACSTable | *.table.&lt;region>.&lt;fqdn><br>(ワイルドカード SSL 証明書) | Table Storage | table.&lt;region>.&lt;fqdn> |
| ACSQueue | *.queue.&lt;region>.&lt;fqdn><br>(ワイルドカード SSL 証明書) | Queue Storage | queue.&lt;region>.&lt;fqdn> |
| KeyVault | *.vault.&lt;region>.&lt;fqdn><br>(ワイルドカード SSL 証明書) | Key Vault | vault.&lt;region>.&lt;fqdn> |
| KeyVaultInternal | *.adminvault.&lt;region>.&lt;fqdn><br>(ワイルドカード SSL 証明書) |  内部 Keyvault |  adminvault.&lt;region>.&lt;fqdn> |
| 管理者拡張機能ホスト | *.adminhosting.\<region>.\<fqdn> (ワイルドカード SSL 証明書) | 管理者拡張機能ホスト | adminhosting.\<region>.\<fqdn> |
| パブリック拡張機能ホスト | *.hosting.\<region>.\<fqdn> (ワイルドカード SSL 証明書) | パブリック拡張機能ホスト | hosting.\<region>.\<fqdn> |

Azure AD デプロイ モードを使用して Azure Stack Hub をデプロイする場合は、前の表に一覧表示されている証明書を要求するだけで済みます。 ただし、AD FS デプロイ モードを使用して Azure Stack Hub をデプロイする場合は、次の表で説明されている証明書も要求する必要があります。

|デプロイ フォルダー|必要な証明書のサブジェクト名とサブジェクトの別名 (SAN)|スコープ (リージョンごと)|サブドメインの名前空間|
|-----|-----|-----|-----|
|ADFS|adfs. *&lt;region>.&lt;fqdn>*<br>(SSL 証明書)|ADFS|*&lt;region>.&lt;fqdn>*|
|グラフ|graph. *&lt;region>.&lt;fqdn>*<br>(SSL 証明書)|グラフ|*&lt;region>.&lt;fqdn>*|
|

> [!IMPORTANT]
> このセクションに一覧表示されているすべての証明書のパスワードは同じである必要があります。

## <a name="optional-paas-certificates"></a>オプションの PaaS 証明書
Azure Stack Hub がデプロイおよび構成された後に追加の Azure Stack Hub PaaS サービス (SQL、MySQL、App Service、Event Hubs など) をデプロイする予定がある場合は、それらの PaaS サービスのエンドポイントに対応する追加の証明書を要求する必要があります。

> [!IMPORTANT]
> リソース プロバイダーに使用する証明書は、グローバル Azure Stack Hub エンドポイントに使用されるものとルート証明機関が同じである必要があります。

次の表では、リソースプロバイダーに必要なエンドポイントと証明書について説明します。 これらの証明書を Azure Stack Hub デプロイ フォルダーにコピーする必要はありません。 リソースプロバイダーをインストールするときにこれらの証明書を指定します。

|スコープ (リージョンごと)|Certificate|必要な証明書のサブジェクト名とサブジェクトの別名 (SAN)|サブドメインの名前空間|
|-----|-----|-----|-----|
|App Service|Web トラフィックの既定の SSL 証明書|&#42;.appservice. *&lt;region>.&lt;fqdn>*<br>&#42;.scm.appservice. *&lt;region>.&lt;fqdn>*<br>&#42;.sso.appservice. *&lt;region>.&lt;fqdn>*<br>(マルチドメイン ワイルドカード SSL 証明書<sup>1</sup>)|appservice. *&lt;region>.&lt;fqdn>*<br>scm.appservice. *&lt;region>.&lt;fqdn>*|
|App Service|API|api.appservice. *&lt;region>.&lt;fqdn>*<br>(SSL 証明書<sup>2</sup>)|appservice. *&lt;region>.&lt;fqdn>*<br>scm.appservice. *&lt;region>.&lt;fqdn>*|
|App Service|FTP|ftp.appservice. *&lt;region>.&lt;fqdn>*<br>(SSL 証明書<sup>2</sup>)|appservice. *&lt;region>.&lt;fqdn>*<br>scm.appservice. *&lt;region>.&lt;fqdn>*|
|App Service|SSO|sso.appservice. *&lt;region>.&lt;fqdn>*<br>(SSL 証明書<sup>2</sup>)|appservice. *&lt;region>.&lt;fqdn>*<br>scm.appservice. *&lt;region>.&lt;fqdn>*|
|Event Hubs|SSL|&#42;.eventhub. *&lt;region>.&lt;fqdn>*<br>(ワイルドカード SSL 証明書)|eventhub. *&lt;region>.&lt;fqdn>* |
|IoT Hub|SSL|&#42;.mgmtiothub. *&lt;region>.&lt;fqdn>*<br>(ワイルドカード SSL 証明書)|mgmtiothub. *&lt;region>.&lt;fqdn>* |
|SQL、MySQL|SQL および MySQL|&#42;.dbadapter. *&lt;region>.&lt;fqdn>*<br>(ワイルドカード SSL 証明書)|dbadapter. *&lt;region>.&lt;fqdn>*|

<sup>1</sup> 複数のワイルドカード サブジェクトの別名を持つ 1 つの証明書が必要です。 単一の証明書での複数のワイルドカード SAN は、公的証明機関によってはサポートされていない場合があります。

<sup>2</sup> これら 3 つの証明書 (api.appservice. *&lt;region>.&lt;fqdn>* 、ftp.appservice. *&lt;region>.&lt;fqdn>* 、sso.appservice. *&lt;region>.&lt;fqdn>* ) の代わりに &#42;.appservice. *&lt;region>.&lt;fqdn>* ワイルドカード証明書を使用することはできません。 App Service では、これらのエンドポイントに個別の証明書を明示的に使用する必要があります。

## <a name="learn-more"></a>詳細情報
[Azure Stack Hub デプロイのための PKI 証明書を生成する](azure-stack-get-pki-certs.md)方法について理解を深めましょう。

## <a name="next-steps"></a>次のステップ
[AD FS ID を Azure Stack Hub データセンターに統合する](azure-stack-integrate-identity.md)
