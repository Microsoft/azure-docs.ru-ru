---
title: Истечение времени ожидания при выполнении команды "hbase hbck" в Azure HDInsight
description: Возникла ошибка при выполнении команды "HBase hbck" при исправлении назначений регионов.
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 08/16/2019
ms.openlocfilehash: acf4d1237841f8c20c014598e00a5e8961e2a012
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98936695"
---
# <a name="scenario-timeouts-with-hbase-hbck-command-in-azure-hdinsight"></a>Сценарий: время ожидания команды "HBase hbck" в Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы решения проблем при взаимодействии с кластерами Azure HDInsight.

## <a name="issue"></a>Проблема

`hbase hbck`При исправлении назначений регионов возникают времена ожидания с помощью команды.

## <a name="cause"></a>Причина

Потенциальной причиной проблем с временем ожидания при использовании команды `hbck` может быть то, что несколько регионов находятся в состоянии "идет переход" в течение долгого времени. Эти регионы можно увидеть как отключенные в пользовательском интерфейсе главного узла HBase. Так как большое число регионов пытается выполнить переход, HBase Master может исключаться из-за истечения времени ожидания и не сможет вернуть эти регионы в оперативный режим.

## <a name="resolution"></a>Решение

1. Войдите в кластер HDInsight HBase с помощью SSH.

1. Выполните `hbase zkcli` команду, чтобы подключиться с помощью оболочки Apache ZooKeeper.

1. Выполните `rmr /hbase/regions-in-transition` `rmr /hbase-unsecure/regions-in-transition` команду или.

1. Выйдите из `hbase zkcli` оболочки с помощью `exit` команды.

1. В пользовательском интерфейсе Apache Ambari перезапустите активную службу HBase Master.

1. Выполните команду `hbase hbck -fixAssignments`.

1. Отслеживайте HBase Master пользовательского интерфейса в области переходов, чтобы не зависнуть регион.

## <a name="next-steps"></a>Дальнейшие действия

Если вы не видите своего варианта проблемы или вам не удается ее устранить, дополнительные сведения можно получить, посетив один из следующих каналов.

- Получите ответы специалистов Azure на [сайте поддержки сообщества пользователей Azure](https://azure.microsoft.com/support/community/).

- Подпишитесь на [@AzureSupport](https://twitter.com/azuresupport) — официальный канал Microsoft Azure для работы с клиентами. Вступайте в сообщество Azure для получения нужных ресурсов: ответов, поддержки и советов экспертов.

- Если вам нужна дополнительная помощь, отправьте запрос в службу поддержки на [портале Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Выберите **Поддержка** в строке меню или откройте центр **Справка и поддержка**. Дополнительные сведения см. в статье [Создание запроса на поддержку Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). Доступ к управлению подписками и поддержкой выставления счетов уже включен в вашу подписку Microsoft Azure, а техническая поддержка предоставляется в рамках одного из [планов Службы поддержки Azure](https://azure.microsoft.com/support/plans/).