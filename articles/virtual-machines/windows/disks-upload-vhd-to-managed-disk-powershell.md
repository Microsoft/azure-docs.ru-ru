---
title: Отправка виртуального жесткого диска в Azure или копирование диска по регионам — Azure PowerShell
description: Узнайте, как передать VHD на управляемый диск Azure и скопировать управляемый диск в регионах с помощью Azure PowerShell, используя прямую передачу.
author: roygara
ms.author: rogarana
ms.date: 06/15/2020
ms.topic: how-to
ms.service: virtual-machines
ms.tgt_pltfrm: linux
ms.subservice: disks
ms.openlocfilehash: c766c78705a1c1e40a9385360d35ac06a3db3a5d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99252244"
---
# <a name="upload-a-vhd-to-azure-or-copy-a-managed-disk-to-another-region---azure-powershell"></a>Отправка VHD в Azure или копирование управляемого диска в другой регион — Azure PowerShell

[!INCLUDE [disks-upload-vhd-to-disk-intro](../../../includes/disks-upload-vhd-to-disk-intro.md)]

## <a name="prerequisites"></a>Предварительные условия

- Скачайте последнюю [версию AzCopy V10](../../storage/common/storage-use-azcopy-v10.md#download-and-install-azcopy).
- [Установите модуль Azure PowerShell](/powershell/azure/install-Az-ps).
- Если вы планируете передать виртуальный жесткий диск из локальной среды, выполните предварительную загрузку виртуального жесткого диска с фиксированным размером, который [был подготовлен для Azure](prepare-for-upload-vhd-image.md)и хранится локально.
- Или управляемый диск в Azure, если планируется выполнить действие копирования.

## <a name="getting-started"></a>Начало работы

Если вы предпочитаете отправлять диски через графический интерфейс, это можно сделать с помощью Обозреватель службы хранилища Azure. Дополнительные сведения см. в статье [использование обозреватель службы хранилища Azure для управления управляемыми дисками Azure](../disks-use-storage-explorer-managed-disks.md) .

Чтобы отправить VHD в Azure, необходимо создать пустой управляемый диск, настроенный для этого процесса отправки. Прежде чем создать его, необходимо ознакомиться с дополнительными сведениями об этих дисках.

Этот тип управляемого диска имеет два уникальных состояния:

- Реадтауплоад. Это означает, что диск готов к получению отправки, но [подпись безопасного доступа](../../storage/common/storage-sas-overview.md) (SAS) не была создана.
- Активеуплоад. Это означает, что диск готов к получению отправки, и создан SAS.

> [!NOTE]
> Хотя в одном из этих состояний плата за управляемый диск будет взиматься по [стандартному тарифу на HDD](https://azure.microsoft.com/pricing/details/managed-disks/), независимо от фактического типа диска. Например, P10 будет оплачиваться как S10. Это справедливо до тех пор `revoke-access` , пока на управляемом диске не будет вызван, что необходимо для подключения диска к виртуальной машине.

## <a name="create-an-empty-managed-disk"></a>Создание пустого управляемого диска

Перед созданием пустого стандартного жесткого диска для отправки вам потребуется размер файла виртуального жесткого диска, который требуется передать, в байтах. Пример кода получит это, но для самостоятельного использования можно использовать: `$vhdSizeBytes = (Get-Item "<fullFilePathHere>").length` . Это значение используется при указании параметра **-уплоадсизеинбитес** .

Теперь в локальной оболочке создайте пустой жесткий диск "Стандартный" для отправки, указав параметр **отправки** в параметре **-CreateOption** , а также параметр **-уплоадсизеинбитес** в командлете [New-аздискконфиг](/powershell/module/az.compute/new-azdiskconfig) . Затем вызовите [New-аздиск](/powershell/module/az.compute/new-azdisk) , чтобы создать диск.

Замените `<yourdiskname>` , `<yourresourcegroupname>` и `<yourregion>` выполните следующие команды:

> [!TIP]
> При создании диска ОС добавьте `-HyperVGeneration '<yourGeneration>'` в `New-AzDiskConfig` .

```powershell
$vhdSizeBytes = (Get-Item "<fullFilePathHere>").length

$diskconfig = New-AzDiskConfig -SkuName 'Standard_LRS' -OsType 'Windows' -UploadSizeInBytes $vhdSizeBytes -Location '<yourregion>' -CreateOption 'Upload'

New-AzDisk -ResourceGroupName '<yourresourcegroupname>' -DiskName '<yourdiskname>' -Disk $diskconfig
```

Если вы хотите передать либо твердотельный SSD класса Premium, либо стандартный SSD, замените **Standard_LRS** на **Premium_LRS** или **StandardSSD_LRS**. Ultra Disks пока не поддерживаются.

Теперь, когда вы создали пустой управляемый диск, настроенный для процесса отправки, вы можете передать в него VHD. Чтобы отправить VHD на диск, вам потребуется доступный для записи SAS, чтобы можно было ссылаться на него как на место назначения для отправки.

Чтобы создать SAS с возможностью записи для пустого управляемого диска, замените `<yourdiskname>` и `<yourresourcegroupname>` , а затем используйте следующие команды:

```powershell
$diskSas = Grant-AzDiskAccess -ResourceGroupName '<yourresourcegroupname>' -DiskName '<yourdiskname>' -DurationInSecond 86400 -Access 'Write'

$disk = Get-AzDisk -ResourceGroupName '<yourresourcegroupname>' -DiskName '<yourdiskname>'
```

## <a name="upload-a-vhd"></a>Отправка VHD

Теперь, когда у вас есть SAS для пустого управляемого диска, его можно использовать для задания управляемого диска в качестве места назначения для команды отправки.

Используйте AzCopy V10 для передачи локального VHD-файла на управляемый диск, указав созданный URI SAS.

Эта передача имеет ту же пропускную способность, что и эквивалент [стандартного жесткого диска](../disks-types.md#standard-hdd). Например, если размер равен S4, у вас будет пропускная способность до 60 MiB/с. Но если у вас есть размер, который соответствует S70, вы получите пропускную способность до 500 MiB/с.

```
AzCopy.exe copy "c:\somewhere\mydisk.vhd" $diskSas.AccessSAS --blob-type PageBlob
```

После завершения передачи вам больше не нужно писать какие-либо данные на диск, отозвать SAS. Отзыв SAS изменит состояние управляемого диска и позволит подключить диск к виртуальной машине.

Замените `<yourdiskname>` и `<yourresourcegroupname>` , затем выполните следующую команду:

```powershell
Revoke-AzDiskAccess -ResourceGroupName '<yourresourcegroupname>' -DiskName '<yourdiskname>'
```

## <a name="copy-a-managed-disk"></a>Копирование управляемого диска

Прямая загрузка также упрощает процесс копирования управляемого диска. Можно либо скопировать в один регион, либо скопировать управляемый диск в другой регион.

Следующий сценарий сделает это за вас, процесс аналогичен описанному ранее действиям с некоторыми отличиями, так как вы работаете с существующим диском.

> [!IMPORTANT]
> При предоставлении размера диска в байтах управляемого диска из Azure необходимо добавить смещение 512. Это обусловлено тем, что при возврате диска в Azure нижний колонтитул опускается. Если этого не сделать, копирование завершится ошибкой. Следующий сценарий уже выполняет это.

Замените `<sourceResourceGroupHere>` , `<sourceDiskNameHere>` , `<targetDiskNameHere>` , `<targetResourceGroupHere>` `<yourOSTypeHere>` и `<yourTargetLocationHere>` (примером значения Location будет uswest2) со своими значениями, а затем выполните следующий сценарий, чтобы скопировать управляемый диск.

> [!TIP]
> При создании диска ОС добавьте параметр-Хипервженератион " <yourGeneration> " в `New-AzDiskConfig` .

```powershell

$sourceRG = <sourceResourceGroupHere>
$sourceDiskName = <sourceDiskNameHere>
$targetDiskName = <targetDiskNameHere>
$targetRG = <targetResourceGroupHere>
$targetLocate = <yourTargetLocationHere>
#Expected value for OS is either "Windows" or "Linux"
$targetOS = <yourOSTypeHere>

$sourceDisk = Get-AzDisk -ResourceGroupName $sourceRG -DiskName $sourceDiskName

# Adding the sizeInBytes with the 512 offset, and the -Upload flag
$targetDiskconfig = New-AzDiskConfig -SkuName 'Standard_LRS' -osType $targetOS -UploadSizeInBytes $($sourceDisk.DiskSizeBytes+512) -Location $targetLocate -CreateOption 'Upload'

$targetDisk = New-AzDisk -ResourceGroupName $targetRG -DiskName $targetDiskName -Disk $targetDiskconfig

$sourceDiskSas = Grant-AzDiskAccess -ResourceGroupName $sourceRG -DiskName $sourceDiskName -DurationInSecond 86400 -Access 'Read'

$targetDiskSas = Grant-AzDiskAccess -ResourceGroupName $targetRG -DiskName $targetDiskName -DurationInSecond 86400 -Access 'Write'

azcopy copy $sourceDiskSas.AccessSAS $targetDiskSas.AccessSAS --blob-type PageBlob

Revoke-AzDiskAccess -ResourceGroupName $sourceRG -DiskName $sourceDiskName

Revoke-AzDiskAccess -ResourceGroupName $targetRG -DiskName $targetDiskName 
```

## <a name="next-steps"></a>Дальнейшие действия

После успешной отправки виртуального жесткого диска на управляемый диск можно подключить его к виртуальной машине и начать использовать.

Сведения о подключении диска данных к виртуальной машине см. в статье о [подключении диска данных к виртуальной машине Windows с помощью PowerShell](attach-disk-ps.md). Сведения о том, как использовать диск в качестве диска операционной системы, см. в статье [Создание виртуальной машины Windows на основе специализированного диска](create-vm-specialized.md#create-the-new-vm).
