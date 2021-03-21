---
title: Использование Apache Spark для чтения и записи данных в базе данных SQL Azure
description: Узнайте, как настроить подключение между кластером HDInsight Spark и базой данных SQL Azure. Чтение данных, запись данных и потоковая передача данных в базу данных SQL
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 04/20/2020
ms.openlocfilehash: 4d42cedbc5dc20c929703be106e732b4806f3902
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98940599"
---
# <a name="use-hdinsight-spark-cluster-to-read-and-write-data-to-azure-sql-database"></a>Использование кластера HDInsight Spark для чтения и записи данных в базе данных SQL Azure

Узнайте, как подключить кластер Apache Spark в Azure HDInsight к базе данных SQL Azure. Затем чтение, запись и потоковая передача данных в базу данных SQL. В инструкциях, приведенных в этой статье, используется Jupyter Notebook для выполнения фрагментов кода Scala. Однако можно создать автономное приложение в Scala или Python и выполнить те же задачи.

## <a name="prerequisites"></a>Предварительные условия

* Кластер Azure HDInsight Spark.  Инструкции см. в статье [Создание кластера Apache Spark в Azure HDInsight](apache-spark-jupyter-spark-sql.md).

* База данных SQL Azure. Следуйте инструкциям в статье [Создание базы данных в базе данных SQL Azure](../../azure-sql/database/single-database-create-quickstart.md). Создайте базу данных с примером схемы и данными **AdventureWorksLT**. Кроме того, обязательно создайте правило брандмауэра уровня сервера, чтобы разрешить IP-адресу клиента доступ к базе данных SQL. Инструкции по добавлению правила брандмауэра находятся в той же статье. После создания базы данных SQL убедитесь, что вы хотите использовать следующие значения. Они понадобятся вам для подключения к базе данных из кластера Spark.

    * Имя сервера.
    * имя базы данных.
    * Имя пользователя и пароль администратора базы данных SQL Azure.

* SQL Server Management Studio (SSMS). Инструкции см. в статье [Подключайтесь к базе данных Azure SQL и создавайте запросы к ней с помощью SQL Server Management Studio](../../azure-sql/database/connect-query-ssms.md).

## <a name="create-a-jupyter-notebook"></a>Создание записной книжки Jupyter

Начните с создания Jupyter Notebook, связанного с кластером Spark. Она понадобится для выполнения фрагментов кода, используемых в этой статье.

1. Откройте кластер на [портале Azure](https://portal.azure.com/).
1. В правой части выберите **Jupyter Notebook** **панели мониторинга кластера** .  Если **панели мониторинга кластера** не отображаются, выберите **Обзор** в меню слева. При появлении запроса введите учетные данные администратора для кластера.

    ![Jupyter Notebook на Apache Spark](./media/apache-spark-connect-to-sql-database/hdinsight-spark-cluster-dashboard-jupyter-notebook.png "Jupyter Notebook в Spark")

   > [!NOTE]  
   > Вы также можете получить доступ к Jupyter Notebook в кластере Spark, открыв следующий URL-адрес в браузере. Замените **CLUSTERNAME** именем кластера:
   >
   > `https://CLUSTERNAME.azurehdinsight.net/jupyter`

1. В Jupyter Notebook в правом верхнем углу щелкните **создать**, а затем нажмите **Spark** , чтобы создать записную книжку Scala. Записные книжки Jupyter в кластере HDInsight Spark также предоставляют ядро **PySpark** для приложений python2 и ядро **PySpark3** для приложений Python3. В этой статье мы создаем записную книжку Scala.

    ![Ядра для Jupyter Notebook в Spark](./media/apache-spark-connect-to-sql-database/kernel-jupyter-notebook-on-spark.png "Ядра для Jupyter Notebook в Spark")

    Дополнительные сведения о ядрах см. в статье [использование Jupyter Notebook ядер с Apache Spark кластерами в HDInsight](apache-spark-jupyter-notebook-kernels.md).

   > [!NOTE]  
   > В этой статье мы используем ядро Spark (Scala), так как потоковая передача данных из Spark в базу данных SQL в настоящее время поддерживается только в Scala и Java. Несмотря на то что чтение и запись в SQL можно выполнить с помощью Python, для обеспечения согласованности в этой статье мы используем Scala для всех трех операций.

1. Откроется новая Записная книжка с именем по умолчанию без **названия**. Щелкните это имя и введите имя по своему усмотрению.

    ![Указание имени для записной книжки](./media/apache-spark-connect-to-sql-database/hdinsight-spark-jupyter-notebook-name.png "Указание имени для записной книжки")

Теперь можно приступить к созданию приложения.

## <a name="read-data-from-azure-sql-database"></a>Чтение данных из базы данных SQL Azure

В этом разделе вы выполняете операцию чтения данных из таблицы (например, **SalesLT.Address**), которая находится в базе данных AdventureWorks.

1. В новом Jupyter Notebook в ячейке кода вставьте приведенный ниже фрагмент и замените значения заполнителей значениями для базы данных.

    ```scala
    // Declare the values for your database

    val jdbcUsername = "<SQL DB ADMIN USER>"
    val jdbcPassword = "<SQL DB ADMIN PWD>"
    val jdbcHostname = "<SQL SERVER NAME HOSTING SDL DB>" //typically, this is in the form or servername.database.windows.net
    val jdbcPort = 1433
    val jdbcDatabase ="<AZURE SQL DB NAME>"
    ```

    Чтобы выполнить ячейку кода, нажмите клавиши **SHIFT+ВВОД**.  

1. Используйте приведенный ниже фрагмент кода, чтобы создать URL-адрес JDBC, который можно передать в интерфейсы API кадров Spark. Код создает `Properties` объект для хранения параметров. Вставьте фрагмент кода в ячейку кода и нажмите клавиши **SHIFT+ВВОД**, чтобы выполнить код.

    ```scala
    import java.util.Properties

    val jdbc_url = s"jdbc:sqlserver://${jdbcHostname}:${jdbcPort};database=${jdbcDatabase};encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=60;"
    val connectionProperties = new Properties()
    connectionProperties.put("user", s"${jdbcUsername}")
    connectionProperties.put("password", s"${jdbcPassword}")
    ```

1. Используйте приведенный ниже фрагмент кода, чтобы создать кадр данных с данными из таблицы в базе данных. В этом фрагменте кода используется `SalesLT.Address` Таблица, доступная как часть базы данных **AdventureWorksLT** . Вставьте фрагмент кода в ячейку кода и нажмите клавиши **SHIFT+ВВОД**, чтобы выполнить код.

    ```scala
    val sqlTableDF = spark.read.jdbc(jdbc_url, "SalesLT.Address", connectionProperties)
    ```

1. Теперь вы можете выполнять операции с таблицей данных, например получить схему:

    ```scala
    sqlTableDF.printSchema
    ```

    Вы увидите результат, аналогичный следующему изображению:

    ![выходные данные схемы](./media/apache-spark-connect-to-sql-database/read-from-sql-schema-output.png "выходные данные схемы")

1. Вы также можете выполнять такие операции, как получение первых 10 строк.

    ```scala
    sqlTableDF.show(10)
    ```

1. Можно также извлечь определенные столбцы из набора данных.

    ```scala
    sqlTableDF.select("AddressLine1", "City").show(10)
    ```

## <a name="write-data-into-azure-sql-database"></a>Запись данных в базу данных SQL Azure

В этом разделе мы используем пример CSV-файла, доступного в кластере, чтобы создать таблицу в базе данных и заполнить ее данными. Пример CSV-файла (**HVAC.csv**) содержится во всех кластерах HDInsight в папке `HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv`.

1. В новом Jupyter Notebook в ячейке кода вставьте приведенный ниже фрагмент и замените значения заполнителей значениями для базы данных.

    ```scala
    // Declare the values for your database

    val jdbcUsername = "<SQL DB ADMIN USER>"
    val jdbcPassword = "<SQL DB ADMIN PWD>"
    val jdbcHostname = "<SQL SERVER NAME HOSTING SDL DB>" //typically, this is in the form or servername.database.windows.net
    val jdbcPort = 1433
    val jdbcDatabase ="<AZURE SQL DB NAME>"
    ```

    Чтобы выполнить ячейку кода, нажмите клавиши **SHIFT+ВВОД**.  

1. В следующем фрагменте кода создается URL-адрес JDBC, который можно передать в интерфейсы API "кадры данных Spark". Код создает `Properties` объект для хранения параметров. Вставьте фрагмент кода в ячейку кода и нажмите клавиши **SHIFT+ВВОД**, чтобы выполнить код.

    ```scala
    import java.util.Properties

    val jdbc_url = s"jdbc:sqlserver://${jdbcHostname}:${jdbcPort};database=${jdbcDatabase};encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=60;"
    val connectionProperties = new Properties()
    connectionProperties.put("user", s"${jdbcUsername}")
    connectionProperties.put("password", s"${jdbcPassword}")
    ```

1. В приведенном ниже фрагменте кода извлекается схема данных в HVAC.csv и с ее помощью данные загружаются из CSV-файла в кадр данных `readDf`. Вставьте фрагмент кода в ячейку кода и нажмите клавиши **SHIFT+ВВОД**, чтобы выполнить код.

    ```scala
    val userSchema = spark.read.option("header", "true").csv("wasbs:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv").schema
    val readDf = spark.read.format("csv").schema(userSchema).load("wasbs:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")
    ```

1. Используя кадр данных `readDf`, создайте временную таблицу `temphvactable`. Затем с ее помощью создайте таблицу hive `hvactable_hive`.

    ```scala
    readDf.createOrReplaceTempView("temphvactable")
    spark.sql("create table hvactable_hive as select * from temphvactable")
    ```

1. Наконец, используйте таблицу Hive для создания таблицы в базе данных. Следующий фрагмент кода создается `hvactable` в базе данных SQL Azure.

    ```scala
    spark.table("hvactable_hive").write.jdbc(jdbc_url, "hvactable", connectionProperties)
    ```

1. Подключитесь к базе данных SQL Azure с помощью SSMS и убедитесь, что `dbo.hvactable` там отображается.

    а. Запустите SSMS и подключитесь к базе данных SQL Azure, предоставив сведения о подключении, как показано на снимке экрана ниже.

    ![Подключение к базе данных SQL с помощью SSMS1](./media/apache-spark-connect-to-sql-database/connect-to-sql-db-ssms.png "Подключение к базе данных SQL с помощью SSMS1")

    b. В **обозревателе объектов** разверните базу данных и узел таблицы, чтобы увидеть созданную таблицу **dbo. таблицу hvactable** .

    ![Подключение к базе данных SQL с помощью SSMS2](./media/apache-spark-connect-to-sql-database/connect-to-sql-db-ssms-locate-table.png "Подключение к базе данных SQL с помощью SSMS2")

1. Выполните запрос в SSMS для просмотра всех столбцов в таблице.

    ```sql
    SELECT * from hvactable
    ```

## <a name="stream-data-into-azure-sql-database"></a>Потоковая передача данных в базу данных SQL Azure

В этом разделе мы будем передавать данные в объект `hvactable` , созданный в предыдущем разделе.

1. В качестве первого шага убедитесь, что в нет записей `hvactable` . С помощью SSMS выполните следующий запрос к таблице.

    ```sql
    TRUNCATE TABLE [dbo].[hvactable]
    ```

1. Создайте новый Jupyter Notebook в кластере HDInsight Spark. Вставьте следующий фрагмент кода в ячейку кода и нажмите клавиши **SHIFT+ВВОД**.

    ```scala
    import org.apache.spark.sql._
    import org.apache.spark.sql.types._
    import org.apache.spark.sql.functions._
    import org.apache.spark.sql.streaming._
    import java.sql.{Connection,DriverManager,ResultSet}
    ```

1. Потоковая передача данных из **HVAC.csv** в `hvactable` . Файл HVAC.csv доступен в кластере по адресу `/HdiSamples/HdiSamples/SensorSampleData/HVAC/` . В следующем фрагменте кода мы сначала получаем схему данных для потоковой передачи. Затем создаем кадр данных потоковой передачи с помощью этой схемы. Вставьте фрагмент кода в ячейку кода и нажмите клавиши **SHIFT+ВВОД**, чтобы выполнить код.

    ```scala
    val userSchema = spark.read.option("header", "true").csv("wasbs:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv").schema
    val readStreamDf = spark.readStream.schema(userSchema).csv("wasbs:///HdiSamples/HdiSamples/SensorSampleData/hvac/") 
    readStreamDf.printSchema
    ```

1. Выходные данные будут содержать схему файла **HVAC.csv**. Также `hvactable` имеет ту же схему. В выходных данных перечислены столбцы в таблице.

    !["Таблица схемы hdinsight Apache Spark"](./media/apache-spark-connect-to-sql-database/hdinsight-schema-table.png "Схема таблицы")

1. Наконец, используйте следующий фрагмент кода для чтения данных из HVAC.csv и потоковой передачи в в `hvactable` базе данных. Вставьте фрагмент в ячейку кода, замените значения заполнителей значениями для базы данных, а затем нажмите клавиши **SHIFT + ВВОД** для запуска.

    ```scala
    val WriteToSQLQuery  = readStreamDf.writeStream.foreach(new ForeachWriter[Row] {
        var connection:java.sql.Connection = _
        var statement:java.sql.Statement = _

        val jdbcUsername = "<SQL DB ADMIN USER>"
        val jdbcPassword = "<SQL DB ADMIN PWD>"
        val jdbcHostname = "<SQL SERVER NAME HOSTING SDL DB>" //typically, this is in the form or servername.database.windows.net
        val jdbcPort = 1433
        val jdbcDatabase ="<AZURE SQL DB NAME>"
        val driver = "com.microsoft.sqlserver.jdbc.SQLServerDriver"
        val jdbc_url = s"jdbc:sqlserver://${jdbcHostname}:${jdbcPort};database=${jdbcDatabase};encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=30;"

        def open(partitionId: Long, version: Long):Boolean = {
        Class.forName(driver)
        connection = DriverManager.getConnection(jdbc_url, jdbcUsername, jdbcPassword)
        statement = connection.createStatement
        true
        }

        def process(value: Row): Unit = {
        val Date  = value(0)
        val Time = value(1)
        val TargetTemp = value(2)
        val ActualTemp = value(3)
        val System = value(4)
        val SystemAge = value(5)
        val BuildingID = value(6)  

        val valueStr = "'" + Date + "'," + "'" + Time + "'," + "'" + TargetTemp + "'," + "'" + ActualTemp + "'," + "'" + System + "'," + "'" + SystemAge + "'," + "'" + BuildingID + "'"
        statement.execute("INSERT INTO " + "dbo.hvactable" + " VALUES (" + valueStr + ")")
        }

        def close(errorOrNull: Throwable): Unit = {
        connection.close
        }
        })

    var streamingQuery = WriteToSQLQuery.start()
    ```

1. Убедитесь, что данные передаются в поток, `hvactable` выполнив следующий запрос в SQL Server Management Studio (SSMS). При каждом выполнении запроса отображется большее количество строк в таблице.

    ```sql
    SELECT COUNT(*) FROM hvactable
    ```

## <a name="next-steps"></a>Дальнейшие действия

* [Использование кластера HDInsight Spark для анализа данных в Data Lake Storage](apache-spark-use-with-data-lake-store.md)
* [Загрузка данных и выполнение запросов в кластере Apache Spark в Azure HDInsight](apache-spark-load-data-run-query.md)
* [Использование структурированной потоковой передачи Apache Spark с Apache Kafka в HDInsight](../hdinsight-apache-kafka-spark-structured-streaming.md)
