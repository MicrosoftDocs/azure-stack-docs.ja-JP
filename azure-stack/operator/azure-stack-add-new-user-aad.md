---
title: 新しい Azure Stack テナント アカウントを Azure Active Directory に追加する | Microsoft Docs
description: Microsoft Azure Stack Development Kit のデプロイを行った後、少なくとも 1 つのテナント ユーザー アカウントを作成して、テナント ポータルを表示できるようにする必要があります。
services: azure-stack
documentationcenter: ''
author: PatAltimore
manager: femila
editor: ''
ms.assetid: a75d5c88-5b9e-4e9a-a6e3-48bbfa7069a7
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/20/2019
ms.author: patricka
ms.reviewer: thoroet
ms.lastreviewed: 09/17/2018
ms.openlocfilehash: 70151d7793ef1f58b544517cecb7aa53bf5b3041
ms.sourcegitcommit: 7f39bdc83717c27de54fe67eb23eb55dbab258a9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/05/2019
ms.locfileid: "66691403"
---
# <a name="add-a-new-azure-stack-tenant-account-in-azure-active-directory"></a>新しい Azure Stack テナント アカウントをAzure Active Directory に追加する

[Azure Stack Development Kit をデプロイ](../asdk/asdk-install.md)した後は、テナント ポータルを操作し、オファーとプランをテストするためにテナント ユーザー アカウントが必要になります。 テナント アカウントを作成するには、[Azure portal](#create-an-azure-stack-tenant-account-using-the-azure-portal) または PowerShell を使用します。

## <a name="create-an-azure-stack-tenant-account-using-the-azure-portal"></a>Azure ポータルを使用して Azure Stack テナント アカウントを作成する

Azure ポータルを使用するには Azure サブスクリプションが必要です。

1. [Azure](https://portal.azure.com) にサインインします。
2. 左側のナビゲーション バーで **Active Directory** を選択し、Azure Stack で使用するディレクトリに切り替えるか、新しいディレクトリを作成します。
3. **[Azure Active Directory]**  >  **[ユーザー]**  >  **[新しいユーザー]** を選択します。

    ![[ユーザー] - [すべてのユーザー] ページ ([新しいユーザー] が強調表示されている)](media/azure-stack-add-new-user-aad/new-user-all-users.png)

4. **[ユーザー]** ページで、必要な情報を入力します。

    ![新しいユーザーの追加 (ユーザー情報が入力された [ユーザー] ページ)](media/azure-stack-add-new-user-aad/new-user-user.png)

   - **[名前] (必須)。** 新しいユーザーの氏名です。 たとえば、Mary Parker などです。
   - **[ユーザー名] (必須)。** 新しいユーザーのユーザー名です。 たとえば、「 mary@contoso.com 」のように入力します。
       ユーザー名のドメイン部分には、既定の初期ドメイン名の <_yourdomainname_>.onmicrosoft.com、またはカスタム ドメイン名 (contoso.com など) のいずれかを使用する必要があります。 カスタム ドメイン名の作成方法の詳細については、[Azure Active Directory にカスタム ドメイン名を追加する方法](/azure/active-directory/fundamentals/add-custom-domain)に関するページを参照してください。
   - **[プロファイル]。** オプションで、ユーザーに関する詳細情報を追加することができます。 後でユーザー情報を追加することもできます。 ユーザー情報の追加方法については、「[ユーザー プロファイル情報の追加または変更方法](/azure/active-directory/fundamentals/active-directory-users-profile-azure-portal)」を参照してください。
   - **ディレクトリ ロール。**  **[ユーザー]** を選択します。

5. **[パスワードの表示]** チェックボックスを選択し、 **[パスワード]** ボックスに表示される自動生成されたパスワードをコピーします。 このパスワードは、最初のサインイン プロセスで必要になります。

6. **作成** を選択します。

    ユーザーが作成され、Azure AD テナントに追加されます。

7. 新しいアカウントで Microsoft Azure portal にサインインします。 パスワードの変更を求められたら、変更します。
8. `https://portal.local.azurestack.external` に新しいアカウントでサインインして、テナント ポータルを表示します。

## <a name="create-an-azure-stack-user-account-using-powershell"></a>PowerShell を使用して Azure Stack ユーザー アカウントを作成する

Azure サブスクリプションがない場合は、Azure Portal を使用してテナント ユーザー アカウントを追加できません。 その場合は、代わりに Windows PowerShell 用 Azure Active Directory モジュールを利用できます。

> [!NOTE]
> Microsoft アカウントを使用して Azure Stack Development Kit をデプロイしている場合は、Azure AD PowerShell を使用してテナント アカウントを作成できません。 

1. **64 ビット** バージョンの [IT プロフェッショナル用 Microsoft Online Services サインイン アシスタント RTW](https://go.microsoft.com/fwlink/p/?LinkId=286152) をインストールします。

2. 以下の手順で Windows PowerShell 用 Microsoft Azure Active Directory モジュールをインストールします。

    - 管理者特権での Windows PowerShell コマンド プロンプトを開きます (管理者として Windows PowerShell を実行します)。
    - **Install-Module MSOnline** コマンドを実行します。
    - NuGet プロバイダーをインストールするように求められたら、**Y** キーと **Enter** キーを押します。
    - PSGallery からモジュールをインストールするように求められたら、**Y** キーと **Enter** キーを押します。

3. 次のコマンドレットを実行します。

    ```powershell
    # Provide the AAD credential you use to deploy Azure Stack Development Kit

            $msolcred = get-credential

    # Add a tenant account "Tenant Admin <username>@<yourdomainname>" with the initial password "<password>".

            connect-msolservice -credential $msolcred
            $user = new-msoluser -DisplayName "Tenant Admin" -UserPrincipalName <username>@<yourdomainname> -Password <password>
            Add-MsolRoleMember -RoleName "Company Administrator" -RoleMemberType User -RoleMemberObjectId $user.ObjectId

    ```

1. 新しいアカウントで Microsoft Azure にサインインします。 パスワードの変更を求められたら、変更します。
2. `https://portal.local.azurestack.external` に新しいアカウントでサインインして、テナント ポータルを表示します。

## <a name="next-steps"></a>次の手順

[AD FS の Azure Stack ユーザーを追加する](azure-stack-add-users-adfs.md)
