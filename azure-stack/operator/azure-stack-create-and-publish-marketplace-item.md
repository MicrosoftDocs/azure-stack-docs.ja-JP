---
title: Azure Stack Hub で Marketplace 項目を作成して発行する
description: Azure Stack Hub で Marketplace アイテムを作成して発行する方法について説明します。
author: sethmanheim
ms.topic: article
ms.date: 03/09/2021
ms.author: sethm
ms.reviewer: avishwan
ms.lastreviewed: 11/16/2020
ms.openlocfilehash: deb0c21a6e352d2e2fa4fdab3cab718512746591
ms.sourcegitcommit: 4f1d22747c02ae280609174496933fca8c04a6cf
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/10/2021
ms.locfileid: "102606449"
---
# <a name="create-and-publish-a-custom-azure-stack-hub-marketplace-item"></a>Azure Stack Hub でカスタム Marketplace アイテムを作成して発行する

Azure Stack Hub Marketplace に発行されるすべてのアイテムでは、Azure ギャラリー パッケージ (.azpkg) 形式を使用します。 *Azure Gallery Packager* ツールを使用すると、Azure Stack Hub の Marketplace にアップロードしてユーザーが後からダウンロードできる、カスタム Azure ギャラリー パッケージを作成できます。 デプロイ プロセスでは、Azure Resource Manager テンプレートを使用します。

## <a name="marketplace-items"></a>Marketplace アイテム

この記事の例では、Windows または Linux の種類で単一の VM Marketplace オファーを作成する方法について説明します。

### <a name="prerequisites"></a>前提条件

VM Marketplace アイテムを作成する前に、次の手順を実行します。

1. [VM イメージの Azure Stack Hub への追加](azure-stack-add-vm-image.md)に関するページの手順に従って、カスタム VM イメージを Azure Stack Hub ポータルにアップロードします。
2. この記事の手順に従ってイメージをパッケージ化し (.azpkg を作成)、それを Azure Stack Hub の Marketplace にアップロードします。

## <a name="create-a-marketplace-item"></a>Marketplace アイテムの作成

カスタム Marketplace アイテムを作成するには、次の手順のようにします。

1. [Azure Gallery Packager ツール](https://aka.ms/azsmarketplaceitem)をダウンロードします。

   :::image type="content" source="media/azure-stack-create-and-publish-marketplace-item/samples-tool.png" alt-text="Gallery Packager":::

2. このツールには、.azpk 形式のサンプル パッケージが含まれているため、最初に抽出する必要があります。 ファイル拡張子の名前を ".azpk" から ".zip" に変更することも、任意のアーカイバー ツールを使用することもできます。

   :::image type="content" source="media/azure-stack-create-and-publish-marketplace-item/sample-packages.png" alt-text="サンプル パッケージ":::

3. 抽出すると、.zip ファイルに使用可能な Linux または Windows の Azure Resource Manager テンプレートが含まれます。 事前に作成されている Resource Manager テンプレートを再利用して、対応するパラメーターを、ご自分の Azure Stack Hub ポータルに表示するアイテムの製品詳細に指定して変更することができます。 または、.azpkg ファイルを再利用すると、次の手順をスキップして独自のギャラリー パッケージをカスタマイズできます。

4. Azure Resource Manager テンプレートを作成するか、Windows/Linux 用のサンプル テンプレートを使用します。 これらのサンプル テンプレートは、手順 1 でダウンロードした Packager ツールの .zip ファイルで提供されています。 テンプレートを使用してテキスト フィールドを変更するか、構成済みのテンプレートを GitHub からダウンロードすることができます。 Azure Resource Manager テンプレートの詳細については、[Azure Resource Manager テンプレート](/azure/azure-resource-manager/resource-group-authoring-templates)に関するページを参照してください。

5. ギャラリー パッケージは、次のような構造になっている必要があります。

   ![ギャラリー パッケージの構造のスクリーンショット](media/azure-stack-create-and-publish-marketplace-item/gallerypkg1.png)

6. **Manifest.json** テンプレートで次の強調表示されている値 (番号が付いているもの) を、[カスタム イメージをアップロード](azure-stack-add-vm-image.md#add-a-platform-image)するときに指定した値に置き換えます。

   > [!NOTE]  
   > プロダクト キー、パスワード、お客様を特定できる情報などの機密情報を Azure Resource Manager テンプレートにハード コーディングしないでください。 テンプレート JSON ファイルは、ギャラリーで公開されると、認証の必要なくアクセスできます。 機密情報はすべて [Key Vault](/azure/azure-resource-manager/resource-manager-keyvault-parameter) に格納し、テンプレート内から呼び出してください。

   独自のカスタム テンプレートを発行する前に、サンプルをそのまま発行して、環境内で動作することを確認することをお勧めします。 この手順が正常に実行されたことを確認したら、ギャラリーからサンプルを削除し、結果に満足できるまで反復的な変更を行います。

   次のテンプレートは、Manifest.json ファイルのサンプルです。

    ```json
    {
       "$schema": "https://gallery.azure.com/schemas/2015-10-01/manifest.json#",
       "name": "Test", (1)
       "publisher": "<Publisher name>", (2)
       "version": "<Version number>", (3)
       "displayName": "ms-resource:displayName", (4)
       "publisherDisplayName": "ms-resource:publisherDisplayName", (5)
       "publisherLegalName": "ms-resource:publisherDisplayName", (6)
       "summary": "ms-resource:summary",
       "longSummary": "ms-resource:longSummary",
       "description": "ms-resource:description",
       "longDescription": "ms-resource:description",
       "links": [
        { "displayName": "ms-resource:documentationLink", "uri": "http://go.microsoft.com/fwlink/?LinkId=532898" }
        ],
       "artifacts": [
          {
             "isDefault": true
          }
       ],
       "images": [{
          "context": "ibiza",
          "items": [{
             "id": "small",
             "path": "icons\\Small.png", (7)
             "type": "icon"
             },
             {
                "id": "medium",
                "path": "icons\\Medium.png",
                "type": "icon"
             },
             {
                "id": "large",
                "path": "icons\\Large.png",
                "type": "icon"
             },
             {
                "id": "wide",
                "path": "icons\\Wide.png",
                "type": "icon"
             }]
        }]
    }
    ```

    次の一覧では、前のテンプレート例で示した番号付きの値について説明します。

    - (1) - オファーの名前。
    - (2) - 公開元の名前 (スペースが含まれないもの)。
    - (3) - テンプレートのバージョン (スペースが含まれないもの)。
    - (4) - 顧客に表示される名前。
    - (5) - 顧客に表示される公開元の名前。
    - (6) - 公開元の正式な名前。
    - (7) - 各アイコンのパスと名前。

7. **ms-resource** が参照されているすべてのフィールドを、**strings/resources.json** ファイル内の適切な値に変更する必要があります。

    ```json
    {
    "displayName": "<OfferName.PublisherName.Version>",
    "publisherDisplayName": "<Publisher name>",
    "summary": "Create a simple VM",
    "longSummary": "Create a simple VM and use it",
    "description": "<p>This is just a sample of the type of description you could create for your gallery item!</p><p>This is a second paragraph.</p>",
    "documentationLink": "Documentation"
    }
    ```

8. デプロイ テンプレート ファイルの構造は次のように表示されます。

   :::image type="content" source="media/azure-stack-create-and-publish-marketplace-item/deployment-templates.png" alt-text="デプロイ テンプレート":::

   **createuidefinition.js** ファイルにあるイメージの値を、カスタム イメージをアップロードするときに指定した値に置き換えます。

9. リソースを正常にデプロイできるようにするには、[Azure Stack Hub API](../user/azure-stack-profiles-azure-resource-manager-versions.md) を使用してテンプレートをテストします。

10. お使いのテンプレートが仮想マシン (VM) のイメージに依存する場合、指示に従って [Azure Stack Hub に VM イメージを追加](azure-stack-add-vm-image.md)します。

11. Azure Resource Manager テンプレートを **/Contoso.TodoList/DeploymentTemplates/** フォルダーに保存します。

12. Marketplace アイテムのアイコンとテキストを選択します。 **Icons** フォルダーにアイコンを追加し、**Strings** フォルダー内の **resources** ファイルにテキストを追加します。 アイコンには **small**、**medium**、**large**、および **wide** の名前付け規則を使用します。 これらのサイズの詳細については、「[Marketplace アイテムの UI リファレンス](#reference-marketplace-item-ui)」を参照してください。

    > [!NOTE]
    > Marketplace アイテムを正しく構築するには、4 つすべてのアイコン サイズ (小、中、大、ワイド) が必要です。

13. **Manifest.json** に対するその他の編集については、「[リファレンス: Marketplace アイテム manifest.json](#reference-marketplace-item-manifestjson)」を参照してください。

14. ファイルの変更が終わったら、それを .azpkg ファイルに変換します。 変換は、**AzureGallery.exe** ツールと、前にダウンロードしたサンプル ギャラリー パッケージを使用して実行します。 次のコマンドを実行します。

    ```shell
    .\AzureGalleryHubGallery.exe package -m c:\<path>\<gallery package name>\manifest.json -o c:\Temp
    ```

    > [!NOTE]
    > 出力パスは任意のパスを指定でき、C: ドライブの下でなくてもかまいません。 ただし、manifest.json ファイルと出力パッケージ両方への完全なパスが存在している必要があります。 たとえば、出力パスが `C:\<path>\galleryPackageName.azpkg` の場合、フォルダー `C:\<path>` が存在している必要があります。
    >
    >

## <a name="publish-a-marketplace-item"></a>Marketplace アイテムの発行

### <a name="az-modules"></a>[Az モジュール](#tab/az)

1. PowerShell または Azure Storage Explorer を使用して、Marketplace アイテム (.azpkg) を Azure Blob Storage にアップロードします。 ローカルの Azure Stack Hub ストレージにアップロードすることも、パッケージの一時的な場所である Azure Storage にアップロードすることもできます。 BLOB がパブリックにアクセスできることを確認します。

2. ギャラリー パッケージを Azure Stack Hub にインポートするには、まずクライアント VM にリモート接続 (RDP) して、お使いの Azure Stack Hub に作成したばかりのファイルをコピーします。

3. コンテキストを追加します。

    ```powershell
    $ArmEndpoint = "https://adminmanagement.local.azurestack.external"
    Add-AzEnvironment -Name "AzureStackAdmin" -ArmEndpoint $ArmEndpoint
    Connect-AzAccount -EnvironmentName "AzureStackAdmin"
    ```

4. 次のスクリプトを実行して、リソースをギャラリーにインポートします。

    ```powershell
    Add-AzsGalleryItem -GalleryItemUri `
    https://sample.blob.core.windows.net/<temporary blob name>/<offerName.publisherName.version>.azpkg -Verbose
    ```

5. アイテムの格納に使用できる有効なストレージ アカウントがあることを確認します。 `GalleryItemURI` の値は、Azure Stack Hub 管理者ポータルから取得できます。 **[ストレージ アカウント] > [BLOB のプロパティ] > [URL]** の順に選択し、拡張子を .azpkg にします。 ストレージ アカウントは、Marketplace に発行するために一時的に使用するためのものです。

   ギャラリー パッケージが完成し、**Add-AzsGalleryItem** を使用してアップロードすると、カスタム VM が Marketplace と **[リソースの作成]** ビューに表示されるようになります。 **Marketplace の管理** にはカスタム ギャラリー パッケージが表示されないことに注意してください。

   [![アップロードされたカスタムのマーケットプレース項目](media/azure-stack-create-and-publish-marketplace-item/pkg6sm.png "アップロードされたカスタムのマーケットプレース項目")](media/azure-stack-create-and-publish-marketplace-item/pkg6.png#lightbox)

6. アイテムが Marketplace に正常に発行されたら、ストレージ アカウントからコンテンツを削除できます。

   すべての既定のギャラリー アイテムと、カスタムのギャラリー アイテムは、次の URL での認証を使用せずにアクセスできるようになりました。

   - `https://galleryartifacts.adminhosting.[Region].[externalFQDN]/artifact/20161101/[TemplateName]/DeploymentTemplates/Template.json`
   - `https://galleryartifacts.hosting.[Region].[externalFQDN]/artifact/20161101/[TemplateName]/DeploymentTemplates/Template.json`

7. Marketplace アイテムを削除するには、**Remove-AzGalleryItem** コマンドレットを使用します。 次に例を示します。

   ```powershell
   Remove-AzsGalleryItem -Name <Gallery package name> -Verbose
   ```

> [!Note]  
> アイテムを削除した後に Marketplace UI でエラーが表示される可能性があります。 このエラーを解決するには、ポータルで **[設定]** をクリックします。 次に、 **[ポータルのカスタマイズ]** で **[変更を破棄する]** を選択します。

### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm)

1. PowerShell または Azure Storage Explorer を使用して、Marketplace アイテム (.azpkg) を Azure Blob Storage にアップロードします。 ローカルの Azure Stack Hub ストレージにアップロードすることも、パッケージの一時的な場所である Azure Storage にアップロードすることもできます。 BLOB がパブリックにアクセスできることを確認します。

2. ギャラリー パッケージを Azure Stack Hub にインポートするには、まずクライアント VM にリモート接続 (RDP) して、お使いの Azure Stack Hub に作成したばかりのファイルをコピーします。

3. コンテキストを追加します。

    ```powershell
    $ArmEndpoint = "https://adminmanagement.local.azurestack.external"
    Add-AzureRMEnvironment -Name "AzureStackAdmin" -ArmEndpoint $ArmEndpoint
    Add-AzureRMAccount -EnvironmentName "AzureStackAdmin"
    ```

4. 次のスクリプトを実行して、リソースをギャラリーにインポートします。

    ```powershell
    Add-AzsGalleryItem -GalleryItemUri `
    https://sample.blob.core.windows.net/<temporary blob name>/<offerName.publisherName.version>.azpkg -Verbose
    ```

5. アイテムの格納に使用できる有効なストレージ アカウントがあることを確認します。 `GalleryItemURI` の値は、Azure Stack Hub 管理者ポータルから取得できます。 **[ストレージ アカウント] > [BLOB のプロパティ] > [URL]** の順に選択し、拡張子を .azpkg にします。 ストレージ アカウントは、Marketplace に発行するために一時的に使用するためのものです。

   ギャラリー パッケージが完成し、**Add-AzsGalleryItem** を使用してアップロードすると、カスタム VM が Marketplace と **[リソースの作成]** ビューに表示されるようになります。 **Marketplace の管理** にはカスタム ギャラリー パッケージが表示されないことに注意してください。

   [![アップロードされたカスタムのマーケットプレース項目](media/azure-stack-create-and-publish-marketplace-item/pkg6sm.png "アップロードされたカスタムのマーケットプレース項目")](media/azure-stack-create-and-publish-marketplace-item/pkg6.png#lightbox)

6. アイテムが Marketplace に正常に発行されたら、ストレージ アカウントからコンテンツを削除できます。

   すべての既定のギャラリー アイテムと、カスタムのギャラリー アイテムは、次の URL での認証を使用せずにアクセスできるようになりました。

   - `https://galleryartifacts.adminhosting.[Region].[externalFQDN]/artifact/20161101/[TemplateName]/DeploymentTemplates/Template.json`
   - `https://galleryartifacts.hosting.[Region].[externalFQDN]/artifact/20161101/[TemplateName]/DeploymentTemplates/Template.json`

7. Marketplace アイテムを削除するには、**Remove-AzGalleryItem** コマンドレットを使用します。 次に例を示します。

   ```powershell
   Remove-AzsGalleryItem -Name <Gallery package name> -Verbose
   ```

> [!Note]  
> アイテムを削除した後に Marketplace UI でエラーが表示される可能性があります。 このエラーを解決するには、ポータルで **[設定]** をクリックします。 次に、 **[ポータルのカスタマイズ]** で **[変更を破棄する]** を選択します。

---

## <a name="reference-marketplace-item-manifestjson"></a>リファレンス: Marketplace アイテム manifest.json

### <a name="identity-information"></a>ID 情報

| 名前 | 必須 | Type | 制約 | 説明 |
| --- | --- | --- | --- | --- |
| 名前 |X |String |[A-Za-z0-9]+ | |
| Publisher |X |String |[A-Za-z0-9]+ | |
| Version |X |String |[SemVer v2](https://semver.org/) | |

### <a name="metadata"></a>Metadata

| 名前 | 必須 | Type | 制約 | 説明 |
| --- | --- | --- | --- | --- |
| DisplayName |X |String |推奨 80 文字 |項目名が 80 文字より長い場合、ポータルで適切に表示されないことがあります。 |
| PublisherDisplayName |X |String |推奨 30 文字 |発行元の名前が 30 文字より長い場合、ポータルで適切に表示されないことがあります。 |
| PublisherLegalName |X |String |最大 256 文字 | |
| まとめ |X |String |60 ～ 100 文字 | |
| LongSummary |X |String |140 ～ 256 文字 |現在はまだ Azure Stack Hub には該当しません。 |
| 説明 |X |[HTML](https://github.com/Azure/portaldocs/blob/master/gallery-sdk/generated/index-gallery.md#gallery-item-metadata-html-sanitization) |500 ～ 5,000 文字 | |

### <a name="images"></a>イメージ

Marketplace では、次のアイコンを使用します。

| 名前 | 幅 | [高さ] | Notes |
| --- | --- | --- | --- |
| Wide |255 px |115 px |常に必要 |
| Large |115 px |115 px |常に必要 |
| Medium |90 px |90 px |常に必要 |
| Small |40 px |40 px |常に必要 |
| Screenshot |533 px |324 px |省略可能 |

### <a name="categories"></a>Categories

Marketplace の各アイテムは、そのアイテムのポータル UI における表示場所を識別するカテゴリでタグ付けする必要があります。 Azure Stack Hub 内の既存のカテゴリのいずれか ( **[コンピューティング]** 、 **[データ + ストレージ]** など) を選択するか、新しいものを選択します。

### <a name="links"></a>リンク

各 Marketplace アイテムには、追加コンテンツへのさまざまなリンクを含めることができます。 これらのリンクは、次の名前と URI の一覧として指定されます。

| 名前 | 必須 | Type | 制約 | 説明 |
| --- | --- | --- | --- | --- |
| DisplayName |X |String |最大 64 文字。 | |
| Uri |X |URI | | |

### <a name="additional-properties"></a>追加のプロパティ

前述のメタデータに加えて、Marketplace 作成者は次の形式でカスタムのキー/値のペアでデータを指定することができます。

| 名前 | 必須 | Type | 制約 | 説明 |
| --- | --- | --- | --- | --- |
| DisplayName |X |String |最大 25 文字。 | |
| 値 |X |String |最大 30 文字。 | |

### <a name="html-sanitization"></a>HTML のサニタイズ

HTML を許可するフィールドでは、次の[要素と属性を使用できます](https://github.com/Azure/portaldocs/blob/master/gallery-sdk/generated/index-gallery.md#gallery-item-metadata-html-sanitization)。

`h1, h2, h3, h4, h5, p, ol, ul, li, a[target|href], br, strong, em, b, i`

## <a name="reference-marketplace-item-ui"></a>リファレンス: Marketplace アイテムの UI

Azure Stack Hub ポータルに表示される Marketplace アイテムのアイコンとテキストは、次のとおりです。

### <a name="create-blade"></a>[作成] ブレード

![作成ブレード - Azure Stack Hub の Marketplace アイテム](media/azure-stack-create-and-publish-marketplace-item/image1.png)

### <a name="marketplace-item-details-blade"></a>[Marketplace item details (Marketplace アイテムの詳細)] ブレード

![Azure Stack Hub の [Marketplace item details (Marketplace アイテムの詳細)] ブレード](media/azure-stack-create-and-publish-marketplace-item/image3.png)

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub Marketplace の概要](azure-stack-marketplace.md)
- [Marketplace アイテムのダウンロード](azure-stack-download-azure-marketplace-item.md)
- [Azure Resource Manager テンプレートのフォーマットと構造](/azure/azure-resource-manager/resource-group-authoring-templates)
