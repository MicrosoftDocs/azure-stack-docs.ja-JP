---
title: Azure での Azure Stack HCI クラスター登録の管理
description: Azure Stack HCI クラスターの Azure 登録を管理し、登録状態を把握して、使用停止の準備ができたときにクラスターの登録を解除する方法。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.date: 02/10/2021
ms.openlocfilehash: 7128ddae1a1b67e0085806ecd988f475c9bf8708
ms.sourcegitcommit: 5ea0e915f24c8bcddbcaf8268e3c963aa8877c9d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2021
ms.locfileid: "100487461"
---
# <a name="manage-cluster-registration-with-azure"></a>Azure でのクラスター登録の管理

> Azure Stack HCI v20H2 に適用されます

Azure Stack HCI クラスターを作成したら、[Windows Admin Center を Azure に登録](register-windows-admin-center.md)してから、[そのクラスターを Azure に登録](../deploy/register-with-azure.md)する必要があります。 クラスターが登録されると、オンプレミス クラスターとクラウドの間で定期的に情報が同期されます。 このトピックでは、登録状態を把握し、Azure Active Directory のアクセス許可を付与し、使用停止の準備ができたらクラスターの登録を解除する方法について説明します。

## <a name="understanding-registration-status-using-windows-admin-center"></a>Windows Admin Center を使用して登録状態を把握する

Windows Admin Center を使用してクラスターに接続すると、ダッシュボードが表示され、Azure の接続状態が表示されます。 "**接続済み**" とは、クラスターが既に Azure に登録され、過去 1 日以内にクラウドと正常に同期されたことを意味します。

   :::image type="content" source="media/manage-azure-registration/registration-status.png" alt-text="Windows Admin Center ダッシュボードにはクラスター接続状態が常に表示されます" lightbox="media/manage-azure-registration/registration-status.png":::

詳細情報を表示するには、左側の **[ツール]** メニューの一番下にある **[設定]** を選択し、 **[Azure Stack HCI registration]\(Azure Stack HCI の登録\)** を選択します。

   :::image type="content" source="media/manage-azure-registration/azure-stack-hci-registration.png" alt-text="[設定] > [ツール] > [Azure Stack HCI registration]\(Azure Stack HCI の登録\) を選択して詳細を表示" lightbox="media/manage-azure-registration/azure-stack-hci-registration.png":::

## <a name="understanding-registration-status-using-powershell"></a>PowerShell を使用して登録状態を把握する

Windows PowerShell を使用して登録状態を把握するには、`Get-AzureStackHCI` PowerShell コマンドレットと、`ClusterStatus`、`RegistrationStatus`、および `ConnectionStatus` の各プロパティを使用します。 たとえば、Azure Stack HCI オペレーティング システムをインストールした後、クラスターの作成または参加を行う前は、`ClusterStatus` プロパティには "not yet" (未完了) の状態が表示されます。

:::image type="content" source="media/manage-azure-registration/1-get-azurestackhci.png" alt-text="クラスターを作成する前の Azure の登録状態":::

クラスターが作成されると、`RegistrationStatus` にのみ "not yet" (未完了) の状態が表示されます。

:::image type="content" source="media/manage-azure-registration/2-get-azurestackhci.png" alt-text="クラスターを作成した後の Azure の登録状態":::

Azure Stack HCI では、Azure のオンライン サービス条件に従って、インストールから 30 日以内に登録する必要があります。 30 日間の経過後にクラスター化されていない場合、`ClusterStatus` には `OutOfPolicy` が表示され、30 日間の経過後に登録されていない場合、`RegistrationStatus` には `OutOfPolicy` が表示されます。

クラスターが登録されると、`ConnectionStatus` と `LastConnected` の時刻が表示されます。クラスターがインターネットから一時的に切断されていなければ、通常、これは最終日の値です。 Azure Stack HCI クラスターは、最大 30 日間、完全にオフラインで動作できます。

:::image type="content" source="media/manage-azure-registration/3-get-azurestackhci.png" alt-text="登録後の Azure の登録状態":::

その最大期間を超えた場合、`ConnectionStatus` には `OutOfPolicy` が表示されます。

## <a name="azure-active-directory-app-permissions"></a>Azure Active Directory アプリのアクセス許可

サブスクリプションで Azure リソースを作成することに加えて、Azure Stack HCI を登録すると、概念的にはユーザーに似たアプリ ID が Azure Active Directory テナントに作成されます。 アプリ ID にはクラスター名が継承されます。 この ID は、サブスクリプション内で、Azure Stack HCI クラウド サービスに代わって適切に機能します。

クラスターを登録するユーザーが Azure Active Directory 管理者であるか、十分なアクセス許可が委任されている場合、これはすべて自動的に行われるため、追加の操作は必要ありません。 そうでない場合は、登録を完了するには、Azure Active Directory 管理者の承認が必要になる場合があります。 管理者は、アプリに明示的に同意するか、アプリに同意できるアクセス許可を委任することができます。

:::image type="content" source="media/manage-azure-registration/aad-permissions.png" alt-text="Azure Active Directory のアクセス許可と ID の図" border="false":::

同意するには、[portal.azure.com](https://portal.azure.com) を開き、Azure Active Directory に対して十分なアクセス許可を持つ Azure アカウントでサインインします。 **Azure Active Directory** に移動し、 **[アプリの登録]** を選択します。 クラスターにちなんで名付けられたアプリ ID を選択し、 **[API のアクセス許可]** に移動します。

Azure Stack HCI の一般提供 (GA) リリースの場合、アプリには次のアクセス許可が必要です。これは、パブリック プレビューの場合に必要なアプリのアクセス許可とは異なります。

```http
https://azurestackhci-usage.trafficmanager.net/AzureStackHCI.Cluster.Read

https://azurestackhci-usage.trafficmanager.net/AzureStackHCI.Cluster.ReadWrite

https://azurestackhci-usage.trafficmanager.net/AzureStackHCI.ClusterNode.Read

https://azurestackhci-usage.trafficmanager.net/AzureStackHCI.ClusterNode.ReadWrite
```

パブリック プレビューの場合、アプリのアクセス許可は次の内容でした (現在、これらは非推奨です)。

```http
https://azurestackhci-usage.trafficmanager.net/AzureStackHCI.Census.Sync

https://azurestackhci-usage.trafficmanager.net/AzureStackHCI.Billing.Sync
```

Azure Active Directory 管理者から承認を受けるには時間がかかる場合があるため、`Register-AzStackHCI` コマンドレットは終了し、登録は "pending admin consent" (管理者の同意待ち)、つまり部分的に完了した状態のままになります。 同意が得られたら、`Register-AzStackHCI` をもう一度実行するだけで登録は完了します。

## <a name="azure-active-directory-user-permissions"></a>Azure Active Directory ユーザーのアクセス許可

Register-AzStackHCI を実行するユーザーには、以下のために Azure AD のアクセス許可が必要です。

- Azure AD アプリケーションの作成、取得、設定、削除 (New/Get/Set/Remove-AzureADApplication)
- Azure AD サービス プリンシパルの作成、取得 (New/Get-New-AzureADServicePrincipal)
- AD アプリケーション シークレットの管理 (New/Get/Remove-AzureADApplicationKeyCredential)
- 特定のアプリケーションのアクセス許可を使用する同意の付与 (New/Get/Remove AzureADServiceAppRoleAssignments)

これを実現するには、次の 3 つの方法があります。

### <a name="option-1-allow-any-user-to-register-applications"></a>オプション 1: すべてのユーザーにアプリケーションの登録を許可する

Azure Active Directory で、 **[ユーザー設定] > [アプリの登録]** に移動します。 **[ユーザーはアプリケーションを登録できる]** で **[はい]** を選択します。

これにより、すべてのユーザーはアプリケーションを登録できるようになります。 ただし、ユーザーは、クラスターの登録時に、Azure AD 管理者に同意を付与してもらう必要があります。 これはテナント レベルの設定であるため、大企業のお客様には適さない場合があることに注意してください。

### <a name="option-2-assign-cloud-application-administration-role"></a>オプション 2:Cloud Application Administration ロールを割り当てる

組み込みの "Cloud Application Administration" Azure AD ロールをユーザーに割り当てます。 こうすることで、ユーザーは追加の AD 管理者の同意なしにクラスターを登録および登録解除できます。

### <a name="option-3-create-a-custom-ad-role-and-consent-policy"></a>オプション 3:カスタムの AD ロールと同意ポリシーを作成する

最も制限の厳しいオプションは、Azure Stack HCI サービスに必要なアクセス許可について、テナント全体の管理者の同意を委任するカスタムの同意ポリシーを使用してカスタムの AD ロールを作成することです。 このカスタム ロールが割り当てられると、ユーザーは追加の AD 管理者の同意を必要とすることなく、登録と同意の付与の両方を行うことができます。

   > [!NOTE]
   > このオプションには Azure AD Premium ライセンスが必要であり、現在はパブリック プレビュー段階にあるカスタムの AD ロールとカスタムの同意ポリシー機能が使用されます。

   1. Azure AD に接続します。
   
      ```powershell
      Connect-AzureAD
      ```

   2. カスタムの同意ポリシーを作成します。

      ```powershell
      New-AzureADMSPermissionGrantPolicy -Id "AzSHCI-registration-consent-policy" -DisplayName "Azure Stack HCI registration admin app consent policy" -Description "Azure Stack HCI registration admin app consent policy"
      ```

   3. アプリ ID が 1322e676-dee7-41ee-a874-ac923822781c である Azure Stack HCI サービスに必要なアプリのアクセス許可を含む条件を追加します。 次のアクセス許可は Azure Stack HCI の GA リリース用です。クラスター内のすべてのサーバーに [2020 年 11 月 23 日のプレビュー更新プログラム (KB4586852)](https://support.microsoft.com/help/4595086/azure-stack-hci-release-notes-overview) を適用し、Az.StackHCI モジュール バージョン 0.4.1 以降をダウンロードしていない限り、パブリック プレビューでは機能しません。
   
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

   5. 新しいカスタムの AD ロールを作成します。

      ```powershell
      $customADRole = New-AzureADMSRoleDefinition -RolePermissions $rolePermissions -DisplayName $displayName -Description $description -TemplateId $templateId -IsEnabled $true
      ```

   6. [これらの手順](/azure/active-directory/fundamentals/active-directory-users-assign-role-azure-portal?context=/azure/active-directory/roles/context/ugr-context)に従って、Azure Stack HCI クラスターを Azure に登録するユーザーに新しいカスタムの AD ロールを割り当てます。

## <a name="unregister-azure-stack-hci-using-windows-admin-center"></a>Windows Admin Center を使用して Azure Stack HCI の登録を解除する

ご自身の Azure Stack HCI クラスターの使用を停止する準備ができたら、Windows Admin Center を使用してクラスターに接続し、左側の **[ツール]** メニューの一番下にある **[設定]** を選択するだけです。 次に、 **[Azure Stack HCI registration]\(Azure Stack HCI の登録\)** を選択し、 **[登録解除]** をクリックします。 登録解除プロセスによって、クラスターを表す Azure リソース、Azure リソース グループ (登録中に作成されたグループで、他のリソースが含まれていない場合)、および Azure AD アプリ ID が自動的にクリーンアップされます。 これにより、Azure Arc による監視、サポート、課金のすべての機能が停止します。

   > [!NOTE]
   > Azure Stack HCI クラスターの登録を解除するには、Azure Active Directory 管理者、または十分なアクセス許可を委任されている他のユーザーが必要です。 「[Azure Active Directory ユーザーのアクセス許可](#azure-active-directory-user-permissions)」をご覧ください。 Windows Admin Center ゲートウェイが、クラスターの初期登録に使用されたものとは異なる Azure Active Directory (テナント) ID に登録されている場合、Windows Admin Center を使用してクラスターの登録を解除しようとすると問題が発生する可能性があります。 これが発生した場合は、次の PowerShell の手順に従います。

## <a name="unregister-azure-stack-hci-using-powershell"></a>PowerShell を使用して Azure Stack HCI の登録を解除する

また、`Unregister-AzStackHCI` コマンドレットを使用して、Azure Stack HCI クラスターの登録を解除することもできます。 コマンドレットは、クラスター ノードまたは管理 PC のいずれかで実行できます。

場合によっては、最新バージョンの `Az.StackHCI` モジュールをインストールする必要があります。 " **'PSGallery' からモジュールをインストールしますか?** " というプロンプトが表示されたら、 **[はい]** (Y) と答えます。

```PowerShell
Install-Module -Name Az.StackHCI
```

### <a name="unregister-from-a-cluster-node"></a>クラスター ノードから登録を解除する

クラスター内のサーバー上で `Unregister-AzStackHCI` コマンドレットを実行する場合、この構文を使用し、自分の Azure サブスクリプション ID と、登録を解除したい Azure Stack HCI クラスターのリソース名を指定します。

```PowerShell
Unregister-AzStackHCI -SubscriptionId "e569b8af-6ecc-47fd-a7d5-2ac7f23d8bfe" -ResourceName HCI001
```

別のデバイス (ご自分の PC や携帯電話など) で microsoft.com/devicelogin にアクセスするよう求められるので、コードを入力し、そこにサインインして Azure の認証を行います。

### <a name="unregister-from-a-management-pc"></a>管理 PC から登録を解除する

管理用 PC からコマンドレットを実行する場合、クラスター内のサーバーの名前も指定する必要があります。

```PowerShell
Unregister-AzStackHCI -ComputerName ClusterNode1 -SubscriptionId "e569b8af-6ecc-47fd-a7d5-2ac7f23d8bfe" -ResourceName HCI001
```

対話型の Azure ログイン ウィンドウがポップアップ表示されます。 表示される正確なプロンプトは、セキュリティ設定 (2 要素認証など) によって異なります。 画面の指示に従ってログインします。

## <a name="cleaning-up-after-a-cluster-that-was-not-properly-unregistered"></a>正常に登録が解除されなかったクラスターを整理する

ユーザーがホストサーバーの再イメージングや仮想クラスター ノードの削除などによって、Azure Stack HCI クラスターの登録を解除せずに破棄すると、成果物が Azure に残されます。 これらは無害であり、課金されたり、リソースを消費したりすることはありませんが、そのせいで Azure portal が煩雑になることはあります。 これを整理するために、手動で削除することができます。

Azure Stack HCI リソースを削除するには、Azure portal のそのページに移動し、上部の操作バーから **[削除]** を選択します。 削除を確認するためにリソースの名前を入力し、 **[削除]** をクリックします。 Azure AD アプリ ID を削除するには、**Azure AD**、 **[アプリの登録]** の順に移動して、 **[すべてのアプリケーション]** の下にそれを表示します。 **[削除]** を選択して、確定します。

PowerShell を使用して Azure Stack HCI リソースを削除することもできます。

```PowerShell
Remove-AzResource -ResourceId "HCI001"
```

`Az.Resources` モジュールのインストールが必要になる場合があります。

```PowerShell
Install-Module -Name Az.Resources
```

リソース グループが登録時に作成され、他のリソースが含まれていない場合は、それも削除できます。

```PowerShell
Remove-AzResourceGroup -Name "HCI001-rg"
```

## <a name="next-steps"></a>次のステップ

関連情報については、以下もご覧ください。

- [Windows Admin Center を Azure に登録する](register-windows-admin-center.md)
- [Azure Stack HCI を Azure に接続する](../deploy/register-with-azure.md)