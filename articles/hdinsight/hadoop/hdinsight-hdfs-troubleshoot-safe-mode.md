---
title: Локальная служба HDFS зависает в защищенном режиме в кластере Azure HDInsight
description: Устранение неполадок локального Apache HDFS в защищенном режиме в кластере Apache в Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 08/14/2019
ms.openlocfilehash: d34bf8d82aee14f5ba835f68a061555d24ee2621
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98944441"
---
# <a name="scenario-local-hdfs-stuck-in-safe-mode-on-azure-hdinsight-cluster"></a>Сценарий: локальная служба HDFS зависает в защищенном режиме в кластере Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы решения проблем при взаимодействии с кластерами Azure HDInsight.

## <a name="issue"></a>Проблема

Локальная распределенная файловая система Apache Hadoop (HDFS) зависла в безопасном режиме в кластере HDInsight. Появится сообщение об ошибке, похожее на следующее:

```output
hdiuser@spark2:~$ hdfs dfs -D "fs.default.name=hdfs://mycluster/" -mkdir /temp
17/04/05 16:20:52 WARN retry.RetryInvocationHandler: Exception while invoking ClientNamenodeProtocolTranslatorPB.mkdirs over spark2.2oyzcdm4sfjuzjmj5dnmvscjpg.dx.internal.cloudapp.net/10.0.0.22:8020. Not retrying because try once and fail.
org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.hdfs.server.namenode.SafeModeException): Cannot create directory /temp. Name node is in safe mode.
It was turned on manually. Use "hdfs dfsadmin -safemode leave" to turn safe mode off.
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.checkNameNodeSafeMode(FSNamesystem.java:1359)
...
mkdir: Cannot create directory /temp. Name node is in safe mode.
```

## <a name="cause"></a>Причина

Кластер HDInsight был уменьшен до нескольких узлов ниже, или количество узлов близко к фактору репликации HDFS.

## <a name="resolution"></a>Решение

1. Сообщите о состоянии HDFS в кластере HDInsight с помощью следующей команды:

    ```bash
    hdfs dfsadmin -D "fs.default.name=hdfs://mycluster/" -report
    ```

1. Проверьте целостность HDFS в кластере HDInsight с помощью следующей команды:

    ```bash
    hdiuser@spark2:~$ hdfs fsck -D "fs.default.name=hdfs://mycluster/" /
    ```

1. Если вы определили, что отсутствующих, поврежденных, либо нереплицированных блоков нет или что такие блоки можно игнорировать, выполните следующую команду, чтобы вывести узел имен из безопасного режима:

    ```bash
    hdfs dfsadmin -D "fs.default.name=hdfs://mycluster/" -safemode leave
    ```

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]