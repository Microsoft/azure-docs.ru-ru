---
title: Запрос файлов Parquet с помощью бессерверного пула SQL
description: В этой статье вы узнаете, как выполнять запросы к файлам Parquet с помощью бессерверного пула SQL.
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 05/20/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: cce4c6aff986c2e8c3d879d962714e13f6b2e7ae
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97694685"
---
# <a name="query-parquet-files-using-serverless-sql-pool-in-azure-synapse-analytics"></a>Запрос файлов Parquet с помощью бессерверного пула SQL в Azure синапсе Analytics

В этой статье вы узнаете, как написать запрос с помощью бессерверного пула SQL, который будет считывать файлы Parquet.

## <a name="quickstart-example"></a>Пример краткого руководства

`OPENROWSET` функция позволяет читать содержимое файла Parquet, предоставляя URL-адрес файла.

### <a name="read-parquet-file"></a>Чтение файла Parquet

Самый простой способ увидеть содержимое `PARQUET` файла — предоставить URL-адрес файла для `OPENROWSET` работы и указать Parquet `FORMAT` . Если файл является общедоступным или удостоверение Azure AD имеет доступ к этому файлу, вы увидите содержимое файла с помощью запроса, как показано в следующем примере:

```sql
select top 10 *
from openrowset(
    bulk 'https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases/latest/ecdc_cases.parquet',
    format = 'parquet') as rows
```

Убедитесь, что у вас есть доступ к этому файлу. Если файл защищен с помощью ключа SAS или настраиваемого удостоверения Azure, необходимо настроить [учетные данные уровня сервера для входа SQL](develop-storage-files-storage-access-control.md?tabs=shared-access-signature#server-scoped-credential).

> [!IMPORTANT]
> Убедитесь, что используются параметры сортировки базы данных UTF-8 (например, `Latin1_General_100_BIN2_UTF8` ), так как строковые значения в файлах Parquet кодируются с использованием кодировки UTF-8.
> Несоответствие текстовых кодировок в файле PARQUET и параметров сортировки может привести к непредвиденным ошибкам преобразования.
> Параметры сортировки по умолчанию для текущей базы данных можно легко изменить с помощью следующей инструкции T-SQL: `alter database current collate Latin1_General_100_BIN2_UTF8`

### <a name="data-source-usage"></a>Использование источника данных

В предыдущем примере используется полный путь к файлу. В качестве альтернативы можно создать внешний источник данных с расположением, которое указывает на корневую папку хранилища, и использовать этот источник данных и относительный путь к файлу в `OPENROWSET` функции:

```sql
create external data source covid
with ( location = 'https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases' );
go
select top 10 *
from openrowset(
        bulk 'latest/ecdc_cases.parquet',
        data_source = 'covid',
        format = 'parquet'
    ) as rows
```

Если источник данных защищен с помощью ключа SAS или пользовательского удостоверения, можно настроить [источник данных с учетными данными для базы данных](develop-storage-files-storage-access-control.md?tabs=shared-access-signature#database-scoped-credential).

### <a name="explicitly-specify-schema"></a>Явное указание схемы

`OPENROWSET` позволяет явно указать столбцы, которые необходимо считать из файла с помощью `WITH` предложения:

```sql
select top 10 *
from openrowset(
        bulk 'latest/ecdc_cases.parquet',
        data_source = 'covid',
        format = 'parquet'
    ) with ( date_rep date, cases int, geo_id varchar(6) ) as rows
```

> [!IMPORTANT]
> Убедитесь, что експлиЦилти заданы некоторые параметры сортировки UTF-8 (например, `Latin1_General_100_BIN2_UTF8` ) для всех строковых столбцов в `WITH` предложении, или задайте некоторые параметры сортировки UTF-8 на уровне базы данных.
> Несоответствие кодировки текста в параметрах сортировки столбца file и String может привести к непредвиденным ошибкам преобразования.
> Параметры сортировки по умолчанию для текущей базы данных можно легко изменить с помощью следующей инструкции T-SQL: `alter database current collate Latin1_General_100_BIN2_UTF8`
> Параметры сортировки для типов столбцов можно легко задать с помощью следующего определения: `geo_id varchar(6) collate Latin1_General_100_BIN2_UTF8`

В следующих разделах показано, как выполнять запросы к различным типам файлов PARQUET.

## <a name="prerequisites"></a>Предварительные требования

Первым делом вам нужно **создать базу данных**, у которой источник данных ссылается на учетную запись хранения [набора NYC Yellow Taxi](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/). Затем инициализируйте объекты, выполнив [скрипт настройки](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) для этой базы данных. Этот сценарий установки создает источники данных, учетные данные области базы данных и форматы внешних файлов, которые используются в этих примерах.

## <a name="dataset"></a>Dataset

В этом примере используется набор данных [NYC Yellow Taxi](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/). Запросить файлы Parquet можно таким же образом, как вы [считываете CSV-файлы](query-parquet-files.md). Единственное различие состоит в том, что параметр `FILEFORMAT` должен иметь значение `PARQUET`. В примерах в этой статье показаны особенности считывания файлов Parquet.

## <a name="query-set-of-parquet-files"></a>Запрашивание набора файлов Parquet

При запросе файлов Parquet можно указать только необходимые столбцы.

```sql
SELECT
        YEAR(tpepPickupDateTime),
        passengerCount,
        COUNT(*) AS cnt
FROM  
    OPENROWSET(
        BULK 'puYear=2018/puMonth=*/*.snappy.parquet',
        DATA_SOURCE = 'YellowTaxi',
        FORMAT='PARQUET'
    ) WITH (
        tpepPickupDateTime DATETIME2,
        passengerCount INT
    ) AS nyc
GROUP BY
    passengerCount,
    YEAR(tpepPickupDateTime)
ORDER BY
    YEAR(tpepPickupDateTime),
    passengerCount;
```

## <a name="automatic-schema-inference"></a>Автоматический вывод схемы

При чтении файлов Parquet не нужно использовать предложение OPENROWSET WITH. Имена столбцов и типы данных автоматически считываются из файлов Parquet.

В приведенном ниже примере показаны возможности автоматического вывода схемы для файлов Parquet. Он возвращает число строк за сентябрь 2017 г. без указания схемы.

> [!NOTE]
> При чтении файлов Parquet указывать столбцы в предложении OPENROWSET WITH не требуется. В этом случае серверная служба запросов пула SQL будет использовать метаданные в файле Parquet и привязывать столбцы по имени.

```sql
SELECT TOP 10 *
FROM  
    OPENROWSET(
        BULK 'puYear=2018/puMonth=*/*.snappy.parquet',
        DATA_SOURCE = 'YellowTaxi',
        FORMAT='PARQUET'
    ) AS nyc
```

### <a name="query-partitioned-data"></a>Запрашивание секционированных данных

Набор данных, представленный в этом примере, разделен на отдельные вложенные папки. Вы можете выбрать конкретные разделы с помощью функции filepath. В этом примере показаны суммы по тарифам за год, месяц и payment_type за первые три месяца 2017 года.

> [!NOTE]
> Запрос пула SQL без сервера совместим с схемой секционирования Hive/Hadoop.

```sql
SELECT
        YEAR(tpepPickupDateTime),
        passengerCount,
        COUNT(*) AS cnt
FROM  
    OPENROWSET(
        BULK 'puYear=*/puMonth=*/*.snappy.parquet',
        DATA_SOURCE = 'YellowTaxi',
        FORMAT='PARQUET'
    ) nyc
WHERE
    nyc.filepath(1) = 2017
    AND nyc.filepath(2) IN (1, 2, 3)
    AND tpepPickupDateTime BETWEEN CAST('1/1/2017' AS datetime) AND CAST('3/31/2017' AS datetime)
GROUP BY
    passengerCount,
    YEAR(tpepPickupDateTime)
ORDER BY
    YEAR(tpepPickupDateTime),
    passengerCount;
```

## <a name="type-mapping"></a>Сопоставление типов

Для сопоставления типа Parquet с [сопоставлением типов](develop-openrowset.md#type-mapping-for-parquet)проверки собственного типа SQL для Parquet.

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к следующей статье, чтобы узнать, как [запросить вложенные типы Parquet](query-parquet-nested-types.md).
