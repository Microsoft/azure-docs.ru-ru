---
title: Счетчики событий в Application Insights | Документация Майкрософт
description: Мониторинг системных и пользовательских объектов EventCounter платформы .NET/.NET Core в Application Insights.
ms.topic: conceptual
ms.date: 09/20/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: d1ae0937c25a68798acd87fe8b2a0a54aa765b35
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100579529"
---
# <a name="eventcounters-introduction"></a>Знакомство с объектами EventCounter

[`EventCounter`](/dotnet/core/diagnostics/event-counters) является механизмом .NET и .NET Core для публикации и использования счетчиков или статистики. Счетчики событий (объекты EventCounter) поддерживаются во всех платформах ОС — Windows, Linux и macOS. Их можно рассматривать как кроссплатформенный эквивалент объектов [PerformanceCounter](/dotnet/api/system.diagnostics.performancecounter), которые поддерживаются только в Windows-системах.

Хотя пользователи могут публиковать любые пользовательские решения `EventCounters` в соответствии с их потребностями, среда выполнения .NET Core 3,0 и более поздних версий по умолчанию публикует набор этих счетчиков. В этом документе рассматриваются шаги, необходимые для получения и просмотра `EventCounters` (определяемые системой или определяемые пользователем) в Azure Application Insights.

## <a name="using-application-insights-to-collect-eventcounters"></a>Использование Application Insights для сбора объектов EventCounter

Application Insights поддерживает сбор `EventCounters` с помощью `EventCounterCollectionModule` , который входит в состав недавно выпущенного пакета NuGet [Microsoft. ApplicationInsights. евенткаунтерколлектор](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EventCounterCollector). Модуль `EventCounterCollectionModule` автоматически включается при использовании [AspNetCore](asp-net-core.md) или [WorkerService](worker-service.md). `EventCounterCollectionModule` собирает счетчики с частотой 1 раз в 60 секунд (эту частоту сбора нельзя изменить). Для сбора объектов EventCounter не нужны специальные разрешения.

## <a name="default-counters-collected"></a>Счетчики, собираемые по умолчанию

Начиная с версии 2.15.0 пакета SDK [AspNetCore](asp-net-core.md) или [воркерсервице SDK](worker-service.md), счетчики не собираются по умолчанию. Сам модуль включен, поэтому пользователи могут просто добавить нужные счетчики для их получения.

Чтобы получить список хорошо известных счетчиков, опубликованных средой выполнения .NET, см. статью [Доступные счетчики](/dotnet/core/diagnostics/event-counters#available-counters) .

## <a name="customizing-counters-to-be-collected"></a>Настройка собираемых счетчиков

В следующем примере показано, как следует добавлять или удалять счетчики. Эта настройка выполняется в методе `ConfigureServices` вашего приложения после того, как будет включен сбор данных телеметрии Application Insights с помощью метода `AddApplicationInsightsTelemetry()` или `AddApplicationInsightsWorkerService()`. Ниже приведен пример кода из приложения для ASP.NET Core. Информацию о других типах приложений можно найти в [этом](worker-service.md#configuring-or-removing-default-telemetrymodules) документе.

```csharp
    using Microsoft.ApplicationInsights.Extensibility.EventCounterCollector;
    using Microsoft.Extensions.DependencyInjection;

    public void ConfigureServices(IServiceCollection services)
    {
        //... other code...

        // The following code shows how to configure the module to collect
        // additional counters.
        services.ConfigureTelemetryModule<EventCounterCollectionModule>(
            (module, o) =>
            {
                // This removes all default counters, if any.
                module.Counters.Clear();

                // This adds a user defined counter "MyCounter" from EventSource named "MyEventSource"
                module.Counters.Add(new EventCounterCollectionRequest("MyEventSource", "MyCounter"));

                // This adds the system counter "gen-0-size" from "System.Runtime"
                module.Counters.Add(new EventCounterCollectionRequest("System.Runtime", "gen-0-size"));
            }
        );
    }
```

## <a name="disabling-eventcounter-collection-module"></a>Отключение модуля сбора Евенткаунтер

`EventCounterCollectionModule` можно отключить с помощью `ApplicationInsightsServiceOptions` . Пример использования ASP.NET Core SDK показан ниже.

```csharp
    using Microsoft.ApplicationInsights.AspNetCore.Extensions;
    using Microsoft.Extensions.DependencyInjection;

    public void ConfigureServices(IServiceCollection services)
    {
        //... other code...

        var applicationInsightsServiceOptions = new ApplicationInsightsServiceOptions();
        applicationInsightsServiceOptions.EnableEventCounterCollectionModule = false;
        services.AddApplicationInsightsTelemetry(applicationInsightsServiceOptions);
    }
```

Аналогичный подход можно также использовать и для пакета SDK Воркерсервице, но необходимо изменить пространство имен, как показано в примере ниже.

```csharp
    using Microsoft.ApplicationInsights.WorkerService;
    using Microsoft.Extensions.DependencyInjection;

    var applicationInsightsServiceOptions = new ApplicationInsightsServiceOptions();
    applicationInsightsServiceOptions.EnableEventCounterCollectionModule = false;
    services.AddApplicationInsightsTelemetryWorkerService(applicationInsightsServiceOptions);
```

## <a name="event-counters-in-metric-explorer"></a>Счетчики событий в обозревателе метрик

Чтобы просмотреть метрики EventCounter в [обозревателе метрик](../essentials/metrics-charts.md), выберите ресурс Application Insights и в качестве пространства имен метрик выберите метрики на основе журнала. После этого метрики EventCounter отобразятся в пользовательской категории.

> [!div class="mx-imgBorder"]
> ![Счетчики событий, зарегистрированные в обозревателе метрик Application Insights](./media/event-counters/metrics-explorer-counter-list.png)

## <a name="event-counters-in-analytics"></a>Счетчики событий в службе аналитики

Отчеты по счетчикам событий также можно искать и просматривать в [службе аналитики](../logs/log-query-overview.md), в таблице **customMetrics**.

Выполните, к примеру, следующий запрос, чтобы посмотреть, какие счетчики собираются и доступны для запроса:

```Kusto
customMetrics | summarize avg(value) by name
```

> [!div class="mx-imgBorder"]
> ![Счетчики событий, о которых сообщается в Application Insights Analytics](./media/event-counters/analytics-event-counters.png)

Чтобы получить диаграмму для определенного счетчика (например, `ThreadPool Completed Work Item Count`) за последний период времени, выполните следующий запрос.

```Kusto
customMetrics 
| where name contains "System.Runtime|ThreadPool Completed Work Item Count"
| where timestamp >= ago(1h)
| summarize  avg(value) by cloud_RoleInstance, bin(timestamp, 1m)
| render timechart
```
> [!div class="mx-imgBorder"]
> ![Диаграмма для одного счетчика в Application Insights](./media/event-counters/analytics-completeditems-counters.png)

Как и другие данные телеметрии, данные **customMetrics** также содержат столбец `cloud_RoleInstance`, который указывает идентификатор экземпляра сервера, на котором выполняется приложение. Приведенный выше запрос показывает значение счетчика для каждого экземпляра и может использоваться для сравнения производительности различных экземпляров сервера.

## <a name="alerts"></a>видны узлы
Как и для других метрик, вы можете [задать оповещение](../alerts/alerts-log.md), которое предупредит вас о том, что значение счетчика событий выходит за установленные пределы. Откройте колонку "Оповещения" и щелкните "Добавить оповещение".

## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

### <a name="can-i-see-eventcounters-in-live-metrics"></a>Можно ли просматривать объекты EventCounter в динамических метриках?

В настоящее время в динамических метриках нельзя просматривать объекты EventCounter. Для просмотра данных телеметрии используйте обозреватель метрик или службу аналитики.

### <a name="i-have-enabled-application-insights-from-azure-web-app-portal-but-i-cant-see-eventcounters"></a>Служба Application Insights была включена на портале веб-приложений Azure. Но почему я не вижу счетчики событий (объекты EventCounter)?

 [Расширение Application Insights](./azure-web-apps.md) для ASP.NET Core пока не поддерживает эту функцию. После того как эта функция начнет поддерживаться, соответствующая информация появится в этом документе.

## <a name="next-steps"></a><a name="next"></a>Следующие шаги

* [Отслеживание зависимостей](./asp-net-dependencies.md)

