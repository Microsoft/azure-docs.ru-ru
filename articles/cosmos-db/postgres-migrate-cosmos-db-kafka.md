---
title: Перенос данных из PostgreSQL в учетную запись API Cassandra Azure Cosmos DB с помощью Apache Kafka
description: Узнайте, как использовать Kafka Connect для синхронизации данных из PostgreSQL с Azure Cosmos DB API Cassandra в режиме реального времени.
author: abhirockzz
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 01/05/2021
ms.author: abhishgu
ms.reviewer: abhishgu
ms.openlocfilehash: 0038219ee8c1721ff5ab2be76231d33d2bd9064d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98203071"
---
# <a name="migrate-data-from-postgresql-to-azure-cosmos-db-cassandra-api-account-using-apache-kafka"></a>Перенос данных из PostgreSQL в учетную запись API Cassandra Azure Cosmos DB с помощью Apache Kafka
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

API Cassandra в Azure Cosmos DB стали отличным выбором для корпоративных рабочих нагрузок, работающих на Apache Cassandra, по различным причинам:

* **Значительные сокращения затрат:** Вы можете сэкономить затраты с Azure Cosmos DB, включая стоимость виртуальных машин, пропускной способности и применимых лицензий Oracle. Кроме того, вам не нужно управлять центрами обработки данных, серверами, хранилищем SSD, сетью и расходами электроэнергии.

* **Улучшенная масштабируемость и доступность:** Она устраняет единые точки отказа, улучшает масштабируемость и доступность приложений.

* **Отсутствие дополнительных издержек при управлении и мониторинге:** В качестве полностью управляемой облачной службы Azure Cosmos DB устраняет издержки на управление множеством параметров и наблюдение за ними.

[Kafka Connect](https://kafka.apache.org/documentation/#connect) — это платформа для потоковой передачи данных между [Apache Kafka](https://kafka.apache.org/) и другими системами в масштабируемом и надежном режиме. Он поддерживает несколько соединителей полки. Это означает, что вам не нужен пользовательский код для интеграции внешних систем с Apache Kafka.

В этой статье показано, как использовать сочетание соединителей Kafka для настройки конвейера данных для непрерывной синхронизации записей из реляционной базы данных, например [PostgreSQL](https://www.postgresql.org/) [Azure Cosmos DB API Cassandra](cassandra-introduction.md).

## <a name="overview"></a>Обзор

Ниже приведен общий обзор сквозного потока, представленного в этой статье.

Данные в таблице PostgreSQL будут отправлены в Apache Kafka с помощью [соединителя Дебезиум PostgreSQL](https://debezium.io/documentation/reference/1.2/connectors/postgresql.html), который является соединителем Kafka Connect **Source** . Операции вставки, обновления или удаления записей в таблице PostgreSQL будут регистрироваться в виде `change data` событий и отправляться в разделы Kafka. [Соединитель DataStax Apache Kafka](https://docs.datastax.com/en/kafka/doc/kafka/kafkaIntro.html) (соединитель **приемника** Kafka Connect) формирует вторую часть конвейера. Он синхронизирует события изменения данных из Kafka, чтобы Azure Cosmos DB таблицы API Cassandra.

> [!NOTE]
> Использование конкретных функций соединителя DataStax Apache Kafka позволяет отправлять данные в несколько таблиц. В этом примере соединитель поможет нам сохранить записи измененных данных в две Cassandra таблицы, которые могут поддерживать различные требования к запросам.

## <a name="prerequisites"></a>Предварительные условия

* [Предоставление учетной записи API Cassandra для Azure Cosmos DB](create-cassandra-dotnet.md#create-a-database-account)
* [Использование cqlsh или размещенной оболочки для проверки](cassandra-support.md#hosted-cql-shell-preview)
* JDK 8 или более поздней версии
* [DOCKER](https://www.docker.com/) (необязательно)

## <a name="base-setup"></a>Базовая настройка

### <a name="set-up-postgresql-database-if-you-havent-already"></a>Если вы еще этого не сделали, настройте базу данных PostgreSQL. 

Это может быть существующая локальная база данных, или вы можете [скачать и установить](https://www.postgresql.org/download/) ее на локальном компьютере. Также можно использовать [контейнер DOCKER](https://hub.docker.com/_/postgres).

Чтобы запустить контейнер, выполните следующие действия.

```bash
docker run --rm -p 5432:5432 -e POSTGRES_PASSWORD=<enter password> postgres
```

Подключитесь к экземпляру PostgreSQL с помощью [`psql`](https://www.postgresql.org/docs/current/app-psql.html) клиента:

```bash
psql -h localhost -p 5432 -U postgres -W -d postgres
```

Создайте таблицу для хранения примеров сведений о заказах:

```sql
CREATE SCHEMA retail;

CREATE TABLE retail.orders_info (
    orderid SERIAL NOT NULL PRIMARY KEY,
    custid INTEGER NOT NULL,
    amount INTEGER NOT NULL,
    city VARCHAR(255) NOT NULL,
    purchase_time VARCHAR(40) NOT NULL
);
```

### <a name="using-the-azure-portal-create-the-cassandra-keyspace-and-the-tables-required-for-the-demo-application"></a>С помощью портал Azure создайте пространства ключей Cassandra и таблицы, необходимые для демонстрационного приложения.

> [!NOTE]
> Используйте те же имена пространства ключей и таблицы, которые приведены ниже.

```sql
CREATE KEYSPACE retail WITH REPLICATION = {'class' : 'NetworkTopologyStrategy', 'datacenter1' : 1};

CREATE TABLE retail.orders_by_customer (order_id int, customer_id int, purchase_amount int, city text, purchase_time timestamp, PRIMARY KEY (customer_id, purchase_time)) WITH CLUSTERING ORDER BY (purchase_time DESC) AND cosmosdb_cell_level_timestamp=true AND cosmosdb_cell_level_timestamp_tombstones=true AND cosmosdb_cell_level_timetolive=true;

CREATE TABLE retail.orders_by_city (order_id int, customer_id int, purchase_amount int, city text, purchase_time timestamp, PRIMARY KEY (city,order_id)) WITH cosmosdb_cell_level_timestamp=true AND cosmosdb_cell_level_timestamp_tombstones=true AND cosmosdb_cell_level_timetolive=true;
```

### <a name="setup-apache-kafka"></a>Apache Kafka установки 

В этой статье используется локальный кластер, но можно выбрать любой другой вариант. [Скачайте Kafka](https://kafka.apache.org/downloads), распакуйте его, запустите кластер Zookeeper и Kafka.

```bash
cd <KAFKA_HOME>/bin

#start zookeeper
bin/zookeeper-server-start.sh config/zookeeper.properties

#start kafka (in another terminal)
bin/kafka-server-start.sh config/server.properties
```

### <a name="setup-connectors"></a>Установка соединителей

Установите соединитель Дебезиум PostgreSQL и DataStax Apache Kafka. Скачайте архив подключаемого модуля соединителя Дебезиум PostgreSQL. Например, чтобы скачать версию 1.3.0 соединителя (последняя версия на момент написания статьи), используйте [эту ссылку](https://repo1.maven.org/maven2/io/debezium/debezium-connector-postgres/1.3.0.Final/debezium-connector-postgres-1.3.0.Final-plugin.tar.gz). Скачайте соединитель Apache Kafka DataStax по [этой ссылке](https://downloads.datastax.com/#akc).

Распакуйте оба архива соединителя и скопируйте JAR-файлы в [подключаемый модуль Kafka Connect. путь](https://kafka.apache.org/documentation/#connectconfigs).


```bash
cp <path_to_debezium_connector>/*.jar <KAFKA_HOME>/libs
cp <path_to_cassandra_connector>/*.jar <KAFKA_HOME>/libs
```

> Дополнительные сведения см. в документации по [дебезиум](https://debezium.io/documentation/reference/1.2/connectors/postgresql.html#postgresql-deploying-a-connector) и [DataStax](https://docs.datastax.com/en/kafka/doc/) .

## <a name="configure-kafka-connect-and-start-data-pipeline"></a>Настройка Kafka Connect и запуск конвейера данных

### <a name="start-kafka-connect-cluster"></a>Запуск кластера Kafka Connect

```bash
cd <KAFKA_HOME>/bin
./connect-distributed.sh ../config/connect-distributed.properties
```

### <a name="start-postgresql-connector-instance"></a>Запуск экземпляра соединителя PostgreSQL

Пример сохранения конфигурации соединителя (JSON) в файле `pg-source-config.json`

```json
{
    "name": "pg-orders-source",
    "config": {
        "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
        "database.hostname": "localhost",
        "database.port": "5432",
        "database.user": "postgres",
        "database.password": "password",
        "database.dbname": "postgres",
        "database.server.name": "myserver",
        "plugin.name": "wal2json",
        "table.include.list": "retail.orders_info",
        "value.converter": "org.apache.kafka.connect.json.JsonConverter"
    }
}
```

Чтобы запустить экземпляр соединителя PostgreSQL, выполните следующие действия.

```bash
curl -X POST -H "Content-Type: application/json" --data @pg-source-config.json http://localhost:8083/connectors
```

> [!NOTE]
> Для удаления можно использовать: `curl -X DELETE http://localhost:8083/connectors/pg-orders-source`


### <a name="insert-data"></a>Добавление данных

`orders_info`Таблица содержит подробные сведения о заказе, такие как идентификатор заказа, идентификатор клиента, город и т. д. Заполните таблицу случайными данными, используя приведенный ниже SQL.

```sql
insert into retail.orders_info (
    custid, amount, city, purchase_time
)
select
    random() * 10000 + 1,
    random() * 200,
    ('{New Delhi,Seattle,New York,Austin,Chicago,Cleveland}'::text[])[ceil(random()*3)],
    NOW() + (random() * (interval '1 min'))
from generate_series(1, 10) s(i);
```

Он должен вставить 10 записей в таблицу. Не забудьте обновить число записей `generate_series(1, 10)` ниже в соответствии с требованиями к примеру, чтобы вставить `100` записи, используйте `generate_series(1, 100)`

Для подтверждения:

```bash
select * from retail.orders_info;
```

Проверьте события отслеживания измененных данных в разделе Kafka

> [!NOTE]
> Обратите внимание, что `myserver.retail.orders_info` согласно [соглашению о соединителях](https://debezium.io/documentation/reference/1.3/connectors/postgresql.html#postgresql-topic-names) указано имя раздела.

```bash
cd <KAFKA_HOME>/bin

./kafka-console-consumer.sh --topic myserver.retail.orders_info --bootstrap-server localhost:9092 --from-beginning
```

Вы должны увидеть события изменения данных в формате JSON.

### <a name="start-datastax-apache-kafka-connector-instance"></a>Запуск экземпляра соединителя DataStax Apache Kafka

Сохраните конфигурацию соединителя (JSON) в файл, `cassandra-sink-config.json` а затем обновите свойства в соответствии со своей средой.

```json
{
    "name": "kafka-cosmosdb-sink",
    "config": {
        "connector.class": "com.datastax.oss.kafka.sink.CassandraSinkConnector",
        "tasks.max": "1",
        "topics": "myserver.retail.orders_info",
        "contactPoints": "<Azure Cosmos DB account name>.cassandra.cosmos.azure.com",
        "loadBalancing.localDc": "<Azure Cosmos DB region e.g. Southeast Asia>",
        "datastax-java-driver.advanced.connection.init-query-timeout": 5000,
        "ssl.hostnameValidation": true,
        "ssl.provider": "JDK",
        "ssl.keystore.path": "<path to JDK keystore path e.g. <JAVA_HOME>/jre/lib/security/cacerts>",
        "ssl.keystore.password": "<keystore password: it is 'changeit' by default>",
        "port": 10350,
        "maxConcurrentRequests": 500,
        "maxNumberOfRecordsInBatch": 32,
        "queryExecutionTimeout": 30,
        "connectionPoolLocalSize": 4,
        "auth.username": "<Azure Cosmos DB user name (same as account name)>",
        "auth.password": "<Azure Cosmos DB password>",
        "topic.myserver.retail.orders_info.retail.orders_by_customer.mapping": "order_id=value.orderid, customer_id=value.custid, purchase_amount=value.amount, city=value.city, purchase_time=value.purchase_time",
        "topic.myserver.retail.orders_info.retail.orders_by_city.mapping": "order_id=value.orderid, customer_id=value.custid, purchase_amount=value.amount, city=value.city, purchase_time=value.purchase_time",
        "key.converter": "org.apache.kafka.connect.storage.StringConverter",
        "transforms": "unwrap",
        "transforms.unwrap.type": "io.debezium.transforms.ExtractNewRecordState",
        "offset.flush.interval.ms": 10000
    }
}
```

Чтобы запустить экземпляр соединителя, выполните следующие действия.

```bash
curl -X POST -H "Content-Type: application/json" --data @cassandra-sink-config.json http://localhost:8083/connectors
```

Соединитель должен быть в действии, а сквозной конвейер от PostgreSQL до Azure Cosmos DB будет работать.

### <a name="query-azure-cosmos-db"></a>Запрос БД Azure Cosmos DB

Проверьте таблицы Cassandra в Azure Cosmos DB. Ниже приведены некоторые запросы, которые можно использовать.

```sql
select count(*) from retail.orders_by_customer;
select count(*) from retail.orders_by_city;

select * from retail.orders_by_customer;
select * from retail.orders_by_city;

select * from retail.orders_by_city where city='Seattle';
select * from retail.orders_by_customer where customer_id = 10;
```

Можно продолжить вставку данных в PostgreSQL и убедиться, что записи синхронизированы с Azure Cosmos DB.

## <a name="next-steps"></a>Дальнейшие действия

* [Интеграция Apache Kafka и Azure Cosmos DB API Cassandra с помощью Kafka Connect](cassandra-kafka-connect.md)
* [Интеграция Apache Kafka подключение в концентраторах событий Azure (Предварительная версия) с Дебезиум для отслеживания измененных данных](../event-hubs/event-hubs-kafka-connect-debezium.md)
* [Перенос данных из Oracle в Azure Cosmos DB API Cassandra с помощью Блитзз](oracle-migrate-cosmos-db-blitzz.md)
* [Обеспечение необходимой пропускной способности для контейнеров и баз данных](set-throughput.md) 
* [Рекомендации по использованию ключа секции](partitioning-overview.md#choose-partitionkey)
* [Оценка единиц запросов в секунду с помощью планировщика ресурсов Azure Cosmos DB](estimate-ru-with-capacity-planner.md)

