---
title: Индексирование в учетной записи API Cassandra Azure Cosmos DB
description: Узнайте, как работает вторичное индексирование в учетной записи Azure Azure Cosmos DB API Cassandra.
author: TheovanKraay
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: conceptual
ms.date: 04/04/2020
ms.author: thvankra
ms.reviewer: sngun
ms.openlocfilehash: 98e8f713ad2e4eef47e40d89a23dbf49a98ad67c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93339902"
---
# <a name="secondary-indexing-in-azure-cosmos-db-cassandra-api"></a>Дополнительное индексирование в Azure Cosmos DB API Cassandra
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

API Cassandra в Azure Cosmos DB использует базовую инфраструктуру индексирования для предоставления силы индексирования, характерной для платформы. Однако, в отличие от основного API SQL, API Cassandra в Azure Cosmos DB не индексирует все атрибуты по умолчанию. Вместо этого он поддерживает вторичное индексирование, чтобы создать индекс для определенных атрибутов, который ведет себя так же, как Apache Cassandra.  

Как правило, не рекомендуется выполнять запросы фильтра для столбцов, которые не секционированы. Необходимо явно использовать синтаксис инструкции ALLOW Filter, что приводит к неправильной работе операции. В Azure Cosmos DB можно выполнять такие запросы с атрибутами низкого уровня кратности, так как они выводятся из одного раздела в разделы для получения результатов.

Не рекомендуется создавать индекс для часто обновляемого столбца. При определении таблицы разумно создать индекс. Это гарантирует, что данные и индексы будут находиться в стабильном состоянии. Если вы создаете новый индекс для существующих данных, в настоящее время отслеживать изменение хода выполнения индекса для таблицы невозможно. Если необходимо отслеживать ход выполнения этой операции, необходимо запросить изменение хода выполнения с помощью запроса в [службу поддержки]( https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request).


> [!NOTE]
> Вторичный индекс не поддерживается для следующих объектов:
> - типы данных, такие как замороженные типы коллекций, типы Decimal и Variant.
> - Статические столбцы
> - Ключи кластеризации

## <a name="indexing-example"></a>Пример индексирования

Сначала создайте пример пространства ключей и таблицу, выполнив следующие команды в командной строке оболочки CQL:

```shell
CREATE KEYSPACE sampleks WITH REPLICATION = {'class' : 'SimpleStrategy'};
CREATE TABLE sampleks.t1(user_id int PRIMARY KEY, lastname text) WITH cosmosdb_provisioned_throughput=400;
``` 

Затем вставьте пример данных пользователя с помощью следующих команд:

```shell
insert into sampleks.t1(user_id,lastname) values (1, 'nishu');
insert into sampleks.t1(user_id,lastname) values (2, 'vinod');
insert into sampleks.t1(user_id,lastname) values (3, 'bat');
insert into sampleks.t1(user_id,lastname) values (5, 'vivek');
insert into sampleks.t1(user_id,lastname) values (6, 'siddhesh');
insert into sampleks.t1(user_id,lastname) values (7, 'akash');
insert into sampleks.t1(user_id,lastname) values (8, 'Theo');
insert into sampleks.t1(user_id,lastname) values (9, 'jagan');
```

Если вы попробуете выполнить следующую инструкцию, вы получите сообщение об ошибке, в котором будет предложено использовать `ALLOW FILTERING` : 

```shell
select user_id, lastname from sampleks.t1 where lastname='nishu';
``` 

Хотя API Cassandra поддерживает ФИЛЬТРАЦИю, как упоминалось в предыдущем разделе, делать это не рекомендуется. Вместо этого следует создать индекс в, как показано в следующем примере:

```shell
CREATE INDEX ON sampleks.t1 (lastname);
```
После создания индекса в поле LastName теперь можно успешно выполнить предыдущий запрос. При использовании API Cassandra в Azure Cosmos DB не нужно указывать имя индекса. Используется индекс по умолчанию с форматом `tablename_columnname_idx` . Например, ` t1_lastname_idx` — это имя индекса для предыдущей таблицы.

## <a name="dropping-the-index"></a>Удаление индекса 
Необходимо знать имя индекса для удаления индекса. Выполните `desc schema` команду, чтобы получить описание таблицы. Выходные данные этой команды включают имя индекса в формате `CREATE INDEX tablename_columnname_idx ON keyspacename.tablename(columnname)` . Затем можно использовать имя индекса для удаления индекса, как показано в следующем примере:

```shell
drop index sampleks.t1_lastname_idx;
```

## <a name="next-steps"></a>Дальнейшие действия
* Узнайте, как работает [Автоматическое индексирование](index-overview.md) в Azure Cosmos DB
* [Функции Apache Cassandra, поддерживаемые API Cassandra для Azure Cosmos DB](cassandra-support.md)
