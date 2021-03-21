---
title: Get-Metric в Azure Monitor Application Insights
description: Узнайте, как эффективно использовать вызов методаической метрики () для записи предварительно агрегированных метрик для приложений .NET и .NET Core с помощью Azure Monitor Application Insights
ms.service: azure-monitor
ms.subservice: application-insights
ms.topic: conceptual
ms.date: 04/28/2020
ms.openlocfilehash: 0ce2651d5cfcb1578d78982af109a004aaac11f4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101719786"
---
# <a name="custom-metric-collection-in-net-and-net-core"></a>Настраиваемая коллекция метрик в .NET и .NET Core

В пакетах SDK для .NET и .NET Core Azure Monitor Application Insights два разных способа сбора пользовательских метрик, `TrackMetric()` и `GetMetric()` . Основное различие между этими двумя методами заключается в локальном статистическом вычислении. `TrackMetric()` не имеет предварительных статистических вычислений `GetMetric()` , пока имеет предварительное агрегирование. Поэтому рекомендуется использовать агрегирование, поэтому `TrackMetric()` он больше не является предпочтительным методом сбора пользовательских метрик. В этой статье приведены пошаговые инструкции по использованию метода "Metric ()" и некоторые принципы его работы.

## <a name="trackmetric-versus-getmetric"></a>Сравнение TrackMetric и неметрики

`TrackMetric()` отправляет необработанные данные телеметрии, обозначая метрику. Отправка одного элемента телеметрии для каждого значения неэффективна. `TrackMetric()` также является неэффективным в плане производительности, поскольку каждый из них `TrackMetric(item)` проходит полный конвейер инициализаторов телеметрии и процессоров. В отличие от `TrackMetric()` , `GetMetric()` обрабатывает локальное предварительное агрегирование для вас, а затем отправляет агрегированную сводную метрику только через фиксированный интервал, равный одной минуте. Поэтому, если необходимо тщательно отслеживать какую – либо пользовательскую метрику на втором или даже миллисекунде, это можно сделать, если только объем хранилища и сетевой трафик отслеживаются каждую минуту. Это также значительно снижает риск регулирования, поскольку общее число элементов телеметрии, которые необходимо отправить для агрегированной метрики, значительно сокращается.

В Application Insights пользовательские метрики, собранные с помощью `TrackMetric()` и `GetMetric()` , не подчиняются [выборки](./sampling.md). Важные метрики выборки могут привести к ситуациям, когда предупреждения, которые могут быть созданы на основе этих метрик, станут ненадежными. Не выполнив выборку пользовательских метрик, вы, как правило, должны быть уверены, что при нарушении порогов оповещений будет срабатывать предупреждение.  Но поскольку пользовательские метрики не вычисляются, существуют некоторые потенциальные проблемы.

Если необходимо отслеживать тенденции в метрике каждую секунду или даже более детализированный интервал, это может привести к следующим результатам:

- Увеличение затрат на хранение данных. Объем данных, отправляемых в Azure Monitor, связан с затратами. (Чем больше данных вы отправляете, тем больше Общая стоимость мониторинга.)
- Увеличение сетевого трафика и снижения производительности. (В некоторых сценариях это может быть как денежной, так и последующей ценой за производительность приложения.)
- Риск регулирования приема. (Служба Azure Monitor сбрасывает (регулирование) точки данных, когда приложение отправляет очень высокую частоту телеметрии в течение короткого интервала времени.)

Регулирование — это конкретная проблема, такая как выборка, регулирование может привести к пропущенным предупреждениям, так как условие запуска предупреждения может произойти локально, а затем удаляться на конечной точке приема из-за слишком большого объема отправляемых данных. Именно поэтому для .NET и .NET Core мы не рекомендуем использовать, `TrackMetric()` Если вы не реализовали собственную локальную логику статистической обработки. Если вы пытаетесь отыскать каждый экземпляр события, возникающего в течение заданного периода времени, может оказаться, что [`TrackEvent()`](./api-custom-events-metrics.md#trackevent) он лучше подходит. Несмотря на то, что в отличие от пользовательских метрик, для пользовательских событий подчиняются выборка. Вы по-прежнему можете использовать `TrackMetric()` даже без написания собственного локального предварительно агрегирования, но, если это так, помните о таких ловушках.

В сводке `GetMetric()` является рекомендуемым подходом, поскольку он выполняет предварительную статистическую обработку, он накапливает значения из всех вызовов Track () и отправляет сводку или статистическое выражение каждую минуту. Это может значительно снизить затраты и издержки на производительность, отправляя меньше точек данных, сохраняя все важные сведения.

> [!NOTE]
> Только пакеты SDK для .NET и .NET Core имеют метод-Metric (). При использовании Java можно использовать [метрики микрометер](./micrometer-java.md) или `TrackMetric()` . Для JavaScript и Node.js вы по-прежнему используете `TrackMetric()` , но помните о предупреждениях, описанных в предыдущем разделе. Для Python можно использовать [опенценсус. stats](./opencensus-python.md#metrics) для отправки пользовательских метрик, но реализация метрик отличается.

## <a name="getting-started-with-getmetric"></a>Приступая к работе с параметром Metric

В нашем примере мы будем использовать базовое приложение рабочей службы .NET Core 3,1. Если вы хотите точно реплицировать тестовую среду, которая использовалась в этих примерах, выполните шаги 1-6 из [статьи мониторинг рабочей роли](./worker-service.md#net-core-30-worker-service-application) , чтобы добавить Application Insights в базовый шаблон проекта рабочей службы. Эти понятия относятся к любому общему приложению, в котором можно использовать пакет SDK, включая веб-приложения и консольные приложения.

### <a name="sending-metrics"></a>Отправка метрик

Замените содержимое `worker.cs` файла следующим:

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;
using Microsoft.ApplicationInsights;

namespace WorkerService3
{
    public class Worker : BackgroundService
    {
        private readonly ILogger<Worker> _logger;
        private TelemetryClient _telemetryClient;

        public Worker(ILogger<Worker> logger, TelemetryClient tc)
        {
            _logger = logger;
            _telemetryClient = tc;
        }

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {   // The following line demonstrates usages of GetMetric API.
            // Here "computersSold", a custom metric name, is being tracked with a value of 42 every second.
            while (!stoppingToken.IsCancellationRequested)
            {
                _telemetryClient.GetMetric("ComputersSold").TrackValue(42);

                _logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);
                await Task.Delay(1000, stoppingToken);
            }
        }
    }
}
```

Если запустить приведенный выше код и посмотреть, что данные телеметрии отправляются через окно вывода Visual Studio или средство, такое как Telerik Fiddler, цикл while будет повторяться без передачи данных телеметрии, а затем один элемент телеметрии будет отправлен примерно через 60-секундный знак, который в случае нашего теста выглядит следующим образом: :

```json
Application Insights Telemetry: {"name":"Microsoft.ApplicationInsights.Dev.00000000-0000-0000-0000-000000000000.Metric", "time":"2019-12-28T00:54:19.0000000Z",
"ikey":"00000000-0000-0000-0000-000000000000",
"tags":{"ai.application.ver":"1.0.0.0",
"ai.cloud.roleInstance":"Test-Computer-Name",
"ai.internal.sdkVersion":"m-agg2c:2.12.0-21496",
"ai.internal.nodeName":"Test-Computer-Name"},
"data":{"baseType":"MetricData",
"baseData":{"ver":2,"metrics":[{"name":"ComputersSold",
"kind":"Aggregation",
"value":1722,
"count":41,
"min":42,
"max":42,
"stdDev":0}],
"properties":{"_MS.AggregationIntervalMs":"42000",
"DeveloperMode":"true"}}}}
```

Этот единственный элемент телеметрии представляет собой совокупность 41 различных измерений метрик. Так как мы отправили одно и то же значение повторно, мы имеем *стандартное отклонение (stDev)* 0 с идентичным *максимальным (максимальным)* и *минимальным (min)* значениями. Свойство *value* представляет сумму всех отдельных значений, которые были агрегированы.

> [!NOTE]
> Параметр METRIC не поддерживает отслеживание последнего значения (т. е. "датчика"), а также гистограмм и распределений.

Если изучить наш Application Insights ресурс в службе "журналы (аналитика)", этот отдельный элемент телеметрии будет выглядеть следующим образом:

![Представление запроса Log Analytics](./media/get-metric/log-analytics.png)

> [!NOTE]
> Хотя Необработанный элемент телеметрии не содержал явно заданное свойство суммы или поле после получения, мы создаем его. В этом случае `value` и свойство, и `valueSum` представляют одно и то же.

Вы также можете получить доступ к пользовательской телеметрии метрик в разделе [_метрик_](../essentials/metrics-charts.md) на портале. В качестве [пользовательской метрики на основе журнала и](pre-aggregated-metrics-log-metrics.md). (На следующем снимке экрана показан пример на основе журнала.) ![Представление обозревателя метрик](./media/get-metric/metrics-explorer.png)

### <a name="caching-metric-reference-for-high-throughput-usage"></a>Кэширование ссылки на метрику для использования высокой пропускной способности

В некоторых случаях значения метрик наблюдаются очень часто. Например, служба высокой пропускной способности, обрабатывающая 500 запросов в секунду, может захотеть выдать 20 метрик телеметрии для каждого запроса. Это означает отслеживание 10 000 значений в секунду. В таких сценариях с высокой пропускной способностью пользователям может потребоваться помощь пакета SDK, избегая некоторых уточняющих запросов.

Например, в приведенном выше примере выполнялся поиск маркера для метрики «Компутерссолд», а затем наблюдаемое значение 42. Вместо этого маркер может быть кэширован для нескольких вызовов записи.

```csharp
//...

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            // This is where the cache is stored to handle faster lookup
            Metric computersSold = _telemetryClient.GetMetric("ComputersSold");
            while (!stoppingToken.IsCancellationRequested)
            {

                computersSold.TrackValue(42);

                computersSold.TrackValue(142);

                _logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);
                await Task.Delay(50, stoppingToken);
            }
        }

```

Помимо кэширования маркера метрики, приведенный выше пример также сокращает `Task.Delay` до 50 миллисекунд, чтобы цикл выполнялся чаще, что приводит к `TrackValue()` вызову 772.

## <a name="multi-dimensional-metrics"></a>Многомерные метрики

В примерах из предыдущего раздела показаны метрики нулевой размерности. Метрики также могут быть многомерными. В настоящее время поддерживается до 10 измерений.

 Ниже приведен пример создания одномерной метрики.

```csharp
//...

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            // This is an example of a metric with a single dimension.
            // FormFactor is the name of the dimension.
            Metric computersSold= _telemetryClient.GetMetric("ComputersSold", "FormFactor");

            while (!stoppingToken.IsCancellationRequested)
            {
                // The number of arguments (dimension values)
                // must match the number of dimensions specified while GetMetric.
                // Laptop, Tablet, etc are values for the dimension "FormFactor"
                computersSold.TrackValue(42, "Laptop");
                computersSold.TrackValue(20, "Tablet");
                computersSold.TrackValue(126, "Desktop");


                _logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);
                await Task.Delay(50, stoppingToken);
            }
        }

```

Выполнение этого кода в течение не менее 60 секунд приведет к отправке трех отдельных элементов телеметрии в Azure, каждый из которых представляет собой статистическую функцию одного из трех конструктивных факторов. Как и прежде, вы сможете просмотреть их в представлении журналов (Analytics):

![Представление log Analytics многомерной метрики](./media/get-metric/log-analytics-multi-dimensional.png)

А также в обозревателе метрик:

![Настраиваемые метрики](./media/get-metric/custom-metrics.png)

Однако вы заметите, что вы не можете разделить метрику по новому пользовательскому измерению или просмотреть пользовательское измерение в представлении метрики:

![Поддержка разбиения](./media/get-metric/splitting-support.png)

По умолчанию многомерные метрики в обозревателе метрик не включены в Application Insightsных ресурсах.

### <a name="enable-multi-dimensional-metrics"></a>Включить многомерные метрики

Чтобы включить многомерные метрики для ресурса Application Insights, выберите **использование и оценочные затраты**  >  **Настраиваемые метрики**  >  **Включить оповещения для пользовательских измерений метрики**  >  **ОК**. Дополнительные сведения об этом можно найти [здесь](pre-aggregated-metrics-log-metrics.md#custom-metrics-dimensions-and-pre-aggregation).

После внесения этого изменения и отправки новой многомерной телеметрии можно будет **Применить разделение**.

> [!NOTE]
> Только вновь отправленные метрики после включения компонента на портале будут храниться в измерениях.

![Применение разделения](./media/get-metric/apply-splitting.png)

И просмотрите агрегаты метрик для каждого измерения _формфактор_ :

![Конструктивные факторы](./media/get-metric/formfactor.png)

### <a name="how-to-use-metricidentifier-when-there-are-more-than-three-dimensions"></a>Использование МетриЦидентифиер при наличии более трех измерений

Сейчас поддерживаются 10 измерений, однако более трех измерений требует использования `MetricIdentifier` следующих элементов:

```csharp
// Add "using Microsoft.ApplicationInsights.Metrics;" to use MetricIdentifier
// MetricIdentifier id = new MetricIdentifier("[metricNamespace]","[metricId],"[dim1]","[dim2]","[dim3]","[dim4]","[dim5]");
MetricIdentifier id = new MetricIdentifier("CustomMetricNamespace","ComputerSold", "FormFactor", "GraphicsCard", "MemorySpeed", "BatteryCapacity", "StorageCapacity");
Metric computersSold  = _telemetryClient.GetMetric(id);
computersSold.TrackValue(110,"Laptop", "Nvidia", "DDR4", "39Wh", "1TB");
```

## <a name="custom-metric-configuration"></a>Настраиваемая конфигурация метрики

Если вы хотите изменить конфигурацию метрики, это необходимо сделать в том месте, где была инициализирована метрика.

### <a name="special-dimension-names"></a>Специальные имена измерений

Метрики не используют контекст телеметрии, `TelemetryClient` используемый для доступа к ним `MetricDimensionNames` . для этого ограничения лучше подходят специальные имена измерений, доступные в качестве констант в классе.

Агрегаты метрик, отправленные ниже "Специальный размер запроса операции" — метрика **не** будут иметь `Context.Operation.Name` значение "Специальная операция". В то время как `TrackMetric()` или любые другие тракккскскс () будут `OperationName` правильно настроены для "специальной операции".

``` csharp
        //...
        TelemetryClient specialClient;
        private static int GetCurrentRequestSize()
        {
            // Do stuff
            return 1100;
        }
        int requestSize = GetCurrentRequestSize()

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            while (!stoppingToken.IsCancellationRequested)
            {
                //...
                specialClient.Context.Operation.Name = "Special Operation";
                specialClient.GetMetric("Special Operation Request Size").TrackValue(requestSize);
                //...
            }
                   
        }
```

В этом случае используйте специальные имена измерений, перечисленные в классе, `MetricDimensionNames` чтобы указать `TelemetryContext` значения.

Например, когда статистическая метрика, полученная из следующей инструкции, отправляется в Application Insights облачную конечную точку, ее `Context.Operation.Name` поле данных будет иметь значение "Special Operation":

```csharp
_telemetryClient.GetMetric("Request Size", MetricDimensionNames.TelemetryContext.Operation.Name).TrackValue(requestSize, "Special Operation");
```

Значения этого специального измерения будут скопированы в `TelemetryContext` и не будут использоваться в качестве "нормального" измерения. Если необходимо также обеспечить измерение операции для обычного исследования метрик, необходимо создать отдельное измерение для этой цели:

```csharp
_telemetryClient.GetMetric("Request Size", "Operation Name", MetricDimensionNames.TelemetryContext.Operation.Name).TrackValue(requestSize, "Special Operation", "Special Operation");
```

### <a name="dimension-and-time-series-capping"></a>Ограниченное измерение и временной ряд

 Чтобы предотвратить случайное использование ресурсов подсистемой телеметрии, можно управлять максимальным количеством рядов данных на метрику. Ограничения по умолчанию — не более 1000 всего рядов данных на метрику и не более 100 различных значений для каждого измерения.

 В контексте измерения и временных рядов, которые мы используем `Metric.TrackValue(..)` , чтобы убедиться, что ограничения наблюдаются. Если ограничения уже достигнуты, `Metric.TrackValue(..)` возвращает значение «false», и оно не будет относиться к отслеживанию. В противном случае будет возвращено значение "true". Это полезно, если данные метрики берутся из введенных пользователем данных.

`MetricConfiguration`Конструктор принимает некоторые параметры для управления различными рядами в соответствующей метрике и объектом класса, реализующего `IMetricSeriesConfiguration` , который определяет поведение статистической обработки для каждого отдельного ряда метрики:

``` csharp
var metConfig = new MetricConfiguration(seriesCountLimit: 100, valuesPerDimensionLimit:2,
                new MetricSeriesConfigurationForMeasurement(restrictToUInt32Values: false));

Metric computersSold = _telemetryClient.GetMetric("ComputersSold", "Dimension1", "Dimension2", metConfig);

// Start tracking.
computersSold.TrackValue(100, "Dim1Value1", "Dim2Value1");
computersSold.TrackValue(100, "Dim1Value1", "Dim2Value2");

// The following call gives 3rd unique value for dimension2, which is above the limit of 2.
computersSold.TrackValue(100, "Dim1Value1", "Dim2Value3");
// The above call does not track the metric, and returns false.
```

* `seriesCountLimit` Максимальное количество временных рядов данных, которое может содержать метрика. По достижении этого ограничения вызовы метода `TrackValue()` не будут относиться.
* `valuesPerDimensionLimit` Аналогичным образом ограничивает количество уникальных значений на измерение.
* `restrictToUInt32Values` Определяет, следует ли относиться к отслеживанию неотрицательных целочисленных значений.

Ниже приведен пример того, как можно отправить сообщение, чтобы получить сведения о превышении лимитов ограничения.

```csharp
if (! computersSold.TrackValue(100, "Dim1Value1", "Dim2Value3"))
{
// Add "using Microsoft.ApplicationInsights.DataContract;" to use SeverityLevel.Error
_telemetryClient.TrackTrace("Metric value not tracked as value of one of the dimension exceeded the cap. Revisit the dimensions to ensure they are within the limits",
SeverityLevel.Error);
}
```

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные [сведения ](./worker-service.md)о мониторинге приложений службы рабочей роли.
* Дополнительные сведения о [метриках на основе журналов и предварительно агрегированных](./pre-aggregated-metrics-log-metrics.md)данных.
* [Обозреватель метрик](../essentials/metrics-getting-started.md)
* Включение Application Insights для [ASP.NET Core приложений](asp-net-core.md)
* Включение Application Insights для [приложений ASP.NET](asp-net.md)
