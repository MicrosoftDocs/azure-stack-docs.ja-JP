---
title: SDN Express を使用して SDN インフラストラクチャをデプロイする
description: SDN Express を使用して SDN インフラストラクチャをデプロイする方法を説明します
author: v-dasis
ms.topic: how-to
ms.date: 02/17/2021
ms.author: v-dasis
ms.reviewer: JasonGerend
ms.openlocfilehash: e367602252207a673316caf3482d7805bff02ba8
ms.sourcegitcommit: 4c97ed2caf054ebeefa94da1f07cfb6be5929aac
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/18/2021
ms.locfileid: "100647812"
---
# <a name="deploy-an-sdn-infrastructure-using-sdn-express"></a>SDN Express を使用して SDN インフラストラクチャをデプロイする

> 適用対象: Azure Stack HCI バージョン 20H2

このトピックでは、SDN Express PowerShell スクリプトを使用して、エンドツーエンドのソフトウェア定義ネットワーク (SDN) インフラストラクチャをデプロイします。 インフラストラクチャには、高可用性 (HA) ネットワーク コントローラー (NC)、高可用性ソフトウェア ロード バランサー (SLB)、高可用性ゲートウェイ (GW) が含まれる場合があります。  スクリプトでは段階的デプロイがサポートされており、ネットワーク コントローラー コンポーネントのみをデプロイして、最小限のネットワーク要件に対応するコア セットの機能を実現できます。

System Center Virtual Machine Manager (VMM) を使用して SDN インフラストラクチャをデプロイすることもできます。 詳細については、「[VMM ファブリックで SDN リソースを管理する](/system-center/vmm/network-sdn)」を参照してください。

## <a name="before-you-begin"></a>開始する前に

SDN のデプロイを始める前に、物理およびホストのネットワーク インフラストラクチャを計画して構成します。 次の記事をご覧ください。

- [物理ネットワークの要件](../concepts/physical-network-requirements.md)
- [ホスト ネットワークの要件](../concepts/host-network-requirements.md)
- [Windows Admin Center を使用してクラスターを作成する](../deploy/create-cluster.md)
- [Windows PowerShell を使用してクラスターを作成する](../deploy/create-cluster-powershell.md)
- [ソフトウェア定義ネットワーク インフラストラクチャを計画する](../concepts/plan-software-defined-networking-infrastructure.md)

すべての SDN コンポーネントをデプロイする必要はありません。 「[ソフトウェア定義ネットワーク インフラストラクチャを計画する](../concepts/plan-software-defined-networking-infrastructure.md)」の「[段階的デプロイ](../concepts/plan-software-defined-networking-infrastructure.md#phased-deployment)」セクションを参照して、必要なインフラストラクチャ コンポーネントを決定した後、それに従ってスクリプトを実行します。

すべてのホスト サーバーに Azure Stack HCI オペレーティング システムがインストールされていることを確認してください。 「[Azure Stack HCI オペレーティング システムのデプロイ](../deploy/operating-system.md)」で、これを実行する方法をご覧ください。

## <a name="requirements"></a>要件

SDN を正常にデプロイするには、次の要件が満たされている必要があります。

- すべてのホスト サーバーで Hyper-V が有効になっている
- すべてのホスト サーバーが Active Directory に参加している
- 仮想スイッチが作成されている
- 物理ネットワークが、構成ファイルで定義されているサブネットと VLAN に対して構成されている
- 構成ファイルで指定された VHDX ファイルが、SDN Express スクリプトが実行されているデプロイ コンピューターから到達可能である

## <a name="create-the-vdx-file"></a>VDX ファイルを作成する

SDN によって、Azure Stack HCI オペレーティング システム (OS) が含まれる VHDX ファイルが、SDN 仮想マシン (VM) を作成するためのソースとして使用されます。 VHDX 内の OS のバージョンは、Azure Stack HCI の Hyper-V ホストで使用されているバージョンと一致している必要があります。 この VHDX ファイルは、すべての SDN インフラストラクチャ コンポーネントによって使用されます。

ISO から Azure Stack HCI OS をダウンロードしてインストールしている場合は、[Convert-WindowsImage](https://www.powershellgallery.com/packages/Convert-WindowsImage/10.0) ユーティリティを使用して VHDX を作成できます。

`Convert-WindowsImage` の使用例を次に示します。

 ```powershell
$wimpath = "d:\sources\install.wim"
$vhdpath = "c:\temp\WindowsServerDatacenter.vhdx"
$Edition = 4   # 4 = Full Desktop, 3 = Server Core

import-module ".\convert-windowsimage.ps1"

Convert-WindowsImage -SourcePath $wimpath -Edition $Edition -VHDPath $vhdpath -SizeBytes 500GB -DiskLayout UEFI
```

## <a name="download-the-github-repository"></a>GitHub リポジトリをダウンロードする

SDN Express スクリプト ファイルは GitHub 内にあります。 最初の手順として、必要なファイルとフォルダーをお使いのデプロイ コンピューターで取得します。

1. [Microsoft SDN GitHub](https://github.com/microsoft/SDN) リポジトリに移動します。

1. リポジトリで、 **[Code]\(コード\)** ドロップダウン リストを展開し、 **[Clone]\(クローン\)** または **[Download ZIP]\(ZIP のダウンロード\)** を選択して、指定したデプロイ コンピューターに SDN ファイルをダウンロードします。

    > [!NOTE]
    > 指定したデプロイ コンピューターでは、Windows Server 2016 以降が実行されている必要があります。

1. ZIP ファイルを展開し、`SDNExpress` フォルダーをデプロイ コンピューターの `C:\` フォルダーにコピーします。

## <a name="edit-the-configuration-file"></a>構成ファイルを編集する

PowerShell `MultiNodeSampleConfig.psd1` 構成データ ファイルには、さまざまなパラメーターと構成設定の入力として SDN Express スクリプトに必要なすべてのパラメーターと設定が含まれています。 このファイルには、ネットワーク コントローラー コンポーネントだけをデプロイするのか、ソフトウェア ロード バランサーとゲートウェイ コンポーネントも同様にデプロイするのかに基づいて、何を入力する必要があるかについての具体的な情報が含まれています。 詳細については、「[ソフトウェア定義ネットワーク インフラストラクチャを計画する](../concepts/plan-software-defined-networking-infrastructure.md)」をご覧ください。

`C:\SDNExpress\scripts` フォルダーに移動し、好みのテキスト エディターで `MultiNodeSampleConfig.psd1` ファイルを開きます。 特定のパラメーター値を、インフラストラクチャとデプロイに合わせて変更します。

### <a name="general-settings-and-parameters"></a>全般設定とパラメーター

この設定とパラメーターは、SDN によってすべてのデプロイで一般的に使用されます。

- **VHDPath** -すべての SDN インフラストラクチャ VM (NC、SLB、GW) によって使用される VHD ファイル パス
- **VHDFile** -すべての SDN インフラストラクチャ VM (NC、SLB、GW) によって使用される VHD ファイル名
- **VMLocation** - SDN インフラストラクチャ VM へのファイル パス
- **JoinDomain** - SDN インフラストラクチャ VM が参加しているドメイン
- **SDNMacPoolStart** - クライアント ワークロード VM の開始 MAC プール アドレス
- **SDNMacPoolEnd** - クライアント ワークロード VM の終了 MAC プール アドレス
- **ManagementSubnet** - Hyper-V ホスト、SLB、GW コンポーネントを管理するために NC によって使用される管理ネットワーク サブネット
- **ManagementGateway** - 管理ネットワークのゲートウェイ アドレス
- **ManagementDNS** - 管理ネットワークの DNS サーバー
- **ManagementVLANID** - 管理ネットワークの VLAN ID サーバー
- **DomainJoinUsername** - 管理者のユーザー名
- **LocalAdminDomainUser** - 管理者のパスワード
- **RestName** - NC と通信するために管理クライアント (Windows Admin Center など) によって使用される DNS 名
- **HyperVHosts** - ネットワーク コントローラーによって管理されるホスト サーバー
- **NCUsername** - ネットワーク コントローラー アカウントのユーザー名
- **ProductKey** - SDN インフラストラクチャ VM のプロダクト キー
- **SwitchName** - Hyper-V ホストに複数の仮想スイッチが存在する場合にのみ必要
- **VMMemory** - インフラストラクチャ VM に割り当てるメモリ (GB)。 既定値は 4 GB です
- **VMProcessorCount** - インフラストラクチャ VM に割り当てるプロセッサの数。 既定値は 8 です
- **Locale** - 指定しないと、デプロイ コンピューターのロケールが使用されます
- **TimeZone** - 指定しないと、デプロイ コンピューターのロケールが使用されます

テキストでエンコードされてセキュリティ保護された文字列として暗号化されて格納される場合は、必要に応じてパスワードを含めることができます。  パスワードは、パスワードが暗号化されたのと同じコンピューター上で SDN Express スクリプトが実行される場合にのみ使用されます。そうでない場合は、パスワードの入力を求められます。

- **DomainJoinSecurePassword** - ドメイン アカウントの場合
- **LocalAdminSecurePassword** - ローカル管理者アカウントの場合
- **NCSecurePassword** - ネットワーク コントローラー アカウントの場合

> [!NOTE]
> SDN Express スクリプトでは、DHCP アドレス指定はサポートされていません。 すべての SDN インフラストラクチャ VM の MAC アドレスが、`SDNMACPool` パラメーターの範囲の外側にあることを確認します。

### <a name="network-controller-vm-section"></a>ネットワーク コントローラー VM セクション

`NCs = @()` セクションは、ネットワーク コントローラー VM に使用されます。 各 NC VM の MAC アドレスが、全般設定で指定されている `SDNMACPool` の範囲の外側にあることを確認します。

- **ComputerName** - NC VM の名前
- **HostName** - NC VM が配置されているサーバーのホスト名
- **ManagementIP** - NC VM の管理ネットワーク IP アドレス
- **MACAddress** - NC VM の MAC アドレス

### <a name="software-load-balancer-vm-section"></a>ソフトウェア ロード バランサー VM セクション

`Muxes = @()` セクションは、SLB VM に使用されます。 各 SLB VM の MAC アドレスが、全般設定で指定されている `SDNMACPool` の範囲の外側にあることを確認します。 SLB コンポーネントをデプロイしない場合は、このセクションを空のままにします (`Muxes = @()`)。

- **ComputerName** - SLB VM の名前
- **HostName** - SLB VM が配置されているサーバーのホスト名
- **ManagementIP** - SLB VM の管理ネットワーク IP アドレス
- **MACAddress** - SLB VM の MAC アドレス
- **PAIPAddress** - SLB VM のプロバイダー ネットワーク IP アドレス (PA)
- **PAMACAddress** - SLB VM のプロバイダー ネットワーク IP アドレス (PA)

### <a name="gateway-vm-section"></a>ゲートウェイ VM セクション

`Gateways = @()` セクションは、ゲートウェイ VM に使用されます。 各ゲートウェイ VM の MAC アドレスが、全般設定で指定されている `SDNMACPool` の範囲の外側にあることを確認します。 ゲートウェイ コンポーネントをデプロイしない場合は、このセクションを空のままにします (`Gateways = @()`)。

- **ComputerName** - ゲートウェイ VM の名前
- **HostName** - ゲートウェイ VM が配置されているサーバーのホスト名
- **ManagementIP** - ゲートウェイ VM の管理ネットワーク IP アドレス
- **MACAddress** - ゲートウェイ VM の MAC アドレス
- **FrontEndIp** - ゲートウェイ VM のプロバイダー ネットワーク フロントエンド IP アドレス
- **FrontEndMac** - ゲートウェイ VM のプロバイダー ネットワーク フロントエンド MAC アドレス
- **BackEndMac** - ゲートウェイ VM のプロバイダー ネットワーク バックエンド MAC アドレス

### <a name="additional-settings-for-slb-and-gateway"></a>SLB とゲートウェイの追加設定

次の追加パラメーターは、SLB とゲートウェイの VM によって使用されます。 SLB またはゲートウェイの VM をデプロイしない場合は、これらの値を空白のままにします。

- **SDNASN** - ネットワーク スイッチとのピアリングのために SDN によって使用される自律システム番号 (ASN)
- **RouterASN** - ゲートウェイ ルーターの ASN
- **RouterIPAddress** - ゲートウェイ ルーターの IP アドレス
- **PrivateVIPSubnet** - プライベート サブネットの仮想 IP アドレス (VIP)
- **PublicVIPSubnet** - パブリック サブネットの仮想 IP アドレス

次の追加パラメーターは、ゲートウェイ VM によってのみ使用されます。 ゲートウェイの VM をデプロイしない場合は、これらの値を空白のままにします。

- **PoolName** - すべてのゲートウェイ VM によって使用されるプール名
- **GRESubnet** - GRE の VIP サブネット (GRE 接続を使用している場合)
- **Capacity** - プール内の各ゲートウェイ VM の容量 (Kbps)

### <a name="settings-for-tenant-overlay-networks"></a>テナント オーバーレイ ネットワークの設定

次のパラメーターは、テナント用にオーバーレイ仮想化ネットワークをデプロイして管理する場合に使用されます。 代わりにネットワーク コントローラーを使用して従来の VLAN ネットワークを管理している場合は、これらの値は空白のままにしてかまいません。

- **PASubnet** - プロバイダー アドレス (PA) ネットワークのサブネット
- **PAVLANID** - PA ネットワークの VLAN ID
- **PAGateway** - PA ネットワーク ゲートウェイの IP アドレス
- **PAPoolStart** - PA ネットワーク プールの開始 IP アドレス
- **PAPoolEnd** - PA ネットワーク プールの終了 IP アドレス

## <a name="run-the-deployment-script"></a>展開スクリプトを実行する

SDN Express スクリプトにより、指定した SDN インフラストラクチャがデプロイされます。 スクリプトが完了すると、SDN インフラストラクチャは VM ワークロードのデプロイに使用できるようになります。

1. `README.md` ファイルで、デプロイ スクリプトの実行方法に関する最新情報を確認します。  

1. クラスター ホスト サーバーの管理者資格情報を持つユーザー アカウントから、次のコマンドを実行します。

    ```powershell
    SDNExpress\scripts\SDNExpress.ps1 -ConfigurationDataFile MultiNodeSampleConfig.psd1 -Verbose
    ```

1. NC VM が作成された後、DNS サーバー上のネットワーク コントローラーのクラスター名用に動的な DNS 更新を構成します。 詳細については、[DNS 動的更新を構成する方法](https://docs.microsoft.com/troubleshoot/windows-server/networking/configure-dns-dynamic-updates-windows-server-2003)に関するページを参照してください。

## <a name="next-steps"></a>次のステップ

VM を管理します。 詳細については、[VM の管理](../manage/vm.md)に関するページを参照してください。
