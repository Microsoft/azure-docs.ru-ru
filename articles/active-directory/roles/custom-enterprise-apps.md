---
title: Создание пользовательских ролей для управления корпоративными приложениями в Azure Active Directory
description: Создание и назначение пользовательских ролей Azure AD для доступа к корпоративным приложениям в Azure Active Directory
services: active-directory
author: rolyon
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: how-to
ms.date: 11/04/2020
ms.author: rolyon
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1a76e2d37e9dcdd285a8608fdbfd715bfb834eb8
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103467754"
---
# <a name="create-custom-roles-to-manage-enterprise-apps-in-azure-active-directory"></a>Создание пользовательских ролей для управления корпоративными приложениями в Azure Active Directory

В этой статье объясняется, как создать настраиваемую роль с разрешениями на управление назначениями корпоративных приложений для пользователей и групп в Azure Active Directory (Azure AD). Сведения об элементах назначений ролей и значении терминов, таких как подтип, разрешение и набор свойств, см. в разделе [Обзор пользовательских ролей](custom-overview.md).

## <a name="enterprise-app-role-permissions"></a>Разрешения роли корпоративного приложения

В этой статье обсуждаются два разрешения корпоративного приложения. Во всех примерах используется разрешение UPDATE.

* Чтобы считать назначения пользователей и групп в области, предоставьте `microsoft.directory/servicePrincipals/appRoleAssignedTo/read` разрешение
* Чтобы управлять назначениями пользователей и групп в области, предоставьте `microsoft.directory/servicePrincipals/appRoleAssignedTo/update` разрешение

Предоставление разрешения на обновление дает назначенному пользователю возможность управлять назначениями пользователей и групп для корпоративных приложений. Область назначения пользователей и групп может быть предоставлена для одного приложения или предоставлена для всех приложений. При предоставлении на уровне всей Организации уполномоченный объект может управлять назначениями для всех приложений. Если выполняется на уровне приложения, уполномоченный объект может управлять назначениями только для указанного приложения.

Предоставление разрешения на обновление выполняется в два этапа:

1. Создание пользовательской роли с разрешениями `microsoft.directory/servicePrincipals/appRoleAssignedTo/update`
1. Предоставьте пользователям или группам разрешения на управление назначениями пользователей и групп для корпоративных приложений. Это можно сделать в том случае, если областью действия является уровень всей организации или одно приложение.

## <a name="use-the-azure-ad-admin-center"></a>Использование центра администрирования Azure AD

### <a name="create-a-new-custom-role"></a>Создание новой настраиваемой роли

>[!NOTE]
> Пользовательские роли создаются и управляются на уровне всей Организации и доступны только на странице обзора Организации.

1. Войдите в [центр администрирования Azure AD](https://aad.portal.azure.com) с правами администратора привилегированных ролей или глобального администратора в вашей организации.
1. Последовательно выберите **Azure Active Directory**, **Роли и администраторы** и **Новая пользовательская роль**.

    ![Добавление новой настраиваемой роли из списка ролей в Azure AD](./media/custom-enterprise-apps/new-custom-role.png)

1. На вкладке **Основные сведения** укажите в поле "Управление назначениями пользователей и групп" имя роли и "предоставление разрешений для управления назначениями пользователей и групп" для описания роли, а затем нажмите кнопку **Далее**.

    ![Укажите имя и описание настраиваемой роли](./media/custom-enterprise-apps/role-name-and-description.png)

1. На вкладке **разрешения** введите Microsoft. Directory/свойства serviceprincipals/appRoleAssignedTo/Update в поле поиска, а затем установите флажки рядом с нужными разрешениями и нажмите кнопку **Далее**.

    ![Добавление разрешений для пользовательской роли](./media/custom-enterprise-apps/role-custom-permissions.png)

1. На вкладке **Просмотр и создание** проверьте разрешения и нажмите кнопку **Создать**.

    ![Теперь можно создать настраиваемую роль.](./media/custom-enterprise-apps/role-custom-create.png)

### <a name="assign-the-role-to-a-user-using-the-azure-ad-portal"></a>Назначение роли пользователю с помощью портала Azure AD

1. Войдите в [центр администрирования Azure AD](https://aad.portal.azure.com) , используя разрешения роли администратора привилегированных ролей.
1. Выберите **Azure Active Directory**, а затем — **Роли и администраторы**.
1. Выберите **разрешения предоставление разрешений для управления ролью назначения пользователей и групп** .

    ![Открытие ролей и администраторов и поиск настраиваемой роли](./media/custom-enterprise-apps/select-custom-role.png)

1. Выберите **добавить назначение**, выберите нужного пользователя и нажмите кнопку **выбрать** , чтобы добавить назначение ролей пользователю.

    ![Добавление назначения пользовательской роли пользователю](./media/custom-enterprise-apps/assign-user-to-role.png)

#### <a name="assignment-tips"></a>Советы по назначению

* Чтобы предоставить разрешения исполнителям для управления пользователями и доступом к группам для всех корпоративных приложений в масштабах всей Организации, начните с списка **ролей и администраторов** всей Организации на странице **обзора** Azure AD для вашей организации.
* Чтобы предоставить разрешения исполнителям для управления пользователями и доступом к группе для конкретного корпоративного приложения, перейдите к этому приложению в Azure AD и откройте список **роли и администраторы** для этого приложения. Выберите новую настраиваемую роль и заполните назначение пользователя или группы. Исполнителям может управлять пользователями и доступом к группам только для конкретного приложения.
* Чтобы проверить назначение пользовательской роли, войдите как уполномоченный и откройте страницу **Пользователи и группы** приложения, чтобы убедиться, что включен параметр **Добавить пользователя** .

    ![Проверка разрешений пользователя](./media/custom-enterprise-apps/verify-user-permissions.png)

## <a name="use-azure-ad-powershell"></a>Использование Azure AD PowerShell

Дополнительные сведения см. в статьях [Создание и назначение пользовательской роли](custom-create.md) и [Назначение пользовательских ролей с областью действия ресурса с помощью PowerShell](custom-assign-powershell.md).

Сначала установите модуль Azure AD PowerShell из [коллекция PowerShell](https://www.powershellgallery.com/packages/AzureADPreview/2.0.0.17). Импортируйте модуль Azure AD PowerShell (предварительная версия) с помощью следующей команды:

```powershell
Import-Module -Name AzureADPreview
```

Чтобы убедиться в готовности модуля к использованию, сопоставьте версию, возвращенную следующей командой, с указанной здесь.

```powershell
Get-Module -Name AzureADPreview
  ModuleType Version      Name                         ExportedCommands
  ---------- ---------    ----                         ----------------
  Binary     2.0.0.115    AzureADPreview               {Add-AzureADAdministrati...}
```

### <a name="create-a-custom-role"></a>Создание пользовательской роли

Создайте роль с помощью следующего скрипта PowerShell:

```PowerShell
# Basic role information
$description = "Manage user and group assignments"
$displayName = "Can manage user and group assignments for Applications"
$templateId = (New-Guid).Guid

# Set of permissions to grant
$allowedResourceAction =@( "microsoft.directory/servicePrincipals/appRoleAssignedTo/update")
$resourceActions = @{'allowedResourceActions'= $allowedResourceAction}
$rolePermission = @{'resourceActions' = $resourceActions}
$rolePermissions = $rolePermission

# Create new custom admin role
$customRole = New-AzureADMSRoleDefinition -RolePermissions $rolePermissions -DisplayName $displayName -Description $description -TemplateId $templateId -IsEnabled $true
```

### <a name="assign-the-custom-role"></a>Назначение настраиваемой роли

Назначьте роль с помощью этого скрипта PowerShell.

```powershell
PowerShell
# Basic role information

$description = "Manage user and group assignments"
$displayName = "Can manage user and group assignments for Applications"
$templateId = (New-Guid).Guid

# Set of permissions to grant
$allowedResourceAction =
@(
    "microsoft.directory/servicePrincipals/appRoleAssignedTo/update"
)
$resourceActions = @{'allowedResourceActions'= $allowedResourceAction}
$rolePermission = @{'resourceActions' = $resourceActions}
$rolePermissions = $rolePermission

# Create new custom role
$customRole = New-AzureAdRoleDefinition -RolePermissions $rolePermissions -DisplayName $displayName -Description $description -TemplateId $templateId -IsEnabled $true
```

## <a name="use-the-microsoft-graph-api"></a>Использование API Microsoft Graph

Создайте настраиваемую роль, используя предоставленный пример в API Microsoft Graph. Дополнительные сведения см. в статьях [Создание и назначение пользовательской роли](custom-create.md) и [Назначение пользовательских ролей администратора с помощью API Microsoft Graph](custom-assign-graph.md).

Ниже приведен HTTP-запрос на создание настраиваемой роли.

```HTTP
POST
https://graph.microsoft.com/beta/roleManagement/directory/roleDefinitionsIsEnabled $true
{
    "description":"Can manage user and group assignments for Applications.",
    "displayName":" Manage user and group assignments",
    "isEnabled":true,
    "rolePermissions":
    [
        {
            "resourceActions":
            {
                "allowedResourceActions":
                [
                    "microsoft.directory/servicePrincipals/appRoleAssignedTo/update"
                ]
            },
            "condition":null
        }
    ],
    "templateId":"<PROVIDE NEW GUID HERE>",
    "version":"1"
}
```

### <a name="assign-the-custom-role-using-microsoft-graph-api"></a>Назначение настраиваемой роли с помощью API Microsoft Graph

Назначение роли объединяет идентификатор субъекта безопасности (который может быть пользователем или субъектом-службой), идентификатор определения роли и область ресурсов Azure AD. Дополнительные сведения об элементах назначения ролей см. в разделе [Общие сведения о пользовательских ролях](custom-overview.md) .

Отправьте HTTP-запрос на назначение настраиваемой роли.

```HTTP
POST https://graph.microsoft.com/beta/roleManagement/directory/roleAssignments

{
    "principalId":"<PROVIDE OBJECTID OF USER TO ASSIGN HERE>",
    "roleDefinitionId":"<PROVIDE OBJECTID OF ROLE DEFINITION HERE>",
    "resourceScopes":["/"]
}
```

## <a name="next-steps"></a>Следующие шаги

* [Просмотр доступных разрешений пользовательской роли для корпоративных приложений](custom-enterprise-app-permissions.md)
