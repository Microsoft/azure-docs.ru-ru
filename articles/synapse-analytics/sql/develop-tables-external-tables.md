---
title: Использование внешних таблиц в Synapse SQL
description: Чтение или запись файлов данных в Synapse SQL
services: synapse-analytics
author: julieMSFT
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 05/07/2020
ms.author: jrasnick
ms.reviewer: jrasnick
ms.openlocfilehash: 78e4b35feb4e830a9f4335614a55d49ca90cd791
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101667638"
---
# <a name="use-external-tables-with-synapse-sql"></a>Использование внешних таблиц в Synapse SQL

Внешняя таблица указывает на данные, расположенные в Hadoop, большом двоичном объекте в службе хранилища Azure или Azure Data Lake Storage. Внешние таблицы используются для чтения данных из файлов или записи данных в файлы, расположенные в службе хранилища Azure. В Synapse SQL внешние таблицы можно использовать для чтения данных в выделенном пуле SQL или бессерверном пуле SQL, а также записи данных в такие пулы.

## <a name="external-tables-in-dedicated-sql-pool-and-serverless-sql-pool"></a>Внешние таблицы в выделенном пуле SQL и бессерверном пуле SQL

### <a name="dedicated-sql-pool"></a>[Выделенный пул SQL](#tab/sql-pool) 

В выделенном пуле SQL внешняя таблица используется в следующих целях.

- Запрос по данным в хранилище BLOB-объектов Azure и Azure Data Lake Storage 2-го поколения с использованием инструкций Transact-SQL.
- Импорт данных из Хранилища BLOB-объектов Azure и Azure Data Lake Storage с сохранением в выделенный пул SQL.

При использовании в сочетании с инструкцией [CREATE TABLE AS SELECT](../sql-data-warehouse/sql-data-warehouse-develop-ctas.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) выбранные из внешней таблицы данные импортируются в таблицу в пуле SQL. В дополнение к [инструкции COPY](/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest&preserve-view=true) внешние таблицы можно использовать для загрузки данных. 

Учебник по загрузке данных см. в статье [Загрузка данных из хранилища BLOB-объектов Azure в хранилище данных SQL Azure с помощью PolyBase](../sql-data-warehouse/load-data-from-azure-blob-storage-using-copy.md?bc=%2fazure%2fsynapse-analytics%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fsynapse-analytics%2ftoc.json).

### <a name="serverless-sql-pool"></a>[Бессерверный пул SQL](#tab/sql-on-demand)

В бессерверном пуле SQL внешняя таблица используется в следующих целях:

- Запрос по данным в хранилище BLOB-объектов Azure и Azure Data Lake Storage с использованием инструкций Transact-SQL.
- Сохранение результатов запросов, выполняемых в бессерверном пуле SQL, в файлах в Хранилище BLOB-объектов Azure или Azure Data Lake Storage с помощью [CETAS](develop-tables-cetas.md)

Вы можете создать внешние таблицы в бессерверном пуле SQL, выполнив следующие действия:

1. CREATE EXTERNAL DATA SOURCE
2. CREATE EXTERNAL FILE FORMAT
3. CREATE EXTERNAL TABLE

---

### <a name="security"></a>Безопасность

Для чтения данных у вас должно быть разрешение `SELECT` для внешней таблицы.
Внешняя таблица обращается к базовой службе хранилища Azure с использованием учетных данных базы данных, определенных в источнике данных. При обращении внешняя таблица руководствуется следующими правилами:
- Источник данных без учетных данных позволяет внешним таблицам получать доступ к общедоступным файлам в службе хранилища Azure.
- У источника данных могут быть учетные данные, позволяющие внешним таблицам получать доступ только к файлам в службе хранилища Azure с помощью маркера SAS или управляемого удостоверения рабочей области. Примеры см. в статье [Управление доступом к учетной записи хранения в SQL по запросу (предварительная версия)](develop-storage-files-storage-access-control.md#examples).

> [!IMPORTANT]
> В выделенном пуле SQL источник данных, созданный без учетных данных, позволяет пользователю Azure AD получать доступ к файлам хранилища с помощью удостоверения Azure AD. В бессерверном пуле SQL необходимо создать источник данных с учетными данными базы данных, которые имеют свойство `IDENTITY='User Identity'`. Примеры см. [здесь](develop-storage-files-storage-access-control.md#examples).

## <a name="create-external-data-source"></a>CREATE EXTERNAL DATA SOURCE

Внешние источники данных используются для подключения к учетным записям хранения. Полная документация приводится [здесь](/sql/t-sql/statements/create-external-data-source-transact-sql?view=azure-sqldw-latest&preserve-view=true).

### <a name="syntax-for-create-external-data-source"></a>Синтаксис инструкции CREATE EXTERNAL DATA SOURCE

#### <a name="dedicated-sql-pool"></a>[Выделенный пул SQL](#tab/sql-pool)

```syntaxsql
CREATE EXTERNAL DATA SOURCE <data_source_name>
WITH
(    LOCATION         = '<prefix>://<path>'
     [, CREDENTIAL = <database scoped credential> ]
     , TYPE = HADOOP
)
[;]
```

#### <a name="serverless-sql-pool"></a>[Бессерверный пул SQL](#tab/sql-on-demand)

```syntaxsql
CREATE EXTERNAL DATA SOURCE <data_source_name>
WITH
(    LOCATION         = '<prefix>://<path>'
     [, CREDENTIAL = <database scoped credential> ]
)
[;]
```

---

### <a name="arguments-for-create-external-data-source"></a>Аргументы инструкции CREATE EXTERNAL DATA SOURCE

data_source_name

Задает определенное пользователем имя для источника данных. В базе данных это имя должно быть уникальным.

#### <a name="location"></a>Расположение
LOCATION = `'<prefix>://<path>'` предоставляет протокол и путь подключения к внешнему источнику данных. В расположении можно использовать следующие шаблоны:

| Внешний источник данных        | Префикс расположения | Путь к расположению                                         |
| --------------------------- | --------------- | ----------------------------------------------------- |
| хранилище BLOB-объектов Azure          | `wasb[s]`       | `<container>@<storage_account>.blob.core.windows.net` |
| хранилище BLOB-объектов Azure          | `http[s]`       | `<storage_account>.blob.core.windows.net/<container>/subfolders` |
| Azure Data Lake Store Gen 1 | `http[s]`       | `<storage_account>.azuredatalakestore.net/webhdfs/v1` |
| Azure Data Lake Store Gen 2 | `http[s]`       | `<storage_account>.dfs.core.windows.net/<container>/subfolders`  |

Префикс `https:` позволяет использовать в пути вложенную папку.

#### <a name="credential"></a>Учетные данные
CREDENTIAL = `<database scoped credential>`. Это необязательные учетные данные, которые будут использоваться для проверки подлинности в службе хранилища Azure. Внешний источник данных без учетных данных может получить доступ к общедоступной учетной записи хранения. 

Внешние источники данных без учетных данных в выделенном пуле SQL будут использовать также удостоверение Azure AD вызывающей стороны для доступа к файлам в хранилище. Внешний источник данных для бессерверного пула SQL с учетными данными `IDENTITY='User Identity'` будет использовать для доступа к файлам удостоверение Azure AD вызывающей стороны.
- В выделенном пуле SQL в учетных данных базы данных можно указать настраиваемое удостоверение приложения, управляемое удостоверение рабочей области или ключ SAK. 
- В бессерверном пуле SQL учетные данные базы данных могут указывать удостоверение Azure AD вызывающей стороны, управляемое удостоверение рабочей области или ключ SAS. 

#### <a name="type"></a>TYPE
TYPE = `HADOOP`. Обязательный параметр в выделенном пуле SQL, указывающий, что для доступа к базовым файлам используется технология Polybase. Этот параметр не может использоваться в бессерверном пуле SQL, если в него встроен собственный модуль чтения.

### <a name="example-for-create-external-data-source"></a>Пример инструкции CREATE EXTERNAL DATA SOURCE

#### <a name="dedicated-sql-pool"></a>[Выделенный пул SQL](#tab/sql-pool)

Следующий пример создает внешний источник данных для Azure Data Lake 2-го поколения, который указывает на набор данных New York:

```sql
CREATE EXTERNAL DATA SOURCE AzureDataLakeStore
WITH
  -- Please note the abfss endpoint when your account has secure transfer enabled
  ( LOCATION = 'abfss://data@newyorktaxidataset.dfs.core.windows.net' ,
    CREDENTIAL = ADLS_credential ,
    TYPE = HADOOP
  ) ;
```

#### <a name="serverless-sql-pool"></a>[Бессерверный пул SQL](#tab/sql-on-demand)

В следующем примере создается внешний источник данных для Azure Data Lake 2-го поколения, доступ к которому можно получить с помощью учетных данных SAS.

```sql
CREATE DATABASE SCOPED CREDENTIAL [sqlondemand]
WITH IDENTITY='SHARED ACCESS SIGNATURE',  
SECRET = 'sv=2018-03-28&ss=bf&srt=sco&sp=rl&st=2019-10-14T12%3A10%3A25Z&se=2061-12-31T12%3A10%3A00Z&sig=KlSU2ullCscyTS0An0nozEpo4tO5JAgGBvw%2FJX2lguw%3D'
GO

CREATE EXTERNAL DATA SOURCE SqlOnDemandDemo WITH (
    LOCATION = 'https://sqlondemandstorage.blob.core.windows.net',
    CREDENTIAL = sqlondemand
);
```

В следующем примере создается внешний источник данных для Azure Data Lake 2-го поколения, который указывает на общедоступный набор данных New York.

```sql
CREATE EXTERNAL DATA SOURCE YellowTaxi
WITH ( LOCATION = 'https://azureopendatastorage.blob.core.windows.net/nyctlc/yellow/')
```
---

## <a name="create-external-file-format"></a>CREATE EXTERNAL FILE FORMAT

Создает объект формата внешнего файла, в котором определяются внешние данные, сохраненные в хранилище BLOB-объектов Azure или Azure Data Lake Storage. Создание формата внешнего файла — обязательное условие для создания внешней таблицы. Полная документация приводится [здесь](/sql/t-sql/statements/create-external-file-format-transact-sql?view=azure-sqldw-latest&preserve-view=true).

Создавая формат внешнего файла, вы указываете фактическую структуру данных, на которые ссылается внешняя таблица.

### <a name="syntax-for-create-external-file-format"></a>Синтаксис инструкции CREATE EXTERNAL FILE FORMAT

#### <a name="sql-pool"></a>[Пул SQL](#tab/sql-pool)

```syntaxsql
-- Create an external file format for PARQUET files.  
CREATE EXTERNAL FILE FORMAT file_format_name  
WITH (  
    FORMAT_TYPE = PARQUET  
    [ , DATA_COMPRESSION = {  
        'org.apache.hadoop.io.compress.SnappyCodec'  
      | 'org.apache.hadoop.io.compress.GzipCodec'      }  
    ]);  

--Create an external file format for DELIMITED TEXT files
CREATE EXTERNAL FILE FORMAT file_format_name  
WITH (  
    FORMAT_TYPE = DELIMITEDTEXT  
    [ , DATA_COMPRESSION = 'org.apache.hadoop.io.compress.GzipCodec' ]
    [ , FORMAT_OPTIONS ( <format_options> [ ,...n  ] ) ]  
    );  

<format_options> ::=  
{  
    FIELD_TERMINATOR = field_terminator  
    | STRING_DELIMITER = string_delimiter
    | First_Row = integer
    | USE_TYPE_DEFAULT = { TRUE | FALSE }
    | Encoding = {'UTF8' | 'UTF16'}
}
```

#### <a name="serverless-sql-pool"></a>[Бессерверный пул SQL](#tab/sql-on-demand)

```syntaxsql
-- Create an external file format for PARQUET files.  
CREATE EXTERNAL FILE FORMAT file_format_name  
WITH (  
    FORMAT_TYPE = PARQUET  
    [ , DATA_COMPRESSION = {  
        'org.apache.hadoop.io.compress.SnappyCodec'  
      | 'org.apache.hadoop.io.compress.GzipCodec'      }  
    ]);  

--Create an external file format for DELIMITED TEXT files
CREATE EXTERNAL FILE FORMAT file_format_name  
WITH (  
    FORMAT_TYPE = DELIMITEDTEXT  
    [ , DATA_COMPRESSION = 'org.apache.hadoop.io.compress.GzipCodec' ]
    [ , FORMAT_OPTIONS ( <format_options> [ ,...n  ] ) ]  
    );  

<format_options> ::=  
{  
    FIELD_TERMINATOR = field_terminator  
    | STRING_DELIMITER = string_delimiter
    | First_Row = integer
    | USE_TYPE_DEFAULT = { TRUE | FALSE }
    | Encoding = {'UTF8' | 'UTF16'}
    | PARSER_VERSION = {'parser_version'}
}
```

---


### <a name="arguments-for-create-external-file-format"></a>Аргументы инструкции CREATE EXTERNAL FILE FORMAT

file_format_name задает имя формата внешнего файла.

FORMAT_TYPE = [ PARQUET | DELIMITEDTEXT] задает формат внешних данных.

- PARQUET задает формат Parquet.
- DELIMITEDTEXT задает формат текста с разделителями столбцов (которые также называются признаками конца поля).

FIELD_TERMINATOR = *field_terminator* применяется только для текстовых файлов с разделителями. Признак конца поля задает один или несколько символов, отмечающих окончание каждого поля (столбца) в файле с разделителями текста. По умолчанию используется вертикальная черта ("|").

Примеры:

- FIELD_TERMINATOR = '|'
- FIELD_TERMINATOR = ' '
- FIELD_TERMINATOR = ꞌ\tꞌ

STRING_DELIMITER = *string_delimiter* задает признак конца поля для данных строкового типа в текстовом файле с разделителями. Разделитель строк имеет длину один или несколько символов и заключен в одинарные кавычки. По умолчанию используется пустая строка ("").

Примеры:

- STRING_DELIMITER = '"'
- STRING_DELIMITER = '*'
- STRING_DELIMITER = ꞌ,ꞌ

FIRST_ROW = *First_row_int* указывает номер строки, которая считывается первой во всех файлах. Если задано значение 2, при загрузке данных в каждом файле пропускается первая строка (строка заголовка). Строки пропускаются по признакам конца строк (/r/n, /r, /n).

USE_TYPE_DEFAULT = { TRUE | **FALSE** } указывает способ обработки отсутствующих значений в текстовых файлах с разделителями, когда извлекаются данные из текстового файла.

TRUE обозначает, что при извлечении данных из текстового файла каждое отсутствующее значение заменяется значением по умолчанию для типа данных, указанном в определении внешней таблицы для соответствующего столбца. Например, замените отсутствующее значение следующим:

- 0, если столбец определен как числовой. Столбцы decimal не поддерживаются, а их наличие вызовет ошибку.
- Пустая строка (""), если столбец является строковым.
- 1900-01-01, если столбец является столбцом дат.

FALSE означает, что все отсутствующие значения сохраняются как NULL. Любые значения NULL, сохраненные с использованием слова NULL в текстовом файле с разделителями, импортируются в качестве строки NULL.

Encoding = {'UTF8' | 'UTF16'} — бессерверный пул SQL умеет считывать текстовые файлы с разделителями в кодировке UTF8 или UTF16.

DATA_COMPRESSION = *data_compression_method* задает метод сжатия данных для внешних данных. 

Тип файлового формата PARQUET поддерживает следующие методы сжатия:

- DATA_COMPRESSION = 'org.apache.hadoop.io.compress.GzipCodec'
- DATA_COMPRESSION = 'org.apache.hadoop.io.compress.SnappyCodec'

Этот аргумент игнорируется при чтении из внешних таблиц PARQUET, но используется при записи во внешние таблицы с помощью [CETAS](develop-tables-cetas.md).

Тип формата файлов DELIMITEDTEXT поддерживает следующий метод сжатия:

- DATA_COMPRESSION = 'org.apache.hadoop.io.compress.GzipCodec'

PARSER_VERSION = 'parser_version' позволяет указать версию средства синтаксического анализа, которая используется при чтении файлов. Дополнительные сведения об аргументе PARSER_VERSION см. в разделе об [аргументах OPENROWSET](develop-openrowset.md#arguments).

### <a name="example-for-create-external-file-format"></a>Пример инструкции CREATE EXTERNAL FILE FORMAT

В следующем примере создается формат внешнего файла для данных о переписи населения:

```sql
CREATE EXTERNAL FILE FORMAT census_file_format
WITH
(  
    FORMAT_TYPE = PARQUET,
    DATA_COMPRESSION = 'org.apache.hadoop.io.compress.SnappyCodec'
)
```

## <a name="create-external-table"></a>CREATE EXTERNAL TABLE

Инструкция CREATE EXTERNAL TABLE создает в Synapse SQL внешнюю таблицу для доступа к данным в хранилище BLOB-объектов Azure или Azure Data Lake Storage. 

### <a name="syntax-for-create-external-table"></a>Синтаксис инструкции CREATE EXTERNAL TABLE

```sql
CREATE EXTERNAL TABLE { database_name.schema_name.table_name | schema_name.table_name | table_name }
    ( <column_definition> [ ,...n ] )  
    WITH (
        LOCATION = 'folder_or_filepath',  
        DATA_SOURCE = external_data_source_name,  
        FILE_FORMAT = external_file_format_name
    )  
[;]  

<column_definition> ::=
column_name <data_type>
    [ COLLATE collation_name ]
```

### <a name="arguments-create-external-table"></a>Аргументы инструкции CREATE EXTERNAL TABLE

*{ database_name.schema_name.table_name | schema_name.table_name | table_name }*

Имя создаваемой таблицы, состоящее из одной, двух или трех частей. При использовании внешней таблицы бессерверный пул SQL хранит только метаданные таблицы. Сами данные не перемещаются в бессерверный пул SQL и не хранятся там.

<column_definition>, ...*n* ]

CREATE EXTERNAL TABLE поддерживает возможность настроить имя столбца, тип данных и параметры сортировки. Параметр DEFAULT CONSTRAINT нельзя использовать с внешними таблицами.

>[!IMPORTANT]
>Определения столбцов, включая типы данных и количество столбцов, должны соответствовать данным во внешних файлах. В случае несоответствия при запросе данных строки файла будут отклонены.

При чтении из файлов Parquet можно указать только нужные столбцы, чтобы пропустить все остальные.

LOCATION = *путь_к_папке_или_файлу*

Указывает путь к папке или файлу и имя файла для фактических данных в хранилище больших двоичных объектов Azure. Расположение начинается с корневой папки. Корневая папка — это расположение данных, указанное во внешнем источнике данных.

Если указан параметр LOCATION для папки, бессерверный пул SQL будет выбирать данные из внешней таблицы и получать файлы из указанной папки.

> [!NOTE]
> В отличие от Hadoop и PolyBase, бессерверный пул SQL не возвращает вложенные папки. Она возвращает файлы, имена которых начинаются с символов подчеркивания (_) или точки (.).

В нашем примере, если указано LOCATION='/webdata/', запрос бессерверного пула SQL вернет строки из файлов mydata.txt и _hidden.txt. Файлы mydata2.txt и mydata3.txt не возвращаются, так как они находятся во вложенной папке.

![Рекурсивные данные для внешних таблиц](./media/develop-tables-external-tables/folder-traversal.png)

DATA_SOURCE = *имя_внешнего_источника_данных*. Задает имя внешнего источника данных, который содержит расположение внешних данных. Для создания внешнего источника данных используйте инструкцию [CREATE EXTERNAL DATA SOURCE](#create-external-data-source).

FILE_FORMAT = *имя_формата_внешнего_файла*. Задает имя объекта формата внешнего файла, который хранит тип файла и метод сжатия внешних данных. Чтобы создать формат внешнего файла, используйте [CREATE EXTERNAL FILE FORMAT](#create-external-file-format).

### <a name="permissions-create-external-table"></a>Разрешения для CREATE EXTERNAL TABLE

Для выбора данных из внешней таблицы нужны учетные данные с разрешениями на чтение и получение списка.

### <a name="example-create-external-table"></a>Пример инструкции CREATE EXTERNAL TABLE

В следующем примере создается внешняя таблица. Далее возвращается ее первая строка.

```sql
CREATE EXTERNAL TABLE census_external_table
(
    decennialTime varchar(20),
    stateName varchar(100),
    countyName varchar(100),
    population int,
    race varchar(50),
    sex    varchar(10),
    minAge int,
    maxAge int
)  
WITH (
    LOCATION = '/parquet/',
    DATA_SOURCE = population_ds,  
    FILE_FORMAT = census_file_format
)
GO

SELECT TOP 1 * FROM census_external_table
```

## <a name="create-and-query-external-tables-from-a-file-in-azure-data-lake"></a>Создание внешних таблиц из файла в Azure Data Lake и запросы по ним

Используя возможности просмотра Data Lake, вы можете создавать внешнюю таблицу и выполнять запросы к ней через выделенный или бессерверный пул SQL, просто щелкнув файл правой кнопкой мыши. Создание внешних таблиц одним щелчком в учетной записи хранения ADLS 2-го поколения поддерживается только для файлов Parquet. 

### <a name="prerequisites"></a>Предварительные требования

- В системе доступа ARM у вас должен быть доступ к рабочей области по меньшей мере с ролью участника данных BLOB-объектов хранилища для учетной записи ADLS 2-го поколения.

- Вам нужны по меньшей мере [права на создание](/sql/t-sql/statements/create-external-table-transact-sql?view=azure-sqldw-latest#permissions-2&preserve-view=true) внешних таблиц и на выполнение запросов в пуле SQL или SQL по запросу.

На панели "Данные" выберите файл, из которого нужно создать внешнюю таблицу.
> [!div class="mx-imgBorder"]
>![externaltable1](./media/develop-tables-external-tables/external-table-1.png)

Откроется диалоговое окно. Выберите выделенный или бессерверный пул SQL, присвойте таблице имя и щелкните "Открыть скрипт"

> [!div class="mx-imgBorder"]
>![externaltable2](./media/develop-tables-external-tables/external-table-2.png)

Автоматически создается скрипт SQL на основании схемы, определенной по файлу.
> [!div class="mx-imgBorder"]
>![externaltable3](./media/develop-tables-external-tables/external-table-3.png)

Выполните скрипт. Этот скрипт автоматически выполняет инструкцию SELECT Top 100 *.
> [!div class="mx-imgBorder"]
>![externaltable4](./media/develop-tables-external-tables/external-table-4.png)

Теперь создается внешняя таблица для последующего изучения ее содержимого, для которого пользователь может выполнять запросы непосредственно на панели данных.
> [!div class="mx-imgBorder"]
>![externaltable5](./media/develop-tables-external-tables/external-table-5.png)

## <a name="next-steps"></a>Дальнейшие действия

В статье [про CETAS](develop-tables-cetas.md) описано, как сохранить результаты запроса во внешней таблице в службе хранилища Azure. Вы также можете начать работать с запросами ко [внешним таблицам Apache Spark для Azure Synapse](develop-storage-files-spark-tables.md).
