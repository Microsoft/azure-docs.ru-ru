---
title: Счетчики производительности в Application Insights | Документация Майкрософт
description: Мониторинг системных и пользовательских счетчиков производительности .NET в Application Insights.
ms.topic: conceptual
ms.date: 12/13/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 423e17ef2b44286c28b464836075284929d8644c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102031367"
---
# <a name="system-performance-counters-in-application-insights"></a>Системные счетчики производительности в Application Insights

В Windows предусмотрены самые разные [счетчики производительности](/windows/desktop/perfctrs/about-performance-counters), которые отображают показатели использования ЦП, памяти, диска и сети. Вы также можете определить собственные счетчики производительности. Сбор данных со счетчиков производительности поддерживается, если приложение запущено под управлением службы IIS на локальном узле или виртуальной машине, к которой у вас есть доступ с правами администратора. Хотя приложения, работающие как веб-приложения Azure, не имеют прямого доступа к счетчикам производительности, Application Insights собирает информацию с части доступных счетчиков.

## <a name="view-counters"></a>Просмотр счетчиков

В колонке "Метрики" показан стандартный набор счетчиков производительности.

![Счетчики производительности, отображаемые в Application Insights](./media/performance-counters/performance-counters.png)

Текущие счетчики по умолчанию, настроенные для сбора для веб-приложений ASP.NET:
- % процесса\\Загруженность процессора
- % процесса\\Нормализованная загруженность процессора
- Память\\Доступно байтов
- Запросов ASP.NET/сек
- Исключений .NET CLR/сек
- Время исполнения запросов ApplicationsRequest ASP.NET
- Процесс\\Байтов исключительного пользования
- Процесс\\Обмен данными ввода-вывода, байт/сек
- Приложения ASP.NET\\Запросов в очереди приложения
- Процессор(_общий объем ресурсов)\\% загруженности процессора

Текущие счетчики по умолчанию, настроенные для сбора для ASP.NET Core веб-приложений:
- % процесса\\Загруженность процессора
- % процесса\\Нормализованная загруженность процессора
- Память\\Доступно байтов
- Процесс\\Байтов исключительного пользования
- Процесс\\Обмен данными ввода-вывода, байт/сек
- Процессор(_общий объем ресурсов)\\% загруженности процессора

## <a name="add-counters"></a>Добавление счетчиков

Если нужный счетчик производительности не включен в список метрик, можно добавить его.

1. Чтобы получить список счетчиков, доступных на сервере, выполните на локальном сервере такую команду PowerShell:

    `Get-Counter -ListSet *`

    (См. [`Get-Counter`](/powershell/module/microsoft.powershell.diagnostics/get-counter).)
2. Откройте файл ApplicationInsights.config.

   * Если вы добавили Application Insights в приложение во время разработки, отредактируйте файл ApplicationInsights.config в проекте и повторно разверните его на серверах.
3. Измените директиву сборщика данных производительности:

    ```XML

        <Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.PerformanceCollectorModule, Microsoft.AI.PerfCounterCollector">
          <Counters>
            <Add PerformanceCounter="\Objects\Processes"/>
            <Add PerformanceCounter="\Sales(photo)\# Items Sold" ReportAs="Photo sales"/>
          </Counters>
        </Add>
    ```

> [!NOTE]
> Приложения ASP.NET Core не имеют `ApplicationInsights.config`, поэтому приведенный выше метод не является допустимым для приложений ASP.NET Core.

Вы можете собирать показания как стандартных счетчиков, так и созданных вами самостоятельно. Пример стандартного счетчика `\Objects\Processes` доступен во всех системах Windows. Пример пользовательского счетчика, который можно реализовать в веб-службе: `\Sales(photo)\# Items Sold`.

Используется формат `\Category(instance)\Counter"`, а для категорий без экземпляров — просто `\Category\Counter`.

`ReportAs` — обязательный параметр для счетчиков, имена которых не соответствуют маске `[a-zA-Z()/-_ \.]+`, т. е. содержат любые символы, не входящие в этот стандартный набор: буквы, круглые скобки, косую черту, дефис, символ подчеркивания, пробел и точку.

При указании экземпляра он будет собираться в качестве измерения CounterInstanceName обнаруженной метрики.

### <a name="collecting-performance-counters-in-code-for-aspnet-web-applications-or-netnet-core-console-applications"></a>Сбор счетчиков производительности в коде для веб-приложений ASP.NET или консольных приложений .NET и .NET Core
Чтобы собрать данные счетчиков производительности системы и передать их в Application Insights, можно использовать следующий фрагмент кода:


```csharp
    var perfCollectorModule = new PerformanceCollectorModule();
    perfCollectorModule.Counters.Add(new PerformanceCounterCollectionRequest(
      @"\Process([replace-with-application-process-name])\Page Faults/sec", "PageFaultsPerfSec"));
    perfCollectorModule.Initialize(TelemetryConfiguration.Active);
```

То же самое можно сделать с пользовательскими метриками, созданными вами:

```csharp
    var perfCollectorModule = new PerformanceCollectorModule();
    perfCollectorModule.Counters.Add(new PerformanceCounterCollectionRequest(
      @"\Sales(photo)\# Items Sold", "Photo sales"));
    perfCollectorModule.Initialize(TelemetryConfiguration.Active);
```

### <a name="collecting-performance-counters-in-code-for-aspnet-core-web-applications"></a>Сбор счетчиков производительности в коде для веб-приложений ASP.NET Core

Измените `ConfigureServices` метод в классе `Startup.cs`, как показано ниже.

```csharp
using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector;

    public void ConfigureServices(IServiceCollection services)
    {
        services.AddApplicationInsightsTelemetry();

        // The following configures PerformanceCollectorModule.
  services.ConfigureTelemetryModule<PerformanceCollectorModule>((module, o) =>
            {
                // the application process name could be "dotnet" for ASP.NET Core self-hosted applications.
                module.Counters.Add(new PerformanceCounterCollectionRequest(
    @"\Process([replace-with-application-process-name])\Page Faults/sec", "DotnetPageFaultsPerfSec"));
            });
    }
```

## <a name="performance-counters-in-analytics"></a>Счетчики производительности в службе аналитики
В [службе аналитики](../logs/log-query-overview.md) можно выполнять поиск отчетов по счетчикам производительности и отображать их.

Схема **PerformanceCounters** предоставляет `category`, имя `counter` и имя `instance` каждого счетчика производительности.  В данных телеметрии для каждого приложения вы увидите только счетчики для этого приложения. Например, вот как можно увидеть, какие счетчики доступны. 

![Счетчики производительности в аналитике Application Insights](./media/performance-counters/analytics-performance-counters.png)

("Экземпляр" здесь обозначает экземпляр счетчика производительности, а не экземпляр роли или сервера. Имя экземпляра счетчика производительности обычно разделяет такие счетчики, как загруженность процессора, по именам процессов или приложений.)

Вот как можно получить диаграмму доступной памяти за последний период. 

![Временная диаграмма памяти в аналитике Application Insights](./media/performance-counters/analytics-available-memory.png)

Как и другие данные телеметрии, данные **performanceCounters** также содержат столбец `cloud_RoleInstance`, который определяет экземпляр сервера, на котором выполняется приложение. Например, вот как можно сравнить производительность приложения на разных компьютерах. 

![Производительность, сегментированная по экземпляру роли в аналитике Application Insights](./media/performance-counters/analytics-metrics-role-instance.png)

## <a name="aspnet-and-application-insights-counts"></a>ASP.NET и счетчики Application Insights

*Чем отличаются метрики "Частота исключений" и "Исключения"?*

* *Частота исключений* — это системный счетчик производительности. Среда CLR подсчитывает все обработанные и необработанные исключения и делит их общее количество в интервале выборки на продолжительность интервала. Пакет SDK Application Insights получает этот результат и отправляет его на портал.

* *Исключения* — это количество отчетов TrackException, полученных порталом в интервале выборки диаграммы. Включает только обработанные исключения, в коде которых прописаны вызовы TrackException, и не включает [необработанные исключения](./asp-net-exceptions.md). 

## <a name="performance-counters-for-applications-running-in-azure-web-apps"></a>Счетчики производительности для приложений, работающих в веб-приложениях Azure

Приложения ASP.NET и ASP.NET Core, развернутые в веб-приложениях Azure, выполняются в специальной изолированной среде. Эта среда не разрешает прямой доступ к счетчикам производительности системы. Но ограниченный набор счетчиков предоставляется в виде переменных среды, как описано [здесь](https://github.com/projectkudu/kudu/wiki/Perf-Counters-exposed-as-environment-variables). Пакет SDK Application Insights для ASP.NET и ASP.NET Core собирает счетчики производительности веб-приложений Azure из этих специальных переменных среды. В этой среде доступна только часть счетчиков, а полный список можно найти [здесь](https://github.com/microsoft/ApplicationInsights-dotnet-server/blob/develop/WEB/Src/PerformanceCollector/Perf.Shared/Implementation/WebAppPerformanceCollector/CounterFactory.cs).

## <a name="performance-counters-in-aspnet-core-applications"></a>Счетчики производительности в приложениях ASP.NET Core

Поддержка счетчиков производительности в приложениях ASP.NET Core ограничена.

* [Пакеты SDK](https://nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore) версии 2.4.1 и более поздних собирают данные счетчиков производительности, если приложение выполняется в веб-приложениях Azure (Windows).
* Пакеты SDK версии 2.7.1 и более поздних собирают счетчики производительности, если приложение выполняется в Windows и предназначено для `NETSTANDARD2.0` или более поздних версий.
* Для приложений, предназначенных для .NET Framework, все версии пакетов SDK поддерживают счетчики производительности.
* Пакеты SDK версии 2.8.0 и более поздних поддерживают счетчик ЦП/памяти в Linux. В Linux не поддерживаются никакие другие счетчики. Для получения системных счетчиков в Linux (и других средах, отличных от Windows) рекомендуется использовать [EventCounters](eventcounters.md).

## <a name="alerts"></a>видны узлы
Как и для других метрик, вы можете [установить оповещение](../alerts/alerts-log.md), которое предупредит о выходе показаний счетчика производительности за установленные пределы. Откройте колонку "Оповещения" и щелкните "Добавить оповещение".

## <a name="next-steps"></a><a name="next"></a>Следующие шаги

* [Отслеживание зависимостей](./asp-net-dependencies.md)
* [Отслеживание исключений](./asp-net-exceptions.md)

