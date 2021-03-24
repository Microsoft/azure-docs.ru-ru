---
title: Руководство по Создание настраиваемой роли Azure с помощью Azure PowerShell — Azure RBAC
description: Из этой статьи вы узнаете, как создать настраиваемую роль Azure с помощью Azure PowerShell и управления доступом на основе ролей Azure (Azure RBAC).
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
ms.openlocfilehash: 8b1815e7598410cd709572d93082d5dee5e0b0fb
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97369248"
---
# <a name="tutorial-create-an-azure-custom-role-using-azure-powershell"></a>Руководство по Создание настраиваемой роли Azure с помощью Azure PowerShell

Если [встроенные роли Azure](built-in-roles.md) не соответствуют потребностям вашей организации, вы можете создать собственные настраиваемые роли. С помощью этого руководства и Azure PowerShell вы создадите настраиваемую роль с именем "Запросы в службу поддержки от читателя". Она позволяет пользователю просматривать все объекты в плоскости управления подписки, а также делать запросы в службу поддержки.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создание настраиваемой роли
> * Вывод списка настраиваемых ролей
> * Обновление пользовательской роли
> * Удаление настраиваемой роли

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [az-powershell-update](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные требования

Для выполнения этого руководства потребуется следующее:

- разрешения на создание пользовательских ролей, такие как [Владелец](built-in-roles.md#owner) или [Администратор доступа пользователя](built-in-roles.md#user-access-administrator);
- [Azure Cloud Shell](../cloud-shell/overview.md) или [Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="sign-in-to-azure-powershell"></a>Вход в Azure PowerShell

Войдите в [Azure PowerShell](/powershell/azure/authenticate-azureps).

## <a name="create-a-custom-role"></a>Создание настраиваемой роли

Создать пользовательскую роль проще всего с помощью встроенной роли, в которую вы можете добавлять изменения для создания новой роли.

1. В PowerShell вы можете получить список операций, доступных поставщику ресурсов с помощью команды [Get-AzProviderOperation](/powershell/module/az.resources/get-azprovideroperation). Полезно знать операции, которые доступны для создания разрешений. Список всех операций см. в статье [Операции с поставщиками ресурсов Azure](resource-provider-operations.md#microsoftsupport).

    ```azurepowershell
    Get-AzProviderOperation "Microsoft.Support/*" | FT Operation, Description -AutoSize
    ```
    
    ```Output
    Operation                              Description
    ---------                              -----------
    Microsoft.Support/register/action      Registers to Support Resource Provider
    Microsoft.Support/supportTickets/read  Gets Support Ticket details (including status, severity, contact ...
    Microsoft.Support/supportTickets/write Creates or Updates a Support Ticket. You can create a Support Tic...
    ```

1. Используйте команду [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition), чтобы отобразить роль [Читатель](built-in-roles.md#reader) в формате JSON.

    ```azurepowershell
    Get-AzRoleDefinition -Name "Reader" | ConvertTo-Json | Out-File C:\CustomRoles\ReaderSupportRole.json
    ```

1. Откройте в редакторе файл **ReaderSupportRole.json**.

    Ниже приведены выходные данные JSON. См. дополнительные сведения о [различных свойствах настраиваемых ролей](custom-roles.md).

    ```json
    {
      "Name": "Reader",
      "Id": "acdd72a7-3385-48ef-bd42-f606fba81ae7",
      "IsCustom": false,
      "Description": "Lets you view everything, but not make any changes.",
      "Actions": [
        "*/read"
      ],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/"
      ]
    }
    ```
    
1. Измените файл JSON, добавив операцию `"Microsoft.Support/*"` в свойство `Actions`. Не забудьте включить запятую после операции чтения. Это действие разрешит пользователю создавать запросы в службу поддержки.

1. Получите идентификатор подписки с помощью команды [Get-AzSubscription](/powershell/module/Az.Accounts/Get-AzSubscription).

    ```azurepowershell
    Get-AzSubscription
    ```

1. В `AssignableScopes` добавьте свой идентификатор подписки в следующем формате: `"/subscriptions/00000000-0000-0000-0000-000000000000"`

    Вам нужно добавить явные идентификаторы подписки, иначе вы не сможете импортировать роль в подписку.

1. Удалите строку свойства `Id` и измените свойство `IsCustom` на `true`.

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
    
1. Чтобы создать пользовательскую роль, используйте команду [New-AzRoleDefinition](/powershell/module/az.resources/new-azroledefinition) и укажите файл определения роли JSON.

    ```azurepowershell
    New-AzRoleDefinition -InputFile "C:\CustomRoles\ReaderSupportRole.json"
    ```

    ```Output
    Name             : Reader Support Tickets
    Id               : 22222222-2222-2222-2222-222222222222
    IsCustom         : True
    Description      : View everything in the subscription and also open support tickets.
    Actions          : {*/read, Microsoft.Support/*}
    NotActions       : {}
    DataActions      : {}
    NotDataActions   : {}
    AssignableScopes : {/subscriptions/00000000-0000-0000-0000-000000000000}
    ```
        
    Новая пользовательская роль станет доступной на портале Azure. Ее можно назначать пользователям, группам или субъектам-службам, как и встроенные роли.

## <a name="list-custom-roles"></a>Вывод списка настраиваемых ролей

- Получить список всех пользовательских ролей можно с помощью команды [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition).

    ```azurepowershell
    Get-AzRoleDefinition | ? {$_.IsCustom -eq $true} | FT Name, IsCustom
    ```

    ```Output
    Name                   IsCustom
    ----                   --------
    Reader Support Tickets     True
    ```
    
    Пользовательскую роль также можно просмотреть на портале Azure.

    ![Снимок экрана пользовательской роли, импортированной на портал Azure](./media/tutorial-custom-role-powershell/custom-role-reader-support-tickets.png)

## <a name="update-a-custom-role"></a>Обновление пользовательской роли

Чтобы обновить пользовательскую роль, обновите файл JSON или используйте объект `PSRoleDefinition`.

1. Используйте команду [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition), чтобы обновить файл JSON и отобразить пользовательскую роль в формате JSON.

    ```azurepowershell
    Get-AzRoleDefinition -Name "Reader Support Tickets" | ConvertTo-Json | Out-File C:\CustomRoles\ReaderSupportRole2.json
    ```

1. Откройте файл в редакторе.

1. В `Actions` добавьте операцию для создания и администрирования развертываний группы ресурсов `"Microsoft.Resources/deployments/*"`.

    Обновленный файл JSON должен выглядеть примерно так:

    ```json
    {
      "Name": "Reader Support Tickets",
      "Id": "22222222-2222-2222-2222-222222222222",
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
        
1. Чтобы обновить пользовательскую роль, используйте команду [Set-AzRoleDefinition](/powershell/module/az.resources/set-azroledefinition) и укажите обновленный файл JSON.

    ```azurepowershell
    Set-AzRoleDefinition -InputFile "C:\CustomRoles\ReaderSupportRole2.json"
    ```

    ```Output
    Name             : Reader Support Tickets
    Id               : 22222222-2222-2222-2222-222222222222
    IsCustom         : True
    Description      : View everything in the subscription and also open support tickets.
    Actions          : {*/read, Microsoft.Support/*, Microsoft.Resources/deployments/*}
    NotActions       : {}
    DataActions      : {}
    NotDataActions   : {}
    AssignableScopes : {/subscriptions/00000000-0000-0000-0000-000000000000}
    ```

1. Чтобы с помощью объекта `PSRoleDefintion` обновить пользовательскую роль, сначала получите эту роль, используя команду [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition).

    ```azurepowershell
    $role = Get-AzRoleDefinition "Reader Support Tickets"
    ```
    
1. Вызовите метод `Add`, чтобы добавить операцию для чтения параметров диагностики.

    ```azurepowershell
    $role.Actions.Add("Microsoft.Insights/diagnosticSettings/*/read")
    ```

1. Обновите роль с помощью команды [Set-AzRoleDefinition](/powershell/module/az.resources/set-azroledefinition).

    ```azurepowershell
    Set-AzRoleDefinition -Role $role
    ```
    
    ```Output
    Name             : Reader Support Tickets
    Id               : 22222222-2222-2222-2222-222222222222
    IsCustom         : True
    Description      : View everything in the subscription and also open support tickets.
    Actions          : {*/read, Microsoft.Support/*, Microsoft.Resources/deployments/*,
                       Microsoft.Insights/diagnosticSettings/*/read}
    NotActions       : {}
    DataActions      : {}
    NotDataActions   : {}
    AssignableScopes : {/subscriptions/00000000-0000-0000-0000-000000000000}
    ```
    
## <a name="delete-a-custom-role"></a>Удаление настраиваемой роли

1. Получите идентификатор пользовательской роли с помощью команды [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition).

    ```azurepowershell
    Get-AzRoleDefinition "Reader Support Tickets"
    ```

1. Укажите идентификатор пользовательской роли, чтобы удалить пользовательскую роль, с помощью команды [Remove-AzRoleDefinition](/powershell/module/az.resources/remove-azroledefinition).

    ```azurepowershell
    Remove-AzRoleDefinition -Id "22222222-2222-2222-2222-222222222222"
    ```

    ```Output
    Confirm
    Are you sure you want to remove role definition with id '22222222-2222-2222-2222-222222222222'.
    [Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"):
    ```

1. В запросе на подтверждение выберите **Да**.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Создание или изменение настраиваемых ролей Azure с помощью Azure PowerShell](custom-roles-powershell.md)
