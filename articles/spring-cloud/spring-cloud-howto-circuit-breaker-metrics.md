---
title: Собирайте метрики отResilience4Jного прерывания весны облака
description: Как получить метрики для Resilience4Jного прерывания весны в облаке.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 12/15/2020
ms.custom: devx-track-java
ms.openlocfilehash: 0227fcebd6d5a136e3306573b7b5aa0d6e39ba1f
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102122805"
---
# <a name="collect-spring-cloud-resilience4j-circuit-breaker-metrics-preview"></a>Собирайте метрики отResilience4Jного прерывания весны облака (Предварительная версия)

В этом документе объясняется, как получить метрики отResilience4jного прерывания весны облака с помощью агента обработки Java в Application Insights.  С помощью этой функции можно отслеживать метрики автоматического выключения resilience4j из Application Insights.

Мы используем [демонстрацию](https://github.com/spring-cloud-samples/spring-cloud-circuitbreaker-demo) , чтобы продемонстрировать, как это работает.

## <a name="prerequisites"></a>Предварительные требования

* Пошаговое руководств по установке агента In-Process Java из [агента In-Process Java для Application Insights](./spring-cloud-howto-application-insights.md#enable-java-in-process-agent-for-application-insights). 

* Включите сбор измерений для метрик resilience4j из [Application Insights Guide](../azure-monitor/app/pre-aggregated-metrics-log-metrics.md#custom-metrics-dimensions-and-pre-aggregation).

* Установите Git, Maven и Java, если они еще не используются на компьютере разработчика.

## <a name="build-and-deploy-apps"></a>Сборка и развертывание приложений

Следующая процедура предназначена для сборки и развертывания приложений.

1. Клонировать и создать демонстрационный репозиторий.

```bash
git clone https://github.com/spring-cloud-samples/spring-cloud-circuitbreaker-demo.git
cd spring-cloud-circuitbreaker-demo && mvn clean package -DskipTests
```

2. Создание приложений с помощью конечных точек

```azurecli
az spring-cloud app create --name resilience4j --assign-endpoint \
    -s ${asc-service-name} -g ${asc-resource-group}
az spring-cloud app create --name reactive-resilience4j --assign-endpoint \
    -s ${asc-service-name} -g ${asc-resource-group}
```

3. Развертывание приложений.

```azurecli
az spring-cloud app deploy -n resilience4j \
    --jar-path ./spring-cloud-circuitbreaker-demo-resilience4j/target/spring-cloud-circuitbreaker-demo-resilience4j-0.0.1.BUILD-SNAPSHOT.jar \
    -s ${service_name} -g ${resource_group}
az spring-cloud app deploy -n reactive-resilience4j \
    --jar-path ./spring-cloud-circuitbreaker-demo-reactive-resilience4j/target/spring-cloud-circuitbreaker-demo-reactive-resilience4j-0.0.1.BUILD-SNAPSHOT.jar \
    -s ${service_name} -g ${resource_group}
```

> [!Note]
>
> * Включите требуемую зависимость для Resilience4j:
>
>   ```xml
>   <dependency>
>       <groupId>io.github.resilience4j</groupId>
>       <artifactId>resilience4j-micrometer</artifactId>
>   </dependency>
>   <dependency>
>       <groupId>org.springframework.cloud</groupId>
>       <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
>   </dependency>
>   ```
> * Код клиента должен использовать API из `CircuitBreakerFactory` , который реализуется `bean` автоматически при включении аварийного прерывания Cloud цепь Starter. Дополнительные сведения см. в разделе [Весна Cloud замыкание](https://spring.io/projects/spring-cloud-circuitbreaker#overview).
>
> * Следующие 2 зависимости имеют конфликты с resilient4j пакетами выше.  Убедитесь, что клиент не включил их.
>
>   ```xml
>   <dependency>
>       <groupId>org.springframework.cloud</groupId>
>       <artifactId>spring-cloud-starter-sleuth</artifactId>
>   </dependency>
>   <dependency>
>       <groupId>org.springframework.cloud</groupId>
>       <artifactId>spring-cloud-starter-zipkin</artifactId>
>   </dependency>
>   ```
>
>
> Перейдите по URL-адресу, предоставленному приложениями шлюза, и получите доступ к конечной точке из [пружины-Cloud-цепью-Break-Demo](https://github.com/spring-cloud-samples/spring-cloud-circuitbreaker-demo) , как показано ниже.
>
>   ```console
>   /get
>   /get/delay/{seconds}
>   /get/fluxdelay/{seconds}
>   ```

## <a name="locate-resilence4j-metrics-from-portal"></a>Определение метрик Resilence4j на портале

1. Выберите колонку **Application Insights** на портале Azure пружины Cloud и щелкните **Application Insights**.

   [![resilience4J 0](media/spring-cloud-resilience4j/resilience4J-0.png)](media/spring-cloud-resilience4j/resilience4J-0.PNG)

2. На странице **Application Insights** выберите **метрики** .  Выберите **Azure. applicationinsights** из **пространства имен метрик**.  Также выберите **resilience4j_circuitbreaker_buffered_calls** метрики со **средним значением**.

   [![resilience4J 1](media/spring-cloud-resilience4j/resilience4J-1.png)](media/spring-cloud-resilience4j/resilience4J-1.PNG)

3. Выберите **resilience4j_circuitbreaker_calls** метрики и **среднее значение**.

   [![resilience4J 2](media/spring-cloud-resilience4j/resilience4J-2.png)](media/spring-cloud-resilience4j/resilience4J-2.PNG)

4. Выберите **resilience4j_circuitbreaker_calls**  метрики и **среднее значение**.  Нажмите кнопку **Добавить фильтр**, а затем выберите имя как **креатеневаккаунт**.

   [![resilience4J 3](media/spring-cloud-resilience4j/resilience4J-3.png)](media/spring-cloud-resilience4j/resilience4J-3.PNG)

5. Выберите **resilience4j_circuitbreaker_calls**  метрики и **среднее значение**.  Затем щелкните **Применить разделение** и выберите **тип**.

   [![resilience4J 4](media/spring-cloud-resilience4j/resilience4J-4.png)](media/spring-cloud-resilience4j/resilience4J-4.PNG)

6. Выберите **resilience4j_circuitbreaker_calls**, "**resilience4j_circuitbreaker_buffered_calls** и **resilience4j_circuitbreaker_slow_calls** метрики со **средним значением**.

   [![resilience4J 5](media/spring-cloud-resilience4j/resilience4j-5.png)](media/spring-cloud-resilience4j/resilience4j-5.PNG)

## <a name="see-also"></a>См. также раздел

* [Application Insights](./spring-cloud-howto-application-insights.md)
* [Распределенная трассировка](spring-cloud-tutorial-distributed-tracing.md)
* [Панель мониторинга выключения цепи](spring-cloud-tutorial-circuit-breaker.md)