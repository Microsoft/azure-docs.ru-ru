---
title: Краткое руководство. Apache HBase и Apache Phoenix — Azure HDInsight
description: В этом кратком руководстве вы узнаете, как использовать Apache Phoenix в HDInsight. Также здесь объясняется, как установить и настроить SQLLine на компьютере для подключения к кластеру HBase в HDInsight.
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: quickstart
ms.date: 06/12/2019
ms.openlocfilehash: 6b9196d9ad5cf07cd210726bc1af7121c08094a8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98935988"
---
# <a name="quickstart-query-apache-hbase-in-azure-hdinsight-with-apache-phoenix"></a>Краткое руководство. Выполнение запросов Apache HBase в Azure HDInsight с помощью Apache Phoenix

В этом кратком руководстве вы узнаете, как использовать Apache Phoenix для выполнения запросов HBase в Azure HDInsight. Apache Phoenix — это система запросов SQL для Apache HBase. Доступ к системе осуществляется с помощью драйвера JDBC, она позволяет создавать запросы и управлять таблицами HBase с использованием SQL. [SQLLine](http://sqlline.sourceforge.net/) — это программа командной строки для выполнения SQL.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

* Кластер Apache HBase. Чтобы создать кластер HDInsight, обратитесь к разделу [Создание кластера Hadoop](../hadoop/apache-hadoop-linux-tutorial-get-started.md).  Убедитесь, что выбран тип кластера **HBase**.

* Клиент SSH. Дополнительные сведения см. в руководстве по [подключению к HDInsight (Apache Hadoop) с помощью SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="identify-a-zookeeper-node"></a>Определение узла ZooKeeper

При подключении к кластеру HBase необходимо подключиться к одному из узлов Apache ZooKeeper. Каждый кластер HDInsight содержит три узла Zookeeper. Для быстрого определения узла ZooKeeper можно использовать Curl. Измените указанную ниже команду Curl, заменив `PASSWORD` и `CLUSTERNAME` с соответствующими значениями, а затем введите в командной строке команду:

```cmd
curl -u admin:PASSWORD -sS -G https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/ZOOKEEPER/components/ZOOKEEPER_SERVER
```

Часть выходных данных будет выглядеть следующим образом:

```output
    {
      "href" : "http://hn1-brim.432dc3rlshou3ocf251eycoapa.bx.internal.cloudapp.net:8080/api/v1/clusters/myCluster/hosts/zk0-brim.432dc3rlshou3ocf251eycoapa.bx.internal.cloudapp.net/host_components/ZOOKEEPER_SERVER",
      "HostRoles" : {
        "cluster_name" : "myCluster",
        "component_name" : "ZOOKEEPER_SERVER",
        "host_name" : "zk0-brim.432dc3rlshou3ocf251eycoapa.bx.internal.cloudapp.net"
      }
```

Запишите значение `host_name` для последующего использования.

## <a name="create-a-table-and-manipulate-data"></a>Создание таблицы и обработка данных

Для подключения к кластерам HBase можно использовать протокол SSH, а для создания таблиц HBase, вставки данных и создания запросов к данным — Apache Phoenix.

1. С помощью команды `ssh` подключитесь к кластеру HBase. Измените приведенную ниже команду, заменив `CLUSTERNAME` именем своего кластера, а затем введите команду:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

2. Перейдите в каталог клиента Phoenix. Введите следующую команду:

    ```bash
    cd /usr/hdp/current/phoenix-client/bin
    ```

3. Запустите [SQLLine](http://sqlline.sourceforge.net/). Измените указанную ниже команду, заменив `ZOOKEEPER` на узел ZooKeeper, определенный ранее, и введите команду:

    ```bash
    ./sqlline.py ZOOKEEPER:2181:/hbase-unsecure
    ```

4. Создайте таблицу HBase. Введите следующую команду:

    ```sql
    CREATE TABLE Company (company_id INTEGER PRIMARY KEY, name VARCHAR(225));
    ```

5. С помощью команды SQLLine `!tables` выведите список всех таблиц в HBase. Введите следующую команду:

    ```sqlline
    !tables
    ```

6. Вставьте значения в таблицу. Введите следующую команду:

    ```sql
    UPSERT INTO Company VALUES(1, 'Microsoft');
    UPSERT INTO Company VALUES(2, 'Apache');
    ```

7. Отправьте запрос к таблице. Введите следующую команду:

    ```sql
    SELECT * FROM Company;
    ```

8. Удалите запись. Введите следующую команду:

    ```sql
    DELETE FROM Company WHERE COMPANY_ID=1;
    ```

9. Удалите таблицу. Введите следующую команду:

    ```hbase
    DROP TABLE Company;
    ```

10. С помощью команды SQLLine `!quit` выйдите из SQLLine. Введите следующую команду:

    ```sqlline
    !quit
    ```

## <a name="clean-up-resources"></a>Очистка ресурсов

После завершения работы с этим кратким руководством кластер можно удалить. В случае с HDInsight ваши данные хранятся в службе хранилища Azure, что позволяет безопасно удалить неиспользуемый кластер. Плата за кластеры HDInsight взимается, даже когда они не используются. Поскольку стоимость кластера во много раз превышает стоимость хранилища, экономически целесообразно удалять неиспользуемые кластеры.

Инструкции по удалению кластера см. в статье [Delete an HDInsight cluster using your browser, PowerShell, or the Azure CLI](../hdinsight-delete-cluster.md) (Удаление кластера HDInsight с помощью браузера, PowerShell или Azure CLI).

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнаете, как использовать Apache Phoenix для выполнения запросов HBase в Azure HDInsight. Дополнительные сведения об Apache Phoenix можно найти в следующей статье.

> [!div class="nextstepaction"]
> [Apache Phoenix в HDInsight](../hdinsight-phoenix-in-hdinsight.md)
