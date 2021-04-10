---
title: 'Учебник по регрессии: Автоматическое Машинное обучение'
titleSuffix: Azure Machine Learning
description: Создайте автоматизированный эксперимент машинного обучения, который создает модель регрессии на основе предоставленных вами данных обучения и параметров конфигурации.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: aniththa
ms.author: anumamah
ms.reviewer: nibaccam
ms.date: 08/14/2020
ms.custom: devx-track-python, automl
ms.openlocfilehash: 7f3052905d7594d64be9455c16239ebabd219849
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105565084"
---
# <a name="tutorial-use-automated-machine-learning-to-predict-taxi-fares"></a>Руководство по Использованию автоматического машинного обучения для прогнозирования тарифов на такси


В этом руководстве показано, как с помощью функции автоматизированного машинного обучения в Машинном обучении Azure создать модель регрессии для прогнозирования тарифов на такси в Нью-Йорке. В ходе этого процесса добавляются данные для обучения и настраиваются параметры, а также выполняется автоматическая итерация с использованием комбинаций разных методов нормализации или стандартизации признаков, моделей и настроек гиперпараметров для получения оптимальной модели.

![Схема процесса](./media/tutorial-auto-train-models/flow2.png)

В этом руководстве описано, как выполнять такие задачи:

> [!div class="checklist"]
> * Загрузка, преобразование и очистка данных с помощью Открытых наборов данных Azure.
> * Обучение автоматизированной регрессионной модели машинного обучения.
> * Расчет точности модели.

Если у вас еще нет подписки Azure, создайте бесплатную учетную запись Azure, прежде чем начинать работу. Опробуйте [бесплатную или платную версию](https://aka.ms/AMLFree) Машинного обучения Azure.

## <a name="prerequisites"></a>Предварительные требования

* Выполните инструкции из [руководства по установке](tutorial-1st-experiment-sdk-setup.md), если у вас еще нет рабочей области Машинного обучения Azure или виртуальной машины записных книжек.
* Завершив настройку, откройте записную книжку *tutorials/regression-automl-nyc-taxi-data/regression-automated-ml.ipynb*, используя тот же сервер записных книжек.

Это руководство также доступно на сайте [GitHub](https://github.com/Azure/MachineLearningNotebooks/tree/master/tutorials), если вы хотите запустить его в собственной [локальной среде](how-to-configure-environment.md#local). Чтобы получить нужные пакеты, сделайте следующее: 
* [Установите полную версию клиента `automl`](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/README.md#setup-using-a-local-conda-environment).
* Выполните команду `pip install azureml-opendatasets azureml-widgets`, чтобы получить необходимые пакеты.

## <a name="download-and-prepare-data"></a>Скачивание и подготовка данных

Импортируйте необходимые пакеты. Пакет открытых наборов данных содержит класс, представляющий каждый источник данных (например `NycTlcGreen`), для упрощения фильтрации параметров данных перед скачиванием.

```python
from azureml.opendatasets import NycTlcGreen
import pandas as pd
from datetime import datetime
from dateutil.relativedelta import relativedelta
```

Начните с создания кадра данных для хранения данных о поездках на такси. При работе в среде, отличающейся от Spark, в открытых наборах данных можно скачать за раз данные некоторых классов только за один месяц. Это ограничение позволяет избежать ошибки `MemoryError` при использовании больших наборов данных.

Чтобы получить данные о поездках на такси, поочередно скачайте данные за каждый месяц и, прежде чем добавить их к`green_taxi_df`случайной выборке 2,000 записей за каждый месяц, чтобы не раздувать кадр данных. После этого проверьте данные.


```python
green_taxi_df = pd.DataFrame([])
start = datetime.strptime("1/1/2015","%m/%d/%Y")
end = datetime.strptime("1/31/2015","%m/%d/%Y")

for sample_month in range(12):
    temp_df_green = NycTlcGreen(start + relativedelta(months=sample_month), end + relativedelta(months=sample_month)) \
        .to_pandas_dataframe()
    green_taxi_df = green_taxi_df.append(temp_df_green.sample(2000))

green_taxi_df.head(10)
```

|vendorID| lpepPickupDatetime|  lpepDropoffDatetime|    passengerCount| tripDistance|   puLocationId|   doLocationId|   pickupLongitude|    pickupLatitude| dropoffLongitude    |...|   paymentType |fareAmount |extra| mtaTax| improvementSurcharge|   tipAmount|  tollsAmount|    ehailFee|   totalAmount|    tripType|
|----|----|----|----|----|----|---|--|---|---|---|----|----|----|--|---|----|-----|----|----|----|
|131969|2|2015-01-11 05:34:44|2015-01-11 05:45:03|3|4,84|None|None|-73,88|40,84|-73,94|...|2|15,00|0,50|0,50|0,3|0,00|0,00|не число|16,30|
|1129817|2|2015-01-20 16:26:29|2015-01-20 16:30:26|1|0,69|None|None|-73,96|40,81|-73,96|...|2|4.50|1.00|0,50|0,3|0,00|0,00|не число|6,30|
|1278620|2|2015-01-01 05:58:10|2015-01-01 06:00:55|1|0,45|None|None|-73,92|40,76|-73,91|...|2|4,00|0,00|0,50|0,3|0,00|0,00|не число|4,80|
|348430|2|2015-01-17 02:20:50|2015-01-17 02:41:38|1|0,00|None|None|-73,81|40,70|-73,82|...|2|12,50|0,50|0,50|0,3|0,00|0,00|не число|13,80|
1269627|1|2015-01-01 05:04:10|2015-01-01 05:06:23|1|0,50|None|None|-73,92|40,76|-73,92|...|2|4,00|0,50|0,50|0|0,00|0,00|не число|5.00|
|811755|1|2015-01-04 19:57:51|2015-01-04 20:05:45|2|1,10|None|None|-73,96|40,72|-73,95|...|2|6,50|0,50|0,50|0,3|0,00|0,00|не число|7,80|
|737281|1|2015-01-03 12:27:31|2015-01-03 12:33:52|1|0,90|None|None|-73,88|40,76|-73,87|...|2|6,00|0,00|0,50|0,3|0,00|0,00|не число|6,80|
|113951|1|2015-01-09 23:25:51|2015-01-09 23:39:52|1|3,30|None|None|-73,96|40,72|-73,91|...|2|12,50|0,50|0,50|0,3|0,00|0,00|не число|13,80|
|150436|2|2015-01-11 17:15:14|2015-01-11 17:22:57|1|1,19|None|None|-73,94|40,71|-73,95|...|1|7,00|0,00|0,50|0,3|1,75|0,00|не число|9,55|
|432136|2|2015-01-22 23:16:33   2015-01-22 23:20:13 1   0.65|None|None|-73,94|40,71|-73,94|...|2|5.00|0,50|0,50|0,3|0,00|0,00|не число|6,30|

Теперь, когда начальные данные загружены, определите функцию для создания различных признаков на основе времени из поля с временем и датой начала поездки. При этом будут созданы новые поля для номера месяца, дня месяца, дня недели и времени дня, что позволит модели учитывать сезонные колебания на основе времени. Примените функцию `apply()` к кадру данных, чтобы последовательно применить функцию `build_time_features()` к каждой строке данных о поездках на такси.

```python
def build_time_features(vector):
    pickup_datetime = vector[0]
    month_num = pickup_datetime.month
    day_of_month = pickup_datetime.day
    day_of_week = pickup_datetime.weekday()
    hour_of_day = pickup_datetime.hour

    return pd.Series((month_num, day_of_month, day_of_week, hour_of_day))

green_taxi_df[["month_num", "day_of_month","day_of_week", "hour_of_day"]] = green_taxi_df[["lpepPickupDatetime"]].apply(build_time_features, axis=1)
green_taxi_df.head(10)
```

|vendorID| lpepPickupDatetime|  lpepDropoffDatetime|    passengerCount| tripDistance|   puLocationId|   doLocationId|   pickupLongitude|    pickupLatitude| dropoffLongitude    |...|   paymentType|fareAmount  |extra| mtaTax| improvementSurcharge|   tipAmount|  tollsAmount|    ehailFee|   totalAmount|tripType|month_num|day_of_month|day_of_week|hour_of_day
|----|----|----|----|----|----|---|--|---|---|---|----|----|----|--|---|----|-----|----|----|----|----|---|----|----|
|131969|2|2015-01-11 05:34:44|2015-01-11 05:45:03|3|4,84|None|None|-73,88|40,84|-73,94|...|2|15,00|0,50|0,50|0,3|0,00|0,00|не число|16,30|1.00|1|11|6|
|1129817|2|2015-01-20 16:26:29|2015-01-20 16:30:26|1|0,69|None|None|-73,96|40,81|-73,96|...|2|4.50|1.00|0,50|0,3|0,00|0,00|не число|6,30|1.00|1|20|1|
|1278620|2|2015-01-01 05:58:10|2015-01-01 06:00:55|1|0,45|None|None|-73,92|40,76|-73,91|...|2|4,00|0,00|0,50|0,3|0,00|0,00|не число|4,80|1.00|1|1|3|
|348430|2|2015-01-17 02:20:50|2015-01-17 02:41:38|1|0,00|None|None|-73,81|40,70|-73,82|...|2|12,50|0,50|0,50|0,3|0,00|0,00|не число|13,80|1.00|1|17|5|
1269627|1|2015-01-01 05:04:10|2015-01-01 05:06:23|1|0,50|None|None|-73,92|40,76|-73,92|...|2|4,00|0,50|0,50|0|0,00|0,00|не число|5.00|1.00|1|1|3|
|811755|1|2015-01-04 19:57:51|2015-01-04 20:05:45|2|1,10|None|None|-73,96|40,72|-73,95|...|2|6,50|0,50|0,50|0,3|0,00|0,00|не число|7,80|1.00|1|4|6|
|737281|1|2015-01-03 12:27:31|2015-01-03 12:33:52|1|0,90|None|None|-73,88|40,76|-73,87|...|2|6,00|0,00|0,50|0,3|0,00|0,00|не число|6,80|1.00|1|3|5|
|113951|1|2015-01-09 23:25:51|2015-01-09 23:39:52|1|3,30|None|None|-73,96|40,72|-73,91|...|2|12,50|0,50|0,50|0,3|0,00|0,00|не число|13,80|1.00|1|9|4|
|150436|2|2015-01-11 17:15:14|2015-01-11 17:22:57|1|1,19|None|None|-73,94|40,71|-73,95|...|1|7,00|0,00|0,50|0,3|1,75|0,00|не число|9,55|1.00|1|11|6|
|432136|2|2015-01-22 23:16:33   2015-01-22 23:20:13 1   0.65|None|None|-73,94|40,71|-73,94|...|2|5.00|0,50|0,50|0,3|0,00|0,00|не число|6,30|1.00|1|22|3|

Удалите столбцы, которые не требуются для обучения или построения дополнительных компонентов.

```python
columns_to_remove = ["lpepPickupDatetime", "lpepDropoffDatetime", "puLocationId", "doLocationId", "extra", "mtaTax",
                     "improvementSurcharge", "tollsAmount", "ehailFee", "tripType", "rateCodeID",
                     "storeAndFwdFlag", "paymentType", "fareAmount", "tipAmount"
                    ]
for col in columns_to_remove:
    green_taxi_df.pop(col)

green_taxi_df.head(5)
```

### <a name="cleanse-data"></a>Удаление данных

Примените функцию `describe()` к новому кадру данных, чтобы просмотреть сводные статистические данные для каждого поля.

```python
green_taxi_df.describe()
```

|vendorID|passengerCount|tripDistance|pickupLongitude|pickupLatitude|dropoffLongitude|dropoffLatitude|  totalAmount|month_num   day_of_month|day_of_week|hour_of_day
|----|----|---|---|----|---|---|---|---|---|---|
|count|48000,00|48000,00|48000,00|48000,00|48000,00|48000,00|48000,00|48000,00|48000,00|48000,00|
|mean|1,78|1,37|2,87|-73,83|40,69|-73.84|40,70|14,75|6,50|15,13|
|std|0,41|1,04|2,93|2.76|1,52|2.61|1,44|12,08|3.45|8,45|
|Min|1.00|0,00|0,00|-74,66|0,00|-74,66|0,00|-300,00|1.00|1.00|
|25 %|2,00|1.00|1,06|-73,96|40,70|-73,97|40,70|7,80|3,75|8,00|
|50%|2,00|1.00|1,90|-73,94|40,75|-73,94|40,75|11,30|6,50|15,00|
|75 %|2,00|1.00|3.60|-73,92|40,80|-73,91|40,79|17,80|9,25|22,00|
|max|2,00|9,00|97,57|0,00|41,93|0,00|41,94|450,00|12,00|30,00|


Из сводных статистических данных видно, что в некоторых полях присутствуют выбросы или значения, снижающие точность модели. Сначала отфильтруйте поля ширины/долготы в границах округа Манхэттен. Это отфильтрует более длительные поездки на такси или поездки, которые являются выбросами по отношению к другим компонентам.

Кроме того, отфильтруйте поле `tripDistance`, чтобы его значение было больше нуля, но меньше 31 мили (расстояние между двумя парами ширина/долгота по формуле гаверсинуса). Это исключает длительные поездки-выбросы, которые имеют несогласованную стоимость.

Наконец, поле `totalAmount` содержит отрицательные значения для тарифов на такси, которые не имеют смысла в контексте нашей модели, а поле `passengerCount` содержит неверные данные с минимальными значениями, равными нулю.

Отфильтруйте эти аномалии с помощью функций запросов, а затем удалите несколько последних столбцов, не требуемых для обучения.


```python
final_df = green_taxi_df.query("pickupLatitude>=40.53 and pickupLatitude<=40.88")
final_df = final_df.query("pickupLongitude>=-74.09 and pickupLongitude<=-73.72")
final_df = final_df.query("tripDistance>=0.25 and tripDistance<31")
final_df = final_df.query("passengerCount>0 and totalAmount>0")

columns_to_remove_for_training = ["pickupLongitude", "pickupLatitude", "dropoffLongitude", "dropoffLatitude"]
for col in columns_to_remove_for_training:
    final_df.pop(col)
```

Снова вызовите `describe()`, чтобы убедиться в том, что данные очищены должным образом. Теперь у вас есть подготовленный набор очищенных данных о поездках на такси, праздничных днях и погоде для обучения модели машинного обучения.

```python
final_df.describe()
```

## <a name="configure-workspace"></a>Настройка рабочей области

В существующей рабочей области создайте объект. Класс [Workspace](/python/api/azureml-core/azureml.core.workspace.workspace) принимает сведения о подписке и ресурсах Azure. Он также создает облачный ресурс для мониторинга и отслеживания работы модели. `Workspace.from_config()` считывает файл **config.json** и загружает сведения о проверке подлинности в объект с именем `ws`. `ws` используется в остальном коде в этом руководстве.

```python
from azureml.core.workspace import Workspace
ws = Workspace.from_config()
```

## <a name="split-the-data-into-train-and-test-sets"></a>Разделение данных на обучающий и тестовый наборы

Разделите данные на обучающий и тестовый наборы с помощью функции `train_test_split` из библиотеки `scikit-learn`. Эта функция выполняет разделение данных на набор данных X (**функции**) для обучения модели и набор данных Y (**прогнозируемые значения**) для тестирования.

Параметр `test_size` определяет процент данных, выделяемых для тестирования. Параметр `random_state` задает начальное значение для случайного генератора, чтобы разделение тестового обучения было детерминированным.

```python
from sklearn.model_selection import train_test_split

x_train, x_test = train_test_split(final_df, test_size=0.2, random_state=223)
```

Цель этого шага — получить точки данных для проверки готовой модели, которые не использовались для обучения модели, чтобы измерить истинную точность.

Другими словами, хорошо обученная модель должна уметь делать точные прогнозы на основе данных, которые она еще не видела. Теперь у вас есть данные, подготовленные для автоматического обучения модели машинного обучения.

## <a name="automatically-train-a-model"></a>Автоматическое обучение модели

Чтобы автоматически обучить модель, выполните следующие действия.
1. Определите параметры для запуска эксперимента. Вложите данные для обучения в конфигурацию и измените параметры, которые управляют процессом обучения.
1. Отправьте эксперимент для настройки модели. После отправки эксперимента процесс повторяется в различных алгоритмах машинного обучения и значениях гиперпараметров в соответствии с заданными вами ограничениями. Он выбирает наиболее подходящую модель путем оптимизации метрики точности.

### <a name="define-training-settings"></a>Определение параметров обучения

Определите параметры эксперимента и модели для обучения. Просмотрите полный список параметров [здесь](how-to-configure-auto-train.md). Отправка эксперимента с этими параметрами по умолчанию занимает примерно 5–20 минут, но если вы хотите снизить время выполнения, уменьшите значение параметра `experiment_timeout_hours`.

|Свойство| Значение в этом руководстве |Описание|
|----|----|---|
|**iteration_timeout_minutes**|10|Максимальная длительность каждой итерации в минутах. Увеличьте это значение для больших наборов данных, которым требуется больше времени для каждой итерации.|
|**experiment_timeout_hours**|0,3|Максимальное количество времени в часах, в течение которого могут быть пройдены все итерации до завершения эксперимента.|
|**enable_early_stopping**|True|Флаг для включения преждевременного завершения, если оценка не улучшается за короткое время.|
|**primary_metric**| spearman_correlation; | Метрика, который вы хотите оптимизировать. Наиболее подходящая модель будет выбрана на основе этой метрики.|
|**featurization**| авто | Используя значение **auto**, эксперимент может предварительно обработать входные данные (обработка отсутствующих данных, преобразование текста в числовой тип и т. д.).|
|**verbosity**| logging.INFO | Определяет уровень ведения журнала.|
|**n_cross_validations**|5|Количество разделений перекрестных проверок, которые нужно выполнить, если данные проверки не указаны.|

```python
import logging

automl_settings = {
    "iteration_timeout_minutes": 10,
    "experiment_timeout_hours": 0.3,
    "enable_early_stopping": True,
    "primary_metric": 'spearman_correlation',
    "featurization": 'auto',
    "verbosity": logging.INFO,
    "n_cross_validations": 5
}
```

Используйте определенные вами параметры обучения в качестве параметра `**kwargs` для объекта `AutoMLConfig`. Кроме того, укажите свои данные для обучения и тип модели, которая является `regression` в данном случае.

```python
from azureml.train.automl import AutoMLConfig

automl_config = AutoMLConfig(task='regression',
                             debug_log='automated_ml_errors.log',
                             training_data=x_train,
                             label_column_name="totalAmount",
                             **automl_settings)
```

> [!NOTE]
> Этапы предварительной обработки автоматизированного машинного обучения (нормализация компонентов, обработка недостающих данных, преобразование текста в числовой формат и т. д.) становятся частью базовой модели. При использовании модели для прогнозирования эти этапы предварительной обработки, которые выполнялись во время обучения, автоматически выполняются для входных данных.

### <a name="train-the-automatic-regression-model"></a>Обучение модели автоматической регрессии

Создайте объект эксперимента в рабочей области. Эксперимент выступает в качестве контейнера для отдельных запусков. Передайте определенный объект `automl_config` в эксперимент и задайте для выходных данных значение `True`, чтобы просмотреть ход выполнения.

После запуска эксперимента показанный вывод обновляется в реальном времени по мере выполнения эксперимента. Для каждой итерации вы видите тип модели, длительность выполнения и точность обучения. Поле `BEST` отслеживает лучшую оценку выполнения обучения на основе вашего типа метрики.

```python
from azureml.core.experiment import Experiment
experiment = Experiment(ws, "Tutorial-NYCTaxi")
local_run = experiment.submit(automl_config, show_output=True)
```

```output
Running on local machine
Parent Run ID: AutoML_1766cdf7-56cf-4b28-a340-c4aeee15b12b
Current status: DatasetFeaturization. Beginning to featurize the dataset.
Current status: DatasetEvaluation. Gathering dataset statistics.
Current status: FeaturesGeneration. Generating features for the dataset.
Current status: DatasetFeaturizationCompleted. Completed featurizing the dataset.
Current status: DatasetCrossValidationSplit. Generating individually featurized CV splits.
Current status: ModelSelection. Beginning model selection.

****************************************************************************************************
ITERATION: The iteration being evaluated.
PIPELINE: A summary description of the pipeline being evaluated.
DURATION: Time taken for the current iteration.
METRIC: The result of computing score on the fitted pipeline.
BEST: The best observed score thus far.
****************************************************************************************************

 ITERATION   PIPELINE                                       DURATION      METRIC      BEST
         0   StandardScalerWrapper RandomForest             0:00:16       0.8746    0.8746
         1   MinMaxScaler RandomForest                      0:00:15       0.9468    0.9468
         2   StandardScalerWrapper ExtremeRandomTrees       0:00:09       0.9303    0.9468
         3   StandardScalerWrapper LightGBM                 0:00:10       0.9424    0.9468
         4   RobustScaler DecisionTree                      0:00:09       0.9449    0.9468
         5   StandardScalerWrapper LassoLars                0:00:09       0.9440    0.9468
         6   StandardScalerWrapper LightGBM                 0:00:10       0.9282    0.9468
         7   StandardScalerWrapper RandomForest             0:00:12       0.8946    0.9468
         8   StandardScalerWrapper LassoLars                0:00:16       0.9439    0.9468
         9   MinMaxScaler ExtremeRandomTrees                0:00:35       0.9199    0.9468
        10   RobustScaler ExtremeRandomTrees                0:00:19       0.9411    0.9468
        11   StandardScalerWrapper ExtremeRandomTrees       0:00:13       0.9077    0.9468
        12   StandardScalerWrapper LassoLars                0:00:15       0.9433    0.9468
        13   MinMaxScaler ExtremeRandomTrees                0:00:14       0.9186    0.9468
        14   RobustScaler RandomForest                      0:00:10       0.8810    0.9468
        15   StandardScalerWrapper LassoLars                0:00:55       0.9433    0.9468
        16   StandardScalerWrapper ExtremeRandomTrees       0:00:13       0.9026    0.9468
        17   StandardScalerWrapper RandomForest             0:00:13       0.9140    0.9468
        18   VotingEnsemble                                 0:00:23       0.9471    0.9471
        19   StackEnsemble                                  0:00:27       0.9463    0.9471
```

## <a name="explore-the-results"></a>Просмотр результатов.

Просмотрите результаты автоматического обучения с помощью [мини-приложения Jupyter](/python/api/azureml-widgets/azureml.widgets). Мини-приложение позволяет просматривать график и таблицу всех отдельных итераций выполнения, а также метрики и метаданные точности обучения. Кроме того, с помощью раскрывающегося списка вы можете выполнить фильтрацию по метрикам точности, отличным от основной.

```python
from azureml.widgets import RunDetails
RunDetails(local_run).show()
```

![Мини-приложение Jupyter, данные о выполнении](./media/tutorial-auto-train-models/automl-dash-output.png)
![График в мини-приложении Jupyter](./media/tutorial-auto-train-models/automl-chart-output.png)

### <a name="retrieve-the-best-model"></a>Извлечение наиболее эффективной модели

Выберите наиболее эффективную модель из итераций. Функция `get_output` возвращает лучшее выполнение и соответствующую модель для последнего соответствующего вызова. Используя перегрузки `get_output`, вы можете извлекать лучшее выполнение и соответствующую ему модель по любой зарегистрированной метрике или по определенной итерации.

```python
best_run, fitted_model = local_run.get_output()
print(best_run)
print(fitted_model)
```

### <a name="test-the-best-model-accuracy"></a>Тестирование точности лучшей модели

Используйте лучшую модель для выполнения прогнозирования в тестовом наборе данных, чтобы прогнозировать тарифы на такси. Функция `predict` использует лучшую модель и прогнозирует значения Y **стоимость поездки** из набора данных `x_test`. Распечатайте первые 10 прогнозируемых значений стоимости из `y_predict`.

```python
y_test = x_test.pop("totalAmount")

y_predict = fitted_model.predict(x_test)
print(y_predict[:10])
```

Вычислите значение `root mean squared error` результатов. Преобразуйте кадр данных `y_test` в список для сравнения с прогнозируемыми значениями. Функция `mean_squared_error` принимает два массива значений и вычисляет значение среднеквадратической погрешности между ними. Квадратный корень из результата позволяет получить оценку ошибку в тех же единицах, что и для переменной y (**стоимость**). Она обозначает, насколько далеки полученные прогнозы тарифов на такси от фактических тарифов.

```python
from sklearn.metrics import mean_squared_error
from math import sqrt

y_actual = y_test.values.flatten().tolist()
rmse = sqrt(mean_squared_error(y_actual, y_predict))
rmse
```

Чтобы вычислить среднюю абсолютную погрешность в процентах (MAPE), запустите следующий код с помощью полных наборов данных `y_actual` и `y_predict`. Эта метрика вычисляет абсолютное отклонение между каждой парой прогнозируемого и фактического значения, а затем суммирует все отклонения. Затем эта сумма выражается в процентах от общей суммы фактических значений.

```python
sum_actuals = sum_errors = 0

for actual_val, predict_val in zip(y_actual, y_predict):
    abs_error = actual_val - predict_val
    if abs_error < 0:
        abs_error = abs_error * -1

    sum_errors = sum_errors + abs_error
    sum_actuals = sum_actuals + actual_val

mean_abs_percent_error = sum_errors / sum_actuals
print("Model MAPE:")
print(mean_abs_percent_error)
print()
print("Model Accuracy:")
print(1 - mean_abs_percent_error)
```

```output
Model MAPE:
0.14353867606052823

Model Accuracy:
0.8564613239394718
```


Из двух метрик точности прогнозирования вы видите, что модель достаточно хороша в прогнозировании тарифов на такси на основе функций набора данных, обычно в пределах + - 4,00 доллара США и приблизительной погрешностью 15%.

Традиционный процесс разработки модели машинного обучения является очень ресурсоемким и требует значительных предметных знаний и времени для выполнения десятков моделей и сравнения их результатов. Использование автоматического машинного обучения является отличным способом быстро протестировать множество различных моделей для вашего сценария.

## <a name="clean-up-resources"></a>Очистка ресурсов

Не выполняйте инструкции из этого раздела, если вы собираетесь изучать другие руководства по Машинному обучению Azure.

### <a name="stop-the-compute-instance"></a>Остановка вычислительной операции

[!INCLUDE [aml-stop-server](../../includes/aml-stop-server.md)]

### <a name="delete-everything"></a>Удаление всех ресурсов

Если вы не планируете использовать созданные ресурсы, удалите их, чтобы с вас не взималась плата.

1. На портале Azure выберите **Группы ресурсов** в левой части окна.
1. В списке выберите созданную группу ресурсов.
1. Выберите **Удалить группу ресурсов**.
1. Введите имя группы ресурсов. Теперь щелкните **Удалить**.

Вы также можете сохранить группу ресурсов, но удалить одну рабочую область. Отобразите свойства рабочей области и нажмите кнопку **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

Изучив это руководство об автоматическом машинном обучении, вы выполнили следующие задачи.

> [!div class="checklist"]
> * Настройка рабочей области и подготовка данных для эксперимента.
> * Обучение с помощью локальной автоматической модели регрессии с пользовательскими параметрами.
> * Просмотр и проверка результатов обучения.

[Разверните модель](tutorial-deploy-models-with-aml.md) с помощью службы "Машинное обучение Azure".
