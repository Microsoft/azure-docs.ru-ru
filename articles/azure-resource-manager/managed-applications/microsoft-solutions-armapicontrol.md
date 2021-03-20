---
title: Элемент пользовательского интерфейса Армапиконтрол
description: Описывает элемент пользовательского интерфейса Microsoft. Solutions. Армапиконтрол для портал Azure. Используется для вызова операций API.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: tomfitz
ms.openlocfilehash: bbe36e072d10b81c421331b2212d8b161afd2693
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87098153"
---
# <a name="microsoftcommonarmapicontrol-ui-element"></a>Элемент пользовательского интерфейса Microsoft. Common. Армапиконтрол

Армапиконтрол позволяет получать результаты операции API Azure Resource Manager. Используйте результаты для заполнения динамического содержимого в других элементах управления.

## <a name="ui-sample"></a>Пример элемента пользовательского интерфейса

Отсутствует пользовательский интерфейс для этого элемента управления.

## <a name="schema"></a>схема

В следующем примере показана схема для этого элемента управления:

```json
{
    "name": "testApi",
    "type": "Microsoft.Solutions.ArmApiControl",
    "request": {
        "method": "{HTTP-method}",
        "path": "{path-for-the-URL}",
        "body": {
            "key1": "val1",
            "key2": "val2"
        }
    }
}
```

## <a name="sample-output"></a>Пример выходных данных

Выходные данные элемента управления не отображаются для пользователя. Вместо этого результат операции используется в других элементах управления.

## <a name="remarks"></a>Комментарии

- `request.method`Свойство указывает метод HTTP. `GET` `POST` Разрешены только или.
- `request.path`Свойство указывает относительный путь URL-адреса. Это может быть статический путь, или его можно создать динамически, обратившись к выходным значениям других элементов управления.
- Свойство `request.body` необязательное. Используйте его, чтобы указать тело JSON, отправляемое с запросом. Текст может быть статическим или динамически созданным путем обращения к выходным значениям из других элементов управления.

## <a name="example"></a>Пример

В следующем примере `providersApi` элемент вызывает API для получения массива объектов поставщика.

`allowedValues`Свойство `providersDropDown` элемента настраивается для получения имен поставщиков. Он отображает их в раскрывающемся списке.

```json
{
    "name": "providersApi",
    "type": "Microsoft.Solutions.ArmApiControl",
    "request": {
        "method": "GET",
        "path": "[concat(subscription().id, '/providers/Microsoft.Network/expressRouteServiceProviders?api-version=2019-02-01')]"
    }
},
{
    "name": "providerDropDown",
    "type": "Microsoft.Common.DropDown",
    "label": "Provider",
    "toolTip": "The provider that offers the express route connection.",
    "constraints": {
        "allowedValues": "[map(steps('settings').providersApi.value, (item) => parse(concat('{\"label\":\"', item.name, '\",\"value\":\"', item.name, '\"}')))]",
        "required": true
    },
    "visible": true
}
```

Пример использования Армапиконтрол для проверки доступности имени ресурса см. в разделе [Microsoft. Common. TextBox](microsoft-common-textbox.md).

## <a name="next-steps"></a>Дальнейшие действия

* Общие сведения о создании определений пользовательского интерфейса см. в статье [Начало работы с CreateUiDefinition](create-uidefinition-overview.md).
* Дополнительные сведения об общих свойствах элементов пользовательского интерфейса см. в статье [Элементы CreateUiDefinition](create-uidefinition-elements.md).
