---
title: Проблемы с серверами регионов в Azure HDInsight
description: Проблемы с серверами регионов в Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 08/16/2019
ms.openlocfilehash: 968a0c6e1717245171bf84821a58cad4e440046e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98936615"
---
# <a name="issues-with-region-servers-in-azure-hdinsight"></a>Проблемы с серверами регионов в Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы решения проблем при взаимодействии с кластерами Azure HDInsight.

## <a name="scenario-unassigned-regions"></a>Сценарий: неназначенные регионы

### <a name="issue"></a>Проблема

При выполнении `hbase hbck` команды отображается сообщение об ошибке следующего вида:

```
multiple regions being unassigned or holes in the chain of regions
```

В пользовательском интерфейсе Apache HBase Master можно увидеть число регионов, которые не сбалансированы по всем серверам регионов. Затем можно выполнить команду `hbase hbck`, чтобы увидеть пропуски в цепочке регионов.

### <a name="cause"></a>Причина

Отверстия могут быть результатом автономных регионов.

### <a name="resolution"></a>Решение

Исправьте назначения. Выполните следующие действия, чтобы вернуть неназначенные регионы в нормальное состояние.

1. Войдите в кластер HDInsight HBase с помощью SSH.

1. Выполните `hbase zkcli` команду, чтобы подключиться к оболочке ZooKeeper.

1. Выполните `rmr /hbase/regions-in-transition` `rmr /hbase-unsecure/regions-in-transition` команду или.

1. Выйдите из оболочки Zookeeper с помощью `exit` команды.

1. Откройте пользовательский интерфейс Ambari Apache и перезапустите службу активного главного узла HBase.

1. Выполните `hbase hbck` команду еще раз (без дополнительных параметров). Проверьте выходные данные и убедитесь, что все регионы назначены.

---

## <a name="scenario-dead-region-servers"></a>Сценарий: неработающие серверы регионов

### <a name="issue"></a>Проблема

Не удается запустить серверы регионов.

### <a name="cause"></a>Причина

Множественное разделение каталогов WAL.

1. Получить список текущих Wal: `hadoop fs -ls -R /hbase/WALs/ > /tmp/wals.out` .

1. Проверьте `wals.out` файл. Если слишком много разделенных каталогов (начиная с *-разделение), возможно, сервер регионов не работает из-за этих каталогов.

### <a name="resolution"></a>Решение

1. Отключите HBase на портале Ambari.

1. Выполните процедуру `hadoop fs -ls -R /hbase/WALs/ > /tmp/wals.out` , чтобы получить актуальный список Wal.

1. Переместите *-разделение каталогов во временную папку `splitWAL` и удалите каталоги, разделенные на *.

1. Выполните `hbase zkcli` команду, чтобы подключиться к оболочке Zookeeper.

1. Выполните процедуру `rmr /hbase-unsecure/splitWAL`.

1. Перезапустите службу HBase.

## <a name="next-steps"></a>Дальнейшие действия

Если вы не видите своего варианта проблемы или вам не удается ее устранить, дополнительные сведения можно получить, посетив один из следующих каналов.

* Получите ответы специалистов Azure на [сайте поддержки сообщества пользователей Azure](https://azure.microsoft.com/support/community/).

* Подпишитесь на [@AzureSupport](https://twitter.com/azuresupport) — официальный канал Microsoft Azure для работы с клиентами. Вступайте в сообщество Azure для получения нужных ресурсов: ответов, поддержки и советов экспертов.

* Если вам нужна дополнительная помощь, отправьте запрос в службу поддержки на [портале Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Выберите **Поддержка** в строке меню или откройте центр **Справка и поддержка**. Дополнительные сведения см. в статье [Создание запроса на поддержку Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). Доступ к управлению подписками и поддержкой выставления счетов уже включен в вашу подписку Microsoft Azure, а техническая поддержка предоставляется в рамках одного из [планов Службы поддержки Azure](https://azure.microsoft.com/support/plans/).