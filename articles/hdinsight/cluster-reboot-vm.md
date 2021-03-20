---
title: Перезагрузка виртуальных машин для кластеров Azure HDInsight
description: Узнайте, как перезагружать неотвечающие виртуальные машины для кластеров Azure HDInsight.
ms.custom: hdinsightactive
ms.service: hdinsight
ms.topic: how-to
ms.date: 06/22/2020
ms.openlocfilehash: a61735dba60860459d007eb54d4655f41d5ae51a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98946863"
---
# <a name="reboot-vms-for-hdinsight-clusters"></a>Перезагрузка виртуальных машин для кластеров HDInsight

Кластеры Azure HDInsight содержат группы виртуальных машин (ВМ) в качестве узлов кластера. Для длительных кластеров эти узлы могут перестать отвечать по различным причинам. В этой статье описывается, как перезагружать неотвечающие виртуальные машины в кластере HDInsight.

## <a name="when-to-reboot"></a>Время перезагрузки

> [!WARNING]
> При перезагрузке виртуальных машин в кластере узел недоступен для использования, и службы на узле должны перезапускаться.

При перезагрузке узла кластер может стать неработоспособным, а задания могут замедлить работу или завершиться сбоем. Если вы пытаетесь перезагрузить активный головной узел, все выполняющиеся задания будут остановлены. Вы не сможете отправлять задания в кластер, пока службы не будут снова запускаться. По этим причинам следует перезагружать виртуальные машины только при необходимости. Рассмотрите возможность перезагрузки виртуальных машин в следующих случаях:

- Вы не можете использовать SSH для получения узла, но он отвечает на запросы проверки связи.
- Рабочий узел не работает без пульса в пользовательском интерфейсе Ambari.
- Диск TEMP заполнен на узле.
- В таблице процессов на виртуальной машине содержится много записей, где процесс был завершен, но он отображается с состоянием "завершено".

> [!NOTE]
> Перезагрузка виртуальных машин не поддерживается для кластеров **HBase** и **Kafka** , так как перезагрузка может привести к потере данных.

## <a name="use-powershell-to-reboot-vms"></a>Перезагрузка виртуальных машин с помощью PowerShell

Для использования операции перезагрузки узла требуется два шага: список узлов и перезагрузка узлов.

1. Список узлов. Список узлов кластера можно получить в [Get-аздинсигхсост](/powershell/module/az.hdinsight/get-azhdinsighthost).

      ```
      Get-AzHDInsightHost -ClusterName myclustername
      ```

1. Перезапустите узлы. После получения имен узлов, которые требуется перезагрузить, перезапустите узлы с помощью [restart-аздинсигхсост](/powershell/module/az.hdinsight/restart-azhdinsighthost).

      ```
      Restart-AzHDInsightHost -ClusterName myclustername -Name wn0-myclus, wn1-myclus
      ```

## <a name="use-a-rest-api-to-reboot-vms"></a>Использование REST API для перезагрузки виртуальных машин

Для отправки запросов в HDInsight можно использовать функцию **попробовать** в документации по API. Для использования операции перезагрузки узла требуется два шага: список узлов и перезагрузка узлов.

1. Список узлов. Список узлов кластера можно получить из REST API или в Ambari. Дополнительные сведения см. в разделе [список HDInsight hosts REST API Operation](/rest/api/hdinsight/virtualmachines/listhosts).

    ```
    POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.HDInsight/clusters/{clusterName}/listHosts?api-version=2018-06-01-preview
    ```

1. Перезапустите узлы. После получения имен узлов, которые требуется перезагрузить, перезапустите узлы, используя REST API для перезагрузки узлов. Имя узла соответствует шаблону *NodeType (WN/HN/ZK/GW)*  +  *x*  +  *первых шести символов имени кластера*. Дополнительные сведения см. в статье [Перезагрузка узлов в HDInsight REST API операция](/rest/api/hdinsight/virtualmachines/restarthosts).

    ```
    POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.HDInsight/clusters/{clusterName}/restartHosts?api-version=2018-06-01-preview
    ```

Фактические имена узлов, которые требуется перезагружать, задаются в массиве JSON в тексте запроса.

```json
[
  "wn0-abcdef",
  "zk1-abcdef"
]
```

## <a name="next-steps"></a>Дальнейшие действия

* [Restart-Аздинсигхсост](/powershell/module/az.hdinsight/restart-azhdinsighthost)
* [Виртуальные машины HDInsight REST API](/rest/api/hdinsight/virtualmachines)
* [REST API HDInsight](/rest/api/hdinsight/)