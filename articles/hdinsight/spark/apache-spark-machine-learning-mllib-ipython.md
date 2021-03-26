---
title: Пример машинного обучения с использованием Spark MLlib в HDInsight — Azure
description: Узнайте, как использовать Spark MLlib для создания приложения машинного обучения, которое анализирует набор данных с помощью классификации посредством логистической регрессии.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017,seoapr2020, devx-track-python
ms.date: 04/27/2020
ms.openlocfilehash: 224ebae58a5688f81d1b64b4a9c81d5abd8a9690
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104868857"
---
# <a name="use-apache-spark-mllib-to-build-a-machine-learning-application-and-analyze-a-dataset"></a>Использование Apache Spark MLlib для создания приложения машинного обучения и анализа набора данных

Узнайте, как использовать Apache Spark MLlib для создания приложения машинного обучения. Приложение выполняет прогнозный анализ на открытом наборе данных. В этом примере используется *классификация* посредством логистической регрессии на основе встроенных библиотек машинного обучения Spark.

MLlib — это основная библиотека Spark, которая предоставляет множество служебных программ, полезных для задач машинного обучения, таких как:

* Классификация
* Регрессия
* Кластеризация
* Моделирование
* сингулярного разложения и анализа по методу главных компонент;
* проверки гипотез и статистической выборки.

## <a name="understand-classification-and-logistic-regression"></a>Общие сведения о классификации и логистической регрессии

*Классификация* — это распространенная задача машинного обучения, которая представляет собой процесс сортировки входных данных по категориям. Это задание алгоритма классификации, позволяющее определить, как назначить "метки" входным данным, которые вы предоставляете. Например, можно представить алгоритм машинного обучения, который принимает в качестве входных данных данные о акции. Затем делит биржевую акцию на две категории: акции, которые следует продавать и акции, которые следует хранить.

Логистическая регрессия — один из алгоритмов классификации. API Spark для логистической регрессии подходит для задач *двоичной классификации* или разделения входных данных на две группы. Дополнительные сведения о логистической регрессии см. в статье [Википедии](https://en.wikipedia.org/wiki/Logistic_regression).

В целом, процесс логистической регрессии создает *логистическую функцию*. Используйте функцию для прогнозирования вероятности того, что входной вектор принадлежит одной группе или другой.  

## <a name="predictive-analysis-example-on-food-inspection-data"></a>Пример прогнозного анализа на основе данных контроля качества пищевых продуктов

В этом примере используется Spark для прогнозного анализа данных контроля пищи (**Food_Inspections1.csv**). Данные, полученные по [городу портала данных в Чикаго](https://data.cityofchicago.org/). Этот набор данных содержит сведения об исследованиях установки продуктов, проведенных в Чикаго. Включая сведения о каждом установлении, обнаруженные нарушения (если они есть) и результаты проверки. CSV-файл данных уже доступен в учетной записи хранения, связанной с кластером **/HdiSamples/HdiSamples/FoodInspectionData/Food_Inspections1.csv**.

В следующих разделах описаны шаги по разработке модели, которая позволит узнать, что нужно, чтобы пройти контроль качества пищевых продуктов.

## <a name="create-an-apache-spark-mllib-machine-learning-app"></a>Создание приложения машинного обучения Apache Spark MLlib

1. Создайте записную книжку Jupyter Notebook с помощью ядра PySpark. Инструкции см. в разделе [Создание файла записной книжки Jupyter Notebook](./apache-spark-jupyter-spark-sql.md#create-a-jupyter-notebook-file).

2. Импортируйте типы, необходимые для этого приложения. Скопируйте и вставьте следующий код в пустую ячейку, а затем нажмите клавиши **SHIFT + ВВОД**.

    ```PySpark
    from pyspark.ml import Pipeline
    from pyspark.ml.classification import LogisticRegression
    from pyspark.ml.feature import HashingTF, Tokenizer
    from pyspark.sql import Row
    from pyspark.sql.functions import UserDefinedFunction
    from pyspark.sql.types import *
    ```

    Ядро PySpark позволяет не задавать контексты явным образом. Контексты Spark и Hive создаются автоматически при запуске первой ячейки кода.

## <a name="construct-the-input-dataframe"></a>Создание входного кадра данных

Используйте контекст Spark для извлечения необработанных данных CSV в память в виде неструктурированного текста. Затем с помощью библиотеки CSV Python проанализируйте каждую строку данных.

1. Запустите следующие строки, чтобы импортировать входные данные и создать на их основе устойчивый распределенный набор данных (RDD).

    ```PySpark
    def csvParse(s):
        import csv
        from StringIO import StringIO
        sio = StringIO(s)
        value = csv.reader(sio).next()
        sio.close()
        return value

    inspections = sc.textFile('/HdiSamples/HdiSamples/FoodInspectionData/Food_Inspections1.csv')\
                    .map(csvParse)
    ```

2. Выполните следующий код, чтобы получить из набора RDD одну строку, которая позволит изучить схему данных.

    ```PySpark
    inspections.take(1)
    ```

    Результат.

    ```
    [['413707',
        'LUNA PARK INC',
        'LUNA PARK  DAY CARE',
        '2049789',
        "Children's Services Facility",
        'Risk 1 (High)',
        '3250 W FOSTER AVE ',
        'CHICAGO',
        'IL',
        '60625',
        '09/21/2010',
        'License-Task Force',
        'Fail',
        '24. DISH WASHING FACILITIES: PROPERLY DESIGNED, CONSTRUCTED, MAINTAINED, INSTALLED, LOCATED AND OPERATED - Comments: All dishwashing machines must be of a type that complies with all requirements of the plumbing section of the Municipal Code of Chicago and Rules and Regulation of the Board of Health. OBSEVERD THE 3 COMPARTMENT SINK BACKING UP INTO THE 1ST AND 2ND COMPARTMENT WITH CLEAR WATER AND SLOWLY DRAINING OUT. INST NEED HAVE IT REPAIR. CITATION ISSUED, SERIOUS VIOLATION 7-38-030 H000062369-10 COURT DATE 10-28-10 TIME 1 P.M. ROOM 107 400 W. SURPERIOR. | 36. LIGHTING: REQUIRED MINIMUM FOOT-CANDLES OF LIGHT PROVIDED, FIXTURES SHIELDED - Comments: Shielding to protect against broken glass falling into food shall be provided for all artificial lighting sources in preparation, service, and display facilities. LIGHT SHIELD ARE MISSING UNDER HOOD OF  COOKING EQUIPMENT AND NEED TO REPLACE LIGHT UNDER UNIT. 4 LIGHTS ARE OUT IN THE REAR CHILDREN AREA,IN THE KINDERGARDEN CLASS ROOM. 2 LIGHT ARE OUT EAST REAR, LIGHT FRONT WEST ROOM. NEED TO REPLACE ALL LIGHT THAT ARE NOT WORKING. | 35. WALLS, CEILINGS, ATTACHED EQUIPMENT CONSTRUCTED PER CODE: GOOD REPAIR, SURFACES CLEAN AND DUST-LESS CLEANING METHODS - Comments: The walls and ceilings shall be in good repair and easily cleaned. MISSING CEILING TILES WITH STAINS IN WEST,EAST, IN FRONT AREA WEST, AND BY THE 15MOS AREA. NEED TO BE REPLACED. | 32. FOOD AND NON-FOOD CONTACT SURFACES PROPERLY DESIGNED, CONSTRUCTED AND MAINTAINED - Comments: All food and non-food contact equipment and utensils shall be smooth, easily cleanable, and durable, and shall be in good repair. SPLASH GUARDED ARE NEEDED BY THE EXPOSED HAND SINK IN THE KITCHEN AREA | 34. FLOORS: CONSTRUCTED PER CODE, CLEANED, GOOD REPAIR, COVING INSTALLED, DUST-LESS CLEANING METHODS USED - Comments: The floors shall be constructed per code, be smooth and easily cleaned, and be kept clean and in good repair. INST NEED TO ELEVATE ALL FOOD ITEMS 6INCH OFF THE FLOOR 6 INCH AWAY FORM WALL.  ',
        '41.97583445690982',
        '-87.7107455232781',
        '(41.97583445690982, -87.7107455232781)']]
    ```

    Эти выходные данные позволяют составить представление о схеме данных во входном файле. Он включает название каждого учреждения и тип установки. Кроме прочего, адрес, данные проверки и расположение.

3. Выполните следующий код, чтобы создать кадр данных под (*df*) и временную таблицу (*CountResults*) с несколькими столбцами, которые помогут выполнить прогностический анализ. `sqlContext` используется для преобразования структурированных данных.

    ```PySpark
    schema = StructType([
    StructField("id", IntegerType(), False),
    StructField("name", StringType(), False),
    StructField("results", StringType(), False),
    StructField("violations", StringType(), True)])

    df = spark.createDataFrame(inspections.map(lambda l: (int(l[0]), l[1], l[12], l[13])) , schema)
    df.registerTempTable('CountResults')
    ```

    Четыре столбца, представляющие интерес в кадре данных, — это **идентификатор**, **имя**, **результаты** и **нарушения**.

4. Выполните следующий код, чтобы получить небольшую выборку данных.

    ```PySpark
    df.show(5)
    ```

    Результат.

    ```
    +------+--------------------+-------+--------------------+
    |    id|                name|results|          violations|
    +------+--------------------+-------+--------------------+
    |413707|       LUNA PARK INC|   Fail|24. DISH WASHING ...|
    |391234|       CAFE SELMARIE|   Fail|2. FACILITIES TO ...|
    |413751|          MANCHU WOK|   Pass|33. FOOD AND NON-...|
    |413708|BENCHMARK HOSPITA...|   Pass|                    |
    |413722|           JJ BURGER|   Pass|                    |
    +------+--------------------+-------+--------------------+
    ```

## <a name="understand-the-data"></a>Используемые данные

Давайте подробно рассмотрим содержимое нашего набора данных. 

1. Выполните следующий код, чтобы увидеть конкретные значения из столбца **результаты**:

    ```PySpark
    df.select('results').distinct().show()
    ```

    Результат.

    ```
    +--------------------+
    |             results|
    +--------------------+
    |                Fail|
    |Business Not Located|
    |                Pass|
    |  Pass w/ Conditions|
    |     Out of Business|
    +--------------------+
    ```

2. Выполните следующий код, чтобы создать визуализацию на основе этих результатов.

    ```PySpark
    %%sql -o countResultsdf
    SELECT COUNT(results) AS cnt, results FROM CountResults GROUP BY results
    ```

    Волшебное слово `%%sql`, за которым следует `-o countResultsdf`, гарантирует, что вывод запроса сохраняется локально на сервере Jupyter (обычно это головной узел кластера). Выходные данные сохраняются в кадре данных [Pandas](https://pandas.pydata.org/) с именем **countResultsdf**. Дополнительные сведения о `%%sql` волшебе и других волшебных возможностях, доступных в ядре PySpark, см. в разделе [ядра, доступные в записных книжках Jupyter с Apache Spark кластерами HDInsight](apache-spark-jupyter-notebook-kernels.md#parameters-supported-with-the-sql-magic).

    Результат.

    :::image type="content" source="./media/apache-spark-machine-learning-mllib-ipython/spark-machine-learning-query-output.png " alt-text="Результат SQL-запроса" border="true":::

3. Также можно создать диаграмму с помощью Matplotlib, библиотеки, используемой для визуализации данных. Так как диаграмма должна создаваться из локально сохраненного кадра данных **countResultsdf**, фрагмент кода должен начинаться с волшебного слова `%%local`. Это действие гарантирует, что код будет выполняться локально на сервере Jupyter.

    ```PySpark
    %%local
    %matplotlib inline
    import matplotlib.pyplot as plt

    labels = countResultsdf['results']
    sizes = countResultsdf['cnt']
    colors = ['turquoise', 'seagreen', 'mediumslateblue', 'palegreen', 'coral']
    plt.pie(sizes, labels=labels, autopct='%1.1f%%', colors=colors)
    plt.axis('equal')
    ```

    Чтобы спрогнозировать результат проверки пищевых продуктов, вам нужно разработать модель для анализа нарушений. Логистическая регрессия является методом двоичной классификации, а значит данные целесообразно разделить на две категории: **Fail** и **Pass**.

   - Успех
       - Успех
       - Pass w/ conditions;
   - Ошибка
       - Ошибка
   - Игнорировать
       - Business not located;
       - Out of Business.

     Данные с другими результатами ("предприятие не размещается" или "вне организации") не полезны, и они все равно составляют небольшой процент результатов.

4. Выполните следующий код, чтобы преобразовать существующий кадр данных (`df`) в новый кадр данных, где каждая проверка будет представлена в виде пары "метка — нарушение". В этом случае метка `0.0` представляет сбой, метка `1.0` представляет успех, а метка `-1.0` представляет некоторые результаты, помимо этих двух результатов.

    ```PySpark
    def labelForResults(s):
        if s == 'Fail':
            return 0.0
        elif s == 'Pass w/ Conditions' or s == 'Pass':
            return 1.0
        else:
            return -1.0
    label = UserDefinedFunction(labelForResults, DoubleType())
    labeledData = df.select(label(df.results).alias('label'), df.violations).where('label >= 0')
    ```

5. Выполните следующий код, чтобы отобразить одну строку данных с меткой:

    ```PySpark
    labeledData.take(1)
    ```

    Результат.

    ```
    [Row(label=0.0, violations=u"41. PREMISES MAINTAINED FREE OF LITTER, UNNECESSARY ARTICLES, CLEANING  EQUIPMENT PROPERLY STORED - Comments: All parts of the food establishment and all parts of the property used in connection with the operation of the establishment shall be kept neat and clean and should not produce any offensive odors.  REMOVE MATTRESS FROM SMALL DUMPSTER. | 35. WALLS, CEILINGS, ATTACHED EQUIPMENT CONSTRUCTED PER CODE: GOOD REPAIR, SURFACES CLEAN AND DUST-LESS CLEANING METHODS - Comments: The walls and ceilings shall be in good repair and easily cleaned.  REPAIR MISALIGNED DOORS AND DOOR NEAR ELEVATOR.  DETAIL CLEAN BLACK MOLD LIKE SUBSTANCE FROM WALLS BY BOTH DISH MACHINES.  REPAIR OR REMOVE BASEBOARD UNDER DISH MACHINE (LEFT REAR KITCHEN). SEAL ALL GAPS.  REPLACE MILK CRATES USED IN WALK IN COOLERS AND STORAGE AREAS WITH PROPER SHELVING AT LEAST 6' OFF THE FLOOR.  | 38. VENTILATION: ROOMS AND EQUIPMENT VENTED AS REQUIRED: PLUMBING: INSTALLED AND MAINTAINED - Comments: The flow of air discharged from kitchen fans shall always be through a duct to a point above the roofline.  REPAIR BROKEN VENTILATION IN MEN'S AND WOMEN'S WASHROOMS NEXT TO DINING AREA. | 32. FOOD AND NON-FOOD CONTACT SURFACES PROPERLY DESIGNED, CONSTRUCTED AND MAINTAINED - Comments: All food and non-food contact equipment and utensils shall be smooth, easily cleanable, and durable, and shall be in good repair.  REPAIR DAMAGED PLUG ON LEFT SIDE OF 2 COMPARTMENT SINK.  REPAIR SELF CLOSER ON BOTTOM LEFT DOOR OF 4 DOOR PREP UNIT NEXT TO OFFICE.")]
    ```

## <a name="create-a-logistic-regression-model-from-the-input-dataframe"></a>Создание модели логистической регрессии на основе входной таблицы данных

Последняя задача — преобразование помеченных данных. Преобразуйте данные в формат, который можно проанализировать с помощью логистической регрессии. Входным данным для алгоритма логистической регрессии требуется набор *пар «метка-вектор»*. Где "вектор признаков" — это вектор чисел, который представляет точку входа. Поэтому необходимо преобразовать столбец «нарушения», который является частично структурированным, и содержит много комментариев в свободной текстовой области. Преобразовать столбец в массив вещественных чисел, которые может легко понять компьютер.

Одним из стандартных подходов машинного обучения для обработки естественного языка является назначение каждого отдельного слова "index". Затем передайте вектор в алгоритм машинного обучения. Таким, что каждое значение индекса содержит относительную частоту этого слова в текстовой строке.

MLlib предоставляет простой способ выполнения этой операции. Сначала пометим каждую строку нарушений с помощью маркеров, чтобы определить отдельные слова в каждой строке. Затем используем `HashingTF`, чтобы преобразовать каждый набор маркеров в вектор признаков, который затем может быть передан в алгоритм логистической регрессии для создания модели. Для последовательного выполнения всех этих действий воспользуемся оператором pipeline.

```PySpark
tokenizer = Tokenizer(inputCol="violations", outputCol="words")
hashingTF = HashingTF(inputCol=tokenizer.getOutputCol(), outputCol="features")
lr = LogisticRegression(maxIter=10, regParam=0.01)
pipeline = Pipeline(stages=[tokenizer, hashingTF, lr])

model = pipeline.fit(labeledData)
```

## <a name="evaluate-the-model-using-another-dataset"></a>Оценка модели по другому набору данных

Вы можете использовать созданную ранее модель для *прогнозирования* результатов новых проверок. Прогнозы основываются на произошедших нарушениях. Мы обучили эту модель по набору данных **Food_Inspections1.csv**. Вы можете применить другой набор данных, например **Food_Inspections2.csv**, чтобы *оценить* эффективность полученной модели на основе новых данных. Этот новый набор данных (**Food_Inspections2.csv**) следует поместить в контейнер хранилища по умолчанию, связанный с кластером.

1. Выполните приведенный ниже фрагмент кода, чтобы создать кадр данных **predictionsDf** с прогнозами нашей модели. Фрагмент кода также создает временную таблицу **Predictions** на основе кадра данных.

    ```PySpark
    testData = sc.textFile('wasbs:///HdiSamples/HdiSamples/FoodInspectionData/Food_Inspections2.csv')\
                .map(csvParse) \
                .map(lambda l: (int(l[0]), l[1], l[12], l[13]))
    testDf = spark.createDataFrame(testData, schema).where("results = 'Fail' OR results = 'Pass' OR results = 'Pass w/ Conditions'")
    predictionsDf = model.transform(testDf)
    predictionsDf.registerTempTable('Predictions')
    predictionsDf.columns
    ```

    Вы должны увидеть выходные данные, как в следующем тексте:

    ```
    ['id',
        'name',
        'results',
        'violations',
        'words',
        'features',
        'rawPrediction',
        'probability',
        'prediction']
    ```

1. Чтобы просмотреть пример прогноза, выполните этот фрагмент кода:

    ```PySpark
    predictionsDf.take(1)
    ```

   Существует прогноз для первой записи в наборе проверочных данных.

1. Метод `model.transform()` таким же образом преобразовывает новые данные и определяет, как их классифицировать. Чтобы понять, как были выполнены прогнозы, можно выполнить некоторую статистику.

    ```PySpark
    numSuccesses = predictionsDf.where("""(prediction = 0 AND results = 'Fail') OR
                                            (prediction = 1 AND (results = 'Pass' OR
                                                                results = 'Pass w/ Conditions'))""").count()
    numInspections = predictionsDf.count()

    print "There were", numInspections, "inspections and there were", numSuccesses, "successful predictions"
    print "This is a", str((float(numSuccesses) / float(numInspections)) * 100) + "%", "success rate"
    ```

    Выходные данные выглядят так:

    ```
    There were 9315 inspections and there were 8087 successful predictions
    This is a 86.8169618894% success rate
    ```

    Использование логистической регрессии с Spark дает модель связи между описаниями нарушений на английском языке. А также от того, будет ли данный бизнес пройден или не прошел проверку пищи.

## <a name="create-a-visual-representation-of-the-prediction"></a>Создание визуального представления прогноза

Теперь создайте итоговую визуализацию, которая позволит оценить результаты теста.

1. Начнем с извлечения разных прогнозов и результатов из временной таблицы **Predictions**, созданной ранее. Следующие запросы разделяют выходные данные на *true_positive*, *false_positive*, *true_negative* и *false_negative*. В приведенных ниже запросах мы отключим визуализацию с помощью `-q`, а также сохраним выходные данные (с помощью `-o`) как кадры данных, которые затем можно будет использовать с волшебным словом `%%local`.

    ```PySpark
    %%sql -q -o true_positive
    SELECT count(*) AS cnt FROM Predictions WHERE prediction = 0 AND results = 'Fail'
    ```

    ```PySpark
    %%sql -q -o false_positive
    SELECT count(*) AS cnt FROM Predictions WHERE prediction = 0 AND (results = 'Pass' OR results = 'Pass w/ Conditions')
    ```

    ```PySpark
    %%sql -q -o true_negative
    SELECT count(*) AS cnt FROM Predictions WHERE prediction = 1 AND results = 'Fail'
    ```

    ```PySpark
    %%sql -q -o false_negative
    SELECT count(*) AS cnt FROM Predictions WHERE prediction = 1 AND (results = 'Pass' OR results = 'Pass w/ Conditions')
    ```

1. Наконец, используйте следующий фрагмент кода для создания диаграммы с помощью **Matplotlib**.

    ```PySpark
    %%local
    %matplotlib inline
    import matplotlib.pyplot as plt

    labels = ['True positive', 'False positive', 'True negative', 'False negative']
    sizes = [true_positive['cnt'], false_positive['cnt'], false_negative['cnt'], true_negative['cnt']]
    colors = ['turquoise', 'seagreen', 'mediumslateblue', 'palegreen', 'coral']
    plt.pie(sizes, labels=labels, autopct='%1.1f%%', colors=colors)
    plt.axis('equal')
    ```

    Вы должны увидеть следующий результат:

    :::image type="content" source="./media/apache-spark-machine-learning-mllib-ipython/spark-machine-learning-result-output-2.png " alt-text="Выходные данные приложения машинного обучения Spark — процентные доли непройденных проверок пищи." border="true":::

    На этой диаграмме «положительный» результат представляет собой непройденную проверку, а отрицательный — пройденную.

## <a name="shut-down-the-notebook"></a>Завершение работы записной книжки

Завершив работу с приложением, следует закрыть записную книжку, чтобы освободить ресурсы. Для этого в меню **File** (Файл) записной книжки выберите пункт **Close and Halt** (Закрыть и остановить). Записная книжка завершит работу и закроется.

## <a name="next-steps"></a>Дальнейшие действия

* [Обзор: Spark в Azure HDInsight](apache-spark-overview.md)
* [Анализ журналов веб-сайтов с помощью Apache Spark в HDInsight](apache-spark-custom-library-website-log-analysis.md)
* [Microsoft Cognitive Toolkit модели глубокого обучения с помощью Azure HDInsight](apache-spark-microsoft-cognitive-toolkit.md)
