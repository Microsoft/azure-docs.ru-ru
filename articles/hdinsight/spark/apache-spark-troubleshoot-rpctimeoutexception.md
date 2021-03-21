---
title: Рпктимеаутексцептион для Apache Spark Thrift — Azure HDInsight
description: При обработке больших наборов данных с помощью Apache Spark Thrift Server отображается ошибка 502
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 07/29/2019
ms.openlocfilehash: c2975599272474fed9d61702fc1dbceb946c1190
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98932713"
---
# <a name="scenario-rpctimeoutexception-for-apache-spark-thrift-server-in-azure-hdinsight"></a>Сценарий: Рпктимеаутексцептион для сервера Apache Spark Thrift в Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы исправления проблем, возникающих при использовании компонентов Apache Spark в кластерах Azure HDInsight.

## <a name="issue"></a>Проблема

Приложение Spark завершается с `org.apache.spark.rpc.RpcTimeoutException` исключением и сообщением: `Futures timed out` , как показано в следующем примере:

```
org.apache.spark.rpc.RpcTimeoutException: Futures timed out after [120 seconds]. This timeout is controlled by spark.rpc.askTimeout
 at org.apache.spark.rpc.RpcTimeout.org$apache$spark$rpc$RpcTimeout$$createRpcTimeoutException(RpcTimeout.scala:48)
```

`OutOfMemoryError``overhead limit exceeded`Кроме того, ошибки также могут присутствовать в, `sparkthriftdriver.log` как в следующем примере:

```
WARN  [rpc-server-3-4] server.TransportChannelHandler: Exception in connection from /10.0.0.17:53218
java.lang.OutOfMemoryError: GC overhead limit exceeded
```

## <a name="cause"></a>Причина

Эти ошибки вызваны недостатком ресурсов памяти при обработке данных. Если запускается процесс сборки мусора Java, это может привести к тому, что приложение Spark перестает отвечать на запросы. Запросы начнут истечет время ожидания и прерывать обработку. Эта `Futures timed out` ошибка указывает на кластер в условиях сильной нагрузки.

## <a name="resolution"></a>Решение

Увеличьте размер кластера, добавив дополнительные рабочие узлы или увеличив объем памяти существующих узлов кластера. Можно также настроить конвейер данных, чтобы уменьшить объем обрабатываемых данных.

`spark.network.timeout`Управляет временем ожидания для всех сетевых подключений. Увеличение времени ожидания сети может послужить большим временем для завершения некоторых критических операций, но это не позволит полностью устранить проблему.

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]