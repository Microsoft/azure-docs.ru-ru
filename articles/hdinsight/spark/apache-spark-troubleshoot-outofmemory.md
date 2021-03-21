---
title: Исключения OutOfMemoryError для Apache Spark в Azure HDInsight
description: Различные исключения OutOfMemoryError для кластера Apache Spark в Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 08/15/2019
ms.openlocfilehash: a15d79f2ae9c3d20a73ec557c57a5c189b18111b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98946345"
---
# <a name="outofmemoryerror-exceptions-for-apache-spark-in-azure-hdinsight"></a>Исключения OutOfMemoryError для Apache Spark в Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы исправления проблем, возникающих при использовании компонентов Apache Spark в кластерах Azure HDInsight.

## <a name="scenario-outofmemoryerror-exception-for-apache-spark"></a>Сценарий: исключение OutOfMemoryError для Apache Spark

### <a name="issue"></a>Проблема

Сбой приложения Apache Spark с необработанным исключением OutOfMemoryError. Может появиться сообщение об ошибке следующего вида:

```error
ERROR Executor: Exception in task 7.0 in stage 6.0 (TID 439)

java.lang.OutOfMemoryError
    at java.io.ByteArrayOutputStream.hugeCapacity(Unknown Source)
    at java.io.ByteArrayOutputStream.grow(Unknown Source)
    at java.io.ByteArrayOutputStream.ensureCapacity(Unknown Source)
    at java.io.ByteArrayOutputStream.write(Unknown Source)
    at java.io.ObjectOutputStream$BlockDataOutputStream.drain(Unknown Source)
    at java.io.ObjectOutputStream$BlockDataOutputStream.setBlockDataMode(Unknown Source)
    at java.io.ObjectOutputStream.writeObject0(Unknown Source)
    at java.io.ObjectOutputStream.writeObject(Unknown Source)
    at org.apache.spark.serializer.JavaSerializationStream.writeObject(JavaSerializer.scala:44)
    at org.apache.spark.serializer.JavaSerializerInstance.serialize(JavaSerializer.scala:101)
    at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:239)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
    at java.lang.Thread.run(Unknown Source)
```

```error
ERROR SparkUncaughtExceptionHandler: Uncaught exception in thread Thread[Executor task launch worker-0,5,main]

java.lang.OutOfMemoryError
    at java.io.ByteArrayOutputStream.hugeCapacity(Unknown Source)
    ...
```

### <a name="cause"></a>Причина

Наиболее вероятной причиной этого исключения является нехватка памяти кучи, выделенной для виртуальных машин Java. Эти виртуальных машин Java запускаются как исполнители или драйверы в составе приложения Apache Spark.

### <a name="resolution"></a>Решение

1. Определите максимальный объем данных, которые будет обрабатывать приложение Spark. Оценка размера в зависимости от максимального размера входных данных — промежуточных данных, полученных путем преобразования входных данных и выходных данных, созданных для преобразования промежуточных данных. Если начальная оценка недостаточна, увеличьте размер и выполните итерацию до подчасти ошибок памяти.

1. Убедитесь, что в кластере HDInsight, который должен использоваться, достаточно ресурсов, таких как память и количество ядер, для работы приложения Spark. Это можно определить, просмотрев раздел метрики кластера в пользовательском интерфейсе YARN в кластере на предмет значений **используемой памяти** и **общего объема памяти** , а также **виртуальных ядер, используемых** в сравнении с **виртуальных ядер**.

    ![представление памяти Yarn Core](./media/apache-spark-ts-outofmemory/yarn-core-memory-view.png)

1. Задайте для следующих конфигураций Spark соответствующие значения. Балансировка требований приложения с помощью доступных ресурсов в кластере. Эти значения не должны превышать 90% доступной памяти и ядер, отображаемых YARN, а также должны соответствовать минимальным требованиям к памяти для приложения Spark:

    ```
    spark.executor.instances (Example: 8 for 8 executor count)
    spark.executor.memory (Example: 4g for 4 GB)
    spark.yarn.executor.memoryOverhead (Example: 384m for 384 MB)
    spark.executor.cores (Example: 2 for 2 cores per executor)
    spark.driver.memory (Example: 8g for 8GB)
    spark.driver.cores (Example: 4 for 4 cores)
    spark.yarn.driver.memoryOverhead (Example: 384m for 384MB)
    ```

    Общий объем памяти, используемый всеми исполнителями =

    ```
    spark.executor.instances * (spark.executor.memory + spark.yarn.executor.memoryOverhead) 
    ```

    Общий объем памяти, используемый драйвером =

    ```
    spark.driver.memory + spark.yarn.driver.memoryOverhead
    ```

---

## <a name="scenario-java-heap-space-error-when-trying-to-open-apache-spark-history-server"></a>Сценарий: ошибка пространства кучи Java при попытке открыть сервер журнала Apache Spark

### <a name="issue"></a>Проблема

При открытии событий на сервере журнала Spark появляется следующее сообщение об ошибке:

```
scala.MatchError: java.lang.OutOfMemoryError: Java heap space (of class java.lang.OutOfMemoryError)
```

### <a name="cause"></a>Причина

Эта проблема часто возникает из-за недостатка ресурсов при открытии больших файлов Spark-событий. Размер кучи Spark по умолчанию устанавливается равным 1 ГБ, но для больших файлов событий Spark может потребоваться больше.

Если вы хотите проверить размер загружаемых файлов, можно выполнить следующие команды:.

```bash
hadoop fs -du -s -h wasb:///hdp/spark2-events/application_1503957839788_0274_1/
**576.5 M**  wasb:///hdp/spark2-events/application_1503957839788_0274_1

hadoop fs -du -s -h wasb:///hdp/spark2-events/application_1503957839788_0264_1/
**2.1 G**  wasb:///hdp/spark2-events/application_1503957839788_0264_1
```

### <a name="resolution"></a>Решение

Можно увеличить объем памяти сервера журнала Spark, отредактировав `SPARK_DAEMON_MEMORY` свойство в конфигурации Spark и перезапуская все службы.

Это можно сделать в пользовательском интерфейсе браузера Ambari, выбрав раздел Spark2/config/Advanced Spark2-env.

![Расширенный раздел spark2-env](./media/apache-spark-ts-outofmemory-heap-space/apache-spark-image01.png)

Добавьте следующее свойство, чтобы изменить объем памяти сервера журнала Spark с 1 ГБ на 4G: `SPARK_DAEMON_MEMORY=4g` .

![Spark, свойство](./media/apache-spark-ts-outofmemory-heap-space/apache-spark-image02.png)

Не забудьте перезапустить все затронутые службы из Ambari.

---

## <a name="scenario-livy-server-fails-to-start-on-apache-spark-cluster"></a>Сценарий: Livy Server не запускается в кластере Apache Spark

### <a name="issue"></a>Проблема

Livy Server нельзя запустить на Apache Spark [(Spark 2,1 в Linux (HDI 3,6)]. Попытка перезапуска приводит к последующему стеку ошибок из журналов Livy:

```log
17/07/27 17:52:50 INFO CuratorFrameworkImpl: Starting
17/07/27 17:52:50 INFO ZooKeeper: Client environment:zookeeper.version=3.4.6-29--1, built on 05/15/2017 17:55 GMT
17/07/27 17:52:50 INFO ZooKeeper: Client environment:host.name=10.0.0.66
17/07/27 17:52:50 INFO ZooKeeper: Client environment:java.version=1.8.0_131
17/07/27 17:52:50 INFO ZooKeeper: Client environment:java.vendor=Oracle Corporation
17/07/27 17:52:50 INFO ZooKeeper: Client environment:java.home=/usr/lib/jvm/java-8-openjdk-amd64/jre
17/07/27 17:52:50 INFO ZooKeeper: Client environment:java.class.path= <DELETED>
17/07/27 17:52:50 INFO ZooKeeper: Client environment:java.library.path= <DELETED>
17/07/27 17:52:50 INFO ZooKeeper: Client environment:java.io.tmpdir=/tmp
17/07/27 17:52:50 INFO ZooKeeper: Client environment:java.compiler=<NA>
17/07/27 17:52:50 INFO ZooKeeper: Client environment:os.name=Linux
17/07/27 17:52:50 INFO ZooKeeper: Client environment:os.arch=amd64
17/07/27 17:52:50 INFO ZooKeeper: Client environment:os.version=4.4.0-81-generic
17/07/27 17:52:50 INFO ZooKeeper: Client environment:user.name=livy
17/07/27 17:52:50 INFO ZooKeeper: Client environment:user.home=/home/livy
17/07/27 17:52:50 INFO ZooKeeper: Client environment:user.dir=/home/livy
17/07/27 17:52:50 INFO ZooKeeper: Initiating client connection, connectString=zk2-kcspark.cxtzifsbseee1genzixf44zzga.gx.internal.cloudapp.net:2181,zk3-kcspark.cxtzifsbseee1genzixf44zzga.gx.internal.cloudapp.net:2181,zk6-kcspark.cxtzifsbseee1genzixf44zzga.gx.internal.cloudapp.net:2181 sessionTimeout=60000 watcher=org.apache.curator.ConnectionState@25fb8912
17/07/27 17:52:50 INFO StateStore$: Using ZooKeeperStateStore for recovery.
17/07/27 17:52:50 INFO ClientCnxn: Opening socket connection to server 10.0.0.61/10.0.0.61:2181. Will not attempt to authenticate using SASL (unknown error)
17/07/27 17:52:50 INFO ClientCnxn: Socket connection established to 10.0.0.61/10.0.0.61:2181, initiating session
17/07/27 17:52:50 INFO ClientCnxn: Session establishment complete on server 10.0.0.61/10.0.0.61:2181, sessionid = 0x25d666f311d00b3, negotiated timeout = 60000
17/07/27 17:52:50 INFO ConnectionStateManager: State change: CONNECTED
17/07/27 17:52:50 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
17/07/27 17:52:50 INFO AHSProxy: Connecting to Application History server at headnodehost/10.0.0.67:10200
Exception in thread "main" java.lang.OutOfMemoryError: unable to create new native thread
  at java.lang.Thread.start0(Native Method)
  at java.lang.Thread.start(Thread.java:717)
  at com.cloudera.livy.Utils$.startDaemonThread(Utils.scala:98)
  at com.cloudera.livy.utils.SparkYarnApp.<init>(SparkYarnApp.scala:232)
  at com.cloudera.livy.utils.SparkApp$.create(SparkApp.scala:93)
  at com.cloudera.livy.server.batch.BatchSession$$anonfun$recover$2$$anonfun$apply$4.apply(BatchSession.scala:117)
  at com.cloudera.livy.server.batch.BatchSession$$anonfun$recover$2$$anonfun$apply$4.apply(BatchSession.scala:116)
  at com.cloudera.livy.server.batch.BatchSession.<init>(BatchSession.scala:137)
  at com.cloudera.livy.server.batch.BatchSession$.recover(BatchSession.scala:108)
  at com.cloudera.livy.sessions.BatchSessionManager$$anonfun$$init$$1.apply(SessionManager.scala:47)
  at com.cloudera.livy.sessions.BatchSessionManager$$anonfun$$init$$1.apply(SessionManager.scala:47)
  at scala.collection.TraversableLike$$anonfun$map$1.apply(TraversableLike.scala:244)
  at scala.collection.TraversableLike$$anonfun$map$1.apply(TraversableLike.scala:244)
  at scala.collection.mutable.ResizableArray$class.foreach(ResizableArray.scala:59)
  at scala.collection.mutable.ArrayBuffer.foreach(ArrayBuffer.scala:47)
  at scala.collection.TraversableLike$class.map(TraversableLike.scala:244)
  at scala.collection.AbstractTraversable.map(Traversable.scala:105)
  at com.cloudera.livy.sessions.SessionManager.com$cloudera$livy$sessions$SessionManager$$recover(SessionManager.scala:150)
  at com.cloudera.livy.sessions.SessionManager$$anonfun$1.apply(SessionManager.scala:82)
  at com.cloudera.livy.sessions.SessionManager$$anonfun$1.apply(SessionManager.scala:82)
  at scala.Option.getOrElse(Option.scala:120)
  at com.cloudera.livy.sessions.SessionManager.<init>(SessionManager.scala:82)
  at com.cloudera.livy.sessions.BatchSessionManager.<init>(SessionManager.scala:42)
  at com.cloudera.livy.server.LivyServer.start(LivyServer.scala:99)
  at com.cloudera.livy.server.LivyServer$.main(LivyServer.scala:302)
  at com.cloudera.livy.server.LivyServer.main(LivyServer.scala)
  
  ## using "vmstat" found  we had enough free memory
```

### <a name="cause"></a>Причина

`java.lang.OutOfMemoryError: unable to create new native thread` ОС с выделенными фрагментами не может назначить больше машинных потоков для виртуальных машин Java. Подтверждает, что это исключение вызвано нарушением числа потоков для каждого процесса.

При неожиданном завершении работы сервера Livy все подключения к кластерам Spark также завершаются. Это означает, что все задания и связанные данные будут потеряны. В механизме восстановления сеанса HDP 2,6 Livy сохраняет сведения о сеансе в Zookeeper для восстановления после возвращения сервера Livy.

При отправке большого количества заданий через Livy в рамках высокой доступности для Livy Server сохраняет эти состояния сеанса в ZK (в кластерах HDInsight) и восстанавливает эти сеансы при перезапуске службы Livy. При перезапуске после непредвиденного завершения Livy создает по одному потоку на сеанс, и это накапливает определенное количество запросов, которые будут восстановлены, что привело к созданию слишком большого количества потоков.

### <a name="resolution"></a>Решение

Удалите все записи с помощью действий, описанных ниже.

1. Получите IP-адрес узлов Zookeeper с помощью

    ```bash
    grep -R zk /etc/hadoop/conf  
    ```

1. Команда выше выводит список всех Zookeeper для кластера

    ```bash
    /etc/hadoop/conf/core-site.xml:      <value>zk1-hwxspa.lnuwp5akw5ie1j2gi2amtuuimc.dx.internal.cloudapp.net:2181,zk2-      hwxspa.lnuwp5akw5ie1j2gi2amtuuimc.dx.internal.cloudapp.net:2181,zk4-hwxspa.lnuwp5akw5ie1j2gi2amtuuimc.dx.internal.cloudapp.net:2181</value>
    ```

1. Получение всех IP-адресов узлов Zookeeper с помощью проверки связи или подключение к Zookeeper из головного узла с помощью имени ZK

    ```bash
    /usr/hdp/current/zookeeper-client/bin/zkCli.sh -server zk2-hwxspa:2181
    ```

1. После подключения к Zookeeper выполните следующую команду, чтобы получить список всех сеансов, которые пытаются перезапуститься.

    1. В большинстве случаев это может быть список из более чем 8000 сеансов. ####

        ```bash
        ls /livy/v1/batch
        ```

    1. Следующая команда позволяет удалить все восстановленные сеансы. #####

        ```bash
        rmr /livy/v1/batch
        ```

1. Дождитесь завершения выполнения приведенной выше команды и курсора, чтобы вернуть запрос, а затем перезапустите службу Livy из Ambari, которая должна быть выполнена.

> [!NOTE]
> `DELETE` сеанс Livy после завершения его выполнения. Сеансы пакетной службы Livy не будут удаляться автоматически сразу после завершения работы приложения Spark, что является конструкцией. Сеанс Livy — это сущность, созданная запросом POST к серверу Livy RESTful. `DELETE`Для удаления этой сущности требуется вызов. Или следует подождать, пока сборщик мусора не запустится.

---

## <a name="next-steps"></a>Дальнейшие действия

Если вы не видите своего варианта проблемы или вам не удается ее устранить, дополнительные сведения можно получить, посетив один из следующих каналов.

* [Общие сведения об управлении памятью Spark](https://spark.apache.org/docs/latest/tuning.html#memory-management-overview).

* [Отладка приложения Spark в кластерах HDInsight](/archive/blogs/azuredatalake/spark-debugging-101).

* Получите ответы специалистов Azure на [сайте поддержки сообщества пользователей Azure](https://azure.microsoft.com/support/community/).

* Подпишитесь на [@AzureSupport](https://twitter.com/azuresupport) — официальный канал Microsoft Azure для работы с клиентами. Вступайте в сообщество Azure для получения нужных ресурсов: ответов, поддержки и советов экспертов.

* Если вам нужна дополнительная помощь, отправьте запрос в службу поддержки на [портале Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Выберите **Поддержка** в строке меню или откройте центр **Справка и поддержка**. Дополнительные сведения см. в статье [Создание запроса на поддержку Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). Доступ к управлению подписками и поддержкой выставления счетов уже включен в вашу подписку Microsoft Azure, а техническая поддержка предоставляется в рамках одного из [планов Службы поддержки Azure](https://azure.microsoft.com/support/plans/).