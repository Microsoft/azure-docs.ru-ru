---
title: Краткое руководство. Использование библиотеки Хранилища BLOB-объектов Azure версии 12 для C++
description: Из этого краткого руководства вы узнаете, как использовать клиентскую библиотеку службы Хранилища BLOB-объектов Azure версии 12 для C++ для создания контейнера и большого двоичного объекта в хранилище BLOB-объектов. Далее вы узнаете, как скачать большой двоичный объект на локальный компьютер и как получить список всех больших двоичных объектов в контейнере.
author: twooley
ms.author: twooley
ms.date: 10/21/2020
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.openlocfilehash: e6832cc835a464a2a3d17d5ed286e71b17bc980e
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107312286"
---
# <a name="quickstart-azure-blob-storage-client-library-v12-for-c"></a>Краткое руководство. Использование библиотеки Хранилища BLOB-объектов Azure версии 12 для C++

Приступите к работе с клиентской библиотекой Хранилища BLOB-объектов Azure версии 12 для C++. Хранилище BLOB-объектов Azure — это решение корпорации Майкрософт для хранения объектов в облаке. Чтобы установить пакет и испробовать пример кода для выполнения базовых задач, выполните приведенные здесь действия. Хранилище BLOB-объектов оптимизировано для хранения больших объемов неструктурированных данных.

Клиентскую библиотеку Хранилища BLOB-объектов Azure версии 12 для C++ можно использовать для выполнения таких задач:

- Создание контейнера
- передача большого двоичного объекта в хранилище Azure;
- перечисление всех больших двоичных объектов в контейнере;
- скачивание большого двоичного объекта на локальный компьютер;
- Удаление контейнера

Ресурсы:

- [Справочная документация по API](https://azure.github.io/azure-sdk-for-cpp/storage.html)
- [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-cpp/tree/master/sdk/storage)
- [Примеры](../common/storage-samples-c-plus-plus.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

## <a name="prerequisites"></a>Предварительные требования

- [Подписка Azure.](https://azure.microsoft.com/free/)
- [Учетная запись хранения Azure](../common/storage-account-create.md)
- [Компилятор C++](https://azure.github.io/azure-sdk/cpp_implementation.html#supported-platforms).
- [CMake](https://cmake.org/).
- [Диспетчер пакетов Vcpkg - C и C++](https://github.com/microsoft/vcpkg/blob/master/docs/README.md).
- [LibCurl](https://curl.haxx.se/libcurl/).
- [LibXML2](http://www.xmlsoft.org/).

## <a name="setting-up"></a>Настройка

В этом разделе рассматривается подготовка проекта для работы с клиентской библиотекой Хранилища BLOB-объектов Azure версии 12 для C++.

### <a name="install-the-packages"></a>Установка пакетов

Установите пакеты LibCurl и LibXML2 с помощью команды `vcpkg install`, если вы этого еще не сделали.

```console
vcpkg.exe install libxml2:x64-windows curl:x64-windows
```

Следуйте инструкциям на сайте GitHub, чтобы получить и создать [пакет Azure SDK для C++](https://github.com/Azure/azure-sdk-for-cpp/).

### <a name="create-the-project"></a>Создание проекта

В Visual Studio создайте консольное приложение C++ для Windows с именем *BlobQuickstartV12*.

:::image type="content" source="./media/quickstart-blobs-c-plus-plus/vs-create-project.jpg" alt-text="Диалоговое окно Visual Studio для настройки нового консольного приложения C++ для Windows":::

Добавьте в проект следующие библиотеки:

- libcurl.lib;
- libxml2.lib;
- bcrypt.lib;
- Crypt32.Lib;
- WS2_32.Lib;
- azure-core.lib;
- azure-storage-common.lib;
- azure-storage-blobs.lib.

[!INCLUDE [storage-quickstart-credentials-include](../../../includes/storage-quickstart-credentials-include.md)]

## <a name="object-model"></a>Объектная модель

Хранилище BLOB-объектов Azure оптимизировано для хранения больших объемов неструктурированных данных. Неструктурированные данные — это данные, которые не соответствуют определенной модели данных или определению, например текстовых или двоичных данных. В Хранилище BLOB-объектов предлагается три типа ресурсов:

- учетная запись хранения;
- контейнер в учетной записи хранения;
- большой двоичный объект в контейнере.

На следующей схеме показана связь между этими ресурсами.

![Схема архитектуры службы Хранилища BLOB-объектов](./media/storage-blobs-introduction/blob1.png)

Используйте следующие классы C++ для взаимодействия с этими ресурсами:

- [BlobServiceClient](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_service_client.html). Класс `BlobServiceClient` позволяет управлять ресурсами службы хранилища Azure и контейнерами больших двоичных объектов.
- [BlobContainerClient](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_container_client.html). Класс `BlobContainerClient` позволяет управлять контейнерами службы хранилища Azure и содержащимися в них большими двоичными объектами.
- [BlobClient](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_client.html). Класс `BlobClient` позволяет управлять большими двоичными объектами службы хранилища Azure. Это базовый класс для всех специализированных классов больших двоичных объектов.
- [BlockBlobClient.](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_block_blob_client.html) Класс `BlockBlobClient` позволяет управлять блочными BLOB-объектами службы хранилища Azure.

## <a name="code-examples"></a>Примеры кода

В этих примерах фрагментов кода показано, как выполнять следующие задачи с помощью клиентской библиотеки Хранилища BLOB-объектов Azure для C++:

- [Добавление включения файлов](#add-include-files)
- [Получение строки подключения](#get-the-connection-string)
- [Создание контейнера](#create-a-container)
- [отправка больших двоичных объектов в контейнер](#upload-blobs-to-a-container);
- [перечисление больших двоичных объектов в контейнере](#list-the-blobs-in-a-container);
- [скачивание больших двоичных объектов](#download-blobs);
- [Удаление контейнера](#delete-a-container)

### <a name="add-include-files"></a>Добавление включения файлов

Из каталога проекта:

1. Откройте файл решения *BlobQuickstartV12.sln* в Visual Studio.
1. В Visual Studio откройте исходный файл *BlobQuickstartV12.cpp*.
1. Удалите весь код в `main`, который был создан автоматически.
1. Добавьте в него операторы `#include`.

:::code language="cpp" source="~/azure-storage-snippets/blobs/quickstarts/C++/V12/BlobQuickstartV12/BlobQuickstartV12/BlobQuickstartV12.cpp" id="Snippet_Includes":::

### <a name="get-the-connection-string"></a>Получение строки подключения

Приведенный ниже код извлекает строку подключения для вашей учетной записи хранения из переменной среды, созданной в разделе [Настройка строки подключения хранилища](#configure-your-storage-connection-string).

Добавьте этот код в `main()`:

:::code language="cpp" source="~/azure-storage-snippets/blobs/quickstarts/C++/V12/BlobQuickstartV12/BlobQuickstartV12/BlobQuickstartV12.cpp" id="Snippet_ConnectionString":::

### <a name="create-a-container"></a>Создание контейнера

Создайте экземпляр класса [BlobContainerClient](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_container_client.html), вызвав функцию [CreateFromConnectionString](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_container_client.html#ad103d1e3a7ce7c53a82da887d12ce6fe). Затем вызовите метод [Create](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_container_client.html#a22a1e001068a4ec52bb1b6bd8b52c047), чтобы создать контейнер в учетной записи хранения.

> [!IMPORTANT]
> Имена контейнеров должны состоять из знаков нижнего регистра. Дополнительные сведения об именовании контейнеров и больших двоичных объектов см. в статье [Naming and Referencing Containers, Blobs, and Metadata](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata) (Именование контейнеров, больших двоичных объектов и метаданных и ссылка на них).

Добавьте этот код в конец `main()`:

:::code language="cpp" source="~/azure-storage-snippets/blobs/quickstarts/C++/V12/BlobQuickstartV12/BlobQuickstartV12/BlobQuickstartV12.cpp" id="Snippet_CreateContainer":::

### <a name="upload-blobs-to-a-container"></a>Отправка больших двоичных объектов в контейнер

Приведенный ниже фрагмент кода:

1. Объявляет строку, содержащую Hello Azure!
1. Возвращает ссылку на объект [BlockBlobClient](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_block_blob_client.html), вызывая [GetBlockBlobClient](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_container_client.html#acd8c68e3f37268fde0010dd478ff048f) для контейнера из раздела [Создание контейнера](#create-a-container).
1. Передает строку в большой двоичный объект, вызывая функцию [UploadFrom](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_block_blob_client.html#af93af7e37f8806e39481596ef253f93d). С помощью этой функции создается большой двоичный объект, если он не был создан ранее, или же, если он имеется, происходит его обновление.

Добавьте этот код в конец `main()`:

:::code language="cpp" source="~/azure-storage-snippets/blobs/quickstarts/C++/V12/BlobQuickstartV12/BlobQuickstartV12/BlobQuickstartV12.cpp" id="Snippet_UploadBlob":::

### <a name="list-the-blobs-in-a-container"></a>Перечисление BLOB-объектов в контейнере

Выведите список больших двоичных объектов в контейнере, вызвав функцию [ListBlobsFlatSegment](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_container_client.html#ac7712bc46909dc061d6bf554b496550c). В контейнер был добавлен лишь один большой двоичный объект, поэтому операция возвращает только его.

Добавьте этот код в конец `main()`:

:::code language="cpp" source="~/azure-storage-snippets/blobs/quickstarts/C++/V12/BlobQuickstartV12/BlobQuickstartV12/BlobQuickstartV12.cpp" id="Snippet_ListBlobs":::

### <a name="download-blobs"></a>Скачивание больших двоичных объектов

Получите свойства отправленного большого двоичного объекта. Затем объявите новый объект `std::string` и измените его размер, используя свойства отправленного большого двоичного объекта. Скачайте ранее созданный большой двоичный объект в новый `std::string`, вызвав функцию [DownloadTo](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_client.html#aa844f37a8c216f3cb0f27912b114c4d2) в базовом классе [BlobClient](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_client.html). Наконец, отобразите скачанные данные большого двоичного объекта.

Добавьте этот код в конец `main()`:

:::code language="cpp" source="~/azure-storage-snippets/blobs/quickstarts/C++/V12/BlobQuickstartV12/BlobQuickstartV12/BlobQuickstartV12.cpp" id="Snippet_DownloadBlob":::

### <a name="delete-a-blob"></a>Удаление BLOB-объекта

Следующий код удаляет большой двоичный объект из контейнера Хранилища BLOB-объектов Azure, вызывая функцию [BlobClient.Delete](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_client.html#a621eabcc8d23893ca1eb106494198615).

:::code language="cpp" source="~/azure-storage-snippets/blobs/quickstarts/C++/V12/BlobQuickstartV12/BlobQuickstartV12/BlobQuickstartV12.cpp" id="Snippet_DeleteBlob":::

### <a name="delete-a-container"></a>Удаление контейнера

Следующий код очищает созданные приложением ресурсы, полностью удаляя контейнер с помощью метода [BlobContainerClient.Delete](https://azuresdkdocs.blob.core.windows.net/$web/cpp/azure-storage-blobs/1.0.0-beta.2/class_azure_1_1_storage_1_1_blobs_1_1_blob_container_client.html#aa6b1db52697ae92e9a1227e2e02a5178).

Добавьте этот код в конец `main()`:

:::code language="cpp" source="~/azure-storage-snippets/blobs/quickstarts/C++/V12/BlobQuickstartV12/BlobQuickstartV12/BlobQuickstartV12.cpp" id="Snippet_DeleteContainer":::

## <a name="run-the-code"></a>Выполнение кода

Это приложение создает контейнер и передает текстовый файл в Хранилище BLOB-объектов Azure. Затем пример выводит список больших двоичных объектов в контейнере, скачивает файл и отображает содержимое файла. Наконец, приложение удаляет большой двоичный объект и контейнер.

Вы должны увидеть выходные данные приложения, как показано ниже.

```output
Azure Blob Storage v12 - C++ quickstart sample
Creating container: myblobcontainer
Uploading blob: blob.txt
Listing blobs...
Blob name: blob.txt
Downloaded blob contents: Hello Azure!
Deleting blob: blob.txt
Deleting container: myblobcontainer
```

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как передавать, загружать и перечислять файлы в большие двоичные объекты с помощью C++. Кроме того, вы научились создавать и удалять контейнер Хранилища BLOB-объектов Azure.

Чтобы просмотреть примеры C++ для Хранилища BLOB-объектов, перейдите к следующему разделу:

> [!div class="nextstepaction"]
> [Пример для пакета SDK Хранилища BLOB-объектов Azure версии 12 для C++](https://github.com/Azure/azure-sdk-for-cpp/tree/master/sdk/storage/azure-storage-blobs/sample)
