---
title: 'Устранение неполадок: Apache Hive журналов заполнение дискового пространства — Azure HDInsight'
description: В этой статье приведены инструкции по устранению неполадок, которые необходимо выполнить, когда Apache Hive журналы заполняют дисковое пространство головных узлов в Azure HDInsight.
ms.service: hdinsight
ms.topic: troubleshooting
author: nisgoel
ms.author: nisgoel
ms.date: 10/05/2020
ms.openlocfilehash: cd7e6a7f13f6cccb5be5d23d69c2a44fc655cf55
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98930949"
---
# <a name="scenario-apache-hive-logs-are-filling-up-the-disk-space-on-the-head-nodes-in-azure-hdinsight"></a>Сценарий: журналы Apache Hive заполняют дисковое пространство головных узлов в Azure HDInsight.

В этой статье описываются действия по устранению неполадок и возможные способы решения проблем, связанных с нехваткой дискового пространства на головных узлах в кластерах Azure HDInsight.

## <a name="issue"></a>Проблема

В кластере Apache Hive или LLAP нежелательные журналы занимают все дисковое пространство на головных узлах. Это состояние может вызвать следующие проблемы.

- SSH-доступ завершается сбоем, так как на головном узле не осталось места.
- Ambari выдает *ошибку HTTP: служба 503 недоступна*.
- HiveServer2 Interactive не перезапускается.

`ambari-agent`При возникновении проблемы в журналах будут содержаться следующие записи:
```
ambari_agent - Controller.py - [54697] - Controller - ERROR - Error:[Errno 28] No space left on device
```
```
ambari_agent - HostCheckReportFileHandler.py - [54697] - ambari_agent.HostCheckReportFileHandler - ERROR - Can't write host check file at /var/lib/ambari-agent/data/hostcheck.result
```

## <a name="cause"></a>Причина

В расширенных конфигурациях Hive log4j текущее расписание удаления по умолчанию — это удаление файлов старше 30 дней на основе даты последнего изменения.

## <a name="resolution"></a>Решение

1. Перейдите на страницу Сводка по компоненту Hive на портале Ambari и выберите вкладку **configs (конфигурации** ).

2. Перейдите к `Advanced hive-log4j` разделу **Дополнительные параметры**.

3. Задайте `appender.RFA.strategy.action.condition.age` для параметра возраст по своему усмотрению. В этом примере для параметра Age будет задано значение 14 дней: `appender.RFA.strategy.action.condition.age = 14D`

4. Если вы не видите связанные параметры, добавьте следующие параметры:
    ```
    # automatically delete hive log
    appender.RFA.strategy.action.type = Delete
    appender.RFA.strategy.action.basePath = ${sys:hive.log.dir}
    appender.RFA.strategy.action.condition.type = IfLastModified
    appender.RFA.strategy.action.condition.age = 30D
    appender.RFA.strategy.action.PathConditions.type = IfFileName
    appender.RFA.strategy.action.PathConditions.regex = hive*.*log.*
    ```

5. Задайте `hive.root.logger` для значение `INFO,RFA` , как показано в следующем примере. Значение по умолчанию — `DEBUG` , что делает журналы большими.

    ```
    # Define some default values that can be overridden by system properties
    hive.log.threshold=ALL
    hive.root.logger=INFO,RFA
    hive.log.dir=${java.io.tmpdir}/${user.name}
    hive.log.file=hive.log
    ```

6. Сохраните настройки и перезапустите необходимые компоненты.

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]
