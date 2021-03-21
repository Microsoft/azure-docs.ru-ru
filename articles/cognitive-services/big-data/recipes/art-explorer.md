---
title: 'Рецепт: интеллектуальное исследование с помощью Cognitive Services для больших данных'
titleSuffix: Azure Cognitive Services
description: В этом рецепте показано, как создать базу данных с возможностью поиска с помощью службы поиска Azure и MMLSpark.
services: cognitive-services
author: mhamilton723
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: how-to
ms.date: 07/06/2020
ms.author: marhamil
ms.custom: devx-track-python
ms.openlocfilehash: 5a65ff28a38e42e05844063a330c0325f16b2247
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94363295"
---
# <a name="recipe-intelligent-art-exploration-with-the-cognitive-services-for-big-data"></a>Рецепт: интеллектуальное исследование с помощью Cognitive Services для больших данных

В этом примере мы будем использовать Cognitive Services для больших данных, чтобы добавить интеллектуальные заметки в коллекцию Open Access из музей, заносимого в рисунок. Это позволит нам создать интеллектуальную поисковую подсистему с помощью поиска Azure даже без заметок вручную. 

## <a name="prerequisites"></a>Предварительные условия

* Необходимо иметь ключ подписки для Компьютерное зрение и Когнитивный поиск. Следуйте инструкциям в разделе [Создание учетной записи Cognitive Services](../../cognitive-services-apis-create-account.md) , чтобы подписаться на компьютерное зрение и получить ключ.
  > [!NOTE]
  > Сведения о ценах см. в статье [когнитивный Поиск Azure](https://azure.microsoft.com/services/search/#pricing).

## <a name="import-libraries"></a>Импорт библиотек

Выполните следующую команду, чтобы импортировать библиотеки для этого рецепта.

```python
import os, sys, time, json, requests
from pyspark.ml import Transformer, Estimator, Pipeline
from pyspark.ml.feature import SQLTransformer
from pyspark.sql.functions import lit, udf, col, split
```

## <a name="set-up-subscription-keys"></a>Настройка ключей подписки

Выполните следующую команду, чтобы настроить переменные для ключей служб. Вставьте ключи подписки для Компьютерное зрение и Azure Когнитивный поиск.

```python
VISION_API_KEY = 'INSERT_COMPUTER_VISION_SUBSCRIPTION_KEY'
AZURE_SEARCH_KEY = 'INSERT_AZURE_COGNITIVE_SEARCH_SUBSCRIPTION_KEY'
search_service = "mmlspark-azure-search"
search_index = "test"
```

## <a name="read-the-data"></a>Чтение данных

Выполните следующую команду, чтобы загрузить данные из открытой коллекции доступа.

```python
data = spark.read\
  .format("csv")\
  .option("header", True)\
  .load("wasbs://publicwasb@mmlspark.blob.core.windows.net/metartworks_sample.csv")\
  .withColumn("searchAction", lit("upload"))\
  .withColumn("Neighbors", split(col("Neighbors"), ",").cast("array<string>"))\
  .withColumn("Tags", split(col("Tags"), ",").cast("array<string>"))\
  .limit(25)
```

<a name="AnalyzeImages"></a>

## <a name="analyze-the-images"></a>Анализ изображений

Выполните следующую команду, чтобы использовать Компьютерное зрение в коллекции Open Access артворкс. В результате вы получите визуальные функции из артворкс.

```python
from mmlspark.cognitive import AnalyzeImage
from mmlspark.stages import SelectColumns

#define pipeline
describeImage = (AnalyzeImage()
  .setSubscriptionKey(VISION_API_KEY)
  .setLocation("eastus")
  .setImageUrlCol("PrimaryImageUrl")
  .setOutputCol("RawImageDescription")
  .setErrorCol("Errors")
  .setVisualFeatures(["Categories", "Tags", "Description", "Faces", "ImageType", "Color", "Adult"])
  .setConcurrency(5))

df2 = describeImage.transform(data)\
  .select("*", "RawImageDescription.*").drop("Errors", "RawImageDescription")
```

<a name="CreateSearchIndex"></a>

## <a name="create-the-search-index"></a>Создание индекса поиска

Выполните следующую команду, чтобы записать результаты в службу поиска Azure, чтобы создать поисковую подсистему артворкс с обогащенными метаданными из Компьютерное зрение.

```python
from mmlspark.cognitive import *
df2.writeToAzureSearch(
  subscriptionKey=AZURE_SEARCH_KEY,
  actionCol="searchAction",
  serviceName=search_service,
  indexName=search_index,
  keyCol="ObjectID"
)
```

## <a name="query-the-search-index"></a>Запрос индекса поиска

Выполните следующую команду, чтобы запросить индекс поиска Azure.

```python
url = 'https://{}.search.windows.net/indexes/{}/docs/search?api-version=2019-05-06'.format(search_service, search_index)
requests.post(url, json={"search": "Glass"}, headers = {"api-key": AZURE_SEARCH_KEY}).json()
```

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как использовать [Cognitive Services для больших данных при обнаружении аномалий](anomaly-detection.md).