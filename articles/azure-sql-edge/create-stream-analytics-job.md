---
title: Создание задания потоковой передачи T-SQL в SQL Azure для пограничных вычислений
description: Подробнее о создании заданий Stream Analytics в SQL Azure для пограничных вычислений.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 07/27/2020
ms.openlocfilehash: 97189fd7a232c2467981b23dc20da51ebef08252
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97656348"
---
# <a name="create-a-data-streaming-job-in-azure-sql-edge"></a>Создание задания потоковой передачи данных в SQL Azure для пограничных вычислений 

В этой статье описано, как создать задание потоковой передачи T-SQL в SQL Azure для пограничных вычислений. Вы создаете входные и выходные объекты внешнего потока, а затем определяете потоковый запрос в ходе создания задания потоковой передачи.

## <a name="configure-the-external-stream-input-and-output-objects"></a>Настройка входных и выходных объектов внешнего потока

Потоковая передача T-SQL использует функциональные возможности внешнего источника данных SQL Server, чтобы определить источники данных, связанные с входными и выходными объектами внешнего потока в задании потоковой передачи. Используйте следующие команды T-SQL, чтобы создать входной или выходной объект внешнего потока.

- [CREATE EXTERNAL FILE FORMAT (Transact-SQL)](/sql/t-sql/statements/create-external-file-format-transact-sql)

- [CREATE EXTERNAL DATA SOURCE (Transact-SQL)](/sql/t-sql/statements/create-external-data-source-transact-sql)

- [CREATE EXTERNAL STREAM (Transact-SQL)](#example-create-an-external-stream-object-to-azure-sql-database)

Кроме того, если в качестве потока вывода используется SQL Azure для пограничных вычислений, SQL Server или База данных SQL Azure, необходимо [СОЗДАТЬ УЧЕТНЫЕ ДАННЫЕ ДЛЯ БАЗЫ ДАННЫХ (Transact-SQL)](/sql/t-sql/statements/create-database-scoped-credential-transact-sql). Эта команда T-SQL определяет учетные данные для доступа к базе данных.

### <a name="supported-input-and-output-stream-data-sources"></a>Поддерживаемые источники данных для входного потока и потока вывода

В настоящее время SQL Azure для пограничных вычислений поддерживает только следующие источники данных в качестве входных потоков и потоков вывода.

| Тип источника данных | Входные данные | Выходные данные | Описание |
|------------------|-------|--------|------------------|
| Центр Azure IoT Edge | Да | Да | Источник данных для чтения и записи потоковых данных в центр Azure IoT Edge. Подробнее см. в статье об [Azure IoT Edge](../iot-edge/iot-edge-runtime.md#iot-edge-hub).|
| База данных SQL | Нет | Да | Соединение с источником данных для записи потоковых данных в Базу данных SQL. Базой данных может быть локальная база данных SQL Azure для пограничных вычислений, удаленная база данных SQL Server или База данных SQL Azure.|
| Kafka | Да | Нет | Источник данных для чтения потоковых данных из раздела Kafka. Этот адаптер сейчас доступен только в Azure SQL для пограничных вычислений версии Intel или AMD. Он недоступен в Azure SQL для пограничных вычислений версии ARM64.|

### <a name="example-create-an-external-stream-inputoutput-object-for-azure-iot-edge-hub"></a>Пример. Создание входного или выходного объекта внешнего потока для центра Azure IoT Edge

В следующем примере создается объект внешнего потока для центра Azure IoT Edge. Чтобы создать источник входных и выходных данных внешнего потока для центра Azure IoT Edge, сначала необходимо создать формат внешнего файла для структуры данных, которые считываются или записываются.

1. Создайте формат внешнего файла типа JSON.

    ```sql
    Create External file format InputFileFormat
    WITH 
    (  
       format_type = JSON,
    )
    go
    ```

2. Создайте внешний источник данных для центра Azure IoT Edge. Следующий скрипт T-SQL создает подключение источника данных к центру IoT Edge, который выполняется на том же узле Docker, что и SQL Azure для пограничных вычислений.

    ```sql
    CREATE EXTERNAL DATA SOURCE EdgeHubInput 
    WITH 
    (
        LOCATION = 'edgehub://'
    )
    go
    ```

3. Создайте объект внешнего потока для центра Azure IoT Edge. Следующий скрипт T-SQL создает объект потока для центра IoT Edge. В случае с объектом потока центра IoT Edge параметр LOCATION представляет собой имя раздела или канала центра IoT Edge, в котором выполняется операция считывания или записи.

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

### <a name="example-create-an-external-stream-object-to-azure-sql-database"></a>Пример. Создание объекта внешнего потока для Базы данных SQL Azure

В приведенном ниже примере создается объект внешнего потока для локальной базы данных SQL Azure для пограничных вычислений. 

1. Создайте главный ключ в базе данных. Это необходимо для шифрования секрета учетных данных.

    ```sql
    CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<<Strong_Password_For_Master_Key_Encryption>>';
    ```

2. Создайте учетные данные базы данных для доступа к источнику SQL Server. В следующем примере создаются учетные данные для внешнего источника данных, где IDENTITY = имя пользователя, а SECRET = пароль.

    ```sql
    CREATE DATABASE SCOPED CREDENTIAL SQLCredential
    WITH IDENTITY = '<SQL_Login>', SECRET = '<SQL_Login_PASSWORD>'
    go
    ```

3. Создайте внешний источник данных с помощью инструкции CREATE EXTERNAL DATA SOURCE. Следующий пример:

    * Создает внешний источник данных с именем *LocalSQLOutput*.
    * Определяет внешний источник данных (LOCATION = <vendor>://<server>[:<port>]). В примере он указывает на локальный экземпляр SQL Azure для пограничных вычислений.
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

4. Создайте объект внешнего потока. В приведенном ниже примере в базе данных *MySQLDatabase* создается объект внешнего потока, указывающий на таблицу *dbo.TemperatureMeasurements*.

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

В приведенном ниже примере создается объект внешнего потока для локальной базы данных SQL Azure для пограничных вычислений. В приведенном ниже примере предполагается, что сервер Kafka настроен для анонимного доступа. 

1. Создайте внешний источник данных с помощью инструкции CREATE EXTERNAL DATA SOURCE. Следующий пример:

    ```sql
    Create EXTERNAL DATA SOURCE [KafkaInput] 
    With
    (
        LOCATION = N'kafka://<kafka_bootstrap_server_name_ip>:<port_number>'
    )
    GO
    ```
2. Создайте формат внешнего файла для входных данных Kafka. В следующем примере создается формат файла JSON с со сжатием GZip. 

   ```sql
   CREATE EXTERNAL FILE FORMAT JsonGzipped  
    WITH 
    (  
        FORMAT_TYPE = JSON , 
        DATA_COMPRESSION = 'org.apache.hadoop.io.compress.GzipCodec' 
    )
   ```

3. Создайте объект внешнего потока. В следующем примере создается объект внешнего потока, указывающий на раздел Kafka `*TemperatureMeasurement*`.

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

Чтобы определить потоковые запросы и создать задание потоковой передачи, используйте системную хранимую процедуру `sys.sp_create_streaming_job`. Хранимая процедура `sp_create_streaming_job` принимает следующие параметры:

- `job_name` — имя задания потоковой передачи. Имена заданий потоковой передачи уникальны в пределах экземпляра.
- `statement` — операторы потокового запроса на основе [языка запросов Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference).

В приведенном ниже примере создается простое задание потоковой передачи с одним потоковым запросом. Этот запрос считывает входные данные из центра IoT Edge и выполняет запись в `dbo.TemperatureMeasurements` базы данных.

```sql
EXEC sys.sp_create_streaming_job @name=N'StreamingJob1',
@statement= N'Select * INTO TemperatureMeasurements from MyEdgeHubInput'
```

В следующем примере создается более сложное задание потоковой передачи с несколькими разными запросами. Один из этих запросов использует встроенную функцию `AnomalyDetection_ChangePoint` для обнаружения аномалий в данных температуры.

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

Чтобы запустить задание потоковой передачи в SQL Azure для пограничных вычислений, выполните хранимую процедуру `sys.sp_start_streaming_job`. В качестве входного объекта для хранимой процедуры необходимо указать имя задания потоковой передачи, которое нужно запустить.

```sql
exec sys.sp_start_streaming_job @name=N'StreamingJob1'
go
```

Чтобы завершить задание потоковой передачи, выполните хранимую процедуру `sys.sp_stop_streaming_job`. В качестве входного объекта для хранимой процедуры необходимо указать имя задания потоковой передачи, которое нужно остановить.

```sql
exec sys.sp_stop_streaming_job @name=N'StreamingJob1'
go
```

Чтобы прервать (удалить) задание потоковой передачи, выполните хранимую процедуру `sys.sp_drop_streaming_job`. В качестве входного объекта для хранимой процедуры необходимо указать имя задания потоковой передачи, которое нужно прервать.

```sql
exec sys.sp_drop_streaming_job @name=N'StreamingJob1'
go
```

Чтобы получить состояние текущего задания потоковой передачи, выполните хранимую процедуру `sys.sp_get_streaming_job`. В качестве входного объекта для хранимой процедуры необходимо указать имя задания потоковой передачи, которое нужно прервать. Хранимая процедура выводит имя и текущее состояние задания потоковой передачи.

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

Возможные состояния задания потоковой передачи:

| Состояние | Описание |
|--------| ------------|
| Создание | Задание потоковой передачи создано, но не запущено. |
| Запуск | Задание потоковой передачи запускается. |
| Бездействие | Задание потоковой передачи выполняется, но входные данные для обработки отсутствуют. |
| Обработка | Задание потоковой передачи выполняется, и входные данные обрабатываются. Это свидетельствует о работоспособности задания потоковой передачи. |
| Деградация | Задание потоковой передачи выполняется, но при обработке входных данных возникли некритические ошибки. Входное задание продолжит выполняться, но входные данные, вызывающие ошибки, будут удаляться. |
| Остановлена | Выполнение задания потоковой передачи остановлено. |
| Сбой | При выполнении задания потоковой передачи произошел сбой. Обычно это указывает на неустранимую ошибку при обработке. |

## <a name="next-steps"></a>Дальнейшие действия

- [Просмотр метаданных, связанных с заданиями потоковой передачи в SQL Azure для пограничных вычислений](streaming-catalog-views.md) 
- [Создание внешнего потока](create-external-stream-transact-sql.md)