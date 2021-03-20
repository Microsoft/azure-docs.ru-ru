---
title: Журналы запросов с высокой производительностью — база данных Azure для MariaDB
description: Описание журналов, доступных в базе данных Azure для MariaDB, и параметров для включения различных уровней ведения журналов.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: conceptual
ms.date: 11/6/2020
ms.openlocfilehash: 2e7e56616300566839fadef762c2165c8d989e6e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100570808"
---
# <a name="slow-query-logs-in-azure-database-for-mariadb"></a>Журналы запросов в базе данных Azure для MariaDB
В базе данных Azure для MariaDB пользователям доступен журнал медленных запросов. Доступ к журналам транзакций не поддерживается. Журнал медленных запросов можно использовать для выявления проблем с производительностью при устранении неполадок.

Дополнительные сведения об этих журналах см. в документации к MariaDB по [журналам медленных запросов](https://mariadb.com/kb/en/library/slow-query-log-overview/).

Если [хранилище запросов](concepts-query-store.md) включено на сервере, вы можете увидеть запросы, например "", `CALL mysql.az_procedure_collect_wait_stats (900, 30);` записанные в журналы запросов с задержкой. Это ожидаемое поведение, так как функция хранилища запросов собирает статистические данные о запросах. 

## <a name="configure-slow-query-logging"></a>Настройка ведения журнала запросов с задержкой
Журнал медленных запросов по умолчанию отключен. Чтобы включить его, задайте значение `slow_query_log` On. Это можно включить с помощью портал Azure или Azure CLI. 

Вы можете настроить еще несколько параметров.

- **long_query_time.** Если запрос занимает больше времени, чем задано значением long_query_time (в секундах), информация о нем заносится в журнал. Значение по умолчанию — 10 секунд.
- **log_slow_admin_statements.** Указывает, нужно ли сохранять в журнал slow_query_log административные инструкции, например ALTER_TABLE и ANALYZE_TABLE.
- **log_queries_not_using_indexes**. Указывает, нужно ли сохранять в журнал slow_query_log запросы, не использующие индексы.
- **log_throttle_queries_not_using_indexes.** Ограничивает число не использующих индексы запросов, сохраняемых в журнале медленных запросов. Этот параметр применяется, только если log_queries_not_using_indexes имеет значение "ON" (Включено).
- **log_output**. Если значение — "File", журнал запросов может быть записан как в локальное хранилище сервера, так и в Azure Monitor журналов диагностики. Если задано значение "Нет", журнал медленных запросов будет записываться только в журналы диагностики Azure Monitor. 

> [!IMPORTANT]
> Если таблицы не индексируются, установка `log_queries_not_using_indexes` `log_throttle_queries_not_using_indexes` параметров и в значение On может повлиять на производительность MariaDB, так как все запросы к этим неиндексированным таблицам будут записываться в журнал медленных запросов.<br><br>
> Если вы планируете вести длительные запросы в течение продолжительного периода времени, рекомендуется установить значение `log_output` "нет". Если задано значение "File", эти журналы записываются в хранилище локального сервера и могут повлиять на производительность MariaDB. 

Полное описание параметров журнала медленных запросов см. в [соответствующей документации к MariaDB](https://mariadb.com/kb/en/library/slow-query-log-overview/).

## <a name="access-slow-query-logs"></a>Доступ к журналам запросов с задержкой
Существует два варианта доступа к журналам запросов в базе данных Azure для MariaDB: хранилище локального сервера или журналы диагностики Azure Monitor. Это задается с помощью `log_output` параметра.

Для хранилища локального сервера можно перечислить и скачать журналы запросов с помощью портал Azure или Azure CLI. В портал Azure перейдите к своему серверу в портал Azure. В разделе **Мониторинг** найдите страницу **Журналы сервера**. Описание работы с Azure CLI см. в статье [Настройка журналов сервера и получение к ним доступа с помощью Azure CLI](howto-configure-server-logs-cli.md). 

Azure Monitor журналы диагностики позволяют передавать журналы запросов с задержкой в Azure Monitor журналы (Log Analytics), службу хранилища Azure или концентраторы событий. Дополнительные сведения см. [ниже](concepts-server-logs.md#diagnostic-logs) .

## <a name="local-server-storage-log-retention"></a>Хранение журнала на локальном сервере
При ведении журнала в локальное хранилище сервера журналы доступны в течение семи дней с момента их создания. Если общий объем доступных журналов превышает 7 ГБ, по мере необходимости удаляются самые старые файлы.

Новый файл журнала создается каждые 24 часа или при достижении размера файла 7 ГБ.

> [!Note]
> Приведенные выше сроки хранения журналов не применяются к журналам, перенаправленным с помощью Azure Monitor журналов диагностики. Можно изменить срок хранения для создаваемых приемников данных (например, Служба хранилища Azure).

## <a name="diagnostic-logs"></a>Журналы диагностики
База данных Azure для MariaDB интегрирована с журналами диагностики Azure Monitor. Включив журналы запросов на сервере MariaDB, вы можете выбрать их для Azure Monitor журналов, концентраторов событий или службы хранилища Azure. Дополнительные сведения о том, как включить журналы диагностики, см. в статье [Сбор и использование данных журнала из ресурсов Azure](../azure-monitor/essentials/platform-logs-overview.md).

В приведенной ниже таблице описывается содержимое каждого журнала. Порядок появления выбранных полей зависит от выбранного метода вывода.

| **Свойство** | **Описание** |
|---|---|
| `TenantId` | Идентификатор клиента |
| `SourceSystem` | `Azure` |
| `TimeGenerated` ФОРМАТА | Метка времени, когда журнал был записан в формате UTC |
| `Type` | Тип журнала Всегда `AzureDiagnostics`. |
| `SubscriptionId` | Идентификатор GUID для подписки, принадлежащей серверу |
| `ResourceGroup` | Имя группы ресурсов, принадлежащей серверу |
| `ResourceProvider` | Имя поставщика ресурсов. Всегда `MICROSOFT.DBFORMARIADB`. |
| `ResourceType` | `Servers` |
| `ResourceId` | Универсальный код ресурса (URI) |
| `Resource` | Имя сервера |
| `Category` | `MySqlSlowLogs` |
| `OperationName` | `LogEvent` |
| `Logical_server_name_s` | Имя сервера |
| `start_time_t` ФОРМАТА | Время начала запроса. |
| `query_time_s` | Общее время, которое потребовалось для выполнения запроса. |
| `lock_time_s` | Общее время блокировки запроса. |
| `user_host_s` | Имя пользователя |
| `rows_sent_s` | Количество отправленных строк. |
| `rows_examined_s` | Число проверенных строк. |
| `last_insert_id_s` | [last_insert_id](https://mariadb.com/kb/en/library/last_insert_id/) |
| `insert_id_s` | Вставить идентификатор |
| `sql_text_s` | Полный запрос. |
| `server_id_s` | Идентификатор сервера |
| `thread_id_s` | Идентификатор потока |
| `\_ResourceId` | Универсальный код ресурса (URI) |

> [!Note]
> Для `sql_text` Журнал будет обрезан, если его длина превышает 2048 символов.

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

- Отображение запросов дольше 10 секунд для всех серверов MariaDB с включенными журналами диагностики

    ```Kusto
    AzureDiagnostics
    | where Category == 'MySqlSlowLogs'
    | project TimeGenerated, LogicalServerName_s, event_class_s, start_time_t , query_time_d, sql_text_s 
    | where query_time_d > 10
    ```    
    
## <a name="next-steps"></a>Дальнейшие действия
- [Настройка журналов запросов от портал Azure](howto-configure-server-logs-portal.md)
- [Настройка журналов запросов от Azure CLI](howto-configure-server-logs-cli.md)
