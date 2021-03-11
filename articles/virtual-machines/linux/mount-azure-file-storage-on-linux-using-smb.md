---
title: Подключение хранилища файлов Azure на виртуальных машинах Linux с помощью протокола SMB
description: Как подключить хранилище файлов Azure на виртуальных машинах Linux, используя протокол SMB и Azure CLI
author: cynthn
ms.service: virtual-machines
ms.collection: linux
ms.topic: how-to
ms.workload: infrastructure
ms.date: 06/28/2018
ms.author: cynthn
ms.openlocfilehash: 45042f474f39b3f64d45913905765016fae3cb26
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102551799"
---
# <a name="mount-azure-file-storage-on-linux-vms-using-smb"></a>Подключение хранилища файлов Azure на виртуальных машинах Linux с помощью протокола SMB

В этой статье описано, как работать со службой хранилища файлов Azure на виртуальной машине Linux, используя SMB-подключение и Azure CLI. Хранилище файлов Azure предоставляет общие папки в облаке с доступом по стандартному протоколу SMB. 

Хранилище файлов предоставляет общие папки в облаке с доступом по стандартному протоколу SMB. Вы можете подключить общую папку из любой ОС, которая поддерживает протокол SMB 3.0. Использование подключения SMB в Linux обеспечивает простое резервное копирование в надежное постоянное место хранения для архивации, которое поддерживается Соглашением об уровне обслуживания.

Перемещение файлов с виртуальной машины в точку подключения SMB, размещенную в хранилище файлов — отличный способ отладки с использованием журналов. Этот же общий ресурс SMB можно подключить локально к рабочей станции с ОС Mac, Linux или Windows. SMB — не лучшее решение для потоковой передачи журналов Linux или журналов приложений в режиме реального времени, так этот протокол не предназначен для такого интенсивного ведения журнала. Вместо SMB для сбора выходных данных журналов Linux и приложений лучше использовать специальный единый инструмент, работающий на уровне ведения журналов, например Fluentd.

Для работы с этим руководством требуется Azure CLI версии 2.0.4 или более поздней. Чтобы узнать версию, выполните команду **az --version**. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI](/cli/azure/install-azure-cli). 


## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов с именем *myResourceGroup* в расположении *Восточная часть США*.

```azurecli
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-storage-account"></a>Создание учетной записи хранения

Создайте учетную запись хранения в созданной вами группе ресурсов с помощью команды [az storage account create](/cli/azure/storage/account). В этом примере создается учетная запись хранения с именем *mySTORAGEACCT\<random number>*, которое добавляется в переменную **STORAGEACCT**. Имена учетных записей хранения должны быть уникальными. Для этого используйте команду `$RANDOM`, которая добавляет к нему номер.

```azurecli
STORAGEACCT=$(az storage account create \
    --resource-group "myResourceGroup" \
    --name "mystorageacct$RANDOM" \
    --location eastus \
    --sku Standard_LRS \
    --query "name" | tr -d '"')
```

## <a name="get-the-storage-key"></a>Получение ключа хранилища

При создании учетной записи хранения ключи учетной записи создаются парами, поэтому их можно менять без прерывания работы службы. При смене одного из ключей в паре создается еще одна пара ключей. Так как новые ключи учетной записи хранения всегда создаются парами, у вас всегда будет по крайней мере один неиспользованный ключ учетной записи хранения, на который можно сменить текущий ключ.

Ключи учетной записи хранения можно просмотреть, выполнив команду [az storage account keys list](/cli/azure/storage/account/keys). В этом примере значение ключа 1 хранится в переменной **STORAGEKEY**.

```azurecli
STORAGEKEY=$(az storage account keys list \
    --resource-group "myResourceGroup" \
    --account-name $STORAGEACCT \
    --query "[0].value" | tr -d '"')
```

## <a name="create-a-file-share"></a>Создание общей папки

Создайте общий ресурс хранилища файлов с помощью команды [az storage share create](/cli/azure/storage/share). 

Имена общих ресурсов должны содержать только строчные буквы, цифры и отдельные дефисы. Также они не могут начинаться с дефиса. Дополнительные сведения о присвоении имен общим папкам и файлам см. в статье [Именование общих ресурсов, каталогов, файлов и метаданных и ссылка на них](/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata).

В этом примере создается файловый ресурс с именем *myshare* и квотой на 10 ГиБ. 

```azurecli
az storage share create --name myshare \
    --quota 10 \
    --account-name $STORAGEACCT \
    --account-key $STORAGEKEY
```

## <a name="create-a-mount-point"></a>Создание точки подключения

Чтобы подключить файловый ресурс Azure на компьютере под управлением Linux, необходимо убедиться, что у вас установлен пакет **cifs-utils**. Инструкции см. в руководстве по [установке пакета cifs-utils для вашего дистрибутива Linux](../../storage/files/storage-how-to-use-files-linux.md#install-cifs-utils).

Служба файлов Azure использует протокол SMB, который обменивается данными через TCP-порт 445.  Если вам не удается подключить файловый ресурс Azure, убедитесь, что брандмауэр не блокирует TCP-порт 445.


```bash
mkdir -p /mnt/MyAzureFileShare
```

## <a name="mount-the-share"></a>Подключение общего ресурса

Подключите общий ресурс Azure к локальному каталогу. 

```bash
sudo mount -t cifs //$STORAGEACCT.file.core.windows.net/myshare /mnt/MyAzureFileShare -o vers=3.0,username=$STORAGEACCT,password=$STORAGEKEY,dir_mode=0777,file_mode=0777,serverino
```

Приведенная выше команда используется команду [mount](https://linux.die.net/man/8/mount), чтобы подключить общий файловый ресурс Azure и параметры, относящиеся к [cifs](https://linux.die.net/man/8/mount.cifs). Говоря конкретнее, параметры dir_mode и file_mode позволяют установить для каталогов и файлов разрешение `0777`. В рамках разрешения `0777` все пользователи получают доступ на чтение, запись и выполнение. Вы можете изменить эти разрешения. Для этого замените значения другими [разрешениями chmod](https://en.wikipedia.org/wiki/Chmod). Можно также использовать другие параметры [cifs](https://linux.die.net/man/8/mount.cifs), например gid или uid. 


## <a name="persist-the-mount"></a>Сохранение подключения

При перезагрузке виртуальной машины Linux подключенный общий ресурс SMB отключается во время завершения работы. Чтобы общий ресурс SMB повторно подключался при загрузке, добавьте строку в файл Linux /etc/fstab. Linux использует файл fstab, чтобы получить список файловых систем, которые следует подключить во время загрузки. Если добавить общий ресурс SMB, общая папка хранилища файлов станет постоянно подключенной файловой системой в виртуальной машине Linux. Общий ресурс SMB в хранилище файлов можно добавить в новую виртуальную машину, используя cloud-init.

```bash
//myaccountname.file.core.windows.net/mystorageshare /mnt/mymountpoint cifs vers=3.0,username=mystorageaccount,password=myStorageAccountKeyEndingIn==,dir_mode=0777,file_mode=0777
```

Для обеспечения безопасности в рабочих средах следует хранить учетные данные за пределами fstab.

## <a name="next-steps"></a>Дальнейшие действия

- [Настройка виртуальной машины Linux во время создания с помощь cloud-init](using-cloud-init.md)
- [Добавление диска к виртуальной машине Linux](add-disk.md)
- [Шифрование дисков Azure для виртуальных машин под управлением Linux](disk-encryption-overview.md)
