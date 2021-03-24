---
title: Использование Application Insights агента In-Process Java в Azure Веснного облака
description: Мониторинг приложений и микрослужб с помощью Application Insights агента In-Process Java в Azure Веснного облака.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 12/04/2020
ms.custom: devx-track-java
ms.openlocfilehash: f6f1ed1a3e09397a720dfd4d842b79cd88aa738d
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104878559"
---
# <a name="application-insights-java-in-process-agent-in-azure-spring-cloud-preview"></a>Application Insights агента In-Process Java в Azure Веснного облака (Предварительная версия)

В этом документе объясняется, как отслеживать приложения и микрослужбы с помощью Application Insights агента Java в Azure Веснного облака. 

С помощью этой функции можно:

* Поиск данных трассировки с использованием фильтров.
* Просмотр схемы зависимостей микрослужб.
* Проверьте производительность запросов.
* Мониторинг динамических метрик в режиме реального времени.
* Проверьте ошибки запросов.
* Проверьте метрики приложения.

Application Insights предоставить множество наблюдаемых перспектив, в том числе:

* Схема сопоставления приложений
* Производительность
* Сбои
* Метрики
* Интерактивные метрики
* Доступность

## <a name="enable-java-in-process-agent-for-application-insights"></a>Включить агент In-Process Java для Application Insights

Включите функцию предварительной версии агента Java In-Process с помощью следующей процедуры.

1. Перейдите на страницу обзора службы экземпляра службы.
2. В колонке мониторинг щелкните **Application Insights** запись.
3. Нажмите кнопку " **включить Application Insights** ", чтобы включить интеграцию **Application Insights** .
4. Выберите существующий экземпляр Application Insights или создайте новый.
5. Цыпленок **Включение внутрипроцессного агента Java** для включения функции предварительного просмотра Java-агента. Здесь можно также настроить частоту выборки от 0 до 100.
6.  Щелкните **Сохранить**, чтобы сохранить изменение.

## <a name="portal"></a>Портал

1. Перейдите к **службе |** Страница "Обзор" и выберите **Application Insights** в разделе " **мониторинг** ". 
2. Щелкните **включить Application Insights** , чтобы включить Application Insights в Azure веснного облака.
3. Щелкните **включить внутрипроцессный агент Java** , чтобы включить функцию предварительной версии Java IPA. Если функция предварительной версии IPA включена, можно настроить одну дополнительную частоту выборки (по умолчанию 10,0%).

  [![IPA 0](media/spring-cloud-application-insights/insights-process-agent-0.png)](media/spring-cloud-application-insights/insights-process-agent-0.png)

## <a name="using-the-application-insights-feature"></a>Использование функции Application Insights

Если функция **Application Insights** включена, вы можете:

В области навигации слева щелкните **Application Insights** , чтобы перейти на страницу **обзора** Application Insights. 

* Щелкните **схема приложения** , чтобы просмотреть состояние вызовов между приложениями.

  [![IPA 2](media/spring-cloud-application-insights/insights-process-agent-2-map.png)](media/spring-cloud-application-insights/insights-process-agent-2-map.png)

* Щелкните ссылку между Customers-Service и `petclinic` Просмотрите дополнительные сведения, например запрос из SQL.

* В области навигации слева щелкните **производительность** , чтобы просмотреть данные о производительности для всех операций приложений, а также зависимости и роли.

  [![IPA 4](media/spring-cloud-application-insights/insights-process-agent-4-performance.png)](media/spring-cloud-application-insights/insights-process-agent-4-performance.png)

* В области навигации слева щелкните **ошибки** , чтобы проверить, не было ли что-либо непредвиденное из ваших приложений.

  [![IPA 6](media/spring-cloud-application-insights/insights-process-agent-6-failures.png)](media/spring-cloud-application-insights/insights-process-agent-6-failures.png)

* В области навигации слева щелкните **метрики** и выберите пространство имен. Вы увидите метрики пружинной загрузки и пользовательские метрики, если они есть.

  [![IPA 7](media/spring-cloud-application-insights/insights-process-agent-5-metrics.png)](media/spring-cloud-application-insights/insights-process-agent-5-metrics.png)

* В области навигации слева щелкните **динамические метрики** , чтобы просмотреть метрики в реальном времени для различных измерений.

  [![IPA 8](media/spring-cloud-application-insights/petclinic-microservices-live-metrics.jpg)](media/spring-cloud-application-insights/petclinic-microservices-live-metrics.jpg)

* В левой области навигации щелкните **доступность** , чтобы отслеживать доступность и скорость реагирования веб-приложений путем создания [тестов доступности в Application Insights](../azure-monitor/app/monitor-web-app-availability.md).

  [![IPA 9](media/spring-cloud-application-insights/petclinic-microservices-availability.jpg)](media/spring-cloud-application-insights/petclinic-microservices-availability.jpg)

## <a name="arm-template"></a>Шаблон ARM
Чтобы использовать шаблон Azure Resource Manager, скопируйте следующее содержимое в `azuredeploy.json` .

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        {
            "type": "Microsoft.AppPlatform/Spring",
            "name": "customize this",
            "apiVersion": "2020-07-01",
            "location": "[resourceGroup().location]",
            "resources": [
                {
                    "type": "monitoringSettings",
                    "apiVersion": "2020-11-01-preview",
                    "name": "default",
                    "properties": {
                        "appInsightsInstrumentationKey": "00000000-0000-0000-0000-000000000000",
                        "appInsightsSamplingRate": 88.0
                    },
                    "dependsOn": [
                        "[resourceId('Microsoft.AppPlatform/Spring', 'customize this')]"
                    ]
                }
            ],
            "properties": {}
        }
    ]
}
```

## <a name="cli"></a>CLI
Примените шаблон ARM с помощью команды CLI:

* Для существующего облачного экземпляра Azure весны:

```azurecli
az spring-cloud app-insights update [--app-insights/--app-insights-key] "assignedName" [--sampling-rate] "samplingRate" –name "assignedName" –resource-group "resourceGroupName"
```
* Для вновь созданного облачного экземпляра Azure весны:

```azurecli
az spring-cloud create/update [--app-insights]/[--app-insights-key] "assignedName" --disable-app-insights false --enable-java-agent true --name "assignedName" –resource-group "resourceGroupName"
```
* Отключение App-Insight:

```azurecli
az spring-cloud app-insights update --disable –name "assignedName" –resource-group "resourceGroupName"

```

## <a name="see-also"></a>См. также раздел
* [Использование распределенной трассировки в Azure Spring Cloud](spring-cloud-howto-distributed-tracing.md)
* [Анализ журналов и метрик](diagnostic-services.md)
* [Потоковая передача журналов в режиме реального времени](spring-cloud-howto-log-streaming.md)