---
title: Создание баз данных и пользователей — база данных Azure для MySQL
description: В этой статье описывается создание новых учетных записей пользователей для взаимодействия с сервером базы данных Azure для MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 01/13/2021
ms.openlocfilehash: 9a6346a2b62c81dd74cf0ebe9a85df12d3488679
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98251256"
---
# <a name="create-databases-and-users-in-azure-database-for-mysql"></a>Создание баз данных и пользователей в базе данных Azure для MySQL

[!INCLUDE[applies-to-single-flexible-server](includes/applies-to-single-flexible-server.md)]

В этой статье описывается создание пользователей в базе данных Azure для MySQL.

> [!NOTE]
> Эта статья содержит ссылки на термин « _Ведомый_» термин, который корпорация Майкрософт больше не использует. Когда этот термин будет удален из программного обеспечения, мы удалим его из статьи.
>

При создании сервера базы данных Azure для MySQL вы указали имя пользователя и пароль администратора сервера. Дополнительные сведения см. в этом [кратком руководстве](quickstart-create-mysql-server-database-using-azure-portal.md). Вы можете определить имя пользователя администратора сервера в портал Azure.

Пользователь с правами администратора сервера имеет следующие права:

   SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, PROCESS, REFERENCES, INDEX, ALTER, SHOW DATABASES, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER.

После создания базы данных Azure для сервера MySQL можно использовать учетную запись администратора первого сервера для создания дополнительных пользователей и предоставления им прав администратора. Учетную запись администратора сервера также можно использовать для создания менее привилегированных пользователей, имеющих доступ к отдельным схемам базы данных.

> [!NOTE]
> Роль SUPER Privilege и DBA не поддерживаются. Проверьте [права](concepts-limits.md#privileges--data-manipulation-support) в статье ограничения, чтобы понять, что не поддерживается в службе.
>
> Подключаемые модули паролей, такие как `validate_password` и, `caching_sha2_password` не поддерживаются службой.

## <a name="to-create-a-database-with-a-non-admin-user-in-azure-database-for-mysql"></a>Создание базы данных с пользователем без прав администратора в базе данных Azure для MySQL

1. Получите сведения о подключении и имя пользователя администратора.
   Чтобы подключиться к серверу базы данных, вам потребуются учетные данные администратора для входа и полное имя сервера. Имя сервера и данные для входа можно легко найти на странице **Обзор** сервера или на странице **свойства** в портал Azure.

2. Используйте учетную запись и пароль администратора для подключения к серверу базы данных. Используйте предпочтительное клиентское средство, например MySQL Workbench, mysql.exe или Хеидискл.

   Если вы не знаете, как подключиться, см. статью [подключение и запрос данных для одного сервера](./connect-workbench.md) или [подключение и запрос данных для гибкого сервера](./flexible-server/connect-workbench.md).

3. Измените и выполните следующий код SQL. Замените значение заполнителя на `db_user` предполагаемое новое имя пользователя. Замените значение заполнителя `testdb` именем базы данных.

   Этот код SQL создает новую базу данных с именем TestDB. Затем он создает нового пользователя в службе MySQL и предоставляет этому пользователю все привилегии для новой схемы базы данных (testdb. \* ).

   ```sql
   CREATE DATABASE testdb;

   CREATE USER 'db_user'@'%' IDENTIFIED BY 'StrongPassword!';

   GRANT ALL PRIVILEGES ON testdb . * TO 'db_user'@'%';

   FLUSH PRIVILEGES;
   ```

4. Проверьте предоставление разрешений в базе данных:

   ```sql
   USE testdb;

   SHOW GRANTS FOR 'db_user'@'%';
   ```

5. Войдите на сервер, указав указанную базу данных и используя новое имя пользователя и пароль. В этом примере показана командная строка MySQL. При использовании этой команды будет предложено ввести пароль пользователя. Используйте собственное имя сервера, имя базы данных и имя пользователя.

   ### <a name="single-server"></a>[Отдельный сервер](#tab/single-server)

   ```azurecli-interactive
   mysql --host mydemoserver.mysql.database.azure.com --database testdb --user db_user@mydemoserver -p
   ```

   ### <a name="flexible-server"></a>[Гибкий сервер](#tab/flexible-server)

   ```azurecli-interactive
   mysql --host mydemoserver.mysql.database.azure.com --database testdb --user db_user -p
   ```
 ---

## <a name="to-create-more-admin-users-in-azure-database-for-mysql"></a>Создание дополнительных пользователей с правами администратора в базе данных Azure для MySQL

1. Получите сведения о подключении и имя пользователя администратора.
   Чтобы подключиться к серверу базы данных, вам потребуются учетные данные администратора для входа и полное имя сервера. Имя сервера и данные для входа можно легко найти на странице **Обзор** сервера или на странице **свойства** в портал Azure.

2. Используйте учетную запись и пароль администратора для подключения к серверу базы данных. Используйте предпочтительное клиентское средство, например MySQL Workbench, mysql.exe или Хеидискл.

   Если вы не знаете, как подключиться, см. статью [Использование MySQL Workbench для подключения и запроса данных](./connect-workbench.md).

3. Измените и выполните следующий код SQL. Замените значение заполнителя `new_master_user` новым именем пользователя. Этот синтаксис предоставляет пользователю перечисленные права доступа ко всем схемам базы данных *(* `new_master_user` в данном примере это).

   ```sql
   CREATE USER 'new_master_user'@'%' IDENTIFIED BY 'StrongPassword!';

   GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, PROCESS, REFERENCES, INDEX, ALTER, SHOW DATABASES, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER ON *.* TO 'new_master_user'@'%' WITH GRANT OPTION;

   FLUSH PRIVILEGES;
   ```

4. Проверьте предоставление разрешений:

   ```sql
   USE sys;

   SHOW GRANTS FOR 'new_master_user'@'%';
   ```

## <a name="azure_superuser"></a>azure_superuser

Все серверы базы данных Azure для MySQL создаются с помощью пользователя с именем "azure_superuser". Это системная учетная запись, созданная корпорацией Майкрософт для управления сервером с целью проведения мониторинга, резервного копирования и другого регулярного обслуживания. Инженеры по вызову также могут использовать эту учетную запись для доступа к серверу во время инцидента с проверкой подлинности на основе сертификата и должны запрашивать доступ с помощью JIT-процессов.

## <a name="next-steps"></a>Дальнейшие действия

Откройте брандмауэр для IP-адресов компьютеров новых пользователей, чтобы обеспечить их подключение:

* [Создание правил брандмауэра на одном сервере и управление ими](howto-manage-firewall-using-portal.md)
* [Создание правил брандмауэра и управление ими на гибком сервере](flexible-server/how-to-connect-tls-ssl.md)

Дополнительные сведения об управлении учетными записями пользователей см. в документации по продукту MySQL для [управления учетными записями пользователей](https://dev.mysql.com/doc/refman/5.7/en/access-control.html), [синтаксиса Grant](https://dev.mysql.com/doc/refman/5.7/en/grant.html)и [привилегий](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html).
