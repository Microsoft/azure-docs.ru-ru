---
title: Использование пользовательских пакетов Maven с Jupyter в Spark — Azure HDInsight
description: Пошаговые инструкции по настройке записных книжек Jupyter, доступных в кластерах HDInsight Spark, для использования пользовательских пакетов Maven.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 11/22/2019
ms.openlocfilehash: 3ddfdfdfe10d5b6ea7c2d5cd99d325564163c0dd
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104866018"
---
# <a name="use-external-packages-with-jupyter-notebooks-in-apache-spark-clusters-on-hdinsight"></a>Использование внешних пакетов с записными книжками Jupyter в кластерах Apache Spark в HDInsight

Узнайте, как настроить [Jupyter Notebook](https://jupyter.org/) в кластере Apache Spark в HDInsight для использования внешних, участвующих в сообществе пакетов Apache **Maven** , которые не включены в готовый список в кластере.

Полный список доступных пакетов можно найти в [репозитории Maven](https://search.maven.org/) . Его также можно получить из других источников. Например, полный список предоставленных сообществом пакетов можно найти в разделе [Пакеты Spark](https://spark-packages.org/).

В этой статье вы узнаете, как использовать пакет [Spark-CSV](https://search.maven.org/#artifactdetails%7Ccom.databricks%7Cspark-csv_2.10%7C1.4.0%7Cjar) с Jupyter Notebook.

## <a name="prerequisites"></a>Предварительные требования

* Кластер Apache Spark в HDInsight. Инструкции см. в статье [Начало работы. Создание кластера Apache Spark в HDInsight на платформе Linux и выполнение интерактивных запросов с помощью SQL Spark](apache-spark-jupyter-spark-sql.md).

* Опыт работы с записными книжками Jupyter с Spark в HDInsight. Дополнительные сведения см. в статье [Руководство. Загрузка данных и выполнение запросов в кластере Apache Spark в Azure HDInsight](./apache-spark-load-data-run-query.md).

* [Схема универсального кода ресурса (URI)](../hdinsight-hadoop-linux-information.md#URI-and-scheme) для основного хранилища кластеров. Для службы хранилища Azure это будет `wasb://`, для Azure Data Lake Storage 2-го поколения — `abfs://` или `adl://` для Azure Data Lake Storage 1-го поколения. Если для службы хранилища Azure или Azure Data Lake Storage 2-го поколения включена безопасная передача, URI будет `wasbs://` или `abfss://` соответственно (см. также сведения о [безопасной передаче](../../storage/common/storage-require-secure-transfer.md)).

## <a name="use-external-packages-with-jupyter-notebooks"></a>Использование внешних пакетов с записными книжками Jupyter

1. Перейдите в `https://CLUSTERNAME.azurehdinsight.net/jupyter` папку `CLUSTERNAME` , где — имя кластера Spark.

1. Создайте новую записную книжку. Выберите **создать**, а затем выберите **Spark**.

    :::image type="content" source="./media/apache-spark-jupyter-notebook-use-external-packages/hdinsight-spark-create-notebook.png " alt-text="Создание новой Jupyter Notebook Spark" border="true":::

1. Будет создана и открыта записная книжка с именем Untitled.pynb. Выберите имя записной книжки вверху и введите понятное имя.

    :::image type="content" source="./media/apache-spark-jupyter-notebook-use-external-packages/hdinsight-spark-name-notebook.png " alt-text="Указание имени для записной книжки" border="true":::

1. С помощью `%%configure` волшебной настройки записной книжки можно настроить использование внешнего пакета. В записных книжках, которые используют внешние пакеты, следует вызывать волшебную команду `%%configure` в первой ячейке кода. Она настраивает ядро для использования пакета до начала сеанса.

    >[!IMPORTANT]  
    >Если вы забыли настроить ядро в первой ячейке, можно использовать `%%configure` с параметром `-f`, но при этом сеанс будет перезапущен и все изменения будут утеряны.

    | Версия HDInsight | Get-Help |
    |-------------------|---------|
    | Для HDInsight 3.5 и HDInsight 3.6 | `%%configure`<br>`{ "conf": {"spark.jars.packages": "com.databricks:spark-csv_2.11:1.5.0" }}`|
    |Для HDInsight 3.3 и HDInsight 3.4 | `%%configure` <br>`{ "packages":["com.databricks:spark-csv_2.10:1.4.0"] }`|

1. Приведенный выше фрагмент ожидает получить координаты Maven для внешнего пакета в центральном репозитории Maven. В этом фрагменте кода `com.databricks:spark-csv_2.11:1.5.0` — координата пакета **spark-csv** в Maven. Вот как сформировать координаты для пакета.

    а. Найдите пакет в репозитории Maven. В этой статье мы используем [Spark-CSV](https://mvnrepository.com/artifact/com.databricks/spark-csv).

    b. В репозитории найдите значения для параметров **GroupId**, **ArtifactId** и **Version**. Убедитесь, что полученные значения соответствуют кластеру. В этом случае мы используем пакет Scala 2,11 и Spark 1.5.0, но вам может потребоваться выбрать разные версии для соответствующей версии Scala или Spark в кластере. Выяснить версию Scala в кластере можно, выполнив `scala.util.Properties.versionString` для ядра Jupyter Spark или при отправке Spark. Версию Spark в кластере можно узнать, запустив `sc.version` в записных книжках Jupyter.

    :::image type="content" source="./media/apache-spark-jupyter-notebook-use-external-packages/use-external-packages-with-jupyter.png " alt-text="Использование внешних пакетов с Jupyter Notebook" border="true":::

    c. Объедините три значения, разделив их двоеточием (**:**).

    ```scala
    com.databricks:spark-csv_2.11:1.5.0
    ```

1. Запустите ячейку кода с помощью волшебной команды `%%configure` . После выполнения этой команды соответствующий сеанс Livy будет использовать указанный вами пакет. В последующих ячейках записной книжки теперь можно использовать этот пакет, как показано ниже.

    ```scala
    val df = spark.read.format("com.databricks.spark.csv").
    option("header", "true").
    option("inferSchema", "true").
    load("wasb:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")
    ```

    Для HDInsight 3,4 и ниже следует использовать следующий фрагмент кода.

    ```scala
    val df = sqlContext.read.format("com.databricks.spark.csv").
    option("header", "true").
    option("inferSchema", "true").
    load("wasb:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")
    ```

1. Затем можно запустить фрагменты кода, как показано ниже, чтобы просмотреть данные из таблицы данных, созданной на предыдущем шаге.

    ```scala
    df.show()
   
    df.select("Time").count()
    ```

## <a name="see-also"></a><a name="seealso"></a> См. также

* [Обзор: Spark в Azure HDInsight](apache-spark-overview.md)

### <a name="scenarios"></a>Сценарии

* [Использование Apache Spark со средствами бизнес-аналитики. Выполнение интерактивного анализа данных с использованием Spark в HDInsight с помощью средств бизнес-аналитики](apache-spark-use-bi-tools.md)
* [Apache Spark и Машинное обучение. Анализ температуры в здании на основе данных системы кондиционирования с помощью Spark в HDInsight](apache-spark-ipython-notebook-machine-learning.md)
* [Apache Spark и Машинное обучение. Прогнозирование результатов проверки пищевых продуктов с помощью Spark в HDInsight](apache-spark-machine-learning-mllib-ipython.md)
* [Анализ журналов веб-сайтов с помощью Apache Spark в HDInsight](apache-spark-custom-library-website-log-analysis.md)

### <a name="create-and-run-applications"></a>Создание и запуск приложений

* [Создание автономного приложения с использованием Scala](apache-spark-create-standalone-application.md)
* [Удаленный запуск заданий с помощью Apache Livy в кластере Apache Spark](apache-spark-livy-rest-interface.md)

### <a name="tools-and-extensions"></a>Инструменты и расширения

* [Использование внешних пакетов Python с записными книжками Jupyter в кластерах Apache Spark в HDInsight на платформе Linux](apache-spark-python-package-installation.md)
* [Использование подключаемого модуля средств HDInsight для IntelliJ IDEA для создания и отправки приложений Spark Scala](apache-spark-intellij-tool-plugin.md)
* [Удаленная отладка приложений Apache Spark с помощью подключаемого модуля средств HDInsight для IntelliJ IDEA](apache-spark-intellij-tool-plugin-debug-jobs-remotely.md)
* [Использование записных книжек Zeppelin с кластером Apache Spark в Azure HDInsight](apache-spark-zeppelin-notebook.md)
* [Ядра, доступные для Jupyter Notebook в кластере Apache Spark для HDInsight](apache-spark-jupyter-notebook-kernels.md)
* [Установка записной книжки Jupyter на компьютере и ее подключение к кластеру Apache Spark в Azure HDInsight (предварительная версия)](apache-spark-jupyter-notebook-install-locally.md)

### <a name="manage-resources"></a>Управление ресурсами

* [Управление ресурсами кластера Apache Spark в Azure HDInsight](apache-spark-resource-manager.md)
* [Отслеживание и отладка заданий в кластере Apache Spark в HDInsight на платформе Linux](apache-spark-job-debugging.md)
