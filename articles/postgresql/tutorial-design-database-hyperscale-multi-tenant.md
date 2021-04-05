---
title: Руководство по Проектирование мультитенантной базы данных с помощью Базы данных Azure для PostgreSQL с решением "Гипермасштабирование (Citus)"
description: В этом учебнике демонстрируется, как расширить возможности масштабируемого мультитенантного приложения с помощью Базы данных Azure для PostgreSQL с Гипермасштабированием (Citus).
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.custom: mvc
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 05/14/2019
ms.openlocfilehash: d8dee086f55c5596af50245b5271d98536eb7e57
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "95026205"
---
# <a name="tutorial-design-a-multi-tenant-database-by-using-azure-database-for-postgresql--hyperscale-citus"></a>Учебник по проектированию мультитенантной базы данных с помощью Базы данных Azure для PostgreSQL с решением "Гипермасштабирование (Citus)"

В этом учебнике используется решение "Гипермасштабирование (Citus)" в Базе данных Azure для PostgreSQL, чтобы продемонстрировать следующее:

> [!div class="checklist"]
> * создание группы серверов Hyperscale (Citus);
> * Использование служебной программы psql для создания схемы
> * Сегментирование таблиц по узлам
> * Принятие демонстрационных данных
> * Запрос данных клиента
> * Обмен данными между клиентами
> * Настройка схемы для каждого клиента

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [azure-postgresql-hyperscale-create-db](../../includes/azure-postgresql-hyperscale-create-db.md)]

## <a name="use-psql-utility-to-create-a-schema"></a>Использование служебной программы psql для создания схемы

Подключившись с помощью psql к решению "Гипермасштабирование (Citus)" в Базе данных Azure для PostgreSQL, вы сможете выполнить некоторые несложные задачи. В этом руководстве описывается создание веб-приложения, которое позволяет рекламодателям отслеживать свои рекламные кампании.

Это приложение могут использовать несколько организаций сразу, поэтому мы создадим таблицу для списка организаций и еще одну — для рекламных кампаний. В консоли psql выполните следующие команды:

```sql
CREATE TABLE companies (
  id bigserial PRIMARY KEY,
  name text NOT NULL,
  image_url text,
  created_at timestamp without time zone NOT NULL,
  updated_at timestamp without time zone NOT NULL
);

CREATE TABLE campaigns (
  id bigserial,
  company_id bigint REFERENCES companies (id),
  name text NOT NULL,
  cost_model text NOT NULL,
  state text NOT NULL,
  monthly_budget bigint,
  blacklisted_site_urls text[],
  created_at timestamp without time zone NOT NULL,
  updated_at timestamp without time zone NOT NULL,

  PRIMARY KEY (company_id, id)
);
```

Каждая организация оплачивает показ рекламных объявлений. Добавьте таблицу и для рекламных объявлений, выполнив в psql следующий код после приведенного выше:

```sql
CREATE TABLE ads (
  id bigserial,
  company_id bigint,
  campaign_id bigint,
  name text NOT NULL,
  image_url text,
  target_url text,
  impressions_count bigint DEFAULT 0,
  clicks_count bigint DEFAULT 0,
  created_at timestamp without time zone NOT NULL,
  updated_at timestamp without time zone NOT NULL,

  PRIMARY KEY (company_id, id),
  FOREIGN KEY (company_id, campaign_id)
    REFERENCES campaigns (company_id, id)
);
```

Теперь мы добавим отслеживание статистики кликов и показов для каждого рекламного объявления:

```sql
CREATE TABLE clicks (
  id bigserial,
  company_id bigint,
  ad_id bigint,
  clicked_at timestamp without time zone NOT NULL,
  site_url text NOT NULL,
  cost_per_click_usd numeric(20,10),
  user_ip inet NOT NULL,
  user_data jsonb NOT NULL,

  PRIMARY KEY (company_id, id),
  FOREIGN KEY (company_id, ad_id)
    REFERENCES ads (company_id, id)
);

CREATE TABLE impressions (
  id bigserial,
  company_id bigint,
  ad_id bigint,
  seen_at timestamp without time zone NOT NULL,
  site_url text NOT NULL,
  cost_per_impression_usd numeric(20,10),
  user_ip inet NOT NULL,
  user_data jsonb NOT NULL,

  PRIMARY KEY (company_id, id),
  FOREIGN KEY (company_id, ad_id)
    REFERENCES ads (company_id, id)
);
```

Теперь новые таблицы отображаются в полном списке таблиц, который можно получить в psql, выполнив следующую команду:

```postgres
\dt
```

Мультитенантные приложения могут применять уникальность только в пределах каждого клиента, поэтому все первичные и внешние ключи должны включать идентификатор компании.

## <a name="shard-tables-across-nodes"></a>Сегментирование таблиц по узлам

При гипермасштабируемом развертывании строки таблиц распределяются по разным узлам на основе значения столбца, назначенного пользователем. Этот "столбец распределения" определяет, какому клиенту принадлежит какая строка.

Давайте назначим идентификатор клиента company\_id в качестве столбца распределения. Выполните в psql такие функции:

```sql
SELECT create_distributed_table('companies',   'id');
SELECT create_distributed_table('campaigns',   'company_id');
SELECT create_distributed_table('ads',         'company_id');
SELECT create_distributed_table('clicks',      'company_id');
SELECT create_distributed_table('impressions', 'company_id');
```

[!INCLUDE [azure-postgresql-hyperscale-dist-alert](../../includes/azure-postgresql-hyperscale-dist-alert.md)]

## <a name="ingest-sample-data"></a>Принятие демонстрационных данных

Теперь в обычной командной строке (вне psql) скачайте примеры наборов данных:

```bash
for dataset in companies campaigns ads clicks impressions geo_ips; do
  curl -O https://examples.citusdata.com/mt_ref_arch/${dataset}.csv
done
```

Перейдите в psql и выполните массовую загрузку данных. Команду psql нужно выполнять в том же каталоге, куда вы скачали файлы данных.

```sql
SET CLIENT_ENCODING TO 'utf8';

\copy companies from 'companies.csv' with csv
\copy campaigns from 'campaigns.csv' with csv
\copy ads from 'ads.csv' with csv
\copy clicks from 'clicks.csv' with csv
\copy impressions from 'impressions.csv' with csv
```

Теперь эти данные будут распределены между рабочими узлами.

## <a name="query-tenant-data"></a>Запрос данных клиента

Когда приложение запросит данные одного клиента, база данных сможет выполнить такой запрос на одном рабочем узле. Запросы по одному клиенту фильтруются по идентификатору клиента. Например, следующий запрос фильтрует `company_id = 5` по рекламным объявлениям и показам. Попробуйте выполнить его в psql и изучите результаты.

```sql
SELECT a.campaign_id,
       RANK() OVER (
         PARTITION BY a.campaign_id
         ORDER BY a.campaign_id, count(*) desc
       ), count(*) as n_impressions, a.id
  FROM ads as a
  JOIN impressions as i
    ON i.company_id = a.company_id
   AND i.ad_id      = a.id
 WHERE a.company_id = 5
GROUP BY a.campaign_id, a.id
ORDER BY a.campaign_id, n_impressions desc;
```

## <a name="share-data-between-tenants"></a>Обмен данными между клиентами

До сих пор все таблицы распределялись по `company_id`, но некоторые данные нельзя "естественным образом" отнести к конкретному клиенту, то есть они используются клиентами совместно. Например, в нашем примере платформы для показа рекламы всем организациям будет полезно получать сведения о местоположении аудитории на основе IP-адресов.

Создайте таблицу для совместно используемых сведений о географическом расположении. Выполните следующие команды в psql:

```sql
CREATE TABLE geo_ips (
  addrs cidr NOT NULL PRIMARY KEY,
  latlon point NOT NULL
    CHECK (-90  <= latlon[0] AND latlon[0] <= 90 AND
           -180 <= latlon[1] AND latlon[1] <= 180)
);
CREATE INDEX ON geo_ips USING gist (addrs inet_ops);
```

Теперь сделайте `geo_ips` "ссылочной таблицей", чтобы копия этой таблицы хранилась на каждом рабочем узле.

```sql
SELECT create_reference_table('geo_ips');
```

Заполните ее данными из примера. Не забудьте, что эту команду нужно выполнять в psql в том же каталоге, куда вы скачали набор данных.

```sql
\copy geo_ips from 'geo_ips.csv' with csv
```

Объединение таблицы кликов с geo\_ips применяется одновременно на всех узлах.
Такое объединение позволяет найти расположение каждого, кто кликнул рекламное объявление.
290. Попробуйте запустить этот запрос в psql.

```sql
SELECT c.id, clicked_at, latlon
  FROM geo_ips, clicks c
 WHERE addrs >> c.user_ip
   AND c.company_id = 5
   AND c.ad_id = 290;
```

## <a name="customize-the-schema-per-tenant"></a>Настройка схемы для каждого клиента

Каждому клиенту может потребоваться хранить собственные сведения, которые не нужны другим пользователям. Но при этом все клиенты используют общую инфраструктуру с идентичной схемой базы данных. Куда же разместить эти дополнительные данные?

Есть хороший выход — использовать открытый тип столбца, например JSONB в PostgreSQL.  В нашей схеме в таблице `clicks` есть поле типа JSONB с именем `user_data`.
Любая организация (например, пятая из нашего примера) может хранить в этом столбце сведения об использовании мобильных устройств.

Следующий запрос позволяет найти, кто совершает больше кликов: посетители с мобильных или с классических устройств.

```sql
SELECT
  user_data->>'is_mobile' AS is_mobile,
  count(*) AS count
FROM clicks
WHERE company_id = 5
GROUP BY user_data->>'is_mobile'
ORDER BY count DESC;
```

Мы оптимизируем этот запрос для одной организации, создав [частичный индекс](https://www.postgresql.org/docs/current/static/indexes-partial.html).

```sql
CREATE INDEX click_user_data_is_mobile
ON clicks ((user_data->>'is_mobile'))
WHERE company_id = 5;
```

Теоретически мы можем создать [индексы GIN](https://www.postgresql.org/docs/current/static/gin-intro.html) по каждому ключу и значению в столбце.

```sql
CREATE INDEX click_user_data
ON clicks USING gin (user_data);

-- this speeds up queries like, "which clicks have
-- the is_mobile key present in user_data?"

SELECT id
  FROM clicks
 WHERE user_data ? 'is_mobile'
   AND company_id = 5;
```

## <a name="clean-up-resources"></a>Очистка ресурсов

На предыдущих шагах вы создали ресурсы Azure в группе серверов. Если эти ресурсы вам больше не нужны, удалите группу серверов. Нажмите кнопку *Удалить* на странице *Обзор*, чтобы удалить группу серверов. При появлении запроса на всплывающей странице проверьте имя группы серверов и нажмите кнопку *Удалить*.

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как подготовить к работе группу серверов с подключенным решением Hyperscale (Citus). Вы подключились к ней с помощью psql, создали схему и выполнили распределение данных. Вы научились запрашивать данные по одному или нескольким клиентам, а также настраивать схему для каждого клиента.

- Подробнее о [типах узлов](./concepts-hyperscale-nodes.md) группы сервера.
- Определите оптимальный [первоначальный размер](howto-hyperscale-scale-initial.md) для своей группы серверов.
