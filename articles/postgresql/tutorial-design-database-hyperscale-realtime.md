---
title: Руководство по Проектирование панели мониторинга для отображения данных в реальном времени — Гипермасштабирование (Citus) — База данных Azure для PostgreSQL
description: В этом учебнике демонстрируется, как обеспечить параллельное выполнение запросов панели мониторинга в реальном времени с помощью Базы данных Azure для PostgreSQL с Гипермасштабированием (Citus).
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.custom: mvc
ms.topic: tutorial
ms.date: 05/14/2019
ms.openlocfilehash: 480af87519f8e11c14f009058b0f518bdfc34a1b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95026273"
---
# <a name="tutorial-design-a-real-time-analytics-dashboard-by-using-azure-database-for-postgresql--hyperscale-citus"></a>Руководство по проектированию панели мониторинга для отображения данных в реальном времени с помощью Базы данных Azure для PostgreSQL с решением "Гипермасштабирование (Citus)"

В этом учебнике используется решение "Гипермасштабирование (Citus)" в Базе данных Azure для PostgreSQL, чтобы продемонстрировать следующее:

> [!div class="checklist"]
> * создание группы серверов Hyperscale (Citus);
> * Использование служебной программы psql для создания схемы
> * Сегментирование таблиц по узлам
> * Создание примера данных
> * выполнение сведения данных;
> * выполнение запроса к необработанным и агрегированным данным;
> * удаление устаревших данных.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [azure-postgresql-hyperscale-create-db](../../includes/azure-postgresql-hyperscale-create-db.md)]

## <a name="use-psql-utility-to-create-a-schema"></a>Использование служебной программы psql для создания схемы

Подключившись с помощью psql к решению "Гипермасштабирование (Citus)" в Базе данных Azure для PostgreSQL, вы сможете выполнить некоторые несложные задачи. В этом руководстве показано, как выполнить прием данных из средства веб-аналитики и сведение данных для их представления в реальном времени на панели мониторинга.

Давайте создадим таблицу, в которую будут поступать необработанные данные. В окне терминала psql выполните следующие команды:

```sql
CREATE TABLE http_request (
  site_id INT,
  ingest_time TIMESTAMPTZ DEFAULT now(),

  url TEXT,
  request_country TEXT,
  ip_address TEXT,

  status_code INT,
  response_time_msec INT
);
```

Кроме того, мы собираемся создать таблицу, в которой будут находиться поминутно агрегированные данные, и таблицу со сведениями о последнем сведении данных. В psql выполните также следующие команды:

```sql
CREATE TABLE http_request_1min (
  site_id INT,
  ingest_time TIMESTAMPTZ, -- which minute this row represents

  error_count INT,
  success_count INT,
  request_count INT,
  average_response_time_msec INT,
  CHECK (request_count = error_count + success_count),
  CHECK (ingest_time = date_trunc('minute', ingest_time))
);

CREATE INDEX http_request_1min_idx ON http_request_1min (site_id, ingest_time);

CREATE TABLE latest_rollup (
  minute timestamptz PRIMARY KEY,

  CHECK (minute = date_trunc('minute', minute))
);
```

Созданные таблицы можно отобразить в виде списка с помощью этой команды psql:

```postgres
\dt
```

## <a name="shard-tables-across-nodes"></a>Сегментирование таблиц по узлам

При гипермасштабируемом развертывании строки таблиц распределяются по разным узлам на основе значения столбца, назначенного пользователем. Этот столбец определяет, как данные распределяются между узлами.

Назначим столбец распределения ключом сегментирования site\_id. Выполните в psql такие функции:

  ```sql
SELECT create_distributed_table('http_request',      'site_id');
SELECT create_distributed_table('http_request_1min', 'site_id');
```

[!INCLUDE [azure-postgresql-hyperscale-dist-alert](../../includes/azure-postgresql-hyperscale-dist-alert.md)]

## <a name="generate-sample-data"></a>Создание примера данных

Теперь наша группа серверов должна быть готова к приему данных. Следующие команды можно выполнить локально в `psql` для постоянного добавления данных.

```sql
DO $$
  BEGIN LOOP
    INSERT INTO http_request (
      site_id, ingest_time, url, request_country,
      ip_address, status_code, response_time_msec
    ) VALUES (
      trunc(random()*32), clock_timestamp(),
      concat('http://example.com/', md5(random()::text)),
      ('{China,India,USA,Indonesia}'::text[])[ceil(random()*4)],
      concat(
        trunc(random()*250 + 2), '.',
        trunc(random()*250 + 2), '.',
        trunc(random()*250 + 2), '.',
        trunc(random()*250 + 2)
      )::inet,
      ('{200,404}'::int[])[ceil(random()*2)],
      5+trunc(random()*150)
    );
    COMMIT;
    PERFORM pg_sleep(random() * 0.25);
  END LOOP;
END $$;
```

Запрос выполняет вставку примерно восьми строк каждую секунду. Строки сохраняются на разных рабочих узлах, как указано в столбце распределения `site_id`.

   > [!NOTE]
   > Оставьте выполняться запрос на создание данных и откройте второй сеанс psql для ввода оставшихся команд из этого руководства.
   >

## <a name="query"></a>Запрос

Гипермасштабируемое размещение позволяет обрабатывать запросы на нескольких узлах в параллельном режиме, чтобы повысить скорость. Так, база данных вычисляет агрегаты, такие как SUM и COUNT, на рабочих узлах и объединяет результаты в итоговый ответ.

Ниже показан запрос для подсчета веб-запросов в минуту вместе с некоторой статистикой.
Попробуйте выполнить его в psql и просмотрите результаты.

```sql
SELECT
  site_id,
  date_trunc('minute', ingest_time) as minute,
  COUNT(1) AS request_count,
  SUM(CASE WHEN (status_code between 200 and 299) THEN 1 ELSE 0 END) as success_count,
  SUM(CASE WHEN (status_code between 200 and 299) THEN 0 ELSE 1 END) as error_count,
  SUM(response_time_msec) / COUNT(1) AS average_response_time_msec
FROM http_request
WHERE date_trunc('minute', ingest_time) > now() - '5 minutes'::interval
GROUP BY site_id, minute
ORDER BY minute ASC;
```

## <a name="rolling-up-data"></a>Сведение данных

Предыдущий запрос эффективен на ранних этапах, но его производительность падает по мере масштабирования. Даже при распределенной обработке будет быстрее предварительно вычислить эти данные, чем пересчитывать их каждый раз.

Быстродействие панели мониторинга можно обеспечить за счет регулярного сведения необработанных данных в таблицу агрегированных данных. Вы можете поэкспериментировать с длительностью статистической обработки. Мы использовали таблицу поминутной статистической обработки, но вместо этого вы можете разбить данные на интервалы в 5, 15 или 60 минут.

Для простого сведения мы собираемся поместить ее в функцию plpgsql. Выполните следующие команды в psql для создания функции `rollup_http_request`.

```sql
-- initialize to a time long ago
INSERT INTO latest_rollup VALUES ('10-10-1901');

-- function to do the rollup
CREATE OR REPLACE FUNCTION rollup_http_request() RETURNS void AS $$
DECLARE
  curr_rollup_time timestamptz := date_trunc('minute', now());
  last_rollup_time timestamptz := minute from latest_rollup;
BEGIN
  INSERT INTO http_request_1min (
    site_id, ingest_time, request_count,
    success_count, error_count, average_response_time_msec
  ) SELECT
    site_id,
    date_trunc('minute', ingest_time),
    COUNT(1) as request_count,
    SUM(CASE WHEN (status_code between 200 and 299) THEN 1 ELSE 0 END) as success_count,
    SUM(CASE WHEN (status_code between 200 and 299) THEN 0 ELSE 1 END) as error_count,
    SUM(response_time_msec) / COUNT(1) AS average_response_time_msec
  FROM http_request
  -- roll up only data new since last_rollup_time
  WHERE date_trunc('minute', ingest_time) <@
          tstzrange(last_rollup_time, curr_rollup_time, '(]')
  GROUP BY 1, 2;

  -- update the value in latest_rollup so that next time we run the
  -- rollup it will operate on data newer than curr_rollup_time
  UPDATE latest_rollup SET minute = curr_rollup_time;
END;
$$ LANGUAGE plpgsql;
```

Используйте функцию, чтобы выполнить сведение данных.

```sql
SELECT rollup_http_request();
```

Используя неагрегированные данные, можно выполнить запрос к таблице сведения для получения такого же отчета, как и ранее. Выполните следующий запрос:

```sql
SELECT site_id, ingest_time as minute, request_count,
       success_count, error_count, average_response_time_msec
  FROM http_request_1min
 WHERE ingest_time > date_trunc('minute', now()) - '5 minutes'::interval;
 ```

## <a name="expiring-old-data"></a>Удаление устаревших данных

Сведение позволяет быстрее обрабатывать запросы, но нам также нужно удалять устаревшие данные, чтобы избежать расходов на хранение постоянно растущего объема данных. Определите, как долго требуется хранить данные разной степени детализации и используйте стандартные запросы для удаления данных с истекшим сроком хранения. В следующем примере мы решили хранить необработанные данные за один день, а поминутно агрегированные данные — за один месяц.

```sql
DELETE FROM http_request WHERE ingest_time < now() - interval '1 day';
DELETE FROM http_request_1min WHERE ingest_time < now() - interval '1 month';
```

В рабочей среде эти запросы можно реализовать в виде функции и вызывать ее каждую минуту с помощью задания cron.

## <a name="clean-up-resources"></a>Очистка ресурсов

На предыдущих шагах вы создали ресурсы Azure в группе серверов. Если эти ресурсы вам больше не нужны, удалите группу серверов. Нажмите кнопку *Удалить* на странице *Обзор*, чтобы удалить группу серверов. При появлении запроса на всплывающей странице проверьте имя группы серверов и нажмите кнопку *Удалить*.

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как подготовить к работе группу серверов с подключенным решением Hyperscale (Citus). Вы подключились к ней с помощью psql, создали схему и выполнили распределение данных. Вы научились выполнять запросы к необработанным данным и регулярно агрегировать их, выполнять запрос к таблице агрегированных данных и удалять устаревшие данные.

- Подробнее о [типах узлов](./concepts-hyperscale-nodes.md) группы сервера.
- Определите оптимальный [первоначальный размер](howto-hyperscale-scale-initial.md) для своей группы серверов.
