---
title: Azure Stack Hub でマルチテナントを構成する
description: Azure Stack Hub でゲスト Azure Active Directory テナント用のマルチテナントを構成する方法について説明します。
author: BryanLa
ms.topic: how-to
ms.date: 01/26/2021
ms.author: bryanla
ms.reviewer: bryanr
ms.lastreviewed: 01/26/2021
ms.openlocfilehash: 3de6c5db42285f90e1d4ce6c1ebf6736d7ce4863
ms.sourcegitcommit: d542b68b299b73e045f30916afb6018e365e9db6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2021
ms.locfileid: "99975947"
---
# <a name="configure-multi-tenancy-in-azure-stack-hub"></a>Azure Stack Hub でマルチテナントを構成する

他の Azure Active Directory (Azure AD) ディレクトリにいるユーザーからのサインインをサポートするように Azure Stack Hub を構成して、それらのユーザーが Azure Stack Hub のサービスを使用できるようにすることができます。 これらのディレクトリには、Azure Stack Hub との "ゲスト" 関係があり、そのためゲスト Azure AD テナントと見なされます。 ここでは、次のシナリオを例に説明します。

- あなたは、Azure Stack Hub に ID とアクセス管理のサービスを提供するホーム Azure AD テナントである contoso.onmicrosoft.com のサービス管理者です。
- Mary は、ゲスト ユーザーがいるゲスト Azure AD テナントである fabrikam.onmicrosoft.com のディレクトリ管理者です。
- Mary の会社 (Fabrikam) は、あなたの会社の IaaS と PaaS サービスを使用しています。 Fabrikam は、ゲスト ディレクトリ (fabrikam.onmicrosoft.com) のユーザーが、contoso.onmicrosoft.com によってセキュリティ保護された Azure Stack Hub リソースにサインインしてそれを使用できるようにしたいと考えています。

このガイドでは、ゲスト ディレクトリ テナント用に Azure Stack Hub のマルチテナントを有効または無効にするために、このシナリオのコンテキストで必要となる手順について説明します。 あなたと Mary は、ゲスト ディレクトリ テナントを登録または登録解除することにより、このプロセスを実現します。これにより、Fabrikam ユーザーによる Azure Stack Hub のサインインとサービスの使用が、有効または無効になります。 

クラウド ソリューション プロバイダー (CSP) の場合は、[マルチテナント Azure Stack Hub を構成および管理](azure-stack-add-manage-billing-as-a-csp.md)できる追加の方法があります。 

## <a name="prerequisites"></a>前提条件

ゲスト ディレクトリを登録または登録解除する前に、あなたと Mary は、それぞれの Azure AD テナント (Azure Stack Hub のホーム ディレクトリ (Contoso) と、ゲスト ディレクトリ (Fabrikam)) の管理手順を完了する必要があります。

 - PowerShell for Azure Stack Hub を[インストール](powershell-install-az-module.md)して[構成](azure-stack-powershell-configure-admin.md)します。
 - [Azure Stack Hub のツールをダウンロード](azure-stack-powershell-download.md)して、Connect モジュールと Identity モジュールをインポートします。

    ```powershell
    Import-Module .\Identity\AzureStack.Identity.psm1
    ```

## <a name="register-a-guest-directory"></a>ゲスト ディレクトリを登録する

マルチテナント用にゲスト ディレクトリを登録するには、Azure Stack Hub のホーム ディレクトリとゲスト ディレクトリの両方を構成する必要があります。

#### <a name="configure-azure-stack-hub-directory"></a>Azure Stack Hub ディレクトリの構成

contoso.onmicrosoft.com のサービス管理者であるあなたは、最初に Fabrikam のゲスト ディレクトリ テナントを Azure Stack Hub にオンボードする必要があります。 次のスクリプトを実行すると、fabrikam.onmicrosoft.com テナントのユーザーとサービス プリンシパルからのサインインを受け入れるように、Azure Resource Manager が構成されます。

```powershell  
## The following Azure Resource Manager endpoint is for the ASDK. If you're in a multinode environment, contact your operator or service provider to get the endpoint, formatted as adminmanagement.<region>.<FQDN>.
$adminARMEndpoint = "https://adminmanagement.local.azurestack.external"

## Replace the value below with the Azure Stack Hub directory
$azureStackDirectoryTenant = "contoso.onmicrosoft.com"

## Replace the value below with the guest directory tenant. 
$guestDirectoryTenantToBeOnboarded = "fabrikam.onmicrosoft.com"

## Replace the value below with the name of the resource group in which the directory tenant registration resource should be created (resource group must already exist).
$ResourceGroupName = "system.local"

## Replace the value below with the region location of the resource group.
$location = "local"

# Subscription Name
$SubscriptionName = "Default Provider Subscription"

Register-AzSGuestDirectoryTenant -AdminResourceManagerEndpoint $adminARMEndpoint `
 -DirectoryTenantName $azureStackDirectoryTenant `
 -GuestDirectoryTenantName $guestDirectoryTenantToBeOnboarded `
 -Location $location `
 -ResourceGroupName $ResourceGroupName `
 -SubscriptionName $SubscriptionName
```

#### <a name="configure-guest-directory"></a>ゲスト ディレクトリの構成

次に、Mary (Fabrikam のディレクトリ管理者) は、次のスクリプトを実行して、Azure Stack Hub を fabrikam.onmicrosoft.com ゲスト ディレクトリに登録する必要があります。

```powershell
## The following Azure Resource Manager endpoint is for the ASDK. If you're in a multinode environment, contact your operator or service provider to get the endpoint, formatted as management.<region>.<FQDN>.
$tenantARMEndpoint = "https://management.local.azurestack.external"
    
## Replace the value below with the guest directory tenant.
$guestDirectoryTenantName = "fabrikam.onmicrosoft.com"

Register-AzSWithMyDirectoryTenant `
 -TenantResourceManagerEndpoint $tenantARMEndpoint `
 -DirectoryTenantName $guestDirectoryTenantName `
 -Verbose
```

> [!IMPORTANT]
> Azure Stack Hub 管理者が今後新しいサービスや更新プログラムをインストールした場合には、このスクリプトをもう一度実行する必要があります。
>
> このスクリプトを再実行すると、お使いのディレクトリ内の Azure Stack Hub アプリの状態をいつでも確認できます。
>
> VM をマネージド ディスク内に作成する (1808 更新プログラムで導入) 際に問題が起きた場合には、このスクリプトの再実行を求める新しい **ディスク リソース プロバイダー** が追加されました。

### <a name="direct-users-to-sign-in"></a>ユーザーをサインインに誘導する

最後に、Mary は、@fabrikam.onmicrosoft.com アカウントを使用する Fabrikam のユーザーに、[Azure Stack Hub ユーザー ポータル](../user/azure-stack-use-portal.md)にアクセスすることによってサインインするよう指示することができます。 マルチノード システムの場合、ユーザー ポータルの URL は `https://management.<region>.<FQDN>` という形式になります。 ASDK のデプロイの場合、URL は `https://portal.local.azurestack.external` です。

Mary は、外部のプリンシパル (fabrikam.onmicrosoft.com というサフィックスを使用していない Fabrikam ディレクトリのユーザー) にも、`https://<user-portal-url>/fabrikam.onmicrosoft.com` を使用してサインインするよう指示する必要があります。 URL で `/fabrikam.onmicrosoft.com` ディレクトリ テナントを指定しない場合、これらのユーザーは自分の既定のディレクトリに送られ、管理者が同意していないことを示すエラーを受け取ります。

## <a name="unregister-a-guest-directory"></a>ゲスト ディレクトリを登録解除する

ゲスト ディレクトリ テナントからの Azure Stack Hub サービスへのサインインを許可する必要がなくなった場合は、ディレクトリの登録を解除できます。 ここでも、Azure Stack Hub のホーム ディレクトリとゲスト ディレクトリの両方を構成する必要があります。

1. ゲスト ディレクトリの管理者 (このシナリオでは Mary) として、`Unregister-AzsWithMyDirectoryTenant` を実行します。 このコマンドレットは、新しいディレクトリからすべての Azure Stack Hub アプリをアンインストールします。

    ``` PowerShell
    ## The following Azure Resource Manager endpoint is for the ASDK. If you're in a multinode environment, contact your operator or service provider to get the endpoint, formatted as management.<region>.<FQDN>.
    $tenantARMEndpoint = "https://management.local.azurestack.external"
        
    ## Replace the value below with the guest directory tenant.
    $guestDirectoryTenantName = "fabrikam.onmicrosoft.com"
    
    Unregister-AzsWithMyDirectoryTenant `
     -TenantResourceManagerEndpoint $tenantARMEndpoint `
     -DirectoryTenantName $guestDirectoryTenantName `
     -Verbose 
    ```

2. Azure Stack Hub のサービス管理者として (このシナリオではあなた)、`Unregister-AzSGuestDirectoryTenant` コマンドレットを実行します。

    ``` PowerShell
    ## The following Azure Resource Manager endpoint is for the ASDK. If you're in a multinode environment, contact your operator or service provider to get the endpoint, formatted as adminmanagement.<region>.<FQDN>.
    $adminARMEndpoint = "https://adminmanagement.local.azurestack.external"
    
    ## Replace the value below with the Azure Stack Hub directory
    $azureStackDirectoryTenant = "contoso.onmicrosoft.com"
    
    ## Replace the value below with the guest directory tenant. 
    $guestDirectoryTenantToBeDecommissioned = "fabrikam.onmicrosoft.com"
    
    ## Replace the value below with the name of the resource group in which the directory tenant resource was created (resource group must already exist).
    $ResourceGroupName = "system.local"
    
    Unregister-AzSGuestDirectoryTenant -AdminResourceManagerEndpoint $adminARMEndpoint `
     -DirectoryTenantName $azureStackDirectoryTenant `
     -GuestDirectoryTenantName $guestDirectoryTenantToBeDecommissioned `
     -ResourceGroupName $ResourceGroupName
    ```

    > [!WARNING]
    > マルチ テナントを無効にする手順は、正しい順序で実行する必要があります。 手順 2 を先に実行した場合、手順 1 は失敗します。

## <a name="retrieve-azure-stack-hub-identity-health-report"></a>Azure Stack Hub の ID 正常性レポートの取得 

プレースホルダーの `<region>`、`<domain>`、`<homeDirectoryTenant>` を置き換えてから、次のコマンドレットを Azure Stack Hub 管理者として実行します。

```powershell

$AdminResourceManagerEndpoint = "https://adminmanagement.<region>.<domain>"
$DirectoryName = "<homeDirectoryTenant>.onmicrosoft.com"
$healthReport = Get-AzsHealthReport -AdminResourceManagerEndpoint $AdminResourceManagerEndpoint -DirectoryTenantName $DirectoryName
Write-Host "Healthy directories: "
$healthReport.directoryTenants | Where status -EQ 'Healthy' | Select -Property tenantName,tenantId,status | ft


Write-Host "Unhealthy directories: "
$healthReport.directoryTenants | Where status -NE 'Healthy' | Select -Property tenantName,tenantId,status | ft
```

## <a name="update-azure-ad-tenant-permissions"></a>Azure AD テナントのアクセス許可の更新

このアクションにより、ディレクトリの更新が必要であることを示す Azure Stack Hub のアラートがクリアされます。 **Azurestack-tools-master/identity** フォルダーから次のコマンドを実行します。

```powershell
Import-Module ..\Identity\AzureStack.Identity.psm1

$adminResourceManagerEndpoint = "https://adminmanagement.<region>.<domain>"

# This is the primary tenant Azure Stack Hub is registered to:
$homeDirectoryTenantName = "<homeDirectoryTenant>.onmicrosoft.com"

Update-AzsHomeDirectoryTenant -AdminResourceManagerEndpoint $adminResourceManagerEndpoint `
   -DirectoryTenantName $homeDirectoryTenantName -Verbose
```

このスクリプトにより、Azure AD テナントの管理者資格情報の入力が求められます。実行には数分かかります。 このコマンドレットを実行すると、アラートがクリアされます。

## <a name="next-steps"></a>次のステップ

- [委任されたプロバイダーの管理](azure-stack-delegated-provider.md)
- [Azure Stack Hub の主要概念](azure-stack-overview.md)
- [クラウド ソリューション プロバイダーとして Azure Stack Hub の使用状況と課金を管理する](azure-stack-add-manage-billing-as-a-csp.md)
- [Azure Stack Hub に使用量と課金用のテナントを追加する](azure-stack-csp-howto-register-tenants.md)
