---
title: Управление назначением пользователей для приложения в Azure Active Directory
description: Узнайте, как назначать и отменять назначение пользователей и групп для приложения, использующего Azure Active Directory для управления удостоверениями.
services: active-directory
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 02/21/2020
ms.author: kenwith
ms.reviewer: luleon
ms.openlocfilehash: 50dcde478b708cd53d8229d70a9ddf4b1ff271be
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99259740"
---
# <a name="manage-user-assignment-for-an-app-in-azure-active-directory"></a>Управление назначением пользователей для приложения в Azure Active Directory

В этой статье показано, как назначать пользователей и группы корпоративным приложениям в Azure Active Directory (Azure AD) в портал Azure или с помощью PowerShell. При назначении пользователя приложению приложение отображается в окне " [Мои приложения](https://myapps.microsoft.com/) " пользователя для быстрого доступа. Если приложение предоставляет роли, можно также назначить пользователю определенную роль.

Для более полного контроля можно настроить определенные типы корпоративных приложений так, чтобы они [требовали назначения пользователей](#configure-an-application-to-require-user-assignment). 

> [!IMPORTANT]
> После назначения приложению группы, доступ будут иметь только пользователи из этой группы. Назначение не распространяется на вложенные группы.

> [!NOTE]
> Для назначения ролей на основе групп необходимо установить Azure Active Directory Premium P1 или P2. Назначение на основе группы поддерживается только для групп безопасности. Членство во вложенных группах и группы Microsoft 365 в настоящее время не поддерживаются. Требования к лицензированию компонентов, рассматриваемых в этой статье, см. на странице [Цены на Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory). 

## <a name="configure-an-application-to-require-user-assignment"></a>Настройка приложения для требования назначения пользователей

При работе со следующими типами приложений можно сделать обязательным назначение приложения пользователям, прежде чем пользователи смогут осуществлять к нему доступ.

- Приложения, для которых настроен федеративный единый вход с аутентификацией на основе SAML.
- Приложения, которые используют прокси приложения и предварительную аутентификацию Azure AD.
- Приложения, созданные на платформе приложений Azure AD, использующих проверку подлинности OAuth 2.0 или OpenID Connect после того, как пользователь или администратор согласились с условиями использования этого приложения.

Если требуется назначение пользователей, вход будет доступен только пользователям, явным образом назначенным приложению (с помощью прямого назначения пользователей или на основе членства в группе). Они могут осуществлять доступ к приложению на странице "Мои приложения" или используя прямую ссылку. 

Если назначение *не требуется*, поскольку вы установили для этого параметра значение **нет** или если приложение использует другой режим единого входа, любой пользователь сможет получить доступ к приложению, если у него есть прямая ссылка на приложение или **URL-адрес доступа пользователя** на странице **свойств** приложения. 

Этот параметр не влияет на отображение приложения в окне "Мои приложения". Приложения отображаются на панелях доступа пользователей "Мои приложения" после назначения приложению пользователя или группы. Справочные сведения доступны в разделе [Управление доступом к приложениям](what-is-access-management.md).

Чтобы запросить назначение пользователей для приложения, выполните следующие действия.
1. Войдите на [портал Azure](https://portal.azure.com) с учетной записью администратора или в качестве владельца приложения.
2. Выберите **Azure Active Directory**. В области слева выберите **Корпоративные приложения**.
3. Выберите нужное приложение в списке. Если приложение не отображается, начните вводить его название в поле поиска. Либо используйте элементы управления фильтрами, чтобы выбрать тип приложения, статус или видимость, а затем нажмите **Применить**.
4. В области навигации слева щелкните **Свойства**.
5. Убедитесь, что переключатель **Требуется ли назначать пользователей?** находится в положении **Да**.
   > [!NOTE]
   > Если переключатель **Требуется ли назначать пользователей?** недоступен, можно с помощью PowerShell задать свойство appRoleAssignmentRequired для соответствующего субъекта-службы.
6. Нажмите кнопку **Сохранить** вверху экрана.

## <a name="assign-or-unassign-users-and-groups-for-an-app-using-the-azure-portal"></a>Назначение или отмена назначения пользователей и групп для приложения с помощью портал Azure
Сведения о том, как назначить или отменить назначение пользователя или группы с помощью портал Azure, см. в [руководстве по управлению приложениями](add-application-portal-assign-users.md).

## <a name="assign-or-unassign-users-and-groups-for-an-app-using-the-graph-api"></a>Назначение или отмена назначения пользователей и групп для приложения с помощью API Graph
Вы можете использовать API Graph, чтобы назначить или отменить назначение пользователей и групп для приложения. Дополнительные сведения см. в разделе [назначения ролей приложений](/graph/api/resources/approleassignment).

## <a name="assign-users-and-groups-to-an-app-using-powershell"></a>Назначение пользователей и групп для приложения с помощью PowerShell
1. Откройте окно Windows PowerShell с повышенными привилегиями.
   > [!NOTE]
   > Вам потребуется установить модуль Azure AD (с помощью команды `Install-Module -Name AzureAD`). Если будет предложено установить модуль NuGet или новый модуль PowerShell Azure для Active Directory версии 2, введите Y и нажмите клавишу ВВОД.
2. Выполните команду `Connect-AzureAD` и войдите в систему с помощью учетных данных глобального администратора.
3. Чтобы назначить пользователя и его роль в приложении, используйте следующий скрипт:

    ```powershell
    # Assign the values to the variables
    $username = "<Your user's UPN>"
    $app_name = "<Your App's display name>"
    $app_role_name = "<App role display name>"

    # Get the user to assign, and the service principal for the app to assign to
    $user = Get-AzureADUser -ObjectId "$username"
    $sp = Get-AzureADServicePrincipal -Filter "displayName eq '$app_name'"
    $appRole = $sp.AppRoles | Where-Object { $_.DisplayName -eq $app_role_name }

    # Assign the user to the app role
    New-AzureADUserAppRoleAssignment -ObjectId $user.ObjectId -PrincipalId $user.ObjectId -ResourceId $sp.ObjectId -Id $appRole.Id
    ```
Дополнительные сведения о том, как назначить пользователю роль в приложении, см. в документации по командлету [New-AzureADUserAppRoleAssignment](/powershell/module/azuread/new-azureaduserapproleassignment).

Чтобы назначить группу корпоративному приложению, необходимо заменить `Get-AzureADUser` на `Get-AzureADGroup`, а `New-AzureADUserAppRoleAssignment` — на `New-AzureADGroupAppRoleAssignment`.

Дополнительные сведения о том, как назначить группе роль в приложении, см. в документации по командлету [New-AzureADGroupAppRoleAssignment](/powershell/module/azuread/new-azureadgroupapproleassignment).

### <a name="example"></a>Пример

В этом примере пользователь Britta Simon назначается в приложении [Microsoft Workplace Analytics](https://products.office.com/business/workplace-analytics) с помощью PowerShell.

1. В PowerShell присвойте соответствующие значения переменным $username, $app_name и $app_role_name.

    ```powershell
    # Assign the values to the variables
    $username = "britta.simon@contoso.com"
    $app_name = "Workplace Analytics"
    ```
2. В этом примере неизвестно, как именно называется роль приложения, которую мы хотим назначить пользователю Britta Simon. Выполните следующие команды, чтобы получить пользователя ($user) и субъект-службу ($sp) с помощью отображаемых имен участника-пользователя и субъекта-службы.

    ```powershell
    # Get the user to assign, and the service principal for the app to assign to
    $user = Get-AzureADUser -ObjectId "$username"
    $sp = Get-AzureADServicePrincipal -Filter "displayName eq '$app_name'"
    ```
3. Выполните команду `$sp.AppRoles`, чтобы отобразить роли, доступные для приложения Workplace Analytics. В этом примере мы хотим назначить пользователю Britta Simon роль Analyst (Limited Access) (аналитик с ограниченным доступом).
   ![Показывает доступные пользователю роли с использованием роли в приложении Workplace Analytics](./media/assign-user-or-group-access-portal/workplace-analytics-role.png)
4. Присвойте имя роли переменной `$app_role_name`.

    ```powershell
    # Assign the values to the variables
    $app_role_name = "Analyst (Limited access)"
    $appRole = $sp.AppRoles | Where-Object { $_.DisplayName -eq $app_role_name }
    ```
5. Выполните следующую команду, чтобы назначить пользователю роль в приложении:

    ```powershell
    # Assign the user to the app role
    New-AzureADUserAppRoleAssignment -ObjectId $user.ObjectId -PrincipalId $user.ObjectId -ResourceId $sp.ObjectId -Id $appRole.Id
    ```

## <a name="unassign-users-and-groups-from-an-app-using-powershell"></a>Отмена назначения пользователей и групп из приложения с помощью PowerShell

1. Откройте окно Windows PowerShell с повышенными привилегиями.
   > [!NOTE]
   > Вам потребуется установить модуль Azure AD (с помощью команды `Install-Module -Name AzureAD`). Если будет предложено установить модуль NuGet или новый модуль PowerShell Azure для Active Directory версии 2, введите Y и нажмите клавишу ВВОД.
2. Выполните команду `Connect-AzureAD` и войдите в систему с помощью учетных данных глобального администратора.
3. Чтобы удалить пользователя и роль из приложения, используйте следующий скрипт:

    ```powershell
    # Store the proper parameters
    $user = get-azureaduser -ObjectId <objectId>
    $spo = Get-AzureADServicePrincipal -ObjectId <objectId>

    #Get the ID of role assignment 
    $assignments = Get-AzureADServiceAppRoleAssignment -ObjectId $spo.ObjectId | Where {$_.PrincipalDisplayName -eq $user.DisplayName}

    #if you run the following, it will show you what is assigned what
    $assignments | Select *

    #To remove the App role assignment run the following command.
    Remove-AzureADServiceAppRoleAssignment -ObjectId $spo.ObjectId -AppRoleAssignmentId $assignments[assignment #].ObjectId
    ```


## <a name="related-articles"></a>Связанные статьи

- [Дополнительные сведения о доступе к приложениям для конечных пользователей](end-user-experiences.md)
- [Планирование развертывания "Мои приложения" в Azure AD](my-apps-deployment-plan.md)
- [Управление доступом к приложениям](what-is-access-management.md)
 
## <a name="next-steps"></a>Дальнейшие действия

- [Просмотр всех моих групп](../fundamentals/active-directory-groups-view-azure-portal.md)
- [Удаление назначения пользователя или группы из корпоративного приложения]()
- [Отключение входа пользователя в корпоративное приложение](disable-user-sign-in-portal.md)
- [Изменение имени или логотипа корпоративного приложения](./add-application-portal-configure.md)
