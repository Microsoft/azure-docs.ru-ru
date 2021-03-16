---
title: Сценарии шифрования дисков Azure для виртуальных машин Windows
description: В этой статье приведены инструкции по включению Microsoft Azure шифрования дисков для виртуальных машин Windows в различных сценариях.
author: msmbaldwin
ms.service: virtual-machines
ms.subservice: disks
ms.collection: windows
ms.topic: how-to
ms.author: mbaldwin
ms.date: 08/06/2019
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: a01d5f48ca3b10f4c49ee621398ae87392dc34a6
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103493464"
---
# <a name="azure-disk-encryption-scenarios-on-windows-vms"></a>Сценарии шифрования дисков Azure для виртуальных машин Windows

Шифрование дисков Azure для виртуальных машин Windows (VM) использует функцию BitLocker Windows для обеспечения полного шифрования диска операционной системы и диска данных. Кроме того, если параметр VolumeType (Тип тома) имеет значение All (Все), временные диски также шифруются.

Шифрование дисков Azure [интегрируется с Azure Key Vault](disk-encryption-key-vault.md), помогая управлять ключами шифрования дисков и секретами. Общие сведения о службе см. в статье [Шифрование дисков Azure для виртуальных машин Windows](disk-encryption-overview.md).

Шифрование дисков можно применить только к виртуальным машинам [поддерживаемых размеров и операционных систем](disk-encryption-overview.md#supported-vms-and-operating-systems). Также необходимы следующие элементы:

- [Требования к сети](disk-encryption-overview.md#networking-requirements)
- [Требования к групповая политика](disk-encryption-overview.md#group-policy-requirements)
- [Требования к хранилищу ключей шифрования](disk-encryption-overview.md#encryption-key-storage-requirements)

>[!IMPORTANT]
> - Если вы уже использовали шифрование дисков Azure c Azure AD для шифрования виртуальной машины, сохраняйте и далее этот способ шифрования виртуальной машины. См. детали в разделе [Шифрование дисков Azure с использованием приложения Azure AD (предыдущий выпуск)](disk-encryption-overview-aad.md). 
>
> - Перед шифрованием дисков следует создать [моментальный снимок](snapshot-copy-managed-disk.md) и (или) выполнить резервное копирование. Резервные копии обеспечивают возможность восстановления в случае любого непредвиденного сбоя во время шифрования. Для виртуальных машин с управляемыми дисками необходимо создать резервную копию до начала шифрования. После создания резервной копии можно зашифровать управляемые диски с помощью [командлета Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) с параметром -skipVmBackup. Дополнительные сведения о резервном копировании и восстановлении зашифрованных виртуальных машин см. в статье [резервное копирование и восстановление зашифрованной виртуальной машины Azure](../../backup/backup-azure-vms-encryption.md). 
>
> - Шифрование или отключение шифрования может привести к перезагрузке виртуальной машины.

## <a name="install-tools-and-connect-to-azure"></a>Установка средств и подключение к Azure

[!INCLUDE [disk-encryption-install-cli-powershell](../../../includes/disk-encryption-install-cli-powershell.md)]

## <a name="enable-encryption-on-an-existing-or-running-windows-vm"></a>Включение шифрования для существующей или работающей виртуальной машины Windows
В этом сценарии шифрование можно включить с помощью шаблона Resource Manager, командлетов PowerShell или команд интерфейса командной строки. Сведения о схеме для расширения виртуальной машины см. в статье [Шифрование дисков Azure для Windows ](../extensions/azure-disk-enc-windows.md).

### <a name="enable-encryption-on-existing-or-running-vms-with-azure-powershell"></a> Включение шифрования на существующих или работающих виртуальных машинах с помощью Azure PowerShell 
Используйте командлет [Set-азвмдискенкриптионекстенсион](/powershell/module/az.compute/set-azvmdiskencryptionextension) , чтобы включить шифрование на работающей виртуальной машине IaaS в Azure. 

-  **Шифрование работающей виртуальной машины:** Приведенный ниже сценарий позволяет инициализировать переменные и запустить командлет Set-AzVMDiskEncryptionExtension. Группа ресурсов, виртуальная машина и хранилище ключей должны быть уже созданы в качестве необходимых компонентов. Замените значения Микэйваултресаурцеграуп, Мивиртуалмачинересаурцеграуп, Мисекуревм и Мисекуреваулт значениями.

     ```azurepowershell
      $KVRGname = 'MyKeyVaultResourceGroup';
      $VMRGName = 'MyVirtualMachineResourceGroup';
      $vmName = 'MySecureVM';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId;
    ```
- **Шифрование работающей виртуальной машины с использованием KEK:** 

     ```azurepowershell
     $KVRGname = 'MyKeyVaultResourceGroup';
     $VMRGName = 'MyVirtualMachineResourceGroup';
     $vmName = 'MyExtraSecureVM';
     $KeyVaultName = 'MySecureVault';
     $keyEncryptionKeyName = 'MyKeyEncryptionKey';
     $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
     $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
     $keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;

     Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId;

     ```
     
   >[!NOTE]
   > Синтаксис значения параметра disk-encryption-keyvault — это полная строка идентификатора: /subscriptions/[ИД или GUID подписки]/resourceGroups/[имя группы ресурсов]/providers/Microsoft.KeyVault/vaults/[имя хранилища ключей].</br> Синтаксис значения параметра key-encryption-key —это полный универсальный код ресурса (URI) для ключа шифрования ключей: https://[имя хранилища ключей].vault.azure.net/keys/[имя KEK]/[ИД KEK]. 

- **Убедитесь, что диски зашифрованы:** Чтобы проверить состояние шифрования виртуальной машины IaaS, используйте командлет [Get-азвмдискенкриптионстатус](/powershell/module/az.compute/get-azvmdiskencryptionstatus) . 
     ```azurepowershell-interactive
     Get-AzVmDiskEncryptionStatus -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
     ```
    
- **Отключение шифрования дисков:** Для отключения шифрования используйте командлет [Disable-AzVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption). Отключение шифрования диска данных на виртуальной машине Windows, когда оба диска (диск ОС и диск данных)зашифрованы и работают не так, как ожидалось. В этом случае отключите шифрование на всех дисках.

     ```azurepowershell-interactive
     Disable-AzVMDiskEncryption -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
     ```

### <a name="enable-encryption-on-existing-or-running-vms-with-the-azure-cli"></a>Включение шифрования на существующих или выполняющихся виртуальных машинах с Azure CLI
Используйте команду [az vm encryption enable](/cli/azure/vm/encryption#az-vm-encryption-enable), чтобы включить шифрование на работающей виртуальной машине IaaS в Azure.

- **Шифрование работающей виртуальной машины:**

    ```azurecli-interactive
    az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault "MySecureVault" --volume-type [All|OS|Data]
    ```

- **Шифрование работающей виртуальной машины с использованием KEK:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type [All|OS|Data]
     ```

     >[!NOTE]
     > Синтаксис значения параметра disk-encryption-keyvault — это полная строка идентификатора: /subscriptions/[ИД или GUID подписки]/resourceGroups/[имя группы ресурсов]/providers/Microsoft.KeyVault/vaults/[имя хранилища ключей]. </br> Синтаксис значения параметра key-encryption-key —это полный универсальный код ресурса (URI) для ключа шифрования ключей: https://[имя хранилища ключей].vault.azure.net/keys/[имя KEK]/[ИД KEK]. 

- **Убедитесь, что диски зашифрованы:** Чтобы проверить состояние шифрования виртуальной машины IaaS, используйте команду [AZ VM Encryption демонстрация](/cli/azure/vm/encryption#az-vm-encryption-show) . 

     ```azurecli-interactive
     az vm encryption show --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup"
     ```

- **Отключение шифрования:** чтобы отключить шифрование, используйте команду [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable). Отключение шифрования диска данных на виртуальной машине Windows, когда оба диска (диск ОС и диск данных)зашифрованы и работают не так, как ожидалось. В этом случае отключите шифрование на всех дисках.

     ```azurecli-interactive
     az vm encryption disable --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup" --volume-type [ALL, DATA, OS]
     ```

### <a name="using-the-resource-manager-template"></a>Использование шаблона Resource Manager

Включить шифрование дисков на существующих или работающих виртуальных машинах IaaS под управлением Windows в Azure можно с помощью [этого шаблона Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-windows-vm-without-aad).


1. На странице шаблона быстрого запуска Azure нажмите кнопку **Deploy to Azure** (Развернуть в Azure).

2. Выберите подписку, группу ресурсов, расположение, параметры, условия использования и соглашение. Нажмите кнопку **Покупка**, чтобы включить шифрование на существующей или работающей виртуальной машине IaaS.

В следующей таблице перечислены параметры шаблона Resource Manager для имеющихся или работающих виртуальных машин.

| Параметр | Описание |
| --- | --- |
| vmName | Имя виртуальной машины для выполнения операции шифрования. |
| keyVaultName | Имя хранилища ключей, в которое будет передан ключ BitLocker. Его можно получить с помощью командлета `(Get-AzKeyVault -ResourceGroupName <MyKeyVaultResourceGroupName>). Vaultname` или команды Azure CLI `az keyvault list --resource-group "MyKeyVaultResourceGroup"`.|
| keyVaultResourceGroup | Имя группы ресурсов, содержащей хранилище ключей.|
|  keyEncryptionKeyURL | URL-адрес ключа шифрования ключа в формате https:// &lt; keyvault-Name &gt; . Vault.Azure.NET/key/ &lt; Key-Name &gt; . Если вы не хотите использовать KEK, оставьте это поле пустым. |
| volumeType | Тип тома, для которого будет выполняться шифрование. Допустимые значения: _OS_, _Data_ и _All_. 
| forceUpdateTag | Передает уникальное значение, такое как GUID, каждый раз, когда операция должна выполняться принудительно. |
| resizeOSDisk | Если размер раздела ОС будет изменен и займет весь виртуальный жесткий диск ОС до разделения системного тома. |
| location | Расположение для всех ресурсов. |

## <a name="enable-encryption-on-nvme-disks-for-lsv2-vms"></a>Включение шифрования на дисках NVMe для виртуальных машин Lsv2

В этом сценарии описывается включение шифрования дисков Azure на дисках NVMe для виртуальных машин серии Lsv2.  Серия Lsv2 включает в себя локальное хранилище NVMe. Локальные диски NVMe являются временными, и данные будут потеряны на этих дисках, если вы останавливаете или освобождаете виртуальную машину (см. раздел [Lsv2-Series](../lsv2-series.md)).

Чтобы включить шифрование на дисках NVMe, выполните следующие действия.

1. Инициализируйте диски NVMe и создайте тома NTFS.
1. Включите шифрование на виртуальной машине, указав для параметра параметра volumetype значение значение ALL. Это позволит включить шифрование для всех дисков ОС и данных, включая тома, которые поддерживаются дисками NVMe. Дополнительные сведения см. [в разделе Включение шифрования на существующей или работающей виртуальной машине Windows](#enable-encryption-on-an-existing-or-running-windows-vm).

Шифрование будет сохранено на дисках NVMe в следующих сценариях:
- Перезагрузка виртуальной машины
- Повторное создание образа масштабируемого набора виртуальных машин
- Замена ОС

Диски NVMe будут деинициализированы в следующих сценариях:

- Запустить виртуальную машину после освобождения
- Восстановление службы
- Резервное копирование

В этих сценариях диски NVMe должны быть инициализированы после запуска виртуальной машины. Чтобы включить шифрование дисков NVMe, выполните команду, чтобы снова включить шифрование дисков Azure после инициализации дисков NVMe.

В дополнение к сценариям, перечисленным в разделе [неподдерживаемые сценарии](#unsupported-scenarios) , шифрование дисков NVMe не поддерживается для:

- Виртуальные машины, зашифрованные с помощью шифрования дисков Azure с помощью AAD (предыдущий выпуск)
- Диски NVMe с дисковыми пространствами
- Azure Site Recovery номеров SKU с дисками NVMe (см [. таблицу поддержки для аварийного восстановления виртуальной машины Azure между регионами Azure: реплицированные машины — хранилище](../../site-recovery/azure-to-azure-support-matrix.md#replicated-machines---storage)).

## <a name="new-iaas-vms-created-from-customer-encrypted-vhd-and-encryption-keys"></a> Новые виртуальные машины IaaS, при создании которых используются зашифрованные виртуальные жесткие диски и ключи шифрования

В этом сценарии можно создать новую виртуальную машину на основе предварительно зашифрованного виртуального жесткого диска и связанных с ним ключей шифрования с помощью командлетов PowerShell или команд интерфейса командной строки. 

Используйте инструкции из статьи [Подготовка предварительно зашифрованного виртуального жесткого диска Windows](disk-encryption-sample-scripts.md#prepare-a-pre-encrypted-windows-vhd). После создания образа выполните действия, описанные в следующем разделе, чтобы создать зашифрованную виртуальную машину Azure.


### <a name="encrypt-vms-with-pre-encrypted-vhds-with-azure-powershell"></a> Шифрование виртуальных машин с предварительно зашифрованными виртуальными жесткими дисками с помощью Azure PowerShell
Можно включить шифрование дисков для зашифрованного виртуального жесткого диска с помощью командлета PowerShell [Set-AzVMOSDisk](/powershell/module/az.compute/set-azvmosdisk#examples). Приведенный ниже пример имеет некоторые общие параметры. 

```azurepowershell
$VirtualMachine = New-AzVMConfig -VMName "MySecureVM" -VMSize "Standard_A1"
$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name "SecureOSDisk" -VhdUri "os.vhd" Caching ReadWrite -Windows -CreateOption "Attach" -DiskEncryptionKeyUrl "https://mytestvault.vault.azure.net/secrets/Test1/514ceb769c984379a7e0230bddaaaaaa" -DiskEncryptionKeyVaultId "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myKVresourcegroup/providers/Microsoft.KeyVault/vaults/mytestvault"
New-AzVM -VM $VirtualMachine -ResourceGroupName "MyVirtualMachineResourceGroup"
```

## <a name="enable-encryption-on-a-newly-added-data-disk"></a>Включение шифрования на добавленном диске данных
[Добавить новый диск на виртуальную машину Windows можно с помощью PowerShell](attach-disk-ps.md) или [портала Azure](attach-managed-disk-portal.md). 

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-powershell"></a>Включение шифрования на добавленном диске данных с помощью Azure PowerShell
 При использовании PowerShell для шифрования нового диска для виртуальных машин Windows необходимо указать новую версию последовательности. Версия последовательности должна быть уникальной. Приведенный ниже сценарий создает GUID для версии последовательности. В некоторых случаях новый добавленный диск данных может быть зашифрован автоматически с помощью расширения шифрования дисков Azure. Автоматическое шифрование обычно происходит, когда виртуальная машина перезапускается после включения нового диска. Обычно это происходит, так как для типа тома указано значение "Все", если ранее на виртуальной машине использовалось шифрование дисков. Если автоматическое шифрование выполняется на вновь добавленном диске данных, рекомендуется снова запустить командлет Set-AzVmDiskEncryptionExtension с новой версией последовательности. Если для нового диска данных включено автоматическое шифрование и вы хотите отказаться от шифрования, сначала расшифруйте все диски, а затем повторите шифрование с новой версией последовательности, указав соответствующую операционную систему для типа тома. 
  
 

-  **Шифрование работающей виртуальной машины:** Приведенный ниже сценарий позволяет инициализировать переменные и запустить командлет Set-AzVMDiskEncryptionExtension. Группа ресурсов, виртуальная машина и хранилище ключей должны быть уже созданы в качестве необходимых компонентов. Замените значения Микэйваултресаурцеграуп, Мивиртуалмачинересаурцеграуп, Мисекуревм и Мисекуреваулт значениями. В этом примере в качестве значения параметра VolumeType используется значение "All", что включает тома операционной системы и тома данных. Если требуется шифрование только томов операционной системы, используйте "OS" в качестве значения параметра VolumeType. 

     ```azurepowershell
      $KVRGname = 'MyKeyVaultResourceGroup';
      $VMRGName = 'MyVirtualMachineResourceGroup';
      $vmName = 'MySecureVM';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;
      $sequenceVersion = [Guid]::NewGuid();

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType "All" –SequenceVersion $sequenceVersion;
    ```
- **Шифрование работающей виртуальной машины с использованием KEK.** В этом примере в качестве значения параметра VolumeType используется значение "All", что включает тома операционной системы и тома данных. Если требуется шифрование только томов операционной системы, используйте "OS" в качестве значения параметра VolumeType.

     ```azurepowershell
     $KVRGname = 'MyKeyVaultResourceGroup';
     $VMRGName = 'MyVirtualMachineResourceGroup';
     $vmName = 'MyExtraSecureVM';
     $KeyVaultName = 'MySecureVault';
     $keyEncryptionKeyName = 'MyKeyEncryptionKey';
     $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
     $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
     $keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;
     $sequenceVersion = [Guid]::NewGuid();

     Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId -VolumeType "All" –SequenceVersion $sequenceVersion;

     ```

    >[!NOTE]
    > Синтаксис значения параметра disk-encryption-keyvault — это полная строка идентификатора: /subscriptions/[ИД или GUID подписки]/resourceGroups/[имя группы ресурсов]/providers/Microsoft.KeyVault/vaults/[имя хранилища ключей].</br> Синтаксис значения параметра key-encryption-key —это полный универсальный код ресурса (URI) для ключа шифрования ключей: https://[имя хранилища ключей].vault.azure.net/keys/[имя KEK]/[ИД KEK]. 

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-cli"></a>Включение шифрования на добавленном диске данных с помощью интерфейса командной строки Azure
 Команда Azure CLI автоматически предоставит вам новую версию последовательности, когда вы запустите команду для включения шифрования. В этом примере используется "All" для параметра volume-type. Может потребоваться изменить значение параметра volume-type на"OS", если необходимо выполнить шифрование только диска операционной системы. В отличие от синтаксиса Powershell, интерфейс командной строки (CLI) не требует предоставления уникальной версии последовательности при включении шифрования. CLI автоматически создает и использует свое собственное уникальное значение версии последовательности.   

-  **Шифрование работающей виртуальной машины:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault "MySecureVault" --volume-type "All"
     ```

- **Шифрование работающей виртуальной машины с использованием KEK:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type "All"
     ```


## <a name="disable-encryption"></a>Отключение шифрования
[!INCLUDE [disk-encryption-disable-encryption-powershell](../../../includes/disk-encryption-disable-powershell.md)]

## <a name="unsupported-scenarios"></a>Неподдерживаемые сценарии

Шифрование дисков Azure не работает в следующих сценариях, функциях и технологиях:

- Шифрование виртуальных машин базового уровня или виртуальных машин, созданных с помощью классического метода создания виртуальной машины.
- Шифрование виртуальных машин, настроенных на основе программных RAID-систем.
- Шифрование виртуальных машин, настроенных с помощью Локальные дисковые пространства (S2D), или версий Windows Server до 2016, настроенных с помощью дисковых пространств Windows.
- Интеграция с локальной системой управления ключами.
- служба файлов Azure (общая файловая система);
- сетевая файловая система (NFS);
- динамические тома;
- Контейнеры Windows Server, которые создают динамические тома для каждого контейнера.
- Временные диски ОС.
- Шифрование общих и распределенных файловых систем, таких как (но не ограничиваясь ими), DFS, GFS, ДРДБ и Цеффс.
- Перемещение зашифрованной виртуальной машины в другую подписку или регион.
- Создание образа или моментального снимка зашифрованной виртуальной машины и его использование для развертывания дополнительных виртуальных машин.
- Виртуальные машины серии M с Ускоритель записиными дисками.
- Применение ADE к виртуальной машине с дисками, зашифрованными с помощью [шифрования на стороне сервера, с помощью ключей, управляемых клиентом](../disk-encryption.md) (SSE + CMK). Применение SSE + CMK к диску данных на виртуальной машине, зашифрованном с помощью ADE, также является неподдерживаемым сценарием.
- Перенос виртуальной машины, зашифрованной с помощью ADE, или **когда-либо** зашифрованный с помощью ade, для [шифрования на стороне сервера с ключами, управляемыми клиентом](../disk-encryption.md).
- Шифрование виртуальных машин в отказоустойчивых кластерах.

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения о шифровании дисков Azure](disk-encryption-overview.md)
- [Примеры скриптов шифрования дисков Azure](disk-encryption-sample-scripts.md)
- [Устранение неполадок с шифрованием дисков Azure](disk-encryption-troubleshooting.md)