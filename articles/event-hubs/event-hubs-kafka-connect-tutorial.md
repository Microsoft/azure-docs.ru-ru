---
title: Интеграция Apache Kafka Connect с Центрами событий Azure | Документация Майкрософт
description: В этой статье содержатся сведения о том, как использовать Kafka Connect с концентраторами событий Azure для Kafka.
ms.topic: how-to
ms.date: 01/06/2021
ms.openlocfilehash: f82dcdafa7921f4a994361371536b2f1ace7cbc5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97935161"
---
# <a name="integrate-apache-kafka-connect-support-on-azure-event-hubs"></a>Интеграция поддержки Apache Kafka Connect в Центрах событий Azure
[Apache Kafka Connect](https://kafka.apache.org/documentation/#connect) — это платформа для подключения и импорта и экспорта данных из любой внешней системы, такой как MySQL, HDFS и файловая система, через кластер Kafka. В этом руководстве описано, как использовать Kafka Connect Framework с концентраторами событий.

> [!WARNING]
> Использование инфраструктуры Apache Kafka Connect и ее соединителей **не подходит для поддержки продуктов с помощью Microsoft Azure**.
>
> Apache Kafka Connect предполагает, что его динамическая конфигурация должна храниться в сжатых разделах с неограниченным сроком хранения. Концентраторы событий Azure [не реализуют сжатие как функцию брокера](event-hubs-federation-overview.md#log-projections) и всегда накладывают ограничение на хранение на основе времени для сохраненных событий, от принципа, в котором концентраторы событий Azure являются механизмом потоковой передачи событий в реальном времени, а не долговременным хранилищем данных или конфигураций.
>
> В то время как проект Apache Kafka может быть удобным при смешении этих ролей, Azure считает, что такие сведения лучше управлять в надлежащей базе данных или хранилище конфигураций.
>
> Многие сценарии Apache Kafka подключения будут работать, но эти концептуальные различия между моделями хранения Apache Kafka и концентраторов событий Azure могут привести к тому, что некоторые конфигурации не будут работать должным образом. 

В этом руководстве описывается интеграция Kafka Connect с концентратором событий и развертывание базовых соединителей Филестреамсаурце и Филестреамсинк. Эта функция в настоящее время находится на стадии предварительной версии. Хотя эти соединители не предназначены для использования в рабочей среде, они показывают комплексный сценарий Kafka Connect, в котором Центры событий Azure действуют в качестве брокера Kafka.

> [!NOTE]
> Этот пример доступен на сайте [GitHub](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/connect).

При работе с этим руководством вы выполните следующие задачи:

> [!div class="checklist"]
> * Создание пространства имен Центров событий
> * Клонирование примера проекта
> * Настройка Kafka Connect для Центров событий
> * Выполнение Kafka Connect
> * Создание соединителей

## <a name="prerequisites"></a>Предварительные условия
Для работы с этим пошаговым руководством выполните следующие предварительные требования:

- Подписка Azure. Если ее нет, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/).
- [Git](https://www.git-scm.com/downloads);
- Linux/MacOS
- Выпуск Kafka (версии 1.1.1, Scala версии 2.11) доступен на сайте [kafka.apache.org](https://kafka.apache.org/downloads#1.1.1)
- Ознакомьтесь со статьей [Центры событий Azure для Apache Kafka (предварительная версия)](./event-hubs-for-kafka-ecosystem-overview.md).

## <a name="create-an-event-hubs-namespace"></a>Создание пространства имен Центров событий
Для отправки и получения данных из любой службы Центров событий требуется пространство имен Центров событий. Инструкции по созданию пространства имен и концентратора событий см. в разделе [Создание концентратора событий](event-hubs-create.md) . Получите строку подключения Центров событий и полное доменное имя (FQDN) для последующего использования. Инструкции см. в статье [Get an Event Hubs connection string](event-hubs-get-connection-string.md) (Получение строки подключения для Центров событий). 

## <a name="clone-the-example-project"></a>Клонирование примера проекта
Клонируйте репозиторий Центров событий Azure и перейдите к руководствам или вложенной папке подключения: 

```
git clone https://github.com/Azure/azure-event-hubs-for-kafka.git
cd azure-event-hubs-for-kafka/tutorials/connect
```

## <a name="configure-kafka-connect-for-event-hubs"></a>Настройка Kafka Connect для Центров событий
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


## <a name="run-kafka-connect"></a>Выполнение Kafka Connect

На этом этапе рабочая роль Kafka Connect запускается локально в распределенном режиме с помощью Центров событий для поддержания состояния кластера.

1. Сохраните файл `connect-distributed.properties`, указанный выше, локально.  Не забудьте заменить все значения в фигурных скобках.
2. Перейдите к расположению выпуска Kafka на своем компьютере.
4. Выполните `./bin/connect-distributed.sh /PATH/TO/connect-distributed.properties`.  REST API рабочей роли Connect готов к взаимодействию при появлении `'INFO Finished starting connectors and tasks'`. 

> [!NOTE]
> Kafka Connect использует API Kafka AdminClient для автоматического создания разделов с рекомендуемыми конфигурациями, включая сжатие. Быстрая проверка пространства имен на портале Azure показывает, что внутренние разделы рабочей роли Connect были созданы автоматически.
>
>Внутренние разделы Kafka Connect **должны использовать сжатия**.  Команда Центров событий не несет ответственности за исправление неправильных конфигураций в случае ненадлежащей настройки внутренних разделов Connect.

### <a name="create-connectors"></a>Создание соединителей
В этом разделе показано использование соединителей FileStreamSource и FileStreamSink. 

1. Создайте каталог для файлов входных и выходных данных.
    ```bash
    mkdir ~/connect-quickstart
    ```

2. Создайте два файла. Один файл с начальными данными, которые читает соединитель FileStreamSource, и другой, в который соединитель FileStreamSink выполняет запись.
    ```bash
    seq 1000 > ~/connect-quickstart/input.txt
    touch ~/connect-quickstart/output.txt
    ```

3. Создайте соединитель FileStreamSource.  Обязательно замените фигурные скобки своим домашним каталогом.
    ```bash
    curl -s -X POST -H "Content-Type: application/json" --data '{"name": "file-source","config": {"connector.class":"org.apache.kafka.connect.file.FileStreamSourceConnector","tasks.max":"1","topic":"connect-quickstart","file": "{YOUR/HOME/PATH}/connect-quickstart/input.txt"}}' http://localhost:8083/connectors
    ```
    Вы должны увидеть `connect-quickstart` Центра событий в экземпляре Центров событий после выполнения предыдущей команды.
4. Проверьте состояние соединителя источника.
    ```bash
    curl -s http://localhost:8083/connectors/file-source/status
    ```
    При необходимости, чтобы убедиться, что события были получены из раздела `connect-quickstart`, можно использовать [Explorer служебной шины](https://github.com/paolosalvatori/ServiceBusExplorer/releases).

5. Создайте соединитель FileStreamSink.  Еще раз убедитесь, что вы заменили фигурные скобки своим путем к домашнему каталогу.
    ```bash
    curl -X POST -H "Content-Type: application/json" --data '{"name": "file-sink", "config": {"connector.class":"org.apache.kafka.connect.file.FileStreamSinkConnector", "tasks.max":"1", "topics":"connect-quickstart", "file": "{YOUR/HOME/PATH}/connect-quickstart/output.txt"}}' http://localhost:8083/connectors
    ```
 
6. Проверьте состояние соединителя приемника.
    ```bash
    curl -s http://localhost:8083/connectors/file-sink/status
    ```

7. Убедитесь, что данные реплицированы между файлами и что данные будут одинаковыми для обоих файлов.
    ```bash
    # read the file
    cat ~/connect-quickstart/output.txt
    # diff the input and output files
    diff ~/connect-quickstart/input.txt ~/connect-quickstart/output.txt
    ```

### <a name="cleanup"></a>Очистка
Kafka Connect создает разделы Центра событий для хранения конфигураций, смещений и состояния, которые сохраняются даже после завершения работы кластера Connect. Если этого не требуется, рекомендуется удалить эти разделы. Вы можете также удалить Центр событий `connect-quickstart`, который был создан при выполнении данного пошагового руководства.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о концентраторах событий для Kafka см. в следующих статьях:  

- [Зеркальное отображение брокера Kafka в концентраторе событий](event-hubs-kafka-mirror-maker-tutorial.md)
- [Подключение Apache Spark к концентратору событий](event-hubs-kafka-spark-tutorial.md)
- [Подключение Apache Flink к концентратору событий](event-hubs-kafka-flink-tutorial.md)
- [Migrating to Azure Event Hubs for Apache Kafka Ecosystems](https://github.com/Azure/azure-event-hubs-for-kafka) (Переход в Центры событий Azure для экосистем Apache Kafka)
- [Подключение Akka Streams к концентратору событий](event-hubs-kafka-akka-streams-tutorial.md)
- [Apache Kafka Guide для разработчиков концентраторов событий Azure](apache-kafka-developer-guide.md)
