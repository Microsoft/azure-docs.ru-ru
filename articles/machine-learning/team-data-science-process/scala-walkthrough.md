---
title: Обработка и анализ данных с использованием Scala и Spark в Azure — процесс обработки и анализа данных группы
description: Узнайте, как использовать Scala, чтобы выполнять контролируемые задачи машинного обучения, применяя масштабируемую библиотеку машинного обучения (MLlib) Spark и пакеты Spark ML в кластере Spark Azure HDInsight.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 9ae4549fe343422bbf60275a97768ca407f2dc7c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93321374"
---
# <a name="data-science-using-scala-and-spark-on-azure"></a>Обработка и анализ данных с использованием Scala и Spark в Azure
В этой статье приводятся сведения о том, как использовать Scala, чтобы выполнять контролируемые задачи машинного обучения, применяя масштабируемую библиотеку машинного обучения (MLlib) Spark и пакеты Spark ML в кластере Spark Azure HDInsight. Здесь подробно расписаны задачи, которые образуют [процесс обработки и анализа данных](./index.yml): прием и исследование данных, визуализация, проектирование признаков, моделирование и использование моделей. В этой статье использованы такие модели, как логистическая и линейная регрессии, случайные леса и деревья с градиентным повышением (GBT), а также две распространенные контролируемые задачи машинного обучения:

* регрессия: прогнозирование суммы чаевых (в долларах) за поездку в такси;
* двоичная классификация: прогнозирование вероятности оплаты чаевых (1 или 0) за поездку в такси.

Моделирование предусматривает обучение и оценку на основе наборов тестовых данных и соответствующих метрик точности. В этой статье вы узнаете, как сохранять эти модели в хранилище BLOB-объектов Azure, а также оценивать и анализировать производительность прогнозирования. В этой статье также подробнее рассматриваются способы оптимизации моделей с помощью перекрестной проверки и перебора гиперпараметров. В качестве данных используется доступный на сайте GitHub пример с числом поездок и тарифами нью-йоркского такси за 2013 год.

[Scala](https://www.scala-lang.org/)— это язык для виртуальных машин Java, который обеспечивает интеграцию объектно-ориентированного подхода и функциональных качеств языка. Это масштабируемый язык, который отлично подходит для распределенной обработки в облаке и работает в кластерах Spark в Azure.

[Spark](https://spark.apache.org/) — это платформа параллельной обработки с открытым исходным кодом, которая поддерживает обработку в памяти, повышая производительность приложений для анализа больших данных. Подсистема обработки Spark призвана ускорить разработку, повысить удобство использования и реализовать сложную аналитику. Возможности распределенного вычисления в памяти Spark отлично подходят для итеративных алгоритмов в машинном обучении и графовых вычислениях. Пакет [spark.ml](https://spark.apache.org/docs/latest/ml-guide.html) включает в себя единый набор API высокого уровня, созданных поверх кадров данных, позволяющих создавать и настраивать действующие конвейеры машинного обучения. [MLlib](https://spark.apache.org/mllib/) — это масштабируемая библиотека машинного обучения Spark, которая предоставляет возможности моделирования для этой распределенной среды.

[HDInsight Spark](../../hdinsight/spark/apache-spark-overview.md) представляет собой версию платформы Spark с открытым исходным кодом, размещенную в Azure. Он также поддерживает в кластере Spark записные книжки Jupyter на языке Scala и может выполнять интерактивные запросы Spark SQL для преобразования, фильтрации и визуализации данных из хранилища BLOB-объектов Azure. В этой статье фрагменты кода Scala, которые предоставляют решения и формируют соответствующие графики с целью визуализации данных, выполняются в записных книжках Jupyter, установленных в кластерах Spark. Этапы моделирования, описанные в этих разделах, содержат код, который демонстрирует способ обучения, анализа, сохранения и использования каждого типа модели.

Действия по настройке и код, указанные в этой статье, предназначены для HDInsight 3.4 Spark 1.6. Однако код в этой статье и в [записных книжках Jupyter на языке Scala](https://github.com/Azure-Samples/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/Scala/Exploration-Modeling-and-Scoring-using-Scala.ipynb) является универсальным и должен работать в любом кластере Spark. Действия по настройке кластера и управлению им могут немного отличаться от приведенных в этой статье, если вы не используете HDInsight Spark.

> [!NOTE]
> О том, как использовать Python вместо Scala для выполнения задач по полной обработке и анализу данных, см. в статье [Общие сведения об обработке и анализе данных с помощью платформы Spark в Azure HDInsight](spark-overview.md).
> 
> 

## <a name="prerequisites"></a>Предварительные требования
* У вас должна быть подписка Azure. Если у вас ее нет, [получите бесплатную пробную версию Azure](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/).
* Для выполнения дальнейших действия требуется кластер Azure HDInsight 3.4 Spark 1.6. Создайте его, выполнив инструкции в статье [Начало работы. Создание кластера Apache Spark в HDInsight на платформе Linux и выполнение интерактивных запросов с помощью SQL Spark](../../hdinsight/spark/apache-spark-jupyter-spark-sql.md). Укажите тип и версию кластера с помощью меню **Выберите тип кластера** .

![Настройка типа кластера HDInsight](./media/scala-walkthrough/spark-cluster-on-portal.png)

> [!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]
> 
> 

Описание данных о поездках в такси Нью-Йорка и инструкции о том, как выполнить код из записной книжки Jupyter в кластере Spark, см. в соответствующих разделах статьи [Общие сведения об обработке и анализе данных с помощью платформы Spark в Azure HDInsight](spark-overview.md).  

## <a name="execute-scala-code-from-a-jupyter-notebook-on-the-spark-cluster"></a>Выполнение кода Scala из записной книжки Jupyter в кластере Spark
Записную книжку Jupyter можно запустить с портала Azure. На панели мониторинга найдите кластер Spark и щелкните его, чтобы войти на страницу управления кластером. Затем щелкните **Панели мониторинга кластера** и выберите элемент **Записная книжка Jupyter**, чтобы открыть записную книжку, связанную с кластером Spark.

![Панель мониторинга кластера и записные книжки Jupyter](./media/scala-walkthrough/spark-jupyter-on-portal.png)

Записные книжки Jupyter также можно просмотреть по адресу https://&lt;clustername&gt;.azurehdinsight.net/jupyter. Замените *имя_кластера* именем кластера. Для доступа к записным книжкам Jupyter требуется пароль учетной записи администратора.

![Перейти к записным книжкам Jupyter, используя имя кластера](./media/scala-walkthrough/spark-jupyter-notebook.png)

Выберите **Scala** , чтобы открыть каталог, содержащий несколько примеров предварительно упакованных записных книжек, в которых используется API PySpark. Записная книжка "Exploration Modeling and Scoring using Scala.ipynb" с примерами кода для этой группы разделов о Spark доступна на сайте [GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/Spark/Scala).

Записную книжку можно отправить непосредственно из GitHub на сервер записных книжек Jupyter в кластере Spark. На домашней странице записной книжки Jupyter нажмите кнопку **Отправить** . В проводнике вставьте URL-адрес GitHub (необработанное содержимое) для записной книжки Scala и нажмите кнопку **Открыть**. Записная книжка Scala доступна по следующей URL-ссылке:

[Exploration-Modeling-and-Scoring-using-Scala.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/Scala/Exploration-Modeling-and-Scoring-using-Scala.ipynb)

## <a name="setup-preset-spark-and-hive-contexts-spark-magics-and-spark-libraries"></a>Настройка: предустановленные контексты Spark и Hive, а также магические команды и библиотеки Spark
### <a name="preset-spark-and-hive-contexts"></a>Предустановленные контексты Spark и Hive

```scala
# SET THE START TIME
import java.util.Calendar
val beginningTime = Calendar.getInstance().getTime()
```

Ядра Spark, входящие в состав записных книжек Jupyter, имеют предустановленные контексты. Перед началом работы с разрабатываемым приложением вам не нужно явно задавать контексты Spark или Hive. Предустановленные контексты:

* `sc` — для контекста Spark;
* `sqlContext` — для контекста Hive.

### <a name="spark-magics"></a>Волшебные команды Spark
Ядро Spark предусматривает несколько "магических" команд. Это специальные команды, которые можно вызывать, используя `%%`. В следующих примерах кода используются две подобные команды.

* `%%local` указывает, что код в последующих строках будет выполнен локально. Это должен быть допустимый код Scala.
* `%%sql -o <variable name>` выполняет запрос Hive к `sqlContext`. Если передан параметр `-o`, то результат запроса сохраняется в контексте `%%local` Scala в качестве кадра данных Spark.

Дополнительные сведения о ядрах для записных книжек Jupyter и предварительно заданных магических командах, вызываемых с помощью оператора `%%` (например, `%%local`), см. в статье [Ядра, доступные для использования записными книжками Jupyter с кластерами Apache Spark в HDInsight на платформе Linux](../../hdinsight/spark/apache-spark-jupyter-notebook-kernels.md).

### <a name="import-libraries"></a>Импорт библиотек
Импортируйте Spark, MLlib и другие необходимые библиотеки, используя следующий код:

```scala
# IMPORT SPARK AND JAVA LIBRARIES
import org.apache.spark.sql.SQLContext
import org.apache.spark.sql.functions._
import java.text.SimpleDateFormat
import java.util.Calendar
import sqlContext.implicits._
import org.apache.spark.sql.Row

# IMPORT SPARK SQL FUNCTIONS
import org.apache.spark.sql.types.{StructType, StructField, StringType, IntegerType, FloatType, DoubleType}
import org.apache.spark.sql.functions.rand

# IMPORT SPARK ML FUNCTIONS
import org.apache.spark.ml.Pipeline
import org.apache.spark.ml.feature.{StringIndexer, VectorAssembler, OneHotEncoder, VectorIndexer, Binarizer}
import org.apache.spark.ml.tuning.{ParamGridBuilder, TrainValidationSplit, CrossValidator}
import org.apache.spark.ml.regression.{LinearRegression, LinearRegressionModel, RandomForestRegressor, RandomForestRegressionModel, GBTRegressor, GBTRegressionModel}
import org.apache.spark.ml.classification.{LogisticRegression, LogisticRegressionModel, RandomForestClassifier, RandomForestClassificationModel, GBTClassifier, GBTClassificationModel}
import org.apache.spark.ml.evaluation.{BinaryClassificationEvaluator, RegressionEvaluator, MulticlassClassificationEvaluator}

# IMPORT SPARK MLLIB FUNCTIONS
import org.apache.spark.mllib.linalg.{Vector, Vectors}
import org.apache.spark.mllib.util.MLUtils
import org.apache.spark.mllib.classification.{LogisticRegressionWithLBFGS, LogisticRegressionModel}
import org.apache.spark.mllib.regression.{LabeledPoint, LinearRegressionWithSGD, LinearRegressionModel}
import org.apache.spark.mllib.tree.{GradientBoostedTrees, RandomForest}
import org.apache.spark.mllib.tree.configuration.BoostingStrategy
import org.apache.spark.mllib.tree.model.{GradientBoostedTreesModel, RandomForestModel, Predict}
import org.apache.spark.mllib.evaluation.{BinaryClassificationMetrics, MulticlassMetrics, RegressionMetrics}

# SPECIFY SQLCONTEXT
val sqlContext = new SQLContext(sc)
```

## <a name="data-ingestion"></a>Прием данных
Чтобы начать анализ и обработку необходимых данных, требуется сначала принять эти данные. Данные принимаются из внешних источников или систем, в которых они находятся, в среду исследования и моделирования данных. В этой статье принимаемые данные являются объединенной 0,1-процентной выборкой файла данных поездок и тарифов такси (хранящегося в виде TSV-файла). В качестве среды исследования и моделирования данных используется среда Spark. Этот раздел содержит код, с помощью которого можно выполнить следующие задачи:

1. Установка пути к каталогам для хранения данных и моделей.
2. Считывание входного набора данных (TSV-файл).
3. Определение схемы для данных и очистка данных.
4. Создание очищенного кадра данных и его кэширование в памяти.
5. Регистрация данных в виде временной таблицы в SQLContext.
6. Выполнение запроса к таблице и импорт результатов в кадр данных.

### <a name="set-directory-paths-for-storage-locations-in-azure-blob-storage"></a>Настройка путей каталога к местам хранения в хранилище BLOB-объектов Azure
Spark может выполнять чтение и запись данных в хранилище BLOB-объектов Azure. Существующие данные можно обрабатывать с помощью Spark, а затем сохранять полученные данные в хранилище BLOB-объектов.

Чтобы сохранить модели или файлы в хранилище BLOB-объектов, необходимо правильно указать путь. Укажите контейнер по умолчанию, присоединенный к кластеру Spark, с помощью пути, который начинается с `wasb:///`. Другие расположения указываются с помощью `wasb://`.

В следующем примере кода указывается расположение входных данных для чтения и путь к хранилищу BLOB-объектов, присоединенному к кластеру Spark, в котором будет сохранена модель.

```scala
# SET PATHS TO DATA AND MODEL FILE LOCATIONS
# INGEST DATA AND SPECIFY HEADERS FOR COLUMNS
val taxi_train_file = sc.textFile("wasb://mllibwalkthroughs@cdspsparksamples.blob.core.windows.net/Data/NYCTaxi/JoinedTaxiTripFare.Point1Pct.Train.tsv")
val header = taxi_train_file.first;

# SET THE MODEL STORAGE DIRECTORY PATH
# NOTE THAT THE FINAL BACKSLASH IN THE PATH IS REQUIRED.
val modelDir = "wasb:///user/remoteuser/NYCTaxi/Models/";
```

### <a name="import-data-create-an-rdd-and-define-a-data-frame-according-to-the-schema"></a>Импорт данных, создание RDD и определение кадра данных согласно схеме

```scala
# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# DEFINE THE SCHEMA BASED ON THE HEADER OF THE FILE
val sqlContext = new SQLContext(sc)
val taxi_schema = StructType(
    Array(
        StructField("medallion", StringType, true),
        StructField("hack_license", StringType, true),
        StructField("vendor_id", StringType, true),
        StructField("rate_code", DoubleType, true),
        StructField("store_and_fwd_flag", StringType, true),
        StructField("pickup_datetime", StringType, true),
        StructField("dropoff_datetime", StringType, true),
        StructField("pickup_hour", DoubleType, true),
        StructField("pickup_week", DoubleType, true),
        StructField("weekday", DoubleType, true),
        StructField("passenger_count", DoubleType, true),
        StructField("trip_time_in_secs", DoubleType, true),
        StructField("trip_distance", DoubleType, true),
        StructField("pickup_longitude", DoubleType, true),
        StructField("pickup_latitude", DoubleType, true),
        StructField("dropoff_longitude", DoubleType, true),
        StructField("dropoff_latitude", DoubleType, true),
        StructField("direct_distance", StringType, true),
        StructField("payment_type", StringType, true),
        StructField("fare_amount", DoubleType, true),
        StructField("surcharge", DoubleType, true),
        StructField("mta_tax", DoubleType, true),
        StructField("tip_amount", DoubleType, true),
        StructField("tolls_amount", DoubleType, true),
        StructField("total_amount", DoubleType, true),
        StructField("tipped", DoubleType, true),
        StructField("tip_class", DoubleType, true)
        )
    )

# CAST VARIABLES ACCORDING TO THE SCHEMA
val taxi_temp = (taxi_train_file.map(_.split("\t"))
                        .filter((r) => r(0) != "medallion")
                         .map(p => Row(p(0), p(1), p(2),
                            p(3).toDouble, p(4), p(5), p(6), p(7).toDouble, p(8).toDouble, p(9).toDouble, p(10).toDouble,
                            p(11).toDouble, p(12).toDouble, p(13).toDouble, p(14).toDouble, p(15).toDouble, p(16).toDouble,
                            p(17), p(18), p(19).toDouble, p(20).toDouble, p(21).toDouble, p(22).toDouble,
                            p(23).toDouble, p(24).toDouble, p(25).toDouble, p(26).toDouble)))


# CREATE AN INITIAL DATA FRAME AND DROP COLUMNS, AND THEN CREATE A CLEANED DATA FRAME BY FILTERING FOR UNWANTED VALUES OR OUTLIERS
val taxi_train_df = sqlContext.createDataFrame(taxi_temp, taxi_schema)

val taxi_df_train_cleaned = (taxi_train_df.drop(taxi_train_df.col("medallion"))
        .drop(taxi_train_df.col("hack_license")).drop(taxi_train_df.col("store_and_fwd_flag"))
        .drop(taxi_train_df.col("pickup_datetime")).drop(taxi_train_df.col("dropoff_datetime"))
        .drop(taxi_train_df.col("pickup_longitude")).drop(taxi_train_df.col("pickup_latitude"))
        .drop(taxi_train_df.col("dropoff_longitude")).drop(taxi_train_df.col("dropoff_latitude"))
        .drop(taxi_train_df.col("surcharge")).drop(taxi_train_df.col("mta_tax"))
        .drop(taxi_train_df.col("direct_distance")).drop(taxi_train_df.col("tolls_amount"))
        .drop(taxi_train_df.col("total_amount")).drop(taxi_train_df.col("tip_class"))
        .filter("passenger_count > 0 and passenger_count < 8 AND payment_type in ('CSH', 'CRD') AND tip_amount >= 0 AND tip_amount < 30 AND fare_amount >= 1 AND fare_amount < 150 AND trip_distance > 0 AND trip_distance < 100 AND trip_time_in_secs > 30 AND trip_time_in_secs < 7200"));

# CACHE AND MATERIALIZE THE CLEANED DATA FRAME IN MEMORY
taxi_df_train_cleaned.cache()
taxi_df_train_cleaned.count()

# REGISTER THE DATA FRAME AS A TEMPORARY TABLE IN SQLCONTEXT
taxi_df_train_cleaned.registerTempTable("taxi_train")

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");
```

**Выходные данные:**

Время, затраченное на выполнение кода, — 8 секунд.

### <a name="query-the-table-and-import-results-in-a-data-frame"></a>Выполнение запроса к таблице и импорт результатов в кадр данных
Теперь запросите данные о поездках, пассажирах и чаевых из таблицы, отфильтруйте поврежденные и несвойственные данные, а затем выведите несколько строк.

```scala
# QUERY THE DATA
val sqlStatement = """
    SELECT fare_amount, passenger_count, tip_amount, tipped
    FROM taxi_train
    WHERE passenger_count > 0 AND passenger_count < 7
    AND fare_amount > 0 AND fare_amount < 200
    AND payment_type in ('CSH', 'CRD')
    AND tip_amount > 0 AND tip_amount < 25
"""
val sqlResultsDF = sqlContext.sql(sqlStatement)

# SHOW ONLY THE TOP THREE ROWS
sqlResultsDF.show(3)
```

**Выходные данные:**

| fare_amount | passenger_count | tip_amount | tipped |
| --- | --- | --- | --- |
|        13,5 |1.0 |2,9 |1.0 |
|        16,0 |2.0 |3.4 |1.0 |
|        10.5 |2.0 |1.0 |1.0 |

## <a name="data-exploration-and-visualization"></a>Исследование и визуализация данных
После помещения данных в Spark необходимо исследовать и визуализировать их, чтобы получить более глубокое представление. В этом разделе вы изучите данные такси, используя запросы SQL. Затем импортируйте результаты в кадр данных, чтобы отобразить целевые переменные и потенциальные функции для визуальной проверки с помощью функции автоматической визуализации Jupyter.

### <a name="use-local-and-sql-magic-to-plot-data"></a>Использование волшебных команд local и SQL для графического представления данных
По умолчанию выходные данные фрагмента кода, выполняемого из записной книжки Jupyter, доступны в контексте сеанса, сохраненного на рабочих узлах. Если для каждого вычисления нужно сохранить данные о поездке на рабочих узлах и все данные, необходимые для вычислений, доступны локально на узле сервера Jupyter (который является головным), то можно использовать магическую команду `%%local` , чтобы выполнить фрагмент кода на сервере Jupyter.

* **SQL Magic** ( `%%sql` ). Ядро HDInsight Spark позволяет легко выполнять встроенные запросы HiveQL к SQLContext. Аргумент (`-o VARIABLE_NAME`) сохраняет выходные данные запроса SQL в формате кадра данных Pandas на сервере Jupyter. Этот параметр означает, что выходные данные будут доступны в локальном режиме.
* `%%local`**Волшебная**. Магическая команда `%%local` локально выполняет код на сервере Jupyter, который представляет собой головной узел кластера HDInsight. Как правило, команда `%%local` используется в комбинации с магической командой `%%sql` с параметром `-o`. Параметр `-o` сохраняет выходные данные запроса SQL локально, после чего магическая команда `%%local` активирует следующий набор фрагментов кода, который выполняется локально с выходными данными запросов SQL, сохраненными на локальном компьютере.

### <a name="query-the-data-by-using-sql"></a>Выполнение запроса на данные с использованием SQL
Этот запрос извлекает данные о поездках в такси на основе сведений суммы к оплате, количества пассажиров и суммы чаевых.

```scala
# RUN THE SQL QUERY
%%sql -q -o sqlResults
SELECT fare_amount, passenger_count, tip_amount, tipped FROM taxi_train WHERE passenger_count > 0 AND passenger_count < 7 AND fare_amount > 0 AND fare_amount < 200 AND payment_type in ('CSH', 'CRD') AND tip_amount > 0 AND tip_amount < 25
```

В приведенном ниже коде магическая команда `%%local` создает локальный кадр данных, sqlResults. Его можно использовать для построения графического представления с применением matplotlib.

> [!TIP]
> В этой статье магическая команда local используется несколько раз. Если используется большой набор данных, сделайте выборку, чтобы создать кадр данных, который можно сохранить в локальной памяти.
> 
> 

### <a name="plot-the-data"></a>Графическое представление данных
Данные можно графически показать с помощью кода Python, когда таблица данных уже находится в локальном контексте в формате кадра данных Pandas.

```scala
# RUN THE CODE LOCALLY ON THE JUPYTER SERVER
%%local

# USE THE JUPYTER AUTO-PLOTTING FEATURE TO CREATE INTERACTIVE FIGURES.
# CLICK THE TYPE OF PLOT TO GENERATE (LINE, AREA, BAR, ETC.)
sqlResults
```

 Ядро Spark автоматически визуализирует выходные данные запросов SQL (HiveQL) после выполнения кода. Вы можете выбрать тип визуализации среди нескольких типов:

* Таблица
* Круговая
* График
* Область
* Гистограмма

Ниже приведен код для графического представления данных:

```scala
# RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
%%local
import matplotlib.pyplot as plt
%matplotlib inline

# PLOT TIP BY PAYMENT TYPE AND PASSENGER COUNT
ax1 = sqlResults[['tip_amount']].plot(kind='hist', bins=25, facecolor='lightblue')
ax1.set_title('Tip amount distribution')
ax1.set_xlabel('Tip Amount ($)')
ax1.set_ylabel('Counts')
plt.suptitle('')
plt.show()

# PLOT TIP BY PASSENGER COUNT
ax2 = sqlResults.boxplot(column=['tip_amount'], by=['passenger_count'])
ax2.set_title('Tip amount by Passenger count')
ax2.set_xlabel('Passenger count')
ax2.set_ylabel('Tip Amount ($)')
plt.suptitle('')
plt.show()

# PLOT TIP AMOUNT BY FARE AMOUNT; SCALE POINTS BY PASSENGER COUNT
ax = sqlResults.plot(kind='scatter', x= 'fare_amount', y = 'tip_amount', c='blue', alpha = 0.10, s=5*(sqlResults.passenger_count))
ax.set_title('Tip amount by Fare amount')
ax.set_xlabel('Fare Amount ($)')
ax.set_ylabel('Tip Amount ($)')
plt.axis([-2, 80, -2, 20])
plt.show()
```

**Выходные данные:**

![Гистограмма суммы чаевых](./media/scala-walkthrough/plot-tip-amount-histogram.png)

![Сумма чаевых в зависимости от количества пассажиров](./media/scala-walkthrough/plot-tip-amount-by-passenger-count.png)

![Сумма чаевых в зависимости от тарифа](./media/scala-walkthrough/plot-tip-amount-by-fare-amount.png)

## <a name="create-features-and-transform-features-and-then-prep-data-for-input-into-modeling-functions"></a>Создание и преобразование признаков, а также последующая подготовка данных для ввода в функции моделирования
Для использования функций древовидного моделирования из Spark ML и MLlib необходимо подготовить целевые объекты и признаки, применяя различные методы, такие как группирование, индексирование, прямое кодирование и векторизация. В этом разделе рассматриваются следующие процедуры:

1. Создайте новую функцию, **группирования** часы в временные периоды трафика.
2. Применение **индексирования и прямого кодирования** к категориальным признакам.
3. **Выборка и разбиение набора данных** на обучающую и проверочную части.
4. **Указание переменной обучения и признаков**, а затем создание индексированных или прямо закодированных обучающих и проверочных устойчивых распределенных наборов данных (RDD) или входящих кадров данных с помеченной вершиной.
5. Автоматическая **категоризация и векторизация целевых объектов и признаков** для использования в качестве входных данных моделей машинного обучения.

### <a name="create-a-new-feature-by-binning-hours-into-traffic-time-buckets"></a>создание нового признака путем группирования часов в периоды трафика;
В следующем коде показано, как создать признак путем группирования часов в периоды трафика и как кэшировать итоговый кадр данных в памяти. Если RDD и кадры данных используются постоянно, то на их кэширование необходимо меньше времени. Ниже описан процесс кэширования RDD и кадров данных, выполняемый в несколько этапов.

```scala
# CREATE FOUR BUCKETS FOR TRAFFIC TIMES
val sqlStatement = """
    SELECT *,
    CASE
     WHEN (pickup_hour <= 6 OR pickup_hour >= 20) THEN "Night"
     WHEN (pickup_hour >= 7 AND pickup_hour <= 10) THEN "AMRush"
     WHEN (pickup_hour >= 11 AND pickup_hour <= 15) THEN "Afternoon"
     WHEN (pickup_hour >= 16 AND pickup_hour <= 19) THEN "PMRush"
    END as TrafficTimeBins
    FROM taxi_train
"""
val taxi_df_train_with_newFeatures = sqlContext.sql(sqlStatement)

# CACHE THE DATA FRAME IN MEMORY AND MATERIALIZE THE DATA FRAME IN MEMORY
taxi_df_train_with_newFeatures.cache()
taxi_df_train_with_newFeatures.count()
```

### <a name="indexing-and-one-hot-encoding-of-categorical-features"></a>Индексирование и прямое кодирование категориальных признаков
Перед использованием в функциях моделирования и прогнозирования MLlib категориальные входные данные сначала необходимо проиндексировать или закодировать. В этом разделе показано, как индексировать и кодировать категориальные признаки в качестве ввода для функций моделирования.

В зависимости от модели этот процесс происходит по-разному. Например, для моделей логистической и линейной регрессии требуется прямое кодирование. Скажем, признак с тремя категориями можно представить в виде трех столбцов признаков. Каждый столбец будет содержать 0 или 1 в зависимости от категории наблюдения. Для прямого кодирования можно использовать функцию [OneHotEncoder](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OneHotEncoder.html#sklearn.preprocessing.OneHotEncoder) в MLlib. Этот кодировщик сопоставляет столбец с индексами меток со столбцом двоичных векторов, имеющих максимум одно отдельное значение. Благодаря этой кодировке алгоритмы, для которых необходимы признаки с числовыми значениями (например, логистическая регрессия), можно применять к категориальным признакам.

Здесь вы преобразуете только четыре переменных, чтобы показать примеры, которые представляют собой строки символов. Вы также можете индексировать другие переменные, такие как день недели, представленные числовыми значениями, в качестве категориальных переменных.

Для индексирования используйте функцию `StringIndexer()`, а для прямого кодирования — `OneHotEncoder()` из MLlib. Ниже приведен пример кода, с помощью которого можно проиндексировать и закодировать категориальные признаки.

```scala
# CREATE INDEXES AND ONE-HOT ENCODED VECTORS FOR SEVERAL CATEGORICAL FEATURES

# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# INDEX AND ENCODE VENDOR_ID
val stringIndexer = new StringIndexer().setInputCol("vendor_id").setOutputCol("vendorIndex").fit(taxi_df_train_with_newFeatures)
val indexed = stringIndexer.transform(taxi_df_train_with_newFeatures)
val encoder = new OneHotEncoder().setInputCol("vendorIndex").setOutputCol("vendorVec")
val encoded1 = encoder.transform(indexed)

# INDEX AND ENCODE RATE_CODE
val stringIndexer = new StringIndexer().setInputCol("rate_code").setOutputCol("rateIndex").fit(encoded1)
val indexed = stringIndexer.transform(encoded1)
val encoder = new OneHotEncoder().setInputCol("rateIndex").setOutputCol("rateVec")
val encoded2 = encoder.transform(indexed)

# INDEX AND ENCODE PAYMENT_TYPE
val stringIndexer = new StringIndexer().setInputCol("payment_type").setOutputCol("paymentIndex").fit(encoded2)
val indexed = stringIndexer.transform(encoded2)
val encoder = new OneHotEncoder().setInputCol("paymentIndex").setOutputCol("paymentVec")
val encoded3 = encoder.transform(indexed)

# INDEX AND TRAFFIC TIME BINS
val stringIndexer = new StringIndexer().setInputCol("TrafficTimeBins").setOutputCol("TrafficTimeBinsIndex").fit(encoded3)
val indexed = stringIndexer.transform(encoded3)
val encoder = new OneHotEncoder().setInputCol("TrafficTimeBinsIndex").setOutputCol("TrafficTimeBinsVec")
val encodedFinal = encoder.transform(indexed)

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");
```

**Выходные данные:**

Время, затраченное на выполнение кода, — 4 секунды.

### <a name="sample-and-split-the-data-set-into-training-and-test-fractions"></a>Выборка и разбиение набора данных на обучающую и проверочную части
С помощью кода в этом разделе создается случайная выборка данных (в примере используется 25 %). Несмотря на то, что из-за небольшого размера используемого набора данных выполнять выборку необязательно, данная статья покажет, как ее делать. Это поможет вам в случае возникновения проблем. Если выборки большие, этот процесс может значительно сэкономить время обучения моделей. Затем мы разобьем выборку данных на наборы для обучения (в примере используется 75 %) и тестирования (25 % соответственно), которые будут использоваться для классификации и регрессии.

Добавьте случайное число (от 0 до 1) в каждую строку (в столбце rand), которую можно использовать для выбора сверток перекрестной проверки во время обучения.

```scala
# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# SPECIFY SAMPLING AND SPLITTING FRACTIONS
val samplingFraction = 0.25;
val trainingFraction = 0.75;
val testingFraction = (1-trainingFraction);
val seed = 1234;
val encodedFinalSampledTmp = encodedFinal.sample(withReplacement = false, fraction = samplingFraction, seed = seed)
val sampledDFcount = encodedFinalSampledTmp.count().toInt

val generateRandomDouble = udf(() => {
    scala.util.Random.nextDouble
})

# ADD A RANDOM NUMBER FOR CROSS-VALIDATION
val encodedFinalSampled = encodedFinalSampledTmp.withColumn("rand", generateRandomDouble());

# SPLIT THE SAMPLED DATA FRAME INTO TRAIN AND TEST, WITH A RANDOM COLUMN ADDED FOR DOING CROSS-VALIDATION (SHOWN LATER)
# INCLUDE A RANDOM COLUMN FOR CREATING CROSS-VALIDATION FOLDS
val splits = encodedFinalSampled.randomSplit(Array(trainingFraction, testingFraction), seed = seed)
val trainData = splits(0)
val testData = splits(1)

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");
```

**Выходные данные:**

Время, затраченное на выполнение кода, — 2 секунды.

### <a name="specify-training-variable-and-features-and-then-create-indexed-or-one-hot-encoded-training-and-testing-input-labeled-point-rdds-or-data-frames"></a>Указание переменной обучения и признаков, а также последующее создание индексированных или прямо закодированных обучающих и проверочных RDD или входящих кадров данных с помеченной вершиной
В этом разделе содержится код, с помощью которого можно индексировать категориальные текстовые данные в тип данных с помеченной вершиной, а затем закодировать их. После этого данные можно использовать для обучения и проверки модели логистической регрессии MLlib и других моделей классификации. Объекты с помеченной вершиной отформатированы в RDD, которые можно использовать в качестве входных данных для большинства алгоритмов машинного обучения в MLlib. Объект [с помеченной вершиной](https://spark.apache.org/docs/latest/mllib-data-types.html#labeled-point) — это разреженный или плотный локальный вектор, связанный с меткой или ответом.

В этом коде вы указываете целевую (зависимую) переменную и признаки, которые будут использоваться для обучения моделей. Затем вы создаете индексированные или прямо закодированные обучающие и проверочные RDD или входящие кадры данных с помеченной вершиной.

```scala
# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# MAP NAMES OF FEATURES AND TARGETS FOR CLASSIFICATION AND REGRESSION PROBLEMS
val featuresIndOneHot = List("paymentVec", "vendorVec", "rateVec", "TrafficTimeBinsVec", "pickup_hour", "weekday", "passenger_count", "trip_time_in_secs", "trip_distance", "fare_amount").map(encodedFinalSampled.columns.indexOf(_))
val featuresIndIndex = List("paymentIndex", "vendorIndex", "rateIndex", "TrafficTimeBinsIndex", "pickup_hour", "weekday", "passenger_count", "trip_time_in_secs", "trip_distance", "fare_amount").map(encodedFinalSampled.columns.indexOf(_))

# SPECIFY THE TARGET FOR CLASSIFICATION ('tipped') AND REGRESSION ('tip_amount') PROBLEMS
val targetIndBinary = List("tipped").map(encodedFinalSampled.columns.indexOf(_))
val targetIndRegression = List("tip_amount").map(encodedFinalSampled.columns.indexOf(_))

# CREATE INDEXED LABELED POINT RDD OBJECTS
val indexedTRAINbinary = trainData.rdd.map(r => LabeledPoint(r.getDouble(targetIndBinary(0).toInt), Vectors.dense(featuresIndIndex.map(r.getDouble(_)).toArray)))
val indexedTESTbinary = testData.rdd.map(r => LabeledPoint(r.getDouble(targetIndBinary(0).toInt), Vectors.dense(featuresIndIndex.map(r.getDouble(_)).toArray)))
val indexedTRAINreg = trainData.rdd.map(r => LabeledPoint(r.getDouble(targetIndRegression(0).toInt), Vectors.dense(featuresIndIndex.map(r.getDouble(_)).toArray)))
val indexedTESTreg = testData.rdd.map(r => LabeledPoint(r.getDouble(targetIndRegression(0).toInt), Vectors.dense(featuresIndIndex.map(r.getDouble(_)).toArray)))

# CREATE INDEXED DATA FRAMES THAT YOU CAN USE TO TRAIN BY USING SPARK ML FUNCTIONS
val indexedTRAINbinaryDF = indexedTRAINbinary.toDF()
val indexedTESTbinaryDF = indexedTESTbinary.toDF()
val indexedTRAINregDF = indexedTRAINreg.toDF()
val indexedTESTregDF = indexedTESTreg.toDF()

# CREATE ONE-HOT ENCODED (VECTORIZED) DATA FRAMES THAT YOU CAN USE TO TRAIN BY USING SPARK ML FUNCTIONS
val assemblerOneHot = new VectorAssembler().setInputCols(Array("paymentVec", "vendorVec", "rateVec", "TrafficTimeBinsVec", "pickup_hour", "weekday", "passenger_count", "trip_time_in_secs", "trip_distance", "fare_amount")).setOutputCol("features")
val OneHotTRAIN = assemblerOneHot.transform(trainData)
val OneHotTEST = assemblerOneHot.transform(testData)

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");
```

**Выходные данные:**

Время, затраченное на выполнение кода, — 4 секунды.

### <a name="automatically-categorize-and-vectorize-features-and-targets-to-use-as-inputs-for-machine-learning-models"></a>Автоматическая категоризация и векторизация целевых объектов и признаков для использования в качестве входных данных моделей машинного обучения
С помощью Spark ML категоризируйте целевой объект и признаки, которые нужно использовать в функциях древовидного моделирования. Код выполняет две задачи:

* Создает двоичный целевой объект для классификации, присваивая значение 0 или 1 каждой точке данных со значением от 0 до 1, используя пороговое значение 0,5.
* Автоматически категоризирует признаки. Если количество уникальных числовых значений для любого из признаков меньше 32, то этот признак категоризирован.

Ниже приведен код для этих двух задач.

```scala
# CATEGORIZE FEATURES AND BINARIZE THE TARGET FOR THE BINARY CLASSIFICATION PROBLEM

# TRAIN DATA
val indexer = new VectorIndexer().setInputCol("features").setOutputCol("featuresCat").setMaxCategories(32)
val indexerModel = indexer.fit(indexedTRAINbinaryDF)
val indexedTrainwithCatFeat = indexerModel.transform(indexedTRAINbinaryDF)
val binarizer: Binarizer = new Binarizer().setInputCol("label").setOutputCol("labelBin").setThreshold(0.5)
val indexedTRAINwithCatFeatBinTarget = binarizer.transform(indexedTrainwithCatFeat)

# TEST DATA
val indexerModel = indexer.fit(indexedTESTbinaryDF)
val indexedTrainwithCatFeat = indexerModel.transform(indexedTESTbinaryDF)
val binarizer: Binarizer = new Binarizer().setInputCol("label").setOutputCol("labelBin").setThreshold(0.5)
val indexedTESTwithCatFeatBinTarget = binarizer.transform(indexedTrainwithCatFeat)

# CATEGORIZE FEATURES FOR THE REGRESSION PROBLEM
# CREATE PROPERLY INDEXED AND CATEGORIZED DATA FRAMES FOR TREE-BASED MODELS

# TRAIN DATA
val indexer = new VectorIndexer().setInputCol("features").setOutputCol("featuresCat").setMaxCategories(32)
val indexerModel = indexer.fit(indexedTRAINregDF)
val indexedTRAINwithCatFeat = indexerModel.transform(indexedTRAINregDF)

# TEST DATA
val indexerModel = indexer.fit(indexedTESTbinaryDF)
val indexedTESTwithCatFeat = indexerModel.transform(indexedTESTregDF)
```


## <a name="binary-classification-model-predict-whether-a-tip-should-be-paid"></a>Модель двоичной классификации: прогнозирование вероятности выплаты чаевых за поездку
В этом разделе создаются модели двоичной классификации трех типов, чтобы составить прогноз того, будут ли выплачены чаевые за поездку:

* **модель логистической регрессии**, используя функцию Spark ML `LogisticRegression()`;
* **модель классификации случайного леса**, используя функцию Spark ML `RandomForestClassifier()`;
* **модель классификации по методу деревьев** с градиентным повышением, используя функцию MLlib `GradientBoostedTrees()`.

### <a name="create-a-logistic-regression-model"></a>создание модели логистической регрессии;
Теперь создайте модель логистической регрессии, используя функцию Spark ML `LogisticRegression()` . Создание кода сборки модели предполагает такую последовательность шагов:

1. **Обучение модели** с использованием данных с одним набором параметров.
2. **Оценка модели** на основе набора тестовых данных с метриками.
3. **Сохранение модели** в хранилище BLOB-объектов для последующего использования.
4. **Оценка модели** на основе тестовых данных.
5. **Графическое представление результатов** в виде кривых ROC (рабочих характеристик приемника).

Ниже приведен код для выполнения этих процедур.

```scala
# CREATE A LOGISTIC REGRESSION MODEL
val lr = new LogisticRegression().setLabelCol("tipped").setFeaturesCol("features").setMaxIter(10).setRegParam(0.3).setElasticNetParam(0.8)
val lrModel = lr.fit(OneHotTRAIN)

# PREDICT ON THE TEST DATA SET
val predictions = lrModel.transform(OneHotTEST)

# SELECT `BinaryClassificationEvaluator()` TO COMPUTE THE TEST ERROR
val evaluator = new BinaryClassificationEvaluator().setLabelCol("tipped").setRawPredictionCol("probability").setMetricName("areaUnderROC")
val ROC = evaluator.evaluate(predictions)
println("ROC on test data = " + ROC)

# SAVE THE MODEL
val datestamp = Calendar.getInstance().getTime().toString.replaceAll(" ", ".").replaceAll(":", "_");
val modelName = "LogisticRegression__"
val filename = modelDir.concat(modelName).concat(datestamp)
lrModel.save(filename);
```

Загрузите, оцените и сохраните результаты.

```scala
# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# LOAD THE SAVED MODEL AND SCORE THE TEST DATA SET
val savedModel = org.apache.spark.ml.classification.LogisticRegressionModel.load(filename)
println(s"Coefficients: ${savedModel.coefficients} Intercept: ${savedModel.intercept}")

# SCORE THE MODEL ON THE TEST DATA
val predictions = savedModel.transform(OneHotTEST).select("tipped","probability","rawPrediction")
predictions.registerTempTable("testResults")

# SELECT `BinaryClassificationEvaluator()` TO COMPUTE THE TEST ERROR
val evaluator = new BinaryClassificationEvaluator().setLabelCol("tipped").setRawPredictionCol("probability").setMetricName("areaUnderROC")
val ROC = evaluator.evaluate(predictions)

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");

# PRINT THE ROC RESULTS
println("ROC on test data = " + ROC)
```

**Выходные данные:**

ROC для тестовых данных — 0,9827381497557599.

Примените код Python к кадрам данных Pandas, чтобы построить кривую ROC.

```scala
# QUERY THE RESULTS
%%sql -q -o sqlResults
SELECT tipped, probability from testResults


# RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
%%local
%matplotlib inline
from sklearn.metrics import roc_curve,auc

sqlResults['probFloat'] = sqlResults.apply(lambda row: row['probability'].values()[0][1], axis=1)
predictions_pddf = sqlResults[["tipped","probFloat"]]

# PREDICT THE ROC CURVE
# predictions_pddf = sqlResults.rename(columns={'_1': 'probability', 'tipped': 'label'})
prob = predictions_pddf["probFloat"]
fpr, tpr, thresholds = roc_curve(predictions_pddf['tipped'], prob, pos_label=1);
roc_auc = auc(fpr, tpr)

# PLOT THE ROC CURVE
plt.figure(figsize=(5,5))
plt.plot(fpr, tpr, label='ROC curve (area = %0.2f)' % roc_auc)
plt.plot([0, 1], [0, 1], 'k--')
plt.xlim([0.0, 1.0])
plt.ylim([0.0, 1.05])
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.title('ROC Curve')
plt.legend(loc="lower right")
plt.show()
```

**Выходные данные:**

![Кривая ROC вероятности выплаты чаевых](./media/scala-walkthrough/plot-roc-curve-tip-or-not.png)

### <a name="create-a-random-forest-classification-model"></a>Создание модели классификации случайного леса
Затем создайте модель классификации случайного леса, используя функцию Spark ML `RandomForestClassifier()` , и оцените модель на основе тестовых данных.

```scala
# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# CREATE THE RANDOM FOREST CLASSIFIER MODEL
val rf = new RandomForestClassifier().setLabelCol("labelBin").setFeaturesCol("featuresCat").setNumTrees(10).setSeed(1234)

# FIT THE MODEL
val rfModel = rf.fit(indexedTRAINwithCatFeatBinTarget)
val predictions = rfModel.transform(indexedTESTwithCatFeatBinTarget)

# EVALUATE THE MODEL
val evaluator = new MulticlassClassificationEvaluator().setLabelCol("label").setPredictionCol("prediction").setMetricName("f1")
val Test_f1Score = evaluator.evaluate(predictions)
println("F1 score on test data: " + Test_f1Score);

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");

# CALCULATE BINARY CLASSIFICATION EVALUATION METRICS
val evaluator = new BinaryClassificationEvaluator().setLabelCol("label").setRawPredictionCol("probability").setMetricName("areaUnderROC")
val ROC = evaluator.evaluate(predictions)
println("ROC on test data = " + ROC)
```

**Выходные данные:**

ROC для тестовых данных — 0,9847103571552683.

### <a name="create-a-gbt-classification-model"></a>Создание модели классификации по методу деревьев с градиентным повышением
Теперь создайте модель классификации по методу деревьев с градиентным повышением (GBT), используя функцию MLlib `GradientBoostedTrees()` , и оцените модель на основе тестовых данных.

```scala
# TRAIN A GBT CLASSIFICATION MODEL BY USING MLLIB AND A LABELED POINT

# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# DEFINE THE GBT CLASSIFICATION MODEL
val boostingStrategy = BoostingStrategy.defaultParams("Classification")
boostingStrategy.numIterations = 20
boostingStrategy.treeStrategy.numClasses = 2
boostingStrategy.treeStrategy.maxDepth = 5
boostingStrategy.treeStrategy.categoricalFeaturesInfo = Map[Int, Int]((0,2),(1,2),(2,6),(3,4))

# TRAIN THE MODEL
val gbtModel = GradientBoostedTrees.train(indexedTRAINbinary, boostingStrategy)

# SAVE THE MODEL IN BLOB STORAGE
val datestamp = Calendar.getInstance().getTime().toString.replaceAll(" ", ".").replaceAll(":", "_");
val modelName = "GBT_Classification__"
val filename = modelDir.concat(modelName).concat(datestamp)
gbtModel.save(sc, filename);

# EVALUATE THE MODEL ON TEST INSTANCES AND THE COMPUTE TEST ERROR
val labelAndPreds = indexedTESTbinary.map { point =>
  val prediction = gbtModel.predict(point.features)
  (point.label, prediction)
}
val testErr = labelAndPreds.filter(r => r._1 != r._2).count.toDouble / indexedTRAINbinary.count()
//println("Learned classification GBT model:\n" + gbtModel.toDebugString)
println("Test Error = " + testErr)

# USE BINARY AND MULTICLASS METRICS TO EVALUATE THE MODEL ON THE TEST DATA
val metrics = new MulticlassMetrics(labelAndPreds)
println(s"Precision: ${metrics.precision}")
println(s"Recall: ${metrics.recall}")
println(s"F1 Score: ${metrics.fMeasure}")

val metrics = new BinaryClassificationMetrics(labelAndPreds)
println(s"Area under PR curve: ${metrics.areaUnderPR}")
println(s"Area under ROC curve: ${metrics.areaUnderROC}")

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");

# PRINT THE ROC METRIC
println(s"Area under ROC curve: ${metrics.areaUnderROC}")
```

**Выходные данные:**

площадь под ROC — 0,9846895479241554.

## <a name="regression-model-predict-tip-amount"></a>Модель регрессии: прогнозирование суммы чаевых
В этом разделе создаются модели регрессии двух типов, чтобы спрогнозировать сумму чаевых:

* **модель регуляризованной линейной регрессии**, используя функцию Spark ML `LinearRegression()`; модель сохраняется и оценивается на основе тестовых данных;
* **модель регрессии по методу деревьев с градиентным повышением**, используя функцию Spark ML `GBTRegressor()`.

### <a name="create-a-regularized-linear-regression-model"></a>Создание регуляризованной линейной регрессии

```scala
# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# CREATE A REGULARIZED LINEAR REGRESSION MODEL BY USING THE SPARK ML FUNCTION AND DATA FRAMES
val lr = new LinearRegression().setLabelCol("tip_amount").setFeaturesCol("features").setMaxIter(10).setRegParam(0.3).setElasticNetParam(0.8)

# FIT THE MODEL BY USING DATA FRAMES
val lrModel = lr.fit(OneHotTRAIN)
println(s"Coefficients: ${lrModel.coefficients} Intercept: ${lrModel.intercept}")

# SUMMARIZE THE MODEL OVER THE TRAINING SET AND PRINT METRICS
val trainingSummary = lrModel.summary
println(s"numIterations: ${trainingSummary.totalIterations}")
println(s"objectiveHistory: ${trainingSummary.objectiveHistory.toList}")
trainingSummary.residuals.show()
println(s"RMSE: ${trainingSummary.rootMeanSquaredError}")
println(s"r2: ${trainingSummary.r2}")

# SAVE THE MODEL IN AZURE BLOB STORAGE
val datestamp = Calendar.getInstance().getTime().toString.replaceAll(" ", ".").replaceAll(":", "_");
val modelName = "LinearRegression__"
val filename = modelDir.concat(modelName).concat(datestamp)
lrModel.save(filename);

# PRINT THE COEFFICIENTS
println(s"Coefficients: ${lrModel.coefficients} Intercept: ${lrModel.intercept}")

# SCORE THE MODEL ON TEST DATA
val predictions = lrModel.transform(OneHotTEST)

# EVALUATE THE MODEL ON TEST DATA
val evaluator = new RegressionEvaluator().setLabelCol("tip_amount").setPredictionCol("prediction").setMetricName("r2")
val r2 = evaluator.evaluate(predictions)
println("R-sqr on test data = " + r2)

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");
```


**Выходные данные:**

Время, затраченное на выполнение кода, — 13 секунд.

```scala
# LOAD A SAVED LINEAR REGRESSION MODEL FROM BLOB STORAGE AND SCORE A TEST DATA SET

# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# LOAD A SAVED LINEAR REGRESSION MODEL FROM AZURE BLOB STORAGE
val savedModel = org.apache.spark.ml.regression.LinearRegressionModel.load(filename)
println(s"Coefficients: ${savedModel.coefficients} Intercept: ${savedModel.intercept}")

# SCORE THE MODEL ON TEST DATA
val predictions = savedModel.transform(OneHotTEST).select("tip_amount","prediction")
predictions.registerTempTable("testResults")

# EVALUATE THE MODEL ON TEST DATA
val evaluator = new RegressionEvaluator().setLabelCol("tip_amount").setPredictionCol("prediction").setMetricName("r2")
val r2 = evaluator.evaluate(predictions)
println("R-sqr on test data = " + r2)

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");

# PRINT THE RESULTS
println("R-sqr on test data = " + r2)
```

**Выходные данные:**

R-sqr для тестовых данных — 0,5960320470835743.

Теперь запросите результаты теста в формате кадра данных и визуализируйте их с помощью AutoVizWidget и matplotlib.

```sql
# RUN A SQL QUERY
%%sql -q -o sqlResults
select * from testResults

# RUN THE CODE LOCALLY ON THE JUPYTER SERVER
%%local

# USE THE JUPYTER AUTO-PLOTTING FEATURE TO CREATE INTERACTIVE FIGURES
# CLICK THE TYPE OF PLOT TO GENERATE (LINE, AREA, BAR, AND SO ON)
sqlResults
```

Этот код создает локальный кадр данных из выходных данных запроса и формирует графическое представление данных. Магическая команда `%%local` создает локальный кадр данных `sqlResults`, который можно использовать для формирования графического представления данных с помощью matplotlib.

> [!NOTE]
> В этой статье данная магическая команда Spark используется несколько раз. Если объем данных большой, сделайте выборку, чтобы создать кадр данных, который можно разместить в локальной памяти.
> 
> 

Создайте график с помощью "matplotlib" Python.

```scala
# RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
%%local
sqlResults
%matplotlib inline
import numpy as np

# PLOT THE RESULTS
ax = sqlResults.plot(kind='scatter', figsize = (6,6), x='tip_amount', y='prediction', color='blue', alpha = 0.25, label='Actual vs. predicted');
fit = np.polyfit(sqlResults['tip_amount'], sqlResults['prediction'], deg=1)
ax.set_title('Actual vs. Predicted Tip Amounts ($)')
ax.set_xlabel("Actual")
ax.set_ylabel("Predicted")
#ax.plot(sqlResults['tip_amount'], fit[0] * sqlResults['prediction'] + fit[1], color='magenta')
plt.axis([-1, 15, -1, 8])
plt.show(ax)
```

**Выходные данные:**

![Сумма чаевых: соотношение фактической и прогнозируемой суммы](./media/scala-walkthrough/plot-actual-vs-predicted-tip-amount.png)

### <a name="create-a-gbt-regression-model"></a>Создание модели регрессии GBT
Создайте модель регрессии по методу деревьев с градиентным повышением (GBT), используя функцию Spark ML `GBTRegressor()` , и оцените модель на основе тестовых данных.

[Деревья с градиентным ускорением](https://spark.apache.org/docs/latest/ml-classification-regression.html#gradient-boosted-trees-gbts) (GBT) — это это совокупности деревьев принятия решений. GBTное обучение деревьев принятия решений для сворачивания функции потери. GBT можно использовать для регрессии и классификации. С их помощью можно обрабатывать категориальные признаки, а также определять нелинейные зависимости и взаимодействия признаков. Этот метод не требует масштабирования признаков. Кроме того, их можно использовать для создания модели мультиклассовой классификации.

```scala
# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# TRAIN A GBT REGRESSION MODEL
val gbt = new GBTRegressor().setLabelCol("label").setFeaturesCol("featuresCat").setMaxIter(10)
val gbtModel = gbt.fit(indexedTRAINwithCatFeat)

# MAKE PREDICTIONS
val predictions = gbtModel.transform(indexedTESTwithCatFeat)

# COMPUTE TEST SET R2
val evaluator = new RegressionEvaluator().setLabelCol("label").setPredictionCol("prediction").setMetricName("r2")
val Test_R2 = evaluator.evaluate(predictions)


# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");

# PRINT THE RESULTS
println("Test R-sqr is: " + Test_R2);
```

**Выходные данные:**

Test R-sqr — 0,7655383534596654.

## <a name="advanced-modeling-utilities-for-optimization"></a>Служебные программы расширенного моделирования для оптимизации
В этом разделе используются служебные программы машинного обучения, которые часто применяют для оптимизации моделей. В частности, есть три разных способа оптимизации моделей машинного обучения с использованием перебора параметров и перекрестной проверки.

* Разбиение данных на наборы данных для обучения и для проверки, оптимизация модели с использованием перебора гиперпараметров набора данных для обучения и оценки проверочного набора (линейная регрессия).
* Оптимизация модели с помощью перекрестной проверки и перебора гиперпараметров с использованием функции Spark ML "CrossValidator" (двоичная классификация).
* Оптимизация модели с помощью настраиваемого кода перекрестной проверки и перебора параметров, чтобы она использовала любые функции машинного обучения и наборы параметров (линейная регрессия).

**Перекрестная проверка** — это метод, который позволяет оценить, насколько хорошо модель, обученная на известном наборе данных, будет обобщаться для прогнозирования признаков наборов данных, на которых она не прошла обучение. Суть этого метода в том, что модель обучается на наборе известных данных, а затем точность ее прогнозов тестируется на независимом наборе данных. Это распространенная реализация, разделяющая набор данных на *k*-свертки и затем обучающая модель путем циклического перебора по всем сверткам, кроме одной.

**Оптимизация параметров Hyper-Parameter** — это проблема выбора набора параметров Hyper-by для алгоритма обучения, обычно с целью оптимизации показателя производительности алгоритма на независимом наборе данных. Гиперпараметр — это значение, которое должно быть указано за пределами процедуры обучения модели. Предположения относительно значений гиперпараметров могут повлиять на гибкость и точность моделей. Деревья принятия решений содержат гиперпараметры, такие как требуемая глубина и число листьев на дереве. Для метода опорных векторов (SVM) необходимо задать условие штрафа за неправильную классификацию.

Распространенный способ выполнения оптимизации гиперпараметров — использовать поиск в сетке. Его также называют **перебором параметров**. Поиск в сетке заключается в выполнении тщательного поиска по значениям указанного подмножества пространства гиперпараметров для алгоритма обучения. Перекрестная проверка может предоставить метрику производительности, чтобы отсортировать оптимальные результаты, выданные алгоритмом поиска в сетке. Использование перекрестной проверки с перебором гиперпараметров помогает ограничить такие проблемы, как лжевзаимосвязи модели с обучающими данными. При этом модель сохраняет возможность применения к общему набору данных, из которого извлекаются данные для обучения.

### <a name="optimize-a-linear-regression-model-with-hyper-parameter-sweeping"></a>Оптимизация модели линейной регрессии с помощью перебора гиперпараметров
Теперь разбейте данные на наборы данных для обучения и для проверки, используйте перебор гиперпараметров набора данных для обучения, чтобы оптимизировать модель, и оцените проверочный набор (линейная регрессия).

```scala
# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# RENAME `tip_amount` AS A LABEL
val OneHotTRAINLabeled = OneHotTRAIN.select("tip_amount","features").withColumnRenamed(existingName="tip_amount",newName="label")
val OneHotTESTLabeled = OneHotTEST.select("tip_amount","features").withColumnRenamed(existingName="tip_amount",newName="label")
OneHotTRAINLabeled.cache()
OneHotTESTLabeled.cache()

# DEFINE THE ESTIMATOR FUNCTION: `THE LinearRegression()` FUNCTION
val lr = new LinearRegression().setLabelCol("label").setFeaturesCol("features").setMaxIter(10)

# DEFINE THE PARAMETER GRID
val paramGrid = new ParamGridBuilder().addGrid(lr.regParam, Array(0.1, 0.01, 0.001)).addGrid(lr.fitIntercept).addGrid(lr.elasticNetParam, Array(0.1, 0.5, 0.9)).build()

# DEFINE THE PIPELINE WITH A TRAIN/TEST VALIDATION SPLIT (75% IN THE TRAINING SET), AND THEN THE SPECIFY ESTIMATOR, EVALUATOR, AND PARAMETER GRID
val trainPct = 0.75
val trainValidationSplit = new TrainValidationSplit().setEstimator(lr).setEvaluator(new RegressionEvaluator).setEstimatorParamMaps(paramGrid).setTrainRatio(trainPct)

# RUN THE TRAIN VALIDATION SPLIT AND CHOOSE THE BEST SET OF PARAMETERS
val model = trainValidationSplit.fit(OneHotTRAINLabeled)

# MAKE PREDICTIONS ON THE TEST DATA BY USING THE MODEL WITH THE COMBINATION OF PARAMETERS THAT PERFORMS THE BEST
val testResults = model.transform(OneHotTESTLabeled).select("label", "prediction")

# COMPUTE TEST SET R2
val evaluator = new RegressionEvaluator().setLabelCol("label").setPredictionCol("prediction").setMetricName("r2")
val Test_R2 = evaluator.evaluate(testResults)

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");

println("Test R-sqr is: " + Test_R2);
```

**Выходные данные:**

Test R-sqr — 0,6226484708501209

### <a name="optimize-the-binary-classification-model-by-using-cross-validation-and-hyper-parameter-sweeping"></a>Оптимизация модели двоичной классификации с помощью перекрестной проверки и перебора гиперпараметров
В этом разделе показано, как оптимизировать модель двоичной классификации с помощью перекрестной проверки и перебора гиперпараметров. Для этого используется функция Spark ML `CrossValidator` .

```scala
# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# CREATE DATA FRAMES WITH PROPERLY LABELED COLUMNS TO USE WITH THE TRAIN AND TEST SPLIT
val indexedTRAINwithCatFeatBinTargetRF = indexedTRAINwithCatFeatBinTarget.select("labelBin","featuresCat").withColumnRenamed(existingName="labelBin",newName="label").withColumnRenamed(existingName="featuresCat",newName="features")
val indexedTESTwithCatFeatBinTargetRF = indexedTESTwithCatFeatBinTarget.select("labelBin","featuresCat").withColumnRenamed(existingName="labelBin",newName="label").withColumnRenamed(existingName="featuresCat",newName="features")
indexedTRAINwithCatFeatBinTargetRF.cache()
indexedTESTwithCatFeatBinTargetRF.cache()

# DEFINE THE ESTIMATOR FUNCTION
val rf = new RandomForestClassifier().setLabelCol("label").setFeaturesCol("features").setImpurity("gini").setSeed(1234).setFeatureSubsetStrategy("auto").setMaxBins(32)

# DEFINE THE PARAMETER GRID
val paramGrid = new ParamGridBuilder().addGrid(rf.maxDepth, Array(4,8)).addGrid(rf.numTrees, Array(5,10)).addGrid(rf.minInstancesPerNode, Array(100,300)).build()

# SPECIFY THE NUMBER OF FOLDS
val numFolds = 3

# DEFINE THE TRAIN/TEST VALIDATION SPLIT (75% IN THE TRAINING SET)
val CrossValidator = new CrossValidator().setEstimator(rf).setEvaluator(new BinaryClassificationEvaluator).setEstimatorParamMaps(paramGrid).setNumFolds(numFolds)

# RUN THE TRAIN VALIDATION SPLIT AND CHOOSE THE BEST SET OF PARAMETERS
val model = CrossValidator.fit(indexedTRAINwithCatFeatBinTargetRF)

# MAKE PREDICTIONS ON THE TEST DATA BY USING THE MODEL WITH THE COMBINATION OF PARAMETERS THAT PERFORMS THE BEST
val testResults = model.transform(indexedTESTwithCatFeatBinTargetRF).select("label", "prediction")

# COMPUTE THE TEST F1 SCORE
val evaluator = new MulticlassClassificationEvaluator().setLabelCol("label").setPredictionCol("prediction").setMetricName("f1")
val Test_f1Score = evaluator.evaluate(testResults)

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");
```

**Выходные данные:**

Время, затраченное на выполнение кода, — 33 секунды.

### <a name="optimize-the-linear-regression-model-by-using-custom-cross-validation-and-parameter-sweeping-code"></a>Оптимизация модели линейной регрессии с помощью пользовательского кода перекрестной проверки и перебора параметров
Теперь оптимизируйте модель с помощью пользовательского кода и определите оптимальные параметры модели, используя критерий высокой точности. Затем создайте окончательную модель, оцените ее на основе тестовых данных и сохраните в хранилище BLOB-объектов. Наконец, загрузите модель, оцените тестовые данные и точность.

```scala
# RECORD THE START TIME
val starttime = Calendar.getInstance().getTime()

# DEFINE THE PARAMETER GRID AND THE NUMBER OF FOLDS
val paramGrid = new ParamGridBuilder().addGrid(rf.maxDepth, Array(5,10)).addGrid(rf.numTrees, Array(10,25,50)).build()

val nFolds = 3
val numModels = paramGrid.size
val numParamsinGrid = 2

# SPECIFY THE NUMBER OF CATEGORIES FOR CATEGORICAL VARIABLES
val categoricalFeaturesInfo = Map[Int, Int]((0,2),(1,2),(2,6),(3,4))

var maxDepth = -1
var numTrees = -1
var param = ""
var paramval = -1
var validateLB = -1.0
var validateUB = -1.0
val h = 1.0 / nFolds;
val RMSE  = Array.fill(numModels)(0.0)

# CREATE K-FOLDS
val splits = MLUtils.kFold(indexedTRAINbinary, numFolds = nFolds, seed=1234)


# LOOP THROUGH K-FOLDS AND THE PARAMETER GRID TO GET AND IDENTIFY THE BEST PARAMETER SET BY LEVEL OF ACCURACY
for (i <- 0 to (nFolds-1)) {
    validateLB = i * h
    validateUB = (i + 1) * h
    val validationCV = trainData.filter($"rand" >= validateLB  && $"rand" < validateUB)
    val trainCV = trainData.filter($"rand" < validateLB  || $"rand" >= validateUB)
    val validationLabPt = validationCV.rdd.map(r => LabeledPoint(r.getDouble(targetIndRegression(0).toInt), Vectors.dense(featuresIndIndex.map(r.getDouble(_)).toArray)));
    val trainCVLabPt = trainCV.rdd.map(r => LabeledPoint(r.getDouble(targetIndRegression(0).toInt), Vectors.dense(featuresIndIndex.map(r.getDouble(_)).toArray)));
    validationLabPt.cache()
    trainCVLabPt.cache()

    for (nParamSets <- 0 to (numModels-1)) {
        for (nParams <- 0 to (numParamsinGrid-1)) {
            param = paramGrid(nParamSets).toSeq(nParams).param.toString.split("__")(1)
            paramval = paramGrid(nParamSets).toSeq(nParams).value.toString.toInt
            if (param == "maxDepth") {maxDepth = paramval}
            if (param == "numTrees") {numTrees = paramval}
        }
        val rfModel = RandomForest.trainRegressor(trainCVLabPt, categoricalFeaturesInfo=categoricalFeaturesInfo,
                                                  numTrees=numTrees, maxDepth=maxDepth,
                                                  featureSubsetStrategy="auto",impurity="variance", maxBins=32)
        val labelAndPreds = validationLabPt.map { point =>
                                                 val prediction = rfModel.predict(point.features)
                                                  ( prediction, point.label )
                                                }
        val validMetrics = new RegressionMetrics(labelAndPreds)
        val rmse = validMetrics.rootMeanSquaredError
        RMSE(nParamSets) += rmse
    }
    validationLabPt.unpersist();
    trainCVLabPt.unpersist();
}
val minRMSEindex = RMSE.indexOf(RMSE.min)

# GET THE BEST PARAMETERS FROM A CROSS-VALIDATION AND PARAMETER SWEEP
var best_maxDepth = -1
var best_numTrees = -1
for (nParams <- 0 to (numParamsinGrid-1)) {
    param = paramGrid(minRMSEindex).toSeq(nParams).param.toString.split("__")(1)
    paramval = paramGrid(minRMSEindex).toSeq(nParams).value.toString.toInt
    if (param == "maxDepth") {best_maxDepth = paramval}
    if (param == "numTrees") {best_numTrees = paramval}
}

# CREATE THE BEST MODEL WITH THE BEST PARAMETERS AND A FULL TRAINING DATA SET
val best_rfModel = RandomForest.trainRegressor(indexedTRAINreg, categoricalFeaturesInfo=categoricalFeaturesInfo,
                                                  numTrees=best_numTrees, maxDepth=best_maxDepth,
                                                  featureSubsetStrategy="auto",impurity="variance", maxBins=32)

# SAVE THE BEST RANDOM FOREST MODEL IN BLOB STORAGE
val datestamp = Calendar.getInstance().getTime().toString.replaceAll(" ", ".").replaceAll(":", "_");
val modelName = "BestCV_RF_Regression__"
val filename = modelDir.concat(modelName).concat(datestamp)
best_rfModel.save(sc, filename);

# PREDICT ON THE TRAINING SET WITH THE BEST MODEL AND THEN EVALUATE
val labelAndPreds = indexedTESTreg.map { point =>
                                        val prediction = best_rfModel.predict(point.features)
                                        ( prediction, point.label )
                                       }

val test_rmse = new RegressionMetrics(labelAndPreds).rootMeanSquaredError
val test_rsqr = new RegressionMetrics(labelAndPreds).r2

# GET THE TIME TO RUN THE CELL
val endtime = Calendar.getInstance().getTime()
val elapsedtime =  ((endtime.getTime() - starttime.getTime())/1000).toString;
println("Time taken to run the above cell: " + elapsedtime + " seconds.");


# LOAD THE MODEL
val savedRFModel = RandomForestModel.load(sc, filename)

val labelAndPreds = indexedTESTreg.map { point =>
                                        val prediction = savedRFModel.predict(point.features)
                                        ( prediction, point.label )
                                       }
# TEST THE MODEL
val test_rmse = new RegressionMetrics(labelAndPreds).rootMeanSquaredError
val test_rsqr = new RegressionMetrics(labelAndPreds).r2
```

**Выходные данные:**

Время, затраченное на выполнение кода, — 61 секунда.

## <a name="consume-spark-built-machine-learning-models-automatically-with-scala"></a>Автоматическое использование моделей машинного обучения на основе Spark с кодом Scala
Обзор разделов с пошаговыми руководствами по задачам, которые входят в процесс обработки и анализа данных в Azure, см. в статье [Процесс обработки и анализа данных группы](./index.yml).

[Пошаговые руководства по процессу обработки и анализа данных](walkthroughs.md) представлены другие пошаговые руководства, которые демонстрируют этапы процесса обработки и анализа данных группы для конкретных сценариев. В пошаговых руководствах также показано, как объединить облачные и локальные инструменты и службы в единый рабочий процесс или конвейер, чтобы создать интеллектуальное приложение.

[Оценка моделей машинного обучения, созданных с помощью Spark](spark-model-consumption.md) показана процедура использования кода Scala для автоматической загрузки и оценки новых наборов данных с помощью встроенных моделей машинного обучения Spark с последующим сохранением в хранилище BLOB-объектов Azure. Чтобы включить автоматическое использование, просто замените код Python кодом Scala, следуя инструкциям в этой статье.