---
title: Выборка данных телеметрии в Azure Application Insights | Документация Майкрософт
description: Как управлять объемом данных телеметрии.
ms.topic: conceptual
ms.date: 01/17/2020
ms.reviewer: vitalyg
ms.custom: fasttrack-edit
ms.openlocfilehash: 7b53b0bc8c7cc3df2123d327bf87a85081f88f50
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100589556"
---
# <a name="sampling-in-application-insights"></a>Выборка в Application Insights

Выборка — это функция [Azure Application Insights](./app-insights-overview.md), Это рекомендуемый способ снизить объем данных телеметрии, затраты на данные и затраты на хранение, сохраняя статистически верный анализ данных приложения. Выборка также помогает избежать Application Insights регулирования телеметрии. Фильтр выборки выбирает связанные элементы, чтобы можно было перемещаться между элементами при выполнении диагностического исследования.

Если счетчики метрик представлены на портале, они будут повторно нормализованы для использования выборки учетных записей. Это снизит влияние на статистику.

## <a name="brief-summary"></a>Краткая сводка

* Существует три разных типа выборки: Адаптивная выборка, выборка с фиксированной частотой и выборка приема.
* Адаптивная выборка включена по умолчанию во всех последних версиях Application Insights ASP.NET и ASP.NET Core пакетах средств разработки программного обеспечения (SDK). Она также используется в [функциях Azure](../../azure-functions/functions-overview.md).
* Выборка с фиксированной частотой доступна в последних версиях Application Insights пакетов SDK для ASP.NET, ASP.NET Core, Java (агент и пакет SDK) и Python.
* Выборка приема работает на конечной точке службы Application Insights. Он применяется только в том случае, если никакая другая выборка не действует. Если пакет SDK производит выборку данных телеметрии, выборка приема отключена.
* Для веб-приложений, если вы зарегистрируете пользовательские события и хотите убедиться, что набор событий сохраняется или удаляется вместе, события должны иметь одно и то же `OperationId` значение.
* При написании запросов аналитики необходимо [учитывать выборку](/azure/data-explorer/kusto/query/samples?&pivots=azuremonitor#aggregations). В частности, вместо простого подсчета записей следует использовать функцию `summarize sum(itemCount)`.
* Некоторые типы телеметрии, включая метрики производительности и пользовательские метрики, всегда сохраняются независимо от того, включена ли выборка.

В следующей таблице перечислены типы выборок, доступные для каждого пакета SDK и типа приложения.

| Пакет SDK для Application Insights | Поддерживается Адаптивная выборка | Поддерживается выборка с фиксированной частотой | Поддерживается выборка приема |
|-|-|-|-|
| ASP.NET | [Да (включено по умолчанию)](#configuring-adaptive-sampling-for-aspnet-applications) | [Да](#configuring-fixed-rate-sampling-for-aspnet-applications) | Только если не действует ни одна другая выборка |
| ASP.NET Core | [Да (включено по умолчанию)](#configuring-adaptive-sampling-for-aspnet-core-applications) | [Да](#configuring-fixed-rate-sampling-for-aspnet-core-applications) | Только если не действует ни одна другая выборка |
| Функции Azure | [Да (включено по умолчанию)](#configuring-adaptive-sampling-for-azure-functions) | Нет | Только если не действует ни одна другая выборка |
| Java | Нет | [Да](#configuring-fixed-rate-sampling-for-java-applications) | Только если не действует ни одна другая выборка |
| Node.JS | Нет | [Да](./nodejs.md#sampling) | Только если не действует ни одна другая выборка
| Python | Нет | [Да](#configuring-fixed-rate-sampling-for-opencensus-python-applications) | Только если не действует ни одна другая выборка |
| Все остальные | Нет | Нет | [Да](#ingestion-sampling) |

> [!NOTE]
> Сведения, приведенные на этой странице, относятся к текущим версиям пакетов SDK для Application Insights. Сведения о более ранних версиях пакетов SDK [см. в разделе ниже](#older-sdk-versions).

## <a name="types-of-sampling"></a>Типы выборки

Существует три разных метода выборки:

* **Адаптивная выборка** автоматически корректирует объем данных телеметрии, отправляемых из пакета SDK в приложении ASP.NET/ASP.NET Core, а также из функций Azure. Это выборка по умолчанию при использовании пакета SDK ASP.NET или ASP.NET Core. Адаптивная выборка сейчас доступна только для данных телеметрии на стороне сервера ASP.NET и для функций Azure.

* **Выборка с фиксированной частотой** сокращает объем данных телеметрии, отправляемых с сервера ASP.NET или ASP.NET Core или Java, а также из браузеров пользователей. Частоту вы задаете сами. Благодаря тому, что клиент и сервер будут синхронизировать свои выборки, при поиске вы сможете перемещаться между связанными представлениями страниц и запросами.

* **Выборка приема** происходит в конечной точке службы Application Insights. отклоняет некоторые данные телеметрии, поступающие из приложения, с заданной скоростью выборки. Выборка не сокращает трафик телеметрии, отправляемый из вашего приложения, но помогает оставаться в пределах месячной квоты. Основное преимущество выборки приема заключается в том, что можно задать частоту выборки без повторного развертывания приложения. Выборка приема действует единообразно для всех серверов и клиентов, но она не применяется при выполнении любых других типов выборки.

> [!IMPORTANT]
> Если для типа телеметрии включены адаптивные или фиксированные методы выборки частоты, выборка приема для этой телеметрии отключается. Однако типы телеметрии, исключенные из выборки на уровне пакета SDK, будут по-прежнему подвергаться выборки приема на основе скорости, установленной на портале.

## <a name="adaptive-sampling"></a>Адаптивная выборка

Адаптивная выборка влияет на объем данных телеметрии, отправляемых в конечную точку службы Application Insights из приложения веб-сервера.

> [!TIP]
> Адаптивная выборка включается по умолчанию при использовании пакета SDK для ASP.NET или пакета SDK для ASP.NET Core и также включается по умолчанию для функций Azure.

Том корректируется автоматически, чтобы он оставался в пределах заданного максимального уровня трафика, и управляется с помощью параметра `MaxTelemetryItemsPerSecond` . Если приложение создает небольшой объем данных телеметрии, например при отладке или из-за низкого уровня использования, элементы не будут удаляться процессором выборки, пока используется том ниже `MaxTelemetryItemsPerSecond` . По мере увеличения объема данных телеметрии частота дискретизации корректируется, чтобы получить целевой том. Корректировка пересчитывается через регулярные интервалы времени в зависимости от скользящего среднего для скорости исходящей передачи.

Для получения целевого объема некоторые формируемые данные телеметрии игнорируются. Однако, как и в других типах выборки, алгоритм сохраняет связанные элементы телеметрии. Например, при проверке телеметрии в поиске это позволит найти запрос, связанный с определенным исключением.

Счетчики метрик, например частота запросов и частота исключений, корректируются с учетом частоты выборки, так что в обозревателе метрик отображаются приблизительно точные значения.

### <a name="configuring-adaptive-sampling-for-aspnet-applications"></a>Настройка адаптивной выборки для приложений ASP.NET

> [!NOTE]
> Этот раздел относится к приложениям ASP.NET, а не ASP.NET Core приложениям. [Дополнительные сведения о настройке адаптивной выборки для ASP.NET Core приложений далее в этом документе.](#configuring-adaptive-sampling-for-aspnet-core-applications)

В [`ApplicationInsights.config`](./configuration-with-applicationinsights-config.md) можно настроить несколько параметров в `AdaptiveSamplingTelemetryProcessor` узле. Ниже представлены значения по умолчанию.

* `<MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>`
  
    Целевая скорость [логических операций](./correlation.md#data-model-for-telemetry-correlation) , которую должен собираются адаптивный алгоритм для накопления **на каждом узле сервера**. Если веб-приложение выполняется на множестве узлов, нужно уменьшить это значение, чтобы не превышать целевую скорость трафика на портале Application Insights.

* `<EvaluationInterval>00:00:15</EvaluationInterval>` 
  
    Интервал повторного вычисления текущей скорости телеметрии. Вычисление выполняется на основе скользящего среднего. Вы можете сократить этот интервал, если наблюдаете неожиданные скачки данных телеметрии в сторону увеличения.

* `<SamplingPercentageDecreaseTimeout>00:02:00</SamplingPercentageDecreaseTimeout>`
  
    Когда изменяется процент выборки, как только после этого можно будет уменьшить процент выборки для сбора меньшего объема данных?

* `<SamplingPercentageIncreaseTimeout>00:15:00</SamplingPercentageIncreaseTimeout>`
  
    Когда изменяется процент выборки, как только после этого можно будет увеличить процент выборки для сбора дополнительных данных?

* `<MinSamplingPercentage>0.1</MinSamplingPercentage>`
  
    Каково минимальное значение, которое может быть задано в процентах выборки?

* `<MaxSamplingPercentage>100.0</MaxSamplingPercentage>`
  
    Каково максимальное значение, которое может быть задано в процентах выборки?

* `<MovingAverageRatio>0.25</MovingAverageRatio>` 
  
    При вычислении скользящего среднего это указывает вес, который должен быть назначен последнему значению. Используйте значение не больше 1. Использование значений ниже рекомендуемых приводит к тому, что скорость реагирования алгоритма на резкие изменения замедляется.

* `<InitialSamplingPercentage>100</InitialSamplingPercentage>`
  
    Объем данных телеметрии для выборки, когда приложение было только что запущено. Не уменьшайте это значение во время отладки.

* `<ExcludedTypes>Trace;Exception</ExcludedTypes>`
  
    Список типов, разделенных точкой с запятой, которые не должны подвергаться выборки. Распознаваемые типы: `Dependency` , `Event` , `Exception` , `PageView` , `Request` , `Trace` . Передаются все данные телеметрии указанных типов. Выбор типов, которые не указаны, будет выполняться.

* `<IncludedTypes>Request;Dependency</IncludedTypes>`
  
    Разделенный точками с запятой список типов, для которых требуется выборка. Распознаваемые типы: `Dependency` , `Event` , `Exception` , `PageView` , `Request` , `Trace` . Указанные типы будут выдаваться по образцу. все данные телеметрии других типов всегда будут передаваться.

**Чтобы отключить** адаптивную выборку, удалите `AdaptiveSamplingTelemetryProcessor` узлы из `ApplicationInsights.config` .

#### <a name="alternative-configure-adaptive-sampling-in-code"></a>Альтернатива: Настройка адаптивной выборки в коде

Вместо установки параметра выборки в `.config` файле можно программно задать эти значения.

1. Удалите `AdaptiveSamplingTelemetryProcessor` из файла все узлы `.config` .
2. Чтобы настроить адаптивную выборку, используйте следующий фрагмент кода:

    ```csharp
    using Microsoft.ApplicationInsights;
    using Microsoft.ApplicationInsights.Extensibility;
    using Microsoft.ApplicationInsights.WindowsServer.Channel.Implementation;
    using Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel;
    
    // ...

    var builder = TelemetryConfiguration.Active.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
    // For older versions of the Application Insights SDK, use the following line instead:
    // var builder = TelemetryConfiguration.Active.TelemetryProcessorChainBuilder;

    // Enable AdaptiveSampling so as to keep overall telemetry volume to 5 items per second.
    builder.UseAdaptiveSampling(maxTelemetryItemsPerSecond:5);

    // If you have other telemetry processors:
    builder.Use((next) => new AnotherProcessor(next));

    builder.Build();
    ```

    ([Сведения о процессорах телеметрии](./api-filtering-sampling.md#filtering).)

Можно также настроить частоту выборки для каждого типа телеметрии отдельно или даже исключить определенные типы из выборки.

```csharp
// The following configures adaptive sampling with 5 items per second, and also excludes Dependency telemetry from being subjected to sampling.
builder.UseAdaptiveSampling(maxTelemetryItemsPerSecond:5, excludedTypes: "Dependency");
```

### <a name="configuring-adaptive-sampling-for-aspnet-core-applications"></a>Настройка адаптивной выборки для приложений ASP.NET Core

`ApplicationInsights.config`Для ASP.NET Core приложений нет, поэтому все настройки выполняются с помощью кода.
Адаптивная выборка включена для всех приложений ASP.NET Core по умолчанию. Вы можете отключить или настроить поведение выборки.

#### <a name="turning-off-adaptive-sampling"></a>Отключение адаптивной выборки

Функцию выборки по умолчанию можно отключить при добавлении службы Application Insights в методе `ConfigureServices` , используя `ApplicationInsightsServiceOptions` в `Startup.cs` файле:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    var aiOptions = new Microsoft.ApplicationInsights.AspNetCore.Extensions.ApplicationInsightsServiceOptions();
    aiOptions.EnableAdaptiveSampling = false;
    services.AddApplicationInsightsTelemetry(aiOptions);

    //...
}
```

Приведенный выше код отключит адаптивную выборку. Чтобы добавить выборку с дополнительными параметрами настройки, выполните следующие действия.

#### <a name="configure-sampling-settings"></a>Настройка параметров выборки

Чтобы настроить поведение выборки, используйте методы расширения `TelemetryProcessorChainBuilder`, как показано ниже.

> [!IMPORTANT]
> Если вы используете этот метод для настройки выборки, обязательно присвойте `aiOptions.EnableAdaptiveSampling` свойству значение `false` при вызове `AddApplicationInsightsTelemetry()` .

```csharp
using Microsoft.ApplicationInsights.Extensibility

public void Configure(IApplicationBuilder app, IHostingEnvironment env, TelemetryConfiguration configuration)
{
    var builder = configuration.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
    // For older versions of the Application Insights SDK, use the following line instead:
    // var builder = configuration.TelemetryProcessorChainBuilder;

    // Using adaptive sampling
    builder.UseAdaptiveSampling(maxTelemetryItemsPerSecond:5);

    // Alternately, the following configures adaptive sampling with 5 items per second, and also excludes DependencyTelemetry from being subject to sampling.
    // builder.UseAdaptiveSampling(maxTelemetryItemsPerSecond:5, excludedTypes: "Dependency");

    // If you have other telemetry processors:
    builder.Use((next) => new AnotherProcessor(next));
    
    builder.Build();

    // ...
}
```

### <a name="configuring-adaptive-sampling-for-azure-functions"></a>Настройка адаптивной выборки для функций Azure

Следуйте инструкциям на [этой странице](../../azure-functions/configure-monitoring.md#configure-sampling) , чтобы настроить адаптивную выборку для приложений, выполняющихся в функциях Azure.

## <a name="fixed-rate-sampling"></a>Выборка с фиксированной частотой

Выборка с фиксированной частотой уменьшает трафик, отправляемый с веб-сервера и веб-браузеров. В отличие от адаптивной выборки объем данных уменьшается в фиксированном, заданном вами размере. Выборка с фиксированной частотой доступна для приложений ASP.NET, ASP.NET Core, Java и Python.

Как и другие методы выборки, это также позволяет хранить связанные элементы. Он также синхронизирует выборку клиента и сервера, чтобы обеспечить сохранность связанных элементов. Например, при просмотре представления страницы в области поиска можно найти соответствующие запросы к серверу. 

В обозревателе метрик такие параметры, как счетчики запросов и исключений, умножаются на коэффициент, компенсирующий частоту выборки, что позволяет получать примерно точные значения.

### <a name="configuring-fixed-rate-sampling-for-aspnet-applications"></a>Настройка выборки с фиксированной частотой для приложений ASP.NET

1. **Отключите адаптивную выборку**: в [`ApplicationInsights.config`](./configuration-with-applicationinsights-config.md) , удалите или закомментируйте `AdaptiveSamplingTelemetryProcessor` узел.

    ```xml
    <TelemetryProcessors>
        <!-- Disabled adaptive sampling:
        <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.AdaptiveSamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
            <MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>
        </Add>
        -->
    ```

2. **Включите модуль выборки с фиксированной частотой.** Добавьте следующий фрагмент кода [`ApplicationInsights.config`](./configuration-with-applicationinsights-config.md) :
   
    ```XML
    <TelemetryProcessors>
        <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.SamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
            <!-- Set a percentage close to 100/N where N is an integer. -->
            <!-- E.g. 50 (=100/2), 33.33 (=100/3), 25 (=100/4), 20, 1 (=100/100), 0.1 (=100/1000) -->
            <SamplingPercentage>10</SamplingPercentage>
        </Add>
    </TelemetryProcessors>
    ```

      Кроме того, вместо настройки параметра выборки в `ApplicationInsights.config` файле можно программно задать следующие значения:

    ```csharp
    using Microsoft.ApplicationInsights.Extensibility;
    using Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel;

    // ...

    var builder = TelemetryConfiguration.Active.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
    // For older versions of the Application Insights SDK, use the following line instead:
    // var builder = TelemetryConfiguration.Active.TelemetryProcessorChainBuilder;

    builder.UseSampling(10.0); // percentage

    // If you have other telemetry processors:
    builder.Use((next) => new AnotherProcessor(next));

    builder.Build();
    ```

    ([Сведения о процессорах телеметрии](./api-filtering-sampling.md#filtering).)

### <a name="configuring-fixed-rate-sampling-for-aspnet-core-applications"></a>Настройка выборки с фиксированной частотой для приложений ASP.NET Core

1. **Отключить адаптивную выборку**: изменения можно вносить в `ConfigureServices` метод с помощью `ApplicationInsightsServiceOptions` :

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        // ...

        var aiOptions = new Microsoft.ApplicationInsights.AspNetCore.Extensions.ApplicationInsightsServiceOptions();
        aiOptions.EnableAdaptiveSampling = false;
        services.AddApplicationInsightsTelemetry(aiOptions);

        //...
    }
    ```

2. **Включите модуль выборки с фиксированной частотой.** Изменения можно вносить в метод, `Configure` как показано в следующем фрагменте кода:

    ```csharp
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        var configuration = app.ApplicationServices.GetService<TelemetryConfiguration>();

        var builder = configuration.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
        // For older versions of the Application Insights SDK, use the following line instead:
        // var builder = configuration.TelemetryProcessorChainBuilder;

        // Using fixed rate sampling
        double fixedSamplingPercentage = 10;
        builder.UseSampling(fixedSamplingPercentage);

        builder.Build();

        // ...
    }
    ```

### <a name="configuring-fixed-rate-sampling-for-java-applications"></a>Настройка выборки с фиксированной частотой для приложений Java

По умолчанию в агенте и пакете Java не включена выборка. В настоящее время поддерживается только фиксированный выбор скорости. Адаптивная выборка не поддерживается в Java.

#### <a name="configuring-java-agent"></a>Настройка агента Java

1. Скачайте [аппликатионинсигхтс-ажент-3.0.0-превиев. 5. jar](https://github.com/microsoft/ApplicationInsights-Java/releases/download/3.0.0-PREVIEW.5/applicationinsights-agent-3.0.0-PREVIEW.5.jar)

1. Чтобы включить выборку, добавьте в `applicationinsights.json` файл следующее:

```json
{
  "sampling": {
    "percentage": 10 //this is just an example that shows you how to enable only only 10% of transaction 
  }
}
```

#### <a name="configuring-java-sdk"></a>Настройка пакета SDK для Java

1. Скачайте и настройте веб-приложение с помощью последнего [пакета SDK для Application Insights Java](./java-get-started.md).

2. **Включите модуль выборки с фиксированной частотой** , добавив следующий фрагмент кода в `ApplicationInsights.xml` файл:

    ```XML
    <TelemetryProcessors>
        <BuiltInProcessors>
            <Processor type="FixedRateSamplingTelemetryProcessor">
                <!-- Set a percentage close to 100/N where N is an integer. -->
                <!-- E.g. 50 (=100/2), 33.33 (=100/3), 25 (=100/4), 20, 1 (=100/100), 0.1 (=100/1000) -->
                <Add name="SamplingPercentage" value="50" />
            </Processor>
        </BuiltInProcessors>
    </TelemetryProcessors>
    ```

3. Вы можете включить или исключить определенные типы данных телеметрии из выборки, используя следующие теги внутри `Processor` тега `FixedRateSamplingTelemetryProcessor` :
   
    ```XML
    <ExcludedTypes>
        <ExcludedType>Request</ExcludedType>
    </ExcludedTypes>

    <IncludedTypes>
        <IncludedType>Exception</IncludedType>
    </IncludedTypes>
    ```

Типы телеметрии, которые можно включать в выборку или исключить из нее: `Dependency` , `Event` ,,, `Exception` `PageView` `Request` и `Trace` .

> [!NOTE]
> В качестве процента выборки выберите значение в процентах, близкое к 100/N, где N — это целое число.  В настоящее время выборка не поддерживает другие значения.

### <a name="configuring-fixed-rate-sampling-for-opencensus-python-applications"></a>Настройка выборки с фиксированной частотой для приложений Python Опенценсус

Выполните инструментирование приложения с помощью последних средств [экспорта опенценсус Azure Monitor](./opencensus-python.md).

> [!NOTE]
> Выборка с фиксированной частотой недоступна для программы экспорта метрик. Это означает, что пользовательские метрики являются единственными типами телеметрии, в которых невозможно настроить выборку. Средство экспорта метрик будет отсылать все данные телеметрии, которые он отслеживает.

#### <a name="fixed-rate-sampling-for-tracing"></a>Выборка с фиксированной частотой для трассировки ####
Вы можете указать `sampler` как часть конфигурации `Tracer`. Если явный выборка не предоставляется, по `ProbabilitySampler` умолчанию будет использоваться. По `ProbabilitySampler` умолчанию будет использоваться частота 1/10000, то есть все запросы 10000 будут отправляться в Application Insights. Чтобы указать свою частоту выборки, см. ниже.

Чтобы указать частоту выборки, убедитесь, что вы `Tracer` указываете выборку с частотой дискретизации от 0,0 до 1,0 включительно. Частота выборки 1,0 представляет 100%, то есть все запросы будут отправляться как данные телеметрии для Application Insights.

```python
tracer = Tracer(
    exporter=AzureExporter(
        instrumentation_key='00000000-0000-0000-0000-000000000000',
    ),
    sampler=ProbabilitySampler(1.0),
)
```

#### <a name="fixed-rate-sampling-for-logs"></a>Выборка с фиксированной частотой для журналов ####
Можно настроить выборку с фиксированной частотой для `AzureLogHandler` , изменив `logging_sampling_rate` необязательный аргумент. Если аргумент не указан, будет использоваться частота выборки, равная 1,0. Частота выборки 1,0 представляет 100%, то есть все запросы будут отправляться как данные телеметрии для Application Insights.

```python
handler = AzureLogHandler(
    instrumentation_key='00000000-0000-0000-0000-000000000000',
    logging_sampling_rate=0.5,
)
```

### <a name="configuring-fixed-rate-sampling-for-web-pages-with-javascript"></a>Настройка выборки с фиксированной частотой для веб-страниц с помощью JavaScript

Веб-страницы на основе JavaScript можно настроить для использования Application Insights. Данные телеметрии отправляются из клиентского приложения, работающего в браузере пользователя, и страницы могут размещаться с любого сервера.

При [настройке веб-страниц на основе JavaScript для Application Insights](javascript.md)измените фрагмент кода JavaScript, полученный на портале Application Insights.

> [!TIP]
> В приложениях ASP.NET с включением JavaScript фрагмент обычно находится в `_Layout.cshtml` .

Вставьте строку, похожую на `samplingPercentage: 10,`, перед ключом инструментирования:

```xml
<script>
    var appInsights = // ... 
    ({ 
      // Value must be 100/N where N is an integer.
      // Valid examples: 50, 25, 20, 10, 5, 1, 0.1, ...
      samplingPercentage: 10, 

      instrumentationKey: ...
    }); 

    window.appInsights = appInsights; 
    appInsights.trackPageView(); 
</script>
```

В качестве процента выборки выберите значение в процентах, близкое к 100/N, где N — это целое число. В настоящее время выборка не поддерживает другие значения.

#### <a name="coordinating-server-side-and-client-side-sampling"></a>Координация выборки на стороне сервера и клиента

Клиентский пакет JavaScript на стороне клиента участвует в выборки с фиксированной частотой в сочетании с пакетом SDK на стороне сервера. Инструментированные страницы будут отсылать данные телеметрии на стороне клиента только от того же пользователя, для которого пакет SDK на стороне сервера принял решение включить в выборку. Эта логика предназначена для поддержания целостности пользовательских сеансов в клиентских и серверных приложениях. В результате из любого конкретного элемента телеметрии в Application Insights можно найти все другие элементы телеметрии для этого пользователя или сеанса, а в поиске можно перемещаться между связанными представлениями страниц и запросами.

Если на клиенте и в телеметрии на стороне сервера не отображаются согласованные образцы:

* Убедитесь, что вы включили выборку как на сервере, так и на клиенте.
* Проверьте, указан ли процент выборки на клиенте и сервере.
* Убедитесь в том, что используется пакет SDK версии 2.0 или выше.

## <a name="ingestion-sampling"></a>Выборка приема

Выборка приема действует в тот момент, когда данные телеметрии веб-сервера, браузеров и устройств достигают конечной точки службы Application Insights. Несмотря на то, что трафик телеметрии, отправляемой из приложения, такая выборка не уменьшает, она сокращает объем данных для обработки и сохранения в службе Application Insights, а значит и размер оплаты.

Используйте этот тип выборки, если ваше приложение часто выходит за рамки ежемесячной квоты, а возможности использовать один из типов выборки на основе SDK нет. 

Задайте частоту выборки на странице "Usage and estimated costs" (Данные об использовании и предполагаемые расходы).

![В колонке обзор приложения щелкните Параметры, квота, примеры, выберите частоту выборки и щелкните Обновить.](./media/sampling/data-sampling.png)

Как и в других типах выборки, алгоритм сохраняет связанные элементы телеметрии. Например, при проверке телеметрии в поиске это позволит найти запрос, связанный с определенным исключением. Правильно сохраняются счетчики метрик, например частота запросов и частота исключений.

Точки данных, отклоненные выборкой, доступны не во всех функциях Application Insights, например [Непрерывный экспорт](./export-telemetry.md).

Выборка приема не работает во время выполнения адаптивной или фиксированной выборки. Адаптивная выборка включена по умолчанию при использовании пакета SDK ASP.NET или ASP.NET Core SDK или при включении Application Insights в [службе приложений Azure ](azure-web-apps.md) или с помощью монитор состояния. Когда конечная точка службы Application Insights получает данные телеметрии, она проверяет данные телеметрии и, если частота выборки не превышает 100% (что означает, что выполняется выборка данных телеметрии), заданная скорость выборки игнорируется.

> [!WARNING]
> Значение, отображаемое на плитке портала, указывает значение, заданное для выборки приема. Он не представляет фактическую частоту выборки, если выполняется операция выборки пакета SDK (с адаптивной или фиксированной частотой).

## <a name="when-to-use-sampling"></a>Когда следует использовать выборку

Как правило, для большинства приложений малого и среднего размера выборка не требуется. Наиболее полезные диагностические сведения и наиболее точную статистику можно получить, собирая данные обо всех действиях пользователей. 

Основные преимущества выборки:

* Application Insights служба сбрасывает ("регулирует") точки данных, когда приложение отправляет очень высокую частоту телеметрии в течение короткого интервала времени. Выборка снижает вероятность того, что в приложении будет выполняться регулирование.
* Вы остаетесь в пределах [квоты](pricing.md) на точки данных для своей ценовой категории. 
* Вам нужно сократить сетевой трафик, который тратится на сбор данных телеметрии. 

### <a name="which-type-of-sampling-should-i-use"></a>Какой тип выборки следует использовать?

**Используйте выборку приема, если:**

* Вы часто используете месячную квоту телеметрии.
* Вы получаете слишком много данных телеметрии из веб-браузеров пользователей.
* Вы используете версию пакета SDK, которая не поддерживает выборки, например версию пакета для ASP.NET ниже 2.

**Используйте выборку с фиксированной частотой, если:**

* Вы хотите синхронизировать выборку между клиентом и сервером, чтобы при исследовании событий в [поиске](./diagnostic-search.md)можно было перемещаться между связанными событиями на стороне клиента и сервера, такими как Просмотры страниц и HTTP-запросы.
* Вам нужна уверенность в выборе соответствующего процента выборки для вашего приложения. Он должен быть достаточно высоким, чтобы получить точные метрики и при этом не превышать ценовую квоту и ограничения регулирования.

**Используйте адаптивную выборку:**

Если условия для использования других форм выборки не применяются, мы рекомендуем адаптивную выборку. Этот параметр включен по умолчанию в пакете SDK для ASP.NET/ASP.NET Core. Он не уменьшает трафик до тех пор, пока не будет достигнута определенная минимальная скорость, поэтому узлы с низким уровнем использования, скорее всего, вообще не будут выдаваться.

## <a name="knowing-whether-sampling-is-in-operation"></a>Принцип работы выборки

Чтобы узнать фактическую частоту выборки (где бы она ни применялась), выполните такой [запрос аналитики](../logs/log-query-overview.md) :

```kusto
union requests,dependencies,pageViews,browserTimings,exceptions,traces
| where timestamp > ago(1d)
| summarize RetainedPercentage = 100/avg(itemCount) by bin(timestamp, 1h), itemType
```

Если вы видите, что `RetainedPercentage` для любого типа меньше 100, то этот тип телеметрии будет выборке.

> [!IMPORTANT]
> В Application Insights не входит пример сеанса, метрики (включая пользовательские метрики) или типы телеметрии счетчиков производительности в любом из методов выборки. Эти типы всегда исключаются из выборки, так как снижение точности может быть очень нежелательным для этих типов телеметрии.

## <a name="how-sampling-works"></a>Принцип работы выборки

Алгоритм выборки определяет, какие элементы телеметрии следует удалить, и какие из них следует удерживать. Это верно, если выборка выполняется пакетом SDK или в службе Application Insights. Решение выборки основано на нескольких правилах, цель которых — сохранение всех точек взаимосвязанных данных без изменений, а также реализация функции диагностики в Application Insights даже с использованием сокращенного набора данных. Например, если в приложении есть неудачный запрос, содержащийся в примере, дополнительные элементы телеметрии (например, исключения и трассировки, регистрируемые для этого запроса) будут сохранены. Выборка либо сохраняет, либо удаляет их вместе. Поэтому при просмотре сведений о запросе в Application Insights вы всегда будете видеть запрос вместе со связанными элементами телеметрии.

Решение выборки основывается на ИДЕНТИФИКАТОРе операции запроса. Это означает, что все элементы телеметрии, относящиеся к определенной операции, либо сохраняются, либо удаляются. Для элементов телеметрии, для которых не задан идентификатор операции (например, элементы телеметрии, отправленные из асинхронных потоков без контекста HTTP), просто захватывает процент элементов телеметрии каждого типа.

Представляя вам данные телеметрии, служба Application Insights корректирует метрики в соответствии с процентом выборки, который использовался во время сбора. Это делается, чтобы компенсировать отсутствующие точки данных. Следовательно, при просмотре данных телеметрии в Application Insights пользователи видят статистически правильные приблизительные значения, очень близкие к реальным цифрам.

Точность приблизительного значения в значительной степени зависит от заданного процента выборки. Кроме того, точность возрастает для приложений, обрабатывающих большой объем сходных запросов от большого количества пользователей. С другой стороны, для приложений, которые не работают с существенной нагрузкой, выборка не требуется, так как такие приложения обычно могут отправлять все данные телеметрии, не выходя за пределы квоты и не вызывая потерю данных в результате регулирования. 

## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

*Что такое поведение выборки по умолчанию в пакетах SDK для ASP.NET и ASP.NET Core?*

* Если вы используете одну из последних версий указанного выше пакета SDK, Адаптивная выборка включается по умолчанию с пятью элементами телеметрии в секунду.
  По `AdaptiveSamplingTelemetryProcessor` умолчанию добавлены два узла, а в `Event` другом — тип, а другой — `Event` тип из выборки. Такая конфигурация означает, что пакет SDK будет пытаться ограничить элементы телеметрии до пяти элементов телеметрии `Event` типов и пять элементов телеметрии всех остальных типов, таким образом обеспечивая `Events` выборку отдельно от других типов телеметрии. События обычно используются для бизнес-телеметрии и, скорее всего, не должны быть затронуты томами телеметрии диагностики.
  
  Ниже показан созданный файл по умолчанию `ApplicationInsights.config` . В ASP.NET Core то же поведение по умолчанию включено в коде. Используйте [примеры из предыдущего раздела на этой странице](#configuring-adaptive-sampling-for-aspnet-core-applications) , чтобы изменить это поведение по умолчанию.

    ```xml
    <TelemetryProcessors>
        <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.AdaptiveSamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
            <MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>
            <ExcludedTypes>Event</ExcludedTypes>
        </Add>
        <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.AdaptiveSamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
            <MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>
            <IncludedTypes>Event</IncludedTypes>
        </Add>
    </TelemetryProcessors>
    ```

*Можно ли выполнить выборку данных телеметрии более одного раза?*

* Нет. Самплингтелеметрипроцессорс не учитывать элементы из соображений выборки, если элемент уже выбран. То же самое справедливо и для выборки приема, что не будет применять выборку к этим элементам, уже выбранным в пакете SDK.

*Почему выборка — это не простой сбор X процентов данных телеметрии каждого типа?*

* Хотя этот подход выборки обеспечивает высокий уровень точности в приближении метрик, он нарушает возможность корреляции диагностических данных для каждого пользователя, сеанса и запроса, что является критически важным для диагностики. Таким образом, выборка работает лучше с политиками, такими как "сбор всех элементов телеметрии для X процентов пользователей приложений", или "сбор всех данных телеметрии для X% запросов приложений". Для элементов телеметрии, не связанных с запросами (например, фоновой асинхронной обработкой), резервным вариантом является "сбор X процентов всех элементов для каждого типа телеметрии". 

*Может ли процент выборки меняться со временем?*

* Да, адаптивная выборка постепенно меняет процент выборки в зависимости от текущего объема данных телеметрии.

*Если я использую выборку с фиксированной частотой, как узнать, какой процент выборки будет оптимальным для моего приложения?*

* Один из способов — начать с адаптивной выборки, узнать подобранную частоту (см. вопрос выше) и переключиться на выборку с этой фиксированной частотой. 
  
    В противном случае придется только угадывать. Анализируйте текущее использование телеметрии в Application Insights, наблюдайте осуществляемое регулирование и оценивайте объем собранных данных телеметрии. Эти три фактора в сочетании с выбранной ценовой категорией позволяют оценить объем собранных данных телеметрии, который может потребоваться сократить. Однако увеличение числа пользователей или некоторые другие факторы, меняющие объем данных телеметрии, могут сделать оценку недействительной.

*Что произойдет, если задать слишком низкое значение в процентах выборки?*

* Чрезмерно низкий процент выборки вызывает чрезмерно интенсивную выборку и снижает точность приближений при Application Insights попытках компенсировать визуализацию данных для уменьшения объема данных. Кроме того, вы можете негативно повлиять на вашу диагностику, так как некоторые из нечасто возникающих или нежелательных запросов могут быть взяты из строя.

*Что произойдет, если задать слишком высокое значение в процентах выборки?*

* Настройка слишком большого процента выборки (недостаточно агрессивно) приводит к недостаточному снижению объема собранных данных телеметрии. У вас по-прежнему может наблюдаться потеря данных телеметрии, связанная с регулированием, а затраты на использование Application Insights могут быть выше запланированных из-за избыточных расходов.

*На каких платформах можно использовать выборку?*

* Выборка приема может выполняться автоматически, когда объем данных телеметрии превышает определенный порог, если пакет SDK не выполняет выборку. Такая конфигурация будет работать, например, при использовании более старой версии пакета SDK для ASP.NET или пакета SDK для Java.
* Если вы используете текущие пакеты SDK ASP.NET или ASP.NET Core (размещенные либо в Azure, либо на собственном сервере), вы получаете адаптивную выборку по умолчанию, но можно переключиться на фиксированную расставку, как описано выше. При выборке с фиксированной частотой пакет SDK браузера выполняет автоматическую синхронизацию с событиями, связанными с выборкой. 
* Если вы используете текущий агент Java, можно настроить `applicationinsights.json` (для пакета SDK для Java, настроить `ApplicationInsights.xml` ) для включения выборки с фиксированной частотой. По умолчанию выборка отключена. При использовании выборки с фиксированной частотой пакет SDK браузера и сервер автоматически синхронизируются с образцом связанных событий.

*Есть определенные редкие события, которые я всегда хочу просматривать. Как сделать так, чтобы модуль выборки не отфильтровывал их?*

* Лучший способ добиться этого — написать пользовательский [TelemetryInitializer](./api-filtering-sampling.md#addmodify-properties-itelemetryinitializer), который задает `SamplingPercentage` для элемента телеметрии значение 100, которое необходимо сохранить, как показано ниже. Так как инициализаторы гарантированно выполняются до обработчиков данных телеметрии (включая выборку), это гарантирует, что все методы выборки будут игнорировать этот элемент из соображений выборки. Пользовательские инициализаторы телеметрии доступны в пакете SDK для ASP.NET, ASP.NET Core пакете SDK, пакете SDK для JavaScript и пакете SDK для Java. Например, можно настроить инициализатор телеметрии с помощью пакета SDK для ASP.NET:

    ```csharp
    public class MyTelemetryInitializer : ITelemetryInitializer
    {
        public void Initialize(ITelemetry telemetry)
        {
            if(somecondition)
            {
                ((ISupportSampling)telemetry).SamplingPercentage = 100;
            }
        }
    }
    ```

## <a name="older-sdk-versions"></a>Более старые версии пакета SDK

Адаптивная выборка доступна для Application Insights пакета SDK для ASP.NET v 2.0.0-beta3 и более поздних версий, Microsoft. ApplicationInsights. AspNetCore SDK v 2.2.0-Beta1 и более поздних версий и включена по умолчанию.

Выборка с фиксированной частотой — это функция пакета SDK в ASP.NET версий от 2.0.0 и Java SDK версии 2.0.1 и выше.

До версии 2.5.0-2 пакета SDK ASP.NET и v 2.2.0-beta3 ASP.NET Core SDK решение выборки было основано на хэш-коде пользователя для приложений, определяющих "User" (то есть наиболее типичных веб-приложений). Для типов приложений, не определяющих пользователей (например, веб-служб), решение выборки было основано на ИДЕНТИФИКАТОРе операции запроса. Последние версии пакетов SDK для ASP.NET и ASP.NET Core используют идентификатор операции для решения выборки.

## <a name="next-steps"></a>Дальнейшие действия

* [Фильтрация](./api-filtering-sampling.md) может обеспечивать более строгий контроль над данными, отправляемыми пакетом SDK.
* Прочтите статью "сеть разработчика", чтобы [оптимизировать телеметрию с помощью Application Insights](/archive/msdn-magazine/2017/may/devops-optimize-telemetry-with-application-insights).

