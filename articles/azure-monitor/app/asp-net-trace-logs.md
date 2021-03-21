---
title: Просмотр журналов трассировки .NET в Application Insights
description: Поиск в журналах, созданных с помощью Trace, Log4Net или NLog.
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 05/08/2019
ms.openlocfilehash: 083ddbd06561550f89e414d6c679cdc6433fa338
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98937559"
---
# <a name="explore-netnet-core-and-python-trace-logs-in-application-insights"></a>Просмотр журналов трассировки Python, .NET или .NET Core в Application Insights

Отправьте диагностические журналы трассировки для приложения ASP.NET или ASP.NET Core из ILogger, NLog, log4Net или System.Diagnostics.Trace в [Azure Application Insights][start]. Для приложений Python отправьте журналы трассировки диагностики с помощью AzureLogHandler в OpenCensus Python для Azure Monitor. После этого их можно просматривать и выполнять по ним поиск. Эти журналы объединяются с другими файлами журналов из вашего приложения, поэтому вы можете выявлять трассировки, связанные с каждым запросом пользователя, и сопоставлять их с другими событиями и отчетами об исключениях.

> [!NOTE]
> Требуется модуль записи журнала? Это полезный адаптер для средств ведения журнала сторонних поставщиков, но если вы еще не используете NLog, log4Net или System.Diagnostics.Trace, рассмотрите возможность вызова метода [**TrackTrace() компонента Application Insights**](./api-custom-events-metrics.md#tracktrace) напрямую.
>
>
## <a name="install-logging-on-your-app"></a>Установка ведения журнала в приложении
Установите выбранную платформу ведения журналов в проект, что должно привести к созданию записи в файле app.config или web.config.

```XML
 <configuration>
  <system.diagnostics>
    <trace>
      <listeners>
        <add name="myAppInsightsListener" type="Microsoft.ApplicationInsights.TraceListener.ApplicationInsightsTraceListener, Microsoft.ApplicationInsights.TraceListener" />
      </listeners>
    </trace>
  </system.diagnostics>
</configuration>
```

## <a name="configure-application-insights-to-collect-logs"></a>Настройка Application Insights для сбора журналов
[Добавьте Application Insights в свой проект](./asp-net.md), если вы еще этого не сделали. Вы увидите параметр для включения сборщика журналов.

Или в Обозревателе решений щелкните проект правой кнопкой мыши и выберите **Настроить Application Insights**. Выберите параметр **Настроить сбор трассировок**.

> [!NOTE]
> Вы не видите меню Application Insights или параметр для включения сборщика журналируемых данных? Попробуйте выполнить [Устранение неполадок](#troubleshooting).

## <a name="manual-installation"></a>Ручная установка
Используйте этот метод, если ваш тип проекта не поддерживается программой установки Application Insights (например, если у вас проект настольного приложения Windows).

1. Если вы планируете использовать log4Net или NLog, установите его в свой проект.
2. В Обозревателе решений щелкните проект правой кнопкой мыши и выберите **Управление пакетами NuGet**.
3. Выполните поиск по запросу "Application Insights".
4. Выберите один из следующих пакетов.

   - Для ILogger: [Microsoft. Extensions. Logging. ApplicationInsights](https://www.nuget.org/packages/Microsoft.Extensions.Logging.ApplicationInsights/) 
 [ ![ NuGet ILogger баннер](https://img.shields.io/nuget/vpre/Microsoft.Extensions.Logging.ApplicationInsights.svg)](https://www.nuget.org/packages/Microsoft.Extensions.Logging.ApplicationInsights/)
   - Для NLog: [Microsoft. ApplicationInsights. нлогтаржет](https://www.nuget.org/packages/Microsoft.ApplicationInsights.NLogTarget/) 
 [ ![ NuGet NLog Banner](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.NLogTarget.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.NLogTarget/)
   - Для log4net: [Microsoft. ApplicationInsights. Log4NetAppender](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Log4NetAppender/) 
 [ ![ NuGet log4net Banner](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.Log4NetAppender.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Log4NetAppender/)
   - Для System. Diagnostics: [Microsoft. ApplicationInsights. TraceListener](https://www.nuget.org/packages/Microsoft.ApplicationInsights.TraceListener/) 
 [ ![ NuGet System. Диагностика баннера](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.TraceListener.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.TraceListener/)
   - [Microsoft. ApplicationInsights. диагностиксаурцелистенер](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DiagnosticSourceListener/) 
 [ ![ Баннер прослушивателя источника диагностики NuGet](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.DiagnosticSourceListener.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DiagnosticSourceListener/)
   - [Microsoft. ApplicationInsights. етвколлектор](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EtwCollector/) 
 [ ![ Баннер сборщика данных трассировки событий Windows NuGet](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.EtwCollector.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EtwCollector/)
   - [Microsoft. ApplicationInsights. евентсаурцелистенер](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EventSourceListener/) 
 [ ![ Баннер прослушивателя источника событий NuGet](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.EventSourceListener.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EventSourceListener/)

Пакет NuGet устанавливает необходимые сборки, а также вносит изменения в файл web.config или app.config, если применимо.

## <a name="ilogger"></a>ILogger

Примеры использования реализации ILogger Application Insights с консольными приложениями и ASP.NET Core см. в статье [ApplicationInsightsLoggerProvider для журналов ILogger приложений .NET Core](ilogger.md).

## <a name="insert-diagnostic-log-calls"></a>Вставка вызовов журнала диагностики
При использовании System.Diagnostics.Trace типичный вызов будет выглядеть следующим образом:

```csharp
System.Diagnostics.Trace.TraceWarning("Slow response - database01");
```

Если вы предпочитаете log4net или NLog, используйте следующий код:

```csharp
    logger.Warn("Slow response - database01");
```

## <a name="use-eventsource-events"></a>Использование событий EventSource
Можно настроить отправку событий [System.Diagnostics.Tracing.EventSource](/dotnet/api/system.diagnostics.tracing.eventsource) в Application Insights в виде трассировок. Сначала установите пакет NuGet `Microsoft.ApplicationInsights.EventSourceListener`. Затем измените раздел `TelemetryModules` файла [ApplicationInsights.config](./configuration-with-applicationinsights-config.md).

```xml
    <Add Type="Microsoft.ApplicationInsights.EventSourceListener.EventSourceTelemetryModule, Microsoft.ApplicationInsights.EventSourceListener">
      <Sources>
        <Add Name="MyCompany" Level="Verbose" />
      </Sources>
    </Add>
```

Для каждого источника можно задать следующие параметры.
 * **Name** задает имя собираемого события EventSource.
 * **Level** задает уровень ведения журнала: *Critical* (Критическая ошибка), *Error* (Ошибка), *Informational* (Информация), *LogAlways* (Постоянная запись), *Verbose* (Подробные сведения) или *Warning* (Предупреждение).
 * **Keywords** (необязательный параметр) задает целочисленное значение используемых комбинаций ключевых слов.

## <a name="use-diagnosticsource-events"></a>Использование событий DiagnosticSource
Можно настроить отправку событий [System.Diagnostics.DiagnosticSource](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md) в Application Insights в виде трассировок. Сначала установите пакет NuGet [`Microsoft.ApplicationInsights.DiagnosticSourceListener`](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DiagnosticSourceListener). Затем измените раздел TelemetryModules файла [ApplicationInsights.config](./configuration-with-applicationinsights-config.md).

```xml
    <Add Type="Microsoft.ApplicationInsights.DiagnosticSourceListener.DiagnosticSourceTelemetryModule, Microsoft.ApplicationInsights.DiagnosticSourceListener">
      <Sources>
        <Add Name="MyDiagnosticSourceName" />
      </Sources>
    </Add>
```

Для каждого объекта DiagnosticSource, трассировку которого необходимо выполнять, добавьте запись с атрибутом **Name**, которому присвоено имя DiagnosticSource.

## <a name="use-etw-events"></a>Использование событий трассировки событий Windows
Можно настроить отправку событий трассировки событий Windows (ETW) в Application Insights в качестве трассировок. Сначала установите пакет NuGet `Microsoft.ApplicationInsights.EtwCollector`. Затем измените раздел TelemetryModules файла [ApplicationInsights.config](./configuration-with-applicationinsights-config.md).

> [!NOTE] 
> События трассировки событий Windows можно собирать только в том случае, если процесс, в котором размещен пакет SDK, выполняется с удостоверением, которое является членом группы "Пользователи журналов производительности" или "Администраторы".

```xml
    <Add Type="Microsoft.ApplicationInsights.EtwCollector.EtwCollectorTelemetryModule, Microsoft.ApplicationInsights.EtwCollector">
      <Sources>
        <Add ProviderName="MyCompanyEventSourceName" Level="Verbose" />
      </Sources>
    </Add>
```

Для каждого источника можно задать следующие параметры.
 * **ProviderName** указывает имя поставщика трассировки событий Windows для сбора.
 * **ProviderGuid** указывает уникальный идентификатор (GUID) поставщика трассировки событий Windows для сбора. Этот параметр можно использовать вместо `ProviderName`.
 * **Level** задает уровень ведения журнала для сбора. Возможны следующие значения: *Critical* (Критическая ошибка), *Error* (Ошибка), *Informational* (Информация), *LogAlways* (Постоянная запись), *Verbose* (Подробные сведения) или *Warning* (Предупреждение).
 * **Keywords** (необязательный параметр) задает целочисленное значение используемых комбинаций ключевых слов.

## <a name="use-the-trace-api-directly"></a>Непосредственное использование API трассировки
API трассировки в Application Insights можно вызывать напрямую. Адаптеры ведения журналов используют этот API.

Пример:

```csharp
TelemetryConfiguration configuration = TelemetryConfiguration.CreateDefault();
var telemetryClient = new TelemetryClient(configuration);
telemetry.TrackTrace("Slow response - database01");
```

Преимуществом TrackTrace является возможность добавления в сообщения относительно длинных данных, например данных POST.

Вы также можете настроить для сообщения уровень серьезности. Как и для других данных телеметрии, вы можете добавлять значения свойства, используемые для фильтрации или поиска различных наборов трассировки. Пример:

  ```csharp
  TelemetryConfiguration configuration = TelemetryConfiguration.CreateDefault();
  var telemetryClient = new TelemetryClient(configuration);
  telemetryClient.TrackTrace("Slow database response",
                              SeverityLevel.Warning,
                              new Dictionary<string, string> { { "database", "db.ID" } });
  ```

Так вы сможете отфильтровать в колонке [Поиск][diagnostic] все сообщения с нужной степенью серьезности, относящиеся к определенной базе данных.

## <a name="azureloghandler-for-opencensus-python"></a>AzureLogHandler для OpenCensus Python
Azure Monitor Log Handler позволяет экспортировать журналы Python в Azure Monitor.

Инструментируйте приложение с помощью [пакета SDK OpenCensus Python](./opencensus-python.md) для Azure Monitor.

В этом примере показано, как отправить журнал уровня предупреждений в Azure Monitor.

```python
import logging

from opencensus.ext.azure.log_exporter import AzureLogHandler

logger = logging.getLogger(__name__)
logger.addHandler(AzureLogHandler(connection_string='InstrumentationKey=<your-instrumentation_key-here>'))
logger.warning('Hello, World!')
```

## <a name="explore-your-logs"></a>Просмотр журналов
Запустите приложение в режиме отладки или разверните его.

В области обзора приложения на [портале Application Insights][portal] выберите элемент [Поиск][diagnostic].

Можно, например:

* фильтровать трассировки журнала или элементы с определенными свойствами;
* просматривать подробные сведения конкретного элемента;
* выполнять поиск других данных системного журнала, связанных с тем же запросом пользователя (с тем же OperationId);
* сохранять конфигурацию страницы в списке избранного.

> [!NOTE]
>Если ваше приложение выполняет отправку больших объемов данных, а вы используете пакет SDK Application Insights для ASP.NET версии 2.0.0-beta3 или более поздней, может сработать функция *адаптивной выборки*, которая отправит только часть данных телеметрии. [Дополнительная информация о выборке.](./sampling.md)
>

## <a name="troubleshooting"></a>Устранение неполадок
### <a name="how-do-i-do-this-for-java"></a>Как это сделать в Java?
В инструментировании Java без программирования (рекомендуется) журналы собираются без каких-либо дополнительных настроек при использовании [агента Java 3.0](./java-in-process-agent.md).

Если вы используете пакет SDK для Java, воспользуйтесь [адаптерами журналов Java](./java-trace-logs.md).

### <a name="theres-no-application-insights-option-on-the-project-context-menu"></a>В контекстном меню проекта нет пункта для Application Insights
* Убедитесь, что на компьютере, на котором ведется разработка, установлено расширение Developer Analytics Tools. В Visual Studio выберите **Сервис** > **Расширения и обновления** и найдите **Developer Analytics Tools**. Если этого расширения нет на вкладке **Установленные**, перейдите на вкладку **В сети** и установите их.
* Возможно, Developer Analytics Tools не поддерживает тип этого проекта. Используйте [установку вручную](#manual-installation).

### <a name="theres-no-log-adapter-option-in-the-configuration-tool"></a>В средстве настройки нет параметра для адаптера журналов
* Сначала установите платформу ведения журналов.
* Если вы используете класс System.Diagnostics.Trace, убедитесь, что он [настроен в *web.config*](/dotnet/api/system.diagnostics.eventlogtracelistener).
* У вас должна быть установлена последняя версия Application Insights. В Visual Studio выберите **Сервис** > **Расширения и обновления** и откройте вкладку **Обновления**. Если в списке есть расширение **Developer Analytics Tools**, выберите его, чтобы обновить.

### <a name="i-get-the-instrumentation-key-cannot-be-empty-error-message"></a><a name="emptykey">Я получаю сообщение об ошибке "Instrumentation key cannot be empty" (Ключ инструментирования не может быть пустым)</a>
Возможно, вы установили пакет NuGet для адаптера ведения журнала без установки Application Insights. В Обозревателе решений щелкните правой кнопкой мыши *ApplicationInsights.config* и выберите пункт **Обновить Application Insights**. Вам будет предложено войти в Azure и создать ресурс Application Insights или повторно использовать существующий. Это устранит проблему.

### <a name="i-can-see-traces-but-not-other-events-in-diagnostic-search"></a>В результатах поиска диагностических данных есть трассировки, но нет других событий
Для получения всех событий и запросов через конвейер может потребоваться некоторое время.

### <a name="how-much-data-is-retained"></a><a name="limits"></a>Какой объем данных сохраняется?
Объем хранящихся данных зависит от нескольких факторов. Дополнительные сведения см. на странице метрик событий клиента в разделе [ограничений](./api-custom-events-metrics.md#limits).

### <a name="i-dont-see-some-log-entries-that-i-expected"></a>Некоторые ожидаемые записи журнала не отображаются
Если ваше приложение выполняет отправку больших объемов данных, а вы используете пакет SDK Application Insights для ASP.NET версии 2.0.0-beta3 или более поздней, может сработать функция адаптивной выборки, которая отправит только часть данных телеметрии. [Дополнительная информация о выборке.](./sampling.md)

## <a name="next-steps"></a><a name="add"></a>Следующие шаги

* [Диагностика ошибок и исключений в ASP.NET][exceptions]
* [Дополнительные сведения о службе поиска][diagnostic]
* [Наблюдение за доступностью и скоростью реагирования веб-сайта][availability]
* [Устранение неполадок][qna]

<!--Link references-->

[availability]: ./monitor-web-app-availability.md
[diagnostic]: ./diagnostic-search.md
[exceptions]: asp-net-exceptions.md
[portal]: https://portal.azure.com/
[qna]: ../faq.md
[start]: ./app-insights-overview.md

