---
title: Анализ событий Azure Service Fabric с помощью Application Insights
description: Ознакомьтесь со сведениями о визуализации и анализе событий с использованием Application Insights для мониторинга и диагностики кластеров Azure Service Fabric.
author: srrengar
ms.topic: conceptual
ms.date: 11/21/2018
ms.author: srrengar
ms.openlocfilehash: f4d77cc3b2f2182b02d47f047070b819daa5f3c1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100589017"
---
# <a name="event-analysis-and-visualization-with-application-insights"></a>Анализ и визуализация событий с помощью Application Insights

Application Insights — это расширяемая платформа для мониторинга и диагностики приложений, предоставляемая в рамках Azure Monitor. Она включает в себя эффективное средство аналитики и запросов, настраиваемую панель мониторинга, визуализации и дополнительные параметры, включая автоматическое оповещение. Интеграция Application Insights с Service Fabric включает средства для Visual Studio и портала Azure, а также некоторые метрики Service Fabric, обеспечивая комплексную фиксацию в журнале всех необходимых данных. Хотя многие журналы создаются автоматически, а данные собираются при использовании Application Insights, мы советуем создавать в своих приложениях дополнительные специализированные журналы для получения более объемной диагностической информации.

Эта статья поможет найти ответ на следующие распространенные вопросы:

* как узнать, что происходит в приложениях и службах, и как собрать данные телеметрии;
* как устранить неполадки в приложениях, особенно если это несколько взаимосвязанных служб;
* как получить метрики выполнения служб, такие как время загрузки страниц и данные об HTTP-запросах.

В этой статье описываются процессы анализа и устранения неполадок в Application Insights. Если вы хотите узнать, как установить и настроить Application Insights с использованием Service Fabric, ознакомьтесь с этим [руководством](service-fabric-tutorial-monitoring-aspnet.md).

## <a name="monitoring-in-application-insights"></a>Мониторинг в Application Insights

Application Insights имеет широкие возможности для использования Service Fabric. На странице "Обзор" в Application Insights содержатся основные сведения о вашей службе, например время отклика и число обработанных запросов. Щелкнув кнопку "Поиск" вверху, вы увидите список последних запросов в приложении. Кроме того, вы увидите здесь неудачные запросы и сможете продиагностировать возникшие ошибки.

![Обзор Application Insights](media/service-fabric-diagnostics-event-analysis-appinsights/ai-overview.png)

На предыдущем рисунке вы видите справа панель со списком элементов двух основных типов: запросы и события. Запросы поступают в виде вызовов API-интерфейса приложения, в нашем примере это HTTP-запросы. Событиями называются любые пользовательские события, которые можно добавить в любое место кода в качестве данных телеметрии. Дополнительные сведения об инструментировании приложений вы найдете в статье [API Application Insights для пользовательских событий и метрик](../azure-monitor/app/api-custom-events-metrics.md). Если щелкнуть запрос, отобразятся дополнительные сведения, как показано на следующем рисунке, в том числе данные, относящиеся к Service Fabric, которые собираются в Application Insights Service Fabric пакет NuGet. Эти данные полезны для устранения неполадок и анализа состояния приложения. Все они доступны для поиска в Application Insights.

![Снимок экрана, показывающий дополнительные сведения, включая данные, относящиеся к Service Fabric, которые собираются в Application Insights Service Fabric пакет NuGet.](media/service-fabric-diagnostics-event-analysis-appinsights/ai-request-details.png)

Application Insights имеет специальное представление для запросов по поступающим данным. Выберите "обозреватель метрик" в верхней части страницы обзора для перехода на портал Application Insights. Здесь вы можете с помощью языка запросов Kusto получить данные об описанных выше пользовательских событиях, запросах, исключениях, счетчиках производительности и других метриках. В следующем примере показаны все запросы за последний час.

![Сведения о запросах Application Insights](media/service-fabric-diagnostics-event-analysis-appinsights/ai-metrics-explorer.png)

Для дальнейшего изучения возможностей портала Application Insights перейдите на [портал документации Application Insights](../azure-monitor/app/overview-dashboard.md).

### <a name="configuring-application-insights-with-eventflow"></a>Настройка Application Insights с использованием EventFlow

Если для статистической обработки событий используется EventFlow, импортируйте пакет NuGet `Microsoft.Diagnostics.EventFlow.Outputs.ApplicationInsights`. В разделе *outputs* файла *eventFlowConfig.json* должен присутствовать следующий код:

```json
"outputs": [
    {
        "type": "ApplicationInsights",
        "instrumentationKey": "***ADD INSTRUMENTATION KEY HERE***"
    }
]
```

Обязательно внесите необходимые изменения в фильтры, а также включите все остальные входные данные (с соответствующими пакетами NuGet).

## <a name="application-insights-sdk"></a>Пакет SDK для Application Insights

Рекомендуется использовать EventFlow и WAD в качестве решений для агрегирования, поскольку они обеспечивают более модульный подход к диагностике и мониторингу, т. е. Если вы хотите изменить выходные данные из EventFlow, это не повлияет на реальное инструментирование, а лишь на простое изменение файла конфигурации. Если вы решили вкладывать данные с помощью Application Insights и, скорее всего, не сможете перейти на другую платформу, следует рассмотреть использование нового пакета SDK Application Insights "для агрегирования событий и отправки их в Application Insights. Благодаря этому вам больше не придется настраивать EventFlow для отправки данных в Application Insights. Вместо этого вы установите пакет NuGet Service Fabric для Application Insights. Подробные сведения о пакете можно найти [здесь](https://github.com/Microsoft/ApplicationInsights-ServiceFabric).

В статье [Application Insights support for Microservices and Containers](https://azure.microsoft.com/blog/app-insights-microservices/) (Поддержка микрослужб и контейнеров в Application Insights) описываются некоторые новые возможности, над которыми ведется работа (сейчас они доступны в виде бета-версии) и которые расширяют набор встроенных параметров мониторинга в Application Insights. Они включают в себя отслеживание зависимостей (используется при построении карты AppMap для всех служб и приложений в кластере и взаимосвязей между ними), а также улучшенное сопоставление трассировок, поступающих из служб (помогает выявлять проблемы в рабочем процессе приложения или службы).

Если вы ведете разработку на .NET, используете модели программирования Service Fabric и хотите применять Application Insights в качестве платформы для визуализации и анализа данных событий и журналов, мы рекомендуем использовать пакет SDK для Application Insights в рабочем процессе мониторинга и диагностики. Ознакомьтесь с документацией по [Application Insights](../azure-monitor/azure-monitor-app-hub.yml) и [журналами трассировки](../azure-monitor/app/asp-net-trace-logs.md) , чтобы приступить к работе с Application Insights для получения и просмотра журналов.

## <a name="navigating-the-application-insights-resource-in-azure-portal"></a>Навигация в ресурсе Application Insights на портале Azure

После настройки Application Insights для сбора событий и журналов сведения должны начать отображаться в ресурсе Application Insights через несколько минут. Перейдите в ресурс Application Insights. При этом откроется информационная панель ресурса Application Insights. Выберите **Поиск** на панели задач Application Insights, чтобы просмотреть последние трассировки, которые она получила, и возможность их фильтрации.

*Обозреватель метрик* — это полезное средство для создания пользовательских панелей мониторинга на основе метрик, которые могут предоставляться приложениями, службами и кластером. Сведения о настройке диаграмм на основе собираемых данных см. в статье [Исследование метрик в Application Insights](../azure-monitor/essentials/metrics-charts.md).

Если щелкнуть **Аналитика**, откроется портал аналитики Application Insights, на котором доступно больше возможностей по запросу событий и трассировок. Дополнительные сведения см. в статье [Аналитика в Application Insights](../azure-monitor/logs/log-query-overview.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Настройте оповещения в Application Insights](../azure-monitor/alerts/alerts-log.md), чтобы узнавать об изменениях в производительности или характере использования.
* [Интеллектуальное обнаружение в Application Insights](../azure-monitor/app/proactive-diagnostics.md) осуществляет упреждающий анализ данных телеметрии, отправляемых в Application Insights, и предупреждает о потенциальных проблемах с производительностью.
