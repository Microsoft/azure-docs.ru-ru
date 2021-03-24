---
title: Руководство. Запуск параллельной рабочей нагрузки с помощью API .NET
description: Руководство. Перекодировка медиафайлов параллельно с ffmpeg в пакетной службе с помощью клиентской библиотеки пакетной службы для .NET
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 09/29/2020
ms.custom: mvc, devx-track-csharp
ms.openlocfilehash: a990a5480a8a6462bb6ef9f84070b78768628fd0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97106552"
---
# <a name="tutorial-run-a-parallel-workload-with-azure-batch-using-the-net-api"></a>Руководство по Запуск параллельной рабочей нагрузки с помощью пакетной службы Azure с использованием .NET API

Используйте пакетную службу Azure, чтобы эффективно выполнять пакетные задания для крупномасштабных параллельных и высокопроизводительных вычислений (HPC). В этом руководстве рассматривается пример C# для запуска параллельной рабочей нагрузки с помощью пакетной службы Azure. Вы изучите общий рабочий процесс приложения пакетной службы и узнаете, как программно взаимодействовать с ресурсами пакетной службы и службы хранилища. Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Добавление пакета приложения в учетную запись пакетной службы.
> * Проверка подлинности с помощью учетных записей хранения и пакетной службы.
> * Передача входных файлов в хранилище.
> * Создание пула вычислительных узлов для запуска приложения.
> * Создание задания и задач для обработки входных файлов.
> * Мониторинг выполнения задач.
> * Извлечение выходных файлов

В этом руководстве вы преобразовываете медиафайлы формата MP4 в формат MP3 с помощью инструмента с открытым кодом [ffmpeg](https://ffmpeg.org/). 

[!INCLUDE [quickstarts-free-trial-note.md](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

* [Visual Studio 2017 или более поздней версии](https://www.visualstudio.com/vs), или [.NET Core 2.1](https://www.microsoft.com/net/download/dotnet-core/2.1) для Linux, macOS или Windows.

* учетная запись пакетной службы и связанная учетная запись службы хранилища Azure. Чтобы создать эти учетные записи, см. примеры быстрого начала работы с пакетной службой с помощью [портала Azure](quick-create-portal.md) или [Azure CLI](quick-create-cli.md).

* [64-разрядная версия ffmpeg 4.3.1 для Windows](https://github.com/GyanD/codexffmpeg/releases/tag/4.3.1-2020-11-08) (.zip). Загрузите ZIP-файл на локальный компьютер. Для этого руководства вам нужен только ZIP-файл. Вам не нужно распаковывать или локально устанавливать файл.

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите на портал Azure по адресу [https://portal.azure.com](https://portal.azure.com).

## <a name="add-an-application-package"></a>Добавление пакета приложения

Добавьте ffmpeg в учетную запись пакетной службы в качестве [пакета приложений](batch-application-packages.md) с помощью портала Azure. Пакеты приложений упрощают управление приложениями задач и их развертывание на вычислительных узлах в пуле. 

1. На портале Azure выберите **Дополнительные службы** > **Учетные записи пакетной службы** и щелкните имя учетной записи пакетной службы.
3. Выберите **Приложения** > **Добавить**.
4. В поле **Идентификатор приложения** введите *ffmpeg* и версию пакета *4.3.1*. Выберите файл zfmpeg zip, который вы загрузили ранее, и нажмите кнопку **ОК**. Пакет приложений ffmpeg добавляется в учетную запись пакетной службы.

![Добавление пакета приложения](./media/tutorial-parallel-dotnet/add-application.png)

[!INCLUDE [batch-common-credentials](../../includes/batch-common-credentials.md)]

## <a name="download-and-run-the-sample"></a>Скачивание и запуск примера приложения

### <a name="download-the-sample"></a>Скачивание примера приложения

[Скачайте или клонируйте пример приложения](https://github.com/Azure-Samples/batch-dotnet-ffmpeg-tutorial) с GitHub. Чтобы клонировать пример репозитория приложения с клиентом Git, выполните следующую команду:

```
git clone https://github.com/Azure-Samples/batch-dotnet-ffmpeg-tutorial.git
```

Перейдите в каталог, в котором содержится файл решения Visual Studio `BatchDotNetFfmpegTutorial.sln`.

Откройте файл решения в Visual Studio и замените строки учетных данных в `Program.cs` значениями, полученными для своих учетных записей. Пример:

```csharp
// Batch account credentials
private const string BatchAccountName = "mybatchaccount";
private const string BatchAccountKey  = "xxxxxxxxxxxxxxxxE+yXrRvJAqT9BlXwwo1CwF+SwAYOxxxxxxxxxxxxxxxx43pXi/gdiATkvbpLRl3x14pcEQ==";
private const string BatchAccountUrl  = "https://mybatchaccount.mybatchregion.batch.azure.com";

// Storage account credentials
private const string StorageAccountName = "mystorageaccount";
private const string StorageAccountKey  = "xxxxxxxxxxxxxxxxy4/xxxxxxxxxxxxxxxxfwpbIC5aAWA8wDu+AFXZB827Mt9lybZB1nUcQbQiUrkPtilK5BQ==";
```

[!INCLUDE [batch-credentials-include](../../includes/batch-credentials-include.md)]

Кроме того, убедитесь, что ссылка на пакет приложения ffmpeg в решении совпадает с идентификатором и версией пакета ffmpeg, который вы отправили в свою учетную запись.

```csharp
const string appPackageId = "ffmpeg";
const string appPackageVersion = "4.3.1";
```

### <a name="build-and-run-the-sample-project"></a>Создание и запуск примера проекта

Создавайте и запускайте приложения в Visual Studio или с помощью команд `dotnet build` и `dotnet run` в командной строке. После запуска приложения просмотрите код, чтобы узнать, как работает каждый компонент приложения. Visual Studio:

* Щелкните правой кнопкой мыши решение в обозревателе решений и выберите **Build Solution** (Сборка решения). 

* Подтвердите восстановление пакетов NuGet, если появится соответствующий запрос. Если вам необходимо скачать отсутствующие пакеты, установите [диспетчер пакетов NuGet](https://docs.nuget.org/consume/installing-nuget).

Затем запустите его. Когда вы запустите пример приложения, консоль будет выглядеть так. Во время выполнения может возникнуть пауза на этапе `Monitoring all tasks for 'Completed' state, timeout in 00:30:00...`, когда будут запускаться вычислительные узлы пула. 

```
Sample start: 11/19/2018 3:20:21 PM

Container [input] created.
Container [output] created.
Uploading file LowPriVMs-1.mp4 to container [input]...
Uploading file LowPriVMs-2.mp4 to container [input]...
Uploading file LowPriVMs-3.mp4 to container [input]...
Uploading file LowPriVMs-4.mp4 to container [input]...
Uploading file LowPriVMs-5.mp4 to container [input]...
Creating pool [WinFFmpegPool]...
Creating job [WinFFmpegJob]...
Adding 5 tasks to job [WinFFmpegJob]...
Monitoring all tasks for 'Completed' state, timeout in 00:30:00...
Success! All tasks completed successfully within the specified timeout period.
Deleting container [input]...

Sample end: 11/19/2018 3:29:36 PM
Elapsed time: 00:09:14.3418742
```

Перейдите в учетную запись пакетной службы на портале Azure, чтобы отследить пул, вычислительные узлы, задания и задачи. Например, чтобы увидеть тепловую карту вычислительных узлов в вашем пуле, выберите **Пулы** > *WinFFmpegPool*.

Во время выполнения задач тепловая карта выглядит следующим образом:

![Тепловая карта пула](./media/tutorial-parallel-dotnet/pool.png)

Обычное время выполнения — примерно **10 минут**, если для приложения задана конфигурация по умолчанию. Для создания пула потребуется больше всего времени.

[!INCLUDE [batch-common-tutorial-download](../../includes/batch-common-tutorial-download.md)]

## <a name="review-the-code"></a>Просмотр кода

В следующих разделах мы разобьем пример приложения на действия, выполняемые для обработки рабочей нагрузки в пакетной службе. Во время работы с оставшейся частью статьи сверяйтесь с файлом `Program.cs` в решении, так как в примере рассматриваются не все строки кода.

### <a name="authenticate-blob-and-batch-clients"></a>Проверка подлинности клиентов больших двоичных объектов и пакетной службы

Чтобы взаимодействовать со связанной учетной записью хранения, приложение использует клиентскую библиотеку службы хранилища для .NET. Оно создает ссылку на учетную запись с [CloudStorageAccount](/dotnet/api/microsoft.azure.cosmos.table.cloudstorageaccount), выполняя проверку подлинности с помощью общего ключа. Затем оно создает [CloudBlobClient](/dotnet/api/microsoft.azure.storage.blob.cloudblobclient).

```csharp
// Construct the Storage account connection string
string storageConnectionString = String.Format("DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}",
                                StorageAccountName, StorageAccountKey);

// Retrieve the storage account
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(storageConnectionString);

CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
```

Приложение создает объект [BatchClient](/dotnet/api/microsoft.azure.batch.batchclient) для создания пулов, заданий и задач в пакетной службе, а также для управления ими. В примере клиент пакетной службы использует проверку подлинности с общим ключом. Пакетная служба Azure также поддерживает аутентификацию отдельных пользователей или автоматических приложений с помощью [Azure Active Directory](batch-aad-auth.md).

```csharp
BatchSharedKeyCredentials sharedKeyCredentials = new BatchSharedKeyCredentials(BatchAccountUrl, BatchAccountName, BatchAccountKey);

using (BatchClient batchClient = BatchClient.Open(sharedKeyCredentials))
...
```

### <a name="upload-input-files"></a>Передача входных файлов

Приложение передает объект `blobClient` методу `CreateContainerIfNotExistAsync`, чтобы создать контейнер для хранения входных файлов (формат MP4) и контейнер для выходных данных задач.

```csharp
CreateContainerIfNotExistAsync(blobClient, inputContainerName);
CreateContainerIfNotExistAsync(blobClient, outputContainerName);
```

Затем файлы передаются в контейнер входных данных из локальной папки `InputFiles`. Файлы в хранилище определяются как объекты пакетной службы [ResourceFile](/dotnet/api/microsoft.azure.batch.resourcefile), которые она может впоследствии скачать на вычислительные узлы. 

При загрузке файлов задействованы два метода из `Program.cs`.

* `UploadFilesToContainerAsync`: возвращает коллекцию объектов ResourceFile и выполняет внутренний вызов `UploadResourceFileToContainerAsync`, чтобы отправить каждый файл, переданный в параметре `inputFilePaths`.
* `UploadResourceFileToContainerAsync`: загружает каждый файл в качестве большого двоичного объекта в контейнер входных данных. После отправки файла метод получает подписанный URL-адрес (SAS) для большого двоичного объекта и возвращает объект ResourceFile, который его представляет.

```csharp
string inputPath = Path.Combine(Environment.CurrentDirectory, "InputFiles");

List<string> inputFilePaths = new List<string>(Directory.GetFileSystemEntries(inputPath, "*.mp4",
    SearchOption.TopDirectoryOnly));

List<ResourceFile> inputFiles = await UploadFilesToContainerAsync(
  blobClient,
  inputContainerName,
  inputFilePaths);
```

Подробнее о загрузке файлов в качестве больших двоичных объектов в учетную запись хранения с .NET см. в статье об [отправке, скачивании и перечислении больших двоичных объектов с помощью .NET](../storage/blobs/storage-quickstart-blobs-dotnet.md).

### <a name="create-a-pool-of-compute-nodes"></a>Создание пула вычислительных узлов

Затем в учетной записи пакетной службы создается пул вычислительных узлов с помощью вызова `CreatePoolIfNotExistAsync`. С помощью метода [BatchClient.PoolOperations.CreatePool](/dotnet/api/microsoft.azure.batch.pooloperations.createpool) можно настроить количество узлов, размер виртуальной машины и конфигурацию пула. Объект [VirtualMachineConfiguration](/dotnet/api/microsoft.azure.batch.virtualmachineconfiguration) указывает [ImageReference](/dotnet/api/microsoft.azure.batch.imagereference) в образе Windows Server, опубликованном в Azure Marketplace. Пакетная служба Azure поддерживает широкий спектр образов виртуальной машины в Azure Marketplace, а также пользовательских образов виртуальной машины.

Количество узлов и размер виртуальной машины настраиваются с помощью определенных констант. Пакетная служба Azure поддерживает выделенные узлы и узлы [с низким приоритетом](batch-low-pri-vms.md). Вы можете использовать их в своих пулах. Выделенные узлы зарезервированы для пула. Низкоприоритетные узлы предлагаются по сниженной цене с учетом избыточных ресурсов виртуальной машины в Azure. Эти узлы становятся недоступны, если в Azure недостаточно ресурсов. Пример по умолчанию создает пул, содержащий только 5 низкоприоритетных узлов размером *Standard_A1_v2*.

>[!Note]
>Проверьте квоты узла. Инструкции по созданию запроса на квоту см. в описании [квот и ограничений пакетной службы](batch-quota-limit.md#increase-a-quota).

Приложение ffmpeg развертывается на вычислительных узлах, добавляя [ApplicationPackageReference](/dotnet/api/microsoft.azure.batch.applicationpackagereference) к конфигурации пула.

Метод [CommitAsync](/dotnet/api/microsoft.azure.batch.cloudpool.commitasync) отправляет пул в пакетную службу.

```csharp
ImageReference imageReference = new ImageReference(
    publisher: "MicrosoftWindowsServer",
    offer: "WindowsServer",
    sku: "2016-Datacenter-smalldisk",
    version: "latest");

VirtualMachineConfiguration virtualMachineConfiguration =
    new VirtualMachineConfiguration(
    imageReference: imageReference,
    nodeAgentSkuId: "batch.node.windows amd64");

pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    targetDedicatedComputeNodes: DedicatedNodeCount,
    targetLowPriorityComputeNodes: LowPriorityNodeCount,
    virtualMachineSize: PoolVMSize,
    virtualMachineConfiguration: virtualMachineConfiguration);

pool.ApplicationPackageReferences = new List<ApplicationPackageReference>
    {
    new ApplicationPackageReference {
    ApplicationId = appPackageId,
    Version = appPackageVersion}};

await pool.CommitAsync();  
```

### <a name="create-a-job"></a>Создание задания

Пакетное задание указывает пул для запуска задач и дополнительные параметры, такие как приоритет и расписание работы. Пример создает задание путем вызова `CreateJobAsync`. С помощью метода [BatchClient.JobOperations.CreateJob](/dotnet/api/microsoft.azure.batch.joboperations.createjob) можно создать задание в пуле.

Метод [CommitAsync](/dotnet/api/microsoft.azure.batch.cloudjob.commitasync) отправляет задание в пакетную службу. Изначально у задания нет задач.

```csharp
CloudJob job = batchClient.JobOperations.CreateJob();
job.Id = JobId;
job.PoolInformation = new PoolInformation { PoolId = PoolId };

await job.CommitAsync();
```

### <a name="create-tasks"></a>Создание задач

Пример создает задачи в задании путем вызова метода `AddTasksAsync`, который создает список объектов [CloudTask](/dotnet/api/microsoft.azure.batch.cloudtask). Каждый `CloudTask` запускает ffmpeg для обработки объекта входных данных `ResourceFile` с помощью свойства [CommandLine](/dotnet/api/microsoft.azure.batch.cloudtask.commandline). ffmpeg был ранее установлен на каждом узле при создании пула. В командной строке выполняется ffmpeg для преобразования каждого входного файла MP4 (видео) в файл MP3 (аудио).

В примере создается объект [OutputFile](/dotnet/api/microsoft.azure.batch.outputfile) для файла MP3 после запуска командной строки. Выходные файлы каждой задачи (в этом случае один) передаются в контейнер в связанной учетной записи с помощью свойства задачи [OutputFiles](/dotnet/api/microsoft.azure.batch.cloudtask.outputfiles). Ранее в примере кода был получен подписанный URL-адрес (`outputContainerSasUrl`), чтобы предоставить доступ на запись к выходному контейнеру. Обратите внимание на набор условий в объекте `outputFile`. Выходной файл из задачи передается в контейнер только после успешного завершения задачи (`OutputFileUploadCondition.TaskSuccess`). Ознакомьтесь с полным [примером кода](https://github.com/Azure-Samples/batch-dotnet-ffmpeg-tutorial) на сайте GitHub для получения дальнейших сведений о реализации.

Затем в примере к заданию добавляются задачи с помощью метода [AddTaskAsync](/dotnet/api/microsoft.azure.batch.joboperations.addtaskasync), который ставит их в очередь для запуска на вычислительных узлах.

```csharp
 // Create a collection to hold the tasks added to the job.
List<CloudTask> tasks = new List<CloudTask>();

for (int i = 0; i < inputFiles.Count; i++)
{
    string taskId = String.Format("Task{0}", i);

    // Define task command line to convert each input file.
    string appPath = String.Format("%AZ_BATCH_APP_PACKAGE_{0}#{1}%", appPackageId, appPackageVersion);
    string inputMediaFile = inputFiles[i].FilePath;
    string outputMediaFile = String.Format("{0}{1}",
        System.IO.Path.GetFileNameWithoutExtension(inputMediaFile),
        ".mp3");
    string taskCommandLine = String.Format("cmd /c {0}\\ffmpeg-4.3.1-2020-09-21-full_build\\bin\\ffmpeg.exe -i {1} {2}", appPath, inputMediaFile, outputMediaFile);

    // Create a cloud task (with the task ID and command line)
    CloudTask task = new CloudTask(taskId, taskCommandLine);
    task.ResourceFiles = new List<ResourceFile> { inputFiles[i] };

    // Task output file
    List<OutputFile> outputFileList = new List<OutputFile>();
    OutputFileBlobContainerDestination outputContainer = new OutputFileBlobContainerDestination(outputContainerSasUrl);
    OutputFile outputFile = new OutputFile(outputMediaFile,
       new OutputFileDestination(outputContainer),
       new OutputFileUploadOptions(OutputFileUploadCondition.TaskSuccess));
    outputFileList.Add(outputFile);
    task.OutputFiles = outputFileList;
    tasks.Add(task);
}

// Add tasks as a collection
await batchClient.JobOperations.AddTaskAsync(jobId, tasks);
return tasks
```

### <a name="monitor-tasks"></a>Мониторинг задач

Когда пакетная служба добавляет задачи к заданию, служба автоматически ставит в очередь и назначает их для выполнения на вычислительных узлах в связанном пуле. Пакетная служба обрабатывает постановку задач в очередь, их планирование, повтор и другие задачи администрирования с учетом указанных параметров.

Есть несколько подходов к отслеживанию выполнения задач. Этот пример определяет метод `MonitorTasks` для отчета о завершении и состоянии задачи (успешного выполнения или сбоя). Код `MonitorTasks` указывает [ODATADetailLevel](/dotnet/api/microsoft.azure.batch.odatadetaillevel), чтобы эффективно выбирать только минимум сведений о задачах. Затем он создает [TaskStateMonitor ](/dotnet/api/microsoft.azure.batch.taskstatemonitor), который предоставляет вспомогательные утилиты для мониторинга состояний задач. В `MonitorTasks` пример ждет, пока все задачи не перейдут в состояние `TaskState.Completed` в течение определенного времени. Затем он завершает работу задания и сообщает о любых выполненных задачах. Однако он может обнаружить сбой, например с ненулевым кодом выхода.

```csharp
TaskStateMonitor taskStateMonitor = batchClient.Utilities.CreateTaskStateMonitor();
try
{
    await taskStateMonitor.WhenAll(addedTasks, TaskState.Completed, timeout);
}
catch (TimeoutException)
{
    batchClient.JobOperations.TerminateJob(jobId);
    Console.WriteLine(incompleteMessage);
    return false;
}
batchClient.JobOperations.TerminateJob(jobId);
 Console.WriteLine(completeMessage);
...

```

## <a name="clean-up-resources"></a>Очистка ресурсов

После выполнения задач приложение автоматически удаляет созданный входной контейнер хранилища, а также предоставляет возможность удалить пул и задания пакетной службы. Классы [JobOperations](/dotnet/api/microsoft.azure.batch.batchclient.joboperations) и [PoolOperations BatchClient](/dotnet/api/microsoft.azure.batch.batchclient.pooloperations) предусматривают соответствующие методы удаления, которые вызываются, если подтвердить удаление. Вы не оплачиваете задания и задачи, но платите за используемые вычислительные узлы. Поэтому рекомендуется выделять пулы только при необходимости. При удалении пула удаляются все выходные данные задачи на узлах. Но выходные файлы сохраняются в учетной записи хранения.

Ставшие ненужными группу ресурсов, учетную запись пакетной службы и учетную запись хранения можно удалить. Для этого на портале Azure выберите группу ресурсов для учетной записи пакетной службы и щелкните **Удалить группу ресурсов**.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
> * Добавление пакета приложения в учетную запись пакетной службы.
> * Проверка подлинности с помощью учетных записей хранения и пакетной службы.
> * Передача входных файлов в хранилище.
> * Создание пула вычислительных узлов для запуска приложения.
> * Создание задания и задач для обработки входных файлов.
> * Мониторинг выполнения задач.
> * Извлечение выходных файлов

Дополнительные примеры использования программного интерфейса .NET для планирования и обработки рабочих нагрузок пакетной службы см. в примерах на сайте GitHub.

> [!div class="nextstepaction"]
> [Образцы пакетной службы C#](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp)
