---
title: Элемент пользовательского интерфейса UserNameTextBox
description: Сведения об элементе пользовательского интерфейса Microsoft.Common.UserNameTextBox для портала Azure. Позволяет пользователям указывать имена пользователей Windows или Linux.
author: tfitzmac
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: tomfitz
ms.openlocfilehash: 82478f322e1df22bde50769b90f0424140920e9a
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87063597"
---
# <a name="microsoftcomputeusernametextbox-ui-element"></a>Элемент пользовательского интерфейса Microsoft.Compute.UserNameTextBox

Элемент управления "Текстовое поле" со встроенной проверкой имен пользователей Windows и Linux.

## <a name="ui-sample"></a>Пример элемента пользовательского интерфейса

![Элемент пользовательского интерфейса Microsoft.Compute.UserNameTextBox](./media/managed-application-elements/microsoft-compute-usernametextbox.png)

## <a name="schema"></a>схема

```json
{
  "name": "element1",
  "type": "Microsoft.Compute.UserNameTextBox",
  "label": "User name",
  "defaultValue": "",
  "toolTip": "",
  "constraints": {
    "required": true,
    "regex": "^[a-z0-9A-Z]{1,30}$",
    "validationMessage": "Only alphanumeric characters are allowed, and the value must be 1-30 characters long."
  },
  "osPlatform": "Windows",
  "visible": true
}
```

## <a name="sample-output"></a>Пример выходных данных

```json
"Example name"
```

## <a name="remarks"></a>Комментарии

- Если для параметра `constraints.required` задано значение **true**, то текстовое поле должно содержать значение, чтобы пройти проверку. Значение по умолчанию — **true**
- Необходимо задать значение для параметра `osPlatform` (**Windows** или **Linux**).
- `constraints.regex` — это шаблон регулярного выражения JavaScript. Если параметр указан, значение текстового поля должно соответствовать шаблону, чтобы пройти проверку. Значение по умолчанию — **null**.
- `constraints.validationMessage` — это строка, которая отображается, когда значение текстового поля не проходит проверку, указанную в `constraints.regex`. Если параметр не указан, используются встроенные сообщения проверки текстового поля. Значение по умолчанию — **null**.
- Этот элемент содержит встроенную проверку, которая основана на значении, заданном для параметра `osPlatform`. Встроенная проверка может использоваться вместе с настраиваемым регулярным выражением. Если для параметра `constraints.regex` значение указано, активируются встроенные и пользовательские проверки.

## <a name="next-steps"></a>Дальнейшие действия

* Общие сведения о создании определений пользовательского интерфейса см. в статье [Начало работы с CreateUiDefinition](create-uidefinition-overview.md).
* Дополнительные сведения об общих свойствах элементов пользовательского интерфейса см. в статье [Элементы CreateUiDefinition](create-uidefinition-elements.md).
