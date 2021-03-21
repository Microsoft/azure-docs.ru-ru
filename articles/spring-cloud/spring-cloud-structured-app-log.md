---
title: Журнал структурированных приложений для Azure Веснного облака | Документация Майкрософт
description: В этой статье объясняется, как создать и получить данные журнала приложений в Azure Веснного облака.
author: MikeDodaro
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 02/05/2021
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: 844530c6d1650b5fddd27b10c775c4364a3f5147
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102124471"
---
# <a name="structured-application-log-for-azure-spring-cloud"></a>Журнал структурированных приложений для Azure Веснного облака

В этой статье объясняется, как создать и получить данные журнала приложений в Azure Веснного облака. С правильной настройкой Azure Веснное облако предоставляет полезные запросы и анализ журналов приложений с помощью Log Analytics.

## <a name="log-schema-requirements"></a>Требования к схеме журнала
Чтобы улучшить работу с запросами журналов, необходимо, чтобы журнал приложений был в формате JSON и соответствовал схеме. В Azure Веснного облака эта схема используется для анализа приложения и потока в Log Analytics. 

**Требования к схеме JSON:**

| Ключ JSON      | Тип значения JSON|  Обязательно | Столбец в Log Analytics| Описание |
| --------------| ------------|-----------|-----------------|--------------------------|
| TIMESTAMP     | строка      |     Да   | апптиместамп    | отметка времени в формате UTC  |
| logger        | Строка      |     Нет    | Средство ведения журнала          | logger                   |
| уровень         | Строка      |     Нет    | кустомлевел     | уровень ведения журнала                |
| thread        | Строка      |     Нет    | Thread          | thread                   |
| message       | строка      |     Нет    | Сообщение         | сообщение журнала              |
| stackTrace    | Строка      |     Нет    | StackTrace      | Трассировка стека исключений    |
| ексцептионкласс| Строка      |     Нет    | ексцептионкласс  | имя класса исключений     |
| MDC           | вложенный JSON |     Нет    |                 | сопоставленный контекст диагностики|
| MDC. traceId   | Строка      |     Нет    | TraceId         |Идентификатор трассировки для распределенной трассировки|
| MDC. Спанид    | Строка      |     Нет    | спанид          |Идентификатор диапазона для распределенной трассировки |
|               |             |           |                 |                          |

* Поле "timestamp" является обязательным и должно быть в формате UTC. все остальные поля являются необязательными.
* для целей трассировки используются "traceId" и "Спанид" в поле "MDC".
* Запись каждой записи JSON в журнал в одной строке. 

**Пример записи журнала** 
 ```
{"timestamp":"2021-01-08T09:23:51.280Z","logger":"com.example.demo.HelloController","level":"ERROR","thread":"http-nio-1456-exec-4","mdc":{"traceId":"c84f8a897041f634","spanId":"c84f8a897041f634"},"stackTrace":"java.lang.RuntimeException: get an exception\r\n\tat com.example.demo.HelloController.throwEx(HelloController.java:54)\r\n\","message":"Got an exception","exceptionClass":"RuntimeException"}
```

## <a name="generate-schema-compliant-json-log"></a>Создать журнал JSON, соответствующий схеме  
Для пружинных приложений можно создать ожидаемый формат журнала JSON с помощью общих [платформ ведения журналов](https://docs.spring.io/spring-boot/docs/2.1.13.RELEASE/reference/html/boot-features-logging.html#boot-features-custom-log-configuration), таких как [logback](http://logback.qos.ch/) и [log4j2](https://logging.apache.org/log4j/2.x/). 

### <a name="log-with-logback"></a>Регистрация с помощью logback 
При использовании стартовых загрузок по умолчанию используется logback. Для приложений logback используйте [logstash-Encoder](https://github.com/logstash/logstash-logback-encoder) для создания журнала в формате JSON. Этот метод поддерживается в пружинной загрузке 2.1 и более поздних версиях. 

Процедура:

1. Добавьте зависимость logstash в файл pom.xml. 

    ```json
    <dependency>
        <groupId>net.logstash.logback</groupId>
        <artifactId>logstash-logback-encoder</artifactId>
        <version>6.5</version>
    </dependency>
    ```
1. Обновите файл конфигурации logback.xml, чтобы задать формат JSON.
    ```json
    <configuration>
        <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
            <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
            <providers>
                <timestamp>
                    <fieldName>timestamp</fieldName>
                    <timeZone>UTC</timeZone>
                </timestamp>
                <loggerName>
                    <fieldName>logger</fieldName>
                </loggerName>
                <logLevel>
                    <fieldName>level</fieldName>
                </logLevel>
                <threadName>
                    <fieldName>thread</fieldName>
                </threadName>
                <nestedField>
                    <fieldName>mdc</fieldName>
                    <providers>
                        <mdc/>
                    </providers>
                </nestedField>
                <stackTrace>
                    <fieldName>stackTrace</fieldName>
                </stackTrace>
                <message/>
                <throwableClassName>
                    <fieldName>exceptionClass</fieldName>
                </throwableClassName>
            </providers>
            </encoder>
        </appender>
        <root level="info">
            <appender-ref ref="stdout"/>
        </root>
    </configuration>
    ```

### <a name="log-with-log4j2"></a>Регистрация с помощью log4j2 

Для приложений log4j2 используйте [JSON-Template-Layout](https://logging.apache.org/log4j/2.x/manual/json-template-layout.html) для создания журнала в формате JSON. Этот метод поддерживается в пружинной загрузке 2.1 и более поздних версиях.

Процедура:

1. Исключите `spring-boot-starter-logging` из `spring-boot-starter` , добавьте `spring-boot-starter-log4j2` зависимости `log4j-layout-template-json` в файл pom.xml.

    ```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                    <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-layout-template-json</artifactId>
            <version>2.14.0</version>
        </dependency>
    ```

2. Подготовьте файл шаблона макета JSON jsonTemplate.jsв пути к классу.

    ```json
        {
            "mdc": {
                "$resolver": "mdc"
            },
            "exceptionClass": {
                "$resolver": "exception",
                "field": "className"
            },
            "stackTrace": {
                "$resolver": "exception",
                "field": "stackTrace",
                "stringified": true
            },
            "message": {
                "$resolver": "message",
                "stringified": true
            },
            "thread": {
                "$resolver": "thread",
                "field": "name"
            },
            "timestamp": {
                "$resolver": "timestamp",
                "pattern": {
                    "format": "yyyy-MM-dd'T'HH:mm:ss.SSS'Z'",
                    "timeZone": "UTC"
                }
            },
            "level": {
                "$resolver": "level",
                "field": "name"
            },
            "logger": {
                "$resolver": "logger",
                "field": "name"
            }
        }
    ```

3. Используйте этот шаблон макета JSON в файле конфигурации log4j2.xml. 

    ```json
        <configuration>
            <appenders>
                <console name="Console" target="SYSTEM_OUT">
                <JsonTemplateLayout eventTemplateUri="classpath:jsonTemplate.json"/>
                </console>
            </appenders>
            <loggers>
                <root level="info">
                <appender-ref ref="Console"/>
                </root>
            </loggers>
        </configuration>
    ```

## <a name="analyze-the-logs-in-log-analytics"></a>Анализ журналов в Log Analytics

После правильной настройки приложения журнал консоли приложения будет передаваться в поток на Log Analytics. Структура обеспечивает эффективный запрос в Log Analytics.

### <a name="check-log-structure-in-log-analytics"></a>Проверка структуры журнала в Log Analytics
Выполните перечисленные ниже действия.
1. Перейдите на страницу обзора службы экземпляра службы.
2. Щелкните `Logs` запись в `Monitoring` разделе раздел.
3. Выполните этот запрос.

   ```
   AppPlatformLogsforSpring
   | where TimeGenerated > ago(1h)
   | project AppTimestamp, Logger, CustomLevel, Thread, Message, ExceptionClass, StackTrace, TraceId, SpanId
   ```

4. Журналы приложений возвращают, как показано на следующем рисунке:

![Отображение журнала JSON](media/spring-cloud-structured-app-log/json-log-query.png)

### <a name="show-log-entries-containing-errors"></a>Отображение записей журнала, содержащих ошибки

Чтобы проверить записи журнала с ошибкой, выполните следующий запрос:

```
AppPlatformLogsforSpring
| where TimeGenerated > ago(1h) and CustomLevel == "ERROR" 
| project AppTimestamp, Logger, ExceptionClass, StackTrace, Message, AppName 
| sort by AppTimestamp
```

Используйте этот запрос для поиска ошибок или изменения условий запроса, чтобы найти конкретный класс исключения или код ошибки. 

### <a name="show-log-entries-for-a-specific-traceid"></a>Отображение записей журнала для определенного traceId
Чтобы проверить записи журнала для определенного идентификатора трассировки "trace_id", выполните следующий запрос:

```
AppPlatformLogsforSpring
| where TimeGenerated > ago(1h)
| where TraceId == "trace_id" 
| project AppTimestamp, Logger, TraceId, SpanId, StackTrace, Message, AppName 
| sort by AppTimestamp
```

## <a name="next-steps"></a>Дальнейшие действия
* Дополнительные сведения о запросе журнала см. в статье [Приступая к работе с запросами журналов в Azure Monitor](../azure-monitor/logs/get-started-queries.md)