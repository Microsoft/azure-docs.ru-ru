---
title: Azure PowerShell. Включение управляемых пользователем ключей с помощью управляемых дисков SSE
description: Включите шифрование на стороне сервера с помощью управляемых клиентом ключей на управляемых дисках с помощью Azure PowerShell.
author: roygara
ms.date: 03/02/2021
ms.topic: how-to
ms.author: rogarana
ms.service: virtual-machines
ms.subservice: disks
ms.openlocfilehash: 37d248fd61cd8fb99259e3776447a719ae365ab9
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105042889"
---
# <a name="azure-powershell---enable-customer-managed-keys-with-server-side-encryption---managed-disks"></a>Azure PowerShell — включить управляемые клиентом ключи с помощью управляемых дисков с шифрованием на стороне сервера

Хранилище дисков Azure позволяет управлять собственными ключами при использовании шифрования на стороне сервера (SSE) для управляемых дисков, если вы решили. Основные сведения о SSE с ключами, управляемыми клиентом, и других типах шифрования управляемых дисков см. в разделе " [ключи, управляемые клиентом](../disk-encryption.md#customer-managed-keys) " статьи о шифровании дисков.

## <a name="restrictions"></a>Ограничения

В настоящее время ключи, управляемые клиентом, имеют следующие ограничения.

- Если эта функция включена для диска, ее нельзя отключить.
    Если нужно обойти это ограничение, необходимо [скопировать все данные](disks-upload-vhd-to-managed-disk-powershell.md#copy-a-managed-disk) на другой управляемый диск, не использующий ключи, управляемые клиентом.
[!INCLUDE [virtual-machines-managed-disks-customer-managed-keys-restrictions](../../../includes/virtual-machines-managed-disks-customer-managed-keys-restrictions.md)]

## <a name="set-up-an-azure-key-vault-and-diskencryptionset-without-automatic-key-rotation"></a>Настройка Azure Key Vault и Дискенкриптионсет без автоматического вращения ключей

Чтобы использовать управляемые клиентом ключи с помощью SSE, необходимо настроить Azure Key Vault и ресурс Дискенкриптионсет.

[!INCLUDE [virtual-machines-disks-encryption-create-key-vault-powershell](../../../includes/virtual-machines-disks-encryption-create-key-vault-powershell.md)]

## <a name="set-up-an-azure-key-vault-and-diskencryptionset-with-automatic-key-rotation-preview"></a>Настройка Azure Key Vault и Дискенкриптионсет с автоматическим сменой ключей (Предварительная версия)

1. Убедитесь, что установлена последняя [версия Azure PowerShell](/powershell/azure/install-az-ps), и вы вошли в учетную запись Azure в с помощью `Connect-AzAccount` .
1. Создайте экземпляр Azure Key Vault и ключ шифрования.

    При создании экземпляра Key Vault необходимо включить защиту от удаления. Защита от удаления гарантирует, что удаленный ключ нельзя удалить навсегда до истечения срока хранения. Этот параметр защищает от потери данных из-за случайного удаления и является обязательным для шифрования управляемых дисков.
    
    ```powershell
    $ResourceGroupName="yourResourceGroupName"
    $LocationName="westcentralus"
    $keyVaultName="yourKeyVaultName"
    $keyName="yourKeyName"
    $keyDestination="Software"
    $diskEncryptionSetName="yourDiskEncryptionSetName"

    $keyVault = New-AzKeyVault -Name $keyVaultName -ResourceGroupName $ResourceGroupName -Location $LocationName -EnablePurgeProtection

    $key = Add-AzKeyVaultKey -VaultName $keyVaultName -Name $keyName -Destination $keyDestination  
    ```

1.  Создайте Дискенкриптионсет с помощью версии API `2020-12-01` и присвоив свойству `rotationToLatestKeyVersionEnabled` значение true через шаблон Azure Resource Manager [CreateDiskEncryptionSetWithAutoKeyRotation.jsв](https://raw.githubusercontent.com/Azure-Samples/managed-disks-powershell-getting-started/master/AutoKeyRotation/CreateDiskEncryptionSetWithAutoKeyRotation.json)
    
    ```powershell
    New-AzResourceGroupDeployment -ResourceGroupName $ResourceGroupName `
    -TemplateUri "https://raw.githubusercontent.com/Azure-Samples/managed-disks-powershell-getting-started/master/AutoKeyRotation/CreateDiskEncryptionSetWithAutoKeyRotation.json" `
    -diskEncryptionSetName $diskEncryptionSetName `
    -keyVaultId $($keyVault.ResourceId) `
    -keyVaultKeyUrl $($key.Key.Kid) `
    -encryptionType "EncryptionAtRestWithCustomerKey" `
    -region $LocationName
    ```

1.  Предоставьте ресурсу DiskEncryptionSet доступ к хранилищу ключей.

    > [!NOTE]
    > Создание удостоверения DiskEncryptionSet в Azure Active Directory может занять несколько минут. Если при выполнении следующей команды появляется сообщение об ошибке вида "Не удается найти объект Active Directory", подождите несколько минут и повторите попытку.

    ```powershell
    $des=Get-AzDiskEncryptionSet -Name $diskEncryptionSetName -ResourceGroupName $ResourceGroupName
    Set-AzKeyVaultAccessPolicy -VaultName $keyVaultName -ObjectId $des.Identity.PrincipalId -PermissionsToKeys wrapkey,unwrapkey,get
    ```

## <a name="examples"></a>Примеры

Теперь, когда вы создали и настроили эти ресурсы, их можно использовать для защиты управляемых дисков. Ниже приведены примеры сценариев, каждый из которых имеет соответствующий сценарий, который можно использовать для защиты управляемых дисков.

### <a name="create-a-vm-using-a-marketplace-image-encrypting-the-os-and-data-disks-with-customer-managed-keys"></a>Создание виртуальной машины с помощью образа Marketplace, шифрование дисков операционной системы и данных с помощью управляемых клиентом ключей

Скопируйте сценарий, замените все примеры значений собственными параметрами, а затем запустите его.

```powershell
$VMLocalAdminUser = "yourVMLocalAdminUserName"
$VMLocalAdminSecurePassword = ConvertTo-SecureString <password> -AsPlainText -Force
$LocationName = "yourRegion"
$ResourceGroupName = "yourResourceGroupName"
$ComputerName = "yourComputerName"
$VMName = "yourVMName"
$VMSize = "yourVMSize"
$diskEncryptionSetName="yourdiskEncryptionSetName"
    
$NetworkName = "yourNetworkName"
$NICName = "yourNICName"
$SubnetName = "yourSubnetName"
$SubnetAddressPrefix = "10.0.0.0/24"
$VnetAddressPrefix = "10.0.0.0/16"
    
$SingleSubnet = New-AzVirtualNetworkSubnetConfig -Name $SubnetName -AddressPrefix $SubnetAddressPrefix
$Vnet = New-AzVirtualNetwork -Name $NetworkName -ResourceGroupName $ResourceGroupName -Location $LocationName -AddressPrefix $VnetAddressPrefix -Subnet $SingleSubnet
$NIC = New-AzNetworkInterface -Name $NICName -ResourceGroupName $ResourceGroupName -Location $LocationName -SubnetId $Vnet.Subnets[0].Id
    
$Credential = New-Object System.Management.Automation.PSCredential ($VMLocalAdminUser, $VMLocalAdminSecurePassword);
    
$VirtualMachine = New-AzVMConfig -VMName $VMName -VMSize $VMSize
$VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName $ComputerName -Credential $Credential -ProvisionVMAgent -EnableAutoUpdate
$VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $NIC.Id
$VirtualMachine = Set-AzVMSourceImage -VM $VirtualMachine -PublisherName 'MicrosoftWindowsServer' -Offer 'WindowsServer' -Skus '2012-R2-Datacenter' -Version latest

$diskEncryptionSet=Get-AzDiskEncryptionSet -ResourceGroupName $ResourceGroupName -Name $diskEncryptionSetName

$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name $($VMName +"_OSDisk") -DiskEncryptionSetId $diskEncryptionSet.Id -CreateOption FromImage

$VirtualMachine = Add-AzVMDataDisk -VM $VirtualMachine -Name $($VMName +"DataDisk1") -DiskSizeInGB 128 -StorageAccountType Premium_LRS -CreateOption Empty -Lun 0 -DiskEncryptionSetId $diskEncryptionSet.Id 
    
New-AzVM -ResourceGroupName $ResourceGroupName -Location $LocationName -VM $VirtualMachine -Verbose
```

### <a name="create-an-empty-disk-encrypted-using-server-side-encryption-with-customer-managed-keys-and-attach-it-to-a-vm"></a>Создайте пустой диск, зашифрованный с помощью шифрования на стороне сервера, с ключами, управляемыми клиентом, и подключите его к виртуальной машине.

Скопируйте сценарий, замените все примеры значений собственными параметрами, а затем запустите его.

```PowerShell
$vmName = "yourVMName"
$LocationName = "westcentralus"
$ResourceGroupName = "yourResourceGroupName"
$diskName = "yourDiskName"
$diskSKU = "Premium_LRS"
$diskSizeinGiB = 30
$diskLUN = 1
$diskEncryptionSetName="yourDiskEncryptionSetName"


$vm = Get-AzVM -Name $vmName -ResourceGroupName $ResourceGroupName 

$diskEncryptionSet=Get-AzDiskEncryptionSet -ResourceGroupName $ResourceGroupName -Name $diskEncryptionSetName

$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Empty -DiskSizeInGB $diskSizeinGiB -StorageAccountType $diskSKU -Lun $diskLUN -DiskEncryptionSetId $diskEncryptionSet.Id 

Update-AzVM -ResourceGroupName $ResourceGroupName -VM $vm

```

### <a name="encrypt-existing-managed-disks"></a>Шифрование существующих управляемых дисков 

Существующие диски не должны быть подключены к работающей виртуальной машине, чтобы их можно было зашифровать с помощью следующего скрипта.

```PowerShell
$rgName = "yourResourceGroupName"
$diskName = "yourDiskName"
$diskEncryptionSetName = "yourDiskEncryptionSetName"
 
$diskEncryptionSet = Get-AzDiskEncryptionSet -ResourceGroupName $rgName -Name $diskEncryptionSetName
 
New-AzDiskUpdateConfig -EncryptionType "EncryptionAtRestWithCustomerKey" -DiskEncryptionSetId $diskEncryptionSet.Id | Update-AzDisk -ResourceGroupName $rgName -DiskName $diskName
```

### <a name="encrypt-an-existing-virtual-machine-scale-set-with-sse-and-customer-managed-keys"></a>Шифрование существующего масштабируемого набора виртуальных машин с помощью SSE и ключей, управляемых клиентом 

Скопируйте сценарий, замените все примеры значений собственными параметрами, а затем запустите его:

```powershell
#set variables 
$vmssname = "name of the vmss that is already created"
$diskencryptionsetname = "name of the diskencryptionset already created"
$vmssrgname = "vmss resourcegroup name"
$diskencryptionsetrgname = "diskencryptionset resourcegroup name"

#get vmss object and create diskencryptionset object attach to vmss os disk
$ssevmss = get-azvmss -ResourceGroupName $vmssrgname -VMScaleSetName $vmssname
$ssevmss.VirtualMachineProfile.StorageProfile.OsDisk.ManagedDisk.DiskEncryptionSet = New-Object -TypeName Microsoft.Azure.Management.Compute.Models.DiskEncryptionSetParameters

#get diskencryption object and retrieve the resource id
$des = Get-AzDiskEncryptionSet -ResourceGroupName $diskencryptionsetrgname -Name $diskencryptionsetname
write-host "the diskencryptionset resource id is:" $des.Id

#associate DES resource id to os disk and update vmss 
$ssevmss.VirtualMachineProfile.StorageProfile.OsDisk.ManagedDisk.DiskEncryptionSet.id = $des.Id
$ssevmss | update-azvmss
```

### <a name="create-a-virtual-machine-scale-set-using-a-marketplace-image-encrypting-the-os-and-data-disks-with-customer-managed-keys"></a>Создание масштабируемого набора виртуальных машин с помощью образа Marketplace, шифрование дисков операционной системы и данных с помощью управляемых клиентом ключей

Скопируйте сценарий, замените все примеры значений собственными параметрами, а затем запустите его.

```PowerShell
$VMLocalAdminUser = "yourLocalAdminUser"
$VMLocalAdminSecurePassword = ConvertTo-SecureString Password@123 -AsPlainText -Force
$LocationName = "westcentralus"
$ResourceGroupName = "yourResourceGroupName"
$ComputerNamePrefix = "yourComputerNamePrefix"
$VMScaleSetName = "yourVMSSName"
$VMSize = "Standard_DS3_v2"
$diskEncryptionSetName="yourDiskEncryptionSetName"
    
$NetworkName = "yourVNETName"
$SubnetName = "yourSubnetName"
$SubnetAddressPrefix = "10.0.0.0/24"
$VnetAddressPrefix = "10.0.0.0/16"
    
$SingleSubnet = New-AzVirtualNetworkSubnetConfig -Name $SubnetName -AddressPrefix $SubnetAddressPrefix

$Vnet = New-AzVirtualNetwork -Name $NetworkName -ResourceGroupName $ResourceGroupName -Location $LocationName -AddressPrefix $VnetAddressPrefix -Subnet $SingleSubnet

$ipConfig = New-AzVmssIpConfig -Name "myIPConfig" -SubnetId $Vnet.Subnets[0].Id 

$VMSS = New-AzVmssConfig -Location $LocationName -SkuCapacity 2 -SkuName $VMSize -UpgradePolicyMode 'Automatic'

$VMSS = Add-AzVmssNetworkInterfaceConfiguration -Name "myVMSSNetworkConfig" -VirtualMachineScaleSet $VMSS -Primary $true -IpConfiguration $ipConfig

$diskEncryptionSet=Get-AzDiskEncryptionSet -ResourceGroupName $ResourceGroupName -Name $diskEncryptionSetName

# Enable encryption at rest with customer managed keys for OS disk by setting DiskEncryptionSetId property 

$VMSS = Set-AzVmssStorageProfile $VMSS -OsDiskCreateOption "FromImage" -DiskEncryptionSetId $diskEncryptionSet.Id -ImageReferenceOffer 'WindowsServer' -ImageReferenceSku '2012-R2-Datacenter' -ImageReferenceVersion latest -ImageReferencePublisher 'MicrosoftWindowsServer'

$VMSS = Set-AzVmssOsProfile $VMSS -ComputerNamePrefix $ComputerNamePrefix -AdminUsername $VMLocalAdminUser -AdminPassword $VMLocalAdminSecurePassword

# Add a data disk encrypted at rest with customer managed keys by setting DiskEncryptionSetId property 

$VMSS = Add-AzVmssDataDisk -VirtualMachineScaleSet $VMSS -CreateOption Empty -Lun 1 -DiskSizeGB 128 -StorageAccountType Premium_LRS -DiskEncryptionSetId $diskEncryptionSet.Id

$Credential = New-Object System.Management.Automation.PSCredential ($VMLocalAdminUser, $VMLocalAdminSecurePassword);

New-AzVmss -VirtualMachineScaleSet $VMSS -ResourceGroupName $ResourceGroupName -VMScaleSetName $VMScaleSetName
```

### <a name="change-the-key-of-a-diskencryptionset-to-rotate-the-key-for-all-the-resources-referencing-the-diskencryptionset"></a>Смена ключа DiskEncryptionSet для всех ресурсов, ссылающихся на DiskEncryptionSet

Скопируйте сценарий, замените все примеры значений собственными параметрами, а затем запустите его.

```PowerShell
$ResourceGroupName="yourResourceGroupName"
$keyVaultName="yourKeyVaultName"
$keyName="yourKeyName"
$diskEncryptionSetName="yourDiskEncryptionSetName"

$keyVault = Get-AzKeyVault -VaultName $keyVaultName -ResourceGroupName $ResourceGroupName

$keyVaultKey = Get-AzKeyVaultKey -VaultName $keyVaultName -Name $keyName

Update-AzDiskEncryptionSet -Name $diskEncryptionSetName -ResourceGroupName $ResourceGroupName -SourceVaultId $keyVault.ResourceId -KeyUrl $keyVaultKey.Id
```

### <a name="find-the-status-of-server-side-encryption-of-a-disk"></a>Определение состояния шифрования диска на стороне сервера

[!INCLUDE [virtual-machines-disks-encryption-status-powershell](../../../includes/virtual-machines-disks-encryption-status-powershell.md)]

> [!IMPORTANT]
> Управляемые клиентом ключи используют управляемые удостоверения для ресурсов Azure — функцию Azure Active Directory (Azure AD). При настройке управляемых пользователем ключей управляемое удостоверение автоматически назначается вашим ресурсам. Если впоследствии вы перемещаете подписку, группу ресурсов или управляемый диск из одного каталога Azure AD в другой, управляемое удостоверение, связанное с управляемыми дисками, не передается в новый клиент, поэтому управляемые клиентом ключи могут перестать работать. Дополнительные сведения см. в статье [Передача подписки между каталогами Azure AD](../../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories).

## <a name="next-steps"></a>Дальнейшие действия

- [Изучите шаблоны Azure Resource Manager для создания зашифрованных дисков с помощью управляемых клиентом ключей](https://github.com/ramankumarlive/manageddiskscmkpreview)
- [Репликация компьютеров, использующих диски с ключами, управляемыми клиентом](../../site-recovery/azure-to-azure-how-to-enable-replication-cmk-disks.md)
- [Настройка аварийного восстановления виртуальных машин VMware в Azure с помощью PowerShell](../../site-recovery/vmware-azure-disaster-recovery-powershell.md#replicate-vmware-vms)
- [Настройка аварийного восстановления виртуальных машин Hyper-V в Azure с помощью PowerShell и Azure Resource Manager](../../site-recovery/hyper-v-azure-powershell-resource-manager.md#step-7-enable-vm-protection)
