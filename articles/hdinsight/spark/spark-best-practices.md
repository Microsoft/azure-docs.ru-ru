---
title: Рекомендации по Apache Spark в Azure HDInsight
description: Ознакомьтесь с рекомендациями по использованию Apache Spark в Azure HDInsight.
ms.service: hdinsight
ms.topic: conceptual
ms.date: 04/28/2020
ms.openlocfilehash: f81435abbedec25cb8abe0b958065a493c15ce4c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98938815"
---
# <a name="apache-spark-guidelines"></a>Рекомендации по Apache Spark

В этой статье приводятся различные рекомендации по использованию Apache Spark в Azure HDInsight.

## <a name="how-do-i-run-or-submit-spark-jobs"></a>Разделы справки запускать или отправлять задания Spark?

| Параметр | Документы |
|---|---|
| VSCode | [Использование средств Spark и Hive для Visual Studio Code](../hdinsight-for-vscode.md) |
| Jupyter Notebook | [Руководство по Загрузка данных и выполнение запросов в кластере Apache Spark в Azure HDInsight](./apache-spark-load-data-run-query.md) |
| IntelliJ | [Учебник. Использование Azure Toolkit for IntelliJ для создания Apache Spark приложений для кластера HDInsight](./apache-spark-intellij-tool-plugin.md) |
| IntelliJ | [Руководство по Создание приложения Scala Maven для Apache Spark в HDInsight с помощью IntelliJ](./apache-spark-create-standalone-application.md) |
| Записные книжки Zeppelin | [Use Apache Zeppelin notebooks with Apache Spark cluster on Azure HDInsight](./apache-spark-zeppelin-notebook.md) (Использование записных книжек Apache Zeppelin с кластером Apache Spark в Azure HDInsight) |
| Отправка удаленных заданий с помощью Livy | [Удаленная отправка заданий Spark в кластер Azure HDInsight с помощью Apache Spark REST API](./apache-spark-livy-rest-interface.md) |
|[Apache Oozie](../hdinsight-use-oozie-linux-mac.md)|Oozie — это система рабочих процессов и координации, управляющая заданиями Hadoop.|
|[Apache Livy](./apache-spark-livy-rest-interface.md)|Вы можете использовать Livy для выполнения интерактивных оболочек Spark или отправки пакетных заданий для запуска в кластере Spark.|
|[Фабрика данных Azure для Apache Spark](../../data-factory/transform-data-using-spark.md)|Действие Spark в конвейере фабрики данных выполняет программу Spark в собственном или [кластере HDInsight по запросу.|
|[Фабрика данных Azure для Apache Hive](../../data-factory/transform-data-using-hadoop-hive.md)|Действие Hive в HDInsight в конвейере фабрики данных выполняет запросы Hive к собственному кластеру HDInsight или по запросу.|

## <a name="how-do-i-monitor-and-debug-spark-jobs"></a>Разделы справки отслеживать задания Spark и выполнять их отладку?

| Параметр | Документы |
|---|---|
| Набор средств Azure для IntelliJ | [Сбой отладки задания Spark с Azure Toolkit for IntelliJ (Предварительная версия)](apache-spark-intellij-tool-failure-debug.md) |
| Azure Toolkit for IntelliJ через SSH | [Удаленная или локальная отладка приложений Apache Spark в кластере HDInsight с помощью набора Azure Toolkit for IntelliJ через SSH](apache-spark-intellij-tool-debug-remotely-through-ssh.md) |
| Azure Toolkit for IntelliJ через VPN | [Удаленная отладка приложений Apache Spark в HDInsight через VPN с помощью Azure Toolkit for IntelliJ](apache-spark-intellij-tool-plugin-debug-jobs-remotely.md) |
| Граф заданий на Apache Spark сервере журнала | [Отладка и диагностика приложений Apache Spark с использованием расширенного сервера журнала Apache Spark](./apache-azure-spark-history-server.md) |

## <a name="how-do-i-make-my-spark-jobs-run-more-efficiently"></a>Разделы справки сделать мои задания Spark более эффективными?

| Параметр | Документы |
|---|---|
| Кэш ввода-вывода | [Повышение производительности рабочих нагрузок Apache Spark с помощью службы IO Cache для Azure HDInsight (предварительная версия)](./apache-spark-improve-performance-iocache.md) |
| Варианты настройки | [Оптимизация заданий Apache Spark](./apache-spark-perf.md) |

## <a name="how-do-i-connect-to-other-azure-services"></a>Разделы справки подключиться к другим службам Azure?

| Параметр | Документы |
|---|---|
| Apache Hive в HDInsight | [Integrate Apache Spark and Apache Hive with the Hive Warehouse Connector](../interactive-query/apache-hive-warehouse-connector.md) (Интеграция Apache Spark и Apache Hive с помощью соединителя хранилища Hive) |
| Apache HBase в HDInsight | [Чтение и запись данных Apache HBase с помощью Apache Spark](../hdinsight-using-spark-query-hbase.md) |
| Apache Kafka в HDInsight | [Руководство по Использование структурированной потоковой передачи Apache Spark с Apache Kafka в HDInsight](../hdinsight-apache-kafka-spark-structured-streaming.md) |
| Azure Cosmos DB | [Сведения об Azure Synapse Link для Azure Cosmos DB](../../cosmos-db/synapse-link.md) |

## <a name="what-are-my-storage-options"></a>Что такое мои параметры хранения?

| Параметр | Документы |
|---|---|
| Azure Data Lake Storage 2-го поколения | [Использование Azure Data Lake Storage Gen2 с кластерами Azure HDInsight](../hdinsight-hadoop-use-data-lake-storage-gen2.md) |
| Хранилище Azure Data Lake Storage 1-го поколения | [Использование Azure Data Lake Storage 1-го поколения с кластерами Azure HDInsight](../hdinsight-hadoop-use-data-lake-storage-gen1.md) |
| хранилище BLOB-объектов Azure | [Использование службы хранилища Azure с кластерами Azure HDInsight](../hdinsight-hadoop-use-blob-storage.md) |

## <a name="next-steps"></a>Дальнейшие действия

* [Настройка параметров Apache Spark](apache-spark-settings.md)
* [Оптимизация заданий Apache Spark в Azure HDInsight](apache-spark-perf.md)
