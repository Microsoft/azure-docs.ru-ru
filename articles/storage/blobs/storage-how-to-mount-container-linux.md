---
title: Как подключить хранилище BLOB-объектов Azure в качестве файловой системы в Linux | Документация Майкрософт
description: Узнайте, как подключить контейнер хранилища BLOB-объектов Azure с помощью blobfuse, драйвера виртуальной файловой системы в Linux.
author: tamram
ms.service: storage
ms.subservice: blobs
ms.topic: how-to
ms.date: 2/1/2019
ms.author: tamram
ms.reviewer: twooley
ms.openlocfilehash: 002e8650a5555b70caf09179e03ce1bad1acdef5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98737546"
---
# <a name="how-to-mount-blob-storage-as-a-file-system-with-blobfuse"></a>Как подключить хранилище BLOB-объектов в качестве файловой системы с использованием blobfuse

## <a name="overview"></a>Обзор
[blobfuse](https://github.com/Azure/azure-storage-fuse) — это виртуальный драйвер файловой системы для хранилища BLOB-объектов Azure. blobfuse позволяет получить доступ к имеющимся данным блочного BLOB-объекта в учетной записи хранения через файловую систему Linux. Blobfuse использует схему виртуального каталога с косой чертой "/" в качестве разделителя.  

В этом руководстве показано, как использовать blobfuse, подключить контейнер хранилища BLOB-объектов в Linux, а также получить доступ к данным. Дополнительные сведения о blobfuse см. в [репозитории blobfuse](https://github.com/Azure/azure-storage-fuse).

> [!WARNING]
> blobfuse не гарантирует полную совместимость с POSIX, он просто преобразовывает запросы в [REST API больших двоичных объектов](/rest/api/storageservices/blob-service-rest-api). Например, операции переименования в POSIX являются атомарными, а в blobfuse — нет.
> Полный список различий между собственной файловой системой и blobfuse см. в [репозитории исходного кода blobfuse](https://github.com/azure/azure-storage-fuse).
> 

## <a name="install-blobfuse-on-linux"></a>Установка blobfuse в Linux
Двоичные файлы Blobfuse доступны в [репозиториях программного обеспечения Майкрософт для Linux](/windows-server/administration/Linux-Package-Repository-for-Microsoft-Software) для Ubuntu, Debian, SUSE, центус, Oracle Linux и RHEL дистрибутивов. Чтобы установить blobfuse в этих дистрибутивах, настройте один из репозиториев из списка. Вы также можете создавать двоичные файлы из исходного кода, следуя [действиям по установке службы хранилища Azure](https://github.com/Azure/azure-storage-fuse/wiki/1.-Installation#option-2---build-from-source), если для вашего дистрибутива нет двоичных файлов.

Blobfuse поддерживает установку в версиях Ubuntu: 16,04, 18,04 и 20,04, Рхелверсионс: 7,5, 7,8, 8,0, 8,1, 8,2, CentOS версии: 7,0, 8,0, Debian версии: 9,0, 10,0, SUSE Version: 15, Ораклелинукс 8,1. Выполните следующую команду, чтобы убедиться в том, что у вас развернута одна из этих версий:
```
lsb_release -a
```

### <a name="configure-the-microsoft-package-repository"></a>Настройка репозитория пакетов Майкрософт
Настройте [репозиторий пакетов Linux для продуктов Майкрософт](/windows-server/administration/Linux-Package-Repository-for-Microsoft-Software).

Например, в дистрибутиве Enterprise Linux 8:
```bash
sudo rpm -Uvh https://packages.microsoft.com/config/rhel/8/packages-microsoft-prod.rpm
```

Аналогичным образом измените URL-адрес на `.../rhel/7/...`, чтобы указать дистрибутив Enterprise Linux 7.

Еще один пример в дистрибутиве Ubuntu 20,04:
```bash
wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get update
```

Аналогичным образом измените URL-адрес на `.../ubuntu/16.04/...` или `.../ubuntu/18.04/...` для ссылки на другую версию Ubuntu.

### <a name="install-blobfuse"></a>Установка blobfuse

В дистрибутиве Ubuntu/Debian:
```bash
sudo apt-get install blobfuse
```

В дистрибутиве Enterprise Linux:
```bash    
sudo yum install blobfuse
```

## <a name="prepare-for-mounting"></a>Подготовка к подключению
Для blobfuse требуется временный путь в файловой системе, чтобы поместить все открытые файлы в буфер и кэшировать их, что помогает обеспечить производительность, близкую к исходной. Для этого временного пути выберите самый высокопроизводительный диск или используйте электронный диск для достижения оптимальной производительности. 

> [!NOTE]
> blobfuse хранит все содержимое открытых файлов во временном пути. Убедитесь, что имеется достаточно места для всех открытых файлов. 
> 

### <a name="optional-use-a-ramdisk-for-the-temporary-path"></a>Использование электронного диска для временного пути (необязательно)
В следующем примере создается электронный диск на 16 ГБ и каталог для blobfuse. Выберите размер в соответствии со своими потребностями. Такой электронный диск позволяет blobfuse открывать файлы размером до 16 ГБ. 
```bash
sudo mkdir /mnt/ramdisk
sudo mount -t tmpfs -o size=16g tmpfs /mnt/ramdisk
sudo mkdir /mnt/ramdisk/blobfusetmp
sudo chown <youruser> /mnt/ramdisk/blobfusetmp
```

### <a name="use-an-ssd-as-a-temporary-path"></a>Использование диска SSD в качестве временного пути
В Azure можно использовать временные диски (SSD), доступные на виртуальной машине, чтобы обеспечить для blobfuse буфер с малой задержкой. В дистрибутивах Ubuntu этот временный диск подключен к расположению "/mnt". В дистрибутивах CentOS и Red Hat диск подключен к расположению "/mnt/resource/".

Для пользователя нужно настроить права доступа к временному пути:
```bash
sudo mkdir /mnt/resource/blobfusetmp -p
sudo chown <youruser> /mnt/resource/blobfusetmp
```

### <a name="configure-your-storage-account-credentials"></a>Настройка учетных данных учетной записи хранения
Для blobfuse требуется, чтобы учетные данные хранились в текстовом файле такого формата: 

```
accountName myaccount
accountKey storageaccesskey
containerName mycontainer
```
`accountName`— Это префикс для учетной записи хранения, а не полный URL-адрес.

Создать этот файл с помощью:

```
touch ~/fuse_connection.cfg
```

После создания и изменения этого файла необходимо ограничить доступ, чтобы другие пользователи не могли его читать.
```bash
chmod 600 fuse_connection.cfg
```

> [!NOTE]
> Если вы создали файл конфигурации в Windows, обязательно запустите `dos2unix`, чтобы очистить данные и преобразовать файл в формат Unix. 
>

### <a name="create-an-empty-directory-for-mounting"></a>Создание пустого каталога для подключения
```bash
mkdir ~/mycontainer
```

## <a name="mount"></a>Подключение

> [!NOTE]
> Полный список параметров подключения см. в [репозитории blobfuse](https://github.com/Azure/azure-storage-fuse#mount-options).  
> 

Чтобы подключить blobfuse, выполните следующую команду с использованием нужного профиля пользователя. Эта команда подключает контейнер, указанный в "/path/to/fuse_connection.cfg", к расположению "/mycontainer".

```bash
sudo blobfuse ~/mycontainer --tmp-path=/mnt/resource/blobfusetmp  --config-file=/path/to/fuse_connection.cfg -o attr_timeout=240 -o entry_timeout=240 -o negative_timeout=120
```

Теперь у вас должен быть доступ к блочным BLOB-объектам через обычные API файловой системы. Только тот пользователь, который подключает каталог, может пользоваться им по умолчанию, что обеспечивает безопасный доступ. Чтобы разрешить доступ всем пользователям, можно выполнить подключение через параметр ```-o allow_other```. 

```bash
cd ~/mycontainer
mkdir test
echo "hello world" > test/blob.txt
```

## <a name="next-steps"></a>Дальнейшие действия

* [Домашняя страница blobfuse](https://github.com/Azure/azure-storage-fuse#blobfuse)
* [Сообщить о проблемах с blobfuse](https://github.com/Azure/azure-storage-fuse/issues)
