---
title: Шифрование дисков для масштабируемых наборов Azure с помощью Azure PowerShell
description: Узнайте, как использовать Azure PowerShell для шифрования экземпляров виртуальной машины и подключенных к ним дисков в масштабируемом наборе виртуальных машин Windows.
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: disks
ms.date: 10/15/2019
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurepowershell
ms.openlocfilehash: 59b015232fcfe2fd98e1b9f28c2eb4fa86606049
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102519577"
---
# <a name="encrypt-os-and-attached-data-disks-in-a-virtual-machine-scale-set-with-azure-powershell"></a>Зашифруйте ОС и подключенные диски данных в масштабируемом наборе виртуальных машин с помощью Azure PowerShell

Модуль PowerShell используется для создания ресурсов Azure и управления ими с помощью командной строки PowerShell или сценариев.  В этой статье показано, как использовать Azure PowerShell для создания и шифрования масштабируемого набора виртуальных машин. Дополнительные сведения о применении шифрования дисков Azure к масштабируемому набору виртуальных машин см. в статье [Шифрование дисков Azure для масштабируемых наборов виртуальных машин](disk-encryption-overview.md).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-an-azure-key-vault-enabled-for-disk-encryption"></a>Создание Azure Key Vault с поддержкой шифрования дисков

В хранилище ключей Azure можно хранить ключи, секреты или пароли, что позволяет безопасно реализовать их в приложениях и службах. Криптографические ключи хранятся в хранилище ключей Azure с применением защиты программного обеспечения. В качестве альтернативы можно импортировать или создать ключи аппаратных модулей безопасности (HSM), сертифицированных по стандартам уровня 2 FIPS 140-2. Эти криптографические ключи используются для шифрования и расшифровки виртуальных дисков, подключенных к виртуальной машине. Вы сохраняете контроль над этими криптографическими ключами и можете проводить аудит их использования.

Создайте Key Vault с помощью [New-азкэйваулт](/powershell/module/az.keyvault/new-azkeyvault). Чтобы этот Key Vault можно было использовать для шифрования диска, укажите параметр *EnabledForDiskEncryption*. Следующий пример определяет переменные для имени группы ресурсов, имени и расположения Key Vault. Укажите уникальное имя для Key Vault:

```azurepowershell-interactive
$rgName="myResourceGroup"
$vaultName="myuniquekeyvault"
$location = "EastUS"

New-AzResourceGroup -Name $rgName -Location $location
New-AzKeyVault -VaultName $vaultName -ResourceGroupName $rgName -Location $location -EnabledForDiskEncryption
```

### <a name="use-an-existing-key-vault"></a>Использование существующего Key Vault

Этот шаг выполняется только в том случае, если у вас уже есть хранилище ключей и вы хотите применить его для шифрования дисков. Пропустите этот шаг, если в предыдущем разделе вы создали новый Key Vault.

Чтобы использовать для шифрования дисков существующее хранилище Key Vault, расположенное в той же подписке и том же регионе, что и масштабируемый набор, выполните командлет [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/Set-AzKeyVaultAccessPolicy). Укажите имя существующего хранилища Key Vault в переменной *$vaultName*, вот так:


```azurepowershell-interactive
$vaultName="myexistingkeyvault"
Set-AzKeyVaultAccessPolicy -VaultName $vaultName -EnabledForDiskEncryption
```

## <a name="create-a-scale-set"></a>Создание масштабируемого набора

Первым делом настройте имя и пароль администратора для экземпляров виртуальных машин с помощью командлета [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential):

```azurepowershell-interactive
$cred = Get-Credential
```

Теперь создайте масштабируемый набор виртуальных машин с помощью командлета [New-AzVmss](/powershell/module/az.compute/new-azvmss). Чтобы распределить трафик между отдельными экземплярами виртуальных машин, создается еще и подсистема балансировки нагрузки. Подсистема балансировки нагрузки определяет правила передачи трафика на TCP-порт 80, а также разрешает подключение удаленного рабочего стола трафик через TCP-порт 3389 и удаленное взаимодействие PowerShell через TCP-порт 5985:

```azurepowershell-interactive
$vmssName="myScaleSet"

New-AzVmss `
    -ResourceGroupName $rgName `
    -VMScaleSetName $vmssName `
    -Location $location `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -PublicIpAddressName "myPublicIPAddress" `
    -LoadBalancerName "myLoadBalancer" `
    -UpgradePolicy "Automatic" `
    -Credential $cred
```

## <a name="enable-encryption"></a>Включение шифрования

Чтобы зашифровать экземпляры виртуальных машин в масштабируемом наборе, сначала нужно получить некоторые сведения об URI и идентификаторе ресурса для Key Vault с помощью командлета [Get-AzKeyVault](/powershell/module/az.keyvault/Get-AzKeyVault). Применяются эти переменные, после чего запускается процесс шифрования с помощью [Set-AzVmssDiskEncryptionExtension](/powershell/module/az.compute/Set-AzVmssDiskEncryptionExtension).


```azurepowershell-interactive
$diskEncryptionKeyVaultUrl=(Get-AzKeyVault -ResourceGroupName $rgName -Name $vaultName).VaultUri
$keyVaultResourceId=(Get-AzKeyVault -ResourceGroupName $rgName -Name $vaultName).ResourceId

Set-AzVmssDiskEncryptionExtension -ResourceGroupName $rgName -VMScaleSetName $vmssName `
    -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $keyVaultResourceId -VolumeType "All"
```

В ответ на запрос введите *y*, чтобы подтвердить продолжение процесса шифрования диска для экземпляров масштабируемого набора виртуальных машин.

### <a name="enable-encryption-using-kek-to-wrap-the-key"></a>Включение шифрования с использованием ключа шифрования ключа для переноса ключа

Ключ шифрования ключа (KEK) можно также использовать для повышения безопасности при шифровании масштабируемого набора виртуальных машин.

```azurepowershell-interactive
$diskEncryptionKeyVaultUrl=(Get-AzKeyVault -ResourceGroupName $rgName -Name $vaultName).VaultUri
$keyVaultResourceId=(Get-AzKeyVault -ResourceGroupName $rgName -Name $vaultName).ResourceId
$keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $vaultName -Name $keyEncryptionKeyName).Key.kid;

Set-AzVmssDiskEncryptionExtension -ResourceGroupName $rgName -VMScaleSetName $vmssName `
    -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $keyVaultResourceId `
    -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $keyVaultResourceId -VolumeType "All"
```

> [!NOTE]
>  Синтаксис значения параметра disk-encryption-keyvault — это строка полного идентификатора:</br>
/subscriptions/[guid_подписки]/resourceGroups/[имя_группы_ресурсов]/providers/Microsoft.KeyVault/vaults/[имя_хранилища_ключей]</br></br>
> Синтаксис значения параметра key-encryption-key — это полный универсальный код ресурса (URI) для KEK:</br>
https://[имя_хранилища_ключей].vault.azure.net/keys/[имя_kek]/[уникальный_идентификатор_kek]

## <a name="check-encryption-progress"></a>Проверка хода выполнения шифрования

Чтобы проверить состояние шифрования диска, используйте [Get-AzVmssDiskEncryption](/powershell/module/az.compute/Get-AzVmssDiskEncryption):


```azurepowershell-interactive
Get-AzVmssDiskEncryption -ResourceGroupName $rgName -VMScaleSetName $vmssName
```

Если экземпляры виртуальных машин зашифрованы, код *EncryptionSummary* возвращает состояние *ProvisioningState/succeeded*, как показано в следующем примере выходных данных:

```powershell
ResourceGroupName            : myResourceGroup
VmScaleSetName               : myScaleSet
EncryptionSettings           :
  KeyVaultURL                : https://myuniquekeyvault.vault.azure.net/
  KeyEncryptionKeyURL        :
  KeyVaultResourceId         : /subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.KeyVault/vaults/myuniquekeyvault
  KekVaultResourceId         :
  KeyEncryptionAlgorithm     :
  VolumeType                 : All
  EncryptionOperation        : EnableEncryption
EncryptionSummary[0]         :
  Code                       : ProvisioningState/succeeded
  Count                      : 2
EncryptionEnabled            : True
EncryptionExtensionInstalled : True
```

## <a name="disable-encryption"></a>Отключение шифрования

Если вы решите отказаться от шифрования дисков для экземпляров виртуальных машин, вы можете отключить его с помощью командлета [Disable-AzVmssDiskEncryption](/powershell/module/az.compute/Disable-AzVmssDiskEncryption) следующим образом:


```azurepowershell-interactive
Disable-AzVmssDiskEncryption -ResourceGroupName $rgName -VMScaleSetName $vmssName
```

## <a name="next-steps"></a>Дальнейшие действия

- В этой статье вы зашифровали масштабируемый набор виртуальных машин с помощью Azure PowerShell. Можно также использовать шаблоны [Azure CLI](disk-encryption-cli.md) или [Azure Resource Manager](disk-encryption-azure-resource-manager.md).
- Если вы хотите, чтобы шифрование дисков Azure применялось после подготовки другого расширения, можно использовать [последовательность расширения](virtual-machine-scale-sets-extension-sequencing.md).
