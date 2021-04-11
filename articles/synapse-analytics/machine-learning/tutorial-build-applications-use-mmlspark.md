---
title: Руководство. Создание приложений машинного обучения с помощью Библиотеки машинного обучения Microsoft для Apache Spark (предварительная версия)
description: Узнайте, как с помощью Библиотеки машинного обучения Microsoft для Apache Spark создавать приложения машинного обучения в Azure Synapse Analytics.
services: synapse-analytics
ms.service: synapse-analytics
ms.subservice: machine-learning
ms.topic: tutorial
ms.reviewer: ''
ms.date: 03/08/2021
author: ruixinxu
ms.author: ruxu
ms.openlocfilehash: 928e2ef8b373626a91a291b1798f3ebb7ef290e8
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105608839"
---
# <a name="tutorial-build-machine-learning-applications-using-microsoft-machine-learning-for-apache-spark-preview"></a>Руководство. Создание приложений машинного обучения с помощью Библиотеки машинного обучения Microsoft для Apache Spark (предварительная версия)

В этой статье показано, как создавать приложения машинного обучения с помощью Библиотеки машинного обучения Microsoft для Apache Spark ([MMLSpark](https://github.com/Azure/mmlspark)). MMLSpark расширяет решение распределенного машинного обучения Apache Spark путем добавления различных средств глубокого обучения, а также обработки и анализа данных, таких как [Azure Cognitive Services](../../cognitive-services/big-data/cognitive-services-for-big-data.md), [OpenCV](https://opencv.org/), [LightGBM](https://github.com/Microsoft/LightGBM) и др.  MMLSpark позволяет создавать мощные и масштабируемые прогнозные и аналитические модели на основе разных источников данных Spark.
Synapse Spark предоставляет встроенные библиотеки MMLSpark, в том числе:

- [Vowpal Wabbit](https://github.com/Azure/mmlspark/blob/master/docs/vw.md) — службы библиотек для машинного обучения, которые позволяют использовать возможности Анализа текста, например анализ тональности в твитах.
- [Cognitive Services в Spark](https://github.com/Azure/mmlspark/blob/master/docs/cogsvc.md) — объединение функций Azure Cognitive Services в конвейерах SparkML, позволяющее получить проект решения для таких служб моделирования данных, как обнаружение аномалий.
- [LightBGM](https://github.com/Azure/mmlspark/blob/master/docs/lightgbm.md) — платформа градиентного усиления, которая использует три базовых алгоритма обучения. Она предназначена для распределения и повышения эффективности.
- Условные модели KNN — масштабируемые модели KNN с условными запросами.
- [HTTP в Spark](https://github.com/Azure/mmlspark/blob/master/docs/http.md) — оркестрация распределенных микрослужб при интеграции Spark и специальных возможностей на основе протокола HTTP.

В этом руководстве рассматриваются примеры использования Azure Cognitive Services в MMLSpark для следующих решений: 

- Анализ текста — получение сведений о тональности (или настроении) набора предложений;
- Компьютерное зрение — получение тегов (описание с использованием одного слова), связанных с набором изображений;
- Поиск изображений Bing — поиск в Интернете изображений, соответствующих запросу на естественном языке;
- Детектор аномалий — обнаружение аномалий в данных временных рядов.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись, прежде чем начинать работу](https://azure.microsoft.com/free/).

## <a name="prerequisites"></a>Предварительные требования 

- [Рабочая область Azure Synapse Analytics](../get-started-create-workspace.md) с учетной записью хранения Azure Data Lake Storage 2-го поколения, настроенной в качестве хранилища по умолчанию. При работе с файловой системой Data Lake Storage 2-го поколения вам нужно иметь права *участника для получения данных Хранилища BLOB-объектов*.
- Пул Spark в рабочей области Azure Synapse Analytics. Дополнительные сведения см. в статье [Создание пула Spark в Azure Synapse](../quickstart-create-sql-pool-studio.md).
- Предварительные действия описаны в руководстве по [настройке Cognitive Services в Azure Synapse](./tutorial-configure-cognitive-services-synapse.md).


## <a name="get-started"></a>Начало работы
Чтобы начать работу, импортируйте mmlspark и настройте ключи служб. 

```python
import mmlspark
mmlspark.__spark_package_version__ # current version: 1.0.0-rc3-6-a862d6b1-SNAPSHOT

from mmlspark.cognitive import *
from notebookutils import mssparkutils

# A general Cognitive Services key for Text Analytics and Computer Vision (or use separate keys that belong to each service)
service_key =  "ADD_YOUR_SUBSCRIPION_KEY" 
# A Bing Search v7 subscription key
bing_search_key = "ADD_YOUR_SUBSCRIPION_KEY" 
# An Anomaly Dectector subscription key
anomaly_key =  "ADD_YOUR_SUBSCRIPION_KEY" 
# Your linked key vault for Synapse workspace
key_vault = "YOUR_KEY_VAULT_NAME"


cognitive_service_key = mssparkutils.credentials.getSecret(key_vault, service_key)
bingsearch_service_key = mssparkutils.credentials.getSecret(key_vault, bing_search_key)
anomalydetector_key = mssparkutils.credentials.getSecret(key_vault, anomaly_key)

```


## <a name="text-analytics-sample"></a>Пример для Анализа текста

Служба [Анализ текста](../../cognitive-services/text-analytics/index.yml) предоставляет несколько алгоритмов для извлечения интеллектуальной аналитики из текста. Например, можно определить тональность заданного входного текста. Служба вернет оценку между 0,0 и 1,0: низкий показатель указывает на отрицательную тональность, а высокий — на положительную. В этом примере используются три простых предложения. Для каждого из них возвращается тональность.

```python
from pyspark.sql.functions import col

# Create a dataframe that's tied to it's column names
df_sentences = spark.createDataFrame([
  ("I am so happy today, its sunny!", "en-US"), 
  ("this is a dog", "en-US"), 
  ("I am frustrated by this rush hour traffic!", "en-US") 
], ["text", "language"])

# Run the Text Analytics service with options
sentiment = (TextSentiment()
    .setTextCol("text")
    .setLocation("eastasia")
    .setSubscriptionKey(cognitive_service_key)
    .setOutputCol("sentiment")
    .setErrorCol("error")
    .setLanguageCol("language"))

# Show the results of your text query in a table format

display(sentiment.transform(df_sentences).select("text", col("sentiment")[0].getItem("sentiment").alias("sentiment")))
```

### <a name="expected-results"></a>Ожидаемые результаты

|text | Тональность|
|--|--|
| I am frustrated by this rush hour traffic! (Меня раздражает дорожное движение в час пик!) | негативная тональность |
| this is a dog (это собака) | нейтральная тональность |
| I am so happy today, its sunny! | позитивная тональность |

## <a name="computer-vision-sample"></a>Пример для Компьютерного зрения
Служба [Компьютерное зрение](../../cognitive-services/computer-vision/index.yml) анализирует изображения для выявления таких структур, как лица, объекты и описания на естественном языке. В этом примере мы добавляем теги к следующему изображению. Теги — это описания вещей, выраженные одним словом, например распознаваемых объектов, людей, пейзажей и действий.


![Изображение](https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/objects.jpg)

```python
# Create a dataframe with the image URL
df_images = spark.createDataFrame([
        ("https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/objects.jpg", )
    ], ["image", ])

# Run the Computer Vision service. Analyze Image extracts information from/about the images.
analysis = (AnalyzeImage()
    .setLocation("eastasia")
    .setSubscriptionKey(cognitive_service_key)
    .setVisualFeatures(["Categories","Color","Description","Faces","Objects","Tags"])
    .setOutputCol("analysis_results")
    .setImageUrlCol("image")
    .setErrorCol("error"))

# Show the results of what you wanted to pull out of the images.
display(analysis.transform(df_images).select("image", "analysis_results.description.tags"))
```
### <a name="expected-results"></a>Ожидаемые результаты

|Изображение | tags|
|--|--|
| `https://raw.githubusercontent.com/Azure-Samples/cognitive-services-sample-data-files/master/ComputerVision/Images/objects.jpg` | [skating, person, man, outdoor, riding, sport, skateboard, young, board, shirt, air, park, boy, side, jumping, ramp, trick, doing, flying] |

## <a name="bing-image-search-sample"></a>Пример для Поиска изображений Bing
[Поиск изображений Bing](../../cognitive-services/bing-image-search/overview.md) — поиск в Интернете изображений, соответствующих запросу пользователя на естественном языке. В этом примере мы используем текстовый запрос для поиска изображений с кавычками. Он возвращает список URL-адресов изображений, содержащих фотографии, связанные с нашим запросом.


```python
from pyspark.ml import PipelineModel

# Number of images Bing will return per query
imgsPerBatch = 2
# A list of offsets, used to page into the search results
offsets = [(i*imgsPerBatch,) for i in range(10)]
# Since web content is our data, we create a dataframe with options on that data: offsets
bingParameters = spark.createDataFrame(offsets, ["offset"])

# Run the Bing Image Search service with our text query
bingSearch = (BingImageSearch()
    .setSubscriptionKey(bingsearch_service_key)
    .setOffsetCol("offset")
    .setQuery("Martin Luther King Jr. quotes")
    .setCount(imgsPerBatch)
    .setOutputCol("images"))

# Transformer that extracts and flattens the richly structured output of Bing Image Search into a simple URL column
getUrls = BingImageSearch.getUrlTransformer("images", "url")
pipeline_bingsearch = PipelineModel(stages=[bingSearch, getUrls])

# Show the results of your search: image URLs
res_bingsearch = pipeline_bingsearch.transform(bingParameters)
display(res_bingsearch.dropDuplicates())
```

### <a name="expected-results"></a>Ожидаемые результаты

|Изображение | 
|--|
|`http://everydaypowerblog.com/wp-content/uploads/2014/01/Martin-Luther-King-Jr.-Quotes-16.jpg` |
|`http://www.scrolldroll.com/wp-content/uploads/2017/06/6-25.png` |
| `http://abettertodaymedia.com/wp-content/uploads/2017/01/86783bd7a92960aedd058c91a1d10253.jpg`|
| `https://weneedfun.com/wp-content/uploads/2016/05/martin-luther-king-jr-quotes-11.jpg` |
| `http://www.sofreshandsogreen.com/wp-content/uploads/2012/01/martin-luther-king-jr-quote-sofreshandsogreendotcom.jpg` |
| `https://cdn.quotesgram.com/img/72/57/1104209728-martin_luther_king_jr_quotes_16.jpg` |
| `http://comicbookandbeyond.com/wp-content/uploads/2019/05/Martin-Luther-King-Jr.-Quotes.jpg` |
| `https://exposingthepain.files.wordpress.com/2015/01/martin-luther-king-jr-quotes-08.png` |
| `https://topmemes.me/wp-content/uploads/2020/01/Top-10-Martin-Luther-King-jr.-Quotes2-1024x538.jpg` |
| `http://img.picturequotes.com/2/581/580286/dr-martin-luther-king-jr-quote-1-picture-quote-1.jpg` |
| `http://parryz.com/wp-content/uploads/2017/06/Amazing-Martin-Luther-King-Jr-Quotes.jpg` |
| `http://everydaypowerblog.com/wp-content/uploads/2014/01/Martin-Luther-King-Jr.-Quotes1.jpg` |
| `https://lessonslearnedinlife.net/wp-content/uploads/2020/05/Martin-Luther-King-Jr.-Quotes-2020.jpg` |
| `https://quotesblog.net/wp-content/uploads/2015/10/Martin-Luther-King-Jr-Quotes-Wallpaper.jpg` |

## <a name="anomaly-detector-sample"></a>Пример для Детектора аномалий

[Детектор аномалий](../../cognitive-services/anomaly-detector/index.yml) — удобное средство для обнаружения несоответствующих данных во временных рядах. В этом примере мы используем службу для поиска аномалий во всех временных рядах.

```python
from pyspark.sql.functions import lit

# Create a dataframe with the point data that Anomaly Detector requires
df_timeseriesdata = spark.createDataFrame([
    ("1972-01-01T00:00:00Z", 826.0),
    ("1972-02-01T00:00:00Z", 799.0),
    ("1972-03-01T00:00:00Z", 890.0),
    ("1972-04-01T00:00:00Z", 900.0),
    ("1972-05-01T00:00:00Z", 766.0),
    ("1972-06-01T00:00:00Z", 805.0),
    ("1972-07-01T00:00:00Z", 821.0),
    ("1972-08-01T00:00:00Z", 20000.0), # anomaly
    ("1972-09-01T00:00:00Z", 883.0),
    ("1972-10-01T00:00:00Z", 898.0),
    ("1972-11-01T00:00:00Z", 957.0),
    ("1972-12-01T00:00:00Z", 924.0),
    ("1973-01-01T00:00:00Z", 881.0),
    ("1973-02-01T00:00:00Z", 837.0),
    ("1973-03-01T00:00:00Z", 9000.0) # anomaly
], ["timestamp", "value"]).withColumn("group", lit("series1"))

# Run the Anomaly Detector service to look for irregular data
anamoly_detector = (SimpleDetectAnomalies()
  .setSubscriptionKey(anomalydetector_key)
  .setLocation("eastasia")
  .setTimestampCol("timestamp")
  .setValueCol("value")
  .setOutputCol("anomalies")
  .setGroupbyCol("group")
  .setGranularity("monthly"))

# Show the full results of the analysis with the anomalies marked as "True"
display(anamoly_detector.transform(df_timeseriesdata).select("timestamp", "value", "anomalies.isAnomaly"))

```

### <a name="expected-results"></a>Ожидаемые результаты

|TIMESTAMP | value | isAnomaly |
|--|--|--|
| 1972-01-01T00:00:00Z|826.0|false|
|1972-02-01T00:00:00Z|799.0|false|
|1972-03-01T00:00:00Z|890.0|false|
|1972-04-01T00:00:00Z|900.0|false|
|1972-05-01T00:00:00Z|766.0|false|
|1972-06-01T00:00:00Z|805.0|false|
|1972-07-01T00:00:00Z|821.0|false|
|1972-08-01T00:00:00Z|20000.0|true|
|1972-09-01T00:00:00Z|883.0|false|
|1972-10-01T00:00:00Z|898.0|false|
|1972-11-01T00:00:00Z|957.0|false|
|1972-12-01T00:00:00Z|924.0|false|
|1973-01-01T00:00:00Z|881.0|false|
|1973-02-01T00:00:00Z|837.0|false|
|1973-03-01T00:00:00Z|9000.0|true|

## <a name="next-steps"></a>Следующие шаги

* [Ознакомьтесь с примерами записных книжек Synapse](https://github.com/Azure-Samples/Synapse/tree/main/Notebooks) 
* [Репозиторий MMLSpark в GitHub](https://github.com/Azure/mmlspark)