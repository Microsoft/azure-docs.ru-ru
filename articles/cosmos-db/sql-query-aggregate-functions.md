---
title: Агрегатные функции в Azure Cosmos DB
description: Сведения о синтаксисе агрегатных функций SQL, типах агрегатных функций, поддерживаемых Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 12/02/2020
ms.author: tisande
ms.openlocfilehash: c0d953c8d99582f63744d51b505852b5c44bc409
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96555416"
---
# <a name="aggregate-functions-in-azure-cosmos-db"></a>Агрегатные функции в Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Агрегатные функции выполняют вычисление для набора значений в `SELECT` предложении и возвращают одно значение. Например, следующий запрос возвращает количество элементов в контейнере:

```sql
    SELECT COUNT(1)
    FROM c
```

## <a name="types-of-aggregate-functions"></a>Типы агрегатных функций

API SQL поддерживает следующие агрегатные функции. `SUM` и `AVG` используют числовые значения, и `COUNT` , и `MIN` `MAX` работают с числами, строками, логическими значениями и значениями NULL.

| Компонент | Описание |
|-------|-------------|
| [AVG](sql-query-aggregate-avg.md) | Возвращает среднее арифметическое значений в выражении. |
| [COUNT](sql-query-aggregate-count.md) | Возвращает число элементов в выражении. |
| [MAX](sql-query-aggregate-max.md) | Возвращает максимальное значение выражения. |
| [MIN](sql-query-aggregate-min.md) | Возвращает минимальное значение выражения. |
| [SUM](sql-query-aggregate-sum.md) | Возвращает сумму всех элементов в выражении. |


Можно также вернуть только скалярное значение статистической функции с помощью ключевого слова VALUE. Например следующий запрос возвращает число значений как одно число:

```sql
    SELECT VALUE COUNT(1)
    FROM Families f
```

Результаты:

```json
    [ 2 ]
```

Кроме того, можно объединять агрегаты с фильтрами. Например, следующий запрос возвращает количество элементов с состоянием адреса `WA` .

```sql
    SELECT VALUE COUNT(1)
    FROM Families f
    WHERE f.address.state = "WA"
```

Результаты:

```json
    [ 1 ]
```

## <a name="remarks"></a>Remarks

Эти статистические системные функции получают преимущества от [индекса диапазона](index-policy.md#includeexclude-strategy). Если предполагается использовать свойство, `AVG` , `COUNT` `MAX` , `MIN` или, необходимо `SUM` [включить соответствующий путь в политику индексирования](index-policy.md#includeexclude-strategy).

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения об Azure Cosmos DB](introduction.md)
- [Системные функции](sql-query-system-functions.md)
- [Определяемые пользователем функции](sql-query-udfs.md)