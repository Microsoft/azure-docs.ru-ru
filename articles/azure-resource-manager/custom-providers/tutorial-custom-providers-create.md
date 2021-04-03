---
title: Создание и использование настраиваемого поставщика
description: В этом руководстве показано, как создать и использовать настраиваемого поставщика Azure. Используйте настраиваемые поставщики для изменения рабочих процессов в Azure.
author: jjbfour
ms.topic: tutorial
ms.date: 06/19/2019
ms.author: jobreen
ms.custom: devx-track-azurecli
ms.openlocfilehash: 4f425af7681b666b42fbcc70ac0e4c31d9df6d49
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "87503758"
---
# <a name="create-and-use-a-custom-provider"></a>Создание и использование настраиваемого поставщика

Настраиваемым поставщиком называется контракт между Azure и конечной точкой. Настраиваемые поставщики позволяют изменять рабочие процессы в Azure. В этом руководстве описано создание настраиваемого поставщика. Если вы еще не знакомы с настраиваемыми поставщиками Azure, перейдите к этой [обзорной статье](overview.md).

## <a name="create-a-custom-provider"></a>Создание настраиваемого поставщика

> [!NOTE]
> Это руководство не затрагивает процесс создания конечной точки. Если у вас нет конечной точки RESTful, выполните задачи из [руководства по созданию конечных точек RESTful](./tutorial-custom-providers-function-authoring.md). На этом руководстве основан текущий документ.

После создания конечной точки вы можете создать настраиваемый поставщик, который представляет договор между поставщиком и конечной точкой. С помощью настраиваемого поставщика можно указывать список определений конечных точек.

```JSON
{
  "name": "myEndpointDefinition",
  "routingType": "Proxy",
  "endpoint": "https://<yourapp>.azurewebsites.net/api/<funcname>?code=<functionkey>"
}
```

Свойство | Обязательно | Описание
---|---|---
**name** | Да | Имя определения конечной точки. Azure предоставляет это имя через свой API-интерфейс в разделе "/subscriptions/{ИД_подписки}/resourceGroups/{Имя_группы_ресурсов}/providers/Microsoft.CustomProviders/<br>/resourceProviders/{имя_поставщика_ресурсов}/{имя_определения_конечной_точки}
**routingType** | Нет | Тип контракта конечной точки. Если это значение не указано, по умолчанию используется тип Proxy (Прокси-сервер).
**endpoint** | Да | Конечная точка для направления запросов. Эта конечная точка обрабатывает ответ и все аспекты, на которые влияет запрос.

Параметр **endpoint** (конечная точка) содержит URL-адрес триггера для приложения-функции Azure. Замените заполнители `<yourapp>`, `<funcname>` и `<functionkey>` реальными значениями для созданного приложения-функции.

## <a name="define-custom-actions-and-resources"></a>Определение настраиваемых действий и ресурсов

Настраиваемый поставщик содержит список определений конечных точек, основанных на свойствах **actions** и **resourceTypes**. Свойство **actions** сопоставляется с настраиваемыми действиями, которые предоставляет настраиваемый поставщик, а свойство **resourceTypes** описывает настраиваемые ресурсы. В нашем примере настраиваемый поставщик имеет свойство **actions** с именем `myCustomAction` и свойство **resourceTypes** с именем `myCustomResources`.

```JSON
{
  "properties": {
    "actions": [
      {
        "name": "myCustomAction",
        "routingType": "Proxy",
        "endpoint": "https://<yourapp>.azurewebsites.net/api/<funcname>?code=<functionkey>"
      }
    ],
    "resourceTypes": [
      {
        "name": "myCustomResources",
        "routingType": "Proxy",
        "endpoint": "https://<yourapp>.azurewebsites.net/api/<funcname>?code=<functionkey>"
      }
    ]
  },
  "location": "eastus"
}
```

## <a name="deploy-the-custom-provider"></a>Развертывание настраиваемого поставщика

> [!NOTE]
> Замените значения **endpoint** URL-адресом триггера из приложения-функции, созданной в предыдущем руководстве.

Вы можете развернуть описанный выше настраиваемый поставщик с помощью шаблона Azure Resource Manager следующим образом:

```JSON
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        {
            "type": "Microsoft.CustomProviders/resourceProviders",
            "name": "myCustomProvider",
            "apiVersion": "2018-09-01-preview",
            "location": "eastus",
            "properties": {
                "actions": [
                    {
                        "name": "myCustomAction",
                        "routingType": "Proxy",
                        "endpoint": "https://<yourapp>.azurewebsites.net/api/<funcname>?code=<functionkey>"
                    }
                ],
                "resourceTypes": [
                    {
                        "name": "myCustomResources",
                        "routingType": "Proxy",
                        "endpoint": "https://<yourapp>.azurewebsites.net/api/<funcname>?code=<functionkey>"
                    }
                ]
            }
        }
    ]
}
```

## <a name="use-custom-actions-and-resources"></a>Использование настраиваемых действий и ресурсов

После создания настраиваемого поставщика вы можете использовать новые API-интерфейсы Azure. На следующих вкладках объясняется, как вызвать и использовать настраиваемый поставщик.

### <a name="custom-actions"></a>Настраиваемые действия

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

> [!NOTE]
> Замените заполнители `{subscriptionId}` и `{resourceGroupName}` именами подписки и группы ресурсов, где развернут настраиваемый поставщик.

```azurecli-interactive
az resource invoke-action --action myCustomAction \
                          --ids /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/myCustomProvider \
                          --request-body
                            '{
                                "hello": "world"
                            }'
```

Параметр | Обязательно | Описание
---|---|---
*action* | Да | Имя действия, определенного в настраиваемом поставщике.
*ids* | Да | Идентификатор ресурса настраиваемого поставщика.
*request-body* | Нет | Текст запроса, который будет отправляться в конечную точку.

# <a name="template"></a>[Шаблон](#tab/template)

Отсутствует.

---

### <a name="custom-resources"></a>Настраиваемые ресурсы

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

> [!NOTE]
> Замените заполнители `{subscriptionId}` и `{resourceGroupName}` именами подписки и группы ресурсов, где развернут настраиваемый поставщик.

#### <a name="create-a-custom-resource"></a>Создание настраиваемого ресурса

```azurecli-interactive
az resource create --is-full-object \
                   --id /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/myCustomProvider/myCustomResources/myTestResourceName1 \
                   --properties
                    '{
                        "location": "eastus",
                        "properties": {
                            "hello" : "world"
                        }
                    }'
```

Параметр | Обязательно | Описание
---|---|---
*full-object* | Да | Указывает, включает ли объект свойств другие параметры, такие как расположение, теги, номер SKU и (или) план.
*id* | Да | Идентификатор ресурса настраиваемого ресурса. Этот идентификатор — расширение идентификатора ресурса для настраиваемого поставщика.
*properties* | Да | Текст запроса, который будет отправляться в конечную точку.

#### <a name="delete-a-custom-resource"></a>Удаление настраиваемого ресурса

```azurecli-interactive
az resource delete --id /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/myCustomProvider/myCustomResources/myTestResourceName1
```

Параметр | Обязательно | Описание
---|---|---
*идентификатор* | Да | Идентификатор ресурса настраиваемого ресурса. Этот идентификатор — расширение идентификатора ресурса для настраиваемого поставщика.

#### <a name="retrieve-a-custom-resource"></a>Извлечение настраиваемого ресурса

```azurecli-interactive
az resource show --id /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/myCustomProvider/myCustomResources/myTestResourceName1
```

Параметр | Обязательно | Описание
---|---|---
*идентификатор* | Да | Идентификатор ресурса настраиваемого ресурса. Этот идентификатор — расширение идентификатора ресурса для настраиваемого поставщика.

# <a name="template"></a>[Шаблон](#tab/template)

Пример шаблона Resource Manager

```JSON
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        {
            "type": "Microsoft.CustomProviders/resourceProviders/myCustomResources",
            "name": "myCustomProvider/myTestResourceName1",
            "apiVersion": "2018-09-01-preview",
            "location": "eastus",
            "properties": {
                "hello": "world"
            }
        }
    ]
}
```

Параметр | Обязательно | Описание
---|---|---
*resourceTypeName* | Да | Значение `name` из свойства **resourceType**, которое определено в настраиваемом поставщике.
*resourceProviderName* | Да | Имя экземпляра настраиваемого поставщика.
*customResourceName* | Да | Имя настраиваемого ресурса.

---

> [!NOTE]
> Завершив развертывание и использование настраиваемого поставщика, не забудьте удалить все созданные ресурсы, включая приложение-функцию Azure.

## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали о настраиваемых поставщиках. Дополнительные сведения см. в разделе:

- [Руководство. Добавление настраиваемых действий в Azure REST API](./custom-providers-action-endpoint-how-to.md)
- [Руководство. Добавление настраиваемых ресурсов в Azure REST API](./custom-providers-resources-endpoint-how-to.md)
