---
title: Application Insights для приложений службы рабочей роли (приложений, отличных от HTTP)
description: Мониторинг приложений .NET Core/. NET Framework, отличных от HTTP, с Azure Monitor Application Insights.
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 05/11/2020
ms.openlocfilehash: c1ca594626d4384c9dfb62990ee2017d2094fca4
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100371861"
---
# <a name="application-insights-for-worker-service-applications-non-http-applications"></a>Application Insights для приложений службы рабочей роли (приложений, отличных от HTTP)

[Application Insights SDK для службы Worker](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WorkerService) — это новый пакет SDK, который лучше всего подходит для рабочих нагрузок, отличных от HTTP, таких как обмен сообщениями, фоновые задачи, консольные приложения и т. д Эти типы приложений не имеют представления о входящем HTTP-запросе, таком как традиционное приложение ASP.NET/ASP.NET Core, и поэтому использование пакетов Application Insights для [ASP.NET](asp-net.md) или [ASP.NET Core](asp-net-core.md) приложений не поддерживается.

Новый пакет SDK не выполняет сбор данных телеметрии самим собой. Вместо этого он переносит другие хорошо известные Application Insights автособирающие, такие как [DependencyCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DependencyCollector/), [перфкаунтерколлектор](https://www.nuget.org/packages/Microsoft.ApplicationInsights.PerfCounterCollector/), [аппликатионинсигхтслоггингпровидер](https://www.nuget.org/packages/Microsoft.Extensions.Logging.ApplicationInsights) и т. д. Этот пакет SDK предоставляет методы расширения `IServiceCollection` для включения и настройки сбора данных телеметрии.

## <a name="supported-scenarios"></a>Поддерживаемые сценарии

Пакет [SDK Application Insights для службы рабочей роли](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WorkerService) лучше всего подходит для приложений, отличных от HTTP, независимо от того, где или как они выполняются. Если приложение работает и имеет сетевое подключение к Azure, можно собирать данные телеметрии. Мониторинг Application Insights поддерживается везде, где поддерживается .NET Core. Этот пакет можно использовать в новой добавленной [службе рабочей роли .NET Core 3,0](https://devblogs.microsoft.com/aspnet/dotnet-core-workers-in-azure-container-instances), [фоновых задачах в ASP.NET Core 2.1/2.2](/aspnet/core/fundamentals/host/hosted-services), консольных приложениях (.NET Core/платформа .NET Framework) и т. д.

## <a name="prerequisites"></a>Предварительные условия

Допустимый ключ инструментирования Application Insights. Этот ключ необходим для отправки любых данных телеметрии в Application Insights. Если необходимо создать новый Application Insights ресурс для получения ключа инструментирования, см. раздел [Создание ресурса Application Insights](./create-new-resource.md).

## <a name="using-application-insights-sdk-for-worker-services"></a>Использование пакета SDK для Application Insights для служб Worker

1. Установите пакет [Microsoft. ApplicationInsights. воркерсервице](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WorkerService) в приложение.
   В следующем фрагменте кода показаны изменения, которые необходимо добавить в `.csproj` файл проекта.

```xml
    <ItemGroup>
        <PackageReference Include="Microsoft.ApplicationInsights.WorkerService" Version="2.13.1" />
    </ItemGroup>
```

1. Вызовите `AddApplicationInsightsTelemetryWorkerService(string instrumentationKey)` метод расширения для `IServiceCollection` , указав ключ инструментирования. Этот метод должен вызываться в начале приложения. Точное расположение зависит от типа приложения.

1. Получите `ILogger` экземпляр или `TelemetryClient` экземпляр из контейнера внедрения зависимостей (DI), вызвав `serviceProvider.GetRequiredService<TelemetryClient>();` или используя внедрение конструктора. На этом шаге запустится Настройка `TelemetryConfiguration` модулей и модули автоматической коллекции.

Конкретные инструкции для каждого типа приложений описаны в следующих разделах.

## <a name="net-core-30-worker-service-application"></a>Приложение рабочей службы для .NET Core 3,0

Полный [Пример общего доступа](https://github.com/microsoft/ApplicationInsights-dotnet/tree/develop/examples/WorkerServiceSDK/WorkerServiceSampleWithApplicationInsights)

1. Загрузка и установка [.NET Core 3,0](https://dotnet.microsoft.com/download/dotnet-core/3.0)
2. Создание нового проекта рабочей службы с помощью нового шаблона проекта или командной строки Visual Studio `dotnet new worker`
3. Установите пакет [Microsoft. ApplicationInsights. воркерсервице](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WorkerService) в приложение.

4. Добавьте в `services.AddApplicationInsightsTelemetryWorkerService();` `CreateHostBuilder()` метод в `Program.cs` классе, как показано в следующем примере:

```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureServices((hostContext, services) =>
            {
                services.AddHostedService<Worker>();
                services.AddApplicationInsightsTelemetryWorkerService();
            });
```

5. Измените `Worker.cs` , как в примере ниже.

```csharp
    using Microsoft.ApplicationInsights;
    using Microsoft.ApplicationInsights.DataContracts;

    public class Worker : BackgroundService
    {
        private readonly ILogger<Worker> _logger;
        private TelemetryClient _telemetryClient;
        private static HttpClient _httpClient = new HttpClient();

        public Worker(ILogger<Worker> logger, TelemetryClient tc)
        {
            _logger = logger;
            _telemetryClient = tc;
        }

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            while (!stoppingToken.IsCancellationRequested)
            {
                _logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);

                using (_telemetryClient.StartOperation<RequestTelemetry>("operation"))
                {
                    _logger.LogWarning("A sample warning message. By default, logs with severity Warning or higher is captured by Application Insights");
                    _logger.LogInformation("Calling bing.com");
                    var res = await _httpClient.GetAsync("https://bing.com");
                    _logger.LogInformation("Calling bing completed with status:" + res.StatusCode);
                    _telemetryClient.TrackEvent("Bing call event completed");
                }

                await Task.Delay(1000, stoppingToken);
            }
        }
    }
```

6. Настройте ключ инструментирования.

    Хотя ключ инструментирования можно указать в качестве аргумента `AddApplicationInsightsTelemetryWorkerService` , рекомендуется указать ключ инструментирования в конфигурации. В следующем образце кода показано, как указать ключ инструментирования в `appsettings.json` . Убедитесь `appsettings.json` , что во время публикации скопировано в корневую папку приложения.

```json
    {
        "ApplicationInsights":
        {
            "InstrumentationKey": "putinstrumentationkeyhere"
        },
        "Logging":
        {
            "LogLevel":
            {
                "Default": "Warning"
            }
        }
    }
```

Кроме того, можно указать ключ инструментирования в любой из следующих переменных среды.
`APPINSIGHTS_INSTRUMENTATIONKEY` или `ApplicationInsights:InstrumentationKey`

Пример: `SET ApplicationInsights:InstrumentationKey=putinstrumentationkeyhere`
НИ `SET APPINSIGHTS_INSTRUMENTATIONKEY=putinstrumentationkeyhere`

Как правило, `APPINSIGHTS_INSTRUMENTATIONKEY` указывает ключ инструментирования для приложений, развернутых в веб-приложениях в виде веб-заданий.

> [!NOTE]
> Ключ инструментирования, указанный в коде, передается через переменную среды `APPINSIGHTS_INSTRUMENTATIONKEY` , которая WINS поверх других параметров.

## <a name="aspnet-core-background-tasks-with-hosted-services"></a>ASP.NET Core фоновых задач с размещенными службами

В [этом](/aspnet/core/fundamentals/host/hosted-services?tabs=visual-studio) документе описывается создание фоновых задач в приложении ASP.NET Core 2.1/2.2.

Полный [Пример общего доступа](https://github.com/microsoft/ApplicationInsights-dotnet/tree/develop/examples/WorkerServiceSDK/BackgroundTasksWithHostedService)

1. Установите пакет [Microsoft. ApplicationInsights. воркерсервице](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WorkerService) в приложение.
2. Добавьте `services.AddApplicationInsightsTelemetryWorkerService();` в `ConfigureServices()` метод, как показано в следующем примере:

```csharp
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .ConfigureAppConfiguration((hostContext, config) =>
            {
                config.AddJsonFile("appsettings.json", optional: true);
            })
            .ConfigureServices((hostContext, services) =>
            {
                services.AddLogging();
                services.AddHostedService<TimedHostedService>();

                // instrumentation key is read automatically from appsettings.json
                services.AddApplicationInsightsTelemetryWorkerService();
            })
            .UseConsoleLifetime()
            .Build();

        using (host)
        {
            // Start the host
            await host.StartAsync();

            // Wait for the host to shutdown
            await host.WaitForShutdownAsync();
        }
    }
```

Ниже приведен код, в `TimedHostedService` котором находится логика фоновой задачи.

```csharp
    using Microsoft.ApplicationInsights;
    using Microsoft.ApplicationInsights.DataContracts;

    public class TimedHostedService : IHostedService, IDisposable
    {
        private readonly ILogger _logger;
        private Timer _timer;
        private TelemetryClient _telemetryClient;
        private static HttpClient httpClient = new HttpClient();

        public TimedHostedService(ILogger<TimedHostedService> logger, TelemetryClient tc)
        {
            _logger = logger;
            this._telemetryClient = tc;
        }

        public Task StartAsync(CancellationToken cancellationToken)
        {
            _logger.LogInformation("Timed Background Service is starting.");

            _timer = new Timer(DoWork, null, TimeSpan.Zero,
                TimeSpan.FromSeconds(1));

            return Task.CompletedTask;
        }

        private void DoWork(object state)
        {
            _logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);

            using (_telemetryClient.StartOperation<RequestTelemetry>("operation"))
            {
                _logger.LogWarning("A sample warning message. By default, logs with severity Warning or higher is captured by Application Insights");
                _logger.LogInformation("Calling bing.com");
                var res = httpClient.GetAsync("https://bing.com").GetAwaiter().GetResult();
                _logger.LogInformation("Calling bing completed with status:" + res.StatusCode);
                _telemetryClient.TrackEvent("Bing call event completed");
            }
        }
    }
```

3. Настройте ключ инструментирования.
   Используйте те же самые, что и в `appsettings.json` примере рабочей службы .NET Core 3,0 выше.

## <a name="net-corenet-framework-console-application"></a>Консольное приложение .NET Core/. NET Framework

Как упоминалось в начале этой статьи, новый пакет можно использовать для включения телеметрия Application Insights даже из обычного консольного приложения. Этот пакет предназначен [`NetStandard2.0`](/dotnet/standard/net-standard) для использования и поэтому может использоваться для консольных приложений в .NET Core 2,0 или более поздней версии, а также платформа .NET Framework 4.7.2 или более поздней версии.

Полный [Пример общего доступа](https://github.com/microsoft/ApplicationInsights-dotnet/tree/develop/examples/WorkerServiceSDK/ConsoleAppWithApplicationInsights)

1. Установите пакет [Microsoft. ApplicationInsights. воркерсервице](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WorkerService) в приложение.

2. Измените программу. cs, как показано ниже.

```csharp
    using Microsoft.ApplicationInsights;
    using Microsoft.ApplicationInsights.DataContracts;
    using Microsoft.Extensions.DependencyInjection;
    using Microsoft.Extensions.Logging;
    using System;
    using System.Net.Http;
    using System.Threading.Tasks;

    namespace WorkerSDKOnConsole
    {
        class Program
        {
            static async Task Main(string[] args)
            {
                // Create the DI container.
                IServiceCollection services = new ServiceCollection();

                // Being a regular console app, there is no appsettings.json or configuration providers enabled by default.
                // Hence instrumentation key and any changes to default logging level must be specified here.
                services.AddLogging(loggingBuilder => loggingBuilder.AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>("Category", LogLevel.Information));
                services.AddApplicationInsightsTelemetryWorkerService("instrumentationkeyhere");

                // Build ServiceProvider.
                IServiceProvider serviceProvider = services.BuildServiceProvider();

                // Obtain logger instance from DI.
                ILogger<Program> logger = serviceProvider.GetRequiredService<ILogger<Program>>();

                // Obtain TelemetryClient instance from DI, for additional manual tracking or to flush.
                var telemetryClient = serviceProvider.GetRequiredService<TelemetryClient>();

                var httpClient = new HttpClient();

                while (true) // This app runs indefinitely. replace with actual application termination logic.
                {
                    logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);

                    // Replace with a name which makes sense for this operation.
                    using (telemetryClient.StartOperation<RequestTelemetry>("operation"))
                    {
                        logger.LogWarning("A sample warning message. By default, logs with severity Warning or higher is captured by Application Insights");
                        logger.LogInformation("Calling bing.com");                    
                        var res = await httpClient.GetAsync("https://bing.com");
                        logger.LogInformation("Calling bing completed with status:" + res.StatusCode);
                        telemetryClient.TrackEvent("Bing call event completed");
                    }

                    await Task.Delay(1000);
                }

                // Explicitly call Flush() followed by sleep is required in Console Apps.
                // This is to ensure that even if application terminates, telemetry is sent to the back-end.
                telemetryClient.Flush();
                Task.Delay(5000).Wait();
            }
        }
    }
```

Это консольное приложение также использует то же значение по умолчанию `TelemetryConfiguration` , и его можно настроить так же, как примеры в предыдущем разделе.

## <a name="run-your-application"></a>Запуск приложения

Запустите приложение. В примере работники из всех перечисленных выше выводят вызов HTTP каждую секунду в bing.com, а также создает несколько журналов с помощью `ILogger` . Эти строки упаковываются внутри `StartOperation` вызова `TelemetryClient` , который используется для создания операции (в этом примере — `RequestTelemetry` "операция"). Application Insights соберет эти журналы ILogger (предупреждение или выше по умолчанию) и зависимости, и они будут связаны с `RequestTelemetry` отношением со связью "родители-потомки". Корреляция также работает с границей перекрестных процессов и сетей. Например, если вызов был выполнен для другого отслеживаемого компонента, он также будет связан с этим родителем.

Эту пользовательскую операцию `RequestTelemetry` можно рассматривать как эквивалент входящего веб-запроса в обычном веб-приложении. Хотя нет необходимости использовать операцию, она подходит для [Application Insightsной модели данных корреляции](./correlation.md) `RequestTelemetry` , которая действует как родительская операция, и все данные телеметрии, созданные в рабочей итерации, рассматриваются как логически относящиеся к одной и той же операции. Такой подход также гарантирует, что все создаваемые данные телеметрии (автоматические и ручные) будут одинаковыми `operation_id` . Когда выборка основана на `operation_id` , алгоритм выборки либо сохраняет, либо удаляет все данные телеметрии из одной итерации.

Ниже перечислены полные данные телеметрии, автоматически собираемые Application Insights.

### <a name="live-metrics"></a>Интерактивные метрики

Для быстрой проверки правильности настройки Application Insights мониторинга можно использовать [динамические метрики](./live-stream.md) . Это может занять несколько минут, прежде чем данные телеметрии будут отображаться на портале и в аналитике, в динамических метриках будет показано использование ЦП запущенного процесса практически в реальном времени. Он также может отображать другие данные телеметрии, такие как запросы, зависимости, трассировки и т. д.

### <a name="ilogger-logs"></a>Журналы ILogger

Журналы `ILogger` , созданные с уровнем серьезности `Warning` или выше, автоматически фиксируются. Следуйте инструкциям [ILogger](ilogger.md#control-logging-level) , чтобы настроить, какие уровни журнала фиксируются Application Insights.

### <a name="dependencies"></a>Зависимости

Коллекция зависимостей включена по умолчанию. В [этой](asp-net-dependencies.md#automatically-tracked-dependencies) статье объясняются автоматически собираемые зависимости, а также инструкции по отслеживанию вручную.

### <a name="eventcounter"></a>евенткаунтер

`EventCounterCollectionModule` параметр включен по умолчанию и будет собираются набор счетчиков по умолчанию из приложений .NET Core 3,0. В руководстве по [евенткаунтер](eventcounters.md) представлен набор собираемых счетчиков. Он также содержит инструкции по настройке списка.

### <a name="manually-tracking-additional-telemetry"></a>Отслеживание дополнительных данных телеметрии вручную

Хотя пакет SDK автоматически собирает данные телеметрии, как описано выше, в большинстве случаев пользователь должен будет отправить дополнительные данные телеметрии в службу Application Insights. Рекомендуемый способ отслеживания дополнительной телеметрии заключается в получении экземпляра `TelemetryClient` из внедрения зависимостей и последующем вызове одного из поддерживаемых `TrackXXX()` методов [API](api-custom-events-metrics.md) . Другим типичным вариантом использования является [пользовательское отслеживание операций](custom-operations-tracking.md). Этот подход показан в примерах рабочих ролей выше.

## <a name="configure-the-application-insights-sdk"></a>Настройка пакета SDK для Application Insights

Значение по умолчанию, `TelemetryConfiguration` используемое пакетом SDK для службы рабочей роли, аналогично автоматической конфигурации, используемой в ASP.NET или ASP.NET Core приложении, за вычетом telemetryinitializer, используемого для обогащения данных телеметрии из `HttpContext` .

Можно настроить Application Insights SDK для службы рабочей роли, чтобы изменить конфигурацию по умолчанию. Пользователи пакета SDK Application Insights ASP.NET Core могут быть знакомы с изменением конфигурации с помощью ASP.NET Core встроенного [внедрения зависимостей](/aspnet/core/fundamentals/dependency-injection). Пакет SDK для Воркерсервице также основан на аналогичных принципах. Сделайте почти все изменения конфигурации в `ConfigureServices()` разделе, вызвав соответствующие методы в `IServiceCollection` , как описано ниже.

> [!NOTE]
> При использовании этого пакета SDK изменение конфигурации посредством изменения не `TelemetryConfiguration.Active` поддерживается, и изменения не будут отражены.

### <a name="using-applicationinsightsserviceoptions"></a>Использование Аппликатионинсигхтссервицеоптионс

Можно изменить несколько общих параметров, передав `ApplicationInsightsServiceOptions` `AddApplicationInsightsTelemetryWorkerService` , как показано в следующем примере:

```csharp
using Microsoft.ApplicationInsights.WorkerService;

public void ConfigureServices(IServiceCollection services)
{
    var aiOptions = new ApplicationInsightsServiceOptions();
    // Disables adaptive sampling.
    aiOptions.EnableAdaptiveSampling = false;

    // Disables QuickPulse (Live Metrics stream).
    aiOptions.EnableQuickPulseMetricStream = false;
    services.AddApplicationInsightsTelemetryWorkerService(aiOptions);
}
```

Обратите внимание, что `ApplicationInsightsServiceOptions` в этом пакете SDK находится в пространстве имен, а не `Microsoft.ApplicationInsights.WorkerService` `Microsoft.ApplicationInsights.AspNetCore.Extensions` в ASP.NET Core SDK.

Часто используемые параметры в `ApplicationInsightsServiceOptions`

|Параметр | Описание | По умолчанию
|---------------|-------|-------
|енаблекуиккпулсеметрикстреам | Включить или отключить функцию Ливеметрикс | true
|енаблеадаптивесамплинг | Включение или отключение адаптивной выборки | true
|енаблехеартбеат | Функция "включить/отключить пульс", которая периодически (по умолчанию составляет 15 минут) отправляет пользовательскую метрику "Хеартбеатстате" со сведениями о среде выполнения, такими как версия .NET, сведения о среде Azure, если применимо, и т. д. | true
|аддаутоколлектедметрицекстрактор | Включите или отключите средство извлечения Аутоколлектедметрикс, которое представляет собой Телеметрипроцессор, который отправляет предварительно агрегированные метрики о запросах и зависимостях перед выполнением выборки. | true
|енабледиагностикстелеметримодуле | Включить или отключить `DiagnosticsTelemetryModule` . Отключение этого параметра приведет к игнорированию следующих параметров. `EnableHeartbeat`, `EnableAzureInstanceMetadataTelemetryModule`, `EnableAppServicesHeartbeatTelemetryModule` | true

См. список [настраиваемых параметров `ApplicationInsightsServiceOptions` в](https://github.com/microsoft/ApplicationInsights-dotnet/blob/develop/NETCORE/src/Shared/Extensions/ApplicationInsightsServiceOptions.cs) для наиболее актуального списка.

### <a name="sampling"></a>Дискретизация

Пакет SDK Application Insights для службы рабочей роли поддерживает как фиксированную, так и адаптивную выборку. Адаптивная выборка включена по умолчанию. Выборку можно отключить с помощью `EnableAdaptiveSampling` параметра в [аппликатионинсигхтссервицеоптионс](#using-applicationinsightsserviceoptions)

Чтобы настроить дополнительные параметры выборки, можно использовать следующий пример.

```csharp
using Microsoft.ApplicationInsights.Extensibility;
using Microsoft.ApplicationInsights.WorkerService;

public void ConfigureServices(IServiceCollection services)
{
    // ...

    var aiOptions = new ApplicationInsightsServiceOptions();
    
    // Disable adaptive sampling.
    aiOptions.EnableAdaptiveSampling = false;
    services.AddApplicationInsightsTelemetryWorkerService(aiOptions);

    // Add Adaptive Sampling with custom settings.
    // the following adds adaptive sampling with 15 items per sec.
    services.Configure<TelemetryConfiguration>((telemetryConfig) =>
        {
            var builder = telemetryConfig.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
            builder.UseAdaptiveSampling(maxTelemetryItemsPerSecond: 15);
            builder.Build();
        });
    //...
}
```

Дополнительные сведения можно найти в документе [выборки](#sampling) .

### <a name="adding-telemetryinitializers"></a>Добавление Telemetryinitializer

Используйте [инициализаторы телеметрии](./api-filtering-sampling.md#addmodify-properties-itelemetryinitializer) , если нужно определить свойства, которые отправляются со всеми данными телеметрии.

Добавьте новые `TelemetryInitializer` в `DependencyInjection` контейнер и пакет SDK, чтобы автоматически добавить их в `TelemetryConfiguration` .

```csharp
    using Microsoft.ApplicationInsights.Extensibility;

    public void ConfigureServices(IServiceCollection services)
    {
        services.AddSingleton<ITelemetryInitializer, MyCustomTelemetryInitializer>();
        services.AddApplicationInsightsTelemetryWorkerService();
    }
```

### <a name="removing-telemetryinitializers"></a>Удаление Telemetryinitializer

Инициализаторы телеметрии представлены по умолчанию. Чтобы удалить все или конкретные инициализаторы телеметрии, используйте следующий пример кода *после* вызова метода `AddApplicationInsightsTelemetryWorkerService()` .

```csharp
   public void ConfigureServices(IServiceCollection services)
   {
        services.AddApplicationInsightsTelemetryWorkerService();
        // Remove a specific built-in telemetry initializer
        var tiToRemove = services.FirstOrDefault<ServiceDescriptor>
                            (t => t.ImplementationType == typeof(AspNetCoreEnvironmentTelemetryInitializer));
        if (tiToRemove != null)
        {
            services.Remove(tiToRemove);
        }

        // Remove all initializers
        // This requires importing namespace by using Microsoft.Extensions.DependencyInjection.Extensions;
        services.RemoveAll(typeof(ITelemetryInitializer));
   }
```

### <a name="adding-telemetry-processors"></a>Добавление обработчиков данных телеметрии

Можно добавить пользовательские обработчики данных телеметрии в с `TelemetryConfiguration` помощью метода расширения `AddApplicationInsightsTelemetryProcessor` в `IServiceCollection` . Вы используете обработчики данных телеметрии в [сценариях расширенной фильтрации](./api-filtering-sampling.md#itelemetryprocessor-and-itelemetryinitializer) , чтобы обеспечить более прямое управление включенными или исключенными из данных телеметрии, отправляемых в службу Application Insights. Используйте следующий пример.

```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddApplicationInsightsTelemetryWorkerService();
        services.AddApplicationInsightsTelemetryProcessor<MyFirstCustomTelemetryProcessor>();
        // If you have more processors:
        services.AddApplicationInsightsTelemetryProcessor<MySecondCustomTelemetryProcessor>();
    }
```

### <a name="configuring-or-removing-default-telemetrymodules"></a>Настройка или удаление Телеметримодулес по умолчанию

Application Insights использует модули телеметрии для автоматического сбора данных телеметрии о конкретных рабочих нагрузках без необходимости отслеживания вручную.

Следующие модули автоматической коллекции включены по умолчанию. Эти модули отвечают за автоматическое сбор данных телеметрии. Их можно отключить или настроить для изменения их поведения по умолчанию.

* `DependencyTrackingTelemetryModule`
* `PerformanceCollectorModule`
* `QuickPulseTelemetryModule`
* `AppServicesHeartbeatTelemetryModule` — (В настоящее время возникла проблема с участием этого модуля телеметрии. Временное решение см. в описании [проблемы GitHub 1689](https://github.com/microsoft/ApplicationInsights-dotnet/issues/1689
).)
* `AzureInstanceMetadataTelemetryModule`

Чтобы настроить любое значение по умолчанию `TelemetryModule` , используйте метод расширения `ConfigureTelemetryModule<T>` для `IServiceCollection` , как показано в следующем примере.

```csharp
    using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse;
    using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector;

    public void ConfigureServices(IServiceCollection services)
    {
        services.AddApplicationInsightsTelemetryWorkerService();

            // The following configures QuickPulseTelemetryModule.
            // Similarly, any other default modules can be configured.
            services.ConfigureTelemetryModule<QuickPulseTelemetryModule>((module, o) =>
            {
                module.AuthenticationApiKey = "keyhere";
            });

            // The following removes PerformanceCollectorModule to disable perf-counter collection.
            // Similarly, any other default modules can be removed.
            var performanceCounterService = services.FirstOrDefault<ServiceDescriptor>
                                        (t => t.ImplementationType == typeof(PerformanceCollectorModule));
            if (performanceCounterService != null)
            {
                services.Remove(performanceCounterService);
            }
    }
```

### <a name="configuring-telemetry-channel"></a>Настройка канала телеметрии

Каналом по умолчанию является `ServerTelemetryChannel` . Его можно переопределить, как показано в следующем примере.

```csharp
using Microsoft.ApplicationInsights.Channel;

    public void ConfigureServices(IServiceCollection services)
    {
        // Use the following to replace the default channel with InMemoryChannel.
        // This can also be applied to ServerTelemetryChannel.
        services.AddSingleton(typeof(ITelemetryChannel), new InMemoryChannel() {MaxTelemetryBufferCapacity = 19898 });

        services.AddApplicationInsightsTelemetryWorkerService();
    }
```

### <a name="disable-telemetry-dynamically"></a>Динамическое отключение телеметрии

Если вы хотите отключить данные телеметрии условно и динамически, вы можете разрешить `TelemetryConfiguration` экземпляр с контейнером ASP.NET Core внедрения зависимостей в любом месте кода и установить `DisableTelemetry` для него флаг.

```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddApplicationInsightsTelemetryWorkerService();
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env, TelemetryConfiguration configuration)
    {
        configuration.DisableTelemetry = true;
        ...
    }
```

## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

### <a name="how-can-i-track-telemetry-thats-not-automatically-collected"></a>Как можно отвестить данные телеметрии, которые не собираются автоматически?

Получите экземпляр `TelemetryClient` с помощью внедрения конструктора и вызовите `TrackXXX()` для него необходимый метод. Мы не рекомендуем создавать новые `TelemetryClient` экземпляры. Одноэлементный экземпляр `TelemetryClient` уже зарегистрирован в `DependencyInjection` контейнере, который совместно `TelemetryConfiguration` используется с остальными данными телеметрии. Создание нового `TelemetryClient` экземпляра рекомендуется, только если требуется конфигурация, отделяющая остальные данные телеметрии.

### <a name="can-i-use-visual-studio-ide-to-onboard-application-insights-to-a-worker-service-project"></a>Можно ли использовать Visual Studio IDE для подключения Application Insights к проекту рабочей службы?

В настоящее время подключение интегрированной среды разработки Visual Studio поддерживается только для приложений ASP.NET/ASP.NET Core. Этот документ будет обновлен, когда Visual Studio отправит поддержку для адаптации приложений рабочей службы.

### <a name="can-i-enable-application-insights-monitoring-by-using-tools-like-status-monitor"></a>Можно ли включить Application Insights мониторинг с помощью таких средств, как монитор состояния?

Нет. В настоящее время [Монитор состояния](./monitor-performance-live-website-now.md) и [Монитор состояния v2](./status-monitor-v2-overview.md) поддерживают только ASP.NET 4. x.

### <a name="if-i-run-my-application-in-linux-are-all-features-supported"></a>Если я запускаю приложение в Linux, поддерживаются ли все функции?

Да. Поддержка функций для этого пакета SDK одинакова на всех платформах, за исключением следующих:

* Счетчики производительности поддерживаются только в Windows, за исключением ЦП или памяти процесса, показанными в динамических метриках.
* Несмотря на то что `ServerTelemetryChannel` включен по умолчанию, если приложение выполняется в Linux или MacOS, канал не создает папку локального хранилища, чтобы временно сохранять данные телеметрии в случае проблем с сетью. Из-за этого ограничения данные телеметрии теряются при наличии временных проблем с сетью или сервером. Чтобы обойти эту ошибку, настройте локальную папку для канала:

```csharp
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel;

    public void ConfigureServices(IServiceCollection services)
    {
        // The following will configure the channel to use the given folder to temporarily
        // store telemetry items during network or Application Insights server issues.
        // User should ensure that the given folder already exists
        // and that the application has read/write permissions.
        services.AddSingleton(typeof(ITelemetryChannel),
                                new ServerTelemetryChannel () {StorageFolder = "/tmp/myfolder"});
        services.AddApplicationInsightsTelemetryWorkerService();
    }
```

## <a name="sample-applications"></a>Примеры приложений

[Консольное приложение .NET Core](https://github.com/microsoft/ApplicationInsights-dotnet/tree/develop/examples/WorkerServiceSDK/ConsoleAppWithApplicationInsights) Используйте этот пример, если вы используете консольное приложение, написанное на .NET Core (2,0 или более поздней версии) или платформа .NET Framework (4.7.2 или более поздней версии).

[Фоновые задачи ASP .NET Core с хостедсервицес](https://github.com/microsoft/ApplicationInsights-dotnet/tree/develop/examples/WorkerServiceSDK/BackgroundTasksWithHostedService) Используйте этот пример, если вы используете Asp.Net Core 2.1/2.2 и создаете фоновые задачи в соответствии с [официальным руководством.](/aspnet/core/fundamentals/host/hosted-services)

[Рабочая служба .NET Core 3,0](https://github.com/microsoft/ApplicationInsights-dotnet/tree/develop/examples/WorkerServiceSDK/WorkerServiceSampleWithApplicationInsights) Используйте этот пример, если у вас есть приложение рабочей службы .NET Core 3,0 в соответствии с [официальным руководством](/aspnet/core/fundamentals/host/hosted-services?tabs=visual-studio#worker-service-template)

## <a name="open-source-sdk"></a>Пакет SDK с открытым исходным кодом

* [Чтение кода и внесение в него вклада](https://github.com/microsoft/ApplicationInsights-dotnet).

Последние обновления и исправления ошибок см. [в заметках о выпуске](./release-notes.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Используйте API](./api-custom-events-metrics.md) для отправки собственных событий и метрик для получения подробного представления о производительности и использовании приложения.
* [Следите за автоматической](./auto-collect-dependencies.md)отслеживанием дополнительных зависимостей.
* [Расширить или отфильтровать автоматические собираемые данные телеметрии](./api-filtering-sampling.md).
* [Внедрение зависимостей в ASP.NET Core](/aspnet/core/fundamentals/dependency-injection).
