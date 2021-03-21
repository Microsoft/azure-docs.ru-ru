---
title: Удаление назначений ролей из группы в Azure Active Directory | Документация Майкрософт
description: Предварительная версия настраиваемых ролей Azure AD для делегирования управления удостоверениями. Управляйте ролями Azure на портале Azure, в PowerShell или API Graph.
services: active-directory
author: rolyon
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: article
ms.date: 11/05/2020
ms.author: rolyon
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 78ed23f563fce9760768a99e5bbf58f68500d665
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103012026"
---
# <a name="remove-role-assignments-from-a-group-in-azure-active-directory"></a>Удаление назначений ролей из группы в Azure Active Directory

В этой статье описывается, как ИТ – администратор может удалять роли Azure AD, назначенные группам. В портал Azure теперь можно удалить как прямые, так и косвенные назначения ролей пользователю. Если пользователю назначена роль по членству в группе, удалите пользователя из группы, чтобы удалить назначение ролей.

## <a name="using-azure-admin-center"></a>Использование центра администрирования Azure

1. Войдите в [центр администрирования Azure AD](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview) с правами администратора привилегированных ролей или глобального администратора в Организации Azure AD.

1. Выберите **роли и**  >  **_имя роли_** администраторов.

1. Выберите группу, из которой необходимо удалить назначение ролей, и щелкните **удалить назначение**.

   ![Удаление назначения роли из выбранной группы.](./media/groups-remove-assignment/remove-assignment.png)

1. При появлении запроса на подтверждение действия выберите **Да**.

## <a name="using-powershell"></a>Использование PowerShell

### <a name="create-a-group-that-can-be-assigned-to-role"></a>Создание группы, которой можно назначить роль

```powershell
$group = New-AzureADMSGroup -DisplayName "Contoso_Helpdesk_Administrators" -Description "This group is assigned to Helpdesk Administrator built-in role in Azure AD." -MailEnabled $true -SecurityEnabled $true -MailNickName "contosohelpdeskadministrators" -IsAssignableToRole $true
```

### <a name="get-the-role-definition-you-want-to-assign-the-group-to"></a>Получение определения роли, для которой нужно назначить группу

```powershell
$roleDefinition = Get-AzureADMSRoleDefinition -Filter "displayName eq 'Helpdesk Administrator'"
```

### <a name="create-a-role-assignment"></a>Создание назначения роли

```powershell
$roleAssignment = New-AzureADMSRoleAssignment -ResourceScope '/' -RoleDefinitionId $roleDefinition.Id -PrincipalId $group.objectId
```

### <a name="remove-the-role-assignment"></a>Удаление назначения роли

```powershell
Remove-AzureAdMSRoleAssignment -Id $roleAssignment.Id 
```

## <a name="using-microsoft-graph-api"></a>Использование Microsoft Graph API

### <a name="create-a-group-that-can-be-assigned-an-azure-ad-role"></a>Создание группы, которой можно назначить роль Azure AD

```http
POST https://graph.microsoft.com/beta/groups
{
"description": "This group is assigned to Helpdesk Administrator built-in role of Azure AD",
"displayName": "Contoso_Helpdesk_Administrators",
"groupTypes": [
"Unified"
],
"mailEnabled": true,
"securityEnabled": true
"mailNickname": "contosohelpdeskadministrators",
"isAssignableToRole": true,
}
```

### <a name="get-the-role-definition"></a>Получение определения роли

```http
GET https://graph.microsoft.com/beta/roleManagement/directory/roleDefinitions?$filter=displayName+eq+'Helpdesk Administrator'
```

### <a name="create-the-role-assignment"></a>Создание назначения роли

```http
POST https://graph.microsoft.com/beta/roleManagement/directory/roleAssignments
{
"principalId":"{object-id-of-group}",
"roleDefinitionId":"{role-definition-id}",
"directoryScopeId":"/"
}
```

### <a name="delete-role-assignment"></a>Удаление назначения роли

```http
DELETE https://graph.microsoft.com/beta/roleManagement/directory/roleAssignments/{role-assignment-id}
```

## <a name="next-steps"></a>Дальнейшие действия

- [Использование облачных групп для управления назначениями ролей](groups-concept.md)
- [Устранение неполадок ролей, назначенных облачным группам](groups-faq-troubleshooting.md)
