---
title: Настраиваемые правила на основе пороговых значений в Azure Stream Analytics
description: В этой статье описывается использование эталонных данных для получения решения для оповещения с правилами на основе настраиваемых порогов в Azure Stream Analytics.
author: enkrumah
ms.author: ebnkruma
ms.service: stream-analytics
ms.topic: how-to
ms.date: 04/30/2018
ms.openlocfilehash: 2f9d132084f0254486be533daea6b54239f4e450
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98019981"
---
# <a name="process-configurable-threshold-based-rules-in-azure-stream-analytics"></a>Обработка правил на основе настраиваемых порогов в Azure Stream Analytics
В этой статье описывается использование эталонных данных для получения решения для оповещения, использующего правила на основе настраиваемых порогов в Azure Stream Analytics.

## <a name="scenario-alerting-based-on-adjustable-rule-thresholds"></a>Сценарий: оповещение на основе правил с настраиваемыми порогами
Вам может потребоваться получать оповещения, когда входящие передаваемые потоком события достигают определенного значения или сводное значение на основе входящих передаваемых потоком событий превышает определенный порог. Несложно настроить запрос Stream Analytics, который сравнивает значение со статическим порогом, который зафиксирован и предопределен. Фиксированный порог может быть жестко запрограммирован в синтаксической конструкции запроса потоковой передачи с помощью простых арифметических операторов сравнения (больше, меньше и равно).

В некоторых случаях значения порогов должны легко настраиваться без необходимости редактировать синтаксические конструкции запросов каждый раз, когда значение порога изменяется. В других случаях может потребоваться обрабатывать одним и тем же запросом множество устройств или пользователей с разными значениями порогов для каждого типа устройства. 

Этот шаблон можно использовать для динамической настройки порогов, выбирая тип устройства, к которому применяется порог, и фильтруя входные данные, а также выбирая поля, включаемые в выходные данные.

## <a name="recommended-design-pattern"></a>Рекомендуемый конструктивный шаблон
Используйте входные эталонные данные в задании Stream Analytics при поиске порогов оповещений.
- Храните значения порогов в эталонных данных (одно значение на ключ).
- Присоедините события входных потоковых данных к эталонным данным по ключевому столбцу.
- Используйте значение с ключом из эталонных данных в качестве значения порога.

## <a name="example-data-and-query"></a>Пример данных и запроса
В этом примере оповещения генерируются, когда результат статистической обработки данных потоковой передачи с устройств на минутном интервале соответствует значениям в правиле, предоставленном в качестве эталонных данных.

В запросе для каждого deviceId и каждого metricName для deviceId можно настроить от 0 до 5 измерений для операции GROUP BY. Группируются только события с соответствующими значениями фильтра. После группирования вычисляются статистические выражения Min, Max и Avg для 60-секундного "переворачивающегося окна". Затем для создания выходного события оповещения вычисляются фильтры по статистическим значениям в соответствии с настроенным порогом в эталонных данных.

Предположим, что имеется задание Stream Analytics с входными эталонными данными **rules** и входными потоковыми данными **metrics**. 

## <a name="reference-data"></a>Ссылочные данные
В этом примере эталонных данных показано, как можно представить правило на основе порогов. JSON-файл содержит эталонные данные и сохраняется в хранилище BLOB-объектов Azure, и этот контейнер хранилища BLOB-объектов используется в качестве входных эталонных данных **rules**. Со временем можно будет перезаписать этот JSON-файл и заменить конфигурацию правила, не останавливая или запуская задание потоковой передачи.

- Пример правила представляет настраиваемое оповещение о ситуации, когда загрузка ЦП (среднее значение больше или равно) превышает `90` процентов. При необходимости можно настроить поле `value`.
- Обратите внимание на то, в правиле имеется поле **operator**, которое позже динамически интерпретируется в синтаксической конструкции запроса на основе `AVGGREATEROREQUAL`. 
- Правило фильтрует данные по определенному ключу измерения `2` со значением `C1`. Остальные поля являются пустыми строками, то есть входной поток не фильтруется по этим полям событий. При необходимости можно настроить дополнительные правила ЦП для фильтрации по другим полям.
- Не все столбцы будут включены в выходное событие оповещения. В этом случае для ключа `includedDim` номер `2` задано значение `TRUE`, чтобы выходные события содержали поле номер 2 данных событий из потока. Другие поля не будут включены в выходное оповещение, однако список полей можно изменить.


```json
{
    "ruleId": 1234, 
    "deviceId" : "978648", 
    "metricName": "CPU", 
    "alertName": "hot node AVG CPU over 90",
    "operator" : "AVGGREATEROREQUAL",
    "value": 90, 
    "includeDim": {
        "0": "FALSE", 
        "1": "FALSE", 
        "2": "TRUE", 
        "3": "FALSE", 
        "4": "FALSE"
    },
    "filter": {
        "0": "", 
        "1": "",
        "2": "C1", 
        "3": "", 
        "4": ""
    }    
}
```

## <a name="example-streaming-query"></a>Пример запроса потоковых данных
Этот пример запроса Stream Analytics объединяет эталонные данные **rules** из примера выше с входным потоком данных **metrics**.

```sql
WITH transformedInput AS
(
    SELECT
        dim0 = CASE rules.includeDim.[0] WHEN 'TRUE' THEN metrics.custom.dimensions.[0].value ELSE NULL END,
        dim1 = CASE rules.includeDim.[1] WHEN 'TRUE' THEN metrics.custom.dimensions.[1].value ELSE NULL END,
        dim2 = CASE rules.includeDim.[2] WHEN 'TRUE' THEN metrics.custom.dimensions.[2].value ELSE NULL END,
        dim3 = CASE rules.includeDim.[3] WHEN 'TRUE' THEN metrics.custom.dimensions.[3].value ELSE NULL END,
        dim4 = CASE rules.includeDim.[4] WHEN 'TRUE' THEN metrics.custom.dimensions.[4].value ELSE NULL END,
        metric = metrics.metric.value,
        metricName = metrics.metric.name,
        deviceId = rules.deviceId, 
        ruleId = rules.ruleId, 
        alertName = rules.alertName,
        ruleOperator = rules.operator, 
        ruleValue = rules.value
    FROM 
        metrics
        timestamp by eventTime
    JOIN 
        rules
        ON metrics.deviceId = rules.deviceId AND metrics.metric.name = rules.metricName
    WHERE
        (rules.filter.[0] = '' OR metrics.custom.filters.[0].value = rules.filter.[0]) AND 
        (rules.filter.[1] = '' OR metrics.custom.filters.[1].value = rules.filter.[1]) AND
        (rules.filter.[2] = '' OR metrics.custom.filters.[2].value = rules.filter.[2]) AND
        (rules.filter.[3] = '' OR metrics.custom.filters.[3].value = rules.filter.[3]) AND
        (rules.filter.[4] = '' OR metrics.custom.filters.[4].value = rules.filter.[4])
)

SELECT
    System.Timestamp as time, 
    transformedInput.deviceId as deviceId,
    transformedInput.ruleId as ruleId,
    transformedInput.metricName as metric,
    transformedInput.alertName as alert,
    AVG(metric) as avg,
    MIN(metric) as min, 
    MAX(metric) as max, 
    dim0, dim1, dim2, dim3, dim4
FROM
    transformedInput
GROUP BY
    transformedInput.deviceId,
    transformedInput.ruleId,
    transformedInput.metricName,
    transformedInput.alertName,
    dim0, dim1, dim2, dim3, dim4,
    ruleOperator, 
    ruleValue, 
    TumblingWindow(second, 60)
HAVING
    (
        (ruleOperator = 'AVGGREATEROREQUAL' AND avg(metric) >= ruleValue) OR
        (ruleOperator = 'AVGEQUALORLESS' AND avg(metric) <= ruleValue) 
    )
```

## <a name="example-streaming-input-event-data"></a>Пример входных потоковых данных события
Этот пример данных JSON представляет входные данные **metrics**, используемые в запросе потоковых данных, приведенном выше. 

- На минутном временном диапазоне отображены три примера событий со значением `T14:50`. 
- У всех трех событий `deviceId` имеет значение `978648`.
- Значения метрики ЦП отличаются в каждом событии: `98`, `95` и `80` соответственно. Только в первых двух примерах событий превышен порог правила генерации оповещений для ЦП, установленный в правиле.
- Поле includeDim в правиле генерации оповещений было ключом номер 2. Соответствующее поле ключа 2 в примере события называется `NodeName`. Три примера событий имеют значения `N024`, `N024` и `N014` соответственно. В выходных данных отображается только узел `N024`, так как это единственные данные, соответствующие условиям оповещения о высокой загрузке ЦП. `N014` не превышает порог высокой загрузки ЦП.
- В правиле генерации оповещений настроен оператор `filter` только по ключу номер 2, который соответствует полю `cluster` в примерах событий. Все три примера событий имеют значение `C1` и соответствуют условиям фильтра.

```json
{
    "eventTime": "2018-04-30T14:50:23.1324132Z",
    "deviceId": "978648",
    "custom": {
        "dimensions": {
            "0": {
                "name": "NodeType",
                "value": "N1"
            },
            "1": {
                "name": "Cluster",
                "value": "C1"
            },
            "2": {
                "name": "NodeName",
                "value": "N024"
            }
        },
        "filters": {
            "0": {
                "name": "application",
                "value": "A1"
            },
            "1": {
                "name": "deviceType",
                "value": "T1"
            },
            "2": {
                "name": "cluster",
                "value": "C1"
            },
            "3": {
                "name": "nodeType",
                "value": "N1"
            }
        }
    },
    "metric": {
        "name": "CPU",
        "value": 98,
        "count": 1.0,
        "min": 98,
        "max": 98,
        "stdDev": 0.0
    }
}
{
    "eventTime": "2018-04-30T14:50:24.1324138Z",
    "deviceId": "978648",
    "custom": {
        "dimensions": {
            "0": {
                "name": "NodeType",
                "value": "N2"
            },
            "1": {
                "name": "Cluster",
                "value": "C1"
            },
            "2": {
                "name": "NodeName",
                "value": "N024"
            }
        },
        "filters": {
            "0": {
                "name": "application",
                "value": "A1"
            },
            "1": {
                "name": "deviceType",
                "value": "T1"
            },
            "2": {
                "name": "cluster",
                "value": "C1"
            },
            "3": {
                "name": "nodeType",
                "value": "N2"
            }
        }
    },
    "metric": {
        "name": "CPU",
        "value": 95,
        "count": 1,
        "min": 95,
        "max": 95,
        "stdDev": 0
    }
}
{
    "eventTime": "2018-04-30T14:50:37.1324130Z",
    "deviceId": "978648",
    "custom": {
        "dimensions": {
            "0": {
                "name": "NodeType",
                "value": "N3"
            },
            "1": {
                "name": "Cluster",
                "value": "C1 "
            },
            "2": {
                "name": "NodeName",
                "value": "N014"
            }
        },
        "filters": {
            "0": {
                "name": "application",
                "value": "A1"
            },
            "1": {
                "name": "deviceType",
                "value": "T1"
            },
            "2": {
                "name": "cluster",
                "value": "C1"
            },
            "3": {
                "name": "nodeType",
                "value": "N3"
            }
        }
    },
    "metric": {
        "name": "CPU",
        "value": 80,
        "count": 1,
        "min": 80,
        "max": 80,
        "stdDev": 0
    }
}
```

## <a name="example-output"></a>Пример выходных данных
В этом примере выходных данных JSON показано отдельное событие оповещения, которые было сгенерировано на основе правила порога ЦП, определенного в эталонных данных. Выходное событие содержит имя оповещения, а также результаты статистических выражений (average, min и max) для рассматриваемых полей. Данные выходного события содержат значение `N024` ключа поля номер 2 `NodeName` ввиду конфигурации правила. (Данные JSON были изменены для отображения разрывов строк для удобства чтения.)

```JSON
{"time":"2018-05-01T02:03:00.0000000Z","deviceid":"978648","ruleid":1234,"metric":"CPU",
"alert":"hot node AVG CPU over 90","avg":96.5,"min":95.0,"max":98.0,
"dim0":null,"dim1":null,"dim2":"N024","dim3":null,"dim4":null}
```
