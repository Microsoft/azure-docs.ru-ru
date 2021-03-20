---
title: Пользовательские роли Azure — Azure RBAC
description: Узнайте, как создавать пользовательские роли Azure с помощью управления доступом на основе ролей Azure (Azure RBAC) для точного управления доступом к ресурсам Azure.
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.topic: conceptual
ms.workload: identity
ms.date: 12/15/2020
ms.author: rolyon
ms.openlocfilehash: 79aaeee942a6d46243ee1c72d5904484b8698ebe
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97617329"
---
# <a name="azure-custom-roles"></a>Настраиваемые роли Azure

> [!IMPORTANT]
> В настоящее время добавление группы управления в `AssignableScopes` доступно в режиме предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Если [встроенные роли Azure](built-in-roles.md) не соответствуют потребностям вашей организации, вы можете создать собственные настраиваемые роли. Так же, как и встроенные роли, можно назначать пользовательские роли пользователям, группам и субъектам-службам в областях групп управления, подписок и групп ресурсов.

Пользовательские роли могут совместно использоваться подписками, которые доверяют одному каталогу Azure AD. Существует ограничение в **5 000** пользовательских ролей для каталога. (Для Azure для Германии и Azure для Китая с помощью 21Vianet ограничение составляет 2 000 пользовательских ролей.) Пользовательские роли можно создавать с помощью портал Azure, Azure PowerShell, Azure CLI или REST API.

## <a name="steps-to-create-a-custom-role"></a>Процедура создания пользовательской роли

Ниже приведены основные шаги по созданию пользовательской роли.

1. Определите необходимые разрешения.

    При создании пользовательской роли необходимо знать, какие операции доступны для определения разрешений. В общем случае вы начинаете с существующей встроенной роли и видоизменяете ее под свои потребности. Вы добавите операции в `Actions` `NotActions` свойства или [определения роли](role-definitions.md). Если у вас есть операции с данными, вы добавите их `DataActions` в `NotDataActions` свойства или.

    Дополнительные сведения см. в следующем разделе [как определить необходимые разрешения](#how-to-determine-the-permissions-you-need).

1. Выберите способ создания настраиваемой роли.

    Пользовательские роли можно создавать с помощью [портал Azure](custom-roles-portal.md), [Azure PowerShell](custom-roles-powershell.md), [Azure CLI](custom-roles-cli.md)или [REST API](custom-roles-rest.md).

1. Создайте настраиваемую роль.

    Самый простой способ — использовать портал Azure. Инструкции по созданию настраиваемой роли с помощью портал Azure см. в разделе [Создание или обновление пользовательских ролей Azure с помощью портал Azure](custom-roles-portal.md).

1. Проверьте пользовательскую роль.

    Создав пользовательскую роль, нужно проверить правильность ее работы. Если позже потребуется корректировка, вы можете обновить пользовательскую роль.

## <a name="how-to-determine-the-permissions-you-need"></a>Определение необходимых разрешений

В Azure есть тысячи разрешений, которые потенциально можно включить в пользовательскую роль. Ниже приведены некоторые методы, которые могут помочь определить разрешения, которые нужно добавить в пользовательскую роль.

- Просмотрите существующие [встроенные роли](built-in-roles.md).

    Может потребоваться изменить существующую роль или объединить разрешения, используемые в нескольких ролях.

- Перечислите службы Azure, к которым вы хотите предоставить доступ.

- Определите [поставщиков ресурсов, которые сопоставляются со службами Azure](../azure-resource-manager/management/azure-services-resource-providers.md).

    Службы Azure предоставляют свои функции и разрешения с помощью [поставщиков ресурсов](../azure-resource-manager/management/overview.md). Например, поставщик ресурсов Microsoft. COMPUTE предоставляет ресурсы виртуальной машины, а поставщик ресурсов Microsoft. Billing предоставляет подписку и ресурсы для выставления счетов. Знание поставщиков ресурсов поможет вам сократить и определить разрешения, необходимые для пользовательской роли.

    При создании пользовательской роли с помощью портал Azure можно также определить поставщиков ресурсов, выполнив поиск по ключевым словам. Эта функция поиска описана в разделе [Создание или обновление пользовательских ролей Azure с помощью портал Azure](custom-roles-portal.md#step-4-permissions).

    ![Панель "Добавление разрешений" с поставщиком ресурсов](./media/custom-roles-portal/add-permissions-provider.png)

- Выполните поиск [доступных разрешений](resource-provider-operations.md) , чтобы найти разрешения, которые нужно включить.

    При создании пользовательской роли с помощью портал Azure можно выполнять поиск разрешений по ключевому слову. Например, можно выполнить поиск по *виртуальной машине* или разрешениям на *выставление счетов* . Вы также можете скачать все разрешения как CSV-файл, а затем выполнить поиск в этом файле. Эта функция поиска описана в разделе [Создание или обновление пользовательских ролей Azure с помощью портал Azure](custom-roles-portal.md#step-4-permissions).

    ![Добавить список разрешений](./media/custom-roles-portal/add-permissions-list.png)

## <a name="custom-role-example"></a>Пример пользовательской роли

Ниже показано, как выглядит пользовательская роль, отображаемая с помощью Azure PowerShell в формате JSON. Эта пользовательская роль позволяет выполнять мониторинг и перезапуск виртуальных машин.

```json
{
  "Name": "Virtual Machine Operator",
  "Id": "88888888-8888-8888-8888-888888888888",
  "IsCustom": true,
  "Description": "Can monitor and restart virtual machines.",
  "Actions": [
    "Microsoft.Storage/*/read",
    "Microsoft.Network/*/read",
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Authorization/*/read",
    "Microsoft.ResourceHealth/availabilityStatuses/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read",
    "Microsoft.Insights/alertRules/*",
    "Microsoft.Insights/diagnosticSettings/*",
    "Microsoft.Support/*"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/{subscriptionId1}",
    "/subscriptions/{subscriptionId2}",
    "/providers/Microsoft.Management/managementGroups/{groupId1}"
  ]
}
```

Ниже показана та же самая настраиваемая роль, которая отображается с помощью Azure CLI.

```json
[
  {
    "assignableScopes": [
      "/subscriptions/{subscriptionId1}",
      "/subscriptions/{subscriptionId2}",
      "/providers/Microsoft.Management/managementGroups/{groupId1}"
    ],
    "description": "Can monitor and restart virtual machines.",
    "id": "/subscriptions/{subscriptionId1}/providers/Microsoft.Authorization/roleDefinitions/88888888-8888-8888-8888-888888888888",
    "name": "88888888-8888-8888-8888-888888888888",
    "permissions": [
      {
        "actions": [
          "Microsoft.Storage/*/read",
          "Microsoft.Network/*/read",
          "Microsoft.Compute/*/read",
          "Microsoft.Compute/virtualMachines/start/action",
          "Microsoft.Compute/virtualMachines/restart/action",
          "Microsoft.Authorization/*/read",
          "Microsoft.ResourceHealth/availabilityStatuses/read",
          "Microsoft.Resources/subscriptions/resourceGroups/read",
          "Microsoft.Insights/alertRules/*",
          "Microsoft.Insights/diagnosticSettings/*",
          "Microsoft.Support/*"
        ],
        "dataActions": [],
        "notActions": [],
        "notDataActions": []
      }
    ],
    "roleName": "Virtual Machine Operator",
    "roleType": "CustomRole",
    "type": "Microsoft.Authorization/roleDefinitions"
  }
]
```

После создания пользовательская роль отображается на портале Azure с оранжевым значком ресурса.

![Имя пользовательской роли](./media/custom-roles/roles-custom-role-icon.png)

## <a name="custom-role-properties"></a>Свойства пользовательской роли

В следующей таблице описано, что означают свойства пользовательской роли.

| Свойство | Обязательно | Тип | Описание |
| --- | --- | --- | --- |
| `Name`</br>`roleName` | Да | Строка | Отображаемое имя пользовательской роли. Хотя определение роли — это группа управления или ресурс уровня подписки, определение роли можно использовать в нескольких подписках, использующих один каталог Azure AD. Это отображаемое имя должно быть уникальным в рамках каталога Azure AD. Может содержать буквы, цифры, пробелы и специальные знаки. Максимальное количество знаков — 128. |
| `Id`</br>`name` | Да | Строка | Уникальный идентификатор настраиваемой роли. Для Azure PowerShell и Azure CLI этот идентификатор формируется автоматически при создании роли. |
| `IsCustom`</br>`roleType` | Да | Строка | Указывает, является ли эта роль настраиваемой. Задайте для параметра значение `true` или `CustomRole` для пользовательских ролей. Задайте значение `false` или `BuiltInRole` для встроенных ролей. |
| `Description`</br>`description` | Да | Строка | Описание пользовательской роли. Может содержать буквы, цифры, пробелы и специальные знаки. Максимальное количество знаков — 1024. |
| `Actions`</br>`actions` | Да | String[] | Массив строк, описывающий операции управления, которые роль разрешает выполнять. Дополнительные сведения см. в разделе [Actions](role-definitions.md#actions). |
| `NotActions`</br>`notActions` | Нет | String[] | Массив строк, указывающий операции управления, которые исключаются из разрешенных `Actions`. Дополнительные сведения см. в разделе [NotActions](role-definitions.md#notactions). |
| `DataActions`</br>`dataActions` | Нет | String[] | Массив строк, указывающий операции с данными, которые роль разрешает выполнять с вашими данными внутри этого объекта. Если создать пользовательскую роль с `DataActions` , эту роль нельзя будет назначить в области группы управления. Дополнительные сведения см. в разделе [действия](role-definitions.md#dataactions)с данными. |
| `NotDataActions`</br>`notDataActions` | Нет | String[] | Массив строк, указывающий операции с данными, которые исключаются из разрешенных `DataActions`. Дополнительные сведения см. в разделе [нотдатаактионс](role-definitions.md#notdataactions). |
| `AssignableScopes`</br>`assignableScopes` | Да | String[] | Массив строк, который указывает области, в которых пользовательская роль может быть назначена. В пользовательской роли можно определить только одну группу управления `AssignableScopes` . В настоящее время добавление группы управления в `AssignableScopes` доступно в режиме предварительной версии. Дополнительные сведения см. в разделе [AssignableScopes](role-definitions.md#assignablescopes). |

## <a name="wildcard-permissions"></a>Разрешения с подстановочными знаками

`Actions`, `NotActions` , `DataActions` и `NotDataActions` поддерживают подстановочные знаки ( `*` ) для определения разрешений. Подстановочный знак ( `*` ) расширяет разрешение на все, что соответствует предоставленной строке действий. Например, предположим, что вы решили добавить все разрешения, связанные с управлением затратами Azure и экспортами. Вы можете добавить все следующие строки действий:

```
Microsoft.CostManagement/exports/action
Microsoft.CostManagement/exports/read
Microsoft.CostManagement/exports/write
Microsoft.CostManagement/exports/delete
Microsoft.CostManagement/exports/run/action
```

Вместо добавления всех этих строк можно просто добавить строку с подстановочными знаками. Например, следующая строка с подстановочными знаками эквивалентна предыдущим пяти строкам. Сюда также входят все будущие разрешения экспорта, которые могут быть добавлены.

```
Microsoft.CostManagement/exports/*
```

В строке можно также использовать несколько подстановочных знаков. Например, следующая строка представляет все разрешения на запросы для управления затратами.

```
Microsoft.CostManagement/*/query/*
```

## <a name="who-can-create-delete-update-or-view-a-custom-role"></a>Кто может создавать, удалять, обновлять или просматривать пользовательскую роль

Как и в случае со встроенными ролями, свойство `AssignableScopes` указывает области, в которых роль может быть назначена. Свойство `AssignableScopes` пользовательской роли также определяет, кто может создавать, удалять, изменять или просматривать пользовательскую роль.

| Задача | Операция | Описание |
| --- | --- | --- |
| Создание или удаление пользовательской роли | `Microsoft.Authorization/ roleDefinitions/write` | Пользователи с разрешением на эту операцию для всех `AssignableScopes` пользовательской роли могут создавать (или удалять) пользовательские роли для использования в этих областях. Например, [владельцы](built-in-roles.md#owner) и [Администраторы доступа пользователей](built-in-roles.md#user-access-administrator) групп управления, подписок и групп ресурсов. |
| Обновление пользовательской роли | `Microsoft.Authorization/ roleDefinitions/write` | Пользователи с разрешением на эту операцию для всех `AssignableScopes` пользовательской роли могут обновлять пользовательские роли в этих областях. Например, [владельцы](built-in-roles.md#owner) и [Администраторы доступа пользователей](built-in-roles.md#user-access-administrator) групп управления, подписок и групп ресурсов. |
| Просмотр пользовательской роли | `Microsoft.Authorization/ roleDefinitions/read` | Пользователи с разрешением на эту операцию в определенной области могут просматривать пользовательские роли, которые доступны для назначения в этой области. Все встроенные роли позволяют использовать настраиваемые роли для назначения. |

## <a name="custom-role-limits"></a>Ограничения пользовательской роли

В следующем списке описаны ограничения для пользовательских ролей.

- Каждый каталог может иметь до **5000** пользовательских ролей.
- Azure для Германии и Azure для Китая в Китае могут иметь до 2000 пользовательских ролей для каждого каталога.
- Нельзя задать `AssignableScopes` в качестве корневой области ( `"/"` ).
- В нельзя использовать подстановочные знаки ( `*` ) `AssignableScopes` . Это ограничение по шаблону позволяет гарантировать, что пользователь не сможет получить доступ к области, обновив определение роли.
- В пользовательской роли можно определить только одну группу управления `AssignableScopes` . В настоящее время добавление группы управления в `AssignableScopes` доступно в режиме предварительной версии.
- Пользовательские роли с `DataActions` не могут быть назначены в области группы управления.
- Azure Resource Manager не проверяет существование группы управления в назначаемой области определения роли.

Дополнительные сведения о настраиваемых ролях и группах управления см. в статье [организация ресурсов с помощью групп управления Azure](../governance/management-groups/overview.md#azure-custom-role-definition-and-assignment).

## <a name="input-and-output-formats"></a>Форматы входных и выходных данных

Для создания пользовательской роли с помощью командной строки, как правило, используется JSON для указания нужных свойств настраиваемой роли. В зависимости от используемых средств форматы входных и выходных данных будут немного отличаться. В этом разделе перечислены форматы ввода и вывода в зависимости от инструмента.

### <a name="azure-powershell"></a>Azure PowerShell

Чтобы создать настраиваемую роль с помощью Azure PowerShell, необходимо указать следующие входные данные.

```json
{
  "Name": "",
  "Description": "",
  "Actions": [],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": []
}
```

Чтобы обновить пользовательскую роль с помощью Azure PowerShell, необходимо указать следующие входные данные. Обратите внимание, что `Id` свойство добавлено. 

```json
{
  "Name": "",
  "Id": "",
  "Description": "",
  "Actions": [],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": []
}
```

Ниже приведен пример выходных данных при перечислении пользовательской роли с помощью Azure PowerShell и команды [ConvertTo-JSON](/powershell/module/microsoft.powershell.utility/convertto-json) . 

```json
{
  "Name": "",
  "Id": "",
  "IsCustom": true,
  "Description": "",
  "Actions": [],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": []
}
```

### <a name="azure-cli"></a>Azure CLI

Чтобы создать или обновить настраиваемую роль с помощью Azure CLI, необходимо указать следующие входные данные. Этот формат имеет тот же формат при создании пользовательской роли с помощью Azure PowerShell.

```json
{
  "Name": "",
  "Description": "",
  "Actions": [],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": []
}
```

Ниже приведен пример выходных данных при перечислении пользовательской роли с помощью Azure CLI.

```json
[
  {
    "assignableScopes": [],
    "description": "",
    "id": "",
    "name": "",
    "permissions": [
      {
        "actions": [],
        "dataActions": [],
        "notActions": [],
        "notDataActions": []
      }
    ],
    "roleName": "",
    "roleType": "CustomRole",
    "type": "Microsoft.Authorization/roleDefinitions"
  }
]
```

### <a name="rest-api"></a>REST API

Чтобы создать или обновить настраиваемую роль с помощью REST API, необходимо указать следующие входные данные. Этот формат имеет тот же формат, который создается при создании пользовательской роли с помощью портал Azure.

```json
{
  "properties": {
    "roleName": "",
    "description": "",
    "assignableScopes": [],
    "permissions": [
      {
        "actions": [],
        "notActions": [],
        "dataActions": [],
        "notDataActions": []
      }
    ]
  }
}
```

Ниже приведен пример выходных данных при перечислении пользовательской роли с помощью REST API.

```json
{
    "properties": {
        "roleName": "",
        "type": "CustomRole",
        "description": "",
        "assignableScopes": [],
        "permissions": [
            {
                "actions": [],
                "notActions": [],
                "dataActions": [],
                "notDataActions": []
            }
        ],
        "createdOn": "",
        "updatedOn": "",
        "createdBy": "",
        "updatedBy": ""
    },
    "id": "",
    "type": "Microsoft.Authorization/roleDefinitions",
    "name": ""
}
```

## <a name="next-steps"></a>Дальнейшие действия

- [Руководство по Создание настраиваемой роли Azure с помощью Azure PowerShell](tutorial-custom-role-powershell.md)
- [Руководство по Создание настраиваемой роли Azure с помощью Azure CLI](tutorial-custom-role-cli.md)
- [Общие сведения о определениях ролей Azure](role-definitions.md)
- [Устранение неполадок в Azure RBAC](troubleshooting.md)
