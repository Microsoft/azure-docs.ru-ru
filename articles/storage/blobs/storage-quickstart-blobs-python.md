---
title: Краткое руководство. Библиотека Хранилища BLOB-объектов Azure версии 12 для Python
description: Из этого краткого руководства вы узнаете, как с помощью клиентской библиотеки Хранилища BLOB-объектов Azure версии 12 для Python создать контейнер и большой двоичный объект в Хранилище BLOB-объектов. Далее вы узнаете, как скачать большой двоичный объект на локальный компьютер и как получить список всех больших двоичных объектов в контейнере.
author: twooley
ms.author: twooley
ms.date: 01/28/2021
ms.topic: quickstart
ms.service: storage
ms.subservice: blobs
ms.custom:
- devx-track-python
- mode-api
ms.openlocfilehash: 9a6ccbe5482ac8ffe484c3155334cba1b50ba0dc
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107535644"
---
# <a name="quickstart-manage-blobs-with-python-v12-sdk"></a>Краткое руководство. Управление большими двоичными объектами с помощью пакета SDK для Python версии 12

Из этого краткого руководства вы узнаете, как управлять большими двоичными объектами с использованием Python. Большие двоичные объекты — это объекты, которые могут содержать большие объемы текстовых или двоичных данных, включая изображения, документы, потоковое мультимедиа и архивные данные. Вы научитесь отправлять и скачивать большие двоичные объекты, получать список таких объектов, а также создавать и удалять контейнеры.

Дополнительные ресурсы:

* [Справочная документация по API](/python/api/azure-storage-blob)
* [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-blob)
* [Пакет (индекс пакета Python)](https://pypi.org/project/azure-storage-blob/)
* [Примеры](../common/storage-samples-python.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#blob-samples)

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.
- Учетная запись хранения Azure. [Создание учетной записи хранения](../common/storage-account-create.md).
- [Python](https://www.python.org/downloads/) 2.7, 3.6 или более поздней версии.

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

## <a name="setting-up"></a>Настройка

В этом разделе рассматривается подготовка проекта для работы с клиентской библиотекой Хранилища BLOB-объектов Azure версии 12 для Python.

### <a name="create-the-project"></a>Создание проекта

Создайте приложение Python с именем *blob-quickstart-v12*.

1. В окне консоли (командная строка, PowerShell или Bash) создайте каталог для проекта.

    ```console
    mkdir blob-quickstart-v12
    ```

1. Перейдите в только что созданный каталог *blob-quickstart-v12*.

    ```console
    cd blob-quickstart-v12
    ```

1. В каталоге *blob-quickstart-v12* создайте каталог *data*. Это каталог для создания и хранения файлов данных для больших двоичных объектов.

    ```console
    mkdir data
    ```

### <a name="install-the-package"></a>Установка пакета

Оставаясь в каталоге приложения, установите пакет клиентской библиотеки Хранилища BLOB-объектов Azure для Python с помощью команды `pip install`.

```console
pip install azure-storage-blob
```

Эта команда устанавливает пакет клиентской библиотеки Хранилища BLOB-объектов Azure для Python и все библиотеки, от которых она зависит. В данном случае это только основная библиотека Azure для Python.

### <a name="set-up-the-app-framework"></a>Настройка платформы приложения

Из каталога проекта:

1. Откройте новый текстовый файл в редакторе кода.
1. Добавьте в него операторы `import`.
1. Создайте структуру программы, включая простую обработку исключений.

    Вот этот код:

    :::code language="python" source="~/azure-storage-snippets/blobs/quickstarts/python/V12/app_framework.py":::

1. Сохраните новый файл как *blob-quickstart-v12.py* в каталоге *blob-quickstart-v12*.

[!INCLUDE [storage-quickstart-credentials-include](../../../includes/storage-quickstart-credentials-include.md)]

## <a name="object-model"></a>Объектная модель

Хранилище BLOB-объектов Azure оптимизировано для хранения больших объемов неструктурированных данных. Неструктурированные данные — это данные, которые не соответствуют определенной модели данных или определению, например текстовых или двоичных данных. В хранилище BLOB-объектов предлагается три типа ресурсов:

* учетная запись хранения;
* контейнер в учетной записи хранения;
* большой двоичный объект в контейнере.

На следующей схеме показана связь между этими ресурсами.

![Схема архитектуры службы хранилища BLOB-объектов](./media/storage-blobs-introduction/blob1.png)

Используйте следующие классы Python для взаимодействия с этими ресурсами.

* [BlobServiceClient](/python/api/azure-storage-blob/azure.storage.blob.blobserviceclient). Класс `BlobServiceClient` позволяет управлять ресурсами службы хранилища Azure и контейнерами больших двоичных объектов.
* [ContainerClient](/python/api/azure-storage-blob/azure.storage.blob.containerclient). Класс `ContainerClient` позволяет управлять контейнерами службы хранилища Azure и содержащимися в них большими двоичными объектами.
* [BlobClient](/python/api/azure-storage-blob/azure.storage.blob.blobclient). Класс `BlobClient` позволяет управлять большими двоичными объектами службы хранилища Azure.

## <a name="code-examples"></a>Примеры кода

В этих примерах фрагментов кода показано, как выполнять следующие задачи с помощью клиентской библиотеки Хранилища BLOB-объектов Azure для Python:

* [Получение строки подключения](#get-the-connection-string)
* [Создание контейнера](#create-a-container)
* [отправка больших двоичных объектов в контейнер](#upload-blobs-to-a-container);
* [перечисление больших двоичных объектов в контейнере](#list-the-blobs-in-a-container);
* [скачивание больших двоичных объектов](#download-blobs);
* [Удаление контейнера](#delete-a-container)

### <a name="get-the-connection-string"></a>Получение строки подключения

Приведенный ниже код извлекает строку подключения к учетной записи хранения из переменной среды, созданной при изучении раздела [Настройка строки подключения хранилища](#configure-your-storage-connection-string).

Добавьте этот код в блок `try`.

:::code language="python" source="~/azure-storage-snippets/blobs/quickstarts/python/V12/blob-quickstart-v12.py" id="Snippet_ConnectionString":::

### <a name="create-a-container"></a>Создание контейнера

Выберите имя нового контейнера. Приведенный ниже код добавляет к имени контейнера значение UUID, чтобы сделать это имя уникальным.

> [!IMPORTANT]
> Имена контейнеров должны состоять из знаков нижнего регистра. Дополнительные сведения об именовании контейнеров и больших двоичных объектов см. в статье [Naming and Referencing Containers, Blobs, and Metadata](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata) (Именование контейнеров, больших двоичных объектов и метаданных и ссылка на них).

Создайте экземпляр класса [BlobServiceClient](/python/api/azure-storage-blob/azure.storage.blob.blobserviceclient), вызвав метод [from_connection_string](/python/api/azure-storage-blob/azure.storage.blob.blobserviceclient#from-connection-string-conn-str--credential-none----kwargs-). Затем вызовите метод [create_container](/python/api/azure-storage-blob/azure.storage.blob.blobserviceclient#create-container-name--metadata-none--public-access-none----kwargs-), чтобы создать контейнер в учетной записи хранения.

Добавьте следующий код в конец блока `try`.

:::code language="python" source="~/azure-storage-snippets/blobs/quickstarts/python/V12/blob-quickstart-v12.py" id="Snippet_CreateContainer":::

### <a name="upload-blobs-to-a-container"></a>Отправка больших двоичных объектов в контейнер

Приведенный ниже фрагмент кода:

1. Создает локальный каталог для хранения файлов данных.
1. Создает текстовый файл в локальном каталоге.
1. Возвращает ссылку на объект [BlobClient](/python/api/azure-storage-blob/azure.storage.blob.blobclient), вызывая метод [get_blob_client](/python/api/azure-storage-blob/azure.storage.blob.containerclient#get-blob-client-blob--snapshot-none-) для [BlobServiceClient](/python/api/azure-storage-blob/azure.storage.blob.blobserviceclient) из раздела [Создание контейнера](#create-a-container).
1. Передает локальный текстовый файл в большой двоичный объект, вызывая метод [upload_blob](/python/api/azure-storage-blob/azure.storage.blob.blobclient#upload-blob-data--blob-type--blobtype-blockblob---blockblob----length-none--metadata-none----kwargs-).

Добавьте следующий код в конец блока `try`.

:::code language="python" source="~/azure-storage-snippets/blobs/quickstarts/python/V12/blob-quickstart-v12.py" id="Snippet_UploadBlobs":::

### <a name="list-the-blobs-in-a-container"></a>Перечисление BLOB-объектов в контейнере

Выведите список больших двоичных объектов в контейнере, вызвав метод [list_blobs](/python/api/azure-storage-blob/azure.storage.blob.containerclient#list-blobs-name-starts-with-none--include-none----kwargs-). В этом случае в контейнер был добавлен лишь один большой двоичный объект, поэтому операция перечисления возвращает только его.

Добавьте следующий код в конец блока `try`.

:::code language="python" source="~/azure-storage-snippets/blobs/quickstarts/python/V12/blob-quickstart-v12.py" id="Snippet_ListBlobs":::

### <a name="download-blobs"></a>Скачивание больших двоичных объектов

Скачайте созданный ранее большой двоичный объект, вызвав метод [download_blob](/python/api/azure-storage-blob/azure.storage.blob.blobclient#download-blob-offset-none--length-none----kwargs-). Пример кода добавляет суффикс "DOWNLOAD" к имени файла, чтобы в локальной файловой системе можно было просмотреть оба файла.

Добавьте следующий код в конец блока `try`.

:::code language="python" source="~/azure-storage-snippets/blobs/quickstarts/python/V12/blob-quickstart-v12.py" id="Snippet_DownloadBlobs":::

### <a name="delete-a-container"></a>Удаление контейнера

Следующий код очищает созданные приложением ресурсы, полностью удаляя контейнер с помощью метода [delete_container](/python/api/azure-storage-blob/azure.storage.blob.containerclient#delete-container---kwargs-). Кроме того, при необходимости можно удалить локальные файлы.

Приложение приостанавливается для ввода пользователя, вызывая `input()`, перед удалением большого двоичного объекта, контейнера и локальных файлов. Перед удалением ресурсов убедитесь, что они были созданы правильно.

Добавьте следующий код в конец блока `try`.

:::code language="python" source="~/azure-storage-snippets/blobs/quickstarts/python/V12/blob-quickstart-v12.py" id="Snippet_CleanUp":::

## <a name="run-the-code"></a>Выполнение кода

В этом приложении тестовый файл создается в локальной папке, а затем передается в Хранилище BLOB-объектов Azure. Затем пример выводит список больших двоичных объектов в контейнере и скачивает файл с новым именем. Теперь вы можете сравнить старый и новый файлы.

Перейдите в каталог, содержащий файл *blob-quickstart-v12.py*, а затем выполните указанную команду `python`, чтобы запустить приложение.

```console
python blob-quickstart-v12.py
```

Вы должны увидеть выходные данные приложения, как показано ниже.

```output
Azure Blob Storage v12 - Python quickstart sample

Uploading to Azure Storage as blob:
        quickstartcf275796-2188-4057-b6fb-038352e35038.txt

Listing blobs...
        quickstartcf275796-2188-4057-b6fb-038352e35038.txt

Downloading blob to
        ./data/quickstartcf275796-2188-4057-b6fb-038352e35038DOWNLOAD.txt

Press the Enter key to begin clean up

Deleting blob container...
Deleting the local source and downloaded files...
Done
```

Прежде чем начать процесс удаления, проверьте наличие двух файлов в папке *data*. Вы можете открыть их и убедиться, что они идентичны.

После проверки файлов нажмите клавишу **ВВОД**, чтобы завершить работу с демонстрационной версией и удалить тестовые файлы.

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы узнали, как передавать и скачивать большие двоичные объекты, а также выводить их список с помощью Python.

Чтобы просмотреть примеры приложений для хранилища BLOB-объектов, перейдите к следующему разделу:

> [!div class="nextstepaction"]
> [Примеры для пакета SDK Хранилища BLOB-объектов Azure версии 12 для Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-blob/samples)

* Чтобы узнать больше, ознакомьтесь с [клиентскими библиотеками службы хранилища Azure для Python](/azure/developer/python/sdk/storage/overview).
* Учебники, примеры, краткие руководства и другую документацию можно найти на странице [Azure для разработчиков Python](/azure/python/).
