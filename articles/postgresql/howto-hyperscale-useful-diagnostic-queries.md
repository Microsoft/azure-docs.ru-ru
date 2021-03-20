---
title: Полезные диагностические запросы — масштабирование (Цитус) — база данных Azure для PostgreSQL
description: Запросы на изучение распределенных данных и многое другое
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 1/5/2021
ms.openlocfilehash: 4858f650aca1b704ac79482e0158fd83fc0264b8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98165247"
---
# <a name="useful-diagnostic-queries"></a>Полезные диагностические запросы

## <a name="finding-which-node-contains-data-for-a-specific-tenant"></a>Поиск узла, содержащего данные для определенного клиента

В случае использования нескольких клиентов можно определить, какой рабочий узел содержит строки для определенного клиента.  Цитус (масштабирование) группирует строки распределенных таблиц в сегменты и размещает каждый сегмент на рабочем узле в группе серверов. 

Предположим, что клиенты нашего приложения — это магазины, и нам нужно найти рабочий узел, содержащий данные для магазина с ИДЕНТИФИКАТОРом 4.  Иными словами, мы хотим найти размещение сегмента, содержащего строки, в столбце распределения которых есть значение 4:

``` postgresql
SELECT shardid, shardstate, shardlength, nodename, nodeport, placementid
  FROM pg_dist_placement AS placement,
       pg_dist_node AS node
 WHERE placement.groupid = node.groupid
   AND node.noderole = 'primary'
   AND shardid = (
     SELECT get_shard_id_for_distribution_column('stores', 4)
   );
```

Выходные данные содержат узел и порт рабочей базы данных.

```
┌─────────┬────────────┬─────────────┬───────────┬──────────┬─────────────┐
│ shardid │ shardstate │ shardlength │ nodename  │ nodeport │ placementid │
├─────────┼────────────┼─────────────┼───────────┼──────────┼─────────────┤
│  102009 │          1 │           0 │ 10.0.0.16 │     5432 │           2 │
└─────────┴────────────┴─────────────┴───────────┴──────────┴─────────────┘
```

## <a name="finding-the-distribution-column-for-a-table"></a>Поиск столбца распределения для таблицы

Каждая распределенная таблица в горизонтальном масштабировании (Цитус) имеет столбец распределения. (Дополнительные сведения см. в разделе [распределенное моделирование данных](concepts-hyperscale-choose-distribution-column.md).) Может быть важно понять, в каком столбце он находится. Например, при присоединении или фильтрации таблиц могут отображаться сообщения об ошибках с такими указаниями, как «добавление фильтра в столбец распределения».

`pg_dist_*`Таблицы в узле координатор содержат различные метаданные о распределенной базе данных. В частности `pg_dist_partition` , содержит сведения о столбце распределения для каждой таблицы. Вы можете использовать удобную служебную функцию для поиска имени столбца распределения из сведений низкого уровня в метаданных. Ниже приведен пример и его выходные данные:

``` postgresql
-- create example table

CREATE TABLE products (
  store_id bigint,
  product_id bigint,
  name text,
  price money,

  CONSTRAINT products_pkey PRIMARY KEY (store_id, product_id)
);

-- pick store_id as distribution column

SELECT create_distributed_table('products', 'store_id');

-- get distribution column name for products table

SELECT column_to_column_name(logicalrelid, partkey) AS dist_col_name
  FROM pg_dist_partition
 WHERE logicalrelid='products'::regclass;
```

Выходные данные примера:

```
┌───────────────┐
│ dist_col_name │
├───────────────┤
│ store_id      │
└───────────────┘
```

## <a name="detecting-locks"></a>Обнаружение блокировок

Этот запрос будет выполняться на всех рабочих узлах и выявляет блокировки, время их открытия и запросы, вызывающие неполадки:

``` postgresql
SELECT run_command_on_workers($cmd$
  SELECT array_agg(
    blocked_statement || ' $ ' || cur_stmt_blocking_proc
    || ' $ ' || cnt::text || ' $ ' || age
  )
  FROM (
    SELECT blocked_activity.query    AS blocked_statement,
           blocking_activity.query   AS cur_stmt_blocking_proc,
           count(*)                  AS cnt,
           age(now(), min(blocked_activity.query_start)) AS "age"
    FROM pg_catalog.pg_locks         blocked_locks
    JOIN pg_catalog.pg_stat_activity blocked_activity
      ON blocked_activity.pid = blocked_locks.pid
    JOIN pg_catalog.pg_locks         blocking_locks
      ON blocking_locks.locktype = blocked_locks.locktype
     AND blocking_locks.DATABASE IS NOT DISTINCT FROM blocked_locks.DATABASE
     AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
     AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
     AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
     AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
     AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
     AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
     AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
     AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
     AND blocking_locks.pid != blocked_locks.pid
    JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
    WHERE NOT blocked_locks.GRANTED
     AND blocking_locks.GRANTED
    GROUP BY blocked_activity.query,
             blocking_activity.query
    ORDER BY 4
  ) a
$cmd$);
```

Выходные данные примера:

```
┌───────────────────────────────────────────────────────────────────────────────────┐
│                               run_command_on_workers                              │
├───────────────────────────────────────────────────────────────────────────────────┤
│ (10.0.0.16,5432,t,"")                                                             │
│ (10.0.0.20,5432,t,"{""update ads_102277 set name = 'new name' where id = 1; $ sel…│
│…ect * from ads_102277 where id = 1 for update; $ 1 $ 00:00:03.729519""}")         │
└───────────────────────────────────────────────────────────────────────────────────┘
```

## <a name="querying-the-size-of-your-shards"></a>Запрос размера сегментов

Этот запрос предоставит вам размер каждого сегмента данной распределенной таблицы с именем `my_distributed_table` :

``` postgresql
SELECT *
FROM run_command_on_shards('my_distributed_table', $cmd$
  SELECT json_build_object(
    'shard_name', '%1$s',
    'size',       pg_size_pretty(pg_table_size('%1$s'))
  );
$cmd$);
```

Выходные данные примера:

```
┌─────────┬─────────┬───────────────────────────────────────────────────────────────────────┐
│ shardid │ success │                                result                                 │
├─────────┼─────────┼───────────────────────────────────────────────────────────────────────┤
│  102008 │ t       │ {"shard_name" : "my_distributed_table_102008", "size" : "2416 kB"}    │
│  102009 │ t       │ {"shard_name" : "my_distributed_table_102009", "size" : "3960 kB"}    │
│  102010 │ t       │ {"shard_name" : "my_distributed_table_102010", "size" : "1624 kB"}    │
│  102011 │ t       │ {"shard_name" : "my_distributed_table_102011", "size" : "4792 kB"}    │
└─────────┴─────────┴───────────────────────────────────────────────────────────────────────┘
```

## <a name="querying-the-size-of-all-distributed-tables"></a>Запрос размера всех распределенных таблиц

Этот запрос возвращает список размеров для каждой распределенной таблицы и размер их индексов.

``` postgresql
SELECT
  tablename,
  pg_size_pretty(
    citus_total_relation_size(tablename::text)
  ) AS total_size
FROM pg_tables pt
JOIN pg_dist_partition pp
  ON pt.tablename = pp.logicalrelid::text
WHERE schemaname = 'public';
```

Выходные данные примера:

```
┌───────────────┬────────────┐
│   tablename   │ total_size │
├───────────────┼────────────┤
│ github_users  │ 39 MB      │
│ github_events │ 98 MB      │
└───────────────┴────────────┘
```

Обратите внимание, что существуют другие функции масштабирования (Цитус) для запроса размера распределенной таблицы. см. раздел [Определение размера таблицы](howto-hyperscale-table-size.md).

## <a name="identifying-unused-indices"></a>Определение неиспользуемых индексов

Следующий запрос будет выявление неиспользуемых индексов на рабочих узлах для заданной распределенной таблицы ( `my_distributed_table` )

``` postgresql
SELECT *
FROM run_command_on_shards('my_distributed_table', $cmd$
  SELECT array_agg(a) as infos
  FROM (
    SELECT (
      schemaname || '.' || relname || '##' || indexrelname || '##'
                 || pg_size_pretty(pg_relation_size(i.indexrelid))::text
                 || '##' || idx_scan::text
    ) AS a
    FROM  pg_stat_user_indexes ui
    JOIN  pg_index i
    ON    ui.indexrelid = i.indexrelid
    WHERE NOT indisunique
    AND   idx_scan < 50
    AND   pg_relation_size(relid) > 5 * 8192
    AND   (schemaname || '.' || relname)::regclass = '%s'::regclass
    ORDER BY
      pg_relation_size(i.indexrelid) / NULLIF(idx_scan, 0) DESC nulls first,
      pg_relation_size(i.indexrelid) DESC
  ) sub
$cmd$);
```

Выходные данные примера:

```
┌─────────┬─────────┬───────────────────────────────────────────────────────────────────────┐
│ shardid │ success │                            result                                     │
├─────────┼─────────┼───────────────────────────────────────────────────────────────────────┤
│  102008 │ t       │                                                                       │
│  102009 │ t       │ {"public.my_distributed_table_102009##some_index_102009##28 MB##0"}   │
│  102010 │ t       │                                                                       │
│  102011 │ t       │                                                                       │
└─────────┴─────────┴───────────────────────────────────────────────────────────────────────┘
```

## <a name="monitoring-client-connection-count"></a>Наблюдение за числом подключений клиентов

Следующий запрос подсчитывает число открытых подключений к координатору и группирует их по типу.

``` sql
SELECT state, count(*)
FROM pg_stat_activity
GROUP BY state;
```

Выходные данные примера:

```
┌────────┬───────┐
│ state  │ count │
├────────┼───────┤
│ active │     3 │
│ idle   │     3 │
│ ∅      │     6 │
└────────┴───────┘
```

## <a name="index-hit-rate"></a>Количество попаданий в индекс

Этот запрос обеспечит скорость попадания индекса по всем узлам. Скорость попадания индекса полезна при определении частоты использования индексов при выполнении запросов:

``` postgresql
SELECT nodename, result as index_hit_rate
FROM run_command_on_workers($cmd$
  SELECT CASE sum(idx_blks_hit)
    WHEN 0 THEN 'NaN'::numeric
    ELSE to_char((sum(idx_blks_hit) - sum(idx_blks_read)) / sum(idx_blks_hit + idx_blks_read), '99.99')::numeric
    END AS ratio
  FROM pg_statio_user_indexes
$cmd$);
```

Выходные данные примера:

```
┌───────────┬────────────────┐
│ nodename  │ index_hit_rate │
├───────────┼────────────────┤
│ 10.0.0.16 │ 0.88           │
│ 10.0.0.20 │ 0.89           │
└───────────┴────────────────┘
```

## <a name="cache-hit-rate"></a>Частота попаданий в кэш

Большинство приложений обычно обращаются к небольшой части итоговых данных одновременно. PostgreSQL сохраняет часто запрашиваемые данные в памяти, чтобы избежать снижения скорости чтения с диска. В представлении [pg_statio_user_tables](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STATIO-ALL-TABLES-VIEW) можно просмотреть статистические данные о ней.

Важно отметить, какой процент данных поступает из кэша памяти в сравнении с диском в рабочей нагрузке.

``` postgresql
SELECT
  sum(heap_blks_read) AS heap_read,
  sum(heap_blks_hit)  AS heap_hit,
  sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) AS ratio
FROM
  pg_statio_user_tables;
```

Выходные данные примера:

```
 heap_read | heap_hit |         ratio
-----------+----------+------------------------
         1 |      132 | 0.99248120300751879699
```

Если вы считаете, что соотношение менее чем на 99%, вам, вероятно, потребуется увеличить объем кэша, доступный для базы данных.

## <a name="next-steps"></a>Дальнейшие действия

* Сведения о других [системных таблицах](reference-hyperscale-metadata.md) , полезных для диагностики
