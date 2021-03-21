---
title: Apache Hive подключения к Apache Zookeeper — Azure HDInsight
description: Apache Hive просмотр недоступен из-за проблем Apache Zookeeper в Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 07/30/2019
ms.openlocfilehash: 42fb9f5389cbc31e772dc9cf36b6a975c5e18d3c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98939300"
---
# <a name="scenario-apache-hive-fails-to-establish-a-connection-to-apache-zookeeper-in-azure-hdinsight"></a>Сценарий: Apache Hive не удается установить подключение к Apache Zookeeper в Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы решения проблем при использовании интерактивных компонентов запросов в кластерах Azure HDInsight.

## <a name="issue"></a>Проблема

Представление Hive недоступно, а в журналах `/var/log/hive` отображается ошибка следующего вида:

```
ERROR [Curator-Framework-0]: curator.ConnectionState (ConnectionState.java:checkTimeouts(200)) - Connection timed out for connection string (zk0-cluster.cloud.wbmi.com:2181,zk1-cluster.cloud.wbmi.com:2181,zk2-cluster.cloud.wbmi.com:2181) and timeout (15000) / elapsed (21852)
```

## <a name="cause"></a>Причина

Возможно, Hive не сможет установить подключение к Zookeeper, что не позволит запустить представление Hive.

## <a name="resolution"></a>Решение

1. Убедитесь, что служба Zookeeper работоспособна.

1. Проверьте, имеет ли служба Zookeeper запись Znode будет удален для Hive Server2. Значение будет отсутствовать или быть неправильным.

    ```
    /usr/hdp/2.6.2.25-1/zookeeper/bin/zkCli.sh -server zk1-wbwdhs
    [zk: zk0-cluster(CONNECTED) 0] ls /hiveserver2-hive2
    ```

1. Чтобы повторно установить подключение, перезагрузите узлы Zookeeper и перезагрузите HiveServer2.

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]