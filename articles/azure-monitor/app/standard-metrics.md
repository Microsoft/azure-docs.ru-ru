---
title: Стандартные метрики Azure Application Insights | Документация Майкрософт
description: В этой статье перечислены метрики Application Insights Azure с поддерживаемыми агрегатами и измерениями.
services: azure-monitor
ms.topic: reference
ms.date: 07/03/2019
ms.subservice: application-insights
ms.openlocfilehash: 0a18088fa434efa76007607c067feec107bdae57
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100572355"
---
# <a name="application-insights-standard-metrics"></a>Application Insights стандартных метрик

Стандартные метрики предварительно агрегированы во время сбора, они имеют лучшую производительность во время выполнения запроса. Это делает их лучшим выбором для панелей мониторинга и оповещений в режиме реального времени.

## <a name="availability-metrics"></a>Метрики доступности

Метрики в категории доступности позволяют просматривать работоспособность веб-приложения с точки зрения мира. [Настройте тесты доступности](../app/monitor-web-app-availability.md) , чтобы начать использовать метрики из этой категории.

### <a name="availability-availabilityresultsavailabilitypercentage"></a>Доступность (availabilityResults/Аваилабилитиперцентаже)
Метрика *доступности* показывает процент запусков веб-тестов, которые не обнаруживают проблемы. Наименьшее возможное значение равно 0, что означает сбой всех выполнений веб-тестов. Значение 100 означает, что все выполненные веб-тесты прошли условия проверки.

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|---|---|---|
|Процент|Среднее значение|`Run location`, `Test name`|


### <a name="availability-test-duration-availabilityresultsduration"></a>Длительность теста доступности (availabilityResults/Duration)

Метрика " *длительность теста доступности* " показывает, сколько времени заняло выполнение веб-теста. Для [многошаговых веб-тестов](../app/availability-multistep.md)метрика отражает общее время выполнения всех шагов.

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|---|---|---|
|Миллисекунды|Average, min, Max|`Run location`, `Test name`, `Test result`

### <a name="availability-tests-availabilityresultscount"></a>Тесты доступности (availabilityResults/Count)

Метрика *тесты доступности* отражает количество веб-тестов, выполняемых Azure Monitor.

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|---|---|---|
|Count|Count|`Run location`, `Test name`, `Test result`|


## <a name="browser-metrics"></a>Метрики браузера

Метрики браузера собираются Application Insightsным пакетом SDK JavaScript из реальных браузеров конечных пользователей. Они предоставляют ценные сведения о работе пользователей с вашим веб-приложением. Метрики браузера обычно не вычисляются. Это означает, что они обеспечивают большую точность чисел использования по сравнению с метриками на стороне сервера, которые могут быть отклонены выборкой.

> [!NOTE]
> Для сбора метрик браузера приложение должно быть инструментировано с помощью [пакета SDK для Application Insights JavaScript](../app/javascript.md).

### <a name="browser-page-load-time-browsertimingstotalduration"></a>Время загрузки страницы браузера (Бровсертимингс/Тоталдуратион)

|Единица измерения|Поддерживаемые агрегаты| Поддерживаемые измерения|
|---|---|---|
|Миллисекунды|Average, min, Max|Нет|

### <a name="client-processing-time-browsertimingprocessingduration"></a>Время обработки клиента (Бровсертиминг/Процессингдуратион)

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|
|Миллисекунды|Average, min, Max|Нет|

### <a name="page-load-network-connect-time-browsertimingsnetworkduration"></a>Время подключения к сети на странице (Бровсертимингс/Нетворкдуратион)

|Единица измерения|Поддерживаемые агрегаты| Поддерживаемые измерения|
|---|---|---|
|Миллисекунды|Average, min, Max|Нет|

### <a name="receiving-response-time-browsertimingsreceiveduration"></a>Получение времени ответа (Бровсертимингс/Рецеиведуратион)

|Единица измерения|Поддерживаемые агрегаты| Поддерживаемые измерения|
|---|---|---|
|Миллисекунды|Average, min, Max|Нет|

### <a name="send-request-time-browsertimingssendduration"></a>Время запроса на отправку (Бровсертимингс/Сенддуратион)

|Единица измерения|Поддерживаемые агрегаты| Поддерживаемые измерения|
|---|---|---|
|Миллисекунды|Average, min, Max|Нет|

## <a name="failure-metrics"></a>Метрики сбоев

Метрики в **сбоях** показывают проблемы с запросами обработки, вызовами зависимостей и вызванными исключениями.

### <a name="browser-exceptions-exceptionsbrowser"></a>Исключения браузера (исключения и браузер)

Эта метрика отражает число вызванных исключений из кода приложения, выполняемого в браузере. В метрику включаются только те исключения, которые были записаны с помощью ```trackException()``` вызова API Application Insights.

|Единица измерения|Поддерживаемые агрегаты | Поддерживаемые измерения|
|---|---|---|---|
| Count | Count | `Cloud role name` |

### <a name="dependency-call-failures-dependenciesfailed"></a>Сбои при вызове зависимостей (зависимости/сбой)

Число неудачных вызовов зависимостей.

|Единица измерения|Поддерживаемые агрегаты | Поддерживаемые измерения |
|---|---|---|---|
|Count|Count| `Cloud role instance`, `Cloud role name`, `Dependency performance`, `Dependency type`, `Is traffic synthetic`, `Result code`, `Rarget of dependency call`.


### <a name="exceptions-exceptionscount"></a>Исключения (исключения/число)

Каждый раз, когда вы регистрируете исключение в Application Insights, вызывается [метод trackException ()](../app/api-custom-events-metrics.md#trackexception) пакета SDK. Метрика исключения показывает количество зарегистрированных исключений.

|Единица измерения|Поддерживаемые агрегаты | Поддерживаемые измерения |
|---|---|---|---|
|Count|Count|`Cloud role instance`, `Cloud role name`, `Device type`|

### <a name="failed-requests-requestsfailed"></a>Неудачные запросы (запросы или сбой)

Число отслеживаний запросов к серверу, отмеченных как *сбойные*. По умолчанию пакет SDK Application Insights автоматически помечает каждый запрос сервера, который вернул код ответа HTTP 5xx или 4xx в качестве невыполненного запроса. Эту логику можно настроить, изменив свойство  *Success* элемента телеметрии запроса в [пользовательском инициализаторе телеметрии](../app/api-filtering-sampling.md#addmodify-properties-itelemetryinitializer).


|Единица измерения|Поддерживаемые агрегаты | Поддерживаемые измерения |
|---|---|---|---|
|Count|Count|`Cloud role instance`, `Cloud role name`, `Is synthetic traffic`, `Request performance`, `Result code`|


### <a name="server-exceptions-exceptionsserver"></a>Исключения сервера (исключения/сервер)

Эта метрика показывает количество исключений сервера.

|Единица измерения|Поддерживаемые агрегаты | Поддерживаемые измерения |
|---|---|---|---|
|Count|Count|`Cloud role instance`, `Cloud role name`|

## <a name="performance-counters"></a>Счетчики производительности

Используйте метрики в категории **счетчики производительности** для доступа к [счетчикам производительности системы, собранным Application Insights](../app/performance-counters.md).

### <a name="available-memory-performancecountersavailablememory"></a>Доступная память (performanceCounters/Аваилаблемемори)

|Единица измерения|Поддерживаемые агрегаты | Поддерживаемые измерения |
|---|---|---|---|
|Зависимые данные: мегабайты, гигабайты|Average, Max, min|`Cloud role instance`|


### <a name="exception-rate-performancecountersexceptionrate"></a>Частота исключений (performanceCounters/Ексцептионрате)

|Единица измерения|Поддерживаемые агрегаты | Поддерживаемые измерения |
|---|---|---|---|
| Count | Average, Max, min | `Cloud role instance` |


### <a name="http-request-execution-time-performancecountersrequestexecutiontime"></a>Время выполнения HTTP-запроса (performanceCounters/Рекуестексекутионтиме)

|Единица измерения|Поддерживаемые агрегаты | Поддерживаемые измерения |
|---|---|---|---|
| Миллисекунды | Average, Max, min | `Cloud role instance` |

### <a name="http-request-rate-performancecountersrequestspersecond"></a>Частота запросов HTTP (performanceCounters/Рекуестсперсеконд)

|Единица измерения|Поддерживаемые агрегаты | Поддерживаемые измерения |
|---|---|---|---|
| Число запросов в секунду | Average, Max, min | `Cloud role instance` |

### <a name="http-requests-in-application-queue-performancecountersrequestsinqueue"></a>HTTP-запросы в очереди приложений (performanceCounters/Рекуестсинкуеуе)

|Единица измерения|Поддерживаемые агрегаты | Поддерживаемые измерения |
|---|---|---|---|
| Count | Average, Max, min | `Cloud role instance` |


### <a name="process-cpu-performancecountersprocesscpupercentage"></a>ЦП процесса (performanceCounters/Процесскпуперцентаже)

Метрика показывает, какая часть общей мощности процессора потребляется процессом, на котором размещается отслеживаемое приложение.

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|
|Процент|Average, Max, min| `Cloud role instance` |


### <a name="process-io-rate-performancecountersprocessiobytespersecond"></a>Скорость ввода-вывода процесса (performanceCounters/Процессиобитесперсеконд)

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|
|Байт в секунду|Average, min, Max|`Cloud role instance` |



### <a name="process-private-bytes-performancecountersprocessprivatebytes"></a>Байт исключительного выполнения процесса (performanceCounters/Процессприватебитес)

Объем необщей памяти, выделенный отслеживаемым процессом для данных.

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|
|Байты|Average, min, Max|`Cloud role instance` |

### <a name="processor-time-performancecountersprocessorcpupercentage"></a>Процессорное время (performanceCounters/Процессоркпуперцентаже)

Использование ЦП *всеми* процессами, работающими на экземпляре наблюдаемого сервера.

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|
|Процент|Average, min, Max|`Cloud role instance` |

>[!NOTE]
> Метрика "процессорное время" недоступна для приложений, размещенных в службах приложений Azure. Используйте показатель  [ЦП процесса](#process-cpu-performancecountersprocesscpupercentage) для мониторинга загрузки ЦП веб-приложений, размещенных в службах приложений.

## <a name="server-metrics"></a>Метрики сервера

### <a name="dependency-calls-dependenciescount"></a>Вызовы зависимостей (зависимости/число)

Эта метрика зависит от количества вызовов зависимостей.

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|
| Count | Count | `Cloud role instance`, `Cloud role name`, `Dependency performance`, `Dependency type`, `Is traffic synthetic`, `Result code`, `Successful call`, `Target of a dependency call` |

### <a name="dependency-duration-dependenciesduration"></a>Длительность зависимости (зависимости/длительность)

Эта метрика относится к длительности вызовов зависимостей.

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|
| время; | Average, min, Max | `Cloud role instance`, `Cloud role name`, `Dependency performance`, `Dependency type`, `Is traffic synthetic`, `Result code`, `Successful call`, `Target of a dependency call` |


### <a name="server-request-rate-requestscount"></a>Частота запросов сервера (запросов/подсчетов)

Эта метрика отражает количество входящих запросов сервера, полученных веб-приложением.

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|
| Count | Среднее | `Cloud role instance`, `Cloud role name`, `Is traffic synthetic`, `Result performance` `Result code`, `Successful request` |

### <a name="server-requests-requestscount"></a>Запросы сервера (запросов/счетчиков)

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|
| Count | Count | `Cloud role instance`, `Cloud role name`, `Is traffic synthetic`, `Result performance` `Result code`, `Successful request` |

### <a name="server-response-time-requestsduration"></a>Время ответа сервера (запросов в течение длительности)

Эта метрика отражает время, затраченное на обработку входящих запросов серверами.

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|
| время; | Average, min, Max | `Cloud role instance`, `Cloud role name`, `Is traffic synthetic`, `Result performance` `Result code`, `Successful request` |

## <a name="usage-metrics"></a>Метрики использования

### <a name="page-view-load-time-pageviewsduration"></a>Время загрузки просмотра страницы (pageViews/Duration)

Эта метрика относится к количеству времени, затраченного на загрузку событий PageView.

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|
| время; | Average, min, Max | `Cloud role name`, `Is traffic synthetic` |

### <a name="page-views-pageviewscount"></a>Просмотры страниц (pageViews/Count)

Число событий PageView, регистрируемых с помощью API TrackPageView () Application Insights.

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|
| Count | Count | `Cloud role name`, `Is traffic synthetic` |

### <a name="traces-tracescount"></a>Трассировки (трассировка/число)

Число трассировочных инструкций, регистрируемых с помощью вызова API TrackTrace () Application Insights.

|Единица измерения|Поддерживаемые агрегаты|Поддерживаемые измерения|
|---|---|---|
| Count | Count | `Cloud role instance`, `Cloud role name`,  `Is traffic synthetic`, `Severity level` |


## <a name="next-steps"></a>Дальнейшие действия

* Сведения об [метриках на основе журналов и предварительно агрегированных](./pre-aggregated-metrics-log-metrics.md)данных.
* [Запросы и определения метрик на основе журнала](../essentials/app-insights-metrics.md).