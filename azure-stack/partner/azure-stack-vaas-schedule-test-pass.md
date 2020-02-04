---
title: Azure Stack のサービスとしての検証ポータルを使用して初めてのテストをスケジュールする
description: Azure Stack のサービスとしての検証ポータルを使用して初めてのテストをスケジュールします。
author: mattbriggs
ms.topic: conceptual
ms.date: 11/11/2019
ms.author: mabrigg
ms.reviewer: johnhas
ms.lastreviewed: 11/11/2019
ms.openlocfilehash: 166a75da9f6571e80cdfe213fe0ea32c58279dac
ms.sourcegitcommit: fd5d217d3a8adeec2f04b74d4728e709a4a95790
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/29/2020
ms.locfileid: "76885129"
---
# <a name="scheduling-a-test"></a>テストのスケジュール設定

[!INCLUDE [Azure_Stack_Partner](./includes/azure-stack-partner-appliesto.md)]

サービスとしての検証 (VaaS) ポータルで、Azure Stack ソリューションのテストをスケジュールします。 VaaS ソリューションは、特定のハードウェア部品表 (BoM) を含む Azure Stack ソリューションを表します。 ハードウェアが Azure Stack を実行できることを確認するためのテストをスケジュールできます。

ソリューションを確認するには、テストのワークフローを作成します。 VaaS ワークフローは、VaaS ソリューションのコンテキスト内で動作します。 ワークフローは、ハードウェア上で Azure Stack デプロイの機能を実行する一連のテスト スイートを表します。 ソリューションの環境パラメーターを追加し、ソリューションで実行する 1 つ以上のテストを選択します。

テスト成功ワークフローを使用して、VaaS で提供されるテスト (検証ワークフローに含まれているテストなど) を実行できますが、テスト成功ワークフローの結果は "*公式*" とは見なされません。 公式の検証ワークフローについては、「[ワークフロー](azure-stack-vaas-key-concepts.md#workflows)」をご覧ください。

## <a name="prerequisites"></a>前提条件

このクイック スタートを実行する前に、次の項目を完了してください。

- [サービスとしての検証のリソースを設定する](azure-stack-vaas-set-up-resources.md)
- [ローカル エージェントをデプロイする](azure-stack-vaas-local-agent.md) (必須)
- [サービスとしての検証の主要概念](azure-stack-vaas-key-concepts.md) (必須)

## <a name="start-a-workflow"></a>ワークフローの開始

![VaaS ポータルにサインインする](media/vaas_portalsignin.png)

ポータルにサインインし、ソリューションを選択するか、ソリューションを作成してそのソリューションを選択します。

1. [VaaS ポータル](https://azurestackvalidation.com)にサインインします。
2. 既存のソリューションの名前を入力するか、 **[新しいソリューション]** を選択して新しいソリューションを作成します。 手順については、「[VaaS ポータルでソリューションを作成する](azure-stack-vaas-key-concepts.md#create-a-solution-in-the-vaas-portal)」をご覧ください。
3. **[Test Passes]\(テスト成功\)** タイルで **[開始]** を選択します。

## <a name="specify-parameters"></a>パラメーターの指定

![代替テキスト](media/vaas_test_pass_parameters.png)

ワークフロー内のすべてのテストに適用されるパラメーターを指定します。

1. [!INCLUDE [azure-stack-vaas-workflow-step_naming](includes/azure-stack-vaas-workflow-step_naming.md)]
2. [!INCLUDE [azure-stack-vaas-workflow-step_upload-stampinfo](includes/azure-stack-vaas-workflow-step_upload-stampinfo.md)]
3. [!INCLUDE [azure-stack-vaas-workflow-step_test-params](includes/azure-stack-vaas-workflow-step_test-params.md)]
4. [!INCLUDE [azure-stack-vaas-workflow-step_tags](includes/azure-stack-vaas-workflow-step_tags.md)]
5. **[次へ]** を選択して、スケジュールするテストを選択します。

## <a name="select-tests-to-run"></a>実行するテストの選択

選択したテストは、ワークフローの作成後にスケジュールされます。

1. ワークフローで実行するテストを選択します。

    テストの共通パラメーター (前のセクションで指定したパラメーター) を上書きする場合は、横の **[編集]** リンクを選択して新しい値を指定します。

1. [!INCLUDE [azure-stack-vaas-workflow-step_select-agent](includes/azure-stack-vaas-workflow-step_select-agent.md)]

1. **[次へ]** を選択して、ワークフローを確認します。

## <a name="review-and-submit"></a>確認と送信

ワークフローの作成を完了します。

1. 表示された情報を確認します。

    サービスにより、指定した情報を使用してワークフローが作成され、選択したテストがスケジュールされます。

    誤りがある場合は、 **[前へ]** ボタンを使用して前のセクションに移動します。

1. [!INCLUDE [azure-stack-vaas-workflow-step_submit](includes/azure-stack-vaas-workflow-step_submit.md)]

## <a name="next-steps"></a>次のステップ

- [VaaS ポータルでのテストの監視と管理](azure-stack-vaas-monitor-test.md)
