---
title: Azure Stack HCI 用にファイアウォールを構成する
description: このトピックでは、Azure Stack HCI オペレーティング システム用にファイアウォールを構成する方法に関するガイダンスを提供します。
author: JohnCobb1
ms.author: v-johcob
ms.topic: how-to
ms.date: 02/12/2021
ms.openlocfilehash: 0bfd97b71774662ec11074951dcc956391d0fc65
ms.sourcegitcommit: 5ea0e915f24c8bcddbcaf8268e3c963aa8877c9d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/14/2021
ms.locfileid: "100487393"
---
# <a name="configure-firewalls-for-azure-stack-hci"></a>Azure Stack HCI 用にファイアウォールを構成する

>適用対象:Azure Stack HCI バージョン 20H2

このトピックでは、Azure Stack HCI オペレーティング システム用にファイアウォールを構成する方法に関するガイダンスを提供します。 接続の要件が含まれており、オペレーティング システムからアクセスする必要がある Azure 内の IP アドレスをサービス タグでグループ化する方法について説明しています。 このトピックでは、Microsoft Defender ファイアウォールを更新する手順も提供します。

## <a name="connectivity-requirements"></a>接続の要件
Azure Stack HCI は、定期的に Azure に接続する必要があります。 アクセスは次のものにのみ制限されます。
- 既知の Azure IP
- 送信方向
- ポート 443 (HTTPS)

詳細については、「[Azure Stack HCI の FAQ](../faq.md)」の「Azure Stack HCI の接続」セクションを参照してください

このトピックでは、許可リストに含まれているものを除くすべての送信先へのすべてのトラフィックをブロックするために、頻繁にロックダウンされるファイアウォールの構成を必要に応じて使用する方法について説明します。

   >[!IMPORTANT]
   > 送信接続が外部の企業のファイアウォールやプロキシ サーバーによって制限されている場合は、必ず、以下の表に一覧表示されている URL がブロックされないようにしてください。 関連情報については、「[Azure Arc 対応サーバー エージェントの概要](/azure/azure-arc/servers/agent-overview#networking-configuration)」の「ネットワーク構成」セクションを参照してください。


以下に示すように、Azure Stack HCI の場合、複数のファイアウォールを使用して Azure にアクセスします。

:::image type="content" source="./media/configure-firewalls/firewalls-diagram.png" alt-text="図は、ファイアウォールのポート 443 (HTTPS) を介してサービス タグ エンドポイントにアクセスする Azure Stack HCI を示しています。" lightbox="./media/configure-firewalls/firewalls-diagram.png":::

## <a name="working-with-service-tags"></a>サービス タグの使用
"*サービス タグ*" は、指定された Azure サービスからの IP アドレスのグループを表します。 Microsoft では、サービス タグに含まれる IP アドレスを管理し、IP アドレスが変更されるとサービス タグを自動的に更新して更新を最小限に抑えるようにしています。 詳細については、「[仮想ネットワーク サービス タグ](/azure/virtual-network/service-tags-overview)」を参照してください。

## <a name="required-endpoint-daily-access-after-azure-registration"></a>必要なエンドポイントの毎日のアクセス (Azure の登録後)
Azure では、サービス タグを使用してまとめられた Azure サービスの既知の IP アドレスを保持します。 Azure では、あらゆるサービスのすべての IP アドレスの週単位の JSON ファイルを発行します。 IP アドレスは頻繁には変更されませんが、年に数回変更されます。 次の表は、オペレーティング システムからアクセスする必要があるサービス タグ エンドポイントを示しています。

| 説明                   | IP 範囲のサービス タグ  | [URL]                                                                                 |
| :-----------------------------| :-----------------------  | :---------------------------------------------------------------------------------- |
| Azure Active Directory        | AzureActiveDirectory      | `https://login.microsoftonline.com`<br> `https://graph.microsoft.com`               |
| Azure Resource Manager        | AzureResourceManager      | `https://management.azure.com`                        |
| Azure Stack HCI クラウド サービス | AzureFrontDoor.Frontend   | `https://azurestackhci.azurefd.net` |
| Azure Arc                     | AzureArcInfrastructure<br> AzureTrafficManager | 使用する機能によって異なります。<br> ハイブリッド ID サービス: `*.his.arc.azure.com`<br> ゲスト構成: `*.guestconfiguration.azure.com`<br> **注:** より多くの機能を有効にすると、URL が増えることが予想されます。 |

## <a name="update-microsoft-defender-firewall"></a>Microsoft Defender ファイアウォールを更新する
このセクションでは、サービス タグに関連付けられた IP アドレスでオペレーティング システムに接続できるように Microsoft Defender ファイアウォールを構成する方法を示します。

1. 次のリソースから、オペレーティング システムを実行しているターゲット コンピューターに JSON ファイルをダウンロードします: [Azure IP 範囲とサービス タグ – パブリック クラウド](https://www.microsoft.com/download/details.aspx?id=56519)。

1. 次の PowerShell コマンドを使用して JSON ファイルを開きます。

    ```powershell
    $json = Get-Content -Path .\ServiceTags_Public_20201012.json | ConvertFrom-Json
    ```

1. "AzureResourceManager" サービス タグなど、特定のサービス タグの IP アドレス範囲の一覧を取得します。

    ```powershell
    $IpList = ($json.values | where Name -Eq "AzureResourceManager").properties.addressPrefixes
    ```

1. 許可リストを使用する場合は、外部の企業のファイアウォールに IP アドレスの一覧をインポートします。

1. クラスター内の各サーバーに対してファイアウォール規則を作成し、IP アドレス範囲の一覧への送信 443 (HTTPS) トラフィックを許可します。

    ```powershell
    New-NetFirewallRule -DisplayName "Allow Azure Resource Manager" -RemoteAddress $IpList -Direction Outbound -LocalPort 443 -Protocol TCP -Action Allow -Profile Any -Enabled True
    ```

## <a name="additional-endpoint-for-one-time-azure-registration"></a>1 回限りの Azure 登録用の追加のエンドポイント
Azure の登録プロセス中に、`Register-AzStackHCI` を実行するか、Windows Admin Center を使用すると、Az や AzureAD などの必要な PowerShell モジュールの最新バージョンがあることを確認するために、コマンドレットによって PowerShell ギャラリーへの接続が試行されます。

PowerShell ギャラリーは Azure でホストされていますが、現在はそのためのサービス タグはありません。 インターネットにアクセスできないためにサーバー ノードから `Register-AzStackHCI` コマンドレットを実行できない場合は、管理コンピューターにモジュールをダウンロードしてから、コマンドレットを実行するサーバー ノードにそれらを手動で転送することをお勧めします。

## <a name="set-up-a-proxy-server"></a>プロキシ サーバーを設定する
Azure Stack HCI のプロキシ サーバーを設定するには、次の PowerShell コマンドを管理者として実行します。

```powershell
Set-WinInetProxy -ProxySettingsPerUser 0 -ProxyServer webproxy1.com:9090
```

既定では、`ProxySettingsPerUser 0` フラグを使用して、プロキシ構成をユーザーごとではなくサーバー全体にします。 

次の場所で WinInetProxy.psm1 スクリプトをダウンロードします:[PowerShell ギャラリー | WinInetProxy.psm1 0.1.0](https://www.powershellgallery.com/packages/WinInetProxy/0.1.0/Content/WinInetProxy.psm1)。

## <a name="network-port-requirements"></a>ネットワーク ポートの要件
サイト内およびサイト間 (ストレッチ クラスターの場合) の両方のすべてのサーバー ノード間で、適切なネットワーク ポートが開いていることを確認します。 ファイアウォールとルーターには、クラスター内のすべてのサーバー間で ICMP、SMB (ポート 445、および SMB ダイレクト用のポート 5445)、WS-MAN (ポート 5985) の双方向トラフィックを許可するための適切な規則が必要になります。

Windows Admin Center で [クラスターの作成] ウィザードを使用してクラスターを作成すると、クラスター内の各サーバーで、フェールオーバー クラスタリング、Hyper-V、記憶域レプリカ用の該当するファイアウォール ポートがウィザードによって自動的に開かれます。 各サーバーで異なるファイアウォールを使用している場合は、次のポートを開きます。

### <a name="failover-clustering-ports"></a>フェールオーバー クラスタリングのポート
- ICMPv4 および ICMPv6
- TCP ポート 445
- RPC 動的ポート
- TCP ポート 135
- TCP ポート 137
- TCP ポート 3343
- UDP ポート 3343

### <a name="hyper-v-ports"></a>Hyper-V ポート
- TCP ポート 135
- TCP ポート 80 (HTTP 接続)
- TCP ポート 443 (HTTPS 接続)
- TCP ポート 6600
- TCP ポート 2179
- RPC 動的ポート
- RPC エンドポイント マッパー
- TCP ポート 445

### <a name="storage-replica-ports-stretched-cluster"></a>ストレージ レプリカ ポート (ストレッチ クラスター)
- TCP ポート 445
- TCP 5445 (iWarp RDMA を使用している場合)
- TCP ポート 5985
- ICMPv4 および ICMPv6 (`Test-SRTopology` PowerShell コマンドレットを使用している場合)

## <a name="next-steps"></a>次のステップ
詳細については、次のトピックも参照してください。
- 「[Azure Stack HCI の FAQ](../faq.md)」の接続に関するセクション