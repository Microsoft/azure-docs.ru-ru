---
title: Повышение прав доступа для управления всеми подписками Azure и группами управления
description: Сведения о том, как с помощью портала Azure или REST API повысить права доступа глобального администратора для управления всеми подписками и группами управления в Azure Active Directory.
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.topic: how-to
ms.workload: identity
ms.date: 06/09/2020
ms.author: rolyon
ms.openlocfilehash: efc3bfef7c182ea005ac17a59793ac8f6484a424
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102042017"
---
# <a name="elevate-access-to-manage-all-azure-subscriptions-and-management-groups"></a>Повышение прав доступа для управления всеми подписками Azure и группами управления

Даже если вы являетесь глобальным администратором в Azure Active Directory (Azure AD), у вас может не быть права доступа ко всем подпискам и группам управления в вашем каталоге. В этой статье описываются способы, с помощью которых вы можете повысить права доступа для всех подписок и групп управления.

[!INCLUDE [gdpr-dsr-and-stp-note](../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="why-would-you-need-to-elevate-your-access"></a>Для чего нужно повышение прав доступа

Если вы являетесь глобальным администратором, могут возникнуть ситуации, когда необходимо выполнить следующие действия.

- восстановление доступа к подписке Azure или группе управления, если пользователь потерял доступ;
- предоставление другому пользователю или себе доступа к подписке Azure или группе управления;
- просмотр всех подписок Azure или групп управления в организации;
- предоставление приложению автоматизации (например, приложению для выставления счетов или аудита) доступа ко всем подпискам Azure или группам управления.

## <a name="how-does-elevated-access-work"></a>Как работает повышение прав доступа?

Azure AD и ресурсы Azure защищены независимо друг от друга. То есть назначения ролей Azure AD не предоставляют доступ к ресурсам Azure, а назначения ролей Azure не предоставляют доступ к Azure AD. Однако, если вы являетесь [глобальным администратором](../active-directory/roles/permissions-reference.md#global-administrator) в Azure AD, вы можете назначить себе доступ ко всем подпискам Azure и группам управления в каталоге. Эту возможность следует использовать, если у вас нет доступа к ресурсам в подписке Azure, например к виртуальным машинам или учетным записям хранения, и вы хотите использовать свои привилегии глобального администратора для получения доступа к этим ресурсам.

При повышении прав доступа вам будет назначена роль [администратора доступа пользователей](built-in-roles.md#user-access-administrator) в Azure, охватывающая корень каталога (`/`).Это позволяет просматривать все ресурсы и назначать доступ в любой подписке или группе управления в каталоге. Назначения ролей администратора доступа пользователей можно удалить с помощью Azure PowerShell, Azure CLI или REST API.

После внесения изменений, которые необходимо выполнить на уровне корня, следует удалить этот повышенный уровень доступа.

![Повышение прав доступа](./media/elevate-access-global-admin/elevate-access.png)

## <a name="azure-portal"></a>Портал Azure

### <a name="elevate-access-for-a-global-administrator"></a>Повышение прав доступа глобального администратора

Выполните следующие действия, чтобы повысить права доступа глобального администратора с помощью портала Azure.

1. Войдите на [портал Azure](https://portal.azure.com) или [центр администрирования Azure Active Directory](https://aad.portal.azure.com) как глобальный администратор.

    Если вы используете Azure AD Privileged Identity Management, [активируйте назначение роли глобального администратора](../active-directory/privileged-identity-management/pim-how-to-activate-role.md).

1. Откройте **Azure Active Directory**.

1. В разделе **Управление** выберите **Свойства**.

   ![Выберите свойства для Azure Active Directory свойства — снимок экрана](./media/elevate-access-global-admin/azure-active-directory-properties.png)

1. В разделе **Управление доступом для ресурсов Azure** установите значение **Да**.

   ![Снимок экрана: управление доступом для ресурсов Azure](./media/elevate-access-global-admin/aad-properties-global-admin-setting.png)

   Если установить переключатель в положение **Да**, вы назначаете роль администратора доступа пользователей в Azure RBAC в корневой области (/). Это предоставляет разрешение на назначение ролей во всех подписках и группах управления Azure, связанных с данным каталогом Azure AD. Этот переключатель доступен только пользователям, которым назначена роль глобального администратора в Azure AD.

   Если для переключателя задать значение **Нет**, роль администратора доступа пользователей в Azure RBAC будет удалена из вашей учетной записи. Вы больше не сможете назначать роли во всех подписках и группах управления Azure, связанных с данным каталогом Azure AD. Вы сможете просматривать (и управлять ими) только подписки и группы управления Azure, к которым вам был предоставлен доступ.

    > [!NOTE]
    > Если вы используете [Управление привилегированными пользователями](../active-directory/privileged-identity-management/pim-configure.md), деактивация назначения ролей не приводит к изменению ключа **управления доступом для ресурсов Azure** **.** Чтобы обеспечить минимальный привилегированный доступ, рекомендуется установить этот переключатель в положение **нет** , прежде чем отключить назначение ролей.
    
1. Нажмите кнопку **Сохранить**, чтобы сохранить настройки.

   Этот параметр не является глобальным свойством и применяется только к текущему выполнившему вход пользователю. Вы не можете повысить права доступа для всех членов роли глобального администратора.

1. Выйдите из системы и войдите снова, чтобы обновить доступ.

    Теперь у вас есть доступ ко всем подпискам и группам управления в своем каталоге. При просмотре панели управления доступом (IAM) вы заметите, что роль администратора доступа пользователей была назначена в корневой области.

   ![Назначения ролей подписки с помощью корневой области. Снимок экрана](./media/elevate-access-global-admin/iam-root.png)

1. Внесите необходимые изменения при повышенном доступе.

    Сведения о назначении ролей см. [в статье назначение ролей Azure с помощью портал Azure](role-assignments-portal.md). Если вы используете управление привилегированными пользователями, см. статью [обнаружение ресурсов Azure для управления](../active-directory/privileged-identity-management/pim-resource-roles-discover-resources.md) и [назначения ролей ресурсов Azure](../active-directory/privileged-identity-management/pim-resource-roles-assign-roles.md).

1. Выполните действия, описанные в следующем разделе, чтобы удалить доступ с повышенными правами.

### <a name="remove-elevated-access"></a>Удаление повышенного права доступа

Чтобы удалить назначение роли администратора доступа пользователей в корневой области ( `/` ), выполните следующие действия.

1. Войдите в систему как пользователь, который использовался для повышения уровня доступа.

1. В списке навигации выберите **Azure Active Directory**, а затем нажмите кнопку **Свойства**.

1. Для параметра **Управление доступом для ресурсов Azure** переключитесь обратно на **нет**. Так как это параметр для отдельного пользователя, необходимо войти как тот же пользователь, который использовался для повышения доступа.

    Если вы попытаетесь удалить назначение роли администратора доступа пользователей на панели управления доступом (IAM), вы увидите следующее сообщение. Чтобы удалить назначение ролей, необходимо установить переключатель в положение **нет** или использовать Azure PowerShell, Azure CLI или REST API.

    ![Удаление назначений ролей с корневой областью](./media/elevate-access-global-admin/iam-root-remove.png)

1. Выйдите от имени глобального администратора.

    Если вы используете управление привилегированными пользователями, отключите назначение роли глобального администратора.

    > [!NOTE]
    > Если вы используете [Управление привилегированными пользователями](../active-directory/privileged-identity-management/pim-configure.md), деактивация назначения ролей не приводит к изменению ключа **управления доступом для ресурсов Azure** **.** Чтобы обеспечить минимальный привилегированный доступ, рекомендуется установить этот переключатель в положение **нет** , прежде чем отключить назначение ролей.

## <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [az-powershell-update](../../includes/updated-for-az.md)]

### <a name="list-role-assignment-at-root-scope-"></a>Вывод списка назначений ролей в корневой области (/)

Чтобы получить список назначения ролей администратора доступа пользователей в корневой области ( `/` ), используйте команду [Get-азролеассигнмент](/powershell/module/az.resources/get-azroleassignment) .

```azurepowershell
Get-AzRoleAssignment | where {$_.RoleDefinitionName -eq "User Access Administrator" `
  -and $_.SignInName -eq "<username@example.com>" -and $_.Scope -eq "/"}
```

```Example
RoleAssignmentId   : /providers/Microsoft.Authorization/roleAssignments/11111111-1111-1111-1111-111111111111
Scope              : /
DisplayName        : username
SignInName         : username@example.com
RoleDefinitionName : User Access Administrator
RoleDefinitionId   : 18d7d88d-d35e-4fb5-a5c3-7773c20a72d9
ObjectId           : 22222222-2222-2222-2222-222222222222
ObjectType         : User
CanDelegate        : False
```

### <a name="remove-elevated-access"></a>Удаление повышенного права доступа

Чтобы удалить назначение роли администратора доступа пользователей для себя или другого пользователя в корневой области ( `/` ), выполните следующие действия.

1. Войдите в систему как пользователь, который может удалить повышенный уровень доступа. Это может быть тот же пользователь, который использовался для повышения уровня доступа или другого глобального администратора с повышенными правами доступа в корневой области.

1. Чтобы удалить назначение роли администратора доступа пользователей, воспользуйтесь командой [Remove-AzRoleAssignment](/powershell/module/az.resources/remove-azroleassignment).

    ```azurepowershell
    Remove-AzRoleAssignment -SignInName <username@example.com> `
      -RoleDefinitionName "User Access Administrator" -Scope "/"
    ```

## <a name="azure-cli"></a>Azure CLI

### <a name="elevate-access-for-a-global-administrator"></a>Повышение прав доступа глобального администратора

Выполните следующие основные действия, чтобы повысить уровень доступа для глобального администратора с помощью Azure CLI.

1. Используйте команду [AZ RESTful](/cli/azure/reference-index#az_rest) , чтобы вызвать `elevateAccess` конечную точку, которая предоставляет роль администратора доступа пользователей в корневой области ( `/` ).

    ```azurecli
    az rest --method post --url "/providers/Microsoft.Authorization/elevateAccess?api-version=2016-07-01"
    ```

1. Внесите необходимые изменения при повышенном доступе.

    Сведения о назначении ролей см. [в статье назначение ролей Azure с помощью Azure CLI](role-assignments-cli.md).

1. Выполните действия, описанные в следующем разделе, чтобы удалить доступ с повышенными правами.

### <a name="list-role-assignment-at-root-scope-"></a>Вывод списка назначений ролей в корневой области (/)

Чтобы получить список назначения ролей администратора доступа пользователей в корневой области ( `/` ), используйте команду [AZ Role назначение List](/cli/azure/role/assignment#az-role-assignment-list) .

```azurecli
az role assignment list --role "User Access Administrator" --scope "/"
```

```Example
[
  {
    "canDelegate": null,
    "id": "/providers/Microsoft.Authorization/roleAssignments/11111111-1111-1111-1111-111111111111",
    "name": "11111111-1111-1111-1111-111111111111",
    "principalId": "22222222-2222-2222-2222-222222222222",
    "principalName": "username@example.com",
    "principalType": "User",
    "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/18d7d88d-d35e-4fb5-a5c3-7773c20a72d9",
    "roleDefinitionName": "User Access Administrator",
    "scope": "/",
    "type": "Microsoft.Authorization/roleAssignments"
  }
]

```

### <a name="remove-elevated-access"></a>Удаление повышенного права доступа

Чтобы удалить назначение роли администратора доступа пользователей для себя или другого пользователя в корневой области ( `/` ), выполните следующие действия.

1. Войдите в систему как пользователь, который может удалить повышенный уровень доступа. Это может быть тот же пользователь, который использовался для повышения уровня доступа или другого глобального администратора с повышенными правами доступа в корневой области.

1. Используйте команду [AZ Role назначение Delete](/cli/azure/role/assignment#az-role-assignment-delete) , чтобы удалить назначение роли администратора доступа пользователей.

    ```azurecli
    az role assignment delete --assignee username@example.com --role "User Access Administrator" --scope "/"
    ```

## <a name="rest-api"></a>REST API

### <a name="elevate-access-for-a-global-administrator"></a>Повышение прав доступа глобального администратора

Чтобы повысить права доступа глобального администратора с помощью REST API, выполните приведенные ниже основные действия.

1. Используя функцию RESTFUL, вызовите метод `elevateAccess` , который предоставляет пользователю роль администратора доступа пользователей в корневой области ( `/` ).

   ```http
   POST https://management.azure.com/providers/Microsoft.Authorization/elevateAccess?api-version=2016-07-01
   ```

1. Внесите необходимые изменения при повышенном доступе.

    Сведения о назначении ролей см. [в статье назначение ролей Azure с помощью REST API](role-assignments-rest.md).

1. Выполните действия, описанные в следующем разделе, чтобы удалить доступ с повышенными правами.

### <a name="list-role-assignments-at-root-scope-"></a>Вывод списка назначений ролей в корневой области (/)

Вы можете получить список всех назначений ролей для пользователя в корневой области ( `/` ).

- Вызовите [GET roleAssignments](/rest/api/authorization/roleassignments/listforscope), где `{objectIdOfUser}` — идентификатор объекта пользователя, чьи назначения роли вы хотите получить.

   ```http
   GET https://management.azure.com/providers/Microsoft.Authorization/roleAssignments?api-version=2015-07-01&$filter=principalId+eq+'{objectIdOfUser}'
   ```

### <a name="list-deny-assignments-at-root-scope-"></a>Список запрещенных назначений в корневой области (/)

Можно перечислить все назначения Deny для пользователя в корневой области ( `/` ).

- Вызовите GET denyAssignments, где `{objectIdOfUser}` — идентификатор объекта пользователя, запреты назначений которого вы хотите получить.

   ```http
   GET https://management.azure.com/providers/Microsoft.Authorization/denyAssignments?api-version=2018-07-01-preview&$filter=gdprExportPrincipalId+eq+'{objectIdOfUser}'
   ```

### <a name="remove-elevated-access"></a>Удаление повышенного права доступа

При вызове `elevateAccess` вы создаете назначение ролей для себя, поэтому для отзыва этих привилегий необходимо удалить назначение роли администратора доступа пользователей в корневой области ( `/` ).

1. Вызовите [GET roleDefinitions](/rest/api/authorization/roledefinitions/get) (где `roleName` — это администратор доступа пользователей), чтобы определить GUID имени роли этого администратора.

    ```http
    GET https://management.azure.com/providers/Microsoft.Authorization/roleDefinitions?api-version=2015-07-01&$filter=roleName+eq+'User Access Administrator'
    ```

    ```json
    {
      "value": [
        {
          "properties": {
      "roleName": "User Access Administrator",
      "type": "BuiltInRole",
      "description": "Lets you manage user access to Azure resources.",
      "assignableScopes": [
        "/"
      ],
      "permissions": [
        {
          "actions": [
            "*/read",
            "Microsoft.Authorization/*",
            "Microsoft.Support/*"
          ],
          "notActions": []
        }
      ],
      "createdOn": "0001-01-01T08:00:00.0000000Z",
      "updatedOn": "2016-05-31T23:14:04.6964687Z",
      "createdBy": null,
      "updatedBy": null
          },
          "id": "/providers/Microsoft.Authorization/roleDefinitions/18d7d88d-d35e-4fb5-a5c3-7773c20a72d9",
          "type": "Microsoft.Authorization/roleDefinitions",
          "name": "18d7d88d-d35e-4fb5-a5c3-7773c20a72d9"
        }
      ],
      "nextLink": null
    }
    ```

    Сохраните ИД из параметра `name`. В этом случае — `18d7d88d-d35e-4fb5-a5c3-7773c20a72d9`.

1. Также необходимо вывести назначений ролей для администратора каталога в области каталога. Выведите список всех назначений в области каталога для `principalId` администратора каталога, совершившего вызов для повышения прав доступа. При этом будут перечислены все назначения в каталоге для objectid.

    ```http
    GET https://management.azure.com/providers/Microsoft.Authorization/roleAssignments?api-version=2015-07-01&$filter=principalId+eq+'{objectid}'
    ```
        
    >[!NOTE] 
    >У администратора каталога не должно быть много назначений. Если предыдущий запрос возвращает слишком много назначений, можно запросить все назначения только на уровне области каталога, а затем отфильтровать результаты: `GET https://management.azure.com/providers/Microsoft.Authorization/roleAssignments?api-version=2015-07-01&$filter=atScope()`.
            
1. Предыдущие вызовы возвращают список назначения ролей. Найдите назначение роли, у которого задана область `"/"` и `roleDefinitionId` заканчивается значением ИД имени роли, найденным на шаге 1, а `principalId` совпадает со значением objectId администратора каталога. 
    
    Пример назначения ролей:
    
    ```json
    {
      "value": [
        {
          "properties": {
            "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/18d7d88d-d35e-4fb5-a5c3-7773c20a72d9",
            "principalId": "{objectID}",
            "scope": "/",
            "createdOn": "2016-08-17T19:21:16.3422480Z",
            "updatedOn": "2016-08-17T19:21:16.3422480Z",
            "createdBy": "22222222-2222-2222-2222-222222222222",
            "updatedBy": "22222222-2222-2222-2222-222222222222"
          },
          "id": "/providers/Microsoft.Authorization/roleAssignments/11111111-1111-1111-1111-111111111111",
          "type": "Microsoft.Authorization/roleAssignments",
          "name": "11111111-1111-1111-1111-111111111111"
        }
      ],
      "nextLink": null
    }
    ```
    
    Опять же, сохраните идентификатор из `name` параметра, в данном случае 11111111-1111-1111-1111-111111111111.

1. Наконец, используйте ИД назначения роли, чтобы удалить назначение, добавленное `elevateAccess`:

    ```http
    DELETE https://management.azure.com/providers/Microsoft.Authorization/roleAssignments/11111111-1111-1111-1111-111111111111?api-version=2015-07-01
    ```

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения о различных ролях](rbac-and-directory-admin-roles.md)
- [Назначение ролей Azure с помощью REST API](role-assignments-rest.md)
