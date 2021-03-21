---
title: Перенос данных с помощью библиотеки перемещения данных для .NET
titleSuffix: Azure Storage
description: Используйте библиотеку перемещения данных для перемещения или копирования данных в BLOB-объект и содержимое файла. Копируйте данные в хранилище Azure из локальных файлов, а также внутри учетной записи хранения и из одной такой учетной записи в другую. Легко переносите данные в хранилище Azure.
services: storage
author: tamram
ms.service: storage
ms.devlang: dotnet
ms.topic: how-to
ms.date: 06/16/2020
ms.author: tamram
ms.subservice: common
ms.custom: devx-track-csharp
ms.openlocfilehash: 8ec35a651d4d3ef9e0877463329a654bc7491f4c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96023420"
---
# <a name="transfer-data-with-the-data-movement-library"></a>Перенос данных с помощью библиотеки перемещения данных

Библиотека перемещения данных службы хранилища Azure — это кросс-платформенная библиотека с открытым исходным кодом, предназначенная для обеспечения высокой производительности при передаче, скачивании и копировании больших двоичных объектов и файлов. Библиотека перемещения данных предоставляет удобные методы, которые недоступны в клиентской библиотеке службы хранилища Azure для .NET. Эти методы предоставляют возможность задать количество параллельных операций, отслеживать ход передачи, легко возобновить отмененную операцию передачи и многое другое.

Эта библиотека также использует .NET Core, а это означает, что ее можно применять при создании приложений .NET для Windows, Linux и MacOS. Дополнительные сведения о .NET Core см. [документации по .NET Core](https://dotnet.github.io/). Эта библиотека также подходит для традиционных приложений .NET Framework для Windows.

В этом документе показано, как создать консольное приложение .NET Core, которое работает в Windows, Linux и MacOS и выполняет приведенные ниже функции.

- Передача файлов и каталогов в хранилище BLOB-объектов.
- Определение количества параллельных операций при передаче данных.
- Отслеживание выполнения передачи данных.
- Возобновление отмененной передачи данных.
- Копирование файла по URL-адресу в хранилище BLOB-объектов.
- Копирование из хранилища BLOB-объектов в хранилище BLOB-объектов.

## <a name="prerequisites"></a>Предварительные требования

- [Visual Studio Code](https://code.visualstudio.com/)
- [Учетная запись хранения Azure](storage-account-create.md) ;

## <a name="setup"></a>Настройка

1. Ознакомьтесь с [руководством по установке .NET Core](https://www.microsoft.com/net/core), чтобы установить .NET Core. При выборе среды щелкните параметр командной строки.
2. В командной строке создайте каталог для проекта. Перейдите в этот каталог и введите `dotnet new console -o <sample-project-name>` для создания проекта консольного приложения C#.
3. Откройте этот каталог в Visual Studio Code. Этот шаг можно быстро выполнить, введя в командной строке `code .` в Windows.
4. Установите [расширение C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) из Visual Studio Code Marketplace. Перезапустите Visual Studio Code.
5. На этом этапе должны отобразиться два запроса. Первый — для добавления необходимых ресурсов для сборки. Щелкните "Да". Второй запрос — для восстановления неразрешенных зависимостей. Щелкните "Восстановить".
6. Измените `launch.json` в `.vscode`, чтобы использовать внешний терминал в качестве консоли. Этот параметр следует читать как `"console": "externalTerminal"`.
7. Visual Studio Code позволяет отлаживать приложения .NET Core. Нажмите клавишу `F5`, чтобы запустить приложение и проверить настройки. Вы должны увидеть текст "Hello World!" в консоли.

## <a name="add-the-data-movement-library-to-your-project"></a>Добавление библиотеки перемещения данных в проект

1. Добавьте последнюю версию библиотеки перемещения данных в `dependencies` раздел `<project-name>.csproj` файла. На момент написания статьи это версия `"Microsoft.Azure.Storage.DataMovement": "0.6.2"`.
2. Должен появиться запрос на восстановление проекта. Нажмите кнопку "Восстановить". Можно также восстановить проект из командной строки, введя команду `dotnet restore` в корневом каталоге проекта.

Измените `<project-name>.csproj`.

```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>netcoreapp2.0</TargetFramework>
    </PropertyGroup>
    <ItemGroup>
        <PackageReference Include="Microsoft.Azure.Storage.DataMovement" Version="0.6.2" />
        </ItemGroup>
    </Project>
```

## <a name="set-up-the-skeleton-of-your-application"></a>Настройка схемы приложения

Первое, что мы сделаем — настроим схему кода приложения. Этот код запрашивает имя и ключ учетной записи хранения и использует эти учетные данные для создания объекта `CloudStorageAccount`. Этот объект используется для взаимодействия с нашей учетной записи хранения во всех сценариях передачи данных. Код также предлагает выбрать тип операции передачи, которую нужно выполнить.

Измените `Program.cs`.

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;
using System.Diagnostics;
using Microsoft.Azure.Storage;
using Microsoft.Azure.Storage.Blob;
using Microsoft.Azure.Storage.DataMovement;

namespace DMLibSample
{
    public class Program
    {
        public static void Main()
        {
            Console.WriteLine("Enter Storage account name:");
            string accountName = Console.ReadLine();

            Console.WriteLine("\nEnter Storage account key:");
            string accountKey = Console.ReadLine();

            string storageConnectionString = "DefaultEndpointsProtocol=https;AccountName=" + accountName + ";AccountKey=" + accountKey;
            CloudStorageAccount account = CloudStorageAccount.Parse(storageConnectionString);

            ExecuteChoice(account);
        }

        public static void ExecuteChoice(CloudStorageAccount account)
        {
            Console.WriteLine("\nWhat type of transfer would you like to execute?\n1. Local file --> Azure Blob\n2. Local directory --> Azure Blob directory\n3. URL (e.g. Amazon S3 file) --> Azure Blob\n4. Azure Blob --> Azure Blob");
            int choice = int.Parse(Console.ReadLine());

            if(choice == 1)
            {
                TransferLocalFileToAzureBlob(account).Wait();
            }
            else if(choice == 2)
            {
                TransferLocalDirectoryToAzureBlobDirectory(account).Wait();
            }
            else if(choice == 3)
            {
                TransferUrlToAzureBlob(account).Wait();
            }
            else if(choice == 4)
            {
                TransferAzureBlobToAzureBlob(account).Wait();
            }
        }

        public static async Task TransferLocalFileToAzureBlob(CloudStorageAccount account)
        {

        }

        public static async Task TransferLocalDirectoryToAzureBlobDirectory(CloudStorageAccount account)
        {

        }

        public static async Task TransferUrlToAzureBlob(CloudStorageAccount account)
        {

        }

        public static async Task TransferAzureBlobToAzureBlob(CloudStorageAccount account)
        {

        }
    }
}
```

## <a name="upload-a-local-file-to-a-blob"></a>Передача локального файла в большой двоичный объект

Добавьте методы `GetSourcePath` и `GetBlob` в `Program.cs`.

```csharp
public static string GetSourcePath()
{
    Console.WriteLine("\nProvide path for source:");
    string sourcePath = Console.ReadLine();

    return sourcePath;
}

public static CloudBlockBlob GetBlob(CloudStorageAccount account)
{
    CloudBlobClient blobClient = account.CreateCloudBlobClient();

    Console.WriteLine("\nProvide name of Blob container:");
    string containerName = Console.ReadLine();
    CloudBlobContainer container = blobClient.GetContainerReference(containerName);
    container.CreateIfNotExistsAsync().Wait();

    Console.WriteLine("\nProvide name of new Blob:");
    string blobName = Console.ReadLine();
    CloudBlockBlob blob = container.GetBlockBlobReference(blobName);

    return blob;
}
```

Измените метод `TransferLocalFileToAzureBlob`.

```csharp
public static async Task TransferLocalFileToAzureBlob(CloudStorageAccount account)
{
    string localFilePath = GetSourcePath();
    CloudBlockBlob blob = GetBlob(account);
    Console.WriteLine("\nTransfer started...");
    await TransferManager.UploadAsync(localFilePath, blob);
    Console.WriteLine("\nTransfer operation complete.");
    ExecuteChoice(account);
}
```

Этот код запрашивает путь к локальному файлу, имя нового или существующего контейнера и имя нового большого двоичного объекта. На основе этой информации метод `TransferManager.UploadAsync` выполняет передачу.

Нажмите клавишу `F5`, чтобы запустить приложение. Чтобы проверить, выполнена ли передача, можно просмотреть свою учетную запись хранения в [Microsoft Azure Storage Explorer](https://storageexplorer.com/).

## <a name="set-the-number-of-parallel-operations"></a>Задание числа параллельных операций

Одной из функций, предлагаемых библиотекой перемещения данных, является возможность задать количество параллельных операций для увеличения пропускной способности передачи данных. По умолчанию библиотека перемещения данных задает для числа параллельных операций значение 8 * число ядер на компьютере.

Следует помнить, что большое количество одновременных операций в среде с низкой пропускной способностью может перегрузить сетевое подключение и фактически помешать полному завершению операций. Вам потребуется поэкспериментировать с этим параметром, чтобы определить, что лучше всего подходит для имеющейся пропускной способности сети.

Давайте добавим код, который позволяет задать число параллельных операций. Также добавим код, определяющий допустимую длительность передачи.

Добавьте `SetNumberOfParallelOperations` в метод `Program.cs`.

```csharp
public static void SetNumberOfParallelOperations()
{
    Console.WriteLine("\nHow many parallel operations would you like to use?");
    string parallelOperations = Console.ReadLine();
    TransferManager.Configurations.ParallelOperations = int.Parse(parallelOperations);
}
```

Измените метод `ExecuteChoice`, чтобы использовать `SetNumberOfParallelOperations`.

```csharp
public static void ExecuteChoice(CloudStorageAccount account)
{
    Console.WriteLine("\nWhat type of transfer would you like to execute?\n1. Local file --> Azure Blob\n2. Local directory --> Azure Blob directory\n3. URL (e.g. Amazon S3 file) --> Azure Blob\n4. Azure Blob --> Azure Blob");
    int choice = int.Parse(Console.ReadLine());

    SetNumberOfParallelOperations();

    if(choice == 1)
    {
        TransferLocalFileToAzureBlob(account).Wait();
    }
    else if(choice == 2)
    {
        TransferLocalDirectoryToAzureBlobDirectory(account).Wait();
    }
    else if(choice == 3)
    {
        TransferUrlToAzureBlob(account).Wait();
    }
    else if(choice == 4)
    {
        TransferAzureBlobToAzureBlob(account).Wait();
    }
}
```

Измените метод `TransferLocalFileToAzureBlob`, чтобы использовать таймер.

```csharp
public static async Task TransferLocalFileToAzureBlob(CloudStorageAccount account)
{
    string localFilePath = GetSourcePath();
    CloudBlockBlob blob = GetBlob(account);
    Console.WriteLine("\nTransfer started...");
    Stopwatch stopWatch = Stopwatch.StartNew();
    await TransferManager.UploadAsync(localFilePath, blob);
    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

## <a name="track-transfer-progress"></a>Отслеживание выполнения передачи

Знание времени, затраченного на пересылку данных, полезно. Тем не менее, возможность видеть ход перемещения *во время* операции перемещения будет еще лучше. Чтобы получить такую возможность, необходимо создать объект `TransferContext`. Объект `TransferContext` бывает двух типов: `SingleTransferContext` и `DirectoryTransferContext`. Первый из них предназначен для передачи одного файла, а второй — для передачи каталога файлов.

Добавьте методы `GetSingleTransferContext` и `GetDirectoryTransferContext` в `Program.cs`.

```csharp
public static SingleTransferContext GetSingleTransferContext(TransferCheckpoint checkpoint)
{
    SingleTransferContext context = new SingleTransferContext(checkpoint);

    context.ProgressHandler = new Progress<TransferStatus>((progress) =>
    {
        Console.Write("\rBytes transferred: {0}", progress.BytesTransferred );
    });

    return context;
}

public static DirectoryTransferContext GetDirectoryTransferContext(TransferCheckpoint checkpoint)
{
    DirectoryTransferContext context = new DirectoryTransferContext(checkpoint);

    context.ProgressHandler = new Progress<TransferStatus>((progress) =>
    {
        Console.Write("\rBytes transferred: {0}", progress.BytesTransferred );
    });

    return context;
}
```

Измените метод `TransferLocalFileToAzureBlob`, чтобы использовать `GetSingleTransferContext`.

```csharp
public static async Task TransferLocalFileToAzureBlob(CloudStorageAccount account)
{
    string localFilePath = GetSourcePath();
    CloudBlockBlob blob = GetBlob(account);
    TransferCheckpoint checkpoint = null;
    SingleTransferContext context = GetSingleTransferContext(checkpoint);
    Console.WriteLine("\nTransfer started...\n");
    Stopwatch stopWatch = Stopwatch.StartNew();
    await TransferManager.UploadAsync(localFilePath, blob, null, context);
    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

## <a name="resume-a-canceled-transfer"></a>Возобновление отмененной передачи

Еще одна удобная функция, предоставляемая библиотекой перемещения данных, — это возможность возобновить отмененную передачу. Давайте добавим код, позволяющий временно отменить передачу, введя `c`, а через 3 секунды возобновить ее.

Измените `TransferLocalFileToAzureBlob`.

```csharp
public static async Task TransferLocalFileToAzureBlob(CloudStorageAccount account)
{
    string localFilePath = GetSourcePath();
    CloudBlockBlob blob = GetBlob(account);
    TransferCheckpoint checkpoint = null;
    SingleTransferContext context = GetSingleTransferContext(checkpoint);
    CancellationTokenSource cancellationSource = new CancellationTokenSource();
    Console.WriteLine("\nTransfer started...\nPress 'c' to temporarily cancel your transfer...\n");

    Stopwatch stopWatch = Stopwatch.StartNew();
    Task task;
    ConsoleKeyInfo keyinfo;
    try
    {
        task = TransferManager.UploadAsync(localFilePath, blob, null, context, cancellationSource.Token);
        while(!task.IsCompleted)
        {
            if(Console.KeyAvailable)
            {
                keyinfo = Console.ReadKey(true);
                if(keyinfo.Key == ConsoleKey.C)
                {
                    cancellationSource.Cancel();
                }
            }
        }
        await task;
    }
    catch(Exception e)
    {
        Console.WriteLine("\nThe transfer is canceled: {0}", e.Message);
    }

    if(cancellationSource.IsCancellationRequested)
    {
        Console.WriteLine("\nTransfer will resume in 3 seconds...");
        Thread.Sleep(3000);
        checkpoint = context.LastCheckpoint;
        context = GetSingleTransferContext(checkpoint);
        Console.WriteLine("\nResuming transfer...\n");
        await TransferManager.UploadAsync(localFilePath, blob, null, context);
    }

    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

До сих пор параметр `checkpoint` всегда имел значение `null`. Теперь, если мы отменим передачу, то получим ее последнюю контрольную точку, а затем используем эту новую контрольную точку в контексте передачи.

## <a name="transfer-a-local-directory-to-blob-storage"></a>Перенос локального каталога в хранилище BLOB-объектов

Это было бы разочаровать, если библиотека перемещения данных может передавать только один файл за раз. К счастью, это не так. Библиотека перемещения данных предоставляет возможность передачи каталога файлов и всех его подкаталогов. Давайте добавим код, позволяющий сделать это.

Сначала добавьте метод `GetBlobDirectory` в `Program.cs`.

```csharp
public static CloudBlobDirectory GetBlobDirectory(CloudStorageAccount account)
{
    CloudBlobClient blobClient = account.CreateCloudBlobClient();

    Console.WriteLine("\nProvide name of Blob container. This can be a new or existing Blob container:");
    string containerName = Console.ReadLine();
    CloudBlobContainer container = blobClient.GetContainerReference(containerName);
    container.CreateIfNotExistsAsync().Wait();

    CloudBlobDirectory blobDirectory = container.GetDirectoryReference("");

    return blobDirectory;
}
```

Затем измените `TransferLocalDirectoryToAzureBlobDirectory`.

```csharp
public static async Task TransferLocalDirectoryToAzureBlobDirectory(CloudStorageAccount account)
{
    string localDirectoryPath = GetSourcePath();
    CloudBlobDirectory blobDirectory = GetBlobDirectory(account);
    TransferCheckpoint checkpoint = null;
    DirectoryTransferContext context = GetDirectoryTransferContext(checkpoint);
    CancellationTokenSource cancellationSource = new CancellationTokenSource();
    Console.WriteLine("\nTransfer started...\nPress 'c' to temporarily cancel your transfer...\n");

    Stopwatch stopWatch = Stopwatch.StartNew();
    Task task;
    ConsoleKeyInfo keyinfo;
    UploadDirectoryOptions options = new UploadDirectoryOptions()
    {
        Recursive = true
    };

    try
    {
        task = TransferManager.UploadDirectoryAsync(localDirectoryPath, blobDirectory, options, context, cancellationSource.Token);
        while(!task.IsCompleted)
        {
            if(Console.KeyAvailable)
            {
                keyinfo = Console.ReadKey(true);
                if(keyinfo.Key == ConsoleKey.C)
                {
                    cancellationSource.Cancel();
                }
            }
        }
        await task;
    }
    catch(Exception e)
    {
        Console.WriteLine("\nThe transfer is canceled: {0}", e.Message);
    }

    if(cancellationSource.IsCancellationRequested)
    {
        Console.WriteLine("\nTransfer will resume in 3 seconds...");
        Thread.Sleep(3000);
        checkpoint = context.LastCheckpoint;
        context = GetDirectoryTransferContext(checkpoint);
        Console.WriteLine("\nResuming transfer...\n");
        await TransferManager.UploadDirectoryAsync(localDirectoryPath, blobDirectory, options, context);
    }

    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

Существует ряд различий между этим методом и методом для передачи отдельного файла. Сейчас мы используем методы `TransferManager.UploadDirectoryAsync` и `getDirectoryTransferContext`, созданные ранее. Кроме того, мы задаем значение `options` для нашей операции передачи. Это позволяет указать, что в нее нужно включить подкаталоги.

## <a name="copy-a-file-from-url-to-a-blob"></a>Копирование файла из URL-адреса в большой двоичный объект

Теперь давайте добавим код, который позволяет скопировать файл по URL-адресу в большой двоичный объект Azure.

Измените `TransferUrlToAzureBlob`.

```csharp
public static async Task TransferUrlToAzureBlob(CloudStorageAccount account)
{
    Uri uri = new Uri(GetSourcePath());
    CloudBlockBlob blob = GetBlob(account);
    TransferCheckpoint checkpoint = null;
    SingleTransferContext context = GetSingleTransferContext(checkpoint);
    CancellationTokenSource cancellationSource = new CancellationTokenSource();
    Console.WriteLine("\nTransfer started...\nPress 'c' to temporarily cancel your transfer...\n");

    Stopwatch stopWatch = Stopwatch.StartNew();
    Task task;
    ConsoleKeyInfo keyinfo;
    try
    {
        task = TransferManager.CopyAsync(uri, blob, true, null, context, cancellationSource.Token);
        while(!task.IsCompleted)
        {
            if(Console.KeyAvailable)
            {
                keyinfo = Console.ReadKey(true);
                if(keyinfo.Key == ConsoleKey.C)
                {
                    cancellationSource.Cancel();
                }
            }
        }
        await task;
    }
    catch(Exception e)
    {
        Console.WriteLine("\nThe transfer is canceled: {0}", e.Message);
    }

    if(cancellationSource.IsCancellationRequested)
    {
        Console.WriteLine("\nTransfer will resume in 3 seconds...");
        Thread.Sleep(3000);
        checkpoint = context.LastCheckpoint;
        context = GetSingleTransferContext(checkpoint);
        Console.WriteLine("\nResuming transfer...\n");
        await TransferManager.CopyAsync(uri, blob, true, null, context, cancellationSource.Token);
    }

    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

Одним из важных применений этой функции является перемещение данных из другой облачной службы (например, AWS) в Azure. Если у вас есть URL-адрес, который предоставляет доступ к ресурсу, то этот ресурс можно легко переместить в большие двоичные объекты Azure, воспользовавшись методом `TransferManager.CopyAsync`. В этом методе также представлен новый логический параметр. Если присвоить ему значение `true`, то будет выполнено асинхронное копирование на стороне сервера. Если присвоить ему значение `false`, то будет выполнено синхронное копирование, то есть ресурс сначала скачивается на локальный компьютер, а затем передается в большой двоичный объект Azure. В данный момент синхронное копирование доступно только для копирования между ресурсами службы хранилища Azure.

## <a name="copy-a-blob"></a>Копирование большого двоичного объекта

Другой функцией, которая уникально предоставляется библиотекой перемещения данных, является возможность копирования из одного ресурса хранилища Azure в другой.

Измените `TransferAzureBlobToAzureBlob`.

```csharp
public static async Task TransferAzureBlobToAzureBlob(CloudStorageAccount account)
{
    CloudBlockBlob sourceBlob = GetBlob(account);
    CloudBlockBlob destinationBlob = GetBlob(account);
    TransferCheckpoint checkpoint = null;
    SingleTransferContext context = GetSingleTransferContext(checkpoint);
    CancellationTokenSource cancellationSource = new CancellationTokenSource();
    Console.WriteLine("\nTransfer started...\nPress 'c' to temporarily cancel your transfer...\n");

    Stopwatch stopWatch = Stopwatch.StartNew();
    Task task;
    ConsoleKeyInfo keyinfo;
    try
    {
        task = TransferManager.CopyAsync(sourceBlob, destinationBlob, CopyMethod.ServiceSideAsyncCopy, null, context, cancellationSource.Token);
        while(!task.IsCompleted)
        {
            if(Console.KeyAvailable)
            {
                keyinfo = Console.ReadKey(true);
                if(keyinfo.Key == ConsoleKey.C)
                {
                    cancellationSource.Cancel();
                }
            }
        }
        await task;
    }
    catch(Exception e)
    {
        Console.WriteLine("\nThe transfer is canceled: {0}", e.Message);
    }

    if(cancellationSource.IsCancellationRequested)
    {
        Console.WriteLine("\nTransfer will resume in 3 seconds...");
        Thread.Sleep(3000);
        checkpoint = context.LastCheckpoint;
        context = GetSingleTransferContext(checkpoint);
        Console.WriteLine("\nResuming transfer...\n");
        await TransferManager.CopyAsync(sourceBlob, destinationBlob, false, null, context, cancellationSource.Token);
    }

    stopWatch.Stop();
    Console.WriteLine("\nTransfer operation completed in " + stopWatch.Elapsed.TotalSeconds + " seconds.");
    ExecuteChoice(account);
}
```

В этом примере мы присваиваем логическому параметру в `TransferManager.CopyAsync` значение `false`, чтобы указать, что нам нужно выполнить синхронное копирование. Это означает, что ресурс сначала скачивается на локальный компьютер, а затем передается в большой двоичный объект Azure. Синхронное копирование — отличный способ обеспечить постоянную скорость копирования. Напротив, скорость асинхронного копирования на стороне сервера зависит от доступной пропускной способности на сервере, которая может меняться. Однако синхронное копирование может повлечь дополнительные затраты на исходящий трафик по сравнению с асинхронным копированием. Во избежание таких затрат мы советуем использовать синхронное копирование в виртуальных машинах Azure, которые находятся в одном регионе с вашей учетной записью хранения.

Теперь приложение перемещения данных завершено. [Полный пример кода можно найти на сайте GitHub](https://github.com/azure-samples/storage-dotnet-data-movement-library-app).

## <a name="next-steps"></a>Дальнейшие действия

[Справочная документация по библиотеке перемещения данных службы хранилища Azure](https://azure.github.io/azure-storage-net-data-movement).

[!INCLUDE [storage-try-azure-tools-blobs](../../../includes/storage-try-azure-tools-blobs.md)]
