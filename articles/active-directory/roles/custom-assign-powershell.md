---
title: Назначение пользовательских ролей с помощью Azure AD PowerShell в Azure AD | Документация Майкрософт
description: Управление членами настраиваемой роли администратора Azure AD с помощью Azure AD PowerShell.
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
ms.openlocfilehash: 9f0fb81a4daa57b473e8b2b4b937426eafbf903d
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103014542"
---
# <a name="assign-custom-roles-with-resource-scope-using-powershell-in-azure-active-directory"></a>Назначение пользовательских ролей с областью действия ресурса с помощью PowerShell в Azure Active Directory

В этой статье описывается, как создать назначение роли в масштабе всей Организации в Azure Active Directory (Azure AD). Назначение роли в масштабе всей организации предоставляет доступ в рамках всей Организации Azure AD. Сведения о создании назначения ролей с областью действия одного ресурса Azure AD см. в статье [Создание настраиваемой роли и ее назначение в области ресурсов](custom-create.md). В этой статье используется модуль [Azure Active Directory PowerShell версии 2](/powershell/module/azuread/#directory_roles) .

Дополнительные сведения о ролях администратора Azure AD см. [в разделе Назначение ролей администратора в Azure Active Directory](permissions-reference.md).

## <a name="required-permissions"></a>Необходимые разрешения

Подключитесь к Организации Azure AD с помощью учетной записи глобального администратора, чтобы назначить или удалить роли.

## <a name="prepare-powershell"></a>Подготовка PowerShell

Установите модуль Azure AD PowerShell из [коллекция PowerShell](https://www.powershellgallery.com/packages/AzureADPreview). Импортируйте модуль Azure AD PowerShell (предварительная версия) с помощью следующей команды:

``` PowerShell
Import-Module -Name AzureADPreview
```

Чтобы убедиться в готовности модуля к использованию, сопоставьте версию, возвращенную следующей командой, с указанной здесь.

``` PowerShell
Get-Module -Name AzureADPreview
  ModuleType Version      Name                         ExportedCommands
  ---------- ---------    ----                         ----------------
  Binary     2.0.0.115    AzureADPreview               {Add-AzureADMSAdministrati...}
```

Теперь можно начать использование командлетов в модуле. Полное описание командлетов в модуле Azure AD см. в интерактивной справочной документации по [модулю Azure AD Preview](https://www.powershellgallery.com/packages/AzureADPreview).

## <a name="assign-a-directory-role-to-a-user-or-service-principal-with-resource-scope"></a>Назначение роли каталога пользователю или субъекту-службе с областью действия ресурса

1. Загрузите модуль Azure AD PowerShell (Предварительная версия).
1. Выполните вход, выполнив команду `Connect-AzureAD` .
1. Создайте новую роль, используя следующий сценарий PowerShell.

``` PowerShell
## Assign a role to a user or service principal with resource scope
# Get the user and role definition you want to link
$user = Get-AzureADUser -Filter "userPrincipalName eq 'cburl@f128.info'"
$roleDefinition = Get-AzureADMSRoleDefinition -Filter "displayName eq 'Application Support Administrator'"

# Get app registration and construct resource scope for assignment.
$appRegistration = Get-AzureADApplication -Filter "displayName eq 'f/128 Filter Photos'"
$resourceScope = '/' + $appRegistration.objectId

# Create a scoped role assignment
$roleAssignment = New-AzureADMSRoleAssignment -ResourceScope $resourceScope -RoleDefinitionId $roleDefinition.Id -PrincipalId $user.objectId
```

Чтобы назначить роль субъекту-службе, а не пользователю, используйте командлет [Get-азуреадмссервицепринЦипал](/powershell/module/azuread/get-azureadserviceprincipal) .

## <a name="role-definitions"></a>Определения ролей

Объекты определения ролей содержат определение встроенной или пользовательской роли вместе с разрешениями, предоставленными этим назначением роли. В этом ресурсе отображаются пользовательские определения ролей и встроенные роли каталога (которые отображаются в эквивалентной форме определения роли). Сейчас в Организации Azure AD может быть определено не более 30 уникальных определений пользовательских ролей.

### <a name="create-a-role-definition"></a>Создание определения роли

``` PowerShell
# Basic information
$description = "Can manage credentials of application registrations"
$displayName = "Application Registration Credential Administrator"
$templateId = (New-Guid).Guid

# Set of actions to include
$rolePermissions = @{
    "allowedResourceActions" = @(
        "microsoft.directory/applications/standard/read",
        "microsoft.directory/applications/credentials/update"
    )
}

# Create new custom directory role
$customAdmin = New-AzureADMSRoleDefinition -RolePermissions $rolePermissions -DisplayName $displayName -Description $description -TemplateId $templateId -IsEnabled $true
```

### <a name="read-and-list-role-definitions"></a>Чтение и перечисление определений ролей

``` PowerShell
# Get all role definitions
Get-AzureADMSRoleDefinitions

# Get single role definition by ID
Get-AzureADMSRoleDefinition -Id 86593cfc-114b-4a15-9954-97c3494ef49b

# Get single role definition by templateId
Get-AzureADMSRoleDefinition -Filter "templateId eq 'c4e39bd9-1100-46d3-8c65-fb160da0071f'"
```

### <a name="update-a-role-definition"></a>Обновление определения роли

``` PowerShell
# Update role definition
# This works for any writable property on role definition. You can replace display name with other
# valid properties.
Set-AzureADMSRoleDefinition -Id c4e39bd9-1100-46d3-8c65-fb160da0071f -DisplayName "Updated DisplayName"
```

### <a name="delete-a-role-definition"></a>Удаление определения роли

``` PowerShell
# Delete role definition
Remove-AzureADMSRoleDefinitions -Id c4e39bd9-1100-46d3-8c65-fb160da0071f
```

## <a name="role-assignments"></a>Назначения ролей

Назначения ролей содержат сведения, связывающие указанного участника безопасности (субъекта-службы) с определением роли. При необходимости можно добавить область одного ресурса Azure AD для назначенных разрешений.  Ограничение области назначения ролей поддерживается для встроенных и пользовательских ролей.

### <a name="create-a-role-assignment"></a>Создание назначения роли

``` PowerShell
# Get the user and role definition you want to link
$user = Get-AzureADUser -Filter "userPrincipalName eq 'cburl@f128.info'"
$roleDefinition = Get-AzureADMSRoleDefinition -Filter "displayName eq 'Application Support Administrator'"

# Get app registration and construct resource scope for assignment.
$appRegistration = Get-AzureADApplication -Filter "displayName eq 'f/128 Filter Photos'"
$resourceScope = '/' + $appRegistration.objectId

# Create a scoped role assignment
$roleAssignment = New-AzureADMSRoleAssignment -ResourceScope $resourceScope -RoleDefinitionId $roleDefinition.Id -PrincipalId $user.objectId
```

### <a name="read-and-list-role-assignments"></a>Чтение и перечисление назначений ролей

``` PowerShell
# Get role assignments for a given principal
Get-AzureADMSRoleAssignment -Filter "principalId eq '27c8ca78-ab1c-40ae-bd1b-eaeebd6f68ac'"

# Get role assignments for a given role definition 
Get-AzureADMSRoleAssignment -Filter "roleDefinitionId eq '355aed8a-864b-4e2b-b225-ea95482e7570'"
```

### <a name="delete-a-role-assignment"></a>Удаление назначения ролей

``` PowerShell
# Delete role assignment
Remove-AzureADMSRoleAssignment -Id 'qiho4WOb9UKKgng_LbPV7tvKaKRCD61PkJeKMh7Y458-1'
```

## <a name="next-steps"></a>Дальнейшие действия

- Поделитесь с нами на [форуме по административным ролям Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=166032)
- Дополнительные сведения о ролях и назначениях ролей администратора Azure AD см. в статье [назначение ролей администратора](permissions-reference.md) .
- Сведения о разрешениях пользователей по умолчанию см. в разделе [Сравнение разрешений гостей по умолчанию и пользователя-участника](../fundamentals/users-default-permissions.md) .
