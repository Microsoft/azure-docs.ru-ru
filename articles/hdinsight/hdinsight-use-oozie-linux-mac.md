---
title: Использование рабочих процессов Hadoop Oozie в Azure HDInsight на основе Linux
description: Использование Hadoop Oozie в HDInsight на основе Linux. Узнайте, как определить рабочий процесс и отправить задание для Oozie.
ms.service: hdinsight
ms.topic: how-to
ms.custom: seoapr2020
ms.date: 04/27/2020
ms.openlocfilehash: 00573f01aab5e9aa86befaebb6b514c76a01b67d
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104871781"
---
# <a name="use-apache-oozie-with-apache-hadoop-to-define-and-run-a-workflow-on-linux-based-azure-hdinsight"></a>Использование Apache Oozie с Apache Hadoop для определения и запуска рабочих процессов в Azure HDInsight под управлением Linux

Узнайте, как использовать Apache Oozie с Apache Hadoop в Azure HDInsight. Oozie — это система рабочих процессов и координации, управляющая заданиями Hadoop. Служба Oozie интегрирована со стеком Hadoop и поддерживает следующие задания:

* Apache Hadoop MapReduce
* Apache Pig
* Apache Hive
* Apache Sqoop

Вы также можете использовать Oozie для планирования системных заданий, например Java-программ и сценариев оболочки.

> [!NOTE]  
> Еще один способ определения рабочих процессов с помощью HDInsight — использование фабрики данных Azure. Дополнительные сведения о фабрике данных см. в статье [Использование данных Apache Pig and Apache Hive в фабрике данных Azure](../data-factory/transform-data.md). Сведения об использовании Oozie в кластерах с Корпоративным пакетом безопасности см. в статье [Запуск Apache Oozie в кластерах Hadoop HDInsight с Корпоративным пакетом безопасности](domain-joined/hdinsight-use-oozie-domain-joined-clusters.md).

## <a name="prerequisites"></a>Предварительные требования

* **Кластер Hadoop в HDInsight**. Ознакомьтесь со статьей [Краткое руководство. Использование Apache Hadoop и Apache Hive в Azure HDInsight с шаблоном Resource Manager](hadoop/apache-hadoop-linux-tutorial-get-started.md).

* **Клиент SSH**. См. раздел [Подключение к HDInsight (Apache Hadoop) с помощью SSH](hdinsight-hadoop-linux-use-ssh-unix.md).

* **База данных SQL Azure**.  См. раздел [Создание базы данных в базе данных SQL Azure в портал Azure](../azure-sql/database/single-database-create-quickstart.md).  В этой статье используется база данных с именем **узиетест**.

* Схема универсального кода ресурса (URI) для основного хранилища кластеров. `wasb://` для службы хранилища Azure `abfs://` для Azure Data Lake Storage 2-го поколения или `adl://` для Azure Data Lake Storage 1-го поколения. Если для службы хранилища Azure включено безопасное перемещение, URI будет таким: `wasbs://`. См. также сведения о [безопасной передаче](../storage/common/storage-require-secure-transfer.md).

## <a name="example-workflow"></a>Пример рабочего процесса

Рабочий процесс, используемый в этой статье, включает два действия. Действия являются определениями задач, таких как запуск Hive, Sqoop, MapReduce или других процессов:

:::image type="content" source="./media/hdinsight-use-oozie-linux-mac/oozie-workflow-diagram.png" alt-text="Схема рабочего процесса HDInsight Oozie" border="false":::

1. Действие Hive запускает скрипт HiveQL для извлечения записей из `hivesampletable` , которые входят в состав HDInsight. Каждая строка данных описывает посещение с определенного мобильного устройства. Формат записи таков:

    ```output
    8       18:54:20        en-US   Android Samsung SCH-i500        California     United States    13.9204007      0       0
    23      19:19:44        en-US   Android HTC     Incredible      Pennsylvania   United States    NULL    0       0
    23      19:19:46        en-US   Android HTC     Incredible      Pennsylvania   United States    1.4757422       0       1
    ```

    Сценарий Hive, используемый в этом документе, подсчитывает общее количество посещений для каждой платформы (например, Android или iPhone) и сохраняет результаты в новой таблице Hive.

    Дополнительные сведения о Hive см. в статье [использование Apache Hive с HDInsight] [hdinsight-use-Hive].

2. Действие Sqoop экспортирует содержимое новой таблицы Hive в таблицу, созданную в базе данных SQL Azure. Дополнительные сведения о Sqoop см. в статье, [посвященной использованию Apache Sqoop в HDInsight](hadoop/apache-hadoop-use-sqoop-mac-linux.md).

> [!NOTE]  
> Сведения о поддерживаемых версиях Oozie в кластерах HDInsight см. в статье [Что представляют собой различные компоненты Hadoop, доступные в HDInsight?](hdinsight-component-versioning.md)

## <a name="create-the-working-directory"></a>Создайте рабочий каталог

В Oozie предполагается, что все ресурсы, необходимые для выполнения задания, должны находиться в том же каталоге. В этом примере используется `wasbs:///tutorials/useoozie`. Чтобы создать каталог, выполните следующие шаги:

1. Измените приведенный ниже код, чтобы заменить `sshuser` именем пользователя SSH для кластера, а замените `CLUSTERNAME` именем кластера.  Затем введите код для подключения к кластеру HDInsight с [помощью SSH](hdinsight-hadoop-linux-use-ssh-unix.md).  

    ```bash
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

2. Чтобы создать каталог, выполните следующую команду:

    ```bash
    hdfs dfs -mkdir -p /tutorials/useoozie/data
    ```

    > [!NOTE]  
    > Параметр `-p` означает, что будут созданы все каталоги для пути. `data`Каталог используется для хранения данных, используемых `useooziewf.hql` скриптом.

3. Измените приведенный ниже код, чтобы заменить `sshuser` его именем пользователя SSH.  Выполните следующую команду, чтобы убедиться, что Oozie может олицетворять учетную запись пользователя:

    ```bash
    sudo adduser sshuser users
    ```

    > [!NOTE]  
    > Ошибки о том, что пользователь уже является членом группы `users`, можно игнорировать.

## <a name="add-a-database-driver"></a>Добавление драйвера базы данных

Этот рабочий процесс использует Sqoop для экспорта данных в базу данных SQL. Поэтому необходимо предоставить копию драйвера JDBC, который используется для взаимодействия с базой данных SQL. Чтобы скопировать драйвер JDBC в рабочую папку, используйте следующую команду из сеанса SSH:

```bash
hdfs dfs -put /usr/share/java/sqljdbc_7.0/enu/mssql-jdbc*.jar /tutorials/useoozie/
```

> [!IMPORTANT]  
> Проверьте фактический драйвер JDBC, существующий по адресу `/usr/share/java/` .

Если рабочий процесс использовал другие ресурсы, например JAR-файл, содержащий приложение MapReduce, необходимо добавить и эти ресурсы.

## <a name="define-the-hive-query"></a>Определение запроса Hive

Для создания сценария языка запросов Hive (HiveQL), определяющего запрос, сделайте следующее. Этот запрос будет использоваться в рабочем процессе Oozie далее в этом документе.

1. В сеансе SSH создайте файл с именем `useooziewf.hql`, выполнив следующую команду:

    ```bash
    nano useooziewf.hql
    ```

1. Открыв редактор GNU nano, используйте следующий запрос в качестве содержимого файла:

    ```hiveql
    DROP TABLE ${hiveTableName};
    CREATE EXTERNAL TABLE ${hiveTableName}(deviceplatform string, count string) ROW FORMAT DELIMITED
    FIELDS TERMINATED BY '\t' STORED AS TEXTFILE LOCATION '${hiveDataFolder}';
    INSERT OVERWRITE TABLE ${hiveTableName} SELECT deviceplatform, COUNT(*) as count FROM hivesampletable GROUP BY deviceplatform;
    ```

    В данном сценарии используются три следующие переменные:

   * `${hiveTableName}`: содержит имя создаваемой таблицы.

   * `${hiveDataFolder}`: содержит расположения файлов данных для таблицы.

     Файл определения рабочего процесса, workflow.xml в этой статье, передает эти значения в этот скрипт HiveQL во время выполнения.

1. Чтобы сохранить файл, нажмите **клавиши CTRL + X**, введите **Y**, а затем нажмите клавишу **Ввод**.  

1. Используйте следующую команду для копирования `useooziewf.hql` в `wasbs:///tutorials/useoozie/useooziewf.hql` :

    ```bash
    hdfs dfs -put useooziewf.hql /tutorials/useoozie/useooziewf.hql
    ```

    Эта команда сохраняет файл `useooziewf.hql` в HDFS-совместимом хранилище кластера.

## <a name="define-the-workflow"></a>Определение рабочего процесса

Определения рабочего процесса Oozie записываются на языке определения процессов Hadoop (hPDL), который является языком определения процессов XML. Для определения рабочего процесса выполните следующие действия.

1. Для создания и открытия файла выполните следующий запрос:

    ```bash
    nano workflow.xml
    ```

2. Открыв редактор nano, введите следующий код XML в качестве содержимого файла:

    ```xml
    <workflow-app name="useooziewf" xmlns="uri:oozie:workflow:0.2">
        <start to = "RunHiveScript"/>
        <action name="RunHiveScript">
        <hive xmlns="uri:oozie:hive-action:0.2">
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
            <property>
                <name>mapred.job.queue.name</name>
                <value>${queueName}</value>
            </property>
            </configuration>
            <script>${hiveScript}</script>
            <param>hiveTableName=${hiveTableName}</param>
            <param>hiveDataFolder=${hiveDataFolder}</param>
        </hive>
        <ok to="RunSqoopExport"/>
        <error to="fail"/>
        </action>
        <action name="RunSqoopExport">
        <sqoop xmlns="uri:oozie:sqoop-action:0.2">
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
            <property>
                <name>mapred.compress.map.output</name>
                <value>true</value>
            </property>
            </configuration>
            <arg>export</arg>
            <arg>--connect</arg>
            <arg>${sqlDatabaseConnectionString}</arg>
            <arg>--table</arg>
            <arg>${sqlDatabaseTableName}</arg>
            <arg>--export-dir</arg>
            <arg>${hiveDataFolder}</arg>
            <arg>-m</arg>
            <arg>1</arg>
            <arg>--input-fields-terminated-by</arg>
            <arg>"\t"</arg>
            <archive>mssql-jdbc-7.0.0.jre8.jar</archive>
            </sqoop>
        <ok to="end"/>
        <error to="fail"/>
        </action>
        <kill name="fail">
        <message>Job failed, error message[${wf:errorMessage(wf:lastErrorNode())}] </message>
        </kill>
        <end name="end"/>
    </workflow-app>
    ```

    В рабочем процессе определены два действия:

   * `RunHiveScript`. Это действие запускает сценарий Hive `useooziewf.hql`.

   * `RunSqoopExport`. Это действие экспортирует созданные данные из сценария Hive в базу данных SQL, используя Sqoop. Это действие будет выполнено только после успешного завершения действия `RunHiveScript`.

     В рабочем процессе есть несколько записей, таких как `${jobTracker}`. Эти записи будут заменены значениями, используемыми в определении задания. Определение задания будет создано далее в этом документе.

     Также обратите внимание на параметр `<archive>mssql-jdbc-7.0.0.jre8.jar</archive>` в разделе Sqoop. Эта запись означает, что этот архив должен стать доступным для Sqoop при выполнении этого действия.

3. Чтобы сохранить файл, нажмите **клавиши CTRL + X**, введите **Y**, а затем нажмите клавишу **Ввод**.  

4. Выполните следующую команду, чтобы скопировать файл `workflow.xml` в `/tutorials/useoozie/workflow.xml`.

    ```bash
    hdfs dfs -put workflow.xml /tutorials/useoozie/workflow.xml
    ```

## <a name="create-a-table"></a>Создание таблицы

> [!NOTE]  
> Существует множество способов подключения к базе данных SQL для создания таблицы. В приведенных ниже действиях используется [FreeTDS](https://www.freetds.org/) из кластера HDInsight.

1. Для установки FreeTDS в кластер HDInsight воспользуйтесь следующей командой:

    ```bash
    sudo apt-get --assume-yes install freetds-dev freetds-bin
    ```

2. Измените приведенный ниже код `<serverName>` , чтобы заменить [ЛОГИЧЕСКИМ именем SQL Server](../azure-sql/database/logical-servers.md) и `<sqlLogin>` именем входа на сервер.  Введите команду для подключения к требуемой базе данных SQL.  Введите пароль в командной строке.

    ```bash
    TDSVER=8.0 tsql -H <serverName>.database.windows.net -U <sqlLogin> -p 1433 -D oozietest
    ```

    Выводятся сведения следующего вида:

    ```output
    locale is "en_US.UTF-8"
    locale charset is "UTF-8"
    using default charset "UTF-8"
    Default database being set to oozietest
    1>
    ```

3. В командной строке `1>` введите следующее:

    ```sql
    CREATE TABLE [dbo].[mobiledata](
    [deviceplatform] [nvarchar](50),
    [count] [bigint])
    GO
    CREATE CLUSTERED INDEX mobiledata_clustered_index on mobiledata(deviceplatform)
    GO
    ```

    Если вводится инструкция `GO`, то оцениваются предыдущие инструкции. Эти инструкции создают таблицу с именем `mobiledata` , которая используется рабочим процессом.

    Используйте следующие команды для проверки создания таблицы:

    ```sql
    SELECT * FROM information_schema.tables
    GO
    ```

    Выводятся сведения следующего вида:

    ```output
    TABLE_CATALOG   TABLE_SCHEMA    TABLE_NAME      TABLE_TYPE
    oozietest       dbo             mobiledata      BASE TABLE
    ```

4. Выйдите из программы TSQL, введя `exit` в `1>` командной строке.

## <a name="create-the-job-definition"></a>Создание определения задания

Определение задания содержит информацию о местонахождении файла workflow.xml, а также о том, где можно найти другие файлы, используемые рабочим процессом, например `useooziewf.hql`. Кроме того, он определяет значения для свойств, используемых в рабочем процессе и связанных файлах.

1. Для получения полного адреса хранилища по умолчанию, воспользуйтесь указанной ниже командой. Этот адрес используется в файле конфигурации, который вы создадите на следующем шаге.

    ```bash
    sed -n '/<name>fs.default/,/<\/value>/p' /etc/hadoop/conf/core-site.xml
    ```

    Эта команда возвращает информацию, как в следующем коде XML:

    ```xml
    <name>fs.defaultFS</name>
    <value>wasbs://mycontainer@mystorageaccount.blob.core.windows.net</value>
    ```

    > [!NOTE]  
    > Если кластер HDInsight использует службу хранилища Azure в качестве хранилища по умолчанию, содержимое элемента `<value>` начинается с `wasbs://`. Если используется Data Lake Storage 1-го поколения, оно начинается с `adl://`. Если используется Data Lake Storage 2-го поколения, оно начинается с `abfs://`.

    Сохраните содержимое элемента `<value>`, которое потребуется нам на следующих шагах.

2. Измените приведенный ниже XML-код следующим образом:

    |Значение заполнителя| Замененное значение|
    |---|---|
    |wasbs://mycontainer \@ mystorageaccount.BLOB.Core.Windows.NET| Значение, полученное на шаге 1.|
    |admin| Имя входа для кластера HDInsight, если не является администратором.|
    |serverName| Имя сервера базы данных SQL Azure.|
    |sqlLogin| Имя входа сервера базы данных SQL Azure.|
    |sqlPassword| Пароль для входа на сервер базы данных SQL Azure.|

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <configuration>

        <property>
        <name>nameNode</name>
        <value>wasbs://mycontainer@mystorageaccount.blob.core.windows.net</value>
        </property>

        <property>
        <name>jobTracker</name>
        <value>headnodehost:8050</value>
        </property>

        <property>
        <name>queueName</name>
        <value>default</value>
        </property>

        <property>
        <name>oozie.use.system.libpath</name>
        <value>true</value>
        </property>

        <property>
        <name>hiveScript</name>
        <value>wasbs://mycontainer@mystorageaccount.blob.core.windows.net/tutorials/useoozie/useooziewf.hql</value>
        </property>

        <property>
        <name>hiveTableName</name>
        <value>mobilecount</value>
        </property>

        <property>
        <name>hiveDataFolder</name>
        <value>wasbs://mycontainer@mystorageaccount.blob.core.windows.net/tutorials/useoozie/data</value>
        </property>

        <property>
        <name>sqlDatabaseConnectionString</name>
        <value>"jdbc:sqlserver://serverName.database.windows.net;user=sqlLogin;password=sqlPassword;database=oozietest"</value>
        </property>

        <property>
        <name>sqlDatabaseTableName</name>
        <value>mobiledata</value>
        </property>

        <property>
        <name>user.name</name>
        <value>admin</value>
        </property>

        <property>
        <name>oozie.wf.application.path</name>
        <value>wasbs://mycontainer@mystorageaccount.blob.core.windows.net/tutorials/useoozie</value>
        </property>
    </configuration>
    ```

    Большая часть информации в этом файле используется для заполнения значений, используемых в файлах workflow.xml или ooziewf.hql, таких как `${nameNode}`.  Если используется путь `wasbs`, его необходимо указать полностью. Не сокращайте его `wasbs:///` . Запись `oozie.wf.application.path` определяет, где можно найти файл workflow.xml. Этот файл содержит рабочий процесс, запущенный этим заданием.

3. Используйте следующую команду для создания конфигурации определения задания Oozie:

    ```bash
    nano job.xml
    ```

4. После открытия редактора nano вставьте измененный XML-файл в качестве содержимого файла.

5. Чтобы сохранить файл, нажмите **клавиши CTRL + X**, введите **Y**, а затем нажмите клавишу **Ввод**.

## <a name="submit-and-manage-the-job"></a>Отправка задания и управление им

Далее используется команда Oozie для отправки рабочих процессов Oozie в кластер и управления ими. Команда Oozie предоставляет удобный интерфейс для [Oozie REST API](https://oozie.apache.org/docs/4.1.0/WebServicesAPI.html).

> [!IMPORTANT]  
> При использовании команды Oozie необходимо использовать полное доменное имя для головного узла HDInsight. Это полное доменное имя доступно только из кластера или, если кластер находится в виртуальной сети Azure, с других компьютеров той же сети.

1. Для получения URL-адреса службы Oozie, воспользуйтесь следующей командой:

    ```bash
    sed -n '/<name>oozie.base.url/,/<\/value>/p' /etc/oozie/conf/oozie-site.xml
    ```

    Эта команда возвращает информацию, как в следующем коде XML:

    ```xml
    <name>oozie.base.url</name>
    <value>http://ACTIVE-HEADNODE-NAME.UNIQUEID.cx.internal.cloudapp.net:11000/oozie</value>
    ```

    Здесь фрагмент `http://ACTIVE-HEADNODE-NAME.UNIQUEID.cx.internal.cloudapp.net:11000/oozie` обозначает URL-адрес, который используется в команде Oozie.

2. Измените код, чтобы заменить URL-адрес на тот, который был получен ранее. Используйте следующую команду для создания переменной среды для URL-адреса, чтобы не вводить его в каждой команде:

    ```bash
    export OOZIE_URL=http://HOSTNAMEt:11000/oozie
    ```

3. Чтобы отправить задание, используйте следующий код:

    ```bash
    oozie job -config job.xml -submit
    ```

    Эта команда загружает сведения о задании из `job.xml` и отправляет их в Oozie, но не запускает ее.

    После завершения команда должна вернуть идентификатор задания, например `0000005-150622124850154-oozie-oozi-W`. Этот идентификатор используется для управления заданием.

4. Измените приведенный ниже код, чтобы заменить `<JOBID>` идентификатором, возвращенным на предыдущем шаге.  Чтобы просмотреть состояние задания, используйте следующую команду:

    ```bash
    oozie job -info <JOBID>
    ```

    Эта команда возвращает следующую информацию:

    ```output
    Job ID : 0000005-150622124850154-oozie-oozi-W
    ------------------------------------------------------------------------------------------------------------------------------------
    Workflow Name : useooziewf
    App Path      : wasb:///tutorials/useoozie
    Status        : PREP
    Run           : 0
    User          : USERNAME
    Group         : -
    Created       : 2015-06-22 15:06 GMT
    Started       : -
    Last Modified : 2015-06-22 15:06 GMT
    Ended         : -
    CoordAction ID: -
    ------------------------------------------------------------------------------------------------------------------------------------
    ```

    Это задание находится в состоянии `PREP`, Это состояние указывает на то, что задание было создано, но не запущено.

5. Измените приведенный ниже код, чтобы заменить `<JOBID>` идентификатором, возвращенным ранее.  Для запуска задания выполните следующую команду:

    ```bash
    oozie job -start <JOBID>
    ```

    Если вы проверите состояние после этой команды, то увидите состояние выполнения и информацию о действиях для этого задания.  Выполнение задания займет несколько минут.

6. Измените приведенный ниже код, заменив его `<serverName>` именем сервера и именем `<sqlLogin>` входа сервера.  *После успешного завершения задачи* можно убедиться, что данные были созданы и экспортированы в таблицу базы данных SQL с помощью следующей команды.  Введите пароль в командной строке.

    ```bash
    TDSVER=8.0 tsql -H <serverName>.database.windows.net -U <sqlLogin> -p 1433 -D oozietest
    ```

    В командной строке `1>` введите следующий запрос.

    ```sql
    SELECT * FROM mobiledata
    GO
    ```

    При этом возвращается следующая информация:

    ```output
    deviceplatform  count
    Android 31591
    iPhone OS       22731
    proprietary development 3
    RIM OS  3464
    Unknown 213
    Windows Phone   1791
    (6 rows affected)
    ```

Дополнительные сведения о команде Oozie см. на [странице, посвященной программе командной строки Apache Oozie](https://oozie.apache.org/docs/4.1.0/DG_CommandLineTool.html).

## <a name="oozie-rest-api"></a>Oozie REST API

Oozie REST API позволяет создавать собственные утилиты для работы с Oozie. Ниже приведены сведения об использовании REST API Oozie в HDInsight.

* **URI.** Вы можете получить доступ к REST API из-за пределов кластера по адресу `https://CLUSTERNAME.azurehdinsight.net/oozie`.

* **Аутентификация.** Для аутентификации используйте API, учетную запись кластера HTTP (администратор) и пароль. Например:

    ```bash
    curl -u admin:PASSWORD https://CLUSTERNAME.azurehdinsight.net/oozie/versions
    ```

Дополнительные сведения об использовании Oozie REST API см. в статье об [API веб-служб Apache Oozie](https://oozie.apache.org/docs/4.1.0/WebServicesAPI.html).

## <a name="oozie-web-ui"></a>веб-интерфейс Oozie

Веб-интерфейс Oozie позволяет получить информацию о состоянии задания Oozie в кластере. С помощью веб-интерфейса вы можете просмотреть следующую информацию:

   * Состояние задания
   * определение задания;
   * Конфигурация
   * диаграмму действий задания;
   * журналы задания.

Кроме того, можно просмотреть подробную информацию о действиях в рамках задания.

Для доступа к веб-интерфейсу Oozie сделайте следующее:

1. Создайте туннель SSH для кластера HDInsight. Дополнительные сведения см. в статье [Использование туннелирования SSH с для доступа к веб-интерфейсу Ambari, JobHistory, NameNode, Oozie и другим веб-интерфейсам](hdinsight-linux-ambari-ssh-tunnel.md).

2. После создания туннеля откройте веб-интерфейс Ambari в веб-браузере с помощью URI `http://headnodehost:8080` .

3. В левой части страницы выберите **Oozie**  >  **быстрые ссылки**  >  **Oozie веб-интерфейс**.

    :::image type="content" source="./media/hdinsight-use-oozie-linux-mac/hdi-oozie-web-ui-steps.png" alt-text="Шаги пользовательского веб-интерфейса Apache Ambari Oozie" border="true":::

4. По умолчанию в веб-интерфейсе Oozie отображаются запущенные задания рабочих процессов. Чтобы просмотреть все задания рабочего процесса, выберите **All Jobs** (Все задания).

    :::image type="content" source="./media/hdinsight-use-oozie-linux-mac/hdinsight-oozie-jobs.png" alt-text="Задания рабочего процесса веб-консоли Oozie" border="true":::

5. Чтобы просмотреть дополнительные сведения о задании, выберите это задание.

    :::image type="content" source="./media/hdinsight-use-oozie-linux-mac/hdinsight-oozie-job-info.png" alt-text="Сведения о задании HDInsight Apache Oozie" border="true":::

6. На вкладке **Job Info** (Сведения о задании) можно просмотреть базовую информацию о задании, а также отдельные действия в рамках задания. С помощью вкладок вверху можно просмотреть **определение задания**, **конфигурацию задания**, обратиться к **журналу задания** или просмотреть направленный ациклический граф (DAG) задания в разделе **Job DAG** (Направленный ациклический граф задания).

   * **Job Log** (Журнал задания). Нажмите кнопку **Get Logs** (Получить журналы) для просмотра всех журналов задания или воспользуйтесь полем **Enter Search Filter** (Введите фильтр поиска) для выбора журналов с помощью фильтра.

       :::image type="content" source="./media/hdinsight-use-oozie-linux-mac/hdinsight-oozie-job-log.png" alt-text="Журнал заданий HDInsight Apache Oozie" border="true":::

   * **Job DAG** (Направленный ациклический граф задания). DAG представляет собой графическое отображение путей данных рабочего процесса.

       :::image type="content" source="./media/hdinsight-use-oozie-linux-mac/hdinsight-oozie-job-dag.png" alt-text="&quot;HDInsight Apache Oozie Job DAG&quot;" border="true":::

7. Выбрав одно из действий на вкладке **Job Info** (Сведения о задании), вы увидите информацию об этом действии. Например, выберите действие **RunSqoopExport**.

    :::image type="content" source="./media/hdinsight-use-oozie-linux-mac/oozie-job-action-info.png" alt-text="Сведения о действии задания Oozie HDInsight" border="true":::

8. Вы можете просмотреть подробную информацию о действии, например ссылку на **URL-адрес консоли**. Используйте эту ссылку для просмотра сведений о задании в средстве отслеживания заданий.

## <a name="schedule-jobs"></a>Запланированные задания

Вы можете использовать координатор, чтобы указать время начала, окончания и частоту выполнения заданий. Чтобы задать расписание для рабочего процесса, выполните следующие действия:

1. Выполните следующую команду для создания файла с именем **coordinator.xml**:

    ```bash
    nano coordinator.xml
    ```

    Используйте следующий код XML в качестве содержимого файла:

    ```xml
    <coordinator-app name="my_coord_app" frequency="${coordFrequency}" start="${coordStart}" end="${coordEnd}" timezone="${coordTimezone}" xmlns="uri:oozie:coordinator:0.4">
        <action>
        <workflow>
            <app-path>${workflowPath}</app-path>
        </workflow>
        </action>
    </coordinator-app>
    ```

    > [!NOTE]  
    > При запуске задания переменные `${...}` будут заменены значениями, указанными в определении задания. Используются следующие переменные:
    >
    > * `${coordFrequency}`: интервал времени между запуском повторных экземпляров задания.
    > * `${coordStart}`: время запуска задания.
    > * `${coordEnd}`: время завершения задания.
    > * `${coordTimezone}`: задания координатора задаются для конкретного часового пояса без летнего времени (обычно представляется в виде времени в формате UTC). Этот часовой пояс называется *часовым поясом обработки Oozie.*
    > * `${wfPath}`: путь к файлу workflow.xml.

2. Чтобы сохранить файл, нажмите **клавиши CTRL + X**, введите **Y**, а затем нажмите клавишу **Ввод**.

3. Чтобы скопировать этот файл в рабочий каталог задания, воспользуйтесь следующей командой:

    ```bash
    hadoop fs -put coordinator.xml /tutorials/useoozie/coordinator.xml
    ```

4. Чтобы изменить `job.xml` созданный ранее файл, используйте следующую команду:

    ```bash
    nano job.xml
    ```

    Внесите следующие изменения:

   * Чтобы служба Oozie запускала файл координатора вместо файла рабочего процесса, измените `<name>oozie.wf.application.path</name>` на `<name>oozie.coord.application.path</name>`.

   * Чтобы задать переменную `workflowPath`, используемую координатором, добавьте следующий код XML:

        ```xml
        <property>
            <name>workflowPath</name>
            <value>wasbs://mycontainer@mystorageaccount.blob.core.windows.net/tutorials/useoozie</value>
        </property>
        ```

       Замените текст `wasbs://mycontainer@mystorageaccount.blob.core.windows` значением, которое используется в других записях файла job.xml.

   * Чтобы определить начало, окончание и частоту выполнения для координатора, добавьте следующий код XML:

        ```xml
        <property>
            <name>coordStart</name>
            <value>2018-05-10T12:00Z</value>
        </property>

        <property>
            <name>coordEnd</name>
            <value>2018-05-12T12:00Z</value>
        </property>

        <property>
            <name>coordFrequency</name>
            <value>1440</value>
        </property>

        <property>
            <name>coordTimezone</name>
            <value>UTC</value>
        </property>
        ```

       Эти значения задают время начала, равное 12:00, в 10 мая 2018, а время окончания — 12 мая 2018. Интервал запуска этого задания — "ежедневно". Интервал задается в минутах, то есть 24 часа x 60 минут = 1440 минут. И наконец, часовой пояс устанавливается в формате UTC.

5. Чтобы сохранить файл, нажмите **клавиши CTRL + X**, введите **Y**, а затем нажмите клавишу **Ввод**.

6. Чтобы отправить и запустить задание, используйте следующую команду:

    ```bash
    oozie job -config job.xml -run
    ```

7. Если вы зайдете в веб-интерфейс Oozie и выберете вкладку **Coordinator Jobs** (Задания координатора), то увидите похожую информацию:

    :::image type="content" source="./media/hdinsight-use-oozie-linux-mac/coordinator-jobs-tab.png" alt-text="Вкладка заданий координатора веб-консоли Oozie" border="true":::

    Запись **Next Materialization** (Следующая материализация) определяет момент следующего запуска задания.

8. Как и для предыдущих заданий рабочего процесса, если вы выберете это задание в веб-интерфейсе, то увидите информацию о нем:

    :::image type="content" source="./media/hdinsight-use-oozie-linux-mac/coordinator-job-info.png" alt-text="Сведения о задании координатора Apache Oozie" border="true":::

    > [!NOTE]  
    > На этом изображении показаны только сведения об успешных запусках задания, а не сведения об отдельных действиях запланированного рабочего процесса. Для просмотра отдельных действий выберите одну из записей **Action** (Действие).

    :::image type="content" source="./media/hdinsight-use-oozie-linux-mac/coordinator-action-job.png" alt-text="Вкладка сведений о задании веб-консоли OOzie" border="true":::

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали, как определить рабочий процесс Oozie и как выполнить задание Oozie. Дополнительные сведения о работе с HDInsight приведены в следующих статьях:

* [Отправка данных для заданий Apache Hadoop в HDInsight](hdinsight-upload-data.md)
* [Использование Apache Sqoop с Apache Hadoop в HDInsight](hadoop/apache-hadoop-use-sqoop-mac-linux.md)
* [Использование Apache Hive с Apache Hadoop в HDInsight](hadoop/hdinsight-use-hive.md)
* [Устранение неполадок с Apache Oozie](./troubleshoot-oozie.md)