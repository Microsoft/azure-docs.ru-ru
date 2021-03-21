---
title: Hive Warehouse Connector — использование Apache Zeppelin с Livy (Azure HDInsight)
description: Узнайте, как интегрировать Hive Warehouse Connector с Apache Zeppelin в Azure HDInsight.
author: nis-goel
ms.author: nisgoel
ms.service: hdinsight
ms.topic: how-to
ms.date: 05/28/2020
ms.openlocfilehash: 4859ff098bf3fdffbecc70608cb147d17d9d7f59
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98941165"
---
# <a name="integrate-apache-zeppelin-with-hive-warehouse-connector-in-azure-hdinsight"></a>Интеграция Apache Zeppelin с Hive Warehouse Connector в Azure HDInsight

Кластеры Spark HDInsight включают в себя записные книжки Apache Zeppelin с разными интерпретаторами. В этой статье мы рассмотрим применение интерпретатора Livy для доступа к таблицам Hive из Spark с помощью Hive Warehouse Connector.

> [!NOTE]
> Эта статья содержит ссылки на термин *список разрешений*. Корпорация Майкрософт больше не использует его. Когда этот термин будет удален из программного обеспечения, мы удалим его из статьи.

## <a name="prerequisite"></a>Предварительные требования

Выполните инструкции из статьи [Настройка Hive Warehouse Connector](apache-hive-warehouse-connector.md#hive-warehouse-connector-setup).

## <a name="getting-started"></a>Начало работы

1. С помощью [команды ssh](../hdinsight-hadoop-linux-use-ssh-unix.md) подключитесь к кластеру Apache Spark. Измените приведенную ниже команду, заменив CLUSTERNAME именем своего кластера, а затем введите команду:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. В сеансе SSH выполните следующую команду, чтобы узнать версии `hive-warehouse-connector-assembly` и `pyspark_hwc`.

    ```bash
    ls /usr/hdp/current/hive_warehouse_connector
    ```

    Сохраните выходные данные, которые нам потребуются для настройки Apache Zeppelin.

## <a name="configure-livy"></a>Настройка Livy

Для доступа к таблицам Hive из Zeppelin с помощью интерпретатора Livy требуются следующие конфигурации.

### <a name="interactive-query-cluster"></a>Кластер Interactive Query

1. В веб-браузере перейдите по адресу `https://LLAPCLUSTERNAME.azurehdinsight.net/#/main/services/HDFS/configs`, указав вместо LLAPCLUSTERNAME имя кластера Interactive Query.

1. Перейдите в раздел **Advanced** > **Custom core-site** (Дополнительно > Пользовательский базовый сайт). Щелкните **Add Property...** (Добавить свойство...), чтобы добавить следующие конфигурации:

    | Конфигурация                 | Значение |
    | ----------------------------- |-------|
    | hadoop.proxyuser.livy.groups  | *     |
    | hadoop.proxyuser.livy.hosts   | *     |

1. Сохраните изменения и перезапустите все затронутые компоненты.

### <a name="spark-cluster"></a>Кластер Spark

1. В веб-браузере перейдите на страницу с адресом `https://CLUSTERNAME.azurehdinsight.net/#/main/services/SPARK2/configs`, заменив CLUSTERNAME именем кластера Apache Spark.

1. Разверните **Custom livy2-conf**. Щелкните **Add Property...** (Добавить свойство...), чтобы добавить следующие конфигурации.

    | Конфигурация                 | Значение                                      |
    | ----------------------------- |------------------------------------------  |
    | livy.file.local-dir-whitelist | /usr/hdp/current/hive_warehouse_connector/ |

1. Сохраните изменения и перезапустите все затронутые компоненты.

### <a name="configure-livy-interpreter-in-zeppelin-ui-spark-cluster"></a>Настройка интерпретатора Livy в пользовательском интерфейсе Zeppelin (кластер Spark)

1. В веб-браузере перейдите на страницу с адресом `https://CLUSTERNAME.azurehdinsight.net/zeppelin/#/interpreter`, заменив `CLUSTERNAME` именем кластера Apache Spark.

1. Перейдите к **livy2**.

1. Добавьте перечисленные ниже конфигурации.

    | Конфигурация                 | Значение                                      |
    | ----------------------------- |:------------------------------------------:|
    | livy.spark.hadoop.hive.llap.daemon.service.hosts | @llap0 |
    | livy.spark.security.credentials.hiveserver2.enabled | Да |
    | livy.spark.sql.hive.llap | Да |
    | livy.spark.yarn.security.credentials.hiveserver2.enabled | Да |
    | livy.superusers | livy,zeppelin |
    | livy.spark.jars | `file:///usr/hdp/current/hive_warehouse_connector/hive-warehouse-connector-assembly-VERSION.jar`.<br>Замените VERSION значением, которое вы получили ранее в разделе [Начало работы](#getting-started). |
    | livy.spark.submit.pyFiles | `file:///usr/hdp/current/hive_warehouse_connector/pyspark_hwc-VERSION.zip`.<br>Замените VERSION значением, которое вы получили ранее в разделе [Начало работы](#getting-started). |
    | livy.spark.sql.hive.hiveserver2.jdbc.url | Задайте для него URL-адрес HiveServer2 Interactive JDBC для кластера Interactive Query. |
    | spark.security.credentials.hiveserver2.enabled | Да |

1. Для кластеров ESP добавьте также следующую конфигурацию.

    | Конфигурация| Значение|
    |---|---|
    | livy.spark.sql.hive.hiveserver2.jdbc.url.principal | `hive/<llap-headnode>@<AAD-Domain>` |

    * В веб-браузере перейдите в `https://CLUSTERNAME.azurehdinsight.net/#/main/services/HIVE/summary` папку, где имя_кластера — имя кластера интерактивных запросов. Щелкните **HiveServer2 Interactive (интерактивный**). Вы увидите полное доменное имя (FQDN) головного узла, на котором работает LLAP, как показано на снимке экрана. Замените `<llap-headnode>` на это значение.

        ![Головной узел соединителя хранилища Hive](./media/apache-hive-warehouse-connector/head-node-hive-server-interactive.png)

    * Используйте [команду SSH](../hdinsight-hadoop-linux-use-ssh-unix.md) для подключения к кластеру интерактивных запросов. Найдите `default_realm` параметр в `/etc/krb5.conf` файле. Замените `<AAD-DOMAIN>` на это значение строкой в верхнем регистре, в противном случае не будут найдены учетные данные.

        ![соединитель хранилища Hive домен AAD](./media/apache-hive-warehouse-connector/aad-domain.png)

    * Например, `hive/hn0-ng36ll.mjry42ikpruuxgs2qy2kpg4q5e.cx.internal.cloudapp.net@PKRSRVUQVMAE6J85.D2.INTERNAL.CLOUDAPP.NET` .

1. Сохраните изменения и перезапустите интерпретатор Livy.

Если интерпретатор Livy недоступен, измените файл `shiro.ini`, который содержится в компоненте Zeppelin в Ambari. Дополнительные сведения см. в статье [Configuring Apache Zeppelin Security](https://docs.cloudera.com/HDPDocuments/HDP3/HDP-3.0.1/configuring-zeppelin-security/content/enabling_access_control_for_interpreter__configuration__and_credential_settings.html) (Настройка безопасности в Apache Zeppelin).  


## <a name="running-queries-in-zeppelin"></a>Выполнение запросов в Zeppelin 

Запустите записную книжку Zeppelin с помощью интерпретатора Livy и выполните следующий код.

```python
%livy2

import com.hortonworks.hwc.HiveWarehouseSession
import com.hortonworks.hwc.HiveWarehouseSession._
import org.apache.spark.sql.SaveMode

# Initialize the hive context
val hive = HiveWarehouseSession.session(spark).build()

# Create a database
hive.createDatabase("hwc_db",true)
hive.setDatabase("hwc_db")

# Create a Hive table
hive.createTable("testers").ifNotExists().column("id", "bigint").column("name", "string").create()

val dataDF = Seq( (1, "foo"), (2, "bar"), (8, "john")).toDF("id", "name")

# Validate writes to the table
dataDF.write.format("com.hortonworks.spark.sql.hive.llap.HiveWarehouseConnector").mode("append").option("table", "hwc_db.testers").save()

# Validate reads
hive.executeQuery("select * from testers").show()

```

## <a name="next-steps"></a>Дальнейшие действия

* [Операции Apache Spark и Hive Warehouse Connector](./apache-hive-warehouse-connector-operations.md)
* [Интеграция Hive Warehouse Connector с Apache Spark и Apache Hive](./apache-hive-warehouse-connector.md)
* [Use Interactive Query with HDInsight](./apache-interactive-query-get-started.md) (Использование Interactive Query в HDInsight)
