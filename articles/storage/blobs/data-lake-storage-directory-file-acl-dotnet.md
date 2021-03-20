---
title: Использование .NET для управления данными в Azure Data Lake Storage 2-го поколения
description: Используйте клиентскую библиотеку службы хранилища Azure для .NET, чтобы управлять каталогами и файлами в учетных записях хранения с включенным иерархическим пространством имен.
author: normesta
ms.service: storage
ms.date: 02/17/2021
ms.author: normesta
ms.topic: how-to
ms.subservice: data-lake-storage-gen2
ms.reviewer: prishet
ms.custom: devx-track-csharp
ms.openlocfilehash: 52e993a22a512a94c8b5b8b050205db0c4ce0b1b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100650326"
---
# <a name="use-net-to-manage-directories-and-files-in-azure-data-lake-storage-gen2"></a>Использование .NET для управления каталогами и файлами в Azure Data Lake Storage 2-го поколения

В этой статье показано, как использовать .NET для создания каталогов и файлов в учетных записях хранения с иерархическим пространством имен и управления ими.

Сведения о том, как получить, задать и обновить списки управления доступом (ACL) для каталогов и файлов, см. [в разделе Использование .NET для управления списками ACL в Azure Data Lake Storage 2-го поколения](data-lake-storage-acl-dotnet.md).

[Пакет (NuGet)](https://www.nuget.org/packages/Azure.Storage.Files.DataLake)  |  [Примеры](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Azure.Storage.Files.DataLake)  |  [Справочник](/dotnet/api/azure.storage.files.datalake)  |  по API Сопоставление Gen1 с [Gen2](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Azure.Storage.Files.DataLake/GEN1_GEN2_MAPPING.md)  |  [Отправить отзыв](https://github.com/Azure/azure-sdk-for-net/issues)

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).

- Учетная запись хранения с включенным иерархическим пространством имен. Выполните [эти](create-data-lake-storage-account.md) инструкции, чтобы создать учетную запись.

## <a name="set-up-your-project"></a>Настройка проекта

Чтобы приступить к работе, установите пакет NuGet [Azure. Storage. Files. Data Lake](https://www.nuget.org/packages/Azure.Storage.Files.DataLake/) .

Дополнительные сведения об установке пакетов NuGet см. в разделе [Установка пакетов и управление ими в Visual Studio с помощью диспетчера пакетов NuGet](/nuget/consume-packages/install-use-packages-visual-studio).

Затем добавьте эти операторы using в начало файла кода.

```csharp
using Azure;
using Azure.Storage.Files.DataLake;
using Azure.Storage.Files.DataLake.Models;
using Azure.Storage;
using System.IO;

```

## <a name="connect-to-the-account"></a>Подключение к учетной записи

Чтобы использовать фрагменты кода в этой статье, необходимо создать экземпляр [даталакесервицеклиент](/dotnet/api/azure.storage.files.datalake.datalakeserviceclient) , представляющий учетную запись хранения. 

### <a name="connect-by-using-an-account-key"></a>Подключение с помощью ключа учетной записи

Это самый простой способ подключения к учетной записи. 

В этом примере создается экземпляр [даталакесервицеклиент](/dotnet/api/azure.storage.files.datalake.datalakeserviceclient) с помощью ключа учетной записи.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Authorize_DataLake.cs" id="Snippet_AuthorizeWithKey":::

### <a name="connect-by-using-azure-active-directory-azure-ad"></a>Подключение с помощью Azure Active Directory (Azure AD)

Для проверки подлинности приложения в Azure AD можно использовать [клиентскую библиотеку удостоверений Azure для .NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/identity/Azure.Identity) .

В этом примере создается экземпляр [даталакесервицеклиент](/dotnet/api/azure.storage.files.datalake.datalakeserviceclient) с использованием идентификатора клиента, секрета клиента и идентификатора клиента.  Чтобы получить эти значения, см. статью [Получение маркера из Azure AD для авторизации запросов из клиентского приложения](../common/storage-auth-aad-app.md).

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Authorize_DataLake.cs" id="Snippet_AuthorizeWithAAD":::

> [!NOTE]
> Дополнительные примеры см. в документации по [клиентской библиотеке удостоверений Azure для .NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/identity/Azure.Identity) .

## <a name="create-a-container"></a>Создание контейнера

Контейнер выступает в качестве файловой системы для файлов. Его можно создать, вызвав метод [даталакесервицеклиент. креатефилесистем](/dotnet/api/azure.storage.files.datalake.datalakeserviceclient.createfilesystemasync) .

В этом примере создается контейнер с именем `my-file-system` . 

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD_DataLake.cs" id="Snippet_CreateContainer":::

## <a name="create-a-directory"></a>Создание каталога

Создайте ссылку на каталог, вызвав метод [даталакефилесистемклиент. креатедиректорясинк](/dotnet/api/azure.storage.files.datalake.datalakefilesystemclient.createdirectoryasync) .

В этом примере добавляется каталог с именем `my-directory` в контейнер, а затем добавляется подкаталог с именем `my-subdirectory` . 

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD_DataLake.cs" id="Snippet_CreateDirectory":::

## <a name="rename-or-move-a-directory"></a>Переименование или перемещение каталога

Переименуйте или переместите каталог, вызвав метод [даталакедиректориклиент. ренамеасинк](/dotnet/api/azure.storage.files.datalake.datalakedirectoryclient.renameasync) . Передайте путь к нужному каталогу параметру. 

В этом примере подкаталог переименовывается в имя `my-subdirectory-renamed` .

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD_DataLake.cs" id="Snippet_RenameDirectory":::

В этом примере каталог с именем перемещается `my-subdirectory-renamed` в подкаталог каталога с именем `my-directory-2` . 

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD_DataLake.cs" id="Snippet_MoveDirectory":::

## <a name="delete-a-directory"></a>Удаление каталога

Удалите каталог, вызвав метод [даталакедиректориклиент. Delete](/dotnet/api/azure.storage.files.datalake.datalakedirectoryclient.delete) .

В этом примере удаляется каталог `my-directory`.  

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD_DataLake.cs" id="Snippet_DeleteDirectory":::

## <a name="upload-a-file-to-a-directory"></a>Отправка файла в каталог

Сначала создайте ссылку на файл в целевом каталоге, создав экземпляр класса [даталакефилеклиент](/dotnet/api/azure.storage.files.datalake.datalakefileclient) . Отправьте файл, вызвав метод [даталакефилеклиент. аппендасинк](/dotnet/api/azure.storage.files.datalake.datalakefileclient.appendasync) . Обязательно завершите передачу, вызвав метод [даталакефилеклиент. FlushAsync](/dotnet/api/azure.storage.files.datalake.datalakefileclient.flushasync) .

В этом примере текстовый файл перегружается в каталог с именем `my-directory` . 

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD_DataLake.cs" id="Snippet_UploadFile":::

> [!TIP]
> Если размер файла большой, код должен выполнить несколько вызовов [даталакефилеклиент. аппендасинк](/dotnet/api/azure.storage.files.datalake.datalakefileclient.appendasync). Вместо этого рекомендуется использовать метод [даталакефилеклиент. UploadAsync](/dotnet/api/azure.storage.files.datalake.datalakefileclient.uploadasync#Azure_Storage_Files_DataLake_DataLakeFileClient_UploadAsync_System_IO_Stream_) . Таким образом, вы можете передать весь файл в одном вызове.
>
> Пример см. в следующем разделе.

## <a name="upload-a-large-file-to-a-directory"></a>Отправка большого файла в каталог

Используйте метод [даталакефилеклиент. UploadAsync](/dotnet/api/azure.storage.files.datalake.datalakefileclient.uploadasync#Azure_Storage_Files_DataLake_DataLakeFileClient_UploadAsync_System_IO_Stream_) для отправки больших файлов без выполнения нескольких вызовов к методу [даталакефилеклиент. аппендасинк](/dotnet/api/azure.storage.files.datalake.datalakefileclient.appendasync) .

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD_DataLake.cs" id="Snippet_UploadFileBulk":::

## <a name="download-from-a-directory"></a>Скачивание из каталога 

Сначала создайте экземпляр [даталакефилеклиент](/dotnet/api/azure.storage.files.datalake.datalakefileclient) , представляющий файл, который требуется скачать. Используйте метод [даталакефилеклиент. ReadAsync](/dotnet/api/azure.storage.files.datalake.datalakefileclient.readasync)  и проанализируйте возвращаемое значение, чтобы получить объект [потока](/dotnet/api/system.io.stream) . Используйте любой API-интерфейс обработки файлов .NET для сохранения байтов из потока в файл. 

В этом примере для сохранения байтов в файл используются объект [BinaryReader](/dotnet/api/system.io.binaryreader) и [файловый поток](/dotnet/api/system.io.filestream) . 

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD_DataLake.cs" id="Snippet_DownloadBinaryFromDirectory":::

## <a name="list-directory-contents"></a>Вывод содержимого каталогов

Перечислите содержимое каталога, вызвав метод [филесистемклиент. жетпассасинк](/dotnet/api/azure.storage.files.datalake.datalakefilesystemclient.getpathsasync) , а затем перечислить результаты.

В этом примере выводятся имена каждого файла, расположенного в каталоге с именем `my-directory` .

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/CRUD_DataLake.cs" id="Snippet_ListFilesInDirectory":::

## <a name="see-also"></a>См. также раздел

- [Справочная документация по API](/dotnet/api/azure.storage.files.datalake)
- [Пакет (NuGet)](https://www.nuget.org/packages/Azure.Storage.Files.DataLake)
- [Примеры](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Azure.Storage.Files.DataLake)
- [Сопоставление 1-го и 2-го поколения](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Azure.Storage.Files.DataLake/GEN1_GEN2_MAPPING.md)
- [Известные проблемы](data-lake-storage-known-issues.md#api-scope-data-lake-client-library)
- [Оставить отзыв](https://github.com/Azure/azure-sdk-for-net/issues)