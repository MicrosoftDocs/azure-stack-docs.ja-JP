---
title: AKS on Azure Stack HCI 用のプロキシ設定を構成する
description: インターネットに接続するためのプロキシ サポートを計画および構成する方法について説明します。
author: abha
ms.topic: how-to
ms.date: 01/27/2021
ms.author: abha
ms.reviewer: ''
ms.openlocfilehash: 83a3a4928f5df1b6b4520cdc6ffd277e497425ff
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101874600"
---
# <a name="configure-proxy-settings-on-aks-on-azure-stack-hci"></a>AKS on Azure Stack HCI 上でプロキシ設定を構成する

このドキュメントでは、ネットワークでインターネットへの接続にプロキシ サーバーを使用する必要がある場合に、AksHci および WinInetProxy PowerShell モジュールを使用して AKS on Azure Stack HCI 上でプロキシ サポートを設定する際に必要な手順について説明します。 

WinInetProxy を使用しない場合は、**インターネットのプロパティ** (inetcpl.cpl) を使用し、 **[接続]** タブと **[LAN の設定]** をクリックして、プロキシ サーバーの設定を構成および検証することもできます。


## <a name="before-you-begin"></a>開始する前に

PowerShell 管理ウィンドウ内で次のコマンドを実行して、プロキシ サーバーを構成するための [WinInetProxy](https://www.powershellgallery.com/packages/WinInetProxy/0.1.0) PowerShell モジュールをインストールします。

```Powershell
Install-Module -Name WinInetProxy -Repository PSGallery
```

その他の前提条件については、[システム必要条件](./system-requirements.md)に関するページをご覧ください。 AksHci PowerShell モジュールをダウンロードするには、[AksHci PowerShell モジュールの設定](./setup-powershell.md)に関するページをご覧ください。

## <a name="configure-a-proxy-server-without-authentication"></a>認証なしでプロキシ サーバーを構成する

認証を必要としないプロキシ サーバーを構成するには、Azure Stack HCI クラスター ノードごとに次のコマンドを実行します。

```powershell
Set-WinInetProxy -ProxyServer http://proxy.contoso.com:3128 -ProxyBypass "local"
```

## <a name="configure-a-proxy-server-with-authentication"></a>認証を使用してプロキシ サーバーを構成する

認証を必要とするプロキシ サーバーを構成するには、WinINet プロキシで使用する AKS on Azure Stack ホストに対して資格情報を構成する必要があります。 これは、プロキシで必要な認証の種類によって異なり、NTLM/Kerberos 認証または基本認証のいずれかになります。

> [!NOTE]
> 証明書を使用してプロキシ サーバーに接続するには、その証明書を、ホスト上の適切な証明書ストアに対してプロビジョニングし、それが信頼されていることを確認する必要があります。

### <a name="configure-a-proxy-server-with-ntlmkerberos-authentication"></a>NTLM/Kerberos 認証を使用してプロキシ サーバーを構成する

以下のコマンドを使用して、新しい Windows 資格情報を追加します。 パスワードを求められたら、ご自身の安全なパスワードを入力します。 または、 **[Windows 資格情報]** の下の Windows Credential Manager を使用して、Windows 視覚情報に対して新しいエントリを追加すること もできます。 

```powershell
cmdkey /generic:proxy.contoso.com /user:username /pass
```
このコマンドを実行すると、パスワードを入力するように求められます。

PowerShell プロファイル内で、次のコマンドを追加して、キャッシュされた資格情報を AKS HCI エージェントが使用できるようにします。

```powershell
Set-WinInetProxy -ProxyServer http://proxy.contoso.com:3128 -ProxyBypass "local"
notepad $PROFILE
[System.Net.WebRequest]::DefaultWebProxy.Credentials = [System.Net.CredentialCache]::DefaultCredentials
```  

### <a name="configure-a-proxy-server-with-basic-authentication"></a>基本認証を使用してプロキシ サーバーを構成する

基本認証の資格情報を PowerShell プロファイルに追加して、ダウンロード エージェント モジュールが使用できるようにする必要があります。 

> [!NOTE]
> PowerShell プロファイルの資格情報は暗号化されず、クリア テキストで表示されます。 PowerShell プロファイルがアクセス制御リスト (ACL) によって保護されていることを確認してください。これにより、管理者と LocalSystem アカウントだけが、そのプロファイルを表示できるようになります。

次に示すように、PowerShell プロファイルを編集して、ユーザー名とパスワードを基本認証の資格情報に置き換えます。

```powershell
Set-WinInetProxy -ProxyServer http://proxy.contoso.com:3128 -ProxyBypass "local"
notepad $profile
[System.Net.WebRequest]::DefaultWebProxy.Credentials = new-object System.Net.NetworkCredential("username", "password")
```

## <a name="deploy-aks-on-azure-stack-hci-host-using-a-proxy-server"></a>プロキシ サーバーを使用して AKS on Azure Stack HCI ホストをデプロイする

プロキシ サーバーを構成したら、`Set-AksHciConfig` コマンドを使用して、AKS ホスト インストールに対してプロキシ構成を設定できます。 プロキシ サーバーが認証を必要とするかどうかに応じて、さまざまな手順があります。

以下に示すオプションを使用して、ご自身のデプロイを構成したら、[Azure Stack HCI 上で AKS ホストをインストール](./setup-powershell.md)するか、[PowerShell を使用して AKS クラスターを作成](./create-kubernetes-cluster-powershell.md)できます。

### <a name="configure-an-aks-host-for-a-proxy-server-with-basic-authentication"></a>基本認証を使用してプロキシ サーバーに対して AKS ホストを構成する  

プロキシ サーバーで認証が必要な場合は、次のコマンドを実行して資格情報を取得し、構成の詳細を設定します。

```powershell
$proxyCred = Get-Credential
Set-AksHciConfig -proxyServerHTTP "http://proxy.contoso.com:8888" -proxyServerHTTPS "http://proxy.contoso.com:8888" -proxyServerCredential $ProxyCred
```

## <a name="configure-an-aks-host-for-a-proxy-server-with-ntlmkerberos-authentication"></a>NTLM/Kerberos 認証を使用してプロキシ サーバーに対して AKS ホストを構成する

```powershell
$credential = Get-Credential # get the credential for the proxy server
Set-AksHciConfig -proxyServerHttp "http://proxy.contoso.com:8888" -proxyServerHttps "http://proxy.contoso.com:8888" -proxyServerCredential $credential
```

### <a name="configure-an-aks-host-for-a-proxy-server-without-authentication"></a>認証なしでプロキシ サーバーに対して AKS ホストを構成する  

お使いのプロキシ サーバーが認証を必要としない場合は、管理者として PowerShell を開き、次のコマンドを実行します。

```powershell
Set-AksHciConfig -proxyServerHTTP "http://proxy.contoso.com:8888" -proxyServerHTTPS "http://proxy.contoso.com:8888"
```

### <a name="configure-an-aks-host-for-a-proxy-server-with-a-trusted-certificate"></a>信頼された証明書を使用してプロキシ サーバーに対して AKS ホストを構成する

お使いのプロキシ サーバーにより、プロキシ クライアントが証明書を信頼することが求められている場合は、`Set-AksHciConfig` を実行するときに証明書ファイルを指定します。 証明書ファイルの形式は *Base-64 encoded X .509* です。 これにより、スタック全体で証明書をプロビジョニングし、信頼できるようになります。

```powershell
Set-AksHciConfig -proxyServerHTTP "http://proxy.contoso.com:8888" -proxyServerHTTPS "http://proxy.contoso.com:8888" -proxyServerCertFile "C:\proxycertificate.crt"
```

> [!NOTE]
> Windows Kubernetes ワーカー ノード上では、プロキシ証明書はまだプロビジョニング/信頼されていません。 Windows worker のサポートは、今後のリリースで有効になります。


## <a name="exclude-specific-hosts-or-domains-from-using-the-proxy-server"></a>特定のホストまたはドメインをプロキシ サーバーの使用から除外する

ほとんどのネットワークで、特定のネットワーク、ホスト、またはドメインを、プロキシ サーバー経由でのアクセスから除外する必要があります。 これを行うには、`Set-AksHciConfig` に `-proxyServerNoProxy` パラメーターを指定して、アドレス文字列を除外します。

`proxyServerNoProxy` の既定値は `localhost,127.0.0.1,.svc,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16` です

このコマンドを実行すると、次が除外されます。

- localhost トラフィック (localhost, 127.0.0.1)
- 内部 Kubernetes サービス トラフィック (.svc)。ここで、 _.svc_ はワイルドカード名を表します。 これは saying *.svc と似ていますが、no* はこのスキーマ内では使用されません。
- プライベート ネットワーク アドレス空間 (10.0.0.0/8,172.16.0.0/12,192.168.0.0/16)。 プライベート ネットワーク アドレス空間には、Kubernetes Service CIDR (10.96.0.0/12) や Kubernetes POD CIDR (10.244.0.0/16) などの重要なネットワークが含まれていることに注意してください。

これらの既定値は多くのネットワークで機能しますが、除外リストへのサブネットの範囲や名前の追加が必要になる場合があります。 たとえば、エンタープライズ名前空間 (.contoso.com) が、プロキシ経由で転送されないようにすることができます。 それには、proxyServerNoProxy リスト内で値を指定します。

```powershell
Set-AksHciConfig -proxyServerHttp "http://proxy.contoso.com:8888" -proxyServerHttps "http://proxy.contoso.com:8888" -proxyServerNoProxy "localhost,127.0.0.1,.svc,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16,.contoso.com"
```

## <a name="next-steps"></a>次のステップ

- [Kubernetes クラスターに Linux アプリケーションをデプロイします](./deploy-linux-application.md)。
- [Kubernetes クラスターに Windows アプリケーションをデプロイします](./deploy-windows-application.md)。
