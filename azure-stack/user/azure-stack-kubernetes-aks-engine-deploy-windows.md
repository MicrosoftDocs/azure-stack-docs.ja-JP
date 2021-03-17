---
title: Azure Stack Hub の Windows に AKS エンジンをデプロイする
description: Kubernetes クラスターをデプロイおよび管理するために、Azure Stack Hub の Windows マシンを使用して AKS エンジンをホストする方法について説明します。
author: mattbriggs
ms.topic: article
ms.date: 3/4/2021
ms.author: mabrigg
ms.reviewer: waltero
ms.lastreviewed: 3/4/2021
ms.openlocfilehash: a8525cb770bb833d338887e010a365bb411e8721
ms.sourcegitcommit: ccc4ee05d71496653b6e27de1bb12e4347e20ba4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102231474"
---
# <a name="install-the-aks-engine-on-windows-in-azure-stack-hub"></a>Azure Stack Hub の Windows に AKS エンジンをインストールする

Kubernetes クラスターをデプロイおよび管理するには、Azure Stack Hub の Windows マシンを使用して AKS エンジンをホストできます。 この記事では、お使いの接続および切断されている両 Azure Stack Hub インスタンスのクラスターを管理するためのクライアント VM の準備、インストールの確認、ASDK でのクライアント VM の設定について説明します。

## <a name="prepare-the-client-vm"></a>クライアント VM の準備

AKS エンジンとは、お使いの Kubernetes クラスターをデプロイおよび管理するために使用するコマンドライン ツールです。 このエンジンは、お使いの Azure Stack Hub のコンピューターで実行できます。 このコンピューターから AKS エンジンを実行して、お使いのクラスターの実行に必要な IaaS リソースとソフトウェアをデプロイします。 その後、このエンジンを実行しているコンピューターを使用して、お使いのクラスターで管理タスクを実行できます。

お使いのクライアント コンピューターを選択する場合には、次の点を考慮してください。

1. 障害が発生した場合にクライアント コンピューターを回復可能にする必要があるか。
3. クライアント コンピューターにどのように接続するかと、コンピューターがどのようにご自分のクラスターと通信するか。

## <a name="install-in-a-connected-environment"></a>接続されている環境へのインストール

クライアント VM をインストールし、インターネットに接続されている Azure Stack Hub 上のお使いの Kubernetes クラスターを管理できます。

1. Azure Stack Hub で Windows VM を作成します。 手順については、「[クイック スタート:Azure Stack Hub ポータルを使用して Windows サーバー VM を作成する](./azure-stack-quick-windows-portal.md)」を参照してください。
2. お使いの VM に接続します。
3. [PowerShell の手順に従って Chocolatey をインストールします](https://chocolatey.org/install#install-with-powershellexe)。 

    Chocolatey の Web サイトによると:Chocolatey は、Windows のパッケージ マネージャーです。例えば、Windows 用の apt-get や yum などです。 必要なアプリケーションとツールを迅速にインストールするための分散型フレームワークとして設計されています。 現在 PowerShell を使用している NuGet インフラストラクチャ上に構築されており、ディストリビューションからドア、err、コンピューターへのパッケージの配信に重点を置いています。
4. [AKS エンジンと Azure Stack のバージョン マッピング](kubernetes-aks-engine-release-notes.md#aks-engine-and-azure-stack-version-mapping) テーブルで、AKS エンジンのバージョンを確認します。 この AKS ベースのエンジンは、ご自分の Azure Stack Hub の Marketplace で入手できるようになっている必要があります。 コマンドを実行するときに、バージョン `--version v0.xx.x` を指定する必要があります。 バージョンを指定しないと、このコマンドによって最新バージョンがインストールされ、最新バージョンに必要な VHD イメージがご自分の Marketplace にはない可能性があります。
    > [!NOTE]  
    > Azure Stack Hub と AKS エンジンのバージョン番号のマッピングについては、[AKS エンジンのリリース ノート](kubernetes-aks-engine-release-notes.md#aks-engine-and-azure-stack-version-mapping)をご覧ください。
5. 管理者特権でのプロンプトで次のコマンドを実行し、バージョン番号を含めます。

    ```PowerShell  
        choco install aks-engine --version 0.xx.x -y
    ```

> [!NOTE]  
> このインストール方法が失敗する場合は、[切断されている環境](#install-in-a-disconnected-environment)の手順を試すか、別のパッケージ マネージャーである [GoFish](azure-stack-kubernetes-aks-engine-troubleshoot.md#try-gofish) を試してください。

## <a name="install-in-a-disconnected-environment"></a>切断されている環境へのインストール

インターネットから切断されている Azure Stack Hub 上のお使いの Kubernetes クラスターは、クライアント VM をインストールして管理できます。

1.  インターネットにアクセスできるコンピューターから、GitHub [Azure/aks-engine](https://github.com/Azure/aks-engine/releases/latest) に移動します。 `aks-engine-v0.xx.x-windows-amd64.tar.gz` など、Windows マシンのアーカイブ (*.tar.gz) をダウンロードします。 [サポート対象の Kubernetes バージョンの表](kubernetes-aks-engine-release-notes.md#aks-engine-and-azure-stack-version-mapping)で、その AKS エンジンのバージョンを確認します。

2.  お使いの Azure Stack Hub インスタンスにストレージ アカウントを作成し、AKS エンジン バイナリを使用してアーカイブ ファイル (*.tar.gz) をアップロードします。 Azure Storage Explorer の使用方法については、[Azure Stack Hub と Azure Storage Explorer](./azure-stack-storage-connect-se.md) に関するページを参照してください。

3. Azure Stack Hub で Windows VM を作成します。 手順については、「[クイック スタート:Azure Stack Hub ポータルを使用して Windows サーバー VM を作成する](./azure-stack-quick-windows-portal.md)」を参照してください

4.  アーカイブ ファイル (* tar.gz) をアップロードした Azure Stack Hub ストレージ アカウントの BLOB URL から、お使いの管理 VM にファイルをダウンロードします。 コマンド プロンプトから、アクセス権のあるディレクトリにアーカイブを抽出します。

5. お使いの VM に接続します。

6. [PowerShell の手順に従って Chocolatey をインストールします](https://chocolatey.org/install#install-with-powershellexe)。 

7.  管理者特権でのプロンプトで、次のコマンドを実行します。 適切なバージョン番号を含めます。

    ```PowerShell  
        choco install aks-engine --version 0.xx.x -y
    ```

## <a name="verify-the-installation"></a>インストールの確認

お使いのクライアント VM がセットアップされたら、AKS エンジンがインストールされていることを確認します。

1. お使いのクライアント VM に接続します。
2. 次のコマンドを実行します。

    ```PowerShell  
    aks-engine version
    ```

クライアント VM に AKS エンジンがインストールされていることを確認できない場合は、[AKS エンジンのインストールのトラブルシューティング](azure-stack-kubernetes-aks-engine-troubleshoot.md)に関するページを参照してください。


## <a name="asdk-installation"></a>ASDK のインストール

ASDK の外部のマシン上の ASDK で AKS エンジン用のクライアント VM を実行する場合には、証明書を追加する必要があります。 ASDK 環境内で Windows VM を使用している場合、マシンでは既に ASDK 証明書が信頼されています。 クライアント マシンが ASDK の外部にある場合は、ASDK から証明書を抽出し、Windows マシンに追加する必要があります。

ご自分の Azure Resource Manager エンドポイントが自己署名証明書を使用する ASDK を使用している場合、コンピューターの信頼された証明書ストアにこの証明書を明示的に追加する必要があります。 ASDK のルート証明書は、ASDK にデプロイするすべての VM にあります。

1. CA ルート証明書をエクスポートします。 手順については、「[Azure Stack Hub の CA ルート証明書をエクスポートする](../asdk/asdk-cli.md#export-the-azure-stack-hub-ca-root-certificate)」を参照してください。
2. Azure Stack Hub の CA ルート証明書を信頼します。 手順については、「[Azure Stack Hub の CA ルート証明書を信頼する](../asdk/asdk-cli.md#trust-the-certificate)」を参照してください。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [AKS エンジンを使用して Azure Stack Hub に Kubernetes クラスターをデプロイする](azure-stack-kubernetes-aks-engine-deploy-cluster.md)
