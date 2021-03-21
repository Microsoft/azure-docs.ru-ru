---
title: Просмотр ролей, назначенных группе, в Azure Active Directory | Документация Майкрософт
description: Узнайте, как можно просматривать роли, назначенные группе, с помощью центра администрирования Azure AD. Просмотр групп и назначенных ролей являются разрешениями пользователя по умолчанию.
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
ms.openlocfilehash: 7dac23897f8a17c9adb4ae78736a6a8afa85a18b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103012009"
---
# <a name="view-roles-assigned-to-a-group-in-azure-active-directory"></a>Просмотр ролей, назначенных группе, в Azure Active Directory

В этом разделе описывается, как можно просмотреть роли, назначенные группе, с помощью центра администрирования Azure AD. Просмотр групп и назначенных ролей являются разрешениями пользователя по умолчанию.

1. Войдите в [центр администрирования Azure AD](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview) , используя любые учетные данные без прав администратора или администратора.

1. Выберите интересующую Вас группу.

1. Выберите **назначенные роли**. Теперь вы можете увидеть все роли Azure AD, назначенные этой группе.

   ![Просмотреть все роли, назначенные выбранной группе](./media/groups-view-assignments/view-assignments.png)

## <a name="using-powershell"></a>Использование PowerShell

### <a name="get-object-id-of-the-group"></a>Получение идентификатора объекта группы

```powershell
Get-AzureADMSGroup -SearchString "Contoso_Helpdesk_Administrators"
```

### <a name="view-role-assignment-to-a-group"></a>Просмотр назначения роли группе

```powershell
Get-AzureADMSRoleAssignment -Filter "principalId eq '<object id of group>" 
```

## <a name="using-microsoft-graph-api"></a>Использование Microsoft Graph API

### <a name="get-object-id-of-the-group"></a>Получение идентификатора объекта группы

```http
GET https://graph.microsoft.com/beta/groups?$filter=displayName+eq+'Contoso_Helpdesk_Administrator'
```

### <a name="get-role-assignments-to-a-group"></a>Получение назначений ролей группе

```http
GET https://graph.microsoft.com/beta/roleManagement/directory/roleAssignments?$filter=principalId eq
```

## <a name="next-steps"></a>Дальнейшие действия

- [Использование облачных групп для управления назначениями ролей](groups-concept.md)
- [Устранение неполадок ролей, назначенных облачным группам](groups-faq-troubleshooting.md)
