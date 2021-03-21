---
title: DatetimeV2 предварительно построенные сущности — LUIS
titleSuffix: Azure Cognitive Services
description: В этой статье описана готовая сущность datetimeV2 в службе "Распознавание речи" (LUIS).
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 04/13/2020
ms.openlocfilehash: 83522de9c00056a3808b002b3103f45c72553399
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96013077"
---
# <a name="datetimev2-prebuilt-entity-for-a-luis-app"></a>Предварительно созданная сущность datetimeV2 для приложения LUIS

Предварительно созданная сущность **datetimeV2** извлекает значения даты и времени. Эти значения имеют стандартный формат, за счет чего их могут использовать клиентские программы. Если во фразе указана неполная дата или время, LUIS возвращает в ответе конечной точки _прошедшие и будущие значения_. Так как эта сущность уже обучена, добавлять примеры фраз, содержащие сущность datetimeV2 в приложение, не нужно.

## <a name="types-of-datetimev2"></a>Типы сущностей datetimeV2
Управление DatetimeV2 осуществляется из репозитория [распознавания текста](https://github.com/Microsoft/Recognizers-Text/blob/master/Patterns/English/English-DateTime.yaml) GitHub.

## <a name="example-json"></a>Пример JSON

Ниже показана следующая utterance и его частичный ответ JSON.

`8am on may 2nd 2019`

#### <a name="v3-response"></a>[V3 ответ](#tab/1-1)

```json
"entities": {
    "datetimeV2": [
        {
            "type": "datetime",
            "values": [
                {
                    "timex": "2019-05-02T08",
                    "resolution": [
                        {
                            "value": "2019-05-02 08:00:00"
                        }
                    ]
                }
            ]
        }
    ]
}
```

#### <a name="v3-verbose-response"></a>[V3 подробный ответ](#tab/1-2)

```json

"entities": {
    "datetimeV2": [
        {
            "type": "datetime",
            "values": [
                {
                    "timex": "2019-05-02T08",
                    "resolution": [
                        {
                            "value": "2019-05-02 08:00:00"
                        }
                    ]
                }
            ]
        }
    ],
    "$instance": {
        "datetimeV2": [
            {
                "type": "builtin.datetimeV2.datetime",
                "text": "8am on may 2nd 2019",
                "startIndex": 0,
                "length": 19,
                "modelTypeId": 2,
                "modelType": "Prebuilt Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ]
    }
}
```

#### <a name="v2-response"></a>[Ответ v2](#tab/1-3)

```json
"entities": [
  {
    "entity": "8am on may 2nd 2019",
    "type": "builtin.datetimeV2.datetime",
    "startIndex": 0,
    "endIndex": 18,
    "resolution": {
      "values": [
        {
          "timex": "2019-05-02T08",
          "type": "datetime",
          "value": "2019-05-02 08:00:00"
        }
      ]
    }
  }
]
 ```

|Имя свойства |Тип и описание свойства|
|---|---|
|Сущность|**Строка.** Текст, извлеченный из фразы, с типом даты, времени, диапазоном дат или времени.|
|тип|**Строка.** Один из [подтипов сущности datetimeV2](#subtypes-of-datetimev2).
|startIndex|**Целое число.** Индекс во фразе, в которой начинается сущность.|
|endIndex|**Целое число.** Индекс во фразе, в которой заканчивается сущность.|
|разрешение|Имеет массив `values` с одним, двумя или четырьмя [значениями разрешения](#values-of-resolution).|
|end|Конечное значение времени или диапазона дат в том же формате, что и значение (`value`). Используется, только если свойство `type` имеет значение `daterange`, `timerange` или `datetimerange`|

* * *

## <a name="subtypes-of-datetimev2"></a>Подтипы сущности datetimeV2

Предварительно созданная сущность **datetimeV2** имеет следующие подтипы (примеры каждого из них приведены в таблице ниже):
* `date`
* `time`
* `daterange`
* `timerange`
* `datetimerange`


## <a name="values-of-resolution"></a>Значения разрешения
* Массив содержит один элемент, если дата или время во фразе полностью указаны и однозначны.
* Массив содержит два элемента, если значение datetimeV2 неоднозначное. Неоднозначность означает отсутствие года, времени и диапазона времени. Примеры см. в разделе [Неоднозначные даты](#ambiguous-dates). Если неоднозначно указано время до полудня или после полудня, включаются оба значения.
* Массив содержит четыре элемента, если фраза содержит два неоднозначных элемента. К неоднозначным относятся элементы, в которых:
  * в дате или диапазоне дат указан неоднозначно год;
  * во времени или диапазоне времени указано неоднозначно время до или или после полудня (например, 3 апреля 3:00).

Ниже приведены поля, которые может иметь каждый элемент массива `values`.

|Имя свойства|Описание свойства|
|--|--|
|timex|Время, дата или диапазон дат, выраженные в формате TIMEX, который соответствует [стандарту ISO 8601](https://en.wikipedia.org/wiki/ISO_8601), и в атрибутах TIMEX3 заметки с помощью языка TimeML.|
|mod|термин, используемый для описания использования значения `before` , например, `after` .|
|тип|Подтип может иметь один из следующих элементов: `datetime` , `date` ,,,, `time` `daterange` `timerange` `datetimerange` , `duration` , `set` .|
|value|**Необязательный элемент.** Объект DateTime в формате гггг-мм-дд (дата), чч: мм: СС (Time) гггг-мм-дд чч: мм: СС (DateTime). Если свойство `type` имеет значение `duration`, в качестве значения указывается количество секунд (длительность). <br/> Используется, только если свойство `type` имеет значение `datetime` или `date`, `time` или `duration.|

## <a name="valid-date-values"></a>Допустимые значения даты

Сущность **datetimeV2** поддерживает даты в следующих диапазонах:

| Min | Max |
|----------|-------------|
| 1 января 1900 г.   | 31 декабря 2099 г. |

## <a name="ambiguous-dates"></a>Неоднозначные даты

Если дата может быть в прошлом или будущем, LUIS предоставляет оба значения. В качестве примера можно привести фразу с датой и месяцем без года.

Например, при наличии следующих utterance:

`May 2nd`

* Если сегодня 3 мая 2017 года, LUIS возвращает два значения: 2017-05-02 и 2018-05-02.
* Если сегодня 1 мая 2017 года, LUIS возвращает два значения: 2016-05-02 и 2017-05-02.

В следующем примере показано разрешение сущности "may 2nd". Это разрешение предполагает, что сегодняшняя дата — это дата между 2 мая 2017 г. и 1 мая 2018 г.
Поля с `X` в поле `timex` — это части даты, которые явно не указаны во фразе.

## <a name="date-resolution-example"></a>Пример разрешения даты


Ниже показана следующая utterance и его частичный ответ JSON.

`May 2nd`

#### <a name="v3-response"></a>[V3 ответ](#tab/2-1)

```json
"entities": {
    "datetimeV2": [
        {
            "type": "date",
            "values": [
                {
                    "timex": "XXXX-05-02",
                    "resolution": [
                        {
                            "value": "2019-05-02"
                        },
                        {
                            "value": "2020-05-02"
                        }
                    ]
                }
            ]
        }
    ]
}
```

#### <a name="v3-verbose-response"></a>[V3 подробный ответ](#tab/2-2)

```json
"entities": {
    "datetimeV2": [
        {
            "type": "date",
            "values": [
                {
                    "timex": "XXXX-05-02",
                    "resolution": [
                        {
                            "value": "2019-05-02"
                        },
                        {
                            "value": "2020-05-02"
                        }
                    ]
                }
            ]
        }
    ],
    "$instance": {
        "datetimeV2": [
            {
                "type": "builtin.datetimeV2.date",
                "text": "May 2nd",
                "startIndex": 0,
                "length": 7,
                "modelTypeId": 2,
                "modelType": "Prebuilt Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ]
    }
}
```

#### <a name="v2-response"></a>[Ответ v2](#tab/2-3)

```json
  "entities": [
    {
      "entity": "may 2nd",
      "type": "builtin.datetimeV2.date",
      "startIndex": 0,
      "endIndex": 6,
      "resolution": {
        "values": [
          {
            "timex": "XXXX-05-02",
            "type": "date",
            "value": "2019-05-02"
          },
          {
            "timex": "XXXX-05-02",
            "type": "date",
            "value": "2020-05-02"
          }
        ]
      }
    }
  ]
```
* * *

## <a name="date-range-resolution-examples-for-numeric-date"></a>Примеры разрешения диапазона числовых дат

Сущность `datetimeV2` извлекает диапазоны дат и времени. Поля `start` и `end` определяют начало и конец диапазона. Для utterance `May 2nd to May 5th` Luis предоставляет значения **daterange** как для текущего года, так и для следующего года. В поле `timex` значения `XXXX` указывают неоднозначность года. Значение `P3D` указывает на трехдневный период времени.

Ниже показана следующая utterance и его частичный ответ JSON.

`May 2nd to May 5th`

#### <a name="v3-response"></a>[V3 ответ](#tab/3-1)

```json

"entities": {
    "datetimeV2": [
        {
            "type": "daterange",
            "values": [
                {
                    "timex": "(XXXX-05-02,XXXX-05-05,P3D)",
                    "resolution": [
                        {
                            "start": "2019-05-02",
                            "end": "2019-05-05"
                        },
                        {
                            "start": "2020-05-02",
                            "end": "2020-05-05"
                        }
                    ]
                }
            ]
        }
    ]
}
```


#### <a name="v3-verbose-response"></a>[V3 подробный ответ](#tab/3-2)

```json

"entities": {
    "datetimeV2": [
        {
            "type": "daterange",
            "values": [
                {
                    "timex": "(XXXX-05-02,XXXX-05-05,P3D)",
                    "resolution": [
                        {
                            "start": "2019-05-02",
                            "end": "2019-05-05"
                        },
                        {
                            "start": "2020-05-02",
                            "end": "2020-05-05"
                        }
                    ]
                }
            ]
        }
    ],
    "$instance": {
        "datetimeV2": [
            {
                "type": "builtin.datetimeV2.daterange",
                "text": "May 2nd to May 5th",
                "startIndex": 0,
                "length": 18,
                "modelTypeId": 2,
                "modelType": "Prebuilt Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ]
    }
}
```

#### <a name="v2-response"></a>[Ответ v2](#tab/3-3)

```json
"entities": [
    {
      "entity": "may 2nd to may 5th",
      "type": "builtin.datetimeV2.daterange",
      "startIndex": 0,
      "endIndex": 17,
      "resolution": {
        "values": [
          {
            "timex": "(XXXX-05-02,XXXX-05-05,P3D)",
            "type": "daterange",
            "start": "2019-05-02",
            "end": "2019-05-05"
          }
        ]
      }
    }
  ]
```
* * *

## <a name="date-range-resolution-examples-for-day-of-week"></a>Примеры разрешения диапазона дат для дня недели

В следующем примере показано, как LUIS использует **datetimeV2** для разрешения utterance `Tuesday to Thursday` . В этом примере ниже текущей датой является 19 июня. LUIS включает значения **daterange** обоих диапазонов дат перед и после текущей даты.

Ниже показана следующая utterance и его частичный ответ JSON.

`Tuesday to Thursday`

#### <a name="v3-response"></a>[V3 ответ](#tab/4-1)

```json
"entities": {
    "datetimeV2": [
        {
            "type": "daterange",
            "values": [
                {
                    "timex": "(XXXX-WXX-2,XXXX-WXX-4,P2D)",
                    "resolution": [
                        {
                            "start": "2019-10-08",
                            "end": "2019-10-10"
                        },
                        {
                            "start": "2019-10-15",
                            "end": "2019-10-17"
                        }
                    ]
                }
            ]
        }
    ]
}
```

#### <a name="v3-verbose-response"></a>[V3 подробный ответ](#tab/4-2)

```json
"entities": {
    "datetimeV2": [
        {
            "type": "daterange",
            "values": [
                {
                    "timex": "(XXXX-WXX-2,XXXX-WXX-4,P2D)",
                    "resolution": [
                        {
                            "start": "2019-10-08",
                            "end": "2019-10-10"
                        },
                        {
                            "start": "2019-10-15",
                            "end": "2019-10-17"
                        }
                    ]
                }
            ]
        }
    ],
    "$instance": {
        "datetimeV2": [
            {
                "type": "builtin.datetimeV2.daterange",
                "text": "Tuesday to Thursday",
                "startIndex": 0,
                "length": 19,
                "modelTypeId": 2,
                "modelType": "Prebuilt Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ]
    }
}
```

#### <a name="v2-response"></a>[Ответ v2](#tab/4-3)

```json
  "entities": [
    {
      "entity": "tuesday to thursday",
      "type": "builtin.datetimeV2.daterange",
      "startIndex": 0,
      "endIndex": 19,
      "resolution": {
        "values": [
          {
            "timex": "(XXXX-WXX-2,XXXX-WXX-4,P2D)",
            "type": "daterange",
            "start": "2019-04-30",
            "end": "2019-05-02"
          }
        ]
      }
    }
  ]
```
* * *

## <a name="ambiguous-time"></a>Неоднозначное время
Массив значений имеет два элемента времени, если время или диапазон времени неоднозначны. Если время указано неоднозначно, возвращается значение до и после полудня .

## <a name="time-range-resolution-example"></a>Пример разрешения диапазона времени

В API V3 изменен ответ DatetimeV2 JSON. В следующем примере показано, как с помощью сущности **datetimeV2** LUIS обрабатывает фразу с диапазон времени.

Изменения из API версии 2:
* `datetimeV2.timex.type` свойство больше не возвращается, так как оно возвращается на родительском уровне, `datetimev2.type` .
* `datetimeV2.value`Свойство было переименовано в `datetimeV2.timex` .

Ниже показана следующая utterance и его частичный ответ JSON.

`from 6pm to 7pm`

#### <a name="v3-response"></a>[V3 ответ](#tab/5-1)

Следующий код JSON имеет параметр со `verbose` значением `false` :

```JSON

"entities": {
    "datetimeV2": [
        {
            "type": "timerange",
            "values": [
                {
                    "timex": "(T18,T19,PT1H)",
                    "resolution": [
                        {
                            "start": "18:00:00",
                            "end": "19:00:00"
                        }
                    ]
                }
            ]
        }
    ]
}
```
#### <a name="v3-verbose-response"></a>[V3 подробный ответ](#tab/5-2)

Следующий код JSON имеет параметр со `verbose` значением `true` :

```json

"entities": {
    "datetimeV2": [
        {
            "type": "timerange",
            "values": [
                {
                    "timex": "(T18,T19,PT1H)",
                    "resolution": [
                        {
                            "start": "18:00:00",
                            "end": "19:00:00"
                        }
                    ]
                }
            ]
        }
    ],
    "$instance": {
        "datetimeV2": [
            {
                "type": "builtin.datetimeV2.timerange",
                "text": "from 6pm to 7pm",
                "startIndex": 0,
                "length": 15,
                "modelTypeId": 2,
                "modelType": "Prebuilt Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ]
    }
}
```
#### <a name="v2-response"></a>[Ответ v2](#tab/5-3)

```json
  "entities": [
    {
      "entity": "6pm to 7pm",
      "type": "builtin.datetimeV2.timerange",
      "startIndex": 0,
      "endIndex": 9,
      "resolution": {
        "values": [
          {
            "timex": "(T18,T19,PT1H)",
            "type": "timerange",
            "start": "18:00:00",
            "end": "19:00:00"
          }
        ]
      }
    }
  ]
```

* * *

## <a name="time-resolution-example"></a>Пример разрешения времени

Ниже показана следующая utterance и его частичный ответ JSON.

`8am`

#### <a name="v3-response"></a>[V3 ответ](#tab/6-1)

```json
"entities": {
    "datetimeV2": [
        {
            "type": "time",
            "values": [
                {
                    "timex": "T08",
                    "resolution": [
                        {
                            "value": "08:00:00"
                        }
                    ]
                }
            ]
        }
    ]
}
```
#### <a name="v3-verbose-response"></a>[V3 подробный ответ](#tab/6-2)

```json
"entities": {
    "datetimeV2": [
        {
            "type": "time",
            "values": [
                {
                    "timex": "T08",
                    "resolution": [
                        {
                            "value": "08:00:00"
                        }
                    ]
                }
            ]
        }
    ],
    "$instance": {
        "datetimeV2": [
            {
                "type": "builtin.datetimeV2.time",
                "text": "8am",
                "startIndex": 0,
                "length": 3,
                "modelTypeId": 2,
                "modelType": "Prebuilt Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ]
    }
}
```
#### <a name="v2-response"></a>[Ответ v2](#tab/6-3)

```json
"entities": [
  {
    "entity": "8am",
    "type": "builtin.datetimeV2.time",
    "startIndex": 0,
    "endIndex": 2,
    "resolution": {
      "values": [
        {
          "timex": "T08",
          "type": "time",
          "value": "08:00:00"
        }
      ]
    }
  }
]
```

* * *

## <a name="deprecated-prebuilt-datetime"></a>Устаревшая предварительно созданная сущность datetime

Устаревшая предварительно созданная сущность `datetime` заменяется сущностью **datetimeV2**.

Чтобы заменить сущность `datetime` на `datetimeV2` в приложении LUIS, сделайте следующее:

1. Откройте область **Сущности** веб-интерфейса LUIS.
2. Удалите предварительно созданную сущность **datetime**.
3. Щелкните **Add prebuilt entity** (Добавить предварительно созданную сущность).
4. Выберите **datetimeV2** и нажмите кнопку **Сохранить**.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о [конечной точке прогнозирования V3](luis-migration-api-v3.md).

Узнайте больше о сущностях [dimension](luis-reference-prebuilt-dimension.md), [email](luis-reference-prebuilt-email.md) и [number](luis-reference-prebuilt-number.md).

