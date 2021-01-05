---
title: Комплексная трассировка и диагностика в служебной шине Azure | Документация Майкрософт
description: Общие сведения о диагностике клиентов служебной шины и сквозной трассировке (клиент с помощью всех служб, участвующих в обработке).
ms.topic: article
ms.date: 06/23/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: bc7dab21fc01b624e8ab122fe883be89ea8633f6
ms.sourcegitcommit: 8be279f92d5c07a37adfe766dc40648c673d8aa8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/31/2020
ms.locfileid: "97832698"
---
# <a name="distributed-tracing-and-correlation-through-service-bus-messaging"></a>Распределенная трассировка и корреляция путем обмена сообщениями через служебную шину

Одна из распространенных проблем в разработке микрослужб — это возможность трассировки операции из клиента во всех службах, участвующих в обработке. Это удобно для выполнения отладки, анализа производительности, A/B-тестирования и других стандартных сценариев диагностики.
Одним из аспектов этой проблемы является отслеживание логических рабочих частей. Сюда входит результат обработки сообщений, задержка вызовов зависимостей и внешние вызовы зависимостей. Другой аспект — это корреляция этих диагностических событий, которые выходят за границы процессов.

Когда производитель отправляет сообщение через очередь, это обычно происходит в области какой-нибудь другой логической операции, инициированной другим клиентом или другой службой. При получении сообщения потребитель продолжает выполнять ту же операцию. Производитель и потребитель (и другие службы, обрабатывающие операции), предположительно генерируют события телеметрии для выполнения трассировки потока и результата операции. Чтобы сопоставить эти события и выполнить комплексную трассировку операции, каждая служба, отправляющая отчеты с данными телеметрии, должна помечать каждое событие с контекстом трассировки.

Служебная шина Microsoft Azure для обмена сообщениями имеет определенные свойства полезных данных, которые производитель и потребитель должны использовать для передачи этого контекста трассировки.
Протокол основан на [протоколе HTTP для корреляции](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/HttpCorrelationProtocol.md).

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

Если вы используете [шаблон обработчика сообщений](/dotnet/api/microsoft.azure.servicebus.queueclient.registermessagehandler) для обработки, больше ничего делать не нужно: все вызовы служебной шины, выполненные вашей службой, автоматически отслеживаются и связываются с другими элементами телеметрии. В противном случае ознакомьтесь с указанным ниже примером для отслеживания обработки сообщений вручную.

#### <a name="trace-message-processing"></a>Обработка сообщений трассировки

```csharp
private readonly TelemetryClient telemetryClient;

async Task ProcessAsync(Message message)
{
    var activity = message.ExtractActivity();
    
    // If you are using Microsoft.ApplicationInsights package version 2.6-beta or higher, you should call StartOperation<RequestTelemetry>(activity) instead
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

Если вы отправили вызовы к поддерживаемым внешним компонентам во время обработки сообщения, они также будут автоматически отслеживаться и коррелироваться. Дополнительные сведения о выполнении отслеживания и корреляции вручную см. в статье [Отслеживание пользовательских операций с помощью пакета SDK Application Insights для .NET](../azure-monitor/app/custom-operations-tracking.md).

Если вы используете внешний код в дополнение к пакету SDK для Application Insights, то при просмотре Application Insights журналов необходимо больше **времени** . 

![Более длительная продолжительность в журнале Application Insights](./media/service-bus-end-to-end-tracing/longer-duration.png)

Это не значит, что во время получения сообщения произошла задержка. В этом сценарии сообщение уже было получено с момента передачи сообщения в качестве параметра в код пакета SDK. Кроме того, тег **Name** в журналах App Insights (**процесс**) указывает на то, что сообщение теперь обрабатывается кодом обработки внешнего события. Эта проблема не связана с Azure. Вместо этого эти метрики обращаются к эффективности внешнего кода, учитывая, что сообщение уже получено от служебной шины. См. [этот файл в GitHub](https://github.com/Azure/azure-sdk-for-net/blob/4bab05144ce647cc9e704d46d3763de5f9681ee0/sdk/servicebus/Microsoft.Azure.ServiceBus/src/ServiceBusDiagnosticsSource.cs) , чтобы узнать, где создан тег **Process** и назначен после получения сообщения от служебной шины. 

### <a name="tracking-without-tracing-system"></a>Отслеживание без системы трассировки
Если ваша система трассировки не поддерживает автоматическое отслеживание вызовов служебной шины, вы можете рассмотреть возможность добавить эту поддержку в систему трассировки или приложение. В этом разделе описываются события диагностики, отправленные клиентом .NET служебной шины.  

Клиент .NET служебной шины инструментируется с помощью примитивов трассировки .NET [System.Diagnostics.Activity](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/ActivityUserGuide.md) и [System.Diagnostics.DiagnosticSource](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md).

`Activity` используется как контекст трассировки, в то время как `DiagnosticSource` — это механизм уведомлений. 

Если для событий DiagnosticSource нет прослушивателя, инструментирование будет отключено, чтобы за него не взималась плата. DiagnosticSource передает весь контроль прослушивателю:
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

Для каждой операции отправляются два события: Start и Stop. Скорее всего, вас интересуют только события Stop. Они предоставляют результат операции, а также время начала и длительность в качестве свойств действия.

Полезные данные события предоставляют событию контекст операции, они реплицируют входящие параметры API и возвращаемое значение. Полезные данные события Stop имеют все свойства полезных данных события Start. Таким образом, вы можете полностью игнорировать событие Start.

Все события также имеют свойства Entity и Endpoint. Они указаны в таблице ниже.
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

#### <a name="logging-additional-properties"></a>Ведение журналов дополнительных свойств

`Activity.Current` предоставляет подробные сведения о контексте текущей операции и ее родительских объектов. Дополнительные сведения см. в [документации по действию](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/ActivityUserGuide.md).
Инструментирование служебной шины предоставляет дополнительные сведения в `Activity.Current.Tags`. Они удерживают `MessageId` и `SessionId` каждый раз, когда они доступны.

Действия, отслеживающие события Receive, Peek и ReceiveDeferred, также могут содержать тег `RelatedTo`. Он содержит список уникальных значений `Diagnostic-Id` для сообщений, полученных в качестве результата.
Такая операция может привести к получению нескольких несвязанных сообщений. Кроме того, при запуске операции `Diagnostic-Id` неизвестен, поэтому операции Receive могут быть сопоставлены с операциями Process с использованием только этого тега. Это удобно при анализе проблем с производительностью, чтобы иметь возможность проверить, сколько времени потребовалось для получения сообщения.

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

1. `IsEnabled(<OperationName>, string entity, null)`, например, `IsEnabled("Microsoft.Azure.ServiceBus.Send", "MyQueue1")`. Обратите внимание, что в конце нет события Start или Stop. Используйте его, чтобы отфильтровать конкретные операции или очереди. Если обратный вызов возвращает `false`, события для операции не отправляются.

   * Для операций Process и ProcessSession вы также получите обратный вызов `IsEnabled(<OperationName>, string entity, Activity activity)`. С помощью него можно фильтровать события на основе `activity.Id` или свойства тегов.
  
2. `IsEnabled(<OperationName>.Start)`, например, `IsEnabled("Microsoft.Azure.ServiceBus.Send.Start")`. Проверяет, следует ли вызывать событие Start. Результат влияет только на событие Start, но дальнейшее инструментирование от него не зависит.

Для события Stop отсутствует `IsEnabled`.

Если результатом некоторых операций является исключение, вызывается `IsEnabled("Microsoft.Azure.ServiceBus.Exception")`. Вы можете только подписаться на события Exception и запретить остальное инструментирование. В этом случае вам по-прежнему следует обработать такие исключения. После отключения других операций инструментирования не следует ожидать, что контекст трассировки будет передаваться с сообщением от потребителя к производителю.

Вы также можете использовать `IsEnabled`, который также реализует стратегии выборки. Выборка на основе `Activity.Id` или `Activity.RootId` гарантирует согласованную выборку на всех ресурсах (при условии, что она распространяется системой трассировки или вашим кодом).

При наличии нескольких прослушивателей `DiagnosticSource` для одного источника будет достаточно, чтобы событие принял один из них, поэтому `IsEnabled` вызывать необязательно.

## <a name="next-steps"></a>Дальнейшие действия

* [Корреляция данных телеметрии в Application Insights](../azure-monitor/app/correlation.md)
* [Настройка Application Insights: отслеживание зависимостей](../azure-monitor/app/asp-net-dependencies.md), чтобы выяснить, что стало причиной медленной работы: REST, SQL или другие внешние ресурсы.
* [Отслеживание пользовательских операций с помощью пакета SDK Application Insights для .NET](../azure-monitor/app/custom-operations-tracking.md)
