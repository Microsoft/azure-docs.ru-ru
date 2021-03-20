---
title: Получение данных об использовании виртуальной машины Azure с помощью REST API
description: Используйте API REST Azure для сбора метрик использования для виртуальной машины.
author: rloutlaw
ms.service: virtual-machines
ms.subservice: monitoring
ms.custom: REST
ms.topic: how-to
ms.date: 06/13/2018
ms.author: routlaw
ms.openlocfilehash: a7237bfc82a932b774b4b6ef293c242a84fd75af
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100591211"
---
# <a name="get-virtual-machine-usage-metrics-using-the-rest-api"></a>Получение метрик использования виртуальной машины с помощью REST API

В этом примере показано, как получить ЦП для виртуальной машины Linux с помощью [Azure REST API](/rest/api/azure/).

Полная справочная документация и дополнительные образцы для REST API можно найти в [Azure Monitor REST API reference](/rest/api/monitor) (Справочник по REST API Azure Monitor). 

## <a name="build-the-request"></a>Создание запроса

Используйте следующий запрос GET для сбора с виртуальной машины [метрики ЦП в процентах](../../azure-monitor/essentials/metrics-supported.md#microsoftcomputevirtualmachines)

```http
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmname}/providers/microsoft.insights/metrics?api-version=2018-01-01&metricnames=Percentage%20CPU&timespan=2018-06-05T03:00:00Z/2018-06-07T03:00:00Z
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
| имя_группы_ресурсов | Имя группы ресурсов Azure, связанное с ресурсом. Вы можете получить это значение из API-интерфейса Azure Resource Manager, CLI или портала. |
| vmname | Имя виртуальной машины. |
| metricnames | Разделенный запятыми список допустимых [метрик Load Balancer](../../load-balancer/load-balancer-standard-diagnostics.md). |
| api-version | Версия API для использования в запросе.<br /><br /> В этом документе рассматривается API версии `2018-01-01`, которая включена в приведенный выше URL-адрес.  |
| Интервал времени | Строка, в следующем формате `startDateTime_ISO/endDateTime_ISO`, определяет диапазон времени для возвращаемых метрик. Этот необязательный параметр имеет значение для возврата суточных данных в примере. |
| &nbsp; | &nbsp; |

### <a name="request-body"></a>Текст запроса

Для этой операции текст запроса не требуется.

## <a name="handle-the-response"></a>Обработка ответа

Код состояния 200 возвращается в том случае, если успешно возвращается список значений метрик. Полный список кодов ошибок доступен в [справочной документации](/rest/api/monitor/metrics/list#errorresponse).

## <a name="example-response"></a>Пример ответа 

```json
{
    "cost": 0,
    "timespan": "2018-06-08T23:48:10Z/2018-06-09T00:48:10Z",
    "interval": "PT1M",
    "value": [
        {
            "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmname}/providers/microsoft.insights/metrics?api-version=2018-01-01&metricnames=Percentage%20CPU",
            "type": "Microsoft.Insights/metrics",
            "name": {
                "value": "Percentage CPU",
                "localizedValue": "Percentage CPU"
            },
            "unit": "Percent",
            "timeseries": [
                {
                    "metadatavalues": [],
                    "data": [
                        {
                            "timeStamp": "2018-06-08T23:48:00Z",
                            "average": 0.44
                        },
                        {
                            "timeStamp": "2018-06-08T23:49:00Z",
                            "average": 0.31
                        },
                        {
                            "timeStamp": "2018-06-08T23:50:00Z",
                            "average": 0.29
                        },
                        {
                            "timeStamp": "2018-06-08T23:51:00Z",
                            "average": 0.29
                        },
                        {
                            "timeStamp": "2018-06-08T23:52:00Z",
                            "average": 0.285
                        } ]
                } ]
        } ]
}
```
