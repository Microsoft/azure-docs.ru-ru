---
title: Управление разрешениями пользователя и администратора — Azure Active Directory | Документация Майкрософт
description: Узнайте, как просматривать разрешения для приложения в Azure AD и управлять ими. Например, отмените все разрешения, предоставленные приложению.
services: active-directory
author: msmimart
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 7/10/2020
ms.author: mimart
ms.reviewer: luleonpla
ms.collection: M365-identity-device-management
ms.openlocfilehash: cbd2bc7b9ccc9efe0c6fdf8de02665160667cb8f
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99256938"
---
# <a name="take-action-on-overprivileged-or-suspicious-applications-in-azure-active-directory"></a>Выполнение действий с недостаточными или подозрительными приложениями в Azure Active Directory

Узнайте, как просматривать разрешения приложения и управлять ими. В этой статье приведены различные действия, которые можно предпринять для защиты приложения в соответствии с ситуацией. Эти действия применяются ко всем приложениям, добавленным в клиент Azure Active Directory (Azure AD) через согласие пользователя или администратора.

Дополнительные сведения о предоставлении согласия для приложений см. в статье [Платформа предоставления согласия Azure Active Directory](../develop/consent-framework.md).

## <a name="prerequisites"></a>Предварительные условия

Чтобы выполнить следующие действия, необходимо войти в систему как глобальный администратор, администратор приложения или администратор облачных приложений.

Чтобы ограничить доступ к приложениям, необходимо запросить назначение пользователя, а затем назначить пользователей или группы приложению.  Подробнее см. статью [Методы назначения пользователей и групп](./assign-user-or-group-access-portal.md).

Чтобы получить контекстные скрипты PowerShell для выполнения действий, можно воспользоваться порталом Azure AD.
 
1. Войдите на [портал Azure](https://portal.azure.com) от имени глобального администратора, администратора приложения или администратора облачных приложений.
2. Выберите **Azure Active Directory**  >  **корпоративные приложения**.
3. Выберите приложение, доступ к которому необходимо ограничить.
4. Нажмите кнопку **Разрешения**. На панели команд выберите **проверить разрешения**.

![Снимок экрана: окно "Проверка разрешений".](./media/manage-application-permissions/review-permissions.png)


## <a name="control-access-to-an-application"></a>Управление доступом к приложению

Рекомендуется ограничить доступ к приложению, включив параметр **назначения пользователя** .

1. Войдите на [портал Azure](https://portal.azure.com) от имени глобального администратора, администратора приложения или администратора облачных приложений.
2. Выберите **Azure Active Directory**  >  **корпоративные приложения**.
3. Выберите приложение, доступ к которому необходимо ограничить.
4. Выберите **Свойства**, а затем задайте для параметра **требование к пользователю** значение **Да**.
5. Выберите **пользователь и группы**, а затем удалите ненужных пользователей, назначенных приложению.
6. Назначьте приложению пользователей или группы.

При необходимости можно удалить всех пользователей, назначенных приложению, с помощью PowerShell.

## <a name="revoke-all-permissions-for-an-application"></a>Отозвать все разрешения для приложения

С помощью скрипта PowerShell отменяются все разрешения, предоставленные этому приложению.

> [!NOTE]
> Отмена текущего предоставленного разрешения не приведет к запрету повторной отправки пользователей в приложение. Если вы хотите запретить пользователям отправку, прочитайте статью [Настройка согласия пользователей для приложений](configure-user-consent.md).

При необходимости можно отключить приложение, чтобы запретить пользователям доступ к приложению и запретить приложению доступ к данным.

1. Войдите на [портал Azure](https://portal.azure.com) от имени глобального администратора, администратора приложения или администратора облачных приложений.
2. Выберите **Azure Active Directory**  >  **корпоративные приложения**.
3. Выберите приложение, доступ к которому необходимо ограничить.
4. Выберите **Свойства**, а затем установите флажок **включено для пользователей для входа** **в систему.**

## <a name="investigate-a-suspicious-application"></a>Исследование подозрительного приложения

Рекомендуется ограничить доступ к приложению, включив параметр **назначения пользователя** . Затем просмотрите разрешения, предоставленные приложению пользователям и администраторам.

1. Войдите на [портал Azure](https://portal.azure.com) от имени глобального администратора, администратора приложения или администратора облачных приложений.
3. Выберите **Azure Active Directory**  >  **корпоративные приложения**.
5. Выберите приложение, доступ к которому необходимо ограничить.
6. Выберите **Свойства**, а затем задайте для параметра **требование к пользователю** значение **Да**.
7. Выберите **разрешения** и проверьте разрешения, предоставленные администратором и пользователем.

При необходимости с помощью PowerShell можно:

- Удалите всех назначенных пользователей, чтобы запретить им вход в приложение.
- Делает недействительными маркеры обновления для пользователей, имеющих доступ к приложению.
- Отозвать все разрешения для приложения.

Также можно отключить приложение, чтобы блокировать доступ пользователей и запретить приложению доступ к данным.


## <a name="disable-a-malicious-application"></a>Отключение вредоносного приложения 

Рекомендуется отключить приложение, чтобы блокировать доступ пользователей и запретить приложению доступ к данным. Если вместо этого вы удалите приложение, пользователи смогут повторно согласиться с этим приложением и предоставить доступ к данным.

1. Войдите на [портал Azure](https://portal.azure.com) от имени глобального администратора, администратора приложения или администратора облачных приложений.
2. Выберите **Azure Active Directory**  >  **корпоративные приложения**.
3. Выберите приложение, доступ к которому необходимо ограничить.
4. Выберите **Свойства**, а затем скопируйте идентификатор объекта.

### <a name="powershell-commands"></a>Команды PowerShell


Получение идентификатора объекта субъекта-службы.

1. Войдите на [портал Azure](https://portal.azure.com) от имени глобального администратора, администратора приложения или администратора облачных приложений.
2. Выберите **Azure Active Directory**  >  **корпоративные приложения**.
3. Выберите приложение, доступ к которому необходимо ограничить.
4. Выберите **Свойства**, а затем скопируйте идентификатор объекта.

```powershell
$sp = Get-AzureADServicePrincipal -Filter "displayName eq '$app_name'"
$sp.ObjectId
```
Удалите всех пользователей, назначенных приложению.
 ```powershell
Connect-AzureAD

# Get Service Principal using objectId
$sp = Get-AzureADServicePrincipal -ObjectId "<ServicePrincipal objectID>"

# Get Azure AD App role assignments using objectId of the Service Principal
$assignments = Get-AzureADServiceAppRoleAssignment -ObjectId $sp.ObjectId -All $true

# Remove all users and groups assigned to the application
$assignments | ForEach-Object {
    if ($_.PrincipalType -eq "User") {
        Remove-AzureADUserAppRoleAssignment -ObjectId $_.PrincipalId -AppRoleAssignmentId $_.ObjectId
    } elseif ($_.PrincipalType -eq "Group") {
        Remove-AzureADGroupAppRoleAssignment -ObjectId $_.PrincipalId -AppRoleAssignmentId $_.ObjectId
    }
}
 ```

Отозвать разрешения, предоставленные приложению.

```powershell
Connect-AzureAD

# Get Service Principal using objectId
$sp = Get-AzureADServicePrincipal -ObjectId "<ServicePrincipal objectID>"

# Get all delegated permissions for the service principal
$spOAuth2PermissionsGrants = Get-AzureADOAuth2PermissionGrant -All $true| Where-Object { $_.clientId -eq $sp.ObjectId }

# Remove all delegated permissions
$spOAuth2PermissionsGrants | ForEach-Object {
    Remove-AzureADOAuth2PermissionGrant -ObjectId $_.ObjectId
}

# Get all application permissions for the service principal
$spApplicationPermissions = Get-AzureADServiceAppRoleAssignedTo -ObjectId $sp.ObjectId -All $true | Where-Object { $_.PrincipalType -eq "ServicePrincipal" }

# Remove all delegated permissions
$spApplicationPermissions | ForEach-Object {
    Remove-AzureADServiceAppRoleAssignment -ObjectId $_.PrincipalId -AppRoleAssignmentId $_.objectId
}
```
Сделать токены обновления недействительными.
```powershell
Connect-AzureAD

# Get Service Principal using objectId
$sp = Get-AzureADServicePrincipal -ObjectId "<ServicePrincipal objectID>"

# Get Azure AD App role assignments using objectID of the Service Principal
$assignments = Get-AzureADServiceAppRoleAssignment -ObjectId $sp.ObjectId -All $true | Where-Object {$_.PrincipalType -eq "User"}

# Revoke refresh token for all users assigned to the application
$assignments | ForEach-Object {
    Revoke-AzureADUserAllRefreshToken -ObjectId $_.PrincipalId
}
```
## <a name="next-steps"></a>Дальнейшие действия
- [Управление согласия для приложений и оценка запроса согласия](manage-consent-requests.md)
- [Настройка согласия пользователя](configure-user-consent.md)
- [Настройка рабочего процесса согласия администратора](configure-admin-consent-workflow.md)
