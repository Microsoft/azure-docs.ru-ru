---
title: Предварительный просмотр — создание версии образа, зашифрованной с помощью собственных ключей
description: Создание версии образа в общей коллекции образов с помощью управляемых клиентом ключей шифрования.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 11/3/2020
ms.author: cynthn
ms.openlocfilehash: f8ea2224020e6fa6c5dae135dc575cc77510efd5
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102123944"
---
# <a name="preview-use-customer-managed-keys-for-encrypting-images"></a>Предварительный просмотр: Использование управляемых клиентом ключей для шифрования образов

Образы в общей коллекции образов хранятся в виде моментальных снимков, поэтому они автоматически шифруются при шифровании на стороне сервера. Шифрование на стороне сервера использует 256-разрядное [шифрование AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard), один из наиболее подходящего блочного шифра. Шифрование на стороне сервера также совместимо с FIPS 140-2. Дополнительные сведения о криптографических модулях, лежащих в основе управляемых дисков Azure, см. в разделе [API шифрования: следующее поколение](/windows/desktop/seccng/cng-portal).

Вы можете использовать ключи, управляемые платформой, для шифрования образов или собственные ключи. Кроме того, для двойного шифрования можно использовать и совместно. Если вы решили управлять шифрованием с помощью собственных ключей, вы можете указать *управляемый клиентом ключ*, который будет использоваться для шифрования и расшифровки всех дисков в образах. 

Шифрование на стороне сервера с помощью управляемых клиентом ключей использует Azure Key Vault. Вы можете импортировать [ключи RSA](../key-vault/keys/hsm-protected-keys.md) в хранилище ключей или создать новые ключи rsa в Azure Key Vault.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этой статьей необходимо, чтобы в каждом регионе, где вы хотите реплицировать образ, был задан набор шифрования дисков:

- Сведения об использовании только управляемого клиентом ключа см. в статьях о включении управляемых клиентом ключей с шифрованием на стороне сервера с помощью [портал Azure](./disks-enable-customer-managed-keys-portal.md) или [PowerShell](./windows/disks-enable-customer-managed-keys-powershell.md#set-up-an-azure-key-vault-and-diskencryptionset-without-automatic-key-rotation).

- Чтобы использовать управляемые платформой и управляемые клиентом ключи (для двойного шифрования), см. статьи о включении двойного шифрования при хранении с помощью [портал Azure](./disks-enable-double-encryption-at-rest-portal.md) или [PowerShell](./windows/disks-enable-double-encryption-at-rest-powershell.md).

   > [!IMPORTANT]
   > [https://aka.ms/diskencryptionupdates](https://aka.ms/diskencryptionupdates)Для доступа к портал Azure необходимо использовать ссылку. Двойное шифрование неактивных в настоящее время не отображается в общедоступной портал Azure, если не используется эта ссылка.

## <a name="limitations"></a>Ограничения

Если вы используете ключи, управляемые клиентом, для шифрования образов в общей коллекции образов, применяются следующие ограничения.   

- Наборы ключей шифрования должны находиться в той же подписке, что и ваш образ.

- Наборы ключей шифрования являются региональными ресурсами, поэтому для каждого региона требуется другой набор ключей шифрования.

- Нельзя копировать или совместно использовать образы, использующие управляемые клиентом ключи. 

- После того как вы использовали собственные ключи для шифрования диска или образа, вы не сможете вернуться к использованию ключей, управляемых платформой, для шифрования этих дисков или образов.


> [!IMPORTANT]
> Шифрование через управляемые клиентом ключи в настоящее время находится в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендуется для рабочих нагрузок в рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


## <a name="powershell"></a>PowerShell

Для общедоступной предварительной версии сначала необходимо зарегистрировать функцию:

```azurepowershell-interactive
Register-AzProviderFeature -FeatureName SIGEncryption -ProviderNamespace Microsoft.Compute
```

Для завершения регистрации потребуется несколько минут. Используйте `Get-AzProviderFeature` для проверки состояния регистрации компонента:

```azurepowershell-interactive
Get-AzProviderFeature -FeatureName SIGEncryption -ProviderNamespace Microsoft.Compute
```

При `RegistrationState` возврате `Registered` можно перейти к следующему шагу.

Проверьте регистрацию поставщика. Убедитесь, что она возвращает значение `Registered`.

```azurepowershell-interactive
Get-AzResourceProvider -ProviderNamespace Microsoft.Compute | Format-table -Property ResourceTypes,RegistrationState
```

Если они не возвращаются `Registered` , используйте следующий код для регистрации поставщиков:

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
```

Чтобы указать набор шифрования диска для версии образа, используйте командлет  [New-азгаллеримажедефинитион](/powershell/module/az.compute/new-azgalleryimageversion) с `-TargetRegion` параметром: 

```azurepowershell-interactive

$sourceId = <ID of the image version source>

$osDiskImageEncryption = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myDESet'}

$dataDiskImageEncryption1 = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myDESet1';Lun=1}

$dataDiskImageEncryption2 = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myDESet2';Lun=2}

$dataDiskImageEncryptions = @($dataDiskImageEncryption1,$dataDiskImageEncryption2)

$encryption1 = @{OSDiskImage=$osDiskImageEncryption;DataDiskImages=$dataDiskImageEncryptions}

$region1 = @{Name='West US';ReplicaCount=1;StorageAccountType=Standard_LRS;Encryption=$encryption1}

$eastUS2osDiskImageEncryption = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myEastUS2DESet'}

$eastUS2dataDiskImageEncryption1 = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myEastUS2DESet1';Lun=1}

$eastUS2dataDiskImageEncryption2 = @{DiskEncryptionSetId='subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Compute/diskEncryptionSets/myEastUS2DESet2';Lun=2}

$eastUS2DataDiskImageEncryptions = @($eastUS2dataDiskImageEncryption1,$eastUS2dataDiskImageEncryption2)

$encryption2 = @{OSDiskImage=$eastUS2osDiskImageEncryption;DataDiskImages=$eastUS2DataDiskImageEncryptions}

$region2 = @{Name='East US 2';ReplicaCount=1;StorageAccountType=Standard_LRS;Encryption=$encryption2}

$targetRegion = @($region1, $region2)


# Create the image
New-AzGalleryImageVersion `
   -ResourceGroupName $rgname `
   -GalleryName $galleryName `
   -GalleryImageDefinitionName $imageDefinitionName `
   -Name $versionName -Location $location `
   -SourceImageId $sourceId `
   -ReplicaCount 2 `
   -StorageAccountType Standard_LRS `
   -PublishingProfileEndOfLifeDate '2020-12-01' `
   -TargetRegion $targetRegion
```

### <a name="create-a-vm"></a>Создание виртуальной машины

Вы можете создать виртуальную машину из общей коллекции образов и использовать управляемые клиентом ключи для шифрования дисков. Синтаксис такой же, как и создание [обобщенной](vm-generalized-image-version-powershell.md) или [специализированной](vm-specialized-image-version-powershell.md) виртуальной машины из образа. Используйте набор расширенных параметров и добавьте `Set-AzVMOSDisk -Name $($vmName +"_OSDisk") -DiskEncryptionSetId $diskEncryptionSet.Id -CreateOption FromImage` в конфигурацию виртуальной машины.

Для дисков данных добавьте `-DiskEncryptionSetId $setID` параметр при использовании командлета [Add-азвмдатадиск](/powershell/module/az.compute/add-azvmdatadisk).


## <a name="cli"></a>CLI 

Для общедоступной предварительной версии сначала необходимо зарегистрироваться для получения функции. Регистрация занимает около 30 минут.

```azurecli-interactive
az feature register --namespace Microsoft.Compute --name SIGEncryption
```

Проверьте состояние регистрации компонента:

```azurecli-interactive
az feature show --namespace Microsoft.Compute --name SIGEncryption | grep state
```

После возврата этого кода `"state": "Registered"` можно перейти к следующему шагу.

Проверьте регистрацию:

```azurecli-interactive
az provider show -n Microsoft.Compute | grep registrationState
```

Если он не зарегистрирован, выполните следующую команду:

```azurecli-interactive
az provider register -n Microsoft.Compute
```


Чтобы указать набор шифрования диска для версии образа, используйте команду [AZ Image Gallery Create-Image-Version](/cli/azure/sig/image-version#az-sig-image-version-create) с `--target-region-encryption` параметром. Формат `--target-region-encryption` представляет собой разделенный запятыми список ключей для шифрования дисков операционной системы и данных. Результат будет выглядеть так: `<encryption set for the OS disk>,<Lun number of the data disk>,<encryption set for the data disk>,<Lun number for the second data disk>,<encryption set for the second data disk>`. 

Если источником для диска ОС является управляемый диск или виртуальная машина, используйте `--managed-image`, чтобы указать источник для версии образа. В этом примере источником является управляемый образ с диском ОС и диском данных в LUN 0. Диск ОС будет зашифрован с помощью DiskEncryptionSet1, а диск данных будет зашифрован с помощью DiskEncryptionSet2.

```azurecli-interactive
az sig image-version create \
   -g MyResourceGroup \
   --gallery-image-version 1.0.0 \
   --location westus \
   --target-regions westus=2=standard_lrs eastus2 \
   --target-region-encryption WestUSDiskEncryptionSet1,0,WestUSDiskEncryptionSet2 EastUS2DiskEncryptionSet1,0,EastUS2DiskEncryptionSet2 \
   --gallery-name MyGallery \
   --gallery-image-definition MyImage \
   --managed-image "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/images/myImage"
```

Если источником для диска ОС является моментальный снимок, используйте `--os-snapshot`, чтобы указать диск ОС. Если имеются моментальные снимки диска данных, которые также должны быть частью версии образа, добавьте их. Используйте `--data-snapshot-luns` для указания LUN и используйте `--data-snapshots` для указания моментальных снимков.

В этом примере источниками являются моментальные снимки диска. В LUN 0 есть диск операционной системы и диск данных. Диск ОС будет зашифрован с помощью DiskEncryptionSet1, а диск данных будет зашифрован с помощью DiskEncryptionSet2.

```azurecli-interactive
az sig image-version create \
   -g MyResourceGroup \
   --gallery-image-version 1.0.0 \
   --location westus\
   --target-regions westus=2=standard_lrs eastus\
   --target-region-encryption WestUSDiskEncryptionSet1,0,WestUSDiskEncryptionSet2 EastUS2DiskEncryptionSet1,0,EastUS2DiskEncryptionSet2 \
   --os-snapshot "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/snapshots/myOSSnapshot" \
   --data-snapshot-luns 0 \
   --data-snapshots "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/snapshots/myDDSnapshot" \
   --gallery-name MyGallery \
   --gallery-image-definition MyImage 
   
```

### <a name="create-the-vm"></a>Создание виртуальной машины

Вы можете создать виртуальную машину на основе общей коллекции образов и использовать управляемые клиентом ключи для шифрования дисков. Синтаксис такой же, как и создание [обобщенной](vm-generalized-image-version-cli.md) или [специализированной](vm-specialized-image-version-cli.md) виртуальной машины из образа. Просто добавьте `--os-disk-encryption-set` параметр с идентификатором набора шифрования. Для дисков данных добавьте `--data-disk-encryption-sets` разделенный пробелами список наборов шифрования дисков для дисков данных.


## <a name="portal"></a>Портал

При создании версии образа на портале можно использовать вкладку **Шифрование** для применения наборов шифрования хранилища.

> [!IMPORTANT]
> Чтобы использовать двойное шифрование, необходимо использовать ссылку [https://aka.ms/diskencryptionupdates](https://aka.ms/diskencryptionupdates) для доступа к портал Azure. Двойное шифрование неактивных в настоящее время не отображается в общедоступной портал Azure, если не используется эта ссылка.


1. На странице **Создание версии образа** выберите вкладку **Шифрование** .
2. В разделе **тип шифрования** выберите **Шифрование неактивных с помощью управляемого клиентом ключа** или **двойное шифрование с помощью управляемых платформой ключей и управляемых клиентом ключами**. 
3. Для каждого диска в образе выберите набор шифрования из раскрывающегося списка **набор шифрования диска** . 

### <a name="create-the-vm"></a>Создание виртуальной машины

Вы можете создать виртуальную машину из версии образа и использовать управляемые клиентом ключи для шифрования дисков. При создании виртуальной машины на портале на вкладке **диски** выберите **Шифрование неактивных с помощью управляемых клиентом ключей** или **двойное шифрование с управляемыми платформой ключами** для **типа шифрования**. Затем можно выбрать набор шифрования из раскрывающегося списка.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о [шифровании дисков на стороне сервера](./disk-encryption.md).

Сведения о том, как предоставить сведения о плане покупки, см. в разделе [предоставление сведений о плане покупки Azure Marketplace при создании образов](marketplace-images.md).