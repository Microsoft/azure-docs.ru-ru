---
title: Ноклассдеффаундеррор-Apache Spark с данными Apache Kafka в Azure HDInsight
description: Задание потоковой передачи Apache Spark, считывающее данные из кластера Apache Kafka, завершается сбоем с Ноклассдеффаундеррор в Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 07/29/2019
ms.openlocfilehash: 4d00cbcb0151da39feb0cb015660291af544d7f4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98946386"
---
# <a name="apache-spark-streaming-job-that-reads-apache-kafka-data-fails-with-noclassdeffounderror-in-hdinsight"></a>Задание потоковой передачи Apache Spark, считывающее Apache Kafka данные, завершается сбоем с Ноклассдеффаундеррор в HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы исправления проблем, возникающих при использовании компонентов Apache Spark в кластерах Azure HDInsight.

## <a name="issue"></a>Проблема

Кластер Apache Spark запускает задание потоковой передачи Spark, которое считывает данные из кластера Apache Kafka. Задание потоковой передачи Spark завершается ошибкой, если включено сжатие потоков Kafka. В этом случае сбой application_1525986016285_0193 приложения потоковой передачи Spark Yarn из-за ошибки:

```
18/05/17 20:01:33 WARN YarnAllocator: Container marked as failed: container_e25_1525986016285_0193_01_000032 on host: wn87-Scaled.2ajnsmlgqdsutaqydyzfzii3le.cx.internal.cloudapp.net. Exit status: 50. Diagnostics: Exception from container-launch.
Container id: container_e25_1525986016285_0193_01_000032
Exit code: 50
Stack trace: ExitCodeException exitCode=50: 
 at org.apache.hadoop.util.Shell.runCommand(Shell.java:944)
```

## <a name="cause"></a>Причина

Эта ошибка может быть вызвана указанием версии `spark-streaming-kafka` JAR-файла, который отличается от версии кластера Kafka, который вы используете.

Например, если вы используете кластер Kafka версии 0.10.1, следующая команда приведет к ошибке:

```
spark-submit \
--packages org.apache.spark:spark-streaming-kafka-0-8_2.11:2.2.0
--conf spark.executor.instances=16 \
...
~/Kafka_Spark_SQL.py <bootstrap server details>
```

## <a name="resolution"></a>Решение

Используйте команду Spark-Submit с `–packages` параметром и убедитесь, что версия файла JAR-Streaming-Kafka совпадает с версией работающего кластера Kafka.

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]