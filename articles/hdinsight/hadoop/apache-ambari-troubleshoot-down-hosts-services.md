---
title: Пользовательский интерфейс Apache Ambari показывает узлы и службы в Azure HDInsight
description: Устранение неполадок пользовательского интерфейса Apache Ambari при отображении узлов и служб в Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 08/02/2019
ms.openlocfilehash: 0d3a393dc1cbd51b83edc0b6989bb5e7a5670a29
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98943271"
---
# <a name="scenario-apache-ambari-ui-shows-down-hosts-and-services-in-azure-hdinsight"></a>Сценарий: Пользовательский интерфейс Apache Ambari показывает узлы и службы в Azure HDInsight.

В этой статье описываются действия по устранению неполадок и возможные способы решения проблем при взаимодействии с кластерами Azure HDInsight.

## <a name="issue"></a>Проблема

Пользовательский интерфейс Apache Ambari доступен, но в пользовательском интерфейсе отображаются почти все службы, но все узлы, отображающие пульс, теряются.

## <a name="cause"></a>Причина

В большинстве случаев это проблема с Ambari Server, который не работает на активном головного узла. Проверьте, какая головного узла является активной головного узла, и убедитесь, что ambari-сервер работает справа. Не запускайте ambari-Server вручную. Служба отказоустойчивого контроллера должна отвечать за запуск ambari-Server в правой головного узла. Перезагрузите активный головного узла, чтобы принудительно выполнить отработку отказа.

Эта проблема также может быть вызвана неполадками в сети. На каждом узле кластера см. раздел Проверка возможности проверки связи `headnodehost` . Существует редкий случай, когда ни один из узлов кластера не может подключиться к `headnodehost` :

```
$>telnet headnodehost 8440
... No route to host
```

## <a name="resolution"></a>Решение

Обычно перезагрузка Active головного узла позволит устранить эту ошибку. Если вы не можете обратиться в службу поддержки HDInsight.

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]