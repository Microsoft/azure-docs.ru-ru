---
title: Назначение роли "Читатели каталогов" группе Azure AD и управление назначениями ролей
description: В этой статье описывается включение роли "Читатели каталогов" с помощью групп Azure AD для управления назначениями ролей Azure AD с использованием Базы данных SQL Azure, Управляемого экземпляра SQL Azure и Azure Synapse Analytics.
ms.service: sql-db-mi
ms.subservice: security
ms.custom: azure-synapse
ms.topic: tutorial
author: GithubMirek
ms.author: mireks
ms.reviewer: vanto
ms.date: 08/14/2020
ms.openlocfilehash: bc809cf02b827b7498890cb7d929c44bd360ab53
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99094747"
---
# <a name="tutorial-assign-directory-readers-role-to-an-azure-ad-group-and-manage-role-assignments"></a>Руководство по назначению роли "Читатели каталогов" группе Azure AD и управлению назначениями ролей

[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

> [!NOTE]
> Описанное в этой статье назначение роли **Читатели каталогов** группе находится в **общедоступной предварительной версии**. 

В этой статье описано, как создать группу в Azure Active Directory (Azure AD) и назначить ей роль [**Читатели каталогов**](../../active-directory/roles/permissions-reference.md#directory-readers). Разрешения "Читатели каталогов" позволяют владельцам групп добавлять в них дополнительных членов, таких как [управляемые удостоверения](../../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types) [Базы данных SQL Azure](sql-database-paas-overview.md), [Управляемого экземпляра SQL Azure](../managed-instance/sql-managed-instance-paas-overview.md) и [Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md). В этом случае для назначения роли "Читатели каталогов" непосредственно для каждого удостоверения логического сервера Azure SQL в клиенте не требуются права [глобального администратора](../../active-directory/roles/permissions-reference.md#global-administrator) или [администратора привилегированных ролей](../../active-directory/roles/permissions-reference.md#privileged-role-administrator).

В этом учебнике используется функция, описанная в статье [Использование облачных групп для управления назначением ролей в Azure Active Directory (предварительная версия)](../../active-directory/roles/groups-concept.md). 

Дополнительные сведения о преимуществах назначения роли "Читатели каталогов" группе Azure AD для Azure SQL см. в статье о [роли"Читатели каталогов" в Azure Active Directory для Azure SQL](authentication-aad-directory-readers-role.md).

## <a name="prerequisites"></a>Обязательные условия

- Экземпляр Azure AD. Дополнительные сведения см. в статье о [настройке проверки подлинности Azure AD и управлении ею с помощью Azure SQL](authentication-aad-configure.md).
- База данных SQL, Управляемый экземпляр SQL или Azure Synapse.

## <a name="directory-readers-role-assignment-using-the-azure-portal"></a>Назначение роли "Читатели каталогов" с помощью портала Azure

### <a name="create-a-new-group-and-assign-owners-and-role"></a>Создание новой группы и назначение владельцев и роли

1. Для этой первоначальной настройки пользователю требуются разрешения [Глобальный администратор](../../active-directory/roles/permissions-reference.md#global-administrator) или [Администратор привилегированных ролей](../../active-directory/roles/permissions-reference.md#privileged-role-administrator).
1. Привилегированный пользователь должен войти на [портал Azure](https://portal.azure.com).
1. Перейдите к ресурсу **Azure Active Directory**. В разделе **Управляемые** перейдите в раздел **Группы**. Чтобы создать группу, выберите **Создать группу**.
1. Выберите **Безопасность** в качестве типа группы и заполните остальные поля. Убедитесь, что для параметра **Azure AD roles can be assigned to the group (Preview)** (Роли Azure AD можно назначить группе (предварительная версия)) установлено значение **Да**. Затем назначьте группе роль **Читатели каталогов** Azure AD.
1. Назначьте пользователей Azure AD в качестве владельцев созданной группы. Владельцем группы может быть обычный пользователь AD, не имеющий назначенной роли администратора Azure AD. Владельцем должен быть пользователь, управляющий Базой данных SQL, Управляемым экземпляром SQL или Azure Synapse.

   :::image type="content" source="media/authentication-aad-directory-readers-role/new-group.png" alt-text="aad-new-group":::

1. Нажмите кнопку **Создать**

### <a name="checking-the-group-that-was-created"></a>Проверка созданной группы

> [!NOTE]
> Убедитесь, что для параметра **Тип группы** указано **Безопасность**. Группы *Microsoft 365* не поддерживаются для Azure SQL.

Чтобы проверить созданную группу и управлять ею, вернитесь в область **Группы** на портале Azure и найдите имя группы. После выбора группы можно добавить дополнительных владельцев и членов, выбрав меню **Владельцы** и **Члены** в параметре **Управление**. Кроме того, можно проверить **назначенные роли** для группы.

:::image type="content" source="media/authentication-aad-directory-readers-role/azure-ad-group-created.png" alt-text="Снимок экрана: область &quot;Группа&quot; с выделенными ссылками, по которым открываются меню параметров для членов, владельцев и назначенных ролей (предварительная версия).":::

### <a name="add-azure-sql-managed-identity-to-the-group"></a>Добавление управляемого удостоверения SQL Azure в группу

> [!NOTE]
> В этом примере мы используем Управляемый экземпляр SQL, но аналогичные действия можно применить и к Базе данных SQL или Azure Synapse, получив те же результаты.

Для последующих шагов пользователю больше не нужны права глобального администратора или администратора привилегированных ролей.

1. Войдите на портал Azure в качестве пользователя, управляющего Управляемым экземпляром SQL и владельца группы, созданной ранее.

1. Найдите на портале Azure имя ресурса **управляемого экземпляра SQL**.

   :::image type="content" source="media/authentication-aad-directory-readers-role/azure-ad-managed-instance.png" alt-text="Снимок экрана: страница &quot;Управляемые экземпляры SQL&quot; с выделенными именем экземпляра SQL ssomitest и именем подсети ManagedInstance.":::

   Во время создания Управляемого экземпляра SQL для вашего экземпляра было создано удостоверение Azure. Созданное удостоверение имеет то же имя, что и префикс имени Управляемого экземпляра SQL. Чтобы найти субъект-службу для удостоверения Управляемого экземпляра SQL, созданного в качестве приложения Azure AD, выполните следующие действия:

    - Перейдите к ресурсу **Azure Active Directory**. В разделе параметра **Управление** выберите **Корпоративные приложения**. **Идентификатор объекта** — это идентификатор экземпляра.
    
    :::image type="content" source="media/authentication-aad-directory-readers-role/azure-ad-managed-instance-service-principal.png" alt-text="Снимок экрана: страница &quot;Корпоративные приложения&quot; для ресурса Azure Active Directory с выделенным идентификатором объекта для управляемого экземпляра SQL.":::

1. Перейдите к ресурсу **Azure Active Directory**. В разделе **Управляемые** перейдите в раздел **Группы**. Выберите созданную группу. В параметре вашей группы **Управляемые** выберите **Члены**. Выберите **Добавление членов** и добавьте субъект-службу Управляемого экземпляра SQL в качестве члена группы, выполнив поиск по имени, указанному выше.

   :::image type="content" source="media/authentication-aad-directory-readers-role/azure-ad-add-managed-instance-service-principal.png" alt-text="Снимок экрана: страница &quot;Члены&quot; для ресурса Azure Active Directory с выделенными вариантами для добавления управляемого экземпляра SQL в качестве нового члена.":::

> [!NOTE]
> Распространение разрешений субъекта-службы через систему Azure и предоставление доступа к API Graph Azure AD может занять несколько минут. Возможно, вам придется подождать несколько минут, прежде чем приступить к подготовке администратора Azure AD для Управляемого экземпляра SQL.

### <a name="remarks"></a>Комментарии

Для Базы данных SQL и Azure Synapse удостоверение сервера можно создать во время создания логического сервера Azure SQL или после его создания. Дополнительные сведения о создании или настройке удостоверения сервера в Базе данных SQL или Azure Synapse см. в разделе о [включении субъектов-служб для создания пользователей Azure AD](authentication-aad-service-principal.md#enable-service-principals-to-create-azure-ad-users).

Чтобы [настроить администратора Azure AD для Управляемого экземпляра SQL](authentication-aad-configure.md#provision-azure-ad-admin-sql-managed-instance), удостоверению управляемого экземпляра необходимо назначить роль **Читатели каталогов**.

Для Базы данных SQL или Azure Synapse при настройке администратора Azure AD для логического сервера назначение роли **Читатели каталогов** удостоверению сервера не требуется. Однако для создания объекта Azure AD в Базе данных SQL или Azure Synapse от имени приложения Azure AD требуется роль **Читатели каталогов**. Если роль удостоверению логического сервера SQL не назначена, создание пользователей Azure AD в Azure SQL завершится ошибкой. Дополнительные сведения см. в статье [Субъект-служба Azure Active Directory с SQL Azure](authentication-aad-service-principal.md).

## <a name="directory-readers-role-assignment-using-powershell"></a>Назначение роли "Читатели каталогов" с помощью PowerShell

> [!IMPORTANT]
> [Глобальному администратору](../../active-directory/roles/permissions-reference.md#global-administrator) или [администратору привилегированных ролей](../../active-directory/roles/permissions-reference.md#privileged-role-administrator) потребуется выполнить начальные действия. Помимо PowerShell, Azure AD предлагает API Microsoft Graph для [создания группы с назначением роли в Azure AD](../../active-directory/roles/groups-create-eligible.md#using-microsoft-graph-api).

1. Скачайте модуль Azure AD PowerShell (предварительная версия) с помощью указанной ниже команды. Возможно, потребуется запустить PowerShell от имени администратора.

    ```powershell
    Install-Module azureadpreview
    Import-Module azureadpreview
    #To verify that the module is ready to use, use the following command:
    Get-Module azureadpreview
    ```

1. Подключитесь к клиенту Azure AD.

    ```powershell
    Connect-AzureAD
    ```

1. Создайте группу безопасности, чтобы назначить роль **Читатели каталогов**.

    - `DirectoryReaderGroup`, `Directory Reader Group` и `DirRead` можно изменить в соответствии с вашими предпочтениями.

    ```powershell
    $group = New-AzureADMSGroup -DisplayName "DirectoryReaderGroup" -Description "Directory Reader Group" -MailEnabled $False -SecurityEnabled $true -MailNickName "DirRead" -IsAssignableToRole $true
    $group
    ```

1. Назначьте группе роль **Читатели каталогов**.

    ```powershell
    # Displays the Directory Readers role information
    $roleDefinition = Get-AzureADMSRoleDefinition -Filter "displayName eq 'Directory Readers'" 
    $roleDefinition
    ```

    ```powershell
    # Assigns the Directory Readers role to the group
    $roleAssignment = New-AzureADMSRoleAssignment -ResourceScope '/' -RoleDefinitionId $roleDefinition.Id -PrincipalId $group.Id
    $roleAssignment
    ```

1. Назначьте группе владельцев.

    - Замените `<username>` именем пользователя, которого вы хотите назначить владельцем этой группы. Чтобы добавить несколько владельцев, повторите эти действия.

    ```powershell
    $RefObjectID = Get-AzureADUser -ObjectId "<username>"
    $RefObjectID
    ```

    ```powershell
    $GrOwner = Add-AzureADGroupOwner -ObjectId $group.ID -RefObjectId $RefObjectID.ObjectID
    ```

    Проверьте владельцев группы с помощью следующей команды:

    ```powershell
    Get-AzureADGroupOwner -ObjectId $group.ID
    ```

    Вы также можете проверить владельцев группы на [портале Azure](https://portal.azure.com). Выполните действия, описанные в разделе о [проверке созданной группы](#checking-the-group-that-was-created).

### <a name="assigning-the-service-principal-as-a-member-of-the-group"></a>Назначение субъекта-службы в качестве члена группы

Для последующих шагов пользователю больше не нужны права глобального администратора или администратора привилегированных ролей.

1. Используя владельца группы, который также управляет ресурсом Azure SQL, выполните следующую команду, чтобы подключиться к Azure AD.

    ```powershell
    Connect-AzureAD
    ```

1. Назначьте субъект-службу в качестве члена созданной группы.

    - Замените `<ServerName>` именем логического сервера Azure SQL или именем Управляемого экземпляра. Дополнительные сведения см. в разделе о [добавлении удостоверения службы Azure SQL в группу](#add-azure-sql-managed-identity-to-the-group).

    ```powershell
    # Returns the service principal of your Azure SQL resource
    $miIdentity = Get-AzureADServicePrincipal -SearchString "<ServerName>"
    $miIdentity
    ```

    ```powershell
    # Adds the service principal to the group as a member
    Add-AzureADGroupMember -ObjectId $group.ID -RefObjectId $miIdentity.ObjectId 
    ```

    Следующая команда возвращает идентификатор объекта субъекта-службы, указывая, что он добавлен в группу:

    ```powershell
    Add-AzureADGroupMember -ObjectId $group.ID -RefObjectId $miIdentity.ObjectId
    ```

## <a name="next-steps"></a>Дальнейшие действия

- [Роль "Читатели каталогов" в Azure Active Directory для Azure SQL](authentication-aad-directory-readers-role.md)
- [Руководство по созданию пользователей Azure AD с помощью приложений Azure AD](authentication-aad-service-principal-tutorial.md)
- [Настройка и администрирование проверки подлинности Azure Active Directory с помощью Azure SQL](authentication-aad-configure.md)