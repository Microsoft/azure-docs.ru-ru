---
title: Код кольцевого буфера XEvent
description: Содержит пример кода Transact-SQL, обеспечивающего простоту и удобство использования целевого объекта "Кольцевой буфер" в Базе данных SQL Azure.
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: sqldbrb=1
ms.devlang: PowerShell
ms.topic: sample
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: sstein
ms.date: 12/19/2018
ms.openlocfilehash: a646588616b874e40b1ed2a5a0b5e691b075075d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96487309"
---
# <a name="ring-buffer-target-code-for-extended-events-in-azure-sql-database"></a>Код целевого кольцевого буфера для расширенных событий в Базе данных SQL Azure
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

[!INCLUDE [sql-database-xevents-selectors-1-include](../../../includes/sql-database-xevents-selectors-1-include.md)]

Вам нужен полный образец кода для простой и быстрой регистрации и сообщения сведений о расширенных событиях в процессе тестирования. Самый простой целевой объект для данных расширенных событий — это [целевой объект "Кольцевой буфер"](/previous-versions/sql/sql-server-2016/bb630339(v=sql.130)).

В этой статье представлен пример кода Transact-SQL, который:

1. создает таблицу с данными для демонстрации;
2. Создает сеанс для имеющегося расширенного события, а именно **sqlserver.sql_statement_starting**.

   * Событие ограничивается операторами SQL, содержащими определенную строку обновления: **оператор LIKE '%UPDATE tabEmployee%'**.
   * Выбирает отправку выходных данных события в целевой объект типа "Кольцевой буфер", а именно **package0.ring_buffer**.
3. Запускает сеанс событий.
4. Выдает пару простых операторов SQL UPDATE.
5. Выдает инструкцию SQL SELECT для извлечения выходных данных события из кольцевого буфера.

   * **sys.dm_xe_database_session_targets** и другие динамические административные представления (DMV) объединяются.
6. Останавливает сеанс событий.
7. Удаляет целевой объект "Кольцевой буфер" для освобождения ресурсов.
8. Удаляет сеанс событий и демонстрационную таблицу.

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись и подписка Azure. Вы можете зарегистрироваться, чтобы получить [бесплатную пробную версию](https://azure.microsoft.com/pricing/free-trial/).
* Любая база данных, позволяющая создать таблицу.
  
  * При необходимости вы можете быстро [создать демонстрационную базу данных **AdventureWorksLT**](single-database-create-quickstart.md).
* SQL Server Management Studio (ssms.exe), в идеале — последняя ежемесячная версия обновления.
  Ресурсы для загрузки последней версии файла ssms.exe:
  
  * Статья [Скачивание SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms).
  * [Прямая ссылка на загрузку.](https://go.microsoft.com/fwlink/?linkid=616025)

## <a name="code-sample"></a>Пример кода

С небольшими изменениями приведенный ниже пример кода кольцевого буфера можно выполнять как в базе данных SQL Azure, так и на Microsoft SQL Server. Разница заключается в наличии узла "_database" в имени некоторых динамических административных представлений, используемых в части FROM (шаг 5). Пример:

* sys.dm_xe<strong>_database</strong>_session_targets
* sys.dm_xe_session_targets

&nbsp;

```sql
GO
----  Transact-SQL.
---- Step set 1.

SET NOCOUNT ON;
GO


IF EXISTS
    (SELECT * FROM sys.objects
        WHERE type = 'U' and name = 'tabEmployee')
BEGIN
    DROP TABLE tabEmployee;
END
GO


CREATE TABLE tabEmployee
(
    EmployeeGuid         uniqueIdentifier   not null  default newid()  primary key,
    EmployeeId           int                not null  identity(1,1),
    EmployeeKudosCount   int                not null  default 0,
    EmployeeDescr        nvarchar(256)          null
);
GO


INSERT INTO tabEmployee ( EmployeeDescr )
    VALUES ( 'Jane Doe' );
GO

---- Step set 2.


IF EXISTS
    (SELECT * from sys.database_event_sessions
        WHERE name = 'eventsession_gm_azuresqldb51')
BEGIN
    DROP EVENT SESSION eventsession_gm_azuresqldb51
        ON DATABASE;
END
GO


CREATE
    EVENT SESSION eventsession_gm_azuresqldb51
    ON DATABASE
    ADD EVENT
        sqlserver.sql_statement_starting
            (
            ACTION (sqlserver.sql_text)
            WHERE statement LIKE '%UPDATE tabEmployee%'
            )
    ADD TARGET
        package0.ring_buffer
            (SET
                max_memory = 500   -- Units of KB.
            );
GO

---- Step set 3.


ALTER EVENT SESSION eventsession_gm_azuresqldb51
    ON DATABASE
    STATE = START;
GO

---- Step set 4.


SELECT 'BEFORE_Updates', EmployeeKudosCount, * FROM tabEmployee;

UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 102;

UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 1015;

SELECT 'AFTER__Updates', EmployeeKudosCount, * FROM tabEmployee;
GO

---- Step set 5.


SELECT
    se.name                      AS [session-name],
    ev.event_name,
    ac.action_name,
    st.target_name,
    se.session_source,
    st.target_data,
    CAST(st.target_data AS XML)  AS [target_data_XML]
FROM
               sys.dm_xe_database_session_event_actions  AS ac

    INNER JOIN sys.dm_xe_database_session_events         AS ev  ON ev.event_name = ac.event_name
        AND CAST(ev.event_session_address AS BINARY(8)) = CAST(ac.event_session_address AS BINARY(8))

    INNER JOIN sys.dm_xe_database_session_object_columns AS oc
         ON CAST(oc.event_session_address AS BINARY(8)) = CAST(ac.event_session_address AS BINARY(8))

    INNER JOIN sys.dm_xe_database_session_targets        AS st
         ON CAST(st.event_session_address AS BINARY(8)) = CAST(ac.event_session_address AS BINARY(8))

    INNER JOIN sys.dm_xe_database_sessions               AS se
         ON CAST(ac.event_session_address AS BINARY(8)) = CAST(se.address AS BINARY(8))
WHERE
        oc.column_name = 'occurrence_number'
    AND
        se.name        = 'eventsession_gm_azuresqldb51'
    AND
        ac.action_name = 'sql_text'
ORDER BY
    se.name,
    ev.event_name,
    ac.action_name,
    st.target_name,
    se.session_source
;
GO

---- Step set 6.


ALTER EVENT SESSION eventsession_gm_azuresqldb51
    ON DATABASE
    STATE = STOP;
GO

---- Step set 7.


ALTER EVENT SESSION eventsession_gm_azuresqldb51
    ON DATABASE
    DROP TARGET package0.ring_buffer;
GO

---- Step set 8.


DROP EVENT SESSION eventsession_gm_azuresqldb51
    ON DATABASE;
GO

DROP TABLE tabEmployee;
GO
```

&nbsp;

## <a name="ring-buffer-contents"></a>Содержимое кольцевого буфера

Для запуска примера кода мы использовали `ssms.exe`.

Чтобы просмотреть результаты, мы щелкнули ячейку под заголовком столбца **target_data_XML**.

Затем в области результатов мы щелкнули ячейку под заголовком столбца **target_data_XML**. В результате в файле ssms.exe была создана дополнительная вкладка, на которой в виде XML-кода отображается содержимое итоговой ячейки.

Выходные данные показаны в приведенном ниже блоке. Он выглядит длинным, но содержит всего два элемента **\<event>** .

&nbsp;

```
<RingBufferTarget truncated="0" processingTime="0" totalEventsProcessed="2" eventCount="2" droppedCount="0" memoryUsed="1728">
  <event name="sql_statement_starting" package="sqlserver" timestamp="2015-09-22T15:29:31.317Z">
    <data name="state">
      <type name="statement_starting_state" package="sqlserver" />
      <value>0</value>
      <text>Normal</text>
    </data>
    <data name="line_number">
      <type name="int32" package="package0" />
      <value>7</value>
    </data>
    <data name="offset">
      <type name="int32" package="package0" />
      <value>184</value>
    </data>
    <data name="offset_end">
      <type name="int32" package="package0" />
      <value>328</value>
    </data>
    <data name="statement">
      <type name="unicode_string" package="package0" />
      <value>UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 102</value>
    </data>
    <action name="sql_text" package="sqlserver">
      <type name="unicode_string" package="package0" />
      <value>
---- Step set 4.


SELECT 'BEFORE_Updates', EmployeeKudosCount, * FROM tabEmployee;

UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 102;

UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 1015;

SELECT 'AFTER__Updates', EmployeeKudosCount, * FROM tabEmployee;
</value>
    </action>
  </event>
  <event name="sql_statement_starting" package="sqlserver" timestamp="2015-09-22T15:29:31.327Z">
    <data name="state">
      <type name="statement_starting_state" package="sqlserver" />
      <value>0</value>
      <text>Normal</text>
    </data>
    <data name="line_number">
      <type name="int32" package="package0" />
      <value>10</value>
    </data>
    <data name="offset">
      <type name="int32" package="package0" />
      <value>340</value>
    </data>
    <data name="offset_end">
      <type name="int32" package="package0" />
      <value>486</value>
    </data>
    <data name="statement">
      <type name="unicode_string" package="package0" />
      <value>UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 1015</value>
    </data>
    <action name="sql_text" package="sqlserver">
      <type name="unicode_string" package="package0" />
      <value>
---- Step set 4.


SELECT 'BEFORE_Updates', EmployeeKudosCount, * FROM tabEmployee;

UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 102;

UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 1015;

SELECT 'AFTER__Updates', EmployeeKudosCount, * FROM tabEmployee;
</value>
    </action>
  </event>
</RingBufferTarget>
```

### <a name="release-resources-held-by-your-ring-buffer"></a>Освобождение ресурсов, занятых кольцевым буфером

По завершении работы с кольцевым буфером его можно удалить и освободить таким образом ресурсы. Для этого используется оператор **ALTER**.

```sql
ALTER EVENT SESSION eventsession_gm_azuresqldb51
    ON DATABASE
    DROP TARGET package0.ring_buffer;
GO
```

Определение сеанса событий обновляется, но не удаляется. Впоследствии в сеанс событий можно добавить еще один экземпляр кольцевого буфера.

```sql
ALTER EVENT SESSION eventsession_gm_azuresqldb51
    ON DATABASE
    ADD TARGET
        package0.ring_buffer
            (SET
                max_memory = 500   -- Units of KB.
            );
```

## <a name="more-information"></a>Дополнительные сведения

Основная статья о расширенных событиях в Базе данных SQL Azure:

* [Рекомендации по работе с расширенными событиями в Базе данных SQL Azure](xevent-db-diff-from-svr.md), где сравниваются аспекты расширенных событий в Базе данных SQL Azure и Microsoft SQL Server.

Другие статьи с примерами кода для работы с расширенными событиями доступны по приведенным ниже ссылкам. Обязательно проверяйте, предназначен ли пример для Microsoft SQL Server или для базы данных SQL Azure. После этого вы сможете решить, какие поправки нужно внести в пример кода.

* Пример кода для Базы данных SQL Azure: [Код целевого файла событий для расширенных событий в Базе данных SQL Azure](xevent-code-event-file.md)

<!--
('lock_acquired' event.)

- Code sample for SQL Server: [Determine Which Queries Are Holding Locks](/sql/relational-databases/extended-events/determine-which-queries-are-holding-locks)
- Code sample for SQL Server: [Find the Objects That Have the Most Locks Taken on Them](/sql/relational-databases/extended-events/find-the-objects-that-have-the-most-locks-taken-on-them)
-->