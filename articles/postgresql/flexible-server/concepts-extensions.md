---
title: Расширения — база данных Azure для PostgreSQL — гибкий сервер
description: Сведения о доступных расширениях postgres в базе данных Azure для PostgreSQL-гибкого сервера
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: conceptual
ms.date: 03/17/2021
ms.openlocfilehash: 998154376895d8bcfc7cf36665a6a36f5c43e3b4
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104594994"
---
# <a name="postgresql-extensions-in-azure-database-for-postgresql---flexible-server"></a>Расширения PostgreSQL в базе данных Azure для PostgreSQL-гибкого сервера

> [!IMPORTANT]
> Гибкий сервер Базы данных Azure для PostgreSQL предоставляется в режиме предварительной версии

PostgreSQL предоставляет возможность расширить функциональность базы данных с помощью расширений. Расширения объединяют несколько связанных объектов SQL в один пакет, который можно загрузить или удалить из базы данных с помощью команды. После загрузки в базу данных расширения действуют как встроенные функции.

## <a name="how-to-use-postgresql-extensions"></a>Как использовать расширения PostgreSQL
Чтобы использовать расширения PostgreSQL, их необходимо сначала установить в базу данных. Чтобы установить определенное расширение, выполните команду " [создать расширение](https://www.postgresql.org/docs/current/sql-createextension.html) ". Эта команда загружает упакованные объекты в базу данных.

База данных Azure для PostgreSQL поддерживает подмножество расширений ключей, как указано ниже. Эти сведения также можно получить, выполнив `SHOW azure.extensions;`. Расширения, не перечисленные в этом документе, не поддерживаются в базе данных Azure для PostgreSQL-гибкого сервера. Вы не можете создать или загрузить собственное расширение в базе данных Azure для PostgreSQL.


## <a name="postgres-12-extensions"></a>Расширения postgres 12

В базе данных Azure для PostgreSQL-гибких серверов, имеющих postgres версии 12, доступны следующие расширения. 

> [!div class="mx-tableFixed"]
> | **Расширение**| **версия расширения;** | **Описание** |
> |---|---|---|
> |[address_standardizer](http://postgis.net/docs/Address_Standardizer.html)         | 3.0.0           | Используются для анализа адреса в составных элементы. |
> |[address_standardizer_data_us](http://postgis.net/docs/Address_Standardizer.html) | 3.0.0           | Пример набора данных для стандартизации адресов США|
> |[амчекк](https://www.postgresql.org/docs/12/amcheck.html)                    | 1.2             | функции для проверки целостности отношений|
> |[раскрытия](https://www.postgresql.org/docs/12/bloom.html)                    | 1.0             | метод доступа раскрытия — индекс на основе файла сигнатуры|
> |[btree_gin](https://www.postgresql.org/docs/12/btree-gin.html)                    | 1,3             | Поддержка индексирования общих типов в Ло|
> |[btree_gist](https://www.postgresql.org/docs/12/btree-gist.html)                   | 1.5             | Поддержка индексирования общих типов в реестрах|
> |[citext](https://www.postgresql.org/docs/12/citext.html)                       | 1.6             | тип данных для символьных строк без учета регистра|
> |[Куба](https://www.postgresql.org/docs/12/cube.html)                         | 1.4             | тип данных для многомерных кубов|
> |[dblink](https://www.postgresql.org/docs/12/dblink.html)                       | 1.2             | подключение к другим базам данных PostgreSQL из базы данных|
> |[dict_int](https://www.postgresql.org/docs/12/dict-int.html)                     | 1.0             | шаблон словаря поиска текста для целых чисел|
> |[dict_xsyn](https://www.postgresql.org/docs/12/dict-xsyn.html)                     | 1.0             | шаблон словаря поиска текста для обработки расширенных синонимов|
> |[earthdistance](https://www.postgresql.org/docs/12/earthdistance.html)                | 1,1             | Вычислите большие расстояния на поверхности земли|
> |[fuzzystrmatch](https://www.postgresql.org/docs/12/fuzzystrmatch.html)                | 1,1             | определение сходства и расстояния между строками|
> |[hstore](https://www.postgresql.org/docs/12/hstore.html)                       | 1.6             | тип данных для хранения наборов пар (ключ, значение)|
> |[интагг](https://www.postgresql.org/docs/12/intagg.html)                     | 1,1             | целочисленный агрегат и перечислитель. (Устарело)|
> |[intarray](https://www.postgresql.org/docs/12/intarray.html)                     | 1.2             | функции, операторы и поддержка индексов для 1-D массивов целых чисел|
> |[isn](https://www.postgresql.org/docs/12/isn.html)                          | 1.2             | типы данных для международных стандартов нумерации продуктов|
> |[ltree](https://www.postgresql.org/docs/12/ltree.html)                        | 1,1             | тип данных для иерархических структур, схожих с деревом|
> |[пажеинспект](https://www.postgresql.org/docs/12/pageinspect.html)                        | 1.7             | Проверка содержимого страниц базы данных на низком уровне|
> |[pg_buffercache](https://www.postgresql.org/docs/12/pgbuffercache.html)               | 1,3             | Проверка общего кэша буфера|
> |[pg_cron](https://github.com/citusdata/pg_cron/tree/b6e7dc9627515bf00e2086f168b3faa660e5fd36)                        | 1.2             | Планировщик заданий для PostgreSQL|
> |[pg_freespacemap](https://www.postgresql.org/docs/12/pgfreespacemap.html)               | 1.2             | изучение схемы свободного места (FSM)|
> |[pg_prewarm](https://www.postgresql.org/docs/12/pgprewarm.html)                   | 1.2             | предтеплое отношение данных|
> |[pg_stat_statements](https://www.postgresql.org/docs/12/pgstatstatements.html)           | 1.7             | Трассировка статистики выполнения всех выполненных инструкций SQL|
> |[pg_trgm](https://www.postgresql.org/docs/12/pgtrgm.html)                      | 1.4             | измерение подобия текста и поиск по индексу на основе триграммов|
> |[pg_visibility](https://www.postgresql.org/docs/12/pgvisibility.html)                      | 1.2             | Проверка сведений о видимости (на виртуальной машине) и видимости на уровне страницы|
> |[пгаудит](https://www.pgaudit.org/)                     | 1.4             | предоставляет функции аудита|
> |[pgcrypto](https://www.postgresql.org/docs/12/pgcrypto.html)                     | 1,3             | криптографические функции|
> |[pgrowlocks](https://www.postgresql.org/docs/12/pgrowlocks.html)                   | 1.2             | показывать сведения о блокировке на уровне строк|
> |[pgstattuple](https://www.postgresql.org/docs/12/pgstattuple.html)                  | 1.5             | Отображение статистики на уровне кортежей|
> |[plpgsql](https://www.postgresql.org/docs/12/plpgsql.html)                      | 1.0             | Процедурный язык PL/pgSQL|
> |[PostGIS](https://www.postgis.net/)                      | 3.0.0           | PostGIS геометрия, география |
> |[postgis_raster](https://www.postgis.net/)               | 3.0.0           | Растровые типы и функции PostGIS| 
> |[postgis_sfcgal](https://www.postgis.net/)               | 3.0.0           | Функции СФКГАЛ PostGIS|
> |[postgis_tiger_geocoder](https://www.postgis.net/)       | 3.0.0           | PostGIS Tiger для геокодирования и инвертированного геокодирования|
> |[postgis_topology](https://postgis.net/docs/Topology.html)             | 3.0.0           | Пространственные типы и функции топологии PostGIS|
> |[postgres_fdw](https://www.postgresql.org/docs/12/postgres-fdw.html)                 | 1.0             | оболочка внешних данных для удаленных серверов PostgreSQL|
> |[сслинфо](https://www.postgresql.org/docs/12/sslinfo.html)                    | 1.2             | сведения о SSL-сертификатах|
> |[tsm_system_rows](https://www.postgresql.org/docs/12/tsm-system-rows.html)                    | 1.0             |  Метод TABLESAMPLE, который принимает число строк в качестве ограничения|
> |[tsm_system_time](https://www.postgresql.org/docs/12/tsm-system-time.html)                    | 1.0             |  Метод TABLESAMPLE, который принимает в качестве ограничения время в миллисекундах|
> |[unaccent](https://www.postgresql.org/docs/12/unaccent.html)                     | 1,1             | словарь поиска текста, который удаляет диакритические знаки|
> |[uuid-ossp](https://www.postgresql.org/docs/12/uuid-ossp.html)                    | 1,1             | создать универсальные уникальные идентификаторы (UUID)|

## <a name="postgres-11-extensions"></a>Расширения postgres 11

Следующие расширения доступны в базе данных Azure для PostgreSQL-гибких серверов с Postgres версии 11. 

> [!div class="mx-tableFixed"]
> | **Расширение**| **версия расширения;** | **Описание** |
> |---|---|---|
> |[address_standardizer](http://postgis.net/docs/Address_Standardizer.html)         | 2.5.1           | Используются для анализа адреса в составных элементы. |
> |[address_standardizer_data_us](http://postgis.net/docs/Address_Standardizer.html) | 2.5.1           | Пример набора данных для стандартизации адресов США|
> |[амчекк](https://www.postgresql.org/docs/11/amcheck.html)                    | 1,1             | функции для проверки целостности отношений|
> |[раскрытия](https://www.postgresql.org/docs/11/bloom.html)                    | 1.0             | метод доступа раскрытия — индекс на основе файла сигнатуры|
> |[btree_gin](https://www.postgresql.org/docs/11/btree-gin.html)                    | 1,3             | Поддержка индексирования общих типов в Ло|
> |[btree_gist](https://www.postgresql.org/docs/11/btree-gist.html)                   | 1.5             | Поддержка индексирования общих типов в реестрах|
> |[citext](https://www.postgresql.org/docs/11/citext.html)                       | 1.5             | тип данных для символьных строк без учета регистра|
> |[Куба](https://www.postgresql.org/docs/11/cube.html)                         | 1.4             | тип данных для многомерных кубов|
> |[dblink](https://www.postgresql.org/docs/11/dblink.html)                       | 1.2             | подключение к другим базам данных PostgreSQL из базы данных|
> |[dict_int](https://www.postgresql.org/docs/11/dict-int.html)                     | 1.0             | шаблон словаря поиска текста для целых чисел|
> |[dict_xsyn](https://www.postgresql.org/docs/11/dict-xsyn.html)                     | 1.0             | шаблон словаря поиска текста для обработки расширенных синонимов|
> |[earthdistance](https://www.postgresql.org/docs/11/earthdistance.html)                | 1,1             | Вычислите большие расстояния на поверхности земли|
> |[fuzzystrmatch](https://www.postgresql.org/docs/11/fuzzystrmatch.html)                | 1,1             | определение сходства и расстояния между строками|
> |[hstore](https://www.postgresql.org/docs/11/hstore.html)                       | 1.5             | тип данных для хранения наборов пар (ключ, значение)|
> |[интагг](https://www.postgresql.org/docs/11/intagg.html)                     | 1,1             | целочисленный агрегат и перечислитель. (Устарело)|
> |[intarray](https://www.postgresql.org/docs/11/intarray.html)                     | 1.2             | функции, операторы и поддержка индексов для 1-D массивов целых чисел|
> |[isn](https://www.postgresql.org/docs/11/isn.html)                          | 1.2             | типы данных для международных стандартов нумерации продуктов|
> |[ltree](https://www.postgresql.org/docs/11/ltree.html)                        | 1,1             | тип данных для иерархических структур, схожих с деревом|
> |[пажеинспект](https://www.postgresql.org/docs/11/pageinspect.html)                        | 1.7             | Проверка содержимого страниц базы данных на низком уровне|
> |[pg_buffercache](https://www.postgresql.org/docs/11/pgbuffercache.html)               | 1,3             | Проверка общего кэша буфера|
> |[pg_cron](https://github.com/citusdata/pg_cron/tree/b6e7dc9627515bf00e2086f168b3faa660e5fd36)                        | 1.2             | Планировщик заданий для PostgreSQL|
> |[pg_freespacemap](https://www.postgresql.org/docs/11/pgfreespacemap.html)               | 1.2             | изучение схемы свободного места (FSM)|
> |[pg_prewarm](https://www.postgresql.org/docs/11/pgprewarm.html)                   | 1.2             | предтеплое отношение данных|
> |[pg_stat_statements](https://www.postgresql.org/docs/11/pgstatstatements.html)           | 1.6             | Трассировка статистики выполнения всех выполненных инструкций SQL|
> |[pg_trgm](https://www.postgresql.org/docs/11/pgtrgm.html)                      | 1.4             | измерение подобия текста и поиск по индексу на основе триграммов|
> |[pg_visibility](https://www.postgresql.org/docs/11/pgvisibility.html)                      | 1.2             | Проверка сведений о видимости (на виртуальной машине) и видимости на уровне страницы|
> |[пгаудит](https://www.pgaudit.org/)                     | 1.3.1             | предоставляет функции аудита|
> |[pgcrypto](https://www.postgresql.org/docs/11/pgcrypto.html)                     | 1,3             | криптографические функции|
> |[pgrowlocks](https://www.postgresql.org/docs/11/pgrowlocks.html)                   | 1.2             | показывать сведения о блокировке на уровне строк|
> |[pgstattuple](https://www.postgresql.org/docs/11/pgstattuple.html)                  | 1.5             | Отображение статистики на уровне кортежей|
> |[plpgsql](https://www.postgresql.org/docs/11/plpgsql.html)                      | 1.0             | Процедурный язык PL/pgSQL|
> |[PostGIS](https://www.postgis.net/)                      | 2.5.1           | PostGISие геометрических, географических и растровых пространственных типов и функций|
> |[postgis_sfcgal](https://www.postgis.net/)               | 2.5.1           | Функции СФКГАЛ PostGIS|
> |[postgis_tiger_geocoder](https://www.postgis.net/)       | 2.5.1           | PostGIS Tiger для геокодирования и инвертированного геокодирования|
> |[postgis_topology](https://postgis.net/docs/Topology.html)             | 2.5.1           | Пространственные типы и функции топологии PostGIS|
> |[postgres_fdw](https://www.postgresql.org/docs/11/postgres-fdw.html)                 | 1.0             | оболочка внешних данных для удаленных серверов PostgreSQL|
> |[сслинфо](https://www.postgresql.org/docs/11/sslinfo.html)                    | 1.2             | сведения о SSL-сертификатах|
> |[tablefunc](https://www.postgresql.org/docs/11/tablefunc.html)                    | 1.0             | функции, управляющие целыми таблицами, включая перекрестный|
> |[tsm_system_rows](https://www.postgresql.org/docs/11/tsm-system-rows.html)                    | 1.0             |  Метод TABLESAMPLE, который принимает число строк в качестве ограничения|
> |[tsm_system_time](https://www.postgresql.org/docs/11/tsm-system-time.html)                    | 1.0             |  Метод TABLESAMPLE, который принимает в качестве ограничения время в миллисекундах|
> |[unaccent](https://www.postgresql.org/docs/11/unaccent.html)                     | 1,1             | словарь поиска текста, который удаляет диакритические знаки|
> |[uuid-ossp](https://www.postgresql.org/docs/11/uuid-ossp.html)                    | 1,1             | создать универсальные уникальные идентификаторы (UUID)|


## <a name="dblink-and-postgres_fdw"></a>дблинк и postgres_fdw
[дблинк](https://www.postgresql.org/docs/current/contrib-dblink-function.html) и [postgres_fdw](https://www.postgresql.org/docs/current/postgres-fdw.html) позволяют подключаться с одного сервера PostgreSQL к другому или к другой базе данных на том же сервере. Гибкий сервер поддерживает как входящие, так и исходящие подключения к любому серверу PostgreSQL. Отправляющий сервер должен разрешать исходящие подключения к принимающему серверу. Аналогичным образом принимающему серверу необходимо разрешить соединения от отправляющего сервера. 

Если вы планируете использовать эти два расширения, рекомендуется развернуть серверы с помощью [интеграции с виртуальной](concepts-networking.md) сетью. По умолчанию интеграция с виртуальной сетью разрешает подключения между серверами в виртуальной сети. Для настройки доступа можно также использовать [группы безопасности сети VNet](../../virtual-network/manage-network-security-group.md) .

## <a name="pg_cron"></a>pg_cron

[pg_cron](https://github.com/citusdata/pg_cron/tree/b6e7dc9627515bf00e2086f168b3faa660e5fd36) — это простой планировщик заданий на основе cron для PostgreSQL, который работает в базе данных как расширение. Расширение pg_cron можно использовать для выполнения запланированных задач обслуживания в базе данных PostgreSQL. Например, можно запустить периодическую очистки таблицы или удалить старые задания данных.

`pg_cron` может выполнять несколько заданий параллельно, но в каждый момент времени выполняется не более одного экземпляра задания. Если второй запуск должен начаться до завершения первого, второй запуск помещается в очередь и запускается сразу после завершения первого выполнения. Это гарантирует, что задания выполняются в точном количестве раз по расписанию и не выполняются одновременно с ними.

Некоторые примеры.

Удаление старых данных в субботу в 3:10:30 (GMT)
```
SELECT cron.schedule('30 3 * * 6', $$DELETE FROM events WHERE event_time < now() - interval '1 week'$$);
```
Запуск чистильщика каждый день в 10:8:00 (GMT)
```
SELECT cron.schedule('0 10 * * *', 'VACUUM');
```

Чтобы отменять расписание всех задач от pg_cron
```
SELECT cron.unschedule(jobid) FROM cron.job;
```

## <a name="pg_prewarm"></a>pg_prewarm

Расширение pg_prewarm загружает реляционные данные в кэш. Предварительная подготовка кэшей означает, что запросы имеют лучшее время ответа при первом запуске после перезагрузки. Функция автоматического предгорячего использования в настоящее время недоступна в базе данных Azure для PostgreSQL-гибкого сервера.

## <a name="pg_stat_statements"></a>pg_stat_statements
[Расширение pg_stat_statements](https://www.postgresql.org/docs/current/pgstatstatements.html) предварительно загружается на каждом гибком сервере базы данных Azure для PostgreSQL, чтобы обеспечить возможность отслеживания статистики выполнения инструкций SQL.
Параметр `pg_stat_statements.track`, который управляет тем, какие инструкции учитываются расширением, по умолчанию имеет значение `top`. Это означает, что все инструкции, выпущенные непосредственно клиентами, отслеживаются. Два других уровня отслеживания: `none` и `all`. Этот параметр можно настроить в качестве параметра сервера.

Существует компромисс между информацией о выполнении запроса, предоставляемой pg_stat_statements, и воздействием на производительность сервера при регистрации каждой инструкции SQL. Если расширение pg_stat_statements не используется активно, рекомендуется задать для параметра `pg_stat_statements.track` значение `none`. Обратите внимание, что некоторые сторонние службы мониторинга могут зависеть от pg_stat_statements при предоставлении информации о производительности запроса, поэтому подтвердите ваш ли это случай.


## <a name="next-steps"></a>Дальнейшие действия

Если вы не видите расширение, которое вы хотите использовать, сообщите нам. Проголосуйте за существующие запросы или создавайте новые запросы на отзыв на [форуме отзывов](https://feedback.azure.com/forums/597976-azure-database-for-postgresql).