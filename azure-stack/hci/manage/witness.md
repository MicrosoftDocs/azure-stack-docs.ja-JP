---
title: クラスター監視のセットアップ
description: クラスター監視のセットアップ方法を学習します
author: v-dasis
ms.topic: how-to
ms.date: 02/17/2021
ms.author: v-dasis
ms.reviewer: JasonGerend
ms.openlocfilehash: 32d0f717e987d757f5315cfe048c75300ae9c776
ms.sourcegitcommit: 4c97ed2caf054ebeefa94da1f07cfb6be5929aac
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/18/2021
ms.locfileid: "100647961"
---
# <a name="set-up-a-cluster-witness"></a>クラスター監視のセットアップ

> 適用対象: Azure Stack HCI バージョン 20H2、Windows Server 2019

監視リソースのセットアップはすべてのクラスターで強く推奨され、クラスターを作成した直後にセットアップする必要があります。 2 ノード クラスターでは、どちらかのサーバーがオフラインになったときに他のノードも使用できなくなることがないように監視が必要です。 3 つ以上のノードのクラスターでは、2 台のサーバーが障害が発生するかそれらがオフラインになったときに備えて監視が必要です。  

SMB ファイル共有を監視として使用するか、Azure クラウド監視を使用することができます。 クラスター内のすべてのサーバー ノードに信頼性の高いインターネット接続がある場合は、Azure クラウド監視が推奨されます。 この記事では、クラウド監視の作成について説明します。

## <a name="before-you-begin"></a>始める前に

クラウド監視を作成する前に、Azure アカウントとサブスクリプションが必要であり、Azure Stack HCI クラスターを Azure に登録しておく必要があります。 詳細については、次の記事を参照してください。

- [Azure アカウントを作成する](https://docs.microsoft.com/dotnet/azure/create-azure-account)
- 該当する場合は、[追加の Azure サブスクリプションを作成](https://docs.microsoft.com/azure/cost-management-billing/manage/create-subscription)します
- [Azure Stack HCI を Azure に接続する](../deploy/register-with-azure.md)

ファイル共有監視の場合、ファイル サーバーに関する要件があります。 詳細については、[システム要件](../concepts/system-requirements.md)に関するページを参照してください。

## <a name="create-an-azure-storage-account"></a>Azure のストレージ アカウントの作成

このセクションでは、Azure ストレージ アカウントを作成する方法について説明します。 このアカウントは、特定のクラスターの仲裁に使用される Azure Blob ファイルを格納するために使用されます。 同じ Azure ストレージ アカウントを使用して、複数のクラスター用のクラウド監視を構成することができます。

1. [Azure portal](https://portal.azure.com) にサインインします。
1. Azure portal のホームのメニューで、 **[Azure サービス]** の **[ストレージ アカウント]** を選択します。 このアイコンがない場合は、 **[リソースの作成]** を選択して、最初に "*ストレージ アカウント*" リソースを作成します。

    :::image type="content" source="media/witness/cloud-witness-home.png" alt-text="Azure portal ホームの画面" lightbox="media/witness/cloud-witness-home.png":::

1. **[ストレージ アカウント]** ページで、 **[新規]** を選択します。

    :::image type="content" source="media/witness/cloud-witness-create.png" alt-text="Azure の新しいストレージ アカウント" lightbox="media/witness/cloud-witness-create.png":::

1. **[ストレージ アカウントの作成]** ページで、次の手順のようにします。
    1. ストレージ アカウントを適用する Azure の **[サブスクリプション]** を選択します。
    1. ストレージ アカウントを適用する Azure の **[リソース グループ]** を選択します。
    1. **[ストレージ アカウント名]** を入力します。
    <br>ストレージ アカウント名の長さは 3 ～ 24 文字で、数字と小文字のみを使用できます。 この名前は、Azure 内でも一意である必要があります。
    1. 物理的に自分に最も近い **[場所]** を選択します。
    1. **[パフォーマンス]** には **[Standard]** を選択します
    1. **[アカウントの種類]** で **ストレージ汎用** を選択します。
    1. **[レプリケーション]** には **[ローカル冗長ストレージ (LRS)]** を選択します。
    1. 完了したら、 **[確認および作成]** をクリックします。

    :::image type="content" source="media/witness/cloud-witness-create-storage-account.png" alt-text="Azure のストレージ アカウントの作成" lightbox="media/witness/cloud-witness-create-storage-account.png":::

1. ストレージ アカウントが検証に合格することを確認した後、アカウントの設定を確認します。 完了したら、 **[作成]** をクリックします。

    :::image type="content" source="media/witness/cloud-witness-validation.png" alt-text="Azure のストレージ アカウント検証" lightbox="media/witness/cloud-witness-validation.png":::

1. Azure でアカウントのデプロイが行われるまでに数秒かかることがあります。 デプロイが完了したら、 **[リソースに移動]** をクリックします。

    :::image type="content" source="media/witness/cloud-witness-deployment.png" alt-text="Azure のストレージ アカウントのデプロイ" lightbox="media/witness/cloud-witness-deployment.png":::

## <a name="copy-the-access-key-and-endpoint-url"></a>アクセス キーとエンドポイント URL をコピーする

Azure ストレージ アカウントを作成すると、プロセスによって自動的に 2 つのアクセスキー、プライマリ キー (key1) とセカンダリ キー (key2) が生成されます。 クラウド監視を初めて作成するときは、**key1** が使用されます。 エンドポイント URL も自動的に生成されます。

Azure クラウド監視のストレージには BLOB ファイルが使用され、エンドポイントは *storage_account_name.blob.core.windows.net* という形式で生成されます。 

> [!NOTE]  
> Azure クラウド監視と Azure BLOB サービスとの通信の確立には、HTTPS (既定のポートは 443) が使用されます。 HTTPS ポートがアクセス可能であることを確認します。

### <a name="copy-the-account-name-and-access-key"></a>アカウント名とアクセス キーをコピーする

1. Azure portal の **[設定]** で **[アクセス キー]** を選択します。
1. **[Show keys]\(キーの表示\)** を選択して、キーの情報を表示します。
1. **[ストレージ アカウント名]** および **[key1]** フィールドの右側にあるコピーと貼り付けアイコンをクリックし、各テキスト文字列をメモ帳などのテキスト エディターに貼り付けます。

    :::image type="content" source="media/witness/cloud-witness-access-keys.png" alt-text="Azure ストレージ アカウントのアクセス キー" lightbox="media/witness/cloud-witness-access-keys.png":::

### <a name="copy-the-endpoint-url-optional"></a>エンドポイント URL をコピーする (省略可能)

エンドポイント URL は省略可能であり、クラウド監視には必ずしも必要ではありません。

1. Azure portal で **[プロパティ]** を選択します。
1. **[Show keys]\(キーの表示\)** を選択して、エンドポイントの情報を表示します。
1. **[Blob service]** で、 **[Blob service]** フィールドの右側にあるコピーと貼り付けアイコンをクリックし、テキスト文字列をメモ帳などのテキスト エディターに貼り付けます。

    :::image type="content" source="media/witness/cloud-witness-blob-service.png" alt-text="Azure Blob エンドポイント" lightbox="media/witness/cloud-witness-blob-service.png":::

## <a name="create-a-cloud-witness-using-windows-admin-center"></a>Windows Admin Center を使用してクラウド監視を作成する

これで、Windows Admin Center を使用してクラスター用の監視インスタンスを作成する準備ができました。

1. Windows Admin Center で、上部のドロップダウン矢印から **[Cluster Manager]\(クラスター マネージャー\)** を選択します。
1. **[Cluster connections]\(クラスター接続\)** の下で、クラスターを選択します。
1. **[ツール]** の **[設定]** を選択します。
1. 右側のペインで **[Witness]\(監視\)** を選択します。
1. **[Witness type]\(監視の種類\)** ボックスの一覧で、次のいずれかを選択します。
      - **[Cloud witness]\(クラウド監視\)** - 前に説明した Azure ストレージ アカウント名、アクセス キー、エンドポイント URL を入力します
      - **[ファイル共有監視]** - ファイル共有パス "(//server/share)" を入力します
1. クラウド監視の場合、次のフィールドには、前にコピーしたテキスト文字列を貼り付けます。
    1. **Azure ストレージ アカウント名**
    1. **Azure ストレージ アクセス キー**
    1. **Azure サービス エンドポイント**

    :::image type="content" source="media/witness/cloud-witness-1.png" alt-text="クラウド監視のアクセス キー" lightbox="media/witness/cloud-witness-1.png":::

1. 完了したら、 **[保存]** をクリックします。 情報が Azure に伝えられるまで、少し時間がかかることがあります。

> [!NOTE]
> 3 番目のオプションである **[ディスク監視]** は、ストレッチ クラスターでの使用に適していません。

## <a name="create-a-cloud-witness-using-windows-powershell"></a>Windows PowerShell を使用してクラウド監視を作成する

または、PowerShell を使用してクラスター用の監視インスタンスを作成することもできます。

Azure クラウド監視を作成するには、次のコマンドレットを使用します。 前に説明したように、Azure ストレージ アカウント名とアクセス キー情報を入力します。

```powershell
Set-ClusterQuorum –Cluster "Cluster1" -CloudWitness -AccountName "AzureStorageAccountName" -AccessKey "AzureStorageAccountAccessKey"
```

ファイル共有監視を作成するには、次のコマンドレットを使用します。 ファイル サーバー共有のパスを入力します。

```powershell
Set-ClusterQuorum -FileShareWitness "\\fileserver\share" -Credential (Get-Credential)
```

## <a name="next-steps"></a>次のステップ

- クラスター クォーラムの詳細については、「[Azure Stack HCI のクラスターとプールのクォーラムについて](../concepts/quorum.md)」を参照してください。

- Azure ストレージ アカウントの作成と管理の詳細については、「[ストレージ アカウントを作成する](https://docs.microsoft.com/azure/storage/common/storage-account-create)」を参照してください。