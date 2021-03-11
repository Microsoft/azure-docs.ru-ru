---
title: Шифрование дисков Azure с использованием виртуальных машин IaaS под управлением Linux для Azure AD App (предыдущий выпуск)
description: В этой статье приведены инструкции по включению шифрования дисков Microsoft Azure для виртуальных машин IaaS под управлением Linux.
author: msmbaldwin
ms.service: virtual-machines
ms.subservice: disks
ms.collection: linux
ms.topic: conceptual
ms.author: mbaldwin
ms.date: 03/15/2019
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: d1607ef4ff277f9c9cdb55db3e58da1052a00756
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102558409"
---
# <a name="enable-azure-disk-encryption-with-azure-ad-on-linux-vms-previous-release"></a>Включение шифрования дисков Azure с помощью Azure AD на виртуальных машинах Linux (предыдущий выпуск)

Новый выпуск шифрования дисков Azure исключает необходимость предоставления параметра приложения Azure Active Directory (Azure AD) для включения шифрования дисков виртуальной машины. В новом выпуске больше не требуется вводить учетные данные Azure AD на этапе включения шифрования. Все новые виртуальные машины должны быть зашифрованы без параметров приложения Azure AD с помощью нового выпуска. Инструкции по включению шифрования дисков виртуальной машины с помощью нового выпуска см. в статье [Шифрование дисков Azure для виртуальных машин Linux](disk-encryption-linux.md). Виртуальные машины, которые уже были зашифрованы с помощью параметров приложения Azure AD, по-прежнему поддерживаются. Их следует и дальше обслуживать с использованием синтаксиса AAD.

Можно включить много сценариев шифрования дисков, и действия могут отличаться в зависимости от сценария. В следующих разделах более подробно рассматриваются сценарии для виртуальных машин в инфраструктуре Linux как услуги (IaaS). Шифрование дисков можно применить только к виртуальным машинам [поддерживаемых размеров и операционных систем](disk-encryption-overview.md#supported-vms-and-operating-systems). Также необходимы следующие элементы:

- [Дополнительные требования к ВМ](disk-encryption-overview.md#supported-vms-and-operating-systems)
- [Сетевые подключения и групповая политика](disk-encryption-overview-aad.md#networking-and-group-policy)
- [Требования к хранилищу ключей шифрования](disk-encryption-overview-aad.md#encryption-key-storage-requirements)

Создайте [моментальный снимок](snapshot-copy-managed-disk.md), создайте резервную копию или и то, и другое перед шифрованием дисков. Резервные копии обеспечивают возможность восстановления в случае любого непредвиденного сбоя во время шифрования. Для виртуальных машин с управляемыми дисками необходимо создать резервную копию до начала шифрования. После создания резервной копии можно использовать командлет Set-AzVMDiskEncryptionExtension для шифрования управляемых дисков, указав параметр-skipVmBackup. Дополнительные сведения о резервном копировании и восстановлении зашифрованных виртуальных машин см. в разделе [Azure Backup](../../backup/backup-azure-vms-encryption.md). 

>[!WARNING]
 > - Если вы ранее использовали [Шифрование дисков Azure с приложением Azure AD](disk-encryption-overview-aad.md) для шифрования этой виртуальной машины, необходимо продолжить использовать этот параметр для шифрования виртуальной машины. Вы не можете использовать [Шифрование дисков Azure](disk-encryption-overview.md) на этой ЗАШИФРОВАННОЙ виртуальной машине, так как это не поддерживаемый сценарий, что означает, что отключение от приложения Azure AD для этой ЗАШИФРОВАННОЙ виртуальной машины пока не поддерживается.
 > - Чтобы гарантировать, что секреты шифрования не пересекают региональные границы, для шифрования дисков Azure требуется хранилище ключей и виртуальные машины, расположенные в одном регионе. Создайте и используйте хранилище ключей, которое находится в том же регионе, что и виртуальная машина для шифрования.
 > - При шифровании томов ОС Linux процесс может занять несколько часов. Для томов Linux, которые могут зашифроваться дольше, можно использовать больше времени, чем тома данных.
> - При шифровании томов Linux ОС виртуальная машина будет считаться недоступной. Настоятельно рекомендуется избегать входа SSH во время шифрования, чтобы избежать блокировки открытых файлов, к которым необходимо получить доступ во время процесса шифрования. Чтобы проверить ход выполнения, используйте команды [Get-азвмдискенкриптионстатус](/powershell/module/az.compute/get-azvmdiskencryptionstatus) или [шифрования ВМ](/cli/azure/vm/encryption#az-vm-encryption-show) . Этот процесс может занять несколько часов для тома операционной системы размером 30 ГБ, а также дополнительное время для шифрования томов данных. Время шифрования тома данных пропорционально размеру и количеству томов данных, если только не используется параметр **шифровать формат ALL** . 
 > - Отключение шифрования на виртуальных машинах Linux поддерживается только для томов данных. Он не поддерживается на томах данных или ОС, если том ОС зашифрован. 

 

## <a name="enable-encryption-on-an-existing-or-running-iaas-linux-vm"></a><a name="bkmk_RunningLinux"> </a> Включение шифрования на виртуальной машине IaaS под управлением Linux, которая уже существует или работает

В этом сценарии шифрование можно включить с помощью шаблона Azure Resource Manager, командлетов PowerShell или Azure CLI команд. 

>[!IMPORTANT]
 >Необходимо создать моментальный снимок или создать резервную копию управляемого диска виртуальной машины, находящегося за пределами и до включения шифрования дисков Azure. Вы можете создать моментальный снимок управляемого диска из портал Azure или использовать [Azure Backup](../../backup/backup-azure-vms-encryption.md). Резервные копии обеспечивают возможность восстановления при любом непредвиденном сбое во время шифрования. После создания резервной копии используйте командлет Set-AzVMDiskEncryptionExtension для шифрования управляемых дисков, указав параметр-skipVmBackup. Команда Set-AzVMDiskEncryptionExtension не выполняется для управляемых виртуальных машин на основе дисков до тех пор, пока не будет создана резервная копия и указан этот параметр. 
>
>Шифрование или отключение шифрования может привести к перезагрузке виртуальной машины. 
>

### <a name="enable-encryption-on-an-existing-or-running-linux-vm-by-using-the-azure-cli"></a><a name="bkmk_RunningLinuxCLI"> </a>Включение шифрования на существующей или работающей виртуальной машине Linux с помощью Azure CLI 
Вы можете включить шифрование дисков на зашифрованном виртуальном жестком диске, установив и используя программу командной строки [Azure CLI 2,0](/cli/azure) . Его можно использовать в браузере с [Azure Cloud Shell](../../cloud-shell/overview.md), а также установить на локальном компьютере и использовать в любом сеансе PowerShell. Чтобы включить шифрование на виртуальных машинах IaaS под управлением Linux, которые уже существуют или работают в Azure, используйте следующие команды CLI.

Используйте команду [az vm encryption enable](/cli/azure/vm/encryption#az-vm-encryption-enable), чтобы включить шифрование на работающей виртуальной машине IaaS в Azure.

-  **Зашифруйте работающую виртуальную машину с помощью секрета клиента:**
    
     ```azurecli-interactive
         az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --aad-client-id "<my spn created with CLI/my Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault "MySecureVault" --volume-type [All|OS|Data]
     ```

- **Зашифруйте работающую виртуальную машину с помощью KEK, чтобы заключить секрет клиента:**
    
     ```azurecli-interactive
         az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --aad-client-id "<my spn created with CLI which is the Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type [All|OS|Data]
     ```

   >[!NOTE]
   > Синтаксис значения параметра Disk-Encryption-keyvault — это полная строка идентификатора:/Subscriptions/[Subscription-ID-GUID]/resourceGroups/[Resource-Group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name].</br> </br> Синтаксис значения параметра ключа шифрования ключей — это полный универсальный код ресурса (URI) для KEK, как в: https://[keyvault-name]. Vault. Azure. NET/ключи/[кекнаме]/[KEK-Unique-ID].

- **Убедитесь, что диски зашифрованы:** Чтобы проверить состояние шифрования виртуальной машины IaaS, используйте команду [AZ VM Encryption демонстрация](/cli/azure/vm/encryption#az-vm-encryption-show) . 

     ```azurecli-interactive
         az vm encryption show --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup"
     ```

- **Отключение шифрования:** чтобы отключить шифрование, используйте команду [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable). Для виртуальных машин Linux отключение шифрования возможно только для томов данных.
    
     ```azurecli-interactive
         az vm encryption disable --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup" --volume-type DATA
     ```

### <a name="enable-encryption-on-an-existing-or-running-linux-vm-by-using-powershell"></a><a name="bkmk_RunningLinuxPSH"></a> Включение шифрования на существующей или работающей виртуальной машине Linux с помощью PowerShell
Используйте командлет [Set-азвмдискенкриптионекстенсион](/powershell/module/az.compute/set-azvmdiskencryptionextension) , чтобы включить шифрование на работающей виртуальной машине IaaS в Azure. Создайте [моментальный снимок](snapshot-copy-managed-disk.md) или создайте резервную копию виртуальной машины с [Azure Backup](../../backup/backup-azure-vms-encryption.md) перед шифрованием дисков. Параметр -skipVmBackup уже указан в сценариях PowerShell для шифрования работающей виртуальной машины Linux.

- **Зашифруйте работающую виртуальную машину с помощью секрета клиента:** Следующий скрипт инициализирует переменные и запускает командлет Set-AzVMDiskEncryptionExtension. Группа ресурсов, виртуальная машина, хранилище ключей, приложение Azure AD и секрет клиента должны быть уже созданы в качестве необходимых компонентов. Замените Мивиртуалмачинересаурцеграуп, Микэйваултресаурцеграуп, Мисекуревм, Мисекуреваулт, My-AAD-Client-ID, а мой-AAD-Client-Secret значениями. Измените значение параметра -VolumeType, чтобы указать диски, для которых выполняется шифрование.

     ```azurepowershell
         $VMRGName = 'MyVirtualMachineResourceGroup';
         $KVRGname = 'MyKeyVaultResourceGroup';
         $vmName = 'MySecureVM';
         $aadClientID = 'My-AAD-client-ID';
         $aadClientSecret = 'My-AAD-client-secret';
         $KeyVaultName = 'MySecureVault';
         $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
         $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
         $KeyVaultResourceId = $KeyVault.ResourceId;
         $sequenceVersion = [Guid]::NewGuid();
    
         Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType '[All|OS|Data]' -SequenceVersion $sequenceVersion -skipVmBackup;
     ```
- **Зашифруйте работающую виртуальную машину с помощью KEK, чтобы заключить секрет клиента:** Шифрование дисков Azure позволяет указать существующий ключ в хранилище ключей, чтобы включить в него секреты шифрования дисков, созданные при включении шифрования. Если указан ключ шифрования ключа, шифрование дисков Azure использует этот ключ для упаковки секретов шифрования перед записью в хранилище ключей. Измените значение параметра -VolumeType, чтобы указать диски, для которых выполняется шифрование. 

     ```azurepowershell
         $KVRGname = 'MyKeyVaultResourceGroup';
         $VMRGName = 'MyVirtualMachineResourceGroup';
         $aadClientID = 'My-AAD-client-ID';
         $aadClientSecret = 'My-AAD-client-secret';
         $KeyVaultName = 'MySecureVault';
         $keyEncryptionKeyName = 'MyKeyEncryptionKey';
         $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
         $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
         $KeyVaultResourceId = $KeyVault.ResourceId;
         $keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;
         $sequenceVersion = [Guid]::NewGuid();
    
         Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId -VolumeType '[All|OS|Data]' -SequenceVersion $sequenceVersion -skipVmBackup;
     ```

  >[!NOTE]
  > Синтаксис значения параметра Disk-Encryption-keyvault — это полная строка идентификатора:/Subscriptions/[Subscription-ID-GUID]/resourceGroups/[Квресаурце-Group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name].</br> </br>
  Синтаксис значения параметра ключа шифрования ключей — это полный универсальный код ресурса (URI) для KEK, как в: https://[keyvault-name]. Vault. Azure. NET/ключи/[кекнаме]/[KEK-Unique-ID]. 
    
- **Убедитесь, что диски зашифрованы:** Чтобы проверить состояние шифрования виртуальной машины IaaS, используйте командлет [Get-азвмдискенкриптионстатус](/powershell/module/az.compute/get-azvmdiskencryptionstatus) . 
    
     ```azurepowershell-interactive 
         Get-AzVmDiskEncryptionStatus -ResourceGroupName MyVirtualMachineResourceGroup -VMName MySecureVM
     ```
    
- **Отключение шифрования дисков.** Для отключения шифрования используйте командлет [Disable-AzureRmVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption). Для виртуальных машин Linux отключение шифрования возможно только для томов данных.
     
     ```azurepowershell-interactive 
         Disable-AzVMDiskEncryption -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM'
     ```


### <a name="enable-encryption-on-an-existing-or-running-iaas-linux-vm-with-a-template"></a><a name="bkmk_RunningLinux"> </a> Включение шифрования на существующей или работающей виртуальной машине с помощью шаблона

Включить шифрование дисков на существующих или работающих виртуальных машинах IaaS под управлением Linux в Azure можно с помощью [шаблона Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-linux-vm).

1. Выберите **развертывание в Azure** в шаблоне быстрого запуска Azure.

2. Выберите подписку, группу ресурсов, расположение группы ресурсов, параметры, условия использования и соглашение. Выберите **создать** , чтобы включить шифрование на существующей или работающей виртуальной машине IaaS.

В следующей таблице перечислены параметры шаблона Resource Manager для существующих или работающих виртуальных машин, которые используют идентификатор клиента Azure AD.

| Параметр | Описание |
| --- | --- |
| AADClientID | Идентификатор клиента приложения Azure AD, которое имеет разрешения на запись секретов в хранилище ключей. |
| AADClientSecret | Секрет клиента приложения Azure AD, которое имеет разрешения на запись секретов в ваше хранилище ключей. |
| keyVaultName | Имя хранилища ключей, в которое будет передан ключ. Его можно получить с помощью команды интерфейса командной строки Azure `az keyvault show --name "MySecureVault" --query KVresourceGroup`. |
|  keyEncryptionKeyURL | URL-адрес ключа шифрования ключей, который используется для шифрования созданного ключа. Этот параметр является необязательным, если в раскрывающемся списке **UseExistingKek** выбрано **nokek** . Если в раскрывающемся списке **UseExistingKek** выбрать **KEK** , необходимо ввести значение _keyEncryptionKeyURL_ . |
| volumeType | Тип тома, для которого будет выполняться шифрование. Допустимые поддерживаемые значения: _OS_ или _ALL_. (См. раздел Поддерживаемые дистрибутивы Linux и их версии для ОС и дисков данных в разделе Предварительные требования выше.) |
| sequenceVersion | Версия последовательности операций с BitLocker. Этот номер версии увеличивается каждый раз, когда шифрование диска выполняется на той же виртуальной машине. |
| vmName | Имя виртуальной машины, для которой будет выполняться шифрование. |
| парольная фраза | Введите надежную парольную фразу, которая будет служить ключом шифрования данных. |



## <a name="use-the-encryptformatall-feature-for-data-disks-on-linux-iaas-vms"></a><a name="bkmk_EFA"> </a>Использование функции енкриптформаталл для дисков данных на виртуальных машинах IaaS под управлением Linux
Параметр EncryptFormatAll сокращает время шифрования дисков данных Linux. Секции, удовлетворяющие определенным условиям, форматируются (с их текущей файловой системой). Затем они снова подключаются к месту выполнения команды. Если вы хотите исключить диск данных, который соответствует критериям, отключите его перед выполнением команды.

 После выполнения этой команды все подключенные ранее диски форматируются. Затем уровень шифрования начинается поверх пустого диска. Если выбран этот параметр, то временный диск, подключенный к виртуальной машине, также шифруется. Если временный диск сброшен, он переформатируется и повторно шифруется для виртуальной машины с помощью решения шифрования дисков Azure в следующей возможности.

>[!WARNING]
> Енкриптформаталл не следует использовать при наличии необходимых данных на томах данных виртуальной машины. Вы можете исключить диски из шифрования, отключив их. Сначала Испытайте параметр Енкриптформаталл на тестовой виртуальной машине, чтобы понять параметр компонента и его результат, прежде чем использовать его на рабочей виртуальной машине. Параметр Енкриптформаталл форматирует диск данных, поэтому все данные на нем будут потеряны. Прежде чем продолжать, убедитесь, что все диски, которые вы хотите исключить, правильно отключены. </br></br>
 >Если задать этот параметр во время обновления параметров шифрования, это может привести к перезагрузке перед фактическим шифрованием. В этом случае необходимо также удалить диск, который вы не хотите отформатировать из файла fstab. Аналогичным образом необходимо добавить секцию, для которой требуется шифрование, в файл fstab перед запуском операции шифрования. 

### <a name="encryptformatall-criteria"></a><a name="bkmk_EFACriteria"></a> Критерии енкриптформаталл
Параметр проходит через все секции и шифрует их при условии, что они отвечают *всем* следующим критериям: 
- это не корневой или загрузочный раздел, а также не раздел операционной системы;
- этот раздел еще не зашифрован;
- это не том BEK;
- это не том RAID;
- это не том LVM;
- он подключен.

Выполняйте шифрование дисков, которые составляют том RAID или LVM, а не самого тома RAID или LVM.

### <a name="use-the-encryptformatall-parameter-with-a-template"></a><a name="bkmk_EFATemplate"> </a> Использование параметра EncryptFormatAll с шаблоном
Чтобы использовать параметр Енкриптформаталл, используйте любой существующий шаблон Azure Resource Manager, который шифрует виртуальную машину Linux и изменяет поле **енкриптионоператион** для ресурса AzureDiskEncryption.

1. Например, используйте [шаблон Resource Manager для шифрования работающей виртуальной машины IaaS под управлением Linux](https://github.com/vermashi/azure-quickstart-templates/tree/encrypt-format-running-linux-vm/201-encrypt-running-linux-vm). 
2. Выберите **развертывание в Azure** в шаблоне быстрого запуска Azure.
3. Измените значение поля **енкриптионоператион** с **EnableEncryption** на **енаблинкриптионформатал**.
4. Выберите подписку, группу ресурсов, расположение группы ресурсов, другие параметры, условия использования и соглашение. Выберите **создать** , чтобы включить шифрование на существующей или работающей виртуальной машине IaaS.


### <a name="use-the-encryptformatall-parameter-with-a-powershell-cmdlet"></a><a name="bkmk_EFAPSH"></a> Использование параметра енкриптформаталл с командлетом PowerShell
Используйте командлет [Set-AzVMDiskEncryptionExtension](/powershell/module/az.compute/set-azvmdiskencryptionextension) с параметром EncryptFormatAll.

**Зашифруйте работающую виртуальную машину с помощью секрета клиента и енкриптформаталл:** Например, следующий скрипт инициализирует переменные и запускает командлет Set-AzVMDiskEncryptionExtension с параметром Енкриптформаталл. Группа ресурсов, виртуальная машина, хранилище ключей, приложение Azure AD и секрет клиента должны быть уже созданы в качестве необходимых компонентов. Замените Микэйваултресаурцеграуп, Мивиртуалмачинересаурцеграуп, Мисекуревм, Мисекуреваулт, My-AAD-Client-ID, а мой-AAD-Client-Secret значениями.
  
   ```azurepowershell
     $KVRGname = 'MyKeyVaultResourceGroup';
     $VMRGName = 'MyVirtualMachineResourceGroup'; 
     $aadClientID = 'My-AAD-client-ID';
     $aadClientSecret = 'My-AAD-client-secret';
     $KeyVaultName = 'MySecureVault';
     $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
     $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
     $KeyVaultResourceId = $KeyVault.ResourceId;
      
     Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -EncryptFormatAll
   ```


### <a name="use-the-encryptformatall-parameter-with-logical-volume-manager-lvm"></a><a name="bkmk_EFALVM"></a> Использование параметра Енкриптформаталл с диспетчером логических ТОМОВ (LVM) 
Рекомендуем установить LVM-on-crypt. Для всех следующих примеров замените Device-Path и точки подключения на любой из вариантов использования. Эту настройку можно выполнить следующим образом.

- Добавить диски данных, которые составляют виртуальную машину.
- Форматировать, подключить и добавить эти диски в FSTAB-файл.

    1. Отформатируйте добавленный диск. Здесь мы используем символические ссылки, созданные Azure. Использование символических ссылок позволяет избежать проблем, связанных с изменениями имен устройств. Дополнительные сведения см. в разделе [Устранение неполадок с именами устройств](../troubleshooting/troubleshoot-device-names-problems.md).
    
        ```console
        mkfs -t ext4 /dev/disk/azure/scsi1/lun0
        ```

    2. Подключите диски.

        ```console
        mount /dev/disk/azure/scsi1/lun0 /mnt/mountpoint
        ```

    3. Добавьте сведения в FSTAB-файл.

        ```console
        echo "/dev/disk/azure/scsi1/lun0 /mnt/mountpoint ext4 defaults,nofail 1 2" >> /etc/fstab
        ```

    4. Выполните командлет PowerShell Set-AzVMDiskEncryptionExtension с параметром-Енкриптформаталл, чтобы зашифровать эти диски.

       ```azurepowershell-interactive
        Set-AzVMDiskEncryptionExtension -ResourceGroupName "MySecureGroup" -VMName "MySecureVM" -DiskEncryptionKeyVaultUrl "https://mykeyvault.vault.azure.net/" -EncryptFormatAll
        ```

    5. Настройте диспетчер логических томов поверх новых дисков. Обратите внимание, что зашифрованные диски разблокируются после завершения загрузки виртуальной машины. Таким образом, подключение диспетчера логических томов также придется отложить.




## <a name="new-iaas-vms-created-from-customer-encrypted-vhd-and-encryption-keys"></a><a name="bkmk_VHDpre"> </a> Новые виртуальные машины IaaS, при создании которых используются зашифрованные виртуальные жесткие диски и ключи шифрования
В этом сценарии шифрование можно включить с помощью шаблона Resource Manager, командлетов PowerShell или команд интерфейса командной строки. В следующих разделах более подробно описаны шаблон Resource Manager и команды интерфейса командной строки. 

Следуйте инструкциям, приведенным в приложении к этой статье, чтобы подготовить предварительно зашифрованные образы, которые можно использовать в Azure. После создания образа выполните действия, описанные в следующем разделе, чтобы создать зашифрованную виртуальную машину Azure.

* [Подготовка предварительно зашифрованного виртуального жесткого диска Linux](disk-encryption-sample-scripts.md)

>[!IMPORTANT]
 >Необходимо создать моментальный снимок или создать резервную копию управляемого диска виртуальной машины, находящегося за пределами и до включения шифрования дисков Azure. Вы можете создать моментальный снимок управляемого диска на портале или использовать [Azure Backup](../../backup/backup-azure-vms-encryption.md). Резервные копии обеспечивают возможность восстановления при любом непредвиденном сбое во время шифрования. После создания резервной копии используйте командлет Set-AzVMDiskEncryptionExtension для шифрования управляемых дисков, указав параметр-skipVmBackup. Команда Set-AzVMDiskEncryptionExtension не выполняется для управляемых виртуальных машин на основе дисков до тех пор, пока не будет создана резервная копия и указан этот параметр. 
>
>Шифрование или отключение шифрования может привести к перезагрузке виртуальной машины.



### <a name="use-azure-powershell-to-encrypt-iaas-vms-with-pre-encrypted-vhds"></a><a name="bkmk_VHDprePSH"> </a> Использование Azure PowerShell для шифрования виртуальных машин IaaS с предварительно зашифрованными виртуальными жесткими дисками 
Можно включить шифрование дисков для зашифрованного виртуального жесткого диска с помощью командлета PowerShell [Set-AzVMOSDisk](/powershell/module/az.compute/set-azvmosdisk#examples). В следующем примере приводятся некоторые общие параметры. 

```powershell
$VirtualMachine = New-AzVMConfig -VMName "MySecureVM" -VMSize "Standard_A1"
$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name "SecureOSDisk" -VhdUri "os.vhd" Caching ReadWrite -Windows -CreateOption "Attach" -DiskEncryptionKeyUrl "https://mytestvault.vault.azure.net/secrets/Test1/514ceb769c984379a7e0230bddaaaaaa" -DiskEncryptionKeyVaultId "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.KeyVault/vaults/mytestvault"
New-AzVM -VM $VirtualMachine -ResourceGroupName "MyVirtualMachineResourceGroup"
```

## <a name="enable-encryption-on-a-newly-added-data-disk"></a>Включение шифрования на добавленном диске данных
Новый диск данных можно добавить с помощью команды [AZ VM Disk Attach](add-disk.md) или с [помощью портал Azure](attach-disk-portal.md). Прежде чем запустить шифрование, необходимо подключить только что присоединенный диск данных. Необходимо запрашивать шифрование диска данных, так как диск будет непригоден для использования во время выполнения шифрования. 

### <a name="enable-encryption-on-a-newly-added-disk-with-the-azure-cli"></a>Включение шифрования для вновь добавленного диска с Azure CLI
 Если виртуальная машина была ранее зашифрована с помощью "ALL", параметр--Volume-Type должен оставаться в любом случае. Значение "All" включает как диски ОС, так и диски данных. Если виртуальная машина была ранее зашифрована с типом тома "OS", то параметр--Volume-Type следует изменить на ALL, чтобы в него включались как ОС, так и новый диск данных. Если виртуальная машина была зашифрована только с типом тома "Data", она может сохранить данные, как показано здесь. Добавление и присоединение нового диска данных к виртуальной машине недостаточно для подготовки к шифрованию. Перед включением шифрования вновь подключенный диск необходимо отформатировать и правильно подключить в виртуальной машине. В Linux диск должен быть подключен в/etc/fstab с [постоянным блочным именем устройства](../troubleshooting/troubleshoot-device-names-problems.md). 

В отличие от синтаксиса PowerShell, CLI не требует предоставления уникальной версии последовательности при включении шифрования. CLI автоматически создает и использует свое собственное уникальное значение версии последовательности.

-  **Зашифруйте работающую виртуальную машину с помощью секрета клиента:** 
    
     ```azurecli-interactive
         az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --aad-client-id "<my spn created with CLI/my Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault "MySecureVault" --volume-type "Data"
     ```

- **Зашифруйте работающую виртуальную машину с помощью KEK, чтобы заключить секрет клиента:**

     ```azurecli-interactive
         az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" --name "MySecureVM" --aad-client-id "<my spn created with CLI which is the Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type "Data"
     ```

### <a name="enable-encryption-on-a-newly-added-disk-with-azure-powershell"></a>Включение шифрования на добавленном диске данных с помощью Azure PowerShell
 При использовании PowerShell для шифрования нового диска для Linux необходимо указать новую версию последовательности. Версия последовательности должна быть уникальной. Следующий скрипт создает идентификатор GUID для версии последовательности. 
 

-  **Зашифруйте работающую виртуальную машину с помощью секрета клиента:** Следующий скрипт инициализирует переменные и запускает командлет Set-AzVMDiskEncryptionExtension. Группа ресурсов, виртуальная машина, хранилище ключей, приложение Azure AD и секрет клиента должны быть уже созданы в качестве необходимых компонентов. Замените Мивиртуалмачинересаурцеграуп, Микэйваултресаурцеграуп, Мисекуревм, Мисекуреваулт, My-AAD-Client-ID, а мой-AAD-Client-Secret значениями. Для параметра -VolumeType устанавливаются диски данных, а не диск операционной системы. Если виртуальная машина была ранее зашифрована с типом тома "OS" или "ALL", параметр-параметра volumetype значение должен быть изменен на ALL, чтобы были включены как ОС, так и новый диск данных.

     ```azurepowershell
         $KVRGname = 'MyKeyVaultResourceGroup';
         $VMRGName = 'MyVirtualMachineResourceGroup'; 
         $vmName = 'MySecureVM';
         $aadClientID = 'My-AAD-client-ID';
         $aadClientSecret = 'My-AAD-client-secret';
         $KeyVaultName = 'MySecureVault';
         $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
         $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
         $KeyVaultResourceId = $KeyVault.ResourceId;
         $sequenceVersion = [Guid]::NewGuid();
    
         Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType 'data' –SequenceVersion $sequenceVersion;
     ```
- **Зашифруйте работающую виртуальную машину с помощью KEK, чтобы заключить секрет клиента:** Шифрование дисков Azure позволяет указать существующий ключ в хранилище ключей, чтобы включить в него секреты шифрования дисков, созданные при включении шифрования. Если указан ключ шифрования ключа, шифрование дисков Azure использует этот ключ для упаковки секретов шифрования перед записью в хранилище ключей. Для параметра -VolumeType устанавливаются диски данных, а не диск операционной системы. Если виртуальная машина была ранее зашифрована с типом тома "OS" или "ALL", параметр-параметра volumetype значение должен быть изменен на ALL, чтобы были включены как ОС, так и новый диск данных.

     ```azurepowershell
         $KVRGname = 'MyKeyVaultResourceGroup';
         $VMRGName = 'MyVirtualMachineResourceGroup';
         $vmName = 'MyExtraSecureVM';
         $aadClientID = 'My-AAD-client-ID';
         $aadClientSecret = 'My-AAD-client-secret';
         $KeyVaultName = 'MySecureVault';
         $keyEncryptionKeyName = 'MyKeyEncryptionKey';
         $KeyVault = Get-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $KVRGname;
         $diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
         $KeyVaultResourceId = $KeyVault.ResourceId;
         $keyEncryptionKeyUrl = (Get-AzKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;
         $sequenceVersion = [Guid]::NewGuid();
    
         Set-AzVMDiskEncryptionExtension -ResourceGroupName $VMRGName -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId -VolumeType 'data' –SequenceVersion $sequenceVersion;
     ```


>[!NOTE]
> Синтаксис значения параметра Disk-Encryption-keyvault — это полная строка идентификатора:/Subscriptions/[Subscription-ID-GUID]/resourceGroups/[Resource-Group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name]. </br> </br>
Синтаксис значения параметра ключа шифрования ключей — это полный универсальный код ресурса (URI) для KEK, как в: https://[keyvault-name]. Vault. Azure. NET/ключи/[кекнаме]/[KEK-Unique-ID].

## <a name="disable-encryption-for-linux-vms"></a>Отключение шифрования для виртуальных машин Linux
Шифрование можно отключить с помощью Azure PowerShell, Azure CLI или шаблона диспетчер ресурсов. 

>[!IMPORTANT]
>Отключение шифрования, выполняемого с помощью службы шифрования Azure, на виртуальных машинах Linux поддерживается только для томов данных. Он не поддерживается на томах данных или ОС, если том ОС зашифрован. 

- **Отключите шифрование дисков с помощью Azure PowerShell:** Чтобы отключить шифрование, используйте командлет [Disable-AzureRmVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption) . 
     ```azurepowershell-interactive
         Disable-AzVMDiskEncryption -ResourceGroupName 'MyVirtualMachineResourceGroup' -VMName 'MySecureVM' [--volume-type {ALL, DATA, OS}]
     ```

- **Отключение шифрования дисков с помощью Azure CLI:** чтобы отключить шифрование, используйте команду [az vm encryption disable](/cli/azure/vm/encryption#az-vm-encryption-disable). 
     ```azurecli-interactive
         az vm encryption disable --name "MySecureVM" --resource-group "MyVirtualMachineResourceGroup" --volume-type [ALL, DATA, OS]
     ```
- **Отключите шифрование с помощью шаблона диспетчер ресурсов:** Чтобы отключить шифрование, используйте шаблон [отключить шифрование на работающей виртуальной машине Linux](https://aka.ms/decrypt-linuxvm) .
     1. Выберите **развертывание в Azure**.
     2. Выберите подписку, группу ресурсов, расположение, виртуальную машину, условия использования и соглашение.
     3. Выберите **приобрести** , чтобы отключить шифрование дисков на работающей виртуальной машине Windows. 


## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения о шифровании дисков Azure для Linux](disk-encryption-overview-aad.md)
- [Создание и Настройка хранилища ключей для шифрования дисков Azure с помощью Azure AD (предыдущий выпуск)](disk-encryption-key-vault-aad.md)
