---
title: Отправка запросов к хранилищу данных с помощью бессерверного пула SQL
description: В этой статье описывается, как выполнять запросы к службе хранилища Azure с помощью ресурса бессерверного пула SQL в Azure Synapse Analytics.
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/15/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: d299afca0bd8070a1da738e02812b64c41a7101c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101675042"
---
# <a name="query-storage-files-with-serverless-sql-pool-in-azure-synapse-analytics"></a>Отправка запросов к файлам хранилища с помощью бессерверного пула SQL в Azure Synapse Analytics

Бессерверный пул SQL позволяет запрашивать данные в озере данных. Предоставляемая контактная зона для запросов T-SQL позволяет запрашивать частично структурированные и неструктурированные данные. Возможность запрашивания данных поддерживает следующие аспекты T-SQL:

- Полная контактная зона инструкции [SELECT](/sql/t-sql/queries/select-transact-sql?view=azure-sqldw-latest&preserve-view=true), включая большинство [функций и операторов SQL](overview-features.md).
- CREATE EXTERNAL TABLE AS SELECT ([CETAS](develop-tables-cetas.md)) создает [внешнюю таблицу](develop-tables-external-tables.md), а затем в параллельном режиме экспортирует результаты инструкции SELECT Transact-SQL в службу хранилища Azure.

Дополнительные сведения о поддерживаемых и неподдерживаемых возможностях см. в статье с [общими сведениями о бессерверном пуле SQL](on-demand-workspace-overview.md) или следующих статьях:
- [Создание возможностей доступа к хранилищу](develop-storage-files-overview.md). Из этой статьи вы узнаете, как использовать [внешние таблицы](develop-tables-external-tables.md) и функцию [OPENROWSET](develop-openrowset.md) для считывания данных из хранилища.
- [Управление доступом к хранилищу](develop-storage-files-storage-access-control.md). Из этой статьи вы узнаете, как обеспечить доступ Synapse SQL к хранилищу с использованием аутентификации SAS или управляемого удостоверения рабочей области.

## <a name="overview"></a>Обзор

Чтобы оптимизировать процесс запрашивания данных, размещенных в файлах службы хранилища Azure, бессерверный пул SQL использует функцию [OPENROWSET](develop-openrowset.md) со следующими дополнительными возможностями:

- [Запрашивание нескольких файлов или папок](#query-multiple-files-or-folders)
- [Формат PARQUET](#query-parquet-files)
- [Запрашивание CSV-файлов и текстовых файлов с разделителями (признак конца поля, признак конца строки, escape-символ)](#query-csv-files)
- [Считывание выбранного подмножества столбцов](#read-a-chosen-subset-of-columns)
- [Вывод схемы](#schema-inference)
- [Функция filename](#filename-function)
- [Функция filepath](#filepath-function)
- [Работа со сложными типами, а также вложенными и повторяющимися структурами данных](#work-with-complex-types-and-nested-or-repeated-data-structures)

## <a name="query-parquet-files"></a>Запрашивание файлов PARQUET

Для запрашивания исходных данных в формате PARQUET используйте параметр FORMAT = 'PARQUET'.

```syntaxsql
SELECT * FROM
OPENROWSET( BULK N'https://myaccount.dfs.core.windows.net//mycontainer/mysubfolder/data.parquet', FORMAT = 'PARQUET') 
WITH (C1 int, C2 varchar(20), C3 varchar(max)) as rows
```

См. [примеры запросов к файлам в формате PARQUET](query-parquet-files.md).

## <a name="query-csv-files"></a>Запросы к CSV-файлам

Чтобы запросить исходные данные в формате CSV, используйте параметр FORMAT = 'CSV'. Вы можете указать схему CSV-файла в функции `OPENROWSET` при запрашивании CSV-файлов:

```sql
SELECT * FROM
OPENROWSET( BULK N'https://myaccount.dfs.core.windows.net/mycontainer/mysubfolder/data.csv', FORMAT = 'CSV', PARSER_VERSION='2.0') 
WITH (C1 int, C2 varchar(20), C3 varchar(max)) as rows
```

Для настройки правил синтаксического анализа пользовательских файлов CSV можно использовать некоторые дополнительные параметры:
- ESCAPE_CHAR = 'char'. Определяет символ в файле, который используется для экранирования самого себя и всех значений разделителей. Если за escape-символом следует значение, отличное от него самого или какого-либо из значений разделителей, при считывании этого значения escape-символ пропускается.
Параметр ESCAPE_CHAR будет применяться независимо от того, включен ли параметр FIELDQUOTE. Он не будет использоваться для экранирования символа кавычек. Символ кавычек нужно экранировать другим символом кавычек. Такой символ может использоваться в значении столбца только в том случае, если значение инкапсулировано с помощью символов кавычек.
- FIELDTERMINATOR ='field_terminator'. Определяет используемый признак конца поля. По умолчанию признаком конца поля считается запятая ( **,** ).
- ROWTERMINATOR ='row_terminator'. Определяет используемый признак конца строки. По умолчанию признаком конца строки считается символ новой строки ( **\r\n**).

## <a name="file-schema"></a>Схема файла

Язык SQL в Synapse SQL позволяет определить схему файла как часть функции `OPENROWSET` и считывать все столбцы или их подмножество. Либо служба попытается автоматически определить типы столбцов из файла с помощью вывода схемы.

### <a name="read-a-chosen-subset-of-columns"></a>Считывание выбранного подмножества столбцов

Чтобы указать столбцы для считывания, включите необязательное предложение WITH в инструкцию `OPENROWSET`.

- Если используются CSV-файлы данных, для считывания всех столбцов укажите имена и типы данных столбцов. Если нужно считать подмножество столбцов, используйте порядковые номера, чтобы выбрать столбцы из исходных файлов данных по порядковому номеру. Столбцы будут привязаны к порядковым обозначениям.
- Если используются файлы данных PARQUET, укажите имена столбцов, совпадающие с именами столбцов в исходных файлах данных. Столбцы будут привязаны к именам.

```sql
SELECT * FROM
OPENROWSET( BULK N'https://myaccount.dfs.core.windows.net/mycontainer/mysubfolder/data.parquet', FORMAT = 'PARQUET') 
WITH (
      C1 int, 
      C2 varchar(20),
      C3 varchar(max)
) as rows
```

Для каждого столбца необходимо указать его имя и тип в предложении `WITH`.
См. примеры в разделе [Возвращение подмножества столбцов](query-single-csv-file.md#return-a-subset-of-columns).

## <a name="schema-inference"></a>Вывод схемы

Если вы не указываете предложение WITH в инструкции `OPENROWSET`, служба попытается автоматически определить (вывести) схему из базовых файлов.

> [!NOTE]
> Сейчас это поддерживается только для формата PARQUET.

```sql
SELECT * FROM
OPENROWSET( BULK N'https://myaccount.dfs.core.windows.net/mycontainer/mysubfolder/data.parquet', FORMAT = 'PARQUET') 
```

Убедитесь, что для оптимальной производительности используются [соответствующие выводимые типы данных](best-practices-sql-on-demand.md#check-inferred-data-types). 

## <a name="query-multiple-files-or-folders"></a>Запрашивание нескольких файлов или папок

Чтобы выполнить запрос T-SQL набора файлов в папке или набора папок в качестве одной сущности или набора строк, укажите путь к папке или шаблон пути (с подстановочными знаками) к набору файлов или папок.

Применяются следующие правила.

- Шаблоны можно использовать как часть пути к папке или в имени файла.
- В одной части пути к каталогу или имени файла могут использоваться несколько шаблонов.
- Если указано несколько подстановочных знаков, в результирующий набор файлов будут включены все файлы, связанные со всеми соответствующими путями.

```sql
SELECT * FROM
OPENROWSET( BULK N'https://myaccount.dfs.core.windows.net/myroot/*/mysubfolder/*.parquet', FORMAT = 'PARQUET' ) as rows
```

См. [примеры запросов к нескольким файлам и папкам](query-folders-multiple-csv-files.md).

## <a name="file-metadata-functions"></a>Функции метаданных файлов

### <a name="filename-function"></a>Функция filename

Эта функция возвращает имя файла, из которого получена строка. 

Для запрашивания определенных файлов используйте инструкции из раздела [Filename](query-specific-files.md#filename).

Тип возвращаемых данных — nvarchar(1024). Для оптимальной производительности всегда приводите результат функции filename к соответствующему типу данных. Если используется символьный тип данных, убедитесь, что используется соответствующая длина.

### <a name="filepath-function"></a>Функция filepath

Эта функция возвращает полный путь или часть пути:

- При вызове без параметра она возвращает полный путь, из которого была получена строка.
- При вызове с параметром она возвращает ту часть пути, которая соответствует подстановочному знаку в позиции, определенной в этом параметре. Например, при значении параметра 1 возвращается часть пути, соответствующая первому подстановочному знаку.

См. сведения в разделе [Filepath](query-specific-files.md#filepath).

Тип возвращаемых данных — nvarchar(1024). Для оптимальной производительности всегда приводите результат функции filepath к соответствующему типу данных. Если используется символьный тип данных, убедитесь, что используется соответствующая длина.

## <a name="work-with-complex-types-and-nested-or-repeated-data-structures"></a>Работа со сложными типами, а также вложенными и повторяющимися структурами данных

Чтобы оптимизировать работу с данными, которые хранятся во вложенных или повторяющихся типах данных (например, в файлах [Parquet](https://github.com/apache/parquet-format/blob/master/LogicalTypes.md#nested-types)), в бессерверный пул SQL добавлены указанные ниже расширения.

#### <a name="project-nested-or-repeated-data"></a>Вложенные или повторяющиеся данные в проекте

Для проецирования данных выполните инструкцию SELECT для файлов PARQUET, которые содержат столбцы вложенных типов данных. При выводе вложенные значения сериализуются в формат JSON и возвращаются с типом данных SQL varchar(8000).

```sql
    SELECT * FROM
    OPENROWSET
    (   BULK 'unstructured_data_path' ,
        FORMAT = 'PARQUET' )
    [AS alias]
```

См. сведения о запрашивании вложенных типов данных в разделе [Вложенные или повторяющиеся данные в проекте](query-parquet-nested-types.md#project-nested-or-repeated-data).

#### <a name="access-elements-from-nested-columns"></a>Доступ к элементам из вложенных столбцов

Чтобы получить доступ к вложенным элементам из вложенных столбцов, таким как Struct, используйте точечную нотацию для конкатенации имен полей в строку пути. Передайте полученный путь в параметре column_name предложения WITH функции `OPENROWSET`.

Ниже приводится пример фрагмента с таким синтаксисом.

```syntaxsql
    OPENROWSET
    (   BULK 'unstructured_data_path' ,
        FORMAT = 'PARQUET' )
    WITH ({'column_name' 'column_type',})
    [AS alias]
    'column_name' ::= '[field_name.] field_name'
```

По умолчанию функция `OPENROWSET` сопоставляет имя исходного поля и путь к нему с именами столбцов в предложении WITH. С помощью предложения WITH можно получить доступ к элементам на разных уровнях вложенности в пределах одного исходного файла PARQUET.

**Возвращаемые значения**

- Функция возвращает скалярное значение, например int, decimal и varchar, из указанного элемента и по указанному пути для всех типов Parquet, которые не находятся в группе вложенных типов.
- Если путь указывает на элемент вложенного типа, функция возвращает фрагмент JSON, который начинается с верхнего элемента указанного пути. Этот фрагмент JSON имеет тип varchar(8000).
- Если свойство не удается найти по указанному значению column_name, функция возвращает ошибку.
- Если свойство не удается найти по указанному значению column_path, в зависимости от настроенного [режима PATH](/sql/relational-databases/json/json-path-expressions-sql-server?view=azure-sqldw-latest&preserve-view=true#PATHMODE), функция возвращает ошибку (строгий режим) или значение NULL (нестрогий режим).

См. примеры таких запросов в разделе [Доступ к элементам из вложенных столбцов](query-parquet-nested-types.md#read-properties-from-nested-object-columns).

#### <a name="access-elements-from-repeated-columns"></a>Доступ к элементам из повторяющихся столбцов

Чтобы получить доступ к элементам из повторяющегося столбца, например Array или Map, используйте функцию [JSON_VALUE](/sql/t-sql/functions/json-value-transact-sql?view=azure-sqldw-latest&preserve-view=true) для каждого проецируемого скалярного элемента, предоставив ей следующие данные:

- Вложенный или повторяющийся столбец в качестве первого параметра.
- [Путь JSON](/sql/relational-databases/json/json-path-expressions-sql-server?view=azure-sqldw-latest&preserve-view=true), который обозначает нужный элемент свойства, в качестве второго параметра.

Чтобы обратиться к элементам с нескалярным значением из повторяющегося столбца, используйте функцию [JSON_QUERY](/sql/t-sql/functions/json-query-transact-sql?view=azure-sqldw-latest&preserve-view=true) для каждого проецируемого нескалярного элемента, предоставив ей следующие данные:

- Вложенный или повторяющийся столбец в качестве первого параметра.
- [Путь JSON](/sql/relational-databases/json/json-path-expressions-sql-server?view=azure-sqldw-latest&preserve-view=true), который обозначает нужный элемент свойства, в качестве второго параметра.

Фрагмент такого синтаксиса приводится ниже.

```syntaxsql
    SELECT
       { JSON_VALUE (column_name, path_to_sub_element), }
       { JSON_QUERY (column_name [ , path_to_sub_element ]), )
    FROM
    OPENROWSET
    (   BULK 'unstructured_data_path' ,
        FORMAT = 'PARQUET' )
    [AS alias]
```

См. [примеры запросов для получения доступа к элементам из повторяющихся столбцов](query-parquet-nested-types.md#access-elements-from-repeated-columns).

## <a name="query-samples"></a>Примеры запросов

Чтобы узнать больше о том, как запрашивать данные различных типов, воспользуйтесь примерами запросов.

### <a name="tools"></a>Инструменты

Ниже указаны средства, необходимые для создания запросов.
    - Azure Synapse Studio 
    - Azure Data Studio
    - SQL Server Management Studio

### <a name="demo-setup"></a>Настройка демонстрации

Для начала **создайте базу данных**, в которой будут выполняться запросы. Затем инициализируйте объекты, выполнив [скрипт настройки](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) для этой базы данных. 

Этот скрипт создает источники данных, учетные данные области базы данных и форматы внешних файлов, которые используются для чтения данных в этих примерах.

> [!NOTE]
> Базы данных используются только для просмотра метаданных, а не для реальных данных.  Запишите имя используемой базы данных, ведь оно потребуется позже.

```sql
CREATE DATABASE mydbname;
```

### <a name="provided-demo-data"></a>Предоставление демонстрационных данных

Демонстрационные данные содержат указанные ниже наборы данных.

- NYC Taxi - Yellow Taxi Trip Records (Такси Нью-Йорка. Записи поездок такси желтого цвета) — часть общедоступного набора данных по Нью-Йорку в формате CSV и Parquet
- Набор данных по населению в формате CSV
- Примеры файлов Parquet со вложенными столбцами:
- Книги в формате JSON

| Путь к папке                                                  | Описание                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| /csv/                                                        | Родительская папка для данных в формате CSV.                         |
| /csv/population/<br />/csv/population-unix/<br />/csv/population-unix-hdr/<br />/csv/population-unix-hdr-escape<br />/csv/population-unix-hdr-quoted | Папки с файлами данных о населении в разных форматах CSV. |
| /csv/taxi/                                                   | Папка с общедоступными файлами данных по Нью-Йорку в формате CSV.              |
| /parquet/                                                    | Родительская папка для данных в формате Parquet.                     |
| /parquet/taxi                                                | Общедоступные файлы данных по Нью-Йорку в формате Parquet, секционированные по годам и месяцам с использованием схемы секционирования Hive на платформе Hadoop. |
| /parquet/nested/                                             | Примеры файлов Parquet со вложенными столбцами:                     |
| /json/                                                       | Родительская папка для данных в формате JSON.                        |
| /json/books/                                                 | Файлы JSON с данными книг.                                   |


## <a name="next-steps"></a>Дальнейшие действия

Сведения о запрашивании файлов разных типов, о создании и использовании представлений см. в следующих статьях:

- [Запрашивание CSV-файлов](query-single-csv-file.md)
- [Запрашивание файлов Parquet](query-parquet-files.md)
- [Запрашивание JSON-файлов](query-json-files.md)
- [Запрашивание вложенных значений](query-parquet-nested-types.md)
- [Запрашивание папок и нескольких CSV-файлов](query-folders-multiple-csv-files.md)
- [Использование метаданных файла в запросах](query-specific-files.md)
- [Создание и использование представлений](create-use-views.md)
