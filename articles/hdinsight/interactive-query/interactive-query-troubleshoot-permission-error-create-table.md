---
title: Ошибка отказа в разрешении с таблицей Apache Hive в Azure HDInsight
description: Ошибка отказа в разрешении при попытке создания таблицы Apache Hive в Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 08/09/2019
ms.openlocfilehash: 6d91ca747bb264b4c7262eec2bb0745b8a40b7a0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98930917"
---
# <a name="scenario-permission-denied-error-when-trying-to-create-an-apache-hive-table-in-azure-hdinsight"></a>Сценарий: ошибка отказа в разрешении при попытке создания таблицы Apache Hive в Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы решения проблем при использовании интерактивных компонентов запросов в кластерах Azure HDInsight.

## <a name="issue"></a>Проблема

При попытке создать таблицу вы увидите следующую ошибку:

```
java.sql.SQLException: Error while compiling statement: FAILED: HiveAccessControlException Permission denied: user [hdiuser] does not have [ALL] privilege on [wasbs://data@xxxxx.blob.core.windows.net/path/table]
```

Если запустить следующую команду хранилища HDFS, вы увидите аналогичное сообщение об ошибке:

```
hdfs dfs -mkdir wasbs://data@xxxxx.blob.core.windows.net/path/table
```

## <a name="cause"></a>Причина

Возможность создания таблицы в Apache Hive определяется разрешениями, применяемыми к учетной записи хранения кластера. Если разрешения учетной записи хранения кластера неверны, создавать таблицы будет невозможно. Это означает, что у вас могут быть правильные политики Ranger для создания таблицы и по-прежнему отображаться ошибки "отказано в разрешении".

## <a name="resolution"></a>Решение

Это вызвано отсутствием достаточных разрешений в используемом контейнере хранилища. Пользователь, создающий таблицу Hive, должен иметь разрешения на чтение, запись и выполнение для контейнера. Дополнительные сведения см. в статье рекомендации [по авторизации Hive с помощью Apache Ranger в HDP 2,2](https://hortonworks.com/blog/best-practices-for-hive-authorization-using-apache-ranger-in-hdp-2-2/).

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]