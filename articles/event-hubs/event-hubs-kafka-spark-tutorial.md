---
title: Интеграция приложения Apache Spark с Центрами событий Azure | Документация Майкрософт
description: В этой статье рассказывается, как использовать приложение Apache Spark с поддержкой Kafka в Центрах событий Azure.
ms.topic: how-to
ms.date: 06/23/2020
ms.openlocfilehash: 84184ed3dffee97863b93c592d1cd577df313605
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92913744"
---
# <a name="connect-your-apache-spark-application-with-azure-event-hubs"></a>Подключение приложения Apache Spark к концентраторам событий Azure
В этом руководстве описывается подключение приложения Spark к концентраторам событий для потоковой передачи в режиме реального времени. Такая интеграция позволяет выполнять потоковую передачу без изменения клиентов протокола или запускать собственные кластеры Kafka или Zookeeper. Для работы с этим руководством требуется Apache Spark v 2.4 + и Apache Kafka v 2.0 +.

> [!NOTE]
> Этот пример можно найти на сайте [GitHub](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/spark/).

В этом руководстве описано следующее:
> [!div class="checklist"]
> * Создание пространства имен в Центрах событий
> * Клонирование примера проекта
> * Запуск Spark.
> * Чтение из Центров событий для Kafka.
> * Запись в Центры событий для Kafka.

## <a name="prerequisites"></a>Предварительные условия

Перед началом работы с данным руководством необходимо выполнить описанные ниже условия.
-   Подписка Azure. Если ее нет, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/).
-   [Apache Spark v2.4](https://spark.apache.org/downloads.html).
-   [Apache Kafka v2.0]( https://kafka.apache.org/20/documentation.html).
-   [Git](https://www.git-scm.com/downloads);

> [!NOTE]
> Адаптер Spark-Kafka был обновлен для поддержки Kafka v2.0, начиная со Spark v2.4. В предыдущих выпусках Spark адаптер поддерживал Kafka v0.10 и более поздние версии, но в основном как база использовались API Kafka v0.10. Так как Центры событий для Kafka не поддерживают Kafka v0.10, адаптеры Spark-Kafka от версий Spark до версии 2.4 не поддерживаются Центрами событий для экосистем Kafka.


## <a name="create-an-event-hubs-namespace"></a>Создание пространства имен Центров событий
Для отправки и получения данных из любой службы Центров событий требуется пространство имен Центров событий. Инструкции по созданию пространства имен и концентратора событий см. в разделе [Создание концентратора событий](event-hubs-create.md) . Получите строку подключения Центров событий и полное доменное имя (FQDN) для последующего использования. Инструкции см. в статье [Get an Event Hubs connection string](event-hubs-get-connection-string.md) (Получение строки подключения для Центров событий). 

## <a name="clone-the-example-project"></a>Клонирование примера проекта
Клонируйте репозиторий Центров событий Azure и перейдите к вложенной папке `tutorials/spark`:

```bash
git clone https://github.com/Azure/azure-event-hubs-for-kafka.git
cd azure-event-hubs-for-kafka/tutorials/spark
```

## <a name="read-from-event-hubs-for-kafka"></a>Чтение из Центров событий для Kafka.
Внеся несколько изменений в конфигурации, вы сможете запустить чтение из Центров событий для Kafka. Обновите **BOOTSTRAP_SERVERS** и **EH_SASL** сведениями о пространстве имен, чтобы запустить потоковую передачу с помощью Центров событий так же, как с Kafka. Чтобы ознакомиться с полным примером кода, просмотрите файл sparkConsumer.scala на сайте GitHub. 

```scala
//Read from your Event Hub!
val df = spark.readStream
    .format("kafka")
    .option("subscribe", TOPIC)
    .option("kafka.bootstrap.servers", BOOTSTRAP_SERVERS)
    .option("kafka.sasl.mechanism", "PLAIN")
    .option("kafka.security.protocol", "SASL_SSL")
    .option("kafka.sasl.jaas.config", EH_SASL)
    .option("kafka.request.timeout.ms", "60000")
    .option("kafka.session.timeout.ms", "30000")
    .option("kafka.group.id", GROUP_ID)
    .option("failOnDataLoss", "true")
    .load()

//Use dataframe like normal (in this example, write to console)
val df_write = df.writeStream
    .outputMode("append")
    .format("console")
    .start()
```

## <a name="write-to-event-hubs-for-kafka"></a>Запись в Центры событий для Kafka.
Вы также можете выполнить запись в Центры событий таким же образом, как и в Kafka. Обновите конфигурацию, чтобы заменить сведения в **BOOTSTRAP_SERVERS** и **EH_SASL** сведениями о пространстве имен в Центре событий.  Чтобы ознакомиться с полным примером кода, просмотрите файл sparkProducer.scala на сайте GitHub. 

```scala
df = /**Dataframe**/

//Write to your Event Hub!
df.writeStream
    .format("kafka")
    .option("topic", TOPIC)
    .option("kafka.bootstrap.servers", BOOTSTRAP_SERVERS)
    .option("kafka.sasl.mechanism", "PLAIN")
    .option("kafka.security.protocol", "SASL_SSL")
    .option("kafka.sasl.jaas.config", EH_SASL)
    .option("checkpointLocation", "./checkpoint")
    .start()
```



## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о Центрах событий и Центрах событий для Kafka см. в следующих статьях:  

- [Зеркальное отображение брокера Kafka в концентраторе событий](event-hubs-kafka-mirror-maker-tutorial.md)
- [Подключение Apache Flink к концентратору событий](event-hubs-kafka-flink-tutorial.md)
- [Интеграция Kafka Connect с концентратором событий](event-hubs-kafka-connect-tutorial.md)
- [Migrating to Azure Event Hubs for Apache Kafka Ecosystems](https://github.com/Azure/azure-event-hubs-for-kafka) (Переход в Центры событий Azure для экосистем Apache Kafka)
- [Подключение Akka Streams к концентратору событий](event-hubs-kafka-akka-streams-tutorial.md)
- [Apache Kafka Guide для разработчиков концентраторов событий Azure](apache-kafka-developer-guide.md)

