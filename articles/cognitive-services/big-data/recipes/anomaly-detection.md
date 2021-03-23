---
title: 'Рецепт: диагностическое обслуживание с помощью Cognitive Services для больших данных'
titleSuffix: Azure Cognitive Services
description: В этом кратком руководстве показано, как выполнить распределенное обнаружение аномалий с помощью Cognitive Services для больших данных.
services: cognitive-services
author: mhamilton723
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: how-to
ms.date: 07/06/2020
ms.author: marhamil
ms.custom: devx-track-python
ms.openlocfilehash: 5a583d74ae0bf421a7a863a65442d250489a2f8f
ms.sourcegitcommit: 2c1b93301174fccea00798df08e08872f53f669c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104775397"
---
# <a name="recipe-predictive-maintenance-with-the-cognitive-services-for-big-data"></a>Рецепт: диагностическое обслуживание с помощью Cognitive Services для больших данных

В этом рецепте показано, как можно использовать Azure синапсе Analytics и Cognitive Services на Apache Spark для диагностического обслуживания устройств IoT. Мы будем следовать примерам [ссылок CosmosDB и синапсе](https://github.com/Azure-Samples/cosmosdb-synapse-link-samples) . Чтобы упростить работу, в этом рецепте мы будем считывать данные прямо из CSV-файла, а не получать потоковые данные через CosmosDB и синапсе Link. Мы настоятельно рекомендуем ознакомиться с примером синапсе Link.

## <a name="hypothetical-scenario"></a>Гипотетический сценарий

Гипотетический сценарий — это завод питания, где устройства IoT отслеживают [Steam турбины](https://en.wikipedia.org/wiki/Steam_turbine). Коллекция Иотсигналс содержит данные об оборотах в минуту (RPM) и МВт (MW) для каждого ветроэлектрической. Сигналы от Steam турбины анализируются и обнаруживаются аномальные сигналы.

В данных может воздержаться выброс случайной частоты. В таких случаях значения RPM будут выводиться, и MW выходные данные будут отключены для защиты канала. Идея состоит в том, чтобы видеть данные в одно и то же время, но с разными сигналами.

## <a name="prerequisites"></a>Предварительные требования

* подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services).
* [Рабочая область Azure синапсе](../../../synapse-analytics/quickstart-create-workspace.md) , настроенная с [бессерверным пулом Apache Spark](../../../synapse-analytics/quickstart-create-apache-spark-pool-portal.md)

## <a name="setup"></a>Настройка

### <a name="create-an-anomaly-detector-resource"></a>Создание ресурса "Детектор аномалий"

Ресурсами Azure, на которые вы подписаны, будет представлено семейство служб Azure Cognitive Services. Создайте ресурс для переводчика с помощью [портал Azure](../../cognitive-services-apis-create-account.md) или [Azure CLI](../../cognitive-services-apis-create-account-cli.md). Также можно:

- Просмотр существующего ресурса в  [портал Azure](https://portal.azure.com/).

Запишите конечную точку и ключ для этого ресурса, и это будет необходимо в этом руководством.

## <a name="enter-your-service-keys"></a>Введите ключи службы

Начнем с добавления ключа и расположения.

```python
service_key = None # Paste your anomaly detector key here
location = None # Paste your anomaly detector location here

assert (service_key is not None)
assert (location is not None)
```

## <a name="read-data-into-a-dataframe"></a>Считать данные в таблицу данных

Теперь выполним чтение файла Иотсигналс в кадр данных. Откройте новую записную книжку в рабочей области синапсе и создайте кадр данных из файла.

```python
df_signals = spark.read.csv("wasbs://publicwasb@mmlspark.blob.core.windows.net/iot/IoTSignals.csv", header=True, inferSchema=True)
```

### <a name="run-anomaly-detection-using-cognitive-services-on-spark"></a>Выполнение обнаружения аномалий с помощью Cognitive Services в Spark

Цель состоит в том, чтобы найти экземпляры, в которых сигналы устройств Интернета вещей были выводить аномальные значения, чтобы мы могли увидеть, что что-то пошло не так, и выполнить прогнозное обслуживание. Чтобы сделать это, давайте воспользуемся средством обнаружения аномалий в Spark:

```python
from pyspark.sql.functions import col, struct
from mmlspark.cognitive import SimpleDetectAnomalies
from mmlspark.core.spark import FluentAPI

detector = (SimpleDetectAnomalies()
    .setSubscriptionKey(service_key)
    .setLocation(location)
    .setOutputCol("anomalies")
    .setGroupbyCol("grouping")
    .setSensitivity(95)
    .setGranularity("secondly"))

df_anomaly = (df_signals
    .where(col("unitSymbol") == 'RPM')
    .withColumn("timestamp", col("dateTime").cast("string"))
    .withColumn("value", col("measureValue").cast("double"))
    .withColumn("grouping", struct("deviceId"))
    .mlTransform(detector)).cache()

df_anomaly.createOrReplaceTempView('df_anomaly')
```

Рассмотрим данные:

```python
df_anomaly.select("timestamp","value","deviceId","anomalies.isAnomaly").show(3)
```

Эта ячейка должна дать результат, который выглядит следующим образом:

| TIMESTAMP           |   value | deviceId   | isAnomaly   |
|:--------------------|--------:|:-----------|:------------|
| 2020-05-01 18:33:51 |    3174 | dev-7      | Неверно       |
| 2020-05-01 18:33:52 |    2976 | dev-7      | Неверно       |
| 2020-05-01 18:33:53 |    2714 | dev-7      | Неверно       |


 ## <a name="visualize-anomalies-for-one-of-the-devices"></a>Визуализация аномалий для одного из устройств

IoTSignals.csv имеет сигналы от нескольких устройств IoT. Мы будем сосредоточиться на конкретном устройстве и визуализировать аномальные выходные данные с устройства.

```python
df_anomaly_single_device = spark.sql("""
select
  timestamp,
  measureValue,
  anomalies.expectedValue,
  anomalies.expectedValue + anomalies.upperMargin as expectedUpperValue,
  anomalies.expectedValue - anomalies.lowerMargin as expectedLowerValue,
  case when anomalies.isAnomaly=true then 1 else 0 end as isAnomaly
from
  df_anomaly
where deviceid = 'dev-1' and timestamp < '2020-04-29'
order by timestamp
limit 200""")
```

Теперь, когда мы создали кадр данных, который представляет аномалии для конкретного устройства, мы можем визуализировать эти аномалии:

```python
import matplotlib.pyplot as plt
from pyspark.sql.functions import col

adf = df_anomaly_single_device.toPandas()
adf_subset = df_anomaly_single_device.where(col("isAnomaly") == 1).toPandas()

plt.figure(figsize=(23,8))
plt.plot(adf['timestamp'],adf['expectedUpperValue'], color='darkred', linestyle='solid', linewidth=0.25, label='UpperMargin')
plt.plot(adf['timestamp'],adf['expectedValue'], color='darkgreen', linestyle='solid', linewidth=2, label='Expected Value')
plt.plot(adf['timestamp'],adf['measureValue'], 'b', color='royalblue', linestyle='dotted', linewidth=2, label='Actual')
plt.plot(adf['timestamp'],adf['expectedLowerValue'],  color='black', linestyle='solid', linewidth=0.25, label='Lower Margin')
plt.plot(adf_subset['timestamp'],adf_subset['measureValue'], 'ro', label = 'Anomaly')
plt.legend()
plt.title('RPM Anomalies with Confidence Intervals')
plt.show()
```

В случае успеха выходные данные будут выглядеть следующим образом:

![График детектора аномалий](../media/anomaly-output.png)

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как выполнить прогнозируемое обслуживание в масштабе с помощью Azure Cognitive Services, Azure синапсе Analytics и Azure CosmosDB. Дополнительные сведения см. в полном примере на сайте [GitHub](https://github.com/Azure-Samples/cosmosdb-synapse-link-samples).
