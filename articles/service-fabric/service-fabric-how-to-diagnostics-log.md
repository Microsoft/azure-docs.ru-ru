---
title: Создание событий журнала из приложения .NET
description: Информация о том, как добавить функцию ведения в приложение .NET Service Fabric, размещенное в Azure или в изолированном кластере.
author: srrengar
ms.topic: conceptual
ms.date: 03/27/2018
ms.author: srrengar
ms.custom: devx-track-csharp
ms.openlocfilehash: a36425acf42a469c7f48b2e954bdacfdfcce1b10
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89011486"
---
# <a name="add-logging-to-your-service-fabric-application"></a>Добавление ведения журнала в приложение Service Fabric

Приложения должно содержать достаточно информации для экспертной отладки при возникновении проблем. Ведение журнала является одной из самых важных дополнительных функций в приложении Service Fabric. Если произойдет сбой, хороший журнал позволит тщательно его расследовать. Анализируя тенденции по записям журнала, вы можете найти пути к улучшению производительности или структуры приложения. Этот документ демонстрирует несколько методов ведения журнала.

## <a name="eventflow"></a>EventFlow

Пакет [библиотек EventFlow](https://github.com/Azure/diagnostics-eventflow) позволяет приложениям определять данные для сбора диагностики и целевые объекты для их вывода. Данные диагностики могут быть любыми, от счетчиков производительности до трассировок действий в приложении. Эти действия выполняются в том же процессе, что и само приложение, что позволяет снизить издержки на взаимодействие. Дополнительные сведения об использовании EventFlow и Service Fabric см. в статье [Агрегирование и сбор событий с помощью EventFlow](service-fabric-diagnostics-event-aggregation-eventflow.md).

### <a name="using-structured-eventsource-events"></a>Использование структурированных событий EventSource

Определив события сообщений для разных вариантов использования, вы сможете собирать пакеты данных о событии в контексте этого события. Имена и значения свойств выбранного события упрощают поиск и фильтрацию данных. Структурированный вывод средств инструментирования упрощает их чтение, но при таком подходе определение событий для каждого варианта использования требует значительных усилий и времени. 

Некоторые определения событий могут быть общими для всего приложения. Например, запуск или остановка метода применяются многократно во многих службах в рамках приложения. Специализированная служба, например система обработки заказов, может использовать событие **CreateOrder**, включающее собственное уникальное событие. Такой подход будет порождать множество событий и может потребовать согласования идентификаторов между командами проектов. 

```csharp
[EventSource(Name = "MyCompany-VotingState-VotingStateService")]
internal sealed class ServiceEventSource : EventSource
{
    public static readonly ServiceEventSource Current = new ServiceEventSource();

    // The instance constructor is private to enforce singleton semantics.
    private ServiceEventSource() : base() { }

    ...

    // The ServiceTypeRegistered event contains a unique identifier, an event attribute that defined the event, and the code implementation of the event.
    private const int ServiceTypeRegisteredEventId = 3;
    [Event(ServiceTypeRegisteredEventId, Level = EventLevel.Informational, Message = "Service host process {0} registered service type {1}", Keywords = Keywords.ServiceInitialization)]
    public void ServiceTypeRegistered(int hostProcessId, string serviceType)
    {
        WriteEvent(ServiceTypeRegisteredEventId, hostProcessId, serviceType);
    }

    // The ServiceHostInitializationFailed event contains a unique identifier, an event attribute that defined the event, and the code implementation of the event.
    private const int ServiceHostInitializationFailedEventId = 4;
    [Event(ServiceHostInitializationFailedEventId, Level = EventLevel.Error, Message = "Service host initialization failed", Keywords = Keywords.ServiceInitialization)]
    public void ServiceHostInitializationFailed(string exception)
    {
        WriteEvent(ServiceHostInitializationFailedEventId, exception);
    }

    ...

```

### <a name="using-eventsource-generically"></a>Универсальное использование EventSource

Так как определение конкретных событий может вызвать трудности, многие разработчики определяют несколько событий с общим набором параметров, информация о которых обычно выводится в строковом формате. При этом они теряют преимущества структурированности, что усложняет поиск и фильтрацию результатов. При таком подходе обычно определяется небольшое число событий, соответствующих уровням ведения журнала. В приведенном ниже фрагменте кода определяются отладочное сообщение и сообщение об ошибке.

```csharp
[EventSource(Name = "MyCompany-VotingState-VotingStateService")]
internal sealed class ServiceEventSource : EventSource
{
    public static readonly ServiceEventSource Current = new ServiceEventSource();

    // The Instance constructor is private, to enforce singleton semantics.
    private ServiceEventSource() : base() { }

    ...

    private const int DebugEventId = 10;
    [Event(DebugEventId, Level = EventLevel.Verbose, Message = "{0}")]
    public void Debug(string msg)
    {
        WriteEvent(DebugEventId, msg);
    }

    private const int ErrorEventId = 11;
    [Event(ErrorEventId, Level = EventLevel.Error, Message = "Error: {0} - {1}")]
    public void Error(string error, string msg)
    {
        WriteEvent(ErrorEventId, error, msg);
    }

    ...

```

Хорошие результаты может дать гибридный подход, сочетающий структурированное и универсальное инструментирование. Структурированное инструментирование используется для уведомления об ошибках и метриках. Универсальные события можно использовать для подробного ведения журнала, который используется инженерами для устранения неполадок.

## <a name="microsoftextensionslogging"></a>Microsoft.Extensions.Logging

Ведение журнала ASP.NET Core ([пакет Microsoft.Extensions.Logging NuGet](https://www.nuget.org/packages/Microsoft.Extensions.Logging)) — это платформа ведения журнала, которая предоставляет стандартный API для ведения журналов в приложениях. К системе ведения журнала ASP.NET Core можно подключить поддержку других серверных систем для ведения журнала. Этот метод дает вам широкий спектр ресурсов для ведения и обработки журналов в приложении, не требуя значительных изменений в коде.

1. Добавьте пакет NuGet **Microsoft. Extensions. Logging** в проект, который необходимо инструментировать. Кроме того, добавьте все пакеты поставщиков. Дополнительные сведения см. в разделе [Introduction to Logging in ASP.NET Core](/aspnet/core/fundamentals/logging) (Вводные сведения о ведении журналов в ASP.NET Core).
2. Добавьте директиву **using** для **Microsoft. Extensions. Logging** в файл службы.
3. Определите частную переменную в классе службы.

   ```csharp
   private ILogger _logger = null;
   ```

4. Добавьте в конструктор класса службы приведенный ниже код.

   ```csharp
   _logger = new LoggerFactory().CreateLogger<Stateless>();
   ```

5. Запустите инструментирование кода в методах. Вот несколько примеров.

   ```csharp
   _logger.LogDebug("Debug-level event from Microsoft.Logging");
   _logger.LogInformation("Informational-level event from Microsoft.Logging");

   // In this variant, we're adding structured properties RequestName and Duration, which have values MyRequest and the duration of the request.
   // Later in the article, we discuss why this step is useful.
   _logger.LogInformation("{RequestName} {Duration}", "MyRequest", requestDuration);
   ```

### <a name="using-other-logging-providers"></a>Использование других поставщиков ведения журнала

Некоторые сторонние поставщики, например [Serilog](https://serilog.net/), [NLog](https://nlog-project.org/) и [Loggr](https://github.com/imobile3/Loggr.Extensions.Logging), используют подход, описанный в предыдущем разделе. Вы можете подключить любой из них к подсистеме ведения журналов ASP.NET Core или использовать их отдельно. Serilog предоставляет возможность создавать дополнительные свойства для сообщений, отправляемых из средства ведения журнала. Эту функцию можно использовать для вывода информации об имени и типе службы, а также секционировании. Чтобы использовать эту возможность в инфраструктуре ASP.NET Core, сделайте следующее.

1. Добавьте в проект пакеты NuGet **Serilog**, **Serilog.Extensions.Logging**, **Serilog.Sinks.Literate** и **Serilog.Sinks.Observable**. 
2. Создайте `LoggerConfiguration` и экземпляр средства ведения журнала.

   ```csharp
   Log.Logger = new LoggerConfiguration().WriteTo.LiterateConsole().CreateLogger();
   ```

3. Добавьте аргумент `Serilog.ILogger` в конструктор службы и передайте в нем созданное средство ведения журнала.

   ```csharp
   ServiceRuntime.RegisterServiceAsync("StatelessType", context => new Stateless(context, Log.Logger)).GetAwaiter().GetResult();
   ```

4. Этот конструктор службы создает дополнительные свойства с именами **ServiceTypeName**, **ServiceName**, **PartitionId** и **InstanceId**.

   ```csharp
   public Stateless(StatelessServiceContext context, Serilog.ILogger serilog)
       : base(context)
   {
       PropertyEnricher[] properties = new PropertyEnricher[]
       {
           new PropertyEnricher("ServiceTypeName", context.ServiceTypeName),
           new PropertyEnricher("ServiceName", context.ServiceName),
           new PropertyEnricher("PartitionId", context.PartitionId),
           new PropertyEnricher("InstanceId", context.ReplicaOrInstanceId),
       };

       serilog.ForContext(properties);

       _logger = new LoggerFactory().AddSerilog(serilog.ForContext(properties)).CreateLogger<Stateless>();
   }
   ```

5. Инструментирование кода выполняется так же, как в ASP.NET Core без SeriLog.

   >[!NOTE]
   >Мы рекомендуем *не использовать* в предыдущем примере статический аргумент `Log.Logger`. Service Fabric позволяет разместить в одном процессе несколько экземпляров службы одного типа. Если вы примените статический аргумент `Log.Logger`, последний модуль записи дополнительных свойств будет содержать значения для всех запущенных экземпляров. Это одна из причин, по которой переменная _logger является частным членом класса службы. Кроме того, переменная `_logger` должна быть доступна в общем коде, который может использоваться несколькими службами.

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [приложении наблюдения за Service Fabric](service-fabric-diagnostics-event-generation-app.md).
- Изучите методы ведения журнала с использованием [EventFlow](service-fabric-diagnostics-event-aggregation-eventflow.md) и [Диагностики Azure для Windows](service-fabric-diagnostics-event-aggregation-wad.md).
