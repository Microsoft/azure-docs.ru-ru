---
title: Субъект-служба Azure Active Directory с SQL Azure
description: Приложения Azure AD (субъекты-службы) поддерживают создание пользователей Azure AD в базе данных SQL Azure, Управляемый экземпляр Azure SQL и Azure синапсе Analytics.
ms.service: sql-db-mi
ms.subservice: security
ms.custom: azure-synapse
ms.topic: conceptual
author: GithubMirek
ms.author: mireks
ms.reviewer: vanto
ms.date: 02/11/2021
ms.openlocfilehash: 68267cdedd2f0b64549791866e8750cf42928ab4
ms.sourcegitcommit: 5f32f03eeb892bf0d023b23bd709e642d1812696
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/12/2021
ms.locfileid: "103201245"
---
# <a name="azure-active-directory-service-principal-with-azure-sql"></a>Субъект-служба Azure Active Directory с SQL Azure

[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

Поддержка создания пользователей Azure Active Directory (Azure AD) в базе данных SQL Azure (SQL DB) и [Azure синапсе Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md) от имени приложений Azure AD (субъектов-служб) в настоящее время **доступна в общедоступной предварительной версии**.

> [!NOTE]
> Эта функция уже поддерживается для SQL Управляемый экземпляр.

## <a name="service-principal-azure-ad-applications-support"></a>Поддержка субъекта-службы (приложения Azure AD)

Эта статья относится к приложениям, интегрированным с Azure AD, и является частью регистрации Azure AD. Этим приложениям часто требуется проверка подлинности и авторизация доступа к Azure SQL для выполнения различных задач. Эта функция в **общедоступной предварительной версии** теперь позволяет субъектам-службам создавать пользователей Azure AD в базе данных SQL и Azure синапсе. Произошло ограничение, препятствующее созданию объектов Azure AD от имени удаленных приложений Azure AD.

При регистрации приложения Azure AD с помощью портал Azure или команды PowerShell в клиенте Azure AD создаются два объекта:

- объект приложения;
- объект субъекта-службы.

Дополнительные сведения о приложениях Azure AD см. [в разделе объекты приложения и субъекта-службы в Azure Active Directory](../../active-directory/develop/app-objects-and-service-principals.md) и [Создайте субъект-службу Azure с Azure PowerShell](/powershell/azure/create-azure-service-principal-azureps).

База данных SQL, Azure синапсе и SQL Управляемый экземпляр поддерживают следующие объекты Azure AD:

- Пользователи Azure AD (управляемые, Федеративные и гостевые)
- Группы Azure AD (управляемые и Федеративные)
-  Приложения Azure AD 

`CREATE USER [Azure_AD_Object] FROM EXTERNAL PROVIDER`Теперь для базы данных SQL и Azure синапсе поддерживается команда T-SQL от имени приложения Azure AD.

## <a name="functionality-of-azure-ad-user-creation-using-service-principals"></a>Функциональные возможности создания пользователей Azure AD с помощью субъектов-служб

Поддержка этой функции полезна в процессах автоматизации приложений Azure AD, где объекты Azure AD создаются и поддерживаются в базе данных SQL и Azure синапсе без вмешательства человека. Субъекты-службы могут быть администратором Azure AD для логического сервера SQL в составе группы или отдельного пользователя. Приложение может автоматизировать создание объектов Azure AD в базе данных SQL и Azure синапсе при выполнении от имени системного администратора и не требует каких-либо дополнительных прав SQL. Это позволяет полностью автоматизировать создание пользователей базы данных. Эта функция также поддерживается для управляемого системой удостоверения и управляемого удостоверения, назначенного пользователем. Дополнительные сведения см. в статье [что такое управляемые удостоверения для ресурсов Azure?](../../active-directory/managed-identities-azure-resources/overview.md)

## <a name="enable-service-principals-to-create-azure-ad-users"></a>Включение субъектов-служб для создания пользователей Azure AD

Чтобы включить создание объектов Azure AD в базе данных SQL и Azure синапсе от имени приложения Azure AD, необходимы следующие параметры:

1. Назначьте удостоверение сервера. Назначенное удостоверение сервера представляет Управляемое удостоверение службы (MSI). В настоящее время удостоверение сервера для Azure SQL не поддерживает управляемое пользователем удостоверение (УМИ).
    - Для нового логического сервера SQL Azure выполните следующую команду PowerShell:
    
    ```powershell
    New-AzSqlServer -ResourceGroupName <resource group> -Location <Location name> -ServerName <Server name> -ServerVersion "12.0" -SqlAdministratorCredentials (Get-Credential) -AssignIdentity
    ```

    Дополнительные сведения см. в описании команды [New-азсклсервер](/powershell/module/az.sql/new-azsqlserver) .

    - Для существующих логических серверов Azure SQL выполните следующую команду:
    
    ```powershell
    Set-AzSqlServer -ResourceGroupName <resource group> -ServerName <Server name> -AssignIdentity
    ```

    Дополнительные сведения см. в описании команды [Set-AzSqlServer](/powershell/module/az.sql/set-azsqlserver).

    - Чтобы проверить, назначено ли удостоверение сервера серверу, выполните команду Get-AzSqlServer.

    > [!NOTE]
    > Удостоверение сервера можно также назначить с помощью команд интерфейса командной строки. Дополнительные сведения см. в разделе [AZ SQL Server Create](/cli/azure/sql/server#az-sql-server-create) и [AZ SQL Server Update](/cli/azure/sql/server#az-sql-server-update).

2. Предоставьте службе " [**читатели каталога**](../../active-directory/roles/permissions-reference.md#directory-readers) Azure AD" разрешение на доступ к удостоверению сервера, созданному или назначенному серверу.
    - Чтобы предоставить это разрешение, следуйте описанию, используемому для SQL Управляемый экземпляр, которое доступно в следующей статье: [предоставление администратора Azure AD (SQL управляемый экземпляр)](authentication-aad-configure.md?tabs=azure-powershell#provision-azure-ad-admin-sql-managed-instance)
    - Пользователь Azure AD, предоставляющий это разрешение, должен входить в роль администратора **глобального администратора** Azure AD или **привилегированных ролей** .

> [!IMPORTANT]
> Шаги 1 и 2 должны выполняться в указанном порядке. Сначала создайте или назначьте удостоверение сервера, а затем предоставьте разрешение на [**чтение каталога**](../../active-directory/roles/permissions-reference.md#directory-readers) . Пропуск одного из этих шагов или и того и другого приведет к ошибке выполнения во время создания объекта Azure AD в Azure SQL от имени приложения Azure AD.
>
> Если вы используете субъект-службу для задания или настройки администратора Azure AD, приложение должно также иметь разрешение " [каталог. чтение. все](/graph/permissions-reference#application-permissions-18) приложения API" в Azure AD. Дополнительные сведения о [разрешениях, необходимых для настройки администратора Azure AD](authentication-aad-service-principal-tutorial.md#permissions-required-to-set-or-unset-the-azure-ad-admin), и пошаговые инструкции по созданию пользователя Azure AD от имени приложения Azure AD см. в разделе [учебник. Создание пользователей Azure AD с помощью приложений Azure AD](authentication-aad-service-principal-tutorial.md).
>
> В **общедоступной предварительной версии** роль " **читатели каталога** " можно назначить группе в Azure AD. Владельцы групп могут добавить управляемое удостоверение в качестве члена этой группы, что позволит пользователям без прав администратора **глобального администратора** или **привилегированных ролей** предоставить роль **читателей каталога** . Дополнительные сведения об этой функции см. в статье [Роль "Читатели каталогов" в Azure Active Directory для Azure SQL](authentication-aad-directory-readers-role.md).

## <a name="troubleshooting-and-limitations-for-public-preview"></a>Устранение неполадок и ограничения для общедоступной предварительной версии

- При создании объектов Azure AD в Azure SQL от имени приложения Azure AD без включения удостоверения сервера и предоставления разрешений на **чтение каталога** операция завершится со следующими возможными ошибками. Ниже приведен пример ошибки для выполнения команды PowerShell для создания пользователя базы данных SQL `myapp` в статье [учебник. Создание пользователей Azure AD с помощью приложений Azure AD](authentication-aad-service-principal-tutorial.md).
    - `Exception calling "ExecuteNonQuery" with "0" argument(s): "'myapp' is not a valid login or you do not have permission. Cannot find the user 'myapp', because it does not exist, or you do not have permission."`
    - `Exception calling "ExecuteNonQuery" with "0" argument(s): "Principal 'myapp' could not be resolved.`
    - `User or server identity does not have permission to read from Azure Active Directory.`
      - В описанной выше ошибке выполните действия, чтобы [назначить удостоверение логическому серверу SQL Azure](authentication-aad-service-principal-tutorial.md#assign-an-identity-to-the-azure-sql-logical-server) и [назначить ему разрешение на чтение каталога для удостоверения логического сервера SQL](authentication-aad-service-principal-tutorial.md#assign-directory-readers-permission-to-the-sql-logical-server-identity).
    > [!NOTE]
    > Приведенные выше сообщения об ошибках будут изменены до общедоступной версии функции, чтобы ясно определить требования недостающей настройки для поддержки приложений Azure AD.
- Настройка приложения Azure AD в качестве администратора Azure AD для Управляемый экземпляр SQL поддерживается только с помощью команды CLI, а команда PowerShell — с помощью команды [AZ. SQL 2.9.0](https://www.powershellgallery.com/packages/Az.Sql/2.9.0) или более поздней версии. Дополнительные сведения см. в разделе команды [AZ SQL MI AD-Admin Create](/cli/azure/sql/mi/ad-admin#az-sql-mi-ad-admin-create) и [Set-азсклинстанцеактиведиректорядминистратор](/powershell/module/az.sql/set-azsqlinstanceactivedirectoryadministrator) . 
    - Если вы хотите использовать портал Azure для SQL Управляемый экземпляр, чтобы задать администратора Azure AD, можно решить эту проблему, создав группу Azure AD. Затем добавьте субъект-службу (приложение Azure AD) в эту группу и задайте эту группу в качестве администратора Azure AD для Управляемый экземпляр SQL.
    - Настройка субъекта-службы (приложения Azure AD) в качестве администратора Azure AD для базы данных SQL и Azure синапсе поддерживается с помощью команд портал Azure, [PowerShell](authentication-aad-configure.md?tabs=azure-powershell#powershell-for-sql-database-and-azure-synapse)и [CLI](authentication-aad-configure.md?tabs=azure-cli#powershell-for-sql-database-and-azure-synapse) .
- Использование приложения Azure AD с субъектом-службой из другого клиента Azure AD завершится ошибкой при доступе к базе данных SQL или Управляемый экземпляр SQL, созданной в другом клиенте. Субъект-служба, назначенный этому приложению, должен принадлежать тому же клиенту, что и логический сервер SQL или Управляемый экземпляр.
- Для настройки отдельного приложения Azure AD в качестве администратора Azure AD для SQL Azure с помощью PowerShell необходим модуль [Az.Sql 2.9.0](https://www.powershellgallery.com/packages/Az.Sql/2.9.0) или более поздней версии. Убедитесь, что используется последняя версия модуля.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Руководство по созданию пользователей Azure AD с помощью приложений Azure AD](authentication-aad-service-principal-tutorial.md)
