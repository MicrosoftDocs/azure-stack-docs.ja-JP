---
title: Windows Admin Center を Azure に登録する
description: Windows Admin Center ゲートウェイを Azure に登録する方法。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.date: 03/10/2021
ms.openlocfilehash: 79b90f028c61319535a7f5ddacdbd7cb1cf25081
ms.sourcegitcommit: e7d6f953e7014900b4e7d710340cfa98d253fce9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/11/2021
ms.locfileid: "102637631"
---
# <a name="register-windows-admin-center-with-azure"></a>Windows Admin Center を Azure に登録する

> 適用対象: Azure Stack HCI バージョン 20H2、Windows Server 2019

Windows Admin Center で Azure サービスを使用するには、まず管理 PC に [Windows Admin Center をインストール](/windows-server/manage/windows-admin-center/deploy/install)し、ご利用の Windows Admin Center ゲートウェイを登録します。 これは、Azure に[クラスターを登録](../deploy/register-with-azure.md)するための前提条件です。

   > [!IMPORTANT]
   > Windows Admin Center を使用して Azure Stack HCI クラスターを登録するためには、所属する組織の Azure AD 管理者によって承認された Azure Active Directory (Azure AD) アプリケーション ID にご利用の Windows Admin Center ゲートウェイを登録する必要があります。同じ Azure Active Directory (テナント) ID を使用して、クラスターの登録に使用する予定の同じ管理 PC に Windows Admin Center を登録します。

## <a name="complete-the-registration-process"></a>登録プロセスを完了する

1. Windows Admin Center を起動し、右上にある **[設定]** 歯車アイコンを選択します。これにより、自分の [アカウント] ページに移動します。 次に、左側の **[ゲートウェイ]** メニューで **[Azure]** を選択し、 **[登録]** をクリックします。

   :::image type="content" source="media/register-wac/register-wac.png" alt-text="[設定] > [ゲートウェイ] > [Azure] を選択した後、[登録] をクリックする" lightbox="media/register-wac/register-wac.png":::

2. 一意のコードが表示されます。 コードの右にある **[コピー]** ボタンをクリックします。 **[コードの入力]** をクリックすると、別のブラウザー ウィンドウが開き、アプリまたはデバイスに表示されているコードを貼り付けることができます。

   :::image type="content" source="media/register-wac/enter-code.png" alt-text="一意のコードをコピーし、[コードの入力] をクリックして、ダイアログ ボックスに貼り付ける" lightbox="media/register-wac/enter-code.png":::

3. コードを貼り付けると、リモート デバイスまたはサービスで Windows Admin Center にサインインしようとしていることを示す通知が表示されます。 メール アドレスまたは電話番号を入力します。 デバイスが管理されている場合は、認証のために組織のサインイン ページが表示されます。 指示に従って、適切な資格情報を入力します。

   :::image type="content" source="media/register-wac/sign-in.png" alt-text="メール アドレスまたは電話番号を使用して Windows Admin Center にサインインする" lightbox="media/register-wac/sign-in.png":::

   次のメッセージが表示されます。"デバイスの Windows Admin Center アプリケーションにサインインしました。" ブラウザー ウィンドウを閉じて、元の登録ページに戻ります。

4. Azure Active Directory (テナント) ID とアプリケーション ID を入力して、Azure Active Directory に接続します。 既に Azure テナント ID を持っていて、前の手順を完了している場合、テナント ID フィールドは事前に設定されている可能性があり、複数のオプションが含まれる可能性があります。 正しいテナント ID を選択してください。 

   Azure AD 管理者からアプリケーション ID が提供されている場合は、 **[既存のものを使用]** をクリックします。空のフィールドが表示されるので、管理者から提供されている ID を入力します。ID を入力すると、その ID のアカウントが見つかることが Windows Admin Center によって確認されます。 既存の ID はあるものの、それが何かわからない場合は、[こちらの手順](/azure/active-directory/develop/howto-create-service-principal-portal#get-values-for-signing-in)に従って取得します。 組織から既存の ID が提供されていない場合は、 **[Azure Active Directory アプリケーション]** の設定を **[新規作成]** のままにしておきます。

   :::image type="content" source="media/register-wac/connect-to-aad.png" alt-text="既存の Azure Active Directory (テナント) ID とアプリケーション ID を入力するか、新しいアプリケーション ID を作成して、Azure Active Directory に接続する" lightbox="media/register-wac/connect-to-aad.png":::

5. **[Connect]** をクリックします。 自分が Azure AD 管理者である場合、または既存のアプリケーション ID を使用した場合は、現在 Azure AD に接続されていることを示す確認メッセージが表示されます。 それ以外の場合は、管理者の承認を必要とすることを示すメッセージが表示されることがあります。 これに該当する場合は、 **[同意せずにアプリケーションに戻る]** を選択し、次の手順 6 の説明に従って、登録時に作成された新しいアプリケーション ID に同意するように Azure AD 管理者に依頼します。

6. 自分が Azure AD 管理者である場合は、 **[Azure Active Directory]** 、 **[アプリの登録]** の順に移動して、Azure でのアクセス許可を付与します。 **[すべてのアプリケーション]** を選択し、**WindowsAdminCenter** を検索します。 登録するゲートウェイの表示名を選択します。 ページの上部付近に表示される **[アプリケーション (クライアント) ID]** をメモしておきます。それをユーザーに提供することが必要になるためです。 次に、 **[API のアクセス許可]** に移動します。 **[同意する]** で、 **[管理者の同意を与えます]** を選択します。 次に、ユーザーにアプリケーション ID を付与します。 複数のユーザーに同じアプリケーション ID を使用する予定である場合は、手順 7 に進みます。それ以外の場合は、手順 8 にスキップします。

7. 管理を便利かつ容易にするために、組織内の複数のユーザーが同じ Azure アプリ ID を使用して Windows Admin Center を登録できるようにすることができます。 これを行うには、すべてのユーザーが同じドメインおよびポート ( *https://localhost:6516* など) に自分のゲートウェイを登録する必要があります。 また、Azure AD 管理者には、追加の手順を行って、Azure portal にリダイレクト URI を追加することが求められます。

   Windows Admin Center で、右上にある **[設定]** 歯車アイコンを選択します。 次に、左側の **[ゲートウェイ]** メニューで **[Azure]** を選択してから、 **[View in Azure]\(Azure で表示\)** をクリックします。これで Azure AD の詳細が表示されます。 Azure portal で、左側のメニューから **[管理] > [認証]** の順に選択します。 **[リダイレクト URI]** ボックスには、アプリ ID に登録されている最初のゲートウェイを表す既存の URI が表示されます。 **[URI の追加]** を選択し、新しいリダイレクト URI を 2 つ追加します ( *http://localhost:6516* や *https://localhost:6516* など)。

   :::image type="content" source="media/register-wac/add-redirect-uris.png" alt-text="組織内の複数のユーザーが同じ Azure アプリ ID を使用して Windows Admin Center を登録できるようにするには、リダイレクト URI を追加します。" lightbox="media/register-wac/add-redirect-uris.png":::

   > [!IMPORTANT]
   > Azure AD 管理者によってリダイレクト URI が追加されていない場合に、複数のユーザーが同じアプリ ID に対して Windows Admin Center を登録しようとすると、応答 URL が一致しないというエラーがユーザーに表示されます。

7. この時点で、ユーザーは登録ウィザードを再実行する必要があります。今回、アプリケーション ID については **[既存のものを使用]** を選択し、Azure AD 管理者によって提供されたアプリケーション ID を指定します。

8. **[サインイン]** を選択し、自分の Azure アカウントを使用して Windows Admin Center にサインインします。

## <a name="unregister-windows-admin-center"></a>Windows Admin Center の登録を解除する

ご利用の Windows Admin Center ゲートウェイの登録を解除するには、右上にある **[設定]** 歯車アイコンを選択します。 次に、左側の **[ゲートウェイ]** メニューで、 **[Azure]** 、 **[登録解除]** 、 **[確認]** の順に選択します。 

これにより、Windows Admin Center と、Azure AD 管理者によって提供された、またはゲートウェイの登録時に作成された Azure AD アプリ ID との関連付けが削除されます。 これによって、Azure AD アプリが削除されることも、サーバーによって使用されている Azure サービスまたは Windows Admin Center によって管理されているクラスターが影響を受けることもありません。

## <a name="next-steps"></a>次のステップ

これで以下の準備が整いました。

- [Azure Stack HCI を Azure に接続する](../deploy/register-with-azure.md)
- [Windows Admin Center で Azure Stack HCI を使用する](../get-started.md)
- [Azure ハイブリッド サービスに接続する](/windows-server/manage/windows-admin-center/azure/)