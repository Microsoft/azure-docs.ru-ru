---
title: Azure Monitor Application Insights Java
description: Наблюдение за производительностью приложений Java, выполняющихся в любой среде, без необходимости изменения кода. Распределенная трассировка и схема приложения.
ms.topic: conceptual
ms.date: 03/29/2020
author: MS-jgol
ms.custom: devx-track-java
ms.author: jgol
ms.openlocfilehash: b695df29b7a4704ee9e4e25e402fa0de8f2b7685
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103008218"
---
# <a name="java-codeless-application-monitoring-azure-monitor-application-insights"></a>Application Insights Azure Monitor отслеживания приложений Java с некодированным кодом

Использовать мониторинг приложений Java без написания кода очень просто: не нужно изменять код, а агент Java можно включить, лишь немного изменив конфигурацию.

 Агент Java работает в любой среде и позволяет отслеживать все приложения Java. Другими словами, независимо от того, выполняются ли приложения Java на виртуальных машинах (локально), в AKS в Windows, Linux — Вы переименовываете его, а агент Java 3,0 будет отслеживать ваше приложение.

Добавление пакета SDK для Application Insights Java в приложение больше не требуется, так как агент 3,0 выполняет автоматический сбор запросов, зависимостей и журналов отдельно.

Вы по-прежнему можете отправить пользовательскую телеметрию из приложения. Агент 3,0 будет отслеживать и сопоставлять его вместе со всеми автособираемыми данными телеметрии.

Агент 3,0 поддерживает Java 8 и более поздних версий.

## <a name="quickstart"></a>Краткое руководство

**1. Скачайте агент**

> [!WARNING]
> **При обновлении с предварительной версии 3,0**
>
> Тщательно изучите все [Параметры конфигурации](./java-standalone-config.md) , так как структура JSON была полностью изменена, а не только само имя файла, что и все строчные буквы.

Скачать [аппликатионинсигхтс-ажент-3.0.2. jar](https://github.com/microsoft/ApplicationInsights-Java/releases/download/3.0.2/applicationinsights-agent-3.0.2.jar)

**2. Указание ВИРТУАЛЬНОЙ машины Java агенту**

Добавить `-javaagent:path/to/applicationinsights-agent-3.0.2.jar` в виртуальной машины Java argss вашего приложения

Стандартные аргументы ВИРТУАЛЬНОЙ машины Java включают `-Xmx512m` и `-XX:+UseG1GC` . Итак, если вы узнаете, куда добавить эти данные, вы уже знакомы с их добавлением.

Дополнительные сведения о настройке аргументов ВИРТУАЛЬНОЙ машины Java приложения см. в статье [Советы по обновлению аргументов виртуальной машины Java](./java-standalone-arguments.md).

**3. Наведите агент на ресурс Application Insights**

Если у вас еще нет ресурса Application Insights, можно создать новый, выполнив действия, описанные в [руководстве по созданию ресурсов](./create-new-resource.md).

Наведите агент на ресурс Application Insights, задав переменную среды:

```
APPLICATIONINSIGHTS_CONNECTION_STRING=InstrumentationKey=...
```

Или создайте файл конфигурации с именем `applicationinsights.json` и поместите его в тот же каталог, что и `applicationinsights-agent-3.0.2.jar` , со следующим содержимым:

```json
{
  "connectionString": "InstrumentationKey=..."
}
```

Строку подключения можно найти в ресурсе Application Insights:

:::image type="content" source="media/java-ipa/connection-string.png" alt-text="Строка подключения Application Insights":::

**4. Вот и все!**

Теперь запустите приложение и перейдите к ресурсу Application Insights в портал Azure, чтобы просмотреть данные мониторинга.

> [!NOTE]
> Для отображения данных мониторинга на портале может потребоваться несколько минут.


## <a name="configuration-options"></a>Варианты настройки

В `applicationinsights.json` файле можно дополнительно настроить:

* Имя облачной роли
* Экземпляр облачной роли
* Выборка
* Метрики JMX
* Пользовательские измерения
* Обработчики данных телеметрии (Предварительная версия)
* Автоматическое собираемое ведение журнала
* Автособираемые метрики Микрометер (включая метрики загрузчика пружины)
* Пульс
* Прокси-сервер HTTP
* Самостоятельная диагностика

Полные сведения см. в разделе [Параметры конфигурации](./java-standalone-config.md) .

## <a name="auto-collected-requests-dependencies-logs-and-metrics"></a>Автоматические собранные запросы, зависимости, журналы и метрики

### <a name="requests"></a>Requests

* Потребители JMS
* Потребители Kafka
* NETTY/Вебфлукс
* Сервлеты
* Планирование весны

### <a name="dependencies-with-distributed-trace-propagation"></a>Зависимости с распространением распределенной трассировки

* Apache HttpClient и Хттпасинкклиент
* gRPC
* Java. NET. Хттпурлконнектион
* JMS
* Kafka
* Клиент NETTY
* OkHttp;

### <a name="other-dependencies"></a>Другие зависимости

* Cassandra
* JDBC
* MongoDB (Async и Sync)
* Redis (салат и Jedis)

### <a name="logs"></a>Журналы

* java.util.logging
* Log4j (включая свойства MDC)
* SLF4J/Logback (включая свойства MDC)

### <a name="metrics"></a>Метрики

* Микрометер (включая метрики огнетушителя пружинной загрузки)
* Метрики JMX

## <a name="send-custom-telemetry-from-your-application"></a>Отправка пользовательских данных телеметрии из приложения

Наша цель в версии 3.0 + позволяет отправить пользовательскую телеметрию с помощью стандартных API.

Мы поддерживаем Микрометер, популярные платформы ведения журналов и Application Insights пакет SDK для Java 2. x до настоящего момента.
Application Insights Java 3,0 автоматически захватывает данные телеметрии, отправляемые через эти API, и сопоставляет их с автоматически собираемой телеметрией.

### <a name="supported-custom-telemetry"></a>Поддерживаемые пользовательские данные телеметрии

Приведенная ниже таблица представляет сейчас поддерживаемые пользовательские типы телеметрии, которые можно включить в дополнение к агенту Java 3,0. Чтобы суммировать, пользовательские метрики поддерживаются через микрометер, пользовательские исключения и трассировки можно включить с помощью платформ ведения журналов, а любой тип пользовательской телеметрии поддерживается с помощью [пакета SDK для Java 2. x Application Insights](#send-custom-telemetry-using-the-2x-sdk).

|                     | Micrometer | Log4j, logback, Июл | пакет SDK для 2. x |
|---------------------|------------|---------------------|---------|
| **Пользовательские события**   |            |                     |  Да    |
| **Пользовательские метрики**  |  Да       |                     |  Да    |
| **Зависимости**    |            |                     |  Да    |
| **Исключения**      |            |  Да                |  Да    |
| **Просмотры страницы**      |            |                     |  Да    |
| **Запросы**        |            |                     |  Да    |
| **Трассировки**          |            |  Да                |  Да    |

В настоящее время мы не планируем выпустить пакет SDK с Application Insights 3,0.

Application Insights Java 3,0 уже прослушивает данные телеметрии, отправляемые в пакет SDK для Application Insights Java 2. x. Эта функция является важной частью истории обновления для существующих пользователей версии 2. x, и она заполняет важный зазор в нашей пользовательской поддержке телеметрии, пока интерфейс API Опентелеметри не будет общедоступен.

### <a name="send-custom-metrics-using-micrometer"></a>Отправка пользовательских метрик с помощью Микрометер

Добавьте Микрометер в приложение:

```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-core</artifactId>
  <version>1.6.1</version>
</dependency>
```

Используйте [глобальный реестр](https://micrometer.io/docs/concepts#_global_registry) микрометер для создания счетчика:

```java
static final Counter counter = Metrics.counter("test_counter");
```

и используйте его для записи метрик.

```java
counter.increment();
```

### <a name="send-custom-traces-and-exceptions-using-your-favorite-logging-framework"></a>Отправка пользовательских трассировок и исключений с помощью избранной инфраструктуры ведения журналов

Log4j, Logback и Java. util. Logging устанавливаются в автоматическом инструментировании, а ведение журнала с помощью этих платформ ведения журналов выполняется в автоматическом виде в виде трассировки и телеметрии исключений.

По умолчанию ведение журнала выполняется только в том случае, если ведение журнала выполняется на уровне INFO или выше.
Сведения о том, как изменить этот уровень, см. в разделе [Параметры конфигурации](./java-standalone-config.md#auto-collected-logging) .

Если вы хотите присоединить пользовательские измерения к журналам, можно использовать [Log4j 1,2 MDC](https://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/MDC.html), [log4j 2 MDC](https://logging.apache.org/log4j/2.x/manual/thread-context.html)или [Logback MDC](http://logback.qos.ch/manual/mdc.html), а Application Insights Java 3,0 автоматически захватит эти свойства MDC в виде пользовательских измерений для трассировки и телеметрии исключений.

### <a name="send-custom-telemetry-using-the-2x-sdk"></a>Отправка пользовательской телеметрии с помощью пакета SDK для 2. x

Добавьте `applicationinsights-core-2.6.2.jar` в приложение (все версии 2. x поддерживаются Application Insights Java 3,0, но следует использовать последнюю версию, если у вас есть вариант):

```xml
<dependency>
  <groupId>com.microsoft.azure</groupId>
  <artifactId>applicationinsights-core</artifactId>
  <version>2.6.2</version>
</dependency>
```

Создайте TelemetryClient:

  ```java
static final TelemetryClient telemetryClient = new TelemetryClient();
```

и используют его для отправки пользовательских данных телеметрии:

##### <a name="events"></a>События

```java
telemetryClient.trackEvent("WinGame");
```

##### <a name="metrics"></a>Метрики

```java
telemetryClient.trackMetric("queueLength", 42.0);
```

##### <a name="dependencies"></a>Зависимости

```java
boolean success = false;
long startTime = System.currentTimeMillis();
try {
    success = dependency.call();
} finally {
    long endTime = System.currentTimeMillis();
    RemoteDependencyTelemetry telemetry = new RemoteDependencyTelemetry();
    telemetry.setSuccess(success);
    telemetry.setTimestamp(new Date(startTime));
    telemetry.setDuration(new Duration(endTime - startTime));
    telemetryClient.trackDependency(telemetry);
}
```

##### <a name="logs"></a>Журналы

```java
telemetryClient.trackTrace(message, SeverityLevel.Warning, properties);
```

##### <a name="exceptions"></a>Исключения

```java
try {
    ...
} catch (Exception e) {
    telemetryClient.trackException(e);
}
```

### <a name="add-request-custom-dimensions-using-the-2x-sdk"></a>Добавление запроса пользовательских измерений с помощью пакета SDK 2. x

> [!NOTE]
> Эта функция доступна только в 3.0.2 и более поздних версиях

Добавьте `applicationinsights-web-2.6.2.jar` в приложение (все версии 2. x поддерживаются Application Insights Java 3,0, но следует использовать последнюю версию, если у вас есть вариант):

```xml
<dependency>
  <groupId>com.microsoft.azure</groupId>
  <artifactId>applicationinsights-web</artifactId>
  <version>2.6.2</version>
</dependency>
```

и добавьте пользовательские измерения в код:

```java
import com.microsoft.applicationinsights.web.internal.ThreadContext;

RequestTelemetry requestTelemetry = ThreadContext.getRequestTelemetryContext().getHttpRequestTelemetry();
requestTelemetry.getProperties().put("mydimension", "myvalue");
```

### <a name="set-the-request-telemetry-user_id-using-the-2x-sdk"></a>Настройка user_Id телеметрии запросов с помощью пакета SDK 2. x

> [!NOTE]
> Эта функция доступна только в 3.0.2 и более поздних версиях

Добавьте `applicationinsights-web-2.6.2.jar` в приложение (все версии 2. x поддерживаются Application Insights Java 3,0, но следует использовать последнюю версию, если у вас есть вариант):

```xml
<dependency>
  <groupId>com.microsoft.azure</groupId>
  <artifactId>applicationinsights-web</artifactId>
  <version>2.6.2</version>
</dependency>
```

и задайте `user_Id` в коде:

```java
import com.microsoft.applicationinsights.web.internal.ThreadContext;

RequestTelemetry requestTelemetry = ThreadContext.getRequestTelemetryContext().getHttpRequestTelemetry();
requestTelemetry.getContext().getUser().setId("myuser");
```

### <a name="override-the-request-telemetry-name-using-the-2x-sdk"></a>Переопределение имени телеметрии запроса с помощью пакета SDK 2. x

> [!NOTE]
> Эта функция доступна только в 3.0.2 и более поздних версиях

Добавьте `applicationinsights-web-2.6.2.jar` в приложение (все версии 2. x поддерживаются Application Insights Java 3,0, но следует использовать последнюю версию, если у вас есть вариант):

```xml
<dependency>
  <groupId>com.microsoft.azure</groupId>
  <artifactId>applicationinsights-web</artifactId>
  <version>2.6.2</version>
</dependency>
```

и задайте имя в коде:

```java
import com.microsoft.applicationinsights.web.internal.ThreadContext;

RequestTelemetry requestTelemetry = ThreadContext.getRequestTelemetryContext().getHttpRequestTelemetry();
requestTelemetry.setName("myname");
```

### <a name="get-the-request-telemetry-id-and-the-operation-id-using-the-2x-sdk"></a>Получение идентификатора телеметрии запроса и идентификатора операции с помощью пакета SDK 2. x

> [!NOTE]
> Эта функция доступна только в 3.0.3-BETA и более поздних версиях

Добавьте `applicationinsights-web-2.6.2.jar` в приложение (все версии 2. x поддерживаются Application Insights Java 3,0, но следует использовать последнюю версию, если у вас есть вариант):

```xml
<dependency>
  <groupId>com.microsoft.azure</groupId>
  <artifactId>applicationinsights-web</artifactId>
  <version>2.6.2</version>
</dependency>
```

и получите идентификатор телеметрии запроса и идентификатор операции в коде:

```java
import com.microsoft.applicationinsights.web.internal.ThreadContext;

RequestTelemetry requestTelemetry = ThreadContext.getRequestTelemetryContext().getHttpRequestTelemetry();
String requestId = requestTelemetry.getId();
String operationId = requestTelemetry.getContext().getOperation().getId();
```
