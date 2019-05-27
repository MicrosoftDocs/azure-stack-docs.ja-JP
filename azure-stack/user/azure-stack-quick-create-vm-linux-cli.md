---
title: Azure Stack で Azure CLI を使用して Linux 仮想マシンを作成する | Microsoft Docs
description: Azure Stack で CLI を使用して、Linux 仮想マシンを作成します。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
ms.date: 05/16/2019
ms.author: mabrigg
ms.custom: mvc
ms.lastreviewed: 01/14/2019
ms.openlocfilehash: 19c856bdf981775b0b3a8ee923046b51e6ef79ca
ms.sourcegitcommit: 889fd09e0ab51ad0e43552a800bbe39dc9429579
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/16/2019
ms.locfileid: "65782787"
---
# <a name="quickstart-create-a-linux-server-virtual-machine-using-azure-cli-in-azure-stack"></a>クイック スタート:Azure Stack で Azure CLI を使用して Linux サーバー仮想マシンを作成する

*適用対象:Azure Stack 統合システムと Azure Stack Development Kit*

Azure CLI を使用して、Ubuntu Server 16.04 LTS 仮想マシンを作成することができます。 この記事の手順に従って仮想マシンを作成し、使用します。 この記事では、以下のことを実行する手順も示します。

* リモート クライアントを使用して仮想マシンに接続する。
* NGINX Web サーバーをインストールし、既定のホーム ページを表示します。
* 使用されていないリソースをクリーンアップします。

## <a name="prerequisites"></a>前提条件

* **Azure Stack Marketplace 内の Linux イメージ**

   Azure Stack Marketplace には、既定では Linux イメージが含まれていません。 必要な **Ubuntu Server 16.04 LTS** イメージを提供する Azure Stack オペレーターを取得します。 オペレーターは、「[Azure から Azure Stack に Marketplace の項目をダウンロードする](../operator/azure-stack-download-azure-marketplace-item.md)」という記事に記載されている手順を使用できます。

* リソースを作成して管理するため、Azure Stack には Azure CLI の特定のバージョンが必要です。 Azure Stack 用に構成された Azure CLI がない場合は、[開発キット](../asdk/asdk-connect.md#connect-to-azure-stack-using-rdp) (または [VPN 経由で接続](../asdk/asdk-connect.md#connect-to-azure-stack-using-vpn)している場合は、Windows ベースの外部クライアント) にサインインし、[Azure CLI のインストールと構成](azure-stack-version-profiles-azurecli2.md)の手順に従います。

* Windows ユーザー プロファイルの .ssh ディレクトリに保存された id_rsa.pub という名前の SSH 公開キー。 SSH キーの作成の詳細については、「[SSH 公開キーの使用方法](azure-stack-dev-start-howto-ssh-public-key.md)」を参照してください。

## <a name="create-a-resource-group"></a>リソース グループの作成

リソース グループは、Azure Stack リソースのデプロイと管理を行うことができる論理コンテナーです。 開発キットまたは Azure Stack 統合システムから、[az group create](/cli/azure/group#az-group-create) コマンドを実行してリソース グループを作成します。

> [!NOTE]
> 値は、コード例のすべての変数に割り当てられます。 ただし、必要に応じて新しい値を割り当てることができます。

次の例では、myResourceGroup という名前のリソース グループをローカルの場所に作成します。 

```cli
az group create --name myResourceGroup --location local
```

## <a name="create-a-virtual-machine"></a>仮想マシンの作成

[az vm create](/cli/azure/vm#az-vm-create) コマンドを使用して仮想マシンを作成します。 次の例では、myVM という名前の VM を作成します。 この例では、管理者ユーザー名に Demouser、管理者パスワードとして Demouser@123 を使用します。 これらの値を、環境に適した内容に更新します。

```cli
az vm create \
  --resource-group "myResourceGroup" \
  --name "myVM" \
  --image "UbuntuLTS" \
  --admin-username "Demouser" \
  --admin-password "Demouser@123" \
  --location local
```

**PublicIpAddress** パラメーターでパブリック IP アドレスが返されます。 仮想マシンを使用する際に必要になるため、このアドレスは書き留めておいてください。

## <a name="open-port-80-for-web-traffic"></a>Web トラフィック用にポート 80 を開く

この仮想マシンでは IIS Web サーバーを実行することになるため、インターネット トラフィックに対してポート 80 を開く必要があります。 [az vm open-port](/cli/azure/vm) コマンドを使用して、目的のポートを開きます。 

```cli
az vm open-port --port 80 --resource-group myResourceGroup --name myVM
```

## <a name="use-ssh-to-connect-to-the-virtual-machine"></a>SSH を使用して仮想マシンに接続する

SSH がインストールされたクライアント コンピューターから、仮想マシンに接続します。 Windows クライアントを使用している場合は、[Putty](https://www.putty.org/) を使用して接続を作成できます。 仮想マシンに接続するには、次のコマンドを使用します。

```bash
ssh <publicIpAddress>
```

## <a name="install-the-nginx-web-server"></a>NGINX Web サーバーのインストール

パッケージ リソースを更新し、最新の NGINX パッケージをインストールするため、次のスクリプトを実行します。

```bash
#!/bin/bash

# update package source
apt-get -y update

# install NGINX
apt-get -y install nginx
```

## <a name="view-the-nginx-welcome-page"></a>NGINX のようこそページの表示

NGINX がインストールされ、仮想マシン上のポート 80 が開かれたので、その仮想マシンのパブリック IP アドレスを使用して Web サーバーにアクセスできます。 ブラウザーを開き、```http://<public IP address>``` に移動します。

![NGINX Web サーバーのようこそページ](./media/azure-stack-quick-create-vm-linux-cli/nginx.png)

## <a name="clean-up-resources"></a>リソースのクリーンアップ

不要になったリソースをクリーンアップします。 これらのリソースを削除するには、[az group delete](/cli/azure/group#az-group-delete) コマンドを使用できます。 リソース グループとそのすべてのリソースを削除するには、次のコマンドを実行します。

```cli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>次の手順

このクイック スタートでは、Web サーバーがインストールされた基本の Linux サーバー仮想マシンをデプロイしました。 Azure Stack 仮想マシンの詳細については、「[Azure Stack の仮想マシンに関する考慮事項](azure-stack-vm-considerations.md)」に進んでください。
