---
title: Преобразование LINQ to SQL в Azure Cosmos DB
description: Узнайте о поддерживаемых операторах LINQ и о том, как запросы LINQ сопоставляются с запросами SQL в Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 11/11/2020
ms.author: tisande
ms.openlocfilehash: 122c95fe9ac017ad7a6957dcdb8323837be34f21
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96545389"
---
# <a name="linq-to-sql-translation"></a>Трансляция из LINQ в SQL
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Поставщик запросов Azure Cosmos DB выполняет наилучшее сопоставление запроса LINQ с запросом на Cosmos DB SQL. Если требуется получить SQL-запрос, преобразованный из LINQ, используйте `ToString()` метод для созданного `IQueryable` объекта. Следующее описание предполагает базовое знание [LINQ](/dotnet/csharp/programming-guide/concepts/linq/introduction-to-linq-queries). Помимо LINQ, Azure Cosmos DB также поддерживает [Entity Framework Core](/ef/core/providers/cosmos/?tabs=dotnet-core-cli) , который работает с API SQL.

Система типов поставщиков запросов поддерживает только типы-примитивы JSON: numeric, Boolean, String и NULL.

Поставщик запросов поддерживает следующие скалярные выражения:

- Постоянные значения, включая постоянные значения типов данных-примитивов во время оценки запроса.
  
- Выражения индекса свойства или массива, которые ссылаются на свойство объекта или элемента массива. Пример:
  
  ```
    family.Id;
    family.children[0].familyName;
    family.children[0].grade;
    family.children[n].grade; //n is an int variable
  ```
  
- Арифметические выражения, включая общие арифметические выражения для числовых и логических значений. Полный список см. в [спецификации Azure Cosmos DB SQL](sql-query-aggregate-functions.md).
  
  ```
    2 * family.children[0].grade;
    x + y;
  ```
  
- Выражения сравнения строк, которые включают сравнение строкового значения с некоторым постоянным строковым значением.  
  
  ```
    mother.familyName == "Wakefield";
    child.givenName == s; //s is a string variable
  ```
  
- Выражения создания объектов и массивов, возвращающие объект составного типа значения или анонимного типа или массив таких объектов. Эти значения можно вложить.
  
  ```
    new Parent { familyName = "Wakefield", givenName = "Robin" };
    new { first = 1, second = 2 }; //an anonymous type with two fields  
    new int[] { 3, child.grade, 5 };
  ```

## <a name="using-linq"></a>Использование языка LINQ

Можно создать запрос LINQ с помощью `GetItemLinqQueryable` . В этом примере показано создание запроса LINQ и асинхронное выполнение с помощью `FeedIterator` :

```csharp
using (FeedIterator<Book> setIterator = container.GetItemLinqQueryable<Book>()
                      .Where(b => b.Title == "War and Peace")
                      .ToFeedIterator<Book>())
 {
     //Asynchronous query execution
     while (setIterator.HasMoreResults)
     {
         foreach(var item in await setIterator.ReadNextAsync()){
         {
             Console.WriteLine(item.cost);
         }
       }
     }
 }
```

## <a name="supported-linq-operators"></a><a id="SupportedLinqOperators"></a>Поддерживаемые операторы LINQ

Поставщик LINQ, входящий в состав пакета SDK для SQL .NET, поддерживает следующие операторы:

- **SELECT**: проекции, переводимые в [SELECT](sql-query-select.md), включая создание объектов.
- **Где**: [фильтрует и поддерживает](sql-query-where.md)преобразование между `&&` , и `||` `!` в операторы SQL
- **SelectMany**: позволяет очищать массивы в предложении [Join](sql-query-join.md) . Используйте, чтобы связать или вложить выражения для фильтрации элементов массива.
- **OrderBy** и **OrderByDescending**: преобразование для [упорядочивания](sql-query-order-by.md) с ASC или DESC.
- Операторы **Count**, **Sum**, **min**, **Max** и **Average** для [агрегирования](sql-query-aggregate-functions.md)и их асинхронные эквиваленты **каунтасинк**, **сумасинк**, **минасинк**, **максасинк** и **аверажеасинк**.
- **CompareTo**: выполняет преобразование в сравнение диапазонов. Обычно используется для строк, так как они не сравнимы в .NET.
- **Пропустить** и **выполнить**: преобразуется в [смещение и ограничивает](sql-query-offset-limit.md) ограничение результатов запроса и выполнение разбиения на страницы.
- **Математические функции**: поддерживают преобразование из .NET,,,,,,,,,,,,,,, `Abs` `Acos` `Asin` `Atan` `Ceiling` `Cos` `Exp` `Floor` `Log` `Log10` `Pow` `Round` `Sign` `Sin` `Sqrt` `Tan` и `Truncate` в эквивалентные [встроенные математические функции](sql-query-mathematical-functions.md).
- **Строковые функции**: поддерживает преобразование из .NET,,,,,,,,,,, `Concat` `Contains` `Count` `EndsWith` `IndexOf` `Replace` `Reverse` `StartsWith` `SubString` `ToLower` `ToUpper` `TrimEnd` и `TrimStart` в эквивалентные [встроенные строковые функции](sql-query-string-functions.md).
- **Функции массивов**: поддерживает преобразование из .NET `Concat` , `Contains` и `Count` в эквивалентные [встроенные функции массивов](sql-query-array-functions.md).
- **Функции геопространственных расширений**: поддерживает преобразование из методов-заглушек `Distance` ,, `IsValid` `IsValidDetailed` и `Within` в эквивалентные [встроенные геопространственные функции](sql-query-geospatial-query.md).
- **Определяемая пользователем функция расширения функции**: поддерживает преобразование из метода-заглушки `UserDefinedFunctionProvider.Invoke` в соответствующую [определяемую пользователем функцию](sql-query-udfs.md).
- **Разное**: поддерживает преобразование `Coalesce` и условные [Операторы](sql-query-operators.md). Может преобразовывать `Contains` в строку Contains, ARRAY_CONTAINS или IN в зависимости от контекста.

## <a name="examples"></a>Примеры

В следующих примерах показано, как некоторые из стандартных операторов запросов LINQ преобразуются в запросы в Azure Cosmos DB.

### <a name="select-operator"></a>Оператор SELECT

Синтаксис: `input.Select(x => f(x))`, где `f` — скалярное выражение. `input`В данном случае — это `IQueryable` объект.

**Оператор SELECT, пример 1:**

- **Лямбда-выражение LINQ**
  
  ```csharp
      input.Select(family => family.parents[0].familyName);
  ```
  
- **SQL**
  
  ```sql
      SELECT VALUE f.parents[0].familyName
      FROM Families f
    ```
  
**Оператор SELECT, пример 2:**

- **Лямбда-выражение LINQ**
  
  ```csharp
      input.Select(family => family.children[0].grade + c); // c is an int variable
  ```
  
- **SQL**
  
  ```sql
      SELECT VALUE f.children[0].grade + c
      FROM Families f
  ```
  
**Оператор SELECT, пример 3:**

- **Лямбда-выражение LINQ**
  
  ```csharp
    input.Select(family => new
    {
        name = family.children[0].familyName,
        grade = family.children[0].grade + 3
    });
  ```
  
- **SQL**
  
  ```sql
      SELECT VALUE {"name":f.children[0].familyName,
                    "grade": f.children[0].grade + 3 }
      FROM Families f
  ```

### <a name="selectmany-operator"></a>Оператор SelectMany

Синтаксис: `input.SelectMany(x => f(x))`, где `f` — скалярное выражение, возвращающее тип контейнера.

- **Лямбда-выражение LINQ**
  
  ```csharp
      input.SelectMany(family => family.children);
  ```
  
- **SQL**

  ```sql
      SELECT VALUE child
      FROM child IN Families.children
  ```

### <a name="where-operator"></a>Оператор Where

Синтаксис: `input.Where(x => f(x))`, где `f` — скалярное выражение, возвращающее логическое значение.

**Оператор WHERE, пример 1:**

- **Лямбда-выражение LINQ**
  
  ```csharp
      input.Where(family=> family.parents[0].familyName == "Wakefield");
  ```
  
- **SQL**
  
  ```sql
      SELECT *
      FROM Families f
      WHERE f.parents[0].familyName = "Wakefield"
  ```
  
**Оператор WHERE, пример 2:**

- **Лямбда-выражение LINQ**
  
  ```csharp
      input.Where(
          family => family.parents[0].familyName == "Wakefield" &&
          family.children[0].grade < 3);
  ```
  
- **SQL**
  
  ```sql
      SELECT *
      FROM Families f
      WHERE f.parents[0].familyName = "Wakefield"
      AND f.children[0].grade < 3
  ```

## <a name="composite-sql-queries"></a>Составные SQL-запросы

Вы можете составить предшествующие операторы для формирования более эффективных запросов. Поскольку Cosmos DB поддерживает вложенные контейнеры, можно объединить или вложить композицию.

### <a name="concatenation"></a>Объединение

Синтаксис: `input(.|.SelectMany())(.Select()|.Where())*`. Объединенный запрос может начинаться с необязательного `SelectMany` запроса, за которым следует несколько `Select` `Where` операторов или.

**Объединение, пример 1:**

- **Лямбда-выражение LINQ**
  
  ```csharp
      input.Select(family => family.parents[0])
          .Where(parent => parent.familyName == "Wakefield");
  ```

- **SQL**
  
  ```sql
      SELECT *
      FROM Families f
      WHERE f.parents[0].familyName = "Wakefield"
  ```

**Объединение, пример 2:**

- **Лямбда-выражение LINQ**
  
  ```csharp
      input.Where(family => family.children[0].grade > 3)
          .Select(family => family.parents[0].familyName);
  ```

- **SQL**
  
  ```sql
      SELECT VALUE f.parents[0].familyName
      FROM Families f
      WHERE f.children[0].grade > 3
  ```

**Объединение, пример 3.**

- **Лямбда-выражение LINQ**
  
  ```csharp
      input.Select(family => new { grade=family.children[0].grade}).
          Where(anon=> anon.grade < 3);
  ```
  
- **SQL**
  
  ```sql
      SELECT *
      FROM Families f
      WHERE ({grade: f.children[0].grade}.grade > 3)
  ```

**Объединение, пример 4:**

- **Лямбда-выражение LINQ**
  
  ```csharp
      input.SelectMany(family => family.parents)
          .Where(parent => parents.familyName == "Wakefield");
  ```
  
- **SQL**
  
  ```sql
      SELECT *
      FROM p IN Families.parents
      WHERE p.familyName = "Wakefield"
  ```

### <a name="nesting"></a>Вложенные операторы

Синтаксис — это `input.SelectMany(x=>x.Q())` `Q` `Select` `SelectMany` оператор, или `Where` .

Вложенный запрос применяет внутренний запрос к каждому элементу внешнего контейнера. Одной из важных возможностей является то, что внутренний запрос может ссылаться на поля элементов во внешнем контейнере, например на самосоединение.

**Вложенность, пример 1:**

- **Лямбда-выражение LINQ**
  
  ```csharp
      input.SelectMany(family=>
          family.parents.Select(p => p.familyName));
  ```

- **SQL**
  
  ```sql
      SELECT VALUE p.familyName
      FROM Families f
      JOIN p IN f.parents
  ```

**Вложенность, пример 2:**

- **Лямбда-выражение LINQ**
  
  ```csharp
      input.SelectMany(family =>
          family.children.Where(child => child.familyName == "Jeff"));
  ```

- **SQL**
  
  ```sql
      SELECT *
      FROM Families f
      JOIN c IN f.children
      WHERE c.familyName = "Jeff"
  ```

**Вложение, пример 3:**

- **Лямбда-выражение LINQ**
  
  ```csharp
      input.SelectMany(family => family.children.Where(
          child => child.familyName == family.parents[0].familyName));
  ```

- **SQL**
  
  ```sql
      SELECT *
      FROM Families f
      JOIN c IN f.children
      WHERE c.familyName = f.parents[0].familyName
  ```

## <a name="next-steps"></a>Дальнейшие действия

- [Примеры .NET для Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3)
- [Данные документов модели](modeling-data.md)