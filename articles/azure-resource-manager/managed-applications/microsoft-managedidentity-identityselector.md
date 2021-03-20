---
title: Элемент пользовательского интерфейса Идентитиселектор
description: Описывает элемент пользовательского интерфейса Microsoft. ManagedIdentity. Идентитиселектор для портал Azure. Используется для назначения управляемых удостоверений ресурсу.
author: tfitzmac
ms.topic: conceptual
ms.date: 02/06/2020
ms.author: tomfitz
ms.openlocfilehash: 91e3cd0b99825fd72eb342ce7a8555b046455538
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87063365"
---
# <a name="microsoftmanagedidentityidentityselector-ui-element"></a>Элемент пользовательского интерфейса Microsoft. ManagedIdentity. Идентитиселектор

Элемент управления для назначения [управляемых удостоверений](../../active-directory/managed-identities-azure-resources/overview.md) для ресурса в развертывании.

## <a name="ui-sample"></a>Пример элемента пользовательского интерфейса

Элемент управления состоит из следующих элементов:

![Первый шаг Microsoft. ManagedIdentity. Идентитиселектор](./media/managed-application-elements/microsoft-managedidentity-identityselector-1.png)

Когда пользователь нажимает кнопку **Добавить**, открывается следующая форма. Пользователь может выбрать одно или несколько назначенных пользователю удостоверений для ресурса.

![Второй шаг Microsoft. ManagedIdentity. Идентитиселектор](./media/managed-application-elements/microsoft-managedidentity-identityselector-2.png)

Выбранные удостоверения отображаются в таблице. Пользователь может добавлять или удалять элементы из этой таблицы.

![Третий шаг Microsoft. ManagedIdentity. Идентитиселектор](./media/managed-application-elements/microsoft-managedidentity-identityselector-3.png)

## <a name="schema"></a>схема

```json
{
  "name": "identity",
  "type": "Microsoft.ManagedIdentity.IdentitySelector",
  "label": "Managed Identity Configuration",
  "toolTip": {
    "systemAssignedIdentity": "Enable system assigned identity to grant the resource access to other existing resources.",
    "userAssignedIdentity": "Add user assigned identities to grant the resource access to other existing resources."
  },
  "defaultValue": {
    "systemAssignedIdentity": "Off"
  },
  "options": {
    "hideSystemAssignedIdentity": false,
    "hideUserAssignedIdentity": false
  },
  "visible": true
}
```

## <a name="sample-output"></a>Пример выходных данных

```json
{
  "identity": {
    "value": {
      "type": "UserAssigned",
      "userAssignedIdentities": {
        "/subscriptions/xxxx/resourceGroups/TestResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/TestUserIdentity1": {}
      }
    }
  }
}
```

## <a name="remarks"></a>Комментарии

- Используйте **defaultValue.sysтемассигнедидентити** , чтобы задать начальное значение для элемента управления "параметры удостоверения, назначенного системой". Значение по умолчанию — **Off**. Допустимы следующие значения:
  - **On** — назначенное системой удостоверение назначается ресурсу.
  - **Off** — назначенное системой удостоверение не назначается ресурсу.
  - **Только** ononly назначенное системой удостоверение назначается ресурсу. Пользователи не могут изменять это значение во время развертывания.
  - **Оффонли** — назначенное системой удостоверение не назначено ресурсу. Пользователи не могут изменять это значение во время развертывания.

- Если **параметр options. хидесистемассигнедидентити** имеет значение **true**, Пользовательский интерфейс для настройки назначенного системой удостоверения не отображается. Значение по умолчанию для этого параметра — **false**.
- Если **параметр options. хидеусерассигнедидентити** имеет значение **true**, Пользовательский интерфейс для настройки назначенного пользователю удостоверения не отображается. Ресурсу не назначено удостоверение, назначенное пользователем. Значение по умолчанию для этого параметра — **false**.

## <a name="next-steps"></a>Дальнейшие действия

- Общие сведения о создании определений пользовательского интерфейса см. в статье [Начало работы с CreateUiDefinition](create-uidefinition-overview.md).
- Дополнительные сведения об общих свойствах элементов пользовательского интерфейса см. в статье [Элементы CreateUiDefinition](create-uidefinition-elements.md).