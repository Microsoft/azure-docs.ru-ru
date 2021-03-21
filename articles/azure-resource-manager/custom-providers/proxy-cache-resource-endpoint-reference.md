---
title: Справочник по кэшу пользовательских ресурсов
description: Справочник по пользовательскому кэшу ресурсов для поставщиков настраиваемых ресурсов Azure. В этой статье рассматриваются требования к конечным точкам, реализующим пользовательские ресурсы кэша.
ms.topic: conceptual
ms.author: jobreen
author: jjbfour
ms.date: 06/20/2019
ms.openlocfilehash: e1b8c44f020d18066423eed236018308fe88b607
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "75650387"
---
# <a name="custom-resource-cache-reference"></a>Ссылка на пользовательский кэш ресурсов

В этой статье рассматриваются требования к конечным точкам, реализующим пользовательские ресурсы кэша. Если вы не знакомы с поставщиками настраиваемых ресурсов Azure, ознакомьтесь [с обзором пользовательских поставщиков ресурсов](overview.md).

## <a name="how-to-define-a-cache-resource-endpoint"></a>Как определить конечную точку ресурса кэша

Ресурс-посредник можно создать, указав для **раутингтипе** значение Proxy, Cache.

Пример пользовательского поставщика ресурсов:

```JSON
{
  "properties": {
    "resourceTypes": [
      {
        "name": "myCustomResources",
        "routingType": "Proxy, Cache",
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

**Конечная** точка, реализующая **конечную точку** ресурса "прокси, кэш", должна поддерживать запрос и ответ для нового API в Azure. В этом случае **ResourceType** создает новый API ресурсов Azure для `PUT` , `GET` и `DELETE` для выполнения CRUD на одном ресурсе, а также `GET` для получения всех существующих ресурсов:

> [!NOTE]
> API Azure создаст методы запроса, `PUT` `GET` и `DELETE` , но **конечной точке** кэша требуется только обработку `PUT` и `DELETE` .
> Мы рекомендуем, чтобы **Конечная точка** также реализовала `GET` .

### <a name="create-a-custom-resource"></a>Создание настраиваемого ресурса

Входящий запрос API Azure:

``` HTTP
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources/{myCustomResourceName}?api-version=2018-09-01-preview
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
- Настраиваемый поставщик ресурсов перезапишет `name` `type` поля, и `id` для запроса.
- Поставщик настраиваемых ресурсов будет возвращать только поля в `properties` объекте для конечной точки кэша.

**Конечная точка** Ответ

``` HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "properties": {
        "myProperty1": "myPropertyValue1",
        "myProperty2": {
            "myProperty3" : "myPropertyValue3"
        }
    }
}
```

`name`Поля, `id` и `type` будут автоматически созданы настраиваемым поставщиком ресурсов для настраиваемого ресурса.

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

Аналогичным образом ответ от **конечной точки** затем перенаправляется обратно клиенту. Ответ от конечной точки должен вернуть:

- Допустимый документ объекта JSON. Все массивы и строки должны быть вложены в верхний объект.
- `Content-Type`Для заголовка необходимо задать значение Application/JSON; charset = UTF-8 ".
- Поставщик настраиваемых ресурсов Azure удаляет элемент из своего кэша, только если возвращается ответ уровня 200. Даже если ресурс не существует, **Конечная точка** должна вернуть 204.

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

Запрос **не** будет перенаправлен в **конечную точку**.

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

Этот запрос **не** будет перенаправляться в **конечную точку**.

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
- [Справочник: Справочник по прокси-службе настраиваемого ресурса](proxy-resource-endpoint-reference.md)
