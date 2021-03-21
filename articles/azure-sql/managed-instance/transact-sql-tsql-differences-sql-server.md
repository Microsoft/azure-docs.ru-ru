---
title: Отличия T-SQL от SQL Server & SQL Azure Управляемый экземпляр
description: В этой статье обсуждаются различия Transact-SQL (T-SQL) между Управляемый экземпляр и SQL Server SQL Azure.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.devlang: ''
ms.topic: reference
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein, bonova, danil
ms.date: 3/16/2021
ms.custom: seoapril2019, sqldbrb=1
ms.openlocfilehash: 1afd5a0e24e144169280e683321b5843e9766136
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103601378"
---
# <a name="t-sql-differences-between-sql-server--azure-sql-managed-instance"></a>Отличия T-SQL от SQL Server & SQL Azure Управляемый экземпляр
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

В этой статье кратко описаны различия в синтаксисе и поведении служб Azure SQL Управляемый экземпляр и SQL Server. 


SQL Управляемый экземпляр обеспечивает высокую совместимость с ядром СУБД SQL Server, и большинство функций поддерживаются в Управляемый экземпляр SQL.

![Простая миграция с SQL Server](./media/transact-sql-tsql-differences-sql-server/migration.png)

Существуют некоторые ограничения PaaS, появившиеся в SQL Управляемый экземпляр и некоторые изменения в поведении по сравнению с SQL Server. Различия делятся на следующие категории: <a name="Differences"></a>

- [Доступность](#availability) включает различия в [Always on группах доступности](#always-on-availability-groups) и [резервных копиях](#backup).
- [Безопасность](#security) включает в себя различия в [аудите](#auditing), [сертификатах](#certificates), [учетных данных](#credential), [поставщиках служб шифрования](#cryptographic-providers), [именах входа и пользователях](#logins-and-users), а также [ключе службы и главный ключ службы](#service-key-and-service-master-key).
- [Конфигурация](#configuration) включает различия в [расширении буферного пула](#buffer-pool-extension), параметрах [сортировки](#collation), [уровнях совместимости](#compatibility-levels), [зеркальном отображении баз данных](#database-mirroring), [параметрах базы данных](#database-options), [Агент SQL Server](#sql-server-agent)и [параметрах таблиц](#tables).
- К [функциональным возможностям](#functionalities) относятся [BULK INSERT/OPENROWSET](#bulk-insert--openrowset), [CLR](#clr), [DBCC](#dbcc), [распределенные транзакции](#distributed-transactions), [Расширенные события](#extended-events), [внешние библиотеки](#external-libraries), [FileStream и FileTable](#filestream-and-filetable), [Полнотекстовый семантический поиск](#full-text-semantic-search), [связанные серверы](#linked-servers), [polybase](#polybase), [репликация](#replication), [Восстановление](#restore-statement), [Service Broker](#service-broker), [хранимые процедуры, функции и триггеры](#stored-procedures-functions-and-triggers).
- [Параметры среды](#Environment) , такие как виртуальных сетей и конфигурации подсети.

Большинство этих функций являются ограничениями архитектуры и представляют функции служб.

Временные известные проблемы, обнаруженные в SQL Управляемый экземпляр и которые будут устранены в будущем, описаны на [странице заметок о выпуске](../database/doc-changes-updates-release-notes.md).

## <a name="availability"></a>Доступность

### <a name="always-on-availability-groups"></a><a name="always-on-availability-groups"></a>Группы доступности Always On

[Высокий уровень доступности](../database/high-availability-sla.md) ВСТРОЕН в SQL управляемый экземпляр и не может контролироваться пользователями. Следующие инструкции не поддерживаются:

- [CREATE ENDPOINT … FOR DATABASE_MIRRORING](/sql/t-sql/statements/create-endpoint-transact-sql);
- [CREATE AVAILABILITY GROUP](/sql/t-sql/statements/create-availability-group-transact-sql)
- [ALTER AVAILABILITY GROUP](/sql/t-sql/statements/alter-availability-group-transact-sql)
- [DROP AVAILABILITY GROUP](/sql/t-sql/statements/drop-availability-group-transact-sql)
- Предложение [Set HADR](/sql/t-sql/statements/alter-database-transact-sql-set-hadr) инструкции [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql)

### <a name="backup"></a>Backup

SQL Управляемый экземпляр имеет автоматическое резервное копирование, поэтому пользователи могут создавать полные `COPY_ONLY` резервные копии баз данных. Разностные резервные копии, журналы и моментальные снимки файлов не поддерживаются.

- С помощью Управляемый экземпляр SQL можно создать резервную копию базы данных экземпляра только в учетной записи хранилища BLOB-объектов Azure.
  - Поддерживается только `BACKUP TO URL`.
  - `FILE``TAPE`устройства резервного копирования не поддерживаются.
- Большинство общих `WITH` параметров поддерживается.
  - `COPY_ONLY` является обязательным.
  - `FILE_SNAPSHOT` не поддерживается.
  - Параметры ленты: `REWIND` , `NOREWIND` , и `UNLOAD` `NOUNLOAD` не поддерживаются.
  - Параметры конкретного журнала: `NORECOVERY` , `STANDBY` и `NO_TRUNCATE` не поддерживаются.

Ограничения 

- С помощью Управляемый экземпляр SQL можно создать резервную копию базы данных экземпляра в резервной копии с 32 полосами, что достаточно для баз данных размером до 4 ТБ, если используется сжатие резервных копий.
- Невозможно выполнить `BACKUP DATABASE ... WITH COPY_ONLY` в базе данных, которая зашифрована с помощью прозрачное шифрование данных, управляемого службой (TDE). Управляемые службой TDE принудительное шифрование резервных копий с помощью внутреннего ключа TDE. Ключ не может быть экспортирован, поэтому восстановить резервную копию невозможно. Используйте автоматическое резервное копирование и восстановление на момент времени или используйте [управляемую клиентом (BYOK) TDE](../database/transparent-data-encryption-tde-overview.md#customer-managed-transparent-data-encryption---bring-your-own-key) . Также можно отключить шифрование базы данных.
- Собственные резервные копии, созданные на Управляемый экземпляр, нельзя восстановить в SQL Server. Это связано с тем, что Управляемый экземпляр имеет более высокую версию внутренней базы данных по сравнению с любой версией SQL Server.
- Максимальный размер полосы резервного копирования с помощью `BACKUP` команды в SQL управляемый экземпляр составляет 195 ГБ, что является максимальным размером большого двоичного объекта. Чтобы уменьшить размер отдельного чередующегося набора и соблюсти это ограничение, можно увеличить число чередующихся наборов в команде резервного копирования.

    > [!TIP]
    > Чтобы обойти это ограничение, при резервном копировании базы данных из SQL Server в локальной среде или на виртуальной машине можно выполнить следующие действия.
    >
    > - Создайте резервную копию до `DISK` вместо резервного копирования на `URL` .
    > - Отправьте файлы резервных копий в хранилище BLOB-объектов.
    > - Восстановление в SQL Управляемый экземпляр.
    >
    > `Restore`Команда в SQL управляемый экземпляр поддерживает большие размеры больших двоичных объектов в файлах резервных копий, так как для хранения загруженных файлов резервных копий используется другой тип больших двоичных объектов.

Сведения о резервном копировании с помощью T-SQL см. в статье [BACKUP (Transact-SQL)](/sql/t-sql/statements/backup-transact-sql).

## <a name="security"></a>Безопасность

### <a name="auditing"></a>Аудит

Основные различия между аудитом в Microsoft Azure SQL и в SQL Server:

- При использовании SQL Управляемый экземпляр аудит работает на уровне сервера. `.xel`Файлы журналов хранятся в хранилище BLOB-объектов Azure.
- При использовании базы данных SQL Azure аудит работает на уровне базы данных. `.xel`Файлы журналов хранятся в хранилище BLOB-объектов Azure.
- При использовании SQL Server, локальных или виртуальных машин аудит работает на уровне сервера. События хранятся в файловой системе или в журналах событий Windows.
 
Аудит XEvent в SQL Управляемый экземпляр поддерживает целевые объекты хранилища BLOB-объектов Azure. Журналы файлов и Windows не поддерживаются.

Основные различия в синтаксисе `CREATE AUDIT` для аудита в хранилище BLOB-объектов Azure:

- Имеется новый синтаксис `TO URL` , который можно использовать для указания URL-адреса контейнера хранилища больших двоичных объектов Azure, в который `.xel` помещаются файлы.
- Синтаксис `TO FILE` не поддерживается, так как SQL управляемый экземпляр не может получить доступ к файловым ресурсам Windows.

Дополнительные сведения см. в разделе: 

- [CREATE SERVER AUDIT](/sql/t-sql/statements/create-server-audit-transact-sql) 
- [ALTER SERVER AUDIT](/sql/t-sql/statements/alter-server-audit-transact-sql)
- [Аудит](/sql/relational-databases/security/auditing/sql-server-audit-database-engine)

### <a name="certificates"></a>Сертификаты

SQL Управляемый экземпляр не может получить доступ к файловым ресурсам и папкам Windows, поэтому применяются следующие ограничения.

- `CREATE FROM` / `BACKUP TO` Файл не поддерживается для сертификатов.
- `CREATE` / `BACKUP` Сертификат из `FILE` / `ASSEMBLY` не поддерживается. Невозможно использовать файлы закрытых ключей. 

См. статьи [Инструкция CREATE CERTIFICATE (Transact-SQL)](/sql/t-sql/statements/create-certificate-transact-sql) и [BACKUP CERTIFICATE (Transact-SQL)](/sql/t-sql/statements/backup-certificate-transact-sql). 
 
**Обходное решение**. вместо создания резервной копии сертификата и восстановления резервной копии [получите двоичное содержимое сертификата и закрытый ключ, сохраните его в виде SQL Server и создайте из двоичного файла](/sql/t-sql/functions/certencoded-transact-sql#b-copying-a-certificate-to-another-database):

```sql
CREATE CERTIFICATE  
   FROM BINARY = asn_encoded_certificate
WITH PRIVATE KEY (<private_key_options>)
```

### <a name="credential"></a>Учетные данные

Поддерживаются удостоверения только Azure Key Vault и `SHARED ACCESS SIGNATURE`. Пользователи Windows не поддерживаются.

См. статьи [CREATE CREDENTIAL (Transact-SQL)](/sql/t-sql/statements/create-credential-transact-sql) и [ALTER CREDENTIAL (Transact-SQL)](/sql/t-sql/statements/alter-credential-transact-sql).

### <a name="cryptographic-providers"></a>Поставщики служб шифрования

SQL Управляемый экземпляр не удается получить доступ к файлам, поэтому создание поставщиков служб шифрования невозможно:

- `CREATE CRYPTOGRAPHIC PROVIDER` не поддерживается. См. статью [CREATE CRYPTOGRAPHIC PROVIDER (Transact-SQL)](/sql/t-sql/statements/create-cryptographic-provider-transact-sql).
- `ALTER CRYPTOGRAPHIC PROVIDER` не поддерживается. См. статью [ALTER CRYPTOGRAPHIC PROVIDER (Transact-SQL)](/sql/t-sql/statements/alter-cryptographic-provider-transact-sql).

### <a name="logins-and-users"></a>Имена входа и пользователи

- Поддерживаются имена входа SQL, созданные с помощью `FROM CERTIFICATE` , `FROM ASYMMETRIC KEY` и `FROM SID` . См. статью [CREATE LOGIN (Transact-SQL)](/sql/t-sql/statements/create-login-transact-sql).
- Поддерживаются субъекты (имена входа) на сервере Azure Active Directory (Azure AD), созданные с помощью синтаксиса [создания имени входа](/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current) или [создания пользователя из имени входа [Azure AD login]](/sql/t-sql/statements/create-user-transact-sql?view=azuresqldb-mi-current) . Эти имена входа создаются на уровне сервера.

    SQL Управляемый экземпляр поддерживает участников базы данных Azure AD с помощью синтаксиса `CREATE USER [AADUser/AAD group] FROM EXTERNAL PROVIDER` . Эта функция также называется пользователями автономной базы данных Azure AD.

- Имена входа Windows, созданные с помощью `CREATE LOGIN ... FROM WINDOWS` синтаксиса, не поддерживаются. Используйте пользователей и имена для входа Azure Active Directory.
- Пользователь Azure AD, создавший экземпляр, имеет [неограниченные права администратора](../database/logins-create-manage.md).
- Пользователи уровня базы данных Azure AD без прав администратора могут создаваться с помощью `CREATE USER ... FROM EXTERNAL PROVIDER` синтаксиса. См [. раздел Создание пользователя... ОТ внешнего поставщика](../database/authentication-aad-configure.md#create-contained-users-mapped-to-azure-ad-identities).
- Участники сервера Azure AD (имена входа) поддерживают функции SQL только в одном Управляемый экземпляр SQL. Функции, требующие взаимодействия между экземплярами, независимо от того, находятся ли они в одном клиенте Azure AD или разных клиентах, не поддерживаются для пользователей Azure AD. Ниже приведены примеры таких функций:

  - Репликация транзакций SQL.
  - Сервер канала.

- Установка имени для входа Azure AD, сопоставленного с группой Azure AD, в качестве владельца базы данных, не поддерживается.
- Поддерживается олицетворение участников уровня сервера Azure AD с помощью других участников Azure AD, например предложения [EXECUTE AS](/sql/t-sql/statements/execute-as-transact-sql) . Ограничения EXECUTE AS:

  - ВЫПОЛНЕНИЕ от имени пользователя не поддерживается для пользователей Azure AD, если оно отличается от имени для входа. Например, если пользователь создан с помощью синтаксиса CREATE USER [Мяадусер] WITH LOGIN [ john@contoso.com ] и выполняется попытка олицетворения через Exec от имени user = _мяадусер_. При создании **пользователя** на сервере-участнике Azure AD (имя входа) укажите user_name как тот же login_name, что и для **входа**.
  - Только участники уровня SQL Server (имена входа), являющиеся частью `sysadmin` роли, могут выполнять следующие операции, предназначенные для субъектов Azure AD:

    - EXECUTE AS USER;
    - EXECUTE AS LOGIN.

  - Для олицетворения пользователя с помощью инструкции EXECUTE AS пользователь должен быть напрямую сопоставлен с сервером-участником Azure AD (имя входа). Пользователи, являющиеся членами групп Azure AD, сопоставленных с участниками сервера Azure AD, не могут эффективно олицетворять с помощью инструкции EXECUTE AS, даже если вызывающая сторона имеет разрешения IMPERSONATE на указанное имя пользователя.

- Экспорт и импорт базы данных с использованием BACPAC-файлов поддерживается для пользователей Azure AD в Управляемый экземпляр SQL с помощью [SSMS v 18.4 или более поздней версии](/sql/ssms/download-sql-server-management-studio-ssms)или [SQLPackage.exe](/sql/tools/sqlpackage-download).
  - С помощью BACPAC-файла базы данных поддерживаются следующие конфигурации: 
    - Экспорт и импорт базы данных между разными экземплярами управления в пределах одного домена Azure AD.
    - Экспорт базы данных из Управляемый экземпляр SQL и импорт в базу данных SQL в том же домене Azure AD. 
    - Экспортируйте базу данных из базы данных SQL и импортируйте ее в Управляемый экземпляр SQL в том же домене Azure AD.
    - Экспорт базы данных из Управляемый экземпляр SQL и импорт в SQL Server (версии 2012 или более поздней).
      - В этой конфигурации все пользователи Azure AD создаются как SQL Server участников базы данных (пользователей) без имен входа. Тип пользователей отображается как `SQL` и отображается `SQL_USER` в sys.database_principals). Их разрешения и роли остаются в метаданных базы данных SQL Server и могут использоваться для олицетворения. Однако они не могут использоваться для доступа к SQL Server и входа в него с помощью учетных данных.

- Только имя входа субъекта уровня сервера, созданное процессом подготовки SQL Управляемый экземпляр, члены ролей сервера, такие как `securityadmin` или `sysadmin` , или другие имена входа с разрешением ALTER ANY LOGIN на уровне сервера, могут создавать субъекты-серверы Azure AD (имена входа) в базе данных master для SQL управляемый экземпляр.
- Если имя для входа представляет собой субъект SQL, команду create для создания имен для входа для учетной записи Azure AD можно использовать только с именами для входа, которые являются частью роли `sysadmin`.
- Имя входа Azure AD должно быть членом Azure AD в том же каталоге, который используется для Управляемый экземпляр Azure SQL.
- Участники сервера Azure AD (имена входа) отображаются в обозревателе объектов, начиная с версии SQL Server Management Studio 18,0 Предварительная версия 5.
- Допускается перекрытие субъектов сервера (имен для входа) Azure AD с учетной записью администратора Azure AD. Участники сервера Azure AD (имена входа) имеют приоритет над администратором Azure AD при разрешении субъекта и применении разрешений к SQL Управляемый экземпляр.
- Во время проверки подлинности применяется следующая последовательность действий для разрешения субъекта проверки подлинности:

    1. Если учетная запись Azure AD существует как непосредственно сопоставленная с субъектом-сервером Azure AD (именем входа), которая присутствует в sys.server_principals как тип "E", предоставьте доступ и примените разрешения участника на сервере Azure AD (имя входа).
    2. Если учетная запись Azure AD является членом группы Azure AD, сопоставленной с участником сервера Azure AD (именем входа), который имеется в sys.server_principals в качестве типа "X", предоставьте доступ и примените разрешения для имени входа группы Azure AD.
    3. Если учетная запись Azure AD является специальным настроенным порталом Azure AD для SQL Управляемый экземпляр, который не существует в системных представлениях SQL Управляемый экземпляр, примените специальные разрешения для администратора Azure AD для Управляемый экземпляр SQL (устаревший режим).
    4. Если учетная запись Azure AD существует как непосредственно сопоставленная с пользователем Azure AD в базе данных, которая присутствует в sys.database_principals как тип "E", предоставьте доступ и примените разрешения для пользователя базы данных Azure AD.
    5. Если учетная запись Azure AD является членом группы Azure AD, сопоставленной с пользователем Azure AD в базе данных, которая имеется в sys.database_principals типа "X", предоставьте доступ и примените разрешения для имени входа группы Azure AD.
    6. Если есть имя входа Azure AD, сопоставленное с учетной записью пользователя Azure AD или учетной записью группы Azure AD, которая разрешается пользователю, который выполняет проверку подлинности, применяются все разрешения из этого имени входа Azure AD.

### <a name="service-key-and-service-master-key"></a>Ключ службы и главный ключ службы

- [Резервное копирование главного ключа](/sql/t-sql/statements/backup-master-key-transact-sql) не поддерживается (управляется службой базы данных SQL).
- [Восстановление главного ключа](/sql/t-sql/statements/restore-master-key-transact-sql) не поддерживается (управляется службой базы данных SQL).
- [Резервное копирование главного ключа службы](/sql/t-sql/statements/backup-service-master-key-transact-sql) не поддерживается (управляется службой базы данных SQL).
- [Восстановление главного ключа службы](/sql/t-sql/statements/restore-service-master-key-transact-sql) не поддерживается (управляется службой базы данных SQL).

## <a name="configuration"></a>Конфигурация

### <a name="buffer-pool-extension"></a>Расширение буферного пула

- [Расширение буферного пула](/sql/database-engine/configure-windows/buffer-pool-extension) не поддерживается.
- `ALTER SERVER CONFIGURATION SET BUFFER POOL EXTENSION` не поддерживается. См. статью [ALTER SERVER CONFIGURATION (Transact-SQL)](/sql/t-sql/statements/alter-server-configuration-transact-sql).

### <a name="collation"></a>Параметры сортировки

Параметр сортировки экземпляра по умолчанию — `SQL_Latin1_General_CP1_CI_AS`. Этот параметр можно указать как параметр создания. См. статью [Параметры сортировки](/sql/t-sql/statements/collations).

### <a name="compatibility-levels"></a>Уровни совместимости

- Поддерживаемые уровни совместимости: 100, 110, 120, 130, 140 и 150.
- Уровни совместимости ниже 100 не поддерживаются.
- Уровень совместимости по умолчанию для новых баз данных — 140. Для восстановленных баз данных уровень совместимости остается неизменным, если он был 100 и выше.

См. статью [Уровень совместимости инструкции ALTER DATABASE (Transact-SQL)](/sql/t-sql/statements/alter-database-transact-sql-compatibility-level)

### <a name="database-mirroring"></a>Зеркальное отображение базы данных

Зеркальное отображение базы данных не поддерживается.

- Параметры `ALTER DATABASE SET PARTNER` и `SET WITNESS` не поддерживаются.
- `CREATE ENDPOINT … FOR DATABASE_MIRRORING` не поддерживается.

Дополнительные сведения см. в статьях [Зеркальное отображение базы данных ALTER DATABASE (Transact-SQL)](/sql/t-sql/statements/alter-database-transact-sql-database-mirroring) и [CREATE ENDPOINT (Transact-SQL)](/sql/t-sql/statements/create-endpoint-transact-sql).

### <a name="database-options"></a>Параметры базы данных

- Несколько файлов журнала не поддерживаются.
- Объекты в памяти не поддерживаются на уровне служб "Общего назначения". 
- Существует ограничение в 280 файлов на экземпляр общего назначения, что подразумевает максимум 280 файлов на одну базу данных. К этому ограничению подсчитываются файлы данных и журналов на общего назначения уровне. [Критически важный для бизнеса уровень поддерживает 32 767 файлов на одну базу данных](./resource-limits.md#service-tier-characteristics).
- База данных не может содержать файловые группы, содержащие данные FILESTREAM. Восстановление завершается сбоем, если bak содержит `FILESTREAM` данные. 
- Каждый файл помещается в хранилище BLOB-объектов Azure. Операции ввода-вывода и пропускная способность каждого файла зависят от размера каждого файла.

#### <a name="create-database-statement"></a>Инструкция CREATE DATABASE

К следующим ограничениям относятся следующие `CREATE DATABASE` .

- Невозможно определить файлы и файловые группы. 
- `CONTAINMENT`Параметр не поддерживается. 
- `WITH` параметры не поддерживаются. 
   > [!TIP]
   > В качестве обходного решения используйте `ALTER DATABASE` After, `CREATE DATABASE` чтобы задать параметры базы данных, чтобы добавить файлы или задать вложение. 

- `FOR ATTACH`Параметр не поддерживается.
- `AS SNAPSHOT OF`Параметр не поддерживается.

Дополнительные сведения см. в статье [CREATE DATABASE (SQL Server Transact-SQL)](/sql/t-sql/statements/create-database-sql-server-transact-sql).

#### <a name="alter-database-statement"></a>Инструкция ALTER DATABASE

Невозможно задать или изменить некоторые свойства файла:

- Путь к файлу не может быть указан в `ALTER DATABASE ADD FILE (FILENAME='path')` инструкции t-SQL. Удалите `FILENAME` из скрипта, так как SQL управляемый экземпляр автоматически помещает файлы. 
- Имя файла нельзя изменить с помощью `ALTER DATABASE` инструкции.

Следующие параметры задаются по умолчанию и не могут быть изменены.

- `MULTI_USER`
- `ENABLE_BROKER`
- `AUTO_CLOSE OFF`

Невозможно изменить следующие параметры:

- `AUTO_CLOSE`
- `AUTOMATIC_TUNING(CREATE_INDEX=ON|OFF)`
- `AUTOMATIC_TUNING(DROP_INDEX=ON|OFF)`
- `DISABLE_BROKER`
- `EMERGENCY`
- `ENABLE_BROKER`
- `FILESTREAM`
- `HADR`
- `NEW_BROKER`
- `OFFLINE`
- `PAGE_VERIFY`
- `PARTNER`
- `READ_ONLY`
- `RECOVERY BULK_LOGGED`
- `RECOVERY_SIMPLE`
- `REMOTE_DATA_ARCHIVE` 
- `RESTRICTED_USER`
- `SINGLE_USER`
- `WITNESS`

Некоторые `ALTER DATABASE` инструкции (например, [Set Contain](/sql/relational-databases/databases/migrate-to-a-partially-contained-database#converting-a-database-to-partially-contained-using-transact-sql)) могут временно завершаться сбоем, например во время автоматической архивации базы данных или сразу после создания базы данных. В этом случае `ALTER DATABASE` следует повторить попытку. Дополнительные сведения о связанных сообщениях об ошибках см. в [разделе "Примечания](/sql/t-sql/statements/alter-database-transact-sql?preserve-view=true&tabs=sqlpool&view=azuresqldb-mi-current#remarks-2)".

Дополнительные сведения см. в статье [Параметры инструкции ALTER DATABASE для файлов и файловых групп (Transact-SQL)](/sql/t-sql/statements/alter-database-transact-sql-file-and-filegroup-options).

### <a name="sql-server-agent"></a>Агент SQL Server

- Включение и отключение агента SQL Server в настоящее время не поддерживается для Управляемого экземпляра SQL. Агент SQL работает всегда.
- Триггер расписания задания на основе неактивного ЦП не поддерживается.
- Параметры агент SQL Server доступны только для чтения. Эта процедура `sp_set_agent_properties` не поддерживается в управляемый экземпляр SQL. 
- Задания
  - Шаги задания T-SQL поддерживаются.
  - Поддерживаются следующие задания репликации:
    - Читатель журнала транзакций.
    - Моментальный снимок
    - Распространитель
  - Поддерживаются шаги задания служб SSIS.
  - Другие типы шагов заданий в настоящее время не поддерживаются:
    - Шаг задания репликации слиянием не поддерживается. 
    - Читатель очереди пока не поддерживается. 
    - Командная оболочка пока не поддерживается.
  - SQL Управляемый экземпляр не может получить доступ к внешним ресурсам, например к сетевым общим папкам через Robocopy. 
  - SQL Server Analysis Services не поддерживается.
- Уведомления поддерживаются частично.
- Уведомление по электронной почте поддерживается, хотя для этого необходимо настроить профиль Database Mail. Агент SQL Server может использовать только один профиль Database Mail и должен вызываться `AzureManagedInstance_dbmail_profile` . 
  - Пейджер не поддерживается.
  - NetSend не поддерживается.
  - Предупреждения пока не поддерживаются.
  - Прокси-серверы не поддерживаются.
- Журнал событий не поддерживается.
- Для создания, изменения или выполнения заданий агента SQL пользователь должен быть напрямую сопоставлен с сервером-участником Azure AD (именем для входа). Пользователи, которые не сопоставлены напрямую, например пользователи, которые принадлежат к группе Azure AD, имеющей права на создание, изменение или выполнение заданий агента SQL, не смогут эффективно выполнять эти действия. Это происходит из-за Управляемый экземпляр олицетворения и [выполнения в качестве ограничений](#logins-and-users).
- Многосерверное администрирование для заданий главного и целевого сервера (MSX/целевого сервера) не поддерживается.

Сведения об агенте SQL Server см. в статье [Агент SQL Server](/sql/ssms/agent/sql-server-agent).

### <a name="tables"></a>Tables

Следующие типы таблиц не поддерживаются:

- [FILESTREAM](/sql/relational-databases/blob/filestream-sql-server)
- [ПРОВЕРОЧ](/sql/relational-databases/blob/filetables-sql-server)
- [Внешняя таблица](/sql/t-sql/statements/create-external-table-transact-sql) (polybase)
- [MEMORY_OPTIMIZED](/sql/relational-databases/in-memory-oltp/introduction-to-memory-optimized-tables) (не поддерживается только на уровне общего назначения)

Сведения о создании и изменении таблиц см. в разделе [CREATE TABLE](/sql/t-sql/statements/create-table-transact-sql) и [ALTER TABLE](/sql/t-sql/statements/alter-table-transact-sql).

## <a name="functionalities"></a>Функциональные возможности

### <a name="bulk-insert--openrowset"></a>BULK INSERT/OPENROWSET

SQL Управляемый экземпляр не может получить доступ к файловым ресурсам и папкам Windows, поэтому файлы необходимо импортировать из хранилища BLOB-объектов Azure:

- `DATASOURCE` требуется в `BULK INSERT` команде при импорте файлов из хранилища BLOB-объектов Azure. См. статью [BULK INSERT (Transact-SQL)](/sql/t-sql/statements/bulk-insert-transact-sql).
- `DATASOURCE` требуется в `OPENROWSET` функции при чтении содержимого файла из хранилища BLOB-объектов Azure. См. статью [OPENROWSET (Transact-SQL)](/sql/t-sql/functions/openrowset-transact-sql).
- `OPENROWSET` может использоваться для чтения данных из базы данных SQL Azure, Управляемый экземпляр Azure SQL или SQL Server экземпляров. Другие источники, такие как базы данных Oracle или файлы Excel, не поддерживаются.

### <a name="clr"></a>CLR

Управляемый экземпляр SQL не может получить доступ к файловым ресурсам и папкам Windows, поэтому применяются следующие ограничения.

- Поддерживается только `CREATE ASSEMBLY FROM BINARY`. См. раздел [Создание сборки из двоичного файла](/sql/t-sql/statements/create-assembly-transact-sql). 
- `CREATE ASSEMBLY FROM FILE` не поддерживается. См. раздел [Создание сборки из файла](/sql/t-sql/statements/create-assembly-transact-sql).
- `ALTER ASSEMBLY` не может ссылаться на файлы. См. статью [ALTER ASSEMBLY](/sql/t-sql/statements/alter-assembly-transact-sql).

### <a name="database-mail-db_mail"></a>Database Mail (db_mail)
 - `sp_send_dbmail` не удается отправить вложения с помощью @file_attachments параметра. Локальная файловая система и внешние общие папки или хранилище BLOB-объектов Azure недоступны из этой процедуры.
 - Ознакомьтесь с известными проблемами, связанными с `@query` параметрами и проверкой подлинности.
 
### <a name="dbcc"></a>DBCC

Недокументированные инструкции DBCC, включенные в SQL Server, не поддерживаются в SQL Управляемый экземпляр.

- Поддерживается только ограниченное число глобальных флагов трассировки. Уровень сеанса `Trace flags` не поддерживается. См. раздел [Флаги трассировки](/sql/t-sql/database-console-commands/dbcc-traceon-trace-flags-transact-sql).
- [DBCC TRACEOFF](/sql/t-sql/database-console-commands/dbcc-traceoff-transact-sql) и [DBCC TRACEON](/sql/t-sql/database-console-commands/dbcc-traceon-transact-sql) работают с ограниченным числом глобальных флагов трассировки.
- [Инструкция DBCC CHECKDB](/sql/t-sql/database-console-commands/dbcc-checkdb-transact-sql) с параметрами REPAIR_ALLOW_DATA_LOSS, REPAIR_FAST и REPAIR_REBUILD не может быть использована, так как база данных не может быть задана в `SINGLE_USER` режиме. см. [инструкции по ИЗМЕНЕНИЮ отличий базы данных](#alter-database-statement). Потенциальное повреждение базы данных обрабатывается группой поддержки Azure. Если есть какие-либо указания по повреждению базы данных, обратитесь в службу поддержки Azure.

### <a name="distributed-transactions"></a>Распределенные транзакции

Частичная поддержка [распределенных транзакций](../database/elastic-transactions-overview.md) в настоящее время доступна в общедоступной предварительной версии. Поддерживаются следующие сценарии:
* Транзакции, в которых участники являются только управляемыми экземплярами SQL Azure, которые являются частью [группы доверия сервера](./server-trust-group-overview.md).
* Транзакции, инициированные из .NET (класс TransactionScope) и Transact-SQL.

В настоящее время Управляемый экземпляр Azure SQL не поддерживает другие сценарии, которые регулярно поддерживаются локальными службами MSDTC или виртуальными машинами Azure.

### <a name="extended-events"></a>Расширенные события

Некоторые из целевых объектов Windows для расширенных событий (XEvents) не поддерживаются:

- `etw_classic_sync`Целевой объект не поддерживается. Храните `.xel` файлы в хранилище BLOB-объектов Azure. См. раздел [Целевой объект etw_classic_sync_target](/sql/relational-databases/extended-events/targets-for-extended-events-in-sql-server#etw_classic_sync_target-target).
- `event_file`Целевой объект не поддерживается. Храните `.xel` файлы в хранилище BLOB-объектов Azure. См. раздел [Целевой объект event_file](/sql/relational-databases/extended-events/targets-for-extended-events-in-sql-server#event_file-target).

### <a name="external-libraries"></a>Внешние библиотеки

Внешние библиотеки R и Python в базе данных поддерживаются в ограниченном общедоступной предварительной версии. См. статью [службы машинного обучения в управляемый экземпляр SQL Azure (Предварительная версия)](machine-learning-services-overview.md).

### <a name="filestream-and-filetable"></a>Filestream и FileTable

- Данные FILESTREAM не поддерживаются.
- База данных не может содержать файловые группы с `FILESTREAM` данными.
- `FILETABLE` не поддерживается.
- Таблицы не могут содержать типы `FILESTREAM`.
- Следующие функции не поддерживаются:
  - `GetPathLocator()`
  - `GET_FILESTREAM_TRANSACTION_CONTEXT()`
  - `PathName()`
  - `GetFileNamespacePat)`
  - `FileTableRootPath()`

Дополнительные сведения см. в статьях [FILESTREAM (SQL Server)](/sql/relational-databases/blob/filestream-sql-server) и [Таблицы FileTable (SQL Server)](/sql/relational-databases/blob/filetables-sql-server).

### <a name="full-text-semantic-search"></a>Полнотекстовый семантический поиск

[Семантический поиск](/sql/relational-databases/search/semantic-search-sql-server) не поддерживается.

### <a name="linked-servers"></a>Связанные серверы

Связанные серверы в SQL Управляемый экземпляр поддерживают ограниченное количество целевых объектов:

- Поддерживаемые целевые объекты: SQL Управляемый экземпляр, база данных SQL, несинапсеный и выделенный пул SQL [Server](https://devblogs.microsoft.com/azure-sql/linked-server-to-synapse-sql-to-implement-polybase-like-scenarios-in-managed-instance/) , а SQL Server экземпляры. 
- Распределенные транзакции с возможностью записи возможны только между управляемыми экземплярами. Дополнительные сведения см. в разделе [распределенные транзакции](../database/elastic-transactions-overview.md). Однако MS DTC не поддерживается.
- Неподдерживаемые целевые объекты — это файлы, Analysis Services и другие РСУБД. Попробуйте использовать собственный импорт CSV из хранилища BLOB-объектов Azure с помощью `BULK INSERT` или `OPENROWSET` в качестве альтернативы для импорта файлов или загрузки файлов с помощью [бессерверного пула SQL в Azure синапсе Analytics](https://devblogs.microsoft.com/azure-sql/linked-server-to-synapse-sql-to-implement-polybase-like-scenarios-in-managed-instance/).

Операции: 

- Транзакции записи [между экземплярами](../database/elastic-transactions-overview.md) поддерживаются только для управляемых экземпляров.
- `sp_dropserver` поддерживается для удаления связанного сервера. См. статью [sp_dropserver (Transact-SQL)](/sql/relational-databases/system-stored-procedures/sp-dropserver-transact-sql).
- `OPENROWSET`Функцию можно использовать для выполнения запросов только на экземплярах SQL Server. Они могут быть как управляемыми, так и локальными, так и виртуальными машинами. См. статью [OPENROWSET (Transact-SQL)](/sql/t-sql/functions/openrowset-transact-sql).
- `OPENDATASOURCE`Функцию можно использовать для выполнения запросов только на экземплярах SQL Server. Они могут быть как управляемыми, так и локальными, так и виртуальными машинами. В `SQLNCLI` `SQLNCLI11` `SQLOLEDB` качестве поставщика поддерживаются только значения, и. Например, `SELECT * FROM OPENDATASOURCE('SQLNCLI', '...').AdventureWorks2012.HumanResources.Employee`. См. статью [OPENDATASOURCE (Transact-SQL)](/sql/t-sql/functions/opendatasource-transact-sql).
- Связанные серверы нельзя использовать для чтения файлов (Excel, CSV) из общих сетевых папок. Попробуйте использовать [BULK INSERT](/sql/t-sql/statements/bulk-insert-transact-sql#e-importing-data-from-a-csv-file), [OPENROWSET](/sql/t-sql/functions/openrowset-transact-sql#g-accessing-data-from-a-csv-file-with-a-format-file) , считывающие CSV-файлы из хранилища BLOB-объектов Azure, или [связанный сервер, который ссылается на серверный пул SQL в синапсе Analytics](https://devblogs.microsoft.com/azure-sql/linked-server-to-synapse-sql-to-implement-polybase-like-scenarios-in-managed-instance/). Отслеживание запросов в [элементе отзыва SQL управляемый экземпляр](https://feedback.azure.com/forums/915676-sql-managed-instance/suggestions/35657887-linked-server-to-non-sql-sources)|

### <a name="polybase"></a>PolyBase

Единственным доступным типом внешних источников является РСУБД (в общедоступной предварительной версии) для базы данных SQL Azure, управляемый экземпляр SQL Azure и пул синапсе Azure. [Внешнюю таблицу, ссылающуюся на серверный пул SQL в синапсе Analytics](https://devblogs.microsoft.com/azure-sql/read-azure-storage-files-using-synapse-sql-external-tables/) , можно использовать в качестве обходного пути для внешних таблиц polybase, которые напрямую считывают данные из службы хранилища Azure. В управляемом экземпляре SQL Azure связанные серверы можно использовать для [бессерверного пула SQL в синапсе Analytics](https://devblogs.microsoft.com/azure-sql/linked-server-to-synapse-sql-to-implement-polybase-like-scenarios-in-managed-instance/) или SQL Server для чтения данных службы хранилища Azure.
Дополнительные сведения о Polybase см. в разделе [polybase](/sql/relational-databases/polybase/polybase-guide).

### <a name="replication"></a>Репликация

- Поддерживаются типы моментальных снимков и двунаправленной репликации. Репликация слиянием, одноранговая репликация и обновляемые подписки не поддерживаются.
- [Репликация транзакций](replication-transactional-overview.md) доступна для общедоступной предварительной версии на SQL управляемый экземпляр с некоторыми ограничениями:
    - Все типы участников репликации (издатель, распространитель, подписчик по запросу и подписчик push-уведомлений) можно разместить на Управляемый экземпляр SQL, но издатель и распространитель должны быть либо в облаке, либо в локальной среде.
    - SQL Управляемый экземпляр может взаимодействовать с последними версиями SQL Server. Дополнительные сведения см. в [матрице поддерживаемых версий](replication-transactional-overview.md#supportability-matrix) .
    - Репликация транзакций имеет некоторые [Дополнительные требования к сети](replication-transactional-overview.md#requirements).

Дополнительные сведения о настройке репликации транзакций см. в следующих учебниках:
- [Репликация между издателем SQL MI и подписчиком SQL MI](replication-between-two-instances-configure-tutorial.md)
- [Репликация между издателем SQL MI, распространителем SQL MI и SQL Server подписчиком](replication-two-instances-and-sql-server-configure-tutorial.md)

### <a name="restore-statement"></a>Инструкция RESTORE 

- Поддерживаемый синтаксис:
  - `RESTORE DATABASE`
  - `RESTORE FILELISTONLY ONLY`
  - `RESTORE HEADER ONLY`
  - `RESTORE LABELONLY ONLY`
  - `RESTORE VERIFYONLY ONLY`
- Неподдерживаемый синтаксис:
  - `RESTORE LOG ONLY`
  - `RESTORE REWINDONLY ONLY`
- Источник: 
  - `FROM URL` (Хранилище BLOB-объектов Azure) является единственным поддерживаемым вариантом.
  - `FROM DISK`/`TAPE` или устройство резервного копирования не поддерживаются.
  - Резервные наборы данных не поддерживаются.
- `WITH` параметры не поддерживаются. Попытки восстановления, включая `WITH` Like `DIFFERENTIAL` , `STATS` , `REPLACE` и т. д., завершатся ошибкой.
- `ASYNC RESTORE`. Восстановление продолжится, даже если соединение с клиентом будет прервано. Если подключение разорвано, можно проверить `sys.dm_operation_status` состояние операции восстановления, а также для создания и удаления базы данных. См. статью [sys.dm_operation_status (база данных SQL Azure)](/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database). 

Следующие параметры базы данных задаются или переопределяются, и их нельзя изменить позже: 

- `NEW_BROKER` значение, если брокер не включен в bak. 
- `ENABLE_BROKER` значение, если брокер не включен в bak. 
- `AUTO_CLOSE=OFF` Если база данных в BAK-файле имеет значение `AUTO_CLOSE=ON` . 
- `RECOVERY FULL` значение, если база данных в BAK-файле `SIMPLE` имеет `BULK_LOGGED` режим восстановления или.
- Файловая группа, оптимизированная для памяти, добавляется и называется XTP, если она не находилась в файле source. bak. 
- Любая существующая файловая группа, оптимизированная для памяти, переименовывается в XTP. 
- `SINGLE_USER``RESTRICTED_USER`Параметры и преобразуются в `MULTI_USER` .

Ограничения 

- Резервные копии поврежденных баз данных могут быть восстановлены в зависимости от типа повреждения, но автоматические резервные копии не будут выполняться до устранения повреждений. Убедитесь, что вы `DBCC CHECKDB` используете исходный SQL управляемый экземпляр и использовали резервное копирование, `WITH CHECKSUM` чтобы избежать этой проблемы.
- Восстановление `.BAK` файла базы данных, содержащего все ограничения, описанные в этом документе (например, `FILESTREAM` или `FILETABLE` объекты), не может быть восстановлено в управляемый экземпляр SQL.
- `.BAK` невозможно восстановить файлы, содержащие несколько резервных наборов данных. 
- `.BAK` невозможно восстановить файлы, содержащие несколько файлов журнала.
- Резервные копии, содержащие базы данных размером более 8 ТБ, активные объекты OLTP в памяти или количество файлов, размер которых превышает 280 файлов на экземпляр, не могут быть восстановлены на экземпляре общего назначения. 
- Резервные копии, содержащие базы данных размером более 4 ТБ или объекты OLTP в памяти с общим размером, превышающим размер, описанный в разделе [ограничения ресурсов](resource-limits.md) , не могут быть восстановлены в критически важный для бизнеса экземпляре.
Дополнительные сведения о инструкциях RESTORE см. в разделе [инструкции RESTORE](/sql/t-sql/statements/restore-statements-transact-sql).

 > [!IMPORTANT]
 > Те же ограничения применяются ко встроенной операции восстановления на момент времени. Например, база данных общего назначения более 4 ТБ не может быть восстановлена в экземпляре критически важный для бизнеса. Критически важный для бизнеса базу данных с файлами в памяти OLTP или более 280 файлов нельзя восстановить в экземпляре общего назначения.

### <a name="service-broker"></a>Service Broker

Обмен сообщениями компонента Service Broker между экземплярами поддерживается только между управляемыми экземплярами SQL Azure:

- `CREATE ROUTE`: Нельзя использовать `CREATE ROUTE` , кроме `ADDRESS` `LOCAL` или DNS-имени другого управляемый экземпляр SQL.
- `ALTER ROUTE`: Нельзя использовать `ALTER ROUTE` , кроме `ADDRESS` `LOCAL` или DNS-имени другого управляемый экземпляр SQL.

Безопасность транспорта поддерживается, безопасность диалога не является:
- `CREATE REMOTE SERVICE BINDING` не поддерживается.

Компонент Service Broker включен по умолчанию и не может быть отключен. Следующие параметры ALTER базе данных не поддерживаются:
- `ENABLE_BROKER`
- `DISABLE_BROKER`

### <a name="stored-procedures-functions-and-triggers"></a>Хранимые процедуры, функции и триггеры

- `NATIVE_COMPILATION` не поддерживается на уровне общего назначения.
- Следующие параметры [sp_configure](/sql/relational-databases/system-stored-procedures/sp-configure-transact-sql) не поддерживаются: 
  - `allow polybase export`
  - `allow updates`
  - `filestream_access_level`
  - `remote access`
  - `remote data archive`
  - `remote proc trans`
  - `scan for startup procs`
- `sp_execute_external_scripts` не поддерживается. См. раздел [Примеры](/sql/relational-databases/system-stored-procedures/sp-execute-external-script-transact-sql#examples).
- `xp_cmdshell` не поддерживается. См. раздел [xp_cmdshell (Transact-SQL)](/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql).
- `Extended stored procedures` не поддерживаются, включая `sp_addextendedproc` и `sp_dropextendedproc` . Эта функция не будет поддерживаться, так как она находится на пути устаревания для SQL Server. Дополнительные сведения см. в разделе [Расширенные хранимые процедуры](/sql/relational-databases/extended-stored-procedures-programming/database-engine-extended-stored-procedures-programming).
- `sp_attach_db`, `sp_attach_single_file_db` и `sp_detach_db` не поддерживаются. См. статьи [sp_attach_db (Transact-SQL)](/sql/relational-databases/system-stored-procedures/sp-attach-db-transact-sql), [sp_attach_single_file_db (Transact-SQL)](/sql/relational-databases/system-stored-procedures/sp-attach-single-file-db-transact-sql) и [sp_detach_db (Transact-SQL)](/sql/relational-databases/system-stored-procedures/sp-detach-db-transact-sql).

### <a name="system-functions-and-variables"></a>Системные функции и переменные

Следующие переменные, функции и представления возвращают различные результаты:

- `SERVERPROPERTY('EngineEdition')` Возвращает значение 8. Это свойство однозначно определяет Управляемый экземпляр SQL. См. статью [SERVERPROPERTY (Transact-SQL)](/sql/t-sql/functions/serverproperty-transact-sql).
- `SERVERPROPERTY('InstanceName')` Возвращает значение NULL, поскольку понятие экземпляра в том виде, в котором оно существует, SQL Server не применяется к SQL Управляемый экземпляр. См. статью [SERVERPROPERTY (Transact-SQL)](/sql/t-sql/functions/serverproperty-transact-sql).
- `@@SERVERNAME` Возвращает полное DNS-имя "Connected" (например, my-managed-instance.wcus17662feb9ce98.database.windows.net). См. раздел [@ @SERVERNAME ](/sql/t-sql/functions/servername-transact-sql). 
- `SYS.SERVERS` Возвращает полное имя "Connected" DNS, например `myinstance.domain.database.windows.net` для свойств "Name" и "data_source". См. статью [sys.servers (Transact-SQL)](/sql/relational-databases/system-catalog-views/sys-servers-transact-sql).
- `@@SERVICENAME` Возвращает значение NULL, поскольку концепция службы в том, что она существует для SQL Server не применяется к SQL Управляемый экземпляр. См. раздел [@ @SERVICENAME ](/sql/t-sql/functions/servicename-transact-sql).
- `SUSER_ID` поддерживается. Он возвращает значение NULL, если имя входа Azure AD не входит в sys.sysимена входа. См. статью [Идентификатор SUSER_ID (Transact-SQL)](/sql/t-sql/functions/suser-id-transact-sql). 
- `SUSER_SID` не поддерживается. Возвращаются неверные данные, что является временной известной проблемой. См. статью [SUSER_SID (Transact-SQL)](/sql/t-sql/functions/suser-sid-transact-sql). 

## <a name="environment-constraints"></a><a name="Environment"></a>Ограничения среды

### <a name="subnet"></a>Подсеть
-  Другие ресурсы (например, виртуальные машины) нельзя поместить в подсеть, в которой развернута Управляемый экземпляр SQL. Разверните эти ресурсы с помощью другой подсети.
- Подсеть должна иметь достаточное количество доступных [IP-адресов](connectivity-architecture-overview.md#network-requirements). Минимальное значение — включать по крайней мере 32 IP-адресов в подсети.
- Количество виртуальных ядер и типов экземпляров, которые можно развернуть в регионе, имеет некоторые [ограничения и](resource-limits.md#regional-resource-limitations)ограничения.
- Существует [Сетевая конфигурация](connectivity-architecture-overview.md#network-requirements) , которую необходимо применить к подсети.

### <a name="vnet"></a>Виртуальная сеть
- Виртуальную сеть можно развернуть с помощью модели ресурсов — классическая модель для виртуальной сети не поддерживается.
- После создания Управляемый экземпляр SQL не поддерживается перемещение Управляемый экземпляр SQL или виртуальной сети в другую группу ресурсов или подписку.
- Для управляемых экземпляров SQL, размещенных в виртуальных кластерах, созданных до 9/22/2020 [глобального пиринга](../../virtual-network/virtual-networks-faq.md#what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers) , не поддерживается. Вы можете подключаться к этим ресурсам через ExpressRoute или VNet-VNet через шлюзы виртуальной сети.

### <a name="failover-groups"></a>Группы отработки отказа
Системные базы данных не реплицируются на дополнительный экземпляр в группе отработки отказа. Таким образом, сценарии, зависящие от объектов системных баз данных, будут невозможны на вторичном экземпляре, если только эти объекты не будут созданы вручную на сервере-получателе.

### <a name="tempdb"></a>БАЗЕ
- Максимальный размер файла `tempdb` не может быть больше 24 ГБ на ядро на уровне общего назначения. Максимальный `tempdb` размер критически важный для бизнеса уровня ограничен размером хранилища SQL управляемый экземпляр. `Tempdb` Размер файла журнала ограничен 120 ГБ на уровне общего назначения. Некоторые запросы могут возвращать ошибку, если им требуется более 24 ГБ на ядро в `tempdb` или если они создают более 120 ГБ данных журнала.
- `Tempdb` всегда разбивается на 12 файлов данных: 1 первичный, также называемый главным, файлом данных и 11 не-первичными файлами данных. Невозможно изменить структуру файла, и добавить новые файлы в невозможно `tempdb` . 
- [ `tempdb` Метаданные, оптимизированные для памяти](/sql/relational-databases/databases/tempdb-database?view=sql-server-ver15#memory-optimized-tempdb-metadata), — Новая функция базы данных SQL Server 2019 в памяти, не поддерживается.
- Объекты, созданные в базе данных model, не могут быть созданы при `tempdb` перезапуске или отработке отказа, поскольку не `tempdb` получают исходный список объектов из базы данных model. Объекты необходимо создавать в `tempdb` ручном режиме после каждой перезагрузки или отработки отказа.

### <a name="msdb"></a>MSDB

Следующие схемы MSDB в SQL Управляемый экземпляр должны принадлежать соответствующим заранее определенным ролям:

- Общие роли
  - Роли TargetServersRole
- [Предопределенные роли базы данных](/sql/ssms/agent/sql-server-agent-fixed-database-roles?view=sql-server-ver15)
  - SQLAgentUserRole
  - SQLAgentReaderRole
  - SQLAgentOperatorRole
- [DatabaseMail роли](/sql/relational-databases/database-mail/database-mail-configuration-objects?view=sql-server-ver15#DBProfile):
  - Член
- [Роли служб Integration Services](/sql/integration-services/security/integration-services-roles-ssis-service?view=sql-server-ver15):
  - msdb
  - db_ssisltduser
  - db_ssisoperator
  
> [!IMPORTANT]
> Изменение стандартных имен ролей, имен схем и владельцев схем клиентами влияет на нормальную работу службы. Любые изменения, внесенные в эти данные, будут возвращены к заранее заданным значениям, как только обнаруженные, или по следующему обновлению службы в последней версии, чтобы обеспечить нормальную работу службы.

### <a name="error-logs"></a>Журналы ошибок

SQL Управляемый экземпляр помещает подробные сведения в журналы ошибок. Существует множество внутренних системных событий, которые регистрируются в журнале ошибок. Используйте пользовательскую процедуру для чтения журналов ошибок, которые отфильтровывают некоторые ненужные записи. Дополнительные сведения см. в разделе [sql управляемый экземпляр – sp_readmierrorlog](/archive/blogs/sqlcat/azure-sql-db-managed-instance-sp_readmierrorlog) или [расширение SQL управляемый экземпляр (Предварительная версия)](/sql/azure-data-studio/azure-sql-managed-instance-extension#logs) для Azure Data Studio.

## <a name="next-steps"></a>Следующие шаги

- Дополнительные сведения о SQL Управляемый экземпляр см. в статье [что такое sql управляемый экземпляр?](sql-managed-instance-paas-overview.md)
- Список функций и сравнительных списков см. в статье [Сравнение функций управляемый экземпляр SQL Azure](../database/features-comparison.md).
- Сведения о состоянии обновлений выпуска и известных проблем см. в разделе [заметки о выпуске SQL управляемый экземпляр](../database/doc-changes-updates-release-notes.md)
- Краткое руководство, в котором показано, как создать новый Управляемый экземпляр SQL, см. в разделе [создание управляемый экземпляр SQL](instance-create-quickstart.md).
