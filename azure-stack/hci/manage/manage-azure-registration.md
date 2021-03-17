---
title: Azure での Azure Stack HCI クラスター登録の管理
description: Azure Stack HCI クラスターの Azure 登録を管理し、登録状態を把握して、使用停止の準備ができたときにクラスターの登録を解除する方法。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.date: 02/10/2021
ms.openlocfilehash: 101d1cbcab097803e9c8a9fd9cd651dcf6fc80d1
ms.sourcegitcommit: e432e7f0a790bd6419987cbb5c5f3811e2e7a4a2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/09/2021
ms.locfileid: "102515636"
---
# <a name="manage-cluster-registration-with-azure"></a>Azure でのクラスター登録の管理

> 適用対象: Azure Stack HCI バージョン 20H2

Azure Stack HCI クラスター作成後、[Windows Admin Center を Azure に登録](register-windows-admin-center.md)してから、[そのクラスターを Azure に登録](../deploy/register-with-azure.md)する必要があります。 クラスター登録後は、オンプレミス クラスターとクラウドの間で定期的に情報が同期されます。 

この記事では、登録状態を表示し、Azure Active Directory (Azure AD) のアクセス許可を付与し、使用停止の準備ができたらクラスターの登録を解除する方法について説明します。

## <a name="view-registration-status-in-windows-admin-center"></a>Windows Admin Center で登録状態を表示する

Windows Admin Center を使用してクラスターに接続すると、ダッシュボードが表示され、Azure の接続状態が表示されます。 "**接続済み**" とは、クラスターが既に Azure に登録され、過去 1 日以内にクラウドと正常に同期されたことを意味します。

:::image type="content" source="media/manage-azure-registration/registration-status.png" alt-text="クラスター接続状態を示す Windows Admin Center ダッシュボードのスクリーンショット。" lightbox="media/manage-azure-registration/registration-status.png":::

詳細情報を表示するには、左側の **[ツール]** メニューの下部にある **[設定]** を選択し、 **[Azure Stack HCI registration]\(Azure Stack HCI の登録\)** を選択します。

:::image type="content" source="media/manage-azure-registration/azure-stack-hci-registration.png" alt-text="Azure Stack H C I の登録情報を取得するための選択内容を示すスクリーンショット。" lightbox="media/manage-azure-registration/azure-stack-hci-registration.png":::

## <a name="view-registration-status-in-powershell"></a>PowerShell で登録状態を表示する

Windows PowerShell を使用して登録状態を表示するには、`Get-AzureStackHCI` PowerShell コマンドレットと、`ClusterStatus`、`RegistrationStatus`、および `ConnectionStatus` の各プロパティを使用します。 

たとえば、Azure Stack HCI オペレーティング システムをインストールした後、クラスターの作成または参加を行う前は、`ClusterStatus` プロパティには `NotYet` 状態が表示されます。

:::image type="content" source="media/manage-azure-registration/1-get-azurestackhci.png" alt-text="クラスター作成前の Azure の登録状態を示すスクリーンショット。":::

クラスター作成後は、`RegistrationStatus` にのみ `NotYet` の状態が表示されます。

:::image type="content" source="media/manage-azure-registration/2-get-azurestackhci.png" alt-text="クラスター作成後の Azure の登録状態を示すスクリーンショット。":::

Azure オンライン サービス条件に定められているように、Azure Stack HCI クラスターはインストール後 30 日以内に登録する必要があります。 30 日後にクラスターの作成または参加が済んでいない場合は、`ClusterStatus` は `OutOfPolicy` と表示されます。 30 日後にクラスターを登録していない場合、`RegistrationStatus` は `OutOfPolicy` と表示されます。

クラスターが登録されると、`ConnectionStatus` と `LastConnected` の日時が表示されます。 クラスターがインターネットから一時的に切断されていなければ、通常、`LastConnected` の日時は最新の日付です。 Azure Stack HCI クラスターは、最大 30 日間、完全にオフラインで動作できます。

:::image type="content" source="media/manage-azure-registration/3-get-azurestackhci.png" alt-text="登録後の Azure 登録状態を示すスクリーンショット。":::

オフライン操作の最大期間を超えると、`ConnectionStatus` は `OutOfPolicy` と表示されます。

## <a name="assign-azure-ad-app-permissions"></a>Azure AD アプリのアクセス許可を割り当てる

サブスクリプションで Azure リソースを作成することに加えて Azure Stack HCI を登録すると、アプリ ID が Azure AD テナント内に作成されます。 この ID は、概念的にはユーザーに似ています。 アプリ ID にはクラスター名が継承されます。 この ID は、サブスクリプション内で、Azure Stack HCI クラウド サービスに代わって適切に機能します。

クラスターを登録するユーザーが Azure Active Directory 管理者であるか、十分なアクセス許可を持っている場合、これはすべて自動的に行われます。 それ以上何もする必要はありません。 それ以外の場合は、登録を完了するために Azure AD 管理者からの承認が必要になる場合があります。 管理者は、アプリに明示的に同意するか、アプリに同意できるアクセス許可を委任することができます。

:::image type="content" source="media/manage-azure-registration/aad-permissions.png" alt-text="Azure Active Directory のアクセス許可と ID を示す図。" border="false":::

同意するには、[portal.azure.com](https://portal.azure.com) を開き、Azure AD で十分なアクセス許可を持つ Azure アカウントを使用してサインインします。 **[Azure Active Directory]**  >  **[アプリの登録]** に移動します。 クラスターに基づいて名付けられたアプリ ID を選択し、 **[API のアクセス許可]** に移動します。

Azure Stack HCI の一般提供 (GA) リリースの場合、アプリには次のアクセス許可が必要です。 パブリック プレビューの場合に必要なアプリのアクセス許可とは異なります。

```http
https://azurestackhci-usage.trafficmanager.net/AzureStackHCI.Cluster.Read

https://azurestackhci-usage.trafficmanager.net/AzureStackHCI.Cluster.ReadWrite

https://azurestackhci-usage.trafficmanager.net/AzureStackHCI.ClusterNode.Read

https://azurestackhci-usage.trafficmanager.net/AzureStackHCI.ClusterNode.ReadWrite
```

パブリック プレビューの場合、アプリのアクセス許可 (現在は非推奨) は次のとおりでした。

```http
https://azurestackhci-usage.trafficmanager.net/AzureStackHCI.Census.Sync

https://azurestackhci-usage.trafficmanager.net/AzureStackHCI.Billing.Sync
```

Azure AD 管理者から承認を受けるには時間がかかる場合があるため、`Register-AzStackHCI` コマンドレットは終了し、登録は `pending admin consent` (部分的に完了した) 状態のままになります。 同意が得られたら、`Register-AzStackHCI` を再度実行して登録を完了します。

## <a name="assign-azure-ad-user-permissions"></a>Azure AD のユーザー アクセス許可を割り当てる

`Register-AzStackHCI` を実行するユーザーには、以下のために Azure AD のアクセス許可が必要です。

- Azure AD アプリケーションの作成 (`New-Remove-AzureADApplication`)、取得 (`Get-Remove-AzureADApplication`)、設定 (`Set-Remove-AzureADApplication`)、または削除 (`Remove-AzureADApplication`)。
- Azure AD サービス プリンシパルの作成 (`New-Get-AzureADServicePrincipal`) または取得 (`Get-AzureADServicePrincipal`)。
- Active Directory アプリケーションのシークレットの管理 (`New-Remove-AzureADApplicationKeyCredential`、`Get-Remove-AzureADApplicationKeyCredential`、または `Remove-AzureADApplicationKeyCredential`)。
- アプリケーションの特定のアクセス許可を使用する同意の付与 (`New-AzureADApplicationKeyCredential`、`Get-AzureADApplicationKeyCredential`、または `Remove-AzureADServiceAppRoleAssignments`)。

これらのアクセス許可を割り当てるには、3 つの方法があります。

### <a name="option-1-allow-any-user-to-register-applications"></a>オプション 1: すべてのユーザーにアプリケーションの登録を許可する

Azure Active Directory で、 **[ユーザー設定]**  >  **[アプリの登録]** に移動します。 **[ユーザーはアプリケーションを登録できる]** で **[はい]** を選択します。

このオプションを使用すると、すべてのユーザーがアプリケーションを登録できます。 ただし、ユーザーは、クラスターの登録時に、Azure AD 管理者に同意を付与してもらう必要があります。 

> [!NOTE]
> このオプションはテナントレベルの設定であるため、大企業のお客様には適さない場合があります。

### <a name="option-2-assign-the-cloud-application-administration-role"></a>オプション 2: Cloud Application Administration ロールを割り当てる

組み込みの Azure AD ロール "*Cloud Application Administration*" をユーザーに割り当てます。 この割り当てをすることで、ユーザーは追加の Active Directory 管理者の同意なしにクラスターを登録および登録解除できます。

### <a name="option-3-create-a-custom-active-directory-role-and-consent-policy"></a>オプション 3: カスタムの Active Directory ロールと同意ポリシーを作成する

最も制限の厳しいオプションは、Azure Stack HCI サービスに必要なアクセス許可について、テナント全体の管理者の同意を委任するカスタムの同意ポリシーを使用してカスタムの Active Directory ロールを作成することです。 このカスタム ロールを割り当てると、ユーザーは追加の Active Directory 管理者の同意を必要とすることなく、登録と同意の付与の両方を行うことができます。

> [!NOTE]
> このオプションを使用するには、Azure AD Premium ライセンスが必要です。 現在パブリック プレビュー段階にある、カスタムの Active Directory ロールとカスタムの同意ポリシー機能を使用します。

1. Azure AD に接続します。
   
   ```powershell
   Connect-AzureAD
   ```

2. カスタムの同意ポリシーを作成します。

   ```powershell
   New-AzureADMSPermissionGrantPolicy -Id "AzSHCI-registration-consent-policy" -DisplayName "Azure Stack HCI registration admin app consent policy" -Description "Azure Stack HCI registration admin app consent policy"
   ```

3. アプリ ID が 1322e676-dee7-41ee-a874-ac923822781c である Azure Stack HCI サービスに必要なアプリのアクセス許可を含む条件を追加します。 
   
   > [!NOTE]
   > 次のアクセス許可は、Azure Stack HCI の GA リリースの対象です。 クラスター内のすべてのサーバーに [2020 年 11 月 23 日のプレビュー更新プログラム (KB4586852)](https://support.microsoft.com/help/4595086/azure-stack-hci-release-notes-overview) を適用し、Az.StackHCI モジュール バージョン 0.4.1 以降をダウンロードしていない限り、パブリック プレビューでは機能しません。
   
   ```powershell
   New-AzureADMSPermissionGrantConditionSet -PolicyId "AzSHCI-registration-consent-policy" -ConditionSetType "includes" -PermissionType "application" -ResourceApplication "1322e676-dee7-41ee-a874-ac923822781c" -Permissions "bbe8afc9-f3ba-4955-bb5f-1cfb6960b242","8fa5445e-80fb-4c71-a3b1-9a16a81a1966","493bd689-9082-40db-a506-11f40b68128f","2344a320-6a09-4530-bed7-c90485b5e5e2"
   ```

4. 手順 2 で作成したカスタムの同意ポリシーに注意して、Azure Stack HCI の登録を許可するアクセス許可を付与します。
   
   ```powershell
   $displayName = "Azure Stack HCI Registration Administrator "
   $description = "Custom AD role to allow registering Azure Stack HCI "
   $templateId = (New-Guid).Guid
   $allowedResourceAction =
   @(
          "microsoft.directory/applications/createAsOwner",
          "microsoft.directory/applications/delete",
          "microsoft.directory/applications/standard/read",
          "microsoft.directory/applications/credentials/update",
          "microsoft.directory/applications/permissions/update",
          "microsoft.directory/servicePrincipals/appRoleAssignedTo/update",
          "microsoft.directory/servicePrincipals/appRoleAssignedTo/read",
          "microsoft.directory/servicePrincipals/appRoleAssignments/read",
          "microsoft.directory/servicePrincipals/createAsOwner",
          "microsoft.directory/servicePrincipals/credentials/update",
          "microsoft.directory/servicePrincipals/permissions/update",
          "microsoft.directory/servicePrincipals/standard/read",
          "microsoft.directory/servicePrincipals/managePermissionGrantsForAll.AzSHCI-registration-consent-policy"
   )
   $rolePermissions = @{'allowedResourceActions'= $allowedResourceAction}
   ```

5. 新しいカスタムの Active Directory ロールを作成します。

   ```powershell
   $customADRole = New-AzureADMSRoleDefinition -RolePermissions $rolePermissions -DisplayName $displayName -Description $description -TemplateId $templateId -IsEnabled $true
   ```

6. [これらの手順](/azure/active-directory/fundamentals/active-directory-users-assign-role-azure-portal?context=/azure/active-directory/roles/context/ugr-context)に従って、Azure Stack HCI クラスターを Azure に登録するユーザーに新しいカスタムの Active Directory ロールを割り当てます。

## <a name="unregister-azure-stack-hci-by-using-windows-admin-center"></a>Windows Admin Center を使用して Azure Stack HCI の登録を解除する

Azure Stack HCI クラスターの使用を停止する準備ができたら、Windows Admin Center を使用してクラスターに接続します。 左側の **[ツール]** メニューの下部にある **[設定]** を選択します。 次に、 **[Azure Stack HCI registration]\(Azure Stack HCI の登録\)** を選択し、 **[登録解除]** を設定します。 

登録解除プロセスによって、クラスターを表す Azure リソース、Azure リソース グループ (登録中に作成されたグループで、他のリソースが含まれていない場合)、および Azure AD アプリ ID が自動的にクリーンアップされます。 このクリーンアップにより、Azure Arc による監視、サポート、課金のすべての機能が停止します。

> [!NOTE]
> Azure Stack HCI クラスターの登録を解除するには、Azure AD 管理者、または[十分なアクセス許可](#assign-azure-ad-user-permissions)が委任されている別のユーザーが必要です。 
>
> Windows Admin Center ゲートウェイが、クラスターの初期登録に使用されたものとは異なる Azure Active Directory (テナント) ID に登録されている場合、Windows Admin Center を使用してクラスターの登録を解除しようとすると、問題が発生する可能性があります。 これが発生した場合は、次の PowerShell の手順を使用します。

## <a name="unregister-azure-stack-hci-by-using-powershell"></a>PowerShell を使用して Azure Stack HCI の登録を解除する

また、`Unregister-AzStackHCI` コマンドレットを使用して、Azure Stack HCI クラスターの登録を解除することもできます。 コマンドレットは、クラスター ノードまたは管理 PC のいずれかで実行できます。

場合によっては、最新バージョンの `Az.StackHCI` モジュールをインストールする必要があります。 `Are you sure you want to install the modules from 'PSGallery'?` のプロンプトが表示されたら、yes (`Y`) で応答します。

```PowerShell
Install-Module -Name Az.StackHCI
```

### <a name="unregister-from-a-cluster-node"></a>クラスター ノードから登録を解除する

クラスター内のサーバーで `Unregister-AzStackHCI` コマンドレットを実行している場合は、次の構文を使用します。 Azure サブスクリプション ID と、登録を解除する Azure Stack HCI クラスターのリソース名を指定します。

```PowerShell
Unregister-AzStackHCI -SubscriptionId "e569b8af-6ecc-47fd-a7d5-2ac7f23d8bfe" -ResourceName HCI001
```

別のデバイス (ご自分の PC や携帯電話など) で microsoft.com/devicelogin にアクセスするように求められます。 コードを入力し、そこにサインインして Azure の認証を行います。

### <a name="unregister-from-a-management-pc"></a>管理 PC から登録を解除する

管理用 PC からコマンドレットを実行している場合、クラスター内のサーバーの名前も指定する必要があります。

```PowerShell
Unregister-AzStackHCI -ComputerName ClusterNode1 -SubscriptionId "e569b8af-6ecc-47fd-a7d5-2ac7f23d8bfe" -ResourceName HCI001
```

対話型の Azure ログイン ウィンドウが表示されます。 表示される正確なプロンプトは、セキュリティ設定 (2 要素認証など) によって異なります。 画面の指示に従ってサインインします。

## <a name="clean-up-after-a-cluster-that-was-not-properly-unregistered"></a>正常に登録が解除されなかったクラスターをクリーンアップする

ユーザーが、ホスト サーバーの再イメージングや仮想クラスター ノードの削除などによって、Azure Stack HCI クラスターを登録解除せずに破棄すると、成果物が Azure に残ります。 そのような成果物は無害であり、課金されたり、リソースを消費したりすることはありませんが、そのせいで Azure portal が煩雑になることはあります。 これを整理するために、手動で削除することができます。

Azure Stack HCI リソースを削除するには、Azure portal のそのページに移動し、上部の操作バーから **[削除]** を選択します。 削除を確認するためにリソースの名前を入力し、 **[削除]** を選択します。 

Azure AD アプリ ID を削除するには、 **[Azure AD]**  >  **[アプリの登録]**  >  **[すべてのアプリケーション]** に移動します。 **[削除]** を選択して、確定します。

PowerShell を使用して Azure Stack HCI リソースを削除することもできます。

```PowerShell
Remove-AzResource -ResourceId "HCI001"
```

`Az.Resources` モジュールをインストールすることが必要になる場合があります。

```PowerShell
Install-Module -Name Az.Resources
```

リソース グループが登録時に作成され、他のリソースが含まれていない場合は、それも削除できます。

```PowerShell
Remove-AzResourceGroup -Name "HCI001-rg"
```

## <a name="next-steps"></a>次のステップ

関連情報については、以下をご覧ください。

- [Windows Admin Center を Azure に登録する](register-windows-admin-center.md)
- [Azure Stack HCI を Azure に接続する](../deploy/register-with-azure.md)