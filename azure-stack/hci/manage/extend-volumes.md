---
title: Azure Stack HCI でボリュームを拡張する
description: Windows Admin Center と PowerShell を使用して Azure Stack HCI でボリュームを拡張する方法について説明します。
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.date: 03/04/2021
ms.openlocfilehash: 275cafad3f813b28e2b926ad2008bca777904e0e
ms.sourcegitcommit: f194f9ca4297864500e62d8658674a0625b29d1d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/05/2021
ms.locfileid: "102187335"
---
# <a name="expanding-volumes-in-azure-stack-hci"></a>Azure Stack HCI でボリュームを拡張する

> 適用対象:Azure Stack HCI バージョン 20H2、Windows Server 2019

このトピックでは、Windows Admin Center と PowerShell を使用してクラスターのボリュームを拡張する手順について説明します。

> [!WARNING]
> **サポート対象外: 記憶域スペース ダイレクトで使用される基盤のストレージのサイズ変更。** Azure を含む仮想ストレージ環境で記憶域スペース ダイレクトを実行している場合、仮想マシンが使用する記憶装置のサイズ変更や特性変更はサポートされず、それを行うとデータにアクセスできなくなります。 代わりに、ボリュームを拡張する前に、[サーバーまたはドライブの追加](/windows-server/storage/storage-spaces/add-nodes)に関するセクションの手順に従って、容量を追加してください。

## <a name="expand-volumes-using-windows-admin-center"></a>Windows Admin Center を使用したボリュームの拡張

1. Windows Admin Center で、Azure Stack HCI クラスターに接続し、 **[ツール]** ウィンドウで **[ボリューム]** を選択します。
2. **[ボリューム]** ページで **[インベントリ]** タブを選択し、拡張するボリュームを選択します。

    ボリュームの詳細ページに、ボリュームのストレージ容量が表示されます。 ボリュームの詳細ページは、ダッシュボードから直接開くこともできます。 ダッシュボードの [アラート] ペインで、ボリュームのストレージ容量が不足している場合に通知が表示されるアラートを選択し、 **[Go To Volume]\(ボリュームへ移動\)** を選択します。

3. ボリュームの詳細ページの上部で、 **[Expand]\(拡張\)** を選択します。
4. より大きな新しいサイズを入力し、 **[Expand]\(拡張\)** を選択します。

    ボリュームの詳細ページに、ボリュームのストレージ容量が大きいことが示され、ダッシュボードのアラートがクリアされます。

## <a name="expand-volumes-using-powershell"></a>PowerShell を使用してボリュームを拡張する

### <a name="capacity-in-the-storage-pool"></a>記憶域プールの容量

ボリュームを拡張する前に、より大きな新しい占有領域に対応できる十分な容量が記憶域プールにあることを確認してください。 たとえば、3 方向ミラー ボリュームを 1 TB から 2 TB に拡張すると、その占有領域は 3 TB から 6 TB に増えます。 拡張を正常に実行するには、記憶域プールに少なくとも 3 TB (6 - 3) の使用可能な容量が必要です。

### <a name="familiarity-with-volumes-in-storage-spaces"></a>記憶域スペースのボリュームに関する知識

記憶域スペース ダイレクトでは、各ボリュームは、クラスターの共有ボリューム (CSV) (ボリューム)、パーティション、ディスク(仮想ディスク)、および複数のストレージ層 (該当する場合) の、複数のスタックされたオブジェクトで構成されます。 ボリュームのサイズを変更するには、これらのオブジェクトのいくつかのサイズを変更する必要があります。

![図は、クラスターの共有ボリューム、ボリューム、パーティション、ディスク、仮想ディスク、ストレージ層など、ボリュームの層を示しています。](media/extend-volumes/volumes-in-smapi.png)

これらに慣れるために、PowerShell で対応する名詞を使用して **Get-** を実行してみてください。

次に例を示します。

```PowerShell
Get-VirtualDisk
```

スタック内のオブジェクト間の関連付けを辿るには、1 つの **Get-** コマンドレットを次のものにパイプ処理します。

たとえば、仮想ディスクからそのボリュームまでを取得する方法を次に示します。

```PowerShell
Get-VirtualDisk <FriendlyName> | Get-Disk | Get-Partition | Get-Volume
```

### <a name="step-1--expand-the-virtual-disk"></a>手順 1 – 仮想ディスクを拡張する

仮想ディスクは、作成方法に応じてストレージ層を使用する場合と使用しない場合があります。

確認するには、次のコマンドレットを実行します。

```PowerShell
Get-VirtualDisk <FriendlyName> | Get-StorageTier
```

コマンドレットから何も返されない場合、仮想ディスクはストレージ層を使用しません。

#### <a name="no-storage-tiers"></a>ストレージ層なし

仮想ディスクにストレージ層がない場合は、**Resize-VirtualDisk** コマンドレットを使用してそれを直接拡張できます。

**-Size** パラメーターに新しいサイズを指定します。

```PowerShell
Get-VirtualDisk <FriendlyName> | Resize-VirtualDisk -Size <Size>
```

**VirtualDisk** を拡張すると、それに応じて **Disk** のサイズも自動的に変更されます。

![このアニメーション化された図では、ボリュームの仮想ディスクが大きくなり、それを受けて、そのすぐ上のディスク層が自動的に大きくなります。](media/extend-volumes/Resize-VirtualDisk.gif)

#### <a name="with-storage-tiers"></a>ストレージ層あり

仮想ディスクでストレージ層が使用されている場合は、**Resize-StorageTier** コマンドレットを使用して、各層を個別に拡張できます。

仮想ディスクからの関連付けを辿って、ストレージ層の名前を取得します。

```PowerShell
Get-VirtualDisk <FriendlyName> | Get-StorageTier | Select FriendlyName
```

次に、各層ごとに、 **-Size** パラメーターに新しいサイズを指定します。

```PowerShell
Get-StorageTier <FriendlyName> | Resize-StorageTier -Size <Size>
```

> [!TIP]
> 使用する層の物理メディアの種類が異なる場合 (**MediaType = SSD** および **MediaType = HDD** など)、各層のより大きな新しい占有領域に対応するために、記憶域プール内に各メディアの種類の十分な容量を確保する必要があります。

**StorageTier** を拡張すると、それに応じて **VirtualDisk** と **Disk** のサイズも自動的に変更されます。

![このアニメーション化された図では、1 つ目と 2 つ目のストレージ層が順に大きくなり、さらに、その上の仮想ディスク層とディスク層も大きくなります。](media/extend-volumes/Resize-StorageTier.gif)

### <a name="step-2--expand-the-partition"></a>手順 2 – パーティションを拡張する

次に、**Resize-Partition** コマンドレットを使用してパーティションを拡張します。 仮想ディスクに 2 つのパーティションがあることが想定されています。最初のものは予約済みであり、変更しないでください。サイズ変更が必要なものは、**PartitionNumber = 2** および **Type = Basic** です。

**-Size** パラメーターに新しいサイズを指定します。 次に示すように、サポートされている最大サイズを使用することをお勧めします。

```PowerShell
# Choose virtual disk
$VirtualDisk = Get-VirtualDisk <FriendlyName>

# Get its partition
$Partition = $VirtualDisk | Get-Disk | Get-Partition | Where PartitionNumber -Eq 2

# Resize to its maximum supported size
$Partition | Resize-Partition -Size ($Partition | Get-PartitionSupportedSize).SizeMax
```

**Partition** を拡張すると、それに応じて **Volume** と **ClusterSharedVolume** のサイズも自動的に変更されます。

![このアニメーション化された図では、ボリュームの最下部にある仮想ディスク層が大きくなっており、その上にある各層も大きくなります。](media/extend-volumes/Resize-Partition.gif)

これで完了です。

> [!TIP]
> **Get-Volume** を実行すると、ボリュームに新しいサイズが設定されたことを確認できます。

## <a name="next-steps"></a>次のステップ

その他の重要な記憶域管理タスクの詳細な手順については、以下も参照してください。

- [ボリュームを計画する](../concepts/plan-volumes.md)
- [ボリュームを作成する](create-volumes.md)
- [ボリュームを削除する](delete-volumes.md)