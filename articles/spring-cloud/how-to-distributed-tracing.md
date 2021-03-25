---
title: Использование распределенной трассировки в Azure Spring Cloud
description: Сведения о том, как использовать распределенную трассировку в Spring Cloud с помощью Azure Application Insights.
author: bmitchell287
ms.service: spring-cloud
ms.topic: how-to
ms.date: 10/06/2019
ms.author: brendm
ms.custom: devx-track-java
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: f55a82eeddc8d4515b0f1333b615244976975097
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104878243"
---
# <a name="use-distributed-tracing-with-azure-spring-cloud"></a>Использование распределенной трассировки в Azure Spring Cloud

Инструменты распределенной трассировки в Azure Spring Cloud позволяют легко выполнять отладку и мониторинг сложных проблем. Azure Spring Cloud интегрирует [Spring Cloud Sleuth](https://spring.io/projects/spring-cloud-sleuth) с Azure [Application Insights](../azure-monitor/app/app-insights-overview.md). Эта интеграция предоставляет мощные возможности распределенной трассировки через портал Azure.

::: zone pivot="programming-language-csharp"
Из этой статьи вы узнаете, как разрешить приложению .NET Core Стилтое использовать распределенную трассировку.

## <a name="prerequisites"></a>Предварительные требования

Для выполнения этих процедур требуется приложение Стилтое, которое уже [подготовлено к развертыванию в Azure веснного облака](how-to-prepare-app-deployment.md).

## <a name="dependencies"></a>Зависимости

Для Стилтое 2.4.4 Добавьте следующие пакеты NuGet:

* [Стилтое. Management. ТраЦингкоре](https://www.nuget.org/packages/Steeltoe.Management.TracingCore/)
* [Стилтое. Management. Експортеркоре](https://www.nuget.org/packages/Microsoft.Azure.SpringCloud.Client/)

Для Стилтое 3.0.0 добавьте следующий пакет NuGet:

* [Стилтое. Management. ТраЦингкоре](https://www.nuget.org/packages/Steeltoe.Management.TracingCore/)

## <a name="update-startupcs"></a>Обновление запуска. CS

1. Для Стилтое 2.4.4 вызовите `AddDistributedTracing` и `AddZipkinExporter` в `ConfigureServices` методе.

   ```csharp
   public void ConfigureServices(IServiceCollection services)
   {
       services.AddDistributedTracing(Configuration);
       services.AddZipkinExporter(Configuration);
   }
   ```

   Для Стилтое 3.0.0 вызовите `AddDistributedTracing` в `ConfigureServices` методе.

   ```csharp
   public void ConfigureServices(IServiceCollection services)
   {
       services.AddDistributedTracing(Configuration, builder => builder.UseZipkinWithTraceOptions(services));
   }
   ```

1. Для Стилтое 2.4.4 вызовите `UseTracingExporter` в `Configure` методе.

   ```csharp
   public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
   {
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllers();
        });
        app.UseTracingExporter();
   }
   ```

   Для Стилтое 3.0.0 в методе не требуется никаких изменений `Configure` .

## <a name="update-configuration"></a>Обновление конфигурации

Добавьте следующие параметры в источник конфигурации, который будет использоваться при запуске приложения в Azure Веснного облака:

1. `management.tracing.alwaysSample` — присвойте значение True.

2. Если вы хотите просмотреть диапазоны трассировки, отправляемые между сервером Еурека, сервером конфигурации и пользовательскими приложениями: значение `management.tracing.egressIgnorePattern` "/API/v2/spans |/v2/Apps/.*/пермиссионс |/Еурека/.*| /оаус/. * ".

Например, *appsettings.jsв* будет содержать следующие свойства:
 
```json
"management": {
    "tracing": {
      "alwaysSample": true,
      "egressIgnorePattern": "/api/v2/spans|/v2/apps/.*/permissions|/eureka/.*|/oauth/.*"
    }
  }
```

Дополнительные сведения о распределенной трассировке в приложениях .NET Core Стилтое см. в разделе [Распределенная трассировка](https://steeltoe.io/docs/3/tracing/distributed-tracing) в документации по стилтое.
::: zone-end
::: zone pivot="programming-language-java"
Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Включение распределенной трассировки на портале Azure.
> * Добавление Spring Cloud Sleuth в приложение.
> * Просмотр сопоставления зависимостей для приложений для микрослужб.
> * Поиск данных трассировки с использованием фильтров.

## <a name="prerequisites"></a>Предварительные требования

Чтобы выполнить процедуры из этого учебника, вам потребуется подготовленная и запущенная служба Azure Spring Cloud. Выполните инструкции из руководства по [развертыванию первого облачного приложения Azure весны](spring-cloud-quickstart.md) , чтобы подготавливать и запускать облачную службу Azure весны.

## <a name="add-dependencies"></a>Добавление зависимостей

1. Добавьте следующую строку в конец файла application.properties.

   ```xml
   spring.zipkin.sender.type = web
   ```

   После этого изменения отправитель Zipkin сможет отправлять данные в Интернет.

1. Вы можете пропустить этот шаг, если уже выполнили [руководство по подготовке приложения Azure Spring Cloud](how-to-prepare-app-deployment.md). В противном случае перейдите в локальную среду разработки и измените файл pom.xml, чтобы включить в него зависимость Spring Cloud Sleuth.

    ```xml
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-sleuth</artifactId>
                <version>${spring-cloud-sleuth.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-sleuth</artifactId>
        </dependency>
    </dependencies>
    ```

1. Повторите сборку и развертывание для службы Azure Spring Cloud, чтобы отразить эти изменения.

## <a name="modify-the-sample-rate"></a>Изменение частоты выборки

Частоту сбора данных телеметрии можно изменить, модифицируя частоту выборки. Например, если вы хотите выполнять выборку вдвое реже, откройте файл application.properties и измените следующую строку.

```xml
spring.sleuth.sampler.probability=0.5
```

Вы можете изменить частоту выборки, даже если уже скомпилировали и развернули приложение. Для этого добавьте указанную выше строку в переменную среды с помощью Azure CLI или портала Azure.
::: zone-end

## <a name="enable-application-insights"></a>Включение Application Insights

1. Перейдите к странице своей службы Azure Spring Cloud на портале Azure.
1. На странице **Мониторинг** выберите элемент **Распределенная трассировка**.
1. Выберите **Изменить параметр**, чтобы изменить или добавить новый параметр.
1. Создайте новый запрос Application Insights или выберите имеющийся.
1. Выберите категорию ведения журнала, которую требуется отслеживать, и укажите период удержания (в днях).
1. Выберите **Применить**, чтобы применить новую трассировку.

## <a name="view-the-application-map"></a>Просмотр схемы приложений

Вернитесь на страницу **Распределенная трассировка** и выберите действие **Просмотреть схему приложений**. Проверьте визуальное представление приложения и параметры мониторинга. Сведения об использовании схемы приложения см. в [статье о рассмотрении распределенных приложений](../azure-monitor/app/app-map.md).

## <a name="use-search"></a>Использование поиска

Используйте функцию поиска для запроса других конкретных элементов телеметрии. На странице **Distributed Tracing** (Распределенная трассировка) выберите **Поиск**. Дополнительные сведения об использовании функции поиска см. в статье [Поиск в Application Insights](../azure-monitor/app/diagnostic-search.md).

## <a name="use-application-insights"></a>Использование Application Insights

Кроме функций поиска и схемы приложений Application Insights предоставляет возможности для мониторинга. Найдите на портале Azure имя своего приложения и откройте для него страницу Application Insights, чтобы получить сведения о мониторинге. Дополнительные сведения об использовании этих средств см. в [документации по запросам к журналам Azure Monitor](/azure/data-explorer/kusto/query/).

## <a name="disable-application-insights"></a>Отключение Application Insights

1. Перейдите к странице своей службы Azure Spring Cloud на портале Azure.
1. В разделе **Мониторинг** выберите элемент **Распределенная трассировка**.
1. Щелкните **Отключить**, чтобы отключить Application Insights

## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали, как включить распределенную трассировку в Azure Spring Cloud, и ознакомились с ее возможностями. Дополнительные сведения о привязке служб к приложению см. в статье [Привязка Базы данных Azure Cosmos DB к приложению Azure Spring Cloud](spring-cloud-howto-bind-cosmos.md).