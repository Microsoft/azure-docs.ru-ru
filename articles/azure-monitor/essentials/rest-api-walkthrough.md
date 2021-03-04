---
title: Пошаговое руководство по REST API Azure Monitor
description: Эта статья содержит сведения об аутентификации запросов и использовании REST API Azure Monitor для получения доступных определений и значений метрик.
ms.topic: conceptual
ms.date: 03/19/2018
ms.custom: has-adal-ref
ms.openlocfilehash: a7cd6ff7c0c3b5d4bee859ef288f16673ebe0835
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102033084"
---
# <a name="azure-monitoring-rest-api-walkthrough"></a>Пошаговое руководство по REST API Azure Monitor

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

В этой статье показано, как выполнять аутентификацию таким образом, чтобы в коде можно было использовать [справочник по REST API Microsoft Azure Monitor](/rest/api/monitor/).

API Azure Monitor позволяет программно получить определения доступных метрик по умолчанию, детализацию и значения метрик. Данные можно сохранить в отдельном хранилище данных, в том числе в базе данных SQL Azure, Azure Cosmos DB или Azure Data Lake. Там при необходимости можно выполнить дополнительный анализ.

Помимо работы с различными точками данных метрик API Monitor позволяет также получить список правил генерации оповещений, просмотреть журналы действий и многое другое. Полный список доступных операций см. в [справочнике по REST API для Microsoft Azure Monitor](/rest/api/monitor/).

## <a name="authenticating-azure-monitor-requests"></a>Аутентификация запросов Azure Monitor

В первую очередь нужно аутентифицировать запрос.

Все задачи, выполняемые с использованием API Azure Monitor, используют модель аутентификации Azure Resource Manager. Поэтому все запросы должны быть аутентифицированы в Azure Active Directory (Azure AD). Один из способов аутентификации клиентского приложения — создание субъекта-службы Azure AD и получение маркера аутентификации (JWT). Приведенный ниже пример сценария демонстрирует создание субъекта-службы Azure AD посредством PowerShell. Более подробные пошаговые инструкции приведены в документации по [использованию Azure PowerShell для создания субъекта-службы для доступа к ресурсам](/powershell/azure/create-azure-service-principal-azureps). Кроме того, [субъект-службу можно создать на портале Azure](../../active-directory/develop/howto-create-service-principal-portal.md).

```powershell
$subscriptionId = "{azure-subscription-id}"
$resourceGroupName = "{resource-group-name}"

# Authenticate to a specific Azure subscription.
Connect-AzAccount -SubscriptionId $subscriptionId

# Password for the service principal
$pwd = "{service-principal-password}"
$secureStringPassword = ConvertTo-SecureString -String $pwd -AsPlainText -Force

# Create a new Azure AD application
$azureAdApplication = New-AzADApplication `
                        -DisplayName "My Azure Monitor" `
                        -HomePage "https://localhost/azure-monitor" `
                        -IdentifierUris "https://localhost/azure-monitor" `
                        -Password $secureStringPassword

# Create a new service principal associated with the designated application
New-AzADServicePrincipal -ApplicationId $azureAdApplication.ApplicationId

# Assign Reader role to the newly created service principal
New-AzRoleAssignment -RoleDefinitionName Reader `
                          -ServicePrincipalName $azureAdApplication.ApplicationId.Guid

```

Чтобы отправить запрос к API Azure Monitor, клиентское приложение должно использовать ранее созданный субъект-службу для прохождения аутентификации. В приведенном ниже примере сценария PowerShell показан один подход, в котором для получения токена аутентификации JWT используется [библиотека аутентификации Active Directory](../../active-directory/azuread-dev/active-directory-authentication-libraries.md) (ADAL). Этот маркер JWT передается как часть параметра проверки подлинности HTTP в запросах к API Azure Monitor.

```powershell
$azureAdApplication = Get-AzADApplication -IdentifierUri "https://localhost/azure-monitor"

$subscription = Get-AzSubscription -SubscriptionId $subscriptionId

$clientId = $azureAdApplication.ApplicationId.Guid
$tenantId = $subscription.TenantId
$authUrl = "https://login.microsoftonline.com/${tenantId}"

$AuthContext = [Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext]$authUrl
$cred = New-Object -TypeName Microsoft.IdentityModel.Clients.ActiveDirectory.ClientCredential -ArgumentList ($clientId, $pwd)

$result = $AuthContext.AcquireTokenAsync("https://management.core.windows.net/", $cred).GetAwaiter().GetResult()

# Build an array of HTTP header values
$authHeader = @{
'Content-Type'='application/json'
'Accept'='application/json'
'Authorization'=$result.CreateAuthorizationHeader()
}
```

После аутентификации можно будет выполнять запросы к REST API Azure Monitor. Можно использовать два полезных запроса:

1. вывод списка определений метрик для ресурса;
2. получение значений метрик.

> [!NOTE]
> Дополнительные сведения о проверке подлинности в Azure REST API см. в [справочнике по azure REST API](/rest/api/azure/).
>
>

## <a name="retrieve-metric-definitions-multi-dimensional-api"></a>Получение определений метрик (многомерные API)

[REST API определений метрик Azure Monitor](/rest/api/monitor/metricdefinitions). Используется для доступа к списку метрик, которые доступны для службы.

**Метод**: GET

**URI запроса**: https: \/ \/ Management.Azure.com/Subscriptions/*{SubscriptionId}*/resourceGroups/*{resourceGroupName}*/providers/*{resourceProviderNamespace}* / *{ResourceType}* / *{resourceName}*/providers/Microsoft.Insights/metricDefinitions? API-Version =*{apiVersion}*

Например, запрос на получение определений метрик для учетной записи хранения Azure будет выглядеть следующим образом:

```powershell
$request = "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage/providers/microsoft.insights/metricDefinitions?api-version=2018-01-01"

Invoke-RestMethod -Uri $request `
                  -Headers $authHeader `
                  -Method Get `
                  -OutFile ".\contosostorage-metricdef-results.json" `
                  -Verbose

```

> [!NOTE]
> Чтобы получить определения метрик с помощью многомерных метрик REST API Azure Monitor, используйте в качестве версии API значение "2018-01-01".
>
>

Полученный текст ответа JSON будет аналогичен приведенному ниже примеру. (Обратите внимание, что вторая метрика имеет измерения.)

```json
{
    "value": [
        {
            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage/providers/microsoft.insights/metricdefinitions/UsedCapacity",
            "resourceId": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage",
            "namespace": "Microsoft.Storage/storageAccounts",
            "category": "Capacity",
            "name": {
                "value": "UsedCapacity",
                "localizedValue": "Used capacity"
            },
            "isDimensionRequired": false,
            "unit": "Bytes",
            "primaryAggregationType": "Average",
            "supportedAggregationTypes": [
                "Total",
                "Average",
                "Minimum",
                "Maximum"
            ],
            "metricAvailabilities": [
                {
                    "timeGrain": "PT1H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT6H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT12H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "P1D",
                    "retention": "P93D"
                }
            ]
        },
        {
            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage/providers/microsoft.insights/metricdefinitions/Transactions",
            "resourceId": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage",
            "namespace": "Microsoft.Storage/storageAccounts",
            "category": "Transaction",
            "name": {
                "value": "Transactions",
                "localizedValue": "Transactions"
            },
            "isDimensionRequired": false,
            "unit": "Count",
            "primaryAggregationType": "Total",
            "supportedAggregationTypes": [
                "Total"
            ],
            "metricAvailabilities": [
                {
                    "timeGrain": "PT1M",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT5M",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT15M",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT30M",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT1H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT6H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT12H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "P1D",
                    "retention": "P93D"
                }
            ],
            "dimensions": [
                {
                    "value": "ResponseType",
                    "localizedValue": "Response type"
                },
                {
                    "value": "GeoType",
                    "localizedValue": "Geo type"
                },
                {
                    "value": "ApiName",
                    "localizedValue": "API name"
                }
            ]
        },
        ...
    ]
}
```

## <a name="retrieve-dimension-values-multi-dimensional-api"></a>Получение значений измерений (многомерные API)

Когда определения доступных метрик известны, могут возникнуть некоторые метрики c измерениями. Перед выполнением запроса для метрик может потребоваться выяснить, какие диапазоны значений имеют измерения. На основе этих значений измерений вы можете выбрать, следует ли фильтровать или сегментировать метрики, на основе значений измерений при запросе метрик.  Это достигается с помощью [REST API метрик Azure Monitor](/rest/api/monitor/metrics).

Используйте для имени метрики в запросах фильтрации для значение value (а не localizedValue) . Если не указано ни одного фильтра, возвращается метрика по умолчанию. При использовании этого API фильтр с подстановочными знаками разрешается только в одном измерении.

> [!NOTE]
> Чтобы получить значения измерений с помощью REST API Azure Monitor, используйте в качестве версии API значение "2018-01-01".
>
>

**Метод**: GET

**URI запроса**: HTTPS \: //Management.Azure.com/Subscriptions/*{Subscription-ID}*/resourceGroups/*{ресурс-Group-Name}*/providers/*{ресурс-поставщик-пространство имен}* / *{ресурс-тип}* / *{Resource-Name}*/providers/Microsoft.Insights/Metrics? metricnames =*{Метрика}*&TimeSpan =*{StartTime/EndTime}*&$Filter =*{Filter}*&resultType = метаданные&API-Version =*{apiVersion}*

Например, чтобы получить список значений измерений, которые были созданы для параметра "API Name dimension" (Измерение имени API) метрики "Транзакции", где измерение GeoType = "Primary" в течение указанного интервала времени, используйте следующий запрос:

```powershell
$filter = "APIName eq '*' and GeoType eq 'Primary'"
$request = "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage/providers/microsoft.insights/metrics?metricnames=Transactions&timespan=2018-03-01T00:00:00Z/2018-03-02T00:00:00Z&resultType=metadata&`$filter=${filter}&api-version=2018-01-01"
Invoke-RestMethod -Uri $request `
    -Headers $authHeader `
    -Method Get `
    -OutFile ".\contosostorage-dimension-values.json" `
    -Verbose
```

Полученный текст ответа JSON будет аналогичен приведенному ниже примеру.

```json
{
  "timespan": "2018-03-01T00:00:00Z/2018-03-02T00:00:00Z",
  "value": [
    {
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage/providers/Microsoft.Insights/metrics/Transactions",
      "type": "Microsoft.Insights/metrics",
      "name": {
        "value": "Transactions",
        "localizedValue": "Transactions"
      },
      "unit": "Count",
      "timeseries": [
        {
          "metadatavalues": [
            {
              "name": {
                "value": "apiname",
                "localizedValue": "apiname"
              },
              "value": "DeleteBlob"
            }
          ]
        },
        {
          "metadatavalues": [
            {
              "name": {
                "value": "apiname",
                "localizedValue": "apiname"
              },
              "value": "SetBlobProperties"
            }
          ]
        },
        ...
      ]
    }
  ],
  "namespace": "Microsoft.Storage/storageAccounts",
  "resourceregion": "eastus"
}
```

## <a name="retrieve-metric-values-multi-dimensional-api"></a>Получение значений метрик (многомерные API)

Когда определения доступных метрик и возможных измерений известны, можно получить соответствующие значения метрик.  Это достигается с помощью [REST API метрик Azure Monitor](/rest/api/monitor/metrics).

Используйте значение value имени метрики (а не localizedValue) для запросов фильтрации. Если фильтры измерений не указаны, возвращается агрегированная метрика со сведением. Если запрос метрики возвращает несколько временных рядов, с помощью параметров запроса Top и OrderBy можно обеспечить возврат только ограниченного упорядоченного списка временных рядов.

> [!NOTE]
> Чтобы получить значения многомерных метрик с помощью REST API Azure Monitor, используйте в качестве версии API значение "2018-01-01".
>
>

**Метод**: GET

**URI запроса**: https: \/ /Management.Azure.com/Subscriptions/*{Subscription-ID}*/resourceGroups/*{ресурс-Group-Name}*/providers/*{ресурс-поставщик-пространство имен}* / *{Resource-Type}* / *{Resource-Name}*/providers/Microsoft.Insights/Metrics? metricnames =*{Метрика}*&TimeSpan =*{StartTime/EndTime}*&$Filter =*{Filter}*&интервал =*{timeGrain}*&агрегат =*{агрегирование}*&API-Version =*{apiVersion}*

Например, чтобы вывести первые 3 API-интерфейса в убывающем порядке по величине показателя "Транзакции" в течение 5-минутного диапазона, где для GeoType установлено значение "Primary", используйте следующий запрос:

```powershell
$filter = "APIName eq '*' and GeoType eq 'Primary'"
$request = "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage/providers/microsoft.insights/metrics?metricnames=Transactions&timespan=2018-03-01T02:00:00Z/2018-03-01T02:05:00Z&`$filter=${filter}&interval=PT1M&aggregation=Total&top=3&orderby=Total desc&api-version=2018-01-01"
Invoke-RestMethod -Uri $request `
    -Headers $authHeader `
    -Method Get `
    -OutFile ".\contosostorage-metric-values.json" `
    -Verbose
```

Полученный текст ответа JSON будет аналогичен приведенному ниже примеру.

```json
{
  "cost": 0,
  "timespan": "2018-03-01T02:00:00Z/2018-03-01T02:05:00Z",
  "interval": "PT1M",
  "value": [
    {
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/ContosoStorage/providers/Microsoft.Insights/metrics/Transactions",
      "type": "Microsoft.Insights/metrics",
      "name": {
        "value": "Transactions",
        "localizedValue": "Transactions"
      },
      "unit": "Count",
      "timeseries": [
        {
          "metadatavalues": [
            {
              "name": {
                "value": "apiname",
                "localizedValue": "apiname"
              },
              "value": "GetBlobProperties"
            }
          ],
          "data": [
            {
              "timeStamp": "2017-09-19T02:00:00Z",
              "total": 2
            },
            {
              "timeStamp": "2017-09-19T02:01:00Z",
              "total": 1
            },
            {
              "timeStamp": "2017-09-19T02:02:00Z",
              "total": 3
            },
            {
              "timeStamp": "2017-09-19T02:03:00Z",
              "total": 7
            },
            {
              "timeStamp": "2017-09-19T02:04:00Z",
              "total": 2
            }
          ]
        },
        ...
      ]
    }
  ],
  "namespace": "Microsoft.Storage/storageAccounts",
  "resourceregion": "eastus"
}
```

## <a name="retrieve-metric-definitions"></a>Получение определений метрик

[REST API определений метрик Azure Monitor](/rest/api/monitor/metricdefinitions). Используется для доступа к списку метрик, которые доступны для службы.

**Метод**: GET

**URI запроса**: https: \/ \/ Management.Azure.com/Subscriptions/*{SubscriptionId}*/resourceGroups/*{resourceGroupName}*/providers/*{resourceProviderNamespace}* / *{ResourceType}* / *{resourceName}*/providers/Microsoft.Insights/metricDefinitions? API-Version =*{apiVersion}*

Например, чтобы получить определения метрик для приложения логики Azure, запрос будет выглядеть следующим образом:

```powershell
$request = "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/microsoft.insights/metricDefinitions?api-version=2016-03-01"

Invoke-RestMethod -Uri $request `
                  -Headers $authHeader `
                  -Method Get `
                  -OutFile ".\contosotweets-metricdef-results.json" `
                  -Verbose
```

> [!NOTE]
> Чтобы получить определения метрик с помощью REST API Azure Monitor, используйте API версии 2016-03-01.
>
>

Полученный текст ответа JSON будет аналогичен приведенному ниже примеру.

```json
{
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/microsoft.insights/metricdefinitions",
  "value": [
    {
      "name": {
        "value": "RunsStarted",
        "localizedValue": "Runs Started"
      },
      "category": "AllMetrics",
      "startTime": "0001-01-01T00:00:00Z",
      "endTime": "0001-01-01T00:00:00Z",
      "unit": "Count",
      "primaryAggregationType": "Total",
      "resourceUri": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets",
      "resourceId": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets",
      "metricAvailabilities": [
        {
          "timeGrain": "PT1M",
          "retention": "P30D",
          "location": null,
          "blobLocation": null
        },
        {
          "timeGrain": "PT1H",
          "retention": "P30D",
          "location": null,
          "blobLocation": null
        }
      ],
      "properties": null,
      "dimensions": null,
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/microsoft.insights/metricdefinitions/RunsStarted",
      "supportedAggregationTypes": [ "None", "Average", "Minimum", "Maximum", "Total", "Count" ]
    }
  ]
}
```

Дополнительные сведения см. в статье [List metric definitions for a resource in Azure Monitor REST API](/rest/api/monitor/metricdefinitions) (Вывод списка определений метрик для ресурса в REST API Azure Monitor).

## <a name="retrieve-metric-values"></a>Получение значений метрик

Когда определения доступных метрик известны, можно получить соответствующие значения метрик. Используйте значение value имени метрики (а не localizedValue) для фильтрации запросов (например, получите точки данных метрик CpuTime и Requests). Если не указано ни одного фильтра, возвращается метрика по умолчанию.

> [!NOTE]
> Чтобы получить значения метрики с помощью REST API Azure Monitor, используйте API версии 2016-09-01.
>
>

**Метод**: `GET`

**URI запроса**: `https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/{resource-provider-namespace}/{resource-type}/{resource-name}/providers/microsoft.insights/metrics?$filter={filter}&api-version={apiVersion}`

Например, запрос для получения точек данных метрики RunsSucceeded в заданном диапазоне времени с интервалом в 1 час будет выглядеть следующим образом.

```powershell
$filter = "(name.value eq 'RunsSucceeded') and aggregationType eq 'Total' and startTime eq 2017-08-18T19:00:00 and endTime eq 2017-08-18T23:00:00 and timeGrain eq duration'PT1H'"
$request = "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/microsoft.insights/metrics?`$filter=${filter}&api-version=2016-09-01"
Invoke-RestMethod -Uri $request `
    -Headers $authHeader `
    -Method Get `
    -OutFile ".\contosotweets-metrics-results.json" `
    -Verbose
```

Полученный текст ответа JSON будет аналогичен приведенному ниже примеру.

```json
{
  "value": [
    {
      "data": [
        {
          "timeStamp": "2017-08-18T19:00:00Z",
          "total": 0.0
        },
        {
          "timeStamp": "2017-08-18T20:00:00Z",
          "total": 159.0
        },
        {
          "timeStamp": "2017-08-18T21:00:00Z",
          "total": 174.0
        },
        {
          "timeStamp": "2017-08-18T22:00:00Z",
          "total": 97.0
        }
      ],
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/Microsoft.Insights/metrics/RunsSucceeded",
      "name": {
        "value": "RunsSucceeded",
        "localizedValue": "Runs Succeeded"
      },
      "type": "Microsoft.Insights/metrics",
      "unit": "Count"
    }
  ]
}
```

Чтобы получить несколько точек данных или точек статистической обработки, добавьте имена определений метрик и типы статистической обработки в фильтр, как показано в примере ниже.

```powershell
$filter = "(name.value eq 'ActionsCompleted' or name.value eq 'RunsSucceeded') and (aggregationType eq 'Total' or aggregationType eq 'Average') and startTime eq 2017-08-18T21:00:00 and endTime eq 2017-08-18T21:30:00 and timeGrain eq duration'PT1M'"
$request = "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/microsoft.insights/metrics?`$filter=${filter}&api-version=2016-09-01"
Invoke-RestMethod -Uri $request `
    -Headers $authHeader `
    -Method Get `
    -OutFile ".\contosotweets-metrics-multiple-results.json" `
    -Verbose
```

Полученный текст ответа JSON будет аналогичен приведенному ниже примеру.

```json
{
  "value": [
    {
      "data": [
        {
          "timeStamp": "2017-08-18T21:03:00Z",
          "total": 5.0,
          "average": 1.0
        },
        {
          "timeStamp": "2017-08-18T21:04:00Z",
          "total": 7.0,
          "average": 1.0
        }
      ],
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/Microsoft.Insights/metrics/ActionsCompleted",
      "name": {
        "value": "ActionsCompleted",
        "localizedValue": "Actions Completed "
      },
      "type": "Microsoft.Insights/metrics",
      "unit": "Count"
    },
    {
      "data": [
        {
          "timeStamp": "2017-08-18T21:03:00Z",
          "total": 5.0,
          "average": 1.0
        },
        {
          "timeStamp": "2017-08-18T21:04:00Z",
          "total": 7.0,
          "average": 1.0
        }
      ],
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/Microsoft.Insights/metrics/RunsSucceeded",
      "name": {
        "value": "RunsSucceeded",
        "localizedValue": "Runs Succeeded"
      },
      "type": "Microsoft.Insights/metrics",
      "unit": "Count"
    }
  ]
}
```

### <a name="use-armclient"></a>Использование ARMClient

Дополнительно на компьютере Windows можно использовать [ARMClient](https://github.com/projectkudu/armclient). ARMClient автоматически обрабатывает аутентификацию в Azure AD (и полученный маркер JWT). Ниже описано использование ARMClient для получения данных метрик.

1. Установите [Chocolatey](https://chocolatey.org/) и [ARMClient](https://github.com/projectkudu/armclient).
2. В окне терминала введите *armclient.exe login*. После этого отобразится запрос на вход в Azure.
3. Введите *armclient GET [ИД_ресурса]/providers/microsoft.insights/metricdefinitions?api-version=2016-03-01*.
4. Введите *armclient GET [ИД_ресурса]/providers/microsoft.insights/metrics?api-version=2016-09-01*.

Например, чтобы получить определения метрик для конкретного приложения логики, выполните следующую команду:

```console
armclient GET /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets/providers/microsoft.insights/metricDefinitions?api-version=2016-03-01
```

## <a name="retrieve-the-resource-id"></a>Получение идентификатора ресурса

Применение REST API действительно помогает понять доступные определения метрик, детализацию и связанные значения. Эта информация полезна при использовании [библиотеки управления Azure](/previous-versions/azure/reference/mt417623(v=azure.100)).

Для приведенного выше кода используемый идентификатор ресурса — это полный путь к нужному ресурсу Azure. Например, для запроса к веб-приложению Azure идентификатор ресурса будет следующим.

*/subscriptions/{ИД_подписки}/resourceGroups/{имя_группы_ресурсов}/providers/Microsoft.Web/sites/{имя_сайта}/*

Приведенный ниже список содержит несколько примеров форматов идентификаторов различных ресурсов Azure.

* **Центр Интернета вещей**: /subscriptions/*{ИД_подписки}*/resourceGroups/*{имя_группы_ресурсов}*/providers/Microsoft.Devices/IotHubs/*{имя_центра}*.
* **Пул эластичных баз данных SQL**: /subscriptions/*{ИД_подписки}*/resourceGroups/*{имя_группы_ресурсов}*/providers/Microsoft.Sql/servers/*{база_данных_в_пуле}*/elasticpools/*{имя_пула_SQL}*.
* **База данных SQL (версии 12)**: /subscriptions/*{ИД_подписки}*/resourceGroups/*{имя_группы_ресурсов}*/providers/Microsoft.Sql/servers/*{имя_сервера}*/databases/*{имя_базы_данных}*.
* **Служебная шина**: /subscriptions/*{ИД_подписки}*/resourceGroups/*{имя_группы_ресурсов}*/providers/Microsoft.ServiceBus/*{пространство_имен}*/*{имя_служебной_шины}*.
* **Масштабируемые наборы виртуальных машин**: /subscriptions/*{ИД_подписки}*/resourceGroups/*{имя_группы_ресурсов}*/providers/Microsoft.Compute/virtualMachineScaleSets/*{имя_ВМ}*.
* **Виртуальные машины**: /subscriptions/*{ИД_подписки}*/resourceGroups/*{имя_группы_ресурсов}*/providers/Microsoft.Compute/virtualMachines/*{имя_ВМ}*.
* **Центры событий**: /subscriptions/*{ИД_подписки}*/resourceGroups/*{имя_группы_ресурсов}*/providers/Microsoft.EventHub/namespaces/*{пространство_имен_центров_событий}*.

Существуют альтернативные подходы к извлечению идентификатора ресурса, включая использование Azure Resource Explorer, PowerShell, интерфейса командной строки Azure и просмотр требуемого ресурса на портале Azure.

### <a name="azure-resource-explorer"></a>Обозреватель ресурсов Azure

Чтобы найти идентификатор требуемого ресурса, удобно использовать инструмент [Azure Resource Explorer](https://resources.azure.com) . Перейдите к нужному ресурсу и просмотрите отображенный идентификатор, как показано на следующем снимке экрана.

![Azure Resource Explorer](./media/rest-api-walkthrough/azure_resource_explorer.png)

### <a name="azure-portal"></a>Портал Azure

Идентификатор ресурса можно также получить на портале Azure. Для этого перейдите к нужному ресурсу и выберите "Свойства". Идентификатор ресурса отображается в разделе "Свойства", как показано на следующем снимке экрана:

![Идентификатор ресурса в колонке "Свойства" на портале Azure](./media/rest-api-walkthrough/resourceid_azure_portal.png)

### <a name="azure-powershell"></a>Azure PowerShell

Идентификатор ресурса можно получить и с помощью командлетов Azure PowerShell. Например, чтобы получить идентификатор ресурса для приложения логики Azure, выполните командлет Get-AzureLogicApp, как показано в следующем примере.

```powershell
Get-AzLogicApp -ResourceGroupName azmon-rest-api-walkthrough -Name contosotweets
```

Результаты должны иметь следующий вид:

```output
Id             : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Logic/workflows/ContosoTweets
Name           : ContosoTweets
Type           : Microsoft.Logic/workflows
Location       : centralus
ChangedTime    : 8/21/2017 6:58:57 PM
CreatedTime    : 8/18/2017 7:54:21 PM
AccessEndpoint : https://prod-08.centralus.logic.azure.com:443/workflows/f3a91b352fcc47e6bff989b85446c5db
State          : Enabled
Definition     : {$schema, contentVersion, parameters, triggers...}
Parameters     : {[$connections, Microsoft.Azure.Management.Logic.Models.WorkflowParameter]}
SkuName        :
AppServicePlan :
PlanType       :
PlanId         :
Version        : 08586982649483762729
```

### <a name="azure-cli"></a>Azure CLI

Чтобы получить идентификатор ресурса для учетной записи хранения Azure с помощью Azure CLI, выполните `az storage account show` команду, как показано в следующем примере:

```azurecli
az storage account show -g azmon-rest-api-walkthrough -n contosotweets2017
```

Результаты должны иметь следующий вид:

```json
{
  "accessTier": null,
  "creationTime": "2017-08-18T19:58:41.840552+00:00",
  "customDomain": null,
  "enableHttpsTrafficOnly": false,
  "encryption": null,
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/azmon-rest-api-walkthrough/providers/Microsoft.Storage/storageAccounts/contosotweets2017",
  "identity": null,
  "kind": "Storage",
  "lastGeoFailoverTime": null,
  "location": "centralus",
  "name": "contosotweets2017",
  "networkAcls": null,
  "primaryEndpoints": {
    "blob": "https://contosotweets2017.blob.core.windows.net/",
    "file": "https://contosotweets2017.file.core.windows.net/",
    "queue": "https://contosotweets2017.queue.core.windows.net/",
    "table": "https://contosotweets2017.table.core.windows.net/"
  },
  "primaryLocation": "centralus",
  "provisioningState": "Succeeded",
  "resourceGroup": "azmon-rest-api-walkthrough",
  "secondaryEndpoints": null,
  "secondaryLocation": "eastus2",
  "sku": {
    "name": "Standard_GRS",
    "tier": "Standard"
  },
  "statusOfPrimary": "available",
  "statusOfSecondary": "available",
  "tags": {},
  "type": "Microsoft.Storage/storageAccounts"
}
```

> [!NOTE]
> Приложения логики Azure еще недоступны в Azure CLI, поэтому в предыдущем примере использовалась учетная запись хранения Azure.
>
>

## <a name="retrieve-activity-log-data"></a>Получение данных журнала действий

Помимо определений метрик и связанных значений можно также использовать REST API Azure Monitor, чтобы получить дополнительные интересные сведения, связанные с ресурсами Azure. Например, можно запросить данные [журнала действий](/rest/api/monitor/activitylogs) . В следующих примерах запросов используется REST API Azure Monitor для запроса журнала действий.

Получение журналов действий с фильтром.

``` HTTP
GET https://management.azure.com/subscriptions/089bd33f-d4ec-47fe-8ba5-0753aa5c5b33/providers/microsoft.insights/eventtypes/management/values?api-version=2015-04-01&$filter=eventTimestamp ge '2018-01-21T20:00:00Z' and eventTimestamp le '2018-01-23T20:00:00Z' and resourceGroupName eq 'MSSupportGroup'
```

Получение журналов действий с фильтром и выбором.

```HTTP
GET https://management.azure.com/subscriptions/089bd33f-d4ec-47fe-8ba5-0753aa5c5b33/providers/microsoft.insights/eventtypes/management/values?api-version=2015-04-01&$filter=eventTimestamp ge '2015-01-21T20:00:00Z' and eventTimestamp le '2015-01-23T20:00:00Z' and resourceGroupName eq 'MSSupportGroup'&$select=eventName,id,resourceGroupName,resourceProviderName,operationName,status,eventTimestamp,correlationId,submissionTimestamp,level
```

Получение журналов действий с выбором.

```HTTP
GET https://management.azure.com/subscriptions/089bd33f-d4ec-47fe-8ba5-0753aa5c5b33/providers/microsoft.insights/eventtypes/management/values?api-version=2015-04-01&$select=eventName,id,resourceGroupName,resourceProviderName,operationName,status,eventTimestamp,correlationId,submissionTimestamp,level
```

Получение журналов действий без фильтра и выбора.

```HTTP
GET https://management.azure.com/subscriptions/089bd33f-d4ec-47fe-8ba5-0753aa5c5b33/providers/microsoft.insights/eventtypes/management/values?api-version=2015-04-01
```

## <a name="next-steps"></a>Дальнейшие действия

* Прочитайте [общие сведения о мониторинге](../overview.md).
* Ознакомьтесь с разделом [Метрики, поддерживаемые Azure Monitor](./metrics-supported.md).
* Ознакомьтесь со [справочником по REST API Microsoft Azure Monitor](/rest/api/monitor/).
* Ознакомьтесь с [библиотекой управления Azure](/previous-versions/azure/reference/mt417623(v=azure.100)).