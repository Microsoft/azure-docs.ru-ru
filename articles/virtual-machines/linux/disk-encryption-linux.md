---
title: Сценарии шифрования дисков Azure для виртуальных машин Linux
description: В этой статье приведены инструкции по включению шифрования дисков Microsoft Azure для виртуальных машин под управлением Linux для различных сценариев
author: msmbaldwin
ms.service: virtual-machines
ms.subservice: disks
ms.collection: linux
ms.topic: conceptual
ms.author: mbaldwin
ms.date: 08/06/2019
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: bf5da120072f49421b6f2194626178048b044f6a
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102564702"
---
# <a name="azure-disk-encryption-scenarios-on-linux-vms"></a>Сценарии шифрования дисков Azure для виртуальных машин Linux

Шифрование дисков Azure для виртуальных машин Linux (VM) использует функцию DM-Crypt в Linux для обеспечения полного шифрования диска операционной системы и дисков данных. Кроме того, он обеспечивает шифрование временного диска при использовании функции EncryptFormatAll.

Шифрование дисков Azure [интегрируется с Azure Key Vault](disk-encryption-key-vault.md), помогая управлять ключами шифрования дисков и секретами. Общие сведения о службе см. в статье [шифрование дисков Azure для виртуальных машин Linux](disk-encryption-overview.md).

Шифрование дисков можно применить только к виртуальным машинам [поддерживаемых размеров и операционных систем](disk-encryption-overview.md#supported-vms-and-operating-systems). Также необходимы следующие элементы:

- [Дополнительные требования к ВМ](disk-encryption-overview.md#supported-vms-and-operating-systems)
- [Требования к сети](disk-encryption-overview.md#networking-requirements)
- [Требования к хранилищу ключей шифрования](disk-encryption-overview.md#encryption-key-storage-requirements)

Во всех случаях следует [сделать моментальный снимок](snapshot-copy-managed-disk.md) и (или) создать резервную копию перед шифрованием дисков. Резервные копии обеспечивают возможность восстановления в случае любого непредвиденного сбоя во время шифрования. Для виртуальных машин с управляемыми дисками необходимо создать резервную копию до начала шифрования. После создания резервной копии можно зашифровать управляемые диски с помощью [командлета Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) с параметром -skipVmBackup. Дополнительные сведения см. в статье [Резервное копирование и восстановление зашифрованных виртуальных машин с помощью службы Azure Backup](../../backup/backup-azure-vms-encryption.md). 

>[!WARNING]
> - Если вы уже использовали шифрование дисков Azure c Azure AD для шифрования виртуальной машины, сохраняйте и далее этот способ шифрования виртуальной машины. См. детали в разделе [Шифрование дисков Azure с использованием приложения Azure AD (предыдущий выпуск)](disk-encryption-overview-aad.md). 
>
> - При шифровании томов Linux ОС виртуальная машина должна считаться недоступной. Настоятельно рекомендуется избегать входа по протоколу SSH во время шифрования, чтобы избежать проблем с блокированием открытых файлов, к которым необходимо получить доступ во время процесса шифрования. Чтобы проверить ход выполнения, используйте командлет PowerShell [Get-AzVMDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus) или команду CLI [vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show). Этот процесс может занять несколько часов для тома операционной системы 30 ГБ, а также дополнительное время для шифрования томов данных. Если не используется параметр шифрования всех форматов, время шифрования тома данных будет соответствовать размеру и количеству томов данных. 
> - Отключение шифрования на виртуальных машинах Linux поддерживается только для томов данных. Если том операционной системы зашифрован, то отключение не поддерживается для томов данных и томов ОС.  

## <a name="install-tools-and-connect-to-azure"></a>Установка средств и подключение к Azure

Шифрованием дисков Azure можно управлять с помощью [Azure CLI](/cli/azure) и [Azure PowerShell](/powershell/azure/new-azureps-module-az). Для этого необходимо установить средства локально и подключиться к подписке Azure.

### <a name="azure-cli"></a>Azure CLI

[Azure CLI 2.0](/cli/azure) — это интерфейс командной строки для управления ресурсами Azure. Этот интерфейс обеспечивает гибкие функции подачи запросов, выполнение длительных операций без блокировки и простое создание скриптов. Его можно установить локально, выполнив действия, описанные в [Установке Azure CLI](/cli/azure/install-azure-cli).

 

Для [входа в подписку Azure в Azure CLI](/cli/azure/authenticate-azure-cli) используйте команду [az login](/cli/azure/reference-index#az_login).

```azurecli
az login
```

Если вы хотите использовать для входа определенный клиент, выполните команду:
    
```azurecli
az login --tenant <tenant>
```

Если у вас есть несколько подписок и нужно указать конкретную, получите список подписок с помощью команды [az account list](/cli/azure/account#az-account-list) и задайте требуемую, используя команду [az account set](/cli/azure/account#az-account-set).
     
```azurecli
az account list
az account set --subscription "<subscription name or ID>"
```

Дополнительные сведения можно найти в документации по [началу работы с Azure CLI 2.0](/cli/azure/get-started-with-azure-cli). 

### <a name="azure-powershell"></a>Azure PowerShell
В [Azure PowerShell az module](/powershell/azure/new-azureps-module-az) доступен набор командлетов, которые используют модель [Azure Resource Manager](../../azure-resource-manager/management/overview.md) для управления ресурсами Azure. Его можно использовать в браузере с [Azure Cloud Shell](../../cloud-shell/overview.md), а также установить на локальном компьютере, следуя инструкциям в статье [Установка модуля Azure PowerShell](/powershell/azure/install-az-ps). 

Если вы уже установили его локально, убедитесь, что вы используете последнюю версию версии пакета SDK для Azure PowerShell для настройки шифрования дисков Azure. Скачайте последнюю версию [Azure PowerShell](https://github.com/Azure/azure-powershell/releases).

Чтобы [войти в учетную запись Azure с помощью Azure PowerShell](/powershell/azure/authenticate-azureps), используйте командлет [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

```powershell
Connect-AzAccount
```

Если у вас есть несколько подписок и вы хотите указать одну из них, используйте командлет [Get-AzSubscription](/powershell/module/Az.Accounts/Get-AzSubscription) для их перечисления, а затем выполните командлет [Set-AzContext](/powershell/module/az.accounts/set-azcontext):

```powershell
Set-AzContext -Subscription -Subscription <SubscriptionId>
```

При выполнении командлета [Get-AzContext](/powershell/module/Az.Accounts/Get-AzContext) будет проверена правильность выбранной подписки.

Чтобы убедиться, что командлеты шифрования дисков Azure установлены, используйте командлет [Get-command](/powershell/module/microsoft.powershell.core/get-command):
     
```powershell
Get-command *diskencryption*
```
Дополнительные сведения см. в разделе [Приступая к работе с командлетами Azure PowerShell](/powershell/azure/get-started-azureps). 

## <a name="enable-encryption-on-an-existing-or-running-linux-vm"></a>Включите шифрование на существующей или запущенной виртуальной машине Linux
В этом сценарии шифрование можно включить с помощью шаблона Resource Manager, командлетов PowerShell или команд интерфейса командной строки. Сведения о схеме для расширения виртуальной машины см. в статье [Шифрование дисков Azure для Linux ](../extensions/azure-disk-enc-linux.md).

>[!IMPORTANT]
 >Нужно обязательно создать образ или резервную копию экземпляра виртуальной машины на основе управляемого диска вне шифрования дисков Azure и перед включением этой функции. Моментальный снимок управляемого диска можно создать с портала или использовать для этого [Azure Backup](../../backup/backup-azure-vms-encryption.md). Резервные копии обеспечивают возможность восстановления при любом непредвиденном сбое во время шифрования. После создания резервной копии можно зашифровать управляемые диски с помощью командлета Set-AzVMDiskEncryptionExtension, указав параметр -skipVmBackup. Команда Set-AzVMDiskEncryptionExtension будет завершаться ошибкой при использовании с виртуальными машинами на основе управляемых дисков, пока не будет сделана резервная копия и не будет указан определенный параметр. 
>
>Шифрование или отключение шифрования может привести к перезагрузке виртуальной машины. 
>

### <a name="enable-encryption-on-an-existing-or-running-linux-vm-using-azure-cli"></a>Включение шифрования на существующей или работающей виртуальной машине Linux с помощью Azure CLI 

Можно включить шифрование дисков для зашифрованного виртуального жесткого диска, установив программу командной строки [Azure CLI](/cli/azure/). Его можно использовать в браузере с [Azure Cloud Shell](../../cloud-shell/overview.md), а также установить на локальном компьютере и использовать в любом сеансе PowerShell. Чтобы включить шифрование на виртуальных машинах под управлением Linux, которые уже существуют или работают в Azure, используйте следующие команды CLI.

Используйте команду [az vm encryption enable](/cli/azure/vm/encryption#az_vm_encryption_show), чтобы включить шифрование на работающей виртуальной машине в Azure.

- **Шифрование работающей виртуальной машины:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault "MySecureVault" --volume-type [All|OS|Data]
     ```

- **Шифрование работающей виртуальной машины с использованием KEK:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type [All|OS|Data]
     ```

    >[!NOTE]
    > Синтаксис значения параметра disk-encryption-keyvault — это полная строка идентификатора: /subscriptions/[ИД или GUID подписки]/resourceGroups/[имя группы ресурсов]/providers/Microsoft.KeyVault/vaults/[имя хранилища ключей].</br>
Синтаксис значения параметра key-encryption-key —это полный универсальный код ресурса (URI) для ключа шифрования ключей: https://[имя хранилища ключей].vault.azure.net/keys/[имя KEK]/[ИД KEK]. 

- **Проверка того, что диски зашифрованы:** Чтобы проверить состояние шифрования виртуальной машины IaaS, используйте команду [az vm encryption show](/cli/azure/vm/encryption#az-vm-encryption-show). 

     ```azurecli-interactive
     az vm encryption show --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup"
     ```

- **Отключение шифрования:** чтобы отключить шифрование, используйте команду [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable). Для виртуальных машин Linux отключение шифрования возможно только для томов данных.

     ```azurecli-interactive
     az vm encryption disable --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup" --volume-type "data"
     ```

### <a name="enable-encryption-on-an-existing-or-running-linux-vm-using-powershell"></a>Включение шифрования на существующей или работающей виртуальной машине Linux с помощью PowerShell
Используйте командлет [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension), чтобы включить шифрование на работающей виртуальной машине в Azure. Создайте [моментальный снимок](snapshot-copy-managed-disk.md) и (или) резервную копию виртуальной машины с помощью [Azure Backup](../../backup/backup-azure-vms-encryption.md) перед шифрованием дисков. Параметр -skipVmBackup уже указан в сценариях PowerShell для шифрования работающей виртуальной машины Linux.

-  **Шифрование работающей виртуальной машины:** Приведенный ниже сценарий позволяет инициализировать переменные и запустить командлет Set-AzVMDiskEncryptionExtension. Группа ресурсов, виртуальная машина и хранилище ключей уже созданы в качестве необходимых компонентов. Замените MyVirtualMachineResourceGroup, MySecureVM и MySecureVault своими значениями. Измените значение параметра -VolumeType, чтобы указать диски, для которых выполняется шифрование.

     ```azurepowershell
      $KVRGname = 'MyKeyVaultResourceGroup';
      $VMRGName = 'MyVirtualMachineResourceGroup';
      $vmName = 'MySecureVM';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;
      $sequenceVersion = [Guid]::NewGuid();  

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType '[All|OS|Data]' -SequenceVersion $sequenceVersion -skipVmBackup;
     ```
- **Шифрование работающей виртуальной машины с использованием KEK:** При шифровании дисков данных, а не диска операционной системы, может потребоваться добавить параметр -VolumeType. 

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

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId -VolumeType '[All|OS|Data]' -SequenceVersion $sequenceVersion -skipVmBackup;
     ```

    >[!NOTE]
    > Синтаксис значения параметра disk-encryption-keyvault — это полная строка идентификатора: /subscriptions/[ИД или GUID подписки]/resourceGroups/[имя группы ресурсов]/providers/Microsoft.KeyVault/vaults/[имя хранилища ключей].</br> Синтаксис значения параметра key-encryption-key —это полный универсальный код ресурса (URI) для ключа шифрования ключей: https://[имя хранилища ключей].vault.azure.net/keys/[имя KEK]/[ИД KEK]. 
    
- **Проверка того, что диски зашифрованы:** Чтобы проверить состояние шифрования виртуальной машины IaaS, используйте командлет [Get-AzVmDiskEncryptionStatus](/powershell/module/az.compute/get-azvmdiskencryptionstatus). 
    
     ```azurepowershell-interactive 
     Get-AzVmDiskEncryptionStatus -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
     ```
    
- **Отключение шифрования дисков:** Для отключения шифрования используйте командлет [Disable-AzVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption). Для виртуальных машин Linux отключение шифрования возможно только для томов данных.
     
     ```azurepowershell-interactive 
     Disable-AzVMDiskEncryption -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
     ```

### <a name="enable-encryption-on-an-existing-or-running-linux-vm-with-a-template"></a>Включение шифрования на существующей или работающей виртуальной машине с помощью шаблона

Включить шифрование дисков на существующих или работающих виртуальных машинах под управлением Linux в Azure можно с помощью [шаблона Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-linux-vm-without-aad).

1. На странице шаблона быстрого запуска Azure нажмите кнопку **Deploy to Azure** (Развернуть в Azure).

2. Выберите подписку, группу ресурсов, расположение группы ресурсов, параметры, условия использования и соглашение. Нажмите кнопку **Создать**, чтобы включить шифрование на существующей или работающей виртуальной машине.

В следующей таблице перечислены параметры шаблона Resource Manager для имеющихся или работающих виртуальных машин.

| Параметр | Описание |
| --- | --- |
| vmName | Имя виртуальной машины для выполнения операции шифрования. |
| keyVaultName | Имя хранилища ключей, в которое будет передан ключ шифрования. Его можно получить с помощью командлета `(Get-AzKeyVault -ResourceGroupName <MyKeyVaultResourceGroupName>). Vaultname` или команды Azure CLI `az keyvault list --resource-group "MyKeyVaultResourceGroupName"`.|
| keyVaultResourceGroup | Имя группы ресурсов, содержащей хранилище ключей. |
|  keyEncryptionKeyURL | URL-адрес ключа шифрования ключей, который используется для шифрования ключа шифрования. Это необязательный параметр, если выбрать **nokek** из раскрывающегося списка UseExistingKek. Если из раскрывающегося списка UseExistingKek выбрано значение **kek**, то потребуется ввести значение _keyEncryptionKeyURL_. |
| volumeType | Тип тома, для которого будет выполняться шифрование. Допустимые значения: _OS_, _Data_ и _All_. 
| forceUpdateTag | Передает уникальное значение, такое как GUID, каждый раз, когда операция должна выполняться принудительно. |
| location | Расположение для всех ресурсов. |

Дополнительные сведения о настройке шаблона шифрования диска виртуальной машины Linux см. в статье [Шифрование дисков Azure для Linux](../extensions/azure-disk-enc-linux.md).

## <a name="use-encryptformatall-feature-for-data-disks-on-linux-vms"></a>Использование функции EncryptFormatAll для дисков данных на виртуальных машинах под управлением Linux

Параметр **EncryptFormatAll** сокращает время шифрования дисков данных Linux. Секции, соответствующие определенным условиям, будут отформатированы вместе с их текущими файловыми системами, а затем снова подключены к устройствам, где они были расположены перед выполнением команды. Если вы хотите исключить диск данных, который соответствует критериям, можно отключить его перед выполнением команды.

 После выполнения этой команды все подключенные ранее диски будут отформатированы, а уровень шифрования будет запущен поверх пустого диска. Если выбран этот параметр, временный диск с ресурсами, подключенный к виртуальной машине, также будет зашифрован. Если временный диск сбрасывается, при следующей возможности он будет переформатирован и повторно зашифрован для виртуальной машины с помощью шифрования дисков Azure. После того как диск ресурсов будет зашифрован, агент [Microsoft Azure Linux](../extensions/agent-linux.md) не сможет управлять диском ресурсов и включать файл подкачки, но вы можете вручную настроить файл подкачки.

>[!WARNING]
> Параметр EncryptFormatAll не следует использовать, если в томах данных виртуальной машины есть требуемые данные. Вы можете исключить диски из шифрования, отключив их. Сначала следует попробовать параметр EncryptFormatAll на тестовой виртуальной машине, понять его функцию и последствия, а уже потом использовать на рабочих виртуальных машинах. Параметр EncryptFormatAll форматирует диск данных, и все данные на этом диске будут потеряны. Прежде чем продолжить, убедитесь, что диски, которые вы хотите исключить, отключены надлежащим образом. </br></br>
 >Если этот параметр устанавливается при обновлении параметров шифрования, это может привести к перезагрузке перед фактическим шифрованием. В этом случае необходимо также удалить из FSTAB-файла диск, который не нужно форматировать. Аналогично, перед началом операции шифрования нужно добавить в FSTAB-файл раздел, который нужно отформатировать и зашифровать. 

### <a name="encryptformatall-criteria"></a>Критерии EncryptFormatAll
Параметр проходит через все разделы и шифрует их, если они соответствуют **всем** из приведенных ниже критериев:
- это не корневой или загрузочный раздел, а также не раздел операционной системы;
- этот раздел еще не зашифрован;
- это не том BEK;
- это не том RAID;
- это не том LVM;
- он подключен.

Выполняйте шифрование дисков, которые составляют том RAID или LVM, а не самого тома RAID или LVM.

### <a name="use-the-encryptformatall-parameter-with-azure-cli"></a>Использование параметра EncryptFormatAll с Azure CLI
Используйте команду [az vm encryption enable](/cli/azure/vm/encryption#az-vm-encryption-enable), чтобы включить шифрование на работающей виртуальной машине в Azure.

-  **Шифрование работающей виртуальной машины с использованием параметра EncryptFormatAll:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault "MySecureVault" --volume-type "data" --encrypt-format-all
     ```

### <a name="use-the-encryptformatall-parameter-with-a-powershell-cmdlet"></a>Использование параметра EncryptFormatAll с командлетом PowerShell
Используйте командлет [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) с параметром EncryptFormatAll. 

**Шифрование работающей виртуальной машины с использованием параметра EncryptFormatAll:** Например, приведенный ниже скрипт позволяет инициализировать переменные и запустить командлет Set-AzVMDiskEncryptionExtension с параметром EncryptFormatAll. Группа ресурсов, виртуальная машина и хранилище ключей уже созданы в качестве необходимых компонентов. Замените MyVirtualMachineResourceGroup, MySecureVM и MySecureVault своими значениями.
  
```azurepowershell
$KVRGname = 'MyKeyVaultResourceGroup';
$VMRGName = 'MyVirtualMachineResourceGroup';
$vmName = 'MySecureVM';
$KeyVaultName = 'MySecureVault';
$KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
$diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
$KeyVaultResourceId = $KeyVault.ResourceId;

Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType "data" -EncryptFormatAll
```


### <a name="use-the-encryptformatall-parameter-with-logical-volume-manager-lvm"></a>Использование параметра EncryptFormatAll с диспетчером логических томов (LVM) 
Рекомендуем установить LVM-on-crypt. Для всех следующих примеров замените путь к устройству и точки подключения в соответствии с вашим вариантом использования. Эту настройку можно выполнить следующим образом.

1.  Добавить диски данных, которые составляют виртуальную машину.

1. Форматировать, подключить и добавить эти диски в FSTAB-файл.

1. Выберите стандартную секцию, создайте секцию, охватывающую весь диск, а затем отформатируйте раздел. Здесь мы используем символические ссылки, созданные Azure. Использование символических ссылок позволяет избежать проблем, связанных с изменениями имен устройств. Дополнительные сведения см. в статье [Устранение неполадок при изменении имени устройства виртуальной машины Linux](../troubleshooting/troubleshoot-device-names-problems.md).
    
    ```bash
    parted /dev/disk/azure/scsi1/lun0 mklabel gpt
    parted -a opt /dev/disk/azure/scsi1/lun0 mkpart primary ext4 0% 100%
    
    mkfs -t ext4 /dev/disk/azure/scsi1/lun0-part1
    ```

1. Подключите диски:

    ```bash
    mount /dev/disk/azure/scsi1/lun0-part1 /mnt/mountpoint
    ````
    
    Добавьте сведения в FSTAB-файл:

    ```bash
    echo "/dev/disk/azure/scsi1/lun0-part1 /mnt/mountpoint ext4 defaults,nofail 0 2" >> /etc/fstab
    ```
    
1. Выполните командлет PowerShell [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) с параметром -EncryptFormatAll для шифрования этих дисков.

    ```azurepowershell-interactive
    $KeyVault = Get-AzKeyVault -VaultName "MySecureVault" -ResourceGroupName "MySecureGroup"
    
    Set-AzVMDiskEncryptionExtension -ResourceGroupName "MySecureGroup" -VMName "MySecureVM" -DiskEncryptionKeyVaultUrl $KeyVault.VaultUri -DiskEncryptionKeyVaultId $KeyVault.ResourceId -EncryptFormatAll -SkipVmBackup -VolumeType Data
    ```

    Если вы хотите использовать ключ шифрования ключа (KEK), передайте универсальный код ресурса (URI) KEK и ResourceID хранилища ключей в параметры -KeyEncryptionKeyUrl и -KeyEncryptionKeyVaultId соответственно:

    ```azurepowershell-interactive
    $KeyVault = Get-AzKeyVault -VaultName "MySecureVault" -ResourceGroupName "MySecureGroup"
    $KEKKeyVault = Get-AzKeyVault -VaultName "MyKEKVault" -ResourceGroupName "MySecureGroup"
    $KEK = Get-AzKeyVaultKey -VaultName "myKEKVault" -KeyName "myKEKName"
    
    Set-AzVMDiskEncryptionExtension -ResourceGroupName "MySecureGroup" -VMName "MySecureVM" -DiskEncryptionKeyVaultUrl $KeyVault.VaultUri -DiskEncryptionKeyVaultId $KeyVault.ResourceId -EncryptFormatAll -SkipVmBackup -VolumeType Data -KeyEncryptionKeyUrl $$KEK.id -KeyEncryptionKeyVaultId $KEKKeyVault.ResourceId
    ```

1. Настройте диспетчер логических томов поверх новых дисков. Обратите внимание, что зашифрованные диски разблокируются после завершения загрузки виртуальной машины. Таким образом, подключение диспетчера логических томов также придется отложить.

## <a name="new-vms-created-from-customer-encrypted-vhd-and-encryption-keys"></a>Новые виртуальные машины, при создании которых используются зашифрованные виртуальные жесткие диски и ключи шифрования
В этом сценарии шифрование можно включить с помощью командлетов PowerShell или команд интерфейса командной строки. 

Следуйте инструкциям, приведенным в тех же сценариях шифрования Azure Disk, чтобы подготовить предварительно зашифрованные образы, которые можно использовать в Azure. После создания образа выполните действия, описанные в следующем разделе, чтобы создать зашифрованную виртуальную машину Azure.

* [Подготовка предварительно зашифрованного виртуального жесткого диска Linux](disk-encryption-sample-scripts.md#prepare-a-pre-encrypted-linux-vhd)

>[!IMPORTANT]
 >Нужно обязательно создать образ или резервную копию экземпляра виртуальной машины на основе управляемого диска вне шифрования дисков Azure и перед включением этой функции. Моментальный снимок управляемого диска можно создать с портала или использовать для этого [Azure Backup](../../backup/backup-azure-vms-encryption.md). Резервные копии обеспечивают возможность восстановления при любом непредвиденном сбое во время шифрования. После создания резервной копии можно зашифровать управляемые диски с помощью командлета Set-AzVMDiskEncryptionExtension, указав параметр -skipVmBackup. Команда Set-AzVMDiskEncryptionExtension будет завершаться ошибкой при использовании с виртуальными машинами на основе управляемых дисков, пока не будет сделана резервная копия и не будет указан определенный параметр. 
>
> Шифрование или отключение шифрования может привести к перезагрузке виртуальной машины. 



### <a name="use-azure-powershell-to-encrypt-vms-with-pre-encrypted-vhds"></a>Использование Azure PowerShell для шифрования виртуальных машин с предварительно зашифрованными виртуальными жесткими дисками 
Можно включить шифрование дисков для зашифрованного виртуального жесткого диска с помощью командлета PowerShell [Set-AzVMOSDisk](/powershell/module/Az.Compute/Set-AzVMOSDisk#examples). Приведенный ниже пример имеет некоторые общие параметры. 

```azurepowershell
$VirtualMachine = New-AzVMConfig -VMName "MySecureVM" -VMSize "Standard_A1"
$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name "SecureOSDisk" -VhdUri "os.vhd" Caching ReadWrite -Linux -CreateOption "Attach" -DiskEncryptionKeyUrl "https://mytestvault.vault.azure.net/secrets/Test1/514ceb769c984379a7e0230bddaaaaaa" -DiskEncryptionKeyVaultId "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.KeyVault/vaults/mytestvault"
New-AzVM -VM $VirtualMachine -ResourceGroupName "MyVirtualMachineResourceGroup"
```

## <a name="enable-encryption-on-a-newly-added-data-disk"></a>Включение шифрования на добавленном диске данных

Новый диск данных можно добавить с помощью команды [az vm disk attach](add-disk.md) или [через портал Azure](attach-disk-portal.md). Прежде чем запустить шифрование, необходимо подключить только что присоединенный диск данных. Нужно запросить шифрование диска данных, так как диск будет неработоспособным во время шифрования. 

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-cli"></a>Включение шифрования на добавленном диске данных с помощью интерфейса командной строки Azure

 Если виртуальная машина ранее была зашифрована со значением All, то в дальнейшем значение параметра volume-type должно оставаться All. Значение "All" включает как диски ОС, так и диски данных. Если виртуальная машина была зашифрована с типом тома OS, в дальнейшем параметр --volume-type следует заменить на All, чтобы были включены диск ОС и новый диск данных. Если виртуальная машина была зашифрована с типом тома "Data", то в дальнейшем он может оставаться со значением "Data", как это показано ниже. Добавления и присоединения нового диска данных к виртуальной машине недостаточно для подготовки к шифрованию. Вновь подключенный диск также должен быть отформатирован и правильно подключен к виртуальной машине перед включением шифрования. В Linux диск монтируется в /etc/fstab с [постоянным именем блочного устройства](../troubleshooting/troubleshoot-device-names-problems.md).  

В отличие от синтаксиса Powershell, интерфейс командной строки (CLI) не требует предоставления уникальной версии последовательности при включении шифрования. CLI автоматически создает и использует свое собственное уникальное значение версии последовательности.

-  **Шифрование томов данных работающей виртуальной машины:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault "MySecureVault" --volume-type "Data"
     ```

- **Шифрование томов данных работающей виртуальной машины с использованием KEK:**

     ```azurecli-interactive
     az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type "Data"
     ```

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-powershell"></a>Включение шифрования на добавленном диске данных с помощью Azure PowerShell
 При использовании PowerShell для шифрования нового диска для Linux необходимо указать новую версию последовательности. Версия последовательности должна быть уникальной. Приведенный ниже сценарий создает GUID для версии последовательности. Создайте [моментальный снимок](snapshot-copy-managed-disk.md) и (или) резервную копию виртуальной машины с помощью [Azure Backup](../../backup/backup-azure-vms-encryption.md) перед шифрованием дисков. Параметр -skipVmBackup уже указан в сценариях PowerShell для шифрования только что добавленного диска данных.
 

-  **Шифрование томов данных работающей виртуальной машины:** Приведенный ниже сценарий позволяет инициализировать переменные и запустить командлет Set-AzVMDiskEncryptionExtension. Группа ресурсов, виртуальная машина и хранилище ключей должны быть уже созданы в качестве необходимых компонентов. Замените MyVirtualMachineResourceGroup, MySecureVM и MySecureVault своими значениями. Допустимые значения параметра -VolumeType: "All", "OS" и "Data". Если виртуальная машина была зашифрована с типом тома OS или All, в дальнейшем параметр -VolumeType следует заменить на All, чтобы были включены диск ОС и новый диск данных.

      ```azurepowershell
      $KVRGname = 'MyKeyVaultResourceGroup';
      $VMRGName = 'MyVirtualMachineResourceGroup';
      $vmName = 'MySecureVM';
      $KeyVaultName = 'MySecureVault';
      $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
      $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
      $KeyVaultResourceId = $KeyVault.ResourceId;
      $sequenceVersion = [Guid]::NewGuid();

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType 'data' –SequenceVersion $sequenceVersion -skipVmBackup;
      ```
- **Шифрование томов данных работающей виртуальной машины с использованием KEK:** Допустимые значения параметра -VolumeType: "All", "OS" и "Data". Если виртуальная машина была зашифрована с типом тома "OS" или "All", в дальнейшем параметр -VolumeType следует заменить на "All", чтобы были включены диск ОС и новый диск данных.

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

      Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId -VolumeType 'data' –SequenceVersion $sequenceVersion -skipVmBackup;
     ```

    >[!NOTE]
    > Синтаксис значения параметра disk-encryption-keyvault — это полная строка идентификатора: /subscriptions/[ИД или GUID подписки]/KVresourceGroups/[имя группы ресурсов]/providers/Microsoft.KeyVault/vaults/[имя хранилища ключей]</br> Синтаксис значения параметра key-encryption-key —это полный универсальный код ресурса (URI) для ключа шифрования ключей: https://[имя хранилища ключей].vault.azure.net/keys/[имя KEK]/[ИД KEK]. 


## <a name="disable-encryption-for-linux-vms"></a>Отключение шифрования для виртуальных машин Linux
[!INCLUDE [disk-encryption-disable-encryption-cli](../../../includes/disk-encryption-disable-cli.md)]

## <a name="unsupported-scenarios"></a>Неподдерживаемые сценарии

Шифрование дисков Azure не работает в следующих сценариях, функциях и технологиях Linux:

- Шифрование виртуальных машин базового уровня или виртуальных машин, созданных с помощью классического метода создания виртуальной машины.
- Отключение шифрования диска операционной системы или диска данных виртуальной машины Linux, когда диск ОС зашифрован.
- Шифрование диска ОС для масштабируемых наборов виртуальных машин Linux.
- Шифрование пользовательских образов на виртуальных машинах Linux.
- Интеграция с локальной системой управления ключами.
- служба файлов Azure (общая файловая система);
- сетевая файловая система (NFS);
- динамические тома;
- Временные диски ОС.
- Шифрование общих и распределенных файловых систем, таких как (но не ограничиваясь ими): DFS, GFS, DRDB и CephFS.
- Перемещение зашифрованной виртуальной машины в другую подписку или регион.
- Создание образа или моментального снимка зашифрованной виртуальной машины и его использование для развертывания дополнительных виртуальных машин.
- Аварийный дамп памяти ядра (kdump).
- Oracle ACFS (файловая система кластера ASM).
- Диски NVMe виртуальных машин серии Lsv2 (см. раздел [Lsv2-Series](../lsv2-series.md)).
- Виртуальная машина с "вложенными точками подключения"; то есть несколько точек подключения находятся по одному пути (например, "/1stmountpoint/Data/2stmountpoint").
- Виртуальная машина с диском данных, подключенным поверх папки ОС.
- Виртуальная машина, на которой был расширен логический том корня (диска ОС) с помощью диска данных.
- Виртуальные машины серии M с Ускоритель записиными дисками.
- Применение ADE к виртуальной машине с дисками, зашифрованными с помощью [шифрования на стороне сервера, с помощью ключей, управляемых клиентом](../disk-encryption.md) (SSE + CMK). Применение SSE + CMK к диску данных на виртуальной машине, зашифрованном с помощью ADE, также является неподдерживаемым сценарием.
- Перенос виртуальной машины, зашифрованной с помощью ADE, или **когда-либо** зашифрованный с помощью ade, для [шифрования на стороне сервера с ключами, управляемыми клиентом](../disk-encryption.md).
- Шифрование виртуальных машин в отказоустойчивых кластерах.

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения о шифровании дисков Azure](disk-encryption-overview.md)
- [Примеры скриптов шифрования дисков Azure](disk-encryption-sample-scripts.md)
- [Устранение неполадок с шифрованием дисков Azure](disk-encryption-troubleshooting.md)
