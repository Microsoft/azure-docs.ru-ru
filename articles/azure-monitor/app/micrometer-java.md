---
title: Как использовать Micrometer с пакетом SDK Azure Application Insights для Java
description: Поэтапное руководство по использованию Micrometer в приложениях Spring Boot и других приложениях для Application Insights.
ms.topic: conceptual
author: MS-jgol
ms.custom: devx-track-java
ms.author: jgol
ms.date: 11/01/2018
ms.openlocfilehash: 8dba7280f6abd6026fabdde500dc76b73129d557
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100589750"
---
# <a name="how-to-use-micrometer-with-azure-application-insights-java-sdk-not-recommended"></a>Как использовать Микрометер с пакетом SDK для Java Application Insights Azure (не рекомендуется)

> [!IMPORTANT]
> Подход, описанный в этом документе, больше не рекомендуется.
> 
> Для мониторинга приложений Java рекомендуется использовать автоматическое инструментирование без изменения кода. Данные телеметрии микрометер собираются с помощью агента Application Insights Java 3,0. Следуйте указаниям для [Application Insights агента java 3,0](./java-in-process-agent.md).

> [!NOTE]
> Application Insights пакет SDK для Java не поддерживает пружинный Вебфлукс-использование [Application Insights агента Java 3,0](./java-in-process-agent.md) . 
>
> Вебфлукс и Микрометер поддерживаются в [Application Insights агенте Java 3,0](./java-on-premises.md) , не требующем инструментирования. 

Мониторинг приложений Micrometer измеряет метрики для кода приложения на основе виртуальной машины Java и позволяет экспортировать данные в предпочитаемые системы мониторинга. В этой статье вы узнаете, как использовать Micrometer с Application Insights для приложений Spring Boot и других приложений.

## <a name="using-spring-boot-15x"></a>Использование Spring Boot 1.5x
Добавьте следующие зависимости в файл pom.xml или build.gradle: 
* [Application Insights пружины-Boot-Starter](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/spring/azure-spring-boot-starter) 2.5.0 или более поздней версии
* Реестр Azure Micrometer 1.1.0 или более поздней версии.
* [Micrometer Spring Legacy](https://micrometer.io/docs/ref/spring/1.5) 1.1.0 или более поздней версии (возвращение кода автонастройки на платформе Spring к более ранней версии).
* [Ресурс ApplicationInsights](./create-new-resource.md)

Шаги

1. Обновите файл pom.xml приложения Spring Boot и включите в него следующие зависимости:

    ```XML
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>applicationinsights-spring-boot-starter</artifactId>
        <version>2.5.0</version>
    </dependency>

    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-spring-legacy</artifactId>
        <version>1.1.0</version>
    </dependency>

    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-registry-azure-monitor</artifactId>
        <version>1.1.0</version>
    </dependency>

    ```
2. Обновите файл application.properties или yml с помощью ключа инструментирования Application Insights, используя следующее свойство:

     `azure.application-insights.instrumentation-key=<your-instrumentation-key-here>`
1. Создайте и запустите приложение.
2. Выполнив указанные выше инструкции, вы сможете приступить к работе с предварительно агрегированными метриками, автоматически собранными в Azure Monitor. Подробнее о том, как настроить начальное приложение Spring Boot Application Insights, см. на странице с [файлом сведений на GitHub](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/spring/azure-spring-boot-starter/README.md).

## <a name="using-spring-2x"></a>Использование Spring 2.x

Добавьте следующие зависимости в файл pom.xml или build.gradle:

* spring-boot-starter для Application Insights версии 2.1.2 или более поздней.
* Azure-пружины-Boot — метрики — начальные 2.0.7 или более поздние версии
* [Ресурс Application Insights](./create-new-resource.md)

Шаги:

1. Обновите файл pom.xml приложения Spring Boot и включите в него следующую зависимость:

    ```XML
    <dependency> 
          <groupId>com.microsoft.azure</groupId>
          <artifactId>azure-spring-boot-metrics-starter</artifactId>
          <version>2.0.7</version>
    </dependency>
    ```
1. Обновите файл application.properties или yml с помощью ключа инструментирования Application Insights, используя следующее свойство:

     `azure.application-insights.instrumentation-key=<your-instrumentation-key-here>`
3. Создайте и запустите приложение.
4. Выполнив указанные выше инструкции, вы сможете приступить к работе с предварительно агрегированными метриками, автоматически собранными в Azure Monitor. Подробнее о том, как настроить начальное приложение Spring Boot Application Insights, см. на странице с [файлом сведений на GitHub](https://github.com/Microsoft/azure-spring-boot/releases/latest).

Метрики по умолчанию:

*    Автоматически настроенные метрики для Tomcat, виртуальной машины Java, метрики Logback, Log4J, метрики работоспособности системы, метрики процессора, FileDescriptorMetrics.
*    Например, если Netflix Hystrix есть в пути к классу, эти метрики также будут получены. 
*    Доступ к следующим метрикам можно получить, добавив соответствующие компоненты. 
        - Качеметрикс (Каффеинекаче, EhCache2, Гуавакаче, Хазелкасткаче, Жкаче)
        - DataBaseTableMetrics 
        - HibernateMetrics 
        - JettyMetrics 
        - Метрики OkHttp3 
        - Метрики Kafka 



Отключение автоматического сбора метрик: 

- Метрики виртуальной машины Java: 
    - management.metrics.binders.jvm.enabled=false 
- Метрики Logback: 
    - management.metrics.binders.logback.enabled=false
- Метрики времени работы: 
    - management.metrics.binders.uptime.enabled=false 
- Метрики процессора:
    -  management.metrics.binders.processor.enabled=false 
- FileDescriptorMetrics:
    - management.Metrics.binders.Files.Enabled=false 
- Метрики Hystrix если библиотека на classpath: 
    - management.Metrics.binders.hystrix.Enabled=false 
- Метрики AspectJ (если библиотека указана в пути к классу): 
    - spring.AOP.Enabled=false 

> [!NOTE]
> Укажите свойства выше в файле application.properties или application.yml своего приложения Spring Boot.

## <a name="use-micrometer-with-non-spring-boot-web-applications"></a>Использование Micrometer со сторонними веб-приложениями

Добавьте следующие зависимости в файл pom.xml или build.gradle:

* Application Insights Web Auto 2.5.0 или более поздней версии
* Реестр Azure Micrometer 1.1.0 или более поздней версии.
* [Ресурс Application Insights](./create-new-resource.md)

Шаги:

1. Добавьте следующие зависимости в файл pom.xml или build.gradle:

    ```XML
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-registry-azure-monitor</artifactId>
            <version>1.1.0</version>
        </dependency>

        <dependency>
            <groupId>com.microsoft.azure</groupId>
            <artifactId>applicationinsights-web-auto</artifactId>
            <version>2.5.0</version>
        </dependency>
     ```

2. Разместить `ApplicationInsights.xml` файл в папке ресурсов:

    ```XML
    <?xml version="1.0" encoding="utf-8"?>
    <ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings" schemaVersion="2014-05-30">

       <!-- The key from the portal: -->
       <InstrumentationKey>** Your instrumentation key **</InstrumentationKey>

       <!-- HTTP request component (not required for bare API) -->
       <TelemetryModules>
          <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebRequestTrackingTelemetryModule"/>
          <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebSessionTrackingTelemetryModule"/>
          <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebUserTrackingTelemetryModule"/>
       </TelemetryModules>

       <!-- Events correlation (not required for bare API) -->
       <!-- These initializers add context data to each event -->
       <TelemetryInitializers>
          <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebOperationIdTelemetryInitializer"/>
          <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebOperationNameTelemetryInitializer"/>
          <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebSessionTelemetryInitializer"/>
          <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebUserTelemetryInitializer"/>
          <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebUserAgentTelemetryInitializer"/>
       </TelemetryInitializers>

    </ApplicationInsights>
    ```

3. Пример класса Servlet (выводит метрику таймера):

    ```Java
        @WebServlet("/hello")
        public class TimedDemo extends HttpServlet {

          private static final long serialVersionUID = -4751096228274971485L;

          @Override
          @Timed(value = "hello.world")
          protected void doGet(HttpServletRequest request, HttpServletResponse response)
              throws ServletException, IOException {

            response.getWriter().println("Hello World!");
            MeterRegistry registry = (MeterRegistry) getServletContext().getAttribute("AzureMonitorMeterRegistry");

        //create new Timer metric
            Timer sampleTimer = registry.timer("timer");
            Stream<Integer> infiniteStream = Stream.iterate(0, i -> i+1);
            infiniteStream.limit(10).forEach(integer -> {
              try {
                Thread.sleep(1000);
                sampleTimer.record(integer, TimeUnit.MILLISECONDS);
              } catch (Exception e) {}
               });
          }
          @Override
          public void init() throws ServletException {
            System.out.println("Servlet " + this.getServletName() + " has started");
          }
          @Override
          public void destroy() {
            System.out.println("Servlet " + this.getServletName() + " has stopped");
          }

        }

    ```

4. Пример класса конфигурации:

    ```Java
         @WebListener
         public class MeterRegistryConfiguration implements ServletContextListener {

           @Override
           public void contextInitialized(ServletContextEvent servletContextEvent) {

         // Create AzureMonitorMeterRegistry
           private final AzureMonitorConfig config = new AzureMonitorConfig() {
             @Override
             public String get(String key) {
                 return null;
             }
            @Override
               public Duration step() {
                 return Duration.ofSeconds(60);}

             @Override
             public boolean enabled() {
                 return false;
             }
         };

      MeterRegistry azureMeterRegistry = AzureMonitorMeterRegistry.builder(config);

             //set the config to be used elsewhere
             servletContextEvent.getServletContext().setAttribute("AzureMonitorMeterRegistry", azureMeterRegistry);

           }

           @Override
           public void contextDestroyed(ServletContextEvent servletContextEvent) {

           }
         }
    ```

Дополнительные сведения о метриках см. в статье [Micrometer documentation](https://micrometer.io/docs/) (Документация по Micrometer).

Другой пример кода для создания различных типов метрик можно найти в[официальном репозитории Микрометер GitHub](https://github.com/micrometer-metrics/micrometer/tree/master/samples/micrometer-samples-core/src/main/java/io/micrometer/core/samples).

## <a name="how-to-bind-additional-metrics-collection"></a>Связывание дополнительной коллекции метрик

### <a name="springbootspring"></a>SpringBoot/Spring

Создайте компонент соответствующей категории метрик. Например, предположим, нам нужен кэш метрик Guava:

```Java
    @Bean
    GuavaCacheMetrics guavaCacheMetrics() {
        Return new GuavaCacheMetrics();
    }
```
Несколько метрик не включены по умолчанию, но их можно связать указанным выше способом. Полный список см. в [официальном репозитории GitHub микрометер](https://github.com/micrometer-metrics/micrometer/tree/master/micrometer-core/src/main/java/io/micrometer/core/instrument/binder ).

### <a name="non-spring-apps"></a>Стороннее приложение
Добавьте следующий код привязки в файл конфигурации:
```Java 
    New GuavaCacheMetrics().bind(registry);
```

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о Микрометер см. в официальной [документации микрометер](https://micrometer.io/docs).
* Дополнительные сведения о пружины в Azure см. в официальной [пружине в документации по Azure](/java/azure/spring-framework/).
