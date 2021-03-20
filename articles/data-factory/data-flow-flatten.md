---
title: Преобразование «сведение» в потоке данных сопоставления
description: Денормализация иерархических данных с помощью преобразования «Сведение»
author: kromerm
ms.author: makromer
ms.review: daperlov
ms.service: data-factory
ms.topic: conceptual
ms.date: 03/09/2020
ms.openlocfilehash: a0e75957a0ab49394dab56f2b7fb847dee4b43cb
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "81413679"
---
# <a name="flatten-transformation-in-mapping-data-flow"></a>Преобразование «сведение» в потоке данных сопоставления

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Используйте преобразование «сведение», чтобы принимать значения массива в иерархических структурах, таких как JSON, и разбить их на отдельные строки. Этот процесс называется денормализацией.

## <a name="configuration"></a>Конфигурация

Преобразование «Сведение» содержит следующие параметры конфигурации.

![Параметры плоской обработки](media/data-flow/flatten1.png "Параметры плоской обработки")

### <a name="unroll-by"></a>Отменить развертывание

Выберите массив для свертывания. Выходные данные будут содержать по одной строке на каждый элемент в каждом массиве. Если невыполненный массив во входной строке имеет значение null или является пустым, то будет выведена одна выходная строка с несведенными значениями NULL.

### <a name="unroll-root"></a>Развернуть корень

По умолчанию преобразование «Сведение» выполняет откат массива к верхней части иерархии, в которой он находится. При необходимости можно выбрать массив в качестве корневого каталога развертывания. Корень unrollа должен быть массивом сложных объектов, которые либо находятся в массиве, либо содержат раскрутку. Если выбран параметр unrollный корень, выходные данные будут содержать по крайней мере одну строку для каждого элемента в корне unrollа. Если входная строка не содержит элементов в корне unrollа, она будет удалена из выходных данных. Выбор корневого корня всегда приведет к выводу меньшего или равного количества строк, чем поведение по умолчанию.

### <a name="flatten-mapping"></a>Плоское сопоставление

Аналогично преобразованию «выбор» выберите проекцию новой структуры из полей входящие и Денормализованный массив. Если Денормализованный массив сопоставлен, то выходной столбец будет иметь тот же тип данных, что и массив. Если массив для раскрутки представляет собой массив сложных объектов, содержащих подмассивы, то сопоставление элемента этого субарри будет выводить массив.

Чтобы проверить выходные данные сопоставления, воспользуйтесь вкладкой Проверка и предварительный просмотр данных.

## <a name="examples"></a>Примеры

См. следующий объект JSON для приведенных ниже примеров преобразования «Сведение»

``` json
{
  "name":"MSFT","location":"Redmond", "satellites": ["Bay Area", "Shanghai"],
  "goods": {
    "trade":true, "customers":["government", "distributer", "retail"],
    "orders":[
        {"orderId":1,"orderTotal":123.34,"shipped":{"orderItems":[{"itemName":"Laptop","itemQty":20},{"itemName":"Charger","itemQty":2}]}},
        {"orderId":2,"orderTotal":323.34,"shipped":{"orderItems":[{"itemName":"Mice","itemQty":2},{"itemName":"Keyboard","itemQty":1}]}}
    ]}}
{"name":"Company1","location":"Seattle", "satellites": ["New York"],
  "goods":{"trade":false, "customers":["store1", "store2"],
  "orders":[
      {"orderId":4,"orderTotal":123.34,"shipped":{"orderItems":[{"itemName":"Laptop","itemQty":20},{"itemName":"Charger","itemQty":3}]}},
      {"orderId":5,"orderTotal":343.24,"shipped":{"orderItems":[{"itemName":"Chair","itemQty":4},{"itemName":"Lamp","itemQty":2}]}}
    ]}}
{"name": "Company2", "location": "Bellevue",
  "goods": {"trade": true, "customers":["Bank"], "orders": [{"orderId": 4, "orderTotal": 123.34}]}}
{"name": "Company3", "location": "Kirkland"}
```

### <a name="no-unroll-root-with-string-array"></a>Нет несведенного корня с массивом строк

| Отменить развертывание | Развернуть корень | Прогнозирование |
| --------- | ----------- | ---------- |
| товары. клиенты | Нет | name <br> Клиент = товары. Клиент |

#### <a name="output"></a>Выходные данные

```
{ 'MSFT', 'government'}
{ 'MSFT', 'distributer'}
{ 'MSFT', 'retail'}
{ 'Company1', 'store'}
{ 'Company1', 'store2'}
{ 'Company2', 'Bank'}
{ 'Company3', null}
```

### <a name="no-unroll-root-with-complex-array"></a>Нет несведенного корня с комплексным массивом

| Отменить развертывание | Развернуть корень | Прогнозирование |
| --------- | ----------- | ---------- |
| товары. заказы. отгруженные. orderItems | Нет | name <br> orderId = товары. Orders. orderId <br> itemName = товары. Orders. отгружено. orderItems. itemName <br> Итемкти = товары. Orders. отгружено. orderItems. Итемкти <br> расположение = расположение |

#### <a name="output"></a>Выходные данные

```
{ 'MSFT', 1, 'Laptop', 20, 'Redmond'}
{ 'MSFT', 1, 'Charger', 2, 'Redmond'}
{ 'MSFT', 2, 'Mice', 2, 'Redmond'}
{ 'MSFT', 2, 'Keyboard', 1, 'Redmond'}
{ 'Company1', 4, 'Laptop', 20, 'Seattle'}
{ 'Company1', 4, 'Charger', 3, 'Seattle'}
{ 'Company1', 5, 'Chair', 4, 'Seattle'}
{ 'Company1', 5, 'Lamp', 2, 'Seattle'}
{ 'Company2', 4, null, null, 'Bellevue'}
{ 'Company3', null, null, null, 'Kirkland'}
```

### <a name="same-root-as-unroll-array"></a>Тот же корень, что и unrollный массив

| Отменить развертывание | Развернуть корень | Прогнозирование |
| --------- | ----------- | ---------- |
| товары. заказы | товары. заказы | name <br> товары. заказы. отгруженные. orderItems. itemName <br> товары. клиенты <br> location |

#### <a name="output"></a>Выходные данные

```
{ 'MSFT', ['Laptop','Charger'], ['government','distributer','retail'], 'Redmond'}
{ 'MSFT', ['Mice', 'Keyboard'], ['government','distributer','retail'], 'Redmond'}
{ 'Company1', ['Laptop','Charger'], ['store', 'store2'], 'Seattle'}
{ 'Company1', ['Chair', 'Lamp'], ['store', 'store2'], 'Seattle'}
{ 'Company2', null, ['Bank'], 'Bellevue'}
```

### <a name="unroll-root-with-complex-array"></a>Развернуть корень с помощью сложного массива

| Отменить развертывание | Развернуть корень | Прогнозирование |
| --------- | ----------- | ---------- |
| товары. заказы. отгруженные. orderItem | товары. заказы |name <br> orderId = товары. Orders. orderId <br> itemName = товары. Orders. отгружено. orderItems. itemName <br> Итемкти = товары. Orders. отгружено. orderItems. Итемкти <br> расположение = расположение |

#### <a name="output"></a>Выходные данные

```
{ 'MSFT', 1, 'Laptop', 20, 'Redmond'}
{ 'MSFT', 1, 'Charger', 2, 'Redmond'}
{ 'MSFT', 2, 'Mice', 2, 'Redmond'}
{ 'MSFT', 2, 'Keyboard', 1, 'Redmond'}
{ 'Company1', 4, 'Laptop', 20, 'Seattle'}
{ 'Company1', 4, 'Charger', 3, 'Seattle'}
{ 'Company1', 5, 'Chair', 4, 'Seattle'}
{ 'Company1', 5, 'Lamp', 2, 'Seattle'}
{ 'Company2', 4, null, null, 'Bellevue'}
```

## <a name="data-flow-script"></a>Скрипт потока данных

### <a name="syntax"></a>Синтаксис

```
<incomingStream>
foldDown(unroll(<unroll cols>),
    mapColumn(
        name,
        each(<array>(type == '<arrayDataType>')),
        each(<array>, match(true())),
        location
    )) ~> <transformationName>
```

### <a name="example"></a>Пример

```
source foldDown(unroll(goods.orders.shipped.orderItems, goods.orders),
    mapColumn(
        name,
        orderId = goods.orders.orderId,
        itemName = goods.orders.shipped.orderItems.itemName,
        itemQty = goods.orders.shipped.orderItems.itemQty,
        location = location
    ),
    skipDuplicateMapInputs: false,
    skipDuplicateMapOutputs: false) 
```    

## <a name="next-steps"></a>Дальнейшие действия

* Используйте [Преобразование «Сведение](data-flow-pivot.md) » для сведения строк в столбцы.
* Используйте [Преобразование отменить сведение](data-flow-unpivot.md) для сведения столбцов в строки.
