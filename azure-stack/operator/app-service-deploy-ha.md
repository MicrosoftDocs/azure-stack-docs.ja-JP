---
title: 高可用性構成で Azure Stack Hub の App Service をデプロイする
description: 高可用性構成を使用して Azure Stack Hub に App Service をデプロイする方法について学習します。
author: BryanLa
ms.topic: article
ms.date: 01/02/2020
ms.author: anwestg
ms.reviewer: anwestg
ms.lastreviewed: 01/02/2019
ms.openlocfilehash: ec4f3dc2a17e362038d11ec988d19ffa9edd6a6e
ms.sourcegitcommit: a630894e5a38666c24e7be350f4691ffce81ab81
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2020
ms.locfileid: "77701855"
---
# <a name="deploy-app-service-in-a-highly-available-configuration"></a>高可用性構成で App Service をデプロイする

この記事では、Azure Stack Hub Marketplace 項目を使用して、高可用性構成で Azure Stack Hub 用に App Service をデプロイする方法について示します。 このソリューションでは、利用可能な Marketplace 項目に加えて、[appservice-fileshare-sqlserver-ha](https://github.com/Azure/azurestack-quickstart-templates/tree/master/appservice-fileserver-sqlserver-ha) Azure Stack Hub クイックスタート テンプレートも使用します。 このテンプレートによって、App Service リソース プロバイダーをホストするための高可用性インフラストラクチャの作成が自動化されます。 その後、この高可用性 VM インフラストラクチャに App Service がインストールされます。 

## <a name="deploy-the-highly-available-app-service-infrastructure-vms"></a>高可用性 App Service インフラストラクチャ VM をデプロイする
[appservice-fileshare-sqlserver-ha](https://github.com/Azure/azurestack-quickstart-templates/tree/master/appservice-fileserver-sqlserver-ha) Azure Stack Hub クイックスタート テンプレートを使用すると、高可用性構成で App Service を簡単にデプロイできます。 これは [Default Provider Subscription]\(既定のプロバイダー サブスクリプション\) にデプロイする必要があります。 

Azure Stack Hub でのカスタム リソースの作成にこのテンプレートを使用すると、次のものが作成されます。
- 仮想ネットワークと必要なサブネット。
- ファイル サーバー、SQL Server、および Active Directory Domain Services (AD DS) サブネットのネットワーク セキュリティ グループ。
- VM ディスクとクラスター クラウド監視のストレージ アカウント。
- プライベート IP が SQL AlwaysOn リスナーにバインドされた SQL VM 用の 1 つの内部ロードバランサー。
- AD DS、ファイル サーバー クラスター、および SQL クラスターの 3 つの可用性セット。
- 2 ノード SQL クラスター。
- 2 ノード ファイル サーバー クラスター。
- 2 台のドメイン コントローラー。

### <a name="required-azure-stack-hub-marketplace-items"></a>必須の Azure Stack Hub Marketplace 項目
このテンプレートを使用する前に、次の [Azure Stack Hub Marketplace 項目](azure-stack-marketplace-azure-items.md)が Azure Stack Hub インスタンスで利用できることを確実にします。

- Windows Server 2016 Datacenter Core イメージ (AD DS とファイルサーバー VM 用)
- Windows Server 2016 (Enterprise) の SQL Server 2016 SP2
- 最新の SQL IaaS 拡張機能 
- 最新の PowerShell Desired State Configuration 拡張機能 

> [!TIP]
> テンプレートの要件と既定値について詳しくは、GitHub の[テンプレートの README ファイル](https://github.com/Azure/azurestack-quickstart-templates/tree/master/appservice-fileserver-sqlserver-ha)を確認してください。 

### <a name="deploy-the-app-service-infrastructure"></a>App Service インフラストラクチャをデプロイする
このセクションの手順を使用し、**appservice-fileshare-sqlserver-ha** Azure Stack Hub クイックスタート テンプレートを使用してカスタム デプロイを作成します。

1. [!INCLUDE [azs-admin-portal](../includes/azs-admin-portal.md)]

2. **\+** **[リソースの作成]**  >  **[カスタム]** の順に選択し、 **[テンプレートのデプロイ]** を選択します。

   ![カスタム テンプレートのデプロイ](media/app-service-deploy-ha/1.png)


3. **[カスタム デプロイ]** ブレードで **[テンプレートの編集]**  >  **[クイック スタート テンプレート]** を選択し、使用できるカスタム テンプレートのドロップダウン リストを使用して **appservice-fileshare-sqlserver-ha** テンプレートを選択します。 **[OK]** 、 **[保存]** の順にクリックします。

   ![appservice-fileshare-sqlserver-ha クイック スタート テンプレートを選択します](media/app-service-deploy-ha/2.png)

4. **[カスタム デプロイ]** ブレードで、 **[パラメーターの編集]** を選択し、下にスクロールしてテンプレートの既定値を確認します。 必要に応じてそれらの値を変更して、必要なすべてのパラメーター情報を入力し、 **[OK]** をクリックします。<br><br> 少なくとも、`ADMINPASSWORD`、`FILESHAREOWNERPASSWORD`、`FILESHAREUSERPASSWORD`、`SQLSERVERSERVICEACCOUNTPASSWORD`、および `SQLLOGINPASSWORD` パラメーターには、複雑なパスワードを指定してください。
    
   ![カスタム デプロイ パラメーターを編集する](media/app-service-deploy-ha/3.png)

5. **[カスタム デプロイ]** ブレードで、使用するサブスクリプションとして **[Default Provider Subscription]\(既定のプロバイダー サブスクリプション\)** が選択されていることを確認してから、カスタム デプロイ用に新しいリソース グループを作成するか、既存のリソース グループを選択します。<br><br> 次に、リソース グループの場所 (ASDK のインストールの場合は **local**) を選択し、 **[作成]** をクリックします。 テンプレートのデプロイが開始される前にカスタムのデプロイ設定が検証されます。

    ![カスタム デプロイを作成する](media/app-service-deploy-ha/4.png)

6. 管理者ポータルで、 **[リソース グループ]** を選択してから、カスタム デプロイ用に作成したリソース グループの名前を選択します (この例では **app-service-ha**)。 すべてのデプロイが正常に完了したことを確認するために、デプロイの状態を表示します。

   > [!NOTE]
   > テンプレートのデプロイが完了するまでに約 1 時間かかります。

   [![](media/app-service-deploy-ha/5-sm.png "Review template deployment status")](media/app-service-deploy-ha/5-lg.png#lightbox)


### <a name="record-template-outputs"></a>テンプレートの出力を記録する
テンプレートのデプロイが正常に完了したら、テンプレートのデプロイの出力を記録します。 この情報は、App Service インストーラーを実行するときに必要になります。

これらの出力値のそれぞれを必ず記録してください。
- FileSharePath
- FileShareOwner
- FileShareUser
- SQLServer
- SQLuser

テンプレートの出力値を見つけるには、次の手順に従ってください。

1. [!INCLUDE [azs-admin-portal](../includes/azs-admin-portal.md)]

2. 管理者ポータルで、 **[リソース グループ]** を選択してから、カスタム デプロイ用に作成したリソース グループの名前を選択します (この例では **app-service-ha**)。 

3. **[デプロイ]** をクリックしてから、 **[Microsoft.Template]** を選択します。

    ![Microsoft.template のデプロイ](media/app-service-deploy-ha/6.png)

4. **[Microsoft.Template]** デプロイを選択した後、 **[出力]** を選択し、テンプレート パラメーターの出力を記録します。 App Service をデプロイするときに、この情報が必要になります。

    ![パラメーターの出力](media/app-service-deploy-ha/7.png)


## <a name="deploy-app-service-in-a-highly-available-configuration"></a>高可用性構成で App Service をデプロイする
このセクションの手順に従って、[appservice-fileshare-sqlserver-ha](https://github.com/Azure/azurestack-quickstart-templates/tree/master/appservice-fileserver-sqlserver-ha) Azure Stack Hub クイックスタート テンプレートに基づき、高可用性構成で Azure Stack Hub 用に App Service をデプロイします。 

App Service リソース プロバイダーをインストールした後で、オファーやプランに含めることができます。 ユーザーはサブスクライブしてサービスを取得し、アプリケーションの作成を開始できます。

> [!IMPORTANT]
> リソース プロバイダーのインストーラーを実行する前に、App Service の各リリースに付属しているリリース ノートを読んで、新しい機能、修正点、およびデプロイに影響を与える可能性のある既知の問題を把握していることを確認してください。

### <a name="prerequisites"></a>前提条件
App Service インストーラーを実行する前に、[App Service on Azure Stack Hub を開始する前に](azure-stack-app-service-before-you-get-started.md)に関する記事で説明されている手順をいくつか実行する必要があります。

> [!TIP]
> テンプレートのデプロイによってインフラストラクチャ VM が自動的に構成されるため、[App Service を開始する前に関する記事](azure-stack-app-service-before-you-get-started.md)で説明されているすべての手順が必ずしも必要というわけではありません。

- [App Service インストーラーおよびヘルパー スクリプトをダウンロードする](azure-stack-app-service-before-you-get-started.md#download-the-installer-and-helper-scripts)。
- [Azure Stack Hub Marketplace から項目をダウンロードする](azure-stack-app-service-before-you-get-started.md#download-items-from-the-azure-marketplace)。
- [必要な証明書を生成する](azure-stack-app-service-before-you-get-started.md#get-certificates)。
- Azure Stack Hub 用に選択した ID プロバイダーに基づいて ID アプリケーションを作成する。 ID アプリケーションは [Azure AD](azure-stack-app-service-before-you-get-started.md#create-an-azure-active-directory-app)または [Active Directory フェデレーション サービス (AD FS)](azure-stack-app-service-before-you-get-started.md#create-an-active-directory-federation-services-app) のいずれかに対して作成し、アプリケーション ID を記録できます。
- Windows Server 2016 Datacenter イメージを Azure Stack Hub Marketplace に確実に追加してください。 このイメージは App Service のインストールに必要です。

### <a name="steps-for-app-service-deployment"></a>App Service のデプロイの手順
App Service リソース プロバイダーのインストールには少なくとも 1 時間かかります。 必要な時間の長さは、デプロイするロール インスタンスの数によって異なります。 デプロイ中に、インストーラーは次のタスクを実行します。

- 指定した Azure Stack Hub ストレージ アカウントに BLOB コンテナーを作成します。
- App Service の DNS ゾーンとエントリを作成します。
- App Service リソース プロバイダーを登録します。
- App Service のギャラリー アイテムを登録します。

App Service リソース プロバイダーをデプロイするには、次の手順を実行します。

1. Azure Stack Hub の管理者の Azure Resource Management エンドポイントにアクセスできるコンピューターから、以前にダウンロードした App Service インストーラー (**appservice.exe**) を管理者として実行します。

2. **[Deploy App Service or upgrade to the latest version]\(App Service をデプロイするか、または最新バージョンにアップグレードする\)** を選択します。

    ![App Service のデプロイまたはアップグレード](media/app-service-deploy-ha/01.png)

3. Microsoft のライセンス条項に同意し、 **[次へ]** をクリックします。

    ![App Service の Microsoft ライセンス条項](media/app-service-deploy-ha/02.png)

4. Microsoft 以外のライセンス条項に同意し、 **[次へ]** をクリックします。

    ![App Service の Microsoft 以外のライセンス条項](media/app-service-deploy-ha/03.png)

5. Azure Stack Hub 環境に App Service クラウド エンドポイント構成を指定します。

    ![App Service のクラウド エンドポイント構成](media/app-service-deploy-ha/04.png)

6. インストールに使用する Azure Stack Hub サブスクリプションに**接続**し、場所を選択します。 

    ![App Service で Azure Stack Hub サブスクリプションに接続する](media/app-service-deploy-ha/05.png)

7. 高可用性テンプレートのデプロイに使用されるリソース グループに **[既存の VNet とサブネットを使用する]** と **[リソース グループ名]** を選択します。<br><br>次に、テンプレートのデプロイの一環として作成された仮想ネットワークを選択してから、ドロップダウン リスト オプションから適切なロールのサブネットを選択します。 

    ![App Service での Vnet の選択](media/app-service-deploy-ha/06.png)

8. ファイル共有パスとファイル共有所有者のパラメーターに、以前に記録したテンプレート出力情報を入力します。 完了したら、 **[次へ]** をクリックします。

    ![App Service でのファイル共有出力情報](media/app-service-deploy-ha/07.png)

9. App Service のインストールに使用されているマシンは、App Service のファイル共有をホストするために使用されているファイル サーバーと同じ VNet 上に配置されていないため、ユーザーは名前を解決することができません。 **このエラーは予想される動作です**。<br><br>ファイル共有 UNC パスとアカウント情報に入力された情報が正しいことを確認します。 次に、アラート ダイアログで **[はい]** をクリックして、App Service のインストールを続行します。

    ![App Service の予期されるエラー ダイアログ](media/app-service-deploy-ha/08.png)

    既存の仮想ネットワークおよび内部 IP アドレスにデプロイして、ファイル サーバーに接続する場合は、送信セキュリティ規則を追加する必要があります。 この規則により、worker サブネットとファイル サーバー間の SMB トラフィックが有効になります。 管理者ポータルで WorkersNsg に移動し、次のプロパティを持つ送信セキュリティ規則を追加します。
    - 送信元: 任意
    - 送信元ポート範囲: *
    - 送信先: IP アドレス
    - 送信先 IP アドレス範囲: ファイル サーバーの IP の範囲
    - 送信先ポート範囲: 445
    - プロトコル: TCP
    - アクション: 許可
    - 優先順位: 700
    - 名前: Outbound_Allow_SMB445

10. Identity Application ID を入力し、ID 証明書のパスとパスワードを入力して、 **[次へ]** をクリックします。
    - Identity Application 証明書 (**sso.appservice.local.azurestack.external.pfx** の形式)
    - Azure Resource Manager ルート証明書 (**AzureStackCertificationAuthority.cer**)

    ![App Service での ID Application 証明書とルート証明書](media/app-service-deploy-ha/008.png)

11. 次に、以下の証明書に必要な残りの情報を入力して、 **[次へ]** をクリックします。
    - 既定の Azure Stack Hub SSL 証明書 ( **_.appservice.local.azurestack.external.pfx** の形式)
    - API SSL 証明書 (**api.appservice.local.azurestack.external.pfx** の形式)
    - パブリッシャー証明書 (**ftp.appservice.local.azurestack.external.pfx** の形式) 

    ![App Service での追加の構成証明書](media/app-service-deploy-ha/09.png)

12. 高可用性テンプレートのデプロイ出力からの SQL Server 接続情報を使用して、SQL Server 接続情報を入力します。

    ![App Service での SQL Server の接続情報](media/app-service-deploy-ha/10.png)

13. App Service のインストールに使用されているマシンは、App Service データベースをホストするために使用されている SQL Server と同じ VNet 上に配置されていないため、ユーザーは名前を解決することができません。  **これは予想される動作です**。<br><br>SQL Server の名前とアカウント情報に入力した情報が正しいことを確認し、 **[はい]** を押して App Service のインストールを続行してください。 **[次へ]** をクリックします。

    ![App Service での SQL Server の接続情報](media/app-service-deploy-ha/11.png)

14. 既定のロール構成値を受け入れるか、または推奨値に変更して **[次へ]** をクリックします。<br><br>高可用性構成の場合は、App Service インフラストラクチャ ロール インスタンスの既定値を次のように変更することをお勧めします。

    |Role|Default|高可用性の推奨設定|
    |-----|-----|-----|
    |コントローラー ロール|2|2|
    |管理ロール|1|3|
    |パブリッシャー ロール|1|3|
    |FrontEnd ロール|1|3|
    |共有 worker ロール|1|2|
    |     |     |     |

    ![App Service でのインフラストラクチャ ロール インスタンスの値](media/app-service-deploy-ha/12.png)

    > [!NOTE]
    > 既定値からこのチュートリアルで推奨されている値に変更すると、App Service をインストールするためのハードウェア要件が増加します。 6 VM 用に既定の 9 コアおよび 16,128 MB の RAM をサポートする代わりに、推奨される 13 VM をサポートするには、合計 18 コアおよび 32,256 MB のRAM が必要です。

15. App Service インフラストラクチャ VM のインストールに使用するプラットフォーム イメージを選択して、 **[次へ]** をクリックします。

    ![App Service でのプラットフォーム イメージの選択](media/app-service-deploy-ha/13.png)

16. 使用する App Service インフラストラクチャ ロールの資格情報を入力して、 **[次へ]** をクリックします。

    ![App Service でのインフラストラクチャ ロールの資格情報](media/app-service-deploy-ha/14.png)

17. App Service のデプロイに使用される情報を確認し、 **[次へ]** をクリックしてデプロイを開始します。

    ![App Service のインストールの概要を確認する](media/app-service-deploy-ha/15.png)

18. App Service デプロイの進行状況を確認します。 特定のデプロイ構成とハードウェアによっては、このデプロイに 1 時間以上かかることがあります。 インストーラーが正常に完了したら、 **[終了]** を選択します。

    ![App Service のセットアップの完了](media/app-service-deploy-ha/16.png)

## <a name="next-steps"></a>次のステップ

SQL Always On インスタンスで App Service リソース プロバイダーを提供した場合は、[appservice_hosting および appservice_metering データベースを可用性グループに追加します](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/availability-group-add-a-database)。 データベースのフェールオーバーが発生した場合のサービスの損失を防ぐため、データベースを同期します。 また、[スクリプト](https://blog.sqlauthority.com/2017/11/30/sql-server-alwayson-availability-groups-script-sync-logins-replicas/)を実行して、元のプライマリ サーバーからフェールオーバー サーバーへ AppServices ログインをインポートすることもできます。

[App Service をスケールアウトします](azure-stack-app-service-add-worker-roles.md)。 ご使用の環境で予想されるアプリ需要を満たすために、App Service インフラストラクチャ ロール worker を追加する必要がある場合があります。 既定では、App Service on Azure Stack Hub は無料の共有 worker 層をサポートしています。 他の worker 階層を追加するには、worker ロールを追加する必要があります。

[デプロイ ソースを構成します](azure-stack-app-service-configure-deployment-sources.md)。 GitHub、BitBucket、OneDrive、DropBox などの複数のソース管理プロバイダーからのオンデマンド デプロイをサポートするには、追加の構成が必要です。

[App Service をバックアップします](app-service-back-up.md)。 App Service を正常にデプロイして構成したら、ディザスター リカバリーに必要なすべてのコンポーネントが確実にバックアップされていることを確認する必要があります。 必須コンポーネントをバックアップすると、復旧操作中にデータ損失や不要なサービス ダウンタイムを防ぐことができます。
