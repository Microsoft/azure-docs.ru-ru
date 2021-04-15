---
title: Расширенное исследование и моделирование данных с помощью Spark — процесс обработки и анализа данных группы
description: Используйте кластер HDInsight Spark для анализа данных, обучения бинарной классификации и моделей регрессии; при этом используются перекрестная проверка и оптимизация гиперпараметров.
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
ms.openlocfilehash: 37f0b48190bcbad7ab190da1d690a1a0485501a1
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/09/2021
ms.locfileid: "107259325"
---
# <a name="advanced-data-exploration-and-modeling-with-spark"></a>Расширенное исследование и моделирование данных с помощью Spark

В этом пошаговом руководстве рассматривается исследование, обучение бинарной классификации и моделей регрессии; при этом используются перекрестная проверка и оптимизация гиперпараметров для обучения моделей на примере набора данных о поездках в такси по Нью-Йорку и тарифах за 2013 г. с помощью HDInsight Spark. Здесь также подробно описаны этапы [анализа и обработки данных](./index.yml) с использованием кластера HDInsight Spark по обработке данных и больших двоичных объектов Azure для хранения данных и моделей. В ходе этой процедуры данные из большого двоичного объекта службы хранилища Azure сначала исследуются и визуализируются, а затем подготавливаются для создания прогнозных моделей. Чтобы создать код решения и составить соответствующие графики, использовался язык Python. Эти модели создаются с помощью набора средств Spark MLlib для выполнения задач двоичной классификации и регрессии. 

* Задача **двоичной классификации** : спрогнозировать, будут ли выплачены чаевые за поездку. 
* Задача **регрессии** : спрогнозировать сумму чаевых в зависимости от других признаков. 

Здесь также содержатся примеры кода, демонстрирующие процесс обучения, анализа и сохранения модели каждого типа. В этой статье рассматриваются некоторые из основ, описанные в статье [Исследование и моделирование данных с помощью Spark](spark-data-exploration-modeling.md). Но эта публикация более расширенная, так как в ней также используется перекрестная проверка с перебором гиперпараметров для обучения оптимально точных моделей классификации и регрессии. 

**Перекрестная проверка** — это метод, который оценивает, насколько хорошо модель, обученная на известном наборе данных, обобщается для прогнозирования характеристик наборов данных, на которых она не прошла обучение.  Здесь используется распространенная реализация, разделяющая набор данных на K сверток и затем обучающая модель путем циклического перебора по всем сверткам, кроме одной. Оценивается умение модели точно прогнозировать, когда проверенный независимый набор данных в этой свертке не используется для обучения модели.

**Оптимизация гиперпараметров** — это задача выбора набора гиперпараметров для алгоритма обучения, обычно с целью оптимизации меры производительности алгоритма на независимом наборе данных. **Гиперпараметры** — это значения, которые должны быть указаны за пределами процедуры обучения модели. Предположения относительно этих значений могут повлиять на гибкость и точность моделей. Деревья принятия решений содержат гиперпараметры, такие как требуемая глубина и число листьев в дереве. Для методов опорных векторов (SVM) необходимо задать условие штрафа за неправильную классификацию. 

Распространенный способ выполнения оптимизации гиперпараметров, который используется здесь, — поиск в сетке или **перебор параметров**. Такой поиск выполняется по подмножеству пространства гиперпараметров для алгоритма обучения. Перекрестная проверка может предоставить метрику производительности, чтобы отсортировать оптимальные результаты, выданные алгоритмом поиска в сетке. Использование перекрестной проверки с перебором гиперпараметров помогает ограничить такие проблемы, как лжевзаимосвязи модели с обучающими данными, чтобы модель сохраняла возможность применения к общему набору данных, из которого извлекаются данные для обучения.

Мы использовали такие модели, как логистическая и линейная регрессия, случайные леса и градиентный бустинг деревьев.

* [Линейная регрессия с применением SGD](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.mllib.regression.LinearRegressionWithSGD.html#pyspark.mllib.regression.LinearRegressionWithSGD
) — модель линейной регрессии, в которой используется метод стохастического градиента для оптимизации и масштабирование признаков для прогнозирования суммы чаевых. 
* [Логистическая регрессия с применением алгоритма LBFGS](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.mllib.classification.LogisticRegressionWithLBFGS.html
) (или логит-регрессия) — регрессионная модель, которая используется для классификации данных, если зависимая переменная является категориальной. Алгоритм LBFGS — это широко используемый в машинном обучении квазиньютоновский алгоритм оптимизации, который представляет собой модифицированный метод Бройдена — Флетчера — Гольдфарба — Шанно (BFGS) с ограниченным использованием компьютерной памяти.
* [Случайные леса](https://spark.apache.org/docs/latest/mllib-ensembles.html#Random-Forests) — это совокупности деревьев принятия решений.  Такие совокупности минимизируют необходимость в переобучении. Случайные леса используются для регрессии и классификации, могут обрабатывать категориальные функции и могут быть расширены до многоклассовой настройки классификации. С их помощью можно определять нелинейные зависимости и взаимодействия признаков. Этот метод не требует масштабирования признаков. Случайные леса — одна из самых эффективных моделей машинного обучения для задач классификации и регрессии.
* [Деревья с градиентным бустингом](https://spark.apache.org/docs/latest/ml-classification-regression.html#gradient-boosted-trees-gbts) (GBTS) — это совокупности деревьев принятия решений. GBTS предусматривает итеративное обучение деревьев принятия решений, что позволяет свести потери к минимуму. Он применяется для задач классификации и регрессии. С его помощью можно обрабатывать категориальные признаки, а также определять нелинейные зависимости и взаимодействия признаков. Этот метод не требует масштабирования признаков. Кроме того, его можно использовать для создания модели мультиклассовой классификации.

Примеры моделирования с использованием перекрестной проверки и перебора гиперпараметров показаны для задачи двоичной классификации. Более простые примеры (без перебора параметров) представлены в основной статье о задачах регрессии. Однако в приложении также представлена проверка с использованием эластичной сети для линейной регрессии и перекрестной проверки с перебором параметров для регрессии случайного леса. **Эластичная сеть** — это метод регуляризованной регрессии для подгонки моделей линейной регрессии, которые линейно объединяют метрики L1 и L2 в качестве штрафов методов [lasso](https://en.wikipedia.org/wiki/Lasso%20%28statistics%29) и [ridge](https://en.wikipedia.org/wiki/Tikhonov_regularization).   

<!-- -->

> [!NOTE]
> Несмотря на то что набор средств Spark MLlib предназначен для работы с большими наборами данных, для удобства мы будем использовать относительно небольшую выборку (приблизительно 30 МБ, 170 тыс. строк, примерно 0,1 % исходного набора данных о поездках в такси по Нью-Йорку). Описанные в этой статье задачи можно эффективно выполнить (примерно за 10 минут) в кластере HDInsight с 2 рабочими узлами. Тот же код (с незначительными изменениями) можно использовать для обработки больших наборов данных. Необходимо лишь внести соответствующие изменения для кэширования данных в памяти или изменения размера кластера.

<!-- -->

## <a name="setup-spark-clusters-and-notebooks"></a>Настройка: Кластеры и записные книжки Spark
Действия по настройке и код, указанные в этом пошаговом руководстве, применимы к использованию для HDInsight Spark 1.6. Но записные книжки Jupyter можно использовать для кластеров HDInsight Spark 1.6 и Spark 2.0. Описание записных книжек и ссылки на них вы можете найти в файле [Readme.md](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Readme.md) для репозитория GitHub с записными книжками. Более того, код здесь и в связанных записных книжках является универсальным и должен работать в любом кластере Spark. Действия по настройке кластера и управлению им могут немного отличаться от приведенных здесь, если вы не используете HDInsight Spark. Для удобства мы приводим ссылки на записные книжки Jupyter для Spark 1.6 и 2.0, которые можно выполнять в ядре pyspark на сервере Jupyter Notebook.

### <a name="spark-16-notebooks"></a>Записные книжки для Spark 1.6

[pySpark-machine-learning-data-science-spark-advanced-data-exploration-modeling.ipynb.](https://github.com/Azure-Samples/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark2.0/Spark2.0-pySpark3-machine-learning-data-science-spark-advanced-data-exploration-modeling.ipynb) Содержит разделы в записной книжке 1 и варианты моделей с использованием настройки гиперпараметров и перекрестной проверки.

### <a name="spark-20-notebooks"></a>Записные книжки для Spark 2.0

[Spark2.0-pySpark3-machine-learning-data-science-spark-advanced-data-exploration-modeling.ipynb.](https://github.com/Azure-Samples/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark2.0/Spark2.0-pySpark3-machine-learning-data-science-spark-advanced-data-exploration-modeling.ipynb) Содержит сведения о том, как выполнять просмотр данных, моделирование и оценку в кластерах Spark 2.0.

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="setup-storage-locations-libraries-and-the-preset-spark-context"></a>Настройка места хранения, библиотек и предустановленного контекста Spark
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
modelDir = "wasb:///user/remoteuser/NYCTaxi/Models/";

# PRINT START TIME
import datetime
datetime.datetime.now()
```

**OUTPUT**

datetime.datetime(2016, 4, 18, 17, 36, 27, 832799)

### <a name="import-libraries"></a>Импорт библиотек
Импортируйте необходимые библиотеки, используя следующий код:

```python
# LOAD PYSPARK LIBRARIES
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

Дополнительные сведения о ядрах для записных книжек Jupyter и предустановленных "волшебных командах", которые они предоставляют, см. в статье [Ядра, доступные для использования записными книжками Jupyter с кластерами Apache Spark в HDInsight на платформе Linux](../../hdinsight/spark/apache-spark-jupyter-notebook-kernels.md).

## <a name="data-ingestion-from-public-blob"></a>Прием данных из открытого большого двоичного объекта:
Чтобы начать анализ и обработку необходимых данных, требуется переместить их из источников, в которых они находятся, в среду исследования и моделирования данных. В нашем пошаговом руководстве — это среда Spark. Этот раздел содержит код, с помощью которого можно выполнить следующие задачи:

* прием выборки данных, которые нужно моделировать;
* считывание входного набора данных (TSV-файл);
* форматирование и очистка данных;
* создание и кэширование объектов (устойчивых распределенных наборов данных или фреймов данных) в памяти;
* регистрация данных в качестве временной таблицы в контексте SQL.

Ниже приведен код для приема данных.

```python
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

# CACHE & MATERIALIZE DATA-FRAME IN MEMORY. GOING THROUGH AND COUNTING NUMBER OF ROWS MATERIALIZES THE DATA-FRAME IN MEMORY
taxi_df_train_cleaned.cache()
taxi_df_train_cleaned.count()

# REGISTER DATA-FRAME AS A TEMP-TABLE IN SQL-CONTEXT
taxi_df_train_cleaned.registerTempTable("taxi_train")

# PRINT HOW MUCH TIME IT TOOK TO RUN THE CELL
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**OUTPUT**

Время выполнения кода выше: 276,62 с

## <a name="data-exploration--visualization"></a>Исследование и визуализация данных
После помещения данных в Spark необходимо исследовать и визуализировать их, чтобы получить более глубокое представление. В этом разделе мы исследуем данные о поездках в такси с помощью SQL-запросов и составим графики целевых переменных и потенциальных признаков для визуального контроля. В частности, мы составим график частоты поездок в такси, частоты оставления чаевых и зависимости суммы чаевых от тарифа и типа платежа.

### <a name="plot-a-histogram-of-passenger-count-frequencies-in-the-sample-of-taxi-trips"></a>Построение гистограммы количества пассажиров в выборке данных о поездках на такси
Этот код и последующие фрагменты используют волшебную команду SQL для запроса примера и локальную волшебную команду для графического представления данных.

* **SQL magic (`%%sql`)** Ядро HDInsight PySpark позволяет легко выполнять встроенные запросы HiveQL к sqlContext. Аргумент (-o VARIABLE_NAME) сохраняет выходные данные запроса SQL в формате Pandas DataFrame на сервере Jupyter. Это означает, что они доступны в локальном режиме.
* Магическая команда **`%%local`** используется для локального выполнения кода на сервере Jupyter, который представляет собой головной узел кластера HDInsight. Как правило, используется команда magic `%%local` после запуска команды magic `%%sql -o` для выполнения запроса. Параметр -o сохранит выходные данные запроса SQL локально. Затем команда magic `%%local` активирует следующий набор фрагментов кода для локального выполнения в выходных данных запроса SQL, который сохранен локально. После выполнения кода выходные данные визуализируются автоматически.

Этот запрос извлекает поездки по количеству пассажиров. 

```sql
# PLOT FREQUENCY OF PASSENGER COUNTS IN TAXI TRIPS

# SQL QUERY
%%sql -q -o sqlResults
SELECT passenger_count, COUNT(*) as trip_counts FROM taxi_train WHERE passenger_count > 0 and passenger_count < 7 GROUP BY passenger_count
```

Этот код создает локальный фрейм данных из выходных данных запроса и формирует графическое представление данных. Магическая команда `%%local` создает локальный кадр данных `sqlResults`, который можно использовать для формирования графического представления данных с помощью matplotlib. 

<!-- -->

> [!NOTE]
> В этом пошаговом руководстве волшебная команда PySpark используется несколько раз. Если объем данных большой, сделайте выборку, чтобы создать фрейм данных, который можно разместить в локальной памяти.

<!-- -->

```python
# RUN THE CODE LOCALLY ON THE JUPYTER SERVER
%%local

# USE THE JUPYTER AUTO-PLOTTING FEATURE TO CREATE INTERACTIVE FIGURES. 
# CLICK ON THE TYPE OF PLOT TO BE GENERATED (E.G. LINE, AREA, BAR ETC.)
sqlResults
```

Код для получения графического представления поездок по числу пассажиров

```python
# RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
%%local
import matplotlib.pyplot as plt
%matplotlib inline

# PLOT PASSENGER NUMBER VS TRIP COUNTS
x_labels = sqlResults['passenger_count'].values
fig = sqlResults[['trip_counts']].plot(kind='bar', facecolor='lightblue')
fig.set_xticklabels(x_labels)
fig.set_title('Counts of trips by passenger count')
fig.set_xlabel('Passenger count in trips')
fig.set_ylabel('Trip counts')
plt.show()
```

**OUTPUT**

![Частота поездок в зависимости от количества пассажиров](./media/spark-advanced-data-exploration-modeling/frequency-of-trips-by-passenger-count.png)

С помощью кнопок в меню **Тип** в записной книжке можно выбрать один из нескольких типов визуализации (таблица либо круговая, линейная, комбинированная или столбчатая диаграмма). Здесь показано представление столбчатой диаграммы.

### <a name="plot-a-histogram-of-tip-amounts-and-how-tip-amount-varies-by-passenger-count-and-fare-amounts"></a>Построение гистограммы суммы чаевых и ее зависимости от количества пассажиров и тарифа.
Используйте SQL-запрос для выборки данных.

```sql
# SQL SQUERY
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
# RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
%%local
%matplotlib inline

# TIP BY PAYMENT TYPE AND PASSENGER COUNT
ax1 = resultsPDDF[['tip_amount']].plot(kind='hist', bins=25, facecolor='lightblue')
ax1.set_title('Tip amount distribution')
ax1.set_xlabel('Tip Amount ($)')
ax1.set_ylabel('Counts')
plt.suptitle('')
plt.show()

# TIP BY PASSENGER COUNT
ax2 = resultsPDDF.boxplot(column=['tip_amount'], by=['passenger_count'])
ax2.set_title('Tip amount ($) by Passenger count')
ax2.set_xlabel('Passenger count')
ax2.set_ylabel('Tip Amount ($)')
plt.suptitle('')
plt.show()

# TIP AMOUNT BY FARE AMOUNT, POINTS ARE SCALED BY PASSENGER COUNT
ax = resultsPDDF.plot(kind='scatter', x= 'fare_amount', y = 'tip_amount', c='blue', alpha = 0.10, s=5*(resultsPDDF.passenger_count))
ax.set_title('Tip amount by Fare amount ($)')
ax.set_xlabel('Fare Amount')
ax.set_ylabel('Tip Amount')
plt.axis([-2, 120, -2, 30])
plt.show()
```

**ВЫХОДНЫЕ ДАННЫЕ:** 

![Распределение суммы чаевых](./media/spark-advanced-data-exploration-modeling/tip-amount-distribution.png)

![Сумма чаевых в зависимости от количества пассажиров](./media/spark-advanced-data-exploration-modeling/tip-amount-by-passenger-count.png)

![Сумма чаевых в зависимости от тарифа](./media/spark-advanced-data-exploration-modeling/tip-amount-by-fare-amount.png)

## <a name="feature-engineering-transformation-and-data-preparation-for-modeling"></a>Проектирование признаков, преобразование и подготовка данных к моделированию
В этом разделе приведен пример кода для выполнения процедур по подготовке данных, которые будут использоваться в моделях машинного обучения. Здесь также показано, как выполнять следующие задачи:

* Создание нового признака путем секционирования часов в ячейки трафика.
* Индексация и прямое кодирование категориальных признаков
* Создание объектов с помеченной вершиной в качестве ввода для функций машинного обучения
* Создание случайной вложенной выборки данных и ее разделение на наборы для обучения и тестирования
* масштабирование признаков;
* кэширование объектов в памяти.

### <a name="create-a-new-feature-by-partitioning-traffic-times-into-bins"></a>Создание нового признака путем секционирования часов трафика в ячейки.
В следующем коде показано, как создать признак методом секционирования часов в ячейки трафика, а затем кэшировать итоговый фрейм данных в памяти. Если устойчивые распределенные наборы данных и фреймы данных используются постоянно, на их кэширование необходимо меньше времени. В этом пошаговом руководстве процесс кэширования существующих данных выполняется в несколько этапов.

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
```

```python
# CACHE DATA-FRAME IN MEMORY & MATERIALIZE DF IN MEMORY
# THE .COUNT() GOES THROUGH THE ENTIRE DATA-FRAME,
# MATERIALIZES IT IN MEMORY, AND GIVES THE COUNT OF ROWS.
taxi_df_train_with_newFeatures.cache()
taxi_df_train_with_newFeatures.count()
```

**OUTPUT**

126 050.

### <a name="index-and-one-hot-encode-categorical-features"></a>Индексация и прямое кодирование категориальных признаков
В этом разделе показано, как индексировать и кодировать категориальные признаки в качестве ввода для функций моделирования. Перед использованием в функциях моделирования и прогнозирования MLlib категориальные входные данные сначала необходимо проиндексировать или закодировать. 

В зависимости от модели этот процесс происходит по-разному. Скажем, для моделей логистической и линейной регрессии требуется прямое кодирование, при котором, например, признак с тремя категориями можно разделить на три столбца признаков, в каждом из которых в зависимости от категории наблюдения содержатся значения 0 или 1. Для этого можно использовать функцию [OneHotEncoder](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OneHotEncoder.html#sklearn.preprocessing.OneHotEncoder) в MLlib. Этот кодировщик сопоставляет столбец с индексами меток со столбцом двоичных векторов как минимум с одним отдельным значением. Благодаря этой кодировке алгоритмы, для которых необходимы признаки с числовыми значениями (например, логистическая регрессия), можно применять к категориальным признакам.

Ниже приведен пример кода, с помощью которого можно проиндексировать и закодировать категориальные признаки.

```python
# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD PYSPARK LIBRARIES
from pyspark.ml.feature import OneHotEncoder, StringIndexer, VectorAssembler, OneHotEncoder, VectorIndexer

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

**OUTPUT**

Время выполнения кода выше: 3,14 с

### <a name="create-labeled-point-objects-for-input-into-ml-functions"></a>Создание объектов с помеченной вершиной в качестве ввода для функций машинного обучения
Этот раздел содержит код, который показывает, как индексировать категориальные текстовые данные в виде типа данных с меткой и как их кодировать. Это преобразование подготавливает текстовые данные для обучения и тестирования логистической регрессии MLlib и других моделей классификации. Объекты с помеченной вершиной отформатированы в устойчивые распределенные наборы данных, которые можно использовать в качестве входных для большинства алгоритмов машинного обучения в MLlib. Объект [с помеченной вершиной](https://spark.apache.org/docs/latest/mllib-data-types.html#labeled-point) — это разреженный или плотный локальный вектор, связанный с меткой или ответом.

Ниже приведен пример кода, с помощью которого можно проиндексировать и закодировать текстовые характеристики бинарной классификации.

```python
# FUNCTIONS FOR BINARY CLASSIFICATION

# LOAD LIBRARIES
from pyspark.mllib.regression import LabeledPoint
from numpy import array

# INDEXING CATEGORICAL TEXT FEATURES FOR INPUT INTO TREE-BASED MODELS
def parseRowIndexingBinary(line):
    features = np.array([line.paymentIndex, line.vendorIndex, line.rateIndex, line.pickup_hour, line.weekday,
                         line.passenger_count, line.trip_time_in_secs, line.trip_distance, line.fare_amount])
    labPt = LabeledPoint(line.tipped, features)
    return  labPt

# ONE-HOT ENCODING OF CATEGORICAL TEXT FEATURES FOR INPUT INTO LOGISTIC REGRESSION MODELS
def parseRowOneHotBinary(line):
    features = np.concatenate((np.array([line.pickup_hour, line.weekday, line.passenger_count,
                                        line.trip_time_in_secs, line.trip_distance, line.fare_amount]), 
                               line.vendorVec.toArray(), line.rateVec.toArray(), line.paymentVec.toArray()), axis=0)
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
С помощью кода в этом разделе создается случайная выборка данных (25 % всех данных). Хотя из-за небольшого размера используемого набора данных, этот шаг выполнять необязательно, мы все равно покажем, как создавать выборку данных. Это поможет вам в случае возникновения проблем. Если выборки большие, данный процесс может значительно сэкономить время обучения моделей. Затем мы разобьем выборку данных на наборы для обучения (75 %) и тестирования (25 %), которые будут использоваться для двоичной классификации и регрессии.

```python
# RECORD START TIME
timestart = datetime.datetime.now()

# SPECIFY SAMPLING AND SPLITTING FRACTIONS
from pyspark.sql.functions import rand

samplingFraction = 0.25;
trainingFraction = 0.75; testingFraction = (1-trainingFraction);
seed = 1234;
encodedFinalSampled = encodedFinal.sample(False, samplingFraction, seed=seed)

# SPLIT SAMPLED DATA-FRAME INTO TRAIN/TEST, WITH A RANDOM COLUMN ADDED FOR DOING CV (SHOWN LATER)
# INCLUDE RAND COLUMN FOR CREATING CROSS-VALIDATION FOLDS
dfTmpRand = encodedFinalSampled.select("*", rand(0).alias("rand"));
trainData, testData = dfTmpRand.randomSplit([trainingFraction, testingFraction], seed=seed);

# CACHE TRAIN AND TEST DATA
trainData.cache()
testData.cache()

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

**OUTPUT**

Время выполнения кода выше: 0,31 секунды

### <a name="feature-scaling"></a>масштабирование признаков;
Масштабирование признаков (или нормализация данных) гарантирует, что для признаков с широко распределенными значениями не задано высокое значение веса в целевой функции. В коде для масштабирования признаков используется функция [StandardScaler](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.mllib.regression.LinearRegressionWithSGD.html
), которая позволяет масштабировать признаки в зависимости от изменений единицы. MLlib предоставляет функцию StandardScaler для выполнения линейной регрессии с применением метода стохастического градиента (SGD). Алгоритм SGD широко используется для обучения других моделей машинного обучения (например, регуляризованной регрессии или метода опорных векторов).   

> [!TIP]
> Алгоритм LinearRegressionWithSGD чувствителен к масштабированию признаков.   
> 
> 

Ниже приведен код для масштабирования переменных, пригодный для использования с регуляризованным линейным алгоритмом с применением стохастического градиента.

```python
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

**OUTPUT**

Время выполнения кода выше: 11,67 с

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

**OUTPUT** 

Время выполнения кода выше: 0,13 секунды

## <a name="predict-whether-or-not-a-tip-is-paid-with-binary-classification-models"></a>Прогнозирование вероятности выплаты чаевых за поездку с помощью моделей бинарной классификации
В этом разделе показано, как с помощью трех моделей для выполнения задач двоичной классификации спрогнозировать, будут ли оставлены чаевые за поездку в такси. Мы используем следующие модели:

* Логистическая регрессия 
* случайный лес;
* градиентный бустинг деревьев.

Процесс создания модели состоит из следующих этапов: 

1. **Обучение модели** с использованием данных с одним набором параметров.
2. **Оценка модели** на основе набора тестовых данных с метриками.
3. **Сохранение модели** в большом двоичном объекте для последующего использования.

Мы покажем, как выполнить перекрестную проверку с перебором параметров двумя способами:

1. С использованием **универсального** пользовательского кода, который можно применять к любому алгоритму в MLlib и к любым наборам параметров в алгоритме. 
2. С использованием **функции конвейера pySpark CrossValidator**. CrossValidator имеет несколько ограничений для Spark 1.5.0. 
   
   * Модели конвейера невозможно сохранить для последующей обработки.
   * Не может использоваться для каждого параметра в модели.
   * Не может использоваться для каждого алгоритма MLlib.

### <a name="generic-cross-validation-and-hyperparameter-sweeping-used-with-the-logistic-regression-algorithm-for-binary-classification"></a>Универсальная перекрестная проверка и перебор гиперпараметров, используемые с алгоритмом логистической регрессии для двоичной классификации
Приведенный в этом разделе код предназначен для обучения, анализа и сохранения модели логистической регрессии с применением [LBFGS](https://en.wikipedia.org/wiki/Broyden%E2%80%93Fletcher%E2%80%93Goldfarb%E2%80%93Shanno_algorithm) , с помощью которой можно спрогнозировать, будут ли выплачены чаевые за поездку в такси по Нью-Йорку, на основе набора данных и тарифов. Модель обучена с использованием перекрестной проверки и перебора гиперпараметров, реализованных в пользовательском коде, который можно применить к любому из алгоритмов обучения в MLlib.   

<!-- -->

> [!NOTE]
> Выполнение такого пользовательского кода перекрестной проверки может занять несколько минут.

<!-- -->

**Обучение модели логистической регрессии с использованием перекрестной проверки и перебора параметров**

```python
# LOGISTIC REGRESSION CLASSIFICATION WITH CV AND HYPERPARAMETER SWEEPING

# GET ACCURACY FOR HYPERPARAMETERS BASED ON CROSS-VALIDATION IN TRAINING DATA-SET

# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD LIBRARIES
from pyspark.mllib.classification import LogisticRegressionWithLBFGS 
from pyspark.mllib.evaluation import BinaryClassificationMetrics

# CREATE PARAMETER GRID FOR LOGISTIC REGRESSION PARAMETER SWEEP
from sklearn.grid_search import ParameterGrid
grid = [{'regParam': [0.01, 0.1], 'iterations': [5, 10], 'regType': ["l1", "l2"], 'tolerance': [1e-3, 1e-4]}]
paramGrid = list(ParameterGrid(grid))
numModels = len(paramGrid)

# SET NUM FOLDS AND NUM PARAMETER SETS TO SWEEP ON
nFolds = 3;
h = 1.0 / nFolds;
metricSum = np.zeros(numModels);

# BEGIN CV WITH PARAMETER SWEEP
for i in range(nFolds):
    # Create training and x-validation sets
    validateLB = i * h
    validateUB = (i + 1) * h
    condition = (trainData["rand"] >= validateLB) & (trainData["rand"] < validateUB)
    validation = trainData.filter(condition)
    # Create LabeledPoints from data-frames
    if i > 0:
        trainCVLabPt.unpersist()
        validationLabPt.unpersist()
    trainCV = trainData.filter(~condition)
    trainCVLabPt = trainCV.map(parseRowOneHotBinary)
    trainCVLabPt.cache()
    validationLabPt = validation.map(parseRowOneHotBinary)
    validationLabPt.cache()
    # For parameter sets compute metrics from x-validation
    for j in range(numModels):
        regt = paramGrid[j]['regType']
        regp = paramGrid[j]['regParam']
        iters = paramGrid[j]['iterations']
        tol = paramGrid[j]['tolerance']
        # Train logistic regression model with hypermarameter set
        model = LogisticRegressionWithLBFGS.train(trainCVLabPt, regType=regt, iterations=iters,  
                                                  regParam=regp, tolerance = tol, intercept=True)
        predictionAndLabels = validationLabPt.map(lambda lp: (float(model.predict(lp.features)), lp.label))
        # Use ROC-AUC as accuracy metrics
        validMetrics = BinaryClassificationMetrics(predictionAndLabels)
        metric = validMetrics.areaUnderROC
        metricSum[j] += metric

avgAcc = metricSum / nFolds;
bestParam = paramGrid[np.argmax(avgAcc)];

# UNPERSIST OBJECTS
trainCVLabPt.unpersist()
validationLabPt.unpersist()

# TRAIN ON FULL TRAIING SET USING BEST PARAMETERS FROM CV/PARAMETER SWEEP
logitBest = LogisticRegressionWithLBFGS.train(oneHotTRAINbinary, regType=bestParam['regType'], 
                                              iterations=bestParam['iterations'], 
                                              regParam=bestParam['regParam'], tolerance = bestParam['tolerance'], 
                                              intercept=True)


# PRINT COEFFICIENTS AND INTERCEPT OF THE MODEL
# NOTE: There are 20 coefficient terms for the 10 features, 
#       and the different categories for features: vendorVec (2), rateVec, paymentVec (6), TrafficTimeBinsVec (4)
print("Coefficients: " + str(logitBest.weights))
print("Intercept: " + str(logitBest.intercept))

# PRINT ELAPSED TIME    
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**OUTPUT**

Coefficients: [0.0082065285375, -0.0223675576104, -0.0183812028036, -3.48124578069e-05, -0.00247646947233, -0.00165897881503, 0.0675394837328, -0.111823113101, -0.324609912762, -0.204549780032, -1.36499216354, 0.591088507921, -0.664263411392, -1.00439726852, 3.46567827545, -3.51025855172, -0.0471341112232, -0.043521833294, 0.000243375810385, 0.054518719222]

Intercept: -0.0111216486893

Время выполнения кода выше: 14,43 секунды

**Оценка модели бинарной классификации со стандартными метриками**

Код в этом разделе показывает, как оценить модель логистической регрессии по тестовому набору данных, включая построение кривой ROC.

```python
# RECORD START TIME
timestart = datetime.datetime.now()

#IMPORT LIBRARIES
from sklearn.metrics import roc_curve,auc
from pyspark.mllib.evaluation import BinaryClassificationMetrics
from pyspark.mllib.evaluation import MulticlassMetrics

# PREDICT ON TEST DATA WITH BEST/FINAL MODEL
predictionAndLabels = oneHotTESTbinary.map(lambda lp: (float(logitBest.predict(lp.features)), lp.label))

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

# OUTPUT PROBABILITIES AND REGISTER TEMP TABLE
logitBest.clearThreshold(); # This clears threshold for classification (0.5) and outputs probabilities
predictionAndLabelsDF = predictionAndLabels.toDF()
predictionAndLabelsDF.registerTempTable("tmp_results");

# PRINT ELAPSED TIME    
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**OUTPUT**

Area under PR = 0.985336538462

Area under ROC = 0.983383274312

Summary Stats

Precision = 0.984174341679

Recall = 0.984174341679

F1 Score = 0.984174341679

Время выполнения кода выше: 2,67 с

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

#PREDICTIONS
predictions_pddf = sqlResults.rename(columns={'_1': 'probability', '_2': 'label'})
prob = predictions_pddf["probability"] 
fpr, tpr, thresholds = roc_curve(predictions_pddf['label'], prob, pos_label=1);
roc_auc = auc(fpr, tpr)

# PLOT ROC CURVES
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

**OUTPUT**

![Кривая ROC логистической регрессии для универсального подхода](./media/spark-advanced-data-exploration-modeling/logistic-regression-roc-curve.png)

**Сохранение модели в большом двоичном объекте для последующего использования**

Код в этом разделе показывает, как сохранить модель логистической регрессии для использования.

```python
# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD PYSPARK LIBRARIES
from pyspark.mllib.classification import LogisticRegressionModel

# PERSIST MODEL
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
logisticregressionfilename = "LogisticRegressionWithLBFGS_" + datestamp;
dirfilename = modelDir + logisticregressionfilename;

logitBest.save(sc, dirfilename);

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds";
```

**OUTPUT**

Время выполнения кода выше: 34,57 с

### <a name="use-mllibs-crossvalidator-pipeline-function-with-logistic-regression-elastic-regression-model"></a>Использование функции конвейера MLlib CrossValidator с моделью LogisticRegression (эластичная регрессия)
Приведенный в этом разделе код предназначен для обучения, анализа и сохранения модели логистической регрессии с применением [LBFGS](https://en.wikipedia.org/wiki/Broyden%E2%80%93Fletcher%E2%80%93Goldfarb%E2%80%93Shanno_algorithm) , с помощью которой можно спрогнозировать, будут ли выплачены чаевые за поездку в такси по Нью-Йорку, на основе набора данных и тарифов. Модель обучена с использованием перекрестной проверки и перебора гиперпараметров, реализованных с помощью функции конвейера MLlib CrossValidator для перекрестной проверки с перебором гиперпараметров.   

<!-- -->

> [!NOTE]
> Выполнение такого кода перекрестной проверки MLlib может занять несколько минут.

<!-- -->

```python
# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD PYSPARK LIBRARIES
from pyspark.ml.classification import LogisticRegression
from pyspark.ml import Pipeline
from pyspark.ml.evaluation import BinaryClassificationEvaluator
from pyspark.ml.tuning import CrossValidator, ParamGridBuilder
from sklearn.metrics import roc_curve,auc

# DEFINE ALGORITHM / MODEL
lr = LogisticRegression()

# DEFINE GRID PARAMETERS
paramGrid = ParamGridBuilder().addGrid(lr.regParam, (0.01, 0.1))\
                              .addGrid(lr.maxIter, (5, 10))\
                              .addGrid(lr.tol, (1e-4, 1e-5))\
                              .addGrid(lr.elasticNetParam, (0.25,0.75))\
                              .build()

# DEFINE CV WITH PARAMETER SWEEP
cv = CrossValidator(estimator= lr,
                    estimatorParamMaps=paramGrid,
                    evaluator=BinaryClassificationEvaluator(),
                    numFolds=3)

# CONVERT TO DATA-FRAME: THIS DOES NOT RUN ON RDDs
trainDataFrame = sqlContext.createDataFrame(oneHotTRAINbinary, ["features", "label"])

# TRAIN WITH CROSS-VALIDATION
cv_model = cv.fit(trainDataFrame)


## PREDICT AND EVALUATE ON TEST DATA-SET

# USE TEST DATASET FOR PREDICTION
testDataFrame = sqlContext.createDataFrame(oneHotTESTbinary, ["features", "label"])
test_predictions = cv_model.transform(testDataFrame)

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds";
``` 

**OUTPUT**

Время выполнения кода выше: 107,98 с

**Графическое представление кривой ROC.**

*predictionAndLabelsDF* регистрируется как таблица (*tmp_results*) в предыдущей ячейке. *tmp_results* может использоваться для выполнения запросов и вывода результатов во фрейм данных sqlResults для построения графика. Ниже приведен код:

```python
# QUERY RESULTS
%%sql -q -o sqlResults
SELECT label, prediction, probability from tmp_results
```

Ниже приведен код для построения кривых ROC.

```python
# RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES 
%%local
from sklearn.metrics import roc_curve,auc

# ROC CURVE
prob = [x["values"][1] for x in sqlResults["probability"]]
fpr, tpr, thresholds = roc_curve(sqlResults['label'], prob, pos_label=1);
roc_auc = auc(fpr, tpr)

#PLOT
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

**OUTPUT**

![Кривая ROC логистической регрессии с использованием MLlib CrossValidator](./media/spark-advanced-data-exploration-modeling/mllib-crossvalidator-roc-curve.png)

### <a name="random-forest-classification"></a>Классификация случайного леса
Приведенный в этом разделе код предназначен для обучения, анализа и сохранения регрессии случайного леса, с помощью которой можно спрогнозировать, будут ли выплачены чаевые за поездку в такси по Нью-Йорку, на основе набора данных и тарифов.

```python
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
## UN-COMMENT IF YOU WANT TO PRING TREES
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

**OUTPUT**

Area under ROC = 0.985336538462

Время выполнения кода выше: 26,72 с

### <a name="gradient-boosting-trees-classification"></a>Классификация с применением модели градиентного бустинга деревьев
Приведенный в этом разделе код предназначен для обучения, анализа и сохранения модели градиентного бустинга деревьев, с помощью которой можно спрогнозировать, будут ли выплачены чаевые за поездку в такси по Нью-Йорку, на основе набора данных и тарифов.

```python
# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD PYSPARK LIBRARIES
from pyspark.mllib.tree import GradientBoostedTrees, GradientBoostedTreesModel

# SPECIFY NUMBER OF CATEGORIES FOR CATEGORICAL FEATURES. FEATURE #0 HAS 2 CATEGORIES, FEATURE #2 HAS 2 CATEGORIES, AND SO ON
categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}

gbtModel = GradientBoostedTrees.trainClassifier(indexedTRAINbinary, categoricalFeaturesInfo=categoricalFeaturesInfo,
                                                numIterations=10)
## UNCOMMENT IF YOU WANT TO PRINT TREE DETAILS
#print('Learned classification GBT model:')
#print(bgtModel.toDebugString())

# PREDICT ON TEST DATA AND EVALUATE
predictions = gbtModel.predict(indexedTESTbinary.map(lambda x: x.features))
predictionAndLabels = indexedTESTbinary.map(lambda lp: lp.label).zip(predictions)

# Area under ROC curve
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

**OUTPUT**

Area under ROC = 0.985336538462

Время выполнения кода выше: 28,13 с

## <a name="predict-tip-amount-with-regression-models-not-using-cv"></a>Прогнозирование суммы чаевых с помощью моделей регрессии (без использования перекрестной проверки)
В этом разделе показано, как с помощью трех моделей регрессии спрогнозировать сумму чаевых в зависимости от других признаков. Мы используем следующие модели:

* регуляризованная линейная регрессия;
* случайный лес;
* градиентный бустинг деревьев.

Эти модели описаны в начале статьи. Процесс создания модели состоит из следующих этапов: 

1. **Обучение модели** с использованием данных с одним набором параметров.
2. **Оценка модели** на основе набора тестовых данных с метриками.
3. **Сохранение модели** в большом двоичном объекте для последующего использования.   

<!-- -->

> [!NOTE] 
> Перекрестная проверка не используется с тремя моделями регрессии, указанными в этом разделе. Это было подробно описано в отношении моделей логистической регрессии. Пример использования перекрестной проверки с эластичной сетью для линейной регрессии показан в приложении к этой статье.

<!-- -->

<!-- -->

> [!NOTE] 
> На практике в моделях LinearRegressionWithSGD часто возникают проблемы с конвергенцией. Чтобы получить допустимую модель, необходимо осторожно изменить или оптимизировать параметры. Проблему конвергенции можно решить, выполнив масштабирование переменных. Регрессию эластичной сети, показанную в приложении к этой статье, можно также использовать вместо LinearRegressionWithSGD.

<!-- -->

### <a name="linear-regression-with-sgd"></a>Линейная регрессия с применением SGD
С помощью кода, приведенного в этом разделе, на основе масштабируемых признаков можно обучить модель линейной регрессии, в которой для оптимизации используется метод стохастического градиента, а также оценить, проанализировать и сохранить ее в хранилище BLOB-объектов Azure.

> [!TIP]
> На практике в моделях LinearRegressionWithSGD часто возникают проблемы с конвергенцией. Чтобы получить допустимую модель, необходимо осторожно изменить или оптимизировать параметры. Проблему конвергенции можно решить, выполнив масштабирование переменных.
> 
> 

```python
# LINEAR REGRESSION WITH SGD 

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

print("RMSE = %s" % testMetrics.rootMeanSquaredError)
print("R-sqr = %s" % testMetrics.r2)

# SAVE MODEL IN BLOB
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
linearregressionfilename = "LinearRegressionWithSGD_" + datestamp;
dirfilename = modelDir + linearregressionfilename;

linearModel.save(sc, dirfilename)

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**OUTPUT**

Coefficients: [0.0141707753435, -0.0252930927087, -0.0231442517137, 0.247070902996, 0.312544147152, 0.360296120645, 0.0122079566092, -0.00456498588241, -0.0898228505177, 0.0714046248793, 0.102171263868, 0.100022455632, -0.00289545676449, -0.00791124681938, 0.54396316518, -0.536293513569, 0.0119076553369, -0.0173039244582, 0.0119632796147, 0.00146764882502]

Intercept: 0.854507624459

RMSE = 1.23485131376

R-sqr = 0.597963951127

Время выполнения кода выше: 38,62 с

### <a name="random-forest-regression"></a>Регрессия с использованием модели случайного леса
Приведенный в этом разделе код предназначен для обучения, анализа и сохранения модели случайного леса, с помощью которой можно спрогнозировать суму чаевых за поездку в такси по Нью-Йорку.   

<!-- -->

> [!NOTE]
> Перекрестная проверка с перебором параметров с использованием пользовательского кода показана в приложении.

<!-- -->

```python
#PREDICT TIP AMOUNTS USING RANDOM FOREST

# RECORD START TIME
timestart= datetime.datetime.now()

# LOAD PYSPARK LIBRARIES
from pyspark.mllib.tree import RandomForest, RandomForestModel
from pyspark.mllib.util import MLUtils
from pyspark.mllib.evaluation import RegressionMetrics


# TRAIN MODEL
categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}
rfModel = RandomForest.trainRegressor(indexedTRAINreg, categoricalFeaturesInfo=categoricalFeaturesInfo,
                                    numTrees=25, featureSubsetStrategy="auto",
                                    impurity='variance', maxDepth=10, maxBins=32)
# UN-COMMENT IF YOU WANT TO PRING TREES
#print('Learned classification forest model:')
#print(rfModel.toDebugString())

# PREDICT AND EVALUATE ON TEST DATA-SET
predictions = rfModel.predict(indexedTESTreg.map(lambda x: x.features))
predictionAndLabels = oneHotTESTreg.map(lambda lp: lp.label).zip(predictions)

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

**OUTPUT**

RMSE = 0.931981967875

R-sqr = 0.733445485802

Время выполнения кода выше: 25,98 с

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

# TRAIN MODEL
categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}
gbtModel = GradientBoostedTrees.trainRegressor(indexedTRAINreg, categoricalFeaturesInfo=categoricalFeaturesInfo, 
                                                numIterations=10, maxBins=32, maxDepth = 4, learningRate=0.1)

# EVALUATE A TEST DATA-SET
predictions = gbtModel.predict(indexedTESTreg.map(lambda x: x.features))
predictionAndLabels = indexedTESTreg.map(lambda lp: lp.label).zip(predictions)

testMetrics = RegressionMetrics(predictionAndLabels)
print("RMSE = %s" % testMetrics.rootMeanSquaredError)
print("R-sqr = %s" % testMetrics.r2)

# PLOT SCATTER-PLOT BETWEEN ACTUAL AND PREDICTED TIP VALUES
test_predictions= sqlContext.createDataFrame(predictionAndLabels)
test_predictions_pddf = test_predictions.toPandas()

# SAVE MODEL IN BLOB
datestamp = unicode(datetime.datetime.now()).replace(' ','').replace(':','_');
btregressionfilename = "GradientBoostingTreeRegression_" + datestamp;
dirfilename = modelDir + btregressionfilename;
gbtModel.save(sc, dirfilename)

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**OUTPUT**

RMSE = 0.928172197114

R-sqr = 0.732680354389

Время выполнения кода выше: 20,9 с

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
import numpy as np

# PLOT
ax = sqlResults.plot(kind='scatter', figsize = (6,6), x='_1', y='_2', color='blue', alpha = 0.25, label='Actual vs. predicted');
fit = np.polyfit(sqlResults['_1'], sqlResults['_2'], deg=1)
ax.set_title('Actual vs. Predicted Tip Amounts ($)')
ax.set_xlabel("Actual")
ax.set_ylabel("Predicted")
ax.plot(sqlResults['_1'], fit[0] * sqlResults['_1'] + fit[1], color='magenta')
plt.axis([-1, 15, -1, 15])
plt.show(ax)
```

![Actual-vs-predicted-tip-amounts](./media/spark-advanced-data-exploration-modeling/actual-vs-predicted-tips.png)

## <a name="appendix-additional-regression-tasks-using-cross-validation-with-parameter-sweeps"></a>Приложение. Дополнительные задачи регрессии с использованием перекрестной проверки с перебором параметров
В этом приложении содержится код, показывающий, как выполнить перекрестную проверку с использованием эластичной сети для линейной регрессии, а также как выполнить перекрестную проверку с перебором параметров с использованием пользовательского кода для регрессии случайного леса.

### <a name="cross-validation-using-elastic-net-for-linear-regression"></a>Перекрестная проверка с использованием эластичной сети для линейной регрессии
Код в этом разделе показывает, как выполнить перекрестную проверку с использованием эластичной сети для линейной регрессии и как оценить модель по тестовым данным.

```python
###  CV USING ELASTIC NET FOR LINEAR REGRESSION

# RECORD START TIME
timestart = datetime.datetime.now()

# LOAD PYSPARK LIBRARIES
from pyspark.ml.regression import LinearRegression
from pyspark.ml import Pipeline
from pyspark.ml.evaluation import RegressionEvaluator
from pyspark.ml.tuning import CrossValidator, ParamGridBuilder

# DEFINE ALGORITHM/MODEL
lr = LinearRegression()

# DEFINE GRID PARAMETERS
paramGrid = ParamGridBuilder().addGrid(lr.regParam, (0.01, 0.1))\
                              .addGrid(lr.maxIter, (5, 10))\
                              .addGrid(lr.tol, (1e-4, 1e-5))\
                              .addGrid(lr.elasticNetParam, (0.25,0.75))\
                              .build() 

# DEFINE PIPELINE 
# SIMPLY THE MODEL HERE, WITHOUT TRANSFORMATIONS
pipeline = Pipeline(stages=[lr])

# DEFINE CV WITH PARAMETER SWEEP
cv = CrossValidator(estimator= lr,
                    estimatorParamMaps=paramGrid,
                    evaluator=RegressionEvaluator(),
                    numFolds=3)

# CONVERT TO DATA FRAME, AS CROSSVALIDATOR WON'T RUN ON RDDS
trainDataFrame = sqlContext.createDataFrame(oneHotTRAINreg, ["features", "label"])

# TRAIN WITH CROSS-VALIDATION
cv_model = cv.fit(trainDataFrame)


# EVALUATE MODEL ON TEST SET
testDataFrame = sqlContext.createDataFrame(oneHotTESTreg, ["features", "label"])

# MAKE PREDICTIONS ON TEST DOCUMENTS
# cvModel uses the best model found (lrModel).
predictionAndLabels = cv_model.transform(testDataFrame)

# CONVERT TO DF AND SAVE REGISTER DF AS TABLE
predictionAndLabels.registerTempTable("tmp_results");

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**OUTPUT**

Время выполнения кода выше: 161,21 с

**Оценка с помощью метрики R SQR**

*tmp_results* регистрируется как таблица Hive в предыдущей ячейке. Результаты из таблицы передаются во фрейм данных *sqlResults* для формирования графического представления. Ниже приведен код:

```python
# SELECT RESULTS
%%sql -q -o sqlResults
SELECT label,prediction from tmp_results
```

Ниже приведен код для вычисления R-sqr.

```python
# RUN THE CODE LOCALLY ON THE JUPYTER SERVER AND IMPORT LIBRARIES
%%local
from scipy import stats

#R-SQR TEST METRIC
corstats = stats.linregress(sqlResults['label'],sqlResults['prediction'])
r2 = (corstats[2]*corstats[2])
print("R-sqr = %s" % r2)
```

**OUTPUT**

R-sqr = 0.619184907088

### <a name="cross-validation-with-parameter-sweep-using-custom-code-for-random-forest-regression"></a>Перекрестная проверка с перебором параметров с использованием пользовательского кода для регрессии случайного леса
Код в этом разделе показывает, как выполнить перекрестную проверку с перебором параметров с использованием пользовательского кода для регрессии случайного леса и как оценить модель по тестовым данным.

```python
# RECORD START TIME
timestart= datetime.datetime.now()

# LOAD PYSPARK LIBRARIES
# GET ACCURARY FOR HYPERPARAMETERS BASED ON CROSS-VALIDATION IN TRAINING DATA-SET
from pyspark.mllib.tree import RandomForest, RandomForestModel
from pyspark.mllib.util import MLUtils
from pyspark.mllib.evaluation import RegressionMetrics
from sklearn.grid_search import ParameterGrid

## CREATE PARAMETER GRID
grid = [{'maxDepth': [5,10], 'numTrees': [25,50]}]
paramGrid = list(ParameterGrid(grid))

## SPECIFY LEVELS OF CATEGORICAL VARIBLES
categoricalFeaturesInfo={0:2, 1:2, 2:6, 3:4}

# SPECIFY NUMFOLDS AND ARRAY TO HOLD METRICS
nFolds = 3;
numModels = len(paramGrid)
h = 1.0 / nFolds;
metricSum = np.zeros(numModels);

for i in range(nFolds):
    # Create training and x-validation sets
    validateLB = i * h
    validateUB = (i + 1) * h
    condition = (trainData["rand"] >= validateLB) & (trainData["rand"] < validateUB)
    validation = trainData.filter(condition)
    # Create labeled points from data-frames
    if i > 0:
        trainCVLabPt.unpersist()
        validationLabPt.unpersist()
    trainCV = trainData.filter(~condition)
    trainCVLabPt = trainCV.map(parseRowIndexingRegression)
    trainCVLabPt.cache()
    validationLabPt = validation.map(parseRowIndexingRegression)
    validationLabPt.cache()
    # For parameter sets compute metrics from x-validation
    for j in range(numModels):
        maxD = paramGrid[j]['maxDepth']
        numT = paramGrid[j]['numTrees']
        # Train logistic regression model with hypermarameter set
        rfModel = RandomForest.trainRegressor(trainCVLabPt, categoricalFeaturesInfo=categoricalFeaturesInfo,
                                    numTrees=numT, featureSubsetStrategy="auto",
                                    impurity='variance', maxDepth=maxD, maxBins=32)
        predictions = rfModel.predict(validationLabPt.map(lambda x: x.features))
        predictionAndLabels = validationLabPt.map(lambda lp: lp.label).zip(predictions)
        # Use ROC-AUC as accuracy metrics
        validMetrics = RegressionMetrics(predictionAndLabels)
        metric = validMetrics.rootMeanSquaredError
        metricSum[j] += metric

avgAcc = metricSum/nFolds;
bestParam = paramGrid[np.argmin(avgAcc)];

# UNPERSIST OBJECTS
trainCVLabPt.unpersist()
validationLabPt.unpersist()

## TRAIN FINAL MODL WIHT BEST PARAMETERS
rfModel = RandomForest.trainRegressor(indexedTRAINreg, categoricalFeaturesInfo=categoricalFeaturesInfo,
                                    numTrees=bestParam['numTrees'], featureSubsetStrategy="auto",
                                    impurity='variance', maxDepth=bestParam['maxDepth'], maxBins=32)

# EVALUATE MODEL ON TEST DATA
predictions = rfModel.predict(indexedTESTreg.map(lambda x: x.features))
predictionAndLabels = indexedTESTreg.map(lambda lp: lp.label).zip(predictions)

#PRINT TEST METRICS
testMetrics = RegressionMetrics(predictionAndLabels)
print("RMSE = %s" % testMetrics.rootMeanSquaredError)
print("R-sqr = %s" % testMetrics.r2)

# PRINT ELAPSED TIME
timeend = datetime.datetime.now()
timedelta = round((timeend-timestart).total_seconds(), 2) 
print "Time taken to execute above cell: " + str(timedelta) + " seconds"; 
```

**OUTPUT**

RMSE = 0.906972198262

R-sqr = 0.740751197012

Время выполнения кода выше: 69,17 с

### <a name="clean-up-objects-from-memory-and-print-model-locations"></a>Очистка объектов из памяти и вывод расположений моделей
Удалите объекты, кэшированные в памяти, с помощью метода `unpersist()` .

```python
# UNPERSIST OBJECTS CACHED IN MEMORY

# REMOVE ORIGINAL DFs
taxi_df_train_cleaned.unpersist()
taxi_df_train_with_newFeatures.unpersist()
trainData.unpersist()
trainData.unpersist()

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

**OUTPUT**

PythonRDD[122] at RDD at PythonRDD.scala: 43

** Путь к файлам модели для использования в записной книжке данных об использовании. ** Чтобы использовать и оценивать независимые наборы данных, необходимо скопировать и вставить имена файлов в записную книжку данных об использовании.

```python
# PRINT MODEL FILE LOCATIONS FOR CONSUMPTION
print "logisticRegFileLoc = modelDir + \"" + logisticregressionfilename + "\"";
print "linearRegFileLoc = modelDir + \"" + linearregressionfilename + "\"";
print "randomForestClassificationFileLoc = modelDir + \"" + rfclassificationfilename + "\"";
print "randomForestRegFileLoc = modelDir + \"" + rfregressionfilename + "\"";
print "BoostedTreeClassificationFileLoc = modelDir + \"" + btclassificationfilename + "\"";
print "BoostedTreeRegressionFileLoc = modelDir + \"" + btregressionfilename + "\"";
```

**OUTPUT**

logisticRegFileLoc = modelDir + "LogisticRegressionWithLBFGS_2016-05-0316_47_30.096528"

linearRegFileLoc = modelDir + "LinearRegressionWithSGD_2016-05-0316_51_28.433670"

randomForestClassificationFileLoc = modelDir + "RandomForestClassification_2016-05-0316_50_17.454440"

randomForestRegFileLoc = modelDir + "RandomForestRegression_2016-05-0316_51_57.331730"

BoostedTreeClassificationFileLoc = modelDir + "GradientBoostingTreeClassification_2016-05-0316_50_40.138809"

BoostedTreeRegressionFileLoc = modelDir + "GradientBoostingTreeRegression_2016-05-0316_52_18.827237"

## <a name="whats-next"></a>Дальнейшие действия
После создания моделей регрессии и классификации с помощью Spark MlLib необходимо ознакомиться с процессом оценки и анализа этих моделей.

**Использование модели.** См. дополнительные сведения об [оценке и использованию моделей классификации и регрессии, созданных с помощью Spark](spark-model-consumption.md).
