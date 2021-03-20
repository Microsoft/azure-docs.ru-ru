---
title: Расширения базы данных Azure для PostgreSQL — один сервер
description: Сведения о доступных расширениях postgres в базе данных Azure для PostgreSQL-Single Server
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/14/2020
ms.openlocfilehash: 78395873457f9fe53d45dfbfd94aa9ccdccd614d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92485466"
---
# <a name="postgresql-extensions-in-azure-database-for-postgresql---single-server"></a>Расширения PostgreSQL в Базе данных Azure для PostgreSQL — один сервер
PostgreSQL предоставляет возможность расширить функциональность базы данных с помощью расширений. Расширения позволяют объединить несколько связанных объектов SQL в один пакет, чтобы загружать и удалять их из базы данных одной командой. После загрузки в базу данных расширения действуют как встроенные функции.

## <a name="how-to-use-postgresql-extensions"></a>Как использовать расширения PostgreSQL
Чтобы использовать расширения PostgreSQL, их необходимо сначала установить в базу данных. Чтобы установить определенное расширение, выполните команду [CREATE EXTENSION](https://www.postgresql.org/docs/current/sql-createextension.html), используя инструмент psql. При этом упакованные объекты будут загружены в базу данных.

База данных Azure для PostgreSQL поддерживает подмножество расширений ключей, как указано ниже. Эти сведения также можно получить, выполнив `SELECT * FROM pg_available_extensions;`. Расширения, выходящие за рамки перечисленных, не поддерживаются. Вы не можете создать собственное расширение в базе данных Azure для PostgreSQL.

## <a name="postgres-11-extensions"></a>Расширения postgres 11

В базе данных Azure для серверов PostgreSQL доступны следующие расширения с Postgres версии 11. 

> [!div class="mx-tableFixed"]
> | **Расширение**| **версия расширения;** | **Описание** |
> |---|---|---|
> |[address_standardizer](http://postgis.net/docs/Address_Standardizer.html)         | 2.5.1           | Используются для анализа адреса в составных элементы. |
> |[address_standardizer_data_us](http://postgis.net/docs/Address_Standardizer.html) | 2.5.1           | Пример набора данных для стандартизации адресов США|
> |[btree_gin](https://www.postgresql.org/docs/11/btree-gin.html)                    | 1,3             | Поддержка индексирования общих типов в Ло|
> |[btree_gist](https://www.postgresql.org/docs/11/btree-gist.html)                   | 1.5             | Поддержка индексирования общих типов в реестрах|
> |[citext](https://www.postgresql.org/docs/11/citext.html)                       | 1.5             | тип данных для символьных строк без учета регистра|
> |[Куба](https://www.postgresql.org/docs/11/cube.html)                         | 1.4             | тип данных для многомерных кубов|
> |[dblink](https://www.postgresql.org/docs/11/dblink.html)                       | 1.2             | подключение к другим базам данных PostgreSQL из базы данных|
> |[dict_int](https://www.postgresql.org/docs/11/dict-int.html)                     | 1.0             | шаблон словаря поиска текста для целых чисел|
> |[earthdistance](https://www.postgresql.org/docs/11/earthdistance.html)                | 1,1             | Вычислите большие расстояния на поверхности земли|
> |[fuzzystrmatch](https://www.postgresql.org/docs/11/fuzzystrmatch.html)                | 1,1             | определение сходства и расстояния между строками|
> |[hstore](https://www.postgresql.org/docs/11/hstore.html)                       | 1.5             | тип данных для хранения наборов пар (ключ, значение)|
> |[hypopg](https://hypopg.readthedocs.io/en/latest/)                       | 1.1.2           | Гипотетические индексы для PostgreSQL|
> |[intarray](https://www.postgresql.org/docs/11/intarray.html)                     | 1.2             | функции, операторы и поддержка индексов для 1-D массивов целых чисел|
> |[isn](https://www.postgresql.org/docs/11/isn.html)                          | 1.2             | типы данных для международных стандартов нумерации продуктов|
> |[ltree](https://www.postgresql.org/docs/11/ltree.html)                        | 1,1             | тип данных для иерархических структур, схожих с деревом|
> |[орафце](https://github.com/orafce/orafce)                       | 3,7             | Функции и операторы, имитирующие подмножество функций и пакетов из коммерческой RDBMS|
> |[пгаудит](https://www.pgaudit.org/)                     | 1.3.1             | предоставляет функции аудита|
> |[pgcrypto](https://www.postgresql.org/docs/11/pgcrypto.html)                     | 1,3             | криптографические функции|
> |[pgrouting](https://pgrouting.org/)                    | 2.6.2           | Расширение Пграутинг|
> |[pgrowlocks](https://www.postgresql.org/docs/11/pgrowlocks.html)                   | 1.2             | показывать сведения о блокировке на уровне строк|
> |[pgstattuple](https://www.postgresql.org/docs/11/pgstattuple.html)                  | 1.5             | Отображение статистики на уровне кортежей|
> |[pg_buffercache](https://www.postgresql.org/docs/11/pgbuffercache.html)               | 1,3             | Проверка общего кэша буфера|
> |[pg_partman](https://github.com/pgpartman/pg_partman)                   | 4.0.0           | Расширение для управления секционированными таблицами по времени или ИДЕНТИФИКАТОРу|
> |[pg_prewarm](https://www.postgresql.org/docs/11/pgprewarm.html)                   | 1.2             | предтеплое отношение данных|
> |[pg_stat_statements](https://www.postgresql.org/docs/11/pgstatstatements.html)           | 1.6             | Трассировка статистики выполнения всех выполненных инструкций SQL|
> |[pg_trgm](https://www.postgresql.org/docs/11/pgtrgm.html)                      | 1.4             | измерение подобия текста и поиск по индексу на основе триграммов|
> |[plpgsql](https://www.postgresql.org/docs/11/plpgsql.html)                      | 1.0             | Процедурный язык PL/pgSQL|
> |[plv8](https://plv8.github.io/)                         | 2.3.11          | Доверенный процедурный язык PL/JavaScript (V8)|
> |[PostGIS](https://www.postgis.net/)                      | 2.5.1           | PostGISие геометрических, географических и растровых пространственных типов и функций|
> |[postgis_sfcgal](https://www.postgis.net/)               | 2.5.1           | Функции СФКГАЛ PostGIS|
> |[postgis_tiger_geocoder](https://www.postgis.net/)       | 2.5.1           | PostGIS Tiger для геокодирования и инвертированного геокодирования|
> |[postgis_topology](https://postgis.net/docs/Topology.html)             | 2.5.1           | Пространственные типы и функции топологии PostGIS|
> |[postgres_fdw](https://www.postgresql.org/docs/11/postgres-fdw.html)                 | 1.0             | оболочка внешних данных для удаленных серверов PostgreSQL|
> |[tablefunc](https://www.postgresql.org/docs/11/tablefunc.html)                    | 1.0             | функции, управляющие целыми таблицами, включая перекрестный|
> |[тимескаледб](https://docs.timescale.com/latest)                    | 1.3.2             | Включает масштабируемые операции вставки и сложные запросы для данных временных рядов|
> |[unaccent](https://www.postgresql.org/docs/11/unaccent.html)                     | 1,1             | словарь поиска текста, который удаляет диакритические знаки|
> |[uuid-ossp](https://www.postgresql.org/docs/11/uuid-ossp.html)                    | 1,1             | создать универсальные уникальные идентификаторы (UUID)|

## <a name="postgres-10-extensions"></a>Расширения postgres 10 

В базе данных Azure для серверов PostgreSQL доступны следующие расширения с Postgres версии 10.

> [!div class="mx-tableFixed"]
> | **Расширение**| **версия расширения;** | **Описание** |
> |---|---|---|
> |[address_standardizer](http://postgis.net/docs/Address_Standardizer.html)         | 2.5.1           | Используются для анализа адреса в составных элементы. |
> |[address_standardizer_data_us](http://postgis.net/docs/Address_Standardizer.html) | 2.5.1           | Пример набора данных для стандартизации адресов США|
> |[btree_gin](https://www.postgresql.org/docs/10/btree-gin.html)                    | 1,3             | Поддержка индексирования общих типов в Ло|
> |[btree_gist](https://www.postgresql.org/docs/10/btree-gist.html)                   | 1.5             | Поддержка индексирования общих типов в реестрах|
> |[chkpass](https://www.postgresql.org/docs/10/chkpass.html)                       | 1.0             | тип данных для автоматического шифрования паролей|
> |[citext](https://www.postgresql.org/docs/10/citext.html)                       | 1.4             | тип данных для символьных строк без учета регистра|
> |[Куба](https://www.postgresql.org/docs/10/cube.html)                         | 1.2             | тип данных для многомерных кубов|
> |[dblink](https://www.postgresql.org/docs/10/dblink.html)                       | 1.2             | подключение к другим базам данных PostgreSQL из базы данных|
> |[dict_int](https://www.postgresql.org/docs/10/dict-int.html)                     | 1.0             | шаблон словаря поиска текста для целых чисел|
> |[earthdistance](https://www.postgresql.org/docs/10/earthdistance.html)                | 1,1             | Вычислите большие расстояния на поверхности земли|
> |[fuzzystrmatch](https://www.postgresql.org/docs/10/fuzzystrmatch.html)                | 1,1             | определение сходства и расстояния между строками|
> |[hstore](https://www.postgresql.org/docs/10/hstore.html)                       | 1.4             | тип данных для хранения наборов пар (ключ, значение)|
> |[hypopg](https://hypopg.readthedocs.io/en/latest/)                       | 1.1.1           | Гипотетические индексы для PostgreSQL|
> |[intarray](https://www.postgresql.org/docs/10/intarray.html)                     | 1.2             | функции, операторы и поддержка индексов для 1-D массивов целых чисел|
> |[isn](https://www.postgresql.org/docs/10/isn.html)                          | 1,1             | типы данных для международных стандартов нумерации продуктов|
> |[ltree](https://www.postgresql.org/docs/10/ltree.html)                        | 1,1             | тип данных для иерархических структур, схожих с деревом|
> |[орафце](https://github.com/orafce/orafce)                       | 3,7             | Функции и операторы, имитирующие подмножество функций и пакетов из коммерческой RDBMS|
> |[пгаудит](https://www.pgaudit.org/)                     | 1.2             | предоставляет функции аудита|
> |[pgcrypto](https://www.postgresql.org/docs/10/pgcrypto.html)                     | 1,3             | криптографические функции|
> |[pgrouting](https://pgrouting.org/)                    | 2.5.2           | Расширение Пграутинг|
> |[pgrowlocks](https://www.postgresql.org/docs/10/pgrowlocks.html)                   | 1.2             | показывать сведения о блокировке на уровне строк|
> |[pgstattuple](https://www.postgresql.org/docs/10/pgstattuple.html)                  | 1.5             | Отображение статистики на уровне кортежей|
> |[pg_buffercache](https://www.postgresql.org/docs/10/pgbuffercache.html)               | 1,3             | Проверка общего кэша буфера|
> |[pg_partman](https://github.com/pgpartman/pg_partman)                   | 2.6.3           | Расширение для управления секционированными таблицами по времени или ИДЕНТИФИКАТОРу|
> |[pg_prewarm](https://www.postgresql.org/docs/10/pgprewarm.html)                   | 1,1             | предтеплое отношение данных|
> |[pg_stat_statements](https://www.postgresql.org/docs/10/pgstatstatements.html)           | 1.6             | Трассировка статистики выполнения всех выполненных инструкций SQL|
> |[pg_trgm](https://www.postgresql.org/docs/10/pgtrgm.html)                      | 1,3             | измерение подобия текста и поиск по индексу на основе триграммов|
> |[plpgsql](https://www.postgresql.org/docs/10/plpgsql.html)                      | 1.0             | Процедурный язык PL/pgSQL|
> |[plv8](https://plv8.github.io/)                         | 2.1.0          | Доверенный процедурный язык PL/JavaScript (V8)|
> |[PostGIS](https://www.postgis.net/)                      | 2.4.3           | PostGISие геометрических, географических и растровых пространственных типов и функций|
> |[postgis_sfcgal](https://www.postgis.net/)               | 2.4.3           | Функции СФКГАЛ PostGIS|
> |[postgis_tiger_geocoder](https://www.postgis.net/)       | 2.4.3           | PostGIS Tiger для геокодирования и инвертированного геокодирования|
> |[postgis_topology](https://postgis.net/docs/Topology.html)             | 2.4.3           | Пространственные типы и функции топологии PostGIS|
> |[postgres_fdw](https://www.postgresql.org/docs/10/postgres-fdw.html)                 | 1.0             | оболочка внешних данных для удаленных серверов PostgreSQL|
> |[tablefunc](https://www.postgresql.org/docs/10/tablefunc.html)                    | 1.0             | функции, управляющие целыми таблицами, включая перекрестный|
> |[тимескаледб](https://docs.timescale.com/latest)                    | 1.1.1             | Включает масштабируемые операции вставки и сложные запросы для данных временных рядов|
> |[unaccent](https://www.postgresql.org/docs/10/unaccent.html)                     | 1,1             | словарь поиска текста, который удаляет диакритические знаки|
> |[uuid-ossp](https://www.postgresql.org/docs/10/uuid-ossp.html)                    | 1,1             | создать универсальные уникальные идентификаторы (UUID)|

## <a name="postgres-96-extensions"></a>Расширения postgres 9,6 

В базе данных Azure для серверов PostgreSQL доступны следующие расширения с Postgres версии 9,6.

> [!div class="mx-tableFixed"]
> | **Расширение**| **версия расширения;** | **Описание** |
> |---|---|---|
> |[address_standardizer](http://postgis.net/docs/Address_Standardizer.html)         | 2.3.2           | Используются для анализа адреса в составных элементы. |
> |[address_standardizer_data_us](http://postgis.net/docs/Address_Standardizer.html) | 2.3.2           | Пример набора данных для стандартизации адресов США|
> |[btree_gin](https://www.postgresql.org/docs/9.6/btree-gin.html)                    | 1.0             | Поддержка индексирования общих типов в Ло|
> |[btree_gist](https://www.postgresql.org/docs/9.6/btree-gist.html)                   | 1.2             | Поддержка индексирования общих типов в реестрах|
> |[chkpass](https://www.postgresql.org/docs/9.6/chkpass.html)                       | 1.0             | тип данных для автоматического шифрования паролей|
> |[citext](https://www.postgresql.org/docs/9.6/citext.html)                       | 1,3             | тип данных для символьных строк без учета регистра|
> |[Куба](https://www.postgresql.org/docs/9.6/cube.html)                         | 1.2             | тип данных для многомерных кубов|
> |[dblink](https://www.postgresql.org/docs/9.6/dblink.html)                       | 1.2             | подключение к другим базам данных PostgreSQL из базы данных|
> |[dict_int](https://www.postgresql.org/docs/9.6/dict-int.html)                     | 1.0             | шаблон словаря поиска текста для целых чисел|
> |[earthdistance](https://www.postgresql.org/docs/9.6/earthdistance.html)                | 1,1             | Вычислите большие расстояния на поверхности земли|
> |[fuzzystrmatch](https://www.postgresql.org/docs/9.6/fuzzystrmatch.html)                | 1,1             | определение сходства и расстояния между строками|
> |[hstore](https://www.postgresql.org/docs/9.6/hstore.html)                       | 1.4             | тип данных для хранения наборов пар (ключ, значение)|
> |[hypopg](https://hypopg.readthedocs.io/en/latest/)                       | 1.1.1           | Гипотетические индексы для PostgreSQL|
> |[intarray](https://www.postgresql.org/docs/9.6/intarray.html)                     | 1.2             | функции, операторы и поддержка индексов для 1-D массивов целых чисел|
> |[isn](https://www.postgresql.org/docs/9.6/isn.html)                          | 1,1             | типы данных для международных стандартов нумерации продуктов|
> |[ltree](https://www.postgresql.org/docs/9.6/ltree.html)                        | 1,1             | тип данных для иерархических структур, схожих с деревом|
> |[орафце](https://github.com/orafce/orafce)                       | 3,7             | Функции и операторы, имитирующие подмножество функций и пакетов из коммерческой RDBMS|
> |[пгаудит](https://www.pgaudit.org/)                     | 1.1.2             | предоставляет функции аудита|
> |[pgcrypto](https://www.postgresql.org/docs/9.6/pgcrypto.html)                     | 1,3             | криптографические функции|
> |[pgrouting](https://pgrouting.org/)                    | 2.3.2           | Расширение Пграутинг|
> |[pgrowlocks](https://www.postgresql.org/docs/9.6/pgrowlocks.html)                   | 1.2             | показывать сведения о блокировке на уровне строк|
> |[pgstattuple](https://www.postgresql.org/docs/9.6/pgstattuple.html)                  | 1.4             | Отображение статистики на уровне кортежей|
> |[pg_buffercache](https://www.postgresql.org/docs/9.6/pgbuffercache.html)               | 1.2             | Проверка общего кэша буфера|
> |[pg_partman](https://github.com/pgpartman/pg_partman)                   | 2.6.3           | Расширение для управления секционированными таблицами по времени или ИДЕНТИФИКАТОРу|
> |[pg_prewarm](https://www.postgresql.org/docs/9.6/pgprewarm.html)                   | 1,1             | предтеплое отношение данных|
> |[pg_stat_statements](https://www.postgresql.org/docs/9.6/pgstatstatements.html)           | 1.4             | Трассировка статистики выполнения всех выполненных инструкций SQL|
> |[pg_trgm](https://www.postgresql.org/docs/9.6/pgtrgm.html)                      | 1,3             | измерение подобия текста и поиск по индексу на основе триграммов|
> |[plpgsql](https://www.postgresql.org/docs/9.6/plpgsql.html)                      | 1.0             | Процедурный язык PL/pgSQL|
> |[plv8](https://plv8.github.io/)                         | 2.1.0          | Доверенный процедурный язык PL/JavaScript (V8)|
> |[PostGIS](https://www.postgis.net/)                      | 2.3.2           | PostGISие геометрических, географических и растровых пространственных типов и функций|
> |[postgis_sfcgal](https://www.postgis.net/)               | 2.3.2           | Функции СФКГАЛ PostGIS|
> |[postgis_tiger_geocoder](https://www.postgis.net/)       | 2.3.2           | PostGIS Tiger для геокодирования и инвертированного геокодирования|
> |[postgis_topology](https://postgis.net/docs/Topology.html)             | 2.3.2           | Пространственные типы и функции топологии PostGIS|
> |[postgres_fdw](https://www.postgresql.org/docs/9.6/postgres-fdw.html)                 | 1.0             | оболочка внешних данных для удаленных серверов PostgreSQL|
> |[tablefunc](https://www.postgresql.org/docs/9.6/tablefunc.html)                    | 1.0             | функции, управляющие целыми таблицами, включая перекрестный|
> |[тимескаледб](https://docs.timescale.com/latest)                    | 1.1.1             | Включает масштабируемые операции вставки и сложные запросы для данных временных рядов|
> |[unaccent](https://www.postgresql.org/docs/9.6/unaccent.html)                     | 1,1             | словарь поиска текста, который удаляет диакритические знаки|
> |[uuid-ossp](https://www.postgresql.org/docs/9.6/uuid-ossp.html)                    | 1,1             | создать универсальные уникальные идентификаторы (UUID)|

## <a name="postgres-95-extensions"></a>Расширения postgres 9,5 

В базе данных Azure для серверов PostgreSQL доступны следующие расширения с Postgres версии 9,5.

> [!div class="mx-tableFixed"]
> | **Расширение**| **версия расширения;** | **Описание** |
> |---|---|---|
> |[address_standardizer](http://postgis.net/docs/Address_Standardizer.html)         | 2.3.0           | Используются для анализа адреса в составных элементы. |
> |[address_standardizer_data_us](http://postgis.net/docs/Address_Standardizer.html) | 2.3.0           | Пример набора данных для стандартизации адресов США|
> |[btree_gin](https://www.postgresql.org/docs/9.5/btree-gin.html)                    | 1.0             | Поддержка индексирования общих типов в Ло|
> |[btree_gist](https://www.postgresql.org/docs/9.5/btree-gist.html)                   | 1,1             | Поддержка индексирования общих типов в реестрах|
> |[chkpass](https://www.postgresql.org/docs/9.5/chkpass.html)                       | 1.0             | тип данных для автоматического шифрования паролей|
> |[citext](https://www.postgresql.org/docs/9.5/citext.html)                       | 1,1             | тип данных для символьных строк без учета регистра|
> |[Куба](https://www.postgresql.org/docs/9.5/cube.html)                         | 1.0             | тип данных для многомерных кубов|
> |[dblink](https://www.postgresql.org/docs/9.5/dblink.html)                       | 1,1             | подключение к другим базам данных PostgreSQL из базы данных|
> |[dict_int](https://www.postgresql.org/docs/9.5/dict-int.html)                     | 1.0             | шаблон словаря поиска текста для целых чисел|
> |[earthdistance](https://www.postgresql.org/docs/9.5/earthdistance.html)                | 1.0             | Вычислите большие расстояния на поверхности земли|
> |[fuzzystrmatch](https://www.postgresql.org/docs/9.5/fuzzystrmatch.html)                | 1.0             | определение сходства и расстояния между строками|
> |[hstore](https://www.postgresql.org/docs/9.5/hstore.html)                       | 1,3             | тип данных для хранения наборов пар (ключ, значение)|
> |[hypopg](https://hypopg.readthedocs.io/en/latest/)                       | 1.1.1           | Гипотетические индексы для PostgreSQL|
> |[intarray](https://www.postgresql.org/docs/9.5/intarray.html)                     | 1.0             | функции, операторы и поддержка индексов для 1-D массивов целых чисел|
> |[isn](https://www.postgresql.org/docs/9.5/isn.html)                          | 1.0             | типы данных для международных стандартов нумерации продуктов|
> |[ltree](https://www.postgresql.org/docs/9.5/ltree.html)                        | 1.0             | тип данных для иерархических структур, схожих с деревом|
> |[орафце](https://github.com/orafce/orafce)                       | 3,7             | Функции и операторы, имитирующие подмножество функций и пакетов из коммерческой RDBMS|
> |[пгаудит](https://www.pgaudit.org/)                     | 1.0.7             | предоставляет функции аудита|
> |[pgcrypto](https://www.postgresql.org/docs/9.5/pgcrypto.html)                     | 1.2             | криптографические функции|
> |[pgrouting](https://pgrouting.org/)                    | 2.3.0           | Расширение Пграутинг|
> |[pgrowlocks](https://www.postgresql.org/docs/9.5/pgrowlocks.html)                   | 1,1             | показывать сведения о блокировке на уровне строк|
> |[pgstattuple](https://www.postgresql.org/docs/9.5/pgstattuple.html)                  | 1,3             | Отображение статистики на уровне кортежей|
> |[pg_buffercache](https://www.postgresql.org/docs/9.5/pgbuffercache.html)               | 1,1             | Проверка общего кэша буфера|
> |[pg_partman](https://github.com/pgpartman/pg_partman)                   | 2.6.3           | Расширение для управления секционированными таблицами по времени или ИДЕНТИФИКАТОРу|
> |[pg_prewarm](https://www.postgresql.org/docs/9.5/pgprewarm.html)                   | 1.0             | предтеплое отношение данных|
> |[pg_stat_statements](https://www.postgresql.org/docs/9.5/pgstatstatements.html)           | 1,3             | Трассировка статистики выполнения всех выполненных инструкций SQL|
> |[pg_trgm](https://www.postgresql.org/docs/9.5/pgtrgm.html)                      | 1,1             | измерение подобия текста и поиск по индексу на основе триграммов|
> |[plpgsql](https://www.postgresql.org/docs/9.5/plpgsql.html)                      | 1.0             | Процедурный язык PL/pgSQL|
> |[PostGIS](https://www.postgis.net/)                      | 2.3.0           | PostGISие геометрических, географических и растровых пространственных типов и функций|
> |[postgis_sfcgal](https://www.postgis.net/)               | 2.3.0           | Функции СФКГАЛ PostGIS|
> |[postgis_tiger_geocoder](https://www.postgis.net/)       | 2.3.0           | PostGIS Tiger для геокодирования и инвертированного геокодирования|
> |[postgis_topology](https://postgis.net/docs/Topology.html)             | 2.3.0           | Пространственные типы и функции топологии PostGIS|
> |[postgres_fdw](https://www.postgresql.org/docs/9.5/postgres-fdw.html)                 | 1.0             | оболочка внешних данных для удаленных серверов PostgreSQL|
> |[tablefunc](https://www.postgresql.org/docs/9.5/tablefunc.html)                    | 1.0             | функции, управляющие целыми таблицами, включая перекрестный|
> |[unaccent](https://www.postgresql.org/docs/9.5/unaccent.html)                     | 1.0             | словарь поиска текста, который удаляет диакритические знаки|
> |[uuid-ossp](https://www.postgresql.org/docs/9.5/uuid-ossp.html)                    | 1.0             | создать универсальные уникальные идентификаторы (UUID)|


## <a name="pg_stat_statements"></a>pg_stat_statements
[Расширение pg_stat_statements](https://www.postgresql.org/docs/current/pgstatstatements.html) предварительно загружается на каждом сервере базы данных Azure для PostgreSQL для предоставления средств отслеживания статистики выполнения инструкций SQL.
Параметр `pg_stat_statements.track`, который управляет тем, какие инструкции учитываются расширением, по умолчанию имеет значение `top`. Это означает, что все инструкции, выпущенные непосредственно клиентами, отслеживаются. Два других уровня отслеживания: `none` и `all`. Этот параметр можно настроить в качестве параметра сервера с помощью [портала Azure](./howto-configure-server-parameters-using-portal.md) или [Azure CLI](./howto-configure-server-parameters-using-cli.md).

Существует компромисс между информацией о выполнении запроса, предоставляемой pg_stat_statements, и воздействием на производительность сервера при регистрации каждой инструкции SQL. Если расширение pg_stat_statements не используется активно, рекомендуется задать для параметра `pg_stat_statements.track` значение `none`. Обратите внимание, что некоторые сторонние службы мониторинга могут зависеть от pg_stat_statements при предоставлении информации о производительности запроса, поэтому подтвердите ваш ли это случай.

## <a name="dblink-and-postgres_fdw"></a>дблинк и postgres_fdw
[дблинк](https://www.postgresql.org/docs/current/contrib-dblink-function.html) и [postgres_fdw](https://www.postgresql.org/docs/current/postgres-fdw.html) позволяют подключаться с одного сервера PostgreSQL к другому или к другой базе данных на том же сервере. Принимающий сервер должен разрешать подключения с отправляющего сервера при помощи его брандмауэра. При использовании этих расширений для подключения между серверами Базы данных Azure для PostgreSQL, это можно сделать, задав значение "Вкл." для "Разрешить доступ к службам Azure". Это необходимо, если вы хотите использовать расширения для циклического возвращения на тот же сервер. Параметр "Разрешить доступ к службам Azure" можно найти на странице портала Azure для сервера Postgres в разделе "Безопасность подключения". Включив параметр "разрешить доступ к службам Azure", вы помещаете все IP-адреса Azure в список разрешений.

В настоящее время исходящие подключения из базы данных Azure для PostgreSQL не поддерживаются, за исключением подключений к другим серверам базы данных Azure для PostgreSQL в том же регионе.

## <a name="uuid"></a>uuid
Если вы планируете использовать `uuid_generate_v4()` из [расширения UUID-ОССП](https://www.postgresql.org/docs/current/uuid-ossp.html), рассмотрите возможность сравнения с `gen_random_uuid()` из [расширения пгкрипто](https://www.postgresql.org/docs/current/pgcrypto.html) для повышения производительности.

## <a name="pgaudit"></a>пгаудит
[Расширение пгаудит](https://github.com/pgaudit/pgaudit/blob/master/README.md) обеспечивает ведение журнала аудита сеансов и объектов. Сведения об использовании этого расширения в базе данных Azure для PostgreSQL см. в [статье основные понятия аудита](concepts-audit.md). 

## <a name="pg_prewarm"></a>pg_prewarm
Расширение pg_prewarm загружает реляционные данные в кэш. Предварительная подготовка кэшей означает, что запросы имеют лучшее время ответа при первом запуске после перезагрузки. В postgres 10 и ниже предгорячий предназначение выполняется вручную с помощью [предгорячей функции](https://www.postgresql.org/docs/10/pgprewarm.html).

В postgres 11 и более поздних версиях можно настроить предварительную предзагрузку в [автоматическом](https://www.postgresql.org/docs/current/pgprewarm.html)состоянии. Необходимо включить pg_prewarm в `shared_preload_libraries` список параметров и перезапустить сервер, чтобы применить изменения. Параметры можно задать в шаблоне [портал Azure](howto-configure-server-parameters-using-portal.md), [CLI](howto-configure-server-parameters-using-cli.md), REST API или ARM. 

## <a name="timescaledb"></a>тимескаледб
Тимескаледб — это база данных временных рядов, которая упакована как расширение для PostgreSQL. Тимескаледб обеспечивает ориентированные на время аналитические функции, оптимизации и масштабирование postgres для рабочих нагрузок временных рядов.

[Узнайте больше о тимескаледб](https://docs.timescale.com/latest), охраняемом товарном знаке [шкалы времени, Inc.](https://www.timescale.com/) База данных Azure для PostgreSQL предоставляет [выпуск Тимескаледб Apache-2](https://www.timescale.com/legal/licenses).

### <a name="installing-timescaledb"></a>Установка Тимескаледб
Чтобы установить Тимескаледб, необходимо включить его в общие библиотеки предварительной загрузки сервера. Для вступления в `shared_preload_libraries` силу изменения параметра postgres требуется **Перезагрузка сервера** . Параметры можно изменить с помощью [портал Azure](howto-configure-server-parameters-using-portal.md) или [Azure CLI](howto-configure-server-parameters-using-cli.md).

Использование [портал Azure](https://portal.azure.com/):

1. Выберите сервер базы данных Azure для PostgreSQL.

2. На боковой панели выберите **Параметры сервера**.

3. Найдите параметр `shared_preload_libraries`.

4. Выберите **тимескаледб**.

5. Нажмите кнопку **сохранить** , чтобы сохранить изменения. После сохранения изменений появляется уведомление. 

6. После уведомления **перезапустите** сервер, чтобы применить эти изменения. Дополнительные сведения о перезапуске сервера службы "База данных Azure для PostgreSQL" см. в [этой статье](howto-restart-server-portal.md).


Теперь можно включить Тимескаледб в базе данных postgres. Подключитесь к базе данных и выполните следующую команду:
```sql
CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;
```
> [!TIP]
> Если появится сообщение об ошибке, убедитесь, что [сервер был перезагружен](howto-restart-server-portal.md) после сохранения shared_preload_libraries. 

Теперь можно создать таблицу Тимескаледб [с нуля](https://docs.timescale.com/getting-started/creating-hypertables) или перенести [существующие данные временных рядов в PostgreSQL](https://docs.timescale.com/getting-started/migrating-data).

### <a name="restoring-a-timescale-database"></a>Восстановление базы данных шкалы времени
Чтобы восстановить базу данных шкалы времени с помощью pg_dump и pg_restore, необходимо выполнить две вспомогательные процедуры в целевой базе данных: `timescaledb_pre_restore()` и `timescaledb_post restore()` .

Сначала подготовьте целевую базу данных:

```SQL
--create the new database where you'll perform the restore
CREATE DATABASE tutorial;
\c tutorial --connect to the database 
CREATE EXTENSION timescaledb;

SELECT timescaledb_pre_restore();
```

Теперь можно запустить pg_dump в исходной базе данных, а затем выполнить pg_restore. После восстановления убедитесь, что в восстановленной базе данных выполнена следующая команда:

```SQL
SELECT timescaledb_post_restore();
```


## <a name="next-steps"></a>Дальнейшие действия
Если вы не видите расширение, которое вы хотите использовать, сообщите нам. Проголосуйте за существующие запросы или создавайте новые запросы на отзыв на [форуме отзывов](https://feedback.azure.com/forums/597976-azure-database-for-postgresql).