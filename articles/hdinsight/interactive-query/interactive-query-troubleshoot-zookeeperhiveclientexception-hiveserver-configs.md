---
title: Ошибка интерпретатора Apache Hive Zeppelin — Azure HDInsight
description: Интерпретатор JDBC Zeppelin Hive Apache указывает на неправильный URL-адрес в Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 07/30/2019
ms.openlocfilehash: 2d6e660459d9f350cc5c63fb8d312bcbcb300a64
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98930754"
---
# <a name="scenario-apache-hive-zeppelin-interpreter-gives-a-zookeeper-error-in-azure-hdinsight"></a>Сценарий: Apache Hive интерпретатор Zeppelin дает ошибку Zookeeper в Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы решения проблем при использовании интерактивных компонентов запросов в кластерах Azure HDInsight.

## <a name="issue"></a>Проблема

В кластере Apache Hive LLAP интерпретатор Zeppelin выдает следующее сообщение об ошибке при попытке выполнить запрос:

```
java.sql.SQLException: org.apache.hive.jdbc.ZooKeeperHiveClientException: Unable to read HiveServer2 configs from ZooKeeper
```

## <a name="cause"></a>Причина

Интерпретатор JDBC Zeppelin Hive указывает на неверный URL-адрес.

## <a name="resolution"></a>Решение

1. Перейдите к сводке по компоненту Hive и скопируйте в буфер обмена URL-адрес Hive JDBC.

1. Перейдите на страницу `https://clustername.azurehdinsight.net/zeppelin/#/interpreter`.

1. Измените параметры JDBC: измените значение Hive. URL на URL-адрес JDBC для Hive, скопированный на шаге 1.

1. Сохраните, а затем повторите запрос

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]