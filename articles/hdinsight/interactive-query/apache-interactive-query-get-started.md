---
title: Общие сведения об интерактивном запросе в Azure HDInsight
description: Общие сведения об интерактивных запросах (также называются Apache Hive LLAP) в Azure HDInsight
ms.service: hdinsight
ms.topic: overview
ms.custom: hdinsightactive
ms.date: 03/03/2020
ms.openlocfilehash: 2813554700e015c0ac34e47d632d16d97c948c4e
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/28/2021
ms.locfileid: "98941084"
---
# <a name="what-is-interactive-query-in-azure-hdinsight"></a>Общие сведения об интерактивном запросе в Azure HDInsight

Interactive Query (также называется Apache Hive LLAP или [Low Latency Analytical Processing](https://cwiki.apache.org/confluence/display/Hive/LLAP)) — это [тип кластера](../hdinsight-hadoop-provision-linux-clusters.md#cluster-type) Azure HDInsight. Interactive Query поддерживает кэширование в памяти, благодаря чему запросы Apache Hive становятся более быстрыми и интерактивными. Клиенты используют интерактивный запрос для сверхбыстрого запроса данных, хранящихся в службе хранилища Azure и Azure Data Lake Storage. Интерактивный запрос упрощает разработчикам и специалистам по обработке и анализу данных работу с большими данными с помощью предпочитаемых средств бизнес-аналитики. Интерактивный запрос HDInsight поддерживает несколько инструментов для простого доступа к большим данным.

[!INCLUDE [hdinsight-price-change](../../../includes/hdinsight-enhancements.md)]

Кластер Interactive Query отличается от кластера Apache Hadoop. Он содержит только службу Hive.

Получить доступ к службе Hive в кластере Interactive Query можно только с помощью представления Apache Ambari Hive, Beeline и драйвера Microsoft Hive ODBC. Эта служба недоступна через консоль Hive, Templeton, классический интерфейс командной строки Azure и Azure PowerShell.

## <a name="create-an-interactive-query-cluster"></a>Создание кластера Interactive Query

Дополнительные сведения о создании кластера HDInsight см. в статье [Установка кластеров в HDInsight с использованием Hadoop, Spark, Kafka и других технологий](../hdinsight-hadoop-provision-linux-clusters.md). Выберите тип кластера Interactive Query.

> [!IMPORTANT]
> Минимальный размер головного узла для кластеров Interactive Query — это Standard_D13_v2. Дополнительные сведения см. в [спецификации размеров виртуальных машин Azure](../../cloud-services/cloud-services-sizes-specs.md#dv2-series).

## <a name="execute-apache-hive-queries-from-interactive-query"></a>Выполнение запросов Apache Hive из Interactive Query

Есть несколько способов выполнять запросы Hive:

|Метод |Description |
|---|---|
|Microsoft Power BI|Ознакомьтесь со статьей о [визуализации данных интерактивного запроса Apache Hive с помощью Power BI в Azure HDInsight](./apache-hadoop-connect-hive-power-bi-directquery.md) и о [визуализации больших данных с помощью Power BI в Azure HDInsight](../hadoop/apache-hadoop-connect-hive-power-bi.md).|
|Visual Studio|Ознакомьтесь с разделом [Выполнение интерактивных запросов Hive](../hadoop/apache-hadoop-visual-studio-tools-get-started.md#run-interactive-apache-hive-queries).|
|Visual Studio Code|Ознакомьтесь со статьей [Use Azure HDInsight Tools for Visual Studio Code](../hdinsight-for-vscode.md) (Использование средств Azure HDInsight для Visual Studio Code).|
|Представление Ambari Apache Hive|Ознакомьтесь со статьей [Использование представления Hive Apache Ambari с Apache Hadoop в HDInsight](../hadoop/apache-hadoop-use-hive-ambari-view.md). В HDInsight 4.0 больше не используется представление Hive.|
|Apache Beeline|Ознакомьтесь со статьей [Использование клиента Apache Beeline с Apache Hive](../hadoop/apache-hadoop-use-hive-beeline.md). Вы можете использовать Beeline на головном узле или пустом граничном узле. Рекомендуем использовать Beeline на пустом граничном узле. Дополнительные сведения о создании кластера HDInsight с пустым граничным узлом см. в статье [Использование пустых граничных узлов в кластерах Hadoop в HDInsight](../hdinsight-apps-use-edge-node.md).|
|Hive ODBC|Ознакомьтесь со статьей [Подключение Excel к Apache Hadoop с помощью драйвера Microsoft Hive ODBC в Azure HDInsight](../hadoop/apache-hadoop-connect-excel-hive-odbc-driver.md).|

Чтобы найти строку подключения Java Database Connectivity (JDBC):

1. В веб-браузере перейдите на страницу `https://CLUSTERNAME.azurehdinsight.net/#/main/services/HIVE/summary`, где `CLUSTERNAME` — это имя вашего кластера.
1. Чтобы скопировать URL-адрес, щелкните значок буфера обмена:

   ![JDBC для кластеров HDInsight Hadoop типа Interactive Query на LLAP](./media/apache-interactive-query-get-started/hdinsight-hadoop-use-interactive-hive-jdbc.png)

## <a name="next-steps"></a>Дальнейшие действия

* Узнайте, как [создавать кластеры Interactive Query в HDInsight](../hdinsight-hadoop-provision-linux-clusters.md).
* Узнайте, как [визуализировать большие данные с помощью Power BI в Azure HDInsight](../hadoop/apache-hadoop-connect-hive-power-bi.md).
* Узнайте, как [выполнять запросы Apache Hive в Azure HDInsight с помощью Apache Zeppelin](../interactive-query/hdinsight-connect-hive-zeppelin.md).
