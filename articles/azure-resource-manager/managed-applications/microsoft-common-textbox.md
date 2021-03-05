---
title: Элемент пользовательского интерфейса TextBox
description: Сведения об элементе пользовательского интерфейса Microsoft.Common.TextBox для портала Azure. Используется для добавления неформатированного текста.
author: tfitzmac
ms.topic: conceptual
ms.date: 03/03/2021
ms.author: tomfitz
ms.openlocfilehash: 8e4cfcc7fe46c19bf1e58ee5eadf1e0bb8c7bd8e
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102124335"
---
# <a name="microsoftcommontextbox-ui-element"></a>Элемент пользовательского интерфейса Microsoft.Common.TextBox

Элемент пользовательского интерфейса, который можно использовать для добавления неформатированного текста. `Microsoft.Common.TextBox`Элемент является однострочным полем ввода, но поддерживает многострочный вход со `multiLine` свойством.

## <a name="ui-sample"></a>Пример элемента пользовательского интерфейса

`TextBox`Элемент использует однострочное или многострочное текстовое поле.

:::image type="content" source="media/managed-application-elements/microsoft-common-textbox.png" alt-text="Однострочное текстовое поле Microsoft. Common. TextBox.":::

:::image type="content" source="media/managed-application-elements/microsoft-common-textbox-multi-line.png" alt-text="Многострочное текстовое поле элемента Microsoft. Common. TextBox.":::

## <a name="schema"></a>схема

```json
{
  "name": "nameInstance",
  "type": "Microsoft.Common.TextBox",
  "label": "Name",
  "defaultValue": "contoso123",
  "toolTip": "Use only allowed characters",
  "placeholder": "",
  "multiLine": false,
  "constraints": {
    "required": true,
    "validations": [
      {
        "regex": "^[a-z0-9A-Z]{1,30}$",
        "message": "Only alphanumeric characters are allowed, and the value must be 1-30 characters long."
      },
      {
        "isValid": "[startsWith(steps('resourceConfig').nameInstance, 'contoso')]",
        "message": "Must start with 'contoso'."
      }
    ]
  },
  "visible": true
}
```

## <a name="sample-output"></a>Пример выходных данных

```json
"contoso123"
```

## <a name="remarks"></a>Комментарии

- Используйте `toolTip` свойство для отображения текста об элементе при наведении курсора мыши на информационный символ.
- `placeholder`Свойство — это текст справки, который исчезает, когда пользователь начинает редактирование. Если `placeholder` определены и `defaultValue` , то `defaultValue` имеет приоритет и отображается.
- `multiLine`Свойство является логическим `true` или `false` . Чтобы использовать многострочное текстовое поле, задайте для свойства значение `true` . Если многострочный текстовый ящик не требуется, установите свойство в значение `false` или исключите свойство. Для новых строк выходные данные JSON отображаются `\n` для перевода строки. Многострочное текстовое поле принимает `\r` символы возврата каретки (CR) и `\n` перевода строки (LF). Например, значение по умолчанию может содержать `\r\n` , чтобы указать CRLF.
- Если `constraints.required` для задано `true` значение, то для успешного проверки текстовое поле должно быть равно значению. Значение по умолчанию — `false`.
- Свойство представляет собой массив, в который `validations` добавляются условия для проверки значения, указанного в текстовом поле.
- `regex`Свойство является шаблоном регулярного выражения JavaScript. Если указано, значение текстового поля должно соответствовать шаблону для успешной проверки. Значение по умолчанию — `null`. Дополнительные сведения о синтаксисе Regex см. в разделе [краткий справочник по регулярным выражениям](/dotnet/standard/base-types/regular-expression-language-quick-reference).
- `isValid`Свойство содержит выражение, результатом которого является `true` или `false` . В рамках выражения определяется условие, определяющее, является ли текстовое поле допустимым.
- `message`Свойство является строкой, отображаемой, когда значение текстового поля не проходит проверку.
- Можно указать значение в параметре, `regex` Если `required` для задано `false` . В этом случае, чтобы пройти проверку, значение для текстового поля не требуется. Если указано, оно должно соответствовать шаблону регулярного выражения.

## <a name="examples"></a>Примеры

В примерах показано, как использовать однострочное текстовое поле и многострочное текстовое поле.

### <a name="single-line"></a>Одна строка

В следующем примере для проверки доступности имени ресурса используется текстовое поле с элементом управления [Microsoft. Solutions. армапиконтрол](microsoft-solutions-armapicontrol.md) .

```json
"basics": [
  {
    "name": "nameApi",
    "type": "Microsoft.Solutions.ArmApiControl",
    "request": {
      "method": "POST",
      "path": "[concat(subscription().id, '/providers/Microsoft.Storage/checkNameAvailability?api-version=2019-06-01')]",
      "body": "[parse(concat('{\"name\": \"', basics('txtStorageName'), '\", \"type\": \"Microsoft.Storage/storageAccounts\"}'))]"
    }
  },
  {
    "name": "txtStorageName",
    "type": "Microsoft.Common.TextBox",
    "label": "Storage account name",
    "constraints": {
      "validations": [
        {
          "isValid": "[not(equals(basics('nameApi').nameAvailable, false))]",
          "message": "[concat('Name unavailable: ', basics('txtStorageName'))]"
        }
      ]
    }
  }
]
```

### <a name="multi-line"></a>Несколько строк

В этом примере `multiLine` свойство используется для создания многострочного текстового поля с текстом заполнителя.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
  "handler": "Microsoft.Azure.CreateUIDef",
  "version": "0.1.2-preview",
  "parameters": {
    "basics": [
      {
        "name": "demoTextBox",
        "type": "Microsoft.Common.TextBox",
        "label": "Multi-line text box",
        "defaultValue": "",
        "toolTip": "Use 1-60 alphanumeric characters, hyphens, spaces, periods, and colons.",
        "placeholder": "This is a multi-line text box:\nLine 1.\nLine 2.\nLine 3.",
        "multiLine": true,
        "constraints": {
          "required": true,
          "validations": [
            {
              "regex": "^[a-z0-9A-Z -.:\n]{1,60}$",
              "message": "Only 1-60 alphanumeric characters, hyphens, spaces, periods, and colons are allowed."
            }
          ]
        },
        "visible": true
      }
    ],
    "steps": [],
    "outputs": {
      "textBox": "[basics('demoTextBox')]"
    }
  }
}
```

## <a name="next-steps"></a>Дальнейшие действия

- Общие сведения о создании определений пользовательского интерфейса см. [ в статьеCreateUiDefinition.jsо создании приложений управляемого приложения Azure](create-uidefinition-overview.md).
- Дополнительные сведения об общих свойствах элементов пользовательского интерфейса см. в статье [Элементы CreateUiDefinition](create-uidefinition-elements.md).
