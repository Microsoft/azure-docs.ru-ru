---
title: Apache Spark и Hive — Hive Warehouse Connector — Azure HDInsight
description: Узнайте, как интегрировать Apache Spark и Apache Hive с помощью Hive Warehouse Connector в Azure HDInsight.
author: nis-goel
ms.author: nisgoel
ms.service: hdinsight
ms.topic: how-to
ms.date: 05/28/2020
ms.openlocfilehash: 6611f5ca7ddae243c4bc314be73a9030311cec89
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99594440"
---
# <a name="integrate-apache-spark-and-apache-hive-with-hive-warehouse-connector-in-azure-hdinsight"></a>Интеграция Apache Spark и Apache Hive с помощью Hive Warehouse Connector в Azure HDInsight

Apache Hive Warehouse Connector — это библиотека, которая позволяет быстрее работать с Apache Spark и Apache Hive. Она поддерживает такие задачи, как перемещение данных между кадрами и таблицами Hive в Spark. А также направление данных потоковой передачи Spark в таблицы Hive. Hive Warehouse Connector работает как мост между Spark и Hive. Он также поддерживает Scala, Java и Python в качестве языков программирования для разработки.

Hive Warehouse Connector позволяет использовать преимущества уникальных функций Hive и Spark для создания мощных приложений для работы с большими данными.

Apache Hive предлагает поддержку для транзакций базы данных, которые являются атомарными, постоянными, изолированными и устойчивыми (ACID). Дополнительные сведения о ACID и транзакциях в Hive см. в разделе [Транзакции Hive](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions). Hive также предлагает подробные элементы управления безопасностью с помощью Apache Ranger и аналитической обработки с низкой задержкой (LLAP), которая недоступна в Apache Spark.

Apache Spark имеет API структурированной потоковой передачи, который предоставляет возможности потоковой передачи, недоступные в Apache Hive. Начиная с HDInsight 4.0, Apache Spark 2.3.1 и Apache Hive 3.1.0 имеют отдельные хранилища метаданных. Отдельные хранилища метаданных могут усложнить взаимодействие. Hive Warehouse Connector упрощает совместное использование Spark и Hive. Библиотека HWC загружает данные из управляющих программ LLAP в исполнители Spark параллельно. Этот процесс повышает эффективность и адаптацию по сравнению со стандартным соединением JDBC для Spark к Hive.

![Архитектура Hive Warehouse Connector](./media/apache-hive-warehouse-connector/hive-warehouse-connector-architecture.png)

Операции, поддерживаемые Hive Warehouse Connector:

* описание таблицы;
* создание таблицы для данных в формате ORC;
* выбор данных Hive и получение кадра данных;
* пакетная запись кадров данных в Hive;
* выполнение инструкции обновления Hive;
* чтение данных таблицы из Hive, их преобразование в Spark и запись в новую таблицу Hive;
* запись кадра данных или потока Spark в Hive с помощью HiveStreaming.

## <a name="hive-warehouse-connector-setup"></a>Настройка Hive Warehouse Connector

> [!IMPORTANT]
> - Интерактивный экземпляр HiveServer2, установленный на кластерах Spark 2,4 Корпоративный пакет безопасности, не поддерживается для использования с соединителем хранилища Hive. Вместо этого необходимо настроить отдельный интерактивный кластер HiveServer2 для размещения интерактивных рабочих нагрузок HiveServer2. Конфигурация соединителя хранилища Hive, использующая один кластер Spark 2,4, не поддерживается.
> - Библиотека соединителя хранилища Hive (HWC) не поддерживается для использования в кластерах интерактивных запросов, где включена функция управления рабочими нагрузками (ВЛМ). <br>
В сценарии, в котором имеются только рабочие нагрузки Spark и требуется использовать библиотеку HWC, убедитесь, что в интерактивном кластере запросов не включена функция управления рабочими нагрузками ( `hive.server2.tez.interactive.queue` Конфигурация не задана в конфигурациях Hive). <br>
В сценарии, в котором существуют как рабочие нагрузки Spark (HWC), так и собственные рабочие нагрузки LLAP, необходимо создать два отдельных кластера интерактивных запросов с общей базой данных хранилище метаданных. Один кластер для рабочих нагрузок машинного LLAP, где функция ВЛМ может быть включена по мере необходимости, и другой кластер для HWC рабочей нагрузки, в которой не следует настраивать функцию ВЛМ.
Важно отметить, что планы ресурсов ВЛМ можно просматривать из обоих кластеров, даже если они включены только в одном кластере. Не вносите изменения в планы ресурсов в кластере, в котором функция ВЛМ отключена, так как она может повлиять на функциональность ВЛМ в другом кластере.

Для Hive Warehouse Connector требуются отдельные кластеры для рабочих нагрузок Spark и Interactive Query. Выполните следующие действия, чтобы настроить эти кластеры в Azure HDInsight.

### <a name="create-clusters"></a>Создание кластеров

1. Создайте кластер HDInsight Spark **4.0** с учетной записью хранения и настраиваемой виртуальной сетью Azure. Сведения о создании кластера в виртуальной сети Azure см. в статье [Добавление HDInsight в существующую виртуальную сеть](../../hdinsight/hdinsight-plan-virtual-network-deployment.md#existingvnet).

1. Создайте кластер HDInsight Interactive Query (LLAP) **4.0** с той же учетной записью хранения и виртуальной сетью Azure, что и кластер Spark.

### <a name="configure-hwc-settings"></a>Настройка параметров HWC

#### <a name="gather-preliminary-information"></a>Сбор предварительных сведений

1. В веб-браузере перейдите по адресу `https://LLAPCLUSTERNAME.azurehdinsight.net/#/main/services/HIVE`, где LLAPCLUSTERNAME — имя кластера Interactive Query.

1. Перейдите в раздел **Сводка** > **URL-адрес HiveServer2 Interactive JDBC** и запишите значение. Значение может выглядеть следующим образом: `jdbc:hive2://zk0-iqgiro.rekufuk2y2ce.bx.internal.cloudapp.net:2181,zk1-iqgiro.rekufuk2y2ce.bx.internal.cloudapp.net:2181,zk4-iqgiro.rekufuk2y2ce.bx.internal.cloudapp.net:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2-interactive`.

1. Перейдите в раздел **Конфигурации** > **Дополнительно** > **Расширенный hive-site** > **hive.zookeeper.quorum** и запишите значение. Значение может выглядеть следующим образом: `zk0-iqgiro.rekufuk2y2cezcbowjkbwfnyvd.bx.internal.cloudapp.net:2181,zk1-iqgiro.rekufuk2y2cezcbowjkbwfnyvd.bx.internal.cloudapp.net:2181,zk4-iqgiro.rekufuk2y2cezcbowjkbwfnyvd.bx.internal.cloudapp.net:2181`.

1. Перейдите в раздел **Конфигурации** > **Дополнительно** > **Общие** > **hive.metastore.uris** и запишите значение. Значение может выглядеть следующим образом: `thrift://iqgiro.rekufuk2y2cezcbowjkbwfnyvd.bx.internal.cloudapp.net:9083,thrift://hn1-iqgiro.rekufuk2y2cezcbowjkbwfnyvd.bx.internal.cloudapp.net:9083`.

1. Перейдите в раздел **Конфигурации** > **Дополнительно** > **Расширенный hive-interactive-site** > **hive.llap.daemon.service.hosts** и запишите значение. Значение может выглядеть следующим образом: `@llap0`.

#### <a name="configure-spark-cluster-settings"></a>Настройка параметров кластера Spark

1. В веб-браузере перейдите на страницу с адресом `https://CLUSTERNAME.azurehdinsight.net/#/main/services/SPARK2/configs`, где CLUSTERNAME — имя кластера Apache Spark.

1. Разверните **Custom spark2-defaults**.

    ![Настройка Apache Ambari Spark2](./media/apache-hive-warehouse-connector/hive-warehouse-connector-spark2-ambari.png)

1. Щелкните **Добавить свойство...** , чтобы добавить следующие конфигурации:

    | Конфигурация | Значение |
    |----|----|
    |`spark.datasource.hive.warehouse.load.staging.dir`|`wasbs://STORAGE_CONTAINER_NAME@STORAGE_ACCOUNT_NAME.blob.core.windows.net/tmp`. <br> Задайте подходящий каталог временного размещения, совместимый с HDFS. При наличии двух разных кластеров каталог временного размещения должен быть папкой в каталоге временного размещения учетной записи хранения кластера LLAP, чтобы у HiveServer2 был доступ к нему.  Замените `STORAGE_ACCOUNT_NAME` именем учетной записи хранения, используемой кластером, а `STORAGE_CONTAINER_NAME` — именем контейнера хранилища. |
    |`spark.sql.hive.hiveserver2.jdbc.url`| Значение, полученное ранее из **URL-адреса HiveServer2 Interactive JDBC** |
    |`spark.datasource.hive.warehouse.metastoreUri`| Значение, полученное ранее из **hive.metastore.uris**. |
    |`spark.security.credentials.hiveserver2.enabled`|`true` для режима кластера YARN и `false` для режима клиента YARN. |
    |`spark.hadoop.hive.zookeeper.quorum`| Значение, полученное ранее из **hive.zookeeper.quorum**. |
    |`spark.hadoop.hive.llap.daemon.service.hosts`| Значение, полученное ранее из **hive.llap.daemon.service.hosts**. |

1. Сохраните изменения и перезапустите все затронутые компоненты.

### <a name="configure-hwc-for-enterprise-security-package-esp-clusters"></a>Настройка HWC для кластеров с Корпоративным пакетом безопасности (ESP).

Корпоративный пакет безопасности (ESP) обеспечивает возможности корпоративного класса, например проверку подлинности на основе Active Directory, многопользовательскую поддержку и управление доступом на основе ролей для кластеров Apache Hadoop в Azure HDInsight. Дополнительные сведения о Корпоративном пакете безопасности см. в статье [Использование Корпоративного пакета безопасности в HDInsight](../domain-joined/apache-domain-joined-architecture.md).

Помимо конфигураций, упомянутых в предыдущем разделе, добавьте следующую конфигурацию для использования HWC в кластерах ESP.

1. Из веб-интерфейса Ambari в кластере Spark перейдите в раздел **Spark2** > **Конфигурации** > **Custom spark2-defaults**.

1. Обновите следующее свойство.

    | Конфигурация | Значение |
    |----|----|
    | `spark.sql.hive.hiveserver2.jdbc.url.principal`    | `hive/<llap-headnode>@<AAD-Domain>` |
    
    * В веб-браузере перейдите в `https://CLUSTERNAME.azurehdinsight.net/#/main/services/HIVE/summary` папку, где имя_кластера — имя кластера интерактивных запросов. Щелкните **HiveServer2 Interactive (интерактивный**). Вы увидите полное доменное имя (FQDN) головного узла, на котором работает LLAP, как показано на снимке экрана. Замените `<llap-headnode>` на это значение.

        ![Головной узел соединителя хранилища Hive](./media/apache-hive-warehouse-connector/head-node-hive-server-interactive.png)

    * Используйте [команду SSH](../hdinsight-hadoop-linux-use-ssh-unix.md) для подключения к кластеру интерактивных запросов. Найдите `default_realm` параметр в `/etc/krb5.conf` файле. Замените `<AAD-DOMAIN>` на это значение строкой в верхнем регистре, в противном случае не будут найдены учетные данные.

        ![соединитель хранилища Hive домен AAD](./media/apache-hive-warehouse-connector/aad-domain.png)

    * Например, `hive/hn0-ng36ll.mjry42ikpruuxgs2qy2kpg4q5e.cx.internal.cloudapp.net@PKRSRVUQVMAE6J85.D2.INTERNAL.CLOUDAPP.NET` .
    
1. Сохраните изменения и перезапустите необходимые компоненты.

## <a name="hive-warehouse-connector-usage"></a>Использование Hive Warehouse Connector

Вы можете выбрать один из нескольких методов подключения к кластеру Interactive Query и выполнения запросов с помощью Hive Warehouse Connector. Поддерживаемые методы включают следующие инструменты:

* [Spark-shell / PySpark](../spark/apache-spark-shell.md)
* [Spark-submit](#spark-submit)
* [Zeppelin](./apache-hive-warehouse-connector-zeppelin.md)


Ниже приведено несколько примеров для подключения к HWC из Spark.

### <a name="spark-shell"></a>Spark-shell

1. С помощью команды [ssh command](../hdinsight-hadoop-linux-use-ssh-unix.md) подключитесь к кластеру Apache Spark. Измените приведенную ниже команду, заменив CLUSTERNAME именем своего кластера, а затем введите команду:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. В сеансе SSH выполните следующую команду, чтобы отметить версию `hive-warehouse-connector-assembly`:

    ```bash
    ls /usr/hdp/current/hive_warehouse_connector
    ```

1. Измените приведенный ниже код, указав версию `hive-warehouse-connector-assembly`, указанную выше. Затем выполните команду, чтобы запустить оболочку Spark:

    ```bash
    spark-shell --master yarn \
    --jars /usr/hdp/current/hive_warehouse_connector/hive-warehouse-connector-assembly-<VERSION>.jar \
    --conf spark.security.credentials.hiveserver2.enabled=false
    ```

1. После запуска оболочки Spark можно запустить экземпляр Hive Warehouse Connector с помощью следующих команд:

    ```scala
    import com.hortonworks.hwc.HiveWarehouseSession
    val hive = HiveWarehouseSession.session(spark).build()
    ```

### <a name="spark-submit"></a>Spark-submit

После сборки кода Scala/Java вместе с зависимостями в JAR-файл сборки используйте приведенную ниже команду, чтобы запустить приложение Spark. Замените `<VERSION>` и `<APP_JAR_PATH>` соответствующими значениями.

* Режим клиента YARN
    
    ```scala
    spark-submit \
    --class myHwcApp \
    --master yarn \
    --deploy-mode client \
    --jars /usr/hdp/current/hive_warehouse_connector/hive-warehouse-connector-assembly-<VERSION>.jar \
    --conf spark.security.credentials.hiveserver2.enabled=false
    /<APP_JAR_PATH>/myHwcAppProject.jar
    ```

* Режим кластера YARN
    ```scala
    spark-submit \
    --class myHwcApp \
    --master yarn \
    --deploy-mode cluster \
    --jars /usr/hdp/current/hive_warehouse_connector/hive-warehouse-connector-assembly-<VERSION>.jar \
    --conf spark.security.credentials.hiveserver2.enabled=true
    /<APP_JAR_PATH>/myHwcAppProject.jar
    ```

Для Python добавьте также следующую конфигурацию. 

```python
--py-files /usr/hdp/current/hive_warehouse_connector/pyspark_hwc-<VERSION>.zip
```
    
## <a name="run-queries-on-enterprise-security-package-esp-clusters"></a>Запуск запросов на кластерах с Корпоративным пакетом безопасности (ESP)

Используйте `kinit` перед запуском spark-shell или spark-submit. Замените USERNAME именем учетной записи домена, имеющей разрешения на доступ к кластеру, а затем выполните следующую команду:

```bash
kinit USERNAME
```

### <a name="securing-data-on-spark-esp-clusters"></a>Защита данных в кластерах Spark ESP

1. Создайте таблицу `demo` с образцами данных, введя следующие команды:

    ```scala
    create table demo (name string);
    INSERT INTO demo VALUES ('HDinsight');
    INSERT INTO demo VALUES ('Microsoft');
    INSERT INTO demo VALUES ('InteractiveQuery');
    ```

1. Просмотрите содержимое таблицы с помощью следующей команды. Перед применением политики в таблице `demo` отображается полный столбец.

    ```scala
    hive.executeQuery("SELECT * FROM demo").show()
    ```

    ![Демонстрационная таблица перед применением политики Ranger](./media/apache-hive-warehouse-connector/hive-warehouse-connector-table-before-ranger-policy.png)

1. Примените политику маскирования столбцов, которая будет показывать только последние четыре символа столбца.  
    1. Перейдите в пользовательский интерфейс администратора Ranger по адресу `https://LLAPCLUSTERNAME.azurehdinsight.net/ranger/`.
    1. Щелкните службу Hive для кластера в разделе **Hive**.
        ![Диспетчер служб Ranger](./media/apache-hive-warehouse-connector/hive-warehouse-connector-ranger-service-manager.png)
    1. Щелкните вкладку **Маскирование** и нажмите **Добавить новую политику**

        ![список политик hive ranger для hive warehouse connector](./media/apache-hive-warehouse-connector/hive-warehouse-connector-ranger-hive-policy-list.png)

    1. Укажите имя нужной политики. Выберите базу данных: **По умолчанию**, таблица Hive: **демонстрация**, столбец Hive: **имя**, пользователь: **rsadmin2**, типы доступа: **выбранные** и **частичная маска: показывать последние 4** в меню **Выбор параметров маскирования**. Нажмите кнопку **Добавить**.
                ![Создание политики](./media/apache-hive-warehouse-connector/hive-warehouse-connector-ranger-create-policy.png)
1. Снова просмотрите содержимое таблицы. После применения политики Ranger вы увидите только последние четыре символа столбца.

    ![Демонстрационная таблица после применения политики Ranger](./media/apache-hive-warehouse-connector/hive-warehouse-connector-table-after-ranger-policy.png)

## <a name="next-steps"></a>Дальнейшие действия

* [Операции HWC и Apache Spark](./apache-hive-warehouse-connector-operations.md)
* [Use Interactive Query with HDInsight](./apache-interactive-query-get-started.md) (Использование Interactive Query в HDInsight)
* [Интеграция HWC с Apache Zeppelin](./apache-hive-warehouse-connector-zeppelin.md)
* [Примеры взаимодействия с Hive Warehouse Connector с помощью Zeppelin, Livy, spark-submit и pyspark](https://community.hortonworks.com/articles/223626/integrating-apache-hive-with-apache-spark-hive-war.html)
