---
title: Определение размера таблицы — изменение масштаба (Цитус) — база данных Azure для PostgreSQL
description: Как определить истинный размер распределенных таблиц в группе серверов с горизонтальным масштабированием (Цитус)
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 1/5/2021
ms.openlocfilehash: 6ebdbe250862ccd462259900ba56d007f002a52f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97937679"
---
# <a name="determine-table-and-relation-size"></a>Определение размера таблицы и отношения

Обычным способом поиска размеров таблиц в PostgreSQL, значительно менее чем в отчете, является `pg_total_relation_size` Размер распределенных таблиц в режиме масштабирования (Цитус).
Вся эта функция в группе серверов с горизонтальным масштабированием (Цитус) позволяет отобразить размер таблиц на узле координатора.  В действительности данные в распределенных таблицах находятся на рабочих узлах (в сегментах), а не в координаторе. Истинная мера размера распределенной таблицы получается как сумма размеров сегментов. Функция масштабирования (Цитус) предоставляет вспомогательные функции для запроса этих сведений.

<table>
<colgroup>
<col style="width: 40%" />
<col style="width: 59%" />
</colgroup>
<thead>
<tr class="header">
<th>Компонент</th>
<th>Возвращаемое значение</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>citus_relation_size (relation_name)</td>
<td><ul>
<li>Размер фактических данных в таблице ("<a href="https://www.postgresql.org/docs/current/static/storage-file-layout.html">Основная вилка</a>").</li>
<li>Отношение может быть именем таблицы или индекса.</li>
</ul></td>
</tr>
<tr class="even">
<td>citus_table_size (relation_name)</td>
<td><ul>
<li><p>citus_relation_size Plus:</p>
<blockquote>
<ul>
<li>размер <a href="https://www.postgresql.org/docs/current/static/storage-fsm.html">схемы свободного пространства</a></li>
<li>размер <a href="https://www.postgresql.org/docs/current/static/storage-vm.html">схемы видимости</a></li>
</ul>
</blockquote></li>
</ul></td>
</tr>
<tr class="odd">
<td>citus_total_relation_size (relation_name)</td>
<td><ul>
<li><p>citus_table_size Plus:</p>
<blockquote>
<ul>
<li>размер индексов</li>
</ul>
</blockquote></li>
</ul></td>
</tr>
</tbody>
</table>

Эти функции аналогичны трем стандартным [функциям размера объекта](https://www.postgresql.org/docs/current/static/functions-admin.html#FUNCTIONS-ADMIN-DBSIZE)PostgreSQL, за исключением случаев, когда они не могут подключиться к узлу, они выдают ошибку.

## <a name="example"></a>Пример

Вот как можно вывести список размеров всех распределенных таблиц:

``` postgresql
SELECT logicalrelid AS name,
       pg_size_pretty(citus_table_size(logicalrelid)) AS size
  FROM pg_dist_partition;
```

Выходные данные:

```
┌───────────────┬───────┐
│     name      │ size  │
├───────────────┼───────┤
│ github_users  │ 39 MB │
│ github_events │ 37 MB │
└───────────────┴───────┘
```

## <a name="next-steps"></a>Дальнейшие действия

* Научитесь [масштабировать группу серверов](howto-hyperscale-scale-grow.md) для хранения дополнительных данных.
* Различать [табличные типы](concepts-hyperscale-nodes.md) в группе серверов с горизонтальным масштабированием (Цитус).
