---
title: Метрики на основе журнала Application Insights Azure | Документация Майкрософт
description: В этой статье перечислены метрики Application Insights Azure с поддерживаемыми агрегатами и измерениями. Сведения о метриках на основе журнала включают в себя базовые инструкции запроса Kusto.
author: vgorbenko
services: azure-monitor
ms.topic: reference
ms.date: 07/03/2019
ms.author: vitalyg
ms.openlocfilehash: bd28fa9f20775be659de6cef99f9ceda29fe52cc
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102049140"
---
# <a name="application-insights-log-based-metrics"></a>Метрики на основе журналов Application Insights

Application Insights метрики на основе журналов позволяют анализировать работоспособность наблюдаемых приложений, создавать мощные панели мониторинга и настраивать оповещения. Существует два вида метрик:

* [Метрики на основе журнала](../app/pre-aggregated-metrics-log-metrics.md#log-based-metrics) , расположенные за сценой, преобразуются в [запросы Kusto](/azure/kusto/query/) из хранимых событий.
* [Стандартные метрики](../app/pre-aggregated-metrics-log-metrics.md#pre-aggregated-metrics) хранятся в виде предварительно агрегированных временных рядов.

Так как *Стандартные метрики* предварительно агрегированы во время сбора, они имеют лучшую производительность во время выполнения запроса. Это делает их лучшим выбором для панелей мониторинга и оповещений в режиме реального времени. *Метрики на основе журнала* имеют больше измерений, что делает их главным вариантом для анализа данных и автоматизированной диагностики. С помощью [селектора пространства имен](./metrics-getting-started.md#create-your-first-metric-chart) можно переключаться между метриками на основе журнала и стандартными показателями в [обозревателе метрик](./metrics-getting-started.md).

## <a name="interpret-and-use-queries-from-this-article"></a>Интерпретировать и использовать запросы из этой статьи

В этой статье перечислены метрики с поддерживаемыми агрегатами и измерениями. Сведения о метриках на основе журнала включают в себя базовые инструкции запроса Kusto. Для удобства каждый запрос использует значения по умолчанию для гранулярности времени, типа диаграммы и иногда разделяет измерение, которое упрощает использование запроса в Log Analytics без необходимости изменения.

При построении одной метрики в [обозревателе метрик](./metrics-getting-started.md)нет значений по умолчанию — запрос динамически корректируется в соответствии с настройками диаграммы:

- Выбранный **диапазон времени** преобразуется в дополнительное предложение *WHERE timestamp...* для выбора только событий из выбранного диапазона времени. Например, диаграмма, показывающая данные за последние 24 часа, включается в запрос *| где timestamp > назад (24 h)*.

- Выбранная **гранулярность времени** помещается в итоговую *сводку... по ячейке (timestamp, [Гран времени])* .

- Все выбранные измерения **фильтра** преобразуются в дополнительные предложения *WHERE* .

- Выбранное измерение **разделенной диаграммы** преобразуется в дополнительное свойство суммирования. Например, если вы разбиваете диаграмму по *расположению* и графику, используя 5-минутную гранулярность, предложение *суммирования* суммируется *... по ячейке (метка времени, 5 м), расположение*.

> [!NOTE]
> Если вы не знакомы с языком запросов Kusto, начните с копирования и перетаскивания инструкций Kusto на панель запросов Log Analytics, не внося никаких изменений. Нажмите кнопку **выполнить** , чтобы увидеть базовую диаграмму. По мере изучения синтаксиса языка запросов можно приступить к внесению небольших изменений и увидеть влияние изменений. Изучение собственных данных — отличный способ приступить к реализации полной мощности [log Analytics](../logs/log-analytics-tutorial.md) и [Azure Monitor](../overview.md).

## <a name="availability-metrics"></a>Метрики доступности

Метрики в категории доступности позволяют просматривать работоспособность веб-приложения с точки зрения мира. [Настройте тесты доступности](../app/monitor-web-app-availability.md) , чтобы начать использовать метрики из этой категории.

### <a name="availability-availabilityresultsavailabilitypercentage"></a>Доступность (availabilityResults/Аваилабилитиперцентаже)
Метрика *доступности* показывает процент запусков веб-тестов, которые не обнаруживают проблемы. Наименьшее возможное значение равно 0, что означает сбой всех выполнений веб-тестов. Значение 100 означает, что все выполненные веб-тесты прошли условия проверки.

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|---|---|---|
|Процент|Среднее значение|Расположение запуска, имя теста|

```Kusto
availabilityResults 
| summarize sum(todouble(success == 1) * 100) / count() by bin(timestamp, 5m), location
| render timechart
```

### <a name="availability-test-duration-availabilityresultsduration"></a>Длительность теста доступности (availabilityResults/Duration)

Метрика " *длительность теста доступности* " показывает, сколько времени заняло выполнение веб-теста. Для [многошаговых веб-тестов](../app/availability-multistep.md)метрика отражает общее время выполнения всех шагов.

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|---|---|---|
|Миллисекунды|Average, min, Max|Расположение запуска, имя теста, результат теста

```Kusto
availabilityResults
| where notempty(duration)
| extend availabilityResult_duration = iif(itemType == 'availabilityResult', duration, todouble(''))
| summarize sum(availabilityResult_duration)/sum(itemCount) by bin(timestamp, 5m), location
| render timechart
```

### <a name="availability-tests-availabilityresultscount"></a>Тесты доступности (availabilityResults/Count)

Метрика *тесты доступности* отражает количество веб-тестов, выполняемых Azure Monitor.

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|---|---|---|
|Count|Count|Расположение запуска, имя теста, результат теста|

```Kusto
availabilityResults
| summarize sum(itemCount) by bin(timestamp, 5m)
| render timechart
```

## <a name="browser-metrics"></a>Метрики браузера

Метрики браузера собираются Application Insightsным пакетом SDK JavaScript из реальных браузеров конечных пользователей. Они предоставляют ценные сведения о работе пользователей с вашим веб-приложением. Метрики браузера обычно не вычисляются. Это означает, что они обеспечивают большую точность чисел использования по сравнению с метриками на стороне сервера, которые могут быть отклонены выборкой.

> [!NOTE]
> Для сбора метрик браузера приложение должно быть инструментировано с помощью [пакета SDK для Application Insights JavaScript](../app/javascript.md).

### <a name="browser-page-load-time-browsertimingstotalduration"></a>Время загрузки страницы браузера (Бровсертимингс/Тоталдуратион)

|Единица измерения|Поддерживаемые агрегаты|Предварительно агрегированные измерения|
|---|---|---|
|Миллисекунды|Average, min, Max|None|

```Kusto
browserTimings
| where notempty(totalDuration)
| extend _sum = totalDuration
| extend _count = itemCount
| extend _sum = _sum * _count
| summarize sum(_sum) / sum(_count) by bin(timestamp, 5m)
| render timechart
```

### <a name="client-processing-time-browsertimingprocessingduration"></a>Время обработки клиента (Бровсертиминг/Процессингдуратион)

|Единица измерения|Поддерживаемые агрегаты|Предварительно агрегированные измерения|
|---|---|---|
|Миллисекунды|Average, min, Max|None|

```Kusto
browserTimings
| where notempty(processingDuration)
| extend _sum = processingDuration
| extend _count = itemCount
| extend _sum = _sum * _count
| summarize sum(_sum)/sum(_count) by bin(timestamp, 5m)
| render timechart
```

### <a name="page-load-network-connect-time-browsertimingsnetworkduration"></a>Время подключения к сети на странице (Бровсертимингс/Нетворкдуратион)

|Единица измерения|Поддерживаемые агрегаты|Предварительно агрегированные измерения|
|---|---|---|
|Миллисекунды|Average, min, Max|None|

```Kusto
browserTimings
| where notempty(networkDuration)
| extend _sum = networkDuration
| extend _count = itemCount
| extend _sum = _sum * _count
| summarize sum(_sum) / sum(_count) by bin(timestamp, 5m)
| render timechart
```

### <a name="receiving-response-time-browsertimingsreceiveduration"></a>Получение времени ответа (Бровсертимингс/Рецеиведуратион)

|Единица измерения|Поддерживаемые агрегаты|Предварительно агрегированные измерения|
|---|---|---|
|Миллисекунды|Average, min, Max|None|

```Kusto
browserTimings
| where notempty(receiveDuration)
| extend _sum = receiveDuration
| extend _count = itemCount
| extend _sum = _sum * _count
| summarize sum(_sum) / sum(_count) by bin(timestamp, 5m)
| render timechart
```

### <a name="send-request-time-browsertimingssendduration"></a>Время запроса на отправку (Бровсертимингс/Сенддуратион)

|Единица измерения|Поддерживаемые агрегаты|Предварительно агрегированные измерения|
|---|---|---|
|Миллисекунды|Average, min, Max|None|

```Kusto
browserTimings
| where notempty(sendDuration)
| extend _sum = sendDuration
| extend _count = itemCount
| extend _sum = _sum * _count
| summarize sum(_sum) / sum(_count) by bin(timestamp, 5m)
| render timechart
```

## <a name="failure-metrics"></a>Метрики сбоев

Метрики в **сбоях** показывают проблемы с запросами обработки, вызовами зависимостей и вызванными исключениями.

### <a name="browser-exceptions-exceptionsbrowser"></a>Исключения браузера (исключения и браузер)

Эта метрика отражает число вызванных исключений из кода приложения, выполняемого в браузере. В метрику включаются только те исключения, которые были записаны с помощью ```trackException()``` вызова API Application Insights.

|Единица измерения|Поддерживаемые агрегаты|Предварительно агрегированные измерения|Примечания|
|---|---|---|---|
|Count|Count|None|В версии на основе журнала используется агрегирование **сумм**|

```Kusto
exceptions
| where notempty(client_Browser)
| summarize sum(itemCount) by bin(timestamp, 5m)
| render barchart
```

### <a name="dependency-call-failures-dependenciesfailed"></a>Сбои при вызове зависимостей (зависимости/сбой)

Число неудачных вызовов зависимостей.

|Единица измерения|Поддерживаемые агрегаты|Предварительно агрегированные измерения|Примечания|
|---|---|---|---|
|Count|Count|None|В версии на основе журнала используется агрегирование **сумм**|

```Kusto
dependencies
| where success == 'False'
| summarize sum(itemCount) by bin(timestamp, 5m)
| render barchart
```

### <a name="exceptions-exceptionscount"></a>Исключения (исключения/число)

Каждый раз, когда вы регистрируете исключение в Application Insights, вызывается [метод trackException ()](../app/api-custom-events-metrics.md#trackexception) пакета SDK. Метрика исключения показывает количество зарегистрированных исключений.

|Единица измерения|Поддерживаемые агрегаты|Предварительно агрегированные измерения|Примечания|
|---|---|---|---|
|Count|Count|Имя роли облака, экземпляр облачной роли, тип устройства|В версии на основе журнала используется агрегирование **сумм**|

```Kusto
exceptions
| summarize sum(itemCount) by bin(timestamp, 5m)
| render barchart
```

### <a name="failed-requests-requestsfailed"></a>Неудачные запросы (запросы или сбой)

Число отслеживаний запросов к серверу, отмеченных как *сбойные*. По умолчанию пакет SDK Application Insights автоматически помечает каждый запрос сервера, который вернул код ответа HTTP 5xx или 4xx в качестве невыполненного запроса. Эту логику можно настроить, изменив свойство  *Success* элемента телеметрии запроса в [пользовательском инициализаторе телеметрии](../app/api-filtering-sampling.md#addmodify-properties-itelemetryinitializer).

|Единица измерения|Поддерживаемые агрегаты|Предварительно агрегированные измерения|Примечания|
|---|---|---|---|
|Count|Count|Экземпляр роли облака, имя облачной роли, реальный или искусственный трафик, производительность запросов, код ответа|В версии на основе журнала используется агрегирование **сумм**|

```Kusto
requests
| where success == 'False'
| summarize sum(itemCount) by bin(timestamp, 5m)
| render barchart
```

### <a name="server-exceptions-exceptionsserver"></a>Исключения сервера (исключения/сервер)

Эта метрика показывает количество исключений сервера.

|Единица измерения|Поддерживаемые агрегаты|Предварительно агрегированные измерения|Примечания|
|---|---|---|---|
|Count|Count|Имя роли облака, экземпляр облачной роли|В версии на основе журнала используется агрегирование **сумм**|

```Kusto
exceptions
| where isempty(client_Browser)
| summarize sum(itemCount) by bin(timestamp, 5m)
| render barchart
```

## <a name="performance-counters"></a>Счетчики производительности

Используйте метрики в категории **счетчики производительности** для доступа к [счетчикам производительности системы, собранным Application Insights](../app/performance-counters.md).

### <a name="available-memory-performancecountersavailablememory"></a>Доступная память (performanceCounters/Аваилаблемемори)

```Kusto
performanceCounters
| where ((category == "Memory" and counter == "Available Bytes") or name == "availableMemory")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="exception-rate-performancecountersexceptionrate"></a>Частота исключений (performanceCounters/Ексцептионрате)

```Kusto
performanceCounters
| where ((category == ".NET CLR Exceptions" and counter == "# of Exceps Thrown / sec") or name == "exceptionRate")
| extend performanceCounter_value = iif(itemType == 'performanceCounter',value,todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="http-request-execution-time-performancecountersrequestexecutiontime"></a>Время выполнения HTTP-запроса (performanceCounters/Рекуестексекутионтиме)

```Kusto
performanceCounters
| where ((category == "ASP.NET Applications" and counter == "Request Execution Time") or name == "requestExecutionTime")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="http-request-rate-performancecountersrequestspersecond"></a>Частота запросов HTTP (performanceCounters/Рекуестсперсеконд)

```Kusto
performanceCounters
| where ((category == "ASP.NET Applications" and counter == "Requests/Sec") or name == "requestsPerSecond")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="http-requests-in-application-queue-performancecountersrequestsinqueue"></a>HTTP-запросы в очереди приложений (performanceCounters/Рекуестсинкуеуе)

```Kusto
performanceCounters
| where ((category == "ASP.NET Applications" and counter == "Requests In Application Queue") or name == "requestsInQueue")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="process-cpu-performancecountersprocesscpupercentage"></a>ЦП процесса (performanceCounters/Процесскпуперцентаже)

Метрика показывает, какая часть общей мощности процессора потребляется процессом, на котором размещается отслеживаемое приложение.

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|
|Процент|Average, min, Max|Экземпляр облачной роли

```Kusto
performanceCounters
| where ((category == "Process" and counter == "% Processor Time Normalized") or name == "processCpuPercentage")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="process-io-rate-performancecountersprocessiobytespersecond"></a>Скорость ввода-вывода процесса (performanceCounters/Процессиобитесперсеконд)

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|
|Байт в секунду|Average, min, Max|Экземпляр облачной роли

```Kusto
performanceCounters
| where ((category == "Process" and counter == "IO Data Bytes/sec") or name == "processIOBytesPerSecond")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="process-private-bytes-performancecountersprocessprivatebytes"></a>Байт исключительного выполнения процесса (performanceCounters/Процессприватебитес)

Объем необщей памяти, выделенный отслеживаемым процессом для данных.

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|
|Байты|Average, min, Max|Экземпляр облачной роли

```Kusto
performanceCounters
| where ((category == "Process" and counter == "Private Bytes") or name == "processPrivateBytes")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

### <a name="processor-time-performancecountersprocessorcpupercentage"></a>Процессорное время (performanceCounters/Процессоркпуперцентаже)

Использование ЦП *всеми* процессами, работающими на экземпляре наблюдаемого сервера.

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|
|Процент|Average, min, Max|Экземпляр облачной роли

>[!NOTE]
> Метрика "процессорное время" недоступна для приложений, размещенных в службах приложений Azure. Используйте показатель  [ЦП процесса](#process-cpu-performancecountersprocesscpupercentage) для мониторинга загрузки ЦП веб-приложений, размещенных в службах приложений.

```Kusto
performanceCounters
| where ((category == "Processor" and counter == "% Processor Time") or name == "processorCpuPercentage")
| extend performanceCounter_value = iif(itemType == "performanceCounter", value, todouble(''))
| summarize sum(performanceCounter_value) / count() by bin(timestamp, 1m)
| render timechart
```

## <a name="server-metrics"></a>Метрики сервера

### <a name="dependency-calls-dependenciescount"></a>Вызовы зависимостей (зависимости/число)

Эта метрика зависит от количества вызовов зависимостей.

```Kusto
dependencies
| summarize sum(itemCount) by bin(timestamp, 5m)
| render barchart
```

### <a name="dependency-duration-dependenciesduration"></a>Длительность зависимости (зависимости/длительность)

Эта метрика относится к длительности вызовов зависимостей.

```Kusto
dependencies
| where notempty(duration)
| extend dependency_duration = iif(itemType == 'dependency',duration,todouble(''))
| extend _sum = dependency_duration
| extend _count = itemCount
| extend _sum = _sum*_count
| summarize sum(_sum)/sum(_count) by bin(timestamp, 1m)
| render timechart
```

### <a name="server-requests-requestscount"></a>Запросы сервера (запросов/счетчиков)

Эта метрика отражает количество входящих запросов сервера, полученных веб-приложением.

```Kusto
requests
| summarize sum(itemCount) by bin(timestamp, 5m)
| render barchart
```

### <a name="server-response-time-requestsduration"></a>Время ответа сервера (запросов в течение длительности)

Эта метрика отражает время, затраченное на обработку входящих запросов серверами.

```Kusto
requests
| where notempty(duration)
| extend request_duration = iif(itemType == 'request', duration, todouble(''))
| extend _sum = request_duration
| extend _count = itemCount
| extend _sum = _sum*_count
| summarize sum(_sum) / sum(_count) by bin(timestamp, 1m)
| render timechart
```

## <a name="usage-metrics"></a>Метрики использования

### <a name="page-view-load-time-pageviewsduration"></a>Время загрузки просмотра страницы (pageViews/Duration)

Эта метрика относится к количеству времени, затраченного на загрузку событий PageView.

```Kusto
pageViews
| where notempty(duration)
| extend pageView_duration = iif(itemType == 'pageView', duration, todouble(''))
| extend _sum = pageView_duration
| extend _count = itemCount
| extend _sum = _sum * _count
| summarize sum(_sum) / sum(_count) by bin(timestamp, 5m)
| render barchart
```

### <a name="page-views-pageviewscount"></a>Просмотры страниц (pageViews/Count)

Число событий PageView, регистрируемых с помощью API TrackPageView () Application Insights.

```Kusto
pageViews
| summarize sum(itemCount) by bin(timestamp, 1h)
| render barchart
```

### <a name="sessions-sessionscount"></a>Сеансы (сеансы/число)

Эта метрика относится к числу уникальных идентификаторов сеансов.

```Kusto
union traces, requests, pageViews, dependencies, customEvents, availabilityResults, exceptions, customMetrics, browserTimings
| where notempty(session_Id)
| summarize dcount(session_Id) by bin(timestamp, 1h)
| render barchart
```

### <a name="traces-tracescount"></a>Трассировки (трассировка/число)

Число трассировочных инструкций, регистрируемых с помощью вызова API TrackTrace () Application Insights.

```Kusto
traces
| summarize sum(itemCount) by bin(timestamp, 1h)
| render barchart
```

### <a name="users-userscount"></a>Пользователи (пользователи/число)

Число уникальных пользователей, которые обращаются к приложению. Точность этой метрики может значительно повлиять на выборку и фильтрацию данных телеметрии.

```Kusto
union traces, requests, pageViews, dependencies, customEvents, availabilityResults, exceptions, customMetrics, browserTimings
| where notempty(user_Id)
| summarize dcount(user_Id) by bin(timestamp, 1h)
| render barchart
```

### <a name="users-authenticated-usersauthenticated"></a>Пользователи, проверка подлинности (пользователи и проверка подлинности)

Количество уникальных пользователей, которые прошли проверку подлинности в вашем приложении.

```Kusto
union traces, requests, pageViews, dependencies, customEvents, availabilityResults, exceptions, customMetrics, browserTimings
| where notempty(user_AuthenticatedId)
| summarize dcount(user_AuthenticatedId) by bin(timestamp, 1h)
| render barchart
```