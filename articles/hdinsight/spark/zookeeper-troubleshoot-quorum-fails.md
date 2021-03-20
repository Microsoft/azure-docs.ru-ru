---
title: Серверу Apache ZooKeeper не удается сформировать кворум в Azure HDInsight
description: Серверу Apache ZooKeeper не удается сформировать кворум в Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 05/20/2020
ms.openlocfilehash: 3301d00dce6feb00edcb70ba9edfedcce2e31ec9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98929217"
---
# <a name="apache-zookeeper-server-fails-to-form-a-quorum-in-azure-hdinsight"></a>Серверу Apache ZooKeeper не удается сформировать кворум в Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы решения проблем, связанных с Zookeeper в кластерах Azure HDInsight.

## <a name="symptoms"></a>Симптомы

* Оба диспетчера ресурсов переходят в режим ожидания
* Узлы имен находятся в режиме ожидания
* Не удается выполнить задания Spark, Hive и Yarn или запросы Hive из-за сбоев подключения Zookeeper
* Управляющие программы LLAP не запускаются в защищенных кластерах Spark или безопасных интерактивных кластерах Hive

## <a name="sample-log"></a>Пример журнала

В журналах Yarn (/Вар/лог/хадуп-ярн/ярн/ярн-ярн *. log на головных узлах) может появиться сообщение об ошибке следующего вида:

```output
2020-05-05 03:17:18.3916720|Lost contact with Zookeeper. Transitioning to standby in 10000 ms if connection is not reestablished.
Message
2020-05-05 03:17:07.7924490|Received RMFatalEvent of type STATE_STORE_FENCED, caused by org.apache.zookeeper.KeeperException$NoAuthException: KeeperErrorCode = NoAuth
...
2020-05-05 03:17:08.3890350|State store operation failed 
2020-05-05 03:17:08.3890350|Transitioning to standby state
```

## <a name="related-issues"></a>Связанные проблемы

* Существует множество причин, по которым службы высокого уровня доступности, такие как Yarn, NameNode и Livy, могут выйти из строя.
* Обратитесь к журналам, связанным с подключениями Zookeeper
* Убедитесь, что эта ошибка возникает систематически (не используйте эти решения для однократно возникающих проблем).
* Задания могут временно завершаться со сбоем из-за проблем с подключением Zookeeper

## <a name="common-causes-for-zookeeper-failure"></a>Распространенные причины сбоев Zookeeper

* Высокая загрузка ЦП на серверах Zookeeper
  * Если в пользовательском интерфейсе Ambari отображается, что ресурсы ЦП на серверах Zookeeper длительное время используются почти на 100 %, то сеансы Zookeeper, открытые в течение этого времени, могут истечь.
* Клиенты Zookeeper, которые сообщают о частом истечении времени ожидания
  * В журналах диспетчера ресурсов, узла имен и других компонентов будут содержаться сведения о частом окончании времени ожидания подключения клиента.
  * Это может привести к потере кворума, частой отработке отказа и другим проблемам.

## <a name="check-for-zookeeper-status"></a>Проверка состояния Zookeeper

* Найдите серверы Zookeeper в файле /etc/hosts или в пользовательском интерфейсе Ambari.
* Выполните следующую команду.
  * `echo stat | nc <ZOOKEEPER_HOST_IP> 2181` (или 2182)  
  * Порт 2181 является экземпляром Apache Zookeeper.
  * Порт 2182 используется службой HDInsight Zookeeper (для обеспечения высокой доступности служб, которые не имеют собственного уровня высокой доступности).
  * Если команда не отображает выходные данные, это означает, что серверы Zookeeper не работают.
  * Если серверы работают, в результате будут содержаться статические подключения клиентов и другие статистические данные.

```output
Zookeeper version: 3.4.6-8--1, built on 12/05/2019 12:55 GMT
Clients:
 /10.2.0.57:50988[1](queued=0,recved=715,sent=715)
 /10.2.0.57:46632[1](queued=0,recved=138340,sent=138347)
 /10.2.0.34:14688[1](queued=0,recved=264653,sent=353420)
 /10.2.0.52:49680[1](queued=0,recved=134812,sent=134814)
 /10.2.0.57:50614[1](queued=0,recved=19812,sent=19812)
 /10.2.0.56:35034[1](queued=0,recved=2586,sent=2586)
 /10.2.0.52:63982[1](queued=0,recved=72215,sent=72217)
 /10.2.0.57:53024[1](queued=0,recved=19805,sent=19805)
 /10.2.0.57:45126[1](queued=0,recved=19621,sent=19621)
 /10.2.0.56:41270[1](queued=0,recved=1348743,sent=1348788)
 /10.2.0.53:59097[1](queued=0,recved=72215,sent=72217)
 /10.2.0.56:41088[1](queued=0,recved=788,sent=802)
 /10.2.0.34:10246[1](queued=0,recved=19575,sent=19575)
 /10.2.0.56:40944[1](queued=0,recved=717,sent=717)
 /10.2.0.57:45466[1](queued=0,recved=19861,sent=19861)
 /10.2.0.57:59634[0](queued=0,recved=1,sent=0)
 /10.2.0.34:14704[1](queued=0,recved=264622,sent=353355)
 /10.2.0.57:42244[1](queued=0,recved=49245,sent=49248)

Latency min/avg/max: 0/3/14865
Received: 238606078
Sent: 239139381
Connections: 18
Outstanding: 0
Zxid: 0x1004f99be
Mode: follower
Node count: 133212
```

## <a name="cpu-load-peaks-up-every-hour"></a>Пиковая загрузка ЦП наблюдается каждый час.

* Войдите на сервер Zookeeper и проверьте файл /etc/crontab.
* Если в данный момент выполняются какие-либо ежечасные задания, следует задать время начала для разных серверов Zookeeper.
  
## <a name="purging-old-snapshots"></a>Удаление старых моментальных снимков

* Zookeeper настроен на автоматическую очистку старых моментальных снимков.
* По умолчанию сохраняются последние 30 моментальных снимков.
* Для управления количеством хранящихся моментальных снимков используется ключ конфигурации `autopurge.snapRetainCount`. Это свойство находится в следующих файлах:
  * `/etc/zookeeper/conf/zoo.cfg` для Hadoop Zookeeper;
  * `/etc/hdinsight-zookeeper/conf/zoo.cfg` для HDInsight Zookeeper.
* Задайте для параметра `autopurge.snapRetainCount` значение 3 и перезапустите серверы Zookeeper.
  * Обновить конфигурацию Hadoop Zookeeper и перезапустить службу можно с помощью Ambari.
  * Остановите работу и перезапустите HDInsight Zookeeper вручную.
    * `sudo lsof -i :2182` предоставит идентификатор процесса для завершения.
    * `sudo python /opt/startup_scripts/startup_hdinsight_zookeeper.py`
* Не удаляйте моментальные снимки вручную, поскольку это может привести к утере данных.

## <a name="cancelledkeyexception-in-the-zookeeper-server-log-doesnt-require-snapshot-cleanup"></a>CancelledKeyException в журнале сервера Zookeeper не требует удаления моментальных снимков.

* Это исключение будет отображаться на серверах Zookeeper (файлы/Вар/лог/зукипер/зукипер-зукипер-* или/Вар/лог/хдинсигхт-зукипер/зукипер *)
* Это исключение обычно означает, что клиент больше не активен и серверу не удается отправить сообщение.
* Это исключение также указывает, что клиент Zookeeper завершает сеансы преждевременно.
* Определите, имеются ли другие симптомы, описанные в этом документе.

## <a name="next-steps"></a>Дальнейшие действия

Если вы не видите своего варианта проблемы или вам не удается ее устранить, дополнительные сведения можно получить, посетив один из следующих каналов.

- Получите ответы специалистов Azure на [сайте поддержки сообщества пользователей Azure](https://azure.microsoft.com/support/community/).
- Подпишитесь на [@AzureSupport](https://twitter.com/azuresupport) — официальный канал Microsoft Azure для работы с клиентами. Вступайте в сообщество Azure для получения нужных ресурсов: ответов, поддержки и советов экспертов.
- Если вам нужна дополнительная помощь, отправьте запрос в службу поддержки на [портале Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Выберите **Поддержка** в строке меню или откройте центр **Справка и поддержка**. Дополнительные сведения см. в статье [Создание запроса на поддержку Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). Доступ к управлению подписками и поддержкой выставления счетов уже включен в вашу подписку Microsoft Azure, а техническая поддержка предоставляется в рамках одного из [планов Службы поддержки Azure](https://azure.microsoft.com/support/plans/).