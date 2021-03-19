---
title: Элемент пользовательского интерфейса OptionsGroup
description: Сведения об элементе пользовательского интерфейса Microsoft.Common.OptionsGroup для портала Azure. Позволяет пользователям выбирать доступные варианты при развертывании управляемого приложения.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/09/2020
ms.author: tomfitz
ms.openlocfilehash: aa73b4cbded98291a14792a7151df9fdfb885b53
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87004202"
---
# <a name="microsoftcommonoptionsgroup-ui-element"></a>Элемент пользовательского интерфейса Microsoft.Common.OptionsGroup

Элемент управления OptionsGroup позволяет пользователям выбрать один вариант из двух или более вариантов. Пользователь может выбрать только один вариант.

> [!NOTE]
> В прошлом этот элемент управления отображал параметры по горизонтали. Теперь элемент управления представляет параметры по вертикали в виде переключателей.

## <a name="ui-sample"></a>Пример элемента пользовательского интерфейса

:::image type="content" source="./media/managed-application-elements/microsoft-common-optionsgroup-2.png" alt-text="Элемент пользовательского интерфейса Microsoft.Common.OptionsGroup":::

## <a name="schema"></a>схема

```json
{
  "name": "element1",
  "type": "Microsoft.Common.OptionsGroup",
  "label": "Some options group",
  "defaultValue": "Value two",
  "toolTip": "",
  "constraints": {
    "allowedValues": [
      {
        "label": "Value one",
        "value": "one"
      },
      {
        "label": "Value two",
        "value": "two"
      }
    ],
    "required": true
  },
  "visible": true
}
```

## <a name="sample-output"></a>Пример выходных данных

```json
"two"
```

## <a name="remarks"></a>Комментарии

- Метка для `constraints.allowedValues` — это отображаемый текст элемента. Его значение — это выходное значение при выборе элемента.
- Если указано, значение по умолчанию должно быть меткой в `constraints.allowedValues`. Если не указано, по умолчанию выбирается первый элемент в `constraints.allowedValues`. Значение по умолчанию — **null**.
- В `constraints.allowedValues` должен содержаться по крайней мере один элемент.

## <a name="next-steps"></a>Дальнейшие действия

* Общие сведения о создании определений пользовательского интерфейса см. в статье [Начало работы с CreateUiDefinition](create-uidefinition-overview.md).
* Дополнительные сведения об общих свойствах элементов пользовательского интерфейса см. в статье [Элементы CreateUiDefinition](create-uidefinition-elements.md).
