---
title: Начало работы с временными таблицами
description: Узнайте, как приступить к использованию временных таблиц в базе данных SQL Azure и Управляемый экземпляр Azure SQL.
services: sql-database
ms.service: sql-db-mi
ms.subservice: development
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: how-to
author: bonova
ms.author: bonova
ms.reviewer: sstein
ms.date: 06/26/2019
ms.openlocfilehash: ea037d12417c8fad9d80b77df69285ed2c8df31b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91618664"
---
# <a name="getting-started-with-temporal-tables-in-azure-sql-database-and-azure-sql-managed-instance"></a>Начало работы с временными таблицами в базе данных SQL Azure и Azure SQL Управляемый экземпляр
[!INCLUDE[appliesto-sqldb-sqlmi](includes/appliesto-sqldb-sqlmi.md)]

Временные таблицы — это возможность программирования базы данных SQL Azure и Управляемый экземпляр Azure SQL, которая позволяет отслеживать и анализировать полную историю изменений в данных без необходимости написания пользовательского кода. Временные таблицы сохраняют данные, тесно связанные с контекстом времени, чтобы хранимые факты можно было интерпретировать как допустимые только в течение определенного периода. Это свойство временных таблиц позволяет эффективно проводить анализ на основе времени и получать ценные сведения о развитии данных.

## <a name="temporal-scenario"></a>Временный сценарий

В этой статье приведены инструкции по использованию временных таблиц в сценарии приложения. Предположим, что вам нужно отслеживать активность пользователей на новом веб-сайте, который разрабатывался с нуля, или на существующем веб-сайте, который вы хотите дополнить возможностями анализа активности пользователей. В этом упрощенном примере предполагается, что количество посещенных веб-страниц в течение определенного периода времени является индикатором, который необходимо записать и отслеживать в базе данных веб-сайта, размещенной в базе данных SQL Azure или Управляемый экземпляр Azure SQL. Цель исторического анализа активности пользователей — получить входные данные для переработки веб-сайта и улучшения его взаимодействия с посетителями.

Модель базы данных в этом сценарии очень простая: метрика активности пользователей представлена одним целочисленным полем, **PageVisited**, а ее значение записывается вместе с основными сведениями в профиле пользователя. Кроме того, для анализа с учетом времени следует выделить набор строк для каждого пользователя, где каждая строка представляет число страниц, посещенных определенным пользователем в течение определенного периода времени.

![схема](./media/temporal-tables/AzureTemporal1.png)

К счастью, вам не нужно программировать хранение этой информации об активности в приложении. В случае с временными таблицами этот процесс автоматизирован, обеспечивая полную гибкость во время разработки веб-сайтов и больше времени для того, чтобы сосредоточиться на анализе данных. Единственное, что необходимо сделать, — убедиться, что таблица **WebSiteInfo** настроена как [временная с системным управлением версиями](/sql/relational-databases/tables/temporal-tables#what-is-a-system-versioned-temporal-table). Ниже описаны точные действия для использования временных таблиц в этом сценарии.

## <a name="step-1-configure-tables-as-temporal"></a>Шаг 1. Настройка таблиц в качестве временных

В зависимости от того, начинаете вы разработку новых приложений или обновляете существующее приложение, вы создадите временные таблицы или измените существующие, добавляя в них временные атрибуты. В общем случае может потребоваться сделать и то, и другое. Выполните эти действия с помощью [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS), [SQL Server Data Tools](/sql/ssdt/download-sql-server-data-tools-ssdt) (SSDT), [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio)или любого другого средства разработки Transact-SQL.

> [!IMPORTANT]
> Рекомендуется всегда использовать последнюю версию Management Studio для синхронизации с обновлениями базы данных SQL Azure и Azure SQL Управляемый экземпляр. [Обновите среду SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms).

### <a name="create-new-table"></a>Создание новой таблицы

Используйте пункт контекстного меню "New System-Versioned Table" (Новая таблица с системным управлением версиями) в обозревателе объектов SSMS, чтобы открыть редактор запросов с шаблоном сценария временной таблицы, а затем щелкните "Указать значения для параметров шаблона" (Ctrl+Shift+M) для заполнения шаблона.

![SSMSNewTable](./media/temporal-tables/AzureTemporal2.png)

В SSDT при добавлении новых элементов в проект базы данных выберите шаблон "Темпоральная таблица (с системным управлением версиями)". Откроется конструктор таблиц, в котором вы сможете легко указать макет таблицы.

![SSDTNewTable](./media/temporal-tables/AzureTemporal3.png)

Временную таблицу также можно создать, непосредственно указав инструкции Transact-SQL, как показано в следующем примере. Обратите внимание, что обязательными элементами каждой временной таблицы являются определение PERIOD и предложение SYSTEM_VERSIONING со ссылкой на другую таблицу пользователя, в которой будут храниться исторические версии строк.

```sql
CREATE TABLE WebsiteUserInfo
(  
    [UserID] int NOT NULL PRIMARY KEY CLUSTERED
  , [UserName] nvarchar(100) NOT NULL
  , [PagesVisited] int NOT NULL
  , [ValidFrom] datetime2 (0) GENERATED ALWAYS AS ROW START
  , [ValidTo] datetime2 (0) GENERATED ALWAYS AS ROW END
  , PERIOD FOR SYSTEM_TIME (ValidFrom, ValidTo)
 )  
 WITH (SYSTEM_VERSIONING = ON (HISTORY_TABLE = dbo.WebsiteUserInfoHistory));
```

При создании временной таблицы с системным управлением версиями автоматически создается сопутствующая таблица журнала с конфигурацией по умолчанию. Таблица журнала по умолчанию содержит кластеризованный индекс сбалансированного дерева в столбцах периода (конец и начало) с включенным сжатием страниц. Эта конфигурация оптимальна для большинства сценариев, в которых используются временные таблицы, особенно для [аудита данных](/sql/relational-databases/tables/temporal-table-usage-scenarios#enabling-system-versioning-on-a-new-table-for-data-audit).

В данном случае наша цель — выполнить анализ тенденций с учетом времени, используя журнал данных за длительный период и большие наборы данных, поэтому в качестве хранилища для таблицы журнала выбран кластеризованный индекс columnstore. Кластеризованный индекс columnstore обеспечивает очень хорошее сжатие и производительность аналитических запросов. Временные таблицы предоставляют гибкие возможности настройки индексов в текущей и темпоральной таблицах полностью независимо друг от друга.

> [!NOTE]
> Индексы columnstore доступны на уровнях критически важный для бизнеса, общего назначения и Premium, а также на уровне Standard, S3 и выше.

В следующем сценарии показано, как индекс по умолчанию в таблице журнала можно изменить на кластеризованный индекс columnstore.

```sql
CREATE CLUSTERED COLUMNSTORE INDEX IX_WebsiteUserInfoHistory
ON dbo.WebsiteUserInfoHistory
WITH (DROP_EXISTING = ON);
```

Временные таблицы представлены в обозревателе объектов с конкретным значком для упрощения идентификации, а таблица журнала отображается как дочерний узел.

![AlterTable](./media/temporal-tables/AzureTemporal4.png)

### <a name="alter-existing-table-to-temporal"></a>Преобразование существующей таблицы во временную

Рассмотрим альтернативный сценарий, в котором таблица WebsiteUserInfo уже существует, но не была предназначена для хранения журнала изменений. В этом случае можно просто расширить существующую таблицу, превратив ее во временную, как показано в следующем примере.

```sql
ALTER TABLE WebsiteUserInfo
ADD
    ValidFrom datetime2 (0) GENERATED ALWAYS AS ROW START HIDDEN  
        constraint DF_ValidFrom DEFAULT DATEADD(SECOND, -1, SYSUTCDATETIME())
    , ValidTo datetime2 (0)  GENERATED ALWAYS AS ROW END HIDDEN
        constraint DF_ValidTo DEFAULT '9999.12.31 23:59:59.99'
    , PERIOD FOR SYSTEM_TIME (ValidFrom, ValidTo);

ALTER TABLE WebsiteUserInfo  
SET (SYSTEM_VERSIONING = ON (HISTORY_TABLE = dbo.WebsiteUserInfoHistory));
GO

CREATE CLUSTERED COLUMNSTORE INDEX IX_WebsiteUserInfoHistory
ON dbo.WebsiteUserInfoHistory
WITH (DROP_EXISTING = ON);
```

## <a name="step-2-run-your-workload-regularly"></a>Шаг 2. Регулярный запуск рабочей нагрузки

Основным преимуществом временных таблиц является то, что вам не нужно изменять или настраивать веб-сайт любым способом для отслеживания изменений. После создания временные таблицы прозрачно сохраняют предыдущие версии строк каждый раз при выполнении изменений в данных.

Чтобы использовать автоматическое отслеживание изменений в этом конкретном сценарии, давайте просто обновляйте столбец **пажесвиситед** каждый раз, когда пользователь завершает свой сеанс на веб-сайте:

```sql
UPDATE WebsiteUserInfo  SET [PagesVisited] = 5
WHERE [UserID] = 1;
```

Важно отметить, что запросу на обновление не нужно знать точное время самой операции или то, как будут сохранены данные журнала для последующего анализа. Оба аспекта автоматически обрабатываются базой данных SQL Azure и Управляемый экземпляр Azure SQL. Следующая схема иллюстрирует, как при каждом обновлении создаются данные журнала.

![TemporalArchitecture](./media/temporal-tables/AzureTemporal5.png)

## <a name="step-3-perform-historical-data-analysis"></a>Шаг 3. Анализ данных журнала

Теперь, когда временное управления версиями системой включено, анализ данных журнала — дело всего одного запроса. В этой статье мы приведем несколько примеров распространенных сценариев анализа. Чтобы изучить все подробности, ознакомьтесь с возможностями предложения [FOR SYSTEM_TIME](/sql/relational-databases/tables/temporal-tables#how-do-i-query-temporal-data).

Чтобы просмотреть 10 лидирующих пользователей час назад, упорядоченных по числу посещаемых веб-страниц, выполните этот запрос.

```sql
DECLARE @hourAgo datetime2 = DATEADD(HOUR, -1, SYSUTCDATETIME());
SELECT TOP 10 * FROM dbo.WebsiteUserInfo FOR SYSTEM_TIME AS OF @hourAgo
ORDER BY PagesVisited DESC
```

Вы легко можете изменить этот запрос, чтобы анализировать посещения сайта день назад, месяц назад или любой момент в прошлом.

Чтобы выполнить простой статистический анализ за предыдущий день, используйте следующий пример.

```sql
DECLARE @twoDaysAgo datetime2 = DATEADD(DAY, -2, SYSUTCDATETIME());
DECLARE @aDayAgo datetime2 = DATEADD(DAY, -1, SYSUTCDATETIME());

SELECT UserID, SUM (PagesVisited) as TotalVisitedPages, AVG (PagesVisited) as AverageVisitedPages,
MAX (PagesVisited) AS MaxVisitedPages, MIN (PagesVisited) AS MinVisitedPages,
STDEV (PagesVisited) as StDevViistedPages
FROM dbo.WebsiteUserInfo
FOR SYSTEM_TIME BETWEEN @twoDaysAgo AND @aDayAgo
GROUP BY UserId
```

Для поиска активности конкретного пользователя в течение периода времени используйте предложение CONTAINED IN.

```sql
DECLARE @hourAgo datetime2 = DATEADD(HOUR, -1, SYSUTCDATETIME());
DECLARE @twoHoursAgo datetime2 = DATEADD(HOUR, -2, SYSUTCDATETIME());
SELECT * FROM dbo.WebsiteUserInfo
FOR SYSTEM_TIME CONTAINED IN (@twoHoursAgo, @hourAgo)
WHERE [UserID] = 1;
```

Графическое представление особенно удобно для временных запросов, так как можно отобразить тенденции и закономерности использования доступным и интуитивно понятным способом.

![TemporalGraph](./media/temporal-tables/AzureTemporal6.png)

## <a name="evolving-table-schema"></a>Развитие схемы таблицы

Как правило, во время разработки приложения приходится менять схему временной таблицы. Для этого просто выполните регулярные инструкции ALTER TABLE и базу данных SQL Azure или Azure SQL Управляемый экземпляр соответствующим образом распространить изменения в таблице журнала. В следующем сценарии показано, как добавить дополнительный атрибут для отслеживания.

```sql
/*Add new column for tracking source IP address*/
ALTER TABLE dbo.WebsiteUserInfo
ADD  [IPAddress] varchar(128) NOT NULL CONSTRAINT DF_Address DEFAULT 'N/A';
```

Аналогичным образом можно изменить определение столбца при активной рабочей нагрузке.

```sql
/*Increase the length of name column*/
ALTER TABLE dbo.WebsiteUserInfo
    ALTER COLUMN  UserName nvarchar(256) NOT NULL;
```

Наконец, можно удалить столбец, который больше не нужен.

```sql
/*Drop unnecessary column */
ALTER TABLE dbo.WebsiteUserInfo
    DROP COLUMN TemporaryColumn;
```

В качестве альтернативы можно использовать последнюю версию [SSDT](/sql/ssdt/download-sql-server-data-tools-ssdt) , чтобы изменить схему временной таблицы при активном подключении к базе данных (интерактивный режим) или непосредственно в проекте базы данных (автономный режим).

## <a name="controlling-retention-of-historical-data"></a>Управление периодом удержания данных журнала

При использовании временных таблиц с системным управлением версиями таблица журнала может увеличить размер базы данных значительнее, чем обычные таблицы. Большой и постоянно растущий объем таблицы журнала может стать проблемой не только из-за затрат на хранение. Кроме этого, он может повлиять на производительность при выполнении темпоральных запросов. Таким образом, разработка политики хранения данных для управления данными в таблице журнала является важным аспектом планирования и управления жизненным циклом всех темпоральных таблиц. Используя базу данных SQL Azure и Управляемый экземпляр Azure SQL, вы можете использовать следующие подходы для управления историческими данными в темпоральной таблице:

- [секционирование таблиц;](/sql/relational-databases/tables/manage-retention-of-historical-data-in-system-versioned-temporal-tables#using-table-partitioning-approach)
- [пользовательский сценарий очистки.](/sql/relational-databases/tables/manage-retention-of-historical-data-in-system-versioned-temporal-tables#using-custom-cleanup-script-approach)

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о временных таблицах см. в разделе Извлечение [временных таблиц](/sql/relational-databases/tables/temporal-tables).
- Посетите канал 9, чтобы услышать [историю успеха реализации клиента](https://channel9.msdn.com/Blogs/jsturtevant/Azure-SQL-Temporal-Tables-with-RockStep-Solutions) , и просмотрите [действующую временную демонстрацию](https://channel9.msdn.com/Shows/Data-Exposed/Temporal-in-SQL-Server-2016).
