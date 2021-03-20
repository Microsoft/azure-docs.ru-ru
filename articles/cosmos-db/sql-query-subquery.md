---
title: Вложенные запросы SQL для Azure Cosmos DB
description: Сведения о вложенных запросах SQL и их типичных сценариях использования и различных типах вложенных запросов в Azure Cosmos DB
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 12/02/2019
ms.author: tisande
ms.openlocfilehash: f5f209229d17a2587258d21ee90e7560e629d082
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93340861"
---
# <a name="sql-subquery-examples-for-azure-cosmos-db"></a>Примеры вложенных запросов SQL для Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Вложенный запрос — это запрос, расположенный в другом запросе. Вложенный запрос также называется внутренним запросом или внутренней выборкой. Инструкция, содержащая вложенный запрос, обычно называется внешним запросом.

В этой статье описываются вложенные запросы SQL и их распространенные варианты использования в Azure Cosmos DB. Все примеры запросов в этом документе можно выполнить для набора данных информации питании, который предварительно загружен на [Площадка для тестирования запросов Azure Cosmos DB](https://www.documentdb.com/sql/demo).

## <a name="types-of-subqueries"></a>Типы вложенных запросов

Существует два основных типа вложенных запросов:

* **Коррелированный**: вложенный запрос, который ссылается на значения из внешнего запроса. Вложенный запрос вычисляется один раз для каждой строки, обрабатываемой внешним запросом.
* **Некоррелированный**: вложенный запрос, который не зависит от внешнего запроса. Его можно выполнять самостоятельно, не полагаясь на внешний запрос.

> [!NOTE]
> Azure Cosmos DB поддерживает только коррелированные вложенные запросы.

Вложенные запросы можно дополнительно классифицировать на основе числа возвращаемых строк и столбцов. Здесь возможны три варианта:
* **Table**: возвращает несколько строк и несколько столбцов.
* **Множественное значение**: возвращает несколько строк и один столбец.
* **Scalar**: Возвращает одну строку и один столбец.

SQL-запросы в Azure Cosmos DB всегда возвращают один столбец (простое значение или сложный документ). Таким образом, в Azure Cosmos DB применимы только многозначные и скалярные вложенные запросы. Вложенный запрос с несколькими значениями можно использовать только в предложении FROM в качестве реляционного выражения. Скалярный вложенный запрос можно использовать в качестве скалярного выражения в предложении SELECT или WHERE или в качестве реляционного выражения в предложении FROM.

## <a name="multi-value-subqueries"></a>Вложенные запросы с несколькими значениями

Вложенные запросы с несколькими значениями возвращают набор документов и всегда используются в предложении FROM. Они используются для:

* Оптимизация выражений соединений. 
* Оценка дорогостоящих выражений один раз и многократная ссылка.

## <a name="optimize-join-expressions"></a>Выражения оптимизации соединений

Вложенные запросы с несколькими значениями могут оптимизировать выражения объединения, помещая предикаты после каждого выражения SELECT-many, а не после всех перекрестных соединений в предложении WHERE.

Обратите внимание на следующий запрос:

```sql
SELECT Count(1) AS Count
FROM c
JOIN t IN c.tags
JOIN n IN c.nutrients
JOIN s IN c.servings
WHERE t.name = 'infant formula' AND (n.nutritionValue > 0 
AND n.nutritionValue < 10) AND s.amount > 1
```

Для этого запроса индекс будет соответствовать любому документу с тегом с именем "Формула новорожденный". Это элемент нутриент со значением от 0 до 10 и обслуживающим элементом с суммой больше 1. Выражение JOIN здесь выполняет перекрестное произведение всех элементов тегов, нутриентс и обслуживает массивы для каждого соответствующего документа до применения любого фильтра. 

Затем предложение WHERE применит предикат фильтра для каждого <ного кортежа c, t, n, s>. Например, если в соответствующем документе в каждом из трех массивов было 10 элементов, оно будет расширено до 1 x 10 x 10 x 10 (то есть 1 000) кортежей. С помощью вложенных запросов можно фильтровать соединяемые элементы массива перед присоединением к следующему выражению.

Этот запрос эквивалентен предыдущему, но использует вложенные запросы:

```sql
SELECT Count(1) AS Count
FROM c
JOIN (SELECT VALUE t FROM t IN c.tags WHERE t.name = 'infant formula')
JOIN (SELECT VALUE n FROM n IN c.nutrients WHERE n.nutritionValue > 0 AND n.nutritionValue < 10)
JOIN (SELECT VALUE s FROM s IN c.servings WHERE s.amount > 1)
```

Предположим, что только один элемент в массиве Tags соответствует фильтру, а для нутриентс и обслуживания массивов существует пять элементов. Затем выражения объединения разворачиваются до 1 x 1 x 5 x 5 = 25 элементов, а не 1 000 элементов в первом запросе.

## <a name="evaluate-once-and-reference-many-times"></a>Многократное вычисление и ссылка

Вложенные запросы могут помочь оптимизировать запросы с дорогостоящими выражениями, такими как определяемые пользователем функции (UDF), сложные строки или арифметические выражения. Можно использовать вложенный запрос вместе с выражением объединения для вычисления выражения, но ссылаться на него много раз.

Следующий запрос дважды запускает определяемую пользователем функцию `GetMaxNutritionValue` :

```sql
SELECT c.id, udf.GetMaxNutritionValue(c.nutrients) AS MaxNutritionValue
FROM c
WHERE udf.GetMaxNutritionValue(c.nutrients) > 100
```

Ниже приведен эквивалентный запрос, запускающий UDF только один раз:

```sql
SELECT TOP 1000 c.id, MaxNutritionValue
FROM c
JOIN (SELECT VALUE udf.GetMaxNutritionValue(c.nutrients)) MaxNutritionValue
WHERE MaxNutritionValue > 100
``` 

> [!NOTE] 
> Учитывайте поведение перекрестного произведения выражений JOIN. Если выражение UDF может принимать значение undefine, следует убедиться, что выражение JOIN всегда создает одну строку, возвращая объект из вложенного запроса, а не значение напрямую.
>

Вот похожий пример, возвращающий объект, а не значение:

```sql
SELECT TOP 1000 c.id, m.MaxNutritionValue
FROM c
JOIN (SELECT udf.GetMaxNutritionValue(c.nutrients) AS MaxNutritionValue) m
WHERE m.MaxNutritionValue > 100
```

Этот подход не ограничивается UDF. Он применяется к любому потенциально дорогостоящему выражению. Например, можно использовать тот же подход с математической функцией `avg` :

```sql
SELECT TOP 1000 c.id, AvgNutritionValue
FROM c
JOIN (SELECT VALUE avg(n.nutritionValue) FROM n IN c.nutrients) AvgNutritionValue
WHERE AvgNutritionValue > 80
```

## <a name="mimic-join-with-external-reference-data"></a>Имитировать соединение с внешними эталонными данными

Часто бывает необходимо ссылаться на статические данные, которые редко изменяются, например единицы измерения или коды стран. Лучше не дублировать эти данные для каждого документа. Избежание этого дублирования позволит экономить место на хранении и повышать производительность операций записи, сохраняя размер документа. Вложенный запрос можно использовать для имитации семантики внутреннего объединения с помощью коллекции ссылочных данных.

Например, рассмотрим следующий набор ссылочных данных:

| **Единица измерения** | **имя**;            | **Множитель** | **Базовая единица** |
| -------- | ------------------- | -------------- | ------------- |
| NG       | Микрограмматика            | 1,00 e-09       | Gram          |
| μг       | микрограм           | 1,00 e-06       | Gram          |
| mg       | миллиграм           | 1,00 e-03       | Gram          |
| н        | Gram                | 1,00 e + 00       | Gram          |
| кг       | Килограмм            | 1,00 e + 03       | Gram          |
| MG       | мегаграм            | 1,00 e + 06       | Gram          |
| GG       | гигаграм            | 1,00 e + 09       | Gram          |
| nJ       | наножауле           | 1,00 e-09       | жауле         |
| μж       | микрожауле          | 1,00 e-06       | жауле         |
| mJ       | миллижауле          | 1,00 e-03       | жауле         |
| J        | жауле               | 1,00 e + 00       | жауле         |
| kJ       | киложауле           | 1,00 e + 03       | жауле         |
| MJ       | мегажауле           | 1,00 e + 06       | жауле         |
| гж       | гигажауле           | 1,00 e + 09       | жауле         |
| Cal      | калорие             | 1,00 e + 00       | калорие       |
| ккал     | калорие             | 1,00 e + 03       | калорие       |
| IU       | Международные единицы |                |               |


Следующий запрос имитирует соединение с этими данными, чтобы добавить в выходные данные имя единицы измерения:

```sql
SELECT TOP 10 n.id, n.description, n.nutritionValue, n.units, r.name
FROM food
JOIN n IN food.nutrients
JOIN r IN (
    SELECT VALUE [
        {unit: 'ng', name: 'nanogram', multiplier: 0.000000001, baseUnit: 'gram'},
        {unit: 'µg', name: 'microgram', multiplier: 0.000001, baseUnit: 'gram'},
        {unit: 'mg', name: 'milligram', multiplier: 0.001, baseUnit: 'gram'},
        {unit: 'g', name: 'gram', multiplier: 1, baseUnit: 'gram'},
        {unit: 'kg', name: 'kilogram', multiplier: 1000, baseUnit: 'gram'},
        {unit: 'Mg', name: 'megagram', multiplier: 1000000, baseUnit: 'gram'},
        {unit: 'Gg', name: 'gigagram', multiplier: 1000000000, baseUnit: 'gram'},
        {unit: 'nJ', name: 'nanojoule', multiplier: 0.000000001, baseUnit: 'joule'},
        {unit: 'µJ', name: 'microjoule', multiplier: 0.000001, baseUnit: 'joule'},
        {unit: 'mJ', name: 'millijoule', multiplier: 0.001, baseUnit: 'joule'},
        {unit: 'J', name: 'joule', multiplier: 1, baseUnit: 'joule'},
        {unit: 'kJ', name: 'kilojoule', multiplier: 1000, baseUnit: 'joule'},
        {unit: 'MJ', name: 'megajoule', multiplier: 1000000, baseUnit: 'joule'},
        {unit: 'GJ', name: 'gigajoule', multiplier: 1000000000, baseUnit: 'joule'},
        {unit: 'cal', name: 'calorie', multiplier: 1, baseUnit: 'calorie'},
        {unit: 'kcal', name: 'Calorie', multiplier: 1000, baseUnit: 'calorie'},
        {unit: 'IU', name: 'International units'}
    ]
)
WHERE n.units = r.unit
```

## <a name="scalar-subqueries"></a>скалярные вложенные запросы;

Скалярное выражение вложенного запроса — это вложенный запрос, результатом которого является единственное значение. Значением скалярного выражения вложенного запроса является значение проекции (предложение SELECT) вложенного запроса.  Можно использовать скалярное выражение вложенного запроса во многих местах, где допустимо использование скалярного выражения. Например, можно использовать скалярный вложенный запрос в любом выражении в предложениях SELECT и WHERE.

Однако использование скалярного вложенного запроса не всегда помогает оптимизировать. Например, передача скалярного вложенного запроса в качестве аргумента в системную или определяемую пользователем функцию не дает никаких преимуществ в использовании единицы ресурсов (RU) или задержке.

Скалярные вложенные запросы можно дополнительно классифицировать следующим образом:
* Скалярные вложенные запросы в простых выражениях
* Агрегирование скалярных вложенных запросов

## <a name="simple-expression-scalar-subqueries"></a>Скалярные вложенные запросы в простых выражениях

Скалярный вложенный запрос с простым выражением — это Коррелированный вложенный запрос, имеющий предложение SELECT, которое не содержит статистических выражений. Эти вложенные запросы не дают никаких преимуществ оптимизации, поскольку компилятор преобразует их в одно более крупное простое выражение. Нет коррелированного контекста между внутренними и внешними запросами.

Рассмотрим несколько примеров.

**Пример 1**

```sql
SELECT 1 AS a, 2 AS b
```

Можно переписать этот запрос с помощью скалярного вложенного запроса простого выражения, чтобы:

```sql
SELECT (SELECT VALUE 1) AS a, (SELECT VALUE 2) AS b
```

Эти выходные данные создаются в обоих запросах:

```json
[
  { "a": 1, "b": 2 }
]
```

**Пример 2**

```sql
SELECT TOP 5 Concat('id_', f.id) AS id
FROM food f
```

Можно переписать этот запрос с помощью скалярного вложенного запроса простого выражения, чтобы:

```sql
SELECT TOP 5 (SELECT VALUE Concat('id_', f.id)) AS id
FROM food f
```

Выходные данные запроса:

```json
[
  { "id": "id_03226" },
  { "id": "id_03227" },
  { "id": "id_03228" },
  { "id": "id_03229" },
  { "id": "id_03230" }
]
```

**Пример 3**

```sql
SELECT TOP 5 f.id, Contains(f.description, 'fruit') = true ? f.description : undefined
FROM food f
```

Можно переписать этот запрос с помощью скалярного вложенного запроса простого выражения, чтобы:

```sql
SELECT TOP 10 f.id, (SELECT f.description WHERE Contains(f.description, 'fruit')).description
FROM food f
```

Выходные данные запроса:

```json
[
  { "id": "03230" },
  { "id": "03238", "description":"Babyfood, dessert, tropical fruit, junior" },
  { "id": "03229" },
  { "id": "03226", "description":"Babyfood, dessert, fruit pudding, orange, strained" },
  { "id": "03227" }
]
```

### <a name="aggregate-scalar-subqueries"></a>Агрегирование скалярных вложенных запросов

Статистический скалярный вложенный запрос — это вложенный запрос, имеющий агрегатную функцию в проекции или фильтре, результатом которой является единственное значение.

**Пример 1**.

Вот вложенный запрос с одним выражением агрегатной функции в его проекции:

```sql
SELECT TOP 5 
    f.id, 
    (SELECT VALUE Count(1) FROM n IN f.nutrients WHERE n.units = 'mg'
) AS count_mg
FROM food f
```

Выходные данные запроса:

```json
[
  { "id": "03230", "count_mg": 13 },
  { "id": "03238", "count_mg": 14 },
  { "id": "03229", "count_mg": 13 },
  { "id": "03226", "count_mg": 15 },
  { "id": "03227", "count_mg": 19 }
]
```

**Пример 2**

Вот вложенный запрос с несколькими выражениями агрегатных функций:

```sql
SELECT TOP 5 f.id, (
    SELECT Count(1) AS count, Sum(n.nutritionValue) AS sum 
    FROM n IN f.nutrients 
    WHERE n.units = 'mg'
) AS unit_mg
FROM food f
```

Выходные данные запроса:

```json
[
  { "id": "03230","unit_mg": { "count": 13,"sum": 147.072 } },
  { "id": "03238","unit_mg": { "count": 14,"sum": 107.385 } },
  { "id": "03229","unit_mg": { "count": 13,"sum": 141.579 } },
  { "id": "03226","unit_mg": { "count": 15,"sum": 183.91399999999996 } },
  { "id": "03227","unit_mg": { "count": 19,"sum": 94.788999999999987 } }
]
```

**Пример 3**

Вот запрос со статистическим вложенным запросом как в проекции, так и в фильтре:

```sql
SELECT TOP 5 
    f.id, 
    (SELECT VALUE Count(1) FROM n IN f.nutrients WHERE n.units = 'mg') AS count_mg
FROM food f
WHERE (SELECT VALUE Count(1) FROM n IN f.nutrients WHERE n.units = 'mg') > 20
```

Выходные данные запроса:

```json
[
  { "id": "03235", "count_mg": 27 },
  { "id": "03246", "count_mg": 21 },
  { "id": "03267", "count_mg": 21 },
  { "id": "03269", "count_mg": 21 },
  { "id": "03274", "count_mg": 21 }
]
```

Более оптимальный способ написания этого запроса — соединение во вложенном запросе и ссылка на псевдоним вложенного запроса в предложениях SELECT и WHERE. Этот запрос более эффективен, поскольку необходимо выполнить вложенный запрос только внутри инструкции JOIN, а не в проекции и фильтре.

```sql
SELECT TOP 5 f.id, count_mg
FROM food f
JOIN (SELECT VALUE Count(1) FROM n IN f.nutrients WHERE n.units = 'mg') AS count_mg
WHERE count_mg > 20
```

## <a name="exists-expression"></a>Выражение EXISTs

Azure Cosmos DB поддерживает выражения EXISTs. Это совокупный скалярный вложенный запрос, встроенный в Azure Cosmos DB API SQL. EXISTs является логическим выражением, которое принимает выражение вложенного запроса и возвращает значение true, если вложенный запрос возвращает какие-либо строки. В противном случае возвращается значение false.

Так как API Azure Cosmos DB SQL не различает логические выражения и другие скалярные выражения, можно использовать в предложениях SELECT и WHERE. Это отличается от T-SQL, где логическое выражение (например, EXISTs, BETWEEN и IN) ограничено фильтром.

Если вложенный запрос EXISTs возвращает одиночное значение, которое не определено, то параметр EXISTs принимает значение false. Например, рассмотрим следующий запрос, результатом которого является значение false:
```sql
SELECT EXISTS (SELECT VALUE undefined)
```   


Если ключевое слово VALUE в предыдущем вложенном запросе опущено, результатом вычисления запроса будет значение true:
```sql
SELECT EXISTS (SELECT undefined) 
```

Вложенный запрос будет заключать список значений в выбранном списке в объекте. Если выбранный список не содержит значений, вложенный запрос возвратит единственное значение " {} ". Это значение определено, поэтому EXISTs вычисляется как true.

### <a name="example-rewriting-array_contains-and-join-as-exists"></a>Пример: перезапись ARRAY_CONTAINS и присоединение как существует

Распространенным вариантом использования ARRAY_CONTAINS является фильтрация документа по существованию элемента в массиве. В этом случае мы проверяя, содержит ли массив Tags элемент с именем «оранжевый».

```sql
SELECT TOP 5 f.id, f.tags
FROM food f
WHERE ARRAY_CONTAINS(f.tags, {name: 'orange'})
```

Вы можете переписать тот же запрос, чтобы использовать EXISTs:

```sql
SELECT TOP 5 f.id, f.tags
FROM food f
WHERE EXISTS(SELECT VALUE t FROM t IN f.tags WHERE t.name = 'orange')
```

Кроме того, ARRAY_CONTAINS может проверять, равно ли значение любому элементу в массиве. Если требуются более сложные фильтры для свойств массива, используйте JOIN.

Рассмотрим следующий запрос, который фильтруется на основе единиц и `nutritionValue` свойств в массиве: 

```sql
SELECT VALUE c.description
FROM c
JOIN n IN c.nutrients
WHERE n.units= "mg" AND n.nutritionValue > 0
```

Для каждого документа в коллекции перекрестное произведение выполняется с элементами массива. Эта операция объединения позволяет фильтровать свойства в массиве. Однако этот запрос будет иметь большое количество запросов. Например, если в 1 000 документах в каждом массиве содержалось 100 элементов, оно будет расширено до 1 000 x 100 (т. е. 100 000) кортежей.

Использование EXISTs может помочь избежать этого дорогостоящего перекрестного произведения:

```sql
SELECT VALUE c.description
FROM c
WHERE EXISTS(
    SELECT VALUE n
    FROM n IN c.nutrients
    WHERE n.units = "mg" AND n.nutritionValue > 0
)
```

В этом случае вы фильтруете элементы массива внутри вложенного запроса EXISTs. Если элемент массива соответствует фильтру, то его проект и EXISTs будут иметь значение true.

Псевдоним также может существовать и ссылаться на него в проекции:

```sql
SELECT TOP 1 c.description, EXISTS(
    SELECT VALUE n
    FROM n IN c.nutrients
    WHERE n.units = "mg" AND n.nutritionValue > 0) as a
FROM c
```

Выходные данные запроса:

```json
[
    {
        "description": "Babyfood, dessert, fruit pudding, orange, strained",
        "a": true
    }
]
```

## <a name="array-expression"></a>Выражение массива

Можно использовать выражение массива для проецирования результатов запроса в виде массива. Это выражение можно использовать только в предложении SELECT запроса.

```sql
SELECT TOP 1   f.id, ARRAY(SELECT VALUE t.name FROM t in f.tags) AS tagNames
FROM  food f
```

Выходные данные запроса:

```json
[
    {
        "id": "03238",
        "tagNames": [
            "babyfood",
            "dessert",
            "tropical fruit",
            "junior"
        ]
    }
]
```

Как и в случае с другими вложенными запросами, возможны фильтры с выражением массива.

```sql
SELECT TOP 1 c.id, ARRAY(SELECT VALUE t FROM t in c.tags WHERE t.name != 'infant formula') AS tagNames
FROM c
```

Выходные данные запроса:

```json
[
    {
        "id": "03226",
        "tagNames": [
            {
                "name": "babyfood"
            },
            {
                "name": "dessert"
            },
            {
                "name": "fruit pudding"
            },
            {
                "name": "orange"
            },
            {
                "name": "strained"
            }
        ]
    }
]
```

Выражения массива могут также следовать после предложения FROM во вложенных запросах.

```sql
SELECT TOP 1 c.id, ARRAY(SELECT VALUE t.name FROM t in c.tags) as tagNames
FROM c
JOIN n IN (SELECT VALUE ARRAY(SELECT t FROM t in c.tags WHERE t.name != 'infant formula'))
```

Выходные данные запроса:

```json
[
    {
        "id": "03238",
        "tagNames": [
            "babyfood",
            "dessert",
            "tropical fruit",
            "junior"
        ]
    }
]
```

## <a name="next-steps"></a>Дальнейшие действия

- [Примеры .NET для Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3)
- [Данные документов модели](modeling-data.md)
