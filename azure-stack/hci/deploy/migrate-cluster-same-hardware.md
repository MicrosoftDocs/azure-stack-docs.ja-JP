---
title: 同じハードウェア上の Azure Stack HCI に移行する
description: 同じハードウェア上の Azure Stack HCI にクラスターを移行する方法について説明します
author: v-dasis
ms.topic: how-to
ms.date: 02/12/2021
ms.author: v-dasis
ms.reviewer: JasonGerend
ms.openlocfilehash: 593be52321230f3fc1ae4329f8f2284cf964298a
ms.sourcegitcommit: 5a8b6dfdf75df1aa9474e062ec3a91ca1b8e58bb
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/15/2021
ms.locfileid: "100524944"
---
# <a name="migrate-to-azure-stack-hci-on-same-hardware"></a>同じハードウェア上の Azure Stack HCI に移行する

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019、Windows Server 2016、Windows Server 2012 R2、Windows Server 2008 R2

このトピックでは、既存のサーバー ハードウェアを使用して、Windows Server 2016 または Windows Server 2019 クラスターを Azure Stack HCI に移行する方法について説明します。 このプロセスにより、新しい Azure Stack HCI オペレーティング システムをインストールし、既存のクラスター設定と記憶域を保持し、VM をインポートします。

次の図は、同じサーバー ハードウェアを使用した Windows Server クラスターのインプレース移行を示しています。 クラスターをシャットダウンした後、Azure Stack HCI がインストールされ、記憶域が再接続され、VM がインポートされ、高可用 (HA) になります。

:::image type="content" source="media/migrate/migrate-cluster-same-hardware.png" alt-text="同じハードウェア上の Azure Stack HCI にクラスターを移行する" lightbox="media/migrate/migrate-cluster-same-hardware.png":::

VM を新しい Azure Stack HCI ハードウェアに移行するには、[「新しいハードウェア上の Azure Stack HCI に移行する」](migrate-cluster-new-hardware.md)を参照してください。

> [!NOTE]
> ストレッチ クラスターの移行については、この記事では説明しません。

## <a name="before-you-begin"></a>始める前に

移行を始める前に、要件と考慮事項がいくつかあります。

- すべての Windows PowerShell コマンドは、管理者として実行する必要があります。

- Azure Stack HCI の管理者アクセス許可を持つドメイン資格情報が必要です。

- 移行元クラスター上のすべての VM をバックアップします。 すべてのアプリケーションとデータのクラッシュ整合バックアップ、およびすべてのデータベースのアプリケーション整合バックアップを完了します。  Azure へのバックアップについては、[Azure Backup の使用](../manage/use-azure-backup.md)に関するページを参照してください。

- すべてのクラスター ノードとクラスターの名前付けのインベントリと構成、ネットワーク構成、クラスター共有ボリューム (CSV) の回復性と容量、およびクォーラム監視を収集します。

- クラスター VM、オフライン CSV、オフライン記憶域プール、およびクラスター サービスをシャットダウンします。
- クラスター名オブジェクト (CNO) を無効にし (後で再利用されます)、次のことを行います。
    - CNO に、その組織単位 (OU) に対するオブジェクトの作成権限が付与されていることを確認します
    - OU に、継承ポリシーのブロックが設定されていることを確認します
    - この OU で Azure Stack HCI に必要なポリシーを設定します

## <a name="vm-version-support-and-update"></a>VM のバージョンのサポートと更新

同じハードウェア上のインプレース移行でサポートされている Windows Server OS バージョンとその VM バージョンを次の表に示します。

VM を実行できる OS のバージョンに関係なく、Azure Stack HCI への移行でサポートされる VM の最小バージョンはバージョン 5.0 です。 そのため、Windows Server 2016 または Windows Server 2019 クラスターでバージョン 2.0、3.0、または 4.0 で実行されている VM は、移行する前にバージョン 5.0 に更新する必要があります。

|OS バージョン|VM のバージョン|
|---|---|
|Windows Server 2008 SP1|2.0|
|Windows Server 2008 R2|3.0|
|Windows Server 2012|4.0|
|Windows Server 2012 R2|5.0|
|Windows Server 2016|8.0|
|Windows Server 2019|9.0|
|Azure Stack HCI|9.0|

Windows Server 2008 SP1、Windows Server 2008 R2-SP1、および Windows 2012 クラスター上の VM の場合、Azure Stack HCI への直接移行はサポートされていません。 このような場合は、次の 2 つのオプションがあります。

- これらの VM をまず Windows Server 2012 R2、Windows Server 2016、または Windows Server 2019 に移行し、VM のバージョンを更新してから、移行プロセスを開始します。

- Robocopy を使用し、すべての VM VHD を Azure Stack HCI にコピーします。 次に、新しい VM を作成し、コピーした VHD を Azure Stack HCI のそれぞれの VM にアタッチします。 こうすることで、このような古い VM に対する VM バージョンの制限が回避されます。

## <a name="updating-the-vm-version"></a>VM バージョンの更新

1 台のサーバーについてすべての VM のバージョンを表示するには、次のコマンドを使用します。

```powershell
Get-VM * | Format-Table Name,Version
```

Windows Server クラスター上のすべてのノードのすべての VM バージョンを表示するには:

```powershell
Get-VM –ComputerName (Get-ClusterNode)
```

すべての Windows Server ノード上のすべての VM を最新バージョンに更新するには:

```powershell
Get-VM –ComputerName (Get-ClusterNode) | Update-VMVersion -Force
```

## <a name="updating-the-servers-and-cluster"></a>サーバーとクラスターの更新

移行は、Windows Server のデプロイで、VM と記憶域を変更せず、OS をクリーン インストールするように Azure Stack HCI 設定を行うことで構成されます。 これにより、現在のオペレーティング システムは Azure Stack HCI に置き換えられます。 詳細については、「[Azure Stack HCI オペレーティング システムのデプロイ](operating-system.md)」を参照してください。 その後、新しい Azure Stack HCI クラスターを作成し、記憶域を再アタッチして、VM をインポートします。

1. 既存のクラスター VM、オフライン CSV、オフライン記憶域プール、およびクラスター サービスをシャットダウンします。

1. Azure Stack HCI ビットをダウンロードした場所に移動し、各 Windows Server ノード上で Azure Stack HCI 設定を実行します。

1. 設定時に、 **[Custom: Install the newer version of Azure Stack HCI only (advanced)]\(カスタム: 新しいバージョンの Azure Stack HCI のインストールのみを行う (詳細)\)** を選択します。 各サーバーに対してこの手順を繰り返します。

1. 新しい Azure Stack HCI クラスターを作成します。 次に示すように、Windows Admin Center または Windows PowerShell を使用してこれを行うことができます。  

> [!IMPORTANT]
> Hyper-V 仮想スイッチ (`VMSwitch`) の名前は、クラスター構成インベントリで取得されたものと同じにする必要があります。 VM をインポートする前に、Azure Stack HCI クラスターで使用されている仮想スイッチ名が元のソース仮想スイッチ名と一致していることを確認します。

> [!NOTE]
> 新しい VM を作成する前に、Azure Stack HCI クラスターを Azure に登録する必要があります。 詳細については、[Azure への登録](register-with-azure.md)に関するページを参照してください。

### <a name="using-windows-admin-center"></a>Windows Admin Center を使用する

Windows Admin Center を使用して Azure Stack HCI クラスターを作成する場合、クラスターの作成ウィザードにより、必要なすべてのロールと機能が各サーバー ノードに自動的にインストールされます。

クラスターの作成方法の詳細については、「[Windows Admin Center を使用して Azure Stack HCI クラスターを作成する](create-cluster.md)」を参照してください。

> [!IMPORTANT]
> クラスターの作成ウィザードで、ステップ **[4.1 Clean drives]\(4.1 ドライブのクリーニング\)** をスキップします。 そうしないと、既存の VM と記憶域を削除することになります。

1. クラスターの作成ウィザードを開始します。 **[Step 4: Storage]\(ステップ 4 記憶域\)** に進みます。

1. ステップ **[4.1 Clean drives]\(4.1 ドライブのクリーニング\)** をスキップします。 この操作は行わないでください。

1. ウィザードから離れます。

1. PowerShell を開き、次のコマンドレットを実行して新しい `Storagesubsystem Object` ID を作成し、すべての記憶域エンクロージャを再検出し、SES ドライブ番号を割り当てます。

    ```powershell
    Enable-ClusterS2D -Verbose
    ```

    Windows Server 2016 から移行する場合、これにより新しい `ClusterperformanceHistory` ReFS ボリュームも作成され、SDDC クラスター リソース グループに割り当てられます。

    Windows Server 2019 から移行する場合、これにより既存の `ClusterperformanceHistory` ReFS ボリュームも追加され、SDDC クラスター リソース グループに割り当てられます。

1. ウィザードに戻ります。 ステップ **[4.2 Verify drives]\(4.2 ドライブの確認\)** で、すべてのドライブがアラートやエラーなしで一覧表示されていることを確認します。

1. ウィザードを完了します。

### <a name="using-windows-powershell"></a>Windows PowerShell を使用する

PowerShell を使用して Azure Stack HCI クラスターを作成する場合は、次のコマンドレットを使用し、次のロールと機能を各 Azure Stack HCI クラスター ノードにインストールする必要があります。

```powershell
Install-WindowsFeature -Name Hyper-V, Failover-Clustering, FS-Data-Deduplication, Bitlocker, Data-Center-Bridging, RSAT-AD-PowerShell -IncludeAllSubFeature -IncludeManagementTools -Verbose
```

PowerShell を使用してクラスターを作成する方法の詳細については、「[Windows PowerShell を使用して Azure Stack HCI クラスターを作成する](create-cluster-powershell.md)」を参照してください。

> [!NOTE]
> 以前に無効にしたクラスター名オブジェクトと同じ名前を再利用します。

1. 次のコマンドレットを実行してクラスターを作成します。

    ```powershell
    New-cluster –name "clustername" –node Server01,Server02 –staticaddress xx.xx.xx.xx –nostorage
    ```

1. 次のコマンドレットを実行して新しい `Storagesubsystem Object` ID を作成し、すべての記憶域エンクロージャを再検出し、SES ドライブ番号を割り当てます。

    ```powershell
    Enable-ClusterS2D -Verbose
    ```

1. Windows Server 2016 から移行する場合、これにより新しい `ClusterperformanceHistory` ReFS ボリュームも作成され、SDDC クラスター リソース グループに割り当てられます。

    > [!NOTE]
    > 記憶域プールに "Minority Disk errors" (軽微なディスク エラー) (クラスター マネージャーで確認できます) が表示される場合は、`Enable-ClusterS2D -verbose` コマンドレットを再実行します。

1. クラスター マネージャーを使用して、すべての CSV を有効にします。ただし、`ClusterperformanceHistory` ボリュームは除外します。これは ReFS ボリュームです (これが ReFS CSV ではないことを確認してください)。

1. Windows Server 2019 から移行する場合は、`Enable-ClusterS2D -verbose` コマンドレットを再実行します。 これにより、`ClusterperformanceHistory` ReFS ボリュームは SDDC クラスター リソース グループに関連付けられます。

1. 以下を実行し、現在の記憶域プールの名前とバージョンを確認します。

    ```powershell
    Get-StoragePool | ? IsPrimordial -eq $false | ft FriendlyName,Version
    ```

1. 次に、新しい記憶域プールの名前とバージョンを確認します。

    ```powershell
    Get-StoragePool | ? IsPrimordial -eq $false | ft FriendlyName,Version
    ```

1. クォーラム監視を作成します。 方法については、「[クラスター監視のセットアップ](../manage/witness.md)」を参照してください。

1. 以下を使用して、記憶域修復ジョブが完了していることを確認します。

    ```powershell
    Get-StorageJob
    ```

    > [!NOTE]
    >アップグレード中に実行される VM の数によっては、かなりの時間がかかることがあります。

1. すべてのディスクが正常であることを確認します。

    ```powershell
    Get-VirtualDisk
    ```

1. クラスター ノードのバージョンを確認します。`ClusterFunctionalLevel` と `ClusterUpgradeVersion` が表示されます。 これを取得するために、以下を実行します。

    ```powershell
    Get-ClusterNodeSupportedVersion
    ```

    > [!NOTE]
    > `ClusterFunctionalLevel` は自動的に `10` に設定されます。オペレーティング システムとクラスターの新規作成のために更新する必要はありません。

1. 記憶域プールを次のように更新します。

    ```powershell
    Get-StoragePool | Update-StoragePool
    ```

## <a name="refs-volumes"></a>ReFS ボリューム

Windows Server 2016 から移行する場合、Resilient File System (ReFS) ボリュームがサポートされますが、そのようなボリュームは、ミラー高速パリティ (MAP) ボリュームの使用による Azure Stack HCI のパフォーマンス向上の恩恵を受けません。 この機能強化には、PowerShell の `New-Volume` コマンドレットを使用して作成する新しい ReFS ボリュームが必要です。

Windows Server 2016 MAP ボリュームの場合、ReFS 圧縮は使用できませんでした。そのため、これらのボリュームを再アタッチしても問題ありませんが、Azure Stack HCI クラスター上で新しい MAP ボリュームを作成する場合に比べてパフォーマンスは低くなります。

## <a name="import-the-vms"></a>VM をインポートする

VM のワークロードの回復性、パフォーマンス、スケーリングを向上させるためのベスト プラクティスは、クラスター ノードごとに少なくとも 1 つのクラスター共有ボリューム (CSV) を作成し、各 CSV 所有者の VM のバランスを均等にすることです。 既定では、このバランス処理は 5 分ごとに自動的に行われます。移行元クラスター ノードと移行先クラスター ノードの間で Robocopy を使用するときは、そのことを考慮して、移行元と移行先で CSV 所有者を一致させ、最適な転送パスと速度が提供されるようにする必要があります。

Azure Stack HCI クラスターで次の手順を実行して VM をインポートし、高可用性にしてから、それらを開始します。

1. 次のコマンドレットを実行して、すべての CSV 所有者ノードを表示します。

    ```powershell
    Get-ClusterSharedVolume
    ```

1. サーバー ノードごとに、`C:\Clusterstorage\Volume` に移動して、すべての VM のパスを設定します (例: `C:\Clusterstorage\volume01`)。

1. VM をインポートする前に、各 CSV 所有者ノードで次のコマンドレットを実行し、ボリュームごとにすべての VM VMCX ファイルへのパスを表示します。 環境に合わせてパスを変更します。

    ```powershell
    Get-ChildItem -Path "C:\Clusterstorage\Volume01\*.vmcx" -Recurse
    ```

1. サーバー ノードごとに次のコマンドレットを実行することにより、すべての VM をインポートして登録し、各 CSV 所有者ノードでそれらが高可用になるようにします。 これにより、プロセッサとメモリの割り当てが最適になるように VM が均等に分散されます。

    ```powershell
    Get-ChildItem -Path "C:\Clusterstorage\Volume01\*.vmcx" -Recurse | Import-VM -Register | Get-VM | Add-ClusterVirtualMachineRole
    ```

1. 各ノードで各移行先 VM を起動します。

    ```powershell
    Start-VM -Name
    ```

1. ログインし、すべての VM が実行されていること、およびすべてのアプリとデータが存在することを確認します。

    ```powershell
    Get-VM -ComputerName Server01 | Where-Object {$_.State -eq 'Running'}
    ```

1. 最後に、すべての機能強化を利用するため、VM を最新の Azure Stack HCI バージョンに更新します。

    ```powershell
    Get-VM | Update-VMVersion -Force
    ```

## <a name="next-steps"></a>次のステップ

- 移行後にクラスターを検証します。 「[Azure Stack HCI クラスターの検証](validate.md)」を参照してください。
- Windows Server VM を新しい Azure Stack HCI ハードウェアに移行するには、[「新しいハードウェア上の Azure Stack HCI に移行する」](migrate-cluster-new-hardware.md)を参照してください。