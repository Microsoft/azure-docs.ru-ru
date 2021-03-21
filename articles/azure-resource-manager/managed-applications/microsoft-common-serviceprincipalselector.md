---
title: Элемент пользовательского интерфейса СервицепринЦипалселектор
description: Описывает элемент пользовательского интерфейса Microsoft. Common. СервицепринЦипалселектор для портал Azure. Предоставляет элемент управления для выбора приложения и текстового поля для ввода пароля или отпечатка сертификата.
author: tfitzmac
ms.topic: conceptual
ms.date: 11/17/2020
ms.author: tomfitz
ms.openlocfilehash: 2fdbbad467d8c762db485fc7935e9cef78313fd0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96184456"
---
# <a name="microsoftcommonserviceprincipalselector-ui-element"></a>Элемент пользовательского интерфейса Microsoft. Common. СервицепринЦипалселектор

Элемент управления, позволяющий пользователям выбрать существующий [субъект-службу](../../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) или зарегистрировать новое приложение. При выборе **создать новый** необходимо выполнить действия по регистрации нового приложения. При выборе существующего приложения элемент управления предоставляет текстовое поле для ввода пароля или отпечатка сертификата.

## <a name="ui-samples"></a>Примеры пользовательского интерфейса

Можно использовать приложение по умолчанию, создать новое приложение или использовать существующее приложение.

### <a name="use-default-application-or-create-new"></a>Используйте приложение по умолчанию или создайте новое

Представление по умолчанию определяется значениями в `defaultValue` свойстве, а для **типа субъекта-службы** — значение **создать новый**. Если `principalId` свойство содержит допустимый глобальный уникальный идентификатор (GUID), элемент управления ищет приложение `objectId` . Значение по умолчанию применяется, если пользователь не делает выбор из элемента управления.

Если требуется зарегистрировать новое приложение, нажмите кнопку **изменить выбор** и отобразится диалоговое окно **Регистрация приложения** . Введите **имя**, **поддерживаемый тип учетной записи** и нажмите кнопку **зарегистрировать** .

:::image type="content" source="./media/managed-application-elements/microsoft-common-serviceprincipal-default.png" alt-text="Начальное представление Microsoft. Common. СервицепринЦипалселектор.":::

После регистрации нового приложения используйте **тип проверки подлинности** для ввода пароля или отпечатка сертификата.

:::image type="content" source="./media/managed-application-elements/microsoft-common-serviceprincipal-authenticate.png" alt-text="Проверка подлинности Microsoft. Common. СервицепринЦипалселектор.":::

### <a name="use-existing-application"></a>Использовать существующее приложение

Чтобы использовать существующее приложение, щелкните **выбрать существующий** и выберите **сделать выбор**. Используйте диалоговое окно **Выбор приложения** для поиска имени приложения. В результатах выберите приложение, а затем нажмите кнопку **выбрать** . После выбора приложения элемент управления отображает **тип проверки подлинности** для ввода пароля или отпечатка сертификата.

:::image type="content" source="./media/managed-application-elements/microsoft-common-serviceprincipal-existing.png" alt-text="Microsoft. Common. СервицепринЦипалселектор выберите существующее приложение.":::

## <a name="schema"></a>схема

```json
{
  "name": "ServicePrincipal",
  "type": "Microsoft.Common.ServicePrincipalSelector",
  "label": {
    "password": "Password",
    "certificateThumbprint": "Certificate thumbprint",
    "authenticationType": "Authentication Type",
    "sectionHeader": "Service Principal"
  },
  "toolTip": {
    "password": "Password",
    "certificateThumbprint": "Certificate thumbprint",
    "authenticationType": "Authentication Type"
  },
  "defaultValue": {
    "principalId": "<default guid>",
    "name": "(New) default App Id"
  },
  "constraints": {
    "required": true,
    "regex": "^[a-zA-Z0-9]{8,}$",
    "validationMessage": "Password must be at least 8 characters long, contain only numbers and letters"
  },
  "options": {
    "hideCertificate": false
  },
  "visible": true
}
```

## <a name="remarks"></a>Remarks

- Ниже приведены обязательные свойства.
  - `name`
  - `type`
  - `label`
  - `defaultValue`: Указывает значения по умолчанию `principalId` и `name` .

- Ниже приведены необязательные свойства.
  - `toolTip`: Присоединяет подсказку `infoBalloon` к каждой метке.
  - `visible`: Скрытие или отображение элемента управления.
  - `options`: Указывает, следует ли сделать параметр отпечатка сертификата доступным.
  - `constraints`: Регулярные ограничения для проверки пароля.

## <a name="example"></a>Пример

Ниже приведен пример `Microsoft.Common.ServicePrincipalSelector` элемента управления. `defaultValue`Свойство задает `principalId` в `<default guid>` качестве заполнителя для идентификатора приложения по умолчанию.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
  "handler": "Microsoft.Azure.CreateUIDef",
  "version": "0.1.2-preview",
  "parameters": {
    "basics": [],
    "steps": [
      {
        "name": "SPNcontrol",
        "label": "SPNcontrol",
        "elements": [
          {
            "name": "ServicePrincipal",
            "type": "Microsoft.Common.ServicePrincipalSelector",
            "label": {
              "password": "Password",
              "certificateThumbprint": "Certificate thumbprint",
              "authenticationType": "Authentication Type",
              "sectionHeader": "Service Principal"
            },
            "toolTip": {
              "password": "Password",
              "certificateThumbprint": "Certificate thumbprint",
              "authenticationType": "Authentication Type"
            },
            "defaultValue": {
              "principalId": "<default guid>",
              "name": "(New) default App Id"
            },
            "constraints": {
              "required": true,
              "regex": "^[a-zA-Z0-9]{8,}$",
              "validationMessage": "Password must be at least 8 characters long, contain only numbers and letters"
            },
            "options": {
              "hideCertificate": false
            },
            "visible": true
          }
        ]
      }
    ],
    "outputs": {
      "appId": "[steps('SPNcontrol').ServicePrincipal.appId]",
      "objectId": "[steps('SPNcontrol').ServicePrincipal.objectId]",
      "password": "[steps('SPNcontrol').ServicePrincipal.password]",
      "certificateThumbprint": "[steps('SPNcontrol').ServicePrincipal.certificateThumbprint]",
      "newOrExisting": "[steps('SPNcontrol').ServicePrincipal.newOrExisting]",
      "authenticationType": "[steps('SPNcontrol').ServicePrincipal.authenticationType]"
    }
  }
}
```

## <a name="example-output"></a>Пример выходных данных

`appId`— Это идентификатор регистрации приложения, который вы выбрали или создали. `objectId`— Это массив идентификаторов объектов для субъектов-служб, настроенных для регистрации выбранного приложения.

Если выборка не сделана из элемента управления, `newOrExisting` значение свойства — **New**:

```json
{
  "appId": {
    "value": "<default guid>"
  },
  "objectId": {
    "value": ["<default guid>"]
  },
  "password": {
    "value": "<password>"
  },
  "certificateThumbprint": {
    "value": ""
  },
  "newOrExisting": {
    "value": "new"
  },
  "authenticationType": {
    "value": "password"
  }
}
```

Если в элементе управления выбрано **Создание нового** или существующее приложение, то `newOrExisting` значение свойства является **существующим**:

```json
{
  "appId": {
    "value": "<guid>"
  },
  "objectId": {
    "value": ["<guid>"]
  },
  "password": {
    "value": "<password>"
  },
  "certificateThumbprint": {
    "value": ""
  },
  "newOrExisting": {
    "value": "existing"
  },
  "authenticationType": {
    "value": "password"
  }
}
```

## <a name="next-steps"></a>Дальнейшие действия

- Общие сведения о создании определений пользовательского интерфейса см. в статье [Начало работы с CreateUiDefinition](create-uidefinition-overview.md).
- Дополнительные сведения об общих свойствах элементов пользовательского интерфейса см. в статье [Элементы CreateUiDefinition](create-uidefinition-elements.md).