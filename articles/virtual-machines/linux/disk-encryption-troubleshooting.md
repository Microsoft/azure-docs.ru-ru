---
title: Устранение неполадок шифрования дисков Azure для виртуальных машин Linux
description: В этой статье содержатся советы по устранению неполадок Microsoft Azure шифровании дисков для виртуальных машин Linux.
author: msmbaldwin
ms.service: virtual-machines
ms.subservice: disks
ms.collection: linux
ms.topic: troubleshooting
ms.author: mbaldwin
ms.date: 08/06/2019
ms.custom: seodec18
ms.openlocfilehash: 58ea788d6959b039ff5de8a4a889c531b987d826
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102563376"
---
# <a name="azure-disk-encryption-for-linux-vms-troubleshooting-guide"></a>Руководство по устранению неполадок шифрования дисков Azure для виртуальных машин Linux

Это руководство предназначено для ИТ-специалистов, аналитиков в сфере информационной безопасности и администраторов облака, организации которых используют шифрование дисков Azure. Здесь также содержатся рекомендации по устранению неполадок, связанных с шифрованием дисков.

Перед выполнением описанных ниже действий убедитесь, что виртуальные машины, которые вы пытаетесь зашифровать, имеют [поддерживаемые размеры и операционные системы](disk-encryption-overview.md#supported-vms-and-operating-systems) и что выполнены все предварительные требования:

- [Дополнительные требования к ВМ](disk-encryption-overview.md#supported-vms-and-operating-systems)
- [Требования к сети](disk-encryption-overview.md#networking-requirements)
- [Требования к хранилищу ключей шифрования](disk-encryption-overview.md#encryption-key-storage-requirements)

 

## <a name="troubleshooting-linux-os-disk-encryption"></a>Устранение неполадок шифрования диска ОС Linux

Для шифрования диска операционной системы Linux нужно отключить диск операционной системы перед запуском процесса полного шифрования. Если это невозможно, вероятно, отобразится сообщение об ошибке.

Эта ошибка может возникать при попыток шифрования диска ОС на виртуальной машине с средой, которая была изменена из образа коллекции поддерживаемых запасов. Отклонения от поддерживаемого образа могут мешать предоставляемой расширением возможности отключать диск ОС. Примеры отклонений могут включать в себя следующее:
- Настроенные образы, несоответствующие поддерживаемой файловой системе или схеме секционирования.
- Большие приложения, такие как SAP, MongoDB, Apache Cassandra или Docker, не поддерживаются, если они устанавливаются и выполняются в ОС перед шифрованием. Служба шифрования дисков Azure не может безопасно завершить эти процессы (это необходимо сделать при подготовке диска ОС к шифрованию). Если на диске ОС останутся активные процессы, не позволяющие закрыть дескрипторы файлов, диск ОС будет невозможно отключить, что, в свою очередь, приведет к сбою шифрования этого диска. 
- Настраиваемые скрипты, выполняющиеся практически одновременно с включением шифрования или при внесении каких-либо других изменений на виртуальной машине во время процесса шифрования. Это может произойти, если шаблон Azure Resource Manager определяет несколько расширений для одновременного выполнения, или когда настраиваемое расширение скрипта или другое действие выполняются одновременно с шифрованием диска. Сериализация и изоляция таких шагов может помочь решить проблему.
- Если модуль Security Enhanced Linux (SELinux) не был отключен перед включением шифрования, отключение диска завершается ошибкой. SELinux можно будет снова включить после завершения шифрования.
- Диск операционной системы использует схему диспетчера логических томов. При этом доступна ограниченная поддержка диска данных диспетчера логических томов, но диск операционной системы диспетчера логических томов не поддерживается.
- Минимальные требования к памяти не выполняются (для шифрования диска операционной системы рекомендуется 7 ГБ памяти).
- Диски данных рекурсивно подключены в каталоге /mnt/ или любом другом (например, /mnt/data1, /mnt/data2, /data3 + /data3/data4).

## <a name="update-the-default-kernel-for-ubuntu-1404-lts"></a> Обновление ядра по умолчанию для Ubuntu 14.04 LTS

Образ Ubuntu 14.04 LTS поставляется с версией ядра 4.4 по умолчанию. В этой версии ядра существует известная проблема, в которой компонент, призванный решать проблему недостатка памяти, неправильно завершает команду dd во время процесса шифрования ОС. Эта ошибка исправлена в самой последней версии настроенного ядра Linux Azure. Чтобы избежать этой ошибки, перед включением шифрования в образе обновите настройки ядра до версии [Azure 4.15](https://packages.ubuntu.com/trusty/linux-azure) или более поздней с помощью следующих команд.

```
sudo apt-get update
sudo apt-get install linux-azure
sudo reboot
```

После перезапуска виртуальной машины в новое ядро новую версию ядра можете подтвердить с помощью следующей команды.

```
uname -a
```

## <a name="update-the-azure-virtual-machine-agent-and-extension-versions"></a>Обновление агента и версий расширений виртуальной машины Azure

Операции шифрования дисков Azure могут завершаться сбоем в образах виртуальных машин с помощью неподдерживаемых версий агента виртуальной машины Azure. Образы Linux с версиями агента, предшествующими 2.2.38, должны быть обновлены до включения шифрования. Дополнительные сведения см. в статьях [обновление агента Linux для Azure на виртуальной машине](../extensions/update-linux-agent.md) и [Минимальная поддержка версий для агентов виртуальных машин в Azure](https://support.microsoft.com/en-us/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support).

Также требуется правильная версия расширения гостевого агента Microsoft. Azure. Security. AzureDiskEncryption или Microsoft. Azure. Security. AzureDiskEncryptionForLinux. Версии расширений поддерживаются и обновляются автоматически платформой при удовлетворении предварительных требований для агента виртуальной машины Azure и использовании поддерживаемой версии агента виртуальной машины.

Расширение Microsoft. OSTCExtensions. AzureDiskEncryptionForLinux устарело и больше не поддерживается.  

## <a name="unable-to-encrypt-linux-disks"></a>Сбой шифрования дисков Linux

В некоторых случаях шифрование диска зависает на этапе "OS disk encryption started" (Начато шифрование диска ОС) и SSH отключается. При выполнении в готовом образе из коллекции этот процесс может занять от 3 до 16 часов. Если добавляется диск данных объемом в несколько терабайт, процесс может занять несколько дней.

Последовательность шифрования диска операционной системы Linux временно отключает диск операционной системы. Затем выполняется блочное шифрование всего диска операционной системы перед его повторным подключением в зашифрованном состоянии. Шифрование дисков Linux не позволяет одновременно использовать виртуальную машину во время выполнения шифрования. Характеристики производительности виртуальной машины могут существенно повлиять на время, необходимое для завершения шифрования. К таким характеристикам относится размер диска и класс учетной записи хранения ("Стандартный" или "Премиум" (SSD)).

Во время шифрования диска ОС виртуальная машина переходит в состояние обслуживания и отключает SSH, чтобы предотвратить перерывы в работе текущего процесса.  Чтобы проверить состояние шифрования, воспользуйтесь командой Azure PowerShell [Get-азвмдискенкриптионстатус](/powershell/module/az.compute/get-azvmdiskencryptionstatus) и проверьте поле **ProgressMessage** . **ProgressMessage** будет сообщать о ряде состояний по мере шифрования дисков данных и ОС:

```azurepowershell
PS > Get-AzVMDiskEncryptionStatus -ResourceGroupName "MyResourceGroup" -VMName "myVM"

OsVolumeEncrypted          : EncryptionInProgress
DataVolumesEncrypted       : EncryptionInProgress
OsVolumeEncryptionSettings :
ProgressMessage            : Transitioning

PS > Get-AzVMDiskEncryptionStatus -ResourceGroupName "MyResourceGroup" -VMName "myVM"

OsVolumeEncrypted          : EncryptionInProgress
DataVolumesEncrypted       : EncryptionInProgress
OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
ProgressMessage            : Encryption succeeded for data volumes

PS > Get-AzVMDiskEncryptionStatus -ResourceGroupName "MyResourceGroup" -VMName "myVM"

OsVolumeEncrypted          : EncryptionInProgress
DataVolumesEncrypted       : EncryptionInProgress
OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
ProgressMessage            : Provisioning succeeded

PS > Get-AzVMDiskEncryptionStatus -ResourceGroupName "MyResourceGroup" -VMName "myVM"

OsVolumeEncrypted          : EncryptionInProgress
DataVolumesEncrypted       : EncryptionInProgress
OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
ProgressMessage            : OS disk encryption started
```

Для большей части процесса шифрования **ProgressMessage** будет оставаться в **шифровании диска ОС** .  После завершения шифрования и успешного выполнения **ProgressMessage** возвратит:

```azurepowershell
PS > Get-AzVMDiskEncryptionStatus -ResourceGroupName "MyResourceGroup" -VMName "myVM"

OsVolumeEncrypted          : Encrypted
DataVolumesEncrypted       : NotMounted
OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
ProgressMessage            : Encryption succeeded for all volumes
```

Когда это сообщение появится, предполагается, что зашифрованный диск ОС и виртуальная машина готовы к использованию.

Если данные загрузки, сообщение о ходе выполнения или ошибка сообщают о сбое шифрования операционной системы в ходе этого процесса, восстановите виртуальную машину до моментального снимка или резервной копии, созданной непосредственно перед шифрованием. Примером такого сообщения является ошибка сбоя отключения, описанная в этом руководстве.

Перед повторной попытки шифрования повторно оцените характеристики виртуальной машины и убедитесь, что выполнены все необходимые условия.

## <a name="troubleshooting-azure-disk-encryption-behind-a-firewall"></a>Устранение неполадок шифрования диска Azure в связи с брандмауэром

См. раздел [Шифрование дисков в изолированной сети](disk-encryption-isolated-network.md) .

## <a name="troubleshooting-encryption-status"></a>Устранение неполадок с состоянием шифрования 

Портал может отображать диск как зашифрованный даже после того, как он был расшифрован в виртуальной машине.  Это может произойти, если команды низкого уровня используются для непосредственного дешифрования диска в виртуальной машине вместо использования команд управления шифрованием дисков Azure более высокого уровня.  Команды более высокого уровня не только расшифровывают диск в виртуальной машине, но и за пределами виртуальной машины обновляют важные параметры шифрования уровня платформы и параметры расширения, связанные с виртуальной машиной.  Если не поддерживать их согласованность, платформа не сможет сообщить о состоянии шифрования или правильно подготовить к работе виртуальную машину.

Чтобы отключить шифрование дисков Azure с помощью PowerShell, используйте [Disable-AzVMDiskEncryption](/powershell/module/az.compute/disable-azvmdiskencryption), а затем [Remove-AzVMDiskEncryptionExtension](/powershell/module/az.compute/remove-azvmdiskencryptionextension). Выполнение команды Remove-AzVMDiskEncryptionExtension до отключения шифрования невозможно.

Чтобы отключить шифрование дисков Azure с помощью интерфейса командной строки, используйте [az vm encryption disable](/cli/azure/vm/encryption).

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали о некоторых распространенных проблемах в шифровании дисков Azure и их устранении. Дополнительные сведения об этой службе и ее возможностях см. в статьях:

- [Шифрование диска в центре безопасности Azure](../../security-center/asset-inventory.md)
- [Шифрование неактивных данных в Azure](../../security/fundamentals/encryption-atrest.md)