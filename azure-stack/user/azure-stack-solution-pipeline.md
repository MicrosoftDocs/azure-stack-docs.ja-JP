---
title: Azure と Azure Stack にアプリをデプロイする | Microsoft Docs
description: ハイブリッド CI/CD パイプラインを使用して、アプリを Azure と Azure Stack にデプロイする方法を説明します。
services: azure-stack
documentationcenter: ''
author: bryanla
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: solution
ms.date: 03/11/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/07/2018
ms.openlocfilehash: 9fbadb923452fc2420d1f8626a69d377c4d72e12
ms.sourcegitcommit: 2a4cb9a21a6e0583aa8ade330dd849304df6ccb5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/17/2019
ms.locfileid: "68286961"
---
# <a name="deploy-apps-to-azure-and-azure-stack"></a>Azure と Azure Stack へのアプリのデプロイ

*適用対象: Azure Stack 統合システムと Azure Stack Development Kit*

ハイブリッドの継続的インテグレーション/継続的デリバリー (CI/CD) パイプラインを使用して、Azure と Azure Stack にアプリをデプロイする方法について学習します。

このソリューションでは、以下を実現するためのサンプル環境を作成します。

> [!div class="checklist"]
> * コード コミットに基づいて、Azure DevOps Services リポジトリに対して新しいビルドを開始します。
> * ユーザー受け入れテストを目的として、アプリをグローバル Azure に自動的にデプロイします。
> * コードがテストに合格した時点で自動的にアプリを Azure Stack にデプロイします。

## <a name="benefits-of-the-hybrid-delivery-build-pipeline"></a>ハイブリッド デリバリー ビルド パイプラインの利点

継続性、セキュリティ、信頼性は、アプリ デプロイの重要な要素です。 これらは組織にとってこの上なく重要な要素であり、開発チームにとっては生命線ともいうべきものです。 ハイブリッド CI/CD パイプラインを使用することにより、オンプレミス環境およびパブリック クラウド全体でビルド パイプを統合することができます。 ハイブリッド デリバリー モデルを使用すれば、アプリに変更を加えることなくデプロイ先を変更することもできます。

他にも、ハイブリッド アプローチの使用には次のような利点があります。

* オンプレミスの Azure Stack 環境と Azure パブリック クラウドとの間で各種開発ツールの一貫性を確保できます。  一連のツールが共通化されることで、CI/CD のパターンや手法が導入しやすくなります。
* Azure または Azure Stack にデプロイされたアプリとサービスは互換性があり、どちらの場所でも同じコードを実行することができます。 オンプレミスとパブリック クラウドの機能を活かすことができます。

CI と CD については、次の記事をご覧ください。

* [継続的インテグレーションとは](https://www.visualstudio.com/learn/what-is-continuous-integration/)
* [継続的デリバリーとは](https://www.visualstudio.com/learn/what-is-continuous-delivery/)

> [!Tip]  
> ![hybrid-pillars.png](./media/azure-stack-solution-cloud-burst/hybrid-pillars.png)  
> Microsoft Azure Stack は Azure の拡張機能です。 Azure Stack はオンプレミス環境にクラウド コンピューティングの機敏性とイノベーションをもたらし、ハイブリッド アプリをビルドし、どこにでもデプロイできる唯一のハイブリッド クラウドを可能にします。  
> 
> [ハイブリッド アプリケーションのための設計の考慮事項](azure-stack-edge-pattern-overview.md)に関する記事では、ハイブリッド アプリケーションを設計、デプロイ、および運用するためのソフトウェア品質の重要な要素 (配置、スケーラビリティ、可用性、回復性、管理容易性、およびセキュリティ) についてレビューしています。 これらの設計の考慮事項は、ハイブリッド アプリの設計を最適化したり、運用環境での課題を最小限に抑えたりするのに役立ちます。

## <a name="prerequisites"></a>前提条件

ハイブリッド CI/CD パイプラインを構築するには、いくつかのコンポーネントが必要になります。 次のコンポーネントは準備に時間がかかります。

* 運用 Azure Stack は、Azure OEM/ハードウェア パートナーがデプロイできます。 Azure Stack Development Kit (ASDK) は、すべてのユーザーがデプロイできます。
* Azure Stack オペレーターは、App Service のデプロイ、プランとオファーの作成、テナント サブスクリプションの作成、および Windows Server 2016 イメージの追加という項目を完了する必要があります。

>[!NOTE]
>これらのコンポーネントの一部がデプロイ済みである場合は、このソリューションを開始する前にそれらがすべての要件を満たしていることを確認してください。

このソリューションは、Azure と Azure Stack について一定の基本知識があることを前提にしています。 ソリューションを開始する前に詳細を確認するには、次の記事をお読みください。

* [Azure 入門](https://azure.microsoft.com/overview/what-is-azure/)
* [Azure Stack の主要概念](../operator/azure-stack-overview.md)

### <a name="azure-requirements"></a>Azure の要件

* Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) を作成してください。
* Azure で [Web アプリ](https://docs.microsoft.com/azure/app-service/overview)を作成します。 Web アプリの URL は書き留めておいてください。このソリューションの中で必要になります。

### <a name="azure-stack-requirements"></a>Azure Stack の要件

* Azure Stack 統合システムを使用するか、Azure Stack Development Kit (ASDK) をデプロイします。 ASDK をデプロイするには、次の手順を実行します。
  * [ソリューション:インストーラーを使用する ASDK のデプロイ](../asdk/asdk-install.md)に関する記事に詳しいデプロイ手順が示されています。
  * [ConfigASDK.ps1](https://github.com/mattmcspirit/azurestack/blob/master/deployment/ConfigASDK.ps1 ) PowerShell スクリプトを使って ASDK デプロイ後の手順を自動化します。

    > [!Note]
    > ASDK のインストールは完了までに 7 時間ほどかかるため、それに応じて計画を立ててください。

  * [App Service](../operator/azure-stack-app-service-deploy.md) PaaS サービスを Azure Stack にデプロイします。
  * Azure Stack に[プラン/オファー](../operator/azure-stack-plan-offer-quota-overview.md)を作成します。
  * Azure Stack に[テナント サブスクリプション](../operator/azure-stack-subscribe-plan-provision-vm.md)を作成します。
  * テナント サブスクリプションに Web アプリを作成します。 後で使用できるように新しい Web アプリの URL を書き留めておきます。
  * テナント サブスクリプションに Windows Server 2012 仮想マシンをデプロイします。 このサーバーはビルド サーバーとして、Azure DevOps Services を実行するために使用します。
* 仮想マシン (VM) 用に .NET 3.5 を含んだ Windows Server 2016 イメージを用意します。 この VM は、プライベート ビルド エージェントとして Azure Stack に構築されます。

### <a name="developer-tool-requirements"></a>開発者ツールの要件

* [Azure DevOps サービス ワークスペース](https://docs.microsoft.com/azure/devops/repos/tfvc/create-work-workspaces)を作成します。 サインアップ プロセスによって **MyFirstProject** という名前のプロジェクトが作成されます。
* [Visual Studio 2019 をインストール](https://docs.microsoft.com/visualstudio/install/install-visual-studio)して、[Azure DevOps Services にサインイン](https://www.visualstudio.com/docs/setup-admin/team-services/connect-to-visual-studio-team-services)します。
* プロジェクトに接続し、[ローカルに複製](https://www.visualstudio.com/docs/git/gitquickstart)します。

  > [!Note]
  > Azure Stack 環境には、Windows Server と SQL Server を実行する適切なイメージがシンジケート化されている必要があります。 また、App Service がデプロイされている必要があります。

## <a name="prepare-the-private-azure-pipelines-agent-for-azure-devops-services-integration"></a>Azure DevOps Services 統合のためのプライベート Azure Pipelines エージェントを準備する

### <a name="prerequisites"></a>前提条件

Azure DevOps Services では、サービス プリンシパルを使用して、Azure Resource Manager に対する認証が行われます。 Azure Stack サブスクリプションにリソースをプロビジョニングするためには、Azure DevOps Services に**共同作成者**ロールが必要です。

認証を構成するための要件を次の手順で説明します。

1. サービス プリンシパルを作成するか、既存のサービス プリンシパルを使用します。
2. サービス プリンシパルの認証キーを作成します。
3. サービス プリンシパル名 (SPN) を共同作成者ロールに含めることができるように、ロールベースのアクセス制御を使用して、Azure Stack サブスクリプションを検証します。
4. Azure Stack エンドポイントと SPN 情報を使用して、Azure DevOps Services での新しいサービス定義を作成します。

### <a name="create-a-service-principal"></a>サービス プリンシパルを作成する

[サービス プリンシパルの作成](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications)手順を参照してサービス プリンシパルを作成します。 [アプリケーションの種類] で **[Web アプリ/API]** を選択するか、「[Create an Azure Resource Manager service connection with an existing service principal (既存のサービス プリンシパルで Azure Resource Manager サービス接続を作成する)](https://docs.microsoft.com/vsts/pipelines/library/connect-to-azure?view=vsts#create-an-azure-resource-manager-service-connection-with-an-existing-service-principal)」の記事で説明されている [PowerShell スクリプト](https://github.com/Microsoft/vsts-rm-extensions/blob/master/TaskModules/powershell/Azure/SPNCreation.ps1#L5)を使用します。

 > [!Note]  
 > スクリプトを使用して Azure Stack の Azure Resource Manager エンドポイントを作成する場合は、 **-azureStackManagementURL** パラメーターと **-environmentName** パラメーターを渡す必要があります。 例:  
> `-azureStackManagementURL https://management.local.azurestack.external -environmentName AzureStack`

### <a name="create-an-access-key"></a>アクセス キーを作成する

サービス プリンシパルには、認証用のキーが必要です。 次の手順を使用してキーを生成します。

1. Azure Active Directory の **[アプリの登録]** で、アプリを選択します。

    ![アプリケーションを選択する - Azure Active Directory](media/azure-stack-solution-hybrid-pipeline/000_01.png)

2. **[アプリケーション ID]** の値をメモします。 この値は、Azure DevOps Services でサービス エンドポイントを構成するときに使用します。

    ![アプリケーション ID - Azure Active Directory](media/azure-stack-solution-hybrid-pipeline/000_02.png)

3. 認証キーを生成するには、 **[設定]** を選択します。

    ![アプリ設定を構成する - Azure Active Directory](media/azure-stack-solution-hybrid-pipeline/000_03.png)

4. 認証キーを生成するには、 **[キー]** を選択します。

    ![キー設定を構成する - Azure Active Directory](media/azure-stack-solution-hybrid-pipeline/000_04.png)

5. キーの説明を入力し、キーの期間を設定します。 操作が完了したら、 **[保存]** をクリックします。

    ![キーの説明と期間 - Azure Active Directory](media/azure-stack-solution-hybrid-pipeline/000_05.png)

    キーを保存した後、キーの**値**が表示されます。 この値をコピーしてください。この値を後から取得することはできません。 **キー値**は、アプリとしてサインインする際に**アプリケーション ID** と共に入力します。 アプリで取得できる場所にキー値を格納します。

    ![キー値 - Azure Active Directory](media/azure-stack-solution-hybrid-pipeline/000_06.png)

### <a name="get-the-tenant-id"></a>テナント ID を取得する

サービス エンドポイント構成の一部として、Azure DevOps Services には、Azure Stack スタンプのデプロイ先の AAD ディレクトリに対応する**テナント ID** が必要です。 テナント ID を取得するには次の手順を使用します。

1. **[Azure Active Directory]** を選択します。

    ![テナントの Azure Active Directory](media/azure-stack-solution-hybrid-pipeline/000_07.png)

2. テナント ID を取得するには、Azure AD テナントの **[プロパティ]** を選択します。

    ![テナント プロパティを表示する - Azure Active Directory](media/azure-stack-solution-hybrid-pipeline/000_08.png)

3. **ディレクトリ ID** をコピーします。 この値がテナント ID です。

    ![ディレクトリ ID - Azure Active Directory](media/azure-stack-solution-hybrid-pipeline/000_09.png)

### <a name="grant-the-service-principal-rights-to-deploy-resources-in-the-azure-stack-subscription"></a>Azure Stack サブスクリプション内にリソースをデプロイするサービス プリンシパル権限を付与する

サブスクリプション内のリソースにアクセスするには、アプリをロールに割り当てる必要があります。 アプリに最適なアクセス許可を表すロールを決定します。 利用可能なロールについては、「[RBAC: 組み込みロール](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles)」を参照してください。

スコープは、サブスクリプション、リソース グループ、またはリソースのレベルで設定できます。 アクセス許可は、スコープの下位レベルに継承されます。 たとえば、アプリをリソース グループの閲覧者ロールに追加すると、リソース グループとそのすべてのリソースを読み取ることができます。

1. アプリケーションを割り当てるスコープのレベルに移動します。 たとえば、サブスクリプション スコープでロールを割り当てるには、 **[サブスクリプション]** を選択します。

    ![サブスクリプションを選択する - Azure Stack](media/azure-stack-solution-hybrid-pipeline/000_10.png)

2. **[サブスクリプション]** で [Visual Studio Enterprise] を選択します。

    ![Visual Studio Enterprise - Azure Stack](media/azure-stack-solution-hybrid-pipeline/000_11.png)

3. Visual Studio Enterprise で **[アクセス制御 (IAM)]** を選択します。

4. **[ロールの割り当ての追加]** を選択します。

    ![ロール割り当てを追加する - Azure Stack](media/azure-stack-solution-hybrid-pipeline/000_13.png)

5. **[アクセス許可の追加]** で、アプリに割り当てるロールを選択します。 この例では **[所有者]** ロールです。

    ![所有者ロールのアクセス許可 - Azure Stack](media/azure-stack-solution-hybrid-pipeline/000_14.png)

6. 既定では、Azure Active Directory アプリは使用可能なオプションに表示されません。 目的のアプリを見つけるには、その名前を **[選択]** フィールドに指定して検索する必要があります。 アプリを選びます。

    ![アプリの検索結果 - Azure Stack](media/azure-stack-solution-hybrid-pipeline/000_16.png)

7. **[保存]** を選択して、ロールの割り当てを完了します。 アプリは、そのスコープのロールに割り当てられたユーザーのリストで確認できます。

### <a name="role-based-access-control"></a>ロールベースのアクセス制御

‎Azure のアクセス権は、そのロールベースのアクセス制御 (RBAC) によって詳細に管理することができます。 それぞれの職務を遂行するユーザーに必要なアクセスのレベルを RBAC を使用して制御することができます。 ロールベースのアクセス制御の詳細については、[Azure サブスクリプション リソースへのアクセスの管理](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal?toc=%252fazure%252factive-directory%252ftoc.json)に関するページをご覧ください。

### <a name="azure-devops-services-agent-pools"></a>Azure DevOps Services のエージェント プール

各エージェントを個別に管理するのではなく、エージェント プールにまとめて整理することができます。 エージェント プールでは、そのプール内のすべてのエージェントに対して共有境界が定義されています。 Azure DevOps Services では、エージェント プールの対象範囲は、Azure DevOps Services 組織になります。これは、プロジェクト間でエージェント プールを共有できることを意味します。 エージェント プールの詳細については、[エージェント プールとキューの作成](https://docs.microsoft.com/azure/devops/pipelines/agents/pools-queues?view=vsts)に関するページをご覧ください。

### <a name="add-a-personal-access-token-pat-for-azure-stack"></a>Azure Stack の個人用アクセス トークン (PAT) を追加する

Azure DevOps Services にアクセスするための個人用アクセス トークンを作成します。

1. Azure DevOps Services 組織にサインインし、組織のプロファイル名を選択します。

2. **[セキュリティの管理]** を選択して、トークン作成ページにアクセスします。 

    ![セキュリティを管理する - Azure Stack](media/azure-stack-solution-hybrid-pipeline/000_18.png)

3. **[追加]** をクリックして、新しい個人用アクセス トークンを作成します。

    ![個人用アクセス トークンを追加する - Azure Stack](media/azure-stack-solution-hybrid-pipeline/000_18a.png)

    ![トークンを作成する - Azure Stack](media/azure-stack-solution-hybrid-pipeline/000_18b.png)

4. トークンをコピーします。

    > [!Note]
    > トークン情報は保存しておいてください。 この情報は格納されず、この Web ページから離れると二度と表示されません。

    ![個人用アクセス トークン - Azure Stack](media/azure-stack-solution-hybrid-pipeline/000_19.png)

### <a name="install-the-azure-devops-services-build-agent-on-the-azure-stack-hosted-build-server"></a>Azure Stack でホストされているビルド サーバーに Azure DevOps Services ビルド エージェントをインストールする

1. Azure Stack ホストにデプロイしたビルド サーバーに接続します。
2. ご自身の個人用アクセス トークン (PAT) を使用して、ビルド エージェントをダウンロードし、サービスとしてデプロイし、VM 管理者としてそれを実行します。

    ![ビルド エージェントのダウンロード](media/azure-stack-solution-hybrid-pipeline/010_downloadagent.png)

3. 抽出したビルド エージェントのフォルダーに移動します。 管理者特権でのコマンド プロンプトから **config.cmd** ファイルを実行します。

    ![抽出したビルド エージェント](media/azure-stack-solution-hybrid-pipeline/000_20.png)

    ![ビルド エージェントの登録](media/azure-stack-solution-hybrid-pipeline/000_21.png)

4. config.cmd が完了すると、ビルド エージェントのフォルダーが追加ファイルで更新されます。 抽出された内容を含むフォルダーは、次の例のようになるはずです。

    ![ビルド エージェントのフォルダーの更新](media/azure-stack-solution-hybrid-pipeline/009_token_file.png)

    Azure DevOps Services フォルダー内のエージェントを確認できます。

## <a name="endpoint-creation-permissions"></a>エンドポイント作成のアクセス許可

Visual Studio Online (VSTO) のビルドでは、エンドポイントを作成することにより、Azure サービス アプリを Azure Stack にデプロイすることができます。 Azure DevOps Services がビルド エージェントに接続し、そのエージェントが Azure Stack に接続します。

![VSTO における NorthwindCloud サンプル アプリ](media/azure-stack-solution-hybrid-pipeline/012_securityendpoints.png)

1. VSTO にサインインし、アプリの設定ページに移動します。
2. **[設定]** で、 **[セキュリティ]** を選択します。
3. **[Azure DevOps Services グループ]** で、 **[エンドポイント作成者]** を選択します。

    ![NorthwindCloud のエンドポイント作成者](media/azure-stack-solution-hybrid-pipeline/013_endpoint_creators.png)

4. **[メンバー]** タブで **[追加]** を選択します。

    ![メンバーを追加する](media/azure-stack-solution-hybrid-pipeline/014_members_tab.png)

5. **[ユーザーとグループの追加]** ページで、ユーザー名を入力し、そのユーザーをユーザーのリストから選択します。
6. **[変更の保存]** を選択します。
7. **[Azure DevOps Services グループ]** の一覧で、 **[エンドポイント管理者]** を選択します。

    ![NorthwindCloud のエンドポイント管理者](media/azure-stack-solution-hybrid-pipeline/015_save_endpoint.png)

8. **[メンバー]** タブで **[追加]** を選択します。
9. **[ユーザーとグループの追加]** ページで、ユーザー名を入力し、そのユーザーをユーザーのリストから選択します。
10. **[変更の保存]** を選択します。

これでエンドポイント情報が存在するので、Azure DevOps Services から Azure Stack への接続を使用する準備ができました。 Azure Stack のビルド エージェントでは、Azure DevOps Services から命令を受け取ってから、Azure Stack との通信のためのエンドポイント情報を伝達します。

## <a name="create-an-azure-stack-endpoint"></a>Azure Stack エンドポイントを作成する

### <a name="create-an-endpoint-for-azure-ad-deployments"></a>Azure AD デプロイ用のエンドポイントを作成する

「[Create an Azure Resource Manager service connection with an existing service principal (既存のサービス プリンシパルで Azure Resource Manager サービス接続を作成する)](https://docs.microsoft.com/vsts/pipelines/library/connect-to-azure?view=vsts#create-an-azure-resource-manager-service-connection-with-an-existing-service-principal)」の記事の手順に従い、既存のサービス プリンシパルでサービス接続を作成します。次のマッピングを使用してください。

| EnableAdfsAuthentication | 例 | 説明 |
| --- | --- | --- |
| 接続名 | Azure Stack Azure AD | 接続の名前。 |
| 環境 | AzureStack | 環境の名前。 |
| 環境 URL | `https://management.local.azurestack.external` | 管理エンドポイント。 |
| スコープのレベル | Subscription | 接続のスコープ。 |
| サブスクリプション ID | 65710926-XXXX-4F2A-8FB2-64C63CD2FAE9 | Azure Stack のユーザーのサブスクリプション ID |
| サブスクリプション名 | name@contoso.com | Azure Stack のユーザーのサブスクリプション名。 |
| サービス プリンシパルのクライアント ID | FF74AACF-XXXX-4776-93FC-C63E6E021D59 | この記事で取得したプリンシパル ID ([こちら](azure-stack-solution-pipeline.md#create-a-service-principal)のセクションを参照)。 |
| サービス プリンシパルのキー | THESCRETGOESHERE = | 同じ記事で取得したキー (スクリプトを使用した場合はパスワード)。 |
| テナント ID | D073C21E-XXXX-4AD0-B77E-8364FCA78A94 | 「[テナント ID を取得する](azure-stack-solution-pipeline.md#get-the-tenant-id)」の手順に従って取得したテナント ID。  |
| 接続: | 未確認 | サービス プリンシパルに対する接続の設定を確認します。 |

これでエンドポイントが作成されたので、DevOps から Azure Stack への接続を使用する準備は完了です。 Azure Stack のビルド エージェントでは、DevOps から命令を受け取った後、Azure Stack との通信のためのエンドポイント情報を伝達します。

![ビルド エージェントの Azure AD](media/azure-stack-solution-hybrid-pipeline/016_save_changes.png)

### <a name="create-an-endpoint-for-ad-fs"></a>AD FS のエンドポイントの作成

Azure DevOps の最新の更新プログラムにより、認証に証明書を使用したサービス プリンシパルを使ってサービス接続を作成できるようになります。 この接続は、ID プロバイダーとして AD FS を使用して Azure Stack がデプロイされるときに必要です。 

![ビルド エージェントの AD FS](media/azure-stack-solution-hybrid-pipeline/image06.png)

次のマッピングを使用してサービス接続を作成できます。

| EnableAdfsAuthentication | 例 | 説明 |
| --- | --- | --- |
| 接続名 | Azure Stack ADFS | 接続の名前。 |
| 環境 | AzureStack | 環境の名前。 |
| 環境 URL | `https://management.local.azurestack.external` | 管理エンドポイント。 |
| スコープのレベル | Subscription | 接続のスコープ。 |
| サブスクリプション ID | 65710926-XXXX-4F2A-8FB2-64C63CD2FAE9 | Azure Stack のユーザーのサブスクリプション ID |
| サブスクリプション名 | name@contoso.com | Azure Stack のユーザーのサブスクリプション名。 |
| サービス プリンシパルのクライアント ID | FF74AACF-XXXX-4776-93FC-C63E6E021D59 | AD FS 用に作成したサービス プリンシパルのクライアント ID。 |
| 証明書 | `<certificate>` |  証明書ファイルを PFX から PEM に変換します。 証明書の PEM ファイルの内容をこのフィールドに貼り付けます。 <br> PFX から PEM への変換:<br>`openssl pkcs12 -in file.pfx -out file.pem -nodes -password pass:<password_here>` |
| テナント ID | D073C21E-XXXX-4AD0-B77E-8364FCA78A94 | 「[テナント ID を取得する](azure-stack-solution-pipeline.md#get-the-tenant-id)」の手順に従って取得したテナント ID。 |
| 接続: | 未確認 | サービス プリンシパルに対する接続の設定を確認します。 |

これでエンドポイントが作成されたので、Azure DevOps から Azure Stack への接続を使用する準備は完了です。 Azure Stack のビルド エージェントでは、Azure DevOps から命令を受け取った後、Azure Stack との通信のためのエンドポイント情報を伝達します。

> [!Note]
> Azure Resource Manager エンドポイントがインターネットに公開されていない場合、接続の検証は失敗します。 これは想定されているため、簡単なタスクのリリース パイプラインを作成することで接続を検証できます。 

## <a name="develop-your-application-build"></a>アプリケーション ビルドの開発

ソリューションのこの部分では、次のことを行います。

* Azure DevOps Services プロジェクトにコードを追加する。
* 自己完結型 Web アプリ デプロイを作成する。
* 継続的配置プロセスを構成する。

> [!Note]
 > Azure Stack 環境には、Windows Server と SQL Server を実行する適切なイメージがシンジケート化されている必要があります。 また、App Service がデプロイされている必要があります。 Azure Stack オペレーターの要件については、App Service のドキュメントの「前提条件」セクションをご覧ください。

ハイブリッド CI/CD は、アプリケーション コードとインフラストラクチャ コードの両方に適用できます。 Azure DevOps Services から [Azure Resource Manager テンプレート](https://azure.microsoft.com/resources/templates/)を Web アプリ コードのように使用して、両方のクラウドにデプロイします。

### <a name="add-code-to-an-azure-devops-services-project"></a>Azure DevOps Services プロジェクトにコードを追加する

1. Azure Stack でのプロジェクト作成権限がある組織で、Azure DevOps Services にサインインします。 次のキャプチャ画面は、HybridCICD プロジェクトへの接続方法を示しています。

    ![プロジェクトに接続する - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/017_connect_to_project.png)

2. 既定の Web アプリを作成して開くことで、**リポジトリを複製**します。

    ![リポジトリをクローンする - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/018_link_arm.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>両方のクラウドで App Services の自己完結型 Web アプリ デプロイを作成する

1. **WebApplication.csproj** ファイルを編集します。`Runtimeidentifier` を選択してから、`win10-x64.` を追加します。詳細については、[自己完結型のデプロイ](https://docs.microsoft.com/dotnet/core/deploying/#self-contained-deployments-scd)に関するドキュメントを参照してください。

    ![Runtimeidentifier の構成](media/azure-stack-solution-hybrid-pipeline/019_runtimeidentifer.png)

2. チーム エクスプローラーを使用して、コードを Azure DevOps Services にチェックインします。

3. アプリケーション コードが Azure DevOps Services にチェックインされたことを確認します。

### <a name="create-the-build-pipeline"></a>ビルド パイプラインを作成する

1. ビルド パイプラインを作成できる組織で、Azure DevOps Services にサインインします。

2. プロジェクトの **[Build Web Application]\(Web アプリケーションのビルド\)** ページに移動します。

3. **[引数]** に **-r win10-x64** コードを追加します。 この手順は、.NET Core を使用して自己完結型のデプロイをトリガーするために必要です。

    ![ビルド パイプラインの引数の追加](media/azure-stack-solution-hybrid-pipeline/020_publish_additions.png)

4. ビルドを実行します。 [自己完結型のデプロイ ビルド](https://docs.microsoft.com/dotnet/core/deploying/#self-contained-deployments-scd)のプロセスにより、Azure および Azure Stack 上で実行できる成果物が発行されます。

### <a name="use-an-azure-hosted-build-agent"></a>Azure ホスト ビルド エージェントを使用する

Web アプリをビルドしてデプロイする場合、Azure DevOps Services でホスト ビルド エージェントを使用すると便利です。 エージェントのメンテナンスやアップグレードは Microsoft Azure によって自動的に実施されるので、中断のない継続的な開発サイクルが実現します。

### <a name="configure-the-continuous-deployment-cd-process"></a>継続的配置 (CD) プロセスを構成する

Azure DevOps Services および Team Foundation Server (TFS) が提供するパイプラインは自由に構成でき、管理性にも優れ、開発、ステージング、品質保証 (QA)、運用など、さまざまな環境へのリリースに使用できます。 このプロセスの一環として、アプリケーション ライフ サイクルの特定のステージで承認を要求することもできます。

### <a name="create-release-pipeline"></a>リリース パイプラインを作成する

リリース パイプラインの作成は、アプリ ビルド プロセスの最後の手順です。 このリリース パイプラインを使用してリリースを作成し、ビルドをデプロイします。

1. Azure DevOps Services サービスにサインインし、プロジェクトの **Azure Pipelines** に移動します。
2. **[リリース]** タブで **\[ + ]** を選択し、 **[リリース定義の作成]** を選択します。

   ![リリース パイプラインを作成する - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/021a_releasedef.png)

3. **[テンプレートの選択]** ページで、 **[Azure App Service の配置]** を選んでから **[適用]** を選択します。

    ![テンプレートを適用する - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/102.png)

4. **[成果物の追加]** ページの **[ソース (ビルド定義)]** プルダウン メニューから、Azure Cloud ビルド アプリを選択します。

    ![成果物を追加する - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/103.png)

5. **[パイプライン]** タブで、**1 フェーズ**、**1 タスク**のリンクを選択し、**環境のタスクを表示**します。

    ![パイプライン ビューのタスク - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/104.png)

6. **[タスク]** タブで **[環境名]** に「Azure」と入力し、 **[Azure サブスクリプション]** ボックスの一覧から [AzureCloud Traders-Web EP] を選択します。

    ![サービス環境変数を設定する - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/105.png)

7. **Azure App Service の名前**として「northwindtraders」と入力します。次のキャプチャ画面を参照してください。

    ![App Service の名前 - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/106.png)

8. [エージェント フェーズ] で、 **[エージェント キュー]** ボックスの一覧から **[Hosted VS2017]** を選択します。

    ![ホステッド エージェント - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/107.png)

9. **[Azure App Service 配置]** で、環境に対して有効な**パッケージまたはフォルダー**を選択します。

    ![パッケージまたはフォルダーを選択する - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/108.png)

10. **[ファイルまたはフォルダーの選択]** ページで、フォルダーの場所に対して **[OK]** を選択します。

    ![ファイルまたはフォルダーを選択する - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/109.png)

11. すべての変更を保存し、 **[パイプライン]** に戻ります。

    ![変更を保存する - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/110.png)

12. **[パイプライン]** タブで **[成果物の追加]** を選択し、 **[ソース (ビルド定義)]** ボックスの一覧から **[NorthwindCloud Traders-Vessel]** を選択します。

    ![新しい成果物を追加する - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/111.png)

13. **[テンプレートの選択]** ページで、別の環境を追加します。 **[Azure App Service の配置]** を選択し、 **[適用]** を選択します。

    ![テンプレートを選択する - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/112.png)

14. **[環境名]** として「Azure Stack」と入力します。

    ![環境名 - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/113.png)

15. **[タスク]** タブで、[Azure Stack] を見つけて選択します。

    ![Azure Stack 環境 - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/114.png)

16. **[Azure サブスクリプション]** ボックスの一覧から、Azure Stack のエンドポイントに使用する [AzureStack Traders-Vessel EP] を選択します。

    ![Azure サブスクリプションのドロップダウン - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/115.png)

17. **[App Service の名前]** として、Azure Stack Web アプリの名前を入力します。

    ![App Service の名前 - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/116.png)

18. **[エージェントの選択]** で、 **[エージェント キュー]** ボックスの一覧から [AzureStack -bDouglas Fir] を選択します。

    ![エージェントを選ぶ - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/117.png)

19. **[Azure App Service 配置]** で、環境に対して有効な**パッケージまたはフォルダー**を選択します。 **[ファイルまたはフォルダーの選択]** で、フォルダーの**保存先**に対して **[OK]** を選択します。

    ![パッケージまたはフォルダーを選ぶ - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/118.png)

    ![場所を承認する - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/119.png)

20. **[変数]** タブで、**VSTS_ARM_REST_IGNORE_SSL_ERRORS** という名前の変数を見つけます。 変数の値を **[true]** に設定し、そのスコープを **[Azure Stack]** に設定します。

    ![変数を構成する Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/120.png)

21. **[パイプライン]** タブで、NorthwindCloud Traders-Web の成果物に使用する **[継続的配置トリガー]** アイコンを選択し、 **[継続的配置トリガー]** を **[有効]** に設定します。  "NorthwindCloud Traders-Vessel" の成果物についても同じ操作を行います。

    ![継続的デプロイ トリガーを設定する - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/121.png)

22. Azure Stack 環境で**配置前**条件アイコンを選択し、トリガーを**リリース後**に設定します。

    ![デプロイ前条件のトリガーを設定する - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/122.png)

23. すべての変更を保存します。

> [!Note]
> リリース タスクの一部の設定は、テンプレートからリリース パイプラインを作成したときに、[環境変数](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts#custom-variables)として自動的に定義されている可能性があります。 これらの設定は、タスクの設定では変更できません。 ただし親環境の項目では、これらの設定を編集することができます。

## <a name="create-a-release"></a>リリースを作成する

これで、リリース パイプラインの変更が完了しました。次はデプロイを開始します。 デプロイを開始するには、リリース パイプラインからリリースを作成します。 リリースは自動的に作成される場合があります。たとえば、継続的デプロイ トリガーがリリース パイプラインで設定されている場合です。 このトリガーを設定した場合、ソース コードを変更すると、新しいビルドが開始されてから新しいリリースが開始されます。 しかし、このセクションでは、新しいリリースを手動で作成します。

1. **[パイプライン]** タブで、 **[リリース]** ドロップダウン リストを開き、 **[リリースの作成]** を選択します。

    ![リリースを作成する - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/200.png)

2. リリースの説明を入力し、正しい成果物が選択されていることを確認してから、 **[作成]** を選択します。 しばらくすると、新しいリリースが作成されたことを示すバナーが表示され、そのリリース名がリンクとして表示されます。 リンクを選択してリリース概要ページを表示します。

    ![リリース作成バナー - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/201.png)

3. リリースの概要ページに、リリースに関する詳細が表示されます。 次の "Release-2" のキャプチャ画面では、 **[環境]** セクションに、Azure については **[デプロイ状態]** が "進行中" と表示され、Azure Stack の状態は "成功" と表示されています。 Azure 環境のデプロイ状態が "成功" に変わると、リリースの承認準備が完了したことを示すバナーが表示されます。 デプロイが保留中か、失敗した場合は、青い **(i)** 情報アイコンが表示されます。 このアイコンにマウス ポインターを合わせると、遅延または失敗の理由を示すポップアップが表示されます。

    ![リリース概要ページ - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/202.png)

リリースの一覧など、その他のビューにも、承認待ちであることを示すアイコンが表示されます。 このアイコンのポップアップには、環境の名前のほか、デプロイに関連するさらに詳しい情報が表示されます。 管理者は、リリースの全体的な進行状況を見て、どのリリースが承認待ちになっているかを簡単に確認できます。

### <a name="monitor-and-track-deployments"></a>デプロイを監視および追跡する

このセクションでは、すべてのデプロイを監視し、追跡する方法について説明します。 ここでは、2 つの Azure App Services Web サイトをデプロイするためのリリースを例に取り上げています。

1. "Release-2" の概要ページで、 **[ログ]** を選択します。 デプロイ時には、エージェントからリアルタイムのログがこのページに表示されます。 左側のウィンドウには、デプロイに伴う各操作の状態が環境ごとに表示されます。

    デプロイ前またはデプロイ後の承認の **[アクション]** 列で人アイコンを選択し、デプロイを承認 (または拒否) したユーザーと、そのユーザーが入力したメッセージを表示します。

2. デプロイ完了後、ログ ファイル全体が右側のウィンドウに表示されます。 左側のウィンドウでいずれかの**ステップ**を選択し、"ジョブの初期化" など、単一ステップのログ ファイルを表示します。 ログを個別に表示できることから、デプロイ全体を構成する各要素の追跡とデバッグがしやすくなっています。 特定のステップのログ ファイルを**保存**したり、**すべてのログを zip としてダウンロード**したりすることができます。

    ![リリース ログ - Azure DevOps Services](media/azure-stack-solution-hybrid-pipeline/203.png)

3. **[概要]** タブを開くと、そのリリースの概要情報が表示されます。 ビルドとそのデプロイ先となった環境の詳細、デプロイ状態など、リリースに関する各種の情報がこのビューに表示されます。

4. 環境リンク (**Azure** または **Azure Stack**) を選択すると、特定の環境に対する既存のデプロイと保留中のデプロイについての情報が表示されます。 これらのビューを使って簡単に、同じビルドが両方の環境にデプロイされていることを確認できます。

5. **デプロイされた運用アプリ**をブラウザーで開きます。 たとえば、Azure App Services Web サイトの URL `https://[your-app-name].azurewebsites.net` を開きます。

## <a name="next-steps"></a>次の手順

* Azure のクラウド パターンの詳細については、「[Cloud Design Pattern (クラウド設計パターン)](https://docs.microsoft.com/azure/architecture/patterns)」を参照してください。
