---
title: Создание задания потоковой передачи T-SQL в Azure SQL ребро
description: Узнайте, как создавать задания Stream Analytics в Azure SQL.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 07/27/2020
ms.openlocfilehash: 97189fd7a232c2467981b23dc20da51ebef08252
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97656348"
---
# <a name="create-a-data-streaming-job-in-azure-sql-edge"></a>Создание задания потоковой передачи данных в Azure SQL ребро 

В этой статье объясняется, как создать задание потоковой передачи T-SQL в Azure SQL. Вы создаете входные и выходные объекты внешнего потока, а затем определяете запрос задания потоковой передачи в ходе создания задания потоковой передачи.

## <a name="configure-the-external-stream-input-and-output-objects"></a>Настройка входных и выходных объектов внешнего потока

Потоковая передача T-SQL использует функциональность внешнего источника данных SQL Server для определения источников данных, связанных с входными и выходными данными внешнего потока для задания потоковой передачи. Используйте следующие команды T-SQL для создания внешнего потока входного или выходного объекта.

- [CREATE EXTERNAL FILE FORMAT (Transact-SQL)](/sql/t-sql/statements/create-external-file-format-transact-sql)

- [CREATE EXTERNAL DATA SOURCE (Transact-SQL)](/sql/t-sql/statements/create-external-data-source-transact-sql)

- [CREATE EXTERNAL STREAM (Transact-SQL)](#example-create-an-external-stream-object-to-azure-sql-database)

Кроме того, если Azure SQL ребро, SQL Server или база данных SQL Azure используется в качестве выходного потока, необходимо [создать учетные данные для базы данных с областью действия (Transact-SQL)](/sql/t-sql/statements/create-database-scoped-credential-transact-sql). Эта команда T-SQL определяет учетные данные для доступа к базе данных.

### <a name="supported-input-and-output-stream-data-sources"></a>Поддерживаемые источники данных входного и выходного потока

В настоящее время SQL Azure для пограничных вычислений поддерживает только следующие источники данных в качестве входных потоков и потоков вывода.

| Тип источника данных | Входные данные | Выходные данные | Описание |
|------------------|-------|--------|------------------|
| Центр Azure IoT Edge | Да | Да | Источник данных для чтения и записи потоковых данных в центр Azure IoT Edge. Дополнительные сведения см. в разделе [центр IOT Edge](../iot-edge/iot-edge-runtime.md#iot-edge-hub).|
| База данных SQL | Нет | Да | Соединение с источником данных для записи потоковых данных в Базу данных SQL. База данных может быть локальной базой данных в Azure SQL или удаленной базой данных в SQL Server или базе данных SQL Azure.|
| Kafka | Да | Нет | Источник данных для чтения потоковых данных из раздела Kafka. Этот адаптер сейчас доступен только для версий Intel или AMD Azure SQL. Она недоступна для ARM64 версии Azure SQL.|

### <a name="example-create-an-external-stream-inputoutput-object-for-azure-iot-edge-hub"></a>Пример. Создание внешнего потока входного или выходного объекта для центра Azure IoT Edge

В следующем примере создается объект внешнего потока для Azure IoT Edge концентратора. Чтобы создать источник входных или выходных данных внешнего потока для центра Azure IoT Edge, сначала необходимо создать формат внешнего файла для макета данных, которые считываются или записываются.

1. Создайте формат внешнего файла для типа JSON.

    ```sql
    Create External file format InputFileFormat
    WITH 
    (  
       format_type = JSON,
    )
    go
    ```

2. Создайте внешний источник данных для центра Azure IoT Edge. Следующий скрипт T-SQL создает подключение к источнику данных к концентратору IoT Edge, который работает на том же узле DOCKER, что и Azure SQL.

    ```sql
    CREATE EXTERNAL DATA SOURCE EdgeHubInput 
    WITH 
    (
        LOCATION = 'edgehub://'
    )
    go
    ```

3. Создайте объект внешнего потока для концентратора Azure IoT Edge. Следующий скрипт T-SQL создает объект потока для концентратора IoT Edge. В случае с объектом потока концентратора IoT Edge параметр LOCATION — это имя раздела центра IoT Edge или канала, который считывается или записывается в.

    ```sql
    CREATE EXTERNAL STREAM MyTempSensors 
    WITH 
    (
        DATA_SOURCE = EdgeHubInput,
        FILE_FORMAT = InputFileFormat,
        LOCATION = N'TemperatureSensors',
        INPUT_OPTIONS = N'',
        OUTPUT_OPTIONS = N''
    );
    go
    ```

### <a name="example-create-an-external-stream-object-to-azure-sql-database"></a>Пример. Создание объекта внешнего потока для базы данных SQL Azure

В следующем примере создается объект внешнего потока для локальной базы данных в Azure SQL. 

1. Создайте главный ключ в базе данных. Это необходимо для шифрования секрета учетных данных.

    ```sql
    CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<<Strong_Password_For_Master_Key_Encryption>>';
    ```

2. Создайте учетные данные уровня базы данных для доступа к источнику SQL Server. В следующем примере создаются учетные данные для внешнего источника данных с ИДЕНТИФИКАТОРом = username, а SECRET — Password.

    ```sql
    CREATE DATABASE SCOPED CREDENTIAL SQLCredential
    WITH IDENTITY = '<SQL_Login>', SECRET = '<SQL_Login_PASSWORD>'
    go
    ```

3. Создайте внешний источник данных с помощью инструкции CREATE EXTERNAL DATA SOURCE. Следующий пример:

    * Создает внешний источник данных с именем *локалсклаутпут*.
    * Определяет внешний источник данных (LOCATION = <vendor>://<server>[:<port>]). В этом примере он указывает на локальный экземпляр Azure SQL ребр.
    * Использует учетные данные, созданные ранее.

    ```sql
    CREATE EXTERNAL DATA SOURCE LocalSQLOutput 
    WITH 
    (
        LOCATION = 'sqlserver://tcp:.,1433',
        CREDENTIAL = SQLCredential
    )
    go
    ```

4. Создайте объект внешнего потока. В следующем примере создается объект внешнего потока, указывающий на таблицу *dbo. Температуремеасурементс* в *мисклдатабасе* базы данных.

    ```sql
    CREATE EXTERNAL STREAM TemperatureMeasurements 
    WITH 
    (
        DATA_SOURCE = LocalSQLOutput,
        LOCATION = N'MySQLDatabase.dbo.TemperatureMeasurements',
        INPUT_OPTIONS = N'',
        OUTPUT_OPTIONS = N''
    );
    ```

### <a name="example-create-an-external-stream-object-for-kafka"></a>Пример. Создание объекта внешнего потока для Kafka

В следующем примере создается объект внешнего потока для локальной базы данных в Azure SQL. В этом примере предполагается, что сервер Kafka настроен для анонимного доступа. 

1. Создайте внешний источник данных с помощью инструкции CREATE EXTERNAL DATA SOURCE. Следующий пример:

    ```sql
    Create EXTERNAL DATA SOURCE [KafkaInput] 
    With
    (
        LOCATION = N'kafka://<kafka_bootstrap_server_name_ip>:<port_number>'
    )
    GO
    ```
2. Создайте формат внешнего файла для входных данных Kafka. В следующем примере создается формат JSON-файла с со сжатием Gzip сжатием. 

   ```sql
   CREATE EXTERNAL FILE FORMAT JsonGzipped  
    WITH 
    (  
        FORMAT_TYPE = JSON , 
        DATA_COMPRESSION = 'org.apache.hadoop.io.compress.GzipCodec' 
    )
   ```

3. Создайте объект внешнего потока. В следующем примере создается объект внешнего потока, указывающий на раздел Kafka `*TemperatureMeasurement*` .

    ```sql
    CREATE EXTERNAL STREAM TemperatureMeasurement 
    WITH 
    (  
        DATA_SOURCE = KafkaInput, 
        FILE_FORMAT = JsonGzipped,
        LOCATION = 'TemperatureMeasurement',
        INPUT_OPTIONS = 'PARTITIONS: 10' 
    ); 
    ```

## <a name="create-the-streaming-job-and-the-streaming-queries"></a>Создание задания потоковой передачи и потоковых запросов

Используйте `sys.sp_create_streaming_job` системную хранимую процедуру для определения потоковых запросов и создания задания потоковой передачи. `sp_create_streaming_job`Хранимая процедура принимает следующие параметры:

- `job_name`: Имя задания потоковой передачи. Имена заданий потоковой передачи уникальны в пределах экземпляра.
- `statement`: Stream Analytics инструкции запросов потоковой передачи на основе [языка запросов](/stream-analytics-query/stream-analytics-query-language-reference).

В следующем примере создается простое задание потоковой передачи с одним запросом потоковой передачи. Этот запрос считывает входные данные из центра IoT Edge и выполняет запись `dbo.TemperatureMeasurements` в базу данных.

```sql
EXEC sys.sp_create_streaming_job @name=N'StreamingJob1',
@statement= N'Select * INTO TemperatureMeasurements from MyEdgeHubInput'
```

В следующем примере создается более сложная потоковая работа с несколькими разными запросами. Эти запросы включают в себя одну, которая использует встроенную `AnomalyDetection_ChangePoint` функцию для обнаружения аномалий в данных температуры.

```sql
EXEC sys.sp_create_streaming_job @name=N'StreamingJob2', @statement=
N' Select * INTO TemperatureMeasurements1 from MyEdgeHubInput1

Select * Into TemperatureMeasurements2 from MyEdgeHubInput2

Select * Into TemperatureMeasurements3 from MyEdgeHubInput3

SELECT
Timestamp as [Time],
[Temperature] As [Temperature],
GetRecordPropertyValue(AnomalyDetection_ChangePoint(Temperature, 80, 1200) OVER(LIMIT DURATION(minute, 20)), ''Score'') as ChangePointScore,
GetRecordPropertyValue(AnomalyDetection_ChangePoint(Temperature, 80, 1200) OVER(LIMIT DURATION(minute, 20)), ''IsAnomaly'') as IsChangePointAnomaly
INTO TemperatureAnomalies FROM MyEdgeHubInput2;
'
go
```

## <a name="start-stop-drop-and-monitor-streaming-jobs"></a>Запуск, завершение, удаление и отслеживание заданий потоковой передачи

Чтобы запустить задание потоковой передачи в Azure SQL ребро, выполните `sys.sp_start_streaming_job` хранимую процедуру. Хранимая процедура требует, чтобы имя запуска задания потоковой передачи было запущено в качестве входных данных.

```sql
exec sys.sp_start_streaming_job @name=N'StreamingJob1'
go
```

Чтобы прерывать задание потоковой передачи, выполните `sys.sp_stop_streaming_job` хранимую процедуру. Хранимая процедура требует, чтобы имя задания потоковой передачи было приостановлено в качестве входных данных.

```sql
exec sys.sp_stop_streaming_job @name=N'StreamingJob1'
go
```

Чтобы удалить (или удалить) задание потоковой передачи, выполните `sys.sp_drop_streaming_job` хранимую процедуру. Хранимая процедура требует, чтобы имя задания потоковой передачи было удалено в качестве входных данных.

```sql
exec sys.sp_drop_streaming_job @name=N'StreamingJob1'
go
```

Чтобы получить текущее состояние задания потоковой передачи, выполните `sys.sp_get_streaming_job` хранимую процедуру. Хранимая процедура требует, чтобы имя задания потоковой передачи было удалено в качестве входных данных. Он выводит имя и текущее состояние задания потоковой передачи.

```sql
exec sys.sp_get_streaming_job @name=N'StreamingJob1'
        WITH RESULT SETS
(
       (
       name nvarchar(256),
       status nvarchar(256),
       error nvarchar(256)
       )
)
```

Задание потоковой передачи может иметь одно из следующих состояний:

| Состояние | Описание |
|--------| ------------|
| Создание | Задание потоковой передачи было создано, но еще не запущено. |
| Запуск | Задание потоковой передачи запускается. |
| Бездействие | Задание потоковой передачи выполняется, но входные данные для обработки отсутствуют. |
| Обработка | Задание потоковой передачи выполняется, и входные данные обрабатываются. Это свидетельствует о работоспособности задания потоковой передачи. |
| Деградация | Задание потоковой передачи выполняется, но при обработке входных данных возникли неустранимые ошибки. Входное задание продолжит выполняться, но входные данные, вызывающие ошибки, будут удаляться. |
| Остановлена | Выполнение задания потоковой передачи остановлено. |
| Сбой | Сбой задания потоковой передачи. Обычно это указывает на неустранимую ошибку при обработке. |

## <a name="next-steps"></a>Дальнейшие действия

- [Просмотр метаданных, связанных с заданиями потоковой передачи в Azure SQL ребро](streaming-catalog-views.md) 
- [Создание внешнего потока](create-external-stream-transact-sql.md)