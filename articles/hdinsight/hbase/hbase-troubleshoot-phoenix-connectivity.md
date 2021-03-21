---
title: Проблемы с подключением к Apache Phoenix в Azure HDInsight
description: Проблемы с подключением между Apache HBase и Apache Phoenix в Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 08/14/2019
ms.openlocfilehash: 2cc6556f681ece170bdfe02b985f56274c0faa1e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98936966"
---
# <a name="scenario-apache-phoenix-connectivity-issues-in-azure-hdinsight"></a>Сценарий: Apache Phoenix проблемы с подключением в Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы решения проблем при взаимодействии с кластерами Azure HDInsight.

## <a name="issue"></a>Проблема

Не удалось подключиться к Apache HBase с Apache Phoenix. Причины могут отличаться.

## <a name="cause-incorrect-ip"></a>Причина: неверный IP-адрес

Неправильный IP-адрес активного узла Zookeeper.

### <a name="resolution"></a>Решение

IP-адрес активного узла Zookeeper можно определить из пользовательского интерфейса Ambari, следуя ссылкам на   >  **краткие ссылки** на HBase  >  **ZK (Active)**  >  **Zookeeper info**. При необходимости исправьте IP-адрес.

---

## <a name="cause-systemcatalog-table-offline"></a>Причина: SYSTEM. Таблица каталога в автономном режиме

При выполнении таких команд `!tables` , как, появляется сообщение об ошибке следующего вида:

```output
Error while connecting to sqlline.py (Hbase - phoenix) Setting property: [isolation, TRANSACTION_READ_COMMITTED] issuing: !connect jdbc:phoenix:10.2.0.7 none none org.apache.phoenix.jdbc.PhoenixDriver Connecting to jdbc:phoenix:10.2.0.7 SLF4J: Class path contains multiple SLF4J bindings.
```

При выполнении таких команд `count 'SYSTEM.CATALOG'` , как, появляется сообщение об ошибке следующего вида:

```output
ERROR: org.apache.hadoop.hbase.NotServingRegionException: Region SYSTEM.CATALOG,,1485464083256.c0568c94033870c517ed36c45da98129. is not online on 10.2.0.5,16020,1489466172189)
```

### <a name="resolution"></a>Решение

В пользовательском интерфейсе Apache Ambari выполните следующие действия, чтобы перезапустить службу HMaster на всех узлах ZooKeeper:

1. В разделе **Сводка** по HBase перейдите в раздел **HBase**  >  **Active HBase Master**.

1. В разделе **компоненты** перезапустите службу HBase Master.

1. Повторите описанные выше шаги для остальных служб **главного узла HBase в режиме ожидания**.

Для стабилизации и завершения восстановления службы HBase Master может потребоваться до пяти минут. После `SYSTEM.CATALOG` возврата таблицы в нормальное состояние проблемы с подключением к Apache Phoenix должно быть разрешено автоматически.

## <a name="next-steps"></a>Дальнейшие действия

Если вы не видите своего варианта проблемы или вам не удается ее устранить, дополнительные сведения можно получить, посетив один из следующих каналов.

* Получите ответы специалистов Azure на [сайте поддержки сообщества пользователей Azure](https://azure.microsoft.com/support/community/).

* Подпишитесь на [@AzureSupport](https://twitter.com/azuresupport) — официальный канал Microsoft Azure для работы с клиентами. Вступайте в сообщество Azure для получения нужных ресурсов: ответов, поддержки и советов экспертов.

* Если вам нужна дополнительная помощь, отправьте запрос в службу поддержки на [портале Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Выберите **Поддержка** в строке меню или откройте центр **Справка и поддержка**. Дополнительные сведения см. в статье [Создание запроса на поддержку Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). Доступ к управлению подписками и поддержкой выставления счетов уже включен в вашу подписку Microsoft Azure, а техническая поддержка предоставляется в рамках одного из [планов Службы поддержки Azure](https://azure.microsoft.com/support/plans/).