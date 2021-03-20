---
title: Формат журнала аудита базы данных SQL
description: Узнайте, как структурированы журналы аудита базы данных SQL Azure.
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.topic: reference
author: DavidTrigano
ms.author: datrigan
ms.reviewer: vanto
ms.custom: sqldbrb=1
ms.date: 06/03/2020
ms.openlocfilehash: f5c176db4f679c79bb42c6ceb46b3588e9440874
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100572222"
---
# <a name="sql-database-audit-log-format"></a>Формат журнала аудита базы данных SQL

[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

[Аудит базы данных SQL Azure](auditing-overview.md) отслеживает события базы данных и записывает их в журнал аудита в учетной записи хранения Azure или отправляет их в концентратор событий или log Analytics для последующей обработки и анализа.

## <a name="naming-conventions"></a>Соглашения об именах

### <a name="blob-audit"></a>Аудит больших двоичных объектов

Журналы аудита, хранящиеся в хранилище BLOB-объектов Azure, хранятся в контейнере с именем `sqldbauditlogs` в учетной записи хранения Azure. Иерархия каталогов в контейнере имеет форму `<ServerName>/<DatabaseName>/<AuditName>/<Date>/` . Формат имени файла большого двоичного объекта — `<CreationTime>_<FileNumberInSession>.xel` , где `CreationTime` находится в `hh_mm_ss_ms` формате UTC, а `FileNumberInSession` — это выполняемый индекс в случае, если журналы сеансов охватывают несколько файлов большого двоичного объекта.

Например, для базы данных `Database1` `Server1` можно использовать следующий путь:

`Server1/Database1/SqlDbAuditing_ServerAudit_NoRetention/2019-02-03/12_23_30_794_0.xel`

Журналы аудита [реплик только для чтения](read-scale-out.md) хранятся в том же контейнере. Иерархия каталогов в контейнере имеет форму `<ServerName>/<DatabaseName>/<AuditName>/<Date>/RO/` . Имя файла большого двоичного объекта имеет общий формат. Журналы аудита реплик только для чтения хранятся в том же контейнере.


### <a name="event-hub"></a>Концентратор событий

События аудита записываются в пространство имен и концентратор событий, которые были определены во время настройки аудита. Они фиксируются в теле событий [Apache Avro](https://avro.apache.org/) и сохраняются с использованием форматирования JSON в кодировке UTF-8. Для чтения журналов аудита можете использовать [средства Avro](../../event-hubs/event-hubs-capture-overview.md#use-avro-tools) или похожие средства, которые поддерживают данный формат.

### <a name="log-analytics"></a>Log Analytics

События аудита записываются в рабочую область Log Analytics, определенную во время настройки аудита, в таблицу `AzureDiagnostics` с категорией `SQLSecurityAuditEvents`. Дополнительную полезную информацию о языке поиска и командах Log Analytics см. в [документации по поиску Log Analytics](../../azure-monitor/logs/log-query-overview.md).

## <a name="audit-log-fields"></a><a id="subheading-1"></a>Поля журнала аудита

| Имя (BLOB-объект) | Имя (концентраторы событий/Log Analytics) | Описание | Тип большого двоичного объекта | Тип концентраторов событий и Log Analytics |
|-------------|---------------------------------|-------------|-----------|-------------------------------|
| action_id | action_id_s | Идентификатор действия. | varchar(4) | строка |
| action_name | action_name_s | Имя действия | Н/Д | строка |
| additional_information | additional_information_s | Дополнительные сведения о событии, хранящиеся в формате XML | nvarchar(4000) | строка |
| affected_rows | affected_rows_d | Число строк, затронутых запросом | BIGINT | INT |
| application_name | application_name_s| Имя клиентского приложения | NVARCHAR(128) | строка |
| audit_schema_version | audit_schema_version_d | Всегда 1 | INT | INT |
| class_type | class_type_s | Тип проверяемой сущности, на которой выполняется аудит | varchar(2) | строка |
| class_type_desc | class_type_description_s | Описание проверяемой сущности, на которой выполняется аудит | Н/Д | строка |
| client_ip | client_ip_s | Исходный IP-адрес клиентского приложения | NVARCHAR(128) | строка |
| connection_id | Н/Д | Идентификатор соединения на сервере | Код GUID | Н/Д |
| data_sensitivity_information | data_sensitivity_information_s | Типы сведений и метки чувствительности, возвращаемые отслеживаемым запросом, на основе классифицированных столбцов в базе данных. Дополнительные сведения об [обнаружении и классификации данных в базе данных SQL Azure](data-discovery-and-classification-overview.md) | nvarchar(4000) | строка |
| database_name | database_name_s | Контекст базы данных, в котором произошло действие | sysname | строка |
| database_principal_id | database_principal_id_d | Идентификатор контекста пользователя базы данных, в котором выполняется действие | INT | INT |
| database_principal_name | database_principal_name_s | Имя контекста пользователя базы данных, в котором выполняется действие | sysname | строка |
| duration_milliseconds | duration_milliseconds_d | Длительность выполнения запроса в миллисекундах | BIGINT | INT |
| event_time | event_time_t | Дата и время срабатывания проверяемого действия | datetime2 | DATETIME |
| host_name | Н/Д | Имя узла клиента | строка | Н/Д |
| is_column_permission | is_column_permission_s | Флаг, обозначающий разрешение уровня столбца. 1 = true, 0 = false | bit | строка |
| Н/Д | is_server_level_audit_s | Флаг, указывающий, что этот аудит находится на уровне сервера | Н/Д | строка |
| object_ id | object_id_d | Идентификатор сущности, над которой выполнен аудит. К ним относятся: серверные объекты, базы данных, объекты базы данных и объекты схемы. 0, если сущность является самим сервером или если аудит не выполняется на уровне объектов | INT | INT |
| object_name | object_name_s | Имя сущности, для которой проводился аудит. К ним относятся: серверные объекты, базы данных, объекты базы данных и объекты схемы. 0, если сущность является самим сервером или если аудит не выполняется на уровне объектов | sysname | строка |
| permission_bitmask | permission_bitmask_s | Когда применимо, отображаются предоставленные, запрещенные или отмененные разрешения. | varbinary (16) | строка |
| response_rows | response_rows_d | Число строк, возвращенных в результирующем наборе | BIGINT | INT |
| schema_name | schema_name_s | Контекст схемы, в котором выполнялось действие. Значение NULL для аудитов, происходящих за пределами схемы | sysname | строка |
| Н/Д | securable_class_type_s | Защищаемый объект, сопоставляемый с class_type аудита | Н/Д | строка |
| sequence_group_id | sequence_group_id_g | Уникальный идентификатор | varbinary | Код GUID |
| sequence_number | sequence_number_d | Отслеживает последовательность записей в одной записи аудита, которая слишком велика для размещения в буфере записи для аудита | INT | INT |
| server_instance_name | server_instance_name_s | Имя экземпляра сервера, на котором произошел аудит | sysname | строка |
| server_principal_id | server_principal_id_d | Идентификатор контекста имени входа, в котором выполняется действие | INT | INT |
| server_principal_name | server_principal_name_s | Текущее имя входа | sysname | строка |
| server_principal_sid | server_principal_sid_s | Текущий идентификатор безопасности входа | varbinary | строка |
| session_id | session_id_d | Идентификатор сеанса, в котором произошло событие | smallint | INT |
| session_server_principal_name | session_server_principal_name_s | Участник сервера для сеанса | sysname | строка |
| инструкция | statement_s | Выполненная инструкция T-SQL (при наличии) | nvarchar(4000) | строка |
| Успешно | succeeded_s | Показывает, было ли успешным действие, запустившее событие. Для событий, отличных от имени для входа и пакетной службы, он сообщает только о том, успешно ли выполнена проверка разрешений, а не на выполнение операции. 1 = успешное завершение, 0 = сбой | bit | строка |
| target_database_principal_id | target_database_principal_id_d | Участник базы данных, над которым выполняется операция GRANT/DENY/REVOKE. 0, если неприменимо | INT | INT |
| target_database_principal_name | target_database_principal_name_s | Целевой пользователь действия. NULL, если неприменимо | строка | строка |
| target_server_principal_id | target_server_principal_id_d | Участник на уровне сервера, над которым выполняется операция GRANT/DENY/REVOKE. Возвращает 0, если неприменимо | INT | INT |
| target_server_principal_name | target_server_principal_name_s | Целевое имя входа действия. NULL, если неприменимо | sysname | строка |
| target_server_principal_sid | target_server_principal_sid_s | Идентификатор безопасности целевого имени входа. NULL, если неприменимо | varbinary | строка |
| transaction_id | transaction_id_d | Только SQL Server (начиная с 2016)-0 для базы данных SQL Azure | BIGINT | INT |
| user_defined_event_id | user_defined_event_id_d | Определяемый пользователем идентификатор события, передаваемый в качестве аргумента для sp_audit_write. NULL для системных событий (по умолчанию) и ненулевое значение для определяемого пользователем события. Дополнительные сведения см. в разделе [sp_audit_write (Transact-SQL)](/sql/relational-databases/system-stored-procedures/sp-audit-write-transact-sql) . | smallint | INT |
| user_defined_information | user_defined_information_s | Определяемые пользователем сведения, передаваемые в качестве аргумента для sp_audit_write. NULL для системных событий (по умолчанию) и ненулевое значение для определяемого пользователем события. Дополнительные сведения см. в разделе [sp_audit_write (Transact-SQL)](/sql/relational-databases/system-stored-procedures/sp-audit-write-transact-sql) . | nvarchar(4000) | строка |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об [аудите базы данных SQL Azure](auditing-overview.md).