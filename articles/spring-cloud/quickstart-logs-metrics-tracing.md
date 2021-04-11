---
title: Краткое руководство. Мониторинг приложений Azure Spring Cloud с помощью журналов, метрик и трассировки
description: Используйте потоковую передачу журналов, анализ журналов, метрики и трассировку для мониторинга примеров приложений PiggyMetrics в Azure Spring Cloud.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: quickstart
ms.date: 08/04/2020
ms.custom: devx-track-java, devx-track-azurecli
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: 75c50e6342a00ef37215d9e961503dd7ffb0d205
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104878862"
---
# <a name="quickstart-monitoring-azure-spring-cloud-apps-with-logs-metrics-and-tracing"></a>Краткое руководство. Мониторинг приложений Azure Spring Cloud с помощью журналов, метрик и трассировки

::: zone pivot="programming-language-csharp"
Встроенные возможности мониторинга Azure Spring Cloud позволяют легко выполнять отладку и мониторинг при возникновении сложных проблем. Azure Spring Cloud интегрирует [распределенную трассировку](https://steeltoe.io/docs/3/tracing/distributed-tracing) Steeltoe с Azure [Application Insights](../azure-monitor/app/app-insights-overview.md). Эта интеграция предоставляет широкие возможности ведения журналов, анализа метрик и распределенной трассировки с помощью портала Azure.

В следующих процедурах объясняется, как использовать потоковую передачу журналов, анализ журналов, метрики и распределенную трассировку с примером приложения, развернутого в предыдущих руководствах.

## <a name="prerequisites"></a>Предварительные требования

* Пройдите краткие руководства, приведенные в этой серии:

  * [Подготовка службы Azure Spring Cloud](spring-cloud-quickstart-provision-service-instance.md).
  * [Настройка сервера конфигурации Azure Spring Cloud](spring-cloud-quickstart-setup-config-server.md).
  * [Сборка и развертывание приложений](spring-cloud-quickstart-deploy-apps.md)

## <a name="logs"></a>Журналы

Журналы в Azure Spring Cloud можно просматривать двумя способами: с помощью **потоковой передачи журналов** в режиме реального времени для каждого экземпляра приложения или **анализа журналов** для агрегированных журналов с расширенными возможностями запросов.

### <a name="log-streaming"></a>Потоковая передача журналов

Потоковую передачу журнала можно задействовать в Azure CLI с помощью следующей команды.

```azurecli
az spring-cloud app logs -n solar-system-weather -f
```

Результат будет выглядеть примерно так:

```output
=> ConnectionId:0HM2HOMHT82UK => RequestPath:/weatherforecast RequestId:0HM2HOMHT82UK:00000003, SpanId:|e8c1682e-46518cc0202c5fd9., TraceId:e8c1682e-46518cc0202c5fd9, ParentId: => Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather.Controllers.WeatherForecastController.Get (Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather)
Executing action method Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather.Controllers.WeatherForecastController.Get (Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather) - Validation state: Valid
←[40m←[32minfo←[39m←[22m←[49m: Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather.Controllers.WeatherForecastController[0]

=> ConnectionId:0HM2HOMHT82UK => RequestPath:/weatherforecast RequestId:0HM2HOMHT82UK:00000003, SpanId:|e8c1682e-46518cc0202c5fd9., TraceId:e8c1682e-46518cc0202c5fd9, ParentId: => Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather.Controllers.WeatherForecastController.Get (Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather)
Retrieved weather data from 4 planets
←[40m←[32minfo←[39m←[22m←[49m: Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker[2]

=> ConnectionId:0HM2HOMHT82UK => RequestPath:/weatherforecast RequestId:0HM2HOMHT82UK:00000003, SpanId:|e8c1682e-46518cc0202c5fd9., TraceId:e8c1682e-46518cc0202c5fd9, ParentId: => Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather.Controllers.WeatherForecastController.Get (Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather)
Executing ObjectResult, writing value of type 'System.Collections.Generic.KeyValuePair`2[[System.String, System.Private.CoreLib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=7cec85d7bea7798e],[System.String, System.Private.CoreLib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=7cec85d7bea7798e]][]'.
←[40m←[32minfo←[39m←[22m←[49m: Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker[2]
```

> [!TIP]
> Используйте `az spring-cloud app logs -h` для просмотра дополнительных параметров и функциональных возможностей потоковой передачи журналов.

### <a name="log-analytics"></a>Log Analytics

1. На портале Azure перейдите на страницу **Служба | Общие сведения** и выберите **Журналы** в разделе **Мониторинг**. Выберите **Выполнить** для одного из примеров запросов Azure Spring Cloud.

   [ ![Запись анализа журналов](media/spring-cloud-quickstart-logs-metrics-tracing/logs-entry.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/logs-entry.png#lightbox)
    
1. Измените запрос, чтобы удалить предложения WHERE, которые ограничивают отображение предупреждений и журналов ошибок.

1. Затем выберите `Run`, чтобы просмотреть журналы. Дополнительные рекомендации по написанию запросов см. в [документации Azure Log Analytics](../azure-monitor/logs/get-started-queries.md).

   [ ![Добавление запроса на анализ журналов — Steeltoe](media/spring-cloud-quickstart-logs-metrics-tracing/logs-query-steeltoe.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/logs-query-steeltoe.png#lightbox)

## <a name="metrics"></a>Метрики

1. На портале Azure перейдите на страницу **служба | Общие сведения** и выберите **Метрики** в разделе **Мониторинг**. Добавьте свою первую метрику, выбрав одну из метрик .NET в разделе **Производительность (.NET)** или **Запрос (.NET)** в раскрывающемся списке **Метрика**, и выберите значение `Avg` в раскрывающемся списке **Агрегирование**, чтобы просмотреть временную шкалу для такой метрики.

   [ ![Добавление метрик — Steeltoe](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-basic-cpu-steeltoe.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-basic-cpu-steeltoe.png#lightbox)
    
1. На панели инструментов щелкните **Добавить фильтр** и выберите `App=solar-system-weather`, чтобы увидеть загрузку ЦП только для приложения **solar-system-weather**.

   [ ![Использование фильтра в разделе "Метрики" — Steeltoe](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-filter-steeltoe.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-filter-steeltoe.png#lightbox)

1. Закройте созданный на предыдущем шаге фильтр, нажмите **Apply Splitting** (Применить разделение) и выберите `App` в поле **Значения**, чтобы увидеть загрузку ЦП, создаваемую различными приложениями.

   [ ![Применение разделения для метрик — Steeltoe](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-split-steeltoe.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-split-steeltoe.png#lightbox)

## <a name="distributed-tracing"></a>Распределенная трассировка

1. На портале Azure перейдите на страницу **служба | Общие сведения** и выберите **Distributed tracing** (Распределенная трассировка) в разделе **Мониторинг**. Затем перейдите на вкладку **View application map** (Просмотр схемы приложения) справа.

   [ ![Ввод данных для распределенной трассировки — Steeltoe](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-entry.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-entry.png#lightbox)

1. Теперь вы можете видеть состояние вызовов между приложениями. 

   [ ![Обзор распределенной трассировки — Steeltoe](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-overview-steeltoe.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-overview-steeltoe.png#lightbox)
    
1. Выберите ссылку между **solar-system-weather** и **planet-weather-provider**, чтобы получить дополнительные сведения, например сведения о самых медленных вызовах методов HTTP.

   [ ![Распределенная трассировка — Steeltoe](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-call-steeltoe.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-call-steeltoe.png#lightbox)
    
1. Наконец, выберите **Анализ работы**, чтобы воспользоваться расширенными возможностями встроенных средств анализа производительности.

   [ ![Производительность распределенной трассировки — Steeltoe](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-performance-steeltoe.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-performance-steeltoe.png#lightbox)
::: zone-end

::: zone pivot="programming-language-java"
Встроенные возможности мониторинга Azure Spring Cloud позволяют легко выполнять отладку и мониторинг при возникновении сложных проблем. Azure Spring Cloud интегрирует [Spring Cloud Sleuth](https://spring.io/projects/spring-cloud-sleuth) с Azure [Application Insights](../azure-monitor/app/app-insights-overview.md). Эта интеграция предоставляет широкие возможности ведения журналов, анализа метрик и распределенной трассировки с помощью портала Azure. В следующих процедурах объясняется, как использовать потоковую передачу журналов, анализ журналов, метрики и распределенную трассировку с развернутыми приложениями PiggyMetrics.

## <a name="prerequisites"></a>Предварительные требования

Выполните предыдущие шаги. 

* [Подготовка к работе экземпляра Azure Spring Cloud](spring-cloud-quickstart-provision-service-instance.md)
* [Настройка сервера конфигурации](spring-cloud-quickstart-setup-config-server.md)
* [Сборка и развертывание приложений](spring-cloud-quickstart-deploy-apps.md)

## <a name="logs"></a>Журналы

Журналы в Azure Spring Cloud можно просматривать двумя способами: с помощью **потоковой передачи журналов** в режиме реального времени для каждого экземпляра приложения или **анализа журналов** для агрегированных журналов с расширенными возможностями запросов.

### <a name="log-streaming"></a>Потоковая передача журналов

#### <a name="cli"></a>[CLI](#tab/Azure-CLI)

Потоковую передачу журнала можно задействовать в Azure CLI с помощью следующей команды.

```azurecli
az spring-cloud app logs -s <service instance name> -g <resource group name> -n gateway -f
```

Журналы отобразятся в следующем виде:

[ ![Потоковая передача журналов из Azure CLI](media/spring-cloud-quickstart-logs-metrics-tracing/logs-streaming-cli.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/logs-streaming-cli.png#lightbox)

> [!TIP]
> Используйте `az spring-cloud app logs -h` для просмотра дополнительных параметров и функциональных возможностей потоковой передачи журналов.

#### <a name="intellij"></a>[IntelliJ](#tab/IntelliJ)

Чтобы получить журналы с помощью Azure Toolkit for IntelliJ, выполните следующие действия:

1. Выберите **Azure Explorer** и **Spring Cloud**.

1. Щелкните правой кнопкой мыши запущенное приложение.

1. Выберите **Streaming Logs** (Журналы потоковой передачи) из раскрывающегося списка.

   ![Выбор журналов потоковой передачи](media/spring-cloud-intellij-howto/streaming-logs.png)
    
1. Выберите **Экземпляр**.

   ![Выбор экземпляра](media/spring-cloud-intellij-howto/select-instance.png)
    
1. Журнал потоковой передачи отобразится в окне выходных данных.

   ![Вывод журнала потоковой передачи](media/spring-cloud-intellij-howto/streaming-log-output.png)

---
### <a name="log-analytics"></a>Log Analytics

1. Перейдите на страницу **служба | Общие сведения** и выберите **Журналы** в разделе **Мониторинг**. Щелкните **Выполнить** для одного из примеров запросов Azure Spring Cloud. 

   [ ![Запись анализа журналов](media/spring-cloud-quickstart-logs-metrics-tracing/logs-entry.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/logs-entry.png#lightbox)
    
1. Затем отобразятся отфильтрованные журналы. Дополнительные рекомендации по написанию запросов см. в [документации Azure Log Analytics](../azure-monitor/logs/get-started-queries.md).

   [ ![Добавление запроса на анализ журналов](media/spring-cloud-quickstart-logs-metrics-tracing/logs-query.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/logs-query.png#lightbox)

## <a name="metrics"></a>Метрики

1. Перейдите на страницу **служба | Общие сведения** и выберите **Метрики** в разделе **Мониторинг**. Добавьте свою первую метрику, выбрав значение `system.cpu.usage` для параметра **Метрика** и `Avg` для параметра **Агрегирование**, чтобы увидеть временную шкалу для общей загрузки ЦП.

   [ ![Ввод данных для метрики](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-basic-cpu.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-basic-cpu.png#lightbox)
    
1. На панели инструментов сверху щелкните **Добавить фильтр**, выберите `App=Gateway`, чтобы увидеть загрузку ЦП только для приложения **gateway**.

   [ ![Использование фильтра в разделе "Метрики"](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-filter.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-filter.png#lightbox)

1. Закройте созданный ранее фильтр, нажмите **Apply Splitting** (Применить разделение) и выберите `App` в поле **Значения**, чтобы увидеть загрузку ЦП, создаваемую различными приложениями.

   [ ![Применение разделения для метрик](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-split.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/metrics-split.png#lightbox)

## <a name="distributed-tracing"></a>Распределенная трассировка

1. Перейдите на страницу **служба | Общие сведения** и выберите **Distributed tracing** (Распределенная трассировка) в разделе **Мониторинг**. Затем перейдите на вкладку **View application map** (Просмотр схемы приложения) справа.

   [ ![Ввод данных для распределенной трассировки](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-entry.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-entry.png#lightbox)

1. Теперь вы можете видеть состояние вызовов между приложениями PiggyMetrics. 

   [ ![Обзор распределенной трассировки](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-overview.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-overview.png#lightbox)
    
1. Щелкните ссылку между **gateway** и **account-service**, чтобы просмотреть дополнительные сведения, например самые медленные вызовы методов HTTP.

   [ ![Распределенная трассировка](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-call.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-call.png#lightbox)
    
1. Наконец, щелкните **Анализ работы**, чтобы воспользоваться расширенными возможностями встроенных средств анализа производительности.

   [ ![Распределенная трассировка производительности](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-performance.png) ](media/spring-cloud-quickstart-logs-metrics-tracing/tracing-performance.png#lightbox)

::: zone-end

## <a name="clean-up-resources"></a>Очистка ресурсов

В этих кратких руководствах показано, как создать ресурсы Azure, за которые будет взиматься плата, если они останутся в вашей подписке. Если эти ресурсы не понадобятся в будущем, вы можете удалить группу ресурсов с помощью портала, или выполнив приведенную ниже команду в Cloud Shell.

```azurecli
az group delete --name <your resource group name; for example: helloworld-1558400876966-rg> --yes
```

В предыдущем кратком руководстве также показано, как задать имя группы ресурсов по умолчанию. Если вы не планируете перейти к следующему краткому руководству, очистите это значение по умолчанию, выполнив такую команду CLI:

```azurecli
az configure --defaults group=
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о возможностях мониторинга в Azure Spring Cloud см. в следующих статьях:

> [!div class="nextstepaction"]
> [Службы диагностики](diagnostic-services.md)
>
> [Распределенная трассировка](spring-cloud-howto-distributed-tracing.md)
>
> [Потоковая передача журналов в режиме реального времени](spring-cloud-howto-log-streaming.md)
