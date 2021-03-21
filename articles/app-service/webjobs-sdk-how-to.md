---
title: Использование пакета SDK для веб-заданий
description: Ознакомьтесь со сведениями о написании кода для пакета SDK WebJobs. Создавайте задания фоновой обработки на основе событий, с помощью которых можно получить доступ к данным в Azure и сторонних службах.
author: ggailey777
ms.devlang: dotnet
ms.custom: devx-track-csharp
ms.topic: article
ms.date: 02/18/2019
ms.author: glenga
ms.openlocfilehash: 063924dccb7d7b95b962b24ecc1af1870a855194
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102617137"
---
# <a name="how-to-use-the-azure-webjobs-sdk-for-event-driven-background-processing"></a>Использование пакета SDK WebJobs Azure для фоновой обработки на основе событий

В этой статье приводятся рекомендации по работе с пакетом SDK для веб-заданий Azure. Инструкции по началу работы см в разделе [Использование пакета SDK для веб-заданий Azure для фоновой обработки на основе событий](webjobs-sdk-get-started.md). 

## <a name="webjobs-sdk-versions"></a>Версии пакета SDK для веб-заданий

Это ключевые различия между версией 3.*x* и версией 2.*x* пакета SDK для веб-заданий.

* В версии 3.*x* добавлена поддержка .NET Core.
* В версии 3.*x* необходимо явно устанавливать расширения привязки хранилища, которые являются обязательными для пакета SDK веб-заданий. В версии 2.*x* привязки хранилища включены в пакет SDK.
* В Visual Studio для проектов .NET Core (3.*x*) используется инструментарий, отличный от инструментов для проектов .NET Framework (2.*x*). Дополнительные сведения см. в разделе [Разработка и развертывание веб-заданий в Службе приложений Azure с помощью Visual Studio](webjobs-dotnet-deploy-vs.md).

По возможности приводятся примеры как для версии 3.*x*, так и для версии 2.*x*.

> [!NOTE]
> Решение [Функции Azure](../azure-functions/functions-overview.md) разработано на основе пакета SDK для веб-заданий, и эта статья содержит ссылки на документацию по решению "Функции Azure", посвященную отдельным темам. Обратите внимание на следующие различия между пакетом SDK для решения "Функции Azure" и для веб-заданий.
> * Решение "Функции Azure" 2.*x* соответствует пакету SDK для веб-заданий 3.*x*, а "Функции Azure" 1.*x* соответствует пакету SDK для веб-заданий 2.*x*. Репозитории исходного используют нумерацию SDK для веб-заданий.
> * Пример кода для библиотек классов C# решения "Функции Azure" похож на код пакета SDK для веб-заданий, за исключением того, что для проекта пакета SDK для веб-заданий не требуется атрибут `FunctionName`.
> * Некоторые типы привязки поддерживаются только в решении "Функции Azure", например HTTP (веб-перехватчик) и сетка событий (на основе HTTP).
>
> Дополнительные сведения см. в разделе [Сравнение функций и веб-заданий](../azure-functions/functions-compare-logic-apps-ms-flow-webjobs.md#compare-functions-and-webjobs).

## <a name="webjobs-host"></a>Узел веб-заданий

Узел — это контейнер среды выполнения для функций.  Он прослушивает триггеры и вызовы функций. В версии 3.*x* узел является реализацией `IHost`. В версии 2.*x* используется объект `JobHost`. Вам нужно создать экземпляр узла в коде и написать код, чтобы настроить его работу.

Этим работа напрямую с пакетом SDK для веб-заданий отличается от работы с использованием решения "Функции Azure". В решении "Функции Azure" эта служба управляет узлом и ее нельзя настроить путем написания кода. Это решение позволяет настроить поведение узла с помощью настроек в файле host.json. Эти настройки представляют собой строки, а не код, что ограничивает диапазон применимых типов настроек.

### <a name="host-connection-strings"></a>Строки подключения узла

Пакет SDK для веб-заданий ищет строки подключения служебной шины Azure и службы хранилища Azure в файле local.settings.json при локальном запуске или в среде веб-заданий при запуске в Azure. По умолчанию параметр строки подключения службы хранилища `AzureWebJobsStorage` является обязательным.  

Версия 2.*x* пакета SDK позволяет использовать собственные имена для этих строк подключения или хранить их в другом месте. Вы можете задать имена в коде с помощью [`JobHostConfiguration`], как показано ниже.

```cs
static void Main(string[] args)
{
    var _storageConn = ConfigurationManager
        .ConnectionStrings["MyStorageConnection"].ConnectionString;

    //// Dashboard logging is deprecated; use Application Insights.
    //var _dashboardConn = ConfigurationManager
    //    .ConnectionStrings["MyDashboardConnection"].ConnectionString;

    JobHostConfiguration config = new JobHostConfiguration();
    config.StorageConnectionString = _storageConn;
    //config.DashboardConnectionString = _dashboardConn;
    JobHost host = new JobHost(config);
    host.RunAndBlock();
}
```

> [!NOTE]
> Так как он использует API конфигурации .NET Core по умолчанию, в версии 3.*x* отсутствует API для изменения имен строк подключения. См. раздел [Разработка и развертывание веб-заданий с помощью Visual Studio](webjobs-dotnet-deploy-vs.md) .

### <a name="host-development-settings"></a>Параметры разработки узла

Вы можете запустить узел в режиме разработки, чтобы повысить эффективность локальной разработки. Ниже приведены некоторые параметры, которые изменяются при работе в режиме разработки.

| Свойство | Параметр разработки |
| ------------- | ------------- |
| `Tracing.ConsoleLevel` | `TraceLevel.Verbose` — чтобы увеличить выходные данные журнала. |
| `Queues.MaxPollingInterval`  | Низкое значение для обеспечения немедленного запуска методов очереди.  |
| `Singleton.ListenerLockPeriod` | 15 секунд для ускорения последовательной разработки. |

Способ включения режима разработки зависит от версии пакета SDK. 

#### <a name="version-3x"></a>Версия 3.*x*

Версия 3.*x* использует стандартные API ASP.NET Core. Вызовите метод [`UseEnvironment`](/dotnet/api/microsoft.extensions.hosting.hostinghostbuilderextensions.useenvironment) в экземпляре [`HostBuilder`](/dotnet/api/microsoft.extensions.hosting.hostbuilder). Передайте строку с именем `development`, как показано в следующем примере.

```cs
static async Task Main()
{
    var builder = new HostBuilder();
    builder.UseEnvironment("development");
    builder.ConfigureWebJobs(b =>
            {
                b.AddAzureStorageCoreServices();
            });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

#### <a name="version-2x"></a>Версия 2.*x*

В классе `JobHostConfiguration` имеется метод `UseDevelopmentSettings`, который включает режим разработки.  В следующем примере показано, как использовать параметры разработки. Чтобы `config.IsDevelopment` возвращал значение `true` при локальном запуске, установите локальную переменную среды `AzureWebJobsEnv` со значением `Development`.

```cs
static void Main()
{
    config = new JobHostConfiguration();

    if (config.IsDevelopment)
    {
        config.UseDevelopmentSettings();
    }

    var host = new JobHost(config);
    host.RunAndBlock();
}
```

### <a name="managing-concurrent-connections-version-2x"></a><a name="jobhost-servicepointmanager-settings"></a>Управление одновременными подключениями (версия 2.*x*)

В версии 3.*x* ограничение числа подключений сбрасывается к значению по умолчанию (бесконечное число подключений). Если по какой-либо причине необходимо изменить это ограничение, вы можете использовать свойство [`MaxConnectionsPerServer`](/dotnet/api/system.net.http.winhttphandler.maxconnectionsperserver) класса [`WinHttpHandler`](/dotnet/api/system.net.http.winhttphandler).

В версии 2.*x* вы контролируете количество одновременных подключений к узлу с помощью API-интерфейса [ServicePointManager.DefaultConnectionLimit](/dotnet/api/system.net.servicepointmanager.defaultconnectionlimit#System_Net_ServicePointManager_DefaultConnectionLimit). В версии 2.*x* перед запуском узла веб-заданий необходимо увеличить это значение со значения по умолчанию 2.

Все исходящие HTTP-запросы, выполняемые из функции, используя `HttpClient`, проходят через `ServicePointManager`. Когда достигается значение, установленное в `DefaultConnectionLimit`, `ServicePointManager` начинает добавлять запросы в очередь перед их отправкой. Предположим, что для ограничения `DefaultConnectionLimit` установлено значение 2 и код включает 1000 HTTP-запросов. Первоначально в ОС разрешено только два запроса. Остальные 998 поставлены в очередь, пока не появится место для них. Это означает, что может истечь время ожидания `HttpClient`, потому что считается, что запрос выполнен, но на самом деле он никогда не отправлялся ОС на целевой сервер. Таким образом, вы можете столкнуться с поведением, которое не имеет смысла: локальному клиенту `HttpClient` потребуется 10 секунд, чтобы выполнить запрос, но служба будет возвращать каждый запрос за 200 мс. 

Значение по умолчанию для приложений ASP.NET — `Int32.MaxValue`, и это, вероятно, эффективно для веб-заданий, запущенных в базовом плане службы приложений или плане более высокого уровня. Веб-заданиям обычно требуется параметр Always On, который поддерживается только базовыми планами службы приложений или планами более высокого уровня.

Если ваше задание WebJob работает в бесплатном или общем плане службы приложений, приложение ограничено песочницей службы приложений, которая в настоящее время имеет такое [ограничение подключений — 300](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox#per-sandbox-per-appper-site-numerical-limits). При неограниченном подключении в `ServicePointManager` более вероятно, что будет достигнут порог подключения песочницы, а сайт отключится. В этом случае установка более низкого значения параметра `DefaultConnectionLimit`, например 50 или 100, может не допустить события выше и обеспечить достаточную пропускную способность.

Параметр нужно настроить перед выполнением HTTP-запросов. По этой причине узел веб-заданий не должен настраивать параметр автоматически. HTTP-запросы могут выполняться перед запуском узла, что может привести к непредвиденному поведению. Оптимальный подход — сразу же установить значение в методе `Main` перед инициализацией `JobHost`, как показано в следующем примере.

```csharp
static void Main(string[] args)
{
    // Set this immediately so that it's used by all requests.
    ServicePointManager.DefaultConnectionLimit = Int32.MaxValue;

    var host = new JobHost();
    host.RunAndBlock();
}
```

## <a name="triggers"></a>Триггеры

Функции должны быть общедоступными методами и иметь один атрибут триггера или атрибут [`NoAutomaticTrigger`](#manual-triggers).

### <a name="automatic-triggers"></a>Автоматические триггеры

Автоматические триггеры вызывают функцию в ответ на событие. Рассмотрим этот пример функции, которая запускается сообщением, добавленным в хранилище очередей Azure. Она реагирует на считывание большого двоичного объекта из хранилища BLOB-объектов Azure.

```cs
public static void Run(
    [QueueTrigger("myqueue-items")] string myQueueItem,
    [Blob("samples-workitems/{queueTrigger}", FileAccess.Read)] Stream myBlob,
    ILogger log)
{
    log.LogInformation($"BlobInput processed blob\n Name:{myQueueItem} \n Size: {myBlob.Length} bytes");
}
```

Например, атрибут `QueueTrigger` вызывает в среде выполнения функцию всякий раз, когда в очереди `myqueue-items` появляется очередное сообщение. Атрибут `Blob` вынуждает среду выполнения использовать сообщение из очереди для чтения большого двоичного объекта из контейнера *sample-workitems*. Имя элемента BLOB-объекта в контейнере `samples-workitems` получено непосредственно от триггера очереди как выражение привязки (`{queueTrigger}`).

[!INCLUDE [webjobs-always-on-note](../../includes/webjobs-always-on-note.md)]

### <a name="manual-triggers"></a>Ручные триггеры

Чтобы запустить функцию вручную, используйте атрибут `NoAutomaticTrigger`, как показано в следующем примере.

```cs
[NoAutomaticTrigger]
public static void CreateQueueMessage(
ILogger logger,
string value,
[Queue("outputqueue")] out string message)
{
    message = value;
    logger.LogInformation("Creating queue message: ", message);
}
```

Процесс запуска функции вручную зависит от версии пакета SDK.

#### <a name="version-3x"></a>Версия 3.*x*

```cs
static async Task Main(string[] args)
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
    {
        b.AddAzureStorageCoreServices();
        b.AddAzureStorage();
    });
    var host = builder.Build();
    using (host)
    {
        var jobHost = host.Services.GetService(typeof(IJobHost)) as JobHost;
        var inputs = new Dictionary<string, object>
        {
            { "value", "Hello world!" }
        };

        await host.StartAsync();
        await jobHost.CallAsync("CreateQueueMessage", inputs);
        await host.StopAsync();
    }
}
```

#### <a name="version-2x"></a>Версия 2.*x*

```cs
static void Main(string[] args)
{
    JobHost host = new JobHost();
    host.Call(typeof(Program).GetMethod("CreateQueueMessage"), new { value = "Hello world!" });
}
```

## <a name="input-and-output-bindings"></a>Входные и выходные привязки

Входные привязки предоставляют декларативный способ получения данных для кода из внешних служб или служб Azure. Выходные привязки обеспечивают способ обновления данных. Примеры привязок приводятся в статье [Начало работы](webjobs-sdk-get-started.md).

Возвращаемое значение метода можно использовать для привязки выходных данных. Для этого примените атрибут к возвращаемому значению метода. См. пример в разделе [Использование возвращаемого значения решения "Функции Azure"](../azure-functions/functions-bindings-return-value.md).

## <a name="binding-types"></a>Типы привязки

Процесс установки и управления типами привязок зависит от того, используете ли вы версию 3.*x* или 2.*x* пакета SDK. Вы можете найти пакет, устанавливаемый для определенного типа привязки, в разделе "Пакеты" [справочной статьи](#binding-reference-information) для соответствующего типа привязки для решения "Функции Azure". Исключением является триггер и привязка файлов (для локальной файловой системы), которая не поддерживается решением "Функции Azure".

#### <a name="version-3x"></a>Версия 3.*x*

В версии 3.*x* привязки хранилища включены в пакет `Microsoft.Azure.WebJobs.Extensions.Storage`. Вызовите метод расширения `AddAzureStorage` в методе `ConfigureWebJobs`, как показано далее.

```cs
static async Task Main()
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
            {
                b.AddAzureStorageCoreServices();
                b.AddAzureStorage();
            });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

Чтобы использовать другие типы триггеров и привязок, установите пакет NuGet, который их содержит, и вызовите метод расширения `Add<binding>`, реализованный в расширении. Например, если вы хотите использовать привязку Azure Cosmos DB, установите `Microsoft.Azure.WebJobs.Extensions.CosmosDB` и вызовите `AddCosmosDB`, как показано далее.

```cs
static async Task Main()
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
            {
                b.AddAzureStorageCoreServices();
                b.AddCosmosDB();
            });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

Чтобы использовать триггер таймера или привязку файлов, являющиеся частью основных служб, вызовите методы расширения `AddTimers` или `AddFiles` соответственно.

#### <a name="version-2x"></a>Версия 2.*x*

В пакет  *версии 2.* x`Microsoft.Azure.WebJobs` включены следующие типы триггеров и привязок.

* Хранилище BLOB-объектов
* Хранилище очередей
* Хранилище таблиц

Чтобы использовать другие типы триггеров и привязок, установите пакет NuGet, который их содержит, и вызовите метод `Use<binding>` для объекта `JobHostConfiguration`. Например, если вы хотите использовать триггер таймера, установите `Microsoft.Azure.WebJobs.Extensions` и вызовите `UseTimers` в методе `Main`, как показано далее.

```cs
static void Main()
{
    config = new JobHostConfiguration();
    config.UseTimers();
    var host = new JobHost(config);
    host.RunAndBlock();
}
```

Чтобы использовать привязку файлов, установите `Microsoft.Azure.WebJobs.Extensions` и вызовите `UseFiles`.

### <a name="executioncontext"></a>ExecutionContext

Веб-задания позволяют выполнить привязку к [`ExecutionContext`]. С помощью этой привязки вы можете получить доступ к параметру [`ExecutionContext`] в сигнатуре функции. Например, следующий код использует объект контекста для доступа к идентификатору вызова, который вы можете использовать для корреляции всех журналов, созданных данным вызовом функции.  

```cs
public class Functions
{
    public static void ProcessQueueMessage([QueueTrigger("queue")] string message,
        ExecutionContext executionContext,
        ILogger logger)
    {
        logger.LogInformation($"{message}\n{executionContext.InvocationId}");
    }
}
```

Процесс привязки к [`ExecutionContext`] зависит от версии пакета SDK.

#### <a name="version-3x"></a>Версия 3.*x*

Вызовите метод расширения `AddExecutionContextBinding` в методе `ConfigureWebJobs`, как показано далее.

```cs
static async Task Main()
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
            {
                b.AddAzureStorageCoreServices();
                b.AddExecutionContextBinding();
            });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

#### <a name="version-2x"></a>Версия 2.*x*

Пакет `Microsoft.Azure.WebJobs.Extensions`, упомянутый ранее, также предоставляет специальный тип привязки, который вы можете зарегистрировать, вызвав метод `UseCore`. Эта привязка позволяет определить параметр [`ExecutionContext`] в сигнатуре функции, который включается следующим образом.

```cs
class Program
{
    static void Main()
    {
        config = new JobHostConfiguration();
        config.UseCore();
        var host = new JobHost(config);
        host.RunAndBlock();
    }
}
```

## <a name="binding-configuration"></a>Конфигурация привязки

Можно настроить поведение некоторых триггеров и привязок. Процесс настройки зависит от версии пакета SDK.

* **Версия 3.*x*:** конфигурация настраивается при вызове метода `Add<Binding>` в `ConfigureWebJobs`.
* **Версия 2.*x*:** конфигурация настраивается путем настройки свойств в объекте конфигурации, который передается в `JobHost`.

Эти зависимые от привязки параметры эквивалентны параметрам в [файле проекта host.json](../azure-functions/functions-host-json.md) в решении "Функции Azure".

Можно настроить следующие привязки.

* [Триггер Azure CosmosDB](#azure-cosmosdb-trigger-configuration-version-3x)
* [Триггер Центров событий](#event-hubs-trigger-configuration-version-3x)
* [Триггер хранилища очередей](#queue-storage-trigger-configuration)
* [Привязка SendGrid](#sendgrid-binding-configuration-version-3x)
* [Триггер служебной шины](#service-bus-trigger-configuration-version-3x)

### <a name="azure-cosmosdb-trigger-configuration-version-3x"></a>Конфигурация триггера Azure CosmosDB (версия 3.*x*)

В этом примере показано, как настраивать и просматривать журналы триггеров Azure Cosmos DB.

```cs
static async Task Main()
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
    {
        b.AddAzureStorageCoreServices();
        b.AddCosmosDB(a =>
        {
            a.ConnectionMode = ConnectionMode.Gateway;
            a.Protocol = Protocol.Https;
            a.LeaseOptions.LeasePrefix = "prefix1";

        });
    });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

Дополнительные сведения см. в статье [Привязка Azure CosmosDB](../azure-functions/functions-bindings-cosmosdb-v2-output.md#hostjson-settings).

### <a name="event-hubs-trigger-configuration-version-3x"></a>Конфигурация триггеров Центров событий (версия 3.*x*)

В этом примере показано, как настроить триггер Центров событий.

```cs
static async Task Main()
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
    {
        b.AddAzureStorageCoreServices();
        b.AddEventHubs(a =>
        {
            a.BatchCheckpointFrequency = 5;
            a.EventProcessorOptions.MaxBatchSize = 256;
            a.EventProcessorOptions.PrefetchCount = 512;
        });
    });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

Дополнительные сведения см. в статье [Привязка Центров событий](../azure-functions/functions-bindings-event-hubs.md#host-json).

### <a name="queue-storage-trigger-configuration"></a>Конфигурация триггера хранилища очередей

В этих примерах показано, как настроить триггер Хранилища очередей.

#### <a name="version-3x"></a>Версия 3.*x*

```cs
static async Task Main()
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
    {
        b.AddAzureStorageCoreServices();
        b.AddAzureStorage(a => {
            a.BatchSize = 8;
            a.NewBatchThreshold = 4;
            a.MaxDequeueCount = 4;
            a.MaxPollingInterval = TimeSpan.FromSeconds(15);
        });
    });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

Дополнительные сведения см. в статье [Привязка Хранилища очередей](../azure-functions/functions-bindings-storage-queue-trigger.md#hostjson-properties).

#### <a name="version-2x"></a>Версия 2.*x*

```cs
static void Main(string[] args)
{
    JobHostConfiguration config = new JobHostConfiguration();
    config.Queues.BatchSize = 8;
    config.Queues.NewBatchThreshold = 4;
    config.Queues.MaxDequeueCount = 4;
    config.Queues.MaxPollingInterval = TimeSpan.FromSeconds(15);
    JobHost host = new JobHost(config);
    host.RunAndBlock();
}
```

Дополнительные сведения см. в [справочнике по host.json v1.x](../azure-functions/functions-host-json-v1.md#queues).

### <a name="sendgrid-binding-configuration-version-3x"></a>Настройка привязки SendGrid (версия 3.*x*)

В этом примере показано, как настроить привязку выходных данных SendGrid.

```cs
static async Task Main()
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
    {
        b.AddAzureStorageCoreServices();
        b.AddSendGrid(a =>
        {
            a.FromAddress.Email = "samples@functions.com";
            a.FromAddress.Name = "Azure Functions";
        });
    });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

Дополнительные сведения см. в статье [Привязка SendGrid](../azure-functions/functions-bindings-sendgrid.md#hostjson-settings).

### <a name="service-bus-trigger-configuration-version-3x"></a>Настройка триггера служебной шины (версия 3.*x*)

В этом примере показано, как настроить триггер служебной шины.

```cs
static async Task Main()
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
    {
        b.AddAzureStorageCoreServices();
        b.AddServiceBus(sbOptions =>
        {
            sbOptions.MessageHandlerOptions.AutoComplete = true;
            sbOptions.MessageHandlerOptions.MaxConcurrentCalls = 16;
        });
    });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

Дополнительные сведения см. в статье [Привязка служебной шины](../azure-functions/functions-bindings-service-bus-output.md#hostjson-settings).

### <a name="configuration-for-other-bindings"></a>Конфигурация для других привязок

Некоторые типы триггеров и привязок определяют собственные настраиваемые типы конфигурации. Например, триггер файла позволяет указать корневой путь для мониторинга, как показано в этих примерах.

#### <a name="version-3x"></a>Версия 3.*x*

```cs
static async Task Main()
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
    {
        b.AddAzureStorageCoreServices();
        b.AddFiles(a => a.RootPath = @"c:\data\import");
    });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

#### <a name="version-2x"></a>Версия 2.*x*

```cs
static void Main()
{
    config = new JobHostConfiguration();
    var filesConfig = new FilesConfiguration
    {
        RootPath = @"c:\data\import"
    };
    config.UseFiles(filesConfig);
    var host = new JobHost(config);
    host.RunAndBlock();
}
```

## <a name="binding-expressions"></a>Выражения привязки

В параметрах конструктора атрибутов вы можете использовать выражения, которые разрешают значения из разных источников. Например, в следующем коде путь для атрибута `BlobTrigger` создает выражение с именем `filename`. При использовании для выходной привязки `filename` разрешается к имени активирующего большого двоичного объекта.

```cs
public static void CreateThumbnail(
    [BlobTrigger("sample-images/{filename}")] Stream image,
    [Blob("sample-images-sm/{filename}", FileAccess.Write)] Stream imageSmall,
    string filename,
    ILogger logger)
{
    logger.Info($"Blob trigger processing: {filename}");
    // ...
}
```

Дополнительные сведения о выражениях привязки см. в разделе [Выражения привязки и шаблоны](../azure-functions/functions-bindings-expressions-patterns.md) в документации по решению "Функции Azure".

### <a name="custom-binding-expressions"></a>Настраиваемые выражения привязки

Иногда требуется указать в коде имя очереди, имя большого двоичного объекта, контейнера или таблицы, а не жестко задавать их. Например, в некоторых случаях в файле конфигурации или в переменной среды необходимо указывать имя очереди для атрибута `QueueTrigger`.

Для этого можно передать объект `NameResolver` объекту `JobHostConfiguration`. В параметрах конструктора атрибута привязки и триггера нужно указать специальные заполнители, тогда код `NameResolver` будет указывать фактические значения, которые следует использовать вместо этих заполнителей. Вы определяете заполнители, заключив их в знаки процента (%), как показано ниже.

```cs
public static void WriteLog([QueueTrigger("%logqueue%")] string logMessage)
{
    Console.WriteLine(logMessage);
}
```

Этот код позволяет использовать очередь с именем `logqueuetest` в тестовой среде и очередь с именем `logqueueprod` в производственной среде. Вместо фиксированного имени очереди требуется указать имя записи в коллекции `appSettings`.

Есть имя `NameResolver` по умолчанию, которое используется, если не задано настраиваемое имя. Стандартное средство получает значения из настроек приложения или переменных среды.

Класс `NameResolver` получает имя очереди от `appSettings`, как показано в следующем примере.

```cs
public class CustomNameResolver : INameResolver
{
    public string Resolve(string name)
    {
        return ConfigurationManager.AppSettings[name].ToString();
    }
}
```

#### <a name="version-3x"></a>Версия 3.*x*

Распознаватель настраивается путем добавления зависимостей. Для этих примеров необходим следующий оператор `using`:

```cs
using Microsoft.Extensions.DependencyInjection;
```

Сопоставитель добавляется путем вызова метода расширения [`ConfigureServices`] в [`HostBuilder`](/dotnet/api/microsoft.extensions.hosting.hostbuilder), как показано в этом примере.

```cs
static async Task Main(string[] args)
{
    var builder = new HostBuilder();
    var resolver = new CustomNameResolver();
    builder.ConfigureWebJobs(b =>
    {
        b.AddAzureStorageCoreServices();
    });
    builder.ConfigureServices(s => s.AddSingleton<INameResolver>(resolver));
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

#### <a name="version-2x"></a>Версия 2.*x*

Передайте класс `NameResolver` в объект `JobHost`, как показано ниже.

```cs
 static void Main(string[] args)
{
    JobHostConfiguration config = new JobHostConfiguration();
    config.NameResolver = new CustomNameResolver();
    JobHost host = new JobHost(config);
    host.RunAndBlock();
}
```

Решение "Функции Azure" реализует `INameResolver`, чтобы получить значения из настроек приложения, как показано в этом примере. Когда вы используете пакет SDK WebJobs напрямую, вы можете написать специальную реализацию, которая получает значения замены заполнителя из любого выбранного источника.

## <a name="binding-at-runtime"></a>Привязка во время выполнения

Если перед использованием атрибута привязки для веб-заданий, например `Queue`, `Blob` или `Table`, необходимо выполнить какие-либо действия с функцией, можно использовать интерфейс `IBinder`.

В следующем примере в выходной очереди создается новое сообщение с тем же содержимым, что и в сообщении входной очереди. Имя очереди вывода определяется кодом в теле функции.

```cs
public static void CreateQueueMessage(
    [QueueTrigger("inputqueue")] string queueMessage,
    IBinder binder)
{
    string outputQueueName = "outputqueue" + DateTime.Now.Month.ToString();
    QueueAttribute queueAttribute = new QueueAttribute(outputQueueName);
    CloudQueue outputQueue = binder.Bind<CloudQueue>(queueAttribute);
    outputQueue.AddMessageAsync(new CloudQueueMessage(queueMessage));
}
```

Дополнительные сведения см. в разделе [Привязка во время выполнения](../azure-functions/functions-dotnet-class-library.md#binding-at-runtime) в документации по решению "Функции Azure".

## <a name="binding-reference-information"></a>Справочная информация о привязке

В документации по решению "Функции Azure" содержится справочная информация о каждом типе привязки. В каждой справочной статье о привязке вы найдете следующие сведения. (Этот пример учитывает очередь хранилища.)

* [Пакеты](../azure-functions/functions-bindings-storage-queue.md). Сведения о том, какой пакет установить, чтобы включить поддержку привязки в проекте пакета SDK для веб-заданий.
* [Примеры](../azure-functions/functions-bindings-storage-queue-trigger.md). Примеры кода. Пример библиотеки классов C# применяется к пакету SDK для веб-заданий. Просто опустите атрибут `FunctionName`.
* [Атрибуты](../azure-functions/functions-bindings-storage-queue-trigger.md#attributes-and-annotations). Атрибуты, используемые для типа привязки.
* [Конфигурация](../azure-functions/functions-bindings-storage-queue-trigger.md#configuration). Объяснения свойств атрибута и параметров конструктора.
* [Использование](../azure-functions/functions-bindings-storage-queue-trigger.md#usage). Сведения о том, к каким типам можно применить привязку и как она работает. Например: алгоритм опроса, обработка подозрительной очереди.
  
Чтобы ознакомиться со списком справочных статей о привязке, просмотрите сведения в разделе о поддерживаемых привязках в статье [Триггеры и привязки](../azure-functions/functions-triggers-bindings.md#supported-bindings) для решения "Функции Azure". В этом списке привязки HTTP, веб-перехватчика и службы "Сетка событий" поддерживаются только с помощью решения "Функции Azure", а не пакета SDK для веб-заданий.

## <a name="disable-attribute"></a>Атрибут Disable 

Атрибут [`Disable`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/DisableAttribute.cs) позволяет управлять запуском функции. 

В следующем примере, если параметр приложения `Disable_TestJob` имеет значение `1` или `True` (без учета регистра), функция не запускается. В этом случае среда выполнения создает сообщение журнала о том, что *функция Functions.TestJob отключена*.

```cs
[Disable("Disable_TestJob")]
public static void TestJob([QueueTrigger("testqueue2")] string message)
{
    Console.WriteLine("Function with Disable attribute executed!");
}
```

Когда вы изменяете значения параметра приложения на портале Azure, нужно повторно запустить веб-задание с помощью нового параметра.

Атрибут может объявляться на уровне класса, метода или параметра. Имя параметра также может содержать выражения привязки.

## <a name="timeout-attribute"></a>Атрибут Timeout

Атрибут [`Timeout`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/TimeoutAttribute.cs) требует отмены функции, если она не завершена в течение определенного периода времени. В следующем примере функция будет работать в течение одного дня без использования атрибута Timeout. При использовании атрибута Timeout функция будет отменена через 15 секунд.

```cs
[Timeout("00:00:15")]
public static async Task TimeoutJob(
    [QueueTrigger("testqueue2")] string message,
    CancellationToken token,
    TextWriter log)
{
    await log.WriteLineAsync("Job starting");
    await Task.Delay(TimeSpan.FromDays(1), token);
    await log.WriteLineAsync("Job completed");
}
```

Вы можете применить атрибут Timeout на уровне класса или метода и указать глобальное время ожидания, используя `JobHostConfiguration.FunctionTimeout`. Время ожидания на уровне класса или метода переопределяет глобальное время ожидания.

## <a name="singleton-attribute"></a>Атрибут Singleton

Используйте атрибут [`Singleton`](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/SingletonAttribute.cs), чтобы обеспечить выполнение только одного экземпляра функции даже при наличии нескольких экземпляров несущего веб-приложения. Такое выполнение обеспечивается с помощью [распределенного блокирования](#viewing-lease-blobs).

В этом примере только один экземпляр функции `ProcessImage` запускается в любой момент времени.

```cs
[Singleton]
public static async Task ProcessImage([BlobTrigger("images")] Stream image)
{
     // Process the image.
}
```

### <a name="singletonmodelistener"></a>SingletonMode.Listener

У некоторых триггеров есть встроенная поддержка для управления параллелизмом:

* **QueueTrigger**. Присвойте параметру `JobHostConfiguration.Queues.BatchSize` значение `1`.
* **ServiceBusTrigger**. Присвойте параметру `ServiceBusConfiguration.MessageOptions.MaxConcurrentCalls` значение `1`.
* **FileTrigger**. Присвойте параметру `FileProcessor.MaxDegreeOfParallelism` значение `1`.

Эти параметры можно использовать для обеспечения отдельного выполнения функции в одном экземпляре. Чтобы гарантировать, что запущен только один экземпляр функции, если веб-приложение масштабируется до нескольких экземпляров, примените блокировку Singleton для уровня прослушивателя в функции (`[Singleton(Mode = SingletonMode.Listener)]`). Блокировки прослушивателя инициируются при запуске узла JobHost. Если все три горизонтально масштабированные экземпляры запускаются одновременно, блокировки требует только один из экземпляров, а запускается только один прослушиватель.

> [!NOTE]
> Дополнительные сведения о работе Синглетонмоде. function см. в этом [репозитории GitHub](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/SingletonMode.cs) .

### <a name="scope-values"></a>Значения области

Можно указать *выражение или значение области* для атрибута Singleton. Это выражение/значение гарантирует, что все выполнения функции в определенной области будут сериализованы. Реализация более специализированной блокировки может обеспечить некоторый уровень параллелизма функции при сохранении сериализации других вызовов соответственно с вашими требованиями. В следующем коде рассматриваются привязки выражения области к значению `Region` входящего сообщения. Когда очередь содержит три сообщения в регионах "Восток", "Восток" и "Запад", тогда сообщения с регионом "Восток" выполняются последовательно, в то время как сообщение с регионом "Запад" выполняется параллельно с сообщениями региона "Восток".

```csharp
[Singleton("{Region}")]
public static async Task ProcessWorkItem([QueueTrigger("workitems")] WorkItem workItem)
{
     // Process the work item.
}

public class WorkItem
{
     public int ID { get; set; }
     public string Region { get; set; }
     public int Category { get; set; }
     public string Description { get; set; }
}
```

### <a name="singletonscopehost"></a>SingletonScope.Host

Областью по умолчанию для блокировки является `SingletonScope.Function`. Это означает, что область блокировки (путь аренды большого двоичного объекта) привязана к полному имени функции. Чтобы заблокировать функции, укажите узел `SingletonScope.Host` и используйте имя идентификатора области, одинаковое для всех функций, которые не нужно запускать одновременно. В следующем примере выполняется только один экземпляр `AddItem` или `RemoveItem`:

```csharp
[Singleton("ItemsLock", SingletonScope.Host)]
public static void AddItem([QueueTrigger("add-item")] string message)
{
     // Perform the add operation.
}

[Singleton("ItemsLock", SingletonScope.Host)]
public static void RemoveItem([QueueTrigger("remove-item")] string message)
{
     // Perform the remove operation.
}
```

### <a name="viewing-lease-blobs"></a>Просмотр арендуемых больших двоичных объектов

Пакет SDK WebJobs использует аренды [больших двоичных объектов Azure](../storage/blobs/concurrency-manage.md#pessimistic-concurrency-for-blobs) для реализации распределенной блокировки. Арендуемые большие двоичные объекты, используемые Singleton, можно найти в контейнере `azure-webjobs-host` в учетной записи хранения `AzureWebJobsStorage` по пути locks. Например, путь к арендуемому большому двоичному объекту для первого примера `ProcessImage`, рассмотренного ранее, должен быть таким: `locks/061851c758f04938a4426aa9ab3869c0/WebJobs.Functions.ProcessImage`. Все пути включают идентификатор JobHost, в данном случае 061851c758f04938a4426aa9ab3869c0.

## <a name="async-functions"></a>Асинхронные функции

Информацию о том, как закодировать асинхронные функции, см. в [документации по решению "Функции Azure"](../azure-functions/functions-dotnet-class-library.md#async).

## <a name="cancellation-tokens"></a>Токены отмены

Информацию о том, как обрабатывать маркеры отмены, см. в документации по решению "Функции Azure" в разделе [Токены отмены](../azure-functions/functions-dotnet-class-library.md#cancellation-tokens).

## <a name="multiple-instances"></a>Несколько экземпляров

Если ваше веб-приложение работает в нескольких экземплярах, на каждом экземпляре запускается непрерывный WebJob, который прослушивает триггеры и вызывает функции. Различные привязки триггера предназначены для эффективной совместной работы в экземплярах, чтобы масштабирование до большего количества экземпляров позволяло обрабатывать большую нагрузку.

Хотя некоторые триггеры могут стать причиной двойной обработки, триггеры очереди и большого двоичного объекта автоматически препятствуют тому, чтобы функция обрабатывала сообщение очереди или большой двоичный объект более одного раза. Дополнительные сведения см. в разделе [Проектирование с учетом идентичных входных данных](../azure-functions/functions-idempotent.md) в документации по решению "Функции Azure".

Триггер таймера автоматически гарантирует, что запускается только один экземпляр таймера, поэтому вы не получаете больше одного экземпляра функции, запущенного в заданное время.

Если вы хотите, чтобы выполнялся только один экземпляр функции, даже если существует несколько экземпляров несущего веб-приложения, можно использовать атрибут [`Singleton`](#singleton-attribute).

## <a name="filters"></a>Фильтры

Функциональные фильтры (предварительная версия) обеспечивают способ настройки конвейера выполнения WebJobs с помощью вашей собственной логики. Фильтры сходны с [фильтрами ASP.NET Core](/aspnet/core/mvc/controllers/filters). Их можно реализовать как декларативные атрибуты, которые применяются к функциям или классам. Дополнительные сведения см. на странице о [фильтрах функции](https://github.com/Azure/azure-webjobs-sdk/wiki/Function-Filters).

## <a name="logging-and-monitoring"></a>Ведение журналов и мониторинг

Мы рекомендуем использовать платформу журналирования, разработанную для ASP.NET. В статье [Начало работы](webjobs-sdk-get-started.md) показано, как используется эта платформа. 

### <a name="log-filtering"></a>Фильтрация журнала

Каждый журнал, созданный экземпляром `ILogger`, имеет связанные параметры `Category` и `Level`. Параметр [`LogLevel`](/dotnet/api/microsoft.extensions.logging.loglevel) является перечислением целочисленных значений, которые обозначают относительную важность.

|LogLevel    |Код|
|------------|---|
|Трассировка       | 0 |
|Отладка       | 1 |
|Сведения | 2 |
|Предупреждение     | 3 |
|Error       | 4 |
|Critical    | 5 |
|None        | 6 |

Каждую категорию можно независимо отфильтровать в соответствии с определенным [`LogLevel`](/dotnet/api/microsoft.extensions.logging.loglevel). Например, вы можете просмотреть все журналы для обработки триггера большого двоичного объекта, но только `Error` и выше для всего остального.

#### <a name="version-3x"></a>Версия 3.*x*

Версия 3.*x* пакета SDK использует фильтрацию, встроенную в .NET Core. Класс `LogCategories` позволяет определить категории для конкретных функций, триггеров или пользователей. Он также определяет фильтры для определенных состояний узла, таких как `Startup` и `Results`. Это позволяет точно настроить выходные данные журнала. Если в определенных категориях не найдено совпадение, фильтр возвращается к значению `Default` при принятии решения о фильтрации сообщения.

Добавьте в `LogCategories` следующий оператор:

```cs
using Microsoft.Azure.WebJobs.Logging; 
```

В следующем примере создается фильтр, который по умолчанию фильтрует все журналы на уровне `Warning`. Категории `Function` или `results` (эквивалентные `Host.Results` в версии 2.*x*) фильтруются на уровне `Error`. Фильтр сравнивает текущую категорию со всеми зарегистрированными уровнями в экземпляре `LogCategories` и выбирает самое длинное совпадение. Это означает, что уровень `Debug`, зарегистрированный для `Host.Triggers`, будет соответствовать `Host.Triggers.Queue` или `Host.Triggers.Blob`. Это позволит вам управлять более широкими категориями, не добавляя их.

```cs
static async Task Main(string[] args)
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
    {
        b.AddAzureStorageCoreServices();
    });
    builder.ConfigureLogging(logging =>
            {
                logging.SetMinimumLevel(LogLevel.Warning);
                logging.AddFilter("Function", LogLevel.Error);
                logging.AddFilter(LogCategories.CreateFunctionCategory("MySpecificFunctionName"),
                    LogLevel.Debug);
                logging.AddFilter(LogCategories.Results, LogLevel.Error);
                logging.AddFilter("Host.Triggers", LogLevel.Debug);
            });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

#### <a name="version-2x"></a>Версия 2.*x*

В версии 2.*x* пакета SDK класс `LogCategoryFilter` используется для управления фильтрацией. Фильтр `LogCategoryFilter` имеет свойство `Default` с начальным значением `Information`, что означает, что все сообщения с уровнями `Information`, `Warning`, `Error` или `Critical` регистрируются в журнале, а любые сообщения с уровнями `Debug` или `Trace` отфильтровываются.

Как и `LogCategories` в версии 3.*x*, свойство `CategoryLevels` позволяет указать уровни журналов для определенных категорий, чтобы вы могли точно настроить выходные данные журнала. Если в словаре `CategoryLevels` не найдено совпадение, фильтр возвращается к значению `Default` при принятии решения о фильтрации сообщения.

В следующем примере создается фильтр, который по умолчанию фильтрует все журналы на уровне `Warning`. Категории `Function` или `Host.Results` фильтруются на уровне `Error`. Фильтр `LogCategoryFilter` сравнивает текущую категорию со всеми зарегистрированными уровнями `CategoryLevels` и выбирает самое длинное совпадение. Это означает, что уровень `Debug`, зарегистрированный для `Host.Triggers`, будет соответствовать `Host.Triggers.Queue` или `Host.Triggers.Blob`. Это позволит вам управлять более широкими категориями, не добавляя их.

```csharp
var filter = new LogCategoryFilter();
filter.DefaultLevel = LogLevel.Warning;
filter.CategoryLevels[LogCategories.Function] = LogLevel.Error;
filter.CategoryLevels[LogCategories.Results] = LogLevel.Error;
filter.CategoryLevels["Host.Triggers"] = LogLevel.Debug;

config.LoggerFactory = new LoggerFactory()
    .AddApplicationInsights(instrumentationKey, filter.Filter)
    .AddConsole(filter.Filter);
```

### <a name="custom-telemetry-for-application-insights"></a>Пользовательские данные телеметрии для Application Insights

Процесс реализации пользовательской телеметрии для [Application Insights](../azure-monitor/app/app-insights-overview.md) зависит от версии пакета SDK. Чтобы узнать, как настроить Application Insights, ознакомьтесь с разделом [Добавление журнала Application Insights](webjobs-sdk-get-started.md#add-application-insights-logging).

#### <a name="version-3x"></a>Версия 3.*x*

Так как версия 3.*x* пакета SDK для веб-заданий использует универсальный узел .NET Core, настраиваемая фабрика данных телеметрии больше не предоставляется. Однако вы можете добавить пользовательскую телеметрию в конвейер, используя внедрение зависимостей. Для примеров в этом разделе необходимы следующие операторы `using`:

```cs
using Microsoft.ApplicationInsights.Extensibility;
using Microsoft.ApplicationInsights.Channel;
```

Следующая пользовательская реализация [`ITelemetryInitializer`] позволяет добавлять собственную [`ITelemetry`](/dotnet/api/microsoft.applicationinsights.channel.itelemetry) в [`TelemetryConfiguration`] по умолчанию.

```cs
internal class CustomTelemetryInitializer : ITelemetryInitializer
{
    public void Initialize(ITelemetry telemetry)
    {
        // Do something with telemetry.
    }
}
```

Вызовите [`ConfigureServices`] в конструкторе, чтобы добавить пользовательский [`ITelemetryInitializer`] в конвейер.

```cs
static async Task Main()
{
    var builder = new HostBuilder();
    builder.ConfigureWebJobs(b =>
    {
        b.AddAzureStorageCoreServices();
    });
    builder.ConfigureLogging((context, b) =>
    {
        // Add logging providers.
        b.AddConsole();

        // If this key exists in any config, use it to enable Application Insights.
        string appInsightsKey = context.Configuration["APPINSIGHTS_INSTRUMENTATIONKEY"];
        if (!string.IsNullOrEmpty(appInsightsKey))
        {
            // This uses the options callback to explicitly set the instrumentation key.
            b.AddApplicationInsights(o => o.InstrumentationKey = appInsightsKey);
        }
    });
    builder.ConfigureServices(services =>
        {
            services.AddSingleton<ITelemetryInitializer, CustomTelemetryInitializer>();
        });
    var host = builder.Build();
    using (host)
    {
        await host.RunAsync();
    }
}
```

Когда [`TelemetryConfiguration`] создана, все зарегистрированные типы [`ITelemetryInitializer`] включены. Дополнительные сведения см. в разделе [API Application Insights для настраиваемых событий и метрик](../azure-monitor/app/api-custom-events-metrics.md).

В версии 3.*x* больше не нужно освобождать [`TelemetryClient`] при остановке узла. Система внедрения зависимостей .NET Core автоматически удаляет зарегистрированный `ApplicationInsightsLoggerProvider`, который очищает [`TelemetryClient`].

#### <a name="version-2x"></a>Версия 2.*x*

В версии 2.*x* [`TelemetryClient`], созданный изнутри поставщиком Application Insights для пакета SDK для веб-заданий, использует [`ServerTelemetryChannel`](https://github.com/microsoft/ApplicationInsights-dotnet/tree/develop/.publicApi/Microsoft.AI.ServerTelemetryChannel.dll). Когда конечная точка Application Insights недоступна или регулирует входящие запросы, этот канал [сохраняет запросы в файловой системе веб-приложения и позже повторно отправляет их](https://apmtips.com/posts/2015-09-03-more-telemetry-channels/).

[`TelemetryClient`] создается классом, который реализует `ITelemetryClientFactory`. По умолчанию это [`DefaultTelemetryClientFactory`](https://github.com/Azure/azure-webjobs-sdk/blob/dev/src/Microsoft.Azure.WebJobs.Logging.ApplicationInsights/).

Если вы хотите изменить любую часть конвейера Application Insights, вы можете указать свою фабрику `ITelemetryClientFactory`, и узел будет использовать ваш класс для создания клиента [`TelemetryClient`]. Например, этот код переопределяет `DefaultTelemetryClientFactory`, чтобы изменить свойство `ServerTelemetryChannel`.

```csharp
private class CustomTelemetryClientFactory : DefaultTelemetryClientFactory
{
    public CustomTelemetryClientFactory(string instrumentationKey, Func<string, LogLevel, bool> filter)
        : base(instrumentationKey, new SamplingPercentageEstimatorSettings(), filter)
    {
    }

    protected override ITelemetryChannel CreateTelemetryChannel()
    {
        ServerTelemetryChannel channel = new ServerTelemetryChannel();

        // Change the default from 30 seconds to 15 seconds.
        channel.MaxTelemetryBufferDelay = TimeSpan.FromSeconds(15);

        return channel;
    }
}
```

Объект `SamplingPercentageEstimatorSettings` настраивает [адаптивную выборку](../azure-monitor/app/sampling.md). Это означает, что в некоторых сценариях с большим количеством операций Applications Insights отправляет выбранное подмножество данных телеметрии на сервер.

После создания фабрики данных телеметрии ее нужно передать поставщику ведения журнала Application Insights.

```csharp
var clientFactory = new CustomTelemetryClientFactory(instrumentationKey, filter.Filter);

config.LoggerFactory = new LoggerFactory()
    .AddApplicationInsights(clientFactory);
```

## <a name="next-steps"></a><a id="nextsteps"></a> Дальнейшие действия

В этой статье приведены фрагменты кода, демонстрирующие, как обрабатывать общие сценарии для работы с пакетом SDK для веб-заданий. Полные примеры см. на странице с примерами [azure-webjobs-sdk-samples](https://github.com/Azure/azure-webjobs-sdk/tree/dev/sample/SampleHost).

[ExecutionContext]: https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions/Extensions/Core/ExecutionContext.cs
[TelemetryClient]: /dotnet/api/microsoft.applicationinsights.telemetryclient
[ConfigureServices]: /dotnet/api/microsoft.extensions.hosting.hostinghostbuilderextensions.configureservices
[ITelemetryInitializer]: /dotnet/api/microsoft.applicationinsights.extensibility.itelemetryinitializer
[TelemetryConfiguration]: /dotnet/api/microsoft.applicationinsights.extensibility.telemetryconfiguration
[JobHostConfiguration]: https://github.com/Azure/azure-webjobs-sdk/blob/v2.x/src/Microsoft.Azure.WebJobs.Host/JobHostConfiguration.cs