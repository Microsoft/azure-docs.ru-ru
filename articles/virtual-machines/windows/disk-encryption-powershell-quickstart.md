---
title: Создание и шифрование виртуальной машины Windows с помощью Azure PowerShell
description: Из этого краткого руководства вы узнаете, как с помощью Azure PowerShell создать и зашифровать виртуальную машину Windows.
author: msmbaldwin
ms.author: mbaldwin
ms.service: virtual-machines
ms.subservice: disks
ms.collection: windows
ms.topic: quickstart
ms.date: 05/17/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 4aed2ce182e535ebb60eae0007353c9c7bddef78
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102555267"
---
# <a name="quickstart-create-and-encrypt-a-windows-virtual-machine-in-azure-with-powershell"></a>Краткое руководство. Создание и шифрование виртуальной машины Windows в Azure с помощью PowerShell

Модуль PowerShell используется для создания ресурсов Azure и управления ими с помощью командной строки PowerShell или сценариев. В этом кратком руководстве показано, как с помощью модуля Azure PowerShell создать виртуальную машину Windows, Key Vault для хранения ключей шифрования и зашифровать виртуальную машину. 

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.


## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов Azure с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими.

```powershell
New-AzResourceGroup -Name "myResourceGroup" -Location "EastUS"
```

## <a name="create-a-virtual-machine"></a>Создание виртуальной машины

Создайте виртуальную машину Azure с помощью команды [New-AzVM](/powershell/module/az.compute/new-azvm). Необходимо указать учетные данные в командлете. 

```powershell
$cred = Get-Credential 

New-AzVM -Name MyVm -Credential $cred -ResourceGroupName MyResourceGroup -Image win2016datacenter -Size Standard_D2S_V3
```

Развертывание виртуальной машины может занять несколько минут. 

## <a name="create-a-key-vault-configured-for-encryption-keys"></a>Создание Key Vault, настроенного для ключей шифрования

Шифрование дисков Azure хранит ключи шифрования в Azure Key Vault. Создайте Key Vault с помощью командлета [New-AzKeyVault](/powershell/module/az.keyvault/new-azkeyvault). Чтобы включить в Key Vault возможность хранения ключей шифрования, используйте параметр -EnabledForDiskEncryption.

> [!Important]
> Каждый Key Vault должен иметь уникальное имя. В следующем примере показан процесс создания Key Vault с именем *myKV*. Назовите свое хранилище ключей иначе.

```powershell
New-AzKeyvault -name MyKV -ResourceGroupName myResourceGroup -Location EastUS -EnabledForDiskEncryption
```

## <a name="encrypt-the-virtual-machine"></a>Шифрование виртуальной машины

Зашифруйте виртуальную машину, выполнив командлет [Set-AzVmDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension). 

Set-AzVmDiskEncryptionExtension требует некоторых значений из объекта Key Vault. Их можно получить, передав уникальное имя хранилища ключей в [Get-AzKeyvault](/powershell/module/az.keyvault/get-azkeyvault).

```powershell
$KeyVault = Get-AzKeyVault -VaultName MyKV -ResourceGroupName MyResourceGroup

Set-AzVMDiskEncryptionExtension -ResourceGroupName MyResourceGroup -VMName MyVM -DiskEncryptionKeyVaultUrl $KeyVault.VaultUri -DiskEncryptionKeyVaultId $KeyVault.ResourceId
```

Через несколько минут вы получите такой результат:

```
RequestId IsSuccessStatusCode StatusCode ReasonPhrase
--------- ------------------- ---------- ------------
                         True         OK OK
```

Вы можете проверить процесс шифрования, выполнив командлет [Get-AzVmDiskEncryptionStatus](/powershell/module/az.compute/Get-AzVMDiskEncryptionStatus).

```powershell
Get-AzVmDiskEncryptionStatus -VMName MyVM -ResourceGroupName MyResourceGroup
```

Если шифрование включено, в полученных выходных данных вы увидите следующий код:

```
OsVolumeEncrypted          : Encrypted
DataVolumesEncrypted       : NoDiskFound
OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
ProgressMessage            : Provisioning succeeded
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ненужную группу ресурсов, виртуальную машину и все связанные с ней ресурсы, выполнив командлет [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup).

```powershell
Remove-AzResourceGroup -Name "myResourceGroup"
```

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого краткого руководства вы создали виртуальную машину, включенный для ключей шифрования Key Vault и зашифровали виртуальную машину.  Перейдите к следующей статье, чтобы узнать больше о предварительных требованиях для шифрования дисков Azure для виртуальных машин IaaS.

> [!div class="nextstepaction"]
> [Общие сведения о шифровании дисков Azure](disk-encryption-overview.md)
