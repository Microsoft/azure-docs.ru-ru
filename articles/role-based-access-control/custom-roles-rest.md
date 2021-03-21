---
title: Создание или обновление пользовательских ролей Azure с помощью REST API Azure RBAC
description: Узнайте, как перечислять, создавать, обновлять или удалять пользовательские роли Azure с помощью REST API и управления доступом на основе ролей Azure (Azure RBAC).
services: active-directory
documentationcenter: na
author: rolyon
manager: mtillman
editor: ''
ms.assetid: 1f90228a-7aac-4ea7-ad82-b57d222ab128
ms.service: role-based-access-control
ms.workload: multiple
ms.tgt_pltfrm: rest-api
ms.devlang: na
ms.topic: how-to
ms.date: 03/19/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: ab64765fc4e329c8edbf7a4db813113d20d476b3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100557514"
---
# <a name="create-or-update-azure-custom-roles-using-the-rest-api"></a>Создание или обновление пользовательских ролей Azure с помощью REST API

> [!IMPORTANT]
> В настоящее время добавление группы управления в `AssignableScopes` доступно в режиме предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Если [встроенные роли Azure](built-in-roles.md) не соответствуют потребностям вашей организации, вы можете создать собственные настраиваемые роли. В этой статье описывается, как получить список, создать, обновить или удалить пользовательские роли с помощью REST API.

## <a name="list-custom-roles"></a>Вывод списка настраиваемых ролей

Чтобы получить список всех пользовательских ролей в каталоге, используйте REST API [списка "определения ролей](/rest/api/authorization/roledefinitions/list) ".

1. Можете начать со следующего запроса:

    ```http
    GET https://management.azure.com/providers/Microsoft.Authorization/roleDefinitions?api-version=2015-07-01&$filter={filter}
    ```

1. Замените *{Filter}* типом роли.

    > [!div class="mx-tableFixed"]
    > | Filter | Описание |
    > | --- | --- |
    > | `$filter=type+eq+'CustomRole'` | Фильтр, основанный на типе Кустомроле |

## <a name="list-custom-roles-at-a-scope"></a>Вывод списка пользовательских ролей в области

Чтобы получить список пользовательских ролей в области, используйте REST API [списка "определения ролей](/rest/api/authorization/roledefinitions/list) ".

1. Можете начать со следующего запроса:

    ```http
    GET https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions?api-version=2015-07-01&$filter={filter}
    ```

1. В URI замените *{scope}* на область, для которой нужно получить список ролей.

    > [!div class="mx-tableFixed"]
    > | Область | Type |
    > | --- | --- |
    > | `subscriptions/{subscriptionId1}` | Подписка |
    > | `subscriptions/{subscriptionId1}/resourceGroups/{resourceGroup1}` | Группа ресурсов |
    > | `subscriptions/{subscriptionId1}/resourceGroups/{resourceGroup1}/providers/Microsoft.Web/sites/{site1}` | Ресурс |
    > | `providers/Microsoft.Management/managementGroups/{groupId1}` | группа управления; |

1. Замените *{Filter}* типом роли.

    > [!div class="mx-tableFixed"]
    > | Filter | Описание |
    > | --- | --- |
    > | `$filter=type+eq+'CustomRole'` | Фильтр, основанный на типе Кустомроле |

## <a name="list-a-custom-role-definition-by-name"></a>Вывод пользовательского определения роли по имени

Чтобы получить сведения о пользовательской роли по ее отображаемому имени, используйте [определения ролей — get](/rest/api/authorization/roledefinitions/get) REST API.

1. Можете начать со следующего запроса:

    ```http
    GET https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions?api-version=2015-07-01&$filter={filter}
    ```

1. В URI замените *{scope}* на область, для которой нужно получить список ролей.

    > [!div class="mx-tableFixed"]
    > | Область | Type |
    > | --- | --- |
    > | `subscriptions/{subscriptionId1}` | Подписка |
    > | `subscriptions/{subscriptionId1}/resourceGroups/{resourceGroup1}` | Группа ресурсов |
    > | `subscriptions/{subscriptionId1}/resourceGroups/{resourceGroup1}/providers/Microsoft.Web/sites/{site1}` | Ресурс |
    > | `providers/Microsoft.Management/managementGroups/{groupId1}` | группа управления; |

1. Замените *{Filter}* отображаемым именем роли.

    > [!div class="mx-tableFixed"]
    > | Filter | Описание |
    > | --- | --- |
    > | `$filter=roleName+eq+'{roleDisplayName}'` | Используйте точное отображаемое имя роли в формате URL-адреса. Например, `$filter=roleName+eq+'Virtual%20Machine%20Contributor'` |

## <a name="list-a-custom-role-definition-by-id"></a>Вывод определения пользовательской роли по ИДЕНТИФИКАТОРу

Чтобы получить сведения о пользовательской роли по ее уникальному идентификатору, используйте [определения ролей — get](/rest/api/authorization/roledefinitions/get) REST API.

1. Для получения идентификатора GUID роли используйте REST API [Определения ролей — список](/rest/api/authorization/roledefinitions/list).

1. Можете начать со следующего запроса:

    ```http
    GET https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId}?api-version=2015-07-01
    ```

1. В URI замените *{scope}* на область, для которой нужно получить список ролей.

    > [!div class="mx-tableFixed"]
    > | Область | Type |
    > | --- | --- |
    > | `subscriptions/{subscriptionId1}` | Подписка |
    > | `subscriptions/{subscriptionId1}/resourceGroups/{resourceGroup1}` | Группа ресурсов |
    > | `subscriptions/{subscriptionId1}/resourceGroups/{resourceGroup1}/providers/Microsoft.Web/sites/{site1}` | Ресурс |
    > | `providers/Microsoft.Management/managementGroups/{groupId1}` | группа управления; |

1. Замените *{roleDefinitionId}* идентификатором GUID определения роли.

## <a name="create-a-custom-role"></a>Создание пользовательской роли

Чтобы создать пользовательскую роль, используйте REST API [Определения ролей — создание или обновление](/rest/api/authorization/roledefinitions/createorupdate). Чтобы вызвать этот API, необходимо войти в систему с помощью пользователя, которому назначена роль, имеющая `Microsoft.Authorization/roleDefinitions/write` разрешение на доступ ко всем `assignableScopes` . Из встроенных ролей это разрешение включает только [владелец](built-in-roles.md#owner) и [администратор доступа пользователей](built-in-roles.md#user-access-administrator) .

1. Просмотрите список доступных [операций поставщиков ресурсов](resource-provider-operations.md) для создания разрешений для пользовательской роли.

1. Используйте средство GUID для создания уникального идентификатора, который будет использоваться в качестве идентификатора пользовательской роли. Этот идентификатор имеет следующий формат: `00000000-0000-0000-0000-000000000000`

1. Можете начать со следующего запроса и основного текста:

    ```http
    PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId}?api-version=2015-07-01
    ```

    ```json
    {
      "name": "{roleDefinitionId}",
      "properties": {
        "roleName": "",
        "description": "",
        "type": "CustomRole",
        "permissions": [
          {
            "actions": [
    
            ],
            "notActions": [
    
            ]
          }
        ],
        "assignableScopes": [
          "/subscriptions/{subscriptionId1}",
          "/subscriptions/{subscriptionId2}",
          "/subscriptions/{subscriptionId1}/resourceGroups/{resourceGroup1}",
          "/subscriptions/{subscriptionId2}/resourceGroups/{resourceGroup2}",
          "/providers/Microsoft.Management/managementGroups/{groupId1}"
        ]
      }
    }
    ```

1. В URI замените *{scope}* на первую область `assignableScopes` пользовательской роли.

    > [!div class="mx-tableFixed"]
    > | Область | Type |
    > | --- | --- |
    > | `subscriptions/{subscriptionId1}` | Подписка |
    > | `subscriptions/{subscriptionId1}/resourceGroups/{resourceGroup1}` | Группа ресурсов |
    > | `providers/Microsoft.Management/managementGroups/{groupId1}` | группа управления; |

1. Замените *{roleDefinitionId}* идентификатором GUID настраиваемой роли.

1. В тексте запроса замените *{roleDefinitionId}* идентификатором GUID.

1. Если `assignableScopes` является подпиской или группой ресурсов, замените экземпляры *{SubscriptionId}* или *{resourceGroup}* идентификаторами.

1. Если `assignableScopes` является группой управления, замените экземпляр *{groupId}* идентификатором группы управления. В настоящее время добавление группы управления в `assignableScopes` доступно в режиме предварительной версии.

1. В свойстве `actions` добавьте операции, которые разрешают выполнение роли.

1. В свойстве `notActions` добавьте операции, которые исключены из допустимых `actions`.

1. В свойствах `roleName` и `description` укажите уникальное имя и описание роли. Дополнительные сведения о свойствах см. в статье [пользовательские роли Azure](custom-roles.md).

    Ниже приведен пример текста запроса:

    ```json
    {
      "name": "88888888-8888-8888-8888-888888888888",
      "properties": {
        "roleName": "Virtual Machine Operator",
        "description": "Can monitor and restart virtual machines.",
        "type": "CustomRole",
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
              "Microsoft.Support/*"
            ],
            "notActions": []
          }
        ],
        "assignableScopes": [
          "/subscriptions/00000000-0000-0000-0000-000000000000",
          "/providers/Microsoft.Management/managementGroups/marketing-group"
        ]
      }
    }
    ```

## <a name="update-a-custom-role"></a>Обновление пользовательской роли

Чтобы обновить пользовательскую роль, используйте REST API [Определения ролей — создание или обновление](/rest/api/authorization/roledefinitions/createorupdate). Чтобы вызвать этот API, необходимо войти в систему с помощью пользователя, которому назначена роль, имеющая `Microsoft.Authorization/roleDefinitions/write` разрешение на доступ ко всем `assignableScopes` . Из встроенных ролей это разрешение включает только [владелец](built-in-roles.md#owner) и [администратор доступа пользователей](built-in-roles.md#user-access-administrator) .

1. Чтобы получить сведения о пользовательской роли, используйте REST API [Определения ролей — список](/rest/api/authorization/roledefinitions/list) или [Определения ролей — получение](/rest/api/authorization/roledefinitions/get). Дополнительные сведения см. в разделе более ранние [пользовательские роли списка](#list-custom-roles) .

1. Можете начать со следующего запроса:

    ```http
    PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId}?api-version=2015-07-01
    ```

1. В URI замените *{scope}* на первую область `assignableScopes` пользовательской роли.

    > [!div class="mx-tableFixed"]
    > | Область | Type |
    > | --- | --- |
    > | `subscriptions/{subscriptionId1}` | Подписка |
    > | `subscriptions/{subscriptionId1}/resourceGroups/{resourceGroup1}` | Группа ресурсов |
    > | `providers/Microsoft.Management/managementGroups/{groupId1}` | группа управления; |

1. Замените *{roleDefinitionId}* идентификатором GUID настраиваемой роли.

1. На основе сведений о пользовательской роли создайте текст запроса в следующем формате:

    ```json
    {
      "name": "{roleDefinitionId}",
      "properties": {
        "roleName": "",
        "description": "",
        "type": "CustomRole",
        "permissions": [
          {
            "actions": [
    
            ],
            "notActions": [
    
            ]
          }
        ],
        "assignableScopes": [
          "/subscriptions/{subscriptionId1}",
          "/subscriptions/{subscriptionId2}",
          "/subscriptions/{subscriptionId1}/resourceGroups/{resourceGroup1}",
          "/subscriptions/{subscriptionId2}/resourceGroups/{resourceGroup2}",
          "/providers/Microsoft.Management/managementGroups/{groupId1}"
        ]
      }
    }
    ```

1. Внесите необходимые изменения для пользовательской роли в текст запроса.

    Ниже приведен пример текста запроса с добавленным действием "Параметры диагностики":

    ```json
    {
      "name": "88888888-8888-8888-8888-888888888888",
      "properties": {
        "roleName": "Virtual Machine Operator",
        "description": "Can monitor and restart virtual machines.",
        "type": "CustomRole",
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
            "notActions": []
          }
        ],
        "assignableScopes": [
          "/subscriptions/00000000-0000-0000-0000-000000000000",
          "/providers/Microsoft.Management/managementGroups/marketing-group"
        ]
      }
    }
    ```

## <a name="delete-a-custom-role"></a>Удаление настраиваемой роли

Чтобы удалить пользовательскую роль, используйте REST API [Определения ролей — удаление](/rest/api/authorization/roledefinitions/delete). Чтобы вызвать этот API, необходимо войти в систему с помощью пользователя, которому назначена роль, имеющая `Microsoft.Authorization/roleDefinitions/delete` разрешение на доступ ко всем `assignableScopes` . Из встроенных ролей это разрешение включает только [владелец](built-in-roles.md#owner) и [администратор доступа пользователей](built-in-roles.md#user-access-administrator) .

1. Чтобы получить идентификатор GUID пользовательской роли, используйте REST API [Определения ролей — список](/rest/api/authorization/roledefinitions/list) или [Определения ролей — получение](/rest/api/authorization/roledefinitions/get). Дополнительные сведения см. в разделе более ранние [пользовательские роли списка](#list-custom-roles) .

1. Можете начать со следующего запроса:

    ```http
    DELETE https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId}?api-version=2015-07-01
    ```

1. В URI замените *{scope}* на область, для которой нужно удалить пользовательскую роль.

    > [!div class="mx-tableFixed"]
    > | Область | Type |
    > | --- | --- |
    > | `subscriptions/{subscriptionId1}` | Подписка |
    > | `subscriptions/{subscriptionId1}/resourceGroups/{resourceGroup1}` | Группа ресурсов |
    > | `providers/Microsoft.Management/managementGroups/{groupId1}` | группа управления; |

1. Замените *{roleDefinitionId}* идентификатором GUID настраиваемой роли.

## <a name="next-steps"></a>Дальнейшие действия

- [Настраиваемые роли Azure](custom-roles.md)
- [Назначение ролей Azure с помощью REST API](role-assignments-rest.md)
- [Справочник по REST API Azure](/rest/api/azure/)
