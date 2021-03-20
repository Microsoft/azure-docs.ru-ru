---
author: brjohnstmsft
ms.service: cognitive-search
ms.topic: include
ms.date: 06/13/2018
ms.author: brjohnst
ms.openlocfilehash: 998d0f1a84dc9cb2a07fb55286c1089787a263e1
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "80272627"
---
| Тип данных | Функции, разрешенные в лямбда-выражениях с помощью `any` | Функции, разрешенные в лямбда-выражениях с помощью `all` |
|---|---|---|
| `Collection(Edm.ComplexType)` | Все `search.ismatch` , кроме и `search.ismatchscoring` | Аналогично |
| `Collection(Edm.String)` | Сравнения с `eq` или `search.in` <br/><br/> Объединение подвыражений с `or` | Сравнения с `ne` или `not search.in()` <br/><br/> Объединение подвыражений с `and` |
| `Collection(Edm.Boolean)` | Сравнения с `eq` или `ne` | Аналогично |
| `Collection(Edm.GeographyPoint)` | Использование `geo.distance` с `lt` или `le` <br/><br/> `geo.intersects` <br/><br/> Объединение подвыражений с `or` | Использование `geo.distance` с `gt` или `ge` <br/><br/> `not geo.intersects(...)` <br/><br/> Объединение подвыражений с `and` |
| `Collection(Edm.DateTimeOffset)`, `Collection(Edm.Double)`, `Collection(Edm.Int32)`, `Collection(Edm.Int64)` | Сравнения с помощью `eq` , `ne` , `lt` , `gt` , `le` или `ge` <br/><br/> Объединение сравнений с другими подвыражениями с помощью `or` <br/><br/> Объединение сравнений, за исключением `ne` других подвыражений, использующих `and` <br/><br/> Выражения, использующие сочетания `and` и `or` в [обычной форме дисжунктиве (ДНФ)](https://en.wikipedia.org/wiki/Disjunctive_normal_form) | Сравнения с помощью `eq` , `ne` , `lt` , `gt` , `le` или `ge` <br/><br/> Объединение сравнений с другими подвыражениями с помощью `and` <br/><br/> Объединение сравнений, за исключением `eq` других подвыражений, использующих `or` <br/><br/> Выражения, использующие сочетания `and` и `or` в [обычной форме был соединительным (cnf)](https://en.wikipedia.org/wiki/Conjunctive_normal_form) |
