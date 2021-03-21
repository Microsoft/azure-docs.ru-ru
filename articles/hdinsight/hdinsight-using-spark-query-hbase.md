---
title: Чтение и запись данных HBase с помощью Spark в Azure HDInsight
description: Чтение и запись данных из кластера Spark в кластер HBase с помощью соединителя Spark HBase.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 08/12/2020
ms.openlocfilehash: 344caf4080380f5d9dfdaf452798ada6d1dc9f1c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98931224"
---
# <a name="use-apache-spark-to-read-and-write-apache-hbase-data"></a>Чтение и запись данных Apache HBase с помощью Apache Spark

Обычно для запроса Apache HBase применяется низкоуровневый API (сканирует, получает и помещает) или синтаксис SQL, использующий Apache Phoenix. Apache также предоставляет соединитель Apache Spark HBase. Соединитель — это удобная и эффективная альтернатива для запроса и изменения данных, хранящихся в HBase.

## <a name="prerequisites"></a>Предварительные условия

* Два отдельных кластера HDInsight развернуты в одной [виртуальной сети](./hdinsight-plan-virtual-network-deployment.md). Одна HBase и одна Spark с установленным как минимум Spark 2,1 (HDInsight 3,6). Дополнительные сведения см. в статье [Создание кластеров под управлением Linux в HDInsight с помощью портала Azure](hdinsight-hadoop-create-linux-clusters-portal.md).

* Схема универсального кода ресурса (URI) для основного хранилища кластеров. Эта схема будет wasb://для хранилища BLOB-объектов Azure, `abfs://` для Azure Data Lake Storage 2-го поколения или ADL://для Azure Data Lake Storage 1-го поколения. Если для хранилища BLOB-объектов включено безопасное перемещение, URI будет иметь значение `wasbs://` .  См. также сведения о [безопасной передаче](../storage/common/storage-require-secure-transfer.md).

## <a name="overall-process"></a>Общий процесс

Высокоуровневый процесс включения кластера Spark для запроса к кластеру HBase выглядит следующим образом:

1. Подготовка демонстрационных данных в HBase.
2. Получите файл hbase-site.xml из папки конфигурации кластера HBase (/ЕТК/хбасе/конф) и поместите копию hbase-site.xml в папку конфигурации Spark 2 (/etc/spark2/conf). (Необязательно. используйте скрипт, предоставленный группой HDInsight для автоматизации этого процесса)
4. Запуск `spark-shell` со ссылкой на соединитель Spark HBase по его координатам Maven в параметре `packages`.
5. Определение каталога, который сопоставляет схему из Spark с HBase.
6. Взаимодействие с данными HBase с помощью API RDD или таблицы данных.

## <a name="prepare-sample-data-in-apache-hbase"></a>Подготовка демонстрационных данных в Apache HBase

На этом шаге вы создадите и заполните таблицу в Apache HBase, которую затем можно будет запрашивать с помощью Spark.

1. Используйте `ssh` команду для подключения к кластеру HBase. Измените приведенную ниже команду, заменив `HBASECLUSTER` именем кластера HBase, а затем введите следующую команду:

    ```cmd
    ssh sshuser@HBASECLUSTER-ssh.azurehdinsight.net
    ```

2. Используйте `hbase shell` команду, чтобы запустить интерактивную оболочку HBase. В строку SSH-подключения введите следующую команду:

    ```bash
    hbase shell
    ```

3. Используйте `create` команду, чтобы создать таблицу HBase с семейством из двух столбцов. Введите следующую команду:

    ```hbase
    create 'Contacts', 'Personal', 'Office'
    ```

4. Используйте `put` команду, чтобы вставить значения из указанного столбца в указанную строку в определенной таблице. Введите следующую команду:

    ```hbase
    put 'Contacts', '1000', 'Personal:Name', 'John Dole'
    put 'Contacts', '1000', 'Personal:Phone', '1-425-000-0001'
    put 'Contacts', '1000', 'Office:Phone', '1-425-000-0002'
    put 'Contacts', '1000', 'Office:Address', '1111 San Gabriel Dr.'
    put 'Contacts', '8396', 'Personal:Name', 'Calvin Raji'
    put 'Contacts', '8396', 'Personal:Phone', '230-555-0191'
    put 'Contacts', '8396', 'Office:Phone', '230-555-0191'
    put 'Contacts', '8396', 'Office:Address', '5415 San Gabriel Dr.'
    ```

5. Используйте `exit` команду, чтобы прерывать интерактивную оболочку HBase. Введите следующую команду:

    ```hbase
    exit
    ```
    
## <a name="run-scripts-to-set-up-connection-between-clusters"></a>Запуск сценариев для настройки подключения между кластерами

Чтобы настроить связь между кластерами, выполните следующие действия, чтобы запустить два скрипта в кластерах. Эти сценарии автоматизируют процесс копирования файлов, описанный в разделе "Настройка связи вручную" ниже. 

* Скрипт, запускаемый из кластера HBase, будет отправлять `hbase-site.xml` сведения о сопоставлении IP-адресов HBase в хранилище по умолчанию, подключенное к кластеру Spark. 
* Скрипт, запускаемый из кластера Spark, настраивает два задания cron для периодического запуска двух вспомогательных скриптов:  
    1.  Задание HBase cron — скачивание новых `hbase-site.xml` файлов и сопоставление IP-адресов HBase из учетной записи хранения Spark по умолчанию на локальный узел
    2.  Задание Spark cron — проверяет, выполнялось ли масштабирование Spark, а кластер является безопасным. Если это так, измените, `/etc/hosts` чтобы включить IP-сопоставление HBase, сохраненное локально

__Примечание__. прежде чем продолжать, убедитесь, что вы добавили учетную запись хранения кластера Spark в кластер HBase в качестве учетной записи дополнительного хранилища. Убедитесь, что скрипты заданы в порядке, как показано ниже.


1. Используйте [действие сценария](hdinsight-hadoop-customize-cluster-linux.md#script-action-to-a-running-cluster) в кластере HBase, чтобы применить изменения со следующими соображениями. 


    |Свойство. | Значение |
    |---|---|
    |URI bash-скрипта|`https://hdiconfigactions.blob.core.windows.net/hbasesparkconnectorscript/connector-hbase.sh`|
    |Типы узлов|Регион|
    |Параметры|`-s SECONDARYS_STORAGE_URL`|
    |Persisted|да|

    * `SECONDARYS_STORAGE_URL` URL-адрес хранилища по умолчанию на стороне Spark. Пример параметра: `-s wasb://sparkcon-2020-08-03t18-17-37-853z@sparkconhdistorage.blob.core.windows.net`


2.  Используйте действие сценария в кластере Spark, чтобы применить изменения со следующими соображениями.

    |Свойство. | Значение |
    |---|---|
    |URI bash-скрипта|`https://hdiconfigactions.blob.core.windows.net/hbasesparkconnectorscript/connector-spark.sh`|
    |Типы узлов|Головной, Рабочий, Zookeeper|
    |Параметры|`-s "SPARK-CRON-SCHEDULE"` (необязательно) `-h "HBASE-CRON-SCHEDULE"` используемых|
    |Persisted|да|


    * Можно указать, как часто этот кластер должен автоматически проверять наличие обновлений. Default:-s "*/1 * * * *"-h 0 (в этом примере Spark cron выполняется каждую минуту, а cron-сервер HBase не запускается)
    * Поскольку HBase cron не настроен по умолчанию, необходимо повторно выполнить этот сценарий при масштабировании в кластер HBase. Если кластер HBase часто масштабируется, вы можете настроить задание HBase cron автоматически. Например: `-h "*/30 * * * *"` настраивает скрипт для выполнения проверок каждые 30 минут. Это приведет к периодической повторному запуску расписания HBase cron для автоматической загрузки новых сведений HBase в учетную запись хранения на локальный узел.
    
    

## <a name="set-up-communication-manually-optional-if-provided-script-in-above-step-fails"></a>Настройка связи вручную (необязательно, если предоставленный скрипт на предыдущем шаге завершается ошибкой)

__Примечание.__ Эти действия должны выполняться каждый раз, когда один из кластеров проходит операцию масштабирования.

1. Скопируйте hbase-site.xml из локального хранилища в корневую папку хранилища по умолчанию кластера Spark.  Измените приведенную ниже команду, чтобы отразить конфигурацию.  Затем из открытого сеанса SSH в кластер HBase введите команду:

    | Значение синтаксиса | Новое значение|
    |---|---|
    |[Схема URI](hdinsight-hadoop-linux-information.md#URI-and-scheme) | Измените в соответствии с хранилищем.  Приведенный ниже синтаксис предназначен для хранилища BLOB-объектов с включенной безопасной переносю.|
    |`SPARK_STORAGE_CONTAINER`|Замените на имя контейнера хранилища по умолчанию, используемое для кластера Spark.|
    |`SPARK_STORAGE_ACCOUNT`|Замените именем учетной записи хранения по умолчанию, используемой для кластера Spark.|

    ```bash
    hdfs dfs -copyFromLocal /etc/hbase/conf/hbase-site.xml wasbs://SPARK_STORAGE_CONTAINER@SPARK_STORAGE_ACCOUNT.blob.core.windows.net/
    ```

2. Затем завершите подключение SSH к кластеру HBase.

    ```bash
    exit
    ```


3. Подключитесь к головному узлу кластера Spark с помощью SSH. Измените приведенную ниже команду, заменив `SPARKCLUSTER` именем кластера Spark, а затем введите следующую команду:

    ```cmd
    ssh sshuser@SPARKCLUSTER-ssh.azurehdinsight.net
    ```

4. Введите следующую команду, чтобы скопировать данные `hbase-site.xml` из хранилища по умолчанию кластера Spark в папку конфигурации Spark 2 в локальном хранилище кластера:

    ```bash
    sudo hdfs dfs -copyToLocal /hbase-site.xml /etc/spark2/conf
    ```

## <a name="run-spark-shell-referencing-the-spark-hbase-connector"></a>Запуск оболочки Shell со ссылкой на соединитель Spark HBase

После выполнения предыдущего шага вы сможете запустить оболочку Spark, указав соответствующую версию соединителя Spark HBase. Чтобы найти последнюю подходящую версию ядра для соединителя Spark HBase для вашего кластера, см. раздел [репозиторий ШК Core](https://repo.hortonworks.com/content/groups/public/com/hortonworks/shc/shc-core/).

Например, в следующей таблице перечислены две версии и соответствующие команды, которые использует команда HDInsight. Вы можете использовать те же версии для кластеров, если версии HBase и Spark совпадают с указанными в таблице. 


1. В открытом сеансе SSH в кластере Spark введите следующую команду, чтобы запустить оболочку Spark:

    |Версия Spark| Версия HDI HBase  | Версия шк    |  Команда  |
    | :-----------:| :----------: | :-----------: |:----------- |
    |      2.1    | HDI 3,6 (HBase 1,1) | 1.1.1-2.1-s_2.11    | `spark-shell --packages com.hortonworks:shc-core:1.1.1-2.1-s_2.11 --repositories https://repo.hortonworks.com/content/groups/public/` |
    |      2.4    | HDI 4,0 (HBase 2,0) | 1.1.0.3.1.2.2-1  | `spark-shell --packages com.hortonworks.shc:shc-core:1.1.0.3.1.2.2-1 --repositories http://repo.hortonworks.com/content/groups/public/` |

2. Не закрывайте этот экземпляр оболочки Spark и продолжайте [определять каталог и запрос](#define-a-catalog-and-query). Если вы не нашли JAR, соответствующие вашим версиям в репозитории ШК Core, продолжите чтение. 

Вы можете создать JAR непосредственно из ветви GitHub [Spark-HBase-Connector](https://github.com/hortonworks-spark/shc) . Например, если вы используете Spark 2,3 и HBase 1,1, выполните следующие действия:

1. Клонируйте репозиторий:

    ```bash
    git clone https://github.com/hortonworks-spark/shc
    ```
    
2. Перейдите к ветви-2,3:

    ```bash
    git checkout branch-2.3
    ```

3. Выполните сборку из ветви (создает JAR-файл):

    ```bash
    mvn clean package -DskipTests
    ```
    
3. Выполните следующую команду (обязательно измените имя JAR-файла, соответствующее созданному JAR-файлу):

    ```bash
    spark-shell --jars <path to your jar>,/usr/hdp/current/hbase-client/lib/htrace-core-3.1.0-incubating.jar,/usr/hdp/current/hbase-client/lib/hbase-client.jar,/usr/hdp/current/hbase-client/lib/hbase-common.jar,/usr/hdp/current/hbase-client/lib/hbase-server.jar,/usr/hdp/current/hbase-client/lib/hbase-protocol.jar,/usr/hdp/current/hbase-client/lib/htrace-core-3.1.0-incubating.jar
    ```
    
4. Не закрывайте этот экземпляр оболочки Spark и переходите к следующему разделу. 



## <a name="define-a-catalog-and-query"></a>Определение каталога и запроса

На этом этапе вы определяете объект каталога, который сопоставляет схему из Spark с Apache HBase.  

1. В открытой оболочке Spark введите следующие `import` инструкции:

    ```scala
    import org.apache.spark.sql.{SQLContext, _}
    import org.apache.spark.sql.execution.datasources.hbase._
    import org.apache.spark.{SparkConf, SparkContext}
    import spark.sqlContext.implicits._
    ```  

1. Введите следующую команду, чтобы определить каталог для таблицы Contacts, созданной в HBase:

    ```scala
    def catalog = s"""{
        |"table":{"namespace":"default", "name":"Contacts"},
        |"rowkey":"key",
        |"columns":{
        |"rowkey":{"cf":"rowkey", "col":"key", "type":"string"},
        |"officeAddress":{"cf":"Office", "col":"Address", "type":"string"},
        |"officePhone":{"cf":"Office", "col":"Phone", "type":"string"},
        |"personalName":{"cf":"Personal", "col":"Name", "type":"string"},
        |"personalPhone":{"cf":"Personal", "col":"Phone", "type":"string"}
        |}
    |}""".stripMargin
    ```

    Код.  

    1. Определяет схему каталога для таблицы HBase с именем `Contacts` .  
    1. Определяет RowKey как `key` и сопоставляет имена столбцов, используемых в Spark, с семейством столбцов, именем столбца и типом столбца, используемым в HBase.  
    1. Подробно определяет RowKey как именованный столбец ( `rowkey` ), имеющий определенное семейство столбцов `cf` `rowkey` .  

1. Введите следующую команду, чтобы определить метод, который предоставляет кадр данных для `Contacts` таблицы в HBase:

    ```scala
    def withCatalog(cat: String): DataFrame = {
        spark.sqlContext
        .read
        .options(Map(HBaseTableCatalog.tableCatalog->cat))
        .format("org.apache.spark.sql.execution.datasources.hbase")
        .load()
     }
    ```

1. Создайте экземпляр таблицы данных:

    ```scala
    val df = withCatalog(catalog)
    ```  

1. Выполните запрос таблицы данных:

    ```scala
    df.show()
    ```

    Вы должны увидеть две строки данных:

    ```output
    +------+--------------------+--------------+-------------+--------------+
    |rowkey|       officeAddress|   officePhone| personalName| personalPhone|
    +------+--------------------+--------------+-------------+--------------+
    |  1000|1111 San Gabriel Dr.|1-425-000-0002|    John Dole|1-425-000-0001|
    |  8396|5415 San Gabriel Dr.|  230-555-0191|  Calvin Raji|  230-555-0191|
    +------+--------------------+--------------+-------------+--------------+
    ```

1. Зарегистрируйте временную таблицу, чтобы запрашивать таблицу HBase с помощью Spark SQL:

    ```scala
    df.createTempView("contacts")
    ```

1. Выполните SQL-запрос к таблице `contacts`:

    ```scala
    spark.sqlContext.sql("select personalName, officeAddress from contacts").show
    ```

    Вы должны увидеть примерно такой результат:

    ```output
    +-------------+--------------------+
    | personalName|       officeAddress|
    +-------------+--------------------+
    |    John Dole|1111 San Gabriel Dr.|
    |  Calvin Raji|5415 San Gabriel Dr.|
    +-------------+--------------------+
    ```

## <a name="insert-new-data"></a>Вставка новых данных

1. Чтобы вставить новую запись о контакте, определите класс `ContactRecord`:

    ```scala
    case class ContactRecord(
        rowkey: String,
        officeAddress: String,
        officePhone: String,
        personalName: String,
        personalPhone: String
        )
    ```

1. Создайте экземпляр `ContactRecord` и поместите его в массив:

    ```scala
    val newContact = ContactRecord("16891", "40 Ellis St.", "674-555-0110", "John Jackson","230-555-0194")

    var newData = new Array[ContactRecord](1)
    newData(0) = newContact
    ```

1. Сохраните массив новых данных в HBase:

    ```scala
    sc.parallelize(newData).toDF.write.options(Map(HBaseTableCatalog.tableCatalog -> catalog, HBaseTableCatalog.newTable -> "5")).format("org.apache.spark.sql.execution.datasources.hbase").save()
    ```

1. Изучите результаты:

    ```scala  
    df.show()
    ```

    Вы должны увидеть примерно такой результат:

    ```output
    +------+--------------------+--------------+------------+--------------+
    |rowkey|       officeAddress|   officePhone|personalName| personalPhone|
    +------+--------------------+--------------+------------+--------------+
    |  1000|1111 San Gabriel Dr.|1-425-000-0002|   John Dole|1-425-000-0001|
    | 16891|        40 Ellis St.|  674-555-0110|John Jackson|  230-555-0194|
    |  8396|5415 San Gabriel Dr.|  230-555-0191| Calvin Raji|  230-555-0191|
    +------+--------------------+--------------+------------+--------------+
    ```

1. Закройте оболочку Spark, введя следующую команду:

    ```scala
    :q
    ```

## <a name="next-steps"></a>Дальнейшие действия

* [Соединитель Apache Spark HBase](https://github.com/hortonworks-spark/shc)
