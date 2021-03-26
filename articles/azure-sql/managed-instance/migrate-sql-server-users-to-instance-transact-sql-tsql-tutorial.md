---
title: Перенос пользователей и групп Windows, настроенных для SQL Server, в управляемый экземпляр SQL Azure с помощью T-SQL
description: Сведения о том, как перенести пользователей и группы Windows, настроенных для SQL Server, в управляемый экземпляр SQL Azure
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: seo-lt-2019, sqldbrb=1
ms.topic: tutorial
author: GitHubMirek
ms.author: mireks
ms.reviewer: vanto
ms.date: 10/30/2019
ms.openlocfilehash: f2dd34ab7c6ee5be26836e4abb86960605ee44ee
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84708690"
---
# <a name="tutorial-migrate-windows-users-and-groups-in-a-sql-server-instance-to-azure-sql-managed-instance-using-t-sql-ddl-syntax"></a>Руководство по переносу пользователей и групп Windows, настроенных для экземпляра SQL Server, в управляемый экземпляр SQL Azure с использованием синтаксиса T-SQL DDL
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

> [!NOTE]
> Поддерживается **общедоступная предварительная версия** синтаксиса, который используется в рамках этой статьи для переноса пользователей и групп в управляемый экземпляр SQL Azure.

В этой статье описан перенос пользователей и групп Windows, настроенных для SQL Server, в управляемый экземпляр SQL Azure с использованием синтаксиса T-SQL.

В этом руководстве описано следующее:

> [!div class="checklist"]
>
> - создание имен для входа в SQL Server;
> - создание тестовой базы данных для миграции;
> - создание имен для входа, пользователей и ролей;
> - резервное копирование и восстановление базы данных в управляемом экземпляре SQL Azure;
> - перенос пользователей в управляемый экземпляр вручную с использованием синтаксиса ALTER USER;
> - тестирование проверки подлинности для новых сопоставленных пользователей.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством вам потребуется следующее:

- федеративная связь домена Windows с Azure Active Directory (AAD);
- доступ к Active Directory для создания пользователей и групп;
- SQL Server в локальной среде;
- Существующий управляемый экземпляр SQL. См. [Краткое руководство. Создание управляемого экземпляра SQL](instance-create-quickstart.md).
  - Для создания имен входа Azure Active Directory необходимо использовать `sysadmin` в управляемом экземпляре SQL Azure.
- [Создание учетной записи администратора Azure Active Directory для управляемого экземпляра SQL](../database/authentication-aad-configure.md#provision-azure-ad-admin-sql-managed-instance).
- Вы можете подключиться к управляемому экземпляру SQL Azure из локальной сети. Дополнительные сведения см. в следующих статьях:
  - [Connect your application to Azure SQL Managed Instance](connect-application-instance.md) (Подключение приложения к управляемому экземпляру SQL Azure)
  - [Краткое руководство. Configure a point-to-site connection to an Azure SQL Managed Instance from on-premises](point-to-site-p2s-configure.md) (Настройка подключения "точка — сеть" к управляемому экземпляру SQL Azure)
  - [Configure public endpoint in Azure SQL Managed Instance](public-endpoint-configure.md) (Настройка общедоступной конечной точки в управляемом экземпляре SQL Azure)

## <a name="t-sql-ddl-syntax"></a>Синтаксис T-SQL DDL

Ниже описан синтаксис T-SQL DDL, который используется для переноса пользователей и групп Windows, настроенных для экземпляра SQL Server, в управляемый экземпляр SQL Azure с проверкой подлинности Azure Active Directory.

```sql
-- For individual Windows users with logins
ALTER USER [domainName\userName] WITH LOGIN = [loginName@domainName.com];

--For individual groups with logins
ALTER USER [domainName\groupName] WITH LOGIN=[groupName]
```

## <a name="arguments"></a>Аргументы

_domainName_</br>
Указывает имя домена для пользователя.

_userName_</br>
Указывает имя пользователя, найденного в базе данных.

_= loginName\@domainName.com_</br>
Сопоставляет пользователя с именем для входа в AAD

_groupName_</br>
Указывает имя группы, найденной в базе данных.

## <a name="part-1-create-logins-in-sql-server-for-windows-users-and-groups"></a>Часть 1. Создание имен входа в SQL Server для пользователей и групп Windows

> [!IMPORTANT]
> Описанный ниже синтаксис позволяет создать в SQL Server имена пользователей и групп. Перед выполнением описанного ниже синтаксиса необходимо убедиться, что пользователь и группа есть в Active Directory (AD). </br> </br>
> Пользователи: testUser1, testGroupUser </br>
> Группа: migration, в которую должен входить пользователь testGroupUser в AD

В приведенном ниже примере в SQL Server создается имя входа для учетной записи _testUser1_ в домене _aadsqlmi_.

```sql
-- Sign into SQL Server as a sysadmin or a user that can create logins and databases

use master;  
go

-- Create Windows login
create login [aadsqlmi\testUser1] from windows;
go;

/** Create a Windows group login which contains one user [aadsqlmi\testGroupUser].
testGroupUser will need to be added to the migration group in Active Directory
**/
create login [aadsqlmi\migration] from windows;
go;


-- Check logins were created
select * from sys.server_principals;
go;
```

Создайте базу данных для этого теста.

```sql
-- Create a database called [migration]
create database migration
go
```

## <a name="part-2-create-windows-users-and-groups-then-add-roles-and-permissions"></a>Часть 2. Создание пользователей и групп Windows, добавление ролей и разрешений

Используйте описанный ниже синтаксис, чтобы создать тестового пользователя.

```sql
use migration;  
go

-- Create Windows user [aadsqlmi\testUser1] with login
create user [aadsqlmi\testUser1] from login [aadsqlmi\testUser1];
go
```

Проверьте разрешения пользователя:

```sql
-- Check the user in the Metadata
select * from sys.database_principals;
go

-- Display the permissions – should only have CONNECT permissions
select user_name(grantee_principal_id), * from sys.database_permissions;
go
```

Создайте роль и назначьте тестового пользователя для этой роли:

```sql
-- Create a role with some permissions and assign the user to the role
create role UserMigrationRole;
go

grant CONNECT, SELECT, View DATABASE STATE, VIEW DEFINITION to UserMigrationRole;
go

alter role UserMigrationRole add member [aadsqlmi\testUser1];
go
```

По следующему запросу отображаются имена пользователей, которым назначена указанная роль:

```sql
-- Display user name assigned to a specific role
SELECT DP1.name AS DatabaseRoleName,
   isnull (DP2.name, 'No members') AS DatabaseUserName
 FROM sys.database_role_members AS DRM
 RIGHT OUTER JOIN sys.database_principals AS DP1
   ON DRM.role_principal_id = DP1.principal_id
 LEFT OUTER JOIN sys.database_principals AS DP2
   ON DRM.member_principal_id = DP2.principal_id
WHERE DP1.type = 'R'
ORDER BY DP1.name;
```

Используйте описанный ниже синтаксис, чтобы создать группу. Затем добавьте группу в роль `db_owner`.

```sql
-- Create Windows group
create user [aadsqlmi\migration] from login [aadsqlmi\migration];
go

-- ADD 'db_owner' role to this group
sp_addrolemember 'db_owner', 'aadsqlmi\migration';
go

--Check the db_owner role for 'aadsqlmi\migration' group
select is_rolemember('db_owner', 'aadsqlmi\migration')
go
-- Output  ( 1 means YES)
```

Создайте тестовую таблицу и добавьте в нее данные с помощью следующего синтаксиса:

```sql
-- Create a table and add data
create table test ( a int, b int);
go

insert into test values (1,10)
go

-- Check the table values
select * from test;
go
```

## <a name="part-3-backup-and-restore-the-individual-user-database-to-sql-managed-instance"></a>Часть 3. Резервное копирование и восстановление базы данных пользователей в управляемом экземпляре SQL

Создайте резервную копию базы данных, для которой выполняется миграция, используя инструкции из [этой статьи](/sql/relational-databases/databases/copy-databases-with-backup-and-restore) или следующий синтаксис:

```sql
use master;
go
backup database migration to disk = 'C:\Migration\migration.bak';
go
```

Выполните инструкции из статьи [Краткое руководство. Restore a database to an Azure SQL Managed Instance with SSMS](restore-sample-database-quickstart.md) (Восстановление базы данных в управляемый экземпляр SQL Azure с помощью SSMS).

## <a name="part-4-migrate-users-to-sql-managed-instance"></a>Часть 4. Перенос пользователей в управляемый экземпляр SQL Azure

Выполните команду ALTER USER, чтобы завершить миграцию в управляемом экземпляре SQL Azure.

1. Подключитесь к управляемому экземпляру с учетной записью администратора AAD. Затем создайте в управляемом экземпляре имя для входа в AAD, используя следующий синтаксис. Дополнительные сведения см. в статье [Учебник. Security in Azure SQL Managed Instance using Azure AD server principals (logins)](aad-security-configure-tutorial.md) (Обеспечение безопасности управляемого экземпляра в Базе данных SQL Azure с помощью субъектов сервера (имен для входа) Azure Active Directory).

    ```sql
    use master
    go

    -- Create login for AAD user [testUser1@aadsqlmi.net]
    create login [testUser1@aadsqlmi.net] from external provider
    go

    -- Create login for the Azure AD group [migration]. This group contains one user [testGroupUser@aadsqlmi.net]
    create login [migration] from external provider
    go

    --Check the two new logins
    select * from sys.server_principals
    go
    ```

1. Убедитесь, что база данных, таблица и субъекты успешно перенесены.

    ```sql
    -- Switch to the database migration that is already restored for MI
    use migration;
    go

    --Check if the restored table test exist and contain a row
    select * from test;
    go

    -- Check that the SQL on-premises Windows user/group exists  
    select * from sys.database_principals;
    go
    -- the old user aadsqlmi\testUser1 should be there
    -- the old group aadsqlmi\migration should be there
    ```

1. Используйте синтаксис ALTER USER, чтобы связать локального пользователя с именем для входа в Azure AD.

    ```sql
    /** Execute the ALTER USER command to alter the Windows user [aadsqlmi\testUser1]
    to map to the Azure AD user testUser1@aadsqlmi.net
    **/
    alter user [aadsqlmi\testUser1] with login = [testUser1@aadsqlmi.net];
    go

    -- Check the principal
    select * from sys.database_principals;
    go
    -- New user testUser1@aadsqlmi.net should be there instead
    --Check new user permissions  - should only have CONNECT permissions
    select user_name(grantee_principal_id), * from sys.database_permissions;
    go

    -- Check a specific role
    -- Display Db user name assigned to a specific role
    SELECT DP1.name AS DatabaseRoleName,
    isnull (DP2.name, 'No members') AS DatabaseUserName
    FROM sys.database_role_members AS DRM
    RIGHT OUTER JOIN sys.database_principals AS DP1
    ON DRM.role_principal_id = DP1.principal_id
    LEFT OUTER JOIN sys.database_principals AS DP2
    ON DRM.member_principal_id = DP2.principal_id
    WHERE DP1.type = 'R'
    ORDER BY DP1.name;
    ```

1. Используйте синтаксис ALTER USER, чтобы связать локальную группу с именем для входа в Azure AD.

    ```sql
    /** Execute ALTER USER command to alter the Windows group [aadsqlmi\migration]
    to the Azure AD group login [migration]
    **/
    alter user [aadsqlmi\migration] with login = [migration];
    -- old group migration is changed to Azure AD migration group
    go

    -- Check the principal
    select * from sys.database_principals;
    go

    --Check the group permission - should only have CONNECT permissions
    select user_name(grantee_principal_id), * from sys.database_permissions;
    go

    --Check the db_owner role for 'aadsqlmi\migration' user
    select is_rolemember('db_owner', 'migration')
    go
    -- Output 1 means 'YES'
    ```

## <a name="part-5-testing-azure-ad-user-or-group-authentication"></a>Часть 5. Тестирование проверки подлинности для пользователя или группы Azure AD

Проверьте аутентификацию в управляемом экземпляре SQL Azure для пользователя, которого вы сопоставили с именем для входа AAD с помощью синтаксиса ALTER USER.

1. Войдите на федеративную виртуальную машину, используя подписку управляемого экземпляра SQL Azure в качестве `aadsqlmi\testUser1`.
1. Из SQL Server Management Studio (SSMS) войдите в управляемый экземпляр SQL Azure, используя **встроенную в доменные службы Active Directory** проверку подлинности и подключитесь к базе данных `migration`.
    1. Кроме того, вы можете войти с помощью учетных данных testUser1@aadsqlmi.net, используя параметр SSMS **Active Directory — универсальная с поддержкой MFA**. Но в этом случае вы не сможете использовать механизм единого входа и вам придется ввести пароль. Для входа в управляемый экземпляр вам не потребуется использовать федеративную виртуальную машину.
1. Являясь членом роли **SELECT**, вы можете получать данные из таблицы `test`.

    ```sql
    Select * from test  --  and see one row (1,10)
    ```

Проверьте аутентификацию в управляемом экземпляре для члена группы Windows `migration`. Перед началом миграции пользователя `aadsqlmi\testGroupUser` нужно добавить в группу `migration`.

1. Войдите на федеративную виртуальную машину, используя подписку управляемого экземпляра SQL Azure в качестве `aadsqlmi\testGroupUser`.
1. Используя SSMS со **встроенной в доменные службы Active Directory** проверкой подлинности, подключитесь к серверу управляемого экземпляра SQL Azure и к базе данных `migration`.
    1. Кроме того, вы можете войти с помощью учетных данных testGroupUser@aadsqlmi.net, используя параметр SSMS **Active Directory — универсальная с поддержкой MFA**. Но в этом случае вы не сможете использовать механизм единого входа и вам придется ввести пароль. Для входа в управляемый экземпляр SQL Azure вам не потребуется использовать федеративную виртуальную машину.
1. Являясь членом роли `db_owner`, вы можете создать новую таблицу.

    ```sql
    -- Create table named 'new' with a default schema
    Create table dbo.new ( a int, b int)
    ```

> [!NOTE]
> В базе данных SQL Azure есть известная проблема с проектированием, из-за которой инструкция CREATE TABLE завершается ошибкой, если ее выполняет член группы: </br> </br>
> `Msg 2760, Level 16, State 1, Line 4
The specified schema name "testGroupUser@aadsqlmi.net" either does not exist or you do not have permission to use it.` </br> </br>
> Сейчас можно использовать обходное решение, создавая таблицу с существующей схемой, например <dbo.new> для примера выше.

## <a name="next-steps"></a>Дальнейшие действия

[Руководство. Миграция SQL Server в управляемый экземпляр базы данных SQL Azure в автономном режиме с помощью DMS](../../dms/tutorial-sql-server-to-managed-instance.md?toc=/azure/sql-database/toc.json)
