---
title: Azure Stack の VM ディスクを管理する | Microsoft Docs
description: Azure Stack で仮想マシンのディスクを作成します。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/20/2019
ms.author: sethm
ms.reviewer: jiahan
ms.lastreviewed: 01/18/2019
ms.openlocfilehash: 10900095a79da2639212b5fa58becbafb7bee0eb
ms.sourcegitcommit: d2012e765c3fa5bccb4756d190349e890f9f48bd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/20/2019
ms.locfileid: "65941202"
---
# <a name="create-virtual-machine-disk-storage-in-azure-stack"></a>Azure Stack で仮想マシンのディスク記憶域を作成する

*適用対象:Azure Stack 統合システムと Azure Stack Development Kit*

この記事では、Azure Stack ポータルまたは PowerShell を使用して、仮想マシンのディスク記憶域を作成する方法について説明します。

## <a name="overview"></a>概要

バージョン 1808 以降、Azure Stack では、マネージド ディスクとアンマネージド ディスクを、オペレーティング システム (OS) およびデータ ディスクの両方として仮想マシンで使用できます。 バージョン 1808 未満では、アンマネージド ディスクのみがサポートされます。

[マネージド ディスク](/azure/virtual-machines/windows/managed-disks-overview)を使用すると、VM ディスクに関連付けられているストレージ アカウントを管理できるため、Azure IaaS VM のディスク管理が簡素化されます。 Azure Stack では、必要なディスクのサイズを指定するだけで、ディスクの作成と管理が自動的に行われます。

アンマネージド ディスクについては、そのディスクを格納するストレージ アカウントを作成する必要があります。 作成したディスクは VM ディスクと呼ばれ、ストレージ アカウントのコンテナーに格納されます。

### <a name="best-practice-guidelines"></a>ベスト プラクティス ガイドライン

パフォーマンスを改善し、全体的なコストを削減するために、個別のコンテナーに各 VM ディスクを配置することをお勧めします。 1 つのコンテナーには、OS ディスクまたはデータ ディスクのどちらかを保持し、同時に両方を保持しないでください。 ただし、同じコンテナーに両方の種類のディスクを入れることもできます。

VM に 1 つ以上のデータ ディスクを追加する場合は、追加コンテナーを、これらのディスクを格納する場所として使用してください。 追加 VM の OS ディスクも、独自のコンテナーに配置する必要があります。

VM を作成するときは、新しい仮想マシンごとに同じストレージ アカウントを再利用できます。 作成するコンテナーのみを一意にする必要があります。

### <a name="adding-new-disks"></a>新しいディスクを追加する

次の表は、ポータルおよび PowerShell を使用してディスクを追加する方法をまとめたものです。

| Method | オプション
|-|-|
|ユーザー ポータル|- 既存の VM に新しいデータ ディスクを追加します。 新しいディスクは、Azure Stack によって作成されます。 </br> </br>- 前に作成した VM に既存のディスク (.vhd) ファイルを追加します。 これを行うには、.vhd ファイルを準備して、そのファイルを Azure Stack にアップロードする必要があります。 |
|[PowerShell](#use-powershell-to-add-multiple-unmanaged-disks-to-a-vm) | - OS ディスクがある新しい VM を作成し、同時にその VM に 1 つまたは複数のデータ ディスクを追加します。 |

## <a name="use-the-portal-to-add-disks-to-a-vm"></a>ポータルを使用してディスクを VM に追加する

ポータルを使ってマーケットプレース項目のほとんどを対象とする VM を作成する場合、既定では、OS ディスクのみが作成されます。

VM の作成後、ポータルを使って次の操作を行うことができます。

* 新しいデータ ディスクを作成し、VM に接続する。
* 既存のデータ ディスクを作成し、VM にアップロードする。

追加するアンマネージド ディスクはそれぞれ、個別のコンテナーに配置する必要があります。

>[!NOTE]  
>Azure によって作成および管理されているディスクは、[マネージド ディスク](/azure/virtual-machines/windows/managed-disks-overview)と呼ばれます。

### <a name="use-the-portal-to-create-and-attach-a-new-data-disk"></a>ポータルを使用して新しいデータ ディスクを作成して接続する

1. ポータルで、 **[すべてのサービス]** を選択し、次に **[仮想マシン]** を選択します。
   ![例:VM ダッシュボード](media/azure-stack-manage-vm-disks/vm-dashboard.png)

2. 以前に作成した仮想マシンを選択します。
   ![例:ダッシュボードで VM を選択する](media/azure-stack-manage-vm-disks/select-a-vm.png)

3. その仮想マシンについて、 **[ディスク]** を選択し、次に **[データ ディスクの追加]** を選択します。
   ![例:VM に新しいディスクをアタッチする](media/azure-stack-manage-vm-disks/Attach-disks.png)

4. データ ディスクに対して次の操作を実行します。
   * **LUN** を入力します。 LUN には有効な数値を指定してください。
   * **[ディスクの作成]** を選択します。
   ![例:VM に新しいディスクをアタッチする](media/azure-stack-manage-vm-disks/add-a-data-disk-create-disk.png)

5. **[マネージド ディスクの作成]** ブレードで次の操作を行います。
   * **[名前]** にディスクの名前を入力します。
   * 既存のリソース グループを選択するか、新しい**リソース グループ**を作成します。
   * **[場所]** を選択します。 既定では、場所は、OS ディスクを保持するコンテナーに設定されます。
   * **[アカウントの種類]** を選択します。
      ![例:VM に新しいディスクをアタッチする](media/azure-stack-manage-vm-disks/create-manage-disk.png)

      **Premium SSD**  
      Premium ディスク (SSD) の実体はソリッドステート ドライブであり、待ち時間の短い一貫したパフォーマンスが得られます。 価格とパフォーマンスのバランスにきわめて優れており、I/O 集中型のアプリケーションや運用環境のワークロードに最適です。

      **Standard HDD**  
      Standard ディスク (HDD) の実体は磁気ドライブであり、データへのアクセス頻度が低い用途に適しています。 ゾーン冗長ディスクの実体はゾーン冗長ストレージ (ZRS) です。ZRS では、データが複数のゾーンにレプリケートされ、1 つのゾーンがダウンしても可用性が保たれます。

   * **[ソースの種類]** を選択します。

     ディスクは、別のディスクのスナップショットや、ストレージ アカウント内の BLOB から作成できるほか、空のディスクを作成することができます。

      **スナップショット**:スナップショットが利用できる場合は、スナップショットを選択します。 スナップショットは、VM のサブスクリプションと場所から利用できることが必要です。

      **Storage Blob**:
     * ディスク イメージが格納されている BLOB ストレージの URI を追加します。  
     * **[参照]** を選択して、[ストレージ アカウント] ブレードを開きます。 手順については、「[ストレージ アカウントからデータ ディスクを追加する](#add-a-data-disk-from-a-storage-account)」を参照してください。
     * イメージの OS の種類を **[Windows]** 、 **[Linux]** 、 **[なし (データ ディスク)]** のいずれかから選択します。

   * **[サイズ (GiB)]** を選択します。

     Standard ディスクのコストは、ディスクのサイズに応じて高くなります。 Premium ディスクのコストとパフォーマンスは、ディスクのサイズに応じて高くなります。 詳細については、「[Managed Disks の価格](https://go.microsoft.com/fwlink/?linkid=843142)」を参照してください。

   * **作成** を選択します。 Azure Stack でマネージド ディスクが作成されて検証されます。

6. Azure Stack でディスクが作成され、仮想マシンに接続されると、仮想マシンのディスク設定の **[データ ディスク]** に新しいディスクが表示されます。

   ![例:ディスクを表示する](media/azure-stack-manage-vm-disks/view-data-disk.png)

### <a name="add-a-data-disk-from-a-storage-account"></a>ストレージ アカウントからデータ ディスクを追加する

Azure Stack でのストレージ アカウントの使用について詳しくは、「[Azure Stack Storage の概要](azure-stack-storage-overview.md)」を参照してください。

1. 使用する**ストレージ アカウント**を選択します。
2. データ ディスクを配置する**コンテナー**を選択します。 必要に応じて、 **[コンテナー]** ブレードで新しいコンテナーを作成できます。 その後、新しいディスクの場所をそのコンテナーに変更できます。 ディスクごとに別のコンテナーを使用する場合は、データ ディスクの分散配置することでパフォーマンスを向上させることができます。
3. **[選択]** を選択して、選択内容を保存します。

    ![例:コンテナーを選択する](media/azure-stack-manage-vm-disks/select-container.png)

## <a name="attach-an-existing-data-disk-to-a-vm"></a>VM に既存のデータ ディスクを接続する

1. VM のデータ ディスクとして使用する [.Vhd ファイルを準備します](https://docs.microsoft.com/azure/virtual-machines/windows/classic/createupload-vhd)。 .vhd ファイルを接続する VM で使用するストレージ アカウントに .vhd ファイルをアップロードします。

    .vhd ファイルを保持するために使用するコンテナーは、OS ディスクを保持するコンテナーとは別のコンテナーにすることを計画してください。
    ![例:VHD ファイルのアップロード](media/azure-stack-manage-vm-disks/upload-vhd.png)

2. .vhd ファイルをアップロードしたら、VM に VHD をアタッチできます。 左側のメニューで **[仮想マシン]** を選択します。  
 ![例:ダッシュボードで VM を選択する](media/azure-stack-manage-vm-disks/vm-dashboard.png)

3. 一覧から仮想マシンを選択します。

    ![例:ダッシュボードで VM を選択する](media/azure-stack-manage-vm-disks/select-a-vm.png)

4. 仮想マシンのページで、 **[ディスク]** を選択し、次に **[既存のディスクの接続]** を選択します。

    ![例:既存のディスクの接続](media/azure-stack-manage-vm-disks/attach-disks2.png)

5. **[既存のディスクの接続]** ページで、 **[VHD ファイル]** を選択します。 **[ストレージ アカウント]** ページが開きます。

    ![例:VHD ファイルを選択する](media/azure-stack-manage-vm-disks/select-vhd.png)

6. **[ストレージ アカウント]** で、使用するアカウントを選択し、前にアップロードした .vhd ファイルを保持するコンテナーを選択します。 .vhd ファイルを選択し、 **[選択]** を選択して、選択内容を保存します。

    ![例:コンテナーを選択する](media/azure-stack-manage-vm-disks/select-container2.png)

7. **[既存のディスクの接続]** の **[VHD ファイル]** に、選択したファイルが表示されます。 ディスクの **[ホスト キャッシュ]** 設定を更新し、 **[OK]** を選択して VM の新しいディスク構成を保存します。

    ![例:VHD ファイルをアタッチする](media/azure-stack-manage-vm-disks/attach-vhd.png)

8. Azure Stack でディスクが作成され、仮想マシンに接続されると、仮想マシンのディスク設定の **[データ ディスク]** に新しいディスクが表示されます。

    ![例:ディスクのアタッチを完了する](media/azure-stack-manage-vm-disks/complete-disk-attach.png)

## <a name="use-powershell-to-add-multiple-unmanaged-disks-to-a-vm"></a>PowerShell を使用して複数の管理されていないディスクを VM に追加する

PowerShell を使用して VM をプロビジョニングし、新しいデータ ディスクを追加するか、既存の **.vhd** ファイルをデータ ディスクとして接続できます。

**Add-AzureRmVMDataDisk** コマンドレットは、仮想マシンにデータ ディスクを追加します。 データ ディスクは仮想マシンを作成するときに追加できます。または、既存のバーチャル マシンにデータ ディスクを追加できます。 別々のコンテナーにディスクを分散させるには、**VhdUri** パラメーターを指定します。

### <a name="add-data-disks-to-a-new-virtual-machine"></a>新しい仮想マシンにデータ ディスクを追加する

次の例では、PowerShell コマンドを使用して、3 つのデータ ディスクがある VM を作成します。各ディスクは別々のコンテナーに配置されます。

最初のコマンドは、仮想マシン オブジェクトを作成し、それを `$VirtualMachine` 変数に格納します。 このコマンドは、仮想マシンに名前とサイズを割り当てます。

```powershell
$VirtualMachine = New-AzureRmVMConfig -VMName "VirtualMachine" `
                                      -VMSize "Standard_A2"
```

次の 3 つのコマンドは、3 つのデータ ディスクのパスを `$DataDiskVhdUri01`、 `$DataDiskVhdUri02`、および `$DataDiskVhdUri03` の各変数に割り当てます。 URL に異なるパス名を定義して、ディスクを別々のコンテナーに分散させます。

```powershell
$DataDiskVhdUri01 = "https://contoso.blob.local.azurestack.external/test1/data1.vhd"
```

```powershell
$DataDiskVhdUri02 = "https://contoso.blob.local.azurestack.external/test2/data2.vhd"
```

```powershell
$DataDiskVhdUri03 = "https://contoso.blob.local.azurestack.external/test3/data3.vhd"
```

最後の 3 つのコマンドは、`$VirtualMachine` に格納されている仮想マシンにデータ ディスクを追加します。 各コマンドは、ディスクの名前、場所、および追加のプロパティを指定します。 各ディスクの URI は、`$DataDiskVhdUri01`、 `$DataDiskVhdUri02`、および `$DataDiskVhdUri03` に格納されています。

```powershell
$VirtualMachine = Add-AzureRmVMDataDisk -VM $VirtualMachine -Name 'DataDisk1' `
                -Caching 'ReadOnly' -DiskSizeInGB 10 -Lun 0 `
                -VhdUri $DataDiskVhdUri01 -CreateOption Empty
```

```powershell
$VirtualMachine = Add-AzureRmVMDataDisk -VM $VirtualMachine -Name 'DataDisk2' `
                -Caching 'ReadOnly' -DiskSizeInGB 11 -Lun 1 `
                -VhdUri $DataDiskVhdUri02 -CreateOption Empty
```

```powershell
$VirtualMachine = Add-AzureRmVMDataDisk -VM $VirtualMachine -Name 'DataDisk3' `
                -Caching 'ReadOnly' -DiskSizeInGB 12 -Lun 2 `
                -VhdUri $DataDiskVhdUri03 -CreateOption Empty
```

次の PowerShell コマンドを使用して、OS ディスクとネットワーク構成を VM に追加した後、新しい VM を起動します。

```powershell
#set variables
$rgName = "myResourceGroup"
$location = "local"
#Set OS Disk
$osDiskUri = "https://contoso.blob.local.azurestack.external/vhds/osDisk.vhd"
$osDiskName = "osDisk"

$VirtualMachine = Set-AzureRmVMOSDisk -VM $VirtualMachine -Name $osDiskName -VhdUri $osDiskUri `
    -CreateOption FromImage -Windows

# Create a subnet configuration
$subnetName = "mySubNet"
$singleSubnet = New-AzureRmVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix 10.0.0.0/24

# Create a vnet configuration
$vnetName = "myVnetName"
$vnet = New-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $rgName -Location $location `
    -AddressPrefix 10.0.0.0/16 -Subnet $singleSubnet

# Create a public IP
$ipName = "myIP"
$pip = New-AzureRmPublicIpAddress -Name $ipName -ResourceGroupName $rgName -Location $location `
    -AllocationMethod Dynamic

# Create a network security group configuration
$nsgName = "myNsg"
$rdpRule = New-AzureRmNetworkSecurityRuleConfig -Name myRdpRule -Description "Allow RDP" `
    -Access Allow -Protocol Tcp -Direction Inbound -Priority 110 `
    -SourceAddressPrefix Internet -SourcePortRange * `
    -DestinationAddressPrefix * -DestinationPortRange 3389
$nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName $rgName -Location $location `
    -Name $nsgName -SecurityRules $rdpRule

# Create a NIC configuration
$nicName = "myNicName"
$nic = New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $rgName `
-Location $location -SubnetId $vnet.Subnets[0].Id -NetworkSecurityGroupId $nsg.Id -PublicIpAddressId $pip.Id

#Create the new VM
$VirtualMachine = Set-AzureRmVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName VirtualMachine | `
    Set-AzureRmVMSourceImage -PublisherName MicrosoftWindowsServer -Offer WindowsServer `
    -Skus 2016-Datacenter -Version latest | Add-AzureRmVMNetworkInterface -Id $nic.Id
New-AzureRmVM -ResourceGroupName $rgName -Location $location -VM $VirtualMachine
```

### <a name="add-data-disks-to-an-existing-virtual-machine"></a>データ ディスクを既存の仮想マシンに追加する

次の例では、PowerShell コマンドを使用して、既存の VM に 3 つのデータ ディスクを追加します。 最初のコマンドは、**Get-AzureRmVM** コマンドレットを使用して、**VirtualMachine** という名前の仮想マシンを取得します。 このコマンドは仮想マシンを `$VirtualMachine` 変数に保存します。

```powershell
$VirtualMachine = Get-AzureRmVM -ResourceGroupName "myResourceGroup" `
                                -Name "VirtualMachine"
```

次の 3 つのコマンドは、3 つのデータ ディスクのパスを `$DataDiskVhdUri01`、 `$DataDiskVhdUri02`、および `$DataDiskVhdUri03` の各変数に割り当てます。 VHD URI 内の異なるパス名は、ディスクを配置する別々のコンテナーを示します。

```powershell
$DataDiskVhdUri01 = "https://contoso.blob.local.azurestack.external/test1/data1.vhd"
```

```powershell
$DataDiskVhdUri02 = "https://contoso.blob.local.azurestack.external/test2/data2.vhd"
```

```powershell
$DataDiskVhdUri03 = "https://contoso.blob.local.azurestack.external/test3/data3.vhd"
```

次の 3 つのコマンドは、`$VirtualMachine` 変数に格納されている仮想マシンにデータ ディスクを追加します。 各コマンドは、ディスクの名前、場所、および追加のプロパティを指定します。 各ディスクの URI は、`$DataDiskVhdUri01`、 `$DataDiskVhdUri02`、および `$DataDiskVhdUri03` に格納されています。

```powershell
Add-AzureRmVMDataDisk -VM $VirtualMachine -Name "disk1" `
                      -VhdUri $DataDiskVhdUri01 -LUN 0 `
                      -Caching ReadOnly -DiskSizeinGB 10 -CreateOption Empty
```

```powershell
Add-AzureRmVMDataDisk -VM $VirtualMachine -Name "disk2" `
                      -VhdUri $DataDiskVhdUri02 -LUN 1 `
                      -Caching ReadOnly -DiskSizeinGB 11 -CreateOption Empty
```

```powershell
Add-AzureRmVMDataDisk -VM $VirtualMachine -Name "disk3" `
                      -VhdUri $DataDiskVhdUri03 -LUN 2 `
                      -Caching ReadOnly -DiskSizeinGB 12 -CreateOption Empty
```

最後のコマンドは、`-ResourceGroupName` 内の `$VirtualMachine` に格納されている仮想マシンの状態を更新します。

```powershell
Update-AzureRmVM -ResourceGroupName "myResourceGroup" -VM $VirtualMachine
```

## <a name="next-steps"></a>次の手順

Azure Stack 仮想マシンの詳細については、「[Azure Stack の仮想マシンに関する考慮事項](azure-stack-vm-considerations.md)」に進んでください。
