---
title: Добавление настраиваемых ресурсов в Azure REST API
description: Узнайте, как добавить настраиваемые ресурсы в REST API Azure. В этой статье рассматриваются требования и рекомендации для конечных точек, которые хотят реализовать пользовательские ресурсы.
ms.topic: conceptual
ms.author: jobreen
author: jjbfour
ms.date: 06/20/2019
ms.openlocfilehash: b6c5f5b8e437ad2dc2e8a3be3f3f2ed03a613b44
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "75650530"
---
# <a name="adding-custom-resources-to-azure-rest-api"></a>Добавление настраиваемых ресурсов в Azure REST API

В этой статье рассматриваются требования и рекомендации по созданию конечных точек поставщика настраиваемых ресурсов Azure, которые реализуют настраиваемые ресурсы. Если вы не знакомы с поставщиками настраиваемых ресурсов Azure, ознакомьтесь [с обзором пользовательских поставщиков ресурсов](overview.md).

## <a name="how-to-define-a-resource-endpoint"></a>Определение конечной точки ресурса

**Конечная точка** — это URL-адрес, указывающий на службу, которая реализует базовый контракт между ней и Azure. Конечная точка определена в поставщике настраиваемых ресурсов и может быть любым общедоступным URL-адресом. В примере ниже определено значение типа **ResourceType** `myCustomResource` , реализованное в `endpointURL` .

Пример **ResourceProvider**:

```JSON
{
  "properties": {
    "resourceTypes": [
      {
        "name": "myCustomResource",
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

## <a name="building-a-resource-endpoint"></a>Создание конечной точки ресурса

**Конечная точка** , реализующая **ResourceType** , должна отреагировать на запрос и ответ для нового API в Azure. При создании пользовательского поставщика ресурсов с типом **ресурса** он создаст новый набор API в Azure. В этом случае **ResourceType** создает новый API ресурсов Azure для `PUT` , `GET` и `DELETE` для выполнения CRUD в одном ресурсе, а также `GET` для получения всех существующих ресурсов:

Управлять одним ресурсом ( `PUT` , `GET` и `DELETE` ):

``` JSON
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResource/{myCustomResourceName}
```

Получить все ресурсы ( `GET` ):

``` JSON
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResource
```

Для настраиваемых ресурсов поставщики пользовательских ресурсов предлагают два типа **раутингтипес**: " `Proxy` " и " `Proxy, Cache` ".

### <a name="proxy-routing-type"></a>тип маршрутизации прокси-сервера

" `Proxy` " **Раутингтипе** прокси-серверы всех методов запроса к **конечной точке** , указанной в настраиваемом поставщике ресурсов. Когда следует использовать " `Proxy` ":

- Требуется полный контроль над ответом.
- Интеграция систем с существующими ресурсами.

Дополнительные сведения о `Proxy` ресурсах см. [в справочнике по пользовательскому прокси-службе ресурсов](proxy-resource-endpoint-reference.md) .

### <a name="proxy-cache-routing-type"></a>тип маршрутизации кэша прокси-сервера

`Proxy, Cache`Только прокси-серверы **раутингтипе** `PUT` и `DELETE` запрашивают методы в **конечной точке** , указанной в настраиваемом поставщике ресурсов. Настраиваемый поставщик ресурсов автоматически возвращает `GET` запросы в зависимости от того, что он хранил в своем кэше. Если настраиваемый ресурс помечен как кэш, настраиваемый поставщик ресурсов также будет добавлять и перезаписывать поля в ответе, чтобы сделать API-интерфейсы совместимыми с Azure. Когда следует использовать " `Proxy, Cache` ":

- Создание новой системы, не имеющей существующих ресурсов.
- Работа с существующей экосистемой Azure.

Дополнительные сведения о `Proxy, Cache` ресурсах см. [в справочнике по пользовательскому кэшу ресурсов](proxy-cache-resource-endpoint-reference.md) .

## <a name="creating-a-custom-resource"></a>Создание настраиваемого ресурса

Существует два основных способа создания пользовательского ресурса из пользовательского поставщика ресурсов.

- Azure CLI
- Шаблоны Azure Resource Manager

### <a name="azure-cli"></a>Azure CLI

Создание настраиваемого ресурса:

```azurecli-interactive
az resource create --is-full-object \
                   --id /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/{resourceTypeName}/{customResourceName} \
                   --properties \
                    '{
                        "location": "eastus",
                        "properties": {
                            "myProperty1": "myPropertyValue1",
                            "myProperty2": {
                                "myProperty3": "myPropertyValue3"
                            }
                        }
                    }'
```

Параметр | Обязательно | Описание
---|---|---
full-object | *Да* | Указывает, что объект свойств включает в себя другие параметры, такие как местоположение, теги, sku и/или план.
идентификатор | *Да* | Идентификатор ресурса настраиваемого ресурса. Оно должно существовать вне **ResourceProvider**
properties | *Да* | Текст запроса, который будет отправлен в **конечную точку**.

Удаление настраиваемого ресурса Azure:

```azurecli-interactive
az resource delete --id /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/{resourceTypeName}/{customResourceName}
```

Параметр | Обязательно | Описание
---|---|---
идентификатор | *Да* | Идентификатор ресурса настраиваемого ресурса. Он должен существовать вне **ResourceProvider**.

Получение настраиваемого ресурса Azure:

```azurecli-interactive
az resource show --id /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/{resourceTypeName}/{customResourceName}
```

Параметр | Обязательно | Описание
---|---|---
идентификатор | *Да* | Идентификатор ресурса настраиваемого ресурса. Оно должно существовать вне **ResourceProvider**

### <a name="azure-resource-manager-template"></a>Шаблон Azure Resource Manager

> [!NOTE]
> Для ресурсов требуется, чтобы ответ содержал соответствующую `id` `name` `type` **конечную точку**, и.

Azure Resource Manager шаблонах требуется, чтобы, `id` `name` и `type` были правильно возвращены из подчиненной конечной точки. Возвращенный ответ ресурса должен иметь вид:

Пример ответа **конечной точки** :

``` JSON
{
  "properties": {
    "myProperty1": "myPropertyValue1",
    "myProperty2": {
        "myProperty3": "myPropertyValue3"
    }
  },
  "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{customResourceName}",
  "name": "{customResourceName}",
  "type": "Microsoft.CustomProviders/resourceProviders/{resourceTypeName}"
}
```

Пример шаблона Диспетчера ресурсов Azure:

```JSON
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        {
            "type": "Microsoft.CustomProviders/resourceProviders/{resourceTypeName}",
            "name": "{resourceProviderName}/{customResourceName}",
            "apiVersion": "2018-09-01-preview",
            "location": "eastus",
            "properties": {
                "myProperty1": "myPropertyValue1",
                "myProperty2": {
                    "myProperty3": "myPropertyValue3"
                }
            }
        }
    ]
}
```

Параметр | Обязательно | Описание
---|---|---
resourceTypeName | *Да* | **Имя** типа **ресурса** , определенного в настраиваемом поставщике.
resourceProviderName | *Да* | Имя экземпляра поставщика настраиваемых ресурсов.
customResourceName | *Да* | Имя настраиваемого ресурса.

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения о поставщиках настраиваемых ресурсов Azure](overview.md)
- [Краткое руководство. Создание настраиваемого поставщика ресурсов Azure и развертывание настраиваемых ресурсов](./create-custom-provider.md)
- [Руководство. Создание настраиваемых действий и ресурсов в Azure](./tutorial-get-started-with-custom-providers.md)
- [Как добавлять настраиваемые действия в Azure REST API](./custom-providers-action-endpoint-how-to.md)
- [Справочник: Справочник по прокси-службе настраиваемого ресурса](proxy-resource-endpoint-reference.md)
- [Справочник: Справочник по пользовательскому кэшу ресурсов](proxy-cache-resource-endpoint-reference.md)
