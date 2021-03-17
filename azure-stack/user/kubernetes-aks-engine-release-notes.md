---
title: Azure Stack Hub 上の Azure Kubernetes Service (AKS) エンジンのリリース ノート
description: Azure Stack Hub 上の AKS エンジンの更新プログラムを使用して実行する必要がある手順について説明します。
author: mattbriggs
ms.topic: article
ms.date: 03/01/2021
ms.author: mabrigg
ms.reviewer: waltero
ms.lastreviewed: 03/01/2021
ms.openlocfilehash: 9cccf83444e79aede3f88751dbcb77b77bd5ea4a
ms.sourcegitcommit: ccc4ee05d71496653b6e27de1bb12e4347e20ba4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102231729"
---
# <a name="release-notes-for-the-aks-engine-on-azure-stack-hub"></a>Azure Stack Hub 上の AKS エンジンのリリース ノート
::: moniker range=">=azs-2005"
*AKS エンジンのバージョン v0.60.1 に適用されます。*

この記事では、Azure Stack Hub 上の Azure Kubernetes Service (AKS) エンジンの更新プログラムの内容について説明します。 この更新プログラムには、Azure Stack Hub プラットフォームを対象とした最新リリースの AKS engine の機能強化と修正が含まれています。 これは、グローバルな Azure 用の AKS エンジンに関するリリース情報を文書化するためのものではないことに注意してください。

## <a name="update-planning"></a>計画の更新

AKS エンジンのアップグレード コマンドを使用すると、クラスターのアップグレード プロセスが完全に自動化され、仮想マシン (VM)、ネットワーク、ストレージ、Kubernetes、オーケストレーション タスクが処理されます。 更新プログラムを適用する前に、必ずリリース ノート情報を確認してください。

### <a name="upgrade-considerations"></a>アップグレードの考慮事項

-   お使いのバージョンの AKS エンジンに適した Marketplace 項目である AKS Base Ubuntu 16.04-LTS または 18.04 Image ディストリビューションまたは AKS Base Windows Server を使用していますか。 これらのバージョンについては、「新しいイメージと AKS エンジンをダウンロードする」セクションを参照してください。
-   ターゲット クラスターに正しいクラスター仕様 (apimodel.json) とリソース グループを使用していますか。 最初にクラスターをデプロイしたときに、このファイルは出力ディレクトリに生成されました。 deploy コマンドのパラメーターについては、「[Kubernetes クラスターのデプロイ](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster?view=azs-2008#deploy-a-kubernetes-cluster)」を参照してください。
-   AKS エンジンを実行してアップグレード操作を実行するために信頼できるマシンを使用していますか。
-   クラスターに通常の負荷がかかっている前提では、アクティブなワークロードを使用して運用中のクラスターを更新する場合、ワークロードに影響を与えずにアップグレードを適用できます。 ただし、ユーザーをリダイレクトする必要がある場合に備えて、バックアップ クラスターが必要です。 バックアップ クラスターを強くお勧めします。
-   可能であれば、Azure Stack Hub 環境内の VM からコマンドを実行して、ネットワーク ホップと接続障害の可能性を減らします。
-   サブスクリプションにプロセス全体のための十分なクォータがあることを確認します。 このプロセスでは、プロセス中に新しい VM が割り当てられます。 結果として得られる VM の数は元の VM と同じですが、このプロセス中に作成する VM をいくつか増やすことを計画してください。
-   システムの更新やスケジュールされたタスクは計画されていません。
-   運用クラスターと同じ値を使用して構成されたクラスターで段階的なアップグレードをセットアップし、運用クラスターでアップグレードを実行する前に、そこでアップグレードをテストします。

### <a name="use-the-upgrade-command"></a>upgrade コマンドを使用する

「[Azure Stack Hub で Kubernetes クラスターをアップグレードする](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade?view=azs-2008)」の記事で説明されているように、AKS エンジンの upgrade コマンドを使用する必要があります。

### <a name="upgrade-interruptions"></a>アップグレードの中断

予期しない要因によってクラスターのアップグレードが中断されることがあります。 AKS エンジンからエラーが報告されたり、AKS エンジンの実行プロセスに何かが発生したりすると、中断が発生する可能性があります。 中断の原因を調べて対処し、同じアップグレード コマンドを再度送信して、アップグレード プロセスを続行します。 **upgrade** コマンドはべき等です。コマンドを再送信したら、クラスターのアップグレードを再開する必要があります。 通常、中断によって更新が完了するまでの時間は長くなりますが、その完了には影響しません。

### <a name="estimated-upgrade-time"></a>推定アップグレード時間

推定所要時間は、クラスター内の VM あたり 12 分から 15 分です。 たとえば、20 ノードのクラスターのアップグレードには、約 5 時間かかる場合があります。

## <a name="download-new-image-and-aks-engine"></a>新しいイメージと AKS エンジンをダウンロードする

AKS ベースの Ubuntu イメージと AKS エンジンの新しいバージョンをダウンロードします。

Azure Stack Hub 用 AKS エンジンのドキュメントで説明されているように、Kubernetes クラスターのデプロイには次のものが必要です。

-   aks-engine バイナリ (必須)
-   AKS Base Ubuntu 16.04-LTS Image ディストリビューション (必須)
-   AKS Base Ubuntu 18.04-LTS Image ディストリビューション (オプション)
-   AKS Base Windows Server Image ディストリビューション (オプション)

この更新プログラムでは、これらの新しいバージョンを使用できます。

-   Azure Stack Hub オペレーターは、新しい AKS Base Images をスタンプ マーケットプレースにダウンロードする必要があります。
    -   AKS Base Ubuntu 16.04-LTS Image ディストリビューション、2021 年 1 月 (2021.01.28)
    -   AKS Base Ubuntu 18.04-LTS Image ディストリビューション、2021 Q1 (2021.01.28)
    -   AKS Base Windows Image (17763.1697.210129)

        「[Azure Kubernetes Services (AKS) エンジンの前提条件を Azure Stack Hub Marketplace に追加する](https://docs.microsoft.com/azure-stack/operator/azure-stack-aks-engine?view=azs-2008)」の記事の指示に従ってください

-   Kubernetes クラスター管理者 (通常は Azure Stack Hub のテナント ユーザー) は、新しい aks-engine バージョン 0.60.1 をダウンロードする必要があります。 「[Azure Stack Hub の Linux に AKS エンジンをインストールする](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux?view=azs-2008)」の記事 (または Windows の該当記事) の手順を参照してください。 クラスターを初めてインストールしたときに使用したのと同じプロセスで実行できます。 この更新により、以前のバイナリが上書きされます。 たとえば、get-akse.sh スクリプトを使用した場合は、この「[接続されている環境へのインストール](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux?view=azs-2008#install-in-a-connected-environment)」セクションの説明と同じ手順を実行します。 Windows システムにインストールする場合も、「[Azure Stack Hub の Windows に AKS エンジンをインストールする](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows?view=azs-2008)」と同じプロセスが適用されます。

## <a name="aks-engine-and-azure-stack-version-mapping"></a>AKS エンジンと Azure Stack のバージョン マッピング

| Azure Stack Hub のバージョン                    | AKS エンジンのバージョン         |
|------------------------------------------------|--------------------------------|
| 1910                                           | 0.43.0、0.43.1                 |
| 2002                                           | 0.48.0、0.51.0                 |
| 2005                                           | 0.48.0、0.51.0、0.55.0、0.55.4 |
| 2008                                           | 0.55.4、0.60.1                 |

## <a name="kubernetes-version-upgrade-path-in-aks-engine-v0601"></a>AKS エンジン v0.60.1 の Kubernetes バージョンのアップグレード パス

次の Azure Stack Hub の表で、最新のバージョンとアップグレード バージョンを確認できます。 aks-engine get-versions コマンドには、グローバル Azure でサポートされているバージョンも含まれるため、従わないでください。 次のバージョンとアップグレードの表は、Azure Stack Hub の AKS エンジン クラスターに適用されます。

| 現在のバージョン                                       | 使用可能なアップグレード |
|-----------------------------------------------------------|-----------------------|
| 1.15.12                                                   | 1.16.14、1.16.15      |
| 1.16.14                                                   | 1.16.15、1.17.17      |
| 1.17.11                                                   | 1.17.17、1.18.15      |
| 1.17.17                                                   | 1.18.15               |

API モデルの json ファイル内にある orchestratorProfile セクションの下でリリースとバージョンの値を指定してください。たとえば、Kubernetes 1.17.17 をデプロイする場合は、次の 2 つの値を設定する必要があります ([kubernetes-azurestack.json](https://aka.ms/aksengine-json-example-raw) の例を参照してください)。

```json  
    -   "orchestratorRelease": "1.17",
    -   "orchestratorVersion": "1.17.17"
```

## <a name="aks-engine-and-corresponding-image-mapping"></a>AKS エンジンと対応するイメージ マッピング

|      AKS エンジン     |      AKS 基本イメージ     |      Kubernetes のバージョン     |      API モデルのサンプル     |
|-|-|-|-|
|     v0.43.1    |     AKS Base Ubuntu 16.04-LTS Image ディストリビューション、2019 年 10 月 (2019.10.24)    |     1.15.5、1.15.4、1.14.8、1.14.7    |  |
|     v0.48.0    |     AKS Base Ubuntu 16.04-LTS Image ディストリビューション、2020 年 3 月 (2020.03.19)    |     1.15.10、1.14.7    |  |
|     v0.51.0    |     AKS Base Ubuntu 16.04-LTS Image ディストリビューション、2020 年 5 月 (2020.05.13)、AKS Base Windows Image (17763.1217.200513)    |     1.15.12、1.16.8、1.16.9    |     [Linux](https://github.com/Azure/aks-engine/blob/v0.51.0/examples/azure-stack/kubernetes-azurestack.json)、[Windows](https://github.com/Azure/aks-engine/blob/v0.51.0/examples/azure-stack/kubernetes-windows.json)    |
|     v0.55.0    |     AKS Base Ubuntu 16.04-LTS Image ディストリビューション、2020 年 8 月 (2020.08.24)、AKS Base Windows Image (17763.1397.200820)    |     1.15.12、1.16.14、1.17.11    |     [Linux](https://github.com/Azure/aks-engine/blob/v0.55.0/examples/azure-stack/kubernetes-azurestack.json)、[Windows](https://github.com/Azure/aks-engine/blob/v0.55.0/examples/azure-stack/kubernetes-windows.json)    |
|     v0.55.4    |     AKS Base Ubuntu 16.04-LTS Image ディストリビューション、2020 年 9 月 (2020.09.14)、AKS Base Windows Image (17763.1397.200820)    |     1.15.12、1.16.14、1.17.11    |     [Linux](https://raw.githubusercontent.com/Azure/aks-engine/patch-release-v0.60.1/examples/azure-stack/kubernetes-azurestack.json)、[Windows](https://raw.githubusercontent.com/Azure/aks-engine/patch-release-v0.60.1/examples/azure-stack/kubernetes-windows.json)    |
|     V0.60.1    |     AKS Base Ubuntu 16.04-LTS Image ディストリビューション、2021 年 1 月 (2021.01.28)   <br>AKS Base Ubuntu 18.04-LTS Image ディストリビューション、2021 Q1 (2021.01.28) <br>AKS Base Windows Image (17763.1697.210129)    |     1.16.14、1.16.15、1.17.17、1.18.15    |     [Linux](https://raw.githubusercontent.com/Azure/aks-engine/patch-release-v0.60.1/examples/azure-stack/kubernetes-azurestack.json)、[Windows](https://raw.githubusercontent.com/Azure/aks-engine/patch-release-v0.60.1/examples/azure-stack/kubernetes-windows.json)    |

## <a name="whats-new"></a>新機能

プライベート プレビューへの参加に関心がある場合は、[こちら](https://forms.office.com/r/yqxXyiDcGG)からプレビューへのアクセスを要求できます。

新機能は次のとおりです。
-   Ubuntu 18.04 の一般提供
-   証明書ローテーションのパブリック プレビュー [\#4214](https://github.com/Azure/aks-engine/pull/4214)
-   T4 Nvidia GPU のプライベート プレビュー [\#4259](https://github.com/Azure/aks-engine/pull/4259)
-   Azure Active Directory 統合のプライベート プレビュー
-   Azure Blob 用 CSI ドライバーのプライベート プレビュー [\#712](https://github.com/kubernetes-sigs/azuredisk-csi-driver/pull/712)
-   CSI ドライバー Azure Disks のパブリック プレビュー [\#712](https://github.com/kubernetes-sigs/azuredisk-csi-driver/pull/712)
-   CSI ドライバー NFS のパブリック プレビュー [\#712](https://github.com/kubernetes-sigs/azuredisk-csi-driver/pull/712)
-   Kubernetes 1. 17.17 [\#4188](https://github.com/Azure/aks-engine/issues/4188) および 1.18.15 [\#4187](https://github.com/Azure/aks-engine/issues/4187) のサポート

## <a name="known-issues"></a>既知の問題

-   1 つのクラスター内に複数の Kubernetes サービスを並行してデプロイすると、基本的なロード バランサー構成でエラーが発生する可能性があります。 一度に 1 つのサービスをデプロイすることをお勧めします。
-   aks-engine ツールは、Azure と Azure Stack Hub 全体の共有ソース コード リポジトリであるためです。 多くのリリース ノートと pull request を調べると、このツールは前述以外のバージョンの Kubernetes と OS プラットフォームをサポートしていることがわかります。これらを無視して、この更新プログラムの公式ガイドとして、前述のバージョン一覧を使用してください。

> [!NOTE]  
> Windows コンテナーと Azure CNI のサポートは、パブリック プレビューで利用できます。

## <a name="reference"></a>リファレンス

これは、Azure と Azure Stack Hub を組み合わせたリリース ノートの完全なセットです。

-   https://github.com/Azure/aks-engine/releases/tag/v0.56.0
-   https://github.com/Azure/aks-engine/releases/tag/v0.56.1
-   https://github.com/Azure/aks-engine/releases/tag/v0.60.0
-   https://github.com/Azure/aks-engine/releases/tag/v0.60.1
::: moniker-end
::: moniker range=">azs-1910 <=azs-2005"
"*AKS エンジンのバージョン v0.55.4 に適用されます。* "

この記事では、Azure Stack Hub 上の Azure Kubernetes Service (AKS) エンジンの更新プログラムの内容について説明します。 この更新プログラムには、Azure Stack Hub プラットフォームを対象とした最新リリースの AKS engine の機能強化と修正が含まれています。 これは、グローバルな Azure 用の AKS エンジンに関するリリース情報を文書化するためのものではないことに注意してください。

## <a name="update-planning"></a>計画の更新

AKS エンジンのアップグレード コマンドを使用すると、クラスターのアップグレード プロセスが完全に自動化され、仮想マシン (VM)、ネットワーク、ストレージ、Kubernetes、オーケストレーション タスクが処理されます。 更新プログラムを適用する前に、必ずリリース ノート情報を確認してください。

### <a name="upgrade-considerations"></a>アップグレードの考慮事項

-   お使いのバージョンの AKS エンジンに適した Marketplace 項目である AKS 基本 Ubuntu 16.04-LTS イメージ ディストリビューションを使用していますか。 これらのバージョンについては、「新しいイメージと AKS エンジンをダウンロードする」セクションを参照してください。

-   ターゲット クラスターに正しいクラスター仕様 (`apimodel.json`) とリソース グループを使用していますか? 最初にクラスターをデプロイしたときに、このファイルは出力ディレクトリに生成されました。 `deploy` コマンドのパラメーターについては、「[Kubernetes クラスターのデプロイ](./azure-stack-kubernetes-aks-engine-deploy-cluster.md#deploy-a-kubernetes-cluster)」を参照してください。

-   AKS エンジンを実行してアップグレード操作を実行するために信頼できるマシンを使用していますか。

-   クラスターに通常の負荷がかかっている前提では、アクティブなワークロードを使用して運用中のクラスターを更新する場合、ワークロードに影響を与えずにアップグレードを適用できます。 ただし、ユーザーをリダイレクトする必要がある場合に備えて、バックアップ クラスターが必要です。 バックアップ クラスターを強くお勧めします。

-   可能であれば、Azure Stack Hub 環境内の VM からコマンドを実行して、ネットワーク ホップと接続障害の可能性を減らします。

-   サブスクリプションにプロセス全体のための十分なクォータがあることを確認します。 このプロセスでは、プロセス中に新しい VM が割り当てられます。 結果として得られる VM の数は元の VM と同じですが、このプロセス中に作成する VM をいくつか増やすことを計画してください。

-   システムの更新やスケジュールされたタスクは計画されていません。

-   運用クラスターと同じ値を使用して構成されたクラスターで段階的なアップグレードをセットアップし、運用クラスターでアップグレードを実行する前に、そこでアップグレードをテストします。

### <a name="use-the-upgrade-command"></a>upgrade コマンドを使用する

「[Azure Stack Hub で Kubernetes クラスターをアップグレードする](./azure-stack-kubernetes-aks-engine-upgrade.md)」の記事で説明されているように、AKS エンジンの `upgrade` コマンドを使用する必要があります。

### <a name="upgrade-interruptions"></a>アップグレードの中断

予期しない要因によってクラスターのアップグレードが中断されることがあります。 AKS エンジンからエラーが報告されたり、AKS エンジンの実行プロセスに何かが発生したりすると、中断が発生する可能性があります。 中断の原因を調べて対処し、同じアップグレード コマンドを再度送信して、アップグレード プロセスを続行します。 **upgrade** コマンドはべき等です。コマンドを再送信したら、クラスターのアップグレードを再開する必要があります。 通常、中断によって更新が完了するまでの時間は長くなりますが、その完了には影響しません。

### <a name="estimated-upgrade-time"></a>推定アップグレード時間

推定所要時間は、クラスター内の VM あたり 12 分から 15 分です。 たとえば、20 ノードのクラスターのアップグレードには、約 5 時間かかる場合があります。

## <a name="download-new-image-and-aks-engine"></a>新しいイメージと AKS エンジンをダウンロードする

AKS ベースの Ubuntu イメージと AKS エンジンの新しいバージョンをダウンロードします。

Azure Stack Hub 用 AKS エンジンのドキュメントで説明されているように、Kubernetes クラスターのデプロイには 2 つの主要コンポーネントが必要です。

-   aks-engine バイナリ

-   AKS 基本 Ubuntu 16.04-LTS イメージ ディストリビューション

この更新プログラムでは、これらの新しいバージョンを使用できます。

-   Azure Stack Hub オペレーターは、新しい AKS 基本 Ubuntu イメージをスタンプ マーケットプレースにダウンロードする必要があります。

    -   名前:AKS Base Ubuntu 16.04-LTS イメージ ディストリビューション、9 月 2020 (2020.09.14)

    -   バージョン:2020.09.14

    -   「[Azure Kubernetes Services (AKS) エンジンの前提条件を Azure Stack Hub Marketplace に追加する](../operator/azure-stack-aks-engine.md)」の記事の指示に従ってください

-   Kubernetes クラスター管理者は、新しい aks-engine バージョン 0.51.0 をダウンロードする必要があります。 「[Azure Stack Hub の Linux に AKS エンジンをインストールする](./azure-stack-kubernetes-aks-engine-deploy-linux.md)」の記事の手順を参照してください。 クラスターを初めてインストールする場合と同じプロセスで実行できます。 この更新により、以前のバイナリが上書きされます。 たとえば、get-akse.sh スクリプトを使用した場合は、この「[接続されている環境へのインストール](./azure-stack-kubernetes-aks-engine-deploy-linux.md#install-in-a-connected-environment)」セクションの説明と同じ手順を実行します。 Windows システムにインストールする場合も、「[Azure Stack Hub の Windows に AKS エンジンをインストールする](./azure-stack-kubernetes-aks-engine-deploy-windows.md)」と同じプロセスが適用されます。


## <a name="aks-engine-and-azure-stack-version-mapping"></a>AKS エンジンと Azure Stack のバージョン マッピング

| Azure Stack Hub のバージョン | AKS エンジンのバージョン |
| ----------------------------- | ------------------------ |
| 1910 | 0.43.0、0.43.1 |
| 2002 | 0.48.0、0.51.0 |
| 2005 | 0.48.0、0.51.0、0.55.0、0.55.4 |

## <a name="kubernetes-version-upgrade-path-in-aks-engine-v0554"></a>AKS エンジン v0.55.4 の Kubernetes バージョンのアップグレード パス

次の Azure Stack Hub の表で、最新のバージョンとアップグレード バージョンを確認できます。 aks-engine get-versions コマンドには、グローバル Azure でサポートされているバージョンも含まれるため、従わないでください。 次のバージョンとアップグレードの表は、Azure Stack Hub の AKS エンジン クラスターに適用されます。

| 現在のバージョン | 使用可能なアップグレード |
| ------------------------- | ----------------------- |
| 1.15.10 | 1.15.12 |
| 1.15.12、1.16.8、1.16.9 | 1.16.14 |
| 1.16.8、1.16.9、1.16.14 | 1.17.11 |

API モデルの json ファイル内にある `orchestratorProfile` セクションの下にリリースとバージョンの値を指定してください。たとえば、Kubernetes 1.16.14 をデプロイする場合は、次の 2 つの値を設定する必要があります ([kubernetes-azurestack.json](https://aka.ms/aksengine-json-example-raw) の例を参照してください)。

```json  
    -   "orchestratorRelease": "1.16",
    -   "orchestratorVersion": "1.16.14"
```

## <a name="aks-engine-and-corresponding-image-mapping"></a>AKS エンジンと対応するイメージ マッピング

| AKS Engine | AKS 基本イメージ | Kubernetes のバージョン | Notes |
|---|---|---|---|
| v0.43.1 | AKS Base Ubuntu 16.04-LTS イメージ ディストリビューション、2019 年 10 月 (2019.10.24) | 1.15.5、1.15.4、1.14.8、1.14.7 |  |
| v0.48.0 | AKS 基本 Ubuntu 16.04-LTS イメージ ディストリビューション、2020 年 3 月 (2020.03.19) | 1.15.10、1.14.7 |  |
| v0.51.0 | AKS 基本 Ubuntu 16.04-LTS イメージ ディストリビューション、2020 年 5 月 (2020.05.13)、AKS 基本 Windows イメージ (17763.1217.200513) | 1.15.12、1.16.8、1.16.9 | API モデルのサンプル ([Linux](https://github.com/Azure/aks-engine/blob/v0.51.0/examples/azure-stack/kubernetes-azurestack.json)、[Windows](https://github.com/Azure/aks-engine/blob/v0.51.0/examples/azure-stack/kubernetes-windows.json)) |
| v0.55.0 | AKS 基本 Ubuntu 16.04-LTS イメージ ディストリビューション、2020 年 8 月 (2020.08.24)、AKS 基本 Windows イメージ (17763.1397.200820) | 1.15.12、1.16.14、1.17.11 | API モデルのサンプル ([Linux](https://github.com/Azure/aks-engine/blob/v0.55.0/examples/azure-stack/kubernetes-azurestack.json)、[Windows](https://github.com/Azure/aks-engine/blob/v0.55.0/examples/azure-stack/kubernetes-windows.json)) |
| v0.55.4 | AKS 基本 Ubuntu 16.04-LTS イメージ ディストリビューション、2020 年 9 月 (2020.09.14)、AKS 基本 Windows イメージ (17763.1397.200820) | 1.15.12、1.16.14、1.17.11 | API モデルのサンプル ([Linux](https://github.com/Azure/aks-engine/blob/v0.55.0/examples/azure-stack/kubernetes-azurestack.json)、[Windows](https://github.com/Azure/aks-engine/blob/v0.55.0/examples/azure-stack/kubernetes-windows.json)) |

## <a name="whats-new"></a>新機能

- Azure Stack の Linux VHD を 2020.09.14 に更新します #[3828](https://github.com/Azure/aks-engine/pull/3828)
- Azure Stack で K8s v1.17.11 のサポートを追加します [#3702](https://github.com/Azure/aks-engine/pull/3702)
- Azure Stack で K8s v1.16.14 のサポートを追加します [#3704](https://github.com/Azure/aks-engine/pull/3704)
- Linux VHD を 2020.09.14 に更新します [#3750](https://github.com/Azure/aks-engine/pull/3750)
- Windows VHD を 8 月に更新します [#3730](https://github.com/Azure/aks-engine/pull/3730)
- Kubernetes メトリック サーバーを v0.3.7 にアップグレードします [#3669](https://github.com/Azure/aks-engine/pull/3669)
- ログのローテーションの問題を修正するために Docker バージョンをアップグレードします [#3693](https://github.com/Azure/aks-engine/pull/3693)
- CoreDNS を v1.7.0 にアップグレードします [#3608](https://github.com/Azure/aks-engine/pull/3608)
- Moby 19.03.x パッケージを使用します [#3549](https://github.com/Azure/aks-engine/pull/3549)
- azure-cni の更新戦略の修正 [#3571](https://github.com/Azure/aks-engine/pull/3571)

## <a name="known-issues"></a>既知の問題

-   1 つのクラスター内に複数の Kubernetes サービスを並行してデプロイすると、基本的なロード バランサー構成でエラーが発生する可能性があります。 可能であれば、一度に 1 つのサービスをデプロイしてください。
-   aks-engine get-versions を実行すると、Azure および Azure Stack Hub に適用される情報が生成されますが、Azure Stack Hub に対応するものを明確に識別する方法はありません。 アップグレードに使用できるバージョンを特定する目的でこのコマンドを使用しないでください。 前述したアップグレードの参照一覧を使用してください。
-   aks-engine ツールは、Azure と Azure Stack Hub 全体の共有ソース コード リポジトリであるためです。 多くのリリース ノートと pull request を調べると、このツールは前述以外のバージョンの Kubernetes と OS プラットフォームをサポートしていることがわかります。これらを無視して、この更新プログラムの公式ガイドとして、前述のバージョン一覧を使用してください。

> [!NOTE]  
> Windows コンテナーと Azure CNI のサポートは、パブリック プレビューで利用できます。 プレビューへの参加を希望される場合は、[https://aka.ms/ash-k8s-win-azurecni](https://aka.ms/ash-k8s-win-azurecni) でアクセス権をリクエストできます。

## <a name="reference"></a>リファレンス

これは、Azure と Azure Stack Hub を組み合わせたリリース ノートの完全なセットです。

- https://github.com/Azure/aks-engine/releases/tag/v0.51.1
- https://github.com/Azure/aks-engine/releases/tag/v0.52.1
- https://github.com/Azure/aks-engine/releases/tag/v0.53.1
- https://github.com/Azure/aks-engine/releases/tag/v0.54.1
- https://github.com/Azure/aks-engine/releases/tag/v0.55.0
- https://github.com/Azure/aks-engine/releases/tag/v0.55.4

::: moniker-end
::: moniker range="<=azs-1910"
"*AKS エンジンのバージョン 0.48.0 以前に適用されます。* "

この記事では、Azure Stack Hub 上の Azure Kubernetes Service (AKS) エンジンの更新プログラムの内容について説明します。 この更新プログラムには、Azure Stack Hub プラットフォームを対象とした最新リリースの AKS engine の機能強化と修正が含まれています。 これは、グローバルな Azure 用の AKS エンジンに関するリリース情報を文書化するためのものではないことに注意してください。

## <a name="update-planning"></a>計画の更新

AKS エンジンのアップグレード コマンドを使用すると、クラスターのアップグレード プロセスが完全に自動化され、仮想マシン (VM)、ネットワーク、ストレージ、Kubernetes、オーケストレーション タスクが処理されます。 更新プログラムを適用する前に、この記事の情報を必ず確認してください。

### <a name="upgrade-considerations"></a>アップグレードの考慮事項

-   お使いのバージョンの AKS エンジンに適した Marketplace 項目である AKS 基本 Ubuntu 16.04-LTS イメージ ディストリビューションを使用していますか。 これらのバージョンについては、[新しい AKS ベースの Ubuntu イメージと AKS エンジンのバージョンのダウンロード](#download-new-image-and-aks-engine)に関するセクションを参照してください。

-   ターゲット クラスターに正しいクラスター仕様 (apimodel.json) とリソース グループを使用していますか。 最初にクラスターをデプロイしたときに、このファイルは出力ディレクトリに生成されました。 "deploy" コマンドのパラメーターについては、「[Kubernetes クラスターのデプロイ](./azure-stack-kubernetes-aks-engine-deploy-cluster.md#deploy-a-kubernetes-cluster)」を参照してください。

-   AKS エンジンを実行してアップグレード操作を実行するために信頼できるマシンを使用していますか。

-   クラスターに通常の負荷がかかっている前提では、アクティブなワークロードを使用して運用中のクラスターを更新する場合、ワークロードに影響を与えずにアップグレードを適用できます。 ただし、ユーザーをリダイレクトする必要がある場合に備えて、バックアップ クラスターが必要です。

-   可能であれば、Azure Stack Hub 環境内の VM からコマンドを実行して、ネットワーク ホップと接続障害の可能性を減らします。

-   サブスクリプションにプロセス全体のための十分なクォータがあることを確認します。 このプロセスでは、プロセス中に新しい VM が割り当てられます。 結果として得られる VM の数は元の VM と同じですが、このプロセス中に作成する VM をいくつか増やすことを計画してください。

-   システムの更新やスケジュールされたタスクは計画されていません。

-   運用クラスターと同じ値を使用して構成されたクラスターで段階的なアップグレードをセットアップし、運用クラスターでアップグレードを実行する前に、そこでアップグレードをテストします。

### <a name="use-the-upgrade-command"></a>upgrade コマンドを使用する

「[Azure Stack Hub で Kubernetes クラスターをアップグレードする](./azure-stack-kubernetes-aks-engine-upgrade.md)」の記事で説明されているように、AKS エンジンの "upgrade" コマンドを使用する必要があります。

### <a name="upgrade-interruptions"></a>アップグレードの中断

予期しない要因によってクラスターのアップグレードが中断されることがあります。 AKS エンジンからエラーが報告されたり、AKS エンジンの実行プロセスに何かが発生したりすると、中断が発生する可能性があります。 中断の原因を調べて対処し、同じアップグレード コマンドを再度送信して、アップグレード プロセスを続行します。 **upgrade** コマンドはべき等です。コマンドを再送信したら、クラスターのアップグレードを再開する必要があります。 通常、中断によって更新が完了するまでの時間は長くなりますが、その完了には影響しません。

### <a name="estimated-upgrade-time"></a>推定アップグレード時間

推定所要時間は、クラスター内の VM あたり 12 分から 15 分です。 たとえば、20 ノードのクラスターのアップグレードには、約 5 時間かかる場合があります。

## <a name="download-new-image-and-aks-engine"></a>新しいイメージと AKS エンジンをダウンロードする

AKS ベースの Ubuntu イメージと AKS エンジンの新しいバージョンをダウンロードします。

Azure Stack Hub 用 AKS エンジンのドキュメントで説明されているように、Kubernetes クラスターのデプロイには 2 つの主要コンポーネントが必要です。 
- aks-engine バイナリ
- AKS 基本 Ubuntu 16.04-LTS イメージ ディストリビューション

この更新プログラムでは、これらの新しいバージョンを使用できます。

-   Azure Stack Hub オペレーターは、新しい AKS ベースの Ubuntu イメージをダウンロードする必要があります。

    -   名前: `AKS Base Ubuntu 16.04-LTS Image Distro, March 2020`
    -   バージョン: `2020.03.19`
    -   「[Azure Kubernetes Services (AKS) エンジンの前提条件を Azure Stack Hub Marketplace に追加する](../operator/azure-stack-aks-engine.md)」の記事の指示に従ってください

-   Kubernetes クラスター管理者は、新しい aks-engine バージョン 0.48.0 をダウンロードする必要があります。 「[Azure Stack Hub の Linux に AKS エンジンをインストールする](./azure-stack-kubernetes-aks-engine-deploy-linux.md)」の記事の手順を参照してください。 クラスターを初めてインストールする場合と同じプロセスで実行できます。 この更新により、以前のバイナリが上書きされます。 たとえば、`get-akse.sh` スクリプトを使用した場合は、「[接続されている環境へのインストール](./azure-stack-kubernetes-aks-engine-deploy-linux.md#install-in-a-connected-environment)」の記事の説明と同じ手順を実行します。 Windows システムにインストールする場合も、「[Azure Stack Hub の Windows に AKS エンジンをインストールする](./azure-stack-kubernetes-aks-engine-deploy-windows.md)」と同じプロセスが適用されます。

## <a name="kubernetes-version-upgrade-path"></a>Kubernetes バージョンのアップグレード パス

次の Azure Stack Hub の表で、最新のバージョンとアップグレード バージョンを確認できます。 aks-engine `get-versions` コマンドには、グローバル Azure でサポートされているバージョンも含まれるため、従わないでください。 次のバージョンとアップグレードの表は、Azure Stack Hub の AKS エンジン クラスターに適用されます。

| **現在のバージョン** | **使用可能なアップグレード** |
| --------------------| ----------------------|
| 1.14.7 | 1.15.10 |
| 1.14.8 | 1.15.10 |
| 1.15.4 | 1.15.10 |
| 1.15.5 | 1.15.10 |

## <a name="whats-new"></a>新機能 

-   Kubernetes バージョン 1.15.10 のサポート ([\#2834](https://github.com/Azure/aks-engine/issues/2834))。 新しいクラスターをデプロイするときは、api モデルの json ファイル (つまり クラスター定義ファイル) でリリース バージョン番号だけでなくマイナー バージョン番号も指定します。 例については [kubernetes-azurestack.json](https://aka.ms/aksengine-json-example-raw) を参照してください。

    - `"orchestratorRelease": "1.15`,

    - `"orchestratorVersion": "1.15.10"`

    > [!NOTE]  
    > API モデルの json ファイルで Kubernetes のバージョンが明示的に指定されていない場合は、バージョン `1.15` が使用され ([\#2932](https://github.com/Azure/aks-engine/issues/2932))、orchestratorVersion は既定で ` 1.15.11` になります。クラスターのデプロイ中は、この結果がエラーになります。

-   aks-engine v0.43.1 では、受信要求の Azure Stack Hub Resource Manager しきい値制限がある場合、クラウド プロバイダーがその制御ループや他のタスクを実行するための既定の頻度設定は適切に機能しません。 この更新により Azure Stack Hub の既定値が変更され、Azure Stack Hub Resource Manager への再試行の負荷が軽減されます ([\#2861](https://github.com/Azure/aks-engine/issues/2861))。

-   Azure Stack Hub でサポートされていないプロパティが API モデルの json ファイルに含まれている場合、aks-engine の新しい検証手順により、実行が停止されるか警告が表示されます ([\#2717](https://github.com/Azure/aks-engine/issues/2717))。

-   aks-engine では、新しい検証チェックインにより、実行中の aks-engine のバージョンに必要な AKS ベース イメージのバージョンの可用性が検証されます ([\#2342](https://github.com/Azure/aks-engine/issues/2342))。 この処理は、API モデル ファイルの解析後、Azure Stack Hub Resource Manager を呼び出す前に発生します。

-   "upgrade" コマンドの新しい aks-engine オプションである "--control-plane-only" を使用すると、マスター仮想マシンのみを対象とする操作をアップグレードできます ([\#2635](https://github.com/Azure/aks-engine/issues/2635))。

-   Ubuntu 16.04-LTS 用の Linux カーネル バージョン 4.15.0-1071-azure への更新。 詳細については、「[パッケージ: linux-image-4.15.0-1071-azure (4.15.0-1071.76) \[security\]](https://packages.ubuntu.com/xenial/linux-image-4.15.0-1071-azure)」を参照してください。

-   Kubernetes バージョン 1.14.8 および 1.15.10 をサポートする新しい hyperkube 更新プログラム。

-   クラスターの Kubernetes のバージョンと一致するように kubectl を更新してください。 このコンポーネントは Kubernetes クラスターのマスター ノードで使用できます。これはマスターへの SSH で実行することができます。

-   最新の [2020 年 2 月リリース](https://github.com/microsoft/Docker-Provider/blob/ci_feature_prod/README.md)での Azure Container Monitor アドオンの更新 ([\#2850](https://github.com/Azure/aks-engine/issues/2850))。

-   `coredns` をバージョン v1.6.6 にアップグレードします ([\#2555](https://github.com/Azure/aks-engine/issues/2555))。

-   `etcd` をバージョン 3.3.18 にアップグレードします ([\#2462](https://github.com/Azure/aks-engine/issues/2462))。

-   `moby` をバージョン 3.0.11 にアップグレードします ([\#2887](https://github.com/Azure/aks-engine/issues/2887))。

-   このリリースでは、AKS エンジンは `k8s.gcr.io` からの依存関係をなくし、イメージの構築時に公式の `Kubernetes MCR registry @ mcr.microsoft.com` を使用するようになりました ([\#2722](https://github.com/Azure/aks-engine/issues/2722))。

## <a name="known-issues"></a>既知の問題

-  1 つのクラスター内に複数の Kubernetes サービスを並行してデプロイすると、基本的なロード バランサー構成でエラーが発生する可能性があります。 可能であれば、一度に 1 つのサービスをデプロイしてください。

-  aks-engine get-versions を実行すると、Azure および Azure Stack Hub に適用される情報が生成されますが、Azure Stack Hub に対応するものを明確に識別する方法はありません。 アップグレードに使用できるバージョン領域を特定する目的でこのコマンドを使用しないでください。 前述したアップグレードの参照一覧を使用してください。

-  aks-engine ツールは、Azure と Azure Stack Hub 全体の共有ソース コード リポジトリであるためです。 多くのリリース ノートとプル要求を調べると、このツールは前述以外のバージョンの Kubernetes と OS プラットフォームをサポートしていることがわかります。これらを無視して、この更新プログラムの公式ガイドとして、前述のバージョン一覧を使用してください。

## <a name="reference"></a>リファレンス

一部の修正されたバグ一覧と、0.44.0 から 0.48.0 までのリリース ノートの完全なセットを次に示します。後者の一覧には、Azure と Azure Stack Hub の項目が含まれることに注意してください。

### <a name="bug-fixes"></a>バグの修正

-   ウィンドウ `azure.json` の `userAssignedIdentityId` に引用符がありません ([\#2327](https://github.com/Azure/aks-engine/issues/2327))

-   アドオン `update config` はアップグレードのみです ([\#2282](https://github.com/Azure/aks-engine/issues/2282))

-   Windows ノードで管理 IP を取得する際の突然のタイムアウト ([\#2284](https://github.com/Azure/aks-engine/issues/2284))

-   1\.0.28 Azure CNI zip ファイルを Windows VHD に追加 ([\#2268](https://github.com/Azure/aks-engine/issues/2268))

-   IPAddressCount を設定するための正しい既定の順序 ([\#2358](https://github.com/Azure/aks-engine/issues/2358))

-   手動によるエラーを回避し、メンテナンスを簡易化するように、すべての k8s バージョンで 1 つの omsagent yaml を使用するように更新 ([\#2692](https://github.com/Azure/aks-engine/issues/2692))

### <a name="release-notes"></a>リリース ノート

これは、Azure と Azure Stack Hub を組み合わせたリリース ノートの完全なセットです

-   https://github.com/Azure/aks-engine/releases/tag/v0.44.0
-   https://github.com/Azure/aks-engine/releases/tag/v0.44.1
-   https://github.com/Azure/aks-engine/releases/tag/v0.44.2
-   https://github.com/Azure/aks-engine/releases/tag/v0.45.0
-   https://github.com/Azure/aks-engine/releases/tag/v0.46.0
-   https://github.com/Azure/aks-engine/releases/tag/v0.46.1
-   https://github.com/Azure/aks-engine/releases/tag/v0.46.2
-   https://github.com/Azure/aks-engine/releases/tag/v0.46.3
-   https://github.com/Azure/aks-engine/releases/tag/v0.47.0
-   https://github.com/Azure/aks-engine/releases/tag/v0.48.0
::: moniker-end
## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub 上の AKS エンジン](azure-stack-kubernetes-aks-engine-overview.md)を確認してください
