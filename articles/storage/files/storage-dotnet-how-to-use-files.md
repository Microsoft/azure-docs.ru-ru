---
title: Разработка для службы файлов Azure с помощью .NET | Документация Майкрософт
description: Узнайте, как разрабатывать приложения и службы .NET, которые используют службу файлов Azure для хранения данных.
author: roygara
ms.service: storage
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 10/02/2020
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-csharp
ms.openlocfilehash: e112060db4a44884d3094a939b03ff106ba72e65
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96492205"
---
# <a name="develop-for-azure-files-with-net"></a>Разработка для службы файлов Azure с помощью .NET

[!INCLUDE [storage-selector-file-include](../../../includes/storage-selector-file-include.md)]

Изучите основы разработки приложений .NET, использующих службы [файлов Azure](storage-files-introduction.md) для хранения данных. В этой статье показано, как создать простое консольное приложение для следующих действий с .NET и службой файлов Azure.

- Получение содержимого файла.
- Установите максимальный размер или квоту для файлового ресурса.
- Создайте подписанный URL-адрес (SAS) для файла.
- Скопируйте файл в другой файл в той же учетной записи хранения
- Скопируйте файл в BLOB-объект в той же учетной записи хранения
- Создайте моментальный снимок файлового ресурса.
- Восстановление файла из моментального снимка общего ресурса.
- Используйте метрики службы хранилища Azure для устранения неполадок.

Дополнительные сведения о службе файлов Azure см. в статье [что такое служба файлов Azure?](storage-files-introduction.md)

[!INCLUDE [storage-check-out-samples-dotnet](../../../includes/storage-check-out-samples-dotnet.md)]

## <a name="understanding-the-net-apis"></a>Основные сведения об API-интерфейсах .NET

Служба файлов Azure предлагает два широких подхода к использованию клиентских приложений: SMB (блок сообщений сервера) и REST. В .NET API- `System.IO` интерфейсы и представляют собой `Azure.Storage.Files.Shares` абстрактные подходы.

API | Назначение | Примечания
----|-------------|------
[System.IO](/dotnet/api/system.io) | Требования вашего приложения: <ul><li>Требуется чтение и запись файлов с помощью SMB</li><li>выполнение на устройстве, которое получает доступ к учетной записи службы файлов Azure через порт 445;</li><li>не требуется управлять параметрами администрирования общей папки.</li></ul> | Файловый ввод-вывод, реализованный в службе файлов Azure по протоколу SMB, обычно аналогичен вводу-выводу с любым сетевым файловым ресурсом или локальным устройством хранения. Общие сведения о ряде функций .NET, включая ввод-вывод файлов, см. в руководстве по [консольному приложению](/dotnet/csharp/tutorials/console-teleprompter) .
[Azure. Storage. Files. Shares](/dotnet/api/azure.storage.files.shares) | Требования вашего приложения: <ul><li>Не удается получить доступ к службе файлов Azure по протоколу SMB через порт 445 из-за ограничений брандмауэра или поставщика услуг Интернета.</li><li>требуются административные функции, например возможность задать квоту для общей папки или создать подписанный URL-адрес.</li></ul> | В этой статье показано использование `Azure.Storage.Files.Shares` операций файлового ввода-вывода с помощью функции RESTful вместо протокола SMB и управления файловым ресурсом.

## <a name="create-the-console-application-and-obtain-the-assembly"></a>Создание консольного приложения и получение сборки

Клиентскую библиотеку службы файлов Azure можно использовать в любом типе приложения .NET. Эти приложения включают в себя облачные, веб-, классические и мобильные приложения Azure. В этом руководством мы создадим консольное приложение для простоты.

В Visual Studio создайте новое консольное приложение Windows. Ниже показано, как создать консольное приложение в Visual Studio 2019. Эти же действия можно выполнить и в других версиях Visual Studio.

1. Запустите Visual Studio и щелкните **Создать проект**
1. В окне **Создание нового проекта** выберите **консольное приложение (платформа .NET Framework)** для C#, а затем нажмите кнопку **Далее**.
1. В окне **Настройка нового проекта** введите имя приложения и нажмите кнопку **создать**.

Добавьте все примеры кода из этой статьи в `Program` класс в файле *Program. CS* .

## <a name="use-nuget-to-install-the-required-packages"></a>Установка необходимых пакетов с помощью NuGet

Ссылки на эти пакеты в проекте:

# <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

- [Библиотека ядра Azure для .NET](https://www.nuget.org/packages/Azure.Core/): Этот пакет является реализацией конвейера клиента Azure.
- [Клиентская библиотека Azure Storage BLOB для .NET](https://www.nuget.org/packages/Azure.Storage.Blobs/): Этот пакет обеспечивает программный доступ к ресурсам больших двоичных объектов в вашей учетной записи хранения.
- [Клиентская библиотека файлов службы хранилища Azure для .NET](https://www.nuget.org/packages/Azure.Storage.Files.Shares/): Этот пакет обеспечивает программный доступ к файловым ресурсам в вашей учетной записи хранения.
- [Библиотека системных Configuration Manager для .NET](https://www.nuget.org/packages/System.Configuration.ConfigurationManager/): Этот пакет предоставляет класс для хранения и извлечения значений в файле конфигурации.

Для получения пакетов можно использовать NuGet. Выполните следующие действия.

1. В **Обозреватель решений** щелкните правой кнопкой мыши проект и выберите **Управление пакетами NuGet**.
1. В разделе **Диспетчер пакетов NuGet** выберите **Обзор**. Затем найдите и выберите **Azure. Core**, а затем нажмите кнопку **установить**.

   На этом шаге выполняется установка пакета и его зависимостей.

1. Найдите и установите такие пакеты:

   - **Azure. Storage. BLOB**
   - **Azure. Storage. Files. Shares**
   - **System.Configuration.ConfigurationManager**

# <a name="net-v11"></a>[\..NET (версии 11)](#tab/dotnetv11)

- [Служба хранилища Microsoft Azure общей библиотеки для .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.Common/): Этот пакет обеспечивает программный доступ к общим ресурсам в вашей учетной записи хранения.
- [Служба хранилища Microsoft Azure библиотека больших двоичных объектов для .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.Blob/): Этот пакет обеспечивает программный доступ к ресурсам больших двоичных объектов в вашей учетной записи хранения.
- [Библиотека файлов служба хранилища Microsoft Azure для .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.File/): Этот пакет обеспечивает программный доступ к файловым ресурсам в вашей учетной записи хранения.
- [Библиотека Microsoft Azure Configuration Manager для .NET](https://www.nuget.org/packages/Microsoft.Azure.ConfigurationManager/): Этот пакет предоставляет класс для синтаксического анализа строки подключения в файле конфигурации во всех случаях, когда приложение выполняется.

Для получения пакетов можно использовать NuGet. Выполните следующие действия.

1. В **Обозреватель решений** щелкните правой кнопкой мыши проект и выберите **Управление пакетами NuGet**.
1. В разделе **Диспетчер пакетов NuGet** выберите **Обзор**. Затем найдите и выберите **Microsoft. Azure. Storage. BLOB**, а затем нажмите кнопку **установить**.

   На этом шаге выполняется установка пакета и его зависимостей.
1. Найдите и установите такие пакеты:

   - **Microsoft.Azure.Storage.Common**
   - **Microsoft.Azure.Storage.File**
   - **Microsoft.Azure.ConfigurationManager**

---

## <a name="save-your-storage-account-credentials-to-the-appconfig-file"></a>Сохранение учетных данных учетной записи хранения в файл App.config

Затем сохраните учетные данные в файле *App.config* проекта. В **Обозреватель решений** дважды щелкните `App.config` и измените файл, чтобы он был похож на следующий пример.

# <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

Замените `myaccount` именем вашей учетной записи хранения и `mykey` ключом учетной записи хранения.

:::code language="xml" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/app.config" highlight="5,6,7":::

# <a name="net-v11"></a>[\..NET (версии 11)](#tab/dotnetv11)

Замените `myaccount` именем вашей учетной записи хранения и `StorageAccountKeyEndingIn==` ключом учетной записи хранения.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <startup>
    <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5" />
  </startup>
  <appSettings>
    <add key="StorageConnectionString"
      value="DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=StorageAccountKeyEndingIn==" />
  </appSettings>
</configuration>
```

---

> [!NOTE]
> Эмулятор хранения Азурите в настоящее время не поддерживает службы файлов Azure. Для работы с файлами Azure ваша строка подключения должна быть нацелена на учетную запись хранения Azure в облаке.

## <a name="add-using-directives"></a>Добавление директив using

В **Обозреватель решений** откройте файл *Program. CS* и добавьте следующие директивы using в начало файла.

# <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_UsingStatements":::

# <a name="net-v11"></a>[\..NET (версии 11)](#tab/dotnetv11)

```csharp
using Microsoft.Azure; // Namespace for Azure Configuration Manager
using Microsoft.Azure.Storage; // Namespace for Storage Client Library
using Microsoft.Azure.Storage.Blob; // Namespace for Azure Blobs
using Microsoft.Azure.Storage.File; // Namespace for Azure Files
```

[!INCLUDE [storage-cloud-configuration-manager-include](../../../includes/storage-cloud-configuration-manager-include.md)]

---

## <a name="access-the-file-share-programmatically"></a>Доступ к общей папке программным путем

В файле *Program. CS* добавьте следующий код для программного доступа к общей папке.

# <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

Следующий метод создает общую папку, если она еще не существует. Метод начинается с создания объекта [шареклиент](/dotnet/api/azure.storage.files.shares.shareclient) из строки подключения. Затем в примере выполняется попытка загрузить созданный ранее файл. Вызовите этот метод из `Main()` .

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_CreateShare":::

# <a name="net-v11"></a>[\..NET (версии 11)](#tab/dotnetv11)

Затем добавьте следующее содержимое в `Main()` метод после приведенного выше кода, чтобы получить строку подключения. Этот код получает ссылку на созданный ранее файл и выводит его содержимое.

```csharp
// Create a CloudFileClient object for credentialed access to Azure Files.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Get a reference to the file share we created previously.
CloudFileShare share = fileClient.GetShareReference("logs");

// Ensure that the share exists.
if (share.Exists())
{
    // Get a reference to the root directory for the share.
    CloudFileDirectory rootDir = share.GetRootDirectoryReference();

    // Get a reference to the directory we created previously.
    CloudFileDirectory sampleDir = rootDir.GetDirectoryReference("CustomLogs");

    // Ensure that the directory exists.
    if (sampleDir.Exists())
    {
        // Get a reference to the file we created previously.
        CloudFile file = sampleDir.GetFileReference("Log1.txt");

        // Ensure that the file exists.
        if (file.Exists())
        {
            // Write the contents of the file to the console window.
            Console.WriteLine(file.DownloadTextAsync().Result);
        }
    }
}
```

Запустите консольное приложение и получите вывод.

---

## <a name="set-the-maximum-size-for-a-file-share"></a>Установка максимального размера для файлового ресурса

Начиная с версии 5. x клиентской библиотеки файлов Azure, можно задать квоту (максимальный размер) для общей папки. Можно также проверить, какой объем данных хранится в настоящее время в общей папке.

Задание квоты для общего ресурса ограничивает общий размер файлов, хранящихся в общей папке. Если общий размер файлов в общей папке превышает квоту, клиенты не могут увеличить размер существующих файлов. Клиенты также не могут создавать новые файлы, если эти файлы не являются пустыми.

В приведенном ниже примере показано, как проверить текущее использование данных в файловом ресурсе, а также задать для него квоту.

# <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_SetMaxShareSize":::

# <a name="net-v11"></a>[\..NET (версии 11)](#tab/dotnetv11)

```csharp
// Parse the connection string for the storage account.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create a CloudFileClient object for credentialed access to Azure Files.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Get a reference to the file share we created previously.
CloudFileShare share = fileClient.GetShareReference("logs");

// Ensure that the share exists.
if (share.Exists())
{
    // Check current usage stats for the share.
    // Note that the ShareStats object is part of the protocol layer for the File service.
    Microsoft.Azure.Storage.File.Protocol.ShareStats stats = share.GetStats();
    Console.WriteLine("Current share usage: {0} GB", stats.Usage.ToString());

    // Specify the maximum size of the share, in GB.
    // This line sets the quota to be 10 GB greater than the current usage of the share.
    share.Properties.Quota = 10 + stats.Usage;
    share.SetProperties();

    // Now check the quota for the share. Call FetchAttributes() to populate the share's properties.
    share.FetchAttributes();
    Console.WriteLine("Current share quota: {0} GB", share.Properties.Quota);
}
```

---

### <a name="generate-a-shared-access-signature-for-a-file-or-file-share"></a>Создание подписи общего доступа для файла или файлового ресурса

Начиная с версии 5. x клиентской библиотеки службы файлов Azure вы можете создать подписанный URL-адрес (SAS) для общей папки или отдельного файла.

# <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

В следующем примере метод возвращает SAS для файла в указанной общей папке.

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_GetFileSasUri":::

# <a name="net-v11"></a>[\..NET (версии 11)](#tab/dotnetv11)

Вы также можете создать хранимую политику доступа в общей папке для управления подписанными URL-доступом. Рекомендуется создать хранимую политику доступа, так как она позволяет отозвать SAS, если она будет скомпрометирована. В следующем примере создается хранимая политика доступа для общей папки. В примере эта политика используется для предоставления ограничений для SAS в файле в общей папке.

```csharp
// Parse the connection string for the storage account.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create a CloudFileClient object for credentialed access to Azure Files.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Get a reference to the file share we created previously.
CloudFileShare share = fileClient.GetShareReference("logs");

// Ensure that the share exists.
if (share.Exists())
{
    string policyName = "sampleSharePolicy" + DateTime.UtcNow.Ticks;

    // Create a new stored access policy and define its constraints.
    SharedAccessFilePolicy sharedPolicy = new SharedAccessFilePolicy()
        {
            SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
            Permissions = SharedAccessFilePermissions.Read | SharedAccessFilePermissions.Write
        };

    // Get existing permissions for the share.
    FileSharePermissions permissions = share.GetPermissions();

    // Add the stored access policy to the share's policies. Note that each policy must have a unique name.
    permissions.SharedAccessPolicies.Add(policyName, sharedPolicy);
    share.SetPermissions(permissions);

    // Generate a SAS for a file in the share and associate this access policy with it.
    CloudFileDirectory rootDir = share.GetRootDirectoryReference();
    CloudFileDirectory sampleDir = rootDir.GetDirectoryReference("CustomLogs");
    CloudFile file = sampleDir.GetFileReference("Log1.txt");
    string sasToken = file.GetSharedAccessSignature(null, policyName);
    Uri fileSasUri = new Uri(file.StorageUri.PrimaryUri.ToString() + sasToken);

    // Create a new CloudFile object from the SAS, and write some text to the file.
    CloudFile fileSas = new CloudFile(fileSasUri);
    fileSas.UploadText("This write operation is authorized via SAS.");
    Console.WriteLine(fileSas.DownloadText());
}
```

---

Дополнительные сведения о создании и использовании подписанных URL-отправок см. [в статье как работает подпись общего доступа](../common/storage-sas-overview.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json#how-a-shared-access-signature-works).

## <a name="copy-files"></a>Копирование файлов

Начиная с версии 5. x клиентской библиотеки службы файлов Azure можно скопировать файл в другой файл, файл в большой двоичный объект или большой двоичный объект в файл.

AzCopy также можно использовать для копирования одного файла в другой или для копирования большого двоичного объекта в файл или наоборот. См. подробнее о [начале работы с AzCopy](../common/storage-use-azcopy-v10.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

> [!NOTE]
> При копировании большого двоичного объекта в файл или файла в большой двоичный объект необходимо использовать подписанный URL-адрес (SAS) для авторизации доступа к исходному объекту, даже если копирование производится внутри одной и той же учетной записи хранения.

### <a name="copy-a-file-to-another-file"></a>Копирование файла в другой файл

В приведенном ниже примере файл копируется в другой файл в той же общей папке. Для копирования можно использовать [проверку подлинности с помощью общего ключа](/rest/api/storageservices/authorize-with-shared-key) , так как эта операция копирует файлы в одну и ту же учетную запись хранения.

# <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_CopyFile":::

# <a name="net-v11"></a>[\..NET (версии 11)](#tab/dotnetv11)

```csharp
// Parse the connection string for the storage account.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create a CloudFileClient object for credentialed access to Azure Files.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Get a reference to the file share we created previously.
CloudFileShare share = fileClient.GetShareReference("logs");

// Ensure that the share exists.
if (share.Exists())
{
    // Get a reference to the root directory for the share.
    CloudFileDirectory rootDir = share.GetRootDirectoryReference();

    // Get a reference to the directory we created previously.
    CloudFileDirectory sampleDir = rootDir.GetDirectoryReference("CustomLogs");

    // Ensure that the directory exists.
    if (sampleDir.Exists())
    {
        // Get a reference to the file we created previously.
        CloudFile sourceFile = sampleDir.GetFileReference("Log1.txt");

        // Ensure that the source file exists.
        if (sourceFile.Exists())
        {
            // Get a reference to the destination file.
            CloudFile destFile = sampleDir.GetFileReference("Log1Copy.txt");

            // Start the copy operation.
            destFile.StartCopy(sourceFile);

            // Write the contents of the destination file to the console window.
            Console.WriteLine(destFile.DownloadText());
        }
    }
}
```

---

### <a name="copy-a-file-to-a-blob"></a>Копирование файла в большой двоичный объект

В приведенном ниже примере файл создается и копируется в большой двоичный объект в пределах одной и той же учетной записи хранения. В примере для исходного файла создается SAS, который служба использует для авторизации доступа к исходному файлу во время операции копирования.

# <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_CopyFileToBlob":::

# <a name="net-v11"></a>[\..NET (версии 11)](#tab/dotnetv11)

```csharp
// Parse the connection string for the storage account.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create a CloudFileClient object for credentialed access to Azure Files.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Create a new file share, if it does not already exist.
CloudFileShare share = fileClient.GetShareReference("sample-share");
share.CreateIfNotExists();

// Create a new file in the root directory.
CloudFile sourceFile = share.GetRootDirectoryReference().GetFileReference("sample-file.txt");
sourceFile.UploadText("A sample file in the root directory.");

// Get a reference to the blob to which the file will be copied.
CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
CloudBlobContainer container = blobClient.GetContainerReference("sample-container");
container.CreateIfNotExists();
CloudBlockBlob destBlob = container.GetBlockBlobReference("sample-blob.txt");

// Create a SAS for the file that's valid for 24 hours.
// Note that when you are copying a file to a blob, or a blob to a file, you must use a SAS
// to authorize access to the source object, even if you are copying within the same
// storage account.
string fileSas = sourceFile.GetSharedAccessSignature(new SharedAccessFilePolicy()
{
    // Only read permissions are required for the source file.
    Permissions = SharedAccessFilePermissions.Read,
    SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24)
});

// Construct the URI to the source file, including the SAS token.
Uri fileSasUri = new Uri(sourceFile.StorageUri.PrimaryUri.ToString() + fileSas);

// Copy the file to the blob.
destBlob.StartCopy(fileSasUri);

// Write the contents of the file to the console window.
Console.WriteLine("Source file contents: {0}", sourceFile.DownloadText());
Console.WriteLine("Destination blob contents: {0}", destBlob.DownloadText());
```

---

Таким же образом можно скопировать BLOB-объект в файл. Если исходным объектом является большой двоичный объект, создайте SAS для авторизации доступа к этому объекту во время операции копирования.

## <a name="share-snapshots"></a>Моментальные снимки общих ресурсов

Начиная с версии 8,5 клиентской библиотеки файлов Azure, можно создать моментальный снимок общего ресурса. Можно также получить список моментальных снимков общих ресурсов, просмотреть и удалить их. После создания моментальные снимки общих ресурсов доступны только для чтения.

### <a name="create-share-snapshots"></a>Создание моментальных снимков общих ресурсов

В следующем примере создается моментальный снимок общего файлового ресурса.

# <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_CreateShareSnapshot":::

# <a name="net-v11"></a>[\..NET (версии 11)](#tab/dotnetv11)

```csharp
storageAccount = CloudStorageAccount.Parse(ConnectionString); 
fClient = storageAccount.CreateCloudFileClient(); 
string baseShareName = "myazurefileshare"; 
CloudFileShare myShare = fClient.GetShareReference(baseShareName); 
var snapshotShare = myShare.Snapshot();

```

---

### <a name="list-share-snapshots"></a>Вывод списка моментальных снимков общих ресурсов

В следующем примере перечисляются моментальные снимки в общей папке.

# <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_ListShareSnapshots":::

# <a name="net-v11"></a>[\..NET (версии 11)](#tab/dotnetv11)

```csharp
var shares = fClient.ListShares(baseShareName, ShareListingDetails.All);
```

---

### <a name="list-files-and-directories-within-share-snapshots"></a>Вывод списка файлов и каталогов в моментальных снимках общего ресурса

В следующем примере производится просмотр файлов и каталогов в моментальных снимках общего ресурса.

# <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_ListSnapshotContents":::

# <a name="net-v11"></a>[\..NET (версии 11)](#tab/dotnetv11)

```csharp
CloudFileShare mySnapshot = fClient.GetShareReference(baseShareName, snapshotTime); 
var rootDirectory = mySnapshot.GetRootDirectoryReference(); 
var items = rootDirectory.ListFilesAndDirectories();
```

---

### <a name="restore-file-shares-or-files-from-share-snapshots"></a>Восстановление общих файловых ресурсов или файлов из моментальных снимков общего ресурса

Создание моментального снимка общей папки позволяет восстанавливать отдельные файлы или всю общую папку.

Файл можно восстановить из моментального снимка, сделав запрос на моментальные снимки файлового ресурса. Затем можно получить файл, принадлежащий определенному моментальному снимку общего ресурса. Используйте эту версию для непосредственного чтения или восстановления файла.

# <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_RestoreFileFromSnapshot":::

# <a name="net-v11"></a>[\..NET (версии 11)](#tab/dotnetv11)

```csharp
CloudFileShare liveShare = fClient.GetShareReference(baseShareName);
var rootDirOfliveShare = liveShare.GetRootDirectoryReference();
var dirInliveShare = rootDirOfliveShare.GetDirectoryReference(dirName);
var fileInliveShare = dirInliveShare.GetFileReference(fileName);

CloudFileShare snapshot = fClient.GetShareReference(baseShareName, snapshotTime);
var rootDirOfSnapshot = snapshot.GetRootDirectoryReference();
var dirInSnapshot = rootDirOfSnapshot.GetDirectoryReference(dirName);
var fileInSnapshot = dir1InSnapshot.GetFileReference(fileName);

string sasContainerToken = string.Empty;
SharedAccessFilePolicy sasConstraints = new SharedAccessFilePolicy();
sasConstraints.SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24);
sasConstraints.Permissions = SharedAccessFilePermissions.Read;

//Generate the shared access signature on the container, setting the constraints directly on the signature.
sasContainerToken = fileInSnapshot.GetSharedAccessSignature(sasConstraints);

string sourceUri = (fileInSnapshot.Uri.ToString() + sasContainerToken + "&" + fileInSnapshot.SnapshotTime.ToString()); ;
fileInliveShare.StartCopyAsync(new Uri(sourceUri));
```

---

### <a name="delete-share-snapshots"></a>Удаление моментальных снимков общих ресурсов

В следующем примере удаляется моментальный снимок общего файлового ресурса.

# <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_DeleteSnapshot":::

# <a name="net-v11"></a>[\..NET (версии 11)](#tab/dotnetv11)

```csharp
CloudFileShare mySnapshot = fClient.GetShareReference(baseShareName, snapshotTime); mySnapshot.Delete(null, null, null);
```

---

## <a name="troubleshoot-azure-files-by-using-metrics"></a>Устранение неполадок службы файлов Azure с помощью метрик<a name="troubleshooting-azure-files-using-metrics"></a>

Аналитика Службы хранилища Azure поддерживает метрики для службы файлов Azure. Данные метрик позволяют отслеживать запросы и диагностировать проблемы.

Вы можете включить метрики для службы файлов Azure из [портал Azure](https://portal.azure.com). Вы также можете включить метрики программно, вызвав операцию [Set File Service Properties](/rest/api/storageservices/set-file-service-properties) с REST API или одним из его аналогов в клиентской библиотеке файлов Azure.

В следующем примере кода показано, как использовать клиентскую библиотеку .NET для включения метрик для службы "файлы Azure".

# <a name="net-v12"></a>[\..NET (версии 12)](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/files/howto/dotnet/dotnet-v12/FileShare.cs" id="snippet_UseMetrics":::

# <a name="net-v11"></a>[\..NET (версии 11)](#tab/dotnetv11)

Сначала добавьте следующие `using` директивы в файл *Program. CS* вместе с добавленными выше.

```csharp
using Microsoft.Azure.Storage.File.Protocol;
using Microsoft.Azure.Storage.Shared.Protocol;
```

Хотя большие двоичные объекты Azure, таблицы Azure и очереди Azure используют общий `ServiceProperties` тип в `Microsoft.Azure.Storage.Shared.Protocol` пространстве имен, службы файлов Azure используют собственный тип, `FileServiceProperties` тип в `Microsoft.Azure.Storage.File.Protocol` пространстве имен. Однако для компиляции следующего кода необходимо сослаться на оба пространства имен из кода.

```csharp
// Parse your storage connection string from your application's configuration file.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));
// Create the File service client.
CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

// Set metrics properties for File service.
// Note that the File service currently uses its own service properties type,
// available in the Microsoft.Azure.Storage.File.Protocol namespace.
fileClient.SetServiceProperties(new FileServiceProperties()
{
    // Set hour metrics
    HourMetrics = new MetricsProperties()
    {
        MetricsLevel = MetricsLevel.ServiceAndApi,
        RetentionDays = 14,
        Version = "1.0"
    },
    // Set minute metrics
    MinuteMetrics = new MetricsProperties()
    {
        MetricsLevel = MetricsLevel.ServiceAndApi,
        RetentionDays = 7,
        Version = "1.0"
    }
});

// Read the metrics properties we just set.
FileServiceProperties serviceProperties = fileClient.GetServiceProperties();
Console.WriteLine("Hour metrics:");
Console.WriteLine(serviceProperties.HourMetrics.MetricsLevel);
Console.WriteLine(serviceProperties.HourMetrics.RetentionDays);
Console.WriteLine(serviceProperties.HourMetrics.Version);
Console.WriteLine();
Console.WriteLine("Minute metrics:");
Console.WriteLine(serviceProperties.MinuteMetrics.MetricsLevel);
Console.WriteLine(serviceProperties.MinuteMetrics.RetentionDays);
Console.WriteLine(serviceProperties.MinuteMetrics.Version);
```

---

Если возникнут проблемы, см. статью [Устранение неполадок с файлами Azure в Windows](storage-troubleshoot-windows-file-connection-problems.md).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о службе файлов Azure см. в следующих ресурсах:

### <a name="conceptual-articles-and-videos"></a>Тематические статьи и видео

- [Файлы Azure: удобная облачная файловая система SMB для Windows и Linux](https://azure.microsoft.com/documentation/videos/azurecon-2015-azure-files-storage-a-frictionless-cloud-smb-file-system-for-windows-and-linux/)
- [Использование Файлов Azure в Linux](storage-how-to-use-files-linux.md)

### <a name="tooling-support-for-file-storage"></a>Средства для работы с хранилищем файлов

- [Начало работы с AzCopy](../common/storage-use-azcopy-v10.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)
- [Устранение неполадок службы файлов Azure в Windows](./storage-troubleshoot-windows-file-connection-problems.md)

### <a name="reference"></a>Справочник

- [API-интерфейсы службы хранилища Azure для .NET](/dotnet/api/overview/azure/storage)
- [REST API файловой службы](/rest/api/storageservices/File-Service-REST-API)