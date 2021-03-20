---
title: Оптимизация заданий Spark для повышения производительности в Azure HDInsight
description: В этой статье описываются распространенные стратегии повышения производительности кластеров Apache Spark в Azure HDInsight.
ms.service: hdinsight
ms.topic: conceptual
ms.date: 08/21/2020
ms.custom: contperf-fy21q1
ms.openlocfilehash: 567fc2637538408d9727d07d3185a5b0e3cf20c5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98929937"
---
# <a name="optimize-apache-spark-jobs-in-hdinsight"></a>Оптимизация заданий Apache Spark в Azure HDInsight

В этой статье приводятся общие сведения о стратегиях оптимизации заданий Apache Spark в Azure HDInsight.

## <a name="overview"></a>Обзор

Производительность заданий Apache Spark зависит от нескольких факторов. К этим факторам относятся: способ хранения данных, способ настройки кластера и операции, используемые при обработке данных.

Распространенные проблемы, с которыми вы можете столкнуться: ограничения памяти из-за неправильного размера исполнителей, длительных операций и задач, которые привели к появлению операций Декарт.

Также существует множество оптимизаций, которые могут помочь преодолеть такие трудности, как кэширование, и разрешить неравномерное распределение данных.

В каждой из следующих статей можно найти сведения о различных аспектах оптимизации Spark.

* [Оптимизация хранилища данных для Apache Spark](optimize-data-storage.md)
* [Оптимизация обработки данных для Apache Spark](optimize-data-processing.md)
* [Оптимизация потребления памяти для Apache Spark](optimize-memory-usage.md)
* [Оптимизация конфигурации кластера HDInsight для Apache Spark](optimize-cluster-configuration.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Отладка заданий Apache Spark в Azure HDInsight](apache-spark-job-debugging.md)
* [Управление ресурсами для кластера Apache Spark в HDInsight](apache-spark-resource-manager.md)
* [Настройка параметров Apache Spark](apache-spark-settings.md)
