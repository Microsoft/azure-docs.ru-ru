---
title: Создание и шифрование виртуальной машины Linux с помощью Azure Powershell
description: С помощью этого краткого руководства вы узнаете, как создать и зашифровать виртуальную машину Linux, используя Azure PowerShell.
author: msmbaldwin
ms.author: mbaldwin
ms.service: virtual-machines
ms.subservice: disks
ms.collection: linux
ms.topic: quickstart
ms.date: 05/17/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 1833b25554df53a480af9a8fc643755dbc2bbdb8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102564685"
---
# <a name="quickstart-create-and-encrypt-a-linux-vm-in-azure-with-azure-powershell"></a>Краткое руководство. Создание и шифрование виртуальной машины Linux в Azure с помощью Azure PowerShell

Модуль PowerShell используется для создания ресурсов Azure и управления ими с помощью командной строки PowerShell или сценариев. В этом кратком руководстве показано, как с помощью модуля Azure PowerShell создать виртуальную машину Linux, Key Vault для хранения ключей шифрования и зашифровать виртуальную машину. В этом кратком руководстве используется образ Ubuntu 16.04 LTS marketplace от Canonical и виртуальная машина размера Standard_D2S_V3. 

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов Azure с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими.

```azurepowershell-interactive
New-AzResourceGroup -Name "myResourceGroup" -Location "EastUS"
```

## <a name="create-a-virtual-machine"></a>Создание виртуальной машины

Создайте виртуальную машину Azure с помощью командлета [New-AzVM](/powershell/module/az.compute/new-azvm), передавая ему объект конфигурации виртуальной машины, созданный выше.

```powershell-interactive
$cred = Get-Credential

New-AzVM -Name MyVm -Credential $cred -ResourceGroupName MyResourceGroup -Image Canonical:UbuntuServer:18.04-LTS:latest -Size Standard_D2S_V3
```

Развертывание виртуальной машины может занять несколько минут. 

## <a name="create-a-key-vault-configured-for-encryption-keys"></a>Создание Key Vault, настроенного для ключей шифрования

Шифрование дисков Azure хранит ключи шифрования в Azure Key Vault. Создайте Key Vault с помощью командлета [New-AzKeyVault](/powershell/module/az.keyvault/new-azkeyvault). Чтобы включить в Key Vault возможность хранения ключей шифрования, используйте параметр -EnabledForDiskEncryption.

> [!Important]
> Каждое хранилище ключей должно иметь уникальное имя в Azure. В приведенных ниже примерах замените переменную <your-unique-keyvault-name> выбранным именем.

```azurepowershell-interactive
New-AzKeyvault -name "<your-unique-keyvault-name>" -ResourceGroupName "myResourceGroup" -Location EastUS -EnabledForDiskEncryption
```

## <a name="encrypt-the-virtual-machine"></a>Шифрование виртуальной машины

Зашифруйте виртуальную машину, выполнив командлет [Set-AzVmDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension). 

Set-AzVmDiskEncryptionExtension требует некоторых значений из объекта Key Vault. Их можно получить, передав уникальное имя хранилища ключей в [Get-AzKeyvault](/powershell/module/az.keyvault/get-azkeyvault).

```azurepowershell-interactive
$KeyVault = Get-AzKeyVault -VaultName "<your-unique-keyvault-name>" -ResourceGroupName "MyResourceGroup"

Set-AzVMDiskEncryptionExtension -ResourceGroupName MyResourceGroup -VMName "MyVM" -DiskEncryptionKeyVaultUrl $KeyVault.VaultUri -DiskEncryptionKeyVaultId $KeyVault.ResourceId -SkipVmBackup -VolumeType All
```

Через несколько минут вы получите такой результат:

```
RequestId IsSuccessStatusCode StatusCode ReasonPhrase
--------- ------------------- ---------- ------------
                         True         OK OK
```

Вы можете проверить процесс шифрования, выполнив командлет [Get-AzVmDiskEncryptionStatus](/powershell/module/az.compute/Get-AzVMDiskEncryptionStatus).

```azurepowershell-interactive
Get-AzVmDiskEncryptionStatus -VMName MyVM -ResourceGroupName MyResourceGroup
```

Если шифрование включено, в полученных выходных данных вы увидите следующий код:

```
OsVolumeEncrypted          : EncryptionInProgress
DataVolumesEncrypted       : NotMounted
OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
ProgressMessage            : OS disk encryption started
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ненужную группу ресурсов, виртуальную машину и все связанные с ней ресурсы, выполнив командлет [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup).

```azurepowershell-interactive
Remove-AzResourceGroup -Name "myResourceGroup"
```

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого краткого руководства вы создали виртуальную машину, включенный для ключей шифрования Key Vault и зашифровали виртуальную машину.  Перейдите к следующей статье, чтобы узнать больше о шифровании дисков Azure для виртуальных машин Linux.

> [!div class="nextstepaction"]
> [Общие сведения о шифровании дисков Azure](disk-encryption-overview.md)
