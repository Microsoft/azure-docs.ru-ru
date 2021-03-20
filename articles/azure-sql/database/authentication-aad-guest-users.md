---
title: Создание гостевых пользователей Azure AD
description: Как создать гостевых пользователей Azure AD и задать их в качестве администратора Azure AD без использования групп Azure AD в базе данных SQL Azure, Управляемый экземпляр Azure SQL и Azure синапсе Analytics
ms.service: sql-db-mi
ms.subservice: security
ms.custom: azure-synapse
ms.topic: how-to
author: shohamMSFT
ms.author: shohamd
ms.reviewer: vanto
ms.date: 07/27/2020
ms.openlocfilehash: 7a4d9fb9f803a497e84fa189d9a89c2d9097bb70
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92675049"
---
# <a name="create-azure-ad-guest-users-and-set-as-an-azure-ad-admin"></a>Создание гостевых пользователей Azure AD и предоставление им прав администратора Azure AD

[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

> [!NOTE]
> Эта статья предоставляется в режиме **общедоступной предварительной версии**.

Гостевые пользователи в Azure Active Directory (Azure AD) — это пользователи, которые были импортированы в текущую службу Azure AD из других каталогов Azure Active Directory или за ее пределами. Например, гостевые пользователи могут включать пользователей из других каталогов Azure Active Directory или из таких учетных записей, как *\@ Outlook.com*, *\@ Hotmail.com*, *\@ Live.com* или *\@ Gmail.com*. В этой статье показано, как создать гостевой пользователь Azure AD и задать этого пользователя в качестве администратора Azure AD для логического сервера Azure SQL, не требуя, чтобы гостевой пользователь был членом группы в Azure AD.

## <a name="feature-description"></a>Описание компонента

Эта функция отменяет текущее ограничение, которое позволяет гостевым пользователям подключаться к базе данных SQL Azure, Управляемый экземпляр SQL или Azure синапсе Analytics, когда они входят в группу, созданную в Azure AD. Группа, которую необходимо сопоставить с пользователем вручную с помощью инструкции [CREATE USER (Transact-SQL)](/sql/t-sql/statements/create-user-transact-sql) в данной базе данных. После создания пользователя базы данных для группы Azure AD, содержащей гостевого пользователя, гостевой пользователь может войти в базу данных, используя Azure Active Directory с проверкой подлинности MFA. В рамках этой **общедоступной предварительной версии** гостевые пользователи могут быть созданы и подключены непосредственно к базе данных SQL, SQL управляемый экземпляр или Azure синапсе без необходимости сначала добавлять их в группу Azure AD, а затем создавать пользователя базы данных для этой группы Azure AD.

В рамках этой функции вы также можете настроить гостевой пользователь Azure AD непосредственно в качестве администратора AD для логического сервера Azure SQL. Существующие функции, в которых гостевой пользователь может входить в группу Azure AD, и эту группу можно задать в качестве администратора Azure AD для логического сервера Azure SQL. Эти изменения также не затрагивают гостевых пользователей в базе данных, которая является частью группы Azure AD.

Дополнительные сведения о существующей поддержке гостевых пользователей с помощью групп Azure AD см. [в разделе Использование многофакторной проверки подлинности Azure Active Directory](authentication-mfa-ssms-overview.md).

## <a name="prerequisite"></a>Предварительное требование

- [AZ. SQL 2.9.0](https://www.powershellgallery.com/packages/Az.Sql/2.9.0) Module или более поздней версии требуется при использовании PowerShell для задания гостевого пользователя в качестве администратора Azure AD для логического сервера Azure SQL.

## <a name="create-database-user-for-azure-ad-guest-user"></a>Создание пользователя базы данных для гостевого пользователя Azure AD 

Выполните следующие действия, чтобы создать пользователя базы данных с помощью гостевой учетной записи Azure AD.

### <a name="create-guest-user-in-sql-database-and-azure-synapse"></a>Создание гостевого пользователя в базе данных SQL и Azure синапсе

1. Убедитесь, что гостевой пользователь (например, `user1@gmail.com` ) уже добавлен в Azure AD, а администратор Azure AD настроен для сервера базы данных. Для проверки подлинности Azure Active Directory требуется администратор Azure AD.

1. Подключитесь к базе данных SQL в качестве администратора Azure AD или пользователя Azure AD с достаточными разрешениями SQL для создания пользователей и выполните следующую команду в базе данных, в которую необходимо добавить гостевой пользователь:

    ```sql
    CREATE USER [user1@gmail.com] FROM EXTERNAL PROVIDER
    ```

1. Теперь для гостевого пользователя должен быть создан пользователь базы данных `user1@gmail.com` .

1. Выполните следующую команду, чтобы убедиться, что пользователь базы данных успешно создан:

    ```sql
    SELECT * FROM sys.database_principals
    ```

1. Отключите и войдите в базу данных в качестве гостевого пользователя `user1@gmail.com` с помощью [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) , используя метод проверки подлинности **Azure Active Directory-Universal с MFA**. Дополнительные сведения см. [в разделе Использование многофакторной Azure Active Directory проверки подлинности](authentication-mfa-ssms-overview.md).

### <a name="create-guest-user-in-sql-managed-instance"></a>Создание гостевого пользователя в SQL Управляемый экземпляр

> [!NOTE]
> SQL Управляемый экземпляр поддерживает имена входа для пользователей Azure AD, а также пользователей автономной базы данных Azure AD. Ниже описано, как создать имя входа и пользователя для гостевого пользователя Azure AD в Управляемый экземпляр SQL. Вы также можете создать [пользователя автономной базы данных](/sql/relational-databases/security/contained-database-users-making-your-database-portable) в SQL управляемый экземпляр с помощью метода в разделе [Создание гостевого пользователя в базе данных SQL и Azure синапсе](#create-guest-user-in-sql-database-and-azure-synapse) .

1. Убедитесь, что гостевой пользователь (например, `user1@gmail.com` ) уже добавлен в Azure AD, а администратор Azure AD задан для сервера SQL управляемый экземпляр. Для проверки подлинности Azure Active Directory требуется администратор Azure AD.

1. Подключитесь к серверу SQL Управляемый экземпляр в качестве администратора Azure AD или пользователя Azure AD с достаточными разрешениями SQL для создания пользователей и выполните следующую команду в `master` базе данных, чтобы создать имя входа для гостевого пользователя:

    ```sql
    CREATE LOGIN [user1@gmail.com] FROM EXTERNAL PROVIDER
    ```

1. Теперь `user1@gmail.com` в базе данных должно быть создано имя входа для гостевого пользователя `master` .

1. Выполните следующую команду, чтобы убедиться, что имя входа успешно создано:

    ```sql
    SELECT * FROM sys.server_principals
    ```

1. Выполните следующую команду в базе данных, в которую необходимо добавить гостевой пользователь: 

    ```sql
    CREATE USER [user1@gmail.com] FROM LOGIN [user1@gmail.com]
    ```

1. Теперь для гостевого пользователя должен быть создан пользователь базы данных `user1@gmail.com` .

1. Отключите и войдите в базу данных в качестве гостевого пользователя `user1@gmail.com` с помощью [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) , используя метод проверки подлинности **Azure Active Directory-Universal с MFA**. Дополнительные сведения см. [в разделе Использование многофакторной Azure Active Directory проверки подлинности](authentication-mfa-ssms-overview.md).

## <a name="setting-a-guest-user-as-an-azure-ad-admin"></a>Настройка гостевого пользователя в качестве администратора Azure AD

Выполните следующие действия, чтобы настроить гостевой пользователь Azure AD в качестве администратора Azure AD для логического сервера SQL.

### <a name="set-azure-ad-admin-for-sql-database-and-azure-synapse"></a>Настройка администратора Azure AD для базы данных SQL и Azure синапсе

1. Убедитесь, что гостевой пользователь (например, `user1@gmail.com` ) уже добавлен в Azure AD.

1. Выполните следующую команду PowerShell, чтобы добавить гостевого пользователя в качестве администратора Azure AD для логического сервера Azure SQL:

    - Замените `<ResourceGroupName>` именем группы ресурсов Azure, содержащей логический сервер SQL Azure.
    - Замените на `<ServerName>` имя логического сервера SQL Azure. Если имя сервера `myserver.database.windows.net`, замените `<Server Name>` на `myserver`.
    - Замените `<DisplayNameOfGuestUser>` именем гостевого пользователя.

    ```powershell
    Set-AzSqlServerActiveDirectoryAdministrator -ResourceGroupName <ResourceGroupName> -ServerName <ServerName> -DisplayName <DisplayNameOfGuestUser>
    ```

    Вы также можете воспользоваться командой Azure CLI [AZ SQL Server AD-Admin](/cli/azure/sql/server/ad-admin) , чтобы задать гостевого пользователя в качестве администратора Azure AD для логического сервера Azure SQL.

### <a name="set-azure-ad-admin-for-sql-managed-instance"></a>Настройка администратора Azure AD для Управляемый экземпляр SQL

1. Убедитесь, что гостевой пользователь (например, `user1@gmail.com` ) уже добавлен в Azure AD.

1. Перейдите к [портал Azure](https://portal.azure.com)и перейдите к ресурсу **Azure Active Directory** . В разделе **Управление** перейдите в область **Пользователи** . Выберите пользователя Guest и запишите `Object ID` . 

1. Выполните следующую команду PowerShell, чтобы добавить гостевого пользователя в качестве администратора Azure AD для Управляемый экземпляр SQL:

    - Замените `<ResourceGroupName>` именем группы ресурсов Azure, содержащей управляемый экземпляр SQL.
    - Замените на `<ManagedInstanceName>` имя SQL управляемый экземпляр.
    - Замените `<DisplayNameOfGuestUser>` именем гостевого пользователя.
    - Замените `<AADObjectIDOfGuestUser>` на `Object ID` собранный ранее.

    ```powershell
    Set-AzSqlInstanceActiveDirectoryAdministrator -ResourceGroupName <ResourceGroupName> -InstanceName "<ManagedInstanceName>" -DisplayName <DisplayNameOfGuestUser> -ObjectId <AADObjectIDOfGuestUser>
    ```

    Вы также можете использовать команду Azure CLI [AZ SQL MI AD-Admin](/cli/azure/sql/mi/ad-admin) , чтобы задать гостевого пользователя в качестве администратора Azure ad для управляемый экземпляр SQL.

## <a name="limitations"></a>Ограничения

Существует ограничение на портал Azure, которое предотвращает выбор гостевого пользователя Azure AD в качестве администратора Azure AD для Управляемый экземпляр SQL. Для гостевых учетных записей за пределами Azure AD, например *\@ Outlook.com*, *\@ Hotmail.com*, *\@ Live.com* или *\@ Gmail.com*, в селекторе администратора Active Directory отображаются эти учетные записи, но они не могут быть выбраны. Используйте указанные выше [команды PowerShell или CLI](#setting-a-guest-user-as-an-azure-ad-admin) , чтобы задать администратора Azure AD. Кроме того, Группа Azure AD, содержащая гостевого пользователя, может быть задана в качестве администратора Azure AD для Управляемый экземпляр SQL.

Эта функция будет включена для Управляемый экземпляр SQL до общего доступа к этой функции.

## <a name="next-steps"></a>Дальнейшие действия

- [Настройка и администрирование проверки подлинности Azure Active Directory с помощью Azure SQL](authentication-aad-configure.md)
- [Использование многофакторной проверки подлинности Azure Active Directory](authentication-mfa-ssms-overview.md)
- [CREATE USER (Transact-SQL)](/sql/t-sql/statements/create-user-transact-sql)