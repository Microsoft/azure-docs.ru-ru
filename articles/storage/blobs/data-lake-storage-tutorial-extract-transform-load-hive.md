---
title: Руководство по Извлечение, преобразование и загрузка данных с помощью Azure HDInsight
description: В этом руководстве вы узнаете, как извлекать данные из необработанного набора данных в формате CSV, преобразовывать их с помощью Apache Hive в Azure HDInsight и загружать преобразованные данные в базу данных SQL Azure с помощью Sqoop.
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: tutorial
ms.date: 11/19/2019
ms.author: normesta
ms.reviewer: jamesbak
ms.openlocfilehash: f8210c3bc0437180ace110f8decd9f83e18650ed
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98661938"
---
# <a name="tutorial-extract-transform-and-load-data-by-using-azure-hdinsight"></a>Руководство по Извлечение, преобразование и загрузка данных с помощью Azure HDInsight

В этом руководстве рассматривается выполнение операций извлечения, преобразования и загрузки данных. Вы берете необработанный файл данных CSV, импортируете его в кластер Azure HDInsight, преобразовываете его с помощью Apache Hive и загружаете в базу данных SQL Azure с помощью Apache Sqoop.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Извлечение и отправка данных в кластер HDInsight.
> * Преобразование данных с помощью Apache Hive.
> * Загрузка данных в Базу данных SQL Azure с помощью Sqoop.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

* **Учетная запись Azure Data Lake Storage 2-го поколения, настроенная для HDInsight**.

    См. раздел [Use Azure Data Lake Storage Gen2 with Azure HDInsight clusters](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md) (Использование хранилища Azure Data Lake поколения 2 с кластерами Azure HDInsight).

* **Кластер Hadoop в HDInsight на платформе Linux**.

    См. [Краткое руководство. Начало работы с Apache Hadoop и Apache Hive в Azure HDInsight с помощью портала Azure](../../hdinsight/hadoop/apache-hadoop-linux-create-cluster-get-started-portal.md).

* **База данных SQL Azure**. Вы используете Базу данных SQL Azure в качестве целевого хранилища данных. Если у вас нет базы данных в Базе данных SQL, вы можете создать ее, выполнив инструкции из статьи [Краткое руководство. Создание отдельной базы данных в Базе данных SQL Azure](../../azure-sql/database/single-database-create-quickstart.md).

* **Azure CLI.** Если вы не установили интерфейс командной строки Azure, обратитесь к разделу [Установка Azure CLI](/cli/azure/install-azure-cli).

* **Клиент Secure Shell (SSH)** . Дополнительные сведения см. в руководстве по [подключению к HDInsight (Hadoop) с помощью SSH](../../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="download-the-flight-data"></a>Скачивание данных о рейсах

1. Перейдите на страницу [бюро транспортной статистики при администрации по исследованиям и инновационным технологиям](https://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time).

2. На странице выберите следующие значения:

   | Имя | Значение |
   | --- | --- |
   | Фильтр года |2013 |
   | Период фильтра |Январь |
   | Поля |Year, FlightDate, Reporting_Airline, IATA_CODE_Reporting_Airline, Flight_Number_Reporting_Airline, OriginAirportID, Origin, OriginCityName, OriginState, DestAirportID, Dest, DestCityName, DestState, DepDelayMinutes, ArrDelay, ArrDelayMinutes, CarrierDelay, WeatherDelay, NASDelay, SecurityDelay, LateAircraftDelay. |
   
   Очистите все остальные поля.

3. Выберите **Скачать**. Вы получите ZIP-файл с выбранными полями данных.

## <a name="extract-and-upload-the-data"></a>Извлечение и отправка данных

В этом разделе вы отправите данные в кластер HDInsight, а затем скопируете эти данные в учетную запись Azure Data Lake Storage 2-го поколения.

1. Откройте командную строку и воспользуйтесь следующей командой безопасного копирования (SCP), чтобы передать ZIP-файл в головной узел кластера HDInsight:

   ```bash
   scp <file-name>.zip <ssh-user-name>@<cluster-name>-ssh.azurehdinsight.net:<file-name.zip>
   ```

   * Замените заполнитель `<file-name>` именем вашего ZIP-файла.
   * Замените заполнитель `<ssh-user-name>` именем для входа SSH для кластера HDInsight.
   * Замените заполнитель `<cluster-name>` именем кластера HDInsight.

   Если для аутентификации входа посредством SSH используется пароль, будет предложено ввести пароль.

   Если используется открытый ключ, может потребоваться использовать параметр `-i` и указать путь к соответствующему закрытому ключу. Например, `scp -i ~/.ssh/id_rsa <file_name>.zip <user-name>@<cluster-name>-ssh.azurehdinsight.net:`.

2. После завершения отправки можно подключиться к кластеру с помощью SSH. Введите приведенную ниже команду в окне командной строки.

   ```bash
   ssh <ssh-user-name>@<cluster-name>-ssh.azurehdinsight.net
   ```

3. Чтобы распаковать ZIP-файл, используйте следующую команду:

   ```bash
   unzip <file-name>.zip
   ```

   Она извлекает **CSV**-файл.

4. Используйте следующую команду для создания контейнера Data Lake Storage 2-го поколения.

   ```bash
   hadoop fs -D "fs.azure.createRemoteFileSystemDuringInitialization=true" -ls abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/
   ```

   Замените заполнитель `<container-name>` именем, которое хотите присвоить своему контейнеру.

   Замените заполнитель `<storage-account-name>` именем вашей учетной записи хранения.

5. Создайте каталог с помощью следующей команды:

   ```bash
   hdfs dfs -mkdir -p abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/data
   ```

6. Используйте следующую команду для копирования *CSV*-файла в каталог:

   ```bash
   hdfs dfs -put "<file-name>.csv" abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/data/
   ```

   Возьмите имя файла в кавычки, если оно содержит пробелы или специальные символы.

## <a name="transform-the-data"></a>Преобразование данных

В этом разделе вы используете клиент Beeline для выполнения задания Apache Hive.

Как часть задания Apache Hive вы импортируете данные из CSV-файла в таблицу Apache Hive с именем **delays**.

1. В командной строке SSH, которую вы уже использовали для кластера HDInsight, выполните следующую команду для создания и редактирования нового файла **flightdelays.hql**.

   ```bash
   nano flightdelays.hql
   ```

2. Измените следующий текст, заменив заполнители `<container-name>` и `<storage-account-name>` именем контейнера и учетной записи хранения. Затем скопируйте и вставьте текст в nano-консоль, нажав клавишу SHIFT и правую кнопку мыши одновременно.

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
    LOCATION 'abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/data';

    -- Drop the delays table if it exists
    DROP TABLE delays;
    -- Create the delays table and populate it with data
    -- pulled in from the CSV file (via the external table defined previously)
    CREATE TABLE delays
    LOCATION 'abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/processed'
    AS
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

3. Чтобы сохранить файл, нажмите клавиши CTRL+X, а затем при появлении запроса введите `Y`.

4. Для запуска Hive и выполнения файла **flightdelays.hql** используйте следующую команду:

   ```bash
   beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http' -f flightdelays.hql
   ```

5. По завершении выполнения сценария __flightdelays.hql__ используйте следующую команду, чтобы открыть интерактивный сеанс Beeline:

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

   Вы получите список городов, рейсы в которых задержаны из-за погодных условий, а также среднее время задержки. Он будет сохранен в `abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/output`. Позже Sqoop считает данные из этого расположения и экспортирует их в базу данных SQL Azure.

7. Чтобы выйти из Beeline, введите `!quit` в командной строке.

## <a name="create-a-sql-database-table"></a>Создание таблицы базы данных SQL

Вам требуется указать имя сервера из Базы данных SQL для этой операции. Выполните следующие действия, чтобы найти имя вашего сервера.

1. Перейдите на [портал Azure](https://portal.azure.com).

2. Выберите **Базы данных SQL**.

3. Выполните фильтрацию по имени базы данных, которую вы выбрали. Имя сервера указано в столбце **Имя сервера**.

4. Выполните фильтрацию по имени базы данных, которую вы хотите использовать. Имя сервера указано в столбце **Имя сервера**.

    ![Получение сведений о сервере SQL Azure](./media/data-lake-storage-tutorial-extract-transform-load-hive/get-azure-sql-server-details.png "Получение сведений о сервере SQL Azure")

    Существует множество способов подключения к базе данных SQL и создания таблицы. В приведенных ниже действиях используется [FreeTDS](https://www.freetds.org/) из кластера HDInsight.

5. Чтобы установить FreeTDS, выполните следующую команду с помощью SSH-подключения к кластеру:

   ```bash
   sudo apt-get --assume-yes install freetds-dev freetds-bin
   ```

6. После завершения установки используйте следующую команду для подключения к Базе данных SQL.

   ```bash
   TDSVER=8.0 tsql -H '<server-name>.database.windows.net' -U '<admin-login>' -p 1433 -D '<database-name>'
    ```
   * Замените заполнитель `<server-name>` именем логического сервера SQL Server.

   * Замените заполнитель `<admin-login>` именем для входа администратора для Базы данных SQL.

   * Замените заполнитель `<database-name>` именем базы данных.

   При появлении запроса введите пароль администратора Базы данных SQL.

   Должен появиться результат, аналогичный приведенному ниже тексту.

   ```
   locale is "en_US.UTF-8"
   locale charset is "UTF-8"
   using default charset "UTF-8"
   Default database being set to sqooptest
   1>
   ```

7. В окне запроса `1>` введите следующие инструкции:

   ```hiveql
   CREATE TABLE [dbo].[delays](
   [OriginCityName] [nvarchar](50) NOT NULL,
   [WeatherDelay] float,
   CONSTRAINT [PK_delays] PRIMARY KEY CLUSTERED
   ([OriginCityName] ASC))
   GO
   ```

8. Если вводится инструкция `GO`, то оцениваются предыдущие инструкции.

   Этот запрос создает таблицу **delays** с кластеризованным индексом.

9. Используйте следующий запрос для проверки создания таблицы.

   ```hiveql
   SELECT * FROM information_schema.tables
   GO
   ```

   Результат будет аналогичен приведенному ниже:

   ```
   TABLE_CATALOG   TABLE_SCHEMA    TABLE_NAME      TABLE_TYPE
   databaseName       dbo             delays        BASE TABLE
   ```

10. Enter `exit` at the `1>` , чтобы выйти из служебной программы tsql.

## <a name="export-and-load-the-data"></a>Экспорт и загрузка данных

В предыдущих разделах вы скопировали преобразованные данные в расположение `abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/output`. В этом разделе вы с помощью Sqoop экспортируете данные из папки `abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/output` в созданную в Базе данных SQL Azure таблицу.

1. Чтобы проверить, видно ли в Sqoop базу данных SQL, используйте следующую команду:

   ```bash
   sqoop list-databases --connect jdbc:sqlserver://<SERVER_NAME>.database.windows.net:1433 --username <ADMIN_LOGIN> --password <ADMIN_PASSWORD>
   ```

   Эта команда выводит список баз данных, включая базу данных, в которой вы создали таблицу **delays**.

2. Для экспорта данных из таблицы **hivesampletable** в таблицу **delays** используйте следующую команду:

   ```bash
   sqoop export --connect 'jdbc:sqlserver://<SERVER_NAME>.database.windows.net:1433;database=<DATABASE_NAME>' --username <ADMIN_LOGIN> --password <ADMIN_PASSWORD> --table 'delays' --export-dir 'abfs://<container-name>@.dfs.core.windows.net/tutorials/flightdelays/output' --fields-terminated-by '\t' -m 1
   ```

   Sqoop подключается к базе данных, которая содержит таблицу **delays**, и экспортирует данные из каталога `/tutorials/flightdelays/output` в таблицу **delays**.

3. Когда команда `sqoop` будет выполнена, используйте служебную программу tsql для подключения к базе данных:

   ```bash
   TDSVER=8.0 tsql -H <SERVER_NAME>.database.windows.net -U <ADMIN_LOGIN> -P <ADMIN_PASSWORD> -p 1433 -D <DATABASE_NAME>
   ```

4. Используйте следующие инструкции, чтобы проверить состояние экспорта данных в таблицу **delays**:

   ```sql
   SELECT * FROM delays
   GO
   ```

   Вы увидите список данных в таблице. Таблица содержит название города и среднее время задержки рейса для этого города.

5. Введите `exit` для выхода из служебной программы tsql.

## <a name="clean-up-resources"></a>Очистка ресурсов

Все ресурсы, используемые в этом руководстве, предварительно созданы. Очистка не требуется.

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать дополнительные возможности работы с данными в HDInsight, ознакомьтесь со статьей по следующей ссылке:

> [!div class="nextstepaction"]
> [Использование Azure Data Lake Storage Gen2 с кластерами Azure HDInsight](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)