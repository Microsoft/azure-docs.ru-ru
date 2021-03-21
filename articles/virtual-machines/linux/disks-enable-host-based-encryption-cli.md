---
title: Включение сквозного шифрования с помощью шифрования на управляемых дисках Host-Azure CLI
description: Используйте шифрование на узле, чтобы включить сквозное шифрование на управляемых дисках Azure.
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 08/24/2020
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: 3eecb584f468bc170f0325da8d734a1890691483
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104601777"
---
# <a name="use-the-azure-cli-to-enable-end-to-end-encryption-using-encryption-at-host"></a>Использование Azure CLI для включения сквозного шифрования с помощью шифрования на узле

При включении шифрования на узле данные, хранящиеся на узле виртуальной машины, шифруются при хранении и передаются в зашифрованном виде в службу хранилища. Концептуальные сведения о шифровании на узле, а также о других типах шифрования управляемых дисков см. [в статье шифрование на основе шифрования узла для данных виртуальной машины](../disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data).

## <a name="restrictions"></a>Ограничения

[!INCLUDE [virtual-machines-disks-encryption-at-host-restrictions](../../../includes/virtual-machines-disks-encryption-at-host-restrictions.md)]

### <a name="supported-vm-sizes"></a>Поддерживаемые размеры виртуальных машин

[!INCLUDE [virtual-machines-disks-encryption-at-host-suported-sizes](../../../includes/virtual-machines-disks-encryption-at-host-suported-sizes.md)]

Размеры виртуальных машин также можно найти программно. Чтобы узнать, как получить их программно, ознакомьтесь с разделом [Поиск поддерживаемых размеров виртуальных машин](#finding-supported-vm-sizes) .

## <a name="prerequisites"></a>Предварительные требования

Прежде чем использовать свойство Енкриптионасост для виртуальной машины или VMSS, необходимо включить эту функцию для своей подписки. Выполните следующие действия, чтобы включить функцию для подписки.

1.  Выполните следующую команду, чтобы зарегистрировать компонент для подписки.

    ```azurecli
    az feature register --namespace Microsoft.Compute --name EncryptionAtHost
    ```
 
2.  Убедитесь, что состояние регистрации зарегистрировано (займет несколько минут), выполнив приведенную ниже команду, прежде чем пытаться воспользоваться функцией.

    ```azurecli
    az feature show --namespace Microsoft.Compute --name EncryptionAtHost
    ```


### <a name="create-an-azure-key-vault-and-diskencryptionset"></a>Создание Azure Key Vault и Дискенкриптионсет

После включения этой функции необходимо настроить Azure Key Vault и Дискенкриптионсет, если вы еще этого не сделали.

[!INCLUDE [virtual-machines-disks-encryption-create-key-vault-cli](../../../includes/virtual-machines-disks-encryption-create-key-vault-cli.md)]

## <a name="examples"></a>Примеры

### <a name="create-a-vm-with-encryption-at-host-enabled-with-customer-managed-keys"></a>Создайте виртуальную машину с шифрованием на узле, поддерживающем ключи, управляемые клиентом. 

Создайте виртуальную машину с управляемыми дисками с помощью URI ресурса Дискенкриптионсет, созданного ранее, чтобы зашифровать кэш ОС и дисков данных с помощью управляемых клиентом ключей. Временные диски шифруются с помощью ключей, управляемых платформой. 

```azurecli
rgName=yourRGName
vmName=yourVMName
location=eastus
vmSize=Standard_DS2_v2
image=UbuntuLTS 
diskEncryptionSetName=yourDiskEncryptionSetName

diskEncryptionSetId=$(az disk-encryption-set show -n $diskEncryptionSetName -g $rgName --query [id] -o tsv)

az vm create -g $rgName \
-n $vmName \
-l $location \
--encryption-at-host \
--image $image \
--size $vmSize \
--generate-ssh-keys \
--os-disk-encryption-set $diskEncryptionSetId \
--data-disk-sizes-gb 128 128 \
--data-disk-encryption-sets $diskEncryptionSetId $diskEncryptionSetId
```

### <a name="create-a-vm-with-encryption-at-host-enabled-with-platform-managed-keys"></a>Создайте виртуальную машину с шифрованием на узле, поддерживающем ключи, управляемые платформой. 

Создайте виртуальную машину с шифрованием на узле, чтобы шифровать кэш ОС, дисков данных и временных дисков с ключами, управляемыми платформой. 

```azurecli
rgName=yourRGName
vmName=yourVMName
location=eastus
vmSize=Standard_DS2_v2
image=UbuntuLTS 

az vm create -g $rgName \
-n $vmName \
-l $location \
--encryption-at-host \
--image $image \
--size $vmSize \
--generate-ssh-keys \
--data-disk-sizes-gb 128 128 \
```

### <a name="update-a-vm-to-enable-encryption-at-host"></a>Обновите виртуальную машину, чтобы включить шифрование на узле. 

```azurecli
rgName=yourRGName
vmName=yourVMName

az vm update -n $vmName \
-g $rgName \
--set securityProfile.encryptionAtHost=true
```

### <a name="check-the-status-of-encryption-at-host-for-a-vm"></a>Проверка состояния шифрования на узле виртуальной машины

```azurecli
rgName=yourRGName
vmName=yourVMName

az vm show -n $vmName \
-g $rgName \
--query [securityProfile.encryptionAtHost] -o tsv
```

### <a name="create-a-virtual-machine-scale-set-with-encryption-at-host-enabled-with-customer-managed-keys"></a>Создайте масштабируемый набор виртуальных машин с шифрованием на узле, поддерживающем ключи, управляемые клиентом. 

Создайте масштабируемый набор виртуальных машин с управляемыми дисками с помощью URI ресурса Дискенкриптионсет, созданного ранее, чтобы зашифровать кэш ОС и дисков данных с помощью управляемых клиентом ключей. Временные диски шифруются с помощью ключей, управляемых платформой. 

```azurecli
rgName=yourRGName
vmssName=yourVMSSName
location=westus2
vmSize=Standard_DS3_V2
image=UbuntuLTS 
diskEncryptionSetName=yourDiskEncryptionSetName

diskEncryptionSetId=$(az disk-encryption-set show -n $diskEncryptionSetName -g $rgName --query [id] -o tsv)

az vmss create -g $rgName \
-n $vmssName \
--encryption-at-host \
--image UbuntuLTS \
--upgrade-policy automatic \
--admin-username azureuser \
--generate-ssh-keys \
--os-disk-encryption-set $diskEncryptionSetId \
--data-disk-sizes-gb 64 128 \
--data-disk-encryption-sets $diskEncryptionSetId $diskEncryptionSetId
```

### <a name="create-a-virtual-machine-scale-set-with-encryption-at-host-enabled-with-platform-managed-keys"></a>Создайте масштабируемый набор виртуальных машин с шифрованием на узле, поддерживающем ключи, управляемые платформой. 

Создайте масштабируемый набор виртуальных машин с шифрованием на узле, чтобы шифровать кэш ОС, дисков данных и временных дисков с ключами, управляемыми платформой. 

```azurecli
rgName=yourRGName
vmssName=yourVMSSName
location=westus2
vmSize=Standard_DS3_V2
image=UbuntuLTS 

az vmss create -g $rgName \
-n $vmssName \
--encryption-at-host \
--image UbuntuLTS \
--upgrade-policy automatic \
--admin-username azureuser \
--generate-ssh-keys \
--data-disk-sizes-gb 64 128 \
```

### <a name="update-a-virtual-machine-scale-set-to-enable-encryption-at-host"></a>Обновите масштабируемый набор виртуальных машин, чтобы включить шифрование на узле. 

```azurecli
rgName=yourRGName
vmssName=yourVMName

az vmss update -n $vmssName \
-g $rgName \
--set virtualMachineProfile.securityProfile.encryptionAtHost=true
```

### <a name="check-the-status-of-encryption-at-host-for-a-virtual-machine-scale-set"></a>Проверка состояния шифрования на узле для масштабируемого набора виртуальных машин

```azurecli
rgName=yourRGName
vmssName=yourVMName

az vmss show -n $vmssName \
-g $rgName \
--query [virtualMachineProfile.securityProfile.encryptionAtHost] -o tsv
```

## <a name="finding-supported-vm-sizes"></a>Поиск поддерживаемых размеров виртуальных машин

Размеры виртуальных машин прежних версий не поддерживаются. Список поддерживаемых размеров виртуальных машин можно найти одним из следующих форматов:

Вызов [API SKU ресурсов](/rest/api/compute/resourceskus/list) и проверка `EncryptionAtHostSupported` возможности установки значения **true**.

```json
    {
        "resourceType": "virtualMachines",
        "name": "Standard_DS1_v2",
        "tier": "Standard",
        "size": "DS1_v2",
        "family": "standardDSv2Family",
        "locations": [
        "CentralUSEUAP"
        ],
        "capabilities": [
        {
            "name": "EncryptionAtHostSupported",
            "value": "True"
        }
        ]
    }
```

Либо вызовите командлет PowerShell [Get-азкомпутересаурцеску](/powershell/module/az.compute/get-azcomputeresourcesku) .

```powershell
$vmSizes=Get-AzComputeResourceSku | where{$_.ResourceType -eq 'virtualMachines' -and $_.Locations.Contains('CentralUSEUAP')} 

foreach($vmSize in $vmSizes)
{
    foreach($capability in $vmSize.capabilities)
    {
        if($capability.Name -eq 'EncryptionAtHostSupported' -and $capability.Value -eq 'true')
        {
            $vmSize

        }

    }
}
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы создали и настроили эти ресурсы, их можно использовать для защиты управляемых дисков. Следующая ссылка содержит примеры сценариев, каждый из которых имеет соответствующий сценарий, который можно использовать для защиты управляемых дисков.

[Примеры шаблонов Azure Resource Manager](https://github.com/Azure-Samples/managed-disks-powershell-getting-started/tree/master/EncryptionAtHost)
