---
title: Работа с JSON в Azure Cosmos DB
description: Узнайте, как запрашивать вложенные свойства JSON и получать к ним доступ, а также использовать специальные символы в Azure Cosmos DB
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 09/19/2020
ms.author: tisande
ms.openlocfilehash: 9a9300db1adc3ff238c44887012400702690b0e8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93337835"
---
# <a name="working-with-json-in-azure-cosmos-db"></a>Работа с JSON в Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В API-интерфейсе SQL Azure Cosmos DB (Core) элементы хранятся в формате JSON. Система типов и выражения могут обрабатывать только типы JSON. Дополнительные сведения см. в [спецификациях JSON](https://www.json.org/).

Ниже приведены некоторые важные аспекты работы с JSON.

- Объекты JSON всегда начинаются с левой фигурной скобки (`{`) и заканчиваются правой фигурной скобкой (`}`).
- Свойства JSON можно [вкладывать](#nested-properties) друг в друга.
- Значения свойств JSON могут быть массивами.
- В именах свойств JSON учитывается регистр.
- Имя свойства JSON может быть любым строковым значением (включая пробелы или символы, которые не являются буквами).

## <a name="nested-properties"></a>Вложенные свойства

Доступ к вложенному JSON можно получить с помощью метода доступа с точкой. В запросах вложенные свойства JSON можно использовать так же, как и любые другие свойства.

Вот пример документа с вложенным JSON:

```JSON
{
  "id": "AndersenFamily",
  "lastName": "Andersen",
  "address": {
      "state": "WA",
      "county": "King",
      "city": "Seattle"
      },
  "creationDate": 1431620472,
  "isRegistered": true
}
```

В этом случае свойства `state`, `country` и `city` вложены в свойство `address`.

В следующем примере отображаются два вложенных свойства: `f.address.state` и `f.address.city`.

```sql
    SELECT f.address.state, f.address.city
    FROM Families f
    WHERE f.id = "AndersenFamily"
```

Результаты:

```json
    [{
      "state": "WA",
      "city": "Seattle"
    }]
```

## <a name="working-with-arrays"></a>Работа с массивами

Кроме вложенных свойств, JSON также поддерживает массивы.

Ниже приведен пример документа с массивом.

```json
{
  "id": "WakefieldFamily",
  "children": [
      {
        "familyName": "Merriam",
        "givenName": "Jesse",
        "gender": "female",
        "grade": 1,
      },
      {
        "familyName": "Miller",
         "givenName": "Lisa",
         "gender": "female",
         "grade": 8
      }
  ],
}
```

При работе с массивами можно получить доступ к определенному элементу в массиве, обратившись к его расположению.

```sql
SELECT *
FROM Families f
WHERE f.children[0].givenName = "Jesse"
```

Однако в большинстве случаев при работе с массивами вы будете использовать [вложенный запрос](sql-query-subquery.md) или [самосоединение](sql-query-join.md).

Например, ниже приведен документ, в котором показан ежедневный баланс банковского счета клиента.

```json
{
  "id": "Contoso-Checking-Account-2020",
  "balance": [
      {
        "checkingAccount": 1000,
        "savingsAccount": 5000
      },
      {
        "checkingAccount": 100,
        "savingsAccount": 5000
      },
      {
        "checkingAccount": -10,
        "savingsAccount": 5000,
      },
      {
        "checkingAccount": 5000,
        "savingsAccount": 5000,
      }
         ...
  ]
}
```

Если требуется выполнить запрос для отображения всех клиентов с отрицательным балансом в некоторый момент времени, можно использовать [EXISTS](sql-query-subquery.md#exists-expression) с вложенным запросом.

```sql
SELECT c.id
FROM c
WHERE EXISTS(
    SELECT VALUE n
    FROM n IN c.balance
    WHERE n.checkingAccount < 0
)
```

## <a name="difference-between-null-and-undefined"></a>Разница между NULL и undefine

Если свойство не определено в элементе, то его значение равно `undefined` . Свойство со значением `null` должно быть явно определено и присвоено `null` значение.

Например, рассмотрим этот пример элемента:

```json
{
  "id": "AndersenFamily",
  "lastName": "Andersen",
  "address": {
      "state": "WA",
      "county": "King",
      "city": "Seattle"
      },
  "creationDate": null
}
```

В этом примере свойство `isRegistered` имеет значение, `undefined` так как оно пропущено в элементе. Свойство `creationDate` имеет `null` значение.

Azure Cosmos DB поддерживает два полезных системных функции проверки типов для `null` `undefined` свойств и:

* [IS_NULL](sql-query-is-null.md) — проверяет, является ли значение свойства `null`
* [IS_DEFINED](sql-query-is-defined.md) — проверяет, определено ли значение свойства

Вы можете узнать о [поддерживаемых операторах](sql-query-operators.md) и их поведении для `null` `undefined` значений и.

## <a name="reserved-keywords-and-special-characters-in-json"></a>Зарезервированные ключевые слова и специальные символы в JSON

Свойства можно использовать с помощью оператора заключенного в кавычки свойства `[]`. Например, выражение `SELECT c.grade` and `SELECT c["grade"]` являются эквивалентными. Этот синтаксис полезен, когда необходимо экранировать свойство, которое содержит пробелы, специальные символы или совместно использует имя, совпадающее с именем ключевого слова SQL или зарезервированного слова.

Например, вот документ со свойством с именем `order` и свойством `price($)`, содержащим специальные символы:

```json
{
  "id": "AndersenFamily",
  "order": {
         "orderId": "12345",
         "productId": "A17849",
         "price($)": 59.33
   },
  "creationDate": 1431620472,
  "isRegistered": true
}
```

При выполнении запросов, включающих свойство `order` или `price($)`, отобразится синтаксическая ошибка.

```sql
SELECT * FROM c where c.order.orderid = "12345"
```

```sql
SELECT * FROM c where c.order.price($) > 50
```

Результат:

`
Syntax error, incorrect syntax near 'order'
`

Следует переписать те же запросы, что и ниже.

```sql
SELECT * FROM c WHERE c["order"].orderId = "12345"
```

```sql
SELECT * FROM c WHERE c["order"]["price($)"] > 50
```

## <a name="json-expressions"></a>Выражения JSON

Проекция также поддерживает выражения JSON, как показано в следующем примере.

```sql
    SELECT { "state": f.address.state, "city": f.address.city, "name": f.id }
    FROM Families f
    WHERE f.id = "AndersenFamily"
```

Результаты:

```json
    [{
      "$1": {
        "state": "WA",
        "city": "Seattle",
        "name": "AndersenFamily"
      }
    }]
```

В предыдущем примере в предложении `SELECT` необходимо создать объект JSON, а поскольку в примере нет ключа, предложение использует имя неявной переменной аргумента `$1`. Следующий запрос возвращает две неявные переменные аргумента: `$1` и `$2`.

```sql
    SELECT { "state": f.address.state, "city": f.address.city },
           { "name": f.id }
    FROM Families f
    WHERE f.id = "AndersenFamily"
```

Результаты:

```json
    [{
      "$1": {
        "state": "WA",
        "city": "Seattle"
      },
      "$2": {
        "name": "AndersenFamily"
      }
    }]
```

## <a name="aliasing"></a>Псевдонимы

В запросах можно явно задавать значения псевдонимов. Если в запросе имеются два свойства с совпадающими именами, используйте псевдонимы для переименования одного или обоих свойств, чтобы устранить неоднозначность в отображаемом результате.

### <a name="examples"></a>Примеры

Ключевое слово `AS`, используемое для присвоения псевдонимов, является необязательным, как показано в следующем примере при проецировании второго значения как `NameInfo`.

```sql
    SELECT
           { "state": f.address.state, "city": f.address.city } AS AddressInfo,
           { "name": f.id } NameInfo
    FROM Families f
    WHERE f.id = "AndersenFamily"
```

Результаты:

```json
    [{
      "AddressInfo": {
        "state": "WA",
        "city": "Seattle"
      },
      "NameInfo": {
        "name": "AndersenFamily"
      }
    }]
```

### <a name="aliasing-with-reserved-keywords-or-special-characters"></a>Присвоение псевдонимов зарезервированным ключевым словам или специальным символам

Нельзя использовать псевдонимы для проецирования значения в качестве имени свойства с пробелом, специальным символом или зарезервированным словом. Чтобы изменить проекцию значения, например, на имя свойства с пробелом, можно использовать [выражение JSON](#json-expressions).

Ниже приведен пример:

```sql
    SELECT
           {"JSON expression with a space": { "state": f.address.state, "city": f.address.city }},
           {"JSON expression with a special character!": { "name": f.id }}
    FROM Families f
    WHERE f.id = "AndersenFamily"
```

## <a name="next-steps"></a>Дальнейшие действия

- [Начало работы](sql-query-getting-started.md)
- [Предложение SELECT](sql-query-select.md)
- [Предложение WHERE](sql-query-where.md)
