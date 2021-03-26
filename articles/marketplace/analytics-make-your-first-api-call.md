---
title: Сделайте первый вызов API для доступа к данным анализа коммерческих рынков
description: Примеры использования API для доступа к данным анализа коммерческих рынков.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: sayantanroy83
ms.author: sroy
ms.date: 3/08/2021
ms.openlocfilehash: 65ee0847e6a59976eec223b68b1f3e0c464674e8
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105563809"
---
# <a name="make-your-first-api-call-to-access-commercial-marketplace-analytics-data"></a>Сделайте первый вызов API для доступа к данным анализа коммерческих рынков

Список API-интерфейсов для доступа к данным анализа коммерческих рынков см. в статье [интерфейсы API для доступа к данным анализа коммерческих рынков](analytics-available-apis.md). Прежде чем выполнять первый вызов API, убедитесь, что выполнены [Предварительные требования](analytics-prerequisites.md) для программного доступа к данным аналитики Marketplace.

## <a name="token-generation"></a>Создание токенов

Перед вызовом любого из методов сначала необходимо получить маркер доступа Azure Active Directory (Azure AD). Необходимо передать маркер доступа Azure AD в заголовок авторизации каждого метода в API. После получения маркера доступа у вас будет 60 минут, чтобы использовать его до истечения срока действия. По истечении срока действия маркера можно обновить маркер и продолжить его использование для дальнейших вызовов API.

См. пример запроса ниже для создания маркера. Для создания маркера требуются три значения: `clientId` , `clientSecret` и `tenantId` . `resource`Параметр должен иметь значение `https://graph.windows.net` .

***Пример запроса***:

```json
curl --location --request POST 'https://login.microsoftonline.com/{TenantId}/oauth2/token' \
--header 'return-client-request-id: true' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'resource=https://graph.windows.net' \
--data-urlencode 'client_id={client_id}' \
--data-urlencode 'client_secret={client_secret}' \
--data-urlencode 'grant_type=client_credentials'
```

***Пример ответа***:

```json
{
    "token_type": "Bearer",
    "expires_in": "3599",
    "ext_expires_in": "3599",
    "expires_on": "1612794445",
    "not_before": "1612790545",
    "resource": "https://graph.windows.net",
    "access_token": {Token}
}
```

Дополнительные сведения о получении маркера Azure AD для приложения см. в статье [доступ к данным аналитики с помощью служб хранилища](/windows/uwp/monetize/access-analytics-data-using-windows-store-services#step-2-obtain-an-azure-ad-access-token).

## <a name="programmatic-api-call"></a>Программный вызов API

После получения маркера Azure AD, как описано в предыдущем разделе, выполните следующие действия, чтобы создать первый отчет с программным доступом.

Данные можно скачать из следующих наборов данных (datasetName):

- исвкустомер
- исвмаркетплацеинсигхтс
- исвусаже
- исвордер

В следующих разделах мы рассмотрим примеры программного доступа `OrderId` из набора данных исвордер.

### <a name="step-1-make-a-rest-call-using-the-get-datasets-api"></a>Шаг 1. вызов функции RESTFUL с помощью API получения наборов данных

Ответ API предоставляет имя набора данных, откуда можно скачать отчет. Для конкретного набора данных ответ API также предоставляет список выбираемых столбцов, которые можно использовать для пользовательского шаблона отчета. Для получения имен столбцов можно также обратиться к следующим таблицам:

- [Таблица сведений о заказах](orders-dashboard.md#orders-details-table)
- [Таблица сведений об использовании](usage-dashboard.md#usage-details-table)
- [Таблица сведений о клиенте](customer-dashboard.md#customer-details-table)
- [Таблица сведений Marketplace Insights](insights-dashboard.md#marketplace-insights-details-table)

***Пример запроса***:

```json
curl 
--location 
--request GET 'https://api.partnercenter.microsoft.com/insights/v1/cmp/ScheduledDataset ' \ 
--header 'Authorization: Bearer <AzureADToken>'
```

***Пример ответа***:

```json
{
    "value": [
        {
            "datasetName": "ISVOrder",
            "selectableColumns": [
                "MarketplaceSubscriptionId",
                "MonthStartDate",
                "OfferType",
                "AzureLicenseType",
                "Sku",
                "CustomerCountry",
                "IsPreviewSKU",
                "OrderId",
                "OrderQuantity",
                "CloudInstanceName",
                "IsNewCustomer",
                "OrderStatus",
                "OrderCancelDate",
                "CustomerCompanyName",
                "CustomerName",
                "OrderPurchaseDate",
                "OfferName",
                "TrialEndDate",
                "CustomerId",
                "BillingAccountId"
            ],
            "availableMetrics": [],
            "availableDateRanges": [
                "LAST_MONTH",
                "LAST_3_MONTHS",
                "LAST_6_MONTHS",
                "LIFETIME"
            ]
        },
    ],
    "totalCount": 1,
    "message": "Dataset fetched successfully",
    "statusCode": 200
}
```

### <a name="step-2-create-the-custom-query"></a>Шаг 2. Создание настраиваемого запроса

На этом шаге мы будем использовать идентификатор заказа из отчета о заказах, чтобы создать настраиваемый запрос для нужного отчета. Значение по умолчанию, `timespan` если оно не указано в запросе, составляет шесть месяцев.

***Пример запроса***:

```json
curl 
--location 
--request POST ' https://api.partnercenter.microsoft.com/insights/v1/cmp/ScheduledQueries' \ 
--header ' Authorization: Bearer <AzureAD_Token>' \ 
--header 'Content-Type: application/json' \ 
--data-raw 
            '{ 
                "Query": "SELECT OrderId from ISVOrder", 
                "Name": "ISVOrderQuery1", 
                "Description": "Get a list of all Order IDs" 
             }'
```

***Пример ответа***:

```json
{
    "value": [
        {
            "queryId": "78be43f2-e35f-491a-8cd5-78fe14194f9c",
            "name": "ISVOrderQuery1",
            "description": "Get a list of all Order IDs”,
            "query": "SELECT OrderId from ISVOrder",
            "type": "userDefined",
            "user": "142344300",
            "createdTime": "2021-01-06T05:38:34",
            "modifiedTime": null
        }
    ],
    "totalCount": 1,
    "message": "Query created successfully",
    "statusCode": 200
}
```

При успешном выполнении запроса `queryId` создается объект, который необходимо использовать для создания отчета.

### <a name="step-3-execute-test-query-api"></a>Шаг 3. выполнение API тестовых запросов

На этом шаге мы будем использовать API тестовых запросов для получения первых 100 строк для созданного запроса.

***Пример запроса***:

```json
curl 
--location 
--request GET 'https://api.partnercenter.microsoft.com/insights/v1/cmp/ScheduledQueries/testQueryResult?exportQuery=SELECT%20OrderId%20from%20ISVOrder' \ 
--header ' Authorization: Bearer <AzureADToken>'
```

***Пример ответа***:

```json
{
    "value": [
        {
            "OrderId": "086365c6-9c38-4fba-904a-6228f6cb2ba8"
        },
        {
            "OrderId": "086365c6-9c38-4fba-904a-6228f6cb2bb8"
        },
        {
            "OrderId": "086365c6-9c38-4fba-904a-6228f6cb2bc8"
        },
        {
            "OrderId": "086365c6-9c38-4fba-904a-6228f6cb2bd8"
        },
        {
            "OrderId": "086365c6-9c38-4fba-904a-6228f6cb2be8"
        },
               .
               .
               .

        {
            "OrderId": "086365c6-9c38-4fba-904a-6228f6cb2bf0"
        },
        {
            "OrderId": "086365c6-9c38-4fba-904a-6228f6cb2bf1"
        },
        {
            "OrderId": "086365c6-9c38-4fba-904a-6228f6cb2bf2"
        },
        {
            "OrderId": "086365c6-9c38-4fba-904a-6228f6cb2bf3"
        },
        {
            "OrderId": "086365c6-9c38-4fba-904a-6228f6cb2bf4"
        }
    ],
    "totalCount": 100,
    "message": null,
    "statusCode": 200
}
```

### <a name="step-4-create-the-report"></a>Шаг 4. Создание отчета

На этом шаге мы воспользуемся созданным ранее `QueryId` для создания отчета.

***Пример запроса***:

```json
curl 
--location 
--request POST 'https://api.partnercenter.microsoft.com/insights/v1/cmp/ScheduledReport' \ 
--header ' Authorization: Bearer <AzureADToken>' \ 
--header 'Content-Type: application/json' \ 
--data-raw 
                 '{
                   “ReportName": "ISVReport1",
                   "Description": "Report for getting list of Order Ids",
                   "QueryId": "78be43f2-e35f-491a-8cd5-78fe14194f9c",
                   "StartTime": "2021-01-06T19:00:00Z”,
                   "RecurrenceInterval": 48,
                   "RecurrenceCount": 20,
                    "Format": "csv"
                  }'
```

<br>

_**Таблица 1. Описание параметров, используемых в этом примере запроса**_

| Параметр | Описание |
| ------------ | ------------- |
| `Description` | Введите краткое описание создаваемого отчета. |
| `QueryId` | Это тот `queryId` , который был создан при создании запроса в шаге 2. Создание настраиваемого запроса. |
| `StartTime` | Время начала первого выполнения отчета. |
| `RecurrenceInterval` | Интервал повторения, указанный во время создания отчета. |
| `RecurrenceCount` | Число повторений, указанное при создании отчета. |
| `Format` | Поддерживаются форматы файлов CSV и TSV. |
|||

***Пример ответа***:

```json
{
    "value": [
        {
            "reportId": "72fa95ab-35f5-4d44-a1ee-503abbc88003",
            "reportName": "ISVReport1",
            "description": "Report for getting list of Order Ids",
            "queryId": "78be43f2-e35f-491a-8cd5-78fe14194f9c",
            "query": "SELECT OrderId from ISVOrder",
            "user": "142344300",
            "createdTime": "2021-01-06T05:46:00Z",
            "modifiedTime": null,
            "startTime": "2021-01-06T19:00:00Z",
            "reportStatus": "Active",
            "recurrenceInterval": 48,
            "recurrenceCount": 20,
            "callbackUrl": null,
            "format": "csv"
        }
    ],
    "totalCount": 1,
    "message": "Report created successfully",
    "statusCode": 200
}
```

При успешном выполнении `reportId` создается объект, который необходимо использовать для планирования загрузки отчета.

### <a name="step-5-execute-report-executions-api"></a>Шаг 5. выполнение API выполнения отчетов

Чтобы получить безопасное расположение (URL-адрес) отчета, мы теперь выполним API выполнения отчетов.

***Пример запроса***:

```json
Curl
--location
--request GET 'https://api.partnercenter.microsoft.com/insights/v1/cmp/ScheduledReport/execution/72fa95ab-35f5-4d44-a1ee-503abbc88003' \
--header ' Authorization: Bearer <AzureADToken>' \
```

***Пример ответа***:

```json
{
    "value": [
        {
            "executionId": "1f18b53b-df30-4d98-85ee-e6c7e687aeed",
            "reportId": "72fa95ab-35f5-4d44-a1ee-503abbc88003",
            "recurrenceInterval": 48,
            "recurrenceCount": 20,
            "callbackUrl": null,
            "format": "csv",
            "executionStatus": "Pending",
            "reportAccessSecureLink": null,
            "reportExpiryTime": null,
            "reportGeneratedTime": null
        }
    ],
    "totalCount": 1,
    "message": null,
    "statusCode": 200
}
```

## <a name="next-steps"></a>Дальнейшие действия

- Вы можете испытать интерфейсы API через [URL-адрес API Swagger](https://partneranalytics-api.azure-api.net/analytics/cmp/swagger/index.html) .
- [Парадигма программного доступа](analytics-programmatic-access.md)