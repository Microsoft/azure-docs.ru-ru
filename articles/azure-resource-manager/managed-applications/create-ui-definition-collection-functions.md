---
title: Создание функций коллекции определений пользовательского интерфейса
description: Описывает функции, используемые при работе с коллекциями, такими как массивы и объекты.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: tomfitz
ms.openlocfilehash: 2a075c5c99f457681cd49e75014487bf9cca263c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87098202"
---
# <a name="createuidefinition-collection-functions"></a>Функции коллекции CreateUiDefinition

Эти функции могут использоваться с коллекциями, такими как строки, массивы и объекты JSON.

## <a name="contains"></a>содержит

Возвращает `true`, если строка содержит указанную подстроку, массив содержит указанное значение или объект содержит указанный ключ.

### <a name="example-string-contains"></a>Пример: строка содержит

В следующем примере возвращается `true`.

```json
"[contains('webapp', 'web')]"
```

### <a name="example-array-contains"></a>Пример: массив содержит

Предположим, что `element1` возвращает `[1, 2, 3]`. В следующем примере возвращается `false`.

```json
"[contains(steps('demoStep').element1, 4)]"
```

### <a name="example-object-contains"></a>Пример: объект содержит

Предположим, что `element1` возвращает:

```json
{
  "key1": "Linux",
  "key2": "Windows"
}
```

В следующем примере возвращается `true`.

```json
"[contains(steps('demoStep').element1, 'key1')]"
```

## <a name="empty"></a>пустых

Возвращает `true`, если строка, массив или объект имеет значение null или является пустым.

### <a name="example-string-empty"></a>Пример: пустая строка

В следующем примере возвращается `true`.

```json
"[empty('')]"
```

### <a name="example-array-empty"></a>Пример: массив пуст

Предположим, что `element1` возвращает `[1, 2, 3]`. В следующем примере возвращается `false`.

```json
"[empty(steps('demoStep').element1)]"
```

### <a name="example-object-empty"></a>Пример: пустой объект

Предположим, что `element1` возвращает:

```json
{
  "key1": "Linux",
  "key2": "Windows"
}
```

В следующем примере возвращается `false`.

```json
"[empty(steps('demoStep').element1)]"
```

### <a name="example-null-and-undefined"></a>Пример: NULL и undefine

Предположим, что `element1` имеет значение `null` или не определено. В следующем примере возвращается `true`.

```json
"[empty(steps('demoStep').element1)]"
```

## <a name="filter"></a>фильтр

Возвращает новый массив после применения логики фильтрации, предоставляемой в виде лямбда-функции. Первый параметр — это массив, используемый для фильтрации. Вторым параметром является лямбда-функция, указывающая логику фильтрации.

Следующий пример возвращает массив `[ { "name": "abc" } ]` .

```json
"[filter(parse('[{\"name\":\"abc\"},{\"name\":\"xyz\"}]'), (item) => contains(item.name, 'abc'))]"
```

## <a name="first"></a>first

Возвращает первый знак указанной строки, первое значение указанного массива или первый ключ и значение указанного объекта.

### <a name="example-string-first"></a>Пример: строка First

В следующем примере возвращается `"c"`.

```json
"[first('contoso')]"
```

### <a name="example-array-first"></a>Пример: массив First

Предположим, что `element1` возвращает `[1, 2, 3]`. В следующем примере возвращается `1`.

```json
"[first(steps('demoStep').element1)]"
```

#### <a name="example-object-first"></a>Пример: сначала объект

Предположим, что `element1` возвращает:

```json
{
  "key1": "Linux",
  "key2": "Windows"
}
```

В следующем примере возвращается `{"key1": "Linux"}`.

```json
"[first(steps('demoStep').element1)]"
```

## <a name="last"></a>last

Возвращает последний знак указанной строки, последнее значение указанного массива или последний ключ и значение указанного объекта.

### <a name="example-string-last"></a>Пример: Последняя строка

В следующем примере возвращается `"o"`.

```json
"[last('contoso')]"
```

### <a name="example-array-last"></a>Пример: массив Last

Предположим, что `element1` возвращает `[1, 2, 3]`. В следующем примере возвращается `3`.

```json
"[last(steps('demoStep').element1)]"
```

### <a name="example-object-last"></a>Пример: объект Last

Предположим, что `element1` возвращает:

```json
{
  "key1": "Linux",
  "key2": "Windows"
}
```

В следующем примере возвращается `{"key2": "Windows"}`.

```json
"[last(steps('demoStep').element1)]"
```

## <a name="length"></a>length

Возвращает количество символов в строке, количество значений в массиве или количество ключей в объекте.

### <a name="example-string-length"></a>Пример: длина строки

В следующем примере возвращается `7`.

```json
"[length('Contoso')]"
```

### <a name="example-array-length"></a>Пример: длина массива

Предположим, что `element1` возвращает `[1, 2, 3]`. В следующем примере возвращается `3`.

```json
"[length(steps('demoStep').element1)]"
```

### <a name="example-object-length"></a>Пример: длина объекта

Предположим, что `element1` возвращает:

```json
{
  "key1": "Linux",
  "key2": "Windows"
}
```

В следующем примере возвращается `2`.

```json
"[length(steps('demoStep').element1)]"
```

## <a name="map"></a>карта

Возвращает новый массив после вызова лямбда-функции для предоставленного массива. Первый параметр — это массив, используемый для лямбда-функции. Вторым параметром является лямбда-функция.

В следующем примере возвращается новый массив с каждым значением, которое удваивается. Результат равен `[2, 4, 6]`.

```json
"[map(parse('[1, 2, 3]'), (item) => mul(2, item))]"
```

В следующем примере возвращается новый массив `["abc", "xyz"]` .

```json
"[map(parse('[{\"name\":\"abc\"},{\"name\":\"xyz\"}]'), (item) => item.name)]"
```

## <a name="skip"></a>skip

Пропускает заданное число элементов в коллекции и возвращает остальные элементы.

### <a name="example-string-skip"></a>Пример: Пропуск строки

В следующем примере возвращается `"app"`.

```json
"[skip('webapp', 3)]"
```

### <a name="example-array-skip"></a>Пример: пропуск массива

Предположим, что `element1` возвращает `[1, 2, 3]`. В следующем примере возвращается `[3]`.

```json
"[skip(steps('demoStep').element1, 2)]"
```

### <a name="example-object-skip"></a>Пример: Пропуск объекта

Предположим, что `element1` возвращает:

```json
{
  "key1": "Linux",
  "key2": "Windows"
}
```
В следующем примере возвращается `{"key2": "Windows"}`.

```json
"[skip(steps('demoStep').element1, 1)]"
```

## <a name="split"></a>разделение

Возвращает массив строк, содержащий подстроки входных данных, разделенные разделителем.

Следующий пример возвращает массив `[ "555", "867", "5309" ]` .

```json
"[split('555-867-5309', '-')]"
```

## <a name="take"></a>take

Возвращает указанное число последовательных знаков, считая от начала строки, указанное число непрерывных значений от начала массива или указанное число последовательных ключей и значений от начала объекта.

### <a name="example-string-take"></a>Пример: строка take

В следующем примере возвращается `"web"`.

```json
"[take('webapp', 3)]"
```

### <a name="example-array-take"></a>Пример: массив take

Предположим, что `element1` возвращает `[1, 2, 3]`. В следующем примере возвращается `[1, 2]`.

```json
"[take(steps('demoStep').element1, 2)]"
```

### <a name="example-object-take"></a>Пример: объект take

Предположим, что `element1` возвращает:

```json
{
  "key1": "Linux",
  "key2": "Windows"
}
```

В следующем примере возвращается `{"key1": "Linux"}`.

```json
"[take(steps('demoStep').element1, 1)]"
```

## <a name="next-steps"></a>Дальнейшие действия

* Общие сведения об Azure Resource Manager см. в [этой статье](../management/overview.md).
