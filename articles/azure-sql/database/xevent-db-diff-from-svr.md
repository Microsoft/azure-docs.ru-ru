---
title: Расширенные события
description: В статье описываются расширенные события (XEvents) в Базе данных SQL Azure и отличия соответствующих сеансов событий от сеансов событий в Microsoft SQL Server.
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: reference
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: sstein
ms.date: 12/19/2018
ms.openlocfilehash: 139673e46421aa0dc19298697872fbff5fe587af
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96501215"
---
# <a name="extended-events-in-azure-sql-database"></a>Расширенные события в базе данных SQL Azure 
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

[!INCLUDE [sql-database-xevents-selectors-1-include](../../../includes/sql-database-xevents-selectors-1-include.md)]

Набор функций расширенных событий в базе данных SQL Azure — это надежное подмножество функций SQL Server и Azure SQL Управляемый экземпляр.

*XEvents* — это неофициальное обозначение расширенных событий, которое встречается в блоках и других неофициальных источниках.

Дополнительные сведения о расширенных событиях можно найти по адресу:

- [Быстрое начало. Расширенные события в SQL Server](/sql/relational-databases/extended-events/quick-start-extended-events-in-sql-server)
- [Расширенные события](/sql/relational-databases/extended-events/extended-events)

## <a name="prerequisites"></a>Предварительные условия

В данной статье предполагается, чтобы вы уже ознакомились со следующими компонентами:

- [База данных SQL Azure](https://azure.microsoft.com/services/sql-database/)
- [Расширенные события](/sql/relational-databases/extended-events/extended-events)

- Основная часть нашей документации по расширенным событиям относится к SQL Server, базе данных SQL Azure и Управляемый экземпляр SQL Azure.

При выборе файла событий в качестве [целевого объекта](#AzureXEventsTargets)пригодится знание следующих компонентов:

- [Служба хранилища Azure](https://azure.microsoft.com/services/storage/)

- [Azure PowerShell с хранилищем Azure](/powershell/module/az.storage/)

## <a name="code-samples"></a>Примеры кода

Связанные разделы содержат два примера кода.

- [Код целевого кольцевого буфера для расширенных событий в Базе данных SQL Azure](xevent-code-ring-buffer.md)

  - Простой короткий сценарий Transact-SQL.
  - В этой статье с примером кода подчеркивается, что после завершения работы с целевым объектом "Кольцевой буфер" необходимо освободить задействованные им ресурсы с помощью инструкции изменения и удаления `ALTER EVENT SESSION ... ON DATABASE DROP TARGET ...;` . Впоследствии вы сможете добавить другой экземпляр кольцевого буфера с помощью оператора `ALTER EVENT SESSION ... ON DATABASE ADD TARGET ...`.

- [Код целевого файла событий для расширенных событий в Базе данных SQL Azure](xevent-code-event-file.md)

  - Этап 1. PowerShell: создание контейнера хранилища Azure в облаке.
  - Этап 2. Transact-SQL: использование контейнера хранилища Azure.

## <a name="transact-sql-differences"></a>Отличия Transact-SQL

- При выполнении команды [CREATE EVENT SESSION](/sql/t-sql/statements/create-event-session-transact-sql) на сервере SQL Server используется предложение **ON SERVER** . Но в базе данных SQL Azure вместо этого используется предложение **On Database** .
- Предложение **ON DATABASE** применяется также в командах Transact-SQL [ALTER EVENT SESSION](/sql/t-sql/statements/alter-event-session-transact-sql) и [DROP EVENT SESSION](/sql/t-sql/statements/drop-event-session-transact-sql).

- Мы рекомендуем включать параметр сеанса событий **STARTUP_STATE = ON** в операторы **CREATE EVENT SESSION** и **ALTER EVENT SESSION**.
  - Значение **= ON** поддерживает автоматический перезапуск после перенастройки логической базы данных из-за сбоя.

## <a name="new-catalog-views"></a>Новые представления каталога

Функцию расширенных событий поддерживают несколько [представлений каталога](/sql/relational-databases/system-catalog-views/catalog-views-transact-sql). Представления каталога сообщают *метаданные или определения* сеансов событий, созданных пользователями в текущей базе данных. Представления не возвращают сведения об экземплярах активных сеансов событий.

| Имя<br/>представления каталога | Описание |
|:--- |:--- |
| **sys.database_event_session_actions** |Возвращает строку для каждого действия над каждым событием из сеанса событий. |
| **sys.database_event_session_events** |Возвращает строку для каждого события в сеансе событий. |
| **sys.database_event_session_fields** |Возвращает строку для каждого настраиваемого столбца, который можно прямо задавать для событий и целевых объектов. |
| **sys.database_event_session_targets** |Возвращает строку для каждой цели события для сеанса событий. |
| **sys.database_event_sessions** |Возвращает строку для каждого сеанса событий в базе данных. |

В Microsoft SQL Server аналогичные представления каталогов имеют имена, содержащие *.server\_* вместо *.database\_*. Шаблон имени выглядит как **sys.server_event_%**.

## <a name="new-dynamic-management-views-dmvs"></a>Новые динамические административные представления [(DMV)](/sql/relational-databases/system-dynamic-management-views/system-dynamic-management-views)

База данных SQL Azure включает [динамические административные представления (DMV)](/sql/relational-databases/system-dynamic-management-views/extended-events-dynamic-management-views) , которые поддерживают расширенные события. DMV сообщают об *активных* сеансах событий.

| Имя DMV | Описание |
|:--- |:--- |
| **sys.dm_xe_database_session_event_actions** |Возвращает сведения о действиях сеанса событий. |
| **sys.dm_xe_database_session_events** |Возвращает сведения о событиях сеанса. |
| **sys.dm_xe_database_session_object_columns** |Отображает значения конфигурации объектов, привязанных к сеансу. |
| **sys.dm_xe_database_session_targets** |Возвращает сведения о целевых объектах сеанса. |
| **sys.dm_xe_database_sessions** |Возвращает строку для каждого сеанса событий, относящегося к текущей базе данных. |

В Microsoft SQL Server аналогичные представления каталога именуются без части имени *\_ базы данных* , например:

- **sys.dm_xe_sessions** вместо имени<br/>**sys.dm_xe_database_sessions**.

### <a name="dmvs-common-to-both"></a>Общие DMV

Для расширенных событий существуют дополнительные динамические административные представления, которые являются общими для базы данных SQL Azure, Управляемый экземпляр Azure SQL и Microsoft SQL Server.

- **sys.dm_xe_map_values**
- **sys.dm_xe_object_columns**
- **sys.dm_xe_objects**
- **sys.dm_xe_packages**

<a name="sqlfindseventsactionstargets" id="sqlfindseventsactionstargets"></a>

## <a name="find-the-available-extended-events-actions-and-targets"></a>Поиск доступных расширенных событий, действий и целевых объектов

Для получения списка доступных событий, действий и целевых объектов можно выполнить простую SQL-команду **SELECT** .

```sql
SELECT
        o.object_type,
        p.name         AS [package_name],
        o.name         AS [db_object_name],
        o.description  AS [db_obj_description]
    FROM
                   sys.dm_xe_objects  AS o
        INNER JOIN sys.dm_xe_packages AS p  ON p.guid = o.package_guid
    WHERE
        o.object_type in
            (
            'action',  'event',  'target'
            )
    ORDER BY
        o.object_type,
        p.name,
        o.name;
```

<a name="AzureXEventsTargets" id="AzureXEventsTargets"></a> &nbsp;

## <a name="targets-for-your-azure-sql-database-event-sessions"></a>Целевые объекты для сеансов событий базы данных SQL Azure

Ниже приведены целевые объекты, которые могут записывать результаты сеансов событий в базе данных SQL Azure.

- [Целевой объект "Кольцевой буфер"](/previous-versions/sql/sql-server-2016/bb630339(v=sql.130)) — сохраняет данные события в памяти на недолгое время.
- [Целевой объект "Счетчик событий"](/previous-versions/sql/sql-server-2016/ff878025(v=sql.130)) — подсчитывает все события, произошедшие за время сеанса расширенных событий.
- [Целевой объект "Файл событий"](/previous-versions/sql/sql-server-2016/ff878115(v=sql.130)) — записывает все буферы в контейнер хранилища Azure.

API [трассировки событий для Windows (ETW)](/dotnet/framework/wcf/samples/etw-tracing) недоступен для расширенных событий в базе данных SQL Azure.

## <a name="restrictions"></a>Ограничения

Существует несколько отличий, связанных с безопасностью, которые связаны с облачной средой базы данных SQL Azure:

- Функция расширенных событий основана на изоляционной модели с одним клиентом. Сеанс событий в одной базе данных не может получить доступ к данным или событиям в другой базе данных.
- В контексте базы данных **master** оператор **CREATE EVENT SESSION** не вызывается.

## <a name="permission-model"></a>Модель разрешений

Чтобы вызвать оператор **CREATE EVENT SESSION**, требуется разрешение на **управление**. Владелец базы данных (dbo) имеет разрешение на **управление** .

### <a name="storage-container-authorizations"></a>Авторизации контейнера хранилища

Маркер SAS, сформированный для вашего контейнера хранилища Azure, должен указывать **rwl** для разрешений. Значение **rwl** обеспечивает следующие разрешения:

- Чтение
- запись
- Список

## <a name="performance-considerations"></a>Вопросы производительности

Существуют сценарии, в которых интенсивное использование расширенных событий может задействовать больше активной памяти, чем допустимо для сохранения работоспособности всей системы. Поэтому база данных SQL Azure динамически устанавливает и корректирует ограничения на объем активной памяти, который может накапливаться в сеансе событий. Динамические расчеты выполняются с учетом множества факторов.

При появлении сообщения о превышении максимального объема памяти можно предпринять следующие коррекционные меры:

- уменьшить количество одновременно запущенных сеансов событий;
- уменьшить объем памяти, заданный в предложении **MAX\_MEMORY**, с помощью операторов **CREATE** и **ALTER**.

### <a name="network-latency"></a>Задержка в сети

Целевой объект **Файл событий** может столкнуться с медленной работой или отказами сети при сохранении данных в большие двоичные объекты хранилища Azure. Другие события в базе данных SQL Azure могут быть отложены, пока они ожидают завершения сетевого подключения. Такая задержка может замедлить вашу работу.

- Чтобы уменьшить этот риск, старайтесь не указывать для параметра **EVENT_RETENTION_MODE** значение **NO_EVENT_LOSS** в определениях сеанса событий.

## <a name="related-links"></a>Связанные ссылки

- [Использование Azure PowerShell с хранилищем Azure](/powershell/module/az.storage/).
- [Командлеты службы хранилища Azure](/powershell/module/Azure.Storage)
- [Использование Azure PowerShell со службой хранилища Azure](/powershell/module/az.storage/)
- [Использование хранилища BLOB-объектов из .NET](../../storage/blobs/storage-quickstart-blobs-dotnet.md)
- [CREATE CREDENTIAL (Transact-SQL)](/sql/t-sql/statements/create-credential-transact-sql)
- [CREATE EVENT SESSION (Transact-SQL)](/sql/t-sql/statements/create-event-session-transact-sql)
- [Публикации в блоге Джонтана Кехайаса (Jonathan Kehayias) о расширенных событий в Microsoft SQL Server](https://www.sqlskills.com/blogs/jonathan/category/extended-events/)
- Веб-страница с *обновлениями службы* Azure с ограничением по параметру базы данных SQL Azure:
  - [https://azure.microsoft.com/updates/?service=sql-database](https://azure.microsoft.com/updates/?service=sql-database)

<!--
('lock_acquired' event.)

- Code sample for SQL Server: [Determine Which Queries Are Holding Locks](/sql/relational-databases/extended-events/determine-which-queries-are-holding-locks)
- Code sample for SQL Server: [Find the Objects That Have the Most Locks Taken on Them](/sql/relational-databases/extended-events/find-the-objects-that-have-the-most-locks-taken-on-them)
-->