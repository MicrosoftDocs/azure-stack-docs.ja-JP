---
title: Azure Stack Hub で VM ディスク ストレージを作成する
description: Azure Stack Hub で仮想マシンのディスクを作成します。
author: sethmanheim
ms.topic: conceptual
ms.date: 11/22/2020
ms.author: sethm
ms.reviewer: jiahan
ms.lastreviewed: 11/22/2020
ms.openlocfilehash: 2b5f985f5a7ea6537a56f48f87cf77edf711fde5
ms.sourcegitcommit: 34babe5abf1bceee718011b5c5c25f75e1b03b0c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/17/2021
ms.locfileid: "100562651"
---
# <a name="create-vm-disk-storage-in-azure-stack-hub"></a>Azure Stack Hub で VM ディスク ストレージを作成する

この記事では、Azure Stack Hub ポータルまたは PowerShell を使用して、仮想マシン (VM) のディスク記憶域を作成する方法について説明します。

## <a name="overview"></a>概要

Azure Stack Hub では、マネージド ディスクとアンマネージド ディスクを、オペレーティング システム (OS) およびデータ ディスクの両方として VM で使用できます。

[マネージド ディスク](/azure/virtual-machines/windows/managed-disks-overview)を使用すると、VM ディスクに関連付けられているストレージ アカウントを管理できるため、Azure IaaS VM のディスク管理が簡素化されます。 Azure Stack Hub では、必要なディスクのサイズを指定するだけで、ディスクの作成と管理が自動的に行われます。

アンマネージド ディスクについては、そのディスクを格納するストレージ アカウントを作成する必要があります。 作成したディスクは VM ディスクと呼ばれ、ストレージ アカウントのコンテナーに格納されます。

## <a name="best-practice-guidelines"></a>ベスト プラクティス ガイドライン

管理と容量のバランスを簡単に取れるように、VM にはマネージド ディスクを使用することをお勧めします。 マネージド ディスクを使用する前に、ストレージ アカウントとコンテナーを準備する必要はありません。 複数のマネージド ディスクを作成する場合、ディスクは複数のボリュームに分散されるため、ボリュームの容量のバランスを取ることができます。  

アンマネージド ディスクの場合、パフォーマンスを改善し、全体的なコストを削減するために、各アンマネージド ディスクを個別のコンテナーに配置することをお勧めします。 OS ディスクとデータ ディスクの両方を同じコンテナーに配置することもできますが、ベスト プラクティスは 1 つのコンテナーに OS ディスクとデータ ディスクの両方を同時に保持するのではなく、どちらか一方を保持することです。

VM に 1 つ以上のデータ ディスクを追加する場合は、追加コンテナーを、これらのディスクを格納する場所として使用してください。 追加 VM の OS ディスクも、独自のコンテナーに配置する必要があります。

VM を作成するときは、新しい仮想マシンごとに同じストレージ アカウントを再利用できます。 作成するコンテナーのみを一意にする必要があります。

## <a name="adding-new-disks"></a>新しいディスクを追加する

次の表は、ポータルおよび PowerShell を使用してディスクを追加する方法をまとめたものです。

| Method | Options
|-|-|
|ユーザー ポータル| - 既存の VM に新しいデータ ディスクを追加します。 新しいディスクは、Azure Stack Hub によって作成されます。 </br> </br> - 前に作成した VM に既存のディスク (.vhd) ファイルを追加します。 これを行うには、.vhd ファイルを準備して、そのファイルを Azure Stack Hub にアップロードする必要があります。 |
|[PowerShell](#use-powershell-to-add-multiple-disks-to-a-vm) | - OS ディスクがある新しい VM を作成し、同時にその VM に 1 つまたは複数のデータ ディスクを追加します。 |

## <a name="use-the-portal-to-add-disks-to-a-vm"></a>ポータルを使用してディスクを VM に追加する

ポータルを使ってマーケットプレース項目のほとんどを対象とする VM を作成する場合、既定では、OS ディスクのみが作成されます。

VM の作成後、ポータルを使って次の操作を行うことができます。

* 新しいデータ ディスクを作成し、VM に接続する。
* 既存のデータ ディスクを作成し、VM にアップロードする。

追加した各アンマネージド ディスクを別のコンテナーに配置します。

### <a name="use-the-portal-to-create-and-attach-a-new-data-disk"></a>ポータルを使用して新しいデータ ディスクを作成して接続する

1. ポータル内で、 **[すべてのサービス]** 、 **[Compute]** 、 **[仮想マシン]** の順に選択します。
   [![例:VM ダッシュボード](media/azure-stack-manage-vm-disks/vm-dashboard-small.png)](media/azure-stack-manage-vm-disks/vm-dashboard.png#lightbox)

2. 以前に作成した VM を選択します。
   ![選択された仮想マシンを示すスクリーンショット。](media/azure-stack-manage-vm-disks/select-a-vm.png)

3. その VM について、 **[ディスク]** を選択し、次に **[データ ディスクの追加]** を選択します。
   [![新しいディスクを VM に接続する方法を示すスクリーンショット。](media/azure-stack-manage-vm-disks/attach-disks-small.png)](media/azure-stack-manage-vm-disks/attach-disks.png#lightbox)

4. データ ディスクに対して:
   * **LUN** を入力します。 LUN には有効な数値を指定してください。
   * **[ディスクの作成]** を選択します。
   ![新しいデータ ディスクの作成方法を示すスクリーンショット。](media/azure-stack-manage-vm-disks/add-a-data-disk-create-disk.png)

5. **[マネージド ディスクの作成]** ブレードで次の操作を行います。
   * **[名前]** にディスクの名前を入力します。
   * 既存の **リソース グループ** を選択するか、新しいリソース グループを作成します。
   * **[場所]** を選択します。 既定では、場所は、OS ディスクを保持するコンテナーに設定されます。
   * **[アカウントの種類]** を選択します。
      ![例:VM に新しいディスクをアタッチする](media/azure-stack-manage-vm-disks/create-manage-disk.png)

    > [!NOTE]  
    > Premium ディスク (SSD) と標準ディスク (HDD) は、Azure Stack Hub の同じストレージ インフラストラクチャによってサポートされています。 これらのパフォーマンスは同じです。

   * **[ソースの種類]** を選択します。

     ディスクは、別のディスクのスナップショットや、ストレージ アカウント内の BLOB から作成できるほか、空のディスクを作成することができます。

      **スナップショット**:スナップショットが利用できる場合は、スナップショットを選択します。 スナップショットは、VM のサブスクリプションと場所から利用できることが必要です。

      **Storage Blob**:
     * ディスク イメージが格納されている BLOB ストレージの URI を追加します。  
     * **[参照]** を選択して、[ストレージ アカウント] ブレードを開きます。 手順については、「[ストレージ アカウントからデータ ディスクを追加する](#add-a-data-disk-from-a-storage-account)」を参照してください。
     * イメージの OS の種類を **[Windows]** 、 **[Linux]** 、 **[なし (データ ディスク)]** のいずれかから選択します。

   * **[サイズ (GiB)]** を選択します。

     ディスクのコストはディスクのサイズに応じて高くなります。

   * **［作成］** を選択します Azure Stack Hub でマネージド ディスクが作成されて検証されます。

6. Azure Stack Hub によってディスクが作成され、VM に接続されると、VM のディスク設定の **[データ ディスク]** に新しいディスクが表示されます。

   [![例:ディスクを表示する](media/azure-stack-manage-vm-disks/view-data-disk-small.png)](media/azure-stack-manage-vm-disks/view-data-disk.png#lightbox)

### <a name="add-a-data-disk-from-a-storage-account"></a>ストレージ アカウントからデータ ディスクを追加する

Azure Stack Hub でのストレージ アカウントの使用について詳しくは、「[Azure Stack Hub ストレージの概要](azure-stack-storage-overview.md)」を参照してください。

1. 使用する **ストレージ アカウント** を選択します。
2. データ ディスクを配置する **コンテナー** を選択します。 必要に応じて、 **[コンテナー]** ブレードで新しいコンテナーを作成できます。 その後、新しいディスクの場所をそのコンテナーに変更できます。 ディスクごとに別のコンテナーを使用する場合は、データ ディスクの分散配置することでパフォーマンスを向上させることができます。
3. **[選択]** を選択して、選択内容を保存します。

    [![コンテナーの選択方法を示すスクリーンショット。](media/azure-stack-manage-vm-disks/select-container-small.png)](media/azure-stack-manage-vm-disks/select-container.png#lightbox)

## <a name="attach-an-existing-data-disk-to-a-vm"></a>VM に既存のデータ ディスクを接続する

1. VM のデータ ディスクとして使用する [.Vhd ファイルを準備します](/azure/virtual-machines/windows/classic/createupload-vhd)。 .vhd ファイルを接続する VM で使用するストレージ アカウントに .vhd ファイルをアップロードします。

    - .vhd ファイルを保持するために使用するコンテナーは、OS ディスクを保持するコンテナーとは別のコンテナーにすることを計画してください。  
    - VHD を Azure にアップロードする前に、「[Prepare a Windows VHD or VHDX to upload to Azure](/azure/virtual-machines/windows/prepare-for-upload-vhd-image?toc=/azure/virtual-machines/windows/toc.json)」(Azure にアップロードする Windows VHD または VHDX を準備する) に従う必要があります。
    - [Managed Disks](/azure/virtual-machines/windows/managed-disks-overview) への移行を開始する前に、「[Plan for the migration to Managed Disks (Managed Disks への移行の計画)](/azure/virtual-machines/windows/on-prem-to-azure#plan-for-the-migration-to-managed-disks)」をご確認ください。

    ![例:VHD ファイルのアップロード](media/azure-stack-manage-vm-disks/upload-vhd.png)

2. .vhd ファイルをアップロードしたら、VM に VHD をアタッチできます。 左側のメニューで **[仮想マシン]** を選択します。  
 ![選択された仮想マシンを示すスクリーンショット。](media/azure-stack-manage-vm-disks/vm-dashboard.png)

3. 一覧から VM を選択します。

    ![例:ダッシュボードで VM を選択する](media/azure-stack-manage-vm-disks/select-a-vm.png)

4. VM のページで、 **[ディスク]** を選択し、次に **[既存のディスクの接続]** を選択します。

    ![例:既存のディスクの接続](media/azure-stack-manage-vm-disks/attach-disks2.png)

5. **[既存のディスクの接続]** ページで、 **[VHD ファイル]** を選択します。 **[ストレージ アカウント]** ページが開きます。

    ![例:VHD ファイルを選択する](media/azure-stack-manage-vm-disks/select-vhd.png)

6. **[ストレージ アカウント]** で、使用するアカウントを選択し、前にアップロードした .vhd ファイルを保持するコンテナーを選択します。 .vhd ファイルを選択し、 **[選択]** を選択して、選択内容を保存します。

    ![例:コンテナーを選択する](media/azure-stack-manage-vm-disks/select-container2.png)

7. **[既存のディスクの接続]** の **[VHD ファイル]** に、選択したファイルが表示されます。 ディスクの **[ホスト キャッシュ]** 設定を更新し、 **[OK]** を選択して VM の新しいディスク構成を保存します。

    ![例:VHD ファイルをアタッチする](media/azure-stack-manage-vm-disks/attach-vhd.png)

8. Azure Stack Hub でディスクが作成され、VM に接続されると、VM のディスク設定の **[データ ディスク]** に新しいディスクが表示されます。

    ![例:ディスクのアタッチを完了する](media/azure-stack-manage-vm-disks/complete-disk-attach.png)

## <a name="use-powershell-to-add-multiple-disks-to-a-vm"></a>PowerShell を使用して複数のディスクを VM に追加する

PowerShell を使用して VM をプロビジョニングし、新しいデータ ディスクを追加するか、既存のマネージド ディスクまたは .vhd ファイルをデータ ディスクとして接続できます。

**Add-AzVMDataDisk** コマンドレットを使用すると、VM にデータ ディスクが追加されます。 データ ディスクは VM を作成するときに追加できます。または、既存の VM にデータ ディスクを追加できます。 アンマネージド ディスクの場合は、**VhdUri** パラメーターを指定して、別々のコンテナーにディスクを分散させます。

### <a name="add-data-disks-to-a-new-vm"></a>**新しい** VM にデータ ディスクを追加する

次の例では、PowerShell コマンドを使用して、3 つのデータ ディスクがある VM を作成します。 マネージド ディスクまたはアンマネージド ディスクを使用する場合の若干の違いにより、コマンドは複数の部分で提供されます。

#### <a name="create-virtual-machine-configuration-and-network-resources"></a>仮想マシン構成とネットワーク リソースの作成

次のスクリプトは、VM オブジェクトを作成し、それを `$VirtualMachine` 変数に格納します。 このコマンドにより、VM に名前とサイズが割り当てられ、VM のネットワーク リソース (仮想ネットワーク、サブネット、仮想ネットワーク アダプター、NSG、パブリック IP アドレス) が作成されます。

### <a name="az-modules"></a>[Az モジュール](#tab/az1)

```powershell
# Create new virtual machine configuration
$VirtualMachine = New-AzVMConfig -VMName "VirtualMachine" `
                                      -VMSize "Standard_A2"

# Set variables
$rgName = "myResourceGroup"
$location = "local"

# Create a subnet configuration
$subnetName = "mySubNet"
$singleSubnet = New-AzVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix 10.0.0.0/24

# Create a vnet configuration
$vnetName = "myVnetName"
$vnet = New-AzVirtualNetwork -Name $vnetName -ResourceGroupName $rgName -Location $location `
                                  -AddressPrefix 10.0.0.0/16 -Subnet $singleSubnet

# Create a public IP
$ipName = "myIP"
$pip = New-AzPublicIpAddress -Name $ipName -ResourceGroupName $rgName -Location $location `
                                  -AllocationMethod Dynamic

# Create a network security group configuration
$nsgName = "myNsg"
$rdpRule = New-AzNetworkSecurityRuleConfig -Name myRdpRule -Description "Allow RDP" `
                                                -Access Allow -Protocol Tcp -Direction Inbound -Priority 110 `
                                                -SourceAddressPrefix Internet -SourcePortRange * `
                                                -DestinationAddressPrefix * -DestinationPortRange 3389
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $rgName -Location $location `
                                       -Name $nsgName -SecurityRules $rdpRule

# Create a NIC configuration
$nicName = "myNicName"
$nic = New-AzNetworkInterface -Name $nicName -ResourceGroupName $rgName `
                                   -Location $location -SubnetId $vnet.Subnets[0].Id `
                                   -NetworkSecurityGroupId $nsg.Id -PublicIpAddressId $pip.Id

```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm1)

```powershell
# Create new virtual machine configuration
$VirtualMachine = New-AzureRMVMConfig -VMName "VirtualMachine" `
                                      -VMSize "Standard_A2"

# Set variables
$rgName = "myResourceGroup"
$location = "local"

# Create a subnet configuration
$subnetName = "mySubNet"
$singleSubnet = New-AzureRMVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix 10.0.0.0/24

# Create a vnet configuration
$vnetName = "myVnetName"
$vnet = New-AzureRMVirtualNetwork -Name $vnetName -ResourceGroupName $rgName -Location $location `
                                  -AddressPrefix 10.0.0.0/16 -Subnet $singleSubnet

# Create a public IP
$ipName = "myIP"
$pip = New-AzureRMPublicIpAddress -Name $ipName -ResourceGroupName $rgName -Location $location `
                                  -AllocationMethod Dynamic

# Create a network security group configuration
$nsgName = "myNsg"
$rdpRule = New-AzureRMNetworkSecurityRuleConfig -Name myRdpRule -Description "Allow RDP" `
                                                -Access Allow -Protocol Tcp -Direction Inbound -Priority 110 `
                                                -SourceAddressPrefix Internet -SourcePortRange * `
                                                -DestinationAddressPrefix * -DestinationPortRange 3389
$nsg = New-AzureRMNetworkSecurityGroup -ResourceGroupName $rgName -Location $location `
                                       -Name $nsgName -SecurityRules $rdpRule

# Create a NIC configuration
$nicName = "myNicName"
$nic = New-AzureRMNetworkInterface -Name $nicName -ResourceGroupName $rgName `
                                   -Location $location -SubnetId $vnet.Subnets[0].Id `
                                   -NetworkSecurityGroupId $nsg.Id -PublicIpAddressId $pip.Id

```

---




#### <a name="add-managed-disks"></a>マネージド ディスクを追加する

次の 3 つのコマンドは、`$VirtualMachine` に格納されている仮想マシンにマネージド データ ディスクを追加します。 各コマンドでは、ディスクの名前および追加のプロパティを指定します。

### <a name="az-modules"></a>[Az モジュール](#tab/az2)

```powershell
$VirtualMachine = Add-AzVMDataDisk -VM $VirtualMachine -Name 'DataDisk1' `
                                        -Caching 'ReadOnly' -DiskSizeInGB 10 -Lun 0 `
                                        -CreateOption Empty
```

```powershell
$VirtualMachine = Add-AzVMDataDisk -VM $VirtualMachine -Name 'DataDisk2' `
                                        -Caching 'ReadOnly' -DiskSizeInGB 11 -Lun 1 `
                                        -CreateOption Empty
```

```powershell
$VirtualMachine = Add-AzVMDataDisk -VM $VirtualMachine -Name 'DataDisk3' `
                                        -Caching 'ReadOnly' -DiskSizeInGB 12 -Lun 2 `
                                        -CreateOption Empty
```

次のコマンドは、`$VirtualMachine` に格納されている仮想マシンにマネージド ディスクとして OS ディスクを追加します。

```powershell
# Set OS Disk
$osDiskName = "osDisk"
$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name $osDiskName  `
                                      -CreateOption FromImage -Windows
```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm2)

```powershell
$VirtualMachine = Add-AzureRMVMDataDisk -VM $VirtualMachine -Name 'DataDisk1' `
                                        -Caching 'ReadOnly' -DiskSizeInGB 10 -Lun 0 `
                                        -CreateOption Empty
```

```powershell
$VirtualMachine = Add-AzureRMVMDataDisk -VM $VirtualMachine -Name 'DataDisk2' `
                                        -Caching 'ReadOnly' -DiskSizeInGB 11 -Lun 1 `
                                        -CreateOption Empty
```

```powershell
$VirtualMachine = Add-AzureRMVMDataDisk -VM $VirtualMachine -Name 'DataDisk3' `
                                        -Caching 'ReadOnly' -DiskSizeInGB 12 -Lun 2 `
                                        -CreateOption Empty
```

次のコマンドは、`$VirtualMachine` に格納されている仮想マシンにマネージド ディスクとして OS ディスクを追加します。

```powershell
# Set OS Disk
$osDiskName = "osDisk"
$VirtualMachine = Set-AzureRMVMOSDisk -VM $VirtualMachine -Name $osDiskName  `
                                      -CreateOption FromImage -Windows
```

---



#### <a name="add-unmanaged-disks"></a>アンマネージド ディスクを追加する

次の 3 つのコマンドは、3 つのアンマネージド データ ディスクのパスを `$DataDiskVhdUri01`、 `$DataDiskVhdUri02`、`$DataDiskVhdUri03` の各変数に割り当てます。 URL に異なるパス名を定義して、ディスクを別々のコンテナーに分散させます。

```powershell
$DataDiskVhdUri01 = "https://contoso.blob.local.azurestack.external/test1/data1.vhd"
```

```powershell
$DataDiskVhdUri02 = "https://contoso.blob.local.azurestack.external/test2/data2.vhd"
```

```powershell
$DataDiskVhdUri03 = "https://contoso.blob.local.azurestack.external/test3/data3.vhd"
```

次の 3 つのコマンドは、`$VirtualMachine` に格納されている仮想マシンにデータ ディスクを追加します。 各コマンドでは、ディスクの名前および追加のプロパティを指定します。 各ディスクの URI は、`$DataDiskVhdUri01`、`$DataDiskVhdUri02`、および `$DataDiskVhdUri03` に格納されています。

### <a name="az-modules"></a>[Az モジュール](#tab/az3)

```powershell
$VirtualMachine = Add-AzVMDataDisk -VM $VirtualMachine -Name 'DataDisk1' `
                                        -Caching 'ReadOnly' -DiskSizeInGB 10 -Lun 0 `
                                        -VhdUri $DataDiskVhdUri01 -CreateOption Empty
```

```powershell
$VirtualMachine = Add-AzVMDataDisk -VM $VirtualMachine -Name 'DataDisk2' `
                                        -Caching 'ReadOnly' -DiskSizeInGB 11 -Lun 1 `
                                        -VhdUri $DataDiskVhdUri02 -CreateOption Empty
```

```powershell
$VirtualMachine = Add-AzVMDataDisk -VM $VirtualMachine -Name 'DataDisk3' `
                                        -Caching 'ReadOnly' -DiskSizeInGB 12 -Lun 2 `
                                        -VhdUri $DataDiskVhdUri03 -CreateOption Empty
```

次のコマンドは、`$VirtualMachine` に格納されている仮想マシンにアンマネージド OS ディスクを追加します。

```powershell
# Set OS Disk
$osDiskUri = "https://contoso.blob.local.azurestack.external/vhds/osDisk.vhd"
$osDiskName = "osDisk"
$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name $osDiskName -VhdUri $osDiskUri `
                                      -CreateOption FromImage -Windows
```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm3)
 
```powershell
$VirtualMachine = Add-AzureRMVMDataDisk -VM $VirtualMachine -Name 'DataDisk1' `
                                        -Caching 'ReadOnly' -DiskSizeInGB 10 -Lun 0 `
                                        -VhdUri $DataDiskVhdUri01 -CreateOption Empty
```

```powershell
$VirtualMachine = Add-AzureRMVMDataDisk -VM $VirtualMachine -Name 'DataDisk2' `
                                        -Caching 'ReadOnly' -DiskSizeInGB 11 -Lun 1 `
                                        -VhdUri $DataDiskVhdUri02 -CreateOption Empty
```

```powershell
$VirtualMachine = Add-AzureRMVMDataDisk -VM $VirtualMachine -Name 'DataDisk3' `
                                        -Caching 'ReadOnly' -DiskSizeInGB 12 -Lun 2 `
                                        -VhdUri $DataDiskVhdUri03 -CreateOption Empty
```

次のコマンドは、`$VirtualMachine` に格納されている仮想マシンにアンマネージド OS ディスクを追加します。

```powershell
# Set OS Disk
$osDiskUri = "https://contoso.blob.local.azurestack.external/vhds/osDisk.vhd"
$osDiskName = "osDisk"
$VirtualMachine = Set-AzureRMVMOSDisk -VM $VirtualMachine -Name $osDiskName -VhdUri $osDiskUri `
                                      -CreateOption FromImage -Windows
```

---



#### <a name="create-new-virtual-machine"></a>新しい仮想マシンの作成

次の PowerShell コマンドを使用して、OS イメージを設定し、ネットワーク構成を VM に追加した後、新しい VM を起動します。

### <a name="az-modules"></a>[Az モジュール](#tab/az4)

```powershell
#Create the new VM
$VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName VirtualMachine -ProvisionVMAgent | `
                  Set-AzVMSourceImage -PublisherName MicrosoftWindowsServer -Offer WindowsServer `
                  -Skus 2016-Datacenter -Version latest | Add-AzVMNetworkInterface -Id $nic.Id

New-AzVM -ResourceGroupName $rgName -Location $location -VM $VirtualMachine
```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm4)

```powershell
#Create the new VM
$VirtualMachine = Set-AzureRMVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName VirtualMachine -ProvisionVMAgent | `
                  Set-AzureRMVMSourceImage -PublisherName MicrosoftWindowsServer -Offer WindowsServer `
                  -Skus 2016-Datacenter -Version latest | Add-AzureRMVMNetworkInterface -Id $nic.Id

New-AzureRMVM -ResourceGroupName $rgName -Location $location -VM $VirtualMachine
```

---

### <a name="add-data-disks-to-an-existing-vm"></a>既存の VM にデータ ディスクを追加する

次の例では、PowerShell コマンドを使用して、既存の VM に 3 つのデータ ディスクを追加します。

#### <a name="get-virtual-machine"></a>仮想マシンの取得

 最初のコマンドでは、**Get-AzVM** コマンドレットを使用して、**VirtualMachine** という名前の VM を取得します。 このコマンドは VM を `$VirtualMachine` 変数に保存します。

### <a name="az-modules"></a>[Az モジュール](#tab/az5)

```powershell
$VirtualMachine = Get-AzVM -ResourceGroupName "myResourceGroup" `
                                -Name "VirtualMachine"
```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm5)
 
```powershell
$VirtualMachine = Get-AzureRMVM -ResourceGroupName "myResourceGroup" `
                                -Name "VirtualMachine"
```

---

#### <a name="add-managed-disk"></a>マネージド ディスクの追加

次の 3 つのコマンドは、`$VirtualMachine` 変数に格納されている VM にマネージド データ ディスクを追加します。 各コマンドでは、ディスクの名前および追加のプロパティを指定します。

### <a name="az-modules"></a>[Az モジュール](#tab/az6)

```powershell
Add-AzVMDataDisk -VM $VirtualMachine -Name "DataDisk1" -Lun 0 `
                      -Caching ReadOnly -DiskSizeinGB 10 -CreateOption Empty
```

```powershell
Add-AzVMDataDisk -VM $VirtualMachine -Name "DataDisk2" -Lun 1 `
                      -Caching ReadOnly -DiskSizeinGB 11 -CreateOption Empty
```

```powershell
Add-AzVMDataDisk -VM $VirtualMachine -Name "DataDisk3" -Lun 2 `
                      -Caching ReadOnly -DiskSizeinGB 12 -CreateOption Empty
```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm6)
 
```powershell
Add-AzureRMVMDataDisk -VM $VirtualMachine -Name "DataDisk1" -Lun 0 `
                      -Caching ReadOnly -DiskSizeinGB 10 -CreateOption Empty
```

```powershell
Add-AzureRMVMDataDisk -VM $VirtualMachine -Name "DataDisk2" -Lun 1 `
                      -Caching ReadOnly -DiskSizeinGB 11 -CreateOption Empty
```

```powershell
Add-AzureRMVMDataDisk -VM $VirtualMachine -Name "DataDisk3" -Lun 2 `
                      -Caching ReadOnly -DiskSizeinGB 12 -CreateOption Empty
```

---



#### <a name="add-unmanaged-disk"></a>アンマネージド ディスクの追加

次の 3 つのコマンドは、3 つのデータ ディスクのパスを `$DataDiskVhdUri01`、 `$DataDiskVhdUri02`、`$DataDiskVhdUri03` の各変数に割り当てます。 VHD URI 内の異なるパス名は、ディスクを配置する別々のコンテナーを示します。

```powershell
$DataDiskVhdUri01 = "https://contoso.blob.local.azurestack.external/test1/data1.vhd"
```

```powershell
$DataDiskVhdUri02 = "https://contoso.blob.local.azurestack.external/test2/data2.vhd"
```

```powershell
$DataDiskVhdUri03 = "https://contoso.blob.local.azurestack.external/test3/data3.vhd"
```

次の 3 つのコマンドは、`$VirtualMachine` 変数に格納されている VM にデータ ディスクを追加します。 各コマンドは、ディスクの名前、場所、および追加のプロパティを指定します。 各ディスクの URI は、`$DataDiskVhdUri01`、`$DataDiskVhdUri02`、および `$DataDiskVhdUri03` に格納されています。

### <a name="az-modules"></a>[Az モジュール](#tab/az7)

```powershell
Add-AzVMDataDisk -VM $VirtualMachine -Name "DataDisk1" `
                      -VhdUri $DataDiskVhdUri01 -LUN 0 `
                      -Caching ReadOnly -DiskSizeinGB 10 -CreateOption Empty
```

```powershell
Add-AzVMDataDisk -VM $VirtualMachine -Name "DataDisk2" `
                      -VhdUri $DataDiskVhdUri02 -LUN 1 `
                      -Caching ReadOnly -DiskSizeinGB 11 -CreateOption Empty
```

```powershell
Add-AzVMDataDisk -VM $VirtualMachine -Name "DataDisk3" `
                      -VhdUri $DataDiskVhdUri03 -LUN 2 `
                      -Caching ReadOnly -DiskSizeinGB 12 -CreateOption Empty
```

### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm7)

```powershell
Add-AzureRMVMDataDisk -VM $VirtualMachine -Name "DataDisk1" `
                      -VhdUri $DataDiskVhdUri01 -LUN 0 `
                      -Caching ReadOnly -DiskSizeinGB 10 -CreateOption Empty
```

```powershell
Add-AzureRMVMDataDisk -VM $VirtualMachine -Name "DataDisk2" `
                      -VhdUri $DataDiskVhdUri02 -LUN 1 `
                      -Caching ReadOnly -DiskSizeinGB 11 -CreateOption Empty
```

```powershell
Add-AzureRMVMDataDisk -VM $VirtualMachine -Name "DataDisk3" `
                      -VhdUri $DataDiskVhdUri03 -LUN 2 `
                      -Caching ReadOnly -DiskSizeinGB 12 -CreateOption Empty
```


---


#### <a name="update-virtual-machine-state"></a>仮想マシンの状態の更新

次のコマンドは、`-ResourceGroupName` 内の `$VirtualMachine` に格納されている VM の状態を更新します。

### <a name="az-modules"></a>[Az モジュール](#tab/az8)

```powershell
Update-AzVM -ResourceGroupName "myResourceGroup" -VM $VirtualMachine
```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm8)

```powershell
Update-AzureRMVM -ResourceGroupName "myResourceGroup" -VM $VirtualMachine
```

---

## <a name="next-steps"></a>次のステップ

Azure Stack Hub VM の詳細については、「[Azure Stack Hub の仮想マシンに関する考慮事項](azure-stack-vm-considerations.md)」を参照してください。
