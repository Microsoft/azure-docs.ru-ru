---
title: Руководство по Создание настраиваемой роли Azure с помощью Azure CLI — Azure RBAC
description: Из этой статьи вы узнаете, как создать настраиваемую роль Azure с помощью Azure CLI и управления доступом на основе ролей Azure (Azure RBAC).
services: active-directory
documentationCenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: role-based-access-control
ms.devlang: ''
ms.topic: tutorial
ms.tgt_pltfrm: ''
ms.workload: identity
ms.date: 02/20/2019
ms.author: rolyon
ms.custom: devx-track-azurecli
ms.openlocfilehash: e3743697d58d0f5b167b123df59bc5638aa60489
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107771683"
---
# <a name="tutorial-create-an-azure-custom-role-using-azure-cli"></a>Руководство по Создание настраиваемой роли Azure с помощью Azure CLI

Если [встроенные роли Azure](built-in-roles.md) не соответствуют потребностям вашей организации, вы можете создать собственные настраиваемые роли. С помощью этого руководства и Azure CLI вы создадите настраиваемую роль с именем "Запросы в службу поддержки от читателя". Она позволяет пользователю просматривать все объекты в плоскости управления подписки, а также делать запросы в службу поддержки.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создание настраиваемой роли
> * Вывод списка настраиваемых ролей
> * Обновление пользовательской роли
> * Удаление настраиваемой роли

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Для выполнения этого руководства потребуется следующее:

- разрешения на создание пользовательских ролей, такие как [Владелец](built-in-roles.md#owner) или [Администратор доступа пользователя](built-in-roles.md#user-access-administrator);
- [Azure Cloud Shell](../cloud-shell/overview.md) или [Azure CLI](/cli/azure/install-azure-cli).

## <a name="sign-in-to-azure-cli"></a>Вход в Azure CLI

[Войдите в Azure CLI](/cli/azure/authenticate-azure-cli).

## <a name="create-a-custom-role"></a>Создание настраиваемой роли

Создать пользовательскую роль проще всего с помощью шаблона JSON, в который вы можете добавлять изменения для создания новой роли.

1. Просмотрите список операций для [поставщика ресурсов Microsoft.Support](resource-provider-operations.md#microsoftsupport). Полезно знать операции, которые доступны для создания разрешений.

    | Операция | Описание |
    | --- | --- |
    | Microsoft.Support/register/action | Выполняет регистрацию в поставщике ресурсов поддержки. |
    | Microsoft.Support/supportTickets/read | Возвращает подробные сведения о запросе в службу поддержки (включая состояние, серьезность, контактную информацию и сообщения) или возвращает список запросов в службу поддержки в подписках. |
    | Microsoft.Support/supportTickets/write | Создает или обновляет запрос в службу поддержки. Можно создавать запросы в службу поддержки по техническим проблемам, а также проблемам, связанным с выставлением счетов, квотами или управлением подписками. Вы можете изменить уровень серьезности, контактную информацию и сообщения для существующих запросов в службу поддержки. |
    
1. Создайте файл с именем **ReaderSupportRole.json**.

1. Откройте роль ReaderSupportRole.json в редакторе и добавьте приведенный ниже код JSON.

    См. дополнительные сведения о [различных свойствах настраиваемых ролей](custom-roles.md).

    ```json
    {
      "Name": "",
      "IsCustom": true,
      "Description": "",
      "Actions": [],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/subscriptions/{subscriptionId1}"
      ]
    }
    ```
    
1. Добавьте в свойство `Actions` следующие операции: Эти действия позволяют пользователю просматривать все объекты в подписке, а также создавать запросы в службу поддержки.

    ```
    "*/read",
    "Microsoft.Support/*"
    ```

1. Получите идентификатор подписки с помощью команды [az account list](/cli/azure/account#az_account_list).

    ```azurecli
    az account list --output table
    ```

1. В `AssignableScopes` замените `{subscriptionId1}` идентификатором своей подписки.

    Вам нужно добавить явные идентификаторы подписки, иначе вы не сможете импортировать роль в подписку.

1. Измените свойства `Name` и `Description` на "Запросы в службу поддержки от читателя" и "Просмотр всех объектов в подписке и создание запросов в службу поддержки".

    Файл JSON должен выглядеть примерно так:

    ```json
    {
      "Name": "Reader Support Tickets",
      "IsCustom": true,
      "Description": "View everything in the subscription and also open support tickets.",
      "Actions": [
        "*/read",
        "Microsoft.Support/*"
      ],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/subscriptions/00000000-0000-0000-0000-000000000000"
      ]
    }
    ```
    
1. Чтобы создать пользовательскую роль, используйте команду [az role definition create](/cli/azure/role/definition#az_role_definition_create) и укажите файл определения роли JSON.

    ```azurecli
    az role definition create --role-definition "~/CustomRoles/ReaderSupportRole.json"
    ```

    ```Output
    {
      "additionalProperties": {},
      "assignableScopes": [
        "/subscriptions/00000000-0000-0000-0000-000000000000"
      ],
      "description": "View everything in the subscription and also open support tickets.",
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleDefinitions/22222222-2222-2222-2222-222222222222",
      "name": "22222222-2222-2222-2222-222222222222",
      "permissions": [
        {
          "actions": [
            "*/read",
            "Microsoft.Support/*"
          ],
          "additionalProperties": {},
          "dataActions": [],
          "notActions": [],
          "notDataActions": []
        }
      ],
      "roleName": "Reader Support Tickets",
      "roleType": "CustomRole",
      "type": "Microsoft.Authorization/roleDefinitions"
    }
    ```

    Новая пользовательская роль станет доступной на портале Azure. Ее можно назначать пользователям, группам или субъектам-службам, так же как и встроенные роли.

## <a name="list-custom-roles"></a>Вывод списка настраиваемых ролей

- Получить список всех пользовательских ролей можно с помощью команды [az role definition list](/cli/azure/role/definition#az_role_definition_list) с параметром `--custom-role-only`.

    ```azurecli
    az role definition list --custom-role-only true
    ```
    
    ```Output
    [
      {
        "additionalProperties": {},
        "assignableScopes": [
          "/subscriptions/00000000-0000-0000-0000-000000000000"
        ],
        "description": "View everything in the subscription and also open support tickets.",
        "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleDefinitions/22222222-2222-2222-2222-222222222222",
        "name": "22222222-2222-2222-2222-222222222222",
        "permissions": [
          {
            "actions": [
              "*/read",
              "Microsoft.Support/*",
              "Microsoft.Resources/deployments/*",
              "Microsoft.Insights/diagnosticSettings/*/read"
            ],
            "additionalProperties": {},
            "dataActions": [],
            "notActions": [],
            "notDataActions": []
          }
        ],
        "roleName": "Reader Support Tickets",
        "roleType": "CustomRole",
        "type": "Microsoft.Authorization/roleDefinitions"
      }
    ]
    ```
    
    Пользовательскую роль также можно просмотреть на портале Azure.

    ![Снимок экрана пользовательской роли, импортированной на портал Azure](./media/tutorial-custom-role-cli/custom-role-reader-support-tickets.png)

## <a name="update-a-custom-role"></a>Обновление пользовательской роли

Чтобы обновить пользовательскую роль, обновите файл JSON, а затем саму роль.

1. Откройте файл ReaderSupportRole.json.

1. В `Actions` добавьте операцию для создания и администрирования развертываний группы ресурсов `"Microsoft.Resources/deployments/*"`. Не забудьте добавить запятую после предыдущей операции.

    Обновленный файл JSON должен выглядеть примерно так:

    ```json
    {
      "Name": "Reader Support Tickets",
      "IsCustom": true,
      "Description": "View everything in the subscription and also open support tickets.",
      "Actions": [
        "*/read",
        "Microsoft.Support/*",
        "Microsoft.Resources/deployments/*"
      ],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/subscriptions/00000000-0000-0000-0000-000000000000"
      ]
    }
    ```
        
1. Чтобы обновить пользовательскую роль, используйте команду [az role definition update](/cli/azure/role/definition#az_role_definition_update) и укажите обновленный файл JSON.

    ```azurecli
    az role definition update --role-definition "~/CustomRoles/ReaderSupportRole.json"
    ```

    ```Output
    {
      "additionalProperties": {},
      "assignableScopes": [
        "/subscriptions/00000000-0000-0000-0000-000000000000"
      ],
      "description": "View everything in the subscription and also open support tickets.",
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleDefinitions/22222222-2222-2222-2222-222222222222",
      "name": "22222222-2222-2222-2222-222222222222",
      "permissions": [
        {
          "actions": [
            "*/read",
            "Microsoft.Support/*",
            "Microsoft.Resources/deployments/*"
          ],
          "additionalProperties": {},
          "dataActions": [],
          "notActions": [],
          "notDataActions": []
        }
      ],
      "roleName": "Reader Support Tickets",
      "roleType": "CustomRole",
      "type": "Microsoft.Authorization/roleDefinitions"
    }
    ```
    
## <a name="delete-a-custom-role"></a>Удаление настраиваемой роли

- Используйте команду [az role definition delete](/cli/azure/role/definition#az_role_definition_delete) и укажите имя или идентификатор роли для удаления пользовательской роли.

    ```azurecli
    az role definition delete --name "Reader Support Tickets"
    ```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Создание или изменение настраиваемых ролей Azure с помощью Azure CLI](custom-roles-cli.md)
