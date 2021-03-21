---
title: Обновление до последнего поколения выделенного пула SQL (ранее — хранилище данных SQL)
description: Обновите выделенный пул SQL Azure синапсе Analytics (ранее — хранилище данных SQL) до последнего поколения оборудования и архитектуры хранилища Azure.
services: synapse-analytics
author: mlee3gsd
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 02/19/2019
ms.author: martinle
ms.reviewer: jrasnick
ms.custom: seo-lt-2019
ms.openlocfilehash: b5a9d1781bd0498ac6ad74439b1572c52e3c345a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96459268"
---
# <a name="optimize-performance-by-upgrading-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>Оптимизация производительности путем обновления выделенного пула SQL (ранее — хранилища данных SQL) в Azure синапсе Analytics

Обновите выделенный пул SQL (ранее — хранилище данных SQL) до последнего поколения архитектуры оборудования и хранилища Azure.

## <a name="why-upgrade"></a>Зачем выполнять обновление

Теперь можно легко выполнить обновление до выделенного уровня Gen2 пула SQL (прежнее хранилище SQL DW) в портал Azure для [поддерживаемых регионов](gen2-migration-schedule.md#automated-schedule-and-region-availability-table). Если ваш регион не поддерживает самостоятельное обновление, можно перейти на поддерживаемый регион или дождаться, когда функция самостоятельного обновления станет доступна в вашем регионе. Выполните обновление сейчас, чтобы воспользоваться преимуществами последнего поколения оборудования Azure и усовершенствованной архитектуры хранилища, включая повышенную производительность, масштабируемость и неограниченное хранение по столбцам.

> [!VIDEO https://www.youtube.com/embed/9B2F0gLoyss]

> [!IMPORTANT]
> Это обновление применяется к оптимизированным для вычислений Gen1 уровням выделенных пулов SQL (форнмерли SQL DW) в [поддерживаемых регионах](gen2-migration-schedule.md#automated-schedule-and-region-availability-table).

## <a name="before-you-begin"></a>Перед началом

1. Проверьте, поддерживает ли ваш [регион](gen2-migration-schedule.md#automated-schedule-and-region-availability-table) перенос хранилища 1-го поколения в хранилище 2-го поколения. Обратите внимание на даты автоматического переноса. Чтобы избежать конфликтов с автоматизированным процессом, запланируйте ручной перенос на дату, предшествующую дате запуска автоматизированного процесса.
2. Если вы находитесь в регионе, который еще не поддерживается, регулярно проверяйте, не был ли ваш регион добавлен в список поддерживаемых, или выполните [обновление с помощью восстановления](#upgrade-from-an-azure-geographical-region-using-restore-through-the-azure-portal) в поддерживаемый регион.
3. Если ваш регион не поддерживается, выполните [обновление на портале Azure](#upgrade-in-a-supported-region-using-the-azure-portal).
4. **Выберите предлагаемый уровень производительности** для выделенного пула SQL (ранее — хранилище данных SQL) на основе текущего уровня производительности на уровне Gen1, оптимизированного для вычислений, с помощью приведенного ниже сопоставления.

   | Уровень 1-го поколения "Оптимизировано для вычислений" | Уровень 2-го поколения "Оптимизировано для вычислений" |
   | :-------------------------: | :-------------------------: |
   |            DW100            |           DW100c            |
   |            DW200            |           DW200c            |
   |            DW300            |           DW300c            |
   |            DW400            |           DW400c            |
   |            DW500            |           DW500c            |
   |            DW600            |           DW500c            |
   |           DW1000            |           DW1000c           |
   |           DW1200            |           DW1000c           |
   |           DW1500            |           DW1500c           |
   |           DW2000            |           DW2000c           |
   |           DW3000            |           DW3000c           |
   |           DW6000            |           DW6000c           |

> [!NOTE]
> Предлагаемые уровни производительности не гарантируют прямое преобразование. Например, мы рекомендуем перейти с DW600 на DW500c.

## <a name="upgrade-in-a-supported-region-using-the-azure-portal"></a>Обновление в поддерживаемом регионе с помощью портала Azure

- Миграция с GEN1 на Gen2 с помощью портал Azure является постоянной. Не существует процесса для возврата в Gen1.
- Для миграции на Gen2 должен быть выполнен выделенный пул SQL (ранее — хранилище данных SQL).

### <a name="before-you-begin"></a>Перед началом

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

- Войдите на [портал Azure](https://portal.azure.com/).
- Убедитесь, что выделенный пул SQL (ранее — хранилище данных SQL) работает — необходимо перейти на Gen2

### <a name="powershell-upgrade-commands"></a>Команды обновления PowerShell

1. Если для обновления выделенного пула SQL, оптимизированного для вычислений Gen1 (прежнее название — SQL DW), приостанавливается, [возобновите выделенный пул SQL (ранее — хранилище данных SQL)](pause-and-resume-compute-portal.md).

2. Будьте готовы к непродолжительному простою.

3. Определите все ссылки на код, связанные с уровнями производительности 1-го поколения "Оптимизировано для вычислений" и измените их на эквивалентный уровень производительности 2-го поколения "Оптимизировано для вычислений" Ниже представлены два примера, в которых нужно обновить ссылки на код перед обновлением:

   Исходная команда PowerShell 1-го поколения:

   ```powershell
   Set-AzSqlDatabase -ResourceGroupName "myResourceGroup" -DatabaseName "mySampleDataWarehouse" -ServerName "mynewserver-20171113" -RequestedServiceObjectiveName "DW300"
   ```

   Изменено на:

   ```powershell
   Set-AzSqlDatabase -ResourceGroupName "myResourceGroup" -DatabaseName "mySampleDataWarehouse" -ServerName "mynewserver-20171113" -RequestedServiceObjectiveName "DW300c"
   ```

   > [!NOTE]
   > -RequestedServiceObjectiveName "DW300" изменено на -RequestedServiceObjectiveName "DW300 **c**"
   >

   Исходная команда T-SQL 1-го поколения:

   ```SQL
   ALTER DATABASE mySampleDataWarehouse MODIFY (SERVICE_OBJECTIVE = 'DW300') ;
   ```

   Изменено на:

   ```sql
   ALTER DATABASE mySampleDataWarehouse MODIFY (SERVICE_OBJECTIVE = 'DW300c') ;
   ```

   > [!NOTE]
   > SERVICE_OBJECTIVE = 'DW300' изменено на SERVICE_OBJECTIVE = 'DW300 **c**'

## <a name="start-the-upgrade"></a>Начало обновления

1. Перейдите к выделенному пулу SQL, оптимизированному для вычислений Gen1 (ранее — хранилище данных SQL), в портал Azure. Если для обновления выделенного пула SQL, оптимизированного для вычислений Gen1 (прежнее название — SQL DW), приостанавливается, [возобновите выделенный пул SQL](pause-and-resume-compute-portal.md).
2. На вкладке Tasks (задачи) выберите **Upgrade to Gen2 (обновить до** карты): ![ Upgrade_1](./media/upgrade-to-latest-generation/upgrade-to-gen2-1.png)

   > [!NOTE]
   > Если карта **Повысить до Gen2** не отображается на вкладке "Задачи", для типа вашей подписки действуют ограничения в текущем регионе.
   > [Отправьте запрос в службу поддержки](sql-data-warehouse-get-started-create-support-ticket.md) , чтобы получить одобренную подписку.

3. Перед обновлением убедитесь, что рабочая нагрузка завершена и приостановлена. В течение нескольких минут вы получите простой, прежде чем выделенный пул SQL (ранее — хранилище данных SQL) вернется в режим "в сети" как выделенный пул SQL, оптимизированный для вычислений Gen2 (ранее — хранилище данных SQL). **Выберите обновление**:

   ![Upgrade_2](./media/upgrade-to-latest-generation/upgrade-to-gen2-2.png)

4. **Наблюдайте за процессом обновления**, проверяя состояние на портале Azure.

   ![Upgrade3](./media/upgrade-to-latest-generation/upgrade-to-gen2-3.png)

   Первым шагом процесса обновления является масштабирование ("Обновление — автономный режим"), при котором все сеансы будут прерваны, а соединение разорвано.

   Второй шаг процесса обновления — перенос данных ("Обновление — в сети"). Перенос данных является малозаметным фоновым сетевым процессом. Он медленно перемещает данные столбцов из старой архитектуры службы хранилища в новую архитектуру службы хранилища, использующую локальный кэш SSD. В течение этого времени выделенный пул SQL (ранее — хранилище данных SQL) будет подключен к сети для выполнения запросов и загрузки. При этом все данные будут доступны для запросов, вне зависимости от того, были ли они перенесены. Перенос данных происходит с разной скоростью в зависимости от их размера, уровня производительности и количества сегментов columnstore.

5. **Необязательная рекомендация:** После завершения операции масштабирования можно ускорить фоновый процесс переноса данных. Вы можете сразу инициировать перемещение данных, выполнив инструкцию [Alter Index rebuild](sql-data-warehouse-tables-index.md) для всех основных таблиц columnstore, которые вы сможете запрашивать при наличии более высокого целевого уровня обслуживания и оптимального класса ресурсов. Эта операция является **автономной** по сравнению с малозаметным фоновым процессом, который может потребовать несколько часов в зависимости от количества и размеров таблиц. Тем не менее после завершения этой операции перенос данных будет происходить намного быстрее благодаря новой улучшенной архитектуре хранилища с высококачественными группами строк.

> [!NOTE]
> Операция "Alter Index rebuild" — это автономная операция. Таблицы будут недоступны до завершения повторной сборки.

Следующий запрос формирует необходимые команды Alter Index Rebuild для ускорения процесса переноса данных.

```sql
SELECT 'ALTER INDEX [' + idx.NAME + '] ON ['
       + Schema_name(tbl.schema_id) + '].['
       + Object_name(idx.object_id) + '] REBUILD ' + ( CASE
                                                         WHEN (
                                                     (SELECT Count(*)
                                                      FROM   sys.partitions
                                                             part2
                                                      WHERE  part2.index_id
                                                             = idx.index_id
                                                             AND
                                                     idx.object_id =
                                                     part2.object_id)
                                                     > 1 ) THEN
              ' PARTITION = '
              + Cast(part.partition_number AS NVARCHAR(256))
              ELSE ''
                                                       END ) + '; SELECT ''[' +
              idx.NAME + '] ON [' + Schema_name(tbl.schema_id) + '].[' +
              Object_name(idx.object_id) + '] ' + (
              CASE
                WHEN ( (SELECT Count(*)
                        FROM   sys.partitions
                               part2
                        WHERE
                     part2.index_id =
                     idx.index_id
                     AND idx.object_id
                         = part2.object_id) > 1 ) THEN
              ' PARTITION = '
              + Cast(part.partition_number AS NVARCHAR(256))
              + ' completed'';'
              ELSE ' completed'';'
                                                    END )
FROM   sys.indexes idx
       INNER JOIN sys.tables tbl
               ON idx.object_id = tbl.object_id
       LEFT OUTER JOIN sys.partitions part
                    ON idx.index_id = part.index_id
                       AND idx.object_id = part.object_id
WHERE  idx.type_desc = 'CLUSTERED COLUMNSTORE';
```

## <a name="upgrade-from-an-azure-geographical-region-using-restore-through-the-azure-portal"></a>Обновление из географического региона Azure с помощью восстановления на портале Azure

## <a name="create-a-user-defined-restore-point-using-the-azure-portal"></a>Создание определяемой пользователем точки восстановления с помощью портала Azure

1. Войдите на [портал Azure](https://portal.azure.com/).

2. Перейдите к выделенному пулу SQL (ранее SQL DW), для которого нужно создать точку восстановления.

3. В верхней области колонки "Обзор" выберите **Новая точка восстановления**.

    ![Новая точка восстановления](./media/upgrade-to-latest-generation/creating_restore_point_0.png)

4. Укажите имя точки восстановления.

    ![Имя точки восстановления](./media/upgrade-to-latest-generation/creating_restore_point_1.png)

## <a name="restore-an-active-or-paused-database-using-the-azure-portal"></a>Восстановление активной или приостановленной базы данных с помощью портала Azure

1. Войдите на [портал Azure](https://portal.azure.com/).
2. Перейдите к выделенному пулу SQL (ранее SQL DW), из которого требуется выполнить восстановление.
3. В верхней области колонки "Обзор" выберите **Восстановление**.

    ![ Обзор восстановления](./media/upgrade-to-latest-generation/restoring_0.png)

4. Выберите **точки автоматического восстановления** или **определенные пользователем точки восстановления**. Для определяемых пользователем точек восстановления **выберите определенную пользователем точку восстановления** или **Создайте новую, определенную пользователем точку восстановления**. Для сервера выберите **создать** и выберите сервер в Gen2 поддерживаемом географическом регионе.

    ![Точки автоматического восстановления](./media/upgrade-to-latest-generation/restoring_1.png)

## <a name="restore-from-an-azure-geographical-region-using-powershell"></a>Восстановление из географического региона Azure с помощью PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Чтобы восстановить базу данных, используйте командлет [RESTORE-азсклдатабасе](/powershell/module/az.sql/restore-azsqldatabase?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) .

> [!NOTE]
> Можно выполнить геовосстановление в хранилище 2-го поколения! Для этого в качестве необязательного параметра укажите имя ServiceObjectiveName 2-го поколения (например, DW1000 **c**).

1. Откройте Windows PowerShell.
2. Подключитесь к своей учетной записи Azure и выведите список всех подписок, связанных с ней.
3. Выберите подписку, содержащую базу данных, которую нужно восстановить.
4. Получите базу данных, которую требуется восстановить.
5. Создайте запрос на восстановление для базы данных, указав ServiceObjectiveName 2-го поколения.
6. Проверьте состояние геовосстановленной базы данных.

```Powershell
Connect-AzAccount
Get-AzSubscription
Select-AzSubscription -SubscriptionName "<Subscription_name>"

# Get the database you want to recover
$GeoBackup = Get-AzSqlDatabaseGeoBackup -ResourceGroupName "<YourResourceGroupName>" -ServerName "<YourServerName>" -DatabaseName "<YourDatabaseName>"

# Recover database
$GeoRestoredDatabase = Restore-AzSqlDatabase –FromGeoBackup -ResourceGroupName "<YourResourceGroupName>" -ServerName "<YourTargetServer>" -TargetDatabaseName "<NewDatabaseName>" –ResourceId $GeoBackup.ResourceID -ServiceObjectiveName "<YourTargetServiceLevel>" -RequestedServiceObjectiveName "DW300c"

# Verify that the geo-restored database is online
$GeoRestoredDatabase.status
```

> [!NOTE]
> Чтобы настроить базу данных после восстановления, см. раздел [Настройка базы данных после восстановления](../../azure-sql/database/disaster-recovery-guidance.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#configure-your-database-after-recovery).

Восстановленная база данных будет поддерживать прозрачное шифрование данных, если исходная база данных поддерживает прозрачное шифрование данных.

Если у вас возникли проблемы с выделенным пулом SQL, создайте [запрос в службу поддержки](sql-data-warehouse-get-started-create-support-ticket.md) и ссылку «Gen2 Upgrade» в качестве возможной причины.

## <a name="next-steps"></a>Дальнейшие действия

Обновленный выделенный пул SQL (ранее — хранилище данных SQL) находится в сети. Чтобы воспользоваться преимуществами новой, усовершенствованной архитектуры, см. [Классы ресурсов для управления рабочей нагрузкой](resource-classes-for-workload-management.md).
