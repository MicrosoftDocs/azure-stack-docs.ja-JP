---
title: Azure Stack Hub マネージド ディスクの相違点と考慮事項
description: Azure Stack Hub でマネージド ディスクとマネージド イメージを操作する際の相違点と考慮事項について説明します。
author: sethmanheim
ms.topic: article
ms.date: 11/22/2020
ms.author: sethm
ms.reviewer: jiahan
ms.lastreviewed: 11/22/2020
ms.openlocfilehash: fd1f827ec9108d1412df544c94044aabf600224a
ms.sourcegitcommit: d542b68b299b73e045f30916afb6018e365e9db6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2021
ms.locfileid: "99975879"
---
# <a name="azure-stack-hub-managed-disks-differences-and-considerations"></a>Azure Stack Hub マネージド ディスク: 相違点と考慮事項

この記事は、Azure Stack Hub の ["*マネージド ディスク*"](azure-stack-manage-vm-disks.md) と [ Azure のマネージド ディスク](/azure/virtual-machines/windows/managed-disks-overview)の相違点をまとめたものです。 Azure Stack Hub と Azure の相違点の概要については、「[主な考慮事項](azure-stack-considerations.md)」をご覧ください。

マネージド ディスクを使用すると、VM ディスクに関連付けられている[ストレージ アカウント](../operator/azure-stack-manage-storage-accounts.md)を管理できるため、IaaS 仮想マシン (VM) のディスク管理が簡素化されます。

Azure Stack Hub ポータルを使用して VM を作成すると、マネージド ディスクが既定で有効になります。
  
## <a name="cheat-sheet-managed-disk-differences"></a>チート シート: マネージド ディスクの相違点

| 特徴量 | Azure (グローバル) | Azure Stack Hub |
| --- | --- | --- |
|保存データの暗号化 |Azure Storage Service Encryption (SSE)、Azure Disk Encryption (ADE)。     |BitLocker 128 ビット AES 暗号化      |
|Image          | マネージド カスタム イメージ |サポートされています|
|バックアップ オプション | Azure Backup サービス |まだサポートされていません |
|ディザスター リカバリーのオプション | Azure Site Recovery |まだサポートされていません|
|ディスクの種類     |Premium SSD、Standard SSD、および Standard HDD。 |Premium SSD、Standard HDD |
|Premium Disk  |完全にサポートされます。 |プロビジョニング可能ですがパフォーマンス制限や保証がありません  |
|Premium Disk の IOP  |ディスク サイズによって異なります。  |ディスクあたりの 2300 IOP |
|Premium ディスクのスループット |ディスク サイズによって異なります。 |ディスクあたり 145 MB/秒 |
|ディスク サイズ  |Azure Premium ディスク:P4 (32 GiB) ～ P80 (32 TiB)<br>Azure Standard SSD ディスク:E10 (128 GiB) ～ E80 (32 TiB)<br>Azure Standard HDD ディスク:S4 (32 GiB) ～ S80 (32 TiB) |M4:32 GiB<br>M6:64 GiB<br>M10:128 GiB<br>M15:256 GiB<br>M20:512 GiB<br>M30:1023 GiB |
|ディスクのスナップショット コピー|実行中の VM にアタッチされている Azure マネージド ディスクのスナップショットはサポートされています。|まだサポートされていません |
|ディスクのパフォーマンス分析 |集計メトリックおよびディスクあたりのメトリックはサポートされています。 |まだサポートされていません |
|移行      |VM を再作成せずに既存のアンマネージド Azure Resource Manager VM から移行するためのツールを提供します。  |まだサポートされていません |

> [!NOTE]  
> Azure Stack Hub のマネージド ディスクの IOP とスループットは、プロビジョニングされた数値ではなく上限数です。これは Azure Stack Hub で実行されているハードウェアとワークロードによって影響を受ける可能性があります。

## <a name="metrics"></a>メトリック

ストレージ メトリックにも相違点があります。

- Azure Stack Hub では、ストレージ メトリックのトランザクション データで、内部と外部のネットワーク帯域幅は区別されません。
- ストレージ メトリックの Azure Stack Hub トランザクション データには、マウントされたディスクへの仮想マシンのアクセスは含まれません。

## <a name="api-versions"></a>API のバージョン

Azure Stack Hub のマネージド ディスクでは、次の API バージョンがサポートされています。

::: moniker range=">=azs-2008"
- 2019-03-01
- 2018-09-30
- 2018-06-01
- 2018-04-01
- 2017-03-30
::: moniker-end

::: moniker range="<=azs-2005"

- 2017-03-30
- 2017-12-01 (マネージド イメージのみ、ディスクなし、スナップショットなし)
::: moniker-end

## <a name="convert-to-managed-disks"></a>マネージド ディスクへの変換

> [!NOTE]  
> Azure PowerShell コマンドレット **ConvertTo-AzVMManagedDisk** を使用して、Azure Stack Hub 内のアンマネージド ディスクをマネージド ディスクに変換することはできません。 Azure Stack Hub では現在、このコマンドレットはサポートされていません。

次のスクリプトを使用して、現在プロビジョニングされている VM をアンマネージド ディスクからマネージド ディスクに変換できます。 プレースホルダーを実際の値に置き換えてください。

### <a name="az-modules"></a>[Az モジュール](#tab/az1)

```powershell
$SubscriptionId = "SubId"

# The name of your resource group where your VM to be converted exists.
$ResourceGroupName ="MyResourceGroup"

# The name of the managed disk to be created.
$DiskName = "mngddisk"

# The size of the disks in GB. It should be greater than the VHD file size.
$DiskSize = "50"

# The URI of the VHD file that will be used to create the managed disk.
# The VHD file can be deleted as soon as the managed disk is created.
$VhdUri = "https://rgmgddisks347.blob.local.azurestack.external/vhds/unmngdvm20181109013817.vhd"

# The storage type for the managed disk: PremiumLRS or StandardLRS.
$AccountType = "StandardLRS"

# The Azure Stack Hub location where the managed disk will be located.
# The location should be the same as the location of the storage account in which VHD file is stored.
# Configure the new managed VM point to the old unmanaged VM configuration (network config, VM name, location).
$Location = "local"
$VirtualMachineName = "unmngdvm"
$VirtualMachineSize = "Standard_D1"
$PIpName = "unmngdvm-ip"
$VirtualNetworkName = "unmngdrg-vnet"
$NicName = "unmngdvm"

# Set the context to the subscription ID in which the managed disk will be created.
Select-AzSubscription -SubscriptionId $SubscriptionId

# Delete old VM, but keep the OS disk.
Remove-AzVm -Name $VirtualMachineName -ResourceGroupName $ResourceGroupName

# Create the managed disk configuration.
$DiskConfig = New-AzDiskConfig -AccountType $AccountType -Location $Location -DiskSizeGB $DiskSize -SourceUri $VhdUri -CreateOption Import

# Create managed disk.
New-AzDisk -DiskName $DiskName -Disk $DiskConfig -ResourceGroupName $resourceGroupName
$Disk = Get-AzDisk -DiskName $DiskName -ResourceGroupName $ResourceGroupName
$VirtualMachine = New-AzVMConfig -VMName $VirtualMachineName -VMSize $VirtualMachineSize

# Use the managed disk resource ID to attach it to the virtual machine.
# Change the OS type to "-Windows" if the OS disk has the Windows OS.
$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -ManagedDiskId $Disk.Id -CreateOption Attach -Linux

# Create a public IP for the VM.
$PublicIp = Get-AzPublicIpAddress -Name $PIpName -ResourceGroupName $ResourceGroupName

# Get the virtual network where the virtual machine will be hosted.
$VNet = Get-AzVirtualNetwork -Name $VirtualNetworkName -ResourceGroupName $ResourceGroupName

# Create NIC in the first subnet of the virtual network.
$Nic = Get-AzNetworkInterface -Name $NicName -ResourceGroupName $ResourceGroupName

$VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $Nic.Id

# Create the virtual machine with managed disk.
New-AzVM -VM $VirtualMachine -ResourceGroupName $ResourceGroupName -Location $Location
```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm1)

```powershell
$SubscriptionId = "SubId"

# The name of your resource group where your VM to be converted exists.
$ResourceGroupName ="MyResourceGroup"

# The name of the managed disk to be created.
$DiskName = "mngddisk"

# The size of the disks in GB. It should be greater than the VHD file size.
$DiskSize = "50"

# The URI of the VHD file that will be used to create the managed disk.
# The VHD file can be deleted as soon as the managed disk is created.
$VhdUri = "https://rgmgddisks347.blob.local.azurestack.external/vhds/unmngdvm20181109013817.vhd"

# The storage type for the managed disk: PremiumLRS or StandardLRS.
$AccountType = "StandardLRS"

# The Azure Stack Hub location where the managed disk will be located.
# The location should be the same as the location of the storage account in which VHD file is stored.
# Configure the new managed VM point to the old unmanaged VM configuration (network config, VM name, location).
$Location = "local"
$VirtualMachineName = "unmngdvm"
$VirtualMachineSize = "Standard_D1"
$PIpName = "unmngdvm-ip"
$VirtualNetworkName = "unmngdrg-vnet"
$NicName = "unmngdvm"

# Set the context to the subscription ID in which the managed disk will be created.
Select-AzureRMSubscription -SubscriptionId $SubscriptionId

# Delete old VM, but keep the OS disk.
Remove-AzureRMVm -Name $VirtualMachineName -ResourceGroupName $ResourceGroupName

# Create the managed disk configuration.
$DiskConfig = New-AzureRMDiskConfig -AccountType $AccountType -Location $Location -DiskSizeGB $DiskSize -SourceUri $VhdUri -CreateOption Import

# Create managed disk.
New-AzureRMDisk -DiskName $DiskName -Disk $DiskConfig -ResourceGroupName $resourceGroupName
$Disk = Get-AzureRMDisk -DiskName $DiskName -ResourceGroupName $ResourceGroupName
$VirtualMachine = New-AzureRMVMConfig -VMName $VirtualMachineName -VMSize $VirtualMachineSize

# Use the managed disk resource ID to attach it to the virtual machine.
# Change the OS type to "-Windows" if the OS disk has the Windows OS.
$VirtualMachine = Set-AzureRMVMOSDisk -VM $VirtualMachine -ManagedDiskId $Disk.Id -CreateOption Attach -Linux

# Create a public IP for the VM.
$PublicIp = Get-AzureRMPublicIpAddress -Name $PIpName -ResourceGroupName $ResourceGroupName

# Get the virtual network where the virtual machine will be hosted.
$VNet = Get-AzureRMVirtualNetwork -Name $VirtualNetworkName -ResourceGroupName $ResourceGroupName

# Create NIC in the first subnet of the virtual network.
$Nic = Get-AzureRMNetworkInterface -Name $NicName -ResourceGroupName $ResourceGroupName

$VirtualMachine = Add-AzureRMVMNetworkInterface -VM $VirtualMachine -Id $Nic.Id

# Create the virtual machine with managed disk.
New-AzureRMVM -VM $VirtualMachine -ResourceGroupName $ResourceGroupName -Location $Location
```

---




## <a name="managed-images"></a>マネージド イメージ

Azure Stack Hub では "*マネージド イメージ*" がサポートされています。これにより、汎用化された VM (アンマネージドとマネージドの両方) でマネージド イメージ オブジェクトを作成できます。以降、このオブジェクトではマネージド ディスク VM のみを作成できます。 マネージド イメージにより、次の 2 つのシナリオが可能になります。

- 汎用化されたアンマネージド VM があり、今後はマネージド ディスクを使用する。
- 汎用化されたマネージド VM があり、同様のマネージド VM を複数作成する。

### <a name="step-1-generalize-the-vm"></a>手順 1:VM の汎用化

Windows の場合は、「[Sysprep を使用して Windows VM を一般化する](/azure/virtual-machines/windows/capture-image-resource#generalize-the-windows-vm-using-sysprep)」のセクションに従います。 Linux の場合は、手順 1 ([こちら](/azure/virtual-machines/linux/capture-image#step-1-deprovision-the-vm)) に従います。

> [!NOTE]
> 必ず VM を一般化してください。 適切に一般化されていないイメージから VM を作成すると、**VMProvisioningTimeout** エラーが発生する可能性があります。

### <a name="step-2-create-the-managed-image"></a>手順 2:マネージド イメージを作成する

ポータル、PowerShell、または CLI を使用してマネージド イメージを作成できます。 [マネージド イメージを作成する](/azure/virtual-machines/windows/capture-image-resource)手順に従います。

### <a name="step-3-choose-the-use-case"></a>手順 3:ユース ケースを選択する

#### <a name="case-1-migrate-unmanaged-vms-to-managed-disks"></a>ケース 1: アンマネージド VM をマネージド ディスクに移行する

この手順を実行する前に、VM を適切に一般化してください。 一般化の後ではその VM を使用できなくなります。 適切に一般化されていないイメージから VM を作成すると、**VMProvisioningTimeout** エラーが発生します。

「[ストレージ アカウントを使用する VM からイメージを作成する](/azure/virtual-machines/windows/capture-image-resource#create-an-image-from-a-vm-that-uses-a-storage-account)」の手順に従って、ストレージ アカウント内の一般化された VHD からマネージド イメージを作成します。 このイメージは、後でマネージド VM を作成するために使用できます。

#### <a name="case-2-create-managed-vm-from-managed-image-using-powershell"></a>ケース 2: PowerShell を使用してマネージド イメージからマネージド VM を作成する

「[PowerShell を使ってマネージド ディスクからイメージを作成する](/azure/virtual-machines/windows/capture-image-resource#create-an-image-from-a-managed-disk-using-powershell)」のスクリプトを使って既存のマネージド ディスク VM からイメージを作成した後、次のサンプル スクリプトを使って、既存のイメージ オブジェクトから類似の Linux VM を作成します。

Azure Stack Hub PowerShell モジュール 1.7.0 以上: [マネージド イメージから VM を作成する](/azure/virtual-machines/windows/create-vm-generalized-managed)の手順に従います。

Azure Stack Hub PowerShell モジュール 1.6.0 以前:

### <a name="az-modules"></a>[Az モジュール](#tab/az2)

```powershell
# Variables for common values
$ResourceGroupName = "MyResourceGroup"
$Location = "local"
$VirtualMachineName = "MyVM"
$ImageRG = "managedlinuxrg"
$ImageName = "simplelinuxvmm-image-2019122"

# Create credential object
$Cred = Get-Credential -Message "Enter a username and password for the virtual machine."

# Create a resource group
New-AzResourceGroup -Name $ResourceGroupName -Location $Location

# Create a subnet configuration
$SubnetConfig = New-AzVirtualNetworkSubnetConfig -Name "MySubnet" -AddressPrefix "192.168.1.0/24"

# Create a virtual network
$VNet = New-AzVirtualNetwork -ResourceGroupName $ResourceGroupName -Location $Location `
  -Name "MyVNet" -AddressPrefix "192.168.0.0/16" -Subnet $SubnetConfig

# Create a public IP address and specify a DNS name
$PIp = New-AzPublicIpAddress -ResourceGroupName $ResourceGroupName -Location $Location `
  -Name "mypublicdns$(Get-Random)" -AllocationMethod Static -IdleTimeoutInMinutes 4

# Create an inbound network security group rule for port 3389
$NsgRuleSSH = New-AzNetworkSecurityRuleConfig -Name "MyNetworkSecurityGroupRuleSSH"  -Protocol Tcp `
  -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
  -DestinationPortRange 22 -Access Allow

# Create a network security group
$Nsg = New-AzNetworkSecurityGroup -ResourceGroupName $ResourceGroupName -Location $Location `
  -Name "MyNetworkSecurityGroup" -SecurityRules $NsgRuleSSH

# Create a virtual network card and associate with public IP address and NSG
$Nic = New-AzNetworkInterface -Name "MyNic" -ResourceGroupName $ResourceGroupName -Location $Location `
  -SubnetId $VNet.Subnets[0].Id -PublicIpAddressId $PIp.Id -NetworkSecurityGroupId $Nsg.Id

$Image = Get-AzImage -ResourceGroupName $ImageRG -ImageName $ImageName

# Create a virtual machine configuration
$VmConfig = New-AzVMConfig -VMName $VirtualMachineName -VMSize "Standard_D1" | `
Set-AzVMOperatingSystem -Linux -ComputerName $VirtualMachineName -Credential $Cred | `
Set-AzVMSourceImage -Id $Image.Id | `
Set-AzVMOSDisk -VM $VmConfig -CreateOption FromImage -Linux | `
Add-AzVMNetworkInterface -Id $Nic.Id

# Create a virtual machine
New-AzVM -ResourceGroupName $ResourceGroupName -Location $Location -VM $VmConfig
```
### <a name="azurerm-modules"></a>[AzureRM モジュール](#tab/azurerm2)

```powershell
# Variables for common values
$ResourceGroupName = "MyResourceGroup"
$Location = "local"
$VirtualMachineName = "MyVM"
$ImageRG = "managedlinuxrg"
$ImageName = "simplelinuxvmm-image-2019122"

# Create credential object
$Cred = Get-Credential -Message "Enter a username and password for the virtual machine."

# Create a resource group
New-AzureRMResourceGroup -Name $ResourceGroupName -Location $Location

# Create a subnet configuration
$SubnetConfig = New-AzureRMVirtualNetworkSubnetConfig -Name "MySubnet" -AddressPrefix "192.168.1.0/24"

# Create a virtual network
$VNet = New-AzureRMVirtualNetwork -ResourceGroupName $ResourceGroupName -Location $Location `
  -Name "MyVNet" -AddressPrefix "192.168.0.0/16" -Subnet $SubnetConfig

# Create a public IP address and specify a DNS name
$PIp = New-AzureRMPublicIpAddress -ResourceGroupName $ResourceGroupName -Location $Location `
  -Name "mypublicdns$(Get-Random)" -AllocationMethod Static -IdleTimeoutInMinutes 4

# Create an inbound network security group rule for port 3389
$NsgRuleSSH = New-AzureRMNetworkSecurityRuleConfig -Name "MyNetworkSecurityGroupRuleSSH"  -Protocol Tcp `
  -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
  -DestinationPortRange 22 -Access Allow

# Create a network security group
$Nsg = New-AzureRMNetworkSecurityGroup -ResourceGroupName $ResourceGroupName -Location $Location `
  -Name "MyNetworkSecurityGroup" -SecurityRules $NsgRuleSSH

# Create a virtual network card and associate with public IP address and NSG
$Nic = New-AzureRMNetworkInterface -Name "MyNic" -ResourceGroupName $ResourceGroupName -Location $Location `
  -SubnetId $VNet.Subnets[0].Id -PublicIpAddressId $PIp.Id -NetworkSecurityGroupId $Nsg.Id

$Image = Get-AzureRMImage -ResourceGroupName $ImageRG -ImageName $ImageName

# Create a virtual machine configuration
$VmConfig = New-AzureRmVMConfig -VMName $VirtualMachineName -VMSize "Standard_D1" | `
Set-AzureRmVMOperatingSystem -Linux -ComputerName $VirtualMachineName -Credential $Cred | `
Set-AzureRmVMSourceImage -Id $Image.Id | `
Set-AzureRmVMOSDisk -VM $VmConfig -CreateOption FromImage -Linux | `
Add-AzureRmVMNetworkInterface -Id $Nic.Id

# Create a virtual machine
New-AzureRMVM -ResourceGroupName $ResourceGroupName -Location $Location -VM $VmConfig
```

---



ポータルを使用して、マネージド イメージから VM を作成することもできます。 詳細については、Azure マネージド イメージに関する記事「[Azure で汎用化された VM のマネージド イメージを作成する](/azure/virtual-machines/windows/capture-image-resource)」および「[マネージド イメージから VM を作成する](/azure/virtual-machines/windows/create-vm-generalized-managed)」を参照してください。

## <a name="configuration"></a>構成

1808 以降の更新プログラムを適用した後、マネージド ディスクを使用する前に次の構成変更を行う必要があります。

- 1808 更新プログラムの前にサブスクリプションが作成された場合は、次の手順に従ってサブスクリプションを更新します。 そうでないと、このサブスクリプションに VM をデプロイする操作がエラー メッセージ "ディスク マネージャーでの内部エラーです" で失敗することがあります。
   1. Azure Stack Hub ユーザー ポータルで、 **[サブスクリプション]** に移動して、サブスクリプションを検索します。 **[リソース プロバイダー]** をクリックし、 **[Microsoft.Compute]** をクリックした後、 **[再登録]** をクリックします。
   2. 同じサブスクリプションで、 **[アクセス制御 (IAM)]** に移動し、 **[Azure Stack Hub - マネージド ディスク]** がリストに含まれていることを確認します。
- マルチテナント環境を使用している場合は、「[Azure Stack Hub でマルチテナントを構成する](../operator/azure-stack-enable-multitenancy.md#configure-guest-directory)」の手順に従って、各ゲスト ディレクトリを再構成するように (お客様の組織内またはサービス プロバイダーからの) クラウド オペレーターに依頼してください。 そうしないと、そのゲスト ディレクトリに関連付けられているサブスクリプションに VM をデプロイした場合に、"ディスク マネージャーでの内部エラーです" というエラー メッセージが表示され、失敗することがあります。

## <a name="next-steps"></a>次のステップ

- [Azure Stack Hub 仮想マシン](azure-stack-compute-overview.md)について確認してください。
