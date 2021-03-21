---
title: Ошибка Иллегаларгументексцептион для Apache Spark — Azure HDInsight
description: Иллегаларгументексцептион для действия Apache Spark в Azure HDInsight для фабрики данных Azure
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 07/29/2019
ms.openlocfilehash: 429659d605cdaf8aad978841e486a17da321cce4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98929402"
---
# <a name="scenario-illegalargumentexception-for-apache-spark-activity-in-azure-hdinsight"></a>Сценарий: Иллегаларгументексцептион для действия Apache Spark в Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы исправления проблем, возникающих при использовании компонентов Apache Spark в кластерах Azure HDInsight.

## <a name="issue"></a>Проблема

При попытке выполнить действие Spark в конвейере фабрики данных Azure появляется следующее исключение:

```error
Exception in thread "main" java.lang.IllegalArgumentException:
Wrong FS: wasbs://additional@xxx.blob.core.windows.net/spark-examples_2.11-2.1.0.jar, expected: wasbs://wasbsrepro-2017-11-07t00-59-42-722z@xxx.blob.core.windows.net
```

## <a name="cause"></a>Причина

Задание Spark завершится ошибкой, если JAR-файл приложения не размещается в хранилище по умолчанию (первичном) кластера Spark.

Это известная проблема с инфраструктурой открытого кода Spark, проданной в этой ошибке: [Задание Spark завершается ошибкой, если в FS. дефаултфс и JAR-файл хранятся разные URL](https://issues.apache.org/jira/browse/SPARK-22587).

Эта проблема решена в Spark 2.3.0.

## <a name="resolution"></a>Решение

Убедитесь, что JAR-файл приложения хранится в хранилище по умолчанию (первичном) для кластера HDInsight. В случае фабрики данных Azure убедитесь, что связанная служба ADF указывает на контейнер HDInsight по умолчанию, а не на дополнительный контейнер.

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]