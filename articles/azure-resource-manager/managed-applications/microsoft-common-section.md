---
title: Элемент пользовательского интерфейса раздела
description: Сведения об элементе пользовательского интерфейса Microsoft.Common.Section для портала Azure. Используйте для группировки элементов на портале для развертывания управляемых приложений.
author: tfitzmac
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: tomfitz
ms.openlocfilehash: 924aff8f2ba3d796b65f52494845f3b10018065c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87063956"
---
# <a name="microsoftcommonsection-ui-element"></a>Элемент пользовательского интерфейса Microsoft.Common.Section

Элемент управления, группирующий один или несколько элементов под заголовком.

## <a name="ui-sample"></a>Пример элемента пользовательского интерфейса

![Элемент пользовательского интерфейса Microsoft.Common.Section](./media/managed-application-elements/microsoft-common-section.png)

## <a name="schema"></a>схема

```json
{
  "name": "section1",
  "type": "Microsoft.Common.Section",
  "label": "Example section",
  "elements": [
    {
      "name": "text1",
      "type": "Microsoft.Common.TextBox",
      "label": "Example text box 1"
    },
    {
      "name": "text2",
      "type": "Microsoft.Common.TextBox",
      "label": "Example text box 2"
    }
  ],
  "visible": true
}
```

## <a name="remarks"></a>Комментарии

- Параметр `elements` должен содержать по крайней мере один элемент. Этот параметр может содержать элементы любого типа, кроме `Microsoft.Common.Section`.
- Этот элемент не поддерживает свойство `toolTip`.

## <a name="sample-output"></a>Пример выходных данных
Для доступа к выходным значениям элементов в `elements` используйте функции [basics()](create-ui-definition-referencing-functions.md#basics) или [steps()](create-ui-definition-referencing-functions.md#steps) и точечную нотацию:

```json
steps('configuration').section1.text1
```

Элементы типа `Microsoft.Common.Section` не содержат выходные значения.

## <a name="next-steps"></a>Дальнейшие действия

* Общие сведения о создании определений пользовательского интерфейса см. в статье [Начало работы с CreateUiDefinition](create-uidefinition-overview.md).
* Дополнительные сведения об общих свойствах элементов пользовательского интерфейса см. в статье [Элементы CreateUiDefinition](create-uidefinition-elements.md).
