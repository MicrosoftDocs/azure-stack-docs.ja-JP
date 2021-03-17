---
title: Azure App Service on Azure Stack Hub をデプロイするための前提条件
description: Azure App Service on Azure Stack Hub をデプロイするための前提となる手順について説明します。これらは、デプロイする前に完了しておく必要があります。
author: BryanLa
ms.topic: article
ms.date: 10/28/2020
ms.author: anwestg
ms.reviewer: anwestg
ms.lastreviewed: 10/28/2019
zone_pivot_groups: state-connected-disconnected
ms.openlocfilehash: 82c97abf81226c22e2878bb6e6947d53f79cba77
ms.sourcegitcommit: 02a4c34fb829e053016912a4fffcc51e32685425
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102532445"
---
# <a name="prerequisites-for-deploying-app-service-on-azure-stack-hub"></a>App Service on Azure Stack Hub のデプロイの前提条件

[!INCLUDE [Azure Stack Hub update reminder](../includes/app-service-hub-update-banner.md)]

Azure App Service on Azure Stack Hub をデプロイする前に、この記事にある、前提となる手順を完了する必要があります。

## <a name="before-you-get-started"></a>開始する前に 

このセクションでは、統合システムと Azure Stack Development Kit (ASDK) の両方のデプロイの前提条件を示します。

### <a name="resource-provider-prerequisites"></a>リソース プロバイダーの前提条件

[!INCLUDE [Common RP prerequisites](../includes/resource-provider-prerequisites.md)]

### <a name="installer-and-helper-scripts"></a>インストーラーとヘルパー スクリプト

1. [Azure App Service on Azure Stack Hub デプロイ ヘルパー スクリプト](https://aka.ms/appsvconmashelpers)をダウンロードします。
2. [Azure App Service on Azure Stack Hub インストーラー](https://aka.ms/appsvconmasinstaller)をダウンロードします。
3. ヘルパー スクリプトの .zip ファイルからファイルを展開します。 次のファイルとフォルダーが展開されます。

   - Common.ps1
   - Create-AADIdentityApp.ps1
   - Create-ADFSIdentityApp.ps1
   - Create-AppServiceCerts.ps1
   - Get-AzureStackRootCert.ps1
   - モジュール フォルダー
     - GraphAPI.psm1

<!-- MultiNode Only --->
## <a name="certificates-and-server-configuration-integrated-systems"></a>証明書とサーバーの構成 (統合システム)

このセクションでは、統合システムのデプロイの前提条件を示します。

### <a name="certificate-requirements"></a>証明書の要件

運用環境でリソース プロバイダーを実行するには、次の証明書を提供する必要があります。

- 既定のドメイン証明書
- API 証明書
- 公開証明書
- ID 証明書

次のセクションに記載されている特定の要件に加えて、後でツールを使用して一般的な要件をテストすることもできます。 次のものを含む検証の完全な一覧については、「[Azure Stack Hub PKI 証明書の検証](azure-stack-validate-pki-certs.md)」を参照してください。
- .PFX の **ファイル形式**
- サーバーとクライアントの認証に設定された **キー使用法**
- その他

#### <a name="default-domain-certificate"></a>既定のドメイン証明書

既定のドメイン証明書は、フロントエンド ロールに配置されます。 ユーザー アプリが Azure App Service へのワイルドカードまたは既定のドメイン要求のためにこの証明書を使用します。 この証明書は、ソース管理操作 (Kudu) にも使用されます。

この証明書は .pfx 形式の、3 つのサブジェクトのワイルドカード証明書である必要があります。 この要件により、1 つの証明書で、既定のドメインとソース管理操作の SCM エンドポイントの両方に対応できます。

| Format | 例 |
| --- | --- |
| `*.appservice.<region>.<DomainName>.<extension>` | `*.appservice.redmond.azurestack.external` |
| `*.scm.appservice.<region>.<DomainName>.<extension>` | `*.scm.appservice.redmond.azurestack.external` |
| `*.sso.appservice.<region>.<DomainName>.<extension>` | `*.sso.appservice.redmond.azurestack.external` |

#### <a name="api-certificate"></a>API 証明書

API 証明書は、管理ロールに配置されます。 リソース プロバイダーは、これを使用して API 呼び出しをセキュリティ保護しやすくします。 公開用の証明書には、API の DNS エントリと一致するサブジェクトが含まれている必要があります。

| Format | 例 |
| --- | --- |
| api.appservice.\<region\>.\<DomainName\>.\<extension\> | api.appservice.redmond.azurestack.external |

#### <a name="publishing-certificate"></a>公開証明書

パブリッシャー ロール用の証明書は、コンテンツ アップロード時に、アプリ所有者の FTPS トラフィックを保護します。 公開用の証明書には、FTPS の DNS エントリと一致するサブジェクトが含まれている必要があります。

| Format | 例 |
| --- | --- |
| ftp.appservice.\<region\>.\<DomainName\>.\<extension\> | ftp.appservice.redmond.azurestack.external |

#### <a name="identity-certificate"></a>ID 証明書

ID アプリ用の証明書は以下が可能です。

- コンピューティング リソース プロバイダーとの統合をサポートするための Azure Active Directory (Azure AD) または Active Directory フェデレーション サービス (AD FS) ディレクトリ、Azure Stack Hub、App Service の間の統合。
- Azure App Service on Azure Stack Hub 内の複数の高度な開発者ツールでのシングル サインオン シナリオ。

ID 用の証明書には、次の形式に一致するサブジェクトが含まれている必要があります。

| Format | 例 |
| --- | --- |
| sso.appservice.\<region\>.\<DomainName\>.\<extension\> | sso.appservice.redmond.azurestack.external |

### <a name="validate-certificates"></a>証明書を検証する

App Service リソース プロバイダーをデプロイする前に、[PowerShell ギャラリー](https://aka.ms/AzsReadinessChecker)の Azure Stack Hub 適合性チェッカー ツールを使用して、[使用する証明書を検証](azure-stack-validate-pki-certs.md)する必要があります。 Azure Stack Hub 適合性チェッカー ツールは、生成された PKI 証明書が App Service のデプロイに適しているかを検証します。

ベスト プラクティスとして、必要ないずれかの [Azure Stack Hub PKI 証明書](azure-stack-pki-certs.md)を使用する場合には、証明書をテストして必要な場合には再発行するための十分な時間を予定することです。

### <a name="prepare-the-file-server"></a>ファイル サーバーを準備する

Azure App Service では、ファイル サーバーを使用する必要があります。 運用環境のデプロイの場合は、ファイル サーバーを高可用性で、かつ障害を処理できるように構成する必要があります。

#### <a name="quickstart-template-for-highly-available-file-server-and-sql-server"></a>高可用性ファイル サーバーと SQL Server のクイック スタート テンプレート

[参照アーキテクチャ クイックスタート テンプレート](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/appservice-fileserver-sqlserver-ha)が現在使用可能です。これはファイル サーバーと SQL Server をデプロイします。 このテンプレートは、Azure App Service on Azure Stack Hub の高可用性デプロイをサポートするように構成された仮想ネットワークの Active Directory インフラストラクチャをサポートします。

> [!NOTE]
> 統合システム インスタンスでは、デプロイを完了するために、GitHub からリソースをダウンロードできる必要があります。

#### <a name="steps-to-deploy-a-custom-file-server"></a>カスタム ファイル サーバーをデプロイする手順

>[!IMPORTANT]
> App Service を既存の仮想ネットワークにデプロイするように選択した場合、ファイル サーバーを App Service とは別のサブネットにデプロイする必要があります。

>[!NOTE]
> 上記のいずれかのクイックスタート テンプレートを使用してファイル サーバーをデプロイすることを選択した場合は、ファイル サーバーがテンプレートのデプロイの一部として構成されているため、このセクションをスキップできます。

##### <a name="provision-groups-and-accounts-in-active-directory"></a>Active Directory でグループとアカウントをプロビジョニングする

1. 次の Active Directory グローバル セキュリティ グループを作成します。

   - FileShareOwners
   - FileShareUsers

2. 次の Active Directory アカウントをサービス アカウントとして作成します。

   - FileShareOwner
   - FileShareUser

   セキュリティのベスト プラクティスとしては、これらのアカウント (およびすべての Web ロール) のユーザーは一意であり、強力なユーザー名とパスワードを使用する必要があります。 次の条件でパスワードを設定します。

   - **[パスワードを無期限にする]** を有効にします。
   - **[ユーザーはパスワードを変更できない]** を有効にします。
   - **[ユーザーは次回ログオン時にパスワードの変更が必要]** を無効にします。

3. 次のように、グループのメンバーシップにアカウントを追加します。

   - **FileShareOwner** を **FileShareOwners** グループに追加します。
   - **FileShareUser** を **FileShareUsers** グループに追加します。

##### <a name="provision-groups-and-accounts-in-a-workgroup"></a>ワークグループでグループとアカウントをプロビジョニングする

>[!NOTE]
> ファイル サーバーを構成している場合は、**管理者コマンド プロンプト** で次のすべてのコマンドを実行します。 <br>**_PowerShell を使用しないでください。_**

Azure Resource Manager テンプレートを使用するとき、ユーザーは既に作成されています。

1. 次のコマンドを実行して、FileShareOwner アカウントと FileShareUser アカウントを作成します。 `<password>` を独自の値に置き換えます。

   ``` DOS
   net user FileShareOwner <password> /add /expires:never /passwordchg:no
   net user FileShareUser <password> /add /expires:never /passwordchg:no
   ```

2. 次の WMIC コマンドを実行して、アカウントのパスワードを無期限に設定します。

   ``` DOS
   WMIC USERACCOUNT WHERE "Name='FileShareOwner'" SET PasswordExpires=FALSE
   WMIC USERACCOUNT WHERE "Name='FileShareUser'" SET PasswordExpires=FALSE
   ```

3. ローカル グループ FileShareUsers と FileShareOwners を作成し、最初の手順で作成したアカウントをそれらに追加します。

   ``` DOS
   net localgroup FileShareUsers /add
   net localgroup FileShareUsers FileShareUser /add
   net localgroup FileShareOwners /add
   net localgroup FileShareOwners FileShareOwner /add
   ```

#### <a name="provision-the-content-share"></a>コンテンツ共有をプロビジョニングする

コンテンツ共有には、テナント Web サイトのコンテンツが含まれています。 1 つのファイル サーバーでコンテンツ共有をプロビジョニングする手順は、Active Directory とワークグループの両方の環境で同じです。 ただし、Active Directory 内のフェールオーバー クラスターでは異なります。

#### <a name="provision-the-content-share-on-a-single-file-server-active-directory-or-workgroup"></a>1 つのファイル サーバー (Active Directory またはワークグループ) でコンテンツ共有をプロビジョニングする

1 つのファイル サーバーで、管理者特権のコマンド プロンプトで次のコマンドを実行します。 `C:\WebSites` の値を、使用している環境内の対応するパスに置き換えます。

```DOS
set WEBSITES_SHARE=WebSites
set WEBSITES_FOLDER=C:\WebSites
md %WEBSITES_FOLDER%
net share %WEBSITES_SHARE% /delete
net share %WEBSITES_SHARE%=%WEBSITES_FOLDER% /grant:Everyone,full
```

### <a name="configure-access-control-to-the-shares"></a>共有へのアクセス制御を構成する

ファイル サーバーで、またはフェールオーバー クラスター ノード (現在のクラスター リソース所有者) で、管理者特権でのコマンド プロンプトで次のコマンドを実行します。 斜体の値を、使用している環境に固有の値に置き換えます。

#### <a name="active-directory"></a>Active Directory

```DOS
set DOMAIN=<DOMAIN>
set WEBSITES_FOLDER=C:\WebSites
icacls %WEBSITES_FOLDER% /reset
icacls %WEBSITES_FOLDER% /grant Administrators:(OI)(CI)(F)
icacls %WEBSITES_FOLDER% /grant %DOMAIN%\FileShareOwners:(OI)(CI)(M)
icacls %WEBSITES_FOLDER% /inheritance:r
icacls %WEBSITES_FOLDER% /grant %DOMAIN%\FileShareUsers:(CI)(S,X,RA)
icacls %WEBSITES_FOLDER% /grant *S-1-1-0:(OI)(CI)(IO)(RA,REA,RD)
```

#### <a name="workgroup"></a>ワークグループ

```DOS
set WEBSITES_FOLDER=C:\WebSites
icacls %WEBSITES_FOLDER% /reset
icacls %WEBSITES_FOLDER% /grant Administrators:(OI)(CI)(F)
icacls %WEBSITES_FOLDER% /grant FileShareOwners:(OI)(CI)(M)
icacls %WEBSITES_FOLDER% /inheritance:r
icacls %WEBSITES_FOLDER% /grant FileShareUsers:(CI)(S,X,RA)
icacls %WEBSITES_FOLDER% /grant *S-1-1-0:(OI)(CI)(IO)(RA,REA,RD)
```

### <a name="prepare-the-sql-server-instance"></a>SQL Server インスタンスを準備する

> [!NOTE]
> 高可用性ファイル サーバーおよび SQL Server 用のクイック スタート テンプレートをデプロイすることを選択した場合は、テンプレートによって HA 構成で SQL Server がデプロイおよび構成されるため、このセクションをスキップできます。

Azure App Service on Azure Stack Hub のホスティングおよび計測データベースの場合は、App Service データベースを保持するための SQL Server インスタンスを準備する必要があります。

運用と高可用性を目的とする場合は、SQL Server 2014 SP2 以降の完全バージョンを使用し、混合モード認証を有効にして、[高可用性構成](/sql/sql-server/failover-clusters/high-availability-solutions-sql-server)でデプロイする必要があります。

Azure App Service on Azure Stack Hub の SQL Server インスタンスは、すべての App Service ロールからアクセスできる必要があります。 SQL Server は、Azure Stack Hub の既定のプロバイダー サブスクリプション内でデプロイできます。 または、組織内の既存のインフラストラクチャを利用できます (Azure Stack Hub に接続されている場合)。 Azure Marketplace イメージを使用している場合は、それに応じてファイアウォールを構成することを忘れないでください。

> [!NOTE]
> Marketplace 管理機能を介して、さまざまな SQL IaaS VM イメージを使用できます。 Marketplace 項目を使用して VM をデプロイする前に、必ず SQL IaaS 拡張機能の最新バージョンをダウンロードしてください。 これらの SQL イメージは、Azure で使用できる SQL VM と同じです。 これらのイメージから作成された SQL VM の場合、IaaS 拡張機能や対応するポータル拡張機能により、修正プログラムの自動適用やバックアップなどの機能が提供されます。
>
> どの SQL Server ロールの場合も、既定のインスタンスまたは名前付きインスタンスを使用できます。 名前付きインスタンスを使用する場合は、手動で SQL Server Browser サービスを開始し、ポート 1434 を開くようにしてください。

App Service インストーラーは、SQL Server がデータベース包含を有効にしているかどうかを確認します。 App Service データベースをホストする SQL Server に対してデータベース包含を有効にするには、次の SQL コマンドを実行します。

```sql
sp_configure 'contained database authentication', 1;
GO
RECONFIGURE;
GO
```

<!-- ASDK Only --->
## <a name="certificates-and-server-configuration-asdk"></a>証明書とサーバーの構成 (ASDK)

このセクションでは、ASDK のデプロイの前提条件を示します。 

### <a name="certificates-required-for-asdk-deployment-of-azure-app-service"></a>Azure での Azure App Service のデプロイメントに必要な証明書

*Create-AppServiceCerts.ps1* スクリプトは、Azure Stack Hub 証明機関と連携して、App Service で必要となる次の 4 つの証明書を作成します。

| ファイル名 | 用途 |
| --- | --- |
| _.appservice.local.azurestack.external.pfx | App Service の既定の SSL 証明書 |
| api.appservice.local.azurestack.external.pfx | App Service API の SSL 証明書 |
| ftp.appservice.local.azurestack.external.pfx | App Service パブリッシャーの SSL 証明書 |
| sso.appservice.local.azurestack.external.pfx | App Service ID アプリケーションの証明書 |

証明書を作成するには、以下の手順に従ってください。

1. AzureStack\AzureStackAdmin アカウントを使用して ASDK ホストにサインインします。
2. 管理者特権の PowerShell セッションを開きます。
3. ヘルパー スクリプトを展開したフォルダーから *Create-AppServiceCerts.ps1* スクリプトを実行します。 このスクリプトは、証明書を作成するために App Service で必要となるスクリプトと同じフォルダーに 4 つの証明書を作成します。
4. .pfx ファイルを保護するためのパスワードを入力し、そのパスワードを書き留めておきます。 後ほど、これを App Service on Azure Stack Hub インストーラーに入力する必要があります。

#### <a name="create-appservicecertsps1-script-parameters"></a>Create-AppServiceCerts.ps1 スクリプトのパラメーター

| パラメーター | 必須または省略可能 | 既定値 | 説明 |
| --- | --- | --- | --- |
| pfxPassword | 必須 | [Null] | 証明書の秘密キーを保護するのに役立つパスワード |
| DomainName | 必須 | local.azurestack.external | Azure Stack Hub のリージョンとドメイン サフィックス |

### <a name="quickstart-template-for-file-server-for-deployments-of-azure-app-service-on-asdk"></a>ASDK 上の Azure App Service のデプロイ用のファイル サーバーのクイック スタート テンプレート。

ASDK のデプロイのみの場合は、[Azure Resource Manager デプロイ テンプレートの例](https://aka.ms/appsvconmasdkfstemplate)を使用して、構成された単一ノードのファイル サーバーをデプロイできます。 この単一ノードのファイル サーバーは、ワークグループ内に存在します。  

> [!NOTE]
> ASDK インスタンスでは、デプロイを完了するために、GitHub からリソースをダウンロードできる必要があります。

### <a name="sql-server-instance"></a>SQL Server インスタンス (SQL Server instance)

Azure App Service on Azure Stack Hub のホスティングおよび計測データベースの場合は、App Service データベースを保持するための SQL Server インスタンスを準備する必要があります。

ASDK のデプロイの場合は、SQL Server Express 2014 SP2 以降を使用できます。 App Service on Azure Stack Hub は Windows 認証を **サポートしていない** ため、SQL Server は **混合モード** 認証をサポートするように構成する必要があります。

Azure App Service on Azure Stack Hub の SQL Server インスタンスは、すべての App Service ロールからアクセスできる必要があります。 SQL Server は、Azure Stack Hub の既定のプロバイダー サブスクリプション内でデプロイできます。 または、組織内の既存のインフラストラクチャを利用できます (Azure Stack Hub に接続されている場合)。 Azure Marketplace イメージを使用している場合は、それに応じてファイアウォールを構成することを忘れないでください。

> [!NOTE]
> Marketplace 管理機能を介して、さまざまな SQL IaaS VM イメージを使用できます。 Marketplace 項目を使用して VM をデプロイする前に、必ず SQL IaaS 拡張機能の最新バージョンをダウンロードしてください。 これらの SQL イメージは、Azure で使用できる SQL VM と同じです。 これらのイメージから作成された SQL VM の場合、IaaS 拡張機能や対応するポータル拡張機能により、修正プログラムの自動適用やバックアップなどの機能が提供されます。
>
> どの SQL Server ロールの場合も、既定のインスタンスまたは名前付きインスタンスを使用できます。 名前付きインスタンスを使用する場合は、手動で SQL Server Browser サービスを開始し、ポート 1434 を開くようにしてください。

App Service インストーラーは、SQL Server がデータベース包含を有効にしているかどうかを確認します。 App Service データベースをホストする SQL Server に対してデータベース包含を有効にするには、次の SQL コマンドを実行します。

```sql
sp_configure 'contained database authentication', 1;
GO
RECONFIGURE;
GO

```

## <a name="licensing-concerns-for-required-file-server-and-sql-server"></a>必要なファイル サーバーと SQL Server のライセンスの問題

Azure App Service on Azure Stack Hub では、ファイル サーバーと SQL Server が稼働している必要があります。 お客様の Azure Stack Hub デプロイの外部にある既存のリソースを使用しても、Azure Stack Hub の既定のプロバイダー サブスクリプション内のリソースをデプロイしてもかまいません。

Azure Stack Hub の既定のプロバイダー サブスクリプション内のリソースをデプロイする場合は、それらのリソースのライセンス (Windows Server のライセンスと SQL Server のライセンス) が Azure App Service on Azure Stack Hub のコストに含まれます。また、次の制約が適用されます。

- インフラストラクチャは、既定のプロバイダー サブスクリプションにデプロイされます。
- インフラストラクチャは、Azure App Service on Azure Stack Hub リソース プロバイダーによって排他的に使用されます。 他のワークロード (管理 (SQL RP などの他のリソース プロバイダー) またはテナント (データベースを必要とするテナント アプリなど)) には、このインフラストラクチャの使用が許可されません。

## <a name="operational-responsibility-of-file-and-sql-servers"></a>ファイル サーバーと SQL Server の操作の責任

ファイル サーバーと SQL Server の保守と操作を担当するのは、クラウド オペレーターです。  リソース プロバイダーは、これらのリソースの管理は行いません。  クラウドオ ペレーターが、App Service データベースとテナント コンテンツ ファイル共有のバックアップを行います。

## <a name="retrieve-the-azure-resource-manager-root-certificate-for-azure-stack-hub"></a>Azure Stack Hub 用の Azure Resource Manager ルート証明書を取得する

Azure Stack Hub 統合システムまたは ASDK ホストで特権エンドポイントにアクセスできるコンピューター上で、管理者特権の PowerShell セッションを開きます。

ヘルパー スクリプトを展開したフォルダーから *Get-AzureStackRootCert.ps1* スクリプトを実行します。 このスクリプトは、証明書を作成するために App Service で必要となるスクリプトと同じフォルダーにルート証明書を作成します。

次の PowerShell コマンドを実行するときに、AzureStack\CloudAdmin の特権エンドポイントおよび資格情報を提供する必要があります。

```powershell
    Get-AzureStackRootCert.ps1
```

#### <a name="get-azurestackrootcertps1-script-parameters"></a>Get-AzureStackRootCert.ps1 スクリプトのパラメーター

| パラメーター | 必須または省略可能 | 既定値 | 説明 |
| --- | --- | --- | --- |
| PrivilegedEndpoint | 必須 | AzS-ERCS01 | 特権エンドポイント |
| CloudAdminCredential | 必須 | AzureStack\CloudAdmin | Azure Stack Hub クラウド管理者のドメイン アカウントの資格情報 |

## <a name="network-and-identity-configuration"></a>ネットワークと ID の構成

### <a name="virtual-network"></a>仮想ネットワーク

> [!NOTE]
> カスタム仮想ネットワークの事前作成を省略できるのは、Azure App Service on Azure Stack Hub では必須の仮想ネットワークを作成できるためです。ただし、その後、パブリック IP アドレス経由で SQL Server およびファイル サーバーと通信する必要があります。  App Service HA ファイル サーバーと SQL Server クイックスタート テンプレートを使用して、前提条件となる SQL リソースとファイル サーバー リソースをデプロイする場合は、テンプレートによって仮想ネットワークもデプロイされます。

Azure App Service on Azure Stack Hub を使用すると、リソース プロバイダーを既存の仮想ネットワークにデプロイできます。または、デプロイの一部として仮想ネットワークを作成できます。 既存の仮想ネットワークを使用すると、内部 IP を使用して、Azure App Service on Azure Stack Hub で必要なファイル サーバーと SQL Server に接続できます。 Azure App Service on Azure Stack Hub をインストールする前に、仮想ネットワークを次のアドレス範囲とサブネットで構成する必要があります。

仮想ネットワーク - /16

サブネット

- ControllersSubnet /24
- ManagementServersSubnet /24
- FrontEndsSubnet /24
- PublishersSubnet /24
- WorkersSubnet /21

>[!IMPORTANT]
> App Service を既存の仮想ネットワークにデプロイするように選択した場合、SQL Server を App Service とファイル サーバーとは別のサブネットにデプロイする必要があります。
>

### <a name="create-an-identity-application-to-enable-sso-scenarios"></a>SSO シナリオを有効にする ID アプリケーションを作成する

Azure App Service では、ID アプリケーション (サービス プリンシパル) を使用して、次の操作をサポートします。

- worker 層での仮想マシン スケール セットの統合。
- Azure Functions ポータルと高度な開発者ツール (Kudu) のための SSO。

Azure Stack Hub で使用している ID プロバイダー (Azure Active Directory (Azure AD) または Active Directory フェデレーションサービス (ADFS)) に応じて、以下の適切な手順に従って、Azure App Service on Azure Stack Hub リソース プロバイダーで使用するサービス プリンシパルを作成する必要があります。

::: zone pivot="state-connected"
#### <a name="create-an-azure-ad-app"></a>Azure AD アプリを作成する

以下の手順に従って、Azure AD テナント内にサービス プリンシパルを作成します。

1. azurestack\AzureStackAdmin として PowerShell インスタンスを開きます。
1. [前提条件の手順](azure-stack-app-service-before-you-get-started.md)でダウンロードして展開したスクリプトの場所に移動します。
1. [PowerShell for Azure Stack Hub をインストールします](powershell-install-az-module.md)。
1. **Create-AADIdentityApp.ps1** スクリプトを実行します。 メッセージが表示されたら、Azure Stack Hub デプロイのために使用している Azure AD テナント ID を入力します。 たとえば、「**myazurestack.onmicrosoft.com**」と入力します。
1. **[資格情報]** ウィンドウで、Azure AD サービスの管理者アカウントとパスワードを入力します。 **[OK]** を選択します。
1. [先ほど作った証明書](azure-stack-app-service-before-you-get-started.md)について、証明書ファイル パスと証明書パスワードを入力します。 既定でこの手順のために作られる証明書は、**sso.appservice.local.azurestack.external.pfx** です。
1. PowerShell の出力で返されるアプリケーション ID を書き留めておきます。 この ID は、アプリケーションのアクセス許可に同意するための次の手順と、インストール中に使用します。 
1. 新しいブラウザー ウィンドウを開き、Azure Active Directory サービス管理者として [Azure Portal](https://portal.azure.com) にサインインします。
1. Azure Active Directory サービスを開きます。
1. 左ウィンドウで **[アプリの登録]** を選択します。
1. 手順 7 で書き留めたアプリケーション ID を検索します。 
1. 一覧から App Service アプリケーションの登録を選択します。
1. 左ウィンドウで **[API のアクセス許可]** を選択します。
1. **[\<tenant\> に管理者の同意を与えます]** を選択します。\<tenant\> は Azure AD テナントの名前です。 **[はい]** を選択して、同意したことを確認します。

```powershell
    Create-AADIdentityApp.ps1
```

| パラメーター | 必須または省略可能 | 既定値 | 説明 |
| --- | --- | --- | --- |
| DirectoryTenantName | 必須 | [Null] | Azure AD テナント ID。 GUID または文字列を指定します。 例として、myazureaaddirectory.onmicrosoft.com があります。 |
| AdminArmEndpoint | 必須 | [Null] | 管理者の Azure Resource Manager エンドポイント。 例として、adminmanagement.local.azurestack.external があります。 |
| TenantARMEndpoint | 必須 | [Null] | テナントの Azure Resource Manager エンドポイント。 例として、management.local.azurestack.external があります。 |
| AzureStackAdminCredential | 必須 | [Null] | Azure AD サービス管理者の資格情報。 |
| CertificateFilePath | 必須 | [Null] | 先ほど生成された ID アプリケーション証明書ファイルへの **完全なパス**。 |
| CertificatePassword | 必須 | [Null] | 証明書の秘密キーを保護するのに役立つパスワード。 |
| 環境 | 省略可能 | AzureCloud | 対象の Azure Active Directory Graph サービスが利用可能な、サポートされているクラウド環境の名前。  使用できる値は以下の通りです。'AzureCloud'、'AzureChinaCloud'、'AzureUSGovernment'、'AzureGermanCloud'。|
::: zone-end

#### <a name="create-an-adfs-app"></a>ADFS アプリを作成する

1. azurestack\AzureStackAdmin として PowerShell インスタンスを開きます。
1. [前提条件の手順](azure-stack-app-service-before-you-get-started.md)でダウンロードして展開したスクリプトの場所に移動します。
1. [PowerShell for Azure Stack Hub をインストールします](powershell-install-az-module.md)。
1. **Create-ADFSIdentityApp.ps1** スクリプトを実行します。
1. **[資格情報]** ウィンドウで、AD FS クラウドの管理者アカウントとパスワードを入力します。 **[OK]** を選択します。
1. [先ほど作った証明書](azure-stack-app-service-before-you-get-started.md)について、証明書ファイル パスと証明書パスワードを入力します。 既定でこの手順のために作られる証明書は、**sso.appservice.local.azurestack.external.pfx** です。

```powershell
    Create-ADFSIdentityApp.ps1
```

| パラメーター | 必須または省略可能 | 既定値 | 説明 |
| --- | --- | --- | --- |
| AdminArmEndpoint | 必須 | [Null] | 管理者の Azure Resource Manager エンドポイント。 例として、adminmanagement.local.azurestack.external があります。 |
| PrivilegedEndpoint | 必須 | [Null] | 特権エンドポイント。 例として、AzS-ERCS01 があります。 |
| CloudAdminCredential | 必須 | [Null] | Azure Stack Hub クラウド管理者のドメイン アカウントの資格情報。 例として、Azurestack\CloudAdmin があります。 |
| CertificateFilePath | 必須 | [Null] | ID アプリケーションの証明書 PFX ファイルへの **完全なパス**。 |
| CertificatePassword | 必須 | [Null] | 証明書の秘密キーを保護するのに役立つパスワード。 |

<!--Connected/Disconnected-->

### <a name="download-items-from-the-azure-marketplace"></a>Azure Marketplace からの項目のダウンロード

Azure App Service on Azure Stack Hub では、項目を [Azure Marketplace からダウンロード](azure-stack-download-azure-marketplace-item.md)し、それらを Azure Stack Hub Marketplace で使用できるようにする必要があります。 App Service on Azure Stack Hub のデプロイまたはアップグレードを開始する前に、これらの項目をダウンロードする必要があります。

<!-- Connected --->
::: zone pivot="state-connected"
> [!IMPORTANT]
> Windows Server Core は、Azure App Service on Azure Stack Hub で使用するためにサポートされているプラットフォーム イメージではありません。
>
> 運用デプロイには評価版イメージを使用しないでください。
>
1. **Windows Server 2016 Datacenter VM イメージの最新バージョン**。
::: zone-end

::: zone pivot="state-disconnected"
<!-- Disconnected --->
1. **Microsoft. Net 3.5.1 SP1 がアクティブになっている Windows Server 2016 Datacenter の完全 VM イメージ**。  Azure App Service on Azure Stack Hub では、デプロイに使用されるイメージ上で Microsoft .NET 3.5.1 SP1 がアクティブになっている必要があります。 Marketplace にシンジケート化された Windows Server 2016 のイメージでは、この機能が有効になっておらず、切断された環境では Microsoft Update に接続できないため、DISM を介してインストールするパッケージをダウンロードできません。 そのため、切断されたデプロイでは、この機能をあらかじめ有効にして Windows Server 2016 イメージを作成し、使用する必要があります。

   カスタム イメージを作成して Marketplace に追加する方法の詳細については、「[Azure Stack Hub にカスタム VM イメージを追加する](azure-stack-add-vm-image.md)」を参照してください。 Marketplace にイメージを追加するときは、必ず次のプロパティを指定してください。

   - 発行元 = MicrosoftWindowsServer
   - オファー = WindowsServer
   - SKU = 2016-Datacenter
   - バージョン = "最新" バージョンを指定

::: zone-end

<!-- For All --> 
2. **カスタム スクリプト拡張機能 v1.9.1 以上**。 この項目は VM 拡張機能です。

## <a name="next-steps"></a>次のステップ

[App Service リソース プロバイダーのインストール](azure-stack-app-service-deploy.md)
