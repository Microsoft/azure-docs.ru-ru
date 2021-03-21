---
title: Журналы аудита — база данных Azure для MariaDB
description: Описание журналов аудита, доступных в базе данных Azure для MariaDB, и доступных параметров для включения уровней ведения журнала.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: conceptual
ms.date: 6/24/2020
ms.openlocfilehash: e66eaf62b54759222421b17ebc7e33b82e018338
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98663763"
---
# <a name="audit-logs-in-azure-database-for-mariadb"></a>Журналы аудита в базе данных Azure для MariaDB

В базе данных Azure для MariaDB журнал аудита доступен для пользователей. Журнал аудита можно использовать для наблюдения за действиями на уровне базы данных и обычно используется для обеспечения соответствия.

## <a name="configure-audit-logging"></a>Настройка ведения журнала аудита

>[!IMPORTANT]
> Рекомендуется регистрировать только те типы событий и пользователей, которые необходимы для целей аудита, чтобы гарантировать, что производительность сервера не окажется сильной.

По умолчанию журнал аудита отключен. Чтобы включить его, задайте значение `audit_log_enabled` On.

Вы можете настроить еще несколько параметров.

- `audit_log_events`: управляет событиями, регистрируемыми в журнале. Конкретные события аудита см. в таблице ниже.
- `audit_log_include_users`— MariaDB пользователей, которые будут включаться в ведение журнала. Значение по умолчанию для этого параметра — пустое, которое будет включать всех пользователей для ведения журнала. Это имеет более высокий приоритет `audit_log_exclude_users` , чем. Максимальная длина параметра — 512 символов.
- `audit_log_exclude_users`— MariaDB пользователей, которых следует исключить из ведения журнала. Допускает не более четырех пользователей. Максимальная длина параметра — 256 символов.

> [!Note]
> `audit_log_include_users` имеет более высокий приоритет `audit_log_exclude_users` , чем. Например, если `audit_log_include_users`  =  `demouser` и `audit_log_exclude_users`  =  `demouser` , пользователь будет включаться в журналы аудита, так как `audit_log_include_users` имеет более высокий приоритет.

| **Событие** | **Описание** |
|---|---|
| `CONNECTION` | — Инициация подключения (успешно или неуспешно); <br> — Повторное применение проверки подлинности пользователя с другим пользователем или паролем во время сеанса <br> — Завершение подключения |
| `DML_SELECT`| Запросы SELECT |
| `DML_NONSELECT` | Операции вставки, удаления и обновления |
| `DML` | DML = DML_SELECT + DML_NONSELECT |
| `DDL` | Такие запросы, как "DROP DATABASE" |
| `DCL` | Запросы, например "предоставление разрешения" |
| `ADMIN` | Такие запросы, как "ОТОБРАЗИТЬ состояние" |
| `GENERAL` | Все в DML_SELECT, DML_NONSELECT, DML, DDL, ДКЛ и ADMIN |

## <a name="access-audit-logs"></a>Доступ к журналам аудита

Журналы аудита можно интегрировать с журналами диагностики Azure Monitor. Включив ведение журналов аудита на сервере MariaDB, вы можете реализовать отправку этих данных в журналы Azure Monitor, Центры событий или службу хранилища Azure. Дополнительные сведения о том, как включить журналы диагностики в портал Azure, см. в [статье о портале журналов аудита](howto-configure-audit-logs-portal.md#set-up-diagnostic-logs).

## <a name="diagnostic-logs-schemas"></a>Схемы журналов диагностики

В следующих разделах описываются выходные данные журналов аудита MariaDB, основанные на типе событий. Порядок появления выбранных полей зависит от выбранного метода вывода.

### <a name="connection"></a>Подключение

| **Свойство** | **Описание** |
|---|---|
| `TenantId` | Идентификатор клиента |
| `SourceSystem` | `Azure` |
| `TimeGenerated [UTC]` | Метка времени, когда журнал был записан в формате UTC |
| `Type` | Тип журнала Всегда `AzureDiagnostics`. |
| `SubscriptionId` | Идентификатор GUID для подписки, принадлежащей серверу |
| `ResourceGroup` | Имя группы ресурсов, принадлежащей серверу |
| `ResourceProvider` | Имя поставщика ресурсов. Всегда `MICROSOFT.DBFORMARIADB`. |
| `ResourceType` | `Servers` |
| `ResourceId` | Универсальный код ресурса (URI) |
| `Resource` | Имя сервера |
| `Category` | `MySqlAuditLogs` |
| `OperationName` | `LogEvent` |
| `event_class_s` | `connection_log` |
| `event_subclass_s` | `CONNECT`, `DISCONNECT` |
| `connection_id_d` | Уникальный идентификатор соединения, созданный MariaDB |
| `host_s` | Пусто |
| `ip_s` | IP-адрес клиента, подключающегося к MariaDB |
| `user_s` | Имя пользователя, который исполняет запрос |
| `db_s` | Имя базы данных, подключенной к |
| `\_ResourceId` | Универсальный код ресурса (URI) |

### <a name="general"></a>Общие сведения

Приведенная ниже схема относится к типам событий GENERAL, DML_SELECT, DML_NONSELECT, DML, DDL, ДКЛ и ADMIN.

> [!NOTE]
> Для `sql_text` Журнал будет обрезан, если его длина превышает 2048 символов.

| **Свойство** | **Описание** |
|---|---|
| `TenantId` | Идентификатор клиента |
| `SourceSystem` | `Azure` |
| `TimeGenerated [UTC]` | Метка времени, когда журнал был записан в формате UTC |
| `Type` | Тип журнала Всегда `AzureDiagnostics`. |
| `SubscriptionId` | Идентификатор GUID для подписки, принадлежащей серверу |
| `ResourceGroup` | Имя группы ресурсов, принадлежащей серверу |
| `ResourceProvider` | Имя поставщика ресурсов. Всегда `MICROSOFT.DBFORMARIADB`. |
| `ResourceType` | `Servers` |
| `ResourceId` | Универсальный код ресурса (URI) |
| `Resource` | Имя сервера |
| `Category` | `MySqlAuditLogs` |
| `OperationName` | `LogEvent` |
| `LogicalServerName_s` | Имя сервера |
| `event_class_s` | `general_log` |
| `event_subclass_s` | `LOG`, `ERROR`, `RESULT` |
| `event_time` | Начало запроса секунд в метке времени UNIX |
| `error_code_d` | Код ошибки, если запрос не выполнен. `0` означает отсутствие ошибок |
| `thread_id_d` | Идентификатор потока, выполнившего запрос |
| `host_s` | Пусто |
| `ip_s` | IP-адрес клиента, подключающегося к MariaDB |
| `user_s` | Имя пользователя, который исполняет запрос |
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

- Перечисление событий аудита по всем серверам MariaDB с включенными журналами диагностики для журналов аудита

    ```kusto
    AzureDiagnostics
    | where Category == 'MySqlAuditLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, event_subclass_s, event_time_t, user_s , ip_s , sql_text_s 
    | order by TimeGenerated asc nulls last
    ``` 

## <a name="next-steps"></a>Дальнейшие действия

- [Настройка журналов аудита в портал Azure](howto-configure-audit-logs-portal.md)