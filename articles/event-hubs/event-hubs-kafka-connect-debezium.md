---
title: Интеграция Apache Kafka подключение к концентраторам событий Azure с помощью Дебезиум для отслеживания измененных данных
description: В этой статье содержатся сведения об использовании Дебезиум с концентраторами событий Azure для Kafka.
ms.topic: how-to
author: abhirockzz
ms.author: abhishgu
ms.date: 01/06/2021
ms.openlocfilehash: 0ad1df23e71e652f7d380ffbabb542b81954e038
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97935178"
---
# <a name="integrate-apache-kafka-connect-support-on-azure-event-hubs-with-debezium-for-change-data-capture"></a>Интеграция поддержки Apache Kafka Connect в концентраторы событий Azure с Дебезиум для отслеживания измененных данных

Система отслеживания **измененных данных (CDC)** — это метод, используемый для отслеживания изменений на уровне строк в таблицах базы данных в ответ на операции создания, обновления и удаления. [Дебезиум](https://debezium.io/) — это распределенная платформа, построенная на основе функций системы отслеживания измененных данных, доступных в разных базах данных (например, [логическое декодирование в PostgreSQL](https://www.postgresql.org/docs/current/static/logicaldecoding-explanation.html)). Он предоставляет набор [соединителей соединения Kafka](https://debezium.io/documentation/reference/1.2/connectors/index.html) , которые отключают изменения на уровне строк в таблицах базы данных и преобразуют их в потоки событий, которые затем отправляются в [Apache Kafka](https://kafka.apache.org/).

> [!WARNING]
> Использование платформы Apache Kafka Connect, а также платформы Дебезиум и ее соединителей **не подходит для поддержки программного продукта с помощью Microsoft Azure**.
>
> Apache Kafka Connect предполагает, что его динамическая конфигурация должна храниться в сжатых разделах с неограниченным сроком хранения. Концентраторы событий Azure [не реализуют сжатие как функцию брокера](event-hubs-federation-overview.md#log-projections) и всегда накладывают ограничение на хранение на основе времени для сохраненных событий, от принципа, в котором концентраторы событий Azure являются механизмом потоковой передачи событий в реальном времени, а не долговременным хранилищем данных или конфигураций.
>
> В то время как проект Apache Kafka может быть удобным при смешении этих ролей, Azure считает, что такие сведения лучше управлять в надлежащей базе данных или хранилище конфигураций.
>
> Многие сценарии Apache Kafka подключения будут работать, но эти концептуальные различия между моделями хранения Apache Kafka и концентраторов событий Azure могут привести к тому, что некоторые конфигурации не будут работать должным образом. 

В этом руководстве описано, как настроить систему на основе системы отслеживания измененных данных в Azure с помощью [концентраторов событий Azure](./event-hubs-about.md?WT.mc_id=devto-blog-abhishgu) (для Kafka), [Azure DB для PostgreSQL](../postgresql/overview.md) и дебезиум. Он будет использовать [соединитель Дебезиум PostgreSQL](https://debezium.io/documentation/reference/1.2/connectors/postgresql.html) для потоковой передачи изменений базы данных из PostgreSQL в Kafka статьи в концентраторах событий Azure.

> [!NOTE]
> Эта статья содержит ссылки на термин *список разрешений*. Корпорация Майкрософт больше не использует его. Когда этот термин будет удален из программного обеспечения, мы удалим его из статьи.

При работе с этим руководством вы выполните следующие задачи:

> [!div class="checklist"]
> * Создание пространства имен Центров событий
> * Установка и Настройка базы данных Azure для PostgreSQL
> * Настройка и запуск Kafka Connect с помощью соединителя Дебезиум PostgreSQL
> * Проверка отслеживания измененных данных
> * Используемых Использование событий измененных данных с помощью `FileStreamSink` соединителя

## <a name="pre-requisites"></a>Предварительные требования
Для выполнения этого пошагового руководства потребуется:

- Подписка Azure. Если ее нет, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/).
- Linux/MacOS
- Выпуск Kafka (версии 1.1.1, Scala версии 2.11) доступен на сайте [kafka.apache.org](https://kafka.apache.org/downloads#1.1.1)
- Ознакомьтесь со статьей [Центры событий Azure для Apache Kafka (предварительная версия)](./event-hubs-for-kafka-ecosystem-overview.md).

## <a name="create-an-event-hubs-namespace"></a>Создание пространства имен Центров событий
Для отправки и получения данных из любой службы Центров событий требуется пространство имен Центров событий. Инструкции по созданию пространства имен и концентратора событий см. в разделе [Создание концентратора событий](event-hubs-create.md) . Получите строку подключения Центров событий и полное доменное имя (FQDN) для последующего использования. Инструкции см. в статье [Get an Event Hubs connection string](event-hubs-get-connection-string.md) (Получение строки подключения для Центров событий). 

## <a name="setup-and-configure-azure-database-for-postgresql"></a>Установка и Настройка базы данных Azure для PostgreSQL
[База данных Azure для PostgreSQL](../postgresql/overview.md) — это служба реляционной базы данных, основанная на версии сообщества ядра СУБД PostgreSQL с открытым исходным кодом, которая доступна в двух вариантах развертывания: одиночный сервер и масштабирование (Цитус). [Выполните эти инструкции](../postgresql/quickstart-create-server-database-portal.md) , чтобы создать сервер базы данных Azure для PostgreSQL с помощью портал Azure. 

## <a name="setup-and-run-kafka-connect"></a>Установка и запуск Kafka Connect
В этом разделе рассматриваются следующие темы:

- Установка соединителя дебезиум
- Настройка Kafka Connect для концентраторов событий
- Запуск кластера Kafka Connect с помощью соединителя Дебезиум

### <a name="download-and-setup-debezium-connector"></a>Скачайте и настройте соединитель Дебезиум.
Чтобы скачать и настроить соединитель, следуйте последним инструкциям в [документации по дебезиум](https://debezium.io/documentation/reference/1.2/connectors/postgresql.html#postgresql-deploying-a-connector) .

- Скачайте архив подключаемого модуля соединителя. Например, чтобы скачать версию `1.2.0` соединителя, используйте эту ссылку. https://repo1.maven.org/maven2/io/debezium/debezium-connector-postgres/1.2.0.Final/debezium-connector-postgres-1.2.0.Final-plugin.tar.gz
- Извлеките JAR-файлы и скопируйте их в [подключаемый модуль Kafka Connect. Path](https://kafka.apache.org/documentation/#connectconfigs).


### <a name="configure-kafka-connect-for-event-hubs"></a>Настройка Kafka Connect для Центров событий
Минимальная перенастройка необходима при перенаправлении пропускной способности Kafka в Центры событий.  Следующий пример `connect-distributed.properties` показывает, как настроить Connect для проверки подлинности и обмена данных с конечной точкой Kafka в Центрах событий:

```properties
bootstrap.servers={YOUR.EVENTHUBS.FQDN}:9093 # e.g. namespace.servicebus.windows.net:9093
group.id=connect-cluster-group

# connect internal topic names, auto-created if not exists
config.storage.topic=connect-cluster-configs
offset.storage.topic=connect-cluster-offsets
status.storage.topic=connect-cluster-status

# internal topic replication factors - auto 3x replication in Azure Storage
config.storage.replication.factor=1
offset.storage.replication.factor=1
status.storage.replication.factor=1

rest.advertised.host.name=connect
offset.flush.interval.ms=10000

key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
internal.key.converter=org.apache.kafka.connect.json.JsonConverter
internal.value.converter=org.apache.kafka.connect.json.JsonConverter

internal.key.converter.schemas.enable=false
internal.value.converter.schemas.enable=false

# required EH Kafka security settings
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{YOUR.EVENTHUBS.CONNECTION.STRING}";

producer.security.protocol=SASL_SSL
producer.sasl.mechanism=PLAIN
producer.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{YOUR.EVENTHUBS.CONNECTION.STRING}";

consumer.security.protocol=SASL_SSL
consumer.sasl.mechanism=PLAIN
consumer.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{YOUR.EVENTHUBS.CONNECTION.STRING}";

plugin.path={KAFKA.DIRECTORY}/libs # path to the libs directory within the Kafka release
```

> [!IMPORTANT]
> Замените `{YOUR.EVENTHUBS.CONNECTION.STRING}` строками подключения для вашего пространства имен Центров событий. Инструкции по получению строки подключения см. в статье [Получение строки подключения Центров событий](event-hubs-get-connection-string.md). Пример конфигурации см. здесь: `sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://mynamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXXXXXXXXXXXXXXX";`


### <a name="run-kafka-connect"></a>Выполнение Kafka Connect
На этом этапе рабочая роль Kafka Connect запускается локально в распределенном режиме с помощью Центров событий для поддержания состояния кластера.

1. Сохраните файл `connect-distributed.properties`, указанный выше, локально.  Не забудьте заменить все значения в фигурных скобках.
2. Перейдите к расположению выпуска Kafka на своем компьютере.
3. Запустите `./bin/connect-distributed.sh /PATH/TO/connect-distributed.properties` и дождитесь запуска кластера.

> [!NOTE]
> Kafka Connect использует API Kafka AdminClient для автоматического создания разделов с рекомендуемыми конфигурациями, включая сжатие. Быстрая проверка пространства имен на портале Azure показывает, что внутренние разделы рабочей роли Connect были созданы автоматически.
>
> Внутренние разделы Kafka Connect **должны использовать сжатия**.  Команда Центров событий не несет ответственности за исправление неправильных конфигураций в случае ненадлежащей настройки внутренних разделов Connect.

### <a name="configure-and-start-the-debezium-postgresql-source-connector"></a>Настройка и запуск соединителя Дебезиум PostgreSQL Source

Создайте файл конфигурации ( `pg-source-connector.json` ) для соединителя источника PostgreSQL. Замените значения в соответствии с экземпляром Azure PostgreSQL.

```json
{
    "name": "todo-connector",
    "config": {
        "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
        "database.hostname": "<replace with Azure PostgreSQL instance name>.postgres.database.azure.com",
        "database.port": "5432",
        "database.user": "<replace with database user name>",
        "database.password": "<replace with database password>",
        "database.dbname": "postgres",
        "database.server.name": "my-server",
        "plugin.name": "wal2json",
        "table.whitelist": "public.todos"
    }
}
```

> [!TIP]
> `database.server.name` атрибут — это логическое имя, идентифицирующее и предоставляющее пространство имен для конкретного сервера базы данных PostgreSQL или наблюдаемого кластера. Подробные сведения см. в [документации по дебезиум](https://debezium.io/documentation/reference/1.2/connectors/postgresql.html#postgresql-property-database-server-name) .

Чтобы создать экземпляр соединителя, используйте конечную точку Kafka Connect REST API.

```bash
curl -X POST -H "Content-Type: application/json" --data @pg-source-connector.json http://localhost:8083/connectors
```

Чтобы проверить состояние соединителя, выполните следующие действия.

```bash
curl -s http://localhost:8083/connectors/todo-connector/status
```

## <a name="test-change-data-capture"></a>Проверка отслеживания измененных данных
Чтобы увидеть, как ведется отслеживание измененных данных в действии, необходимо создать, обновить или удалить записи в базе данных Azure PostgreSQL.

Начните с подключения к базе данных Azure PostgreSQL (в примере ниже используется [psql](https://www.postgresql.org/docs/12/app-psql.html))

```bash
psql -h <POSTGRES_INSTANCE_NAME>.postgres.database.azure.com -p 5432 -U <POSTGRES_USER_NAME> -W -d <POSTGRES_DB_NAME> --set=sslmode=require

e.g. 

psql -h my-postgres.postgres.database.azure.com -p 5432 -U testuser@my-postgres -W -d postgres --set=sslmode=require
```

**Создание таблицы и вставка записей**

```sql
CREATE TABLE todos (id SERIAL, description VARCHAR(50), todo_status VARCHAR(10), PRIMARY KEY(id));

INSERT INTO todos (description, todo_status) VALUES ('setup postgresql on azure', 'complete');
INSERT INTO todos (description, todo_status) VALUES ('setup kafka connect', 'complete');
INSERT INTO todos (description, todo_status) VALUES ('configure and install connector', 'in-progress');
INSERT INTO todos (description, todo_status) VALUES ('start connector', 'pending');
```

Соединитель должен переключиться в действие и отправить события измененных данных в концентраторы событий со следующей таблицей Na, e `my-server.public.todos` , если в `my-server` качестве значения для параметра `database.server.name` и `public.todos` указана таблица, изменения которой отслеживаются (согласно `table.whitelist` конфигурации).

**Раздел "Проверка концентраторов событий"**

Давайте интроспект содержимое раздела, чтобы убедиться, что все работает правильно. В приведенном ниже примере используется [`kafkacat`](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/kafkacat) , но вы также можете [создать потребитель, используя любой из перечисленных здесь параметров](apache-kafka-developer-guide.md) .

Создайте файл с именем `kafkacat.conf` со следующим содержимым:

```
metadata.broker.list=<enter event hubs namespace>.servicebus.windows.net:9093
security.protocol=SASL_SSL
sasl.mechanisms=PLAIN
sasl.username=$ConnectionString
sasl.password=<enter event hubs connection string>
```

> [!NOTE]
> Обновление `metadata.broker.list` и `sasl.password` атрибуты в `kafkacat.conf` как на сведения о концентраторах событий. 

В другом терминале запустите потребитель:

```bash
export KAFKACAT_CONFIG=kafkacat.conf
export BROKER=<enter event hubs namespace>.servicebus.windows.net:9093
export TOPIC=my-server.public.todos

kafkacat -b $BROKER -t $TOPIC -o beginning
```

Вы должны увидеть полезные данные JSON, представляющие события измененных данных, созданные в PostgreSQL, в ответ на строки, только что добавленные в `todos` таблицу. Ниже приведен фрагмент полезных данных:


```json
{
    "schema": {...},
    "payload": {
        "before": null,
        "after": {
            "id": 1,
            "description": "setup postgresql on azure",
            "todo_status": "complete"
        },
        "source": {
            "version": "1.2.0.Final",
            "connector": "postgresql",
            "name": "fullfillment",
            "ts_ms": 1593018069944,
            "snapshot": "last",
            "db": "postgres",
            "schema": "public",
            "table": "todos",
            "txId": 602,
            "lsn": 184579736,
            "xmin": null
        },
        "op": "c",
        "ts_ms": 1593018069947,
        "transaction": null
    }
```

Событие состоит из `payload` вместе с его `schema` (опущено для краткости). В `payload` разделе Обратите внимание на то, как представляется операция создания ( `"op": "c"` ) — `"before": null` означает, что она была вновь созданной `INSERT` строкой, `after` предоставляет значения для столбцов в строке, `source` предоставляет метаданные экземпляра PostgreSQL, откуда это событие было выбрано и т. д.

Вы можете попробовать то же самое с операциями Update или DELETE, а также интроспект события изменения данных. Например, чтобы обновить состояние задачи для `configure and install connector` (при условии, что `id` оно имеет значение `3` ):

```sql
UPDATE todos SET todo_status = 'complete' WHERE id = 3;
```

## <a name="optional-install-filestreamsink-connector"></a>Используемых Установка соединителя Филестреамсинк
Теперь, когда все `todos` изменения таблицы фиксируются в концентраторах событий, мы будем использовать соединитель филестреамсинк (который доступен по умолчанию в Kafka Connect) для использования этих событий.

Создание файла конфигурации ( `file-sink-connector.json` ) для соединителя — замена `file` атрибута в соответствии с файловой системой.

```json
{
    "name": "cdc-file-sink",
    "config": {
        "connector.class": "org.apache.kafka.connect.file.FileStreamSinkConnector",
        "tasks.max": "1",
        "topics": "my-server.public.todos",
        "file": "<enter full path to file e.g. /Users/foo/todos-cdc.txt>"
    }
}
```

Чтобы создать соединитель и проверить его состояние, сделайте следующее:

```bash
curl -X POST -H "Content-Type: application/json" --data @file-sink-connector.json http://localhost:8083/connectors

curl http://localhost:8083/connectors/cdc-file-sink/status
```

Вставка, обновление и удаление записей базы данных и отслеживание записей в настроенном файле приемника вывода:

```bash
tail -f /Users/foo/todos-cdc.txt
```


## <a name="cleanup"></a>Очистка
Kafka Connect создает разделы Центра событий для хранения конфигураций, смещений и состояния, которые сохраняются даже после завершения работы кластера Connect. Если этого не требуется, рекомендуется удалить эти разделы. Также может потребоваться удалить `my-server.public.todos` концентратор событий, созданный в ходе этого руководства.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о концентраторах событий для Kafka см. в следующих статьях:  

- [Зеркальное отображение брокера Kafka в концентраторе событий](event-hubs-kafka-mirror-maker-tutorial.md)
- [Подключение Apache Spark к концентратору событий](event-hubs-kafka-spark-tutorial.md)
- [Подключение Apache Flink к концентратору событий](event-hubs-kafka-flink-tutorial.md)
- [Migrating to Azure Event Hubs for Apache Kafka Ecosystems](https://github.com/Azure/azure-event-hubs-for-kafka) (Переход в Центры событий Azure для экосистем Apache Kafka)
- [Подключение Akka Streams к концентратору событий](event-hubs-kafka-akka-streams-tutorial.md)
- [Apache Kafka Guide для разработчиков концентраторов событий Azure](apache-kafka-developer-guide.md)