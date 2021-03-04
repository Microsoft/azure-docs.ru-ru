---
title: Корреляция данных телеметрии Azure Application Insights | Документация Майкрософт
description: Корреляция данных телеметрии Application Insights
ms.topic: conceptual
author: lgayhardt
ms.author: lagayhar
ms.date: 06/07/2019
ms.reviewer: sergkanz
ms.custom: devx-track-python, devx-track-csharp
ms.openlocfilehash: beaeb0131a2c9b326d663f6fcbb8273a9b52b412
ms.sourcegitcommit: 4b7a53cca4197db8166874831b9f93f716e38e30
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102100973"
---
# <a name="telemetry-correlation-in-application-insights"></a>Корреляция данных телеметрии в Application Insights

В мире микрослужб для каждой логической операции требуется выполнение действий в различных компонентах службы. Каждый из этих компонентов можно отслеживать отдельно с помощью [Application Insights](../../azure-monitor/app/app-insights-overview.md). Application Insights поддерживает распределенную корреляцию данных телеметрии, позволяющую определить, какой компонент отвечает за сбой или снижение производительности.

В этой статье описывается модель данных, используемая службой Application Insights для корреляции данных телеметрии, отправляемых несколькими компонентами. Рассматриваются методы и протоколы распространения контекста. Он также охватывает реализацию тактики корреляции на разных языках и платформах.

## <a name="data-model-for-telemetry-correlation"></a>Модель корреляции для данных телеметрии

Application Insights определяет [модель данных](../../azure-monitor/app/data-model.md) для распределенной корреляции данных телеметрии. Чтобы связать данные телеметрии с логической операцией, каждый элемент телеметрии имеет поле контекста с именем `operation_Id` . Этот идентификатор совместно используется каждым элементом телеметрии в распределенной трассировке. Поэтому даже если вы потеряли данные телеметрии из одного слоя, вы по-прежнему можете связать данные телеметрии, сообщаемые другими компонентами.

Распределенная логическая операция обычно состоит из набора небольших операций, которые являются запросами, обрабатываемыми одним из компонентов. Эти операции определяются [телеметрией запросов](../../azure-monitor/app/data-model-request-telemetry.md). Каждый элемент телеметрии запроса имеет собственный `id` идентификатор, однозначно определяющий его. Все элементы телеметрии (такие как трассировки и исключения), связанные с запросом, должны устанавливать в `operation_parentId` значение запроса `id` .

Каждая исходящая операция (например, вызов HTTP к другому компоненту) представлена [телеметрией зависимостей](../../azure-monitor/app/data-model-dependency-telemetry.md). Данные телеметрии зависимостей также определяют собственный `id` глобально уникальный. Телеметрия запросов, инициированная этим вызовом зависимостей, использует `id` в качестве `operation_parentId`.

Вы можете создать представление распределенной логической операции, используя `operation_Id`, `operation_parentId` и `request.id` с `dependency.id`. Эти поля также определяют причинно-следственную связь вызовов телеметрии.

В среде микрослужб трассировки компонентов могут отправляться в разные хранилища. Каждый компонент может иметь собственный ключ инструментирования в Application Insights. Для получения данных телеметрии для логической операции Application Insights запрашивает данные из каждого элемента хранилища. Если количество элементов хранения велико, вам потребуется подсказка о том, где будет выглядеть следующее. Модель данных Application Insights определяет два поля (`request.source` и `dependency.target`) для решения этой проблемы. Первое поле определяет компонент, который инициировал запрос зависимости. Второе поле определяет, какой компонент вернул ответ на вызов зависимости.

## <a name="example"></a>Пример

Рассмотрим пример. Приложение под названием «цены на акции» показывает текущую стоимость рынка акций с помощью внешнего API, который называется «склад». В приложении "цены на акции" есть страница "акции", которая открывается веб-браузером клиента с помощью `GET /Home/Stock` . Приложение запрашивает стандартный API-интерфейс с помощью вызова HTTP `GET /api/stock/value` .

Вы можете проанализировать итоговые данные телеметрии, выполнив запрос:

```kusto
(requests | union dependencies | union pageViews)
| where operation_Id == "STYz"
| project timestamp, itemType, name, id, operation_ParentId, operation_Id
```

Обратите внимание, что в результатах все элементы телеметрии используют корневой `operation_Id`. При вызове Ajax из страницы новый уникальный идентификатор ( `qJSXU` ) назначается телеметрии зависимости, а идентификатор pageView используется как `operation_ParentId` . Запрос сервера затем использует идентификатор Ajax как `operation_ParentId`.

| itemType   | name                      | ID           | operation_ParentId | operation_Id |
|------------|---------------------------|--------------|--------------------|--------------|
| pageView   | Stock page                | STYz         |                    | STYz         |
| dependency | GET /Home/Stock           | qJSXU        | STYz               | STYz         |
| запрос    | GET Home/Stock            | KqKwlrSt9PA= | qJSXU              | STYz         |
| dependency | GET /api/stock/value      | bBrf2L7mm2g= | KqKwlrSt9PA=       | STYz         |

При вызове `GET /api/stock/value` внешней службы необходимо знать удостоверение этого сервера, чтобы можно было `dependency.target` правильно задать поле. Если внешняя служба не поддерживает мониторинг, то в качестве имени узла службы задается `target` (например, `stock-prices-api.com`). Но если служба идентифицирует себя, возвращая предопределенный заголовок HTTP, `target` содержит удостоверение службы, которое позволяет Application Insights создавать распределенную трассировку, запрашивая данные телеметрии из этой службы.

## <a name="correlation-headers-using-w3c-tracecontext"></a>Заголовки корреляции с использованием W3C Трацеконтекст

Application Insights переходит в [консорциум W3C Trace-context](https://w3c.github.io/trace-context/), который определяет:

- `traceparent`: Содержит глобальный уникальный идентификатор операции и уникальный идентификатор вызова.
- `tracestate`: Содержит контекст трассировки, зависящий от системы.

Последняя версия пакета SDK для Application Insights поддерживает протокол Trace-Context, но может потребоваться согласие на него. (Будет поддерживаться обратная совместимость с предыдущим протоколом корреляции, поддерживаемым пакетом SDK для Application Insights.)

[Протокол HTTP корреляции, также называемый Request-ID](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/HttpCorrelationProtocol.md), является устаревшим. Этот протокол определяет два заголовка:

- `Request-Id`: Содержит глобальный уникальный идентификатор вызова.
- `Correlation-Context`: Содержит коллекцию пар "имя-значение" для свойств распределенной трассировки.

Application Insights также определяет [расширение](https://github.com/lmolkova/correlation/blob/master/http_protocol_proposal_v2.md) для протокола HTTP корреляции. Она использует пары "имя — значение" `Request-Context` для распространения коллекции свойств, используемых непосредственным вызывающим или вызываемым. Пакет SDK для Application Insights использует этот заголовок для задания `dependency.target` `request.source` полей и.

Модель данных " [Трассировка W3C — контекст](https://w3c.github.io/trace-context/) и Application Insights" соответствует следующим образом:

| Application Insights                   | W3C Трацеконтекст                                      |
|------------------------------------    |-------------------------------------------------|
| `Id` из `Request` и `Dependency`     | [родительский идентификатор](https://w3c.github.io/trace-context/#parent-id)                                     |
| `Operation_Id`                         | [Идентификатор трассировки](https://w3c.github.io/trace-context/#trace-id)                                           |
| `Operation_ParentId`                   | [родительский идентификатор](https://w3c.github.io/trace-context/#parent-id) родительского диапазона этого диапазона. Если это корневой диапазон, это поле должно быть пустым.     |

Дополнительные сведения см. в разделе [Application Insightsная модель данных телеметрии](../../azure-monitor/app/data-model.md).

### <a name="enable-w3c-distributed-tracing-support-for-net-apps"></a>Включить поддержку распределенной трассировки W3C для приложений .NET

По умолчанию Распределенная трассировка на основе W3C Трацеконтекст включена во всех последних пакетах SDK платформа .NET Framework/. NET Core, а также обратной совместимости с устаревшим протоколом Request-Id.

### <a name="enable-w3c-distributed-tracing-support-for-java-apps"></a>Включение поддержки распределенной трассировки консорциума W3C для приложений Java

#### <a name="java-30-agent"></a>Агент Java 3,0

  Агент Java 3,0 поддерживает стандарт W3C и не требует дополнительной настройки. 

#### <a name="java-sdk"></a>Пакет SDK для Java
- **Входящая конфигурация**

  - Для приложений Java EE добавьте следующее в `<TelemetryModules>` тег в ApplicationInsights.xml:

    ```xml
    <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebRequestTrackingTelemetryModule>
       <Param name = "W3CEnabled" value ="true"/>
       <Param name ="enableW3CBackCompat" value = "true" />
    </Add>
    ```

  - Для приложений с пружинной загрузкой добавьте следующие свойства:

    - `azure.application-insights.web.enable-W3C=true`
    - `azure.application-insights.web.enable-W3C-backcompat-mode=true`

- **Исходящая конфигурация**

  Добавьте к файлу AI-Agent.xml следующее:

  ```xml
  <Instrumentation>
    <BuiltIn enabled="true">
      <HTTP enabled="true" W3C="true" enableW3CBackCompat="true"/>
    </BuiltIn>
  </Instrumentation>
  ```

  > [!NOTE]
  > По умолчанию включен режим обратной совместимости, и параметр `enableW3CBackCompat` является необязательным. Используйте его, только если вы хотите отключить режим обратной совместимости.
  >
  > В идеале его можно отключить, когда все ваши службы обновлены до новой версии пакетов SDK, поддерживающих протокол консорциума W3C. Мы настоятельно рекомендуем перейти к этим новым пакетам SDK как можно скорее.

> [!IMPORTANT]
> Убедитесь, что входящие и исходящие конфигурации совпадают.

### <a name="enable-w3c-distributed-tracing-support-for-web-apps"></a>Включить поддержку распределенной трассировки W3C для веб-приложений

Эта функция находится в `Microsoft.ApplicationInsights.JavaScript` . Она отключена по умолчанию. Чтобы включить его, используйте `distributedTracingMode` конфигурацию. AI_AND_W3C предоставляется для обеспечения обратной совместимости со всеми устаревшими службами, оснащенными Application Insights.

- **[Установка на основе NPM](./javascript.md#npm-based-setup)**

Добавьте следующую конфигурацию:
  ```JavaScript
    distributedTracingMode: DistributedTracingModes.W3C
  ```

- **[Настройка на основе фрагментов кода](./javascript.md#snippet-based-setup)**

Добавьте следующую конфигурацию:
  ```
      distributedTracingMode: 2 // DistributedTracingModes.W3C
  ```
> [!IMPORTANT] 
> Чтобы просмотреть все конфигурации, необходимые для включения корреляции, см. [документацию по корреляции JavaScript](./javascript.md#enable-correlation).

## <a name="telemetry-correlation-in-opencensus-python"></a>Корреляция телеметрии в Опенценсус Python

Опенценсус Python поддерживает [контекст трассировки W3C](https://w3c.github.io/trace-context/) , не требуя дополнительной настройки.

В качестве образца [можно найти модель](https://github.com/census-instrumentation/opencensus-specs/tree/master/trace)данных опенценсус.

### <a name="incoming-request-correlation"></a>Корреляция входящих запросов

Опенценсус Python сопоставляет заголовки W3C Trace-Context от входящих запросов к диапазонам, которые создаются из самих запросов. Опенценсус сделает это автоматически с интеграцией для этих популярных платформ веб-приложений: Flask, Django и пирамидальная. Необходимо просто заполнить заголовки W3C Trace-Context [правильным форматом](https://www.w3.org/TR/trace-context/#trace-context-http-headers-format) и отправить их вместе с запросом. Вот пример приложения Flask, которое демонстрирует это:

```python
from flask import Flask
from opencensus.ext.azure.trace_exporter import AzureExporter
from opencensus.ext.flask.flask_middleware import FlaskMiddleware
from opencensus.trace.samplers import ProbabilitySampler

app = Flask(__name__)
middleware = FlaskMiddleware(
    app,
    exporter=AzureExporter(),
    sampler=ProbabilitySampler(rate=1.0),
)

@app.route('/')
def hello():
    return 'Hello World!'

if __name__ == '__main__':
    app.run(host='localhost', port=8080, threaded=True)
```

Этот код запускает пример приложения Flask на локальном компьютере, прослушиваемый портом `8080` . Для корреляции контекста трассировки вы отправляете запрос в конечную точку. В этом примере можно использовать `curl` команду:
```
curl --header "traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01" localhost:8080
```
Просмотрев [Формат заголовка Trace-контекст](https://www.w3.org/TR/trace-context/#trace-context-http-headers-format), вы можете получить следующие сведения:

`version`: `00`

`trace-id`: `4bf92f3577b34da6a3ce929d0e0e4736`

`parent-id/span-id`: `00f067aa0ba902b7`

`trace-flags`: `01`

Если взглянуть на запись запроса, отправленную на Azure Monitor, можно увидеть поля, заполненные данными заголовка трассировки. Эти данные можно найти в разделе журналы (аналитика) в ресурсе Azure Monitor Application Insights.

![Запросить данные телеметрии в журналах (аналитика)](./media/opencensus-python/0011-correlation.png)

`id`Поле имеет формат `<trace-id>.<span-id>` , где объект `trace-id` берется из заголовка трассировки, переданного в запросе, и `span-id` является сгенерированным 8-байтовым массивом для этого диапазона.

`operation_ParentId`Поле имеет формат `<trace-id>.<parent-id>` , где `trace-id` и, и, `parent-id` взятые из заголовка трассировки, переданного в запросе.

### <a name="log-correlation"></a>Корреляция журнала

Опенценсус Python позволяет сопоставлять журналы, добавляя идентификатор трассировки, идентификатор диапазона и флаг выборки в записи журнала. Чтобы добавить эти атрибуты, установите [интеграцию с ведением журнала](https://pypi.org/project/opencensus-ext-logging/)опенценсус. Следующие атрибуты будут добавлены в `LogRecord` объекты Python: `traceId` , `spanId` и `traceSampled` . Обратите внимание, что это вступает в силу только для средств ведения журнала, созданных после интеграции.

Вот пример приложения, которое демонстрирует это:

```python
import logging

from opencensus.trace import config_integration
from opencensus.trace.samplers import AlwaysOnSampler
from opencensus.trace.tracer import Tracer

config_integration.trace_integrations(['logging'])
logging.basicConfig(format='%(asctime)s traceId=%(traceId)s spanId=%(spanId)s %(message)s')
tracer = Tracer(sampler=AlwaysOnSampler())

logger = logging.getLogger(__name__)
logger.warning('Before the span')
with tracer.span(name='hello'):
    logger.warning('In the span')
logger.warning('After the span')
```
При выполнении этого кода на консоли выводятся следующие печатные строки:
```
2019-10-17 11:25:59,382 traceId=c54cb1d4bbbec5864bf0917c64aeacdc spanId=0000000000000000 Before the span
2019-10-17 11:25:59,384 traceId=c54cb1d4bbbec5864bf0917c64aeacdc spanId=70da28f5a4831014 In the span
2019-10-17 11:25:59,385 traceId=c54cb1d4bbbec5864bf0917c64aeacdc spanId=0000000000000000 After the span
```
Обратите внимание, что имеется `spanId` сообщение журнала, находящиеся в пределах диапазона. Это то же самое `spanId` , что принадлежит диапазону с именем `hello` .

Данные журнала можно экспортировать с помощью `AzureLogHandler` . Дополнительные сведения см. в [этой статье](./opencensus-python.md#logs).

Также можно передать сведения о трассировке из одного компонента в другой для правильной корреляции. Например, рассмотрим сценарий, в котором есть два компонента `module1` и `module2` . Module1 вызывает функции в Module2, а также для получения журналов из обеих `module1` и `module2` в одной трассировке мы можем использовать следующий подход:

```python
# module1.py
import logging

from opencensus.trace import config_integration
from opencensus.trace.samplers import AlwaysOnSampler
from opencensus.trace.tracer import Tracer
from module2 import function_1

config_integration.trace_integrations(['logging'])
logging.basicConfig(format='%(asctime)s traceId=%(traceId)s spanId=%(spanId)s %(message)s')
tracer = Tracer(sampler=AlwaysOnSampler())

logger = logging.getLogger(__name__)
logger.warning('Before the span')
with tracer.span(name='hello'):
   logger.warning('In the span')
   function_1(tracer)
logger.warning('After the span')


# module2.py

import logging

from opencensus.trace import config_integration
from opencensus.trace.samplers import AlwaysOnSampler
from opencensus.trace.tracer import Tracer

config_integration.trace_integrations(['logging'])
logging.basicConfig(format='%(asctime)s traceId=%(traceId)s spanId=%(spanId)s %(message)s')
tracer = Tracer(sampler=AlwaysOnSampler())

def function_1(parent_tracer=None):
    if parent_tracer is not None:
        tracer = Tracer(
                    span_context=parent_tracer.span_context,
                    sampler=AlwaysOnSampler(),
                )
    else:
        tracer = Tracer(sampler=AlwaysOnSampler())

    with tracer.span("function_1"):
        logger.info("In function_1")
```

## <a name="telemetry-correlation-in-net"></a>Корреляция данных телеметрии в .NET

Среда выполнения .NET поддерживает распространение с помощью [действия](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/ActivityUserGuide.md) и [DiagnosticSource](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md)

Пакет SDK для Application Insights .NET использует `DiagnosticSource` и `Activity` для сбора данных телеметрии и их сопоставления.

<a name="java-correlation"></a>
## <a name="telemetry-correlation-in-java"></a>Корреляция телеметрии в Java

[Агент Java](./java-in-process-agent.md) , а также [пакет SDK для Java](../../azure-monitor/app/java-get-started.md) версии 2.0.0 или более поздней поддерживают автоматическую корреляцию телеметрии. Он автоматически заполняется `operation_id` всеми данными телеметрии (например, трассировками, исключениями и пользовательскими событиями), выданными в области запроса. Он также распространяет заголовки корреляции (описанные выше) для вызовов между службами по протоколу HTTP, если настроен [Агент пакета SDK для Java](../../azure-monitor/app/java-agent.md) .

> [!NOTE]
> Application Insights агент Java Auto собирает запросы и зависимости для JMS, Kafka, NETTY/Вебфлукс и т. д. Для функции корреляции поддерживаются только вызовы пакета SDK для Java, выполненные через Apache HttpClient. Автоматическое распространение контекста по технологиям обмена сообщениями (например, Kafka, RabbitMQ и служебная шина Azure) не поддерживается в пакете SDK. 

> [!NOTE]
> Для сбора пользовательских данных телеметрии необходимо инструментировать приложение с помощью пакета SDK для Java 2,6. 

### <a name="role-names"></a>Имена ролей

Может потребоваться настроить способ отображения имен компонентов в [схеме приложения](../../azure-monitor/app/app-map.md). Для этого можно вручную настроить, `cloud_RoleName` выполнив одно из следующих действий.

- Для Application Insights агента Java 3,0 задайте имя роли облака следующим образом:

    ```json
    {
      "role": {
        "name": "my cloud role name"
      }
    }
    ```
    Вы также можете задать имя роли облака с помощью переменной среды `APPLICATIONINSIGHTS_ROLE_NAME` .

- С помощью Application Insights пакета SDK для Java 2.5.0 и более поздних версий можно указать, `cloud_RoleName` добавив `<RoleName>` в файл ApplicationInsights.xml:

  ```XML
  <?xml version="1.0" encoding="utf-8"?>
  <ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings" schemaVersion="2014-05-30">
     <InstrumentationKey>** Your instrumentation key **</InstrumentationKey>
     <RoleName>** Your role name **</RoleName>
     ...
  </ApplicationInsights>
  ```

- Если вы используете пружинную загрузку с Application Insights пружинной загрузкой, необходимо просто задать пользовательское имя для приложения в файле Application. Properties:

  `spring.application.name=<name-of-app>`

  Начальная загрузочная загрузка автоматически присваивает `cloudRoleName` значение, введенное для `spring.application.name` Свойства.

## <a name="next-steps"></a>Дальнейшие действия

- Запись [пользовательской телеметрии](../../azure-monitor/app/api-custom-events-metrics.md).
- Дополнительные сценарии корреляции в ASP.NET Core и ASP.NET см. в разделе [Track Custom Operations](custom-operations-tracking.md).
- Узнайте об [установке свойства cloud_RoleName](./app-map.md#set-or-override-cloud-role-name) для других пакетов SDK.
- Подключите все компоненты своей микрослужбы с помощью Application Insights. Ознакомьтесь со сведениями о [поддерживаемых платформах](./platforms.md).
- В [этой статье](./data-model.md) представлена модель данных для Application Insights.
- Вы можете узнать, как [расширять и фильтровать данные телеметрии](./api-filtering-sampling.md).
- Просмотрите [справочник по конфигурации в Application Insights](configuration-with-applicationinsights-config.md).