---
title: Создание пользователей с базой данных Azure для MariaDB
description: В этой статье описывается создание учетных записей пользователей для взаимодействия с сервером базы данных Azure для MariaDB.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: how-to
ms.date: 01/18/2021
ms.openlocfilehash: 28ec060e95d09cb150fc699919dde6cc0e1eaf23
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98663712"
---
# <a name="create-users-in-azure-database-for-mariadb"></a>Создание пользователей в базе данных Azure для MariaDB

В этой статье описывается создание пользователей в базе данных Azure для MariaDB.

При создании первой базы данных Azure для MariaDB вы указали имя пользователя и пароль администратора сервера, используемые для входа. Дополнительные сведения см. в [этом кратком руководстве](quickstart-create-mariadb-server-database-using-azure-portal.md). Имя пользователя администратора сервера можно найти на портале Azure.

> [!NOTE]
> Эта статья содержит ссылки на термин « _Ведомый_» термин, который корпорация Майкрософт больше не использует. Когда этот термин будет удален из программного обеспечения, мы удалим его из статьи.

Администратор сервера получает определенные привилегии для выполнения следующих операций на сервере: SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, PROCESS, REFERENCES, INDEX, ALTER, SHOW DATABASES, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER.

После создания сервера базы данных Azure для MariaDB можно использовать учетную запись первого администратора сервера для создания дополнительных пользователей и предоставления им прав администратора. Кроме того, учетная запись администратора сервера может использоваться для создания менее привилегированных пользователей, имеющих доступ к отдельным схемам базы данных.

> [!NOTE]
> Роль SUPER Privilege и DBA не поддерживаются. Проверьте [права](concepts-limits.md#privileges--data-manipulation-support) в статье ограничения, чтобы понять, что не поддерживается в службе.<br><br>
> Такие подключаемые модули паролей, как "validate_password" и "caching_sha2_password", не поддерживаются службой.

## <a name="create-more-admin-users"></a>Создание дополнительных пользователей с правами администратора

1. Получите сведения о подключении и имя пользователя администратора.
   Чтобы подключиться к серверу базы данных, вам потребуются учетные данные администратора для входа и полное имя сервера. Вы можете легко найти данные для входа на странице **Обзор** сервера или на странице **Свойства** на портале Azure.

2. Используйте учетную запись и пароль администратора для подключения к серверу базы данных. Используйте предпочитаемый клиентский инструмент, например MySQL Workbench, mysql.exe, HeidiSQL и т. д.
   Если вы не знаете, как подключиться, см. статью [Использование MySQL Workbench для подключения и запроса данных](./connect-workbench.md)

3. Измените и выполните следующий код SQL. Замените новым именем пользователя значение заполнителя `new_master_user`. Эта синтаксическая конструкция предоставляет перечисленные привилегии для всех схем базы данных (*.*) имени пользователя (new_master_user в этом примере). 

   ```sql
   CREATE USER 'new_master_user'@'%' IDENTIFIED BY 'StrongPassword!';
   
   GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, PROCESS, REFERENCES, INDEX, ALTER, SHOW DATABASES, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER ON *.* TO 'new_master_user'@'%' WITH GRANT OPTION; 
   
   FLUSH PRIVILEGES;
   ```

4. Проверьте предоставленные права.

   ```sql
   USE sys;
   
   SHOW GRANTS FOR 'new_master_user'@'%';
   ```

## <a name="create-database-users"></a>Создание пользователей базы данных

1. Получите сведения о подключении и имя пользователя администратора.
   Чтобы подключиться к серверу базы данных, вам потребуются учетные данные администратора для входа и полное имя сервера. Вы можете легко найти данные для входа на странице **Обзор** сервера или на странице **Свойства** на портале Azure. 

2. Используйте учетную запись и пароль администратора для подключения к серверу базы данных. Используйте предпочитаемый клиентский инструмент, например MySQL Workbench, mysql.exe, HeidiSQL и т. д.
   Если вы не знаете, как подключиться, прочитайте раздел [База данных Azure для MySQL: подключение и запрос данных с помощью MySQL Workbench](./connect-workbench.md).

3. Измените и выполните следующий код SQL. Замените значение заполнителя `db_user` новым именем пользователя, а значение заполнителя `testdb` — именем базы данных.

   Этот код SQL создает базу данных testdb для примера. Затем он создает пользователя в службе базы данных Azure для MariaDB и предоставляет ему все привилегии для новой схемы базы данных (testdb.\*). 

   ```sql
   CREATE DATABASE testdb;
   
   CREATE USER 'db_user'@'%' IDENTIFIED BY 'StrongPassword!';
   
   GRANT ALL PRIVILEGES ON testdb . * TO 'db_user'@'%';
   
   FLUSH PRIVILEGES;
   ```

4. Проверьте предоставление привилегий в базе данных.

   ```sql
   USE testdb;
   
   SHOW GRANTS FOR 'db_user'@'%';
   ```

5. Войдите на сервер, указав указанную базу данных, используя новое имя пользователя и пароль. В этом примере показана командная строка MySQL. С помощью этой команды вам будет предложено ввести пароль для имени пользователя. Укажите собственные имя сервера, имя базы данных и имя пользователя.

   ```bash
   mysql --host mydemoserver.mariadb.database.azure.com --database testdb --user db_user@mydemoserver -p
   ```

   Дополнительные сведения об управлении учетными записями пользователей см. в документации по MariaDB для [управления учетными записями пользователей](https://mariadb.com/kb/en/library/user-account-management/), [синтаксиса Grant](https://mariadb.com/kb/en/library/grant/)и [привилегий](https://mariadb.com/kb/en/library/grant/#privilege-levels).

## <a name="azure_superuser"></a>azure_superuser

Все серверы базы данных Azure для MySQL создаются с помощью пользователя с именем "azure_superuser". Это системная учетная запись, созданная корпорацией Майкрософт для управления сервером с целью проведения мониторинга, резервного копирования и другого регулярного обслуживания. Инженеры по вызову также могут использовать эту учетную запись для доступа к серверу во время инцидента с проверкой подлинности на основе сертификата и должны запрашивать доступ с помощью JIT-процессов.

## <a name="next-steps"></a>Дальнейшие действия

Откройте брандмауэр для IP-адресов компьютеров новых пользователей, чтобы обеспечить их подключение. Для этого ознакомьтесь с разделом [Создание правил брандмауэра базы данных Azure для MariaDB и управление ими с помощью портала Azure](howto-manage-firewall-portal.md).  

<!--or [Azure CLI](howto-manage-firewall-using-cli.md).-->
