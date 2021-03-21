---
title: Назначение ролей и их перечисление с помощью области административной единицы — Azure Active Directory | Документация Майкрософт
description: Используйте административные единицы, чтобы ограничить область назначений ролей в Azure Active Directory.
services: active-directory
documentationcenter: ''
author: rolyon
manager: daveba
ms.service: active-directory
ms.topic: how-to
ms.subservice: roles
ms.workload: identity
ms.date: 11/04/2020
ms.author: rolyon
ms.reviewer: anandy
ms.custom: oldportal;it-pro;
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1fc0c4bf9f71a8fe7e8cf49b83d32ac594dbe062
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103011407"
---
# <a name="assign-scoped-roles-to-an-administrative-unit"></a>Назначение ролей с заданной областью административной единице

В Azure Active Directory (Azure AD) для более детального административного управления можно назначить пользователей роли Azure AD с областью, ограниченной одним или несколькими административными единицами.

Чтобы подготовиться к использованию PowerShell и Microsoft Graph для управления административными единицами, см. статью Приступая к [работе](admin-units-manage.md#get-started).

## <a name="available-roles"></a>Доступные роли

Роль  |  Описание
----- |  -----------
Администратор проверки подлинности  |  Имеет доступ для просмотра, настройки и сброса сведений о методах проверки подлинности для любого пользователя без прав администратора в назначенной административной единице.
Администратор групп  |  Может управлять всеми аспектами групп и групп, таких как политики именования и истечения срока действия, только в назначенной административной единице.
Администратор службы технической поддержки  |  Может сбрасывать пароли для администраторов без прав администратора и службы технической поддержки только в назначенной административной единице.
Администратор лицензий  |  Может назначать, удалять и обновлять назначения лицензий только в административной единице.
администратора паролей.  |  Может сбрасывать пароли для администраторов без прав администратора и паролей только в назначенной административной единице.
администратора пользователей;  |  Может управлять всеми аспектами пользователей и групп, включая сброс паролей для ограниченных администраторов в пределах назначенной административной единицы.

## <a name="security-principals-that-can-be-assigned-to-a-scoped-role"></a>Субъекты безопасности, которые могут быть назначены роли с областью действия

Следующие субъекты безопасности могут быть назначены роли с областью административной единицы.

* Пользователи
* Назначаемые ролями облачные группы (Предварительная версия)
* Имя субъекта-службы (SPN)

## <a name="assign-a-scoped-role"></a>Назначение роли с областью действия

Роль с областью действия можно назначить с помощью портал Azure, PowerShell или Microsoft Graph.

### <a name="use-the-azure-portal"></a>Использование портала Azure

1. В портал Azure перейдите в **Azure AD**.

1. Выберите **административные единицы**, а затем выберите административную единицу, которой необходимо назначить область роли пользователя. 

1. В области слева выберите **роли и администраторы** , чтобы получить список всех доступных ролей.

   ![Снимок экрана: панель "роли и Администраторы" для выбора административной единицы, область роли которой необходимо назначить.](./media/admin-units-assign-roles/select-role-to-scope.png)

1. Выберите роль, которую необходимо назначить, а затем щелкните **добавить назначения**. 

1. На панели **Добавление назначений** выберите одного или нескольких пользователей, которым будет назначена роль.

   ![Выберите роль в области, а затем щелкните Добавить назначения.](./media/admin-units-assign-roles/select-add-assignment.png)

> [!Note]
> Сведения о назначении роли в административной единице с помощью Azure AD Privileged Identity Management (PIM) см. [в статье назначение ролей Azure AD в PIM](../privileged-identity-management/pim-how-to-add-role-to-user.md?tabs=new#assign-a-role-with-restricted-scope).

### <a name="use-powershell"></a>Использование PowerShell

```powershell
$adminUser = Get-AzureADUser -ObjectId "Use the user's UPN, who would be an admin on this unit"
$role = Get-AzureADDirectoryRole | Where-Object -Property DisplayName -EQ -Value "User Account Administrator"
$adminUnitObj = Get-AzureADMSAdministrativeUnit -Filter "displayname eq 'The display name of the unit'"
$roleMember = New-Object -TypeName Microsoft.Open.AzureAD.Model.RoleMemberInfo
$roleMember.ObjectId = $adminUser.ObjectId
Add-AzureADMSScopedRoleMembership -ObjectId $adminUnitObj.ObjectId -RoleObjectId $role.ObjectId -RoleMemberInfo $roleMember
```

Выделенный раздел можно изменить в соответствии с требованиями конкретной среды.

### <a name="use-microsoft-graph"></a>Использование Microsoft Graph

Запрос

```http
POST /directory/administrativeUnits/{admin-unit-id}/scopedRoleMembers
```
    
Текст

```http
{
  "roleId": "roleId-value",
  "roleMemberInfo": {
    "id": "id-value"
  }
}
```

## <a name="view-a-list-of-the-scoped-admins-in-an-administrative-unit"></a>Просмотр списка администраторов с заданной областью в административной единице

Список администраторов с заданной областью можно просмотреть с помощью портал Azure, PowerShell или Microsoft Graph.

### <a name="use-the-azure-portal"></a>Использование портала Azure

Все назначения ролей, созданные с помощью области административной единицы, можно просмотреть в [разделе административные единицы в Azure AD](https://ms.portal.azure.com/?microsoft_aad_iam_adminunitprivatepreview=true&microsoft_aad_iam_rbacv2=true#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/AdminUnit). 

1. В портал Azure перейдите в **Azure AD**.

1. В левой области выберите **административные единицы**, а затем выберите административную единицу для списка назначений ролей, которые требуется просмотреть. 

1. Выберите **роли и администраторы**, а затем откройте роль, чтобы просмотреть назначения в административной единице.

### <a name="use-powershell"></a>Использование PowerShell

```powershell
$adminUnitObj = Get-AzureADMSAdministrativeUnit -Filter "displayname eq 'The display name of the unit'"
Get-AzureADMSScopedRoleMembership -ObjectId $adminUnitObj.ObjectId | fl *
```

Выделенный раздел можно изменить в соответствии с требованиями конкретной среды.

### <a name="use-microsoft-graph"></a>Использование Microsoft Graph

Запрос

```http
GET /directory/administrativeUnits/{admin-unit-id}/scopedRoleMembers
```

Текст

```http
{}
```

## <a name="next-steps"></a>Дальнейшие действия

- [Использование облачных групп для управления назначениями ролей](groups-concept.md)
- [Устранение неполадок ролей, назначенных облачным группам](groups-faq-troubleshooting.md)
