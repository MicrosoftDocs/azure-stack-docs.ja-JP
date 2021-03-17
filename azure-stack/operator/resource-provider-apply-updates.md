---
title: Azure Stack Hub リソース プロバイダーに更新プログラムを適用する
description: Azure Stack Hub のリソース プロバイダーにサービス更新プログラムを適用する方法について説明します。
author: BryanLa
ms.author: bryanla
ms.service: azure-stack
ms.topic: how-to
ms.date: 03/01/2021
ms.reviewer: jfggdl
ms.lastreviewed: 03/01/2021
zone_pivot_groups: state-connected-disconnected
ms.openlocfilehash: 60f273603d03c6625addcfeba345a198f82fbe31
ms.sourcegitcommit: 2c6418ee465e67edd417961b1f5211b2e09dbd5f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "102117007"
---
# <a name="how-to-update-an-azure-stack-hub-resource-provider"></a>Azure Stack Hub リソース プロバイダーを更新する方法

> [!IMPORTANT]
> 続行する前に、必ずリソース プロバイダーの最新リリース ノートを確認して、新しい機能、修正、およびデプロイに影響を与える可能性のある既知の問題について確認してください。 リリース ノートには、リソース プロバイダーに必要な Azure Stack Hub の最小バージョンが指定されている場合もあります。 これまでリソース プロバイダーをインストールしたことがない場合は、リソース プロバイダーの前提条件と最初のインストール手順を代わりに参照してください。

Marketplace からインストールされたリソース プロバイダーには、定期的な保守が必要です。 保守は、Microsoft から定期的に提供されるサービス更新プログラムを適用することによって行います。 更新プログラムには、新機能と修正プログラムの両方が含まれます。  

## <a name="check-for-updates"></a>更新プログラムをチェックする

リソース プロバイダーの更新には、Azure Stack Hub の更新プログラムの適用に使用されるのと同じ更新機能を使用します。

1. Azure Stack Hub 管理者ポータルにサインインします。
2. 左側にある **[すべてのサービス]** リンクを選択し、 **[管理]** セクションで **[更新プログラム]** を選択します。
   [![[すべてのサービス] ページ](media/resource-provider-apply-updates/1-all-services.png)](media/resource-provider-apply-updates/1-all-services.png#lightbox)

3. **[更新プログラム]** ページの **[リソース プロバイダー]** セクションに、リソース プロバイダーの更新プログラムが表示されます。 **[状態]** は "使用可能" になっています。

   [![[リソース プロバイダー] セクションを示すスクリーンショット。](media/resource-provider-apply-updates/3-update-available.png)](media/resource-provider-apply-updates/3-update-available.png#lightbox)

## <a name="download-package"></a>パッケージのダウンロード

特定のリソース プロバイダーで使用可能な更新プログラムがある場合:

::: zone pivot="state-connected"
1. **[更新プログラム]** ページの **[リソース プロバイダー]** セクションで、更新するリソース プロバイダーの行を選択します。 ページの上部にある **[ダウンロード]** リンクが有効になっていることを確認します。
   [![使用可能な更新プログラム ページ](media/resource-provider-apply-updates/4-download.png)](media/resource-provider-apply-updates/3-update-available.png#lightbox)

2. **[ダウンロード]** リンクをクリックして、リソース プロバイダーのインストール パッケージのダウンロードを開始します。 リソース プロバイダー行の **[状態]** 列が "使用可能" から "ダウンロード中" に変更されていることに注目してください。
3. **[状態]** が "インストールの準備完了" に変更されたら、ダウンロードが完了です。 
::: zone-end

::: zone pivot="state-disconnected" 
[!INCLUDE [prereqs](../includes/resource-provider-va-package-download-disconnected.md)]
::: zone-end

## <a name="apply-an-update"></a>更新プログラムを適用する

リソース プロバイダー パッケージがダウンロードされたら、 **[更新プログラム]** ページの **[リソース プロバイダー]** セクションに戻ります。

1. 更新するリソース プロバイダーの行を選択します。 **[状態]** に "インストールの準備完了" と表示され、ページの上部にある **[今すぐインストール]** リンクが有効になります。
2. **[今すぐインストール]** リンクを選択すると、リソース プロバイダーの **[インストール]** ページが開きます。 
3. **[インストール]** ボタンを選択して、インストールを開始します。
4. "インストール中" という通知が右上に表示され、 **[更新プログラム]** ページに戻ります。 リソース プロバイダー行の **[状態]** 列も "インストール中" に変更されます。
5. インストールが完了すると、別の通知によって成功または失敗が示されます。 インストールが成功すると、 **[Marketplace Management - リソース プロバイダー]** ページの **[バージョン]** も更新されます。

## <a name="next-steps"></a>次のステップ

[管理者ダッシュボードの更新機能](azure-stack-apply-updates.md)の詳細を確認します。
