---
title: Отправка виртуального жесткого диска в Azure или копирование диска по регионам — Azure CLI
description: Узнайте, как передать VHD на управляемый диск Azure и скопировать управляемый диск в регионах, используя Azure CLI с помощью прямой отправки.
services: virtual-machines,storage
author: roygara
ms.author: rogarana
ms.date: 06/15/2020
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: disks
ms.openlocfilehash: 473e87904742395eca6b7eeba0875cd93789104d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91978991"
---
# <a name="upload-a-vhd-to-azure-or-copy-a-managed-disk-to-another-region---azure-cli"></a>Отправка VHD в Azure или копирование управляемого диска в другой регион — Azure CLI

[!INCLUDE [disks-upload-vhd-to-disk-intro](../../../includes/disks-upload-vhd-to-disk-intro.md)]

## <a name="prerequisites"></a>Предварительные условия

- Скачайте последнюю [версию AzCopy V10](../../storage/common/storage-use-azcopy-v10.md#download-and-install-azcopy).
- [Установка Azure CLI](/cli/azure/install-azure-cli).
- Если вы планируете передать виртуальный жесткий диск из локальной среды, выполните предварительную загрузку виртуального жесткого диска с фиксированным размером, который [был подготовлен для Azure](../windows/prepare-for-upload-vhd-image.md)и хранится локально.
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

Перед созданием пустого стандартного жесткого диска для отправки вам потребуется размер файла виртуального жесткого диска, который требуется передать, в байтах. Чтобы получить это значение, можно использовать `wc -c <yourFileName>.vhd` или `ls -al <yourFileName>.vhd` . Это значение используется при указании параметра **--upload-size-bytes** .

Создайте пустой жесткий диск "Стандартный" для отправки, указав оба параметра **—-for-upload** и **--upload-size-bytes** в командлете [создания диска](/cli/azure/disk#az-disk-create) .

Замените `<yourdiskname>` , `<yourresourcegroupname>` `<yourregion>` со значениями, выбранными вами. `--upload-size-bytes`Параметр содержит пример значения `34359738880` , заменяя его соответствующим значением.

> [!TIP]
> Если вы создаете диск ОС, добавьте в него параметр--Hyper-v-Generation <yourGeneration> `az disk create` .

```azurecli
az disk create -n <yourdiskname> -g <yourresourcegroupname> -l <yourregion> --for-upload --upload-size-bytes 34359738880 --sku standard_lrs
```

Если вы хотите передать либо твердотельный SSD класса Premium, либо стандартный SSD, замените **standard_lrs** на **premium_LRS** или **standardssd_lrs**. Сейчас поддержка Ultra Disks не поддерживается.

Теперь, когда вы создали пустой управляемый диск, настроенный для процесса отправки, вы можете передать в него VHD. Чтобы отправить VHD на диск, вам потребуется доступный для записи SAS, чтобы можно было ссылаться на него как на место назначения для отправки.

Чтобы создать SAS с возможностью записи для пустого управляемого диска, замените `<yourdiskname>` и `<yourresourcegroupname>` , а затем используйте следующую команду:

```azurecli
az disk grant-access -n <yourdiskname> -g <yourresourcegroupname> --access-level Write --duration-in-seconds 86400
```

Пример возвращаемого значения:

```output
{
  "accessSas": "https://md-impexp-t0rdsfgsdfg4.blob.core.windows.net/w2c3mj0ksfgl/abcd?sv=2017-04-17&sr=b&si=600a9281-d39e-4cc3-91d2-923c4a696537&sig=xXaT6mFgf139ycT87CADyFxb%2BnPXBElYirYRlbnJZbs%3D"
}
```

## <a name="upload-a-vhd"></a>Отправка VHD

Теперь, когда у вас есть SAS для пустого управляемого диска, его можно использовать для задания управляемого диска в качестве места назначения для команды отправки.

Используйте AzCopy V10 для передачи локального VHD-файла на управляемый диск, указав созданный URI SAS.

Эта передача имеет ту же пропускную способность, что и эквивалент [стандартного жесткого диска](../disks-types.md#standard-hdd). Например, если размер равен S4, у вас будет пропускная способность до 60 MiB/с. Но если у вас есть размер, который соответствует S70, вы получите пропускную способность до 500 MiB/с.

```bash
AzCopy.exe copy "c:\somewhere\mydisk.vhd" "sas-URI" --blob-type PageBlob
```

После завершения передачи вам больше не нужно писать какие-либо данные на диск, отозвать SAS. Отзыв SAS изменит состояние управляемого диска и позволит подключить диск к виртуальной машине.

Замените `<yourdiskname>` и `<yourresourcegroupname>` , а затем используйте следующую команду, чтобы сделать диск пригодным для использования:

```azurecli
az disk revoke-access -n <yourdiskname> -g <yourresourcegroupname>
```

## <a name="copy-a-managed-disk"></a>Копирование управляемого диска

Прямая загрузка также упрощает процесс копирования управляемого диска. Можно скопировать в один и тот же регион или между регионами (в другой регион).

Следующий сценарий сделает это для вас, процесс аналогичен описанному ранее действиям с некоторыми отличиями, которые возникают после работы с существующим диском.

> [!IMPORTANT]
> При предоставлении размера диска в байтах управляемого диска из Azure необходимо добавить смещение 512. Это обусловлено тем, что при возврате диска в Azure нижний колонтитул опускается. Если этого не сделать, копирование завершится ошибкой. Следующий сценарий уже выполняет это.

Замените `<sourceResourceGroupHere>` , `<sourceDiskNameHere>` , `<targetDiskNameHere>` , `<targetResourceGroupHere>` и `<yourTargetLocationHere>` (примером значения Location будет uswest2) со своими значениями, а затем выполните следующий сценарий, чтобы скопировать управляемый диск.

> [!TIP]
> Если вы создаете диск ОС, добавьте в него параметр--Hyper-v-Generation <yourGeneration> `az disk create` .

```azurecli
sourceDiskName=<sourceDiskNameHere>
sourceRG=<sourceResourceGroupHere>
targetDiskName=<targetDiskNameHere>
targetRG=<targetResourceGroupHere>
targetLocation=<yourTargetLocationHere>

sourceDiskSizeBytes=$(az disk show -g $sourceRG -n $sourceDiskName --query '[diskSizeBytes]' -o tsv)

az disk create -g $targetRG -n $targetDiskName -l $targetLocation --for-upload --upload-size-bytes $(($sourceDiskSizeBytes+512)) --sku standard_lrs

targetSASURI=$(az disk grant-access -n $targetDiskName -g $targetRG  --access-level Write --duration-in-seconds 86400 -o tsv)

sourceSASURI=$(az disk grant-access -n $sourceDiskName -g $sourceRG --duration-in-seconds 86400 --query [accessSas] -o tsv)

.\azcopy copy $sourceSASURI $targetSASURI --blob-type PageBlob

az disk revoke-access -n $sourceDiskName -g $sourceRG

az disk revoke-access -n $targetDiskName -g $targetRG
```

## <a name="next-steps"></a>Дальнейшие действия

После успешной отправки виртуального жесткого диска на управляемый диск можно подключить его как [диск данных к существующей виртуальной машине](add-disk.md) или [подключить диск к виртуальной машине в качестве диска ОС](upload-vhd.md#create-the-vm), чтобы создать новую виртуальную машину.