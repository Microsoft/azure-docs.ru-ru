---
title: Оповещения каталога Apache Ambari в Azure HDInsight
description: Обсуждение и анализ возможных причин и решений для оповещений каталога Apache Ambari в HDInsight.
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 01/22/2020
ms.openlocfilehash: 47d1f9797a44d7dc918677c21ffc7a124808525d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98943281"
---
# <a name="scenario-apache-ambari-directory-alerts-in-azure-hdinsight"></a>Сценарий: оповещения каталога Apache Ambari в Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы решения проблем при взаимодействии с кластерами Azure HDInsight.

## <a name="issue"></a>Проблема

Вы получаете ошибки Apache Ambari, аналогичные следующим:

```
1/1 local-dirs have errors: [ /mnt/resource/hadoop/yarn/local : Cannot create directory: /mnt/resource/hadoop/yarn/local ]
1/1 log-dirs have errors: [ /mnt/resource/hadoop/yarn/log : Cannot create directory: /mnt/resource/hadoop/yarn/log ]
```

## <a name="cause"></a>Причина

Указанные каталоги из предупреждения Ambari отсутствуют на затронутых рабочих узлах.

## <a name="resolution"></a>Решение

Вручную создайте отсутствующие каталоги на затронутых рабочих узлах.

1. Подключитесь к соответствующему рабочему узлу по протоколу SSH.

1. Получить привилегированного пользователя: `sudo su` .

1. Рекурсивно создайте необходимые каталоги.

1. Измените владельца и группу для этих каталогов.

    ```bash
    chown -R yarn /mnt/resource/hadoop/yarn/local
    chgrp -R hadoop /mnt/resource/hadoop/yarn/local
    chown -R yarn /mnt/resource/hadoop/yarn/log
    chgrp -R hadoop /mnt/resource/hadoop/yarn/log
    ```

1. В пользовательском интерфейсе Apache Ambari отключите, а затем включите оповещение.

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]