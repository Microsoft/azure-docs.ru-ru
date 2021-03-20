---
title: Комплексная трассировка и диагностика в служебной шине Azure | Документация Майкрософт
description: Общие сведения о диагностике клиентов служебной шины и сквозной трассировке (клиент с помощью всех служб, участвующих в обработке).
ms.topic: article
ms.date: 02/03/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: 19b284aceb83fbbc2bcf662b2b58941e6a5b36f9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99539219"
---
# <a name="distributed-tracing-and-correlation-through-service-bus-messaging"></a>Распределенная трассировка и корреляция путем обмена сообщениями через служебную шину

Одной из распространенных проблем при разработке микрослужб является возможность трассировки операций клиента с помощью всех служб, участвующих в обработке. Это удобно для выполнения отладки, анализа производительности, A/B-тестирования и других стандартных сценариев диагностики.
Одним из аспектов этой проблемы является отслеживание логических рабочих частей. Сюда входит результат обработки сообщений, задержка вызовов зависимостей и внешние вызовы зависимостей. Другой аспект — это корреляция этих диагностических событий, которые выходят за границы процессов.

Когда производитель отправляет сообщение через очередь, это обычно происходит в области какой-нибудь другой логической операции, инициированной другим клиентом или другой службой. При получении сообщения потребитель продолжает выполнять ту же операцию. Производитель и потребитель (и другие службы, обрабатывающие операции), предположительно генерируют события телеметрии для выполнения трассировки потока и результата операции. Чтобы сопоставить эти события и выполнить комплексную трассировку операции, каждая служба, отправляющая отчеты с данными телеметрии, должна помечать каждое событие с контекстом трассировки.

Служебная шина Microsoft Azure для обмена сообщениями имеет определенные свойства полезных данных, которые производитель и потребитель должны использовать для передачи этого контекста трассировки.
Протокол основан на [протоколе HTTP для корреляции](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/HttpCorrelationProtocol.md).

# <a name="azuremessagingservicebus-sdk-latest"></a>[Пакет SDK для Azure. Messaging. ServiceBus (последняя версия)](#tab/net-standard-sdk-2)
| Имя свойства        | Описание                                                 |
|----------------------|-------------------------------------------------------------|
|  Diagnostic-Id       | Уникальный идентификатор внешнего вызова от производителя к очереди. Обоснование, рекомендации и сведения о формате см. в разделе [Request-Id](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/HttpCorrelationProtocol.md#request-id) протокола HTTP. |

## <a name="service-bus-net-client-autotracing"></a>Автотрассировка клиента .NET служебной шины
`ServiceBusProcessor`Класс [клиента служебной шины Azure Messaging для .NET](/dotnet/api/azure.messaging.servicebus.servicebusprocessor) предоставляет точки инструментирования трассировки, которые могут быть подключены системами трассировки или частью клиентского кода. Инструментирование позволяет отслеживать все вызовы в службу обмена сообщениями служебной шины со стороны клиента. Если обработка сообщений выполняется с помощью [ `ProcessMessageAsync` метода `ServiceBusProcessor` ](/dotnet/api/azure.messaging.servicebus.servicebusprocessor.processmessageasync) (шаблон обработчика сообщений), то обработка сообщений также будет инструментирована.

### <a name="tracking-with-azure-application-insights"></a>Отслеживание с помощью Azure Application Insights

[Microsoft Application Insights](https://azure.microsoft.com/services/application-insights/) предоставляет широкие возможности мониторинга производительности, включая автоматическое создание запросов и отслеживание зависимостей.

В зависимости от типа проекта установите пакет SDK для Application Insights:
- [ASP.NET](../azure-monitor/app/asp-net.md): установите версию 2.5-beta2 или более позднюю.
- [ASP.NET Core](../azure-monitor/app/asp-net-core.md): установите версию 2.2.0-beta2 или более позднюю.
Эти ссылки содержат подробные сведения об установке пакета SDK, создании ресурсов и настройке пакета SDK (при необходимости). Сведения для приложений без ASP.NET см. в статье [Application Insights for .NET console applications](../azure-monitor/app/console.md) (Application Insights для консольных приложений .NET).

Если для обработки [ `ProcessMessageAsync` `ServiceBusProcessor` сообщений используется шаблон](/dotnet/api/azure.messaging.servicebus.servicebusprocessor.processmessageasync) обработчика сообщений, то обработка сообщений также будет инструментирована. Все вызовы служебной шины, выполненные службой, автоматически отправляются и сопоставляются с другими элементами телеметрии. В противном случае ознакомьтесь с указанным ниже примером для отслеживания обработки сообщений вручную.

#### <a name="trace-message-processing"></a>Обработка сообщений трассировки

```csharp
async Task ProcessAsync(ProcessMessageEventArgs args)
{
    ServiceBusReceivedMessage message = args.Message;
    if (message.ApplicationProperties.TryGetValue("Diagnostic-Id", out var objectId) && objectId is string diagnosticId)
    {
        var activity = new Activity("ServiceBusProcessor.ProcessMessage");
        activity.SetParentId(diagnosticId);
        // If you're using Microsoft.ApplicationInsights package version 2.6-beta or higher, you should call StartOperation<RequestTelemetry>(activity) instead
        using (var operation = telemetryClient.StartOperation<RequestTelemetry>("Process", activity.RootId, activity.ParentId))
        {
            telemetryClient.TrackTrace("Received message");
            try 
            {
            // process message
            }
            catch (Exception ex)
            {
                telemetryClient.TrackException(ex);
                operation.Telemetry.Success = false;
                throw;
            }

            telemetryClient.TrackTrace("Done");
        }
    }
}
```

В этом примере для каждого обработанного сообщения отображается данные телеметрии запроса с меткой времени, длительностью и результатом (успешно). Данные телеметрии также имеют набор свойств корреляции. Вложенные трассировки и исключения, обнаруженные во время обработки сообщения, также имеют свойства корреляции, представляя их в качестве дочерних элементов `RequestTelemetry`.

Если вы вызываете поддерживаемые внешние компоненты во время обработки сообщения, они также будут автоматически отслеживаться и сопоставляться. Дополнительные сведения о выполнении отслеживания и корреляции вручную см. в статье [Отслеживание пользовательских операций с помощью пакета SDK Application Insights для .NET](../azure-monitor/app/custom-operations-tracking.md).

Если вы используете внешний код в дополнение к пакету SDK для Application Insights, при просмотре Application Insights журналов необходимо больше **времени** . 

![Более длительная продолжительность в журнале Application Insights](./media/service-bus-end-to-end-tracing/longer-duration.png)

Это не значит, что во время получения сообщения произошла задержка. В этом сценарии сообщение уже было получено с момента передачи сообщения в качестве параметра в код пакета SDK. Кроме того, тег **Name** в журналах App Insights (**процесс**) указывает на то, что сообщение теперь обрабатывается кодом обработки внешнего события. Эта проблема не связана с Azure. Вместо этого эти метрики обращаются к эффективности внешнего кода, учитывая, что сообщение уже получено от служебной шины. 

### <a name="tracking-without-tracing-system"></a>Отслеживание без системы трассировки
Если система трассировки не поддерживает автоматическое отслеживание вызовов служебной шины, возможно, вы захотите добавить такую поддержку в систему трассировки или в приложение. В этом разделе описываются события диагностики, отправленные клиентом .NET служебной шины.  

Клиент .NET служебной шины инструментируется с помощью примитивов трассировки .NET [System.Diagnostics.Activity](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/ActivityUserGuide.md) и [System.Diagnostics.DiagnosticSource](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md).

`Activity` используется как контекст трассировки, в то время как `DiagnosticSource` — это механизм уведомлений. 

Если прослушиватель для событий DiagnosticSource не существует, инструментирование отключено, что не требует нулевых затрат на инструментирование. DiagnosticSource передает весь контроль прослушивателю:
- прослушиватель определяет, какие источники и события следует прослушивать;
- прослушиватель контролирует частоту событий и выборку;
- события отправляются вместе с полезными данными, которые предоставляют полный контекст, чтобы вы могли получить доступ к объекту сообщения во время события и изменить его.

Ознакомьтесь с [руководством пользователя DiagnosticSource](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md), прежде чем приступить к реализации.

Давайте создадим прослушиватель для событий служебной шины в приложении ASP.NET Core, который записывает журналы с помощью Microsoft.Extension.Logger.
Он использует библиотеку [System.Reactive.Core](https://www.nuget.org/packages/System.Reactive.Core), чтобы подписаться на DiagnosticSource (на DiagnosticSource можно также легко подписаться без прослушивателя).

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory factory, IApplicationLifetime applicationLifetime)
{
    // configuration...

    var serviceBusLogger = factory.CreateLogger("Azure.Messaging.ServiceBus");

    IDisposable innerSubscription = null;
    IDisposable outerSubscription = DiagnosticListener.AllListeners.Subscribe(delegate (DiagnosticListener listener)
    {
        // subscribe to the Service Bus DiagnosticSource
        if (listener.Name == "Azure.Messaging.ServiceBus")
        {
            // receive event from Service Bus DiagnosticSource
            innerSubscription = listener.Subscribe(delegate (KeyValuePair<string, object> evnt)
            {
                // Log operation details once it's done
                if (evnt.Key.EndsWith("Stop"))
                {
                    Activity currentActivity = Activity.Current;
                    serviceBusLogger.LogInformation($"Operation {currentActivity.OperationName} is finished, Duration={currentActivity.Duration}, Id={currentActivity.Id}, StartTime={currentActivity.StartTimeUtc}");
                }
            });
        }
    });

    applicationLifetime.ApplicationStopping.Register(() =>
    {
        outerSubscription?.Dispose();
        innerSubscription?.Dispose();
    });
}
```

В этом примере прослушиватель записывает длительность, результат, уникальный идентификатор и время начала для каждой операции служебной шины.

### <a name="events"></a>События
Все события будут иметь следующие свойства, соответствующие спецификации Open телеметрии: https://github.com/open-telemetry/opentelemetry-specification/blob/master/specification/trace/api.md .

- `message_bus.destination` — путь к очереди, разделу или подписке
- `peer.address` — полное пространство имен
- `kind` — Производитель, потребитель или клиент. Producer используется при отправке сообщений, потребителе при получении, а клиент при сопоставлении.
- `component` – `servicebus`

Все события также имеют свойства "Entity" и "Endpoint", они опущены в таблице ниже
  * `string Entity` — Имя сущности (очередь, тема и т. д.);
  * `Uri Endpoint` — URL-адрес конечной точки служебной шины.

### <a name="instrumented-operations"></a>Инструментированные операции
Ниже приведен полный список инструментированных операций.

| Имя операции | Отслеживаемый API |
| -------------- | ----------- | 
| Сервицебуссендер. Send | Сервицебуссендер. Сендмессажеасинк<br/>Сервицебуссендер. Сендмессажесасинк |
| Сервицебуссендер. Schedule | Сервицебуссендер. ScheduleMessageAsync<br/>Сервицебуссендер. Счедулемессажесасинк | 
| Сервицебуссендер. Cancel | Сервицебуссендер. Канцелсчедуледмессажеасинк<br/>Сервицебуссендер. Канцелсчедуледмессажесасинк |
| Сервицебусрецеивер. Receive | Сервицебусрецеивер. Рецеивемессажеасинк<br/>Сервицебусрецеивер. Рецеивемессажесасинк |
| Сервицебусрецеивер. Рецеиведеферред | Сервицебусрецеивер. Рецеиведеферредмессажесасинк |
| Сервицебусрецеивер. Peek | Сервицебусрецеивер. Пикмессажеасинк<br/>Сервицебусрецеивер. Пикмессажесасинк |
| Сервицебусрецеивер. | Сервицебусрецеивер. Абандонмессажесасинк |
| Сервицебусрецеивер. Complete | Сервицебусрецеивер. Комплетемессажесасинк |
| Сервицебусрецеивер. недоставленный. | Сервицебусрецеивер. Деадлеттермессажесасинк |
| Сервицебусрецеивер. Отсрочка |  Сервицебусрецеивер. Дефермессажесасинк |
| Сервицебусрецеивер. Реневмессажелокк | Сервицебусрецеивер. Реневмессажелоккасинк |
| Сервицебуссессионрецеивер. Реневсессионлокк | Сервицебуссессионрецеивер. Реневсессионлоккасинк |
| Сервицебуссессионрецеивер. Жетсессионстате | Сервицебуссессионрецеивер. Жетсессионстатеасинк |
| Сервицебуссессионрецеивер. Сетсессионстате | Сервицебуссессионрецеивер. Сетсессионстатеасинк |
| Сервицебуспроцессор. ProcessMessage | Обратный вызов процессора задан для Сервицебуспроцессор. Процессмессажеасинк, свойство |
| Сервицебуссессионпроцессор. Процесссессионмессаже | Обратный вызов процессора задан для Сервицебуссессионпроцессор. Процессмессажеасинк, свойство |

### <a name="filtering-and-sampling"></a>Фильтрация и выборка

В некоторых случаях предпочтительно зарегистрировать только часть событий, чтобы снизить потери производительности и уровень использования хранилища. Вы можете зарегистрировать только события Stop (как показано в предыдущем примере) или процент выборок событий. 
`DiagnosticSource` предоставляет способ реализации этих возможностей с помощью предиката `IsEnabled`. Дополнительные сведения см. в разделе [Фильтрация на основе контекста](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md#context-based-filtering).

Вы можете вызвать `IsEnabled` несколько раз для одной операции, чтобы минимизировать влияние на производительность.

`IsEnabled` вызывается в следующей последовательности:

1. `IsEnabled(<OperationName>, string entity, null)`, например, `IsEnabled("ServiceBusSender.Send", "MyQueue1")`. Обратите внимание, что в конце нет "Start" или "останавливаться". Используйте его, чтобы отфильтровать конкретные операции или очереди. Если метод обратного вызова возвращает `false` , события для операции не отправляются.

   * Для операций Process и ProcessSession вы также получите обратный вызов `IsEnabled(<OperationName>, string entity, Activity activity)`. С помощью него можно фильтровать события на основе `activity.Id` или свойства тегов.
  
2. `IsEnabled(<OperationName>.Start)`, например, `IsEnabled("ServiceBusSender.Send.Start")`. Проверяет, следует ли вызывать событие Start. Результат влияет только на событие Start, но дальнейшее инструментирование не зависит от него.

`IsEnabled`Для события "останавливаться" нет.

Если результатом некоторых операций является исключение, вызывается `IsEnabled("ServiceBusSender.Send.Exception")`. Вы можете только подписаться на события Exception и запретить остальное инструментирование. В этом случае вам по-прежнему следует обработать такие исключения. Так как другое инструментирование отключено, контекст трассировки не должен передаваться с сообщениями от потребителя к производителю.

Вы также можете использовать `IsEnabled`, который также реализует стратегии выборки. Выборка на основе `Activity.Id` или `Activity.RootId` обеспечивает согласованную выборку для всех значений (при условии, что она распространяется системой трассировки или собственным кодом).

При наличии нескольких `DiagnosticSource` прослушивателей для одного и того же источника достаточно одного прослушивателя, чтобы принять событие, поэтому нет никакой гарантии, что `IsEnabled` она вызывается.



# <a name="microsoftazureservicebus-sdk"></a>[Пакет SDK Microsoft. Azure. ServiceBus](#tab/net-standard-sdk)

| Имя свойства        | Описание                                                 |
|----------------------|-------------------------------------------------------------|
|  Diagnostic-Id       | Уникальный идентификатор внешнего вызова от производителя к очереди. Обоснование, рекомендации и сведения о формате см. в разделе [Request-Id](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/HttpCorrelationProtocol.md#request-id) протокола HTTP. |
|  Correlation-Context | Контекст операции, который распространяется на все службы, участвующие в обработке операции. Дополнительные сведения см. в разделе [Correlation-Context](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/HttpCorrelationProtocol.md#correlation-context) протокола HTTP. |

## <a name="service-bus-net-client-autotracing"></a>Автотрассировка клиента .NET служебной шины

Начиная с версии 3.0.0, [клиент служебной шины Microsoft Azure для .NET](/dotnet/api/microsoft.azure.servicebus.queueclient) предоставляет точки инструментария трассировки, которые можно обработать с помощью систем трассировки или фрагмента кода клиента.
Инструментирование позволяет отслеживать все вызовы в службу обмена сообщениями служебной шины со стороны клиента. Если обработка сообщений выполняется с помощью [шаблона обработчика сообщений](/dotnet/api/microsoft.azure.servicebus.queueclient.registermessagehandler), она также будет инструментирована.

### <a name="tracking-with-azure-application-insights"></a>Отслеживание с помощью Azure Application Insights

[Microsoft Application Insights](https://azure.microsoft.com/services/application-insights/) предоставляет широкие возможности мониторинга производительности, включая автоматическое создание запросов и отслеживание зависимостей.

В зависимости от типа проекта установите пакет SDK для Application Insights:
- [ASP.NET](../azure-monitor/app/asp-net.md): установите версию 2.5-beta2 или более позднюю.
- [ASP.NET Core](../azure-monitor/app/asp-net-core.md): установите версию 2.2.0-beta2 или более позднюю.
Эти ссылки содержат подробные сведения об установке пакета SDK, создании ресурсов и настройке пакета SDK (при необходимости). Сведения для приложений без ASP.NET см. в статье [Application Insights for .NET console applications](../azure-monitor/app/console.md) (Application Insights для консольных приложений .NET).

Если для обработки сообщений используется [шаблон обработчика сообщений](/dotnet/api/microsoft.azure.servicebus.queueclient.registermessagehandler) , то все вызовы служебной шины, выполненные службой, будут автоматически отслеживаться и сопоставляться с другими элементами телеметрии. В противном случае ознакомьтесь с указанным ниже примером для отслеживания обработки сообщений вручную.

#### <a name="trace-message-processing"></a>Обработка сообщений трассировки

```csharp
private readonly TelemetryClient telemetryClient;

async Task ProcessAsync(Message message)
{
    var activity = message.ExtractActivity();
    
    // If you're using Microsoft.ApplicationInsights package version 2.6-beta or higher, you should call StartOperation<RequestTelemetry>(activity) instead
    using (var operation = telemetryClient.StartOperation<RequestTelemetry>("Process", activity.RootId, activity.ParentId))
    {
        telemetryClient.TrackTrace("Received message");
        try 
        {
           // process message
        }
        catch (Exception ex)
        {
             telemetryClient.TrackException(ex);
             operation.Telemetry.Success = false;
             throw;
        }

        telemetryClient.TrackTrace("Done");
   }
}
```

В этом примере `RequestTelemetry` выводится для каждого обработанного сообщения вместе с меткой времени, продолжительностью и результатом (успешно). Данные телеметрии также имеют набор свойств корреляции.
Вложенные трассировки и исключения, обнаруженные во время обработки сообщения, также имеют свойства корреляции, представляя их в качестве дочерних элементов `RequestTelemetry`.

Если вы вызываете поддерживаемые внешние компоненты во время обработки сообщения, они также будут автоматически отслеживаться и сопоставляться. Дополнительные сведения о выполнении отслеживания и корреляции вручную см. в статье [Отслеживание пользовательских операций с помощью пакета SDK Application Insights для .NET](../azure-monitor/app/custom-operations-tracking.md).

Если вы используете внешний код в дополнение к пакету SDK для Application Insights, при просмотре Application Insights журналов необходимо больше **времени** . 

![Более длительная продолжительность в журнале Application Insights](./media/service-bus-end-to-end-tracing/longer-duration.png)

Это не значит, что во время получения сообщения произошла задержка. В этом сценарии сообщение уже было получено с момента передачи сообщения в качестве параметра в код пакета SDK. Кроме того, тег **Name** в журналах App Insights (**процесс**) указывает на то, что сообщение теперь обрабатывается кодом обработки внешнего события. Эта проблема не связана с Azure. Вместо этого эти метрики обращаются к эффективности внешнего кода, учитывая, что сообщение уже получено от служебной шины. См. [этот файл в GitHub](https://github.com/Azure/azure-sdk-for-net/blob/4bab05144ce647cc9e704d46d3763de5f9681ee0/sdk/servicebus/Microsoft.Azure.ServiceBus/src/ServiceBusDiagnosticsSource.cs) , чтобы узнать, где создан тег **Process** и назначен после получения сообщения от служебной шины. 

### <a name="tracking-without-tracing-system"></a>Отслеживание без системы трассировки
Если система трассировки не поддерживает автоматическое отслеживание вызовов служебной шины, возможно, вы захотите добавить такую поддержку в систему трассировки или в приложение. В этом разделе описываются события диагностики, отправленные клиентом .NET служебной шины.  

Клиент .NET служебной шины инструментируется с помощью примитивов трассировки .NET [System.Diagnostics.Activity](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/ActivityUserGuide.md) и [System.Diagnostics.DiagnosticSource](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md).

`Activity` используется как контекст трассировки, в то время как `DiagnosticSource` — это механизм уведомлений. 

Если прослушиватель для событий DiagnosticSource не существует, инструментирование отключено, что не требует нулевых затрат на инструментирование. DiagnosticSource передает весь контроль прослушивателю:
- прослушиватель определяет, какие источники и события следует прослушивать;
- прослушиватель контролирует частоту событий и выборку;
- события отправляются вместе с полезными данными, которые предоставляют полный контекст, чтобы вы могли получить доступ к объекту сообщения во время события и изменить его.

Ознакомьтесь с [руководством пользователя DiagnosticSource](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md), прежде чем приступить к реализации.

Давайте создадим прослушиватель для событий служебной шины в приложении ASP.NET Core, который записывает журналы с помощью Microsoft.Extension.Logger.
Он использует библиотеку [System.Reactive.Core](https://www.nuget.org/packages/System.Reactive.Core), чтобы подписаться на DiagnosticSource (на DiagnosticSource можно также легко подписаться без прослушивателя).

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory factory, IApplicationLifetime applicationLifetime)
{
    // configuration...

    var serviceBusLogger = factory.CreateLogger("Microsoft.Azure.ServiceBus");

    IDisposable innerSubscription = null;
    IDisposable outerSubscription = DiagnosticListener.AllListeners.Subscribe(delegate (DiagnosticListener listener)
    {
        // subscribe to the Service Bus DiagnosticSource
        if (listener.Name == "Microsoft.Azure.ServiceBus")
        {
            // receive event from Service Bus DiagnosticSource
            innerSubscription = listener.Subscribe(delegate (KeyValuePair<string, object> evnt)
            {
                // Log operation details once it's done
                if (evnt.Key.EndsWith("Stop"))
                {
                    Activity currentActivity = Activity.Current;
                    TaskStatus status = (TaskStatus)evnt.Value.GetProperty("Status");
                    serviceBusLogger.LogInformation($"Operation {currentActivity.OperationName} is finished, Duration={currentActivity.Duration}, Status={status}, Id={currentActivity.Id}, StartTime={currentActivity.StartTimeUtc}");
                }
            });
        }
    });

    applicationLifetime.ApplicationStopping.Register(() =>
    {
        outerSubscription?.Dispose();
        innerSubscription?.Dispose();
    });
}
```

В этом примере прослушиватель записывает длительность, результат, уникальный идентификатор и время начала для каждой операции служебной шины.

#### <a name="events"></a>События

Для каждой операции отправляются два события: Start и Stop. Скорее всего, вас интересуют только события "останавливаться". Они предоставляют результат операции, а также время и длительность запуска в качестве свойств действия.

Полезные данные события предоставляют событию контекст операции, они реплицируют входящие параметры API и возвращаемое значение. Полезные данные события Stop имеют все свойства полезных данных события Start. Таким образом, вы можете полностью игнорировать событие Start.

Все события также имеют свойства "Entity" и "Endpoint", они опущены в таблице ниже
  * `string Entity` — имя сущности (очереди, раздела и т. д.);
  * `Uri Endpoint` — URL-адрес конечной точки служебной шины.

Каждое событие Stop имеет свойство `Status` с асинхронной операцией `TaskStatus`, с помощью которой оно завершилось. Для простоты она также указана в таблице ниже.

Ниже приведен полный список инструментированных операций.

| Имя операции | Отслеживаемый API | Определенные свойства полезных данных|
|----------------|-------------|---------|
| Microsoft.Azure.ServiceBus.Send | [MessageSender.SendAsync](/dotnet/api/microsoft.azure.servicebus.core.messagesender.sendasync) | `IList<Message> Messages` — список отправляемых сообщений. |
| Microsoft.Azure.ServiceBus.ScheduleMessage | [MessageSender.ScheduleMessageAsync](/dotnet/api/microsoft.azure.servicebus.core.messagesender.schedulemessageasync) | `Message Message` — обрабатываемое сообщение.<br/>`DateTimeOffset ScheduleEnqueueTimeUtc` — смещение запланированных сообщений.<br/>`long SequenceNumber` — порядковый номер запланированного сообщения (полезные данные события Stop). |
| Microsoft.Azure.ServiceBus.Cancel | [MessageSender.CancelScheduledMessageAsync](/dotnet/api/microsoft.azure.servicebus.core.messagesender.cancelscheduledmessageasync) | `long SequenceNumber` — порядковый номер сообщения, которое будет отменено. | 
| Microsoft.Azure.ServiceBus.Receive | [MessageReceiver.ReceiveAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.receiveasync) | `int RequestedMessageCount` — максимальное количество сообщений, которые можно получить.<br/>`IList<Message> Messages` — список полученных сообщений (полезные данные события Stop). |
| Microsoft.Azure.ServiceBus.Peek | [MessageReceiver.PeekAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.peekasync) | `int FromSequenceNumber` — отправная точка, с которой нужно начинать просмотр сообщений.<br/>`int RequestedMessageCount` — число извлекаемых сообщений.<br/>`IList<Message> Messages` — список полученных сообщений (полезные данные события Stop). |
| Microsoft.Azure.ServiceBus.ReceiveDeferred | [MessageReceiver.ReceiveDeferredMessageAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.receivedeferredmessageasync) | `IEnumerable<long> SequenceNumbers` — список, содержащий порядковые номера для получения.<br/>`IList<Message> Messages` — список полученных сообщений (полезные данные события Stop). |
| Microsoft.Azure.ServiceBus.Complete | [MessageReceiver.CompleteAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.completeasync) | `IList<string> LockTokens` — список, содержащий маркеры блокировки соответствующих сообщений для выполнения.|
| Microsoft.Azure.ServiceBus.Abandon | [MessageReceiver.AbandonAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.abandonasync) | `string LockToken` — маркер блокировки соответствующего сообщения для отказа. |
| Microsoft.Azure.ServiceBus.Defer | [MessageReceiver.DeferAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.deferasync) | `string LockToken` — маркер блокировки соответствующего сообщения для откладывания. | 
| Microsoft.Azure.ServiceBus.DeadLetter | [MessageReceiver.DeadLetterAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.deadletterasync) | `string LockToken` — маркер блокировки соответствующего сообщения для недоставленных сообщений. | 
| Microsoft.Azure.ServiceBus.RenewLock | [MessageReceiver.RenewLockAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.renewlockasync) | `string LockToken` — маркер блокировки соответствующего сообщения для продления блокировки.<br/>`DateTime LockedUntilUtc` — новые дата и время истечения срока действия маркера блокировки в формате UTC. (Полезные данные события Stop)|
| Microsoft.Azure.ServiceBus.Process | Лямбда-функция обработчика сообщений, указанная в [IReceiverClient.RegisterMessageHandler](/dotnet/api/microsoft.azure.servicebus.core.ireceiverclient.registermessagehandler) | `Message Message` — обрабатываемое сообщение. |
| Microsoft.Azure.ServiceBus.ProcessSession | Лямбда-функция обработчика сеанса обмена сообщений, указанная в [IQueueClient.RegisterSessionHandler](/dotnet/api/microsoft.azure.servicebus.iqueueclient.registersessionhandler) | `Message Message` — обрабатываемое сообщение.<br/>`IMessageSession Session` — обрабатываемый сеанс. |
| Microsoft.Azure.ServiceBus.AddRule | [SubscriptionClient.AddRuleAsync](/dotnet/api/microsoft.azure.servicebus.subscriptionclient.addruleasync) | `RuleDescription Rule` — описание правила, которое определяет правило для добавления. |
| Microsoft.Azure.ServiceBus.RemoveRule | [SubscriptionClient.RemoveRuleAsync](/dotnet/api/microsoft.azure.servicebus.subscriptionclient.removeruleasync) | `string RuleName` — имя правила для удаления. |
| Microsoft.Azure.ServiceBus.GetRules | [SubscriptionClient.GetRulesAsync](/dotnet/api/microsoft.azure.servicebus.subscriptionclient.getrulesasync) | `IEnumerable<RuleDescription> Rules` — все правила, связанные с подпиской. (только полезные данные Stop) |
| Microsoft.Azure.ServiceBus.AcceptMessageSession | [ISessionClient.AcceptMessageSessionAsync](/dotnet/api/microsoft.azure.servicebus.isessionclient.acceptmessagesessionasync) | `string SessionId` — идентификатор сеанса, присутствующий в сообщениях. |
| Microsoft.Azure.ServiceBus.GetSessionState | [IMessageSession.GetStateAsync](/dotnet/api/microsoft.azure.servicebus.imessagesession.getstateasync) | `string SessionId` — идентификатор сеанса, присутствующий в сообщениях.<br/>`byte [] State` — состояние сеанса (полезные данные события Stop). |
| Microsoft.Azure.ServiceBus.SetSessionState | [IMessageSession.SetStateAsync](/dotnet/api/microsoft.azure.servicebus.imessagesession.setstateasync) | `string SessionId` — идентификатор сеанса, присутствующий в сообщениях.<br/>`byte [] State` — состояние сеанса. |
| Microsoft.Azure.ServiceBus.RenewSessionLock | [IMessageSession.RenewSessionLockAsync](/dotnet/api/microsoft.azure.servicebus.imessagesession.renewsessionlockasync) | `string SessionId` — идентификатор сеанса, присутствующий в сообщениях. |
| Microsoft.Azure.ServiceBus.Exception | любой инструментированный API| `Exception Exception` — экземпляр исключения. |

В каждом событии вы можете получить доступ к `Activity.Current`, где содержится контекст текущей операции.

#### <a name="logging-more-properties"></a>Ведение журнала дополнительных свойств

`Activity.Current` предоставляет подробные сведения о контексте текущей операции и ее родительских объектов. Дополнительные сведения см. в [документации по действиям](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/ActivityUserGuide.md).
Инструментарий служебной шины предоставляет дополнительные сведения, которые хранятся в, `Activity.Current.Tags` `MessageId` и `SessionId` каждый раз, когда они доступны.

Действия, отслеживающие события Receive, Peek и ReceiveDeferred, также могут содержать тег `RelatedTo`. Он содержит список уникальных значений `Diagnostic-Id` для сообщений, полученных в качестве результата.
Такая операция может привести к получению нескольких несвязанных сообщений. Кроме того, `Diagnostic-Id` при запуске операции неизвестен, поэтому операции Receive могут быть связаны с операциями "Process" только с этим тегом. Это удобно при анализе проблем с производительностью, чтобы иметь возможность проверить, сколько времени потребовалось для получения сообщения.

Эффективный способ зарегистрировать теги — выполнить их итерацию. Процесс добавления тегов к предыдущему примеру выглядит примерно так: 

```csharp
Activity currentActivity = Activity.Current;
TaskStatus status = (TaskStatus)evnt.Value.GetProperty("Status");

var tagsList = new StringBuilder();
foreach (var tags in currentActivity.Tags)
{
    tagsList.Append($", {tags.Key}={tags.Value}");
}

serviceBusLogger.LogInformation($"{currentActivity.OperationName} is finished, Duration={currentActivity.Duration}, Status={status}, Id={currentActivity.Id}, StartTime={currentActivity.StartTimeUtc}{tagsList}");
```

#### <a name="filtering-and-sampling"></a>Фильтрация и выборка

В некоторых случаях предпочтительно зарегистрировать только часть событий, чтобы снизить потери производительности и уровень использования хранилища. Вы можете зарегистрировать только события Stop (как показано в предыдущем примере) или процент выборок событий. 
`DiagnosticSource` предоставляет способ реализации этих возможностей с помощью предиката `IsEnabled`. Дополнительные сведения см. в разделе [Фильтрация на основе контекста](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md#context-based-filtering).

Вы можете вызвать `IsEnabled` несколько раз для одной операции, чтобы минимизировать влияние на производительность.

`IsEnabled` вызывается в следующей последовательности:

1. `IsEnabled(<OperationName>, string entity, null)`, например, `IsEnabled("Microsoft.Azure.ServiceBus.Send", "MyQueue1")`. Обратите внимание, что в конце нет "Start" или "останавливаться". Используйте его, чтобы отфильтровать конкретные операции или очереди. Если метод обратного вызова возвращает значение `false` , события для операции не отправляются

   * Для операций Process и ProcessSession вы также получите обратный вызов `IsEnabled(<OperationName>, string entity, Activity activity)`. С помощью него можно фильтровать события на основе `activity.Id` или свойства тегов.
  
2. `IsEnabled(<OperationName>.Start)`, например, `IsEnabled("Microsoft.Azure.ServiceBus.Send.Start")`. Проверяет, следует ли вызывать событие Start. Результат влияет только на событие Start, но дальнейшее инструментирование не зависит от него.

`IsEnabled`Для события "останавливаться" нет.

Если результатом некоторых операций является исключение, вызывается `IsEnabled("Microsoft.Azure.ServiceBus.Exception")`. Вы можете только подписаться на события Exception и запретить остальное инструментирование. В этом случае вам по-прежнему следует обработать такие исключения. Так как другое инструментирование отключено, контекст трассировки не должен передаваться с сообщениями от потребителя к производителю.

Вы также можете использовать `IsEnabled`, который также реализует стратегии выборки. Выборка на основе `Activity.Id` или `Activity.RootId` обеспечивает согласованную выборку для всех значений (при условии, что она распространяется системой трассировки или собственным кодом).

При наличии нескольких `DiagnosticSource` прослушивателей для одного и того же источника достаточно одного прослушивателя, чтобы принять событие, поэтому нет никакой гарантии, что `IsEnabled` она вызывается.

---

## <a name="next-steps"></a>Дальнейшие действия

* [Корреляция данных телеметрии в Application Insights](../azure-monitor/app/correlation.md)
* [Настройка Application Insights: отслеживание зависимостей](../azure-monitor/app/asp-net-dependencies.md), чтобы выяснить, что стало причиной медленной работы: REST, SQL или другие внешние ресурсы.
* [Отслеживание пользовательских операций с помощью пакета SDK Application Insights для .NET](../azure-monitor/app/custom-operations-tracking.md)

