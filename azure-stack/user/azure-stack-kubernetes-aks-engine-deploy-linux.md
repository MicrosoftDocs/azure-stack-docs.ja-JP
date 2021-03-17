---
title: Azure Stack Hub の Linux に AKS エンジンをインストールする
description: Kubernetes クラスターをデプロイおよび管理するために、Azure Stack Hub の Linux マシンを使用して AKS エンジンをホストする方法について説明します。
author: mattbriggs
ms.topic: article
ms.date: 3/4/2021
ms.author: mabrigg
ms.reviewer: waltero
ms.lastreviewed: 3/4/2021
ms.openlocfilehash: 949a1ede3d2fc217217219c59f055b5311322576
ms.sourcegitcommit: ccc4ee05d71496653b6e27de1bb12e4347e20ba4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102231525"
---
# <a name="install-the-aks-engine-on-linux-in-azure-stack-hub"></a>Azure Stack Hub の Linux に AKS エンジンをインストールする

Kubernetes クラスターをデプロイおよび管理するには、Azure Stack Hub の Linux マシンを使用して AKS エンジンをホストできます。 この記事では、お使いの接続および切断されている両 Azure Stack Hub インスタンスのクラスターを管理するためのクライアント VM の準備、インストールの確認、ASDK でのクライアント VM の設定について説明します。

## <a name="prepare-the-client-vm"></a>クライアント VM の準備

AKS エンジンとは、お使いの Kubernetes クラスターをデプロイおよび管理するために使用するコマンドライン ツールです。 このエンジンは、お使いの Azure Stack Hub のコンピューターで実行できます。 このコンピューターから AKS エンジンを実行して、お使いのクラスターの実行に必要な IaaS リソースとソフトウェアをデプロイします。 その後、このエンジンを実行しているコンピューターを使用して、お使いのクラスターで管理タスクを実行できます。

お使いのクライアント コンピューターを選択する場合には、次の点を考慮してください。

1. 障害が発生した場合にクライアント コンピューターを回復可能にする必要があるか。
2. クライアント コンピューターにどのように接続するかと、コンピューターがどのようにご自分のクラスターと通信するか。

## <a name="install-in-a-connected-environment"></a>接続されている環境へのインストール

クライアント VM をインストールし、インターネットに接続されている Azure Stack Hub 上のお使いの Kubernetes クラスターを管理できます。

1. お使いの Azure Stack Hub に Linux VM を作成します。 手順については、「[クイック スタート:Azure Stack Hub ポータルを使用して Linux サーバー VM を作成します](./azure-stack-quick-linux-portal.md)。
2. お使いの VM に接続します。
3. [AKS エンジンと Azure Stack のバージョン マッピング](kubernetes-aks-engine-release-notes.md#aks-engine-and-azure-stack-version-mapping) テーブルで、AKS エンジンのバージョンを確認します。 AKS ベースのイメージは、ご自身の Azure Stack Hub の Marketplace で入手できるようになっている必要があります。 コマンドを実行するときに、バージョン `--version v0.xx.x` を指定する必要があります。 バージョンを指定しないと、このコマンドによって最新バージョンがインストールされます。最新バージョンに必要な VHD イメージがご自分の Marketplace にはない可能性があります。
    > [!NOTE]  
    > Azure Stack Hub と AKS エンジンのバージョン番号のマッピングについては、[AKS エンジンのリリース ノート](kubernetes-aks-engine-release-notes.md#aks-engine-and-azure-stack-version-mapping)をご覧ください。
1. 次のコマンドを実行します。

    ```bash  
        curl -o get-akse.sh https://raw.githubusercontent.com/Azure/aks-engine/master/scripts/get-akse.sh
        chmod 700 get-akse.sh
        ./get-akse.sh --version v0.xx.x
    ```

    > [!NOTE]  
    > Azure Stack Hub と AKS エンジンのバージョン番号のマッピングについては、[AKS エンジンのリリース ノート](kubernetes-aks-engine-release-notes.md#aks-engine-and-azure-stack-version-mapping)をご覧ください。

    > [!NOTE]  
    > インストール方法が失敗する場合は、[切断されている環境](#install-in-a-disconnected-environment)の手順を試すか、別のパッケージ マネージャーである [GoFish ](azure-stack-kubernetes-aks-engine-troubleshoot.md#try-gofish)を試してください。

## <a name="install-in-a-disconnected-environment"></a>切断されている環境へのインストール

インターネットから切断されている Azure Stack Hub 上のお使いの Kubernetes クラスターは、クライアント VM をインストールして管理できます。

1.  インターネットにアクセスできるコンピューターから、GitHub [Azure/aks-engine](https://github.com/Azure/aks-engine/releases/latest) に移動します。 `aks-engine-v0.xx.x-linux-amd64.tar.gz` など、Linux コンピューターのアーカイブ (*.tar.gz) をダウンロードします。 [サポート対象の Kubernetes バージョンの表](kubernetes-aks-engine-release-notes.md#aks-engine-and-azure-stack-version-mapping)で、その AKS エンジンのバージョンを確認します。

2.  お使いの Azure Stack Hub インスタンスにストレージ アカウントを作成し、AKS エンジン バイナリを使用してアーカイブ ファイル (*.tar.gz) をアップロードします。 Azure Storage Explorer の使用方法については、[Azure Stack Hub と Azure Storage Explorer](./azure-stack-storage-connect-se.md) に関するページを参照してください。

3. お使いの Azure Stack Hub に Linux VM を作成します。 手順については、「[クイック スタート:Azure Stack Hub ポータルを使用して Linux サーバー VM を作成します](./azure-stack-quick-linux-portal.md)。

3.  アーカイブ ファイル (* tar.gz) をアップロードした Azure Stack Hub ストレージ アカウントの blob URL から、お使いの管理 VM にファイルをダウンロードします。 アーカイブをディレクトリ `/usr/local/bin` に抽出します。

4. お使いの VM に接続します。

5.  次のコマンドを実行します。

    ```bash  
    curl -o aks-engine-v0.xx.x-linux-amd64.tar.gz <httpurl/aks-engine-v0.xx.x-linux-amd64.tar.gz>
    tar xvzf aks-engine-v0.xx.x-linux-amd64.tar.gz -C /usr/local/bin
    ```

## <a name="verify-the-installation"></a>インストールの確認

お使いのクライアント VM がセットアップされたら、AKS エンジンがインストールされていることを確認します。

1. お使いのクライアント VM に接続します。
2. 次のコマンドを実行します。

   ```bash  
   aks-engine version
   ```

3. Azure Resource Manager エンドポイントで自己署名証明書が使用されている場合、コンピューターの信頼された証明書ストアにルート証明書を明示的に追加する必要があります。 VM のルート証明書はディレクトリ /var/lib/waagent/Certificates.pem にあります。 次のコマンドを使用して証明書ファイルをコピーします。 

   ```bash
   sudo cp /var/lib/waagent/Certificates.pem /usr/local/share/ca-certificates/azurestackca.crt 
   sudo update-ca-certificates
   ```

クライアント VM に AKS エンジンがインストールされていることを確認できない場合は、[AKS エンジンのインストールのトラブルシューティング](azure-stack-kubernetes-aks-engine-troubleshoot.md)に関するページを参照してください。


## <a name="asdk-installation"></a>ASDK のインストール

ASDK の AKS エンジン用のクライアント VM を実行する場合には、証明書を追加する必要があります。

ご自分の Azure Resource Manager エンドポイントが自己署名証明書を使用する ASDK を使用している場合、コンピューターの信頼された証明書ストアにこの証明書を明示的に追加する必要があります。 ASDK のルート証明書は、ASDK にデプロイするすべての VM にあります。 たとえば、Ubuntu VM では、このディレクトリは `/var/lib/waagent/Certificates.pem` にあります。 

次のコマンドを使用して証明書ファイルをコピーします。

```bash
sudo cp /var/lib/waagent/Certificates.pem /usr/local/share/ca-certificates/azurestackca.crt

sudo update-ca-certificates
```

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [AKS エンジンを使用して Azure Stack Hub に Kubernetes クラスターをデプロイする](azure-stack-kubernetes-aks-engine-deploy-cluster.md)
