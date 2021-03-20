---
title: Добавление настраиваемых действий в Azure REST API
description: Узнайте, как добавить настраиваемые действия в REST API Azure. В этой статье рассматриваются требования и рекомендации для конечных точек, которые хотят реализовать настраиваемые действия.
ms.topic: conceptual
ms.author: jobreen
author: jjbfour
ms.date: 06/20/2019
ms.openlocfilehash: 6110a7952b7c29609d2b98e135b61032aec3fa52
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "75650400"
---
# <a name="adding-custom-actions-to-azure-rest-api"></a>Добавление настраиваемых действий в Azure REST API

В этой статье рассматриваются требования и рекомендации по созданию конечных точек поставщика пользовательских ресурсов Azure, которые реализуют настраиваемые действия. Если вы не знакомы с поставщиками настраиваемых ресурсов Azure, ознакомьтесь [с обзором пользовательских поставщиков ресурсов](overview.md).

## <a name="how-to-define-an-action-endpoint"></a>Определение конечной точки действия

**Конечная точка** — это URL-адрес, указывающий на службу, которая реализует базовый контракт между ней и Azure. Конечная точка определена в поставщике настраиваемых ресурсов и может быть любым общедоступным URL-адресом. В примере ниже имеется **действие** , вызываемое `myCustomAction` реализацией `endpointURL` .

Пример **ResourceProvider**:

```JSON
{
  "properties": {
    "actions": [
      {
        "name": "myCustomAction",
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

## <a name="building-an-action-endpoint"></a>Создание конечной точки действия

**Конечная точка** , реализующая **действие** , должна отреагировать на запрос и ответ для нового API в Azure. При создании пользовательского поставщика ресурсов с **действием** он создает новый набор API в Azure. В этом случае действие создаст новый API действия Azure для `POST` вызовов:

``` JSON
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomAction
```

Входящий запрос API Azure:

``` HTTP
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomAction?api-version=2018-09-01-preview
Authorization: Bearer eyJ0e...
Content-Type: application/json

{
    "myProperty1": "myPropertyValue1",
    "myProperty2": {
        "myProperty3" : "myPropertyValue3"
    }
}
```

Затем этот запрос будет перенаправлен в **конечную точку** в форме:

``` HTTP
POST https://{endpointURL}/?api-version=2018-09-01-preview
Content-Type: application/json
X-MS-CustomProviders-RequestPath: /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomAction

{
    "myProperty1": "myPropertyValue1",
    "myProperty2": {
        "myProperty3" : "myPropertyValue3"
    }
}
```

Аналогичным образом ответ от **конечной точки** затем перенаправляется обратно клиенту. Ответ от конечной точки должен вернуть:

- Допустимый документ объекта JSON. Все массивы и строки должны быть вложены в верхний объект.
- `Content-Type`Для заголовка необходимо задать значение Application/JSON; charset = UTF-8 ".

``` HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "myProperty1": "myPropertyValue1",
    "myProperty2": {
        "myProperty3" : "myPropertyValue3"
    }
}
```

Ответ поставщика настраиваемых ресурсов Azure:

``` HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "myProperty1": "myPropertyValue1",
    "myProperty2": {
        "myProperty3" : "myPropertyValue3"
    }
}
```

## <a name="calling-a-custom-action"></a>Вызов настраиваемого действия

Существует два основных способа вызова пользовательского действия для пользовательского поставщика ресурсов.

- Azure CLI
- Шаблоны Azure Resource Manager

### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
az resource invoke-action --action {actionName} \
                          --ids /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName} \
                          --request-body \
                            '{
                                "myProperty1": "myPropertyValue1",
                                "myProperty2": {
                                    "myProperty3": "myPropertyValue3"
                                }
                            }'
```

Параметр | Обязательно | Описание
---|---|---
action | *Да* | Имя действия, определенного в **ResourceProvider**.
ids | *Да* | Идентификатор ресурса **ResourceProvider**.
request-body | *Нет* | Текст запроса, который будет отправлен в **конечную точку**.

### <a name="azure-resource-manager-template"></a>Шаблон Azure Resource Manager

> [!NOTE]
> Действия имеют ограниченную поддержку в шаблонах Azure Resource Manager. Чтобы действие вызывалось внутри шаблона, оно должно содержать [`list`](../templates/template-functions-resource.md#list) префикс в имени.

Пример **ResourceProvider** с действием List:

```JSON
{
  "properties": {
    "actions": [
      {
        "name": "listMyCustomAction",
        "routingType": "Proxy",
        "endpoint": "https://{endpointURL}/"
      }
    ]
  },
  "location": "eastus"
}
```

Пример шаблона Диспетчера ресурсов Azure:

``` JSON
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "variables": {
        "resourceIdentifier": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}",
        "apiVersion": "2018-09-01-preview",
        "functionValues": {
            "myProperty1": "myPropertyValue1",
            "myProperty2": {
                "myProperty3": "myPropertyValue3"
            }
        }
    },
    "resources": [],
    "outputs": {
        "myCustomActionOutput": {
            "type": "object",
            "value": "[listMyCustomAction(variables('resourceIdentifier'), variables('apiVersion'), variables('functionValues'))]"
        }
    }
}
```

Параметр | Обязательно | Описание
---|---|---
resourceIdentifier | *Да* | Идентификатор ресурса **ResourceProvider**.
версия_API | *Да* | Версия API среды выполнения ресурсов. Это всегда должно быть "2018-09-01-Preview".
functionValues | *Нет* | Текст запроса, который будет отправлен в **конечную точку**.

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения о поставщиках настраиваемых ресурсов Azure](overview.md)
- [Краткое руководство. Создание настраиваемого поставщика ресурсов Azure и развертывание настраиваемых ресурсов](./create-custom-provider.md)
- [Руководство. Создание настраиваемых действий и ресурсов в Azure](./tutorial-get-started-with-custom-providers.md)
- [Как добавлять пользовательские ресурсы в Azure REST API](./custom-providers-resources-endpoint-how-to.md)
