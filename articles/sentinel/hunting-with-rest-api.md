---
title: Управление запросами на поиск и облаку в Sentinel Azure с помощью REST API | Документация Майкрософт
description: В этой статье описывается, как функции Поиск Sentinel маркеров Azure позволяют воспользоваться преимуществами Log Analytics "REST API для управления запросами на поиск и облаку.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: reference
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/06/2020
ms.author: yelevin
ms.openlocfilehash: 64f05e18ff757d9f086cf06d74109bf64e32a05c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98795692"
---
# <a name="manage-hunting-and-livestream-queries-in-azure-sentinel-using-rest-api"></a>Управление запросами на охоту и потоковую передачу в реальном времени в Azure Sentinel с помощью REST API

Azure Sentinel, которая встроена в Azure Monitor Log Analytics, позволяет использовать Log Analytics "REST API для управления запросами поиском и облаку. В этом документе показано, как создавать запросы на поиск и управлять ими с помощью REST API.  Созданные таким образом запросы будут отображаться в пользовательском интерфейсе Sentinel Azure.

Дополнительные сведения об [API сохраненных поисков](/rest/api/loganalytics/savedsearches)см. в подробном справочнике по REST API.

## <a name="api-examples"></a>Примеры с API

В следующих примерах заменяйте эти заполнители на замену, предписанную в следующей таблице:

| Заполнитель | Заменить на |
|-|-|
| **Подписки** | имя подписки, к которой применяется запрос поиска или облаку. |
| **ResourceGroupName** | имя группы ресурсов, к которой применяется запрос поиска или облаку. |
| **{Саведсеарчид}** | уникальный идентификатор (GUID) для каждого запроса поиска. |
| **WorkspaceName** | имя рабочей области Log Analytics, которая является целью запроса. |
| **DisplayName** | отображаемое имя, выбранное для запроса. |
| **Nописание** | Описание запроса поиска или облаку. |
| **Прием** | соответствующая тактика MITRE АТРИ&CK, применимая к запросу. |
| **Выбор** | выражение запроса для запроса. |
|  

### <a name="example-1"></a>Пример 1

В этом примере показано, как создать или обновить запрос поиска для заданной рабочей области Sentinel Azure.  Для запроса облаку замените *«Category»: «Поиск запросов»* на *«Category»: «облаку Queries»* в **тексте запроса**: 

#### <a name="request-header"></a>Заголовок запроса

```http
PUT https://management.azure.com/subscriptions/{subscriptionId} _
    /resourcegroups/{resourceGroupName} _
    /providers/Microsoft.OperationalInsights/workspaces/{workspaceName} _
    /savedSearches/{savedSearchId}?api-version=2020-03-01-preview
```

#### <a name="request-body"></a>Текст запроса

```json
{
"properties": {
    “Category”: “Hunting Queries”,
    "DisplayName": "HuntingRule02",
    "Query": "SecurityEvent | where EventID == \"4688\" | where CommandLine contains \"-noni -ep bypass $\"",
    “Tags”: [
        { 
        “Name”: “Description”,
        “Value”: “Test Hunting Query”
        },
        { 
        “Name”: “Tactics”,
        “Value”: “Execution, Discovery”
        }
        ]        
    }
}
```

### <a name="example-2"></a>Пример 2

В этом примере показано, как удалить запрос поиска или облаку для указанной рабочей области Sentinel Azure.

```http
DELETE https://management.azure.com/subscriptions/{subscriptionId} _
       /resourcegroups/{resourceGroupName} _
       /providers/Microsoft.OperationalInsights/workspaces/{workspaceName} _
       /savedSearches/{savedSearchId}?api-version=2020-03-01-preview
```

### <a name="example-3"></a>Пример 3

В этом примере показано, как получить запрос поиска или облаку для конкретной рабочей области:

```http
GET https://management.azure.com/subscriptions/{subscriptionId} _
    /resourcegroups/{resourceGroupName} _
    /providers/Microsoft.OperationalInsights/workspaces/{workspaceName} _
    /savedSearches/{savedSearchId}?api-version=2020-03-01-preview
```

## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали, как управлять запросами Поиск и облаку в Sentinel в Azure с помощью API Log Analytics. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- [Упреждающая охота за угрозами](hunting.md)
- [Использование записных книжек для выполнения автоматизированного поисковых кампаний](notebooks.md)