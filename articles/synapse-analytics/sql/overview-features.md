---
title: Отличия функций T-SQL в Synapse SQL
description: Список функций Transact-SQL, доступных в Synapse SQL.
services: synapse analytics
author: jovanpop-msft
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/15/2020
ms.author: jovanpop
ms.reviewer: jrasnick
ms.openlocfilehash: 55639a8d36000af00e23391f39e9e1c7364c8b71
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106076433"
---
# <a name="transact-sql-features-supported-in-azure-synapse-sql"></a>Функции T-SQL, которые поддерживаются в Azure Synapse SQL

Azure Synapse SQL — это служба аналитики больших данных, которая позволяет выполнять запросы и анализ данных с помощью языка T-SQL. Вы можете использовать для анализа данных диалект SQL, соответствующий стандарту ANSI, который используется в SQL Server и Базе данных SQL Azure. 

Язык Transact-SQL используется в бессерверном пуле SQL, а выделенная модель может ссылаться на разные объекты и имеет некоторые отличия в наборе поддерживаемых функций. На этой странице перечислены основные отличия языка Transact-SQL в разных моделях потребления Synapse SQL.

## <a name="database-objects"></a>Объекты базы данных

Модели потребления в Synapse SQL позволяют использовать разные объекты базы данных. В следующей таблице представлено сравнение поддерживаемых типов объектов.

|   | Выделенные | Бессерверные приложения |
| --- | --- | --- |
| **Таблицы** | [Да](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?view=azure-sqldw-latest&preserve-view=true) | Нет, бессерверная модель может запрашивать только внешние данные, размещенные в [службе хранилище Azure](#storage-options) |
| **Представления** | [Да.](/sql/t-sql/statements/create-view-transact-sql?view=azure-sqldw-latest&preserve-view=true) Представления могут использовать [элементы языка запросов](#query-language), доступные в выделенной модели. | [Да.](/sql/t-sql/statements/create-view-transact-sql?view=azure-sqldw-latest&preserve-view=true) Представления могут использовать [элементы языка запросов](#query-language), доступные в бессерверной модели. |
| **Схемы** | [Да](/sql/t-sql/statements/create-schema-transact-sql?view=azure-sqldw-latest&preserve-view=true) | [Да](/sql/t-sql/statements/create-schema-transact-sql?view=azure-sqldw-latest&preserve-view=true) |
| **Временные таблицы** | [Да](../sql-data-warehouse/sql-data-warehouse-tables-temporary.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) | нет |
| **Процедуры** | [Да](/sql/t-sql/statements/create-procedure-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Да |
| **Функции** | [Да](/sql/t-sql/statements/create-function-sql-data-warehouse?view=azure-sqldw-latest&preserve-view=true) | Да. Только встроенные функции с табличным значением. |
| **Триггеры** | нет | нет |
| **Внешние таблицы** | [Да.](/sql/t-sql/statements/create-external-table-transact-sql?view=azure-sqldw-latest&preserve-view=true) См. поддерживаемые [форматы данных](#data-formats). | [Да.](/sql/t-sql/statements/create-external-table-transact-sql?view=azure-sqldw-latest&preserve-view=true) См. поддерживаемые [форматы данных](#data-formats). |
| **Кэширование запросов** | Да, несколько форм (кэширование на твердотельных накопителях, кэширование в памяти, кэширование ResultSet). Кроме того, поддерживаются материализованные представления | нет |
| **Табличные переменные** | [Нет](/sql/t-sql/data-types/table-transact-sql?view=azure-sqldw-latest&preserve-view=true), используйте временные таблицы | нет |
| **[Распределение таблиц](../sql-data-warehouse/sql-data-warehouse-tables-distribute.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)**               | Да | нет |
| **[Индексы таблиц](../sql-data-warehouse/sql-data-warehouse-tables-index.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)**                           | Да | нет |
| **[Разделы таблиц](../sql-data-warehouse/sql-data-warehouse-tables-partition.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)**                     | Да | нет |
| **[Статистика](develop-tables-statistics.md)**            | Да | Да |
| **[Управление рабочими нагрузками, классы ресурсов и контроль параллелизма](../sql-data-warehouse/resource-classes-for-workload-management.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)** | Да    | Нет |
| **Управление затратами** | Да, при использовании действий по вертикальному масштабированию. | Да, при использовании [портала Azure или процедуры T-SQL](./data-processed.md#cost-control). |

## <a name="query-language"></a>Язык запросов

Языки запросов, используемые в Synapse SQL, могут поддерживать разный набор функций в зависимости от модели потребления. В следующей таблице приведены наиболее важные отличия в диалектах языка запросов для Transact-SQL:

|   | Выделенные | Бессерверные приложения |
| --- | --- | --- |
| **Инструкция SELECT** | Да. Не поддерживаются предложения запроса Transact-SQL [FOR XML/FOR JSON](/sql/t-sql/queries/select-for-clause-transact-sql?view=azure-sqldw-latest&preserve-view=true), [MATCH](/sql/t-sql/queries/match-sql-graph?view=azure-sqldw-latest&preserve-view=true) и OFFSET/FETCH. | Да. Не поддерживаются предложения запроса Transact-SQL [FOR XML](/sql/t-sql/queries/select-for-clause-transact-sql?view=azure-sqldw-latest&preserve-view=true), [MATCH](/sql/t-sql/queries/match-sql-graph?view=azure-sqldw-latest&preserve-view=true) и [PREDICT](/sql/t-sql/queries/predict-transact-sql?view=azure-sqldw-latest&preserve-view=true), GROUPNG SETS, а также указания запроса. |
| **Инструкция INSERT** | Да | нет |
| **Инструкция UPDATE** | Да | нет |
| **Инструкция DELETE** | Да | нет |
| **Инструкция MERGE** | Да ([предварительная версия](/sql/t-sql/statements/merge-transact-sql?view=azure-sqldw-latest&preserve-view=true)) | Нет |
| **[Транзакции](develop-transactions.md)** | Да | Да, применяются к объектам meta-data. |
| **[Метки](develop-label.md)** | Да | нет |
| **Загрузка данных** | Да. Лучше всего использовать инструкцию [COPY](/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest&preserve-view=true), но система поддерживает для загрузки данных как массовую загрузку (BCP), так и [CETAS](/sql/t-sql/statements/create-external-table-as-select-transact-sql?view=azure-sqldw-latest&preserve-view=true). | нет |
| **Экспорт данных** | Да. Использование [CETAS](/sql/t-sql/statements/create-external-table-as-select-transact-sql?view=azure-sqldw-latest&preserve-view=true). | Да. Использование [CETAS](/sql/t-sql/statements/create-external-table-as-select-transact-sql?view=azure-sqldw-latest&preserve-view=true). |
| **Типы** | Да, все типы Transact-SQL, за исключением [cursor](/sql/t-sql/data-types/cursor-transact-sql?view=azure-sqldw-latest&preserve-view=true), [hierarchyid](/sql/t-sql/data-types/hierarchyid-data-type-method-reference?view=azure-sqldw-latest&preserve-view=true), [ntext, text и image](/sql/t-sql/data-types/ntext-text-and-image-transact-sql?view=azure-sqldw-latest&preserve-view=true), [rowversion](/sql/t-sql/data-types/rowversion-transact-sql?view=azure-sqldw-latest&preserve-view=true), [пространственных типов](/sql/t-sql/spatial-geometry/spatial-types-geometry-transact-sql?view=azure-sqldw-latest&preserve-view=true), [sql\_variant](/sql/t-sql/data-types/sql-variant-transact-sql?view=azure-sqldw-latest&preserve-view=true) и [xml](/sql/t-sql/xml/xml-transact-sql?view=azure-sqldw-latest&preserve-view=true). | Да, все типы Transact-SQL, за исключением [cursor](/sql/t-sql/data-types/cursor-transact-sql?view=azure-sqldw-latest&preserve-view=true), [hierarchyid](/sql/t-sql/data-types/hierarchyid-data-type-method-reference?view=azure-sqldw-latest&preserve-view=true), [ntext, text и image](/sql/t-sql/data-types/ntext-text-and-image-transact-sql?view=azure-sqldw-latest&preserve-view=true), [rowversion](/sql/t-sql/data-types/rowversion-transact-sql?view=azure-sqldw-latest&preserve-view=true), [пространственных типов](/sql/t-sql/spatial-geometry/spatial-types-geometry-transact-sql?view=azure-sqldw-latest&preserve-view=true), [sql\_variant](/sql/t-sql/data-types/sql-variant-transact-sql?view=azure-sqldw-latest&preserve-view=true), [xml](/sql/t-sql/xml/xml-transact-sql?view=azure-sqldw-latest&preserve-view=true) и табличного типа. |
| **Запросы баз данных** | нет | Да, включая инструкции [USE](/sql/t-sql/language-elements/use-transact-sql?view=azure-sqldw-latest&preserve-view=true). |
| **Встроенные функции (анализ)** | Да, все функции [аналитики](/sql/t-sql/functions/analytic-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true), преобразования, [даты и времени](/sql/t-sql/functions/date-and-time-data-types-and-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true), логические и [математические](/sql/t-sql/functions/mathematical-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true) функции из Transact-SQL, кроме [CHOOSE](/sql/t-sql/functions/logical-functions-choose-transact-sql?view=azure-sqldw-latest&preserve-view=true) и [PARSE](/sql/t-sql/functions/parse-transact-sql?view=azure-sqldw-latest&preserve-view=true). | Да, все функции [аналитики](/sql/t-sql/functions/analytic-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true), преобразования, [даты и времени](/sql/t-sql/functions/date-and-time-data-types-and-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true), логические и [математические](/sql/t-sql/functions/mathematical-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true) функции из Transact-SQL. |
| **Встроенные функции ([строка](https://docs.microsoft.com/sql/t-sql/functions/string-functions-transact-sql))** | Да. Все функции Transact-SQL для работы со [строками](/sql/t-sql/functions/string-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true), [JSON](/sql/t-sql/functions/json-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true) и параметрами сортировки, кроме [STRING_ESCAPE](/sql/t-sql/functions/string-escape-transact-sql?view=azure-sqldw-latest&preserve-view=true) и [TRANSLATE](/sql/t-sql/functions/translate-transact-sql?view=azure-sqldw-latest&preserve-view=true). | Да. Все функции Transact-SQL для работы со [строками](/sql/t-sql/functions/string-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true), [JSON](/sql/t-sql/functions/json-functions-transact-sql?view=azure-sqldw-latest&preserve-view=true) и параметрами сортировки. |
| **Встроенные функции ([криптографические](https://docs.microsoft.com/sql/t-sql/functions/cryptographic-functions-transact-sql))** | Некотор. | нет |
| **Встроенные функции с табличным значением** | Да, все [функции наборов строк в Transact-SQL](/sql/t-sql/functions/functions?view=azure-sqldw-latest&preserve-view=true#rowset-functions), кроме [OPENXML](/sql/t-sql/functions/openxml-transact-sql?view=azure-sqldw-latest&preserve-view=true), [OPENDATASOURCE](/sql/t-sql/functions/opendatasource-transact-sql?view=azure-sqldw-latest&preserve-view=true), [OPENQUERY](/sql/t-sql/functions/openquery-transact-sql?view=azure-sqldw-latest&preserve-view=true) и [OPENROWSET](/sql/t-sql/functions/openrowset-transact-sql?view=azure-sqldw-latest&preserve-view=true). | Да, все [функции наборов строк в Transact-SQL](/sql/t-sql/functions/functions?view=azure-sqldw-latest&preserve-view=true#rowset-functions), кроме [OPENXML](/sql/t-sql/functions/openxml-transact-sql?view=azure-sqldw-latest&preserve-view=true), [OPENDATASOURCE](/sql/t-sql/functions/opendatasource-transact-sql?view=azure-sqldw-latest&preserve-view=true) и [OPENQUERY](/sql/t-sql/functions/openquery-transact-sql?view=azure-sqldw-latest&preserve-view=true).  |
| **Статистические выражения** |  Все встроенные статистические функции Transact-SQL, за исключением [CHECKSUM_AGG](/sql/t-sql/functions/checksum-agg-transact-sql?view=azure-sqldw-latest&preserve-view=true) и [GROUPING_ID](/sql/t-sql/functions/grouping-id-transact-sql?view=azure-sqldw-latest&preserve-view=true). | Все встроенные статистические функции Transact-SQL. |
| **Операторы** | Да, все [операторы Transact-SQL](/sql/t-sql/language-elements/operators-transact-sql?view=azure-sqldw-latest&preserve-view=true), кроме [!>](/sql/t-sql/language-elements/not-greater-than-transact-sql?view=azure-sqldw-latest&preserve-view=true) и [!<](/sql/t-sql/language-elements/not-less-than-transact-sql?view=azure-sqldw-latest&preserve-view=true). | Да, только [операторы Transact-SQL](/sql/t-sql/language-elements/operators-transact-sql?view=azure-sqldw-latest&preserve-view=true).  |
| **Управление потоком** | Да. Все [инструкции Transact-SQL для управления потоком](/sql/t-sql/language-elements/control-of-flow?view=azure-sqldw-latest&preserve-view=true), за исключением [CONTINUE](/sql/t-sql/language-elements/continue-transact-sql?view=azure-sqldw-latest&preserve-view=true), [GOTO](/sql/t-sql/language-elements/goto-transact-sql?view=azure-sqldw-latest&preserve-view=true), [RETURN](/sql/t-sql/language-elements/return-transact-sql?view=azure-sqldw-latest&preserve-view=true), [USE](/sql/t-sql/language-elements/use-transact-sql?view=azure-sqldw-latest&preserve-view=true) и [WAITFOR](/sql/t-sql/language-elements/waitfor-transact-sql?view=azure-sqldw-latest&preserve-view=true). | Да. Все [инструкции Transact-SQL для управления потоком](/sql/t-sql/language-elements/control-of-flow?view=azure-sqldw-latest&preserve-view=true), за исключением запроса SELECT в условии `WHILE (...)`. |
| **Инструкции DDL (CREATE, ALTER, DROP)** | Да. Все инструкции DDL из Transact-SQL, применимые к поддерживаемым типам объектов. | Да. Все инструкции DDL из Transact-SQL, применимые к поддерживаемым типам объектов. |

## <a name="security"></a>Безопасность

Synapse SQL позволяет использовать встроенные функции безопасности для защиты данных и управления доступом. В следующей таблице перечислены основные различия между моделями потребления в Synapse SQL.

|   | Выделенные | Бессерверные приложения |
| --- | --- | --- |
| **Имена входа** | Недоступно (в базах данных поддерживаются только автономные пользователи). | Да |
| **Пользователи** |  Недоступно (в базах данных поддерживаются только автономные пользователи). | Да |
| **[Автономные пользователи](/sql/relational-databases/security/contained-database-users-making-your-database-portable?view=azure-sqldw-latest&preserve-view=true)** | Да. **Примечание.** Только один пользователь Azure AD может быть администратором с неограниченными правами. | Нет |
| **Аутентификация по имени пользователя и паролю в SQL**| Да | Да |
| **Проверка подлинности через Azure Active Directory (Azure AD)**| Да (для пользователей Azure AD) | Да (для учетных данных и пользователей Azure AD) |
| **Сквозная проверка подлинности в службе хранилища Azure Active Directory (Azure AD)** | Да | Да |
| **Аутентификация по маркерам SAS для службы хранилища** | нет | Да, с использованием [DATABASE SCOPED CREDENTIAL](/sql/t-sql/statements/create-database-scoped-credential-transact-sql?view=azure-sqldw-latest&preserve-view=true) в [EXTERNAL DATA SOURCE](/sql/t-sql/statements/create-external-data-source-transact-sql?view=azure-sqldw-latest&preserve-view=true) или [CREDENTIAL](/sql/t-sql/statements/create-credential-transact-sql?view=azure-sqldw-latest&preserve-view=true) на уровне объекта. |
| **Проверка подлинности по ключу доступа к хранилищу** | Да, с использованием [DATABASE SCOPED CREDENTIAL](/sql/t-sql/statements/create-database-scoped-credential-transact-sql?view=azure-sqldw-latest&preserve-view=true) в [EXTERNAL DATA SOURCE](/sql/t-sql/statements/create-external-data-source-transact-sql?view=azure-sqldw-latest&preserve-view=true). | Нет |
| **Проверка подлинности по [управляемому удостоверению](../security/synapse-workspace-managed-identity.md) для службы хранилища** | Да, с использованием [управляемого удостоверения службы](../../azure-sql/database/vnet-service-endpoint-rule-overview.md?bc=%2fazure%2fsynapse-analytics%2fbreadcrumb%2ftoc.json&preserve-view=true&toc=%2fazure%2fsynapse-analytics%2ftoc.json&view=azure-sqldw-latest&preserve-view=true). | Да, с использованием учетных данных `Managed Identity`. |
| **Проверка подлинности по удостоверению приложения для службы хранилища** | [Да](/sql/t-sql/statements/create-external-data-source-transact-sql?view=azure-sqldw-latest&preserve-view=true) | нет |
| **Разрешения — на уровне объектов** | Да, включая применение операций GRANT, DENY и REVOKE к разрешениям пользователей. | Да, включая применение операций GRANT, DENY и REVOKE к разрешениям пользователей и (или) имен входа для поддерживаемых системных объектов. |
| **Разрешения — на уровне схемы** | Да, включая применение операций GRANT, DENY и REVOKE к разрешениям пользователей и (или) имен входа для схемы. | Да, включая применение операций GRANT, DENY и REVOKE к разрешениям пользователей и (или) имен входа для схемы. |
| **Разрешения — [на уровне базы данных](/sql/relational-databases/security/authentication-access/database-level-roles?view=azure-sqldw-latest&preserve-view=true)** | Да | Да |
| **Разрешения — [на уровне сервера](/sql/relational-databases/security/authentication-access/server-level-roles)** | нет | Да, поддерживаются sysadmin и другие серверные роли |
| **Разрешения — [защита на уровне столбцов](../sql-data-warehouse/column-level-security.md?bc=%2fazure%2fsynapse-analytics%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fsynapse-analytics%2ftoc.json)** | Да | Да |
| **Роли и группы** | Да (в области базы данных) | Да (в области сервера и базы данных) |
| **Безопасность и функции идентификации** | Некоторые функции и операторы безопасности Transact-SQL: `CURRENT_USER`, `HAS_DBACCESS`, `IS_MEMBER`, `IS_ROLEMEMBER`, `SESSION_USER`, `SUSER_NAME`, `SUSER_SNAME`, `SYSTEM_USER`, `USER`, `USER_NAME`, `EXECUTE AS`, `OPEN/CLOSE MASTER KEY`. | Некоторые функции и операторы безопасности Transact-SQL: `CURRENT_USER`, `HAS_DBACCESS`, `HAS_PERMS_BY_NAME`, `IS_MEMBER', 'IS_ROLEMEMBER`, `IS_SRVROLEMEMBER`, `SESSION_USER`, `SESSION_CONTEXT`, `SUSER_NAME`, `SUSER_SNAME`, `SYSTEM_USER`, `USER`, `USER_NAME`, `EXECUTE AS` и `REVERT`. Функции безопасности нельзя использовать для запроса по внешним данным (сначала сохраните результат в переменной, чтобы использовать ее в запросе).  |
| **DATABASE SCOPED CREDENTIAL** | Да | Да |
| **SERVER SCOPED CREDENTIAL** | Нет | Да |
| **Безопасность на уровне строк** | [Да](/sql/relational-databases/security/row-level-security?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) | Нет |
| **Прозрачное шифрование данных (TDE)** | [Да](../../azure-sql/database/transparent-data-encryption-tde-overview.md) | Нет | 
| **Обнаружение и классификация данных** | [Да](../../azure-sql/database/data-discovery-and-classification-overview.md) | Нет |
| **Оценка уязвимостей** | [Да](../../azure-sql/database/sql-vulnerability-assessment.md) | Нет |
| **Расширенная защита от угроз** | [Да](../../azure-sql/database/threat-detection-overview.md)
| **Аудит** | [Да](../../azure-sql/database/auditing-overview.md) | Нет |
| **[Правила брандмауэра](../security/synapse-workspace-ip-firewall.md)**| Да | Да |
| **[Частная конечная точка](../security/synapse-workspace-managed-private-endpoints.md)**| Да | Да |

Выделенный и бессерверный пулы SQL используют для запрашивания данных стандартный язык Transact-SQL. Подробные сведения о различиях см. в [справочнике по языку Transact-SQL](/sql/t-sql/language-reference).

## <a name="tools"></a>Инструменты

Вы можете использовать разные средства, чтобы подключиться к Synapse SQL для запроса по данным.

|   | Выделенные | Бессерверные приложения |
| --- | --- | --- |
| **Synapse Studio** | Да, скрипты SQL. | Да, скрипты SQL. |
| **Power BI** | Да | [Да](tutorial-connect-power-bi-desktop.md) |
| **Служба Azure Analysis Services** | Да | Да |
| **Azure Data Studio** | Да | Да, версия 1.18.0 или более поздняя. Поддерживаются скрипты SQL и записные книжки SQL. |
| **Среда SQL Server Management Studio** | Да | Да, версия 18.5 или более поздняя. |

> [!NOTE]
> С помощью SSMS можно подключиться и отправить запрос к бессерверному пулу SQL. Эти средства частично поддерживаются, начиная с версии 18.5, и используется только для подключения и выполнения запросов.

Большинство приложений используют стандартный язык Transact-SQL, который может выполнять запросы по выделенным и бессерверным моделям потребления в Synapse SQL.

## <a name="storage-options"></a>Варианты хранилищ

Анализируемые данные могут храниться в разных типах хранилищ. Все доступные варианты хранилищ перечислены в следующей таблице.

|   | Выделенные | Бессерверные приложения |
| --- | --- | --- |
| **Внутреннее хранилище** | Да | нет |
| **Azure Data Lake версии 2** | Да | Да |
| **Хранилище BLOB-объектов Azure** | Да | Да |
| **Azure SQL (удаленный репозиторий)** | Нет | Нет |
| **Транзакционное хранилище Azure CosmosDB** | Нет | Нет |
| **Аналитическое хранилище Azure Cosmos DB** | Нет | Да, при использовании [Synapse Link (предварительная версия)](../../cosmos-db/synapse-link.md?bc=%2fazure%2fsynapse-analytics%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fsynapse-analytics%2ftoc.json) ([общедоступная предварительная версия](../../cosmos-db/synapse-link.md?bc=%2fazure%2fsynapse-analytics%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fsynapse-analytics%2ftoc.json#limitations)) |
| **Таблицы Apache Spark (в рабочей области)** | Нет | Таблицы PARQUET только при использовании [синхронизации метаданных](develop-storage-files-spark-tables.md) |
| **Таблицы Apache Spark (удаленный репозиторий)** | Нет | Нет |
| **Таблицы Databricks (удаленный репозиторий)** | Нет | Нет |

## <a name="data-formats"></a>Форматы данных

Анализируемые данные могут храниться в разных форматах. Все доступные для анализа варианты форматов перечислены в следующей таблице.

|   | Выделенные | Бессерверные приложения |
| --- | --- | --- |
| **С разделителями** | [Да](/sql/t-sql/statements/create-external-file-format-transact-sql?view=azure-sqldw-latest&preserve-view=true) | [Да](query-single-csv-file.md) |
| **CSV** | Да (многосимвольные разделители не поддерживаются). | [Да](query-single-csv-file.md) |
| **Parquet** | [Да](/sql/t-sql/statements/create-external-file-format-transact-sql?view=azure-sqldw-latest&preserve-view=true) | [Да](query-parquet-files.md), включая файлы с [вложенными типами](query-parquet-nested-types.md). |
| **Hive ORC** | [Да](/sql/t-sql/statements/create-external-file-format-transact-sql?view=azure-sqldw-latest&preserve-view=true) | нет |
| **Hive RC** | [Да](/sql/t-sql/statements/create-external-file-format-transact-sql?view=azure-sqldw-latest&preserve-view=true) | нет |
| **JSON** | Да | [Да](query-json-files.md) |
| **Avro** | Нет | Нет |
| **[Delta-lake](https://delta.io/)** | нет | нет |
| **[CDM](/common-data-model/)** | нет | нет |

## <a name="next-steps"></a>Дальнейшие действия
Рекомендации по работе с выделенным и бессерверным пулами SQL см. в следующих статьях:

- [Рекомендации по использованию выделенного пула SQL](best-practices-dedicated-sql-pool.md)
- [Рекомендации по использованию бессерверного пула SQL](best-practices-serverless-sql-pool.md)
