---
title: Проблемы с пульсом Apache Ambari в Azure HDInsight
description: Анализ различных причин проблем пульса Apache Ambari в Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 02/06/2020
ms.openlocfilehash: 5eebde42098d74f533565d274b693c4a06f2f60d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98946742"
---
# <a name="apache-ambari-heartbeat-issues-in-azure-hdinsight"></a>Проблемы с пульсом Apache Ambari в Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы решения проблем при взаимодействии с кластерами Azure HDInsight.

## <a name="scenario-high-cpu-utilization"></a>Сценарий: высокая загрузка ЦП

### <a name="issue"></a>Проблема

Агент Ambari использует высокую загрузку ЦП, что приводит к предупреждениям пользовательского интерфейса Ambari, который для некоторых узлов теряет пульс агента Ambari. Предупреждение о потере пульса обычно является временным.

### <a name="cause"></a>Причина

Из-за различных ошибок ambari-Agent в редких случаях ambari-Agent может иметь высокий уровень использования ЦП (близко к 100).

### <a name="resolution"></a>Решение

1. Идентификация идентификатора процесса (PID) ambari-Agent:

    ```bash
    ps -ef | grep ambari_agent
    ```

1. Чтобы просмотреть сведения об использовании ЦП, выполните следующую команду:

    ```bash
    top -p <ambari-agent-pid>
    ```

1. Перезапустите ambari-Agent, чтобы устранить проблемы.

    ```bash
    service ambari-agent restart
    ```

1. Если перезапуск не работает, завершите процесс ambari-Agent, а затем запустите его:

    ```bash
    kill -9 <ambari-agent-pid>
    service ambari-agent start
    ```

---

## <a name="scenario-ambari-agent-not-started"></a>Сценарий: агент Ambari не запущен

### <a name="issue"></a>Проблема

Агент Ambari не начал работу, что приводит к предупреждениям из пользовательского интерфейса Ambari, который для некоторых узлов теряет пульс агента Ambari.

### <a name="cause"></a>Причина

Предупреждения вызваны тем, что агент Ambari не работает.

### <a name="resolution"></a>Решение

1. Подтвердите состояние ambari-Agent:

    ```bash
    service ambari-agent status
    ```

1. Проверьте, выполняются ли службы отказоустойчивого контроллера:

    ```bash
    ps -ef | grep failover
    ```

    Если службы контроллера отработки отказа не запущены, скорее всего, это связано с тем, что служба hdinsight-Agent не может запустить контроллер отработки отказа. Проверьте журнал hdinsight-Agent из `/var/log/hdinsight-agent/hdinsight-agent.out` файла.

## <a name="scenario-heartbeat-lost-for-ambari"></a>Сценарий: потеря пульса для Ambari

### <a name="issue"></a>Проблема

Агент пульса Ambari был потерян.

### <a name="cause"></a>Причина

Журналы OMS вызывают высокую загрузку ЦП.

### <a name="resolution"></a>Решение

* Отключите ведение журнала Azure Monitor с помощью командлета PowerShell [Disable-аздинсигхтмониторинг](/powershell/module/az.hdinsight/disable-azhdinsightmonitoring) .
* Удаление `mdsd.warn` файла журнала

---

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]