---
title: Справочник по прокси-серверу пользовательских ресурсов
description: Справочник по пользовательскому прокси-службе ресурсов для поставщиков настраиваемых ресурсов Azure. В этой статье рассматриваются требования к конечным точкам, реализующим настраиваемые ресурсы прокси-сервера.
ms.topic: conceptual
ms.author: jobreen
author: jjbfour
ms.date: 06/20/2019
ms.openlocfilehash: 46b38686b39836f3d4bfb80686d514f932a79bf3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "75650465"
---
# <a name="custom-resource-proxy-reference"></a>Справочник по прокси-службе настраиваемого ресурса

В этой статье рассматриваются требования к конечным точкам, реализующим настраиваемые ресурсы прокси-сервера. Если вы не знакомы с поставщиками настраиваемых ресурсов Azure, ознакомьтесь [с обзором пользовательских поставщиков ресурсов](overview.md).

## <a name="how-to-define-a-proxy-resource-endpoint"></a>Определение конечной точки ресурса прокси-сервера

Ресурс-посредник можно создать, указав для **раутингтипе** значение "Proxy".

Пример пользовательского поставщика ресурсов:

```JSON
{
  "properties": {
    "resourceTypes": [
      {
        "name": "myCustomResources",
        "routingType": "Proxy",
        "endpoint": "https://{endpointURL}/"
      }
    ]
  },
  "location": "eastus",
  "type": "Microsoft.CustomProviders/resourceProviders",
  "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}",
  "name": "{resourceProviderName}"
}
```

## <a name="building-proxy-resource-endpoint"></a>Создание конечной точки ресурса прокси-сервера

**Конечная точка** , реализующая **конечную точку** ресурса "прокси", должна поддерживать запрос и ответ для нового API в Azure. В этом случае **ResourceType** создает новый API ресурсов Azure для `PUT` , `GET` и `DELETE` для выполнения CRUD на одном ресурсе, а также `GET` для получения всех существующих ресурсов.

> [!NOTE]
> `id`Поля, `name` и `type` не требуются, но необходимы для интеграции настраиваемого ресурса с существующей экосистемой Azure.

Образец ресурса:

``` JSON
{
    "name": "{myCustomResourceName}",
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources/{myCustomResourceName}",
    "type": "Microsoft.CustomProviders/resourceProviders/myCustomResources",
    "properties": {
        "myProperty1": "myPropertyValue1",
        "myProperty2": {
            "myProperty3" : "myPropertyValue3"
        }
    }
}
```

Ссылка на параметр:

Свойство | Образец | Описание
---|---|---
name | {myCustomResourceName} | Название настраиваемого ресурса.
тип | Microsoft.CustomProviders/resourceProviders/{имя типа ресурса} | Пространство имен типа ресурса.
идентификатор | /subscriptions/{ИД подписки}/resourceGroups/{имя группы ресурсов}/<br>providers/Microsoft.CustomProviders/resourceProviders/{имя поставщика ресурсов}/<br>Микустомресаурцес/{Микустомресаурценаме} " | Идентификатор ресурса.

### <a name="create-a-custom-resource"></a>Создание настраиваемого ресурса

Входящий запрос API Azure:

``` HTTP
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resource-provider-name}/myCustomResources/{myCustomResourceName}?api-version=2018-09-01-preview
Authorization: Bearer eyJ0e...
Content-Type: application/json

{
    "properties": {
        "myProperty1": "myPropertyValue1",
        "myProperty2": {
            "myProperty3" : "myPropertyValue3"
        }
    }
}
```

Затем этот запрос будет перенаправлен в **конечную точку** в форме:

``` HTTP
PUT https://{endpointURL}/?api-version=2018-09-01-preview
Content-Type: application/json
X-MS-CustomProviders-RequestPath: /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources/{myCustomResourceName}

{
    "properties": {
        "myProperty1": "myPropertyValue1",
        "myProperty2": {
            "myProperty3" : "myPropertyValue3"
        }
    }
}
```

Аналогичным образом ответ от **конечной точки** затем перенаправляется обратно клиенту. Ответ от конечной точки должен вернуть:

- Допустимый документ объекта JSON. Все массивы и строки должны быть вложены в верхний объект.
- `Content-Type`Для заголовка необходимо задать значение Application/JSON; charset = UTF-8 ".

**Конечная точка** Ответ

``` HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "name": "{myCustomResourceName}",
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources/{myCustomResourceName}",
    "type": "Microsoft.CustomProviders/resourceProviders/myCustomResources",
    "properties": {
        "myProperty1": "myPropertyValue1",
        "myProperty2": {
            "myProperty3" : "myPropertyValue3"
        }
    }
}
```

Ответ поставщика настраиваемых ресурсов Azure:

``` HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "name": "{myCustomResourceName}",
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources/{myCustomResourceName}",
    "type": "Microsoft.CustomProviders/resourceProviders/myCustomResources",
    "properties": {
        "myProperty1": "myPropertyValue1",
        "myProperty2": {
            "myProperty3" : "myPropertyValue3"
        }
    }
}
```

### <a name="remove-a-custom-resource"></a>Удаление настраиваемого ресурса

Входящий запрос API Azure:

``` HTTP
Delete https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources/{myCustomResourceName}?api-version=2018-09-01-preview
Authorization: Bearer eyJ0e...
Content-Type: application/json
```

Затем этот запрос будет перенаправлен в **конечную точку** в форме:

``` HTTP
Delete https://{endpointURL}/?api-version=2018-09-01-preview
Content-Type: application/json
X-MS-CustomProviders-RequestPath: /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources/{myCustomResourceName}
```

Аналогичным образом ответ **конечной точки** отправляется обратно клиенту. Ответ от конечной точки должен вернуть:

- Допустимый документ объекта JSON. Все массивы и строки должны быть вложены в верхний объект.
- `Content-Type`Для заголовка необходимо задать значение Application/JSON; charset = UTF-8 ".

**Конечная точка** Ответ

``` HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
```

Ответ поставщика настраиваемых ресурсов Azure:

``` HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
```

### <a name="retrieve-a-custom-resource"></a>Извлечение настраиваемого ресурса

Входящий запрос API Azure:

``` HTTP
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources/{myCustomResourceName}?api-version=2018-09-01-preview
Authorization: Bearer eyJ0e...
Content-Type: application/json
```

Затем этот запрос будет перенаправлен в **конечную точку** в форме:

``` HTTP
GET https://{endpointURL}/?api-version=2018-09-01-preview
Content-Type: application/json
X-MS-CustomProviders-RequestPath: /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources/{myCustomResourceName}
```

Аналогичным образом ответ от **конечной точки** затем перенаправляется обратно клиенту. Ответ от конечной точки должен вернуть:

- Допустимый документ объекта JSON. Все массивы и строки должны быть вложены в верхний объект.
- `Content-Type`Для заголовка необходимо задать значение Application/JSON; charset = UTF-8 ".

**Конечная точка** Ответ

``` HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "name": "{myCustomResourceName}",
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources/{myCustomResourceName}",
    "type": "Microsoft.CustomProviders/resourceProviders/myCustomResources",
    "properties": {
        "myProperty1": "myPropertyValue1",
        "myProperty2": {
            "myProperty3" : "myPropertyValue3"
        }
    }
}
```

Ответ поставщика настраиваемых ресурсов Azure:

``` HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "name": "{myCustomResourceName}",
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources/{myCustomResourceName}",
    "type": "Microsoft.CustomProviders/resourceProviders/myCustomResources",
    "properties": {
        "myProperty1": "myPropertyValue1",
        "myProperty2": {
            "myProperty3" : "myPropertyValue3"
        }
    }
}
```

### <a name="enumerate-all-custom-resources"></a>Перечисление всех настраиваемых ресурсов

Входящий запрос API Azure:

``` HTTP
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources?api-version=2018-09-01-preview
Authorization: Bearer eyJ0e...
Content-Type: application/json
```

Затем этот запрос будет перенаправлен в **конечную точку** в форме:

``` HTTP
GET https://{endpointURL}/?api-version=2018-09-01-preview
Content-Type: application/json
X-MS-CustomProviders-RequestPath: /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources
```

Аналогичным образом ответ от **конечной точки** затем перенаправляется обратно клиенту. Ответ от конечной точки должен вернуть:

- Допустимый документ объекта JSON. Все массивы и строки должны быть вложены в верхний объект.
- `Content-Type`Для заголовка необходимо задать значение Application/JSON; charset = UTF-8 ".
- Список ресурсов должен размещаться в `value` свойстве верхнего уровня.

**Конечная точка** Ответ

``` HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "value" : [
        {
            "name": "{myCustomResourceName}",
            "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources/{myCustomResourceName}",
            "type": "Microsoft.CustomProviders/resourceProviders/myCustomResources",
            "properties": {
                "myProperty1": "myPropertyValue1",
                "myProperty2": {
                    "myProperty3" : "myPropertyValue3"
                }
            }
        }
    ]
}
```

Ответ поставщика настраиваемых ресурсов Azure:

``` HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "value" : [
        {
            "name": "{myCustomResourceName}",
            "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources/{myCustomResourceName}",
            "type": "Microsoft.CustomProviders/resourceProviders/myCustomResources",
            "properties": {
                "myProperty1": "myPropertyValue1",
                "myProperty2": {
                    "myProperty3" : "myPropertyValue3"
                }
            }
        }
    ]
}
```

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения о поставщиках настраиваемых ресурсов Azure](overview.md)
- [Краткое руководство. Создание настраиваемого поставщика ресурсов Azure и развертывание настраиваемых ресурсов](./create-custom-provider.md)
- [Руководство. Создание настраиваемых действий и ресурсов в Azure](./tutorial-get-started-with-custom-providers.md)
- [Как добавлять настраиваемые действия в Azure REST API](./custom-providers-action-endpoint-how-to.md)
- [Справочник: Справочник по пользовательскому кэшу ресурсов](proxy-cache-resource-endpoint-reference.md)
