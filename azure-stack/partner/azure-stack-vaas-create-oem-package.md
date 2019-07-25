---
title: Azure Stack の検証のベスト プラクティス | Microsoft Docs
description: この記事では、サービスとしての検証を使用するためのベスト プラクティスについて説明します。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/23/2019
ms.author: mabrigg
ms.reviewer: johnhas
ms.lastreviewed: 03/11/2019
ROBOTS: NOINDEX
ms.openlocfilehash: a7e866187c81b488cc90f7ddaa443db4868ba4d9
ms.sourcegitcommit: b95983e6e954e772ca5267304cfe6a0dab1cfcab
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/23/2019
ms.locfileid: "68417744"
---
# <a name="create-an-oem-package"></a>OEM パッケージの作成

[!INCLUDE [Azure_Stack_Partner](./includes/azure-stack-partner-appliesto.md)]

Azure Stack OEM 拡張機能パッケージは、Azure Stack インフラストラクチャに OEM 固有のコンテンツを追加するメカニズムであり、デプロイと運用プロセス (更新、拡張、フィールドの置換など) で使用します。

## <a name="creating-the-package"></a>パッケージの作成

OEM 拡張機能パッケージを作成して検証したら、VaaS で使用できます。  続行する前に、[OEM パッケージの作成](https://microsoft.sharepoint.com/:w:/r/teams/cloudsolutions/Sacramento/_layouts/15/Doc.aspx?sourcedoc=%7BD7406069-7661-419C-B3B1-B6A727AB3972%7D&file=Azure%20Stack%20OEM%20Extension%20Package.docx&action=default&mobileredirect=true)の手順を完了していることを確認してください。 その後、パッケージは VaaS のテスト結果と共に Microsoft に提出され、パッケージ検証ワークフローで署名されます。 次の手順は、生成されたファイルを、VaaS で使用できる単一の ZIP ファイルにバンドルする方法の詳細を示しています。

1. パッケージの次のコンテンツを確認します。
    - パッケージのコンテンツが含まれている ZIP ファイル
    - `oemMetadata.xml` という名前のマニフェスト ファイル。パッケージ コンテンツのルートにある metadata.xml ファイルと同じ内容である必要があります。

2. コンテンツ ファイルを選択し、コンテンツから ZIP ファイルを作成します。

    ![ZIP ファイルの内容](media/vaas-create-oem-package-1.png) ![項目のコンテンツの圧縮](media/vaas-create-oem-package-2.png)

3. ファイルを識別できるように、生成されたファイルの名前をわかりやすい名前に変更します。

## <a name="verifying-the-contents"></a>コンテンツの確認

ZIP ファイルの構造を検証するために、ZIP ファイルを検査し、サブフォルダーがないことを確認します。 TLD には圧縮済みのコンテンツがあります。 有効なパッケージ構造を次に示します。
> [!IMPORTANT]
> コンテンツではなく、親フォルダーを圧縮すると、パッケージの署名が失敗します。

![適切に圧縮されたパッケージ コンテンツ](media/vaas-create-oem-package-3.png)

これで、ZIP ファイルを VaaS にアップロードし、パッケージ検証ワークフローで Microsoft が署名できるようになりました。

## <a name="next-steps"></a>次の手順

- [OEM パッケージを検証する](azure-stack-vaas-validate-oem-package.md)
