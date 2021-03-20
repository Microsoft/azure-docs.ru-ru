---
title: Низкая производительность в Apache Hive запросов LLAP в Azure HDInsight
description: Запросы в Apache Hive LLAP выполняются медленнее, чем ожидалось в Azure HDInsight.
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 07/30/2019
ms.openlocfilehash: 80fe51c2131da5a6fcb69eb5526b5d4f33a45e03
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98930907"
---
# <a name="scenario-poor-performance-in-apache-hive-llap-queries-in-azure-hdinsight"></a>Сценарий: низкая производительность в Apache Hive запросов LLAP в Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы решения проблем при использовании интерактивных компонентов запросов в кластерах Azure HDInsight.

## <a name="issue"></a>Проблема

Конфигурации кластера по умолчанию недостаточно настроены для рабочей нагрузки. Запросы в Hive LLAP выполняются медленнее, чем ожидалось.

## <a name="cause"></a>Причина

Это может быть вызвано разными причинами.

## <a name="resolution"></a>Решение

LLAP оптимизирован для запросов, использующих операции объединения и агрегирования. Запросы, подобные приведенным ниже, не работают в кластере интерактивных Hive.

```
select * from table where column = "columnvalue"
```

Чтобы улучшить производительность запросов к точкам в Hive LLAP, задайте следующие конфигурации:

```
hive.llap.io.enabled=false; (disable LLAP IO)
hive.optimize.index.filter=false; (disable ORC row index)
hive.exec.orc.split.strategy=BI; (to avoid recombining splits)
```

Кроме того, можно увеличить использование кэша LLAP, чтобы повысить производительность с помощью следующих изменений конфигурации:

```
hive.fetch.task.conversion=none
```

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]