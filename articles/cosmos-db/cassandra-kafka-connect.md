---
title: Интеграция Apache Kafka и Azure Cosmos DB API Cassandra с помощью Kafka Connect
description: Узнайте, как принимать данные из Kafka в Azure Cosmos DB API Cassandra с помощью соединителя DataStax Apache Kafka
author: abhirockzz
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 12/14/2020
ms.author: abhishgu
ms.reviewer: abhishgu
ms.openlocfilehash: 25972ba2bb30c39838c4822a42af292e8d8b1dba
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97803635"
---
# <a name="ingest-data-from-apache-kafka-into-azure-cosmos-db-cassandra-api-using-kafka-connect"></a>Прием данных из Apache Kafka в Azure Cosmos DB API Cassandra с помощью Kafka Connect
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

Существующие приложения Cassandra могут легко работать с [Azure Cosmos DB API Cassandra](cassandra-introduction.md) из-за [совместимости драйвера CQLv4](https://cassandra.apache.org/doc/latest/getting_started/drivers.html?highlight=driver). Эту возможность можно использовать для интеграции с такими платформами потоковой передачи, как [Apache Kafka](https://kafka.apache.org/) и переноса данных в Azure Cosmos DB.

Данные в Apache Kafka (разделы) полезны только при использовании другими приложениями или в других системах. Можно создать решение с помощью API-интерфейсов [производителя или потребителя Kafka](https://kafka.apache.org/documentation/#api) , [используя выбранный язык и клиентский пакет SDK](https://cwiki.apache.org/confluence/display/KAFKA/Clients). Kafka Connect предоставляет альтернативное решение. Это платформа для потоковой передачи данных между Apache Kafka и другими системами в масштабируемом и надежном режиме. Так как Kafka Connect поддерживает соединители полок, которые включают Cassandra, вам не нужно писать пользовательский код для интеграции Kafka с Azure Cosmos DB API Cassandra. 

В этой статье мы будем использовать [соединитель DataStax Apache Kafka](https://docs.datastax.com/en/kafka/doc/kafka/kafkaIntro.html)с открытым кодом, который работает поверх Kafka Connect Framework, чтобы принимать записи из раздела Kafka в строки одной или нескольких Cassandra таблиц. В примере можно повторно использовать программу установки с помощью Docker Compose. Это очень удобно, поскольку оно позволяет локально загрузить все необходимые компоненты с помощью одной команды. К таким компонентам относятся Kafka, Zookeeper, Kafka Connect Worker и пример приложения генератора данных.

Ниже приведена разбивка компонентов и их определений служб. Вы можете обратиться к полному `docker-compose` файлу [в репозитории GitHub](https://github.com/Azure-Samples/cosmosdb-cassandra-kafka/blob/main/docker-compose.yaml).

- Kafka и Zookeeper используют образы [дебезиум](https://hub.docker.com/r/debezium/kafka/) .
- Для запуска в качестве контейнера DOCKER соединитель DataStax Apache Kafka помогут поверх существующего образа DOCKER- [дебезиум/Connect-Base](https://github.com/debezium/docker-images/tree/master/connect-base/1.2). Этот образ включает установку Kafka и его библиотек Kafka Connect, что делает его действительно удобным для добавления пользовательских соединителей. Вы можете обратиться к [Dockerfile](https://github.com/Azure-Samples/cosmosdb-cassandra-kafka/blob/main/connector/Dockerfile).
- `data-generator`Служба заполняет случайным образом сформированные (JSON) данные в `weather-data` разделе Kafka. Вы можете ссылаться на код и `Dockerfile` в [репозитории GitHub](https://github.com/Azure-Samples/cosmosdb-cassandra-kafka/blob/main/data-generator/) .

## <a name="prerequisites"></a>Предварительные условия

* [Предоставление учетной записи API Cassandra для Azure Cosmos DB](create-cassandra-dotnet.md#create-a-database-account)

* [Использование cqlsh или размещенной оболочки для проверки](cassandra-support.md#hosted-cql-shell-preview)

* Установка [DOCKER](https://docs.docker.com/get-docker/) и [DOCKER Compose](https://docs.docker.com/compose/install)

## <a name="create-keyspace-tables-and-start-the-integration-pipeline"></a>Создание пространства ключей, таблиц и запуск конвейера интеграции

С помощью портал Azure создайте пространства ключей Cassandra и таблицы, необходимые для демонстрационного приложения.

> [!NOTE]
> Используйте те же имена пространства ключей и таблицы, которые приведены ниже.

```sql
CREATE KEYSPACE weather WITH REPLICATION = {'class' : 'NetworkTopologyStrategy', 'datacenter1' : 1};

CREATE TABLE weather.data_by_state (station_id text, temp int, state text, ts timestamp, PRIMARY KEY (state, ts)) WITH CLUSTERING ORDER BY (ts DESC) AND cosmosdb_cell_level_timestamp=true AND cosmosdb_cell_level_timestamp_tombstones=true AND cosmosdb_cell_level_timetolive=true;

CREATE TABLE weather.data_by_station (station_id text, temp int, state text, ts timestamp, PRIMARY KEY (station_id, ts)) WITH CLUSTERING ORDER BY (ts DESC) AND cosmosdb_cell_level_timestamp=true AND cosmosdb_cell_level_timestamp_tombstones=true AND cosmosdb_cell_level_timetolive=true;
```

Клонировать репозиторий GitHub:

```bash
git clone https://github.com/Azure-Samples/cosmosdb-cassandra-kafka
cd cosmosdb-cassandra-kafka
```

Запустите все службы:

```shell
docker-compose --project-name kafka-cosmos-cassandra up --build
```

> [!NOTE]
> Загрузка и запуск контейнеров может занять некоторое время: это всего один процесс.

Чтобы проверить, запущены ли все контейнеры, выполните следующие действия.

```shell
docker-compose -p kafka-cosmos-cassandra ps
```

Приложение генератора данных начнет передавать данные в `weather-data` раздел Kafka. Для подтверждения можно также выполнить быструю работоспособности проверку. Просматриваете контейнер DOCKER, выполняющий рабочую роль Kafka Connect:


```bash
docker exec -it kafka-cosmos-cassandra_cassandra-connector_1 bash
```

После перетаскивания в оболочку контейнеров просто запустите обычный процесс-потребитель консоли Kafka, и вы должны увидеть данные о погоде (в формате JSON), переходящие в.

```bash
cd ../bin
./kafka-console-consumer.sh --bootstrap-server kafka:9092 --topic weather-data
```

## <a name="cassandra-sink-connector-setup"></a>Установка соединителя приемника Cassandra

Скопируйте приведенное ниже содержимое JSON в файл (можно присвоить ему имя `cassandra-sink-config.json` ). Его необходимо обновить в соответствии с настройками, а остальная часть этого раздела будет предоставлять рекомендации по этой теме.

```json
{
    "name": "kafka-cosmosdb-sink",
    "config": {
        "connector.class": "com.datastax.oss.kafka.sink.CassandraSinkConnector",
        "tasks.max": "1",
        "topics": "weather-data",
        "contactPoints": "<cosmos db account name>.cassandra.cosmos.azure.com",
        "port": 10350,
        "loadBalancing.localDc": "<cosmos db region e.g. Southeast Asia>",
        "auth.username": "<enter username for cosmosdb account>",
        "auth.password": "<enter password for cosmosdb account>",
        "ssl.hostnameValidation": true,
        "ssl.provider": "JDK",
        "ssl.keystore.path": "/etc/alternatives/jre/lib/security/cacerts/",
        "ssl.keystore.password": "changeit",
        "datastax-java-driver.advanced.connection.init-query-timeout": 5000,
        "maxConcurrentRequests": 500,
        "maxNumberOfRecordsInBatch": 32,
        "queryExecutionTimeout": 30,
        "connectionPoolLocalSize": 4,
        "topic.weather-data.weather.data_by_state.mapping": "station_id=value.stationid, temp=value.temp, state=value.state, ts=value.created",
        "topic.weather-data.weather.data_by_station.mapping": "station_id=value.stationid, temp=value.temp, state=value.state, ts=value.created",
        "key.converter": "org.apache.kafka.connect.storage.StringConverter",
        "value.converter": "org.apache.kafka.connect.json.JsonConverter",
        "value.converter.schemas.enable": false,
        "offset.flush.interval.ms": 10000
    }
}
```

Ниже приведена сводка атрибутов.

**Основные возможности подключения**

- `contactPoints`: введите контактную точку для Cosmos DB Cassandra
- `loadBalancing.localDc`: введите регион для Cosmos DB учетной записи, например Юго-Восточная Азия.
- `auth.username`: введите имя пользователя.
- `auth.password`: введите пароль.
- `port`: введите значение порта (это не так `10350` `9042` . Оставьте это как есть)

**Конфигурация SSL**

Azure Cosmos DB обеспечивает [безопасное подключение через SSL](database-security.md) и соединитель Kafka Connect также поддерживает SSL.

- `ssl.keystore.path`: путь к хранилищу ключей JDK в контейнере. `/etc/alternatives/jre/lib/security/cacerts/`
- `ssl.keystore.password`— Пароль JDK для хранилища ключей (по умолчанию)
- `ssl.hostnameValidation`: Мы переключим проверку имени узла
- `ssl.provider`: `JDK` используется как поставщик SSL

**Универсальные параметры**

- `key.converter`: Используется преобразователь строк `org.apache.kafka.connect.storage.StringConverter`
- `value.converter`. Поскольку данные в Kafka разделены на JSON, мы используем `org.apache.kafka.connect.json.JsonConverter`
- `value.converter.schemas.enable`. Поскольку полезные данные JSON не имеют связанной со схемой схемы (в целях демонстрационного приложения), необходимо указать Kafka Connect, чтобы не искать схему, задав для этого атрибута значение `false` . Если этого не сделать, это приведет к сбоям.

### <a name="install-the-connector"></a>Установка соединителя

Установите соединитель с помощью конечной точки Kafka Connect RESTFUL.

```shell
curl -X POST -H "Content-Type: application/json" --data @cassandra-sink-config.json http://localhost:8083/connectors
```

Чтобы проверить состояние:

```
curl http://localhost:8080/connectors/kafka-cosmosdb-sink/status
```

Если все работает правильно, соединитель должен начать плетение своего волшебства. Он должен пройти проверку подлинности в Azure Cosmos DB и начать прием данных из раздела Kafka ( `weather-data` ) в таблицы Cassandra. `weather.data_by_state``weather.data_by_station`

Теперь можно запрашивать данные в таблицах. Перейдите к портал Azure, чтобы открыть размещенную оболочку CQL для учетной записи Azure Cosmos DB.

:::image type="content" source="./media/cassandra-kafka-connect/cqlsh.png" alt-text="Открытие CQLSH":::

## <a name="query-data-from-azure-cosmos-db"></a>Запрос данных из Azure Cosmos DB

Проверьте `data_by_state` таблицы и `data_by_station` . Вот несколько примеров запросов, которые помогут вам приступить к работе:

```sql
select * from weather.data_by_state where state = 'state-1';
select * from weather.data_by_state where state IN ('state-1', 'state-2');
select * from weather.data_by_state where state = 'state-3' and ts > toTimeStamp('2020-11-26');

select * from weather.data_by_station where station_id = 'station-1';
select * from weather.data_by_station where station_id IN ('station-1', 'station-2');
select * from weather.data_by_station where station_id IN ('station-2', 'station-3') and ts > toTimeStamp('2020-11-26');
```

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Дальнейшие действия

* [Обеспечение необходимой пропускной способности для контейнеров и баз данных](set-throughput.md) 
* [Рекомендации по использованию ключа секции](partitioning-overview.md#choose-partitionkey)
* [Оценка единиц запросов в секунду с помощью планировщика ресурсов Azure Cosmos DB](estimate-ru-with-capacity-planner.md)
