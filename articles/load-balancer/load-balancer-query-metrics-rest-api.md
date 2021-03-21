---
title: Извлечение метрик через REST API
titleSuffix: Azure Load Balancer
description: В этой статье приступите к работе с API-интерфейсами Azure для сбора метрик работоспособности и использования для Azure Load Balancer.
services: sql-database
author: asudbring
manager: KumudD
ms.service: load-balancer
ms.custom: REST, seodec18
ms.topic: how-to
ms.date: 11/19/2019
ms.author: allensu
ms.openlocfilehash: 391be596d890e05e6a8fdaf35d2cade371e468d6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102213189"
---
# <a name="get-load-balancer-usage-metrics-using-the-rest-api"></a>Получение метрик использования Load Balancer с помощью REST API

Собирайте количество байтов, обработанных [Load Balancer (цен. Категория "Стандартный")](./load-balancer-overview.md) , в течение интервала времени с помощью [REST API Azure](/rest/api/azure/).

Полная справочная документация и дополнительные образцы для REST API можно найти в [Azure Monitor REST API reference](/rest/api/monitor) (Справочник по REST API Azure Monitor). 

## <a name="build-the-request"></a>Создание запроса

Чтобы собрать данные [метрики ByteCount](./load-balancer-standard-diagnostics.md#multi-dimensional-metrics) с Load Balancer ценовой категории "Стандартный", выполните приведенный ниже запрос GET. 

```http
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/loadBalancers/{loadBalancerName}/providers/microsoft.insights/metrics?api-version=2018-01-01&metricnames=ByteCount&timespan=2018-06-05T03:00:00Z/2018-06-07T03:00:00Z
```

### <a name="request-headers"></a>Заголовки запросов

Ниже приведены обязательные заголовки. 

|Заголовок запроса|Описание|  
|--------------------|-----------------|  
|*Content-Type:*|Обязательный элемент. Задайте значение `application/json`.|  
|*Authorization:*|Обязательный элемент. Задайте допустимый [маркер доступа](/rest/api/azure/#authorization-code-grant-interactive-clients) `Bearer`. |  

### <a name="uri-parameters"></a>Параметры универсального кода ресурса (URI)

| Имя | Описание |
| :--- | :---------- |
| subscriptionId | Идентификатор подписки Azure. Если у вас несколько подписок, см. раздел [Работа с несколькими подписками](/cli/azure/manage-azure-subscriptions-azure-cli). |
| имя_группы_ресурсов | Имя группы ресурсов, к которой относится ресурс. Это значение можно получить с помощью API-интерфейса Azure Resource Manager, CLI или портала. |
| loadBalancerName | Имя Azure Load Balancer. |
| имена метрик | Разделенный запятыми список допустимых [метрик Load Balancer](./load-balancer-standard-diagnostics.md). |
| api-version | Версия API для использования в запросе.<br /><br /> В этом документе рассматривается API версии `2018-01-01`, которая включена в приведенный выше URL-адрес.  |
| Интервал времени | Временной диапазон запроса. Это строка в следующем формате `startDateTime_ISO/endDateTime_ISO` . Этот необязательный параметр имеет значение для возврата суточных данных в примере. |
| &nbsp; | &nbsp; |

### <a name="request-body"></a>Текст запроса

Для этой операции текст запроса не требуется.

## <a name="handle-the-response"></a>Обработка ответа

Код состояния 200 возвращается в том случае, если успешно возвращается список значений метрик. Полный список кодов ошибок доступен в [справочной документации](/rest/api/monitor/metrics/list#errorresponse).

## <a name="example-response"></a>Пример ответа 

```json
{
    "cost": 0,
    "timespan": "2018-06-05T03:00:00Z/2018-06-07T03:00:00Z",
    "interval": "PT1M",
    "value": [
        {
            "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/loadBalancers/{loadBalancerName}/providers/Microsoft.Insights/metrics/ByteCount",
            "type": "Microsoft.Insights/metrics",
            "name": {
                "value": "ByteCount",
                "localizedValue": "Byte Count"
            },
            "unit": "Count",
            "timeseries": [
                {
                    "metadatavalues": [],
                    "data": [
                        {
                            "timeStamp": "2018-06-06T17:24:00Z",
                            "total": 1067921034.0
                        },
                        {
                            "timeStamp": "2018-06-06T17:25:00Z",
                            "total": 0.0
                        },
                        {
                            "timeStamp": "2018-06-06T17:26:00Z",
                            "total": 3781344.0
                        },
                    ]
                }
            ]
        }
    ],
    "namespace": "Microsoft.Network/loadBalancers",
    "resourceregion": "eastus"
}
```