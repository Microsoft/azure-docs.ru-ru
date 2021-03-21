---
title: Использование расширений PostgreSQL
description: Использование расширений PostgreSQL
titleSuffix: Azure Arc enabled data services
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 6586375d7db71274f40eb62aeb24f9daad0d7c2e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101688303"
---
# <a name="use-postgresql-extensions-in-your-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Использование расширений PostgreSQL в службе "Дуга Azure", в которой включена PostgreSQLная группа серверов

PostgreSQL лучше использовать с расширениями. На самом деле, ключевым элементом нашей собственной функции масштабирования является расширение, предоставляемое корпорацией Майкрософт, `citus` которое устанавливается по умолчанию, что позволяет postgres прозрачно распределять данные между несколькими узлами.


[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="supported-extensions"></a>Поддерживаемые расширения
Стандартные [`contrib`](https://www.postgresql.org/docs/12/contrib.html) расширения и следующие расширения уже развернуты в контейнерах группы серверов PostgreSQL в службе "Дуга", включенной в дугу Azure.
- [`citus`](https://github.com/citusdata/citus), v: 9,4. Расширение Цитус по [данным Цитус](https://www.citusdata.com/) загружается по умолчанию, так как в него передается возможность масштабирования в подсистему PostgreSQL. Удаление расширения Цитус из группы серверов Microsoft Azure Arc PostgreSQL не поддерживается.
- [`pg_cron`](https://github.com/citusdata/pg_cron), v: 1,2
- [`pgaudit`](https://www.pgaudit.org/), v: 1,4
- плпгскл, v: 1,0
- [`postgis`](https://postgis.net), v: 3.0.2
- [`plv8`](https://plv8.github.io/), v: 2.3.14

Обновления для этого списка будут публиковаться по мере развития со временем.

> [!IMPORTANT]
> Хотя в этой предварительной версии вы можете подключить к группе серверов расширение, отличное от перечисленных выше, оно не будет сохранено в системе. Это означает, что он будет недоступен после перезагрузки системы, и его потребуется снова подключить.

В этом руководство будет использоваться два из следующих расширений:
- [`PostGIS`](https://postgis.net/)
- [`pg_cron`](https://github.com/citusdata/pg_cron)

## <a name="which-extensions-need-to-be-added-to-the-shared_preload_libraries-and-created"></a>Какие расширения необходимо добавить в shared_preload_libraries и создать?

|Расширения   |Необходимо добавить в shared_preload_libraries  |Необходимо создать |
|-------------|--------------------------------------------------|---------------------- |
|`pg_cron`      |Нет       |Да        |
|`pg_audit`     |Да       |Да        |
|`plpgsql`      |Да       |Да        |
|`postgis`      |Нет       |Да        |
|`plv8`      |Нет       |Да        |

## <a name="add-extensions-to-the-shared_preload_libraries"></a>Добавление расширений в shared_preload_libraries
Дополнительные сведения о [shared_preload_libraries см. в документации по](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-SHARED-PRELOAD-LIBRARIES)PostgreSQL:
- Этот шаг не требуется для расширений, входящих в состав `contrib`
- Этот шаг не требуется для расширений, которые не требуются для предварительной загрузки с помощью shared_preload_libraries. Для этих расширений вы можете перейти к следующему абзацу [Создание расширений](https://docs.microsoft.com/azure/azure-arc/data/using-extensions-in-postgresql-hyperscale-server-group#create-extensions).

### <a name="add-an-extension-at-the-creation-time-of-a-server-group"></a>Добавление расширения во время создания группы серверов
```console
azdata arc postgres server create -n <name of your postgresql server group> --extensions <extension names>
```
### <a name="add-an-extension-to-an-instance-that-already-exists"></a>Добавление расширения в уже существующий экземпляр
```console
azdata arc postgres server edit -n <name of your postgresql server group> --extensions <extension names>
```




## <a name="show-the-list-of-extensions-added-to-shared_preload_libraries"></a>Отображение списка расширений, добавленных в shared_preload_libraries
Выполните одну из приведенных ниже команд.

### <a name="with-an-azdata-cli-command"></a>С помощью команды CLI аздата
```console
azdata arc postgres server show -n <server group name>
```
Прокрутите выходные данные и обратите внимание на разделы енгине\екстенсионс в спецификациях вашей группы серверов. Пример:
```console
"engine": {
      "extensions": [
        {
          "name": "citus"
        },
        {
          "name": "pg_cron"
        }
      ]
    },
```
### <a name="with-kubectl"></a>С kubectl
```console
kubectl describe postgresql-12s/postgres02
```
Прокрутите выходные данные и обратите внимание на разделы енгине\екстенсионс в спецификациях вашей группы серверов. Пример:
```console
Engine:
    Extensions:
      Name:  citus
      Name:  pg_cron
```


## <a name="create-extensions"></a>Создание расширений
Подключитесь к группе серверов с помощью выбранного клиентского средства и выполните стандартный запрос PostgreSQL:
```console
CREATE EXTENSION <extension name>;
```

## <a name="show-the-list-of-extensions-created"></a>Отображение списка созданных расширений
Подключитесь к группе серверов с помощью выбранного клиентского средства и выполните стандартный запрос PostgreSQL:
```console
select * from pg_extension;
```

## <a name="drop-an-extension"></a>Удалить расширение
Подключитесь к группе серверов с помощью выбранного клиентского средства и выполните стандартный запрос PostgreSQL:
```console
drop extension <extension name>;
```

## <a name="the-postgis-extension"></a>`PostGIS`Расширение
Нет необходимости добавлять `PostGIS` расширение в `shared_preload_libraries` .
Получите [демонстрационные данные](http://duspviz.mit.edu/tutorials/intro-postgis/) из отдела MIT, & планирования. Выполните команду `apt-get install unzip` , чтобы установить распаковку по мере необходимости.

```console
wget http://duspviz.mit.edu/_assets/data/intro-postgis-datasets.zip
unzip intro-postgis-datasets.zip
```

Давайте подключимся к нашей базе данных и создадим `PostGIS` расширение:

```console
CREATE EXTENSION postgis;
```

> [!NOTE]
> Если вы хотите использовать одно из расширений в `postgis` пакете (например,, `postgis_raster` ...), необходимо `postgis_topology` `postgis_sfcgal` `fuzzystrmatch` сначала создать расширение PostGIS, а затем создать другое расширение. Например: `CREATE EXTENSION postgis` ; `CREATE EXTENSION postgis_raster` ;

И создайте схему:

```sql
CREATE TABLE coffee_shops (
  id serial NOT NULL,
  name character varying(50),
  address character varying(50),
  city character varying(50),
  state character varying(50),
  zip character varying(10),
  lat numeric,
  lon numeric,
  geom geometry(POINT,4326)
);
CREATE INDEX coffee_shops_gist ON coffee_shops USING gist (geom);
```

Теперь можно сочетаться `PostGIS` с возможностями горизонтального масштабирования, сделав coffee_shopsную таблицу распределенной:

```sql
SELECT create_distributed_table('coffee_shops', 'id');
```

Давайте загрузим некоторые данные:

```console
\copy coffee_shops(id,name,address,city,state,zip,lat,lon) from cambridge_coffee_shops.csv CSV HEADER;
```

И заполните `geom` поле правильно закодированной широтой и долготой в `PostGIS` `geometry` типе данных:

```sql
UPDATE coffee_shops SET geom = ST_SetSRID(ST_MakePoint(lon,lat),4326);
```

Теперь мы можем перечислить кафе, ближайшие к MIT (77 Массачусетс AVE в 42,359055,-71,093500):

```sql
SELECT name, address FROM coffee_shops ORDER BY geom <-> ST_SetSRID(ST_MakePoint(-71.093500,42.359055),4326);
```


## <a name="the-pg_cron-extension"></a>`pg_cron`Расширение

Теперь давайте развернемся `pg_cron` к нашей группе серверов PostgreSQL, добавив ее в shared_preload_libraries:

```console
azdata postgres server update -n pg2 -ns arc --extensions pg_cron
```

Группа серверов перезапускает завершение установки расширений. Может потребоваться от 2 до 3 минут.

Теперь можно подключиться снова и создать `pg_cron` расширение:

```sql
CREATE EXTENSION pg_cron;
```

В целях тестирования позволяет создать таблицу `the_best_coffee_shop` , которая принимает случайное имя из предыдущей `coffee_shops` таблицы и вставляет содержимое таблицы:

```sql
CREATE TABLE the_best_coffee_shop(name text);
```

Можно использовать `cron.schedule` плюс несколько инструкций SQL, чтобы получить случайное имя таблицы (Обратите внимание на использование временной таблицы для хранения результата распределенного запроса) и сохранить его в `the_best_coffee_shop` :

```sql
SELECT cron.schedule('* * * * *', $$
  TRUNCATE the_best_coffee_shop;
  CREATE TEMPORARY TABLE tmp AS SELECT name FROM coffee_shops ORDER BY random() LIMIT 1;
  INSERT INTO the_best_coffee_shop SELECT * FROM tmp;
  DROP TABLE tmp;
$$);
```

Еще раз в минуту мы получаем другое имя:

```sql
SELECT * FROM the_best_coffee_shop;
```

```console
      name
-----------------
 B & B Snack Bar
(1 row)
```

Полные сведения о синтаксисе см. в [PG_CRON файле сведений](https://github.com/citusdata/pg_cron) .


## <a name="next-steps"></a>Дальнейшие действия
- Ознакомьтесь с документацией по [`plv8`](https://plv8.github.io/)
- Ознакомьтесь с документацией по [`PostGIS`](https://postgis.net/)
- Ознакомьтесь с документацией по [`pg_cron`](https://github.com/citusdata/pg_cron)
