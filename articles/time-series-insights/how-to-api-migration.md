---
title: Переход на новые версии API службы "аналитика временных рядов Azure Gen2" | Документация Майкрософт
description: Как обновить среды Gen2 временных рядов Azure для использования новых общедоступных версий.
ms.service: time-series-insights
services: time-series-insights
author: shreyasharmamsft
ms.author: shresha
manager: dpalled
ms.workload: big-data
ms.topic: conceptual
ms.date: 10/01/2020
ms.custom: shresha
ms.openlocfilehash: 15f1a814b302611029cf6459b8d2df93a32a2d36
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97740561"
---
# <a name="migrating-to-new-azure-time-series-insights-gen2-api-versions"></a>Переход на новые версии API службы "аналитика временных рядов Azure Gen2"

## <a name="overview"></a>Обзор

Если вы создали среду Gen2 "аналитика временных рядов Azure" в общедоступной предварительной версии (до 16 июля, 2020), обновите среду TSI, чтобы использовать новые общедоступные версии API, выполнив действия, описанные в этой статье. Это изменение не влияет на пользователей, которые используют Gen1 версию службы "аналитика временных рядов Azure".

> [!IMPORTANT]
> Обновления, описанные в этой статье, будут обновлять только версии API, используемые средой TSI. Это изменение не связано с новыми [правилами плоской обработки и экранирования JSON,](./concepts-json-flattening-escaping-rules.md) появившимися в средах Gen2.

Новая версия API — `2020-07-31` и использует обновленный [синтаксис выражения временных рядов](/rest/api/time-series-insights/reference-time-series-expression-syntax).

Пользователи должны переносить [переменные модели временных рядов](./concepts-variables.md), сохраненные запросы, Power BI запросы, а также любые пользовательские инструменты, осуществляющие вызовы к конечным точкам API. Если у вас возникнут вопросы или проблемы, связанные с процессом миграции, отправьте запрос в службу поддержки через портал Azure и сослаться на этот документ.

> [!IMPORTANT]
> Версия API предварительной версии `2018-11-01-preview` будет продолжать поддерживаться до 31 октября 2020 г. Перед тем как избежать сбоев в работе служб, выполните все применимые шаги этой миграции.

## <a name="migrate-time-series-model-and-saved-queries"></a>Миграция модели временных рядов и сохраненных запросов

Чтобы помочь пользователям переносить [переменные модели временных рядов](./concepts-variables.md) и сохраненные запросы, существует встроенное средство, доступное в обозревателе службы " [аналитика временных рядов Azure](https://insights.timeseries.azure.com)". Перейдите к среде, которую необходимо перенести, и выполните следующие действия. **Вы можете выполнить миграцию частично и вернуться к выполнению позже, однако ни одно из обновлений не может быть отменено.**

> [!NOTE]
> Необходимо быть участником среды, чтобы вносить изменения в модель временных рядов и сохраненные запросы. Если вы не являетесь участником, вы сможете перенести только личные сохраненные запросы. Перед продолжением проверьте [политики доступа к среде](./concepts-access-policies.md) и уровень доступа.

1. В обозревателе появится запрос на обновление синтаксиса, используемого переменными модели временных рядов и сохраненными запросами.

    [![prompt](media/api-migration/ux-prompt.png)](media/v2-update-overview/overview-one.png#lightbox)

    Если вы случайно закрыли уведомление, его можно найти на панели уведомлений.

1. Нажмите кнопку " **отобразить обновления** ", чтобы открыть средство миграции.

1. Нажмите кнопку **загрузить типы**. Поскольку при миграции будут перезаписаны текущие типы для изменения синтаксиса переменных, потребуется сохранить копию текущих типов. Средство сообщит о загрузке типов.

    [![Типы загрузки](media/api-migration/ux-migration-tool.png)](media/v2-update-overview/overview-one.png#lightbox)

1. Нажмите кнопку **обновить переменные**. Средство сообщит вам, когда были обновлены переменные.

    > [!IMPORTANT]
    > При обновлении типов пользовательские приложения, использующие более старую версию API ( `2018-11-01-preview` ), должны будут использовать новую версию API ( `2020-07-31` ) для продолжения работы. Если вы не уверены, какую версию API вы используете, обратитесь к администратору перед обновлением. Вы можете закрыть средство миграции и вернуться к этим действиям позже. Дополнительные сведения о [переносе пользовательских приложений](#migrate-custom-applications)см. в этой статье.

    [![Обновить переменные](media/api-migration/ux-migration-tool-downloaded-types.png)](media/v2-update-overview/overview-one.png#lightbox)

1. Щелкните **обновить сохраненные запросы**. Средство сообщит о том, что сохраненные запросы были обновлены.

    [![Обновление сохраненных запросов](media/api-migration/ux-migration-tool-updated-variables.png)](media/v2-update-overview/overview-one.png#lightbox)

1. Нажмите кнопку **Done**(Готово).

    [![Завершенная миграция](media/api-migration/ux-migration-tool-updated-saved-queries.png)](media/v2-update-overview/overview-one.png#lightbox)

Изучите обновленную среду, создав диаграмму с помощью нескольких вновь созданных переменных и сохраненных запросов. Если при работе с диаграммами отображается непредвиденное поведение, отправьте нам отзыв с помощью средства обратной связи в обозревателе.

## <a name="migrate-power-bi-queries"></a>Миграция Power BI запросов

Если вы создали запросы с помощью соединителя Power BI, они осуществляют вызовы к службе "аналитика временных рядов Azure" с помощью синтаксиса предварительной версии API и старого выражения временного ряда. Эти запросы будут продолжать получать данные до тех пор, пока предварительная версия API не станет устаревшей.

Чтобы обновить запросы для использования новой версии API и нового синтаксиса выражений временных рядов, необходимо повторно создать запросы из обозревателя. Узнайте больше о том [, как создавать запросы с помощью соединителя Power BI](./how-to-connect-power-bi.md).

> [!NOTE]
> Необходимо использовать по крайней мере 2020 июля версии Power BI Desktop. Если этого не сделать, может отобразиться ошибка "Недопустимая версия полезных данных запроса".

## <a name="migrate-custom-applications"></a>Миграция пользовательских приложений

Если пользовательское приложение выполняет вызовы к следующим конечным точкам RESTFUL, то достаточно обновить версию API до `2020-07-31` в URI:

- API модели временных рядов
  - API параметров модели
    - [Получить](/rest/api/time-series-insights/dataaccessgen2/modelsettings/get)
    - [Обновление](/rest/api/time-series-insights/dataaccessgen2/modelsettings/update)
  - API экземпляра
    - [Все операции пакетной обработки](/rest/api/time-series-insights/dataaccessgen2/timeseriesinstances/executebatch)
    - [Список](/rest/api/time-series-insights/dataaccessgen2/timeseriesinstances/list)
    - [Поиск](/rest/api/time-series-insights/dataaccessgen2/timeseriesinstances/search)
    - [Рекомендация](/rest/api/time-series-insights/dataaccessgen2/timeseriesinstances/suggest)
  - Интерфейсы API иерархии
    - [Все операции пакетной обработки](/rest/api/time-series-insights/dataaccessgen2/timeserieshierarchies/executebatch)
    - [Список](/rest/api/time-series-insights/dataaccessgen2/timeserieshierarchies/list)
  - Интерфейсы API типов
    - [Удаление, получение операций](/rest/api/time-series-insights/dataaccessgen2/timeseriestypes/executebatch)
    - [Список](/rest/api/time-series-insights/dataaccessgen2/timeseriestypes/list)

Для следующих конечных точек RESTFUL необходимо обновить версию API до `2020-07-31` в URI и убедиться, что все экземпляры `tsx` свойства используют обновленный [синтаксис выражения временных рядов](/rest/api/time-series-insights/reference-time-series-expression-syntax).

- Интерфейсы API типов
  - [Операция размещения](/rest/api/time-series-insights/dataaccessgen2/timeseriestypes/executebatch#typesbatchput)
- Интерфейсы API для запросов
  - [События](/rest/api/time-series-insights/dataaccessgen2/query/execute#getevents)
  - [Серия](/rest/api/time-series-insights/dataaccessgen2/query/execute#getseries)
  - [жетаггрегатесериес](/rest/api/time-series-insights/dataaccessgen2/query/execute#aggregateseries)

### <a name="examples"></a>Примеры

#### <a name="typesbatchput"></a>типесбатчпут

Старый текст запроса (используется `2018-11-01-preview` ):

```JSON
{
  "put": [
    {
      "id": "c1cb7a33-ed9b-4cf1-9958-f3162fed8ee8",
      "name": "OutdoorTemperatureSensor",
      "description": "This is an outdoor temperature sensor.",
      "variables": {
        "AverageTemperature": {
          "kind": "numeric",
          "value": {
            "tsx": "$event.[Temperature_Celsius].Double"
          },
          "filter": {
            "tsx": "$event.[Mode].String = 'outdoor'"
          },
          "aggregation": {
            "tsx": "avg($value)"
          }
        }
      }
    }
  ]
}
```

Обновленный текст запроса (используется `2020-07-31` ):

```JSON
{
  "put": [
    {
      "id": "c1cb7a33-ed9b-4cf1-9958-f3162fed8ee8",
      "name": "OutdoorTemperatureSensor",
      "description": "This is an outdoor temperature sensor.",
      "variables": {
        "AverageTemperature": {
          "kind": "numeric",
          "value": {
            "tsx": "$event['Temperature_Celsius'].Double"
          },
          "filter": {
            "tsx": "$event['Mode'].String = 'outdoor'"
          },
          "aggregation": {
            "tsx": "avg($value)"
          }
        }
      }
    }
  ]
}
```

Кроме того, `filter` может иметь значение `$event.Mode.String = 'outdoor'` . В `value` необходимо использовать квадратные скобки для экранирования специального символа ( `_` ).

#### <a name="getevents"></a>События

Старый текст запроса (используется `2018-11-01-preview` ):

```JSON
{
  "getEvents": {
    "timeSeriesId": [
      "006dfc2d-0324-4937-998c-d16f3b4f1952",
      "T1"
    ],
    "searchSpan": {
      "from": "2016-08-01T00:00:00Z",
      "to": "2016-08-01T00:16:50Z"
    },
    "filter": {
      "tsx": "($event.[Value].Double != null) OR ($event.[Status].String = 'Good')"
    },
    "projectedProperties": [
      {
        "name": "Temperature",
        "type": "Double"
      }
    ]
  }
}
```

Обновленный текст запроса (используется `2020-07-31` ):

```JSON
{
  "getEvents": {
    "timeSeriesId": [
      "006dfc2d-0324-4937-998c-d16f3b4f1952",
      "T1"
    ],
    "searchSpan": {
      "from": "2016-08-01T00:00:00Z",
      "to": "2016-08-01T00:16:50Z"
    },
    "filter": {
      "tsx": "($event.Value.Double != null) OR ($event.Status.String = 'Good')"
    },
    "projectedProperties": [
      {
        "name": "Temperature",
        "type": "Double"
      }
    ]
  }
}
```

Кроме того, `filter` может иметь значение `($event['Value'].Double != null) OR ($event['Status'].String = 'Good')` .

#### <a name="getseries"></a>Серия

Старый текст запроса (используется `2018-11-01-preview` ):

```JSON
{
  "getSeries": {
    "timeSeriesId": [
      "006dfc2d-0324-4937-998c-d16f3b4f1952"
    ],
    "searchSpan": {
      "from": "2016-08-01T00:00:00Z",
      "to": "2016-08-01T00:16:50Z"
    },
    "inlineVariables": {
      "pressure": {
        "kind": "numeric",
        "value": {
          "tsx": "$event.[Bar-Pressure-Offset]"
        },
        "aggregation": {
          "tsx": "avg($value)"
        }
      }
    },
    "projectedVariables": [
      "pressure"
    ]
  }
}
```

Обновленный текст запроса (используется `2020-07-31` ):

```JSON
{
  "getSeries": {
    "timeSeriesId": [
      "006dfc2d-0324-4937-998c-d16f3b4f1952"
    ],
    "searchSpan": {
      "from": "2016-08-01T00:00:00Z",
      "to": "2016-08-01T00:16:50Z"
    },
    "inlineVariables": {
      "pressure": {
        "kind": "numeric",
        "value": {
          "tsx": "$event['Bar-Pressure-Offset']"
        },
        "aggregation": {
          "tsx": "avg($value)"
        }
      }
    },
    "projectedVariables": [
      "pressure"
    ]
  }
}
```

Кроме того, `value` может иметь значение `$event['Bar-Pressure-Offset'].Double` . Если тип данных не указан, всегда предполагается, что тип данных равен Double. Нотация в квадратных скобках должна использоваться для экранирования специального символа ( `-` ).

#### <a name="aggregateseries"></a>аггрегатесериес

Старый текст запроса (используется `2018-11-01-preview` ):

```JSON
{
  "aggregateSeries": {
    "timeSeriesId": [
      "006dfc2d-0324-4937-998c-d16f3b4f1952"
    ],
    "searchSpan": {
      "from": "2016-08-01T00:00:00Z",
      "to": "2016-08-01T00:16:50Z"
    },
    "interval": "PT1M",
    "inlineVariables": {
      "MinTemperature": {
        "kind": "numeric",
        "value": {
          "tsx": "coalesce($event.[Temp].Double, toDouble($event.[Temp].Long))"
        },
        "aggregation": {
          "tsx": "min($value)"
        }
      },
    },
    "projectedVariables": [
      "MinTemperature"
    ]
  }
}
```

Обновленный текст запроса (используется `2020-07-31` ):

```JSON
  "aggregateSeries": {
    "timeSeriesId": [
      "006dfc2d-0324-4937-998c-d16f3b4f1952"
    ],
    "searchSpan": {
      "from": "2016-08-01T00:00:00Z",
      "to": "2016-08-01T00:16:50Z"
    },
    "interval": "PT1M",
    "inlineVariables": {
      "MinTemperature": {
        "kind": "numeric",
        "value": {
          "tsx": "coalesce($event.Temp.Double, toDouble($event.Temp.Long))"
        },
        "aggregation": {
          "tsx": "min($value)"
        }
      },
    },
    "projectedVariables": [
      "MinTemperature"
    ]
  }
}
```

Кроме того, `value` может иметь значение `coalesce($event['Temp'].Double, toDouble($event['Temp'].Long))` .

### <a name="potential-errors"></a>Потенциальные ошибки

#### <a name="invalidinput"></a>InvalidInput

Если отображается следующая ошибка, используется новая версия API ( `2020-07-31` ), но синтаксис целевого сервера не был обновлен. Ознакомьтесь с [синтаксисом выражений временных рядов](/rest/api/time-series-insights/reference-time-series-expression-syntax) и примерами миграции, приведенными выше. Перед повторной `tsx` отправкой запроса API убедитесь, что все свойства правильно обновлены.

```JSON
{
    "error": {
        "code": "InvalidInput",
        "message": "Unable to parse 'value' time series expression (TSX) in variable 'Temperature'.",
        "target": "projectedVariables.Temperature.value",
        "innerError": {
            "parseErrorDetails": [
                {
                    "pos": [
                        0,
                        5
                    ],
                    "line": 1,
                    "msg": "Unsupported Time Series Expression version TSX01 used instead of TSX00.",
                    "code": "UnsupportedTSXVersionTSX01",
                    "target": "$event"
                }
            ],
            "code": "TsxParseError"
        }
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия

- Протестируйте свою среду с помощью обозревателя службы " [аналитика временных рядов Azure](./concepts-ux-panels.md) " или пользовательского приложения.