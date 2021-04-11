---
title: Использование OPENROWSET в бессерверном пуле SQL
description: В этой статье описаны синтаксис OPENROWSET в бессерверном пуле SQL и использование аргументов.
services: synapse-analytics
author: filippopovic
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 05/07/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.openlocfilehash: a6af8f4600544007a9ecb4fcff5ac9016fd7dfbf
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105640347"
---
# <a name="how-to-use-openrowset-using-serverless-sql-pool-in-azure-synapse-analytics"></a>Использование OPENROWSET с бессерверным пулом SQL в Azure Synapse Analytics

Функция `OPENROWSET(BULK...)` позволяет получить доступ к файлам в службе хранилища Azure. Функция `OPENROWSET` считывает содержимое удаленного источника данных (например, файла) и возвращает содержимое в виде набора строк. В ресурсе бессерверного пула SQL доступ к поставщику больших наборов строк OPENROWSET осуществляется путем вызова функции OPENROWSET и указанием параметра BULK.  

Из предложения `FROM` запроса можно ссылаться на функцию `OPENROWSET`, как если бы это было имя таблицы `OPENROWSET`. Функция также поддерживает массовые операции с помощью встроенного поставщика BULK, позволяющего считывать данные из файла и возвращать их в виде набора строк.

## <a name="data-source"></a>Источник данных

Функция OPENROWSET в Synapse SQL считывает содержимое файлов из источника данных. Источник данных — это учетная запись хранения Azure, которую можно явно указать в функции `OPENROWSET` или динамически выводить из URL-адреса файлов, которые вы намерены считать.
Функция `OPENROWSET` может содержать необязательный параметр `DATA_SOURCE` для указания источника данных с нужными файлами.
- `OPENROWSET` без `DATA_SOURCE` позволяет считать содержимого файлов напрямую по URL-адресу, указанному в параметре `BULK`:

    ```sql
    SELECT *
    FROM OPENROWSET(BULK 'http://<storage account>.dfs.core.windows.net/container/folder/*.parquet',
                    FORMAT = 'PARQUET') AS file
    ```

Это быстрый и простой способ считать содержимое файлов без предварительной настройки. Этот параметр позволяет использовать базовую проверку подлинности для доступа к хранилищу (сквозная проверка Azure AD для имен входа Azure AD или маркер SAS для имен входа SQL). 

- `OPENROWSET` с `DATA_SOURCE` можно использовать для доступа к файлам в указанной учетной записи хранения:

    ```sql
    SELECT *
    FROM OPENROWSET(BULK '/folder/*.parquet',
                    DATA_SOURCE='storage', --> Root URL is in LOCATION of DATA SOURCE
                    FORMAT = 'PARQUET') AS file
    ```


    Этот параметр позволяет настроить расположение учетной записи хранения в источнике данных и указать метод проверки подлинности, который должен использоваться для доступа к хранилищу. 
    
    > [!IMPORTANT]
    > `OPENROWSET` без `DATA_SOURCE` обеспечивает быстрый и простой способ доступа к файлам хранилища, но поддерживает мало возможностей для проверки подлинности. Например, субъекты Azure AD могут обращаться к файлам только с использованием [удостоверения Azure AD](develop-storage-files-storage-access-control.md?tabs=user-identity) или общедоступным файлам. Если вам нужны более мощные возможности для проверки подлинности, используйте параметр `DATA_SOURCE` и определите учетные данные, которые нужно использовать для доступа к хранилищу.


## <a name="security"></a>Безопасность

Пользователь базы данных должен иметь разрешение `ADMINISTER BULK OPERATIONS`, чтобы использовать функцию `OPENROWSET`.

Администратор хранилища также должен разрешить пользователю доступ к файлам, предоставив действительный маркер SAS или разрешение субъекту Azure AD для доступа к файлам хранилища. Узнайте больше об управлении доступом к хранилищу в [этой статье](develop-storage-files-storage-access-control.md).

`OPENROWSET` использует следующие правила, чтобы выбрать способ проверки подлинности в хранилище:
- В `OPENROWSET` без `DATA_SOURCE` механизм проверки подлинности зависит от типа вызывающего объекта.
  - Любой пользователь может использовать `OPENROWSET` без `DATA_SOURCE`, чтобы считывать общедоступные файлы в службе хранилища Azure.
  - Имена входа Azure AD могут получать доступ к защищенным файлам с помощью собственных [удостоверений Azure AD](develop-storage-files-storage-access-control.md?tabs=user-identity#supported-storage-authorization-types), если служба хранилища Azure позволяет пользователю Azure AD получать доступ к базовым файлам (например, если у вызывающего объекта есть разрешение `Storage Reader` в службе хранилища Azure).
  - Для имен входа SQL также можно использовать `OPENROWSET` без `DATA_SOURCE` для доступа к общедоступным файлам, защищенным с помощью маркера SAS файлам и управляемым удостоверениям рабочей области Synapse. Чтобы разрешить доступ к файлам хранилища, необходимо [создать учетные данные уровня сервера](develop-storage-files-storage-access-control.md#examples). 
- В `OPENROWSET` с `DATA_SOURCE` механизм проверки подлинности определяется в учетных данных уровня базы данных, которые назначены указанному источнику данных. Этот параметр позволяет обращаться к общедоступному хранилищу, а также к защищенному хранилищу по маркеру SAS, управляемому удостоверению рабочей области или [удостоверению вызывающего объекта Azure AD](develop-storage-files-storage-access-control.md?tabs=user-identity#supported-storage-authorization-types) (если этот вызывающий объект является субъектом Azure AD). Если `DATA_SOURCE` ссылается на хранилище Azure, которое не является общедоступным, придется [создать учетные данные уровня базы данных](develop-storage-files-storage-access-control.md#examples) и указать их в `DATA SOURCE`, чтобы разрешить доступ к файлам хранилища.

Вызывающий объект должен иметь разрешение `REFERENCES` для этих учетных данных, чтобы использовать их для проверки подлинности в хранилище.

## <a name="syntax"></a>Синтаксис

```syntaxsql
--OPENROWSET syntax for reading Parquet files
OPENROWSET  
( { BULK 'unstructured_data_path' , [DATA_SOURCE = <data source name>, ]
    FORMAT='PARQUET' }  
)  
[WITH ( {'column_name' 'column_type' }) ]
[AS] table_alias(column_alias,...n)

--OPENROWSET syntax for reading delimited text files
OPENROWSET  
( { BULK 'unstructured_data_path' , [DATA_SOURCE = <data source name>, ] 
    FORMAT = 'CSV'
    [ <bulk_options> ] }  
)  
WITH ( {'column_name' 'column_type' [ 'column_ordinal' | 'json_path'] })  
[AS] table_alias(column_alias,...n)
 
<bulk_options> ::=  
[ , FIELDTERMINATOR = 'char' ]    
[ , ROWTERMINATOR = 'char' ] 
[ , ESCAPE_CHAR = 'char' ] 
[ , FIRSTROW = 'first_row' ]     
[ , FIELDQUOTE = 'quote_characters' ]
[ , DATA_COMPRESSION = 'data_compression_method' ]
[ , PARSER_VERSION = 'parser_version' ]
[ , HEADER_ROW = { TRUE | FALSE } ]
[ , DATAFILETYPE = { 'char' | 'widechar' } ]
[ , CODEPAGE = { 'ACP' | 'OEM' | 'RAW' | 'code_page' } ]
```

## <a name="arguments"></a>Аргументы

Есть два варианта ввода входных файлов, содержащих целевые данные для запросов. Допустимые значения:

- "CSV" — включает любой текстовый файл с разделителями (разделители строк и столбцов). В качестве разделителя полей можно использовать любой символ, например TSV: FIELDTERMINATOR = знак табуляции.

- "PARQUET" — двоичный файл в формате Parquet. 

**unstructured_data_path**

Аргумент unstructured_data_path, который определяет путь к данным, может содержать абсолютный или относительный путь:
- Абсолютный путь в формате "\<prefix>://\<storage_account_path>/\<storage_path>" позволяет пользователю напрямую читать файлы.
- Относительный путь в формате "<путь_к_хранилищу>", который нужно использовать только с параметром `DATA_SOURCE`, описывает шаблон имени файла в расположении <путь_к_учетной_записи_хранения>, которое определено в `EXTERNAL DATA SOURCE`. 

Ниже приведены соответствующие значения <storage account path>, которые будут связаны с конкретным внешним источником данных. 

| Внешний источник данных       | Prefix | Путь к учетной записи хранения                                 |
| -------------------------- | ------ | ---------------------------------------------------- |
| хранилище BLOB-объектов Azure         | http(s)  | \<storage_account>.blob.core.windows.net/path/file   |
| хранилище BLOB-объектов Azure         | wasb(s)  | \<container>@\<storage_account>.blob.core.windows.net/path/file |
| Azure Data Lake Store 1-го поколения | http(s)  | \<storage_account>.azuredatalakestore.net/webhdfs/v1 |
| Azure Data Lake Store 2-го поколения | http(s)  | \<storage_account>.dfs.core.windows.net /path/file   |
| Azure Data Lake Store 2-го поколения | abfs(s)  | [\<file_system>@\<account_name>.dfs.core.windows.net/path/file](../../storage/blobs/data-lake-storage-introduction-abfs-uri.md#uri-syntax)              |
||||

'\<storage_path>'

Позволяет задать путь в хранилище, указывающий на папку или файл, который нужно считать. Если путь указывает на контейнер или папку, все файлы будут считываться только из этого контейнера или папки. Файлы во вложенных папках не включаются. 

Для выбора нескольких файлов или папок можно использовать подстановочные знаки. Допускается использование нескольких подстановочных знаков, если они размещены не подряд.
Ниже приведен пример для считывания всех файлов *.csv*, в начале имени которых указано *population*, из всех папок, в начале имени которых указано */csv/population*:  
`https://sqlondemandstorage.blob.core.windows.net/csv/population*/population*.csv`

Если указать, что unstructured_data_path является папкой, тогда запрос бессерверного пула SQL будет извлекать файлы из этой папки. 

Вы можете настроить бессерверный пул SQL, так чтобы он перемещался по папкам, указав /* в конце пути, как показано в примере: `https://sqlondemandstorage.blob.core.windows.net/csv/population/**`

> [!NOTE]
> В отличие от Hadoop и PolyBase бессерверный пул SQL не возвращает вложенные папки, если в конце пути не указано /**. Еще одно отличие от Hadoop и PolyBase заключается в том, что бессерверный пул SQL не возвращает файлы, имя которых начинается с подчеркивания (_) или точки (.).

Если в приведенном ниже примере unstructured_data_path = `https://mystorageaccount.dfs.core.windows.net/webdata/`, после обращения в бессерверный пул SQL будут возвращены строки из mydata.txt и _hidden.txt. Файлы mydata2.txt и mydata3.txt не возвращаются, так как они находятся во вложенной папке.

![Рекурсивные данные для внешних таблиц](./media/develop-openrowset/folder-traversal.png)

`[WITH ( {'column_name' 'column_type' [ 'column_ordinal'] }) ]`

Предложение WITH позволяет указать столбцы, которые нужно считать из файлов.

- Для чтения всех столбцов в файлах данных CSV укажите имена столбцов и их типы данных. Если нужно считать подмножество столбцов, используйте порядковые номера, чтобы выбрать столбцы из исходных файлов данных по порядковому номеру. Столбцы будут привязаны к порядковым обозначениям. Если использовать значение HEADER_ROW = TRUE, привязка к столбцу будет выполнена по его имени, а не по порядковому номеру.
    > [!TIP]
    > Вы также можете опустить предложение WITH для CSV-файлов. Типы данных будут автоматически выводиться из содержимого файла. Аргумент HEADER_ROW можно использовать для указания существования строки заголовка. Тогда имена столбцов регистра будут считываться из этой строки заголовка. Узнайте больше об [автоматическом обнаружении схемы](#automatic-schema-discovery).
    
- Для файлов данных Parquet укажите имена столбцов, совпадающих с именами столбцов в исходных файлах данных. Столбцы будут привязаны по имени и учитывать регистр. Если предложение WITH не указано, возвращаются все столбцы из файлов Parquet.
    > [!IMPORTANT]
    > Имена столбцов в файлах Parquet учитывают регистр. Если указать имя столбца с регистром, отличным от имени столбца в файле Parquet, тогда для этого столбца будут возвращены значения NULL.


column_name (имя_столбца) = имя выходного столбца. Если значение указано, это имя переопределяет имя столбца в исходном файле и имя столбца, указанное в пути JSON (если применимо). Если аргумент json_path не указан, он будет автоматически добавлен как $.column_name. Проверьте поведение аргумента json_path.

column_type (тип_столбца) = тип данных в выходном столбце. Выполняется неявное преобразование типов данных.

column_ordinal (порядковый_номер_столбца) = порядковый номер столбца в исходных файлах. Этот аргумент не учитывается для файлов Parquet, так как привязка выполняется по имени. В следующем примере из CSV-файла будет возвращен только второй столбец:

```sql
WITH (
    --[country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
    [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2 2
    --[year] smallint,
    --[population] bigint
)
```

json_path = [выражение пути JSON](/sql/relational-databases/json/json-path-expressions-sql-server?view=azure-sqldw-latest&preserve-view=true) к столбцу или вложенному свойству. [Режим пути](/sql/relational-databases/json/json-path-expressions-sql-server?view=azure-sqldw-latest&preserve-view=true#PATHMODE) по умолчанию является нестрогим.

> [!NOTE]
> В строгом режиме выполнение запроса завершится сбоем, если указанный путь не существует. В нестрогом режиме запрос будет выполнен, и выражение пути JSON получит значение NULL.

**\<bulk_options>**

FIELDTERMINATOR ='field_terminator' (признак_конца_поля)

Указывает признак конца поля, который нужно использовать. Признак конца поля по умолчанию — запятая (" **,** ").

ROWTERMINATOR ='row_terminator' (признак_конца_строки)

Указывает признак конца строки, который нужно использовать. Если признак конца строки не указан, будет использоваться один из признаков конца строки по умолчанию. По умолчанию для версии PARSER_VERSION = "1.0" используются признаки \r\n, \n и \r. По умолчанию для версии PARSER_VERSION = "2.0" используются признаки \r\n и \n.

ESCAPE_CHAR = 'char'

Задает символ в файле, используемый для экранирования собственно этого символа и всех значений разделителей в файле. Если за escape-символом следует значение, отличное от него самого или какого-либо из значений разделителей, при считывании этого значения escape-символ пропускается. 

Параметр ESCAPE_CHAR будет применяться независимо от того, включен ли параметр FIELDQUOTE. Он не будет использоваться для экранирования символа кавычек. Символ кавычек нужно экранировать другим символом кавычек. Такой символ может использоваться в значении столбца только в том случае, если значение инкапсулировано с помощью символов кавычек.

FIRSTROW = 'first_row' (первая_строка) 

Указывает номер первой строки для загрузки. Значение по умолчанию — 1. Оно указывает на первую строку в используемом файле данных. Номера строк определяются с помощью подсчета признаков конца строки. Значения аргумента FIRSTROW начинаются с 1.

FIELDQUOTE = 'field_quote' (символ_кавычек) 

Определяет символ, который будет использоваться в качестве символа кавычки в CSV-файле. Если значение не указано, будут использоваться текущий символ кавычек ("). 

DATA_COMPRESSION = "метод_cжатия_данных"

Позволяет указать метод сжатия. Поддерживается только в PARSER_VERSION = "1.0". Поддерживается следующий метод сжатия:

- GZIP

PARSER_VERSION = "версия_средства_синтаксического_анализа"

Позволяет указать версию средства синтаксического анализа, которая используется при чтении файлов. В настоящее время поддерживаются версии 1.0 и 2.0 средства синтаксического анализа для CSV-файлов.

- PARSER_VERSION = "1.0"
- PARSER_VERSION = "2.0"

Средство синтаксического анализа для CSV-файлов версии 1.0 обладает широким набором функций и используется по умолчанию, а версия 2.0 оптимизирована для производительности и поддерживает не все параметры и кодировки. 

Особенности средства синтаксического анализа CSV версии 1.0:

- Приведенные ниже параметры не поддерживаются. HEADER_ROW.

Особенности средства синтаксического анализа для CSV-файлов версии 2.0:

- Поддерживаются не все типы данных.
- Максимальная длина символьного столбца — 8000.
- Размер строки не может превышать 8 МБ.
- Приведенные ниже параметры не поддерживаются. DATA_COMPRESSION.
- Пустая строка в кавычках ("") интерпретируется как пустая строка.
- Поддерживаемый формат для типа данных DATE: ГГГГ-ММ-ДД
- Поддерживаемый формат для типа данных TIME: ЧЧ:ММ:СС[.доли секунды].
- Поддерживаемый формат для типа данных DATETIME2: ГГГГ-ММ-ДД ЧЧ:ММ:СС[.доли секунды]

HEADER_ROW = { TRUE | FALSE }

Указывает, содержит ли CSV-файл строку заголовка. Значение по умолчанию — FALSE. Поддерживается в PARSER_VERSION = "2.0". Если значение равно TRUE, то имена столбцов будут считываться из первой строки в соответствии с аргументом FIRSTROW. Если значение равно TRUE и схема указана с помощью предложения WITH, привязка имен столбцов будет выполняться по именам столбцов, а не по их порядковым номерам.

DATAFILETYPE = { 'char' | 'widechar' }

Указывает на кодировку: char используется для файлов UTF8, а widechar — для UTF16.

CODEPAGE = { 'ACP' | 'OEM' | 'RAW' | 'code_page' }

Указывает кодовую страницу данных в файле данных. Значение по умолчанию — 65001 (кодировка UTF-8). Дополнительные сведения об этом параметре доступны [здесь](/sql/t-sql/functions/openrowset-transact-sql?view=sql-server-ver15&preserve-view=true#codepage).

## <a name="fast-delimited-text-parsing"></a>Быстрый синтаксический анализ текста с разделителями

Можно использовать две версии средства синтаксического анализа текста с разделителями. Средство синтаксического анализа CSV-файла версии 1.0 является стандартным и изначально имеет больше возможностей, а средство синтаксического анализа версии 2.0 создано для повышения производительности. Улучшение производительности в средстве синтаксического анализа версии 2.0 создается за счет расширенных техник синтаксического анализа и многопоточности. С увеличением размера файла также будет увеличиваться разница в скорости.

## <a name="automatic-schema-discovery"></a>Автоматическое обнаружение схем

Файлы CSV и Parquet можно запрашивать, не зная и не указывая схему, опустив предложение WITH. Имена столбцов и типы данных будут выводиться из файлов.

Файлы Parquet содержат метаданные столбцов, которые считываются. Сопоставления типов можно найти в разделе [Сопоставление типов для Parquet](#type-mapping-for-parquet). Проверьте [считывание файлов Parquet без указания схем](#read-parquet-files-without-specifying-schema) для выборки.

Имена столбцов для CSV-файлов можно считывать из строки заголовка. Можно указать, существует ли строка заголовка, с помощью аргумента HEADER_ROW. Если HEADER_ROW = FALSE, тогда будут использоваться общие имена столбцов: C1, C2, ... Cn, где n — число столбцов в файле. Типы данных будут выводиться из первых 100 строк данных. Проверьте [считывание CSV-файлов без указания схемы](#read-csv-files-without-specifying-schema) для примеров.

> [!IMPORTANT]
> В некоторых случаях из-за недостатка информации не удается вычислить подходящий тип данных, поэтому вместо него используется больший тип данных. Это приводит к снижению производительности, что важно для столбцов со знаками, которые будут выводиться как varchar(8000). Чтобы обеспечить оптимальную производительность, проверьте [выводимые типы данных](best-practices-sql-on-demand.md#check-inferred-data-types) и [используйте соответствующие типы данных](best-practices-sql-on-demand.md#use-appropriate-data-types).

### <a name="type-mapping-for-parquet"></a>Сопоставление типов для Parquet

Файлы Parquet содержат описания типов для каждого столбца. В приведенной ниже таблице показано, как типы Parquet сопоставляются с собственными типами SQL.

| Тип Parquet | Логический тип Parquet (заметка) | Тип данных SQL |
| --- | --- | --- |
| BOOLEAN | | bit |
| BINARY / BYTE_ARRAY | | varbinary |
| DOUBLE | | FLOAT |
| FLOAT | | real |
| INT32 | | INT |
| INT64 | | BIGINT |
| INT96 | |datetime2 |
| FIXED_LEN_BYTE_ARRAY | |binary |
| BINARY |UTF8 |varchar \*(параметры сортировки UTF8) |
| BINARY |STRING |varchar \*(параметры сортировки UTF8) |
| BINARY |ENUM|varchar \*(параметры сортировки UTF8) |
| FIXED_LEN_BYTE_ARRAY |UUID |UNIQUEIDENTIFIER |
| BINARY |DECIMAL |Decimal |
| BINARY |JSON |varchar(8000) \*(параметры сортировки UTF8) |
| BINARY |BSON | Не поддерживается |
| FIXED_LEN_BYTE_ARRAY |DECIMAL |Decimal |
| BYTE_ARRAY |INTERVAL | Не поддерживается |
| INT32 |INT(8, true) |smallint |
| INT32 |INT(16, true) |smallint |
| INT32 |INT(32, true) |INT |
| INT32 |INT(8, false) |tinyint |
| INT32 |INT(16, false) |INT |
| INT32 |INT(32, false) |BIGINT |
| INT32 |DATE |Дата |
| INT32 |DECIMAL |Decimal |
| INT32 |TIME (MILLIS)|time |
| INT64 |INT(64, true) |BIGINT |
| INT64 |INT(64, false) |decimal(20,0) |
| INT64 |DECIMAL |Decimal |
| INT64 |TIME (MICROS) |time — TIME(NANOS) не поддерживается |
|INT64 |TIMESTAMP (MILLIS / MICROS) |datetime2 — TIMESTAMP(NANOS) не поддерживается |
|[Сложный тип](https://github.com/apache/parquet-format/blob/master/LogicalTypes.md#lists) |Список |varchar(8000), сериализованный в JSON |
|[Сложный тип](https://github.com/apache/parquet-format/blob/master/LogicalTypes.md#maps)|MAP|varchar(8000), сериализованный в JSON |

## <a name="examples"></a>Примеры

### <a name="read-csv-files-without-specifying-schema"></a>Считывание CSV-файлов без указания схемы

В следующем примере считывается CSV-файл, содержащий строку заголовка, без указания имен столбцов и типов данных: 

```sql
SELECT 
    *
FROM OPENROWSET(
    BULK 'https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases/latest/ecdc_cases.csv',
    FORMAT = 'CSV',
    PARSER_VERSION = '2.0',
    HEADER_ROW = TRUE) as [r]
```

В следующем примере считывается CSV-файл, который не содержит строку заголовка, без указания имен столбцов и типов данных: 

```sql
SELECT 
    *
FROM OPENROWSET(
    BULK 'https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases/latest/ecdc_cases.csv',
    FORMAT = 'CSV',
    PARSER_VERSION = '2.0') as [r]
```

### <a name="read-parquet-files-without-specifying-schema"></a>Считывание файлов Parquet без указания схемы

В следующем примере возвращаются все столбцы первой строки из набора данных переписи в формате Parquet без указания имен столбцов и типов данных. 

```sql
SELECT 
    TOP 1 *
FROM  
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer/release/us_population_county/year=20*/*.parquet',
        FORMAT='PARQUET'
    ) AS [r]
```

### <a name="read-specific-columns-from-csv-file"></a>Считывание конкретных столбцов из CSV-файла

В следующем примере из файлов population*.csv возвращаются только два столбца с порядковыми номерами 1 и 4. В файлах нет строки заголовка, поэтому чтение начинается с первой строки.

```sql
SELECT 
    * 
FROM OPENROWSET(
        BULK 'https://sqlondemandstorage.blob.core.windows.net/csv/population/population*.csv',
        FORMAT = 'CSV',
        FIRSTROW = 1
    )
WITH (
    [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2 1,
    [population] bigint 4
) AS [r]
```

### <a name="read-specific-columns-from-parquet-file"></a>Считывание конкретных столбцов из файла Parquet

В следующем примере возвращаются только два столбца первой строки из набора данных переписи в формате Parquet: 

```sql
SELECT 
    TOP 1 *
FROM  
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer/release/us_population_county/year=20*/*.parquet',
        FORMAT='PARQUET'
    )
WITH (
    [stateName] VARCHAR (50),
    [population] bigint
) AS [r]
```

### <a name="specify-columns-using-json-paths"></a>Определение столбцов с помощью путей JSON

В следующем примере показано, как можно использовать выражения пути [JSON](/sql/relational-databases/json/json-path-expressions-sql-server?view=azure-sqldw-latest&preserve-view=true) в предложении WITH и в чем заключается различие между строгими и нестрогими режимами пути: 

```sql
SELECT 
    TOP 1 *
FROM  
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer/release/us_population_county/year=20*/*.parquet',
        FORMAT='PARQUET'
    )
WITH (
    --lax path mode samples
    [stateName] VARCHAR (50), -- this one works as column name casing is valid - it targets the same column as the next one
    [stateName_explicit_path] VARCHAR (50) '$.stateName', -- this one works as column name casing is valid
    [COUNTYNAME] VARCHAR (50), -- STATEname column will contain NULLs only because of wrong casing - it targets the same column as the next one
    [countyName_explicit_path] VARCHAR (50) '$.COUNTYNAME', -- STATEname column will contain NULLS only because of wrong casing and default path mode being lax

    --strict path mode samples
    [population] bigint 'strict $.population' -- this one works as column name casing is valid
    --,[population2] bigint 'strict $.POPULATION' -- this one fails because of wrong casing and strict path mode
)
AS [r]
```

## <a name="next-steps"></a>Дальнейшие действия

См. примеры в [кратком руководстве по работе с запросами к хранилищу данных](query-data-storage.md), где показано, как использовать `OPENROWSET` для чтения файлов в формате [CSV](query-single-csv-file.md), [PARQUET](query-parquet-files.md) и [JSON](query-json-files.md). Ознакомьтесь с [рекомендациями](best-practices-sql-on-demand.md), чтобы иметь возможность достичь оптимальной производительности. Вы также узнаете, как сохранить результаты запроса в службе хранилища Azure с помощью [CETAS](develop-tables-cetas.md).