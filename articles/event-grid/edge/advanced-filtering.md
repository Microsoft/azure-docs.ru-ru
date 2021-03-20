---
title: Расширенная фильтрация в службе "Сетка событий Azure" IoT Edge | Документация Майкрософт
description: Расширенная фильтрация в сетке событий на IoT Edge.
author: HiteshMadan
manager: rajarv
ms.author: himad
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: 64b8956c47cbdbf31bb8253dac0c1e1f12833bf7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96001054"
---
# <a name="advanced-filtering"></a>Расширенная фильтрация
Сетка событий позволяет задавать фильтры для любого свойства в полезных данных JSON. Эти фильтры моделируются в виде набора `AND` условий, при этом каждое внешнее условие имеет необязательные внутренние `OR` условия. Для каждого `AND` условия задаются следующие значения:

* `OperatorType` — Тип сравнения.
* `Key` — Путь JSON к свойству, к которому применяется фильтр.
* `Value` — Ссылочное значение, для которого выполняется фильтр (или) `Values` — набор ссылочных значений, по которым выполняется фильтр.

## <a name="json-syntax"></a>Синтаксис JSON

Синтаксис JSON для расширенного фильтра выглядит следующим образом:

```json
{
    "filter": {
        "advancedFilters": [{
                "operatorType": "NumberGreaterThanOrEquals",
                "key": "Data.Key1",
                "value": 5
            }, {
                "operatorType": "StringContains",
                "key": "Subject",
                "values": ["container1", "container2"]
            }
        ]
    }
}
```

## <a name="filtering-on-array-values"></a>Фильтрация по значениям массива

Сейчас сетка событий не поддерживает фильтрацию по массиву значений. Если входящее событие имеет значение массива для ключа расширенного фильтра, операция сопоставления завершается ошибкой. Входящее событие не соответствует подписке на события.

## <a name="and-or-not-semantics"></a>Семантика AND-AND-NOT

Обратите внимание, что в примере JSON, приведенном ранее, `AdvancedFilters` является массивом. Каждый `AdvancedFilter` элемент массива следует рассматривать как `AND` условие.

Для операторов, которые поддерживают несколько значений (таких как `NumberIn` , `NumberNotIn` , `StringIn` и т. д.), каждое значение рассматривается как `OR` условие. Таким образом, `StringBeginsWith("a", "b", "c")` значение будет соответствовать любому строковому значению, которое начинается с либо или `a` `b` `c` .

> [!CAUTION]
> Операторы NOT `NumberNotIn` и `StringNotIn` ведут себя как условия для каждого значения, заданного в `Values` поле.
>
> Если этого не сделать, фильтр будет Accept-All фильтром и отменить назначение фильтрации.

## <a name="floating-point-rounding-behavior"></a>Поведение округления с плавающей точкой

Сетка событий использует `decimal` тип .NET для управления всеми числовыми значениями. Числовые значения, указанные в JSON подписки на события, не действуют в соответствии с режимом округления с плавающей запятой.

## <a name="case-sensitivity-of-string-filters"></a>Чувствительность к регистру строковых фильтров

Все сравнения строк не учитывают регистр. Это поведение можно изменить уже сегодня.

## <a name="allowed-advanced-filter-keys"></a>Разрешенные дополнительные ключи фильтра

`Key`Свойство может быть хорошо известным свойством верхнего уровня или представлять собой путь JSON с несколькими точками, где каждая точка обозначает шаг с заходом в вложенный объект JSON.

Сетка событий не имеет особого смысла для `$` символа в ключе, в отличие от спецификации JSONPath.

### <a name="event-grid-schema"></a>Схема сетки событий

Для событий в схеме сетки событий:

* ID
* Раздел
* Субъект
* EventType
* DataVersion;
* Data. Prop1
* Data. prop * Prop2. Prop3. Prop4. Prop5

### <a name="custom-event-schema"></a>Пользовательская схема событий

`Key`В схеме пользовательского события нет ограничений, так как сетка событий не применяет схему конвертов в полезных данных.

## <a name="numeric-single-value-filter-examples"></a>Примеры однозначных фильтров с одним значением

* NumberGreaterThan;
* NumberGreaterThanOrEquals;
* NumberLessThan;
* NumberGreaterThanOrEquals;

```json
{
    "filter": {
        "advancedFilters": [
            {
                "operatorType": "NumberGreaterThan",
                "key": "Data.Key1",
                "value": 5
            },
            {
                "operatorType": "NumberGreaterThanOrEquals",
                "key": "Data.Key2",
                "value": *456
            },
            {
                "operatorType": "NumberLessThan",
                "key": "Data.P*P2.P3",
                "value": 1000
            },
            {
                "operatorType": "NumberLessThanOrEquals",
                "key": "Data.P*P2",
                "value": 999
            }
        ]
    }
}
```

## <a name="numeric-range-value-filter-examples"></a>Числовые примеры фильтра значений диапазона

* NumberIn;
* NumberNotIn.

```json
{
    "filter": {
        "advancedFilters": [
            {
                "operatorType": "NumberIn",
                "key": "Data.Key1",
                "values": [1, 10, 100]
            },
            {
                "operatorType": "NumberNotIn",
                "key": "Data.Key2",
                "values": [2, 3, 4.56]
            }
        ]
    }
}
```

## <a name="string-range-value-filter-examples"></a>Примеры фильтров значений строкового диапазона

* StringContains;
* StringBeginsWith;
* StringEndsWith;
* StringIn;
* StringNotIn.

```json
{
    "filter": {
        "advancedFilters": [
            {
                "operatorType": "StringContains",
                "key": "Data.Key1",
                "values": ["microsoft", "azure"]
            },
            {
                "operatorType": "StringBeginsWith",
                "key": "Data.Key2",
                "values": ["event", "grid"]
            },
            {
                "operatorType": "StringEndsWith",
                "key": "Data.P3.P4",
                "values": ["jpg", "jpeg", "png"]
            },
            {
                "operatorType": "StringIn",
                "key": "RootKey",
                "values": ["exact", "string", "matches"]
            },
            {
                "operatorType": "StringNotIn",
                "key": "RootKey",
                "values": ["aws", "bridge"]
            }
        ]
    }
}
```

## <a name="boolean-single-value-filter-examples"></a>Примеры логических фильтров с одним значением

* BoolEquals

```json
{
    "filter": {
        "advancedFilters": [
            {
                "operatorType": "BoolEquals",
                "key": "BoolKey1",
                "value": true
            },
            {
                "operatorType": "BoolEquals",
                "key": "BoolKey2",
                "value": false
            }
        ]
    }
}
```
