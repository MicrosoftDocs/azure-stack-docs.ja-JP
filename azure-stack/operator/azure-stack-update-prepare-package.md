---
title: Azure Stack Hub で更新プログラム パッケージを準備する
description: Azure Stack Hub で更新プログラム パッケージを準備する方法について説明します。
author: sethmanheim
ms.topic: how-to
ms.date: 03/08/2021
ms.author: sethm
ms.lastreviewed: 03/10/2021
ms.reviewer: sranthar
ms.openlocfilehash: cdc0968519dc5ac50153fb5a14faaafd3b048843
ms.sourcegitcommit: e432e7f0a790bd6419987cbb5c5f3811e2e7a4a2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/09/2021
ms.locfileid: "102515824"
---
# <a name="prepare-an-azure-stack-hub-update-package"></a>Azure Stack Hub 更新プログラム パッケージを準備する

この記事では、Azure Stack Hub 環境を更新するために使用できるように Azure Stack Hub 更新パッケージを準備することの概要を説明します。 このプロセスは次のステップで構成されます:

- [更新プログラム パッケージをダウンロードします](#download-the-update-package)。
- [Azure Stack Hub 管理者ポータルを使用して、Azure Stack Hub 環境に更新プログラム パッケージをインポートします](#import-and-install-updates)。

自動更新エンドポイントに接続できるシステムでは、Azure Stack Hub のソフトウェア更新プログラムと修正プログラムが自動的にダウンロードされ、準備されます。 接続できないシステムと相手先ブランド供給 (OEM) からの更新プログラムの場合は、この記事の説明に従って更新プログラム パッケージを準備する必要があります。  

次の表は、更新プログラム パッケージが手動による準備を必要とするタイミングと、それらが自動的に準備されるタイミングを示しています。

| 更新の種類 | 接続 | 必要な操作 |
| --- | --- | --- |
| Azure Stack Hub ソフトウェア更新プログラム | 接続中 | 更新が適用されると、更新プログラムが自動的にダウンロードされ、準備されます。 |
| Azure Stack Hub 修正プログラム | 接続中 | 更新が適用されると、更新プログラムが自動的にダウンロードされ、準備されます。 |
| OEM パッケージの更新プログラム | 接続中 | 更新プログラム パッケージを準備する必要があります。 この記事ではその手順を説明します。 |
| Azure Stack Hub ソフトウェア更新プログラム | 切断されているか、または弱い接続 | 更新プログラム パッケージを準備する必要があります。 この記事ではその手順を説明します。 |
| Azure Stack Hub 修正プログラム | 切断されているか、または弱い接続 | 更新プログラム パッケージを準備する必要があります。 この記事ではその手順を説明します。 |
| OEM パッケージの更新プログラム | 切断されているか、または弱い接続 | 更新プログラム パッケージを準備する必要があります。 この記事ではその手順を説明します。 |

## <a name="download-the-update-package"></a>更新プログラム パッケージをダウンロードする

接続されているシステムに対する Azure Stack Hub の更新プログラムおよび修正プログラムの更新プログラム パッケージは、ポータルの更新ブレードから入手できます。 OEM パッケージを更新する場合、または切断されたシステムをサポートしている場合、パッケージをダウンロードして、お使いの Azure Stack Hub インスタンスにアクセスできる場所にパッケージを移動します。 また、接続が途切れがちなシステムを実行している場合、パッケージをダウンロードしてから、アクセス可能な場所にアップロードすることが必要になる場合もあります。

パッケージの内容を確認します。 通常、更新プログラム パッケージは、次のファイルで構成されます。

- **自己解凍形式の \<PackageName>.zip ファイル**。 このファイルには、更新プログラムのペイロードが含まれています。
- **Metadata.xml ファイル**。 このファイルには、更新プログラムに関する重要な情報 (発行元、名前、前提条件、サイズ、サポート パス URL など) が含まれます。

### <a name="automatic-download-and-preparation-for-update-packages"></a>更新プログラム パッケージの自動ダウンロードと準備

Azure Stack Hub のソフトウェア更新プログラムと修正プログラムは、**Azure Stack Hub 自動更新エンドポイント**  (`https://*.azureedge.net` および `https://aka.ms/azurestackautomaticupdate`) に接続するシステムに対して自動的に準備されます。 **Azure Stack Hub 自動更新エンドポイント** への接続の設定の詳細については、[Azure Stack Hub へのファイアウォールの統合](./azure-stack-integrate-endpoints.md#ports-and-urls-outbound)に関するページで説明されている **修正プログラムと更新** エンドポイントを参照してください。

### <a name="where-to-download-azure-stack-hub-update-packages"></a>Azure Stack Hub 更新プログラム パッケージをダウンロードする場所

[フル更新と高速更新](./azure-stack-updates.md#update-package-types)用の Azure Stack Hub 更新プログラムは、セキュリティで保護された Azure エンドポイントでホストされます。 更新プログラムが利用可能になると、接続されたインスタンスを担当する Azure Stack Hub オペレーターには、[管理者ポータルで、Azure Stack Hub の更新プログラムが自動的に表示](#automatic-download-and-preparation-for-update-packages)されます。 接続されていないシステム、またはインターネット接続が弱いシステムの場合は、[Azure Stack Hub 更新プログラムのダウンローダー ツール](https://aka.ms/azurestackupdatedownload)を使用して更新プログラム パッケージをダウンロードできます。 Azure Stack Hub ソフトウェアの更新プログラム パッケージには、Azure Stack Hub サービスの更新プログラム以外に、お使いの Azure Stack Hub のスケール ユニットのオペレーティング システムの更新プログラムが含まれている場合があります。

>[!NOTE]
>更新プログラム パッケージとそのコンテンツ (バイナリ、PowerShell スクリプトなど) は、Microsoft が所有する証明書で署名されています。 パッケージを改ざんすると、この署名は無効になります。

### <a name="where-to-download-azure-stack-hub-hotfix-packages"></a>Azure Stack Hub の修正プログラム パッケージをダウンロードする場所

[Azure Stack Hub 修正プログラム](./azure-stack-updates.md#update-package-types)のパッケージは、Azure Stack Hub 更新プログラムと同じセキュリティで保護された Azure エンドポイントでホストされます。 それらが利用可能になると、接続されたインスタンスを担当する Azure Stack Hub オペレーターには、[管理者ポータルで、Azure Stack Hub の更新プログラムが自動的に表示](#automatic-download-and-preparation-for-update-packages)されます。 各修正プログラムの KB 記事の埋め込みリンクを使用して、それらをダウンロードできます。 修正プログラムの KB 記事へのリンクは、お使いの Azure Stack Hub のバージョンに対応するリリース ノートでも見つかります。

### <a name="where-to-download-oem-update-packages"></a>OEM 更新プログラム パッケージをダウンロードする場所

また、OEM ベンダーからドライバーやファームウェアの更新プログラムなど、更新プログラムがリリースされることもあります。 これらの更新プログラムは、ハードウェア ベンダーから個別の [OEM パッケージ更新プログラム](./azure-stack-updates.md#update-package-types)として提供されますが、それらは引き続き、Microsoft の更新プログラム パッケージと同じようにインポート、インストール、および管理されます。 ベンダーの連絡先リンクの一覧については、[Azure Stack Hub に OEM の更新プログラムを適用する](./azure-stack-update-oem.md#oem-contact-information)方法に関する記事を参照してください。

## <a name="import-and-install-updates"></a>更新プログラムをインポートしてインストールする

次の手順は、管理者ポータルで更新プログラム パッケージをインポートしてインストールする方法を示しています。

> [!IMPORTANT]  
> メンテナンス操作についてユーザーに通知し、通常のメンテナンス期間をできるだけ勤務時間外にスケジュールします。 メンテナンス操作は、ユーザーのワークロードとポータル操作の両方に影響を及ぼす可能性があります。

1. 管理者ポータルで、 **[すべてのサービス]** を選択します。 次に、 **[ストレージ]** カテゴリで、 **[ストレージ アカウント]** を選択します。 または、フィルター ボックスで「**ストレージ アカウント**」の入力を開始して選択します。

    [![Azure Stack Hub の更新プログラム](./media/azure-stack-update-prepare-package/select-storage-small.png)](./media/azure-stack-update-prepare-package/select-storage.png#lightbox)

2. フィルター ボックスに「**更新**」と入力し、**updateadminaccount** ストレージ アカウントを選択します。

3. **[すべてのサービス]** の **[Essentials]** または **[Blob service]** で、 **[コンテナー]** を選択します。

    [![Azure Stack Hub の更新プログラム ‐ BLOB](./media/azure-stack-update-prepare-package/select-containers-small.png)](./media/azure-stack-update-prepare-package/select-containers.png#lightbox)

4. **[コンテナー]** で、 **[+ コンテナー]** を選択し、コンテナーを作成します。 名前 (例: **update-2102**) を入力し、 **[作成]** を選択します。

    [![Azure Stack Hub の更新プログラム - コンテナー](./media/azure-stack-update-prepare-package/new-container-small.png)](./media/azure-stack-update-prepare-package/new-container.png#lightbox)

5. コンテナーが作成されたら、コンテナー名を選択した後、 **[アップロード]** を選択してパッケージ ファイルをコンテナーにアップロードします。

    [![Azure Stack Hub の更新プログラム ‐ アップロード](./media/azure-stack-update-prepare-package/upload-package-small.png)](./media/azure-stack-update-prepare-package/upload-package.png#lightbox)

6. **[BLOB のアップロード]** で、フォルダー アイコンを選択し、更新プログラム パッケージの .zip ファイルを参照して、エクスプローラー ウィンドウで **[開く]** を選択します。

7. **[BLOB のアップロード]** で、 **[アップロード]** を選択します。

    ![Azure Stack Hub の更新プログラム - BLOB のアップロード](./media/azure-stack-update-prepare-package/upload-blob.png)

8. 更新プログラム パッケージの **Metadata.xml** ファイルおよび追加の .zip ファイルに対して、ステップ 6 と 7 を繰り返します。 **Supplemental Notice.txt** ファイルが含まれている場合、そのファイルはインポートしないでください。

9. 終了したら、通知を確認できます (ポータルの右上隅にあるベル アイコンを選択)。 アップロードの完了が通知に示されている必要があります。

10. ダッシュボードの **[更新]** ブレードに戻ります。 ブレードに更新プログラムが使用可能であることが示されているはずです。 これは、更新が正常に準備されたことを示します。 新しく追加された更新プログラム パッケージを確認するには、ブレードを選択します。

11. 更新プログラムをインストールするには、 **[準備完了]** とマークされているパッケージを選択し、 **[今すぐ更新]** を選択します。

12. 更新プログラム パッケージのインストールを選択したら、 **[Update run details]\(更新実行の詳細\)** 領域でその状態を確認できます。 ここから **[Download summary]\(サマリーのダウンロード\)** を選択して、ログ ファイルをダウンロードすることもできます。 更新実行のログは、試行が終了してから 6 か月間は利用できます。

13. 更新プログラムが完了すると、更新された Azure Stack Hub のバージョンが **[更新]** ブレードに表示されます。

更新プログラムは Azure Stack Hub にインストール後、手動でストレージ アカウントから削除できます。 Azure Stack Hub は古い更新プログラム パッケージを定期的にチェックし、それらをストレージから削除します。 Azure Stack Hub では古いパッケージの削除に最大で 2 週間かかる場合があります。

## <a name="next-steps"></a>次のステップ

[更新プログラムを適用する](azure-stack-apply-updates.md)
