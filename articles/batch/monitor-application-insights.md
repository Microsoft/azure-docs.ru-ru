---
title: Мониторинг пакетной службы с помощью Azure Application Insights
description: Узнайте, как инструментировать приложение .NET пакетной службы Azure с помощью библиотеки Azure Application Insights.
ms.topic: how-to
ms.custom: devx-track-csharp
ms.date: 04/05/2018
ms.openlocfilehash: 9decb99c3de798df43dedc2441208066d18e3a13
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104605789"
---
# <a name="monitor-and-debug-an-azure-batch-net-application-with-application-insights"></a>Мониторинг и отладка приложения .NET пакетной службы Azure с помощью Application Insights

[Application Insights](../azure-monitor/app/app-insights-overview.md) предоставляет разработчикам элегантный и эффективный способ мониторинга и отладки приложений, развернутых в службах Azure. Application Insights можно использовать для мониторинга счетчиков производительности и исключений, а также инструментирования кода с помощью настраиваемых метрик и трассировки. Интеграция Application Insights с приложением пакетной службы позволяет получать полное представление о поведении и исследовать проблемы практически в режиме реального времени.

В этой статье показано, как добавить и настроить библиотеку Application Insights в вашем решении .NET пакетной службы Azure и инструментировать код приложения. Здесь также описаны способы мониторинга приложения на портале Azure и создания настраиваемых панелей мониторинга. Для поддержки Application Insights на других языках ознакомьтесь с [документацией по языкам, платформам и интеграции](../azure-monitor/app/platforms.md).

Пример решения C# с кодом, который служит дополнением к этой статье, доступен в [GitHub](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/ApplicationInsights). В этом примере код инструментирования Application Insights добавляется в пример [TopNWords](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/TopNWords). Если вы не знакомы с этим примером, сначала создайте и запустите TopNWords. Это поможет вам получить представление о базовом рабочем процессе обработки набора входных больших двоичных объектов в параллельном режиме на нескольких вычислительных узлах в пакетной службе. 

> [!TIP]
> Также можно настроить в решении пакетной службы отображение данных из Application Insights, таких как счетчики производительности виртуальных машин в Batch Explorer. [Batch Explorer](https://github.com/Azure/BatchExplorer) — это бесплатный автономный клиентский инструмент с множеством функций для создания, отладки и мониторинга приложений пакетной службы Azure. Скачайте [пакет установки](https://azure.github.io/BatchExplorer/) для Mac, Linux или Windows. См. [репозиторий batch-insights](https://github.com/Azure/batch-insights) с описанием быстрых действий, позволяющих включить данные Application Insights в Batch Explorer. 
>

## <a name="prerequisites"></a>Предварительные требования
* [Visual Studio 2017 или более поздней версии](https://www.visualstudio.com/vs)

* [Учетная запись пакетной службы и связанная учетная запись хранения](batch-account-create-portal.md).

* [Ресурс Application Insights](../azure-monitor/app/create-new-resource.md ).
  
   * Создайте *ресурс* Application Insights на портале Azure. Выберите *Общие* **Тип приложения**.

   * Скопируйте [ключ инструментирования](../azure-monitor/app/create-new-resource.md#copy-the-instrumentation-key) на портале. Он понадобится вам позже.
  
  > [!NOTE]
  > За данные, хранимые в Application Insights, может [взиматься плата](https://azure.microsoft.com/pricing/details/application-insights/). В том числе и данные диагностики и мониторинга, о которых идет речь в этой статье.
  > 

## <a name="add-application-insights-to-your-project"></a>Добавление Application Insights в ваш проект

Для вашего проекта необходим пакет NuGet **Microsoft.ApplicationInsights.WindowsServer** и его зависимости. Добавьте или восстановите их в проект приложения. Чтобы установить пакет, используйте команду `Install-Package` или диспетчер пакетов NuGet.

```powershell
Install-Package Microsoft.ApplicationInsights.WindowsServer
```
Создайте ссылку на Application Insights из приложения .NET с помощью пространства имен **Microsoft.ApplicationInsights**.

## <a name="instrument-your-code"></a>Инструментирование кода

Чтобы инструментировать код, ваше решение должно создать [TelemetryClient](/dotnet/api/microsoft.applicationinsights.telemetryclient) Application Insights. В примере TelemetryClient загружает свою конфигурацию из файла [ApplicationInsights.config](../azure-monitor/app/configuration-with-applicationinsights-config.md). В файл ApplicationInsights.config в следующих проектах добавьте свой ключ инструментирования Application Insights: Microsoft.Azure.Batch.Samples.TelemetryStartTask и TopNWordsSample.

```xml
<InstrumentationKey>YOUR-IKEY-GOES-HERE</InstrumentationKey>
```
Кроме того, добавьте ключ инструментирования в файл TopNWords.cs.

В примере в TopNWords.cs используются следующие [вызовы инструментария](../azure-monitor/app/api-custom-events-metrics.md) из API Application Insights:
* `TrackMetric()` — отслеживает среднюю длительность загрузки необходимого текстового файла на вычислительном узле.
* `TrackTrace()` — добавляет вызовы отладки в код.
* `TrackEvent()` — отслеживает интересующие события, сведения о которых необходимо записать.

В этом примере намеренно не указана обработка исключений. Вместо этого Application Insights автоматически отправляет отчеты о необработанных исключениях, что значительно улучшает процесс отладки. 

В следующем фрагменте кода показано, как использовать эти методы.

```csharp
public void CountWords(string blobName, int numTopN, string storageAccountName, string storageAccountKey)
{
    // simulate exception for some set of tasks
    Random rand = new Random();
    if (rand.Next(0, 10) % 10 == 0)
    {
        blobName += ".badUrl";
    }

    // log the url we are downloading the file from
    insightsClient.TrackTrace(new TraceTelemetry(string.Format("Task {0}: Download file from: {1}", this.taskId, blobName), SeverityLevel.Verbose));

    // open the cloud blob that contains the book
    var storageCred = new StorageCredentials(storageAccountName, storageAccountKey);
    CloudBlockBlob blob = new CloudBlockBlob(new Uri(blobName), storageCred);
    using (Stream memoryStream = new MemoryStream())
    {
        // calculate blob download time
        DateTime start = DateTime.Now;
        blob.DownloadToStream(memoryStream);
        TimeSpan downloadTime = DateTime.Now.Subtract(start);

        // track how long the blob takes to download on this node
        // this will help debug timing issues or identify poorly performing nodes
        insightsClient.TrackMetric("Blob download in seconds", downloadTime.TotalSeconds, this.CommonProperties);

        memoryStream.Position = 0; //Reset the stream
        var sr = new StreamReader(memoryStream);
        var myStr = sr.ReadToEnd();
        string[] words = myStr.Split(' ');

        // log how many words were found in the text file
        insightsClient.TrackTrace(new TraceTelemetry(string.Format("Task {0}: Found {1} words", this.taskId, words.Length), SeverityLevel.Verbose));
        var topNWords =
            words.
                Where(word => word.Length > 0).
                GroupBy(word => word, (key, group) => new KeyValuePair<String, long>(key, group.LongCount())).
                OrderByDescending(x => x.Value).
                Take(numTopN).
                ToList();
        foreach (var pair in topNWords)
        {
            Console.WriteLine("{0} {1}", pair.Key, pair.Value);
        }

        // emit an event to track the completion of the task
        insightsClient.TrackEvent("Done counting words");
    }
}
```

### <a name="azure-batch-telemetry-initializer-helper"></a>Вспомогательное приложение инициализатора данных телеметрии пакетной службы Azure
При создании отчетов о данных телеметрии для данного сервера и экземпляра Application Insights использует роль виртуальной машины Azure и имя виртуальной машины в качестве значений по умолчанию. В контексте пакетной службы Azure в примере показано, как вместо этого использовать имя пула и вычислительного узла. Используйте [инициализатор данных телеметрии](../azure-monitor/app/api-filtering-sampling.md#add-properties), чтобы переопределить значения по умолчанию. 

```csharp
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.Extensibility;
using System;
using System.Threading;

namespace Microsoft.Azure.Batch.Samples.TelemetryInitializer
{
    public class AzureBatchNodeTelemetryInitializer : ITelemetryInitializer
    {
        // Azure Batch environment variables
        private const string PoolIdEnvironmentVariable = "AZ_BATCH_POOL_ID";
        private const string NodeIdEnvironmentVariable = "AZ_BATCH_NODE_ID";

        private string roleInstanceName;
        private string roleName;

        public void Initialize(ITelemetry telemetry)
        {
            if (string.IsNullOrEmpty(telemetry.Context.Cloud.RoleName))
            {
                // override the role name with the Azure Batch Pool name
                string name = LazyInitializer.EnsureInitialized(ref this.roleName, this.GetPoolName);
                telemetry.Context.Cloud.RoleName = name;
            }

            if (string.IsNullOrEmpty(telemetry.Context.Cloud.RoleInstance))
            {
                // override the role instance with the Azure Batch Compute Node name
                string name = LazyInitializer.EnsureInitialized(ref this.roleInstanceName, this.GetNodeName);
                telemetry.Context.Cloud.RoleInstance = name;
            }
        }

        private string GetPoolName()
        {
            return Environment.GetEnvironmentVariable(PoolIdEnvironmentVariable) ?? string.Empty;
        }

        private string GetNodeName()
        {
            return Environment.GetEnvironmentVariable(NodeIdEnvironmentVariable) ?? string.Empty;
        }
    }
}
```

Для включения инициализатора данных телеметрии в файле ApplicationInsights.config в проекте TopNWordsSample содержится следующий код:

```xml
<TelemetryInitializers>
    <Add Type="Microsoft.Azure.Batch.Samples.TelemetryInitializer.AzureBatchNodeTelemetryInitializer, Microsoft.Azure.Batch.Samples.TelemetryInitializer"/>
</TelemetryInitializers>
``` 

## <a name="update-the-job-and-tasks-to-include-application-insights-binaries"></a>Добавление двоичных файлов Application Insights в задание и задачи

Для корректной работы Application Insights на вычислительных узлах двоичные файлы должны быть правильно расположены. Добавьте необходимые двоичные файлы в коллекцию файлов ресурсов вашей задачи, чтобы они загрузились в момент выполнения вашей задачи. Следующие фрагменты кода аналогичны коду в Job.cs.

Сначала создайте статический список файлов Application Insights для отправки.

```csharp
private static readonly List<string> AIFilesToUpload = new List<string>()
{
    // Application Insights config and assemblies
    "ApplicationInsights.config",
    "Microsoft.ApplicationInsights.dll",
    "Microsoft.AI.Agent.Intercept.dll",
    "Microsoft.AI.DependencyCollector.dll",
    "Microsoft.AI.PerfCounterCollector.dll",
    "Microsoft.AI.ServerTelemetryChannel.dll",
    "Microsoft.AI.WindowsServer.dll",
    
    // custom telemetry initializer assemblies
    "Microsoft.Azure.Batch.Samples.TelemetryInitializer.dll",
 };
...
```

Затем создайте промежуточные файлы, используемые задачей.
```csharp
...
// create file staging objects that represent the executable and its dependent assembly to run as the task.
// These files are copied to every node before the corresponding task is scheduled to run on that node.
FileToStage topNWordExe = new FileToStage(TopNWordsExeName, stagingStorageAccount);
FileToStage storageDll = new FileToStage(StorageClientDllName, stagingStorageAccount);

// Upload Application Insights assemblies
List<FileToStage> aiStagedFiles = new List<FileToStage>();
foreach (string aiFile in AIFilesToUpload)
{
    aiStagedFiles.Add(new FileToStage(aiFile, stagingStorageAccount));
}
...
```

Метод `FileToStage` — это вспомогательная функция в примере кода, которая позволяет легко отправить файл с локального диска в большой двоичный объект службы хранилища Azure. Каждый файл позже загружается на вычислительный узел и на него ссылается задача.

Наконец, добавьте задачи в задание и необходимые двоичные файлы Application Insights.
```csharp
...
// initialize a collection to hold the tasks that will be submitted in their entirety
List<CloudTask> tasksToRun = new List<CloudTask>(topNWordsConfiguration.NumberOfTasks);
for (int i = 1; i <= topNWordsConfiguration.NumberOfTasks; i++)
{
    CloudTask task = new CloudTask("task_no_" + i, String.Format("{0} --Task {1} {2} {3} {4}",
        TopNWordsExeName,
        string.Format("https://{0}.blob.core.windows.net/{1}",
            accountSettings.StorageAccountName,
            documents[i]),
        topNWordsConfiguration.TopWordCount,
        accountSettings.StorageAccountName,
        accountSettings.StorageAccountKey));

    //This is the list of files to stage to a container -- for each job, one container is created and 
    //files all resolve to Azure Blobs by their name (so two tasks with the same named file will create just 1 blob in
    //the container).
    task.FilesToStage = new List<IFileStagingProvider>
                        {
                            // required application binaries
                            topNWordExe,
                            storageDll,
                        };
    foreach (FileToStage stagedFile in aiStagedFiles)
   {
        task.FilesToStage.Add(stagedFile);
   }    
    task.RunElevated = false;
    tasksToRun.Add(task);
}
```

## <a name="view-data-in-the-azure-portal"></a>Просмотр данных на портале Azure

Теперь после настройки задания и задач для использования Application Insights запустите пример задания в пуле. Перейдите на портал Azure и откройте подготовленный ресурс Application Insights. После подготовки пула вы должны увидеть поток данных и войти в систему. В оставшейся части этой статьи затрагиваются только некоторые возможности Application Insights, но вы можете просмотреть полный набор функций.

### <a name="view-live-stream-data"></a>Просмотр данных Live Stream

Чтобы просмотреть журналы трассировки в ресурсе Application Insights, щелкните **Live Stream**. На следующем снимке экрана показано, как просматривать динамические данные, поступающие из вычислительных узлов в пуле, например данные об использовании ЦП одного вычислительного узла.

![Данные вычислительного узла Live Stream](./media/monitor-application-insights/applicationinsightslivestream.png)

### <a name="view-trace-logs"></a>Просмотр журналов трассировки

Чтобы просмотреть журналы трассировки в ресурсе Application Insights, щелкните **Поиск**. В этом представлении отображается список диагностических данных, полученных Application Insights, включая трассировки, события и исключения. 

На следующем снимке экрана показано, как одиночная трассировка для задачи записывается в журнал, а затем запрашивается для целей отладки.

![Изображение журналов трассировки](./media/monitor-application-insights/tracelogsfortask.png)

### <a name="view-unhandled-exceptions"></a>Просмотр необработанных исключений

На следующих снимках экрана показано, каким образом Application Insights записывает в журнал исключения, возникшие в приложении. В этом случае в течение нескольких секунд после создания исключения в приложении можно перейти к определенному исключению и диагностировать проблему.

![Необработанные исключения](./media/monitor-application-insights/exception.png)

### <a name="measure-blob-download-time"></a>Измерение времени загрузки больших двоичных объектов

Настраиваемые метрики также являются полезным средством на портале. Например, можно отобразить среднее время, затраченное на каждом вычислительном узле для загрузки необходимого текстового файла, который он обрабатывал.

Создание примера диаграммы
1. В ресурсе Application Insights щелкните **Обозреватель метрик** > **Добавить диаграмму**.
2. Нажмите кнопку **Изменить** на добавленной диаграмме.
2. Обновите сведения диаграммы следующим образом:
   * Для параметра **Тип диаграммы** задайте значение **Сетка**.
   * Для параметра **Агрегирование** задайте значение **Среднее**.
   * Для параметра **Группировать по** задайте значение **NodeId**.
   * В разделе **Метрики** выберите **Настраиваемая** > **Blob download in seconds** (Загрузка большого двоичного объекта в секундах).
   * Выберите нужный цвет в поле **Палитра цветов**. 

![Время загрузки больших двоичных объектов на одном узле](./media/monitor-application-insights/blobdownloadtime.png)


## <a name="monitor-compute-nodes-continuously"></a>Постоянный мониторинг вычислительных узлов

Вы могли заметить, что все метрики, включая счетчики производительности, заносятся в журнал, только если выполняются задачи. Это полезно, так как ограничивается объем данных, который регистрирует Application Insights. Тем не менее в некоторых случаях может потребоваться отслеживать вычислительные узлы постоянно. Например, они могут выполнять фоновую работу, которая не запланирована в пакетной службе. В этом случае настройте выполнение процесса мониторинга в течение жизненного цикла вычислительного узла. 

Для этого можно создать процесс, который загружает библиотеку Application Insights и выполняется в фоновом режиме. В примере начальная задача загружает двоичные файлы на компьютер и этот процесс выполняется в течение неограниченного времени. Настройте файл конфигурации Application Insights для этого процесса, чтобы выдать дополнительные данные, которые вас интересуют, например счетчики производительности.

```csharp
...
 // Batch start task telemetry runner
private const string BatchStartTaskFolderName = "StartTask";
private const string BatchStartTaskTelemetryRunnerName = "Microsoft.Azure.Batch.Samples.TelemetryStartTask.exe";
private const string BatchStartTaskTelemetryRunnerAIConfig = "ApplicationInsights.config";
...
CloudPool pool = client.PoolOperations.CreatePool(
    topNWordsConfiguration.PoolId,
    targetDedicated: topNWordsConfiguration.PoolNodeCount,
    virtualMachineSize: "standard_d1_v2",
    cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));
...

// Create a start task which will run a dummy exe in background that simply emits performance
// counter data as defined in the relevant ApplicationInsights.config.
// Note that the waitForSuccess on the start task was not set so the Compute Node will be
// available immediately after this command is run.
pool.StartTask = new StartTask()
{
    CommandLine = string.Format("cmd /c {0}", BatchStartTaskTelemetryRunnerName),
    ResourceFiles = resourceFiles
};
...
```

> [!TIP]
> Чтобы улучшить управляемость вашего решения, можно объединить сборку в [пакете приложения](./batch-application-packages.md). Далее, чтобы автоматически развернуть пакет приложения в пулы, добавьте ссылку на пакет приложения в конфигурацию пула.
>

## <a name="throttle-and-sample-data"></a>Регулирование и образцы данных 

Из-за особенностей крупномасштабных приложений пакетной службы Azure, работающих в рабочей среде, для управления затратами может потребоваться ограничить объем данных, собираемых Application Insights. Сведения о механизмах, с помощью которых этого можно добиться, см. в статье [Выборка в Application Insights](../azure-monitor/app/sampling.md).


## <a name="next-steps"></a>Дальнейшие действия
* Дополнительные сведения об [Application Insights](../azure-monitor/app/app-insights-overview.md).

* Для поддержки Application Insights на других языках ознакомьтесь с [документацией по языкам, платформам и интеграции](../azure-monitor/app/platforms.md).


