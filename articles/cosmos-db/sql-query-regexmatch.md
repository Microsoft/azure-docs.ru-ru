---
title: Режексматч на языке запросов Azure Cosmos DB
description: Сведения о системной функции Режексматч SQL в Azure Cosmos DB
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 10/13/2020
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: 7f5b2831f45e902b312636e4133557a16ee7ec95
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93341643"
---
# <a name="regexmatch-azure-cosmos-db"></a>РЕЖЕКСМАТЧ (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Предоставляет возможности регулярных выражений. Регулярные выражения — это краткая и гибкая нотация для поиска шаблонов текста. Azure Cosmos DB использует [регулярные выражения, совместимые с Perl (PCRE)](http://www.pcre.org/). 

## <a name="syntax"></a>Синтаксис
  
```sql
RegexMatch(<str_expr1>, <str_expr2>, [, <str_expr3>])  
```  
  
## <a name="arguments"></a>Аргументы
  
*str_expr1*  
   Строковое выражение, в котором выполняется поиск.  
  
*str_expr2*  
   Регулярное выражение.

*str_expr3*  
   Строка выбранных модификаторов для использования с регулярным выражением. Это строковое значение является необязательным. Если вы хотите выполнить Режексматч без модификаторов, можно либо добавить пустую строку, либо полностью опустить. 

Дополнительные сведения о [синтаксисе для создания регулярных выражений можно получить на Perl](https://perldoc.perl.org/perlre). 

Azure Cosmos DB поддерживает следующие четыре модификатора:

| Модификатор | Описание |
| ------ | ----------- |
| `m` | Рассматривайте строковое выражение для поиска в виде нескольких строк. Без этого параметра "^" и "$" будут соответствовать началу или концу строки, а не каждой отдельной строке. |
| `s` | Разрешить "." для поиска соответствия любому символу, включая символ новой строки. | 
| `i` | Не учитывать регистр при сопоставлении шаблонов. |
| `x` | Пропуск всех символов пробела. |

## <a name="return-types"></a>Типы возвращаемых данных
  
  Возвращает логическое выражение. Возвращает значение undefine, если строковое выражение для поиска, регулярное выражение или выбранные модификаторы недопустимы.
  
## <a name="examples"></a>Примеры
  
В следующем простом примере Режексматч проверяется строка "abcd" для сопоставления регулярного выражения с помощью нескольких различных модификаторов.
  
```sql
SELECT RegexMatch ("abcd", "ABC", "") AS NoModifiers, 
RegexMatch ("abcd", "ABC", "i") AS CaseInsensitive, 
RegexMatch ("abcd", "ab.", "") AS WildcardCharacter,
RegexMatch ("abcd", "ab c", "x") AS IgnoreWhiteSpace, 
RegexMatch ("abcd", "aB c", "ix") AS CaseInsensitiveAndIgnoreWhiteSpace 
```  
  
 Результирующий набор:  
  
```json
[
    {
        "NoModifiers": false,
        "CaseInsensitive": true,
        "WildcardCharacter": true,
        "IgnoreWhiteSpace": true,
        "CaseInsensitiveAndIgnoreWhiteSpace": true
    }
]
```

С помощью Режексматч можно использовать метасимволы для более сложных операций поиска строк, которые в противном случае не могли бы быть доступны в системных функциях StartsWith, EndsWith, Contains или Стринжекуалс. Ниже приведены некоторые дополнительные примеры, которые можно выполнить с помощью набора данных информации питании, доступного в [Площадка для тестирования запросов Azure Cosmos DB](https://www.documentdb.com/sql/demo). 

> [!NOTE] 
> Если необходимо использовать метасимвол в регулярном выражении и не требуется, чтобы он имел специальное значение, необходимо экранировать метасимвол с помощью `\` .

**Проверьте элементы, которые имеют описание, содержащее слово "Salt", ровно один раз:**

```sql
SELECT * 
FROM c 
WHERE RegexMatch (c.description, "salt{1}","")
```

**Проверьте элементы с описанием, содержащим число от 0 до 99:**

```sql
SELECT * 
FROM c 
WHERE RegexMatch (c.description, "[0-99]","")
```

**Проверьте элементы с описанием, содержащим четыре буквы, начинающиеся с "S" или "s":**

```sql
SELECT * 
FROM c 
WHERE RegexMatch (c.description, " s... ","i")
```

## <a name="remarks"></a>Remarks

Эта системная функция будет выиграть от [индекса диапазона](index-policy.md#includeexclude-strategy) , если регулярное выражение можно разделить на системные функции startsWith, EndsWith, Contains или стринжекуалс.

## <a name="next-steps"></a>Дальнейшие действия

- [Строковые функции Azure Cosmos DB](sql-query-string-functions.md)
- [Системные функции Azure Cosmos DB](sql-query-system-functions.md)
- [Знакомство со службой Azure Cosmos DB. API DocumentDB](introduction.md)
