---
title: Использование Akka Streams для Apache Kafka с Центрами событий Azure | Документация Майкрософт
description: В этой статье содержатся сведения о подключении потоков Akka Streams к концентратору событий Azure.
ms.topic: how-to
ms.date: 06/23/2020
ms.openlocfilehash: 92ab927189329493696c70b61ffc7f11cad22a66
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92369579"
---
# <a name="using-akka-streams-with-event-hubs-for-apache-kafka"></a>Использование Akka Streams с Центрами событий для Apache Kafka

В этом руководстве показано, как подключить потоки Akka Streams через поддержку концентраторов событий для Apache Kafka без изменения клиентов протокола или выполнения собственных кластеров. 

В этом руководстве описано следующее:
> [!div class="checklist"]
> * Создание пространства имен в Центрах событий
> * Клонирование примера проекта
> * Запуск производителя Akka Streams 
> * Запуск потребителя Akka Streams

> [!NOTE]
> Этот пример можно найти на сайте [GitHub](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/akka/java).

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством выполните следующие предварительные требования:

* Прочтите статью [Центры событий Azure для Apache Kafka](event-hubs-for-kafka-ecosystem-overview.md). 
* Подписка Azure. Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio), прежде чем начать работу.
* [Комплект разработчика Java (JDK) 1.8+](/azure/developer/java/fundamentals/java-jdk-long-term-support).
    * В Ubuntu выполните команду `apt-get install default-jdk`, чтобы установить JDK.
    * Обязательно настройте переменную среды JAVA_HOME так, чтобы она указывала на папку, в которой установлен пакет JDK.
* [Скачивание](https://maven.apache.org/download.cgi) и [Установка](https://maven.apache.org/install.html) двоичного архива Maven
    * В Ubuntu выполните команду `apt-get install maven`, чтобы установить Maven.
* [Git](https://www.git-scm.com/downloads);
    * В Ubuntu выполните команду `sudo apt-get install git`, чтобы установить Git.

## <a name="create-an-event-hubs-namespace"></a>Создание пространства имен Центров событий

Для отправки и получения данных из любой службы концентраторов событий требуется пространство имен концентраторов событий. Подробные сведения см. [в разделе Создание концентратора событий](event-hubs-create.md) . Скопируйте строку подключения к Центрам событий для дальнейшего использования.

## <a name="clone-the-example-project"></a>Клонирование примера проекта

Теперь, когда у вас есть строка подключения концентраторов событий, выполните Клонирование репозитория концентраторов событий Azure для Kafka и перейдите к `akka` вложенной папке:

```shell
git clone https://github.com/Azure/azure-event-hubs-for-kafka.git
cd azure-event-hubs-for-kafka/tutorials/akka/java
```

## <a name="run-akka-streams-producer"></a>Запуск производителя Akka Streams

Используя предоставленный пример производителя Akka Streams, отправьте сообщения в службу Центров событий.

### <a name="provide-an-event-hubs-kafka-endpoint"></a>Предоставление конечной точки Kafka в Центрах событий

#### <a name="producer-applicationconf"></a>application.conf производителя

Обновите значения `bootstrap.servers` и `sasl.jaas.config` в `producer/src/main/resources/application.conf`, чтобы перенаправить производителя на конечную точку Kafka Центров событий с правильной аутентификацией.

```xml
akka.kafka.producer {
    #Akka Kafka producer properties can be defined here


    # Properties defined by org.apache.kafka.clients.producer.ProducerConfig
    # can be defined in this configuration section.
    kafka-clients {
        bootstrap.servers="{YOUR.EVENTHUBS.FQDN}:9093"
        sasl.mechanism=PLAIN
        security.protocol=SASL_SSL
        sasl.jaas.config="org.apache.kafka.common.security.plain.PlainLoginModule required username=\"$ConnectionString\" password=\"{YOUR.EVENTHUBS.CONNECTION.STRING}\";"
    }
}
```

> [!IMPORTANT]
> Замените `{YOUR.EVENTHUBS.CONNECTION.STRING}` строками подключения для вашего пространства имен Центров событий. Инструкции по получению строки подключения см. в статье [Получение строки подключения Центров событий](event-hubs-get-connection-string.md). Пример конфигурации см. здесь: `sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://mynamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXXXXXXXXXXXXXXX";`


### <a name="run-producer-from-the-command-line"></a>Запуск производителя из командной строки

Чтобы запустить производитель из командной строки, создайте JAR-файл, а затем запустите его из Maven (или создайте JAR-файл с помощью Maven, затем запустите его в Java, добавив необходимые JAR-файлы Kafka в путь к классу):

```shell
mvn clean package
mvn exec:java -Dexec.mainClass="AkkaTestProducer"
```

Производитель начинает отправлять события в концентратор событий в разделе `test` и выводит события в stdout.

## <a name="run-akka-streams-consumer"></a>Запуск потребителя Akka Streams

Используя предоставленный пример потребителя, вы получаете сообщения из концентратора событий.

### <a name="provide-an-event-hubs-kafka-endpoint"></a>Предоставление конечной точки Kafka в Центрах событий

#### <a name="consumer-applicationconf"></a>application.conf потребителя

Обновите значения `bootstrap.servers` и `sasl.jaas.config` в `consumer/src/main/resources/application.conf`, чтобы перенаправить потребителя на конечную точку Kafka в Центрах событий с правильной аутентификацией.

```xml
akka.kafka.consumer {
    #Akka Kafka consumer properties defined here
    wakeup-timeout=60s

    # Properties defined by org.apache.kafka.clients.consumer.ConsumerConfig
    # defined in this configuration section.
    kafka-clients {
       request.timeout.ms=60000
       group.id=akka-example-consumer

       bootstrap.servers="{YOUR.EVENTHUBS.FQDN}:9093"
       sasl.mechanism=PLAIN
       security.protocol=SASL_SSL
       sasl.jaas.config="org.apache.kafka.common.security.plain.PlainLoginModule required username=\"$ConnectionString\" password=\"{YOUR.EVENTHUBS.CONNECTION.STRING}\";"
    }
}
```

> [!IMPORTANT]
> Замените `{YOUR.EVENTHUBS.CONNECTION.STRING}` строками подключения для вашего пространства имен Центров событий. Инструкции по получению строки подключения см. в статье [Получение строки подключения Центров событий](event-hubs-get-connection-string.md). Пример конфигурации см. здесь: `sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://mynamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXXXXXXXXXXXXXXX";`


### <a name="run-consumer-from-the-command-line"></a>Запуск потребителя из командной строки

Чтобы запустить потребителя из командной строки, создайте JAR-файл, а затем запустите его из Maven (или создайте JAR-файл с помощью Maven, затем запустите его в Java, добавив необходимые JAR-файлы Kafka в путь к классу):

```shell
mvn clean package
mvn exec:java -Dexec.mainClass="AkkaTestConsumer"
```

Если концентратор событий имеет события (например, если производитель также работает), потребитель начинает получать события из раздела `test` . 

Дополнительные сведения об Akka Streams см. в руководстве по [Akka Streams Kafka](https://doc.akka.io/docs/akka-stream-kafka/current/home.html).

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о концентраторах событий для Kafka см. в следующих статьях:  

- [Зеркальное отображение брокера Kafka в концентраторе событий](event-hubs-kafka-mirror-maker-tutorial.md)
- [Подключение Apache Spark к концентратору событий](event-hubs-kafka-spark-tutorial.md)
- [Подключение Apache Flink к концентратору событий](event-hubs-kafka-flink-tutorial.md)
- [Интеграция Kafka Connect с концентратором событий](event-hubs-kafka-connect-tutorial.md)
- [Migrating to Azure Event Hubs for Apache Kafka Ecosystems](https://github.com/Azure/azure-event-hubs-for-kafka) (Переход в Центры событий Azure для экосистем Apache Kafka)
- [Apache Kafka Guide для разработчиков концентраторов событий Azure](apache-kafka-developer-guide.md)