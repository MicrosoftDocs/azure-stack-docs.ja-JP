---
title: Azure Stack Hub で Kubernetes クラスターをアップグレードする
description: Azure Stack Hub で Kubernetes クラスターをアップグレードする方法を学習します。
author: mattbriggs
ms.topic: article
ms.date: 3/4/2021
ms.author: mabrigg
ms.reviewer: waltero
ms.lastreviewed: 3/4/2021
ms.openlocfilehash: 0db454750b56e9c4dbb765092c48643b1df15470
ms.sourcegitcommit: ccc4ee05d71496653b6e27de1bb12e4347e20ba4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102231559"
---
# <a name="upgrade-a-kubernetes-cluster-on-azure-stack-hub"></a>Azure Stack Hub で Kubernetes クラスターをアップグレードする

AKS エンジンでは、ツールを使用して最初にデプロイされたクラスターをアップグレードすることができます。 AKS エンジンを使用してクラスターを維持できます。 メンテナンス タスクは、任意の IaaS システムと同様です。 新しい更新プログラムが利用可能かどうかを認識し、AKS エンジンを使用してそれらを適用します。
## <a name="upgrade-a-cluster"></a>クラスターのアップグレード

アップグレード コマンドを実行すると、Kubernetes のバージョンと OS の基本イメージが更新されます。 アップグレード コマンドを実行するたびに、AKS エンジンでは、クラスターのすべてのノードに対して、使用中の **aks-engine** のバージョンに関連付けられた AKS 基本イメージを使用して新しい VM が作成されます。 `aks-engine upgrade` コマンドを使用すると、クラスター内のすべてのマスター ノードとエージェント ノードの現行の状態を維持できます。 

Microsoft ではお客様のクラスターを管理しません。 ただし、Microsoft では、クラスターの管理に使用できるツールと VM イメージを用意しています。 

デプロイされたクラスターのアップグレードでは、以下がカバーされます。

-   Kubernetes
-   Azure Stack Hub Kubernetes プロバイダー
-   基本 OS

運用クラスターをアップグレードする場合は、次の点を考慮してください。

-   ターゲット クラスターに正しいクラスター仕様 (`apimodel.json`) とリソース グループを使用していますか?
-   AKS エンジンを実行してアップグレード操作を実行するクライアント マシンとして、信頼できるマシンを使用していますか?
-   バックアップ クラスターがあり、それが動作していることを確認します。
-   可能であれば、Azure Stack Hub 環境内の VM からコマンドを実行して、ネットワーク ホップと接続障害の可能性を減らします。
-   サブスクリプションにプロセス全体のための十分な領域があることを確認します。 このプロセスでは、プロセス中に新しい VM が割り当てられます。
-   システムの更新やスケジュールされたタスクは計画されていません。
-   運用クラスターとまったく同じように構成されたクラスターで段階的なアップグレードをセットアップし、運用クラスターでアップグレードを実行する前に、そこでアップグレードをテストします。

## <a name="steps-to-upgrade-to-a-newer-kubernetes-version"></a>新しい Kubernetes バージョンにアップグレードする手順

> [!NOTE]  
> さらに新しいバージョンの aks-engine を使用していて、マーケットプレースでイメージを入手できる場合は、AKS 基本イメージもアップグレードされます。

次の手順では、最小限の手順でアップグレードを実行します。 詳細については、「[Kubernetes クラスターのアップグレード](kubernetes-aks-engine-release-notes.md#aks-engine-and-azure-stack-version-mapping)」の記事を参照してください。

1. まず、アップグレードの対象となるバージョンを特定する必要があります。 このバージョンは現在お持ちになっているバージョンによって異なります。そのバージョン値を使用してアップグレードを実行します。 ご使用の AKS エンジンでサポートされている Kubernetes のバージョンを一覧表示するには、次のコマンドを実行します。
    
    ```bash
    aks-engine get-versions --azure-env AzureStackCloud
    ```
    
    AKS エンジン、AKS Base Image、および Kubernetes の各バージョンの完全なマッピングについては、「[サポートされている AKS エンジンのバージョン](kubernetes-aks-engine-release-notes.md#aks-engine-and-azure-stack-version-mapping)」を参照してください。

2. `upgrade` コマンドを実行するために必要な情報を収集します。 アップグレードでは、次のパラメーターが使用されます。

    | パラメーター | 例 | 説明 |
    | --- | --- | --- |
    | azure-env | AzureStackCloud | AKS エンジンに対して、ターゲット プラットフォームが Azure Stack Hub であることを示すには、`AzureStackCloud` を使用します。 |
    | location | local | Azure Stack Hub のリージョン名。 ASDK の場合、リージョンは `local` に設定されます。 |
    | resource-group | kube-rg | 新しいリソース グループの名前を入力するか、既存のリソース グループを選択します。 リソース名は、英数字かつ小文字にする必要があります。 |
    | subscription-id | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | サブスクリプション ID を入力します。 詳細については、「[プランへのサブスクライブ](./azure-stack-subscribe-services.md#subscribe-to-an-offer)」を参照してください。 |
    | api-model | ./kubernetes-azurestack.json | クラスター構成ファイルまたは API モデルへのパス。 |
    | client-id | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | サービス プリンシパル GUID を入力します。 Azure Stack Hub 管理者がサービス プリンシパルを作成したときにアプリケーション ID として識別されたクライアント ID。 |
    | client-secret | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | サービス プリンシパル シークレットを入力します。 これは、サービスの作成時に設定するクライアント シークレットです。 |
    | identity-system | adfs | 省略可能。 Active Directory フェデレーション サービス (AD FS) を使用している場合に、ID 管理ソリューションを指定します。 |

3. 値を指定して、次のコマンドを実行します。

    ```bash  
    aks-engine upgrade \
    --azure-env AzureStackCloud \
    --location <for an ASDK is local> \
    --resource-group kube-rg \
    --subscription-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --api-model kube-rg/apimodel.json \
    --upgrade-version 1.18.15 \
    --client-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --client-secret xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --identity-system adfs # required if using AD FS
    ```

4.  何らかの理由でアップグレード操作でエラーが発生した場合は、問題に対処した後でアップグレード コマンドを再実行できます。 AKS エンジンにより、前の時点で失敗した操作が再開されます。

## <a name="steps-to-only-upgrade-the-os-image"></a>OS イメージのみをアップグレードする手順

1. [サポートされている Kubernetes バージョン情報の一覧](kubernetes-aks-engine-release-notes.md#aks-engine-and-azure-stack-version-mapping)を確認し、アップグレードを計画している aks-engine と AKS 基本イメージのバージョンがあるかどうかを判断します。 aks-engine のバージョンを表示するには、`aks-engine version` を実行します。
2. 必要に応じて、aks-engine をインストールしたマシンで `./get-akse.sh --version vx.xx.x` を実行して AKS エンジンをアップグレードします。**x.xx.x** は、対象のバージョン情報に置き換えます。
3. 使用する予定の Azure Stack Hub Marketplace で必要な AKS 基本イメージのバージョンを追加するには、社内の Azure Stack Hub オペレーターに依頼します。
4. 既に使用しているものと同じバージョンの Kubernetes を使用し、`--force` を追加して `aks-engine upgrade` コマンドを実行します。 例については、「[アップグレードの強制](#forcing-an-upgrade)」を参照してください。


## <a name="steps-to-update-cluster-to-os-version-ubuntu-1804"></a>クラスターを OS バージョン Ubuntu 18.04 に更新する手順

AKS エンジン バージョン 0.60.1 以降では、クラスター VM を Ubuntu 16.04 から 18.04 にアップグレードできます。 次の手順に従います。

1. デプロイ中に生成された `api-model.json` ファイルを探して編集します。 これは、`aks-engine` でのアップグレードまたはスケーリング操作に使用されるのと同じファイルである必要があります。
2. `masterProfile` と `agentPoolProfiles` のセクションを見つけ、これらのセクション内で `distro` の値を `aks-ubuntu-18.04` に変更します。
2. [新しい Kubernetes バージョンにアップグレードする手順](#steps-to-upgrade-to-a-newer-kubernetes-version)で行う場合と同様に、`api-model.json` ファイルを保存し、` aks-engin upgrade` コマンドで `api-model.json` ファイルを使用します。

## <a name="forcing-an-upgrade"></a>アップグレードの強制

クラスターを強制的にアップグレードする必要がある場合があります。 たとえば、1 日目に最新の Kubernetes バージョンを使用して、切断された環境にクラスターをデプロイするとします。 次の日に、Ubuntu が、Microsoft の新しい **AKS 基本イメージ** によってもたらされる脆弱性に対する修正プログラムをリリースします。 既にデプロイしたのと同じ Kubernetes バージョンを使用してアップグレードを強制することで、この新しいイメージを適用できます。

```bash  
aks-engine upgrade \
--azure-env AzureStackCloud   
--location <for an ASDK is local> \
--resource-group kube-rg \
--subscription-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
--api-model kube-rg/apimodel.json \
--upgrade-version 1.18.15 \
--client-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
--client-secret xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
--force
```

手順については、[アップグレードの強制](https://github.com/Azure/aks-engine/blob/master/docs/topics/upgrade.md#force-upgrade)に関するセクションを参照してください。

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub 上の AKS エンジン](azure-stack-kubernetes-aks-engine-overview.md)を確認してください
- [Azure Stack Hub で Kubernetes クラスターをスケールする](azure-stack-kubernetes-aks-engine-scale.md)
