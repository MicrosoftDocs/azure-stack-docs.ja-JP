---
title: Azure Stack Hub の ID アーキテクチャ
description: Azure Stack Hub の ID アーキテクチャと、Azure AD と AD FS の違いについて説明します。
author: BryanLa
ms.topic: conceptual
ms.date: 03/10/2021
ms.author: bryanla
ms.reviewer: fiseraci
ms.lastreviewed: 03/10/2021
ms.openlocfilehash: 5f03c3b1f5ca6aa24c2d54acaa090984ff4db90a
ms.sourcegitcommit: e7d6f953e7014900b4e7d710340cfa98d253fce9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/11/2021
ms.locfileid: "102637614"
---
# <a name="identity-architecture-for-azure-stack-hub"></a>Azure Stack Hub の ID アーキテクチャ

Azure Stack Hub で使用する ID プロバイダーを選択する場合は、Azure Active Directory (Azure AD) と Active Directory フェデレーション サービス (AD FS) のオプションの重要な違いを理解する必要があります。

## <a name="capabilities-and-limitations"></a>機能と制限

選択する ID プロバイダーによっては、マルチテナントのサポートなどのオプションが制限されることがあります。

|機能またはシナリオ        |Azure AD  |AD FS  |
|------------------------------|----------|-------|
|インターネットに接続されている     |はい       |省略可能|
|マルチテナントのサポート     |はい       |いいえ      |
|Marketplace で項目を提供する |はい       |はい ([オフラインの Marketplace シンジケーション](azure-stack-download-azure-marketplace-item.md?pivots=state-disconnected) ツールを使用する必要があります)|
|Active Directory Authentication Library (ADAL) のサポート |はい |はい|
|Azure CLI、Visual Studio、PowerShell などのツールのサポート  |はい |はい|
|Azure Portal を通じてサービス プリンシパルを作成する     |はい |いいえ|
|証明書でサービス プリンシパルを作成する      |はい |はい|
|シークレット (キー) でサービス プリンシパルを作成する    |はい |はい|
|アプリケーションが Graph サービスを使用できる           |はい |いいえ|
|アプリケーションがサインインに ID プロバイダーを使用できる |はい |はい (アプリがオンプレミスの AD FS インスタンスとフェデレーションする必要があります) |
| マネージド システム ID | いいえ | いいえ |

## <a name="topologies"></a>トポロジ

以降のセクションでは、使用できる各種 ID トポロジについて説明します。

### <a name="azure-ad-single-tenant-topology"></a>Azure AD: シングルテナント トポロジ

既定では、Azure Stack Hub をインストールして Azure AD を使用すると、Azure Stack Hub でシングルテナント トポロジが使用されます。

シングルテナント トポロジは、次のような場合に便利です。
- すべてのユーザーが、同じテナントに属している。
- サービス プロバイダーが、組織の Azure Stack Hub インスタンスをホストする。

![Azure AD による Azure Stack Hub のシングルテナント トポロジ](media/azure-stack-identity-architecture/single-tenant.svg)

このトポロジには次の特徴があります。

- Azure Stack Hub では、すべてのアプリとサービスが同一の Azure AD テナント ディレクトリに登録されます。
- Azure Stack Hub では、トークンを含む、そのディレクトリのユーザーとアプリのみが認証されます。
- 管理者 (クラウド オペレーター) とテナント ユーザーの ID が、同じディレクトリ テナントにあります。
- 別のディレクトリのユーザーがこの Azure Stack Hub 環境にアクセスできるようにするには、テナント ディレクトリに[ユーザーをゲストとして招待](azure-stack-identity-overview.md#guest-users)する必要があります。

### <a name="azure-ad-multi-tenant-topology"></a>Azure AD: マルチテナント トポロジ

クラウド オペレーターは、1 つまたは複数の組織のテナントからアプリにアクセスすることを許可するように、Azure Stack Hub を構成できます。 ユーザーは、Azure Stack Hub ユーザー ポータルからアプリにアクセスします。 この構成では、管理者ポータル (クラウド オペレーターによって使用されます) は、単一のディレクトリのユーザーに限定されます。

マルチテナント トポロジは、次のような場合に便利です。

- サービス プロバイダーが、複数の組織のユーザーに Azure Stack Hub へのアクセスを許可しようとしている。

![Azure AD による Azure Stack Hub のマルチテナント トポロジ](media/azure-stack-identity-architecture/multi-tenant.svg)

このトポロジには次の特徴があります。

- リソースへのアクセスは、組織単位で行う必要があります。
- 1 つの組織のユーザーが組織外のユーザーにリソースへのアクセスを許可できないようにする必要があります。
- 管理者 (クラウド オペレーター) の ID は、ユーザーの ID とは別のディレクトリ テナントに置くことができます。 このように分けることで、ID プロバイダー レベルでのアカウント分離が実現します。
 
### <a name="ad-fs"></a>AD FS

AD FS トポロジは、次のいずれかの条件に該当する場合に必要です。

- Azure Stack Hub がインターネットに接続されない。
- Azure Stack Hub はインターネットに接続できるが、ID プロバイダーのために AD FS を使用することを選択する。
  
![AD FS を使用する Azure Stack Hub トポロジ](media/azure-stack-identity-architecture/adfs.svg)

このトポロジには次の特徴があります。

- 運用環境でのこのトポロジの使用をサポートするには、フェデレーションの信頼を介して、組み込みの Azure Stack Hub AD FS インスタンスを、Active Directory で支援されている既存の AD FS インスタンスに統合する必要があります。
- Azure Stack Hub の Graph サービスを、既存の Active Directory インスタンスと統合することができます。 また、Azure AD Graph API との一貫性がある API をサポートしている OData ベースの Graph API サービスを使用することもできます。

  Active Directory インスタンスと対話するためには、Graph API に、Active Directory インスタンスに対する読み取り専用アクセス許可を持つユーザーの資格情報が必要です。これは次にアクセスします。  
  - 組み込みの AD FS インスタンス。
  - 使用する AD FS と Active Directory インスタンス。これらは、Windows Server 2012 以降をベースとするものである必要があります。
  
  使用する Active Directory インスタンスと組み込み AD FS インスタンスの間で、対話は OpenID Connect に限定されず、相互にサポートされている任意のプロトコルを使用できます。
  - ユーザー アカウントは、オンプレミスの Active Directory インスタンスで作成され、管理されます。
  - アプリのサービス プリンシパルと登録は、組み込みの Active Directory インスタンス内で管理されます。

## <a name="next-steps"></a>次のステップ

- [ID の概要](azure-stack-identity-overview.md)
- [データセンターの統合 - ID](azure-stack-integrate-identity.md)
