---
title: Диагностика с помощью Live Metrics Stream Azure Application Insights
description: Мониторинг веб-приложения в реальном времени с помощью пользовательских метрик и диагностика проблем с помощью динамического веб-канала сбоев, трассировок и событий.
ms.topic: conceptual
ms.date: 04/22/2019
ms.reviewer: sdash
ms.openlocfilehash: 865de94f1d9b4012a908643bbf87f38aeb8594a0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98679472"
---
# <a name="live-metrics-stream-monitor--diagnose-with-1-second-latency"></a>Live Metrics Stream: мониторинг и диагностика с задержкой в 1 секунду

Отслеживайте интерактивное веб-приложение с помощью Live Metrics Stream (также известного как Куиккпулсе) от [Application Insights](./app-insights-overview.md). Выбирайте и фильтруйте метрики и счетчики производительности для отслеживания в режиме реального времени, не нарушая работу служб. Проверяйте трассировки стека на основе образцов неудавшихся запросов и исключений. Вместе с [профилировщиком](./profiler.md) и [отладчиком моментальных снимков](./snapshot-debugger.md)Live Metrics Stream предоставляет мощный и неагрессивный диагностический инструмент для веб-сайта.

С помощью Live Metrics Stream можно выполнять следующие действия:

* Проверять выпущенное исправление, наблюдая за производительностью и числом сбоев.
* Наблюдать за результатом применения тестовых нагрузок и диагностировать проблемы в режиме реального времени.
* Сосредотачиваться на определенных тестовых сеансах или отфильтровывать известные проблемы, выбирая и фильтруя метрики, подлежащие отслеживанию.
* Получать трассировки исключений по мере того, как они возникают.
* Экспериментировать с фильтрами, чтобы найти наиболее важные ключевые показатели эффективности.
* Отслеживать любые счетчики производительности Windows в режиме реального времени.
* Легко определить сервер, на котором возникают проблемы, и с помощью фильтра получить все КПЭ и динамические веб-каналы, относящиеся только к этому серверу.

![Вкладка "динамические метрики"](./media/live-stream/live-metric.png)

В настоящее время динамические метрики поддерживаются для ASP.NET, ASP.NET Core, функций Azure, Java и Node.js приложений.

## <a name="get-started"></a>Начало работы

1. Следуйте рекомендациям для конкретного языка, чтобы включить динамические метрики.
   * [ASP.NET](./asp-net.md) — динамические метрики включены по умолчанию.
   * [ASP.NET Core](./asp-net-core.md). метрики Live включены по умолчанию.
   * [Консоль/Рабочая роль .NET/.NET Core](./worker-service.md)— динамические метрики включены по умолчанию.
   * [Приложения .NET — позволяют использовать код](#enable-livemetrics-using-code-for-any-net-application).
    * Активные метрики [Java](./java-in-process-agent.md) включены по умолчанию.
   * [Node.js](./nodejs.md#live-metrics)

2. На [портале Azure](https://portal.azure.com) откройте ресурс Application Insights для своего приложения, а затем откройте Live Stream.

3. [Защитите канал управления](#secure-the-control-channel), если в фильтрах могут использоваться конфиденциальные данные, например имена клиентов.

### <a name="enable-livemetrics-using-code-for-any-net-application"></a>Включение Ливеметрикс с помощью кода для любого приложения .NET

Несмотря на то, что Ливеметрикс включен по умолчанию при подключении с использованием рекомендуемых инструкций для приложений .NET, ниже показано, как настроить динамические метрики вручную.

1. Установка пакета NuGet [Microsoft. ApplicationInsights. перфкаунтерколлектор](https://www.nuget.org/packages/Microsoft.ApplicationInsights.PerfCounterCollector)
2. В следующем примере кода консольного приложения показано, как настроить динамические метрики.

```csharp
using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.Extensibility;
using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse;
using System;
using System.Threading.Tasks;

namespace LiveMetricsDemo
{
    class Program
    {
        static void Main(string[] args)
        {
            // Create a TelemetryConfiguration instance.
            TelemetryConfiguration config = TelemetryConfiguration.CreateDefault();
            config.InstrumentationKey = "INSTRUMENTATION-KEY-HERE";
            QuickPulseTelemetryProcessor quickPulseProcessor = null;
            config.DefaultTelemetrySink.TelemetryProcessorChainBuilder
                .Use((next) =>
                {
                    quickPulseProcessor = new QuickPulseTelemetryProcessor(next);
                    return quickPulseProcessor;
                })
                .Build();

            var quickPulseModule = new QuickPulseTelemetryModule();

            // Secure the control channel.
            // This is optional, but recommended.
            quickPulseModule.AuthenticationApiKey = "YOUR-API-KEY-HERE";
            quickPulseModule.Initialize(config);
            quickPulseModule.RegisterTelemetryProcessor(quickPulseProcessor);

            // Create a TelemetryClient instance. It is important
            // to use the same TelemetryConfiguration here as the one
            // used to setup Live Metrics.
            TelemetryClient client = new TelemetryClient(config);

            // This sample runs indefinitely. Replace with actual application logic.
            while (true)
            {
                // Send dependency and request telemetry.
                // These will be shown in Live Metrics stream.
                // CPU/Memory Performance counter is also shown
                // automatically without any additional steps.
                client.TrackDependency("My dependency", "target", "http://sample",
                    DateTimeOffset.Now, TimeSpan.FromMilliseconds(300), true);
                client.TrackRequest("My Request", DateTimeOffset.Now,
                    TimeSpan.FromMilliseconds(230), "200", true);
                Task.Delay(1000).Wait();
            }
        }
    }
}
```

Хотя приведенный выше пример предназначен для консольного приложения, один и тот же код может использоваться в любых приложениях .NET. Если включены какие-либо другие Телеметримодулес, для которых автоматическое собираются данные телеметрии, важно убедиться, что та же конфигурация, используемая для инициализации этих модулей, используется и для модуля Live метрик.

## <a name="how-does-live-metrics-stream-differ-from-metrics-explorer-and-analytics"></a>Чем Live Metrics Stream отличается от обозревателя метрик и службы аналитики?

| |Live Stream | Обозреватель метрик и служба аналитики |
|---|---|---|
|**Задержка**|Данные отображаются в течение одной секунды|Агрегирование выполняется в течение нескольких минут|
|**Нет сохранения**|Данные сохраняются, только пока они отображаются на диаграмме, а затем удаляются.|[Данные сохраняются 90 дней](./data-retention-privacy.md#how-long-is-the-data-kept)|
|**по запросу**|Данные передаются только в том случае, если открыта панель "активные метрики" |Данные отправляются, когда пакет SDK установлен и включен|
|**Бесплатный**|Плата за данные Live Stream не взимается|Действуют [расценки](./pricing.md)
|**Выборка**|Передаются все выбранные метрики и счетчики. Производится выборка сбоев и трассировок стека. |Может производиться [выборка](./api-filtering-sampling.md) событий.|
|**Канал управления**|В пакет SDK отправляются управляющие сигналы фильтрации. Мы советуем защитить этот канал.|Обмен данными — это один из способов, на портале|

## <a name="select-and-filter-your-metrics"></a>Выбор и фильтрация метрик

(Доступно с ASP.NET, ASP.NET Core и Функциями Azure версии 2.)

Можно отслеживать пользовательский КПЭ в реальном времени, применяя произвольные фильтры к любым данным телеметрии Application Insights на портале. Щелкните элемент управления фильтром, появляющийся при наведении указателя мыши на любую диаграмму. На диаграмме ниже показан пользовательский КПЭ числа запросов с фильтрами по для атрибутам URL-адреса и длительности. Проверьте результат фильтрации в разделе "Просмотр потока", в котором отображается динамический веб-канал телеметрии, соответствующей критериям, которые можно указать в любой момент времени.

![Частота запросов фильтра](./media/live-stream/filter-request.png)

Вы можете отслеживать не только количество. Параметры зависят от типа потока. Он может передавать любые данные телеметрии Application Insights: запросы, зависимости, исключения, трассировки, события или метрики. Поток может передавать и ваше [пользовательское измерение](./api-custom-events-metrics.md#properties).

![Построитель запросов на частоту запросов с пользовательской метрикой](./media/live-stream/query-builder-request.png)

Помимо телеметрии Application Insights, можно также отслеживать любой счетчик производительности Windows, выбрав его в параметрах потока и указав имя счетчика производительности.

Динамические метрики объединяются дважды: локально на каждом сервере, а затем на всех серверах. Можно изменить режим по умолчанию для обоих случаев, выбрав другие параметры в соответствующих раскрывающихся списках.

## <a name="sample-telemetry-custom-live-diagnostic-events"></a>Пример данных телеметрии: пользовательские события динамической диагностики
По умолчанию динамический веб-канал событий отображает примеры завершившихся сбоем запросов и вызовов зависимостей, исключений, событий и трассировок. Щелкните значок фильтра, чтобы просмотреть действующие критерии в любой момент времени.

![Кнопка фильтра](./media/live-stream/filter.png)

Как и для метрик, можно указать любые произвольные критерии для любого типа данных телеметрии Application Insights. В этом примере мы выбираем определенные ошибки запросов и события.

![Построитель запросов](./media/live-stream/query-builder.png)

> [!NOTE]
> Сейчас для критериев на основе сообщений об исключениях используйте сообщение о внешнем исключении. В предыдущем примере, чтобы отфильтровать неопасные исключения с сообщением о внутреннем исключении (после разделителя "<--") "Клиент отключен", использовался критерий "Message not-contains "Error reading request content"" (Сообщение не содержит "Ошибка при чтении содержимого запроса").

Просмотрите сведения об элементе динамического веб-канала, щелкнув его. Можно приостановить веб-канал, щелкнув **Приостановить**, прокрутив вниз или щелкнув элемент. Работа динамического веб-канала возобновится, когда вы прокрутите его обратно до начала или щелкните счетчик элементов, собранных во время приостановки.

![На снимке экрана показан пример окна телеметрии с выбранным исключением и сведения об исключении, отображаемые в нижней части окна.](./media/live-stream/sample-telemetry.png)

## <a name="filter-by-server-instance"></a>Фильтрация по экземпляру сервера

Если требуется отслеживать определенный экземпляр роли сервера, можно применить фильтрацию по серверу. Для фильтрации выберите имя сервера в разделе *серверы*.

![Выборка динамических ошибок](./media/live-stream/filter-by-server.png)

## <a name="secure-the-control-channel"></a>Защита канала управления

> [!NOTE]
> В настоящее время можно настроить только канал с проверкой подлинности с помощью мониторинга на основе кода и не может проверять подлинность серверов с помощью подключения без кода.

Критерии настраиваемых фильтров, указываемые на портале динамических метрик, отправляются обратно в динамический компонент метрик в пакете SDK для Application Insights. Фильтры могут содержать конфиденциальные сведения, например идентификаторы клиентов. Помимо ключа инструментирования, канал для их передачи можно защитить секретным ключом API.

### <a name="create-an-api-key"></a>Создание ключа API

![Ключ API > создать ключ API создание ключа API ](./media/live-stream/api-key.png)
 ![ . Выберите "проверить подлинность канала управления SDK", а затем "создать ключ".](./media/live-stream/create-api-key.png)

### <a name="add-api-key-to-configuration"></a>Добавление ключа API в конфигурацию

### <a name="aspnet"></a>ASP.NET

В файле applicationinsights.config добавьте AuthenticationApiKey в QuickPulseTelemetryModule.

```XML
<Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse.QuickPulseTelemetryModule, Microsoft.AI.PerfCounterCollector">
      <AuthenticationApiKey>YOUR-API-KEY-HERE</AuthenticationApiKey>
</Add>
```

### <a name="aspnet-core"></a>ASP.NET Core

Для [ASP.NET Core](./asp-net-core.md) приложений следуйте приведенным ниже инструкциям.

Измените `ConfigureServices` файл startup. cs следующим образом:

Добавьте следующее пространство имен.

```csharp
using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse;
```

Затем измените `ConfigureServices` метод, как показано ниже.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // existing code which include services.AddApplicationInsightsTelemetry() to enable Application Insights.
    services.ConfigureTelemetryModule<QuickPulseTelemetryModule> ((module, o) => module.AuthenticationApiKey = "YOUR-API-KEY-HERE");
}
```

Дополнительные сведения о настройке ASP.NET Core приложений можно найти в нашем руководстве по [настройке модулей телеметрии в ASP.NET Core](./asp-net-core.md#configuring-or-removing-default-telemetrymodules).

### <a name="workerservice"></a>воркерсервице

Для приложений [воркерсервице](./worker-service.md) следуйте приведенным ниже инструкциям.

Добавьте следующее пространство имен.

```csharp
using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse;
```

Затем добавьте следующую строку перед вызовом `services.AddApplicationInsightsTelemetryWorkerService` .

```csharp
    services.ConfigureTelemetryModule<QuickPulseTelemetryModule> ((module, o) => module.AuthenticationApiKey = "YOUR-API-KEY-HERE");
```

Дополнительные сведения о настройке приложений Воркерсервице можно найти в нашем руководстве по [настройке модулей телеметрии в воркерсервицес](./worker-service.md#configuring-or-removing-default-telemetrymodules).

### <a name="azure-function-apps"></a>Приложения-функции Azure

Для приложений-функций Azure (v2) Защита канала с помощью ключа API может быть выполнена с помощью переменной среды.

Создайте ключ API из ресурса Application Insights и перейдите к **параметрам > конфигурации** приложение-функция. Выберите **параметр создать приложение** и введите имя `APPINSIGHTS_QUICKPULSEAUTHAPIKEY` и значение, соответствующее ключу API.

Однако, если все подключенные серверы опознаны и надежны, можно использовать настраиваемые фильтры без аутентифицированного канала. Эта возможность доступна в течение шести месяцев. Это переопределение требуется после каждого создания сеанса или подключения нового сервера.

![Параметры аутентификации Live Metrics](./media/live-stream/live-stream-auth.png)

>[!NOTE]
>Настоятельно рекомендуется установить аутентифицированный канал перед вводом в критерии фильтра потенциально конфиденциальной информации, например идентификатора клиента.
>

## <a name="supported-features-table"></a>Таблица поддерживаемых возможностей

| Язык                         | Основные метрики       | Метрики производительности | Настраиваемая фильтрация    | Пример телеметрии    | Разделение ЦП по процессам |
|----------------------------------|:--------------------|:--------------------|:--------------------|:--------------------|:---------------------|
| .NET Framework                   | Поддерживается (V 2.7.2 +) | Поддерживается (V 2.7.2 +) | Поддерживается (V 2.7.2 +) | Поддерживается (V 2.7.2 +) | Поддерживается (V 2.7.2 +)  |
| .NET Core (целевой объект =. NET Framework)| Поддерживается (V 2.4.1 +) | Поддерживается (V 2.4.1 +) | Поддерживается (V 2.4.1 +) | Поддерживается (V 2.4.1 +) | Поддерживается (V 2.4.1 +)  |
| .NET Core (целевой объект =. NET Core)     | Поддерживается (V 2.4.1 +) | Поддерживается*          | Поддерживается (V 2.4.1 +) | Поддерживается (V 2.4.1 +) | **Не поддерживается**    |
| Функции Azure версии 2               | Поддерживается           | Поддерживается           | Поддерживается           | Поддерживается           | **Не поддерживается**    |
| Java                             | Поддерживается (V 2.0.0 +) | Поддерживается (V 2.0.0 +) | **Не поддерживается**   | **Не поддерживается**   | **Не поддерживается**    |
| Node.js                          | Поддерживается (V 1.3.0 +) | Поддерживается (V 1.3.0 +) | **Не поддерживается**   | Поддерживается (V 1.3.0 +) | **Не поддерживается**    |

Основные метрики включают в себя запрос, зависимость и частоту исключений. Метрики производительности (счетчики производительности) включают в себя память и ЦП. Пример телеметрии показывает поток подробных сведений о неудачных запросах и зависимостях, исключениях, событиях и трассировках.

 \* Поддержка PerfCounters незначительно различается в разных версиях .NET Core, которые не предназначены для платформа .NET Framework:

- Метрики PerfCounters поддерживаются при работе в службе приложений Azure для Windows. (Пакет SDK для AspNetCore версии 2.4.1 или более поздней)
- PerfCounters поддерживаются, когда приложение выполняется на любых компьютерах Windows (виртуальная машина или облачная служба или локальная и т. д.). (Пакет SDK для AspNetCore версии 2.7.1 или более поздней), но для приложений, предназначенных для .NET Core 2,0 или более поздних версий.
- PerfCounters поддерживаются, когда приложение работает где УГОДно (Linux, Windows, служба приложений для Linux, контейнеры и т. д.) в последних версиях (т. е. AspNetCore SDK версии 2.8.0 или более поздней), но только для приложений, предназначенных для .NET Core 2,0 или более поздней версии.

## <a name="troubleshooting"></a>Устранение неполадок

Live Metrics Stream использует IP-адреса, отличные от других Application Insights телеметрии. Убедитесь, что [эти IP-адреса](./ip-addresses.md) открыты в брандмауэре. Также убедитесь, что [исходящие порты для Live Metrics Stream](./ip-addresses.md#outgoing-ports) открыты в брандмауэре серверов.

## <a name="next-steps"></a>Дальнейшие действия

* [Отслеживание использования Application Insights.](./usage-overview.md)
* [Использование диагностического поиска](./diagnostic-search.md)
* [Profiler](./profiler.md)
* [Отладчик моментальных снимков](./snapshot-debugger.md)