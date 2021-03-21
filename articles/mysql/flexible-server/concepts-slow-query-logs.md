---
title: Журналы запросов с высокой занесениеми. база данных Azure для MySQL — гибкий сервер
description: Описание журналов запросов с высокой производительностью, доступных в базе данных Azure для гибкого сервера MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 9/21/2020
ms.openlocfilehash: d311ea3158e1f9d53c51fe239103039849597d11
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100579193"
---
# <a name="slow-query-logs-in-azure-database-for-mysql-flexible-server-preview"></a>Журналы запросов в базе данных Azure для гибкого сервера MySQL (Предварительная версия)

> [!IMPORTANT] 
> Сейчас предоставляется общедоступная предварительная версия Гибкого сервера Базы данных Azure для MySQL.

В базе данных Azure для гибкого сервера MySQL для настройки и доступа пользователям доступен журнал запросов с задержкой. Журналы медленных запросов по умолчанию отключены и могут быть включены для выявления узких мест производительности во время устранения неполадок.

Дополнительные сведения о журнале медленных запросов MySQL см. в [разделе Журнал медленных запросов](https://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html) документации по модулю MySQL.

## <a name="configure-slow-query-logging"></a>Настройка ведения журнала запросов с задержкой 
По умолчанию журнал запросов с задержкой отключен. Чтобы включить журналы, установите `slow_query_log` для параметра сервера значение *On*. Это можно настроить с помощью портал Azure или Azure CLI <!-- add link to server parameter-->. 

Другие параметры, которые можно настроить для управления поведением журнала запросов с задержкой, включают:

- **long_query_time**: заносить в журнал запрос, если его выполнение занимает больше времени `long_query_time` (в секундах). Значение по умолчанию — 10 секунд.
- **log_slow_admin_statements**: определяет, являются ли инструкции администрирования (например, `ALTER_TABLE`, `ANALYZE_TABLE` ) записываются в журнал.
- **log_queries_not_using_indexes**: определяет, заносятся ли в журнал запросы, не использующие индексы.
- **log_throttle_queries_not_using_indexes**: ограничивает количество неиндексированных запросов, которые могут быть записаны в журнал запросов с задержкой. Этот параметр вступает в силу `log_queries_not_using_indexes` , если для задано значение *On* .

> [!IMPORTANT]
> Если таблицы не индексируются, установка `log_queries_not_using_indexes` `log_throttle_queries_not_using_indexes` параметров и в значение **On** может повлиять на производительность MySQL, так как все запросы к этим неиндексированным таблицам будут записываться в журнал медленных запросов.

Полное описание параметров, применимых для журнала медленных запросов, вы найдете в [соответствующем разделе документации по MySQL](https://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html).

## <a name="access-slow-query-logs"></a>Доступ к журналам запросов с задержкой

Журналы запросов с высокой занесениею интегрируются с Azure Monitor параметрами диагностики. После включения журналов запросов на гибком сервере MySQL их можно отправить в журналы Azure Monitor, концентраторы событий или службу хранилища Azure. Дополнительные сведения о параметрах диагностики см. в [документации по журналам диагностики](../../azure-monitor/essentials/platform-logs-overview.md). Дополнительные сведения о включении параметров диагностики в портал Azure см. в [статье о медленных журналах запросов на портале](how-to-configure-slow-query-logs-portal.md#set-up-diagnostics).

В следующей таблице описаны выходные данные журнала запросов с задержкой. Порядок появления выбранных полей зависит от выбранного метода вывода.

| **Свойство** | **Описание** |
|---|---|
| `TenantId` | Идентификатор клиента |
| `SourceSystem` | `Azure` |
| `TimeGenerated` ФОРМАТА | Метка времени, когда журнал был записан в формате UTC |
| `Type` | Тип журнала Всегда `AzureDiagnostics`. |
| `SubscriptionId` | Идентификатор GUID для подписки, принадлежащей серверу |
| `ResourceGroup` | Имя группы ресурсов, принадлежащей серверу |
| `ResourceProvider` | Имя поставщика ресурсов. Всегда `MICROSOFT.DBFORMYSQL`. |
| `ResourceType` | `Servers` |
| `ResourceId` | Универсальный код ресурса (URI) |
| `Resource` | Имя сервера |
| `Category` | `MySqlSlowLogs` |
| `OperationName` | `LogEvent` |
| `Logical_server_name_s` | Имя сервера |
| `start_time_t` ФОРМАТА | Время начала запроса. |
| `query_time_s` | Общее время в секундах, затраченное на выполнение запроса |
| `lock_time_s` | Общее время блокировки запроса (в секундах) |
| `user_host_s` | Имя пользователя |
| `rows_sent_s` | Количество отправленных строк. |
| `rows_examined_s` | Число проверенных строк. |
| `last_insert_id_s` | [last_insert_id](https://dev.mysql.com/doc/refman/5.7/en/information-functions.html#function_last-insert-id) |
| `insert_id_s` | Вставить идентификатор |
| `sql_text_s` | Полный запрос. |
| `server_id_s` | Идентификатор сервера |
| `thread_id_s` | Идентификатор потока |
| `\_ResourceId` | Универсальный код ресурса (URI) |

> [!Note]
> Для `sql_text_s` Журнал будет обрезан, если его длина превышает 2048 символов.

## <a name="analyze-logs-in-azure-monitor-logs"></a>Анализ журналов в журналах Azure Monitor

После передвижения журналов запросов в Azure Monitor журналов с помощью журналов диагностики можно выполнить дальнейший анализ запросов. Ниже приведены примеры запросов, которые помогут приступить к работе. Обязательно обновите указанные ниже имена серверов.

- Запросы с более чем 10 секундами на определенном сервере

    ```Kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlSlowLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, start_time_t , query_time_d, sql_text_s 
    | where query_time_d > 10
    ```

- Вывод списка 5 самых длинных запросов на определенном сервере

    ```Kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlSlowLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, start_time_t , query_time_d, sql_text_s 
    | order by query_time_d desc
    | take 5
    ```

- Суммирование запросов с минимальным, максимальным, средним и стандартным отклонением времени запроса на определенном сервере

    ```Kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlSlowLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, start_time_t , query_time_d, sql_text_s 
    | summarize count(), min(query_time_d), max(query_time_d), avg(query_time_d), stdev(query_time_d), percentile(query_time_d, 95) by LogicalServerName_s
    ```

- График распределения скорости запросов на определенном сервере

    ```Kusto
    AzureDiagnostics
    | where LogicalServerName_s == '<your server name>'
    | where Category == 'MySqlSlowLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, start_time_t , query_time_d, sql_text_s 
    | summarize count() by LogicalServerName_s, bin(TimeGenerated, 5m)
    | render timechart
    ```

- Отображение запросов дольше 10 секунд на всех серверах MySQL с включенными журналами диагностики

    ```Kusto
    AzureDiagnostics
    | where Category == 'MySqlSlowLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, start_time_t , query_time_d, sql_text_s 
    | where query_time_d > 10
    ```    
    
## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения о [журналах аудита](concepts-audit-logs.md)
- Настройка журналов запросов от [портал Azure](how-to-configure-slow-query-logs-portal.md)
<!-- - [How to configure slow query logs from the Azure CLI](howto-configure-server-logs-in-cli.md). -->