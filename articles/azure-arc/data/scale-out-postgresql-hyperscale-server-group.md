---
title: Масштабирование базы данных Azure для группы серверов PostgreSQL Scale
description: Масштабирование базы данных Azure для группы серверов PostgreSQL Scale
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 17bdae658c7095c44a7ae9f30fd85a6c45bf1546
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96779980"
---
# <a name="scale-out-your-azure-arc-enabled-postgresql-hyperscale-server-group-by-adding-more-worker-nodes"></a>Масштабирование группы масштабируемых серверов PostgreSQL в службе "Дуга Azure" путем добавления дополнительных рабочих узлов
В этом документе объясняется, как масштабировать группу серверов PostgreSQL с поддержкой дуги Azure. Это можно сделать, выполнив сценарий. **Если вы не хотите выполнять сценарий и хотите просто прочесть сведения о том, как масштабировать, перейдите к абзацу [масштабное](#scale-out)** развертывание.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="get-started"></a>Начало работы
Если вы уже знакомы с моделью масштабирования PostgreSQLного масштабирования Azure или базы данных Azure для PostgreSQL Scale (Цитус), вы можете пропустить этот абзац. Если это не так, рекомендуется начать с ознакомления с этой моделью масштабирования на странице документации по масштабированию базы данных Azure для PostgreSQL (Цитус). Служба "база данных Azure для PostgreSQL" (Цитус) — это та же технология, которая размещена как услуга в Azure (платформа как услуга, также известная как PAAS), а не предлагается как часть служб данных, поддерживающих дугу Azure.
- [Узлы и таблицы](../../postgresql/concepts-hyperscale-nodes.md)
- [Определение типа приложения](../../postgresql/concepts-hyperscale-app-type.md)
- [Выбор столбца распределения](../../postgresql/concepts-hyperscale-choose-distribution-column.md)
- [Совместное размещение таблиц](../../postgresql/concepts-hyperscale-colocation.md)
- [Распространение и изменение таблиц](../../postgresql/howto-hyperscale-modify-distributed-tables.md)
- [Проектирование базы данных с несколькими клиентами](../../postgresql/tutorial-design-database-hyperscale-multi-tenant.md)*
- [Разработка панели мониторинга для аналитики в режиме реального времени](../../postgresql/tutorial-design-database-hyperscale-realtime.md)*

> \* В приведенных выше документах пропустите разделы **Вход в портал Azure**& **создать базу данных Azure для PostgreSQL-Scale (Цитус)**. Реализуйте оставшиеся шаги в развертывании Azure ARC. Эти разделы относятся к службе "база данных Azure для PostgreSQL" (Цитус), предлагаемой в качестве службы PaaS в облаке Azure, но другие части документов напрямую применимы к вашей геомасштабированию Azure с включенной PostgreSQL.

## <a name="scenario"></a>Сценарий
Этот сценарий относится к группе серверов PostgreSQL Scale, которая была создана в качестве примера в документации по [созданию группы серверов Microsoft Azure Arc PostgreSQL](create-postgresql-hyperscale-server-group.md) .

### <a name="load-test-data"></a>Загрузка тестовых данных
В сценарии используется пример общедоступных данных GitHub, доступных на [веб-сайте данных Цитус](https://www.citusdata.com/) (данные Цитус являются частью Microsoft).

#### <a name="connect-to-your-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Подключитесь к своей службе "Дуга Azure" PostgreSQL "Масштабируемая группа серверов"

##### <a name="list-the-connection-information"></a>Вывод сведений о подключении
Чтобы получить сведения о подключении, подключитесь к службе "Дуга Azure с включенной PostgreSQL". для этого в качестве общего формата этой команды используется
```console
azdata arc postgres endpoint list -n <server name>
```
Например, такое:
```console
azdata arc postgres endpoint list -n postgres01
```

Выходные данные примера:

```console
[
  {
    "Description": "PostgreSQL Instance",
    "Endpoint": "postgresql://postgres:<replace with password>@12.345.123.456:1234"
  },
  {
    "Description": "Log Search Dashboard",
    "Endpoint": "https://12.345.123.456:12345/kibana/app/kibana#/discover?_a=(query:(language:kuery,query:'custom_resource_name:\"postgres01\"'))"
  },
  {
    "Description": "Metrics Dashboard",
    "Endpoint": "https://12.345.123.456:12345/grafana/d/postgres-metrics?var-Namespace=arc3&var-Name=postgres01"
  }
]
```

##### <a name="connect-with-the-client-tool-of-your-choice"></a>Подключитесь с помощью выбранного клиентского средства.

Выполните следующий запрос, чтобы убедиться в наличии двух (или нескольких рабочих узлов), каждый из которых соответствует Kubernetes Pod:

```sql
SELECT * FROM pg_dist_node;
```

```console
 nodeid | groupid |                       nodename                        | nodeport | noderack | hasmetadata | isactive | noderole | nodecluster | metadatasynced | shouldhaveshards
--------+---------+-------------------------------------------------------+----------+----------+-------------+----------+----------+-------------+----------------+------------------
      1 |       1 | pg1-1.pg1-svc.default.svc.cluster.local |     5432 | default  | f           | t        | primary  | default     | f              | t
      2 |       2 | pg1-2.pg1-svc.default.svc.cluster.local |     5432 | default  | f           | t        | primary  | default     | f              | t
(2 rows)
```

#### <a name="create-a-sample-schema"></a>Создание образца схемы
Создайте две таблицы, выполнив следующий запрос:

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

ЖСОНБ — это тип данных JSON в двоичном формате в PostgreSQL. Она хранит гибкую схему в одном столбце и с помощью PostgreSQL. Схема будет иметь индекс Ло для индексации всех ключей и значений внутри него. Ло индекс позволяет быстро и легко запрашивать различные условия непосредственно на этих полезных данных. Итак, мы создадим несколько индексов, прежде чем загружать наши данные:

```sql
CREATE INDEX event_type_index ON github_events (event_type);
CREATE INDEX payload_index ON github_events USING GIN (payload jsonb_path_ops);
```

Для сегментирования стандартных таблиц выполните запрос для каждой таблицы. Укажите таблицу, для которой необходимо создать сегментирование, и ключ, на котором она должна быть сегментирована. Мы будем сегментирование таблицы Events and Users на user_id:

```sql
SELECT create_distributed_table('github_events', 'user_id');
SELECT create_distributed_table('github_users', 'user_id');
```

#### <a name="load-sample-data"></a>Загрузка примера данных
Загрузить данные с помощью копирования... ИЗ ПРОГРАММЫ:

```sql
COPY github_users FROM PROGRAM 'curl "https://examples.citusdata.com/users.csv"' WITH ( FORMAT CSV );
COPY github_events FROM PROGRAM 'curl "https://examples.citusdata.com/events.csv"' WITH ( FORMAT CSV );
```

#### <a name="query-the-data"></a>Запрос данных
Теперь можно измерять, как долго выполняется простой запрос с двумя узлами:

```sql
SELECT COUNT(*) FROM github_events;
```
Запишите время выполнения запроса.


## <a name="scale-out"></a>Горизонтальное увеличение масштаба
Общий формат команды масштабирования:
```console
azdata arc postgres server edit -n <server group name> -w <target number of worker nodes>
```

> [!CAUTION]
> Выпуск предварительной версии не поддерживает обратное масштабирование. Например, пока невозможно сократить количество рабочих узлов. Если это необходимо, то нужно извлечь или создать резервную копию данных, удалить группу серверов, создать новую группу серверов с меньшим количеством рабочих узлов, а затем импортировать данные.

В этом примере мы увеличив число рабочих узлов с 2 до 4, выполнив следующую команду:

```console
azdata arc postgres server edit -n postgres01 -w 4
```

После добавления узлов вы увидите состояние ожидания для группы серверов. Пример:
```console
azdata arc postgres server list
```

```console
Name        State          Workers
----------  -------------  ---------
postgres01  Pending 4/5    4
```

После того как узлы будут доступны, подсистема балансировки для сегментов будет автоматически запущена, а данные перераспределяются на новые узлы. Операция горизонтального масштабирования выполняется в оперативном режиме. При добавлении узлов и перераспределении данных между узлами данные остаются доступными для запросов.

### <a name="verify-the-new-shape-of-the-server-group-optional"></a>Проверка новой формы группы серверов (необязательно)
Используйте один из приведенных ниже методов, чтобы убедиться, что группа серверов теперь использует дополнительные рабочие узлы, которые вы добавили.

#### <a name="with-azdata"></a>С аздата:
Выполните приведенную ниже команду.
```console
azdata arc postgres server list
```

Он возвращает список групп серверов, созданных в пространстве имен, и указывает их количество рабочих узлов. Пример:
```console
Name        State    Workers
----------  -------  ---------
postgres01  Ready    4
```

#### <a name="with-kubectl"></a>С kubectl:
Выполните приведенную ниже команду.
```console
kubectl get postgresql-12
```

Он возвращает список групп серверов, созданных в пространстве имен, и указывает их количество рабочих узлов. Пример:
```console
NAME         STATE   READY-PODS   EXTERNAL-ENDPOINT   AGE
postgres01   Ready   4/4          10.0.0.4:31066      4d20h
```
> [!NOTE]
> Если вы создали группу серверов версии 11 PostgreSQL, а не 12, выполните следующую команду: _kubectl Get PostgreSQL-11_ .

#### <a name="with-a-sql-query"></a>С SQL-запросом:
Подключитесь к группе серверов с помощью выбранного клиентского средства и выполните следующий запрос:

```sql
SELECT * FROM pg_dist_node;
```

```console
 nodeid | groupid |                       nodename                        | nodeport | noderack | hasmetadata | isactive | noderole | nodecluster | metadatasynced | shouldhaveshards
--------+---------+-------------------------------------------------------+----------+----------+-------------+----------+----------+-------------+----------------+------------------
      1 |       1 | pg1-1.pg1-svc.default.svc.cluster.local |     5432 | default  | f           | t        | primary  | default     | f              | t
      2 |       2 | pg1-2.pg1-svc.default.svc.cluster.local |     5432 | default  | f           | t        | primary  | default     | f              | t
      3 |       3 | pg1-3.pg1-svc.default.svc.cluster.local |     5432 | default  | f           | t        | primary  | default     | f              | t
      4 |       4 | pg1-4.pg1-svc.default.svc.cluster.local |     5432 | default  | f           | t        | primary  | default     | f              | t
(4 rows)
```

## <a name="return-to-the-scenario"></a>Вернуться к сценарию

Если вы хотите сравнить время выполнения запроса SELECT Count с образцом набора данных, используйте тот же запрос Count. Его можно использовать на четырех рабочих узлах без каких бы то ни было изменений в инструкции SQL.

```sql
SELECT COUNT(*) FROM github_events;
```
Обратите внимание на время выполнения.


> [!NOTE]
> В зависимости от среды — например, если вы развернули группу тестовых серверов `kubeadm` на виртуальной машине с одним узлом, вы можете увидеть небольшое повышение времени выполнения. Чтобы лучше понять тип улучшения производительности, который можно получить с помощью PostgreSQLного масштабирования Azure с поддержкой Arc, просмотрите следующие короткие видеоматериалы:
>* [Высокая производительность HTAP с помощью масштабирования PostgreSQL Azure (Цитус)](https://www.youtube.com/watch?v=W_3e07nGFxY)
>* [Создание HTAP приложений с помощью Python & Azure PostgreSQL Scale (Цитус)](https://www.youtube.com/watch?v=YDT8_riLLs0)


## <a name="next-steps"></a>Дальнейшие действия

- Узнайте о том [, как увеличить и уменьшить масштаб (память, виртуальных ядер) в вашей ГеоPostgreSQLной группе серверов Azure Arc](scale-up-down-postgresql-hyperscale-server-group-using-cli.md)
- Узнайте о том, как задать параметры сервера в группе PostgreSQL "масштабируемый сервер" в службе "Дуга Azure"
- Ознакомьтесь с основными понятиями и руководствами по использованию службы "база данных Azure для PostgreSQL", чтобы распределить данные между несколькими узлами PostgreSQL, а также получить преимущества от всех возможностей службы "база данных Azure для postgres". :
    * [Узлы и таблицы](../../postgresql/concepts-hyperscale-nodes.md)
    * [Определение типа приложения](../../postgresql/concepts-hyperscale-app-type.md)
    * [Выбор столбца распределения](../../postgresql/concepts-hyperscale-choose-distribution-column.md)
    * [Совместное размещение таблиц](../../postgresql/concepts-hyperscale-colocation.md)
    * [Распространение и изменение таблиц](../../postgresql/howto-hyperscale-modify-distributed-tables.md)
    * [Проектирование базы данных с несколькими клиентами](../../postgresql/tutorial-design-database-hyperscale-multi-tenant.md)*
    * [Разработка панели мониторинга для аналитики в режиме реального времени](../../postgresql/tutorial-design-database-hyperscale-realtime.md)*

 > \* В приведенных выше документах пропустите разделы **Вход в портал Azure**& **создать базу данных Azure для PostgreSQL-Scale (Цитус)**. Реализуйте оставшиеся шаги в развертывании Azure ARC. Эти разделы относятся к службе "база данных Azure для PostgreSQL" (Цитус), предлагаемой в качестве службы PaaS в облаке Azure, но другие части документов напрямую применимы к вашей геомасштабированию Azure с включенной PostgreSQL.

- [Основные понятия конфигурации хранилища и хранилища Kubernetes](storage-configuration.md)
- [Модель ресурсов Kubernetes](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities)
