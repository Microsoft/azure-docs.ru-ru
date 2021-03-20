---
title: Элемент пользовательского интерфейса Ресаурцеселектор
description: Описывает элемент пользовательского интерфейса Microsoft. Solutions. Ресаурцеселектор для портал Azure. Используется для получения списка существующих ресурсов.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: tomfitz
ms.openlocfilehash: d4b21f092733f6ebb3c2bf5aa06c1d2782323730
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87099059"
---
# <a name="microsoftsolutionsresourceselector-ui-element"></a>Элемент пользовательского интерфейса Microsoft. Solutions. Ресаурцеселектор

Ресаурцеселектор позволяет пользователям выбрать существующий ресурс из подписки.

## <a name="ui-sample"></a>Пример элемента пользовательского интерфейса

![Microsoft. Solutions. Ресаурцеселектор](./media/managed-application-elements/microsoft-solutions-resourceselector.png)

## <a name="schema"></a>схема

```json
{
    "name": "storageSelector",
    "type": "Microsoft.Solutions.ResourceSelector",
    "label": "Select storage accounts",
    "resourceType": "Microsoft.Storage/storageAccounts",
    "options": {
        "filter": {
            "subscription": "onBasics",
            "location": "onBasics"
        }
    }
}
```

## <a name="sample-output"></a>Пример выходных данных

```json
"name": "{resource-name}",
"id": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/{resource-provider-namespace}/{resource-type}/{resource-name}",
"location": "{deployed-location}"
```

## <a name="remarks"></a>Комментарии

В `resourceType` свойстве укажите пространство имен поставщика ресурсов и имя типа ресурса для ресурса, который вы хотите отобразить в списке.

`filter`Свойство ограничивают доступные параметры для ресурсов. Результаты можно ограничить по расположению или подписке. Чтобы отобразить только ресурсы, соответствующие выделенному фрагменту в области основные сведения, используйте `onBasics` . Чтобы отобразить все ресурсы, используйте `all` . Значение по умолчанию — `all`.

## <a name="next-steps"></a>Дальнейшие действия

* Общие сведения о создании определений пользовательского интерфейса см. в статье [Начало работы с CreateUiDefinition](create-uidefinition-overview.md).
* Дополнительные сведения об общих свойствах элементов пользовательского интерфейса см. в статье [Элементы CreateUiDefinition](create-uidefinition-elements.md).
