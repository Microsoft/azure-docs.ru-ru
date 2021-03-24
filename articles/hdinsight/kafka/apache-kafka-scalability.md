---
title: Повышение масштабируемости Apache Kafka в Azure HDInsight
description: Узнайте, как настроить управляемые диски для кластера Apache Kafka в Azure HDInsight, чтобы повысить масштабируемость.
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive
ms.date: 12/09/2019
ms.openlocfilehash: 9aa11be42aca59458fea0462a90b6aeb70df893d
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104863145"
---
# <a name="configure-storage-and-scalability-for-apache-kafka-on-hdinsight"></a>Настройка объема хранилища и уровня масштабируемости для Apache Kafka в HDInsight

Узнайте, как настроить число управляемых дисков, используемых [Apache Kafka](https://kafka.apache.org/) в HDInsight.

Kafka в HDInsight использует локальный диск для виртуальных машин в кластере HDInsight. Так как Kafka обрабатывает большое количество операций ввода-вывода, используются [управляемые диски Azure](../../virtual-machines/managed-disks-overview.md), чтобы обеспечить высокую пропускную способность и предоставить дополнительное хранилище для каждого узла. Если для Kafka использовать стандартные виртуальные жесткие диски (VHD), на каждом узле будет доступен 1 ТБ памяти. Но благодаря управляемым дискам можно использовать несколько дисков и увеличить объем каждого узла кластера до 16 ТБ.

На схеме ниже сравниваются две версии Kafka в HDInsight: до использования управляемых дисков и с ними.

:::image type="content" source="./media/apache-kafka-scalability/kafka-with-managed-disks-architecture.png" alt-text="Архитектура управляемых дисков Kafka" border="false":::

## <a name="configure-managed-disks-azure-portal"></a>Настройка управляемых дисков на портале Azure

1. Следуйте указаниям в статье о [создании кластеров HDInsight](../hdinsight-hadoop-create-linux-clusters-portal.md), чтобы ознакомиться с основными действиями для создания кластера с помощью портала. Не завершайте процесс создания портала.

2. В разделе **цены на конфигурацию &** используйте поле __число узлов__ , чтобы настроить количество дисков.

    > [!NOTE]  
    > Управляемый диск может быть двух типов: __Стандартный__ (HDD) или __Премиум__ (SSD). Диски категории "Премиум" используются с виртуальными машинами серий DS и GS. Для всех остальных виртуальных машин используются стандартные управляемые диски.

    :::image type="content" source="./media/apache-kafka-scalability/azure-portal-cluster-configuration-pricing-kafka-disks.png" alt-text="раздел &quot;размер кластера&quot; с выделенным узлом &quot;диски на рабочий узел&quot;" border="true":::

## <a name="configure-managed-disks-resource-manager-template"></a>Настройка управляемых дисков с использованием шаблона Resource Manager

Чтобы управлять количеством дисков, которое используется рабочими узлами в кластере Kafka, используйте следующий раздел шаблона:

```json
"dataDisksGroups": [
    {
        "disksPerNode": "[variables('disksPerWorkerNode')]"
    }
    ],
```

Полный шаблон, демонстрирующий настройку управляемых дисков, можно найти в [https://hditutorialdata.blob.core.windows.net/armtemplates/create-linux-based-kafka-mirror-cluster-in-vnet-v2.1.json](https://hditutorialdata.blob.core.windows.net/armtemplates/create-linux-based-kafka-mirror-cluster-in-vnet-v2.1.json) .

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о работе с Apache Kafka HDInsight см. в следующих документах:

* [Репликация разделов Apache Kafka с помощью Kafka в HDInsight и MirrorMaker](apache-kafka-mirroring.md)
* [Использование Apache Storm с Apache Kafka в HDInsight](../hdinsight-apache-storm-with-kafka.md)
* [Использование Apache Spark с Apache Kafka в HDInsight](../hdinsight-apache-spark-with-kafka.md)
* [Подключение к Apache Kafka с помощью виртуальной сети Azure](apache-kafka-connect-vpn-gateway.md)

* [Блог HDInsight об управляемых дисках в Apache Kafka](https://azure.microsoft.com/blog/announcing-public-preview-of-apache-kafka-on-hdinsight-with-azure-managed-disks/)
