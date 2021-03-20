---
title: Элемент пользовательского интерфейса Едитаблегрид
description: Описывает элемент пользовательского интерфейса Microsoft. Common. Едитаблегрид для портал Azure. Позволяет пользователям собирать табличные данные.
author: tfitzmac
ms.topic: conceptual
ms.date: 08/24/2020
ms.author: tomfitz
ms.openlocfilehash: 04f86883a75110985d1cbe050fe3fd3e0582986a
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88893817"
---
# <a name="microsoftcommoneditablegrid-ui-element"></a>Элемент пользовательского интерфейса Microsoft. Common. Едитаблегрид

Элемент управления для сбора табличных входных данных. Все поля в сетке доступны для редактирования, а количество строк может меняться.

## <a name="ui-sample"></a>Пример элемента пользовательского интерфейса

:::image type="content" source="./media/managed-application-elements/microsoft-common-editablegrid.png" alt-text="Microsoft. Common. Едитаблегрид":::

## <a name="schema"></a>схема

```json
{
  "name": "people",
  "type": "Microsoft.Common.EditableGrid",
  "ariaLabel": "Enter information per person",
  "label": "People",
  "constraints": {
    "width": "Full",
    "rows": {
      "count": {
        "min": 1,
        "max": 10
      }
    },
    "columns": [
      {
        "id": "colName",
        "header": "Name",
        "width": "1fr",
        "element": {
          "type": "Microsoft.Common.TextBox",
          "placeholder": "Full name",
          "constraints": {
            "required": true,
            "validations": [
              {
                "isValid": "[startsWith(last(take(steps('grid').people, $rowIndex)).colName, 'contoso')]",
                "message": "Must start with 'contoso'."
              },
              {
                "regex": "^[a-z0-9A-Z]{1,30}$",
                "message": "Only alphanumeric characters are allowed, and the value must be 1-30 characters long."
              }
            ]
          }
        }
      },
      {
        "id": "colGender",
        "header": "Gender",
        "width": "1fr",
        "element": {
          "name": "dropDown1",
          "type": "Microsoft.Common.DropDown",
          "placeholder": "Select a gender...",
          "constraints": {
            "allowedValues": [
              {
                "label": "Female",
                "value": "female"
              },
              {
                "label": "Male",
                "value": "male"
              },
              {
                "label": "Other",
                "value": "other"
              }
            ],
            "required": true
          }
        }
      },
      {
        "id": "colContactPreference",
        "header": "Contact preference",
        "width": "1fr",
        "element": {
          "type": "Microsoft.Common.OptionsGroup",
          "constraints": {
            "allowedValues": [
              {
                "label": "Email",
                "value": "email"
              },
              {
                "label": "Text",
                "value": "text"
              }
            ],
            "required": true
          }
        }
      }
    ]
  }
}
```

## <a name="sample-output"></a>Пример выходных данных

```json
{
  "colName": "contoso",
  "colGender": "female",
  "colContactPreference": "email"
}
```

## <a name="remarks"></a>Комментарии

- Единственными допустимыми элементами управления в массиве Columns являются [TextBox](microsoft-common-textbox.md), [OptionsGroup](microsoft-common-optionsgroup.md)и [dropdown](microsoft-common-dropdown.md).
- `$rowIndex`Переменная допустима только в выражениях, содержащихся в дочерних столбцах сетки. Это целое число, представляющее индекс относительной строки элемента, а отсчет начинается с единицы и увеличивается на единицу. Как показано в `"columns":` разделе схемы, `$rowIndex` используется для проверки.
- При выполнении проверок с помощью `$rowIndex` переменной можно получить значение текущей строки, объединив `last()` `take()` команды и.

  Пример:

  `last(take(<reference_to_grid>, $rowIndex))`

- `label`Свойство не отображается как часть элемента управления, но отображается на последней вкладке Сводка.
- `ariaLabel`Свойство является меткой доступности для сетки. Укажите полезный текст для пользователей, использующих средства чтения с экрана.
- `constraints.width`Свойство используется для задания общей ширины сетки. Параметры _заполнены_, _средние_, _малые_. Значение по умолчанию — _Full_.
- `width`Свойство для дочерних столбцов определяет ширину столбца. Ширина задается с помощью дробных единиц, таких как _3FR_, и общего пространства, выделенного для столбцов, пропорциональных их единицам. Если ширина столбца не указана, по умолчанию используется значение _1fr_.

## <a name="next-steps"></a>Дальнейшие действия

- Общие сведения о создании определений пользовательского интерфейса см. в статье [Начало работы с CreateUiDefinition](create-uidefinition-overview.md).
- Дополнительные сведения об общих свойствах элементов пользовательского интерфейса см. в статье [Элементы CreateUiDefinition](create-uidefinition-elements.md).
