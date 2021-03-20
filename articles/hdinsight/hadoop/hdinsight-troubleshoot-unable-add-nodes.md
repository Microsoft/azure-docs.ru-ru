---
title: Не удалось добавить узлы в кластер Azure HDInsight
description: Устранение неполадок, почему не удается добавить узлы в кластер Apache Hadoop в Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 07/31/2019
ms.openlocfilehash: b11d1edef2f3a6fa0fb39c76d1f25ec05ff15d07
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98944327"
---
# <a name="scenario-unable-to-add-nodes-to-azure-hdinsight-cluster"></a>Сценарий: не удается добавить узлы в кластер Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы решения проблем при взаимодействии с кластерами Azure HDInsight.

## <a name="issue"></a>Проблема

Не удалось добавить узлы в кластер Azure HDInsight.

## <a name="cause"></a>Причина

Причины могут отличаться.

## <a name="resolution"></a>Решение

Используя функцию [Размер кластера](../hdinsight-scaling-best-practices.md) , вычислите количество дополнительных ядер, необходимых для кластера. Количество зависит от общего количества ядер в новых рабочих узлах. Затем выполните одно или несколько из следующих действий.

* Проверьте наличие доступных ядер в расположении кластера.

* Узнайте, сколько ядер доступно в других расположениях. Оцените целесообразность повторного создания кластера в другом расположении с достаточным количеством доступных ядер.

* Если вы хотите увеличить количество ядер в определенном расположении, отправьте запрос в службу поддержки на увеличение квоты для HDInsight.

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]