---
title: Set-AksHciConfig
author: jessicaguan
description: Set-AksHciConfig PowerShell コマンドは、Azure Kubernetes Service ホスト用の構成設定を更新します。
ms.topic: reference
ms.date: 2/12/2021
ms.author: jeguan
ms.openlocfilehash: 80a6fc50bbfcaf028d6c810bbef7e2d6b4508cab
ms.sourcegitcommit: b844c19d1e936c36a85f450b7afcb02149589433
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/04/2021
ms.locfileid: "101874607"
---
# <a name="set-akshciconfig"></a>Set-AksHciConfig

## <a name="synopsis"></a>構文
Azure Kubernetes Service ホストの構成を設定または更新します。

## <a name="syntax"></a>構文

### <a name="set-configuration-for-host"></a>ホストの構成を設定します
```powershell
Set-AksHciConfig [-imageDir <String>]
                 [-cloudConfigLocation <String>]
                 [-nodeConfigLocation <String>]
                 [-vnet <Virtual Network>]
                 [-controlPlaneVmSize <VmSize>]
                 [-sshPublicKey <String>]
                 [-macPoolStart <String>]
                 [-macPoolEnd <String>]       
                 [-proxyServerHTTP <String>]
                 [-proxyServerHTTPS <String>]
                 [-proxyServerNoProxy <String>]
                 [-proxyServerCertFile <String>]
                 [-proxyServerCredential <PSCredential>]
                 [-cloudServiceCidr <String>]
                 [-workingDir <String>]
                 [-version <String>]
                 [-nodeAgentPort <int>]
                 [-nodeAgentAuthorizerPort <int>]
                 [-clusterRoleName <String>]
                 [-cloudLocation <String>]
                 [-skipHostLimitChecks]
                 [-insecure]
                 [-skipUpdates]
                 [-forceDnsReplication]
                 [-enableDiagnosticData]   
```

## <a name="description"></a>説明
Azure Kubernetes Service ホストの構成を設定します。 2 から 4 ノードの Azure Stack HCI クラスターまたは Windows Server 2019 Datacenter フェールオーバー クラスターにデプロイする場合は、imageDir パラメーターおよび cloudConfigLocation パラメーターを指定する必要があります。 単一ノードの Windows Server 2019 Datacenter の場合、すべてのパラメーターは省略可能で、既定値に設定されます。 ただし、最適なパフォーマンスを得るには、2 から 4 ノードの Azure Stack HCI クラスターのデプロイを使用することをお勧めします。

## <a name="examples"></a>例

### <a name="to-deploy-on-a-2-4-node-cluster-with-dhcp-networking"></a>DHCP ネットワークを使用して 2 から 4 ノードのクラスターにデプロイするには
```powershell
PS C:\> Set-AksHciConfig -imageDir c:\clusterstorage\volume1\Images -cloudConfigLocation c:\clusterstorage\volume1\Config
```

### <a name="to-deploy-with-a-virtual-ip-pool"></a>仮想 IP プールを使用してデプロイするには
```powershell
PS C:\> New-AksHciNetworkSetting -name newNetwork -subnetCidr 192.168.2.0/32 -defaultGateway 192.168.2.1 -subnetVSwitch External -vipPoolStart 192.168.2.20 -vipPoolEnd 192.168.2.80   
PS C:\> Set-AksHciConfig -imageDir c:\clusterstorage\volume1\Images -cloudConfigLocation c:\clusterstorage\volume1\Config -networkSettings newNetwork
```

### <a name="to-deploy-with-a-proxy-server"></a>プロキシ サーバーを使用してデプロイするには
```powershell
PS C:\> $credential = Get-Credential
```

```powershell
PS C:\> Set-AksHciConfig -imageDir c:\clusterstorage\volume1\Images -cloudConfigLocation c:\clusterstorage\volume1\Config -proxyServerHttp "http://proxy.contoso.com:8888" -proxyServerHttps "http://proxy.contoso.com:8888" -proxyServerNoProxy "localhost,127.0.0.1,.svc,10.96.0.0/12,10.244.0.0/16,10.231.110.0/24,10.68.237.0/24" -proxyServerCredential $credential
```

## <a name="parameters"></a>パラメーター

### <a name="-imagedir"></a>-imageDir
Azure Stack HCI 上の Azure Kubernetes Service によって VHD イメージが格納されるディレクトリへのパス。 単一ノードのデプロイの場合、既定値は `%systemdrive%\AksHciImageStore` です。 "複数ノードのデプロイの場合、このパラメーターを指定する必要があります"。 パスは、共有ストレージ パス ( `C:\ClusterStorage\Volume2\ImageStore`  など) または SMB 共有 ( `\\FileShare\ImageStore` など) を指す必要があります。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: %systemdrive%\AksHciImageStore
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-cloudconfiglocation"></a>-cloudConfigLocation
クラウド エージェントによって構成が格納される場所。 単一ノードのデプロイの場合、既定値は `%systemdrive%\wssdcloudagent` です。 この場所は、上記の imageDir のパスと同じにすることができます。 "複数ノードのデプロイの場合、このパラメーターを指定する必要があります"。 パスは、共有ストレージ パス ( `C:\ClusterStorage\Volume2\ImageStore`  など) または SMB 共有 ( `\\FileShare\ImageStore` など) を指す必要があります。 ストレージに常にアクセスできるようにするために、この場所は可用性の高い共有に存在する必要があります。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: %systemdrive%\wssdcloudagent
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-nodeconfiglocation"></a>-nodeConfigLocation
ノード エージェントによって構成が格納される場所。 すべてのノードにノード エージェントがあるため、その構成はノードごとにローカルになります。 この場所はローカル パスである必要があります。 既定では、すべてのデプロイで `%systemdrive%\programdata\wssdagent` になります。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: %systemdrive%\programdata\wssdagent
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-vnet"></a>-vnet
New-AksHciNetworkSetting コマンドを使用して作成された AksHciNetworkSetting オブジェクトの名前。
```yaml
Type: VirtualNetwork
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-controlplanevmsize"></a>-controlPlaneVmSize
コントロール プレーン用に作成する VM のサイズ。 既定値は Standard_A4_V2 です。 使用できる VM サイズの一覧を取得するには、`Get-AksHciVmSize` を実行します。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: Standard_A4_V2
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-sshpublickey"></a>-sshPublicKey
SSH 公開キー ファイルへのパス。 この公開キーを使用すると、Azure Stack HCI デプロイ上の Azure Kubernetes Service によって作成された任意の VM にログインできます。 独自の SSH 公開キーを使用する場合は、ここでその場所を渡します。 キーを指定しなかった場合は、`%systemdrive%\akshci\.ssh\akshci_rsa`.pub でそれが検索されます。 そのファイルが存在しない場合は、この場所で SSH キー ペアが生成されて使用されます。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-macpoolstart"></a>-macPoolStart
これは、Azure Kubernetes Service ホスト VM に使用する MAC プールの MAC アドレスの先頭を指定するために使用されます。 MAC アドレスの構文では、先頭のバイトの最下位ビットが常に 0 である必要があります。また、先頭のバイトは常に偶数 (つまり、00、02、04、06...) である必要があります。一般的な MAC アドレスは、02:1E:2B:78:00:00 のようになります。 割り当てられる MAC アドレスが一貫性のあるものになるように、長期間維持されるデプロイには MAC プールを使用します。 これは、VM に特定の MAC アドレスが割り当てられる必要がある場合に便利です。 既定値は none です。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-macpoolend"></a>-macPoolEnd
これは、Azure Kubernetes Service ホスト VM に使用する MAC プールの MAC アドレスの末尾を指定するために使用されます。 MAC アドレスの構文では、先頭のバイトの最下位ビットが常に 0 である必要があります。また、先頭のバイトは常に偶数 (つまり、00、02、04、06...) である必要があります。-macPoolEnd として渡されたアドレスの先頭のバイトは、-macPoolStart として渡されたアドレスの先頭のバイトと同じである必要があります。 割り当てられる MAC アドレスが一貫性のあるものになるように、長期間維持されるデプロイには MAC プールを使用します。 これは、VM に特定の MAC アドレスが割り当てられる必要がある場合に便利です。 既定値は none です。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-proxyserverhttp"></a>-proxyServerHTTP
これにより、HTTP エンドポイントに接続する必要があるすべてのコンポーネントによって使用されるプロキシ サーバー URI が提供されます。 URI 形式には、URI スキーマ、サーバー アドレス、ポートが含まれます (つまり https://server.com:8888) )。 既定値は none です。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-proxyserverhttps"></a>-proxyServerHTTPS
これにより、HTTPS エンドポイントに接続する必要があるすべてのコンポーネントで使用されるプロキシサーバー URI が提供されます。 URI 形式には、URI スキーマ、サーバー アドレス、ポートが含まれます (つまり https://server.com:8888) )。 既定値は none です。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-proxyservernoproxy"></a>-proxyServerNoProxy
これは、プロキシから除外されるアドレスのコンマ区切りの文字列です。 既定値は localhost,127.0.0.1,.svc,10.96.0.0/12,10.244.0.0/16 です。 これにより、localhost トラフィック (localhost、127.0.0.1)、内部の Kubernetes サービス トラフィック (.svc)、Kubernetes Service CIDR (10.96.0.0/12)、および Kubernetes POD CIDR (10.244.0.0/16) がプロキシ サーバーから除外されます。 このパラメーターを使用して、サブネット範囲または名前の除外をさらに追加できます。 **このパラメーターの設定は非常に重要です。正しく構成されていないと、内部の Kubernetes クラスター トラフィックがプロキシに予期せずルーティングされる可能性があるためです。これにより、ネットワーク通信でさまざまなエラーが発生する可能性があります。**

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-proxyservercertfile"></a>-proxyServerCertFile
プロキシ サーバーに対する認証に使用される証明書。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:
 
Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-proxyservercredential"></a>-proxyServerCredential
これにより、HTTP または HTTPS プロキシ サーバーに対する認証のためのユーザー名とパスワードが提供されます。 このパラメーターに渡す PSCredential オブジェクトを生成するために `Get-Credential` を使用できます。 既定値は none です。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-cloudservicecidr"></a>-cloudServiceCidr
これを使用すると、MOC CloudAgent サービスに割り当てられる静的 IP/ネットワーク プレフィックスを指定できます。 この値は、CIDR 形式を使用して指定する必要があります (例: 192.168.1.2/16)。 (例: 192.168.1.2/16)。 これを指定すると、IP アドレスが変更されないため、ネットワーク上で重要なすべてのものに常にアクセスできるようにすることができます。 既定値は none です。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-workingdir"></a>-workingDir
これは、小さいファイルを格納するためにモジュールで使用する作業ディレクトリです。 既定値は `%PROGRAMFILES%\AksHci`  であり、ほとんどのデプロイでは変更できません。 既定値は変更しないことをお勧めします。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: %PROGRAMFILES%\AksHci
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-version"></a>-version
デプロイする Azure Stack HCI 上の Azure Kubernetes Service のバージョン。 既定値は最新バージョンです。 既定値は変更しないことをお勧めします。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: Latest version
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-nodeagentport"></a>-nodeAgentPort
ノード エージェントでリッスンする必要がある TCP/IP ポート番号。 既定値は 45000 です。 既定値は変更しないことをお勧めします。

```yaml
Type: System.Int32
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: 45000
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-nodeagentauthorizerport"></a>-nodeAgentAuthorizerPort
ノード エージェントで承認ポートに使用する必要がある TCP/IP ポート番号。 既定値は 45001 です。 既定値は変更しないことをお勧めします。 

```yaml
Type: System.Int32
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: 45001
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-clusterrolename"></a>-clusterRoleName
これにより、クラスター内の汎用サービスとしてクラウド エージェントを作成するときに使用する名前が指定されます。 これは、既定では、プレフィックス ca- と GUID サフィックスを持つ一意の名前になります (例: "ca-9e6eb299-bc0b-4f00-9fd7-942843820c26")。 既定値は変更しないことをお勧めします。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: A unique name with a prefix of ca- and a guid suffix
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-cloudlocation"></a>-cloudLocation
このパラメーターにより、Microsoft が運営するカスタム クラウドの場所の名前を提供します。 既定の名前は "MocLocation" です。 既定値は変更しないことをお勧めします。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: MocLocation
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-skiphostlimitchecks"></a>-skipHostLimitChecks
デプロイを続行する前に、メモリとディスク領域が使用可能であることを確認するために実行するすべてのチェックをスキップするよう、スクリプトに要求します。 この設定を使用することは推奨されません。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-insecure"></a>-insecure
セキュリティ保護されていないモード (接続が TLS によって保護されていない) で、クラウド エージェントやノード エージェントなどの Azure Kubernetes Service on Azure Stack HCI コンポーネントをデプロイします。  運用環境に、セキュリティ保護されていないモードを使用しないことをお勧めします。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-skipupdates"></a>-skipUpdates
利用可能な更新プログラムをスキップする場合は、このフラグを使用します。 この設定を使用することは推奨されません。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-forcednsreplication"></a>-forceDnsReplication
一部のシステムでは、DNS レプリケーションに最大 1 時間かかることがあります。 これにより、デプロイの速度が低下します。 この問題が発生した場合は、Install-AksHci がループに入っていることがわかります。 この問題を解決するには、このフラグを使用してみてください。 -forceDnsReplication フラグは、保証されている修正プログラムではありません。 フラグの背後にあるロジックが失敗した場合、エラーは非表示になり、フラグが指定されなかったかのようにコマンドが実行されます。

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### <a name="-enablediagnosticdata"></a>-enableDiagnosticData`

このフラグを使用して、必要な診断データを Microsoft に送信することに同意します。 これは、それ以外の場合に、`Set-AksHciConfig` によって提示される同意プロンプトをスキップする場合に便利です。 `Set-AksHciConfig` によって同意プロンプトが表示されたときに同意しないと、`Install-AksHci` は失敗します。
