---
title: Мониторинг пользовательских операций с помощью пакета SDK Azure Application Insights .NET
description: Отслеживание пользовательских операций с помощью пакета SDK .NET Azure Application Insights
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 11/26/2019
ms.reviewer: sergkanz
ms.openlocfilehash: 42a5318325f9961483465357403089755feb130d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88933313"
---
# <a name="track-custom-operations-with-application-insights-net-sdk"></a>Отслеживание пользовательских операций с помощью пакета SDK Application Insights для .NET

Пакеты SDK Azure Application Insights автоматически отслеживают входящие HTTP-запросы и вызовы зависимых служб, в том числе HTTP-запросы и SQL-запросы. Отслеживание и корреляция запросов и зависимостей позволяет контролировать скорость реагирования приложения в целом, а также надежность всех микрослужб, составляющих это приложение. 

Существует класс шаблонов приложений, которые в принципе не поддерживаются. Для надлежащего мониторинга таких шаблонов требуется инструментирование кода вручную. В этой статье описывается несколько шаблонов, которые могут потребовать ручного инструментирования, например обработка пользовательской очереди сообщений и выполнение продолжительных фоновых задач.

В данном документе представлены инструкции по организации отслеживания пользовательских операций с помощью пакета SDK Application Insights. Эта документация предназначена для:

- Application Insights для .NET (базовый пакет SDK) версии 2.4 или более поздней версии;
- Application Insights для веб-приложений (выполнение ASP.NET) версии 2.4 или более поздней версии;
- Application Insights для ASP.NET Core версии 2.1 или более поздней версии.

## <a name="overview"></a>Обзор
Операция — это логический элемент работы, выполняемый приложением. У нее есть имя, время запуска, длительность, результат и контекст выполнения, такой как имя пользователя, свойства и результат. Если операция A была инициирована операцией B, то операция B является родительской для A. У операции может быть только одна родительская операция, но множество дочерних операций. Дополнительные сведения об операциях и корреляции телеметрии см. в разделе [Корреляция данных телеметрии в Application Insights](correlation.md).

В пакете SDK Application Insights для .NET операция описывается абстрактным классом [OperationTelemetry](https://github.com/microsoft/ApplicationInsights-dotnet/blob/7633ae849edc826a8547745b6bf9f3174715d4bd/BASE/src/Microsoft.ApplicationInsights/Extensibility/Implementation/OperationTelemetry.cs) и его потомками [RequestTelemetry](https://github.com/microsoft/ApplicationInsights-dotnet/blob/7633ae849edc826a8547745b6bf9f3174715d4bd/BASE/src/Microsoft.ApplicationInsights/DataContracts/RequestTelemetry.cs) и [DependencyTelemetry](https://github.com/microsoft/ApplicationInsights-dotnet/blob/7633ae849edc826a8547745b6bf9f3174715d4bd/BASE/src/Microsoft.ApplicationInsights/DataContracts/DependencyTelemetry.cs).

## <a name="incoming-operations-tracking"></a>Отслеживание входящих операций 
Пакет SDK Application Insights автоматически собирает HTTP-запросы для приложений ASP.NET, которые выполняются в конвейере служб IIS, и всех приложений ASP.NET Core. Существуют решения для других платформ, поддержка которых осуществляется силами сообщества разработчиков. Однако если приложение не поддерживается ни одним стандартным решением или решением сообщества разработчиков, его можно инструментировать вручную.

Другой пример, в котором требуется настраиваемое отслеживание, — это рабочая роль, которая получает элементы из очереди. Для некоторых очередей вызов для добавления сообщения в эту очередь отслеживается как зависимость. Однако операции высокого уровня, описывающие обработку сообщения, не собираются автоматически.

Посмотрим, как можно отследить эти операции.

На высоком уровне задачей является создание `RequestTelemetry` и настройка известных свойств. После завершения этой операции можно отслеживать данные телеметрии. Эта задача показана в следующем примере.

### <a name="http-request-in-owin-self-hosted-app"></a>HTTP-запрос в резидентном приложении Owin
В этом примере контекст трассировки распространяется согласно [протоколу HTTP для корреляции](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/HttpCorrelationProtocol.md). Следует ожидать заголовки, которые в нем описаны.

```csharp
public class ApplicationInsightsMiddleware : OwinMiddleware
{
    // you may create a new TelemetryConfiguration instance, reuse one you already have
    // or fetch the instance created by Application Insights SDK.
    private readonly TelemetryConfiguration telemetryConfiguration = TelemetryConfiguration.CreateDefault();
    private readonly TelemetryClient telemetryClient = new TelemetryClient(telemetryConfiguration);
    
    public ApplicationInsightsMiddleware(OwinMiddleware next) : base(next) {}

    public override async Task Invoke(IOwinContext context)
    {
        // Let's create and start RequestTelemetry.
        var requestTelemetry = new RequestTelemetry
        {
            Name = $"{context.Request.Method} {context.Request.Uri.GetLeftPart(UriPartial.Path)}"
        };

        // If there is a Request-Id received from the upstream service, set the telemetry context accordingly.
        if (context.Request.Headers.ContainsKey("Request-Id"))
        {
            var requestId = context.Request.Headers.Get("Request-Id");
            // Get the operation ID from the Request-Id (if you follow the HTTP Protocol for Correlation).
            requestTelemetry.Context.Operation.Id = GetOperationId(requestId);
            requestTelemetry.Context.Operation.ParentId = requestId;
        }

        // StartOperation is a helper method that allows correlation of 
        // current operations with nested operations/telemetry
        // and initializes start time and duration on telemetry items.
        var operation = telemetryClient.StartOperation(requestTelemetry);

        // Process the request.
        try
        {
            await Next.Invoke(context);
        }
        catch (Exception e)
        {
            requestTelemetry.Success = false;
            telemetryClient.TrackException(e);
            throw;
        }
        finally
        {
            // Update status code and success as appropriate.
            if (context.Response != null)
            {
                requestTelemetry.ResponseCode = context.Response.StatusCode.ToString();
                requestTelemetry.Success = context.Response.StatusCode >= 200 && context.Response.StatusCode <= 299;
            }
            else
            {
                requestTelemetry.Success = false;
            }

            // Now it's time to stop the operation (and track telemetry).
            telemetryClient.StopOperation(operation);
        }
    }
    
    public static string GetOperationId(string id)
    {
        // Returns the root ID from the '|' to the first '.' if any.
        int rootEnd = id.IndexOf('.');
        if (rootEnd < 0)
            rootEnd = id.Length;

        int rootStart = id[0] == '|' ? 1 : 0;
        return id.Substring(rootStart, rootEnd - rootStart);
    }
}
```

Протокол HTTP для корреляции также объявляет заголовок `Correlation-Context`. Однако здесь он опускается для простоты.

## <a name="queue-instrumentation"></a>Инструментирование очереди
Хотя существует [контекст трассировки W3C](https://www.w3.org/TR/trace-context/) и [протокол HTTP для корреляции](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/HttpCorrelationProtocol.md) для передачи сведений о корреляции с HTTP-запросом, каждый протокол очереди должен определять, как передаются одни и те же сведения в сообщении очереди. Некоторые протоколы очереди (например, AMQP) разрешают передавать дополнительные метаданные. Другим протоколам (например, очереди службы хранилища Azure) требуется контекст для кодирования в полезных данных сообщения.

> [!NOTE]
> * **Трассировка между компонентами еще не поддерживается для очередей** При использовании протокола HTTP, если производитель и потребитель отправляют данные телеметрии в разные Application Insights ресурсы, процесс диагностики транзакций и схема приложений показывают транзакции и сквозную карту. В случае очередей это еще не поддерживается. 

### <a name="service-bus-queue"></a>Очередь служебной шины
Application Insights отслеживает вызовы обмена сообщениями в службе "Служебная шина" с новым [клиентом службы "Служебная шина Microsoft Azure" для .NET](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus/) версии 3.0.0 и выше.
Если вы используете [шаблон обработчика сообщений](/dotnet/api/microsoft.azure.servicebus.queueclient.registermessagehandler) для обработки, больше ничего делать не нужно: все вызовы служебной шины, выполненные вашей службой, автоматически отслеживаются и связываются с другими элементами телеметрии. Если вы обрабатываете сообщения вручную, см. статью о [трассировке клиента служебной шины с помощью Microsoft Application Insights](../../service-bus-messaging/service-bus-end-to-end-tracing.md).

Если вы используете пакет [WindowsAzure.ServiceBus](https://www.nuget.org/packages/WindowsAzure.ServiceBus/), продолжайте читать эту статью. Следующие примеры демонстрируют, как отслеживать (и коррелировать) вызовы в служебную шину, если очередь служебной шины использует протокол AMQP и Application Insights не отслеживает операции очереди автоматически.
Идентификаторы корреляции передаются в свойствах сообщения.

#### <a name="enqueue"></a>Постановка в очередь

```csharp
public async Task Enqueue(string payload)
{
    // StartOperation is a helper method that initializes the telemetry item
    // and allows correlation of this operation with its parent and children.
    var operation = telemetryClient.StartOperation<DependencyTelemetry>("enqueue " + queueName);
    
    operation.Telemetry.Type = "Azure Service Bus";
    operation.Telemetry.Data = "Enqueue " + queueName;

    var message = new BrokeredMessage(payload);
    // Service Bus queue allows the property bag to pass along with the message.
    // We will use them to pass our correlation identifiers (and other context)
    // to the consumer.
    message.Properties.Add("ParentId", operation.Telemetry.Id);
    message.Properties.Add("RootId", operation.Telemetry.Context.Operation.Id);

    try
    {
        await queue.SendAsync(message);
        
        // Set operation.Telemetry Success and ResponseCode here.
        operation.Telemetry.Success = true;
    }
    catch (Exception e)
    {
        telemetryClient.TrackException(e);
        // Set operation.Telemetry Success and ResponseCode here.
        operation.Telemetry.Success = false;
        throw;
    }
    finally
    {
        telemetryClient.StopOperation(operation);
    }
}
```

#### <a name="process"></a>Процесс
```csharp
public async Task Process(BrokeredMessage message)
{
    // After the message is taken from the queue, create RequestTelemetry to track its processing.
    // It might also make sense to get the name from the message.
    RequestTelemetry requestTelemetry = new RequestTelemetry { Name = "process " + queueName };

    var rootId = message.Properties["RootId"].ToString();
    var parentId = message.Properties["ParentId"].ToString();
    // Get the operation ID from the Request-Id (if you follow the HTTP Protocol for Correlation).
    requestTelemetry.Context.Operation.Id = rootId;
    requestTelemetry.Context.Operation.ParentId = parentId;

    var operation = telemetryClient.StartOperation(requestTelemetry);

    try
    {
        await ProcessMessage();
    }
    catch (Exception e)
    {
        telemetryClient.TrackException(e);
        throw;
    }
    finally
    {
        // Update status code and success as appropriate.
        telemetryClient.StopOperation(operation);
    }
}
```

### <a name="azure-storage-queue"></a>Очередь службы хранилища Azure
В следующем примере показано, как отслеживать операции [очереди службы хранилища Azure](../../storage/queues/storage-dotnet-how-to-use-queues.md) и сопоставлять данные телеметрии производителя, потребителя и службы хранилища Azure. 

Очередь службы хранилища использует API HTTP. Все вызовы, отправляемые к очереди, отслеживаются сборщиком зависимостей Application Insights на наличие HTTP-запросов.
Он по умолчанию настроен для ASP.NET и ASP.NET Core приложений с другими типами приложений, которые можно найти в [документации по консольным приложениям](./console.md) .

Кроме того, может потребоваться сопоставить идентификатор операции Application Insights с идентификатором запроса службы хранилища. Сведения о том, как настроить и получить идентификатор клиента для запроса службы хранилища и идентификатор запроса сервера, см. в разделе [Мониторинг, диагностика и устранение неполадок с хранилищем Azure](../../storage/common/storage-monitoring-diagnosing-troubleshooting.md#end-to-end-tracing).

#### <a name="enqueue"></a>Постановка в очередь
Так как очереди службы хранилища Azure поддерживают API HTTP, все операции с очередью автоматически отслеживаются Application Insights. Во многих случаях этого инструментария должно быть достаточно. Однако для сопоставления трассировок на стороне потребителя с трассировками производителя необходимо передать некоторый контекст корреляции, точно так же, как мы сделали это для протокола HTTP для корреляции. 

В этом примере показано, как отслеживать операцию `Enqueue`. Вы можете:

 - **Сопоставить повторные попытки (если таковые имеются)**: все они имеют одну общую родительскую операцию, `Enqueue`. В противном случае они отслеживаются как дочерние элементы входящего запроса. В случае нескольких логических запросов к очереди может оказаться затруднительным определить, какой вызов совершался повторно.
 - **Сопоставить журналы службы хранилища Azure (при необходимости)**: они сопоставляются с данными телеметрии Application Insights.

Операция `Enqueue` является дочерним элементом родительской операции (например, входящего HTTP-запроса). Вызовов зависимости HTTP является дочерним элементом операции `Enqueue` и внучатым элементом входящего запроса.

```csharp
public async Task Enqueue(CloudQueue queue, string message)
{
    var operation = telemetryClient.StartOperation<DependencyTelemetry>("enqueue " + queue.Name);
    operation.Telemetry.Type = "Azure queue";
    operation.Telemetry.Data = "Enqueue " + queue.Name;

    // MessagePayload represents your custom message and also serializes correlation identifiers into payload.
    // For example, if you choose to pass payload serialized to JSON, it might look like
    // {'RootId' : 'some-id', 'ParentId' : '|some-id.1.2.3.', 'message' : 'your message to process'}
    var jsonPayload = JsonConvert.SerializeObject(new MessagePayload
    {
        RootId = operation.Telemetry.Context.Operation.Id,
        ParentId = operation.Telemetry.Id,
        Payload = message
    });
    
    CloudQueueMessage queueMessage = new CloudQueueMessage(jsonPayload);

    // Add operation.Telemetry.Id to the OperationContext to correlate Storage logs and Application Insights telemetry.
    OperationContext context = new OperationContext { ClientRequestID = operation.Telemetry.Id};

    try
    {
        await queue.AddMessageAsync(queueMessage, null, null, new QueueRequestOptions(), context);
    }
    catch (StorageException e)
    {
        operation.Telemetry.Properties.Add("AzureServiceRequestID", e.RequestInformation.ServiceRequestID);
        operation.Telemetry.Success = false;
        operation.Telemetry.ResultCode = e.RequestInformation.HttpStatusCode.ToString();
        telemetryClient.TrackException(e);
    }
    finally
    {
        // Update status code and success as appropriate.
        telemetryClient.StopOperation(operation);
    }
}  
```

Чтобы сократить объем данных телеметрии, передаваемой приложением, или если не требуется отслеживать операцию `Enqueue` по иным причинам, можно напрямую использовать API `Activity`.

- Создайте (и запустите) новый элемент `Activity` вместо запуска операции Application Insights. *Не* нужно назначать в нем какие-либо свойства, за исключением имени операции.
- Сериализируйте `yourActivity.Id` в полезные данные сообщения вместо `operation.Telemetry.Id`. Также можно использовать `Activity.Current.Id`.


#### <a name="dequeue"></a>Выведение из очереди
Так же, как и `Enqueue`, фактические HTTP-запросы к очереди службы хранилища Azure автоматически отслеживаются Application Insights. Однако операция `Enqueue` предположительно выполняется в контексте родительского элемента, такого как контекст входящего запроса. Пакеты SDK Application Insights автоматически сопоставляют такие операции (и их части HTTP) с родительским запросом и другими данными телеметрии, переданными в той же области.

С операцией `Dequeue` все не так просто. Пакет SDK Application Insights автоматически отслеживает HTTP-запросы. Тем не менее ему не известен контекст корреляции, пока выполняется анализ сообщения. Невозможно сопоставить HTTP-запрос для получения сообщения с остальными данными телеметрии, особенно если получено более одного сообщения.

```csharp
public async Task<MessagePayload> Dequeue(CloudQueue queue)
{
    var operation = telemetryClient.StartOperation<DependencyTelemetry>("dequeue " + queue.Name);
    operation.Telemetry.Type = "Azure queue";
    operation.Telemetry.Data = "Dequeue " + queue.Name;
    
    try
    {
        var message = await queue.GetMessageAsync();
    }
    catch (StorageException e)
    {
        operation.telemetry.Properties.Add("AzureServiceRequestID", e.RequestInformation.ServiceRequestID);
        operation.telemetry.Success = false;
        operation.telemetry.ResultCode = e.RequestInformation.HttpStatusCode.ToString();
        telemetryClient.TrackException(e);
    }
    finally
    {
        // Update status code and success as appropriate.
        telemetryClient.StopOperation(operation);
    }

    return null;
}
```

#### <a name="process"></a>Процесс

В следующем примере мы отслеживаем входящее сообщение так же, как выполняем трассировку входящего HTTP-запроса.

```csharp
public async Task Process(MessagePayload message)
{
    // After the message is dequeued from the queue, create RequestTelemetry to track its processing.
    RequestTelemetry requestTelemetry = new RequestTelemetry { Name = "process " + queueName };
    
    // It might also make sense to get the name from the message.
    requestTelemetry.Context.Operation.Id = message.RootId;
    requestTelemetry.Context.Operation.ParentId = message.ParentId;

    var operation = telemetryClient.StartOperation(requestTelemetry);

    try
    {
        await ProcessMessage();
    }
    catch (Exception e)
    {
        telemetryClient.TrackException(e);
        throw;
    }
    finally
    {
        // Update status code and success as appropriate.
        telemetryClient.StopOperation(operation);
    }
}
```

Другие операции очереди также можно инструментировать. Операцию заглядывания следует инструментировать так же, как операцию выведения из очереди. Инструментировать операции управления очередью необязательно. Application Insights отслеживает такие операции, как HTTP, и в большинстве случаев этого достаточно.

При инструментировании удаления сообщения убедитесь в том, что заданы идентификаторы операции (корреляция). Кроме того, можно использовать API `Activity`. Вам не нужно задавать идентификаторы операций для элементов телеметрии, так как пакет SDK для Application Insights делает это автоматически.

- Получив элемент из очереди, создайте `Activity`.
- Используйте `Activity.SetParentId(message.ParentId)` для сопоставления журналов потребителя и производителя.
- Запустите `Activity`.
- Отслеживайте операции выведения из очереди, обработки и удаления с помощью вспомогательных методов `Start/StopOperation`. Это необходимо выполнить из одного и того же асинхронного потока управления (контекста выполнения). Таким образом они будут сопоставлены должным образом.
- Остановите `Activity`.
- Используйте `Start/StopOperation` или вручную вызовите телеметрию `Track`.

### <a name="dependency-types"></a>Типы зависимостей

Application Insights использует тип зависимости для настройки интерфейсов пользовательского интерфейса. Для очередей он распознает следующие типы `DependencyTelemetry` , улучшающие [возможности диагностики транзакций](./transaction-diagnostics.md):
- `Azure queue` для очередей службы хранилища Azure
- `Azure Event Hubs` для концентраторов событий Azure
- `Azure Service Bus` для служебной шины Azure

### <a name="batch-processing"></a>Пакетная обработка
Некоторые очереди поддерживают вывод из очереди нескольких сообщений с помощью одного запроса. Обработка таких сообщений предположительно является независимой и относится к разным логическим операциям. Невозможно сопоставить `Dequeue` операцию с определенным сообщением, которое обрабатывается.

Обработка каждого сообщения должна выполняться в собственном асинхронном потоке управления. Дополнительные сведения см. в разделе [Отслеживание исходящих зависимостей](#outgoing-dependencies-tracking).

## <a name="long-running-background-tasks"></a>Длительные фоновые задачи

Некоторые приложения запускают длительные операции, которые могут быть вызваны запросами пользователей. С точки зрения трассировки и инструментирования это ничем не отличается от инструментирования запроса или зависимости. 

```csharp
async Task BackgroundTask()
{
    var operation = telemetryClient.StartOperation<DependencyTelemetry>(taskName);
    operation.Telemetry.Type = "Background";
    try
    {
        int progress = 0;
        while (progress < 100)
        {
            // Process the task.
            telemetryClient.TrackTrace($"done {progress++}%");
        }
        // Update status code and success as appropriate.
    }
    catch (Exception e)
    {
        telemetryClient.TrackException(e);
        // Update status code and success as appropriate.
        throw;
    }
    finally
    {
        telemetryClient.StopOperation(operation);
    }
}
```

В этом примере `telemetryClient.StartOperation` создает `DependencyTelemetry` и заполняет контекст корреляции. Предположим, имеется родительская операция, которая была создана входящими запросами, запланировавшими эту операцию. Так как `BackgroundTask` запускается в том же асинхронном потоке управления, что и входящий запрос, он сопоставляется с этой родительской операцией. `BackgroundTask` и все вложенные элементы телеметрии автоматически сопоставляются с запросом, вызвавшим ее, даже после завершения запроса.

Если задача запущена из фонового потока, с которым не связана ни одна операция (`Activity`), у задачи `BackgroundTask` отсутствуют какие-либо родительские элементы. Тем не менее у нее могут быть вложенные операции. Все элементы телеметрии, полученные от этой задачи, сопоставляются с элементом `DependencyTelemetry`, созданным в `BackgroundTask`.

## <a name="outgoing-dependencies-tracking"></a>Отслеживание исходящих зависимостей
Можно отслеживать собственный тип зависимостей или операцию, не поддерживаемую Application Insights.

Примером такого настраиваемого отслеживания может служить метод `Enqueue` в очереди служебной шины или очереди службы хранилища Azure.

Общий подход к настраиваемому отслеживанию зависимостей заключается в следующем.

- Вызовите метод `TelemetryClient.StartOperation` (расширение), который заполняет свойства `DependencyTelemetry`, необходимые для корреляции, и ряд других свойств (метка времени запуска, длительность).
- Задайте другие настраиваемые свойства `DependencyTelemetry`, например имя и любой контекст, который требуется.
- Создайте вызов зависимости и дождитесь его выполнения.
- Остановите операцию после завершения с помощью `StopOperation`.
- Выполните обработку исключений.

```csharp
public async Task RunMyTaskAsync()
{
    using (var operation = telemetryClient.StartOperation<DependencyTelemetry>("task 1"))
    {
        try 
        {
            var myTask = await StartMyTaskAsync();
            // Update status code and success as appropriate.
        }
        catch(...) 
        {
            // Update status code and success as appropriate.
        }
    }
}
```

Если операция удаляется, обработка останавливается. Поэтому вы можете удалить операцию вместо того, чтобы вызывать метод `StopOperation`.

*Предупреждение.* В некоторых случаях необработанное исключение может [помешать вызову](/dotnet/csharp/language-reference/keywords/try-finally) `finally`. Поэтому операции могут не отслеживаться.

### <a name="parallel-operations-processing-and-tracking"></a>Обработка и отслеживание параллельных операций

`StopOperation` останавливает только операцию, которая была запущена. Если текущая выполняемая операция не совпадает с операцией, которую нужно остановить, то `StopOperation` не выполняет никаких действий. Это может произойти в случае, если в одном и том же контексте выполнения параллельно запущено несколько операций.

```csharp
var firstOperation = telemetryClient.StartOperation<DependencyTelemetry>("task 1");
var firstTask = RunMyTaskAsync();

var secondOperation = telemetryClient.StartOperation<DependencyTelemetry>("task 2");
var secondTask = RunMyTaskAsync();

await firstTask;

// FAILURE!!! This will do nothing and will not report telemetry for the first operation
// as currently secondOperation is active.
telemetryClient.StopOperation(firstOperation); 

await secondTask;
```

Всегда вызывайте метод `StartOperation` и обрабатывайте операцию в том же методе **async**, чтобы изолировать параллельные операции. Если операция является синхронной (или не асинхронной), создайте оболочку для процесса и отслеживайте его с помощью команды `Task.Run`:

```csharp
public void RunMyTask(string name)
{
    using (var operation = telemetryClient.StartOperation<DependencyTelemetry>(name))
    {
        Process();
        // Update status code and success as appropriate.
    }
}

public async Task RunAllTasks()
{
    var task1 = Task.Run(() => RunMyTask("task 1"));
    var task2 = Task.Run(() => RunMyTask("task 2"));
    
    await Task.WhenAll(task1, task2);
}
```

## <a name="applicationinsights-operations-vs-systemdiagnosticsactivity"></a>Операции ApplicationInsights и System. Diagnostics. Activity
`System.Diagnostics.Activity` представляет распределенный контекст трассировки и используется платформами и библиотеками для создания и распространения контекста внутри и вне процесса и корреляции элементов телеметрии. Действие совместно с `System.Diagnostics.DiagnosticSource` — механизм уведомления между платформой или библиотекой для уведомления о интересных событиях (входящие или исходящие запросы, исключения и т. д.).

Действия являются привилегированными гражданами в Application Insights и автоматическая зависимость и сбор запросов в значительной степени полагаются на них вместе с `DiagnosticSource` событиями. При создании действия в приложении это не приведет к созданию Application Insights телеметрии. Application Insights необходимо получить события DiagnosticSource и выяснить имена и полезные данные событий для преобразования действия в телеметрию.

Каждая операция Application Insights (запрос или зависимость) включает `Activity` в себя — когда `StartOperation` вызывается, она создает действие под. `StartOperation` является рекомендуемым способом отслеживать телеметрии запроса или зависимости вручную и гарантировать, что все согласовано.

## <a name="next-steps"></a>Дальнейшие действия

- Изучите основы [корреляции данных телеметрии](correlation.md) в Application Insights.
- Узнайте, как коррелированные данные заключаются в [диагностике транзакций](./transaction-diagnostics.md) и [схеме приложений](./app-map.md).
- В [этой статье](./data-model.md) представлены типы данных и модель данных для Application Insights.
- Передача настраиваемых [событий и метрик](./api-custom-events-metrics.md) в Application Insights.
- Изучите стандартную [конфигурацию](configuration-with-applicationinsights-config.md#telemetry-initializers-aspnet) коллекции свойств контекста.
- Ознакомьтесь с [руководством пользователя по System.Diagnostics.Activity](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/ActivityUserGuide.md), чтобы узнать, как осуществляется корреляция данных телеметрии.

