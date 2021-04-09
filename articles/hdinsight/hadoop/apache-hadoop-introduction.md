---
title: Сведения об Apache Hadoop и MapReduce — Azure HDInsight
description: Общие сведения о HDInsight и стеке технологий и компонентах Apache Hadoop.
ms.service: hdinsight
ms.topic: overview
ms.custom: hdinsightactive,hdiseo17may2017,mvc,seodec18
ms.date: 02/27/2020
ms.openlocfilehash: ad1f7422919f224889db84a2599ad3c1c48efcc5
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104863816"
---
# <a name="what-is-apache-hadoop-in-azure-hdinsight"></a>Что такое Apache Hadoop в Azure HDInsight?

Первоначально технология [Apache Hadoop](https://hadoop.apache.org/) была платформой с открытым кодом для распределенной обработки и анализа наборов больших данных в кластерах. Экосистема Hadoop состоит из взаимосвязанного программного обеспечения и служебных программ, таких как Apache Hive, Apache HBase, Spark, Kafka и т. д.

Azure HDInsight — это полностью управляемая комплексная облачная служба аналитики с открытым кодом, предназначенная для предприятий. Тип кластера Apache Hadoop в Azure HDInsight позволяет использовать [распределенную файловую систему Apache Hadoop (HDFS)](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html) и управление ресурсами [Apache Hadoop YARN](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html), а также простую модель программирования [MapReduce](https://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html) для параллельной обработки и анализа пакетных данных.  Кластеры Hadoop в HDInsight совместимы с [Хранилищем BLOB-объектов Azure](../../storage/common/storage-introduction.md), [Azure Data Lake Storage 1-го поколения](../../data-lake-store/data-lake-store-overview.md) или [Azure Data Lake Storage 2-го поколения](../../storage/blobs/data-lake-storage-introduction.md).

Просмотреть доступные компоненты стека технологии Hadoop в HDInsight можно в статье [Что представляют собой компоненты и версии Hadoop, доступные в HDInsight?](../hdinsight-component-versioning.md) Дополнительные сведения о Hadoop в HDInsight см. на странице [возможностей HDInsight в Azure](https://azure.microsoft.com/services/hdinsight/).

## <a name="what-is-mapreduce"></a>Что такое MapReduce

[Apache Hadoop MapReduce](https://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html) — это программная платформа для создания заданий, обрабатывающих большие объемы данных. Входные данные разбиваются на независимые блоки, которые затем обрабатываются параллельно на узлах кластера. Задание MapReduce состоит из двух функций.

* **Mapper**(Модуль сопоставления) — принимает входные данные, анализирует их (обычно с помощью фильтрации и сортировки) и передает кортежи (пары «ключ-значение»).

* **Reducer**(Редуктор) — принимает кортежи, сформированные в модуле сопоставления, и выполняет операцию сводки, которая создает результат меньшего размера, объединяющий данные модуля сопоставления

На следующей диаграмме показан пример задания MapReduce, которое выполняет простую операцию подсчета слов:

 :::image type="content" source="./media/apache-hadoop-introduction/hdi-word-count-diagram.gif" alt-text="HDI.WordCountDiagram" border="true":::

Выходные данные этого задания представляют собой частоту использования каждого слова в тексте.

* Процедура map берет каждую строку из входного текста в качестве входных данных и разбивает ее на слова. Она генерирует пару «ключ-значение» каждый раз, когда встречается слово, за которым следует 1. Перед отправкой на обработку редуктором выходные данные сортируются.
* Затем редуктор суммирует эти отдельные счетчики для каждого слова и выдает одну пару «ключ-значение», содержащую слово, за которым следует частота его использования.

Задание MapReduce может быть реализовано на различных языках. Java — это наиболее распространенная реализация, которая используется в данном документе для примера.

## <a name="development-languages"></a>Языки разработки

Языки или платформы на основе Java или виртуальной машины Java можно запускать непосредственно как [задание MapReduce](..//hadoop/submit-apache-hadoop-jobs-programmatically.md). В качестве примера в этом документе приведено приложение MapReduce на языке Java. Прочие языки и платформы, например C# или Python, или изолированные исполняемые файлы должны использовать **потоковую передачу Hadoop**.

Потоковая передача Hadoop взаимодействует с модулями сопоставления и редукции через потоки STDIN и STDOUT. Модули сопоставления и редукции построчно считывают данные из потока STDIN и записывают выходные данные в поток STDOUT. Каждая строка, которая читается или генерируется модулем сопоставления или редукции, должна быть в формате пар "ключ-значение", разделенных знаком табуляции.

`[key]\t[value]`

Дополнительные сведения см. в документации по [потоковой передаче Hadoop](https://hadoop.apache.org/docs/current/hadoop-streaming/HadoopStreaming.html).

Примеры использования потоковой передачи Hadoop с HDInsight см. в следующих документах:

* [Использование языка C# для потоковой передачи MapReduce в Hadoop в HDInsight](apache-hadoop-dotnet-csharp-mapreduce-streaming.md)

## <a name="where-do-i-start"></a>С чего начать

* [Краткое руководство. Создание кластера Apache Hadoop в Azure HDInsight с помощью портала Azure](../hadoop/apache-hadoop-linux-create-cluster-get-started-portal.md)
* [Учебник. Отправка заданий Apache Hadoop в HDInsight](../hadoop/submit-apache-hadoop-jobs-programmatically.md)
* [Разработка программ MapReduce на Java для Apache Hadoop в HDInsight](../hadoop/apache-hadoop-develop-deploy-java-mapreduce-linux.md)
* [Использование Apache Hive как средства для извлечения, преобразования и загрузки](../hadoop/apache-hadoop-using-apache-hive-as-an-etl-tool.md)
* [Extract, transform, and load (ETL) at scale](../hadoop/apache-hadoop-etl-at-scale.md) (Извлечение, преобразование и загрузка (ETL) в масштабе)
* [Ввод в эксплуатацию конвейера аналитики данных](../hdinsight-operationalize-data-pipeline.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Создание кластера Apache Hadoop в HDInsight с помощью портала](../hadoop/apache-hadoop-linux-create-cluster-get-started-portal.md)
* [Создание кластера Apache Hadoop в HDInsight с помощью шаблона ARM](../hadoop/apache-hadoop-linux-tutorial-get-started.md)