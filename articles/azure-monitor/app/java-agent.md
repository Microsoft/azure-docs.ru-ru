---
title: Мониторинг производительности веб-приложений Java — Azure Application Insights
description: Расширенный мониторинг производительности и использования веб-сайта Java с помощью Application Insights.
ms.topic: conceptual
ms.date: 01/10/2019
author: MS-jgol
ms.custom: devx-track-java
ms.author: jgol
ms.openlocfilehash: c753e4e254890f9198da9bc913b29bdaae335b78
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100573830"
---
# <a name="monitor-dependencies-caught-exceptions-and-method-execution-times-in-java-web-apps"></a>Мониторинг зависимостей, перехваченных исключений и времени выполнения методов в веб-приложениях Java

> [!IMPORTANT]
> Подход, описанный в этом документе, больше не рекомендуется.
>
> Для мониторинга приложений Java рекомендуется использовать автоматическое инструментирование без изменения кода. Следуйте указаниям для [Application Insights агента Java 3,0](./java-in-process-agent.md).

Если вы выполнили [Инструментирование веб-приложения Java с помощью пакета SDK для Application Insights][java], вы можете использовать агент Java для получения более подробных сведений, не внося изменения в код:

* **Зависимости** — данные о вызовах других компонентов в вашем приложении, включая:
  * **Исходящие вызовы HTTP** , выполненные через Apache HttpClient, OkHttp и, `java.net.HttpURLConnection` фиксируются.
  * **Вызовы Redis** , выполненные через клиент Jedis, фиксируются.
  * **Запросы JDBC** — для MySQL и PostgreSQL, если вызов занимает больше 10 секунд, агент сообщает о плане запроса.

* **Ведение журнала приложения:** Запись и сопоставление журналов приложений с запросами HTTP и другими данными телеметрии
  * **Log4j 1,2**
  * **Log4j2**
  * **Logback**

* **Улучшенное именование операций:** (используется для агрегирования запросов на портале)
  * **Пружинный** на основе `@RequestMapping` .
  * **JAX-RS** -на основе `@Path` . 

Чтобы использовать агент для Java, его необходимо установить на сервере. Веб-приложения необходимо инструментировать [пакетом SDK для Java Application Insights][java]. 

## <a name="install-the-application-insights-agent-for-java"></a>Установка агента Application Insights для Java
1. На компьютере, на котором работает сервер Java, [Скачайте агент 2. x](https://github.com/microsoft/ApplicationInsights-Java/releases/tag/2.6.2). Убедитесь, что используемая версия агента Java для версии 2. x соответствует версии пакета SDK для Java, установленного на языке 2. x Application Insights.
2. Измените скрипт запуска сервера приложений и добавьте следующий аргумент ВИРТУАЛЬНОЙ машины Java:
   
    `-javaagent:<full path to the agent JAR file>`
   
    Например, в Tomcat на компьютере Linux:
   
    `export JAVA_OPTS="$JAVA_OPTS -javaagent:<full path to agent JAR file>"`
3. Перезапустите сервер приложений.

## <a name="configure-the-agent"></a>Настройка агента
Создайте файл с именем `AI-Agent.xml` и поместите его в ту же папку, где находится JAR-файл агента.

Настройка содержимого XML-файла. Измените приведенный ниже пример, включив необходимые функции или убрав ненужные.

```XML
<?xml version="1.0" encoding="utf-8"?>
<ApplicationInsightsAgent>
   <Instrumentation>
      <BuiltIn enabled="true">

         <!-- capture logging via Log4j 1.2, Log4j2, and Logback, default is true -->
         <Logging enabled="true" />

         <!-- capture outgoing HTTP calls performed through Apache HttpClient, OkHttp,
              and java.net.HttpURLConnection, default is true -->
         <HTTP enabled="true" />

         <!-- capture JDBC queries, default is true -->
         <JDBC enabled="true" />

         <!-- capture Redis calls, default is true -->
         <Jedis enabled="true" />

         <!-- capture query plans for JDBC queries that exceed this value (MySQL, PostgreSQL),
              default is 10000 milliseconds -->
         <MaxStatementQueryLimitInMS>1000</MaxStatementQueryLimitInMS>

      </BuiltIn>
   </Instrumentation>
</ApplicationInsightsAgent>
```

## <a name="additional-config-spring-boot"></a>Дополнительная конфигурация (пружинная загрузка)

`java -javaagent:/path/to/agent.jar -jar path/to/TestApp.jar`

Для служб приложений Azure выполните следующие действия.

* Выберите элементы "Параметры > Параметры приложения".
* В разделе "Параметры приложения" добавьте новую пару "ключ — значение":

Ключ: `JAVA_OPTS` значение: `-javaagent:D:/home/site/wwwroot/applicationinsights-agent-2.6.2.jar`

Агент должен быть упакован в проект в виде ресурса таким образом, чтобы он закончится в папке D:/Home, site/wwwroot/. Чтобы убедиться, что агент находится в нужном каталоге службы приложений, перейдите в **меню средства разработки**  >  **Дополнительные инструменты**  >  **консоль отладки** и проверьте содержимое каталога сайта.    

* Сохраните параметры и перезапустите приложение. (Эти действия применимы только к службам приложений, работающим в Windows.)

> [!NOTE]
> Файлы агента AI-Agent.xml и JAR-файл должны быть в одной папке. Они часто расположены вместе в папке `/resources` проекта.  

#### <a name="enable-w3c-distributed-tracing"></a>Активация распределенной трассировки W3C

Добавьте к файлу AI-Agent.xml следующее:

```xml
<Instrumentation>
   <BuiltIn enabled="true">
      <HTTP enabled="true" W3C="true" enableW3CBackCompat="true"/>
   </BuiltIn>
</Instrumentation>
```

> [!NOTE]
> Режим обратной совместимости включен по умолчанию, и параметр enableW3CBackCompat является необязательным и должен использоваться только в том случае, если вы хотите отключить эту функцию. 

В идеале он может использоваться, когда все ваши службы обновлены к новой версии пакетов SDK, поддерживающих протокол консорциума W3C. Настоятельно рекомендуется перейти к новой версии пакетов SDK с поддержкой W3C как можно скорее.

Убедитесь, что **оба [входящий](correlation.md#enable-w3c-distributed-tracing-support-for-java-apps) и исходящий (агенты) конфигурации** совпадают.

## <a name="view-the-data"></a>Просмотр данных
В ресурсе Application Insights агрегированная Удаленная зависимость и время выполнения метода отображаются [под плиткой "производительность"][metrics].

Для поиска отдельных экземпляров отчетов по зависимостям, исключениям и методам откройте [Поиск][diagnostic].

[Дополнительные сведения о диагностировании проблем зависимостей](./asp-net-dependencies.md#diagnosis).

## <a name="questions-problems"></a>У вас появились вопросы? Проблемы?
* Данные отсутствуют? [Настройка исключений брандмауэра](./ip-addresses.md)
* [Устранение неполадок Java](java-troubleshoot.md)

<!--Link references-->

[api]: ./api-custom-events-metrics.md
[apiexceptions]: ./api-custom-events-metrics.md#track-exception
[availability]: ./monitor-web-app-availability.md
[diagnostic]: ./diagnostic-search.md
[eclipse]: app-insights-java-eclipse.md
[java]: java-get-started.md
[javalogs]: java-trace-logs.md
[metrics]: ../essentials/metrics-charts.md

