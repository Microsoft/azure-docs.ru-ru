---
title: Назначение роли группе с помощью управления привилегированными пользователями в Azure AD | Документация Майкрософт
description: Узнайте, как можно назначить роль Azure Active Directory (Azure AD) группе с помощью Azure AD Privileged Identity Management (PIM).
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
ms.openlocfilehash: 02cd3f54823b80ae201316fee29c02616b9d8502
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103012043"
---
# <a name="assign-a-role-to-a-group-using-privileged-identity-management"></a>Назначение роли группе с помощью управления привилегированными пользователями

В этой статье описывается, как можно назначить роль Azure Active Directory (Azure AD) группе с помощью Azure AD Privileged Identity Management (PIM).

> [!NOTE]
> Чтобы назначить группу роли Azure AD с помощью PIM, необходимо использовать обновленную версию Azure AD Privileged Identity Management. Вы можете использовать более старую версию PIM, если ваша организация Azure AD использует API Privileged Identity Management. В этом случае обратитесь к команде pim_preview@microsoft.com, чтобы переместить организацию и обновить API. Дополнительные сведения о [ролях и функциях Azure AD в PIM](../privileged-identity-management/azure-ad-roles-features.md).

## <a name="using-azure-ad-admin-center"></a>Использование Центра администрирования Azure AD

1. Войдите в [Azure AD privileged Identity Management](https://ms.portal.azure.com/?Microsoft_AAD_IAM_GroupRoles=true&Microsoft_AAD_IAM_userRolesV2=true&Microsoft_AAD_IAM_enablePimIntegration=true#blade/Microsoft_Azure_PIMCommon/CommonMenuBlade/quickStart) как администратор привилегированных ролей или глобальный администратор в организации.

1. Выберите **Privileged Identity Management** > **Роли Azure AD** > **Роли** > **Добавить назначения**

1. Выберите роль, а затем группу. Отображаются только группы с возможностью назначения ролей, а не все группы.

    ![Снимок экрана, на котором показана страница "Добавление назначений" с выделенными разделами "Выбор роли" и "Выбор элементов".](./media/groups-pim-eligible/select-member.png)

1. Выберите нужный параметр членства. Для ролей, которые необходимо активировать, выберите **соответствует**. По умолчанию пользователь будет соответствовать всегда, но можно также задать время начала и окончания. По завершении нажмите кнопку "Сохранить и добавить", чтобы завершить назначение роли.

    ![Выбор участника, которому назначается роль](./media/groups-pim-eligible/set-assignment-settings.png)

## <a name="using-powershell"></a>Использование PowerShell

### <a name="download-the-azure-ad-preview-powershell-module"></a>Загрузка предварительной версии модуля Azure AD PowerShell

Чтобы установить модуль Azure AD PowerShell, используйте следующие командлеты:

```powershell
Install-Module -Name AzureADPreview
Import-Module -Name AzureADPreview
```

Чтобы проверить, готов ли модуль к использованию, примените следующий командлет:

```powershell
Get-Module -Name AzureADPreview
```

### <a name="assign-a-group-as-an-eligible-member-of-a-role"></a>Назначение группы в качестве соответствующего члена роли

```powershell
$schedule = New-Object Microsoft.Open.MSGraph.Model.AzureADMSPrivilegedSchedule
$schedule.Type = "Once"
$schedule.StartDateTime = "2019-04-26T20:49:11.770Z"
$schedule.endDateTime = "2019-07-25T20:49:11.770Z"
Open-AzureADMSPrivilegedRoleAssignmentRequest -ProviderId aadRoles -Schedule $schedule -ResourceId "[YOUR TENANT ID]" -RoleDefinitionId "9f8c1837-f885-4dfd-9a75-990f9222b21d" -SubjectId "[YOUR GROUP ID]" -AssignmentState "Eligible" -Type "AdminAdd"
```

## <a name="using-microsoft-graph-api"></a>Использование API Microsoft Graph

```http
POST
https://graph.microsoft.com/beta/privilegedAccess/aadroles/roleAssignmentRequests
{
 "roleDefinitionId": {roleDefinitionId},
 "resourceId": {tenantId},
 "subjectId": {GroupId},
 "assignmentState": "Eligible",
 "type": "AdminAdd",
 "reason": "reason string",
 "schedule": {
   "startDateTime": {DateTime},
   "endDateTime": {DateTime},
   "type": "Once"
 }
}
```

## <a name="next-steps"></a>Дальнейшие действия

- [Использование облачных групп для управления назначениями ролей](groups-concept.md)
- [Устранение неполадок ролей, назначенных облачным группам](groups-faq-troubleshooting.md)
- [Настройка параметров роли администратора Azure AD в Privileged Identity Management](../privileged-identity-management/pim-how-to-change-default-settings.md)
- [Роли ресурсов Azure в Privileged Identity Management](../privileged-identity-management/pim-resource-roles-assign-roles.md)
