---
title: Использование Azure CLI для управления данными (Azure Data Lake Storage 2-го поколения)
description: Используйте Azure CLI для управления каталогами и файлами в учетных записях хранения, имеющих иерархическое пространство имен.
services: storage
author: normesta
ms.service: storage
ms.subservice: data-lake-storage-gen2
ms.topic: how-to
ms.date: 02/17/2021
ms.author: normesta
ms.reviewer: prishet
ms.custom: devx-track-azurecli
ms.openlocfilehash: 3e9afd4617eb7445ba83948d46eef0890832e2be
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100650360"
---
# <a name="use-azure-cli-to-manage-directories-and-files-in-azure-data-lake-storage-gen2"></a>Использование Azure CLI для управления каталогами и файлами в Azure Data Lake Storage 2-го поколения

В этой статье показано, как использовать [интерфейс Azure Command-Line](/cli/azure/) для создания каталогов и файлов в учетных записях хранения с иерархическим пространством имен и управления ими.

Дополнительные сведения о получении, установке и обновлении списков управления доступом (ACL) для каталогов и файлов см. в разделе [использование Azure CLI для управления списками ACL в Azure Data Lake Storage 2-го поколения](data-lake-storage-acl-cli.md).

[Примеры](https://github.com/Azure/azure-cli/blob/dev/src/azure-cli/azure/cli/command_modules/storage/docs/ADLS%20Gen2.md)  |  [Отправить отзыв](https://github.com/Azure/azure-cli-extensions/issues)

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).

- Учетная запись хранения с включенным иерархическим пространством имен. Выполните [эти](create-data-lake-storage-account.md) инструкции, чтобы создать учетную запись.

- Azure CLI версии`2.6.0` или выше.

## <a name="ensure-that-you-have-the-correct-version-of-azure-cli-installed"></a>Убедитесь, что у вас установлена правильная версия Azure CLI

1. Откройте [Azure Cloud Shell](../../cloud-shell/overview.md) или, если вы [установили](/cli/azure/install-azure-cli) Azure CLI локально, командное консольное приложение (например, Windows PowerShell).

2. Убедитесь, что установлена версия Azure CLI`2.6.0` или выше, выполнив следующую команду.

   ```azurecli
    az --version
   ```

   Если ваша версия Azure CLI ниже чем `2.6.0`, установите более позднюю версию. Подробнее см. статью [Установка Azure CLI](/cli/azure/install-azure-cli).

## <a name="connect-to-the-account"></a>Подключение к учетной записи

1. Если вы используете Azure CLI локально, выполните команду для входа.

   ```azurecli
   az login
   ```

   Если в CLI можно запустить браузер по умолчанию, откроется браузер со страницей входа.

   В противном случае самостоятельно откройте в браузере страницу [https://aka.ms/devicelogin](https://aka.ms/devicelogin) и введите код авторизации, отображаемый в терминале. Затем выполните вход в браузере с помощью учетных данных.

   Дополнительные сведения о различных методах аутентификации см. на странице [Авторизация доступа к данным BLOB-объектов или очередей с помощью Azure CLI](./authorize-data-operations-cli.md).

2. Если удостоверение связано с несколькими подписками, установите активную подписку в качестве подписки учетной записи хранения, в которой будет размещен статический веб-сайт.

   ```azurecli
   az account set --subscription <subscription-id>
   ```

   Замените значение заполнителя `<subscription-id>` идентификатором своей подписки.

> [!NOTE]
> В примере, приведенном в этой статье, показана авторизация Azure Active Directory (Azure AD). Дополнительные сведения о методах аутентификации см. на странице [Авторизация доступа к данным BLOB-объектов или очередей с помощью Azure CLI](./authorize-data-operations-cli.md).

## <a name="create-a-container"></a>Создание контейнера

Контейнер выступает в качестве файловой системы для файлов. Вы можете создать ее с помощью команды `az storage fs create`. 

В этом примере создается контейнер с именем `my-file-system` .

```azurecli
az storage fs create -n my-file-system --account-name mystorageaccount --auth-mode login
```

## <a name="show-container-properties"></a>Отобразить свойства контейнера

Свойства контейнера можно вывести на консоль с помощью `az storage fs show` команды.

```azurecli
az storage fs show -n my-file-system --account-name mystorageaccount --auth-mode login
```

## <a name="list-container-contents"></a>Список содержимого контейнера

Выведите содержимое каталога с помощью команды `az storage fs file list`.

В этом примере выводится содержимое контейнера с именем `my-file-system` .

```azurecli
az storage fs file list -f my-file-system --account-name mystorageaccount --auth-mode login
```

## <a name="delete-a-container"></a>Удаление контейнера

Удалите контейнер с помощью `az storage fs delete` команды.

В этом примере удаляется контейнер с именем `my-file-system` . 

```azurecli
az storage fs delete -n my-file-system --account-name mystorageaccount --auth-mode login
```

## <a name="create-a-directory"></a>Создание каталога

Создайте ссылку на каталог с помощью команды `az storage fs directory create`. 

В этом примере добавляется каталог с именем `my-directory` в контейнер с именем `my-file-system` , который находится в учетной записи с именем `mystorageaccount` .

```azurecli
az storage fs directory create -n my-directory -f my-file-system --account-name mystorageaccount --auth-mode login
```

## <a name="show-directory-properties"></a>Отображение свойств каталога

Свойства каталога можно вывести на консоль с помощью команды `az storage fs directory show`.

```azurecli
az storage fs directory show -n my-directory -f my-file-system --account-name mystorageaccount --auth-mode login
```

## <a name="rename-or-move-a-directory"></a>Переименование или перемещение каталога

Переименуйте или переместите каталог с помощью команды `az storage fs directory move`.

Этот пример переименовывает каталог из имени в имя `my-directory` `my-new-directory` в том же контейнере.

```azurecli
az storage fs directory move -n my-directory -f my-file-system --new-directory "my-file-system/my-new-directory" --account-name mystorageaccount --auth-mode login
```

В этом примере каталог перемещается в контейнер с именем `my-second-file-system` .

```azurecli
az storage fs directory move -n my-directory -f my-file-system --new-directory "my-second-file-system/my-new-directory" --account-name mystorageaccount --auth-mode login
```

## <a name="delete-a-directory"></a>Удаление каталога

Удалите каталог с помощью команды `az storage fs directory delete`.

В этом примере удаляется каталог `my-directory`. 

```azurecli
az storage fs directory delete -n my-directory -f my-file-system  --account-name mystorageaccount --auth-mode login 
```

## <a name="check-if-a-directory-exists"></a>Проверка наличия каталога

Определите, существует ли определенный каталог в контейнере, с помощью `az storage fs directory exists` команды.

Этот пример показывает, `my-directory` существует ли в контейнере каталог с именем `my-file-system` . 

```azurecli
az storage fs directory exists -n my-directory -f my-file-system --account-name mystorageaccount --auth-mode login 
```

## <a name="download-from-a-directory"></a>Скачивание из каталога

Скачайте файл из каталога, используя команду `az storage fs file download`.

В этом примере из каталога `my-directory` скачивается файл `upload.txt`. 

```azurecli
az storage fs file download -p my-directory/upload.txt -f my-file-system -d "C:\myFolder\download.txt" --account-name mystorageaccount --auth-mode login
```

## <a name="list-directory-contents"></a>Вывод содержимого каталогов

Выведите содержимое каталога с помощью команды `az storage fs file list`.

В этом примере выводится содержимое каталога с именем `my-directory` , расположенного в `my-file-system` контейнере учетной записи хранения с именем `mystorageaccount` . 

```azurecli
az storage fs file list -f my-file-system --path my-directory --account-name mystorageaccount --auth-mode login
```

## <a name="upload-a-file-to-a-directory"></a>Отправка файла в каталог

Отправьте файл в каталог с помощью команды `az storage fs directory upload`.

В этом примере в каталог `my-directory` отправляется файл `upload.txt`. 

```azurecli
az storage fs file upload -s "C:\myFolder\upload.txt" -p my-directory/upload.txt  -f my-file-system --account-name mystorageaccount --auth-mode login
```

## <a name="show-file-properties"></a>Отображение свойств файла

Свойства файла можно вывести на консоль с помощью команды `az storage fs file show`.

```azurecli
az storage fs file show -p my-file.txt -f my-file-system --account-name mystorageaccount --auth-mode login
```

## <a name="rename-or-move-a-file"></a>Переименование или перемещение файла

Переименуйте или переместите файл с помощью команды `az storage fs file move`.

В этом примере файл `my-file.txt` переименовывается на `my-file-renamed.txt`.

```azurecli
az storage fs file move -p my-file.txt -f my-file-system --new-path my-file-system/my-file-renamed.txt --account-name mystorageaccount --auth-mode login
```

## <a name="delete-a-file"></a>Удаление файла

Удалите файл, выполнив команду `az storage fs file delete`.

В этом примере удаляется файл с именем `my-file.txt`.

```azurecli
az storage fs file delete -p my-directory/my-file.txt -f my-file-system  --account-name mystorageaccount --auth-mode login 
```

## <a name="see-also"></a>См. также раздел

- [Примеры](https://github.com/Azure/azure-cli/blob/dev/src/azure-cli/azure/cli/command_modules/storage/docs/ADLS%20Gen2.md)
- [Отправка отзывов](https://github.com/Azure/azure-cli-extensions/issues)
- [Известные проблемы](data-lake-storage-known-issues.md#api-scope-data-lake-client-library)
- [Использование Azure CLI для управления списками ACL в Azure Data Lake Storage 2-го поколения](data-lake-storage-acl-cli.md)