---
title: Модель данных Azure Application Insights | Документация Майкрософт
description: Описание свойств, экспортируемых с помощью непрерывного экспорта в формате JSON и используемых в качестве фильтров.
ms.topic: conceptual
ms.date: 01/08/2019
ms.openlocfilehash: b4609d54c1c3c33a654dd58a3bceaca4974fda15
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100584221"
---
# <a name="application-insights-export-data-model"></a>Экспорт модели данных Application Insights
В этой таблице перечислены свойства телеметрии, отправляемой из различных пакетов SDK для [Application Insights](./app-insights-overview.md) на портал.
Вы увидите эти свойства в выходных данных [непрерывного экспорта](export-telemetry.md).
Они также отображаются в фильтрах свойств в [обозревателе метрик](../essentials/metrics-charts.md) и при [диагностическом поиске](./diagnostic-search.md).

Примечания:

* `[0]` в этих таблицах обозначает точку в пути, куда необходимо вставить индекс. При этом значение не всегда равно 0.
* Продолжительность времени указана в десятых долях микросекунды, поэтому 10 000 000 = 1 с.
* Значения даты и времени в формате UTC указаны в формате ISO `yyyy-MM-DDThh:mm:ss.sssZ`

## <a name="example"></a>Пример

```json
// A server report about an HTTP request
{
  "request": [
    {
      "urlData": { // derived from 'url'
        "host": "contoso.org",
        "base": "/",
        "hashTag": ""
      },
      "responseCode": 200, // Sent to client
      "success": true, // Default == responseCode<400
      // Request id becomes the operation id of child events
      "id": "fCOhCdCnZ9I=",  
      "name": "GET Home/Index",
      "count": 1, // 100% / sampling rate
      "durationMetric": {
        "value": 1046804.0, // 10000000 == 1 second
        // Currently the following fields are redundant:
        "count": 1.0,
        "min": 1046804.0,
        "max": 1046804.0,
        "stdDev": 0.0,
        "sampledValue": 1046804.0
      },
      "url": "/"
    }
  ],
  "internal": {
    "data": {
      "id": "7f156650-ef4c-11e5-8453-3f984b167d05",
      "documentVersion": "1.61"
    }
  },
  "context": {
    "device": { // client browser
      "type": "PC",
      "screenResolution": { },
      "roleInstance": "WFWEB14B.fabrikam.net"
    },
    "application": { },
    "location": { // derived from client ip
      "continent": "North America",
      "country": "United States",
      // last octagon is anonymized to 0 at portal:
      "clientip": "168.62.177.0",
      "province": "",
      "city": ""
    },
    "data": {
      "isSynthetic": true, // we identified source as a bot
      // percentage of generated data sent to portal:
      "samplingRate": 100.0,
      "eventTime": "2016-03-21T10:05:45.7334717Z" // UTC
    },
    "user": {
      "isAuthenticated": false,
      "anonId": "us-tx-sn1-azr", // bot agent id
      "anonAcquisitionDate": "0001-01-01T00:00:00Z",
      "authAcquisitionDate": "0001-01-01T00:00:00Z",
      "accountAcquisitionDate": "0001-01-01T00:00:00Z"
    },
    "operation": {
      "id": "fCOhCdCnZ9I=",
      "parentId": "fCOhCdCnZ9I=",
      "name": "GET Home/Index"
    },
    "cloud": { },
    "serverDevice": { },
    "custom": { // set by custom fields of track calls
      "dimensions": [ ],
      "metrics": [ ]
    },
    "session": {
      "id": "65504c10-44a6-489e-b9dc-94184eb00d86",
      "isFirst": true
    }
  }
}
```

## <a name="context"></a>Контекст
Для каждого типа данных телеметрии приведен пример с разделом контекста. Не все эти поля передаются со всеми точками данных.

| Путь | Type | Примечания |
| --- | --- | --- |
| context.custom.dimensions [0] |объект [ ] |Набор пар "ключ — значение", заданный параметром пользовательских свойств. Максимальная длина ключа — 100, максимальная длина значения —1024. Более 100 уникальных значений. Свойства можно использовать для поиска, но не для сегментации. Максимальное количество — 200 ключей на ключ ikey. |
| context.custom.metrics [0] |объект [ ] |Набор пар "ключ — значение", заданный параметром пользовательских измерений и метриками TrackMetric. Максимальная длина ключа — 100. Значения могут быть числовыми. |
| context.data.eventTime |строка |Формат UTC. |
| context.data.isSynthetic |Логическое |Запрос поступает от программы-робота или веб-теста. |
| context.data.samplingRate |Число |Процентная доля данных телеметрии, созданных с помощью пакета SDK, отправленного на портал. Диапазон 0,0–100,0. |
| context.device |object |Устройство клиента |
| context.device.browser |строка |IE, Chrome… |
| context.device.browserVersion |строка |Chrome 48.0… |
| context.device.deviceModel |строка | |
| context.device.deviceName |строка | |
| context.device.id |строка | |
| context.device.locale |строка |en-GB, de-DE… |
| context.device.network |строка | |
| context.device.oemName |строка | |
| context.device.os |строка | |
| context.device.osVersion |строка |ОС узла |
| context.device.roleInstance |строка |Идентификатор узла сервера |
| context.device.roleName |строка | |
| context.device.screenResolution |строка | |
| context.device.type |строка |ПК, браузер… |
| context.location |object |Производное от `clientip`. |
| context.location.city |строка |Производный от `clientip` , если известен |
| context.location.clientip |строка |Последний восьмиугольник анонимизирован и имеет значение 0. |
| context.location.continent |строка | |
| context.location.country |строка | |
| context.location.province |строка |Страна или область |
| context.operation.id |строка |Элементы, имеющие одинаковые, отображаются на `operation id` портале как связанные элементы. Обычно `request id` . |
| context.operation.name |строка |URL-адрес или имя запроса |
| context.operation.parentId |строка |Разрешает использование вложенных связанных элементов. |
| context.session.id |строка |`Id` группы операций из одного источника. 30-минутный период без операций указывает на завершение сеанса. |
| context.session.isFirst |Логическое | |
| context.user.accountAcquisitionDate |строка | |
| context.user.accountId |строка | |
| context.user.anonAcquisitionDate |строка | |
| context.user.anonId |строка | |
| context.user.authAcquisitionDate |строка |[прошедший проверку пользователь](./api-custom-events-metrics.md#authenticated-users) |
| context.user.authId |строка | |
| context.user.isAuthenticated |Логическое | |
| context.user.storeRegion |строка | |
| internal.data.documentVersion |строка | |
| internal.data.id |строка | `Unique id` , который назначается при приеме элемента в Application Insights |

## <a name="events"></a>События
Пользовательские события, создаваемые элементом [TrackEvent()](./api-custom-events-metrics.md#trackevent).

| Путь | Type | Примечания |
| --- | --- | --- |
| event [0] count |Целое число |100/(частота[выборки](./sampling.md) ). Например, 4 = &gt; 25 %. |
| event [0] name |строка |Имя события.  Максимальная длина: 250 |
| event [0] url |строка | |
| event [0] urlData.base |строка | |
| event [0] urlData.host |строка | |

## <a name="exceptions"></a>Исключения
Отправляются сведения об [исключениях](./asp-net-exceptions.md) на сервере и в браузере.

| Путь | Type | Примечания |
| --- | --- | --- |
| basicException [0] assembly |строка | |
| basicException [0] count |Целое число |100/(частота[выборки](./sampling.md) ). Например, 4 = &gt; 25 %. |
| basicException [0] exceptionGroup |строка | |
| basicException [0] exceptionType |строка | |
| basicException [0] failedUserCodeMethod |строка | |
| basicException [0] failedUserCodeAssembly |строка | |
| basicException [0] handledAt |строка | |
| basicException [0] hasFullStack |Логическое | |
| Басицексцептион [0] `id` |строка | |
| basicException [0] method |строка | |
| basicException [0] message |строка |Сообщение об исключении. Максимальная длина: 10 000 |
| basicException [0] outerExceptionMessage |строка | |
| basicException [0] outerExceptionThrownAtAssembly |строка | |
| basicException [0] outerExceptionThrownAtMethod |строка | |
| basicException [0] outerExceptionType |строка | |
| basicException [0] outerId |строка | |
| basicException [0] parsedStack [0] assembly |строка | |
| basicException [0] parsedStack [0] fileName |строка | |
| basicException [0] parsedStack [0] level |Целое число | |
| basicException [0] parsedStack [0] line |Целое число | |
| basicException [0] parsedStack [0] method |строка | |
| basicException [0] stack |строка |Максимальная длина: 10 000 |
| basicException [0] typeName |строка | |

## <a name="trace-messages"></a>Сообщения трассировки
Отправитель: [TrackTrace](./api-custom-events-metrics.md#tracktrace) и [адаптеры ведения журналов](./asp-net-trace-logs.md).

| Путь | Type | Примечания |
| --- | --- | --- |
| message [0] loggerName |строка | |
| message [0] parameters |строка | |
| message [0] raw |строка |Сообщение журнала, максимальная длина — 10 тысяч символов. |
| message [0] severityLevel |строка | |

## <a name="remote-dependency"></a>Удаленная зависимость
Отправитель: TrackDependency. Используется для создания отчетов о производительности и использовании [вызовов к зависимостям](./asp-net-dependencies.md) на сервере, а также вызовов AJAX в браузере.

| Путь | Type | Примечания |
| --- | --- | --- |
| remoteDependency [0] async |Логическое | |
| remoteDependency [0] baseName |строка | |
| remoteDependency [0] commandName |строка |Например, home/index |
| remoteDependency [0] count |Целое число |100/(частота[выборки](./sampling.md) ). Например, 4 = &gt; 25 %. |
| remoteDependency [0] dependencyTypeName |строка |HTTP, SQL, … |
| remoteDependency [0] durationMetric.value |Число |Время от вызова до завершения отклика зависимостью. |
| Ремотедепенденци [0] `id` |строка | |
| remoteDependency [0] name |строка |URL-адрес. Максимальная длина: 250 |
| remoteDependency [0] resultCode |строка |Из зависимости HTTP. |
| remoteDependency [0] success |Логическое | |
| remoteDependency [0] type |строка |HTTP, SQL, … |
| remoteDependency [0] url |строка |Максимальная длина: 2000 |
| remoteDependency [0] urlData.base |строка |Максимальная длина: 2000 |
| remoteDependency [0] urlData.hashTag |строка | |
| remoteDependency [0] urlData.host |строка |Максимальная длина: 200 |

## <a name="requests"></a>Requests
Отправитель: [TrackRequest](./api-custom-events-metrics.md#trackrequest). Используется стандартными модулями для создания отчетов о времени отклика сервера (измеряется на сервере).

| Путь | Type | Примечания |
| --- | --- | --- |
| request [0] count |Целое число |100/(частота[выборки](./sampling.md) ). Например: 4 =&gt; 25 %. |
| request [0] durationMetric.value |Число |Время от поступления запроса до отклика. 1e7 = 1 с. |
| запрос [0] `id` |строка |`Operation id` |
| request [0] name |строка |GET или POST + базовый URL-адрес.  Максимальная длина: 250 |
| request [0] responseCode |Целое число |HTTP-отклик, отправленный клиенту. |
| request [0] success |Логическое |Значение по умолчанию == (responseCode &lt; 400) |
| request [0] url |строка |Не включая узел. |
| request [0] urlData.base |строка | |
| request [0] urlData.hashTag |строка | |
| request [0] urlData.host |строка | |

## <a name="page-view-performance"></a>Производительность просмотра страницы
Отправитель: браузер. Измеряет время обработки страницы — с момента инициации пользователем запроса до полного отображения страницы (за исключением асинхронных вызовов AJAX).

Контекстные значения показывают версию клиентской ОС и версию браузера.

| Путь | Type | Примечания |
| --- | --- | --- |
| clientPerformance [0] clientProcess.value |Целое число |Время от завершения получения HTML до отображения страницы. |
| clientPerformance [0] name |строка | |
| clientPerformance [0] networkConnection.value |Целое число |Время, необходимое для установки подключения к сети. |
| clientPerformance [0] receiveRequest.value |Целое число |Время от завершения отправки запроса до получения HTML в отклике. |
| clientPerformance [0] sendRequest.value |Целое число |Время на отправку HTTP-запроса. |
| clientPerformance [0] total.value |Целое число |Время от запуска отправки запроса до отображения страницы. |
| clientPerformance [0] url |строка |URL-адрес запроса. |
| clientPerformance [0] urlData.base |строка | |
| clientPerformance [0] urlData.hashTag |строка | |
| clientPerformance [0] urlData.host |строка | |
| clientPerformance [0] urlData.protocol |строка | |

## <a name="page-views"></a>Просмотры страницы
Отправитель: trackPageView() или [stopTrackPage](./api-custom-events-metrics.md#page-views)

| Путь | Type | Примечания |
| --- | --- | --- |
| view [0] count |Целое число |100/(частота[выборки](./sampling.md) ). Например, 4 = &gt; 25 %. |
| view [0] durationMetric.value |Целое число |При необходимости значение можно указать в методе trackPageView() или с помощью метода start/stopTrackPage(). Не совпадает со значениями clientPerformance. |
| view [0] name |строка |Заголовок страницы.  Максимальная длина: 250 |
| view [0] url |строка | |
| view [0] urlData.base |строка | |
| view [0] urlData.hashTag |строка | |
| view [0] urlData.host |строка | |

## <a name="availability"></a>Доступность
Это свойство создает отчеты о [веб-тестах на доступность](./monitor-web-app-availability.md).

| Путь | Type | Примечания |
| --- | --- | --- |
| availability [0] availabilityMetric.name |строка |availability |
| availability [0] availabilityMetric.value |Число |1,0 или 0,0. |
| availability [0] count |Целое число |100/(частота[выборки](./sampling.md) ). Например, 4 = &gt; 25 %. |
| availability [0] dataSizeMetric.name |строка | |
| availability [0] dataSizeMetric.value |Целое число | |
| availability [0] durationMetric.name |строка | |
| availability [0] durationMetric.value |Число |Продолжительность теста. 1e7 = 1 с. |
| availability [0] message |строка |Диагностика сбоя. |
| availability [0] result |строка |Успех или сбой. |
| availability [0] runLocation |строка |Географический объект-источник HTTP-запроса. |
| availability [0] testName |строка | |
| availability [0] testRunId |строка | |
| availability [0] testTimestamp |строка | |

## <a name="metrics"></a>Метрики
Создатель: TrackMetric().

Значение метрики можно найти в context.custom.metrics[0].

Пример:

```json
{
  "metric": [ ],
  "context": {
  ...
    "custom": {
      "dimensions": [
        { "ProcessId": "4068" }
      ],
      "metrics": [
        {
          "dispatchRate": {
            "value": 0.001295,
            "count": 1.0,
            "min": 0.001295,
            "max": 0.001295,
            "stdDev": 0.0,
            "sampledValue": 0.001295,
            "sum": 0.001295
          }
        }
      ]  
    }
  }
}
```

## <a name="about-metric-values"></a>О значениях метрик
Значения метрик (как в отчетах, так и в других элементах) сообщаются в рамках стандартной структуры объекта. Пример:

```json
"durationMetric": {
  "name": "contoso.org",
  "type": "Aggregation",
  "value": 468.71603053650279,
  "count": 1.0,
  "min": 468.71603053650279,
  "max": 468.71603053650279,
  "stdDev": 0.0,
  "sampledValue": 468.71603053650279
}
```

Сейчас (хотя это может измениться в будущем) во всех значениях, включенных в отчеты стандартных модулей SDK, полезными являются только поля `name` и `value`, а также `count==1`. Они будут отличаться, только если вы напишете собственный вызов TrackMetric, указав другие параметры.

Другие поля нужны для того, чтобы разрешить статистическое вычисление метрик в пакете SDK, тем самым снизив нагрузку (в виде трафика) на портал. Например, перед отправкой каждого отчета с метриками вы можете получить среднее значение для нескольких последовательных показаний. Затем вы можете рассчитать минимальное и максимальное значение, а также стандартное отклонение и агрегированное значение (сумму или среднее), а затем указать в счетчике количество показаний, представленных в отчете.

В приведенных выше таблицах пропущены редко используемые поля Count, min, Max, stdDev и Сампледвалуе.

Вместо предварительного статистического вычисления метрик вы можете использовать [выборки](./sampling.md) , чтобы сократить объем данных телеметрии.

### <a name="durations"></a>Длительность
За исключением оговоренных случаев, показатели длительности представлены в десятых долях микросекунды, то есть 10 000 000,0 — это 1 с.

## <a name="see-also"></a>См. также раздел
* [Application Insights](./app-insights-overview.md)
* [Непрерывный экспорт](export-telemetry.md)
* [Примеры кода](export-telemetry.md#code-samples)

