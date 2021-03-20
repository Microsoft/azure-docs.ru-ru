---
title: Массовая загрузка данных в Apache Phoenix с помощью psql в Azure HDInsight
description: Использование средства psql для загрузки данных с массовыми загрузками в Apache Phoenix таблицы в Azure HDInsight
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/17/2019
ms.openlocfilehash: d56c9b33d53be0e30769aef8934a2dbf43d06867
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98944836"
---
# <a name="bulk-load-data-into-apache-phoenix-using-psql"></a>Массовая загрузка данных в Apache Phoenix с помощью psql

[Apache Phoenix](https://phoenix.apache.org/) — это реляционная база данных на основе [Apache HBase](../hbase/apache-hbase-overview.md) с открытым кодом и высоким уровнем параллелизма. Phoenix поддерживает запросы к HBase в стиле SQL. Phoenix использует драйверы JDBC. Это позволяет создавать, удалять и изменять SQL-таблицы, индексы, представления и последовательности, а также вставлять строки по одной или в пакетном режиме. Phoenix использует для сборки запросов компиляцию в машинный код noSQL, а не MapReduce, что позволяет создавать на основе HBase приложения с низким уровнем задержки. Кроме того, Phoenix поддерживает сопроцессоры для выполнения пользовательского кода в адресном пространстве сервера, то есть прямо в месте размещения данных. Это сводит к минимуму трафик между клиентом и сервером.  Чтобы применить Phoenix для работы с данными в HDInsight, сначала создайте таблицы и загрузите в них данные.

## <a name="bulk-loading-with-apache-phoenix"></a>Массовая загрузка с помощью Apache Phoenix

Есть много способов загрузить данные в HBase, например клиентские интерфейсы API, задание MapReduce с TableOutputFormat или ввод данных вручную через оболочку HBase. Phoenix предлагает два метода загрузки данных в формате CSV в таблицы Phoenix: клиентское средство загрузки `psql` и средство массовой загрузки на основе MapReduce.

Средство `psql` является однопоточным и хорошо подходит для загрузки нескольких мегабайт или гигабайт данных. Все загружаемые CSV-файлы должны иметь расширение .csv.  Также вы можете указать в командной строке `psql` SQL-файлы скриптов с расширением .sql.

Массовая загрузка с помощью MapReduce применяется для данных существенно большего объема, в том числе в промышленных средах, поскольку в MapReduce используется многопоточная схема работы.

Прежде чем загружать данные, убедитесь, что Phoenix включен и для запросов правильно настроены параметры времени ожидания.  Откройте панель мониторинга [Apache Ambari](https://ambari.apache.org/) для кластера HDInsight, выберите HBase и перейдите на вкладку Конфигурация.  Прокрутите вниз, чтобы убедиться, что Apache Phoenix имеет значение `enabled` , как показано ниже.

![Параметры кластера HDInsight для Apache Phoenix](./media/apache-hbase-phoenix-psql/apache-ambari-phoenix.png)

### <a name="use-psql-to-bulk-load-tables"></a>Использование `psql` для массовой загрузки таблиц

1. Создайте файл с именем `createCustomersTable.sql` и скопируйте приведенный ниже код в файл. Сохраните и закройте файл.

    ```sql
    CREATE TABLE Customers (
        ID varchar NOT NULL PRIMARY KEY,
        Name varchar,
        Income decimal,
        Age INTEGER,
        Country varchar);
    ```

1. Создайте файл с именем `listCustomers.sql` и скопируйте приведенный ниже код в файл. Сохраните и закройте файл.

    ```sql
    SELECT * from Customers;
    ```

1. Создайте файл с именем `customers.csv` и скопируйте приведенный ниже код в файл. Сохраните и закройте файл.

    ```txt
    1,Samantha,260000.0,18,US
    2,Sam,10000.5,56,US
    3,Anton,550150.0,42,Norway
    ```

1. Создайте файл с именем `customers2.csv` и скопируйте приведенный ниже код в файл. Сохраните и закройте файл.

    ```txt
    4,Nicolle,180000.0,22,US
    5,Kate,210000.5,24,Canada
    6,Ben,45000.0,32,Poland
    ```

1. Откройте командную строку и перейдите в каталог, где находятся только что созданные файлы. Замените ИМЯ_КЛАСТЕРА, ниже, фактическим именем кластера HBase. Затем выполните код, чтобы передать файлы в головного узла кластера:

    ```cmd
    scp customers.csv customers2.csv createCustomersTable.sql listCustomers.sql sshuser@CLUSTERNAME-ssh.azurehdinsight.net:/tmp
    ```

1. С помощью команды [ssh command](../hdinsight-hadoop-linux-use-ssh-unix.md) подключитесь к кластеру. Измените приведенную ниже команду, заменив CLUSTERNAME именем своего кластера, а затем введите команду:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. В сеансе SSH измените каталог на расположение средства **psql** . Выполните следующую команду:

    ```bash
    cd /usr/hdp/current/phoenix-client/bin
    ```

1. Выполнить небольшую загрузку данных. Приведенный ниже код создаст таблицу **Customers** , а затем отправит данные.

    ```bash
    python psql.py /tmp/createCustomersTable.sql /tmp/customers.csv
    ```

    После `psql` завершения операции вы увидите примерно следующее сообщение:

    ```output
    csv columns from database.
    CSV Upsert complete. 3 rows upserted
    Time: 0.081 sec(s)
    ```

1. Можно продолжить использовать `psql` для просмотра содержимого таблицы Customers. Выполните приведенный ниже код.

    ```bash
    python psql.py /tmp/listCustomers.sql
    ```

    Кроме того, для запроса данных можно использовать [оболочку HBase](./query-hbase-with-hbase-shell.md)или [Apache Zeppelin](./apache-hbase-phoenix-zeppelin.md) .

1. Отправьте дополнительные данные. Теперь, когда таблица уже существует, команда указывает таблицу. Выполните следующую команду:

    ```bash
    python psql.py -t CUSTOMERS /tmp/customers2.csv
    ```

## <a name="use-mapreduce-to-bulk-load-tables"></a>Использование MapReduce для массовой загрузки таблиц

Чтобы повысить пропускную способность и распределить нагрузку в кластере, используйте средство загрузки MapReduce. Этот загрузчик преобразует все данные в формат HFiles и передает созданные файлы HFiles в HBase.

1. Этот раздел продолжится с помощью сеанса SSH и созданных ранее объектов. При необходимости создайте таблицу **Customers** и файл **customers.csv** , выполнив действия, описанные выше. При необходимости восстановите SSH-подключение.

1. Усечение содержимого таблицы **Customers** . В открытом сеансе SSH выполните приведенные ниже команды.

    ```bash
    hbase shell
    truncate 'CUSTOMERS'
    exit
    ```

1. Скопируйте `customers.csv` файл из головного узла в службу хранилища Azure.

    ```bash
    hdfs dfs -put /tmp/customers.csv wasbs:///tmp/customers.csv
    ```

1. Откройте каталог выполнения команды массовой загрузки MapReduce:

    ```bash
    cd /usr/hdp/current/phoenix-client
    ```

1. Запустите загрузчик CSV MapReduce с помощью команды `hadoop`, указав клиентский JAR-файл Phoenix:

    ```bash
    HADOOP_CLASSPATH=/usr/hdp/current/hbase-client/lib/hbase-protocol.jar:/etc/hbase/conf hadoop jar phoenix-client.jar org.apache.phoenix.mapreduce.CsvBulkLoadTool --table Customers --input /tmp/customers.csv
    ```

    После завершения отправки вы увидите примерно следующее сообщение:

    ```output
    19/12/18 18:30:57 INFO client.ConnectionManager$HConnectionImplementation: Closing master protocol: MasterService
    19/12/18 18:30:57 INFO client.ConnectionManager$HConnectionImplementation: Closing zookeeper sessionid=0x26f15dcceff02c3
    19/12/18 18:30:57 INFO zookeeper.ZooKeeper: Session: 0x26f15dcceff02c3 closed
    19/12/18 18:30:57 INFO zookeeper.ClientCnxn: EventThread shut down
    19/12/18 18:30:57 INFO mapreduce.AbstractBulkLoadTool: Incremental load complete for table=CUSTOMERS
    19/12/18 18:30:57 INFO mapreduce.AbstractBulkLoadTool: Removing output directory /tmp/50254426-aba6-400e-88eb-8086d3dddb6
    ```

1. Чтобы использовать MapReduce с Azure Data Lake Storage вы должны определить корневой каталог Data Lake Storage (значение `hbase.rootdir` в `hbase-site.xml`). В следующей команде корневой каталог Data Lake Storage имеет значение `adl://hdinsightconf1.azuredatalakestore.net:443/hbase1`. Передайте при помощи этой команды входной и выходной каталоги Data Lake Storage в качестве параметров.

    ```bash
    cd /usr/hdp/current/phoenix-client

    $HADOOP_CLASSPATH=$(hbase mapredcp):/etc/hbase/conf hadoop jar /usr/hdp/2.4.2.0-258/phoenix/phoenix-4.4.0.2.4.2.0-258-client.jar

    org.apache.phoenix.mapreduce.CsvBulkLoadTool --table Customers --input adl://hdinsightconf1.azuredatalakestore.net:443/hbase1/data/hbase/temp/input/customers.csv –zookeeper ZookeeperQuorum:2181:/hbase-unsecure --output  adl://hdinsightconf1.azuredatalakestore.net:443/hbase1/data/hbase/output1
    ```

1. Для запроса и просмотра данных можно использовать **psql** , как описано выше. Можно также использовать [оболочку HBase](./query-hbase-with-hbase-shell.md)или [Apache Zeppelin](./apache-hbase-phoenix-zeppelin.md).

## <a name="recommendations"></a>Рекомендации

* Используйте одну среду хранения для входных и выходных папок Azure Storage (WASB) или Azure Data Lake Storage (ADL). Передать данные из службы хранилища Azure в Data Lake Storage можно с помощью команды `distcp`.

    ```bash
    hadoop distcp wasb://@.blob.core.windows.net/example/data/gutenberg adl://.azuredatalakestore.net:443/myfolder
    ```

* Используйте рабочие узлы большого размера. Процессы сопоставления в операции массового копирования MapReduce создают большие объемы временных выходных данных, которые занимают все доступное место, кроме DFS. Для массовой загрузки большого объема данных используйте значительное количество рабочих узлов большого размера. Число рабочих узлов, выделенных для кластера, прямо влияет на скорость обработки.

* Разделите входные файлы на фрагменты объемом около 10 ГБ. Для массовой загрузки данных используются большие объемы хранилища. Поэтому разделение входных данных на небольшие фрагменты позволит повысить производительность.

* Избегайте перегрузки региональных серверов. Если значение ключа в строках монотонно возрастает, последовательные операции записи в HBase приводят к перегрузке региональных серверов. Добавив *соль* к ключу строк, вы сможете избежать последовательных операций записи. Phoenix позволяет прозрачно добавлять случайные значения к ключу строк, используя байты соли для конкретных таблиц. Этот механизм описан в статье, ссылка на которую приводится ниже.

## <a name="next-steps"></a>Дальнейшие действия

* [Массовая загрузка данных с помощью Apache Phoenix](https://phoenix.apache.org/bulk_dataload.html)
* [Использование Apache Phoenix с кластерами Apache HBase под управлением Linux в HDInsight](../hbase/apache-hbase-query-with-phoenix.md)
* [Таблицы случайных данных](https://phoenix.apache.org/salted.html)
* [Грамматика Apache Phoenix](https://phoenix.apache.org/language/index.html)
