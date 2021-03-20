---
title: Совместное размещение таблицы — масштабирование (Цитус) — база данных Azure для PostgreSQL
description: Как хранить связанные данные в сочетании для ускорения запросов
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 05/06/2019
ms.openlocfilehash: 842563319e09a001fd6e85403d8aee6fb14690ee
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90884431"
---
# <a name="table-colocation-in-azure-database-for-postgresql--hyperscale-citus"></a>Совместное размещение таблиц в базе данных Azure для PostgreSQL — масштабирование (Цитус)

Совместное размещение означает хранение связанных данных на одних и тех же узлах. Запросы могут работать быстро, когда все необходимые данные доступны без какого-либо сетевого трафика. Совместное размещение взаимосвязанных данных на разных узлах позволяет эффективно выполнять запросы на каждом узле.

## <a name="data-colocation-for-hash-distributed-tables"></a>Совместное размещение данных для таблиц, распределенных по хэшу

В службе "база данных Azure для PostgreSQL — масштабирование" (Цитус) строка хранится в сегменте, если хэш значения в столбце распределения попадает в диапазон хэша сегмента. Сегменты с одинаковым диапазоном хэша всегда размещаются на одном узле. Строки с одинаковыми значениями столбца распределения всегда находятся на одном узле в разных таблицах.

:::image type="content" source="media/concepts-hyperscale-colocation/colocation-shards.png" alt-text="Сегментов":::

## <a name="a-practical-example-of-colocation"></a>Практичный пример совместного размещения

Рассмотрим следующие таблицы, которые могут входить в состав клиентского веб-аналитики SaaS:

```sql
CREATE TABLE event (
  tenant_id int,
  event_id bigint,
  page_id int,
  payload jsonb,
  primary key (tenant_id, event_id)
);

CREATE TABLE page (
  tenant_id int,
  page_id int,
  path text,
  primary key (tenant_id, page_id)
);
```

Теперь мы хотим ответить на запросы, которые могут выдаваться на панели мониторинга, ориентированной на клиента. Пример запроса: "Возврат числа посещений за прошлую неделю для всех страниц, начинающихся с"/блог "в клиенте" шесть "".

Если наши данные были в Single-Server развертывания, можно легко выразить наш запрос с помощью обширного набора реляционных операций, предлагаемых SQL:

```sql
SELECT page_id, count(event_id)
FROM
  page
LEFT JOIN  (
  SELECT * FROM event
  WHERE (payload->>'time')::timestamptz >= now() - interval '1 week'
) recent
USING (tenant_id, page_id)
WHERE tenant_id = 6 AND path LIKE '/blog%'
GROUP BY page_id;
```

Пока [Рабочий набор](https://en.wikipedia.org/wiki/Working_set) для этого запроса не умещается в памяти, подходящим решением является таблица с одним сервером. Рассмотрим возможности масштабирования модели данных с помощью параметра развертывания с помощью функции масштабирования (Цитус).

### <a name="distribute-tables-by-id"></a>Распределение таблиц по ИДЕНТИФИКАТОРу

Запросы с одним сервером начинают замедляться по мере роста числа клиентов и данных, хранимых для каждого клиента. Рабочий набор прекращает подгонку памяти, и ЦП становится узким местом.

В этом случае мы можем сегментировановать данные между несколькими узлами с помощью функции масштабирования (Цитус). Первый и самый важный выбор, который необходимо принять, когда мы решили сегментирование, — столбец распределения. Начнем с упрощенного выбора использования `event_id` для таблицы событий и `page_id` для `page` таблицы:

```sql
-- naively use event_id and page_id as distribution columns

SELECT create_distributed_table('event', 'event_id');
SELECT create_distributed_table('page', 'page_id');
```

Если данные распределены между разными рабочими процессами, мы не можем выполнить соединение, как в одном PostgreSQL узле. Вместо этого необходимо выполнить два запроса:

```sql
-- (Q1) get the relevant page_ids
SELECT page_id FROM page WHERE path LIKE '/blog%' AND tenant_id = 6;

-- (Q2) get the counts
SELECT page_id, count(*) AS count
FROM event
WHERE page_id IN (/*…page IDs from first query…*/)
  AND tenant_id = 6
  AND (payload->>'time')::date >= now() - interval '1 week'
GROUP BY page_id ORDER BY count DESC LIMIT 10;
```

После этого результаты выполнения двух шагов должны быть объединены приложением.

Выполнение запросов должно обращаться к данным в сегментах, разбросанных по узлам.

:::image type="content" source="media/concepts-hyperscale-colocation/colocation-inefficient-queries.png" alt-text="Неэффективные запросы":::

В этом случае распределение данных создает существенные недостатки.

-   Издержки от запроса каждого сегмента и выполнения нескольких запросов.
-   Затраты на Q1 возвращают множество строк клиенту.
-   Q2 станет большим.
-   Необходимость записи запросов в несколько шагов требует внесения изменений в приложение.

Данные распределены, поэтому запросы могут быть параллельными. Это полезно только в том случае, если объем работы, выполненной запросом, значительно превышает нагрузку, связанную с запросом многих сегментов.

### <a name="distribute-tables-by-tenant"></a>Распределение таблиц по клиентам

В масштабе Цитус строки с одинаковым значением столбца распределения гарантированно будут находиться на одном узле. Начиная с начала, мы можем создавать таблицы с помощью в `tenant_id` качестве столбца распределения.

```sql
-- co-locate tables by using a common distribution column
SELECT create_distributed_table('event', 'tenant_id');
SELECT create_distributed_table('page', 'tenant_id', colocate_with => 'event');
```

Теперь масштабирование (Цитус) может ответить на исходный односерверный запрос без изменения (Q1):

```sql
SELECT page_id, count(event_id)
FROM
  page
LEFT JOIN  (
  SELECT * FROM event
  WHERE (payload->>'time')::timestamptz >= now() - interval '1 week'
) recent
USING (tenant_id, page_id)
WHERE tenant_id = 6 AND path LIKE '/blog%'
GROUP BY page_id;
```

Из-за фильтрации и объединения в tenant_id, Цитуса, чтобы получить ответ на весь запрос, можно использовать набор соразмещенных сегментов, содержащих данные для конкретного клиента. Один PostgreSQL узел может ответить на запрос за один шаг.

:::image type="content" source="media/concepts-hyperscale-colocation/colocation-better-query.png" alt-text="Улучшенный запрос":::

В некоторых случаях необходимо изменить запросы и схемы таблиц, чтобы включить идентификатор клиента в ограничения UNIQUE и условия JOIN. Это изменение обычно является простым.

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как данные клиента сохранятся в [учебнике по нескольким клиентам](tutorial-design-database-hyperscale-multi-tenant.md).
