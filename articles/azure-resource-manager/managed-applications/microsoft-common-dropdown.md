---
title: Элемент пользовательского интерфейса DropDown
description: Сведения об элементе пользовательского интерфейса Microsoft.Common.DropDown для портала Azure. Используйте для выбора из доступных параметров при развертывании управляемого приложения.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: tomfitz
ms.openlocfilehash: b9b27a432776635290c7e8e796e84d8c1e0e8675
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92168347"
---
# <a name="microsoftcommondropdown-ui-element"></a>Элемент пользовательского интерфейса Microsoft.Common.DropDown

Элемент управления для выбора с раскрывающимся списком. Можно разрешить выбор только одного элемента или нескольких элементов. При необходимости можно также включить описание с элементами.

## <a name="ui-sample"></a>Пример элемента пользовательского интерфейса

Элемент DropDown имеет различные параметры, которые определяют его внешний вид на портале.

Если для выбора разрешен только один элемент, элемент управления отображается как:

:::image type="content" source="./media/managed-application-elements/microsoft-common-dropdown-1.png" alt-text="Один выделенный фрагмент Microsoft. Common. DropDown":::

Если описания включены, элемент управления выглядит следующим образом:

:::image type="content" source="./media/managed-application-elements/microsoft-common-dropdown-2.png" alt-text="Один выделенный фрагмент Microsoft. Common. DropDown с описаниями":::

Если включен множественный выбор, то элемент управления добавляет параметр **выбрать все** и флажки для выбора более одного элемента:

:::image type="content" source="./media/managed-application-elements/microsoft-common-dropdown-3.png" alt-text="Множественный выбор Microsoft. Common. DropDown":::

Описания могут включаться при включенном множественном выборе.

:::image type="content" source="./media/managed-application-elements/microsoft-common-dropdown-4.png" alt-text="Снимок экрана, показывающий, как можно включить несколько описаний при включенном множественном выборе":::

Если фильтрация включена, элемент управления включает текстовое поле для добавления значения фильтрации.

:::image type="content" source="./media/managed-application-elements/microsoft-common-dropdown-5.png" alt-text="Множественный выбор с описанием Microsoft. Common. DropDown":::

## <a name="schema"></a>схема

```json
{
    "name": "element1",
    "type": "Microsoft.Common.DropDown",
    "label": "Example drop down",
    "placeholder": "",
    "defaultValue": "Value two",
    "toolTip": "",
    "multiselect": true,  
    "selectAll": true,  
    "filter": true,  
    "filterPlaceholder": "Filter items ...",  
    "multiLine": true,  
    "defaultDescription": "A value for selection",  
    "constraints": {
        "allowedValues": [
            {
                "label": "Value one",
                "description": "The value to select for option 1.",
                "value": "one"
            },
            {
                "label": "Value two",
                "description": "The value to select for option 2.",
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

## <a name="remarks"></a>Remarks

- Используйте `multiselect` , чтобы указать, могут ли пользователи выбирать более одного элемента.
- По умолчанию `selectAll` , `true` Если включен множественный выбор.
- `filter`Свойство позволяет пользователям выполнять поиск в длинном списке параметров.
- Метка для `constraints.allowedValues` — это отображаемый текст элемента. Его значение — это выходное значение при выборе элемента.
- Если указано, значение по умолчанию должно быть меткой в `constraints.allowedValues`. Если не указано, выбирается первый элемент в `constraints.allowedValues`. Значение по умолчанию — **null**.
- В `constraints.allowedValues` должен содержаться по крайней мере один элемент.
- Чтобы эмулировать значение, которое не требуется, добавьте элемент с меткой и значением `""` (пустая строка) в `constraints.allowedValues`.
- `defaultDescription`Свойство используется для элементов, у которых нет описания.
- `placeholder`Свойство — это текст справки, который исчезает, когда пользователь начинает редактирование. Если `placeholder` определены и `defaultValue` , то `defaultValue` имеет приоритет и отображается.

## <a name="next-steps"></a>Дальнейшие действия

* Общие сведения о создании определений пользовательского интерфейса см. в статье [Начало работы с CreateUiDefinition](create-uidefinition-overview.md).
* Дополнительные сведения об общих свойствах элементов пользовательского интерфейса см. в статье [Элементы CreateUiDefinition](create-uidefinition-elements.md).
