---
title: Рекомендации по загрузке данных для выделенных пулов SQL
description: Рекомендации и оптимизация производительности для загрузки данных с помощью выделенных пулов SQL в Azure синапсе Analytics.
services: synapse-analytics
author: gaursa
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 11/20/2020
ms.author: gaursa
ms.reviewer: igorstan
ms.custom: azure-synapse
ms.openlocfilehash: 004084097e2f24a4ceb36f7a016049a574b80b42
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104594841"
---
# <a name="best-practices-for-loading-data-using-dedicated-sql-pools-in-azure-synapse-analytics"></a>Рекомендации по загрузке данных с помощью выделенных пулов SQL в Azure синапсе Analytics

В этой статье вы узнаете о рекомендациях и оптимизации производительности для загрузки данных с помощью выделенного пула SQL.

## <a name="preparing-data-in-azure-storage"></a>Подготовка данных в службе хранилища Azure

Чтобы максимально сокращать задержку, соположите свой уровень хранилища и выделенный пул SQL.

При экспорте данных в файл формата ORC могут возникнуть ошибки, связанные с нехваткой памяти в Java, если есть текстовые столбцы больших размеров. Чтобы обойти это ограничение, экспортируйте только подмножество столбцов.

Все форматы файлов имеют разные характеристики производительности. Чтобы максимально ускорить загрузку, используйте сжатые текстовые файлы с разделителями. Разница между производительностью UTF-8 и UTF-16 будет минимальной.

Разбейте большие сжатые файлы на сжатые файлы меньшего размера.

## <a name="running-loads-with-enough-compute"></a>Выполнение загрузок с достаточным объемом вычислений

Чтобы максимально ускорить загрузку, выполняйте только одно задание загрузки за раз. Если это нецелесообразно, запустите минимальное количество загрузок одновременно. Если вы планируете большое задание загрузки, рассмотрите возможность масштабирования выделенного пула SQL до загрузки.

Чтобы запускать загрузки в соответствующих вычислительных ресурсах, создайте пользователей, назначенных для выполнения загрузок. Классификация каждого пользователя, который загружается, в определенную группу рабочей нагрузки. Чтобы выполнить загрузку, войдите в качестве одного из загрузок пользователей, а затем запустите загрузку. Загрузка выполняется с группой рабочей нагрузки пользователя.  

### <a name="example-of-creating-a-loading-user"></a>Пример создания пользователя, выполняющего загрузку

В этом примере создается пользователь, классифицированный как относящийся к определенной группе рабочей нагрузки. Первым шагом является **Подключение к Master** и создание имени входа.

```sql
   -- Connect to master
   CREATE LOGIN loader WITH PASSWORD = 'a123STRONGpassword!';
```

Подключитесь к выделенному пулу SQL и создайте пользователя. В коде ниже предполагается, что вы установили подключение к базе данных с именем mySampleDataWarehouse. Здесь показано, как создать пользователя с именем Loader и получить разрешения пользователя на создание таблиц и загрузку с помощью [инструкции Copy](/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest&preserve-view=true). Затем пользователь классифицируется как принадлежащий группе рабочей нагрузки DataLoads с максимальным количеством ресурсов. 

```sql
   -- Connect to the dedicated SQL pool
   CREATE USER loader FOR LOGIN loader;
   GRANT ADMINISTER DATABASE BULK OPERATIONS TO loader;
   GRANT INSERT ON <yourtablename> TO loader;
   GRANT SELECT ON <yourtablename> TO loader;
   GRANT CREATE TABLE TO loader;
   GRANT ALTER ON SCHEMA::dbo TO loader;
   
   CREATE WORKLOAD GROUP DataLoads
   WITH ( 
       MIN_PERCENTAGE_RESOURCE = 100
       ,CAP_PERCENTAGE_RESOURCE = 100
       ,REQUEST_MIN_RESOURCE_GRANT_PERCENT = 100
    );

   CREATE WORKLOAD CLASSIFIER [wgcELTLogin]
   WITH (
         WORKLOAD_GROUP = 'DataLoads'
       ,MEMBERNAME = 'loader'
   );
```
<br><br>
>[!IMPORTANT] 
>Это очень большой пример выделения ресурсов в 100% пула SQL для одной нагрузки. Это обеспечит максимальную степень параллелизма 1. Имейте в виду, что этот параметр следует использовать только для начальной загрузки, где необходимо создать дополнительные группы рабочей нагрузки с собственными конфигурациями, чтобы баланаце ресурсы в рабочих нагрузках. 

Чтобы выполнить загрузку с ресурсами для группы рабочей нагрузки "Загрузка", войдите в систему как загрузчик и запустите загрузку.

## <a name="allowing-multiple-users-to-load-polybase"></a>Разрешение загрузки нескольких пользователей (Polybase)

Часто требуется, чтобы несколько пользователей загружали данные в выделенный пул SQL. Для загрузки с [CREATE TABLE в качестве инструкции SELECT (Transact-SQL)](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) (polybase) требуются разрешения CONTROL базы данных.  Разрешение CONTROL предоставляет возможность контроля доступа ко всем схемам.

Возможно, потребуется, чтобы не все пользователи, выполняющие загрузку, имели возможность контролировать доступ ко всем схемам. Чтобы ограничить разрешения, можно использовать инструкцию DENY CONTROL.

Пример. Рассмотрим схемы базы данных schema_A для отдела A и schema_B для отдела B. Задайте пользователей базы данных user_A и user_B в качестве пользователей для загрузки PolyBase в отделе A и B соответственно. Им обоим предоставлены разрешения CONTROL для базы данных. Теперь создатели схем A и B блокируют свои схемы, используя инструкцию DENY:

```sql
   DENY CONTROL ON SCHEMA :: schema_A TO user_B;
   DENY CONTROL ON SCHEMA :: schema_B TO user_A;
```

User_A и user_B теперь заблокированы из схемы другого отдела.

## <a name="loading-to-a-staging-table"></a>Загрузка в промежуточную таблицу

Чтобы добиться максимальной скорости загрузки для перемещения данных в выделенную таблицу пула SQL, загрузите данные в промежуточную таблицу.  Определите промежуточную таблицу в виде кучи и используйте циклический перебор как вариант распространения.

Учтите, что загрузка обычно состоит из двух этапов, в которых сначала выполняется загрузка в промежуточную таблицу, а затем данные вставляются в таблицу выделенного пула SQL. Если в рабочей таблице используется хэш-распределение, общее время загрузки и вставки можно сократить, если определить промежуточную таблицу с хэш-распределением.

Загрузка в промежуточную таблицу занимает больше времени, но на втором шаге вставки строк в рабочую таблицу перемещение данных между распределениями не происходит.

## <a name="loading-to-a-columnstore-index"></a>Загрузка в индекс columnstore

Индексы columnstore требуют огромные объемы памяти для сжатия данных в группы строк высокого качества. Чтобы обеспечить лучшее сжатие и эффективность индексов, индекс columnstore должен сжать в каждую группу строк не более 1 048 576 строк.

При нехватке памяти индекс columnstore не сможет добиться максимального сжатия. Этот сценарий, в свою очередь, влияет на производительность запросов. Дополнительные сведения см. в статье [Максимальное повышение качества группы строк для индекса columnstore](sql-data-warehouse-memory-optimizations-for-columnstore-compression.md).

- Чтобы пользователям, выполняющим загрузку, хватало памяти для максимального сжатия, они должны быть членами средних и крупных классов ресурсов.
- Загрузите достаточно строк, чтобы полностью заполнить новые группы строк. При массовой загрузке каждые 1 048 576 строк сжимаются непосредственно в индекс columnstore в виде полной группы строк. При загрузках менее 102 400 строк строки отправляются в deltastore, где они хранятся в индексе сбалансированного дерева.

> [!NOTE]
> Если загрузить слишком мало строк, они могут маршрутизироваться к deltastore и не будут сразу же сжиматься в формате columnstore.

## <a name="increase-batch-size-when-using-sqlbulkcopy-api-or-bcp"></a>Увеличение размера пакета при использовании API SqLBulkCopy или bcp

Загрузка с помощью инструкции COPY обеспечит максимальную пропускную способность с помощью выделенных пулов SQL. Если вы не можете использовать копию для загрузки и должны использовать [API SqLBulkCopy](/dotnet/api/system.data.sqlclient.sqlbulkcopy?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) или [bcp](/sql/tools/bcp-utility?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true), следует рассмотреть возможность увеличения размера пакета для повышения пропускной способности.

> [!TIP]
> Размер пакета в диапазоне от 100 до 1 МБ является рекомендуемым базовым показателем для определения оптимальной емкости пакета.

## <a name="handling-loading-failures"></a>Обработка ошибок загрузки

Загрузка с помощью внешней таблицы может завершиться ошибкой *Запрос прерван — достигнуто максимальное пороговое значение отклонения при чтении из внешнего источника*. Это сообщение означает, что внешние данные содержат "грязные" записи.

Запись данных считается "грязной", если она соответствует одному из следующих условий:

- Типы данных и количество столбцов не соответствуют определениям столбцов внешней таблицы.
- Данные не соответствуют указанному формату внешнего файла.

Чтобы устранить "грязные" записи, убедитесь в правильности определений внешней таблицы и формата внешнего файла, а также в том, что внешние данные соответствуют этим определениям.

Если подмножество записей внешних данных является «грязным», можно отклонить эти записи для запросов с помощью параметров отклонения в [CREATE External Table (Transact-SQL)](/sql/t-sql/statements/create-external-table-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

## <a name="inserting-data-into-a-production-table"></a>Вставка данных в рабочую таблицу

Одноразовую загрузку в небольшую таблицу или даже периодическую перезагрузку результатов поиска рекомендуется выполнять, используя такой [синтаксис инструкции INSERT](/sql/t-sql/statements/insert-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true): `INSERT INTO MyLookup VALUES (1, 'Type 1')`.  Тем не менее единоразовые множественные вставки не так эффективны, как массовая загрузка.

Если вы выполняете тысячи или более вставок в течение дня, объедините вставки в пакет, чтобы выполнить массовую загрузку.  Разработайте процессы для добавления единоразовых вставок в файл, а затем создайте другой процесс, который периодически загружает файл.

## <a name="creating-statistics-after-the-load"></a>Создание статистики после загрузки

Чтобы повысить производительность запросов, важно сформировать статистические данные для всех столбцов всех таблиц после первой загрузки или после значительных изменений в данных. Создание статистики может выполняться вручную, или можно включить [AUTO_CREATE_STATISTICS](sql-data-warehouse-tables-statistics.md#automatic-creation-of-statistic).

Дополнительные сведения см. в статье об [управлении статистикой](sql-data-warehouse-tables-statistics.md). В следующем примере показано, как вручную создать статистику для пяти столбцов таблицы Customer_Speed.

```sql
create statistics [SensorKey] on [Customer_Speed] ([SensorKey]);
create statistics [CustomerKey] on [Customer_Speed] ([CustomerKey]);
create statistics [GeographyKey] on [Customer_Speed] ([GeographyKey]);
create statistics [Speed] on [Customer_Speed] ([Speed]);
create statistics [YearMeasured] on [Customer_Speed] ([YearMeasured]);
```

## <a name="rotate-storage-keys-polybase"></a>Вращение ключей хранилища (Polybase)

В целях безопасности рекомендуем регулярно менять ключ доступа к хранилищу BLOB-объектов. У вас есть два ключа к хранилищу данных для учетной записи хранения больших двоичных объектов, что дает возможность перемещать ключи.

Для смены ключей учетной записи службы хранилища Azure выполните следующие действия.

Для каждой учетной записи хранения с измененным ключом выполните [ALTER DATABASE SCOPED CREDENTIAL](/sql/t-sql/statements/alter-database-scoped-credential-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

Пример

Исходный ключ создан

```sql
CREATE DATABASE SCOPED CREDENTIAL my_credential WITH IDENTITY = 'my_identity', SECRET = 'key1'
```

Замена ключа 1 ключом 2

```sql
ALTER DATABASE SCOPED CREDENTIAL my_credential WITH IDENTITY = 'my_identity', SECRET = 'key2'
```

Вносить изменения в базовые внешние источники данных больше не нужно.

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о инструкции COPY или Polybase при проектировании процесса извлечения, загрузки и преобразования (ELT) см. в статье [проектирование ELT для Azure синапсе Analytics](design-elt-data-loading.md).
- В руководстве по загрузке [используйте инструкцию Copy для загрузки данных из хранилища BLOB-объектов Azure в СИНАПСЕ SQL](./load-data-from-azure-blob-storage-using-copy.md).
- Инструкции по мониторингу загрузки данных см. в статье [Мониторинг рабочей нагрузки с помощью динамических административных представлений](sql-data-warehouse-manage-monitor.md).