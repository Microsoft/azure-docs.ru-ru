---
title: 'Краткое руководство: создание серверной группы — Гипермасштабирование (Citus) — База данных Azure для PostgreSQL'
description: Краткое руководство по созданию распределенных таблиц и выполнению к ним запросов с помощью Базы данных Azure для PostgreSQL с решением "Гипермасштабирование (Citus)".
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.custom: mvc
ms.topic: quickstart
ms.date: 08/17/2020
ms.openlocfilehash: 03a6e927a074067e85f1a3adca38cae386d1af38
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95026222"
---
# <a name="quickstart-create-a-hyperscale-citus-server-group-in-the-azure-portal"></a>Краткое руководство. Создание серверной группы Гипермасштабирования (Citus) на портале Azure

База данных Azure для PostgreSQL — это управляемая служба, с помощью которой можно запускать и масштабировать базы данных PostgreSQL высокой доступности, а также управлять ими в облаке. Из этого краткого руководства вы узнаете, как создать серверную группу Базы данных Azure для PostgreSQL с решением "Гипермасштабирование (Citus)" на портале Azure. Вы узнаете о распределении данных — сегментировании таблиц по узлам, приеме данных и выполнении запросов на нескольких узлах.

[!INCLUDE [azure-postgresql-hyperscale-create-db](../../includes/azure-postgresql-hyperscale-create-db.md)]

## <a name="create-and-distribute-tables"></a>Создание и распределение таблиц

После подключения к узлу координатора Hyperscale с помощью psql можно выполнить некоторые основные задачи.

На серверах Hyperscale (Citus) используется три вида таблиц:

- распределенные или сегментированные таблицы (распределенные в целях масштабирования для повышения производительности и работы в параллельном режиме);
- ссылочные таблицы (несколько копий);
- локальные таблицы (часто используется как внутренние таблицы администрирования).

В этом кратком руководстве основное внимание уделяется распределенным таблицам и работе с ними.

Мы будем использовать простую модель данных из репозитория GitHub с данными о пользователях и событиях. К событиям относится создание вилки, фиксации GIT, относящиеся к организации, и прочее.

Выполнив подключение с помощью psql, мы создадим таблицы. В консоли psql выполните такие команды:

```sql
CREATE TABLE github_events
(
    event_id bigint,
    event_type text,
    event_public boolean,
    repo_id bigint,
    payload jsonb,
    repo jsonb,
    user_id bigint,
    org jsonb,
    created_at timestamp
);

CREATE TABLE github_users
(
    user_id bigint,
    url text,
    login text,
    avatar_url text,
    gravatar_id text,
    display_login text
);
```

Поле `payload` в `github_events` содержит данные в формате JSONB. JSONB — это используемая в Postgres разновидность формата JSON с данными в двоичном формате. Этот тип данных позволяет хранить гибкую схему в одном столбце.

Postgres может создать индекс `GIN` для этого типа, который будет индексировать каждый ключ и значение внутри него. С помощью индекса можно быстрее и проще запрашивать полезные данные с разными условиями. Теперь создадим несколько индексов, прежде чем загружать данные. В psql выполните такие команды:

```sql
CREATE INDEX event_type_index ON github_events (event_type);
CREATE INDEX payload_index ON github_events USING GIN (payload jsonb_path_ops);
```

Далее мы поместим эти таблицы Postgres на узле координатора и распределим их по рабочим узлам с помощью решения Hyperscale (Citus). Для этого выполним запрос к каждой таблице, указав ключ для ее сегментирования. В этом примере мы будем сегментировать обе таблицы — с данными о пользователях и данными о событиях с помощью `user_id`.

```sql
SELECT create_distributed_table('github_events', 'user_id');
SELECT create_distributed_table('github_users', 'user_id');
```

[!INCLUDE [azure-postgresql-hyperscale-dist-alert](../../includes/azure-postgresql-hyperscale-dist-alert.md)]

Теперь можно загрузить данные. Воспользуйтесь psql, чтобы скачать файлы:

```sql
\! curl -O https://examples.citusdata.com/users.csv
\! curl -O https://examples.citusdata.com/events.csv
```

Затем загрузите данные из файлов в распределенные таблицы:

```sql
SET CLIENT_ENCODING TO 'utf8';

\copy github_events from 'events.csv' WITH CSV
\copy github_users from 'users.csv' WITH CSV
```

## <a name="run-queries"></a>Выполнение запросов

Теперь можно приступить к выполнению запросов. Начнем с простого — `count (*)`, чтобы узнать, какой объем данных мы загрузили.

```sql
SELECT count(*) from github_events;
```

Замечательно. Мы вскоре вернемся к такого рода агрегированию, а пока выполним несколько других запросов. В столбце JSONB `payload` порядочный объем данных, но они различаются по типу события. События `PushEvent` содержат размер, который включает в себя число уникальных фиксаций для отправки. Его можно использовать для получения общего числа фиксации в час.

```sql
SELECT date_trunc('hour', created_at) AS hour,
       sum((payload->>'distinct_size')::int) AS num_commits
FROM github_events
WHERE event_type = 'PushEvent'
GROUP BY hour
ORDER BY hour;
```

До сих пор запросы касались исключительно данных github\_events, но можно объединить эти сведения с данными github\_users. Так как мы выполнили сегментирование пользователей и событий с помощью одного идентификатора (`user_id`), строки обеих таблиц с совпадающими идентификаторами пользователя будут [размещены вместе](concepts-hyperscale-colocation.md) на одних и тех же узлах базы данных и их можно легко объединить.

Если объединить данные по `user_id`, решение Hyperscale (Citus) сможет выполнить объединение в сегментах в параллельном режиме на рабочих узлах. Например, можно найти пользователей, создавших наибольшее количество репозиториев.

```sql
SELECT gu.login, count(*)
  FROM github_events ge
  JOIN github_users gu
    ON ge.user_id = gu.user_id
 WHERE ge.event_type = 'CreateEvent'
   AND ge.payload @> '{"ref_type": "repository"}'
 GROUP BY gu.login
 ORDER BY count(*) DESC;
```

## <a name="clean-up-resources"></a>Очистка ресурсов

На предыдущих шагах вы создали ресурсы Azure в группе серверов. Если эти ресурсы вам больше не нужны, удалите группу серверов. Нажмите кнопку **Удалить** на странице **Обзор**, чтобы удалить группу серверов. При появлении запроса на всплывающей странице проверьте имя группы серверов и нажмите кнопку **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как подготовить к работе группу серверов Hyperscale (Citus). Вы подключились к ней с помощью psql, создали схему и выполнили распределение данных.

- Ознакомьтесь с учебником по [созданию мультитенантных масштабируемых приложений](./tutorial-design-database-hyperscale-multi-tenant.md).
- Определите оптимальный [первоначальный размер](howto-hyperscale-scale-initial.md) для своей группы серверов.
