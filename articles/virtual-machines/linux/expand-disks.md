---
title: Расширение виртуальных жестких дисков на виртуальной машине Linux
description: Узнайте, как расширить виртуальные жесткие диски на виртуальной машине Linux с помощью Azure CLI.
author: roygara
ms.service: virtual-machines
ms.collection: linux
ms.topic: how-to
ms.date: 10/15/2018
ms.author: rogarana
ms.subservice: disks
ms.openlocfilehash: 72778c431c561f5345dde3d6803e814d6fdebfba
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102549130"
---
# <a name="expand-virtual-hard-disks-on-a-linux-vm-with-the-azure-cli"></a>Расширение виртуальных жестких дисков на виртуальной машине Linux с помощью Azure CLI

В этой статье подробно описано, как расширить управляемые диски для виртуальной машины Linux с помощью Azure CLI. Вы можете [добавить диски данных](add-disk.md), чтобы предоставить дополнительное место для хранения, а также расширить существующий диск данных. Как правило, размер виртуального жесткого диска по умолчанию для операционной системы на виртуальной машине Linux в Azure составляет 30 ГБ. 

> [!WARNING]
> Всегда убедитесь, что ваша файловая система находится в рабочем состоянии, тип таблицы разделов диска будет поддерживать новый размер и перед выполнением операций по изменению размера диска необходимо обеспечить резервное копирование данных. Дополнительные сведения см. в [кратком руководстве по Azure Backup](../../backup/quick-backup-vm-portal.md). 

## <a name="expand-an-azure-managed-disk"></a>Развертывание управляемого диска Azure
Обязательно установите последнюю версию [Azure CLI](/cli/azure/install-az-cli2) и войдите в учетную запись Azure с помощью команды [az login](/cli/azure/reference-index#az-login).

Для этой статьи потребуется существующая виртуальная машина в Azure с хотя бы одним подключенным и подготовленным диском данных. Если у вас нет виртуальной машины, которую можно использовать, см. раздел [о создании и подготовке виртуальной машины с дисками данных](tutorial-manage-disks.md#create-and-attach-disks).

В следующих примерах замените имена параметров собственными значениями, например *myResourceGroup* и *myVM*.

1. Нельзя выполнять операции с виртуальными жесткими дисками на работающей виртуальной машине. Отмените подготовку виртуальной машины, выполнив команду [az vm deallocate](/cli/azure/vm#az-vm-deallocate). В следующем примере освобождается виртуальная машина с именем *myVM* в группе ресурсов с именем *myResourceGroup*:

    ```azurecli
    az vm deallocate --resource-group myResourceGroup --name myVM
    ```

    > [!NOTE]
    > Для расширения виртуального жесткого диска следует отменить распределение виртуальной машины. Если для остановки виртуальной машины используется операция `az vm stop`, вычислительные ресурсы не высвобождаются. Чтобы освободить вычислительные ресурсы, используйте `az vm deallocate`.

1. Просмотрите список управляемых дисков в группе ресурсов, выполнив команду [az disk list](/cli/azure/disk#az-disk-list). В следующем примере выводится список управляемых дисков, входящих в группу ресурсов с именем *myResourceGroup*:

    ```azurecli
    az disk list \
        --resource-group myResourceGroup \
        --query '[*].{Name:name,Gb:diskSizeGb,Tier:accountType}' \
        --output table
    ```

    Расширьте диск, выполнив команду [az disk update](/cli/azure/disk#az-disk-update). В следующем примере размер управляемого диска *myDataDisk* увеличивается до *200* ГБ:

    ```azurecli
    az disk update \
        --resource-group myResourceGroup \
        --name myDataDisk \
        --size-gb 200
    ```

    > [!NOTE]
    > При расширении управляемого диска его новый размер округляется до ближайшего размера управляемого диска. Таблица с размерами и ценовыми категориями доступных управляемых дисков доступна в разделе [Цены и выставление счетов](../managed-disks-overview.md).

1. Запустите виртуальную машину, выполнив команду [az vm start](/cli/azure/vm#az-vm-start). В следующем примере запускается виртуальная машина *myVM*, входящая в группу ресурсов с именем *myResourceGroup*.

    ```azurecli
    az vm start --resource-group myResourceGroup --name myVM
    ```


## <a name="expand-a-disk-partition-and-filesystem"></a>Развертывание раздела диска и файловой системы
Чтобы использовать развернутый диск, необходимо развернуть основной раздел и файловую систему.

1. Подключитесь к виртуальной машине по протоколу SSH, используя соответствующие учетные данные. Вы можете просмотреть общедоступный IP-адрес виртуальной машины с помощью команды [az vm show](/cli/azure/vm#az-vm-show):

    ```azurecli
    az vm show --resource-group myResourceGroup --name myVM -d --query [publicIps] --output tsv
    ```

1. Развертывание основного раздела и файловой системы.

    а. Если диск подключен, отключите его:

    ```bash
    sudo umount /dev/sdc1
    ```

    b. Чтобы просмотреть сведения о диске и изменить размер раздела, используйте команду `parted`.

    ```bash
    sudo parted /dev/sdc
    ```

    Просмотрите сведения о структуре существующего раздела с помощью команды `print`. В результате будут выведены выходные данные, как в примере ниже, указывающие, что размер основного диска составляет 215 ГБ:

    ```bash
    GNU Parted 3.2
    Using /dev/sdc1
    Welcome to GNU Parted! Type 'help' to view a list of commands.
    (parted) print
    Model: Unknown Msft Virtual Disk (scsi)
    Disk /dev/sdc1: 215GB
    Sector size (logical/physical): 512B/4096B
    Partition Table: loop
    Disk Flags:
    
    Number  Start  End    Size   File system  Flags
        1      0.00B  107GB  107GB  ext4
    ```

    c. Разверните раздел с помощью команды `resizepart`. Введите номер (*1*) и размер для нового раздела.

    ```bash
    (parted) resizepart
    Partition number? 1
    End?  [107GB]? 215GB
    ```

    d. Чтобы закрыть командную строку, введите `quit`.

1. Изменив размер раздела, проверьте его целостность с помощью команды `e2fsck`.

    ```bash
    sudo e2fsck -f /dev/sdc1
    ```

1. Измените размер файловой системы, выполнив команду `resize2fs`:

    ```bash
    sudo resize2fs /dev/sdc1
    ```

1. Подключите раздел к необходимому расположению, например `/datadrive`.

    ```bash
    sudo mount /dev/sdc1 /datadrive
    ```

1. Чтобы проверить, был ли изменен размер диска данных, используйте `df -h` . В следующем примере выходных данных показано, что теперь размер основного раздела (*/dev/sdc1*) составляет 200 ГБ:

    ```bash
    Filesystem      Size   Used  Avail Use% Mounted on
    /dev/sdc1        197G   60M   187G   1% /datadrive
    ```

## <a name="next-steps"></a>Дальнейшие действия
* Если вам требуется дополнительное место для хранения, можно также [добавить диски данных в виртуальную машину Linux](add-disk.md). 
* Дополнительные сведения о шифровании дисков см. в статье [Шифрование дисков Azure для виртуальных машин Linux](disk-encryption-overview.md).
