---
title: Перечисление назначений ролей Azure AD
description: Теперь вы можете просматривать членов роли администратора Azure Active Directory и управлять ими в центре администрирования Azure Active Directory.
services: active-directory
author: rolyon
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: how-to
ms.date: 11/05/2020
ms.author: rolyon
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: de546ef091b1a8e996f286b0c9af45e93488b5b4
ms.sourcegitcommit: 3ea12ce4f6c142c5a1a2f04d6e329e3456d2bda5
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/15/2021
ms.locfileid: "103467663"
---
# <a name="list-azure-ad-role-assignments"></a>Перечисление назначений ролей Azure AD

В этой статье описывается, как вывести список ролей, назначенных в Azure Active Directory (Azure AD). В Azure Active Directory (Azure AD) роли можно назначать в масштабах всей организации или с помощью области одного приложения.

- Назначения ролей в масштабе всей организации добавляются в и могут отображаться в списке назначений ролей приложений.
- Назначения ролей в области одного приложения не добавляются в и не отображаются в списке назначений в области всей Организации.

## <a name="list-role-assignments-in-the-azure-portal"></a>Вывод списка назначений ролей в портал Azure

В этой процедуре описывается, как вывести список назначений ролей в масштабе всей Организации.

1. Войдите в [центр администрирования Azure AD](https://aad.portal.azure.com) с правами администратора привилегированных ролей или глобального администратора в Организации Azure AD.
1. Выберите **Azure Active Directory**, выберите **роли и администраторы**, а затем выберите роль, чтобы открыть ее и просмотреть ее свойства.
1. Выберите **назначения** , чтобы вывести список назначений ролей.

    ![Вывод списка назначений ролей и разрешений при открытии роли из списка](./media/view-assignments/role-assignments.png)

## <a name="list-my-role-assignments"></a>Вывод списка моих назначений ролей

Также можно легко получить список собственных разрешений. Выберите **Ваша роль** на странице **Роли и администраторы**, чтобы увидеть роли, назначенные вам сейчас.

## <a name="download-role-assignments"></a>Скачать назначения ролей

Чтобы скачать все назначения для определенной роли, на странице **роли и администраторы** выберите роль, а затем щелкните **загрузить назначения ролей**. CSV-файл, в котором перечисляются назначения для всех областей этой роли.

![скачивание всех назначений для роли](./media/view-assignments/download-role-assignments.png)

## <a name="list-role-assignments-using-azure-ad-powershell"></a>Вывод списка назначений ролей с помощью Azure AD PowerShell

В этом разделе описывается Просмотр назначений роли в масштабе всей Организации. В этой статье используется модуль [Azure Active Directory PowerShell версии 2](/powershell/module/azuread/#directory_roles) . Для просмотра назначений области с одним приложением с помощью PowerShell можно использовать командлеты в этой службе для [назначения пользовательских ролей](custom-assign-powershell.md).

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

### <a name="list-role-assignments"></a>Список назначений ролей

Пример перечисления назначений ролей.

``` PowerShell
# Fetch list of all directory roles with object ID
Get-AzureADDirectoryRole

# Fetch a specific directory role by ID
$role = Get-AzureADDirectoryRole -ObjectId "5b3fe201-fa8b-4144-b6f1-875829ff7543"

# Fetch role membership for a role
Get-AzureADDirectoryRoleMember -ObjectId $role.ObjectId | Get-AzureADUser
```

## <a name="list-role-assignments-using-microsoft-graph-api"></a>Вывод списка назначений ролей с помощью Microsoft Graph API

В этом разделе описывается, как вывести список назначений ролей в масштабе всей Организации.  Чтобы составить список назначений ролей области одного приложения с помощью API Graph, можно использовать операции в [назначении пользовательских ролей с API Graph](custom-assign-graph.md).

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

## <a name="list-role-assignments-with-single-application-scope"></a>Вывод списка назначений ролей в области одного приложения

В этом разделе описывается, как вывести список назначений ролей в области одного приложения. Эта функция сейчас доступна в виде общедоступной предварительной версии.

1. Войдите в [центр администрирования Azure AD](https://aad.portal.azure.com) с правами администратора привилегированных ролей или глобального администратора в Организации Azure AD.
1. Выберите **Регистрация приложений**, а затем выберите регистрацию приложения, чтобы просмотреть его свойства. Чтобы просмотреть полный список регистраций приложений в вашей организации Azure AD, выберите **Все приложения**.

    ![Создание или изменение регистраций приложений на странице Регистрация приложений](./media/view-assignments/app-reg-all-apps.png)

1. В окне Регистрация приложения выберите **роли и администраторы**, а затем выберите роль, чтобы просмотреть ее свойства.

    ![Вывод списка назначений ролей регистрации приложения на странице Регистрация приложений](./media/view-assignments/app-reg-assignments.png)

1. Выберите **назначения** , чтобы вывести список назначений ролей. При открытии страницы назначения в процессе регистрации приложения отображаются назначения ролей, которые относятся к этому ресурсу Azure AD.

    ![Вывод списка назначений ролей регистрации приложения из свойств регистрации приложения](./media/view-assignments/app-reg-assignments-2.png)

## <a name="next-steps"></a>Дальнейшие действия

* Вы можете оставить комментарий на [форуме об административных ролях Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=166032).
* Дополнительные сведения о ролях и назначении роли администратора см. в разделе [Назначение ролей администратора](permissions-reference.md).
* Сведения о разрешениях пользователей по умолчанию см. в разделе [сравнение разрешений гостевых пользователей и пользователей-участников](../fundamentals/users-default-permissions.md).
