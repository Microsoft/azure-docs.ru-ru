---
title: Использование Apache Kafka MirrorMaker в службе "Центры событий Azure" | Документация Майкрософт
description: В этой статье описано, как с помощью Kafka MirrorMaker настроить зеркальное отображение кластера Kafka в службе "Центры событий Azure".
ms.topic: how-to
ms.date: 01/04/2021
ms.openlocfilehash: 654e9e19dfde0d0c58d00e41cf8ab0ba8e1484d7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97860997"
---
# <a name="use-apache-kafka-mirrormaker-with-event-hubs"></a>Использование Apache Kafka MirrorMaker с концентраторами событий

В этом руководстве показано, как создать зеркальную копию брокера Kafka в концентратор событий Azure с помощью Kafka MirrorMaker. Если вы размещаете Apache Kafka в Kubernetes с помощью оператора КНКФ Стримзи, вы можете ознакомиться с руководством в [этой записи блога](https://strimzi.io/blog/2020/06/09/mirror-maker-2-eventhub/) , чтобы узнать, как настроить Kafka с помощью Стримзи и Mirror Maker 2. 

   ![Kafka MirrorMaker и Центры событий](./media/event-hubs-kafka-mirror-maker-tutorial/evnent-hubs-mirror-maker1.png)

> [!NOTE]
> Этот пример можно найти на сайте [GitHub](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/mirror-maker).

> [!NOTE]
> Эта статья содержит ссылки на термин *список разрешений*. Корпорация Майкрософт больше не использует его. Когда этот термин будет удален из программного обеспечения, мы удалим его из статьи.

В этом руководстве описано следующее:
> [!div class="checklist"]
> * Создание пространства имен в Центрах событий
> * Клонирование примера проекта
> * Настройка кластера Kafka
> * Настройка Kafka MirrorMaker
> * Запуск Kafka MirrorMaker

## <a name="introduction"></a>Введение
В этом руководстве показано, как концентратор событий и Kafka MirrorMaker могут интегрировать существующий конвейер Kafka в Azure, выполнив "зеркальное отображение" входного потока Kafka в службе концентраторов событий, что позволяет интегрировать Apache Kafka потоки с помощью нескольких [шаблонов Федерации](event-hubs-federation-overview.md). 

Конечная точка Kafka Центров событий Azure позволяет подключаться к Центрам событий Azure с помощью протокола Kafka (т. е. клиентов Kafka). После внесения минимальных изменений в приложение Kafka вы сможете подключаться к Центрам событий Azure и пользоваться преимуществами экосистемы Azure. Концентраторы событий в настоящее время поддерживают протокол Apache Kafka версии 1,0 и более поздних версий.

Вы можете использовать Apache Kafka MirrorMaker 1 однонаправленно от Apache Kafka к концентраторам событий. MirrorMaker 2 можно использовать в обоих направлениях, но [ `MirrorCheckpointConnector` и `MirrorHeartbeatConnector` , которые настраиваются в MirrorMaker 2](https://cwiki.apache.org/confluence/display/KAFKA/KIP-382%3A+MirrorMaker+2.0) , должны быть настроены так, чтобы они указывали на Apache Kafka Broker, а не в концентраторы событий. В этом руководстве показано, как настроить MirrorMaker 1.

## <a name="prerequisites"></a>Предварительные требования

В рамках этого руководства вам потребуются:

* Прочтите статью [Центры событий Azure для Apache Kafka](event-hubs-for-kafka-ecosystem-overview.md). 
* Подписка Azure. Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio), прежде чем начать работу.
* [Пакет Java Development Kit (JDK) 1.7 +](/azure/developer/java/fundamentals/java-jdk-long-term-support)
    * В Ubuntu выполните команду `apt-get install default-jdk`, чтобы установить JDK.
    * Обязательно настройте переменную среды JAVA_HOME так, чтобы она указывала на папку, в которой установлен пакет JDK.
* [Скачивание](https://maven.apache.org/download.cgi) и [Установка](https://maven.apache.org/install.html) двоичного архива Maven
    * В Ubuntu выполните команду `apt-get install maven`, чтобы установить Maven.
* [Git](https://www.git-scm.com/downloads);
    * В Ubuntu выполните команду `sudo apt-get install git`, чтобы установить Git.

## <a name="create-an-event-hubs-namespace"></a>Создание пространства имен Центров событий

Для отправки и получения данных из любой службы Центров событий требуется пространство имен Центров событий. Инструкции по созданию пространства имен и концентратора событий см. в разделе [Создание концентратора событий](event-hubs-create.md) . Скопируйте строку подключения к Центрам событий для дальнейшего использования.

## <a name="clone-the-example-project"></a>Клонирование примера проекта

Теперь, когда у вас есть строка подключения концентраторов событий, выполните Клонирование репозитория концентраторов событий Azure для Kafka и перейдите к `mirror-maker` вложенной папке:

```shell
git clone https://github.com/Azure/azure-event-hubs-for-kafka.git
cd azure-event-hubs-for-kafka/tutorials/mirror-maker
```

## <a name="set-up-a-kafka-cluster"></a>Настройка кластера Kafka

Инструкции по настройке кластера с требуемыми параметрами (или использовании существующего кластера Kafka) см. в [кратком руководстве по Kafka](https://kafka.apache.org/quickstart).

## <a name="configure-kafka-mirrormaker"></a>Настройка Kafka MirrorMaker

Средство Kafka MirrorMaker позволяет выполнять зеркальное отображение потока. При наличии исходного и конечного кластеров Kafka средство MirrorMaker гарантирует, что любые сообщения, отправляемые в исходный кластер, поступают в исходный и конечный кластеры. В этом примере показано, как зеркально отобразить исходный кластер Kafka с целевым концентратором событий. Этот сценарий можно использовать для отправки данных из существующего конвейера Kafka в Центры событий без прерывания потока данных. 

Более подробные сведения о средстве Kafka MirrorMaker см. в [руководстве по зеркальному отображению Kafka и средству MirrorMaker](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=27846330).

Чтобы настроить Kafka MirrorMaker, назначьте ему кластер Kafka в качестве потребителя, источника и концентратора событий в качестве его производителя или назначения.

#### <a name="consumer-configuration"></a>Конфигурация потребителя

Обновите файл конфигурации потребителя `source-kafka.config`, из которого средство MirrorMaker получает свойства исходного кластера Kafka.

##### <a name="source-kafkaconfig"></a>source-kafka.config

```
bootstrap.servers={SOURCE.KAFKA.IP.ADDRESS1}:{SOURCE.KAFKA.PORT1},{SOURCE.KAFKA.IP.ADDRESS2}:{SOURCE.KAFKA.PORT2},etc
group.id=example-mirrormaker-group
exclude.internal.topics=true
client.id=mirror_maker_consumer
```

#### <a name="producer-configuration"></a>Конфигурация производителя

Теперь обновите файл конфигурации производителя `mirror-eventhub.config`, который указывает средству MirrorMaker отправлять повторяющиеся (или зеркальные) данные в службу Центров событий. В частности, измените `bootstrap.servers` и `sasl.jaas.config` так, чтобы они указывали на конечную точку Kafka Центров событий. Для работы службы Центров событий требуется безопасное подключение (SASL), которое обеспечивается путем установки последних трех свойств в следующей конфигурации: 

##### <a name="mirror-eventhubconfig"></a>mirror-eventhub.config

```
bootstrap.servers={YOUR.EVENTHUBS.FQDN}:9093
client.id=mirror_maker_producer

#Required for Event Hubs
sasl.mechanism=PLAIN
security.protocol=SASL_SSL
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{YOUR.EVENTHUBS.CONNECTION.STRING}";
```

> [!IMPORTANT]
> Замените `{YOUR.EVENTHUBS.CONNECTION.STRING}` строками подключения для вашего пространства имен Центров событий. Инструкции по получению строки подключения см. в статье [Получение строки подключения Центров событий](event-hubs-get-connection-string.md). Пример конфигурации см. здесь: `sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://mynamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXXXXXXXXXXXXXXX";`

## <a name="run-kafka-mirrormaker"></a>Запуск Kafka MirrorMaker

Запустите сценарий Kafka MirrorMaker из корневого каталога Kafka с помощью только что обновленных файлов конфигурации. Скопируйте файлы конфигурации в корневой каталог Kafka или обновите пути к ним в следующей команде.

```shell
bin/kafka-mirror-maker.sh --consumer.config source-kafka.config --num.streams 1 --producer.config mirror-eventhub.config --whitelist=".*"
```

Чтобы убедиться, что события достигли концентратора событий, см. статистику входящих данных в [портал Azure](https://azure.microsoft.com/features/azure-portal/)или запустите потребитель в концентраторе событий.

При выполнении MirrorMaker все события, отправляемые в исходный кластер Kafka, получаются как кластером Kafka, так и зеркальным концентратором событий. С помощью MirrorMaker и конечной точки Kafka Центров событий можно перенести существующий конвейер Kafka в управляемую службу Центров событий Azure, не изменяя существующий кластер и не прерывая текущий поток данных.

## <a name="samples"></a>Примеры
См. следующие примеры на сайте GitHub:

- [Пример кода для этого руководства на GitHub](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/mirror-maker)
- [Службы концентраторов событий Azure Kafka MirrorMaker, выполняющиеся на экземпляре контейнера Azure](https://github.com/djrosanova/EventHubsMirrorMaker)

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о концентраторах событий для Kafka см. в следующих статьях:  

- [Подключение Apache Spark к концентратору событий](event-hubs-kafka-spark-tutorial.md)
- [Подключение Apache Flink к концентратору событий](event-hubs-kafka-flink-tutorial.md)
- [Интеграция Kafka Connect с концентратором событий](event-hubs-kafka-connect-tutorial.md)
- [Migrating to Azure Event Hubs for Apache Kafka Ecosystems](https://github.com/Azure/azure-event-hubs-for-kafka) (Переход в Центры событий Azure для экосистем Apache Kafka)
- [Подключение Akka Streams к концентратору событий](event-hubs-kafka-akka-streams-tutorial.md)
- [Apache Kafka Guide для разработчиков концентраторов событий Azure](apache-kafka-developer-guide.md)