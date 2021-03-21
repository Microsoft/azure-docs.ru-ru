---
title: Журналы аудита — база данных Azure для MySQL — гибкий сервер
description: Описание журналов аудита, доступных в базе данных Azure для гибкого сервера MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 9/21/2020
ms.openlocfilehash: 1232a0753c988f5a28ebba28f9819aa67ce28603
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101718749"
---
# <a name="track-database-activity-with-audit-logs-in-azure-database-for-mysql-flexible-server"></a>Мониторинг активности базы данных с помощью журналов аудита в базе данных Azure для гибкого сервера MySQL

> [!IMPORTANT] 
> Сейчас предоставляется общедоступная предварительная версия Гибкого сервера Базы данных Azure для MySQL.

База данных Azure для гибкого сервера MySQL предоставляет пользователям возможность настраивать журналы аудита. Журналы аудита можно использовать для наблюдения за действиями на уровне базы данных, включая события подключения, администратора, DDL и DML. Эти типы журналов обычно используются для обеспечения соответствия требованиям.

## <a name="configure-audit-logging"></a>Настройка ведения журнала аудита

>[!IMPORTANT]
> Рекомендуется регистрировать только те типы событий и пользователей, которые необходимы для целей аудита, чтобы гарантировать, что производительность сервера не окажется сильной.

По умолчанию журналы аудита отключены. Чтобы включить их, установите для `audit_log_enabled` параметра сервера значение *вкл*. Это можно настроить с помощью портал Azure или Azure CLI <!-- add link to server parameter-->. 

Другие параметры, которые можно настроить для управления поведением ведения журнала аудита, включают:

- `audit_log_events`: управляет событиями, регистрируемыми в журнале. Конкретные события аудита см. в таблице ниже.
- `audit_log_include_users`: Пользователи MySQL, включаемые в ведение журнала. Значение по умолчанию для этого параметра — пустое, которое будет включать всех пользователей для ведения журнала. Это имеет более высокий приоритет `audit_log_exclude_users` , чем. Максимальная длина параметра — 512 символов.
- `audit_log_exclude_users`: Пользователи MySQL, которые будут исключены из ведения журнала. Максимальная длина параметра — 512 символов.

> [!NOTE]
> `audit_log_include_users` имеет более высокий приоритет `audit_log_exclude_users` , чем. Например, если `audit_log_include_users`  =  `demouser` и `audit_log_exclude_users`  =  `demouser` , пользователь будет включаться в журналы аудита, так как `audit_log_include_users` имеет более высокий приоритет.

| **Событие** | **Описание** |
|---|---|
| `CONNECTION` | — Инициация подключения (успешно или неуспешно); <br> — Повторная проверка подлинности пользователя с другим пользователем или паролем во время сеанса <br> — Завершение подключения |
| `DML_SELECT`| Запросы SELECT |
| `DML_NONSELECT` | Операции вставки, удаления и обновления |
| `DML` | DML = DML_SELECT + DML_NONSELECT |
| `DDL` | Такие запросы, как "DROP DATABASE" |
| `DCL` | Запросы, например "предоставление разрешения" |
| `ADMIN` | Такие запросы, как "ОТОБРАЗИТЬ состояние" |
| `GENERAL` | Все в DML_SELECT, DML_NONSELECT, DML, DDL, ДКЛ и ADMIN |
| `TABLE_ACCESS` | — Инструкции чтения таблицы, такие как SELECT или INSERT INTO... МЕТЬТЕ <br> — Инструкции DELETE таблицы, такие как DELETE или TRUNCATE TABLE <br> — Табличные инструкции INSERT, такие как INSERT или Replace <br> -Табличные инструкции UPDATE, например UPDATE |

## <a name="access-audit-logs"></a>Доступ к журналам аудита

Журналы аудита интегрируются с Azure Monitor параметрами диагностики. После включения журналов аудита на гибком сервере MySQL их можно отправить в журналы Azure Monitor, концентраторы событий или службу хранилища Azure. Дополнительные сведения о параметрах диагностики см. в [документации по журналам диагностики](../../azure-monitor/essentials/platform-logs-overview.md). Дополнительные сведения о включении параметров диагностики в портал Azure см. в [статье о портале журналов аудита](how-to-configure-audit-logs-portal.md#set-up-diagnostics).

В следующих разделах описываются выходные данные журналов аудита MySQL на основе типа события. Порядок появления выбранных полей зависит от выбранного метода вывода.

### <a name="connection"></a>Подключение

| **Свойство** | **Описание** |
|---|---|
| `TenantId` | Идентификатор клиента |
| `SourceSystem` | `Azure` |
| `TimeGenerated [UTC]` | Метка времени, когда журнал был записан в формате UTC |
| `Type` | Тип журнала Всегда `AzureDiagnostics`. |
| `SubscriptionId` | Идентификатор GUID для подписки, принадлежащей серверу |
| `ResourceGroup` | Имя группы ресурсов, принадлежащей серверу |
| `ResourceProvider` | Имя поставщика ресурсов. Всегда `MICROSOFT.DBFORMYSQL`. |
| `ResourceType` | `Servers` |
| `ResourceId` | Универсальный код ресурса (URI) |
| `Resource` | Имя сервера |
| `Category` | `MySqlAuditLogs` |
| `OperationName` | `LogEvent` |
| `LogicalServerName_s` | Имя сервера |
| `event_class_s` | `connection_log` |
| `event_subclass_s` | `CONNECT`, `DISCONNECT`, `CHANGE USER` |
| `connection_id_d` | Уникальный идентификатор соединения, созданный MySQL |
| `host_s` | Пусто |
| `ip_s` | IP-адрес клиента, подключающегося к MySQL |
| `user_s` | Имя пользователя, который исполняет запрос |
| `db_s` | Имя базы данных, подключенной к |
| `\_ResourceId` | Универсальный код ресурса (URI) |

### <a name="general"></a>Общие сведения

Приведенная ниже схема относится к типам событий GENERAL, DML_SELECT, DML_NONSELECT, DML, DDL, ДКЛ и ADMIN.

> [!NOTE]
> Для `sql_text_s` Журнал будет обрезан, если его длина превышает 2048 символов.

| **Свойство** | **Описание** |
|---|---|
| `TenantId` | Идентификатор клиента |
| `SourceSystem` | `Azure` |
| `TimeGenerated [UTC]` | Метка времени, когда журнал был записан в формате UTC |
| `Type` | Тип журнала Всегда `AzureDiagnostics`. |
| `SubscriptionId` | Идентификатор GUID для подписки, принадлежащей серверу |
| `ResourceGroup` | Имя группы ресурсов, принадлежащей серверу |
| `ResourceProvider` | Имя поставщика ресурсов. Всегда `MICROSOFT.DBFORMYSQL`. |
| `ResourceType` | `Servers` |
| `ResourceId` | Универсальный код ресурса (URI) |
| `Resource` | Имя сервера |
| `Category` | `MySqlAuditLogs` |
| `OperationName` | `LogEvent` |
| `LogicalServerName_s` | Имя сервера |
| `event_class_s` | `general_log` |
| `event_subclass_s` | `LOG`, `ERROR` , `RESULT` (доступно только для MySQL 5,6) |
| `event_time` | Время начала запроса в формате UTC (метка времени) |
| `error_code_d` | Код ошибки, если запрос не выполнен. `0` означает отсутствие ошибок |
| `thread_id_d` | Идентификатор потока, выполнившего запрос |
| `host_s` | Пусто |
| `ip_s` | IP-адрес клиента, подключающегося к MySQL |
| `user_s` | Имя пользователя, который исполняет запрос |
| `sql_text_s` | Текст полного запроса |
| `\_ResourceId` | Универсальный код ресурса (URI) |

### <a name="table-access"></a>Доступ к таблицам

> [!NOTE]
> Для `sql_text_s` Журнал будет обрезан, если его длина превышает 2048 символов.

| **Свойство** | **Описание** |
|---|---|
| `TenantId` | Идентификатор клиента |
| `SourceSystem` | `Azure` |
| `TimeGenerated [UTC]` | Метка времени, когда журнал был записан в формате UTC |
| `Type` | Тип журнала Всегда `AzureDiagnostics`. |
| `SubscriptionId` | Идентификатор GUID для подписки, принадлежащей серверу |
| `ResourceGroup` | Имя группы ресурсов, принадлежащей серверу |
| `ResourceProvider` | Имя поставщика ресурсов. Всегда `MICROSOFT.DBFORMYSQL`. |
| `ResourceType` | `Servers` |
| `ResourceId` | Универсальный код ресурса (URI) |
| `Resource` | Имя сервера |
| `Category` | `MySqlAuditLogs` |
| `OperationName` | `LogEvent` |
| `LogicalServerName_s` | Имя сервера |
| `event_class_s` | `table_access_log` |
| `event_subclass_s` | `READ`, `INSERT`, `UPDATE` или `DELETE` |
| `connection_id_d` | Уникальный идентификатор соединения, созданный MySQL |
| `db_s` | Имя базы данных, к которой осуществлялся доступ |
| `table_s` | Имя таблицы, к которой осуществлялся доступ |
| `sql_text_s` | Текст полного запроса |
| `\_ResourceId` | Универсальный код ресурса (URI) |

## <a name="analyze-logs-in-azure-monitor-logs"></a>Анализ журналов в журналах Azure Monitor

После нахождения журналов аудита в Azure Monitor журналов с помощью журналов диагностики можно выполнить дальнейший анализ событий аудита. Ниже приведены примеры запросов, которые помогут приступить к работе. Обязательно обновите указанные ниже имена серверов.

- Вывод списка общих событий на определенном сервере

    ```kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlAuditLogs' and event_class_s == "general_log"
    | project TimeGenerated, LogicalServerName_s, event_class_s, event_subclass_s, event_time_t, user_s , ip_s , sql_text_s 
    | order by TimeGenerated asc nulls last 
    ```

- Вывод списка событий подключения на определенном сервере

    ```kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlAuditLogs' and event_class_s == "connection_log"
    | project TimeGenerated, LogicalServerName_s, event_class_s, event_subclass_s, event_time_t, user_s , ip_s , sql_text_s 
    | order by TimeGenerated asc nulls last
    ```

- Обобщение событий аудита на определенном сервере

    ```kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlAuditLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, event_subclass_s, event_time_t, user_s , ip_s , sql_text_s 
    | summarize count() by event_class_s, event_subclass_s, user_s, ip_s
    ```

- Диаграмма распределение типов событий аудита на определенном сервере

    ```kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlAuditLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, event_subclass_s, event_time_t, user_s , ip_s , sql_text_s 
    | summarize count() by LogicalServerName_s, bin(TimeGenerated, 5m)
    | render timechart 
    ```

- Перечисление событий аудита по всем серверам MySQL с включенными журналами диагностики для журналов аудита

    ```kusto
    AzureDiagnostics
    | where Category == 'MySqlAuditLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, event_subclass_s, event_time_t, user_s , ip_s , sql_text_s 
    | order by TimeGenerated asc nulls last
    ``` 

## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения о [журналах медленных запросов](concepts-slow-query-logs.md)
- Настройка журналов запросов аудита из [портал Azure](how-to-configure-audit-logs-portal.md)
<!-- - [How to configure audit logs in the Azure portal](howto-configure-audit-logs-portal.md)-->
