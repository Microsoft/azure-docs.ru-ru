---
title: Перенос базы данных SQL Server в Базу данных SQL Azure
description: Дополнительные сведения о переносе базы данных в базу данных SQL Azure SQL Server.
keywords: миграция базы данных, миграция базы данных SQL Server, средства миграции базы данных, миграция базы данных, миграция базы данных SQL
services: sql-database
ms.service: sql-database
ms.subservice: migration
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 02/11/2019
ms.openlocfilehash: fab52fcea03a2f65c868cfac27f8a8cef115b2be
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94917653"
---
# <a name="sql-server-database-migration-to-azure-sql-database"></a>Перенос базы данных SQL Server в Базу данных SQL Azure
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

В этой статье вы узнаете о первичных методах переноса базы данных SQL Server 2005 или более поздней версии в базу данных SQL Azure. Сведения о миграции на Управляемый экземпляр Azure SQL см. в статье [Перенос экземпляра SQL Server в управляемый экземпляр Azure SQL](../migration-guides/managed-instance/sql-server-to-managed-instance-overview.md). Рекомендации по выбору параметров миграции и средств для переноса в SQL Azure см. в статье [Миграция в Azure SQL](../migration-guides/index.yml) .


## <a name="migrate-to-a-single-database-or-a-pooled-database"></a>Перенос в отдельную базу данных или базу данных в пуле

Существует два основных способа переноса базы данных SQL Server 2005 или более поздней версии в базу данных SQL Azure. Первый метод проще, но миграция происходит с простоем, который может длиться достаточно долго. Второй метод более сложен, но значительно сокращает время простоя при выполнении миграции.

В обоих случаях необходимо обеспечить совместимость базы данных-источника с Базой данных SQL Azure с помощью [Data Migration Assistant (DMA)](https://www.microsoft.com/download/details.aspx?id=53595). База данных SQL приближается к сравнению [функций](features-comparison.md) с SQL Server, за исключением проблем, связанных с операциями на уровне сервера и между базами данных. Базы данных и приложения, использующие [частично поддерживаемые или неподдерживаемые функции](transact-sql-tsql-differences-sql-server.md), требуют небольшой [доработки для устранения этих несовместимостей](migrate-to-database-from-sql-server.md#resolving-database-migration-compatibility-issues) перед миграцией базы данных SQL Server.

> [!NOTE]
> Сведения о переносе в Базу данных SQL Azure баз данных, отличных от SQL Server, в том числе Microsoft Access, Sybase, MySQL Oracle и DB2, см. в блоге, посвященном [помощнику по миграции SQL Server](https://blogs.msdn.microsoft.com/datamigration/2017/09/29/release-sql-server-migration-assistant-ssma-v7-6/).

## <a name="method-1-migration-with-downtime-during-the-migration"></a>Метод 1. Миграция с простоем

 Используйте этот метод для перехода на одну базу данных или в составе пула, если вы можете предоставить некоторое время простоя или выполняете тестовую миграцию рабочей базы данных для последующей миграции. Руководство см. в статье [Migrate a SQL Server database](../../dms/tutorial-sql-server-to-azure-sql.md) (Миграция базы данных SQL Server).

Ниже представлен общий рабочий процесс переноса базы данных SQL Server в отдельную базу данных или базу данных в пуле с помощью этого метода. Инструкции по миграции на SQL Управляемый экземпляр см. [в разделе Миграция в sql управляемый экземпляр](../managed-instance/migrate-to-instance-from-sql-server.md).

  ![Схема переноса VSSSDT](./media/migrate-to-database-from-sql-server/azure-sql-migration-sql-db.png)

1. [Оцените](/sql/dma/dma-assesssqlonprem) базу данных для обеспечения совместимости с помощью [Data Migration Assistant (DMA)](https://www.microsoft.com/download/details.aspx?id=53595) последней версии.
2. Подготовьте все необходимые исправления в виде скриптов Transact-SQL.
3. Создайте транзакционно согласованную копию переносимой базы данных-источника или остановите новые транзакции в этой базе данных на время переноса. К методам выполнения последнего действия относится отключение подключений клиентов или создание [моментального снимка базы данных](/sql/relational-databases/databases/create-a-database-snapshot-transact-sql). После переноса можно использовать репликацию транзакций для обновления перенесенных баз данных с учетом изменений, которые были внесены после точки отсчета переноса. См. раздел [Метод 2. Использование репликации транзакций](migrate-to-database-from-sql-server.md#method-2-use-transactional-replication).  
4. Разверните скрипты Transact-SQL для применения исправлений к копии базы данных.
5. [Перенос](/sql/dma/dma-migrateonpremsql) копии базы данных в новую базу данных в базе данных SQL Azure с помощью помощник по миграции данных.

> [!NOTE]
> Вместо DMA можно также использовать BACPAC-файл. См. раздел [импорт BACPAC-файла в новую базу данных в базе данных SQL Azure](database-import.md).

### <a name="optimizing-data-transfer-performance-during-migration"></a>Оптимизация производительности передачи данных во время миграции

Ниже представлены рекомендации по оптимизации производительности во время импорта.

- Выберите наивысший уровень служб и объем вычислительных ресурсов, который позволяет бюджет, чтобы увеличить производительность передачи данных. Чтобы сэкономить деньги, вы можете уменьшить производительность после завершения миграции.
- Минимизируйте расстояние между BACPAC-файлом и целевым центром обработки данных.
- Отключение автостатистики во время миграции
- Секционированные таблицы и индексы
- Удалите индексированные представления и заново создайте их по завершении миграции.
- Удалите редко запрашиваемые исторические данные в другую базу данных и перенесите эти исторические данные в отдельную базу данных в базе данных SQL Azure. Затем можно запросить эти исторические данные с помощью [эластичных запросов](elastic-query-overview.md).

### <a name="optimize-performance-after-the-migration-completes"></a>Оптимизация производительности после завершения миграции

[Обновите статистику](/sql/t-sql/statements/update-statistics-transact-sql) с использованием полной проверки после завершения миграции.

## <a name="method-2-use-transactional-replication"></a>Метод 2. Использование репликации транзакций

Если вы не можете удалить базу данных SQL Server из рабочей среды во время миграции, можно использовать SQL Serverную репликацию транзакций в качестве решения для миграции. Чтобы использовать этот метод, база данных-источник должна соответствовать [требованиям к репликации транзакций](./replication-to-sql-database.md) и быть совместимой с Базой данных SQL Azure. Получите дополнительные сведения о [настройке репликации для групп доступности AlwaysOn (SQL Server)](/sql/database-engine/availability-groups/windows/configure-replication-for-always-on-availability-groups-sql-server).

Чтобы использовать это решение, необходимо настроить базу данных в базе данных SQL Azure в качестве подписчика на экземпляр SQL Server, который вы хотите перенести. Распространитель репликации транзакций обеспечивает синхронизацию данных из базы данных (издателя) по мере возникновения новых транзакций.

При репликации транзакций все изменения данных или схемы отображаются в базе данных SQL Azure. После завершения синхронизации и подготовки к миграции измените строку подключения приложений, чтобы она указывала на базу данных. После того как репликация транзакций применила все изменения из вашей базы данных-источника и все приложения направлены к базе данных Azure, вы можете удалить репликацию транзакций. Ваша база данных в базе данных SQL Azure теперь является рабочей системой.

 ![Диаграмма SeedCloudTR](./media/migrate-to-database-from-sql-server/SeedCloudTR.png)

> [!TIP]
> Репликацию транзакций также можно использовать для миграции части вашей базы данных-источника. Публикации, которые вы реплицируете в Базу данных SQL Azure, могут быть ограничены подмножеством таблиц в реплицируемой базе данных. Для каждой реплицируемой таблицы вы можете ограничить данные подмножеством строк или подмножеством столбцов.

## <a name="migration-to-sql-database-using-transaction-replication-workflow"></a>Рабочий процесс миграции в базу данных SQL с использованием репликации транзакций

> [!IMPORTANT]
> Используйте последнюю версию SQL Server Management Studio, чтобы сохранить синхронизацию с обновлениями для Azure и базы данных SQL. Более старые версии SQL Server Management Studio не позволяют настроить Базу данных SQL как подписчика. [Обновите среду SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms).

1. Настройка распространения
   - [Использование среды SQL Server Management Studio (SSMS)](/sql/relational-databases/replication/configure-publishing-and-distribution/)
   - [Использование Transact-SQL](/sql/relational-databases/replication/configure-publishing-and-distribution/)

2. Создание публикации
   - [Использование среды SQL Server Management Studio (SSMS)](/sql/relational-databases/replication/configure-publishing-and-distribution/)
   - [Использование Transact-SQL](/sql/relational-databases/replication/configure-publishing-and-distribution/)
3. Создавать подписку
   - [Использование среды SQL Server Management Studio (SSMS)](/sql/relational-databases/replication/create-a-push-subscription/)
   - [Использование Transact-SQL](/sql/relational-databases/replication/create-a-push-subscription/)

Некоторые советы и описание различий при миграции в базу данных SQL

- Использование локального распространителя
  - Это действие влияет на производительность сервера.
  - Если снижение производительности неприемлемо, вы можете использовать другой сервер, но это усложнит управление и администрирование.
- При выборе папки моментальных снимков убедитесь, что ее емкость достаточна для хранения BCP каждой таблицы, которую требуется реплицировать.
- Создание моментального снимка блокирует связанные таблицы до ее завершения, поэтому запланируйте создание моментального снимка соответствующим образом.
- В Базе данных SQL Azure поддерживаются только принудительные подписки. Вы можете добавлять только подписчики из базы данных-источника.

## <a name="resolving-database-migration-compatibility-issues"></a>Устранение проблем совместимости при миграции базы данных

Существует множество различных проблем совместимости, которые могут возникнуть в зависимости от версии SQL Server в базе данных-источнике и сложности переносимой базы данных. В более старых версиях SQL Server имеются дополнительные проблемы совместимости. Воспользуйтесь поиском в Интернете, а также следующими ресурсами:

- [Функции базы данных SQL Server, которые не поддерживаются в Базе данных SQL Azure](transact-sql-tsql-differences-sql-server.md)
- [Неподдерживаемые функции ядра СУБД в SQL Server 2016](/sql/database-engine/discontinued-database-engine-functionality-in-sql-server)
- [Неподдерживаемые функции ядро СУБД в SQL Server 2014](/sql/database-engine/discontinued-database-engine-functionality-in-sql-server?viewFallbackFrom=sql-server-2014)
- [Неподдерживаемые функции ядро СУБД в SQL Server 2012](/previous-versions/sql/sql-server-2012/ms144262(v=sql.110))
- [Неподдерживаемые функции ядра СУБД в SQL Server 2008 R2](/previous-versions/sql/sql-server-2008-r2/ms144262(v=sql.105))
- [Неподдерживаемые функции ядра СУБД в SQL Server 2005](/previous-versions/sql/sql-server-2005/ms144262(v=sql.90))

Помимо поиска в Интернете и использования этих ресурсов, используйте [страницу Microsoft Q&A вопрос для базы данных SQL Azure](/answers/topics/azure-sql-database.html) или [StackOverflow](https://stackoverflow.com/).

> [!IMPORTANT]
> Управляемый экземпляр Azure SQL позволяет перенести существующий экземпляр SQL Server и его базы данных с минимальными затратами на проблемы с совместимостью. См. раздел [что такое управляемый экземпляр](../managed-instance/sql-managed-instance-paas-overview.md).

## <a name="next-steps"></a>Дальнейшие действия

- Воспользуйтесь скриптом в блоге разработчиков EMEA SQL Azure для [отслеживания использования базы данных TempDB во время миграции](/archive/blogs/azuresqlemea/lesson-learned-10-monitoring-tempdb-usage).
- Воспользуйтесь скриптом в блоге разработчиков EMEA SQL Azure, чтобы [отслеживать объем, занимаемый журналом транзакций в базе данных, во время миграции](/archive/blogs/azuresqlemea/lesson-learned-7-monitoring-the-transaction-log-space-of-my-database).
- Сведения о миграции из SQL Server в Базу данных SQL Azure с использованием BACPAC-файлов см. в [блоге группы консультирования клиентов SQL Server](/archive/blogs/sqlcat/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files).
- Сведения об изменении часового пояса по умолчанию для локального часового пояса см. на [этой странице](/archive/blogs/azuresqlemea/lesson-learned-4-modifying-the-default-time-zone-for-your-local-time-zone).
- Сведения об изменении языка по умолчанию в базе данных SQL Azure после миграции см. на [этой странице](/archive/blogs/azuresqlemea/lesson-learned-16-how-to-change-the-default-language-of-azure-sql-database).
