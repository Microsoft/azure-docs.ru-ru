---
title: Изучение журналов трассировки .NET с помощью ILogger Azure Application Insights
description: Примеры использования поставщика Azure Application Insights ILogger с ASP.NET Core и консольными приложениями.
ms.topic: conceptual
ms.date: 02/19/2019
ms.reviewer: mbullwin
ms.openlocfilehash: e7d0bd6f7d93eac944e0d53a8e898463bf9887f9
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104592274"
---
# <a name="applicationinsightsloggerprovider-for-microsoftextensionlogging"></a>Аппликатионинсигхтслогжерпровидер для Microsoft. extension. Logging

В этой статье показано, как использовать `ApplicationInsightsLoggerProvider` для записи `ILogger` журналов в консольных и ASP.NET Core приложениях.
Дополнительные сведения о ведении журнала см. [в разделе Ведение журнала в ASP.NET Core](/aspnet/core/fundamentals/logging).

## <a name="aspnet-core-applications"></a>ASP.NET Core приложения

`ApplicationInsightsLoggerProvider` включен по умолчанию для ASP.NET Core приложений, если ApplicationInsights настроен с использованием [кода](./asp-net-core.md) или подхода [без кода](./azure-web-apps.md?tabs=netcore#enable-agent-based-monitoring) .

В Application Insights по умолчанию отправляются только журналы *предупреждений* и более поздних версий `ILogger` (из всех [категорий](/aspnet/core/fundamentals/logging/#log-category)). Но [это поведение можно настроить](./asp-net-core.md#how-do-i-customize-ilogger-logs-collection). Для записи журналов ILogger из **программы Program. CS** или **Startup. CS** необходимо выполнить дополнительные действия. (См. раздел [захват журналов ILogger из Startup. cs и Program. cs в ASP.NET Core приложениях](#capture-ilogger-logs-from-startupcs-and-programcs-in-aspnet-core-apps).)

Если вы хотите использовать `ApplicationInsightsLoggerProvider` без других Application Insights мониторинга, выполните следующие действия.

1. Установите пакет NuGet:

   ```xml
    <ItemGroup>
        <PackageReference Include="Microsoft.Extensions.Logging.ApplicationInsights" Version="2.15.0" />  
    </ItemGroup>
   ```

1. Измените `Program.cs` , как показано ниже:

   ```csharp
   using Microsoft.AspNetCore;
   using Microsoft.AspNetCore.Hosting;
   using Microsoft.Extensions.Logging;

   public class Program
   {
       public static void Main(string[] args)
       {
           CreateWebHostBuilder(args).Build().Run();
       }

       public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
         WebHost.CreateDefaultBuilder(args)
           .UseStartup<Startup>()
         .ConfigureLogging(
               builder =>
               {
                   // Providing an instrumentation key here is required if you're using
                   // standalone package Microsoft.Extensions.Logging.ApplicationInsights
                   // or if you want to capture logs from early in the application startup
                   // pipeline from Startup.cs or Program.cs itself.
                   builder.AddApplicationInsights("put-actual-ikey-here");

                   // Optional: Apply filters to control what logs are sent to Application Insights.
                   // The following configures LogLevel Information or above to be sent to
                   // Application Insights for all categories.
                   builder.AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>
                                    ("", LogLevel.Information);
               }
           );
   }
   ```

Код на шаге 2 настраивает `ApplicationInsightsLoggerProvider` . В следующем коде показан пример класса контроллера, который использует `ILogger` для отправки журналов. Журналы фиксируются Application Insights.

```csharp
public class ValuesController : ControllerBase
{
    private readonly ILogger _logger;

    public ValuesController(ILogger<ValuesController> logger)
    {
        _logger = logger;
    }

    // GET api/values
    [HttpGet]
    public ActionResult<IEnumerable<string>> Get()
    {
        _logger.LogWarning("An example of a Warning trace..");
        _logger.LogError("An example of an Error level message");
        return new string[] { "value1", "value2" };
    }
}
```

### <a name="capture-ilogger-logs-from-startupcs-and-programcs-in-aspnet-core-apps"></a>Запись журналов ILogger из Startup. cs и Program. cs в ASP.NET Core Apps

> [!NOTE]
> В ASP.NET Core 3,0 и более поздних версий больше нельзя внедрять `ILogger` в Startup. cs и Program. cs. Дополнительные сведения см. в статье https://github.com/aspnet/Announcements/issues/353.

`ApplicationInsightsLoggerProvider` может собирать журналы с раннего запуска приложения. Хотя Аппликатионинсигхтслогжерпровидер автоматически включается в Application Insights (начиная с версии 2.7.1), ключ инструментирования не настраивается до последующего в конвейере. Таким образом, будут записываться только журналы из классов. другие **контроллера**. Чтобы записать каждый журнал, начиная с **программы Program. CS** и **Startup. CS** , необходимо явно включить ключ инструментирования для аппликатионинсигхтслогжерпровидер. Кроме того, *телеметриконфигуратион* не полностью настраивается при регистрации из **программы Program. CS** или **Startup. CS** . Поэтому эти журналы будут иметь минимальную конфигурацию, которая использует [InMemoryChannel](./telemetry-channels.md), без [выборки](./sampling.md)и не имеет стандартных [инициализаторов телеметрии или процессоров](./api-filtering-sampling.md).

В следующих примерах демонстрируется эта возможность с помощью **программы Program. CS** и **Startup. CS**.

#### <a name="example-programcs"></a>Пример программы. CS

```csharp
using Microsoft.AspNetCore;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Logging;

public class Program
{
    public static void Main(string[] args)
    {
        var host = CreateWebHostBuilder(args).Build();
        var logger = host.Services.GetRequiredService<ILogger<Program>>();
        logger.LogInformation("From Program. Running the host now..");
        host.Run();
    }

    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureLogging(
        builder =>
            {
            // Providing an instrumentation key here is required if you're using
            // standalone package Microsoft.Extensions.Logging.ApplicationInsights
            // or if you want to capture logs from early in the application startup 
            // pipeline from Startup.cs or Program.cs itself.
            builder.AddApplicationInsights("ikey");

            // Adding the filter below to ensure logs of all severity from Program.cs
            // is sent to ApplicationInsights.
            builder.AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>
                             (typeof(Program).FullName, LogLevel.Trace);

            // Adding the filter below to ensure logs of all severity from Startup.cs
            // is sent to ApplicationInsights.
            builder.AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>
                             (typeof(Startup).FullName, LogLevel.Trace);
            }
        );
}
```

#### <a name="example-startupcs"></a>Пример запуска. CS

```csharp
public class Startup
{
    private readonly ILogger _logger;

    public Startup(IConfiguration configuration, ILogger<Startup> logger)
    {
        Configuration = configuration;
        _logger = logger;
    }

    public IConfiguration Configuration { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        services.AddApplicationInsightsTelemetry();

        // The following will be picked up by Application Insights.
        _logger.LogInformation("Logging from ConfigureServices.");
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        if (env.IsDevelopment())
        {
            _logger.LogInformation("Configuring for Development environment");
            app.UseDeveloperExceptionPage();
        }
        else
        {
            _logger.LogInformation("Configuring for Production environment");
        }

        app.UseMvc();
    }
}
```

## <a name="migrate-from-the-old-applicationinsightsloggerprovider"></a>Миграция из старого Аппликатионинсигхтслогжерпровидер

Версии пакета SDK для Microsoft. ApplicationInsights. AspNet, предшествующие 2.7.1, поддерживали поставщик ведения журнала, который теперь устарел. Этот поставщик был включен с помощью метода расширения **аддаппликатионинсигхтс ()** объекта илогжерфактори. Рекомендуется выполнить миграцию на новый поставщик, который состоит из двух этапов:

1. Удалите вызов *илогжерфактори. аддаппликатионинсигхтс ()* из метода **Startup.Configключать ()** , чтобы избежать двойного протоколирования.
2. Повторно примените все правила фильтрации в коде, так как они не будут соответствовать новому поставщику. Перегрузки *илогжерфактори. аддаппликатионинсигхтс ()* заняли минимум LogLevel или функций фильтрации. При использовании нового поставщика фильтрация является частью самой платформы ведения журнала. Он не выполняется поставщиком Application Insights. Поэтому все фильтры, предоставляемые с помощью перегрузок *илогжерфактори. аддаппликатионинсигхтс ()* , следует удалить. Правила фильтрации и должны быть предоставлены в разделе [Управление инструкциями уровня ведения журнала](#control-logging-level) . Если для фильтрации журналов используется *appsettings.js* , то будет по-прежнему работать с новым поставщиком, так как в обоих случаях используется один и тот же псевдоним поставщика *ApplicationInsights*.

Вы по-прежнему можете использовать старый поставщик. (Он будет удален только при смене основной версии на 3. *XX*.) Однако рекомендуется перейти на новый поставщик по следующим причинам:

- В предыдущем поставщике отсутствует поддержка [областей журнала](/aspnet/core/fundamentals/logging#log-scopes). В новом поставщике свойства из области автоматически добавляются в собранные данные телеметрии в качестве пользовательских свойств.
- Теперь журналы могут быть захвачены в конвейере запуска приложения. Теперь можно записывать журналы из классов **Program** и **Startup** .
- С новым поставщиком фильтрация выполняется на уровне платформы. Вы можете фильтровать журналы в поставщике Application Insights так же, как и для других поставщиков, включая встроенные поставщики, такие как консоль, отладка и т. д. Вы также можете применить те же фильтры к нескольким поставщикам.
- В ASP.NET Core (2,0 и более поздних версиях) рекомендуемый способ  [включения регистраторов](https://github.com/aspnet/Announcements/issues/255) — использование методов расширения в илоггингбуилдер в самой **программе Program. CS** .

> [!Note]
> Новый поставщик доступен для приложений, предназначенных для NETSTANDARD 2.0 или более поздней версии. Начиная с [Microsoft. ApplicationInsights. ASPNET SDK](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore) версии 2.14.0, новый поставщик также доступен для приложений, предназначенных для платформа .NET Framework NET461 или более поздней версии. Если приложение предназначено для более старых версий .NET Core, например .NET Core 1,1, или если оно предназначено для платформа .NET Framework менее NET46, продолжайте использовать старый поставщик.

## <a name="console-application"></a>Консольное приложение

> [!NOTE]
> Существует новый пакет SDK Application Insights с именем [Microsoft. ApplicationInsights. воркерсервице](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WorkerService) , который можно использовать для включения Application Insights (ILogger и других данных телеметрии Application Insights) для любых консольных приложений. Рекомендуется использовать этот пакет и связанные инструкции [отсюда](./worker-service.md).

Если вы хотите просто использовать Аппликатионинсигхтслогжерпровидер без каких бы то ни было других Application Insights мониторинга, выполните следующие действия.

Установленные пакеты:

```xml
<ItemGroup>
  <PackageReference Include="Microsoft.Extensions.DependencyInjection" Version="2.1.0" />  
  <PackageReference Include="Microsoft.Extensions.Logging.ApplicationInsights" Version="2.15.0" />  
</ItemGroup>
```

```csharp
class Program
{
    static void Main(string[] args)
    {
        // Create the DI container.
        IServiceCollection services = new ServiceCollection();

        // Channel is explicitly configured to do flush on it later.
        var channel = new InMemoryChannel();
        services.Configure<TelemetryConfiguration>(
            (config) =>
            {
                config.TelemetryChannel = channel;
            }
        );

        // Add the logging pipelines to use. We are using Application Insights only here.
        services.AddLogging(builder =>
        {
            // Optional: Apply filters to configure LogLevel Trace or above is sent to
            // Application Insights for all categories.
            builder.AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>
                             ("", LogLevel.Trace);
            builder.AddApplicationInsights("--YourAIKeyHere--");
        });

        // Build ServiceProvider.
        IServiceProvider serviceProvider = services.BuildServiceProvider();

        ILogger<Program> logger = serviceProvider.GetRequiredService<ILogger<Program>>();

        logger.LogInformation("Logger is working");

        // Explicitly call Flush() followed by sleep is required in Console Apps.
        // This is to ensure that even if application terminates, telemetry is sent to the back-end.
        channel.Flush();
        Thread.Sleep(1000);
    }
}
```

В этом примере используется автономный пакет `Microsoft.Extensions.Logging.ApplicationInsights` . По умолчанию в этой конфигурации для отправки данных в Application Insights используется значение "не меньше нуля". Минимальное значение означает, что InMemoryChannel — это используемый канал. Нет выборки и ни одного стандартного Telemetryinitializer. Это поведение можно переопределить для консольного приложения, как показано в следующем примере.

Установите этот дополнительный пакет:

```xml
<PackageReference Include="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel" Version="2.9.1" />
```

В следующем разделе показано, как переопределить Телеметриконфигуратион по умолчанию с помощью метода **services.Configключать \<TelemetryConfiguration> ()** . В этом примере настраивается `ServerTelemetryChannel` и выборке. Он добавляет пользовательский ITelemetryInitializer в Телеметриконфигуратион.

```csharp
    // Create the DI container.
    IServiceCollection services = new ServiceCollection();
    var serverChannel = new ServerTelemetryChannel();
    services.Configure<TelemetryConfiguration>(
        (config) =>
        {
            config.TelemetryChannel = serverChannel;
            config.TelemetryInitializers.Add(new MyTelemetryInitalizer());
            config.DefaultTelemetrySink.TelemetryProcessorChainBuilder.UseSampling(5);
            serverChannel.Initialize(config);
        }
    );
    
    services.AddLogging(loggingBuilder =>
    {
        loggingBuilder.AddApplicationInsights("--YourAIKeyHere--");
    });

    ........
    ........

    // Explicitly calling Flush() followed by sleep is required in Console Apps.
    // This is to ensure that even if the application terminates, telemetry is sent to the back end.
    serverChannel.Flush();
    Thread.Sleep(1000);
```

## <a name="control-logging-level"></a>Управление уровнем ведения журнала

`ILogger` имеет встроенный механизм для применения [фильтрации журналов](/aspnet/core/fundamentals/logging#log-filtering). Это позволяет управлять журналами, которые отправляются каждому зарегистрированному поставщику, включая поставщика Application Insights. Фильтрацию можно выполнить либо в конфигурации (обычно с помощью *appsettings.jsв* файле), либо в коде.

В следующих примерах показано, как применять правила фильтра к `ApplicationInsightsLoggerProvider` .

### <a name="create-filter-rules-in-configuration-with-appsettingsjson"></a>Создание правил фильтрации в конфигурации с appsettings.js

Для Аппликатионинсигхтслогжерпровидер псевдонимом поставщика является `ApplicationInsights` . В следующем разделе *appsettings.jsо* том, что поставщики регистрации обычно задают журнал на уровне *предупреждения* и выше. Затем он переопределяет `ApplicationInsightsLoggerProvider` Категории журнала, которые начинаются с "Microsoft" на уровне " *Ошибка* " и выше.

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    },
    "ApplicationInsights": {
      "LogLevel": {
        "Microsoft": "Error"
      }
    }
  }
}
```

### <a name="create-filter-rules-in-code"></a>Создание правил фильтрации в коде

Следующий фрагмент кода настраивает журналы на наличие *предупреждений* и более поздних версий от всех категорий, а также для *ошибок* и более поздних версий от категорий, которые начинаются с "Microsoft" для отправки `ApplicationInsightsLoggerProvider` . Эта конфигурация такая же, как в предыдущем разделе *appsettings.json*.

```csharp
    WebHost.CreateDefaultBuilder(args)
    .UseStartup<Startup>()
    .ConfigureLogging(logging =>
      logging.AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>
                        ("", LogLevel.Warning)
             .AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>
                        ("Microsoft", LogLevel.Error);
```

## <a name="logging-scopes"></a>Области ведения журнала

`ApplicationInsightsLoggingProvider` поддерживает [области журнала](/aspnet/core/fundamentals/logging#log-scopes) и области по умолчанию включены.

Если область имеет тип `IReadOnlyCollection<KeyValuePair<string,object>>` , каждая пара "ключ-значение" в коллекции добавляется в данные телеметрии Application Insights в качестве пользовательских свойств. В приведенном ниже примере журналы будут записываться как `TraceTelemetry` и будут иметь значение ("myKey", "значения MyValue") в свойствах.

```csharp
    using (_logger.BeginScope(new Dictionary<string, object> { { "MyKey", "MyValue" } }))
    {
        _logger.LogError("An example of an Error level message");
    }
```

Если в качестве области используется любой другой тип, то они будут храниться в свойстве "область" в телеметрии Application Insights. В приведенном ниже примере у `TraceTelemetry` свойства будет имя Scope, содержащее область.

```csharp
    using (_logger.BeginScope("hello scope"))
    {
        _logger.LogError("An example of an Error level message");
    }
```

## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

### <a name="what-are-the-old-and-new-versions-of-applicationinsightsloggerprovider"></a>Каковы старые и новые версии Аппликатионинсигхтслогжерпровидер?

[Пакет SDK Microsoft. ApplicationInsights. AspNet](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore) включал встроенный Аппликатионинсигхтслогжерпровидер (Microsoft. ApplicationInsights. AspNetCore. Logging. аппликатионинсигхтслогжерпровидер), который был включен с помощью методов расширения **илогжерфактори** . Этот поставщик помечен как устаревший из версии 2.7.1. Он будет полностью удален при следующем изменении основной версии. Пакет [2.6.1 Microsoft. ApplicationInsights. AspNetCore](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore) не является устаревшим. Необходимо включить наблюдение за запросами, зависимостями и т. д.

Предложенным альтернативным вариантом является новый автономный пакет [Microsoft. Extensions. Logging. ApplicationInsights](https://www.nuget.org/packages/Microsoft.Extensions.Logging.ApplicationInsights), который содержит усовершенствованные методы Аппликатионинсигхтслогжерпровидер (Microsoft. Extensions. Logging. ApplicationInsights. аппликатионинсигхтслогжерпровидер) и расширения в илогжербуилдер для их включения.

[Пакет SDK для Microsoft. ApplicationInsights. AspNet](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore) версии 2.7.1 принимает зависимость от нового пакета и включает запись ILogger автоматически.

### <a name="why-are-some-ilogger-logs-shown-twice-in-application-insights"></a>Почему некоторые журналы ILogger показаны дважды в Application Insights?

Дублирование может произойти при наличии более старой (устаревшей) версии Аппликатионинсигхтслогжерпровидер с включенным вызовом `AddApplicationInsights` On `ILoggerFactory` . Проверьте, есть ли у метода **настройки** следующие параметры, и удалите его:

```csharp
 public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
 {
     loggerFactory.AddApplicationInsights(app.ApplicationServices, LogLevel.Warning);
     // ..other code.
 }
```

Если при отладке из Visual Studio вы настроили двойное ведение журнала, задайте для параметра значение `EnableDebugLogger`  *false* в коде, который включает Application Insights, как показано ниже. Это дублирование и исправление применимо только при отладке приложения.

```csharp
 public void ConfigureServices(IServiceCollection services)
 {
     ApplicationInsightsServiceOptions options = new ApplicationInsightsServiceOptions();
     options.EnableDebugLogger = false;
     services.AddApplicationInsightsTelemetry(options);
     // ..other code.
 }
```

### <a name="i-updated-to-microsoftapplicationinsightsaspnet-sdk-version-271-and-logs-from-ilogger-are-captured-automatically-how-do-i-turn-off-this-feature-completely"></a>Я обновил [пакет SDK для Microsoft. ApplicationInsights. AspNet](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore) версии 2.7.1, а журналы из ILogger фиксируются автоматически. Разделы справки полностью отключить эту функцию?

Сведения о том, как отфильтровать журналы в целом, см. в разделе [Управление уровнем ведения журнала](#control-logging-level) . Чтобы отключить Аппликатионинсигхтслогжерпровидер, используйте `LogLevel.None` :

**В коде:**

```csharp
    builder.AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>
                      ("", LogLevel.None);
```

**В конфигурации:**

```json
{
  "Logging": {
    "ApplicationInsights": {
      "LogLevel": {
        "Default": "None"
      }
}
```

### <a name="why-do-some-ilogger-logs-not-have-the-same-properties-as-others"></a>Почему некоторые журналы ILogger не имеют тех же свойств, что и другие?

Application Insights записывает и отправляет журналы ILogger, используя те же Телеметриконфигуратион, которые используются для всех остальных данных телеметрии. Но существует исключение. По умолчанию Телеметриконфигуратион не полностью настраивается при входе из **программы Program. CS** или **Startup. CS**. Журналы из этих мест не будут иметь конфигурации по умолчанию, поэтому они не будут выполнять все Telemetryinitializer и TelemetryProcessors.

### <a name="im-using-the-standalone-package-microsoftextensionsloggingapplicationinsights-and-i-want-to-log-some-additional-custom-telemetry-manually-how-should-i-do-that"></a>Я использую автономный пакет Microsoft. Extensions. Logging. ApplicationInsights и хочу записывать некоторые дополнительные пользовательские данные телеметрии вручную. Как это сделать?

При использовании автономного пакета `TelemetryClient` не внедряется в контейнер di, поэтому необходимо создать новый экземпляр `TelemetryClient` и использовать ту же конфигурацию, что и поставщик средства ведения журнала, как показано в следующем коде. Это гарантирует, что одна и та же конфигурация будет использоваться для всех пользовательских данных телеметрии, а также для телеметрии из ILogger.

```csharp
public class MyController : ApiController
{
   // This telemetryclient can be used to track additional telemetry using TrackXXX() api.
   private readonly TelemetryClient _telemetryClient;
   private readonly ILogger _logger;

   public MyController(IOptions<TelemetryConfiguration> options, ILogger<MyController> logger)
   {
        _telemetryClient = new TelemetryClient(options.Value);
        _logger = logger;
   }  
}
```

> [!NOTE]
> Если вы используете пакет Microsoft. ApplicationInsights. AspNetCore для включения Application Insights, измените этот код, чтобы получить его `TelemetryClient` непосредственно в конструкторе. Пример см. в разделе [часто задаваемые вопросы](./asp-net-core.md#frequently-asked-questions).


### <a name="what-application-insights-telemetry-type-is-produced-from-ilogger-logs-or-where-can-i-see-ilogger-logs-in-application-insights"></a>Какой тип телеметрии Application Insights создан из журналов ILogger? Или где можно просмотреть журналы ILogger в Application Insights?

Аппликатионинсигхтслогжерпровидер записывает журналы ILogger и создает Трацетелеметри из них. Если объект исключения передается в `Log` метод в `ILogger` , вместо трацетелеметри создается *ексцептионтелеметри* . Эти элементы телеметрии можно найти в тех же местах, что и другие Трацетелеметри или Ексцептионтелеметри, для Application Insights, включая портал, аналитику или локальный отладчик Visual Studio.

Если вы предпочитаете всегда отсылать Трацетелеметри, используйте следующий фрагмент кода: ```builder.AddApplicationInsights((opt) => opt.TrackExceptionsAsExceptionTelemetry = false);```

### <a name="i-dont-have-the-sdk-installed-and-i-use-the-azure-web-apps-extension-to-enable-application-insights-for-my-aspnet-core-applications-how-do-i-use-the-new-provider"></a>Я не установил пакет SDK, и я использую расширение веб-приложений Azure, чтобы включить Application Insights для приложений ASP.NET Core. Разделы справки использовать новый поставщик? 

Расширение Application Insights в веб-приложениях Azure использует новый поставщик. Правила фильтрации можно изменить в файле *appsettings.js* для приложения.

### <a name="im-using-the-standalone-package-microsoftextensionsloggingapplicationinsights-and-enabling-application-insights-provider-by-calling-builderaddapplicationinsightsikey-is-there-an-option-to-get-an-instrumentation-key-from-configuration"></a>Я использую автономный пакет Microsoft. Extensions. Logging. ApplicationInsights и включив поставщик Application Insights путем вызова **построителя. Аддаппликатионинсигхтс ("iKey")**. Есть ли возможность получить ключ инструментирования из конфигурации?


Измените Program. cs и appsettings.jsследующим образом:

   ```csharp
   public class Program
   {
       public static void Main(string[] args)
       {
           CreateWebHostBuilder(args).Build().Run();
       }

       public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
           WebHost.CreateDefaultBuilder(args)
               .UseStartup<Startup>()
               .ConfigureLogging((hostingContext, logging) =>
               {
                   // hostingContext.HostingEnvironment can be used to determine environments as well.
                   var appInsightKey = hostingContext.Configuration["myikeyfromconfig"];
                   logging.AddApplicationInsights(appInsightKey);
               });
   }
   ```

   Соответствующий раздел из `appsettings.json` :

   ```json
   {
     "myikeyfromconfig": "putrealikeyhere"
   }
   ```

Этот код необходим только при использовании автономного поставщика ведения журнала. Для регулярного мониторинга Application Insights ключ инструментирования загружается автоматически из пути конфигурации *ApplicationInsights: InstrumentationKey*. Appsettings.jsв должен выглядеть следующим образом:

   ```json
   {
     "ApplicationInsights":
       {
           "InstrumentationKey":"putrealikeyhere"
       }
   }
   ```

## <a name="next-steps"></a>Дальнейшие действия

См. также:

* [Ведение журналов в ASP.NET Core](/aspnet/core/fundamentals/logging)
* [Журналы трассировки .NET в Application Insights](./asp-net-trace-logs.md)

