---
title: Использование конечных точек Машинное обучение Azure Studio (классическая модель) в Azure Stream Analytics
description: В этой статье объясняется, как использовать определяемые пользователем функции машинного обучения в Azure Stream Analytics.
author: jseb225
ms.author: jeanb
ms.service: stream-analytics
ms.topic: how-to
ms.date: 06/11/2019
ms.openlocfilehash: 1dc85cb10a9e4300c57ad03900d8c8924988c6d5
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104588126"
---
# <a name="azure-machine-learning-studio-classic-integration-in-stream-analytics"></a>Интеграция Машинное обучение Azure Studio (классическая модель) в Stream Analytics
Stream Analytics поддерживает пользовательские функции, которые обращаются к конечным точкам Студии машинного обучения Azure (классической). Поддержка этой возможности в REST API описана в статье [Библиотека REST API Stream Analytics](/rest/api/streamanalytics/). В этой статье приведены дополнительные сведения, необходимые для успешной реализации этой возможности в Stream Analytics. Также было размещено руководство, которое доступно [здесь](stream-analytics-machine-learning-integration-tutorial.md).

## <a name="overview-azure-machine-learning-studio-classic-terminology"></a>Общие сведения. Терминология Студии машинного обучения Azure (классической)
Студия машинного обучения Microsoft Azure (классическая) — это средство для совместной работы, поддерживающее функцию перетаскивания объектов и предназначенное для создания, тестирования и развертывания решений для прогнозного анализа данных. Это средство называется *машинное обучение Azure Studio (классическая модель)*. Studio (классическая модель) используется для взаимодействия с ресурсами машинного обучения, а также для простого создания, тестирования и итерации проекта. Эти ресурсы и их определения приведены ниже.

* **Рабочая** область. *Рабочая область* — это контейнер, содержащий все остальные ресурсы машинного обучения в контейнере для управления и управления.
* **Эксперимент**. *Эксперименты* создаются специалистами по обработке и анализу данных для использования наборов данных и обучения моделей машинного обучения.
* **Конечная точка**. *конечные точки* — это объект Studio (классический), используемый для получения в качестве входных данных функций, применения указанной модели машинного обучения и возврата выводимых данных.
* **Оценивающая веб-служба**. *Оценивающая веб-служба* — это коллекция конечных точек, упомянутых выше.

Каждая конечная точка имеет API для пакетного и синхронного выполнения. В Stream Analytics используется синхронное выполнение. В Студии машинного обучения Azure (классической) соответствующая служба называется [службой обработки запросов и ответов](../machine-learning/classic/consume-web-services.md).

## <a name="studio-classic-resources-needed-for-stream-analytics-jobs"></a>Ресурсы студии (классические), необходимые для заданий Stream Analytics
Для успешной обработки задания Stream Analytics требуются конечная точка типа "запрос — ответ", [apikey](../machine-learning/classic/consume-web-services.md)и определение swagger. В Stream Analytics есть дополнительная конечная точка, которая создает URL-адрес конечной точки swagger, ищет интерфейс, а затем возвращает пользователю определение определяемой пользователем функции по умолчанию.

## <a name="configure-a-stream-analytics-and-studio-classic-udf-via-rest-api"></a>Настройка Stream Analytics и Studio (классической) UDF через REST API
С помощью интерфейсов API RESTFUL можно настроить задание для вызова функций Studio (классическая модель). Для этого необходимо выполнить следующие шаги:

1. Создание задания Stream Analytics
2. Определение входных данных.
3. Определение выходных данных.
4. Создание определяемой пользователем функции
5. Создание преобразования Stream Analytics, которое вызывает определяемую пользователем функцию.
6. Запустите задание

## <a name="creating-a-udf-with-basic-properties"></a>Создание определяемой пользователем функции с базовыми свойствами
В следующем примере кода создается скалярная определяемая пользователем функция с именем *newudf*, которая привязывается к конечной точке Студии машинного обучения Azure (классической). Обратите внимание, что *конечную точку* (URI службы) можно найти на странице справки по API для выбранной службы, а *apiKey* — на главной странице служб.

```
    PUT : /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.StreamAnalytics/streamingjobs/<streamingjobName>/functions/<udfName>?api-version=<apiVersion>
```

Пример тела запроса:

```json
    {
        "name": "newudf",
        "properties": {
            "type": "Scalar",
            "properties": {
                "binding": {
                    "type": "Microsoft.MachineLearning/WebService",
                    "properties": {
                        "endpoint": "https://ussouthcentral.services.azureml.net/workspaces/f80d5d7a77fb4b46bf2a30c63c078dca/services/b7be5e40fd194258796fb402c1958eaf/execute ",
                        "apiKey": "replacekeyhere"
                    }
                }
            }
        }
    }
```

## <a name="call-retrievedefaultdefinition-endpoint-for-default-udf"></a>Вызов конечной точки RetrieveDefaultDefinition для определяемой пользователем функции по умолчанию
После создания схемы определяемой пользователем функции требуется получить ее полное определение. Конечная точка RetrieveDefaultDefinition помогает получить определение по умолчанию для скалярной функции, которая привязана к конечной точке Студии машинного обучения Azure (классическая). В полезных данных ниже необходимо получить определение UDF по умолчанию для скалярной функции, привязанной к конечной точке студии (классической). Она не указывает фактическую конечную точку, так как она уже была предоставлена во время запроса на размещение. Stream Analytics вызывает указанную в запросе конечную точку, если она задана явно. В противном случае будет использоваться конечная точка, указанная изначально. Здесь определяемая пользователем функция (UDF) принимает один строковой параметр (предложение) и возвращает один вывод строкового типа, который указывает на метку "тональность" для этого предложения.

```
POST : /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.StreamAnalytics/streamingjobs/<streamingjobName>/functions/<udfName>/RetrieveDefaultDefinition?api-version=<apiVersion>
```

Пример тела запроса:

```json
    {
        "bindingType": "Microsoft.MachineLearning/WebService",
        "bindingRetrievalProperties": {
            "executeEndpoint": null,
            "udfType": "Scalar"
        }
    }
```

Вывод будет выглядеть примерно так, как показано ниже.

```json
    {
        "name": "newudf",
        "properties": {
            "type": "Scalar",
            "properties": {
                "inputs": [{
                    "dataType": "nvarchar(max)",
                    "isConfigurationParameter": null
                }],
                "output": {
                    "dataType": "nvarchar(max)"
                },
                "binding": {
                    "type": "Microsoft.MachineLearning/WebService",
                    "properties": {
                        "endpoint": "https://ussouthcentral.services.azureml.net/workspaces/f80d5d7a77ga4a4bbf2a30c63c078dca/services/b7be5e40fd194258896fb602c1858eaf/execute",
                        "apiKey": null,
                        "inputs": {
                            "name": "input1",
                            "columnNames": [{
                                "name": "tweet",
                                "dataType": "string",
                                "mapTo": 0
                            }]
                        },
                        "outputs": [{
                            "name": "Sentiment",
                            "dataType": "string"
                        }],
                        "batchSize": 10
                    }
                }
            }
        }
    }
```

## <a name="patch-udf-with-the-response"></a>Исправление для определяемой пользователем функции с помощью ответа
Теперь необходимо исправить определяемую пользователем функцию с использованием предыдущего ответа, как показано ниже.

```
PATCH : /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.StreamAnalytics/streamingjobs/<streamingjobName>/functions/<udfName>?api-version=<apiVersion>
```

Текст запроса (вывод из RetrieveDefaultDefinition):

```json
    {
        "name": "newudf",
        "properties": {
            "type": "Scalar",
            "properties": {
                "inputs": [{
                    "dataType": "nvarchar(max)",
                    "isConfigurationParameter": null
                }],
                "output": {
                    "dataType": "nvarchar(max)"
                },
                "binding": {
                    "type": "Microsoft.MachineLearning/WebService",
                    "properties": {
                        "endpoint": "https://ussouthcentral.services.azureml.net/workspaces/f80d5d7a77ga4a4bbf2a30c63c078dca/services/b7be5e40fd194258896fb602c1858eaf/execute",
                        "apiKey": null,
                        "inputs": {
                            "name": "input1",
                            "columnNames": [{
                                "name": "tweet",
                                "dataType": "string",
                                "mapTo": 0
                            }]
                        },
                        "outputs": [{
                            "name": "Sentiment",
                            "dataType": "string"
                        }],
                        "batchSize": 10
                    }
                }
            }
        }
    }
```

## <a name="implement-stream-analytics-transformation-to-call-the-udf"></a>Реализация преобразования Stream Analytics для вызова определяемой пользователем функции
Теперь выполните запрос к определяемой пользователем функции (здесь с именем scoreTweet) для каждого входного события и запишите ответ для этого события в вывод.

```json
    {
        "name": "transformation",
        "properties": {
            "streamingUnits": null,
            "query": "select *,scoreTweet(Tweet) TweetSentiment into blobOutput from blobInput"
        }
    }
```


## <a name="get-help"></a>Получить справку
Для получения дополнительной помощи воспользуйтесь [страницей вопросов об Azure Stream Analytics на сайте Microsoft](/answers/topics/azure-stream-analytics.html).

## <a name="next-steps"></a>Дальнейшие действия
* [Введение в Azure Stream Analytics](stream-analytics-introduction.md)
* [Приступая к работе с Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Масштабирование заданий в службе Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Справочник по языку запросов Azure Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference)
* [Справочник по API-интерфейсу REST управления Stream Analytics](/rest/api/streamanalytics/)
