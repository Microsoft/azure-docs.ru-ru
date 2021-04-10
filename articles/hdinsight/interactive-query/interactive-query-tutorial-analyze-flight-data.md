---
title: Руководство по операциям извлечения, преобразования и загрузки (ETL) с помощью Interactive Query в Azure HDInsight
description: Из этого руководства вы узнаете, как извлекать данные из набора необработанных данных формата CSV, преобразовывать их с помощью Interactive Query в HDInsight и загружать преобразованные данные в Базу данных SQL Azure с помощью Apache Sqoop.
ms.service: hdinsight
ms.topic: tutorial
ms.custom: hdinsightactive,mvc
ms.date: 07/02/2019
ms.openlocfilehash: 1aab6b4fe4ffafb3cbba60b2b3774d2577d3d53c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103562473"
---
# <a name="tutorial-extract-transform-and-load-data-using-interactive-query-in-azure-hdinsight"></a>Руководство по извлечению, преобразованию и загрузке данных с помощью интерактивного запроса в Azure HDInsight

В этом руководстве показано, как скачать CSV-файл с общедоступными данными об авиарейсах, импортировать их в хранилище кластера HDInsight, а затем преобразовать эти данные с помощью Interactive Query в Azure HDInsight. После преобразования вы загрузите эти данные в Базу данных SQL Azure с использованием [Apache Sqoop](https://sqoop.apache.org/).

В рамках этого руководства рассматриваются следующие задачи:

> [!div class="checklist"]
> * загрузка образца данных о рейсах;
> * Отправка данных в кластер HDInsight
> * преобразование данных с помощью интерактивного запроса;
> * создание таблицы в Базе данных SQL Azure;
> * использование Sqoop для экспорта данных в Базу данных SQL Azure.

## <a name="prerequisites"></a>Предварительные требования

* Кластер интерактивного запроса в HDInsight. Ознакомьтесь со статьей [Create Linux-based clusters in HDInsight by using the Azure portal](../hdinsight-hadoop-create-linux-clusters-portal.md) (Создание кластеров под управлением Linux в HDInsight с помощью портала Azure) и выберите **Интерактивные запросы** для параметра **Тип кластера**.

* База данных в службе "База данных SQL Azure". Вы используете базу данных в качестве конечного хранилища данных. Если у вас нет базы данных в Базе данных SQL Azure, вы можете создать ее, выполнив инструкции из статьи [Краткое руководство. Создание отдельной базы данных в Базе данных SQL Azure](../../azure-sql/database/single-database-create-quickstart.md).

* Клиент SSH. Дополнительные сведения см. в руководстве по [подключению к HDInsight (Apache Hadoop) с помощью SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="download-the-flight-data"></a>Скачивание данных о рейсах

1. Перейдите на страницу [бюро транспортной статистики при администрации по исследованиям и инновационным технологиям](https://www.transtats.bts.gov/DL_SelectFields.asp?gnoyr_VQ=FGJ).

2. Очистите все поля на странице, а затем выберите следующие значения:

   | Имя | Значение |
   | --- | --- |
   | Фильтр года |2019 |
   | Период фильтра |Январь |
   | Поля |`Year, FlightDate, Reporting_Airline, DOT_ID_Reporting_Airline, Flight_Number_Reporting_Airline, OriginAirportID, Origin, OriginCityName, OriginState, DestAirportID, Dest, DestCityName, DestState, DepDelayMinutes, ArrDelay, ArrDelayMinutes, CarrierDelay, WeatherDelay, NASDelay, SecurityDelay, LateAircraftDelay`. |

3. Выберите **Скачать**. Вы получите ZIP-файл с выбранными полями данных.

## <a name="upload-data-to-an-hdinsight-cluster"></a>Отправка данных в кластер HDInsight

Существует множество способов отправки данных в хранилище, связанное с кластером HDInsight. В этом разделе для отправки данных используется `scp`. Дополнительные сведения о других способах отправки данных см. в статье [Отправка данных для заданий Hadoop в HDInsight](../hdinsight-upload-data.md).

1. Отправьте ZIP-файл в головной узел кластера HDInsight. Измените команду ниже, заменив `FILENAME` именем ZIP-файла и `CLUSTERNAME` именем кластера HDInsight. Затем откройте командную строку, задайте для рабочей папки расположение файла, а затем введите команду.

    ```cmd
    scp FILENAME.zip sshuser@CLUSTERNAME-ssh.azurehdinsight.net:FILENAME.zip
    ```

    При появлении запроса введите yes (да) или no (нет), чтобы продолжить. При вводе текст не отображается в окне.

2. После завершения отправки можно подключиться к кластеру с помощью SSH. Измените команду ниже, заменив `CLUSTERNAME` именем кластера HDInsight. Затем введите следующую команду:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

3. Настройте переменную среды после установления SSL-подключения. Замените `FILE_NAME`, `SQL_SERVERNAME`, `SQL_DATABASE`, `SQL_USER` и `SQL_PASWORD` соответствующими значениями. Затем введите команду:

    ```bash
    export FILENAME=FILE_NAME
    export SQLSERVERNAME=SQL_SERVERNAME
    export DATABASE=SQL_DATABASE
    export SQLUSER=SQL_USER
    export SQLPASWORD='SQL_PASWORD'
    ```

4. Распакуйте ZIP-файл, введя следующую команду:

    ```bash
    unzip $FILENAME.zip
    ```

5. Создайте каталог в хранилище HDInsight, затем скопируйте данный CSV-файл в этот каталог, введя следующую команду:

    ```bash
    hdfs dfs -mkdir -p /tutorials/flightdelays/data
    hdfs dfs -put $FILENAME.csv /tutorials/flightdelays/data/
    ```

## <a name="transform-data-using-a-hive-query"></a>Преобразование данных с помощью запроса Hive

Существует множество способов запуска задания Hive в кластере HDInsight. В этом разделе для выполнения задания используется клиент [Beeline](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients#HiveServer2Clients-Beeline%E2%80%93CommandLineShell). Сведения о других методах выполнения задания Hive см. в статье [Обзор Apache Hive и HiveQL в Azure HDInsight](../hadoop/hdinsight-use-hive.md).

В рамках задания Hive вы импортируете данные из CSV-файла в таблицу Hive с именем **Delays**.

1. В командной строке SSH, которую вы уже использовали для кластера HDInsight, выполните следующую команду для создания и редактирования нового файла **flightdelays.hql**.

    ```bash
    nano flightdelays.hql
    ```

2. В качестве содержимого файла добавьте следующий текст:

    ```hiveql
    DROP TABLE delays_raw;
    -- Creates an external table over the csv file
    CREATE EXTERNAL TABLE delays_raw (
        YEAR string,
        FL_DATE string,
        UNIQUE_CARRIER string,
        CARRIER string,
        FL_NUM string,
        ORIGIN_AIRPORT_ID string,
        ORIGIN string,
        ORIGIN_CITY_NAME string,
        ORIGIN_CITY_NAME_TEMP string,
        ORIGIN_STATE_ABR string,
        DEST_AIRPORT_ID string,
        DEST string,
        DEST_CITY_NAME string,
        DEST_CITY_NAME_TEMP string,
        DEST_STATE_ABR string,
        DEP_DELAY_NEW float,
        ARR_DELAY_NEW float,
        CARRIER_DELAY float,
        WEATHER_DELAY float,
        NAS_DELAY float,
        SECURITY_DELAY float,
        LATE_AIRCRAFT_DELAY float)
    -- The following lines describe the format and location of the file
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
    LINES TERMINATED BY '\n'
    STORED AS TEXTFILE
    LOCATION '/tutorials/flightdelays/data';

    -- Drop the delays table if it exists
    DROP TABLE delays;
    -- Create the delays table and populate it with data
    -- pulled in from the CSV file (via the external table defined previously)
    CREATE TABLE delays AS
    SELECT YEAR AS year,
        FL_DATE AS flight_date,
        substring(UNIQUE_CARRIER, 2, length(UNIQUE_CARRIER) -1) AS unique_carrier,
        substring(CARRIER, 2, length(CARRIER) -1) AS carrier,
        substring(FL_NUM, 2, length(FL_NUM) -1) AS flight_num,
        ORIGIN_AIRPORT_ID AS origin_airport_id,
        substring(ORIGIN, 2, length(ORIGIN) -1) AS origin_airport_code,
        substring(ORIGIN_CITY_NAME, 2) AS origin_city_name,
        substring(ORIGIN_STATE_ABR, 2, length(ORIGIN_STATE_ABR) -1)  AS origin_state_abr,
        DEST_AIRPORT_ID AS dest_airport_id,
        substring(DEST, 2, length(DEST) -1) AS dest_airport_code,
        substring(DEST_CITY_NAME,2) AS dest_city_name,
        substring(DEST_STATE_ABR, 2, length(DEST_STATE_ABR) -1) AS dest_state_abr,
        DEP_DELAY_NEW AS dep_delay_new,
        ARR_DELAY_NEW AS arr_delay_new,
        CARRIER_DELAY AS carrier_delay,
        WEATHER_DELAY AS weather_delay,
        NAS_DELAY AS nas_delay,
        SECURITY_DELAY AS security_delay,
        LATE_AIRCRAFT_DELAY AS late_aircraft_delay
    FROM delays_raw;
    ```

3. Чтобы сохранить файл, нажмите клавиши **Ctrl + X**, затем **Y** и "Ввод".

4. Для запуска Hive и выполнения файла **flightdelays.hql** используйте следующую команду:

    ```bash
    beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http' -f flightdelays.hql
    ```

5. По завершении выполнения сценария **flightdelays.hql** используйте следующую команду, чтобы открыть интерактивный сеанс Beeline:

    ```bash
    beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http'
    ```

6. При появлении командной строки `jdbc:hive2://localhost:10001/>` используйте приведенный ниже запрос, чтобы извлечь информацию из импортированных данных о задержке рейсов:

    ```hiveql
    INSERT OVERWRITE DIRECTORY '/tutorials/flightdelays/output'
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    SELECT regexp_replace(origin_city_name, '''', ''),
        avg(weather_delay)
    FROM delays
    WHERE weather_delay IS NOT NULL
    GROUP BY origin_city_name;
    ```

    Вы получите список городов, рейсы в которых задержаны из-за погодных условий, а также среднее время задержки. Он будет сохранен в `/tutorials/flightdelays/output`. Позже Sqoop считает данные из этого расположения и экспортирует их в базу данных SQL Azure.

7. Чтобы выйти из Beeline, введите `!quit` в командной строке.

## <a name="create-a-sql-database-table"></a>Создание таблицы базы данных SQL

Существует множество способов подключения к базе данных SQL и создания таблицы. В приведенных ниже действиях используется [FreeTDS](http://www.freetds.org/) из кластера HDInsight.

1. Чтобы установить FreeTDS, выполните следующую команду с помощью открытого SSH-подключения к кластеру:

    ```bash
    sudo apt-get --assume-yes install freetds-dev freetds-bin
    ```

2. После завершения установки используйте следующую команду для подключения к Базе данных SQL.

    ```bash
    TDSVER=8.0 tsql -H $SQLSERVERNAME.database.windows.net -U $SQLUSER -p 1433 -D $DATABASE -P $SQLPASWORD
    ```

    Должен появиться результат, аналогичный приведенному ниже тексту.

    ```
    locale is "en_US.UTF-8"
    locale charset is "UTF-8"
    using default charset "UTF-8"
    Default database being set to <yourdatabase>
    1>
    ```

3. В командной строке `1>` введите следующее:

    ```hiveql
    CREATE TABLE [dbo].[delays](
    [origin_city_name] [nvarchar](50) NOT NULL,
    [weather_delay] float,
    CONSTRAINT [PK_delays] PRIMARY KEY CLUSTERED
    ([origin_city_name] ASC))
    GO
    ```

    Если вводится инструкция `GO`, то оцениваются предыдущие инструкции. Эта инструкция создает таблицу **delays** с кластеризованным индексом.

    Используйте следующий запрос команду для проверки создания таблицы.

    ```hiveql
    SELECT * FROM information_schema.tables
    GO
    ```

    Результат будет аналогичен приведенному ниже:

    ```
    TABLE_CATALOG   TABLE_SCHEMA    TABLE_NAME      TABLE_TYPE
    databaseName       dbo             delays        BASE TABLE
    ```

4. Enter `exit` at the `1>` , чтобы выйти из служебной программы tsql.

## <a name="export-data-to-sql-database-using-apache-sqoop"></a>Экспорт данных в Базу данных SQL с помощью Apache Sqoop

В предыдущих разделах вы скопировали преобразованные данные в папку `/tutorials/flightdelays/output`. В этом разделе вы с помощью Sqoop экспортируете данные из папки `/tutorials/flightdelays/output` в созданную в Базе данных SQL Azure таблицу.

1. Проверьте, видно ли в Sqoop базу данных SQL, используя следующую команду:

    ```bash
    sqoop list-databases --connect jdbc:sqlserver://$SQLSERVERNAME.database.windows.net:1433 --username $SQLUSER --password $SQLPASWORD
    ```

    Эта команда выводит список баз данных, включая базу данных, в которой вы ранее создали таблицу `delays`.

2. Экспортируйте данные из `/tutorials/flightdelays/output` в таблицу `delays`, введя следующую команду:

    ```bash
    sqoop export --connect "jdbc:sqlserver://$SQLSERVERNAME.database.windows.net:1433;database=$DATABASE" --username $SQLUSER --password $SQLPASWORD --table 'delays' --export-dir '/tutorials/flightdelays/output' --fields-terminated-by '\t' -m 1
    ```

    Sqoop подключается к базе данных, которая содержит таблицу `delays`, и экспортирует данные из каталога `/tutorials/flightdelays/output` в таблицу `delays`.

3. Когда команда sqoop будет выполнена, используйте служебную программу tsql для подключения к базе данных, введя следующую команду:

    ```bash
    TDSVER=8.0 tsql -H $SQLSERVERNAME.database.windows.net -U $SQLUSER -p 1433 -D $DATABASE -P $SQLPASWORD
    ```

    Используйте следующие инструкции, чтобы проверить состояние экспорта данных в таблицу delays:

    ```sql
    SELECT * FROM delays
    GO
    ```

    Вы увидите список данных в таблице. Таблица содержит название города и среднее время задержки рейса для этого города.

    Введите `exit` для выхода из служебной программы tsql.

## <a name="clean-up-resources"></a>Очистка ресурсов

После завершения работы с этим руководством кластер можно удалить. В случае с HDInsight ваши данные хранятся в службе хранилища Azure, что позволяет безопасно удалить неиспользуемый кластер. Плата за кластеры HDInsight взимается, даже когда они не используются. Так как затраты на кластер во много раз превышают затраты на хранилище, экономически целесообразно удалять неиспользуемые кластеры.

Инструкции по удалению кластера см. в статье [Delete an HDInsight cluster using your browser, PowerShell, or the Azure CLI](../hdinsight-delete-cluster.md) (Удаление кластера HDInsight с помощью браузера, PowerShell или Azure CLI).

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого учебника вы извлекли CSV-файл с необработанными данными, импортировали их в хранилище кластера HDInsight, а затем преобразовали эти данные с помощью интерактивного запроса в Azure HDInsight.  Перейдите к следующему учебнику, чтобы ознакомиться со сведениями о соединителе хранилища Apache Hive.

> [!div class="nextstepaction"]
> [Integrate Apache Spark and Apache Hive with the Hive Warehouse Connector](./apache-hive-warehouse-connector.md) (Интеграция Apache Spark и Apache Hive с помощью соединителя хранилища Hive)
