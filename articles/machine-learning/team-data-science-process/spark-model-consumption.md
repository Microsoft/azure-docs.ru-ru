---
title: Ввод моделей машинного обучения, созданных с помощью Spark, в эксплуатацию — командный процесс обработки и анализа данных | Документация Майкрософт
description: Сведения об отправке и оценке моделей обучения, сохраненных в хранилище BLOB-объектов Azure, с помощью Python.
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
ms.openlocfilehash: 659f61734001163ada6fd26caabe2fb007892f61
ms.sourcegitcommit: 2c1b93301174fccea00798df08e08872f53f669c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104774496"
---
# <a name="operationalize-spark-built-machine-learning-models"></a>Ввод моделей машинного обучения, созданных с помощью Spark, в эксплуатацию

В этой статье показано, как ввести сохраненные модели машинного обучения в эксплуатацию с помощью Python в кластерах HDInsight Spark. Кроме того, здесь описывается, как загрузить модели машинного обучения, созданные с помощью Spark MLlib и сохраненные в хранилище BLOB-объектов Azure, а также как оценить их с помощью наборов данных, которые сохранены в этом же хранилище. Здесь также описано, как выполнить предварительную подготовку входных данных, преобразовать признаки с помощью функций индексирования и кодирования в наборе средств MLlib и как создать объекты данных с помеченной вершиной, которые можно использовать в качестве входных данных для оценки при помощи моделей машинного обучения. Для оценки будут использоваться такие модели, как линейная регрессия, логистическая регрессия, случайные леса и градиентный бустинг деревьев.

## <a name="spark-clusters-and-jupyter-notebooks"></a>Кластеры Spark и записные книжки Jupyter
Действия по настройке и код, используемые для ввода модели машинного обучения в эксплуатацию в рамках этого руководства, можно выполнять как в кластере HDInsight Spark 1.6, так и в кластере Spark 2.0. В записных книжках Jupyter также содержится код для выполнения этих процедур.

### <a name="notebook-for-spark-16"></a>Записная книжка для Spark 1.6
Записная книжка Jupyter [pySpark-machine-learning-data-science-spark-model-consumption.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark1.6/pySpark-machine-learning-data-science-spark-model-consumption.ipynb) показывает, как применять сохраненную модель в кластерах HDInsight с помощью Python. 

### <a name="notebook-for-spark-20"></a>Записная книжка для Spark 2.0
Чтобы изменить записную книжку Jupyter для Spark 1.6 для использования в кластере HDInsight Spark 2.0, замените файл кода Python [этим файлом](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/Python/Spark2.0_ConsumeRFCV_NYCReg.py). Этот код показывает, как использовать модели, созданные в Spark 2.0.


## <a name="prerequisites"></a>Предварительные требования

1. Для работы с этим пошаговым руководством требуется учетная запись Azure и кластер Spark 1.6 или Spark 2.0 HDInsight. См. статью [Общие сведения об обработке и анализе данных с помощью платформы Spark в Azure HDInsight](spark-overview.md) для получения инструкций по выполнению этих требований. В этой статье также содержится описание используемых здесь данных о поездках в такси по Нью-Йорку за 2013 г., и инструкции по выполнению кода из записной книжки Jupyter в кластере Spark. 
2. Создайте модели машинного обучения, которые будут оцениваться здесь, выполнив анализ [данных и моделирование с помощью Spark](spark-data-exploration-modeling.md) для кластера Spark 1,6 или записных книжек Spark 2,0. 
3. Записные книжки Spark 2.0 используют для задачи классификации дополнительный набор данных — известное расписание вылетов авиакомпании за 2011 и 2012 гг. Описание записных книжек и ссылки на них вы можете найти в файле [Readme.md](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Readme.md) для репозитория GitHub с записными книжками. Более того, код здесь и в связанных записных книжках является универсальным и должен работать в любом кластере Spark. Действия по настройке кластера и управлению им могут немного отличаться от приведенных здесь, если вы не используете HDInsight Spark. 

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="setup-storage-locations-libraries-and-the-preset-spark-context"></a>Настройка места хранения, библиотек и предустановленного контекста Spark
Spark может использовать хранилище BLOB-объектов Azure (WASB) для считывания и записи данных. Таким образом, данные из хранилища можно обрабатывать с помощью Spark и сохранять полученные данные в этом же хранилище.

Чтобы сохранить модели или файлы в хранилище BLOB-объектов, необходимо указать путь соответствующим образом. К контейнеру по умолчанию, присоединенному к кластеру Spark, можно обращаться по пути, который начинается с: *"wasb///"*. В следующем примере кода задается расположение данных, которые необходимо считать, и путь для каталога хранилища модели, где сохраняются выходные данные модели: 

### <a name="set-directory-paths-for-storage-locations-in-wasb"></a>Настройка путей каталога к месту хранения в хранилище BLOB-объектов
Модели сохраняются в следующем расположении: wasb:///user/remoteuser/NYCTaxi/Models. Если этот путь задан неправильно, загрузить модели для оценки не удастся.

Результаты оценки сохраняются в следующем расположении: wasb:///user/remoteuser/NYCTaxi/ScoredResults. Если путь задан неправильно, сохранить результаты в эту папку не удастся.   

> [!NOTE]
> Путь к расположению файлов можно скопировать из выходных данных, полученных после выполнения последнего фрагмента кода записной книжки **machine-learning-data-science-spark-data-exploration-modeling.ipynb** , и вставить в заполнители в этом коде.   
> 
> 

Ниже приведен код для настройки путей к каталогам: 

```python
# LOCATION OF DATA TO BE SCORED (TEST DATA)
taxi_test_file_loc = "wasb://mllibwalkthroughs@cdspsparksamples.blob.core.windows.net/Data/NYCTaxi/JoinedTaxiTripFare.Point1Pct.Test.tsv";

# SET THE MODEL STORAGE DIRECTORY PATH 
# NOTE THE LAST BACKSLASH IN THIS PATH IS NEEDED
modelDir = "wasb:///user/remoteuser/NYCTaxi/Models/" 

# SET SCORDED RESULT DIRECTORY PATH
# NOTE THE LAST BACKSLASH IN THIS PATH IS NEEDED
scoredResultDir = "wasb:///user/remoteuser/NYCTaxi/ScoredResults/"; 

# FILE LOCATIONS FOR THE MODELS TO BE SCORED
logisticRegFileLoc = modelDir + "LogisticRegressionWithLBFGS_2016-04-1817_40_35.796789"
linearRegFileLoc = modelDir + "LinearRegressionWithSGD_2016-04-1817_44_00.993832"
randomForestClassificationFileLoc = modelDir + "RandomForestClassification_2016-04-1817_42_58.899412"
randomForestRegFileLoc = modelDir + "RandomForestRegression_2016-04-1817_44_27.204734"
BoostedTreeClassificationFileLoc = modelDir + "GradientBoostingTreeClassification_2016-04-1817_43_16.354770"
BoostedTreeRegressionFileLoc = modelDir + "GradientBoostingTreeRegression_2016-04-1817_44_46.206262"

# RECORD START TIME
import datetime
datetime.datetime.now()
```

**ВЫХОДНЫЕ ДАННЫЕ:**

datetime.datetime(2016, 4, 25, 23, 56, 19, 229403)

### <a name="import-libraries"></a>Импорт библиотек
Чтобы импортировать библиотеки и настроить контекст Spark, используйте следующий код:

```python
#IMPORT LIBRARIES
import pyspark
from pyspark import SparkConf
from pyspark import SparkContext
from pyspark.sql import SQLContext
import matplotlib
import matplotlib.pyplot as plt
from pyspark.sql import Row
from pyspark.sql.functions import UserDefinedFunction
from pyspark.sql.types import *
import atexit
from numpy import array
import numpy as np
import datetime
```

### <a name="preset-spark-context-and-pyspark-magics"></a>Предустановленный контекст Spark и волшебные команды PySpark
Ядра PySpark, входящие в состав записных книжек Jupyter, имеют предустановленный контекст. Поэтому не нужно явно задавать контексты Spark и Hive перед началом работы с разрабатываемым приложением. Эти контексты доступны по умолчанию:

* sc для Spark; 
* sqlContext для Hive.

Ядро PySpark предоставляет несколько "волшебных команд". Это специальные команды, которые можно вызывать с %%. В этих примерах кода используются две подобные команды.

* **%%local** Указывает, что код в последующих строках будет выполнен локально. В качестве кода должен быть указан корректный код Python.
* **%%sql -o \<variable name>** 
* Выполняет запрос Hive к sqlContext. Если передан параметр -o, результат запроса сохраняется в контексте %%local Python в качестве таблицы данных Pandas.

Дополнительные сведения о ядрах для записных книжек Jupyter и предустановленных "волшебных командах", которые они предоставляют, см. в статье [Ядра, доступные для использования записными книжками Jupyter с кластерами Apache Spark в HDInsight на платформе Linux](../../hdinsight/spark/apache-spark-jupyter-notebook-kernels.md).

## <a name="ingest-data-and-create-a-cleaned-data-frame"></a>Прием данных и создание очищенного фрейма данных
В этом разделе содержится код, предназначенный для выполнения задач приема данных, которые нужно смоделировать. С помощью этого кода можно считать файлы в формате TCV (выборку с примерно 0,1 % исходного набора данных о поездках в такси по Нью-Йорку и тарифами), форматировать данные, а затем создавать очищенные фреймы данных.

Файлы с данными о поездках в такси по Нью-Йорку и тарифах были объединены с помощью процедуры, описанной в статье [Процесс обработки и анализа данных группы на практике: использование кластеров HDInsight Hadoop](hive-walkthrough.md) .

```python
# INGEST DATA AND CREATE A CLEANED DATA FRAME

# RECORD START TIME
timestart = datetime.datetime.now()

# IMPORT FILE FROM PUBLIC BLOB
taxi_test_file = sc.textFile(taxi_test_file_loc)

# GET SCHEMA OF THE FILE FROM HEADER
taxi_header = taxi_test_file.filter(lambda l: "medallion" in l)

# PARSE FIELDS AND CONVERT DATA TYPE FOR SOME FIELDS
taxi_temp = taxi_test_file.subtract(taxi_header).map(lambda k: k.split("\t"))\
        .map(lambda p: (p[0],p[1],p[2],p[3],p[4],p[5],p[6],int(p[7]),int(p[8]),int(p[9]),int(p[10]),
                        float(p[11]),float(p[12]),p[13],p[14],p[15],p[16],p[17],p[18],float(p[19]),
                        float(p[20]),float(p[21]),float(p[22]),float(p[23]),float(p[24]),int(p[25]),int(p[26])))

# GET SCHEMA OF THE FILE FROM HEADER
schema_string = taxi_test_file.first()
fields = [StructField(field_name, StringType(), True) for field_name in schema_string.split('\t')]
fields[7].dataType = IntegerType() #Pickup hour
fields[8].dataType = IntegerType() # Pickup week
fields[9].dataType = IntegerType() # Weekday
fields[10].dataType = IntegerType() # Passenger count
fields[11].dataType = FloatType() # Trip time in secs
fields[12].dataType = FloatType() # Trip distance
fields[19].dataType = FloatType() # Fare amount
fields[20].dataType = FloatType() # Surcharge
fields[21].dataType = FloatType() # Mta_tax
fields[22].dataType = FloatType() # Tip amount
fields[23].dataType = FloatType() # Tolls amount
fields[24].dataType = FloatType() # Total amount
fields[25].dataType = IntegerType() # Tipped or not
fields[26].dataType = IntegerType() # Tip class
taxi_schema = StructType(fields)

# CREATE DATA FRAME
taxi_df_test = sqlContext.createDataFrame(taxi_temp, taxi_schema)

# CREATE A CLEANED DATA-FRAME BY DROPPING SOME UN-NECESSARY COLUMNS & FILTERING FOR UNDESIRED VALUES OR OUTLIERS
taxi_df_test_cleaned = taxi_df_test.drop('medallion').drop('hack_license').drop('store_and_fwd_flag').drop('pickup_datetime')\
    .drop('dropoff_datetime').drop('pickup_longitude').drop('pickup_latitude').drop('dropoff_latitude')\
    .drop('dropoff_longitude').drop('tip_class').drop('total_amount').drop('tolls_amount').drop('mta_tax')\
    .drop('direct_distance').drop('surcharge')\
    .filter("passenger_count > 0 and passenger_count < 8 AND payment_type in ('CSH', 'CRD') AND tip_amount >= 0 AND tip_amount < 30 AND fare_amount >= 1 AND fare_amount < 150 AND trip_distance > 0 AND trip_distance < 100 AND trip_time_in_secs > 30 AND trip_time_in_secs < 7200" )

# CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
taxi_df_test_cleaned.cache()
taxi_df_test_cleaned.count()

# REGISTER DATA-FRAME AS A TEMP-TABLE IN SQL-CONTEXT
taxi_df_test_cleaned.registerTempTable("taxi_test")

# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**ВЫХОДНЫЕ ДАННЫЕ:**

Время на выполнение кода выше: 46,37 с.

## <a name="prepare-data-for-scoring-in-spark"></a>Подготовка данных для оценки в Spark
В этом разделе описано, как индексировать, кодировать и масштабировать категориальные признаки, чтобы подготовить их для использования в контролируемых алгоритмах обучения MLlib для моделей классификации и регрессии.

### <a name="feature-transformation-index-and-encode-categorical-features-for-input-into-models-for-scoring"></a>Преобразование признаков. Индексирование и кодирование категориальных признаков для получения входных данных для оцениваемых моделей
В этом разделе описано, как проиндексировать категориальные данные с помощью `StringIndexer` и закодировать признаки с помощью `OneHotEncoder`, чтобы получить входные данные для моделей.

С помощью функции [StringIndexer](https://spark.apache.org/docs/latest/ml-features.html#stringindexer) столбец меток строкового типа кодируется как столбец с индексами меток. При этом индексы упорядочиваются по частоте меток. 

[OneHotEncoder](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OneHotEncoder.html#sklearn.preprocessing.OneHotEncoder) сопоставляет столбец с индексами меток со столбцом двоичных векторов как минимум с одним отдельным значением. Благодаря этой кодировке алгоритмы, для которых требуются признаки со значениями (например, логистическая регрессия), можно применять к категориальным признакам.

```python
#INDEX AND ONE-HOT ENCODE CATEGORICAL FEATURES

# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD PYSPARK LIBRARIES
from pyspark.ml.feature import OneHotEncoder, StringIndexer, VectorAssembler, VectorIndexer

# CREATE FOUR BUCKETS FOR TRAFFIC TIMES
sqlStatement = """
    SELECT *,
    CASE
     WHEN (pickup_hour <= 6 OR pickup_hour >= 20) THEN "Night" 
     WHEN (pickup_hour >= 7 AND pickup_hour <= 10) THEN "AMRush" 
     WHEN (pickup_hour >= 11 AND pickup_hour <= 15) THEN "Afternoon"
     WHEN (pickup_hour >= 16 AND pickup_hour <= 19) THEN "PMRush"
    END as TrafficTimeBins
    FROM taxi_test 
"""
taxi_df_test_with_newFeatures = sqlContext.sql(sqlStatement)

# CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
taxi_df_test_with_newFeatures.cache()
taxi_df_test_with_newFeatures.count()

# INDEX AND ONE-HOT ENCODING
stringIndexer = StringIndexer(inputCol="vendor_id", outputCol="vendorIndex")
model = stringIndexer.fit(taxi_df_test_with_newFeatures) # Input data-frame is the cleaned one from above
indexed = model.transform(taxi_df_test_with_newFeatures)
encoder = OneHotEncoder(dropLast=False, inputCol="vendorIndex", outputCol="vendorVec")
encoded1 = encoder.transform(indexed)

# INDEX AND ENCODE RATE_CODE
stringIndexer = StringIndexer(inputCol="rate_code", outputCol="rateIndex")
model = stringIndexer.fit(encoded1)
indexed = model.transform(encoded1)
encoder = OneHotEncoder(dropLast=False, inputCol="rateIndex", outputCol="rateVec")
encoded2 = encoder.transform(indexed)

# INDEX AND ENCODE PAYMENT_TYPE
stringIndexer = StringIndexer(inputCol="payment_type", outputCol="paymentIndex")
model = stringIndexer.fit(encoded2)
indexed = model.transform(encoded2)
encoder = OneHotEncoder(dropLast=False, inputCol="paymentIndex", outputCol="paymentVec")
encoded3 = encoder.transform(indexed)

# INDEX AND ENCODE TRAFFIC TIME BINS
stringIndexer = StringIndexer(inputCol="TrafficTimeBins", outputCol="TrafficTimeBinsIndex")
model = stringIndexer.fit(encoded3)
indexed = model.transform(encoded3)
encoder = OneHotEncoder(dropLast=False, inputCol="TrafficTimeBinsIndex", outputCol="TrafficTimeBinsVec")
encodedFinal = encoder.transform(indexed)

# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**ВЫХОДНЫЕ ДАННЫЕ:**

Время на выполнение кода выше: 5,37 с.

### <a name="create-rdd-objects-with-feature-arrays-for-input-into-models"></a>Создание объектов устойчивого распределенного набора данных с массивами признаков для получения входных данных для моделей
С помощью кода в этом разделе можно индексировать категориальные текстовые данные в объекты устойчивого распределенного набора данных, а затем выполнить их прямое кодирование. После этого данные можно использовать для обучения и проверки модели логистической регрессии MLlib и моделей с применением дерева. Проиндексированные данные хранятся в объектах [устойчивого распределенного набора данных](https://spark.apache.org/docs/latest/api/java/org/apache/spark/rdd/RDD.html). RDD являются базовой абстракцией в Spark. Объект устойчивого распределенного набора данных — это неизменяемая, секционированная коллекция элементов, которыми можно одновременно управлять с помощью Spark.

Здесь также содержится код, в котором показано, как масштабировать данные с помощью функции MLlib `StandardScalar` для использования в моделях линейной регрессии с применением метода стохастического градиента. Этот алгоритм широко используется для обучения других моделей машинного обучения. [StandardScaler](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.mllib.tree.RandomForest.html#pyspark.mllib.tree.RandomForest
) используется для масштабирования признаков до единичной дисперсии. Масштабирование признаков (или нормализация данных) гарантирует, что для признаков с широко распределенными значениями не задано высокое значение веса в целевой функции. 

```python
# CREATE RDD OBJECTS WITH FEATURE ARRAYS FOR INPUT INTO MODELS

# RECORD START TIME
timestart = datetime.datetime.now()

# IMPORT LIBRARIES
from pyspark.mllib.linalg import Vectors
from pyspark.mllib.feature import StandardScaler, StandardScalerModel
from pyspark.mllib.util import MLUtils
from numpy import array

# INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
def parseRowIndexingBinary(line):
    features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.TrafficTimeBinsIndex,
                         line.pickup_hour, line.weekday, line.passenger_count, line.trip_time_in_secs, 
                         line.trip_distance, line.fare_amount])
    return  features

# ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO LOGISTIC REGRESSION MODELS
def parseRowOneHotBinary(line):
    features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
                                        line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
                                        line.vendorVec.toArray(), line.rateVec.toArray(), 
                                        line.paymentVec.toArray(), line.TrafficTimeBinsVec.toArray()), axis=0)
    return  features

# ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
def parseRowIndexingRegression(line):
    features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.TrafficTimeBinsIndex, 
                         line.pickup_hour, line.weekday, line.passenger_count, line.trip_time_in_secs, 
                         line.trip_distance, line.fare_amount])
    return  features

# INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO LINEAR REGRESSION MODELS
def parseRowOneHotRegression(line):
    features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
                                        line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
                                        line.vendorVec.toArray(), line.rateVec.toArray(), 
                                        line.paymentVec.toArray(), line.TrafficTimeBinsVec.toArray()), axis=0)
    return  features

# FOR BINARY CLASSIFICATION TRAINING AND TESTING
indexedTESTbinary = encodedFinal.map(parseRowIndexingBinary)
oneHotTESTbinary = encodedFinal.map(parseRowOneHotBinary)

# FOR REGRESSION CLASSIFICATION TRAINING AND TESTING
indexedTESTreg = encodedFinal.map(parseRowIndexingRegression)
oneHotTESTreg = encodedFinal.map(parseRowOneHotRegression)

# SCALING FEATURES FOR LINEARREGRESSIONWITHSGD MODEL
scaler = StandardScaler(withMean=False, withStd=True).fit(oneHotTESTreg)
oneHotTESTregScaled = scaler.transform(oneHotTESTreg)

# CACHE RDDS IN MEMORY
indexedTESTbinary.cache();
oneHotTESTbinary.cache();
indexedTESTreg.cache();
oneHotTESTreg.cache();
oneHotTESTregScaled.cache();

# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**ВЫХОДНЫЕ ДАННЫЕ:**

Время на выполнение кода выше: 11,72 с.

## <a name="score-with-the-logistic-regression-model-and-save-output-to-blob"></a>Оценка с использованием модели логистической регрессии и сохранение выходных данных в хранилище BLOB-объектов
С помощью кода, описанного в этом разделе, можно выполнить следующие задачи: загрузить модель логистической регрессии, сохраненную в хранилище BLOB-объектов Azure, использовать ее, чтобы спрогнозировать, будут ли оставлены чаевые за поездку в такси, оценить ее с помощью стандартных показателей классификации, а затем сохранить полученные результаты в это хранилище и отобразить их. Результаты оценки будут сохранены в объектах устойчивого распределенного набора данных. 

```python
# SCORE AND EVALUATE LOGISTIC REGRESSION MODEL

# RECORD START TIME
timestart = datetime.datetime.now()

# IMPORT LIBRARIES
from pyspark.mllib.classification import LogisticRegressionModel

## LOAD SAVED MODEL
savedModel = LogisticRegressionModel.load(sc, logisticRegFileLoc)
predictions = oneHotTESTbinary.map(lambda features: (float(savedModel.predict(features))))

## SAVE SCORED RESULTS (RDD) TO BLOB
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
logisticregressionfilename = "LogisticRegressionWithLBFGS_" + datestamp + ".txt";
dirfilename = scoredResultDir + logisticregressionfilename;
predictions.saveAsTextFile(dirfilename)


# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds";
```

**ВЫХОДНЫЕ ДАННЫЕ:**

Время на выполнение кода выше: 19,22 с.

## <a name="score-a-linear-regression-model"></a>Оценка модели линейной регрессии
Чтобы спрогнозировать сумму чаевых, мы с помощью функции [LinearRegressionWithSGD](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.regression.LinearRegressionWithSGD) обучили модель линейной регрессии, в которой для оптимизации применяется метод стохастического градиента. 

С помощью кода, описанного в этом разделе, можно загрузить модель линейной регрессии из хранилища BLOB-объектов Azure, оценить ее с помощью масштабированных переменных, а затем сохранить результаты в большом двоичном объекте.

```python
#SCORE LINEAR REGRESSION MODEL

# RECORD START TIME
timestart = datetime.datetime.now()

#LOAD LIBRARIES
from pyspark.mllib.regression import LinearRegressionWithSGD, LinearRegressionModel

# LOAD MODEL AND SCORE USING ** SCALED VARIABLES **
savedModel = LinearRegressionModel.load(sc, linearRegFileLoc)
predictions = oneHotTESTregScaled.map(lambda features: (float(savedModel.predict(features))))

# SAVE RESULTS
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
linearregressionfilename = "LinearRegressionWithSGD_" + datestamp;
dirfilename = scoredResultDir + linearregressionfilename;
predictions.saveAsTextFile(dirfilename)

# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**ВЫХОДНЫЕ ДАННЫЕ:**

Время на выполнение кода выше: 16,63 с.

## <a name="score-classification-and-regression-random-forest-models"></a>Оценка моделей классификации и регрессии с применением алгоритма случайного леса
С помощью кода, описанного в этом разделе, можно загрузить сохраненные в хранилище BLOB-объектов Azure модели классификации и регрессии с применением алгоритма случайного леса, оценить их эффективность с помощью стандартных показателей классификации и регрессии, а затем сохранить результаты в том же хранилище.

[Случайные леса](https://spark.apache.org/docs/latest/mllib-ensembles.html#Random-Forests) — это совокупности деревьев принятия решений.  Такие совокупности минимизируют необходимость в переобучении. С помощью этого алгоритма можно обрабатывать категориальные признаки, расширять модель для выполнения задач мультиклассовой классификации, а также определять нелинейные зависимости и взаимодействия признаков. Этот метод не требуют масштабирования признаков. Случайные леса — одна из самых эффективных моделей машинного обучения для задач классификации и регрессии.

[spark.mllib](https://spark.apache.org/mllib/) предусматривает использование метода случайного леса в моделях двоичной, мультиклассовой классификации и регрессии с применением как непрерывных, так и категориальных признаков. 

```python
# SCORE RANDOM FOREST MODELS FOR CLASSIFICATION AND REGRESSION

# RECORD START TIME
timestart = datetime.datetime.now()

#IMPORT MLLIB LIBRARIES    
from pyspark.mllib.tree import RandomForest, RandomForestModel


# CLASSIFICATION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB
savedModel = RandomForestModel.load(sc, randomForestClassificationFileLoc)
predictions = savedModel.predict(indexedTESTbinary)

# SAVE RESULTS
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
rfclassificationfilename = "RandomForestClassification_" + datestamp + ".txt";
dirfilename = scoredResultDir + rfclassificationfilename;
predictions.saveAsTextFile(dirfilename)


# REGRESSION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB
savedModel = RandomForestModel.load(sc, randomForestRegFileLoc)
predictions = savedModel.predict(indexedTESTreg)

# SAVE RESULTS
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
rfregressionfilename = "RandomForestRegression_" + datestamp + ".txt";
dirfilename = scoredResultDir + rfregressionfilename;
predictions.saveAsTextFile(dirfilename)

# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds";
```

**ВЫХОДНЫЕ ДАННЫЕ:**

Время на выполнение кода выше: 31,07 с.

## <a name="score-classification-and-regression-gradient-boosting-tree-models"></a>Оценка моделей классификации и регрессии с применением метода градиентного бустинга деревьев
С помощью кода, описанного в этом разделе, можно загрузить сохраненные в хранилище BLOB-объектов Azure модели классификации и регрессии с применением метода градиентного бустинга деревьев, оценить их эффективность с помощью стандартных показателей классификации и регрессии, а затем сохранить результаты в том же хранилище. 

**Spark. mllib** поддерживает GBT для двоичной классификации и для регрессии, используя как непрерывные, так и функции категорий. 

[Деревья ускоренного градиента](https://spark.apache.org/docs/latest/ml-classification-regression.html#gradient-boosted-trees-gbts) (GBT) — это это совокупности деревьев принятия решений. GBTS предусматривает итеративное обучение деревьев принятия решений, что позволяет свести потери к минимуму. GBT может работать с характеристиками категории, не требуя масштабирования признаков и возможность захвата нелинейности и взаимодействия функций. Этот алгоритм также можно использовать в параметре многоклассовой классификации.

```python
# SCORE GRADIENT BOOSTING TREE MODELS FOR CLASSIFICATION AND REGRESSION

# RECORD START TIME
timestart = datetime.datetime.now()

#IMPORT MLLIB LIBRARIES
from pyspark.mllib.tree import GradientBoostedTrees, GradientBoostedTreesModel

# CLASSIFICATION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB

#LOAD AND SCORE THE MODEL
savedModel = GradientBoostedTreesModel.load(sc, BoostedTreeClassificationFileLoc)
predictions = savedModel.predict(indexedTESTbinary)

# SAVE RESULTS
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
btclassificationfilename = "GradientBoostingTreeClassification_" + datestamp + ".txt";
dirfilename = scoredResultDir + btclassificationfilename;
predictions.saveAsTextFile(dirfilename)


# REGRESSION: LOAD SAVED MODEL, SCORE AND SAVE RESULTS BACK TO BLOB

# LOAD AND SCORE MODEL 
savedModel = GradientBoostedTreesModel.load(sc, BoostedTreeRegressionFileLoc)
predictions = savedModel.predict(indexedTESTreg)

# SAVE RESULTS
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
btregressionfilename = "GradientBoostingTreeRegression_" + datestamp + ".txt";
dirfilename = scoredResultDir + btregressionfilename;
predictions.saveAsTextFile(dirfilename)


# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**ВЫХОДНЫЕ ДАННЫЕ:**

Время на выполнение кода выше: 14,6 с.

## <a name="clean-up-objects-from-memory-and-print-scored-file-locations"></a>Очистка объектов из памяти и вывод расположений оцененных файлов

```python
# UNPERSIST OBJECTS CACHED IN MEMORY
taxi_df_test_cleaned.unpersist()
indexedTESTbinary.unpersist();
oneHotTESTbinary.unpersist();
indexedTESTreg.unpersist();
oneHotTESTreg.unpersist();
oneHotTESTregScaled.unpersist();


# PRINT OUT PATH TO SCORED OUTPUT FILES
print "logisticRegFileLoc: " + logisticregressionfilename;
print "linearRegFileLoc: " + linearregressionfilename;
print "randomForestClassificationFileLoc: " + rfclassificationfilename;
print "randomForestRegFileLoc: " + rfregressionfilename;
print "BoostedTreeClassificationFileLoc: " + btclassificationfilename;
print "BoostedTreeRegressionFileLoc: " + btregressionfilename;
```

**ВЫХОДНЫЕ ДАННЫЕ:**

logisticRegFileLoc: LogisticRegressionWithLBFGS_2016-05-0317_22_38.953814.txt

linearRegFileLoc: LinearRegressionWithSGD_2016-05-0317_22_58.878949

randomForestClassificationFileLoc: RandomForestClassification_2016-05-0317_23_15.939247.txt

randomForestRegFileLoc: RandomForestRegression_2016-05-0317_23_31.459140.txt

BoostedTreeClassificationFileLoc: GradientBoostingTreeClassification_2016-05-0317_23_49.648334.txt

BoostedTreeRegressionFileLoc: GradientBoostingTreeRegression_2016-05-0317_23_56.860740.txt

## <a name="consume-spark-models-through-a-web-interface"></a>Использование моделей Spark через веб-интерфейс
В кластере Spark реализован механизм, который позволяет отправлять пакетные задания или интерактивные запросы удаленно через интерфейс REST с помощью компонента Livy. Livy включен по умолчанию в кластере Spark в HDInsight. Дополнительные сведения о Livy см. в статье [Удаленная отправка заданий Spark с помощью Livy](../../hdinsight/spark/apache-spark-livy-rest-interface.md). 

Вы можете использовать Livy, чтобы удаленно отправить запрос на выполнение пакетного задания оценки файла, сохраненного в большом двоичном объекте Azure, а затем записать результаты в другой большой двоичный объект. Чтобы сделать это, необходимо загрузить сценарий Python из   
репозитория [GitHub](https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/Spark/Python/ConsumeGBNYCReg.py) в большой двоичный объект кластера Spark. Используйте **Microsoft Azure Storage Explorer** или **AzCopy**, чтобы скопировать сценарий в большой двоичный объект кластера. В нашем случае мы отправили сценарий в **_wasb:///example/Python/ConsumeGBNYCReg.py_**.   

> [!NOTE]
> Необходимые ключи доступа можно найти на портале для учетной записи хранения, связанной с кластером Spark. 
> 
> 

После загрузки скрипт будет выполнен в кластере Spark в распределенном контексте. Он загрузит модель и запустит прогнозирование на основе входных файлов в зависимости от модели.  

Этот скрипт можно вызвать удаленно, выполнив простой запрос HTTPS или REST в Livy.  Ниже приведен пример команды Curl, с помощью которой можно создать запрос HTTP для удаленного вызова скрипта Python. Параметры CLUSTERLOGIN, CLUSTERPASSWORD, CLUSTERNAME следует заменить соответствующими значениями для используемого кластера Spark.

```console
# CURL COMMAND TO INVOKE PYTHON SCRIPT WITH HTTP REQUEST

curl -k --user "CLUSTERLOGIN:CLUSTERPASSWORD" -X POST --data "{\"file\": \"wasb:///example/python/ConsumeGBNYCReg.py\"}" -H "Content-Type: application/json" https://CLUSTERNAME.azurehdinsight.net/livy/batches
```

Чтобы вызвать задание Spark через Livy, необходимо выполнить простой вызов HTTP с обычной проверкой подлинности. В удаленной системе для этого можно использовать любой язык.   

> [!NOTE]
> При выполнении вызова HTTP удобнее использовать библиотеку запросов для Python, но она не установлена по умолчанию в функциях Azure. Поэтому вместо нее используются старые библиотеки HTTP.   
> 
> 

Ниже приведен пример кода Python для вызова HTTP.

```python
#MAKE AN HTTPS CALL ON LIVY. 

import os

# OLDER HTTP LIBRARIES USED HERE INSTEAD OF THE REQUEST LIBRARY AS THEY ARE AVAILABLE BY DEFAULT
import httplib, urllib, base64

# REPLACE VALUE WITH ONES FOR YOUR SPARK CLUSTER
host = '<spark cluster name>.azurehdinsight.net:443'
username='<username>'
password='<password>'

#AUTHORIZATION
conn = httplib.HTTPSConnection(host)
auth = base64.encodestring('%s:%s' % (username, password)).replace('\n', '')
headers = {'Content-Type': 'application/json', 'Authorization': 'Basic %s' % auth}

# SPECIFY THE PYTHON SCRIPT TO RUN ON THE SPARK CLUSTER
# IN THE FILE PARAMETER OF THE JSON POST REQUEST BODY
r=conn.request("POST", '/livy/batches', '{"file": "wasb:///example/python/ConsumeGBNYCReg.py"}', headers )
response = conn.getresponse().read()
print(response)
conn.close()
```

Вы также можете добавить этот код Python в [функции Azure](https://azure.microsoft.com/documentation/services/functions/) , чтобы активировать отправку задания Spark, которая оценивает большой двоичный объект на основе различных событий, таких как таймер, создание или обновление большого двоичного объекта. 

Если вы не хотите работать с кодом, используйте [приложения логики Azure](https://azure.microsoft.com/documentation/services/app-service/logic/), чтобы вызвать пакетную оценку Spark. Для этого определите действие HTTP в **конструкторе приложений логики** и настройте соответствующие параметры. 

* В портал Azure создайте новое приложение логики, выбрав **+ создать**  ->  **Интернет и мобильные устройства**  ->  **приложение логики**. 
* Введите имя приложения логики и план служб приложений, чтобы открыть **конструктор приложений логики**.
* Выберите действие HTTP и введите параметры, показанные на рисунке ниже.

![Конструктор Logic Apps](./media/spark-model-consumption/spark-logica-app-client.png)

## <a name="whats-next"></a>Дальнейшие действия
**Перекрестная проверка и очистка параметров**. Дополнительные сведения о том, как можно обучить модели с помощью перекрестной проверки и вычистки параметров Hyper-Parameter, см. в разделе [расширенное исследование данных и моделирование в Spark](spark-advanced-data-exploration-modeling.md) .

