---
title: Краткое руководство. Создание большого двоичного объекта с помощью Azure CLI
titleSuffix: Azure Storage
description: В этом кратком руководстве вы узнаете, как с помощью Azure CLI передать большой двоичный объект в службу хранилища Azure, скачать большой двоичный объект и вывести список больших двоичных объектов в контейнере.
services: storage
author: tamram
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.date: 08/17/2020
ms.author: tamram
ms.custom: devx-track-azurecli
ms.openlocfilehash: 04f793f78cef938c31e7a30aad5569a54eb461a1
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "104613113"
---
# <a name="quickstart-create-download-and-list-blobs-with-azure-cli"></a>Краткое руководство. Создание, скачивание и составление списка больших двоичных объектов с помощью Azure CLI

Azure CLI — это интерфейс командной строки Azure для управления ресурсами Azure. Вы можете использовать его в браузере с Azure Cloud Shell. Его также можно установить в macOS, Linux или Windows и запускать из командной строки. В этом кратком руководстве содержатся сведения об использовании Azure CLI для отправки и скачивания данных в хранилище BLOB-объектов Azure и обратно.

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment-h3.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0.46 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="authorize-access-to-blob-storage"></a>Авторизация доступа к хранилищу BLOB-объектов

Вы можете предоставить доступ к хранилищу BLOB-объектов из Azure CLI либо с использованием учетных данных Azure AD или ключа доступа к учетной записи хранения. Рекомендуется использовать учетные данные Azure AD. В этой статье показано, как авторизовать операции в хранилище BLOB-объектов с помощью Azure AD.

Команды Azure CLI для операций с данными в хранилище BLOB-объектов поддерживают параметр `--auth-mode`, что позволяет указать, как авторизовать определенную операцию. Задайте для параметра `--auth-mode` значение `login`, чтобы выполнять авторизацию с использованием учетных данных Azure AD. Дополнительные сведения см. в статье [Авторизация доступа к данным BLOB-объектов или очередей с помощью Azure CLI](./authorize-data-operations-cli.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

Только операции с данными хранилища BLOB-объектов поддерживают параметр `--auth-mode`. Для авторизации операций управления, таких как создание группы ресурсов или учетной записи хранения, автоматически используются учетные данные Azure AD.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов Azure с помощью команды [az group create](/cli/azure/group). Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими.

Не забудьте заменить значения заполнителей в угловых скобках собственными значениями.

```azurecli
az group create \
    --name <resource-group> \
    --location <location>
```

## <a name="create-a-storage-account"></a>Создание учетной записи хранения

Создайте учетную запись хранения общего назначения с помощью команды [az storage account create](/cli/azure/storage/account). Эта учетная запись хранения может использоваться для всех четырех служб: больших двоичных объектов, файлов, таблиц и очередей.

Не забудьте заменить значения заполнителей в угловых скобках собственными значениями.

```azurecli
az storage account create \
    --name <storage-account> \
    --resource-group <resource-group> \
    --location <location> \
    --sku Standard_ZRS \
    --encryption-services blob
```

## <a name="create-a-container"></a>Создание контейнера

Большие двоичные объекты всегда отправляются в контейнер. Вы можете упорядочивать группы больших двоичных объектов в контейнеры аналогично организации файлов в папках на компьютере. Создайте контейнер для хранения больших двоичных объектов с помощью команды [az storage container create](/cli/azure/storage/container).

В следующем примере учетная запись Azure AD используется для авторизации операции создания контейнера. Прежде чем создавать контейнер, назначьте себе роль [Участник данных в хранилище BLOB-объектов](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor). Даже если вы являетесь владельцем учетной записи, вам потребуются явные разрешения для выполнения операций с данными в учетной записи хранения. Дополнительные сведения о назначении ролей Azure см. в разделе [Использование Azure CLI для назначения роли Azure для доступа](../common/storage-auth-aad-rbac-cli.md?toc=/azure/storage/blobs/toc.json).  

Не забудьте заменить значения заполнителей в угловых скобках собственными значениями.

```azurecli
az ad signed-in-user show --query objectId -o tsv | az role assignment create \
    --role "Storage Blob Data Contributor" \
    --assignee @- \
    --scope "/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>"

az storage container create \
    --account-name <storage-account> \
    --name <container> \
    --auth-mode login
```

> [!IMPORTANT]
> Назначение ролей Azure может занимать несколько минут.

Вы также можете использовать ключ учетной записи хранения, чтобы авторизовать операцию создания контейнера. Дополнительные сведения об авторизации операций с данными с помощью Azure CLI см. в статье [Авторизация доступа к данным большого двоичного объекта или очереди с помощью Azure CLI](./authorize-data-operations-cli.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

## <a name="upload-a-blob"></a>Передача больших двоичных объектов

Хранилище BLOB-объектов поддерживает блочные, добавочные и страничные BLOB-объекты. В примерах в этом кратком руководстве показано, как работать с блочными BLOB-объектами.

Сначала создайте файл для передачи в блочный BLOB-объект. Если вы используете Azure Cloud Shell, создайте этот файл с помощью следующей команды.

```bash
vi helloworld
```

Когда файл откроется, щелкните **Вставить**. Введите *Hello World*, а затем нажмите клавишу **ESC**. Затем введите *:x* и нажмите клавишу **ВВОД**.

В этом примере большой двоичный объект отправляется в контейнер, созданный на последнем шаге, с помощью команды [az storage blob upload](/cli/azure/storage/blob). Нет необходимости указывать путь к файлу, так как файл создан в корневом каталоге. Не забудьте заменить значения заполнителей в угловых скобках собственными значениями.

```azurecli
az storage blob upload \
    --account-name <storage-account> \
    --container-name <container> \
    --name helloworld \
    --file helloworld \
    --auth-mode login
```

С помощью этой операции создается большой двоичный объект, если он не был создан ранее, или же, если он имеется, происходит его замещение. Прежде чем продолжить, отправьте нужное количество файлов.

Чтобы одновременно отправить несколько файлов, используйте команду [az storage blob upload-batch](/cli/azure/storage/blob).

## <a name="list-the-blobs-in-a-container"></a>Перечисление BLOB-объектов в контейнере

Выведите список больших двоичных объектов в контейнере, выполнив команду [az storage blob list](/cli/azure/storage/blob). Не забудьте заменить значения заполнителей в угловых скобках собственными значениями.

```azurecli
az storage blob list \
    --account-name <storage-account> \
    --container-name <container> \
    --output table \
    --auth-mode login
```

## <a name="download-a-blob"></a>Загрузка BLOB-объектов

Выполните команду [az storage blob download](/cli/azure/storage/blob), чтобы скачать отправленный ранее большой двоичный объект. Не забудьте заменить значения заполнителей в угловых скобках собственными значениями.

```azurecli
az storage blob download \
    --account-name <storage-account> \
    --container-name <container> \
    --name helloworld \
    --file ~/destination/path/for/file \
    --auth-mode login
```

## <a name="data-transfer-with-azcopy"></a>Передача данных с помощью AzCopy

Служебная программа командной строки AzCopy обеспечивает высокопроизводительную передачу данных с поддержкой сценариев для службы хранилища Azure. AzCopy можно использовать для передачи данных из хранилища BLOB-объектов, Файлов Azure и обратно. Дополнительные сведения об AzCopy версии 10, которая является последней, см. в статье [Get started with AzCopy](../common/storage-use-azcopy-v10.md) (Начало работы с AzCopy). Дополнительные сведения об использовании AzCopy версии 10 с хранилищем BLOB-объектов см. в статье [Transfer data with AzCopy and Blob storage](../common/storage-use-azcopy-v10.md#transfer-data) (Передача данных с помощью AzCopy и хранилища BLOB-объектов).

В следующем примере AzCopy используется для передачи локального файла в большой двоичный объект. Не забудьте заменить примеры значений собственными значениями:

```bash
azcopy login
azcopy copy 'C:\myDirectory\myTextFile.txt' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt'
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы решили удалить ресурсы, созданные для этого краткого руководства, вместе с группой ресурсов, выполните для этого команду [az group delete](/cli/azure/group). Не забудьте заменить значения заполнителей в угловых скобках собственными значениями.

```azurecli
az group delete \
    --name <resource-group> \
    --no-wait
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы узнали, как передавать файлы между локальной файловой системой и контейнером в хранилище BLOB-объектов Azure. Чтобы узнать больше о работе с хранилищем BLOB-объектов с использованием Azure CLI, изучите примеры Azure CLI для хранилища BLOB-объектов.

> [!div class="nextstepaction"]
> [Примеры Azure CLI для хранилища BLOB-объектов](./storage-samples-blobs-cli.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
