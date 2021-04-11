---
title: Руководство по Создание приложения машинного обучения Spark — Azure HDInsight
description: 'Учебник: пошаговые инструкции по созданию приложения машинного обучения Apache Spark в кластерах HDInsight Spark с помощью записной книжки Jupyter Notebook.'
ms.service: hdinsight
ms.topic: tutorial
ms.custom: hdinsightactive,mvc, devx-track-python
ms.date: 04/07/2020
ms.openlocfilehash: 30ac866aa42861299ed6bac0f0af7522824234e6
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106062868"
---
# <a name="tutorial-build-an-apache-spark-machine-learning-application-in-azure-hdinsight"></a>Руководство по Создание приложения машинного обучения Apache Spark в Azure HDInsight

Из этого руководства вы узнаете, как с помощью [Jupyter Notebook](https://jupyter.org/) создать приложение машинного обучения [Apache Spark](./apache-spark-overview.md) для Azure HDInsight.

[MLlib](https://spark.apache.org/docs/latest/ml-guide.html) — это библиотека машинного обучения Apache Spark, состоящая из общих алгоритмов и служебных программ обучения (классификация, регрессия, кластеризация, совместная фильтрация, уменьшение размерности, а также базовые элементы оптимизации).

В этом руководстве описано следующее:
> [!div class="checklist"]
> * разработка приложения машинного обучения Apache Spark.

## <a name="prerequisites"></a>Предварительные требования

* Кластер Apache Spark в HDInsight. Ознакомьтесь со статьей [Краткое руководство. Создание кластера Apache Spark в HDInsight с помощью шаблона](./apache-spark-jupyter-spark-sql-use-portal.md).

* Опыт работы с записными книжками Jupyter с Spark в HDInsight. Дополнительные сведения см. в статье [Руководство. Загрузка данных и выполнение запросов в кластере Apache Spark в Azure HDInsight](./apache-spark-load-data-run-query.md).

## <a name="understand-the-data-set"></a>Общие сведения о наборе данных

Приложение использует данные из примера файла **HVAC.csv**, который по умолчанию доступен на всех кластерах. Файл находится в папке `\HdiSamples\HdiSamples\SensorSampleData\hvac`. Это данные о целевой температуре и фактической температуре некоторых зданий, в которых установлена система кондиционирования воздуха. В столбце **System** указан идентификатор системы, а в столбце **SystemAge** — срок эксплуатации системы кондиционирования в годах. Вы можете спрогнозировать, будет ли температура здания выше или ниже относительно целевой температуры на основе идентификатора системы и срока ее эксплуатации.

:::image type="content" source="./media/apache-spark-ipython-notebook-machine-learning/spark-machine-learning-understand-data.png " alt-text="Моментальный снимок данных, используемых для примера машинного обучения Spark" border="true":::

## <a name="develop-a-spark-machine-learning-application-using-spark-mllib"></a>Разработка приложения машинного обучения Spark с помощью Spark MLlib

В этом приложении используется [конвейер машинного обучения](https://spark.apache.org/docs/2.2.0/ml-pipeline.html) Spark для выполнения классификации документов. Конвейеры машинного обучения предоставляют универсальный набор интерфейсов API высокого уровня, построенных на основе DataFrames. DataFrames позволяют создавать и настраивать практичные конвейеры машинного обучения. В конвейере вы разобьете документ на слова, преобразуете слова в вектор числового признака и, наконец, создадите модель прогнозирования с помощью вектора и меток признаков. Выполните следующие действия, чтобы создать приложение.

1. Создайте записную книжку Jupyter Notebook с помощью ядра PySpark. Инструкции см. в разделе [Создание файла записной книжки Jupyter Notebook](./apache-spark-jupyter-spark-sql.md#create-a-jupyter-notebook-file).

1. Импортируйте типы, необходимые для этого сценария. Вставьте следующий фрагмент кода в пустую ячейку и нажмите клавиши **SHIFT + ВВОД**.

    ```PySpark
    from pyspark.ml import Pipeline
    from pyspark.ml.classification import LogisticRegression
    from pyspark.ml.feature import HashingTF, Tokenizer
    from pyspark.sql import Row

    import os
    import sys
    from pyspark.sql.types import *

    from pyspark.mllib.classification import LogisticRegressionWithSGD
    from pyspark.mllib.regression import LabeledPoint
    from numpy import array
    ```

1. Загрузите данные (hvac.csv), проанализируйте их и используйте эти данные для обучения модели.

    ```PySpark
    # Define a type called LabelDocument
    LabeledDocument = Row("BuildingID", "SystemInfo", "label")

    # Define a function that parses the raw CSV file and returns an object of type LabeledDocument
    def parseDocument(line):
        values = [str(x) for x in line.split(',')]
        if (values[3] > values[2]):
            hot = 1.0
        else:
            hot = 0.0

        textValue = str(values[4]) + " " + str(values[5])

        return LabeledDocument((values[6]), textValue, hot)

    # Load the raw HVAC.csv file, parse it using the function
    data = sc.textFile("/HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")

    documents = data.filter(lambda s: "Date" not in s).map(parseDocument)
    training = documents.toDF()
    ```

    В фрагменте кода определите функцию, которая сравнивает фактическую температуру с целевой. Если фактическая температура больше, то здание горячее (значение **1.0**). В противном случае в здании холодно (значение **0,0**).

1. Настройте конвейер машинного обучения Spark, который состоит из трех частей: логического анализатора, hashingTF и lr.

    ```PySpark
    tokenizer = Tokenizer(inputCol="SystemInfo", outputCol="words")
    hashingTF = HashingTF(inputCol=tokenizer.getOutputCol(), outputCol="features")
    lr = LogisticRegression(maxIter=10, regParam=0.01)
    pipeline = Pipeline(stages=[tokenizer, hashingTF, lr])
    ```

    См. дополнительные сведения о [конвейере машинного обучения Apache Spark и том, как он работает](https://spark.apache.org/docs/latest/ml-pipeline.html).

1. Впишите конвейер в документ для обучения.

    ```PySpark
    model = pipeline.fit(training)
    ```

1. Проверьте, чтобы в документе для обучения имелась контрольная точка хода выполнения для приложения.

    ```PySpark
    training.show()
    ```

    Выходные данные должны быть следующего вида.

    ```output
    +----------+----------+-----+
    |BuildingID|SystemInfo|label|
    +----------+----------+-----+
    |         4|     13 20|  0.0|
    |        17|      3 20|  0.0|
    |        18|     17 20|  1.0|
    |        15|      2 23|  0.0|
    |         3|      16 9|  1.0|
    |         4|     13 28|  0.0|
    |         2|     12 24|  0.0|
    |        16|     20 26|  1.0|
    |         9|      16 9|  1.0|
    |        12|       6 5|  0.0|
    |        15|     10 17|  1.0|
    |         7|      2 11|  0.0|
    |        15|      14 2|  1.0|
    |         6|       3 2|  0.0|
    |        20|     19 22|  0.0|
    |         8|     19 11|  0.0|
    |         6|      15 7|  0.0|
    |        13|      12 5|  0.0|
    |         4|      8 22|  0.0|
    |         7|      17 5|  0.0|
    +----------+----------+-----+
    ```

    Сравнение выходных данных со сведениями в необработанном CSV-файле. Например, в первой строке CSV-файла содержатся следующие данные:

    :::image type="content" source="./media/apache-spark-ipython-notebook-machine-learning/spark-machine-learning-output-data.png " alt-text="Моментальный снимок выходных данных из примера машинного обучения Spark" border="true":::

    Обратите внимание на то, насколько фактическая температура меньше целевой, что свидетельствует о том, что здание холодное. Значение в первой строке столбца **label** составляет **0,0**. Это означает, что в здании не жарко.

1. Подготовьте набор данных, в отношении которого необходимо выполнить обученную модель. Для этого передайте идентификатор системы и данные о сроке эксплуатации системы (обозначается как **SystemInfo** в выходных данных обучения). Модель прогнозирует, будет ли здание, оборудованное системой с этим идентификатором и сроком эксплуатации, теплее (значение 1,0) или холоднее (значение 0,0).

    ```PySpark
    # SystemInfo here is a combination of system ID followed by system age
    Document = Row("id", "SystemInfo")
    test = sc.parallelize([(1L, "20 25"),
                    (2L, "4 15"),
                    (3L, "16 9"),
                    (4L, "9 22"),
                    (5L, "17 10"),
                    (6L, "7 22")]) \
        .map(lambda x: Document(*x)).toDF()
    ```

1. Наконец, создайте прогнозы на основе тестовых данных.

    ```PySpark
    # Make predictions on test documents and print columns of interest
    prediction = model.transform(test)
    selected = prediction.select("SystemInfo", "prediction", "probability")
    for row in selected.collect():
        print row
    ```

    Выходные данные должны быть следующего вида.

    ```output  
    Row(SystemInfo=u'20 25', prediction=1.0, probability=DenseVector([0.4999, 0.5001]))
    Row(SystemInfo=u'4 15', prediction=0.0, probability=DenseVector([0.5016, 0.4984]))
    Row(SystemInfo=u'16 9', prediction=1.0, probability=DenseVector([0.4785, 0.5215]))
    Row(SystemInfo=u'9 22', prediction=1.0, probability=DenseVector([0.4549, 0.5451]))
    Row(SystemInfo=u'17 10', prediction=1.0, probability=DenseVector([0.4925, 0.5075]))
    Row(SystemInfo=u'7 22', prediction=0.0, probability=DenseVector([0.5015, 0.4985]))
    ```

   Изучите первую строку прогноза. При использовании системы кондиционирования с идентификатором 20 и сроком эксплуатации 25 лет температура в здании будет высокая (**значение прогноза 1,0**). Первое значение параметра DenseVector (0,49999) соответствует прогнозу 0,0, а второе значение (0,5001) — прогнозу 1,0. В выходных данных видно, что, даже если второе значение лишь незначительно выше, модель показывает **прогноз = 1.0**.

1. Завершите работу записной книжки для освобождения ресурсов. Для этого в меню **File** (Файл) записной книжки выберите пункт **Close and Halt** (Закрыть и остановить). Записная книжка завершит работу и закроется.

## <a name="use-anaconda-scikit-learn-library-for-spark-machine-learning"></a>Использование библиотеки scikit-learn Anaconda для машинного обучения Spark

Кластеры Apache Spark в HDInsight включают библиотеки Anaconda. Они также включают библиотеку **scikit-learn** для машинного обучения. Кроме того, библиотека включает различные наборы данных, которые можно использовать для создания примеров приложений прямо в записной книжке Jupyter Notebook. Примеры использования библиотеки scikit-learn см. здесь: [https://scikit-learn.org/stable/auto_examples/index.html](https://scikit-learn.org/stable/auto_examples/index.html).

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не собираетесь использовать это приложение в дальнейшем, удалите созданный кластер, сделав следующее:

1. Войдите на [портал Azure](https://portal.azure.com/).

1. В поле **Поиск** в верхней части страницы введите **HDInsight**.

1. Выберите **Кластеры HDInsight** в разделе **Службы**.

1. В списке кластеров HDInsight, который отобразится, выберите **...** рядом с кластером, созданным при работе с этим учебником.

1. Выберите команду **Удалить**. Выберите **Да**.

:::image type="content" source="./media/apache-spark-ipython-notebook-machine-learning/hdinsight-azure-portal-delete-cluster.png " alt-text="Удаление кластера HDInsight на портале Azure" border="true":::

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как с помощью Jupyter Notebook создать приложение машинного обучения Apache Spark для Azure HDInsight. Перейдите к следующему руководству, чтобы научиться использовать IntelliJ IDEA для заданий Spark.

> [!div class="nextstepaction"]
> [Создание приложения Scala Maven с помощью IntelliJ](./apache-spark-create-standalone-application.md)
