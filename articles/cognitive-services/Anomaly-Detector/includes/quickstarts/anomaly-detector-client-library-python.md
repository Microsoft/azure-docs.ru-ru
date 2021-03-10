---
title: Краткое руководство по использованию клиентской библиотеки Детектора аномалий (Python)
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 11/25/2020
ms.author: mbullwin
ms.openlocfilehash: 216c45bf097718f6a696e64c8bd9c8718fc0185e
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102444914"
---
Приступите к работе с клиентской библиотекой Детектора аномалий для Python. Выполните следующие действия, чтобы установить пакет и приступить к использованию алгоритмов, предоставляемых службой. Служба Детектора аномалий позволяет находить аномалии в данных временных рядов, автоматически применяя для них наиболее подходящие модели, независимо от отрасли, сценария или объема данных.

Клиентскую библиотеку Детектора аномалий для Python можно использовать для таких задач:

* обнаружение аномалий в наборе данных временных рядов с использованием пакетного запроса;
* Обнаружение состояния аномалии последней точки данных во временном ряду.
* обнаружение точек изменения тенденций в наборе данных.

[Справочная документация по библиотеке](https://go.microsoft.com/fwlink/?linkid=2090370) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-cognitiveservices-anomalydetector) | [Пакет (PyPi)](https://pypi.org/project/azure-ai-anomalydetector/) | [Поиск примера кода на GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/anomalydetector/azure-ai-anomalydetector/samples)

## <a name="prerequisites"></a>Предварительные требования

* [Python 3.x](https://www.python.org/)
* [Библиотека анализа данных Pandas](https://pandas.pydata.org/).
* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services).
* Получив подписку Azure, перейдите к <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAnomalyDetector"  title="Создание ресурса Детектора аномалий"  target="_blank">созданию ресурса Детектора аномалий </a> на портале Azure, чтобы получить ключ и конечную точку. Дождитесь, пока закончится развертывание, и нажмите кнопку **Перейти к ресурсу**.
    * Для подключения приложения к API "Детектор аномалий" потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.


## <a name="setting-up"></a>Настройка

[!INCLUDE [anomaly-detector-environment-variables](../environment-variables.md)]

### <a name="create-a-new-python-application"></a>Создание приложения Python

 Создайте файл Python и импортируйте следующие библиотеки.

```python
import os
from azure.ai.anomalydetector import AnomalyDetectorClient
from azure.ai.anomalydetector.models import DetectRequest, TimeSeriesPoint, TimeGranularity, \
    AnomalyDetectorError
from azure.core.credentials import AzureKeyCredential
import pandas as pd
```

Создайте переменные для ключа в качестве переменной среды, путь к файлу данных временных рядов, а также расположение Azure для вашей подписки. Например, `westus2`.

```python
SUBSCRIPTION_KEY = os.environ["ANOMALY_DETECTOR_KEY"]
ANOMALY_DETECTOR_ENDPOINT = os.environ["ANOMALY_DETECTOR_ENDPOINT"]
TIME_SERIES_DATA_PATH = os.path.join("./sample_data", "request-data.csv")
```

### <a name="install-the-client-library"></a>Установка клиентской библиотеки

После установки Python вы можете установить клиентскую библиотеку с помощью следующей команды:

```console
pip install --upgrade azure-ai-anomalydetector
```

## <a name="object-model"></a>Объектная модель

Клиент Детектора аномалий — это объект [AnomalyDetectorClient](https://github.com/Azure/azure-sdk-for-python/blob/0b8622dc249969c2f01c5d7146bd0bb36bb106dd/sdk/cognitiveservices/azure-cognitiveservices-anomalydetector/azure/cognitiveservices/anomalydetector/_anomaly_detector_client.py), который выполняет проверку подлинности в Azure с помощью вашего ключа. Клиент теперь может выполнять обнаружение аномалий с использованием [detect_entire_series](https://github.com/Azure/azure-sdk-for-python/blob/bf9d44f2a50aea46a59c4cb83ccfccaff5e2b218/sdk/anomalydetector/azure-ai-anomalydetector/azure/ai/anomalydetector/operations/_anomaly_detector_client_operations.py#L26) для всего набора данных и [detect_last_point](https://github.com/Azure/azure-sdk-for-python/blob/bf9d44f2a50aea46a59c4cb83ccfccaff5e2b218/sdk/anomalydetector/azure-ai-anomalydetector/azure/ai/anomalydetector/operations/_anomaly_detector_client_operations.py#L87) для последней точки данных. Функция [detect_change_point](https://github.com/Azure/azure-sdk-for-python/blob/bf9d44f2a50aea46a59c4cb83ccfccaff5e2b218/sdk/anomalydetector/azure-ai-anomalydetector/azure/ai/anomalydetector/aio/operations_async/_anomaly_detector_client_operations_async.py#L142) обнаруживает точки, которые отмечают изменения тенденции.

Данные временного ряда отправляются в виде ряда объекта [TimeSeriesPoints](https://github.com/Azure/azure-sdk-for-python/blob/bf9d44f2a50aea46a59c4cb83ccfccaff5e2b218/sdk/anomalydetector/azure-ai-anomalydetector/azure/ai/anomalydetector/models/_models_py3.py#L370). Объект `DetectRequest` содержит свойства для описания данных (например, `TimeGranularity`) и параметры для обнаружения аномалий.

Ответ Детектора аномалий — это объект [LastDetectResponse](/python/api/azure-cognitiveservices-anomalydetector/azure.cognitiveservices.anomalydetector.models.lastdetectresponse), [EntireDetectResponse](/python/api/azure-cognitiveservices-anomalydetector/azure.cognitiveservices.anomalydetector.models.entiredetectresponse) или [ChangePointDetectResponse](https://github.com/Azure/azure-sdk-for-python/blob/bf9d44f2a50aea46a59c4cb83ccfccaff5e2b218/sdk/anomalydetector/azure-ai-anomalydetector/azure/ai/anomalydetector/models/_models_py3.py#L107) (в зависимости от используемого метода).

## <a name="code-examples"></a>Примеры кода

Эти фрагменты кода показывают, как выполнить следующие действия с помощью клиентской библиотеки Детектора аномалий для Python:

* [аутентификация клиента](#authenticate-the-client);
* [загрузка набора данных временного ряда из файла](#load-time-series-data-from-a-file);
* [обнаружение аномалий во всем наборе данных](#detect-anomalies-in-the-entire-data-set);
* [обнаружение состояния аномалии последней точки данных](#detect-the-anomaly-status-of-the-latest-data-point).
* [Обнаружение точек изменения в наборе данных](#detect-change-points-in-the-data-set)

## <a name="authenticate-the-client"></a>Аутентификация клиента

Добавьте переменную расположения Azure в конечную точку и проверьте подлинность клиента с помощью ключа.

```python
client = AnomalyDetectorClient(AzureKeyCredential(SUBSCRIPTION_KEY), ANOMALY_DETECTOR_ENDPOINT)
```

## <a name="load-time-series-data-from-a-file"></a>Загрузка данных временного ряда из файла

Скачайте пример данных для этого краткого руководства с сайта [GitHub](https://github.com/Azure-Samples/AnomalyDetector/blob/master/example-data/request-data.csv):
1. В браузере щелкните правой кнопкой мыши пункт **Без обработки**.
2. Выберите **Сохранить ссылку как**.
3. Сохраните файл в формате CSV в каталоге вашего приложения.

Данные временных рядов форматируются как CSV-файл и будут отправлены в API Детектора аномалий.

Загрузите свой файл данных с помощью метода `read_csv()` библиотеки Pandas и создайте переменную с пустым списком для хранения ряда данных. Выполните итерацию файла и добавьте данные в виде объекта `TimeSeriesPoint`. Этот объект будет содержать метку времени и числовое значение из строк вашего CSV-файла данных.

```python
series = []
data_file = pd.read_csv(TIME_SERIES_DATA_PATH, header=None, encoding='utf-8', parse_dates=[0])
for index, row in data_file.iterrows():
    series.append(TimeSeriesPoint(timestamp=row[0], value=row[1]))
```

Создайте объект `DetectRequest` с вашим временным рядом и `TimeGranularity` (или периодичности) точек данных. Например, `TimeGranularity.daily`.

```python
request = DetectRequest(series=series, granularity=TimeGranularity.daily)
```

## <a name="detect-anomalies-in-the-entire-data-set"></a>Обнаружение аномалий во всем наборе данных

Вызовите API для обнаружения аномалий во всех данных временного ряда с помощью метода `detect_entire_series` клиента. Сохраните возвращенный объект [EntireDetectResponse](/python/api/azure-cognitiveservices-anomalydetector/azure.cognitiveservices.anomalydetector.models.entiredetectresponse). Выполните итерацию списка ответов `is_anomaly` и выведите индексы любых значений `true`. Эти значения соответствуют индексу аномальных точек данных, если они были найдены.

```python
print('Detecting anomalies in the entire time series.')

try:
    response = client.detect_entire_series(request)
except AnomalyDetectorError as e:
    print('Error code: {}'.format(e.error.code), 'Error message: {}'.format(e.error.message))
except Exception as e:
    print(e)

if any(response.is_anomaly):
    print('An anomaly was detected at index:')
    for i, value in enumerate(response.is_anomaly):
        if value:
            print(i)
else:
    print('No anomalies were detected in the time series.')
```

## <a name="detect-the-anomaly-status-of-the-latest-data-point"></a>Обнаружение состояний аномалии последней точки данных

Вызовите API Детектора аномалий, чтобы определить, является ли последняя точка данных аномальной, с помощью метода `detect_last_point` клиента и сохраните возвращенный объект `LastDetectResponse`. Значение `is_anomaly` в ответе представляет собой логическое значение, указывающее состояние аномалий этой точки.  

```python
print('Detecting the anomaly status of the latest data point.')

try:
    response = client.detect_last_point(request)
except AnomalyDetectorError as e:
    print('Error code: {}'.format(e.error.code), 'Error message: {}'.format(e.error.message))
except Exception as e:
    print(e)

if response.is_anomaly:
    print('The latest point is detected as anomaly.')
else:
    print('The latest point is not detected as anomaly.')
```

## <a name="detect-change-points-in-the-data-set"></a>Обнаружение точек изменения в наборе данных

Вызовите API для обнаружения точек изменения в данных временных рядов с помощью метода `detect_change_point` клиента. Сохраните полученный объект `ChangePointDetectResponse`. Выполните итерацию списка ответов `is_change_point` и выведите индексы любых значений `true`. Эти значения соответствуют индексам точек изменения тенденций, если они были обнаружены.

```python
print('Detecting change points in the entire time series.')

try:
    response = client.detect_change_point(request)
except AnomalyDetectorError as e:
    print('Error code: {}'.format(e.error.code), 'Error message: {}'.format(e.error.message))
except Exception as e:
    print(e)

if any(response.is_change_point):
    print('An change point was detected at index:')
    for i, value in enumerate(response.is_change_point):
        if value:
            print(i)
else:
    print('No change point were detected in the time series.')
```

## <a name="run-the-application"></a>Выполнение приложения

Запустите приложение, выполнив команду `python` и используя имя файла.

[!INCLUDE [anomaly-detector-next-steps](../quickstart-cleanup-next-steps.md)]
