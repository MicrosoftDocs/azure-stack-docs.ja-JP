---
title: Azure Stack Hub 統合システムの Azure に接続されたデプロイの決定
description: 課金や ID など、Azure Stack Hub 統合システムの Azure に接続されたデプロイに対するデプロイ計画を決定します。
author: PatAltimore
ms.topic: conceptual
ms.date: 03/04/2020
ms.author: patricka
ms.reviewer: wfayed
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: f929654eb723f900ebe8a2beefd84afb064e7056
ms.sourcegitcommit: 4f1d22747c02ae280609174496933fca8c04a6cf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102606382"
---
# <a name="azure-connected-deployment-planning-decisions-for-azure-stack-hub-integrated-systems"></a>Azure Stack Hub 統合システムの Azure に接続されたデプロイ計画の決定
[Azure Stack Hub をご利用のハイブリッド クラウド環境に統合する方法](azure-stack-connection-models.md)を決定した後、Azure Stack Hub デプロイの決定を完了することができます。

Azure に接続された Azure Stack Hub をデプロイすることは、ID ストアとして Azure Active Directory (Azure AD) または Active Directory フェデレーション サービス (AD FS) のどちらかを使用できることを意味します。 また、従量制または容量ベースのどちらかの課金モデルも選択できます。 接続されたデプロイは、特に Azure と Azure Stack Hub の両方を含むハイブリッド クラウドのシナリオの場合、顧客が Azure Stack Hub を最大限に活用できるため、既定のオプションになります。

## <a name="choose-an-identity-store"></a>ID ストアを選択する
接続されたデプロイでは、ID ストアとして Azure AD または AD FS のどちらかを選択できます。 インターネット接続のない切断されたデプロイは、AD FS しか使用できません。

ID ストアの選択は、テナント仮想マシン (VM) とは何の関係もありません。 テナント VM では、構成される方法に応じて接続先の ID ストアを選択できます (Azure AD、参加している Windows Server Active Directory ドメイン、ワークグループなど)。 これは、Azure Stack Hub ID プロバイダーの決定とは関係ありません。

たとえば、Azure Stack Hub の上に IaaS テナント VM をデプロイし、それを企業の Active Directory ドメインに参加させ、そこからアカウントを使用する場合は、引き続き実行できます。 それらのアカウント用に、ここで選択した Azure AD ID ストアを使用する必要はありません。

### <a name="azure-ad-identity-store"></a>Azure AD ID ストア
ID ストアに Azure AD を使用するには、グローバル管理者アカウントと課金アカウントの 2 つの Azure AD アカウントが必要です。 これらのアカウントは同じアカウントにすることも、別のアカウントにすることもできます。 Azure アカウントの数が制限されている場合は同じユーザー アカウントを使用する方が簡単で便利な可能性がありますが、ビジネス ニーズによって 2 つのアカウントの使用が指定されることもあります。

1. **グローバル管理者アカウント** (接続されたデプロイにのみ必要です)。 これは、Azure AD 内の Azure Stack Hub インフラストラクチャ サービスのためのアプリとサービス プリンシパルを作成するために使用される Azure アカウントです。 このアカウントには、ご利用の Azure Stack Hub システムがデプロイされるディレクトリへのディレクトリ管理者権限が必要です。 これが Azure AD ユーザーの "クラウド オペレーター" グローバル管理者になり、次のタスクのために使用されます。

    - Azure AD および Graph API とやりとりするために必要な、すべての Azure Stack Hub サービス用のアプリとサービス プリンシパルのプロビジョニングと委任を行う。
    - サービス管理者アカウントとして。 このアカウントは、既定のプロバイダー サブスクリプションの所有者 (後で変更可能) です。 このアカウントで Azure Stack Hub 管理者ポータルにログインできるほか、これを使用してオファーやプランを作成したり、クォータを設定したり、Azure Stack Hub のその他の管理機能を実行したりすることができます。

> [!IMPORTANT]
> - グローバル管理者アカウントは、Azure Stack Hub を実行するには必要ではなく、デプロイ後に無効にすることができます。
> - [こちらに記載されているベスト プラクティス](/azure/security/fundamentals/identity-management-best-practices)に従って、グローバル管理者アカウントをセキュリティで保護します。


2. **課金アカウント** (接続されたデプロイと切断されたデプロイの両方に必要です)。 この Azure アカウントは、Azure Stack Hub 統合システムと Azure コマース バックエンドの課金関係を確立するために使用されます。 これは、Azure Stack Hub の料金に対して課金されるアカウントです。 このアカウントは、マーケットプレイスや他のハイブリッド シナリオで項目を提供するためにも使用されます。

### <a name="ad-fs-identity-store"></a>AD FS ID ストア
このオプションは、サービス管理者アカウント用に独自の ID ストア (企業の Active Directory など) を使用する場合に選択します。  

## <a name="choose-a-billing-model"></a>課金モデルを選択する
**従量制** または **容量** 課金モデルのどちらかを選択できます。 従量制課金モデルのデプロイは 30 日ごとに少なくとも 1 回、接続を通して Azure に使用状況を報告できる必要があります。 したがって、従量制課金モデルは、接続されたデプロイでのみ使用できます。  

### <a name="pay-as-you-use"></a>従量制
従量制課金モデルでは、Azure サブスクリプションに対して使用状況が課金されます。 Azure Stack Hub サービスを使用した場合にのみ支払います。 このモデルに決定した場合は、Azure サブスクリプションと、そのサブスクリプションに関連付けられたアカウント ID (serviceadmin@contoso.onmicrosoft.com など) が必要になります。 EA、CSP、および CSP Shared Services のサブスクリプションがサポートされています。 使用状況レポートは、[Azure Stack Hub の登録](azure-stack-registration.md)中に構成されます。

> [!NOTE]
> ほとんどの場合、エンタープライズ顧客は EA サブスクリプションを使用し、サービス プロバイダーは CSP または CSP Shared Services サブスクリプションを使用します。

CSP サブスクリプションを使用する場合は、厳密な CSP シナリオによって正しいアプローチが異なるため、下の表を確認してどの CSP サブスクリプションを使用するかを特定してください。

|シナリオ|ドメインおよびサブスクリプション オプション|
|-----|-----|
|**直接 CSP パートナー** または **間接 CSP プロバイダー** であり、Azure Stack Hub を操作する|CSP Partner Shared Services サブスクリプションを使用します。<br>     または<br>パートナー センターで、わかりやすい名前の Azure AD テナントを作成します  (例: &lt;組織>CSPAdmin とそれに関連付けられた Azure CSP サブスクリプション)。|
|**間接 CSP リセラー** であり、Azure Stack Hub を操作する|パートナー センターを使用して、間接 CSP プロバイダーに、組織の Azure AD テナントとそれに関連付けられた Azure CSP サブスクリプションを作成するよう依頼します。|

### <a name="capacity-based-billing"></a>容量ベースの課金
容量課金モデルを使用することにした場合は、システムの容量に基づいて Azure Stack Hub 容量プラン SKU を購入する必要があります。 正しい数量を購入するには、Azure Stack Hub 内の物理コアの数を把握する必要があります。

容量課金では、登録のために Enterprise Agreement (EA) Azure サブスクリプションが必要です。 その理由は、登録によって Marketplace での項目の入手可能性が設定されるためです。これには Azure サブスクリプションが必要です。 Azure Stack Hub の使用には、このサブスクリプションは使用されません。

## <a name="learn-more"></a>詳細情報
- ユース ケース、購入、パートナー、OEM ハードウェア ベンダーの詳細については、[Azure Stack Hub](https://azure.microsoft.com/overview/azure-stack/) の製品ページを参照してください。
- Azure Stack Hub 統合システムのロードマップと地理的な可用性については、ホワイト ペーパー「[Azure Stack Hub: An extension of Azure](https://azure.microsoft.com/resources/videos/azure-friday-azure-stack-an-extension-of-azure/)」 (Azure Stack: Azure の拡張機能) を参照してください。 
- Microsoft Azure Stack Hub のパッケージと価格の詳細については、[.pdf をダウンロードしてください](https://azure.microsoft.com/mediahandler/files/resourcefiles/5bc3f30c-cd57-4513-989e-056325eb95e1/Azure-Stack-packaging-and-pricing-datasheet.pdf)。 

## <a name="next-steps"></a>次のステップ
[データセンターのネットワーク統合](azure-stack-network.md)
