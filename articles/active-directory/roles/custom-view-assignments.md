---
title: Просмотр настраиваемых назначений ролей на портале Azure AD | Документация Майкрософт
description: Теперь вы можете просматривать членов роли администратора Azure AD и управлять ими в центре администрирования Azure AD.
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
ms.openlocfilehash: 23f95ede778752a8367f77717247fff82ead7131
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103012077"
---
# <a name="view-custom-role-assignments-in-azure-active-directory"></a>Просмотр пользовательских назначений ролей в Azure Active Directory

В этой статье описывается, как просматривать настраиваемые роли, назначенные в Azure Active Directory (Azure AD). В Azure Active Directory (Azure AD) роли можно назначать в масштабах всей организации или с помощью области одного приложения.

- Назначения ролей в масштабе всей организации добавляются в и могут отображаться в списке назначений ролей приложений.
- Назначения ролей в области одного приложения не добавляются в и не отображаются в списке назначений в области всей Организации.

## <a name="view-role-assignments-in-the-azure-portal"></a>Просмотр назначений ролей в портал Azure

Эта процедура описывает Просмотр назначений роли в масштабе всей Организации.

1. Войдите в [центр администрирования Azure AD](https://aad.portal.azure.com) с правами администратора привилегированных ролей или глобального администратора в Организации Azure AD.
1. Выберите **Azure Active Directory**, выберите **роли и администраторы**, а затем выберите роль, чтобы открыть ее и просмотреть ее свойства.
1. Выберите **назначения** , чтобы просмотреть назначения для роли.

    ![Просмотр назначений ролей и разрешений при открытии роли из списка](./media/custom-view-assignments/role-assignments.png)

## <a name="view-role-assignments-using-azure-ad-powershell"></a>Просмотр назначений ролей с помощью Azure AD PowerShell

В этом разделе описывается Просмотр назначений роли в масштабе всей Организации. В этой статье используется модуль [Azure Active Directory PowerShell версии 2](/powershell/module/azuread/#directory_roles) . Для просмотра назначений области с одним приложением с помощью PowerShell можно использовать командлеты в этой службе для [назначения пользовательских ролей](./custom-assign-powershell.md).

### <a name="prepare-powershell"></a>Подготовка PowerShell

Сначала необходимо [скачать модуль PowerShell для предварительной версии Azure AD](https://www.powershellgallery.com/packages/AzureAD/).

Чтобы установить модуль Azure AD PowerShell, выполните следующие команды:

``` PowerShell
Install-Module -Name AzureADPreview
Import-Module -Name AzureADPreview
```

Чтобы проверить, готов ли модуль к использованию, выполните следующую команду:

``` PowerShell
Get-Module -Name AzureADPreview
  ModuleType Version      Name                         ExportedCommands
  ---------- ---------    ----                         ----------------
  Binary     2.0.0.115    AzureADPreview               {Add-AzureADAdministrati...}
```

### <a name="view-the-assignments-of-a-role"></a>Просмотр назначений роли

Пример просмотра назначений роли.

``` PowerShell
# Fetch list of all directory roles with object ID
Get-AzureADDirectoryRole

# Fetch a specific directory role by ID
$role = Get-AzureADDirectoryRole -ObjectId "5b3fe201-fa8b-4144-b6f1-875829ff7543"

# Fetch role membership for a role
Get-AzureADDirectoryRoleMember -ObjectId $role.ObjectId | Get-AzureADUser
```

## <a name="view-role-assignments-using-microsoft-graph-api"></a>Просмотр назначений ролей с помощью API Microsoft Graph

В этом разделе описывается Просмотр назначений роли в масштабе всей Организации.  Для просмотра назначений области с одним приложением с помощью API Graph можно использовать операции в [назначении пользовательских ролей с API Graph](./custom-assign-graph.md).

HTTP-запрос для получения назначения ролей для указанного определения роли.

GET

``` HTTP
https://graph.microsoft.com/beta/roleManagement/directory/roleAssignments&$filter=roleDefinitionId eq ‘<object-id-or-template-id-of-role-definition>’
```

Ответ

``` HTTP
HTTP/1.1 200 OK
{
    "id":"CtRxNqwabEKgwaOCHr2CGJIiSDKQoTVJrLE9etXyrY0-1"
    "principalId":"ab2e1023-bddc-4038-9ac1-ad4843e7e539",
    "roleDefinitionId":"3671d40a-1aac-426c-a0c1-a3821ebd8218",
    "resourceScopes":["/"]
}
```

## <a name="view-assignments-of-single-application-scope"></a>Просмотр назначений области одного приложения

В этом разделе описывается Просмотр назначений роли с областью действия одного приложения. Эта функция сейчас доступна в виде общедоступной предварительной версии.

1. Войдите в [центр администрирования Azure AD](https://aad.portal.azure.com) с правами администратора привилегированных ролей или глобального администратора в Организации Azure AD.
1. Выберите **Регистрация приложений**, а затем выберите регистрацию приложения, чтобы просмотреть его свойства. Чтобы просмотреть полный список регистраций приложений в вашей организации Azure AD, выберите **Все приложения**.

    ![Создание или изменение регистраций приложений на странице Регистрация приложений](./media/custom-view-assignments/appreg-all-apps.png)

1. В окне Регистрация приложения выберите **роли и администраторы**, а затем выберите роль, чтобы просмотреть ее свойства.

    ![Просмотр назначений ролей регистрации приложений на странице Регистрация приложений](./media/custom-view-assignments/appreg-assignments.png)

1. Выберите **назначения** , чтобы просмотреть назначения для роли. При открытии представления назначения из регистрации приложения отображаются назначения, областью которых является этот ресурс Azure AD.

    ![Просмотр назначений ролей регистрации приложений из свойств регистрации приложения](./media/custom-view-assignments/appreg-assignments-2.png)

## <a name="next-steps"></a>Дальнейшие действия

* Вы можете оставить комментарий на [форуме об административных ролях Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=166032).
* Дополнительные сведения о ролях и назначении роли администратора см. в разделе [Назначение ролей администратора](permissions-reference.md).
* Сведения о разрешениях пользователей по умолчанию см. в разделе [сравнение разрешений гостевых пользователей и пользователей-участников](../fundamentals/users-default-permissions.md).
