---
title: Создание или обновление пользовательских ролей Azure с помощью Azure CLI — Azure RBAC
description: Узнайте, как перечислять, создавать, обновлять или удалять пользовательские роли Azure с помощью Azure CLI и управления доступом на основе ролей Azure (Azure RBAC).
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.assetid: 3483ee01-8177-49e7-b337-4d5cb14f5e32
ms.service: role-based-access-control
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 06/17/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: 31dabcf77f0db76047919fa76d00f1c5ed3c96d6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97369146"
---
# <a name="create-or-update-azure-custom-roles-using-azure-cli"></a>Создание или изменение настраиваемых ролей Azure с помощью Azure CLI

> [!IMPORTANT]
> В настоящее время добавление группы управления в `AssignableScopes` доступно в режиме предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Если [встроенные роли Azure](built-in-roles.md) не соответствуют потребностям вашей организации, вы можете создать собственные настраиваемые роли. В этой статье описывается, как получить список, создать, обновить или удалить пользовательские роли с помощью Azure CLI.

Пошаговое руководство по созданию настраиваемой роли см. в разделе [учебник. Создание настраиваемой роли Azure с помощью Azure CLI](tutorial-custom-role-cli.md).

## <a name="prerequisites"></a>Предварительные условия

Для создания пользовательских ролей в Azure требуются:

- разрешения на создание пользовательских ролей, такие как [Владелец](built-in-roles.md#owner) или [Администратор доступа пользователя](built-in-roles.md#user-access-administrator);
- [Azure Cloud Shell](../cloud-shell/overview.md) или [Azure CLI](/cli/azure/install-azure-cli).

## <a name="list-custom-roles"></a>Вывод списка настраиваемых ролей

Чтобы получить список пользовательских ролей, доступных для назначения, используйте команду [az role definition list](/cli/azure/role/definition#az-role-definition-list). В следующем примере перечисляются все пользовательские роли в текущей подписке.

```azurecli
az role definition list --custom-role-only true --output json --query '[].{roleName:roleName, roleType:roleType}'
```

```json
[
  {
    "roleName": "My Management Contributor",
    "type": "CustomRole"
  },
  {
    "roleName": "My Service Reader Role",
    "type": "CustomRole"
  },
  {
    "roleName": "Virtual Machine Operator",
    "type": "CustomRole"
  }
]
```

## <a name="list-a-custom-role-definition"></a>Вывод определения пользовательской роли

Чтобы получить список пользовательских определений ролей, используйте команду [AZ Role Definition List](/cli/azure/role/definition#az-role-definition-list). Это та же команда, которая используется для встроенной роли.

```azurecli
az role definition list --name {roleName}
```

В следующем примере показано определение роли *оператора виртуальной машины* :

```azurecli
az role definition list --name "Virtual Machine Operator"
```

```json
[
  {
    "assignableScopes": [
      "/subscriptions/{subscriptionId}"
    ],
    "description": "Can monitor and restart virtual machines.",
    "id": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/00000000-0000-0000-0000-000000000000",
    "name": "00000000-0000-0000-0000-000000000000",
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

В следующем примере перечисляются только действия роли *оператора виртуальной машины* :

```azurecli
az role definition list --name "Virtual Machine Operator" --output json --query '[].permissions[0].actions'
```

```json
[
  [
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
  ]
]
```

## <a name="create-a-custom-role"></a>Создание пользовательской роли

Чтобы создать пользовательскую роль, используйте команду [az role definition create](/cli/azure/role/definition#az-role-definition-create). Определением роли может быть описание JSON или путь к файлу, содержащему описание JSON.

```azurecli
az role definition create --role-definition {roleDefinition}
```

В следующем примере показано создание пользовательской роли *Оператор виртуальной машины*. Пользовательская роль предоставляет доступ ко всем операциям чтения поставщиков ресурсов *Microsoft.Compute*, *Microsoft.Storage* и *Microsoft.Network*, а также доступ для запуска, перезапуска и мониторинга виртуальных машин. Эту настраиваемую роль можно использовать в двух подписках. В этом примере в качестве входных данных используется JSON-файл.

vmoperator.json

```json
{
  "Name": "Virtual Machine Operator",
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
    "Microsoft.Support/*"
  ],
  "NotActions": [

  ],
  "AssignableScopes": [
    "/subscriptions/{subscriptionId1}",
    "/subscriptions/{subscriptionId2}"
  ]
}
```

```azurecli
az role definition create --role-definition ~/roles/vmoperator.json
```

## <a name="update-a-custom-role"></a>Обновление пользовательской роли

Чтобы обновить пользовательскую роль, сначала используйте команду [az role definition list](/cli/azure/role/definition#az-role-definition-list) для получения определения роли. Затем внесите необходимые изменения в определение роли. И наконец, сохраните обновленное определение роли с помощью команды [az role definition update](/cli/azure/role/definition#az-role-definition-update).

```azurecli
az role definition update --role-definition {roleDefinition}
```

В следующем примере добавляется операция *Microsoft. Insights/diagnosticSettings/* , которая `Actions` добавляет группу управления в `AssignableScopes` для настраиваемой роли *оператора виртуальной машины* . В настоящее время добавление группы управления в `AssignableScopes` доступно в режиме предварительной версии.

vmoperator.json

```json
{
  "Name": "Virtual Machine Operator",
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
  "NotActions": [

  ],
  "AssignableScopes": [
    "/subscriptions/{subscriptionId1}",
    "/subscriptions/{subscriptionId2}",
    "/providers/Microsoft.Management/managementGroups/marketing-group"
  ]
}
```

```azurecli
az role definition update --role-definition ~/roles/vmoperator.json
```

## <a name="delete-a-custom-role"></a>Удаление настраиваемой роли

Чтобы удалить пользовательскую роль, используйте команду [az role definition delete](/cli/azure/role/definition#az-role-definition-delete). Чтобы указать роль, которую необходимо удалить, используйте имя или идентификатор роли. Чтобы определить идентификатор роли, используйте команду [az role definition list](/cli/azure/role/definition#az-role-definition-list).

```azurecli
az role definition delete --name {roleNameOrId}
```

В следующем примере показано удаление пользовательской роли *Оператор виртуальной машины*.

```azurecli
az role definition delete --name "Virtual Machine Operator"
```

## <a name="next-steps"></a>Дальнейшие действия

- [Руководство по Создание настраиваемой роли Azure с помощью Azure CLI](tutorial-custom-role-cli.md)
- [Настраиваемые роли Azure](custom-roles.md)
- [Операции с поставщиками ресурсов Azure](resource-provider-operations.md)