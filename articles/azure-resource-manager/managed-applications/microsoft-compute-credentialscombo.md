---
title: Элемент пользовательского интерфейса CredentialsCombo
description: Сведения об элементе пользовательского интерфейса Microsoft.Compute.CredentialsCombo для портала Azure.
author: tfitzmac
ms.topic: conceptual
ms.date: 09/29/2018
ms.author: tomfitz
ms.openlocfilehash: 47c88e08e5d2eac09fbcd5b60a8ccd73b46c9616
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87063785"
---
# <a name="microsoftcomputecredentialscombo-ui-element"></a>Элемент пользовательского интерфейса Microsoft.Compute.CredentialsCombo

Группа элементов управления со встроенной проверкой паролей и открытых ключей SSH для Windows и Linux.

## <a name="ui-sample"></a>Пример элемента пользовательского интерфейса

Для пользователей Windows:

![Microsoft.Compute.CredentialsCombo для Windows](./media/managed-application-elements/microsoft-compute-credentialscombo-windows.png)

Для пользователей Linux с выбранным паролем:

![Microsoft.Compute.CredentialsCombo для пароля Linux](./media/managed-application-elements/microsoft-compute-credentialscombo-linux-password.png)

Для пользователей Linux с выбранным открытым ключом SSH:

![Microsoft.Compute.CredentialsCombo для ключа Linux](./media/managed-application-elements/microsoft-compute-credentialscombo-linux-key.png)

## <a name="schema"></a>схема

Если вы работаете с Windows, используйте следующую схему:

```json
{
  "name": "element1",
  "type": "Microsoft.Compute.CredentialsCombo",
  "label": {
    "password": "Password",
    "confirmPassword": "Confirm password"
  },
  "toolTip": {
    "password": ""
  },
  "constraints": {
    "required": true,
    "customPasswordRegex": "^(?=.*[A-Za-z])(?=.*\\d)[A-Za-z\\d]{12,}$",
    "customValidationMessage": "The password must be alphanumeric, contain at least 12 characters, and have at least 1 letter and 1 number."
  },
  "options": {
    "hideConfirmation": false
  },
  "osPlatform": "Windows",
  "visible": true
}
```

Если вы работаете с **Linux**, используйте следующую схему:

```json
{
  "name": "element1",
  "type": "Microsoft.Compute.CredentialsCombo",
  "label": {
    "authenticationType": "Authentication type",
    "password": "Password",
    "confirmPassword": "Confirm password",
    "sshPublicKey": "SSH public key"
  },
  "toolTip": {
    "authenticationType": "",
    "password": "",
    "sshPublicKey": ""
  },
  "constraints": {
    "required": true,
    "customPasswordRegex": "^(?=.*[A-Za-z])(?=.*\\d)[A-Za-z\\d]{12,}$",
    "customValidationMessage": "The password must be alphanumeric, contain at least 12 characters, and have at least 1 letter and 1 number."
  },
  "options": {
    "hideConfirmation": false,
    "hidePassword": false
  },
  "osPlatform": "Linux",
  "visible": true
}
```

## <a name="sample-output"></a>Пример выходных данных

Если параметр `osPlatform` имеет значение **Windows** или параметр `osPlatform` — значение **Linux** и пользователь указал пароль вместо открытого ключа SSH, элемент управления возвращает следующие выходные данные:

```json
{
  "authenticationType": "password",
  "password": "p4ssw0rddem0",
}
```

Если параметр `osPlatform` имеет значение **Linux** и пользователь указал открытый ключ SSH, элемент управления возвращает следующие выходные данные:

```json
{
  "authenticationType": "sshPublicKey",
  "sshPublicKey": "AAAAB3NzaC1yc2EAAAABIwAAAIEA1on8gxCGJJWSRT4uOrR13mUaUk0hRf4RzxSZ1zRbYYFw8pfGesIFoEuVth4HKyF8k1y4mRUnYHP1XNMNMJl1JcEArC2asV8sHf6zSPVffozZ5TT4SfsUu/iKy9lUcCfXzwre4WWZSXXcPff+EHtWshahu3WzBdnGxm5Xoi89zcE=",
}
```

## <a name="remarks"></a>Комментарии

- Необходимо задать значение для параметра `osPlatform` (**Windows** или **Linux**).
- Если для параметра `constraints.required` задано значение **true**, то текстовые поля для пароля и открытого ключа SSH должны содержать значения, чтобы пройти проверку. Значение по умолчанию — **true**
- Если для параметра `options.hideConfirmation` задано значение **true**, второе текстовое поле для подтверждения пароля скрыто. Значение по умолчанию — **false**.
- Если для параметра `options.hidePassword` задано значение **true**, возможность использования проверки пароля скрыта. Его можно использовать только тогда, когда параметр `osPlatform` имеет значение **Linux**. Значение по умолчанию — **false**.
- Дополнительные ограничения на разрешенные пароли можно реализовать с помощью свойства `customPasswordRegex`. Строка в `customValidationMessage` отображается, если пароль не прошел пользовательскую проверку. Значение по умолчанию для обоих свойств — **null**.

## <a name="next-steps"></a>Дальнейшие действия

* Общие сведения о создании определений пользовательского интерфейса см. в статье [Начало работы с CreateUiDefinition](create-uidefinition-overview.md).
* Дополнительные сведения об общих свойствах элементов пользовательского интерфейса см. в статье [Элементы CreateUiDefinition](create-uidefinition-elements.md).
