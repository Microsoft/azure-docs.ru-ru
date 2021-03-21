---
title: Запрос CSV-файлов с помощью бессерверного пула SQL
description: В этой статье вы узнаете, как выполнять запросы к отдельным CSV-файлам в разных форматах с помощью бессерверного пула SQL.
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 05/20/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: f2f0cdf307e91fb40c55d4a98139bad1a5eca886
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96462594"
---
# <a name="query-csv-files"></a>Запросы к CSV-файлам

В этой статье вы узнаете, как запрашивать один CSV-файл с помощью бессерверного пула SQL в Azure синапсе Analytics. CSV-файлы могут иметь разные форматы: 

- Со строкой заголовка и без нее
- Со значениями, разделенными запятыми и символами табуляции
- Завершение строк в стиле Windows и UNIX
- Значения, не заключенные в кавычки и заключенные в кавычки, и символы экранирования

Все приведенные выше варианты будут рассмотрены ниже.

## <a name="quickstart-example"></a>Пример краткого руководства

`OPENROWSET` функция позволяет читать содержимое CSV-файла, предоставляя URL для файла.

### <a name="read-a-csv-file"></a>Чтение CSV-файла

Самый простой способ увидеть содержимое `CSV` файла — предоставить URL-адрес файла для `OPENROWSET` работы, указать csv `FORMAT` и 2,0 `PARSER_VERSION` . Если файл является общедоступным или удостоверение Azure AD имеет доступ к этому файлу, вы увидите содержимое файла с помощью запроса, как показано в следующем примере:

```sql
select top 10 *
from openrowset(
    bulk 'https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases/latest/ecdc_cases.csv',
    format = 'csv',
    parser_version = '2.0',
    firstrow = 2 ) as rows
```

Параметр `firstrow` используется для пропуска первой строки в CSV-файле, представляющей заголовок в этом случае. Убедитесь, что у вас есть доступ к этому файлу. Если файл защищен с помощью ключа SAS или пользовательского удостоверения, необходимо настроить [учетные данные на уровне сервера для входа SQL](develop-storage-files-storage-access-control.md?tabs=shared-access-signature#server-scoped-credential).

> [!IMPORTANT]
> Если файл CSV содержит символы UTF-8, убедитесь, что используются параметры сортировки базы данных UTF-8 (например, `Latin1_General_100_CI_AS_SC_UTF8` ).
> Несоответствие кодировки текста в файле и параметров сортировки может привести к непредвиденным ошибкам преобразования.
> Параметры сортировки по умолчанию для текущей базы данных можно легко изменить с помощью следующей инструкции T-SQL: `alter database current collate Latin1_General_100_CI_AI_SC_UTF8`

### <a name="data-source-usage"></a>Использование источника данных

В предыдущем примере используется полный путь к файлу. В качестве альтернативы можно создать внешний источник данных с расположением, которое указывает на корневую папку хранилища:

```sql
create external data source covid
with ( location = 'https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases' );
```

После создания источника данных можно использовать этот источник данных и относительный путь к файлу в `OPENROWSET` функции:

```sql
select top 10 *
from openrowset(
        bulk 'latest/ecdc_cases.csv',
        data_source = 'covid',
        format = 'csv',
        parser_version ='2.0',
        firstrow = 2
    ) as rows
```

Если источник данных защищен с помощью ключа SAS или пользовательского удостоверения, можно настроить [источник данных с учетными данными для базы данных](develop-storage-files-storage-access-control.md?tabs=shared-access-signature#database-scoped-credential).

### <a name="explicitly-specify-schema"></a>Явное указание схемы

`OPENROWSET` позволяет явно указать столбцы, которые необходимо считать из файла с помощью `WITH` предложения:

```sql
select top 10 *
from openrowset(
        bulk 'latest/ecdc_cases.csv',
        data_source = 'covid',
        format = 'csv',
        parser_version ='2.0',
        firstrow = 2
    ) with (
        date_rep date 1,
        cases int 5,
        geo_id varchar(6) 8
    ) as rows
```

Числа после типа данных в `WITH` предложении представляют индекс столбца в CSV-файле.

> [!IMPORTANT]
> Если файл CSV содержит символы UTF-8, убедитесь, что експлиЦилти заданы некоторые параметры сортировки UTF-8 (например `Latin1_General_100_CI_AS_SC_UTF8` ,) для всех столбцов в `WITH` предложении или заданы параметры сортировки UTF-8 на уровне базы данных.
> Несоответствие между кодировкой текста в файле и параметрами сортировки может привести к непредвиденным ошибкам преобразования.
> Параметры сортировки по умолчанию для текущей базы данных можно легко изменить с помощью следующей инструкции T-SQL: `alter database current collate Latin1_General_100_CI_AI_SC_UTF8`
> Параметры сортировки для типов столбцов можно легко задать с помощью следующего определения: `geo_id varchar(6) collate Latin1_General_100_CI_AI_SC_UTF8 8`

В следующих разделах показано, как выполнять запросы к различным типам CSV-файлов.

## <a name="prerequisites"></a>Предварительные требования

Для начала **создайте базу данных**, в которой будут созданы таблицы. Затем инициализируйте объекты, выполнив [сценарий установки](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) для этой базы данных. Этот сценарий установки создает источники данных, учетные данные области базы данных и форматы внешних файлов, которые используются в этих примерах.

## <a name="windows-style-new-line"></a>Новая строка в стиле Windows

Следующий запрос показывает, как считать CSV-файл без строки заголовка, с новой строкой в стиле Windows и столбцами с разделителями-запятыми.

Предварительный просмотр файла:

![Первые 10 строк CSV-файла без заголовка, новая строка в стиле Windows](./media/query-single-csv-file/population.png)

```sql
SELECT *
FROM OPENROWSET(
        BULK 'csv/population/population.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIELDTERMINATOR =',',
        ROWTERMINATOR = '\n'
    )
WITH (
    [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
    [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
    [year] smallint,
    [population] bigint
) AS [r]
WHERE
    country_name = 'Luxembourg'
    AND year = 2017;
```

## <a name="unix-style-new-line"></a>Новая строка в стиле Unix

Следующий запрос показывает, как считать CSV-файл без строки заголовка, с новой строкой в стиле Unix и столбцами с разделителями-запятыми. Обратите внимание на другое расположение файла по сравнению с другими примерами.

Предварительный просмотр файла:

![Первые 10 строк CSV-файла без заголовка, новая строка в стиле Unix.](./media/query-single-csv-file/population-unix.png)

```sql
SELECT *
FROM OPENROWSET(
        BULK 'csv/population-unix/population.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIELDTERMINATOR =',',
        ROWTERMINATOR = '0x0a'
    )
WITH (
    [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
    [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
    [year] smallint,
    [population] bigint
) AS [r]
WHERE
    country_name = 'Luxembourg'
    AND year = 2017;
```

## <a name="header-row"></a>Строка заголовка

Следующий запрос показывает, как считать CSV-файл со строкой заголовка, с новой строкой в стиле Unix и столбцами с разделителями-запятыми. Обратите внимание на другое расположение файла по сравнению с другими примерами.

Предварительный просмотр файла:

![Первые 10 строк CSV-файла с заголовком и новой строкой в стиле Unix.](./media/query-single-csv-file/population-unix-hdr.png)

```sql
SELECT *
FROM OPENROWSET(
        BULK 'csv/population-unix-hdr/population.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIELDTERMINATOR =',',
        FIRSTROW = 2
    )
    WITH (
        [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
        [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
        [year] smallint,
        [population] bigint
    ) AS [r]
WHERE
    country_name = 'Luxembourg'
    AND year = 2017;
```

## <a name="custom-quote-character"></a>Пользовательский символ кавычек

Следующий запрос показывает, как считать CSV-файл со строкой заголовка, с новой строкой в стиле Unix, столбцами с разделителями-запятым и значениями в кавычках. Обратите внимание на другое расположение файла по сравнению с другими примерами.

Предварительный просмотр файла:

![Первые 10 строк CSV-файла с заголовком, новой строкой в стиле Unix и значениями в кавычках.](./media/query-single-csv-file/population-unix-hdr-quoted.png)

```sql
SELECT *
FROM OPENROWSET(
        BULK 'csv/population-unix-hdr-quoted/population.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIELDTERMINATOR =',',
        ROWTERMINATOR = '0x0a',
        FIRSTROW = 2,
        FIELDQUOTE = '"'
    )
    WITH (
        [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
        [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
        [year] smallint,
        [population] bigint
    ) AS [r]
WHERE
    country_name = 'Luxembourg'
    AND year = 2017;
```

> [!NOTE]
> Этот запрос возвращает те же результаты, если пропущен параметр FIELDQUOTE, поскольку значение по умолчанию для FIELDQUOTE является двойной кавычкой.

## <a name="escape-characters"></a>Escape-символы

Следующий запрос показывает, как считать CSV-файл со строкой заголовка, с новой строкой в стиле Unix, столбцами с разделителями-запятым, а также экранированием символа для разделителя полей (запятой) внутри значений. Обратите внимание на другое расположение файла по сравнению с другими примерами.

Предварительный просмотр файла:

![Первые 10 строк CSV-файла с заголовком и новой строкой в стиле Unix, а также экранированием символа разделителя.](./media/query-single-csv-file/population-unix-hdr-escape.png)

```sql
SELECT *
FROM OPENROWSET(
        BULK 'csv/population-unix-hdr-escape/population.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIELDTERMINATOR =',',
        ROWTERMINATOR = '0x0a',
        FIRSTROW = 2,
        ESCAPECHAR = '\\'
    )
    WITH (
        [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
        [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
        [year] smallint,
        [population] bigint
    ) AS [r]
WHERE
    country_name = 'Slovenia';
```

> [!NOTE]
> Этот запрос завершится ошибкой, если ESCAPECHAR не указан, так как запятая в "Slov,enia" будет рассматриваться как разделитель полей, а не как часть названия страны или региона. "Slov,enia" будет обработано как два столбца. Таким образом, в конкретной строке будет на один столбец больше, чем в других строках, и на один столбец больше, чем определено в условии WITH.

### <a name="escape-quoting-characters"></a>Escape-символы в кавычках

В следующем запросе показано, как считать файл со строкой заголовка с новой строкой в стиле UNIX, столбцами с разделителями-запятыми и символами двойной кавычки в значениях. Обратите внимание на другое расположение файла по сравнению с другими примерами.

Предварительный просмотр файла:

![В следующем запросе показано, как считать файл со строкой заголовка с новой строкой в стиле UNIX, столбцами с разделителями-запятыми и символами двойной кавычки в значениях.](./media/query-single-csv-file/population-unix-hdr-escape-quoted.png)

```sql
SELECT *
FROM OPENROWSET(
        BULK 'csv/population-unix-hdr-escape-quoted/population.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIELDTERMINATOR =',',
        ROWTERMINATOR = '0x0a',
        FIRSTROW = 2
    )
    WITH (
        [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
        [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
        [year] smallint,
        [population] bigint
    ) AS [r]
WHERE
    country_name = 'Slovenia';
```

> [!NOTE]
> Символ кавычек нужно экранировать другим символом кавычек. Такой символ может использоваться в значении столбца только в том случае, если значение инкапсулировано с помощью символов кавычек.

## <a name="tab-delimited-files"></a>Файлы с разделителями-табуляциями

Следующий запрос показывает, как считать CSV-файл со строкой заголовка, с новой строкой в стиле Unix и столбцами с разделителями-табуляциями. Обратите внимание на другое расположение файла по сравнению с другими примерами.

Предварительный просмотр файла:

![Первые 10 строк CSV-файла с заголовком, новой строкой в стиле Unix и разделителем — знаком табуляции.](./media/query-single-csv-file/population-unix-hdr-tsv.png)

```sql
SELECT *
FROM OPENROWSET(
        BULK 'csv/population-unix-hdr-tsv/population.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIELDTERMINATOR ='\t',
        ROWTERMINATOR = '0x0a',
        FIRSTROW = 2
    )
    WITH (
        [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
        [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
        [year] smallint,
        [population] bigint
    ) AS [r]
WHERE
    country_name = 'Luxembourg'
    AND year = 2017
```

## <a name="return-a-subset-of-columns"></a>Возврат подмножества столбцов

До сих пор вы создавали схему CSV-файла с помощью WITH и перечисления всех столбцов. В запросе можно указать только те столбцы, которые действительно необходимы, используя порядковый номер для каждого столбца. Вы также пропускаете неважные столбцы.

Следующий запрос возвращает количество уникальных имен стран или регионов в файле, указывая только необходимые столбцы:

> [!NOTE]
> Взгляните на условие WITH в приведенном ниже запросе и обратите внимание на "2" (без кавычек) в конце строки, где определяется столбец *[country_name]* . Это означает, что столбец *[country_name]* является вторым столбцом в файле. Запрос будет игнорировать все столбцы файла, за исключением второго.

```sql
SELECT
    COUNT(DISTINCT country_name) AS countries
FROM OPENROWSET(
        BULK 'csv/population/population.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIELDTERMINATOR =',',
        ROWTERMINATOR = '\n'
    )
WITH (
    --[country_code] VARCHAR (5),
    [country_name] VARCHAR (100) 2
    --[year] smallint,
    --[population] bigint
) AS [r]
```

## <a name="next-steps"></a>Дальнейшие действия

В следующих статьях будет показано:

- [Запрашивание файлов Parquet](query-parquet-files.md)
- [Запрашивание папок и нескольких файлов](query-folders-multiple-csv-files.md)
