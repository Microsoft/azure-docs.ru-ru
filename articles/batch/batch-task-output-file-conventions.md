---
title: Сохранение выходных данных в службе хранилища Azure с помощью библиотеки соглашений об именовании файлов .NET
description: Узнайте, как использовать библиотеку соглашений об именовании пакетных файлов Azure для .NET для сохранения выходных данных задач и заданий пакетной службы в службе хранилища Azure и просматривать сохраненные выходные данные на портале Azure.
ms.topic: how-to
ms.date: 11/14/2018
ms.custom: H1Hack27Feb2017, devx-track-csharp
ms.openlocfilehash: 1a45eed421dd8d734fcef0dd452df1d4a65fd053
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "88936968"
---
# <a name="persist-job-and-task-data-to-azure-storage-with-the-batch-file-conventions-library-for-net"></a>Сохранение данных заданий и задач в службе хранилища Azure с помощью библиотеки соглашений о пакетных файлах для .NET

[!INCLUDE [batch-task-output-include](../../includes/batch-task-output-include.md)]

Одним из способов сохранения данных задач является использование [библиотеки соглашений об именовании пакетных файлов Azure для .NET][nuget_package]. Библиотека файлов соглашений об именовании файлов упрощает процесс сохранения выходных данных задачи в службе хранилища Azure и извлечения из данной службы. Библиотеку соглашений об именовании файлов можно использовать в коде задачи и клиента &mdash; в коде задачи — для сохранения файлов, а в коде клиента — для получения списка файлов и их извлечения. Код задачи также может использовать библиотеку для получения выходных данных вышестоящих задач, например, как в сценарии [зависимостей задач](batch-task-dependencies.md).

Чтобы получить выходные файлы с помощью библиотеки соглашений об именовании файлов, можно найти файлы для данного задания или задачи, указав их по идентификатору и назначению. Имена файлов или их расположение знать не нужно. Например, можно использовать библиотеку соглашений об именовании файлов для вывода списка всех промежуточных файлов для данной задачи или получить файл предварительного просмотра для данного задания.

> [!TIP]
> Начиная с версии 2017-05-01, API пакетной службы поддерживает сохранение выходных данных в службе хранилища Azure для обычных задач и задач диспетчера заданий, которые выполняются в пулах, созданных с конфигурацией виртуальной машины. API пакетной службы предоставляет простой способ сохранения выходных данных в коде, который создает задачу и служит в качестве альтернативы библиотеке соглашений об именовании файлов. Клиентские приложения пакетной службы можно изменить для сохранения выходных данных без необходимости обновления приложения, в котором выполняется ваша задача. Дополнительные сведения можно найти в разделе [Сохранение данных задачи в службе хранилища Azure с помощью API пакетной службы](batch-task-output-files.md).

## <a name="when-do-i-use-the-file-conventions-library-to-persist-task-output"></a>Примеры использования библиотеки соглашений для именования файлов для сохранения выходных данных задачи

Пакетная служба Azure поддерживает несколько способов сохранения выходных данных задачи. Соглашения об именовании файлов наилучшим образом подходит для этих сценариев:

- Код для приложения, в котором выполняется ваша задача, можно легко изменить для сохранения файлов с помощью библиотеки соглашений об именовании файлов.
- Требуется обеспечить потоковую передачу данных в службу хранилища Azure, пока выполняется задача.
- Требуется сохранить данные из пулов, созданных с конфигурацией облачной службы или конфигурации виртуальной машины.
- Клиентское приложение или другие задачи в задании должны находить и скачивать выходные файлы задачи по идентификатору или назначению.
- Требуется просмотреть выходные данные задачи на портале Azure.

Если ваш случай отличается от перечисленных выше, возможно, придется использовать другой метод. Дополнительные сведения о других способах сохранения выходных данных задачи см. в разделе [Сохранение выходных данных заданий и задач в службе хранилища Azure](batch-task-output.md).

## <a name="what-is-the-batch-file-conventions-standard"></a>Что из себя представляет стандарт соглашений о пакетных файлах?

[Стандарт соглашений о пакетных файлах](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/batch/Microsoft.Azure.Batch.Conventions.Files) предоставляет схему именования путей к целевым контейнерам и большим двоичным объектам, в которые записываются выходные файлы. Файлы, сохраненные в службе хранилища Azure, которые соответствуют стандарту соглашений об именовании файлов, автоматически становятся доступны для просмотра на портале Azure. Портал учитывает соглашение об именовании и поэтому может отображать те файлы, которые ему соответствуют.

Библиотека соглашений об именовании файлов для .NET автоматически именует ваши контейнеры хранилища и выходных файлов задач в соответствии со стандартом соглашений об именовании файлов. Библиотека соглашений об именовании файлов также поддерживает методы запроса выходных файлов в службе хранилища Azure в соответствии с идентификатором задания, идентификатором задачи или назначением.

При работе на языке, отличном от .NET, можно самостоятельно реализовать в приложении стандарт соглашений об именовании файлов. Дополнительные сведения см. в разделе [Реализация стандарта соглашений по пакетным файлам](batch-task-output.md#implement-the-batch-file-conventions-standard).

## <a name="link-an-azure-storage-account-to-your-batch-account"></a>Связывание учетной записи хранения Azure со своей учетной записью пакетной службы

Для сохранения выходных данных в службе хранилища Azure с помощью библиотеки соглашений об именовании файлов необходимо связать учетную запись хранения Azure с учетной записью пакетной службы. Если вы этого еще не сделали, свяжите учетную запись хранения с учетной записью пакетной службы с помощью [портала Azure](https://portal.azure.com).

1. Войдите в свою учетную запись пакетной службы на портале Azure.
1. В разделе **Параметры** выберите **Учетная запись хранения**.
1. Если у вас еще нет учетной записи хранения, связанной с вашей учетной записью пакетной службы, нажмите кнопку **Учетная запись хранения (нет)** .
1. Выберите учетную запись хранения в списке для своей подписки. Для обеспечения наилучшей производительности используйте учетную запись хранения Azure в одном регионе с учетной записью пакетной службы, где выполняются задачи.

## <a name="persist-output-data"></a>Сохранение выходных данных

Для сохранения выходных данных заданий и задач с помощью библиотеки соглашений об именовании файлов создайте контейнер в службе хранилища Azure, а затем сохраните в нем выходные данные. Для отправки выходных данных задачи в контейнер используйте [клиентскую библиотеку службы хранилища Azure для .NET](https://www.nuget.org/packages/WindowsAzure.Storage) в коде задачи.

Дополнительные сведения о работе с контейнерами и большими двоичными объектами в службе хранилища Azure см. в разделе [Приступая к работе со службой хранилища больших двоичных объектов Azure с помощью .NET](../storage/blobs/storage-quickstart-blobs-dotnet.md).

> [!WARNING]
> Все выходные данные заданий и задач, сохраненные с помощью библиотеки соглашений об именовании файлов, хранятся в одном и том же контейнере. Если большое число задач попытаются одновременно сохранить файлы, могут быть применены ограничения для регулирования количества запросов к службе хранилища Azure. Дополнительные сведения об этих ограничениях см. в статье [Контрольный список для обеспечения масштабируемости и производительности для хранилища BLOB-объектов](../storage/blobs/storage-performance-checklist.md).

### <a name="create-storage-container"></a>Создание контейнера хранилища

Для сохранения выходных данных задачи в службе хранилища Azure сначала создайте контейнер путем вызова [CloudJob][net_cloudjob].[PrepareOutputStorageAsync][net_prepareoutputasync]. Этот метод расширения принимает объект [CloudStorageAccount][net_cloudstorageaccount] в качестве параметра. Он создает контейнер, именованный в соответствии со стандартом соглашений об именовании файлов, чтобы его содержимое можно было бы обнаружить с помощью портала Azure и методов извлечения данных, которые описаны ниже в этой статье.

Обычно этот код применяется для создания контейнера в клиентском приложении &mdash;, то есть приложении, которое создает пулы, задания и задачи.

```csharp
CloudJob job = batchClient.JobOperations.CreateJob(
    "myJob",
    new PoolInformation { PoolId = "myPool" });

// Create reference to the linked Azure Storage account
CloudStorageAccount linkedStorageAccount =
    new CloudStorageAccount(myCredentials, true);

// Create the blob storage container for the outputs
await job.PrepareOutputStorageAsync(linkedStorageAccount);
```

### <a name="store-task-outputs"></a>Хранение выходных данных задач

После подготовки контейнера в службе хранилища Azure задачи могут сохранять выходные данные в контейнер с помощью класса [TaskOutputStorage][net_taskoutputstorage], находящегося в библиотеке соглашений об именовании файлов.

В коде задачи сначала создайте объект [TaskOutputStorage][net_taskoutputstorage], затем по завершении задачи вызовите метод [TaskOutputStorage][net_taskoutputstorage].[SaveAsync][net_saveasync], чтобы сохранить выходные данные в службе хранилища Azure.

```csharp
CloudStorageAccount linkedStorageAccount = new CloudStorageAccount(myCredentials);
string jobId = Environment.GetEnvironmentVariable("AZ_BATCH_JOB_ID");
string taskId = Environment.GetEnvironmentVariable("AZ_BATCH_TASK_ID");

TaskOutputStorage taskOutputStorage = new TaskOutputStorage(
    linkedStorageAccount, jobId, taskId);

/* Code to process data and produce output file(s) */

await taskOutputStorage.SaveAsync(TaskOutputKind.TaskOutput, "frame_full_res.jpg");
await taskOutputStorage.SaveAsync(TaskOutputKind.TaskPreview, "frame_low_res.jpg");
```

Параметр `kind` метода [TaskOutputStorage](/dotnet/api/microsoft.azure.batch.conventions.files.taskoutputstorage).[ SaveAsync](/dotnet/api/microsoft.azure.batch.conventions.files.taskoutputstorage.saveasync#overloads) классифицирует сохраненные файлы. Существует четыре стандартных типа [TaskOutputKind][net_taskoutputkind]: `TaskOutput`, `TaskPreview`, `TaskLog` и `TaskIntermediate.`. Также можно определить пользовательские категории выходных данных.

Эти типы позволяют указать тип выходных данных, отображаемых в списке, когда вы запрашиваете у пакетной службы сохраненные выходные данные какой-либо определенной задачи. Другими словами, при выводе списка выходных данных их можно отфильтровать по одному из типов выходных данных. Например, "Выдайте мне версию *preview* выходных данных для задачи *109*". Дополнительные сведения о выводе списка и извлечении выходных данных приведены далее, в разделе "Извлечение выходных данных" этой статьи.

> [!TIP]
> Тип выходных данных также определяет, где на портале Azure будет отображаться определенный файл. Файлы, классифицированные по типу *TaskOutput* отображаются в разделе **Task output files** (Выходные данные задач), а файлы *TaskLog* — в разделе **Журналы задач**.

### <a name="store-job-outputs"></a>Хранения выходных данных заданий

Помимо хранения выходных данных задач, можно сохранять выходные данные, связанные со всем заданием. Например, для задачи слияния в задании отрисовки фильма можно сохранить полностью отрисованный фильм в качестве выходных данных задания. По завершении задания клиентское приложение может получить список и извлечь выходные данные этого задания, не отправляя запросы к отдельным задачам.

Для сохранения выходных данных задания используйте вызов метода [JobOutputStorage][net_joboutputstorage].[SaveAsync][net_joboutputstorage_saveasync] и укажите [JobOutputKind][net_joboutputkind] и имя файла:

```csharp
CloudJob job = new JobOutputStorage(acct, jobId);
JobOutputStorage jobOutputStorage = job.OutputStorage(linkedStorageAccount);

await jobOutputStorage.SaveAsync(JobOutputKind.JobOutput, "mymovie.mp4");
await jobOutputStorage.SaveAsync(JobOutputKind.JobPreview, "mymovie_preview.mp4");
```

В качестве значения **TaskOutputKind** для выходных данных задачи используйте тип [JobOutputKind][net_joboutputkind], чтобы классифицировать сохраняемые файлы задания. Позже это позволит запросить (получить список) определенный тип выходных данных. Тип **JobOutputKind** включает в себя категории output и preview, а также позволяет создавать пользовательские категории.

### <a name="store-task-logs"></a>Хранение журналов задач

Помимо сохранения файла в долговременном хранилище при завершении задачи или задания, может возникнуть потребность сохранять файлы, которые обновляются во время выполнения задачи &mdash;, например файлы журнала или `stdout.txt` и `stderr.txt`. Для этой цели библиотека Azure Batch File Conventions предоставляет метод [TaskOutputStorage][net_taskoutputstorage].[SaveTrackedAsync][net_savetrackedasync]. С помощью метода [SaveTrackedAsync][net_savetrackedasync] можно отслеживать обновления файла на узле (с указанным интервалом) и сохранять эти обновления в службе хранилища Azure.

В приведенном ниже фрагменте кода [SaveTrackedAsync][net_savetrackedasync] используется для обновления `stdout.txt` в службе хранилища Azure во время выполнения задачи через каждые 15 секунд.

```csharp
TimeSpan stdoutFlushDelay = TimeSpan.FromSeconds(3);
string logFilePath = Path.Combine(
    Environment.GetEnvironmentVariable("AZ_BATCH_TASK_DIR"), "stdout.txt");

// The primary task logic is wrapped in a using statement that sends updates to
// the stdout.txt blob in Storage every 15 seconds while the task code runs.
using (ITrackedSaveOperation stdout =
        await taskStorage.SaveTrackedAsync(
        TaskOutputKind.TaskLog,
        logFilePath,
        "stdout.txt",
        TimeSpan.FromSeconds(15)))
{
    /* Code to process data and produce output file(s) */

    // We are tracking the disk file to save our standard output, but the
    // node agent may take up to 3 seconds to flush the stdout stream to
    // disk. So give the file a moment to catch up.
     await Task.Delay(stdoutFlushDelay);
}
```

Раздел с комментариями `Code to process data and produce output file(s)` — это просто заполнитель для обычного кода задачи. Например, вам может понадобиться код, который скачивает данные из службы хранилища Azure и выполняет с ними операцию преобразования или вычисления. В данном фрагменте кода важно обратить внимание на то, как такой код был помещен в блок `using` для периодического обновления файла с помощью метода [SaveTrackedAsync][net_savetrackedasync].

Агент узла — это программа, которая выполняется на каждом узле в пуле и предоставляет интерфейс контроля и управления между узлом и пакетной службой. Чтобы агент узла имел время для записи содержимого стандартного потока вывода в файл stdout.txt на узле, в конце этого блока `using` требуется вызов `Task.Delay`. Без этой задержки вывод данных завершится на несколько секунд быстрее и, как результат, будет неполным. Задержка может не потребоваться для всех файлов.

> [!NOTE]
> При включении отслеживания файла с помощью метода **SaveTrackedAsync** в службе хранилища Azure будут сохраняться только данные, *добавляемые* в отслеживаемый файл. Этот метод следует использовать только для отслеживания не участвующих в ротации файлов журнала или других файлов, в конец которых с помощью операций добавления записываются данные.

## <a name="retrieve-output-data"></a>Извлечение выходных данных

Извлечь сохраненные выходные данные с помощью библиотеки Azure Batch File Conventions можно в контексте задачи и задания. Можно запросить выходные данные определенной задачи или задания без необходимости знать ни путь к ним в службе хранилища Azure, ни даже имя файла. Вместо этого выходные файлы можно запрашивать по идентификатору задачи или задания.

Приведенный ниже фрагмент кода перебирает задачи в задании, выводит некоторые сведения о выходных файлах для задачи, а затем скачивает их из службы хранилища.

```csharp
foreach (CloudTask task in myJob.ListTasks())
{
    foreach (OutputFileReference output in
        task.OutputStorage(storageAccount).ListOutputs(
            TaskOutputKind.TaskOutput))
    {
        Console.WriteLine($"output file: {output.FilePath}");

        output.DownloadToFileAsync(
            $"{jobId}-{output.FilePath}",
            System.IO.FileMode.Create).Wait();
    }
}
```

## <a name="view-output-files-in-the-azure-portal"></a>Просмотр выходных файлов на портале Azure

Портал Azure будет отображать выходные данные и журналы задачи, сохраненные в связанной учетной записи службы хранилища Azure с использованием [стандарта соглашений об именовании пакетных файлов](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/batch/Microsoft.Azure.Batch.Conventions.Files). Вы можете самостоятельно реализовать эти соглашения на выбранном языке или использовать библиотеку соглашений об именовании файлов в приложениях .NET.

Чтобы выходные файлы отображались на портале, необходимо выполнить следующие требования:

1. Свяжите учетную запись хранения Azure со своей учетной записью пакетной службы.
1. При сохранении выходных данных нужно соблюдать предварительно определенные соглашения об именовании контейнеров хранилища и файлов. Определение этих соглашений можно найти в [файле сведений][github_file_conventions_readme] о библиотеке соглашений об именовании файлов. Если для сохранения выходных данных используется библиотека [соглашений об именовании пакетных файлов Azure][nuget_package], файлы сохраняются в соответствии со стандартом соглашений об именовании файлов.

Чтобы просмотреть выходные файлы и журналы задачи на портале Azure, перейдите к задаче, выходные данные которой вас интересуют, затем щелкните **Сохраненные выходные файлы** или **Сохраненные журналы**. На приведенном рисунке показан раздел **Saved output files** (Сохраненные выходные файлы) для задачи с идентификатором 007.

![Колонка выходных данных задачи на портале Azure][2]

## <a name="code-sample"></a>Пример кода

Пример проекта [PersistOutputs][github_persistoutputs] — это один из [примеров кода пакетной службы Azure][github_samples] на портале GitHub. Это решение Visual Studio демонстрирует использование библиотеки Azure Batch File Conventions для сохранения выходных данных задачи в долговременном хранилище. Чтобы запустить пример приложения, выполните следующее.

1. Откройте проект в **Visual Studio 2019**.
2. Добавьте свои **данные** учетной записи пакетной службы и учетной записи хранения в **AccountSettings.settings** в проекте Microsoft.Azure.Batch.Samples.Common.
3. **сборку** решения (но не запускайте его). Восстановите необходимые пакеты NuGet в случае появления соответствующего запроса.
4. С помощью портала Azure передайте [пакет приложения](batch-application-packages.md) для **PersistOutputsTask**. Добавьте в ZIP-файл пакета `PersistOutputsTask.exe` и его зависимые сборки , задайте для приложения идентификатор PersistOutputsTask, а для версии пакета приложения — значение "1.0".
5. **Запустите** (выполните) проект **PersistOutputs**.
6. Когда появится запрос на выбор технологии постоянного хранения для выполнения образца, введите **1** для запуска образца с помощью библиотеки соглашений об именовании файлов для сохранения выходных данных задачи. 

## <a name="next-steps"></a>Дальнейшие действия

### <a name="get-the-batch-file-conventions-library-for-net"></a>Получение библиотеки соглашений об именовании пакетных файлов для .NET

Библиотека соглашений об именовании пакетных файлов для .NET доступна в [NuGet][nuget_package]. Библиотека расширяет классы [CloudJob][net_cloudjob] и [CloudTask][net_cloudtask] новыми методами. Сведения о библиотеке соглашений о наименовании файлов см. также в [справочной документации](/dotnet/api/microsoft.azure.batch.conventions.files).

[Исходный код][github_file_conventions] библиотеки соглашений об именовании файлов доступен на сайте GitHub в репозитории пакетов SDK Microsoft Azure для .NET. 

### <a name="explore-other-approaches-for-persisting-output-data"></a>Другие методы сохранения выходных данных

- Обзор методов сохранения данных задач и заданий см. в разделе [Сохранение выходных данных заданий и задач в службе хранилища Azure](batch-task-output.md).
- Сведения о том, как использовать API пакетной службы для сохранения выходных данных, см. в разделе [Сохранение данных задачи в службе хранилища Azure с помощью API пакетной службы](batch-task-output-files.md).

[forum_post]: https://social.msdn.microsoft.com/Forums/en-US/87b19671-1bdf-427a-972c-2af7e5ba82d9/installing-applications-and-staging-data-on-batch-compute-nodes?forum=azurebatch
[github_file_conventions]: https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/batch/Microsoft.Azure.Batch.Conventions.Files
[github_file_conventions_readme]: https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/batch/Microsoft.Azure.Batch.Conventions.Files/README.md
[github_persistoutputs]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/PersistOutputs
[github_samples]: https://github.com/Azure/azure-batch-samples
[net_batchclient]: /dotnet/api/microsoft.azure.batch.batchclient
[net_cloudjob]: /dotnet/api/microsoft.azure.batch.cloudjob
[net_cloudstorageaccount]: /java/api/com.microsoft.azure.storage.cloudstorageaccount
[net_cloudtask]: /dotnet/api/microsoft.azure.batch.cloudtask
[net_fileconventions_readme]: https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/batch/Microsoft.Azure.Batch.Conventions.Files/README.md
[net_joboutputkind]: /dotnet/api/microsoft.azure.batch.conventions.files.joboutputkind
[net_joboutputstorage]: /dotnet/api/microsoft.azure.batch.conventions.files.joboutputstorage
[net_joboutputstorage_saveasync]: /dotnet/api/microsoft.azure.batch.conventions.files.joboutputstorage.saveasync
[net_msdn]: /dotnet/api/microsoft.azure.batch
[net_prepareoutputasync]: /dotnet/api/microsoft.azure.batch.conventions.files.cloudjobextensions.prepareoutputstorageasync
[net_saveasync]: /dotnet/api/microsoft.azure.batch.conventions.files.taskoutputstorage.saveasync
[net_savetrackedasync]: /dotnet/api/microsoft.azure.batch.conventions.files.taskoutputstorage.savetrackedasync
[net_taskoutputkind]: /dotnet/api/microsoft.azure.batch.conventions.files.taskoutputkind
[net_taskoutputstorage]: /dotnet/api/microsoft.azure.batch.conventions.files.taskoutputstorage
[nuget_manager]: https://docs.nuget.org/consume/installing-nuget
[nuget_package]: https://www.nuget.org/packages/Microsoft.Azure.Batch.Conventions.Files
[portal]: https://portal.azure.com
[storage_explorer]: https://storageexplorer.com/

[1]: ./media/batch-task-output/task-output-01.png "Селекторы сохраненных выходных файлов и журналов на портале"
[2]: ./media/batch-task-output/task-output-02.png "Колонка выходных данных задачи на портале Azure"
