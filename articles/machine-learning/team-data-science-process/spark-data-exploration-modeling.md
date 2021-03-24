---
title: Исследование и моделирование данных с помощью Spark — процесс обработки и анализа данных группы
description: В этой статье показаны возможности исследования и моделирования данных с помощью набора средств Spark MLlib в HDInsight Spark.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: sample
ms.date: 06/03/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath, contperf-fy20q4
ms.openlocfilehash: 531e5cf274d40eae19075514d50aec5a6524aeec
ms.sourcegitcommit: 2c1b93301174fccea00798df08e08872f53f669c
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104775346"
---
# <a name="data-exploration-and-modeling-with-spark"></a>Исследование и моделирование данных с помощью Spark

Узнайте, как использовать HDInsight Spark для обучения моделей машинного обучения для прогнозирования тарифов такси с помощью Spark MLlib.

В этом примере показаны различные шаги в [процессе обработки и анализа данных группы](./index.yml). Для загрузки, просмотра и подготовки данных используется набор данных о тарифах и поездках в такси в Нью-Йорке за 2013 год. Затем с помощью Spark MLlib модели двоичной классификации и регрессии обучаются, чтобы предсказать, оставит ли пассажир чаевые, а также приблизительную сумму чаевых.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим пошаговым руководством требуется учетная запись Azure и кластер Spark 1.6 или Spark 2.0 HDInsight. См. статью [Общие сведения об обработке и анализе данных с помощью платформы Spark в Azure HDInsight](spark-overview.md) для получения инструкций по выполнению этих требований. В этой статье также содержится описание используемых здесь данных о поездках в такси по Нью-Йорку за 2013 г., и инструкции по выполнению кода из записной книжки Jupyter в кластере Spark. 

### <a name="spark-clusters-and-notebooks"></a>Кластеры и записные книжки Spark

Действия по настройке и код, указанные в этом пошаговом руководстве, применимы к использованию для HDInsight Spark 1.6. Но записные книжки Jupyter можно использовать для кластеров HDInsight Spark 1.6 и Spark 2.0. Описание записных книжек и ссылки на них вы можете найти в файле [Readme.md](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Readme.md) для репозитория GitHub с записными книжками. Более того, код здесь и в связанных записных книжках является универсальным и должен работать в любом кластере Spark. Действия по настройке кластера и управлению им могут немного отличаться от приведенных здесь, если вы не используете HDInsight Spark. Для удобства мы приводим ссылки на записные книжки Jupyter для Spark 1.6 (выполняемые в ядре PySpark на сервере записной книжки Jupyter) и Spark 2.0 (выполняемые в ядре PySpark3 на сервере записной книжки Jupyter).

- [Записные книжки для Spark 1.6](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark1.6/). Содержит сведения о том, как выполнять просмотр данных, моделирование и оценку с помощью нескольких алгоритмов.
- [Записные книжки для Spark 2.0](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark2.0/). Содержит сведения о выполнении задач регрессии и классификации. Наборы данных могут различаться, но эти шаги и понятия применимы к различным наборам данных.

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

> [!NOTE]
> Описания, приведенные ниже, относятся к Spark 1.6. Для Spark 2.0 используйте записные книжки, описанные выше. 

## <a name="setup"></a>Настройка

Spark может считывать данные и записывать их в хранилище BLOB-объектов Azure (также известное как WASB). Таким образом, данные из хранилища можно обрабатывать с помощью Spark и сохранять полученные данные в этом же хранилище.

Чтобы сохранить модели или файлы в хранилище BLOB-объектов, необходимо указать путь соответствующим образом. Контейнер по умолчанию, присоединенный к кластеру Spark, можно указать с помощью пути, который начинается с "wasb:///". Другие расположения начинаются с wasb://.

### <a name="set-directory-paths-for-storage-locations-in-wasb"></a>Настройка путей каталога к месту хранения в хранилище BLOB-объектов

В следующем примере кода указывается расположение данных, которые необходимо считать, и путь для каталога хранилища модели, где сохраняются выходные данные модели:

```python
# SET PATHS TO FILE LOCATIONS: DATA AND MODEL STORAGE

# LOCATION OF TRAINING DATA
taxi_train_file_loc = "wasb://mllibwalkthroughs@cdspsparksamples.blob.core.windows.net/Data/NYCTaxi/JoinedTaxiTripFare.Point1Pct.Train.tsv";

# SET THE MODEL STORAGE DIRECTORY PATH 
# NOTE THAT THE FINAL BACKSLASH IN THE PATH IS NEEDED.
modelDir = "wasb:///user/remoteuser/NYCTaxi/Models/" 
```

### <a name="import-libraries"></a>Импорт библиотек

Для установки также требуется импорт необходимых библиотек. Чтобы импортировать библиотеки и настроить контекст Spark, используйте следующий код:

```python
# IMPORT LIBRARIES
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

Ядра PySpark, входящие в состав записных книжек Jupyter, имеют предустановленный контекст. Поэтому перед началом работы с разрабатываемым приложением вам не нужно явно задавать контексты Spark или Hive. Эти контексты доступны вам по умолчанию, а именно:

* sc для Spark; 
* sqlContext для Hive.

Ядро PySpark предоставляет несколько "волшебных команд". Это специальные команды, которые можно вызывать с %%. В этих примерах кода используются две подобные команды.

* **%%local** Указывает, что код в последующих строках будет выполнен локально. В качестве кода должен быть указан корректный код Python.
* **%%sql -o \<variable name>** Выполняет запрос Hive к sqlContext. Если передан параметр -o, результат запроса сохраняется в контексте %%local Python в формате Pandas DataFrame.

Дополнительные сведения о ядрах записных книжек Jupyter и предустановленных "волшебных командах" см. в статье [Ядра, доступные для использования записными книжками Jupyter с кластерами Apache Spark в HDInsight на платформе Linux](../../hdinsight/spark/apache-spark-jupyter-notebook-kernels.md).

## <a name="load-the-data"></a>Загрузка данных

Чтобы начать анализ и обработку необходимых данных, требуется переместить их из источников, в которых они находятся, в среду исследования и моделирования данных. В нашем пошаговом руководстве это среда Spark. Этот раздел содержит код, с помощью которого можно выполнить следующие задачи:

* прием выборки данных, которые нужно моделировать;
* считывание входного набора данных (TSV-файл);
* форматирование и очистка данных;
* создание и кэширование объектов (устойчивых распределенных наборов данных или фреймов данных) в памяти;
* регистрация данных в качестве временной таблицы в контексте SQL.

Ниже приведен код для приема данных.

```python
# INGEST DATA

# RECORD START TIME
timestart = datetime.datetime.now()

# IMPORT FILE FROM PUBLIC BLOB
taxi_train_file = sc.textFile(taxi_train_file_loc)

# GET SCHEMA OF THE FILE FROM HEADER
schema_string = taxi_train_file.first()
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

# PARSE FIELDS AND CONVERT DATA TYPE FOR SOME FIELDS
taxi_header = taxi_train_file.filter(lambda l: "medallion" in l)
taxi_temp = taxi_train_file.subtract(taxi_header).map(lambda k: k.split("\t"))\
        .map(lambda p: (p[0],p[1],p[2],p[3],p[4],p[5],p[6],int(p[7]),int(p[8]),int(p[9]),int(p[10]),
                        float(p[11]),float(p[12]),p[13],p[14],p[15],p[16],p[17],p[18],float(p[19]),
                        float(p[20]),float(p[21]),float(p[22]),float(p[23]),float(p[24]),int(p[25]),int(p[26])))


# CREATE DATA FRAME
taxi_train_df = sqlContext.createDataFrame(taxi_temp, taxi_schema)

# CREATE A CLEANED DATA-FRAME BY DROPPING SOME UN-NECESSARY COLUMNS & FILTERING FOR UNDESIRED VALUES OR OUTLIERS
taxi_df_train_cleaned = taxi_train_df.drop('medallion').drop('hack_license').drop('store_and_fwd_flag').drop('pickup_datetime')\
    .drop('dropoff_datetime').drop('pickup_longitude').drop('pickup_latitude').drop('dropoff_latitude')\
    .drop('dropoff_longitude').drop('tip_class').drop('total_amount').drop('tolls_amount').drop('mta_tax')\
    .drop('direct_distance').drop('surcharge')\
    .filter("passenger_count > 0 and passenger_count < 8 AND payment_type in ('CSH', 'CRD') AND tip_amount >= 0 AND tip_amount < 30 AND fare_amount >= 1 AND fare_amount < 150 AND trip_distance > 0 AND trip_distance < 100 AND trip_time_in_secs > 30 AND trip_time_in_secs < 7200" )


# CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
taxi_df_train_cleaned.cache()
taxi_df_train_cleaned.count()

# REGISTER DATA-FRAME AS A TEMP-TABLE IN SQL-CONTEXT
taxi_df_train_cleaned.registerTempTable("taxi_train")

# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds";
```

**ВЫХОДНЫЕ ДАННЫЕ:**

Время выполнения кода выше: 51,72 секунды

## <a name="explore-the-data"></a>Просмотр данных

После помещения данных в Spark необходимо исследовать и визуализировать их, чтобы получить более глубокое представление. В этом разделе мы исследуем данные о поездках в такси с помощью SQL-запросов и составим графики целевых переменных и потенциальных признаков для визуального контроля. В частности, мы составим график частоты поездок в такси, частоты оставления чаевых и зависимости суммы чаевых от тарифа и типа платежа.

### <a name="plot-a-histogram-of-passenger-count-frequencies-in-the-sample-of-taxi-trips"></a>Построение гистограммы количества пассажиров в выборке данных о поездках на такси

Этот код и последующие фрагменты используют волшебную команду SQL для запроса примера и локальную волшебную команду для графического представления данных.

* **SQL magic (`%%sql`)** Ядро HDInsight PySpark позволяет легко выполнять встроенные запросы HiveQL к sqlContext. Аргумент (-o VARIABLE_NAME) сохраняет выходные данные запроса SQL в формате Pandas DataFrame на сервере Jupyter. Этот параметр предоставляет выходные данные в локальном режиме.
* Магическая команда **`%%local`** используется для локального выполнения кода на сервере Jupyter, который представляет собой головной узел кластера HDInsight. Как правило, магическая команда `%%local` используется в комбинации с командой `%%sql` с параметром -o. Параметр -o сохраняет выходные данные запроса SQL локально, после чего волшебная команда %%local активирует следующий набор фрагментов кода, который выполняется локально с выходными данными запросов SQL, сохраненными на локальном компьютере.

После выполнения кода выходные данные визуализируются автоматически.

Этот запрос извлекает поездки по количеству пассажиров. 

```sql
# PLOT FREQUENCY OF PASSENGER COUNTS IN TAXI TRIPS

# HIVEQL QUERY AGAINST THE sqlContext
%%sql -q -o sqlResults
SELECT passenger_count, COUNT(*) as trip_counts 
FROM taxi_train 
WHERE passenger_count > 0 and passenger_count < 7 
GROUP BY passenger_count 
```

Этот код создает локальный фрейм данных из выходных данных запроса и формирует графическое представление данных. Магическая команда `%%local` создает локальный кадр данных `sqlResults`, который можно использовать для формирования графического представления данных с помощью matplotlib. 

> [!NOTE]
> В этом пошаговом руководстве волшебная команда PySpark используется несколько раз. Если объем данных большой, сделайте выборку, чтобы создать фрейм данных, который можно разместить в локальной памяти.

```python
#CREATE LOCAL DATA-FRAME AND USE FOR MATPLOTLIB PLOTTING

# RUN THE CODE LOCALLY ON THE JUPYTER SERVER
%%local

# USE THE JUPYTER AUTO-PLOTTING FEATURE TO CREATE INTERACTIVE FIGURES. 
# CLICK ON THE TYPE OF PLOT TO BE GENERATED (E.G. LINE, AREA, BAR ETC.)
sqlResults
```

Код для получения графического представления поездок по числу пассажиров

```python
# PLOT PASSENGER NUMBER VS. TRIP COUNTS
%%local
import matplotlib.pyplot as plt
%matplotlib inline

x_labels = sqlResults['passenger_count'].values
fig = sqlResults[['trip_counts']].plot(kind='bar', facecolor='lightblue')
fig.set_xticklabels(x_labels)
fig.set_title('Counts of trips by passenger count')
fig.set_xlabel('Passenger count in trips')
fig.set_ylabel('Trip counts')
plt.show()
```

**ВЫХОДНЫЕ ДАННЫЕ:**

![Частота поездок в зависимости от количества пассажиров](./media/spark-data-exploration-modeling/trip-freqency-by-passenger-count.png)

С помощью кнопок в меню **Тип** в записной книжке можно выбрать один из нескольких типов визуализации (таблица либо круговая, линейная, комбинированная или столбчатая диаграмма). Здесь показано представление столбчатой диаграммы.

### <a name="plot-a-histogram-of-tip-amounts-and-how-tip-amount-varies-by-passenger-count-and-fare-amounts"></a>Построение гистограммы суммы чаевых и ее зависимости от количества пассажиров и тарифа.

Используйте SQL-запрос для выборки данных.

```sql
# PLOT HISTOGRAM OF TIP AMOUNTS AND VARIATION BY PASSENGER COUNT AND PAYMENT TYPE

# HIVEQL QUERY AGAINST THE sqlContext
%%sql -q -o sqlResults
SELECT fare_amount, passenger_count, tip_amount, tipped 
FROM taxi_train 
WHERE passenger_count > 0 
AND passenger_count < 7 
AND fare_amount > 0 
AND fare_amount < 200 
AND payment_type in ('CSH', 'CRD') 
AND tip_amount > 0 
AND tip_amount < 25
```

В этой ячейке кода используется SQL-запрос для формирования трех видов графического представления данных.

```python
# RUN THE CODE LOCALLY ON THE JUPYTER SERVER
%%local

# HISTOGRAM OF TIP AMOUNTS AND PASSENGER COUNT
ax1 = sqlResults[['tip_amount']].plot(kind='hist', bins=25, facecolor='lightblue')
ax1.set_title('Tip amount distribution')
ax1.set_xlabel('Tip Amount ($)')
ax1.set_ylabel('Counts')
plt.suptitle('')
plt.show()

# TIP BY PASSENGER COUNT
ax2 = sqlResults.boxplot(column=['tip_amount'], by=['passenger_count'])
ax2.set_title('Tip amount by Passenger count')
ax2.set_xlabel('Passenger count')
ax2.set_ylabel('Tip Amount ($)')
plt.suptitle('')
plt.show()

# TIP AMOUNT BY FARE AMOUNT, POINTS ARE SCALED BY PASSENGER COUNT
ax = sqlResults.plot(kind='scatter', x= 'fare_amount', y = 'tip_amount', c='blue', alpha = 0.10, s=5*(sqlResults.passenger_count))
ax.set_title('Tip amount by Fare amount')
ax.set_xlabel('Fare Amount ($)')
ax.set_ylabel('Tip Amount ($)')
plt.axis([-2, 100, -2, 20])
plt.show()
```

**ВЫХОДНЫЕ ДАННЫЕ:** 

![Распределение суммы чаевых](./media/spark-data-exploration-modeling/tip-amount-distribution.png)

![Сумма чаевых в зависимости от количества пассажиров](./media/spark-data-exploration-modeling/tip-amount-by-passenger-count.png)

![Сумма чаевых в зависимости от тарифа](./media/spark-data-exploration-modeling/tip-amount-by-fare-amount.png)

## <a name="prepare-the-data"></a>Подготовка данных

В этом разделе приведен пример кода для выполнения процедур по подготовке данных, которые будут использоваться в моделях машинного обучения. Здесь также показано, как выполнять следующие задачи:

* создание нового признака путем группирования часов в периоды трафика;
* индексация и кодирование категориальных признаков;
* Создание объектов с помеченной вершиной в качестве ввода для функций машинного обучения
* Создание случайной вложенной выборки данных и ее разделение на наборы для обучения и тестирования
* масштабирование признаков;
* кэширование объектов в памяти.

### <a name="create-a-new-feature-by-binning-hours-into-traffic-time-buckets"></a>создание нового признака путем группирования часов в периоды трафика;

В следующем коде показано, как создать признак путем группирования часов в периоды трафика, а затем кэшировать итоговый фрейм данных в памяти. Если устойчивые распределенные наборы данных и фреймы данных используются постоянно, на их кэширование необходимо меньше времени. В этом пошаговом руководстве процесс кэширования имеющихся данных выполняется в несколько этапов. 

```python
# CREATE FOUR BUCKETS FOR TRAFFIC TIMES
sqlStatement = """
    SELECT *,
    CASE
     WHEN (pickup_hour <= 6 OR pickup_hour >= 20) THEN "Night" 
     WHEN (pickup_hour >= 7 AND pickup_hour <= 10) THEN "AMRush" 
     WHEN (pickup_hour >= 11 AND pickup_hour <= 15) THEN "Afternoon"
     WHEN (pickup_hour >= 16 AND pickup_hour <= 19) THEN "PMRush"
    END as TrafficTimeBins
    FROM taxi_train 
"""
taxi_df_train_with_newFeatures = sqlContext.sql(sqlStatement)

# CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
# THE .COUNT() GOES THROUGH THE ENTIRE DATA-FRAME,
# MATERIALIZES IT IN MEMORY, AND GIVES THE COUNT OF ROWS.
taxi_df_train_with_newFeatures.cache()
taxi_df_train_with_newFeatures.count()
```

**ВЫХОДНЫЕ ДАННЫЕ:**

126 050.

### <a name="index-and-encode-categorical-features-for-input-into-modeling-functions"></a>Индексация и кодирование категориальных признаков в качестве ввода для функций моделирования

В этом разделе показано, как индексировать и кодировать категориальные признаки в качестве ввода для функций моделирования. Перед использованием в функциях моделирования и прогнозирования MLlib категориальные входные данные сначала необходимо проиндексировать или закодировать. В зависимости от модели этот процесс происходит по-разному:  

* Для **моделирования на основе дерева** необходимо, чтобы категории были закодированы как числовые значения (например, признак с тремя категориями можно закодировать с помощью чисел 0, 1, 2). Этот алгоритм предоставляется функцией [StringIndexer](https://spark.apache.org/docs/latest/ml-features.html#stringindexer) в MLlib. С помощью этой функции столбец меток строкового типа кодируется как столбец с индексами меток, которые упорядочены по частоте меток. Несмотря на то, что для данных задаются числовые индексы для ввода и дальнейшей обработки, в алгоритмах на основе дерева можно указать, что данные следует рассматривать как категории. 
* Для **моделей логистической и линейной регрессии** требуется прямое кодирование, при котором, например, признак с тремя категориями можно разделить на три столбца признаков, в каждом из которых в зависимости от категории наблюдения содержатся значения 0 или 1. Для этого можно использовать функцию [OneHotEncoder](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OneHotEncoder.html#sklearn.preprocessing.OneHotEncoder) в MLlib. Этот кодировщик сопоставляет столбец с индексами меток со столбцом двоичных векторов как минимум с одним отдельным значением. Благодаря этой кодировке алгоритмы, для которых необходимы признаки с числовыми значениями (например, логистическая регрессия), можно применять к категориальным признакам.

Ниже приведен пример кода, с помощью которого можно проиндексировать и закодировать категориальные признаки.

```python
# INDEX AND ENCODE CATEGORICAL FEATURES

# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD PYSPARK LIBRARIES    
from pyspark.ml.feature import OneHotEncoder, StringIndexer, VectorAssembler, VectorIndexer

# INDEX AND ENCODE VENDOR_ID
stringIndexer = StringIndexer(inputCol="vendor_id", outputCol="vendorIndex")
model = stringIndexer.fit(taxi_df_train_with_newFeatures) # Input data-frame is the cleaned one from above
indexed = model.transform(taxi_df_train_with_newFeatures)
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

# INDEX AND TRAFFIC TIME BINS
stringIndexer = StringIndexer(inputCol="TrafficTimeBins", outputCol="TrafficTimeBinsIndex")
model = stringIndexer.fit(encoded3)
indexed = model.transform(encoded3)
encoder = OneHotEncoder(dropLast=False, inputCol="TrafficTimeBinsIndex", outputCol="TrafficTimeBinsVec")
encodedFinal = encoder.transform(indexed)

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**ВЫХОДНЫЕ ДАННЫЕ:**

Время выполнения кода выше: 1,28 секунды

### <a name="create-labeled-point-objects-for-input-into-ml-functions"></a>Создание объектов с помеченной вершиной в качестве ввода для функций машинного обучения

В этом разделе содержится код, с помощью которого можно индексировать категориальные текстовые данные в тип данных с помеченной вершиной, а затем закодировать их. После этого данные можно использовать для обучения и проверки модели логистической регрессии MLlib и других моделей классификации. Объекты с помеченной вершиной отформатированы в устойчивые распределенные наборы данных, которые можно использовать в качестве входных для большинства алгоритмов машинного обучения в MLlib. Объект [с помеченной вершиной](https://spark.apache.org/docs/latest/mllib-data-types.html#labeled-point) — это разреженный или плотный локальный вектор, связанный с меткой или ответом.  

В этом разделе содержится код, с помощью которого можно индексировать категориальные текстовые данные в тип данных [с помеченной вершиной](https://spark.apache.org/docs/latest/mllib-data-types.html#labeled-point), а затем закодировать их. После этого данные можно использовать для обучения и проверки модели логистической регрессии MLlib и других моделей классификации. Объекты с помеченной вершиной — это устойчивые распределенные наборы данных, состоящие из метки (переменной ответа или целевой переменной) и вектора признаков. Данные в таком формате используются в качестве входных для алгоритмов машинного обучения в MLlib.

Ниже приведен пример кода, с помощью которого можно проиндексировать и закодировать текстовые характеристики бинарной классификации.

```python
# FUNCTIONS FOR BINARY CLASSIFICATION

# LOAD LIBRARIES
from pyspark.mllib.regression import LabeledPoint
from numpy import array

# INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
def parseRowIndexingBinary(line):
    features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.TrafficTimeBinsIndex,
                         line.pickup_hour, line.weekday, line.passenger_count, line.trip_time_in_secs, 
                         line.trip_distance, line.fare_amount])
    labPt = LabeledPoint(line.tipped, features)
    return  labPt

# ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO LOGISTIC REGRESSION MODELS
def parseRowOneHotBinary(line):
    features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
                                        line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
                                        line.vendorVec.toArray(), line.rateVec.toArray(), 
                                        line.paymentVec.toArray(), line.TrafficTimeBinsVec.toArray()), axis=0)
    labPt = LabeledPoint(line.tipped, features)
    return  labPt
```

Ниже приведен пример кода, с помощью которого можно проиндексировать и закодировать категориальные текстовые характеристики для анализа линейной регрессии.

```python
# FUNCTIONS FOR REGRESSION WITH TIP AMOUNT AS TARGET VARIABLE

# ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
def parseRowIndexingRegression(line):
    features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.TrafficTimeBinsIndex, 
                         line.pickup_hour, line.weekday, line.passenger_count, line.trip_time_in_secs, 
                         line.trip_distance, line.fare_amount])

    labPt = LabeledPoint(line.tip_amount, features)
    return  labPt

# INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO LINEAR REGRESSION MODELS
def parseRowOneHotRegression(line):
    features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
                                        line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
                                        line.vendorVec.toArray(), line.rateVec.toArray(), 
                                        line.paymentVec.toArray(), line.TrafficTimeBinsVec.toArray()), axis=0)
    labPt = LabeledPoint(line.tip_amount, features)
    return  labPt
```

### <a name="create-a-random-subsampling-of-the-data-and-split-it-into-training-and-testing-sets"></a>Создание случайной вложенной выборки данных и ее разделение на наборы для обучения и тестирования

С помощью кода в этом разделе создается случайная выборка данных (25 % всех данных). Несмотря на то, что из-за небольшого размера используемого набора данных, этот шаг выполнять необязательно, мы все равно покажем, как создавать выборку. Это поможет вам в случае возникновения проблем. Если выборки большие, данный процесс может значительно сэкономить время обучения моделей. Затем мы разобьем выборку данных на наборы для обучения (75 %) и тестирования (25 %), которые будут использоваться для двоичной классификации и регрессии.

```python
# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD PYSPARK LIBRARIES
from pyspark.sql.functions import rand

# SPECIFY SAMPLING AND SPLITTING FRACTIONS
samplingFraction = 0.25;
trainingFraction = 0.75; testingFraction = (1-trainingFraction);
seed = 1234;
encodedFinalSampled = encodedFinal.sample(False, samplingFraction, seed=seed)

# SPLIT SAMPLED DATA-FRAME INTO TRAIN/TEST
# INCLUDE RAND COLUMN FOR CREATING CROSS-VALIDATION FOLDS (FOR USE LATER IN AN ADVANCED TOPIC)
dfTmpRand = encodedFinalSampled.select("*", rand(0).alias("rand"));
trainData, testData = dfTmpRand.randomSplit([trainingFraction, testingFraction], seed=seed);

# FOR BINARY CLASSIFICATION TRAINING AND TESTING
indexedTRAINbinary = trainData.map(parseRowIndexingBinary)
indexedTESTbinary = testData.map(parseRowIndexingBinary)
oneHotTRAINbinary = trainData.map(parseRowOneHotBinary)
oneHotTESTbinary = testData.map(parseRowOneHotBinary)

# FOR REGRESSION TRAINING AND TESTING
indexedTRAINreg = trainData.map(parseRowIndexingRegression)
indexedTESTreg = testData.map(parseRowIndexingRegression)
oneHotTRAINreg = trainData.map(parseRowOneHotRegression)
oneHotTESTreg = testData.map(parseRowOneHotRegression)

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**ВЫХОДНЫЕ ДАННЫЕ:**

Время выполнения кода выше: 0,24 секунды

### <a name="feature-scaling"></a>масштабирование признаков;

Масштабирование признаков (или нормализация данных) гарантирует, что для признаков с широко распределенными значениями не задано высокое значение веса в целевой функции. В коде для масштабирования признаков используется функция [StandardScaler](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.mllib.classification.LogisticRegressionWithLBFGS.html#pyspark.mllib.classification.LogisticRegressionWithLBFGS
), которая позволяет масштабировать признаки в зависимости от изменений единицы. MLlib предоставляет функцию StandardScaler для выполнения линейной регрессии с применением метода стохастического градиента. Этот алгоритм широко используется для обучения других моделей машинного обучения (например, регуляризованной регрессии или метода опорных векторов).

> [!NOTE]
> Алгоритм LinearRegressionWithSGD чувствителен к масштабированию признаков.

Ниже приведен код для масштабирования переменных, пригодный для использования с регуляризованным линейным алгоритмом с применением стохастического градиента.

```python
# FEATURE SCALING

# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD PYSPARK LIBRARIES
from pyspark.mllib.regression import LabeledPoint
from pyspark.mllib.linalg import Vectors
from pyspark.mllib.feature import StandardScaler, StandardScalerModel
from pyspark.mllib.util import MLUtils

# SCALE VARIABLES FOR REGULARIZED LINEAR SGD ALGORITHM
label = oneHotTRAINreg.map(lambda x: x.label)
features = oneHotTRAINreg.map(lambda x: x.features)
scaler = StandardScaler(withMean=False, withStd=True).fit(features)
dataTMP = label.zip(scaler.transform(features.map(lambda x: Vectors.dense(x.toArray()))))
oneHotTRAINregScaled = dataTMP.map(lambda x: LabeledPoint(x[0], x[1]))

label = oneHotTESTreg.map(lambda x: x.label)
features = oneHotTESTreg.map(lambda x: x.features)
scaler = StandardScaler(withMean=False, withStd=True).fit(features)
dataTMP = label.zip(scaler.transform(features.map(lambda x: Vectors.dense(x.toArray()))))
oneHotTESTregScaled = dataTMP.map(lambda x: LabeledPoint(x[0], x[1]))

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**ВЫХОДНЫЕ ДАННЫЕ:**

Время выполнения кода выше: 13,17 секунды

### <a name="cache-objects-in-memory"></a>кэширование объектов в памяти.

Время на обучение и тестирование алгоритмов машинного обучения можно сократить, кэшировав объекты входного фрейма данных, которые использовались в качестве масштабируемых признаков, а также признаков для классификации и регрессии.

```python
# RECORD START TIME
timestart = datetime.datetime.now()

# FOR BINARY CLASSIFICATION TRAINING AND TESTING
indexedTRAINbinary.cache()
indexedTESTbinary.cache()
oneHotTRAINbinary.cache()
oneHotTESTbinary.cache()

# FOR REGRESSION TRAINING AND TESTING
indexedTRAINreg.cache()
indexedTESTreg.cache()
oneHotTRAINreg.cache()
oneHotTESTreg.cache()

# SCALED FEATURES
oneHotTRAINregScaled.cache()
oneHotTESTregScaled.cache()

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**ВЫХОДНЫЕ ДАННЫЕ:** 

Время выполнения кода выше: 0,15 секунд

## <a name="train-a-binary-classification-model"></a>Обучение модели двоичной классификации

В этом разделе показано, как с помощью трех моделей для выполнения задач двоичной классификации спрогнозировать, будут ли оставлены чаевые за поездку в такси. Мы используем следующие модели:

* регуляризованная логистическая регрессия; 
* модель случайного леса;
* градиентный бустинг деревьев.

Процесс создания модели состоит из следующих этапов: 

1. **Обучение модели** с использованием данных с одним набором параметров.
2. **Оценка модели** на основе набора тестовых данных с метриками.
3. **Сохранение модели** в большом двоичном объекте для последующего использования.

### <a name="classification-using-logistic-regression"></a>Классификация с применением логистической регрессии

Приведенный в этом разделе код предназначен для обучения, анализа и сохранения модели логистической регрессии с применением [LBFGS](https://en.wikipedia.org/wiki/Broyden%E2%80%93Fletcher%E2%80%93Goldfarb%E2%80%93Shanno_algorithm) , с помощью которой можно спрогнозировать, будут ли выплачены чаевые за поездку в такси по Нью-Йорку, на основе набора данных и тарифов.

**Обучение модели логистической регрессии с использованием перекрестной проверки и перебора параметров**

```python
# LOGISTIC REGRESSION CLASSIFICATION WITH CV AND HYPERPARAMETER SWEEPING

# GET ACCURACY FOR HYPERPARAMETERS BASED ON CROSS-VALIDATION IN TRAINING DATA-SET

# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD LIBRARIES
from pyspark.mllib.classification import LogisticRegressionWithLBFGS 
from sklearn.metrics import roc_curve,auc
from pyspark.mllib.evaluation import BinaryClassificationMetrics
from pyspark.mllib.evaluation import MulticlassMetrics


# CREATE MODEL WITH ONE SET OF PARAMETERS
logitModel = LogisticRegressionWithLBFGS.train(oneHotTRAINbinary, iterations=20, initialWeights=None, 
                                               regParam=0.01, regType='l2', intercept=True, corrections=10, 
                                               tolerance=0.0001, validateData=True, numClasses=2)

# PRINT COEFFICIENTS AND INTERCEPT OF THE MODEL
# NOTE: There are 20 coefficient terms for the 10 features, 
#       and the different categories for features: vendorVec (2), rateVec, paymentVec (6), TrafficTimeBinsVec (4)
print("Coefficients: " + str(logitModel.weights))
print("Intercept: " + str(logitModel.intercept))

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**ВЫХОДНЫЕ ДАННЫЕ:** 

Coefficients: [0.0082065285375, -0.0223675576104, -0.0183812028036, -3.48124578069e-05, -0.00247646947233, -0.00165897881503, 0.0675394837328, -0.111823113101, -0.324609912762, -0.204549780032, -1.36499216354, 0.591088507921, -0.664263411392, -1.00439726852, 3.46567827545, -3.51025855172, -0.0471341112232, -0.043521833294, 0.000243375810385, 0.054518719222]

Intercept: -0.0111216486893

Время выполнения кода выше: 14,43 секунды

**Оценка модели бинарной классификации со стандартными метриками**

```python
#EVALUATE LOGISTIC REGRESSION MODEL WITH LBFGS

# RECORD START TIME
timestart = datetime.datetime.now()

# PREDICT ON TEST DATA WITH MODEL
predictionAndLabels = oneHotTESTbinary.map(lambda lp: (float(logitModel.predict(lp.features)), lp.label))

# INSTANTIATE METRICS OBJECT
metrics = BinaryClassificationMetrics(predictionAndLabels)

# AREA UNDER PRECISION-RECALL CURVE
print("Area under PR = %s" % metrics.areaUnderPR)

# AREA UNDER ROC CURVE
print("Area under ROC = %s" % metrics.areaUnderROC)
metrics = MulticlassMetrics(predictionAndLabels)

# OVERALL STATISTICS
precision = metrics.precision()
recall = metrics.recall()
f1Score = metrics.fMeasure()
print("Summary Stats")
print("Precision = %s" % precision)
print("Recall = %s" % recall)
print("F1 Score = %s" % f1Score)


## SAVE MODEL WITH DATE-STAMP
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
logisticregressionfilename = "LogisticRegressionWithLBFGS_" + datestamp;
dirfilename = modelDir + logisticregressionfilename;
logitModel.save(sc, dirfilename);

# OUTPUT PROBABILITIES AND REGISTER TEMP TABLE
logitModel.clearThreshold(); # This clears threshold for classification (0.5) and outputs probabilities
predictionAndLabelsDF = predictionAndLabels.toDF()
predictionAndLabelsDF.registerTempTable("tmp_results");

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds";
```

**ВЫХОДНЫЕ ДАННЫЕ:** 

Area under PR = 0.985297691373

Area under ROC = 0.983714670256

Summary Stats

Precision = 0.984304060189

Recall = 0.984304060189

F1 Score = 0.984304060189

Время выполнения кода выше: 57,61 секунды

**Графическое представление кривой ROC.**

*predictionAndLabelsDF* регистрируется как таблица (*tmp_results*) в предыдущей ячейке. *tmp_results* может использоваться для выполнения запросов и вывода результатов во фрейм данных sqlResults для построения графика. Ниже приведен код:

```python
# QUERY RESULTS                              
%%sql -q -o sqlResults
SELECT * from tmp_results
```

Ниже приведен код для создания прогнозов и отображения кривой ROC.

```python
# MAKE PREDICTIONS AND PLOT ROC-CURVE

# RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
%%local
%matplotlib inline
from sklearn.metrics import roc_curve,auc

# MAKE PREDICTIONS
predictions_pddf = test_predictions.rename(columns={'_1': 'probability', '_2': 'label'})
prob = predictions_pddf["probability"] 
fpr, tpr, thresholds = roc_curve(predictions_pddf['label'], prob, pos_label=1);
roc_auc = auc(fpr, tpr)

# PLOT ROC CURVE
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

**ВЫХОДНЫЕ ДАННЫЕ:**

![Logistic regression ROC curve.png](./media/spark-data-exploration-modeling/logistic-regression-roc-curve.png)

### <a name="random-forest-classification"></a>Классификация случайного леса

Приведенный в этом разделе код предназначен для обучения, анализа и сохранения модели случайного леса, с помощью которой на основе набора данных о поездках и тарифах по Нью-Йорку можно спрогнозировать, оставит ли пассажир чаевые за поездку в такси.

```python
#PREDICT WHETHER A TIP IS PAID OR NOT USING RANDOM FOREST

# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD PYSPARK LIBRARIES
from pyspark.mllib.tree import RandomForest, RandomForestModel
from pyspark.mllib.util import MLUtils
from pyspark.mllib.evaluation import BinaryClassificationMetrics
from pyspark.mllib.evaluation import MulticlassMetrics

# SPECIFY NUMBER OF CATEGORIES FOR CATEGORICAL FEATURES. FEATURE #0 HAS 2 CATEGORIES, FEATURE #2 HAS 2 CATEGORIES, AND SO ON
categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}

# TRAIN RANDOMFOREST MODEL
rfModel = RandomForest.trainClassifier(indexedTRAINbinary, numClasses=2, 
                                       categoricalFeaturesInfo=categoricalFeaturesInfo,
                                       numTrees=25, featureSubsetStrategy="auto",
                                       impurity='gini', maxDepth=5, maxBins=32)
## UN-COMMENT IF YOU WANT TO PRINT TREES
#print('Learned classification forest model:')
#print(rfModel.toDebugString())

# PREDICT ON TEST DATA AND EVALUATE
predictions = rfModel.predict(indexedTESTbinary.map(lambda x: x.features))
predictionAndLabels = indexedTESTbinary.map(lambda lp: lp.label).zip(predictions)

# AREA UNDER ROC CURVE
metrics = BinaryClassificationMetrics(predictionAndLabels)
print("Area under ROC = %s" % metrics.areaUnderROC)

# PERSIST MODEL IN BLOB
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
rfclassificationfilename = "RandomForestClassification_" + datestamp;
dirfilename = modelDir + rfclassificationfilename;

rfModel.save(sc, dirfilename);

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**ВЫХОДНЫЕ ДАННЫЕ:**

Area under ROC = 0.985297691373

Время выполнения кода выше: 31,09 секунды

### <a name="gradient-boosting-trees-classification"></a>Классификация с применением модели градиентного бустинга деревьев

Приведенный в этом разделе код предназначен для обучения, анализа и сохранения модели градиентного бустинга деревьев, с помощью которой можно спрогнозировать, будут ли выплачены чаевые за поездку в такси по Нью-Йорку, на основе набора данных и тарифов.

```python
#PREDICT WHETHER A TIP IS PAID OR NOT USING GRADIENT BOOSTING TREES

# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD PYSPARK LIBRARIES
from pyspark.mllib.tree import GradientBoostedTrees, GradientBoostedTreesModel

# SPECIFY NUMBER OF CATEGORIES FOR CATEGORICAL FEATURES. FEATURE #0 HAS 2 CATEGORIES, FEATURE #2 HAS 2 CATEGORIES, AND SO ON
categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}

gbtModel = GradientBoostedTrees.trainClassifier(indexedTRAINbinary, categoricalFeaturesInfo=categoricalFeaturesInfo, numIterations=5)
## UNCOMMENT IF YOU WANT TO PRINT TREE DETAILS
#print('Learned classification GBT model:')
#print(bgtModel.toDebugString())

# PREDICT ON TEST DATA AND EVALUATE
predictions = gbtModel.predict(indexedTESTbinary.map(lambda x: x.features))
predictionAndLabels = indexedTESTbinary.map(lambda lp: lp.label).zip(predictions)

# AREA UNDER ROC CURVE
metrics = BinaryClassificationMetrics(predictionAndLabels)
print("Area under ROC = %s" % metrics.areaUnderROC)

# PERSIST MODEL IN A BLOB
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
btclassificationfilename = "GradientBoostingTreeClassification_" + datestamp;
dirfilename = modelDir + btclassificationfilename;

gbtModel.save(sc, dirfilename)

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**ВЫХОДНЫЕ ДАННЫЕ:**

Area under ROC = 0.985297691373

Время выполнения кода выше: 19,76 секунды

## <a name="train-a-regression-model"></a>Обучение регрессионной модели

В этом разделе показано, как с помощью трех моделей регрессии спрогнозировать сумму чаевых в зависимости от других признаков. Мы используем следующие модели:

* регуляризованная линейная регрессия;
* случайный лес;
* градиентный бустинг деревьев.

Эти модели описаны в начале статьи. Процесс создания модели состоит из следующих этапов: 

1. **Обучение модели** с использованием данных с одним набором параметров.
2. **Оценка модели** на основе набора тестовых данных с метриками.
3. **Сохранение модели** в большом двоичном объекте для последующего использования.

### <a name="linear-regression-with-sgd"></a>Линейная регрессия с применением SGD

С помощью кода, приведенного в этом разделе, на основе масштабируемых признаков можно обучить модель линейной регрессии, в которой для оптимизации используется метод стохастического градиента, а также оценить, проанализировать и сохранить ее в хранилище BLOB-объектов Azure.

> [!TIP]
> На практике в моделях LinearRegressionWithSGD часто возникают проблемы с конвергенцией. Чтобы получить допустимую модель, необходимо осторожно изменить или оптимизировать параметры. Проблему конвергенции можно решить, выполнив масштабирование переменных.

```python
#PREDICT TIP AMOUNTS USING LINEAR REGRESSION WITH SGD

# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD LIBRARIES
from pyspark.mllib.regression import LabeledPoint, LinearRegressionWithSGD, LinearRegressionModel
from pyspark.mllib.evaluation import RegressionMetrics
from scipy import stats

# USE SCALED FEATURES TO TRAIN MODEL
linearModel = LinearRegressionWithSGD.train(oneHotTRAINregScaled, iterations=100, step = 0.1, regType='l2', regParam=0.1, intercept = True)

# PRINT COEFFICIENTS AND INTERCEPT OF THE MODEL
# NOTE: There are 20 coefficient terms for the 10 features, 
#       and the different categories for features: vendorVec (2), rateVec, paymentVec (6), TrafficTimeBinsVec (4)
print("Coefficients: " + str(linearModel.weights))
print("Intercept: " + str(linearModel.intercept))

# SCORE ON SCALED TEST DATA-SET & EVALUATE
predictionAndLabels = oneHotTESTregScaled.map(lambda lp: (float(linearModel.predict(lp.features)), lp.label))
testMetrics = RegressionMetrics(predictionAndLabels)

# PRINT TEST METRICS
print("RMSE = %s" % testMetrics.rootMeanSquaredError)
print("R-sqr = %s" % testMetrics.r2)

# SAVE MODEL WITH DATE-STAMP IN THE DEFAULT BLOB FOR THE CLUSTER
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
linearregressionfilename = "LinearRegressionWithSGD_" + datestamp;
dirfilename = modelDir + linearregressionfilename;

linearModel.save(sc, dirfilename)

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**ВЫХОДНЫЕ ДАННЫЕ:**

Coefficients: [0.00457675809917, -0.0226314167349, -0.0191910355236, 0.246793409578, 0.312047890459, 0.359634405999, 0.00928692253981, -0.000987181489428, -0.0888306617845, 0.0569376211553, 0.115519551711, 0.149250164995, -0.00990211159703, -0.00637410344522, 0.545083566179, -0.536756072402, 0.0105762393099, -0.0130117577055, 0.0129304737772, -0.00171065945959]

Intercept: 0.853872718283

RMSE = 1.24190115863

R-sqr = 0.608017146081

Время выполнения кода выше: 58,42 секунды

### <a name="random-forest-regression"></a>Регрессия с использованием модели случайного леса

Приведенный в этом разделе код предназначен для обучения, анализа и сохранения регрессии случайного леса, с помощью которой можно спрогнозировать суму чаевых за поездку в такси по Нью-Йорку.

```python
#PREDICT TIP AMOUNTS USING RANDOM FOREST

# RECORD START TIME
timestart= datetime.datetime.now()

# LOAD PYSPARK LIBRARIES
from pyspark.mllib.tree import RandomForest, RandomForestModel
from pyspark.mllib.util import MLUtils
from pyspark.mllib.evaluation import RegressionMetrics


## TRAIN MODEL
categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}
rfModel = RandomForest.trainRegressor(indexedTRAINreg, categoricalFeaturesInfo=categoricalFeaturesInfo,
                                    numTrees=25, featureSubsetStrategy="auto",
                                    impurity='variance', maxDepth=10, maxBins=32)
## UN-COMMENT IF YOU WANT TO PRING TREES
#print('Learned classification forest model:')
#print(rfModel.toDebugString())

## PREDICT AND EVALUATE ON TEST DATA-SET
predictions = rfModel.predict(indexedTESTreg.map(lambda x: x.features))
predictionAndLabels = oneHotTESTreg.map(lambda lp: lp.label).zip(predictions)

# TEST METRICS
testMetrics = RegressionMetrics(predictionAndLabels)
print("RMSE = %s" % testMetrics.rootMeanSquaredError)
print("R-sqr = %s" % testMetrics.r2)

# SAVE MODEL IN BLOB
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
rfregressionfilename = "RandomForestRegression_" + datestamp;
dirfilename = modelDir + rfregressionfilename;

rfModel.save(sc, dirfilename);

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**ВЫХОДНЫЕ ДАННЫЕ:**

RMSE = 0.891209218139

R-sqr = 0.759661334921

Время выполнения кода выше: 49,21 секунды

### <a name="gradient-boosting-trees-regression"></a>Регрессия с применением градиентного бустинга деревьев

Приведенный в этом разделе код предназначен для обучения, анализа и сохранения модели градиентного бустинга деревьев, с помощью которой можно спрогнозировать суму чаевых за поездку в такси по Нью-Йорку.

**Обучение и анализ**

```python
#PREDICT TIP AMOUNTS USING GRADIENT BOOSTING TREES

# RECORD START TIME
timestart= datetime.datetime.now()

# LOAD PYSPARK LIBRARIES
from pyspark.mllib.tree import GradientBoostedTrees, GradientBoostedTreesModel
from pyspark.mllib.util import MLUtils

## TRAIN MODEL
categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}
gbtModel = GradientBoostedTrees.trainRegressor(indexedTRAINreg, categoricalFeaturesInfo=categoricalFeaturesInfo, 
                                                numIterations=10, maxBins=32, maxDepth = 4, learningRate=0.1)

## EVALUATE A TEST DATA-SET
predictions = gbtModel.predict(indexedTESTreg.map(lambda x: x.features))
predictionAndLabels = indexedTESTreg.map(lambda lp: lp.label).zip(predictions)

# TEST METRICS
testMetrics = RegressionMetrics(predictionAndLabels)
print("RMSE = %s" % testMetrics.rootMeanSquaredError)
print("R-sqr = %s" % testMetrics.r2)

# SAVE MODEL IN BLOB
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
btregressionfilename = "GradientBoostingTreeRegression_" + datestamp;
dirfilename = modelDir + btregressionfilename;
gbtModel.save(sc, dirfilename)

# CONVERT RESULTS TO DF AND REGISTER TEMP TABLE
test_predictions = sqlContext.createDataFrame(predictionAndLabels)
test_predictions.registerTempTable("tmp_results");

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**ВЫХОДНЫЕ ДАННЫЕ:**

RMSE = 0.908473148639

R-sqr = 0.753835096681

Время выполнения кода выше: 34,52 секунды

**Графическое представления**

*tmp_results* регистрируется как таблица Hive в предыдущей ячейке. Результаты из таблицы передаются во фрейм данных *sqlResults* для формирования графического представления. Ниже приведен код:

```python
# PLOT SCATTER-PLOT BETWEEN ACTUAL AND PREDICTED TIP VALUES

# SELECT RESULTS
%%sql -q -o sqlResults
SELECT * from tmp_results
```

Ниже приведен код для формирования графического представления данных с использованием сервера Jupyter.

```python
# RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
%%local
%matplotlib inline
import numpy as np

# PLOT 
ax = test_predictions_pddf.plot(kind='scatter', figsize = (6,6), x='_1', y='_2', color='blue', alpha = 0.25, label='Actual vs. predicted');
fit = np.polyfit(test_predictions_pddf['_1'], test_predictions_pddf['_2'], deg=1)
ax.set_title('Actual vs. Predicted Tip Amounts ($)')
ax.set_xlabel("Actual")
ax.set_ylabel("Predicted")
ax.plot(test_predictions_pddf['_1'], fit[0] * test_predictions_pddf['_1'] + fit[1], color='magenta')
plt.axis([-1, 20, -1, 20])
plt.show(ax)
```

**ВЫХОДНЫЕ ДАННЫЕ:**

![Actual-vs-predicted-tip-amounts](./media/spark-data-exploration-modeling/actual-vs-predicted-tips.png)

## <a name="clean-up-objects-from-memory"></a>Очистка объектов из памяти

Удалите объекты, кэшированные в памяти, с помощью метода `unpersist()` .

```python
# REMOVE ORIGINAL DFs
taxi_df_train_cleaned.unpersist()
taxi_df_train_with_newFeatures.unpersist()

# FOR BINARY CLASSIFICATION TRAINING AND TESTING
indexedTRAINbinary.unpersist()
indexedTESTbinary.unpersist()
oneHotTRAINbinary.unpersist()
oneHotTESTbinary.unpersist()

# FOR REGRESSION TRAINING AND TESTING
indexedTRAINreg.unpersist()
indexedTESTreg.unpersist()
oneHotTRAINreg.unpersist()
oneHotTESTreg.unpersist()

# SCALED FEATURES
oneHotTRAINregScaled.unpersist()
oneHotTESTregScaled.unpersist()
```

## <a name="save-the-models"></a>Сохранение моделей

Чтобы использовать и оценивать независимые наборы данных, описанные в разделе [Оценка моделей машинного обучения, созданных с помощью Spark](spark-model-consumption.md), необходимо скопировать и вставить в записную книжку Jupyter Consumption имена файлов с сохраненными моделями, созданными с помощью инструкций из этой статьи. Ниже приведен код для вывода путей к необходимым файлам моделей.

```python
# MODEL FILE LOCATIONS FOR CONSUMPTION
print "logisticRegFileLoc = modelDir + \"" + logisticregressionfilename + "\"";
print "linearRegFileLoc = modelDir + \"" + linearregressionfilename + "\"";
print "randomForestClassificationFileLoc = modelDir + \"" + rfclassificationfilename + "\"";
print "randomForestRegFileLoc = modelDir + \"" + rfregressionfilename + "\"";
print "BoostedTreeClassificationFileLoc = modelDir + \"" + btclassificationfilename + "\"";
print "BoostedTreeRegressionFileLoc = modelDir + \"" + btregressionfilename + "\"";
```

**OUTPUT**

logisticRegFileLoc = modelDir + "LogisticRegressionWithLBFGS_2016-05-0317_03_23.516568"

linearRegFileLoc = modelDir + "LinearRegressionWithSGD_2016-05-0317_05_21.577773"

randomForestClassificationFileLoc = modelDir + "RandomForestClassification_2016-05-0317_04_11.950206"

randomForestRegFileLoc = modelDir + "RandomForestRegression_2016-05-0317_06_08.723736"

BoostedTreeClassificationFileLoc = modelDir + "GradientBoostingTreeClassification_2016-05-0317_04_36.346583"

BoostedTreeRegressionFileLoc = modelDir + "GradientBoostingTreeRegression_2016-05-0317_06_51.737282"

## <a name="whats-next"></a>Дальнейшие действия

После создания моделей регрессии и классификации с помощью Spark MlLib необходимо ознакомиться с процессом оценки и анализа этих моделей. С помощью записной книжки по расширенному исследованию и моделированию данных можно более подробно ознакомиться с тем, как использовать перекрестную проверку, очистку гиперпараметров и оценку модели. 

**Использование модели.** См. дополнительные сведения об [оценке и использованию моделей классификации и регрессии, созданных с помощью Spark](spark-model-consumption.md).

**Перекрестная проверка и перебор гиперпараметров.** Сведения об обучении моделей с помощью перекрестной проверки и перебора гиперпараметров см. в статье [Расширенное исследование и моделирование данных с помощью Spark](spark-advanced-data-exploration-modeling.md).
