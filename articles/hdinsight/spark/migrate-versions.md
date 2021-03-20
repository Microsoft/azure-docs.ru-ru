---
title: Перенос рабочих нагрузок Apache Spark 2.1 или 2.2 на версию 2.3 или 2.4 (Azure HDInsight)
description: Сведения о том, как выполнить миграцию Apache Spark 2.1 или 2.2 на версию 2.3 или 2.4.
ms.service: hdinsight
ms.topic: conceptual
ms.date: 05/20/2020
ms.openlocfilehash: af1894d2f63357006e87fa8e4533f135ecc02f21
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98944756"
---
# <a name="migrate-apache-spark-21-and-22-workloads-to-23-and-24"></a>Перенос рабочих нагрузок Apache Spark 2.1 и 2.2 на версию 2.3 и 2.4

В этом документе объясняется, как перенести рабочие нагрузки Apache Spark 2.1 или 2.2 на версию 2.3 или 2.4.

Как обсуждалось в [заметках о выпуске](../hdinsight-release-notes.md#upcoming-changes), с 1 июля 2020 года прекратится поддержка следующих конфигураций кластера, и клиенты не смогут создавать новые кластеры с такими конфигурациями:
 - Spark 2.1 и 2.2 для кластера Spark в HDInsight 3.6;
 - Spark 2.3 для кластера Spark в HDInsight 4.0.

Существующие кластеры в этих конфигурациях будут продолжать работу "как есть" без поддержки со стороны корпорации Майкрософт. Если вы сейчас используете Spark 2.1 или 2.2 в HDInsight 3.6, перейдите на версию Spark 2.3 в HDInsight 3.6 до 30 июня 2020 г., чтобы избежать потенциальных проблем или перерывов в работе. Если вы сейчас используете Spark 2.3 в кластере HDInsight 4.0, перейдите на версию Spark 2.4 в HDInsight 4.0 до 30 июня 2020 г., чтобы избежать потенциальных проблем или перерывов в работе.

Общие сведения о переносе кластера HDInsight с версии 3.6 на версию 4.0 см. в статье [Перенос кластера HDInsight в более новую версию](../hdinsight-upgrade-cluster.md). Общие сведения о переходе на более новую версию Apache Spark см. в статье [ Versioning Policy](https://spark.apache.org/versioning-policy.html) (Политика выбора версий) на сайте Apache Spark.

## <a name="guidance-on-spark-version-upgrades-on-hdinsight"></a>Руководство по обновлению версий Spark в HDInsight

| Сценарий обновления | Механизм | Полезная информация | Интеграция Spark и Hive |
|------------------|-----------|--------------------|------------------------|
|С версии Spark 2.1 в HDInsight 3.6 на версию Spark 2.3 в HDInsight 3.6| Восстановление кластеров на Spark 2.3 в HDInsight | Ознакомьтесь со следующими статьями: <br> [Apache Spark: обновление с версии Spark SQL 2.2 до 2.3](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-22-to-23) <br><br> [Apache Spark: обновление с версии Spark SQL 2.1 до 2.2](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-21-to-22) | Нет изменений |
|С версии Spark 2.2 в HDInsight 3.6 на версию Spark 2.3 в HDInsight 3.6 | Восстановление кластеров на Spark 2.3 в HDInsight | Ознакомьтесь со следующими статьями: <br> [Apache Spark: обновление с версии Spark SQL 2.2 до 2.3](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-22-to-23) | Нет изменений |
| С версии Spark 2.1 в HDInsight 3.6 на версию Spark 2.4 в HDInsight 4.0 | Восстановление кластеров на Spark 2.4 в HDInsight 4.0 | Ознакомьтесь со следующими статьями: <br> [Apache Spark: обновление с версии Spark SQL 2.3 до 2.4](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-23-to-24) <br><br> [Apache Spark: обновление с версии Spark SQL 2.2 до 2.3](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-22-to-23) <br><br> [Apache Spark: обновление с версии Spark SQL 2.1 до 2.2](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-21-to-22) | Интеграция Spark и Hive была изменена в HDInsight 4,0. <br><br> В HDInsight 4.0 Spark и Hive используют независимые каталоги для доступа к таблицам SparkSQL или Hive. Таблица, созданная Spark, находится в каталоге Spark. Таблица, созданная Hive, находится в каталоге Hive. Такое поведение отличается от версии HDInsight 3.6, где Hive и Spark совместно использовали общий каталог. Интеграция Hive и Spark в HDInsight 4.0 основана на соединителе Hive Warehouse Connector (HWC). HWC работает как мост между Spark и Hive. Изучите особенности Hive Warehouse Connector. <br> В HDInsight 4.0 можно настроить совместное использование хранилища метаданных для Hive и Spark, указав для свойства хранилища metastore.catalog.default значение hive в кластере Spark. Это свойство можно найти в Ambari Advanced spark2-hive-site-override. Важно понимать, что совместное использование хранилище метаданных работает только для внешних таблиц Hive. это не будет работать при наличии внутренних или управляемых таблиц Hive или таблиц с ACID. <br><br>Дополнительные сведения см. в статье [Перенос рабочих нагрузок Hive из Azure HDInsight 3.6 в HDInsight 4.0](../interactive-query/apache-hive-migrate-workloads.md).<br><br> |
| С версии Spark 2.2 в HDInsight 3.6 на версию Spark 2.4 в HDInsight 4.0 | Восстановление кластеров на Spark 2.4 в HDInsight 4.0 | Ознакомьтесь со следующими статьями: <br> [Apache Spark: обновление с версии Spark SQL 2.3 до 2.4](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-23-to-24) <br><br> [Apache Spark: обновление с версии Spark SQL 2.2 до 2.3](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-22-to-23) | Интеграция Spark и Hive была изменена в HDInsight 4,0. <br><br> В HDInsight 4.0 Spark и Hive используют независимые каталоги для доступа к таблицам SparkSQL или Hive. Таблица, созданная Spark, находится в каталоге Spark. Таблица, созданная Hive, находится в каталоге Hive. Такое поведение отличается от версии HDInsight 3.6, где Hive и Spark совместно использовали общий каталог. Интеграция Hive и Spark в HDInsight 4.0 основана на соединителе Hive Warehouse Connector (HWC). HWC работает как мост между Spark и Hive. Изучите особенности Hive Warehouse Connector. <br> В HDInsight 4.0 можно настроить совместное использование хранилища метаданных для Hive и Spark, указав для свойства хранилища metastore.catalog.default значение hive в кластере Spark. Это свойство можно найти в Ambari Advanced spark2-hive-site-override. Важно понимать, что совместное использование хранилище метаданных работает только для внешних таблиц Hive. это не будет работать при наличии внутренних или управляемых таблиц Hive или таблиц с ACID. <br><br>Дополнительные сведения см. в статье [Перенос рабочих нагрузок Hive из Azure HDInsight 3.6 в HDInsight 4.0](../interactive-query/apache-hive-migrate-workloads.md).|

## <a name="next-steps"></a>Дальнейшие действия

* [Перенос кластера HDInsight в более новую версию](../hdinsight-upgrade-cluster.md)
* [Перенос рабочих нагрузок Hive из Azure HDInsight 3.6 в HDInsight 4.0](../interactive-query/apache-hive-migrate-workloads.md)
