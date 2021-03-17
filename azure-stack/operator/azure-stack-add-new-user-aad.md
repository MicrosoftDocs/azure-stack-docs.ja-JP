---
title: 新しい Azure Stack Hub ユーザー アカウントを Azure Active Directory に追加する
description: ユーザー ポータルを調べることができるように、Azure Active Directory でユーザー アカウントを作成する方法について説明します。
author: BryanLa
ms.topic: article
ms.date: 05/20/2019
ms.author: bryanla
ms.reviewer: thoroet
ms.lastreviewed: 09/17/2019
ms.openlocfilehash: 57b79c62de98380c97050f6c484cd5ba1b6323fd
ms.sourcegitcommit: ccc4ee05d71496653b6e27de1bb12e4347e20ba4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102231372"
---
# <a name="add-a-new-azure-stack-hub-user-account-in-azure-active-directory-azure-ad"></a>新しい Azure Stack Hub ユーザー アカウントを Azure Active Directory に追加する (Azure AD)

オファーやプランのテスト、およびリソースの作成を行う前に、ユーザーアカウントが必要です。 Azure portal または PowerShell を使用して、Azure AD テナントにユーザー アカウントを作成します。

## <a name="create-user-account-using-the-azure-portal"></a>Azure portal を使用してユーザー アカウントを作成する

Azure ポータルを使用するには Azure サブスクリプションが必要です。

1. [Azure](https://portal.azure.com) にサインインします。
2. 左側のナビゲーション バーで **[Active Directory]** を選択し、Azure Stack Hub で使用するディレクトリに切り替えます (または新しいディレクトリを作成します)。
3. **[Azure Active Directory]**  >  **[ユーザー]**  >  **[新しいユーザー]** を選択します。

    ![[ユーザー] - [すべてのユーザー] ページ ([新しいユーザー] が強調表示されている)](media/azure-stack-add-new-user-aad/new-user-all-users.png)

4. **[ユーザー]** ページで、必要な情報を入力します。

    ![新しいユーザーの追加 (ユーザー情報が入力された [ユーザー] ページ)](media/azure-stack-add-new-user-aad/new-user-user.png)

   - **名前 (必須)** : 新しいユーザーの姓と名。 たとえば、Mary Parker などです。
   - **ユーザー名 (必須)** : 新しいユーザーのユーザー名。 たとえば、「 mary@contoso.com 」のように入力します。
       ユーザー名のドメイン部分には、既定の初期ドメイン名の <_yourdomainname_>.onmicrosoft.com、またはカスタム ドメイン名 (contoso.com など) のいずれかを使用する必要があります。 カスタム ドメイン名の作成方法の詳細については、[Azure Active Directory にカスタム ドメイン名を追加する方法](/azure/active-directory/fundamentals/add-custom-domain)に関するページを参照してください。
   - **プロファイル**: (省略可能) ユーザーに関する詳細情報を追加することができます。 後でユーザー情報を追加することもできます。 ユーザー情報の追加方法の詳細については、[ユーザー プロファイル情報を追加または変更する方法](/azure/active-directory/fundamentals/active-directory-users-profile-azure-portal)に関するページを参照してください。
   - **ディレクトリ ロール**: **[ユーザー]** を選択します。

5. **[パスワードの表示]** チェックボックスを選択し、 **[パスワード]** ボックスに表示される自動生成されたパスワードをコピーします。 このパスワードは、最初のサインイン プロセスで必要になります。

6. **［作成］** を選択します

    ユーザーが作成され、Azure AD テナントに追加されます。

7. 新しいアカウントで Azure portal にサインインします。 パスワードの変更を求められたら、変更します。
8. `https://portal.local.azurestack.external` に新しいアカウントでサインインして、ユーザー ポータルを表示します。

## <a name="create-a-user-account-using-powershell"></a>PowerShell を使用してユーザー アカウントを作成する

Azure サブスクリプションがない場合は、Azure Portal を使用してテナント ユーザー アカウントを追加できません。 この場合は、代わりに Windows PowerShell 用 Azure AD モジュールを利用できます。

> [!NOTE]
> Microsoft アカウントを使用して ASDK をデプロイしている場合は、Azure AD PowerShell を使用してテナント アカウントを作成できません。

1. **64 ビット** バージョンの [IT プロフェッショナル用 Microsoft Online Services サインイン アシスタント RTW](https://www.microsoft.com/download/details.aspx?id=28177) をインストールします。

2. 以下の手順で Windows PowerShell 用 Microsoft Azure Active Directory モジュールをインストールします。

    - 管理者特権での Windows PowerShell コマンド プロンプトを開きます (管理者として Windows PowerShell を実行します)。
    - **Install-Module MSOnline** コマンドを実行します。
    - NuGet プロバイダーをインストールするように求められたら、**Y** キーと **Enter** キーを押します。
    - PSGallery からモジュールをインストールするように求められたら、**Y** キーと **Enter** キーを押します。

3. 次のコマンドレットを実行します。

    ```powershell
    # Provide the Azure AD credential you use to deploy the ASDK.

            $msolcred = get-credential

    # Add a user account "Tenant Admin <username>@<yourdomainname>" with the initial password "<password>".

            connect-msolservice -credential $msolcred
            $user = new-msoluser -DisplayName "Tenant Admin" -UserPrincipalName <username>@<yourdomainname> -Password <password>
            Add-MsolRoleMember -RoleName "Company Administrator" -RoleMemberType User -RoleMemberObjectId $user.ObjectId

    ```

1. 新しいアカウントで Azure にサインインします。 パスワードの変更を求められたら、変更します。
2. `https://portal.local.azurestack.external` に新しいアカウントでサインインして、ユーザー ポータルを表示します。

## <a name="next-steps"></a>次のステップ

[AD FS に Azure Stack Hub ユーザーを追加する](azure-stack-add-users-adfs.md)
