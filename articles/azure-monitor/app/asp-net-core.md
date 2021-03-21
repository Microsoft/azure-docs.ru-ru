---
title: Application Insights Azure для ASP.NET Core приложений | Документация Майкрософт
description: Отслеживайте доступность, производительность и использование веб-приложений ASP.NET Core.
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 04/30/2020
ms.openlocfilehash: be4cbe5f95fa6901ae4299662750c232d4700a8a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101711507"
---
# <a name="application-insights-for-aspnet-core-applications"></a>Application Insights для ASP.NET Core приложений

В этой статье описано, как включить Application Insights для [ASP.NET Core](/aspnet/core) приложения. После выполнения инструкций, описанных в этой статье, Application Insights соберет запросы, зависимости, исключения, счетчики производительности, пульсы и журналы из приложения ASP.NET Core.

В качестве примера мы будем использовать [приложение MVC](/aspnet/core/tutorials/first-mvc-app) , предназначенное для `netcoreapp3.0` . Эти инструкции можно применить ко всем ASP.NET Coreным приложениям. Если вы используете [рабочую службу](/aspnet/core/fundamentals/host/hosted-services#worker-service-template), воспользуйтесь инструкциями, приведенными [здесь](./worker-service.md).

## <a name="supported-scenarios"></a>Поддерживаемые сценарии

[Пакет SDK Application Insights для ASP.NET Core](https://nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore) может отслеживать приложения независимо от того, где или как они выполняются. Если приложение работает и имеет сетевое подключение к Azure, можно собирать данные телеметрии. Мониторинг Application Insights поддерживается везде, где поддерживается .NET Core. Поддержка:
* **Операционная система**: Windows, Linux или Mac.
* **Метод размещения**. Внутри или вне процесса.
* **Метод развертывания**. Зависит от платформы либо автономный.
* **Веб-сервер**. IIS (Internet Information Server) или Kestrel.
* **Платформа размещения**. Функция веб-приложений службы приложений Azure, виртуальная машина Azure, Docker, служба Azure Kubernetes (AKS) и т. д.
* **Версия .NET Core**: все официально [Поддерживаемые](https://dotnet.microsoft.com/download/dotnet-core) версии .NET Core.
* **Интегрированная среда разработки**: Visual Studio, VS Code или Командная строка.

> [!NOTE]
> Для ASP.NET Core 3,1 требуется [Application Insights 2.8.0](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore/2.8.0) или более поздней версии.

## <a name="prerequisites"></a>Предварительные условия

- Работающее приложение ASP.NET Core. Если необходимо создать ASP.NET Core приложение, следуйте указаниям в этом [ASP.NET Coreном руководстве](/aspnet/core/getting-started/).
- Допустимый ключ инструментирования Application Insights. Этот ключ необходим для отправки любых данных телеметрии в Application Insights. Если необходимо создать новый Application Insights ресурс для получения ключа инструментирования, см. раздел [Создание ресурса Application Insights](./create-new-resource.md).

> [!IMPORTANT]
> Новые регионы Azure **должны** использовать строки подключения вместо ключей инструментирования. [Строка подключения](./sdk-connection-string.md?tabs=net) определяет ресурс, с которым необходимо связать данные телеметрии. Она также позволяет изменить конечные точки, которые ресурс будет использовать в качестве назначения для данных телеметрии. Необходимо скопировать строку подключения и добавить ее в код приложения или переменную среды.


## <a name="enable-application-insights-server-side-telemetry-visual-studio"></a>Включение телеметрии на стороне сервера Application Insights (Visual Studio)

Для Visual Studio для Mac используйте инструкции, указанные [вручную](#enable-application-insights-server-side-telemetry-no-visual-studio). Эта процедура поддерживается только в версии Windows Visual Studio.

1. Откройте проект в Visual Studio.

    > [!TIP]
    > При необходимости можно настроить систему управления версиями для проекта, чтобы можно было отвестися от всех изменений, которые Application Insights делает. Чтобы включить систему управления версиями, выберите **файл**  >  **Добавить в систему управления версиями**.

2. Выберите **проект**  >  **Добавить телеметрия Application Insights**.

3. Выберите **Начать**. Текст этого варианта может отличаться в зависимости от используемой версии Visual Studio. В некоторых более ранних версиях вместо нее используется кнопка **запустить бесплатно** .

4. Выберите свою подписку. Затем выберите пункт  >  **регистр** ресурсов.

5. После добавления Application Insights в проект убедитесь, что вы используете последний стабильный выпуск пакета SDK. Перейдите в **проект**  >  **Управление пакетами NuGet**  >  **Microsoft. ApplicationInsights. AspNetCore**. При необходимости выберите **Обновить**.

     ![Снимок экрана, на котором показано, где выбрать пакет Application Insights для обновления](./media/asp-net-core/update-nuget-package.png)

6. Если вы применяете необязательную подсказку и добавили проект в систему управления версиями, перейдите к **просмотру**  >  **Team Explorer**  >  **изменений**. Затем выберите каждый файл, чтобы просмотреть различие изменений, внесенных Application Insights телеметрии.

## <a name="enable-application-insights-server-side-telemetry-no-visual-studio"></a>Включить телеметрию на стороне сервера Application Insights (без Visual Studio)

1. Установите [пакет NuGet для Application Insights SDK для ASP.NET Core](https://nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore). Рекомендуется всегда использовать последнюю стабильную версию. Ознакомьтесь с полными сведениями о выпуске пакета SDK в [репозитории GitHub с открытым исходным кодом](https://github.com/Microsoft/ApplicationInsights-dotnet/releases).

    В следующем примере кода показаны изменения, которые необходимо добавить в `.csproj` файл проекта.

    ```xml
        <ItemGroup>
          <PackageReference Include="Microsoft.ApplicationInsights.AspNetCore" Version="2.16.0" />
        </ItemGroup>
    ```

2. Добавьте в `services.AddApplicationInsightsTelemetry();` `ConfigureServices()` метод в `Startup` классе, как показано в следующем примере:

    ```csharp
        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            // The following line enables Application Insights telemetry collection.
            services.AddApplicationInsightsTelemetry();
    
            // This code adds other services for your application.
            services.AddMvc();
        }
    ```

3. Настройте ключ инструментирования.

    Хотя ключ инструментирования можно указать в качестве аргумента `AddApplicationInsightsTelemetry` , рекомендуется указать ключ инструментирования в конфигурации. В следующем образце кода показано, как указать ключ инструментирования в `appsettings.json` . Убедитесь `appsettings.json` , что во время публикации скопировано в корневую папку приложения.

    ```json
        {
          "ApplicationInsights": {
            "InstrumentationKey": "putinstrumentationkeyhere"
          },
          "Logging": {
            "LogLevel": {
              "Default": "Warning"
            }
          }
        }
    ```

    Кроме того, можно указать ключ инструментирования в любой из следующих переменных среды:

    * `APPINSIGHTS_INSTRUMENTATIONKEY`

    * `ApplicationInsights:InstrumentationKey`

    Пример:

    * `SET ApplicationInsights:InstrumentationKey=putinstrumentationkeyhere`

    * `SET APPINSIGHTS_INSTRUMENTATIONKEY=putinstrumentationkeyhere`

    * `APPINSIGHTS_INSTRUMENTATIONKEY` обычно используется в [веб-приложениях Azure](./azure-web-apps.md?tabs=net), но также может использоваться во всех местах, где поддерживается этот пакет SDK. (Если вы выполняете мониторинг веб-приложений без кода, этот формат необходим, если не используются строки подключения.)

    Вместо настройки ключей инструментирования теперь можно использовать [строки подключения](./sdk-connection-string.md?tabs=net).

    > [!NOTE]
    > Ключ инструментирования, указанный в коде, передается через переменную среды `APPINSIGHTS_INSTRUMENTATIONKEY` , которая WINS поверх других параметров.

### <a name="user-secrets-and-other-configuration-providers"></a>Секреты пользователей и другие поставщики конфигурации

Если вы хотите сохранить ключ инструментирования в ASP.NET Core секреты пользователя или получить его из другого поставщика конфигурации, можно использовать перегрузку с `Microsoft.Extensions.Configuration.IConfiguration` параметром. Например, `services.AddApplicationInsightsTelemetry(Configuration);`.
Начиная с Microsoft. ApplicationInsights. AspNetCore Version [2.15.0](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore), вызов `services.AddApplicationInsightsTelemetry()` будет автоматически считывать ключ инструментирования из `Microsoft.Extensions.Configuration.IConfiguration` приложения. Нет необходимости явно предоставлять `IConfiguration` .

## <a name="run-your-application"></a>Запуск приложения

Запустите приложение и выполните запросы к нему. Теперь данные телеметрии должны передаваться в Application Insights. Пакет SDK для Application Insights автоматически собирает входящие веб-запросы в приложение, а также следующие данные телеметрии.

### <a name="live-metrics"></a>Интерактивные метрики

Для быстрой проверки правильности настройки Application Insights мониторинга можно использовать [динамические метрики](./live-stream.md) . Это может занять несколько минут, прежде чем данные телеметрии будут отображаться на портале и в аналитике, в динамических метриках будет показано использование ЦП запущенного процесса практически в реальном времени. Он также может отображать другие данные телеметрии, такие как запросы, зависимости, трассировки и т. д.

### <a name="ilogger-logs"></a>Журналы ILogger

Конфигурация по умолчанию собирает `ILogger` журналы с уровнем серьезности `Warning` и выше. Эту конфигурацию можно [настроить](#how-do-i-customize-ilogger-logs-collection).

### <a name="dependencies"></a>Зависимости

Коллекция зависимостей включена по умолчанию. В [этой](asp-net-dependencies.md#automatically-tracked-dependencies) статье объясняются автоматически собираемые зависимости, а также инструкции по отслеживанию вручную.

### <a name="performance-counters"></a>Счетчики производительности

Поддержка [счетчиков производительности](./performance-counters.md) в ASP.NET Core ограничена:

* Пакеты SDK версии 2.4.1 и более поздних собирают данные счетчиков производительности, если приложение выполняется в веб-приложениях Azure (Windows).
* Пакеты SDK версии 2.7.1 и более поздних собирают счетчики производительности, если приложение выполняется в Windows и предназначено для `NETSTANDARD2.0` или более поздних версий.
* Для приложений, предназначенных для .NET Framework, все версии пакетов SDK поддерживают счетчики производительности.
* Пакеты SDK версии 2.8.0 и более поздних поддерживают счетчик ЦП/памяти в Linux. В Linux не поддерживаются никакие другие счетчики. Для получения системных счетчиков в Linux (и других средах, отличных от Windows) рекомендуется использовать [EventCounters](#eventcounter).

### <a name="eventcounter"></a>евенткаунтер

`EventCounterCollectionModule` параметр включен по умолчанию. В руководстве по [евенткаунтер](eventcounters.md) приведены инструкции по настройке списка собираемых счетчиков.

## <a name="enable-client-side-telemetry-for-web-applications"></a>Включение телеметрии на стороне клиента для веб-приложений

Предыдущие шаги достаточно, чтобы помочь начать сбор данных телеметрии на стороне сервера. Если приложение имеет клиентские компоненты, выполните следующие действия, чтобы начать сбор данных [телеметрии использования](./usage-overview.md).

1. В `_ViewImports.cshtml` добавьте внедрение:

```cshtml
    @inject Microsoft.ApplicationInsights.AspNetCore.JavaScriptSnippet JavaScriptSnippet
```

2. В `_Layout.cshtml` вставьте в `HtmlHelper` конец `<head>` раздела, но перед любым другим сценарием. Если вы хотите сообщить о любых пользовательских данных телеметрии JavaScript со страницы, вставьте ее после этого фрагмента кода:

```cshtml
    @Html.Raw(JavaScriptSnippet.FullScript)
    </head>
```

Кроме того, вы можете использовать, начиная с версии `FullScript` `ScriptBody` SDK v 2.14. Используйте этот параметр, если необходимо управлять `<script>` тегом, чтобы задать политику безопасности содержимого:

```cshtml
 <script> // apply custom changes to this script tag.
     @Html.Raw(JavaScriptSnippet.ScriptBody)
 </script>
```

`.cshtml`Ранее упомянутые имена файлов относятся к шаблону приложения MVC по умолчанию. В конечном счете, если вы хотите правильно включить наблюдение на стороне клиента для вашего приложения, фрагмент JavaScript должен отображаться в `<head>` разделе каждой страницы приложения, которое необходимо отслеживать. Эту цель можно выполнить для этого шаблона приложения, добавив фрагмент кода JavaScript в `_Layout.cshtml` . 

Если проект не включен `_Layout.cshtml` , можно по-прежнему добавить [наблюдение на стороне клиента](./website-monitoring.md). Это можно сделать, добавив фрагмент кода JavaScript в эквивалентный файл, который управляет `<head>` всеми страницами в приложении. Кроме того, можно добавить фрагмент кода на несколько страниц, но это решение сложно поддерживать, и в большинстве случаев это не рекомендуется.

## <a name="configure-the-application-insights-sdk"></a>Настройка пакета SDK для Application Insights

Можно настроить Application Insights SDK для ASP.NET Core, чтобы изменить конфигурацию по умолчанию. Пользователи Application Insights ASP.NET SDK могут быть знакомы с изменением конфигурации с помощью `ApplicationInsights.config` или путем изменения `TelemetryConfiguration.Active` . Для ASP.NET Core почти все изменения конфигурации выполняются в `ConfigureServices()` методе `Startup.cs` класса, если не указано иное. Дополнительные сведения см. в следующих разделах.

> [!NOTE]
> В ASP.NET Coreных приложениях изменение конфигурации путем изменения `TelemetryConfiguration.Active` не поддерживается.

### <a name="using-applicationinsightsserviceoptions"></a>Использование Аппликатионинсигхтссервицеоптионс

Можно изменить несколько общих параметров, передав `ApplicationInsightsServiceOptions` `AddApplicationInsightsTelemetry` , как показано в следующем примере:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    Microsoft.ApplicationInsights.AspNetCore.Extensions.ApplicationInsightsServiceOptions aiOptions
                = new Microsoft.ApplicationInsights.AspNetCore.Extensions.ApplicationInsightsServiceOptions();
    // Disables adaptive sampling.
    aiOptions.EnableAdaptiveSampling = false;

    // Disables QuickPulse (Live Metrics stream).
    aiOptions.EnableQuickPulseMetricStream = false;
    services.AddApplicationInsightsTelemetry(aiOptions);
}
```

Полный список параметров в `ApplicationInsightsServiceOptions`

|Параметр | Описание | По умолчанию
|---------------|-------|-------
|енаблеперформанцекаунтерколлектионмодуле  | Включить или отключить `PerformanceCounterCollectionModule` | true
|енаблерекуесттраккингтелеметримодуле   | Включить или отключить `RequestTrackingTelemetryModule` | true
|енабливенткаунтерколлектионмодуле   | Включить или отключить `EventCounterCollectionModule` | true
|енабледепенденцитраккингтелеметримодуле   | Включить или отключить `DependencyTrackingTelemetryModule` | true
|енаблеаппсервицешеартбеаттелеметримодуле  |  Включить или отключить `AppServicesHeartbeatTelemetryModule` | true
|енаблеазуреинстанцеметадатателеметримодуле   |  Включить или отключить `AzureInstanceMetadataTelemetryModule` | true
|енаблекуиккпулсеметрикстреам | Включить или отключить функцию Ливеметрикс | true
|енаблеадаптивесамплинг | Включение или отключение адаптивной выборки | true
|енаблехеартбеат | Функция "включить/отключить пульс", которая периодически (по умолчанию составляет 15 минут) отправляет пользовательскую метрику "Хеартбеатстате" со сведениями о среде выполнения, такими как версия .NET, сведения о среде Azure, если применимо, и т. д. | true
|аддаутоколлектедметрицекстрактор | Включите или отключите средство извлечения Аутоколлектедметрикс, которое представляет собой Телеметрипроцессор, который отправляет предварительно агрегированные метрики о запросах и зависимостях перед выполнением выборки. | true
|Рекуестколлектионоптионс. Траккексцептионс | Включение и отключение отчетов о необработанном отслеживании исключений модулем сбора запросов. | false в NETSTANDARD 2.0 (поскольку исключения отправляются с помощью Аппликатионинсигхтслогжерпровидер), в противном случае — значение true.
|енабледиагностикстелеметримодуле | Включить или отключить `DiagnosticsTelemetryModule` . Отключение этого параметра приведет к игнорированию следующих параметров. `EnableHeartbeat`, `EnableAzureInstanceMetadataTelemetryModule`, `EnableAppServicesHeartbeatTelemetryModule` | true

См. список [настраиваемых параметров `ApplicationInsightsServiceOptions` в](https://github.com/microsoft/ApplicationInsights-dotnet/blob/develop/NETCORE/src/Shared/Extensions/ApplicationInsightsServiceOptions.cs) для наиболее актуального списка.

### <a name="configuration-recommendation-for-microsoftapplicationinsightsaspnetcore-sdk-2150--above"></a>Рекомендация по настройке пакета SDK для Microsoft. ApplicationInsights. AspNetCore, 2.15.0 & выше

Начиная с Microsoft. ApplicationInsights. AspNetCore SDK версии [2.15.0](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore/2.15.0), рекомендуется настроить каждый параметр, доступный в `ApplicationInsightsServiceOptions` , включая instrumentationkey с помощью экземпляра приложения `IConfiguration` . Параметры должны быть в разделе "ApplicationInsights", как показано в следующем примере. Следующий раздел из appsettings.jsв разделе Настройка ключа инструментирования, а также отключение адаптивной выборки и сбора счетчиков производительности.

```json
{
    "ApplicationInsights": {
    "InstrumentationKey": "putinstrumentationkeyhere",
    "EnableAdaptiveSampling": false,
    "EnablePerformanceCounterCollectionModule": false
    }
}
```

Если `services.AddApplicationInsightsTelemetry(aiOptions)` используется, он переопределяет параметры из `Microsoft.Extensions.Configuration.IConfiguration` .

### <a name="sampling"></a>Дискретизация

Пакет SDK Application Insights для ASP.NET Core поддерживает как фиксированную, так и адаптивную выборку. Адаптивная выборка включена по умолчанию.

Дополнительные сведения см. в статье [Настройка адаптивной выборки для приложений ASP.NET Core](./sampling.md#configuring-adaptive-sampling-for-aspnet-core-applications).

### <a name="adding-telemetryinitializers"></a>Добавление Telemetryinitializer

Используйте [инициализаторы телеметрии](./api-filtering-sampling.md#addmodify-properties-itelemetryinitializer) , если хотите расширить телеметрию с дополнительной информацией.

Добавьте новые `TelemetryInitializer` в `DependencyInjection` контейнер, как показано в следующем коде. Пакет SDK автоматически берет все `TelemetryInitializer` , что добавляется в `DependencyInjection` контейнер.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<ITelemetryInitializer, MyCustomTelemetryInitializer>();
}
```

> [!NOTE]
> `services.AddSingleton<ITelemetryInitializer, MyCustomTelemetryInitializer>();` работает с простыми инициализаторами. Для других требуется следующее: `services.AddSingleton(new MyCustomTelemetryInitializer() { fieldName = "myfieldName" });`
    
### <a name="removing-telemetryinitializers"></a>Удаление Telemetryinitializer

Инициализаторы телеметрии представлены по умолчанию. Чтобы удалить все или конкретные инициализаторы телеметрии, используйте следующий пример кода *после* вызова метода `AddApplicationInsightsTelemetry()` .

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddApplicationInsightsTelemetry();

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

Можно добавить пользовательские обработчики данных телеметрии в с `TelemetryConfiguration` помощью метода расширения `AddApplicationInsightsTelemetryProcessor` в `IServiceCollection` . В [сценариях расширенной фильтрации](./api-filtering-sampling.md#itelemetryprocessor-and-itelemetryinitializer)используются обработчики данных телеметрии. Используйте следующий пример.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    services.AddApplicationInsightsTelemetry();
    services.AddApplicationInsightsTelemetryProcessor<MyFirstCustomTelemetryProcessor>();

    // If you have more processors:
    services.AddApplicationInsightsTelemetryProcessor<MySecondCustomTelemetryProcessor>();
}
```

### <a name="configuring-or-removing-default-telemetrymodules"></a>Настройка или удаление Телеметримодулес по умолчанию

Application Insights использует модули телеметрии для автоматического сбора полезных данных телеметрии о конкретных рабочих нагрузках, не требуя ручного отслеживания пользователем.

Следующие модули автоматической коллекции включены по умолчанию. Эти модули отвечают за автоматическое сбор данных телеметрии. Их можно отключить или настроить для изменения их поведения по умолчанию.

* `RequestTrackingTelemetryModule` — Собирает RequestTelemetry из входящих веб-запросов.
* `DependencyTrackingTelemetryModule` — Собирает [DependencyTelemetry](./asp-net-dependencies.md) из исходящих вызовов HTTP и вызовов SQL.
* `PerformanceCollectorModule` — Собирает данные счетчиков производительности Windows.
* `QuickPulseTelemetryModule` — Собирает данные телеметрии для отображения на портале динамических метрик.
* `AppServicesHeartbeatTelemetryModule` — Собирает неритми сердца (которые отправляются как пользовательские метрики), сведения о среде службы приложений Azure, в которой размещается приложение.
* `AzureInstanceMetadataTelemetryModule` — Собирает неритми сердца (которые отправляются в виде пользовательских метрик), о среде виртуальной машины Azure, в которой размещено приложение.
* `EventCounterCollectionModule` — Собирает [евенткаунтерс.](eventcounters.md) Этот модуль является новой функцией и доступен в пакете SDK версии 2.8.0 и выше.

Чтобы настроить любое значение по умолчанию `TelemetryModule` , используйте метод расширения `ConfigureTelemetryModule<T>` для `IServiceCollection` , как показано в следующем примере.

```csharp
using Microsoft.ApplicationInsights.DependencyCollector;
using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector;

public void ConfigureServices(IServiceCollection services)
{
    services.AddApplicationInsightsTelemetry();

    // The following configures DependencyTrackingTelemetryModule.
    // Similarly, any other default modules can be configured.
    services.ConfigureTelemetryModule<DependencyTrackingTelemetryModule>((module, o) =>
            {
                module.EnableW3CHeadersInjection = true;
            });

    // The following removes all default counters from EventCounterCollectionModule, and adds a single one.
    services.ConfigureTelemetryModule<EventCounterCollectionModule>(
            (module, o) =>
            {
                module.Counters.Add(new EventCounterCollectionRequest("System.Runtime", "gen-0-size"));
            }
        );

    // The following removes PerformanceCollectorModule to disable perf-counter collection.
    // Similarly, any other default modules can be removed.
    var performanceCounterService = services.FirstOrDefault<ServiceDescriptor>(t => t.ImplementationType == typeof(PerformanceCollectorModule));
    if (performanceCounterService != null)
    {
        services.Remove(performanceCounterService);
    }
}
```

Начиная с версии 2.12.2, [`ApplicationInsightsServiceOptions`](#using-applicationinsightsserviceoptions) содержит простой способ отключения всех модулей по умолчанию.

### <a name="configuring-a-telemetry-channel"></a>Настройка канала телеметрии

[Каналом телеметрии](./telemetry-channels.md) по умолчанию является `ServerTelemetryChannel` . Его можно переопределить, как показано в следующем примере.

```csharp
using Microsoft.ApplicationInsights.Channel;

    public void ConfigureServices(IServiceCollection services)
    {
        // Use the following to replace the default channel with InMemoryChannel.
        // This can also be applied to ServerTelemetryChannel.
        services.AddSingleton(typeof(ITelemetryChannel), new InMemoryChannel() {MaxTelemetryBufferCapacity = 19898 });

        services.AddApplicationInsightsTelemetry();
    }
```

### <a name="disable-telemetry-dynamically"></a>Динамическое отключение телеметрии

Если вы хотите отключить данные телеметрии условно и динамически, вы можете разрешить `TelemetryConfiguration` экземпляр с контейнером ASP.NET Core внедрения зависимостей в любом месте кода и установить `DisableTelemetry` для него флаг.

```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddApplicationInsightsTelemetry();
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env, TelemetryConfiguration configuration)
    {
        configuration.DisableTelemetry = true;
        ...
    }
```

Указанные выше данные не предотвращают сбор данных телеметрии для модулей автоматического сбора данных. С помощью описанного выше подхода отключается только отправка данных телеметрии в Application Insights. Если не требуется определенный модуль автоматической коллекции, рекомендуется [удалить модуль телеметрии](#configuring-or-removing-default-telemetrymodules) .

## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

### <a name="does-application-insights-support-aspnet-core-3x"></a>Поддерживает ли Application Insights ASP.NET Core 3. X?

Да. Обновите [пакет SDK для Application Insights для ASP.NET Core](https://nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore) версии 2.8.0 или более поздней. Более старые версии пакета SDK не поддерживают ASP.NET Core 3. X.

Кроме того, если вы используете инструкции на основе Visual Studio [отсюда,](#enable-application-insights-server-side-telemetry-visual-studio)обновите ее до последней версии Visual Studio 2019 (16.3.0). Предыдущие версии Visual Studio не поддерживают автоматическую адаптации для приложений ASP.NET Core 3. X.

### <a name="how-can-i-track-telemetry-thats-not-automatically-collected"></a>Как можно отвестить данные телеметрии, которые не собираются автоматически?

Получите экземпляр `TelemetryClient` с помощью внедрения конструктора и вызовите `TrackXXX()` для него необходимый метод. Не рекомендуется создавать новые `TelemetryClient` экземпляры или `TelemetryConfiguration` в приложении ASP.NET Core. Одноэлементный экземпляр `TelemetryClient` уже зарегистрирован в `DependencyInjection` контейнере, который совместно `TelemetryConfiguration` используется с остальными данными телеметрии. Создание нового `TelemetryClient` экземпляра рекомендуется, только если требуется конфигурация, отделяющая остальные данные телеметрии.

В следующем примере показано, как отслеживанию дополнительных данных телеметрии из контроллера.

```csharp
using Microsoft.ApplicationInsights;

public class HomeController : Controller
{
    private TelemetryClient telemetry;

    // Use constructor injection to get a TelemetryClient instance.
    public HomeController(TelemetryClient telemetry)
    {
        this.telemetry = telemetry;
    }

    public IActionResult Index()
    {
        // Call the required TrackXXX method.
        this.telemetry.TrackEvent("HomePageRequested");
        return View();
    }
```

Дополнительные сведения о настраиваемых отчетах данных в Application Insights см. в разделе [Справочник по API настраиваемых метрик Application Insights](./api-custom-events-metrics.md). Аналогичный подход можно использовать для отправки пользовательских метрик на Application Insights с помощью API- [интерфейса](./get-metric.md).

### <a name="how-do-i-customize-ilogger-logs-collection"></a>Разделы справки настроить сбор журналов ILogger?

По умолчанию автоматически фиксируются только журналы с уровнем серьезности `Warning` и выше. Чтобы изменить это поведение, явно Переопределите конфигурацию ведения журнала для поставщика, `ApplicationInsights` как показано ниже.
Следующая конфигурация позволяет ApplicationInsights записывать все журналы серьезности `Information` и выше.

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    },
    "ApplicationInsights": {
      "LogLevel": {
        "Default": "Information"
      }
    }
  }
}
```

Важно отметить, что следующее не приведет к тому, что поставщик ApplicationInsights будет записывать `Information` журналы. Это связано с тем, что пакет SDK добавляет фильтр ведения журнала по умолчанию, `ApplicationInsights` который указывает на захват только `Warning` и более поздних версий. По этой причине для ApplicationInsights требуется явное переопределение.

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  }
}
```

Подробнее о [конфигурации ILogger](ilogger.md#control-logging-level).

### <a name="some-visual-studio-templates-used-the-useapplicationinsights-extension-method-on-iwebhostbuilder-to-enable-application-insights-is-this-usage-still-valid"></a>Некоторые шаблоны Visual Studio использовали метод расширения UseApplicationInsights () в Ивебхостбуилдер для включения Application Insights. Является ли это использование допустимым?

Хотя метод расширения `UseApplicationInsights()` по-прежнему поддерживается, он отмечается как устаревший в Application Insights SDK версии 2.8.0. Он будет удален в следующей основной версии пакета SDK. Для включения телеметрии Application Insights рекомендуется использовать, `AddApplicationInsightsTelemetry()` так как он предоставляет перегрузки для управления некоторой конфигурацией. Кроме того, в ASP.NET Core 3. X приложений `services.AddApplicationInsightsTelemetry()` — единственный способ включить Application Insights.

### <a name="im-deploying-my-aspnet-core-application-to-web-apps-should-i-still-enable-the-application-insights-extension-from-web-apps"></a>Я развертываю приложение ASP.NET Core в веб-приложениях. Следует ли по-прежнему включать расширение Application Insights из веб-приложений?

Если пакет SDK устанавливается во время сборки, как показано в этой статье, вам не нужно включать [расширение Application Insights](./azure-web-apps.md) на портале службы приложений. Даже если расширение установлено, оно будет восстановлено при обнаружении того, что пакет SDK уже добавлен в приложение. Если включить Application Insights из расширения, вам не нужно устанавливать и обновлять пакет SDK. Но если вы включили Application Insights, следуя инструкциям в этой статье, вы получите большую гибкость, поскольку:

   * Данные телеметрии Application Insights будут продолжать работать в:
       * Все операционные системы, включая Windows, Linux и Mac.
       * Все режимы публикации, включая автономные или зависящие от платформы.
       * Все целевые платформы, включая полный платформа .NET Framework.
       * Все варианты размещения, включая веб-приложения, виртуальные машины, Linux, контейнеры, службу Azure Kubernetes и размещение без Azure.
       * Все версии .NET Core, включая предварительные версии.
   * Вы можете просматривать данные телеметрии локально при отладке из Visual Studio.
   * Вы можете отвести отслеживание дополнительных пользовательских данных телеметрии с помощью `TrackXXX()` API.
   * Вы полностью контролируете конфигурацию.

### <a name="can-i-enable-application-insights-monitoring-by-using-tools-like-status-monitor"></a>Можно ли включить Application Insights мониторинг с помощью таких средств, как монитор состояния?

Нет. В настоящее время [Монитор состояния](./monitor-performance-live-website-now.md) и [Монитор состояния v2](./status-monitor-v2-overview.md) поддерживают только ASP.NET 4. x.

### <a name="if-i-run-my-application-in-linux-are-all-features-supported"></a>Если я запускаю приложение в Linux, поддерживаются ли все функции?

Да. Поддержка функций для пакета SDK одинакова на всех платформах, за исключением следующих:

* Пакет SDK собирает [счетчики событий](./eventcounters.md) в Linux, так как [счетчики производительности](./performance-counters.md) поддерживаются только в Windows. Большинство метрик одинаковы.
* Несмотря на то что `ServerTelemetryChannel` включен по умолчанию, если приложение выполняется в Linux или macOS, канал не создает папку локального хранилища, чтобы временно сохранять данные телеметрии в случае проблем с сетью. Из-за этого ограничения данные телеметрии теряются при наличии временных проблем с сетью или сервером. Чтобы обойти эту ошибку, настройте локальную папку для канала:

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
        services.AddApplicationInsightsTelemetry();
    }
```

Это ограничение неприменимо в [2.15.0](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore/2.15.0) и более новых версиях.

### <a name="is-this-sdk-supported-for-the-new-net-core-3x-worker-service-template-applications"></a>Поддерживается ли этот пакет SDK для новых приложений-шаблонов рабочей службы .NET Core 3. X?

Этот пакет SDK требует `HttpContext` , и, следовательно, не работает в приложениях, не использующих протокол HTTP, включая приложения рабочей службы .NET Core 3. X. Ознакомьтесь с [этим](worker-service.md) документом, чтобы включить Application Insights в таких приложениях с помощью ВЫПУЩЕННОГО пакета SDK для Microsoft. ApplicationInsights. воркерсервице.

## <a name="open-source-sdk"></a>Пакет SDK с открытым исходным кодом

* [Чтение кода и внесение в него вклада](https://github.com/microsoft/ApplicationInsights-dotnet).

Последние обновления и исправления ошибок см. [в заметках о выпуске](./release-notes.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Изучите потоки пользователей](./usage-flows.md) , чтобы понять, как пользователи переходят через приложение.
* [Настройте сбор моментальных снимков](./snapshot-debugger.md) для просмотра состояния исходного кода и переменных в момент возникновения исключения.
* [Используйте API](./api-custom-events-metrics.md) для отправки собственных событий и метрик для получения подробного представления о производительности и использовании приложения.
* Используйте [тесты доступности](./monitor-web-app-availability.md), чтобы постоянно проверять работу приложения из всех точек мира.
* [Использование внедрения зависимостей в ASP.NET Core](/aspnet/core/fundamentals/dependency-injection)
