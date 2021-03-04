---
title: Действия веб-перехватчика для оповещений журнала в оповещениях Azure
description: Описывает, как настроить оповещение журнала push-уведомлений с помощью действия веб-перехватчика и доступных настроек.
author: yanivlavi
ms.author: yalavi
services: monitoring
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: 228193066c45421c4dddee1802aba1feed59e9c8
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102042680"
---
# <a name="webhook-actions-for-log-alert-rules"></a>Действия веб-перехватчика для правил оповещений журнала

[Оповещение журнала](alerts-log.md) поддерживает [настройку групп действий веб-перехватчика](./action-groups.md#webhook). В этой статье мы расскажем о доступных свойствах и настройке пользовательского веб-перехватчика JSON.

> [!NOTE]
> Пользовательский веб-перехватчик на основе JSON в настоящее время не поддерживается в версии API `2020-05-01-preview`

> [!NOTE]
> Рекомендуется использовать [общую схему оповещений](../alerts/alerts-common-schema.md) для интеграции веб-перехватчика. Общая схема предупреждений предоставляет преимущества единого расширяемого и унифицированного набора полезных данных оповещений во всех службах предупреждений в Azure Monitor. Для правил предупреждений журнала, для которых определены пользовательские полезные данные JSON, при включении общей схемы будет возвращена схема полезных данных, описанная [здесь](../alerts/alerts-common-schema-definitions.md#log-alerts). Для оповещений с включенной общей схемой установлен максимальный размер, равный 256 КБ на оповещение, а более крупное предупреждение не будет содержать результаты поиска. Если результаты поиска не включаются, следует использовать `LinkToFilteredSearchResultsAPI` или `LinkToSearchResultsAPI` для доступа к результатам запроса через API log Analytics.

## <a name="webhook-payload-properties"></a>Свойства полезных данных веб-перехватчика

Действия веб-перехватчика позволяют вызывать один запрос HTTP POST. Вызываемая служба должна поддерживать веб-перехватчики и уметь использовать полученные полезные данные.

Свойства действия веб-перехватчика по умолчанию и их пользовательские имена параметров JSON:

| Параметр | Переменная | Описание |
|:--- |:--- |:--- |
| *алертруленаме* |#alertrulename |Имя правила генерации оповещений. |
| *Уровень серьезности* |#severity |Уровень серьезности, установленный для срабатывающего оповещения журнала. |
| *AlertThresholdOperator* |#thresholdoperator |Оператор порога для правила генерации оповещений. |
| *AlertThresholdValue* |#thresholdvalue |Значение порога для правила генерации оповещений. |
| *линктосеарчресултс* |#linktosearchresults |Ссылка на портал аналитики, который возвращает записи из запроса, создавшего предупреждение. |
| *линктосеарчресултсапи* |#linktosearchresultsapi |Ссылка на API аналитики, который возвращает записи из запроса, создавшего предупреждение. |
| *линктофилтередсеарчресултсуи* |#linktofilteredsearchresultsui |Ссылка на портал аналитики, который возвращает записи из запроса, отфильтрованного по значениям измерений, создавшего предупреждение. |
| *линктофилтередсеарчресултсапи* |#linktofilteredsearchresultsapi |Ссылка на API аналитики, который возвращает записи из запроса, отфильтрованного по значениям измерений, создавшего предупреждение. |
| *ResultCount* |#searchresultcount |Число записей в результатах поиска. |
| *Время окончания интервала поиска* |#searchintervalendtimeutc |Время окончания запроса в формате UTC с форматом mm/дд/гггг чч: мм: сс AM/PM. |
| *Интервал поиска* |#searchinterval |Временное окно для правила генерации оповещений с форматом чч: мм: СС. |
| *Время начала интервала поиска* |#searchintervalstarttimeutc |Время начала запроса в формате UTC с форматом mm/дд/гггг чч: мм: сс AM/PM. 
| *SearchQuery* |#searchquery |Запрос поиска журналов, используемый правилом генерации оповещений. |
| *SearchResults* |"IncludeSearchResults": true|Записи, возвращаемые запросом в виде таблицы JSON, ограничены первыми записями 1 000. "IncludeSearchResults": значение true добавляется в определение пользовательского веб-перехватчика JSON в качестве свойства верхнего уровня. |
| *Измерения* |"Инклудедименсионс": true|Сочетания значений измерений, которые активируют предупреждение как раздел JSON. "Инклудедименсионс": значение true добавляется в определение пользовательского веб-перехватчика JSON в качестве свойства верхнего уровня. |
| *Тип оповещения*| #alerttype | Тип правила оповещения журнала, настроенного как [измерение метрик или количество результатов](./alerts-unified-log.md#measure).|
| *WorkspaceID* |#workspaceid |Идентификатор рабочей области Log Analytics. |
| *Идентификатор приложения* |#applicationid |Идентификатор приложения Application Insights. |
| *Идентификатор подписки* |#subscriptionid |ИДЕНТИФИКАТОР используемой подписки Azure. |

## <a name="custom-webhook-payload-definition"></a>Определение пользовательских полезных данных веб-перехватчика

Вы можете использовать **полезные данные ПОЛЬЗОВАТЕЛЬСКОГО JSON для веб-перехватчика** , чтобы получить пользовательские полезные данные JSON, используя приведенные выше параметры. Также можно создать дополнительные свойства.
Например, можно указать следующие пользовательские полезные данные, содержащие один параметр — *text*. Служба, которую вызывает этот веб-перехватчик, принимает этот параметр:

```json

    {
        "text":"#alertrulename fired with #searchresultcount over threshold of #thresholdvalue."
    }
```
В этом примере полезная нагрузка разрешается примерно следующим образом при отправке в веб-перехватчик:

```json
    {
        "text":"My Alert Rule fired with 18 records over threshold of 10 ."
    }
```
Переменные в пользовательском веб-перехватчике должны быть указаны в корпусе JSON. Например, ссылка на "#searchresultcount" в приведенном выше примере веб-перехватчика будет выведена на основе результатов оповещения.

Чтобы включить результаты поиска, добавьте **IncludeSearchResults** в качестве свойства верхнего уровня в пользовательском JSON. Результаты поиска включаются в виде структуры JSON, поэтому на результаты нельзя ссылаться в пользовательских определенных полях. 

> [!NOTE]
> Кнопка **Просмотр веб-перехватчика** рядом с параметром **включить пользовательские полезные данные JSON для веб-перехватчика** отображает предварительную версию предоставленных данных. Он не содержит фактических данных, но является представлением схемы JSON, которая будет использоваться. 

## <a name="sample-payloads"></a>Примеры полезных данных
В этом разделе показаны примеры полезных данных для веб-перехватчиков для оповещений журнала. Примерами полезных данных являются примеры, когда полезные данные являются стандартными и когда они являются пользовательскими.

### <a name="log-alert-for-log-analytics"></a>Оповещение журнала для Log Analytics
Ниже приведен пример полезных данных для стандартного действия веб-перехватчика, используемого для оповещений на основе Log Analytics:

> [!NOTE]
> Значение поля "серьезность" изменяется, если вы [перешли на текущий API счедуледкуерирулес](../alerts/alerts-log-api-switch.md) из [устаревшего api предупреждений log Analytics](./api-alerts.md).

```json
{
    "SubscriptionId": "12345a-1234b-123c-123d-12345678e",
    "AlertRuleName": "AcmeRule",
    "SearchQuery": "Perf | where ObjectName == \"Processor\" and CounterName == \"% Processor Time\" | summarize AggregatedValue = avg(CounterValue) by bin(TimeGenerated, 5m), Computer",
    "SearchIntervalStartTimeUtc": "2018-03-26T08:10:40Z",
    "SearchIntervalEndtimeUtc": "2018-03-26T09:10:40Z",
    "AlertThresholdOperator": "Greater Than",
    "AlertThresholdValue": 0,
    "ResultCount": 2,
    "SearchIntervalInSeconds": 3600,
    "LinkToSearchResults": "https://portal.azure.com/#Analyticsblade/search/index?_timeInterval.intervalEnd=2018-03-26T09%3a10%3a40.0000000Z&_timeInterval.intervalDuration=3600&q=Usage",
    "LinkToFilteredSearchResultsUI": "https://portal.azure.com/#Analyticsblade/search/index?_timeInterval.intervalEnd=2018-03-26T09%3a10%3a40.0000000Z&_timeInterval.intervalDuration=3600&q=Usage",
    "LinkToSearchResultsAPI": "https://api.loganalytics.io/v1/workspaces/workspaceID/query?query=Heartbeat&timespan=2020-05-07T18%3a11%3a51.0000000Z%2f2020-05-07T18%3a16%3a51.0000000Z",
    "LinkToFilteredSearchResultsAPI": "https://api.loganalytics.io/v1/workspaces/workspaceID/query?query=Heartbeat&timespan=2020-05-07T18%3a11%3a51.0000000Z%2f2020-05-07T18%3a16%3a51.0000000Z",
    "Description": "log alert rule",
    "Severity": "Warning",
    "AffectedConfigurationItems": [
        "INC-Gen2Alert"
    ],
    "Dimensions": [
        {
            "name": "Computer",
            "value": "INC-Gen2Alert"
        }
    ],
    "SearchResult": {
        "tables": [
            {
                "name": "PrimaryResult",
                "columns": [
                    {
                        "name": "$table",
                        "type": "string"
                    },
                    {
                        "name": "Computer",
                        "type": "string"
                    },
                    {
                        "name": "TimeGenerated",
                        "type": "datetime"
                    }
                ],
                "rows": [
                    [
                        "Fabrikam",
                        "33446677a",
                        "2018-02-02T15:03:12.18Z"
                    ],
                    [
                        "Contoso",
                        "33445566b",
                        "2018-02-02T15:16:53.932Z"
                    ]
                ]
            }
        ]
    },
    "WorkspaceId": "12345a-1234b-123c-123d-12345678e",
    "AlertType": "Metric measurement"
}
```

### <a name="log-alert-for-application-insights"></a>Оповещение журнала для Application Insights
Ниже приведен пример полезных данных для стандартного веб-перехватчика, который используется для оповещений журнала на основе Application Insightsных ресурсов:
    
```json
{
    "schemaId": "Microsoft.Insights/LogAlert",
    "data": {
        "SubscriptionId": "12345a-1234b-123c-123d-12345678e",
        "AlertRuleName": "AcmeRule",
        "SearchQuery": "requests | where resultCode == \"500\" | summarize AggregatedValue = Count by bin(Timestamp, 5m), IP",
        "SearchIntervalStartTimeUtc": "2018-03-26T08:10:40Z",
        "SearchIntervalEndtimeUtc": "2018-03-26T09:10:40Z",
        "AlertThresholdOperator": "Greater Than",
        "AlertThresholdValue": 0,
        "ResultCount": 2,
        "SearchIntervalInSeconds": 3600,
        "LinkToSearchResults": "https://portal.azure.com/AnalyticsBlade/subscriptions/12345a-1234b-123c-123d-12345678e/?query=search+*+&timeInterval.intervalEnd=2018-03-26T09%3a10%3a40.0000000Z&_timeInterval.intervalDuration=3600&q=Usage",
        "LinkToFilteredSearchResultsUI": "https://portal.azure.com/AnalyticsBlade/subscriptions/12345a-1234b-123c-123d-12345678e/?query=search+*+&timeInterval.intervalEnd=2018-03-26T09%3a10%3a40.0000000Z&_timeInterval.intervalDuration=3600&q=Usage",
        "LinkToSearchResultsAPI": "https://api.applicationinsights.io/v1/apps/0MyAppId0/metrics/requests/count",
        "LinkToFilteredSearchResultsAPI": "https://api.applicationinsights.io/v1/apps/0MyAppId0/metrics/requests/count",
        "Description": null,
        "Severity": "3",
        "Dimensions": [
            {
                "name": "IP",
                "value": "1.1.1.1"
            }
        ],
        "SearchResult": {
            "tables": [
                {
                    "name": "PrimaryResult",
                    "columns": [
                        {
                            "name": "$table",
                            "type": "string"
                        },
                        {
                            "name": "Id",
                            "type": "string"
                        },
                        {
                            "name": "Timestamp",
                            "type": "datetime"
                        }
                    ],
                    "rows": [
                        [
                            "Fabrikam",
                            "33446677a",
                            "2018-02-02T15:03:12.18Z"
                        ],
                        [
                            "Contoso",
                            "33445566b",
                            "2018-02-02T15:16:53.932Z"
                        ]
                    ]
                }
            ]
        },
        "ApplicationId": "123123f0-01d3-12ab-123f-abc1ab01c0a1",
        "AlertType": "Metric measurement"
    }
}
```

### <a name="log-alert-for-other-resources-logs-from-api-version-2020-05-01-preview"></a>Оповещения журнала для других журналов ресурсов (из версии API `2020-05-01-preview` )

> [!NOTE]
> В настоящее время дополнительная плата за использование версий API `2020-05-01-preview` и оповещений журнала не взимается.  Цены на функции, доступные в предварительной версии, будут объявлены в будущем и появится уведомление перед началом выставления счетов. Если вы решили продолжить использование новой версии API и оповещений журнала на основе ресурсов по истечении периода уведомления, плата будет взиматься по применимой ставке.

Ниже приведен пример полезных данных для стандартного веб-перехватчика, который используется для оповещений журнала на основе других журналов ресурсов (за исключением рабочих областей и Application Insights):

```json
{
    "schemaId": "azureMonitorCommonAlertSchema",
    "data": {
        "essentials": {
            "alertId": "/subscriptions/12345a-1234b-123c-123d-12345678e/providers/Microsoft.AlertsManagement/alerts/12345a-1234b-123c-123d-12345678e",
            "alertRule": "AcmeRule",
            "severity": "Sev4",
            "signalType": "Log",
            "monitorCondition": "Fired",
            "monitoringService": "Log Alerts V2",
            "alertTargetIDs": [
                "/subscriptions/12345a-1234b-123c-123d-12345678e/resourcegroups/ai-engineering/providers/microsoft.compute/virtualmachines/testvm"
            ],
            "originAlertId": "123c123d-1a23-1bf3-ba1d-dd1234ff5a67",
            "firedDateTime": "2020-07-09T14:04:49.99645Z",
            "description": "log alert rule V2",
            "essentialsVersion": "1.0",
            "alertContextVersion": "1.0"
        },
        "alertContext": {
            "properties": null,
            "conditionType": "LogQueryCriteria",
            "condition": {
                "windowSize": "PT10M",
                "allOf": [
                    {
                        "searchQuery": "Heartbeat",
                        "metricMeasure": null,
                        "targetResourceTypes": "['Microsoft.Compute/virtualMachines']",
                        "operator": "LowerThan",
                        "threshold": "1",
                        "timeAggregation": "Count",
                        "dimensions": [
                            {
                                "name": "ResourceId",
                                "value": "/subscriptions/12345a-1234b-123c-123d-12345678e/resourceGroups/TEST/providers/Microsoft.Compute/virtualMachines/testvm"
                            }
                        ],
                        "metricValue": 0.0,
                        "failingPeriods": {
                            "numberOfEvaluationPeriods": 1,
                            "minFailingPeriodsToAlert": 1
                        },
                        "linkToSearchResultsUI": "https://portal.azure.com#@12f345bf-12f3-12af-12ab-1d2cd345db67/blade/Microsoft_Azure_Monitoring_Logs/LogsBlade/source/Alerts.EmailLinks/scope/%7B%22resources%22%3A%5B%7B%22resourceId%22%3A%22%2Fsubscriptions%2F12345a-1234b-123c-123d-12345678e%2FresourceGroups%2FTEST%2Fproviders%2FMicrosoft.Compute%2FvirtualMachines%2Ftestvm%22%7D%5D%7D/q/eJzzSE0sKklKTSypUSjPSC1KVQjJzE11T81LLUosSU1RSEotKU9NzdNIAfJKgDIaRgZGBroG5roGliGGxlYmJlbGJnoGEKCpp4dDmSmKMk0A/prettify/1/timespan/2020-07-07T13%3a54%3a34.0000000Z%2f2020-07-09T13%3a54%3a34.0000000Z",
                        "linkToFilteredSearchResultsUI": "https://portal.azure.com#@12f345bf-12f3-12af-12ab-1d2cd345db67/blade/Microsoft_Azure_Monitoring_Logs/LogsBlade/source/Alerts.EmailLinks/scope/%7B%22resources%22%3A%5B%7B%22resourceId%22%3A%22%2Fsubscriptions%2F12345a-1234b-123c-123d-12345678e%2FresourceGroups%2FTEST%2Fproviders%2FMicrosoft.Compute%2FvirtualMachines%2Ftestvm%22%7D%5D%7D/q/eJzzSE0sKklKTSypUSjPSC1KVQjJzE11T81LLUosSU1RSEotKU9NzdNIAfJKgDIaRgZGBroG5roGliGGxlYmJlbGJnoGEKCpp4dDmSmKMk0A/prettify/1/timespan/2020-07-07T13%3a54%3a34.0000000Z%2f2020-07-09T13%3a54%3a34.0000000Z",
                        "linkToSearchResultsAPI": "https://api.loganalytics.io/v1/subscriptions/12345a-1234b-123c-123d-12345678e/resourceGroups/TEST/providers/Microsoft.Compute/virtualMachines/testvm/query?query=Heartbeat%7C%20where%20TimeGenerated%20between%28datetime%282020-07-09T13%3A44%3A34.0000000%29..datetime%282020-07-09T13%3A54%3A34.0000000%29%29&timespan=2020-07-07T13%3a54%3a34.0000000Z%2f2020-07-09T13%3a54%3a34.0000000Z",
                        "linkToFilteredSearchResultsAPI": "https://api.loganalytics.io/v1/subscriptions/12345a-1234b-123c-123d-12345678e/resourceGroups/TEST/providers/Microsoft.Compute/virtualMachines/testvm/query?query=Heartbeat%7C%20where%20TimeGenerated%20between%28datetime%282020-07-09T13%3A44%3A34.0000000%29..datetime%282020-07-09T13%3A54%3A34.0000000%29%29&timespan=2020-07-07T13%3a54%3a34.0000000Z%2f2020-07-09T13%3a54%3a34.0000000Z"
                    }
                ],
                "windowStartTime": "2020-07-07T13:54:34Z",
                "windowEndTime": "2020-07-09T13:54:34Z"
            }
        }
    }
}
```

### <a name="log-alert-with-a-custom-json-payload"></a>Оповещение журнала с пользовательскими полезными данными JSON
Например, чтобы создать пользовательские полезные данные, включающие только имя предупреждения и результаты поиска, используйте следующую конфигурацию: 

```json
    {
       "alertname":"#alertrulename",
       "IncludeSearchResults":true
    }
```

Ниже приведен пример полезных данных для настраиваемого действия веб-перехватчика для любого оповещения журнала.
    
```json
    {
    "alertname":"AcmeRule","IncludeSearchResults":true,
    "SearchResults":
        {
        "tables":[
                    {"name":"PrimaryResult","columns":
                        [
                        {"name":"$table","type":"string"},
                        {"name":"Id","type":"string"},
                        {"name":"TimeGenerated","type":"datetime"}
                        ],
                    "rows":
                        [
                            ["Fabrikam","33446677a","2018-02-02T15:03:12.18Z"],
                            ["Contoso","33445566b","2018-02-02T15:16:53.932Z"]
                        ]
                    }
                ]
        }
    }
```

## <a name="next-steps"></a>Дальнейшие действия
- Сведения об [оповещениях журнала в оповещениях Azure](./alerts-unified-log.md).
- Узнайте, как [управлять оповещениями журнала в Azure](alerts-log.md).
- Создание [групп действий в Azure](./action-groups.md)и управление ими.
- Дополнительные сведения об [Application Insights](../logs/log-query-overview.md).
- Дополнительные сведения о [запросах журналов](../logs/log-query-overview.md).