---
title: Azure Stack HCI にブランチ オフィスとエッジをデプロイする
description: このトピックでは、Azure Stack HCI オペレーティング システム上で ブランチ オフィスとエッジのシナリオを計画、構成、デプロイする方法に関するガイダンスを提供します。
author: JohnCobb1
ms.author: v-johcob
ms.topic: how-to
ms.date: 01/21/2021
ms.openlocfilehash: ee9fef4a7da2312876eff168573b1a26bea0290b
ms.sourcegitcommit: f194f9ca4297864500e62d8658674a0625b29d1d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102186842"
---
# <a name="deploy-branch-office-and-edge-on-azure-stack-hci"></a>Azure Stack HCI にブランチ オフィスとエッジをデプロイする

>適用対象:Azure Stack HCI バージョン 20H2

このトピックでは、Azure Stack HCI オペレーティング システム上で ブランチ オフィスとエッジのシナリオを計画、構成、デプロイする方法に関するガイダンスを提供します。 組織は、このガイダンスに従うことで、リモート ブランチ オフィスおよびエッジのデプロイに含まれる仮想マシン (VM) とコンテナーで、複雑で可用性の高いワークロードを実行できるようになります。 エッジでのコンピューティングにより、データ処理の大部分が、中央のシステムから、速やかにデータを必要とするデバイスやシステムの近くのネットワークのエッジに移されます。

Azure Stack HCI を使用して、推奨されるハードウェア上で高可用性の仮想化アプリケーションとワークロードを実行します。 ハードウェアでは、ストレージの入れ子になった回復性、シンプルで低コストの USB ドライブ クラスター監視、およびブラウザーベースの Windows Admin Center による管理が構成された 2 台のサーバーで構成されるクラスターがサポートされます。 USB デバイス クラスター監視の作成の詳細については、「[ファイル共有監視を展開する](/windows-server/failover-clustering/file-share-witness)」を参照してください。

Azure IoT Edge により、クラウドの分析とカスタム ビジネス ロジックがデバイスに移動されて、ユーザーはデータの管理ではなくビジネスの分析情報に集中できるようになります。 Azure IoT Edge により、Azure Cognitive Services、Machine Learning、Stream Analytics、Functions などのコンテナー化されたクラウド ワークロードに、AI、クラウド、エッジ コンピューティングが統合されます。 ワークロードは、Raspberry Pi から集約エッジ サーバーまで、さまざまなデバイスで実行できます。 エッジ アプリケーションとデバイスを管理するには、[Azure IoT Hub](https://azure.microsoft.com/services/iot-hub) を使用します。

Azure Stack HCI のブランチ オフィスとエッジのデプロイに Azure IoT Edge を追加することで、環境が最新化され、[CI/CD パイプライン](/azure/iot-edge/how-to-continuous-integration-continuous-deployment)のアプリケーション デプロイ フレームワークがサポートされるようになります。 組織の DevOps 担当者は、従来の VM 管理プロセスとツールを使用して IT 部門が構築およびサポートするコンテナー化されたアプリケーションを、デプロイおよび反復処理できます。

Azure IoT Edge の主な機能:
- Microsoft が提供するオープンソース ソフトウェア
- Windows または Linux で実行される
- "エッジ" で実行されて、ほぼリアルタイムの応答を提供する
- ソフトウェアとハードウェアのメカニズムをセキュリティで保護する
- [Azure IoT Edge 用 AI ツールキット](https://github.com/Azure/ai-toolkit-iot-edge)で使用できる
- オープンなプログラム可能性のサポート: Java、.NET Core 2.0、Node.js、C、Python
- オフラインおよび断続的接続のサポート
- Azure IoT Hub からのネイティブ管理

詳細については、「[Azure IoT Edge とは](/azure/iot-edge/about-iot-edge)」を参照してください。

## <a name="deploy-branch-office-and-edge"></a>ブランチ オフィスとエッジをデプロイする
このセクションでは、Azure Stack HCI 上にブランチ オフィスとエッジをデプロイするためのハードウェアを取得する方法と、管理のために Windows Admin Center を使用する方法について説明します。 また、クラウドでコンテナーを管理するための Azure IoT Edge のデプロイについても説明します。

### <a name="step-1-acquire-hardware-from-the-azure-stack-hci-catalog"></a>手順 1:Azure Stack HCI カタログからハードウェアを取得する
最初に、ハードウェアを調達する必要があります。 最も簡単な方法としては、[Azure Stack HCI カタログ](https://hcicatalog.azurewebsites.net)で適切な Microsoft ハードウェア パートナーを見つけて、Azure Stack HCI オペレーティング システムがプレインストールされている統合システムを購入します。 カタログでは、フィルターを適用して、この種類のワークロードに最適化されたベンダーのハードウェアを表示できます。

それ以外の場合は、Azure Stack HCI オペレーティング システムを独自のハードウェアにデプロイする必要があります。 Azure Stack HCI デプロイ オプションの詳細および Windows Admin Center のインストールについては、「[Azure Stack HCI オペレーティング システムのデプロイ](./operating-system.md)」を参照してください。

次に、Windows Admin Center を使用して、[Azure Stack HCI クラスターを作成します](./create-cluster.md)。

### <a name="step-2-set-up-azure-monitor-in-windows-admin-center"></a>手順 2:Windows Admin Center で Azure Monitor を設定する
Windows Admin Center で、Azure Stack HCI のブランチ オフィスとエッジのデプロイのアプリケーション、ネットワーク、サーバーの正常性に関する分析情報を得るために、Azure Monitor を設定します。

詳細については、「[Azure Monitor を使用して Azure Stack HCI を監視する](../manage/azure-monitor.md)」を参照してください。

Windows Admin Center を使用して、Backup、File Sync、Site Recovery、Point-to-Site VPN、Update Management、Security Center などの追加の Azure ハイブリッド サービスを設定することもできます。

### <a name="step-3-use-container-based-apps-and-iot-data-processing"></a>手順 3: コンテナー ベースのアプリと IoT データ処理を使用する
最新のコンテナー ベースのアプリケーション開発と IoT データ処理を使用できる状態になっています。 このセクションの手順では、Windows Admin Center を使用して、Azure IoT Edge を実行する VM をデプロイします。

詳細については、「[Azure IoT Edge とは](/azure/iot-edge/about-iot-edge)」を参照してください。

Azure Stack HCI に Azure IoT Edge をデプロイするには:
1. [Azure Stack HCI に新しい VM を作成する](/windows-server/manage/windows-admin-center/use/manage-virtual-machines#create-a-new-virtual-machine)には、Windows Admin Center を使用します。

    サポートされるオペレーティング システムのバージョン、VM の種類、プロセッサのアーキテクチャ、システムの要件については、「[Azure IoT Edge のサポートされるシステム](/azure/iot-edge/support)」を参照してください。

1. まだ Azure アカウントを持っていない場合は、[無料アカウント](https://azure.microsoft.com/free)を開始します。
1. Azure portal で、[Azure IoT ハブを作成します](/azure/iot-edge/quickstart#create-an-iot-hub)。
1. Azure portal で、[IoT Edge デバイスを登録します](/azure/iot-edge/quickstart#register-an-iot-edge-device)。

    >[!NOTE]
    > IoT Edge デバイスは、Azure Stack HCI 上の Windows または Linux が実行されている VM 上にあります。

1. 手順 1 で作成した VM で、[IoT Edge ランタイムをインストールして起動します](/azure/iot-edge/quickstart#install-and-start-the-iot-edge-runtime)。

   >[!IMPORTANT]
   > ランタイムを Azure IoT Hub に接続するには、手順 4 で作成したデバイス文字列が必要です。

1. Azure IoT Edge に[モジュールをデプロイします](/azure/iot-edge/quickstart#deploy-a-module)。

    Azure Marketplace の [IoT Edge モジュール](https://azuremarketplace.microsoft.com/marketplace/apps/category/internet-of-things?page=1&subcategories=iot-edge-modules) セクションから、構築済みのモジュールを入手してデプロイできます。

## <a name="next-steps"></a>次のステップ
ブランチ オフィスとエッジ、および Azure IoT Edge の詳細については、以下を参照してください。
- [クイック スタート:初めての IoT Edge モジュールを Linux 仮想デバイスにデプロイする](/azure/iot-edge/quickstart-linux?preserve-view=true&view=iotedge-2018-06)
- [クイックスタート:初めての IoT Edge モジュールを Windows デバイスにデプロイする](/azure/iot-edge/quickstart?preserve-view=true&view=iotedge-2018-06)