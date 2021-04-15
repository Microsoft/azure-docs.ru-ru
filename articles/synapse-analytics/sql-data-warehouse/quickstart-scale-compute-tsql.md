---
title: Краткое руководство. Масштабирование вычислительных ресурсов в выделенном пуле SQL (ранее — Хранилище данных SQL) с помощью T-SQL
description: Масштабируйте вычислительные ресурсы в выделенном пуле SQL (ранее — Хранилище данных SQL) с помощью T-SQL и SQL Server Management Studio (SSMS). Горизонтально увеличивайте масштаб вычислительных ресурсов, чтобы повысить производительность, или уменьшайте их масштаб, чтобы сократить расходы.
services: synapse-analytics
author: Antvgski
manager: craigg
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: sql-dw
ms.date: 04/17/2018
ms.author: anvang
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: 3c105fe217834dc9e0e652a42ebf3b526972b228
ms.sourcegitcommit: edc7dc50c4f5550d9776a4c42167a872032a4151
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105961376"
---
# <a name="quickstart-scale-compute-for-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics-using-t-sql"></a>Краткое руководство. Масштабирование вычислительных ресурсов в выделенном пуле SQL (ранее — Хранилище данных SQL) в Azure Synapse Analytics с помощью T-SQL

Масштабируйте вычислительные ресурсы в выделенном пуле SQL (ранее — Хранилище данных SQL) с помощью T-SQL и SQL Server Management Studio (SSMS). [Горизонтально увеличивайте масштаб вычислительных ресурсов](sql-data-warehouse-manage-compute-overview.md), чтобы повысить производительность, или уменьшайте их масштаб, чтобы сократить затраты.

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

## <a name="before-you-begin"></a>Перед началом

Скачайте и установите последнюю версию [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) (SSMS).

## <a name="create-a-dedicated-sql-pool-formerly-sql-dw"></a>Создание выделенного пула SQL (ранее — Хранилище данных SQL)

Выполните инструкции из [краткого руководства по созданию и подключению выделенного пула SQL с помощью портала Azure](create-data-warehouse-portal.md), чтобы создать выделенный пул SQL (ранее — Хранилище данных SQL) с названием **mySampleDataWarehouse**. По завершении работы с кратким руководством должно быть создано правило брандмауэра и подключение к выделенному пулу SQL (ранее — Хранилище данных SQL) из SQL Server Management Studio.

## <a name="connect-to-the-server-as-server-admin"></a>Подключение к серверу от имени администратора сервера

В этом разделе для подключения к серверу SQL Azure используется [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) (SSMS).

1. Откройте среду SQL Server Management Studio.

2. В диалоговом окне **Соединение с сервером** введите следующие данные:

   | Параметр       | Рекомендуемое значение | Описание |
   | ------------ | ------------------ | ------------------------------------------------- |
   | Тип сервера | Ядро СУБД | Это обязательное значение |
   | Имя сервера | Полное имя сервера | Вот пример: **mySampleDataWarehouseservername.database.windows.net**. |
   | Аутентификация | Проверка подлинности SQL Server | В рамках работы с этим руководством мы настроили только один тип аутентификации — аутентификацию SQL. |
   | Имя входа | Учетная запись администратора сервера | Это учетная запись, указанная при создании сервера. |
   | Пароль | Пароль для учетной записи администратора сервера | Пароль, указанный при создании сервера. |

    ![Подключение к серверу](./media/quickstart-scale-compute-tsql/connect-to-server.png)

3. Нажмите кнопку **Соединить**. В SSMS открывается окно обозревателя объектов.

4. В обозревателе объектов разверните узел **Базы данных**. Затем разверните папку **mySampleDataWarehouse**, чтобы просмотреть объекты в новой базе данных.

    ![Объекты базы данных](./media/quickstart-scale-compute-tsql/connected.png)

## <a name="view-service-objective"></a>Просмотр целевого уровня обслуживания

Параметр целевого уровня обслуживания содержит число единиц DWU для выделенного пула SQL (ранее — Хранилище данных SQL).

Чтобы просмотреть текущие значения единиц использования хранилища данных для выделенного пула SQL (ранее — Хранилище данных SQL), выполните следующие действия:

1. В разделе подключения к **mySampleDataWarehouseservername.database.windows.net** разверните узел **Системные базы данных**.
2. Щелкните правой кнопкой мыши **master** и выберите **Создать запрос**. Откроется новое окно запроса.
3. Выполните следующий запрос для выбора из динамического административного представления sys.database_service_objectives.

    ```sql
    SELECT
        db.name [Database]
    ,    ds.edition [Edition]
    ,    ds.service_objective [Service Objective]
    FROM
         sys.database_service_objectives ds
    JOIN
        sys.databases db ON ds.database_id = db.database_id
    WHERE
        db.name = 'mySampleDataWarehouse'
    ```

4. В следующих результатах показано, что для хранилища **mySampleDataWarehouse** задан целевой уровень обслуживания DW400.

    ![iew-current-dwu](./media/quickstart-scale-compute-tsql/view-current-dwu.png)

## <a name="scale-compute"></a>Масштабирование вычислительных ресурсов

В выделенном пуле SQL (ранее — Хранилище данных SQL) вы можете увеличивать и уменьшать объем вычислительных ресурсов, изменяя число единиц использования хранилища данных. В статье [Создание хранилища данных SQL Azure на портале Azure и отправка запросов к этому хранилищу данных](create-data-warehouse-portal.md) мы создали хранилище **mySampleDataWarehouse** и инициализировали его со значением 400 DWU. Ниже описаны шаги по изменению числа единиц DWU для **mySampleDataWarehouse**.

Изменить число единиц использования хранилища данных можно так:

1. Щелкните правой кнопкой мыши **master** и выберите **Создать запрос**.
2. Чтобы изменить целевой уровень обслуживания, используйте инструкцию T-SQL [ALTER DATABASE](/sql/t-sql/statements/alter-database-azure-sql-database?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true). Выполните следующий запрос, чтобы изменить значение целевого уровня обслуживания на DW300.

    ```Sql
    ALTER DATABASE mySampleDataWarehouse
    MODIFY (SERVICE_OBJECTIVE = 'DW300c');
    ```

## <a name="monitor-scale-change-request"></a>Мониторинг запроса на изменение масштаба

Чтобы просмотреть ход выполнения предыдущего запроса на изменение, можно использовать синтаксис T-SQL `WAITFORDELAY` для опроса динамического административного представления (DMV) [sys.dm_operation_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database?view=azure-sqldw-latest&preserve-view=true).

Чтобы опросить состояние изменений объекта службы, сделайте следующее

1. Щелкните правой кнопкой мыши **master** и выберите **Создать запрос**.
2. Выполните следующий запрос, чтобы опросить динамическое административное представление [sys.dm_operation_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database?view=azure-sqldw-latest&preserve-view=true).

    ```sql
    WHILE
    (
        SELECT TOP 1 state_desc
        FROM sys.dm_operation_status
        WHERE
            1=1
            AND resource_type_desc = 'Database'
            AND major_resource_id = 'mySampleDataWarehouse'
            AND operation = 'ALTER DATABASE'
        ORDER BY
            start_time DESC
    ) = 'IN_PROGRESS'
    BEGIN
        RAISERROR('Scale operation in progress',0,0) WITH NOWAIT;
        WAITFOR DELAY '00:00:05';
    END
    PRINT 'Complete';
    ```

3. Результат показывает журнал опроса состояния.

    ![Состояние операции](./media/quickstart-scale-compute-tsql/polling-output.png)

## <a name="check-dedicated-sql-pool-formerly-sql-dw-state"></a>Проверка состояния выделенного пула SQL (ранее — Хранилище данных SQL)

Если выделенный пул SQL (ранее — Хранилище данных SQL) приостановлен, подключиться к нему с помощью T-SQL невозможно. Чтобы просмотреть текущее состояние выделенного пула SQL (ранее — Хранилище данных SQL), можно использовать командлет PowerShell. Пример см. в разделе о [проверке состояния выделенного пула SQL (ранее — Хранилище данных SQL) с помощью PowerShell](quickstart-scale-compute-powershell.md#check-data-warehouse-state).

## <a name="check-operation-status"></a>Проверка состояния операции

Чтобы получить сведения о различных операциях управления, выполняемых в выделенном пуле SQL (ранее — Хранилище данных SQL), отправьте следующий запрос в динамическом административном представлении [sys.dm_operation_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true). Например, это представление возвращает сведения о разных операциях, а также их состояние со значением IN_PROGRESS или COMPLETED.

```sql
SELECT *
FROM
    sys.dm_operation_status
WHERE
    resource_type_desc = 'Database'
AND
    major_resource_id = 'mySampleDataWarehouse'
```

## <a name="next-steps"></a>Дальнейшие действия

Вы узнали, как масштабировать вычислительные ресурсы для выделенного пула SQL (ранее — Хранилище данных SQL). Чтобы узнать больше об Azure Synapse Analytics, перейдите к учебнику по загрузке данных.

> [!div class="nextstepaction"]
>[Загрузка данных в выделенный пул SQL](./load-data-from-azure-blob-storage-using-copy.md)