---
title: Windows Admin Center を Azure に登録する
description: Windows Admin Center ゲートウェイを Azure に登録する方法。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.date: 02/10/2021
ms.openlocfilehash: 0b80a1e607823385d06a5255244373ca3be1af98
ms.sourcegitcommit: 5ea0e915f24c8bcddbcaf8268e3c963aa8877c9d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2021
ms.locfileid: "100487886"
---
# <a name="register-windows-admin-center-with-azure"></a>Windows Admin Center を Azure に登録する

> 適用対象: Azure Stack HCI v20H2、Windows Server 2019

Windows Admin Center で Azure サービスを使用するには、まず管理 PC に [Windows Admin Center をインストール](/windows-server/manage/windows-admin-center/deploy/install)し、Windows Admin Center ゲートウェイの 1 回限りの登録を完了する必要があります。 これは、Azure に[クラスターを登録](../deploy/register-with-azure.md)するための前提条件です。

   > [!IMPORTANT]
   > 同じ Azure Active Directory (テナント) ID を使用して、クラスターの登録に使用する予定の同じ管理 PC に Windows 管理センターを登録します。

## <a name="complete-the-registration-process"></a>登録プロセスを完了する

1. Windows Admin Center を起動し、右上にある **[設定]** 歯車アイコンをクリックします。これにより、[アカウント] ページに移動します。 次に、左側の **[ゲートウェイ]** メニューで **[Azure]** を選択し、 **[登録]** をクリックします。

   :::image type="content" source="media/register-wac/register-wac.png" alt-text="[設定] > [ゲートウェイ] > [Azure] を選択した後、[登録] をクリックする" lightbox="media/register-wac/register-wac.png":::

2. 一意のコードが表示されます。 コードの右にある **[コピー]** ボタンをクリックします。 **[コードの入力]** をクリックすると、別のブラウザー ウィンドウが開き、アプリまたはデバイスに表示されているコードを貼り付けることができます。

   :::image type="content" source="media/register-wac/enter-code.png" alt-text="一意のコードをコピーし、[コードの入力] をクリックして、ダイアログ ボックスに貼り付ける" lightbox="media/register-wac/enter-code.png":::

3. コードを貼り付けると、リモート デバイスまたはサービスで Windows Admin Center にサインインしようとしていることを示す通知が表示されます。 メール アドレスまたは電話番号を入力します。 デバイスが管理されている場合は、認証のために組織のサインイン ページが表示されます。 指示に従って、適切な資格情報を入力します。

   :::image type="content" source="media/register-wac/sign-in.png" alt-text="メール アドレスまたは電話番号を使用して Windows Admin Center にサインインする" lightbox="media/register-wac/sign-in.png":::

   次のメッセージが表示されます。"デバイスの Windows Admin Center アプリケーションにサインインしました。" ブラウザー ウィンドウを閉じて、元の登録ページに戻ります。

4. Azure Active Directory (テナント) ID を入力して、Azure Active Directory に接続します。 既に Azure テナント ID を持っていて、前の手順に従っている場合、ID フィールドは事前に設定されている可能性があります。 組織から既存の ID を提供されていない場合は、 **[Azure Active Directory アプリケーション]** の設定を **[新規作成]** にままにしておきます。 ID を既に持っている場合は、 **[既存のものを使用]** をクリックすると、管理者によって提供された ID を入力するための空のフィールドが表示されます。ID を入力すると、その ID のアカウントが見つかることが Windows Admin Center によって確認されます。 既存の ID はあるものの、それが何かわからない場合は、[こちらの手順](/azure/active-directory/develop/howto-create-service-principal-portal#get-values-for-signing-in)に従って取得します。

   :::image type="content" source="media/register-wac/connect-to-aad.png" alt-text="既存の Azure Active Directory (テナント) ID を入力するか、新しいものを作成して、Azure Active Directory に接続する" lightbox="media/register-wac/connect-to-aad.png":::

5. **[接続]** ボタンをクリックして Azure に接続します。 Azure AD に接続されているという確認が表示されます。

6. Azure portal で **[アプリのアクセス許可]** に移動して、Azure でのアクセス許可を付与します。 **[同意する]** で、 **[管理者の同意を与えます]** を選択します。

7. ウィンドウを閉じ、Azure アカウントを使用して Windows Admin Center にサインインします。

## <a name="next-steps"></a>次のステップ

これで以下の準備が整いました。

- [Azure Stack HCI を Azure に接続する](../deploy/register-with-azure.md)
- [Windows Admin Center で Azure Stack HCI を使用する](../get-started.md)
- [Azure ハイブリッド サービスに接続する](/windows-server/manage/windows-admin-center/azure/)