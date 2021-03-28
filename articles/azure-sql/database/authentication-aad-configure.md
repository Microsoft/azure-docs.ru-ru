---
title: Настройка аутентификации Azure Active Directory
titleSuffix: Azure SQL Database & SQL Managed Instance & Azure Synapse Analytics
description: Узнайте, как подключиться к базе данных SQL, SQL Управляемый экземпляр и Azure синапсе Analytics с помощью Azure Active Directory проверки подлинности после настройки Azure AD.
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
ms.custom: azure-synapse, has-adal-ref, sqldbrb=2, devx-track-azurecli
ms.devlang: ''
ms.topic: how-to
author: GithubMirek
ms.author: mireks
ms.reviewer: vanto, sstein
ms.date: 08/17/2020
ms.openlocfilehash: 9e7b337d4358f9685d683c308d6df9110607207a
ms.sourcegitcommit: c8b50a8aa8d9596ee3d4f3905bde94c984fc8aa2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/28/2021
ms.locfileid: "105643414"
---
# <a name="configure-and-manage-azure-ad-authentication-with-azure-sql"></a>Настройка и администрирование проверки подлинности Azure Active Directory с помощью Azure SQL

[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

В этой статье показано, как создать и заполнить экземпляр Azure Active Directory (Azure AD), а затем использовать Azure AD с [базой данных SQL Azure](sql-database-paas-overview.md), [управляемый экземпляр SQL Azure](../managed-instance/sql-managed-instance-paas-overview.md)и [Azure синапсе Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md). Общие сведения см. в разделе [Azure Active Directoryная проверка подлинности](authentication-aad-overview.md).

## <a name="azure-ad-authentication-methods"></a>Методы проверки подлинности Azure AD

Аутентификация Azure AD поддерживает следующие методы проверки подлинности:

- Удостоверения в облаке Azure AD
- Гибридные удостоверения Azure AD, которые поддерживают:
  - Проверка подлинности в облаке с двумя вариантами с помощью простого единого входа (SSO)
    - Хэш-аутентификация паролей Azure AD
    - Сквозная проверка подлинности Azure AD
  - Федеративная аутентификация

Дополнительные сведения о методах проверки подлинности Azure AD и их выборе см. в разделе [Выбор правильного метода проверки подлинности для Azure Active Directory гибридного решения для идентификации](../../active-directory/hybrid/choose-ad-authn.md).

Дополнительные сведения о гибридных удостоверениях, установке и синхронизации Azure AD см. в следующих статьях:

- Проверка подлинности хэша пароля. [Реализация синхронизации хэшей паролей с Azure AD Connect синхронизацией](../../active-directory/hybrid/how-to-connect-password-hash-synchronization.md)
- Сквозная проверка подлинности — [Azure Active Directory сквозная проверка подлинности](../../active-directory/hybrid/how-to-connect-pta-quick-start.md)
- Федеративная проверка подлинности. [развертывание службы федерации Active Directory (AD FS) в Azure](/windows-server/identity/ad-fs/deployment/how-to-connect-fed-azure-adfs) и [Azure AD Connect и Федерации](../../active-directory/hybrid/how-to-connect-fed-whatis.md)

## <a name="create-and-populate-an-azure-ad-instance"></a>Создание и заполнение экземпляра Azure AD

Создайте экземпляр Azure AD и заполните его пользователями и группами. Azure AD может быть исходным управляемым доменом Azure AD. Azure AD может также быть локальной доменной службой Active Directory, объединенной в федерацию с Azure AD.

Дополнительные сведения см. в статьях [Интеграция локальных удостоверений с Azure Active Directory](../../active-directory/hybrid/whatis-hybrid-identity.md), [Добавление имени личного домена в Azure Active Directory](../../active-directory/fundamentals/add-custom-domain.md), [Microsoft Azure now supports federation with Windows Server Active Directory](https://azure.microsoft.com/blog/20../../windows-azure-now-supports-federation-with-windows-server-active-directory/) (Microsoft Azure теперь поддерживает федерацию с Windows Server Active Directory), [Управление каталогом Azure AD](../../active-directory/fundamentals/active-directory-whatis.md), [Azure Active Directory Cmdlets](/powershell/azure/) (Командлеты для Azure Active Directory) и [Порты и протоколы, необходимые для гибридной идентификации](../../active-directory/hybrid/reference-connect-ports.md).

## <a name="associate-or-add-an-azure-subscription-to-azure-active-directory"></a>Связывание подписки Azure с Azure Active Directory или добавление ее в службу

1. Свяжите свою подписку Azure с Azure Active Directory, сделав каталог доверенным для подписки Azure, в которой размещена база данных. Дополнительные сведения см. [в статье связывание или Добавление подписки Azure в клиент Azure Active Directory](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).

2. Используйте переключатель каталогов на портале Azure, чтобы перейти к подписке, связанной с предметной областью.

   > [!IMPORTANT]
   > Между каждой подпиской Azure и экземпляром Azure AD установлено отношение доверия. Это означает, что она доверяет каталогу проверять подлинность пользователей, служб и устройств. Несколько подписок могут доверять одному и тому же каталогу, но одна конкретная подписка доверяет только одному каталогу. Данное отношение доверия, которое подписка имеет с каталогом, отличается от отношения, которую подписка имеет со всеми другими ресурсами в Azure (веб-сайтами, базами данных и т. д.), которые больше похожи на дочерние ресурсы подписки. Если срок действия подписки истекает, доступ к другим ресурсам, связанным с этой подпиской, также прекращается. Однако каталог останется в Azure, вы можете связать другую подписку с этим каталогом и продолжать управлять пользователями каталога. Дополнительные сведения о ресурсах см. в статье, посвященной [доступу к ресурсам в Azure](../../active-directory/external-identities/add-users-administrator.md). Дополнительные сведения об этих отношениях доверия см. в статье о [связывании подписки Azure с Azure Active Directory или добавлении ее в службу](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).

## <a name="azure-ad-admin-with-a-server-in-sql-database"></a>Администратор Azure AD с сервером в базе данных SQL

Каждый [сервер](logical-servers.md) в Azure (где размещается база данных SQL или Azure синапсе) начинается с одной учетной записи администратора сервера, которая является администратором всего сервера. Создайте вторую учетную запись администратора в качестве учетной записи Azure AD. Этот участник создается как пользователь автономной базы данных в базе данных master сервера. Учетные записи администратора являются членами роли **db_owner** в каждой пользовательской базе данных, и каждая пользовательская база данных вводится как пользователь **dbo** . Дополнительные сведения об учетных записях администратора см. в разделе [Управление базами данных и именами входа](logins-create-manage.md).

При использовании Azure Active Directory с георепликацией необходимо настроить администратора Azure Active Directory для сервера-источника и сервера-получателя. Если у сервера нет администратора Azure Active Directory, Azure Active Directory имена входа и пользователи получают `Cannot connect` ошибку сервера.

> [!NOTE]
> Пользователи, не основанные на учетной записи Azure AD (включая учетную запись администратора сервера), не могут создавать пользователей на основе Azure AD, так как они не имеют разрешения на проверку предложенных пользователей базы данных в Azure AD.

## <a name="provision-azure-ad-admin-sql-managed-instance"></a>Инициализация администратора Azure AD (Управляемый экземпляр SQL)

> [!IMPORTANT]
> Выполните следующие действия, только если вы подготавливаете Управляемый экземпляр Azure SQL. Эта операция может выполняться только глобальным администратором или администратором привилегированных ролей в Azure AD.
>
> В **общедоступной предварительной версии** роль " **читатели каталога** " можно назначить группе в Azure AD. Владельцы групп могут добавить удостоверение управляемого экземпляра в качестве члена этой группы, что позволит вам подготавливать администратора Azure AD для Управляемый экземпляр SQL. Дополнительные сведения об этой функции см. в статье [Роль "Читатели каталогов" в Azure Active Directory для Azure SQL](authentication-aad-directory-readers-role.md).

Управляемый экземпляр SQL требуются разрешения на чтение Azure AD для успешного выполнения таких задач, как проверка подлинности пользователей с помощью членства в группе безопасности или создание новых пользователей. Чтобы это работало, необходимо предоставить разрешение SQL Управляемый экземпляр на чтение Azure AD. Это можно сделать с помощью портал Azure или PowerShell.

### <a name="azure-portal"></a>Портал Azure

Чтобы предоставить разрешение на чтение SQL Управляемый экземпляр Azure AD с помощью портал Azure, войдите в Azure AD в качестве глобального администратора и выполните следующие действия:

1. В [портал Azure](https://portal.azure.com)в правом верхнем углу выберите подключение из раскрывающегося списка возможных каталогов Active Directory.

2. Выберите нужный экземпляр Active Directory в качестве Azure AD по умолчанию.

   Этот шаг связывает подписку, связанную с Active Directory, с Управляемый экземпляр SQL, убедившись, что одна и та же подписка используется как для экземпляра Azure AD, так и для Управляемый экземпляр SQL.

3. Перейдите к Управляемый экземпляр SQL, который вы хотите использовать для интеграции Azure AD.

   ![Снимок экрана с портал Azure отображением страницы администратора Active Directory, открытой для выбранного управляемого экземпляра SQL.](./media/authentication-aad-configure/aad.png)

4. Выберите баннер в верхней части страницы администрирования Active Directory и предоставьте разрешение текущему пользователю.

    ![Снимок экрана диалогового окна для предоставления разрешений управляемому экземпляру SQL для доступа к Active Directory. Выбрана кнопка предоставить разрешения.](./media/authentication-aad-configure/grant-permissions.png)

5. После выполнения операции в правом верхнем углу отобразится следующее уведомление:

    ![Снимок экрана уведомления о том, что разрешения на чтение Active Directory успешно обновлены для управляемого экземпляра.](./media/authentication-aad-configure/success.png)

6. Теперь вы можете выбрать администратора Azure AD для Управляемый экземпляр SQL. Для этого на странице "Администратор Active Directory" щелкните **Задать администратора**.

    ![Снимок экрана, на котором показана команда "задать Администратор", выделенная на странице администрирования Active Directory для выбранного управляемого экземпляра SQL.](./media/authentication-aad-configure/set-admin.png)

7. На странице Администратор Azure AD найдите пользователя, выберите пользователя или группу в качестве администратора, а затем нажмите **кнопку Выбрать**.

   На странице "Администратор Active Directory" отобразятся все участники и группы Active Directory. Пользователей или группы, которые выделены серым цветом, нельзя выбрать; они не поддерживаются ролью администратора Azure AD. Список поддерживаемых администраторов приведен в разделе [Функции и ограничения Azure AD](authentication-aad-overview.md#azure-ad-features-and-limitations). Управление доступом на основе ролей Azure (Azure RBAC) применяется только к портал Azure и не распространяется на базу данных SQL, Управляемый экземпляр SQL или Azure синапсе.

    ![Добавление администратора Azure Active Directory](./media/authentication-aad-configure/add-azure-active-directory-admin.png)

8. В верхней части страницы Администратор Active Directory нажмите кнопку **Сохранить**.

    ![Снимок экрана со страницей администрирования Active Directory с кнопкой "Сохранить" в верхней строке рядом с кнопками "задать администратора" и "удалить Администрирование".](./media/authentication-aad-configure/save.png)

    Изменение администратора может занять несколько минут. После этого новый администратор появится в поле "Администратор Active Directory".

После подготовки администратора Azure AD для Управляемый экземпляр SQL можно приступить к созданию участников сервера Azure AD (имен входа) с помощью синтаксиса [CREATE LOGIN](/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current&preserve-view=true) . Дополнительные сведения см. в разделе [SQL управляемый экземпляр Overview](../managed-instance/sql-managed-instance-paas-overview.md#azure-active-directory-integration).

> [!TIP]
> Чтобы потом удалить учетную запись администратора, в верхней части страницы Администратор Active Directory щелкните **Удалить администратора**, а затем нажмите кнопку **Сохранить**.

### <a name="powershell"></a>PowerShell

Чтобы предоставить разрешение на чтение SQL Управляемый экземпляр Azure AD с помощью PowerShell, выполните следующий скрипт:

```powershell
# Gives Azure Active Directory read permission to a Service Principal representing the SQL Managed Instance.
# Can be executed only by a "Global Administrator" or "Privileged Role Administrator" type of user.

$aadTenant = "<YourTenantId>" # Enter your tenant ID
$managedInstanceName = "MyManagedInstance"

# Get Azure AD role "Directory Users" and create if it doesn't exist
$roleName = "Directory Readers"
$role = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq $roleName}
if ($role -eq $null) {
    # Instantiate an instance of the role template
    $roleTemplate = Get-AzureADDirectoryRoleTemplate | Where-Object {$_.displayName -eq $roleName}
    Enable-AzureADDirectoryRole -RoleTemplateId $roleTemplate.ObjectId
    $role = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq $roleName}
}

# Get service principal for your SQL Managed Instance
$roleMember = Get-AzureADServicePrincipal -SearchString $managedInstanceName
$roleMember.Count
if ($roleMember -eq $null) {
    Write-Output "Error: No Service Principals with name '$    ($managedInstanceName)', make sure that managedInstanceName parameter was     entered correctly."
    exit
}
if (-not ($roleMember.Count -eq 1)) {
    Write-Output "Error: More than one service principal with name pattern '$    ($managedInstanceName)'"
    Write-Output "Dumping selected service principals...."
    $roleMember
    exit
}

# Check if service principal is already member of readers role
$allDirReaders = Get-AzureADDirectoryRoleMember -ObjectId $role.ObjectId
$selDirReader = $allDirReaders | where{$_.ObjectId -match     $roleMember.ObjectId}

if ($selDirReader -eq $null) {
    # Add principal to readers role
    Write-Output "Adding service principal '$($managedInstanceName)' to     'Directory Readers' role'..."
    Add-AzureADDirectoryRoleMember -ObjectId $role.ObjectId -RefObjectId     $roleMember.ObjectId
    Write-Output "'$($managedInstanceName)' service principal added to     'Directory Readers' role'..."

    #Write-Output "Dumping service principal '$($managedInstanceName)':"
    #$allDirReaders = Get-AzureADDirectoryRoleMember -ObjectId $role.ObjectId
    #$allDirReaders | where{$_.ObjectId -match $roleMember.ObjectId}
}
else {
    Write-Output "Service principal '$($managedInstanceName)' is already     member of 'Directory Readers' role'."
}
```

### <a name="powershell-for-sql-managed-instance"></a>PowerShell для SQL Управляемый экземпляр

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы выполнять командлеты PowerShell, необходимо установить и запустить Azure PowerShell. Дополнительные сведения можно узнать в статье [Установка и настройка Azure PowerShell](/powershell/azure/).

> [!IMPORTANT]
> Модуль PowerShell Azure Resource Manager (RM) по-прежнему поддерживается Управляемый экземпляр SQL Azure, но вся будущая разработка предназначена для модуля AZ. SQL. Исправления ошибок для модуля AzureRM будут продолжать выпускаться как минимум до декабря 2020 г.  Аргументы команд в модулях Az и AzureRm практически идентичны. Дополнительные сведения о совместимости см. в статье [Знакомство с новым модулем Az для Azure PowerShell](/powershell/azure/new-azureps-module-az).

Чтобы подготовить администратора Azure AD, выполните следующие команды Azure PowerShell.

- Connect-AzAccount
- Select-AzSubscription

Командлеты, используемые для подготовки администратора Azure AD к Управляемый экземпляр SQL и управления им, перечислены в следующей таблице:

| Имя командлета | Описание |
| --- | --- |
| [Set-Азсклинстанцеактиведиректорядминистратор](/powershell/module/az.sql/set-azsqlinstanceactivedirectoryadministrator) |Подготовка администратора Azure AD для Управляемый экземпляр SQL в текущей подписке. (Должно быть из текущей подписки)|
| [Remove-Азсклинстанцеактиведиректорядминистратор](/powershell/module/az.sql/remove-azsqlinstanceactivedirectoryadministrator) |Удаляет администратора Azure AD для Управляемый экземпляр SQL в текущей подписке. |
| [Get-Азсклинстанцеактиведиректорядминистратор](/powershell/module/az.sql/get-azsqlinstanceactivedirectoryadministrator) |Возвращает сведения о администраторе Azure AD для Управляемый экземпляр SQL в текущей подписке.|

Следующая команда получает сведения о администраторе Azure AD для Управляемый экземпляр SQL с именем ManagedInstance01, связанного с группой ресурсов с именем ResourceGroup01.

```powershell
Get-AzSqlInstanceActiveDirectoryAdministrator -ResourceGroupName "ResourceGroup01" -InstanceName "ManagedInstance01"
```

Следующая команда подготавливает группу администраторов Azure AD с именем DBA для Управляемый экземпляр SQL с именем ManagedInstance01. Этот сервер связан с группой ресурсов ResourceGroup01.

```powershell
Set-AzSqlInstanceActiveDirectoryAdministrator -ResourceGroupName "ResourceGroup01" -InstanceName "ManagedInstance01" -DisplayName "DBAs" -ObjectId "40b79501-b343-44ed-9ce7-da4c8cc7353b"
```

Следующая команда удаляет администратора Azure AD для Управляемый экземпляр SQL с именем ManagedInstanceName01, связанного с группой ресурсов ResourceGroup01.

```powershell
Remove-AzSqlInstanceActiveDirectoryAdministrator -ResourceGroupName "ResourceGroup01" -InstanceName "ManagedInstanceName01" -Confirm -PassThru
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Вы также можете подготавливать администратора Azure AD для Управляемый экземпляр SQL, вызвав следующие команды интерфейса командной строки:

| Get-Help | Описание |
| --- | --- |
|[AZ SQL MI AD — создание администратора](/cli/azure/sql/mi/ad-admin#az-sql-mi-ad-admin-create) | Подготавливает администратора Azure Active Directory для Управляемый экземпляр SQL (должен быть из текущей подписки). |
|[AZ SQL MI AD — администратор удаление](/cli/azure/sql/mi/ad-admin#az-sql-mi-ad-admin-delete) | Удаляет Azure Active Directory администратора для Управляемый экземпляр SQL. |
|[AZ SQL MI AD — список администраторов](/cli/azure/sql/mi/ad-admin#az-sql-mi-ad-admin-list) | Возвращает сведения об администраторе Azure Active Directory, настроенном в настоящее время для Управляемый экземпляр SQL. |
|[AZ SQL MI AD — обновление администратора](/cli/azure/sql/mi/ad-admin#az-sql-mi-ad-admin-update) | Обновляет Active Directory администратора Управляемый экземпляр SQL. |

Дополнительные сведения о командах CLI см. в разделе [AZ SQL MI](/cli/azure/sql/mi).

* * *

## <a name="provision-azure-ad-admin-sql-database"></a>Инициализация администратора Azure AD (база данных SQL)

> [!IMPORTANT]
> Выполните следующие действия, только если вы подготавливаете [сервер](logical-servers.md) для базы данных SQL или Azure синапсе.

В следующих двух процедурах показано, как подготавливать администратора Azure Active Directory для сервера в портал Azure и с помощью PowerShell.

### <a name="azure-portal"></a>Портал Azure

1. В правом верхнем углу [портала Azure](https://portal.azure.com/) щелкните имя подключения, чтобы открыть список доступных каталогов Active Directory. Выберите нужный экземпляр Active Directory в качестве Azure AD по умолчанию. Этот шаг связывает связанную с подпиской Active Directory с сервером, убедившись, что одна и та же подписка используется как для Azure AD, так и для сервера.

2. Найдите и выберите **SQL Server**.

    ![Найдите и выберите серверы SQL Server.](./media/authentication-aad-configure/search-for-and-select-sql-servers.png)

    >[!NOTE]
    > На этой странице, прежде чем выбрать **Серверы SQL Server**, можно щелкнуть **звезду** рядом с именем, чтобы добавить категорию в *Избранное* и добавить **серверы SQL Server** на левую панель навигации.

3. На странице **SQL Server** выберите **Active Directory администратор**.

4. На странице **Администратор Active Directory** щелкните **Задать администратора**.

    ![Серверы SQL Server, настроенные Active Directory Admin](./media/authentication-aad-configure/sql-servers-set-active-directory-admin.png)  

5. На странице **Добавление администратора** найдите пользователя, выберите пользователя или группу в качестве администратора, а затем нажмите **кнопку Выбрать**. На странице "Администратор Active Directory" отобразятся все участники и группы Active Directory. Пользователей или группы, которые выделены серым цветом, нельзя выбрать; они не поддерживаются ролью администратора Azure AD. (См. список поддерживаемых администраторов в разделе **функции и ограничения Azure AD** в статье [Использование Azure Active Directory проверки подлинности для проверки подлинности с помощью базы данных SQL или Azure синапсе](authentication-aad-overview.md).) Управление доступом на основе ролей Azure (Azure RBAC) применяется только к порталу и не распространяется на SQL Server.

    ![Выбор Azure Active Directory администратора](./media/authentication-aad-configure/select-azure-active-directory-admin.png)  

6. В верхней части страницы **Администратор Active Directory** нажмите кнопку **Сохранить**.

    ![сохранение администратора](./media/authentication-aad-configure/save-admin.png)

Изменение администратора может занять несколько минут. После этого новый администратор появится в поле **Администратор Active Directory** .

   > [!NOTE]
   > При настройке администратора Azure AD новое имя администратора (пользователя или группы) уже не может присутствовать в виртуальной базе данных master в качестве пользователя проверки подлинности сервера. Если оно присутствует, настройка администратора Azure AD завершится ошибкой. Будет выполнен откат его создания и появится сообщение, что такой администратор (имя) уже существует. Поскольку такой пользователь проверки подлинности сервера не является частью Azure AD, все усилия по подключению к серверу с помощью проверки подлинности Azure AD не могут быть успешными.

Чтобы потом удалить учетную запись администратора, в верхней части страницы **Администратор Active Directory** щелкните **Удалить администратора**, а затем нажмите кнопку **Сохранить**.

### <a name="powershell-for-sql-database-and-azure-synapse"></a>PowerShell для базы данных SQL и Azure синапсе

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы выполнять командлеты PowerShell, необходимо установить и запустить Azure PowerShell. Дополнительные сведения можно узнать в статье [Установка и настройка Azure PowerShell](/powershell/azure/). Чтобы подготовить администратора Azure AD, выполните следующие команды Azure PowerShell.

- Connect-AzAccount
- Select-AzSubscription

Командлеты, используемые для инициализации и управления администратором Azure AD для базы данных SQL и Azure синапсе:

| Имя командлета | Описание |
| --- | --- |
| [Set-AzSqlServerActiveDirectoryAdministrator](/powershell/module/az.sql/set-azsqlserveractivedirectoryadministrator) |Подготавливает администратора Azure Active Directory для сервера, на котором размещена база данных SQL или Azure синапсе. (Должно быть из текущей подписки) |
| [Remove-Азсклсерверактиведиректорядминистратор](/powershell/module/az.sql/remove-azsqlserveractivedirectoryadministrator) |Удаляет Azure Active Directory администратора для сервера, на котором размещена база данных SQL или Azure синапсе.|
| [Get-Азсклсерверактиведиректорядминистратор](/powershell/module/az.sql/get-azsqlserveractivedirectoryadministrator) |Возвращает сведения о Azure Active Directory администраторе, настроенном в настоящее время для сервера, на котором размещена база данных SQL или Azure синапсе. |

Чтобы просмотреть дополнительные сведения о каждой из этих команд, используйте команду PowerShell Get-Help. Например, `get-help Set-AzSqlServerActiveDirectoryAdministrator`.

Следующий сценарий подготавливает группу администраторов Azure AD с именем **DBA_Group** (идентификатор объекта `40b79501-b343-44ed-9ce7-da4c8cc7353f` ) для сервера **demo_server** в группе ресурсов с именем **Group-23**:

```powershell
Set-AzSqlServerActiveDirectoryAdministrator -ResourceGroupName "Group-23" -ServerName "demo_server" -DisplayName "DBA_Group"
```

Входной параметр **DisplayName** принимает отображаемое имя Azure AD или имя участника-пользователя. Например, ``DisplayName="John Smith"`` и ``DisplayName="johns@contoso.com"``. Для групп Azure AD поддерживается только отображаемое имя Azure AD.

> [!NOTE]
> Команда Azure PowerShell ```Set-AzSqlServerActiveDirectoryAdministrator``` не запрещает подготовку администраторов Azure AD для неподдерживаемых пользователей. Неподдерживаемого пользователя можно подготовить, но он не сможет подключиться к базе данных (См.

В следующем примере используется необязательный параметр **ObjectID**:

```powershell
Set-AzSqlServerActiveDirectoryAdministrator -ResourceGroupName "Group-23" -ServerName "demo_server" `
    -DisplayName "DBA_Group" -ObjectId "40b79501-b343-44ed-9ce7-da4c8cc7353f"
```

> [!NOTE]
> Параметр **ObjectID** Azure AD является обязательным, если параметр **DisplayName** не уникален. Чтобы получить значения **ObjectID** и **DisplayName**, используйте раздел Active Directory классического портала Azure. Там можно просмотреть свойства пользователя или группы.

В следующем примере возвращаются сведения о текущем администраторе Azure AD для сервера:

```powershell
Get-AzSqlServerActiveDirectoryAdministrator -ResourceGroupName "Group-23" -ServerName "demo_server" | Format-List
```

В примере ниже выполняется удаление администратора Azure AD.

```powershell
Remove-AzSqlServerActiveDirectoryAdministrator -ResourceGroupName "Group-23" -ServerName "demo_server"
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Вы можете подготавливать администратора Azure AD, вызвав следующие команды интерфейса командной строки:

| Get-Help | Описание |
| --- | --- |
|[az sql server ad-admin create](/cli/azure/sql/server/ad-admin#az-sql-server-ad-admin-create) | Подготавливает администратора Azure Active Directory для сервера, на котором размещена база данных SQL или Azure синапсе. (Должно быть из текущей подписки) |
|[az sql server ad-admin delete](/cli/azure/sql/server/ad-admin#az-sql-server-ad-admin-delete) | Удаляет Azure Active Directory администратора для сервера, на котором размещена база данных SQL или Azure синапсе. |
|[az sql server ad-admin list](/cli/azure/sql/server/ad-admin#az-sql-server-ad-admin-list) | Возвращает сведения о Azure Active Directory администраторе, настроенном в настоящее время для сервера, на котором размещена база данных SQL или Azure синапсе. |
|[az sql server ad-admin update](/cli/azure/sql/server/ad-admin#az-sql-server-ad-admin-update) | Обновляет Active Directory администратор сервера, на котором размещена база данных SQL или Azure синапсе. |

Дополнительные сведения о командах интерфейса командной строки см. в разделе [AZ SQL Server](/cli/azure/sql/server).

* * *

> [!NOTE]
> Кроме того, можно подготовить администратора Azure Active Directory с помощью интерфейсов REST API. Дополнительные сведения см. в справочнике по REST API управления службами и статье [Operations for Azure SQL Databases](/rest/api/sql/) (Операции для Базы данных SQL Azure).

## <a name="configure-your-client-computers"></a>Настройка клиентских компьютеров

На всех клиентских компьютерах, из которых приложения или пользователи подключаются к базе данных SQL или Azure синапсе с помощью удостоверений Azure AD, необходимо установить следующее программное обеспечение:

- Платформа .NET Framework 4,6 или более поздней версии с [https://msdn.microsoft.com/library/5a4x27ek.aspx](/dotnet/framework/install/guide-for-developers) .
- Библиотека проверки подлинности Azure Active Directory для SQL Server (*ADAL.DLL*). Ниже приведены ссылки для загрузки для установки последнего драйвера SSMS, ODBC и OLE DB, содержащего библиотеку *ADAL.DLL* .
  - [Среда SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms)
  - [ODBC Driver for SQL Server версии 17](/sql/connect/odbc/download-odbc-driver-for-sql-server?view=sql-server-ver15&preserve-view=true)
  - [OLE DB драйвер 18 для SQL Server](/sql/connect/oledb/download-oledb-driver-for-sql-server?view=sql-server-ver15&preserve-view=true)

Вы можете выполнить эти требования, сделав следующее:

- Установка последней версии [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) или [SQL Server Data Tools](/sql/ssdt/download-sql-server-data-tools-ssdt) соответствует требованиям платформа .NET Framework 4,6.
  - SSMS устанавливает версию x86 *ADAL.DLL*.
  - SSDT устанавливает версию amd64 *ADAL.DLL*.
  - Последняя версия Visual Studio из [файлов для загрузки Visual Studio](https://www.visualstudio.com/downloads/download-visual-studio-vs) соответствует требованиям платформа .NET Framework 4,6, но не устанавливает требуемую версию amd64 *ADAL.DLL*.

## <a name="create-contained-users-mapped-to-azure-ad-identities"></a>Создание автономных пользователей, сопоставленных с удостоверениями Azure AD

Поскольку SQL Управляемый экземпляр поддерживает субъекты сервера Azure AD (имена входа), использование пользователей автономной базы данных не требуется. Субъекты сервера (имена для входа) Azure AD позволяют создавать имена для входа для пользователей, групп или приложений Azure AD. Это означает, что вы можете пройти проверку подлинности в Управляемый экземпляр SQL с помощью имени входа сервера Azure AD, а не пользователя автономной базы данных. Дополнительные сведения см. в разделе [SQL управляемый экземпляр Overview](../managed-instance/sql-managed-instance-paas-overview.md#azure-active-directory-integration). Сведения о синтаксисе для создания субъектов сервера (имен для входа) Azure AD см. в статье [CREATE LOGIN (Transact-SQL)](/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current&preserve-view=true).

Однако использование Azure Active Directory проверки подлинности с базой данных SQL и Azure синапсе требует использования пользователей автономной базы данных на основе удостоверения Azure AD. Пользователь автономной базы данных не имеет имени входа в базе данных master и сопоставляется с удостоверением в Azure AD, связанным с базой данных. Удостоверение Azure AD может быть учетной записью отдельного пользователя или группы. Дополнительные сведения о пользователях автономной базы данных см. в статье [Пользователи автономной базы данных — создание переносимой базы данных](/sql/relational-databases/security/contained-database-users-making-your-database-portable).

> [!NOTE]
> Пользователей баз данных (за исключением администраторов) невозможно создавать с помощью портала Azure. Роли Azure не распространяются в базу данных в базе данных SQL, Управляемый экземпляр SQL или Azure синапсе. Роли Azure используются для управления ресурсами Azure и не применяются к разрешениям базы данных. Например, роль **участника SQL Server** не предоставляет доступ для подключения к базе данных в базе данных sql, управляемый экземпляр SQL или Azure синапсе. Разрешение доступа должно быть предоставлено непосредственно в базе данных с помощью инструкций Transact-SQL.

> [!WARNING]
> Специальные символы, такие как двоеточие `:` или амперсанд, `&` если они включены в качестве имен пользователей в T-SQL `CREATE LOGIN` , а `CREATE USER` инструкции не поддерживаются.

Вы можете создать пользователя автономной базы данных на основе Azure AD (кроме администратора сервера, который является владельцем базы данных). Для этого подключитесь к базе данных с помощью удостоверения Azure AD от имени пользователя с уровнем разрешений не ниже, чем **Изменение любого пользователя**. Затем используйте следующий синтаксис Transact-SQL:

```sql
CREATE USER <Azure_AD_principal_name> FROM EXTERNAL PROVIDER;
```

Значением параметра *Azure_AD_principal_name* может быть имя участника-пользователя Azure AD или отображаемое имя группы Azure AD.

**Примеры.** Создание пользователя автономной базы данных, представляющего собой пользователя федеративного или управляемого домена Azure AD:

```sql
CREATE USER [bob@contoso.com] FROM EXTERNAL PROVIDER;
CREATE USER [alice@fabrikam.onmicrosoft.com] FROM EXTERNAL PROVIDER;
```

Чтобы создать пользователя автономной базы данных, представляющего собой группу Azure AD или группу федеративного домена, укажите отображаемое имя группы безопасности.

```sql
CREATE USER [ICU Nurses] FROM EXTERNAL PROVIDER;
```

Чтобы создать пользователя автономной базы данных, который представляет приложение и будет подключаться с помощью маркера Azure AD, выполните следующее.

```sql
CREATE USER [appName] FROM EXTERNAL PROVIDER;
```

> [!NOTE]
> Для этой команды требуется, чтобы SQL Access Azure AD ("внешний поставщик") от имени вошедшего в систему пользователя. В некоторых случаях может возникнуть ситуация, из – за которой Azure AD возвращает исключение обратно в SQL. В таких случаях пользователь увидит ошибку SQL 33134, которая должна содержать сообщение об ошибке, специфическое для Azure AD. В большинстве случаев ошибка говорит о том, что доступ запрещен или пользователь должен зарегистрировать в MFA для доступа к ресурсу, или что доступ между сторонними приложениями должен обрабатываться через предварительную авторизацию. В первых двух случаях эта неполадка обычно вызвана политиками условного доступа, заданными в клиенте Azure AD пользователя: они не позволяют пользователю получить доступ к внешнему поставщику. Обновление политик условного доступа для разрешения доступа к приложению "00000002-0000-0000-C000-000000000000" (идентификатор приложения Azure AD API Graph) должно устранить эту проблему. Если ошибка говорит о том, что доступ между приложениями первого производителя должен обрабатываться через предварительную авторизацию, это связано с тем, что пользователь вошел в качестве субъекта-службы. Команда должна быть выполнена, если вместо этого она выполняется пользователем.

> [!TIP]
> Создать пользователя напрямую из каталога Azure Active Directory можно только в том случае, если этот каталог связан с вашей подпиской Azure. Однако членов других каталогов Active Directory, являющихся импортированными пользователями в связанном каталоге Active Directory (так называемыми внешними пользователями), можно добавить в группу Active Directory в клиенте Active Directory. Создав пользователя автономной базы данных для этой группы AD, пользователи из внешнего каталога Active Directory могут получить доступ к базе данных SQL.

Подробные сведения о создании пользователей автономной базы данных на основе удостоверений Azure Active Directory см. в статье [CREATE USER (Transact-SQL)](/sql/t-sql/statements/create-user-transact-sql).

> [!NOTE]
> Удаление администратора Azure Active Directory для сервера не позволяет любому пользователю проверки подлинности Azure AD подключаться к серверу. При необходимости администратор Базы данных SQL может вручную удалить неиспользуемых пользователей Azure AD.

> [!NOTE]
> При получении ответа **Истек тайм-аут подключения**, возможно, потребуется задать для параметра `TransparentNetworkIPResolution` в строке подключения значение false. Дополнительные сведения см. в статье [Connection timeout issue with .NET Framework 4.6.1 — TransparentNetworkIPResolution](/archive/blogs/dataaccesstechnologies/connection-timeout-issue-with-net-framework-4-6-1-transparentnetworkipresolution) (Проблема с временем ожидания подключения в .NET Framework 4.6.1 — TransparentNetworkIPResolution).

При создании пользователь базы данных получает разрешение **Подключение** и может подключаться к этой базе данных как участник роли **Public**. Изначально пользователю доступны только разрешения, предоставленные роли **Public**, или другие разрешения, предоставленные любой группе Azure AD, в которую входит пользователь. Подготовив пользователя автономной базы данных, использующей Azure AD, можно предоставить ему дополнительные разрешения — так же, как разрешения для любого другого типа пользователя. Обычно разрешения предоставляются ролям базы данных, а затем эти роли назначаются пользователям. Подробные сведения см. в статье [Database Engine Permission Basics](https://social.technet.microsoft.com/wiki/contents/articles/4433.database-engine-permission-basics.aspx) (Основные сведения о разрешениях ядра СУБД). Дополнительные сведения о специальных ролях базы данных SQL см. в статье [Проверка подлинности и авторизация в базе данных SQL Azure: предоставление доступа](logins-create-manage.md).
Для учетной записи пользователя федеративного домена, импортируемая в управляемый домен, следует использовать идентификатор управляемого домена.

> [!NOTE]
> Пользователи Azure AD помечаются в метаданных базы данных как тип E (EXTERNAL_USER), а группы — как тип X (EXTERNAL_GROUPS). Дополнительные сведения см. в статье [sys.database_principals (Transact-SQL)](/sql/relational-databases/system-catalog-views/sys-database-principals-transact-sql).

## <a name="connect-to-the-database-using-ssms-or-ssdt"></a>Подключение к базе данных с помощью SSMS или SSDT  

Чтобы убедиться, что администратор Azure AD настроен правильно, подключитесь к базе данных **master** с помощью учетной записи администратора Azure AD.
Вы можете подготовить пользователя автономной базы данных, использующей Azure AD (кроме администратора сервера, который является владельцем базы данных). Для этого подключитесь к базе данных с помощью удостоверения Azure AD, у которого есть доступ к базе данных.

> [!IMPORTANT]
> Проверку подлинности Azure Active Directory поддерживают [SQL Server 2016 Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) и [SQL Server Data Tools](/sql/ssdt/download-sql-server-data-tools-ssdt) для Visual Studio 2015. Выпуск SSMS за август 2016 года также включает в себя поддержку универсальной аутентификации Active Directory, что позволяет администраторам требовать прохождения многофакторной проверки подлинности с помощью телефонного звонка, текстового сообщения, смарт-карты с ПИН-кодом или уведомления в мобильном приложении.

## <a name="using-an-azure-ad-identity-to-connect-using-ssms-or-ssdt"></a>Использование удостоверения Azure AD для подключения с помощью SSMS или SSDT

В следующих процедурах показано, как подключиться к базе данных SQL с удостоверением Azure AD с помощью SQL Server Management Studio или средств SQL Server Database Tools.

### <a name="active-directory-integrated-authentication"></a>Встроенная аутентификация Active Directory

Используйте этот метод, если вы выполнили вход в Windows с помощью учетных данных Azure Active Directory федеративного домена или управляемого домена, настроенного для простого единого входа и проверки подлинности с помощью хэша паролей. Дополнительные сведения см. в разделе [Устранение неполадок с простым единым входом Azure Active Directory](../../active-directory/hybrid/how-to-connect-sso.md).

1. Запустите Management Studio или средства данных и в диалоговом окне **Подключение к серверу** (или **Подключение к ядро СУБД**) в поле **authentication (проверка подлинности** ) выберите **Azure Active Directory-Integrated**. Пароль не требуется (и его нельзя ввести), так как для подключения используются существующие учетные данные.

   ![Выбор встроенной аутентификации Active Directory][11]

2. Нажмите кнопку **Параметры**, а затем на странице **Свойства подключения** в поле **Подключение к базе данных** введите имя пользовательской базы данных для подключения. Дополнительные сведения см. в статье [многофакторная проверка подлинности Azure AD](authentication-mfa-ssms-overview.md#azure-ad-domain-name-or-tenant-id-parameter) с учетом различий между свойствами подключения для SSMS 17. x и 18. x.

   ![Выбор имени базы данных][13]

### <a name="active-directory-password-authentication"></a>Проверка пароля Active Directory

Используйте этот метод при подключении с помощью имени участника Azure AD, в котором используется управляемый домен Azure AD. Его также можно использовать для федеративных учетных записей без доступа к домену, например при удаленной работе.

Используйте этот метод для проверки подлинности базы данных в базе данных SQL или SQL Управляемый экземпляр с пользователями удостоверений в облаке Azure AD или только теми, кто использует гибридные удостоверения Azure AD. Этот метод поддерживает пользователей, которые хотят использовать учетные данные Windows, но их локальный компьютер не присоединен к домену (например, с помощью удаленного доступа). В этом случае пользователь Windows может указать свою учетную запись домена и пароль, а также пройти проверку подлинности в базе данных SQL, Управляемый экземпляр SQL или Azure синапсе.

1. Запустите Management Studio или средства данных и в диалоговом окне **Подключение к серверу** (или **Подключение к ядро СУБД**) в поле **проверка подлинности** выберите **Azure Active Directory-Password**.

2. В поле **имя пользователя** введите имя пользователя Azure Active Directory в формате **username \@ domain.com**. Имена пользователей должны быть учетными записями из Azure Active Directory или учетной записи из управляемого или федеративного домена с Azure Active Directory.

3. В поле **пароль** введите пароль пользователя для учетной записи Azure Active Directory или управляемой или федеративной учетной записи домена.

    ![Выбор проверки пароля Active Directory][12]

4. Нажмите кнопку **Параметры**, а затем на странице **Свойства подключения** в поле **Подключение к базе данных** введите имя пользовательской базы данных для подключения. (См. рисунок в предыдущем варианте.)

### <a name="active-directory-interactive-authentication"></a>Active Directoryная Интерактивная проверка подлинности

Используйте этот метод для интерактивной проверки подлинности с многофакторной проверкой подлинности (MFA) или без нее с запросом пароля в интерактивном режиме. Этот метод можно использовать для проверки подлинности в базе данных SQL, Управляемый экземпляр SQL и Azure синапсе для облачных пользователей удостоверений Azure AD или тех, кто использует гибридные удостоверения Azure AD.

Дополнительные сведения см. в статьях [Использование многофакторной аутентификации Azure AD с базой данных SQL и Azure синапсе (поддержка SSMS для MFA)](authentication-mfa-ssms-overview.md).

## <a name="using-an-azure-ad-identity-to-connect-from-a-client-application"></a>Использование удостоверения Azure AD для подключения из клиентского приложения

В следующих процедурах показано, как подключиться к базе данных SQL с удостоверением Azure AD из клиентского приложения.

### <a name="active-directory-integrated-authentication"></a>Встроенная аутентификация Active Directory

Чтобы использовать встроенную проверку подлинности Windows, Active Directory домена должны быть объединены с Azure Active Directory или управляемым доменом, для которого настроен простой единый вход или проверка подлинности с помощью хеша паролей. Дополнительные сведения см. в разделе [Устранение неполадок с простым единым входом Azure Active Directory](../../active-directory/hybrid/how-to-connect-sso.md).

> [!NOTE]
> [MSAL.NET (Microsoft. Identity. Client)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki#roadmap) для встроенной проверки подлинности Windows не поддерживается для простого единого входа для сквозной проверки подлинности и хэширования паролей.

Клиентское приложение (или служба), подключаемое к базе данных, должно работать на компьютере, присоединенном к домену, под учетными данными домена пользователя.

Для подключения к базе данных с помощью встроенной проверки подлинности и удостоверения Azure AD ключевое слово Authentication в строке подключения к базе данных должно иметь значение `Active Directory Integrated` . В следующем примере кода C# используется ADO .NET.

```csharp
string ConnectionString = @"Data Source=n9lxnyuzhv.database.windows.net; Authentication=Active Directory Integrated; Initial Catalog=testdb;";
SqlConnection conn = new SqlConnection(ConnectionString);
conn.Open();
```

Ключевое слово `Integrated Security=True` строки подключения не используется для подключения к базе данных SQL Azure. При создании подключения ODBC потребуется удалить пробелы и задать для параметра Authentication значение ActiveDirectoryIntegrated.

### <a name="active-directory-password-authentication"></a>Проверка пароля Active Directory

Для подключения к базе данных с использованием только облачных учетных записей удостоверений Azure AD или пользователей, использующих гибридные удостоверения Azure AD, ключевому слову Authentication должно быть присвоено значение `Active Directory Password` . Строка подключения должна содержать ключевые слова для идентификатора пользователя (UID) и пароля (PWD), а также их значения. В следующем примере кода C# используется ADO .NET.

```csharp
string ConnectionString =
@"Data Source=n9lxnyuzhv.database.windows.net; Authentication=Active Directory Password; Initial Catalog=testdb;  UID=bob@contoso.onmicrosoft.com; PWD=MyPassWord!";
SqlConnection conn = new SqlConnection(ConnectionString);
conn.Open();
```

Дополнительные сведения о методах проверки подлинности Azure AD и примеры кода вы найдете в [демонстрации проверки подлинности Azure AD на GitHub](https://github.com/Microsoft/sql-server-samples/tree/master/samples/features/security/azure-active-directory-auth).

## <a name="azure-ad-token"></a>Токен Azure AD

Этот метод проверки подлинности позволяет службам среднего уровня получать [веб-маркеры JSON (JWT)](../../active-directory/develop/id-tokens.md) для подключения к базе данных в базе данных SQL, SQL управляемый экземпляр или Azure синапсе, получая маркер из Azure AD. Этот метод позволяет использовать различные сценарии приложений, включая удостоверения служб, субъекты-службы и приложения, использующие проверку подлинности на основе сертификатов. Для работы с проверкой подлинности маркера Azure AD необходимо выполнить четыре основных шага.

1. Зарегистрируйте приложение в Azure Active Directory и получите идентификатор клиента для своего кода.
2. Создайте пользователя базы данных, представляющего приложение. (Выполнено ранее на шаге 6.)
3. Создайте сертификат на клиентском компьютере, на котором выполняется приложение.
4. Добавьте сертификат в качестве ключа для вашего приложения.

Пример строки подключения.

```csharp
string ConnectionString =@"Data Source=n9lxnyuzhv.database.windows.net; Initial Catalog=testdb;"
SqlConnection conn = new SqlConnection(ConnectionString);
conn.AccessToken = "Your JWT token"
conn.Open();
```

Дополнительные сведения см. в [блоге о безопасности SQL Server](/archive/blogs/sqlsecurity/token-based-authentication-support-for-azure-sql-db-using-azure-ad-auth). Дополнительные сведения о добавлении сертификата см. в статье [Приступая к работе с аутентификацией на основе сертификата в Azure Active Directory](../../active-directory/authentication/active-directory-certificate-based-authentication-get-started.md).

### <a name="sqlcmd"></a>sqlcmd

Ниже приведены инструкции для подключения с помощью sqlcmd версии 13.1, доступной в [Центре загрузки](https://www.microsoft.com/download/details.aspx?id=53591).

> [!NOTE]
> `sqlcmd` команда не `-G` работает с системными удостоверениями и требует входа субъекта-пользователя.

```cmd
sqlcmd -S Target_DB_or_DW.testsrv.database.windows.net -G  
sqlcmd -S Target_DB_or_DW.testsrv.database.windows.net -U bob@contoso.com -P MyAADPassword -G -l 30
```

## <a name="troubleshoot-azure-ad-authentication"></a>Устранение неполадок проверки подлинности Azure AD

Руководство по устранению неполадок с аутентификацией Azure AD можно найти в следующем блоге: <https://techcommunity.microsoft.com/t5/azure-sql-database/troubleshooting-problems-related-to-azure-ad-authentication-with/ba-p/1062991>

## <a name="next-steps"></a>Дальнейшие шаги

- Общие сведения о именах входа, пользователях, ролях базы данных и разрешениях в базе данных SQL см. в разделе [имена входа, пользователи, роли базы данных и учетные записи пользователей](logins-create-manage.md).
- Дополнительные сведения о субъектах базы данных см. в [этой статье](/sql/relational-databases/security/authentication-access/principals-database-engine).
- Дополнительные сведения о ролях баз данных см. в статье [Роли уровня базы данных](/sql/relational-databases/security/authentication-access/database-level-roles).
- Дополнительные сведения о правилах брандмауэра см. в статье [Обзор правил брандмауэра базы данных SQL Azure](firewall-configure.md).
- Сведения о том, как настроить гостевой пользователь Azure AD в качестве администратора Azure AD, см. в статье [Создание гостевых пользователей Azure AD и установка в качестве администратора Azure AD](authentication-aad-guest-users.md).
- Сведения о том, как использовать субъекты-службы с Azure SQL, см. в статье [Создание пользователей Azure AD с помощью приложений Azure AD](authentication-aad-service-principal-tutorial.md) .

<!--Image references-->

[11]: ./media/authentication-aad-configure/active-directory-integrated.png
[12]: ./media/authentication-aad-configure/12connect-using-pw-auth2.png
[13]: ./media/authentication-aad-configure/13connect-to-db2.png
