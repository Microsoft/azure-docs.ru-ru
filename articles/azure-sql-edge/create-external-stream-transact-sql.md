---
title: Создание внешнего потока (Transact-SQL) — Azure SQL ребро
description: Сведения о инструкции CREATE EXTERNAL STREAM в Azure SQL ребро
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 07/27/2020
ms.openlocfilehash: 203abe2b6def478dc1747dd4ce638b5b62707612
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101659228"
---
# <a name="create-external-stream-transact-sql"></a>CREATE EXTERNAL STREAM (Transact-SQL)

Объект внешнего потока имеет двойное назначение как входного, так и выходного потока. Его можно использовать в качестве входных данных для отправки запросов в потоковые данные из служб приема событий, таких как концентратор событий Azure, центр Интернета вещей Azure (или концентратор ребра) или Kafka, либо можно использовать в качестве выходных данных для указания места и способа хранения результатов из потокового запроса.

Вы также можете указать и создать EXTERNAL STREAM в качестве выходных и входных данных для таких служб, как концентратор событий или хранилище BLOB-объектов. Это способствует созданию цепочек сценариев, в которых потоковый запрос сохраняет результаты во внешний поток в качестве выходных данных, а другой потоковый запрос считывает из того же внешнего потока, что и входные данные.

В настоящее время SQL Azure для пограничных вычислений поддерживает только следующие источники данных в качестве входных потоков и потоков вывода.

| Тип источника данных | Входные данные | Выходные данные | Описание |
|------------------|-------|--------|------------------|
| Центр Azure IoT Edge | Да | Да | Источник данных для чтения и записи потоковых данных в центр Azure IoT Edge. Дополнительные сведения см. в разделе [центр IOT Edge](../iot-edge/iot-edge-runtime.md#iot-edge-hub).|
| База данных SQL | Нет | Да | Соединение с источником данных для записи потоковых данных в Базу данных SQL. База данных может быть локальной базой данных в Azure SQL или удаленной базой данных в SQL Server или базе данных SQL Azure.|
| Kafka | Да | Нет | Источник данных для чтения потоковых данных из раздела Kafka. Поддержка Kafka недоступна для ARM64 версии Azure SQL.|



## <a name="syntax"></a>Синтаксис

```sql
CREATE EXTERNAL STREAM {external_stream_name}  
( <column_definition> [, <column_definition> ] * ) -- Used for Inputs - optional 
WITH  ( <with_options> )

<column_definition> ::=
  column_name <column_data_type>

<data_type> ::=
[ type_schema_name . ] type_name
    [ ( precision [ , scale ] | max ) ]

<with_options> ::=
  DATA_SOURCE = data_source_name, 
  LOCATION = location_name, 
  [FILE_FORMAT = external_file_format_name], --Used for Inputs - optional 
  [<optional_input_options>],
  [<optional_output_options>], 
  TAGS = <tag_column_value>

<optional_input_options> ::= 
  INPUT_OPTIONS = '[<Input_options_data>]'

<Input_option_data> ::= 
      <input_option_values> [ , <input_option_values> ]

<input_option_values> ::=
  PARTITIONS: [number_of_partitions]
  | CONSUMER_GROUP: [ consumer_group_name ] 
  | TIME_POLICY: [ time_policy ] 
  | LATE_EVENT_TOLERANCE: [ late_event_tolerance_value ] 
  | OUT_OF_ORDER_EVENT_TOLERANCE: [ out_of_order_tolerance_value ]
  
<optional_output_options> ::= 
  OUTPUT_OPTIONS = '[<output_option_data>]'

<output_option_data> ::= 
      <output_option_values> [ , <output_option_values> ]

<output_option_values> ::=
   REJECT_POLICY: [ reject_policy ] 
   | MINIMUM_ROWS: [ row_value ] 
   | MAXIMUM_TIME: [ time_value_minutes] 
   | PARTITION_KEY_COLUMN: [ partition_key_column_name ] 
   | PROPERTY_COLUMNS: [ ( [ output_col_name ] ) ] 
   | SYSTEM_PROPERTY_COLUMNS: [ ( [ output_col_name ] ) ] 
   | PARTITION_KEY: [ partition_key_name ] 
   | ROW_KEY: [ row_key_name ] 
   | BATCH_SIZE: [ batch_size_value ] 
   | MAXIMUM_BATCH_COUNT: [ batch_value ] 
   | STAGING_AREA: [ blob_data_source ]
 
<tag_column_value> ::= -- Reserved for Future Usage
); 
```

## <a name="arguments"></a>Аргументы

- [DATA_SOURCE](/sql/t-sql/statements/create-external-data-source-transact-sql/)
- [FILE_FORMAT](/sql/t-sql/statements/create-external-file-format-transact-sql/)
- **LOCATION**. Задает имя для фактических данных или расположения в источнике данных. 
   - Для объектов потока ребра или Kafka, Location указывает имя концентратора ребра или раздела Kafka для чтения или записи.
   - Для объектов потока SQL (SQL Server, база данных SQL Azure или Azure SQL ребро) указывает имя таблицы. Если поток создается в той же базе данных и схеме, что и Целевая таблица, то достаточно лишь имя таблицы. В противном случае необходимо полностью определить (<имя_базы_данных. schema_name. table_name) имя таблицы.
   - Для расположения объекта потока хранилища BLOB-объектов Azure ссылается на шаблон пути, используемый внутри контейнера больших двоичных объектов. Дополнительные сведения об этой функции см. в статье (/артиклес/стреам-Аналитикс/стреам-Аналитикс-дефине-аутпутс.МД # Blob-Storage-and-Azure-Data-Lake-Gen2).

- **INPUT_OPTIONS**: укажите параметры в виде пар "ключ-значение" для таких служб, как Kafka, IOT Edge концентратора, которые являются входными для потоковых запросов.
    - PARTITIONs: количество секций, определенных для раздела. Максимальное число секций, которое можно использовать, ограничено 32.
      - Применяется к входным потокам Kafka
    - CONSUMER_GROUP. Концентратор событий и центры Интернета вещей ограничивают число читателей в одной группе потребителей (до пяти). Если оставить это поле пустым, будет использоваться группа потребителей $Default.
      - Зарезервировано для использования в будущем. Не применяется к Azure SQL Server.  
    - TIME_POLICY. Описывает, следует ли удалять события или настраивать время продолжительности события, если поздние или неупорядоченные события передают свое значение допуска.
      - Зарезервировано для использования в будущем. Не применяется к Azure SQL Server.
    - LATE_EVENT_TOLERANCE. Максимально допустимая задержка времени. Задержка представляет разницу между меткой времени события и системными часами.
      - Зарезервировано для использования в будущем. Не применяется к Azure SQL Server.
    - OUT_OF_ORDER_EVENT_TOLERANCE. После того как события перешли из входных данных в запрос потоковой передачи, они могут поступать не по порядку. Эти события можно принять как есть или приостановить процесс на определенное время, чтобы изменить их порядок.
      - Зарезервировано для использования в будущем. Не применяется к Azure SQL Server.
        
- **OUTPUT_OPTIONS**. Указывает параметры в качестве пар "ключ — значение" для поддерживаемых служб, которые являются выходными данными для запросов потоковой передачи. 
  - REJECT_POLICY.  DROP | RETRY —указывает политики обработки ошибок данных при возникновении ошибок преобразования данных. 
    - Применяется ко всем поддерживаемым выходным данным. 
  - MINIMUM_ROWS.  
    Минимальное количество строк, необходимых для каждого пакета, записанного в выходные данные. Для Parquet каждый пакет создаст файл. 
    - Применяется ко всем поддерживаемым выходным данным. 
  - MAXIMUM_TIME.  
    Максимальное время ожидания в минутах на пакет. По истечении этого времени пакет будет записан в выходные данные, даже если не будет выполнено требование к минимальному числу строк. 
    - Применяется ко всем поддерживаемым выходным данным. 
  - PARTITION_KEY_COLUMN.  
    Столбец, который используется для ключа секции. 
    - Зарезервировано для использования в будущем. Не применяется к Azure SQL Server.
  - PROPERTY_COLUMNS.  
    Разделенный запятыми список имен выходных столбцов, которые, если их предоставить, будут вложены в сообщения в виде настраиваемых свойств.  
    - Зарезервировано для использования в будущем. Не применяется к Azure SQL Server. 
  - SYSTEM_PROPERTY_COLUMNS.  
    Коллекция пар "имя — значение" для имен системных свойств и выходных столбцов, которая будет заполнена на основе сообщений служебной шины, в формате JSON. Например: { "MessageId": "column1", "PartitionKey": "column2"}. 
    - Зарезервировано для использования в будущем. Не применяется к Azure SQL Server. 
  - PARTITION_KEY.  
    Имя выходного столбца, содержащего ключ раздела. Ключ раздела — это уникальный идентификатор раздела в пределах конкретной таблицы, являющийся первой частью первичного ключа сущности. Это строковое значение размером до 1 КБ. 
    - Зарезервировано для использования в будущем. Не применяется к Azure SQL Server.
  - ROW_KEY.  
    Имя выходного столбца, содержащего ключ строки. Ключ строки — это уникальный идентификатор сущности внутри конкретного раздела. Он является второй частью первичного ключа сущности. Ключ строки — это строковое значение размером до 1 КБ. 
    - Зарезервировано для использования в будущем. Не применяется к Azure SQL Server.
  - BATCH_SIZE.  
    Представляет количество транзакций для хранилища таблиц, в которых максимальное число строк может доходить до 100. Для Функции Azure представляет размер пакета в байтах, отправляемого в функцию за один вызов. Значение по умолчанию — 256 КБ. 
    - Зарезервировано для использования в будущем. Не применяется к Azure SQL Server. 
  - MAXIMUM_BATCH_COUNT.  
    Максимальное количество событий, отправленных в функцию, на вызов для Функции Azure. Значение по умолчанию — 100. Для Базы данных SQL представляет максимальное число записей, отправленных с каждой транзакцией массовой вставки. По умолчанию — 10 000. 
    - Применяется ко всем выходам на основе SQL 
  - STAGING_AREA: объект внешнего источника данных в хранилище BLOB-объектов промежуточная область для приема данных с высокой пропускной способностью в Azure синапсе Analytics 
    - Зарезервировано для использования в будущем. Не применяется к Azure SQL Server.


## <a name="examples"></a>Примеры

### <a name="example-1---edgehub"></a>Пример 1. EdgeHub

Тип: Вход или выход<br>

Синтаксис:

```sql
CREATE EXTERNAL DATA SOURCE MyEdgeHub 
WITH  
(      
  LOCATION = 'edgehub://'       
); 
 
CREATE EXTERNAL FILE FORMAT myFileFormat  
WITH (  
   FORMAT_TYPE = JSON, 
); 
 
CREATE EXTERNAL STREAM Stream_A  
WITH    
(   
   DATA_SOURCE = MyEdgeHub, 
   FILE_FORMAT = myFileFormat, 
   LOCATION = '<mytopicname>', 
   OUTPUT_OPTIONS =   
     'REJECT_TYPE: Drop'
);
```

### <a name="example-2---azure-sql-database-azure-sql-edge-sql-server"></a>Пример 2. База данных SQL Azure, SQL Azure для пограничных вычислений, SQL Server

Тип: Выходные данные<br>

Синтаксис:

```sql
CREATE DATABASE SCOPED CREDENTIAL SQLCredName 
WITH IDENTITY = '<user>', 
SECRET = '<password>'; 
 
-- Azure SQL Database 
CREATE EXTERNAL DATA SOURCE MyTargetSQLTabl 
WITH 
(     
  LOCATION = '<my_server_name>.database.windows.net', 
  CREDENTIAL = SQLCredName 
); 
 
--SQL Server or Azure SQL Edge
CREATE EXTERNAL DATA SOURCE MyTargetSQLTabl 
WITH 
(     
  LOCATION = ' <sqlserver://<ipaddress>,<port>', 
  CREDENTIAL = SQLCredName 
); 

CREATE EXTERNAL STREAM Stream_A 
WITH   
(  
    DATA_SOURCE = MyTargetSQLTable, 
    LOCATION = '<DatabaseName>.<SchemaName>.<TableName>' ,
   --Note: If table is contained in the database, <TableName> should be sufficient 
    OUTPUT_OPTIONS =  
      'REJECT_TYPE: Drop'
); 
```

### <a name="example-3---kafka"></a>Пример 3. Kafka

Тип: Входные данные<br>

Синтаксис:
```sql
CREATE EXTERNAL DATA SOURCE MyKafka_tweets 
WITH 
( 
  --The location maps to KafkaBootstrapServer 
  LOCATION = 'kafka://<kafkaserver>:<ipaddress>', 
  CREDENTIAL = kafkaCredName 
); 
 
CREATE EXTERNAL FILE FORMAT myFileFormat 
WITH ( 
    FORMAT_TYPE = JSON, 
    DATA_COMPRESSION = 'org.apache.hadoop.io.compress.GzipCodec'
); 

CREATE EXTERNAL STREAM Stream_A (user_id VARCHAR, tweet VARCHAR) 
WITH   
(  
    DATA_SOURCE = MyKafka_tweets, 
    LOCATION = '<KafkaTopicName>', 
    FILE_FORMAT = myFileFormat,  
    INPUT_OPTIONS =  
      'PARTITIONS: 5'
); 
```

## <a name="see-also"></a>См. также раздел

- [DROP EXTERNAL STREAM (Transact-SQL)](drop-external-stream-transact-sql.md)
