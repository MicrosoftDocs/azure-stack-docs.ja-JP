---
title: Azure Stack クイック スタート - VM ポータルの作成
description: Azure Stack クイック スタート - ポータルを使用した Linux VM の作成
services: azure-stack
cloud: azure-stack
author: mattbriggs
manager: femila
ms.service: azure-stack
ms.topic: quickstart
ms.date: 03/11/2019
ms.author: mabrigg
ms.reviewer: kivenkat
ms.custom: mvc
ms.lastreviewed: 12/03/2018
ms.openlocfilehash: 37ff24258b12c9b042c7b0dc5a113a62d1d1ec60
ms.sourcegitcommit: 41927cb812e6a705d8e414c5f605654da1fc6952
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/25/2019
ms.locfileid: "64477362"
---
# <a name="quickstart-create-a-linux-server-virtual-machine-with-the-azure-stack-portal"></a>クイック スタート: Azure Stack ポータルで Linux サーバー仮想マシンを作成する

*適用対象:Azure Stack 統合システムと Azure Stack Development Kit*

Azure Stack ポータルを使用して、Ubuntu Server 16.04 LTS 仮想マシンを作成できます。 この記事の手順に従って仮想マシンを作成し、使用します。 この記事では、以下のことを実行する手順も示します。

* リモート クライアントを使用して仮想マシンに接続する。
* NGINX Web サーバーをインストールする。
* リソースをクリーンアップする。

> [!NOTE]  
> この記事の画面の画像は、Azure Stack バージョン 1808 で導入された変更に合わせて更新されます。 1808 では、アンマネージド ディスクに加え、"*マネージド ディスク*" の使用のサポートが追加されています。 以前のバージョンを使用する場合は、ディスクの選択など、タスクの一部の画像が、この記事に示されているものとは異なります。  


## <a name="prerequisites"></a>前提条件

* **Azure Stack Marketplace 内の Linux イメージ**

   Azure Stack Marketplace には、既定では Linux イメージが含まれていません。 マーケットプレースで Azure Stack オペレーターに **Ubuntu Server 16.04 LTS** イメージを提供していることを確認してください。 オペレーターは、「[Azure から Azure Stack に Marketplace の項目をダウンロードする](../operator/azure-stack-download-azure-marketplace-item.md)」という記事に記載されている手順を使用できます。

* **SSH クライアントへのアクセス**

   Azure Stack Development Kit (ASDK) を使用している場合は、SSH クライアントにアクセスできない可能性があります。 クライアントが必要な場合は、SSH クライアントが含まれるパッケージがいくつかあります。 たとえば、PuTTY には SSH クライアントと SSH キージェネレーター (puttygen.exe) が含まれています。 利用できるパッケージの詳細については、Azure の次の記事を参照してください。「[SSH 公開キーの使用方法](azure-stack-dev-start-howto-ssh-public-key.md)」。

   このクイック スタートでは、PuTTY を使用して SSH キーを生成し、Linux サーバー仮想マシンに接続します。 PuTTY をダウンロードしてインストールするには、[https://www.putty.org/](https://www.putty.org) にアクセスします。

## <a name="create-an-ssh-key-pair"></a>SSH キー ペアの作成

この記事のすべての手順を完了するには、SSH キー ペアが必要です。 既存の SSH キー ペアがある場合は、この手順をスキップできます。

1. PuTTY のインストール フォルダー (既定の場所は ```C:\Program Files\PuTTY```) に移動し、```puttygen.exe``` を実行します。
2. [PuTTY Key Generator] ウィンドウで、**[Type of key to generate] \(生成するキーの種類)** が **[RSA]** に設定され、**[Number of bits in a generated key] \(生成されるキーのビット数)** が **[2048]** に設定されていることを確認します。 準備ができたら、**[Generate]\(生成\)** をクリックします。

   ![PuTTY Key Generator の構成](media/azure-stack-quick-linux-portal/Putty01.PNG)

3. キーを生成するには、PuTTY Key Generator ウィンドウでマウス カーソルをランダムに移動します。
4. キーの生成が完了したら、**[Save public key] \(公開キーを保存する)** および **[Save private key] \(秘密キーを保存する)** をクリックしてキーをファイルに保存します。

   ![PuTTY Key Generator の結果](media/azure-stack-quick-linux-portal/Putty02.PNG)

## <a name="sign-in-to-the-azure-stack-portal"></a>Azure Stack ポータルへのサインイン

Azure Stack ポータルにサインインします。 Azure Stack ポータルのアドレスは、接続している Azure Stack 製品によって異なります。

* Azure Stack Development Kit (ASDK) の場合は、 https://portal.local.azurestack.external にアクセスします。
* Azure Stack 統合システムの場合は、Azure Stack オペレーターによって提供された URL にアクセスします。

## <a name="create-the-virtual-machine"></a>仮想マシンの作成

1. Azure Stack ポータルの左上隅にある **[リソースの作成]** をクリックします。

2. **[コンピューティング]**、**[Ubuntu Server 16.04 LTS]** の順に選択します。
   
   ![Linux サーバーの選択](media/azure-stack-quick-linux-portal/select.png)
1. **Create** をクリックしてください。

4. 仮想マシンの情報を入力します。 **[認証の種類]** には **[SSH 公開キー]** を選択します。 保存した SSH 公開キーを貼り付け、**[OK]** をクリックします。

   > [!NOTE]
   > キーの先頭または末尾の空白を削除します。

   ![基本パネル - 仮想マシンの構成](media/azure-stack-quick-linux-portal/linux-01.PNG)

5. 仮想マシンとして **[D1]** を選択します。

   ![サイズパネル - 仮想マシンのサイズを選択する](media/azure-stack-quick-linux-portal/linux-02.PNG)

6. **[設定]** ページで、必要に応じて既定値を変更します。
   
   - Azure Stack バージョン 1808 以降では、**ストレージ**を構成できます。ここで "*マネージド ディスク*" の使用を選択できます。 バージョン 1808 より前は、アンマネージド ディスクのみを使用できます。    
     ![マネージド ディスクのストレージを構成する](media/azure-stack-quick-linux-portal/linux-03.PNG)
    
     お使いの構成の準備が整ったら、**[OK]** を選択して続行します。

7. **[概要]** ページで、**[OK]** をクリックして仮想マシンの展開を開始します。  
   ![デプロイする](media/azure-stack-quick-linux-portal/deploy.png)

## <a name="connect-to-the-virtual-machine"></a>仮想マシンへの接続

1. 仮想マシンのページで **[接続]** をクリックします。 仮想マシンに接続するために必要な SSH 接続文字列を見つけることができます。 

2. PuTTY を開きます。

3. [PuTTY Configuration]\(PuTTY の構成\) 画面で、**[Category]\(カテゴリ\)** ウィンドウを使用して上下にスクロールします。 **[SSH]** までスクロールして **[SSH]** を展開し、**[Auth]\(認証\)** をクリックします。**[参照]** をクリックし、保存した秘密キー ファイルを選択します。
   ![仮想マシンを接続する](media/azure-stack-quick-linux-portal/putty03.PNG)

4. **[Category]\(カテゴリ\)** ウィンドウで上にスクロールし、**[Session]\(セッション\)** をクリックします。
5. **[Host Name (or IP address)]\(ホスト名 (または IP アドレス)\)** ボックスで、Azure Stack ポータルに表示されている接続文字列を貼り付けます。 この例では、この文字列は ```asadmin@192.168.102.34``` です。

   ![PuTTY 構成接続文字列](media/azure-stack-quick-linux-portal/Putty04.PNG)

6. **[開く]** をクリックして、仮想マシンへのセッションを開きます。

   ![Linux セッション](media/azure-stack-quick-linux-portal/Putty05.PNG)

## <a name="install-the-nginx-web-server"></a>NGINX Web サーバーのインストール

次の bash コマンドを使用してパッケージ ソースを更新し、仮想マシンに最新の NGINX パッケージをインストールします。

```bash
#!/bin/bash

# update package source
sudo apt-get -y update

# install NGINX
sudo apt-get -y install nginx
```

NGINX のインストールが完了したら、SSH セッションを終了して Azure Stack ポータルの仮想マシンの [概要] ページを開きます。

## <a name="open-port-80-for-web-traffic"></a>Web トラフィック用にポート 80 を開く

受信トラフィックと送信トラフィックのセキュリティは、ネットワーク セキュリティ グループ (NSG) で確保します。 Azure Stack ポータルから仮想マシンが作成されると、SSH 接続用のポート 22 上の受信規則が作成されます。 この仮想マシンは Web サーバーをホストするため、ポート 80 上で Web トラフィックを許可するには NSG 規則を作成する必要があります。

1. 仮想マシンの **[概要]** ページで、**[リソース グループ]** の名前をクリックします。
2. 仮想マシンの**ネットワーク セキュリティ グループ**を選択します。 NSG は **[種類]** 列で確認できます。
3. 左側のメニューの **[設定]** で、**[受信セキュリティ規則]** をクリックします。
4. **[追加]** をクリックします。
5. **[名前]** で「**http**」と入力します。 **[ポート範囲]** が 80 に設定されていることと、**[アクション]** が **[許可]** に設定されていることを確認します。
6. **[OK]**

## <a name="view-the-nginx-welcome-page"></a>NGINX のようこそページの表示

NGINX がインストールされ、仮想マシン上のポート 80 が開かれたので、その仮想マシンのパブリック IP アドレスを使用して Web サーバーにアクセスできます。 (パブリック IP アドレスは仮想マシンの [概要] ページに表示されます)。

Web ブラウザーを開き、```http://<public IP address>``` を参照します。

![NGINX Web サーバーのようこそページ](media/azure-stack-quick-linux-portal/linux-05.PNG)

## <a name="clean-up-resources"></a>リソースのクリーンアップ

不要になったリソースをクリーンアップします。 仮想マシンとそのリソースを削除するには、仮想マシン ページでリソース グループを選択し、**[削除]** をクリックします。

## <a name="next-steps"></a>次の手順

このクイック スタートでは、Web サーバーがインストールされた基本の Linux サーバー仮想マシンをデプロイしました。 Azure Stack 仮想マシンの詳細については、「[Azure Stack の仮想マシンに関する考慮事項](azure-stack-vm-considerations.md)」に進んでください。
