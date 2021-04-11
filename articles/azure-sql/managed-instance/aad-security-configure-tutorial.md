---
title: Защита Управляемого экземпляра SQL с помощью субъектов сервера (имен для входа) Azure AD
description: Узнайте о методах и функциях, применяемых для защиты Управляемого экземпляра SQL Azure, и использовании субъектов сервера (имен для входа) Azure AD
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: sqldbrb=1
ms.topic: tutorial
author: GitHubMirek
ms.author: mireks
ms.reviewer: vanto
ms.date: 11/06/2019
ms.openlocfilehash: 50544507f9d83c216bb6c18e004c5ce7ad1ca346
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105639857"
---
# <a name="tutorial-security-in-azure-sql-managed-instance-using-azure-ad-server-principals-logins"></a>Руководство по обеспечению безопасности Управляемого экземпляра SQL Azure с помощью субъектов сервера (имен для входа) Azure AD
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Управляемый экземпляр SQL Azure обеспечивает практически все функции безопасности, имеющиеся в последней версии ядра СУБД SQL Server (Enterprise Edition):

- ограничение доступа в изолированной среде;
- использование механизмов аутентификации, которые требуют учетных данных: аутентификация Azure Active Directory (Azure AD) и SQL;
- использовать авторизацию на основе разрешений и членства с учетом ролей.
- Включение функций безопасности

В этом руководстве описано следующее:

> [!div class="checklist"]
>
> - создание субъекта сервера (имени для входа) Azure AD для управляемого экземпляра;
> - предоставление разрешений субъектам сервера (именам для входа) Azure AD в управляемом экземпляре;
> - создание пользователей Azure AD на основе субъектов сервера (имен для входа) Azure AD;
> - назначение разрешений пользователям Azure AD и управление безопасностью базы данных;
> - использование олицетворения с пользователями Azure AD;
> - использование межбазовых запросов с пользователями Azure AD;
> - узнаете о функциях безопасности, таких как защита от угроз, аудит, маскирование данных и шифрование.

Дополнительные сведения см. в [обзорной статье об Управляемом экземпляре SQL Azure](sql-managed-instance-paas-overview.md). 

## <a name="prerequisites"></a>Предварительные требования

Для работы с руководством требуется наличие следующих компонентов:

- [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS).
- Управляемый экземпляр.
  - Следуйте инструкциям из этой статьи: [Краткое руководство. Создание управляемого экземпляра](instance-create-quickstart.md)
- Возможность доступа к управляемому экземпляру и [подготовленный пользователь с правами администратора Azure AD для управляемого экземпляра](../database/authentication-aad-configure.md#provision-azure-ad-admin-sql-managed-instance). Дополнительные сведения см. на следующих ресурсах:
  - [Подключение приложения к Управляемому экземпляру Базы данных SQL](connect-application-instance.md).
  - [Архитектура подключения к Управляемому экземпляру SQL](connectivity-architecture-overview.md)
  - [Настройка и администрирование аутентификации Azure Active Directory с помощью SQL](../database/authentication-aad-configure.md)

## <a name="limit-access"></a>Ограничение доступа 

Доступ к управляемому экземпляру возможен по частному IP-адресу. Так же как и в изолированной среде SQL Server, приложениям или пользователям потребуется доступ к сети Управляемого экземпляра SQL, прежде чем можно будет установить подключение. Дополнительные сведения см. в статье [Подключение приложения к Управляемому экземпляру SQL](connect-application-instance.md).

Кроме того, можно настроить конечную точку службы в управляемом экземпляре, которая обеспечит общедоступные подключения таким же образом, как и База данных SQL Azure.
Дополнительные сведения см. в статье о [настройке общедоступной конечной точки в Управляемом экземпляре SQL Azure](public-endpoint-configure.md).

> [!NOTE]
> Даже при включенных конечных точках службы [правила брандмауэра Базы данных SQL Azure](../database/firewall-configure.md) не применяются. В Управляемых экземплярах SQL Azure есть [встроенный брандмауэр](management-endpoint-verify-built-in-firewall.md), управляющий подключениями.

## <a name="create-an-azure-ad-server-principal-login-using-ssms"></a>Создание субъекта сервера (имени для входа) Azure AD с помощью SSMS

Первый субъект сервера Azure AD (имя входа) может быть создан стандартной учетной записью администратора SQL (не Azure AD), которая является `sysadmin`, или администратором Azure AD для управляемого экземпляра, созданного в процессе подготовки. Дополнительные сведения см. в разделе [Подготовка администратора Azure Active Directory для Управляемого экземпляра SQL](../database/authentication-aad-configure.md#provision-azure-ad-admin-sql-managed-instance).

Примеры подключения к Управляемому экземпляру SQL см. в следующих статьях:

- [Краткое руководство. Настройка виртуальной машины Azure для подключения к Управляемому экземпляру SQL](connect-vm-instance-configure.md)
- [Краткое руководство. Настройка соединения "точка — сеть" с Управляемым экземпляром SQL](point-to-site-p2s-configure.md)

1. Войдите в управляемый экземпляр с помощью стандартной учетной записи SQL для входа (не Azure AD): `sysadmin` или учетной записи администратора Azure AD для Управляемого экземпляра SQL, используя [SQL Server Management Studio](point-to-site-p2s-configure.md#connect-with-ssms).

2. В **обозревателе объектов** щелкните сервер правой кнопкой мыши и выберите **Создать запрос**.

3. В окне запроса создайте имя для входа для локальной учетной записи Azure AD, используя следующий синтаксис:

    ```sql
    USE master
    GO
    CREATE LOGIN login_name FROM EXTERNAL PROVIDER
    GO
    ```

    В этом примере создается имя для входа для учетной записи nativeuser@aadsqlmi.onmicrosoft.com.

    ```sql
    USE master
    GO
    CREATE LOGIN [nativeuser@aadsqlmi.onmicrosoft.com] FROM EXTERNAL PROVIDER
    GO
    ```

4. На панели инструментов выберите **Выполнить**, чтобы создать имя для входа.

5. Проверьте добавленное имя для входа, выполнив следующую команду T-SQL:

    ```sql
    SELECT *  
    FROM sys.server_principals;  
    GO
    ```

    ![Снимок экрана: вкладка "Результаты" в обозревателе объектов SSMS со значениями в полях name, principal_id, sid, type и type_desc для добавленного имени для входа](./media/aad-security-configure-tutorial/native-login.png)

Дополнительные сведения можно найти в разделе [CREATE LOGIN (Transact-SQL)](/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current&preserve-view=true).

## <a name="grant-permissions-to-create-logins"></a>Предоставление прав для создания имен для входа

Чтобы создать другие субъекты сервера (имена для входа) Azure AD, субъекту (SQL или Azure AD) необходимо предоставить роли или разрешения SQL Server.

### <a name="sql-authentication"></a>Проверка подлинности SQL

- Если имя для входа представляет собой субъект SQL, команду create для создания имен для входа для учетной записи Azure AD можно использовать только с именами для входа, которые являются частью роли `sysadmin`.

### <a name="azure-ad-authentication"></a>Аутентификация Azure AD

- Чтобы предоставить только что созданному субъекту сервера (имени для входа) Azure AD возможность создавать другие имена входа для других пользователей, групп или приложений Azure AD, предоставьте имени для входа роль сервера `sysadmin` или `securityadmin`.
- Чтобы создавать другие субъекты сервера (имена для входа) Azure AD, применяемому для этого субъекту сервера (имени для входа) Azure AD необходимо предоставить как минимум разрешение **ALTER ANY LOGIN**.
- По умолчанию только что созданным субъектам сервера (именам для входа) Azure AD в базе данных master предоставляются следующие стандартные разрешения: **CONNECT SQL** и **VIEW ANY DATABASE**.
- В управляемом экземпляре роль сервера `sysadmin` может предоставляться многим субъектам сервера (именам для входа) Azure AD.

Чтобы присвоить имени для входа роль сервера `sysadmin`, сделайте следующее:

1. Войдите в управляемый экземпляр еще раз или используйте имеющееся подключение к администратору Azure AD или субъекту SQL (`sysadmin`).

1. В **обозревателе объектов** щелкните сервер правой кнопкой мыши и выберите **Создать запрос**.

1. Предоставьте субъекту сервера (имени для входа) Azure AD роль сервера `sysadmin`, используя следующий синтаксис T-SQL:

    ```sql
    ALTER SERVER ROLE sysadmin ADD MEMBER login_name
    GO
    ```

    В следующем примере роль сервера `sysadmin` предоставляется имени для входа nativeuser@aadsqlmi.onmicrosoft.com.

    ```sql
    ALTER SERVER ROLE sysadmin ADD MEMBER [nativeuser@aadsqlmi.onmicrosoft.com]
    GO
    ```

## <a name="create-additional-azure-ad-server-principals-logins-using-ssms"></a>Создание дополнительных субъектов сервера (имен для входа) Azure AD в SSMS

После создания субъекта сервера (имени для входа) Azure AD и предоставления ему привилегий `sysadmin` с его помощью можно создавать дополнительные имена для входа, используя предложение **FROM EXTERNAL PROVIDER** с **CREATE LOGIN**.

1. Подключитесь к управляемому экземпляру с помощью субъекта сервера (имени для входа) Azure AD в SQL Server Management Studio. Введите имя узла Управляемого экземпляра SQL. При входе в систему с учетной записью Azure AD есть три варианта проверки подлинности в SSMS:

   - "Active Directory — универсальная с поддержкой MFA";
   - "Active Directory — пароль";
   - "Active Directory — встроенная". </br>

     ![Снимок экрана: диалоговое окно "Подключение к серверу в SSMS" с выделенным пунктом "Active Directory — универсальная с поддержкой MFA" в раскрывающемся списке "Аутентификация"](./media/aad-security-configure-tutorial/ssms-login-prompt.png)

     Дополнительные сведения см. в статье [Универсальная проверка подлинности (поддержка SSMS для многофакторной идентификации)](../database/authentication-mfa-ssms-overview.md).

1. Выберите **Active Directory — универсальная с поддержкой MFA**. Откроется окно входа с Многофакторной идентификацией. Войдите с помощью пароля в Azure AD.

    ![Снимок экрана: окно входа с многофакторной проверкой подлинности с курсором в поле для ввода пароля](./media/aad-security-configure-tutorial/mfa-login-prompt.png)

1. В **обозревателе объектов** SSMS щелкните сервер правой кнопкой мыши и выберите **Создать запрос**.
1. В окне запроса создайте имя для входа для еще одной учетной записи Azure AD, используя следующий синтаксис:

    ```sql
    USE master
    GO
    CREATE LOGIN login_name FROM EXTERNAL PROVIDER
    GO
    ```

    В этом примере создается имя для входа для пользователя Azure AD bob@aadsqlmi.net, чей домен aadsqlmi.net включен в федерацию с доменом aadsqlmi.onmicrosoft.com Azure AD.

    Выполните следующую команду T-SQL. Федеративные учетные записи Azure AD — это Управляемые экземпляры SQL, заменяющие локальных пользователей и имена для входа Windows.

    ```sql
    USE master
    GO
    CREATE LOGIN [bob@aadsqlmi.net] FROM EXTERNAL PROVIDER
    GO
    ```

1. Создайте базу данных в управляемом экземпляре с помощью синтаксиса [CREATE DATABASE](/sql/t-sql/statements/create-database-transact-sql?view=azuresqldb-mi-current&preserve-view=true). Эта база данных будет использоваться для проверки имен для входа пользователей в следующем разделе.
    1. В **обозревателе объектов** щелкните сервер правой кнопкой мыши и выберите **Создать запрос**.
    1. В окне запроса создайте базу данных с именем **MyMITestDB**, используя следующий синтаксис.

        ```sql
        CREATE DATABASE MyMITestDB;
        GO
        ```

1. Создайте имя входа Управляемого экземпляра SQL для группы в Azure AD. Чтобы вы смогли добавить имя для входа в Управляемый экземпляр SQL, в Azure AD уже должна существовать группа. Ознакомьтесь со статьей [Создание простой группы и добавление в нее участников с помощью Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md). Создайте группу _mygroup_ и добавьте в нее участников.

1. Откройте окно нового запроса в SQL Server Management Studio.

    В этом примере предполагается, что в Azure AD существует группа с именем _mygroup_. Выполните следующую команду:

    ```sql
    USE master
    GO
    CREATE LOGIN [mygroup] FROM EXTERNAL PROVIDER
    GO
    ```

1. В целях проверки войдите в управляемый экземпляр с использованием только что созданного имени для входа или группы. Установите новое подключение к управляемому экземпляру и используйте новое имя для входа при проверке подлинности.
1. В **обозревателе объектов** щелкните сервер правой кнопкой мыши и выберите **Создать запрос** для нового подключения.
1. Проверьте разрешения сервера для только что созданного субъекта сервера (имени для входа) Azure AD, выполнив следующую команду:

      ```sql
      SELECT * FROM sys.fn_my_permissions (NULL, 'DATABASE')
      GO
      ```

> [!NOTE]
> Гостевые пользователи Azure AD поддерживаются для имен для входа Управляемых экземпляров SQL только в том случае, если добавляются в качестве группы Azure AD. Гостевой пользователь Azure AD — это учетная запись, приглашенная в экземпляр Azure AD, которой принадлежит управляемый экземпляр, из другого экземпляра Azure AD. Например, joe@contoso.com (учетная запись Azure AD) или steve@outlook.com (учетная запись Microsoft) можно добавить в группу в экземпляре aadsqlmi Azure AD. После того как пользователи добавлены в группу, в базе данных **master** Управляемого экземпляра SQL для группы можно создать имя для входа с использованием синтаксиса **CREATE LOGIN**. Гостевые пользователи, которые являются участниками этой группы, могут подключиться к управляемому экземпляру с помощью своих текущих имен для входа (например, joe@contoso.com или steve@outlook.com).

## <a name="create-an-azure-ad-user-from-the-azure-ad-server-principal-login"></a>Создание пользователя Azure AD на основе субъекта сервера (имени для входа) Azure AD

Авторизация для отдельных баз данных в Управляемом экземпляре SQL происходит так же, как и в базах данных в SQL Server. Пользователя можно создать, используя имеющееся имя для входа в базе данных, и ему можно предоставить разрешения для этой базы данных или роль базы данных.

Теперь, когда мы создали базу данных с именем **MyMITestDB** и имя для входа, у которого есть только разрешения по умолчанию, следующим шагом является создание пользователя на основе этого имени для входа. В данный момент имя для входа позволяет подключиться к управляемому экземпляру и видеть все базы данных, но не позволяет взаимодействовать с ними. При входе с помощью учетной записи Azure AD с разрешениями по умолчанию и попытке развернуть только что созданную базу данных отобразится следующая ошибка:

![Снимок экрана: сообщение об ошибке в обозревателе объектов SSMS, информирующее о том, что база данных MyMITestDB недоступна (The database MyMITestDB is not accessible. (ObjectExplorer))](./media/aad-security-configure-tutorial/ssms-db-not-accessible.png)

Дополнительные сведения о предоставлении разрешений к базе данных см. в статье [Приступая к работе с разрешениями Database Engine](/sql/relational-databases/security/authentication-access/getting-started-with-database-engine-permissions).

### <a name="create-an-azure-ad-user-and-create-a-sample-table"></a>Создание пользователя Azure AD и примера таблицы

1. Войдите в управляемый экземпляр с помощью учетной записи `sysadmin` в SQL Server Management Studio.
1. В **обозревателе объектов** щелкните сервер правой кнопкой мыши и выберите **Создать запрос**.
1. В окне запроса создайте пользователя Azure AD на основе субъекта сервера (имени для входа) Azure AD, используя следующий синтаксис:

    ```sql
    USE <Database Name> -- provide your database name
    GO
    CREATE USER user_name FROM LOGIN login_name
    GO
    ```

    В следующем примере создается пользователь bob@aadsqlmi.net на основе имени для входа bob@aadsqlmi.net:.

    ```sql
    USE MyMITestDB
    GO
    CREATE USER [bob@aadsqlmi.net] FROM LOGIN [bob@aadsqlmi.net]
    GO
    ```

1. Кроме того, поддерживается создание пользователя Azure AD на основе субъекта сервера (имени для входа) Azure AD, которое является группой.

    В следующем примере создается имя для входа для группы Azure AD _mygroup_, имеющейся в экземпляре Azure AD.

    ```sql
    USE MyMITestDB
    GO
    CREATE USER [mygroup] FROM LOGIN [mygroup]
    GO
    ```

    Все пользователи, принадлежащие к *mygroup*, могут получить доступ к базе данных **MyMITestDB**.

    > [!IMPORTANT]
    > При создании **пользователя** на основе субъекта сервера (имени для входа) Azure AD необходимо указать user_name, совпадающее с login_name **имени для входа**.

    Дополнительные сведения можно найти в статье [CREATE USER (Transact-SQL)](/sql/t-sql/statements/create-user-transact-sql?view=azuresqldb-mi-current&preserve-view=true).

1. В окне нового запроса создайте тестовую таблицу, используя следующую команду T-SQL:

    ```sql
    USE MyMITestDB
    GO
    CREATE TABLE TestTable
    (
    AccountNum varchar(10),
    City varchar(255),
    Name varchar(255),
    State varchar(2)
    );
    ```

1. Создайте подключение в SSMS под именем созданного пользователя. При этом не будет отображаться таблица **TestTable**, созданная `sysadmin` ранее. Необходимо предоставить пользователю разрешения на чтение данных из базы данных.

1. Вы можете проверить текущие разрешения пользователя, выполнив следующую команду:

    ```sql
    SELECT * FROM sys.fn_my_permissions('MyMITestDB','DATABASE')
    GO
    ```

### <a name="add-users-to-database-level-roles"></a>Назначение пользователей ролям уровня базы данных

Чтобы пользователь мог просматривать данные в базе данных, ему можно предоставить [роли уровня базы данных](/sql/relational-databases/security/authentication-access/database-level-roles).

1. Войдите в управляемый экземпляр с помощью учетной записи `sysadmin` в SQL Server Management Studio.

1. В **обозревателе объектов** щелкните сервер правой кнопкой мыши и выберите **Создать запрос**.

1. Предоставьте пользователю Azure AD роль базы данных `db_datareader`, используя следующий синтаксис T-SQL:

    ```sql
    Use <Database Name> -- provide your database name
    ALTER ROLE db_datareader ADD MEMBER user_name
    GO
    ```

    В следующем примере пользователю bob@aadsqlmi.net и группе _mygroup_ предоставляются разрешения `db_datareader` на базу данных **MyMITestDB**:

    ```sql
    USE MyMITestDB
    GO
    ALTER ROLE db_datareader ADD MEMBER [bob@aadsqlmi.net]
    GO
    ALTER ROLE db_datareader ADD MEMBER [mygroup]
    GO
    ```

1. Проверьте, существует ли пользователь Azure AD, созданный в базе данных, выполнив следующую команду:

    ```sql
    SELECT * FROM sys.database_principals
    GO
    ```

1. Установите новое подключение к управляемому экземпляру с помощью пользователя, которому назначена роль `db_datareader`.
1. Разверните базу данных в **обозревателе объектов**, чтобы просмотреть таблицу.

    ![Снимок экрана: обозреватель объектов SSMS со структурой папок в каталоге "Таблицы" в MyMITestDB и выделенной папкой dbo.TestTable](./media/aad-security-configure-tutorial/ssms-test-table.png)

1. Откройте окно нового запроса и выполните следующую инструкцию SELECT:

    ```sql
    SELECT *
    FROM TestTable
    ```

    Вы можете просматривать данные таблицы? Должны отобразиться возвращаемые столбцы.

    ![Снимок экрана: вкладка "Результаты" в обозревателе объектов SSMS с заголовками столбцов AccountNum, City, Name и State таблицы](./media/aad-security-configure-tutorial/ssms-test-table-query.png)

## <a name="impersonate-azure-ad-server-level-principals-logins"></a>Олицетворение субъектов серверного уровня Azure AD (имена для входа)

Управляемый экземпляр SQL поддерживает олицетворение субъектов серверного уровня Azure AD (имена для входа).

### <a name="test-impersonation"></a>Проверка олицетворения

1. Войдите в управляемый экземпляр с помощью учетной записи `sysadmin` в SQL Server Management Studio.

1. В **обозревателе объектов** щелкните сервер правой кнопкой мыши и выберите **Создать запрос**.

1. В окне запроса создайте новую хранимую процедуру, используя следующую команду:

    ```sql
    USE MyMITestDB
    GO  
    CREATE PROCEDURE dbo.usp_Demo  
    WITH EXECUTE AS 'bob@aadsqlmi.net'  
    AS  
    SELECT user_name();  
    GO
    ```

1. С помощью следующей команды убедитесь, что пользователь, которого вы олицетворяете при выполнении хранимой процедуры, — это **bob\@aadsqlmi.net**.

    ```sql
    Exec dbo.usp_Demo
    ```

1. Проверьте олицетворение с помощью инструкции EXECUTE AS LOGIN:

    ```sql
    EXECUTE AS LOGIN = 'bob@aadsqlmi.net'
    GO
    SELECT SUSER_SNAME()
    REVERT
    GO
    ```

> [!NOTE]
> Только субъекты серверного уровня SQL (имена для входа), которые являются частью роли `sysadmin`, могут выполнять следующие операции, предназначенные для субъектов Azure AD:
>
> - EXECUTE AS USER;
> - EXECUTE AS LOGIN.

## <a name="use-cross-database-queries"></a>Использование межбазовых запросов

Межбазовые запросы поддерживаются для учетных записей Azure AD с субъектами сервера (именами для входа) Azure AD. Чтобы протестировать межбазовый запрос с помощью группы Azure AD, необходимо создать еще одну базу данных и таблицу. Если они уже существуют, их создание можно пропустить.

1. Войдите в управляемый экземпляр с помощью учетной записи `sysadmin` в SQL Server Management Studio.
1. В **обозревателе объектов** щелкните сервер правой кнопкой мыши и выберите **Создать запрос**.
1. В окне запроса создайте базу данных с именем **MyMITestDB2** и таблицу с именем **TestTable2**, используя следующую команду.

    ```sql
    CREATE DATABASE MyMITestDB2;
    GO
    USE MyMITestDB2
    GO
    CREATE TABLE TestTable2
    (
    EmpId varchar(10),
    FirstName varchar(255),
    LastName varchar(255),
    Status varchar(10)
    );
    ```

1. В окне нового запроса выполните следующую команду, чтобы создать пользователя _mygroup_ в новой базе данных **MyMITestDB2**, и предоставьте _mygroup_ разрешения SELECT на эту базу данных:

    ```sql
    USE MyMITestDB2
    GO
    CREATE USER [mygroup] FROM LOGIN [mygroup]
    GO
    GRANT SELECT TO [mygroup]
    GO
    ```

1. Войдите в управляемый экземпляр, используя SQL Server Management Studio в качестве участника группы Azure AD _mygroup_. Откройте окно нового запроса и выполните межбазовую инструкцию SELECT:

    ```sql
    USE MyMITestDB
    SELECT * FROM MyMITestDB2..TestTable2
    GO
    ```

    Вы должны увидеть результаты таблицы **TestTable2**.

## <a name="additional-supported-scenarios"></a>Дополнительные поддерживаемые сценарии

- Субъекты сервера (имена для входа) Azure AD поддерживают выполнение заданий и управление агентом SQL Server.
- Операции резервного копирования и восстановления базы данных могут быть выполнены с помощью субъектов сервера (имен для входа) Azure AD.
- Поддерживается [аудит](auditing-configure.md) всех инструкций, связанных с субъектами сервера (именами для входа) Azure AD и событиями проверки подлинности.
- Выделенное административное соединение для субъектов сервера (имен для входа) Azure AD, которые являются участниками роли сервера `sysadmin`.
- Субъекты сервера (имена для входа) Azure AD поддерживают использование [служебной программы sqlcmd](/sql/tools/sqlcmd-utility) и средства [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms).
- Триггеры входа поддерживаются для событий входа, поступающих от субъектов сервера (имен для входа) Azure AD.
- С использованием субъектов сервера (имен для входа) Azure AD можно настроить компоненты Service Broker и Database Mail.

## <a name="next-steps"></a>Дальнейшие действия

### <a name="enable-security-features"></a>Включение функций безопасности

Полный список способов защиты базы данных приведен в разделе [Возможности безопасности Управляемого экземпляра SQL](sql-managed-instance-paas-overview.md#security-features). Рассматриваются следующие средства безопасности:

- [Аудит управляемого экземпляра SQL](auditing-configure.md)
- [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine)
- [Обнаружение угроз](threat-detection-configure.md)
- [Динамическое маскирование данных](/sql/relational-databases/security/dynamic-data-masking)
- [Безопасность на уровне строк](/sql/relational-databases/security/row-level-security)
- [Прозрачное шифрование данных (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql)

### <a name="sql-managed-instance-capabilities"></a>Возможности Управляемого экземпляра SQL

Полный обзор возможностей Управляемого экземпляра SQL см. здесь:

> [!div class="nextstepaction"]
> [Возможности Управляемого экземпляра SQL](sql-managed-instance-paas-overview.md)