---
title: Изменение распределенных таблиц — масштабирование (Цитус) — база данных Azure для PostgreSQL
description: Команды SQL для создания и изменения распределенных таблиц (Цитус) с помощью портал Azure
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 8/10/2020
ms.openlocfilehash: cf9f9ca5b8690a38c6e5aa6f519378c0a2e3a4f2
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95026443"
---
# <a name="distribute-and-modify-tables"></a>Распространение и изменение таблиц

## <a name="distributing-tables"></a>Распространение таблиц

Чтобы создать распределенную таблицу, необходимо сначала определить схему таблицы. Для этого можно определить таблицу с помощью оператора [CREATE TABLE](http://www.postgresql.org/docs/current/static/sql-createtable.html) так же, как и для обычной таблицы PostgreSQL.

```sql
CREATE TABLE github_events
(
    event_id bigint,
    event_type text,
    event_public boolean,
    repo_id bigint,
    payload jsonb,
    repo jsonb,
    actor jsonb,
    org jsonb,
    created_at timestamp
);
```

Затем можно использовать функцию Created \_ \_ Table () для указания столбца распределения таблицы и создания рабочих сегментов.

```sql
SELECT create_distributed_table('github_events', 'repo_id');
```

Вызов функции информирует функцию "масштабирование" (Цитус) о том, что \_ Таблица событий GitHub должна быть распределена по \_ столбцу идентификаторов репозитория (путем хэширования значения столбца). Эта функция также создает сегменты на рабочих узлах с помощью \_ \_ \_ значений конфигурации коэффициента репликации Цитус. сегментирования и Цитус. сегментирования.

Он создает сумму Цитус. сегментирования \_ количества сегментов, где каждый сегмент владеет частью хэш-пространства и реплицируется на основе \_ \_ значения конфигурации коэффициента репликации Цитус. сегментирования по умолчанию. Реплики сегментов, созданные в рабочей роли, имеют те же схемы таблицы, индексы и определения ограничений, что и таблица в координаторе. После создания реплик функция сохраняет все распределенные метаданные в координаторе.

Каждому созданному сегменту назначается уникальный идентификатор сегмента, и все его реплики имеют одинаковый идентификатор сегмента. Сегменты представлены на рабочем узле как обычные таблицы PostgreSQL с именем \' TableName \_ шардид \' , где TableName — это имя распределенной таблицы, а идентификатор сегмента — присвоенный уникальный идентификатор. Вы можете подключиться к экземплярам Worker postgres, чтобы просматривать или выполнять команды в отдельных сегментах.

Теперь все готово для вставки данных в распределенную таблицу и выполнения запросов к ней. Дополнительные сведения об определяемой пользователем функции, используемой в этом разделе, см. в справочнике по [DDL для таблиц и сегментов](reference-hyperscale-functions.md#table-and-shard-ddl) .

### <a name="reference-tables"></a>Ссылочные таблицы

Приведенный выше метод распределяет таблицы по нескольким горизонтальным сегментам.  Другой возможностью является распространение таблиц в один сегмент и репликация сегмента на каждый рабочий узел. Таблицы, распределенные таким образом, называются *ссылочными таблицами.* Они используются для хранения данных, к которым часто приходится обращаться несколько узлов в кластере.

К общим кандидатам для ссылочных таблиц относятся:

-   Небольшие таблицы, которые должны быть соединены с большими распределенными таблицами.
-   Таблицы в приложениях с несколькими клиентами, в которых отсутствует столбец идентификатора клиента или недопустимые \' t, связанный с клиентом. (Или, во время миграции, даже для некоторых таблиц, связанных с клиентом.)
-   Таблицы, которым требуются уникальные ограничения для нескольких столбцов и которые достаточно малы.

Например, предположим, что многоклиентский сайт электронной коммерции должен рассчитывать налог на продажи для транзакций в любом из своих магазинов. Сведения о налогах не \' относятся к какому бы то ни было клиенту. Имеет смысл разместить его в общей таблице. Справочная таблица, ориентированная на US, может выглядеть следующим образом:

```postgresql
-- a reference table

CREATE TABLE states (
  code char(2) PRIMARY KEY,
  full_name text NOT NULL,
  general_sales_tax numeric(4,3)
);

-- distribute it to all workers

SELECT create_reference_table('states');
```

Теперь такие запросы, как один расчет налога для покупательской корзины, могут присоединиться к `states` таблице без дополнительной нагрузки, а также добавить внешний ключ в код состояния для лучшей проверки.

Помимо распределения таблицы в качестве одного реплицированного сегмента, `create_reference_table` UDF помечает ее как ссылочную таблицу в таблицах метаданных Цитус. Цитус автоматически выполняет две фазы фиксации ([2pc](https://en.wikipedia.org/wiki/Two-phase_commit_protocol)) для изменений в таблицах, помеченных таким образом, что обеспечивает строгую гарантию согласованности.

Если имеется распределенная таблица с количеством сегментов, то можно обновить ее, чтобы она была распознанной ссылочной таблицей следующим образом:

```postgresql
SELECT upgrade_to_reference_table('table_name');
```

Еще один пример использования ссылочных таблиц см. в руководстве по работе с [несколькими клиентами](tutorial-design-database-hyperscale-multi-tenant.md).

### <a name="distributing-coordinator-data"></a>Распространение данных координатора

Если существующая база данных PostgreSQL преобразуется в узел координатора для кластера масштабирования (Цитус), данные в его таблицах могут быть распределены эффективно и с минимальным перерывом в приложении.

`create_distributed_table`Описанная выше функция работает как с пустыми, так и с непустыми таблицами, а для последней она автоматически распределяет строки таблицы по всему кластеру. Вы узнаете, копирует ли данные по наличию сообщения, \" Обратите внимание: копирование данных из локальной таблицы \. . \" Например:

```postgresql
CREATE TABLE series AS SELECT i FROM generate_series(1,1000000) i;
SELECT create_distributed_table('series', 'i');
NOTICE:  Copying data from local table...
 create_distributed_table
 --------------------------

 (1 row)
```

Операции записи в таблицу блокируются во время переноса данных, а ожидающие операции записи обрабатываются как распределенные запросы после фиксации функции. (Если функция завершается ошибкой, запросы снова становятся локальными.) Операции чтения могут продолжаться как обычные и перестанут распределенными запросами после фиксации функции.

При распространении таблиц A и B, где A имеет внешний ключ к B, сначала распределите целевую таблицу с ключом B. Выполнение этого действия в неправильном порядке приведет к ошибке:

```
ERROR:  cannot create foreign key constraint
DETAIL:  Referenced table must be a distributed table or a reference table.
```

Если невозможно выполнить распределение в правильном порядке, удалите внешние ключи, распределите таблицы и повторно создайте внешние ключи.

При переносе данных из внешней базы данных, например из Amazon RDS в горизонтальное масштабирование (Цитус), сначала создайте распределенные таблицы с помощью технологии Scale (Цитус) через `create_distributed_table` , а затем скопируйте данные в таблицу.
Копирование в распределенные таблицы позволяет избежать нехватки пространства на узле координатора.

## <a name="colocating-tables"></a>Совместное размещение таблиц

Совместное размещение означает размещение связанных данных на одних и тех же компьютерах. Он обеспечивает эффективные запросы, одновременно используя возможности горизонтальной масштабируемости для всего набора данных. Дополнительные сведения см. в разделе Совместное [Размещение](concepts-hyperscale-colocation.md).

Таблицы размещаются в группах. Чтобы вручную управлять назначением группы совместного размещения таблицы, используйте необязательный `colocate_with` параметр `create_distributed_table` . Если вы не хотите беспокоиться \' о совместном \' размещении таблиц, этот параметр следует опустить. По умолчанию используется значение `'default'` , которое группирует таблицу с любой другой таблицей совместного размещения по умолчанию, имеющей тот же тип столбца распределения, число сегментов и фактор репликации. Если вы хотите прервать или обновить это неявное совместное размещение, можно использовать `update_distributed_table_colocation()` .

```postgresql
-- these tables are implicitly co-located by using the same
-- distribution column type and shard count with the default
-- co-location group

SELECT create_distributed_table('A', 'some_int_col');
SELECT create_distributed_table('B', 'other_int_col');
```

Если новая таблица не связана с другими, в ее качестве используется неявная коположенная группа, укажите `colocated_with => 'none'` .

```postgresql
-- not co-located with other tables

SELECT create_distributed_table('A', 'foo', colocate_with => 'none');
```

Разделение несвязанных таблиц на их собственные группы совместного размещения повысит производительность повторного [распределения сегментов](howto-hyperscale-scale-rebalance.md) , так как сегменты в одной группе должны перемещаться вместе.

Если таблицы действительно связаны (например, когда они будут присоединены), может быть целесообразно их явное размещение. Увеличение соответствующего совместного местоположения является более важным, чем затраты на перераспределение.

Чтобы явным образом разместить несколько таблиц, распределите их, а затем поместите в свою группу совместного размещения. Пример:

```postgresql
-- distribute stores
SELECT create_distributed_table('stores', 'store_id');

-- add to the same group as stores
SELECT create_distributed_table('orders', 'store_id', colocate_with => 'stores');
SELECT create_distributed_table('products', 'store_id', colocate_with => 'stores');
```

Сведения о группах совместного размещения хранятся в таблице [pg_dist_colocation](reference-hyperscale-metadata.md#colocation-group-table) , в то время как [pg_dist_partition](reference-hyperscale-metadata.md#partition-table) показывает, какие таблицы назначены группам.

## <a name="dropping-tables"></a>Удаление таблиц

Для удаления распределенных таблиц можно использовать стандартную команду DROP TABLE PostgreSQL. Как и в случае с обычными таблицами, инструкция DROP TABLE удаляет все индексы, правила, триггеры и ограничения, которые существуют для целевой таблицы. Кроме того, он удаляет сегменты на рабочих узлах и очищает их метаданные.

```sql
DROP TABLE github_events;
```

## <a name="modifying-tables"></a>Изменение таблиц

Масштабирование (Цитус) автоматически распространяет множество типов инструкций DDL.
Изменение распределенной таблицы на узле координатор приведет к обновлению сегментов на рабочих процессах. Для других инструкций DDL требуется распространение вручную, а некоторые другие запрещены, например, при изменении столбца распределения.
При попытке выполнить DDL, который не подходит для автоматического распространения, произойдет ошибка и таблицы на узле координатора останутся без изменений.

Ниже приведена ссылка на категории инструкций DDL, которые распространяют.
Автоматическое распространение можно включить или отключить с помощью [параметра конфигурации](reference-hyperscale-parameters.md#citusenable_ddl_propagation-boolean) .

### <a name="addingmodifying-columns"></a>Добавление и изменение столбцов

Масштабирование (Цитус) автоматически распространяет большинство команд [ALTER TABLE](https://www.postgresql.org/docs/current/static/ddl-alter.html) . Добавление столбцов или изменение их значений по умолчанию работает так же, как в базе данных PostgreSQL с одним компьютером:

```postgresql
-- Adding a column

ALTER TABLE products ADD COLUMN description text;

-- Changing default value

ALTER TABLE products ALTER COLUMN price SET DEFAULT 7.77;
```

Значительные изменения в существующем столбце, такие как переименование или изменение типа данных, также являются очень точным. Однако тип данных [столбца распределения](concepts-hyperscale-nodes.md#distribution-column) изменить нельзя.
В этой статье определяется, каким образом данные таблицы распределяются через кластер Цитус (масштабирование), а изменение типа данных требует перемещения данных.

Попытка сделать это приведет к ошибке:

```postgres
-- assumining store_id is the distribution column
-- for products, and that it has type integer

ALTER TABLE products
ALTER COLUMN store_id TYPE text;

/*
ERROR:  XX000: cannot execute ALTER TABLE command involving partition column
LOCATION:  ErrorIfUnsupportedAlterTableStmt, multi_utility.c:2150
*/
```

### <a name="addingremoving-constraints"></a>Добавление и удаление ограничений

Использование функции масштабирования (Цитус) позволяет продолжить безопасность реляционной базы данных, включая ограничения базы данных (см. [документацию](https://www.postgresql.org/docs/current/static/ddl-constraints.html)по PostgreSQL).
Из-за природы распределенных систем средство масштабирования (Цитус) не будет пересекать ограничения уникальности или ссылочную целостность между рабочими узлами.

Чтобы настроить внешний ключ между соразмещенными распределенными таблицами, всегда включайте столбец распределения в ключ. Включение столбца распределения может привести к созданию составного ключа.

Внешние ключи могут быть созданы в следующих ситуациях:

-   между двумя локальными (не распределенными) таблицами;
-   между двумя ссылочными таблицами:
-   между двумя [соразмещенными](concepts-hyperscale-colocation.md) распределенными таблицами, когда ключ включает столбец распределения, или
-   как распределенная таблица, ссылающаяся на [ссылочную таблицу](concepts-hyperscale-nodes.md#type-2-reference-tables)

Внешние ключи из ссылочных таблиц в распределенные таблицы не поддерживаются.

> [!NOTE]
>
> Первичные ключи и ограничения уникальности должны включать столбец распределения. Если добавить их в столбец без распределения, будет выдаваться ошибка

В этом примере показано, как создавать первичные и внешние ключи в распределенных таблицах.

```postgresql
--
-- Adding a primary key
-- --------------------

-- We'll distribute these tables on the account_id. The ads and clicks
-- tables must use compound keys that include account_id.

ALTER TABLE accounts ADD PRIMARY KEY (id);
ALTER TABLE ads ADD PRIMARY KEY (account_id, id);
ALTER TABLE clicks ADD PRIMARY KEY (account_id, id);

-- Next distribute the tables

SELECT create_distributed_table('accounts', 'id');
SELECT create_distributed_table('ads',      'account_id');
SELECT create_distributed_table('clicks',   'account_id');

--
-- Adding foreign keys
-- -------------------

-- Note that this can happen before or after distribution, as long as
-- there exists a uniqueness constraint on the target column(s) which
-- can only be enforced before distribution.

ALTER TABLE ads ADD CONSTRAINT ads_account_fk
  FOREIGN KEY (account_id) REFERENCES accounts (id);
ALTER TABLE clicks ADD CONSTRAINT clicks_ad_fk
  FOREIGN KEY (account_id, ad_id) REFERENCES ads (account_id, id);
```

Аналогичным образом включите столбец распределения в ограничения уникальности:

```postgresql
-- Suppose we want every ad to use a unique image. Notice we can
-- enforce it only per account when we distribute by account id.

ALTER TABLE ads ADD CONSTRAINT ads_unique_image
  UNIQUE (account_id, image_url);
```

Ограничения NOT NULL могут применяться к любому столбцу (распространение или нет), так как они не нуждаются в подстановке между рабочими процессами.

```postgresql
ALTER TABLE ads ALTER COLUMN image_url SET NOT NULL;
```

### <a name="using-not-valid-constraints"></a>Использование недопустимых ограничений

В некоторых ситуациях может быть полезно применять ограничения для новых строк, в то же время позволяя существующим несогласованным строкам оставаться без изменений. Функция "масштабирование" (Цитус) поддерживает эту функцию для проверочных ограничений и внешних ключей с помощью PostgreSQL \' s \" не является допустимым \" ограничением.

Например, рассмотрим приложение, которое хранит профили пользователей в [ссылочной таблице](concepts-hyperscale-nodes.md#type-2-reference-tables).

```postgres
-- we're using the "text" column type here, but a real application
-- might use "citext" which is available in a postgres contrib module

CREATE TABLE users ( email text PRIMARY KEY );
SELECT create_reference_table('users');
```

В процессе Представьте себе, что в таблицу получится несколько неадресов.

```postgres
INSERT INTO users VALUES
   ('foo@example.com'), ('hacker12@aol.com'), ('lol');
```

Мы хотели бы проверить адреса, но PostgreSQL не позволяет добавить проверочное ограничение, которое завершается ошибкой для существующих строк. Однако он *разрешает* ограничение, помеченное как недопустимое:

```postgres
ALTER TABLE users
ADD CONSTRAINT syntactic_email
CHECK (email ~
   '^[a-zA-Z0-9.!#$%&''*+/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$'
) NOT VALID;
```

Теперь новые строки защищены.

```postgres
INSERT INTO users VALUES ('fake');

/*
ERROR:  new row for relation "users_102010" violates
        check constraint "syntactic_email_102010"
DETAIL:  Failing row contains (fake).
*/
```

Позже, в непиковых часах, администратор базы данных может попытаться исправить неправильные строки и повторно проверить ограничение.

```postgres
-- later, attempt to validate all rows
ALTER TABLE users
VALIDATE CONSTRAINT syntactic_email;
```

В документации PostgreSQL содержатся дополнительные сведения о недопустимых и ПРОВЕРЯЕМых ОГРАНИЧЕНИЯх в разделе [ALTER TABLE](https://www.postgresql.org/docs/current/sql-altertable.html) .

### <a name="addingremoving-indices"></a>Добавление и удаление индексов

Масштабирование (Цитус) поддерживает добавление и удаление [индексов](https://www.postgresql.org/docs/current/static/sql-createindex.html):

```postgresql
-- Adding an index

CREATE INDEX clicked_at_idx ON clicks USING BRIN (clicked_at);

-- Removing an index

DROP INDEX clicked_at_idx;
```

Добавление индекса выполняет блокировку записи, которая может быть нежелательной в многопользовательской системе с несколькими клиентами \" . \" Чтобы сократить время простоя приложения, вместо этого создайте индекс [одновременно](https://www.postgresql.org/docs/current/static/sql-createindex.html#SQL-CREATEINDEX-CONCURRENTLY) . Этот метод требует больше общего объема работы, чем построение стандартного индекса и требует больше времени на выполнение. Однако, так как она позволяет продолжать нормальную работу во время построения индекса, этот метод удобен для добавления новых индексов в рабочей среде.

```postgresql
-- Adding an index without locking table writes

CREATE INDEX CONCURRENTLY clicked_at_idx ON clicks USING BRIN (clicked_at);
```
