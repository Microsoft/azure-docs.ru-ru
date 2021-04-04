---
title: Краткое руководство. Потоковая передача данных с помощью Центров событий Azure и протокола Kafka
description: Краткое руководство. В этой статье объясняется, как реализовать потоковую передачу в службу "Центры событий Azure" с использованием протокола Kafka и API-интерфейсов.
ms.topic: quickstart
ms.date: 06/23/2020
ms.openlocfilehash: 2020534a3984453bcd6eff7ad0f5c02d9e7a29ff
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92368355"
---
# <a name="quickstart-data-streaming-with-event-hubs-using-the-kafka-protocol"></a>Краткое руководство. Потоковая передача данных с помощью Центров событий Azure и протокола Kafka
В этом кратком руководстве показано, как выполнять потоковую передачу данных в Центры событий без необходимости менять клиенты протоколов или запускать собственные кластеры. Вы узнаете, как обеспечить взаимодействие отправителей и объектов-получателей с Центрами событий, изменив конфигурацию в приложениях. 

> [!NOTE]
> Этот пример можно найти на сайте [GitHub](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/java).

## <a name="prerequisites"></a>Предварительные требования

Ниже указаны требования для работы с этим кратким руководством.

* Прочтите статью [Центры событий Azure для Apache Kafka](event-hubs-for-kafka-ecosystem-overview.md).
* Подписка Azure. Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio), прежде чем начать работу.
* [Комплект разработчика Java (JDK) 1.7+](/azure/developer/java/fundamentals/java-jdk-long-term-support).
* [Скачайте](https://maven.apache.org/download.cgi) и [установите](https://maven.apache.org/install.html) двоичный архив Maven.
* [Git](https://www.git-scm.com/);


## <a name="create-an-event-hubs-namespace"></a>Создание пространства имен в Центрах событий
Если вы создаете пространство имен в Центрах событий уровня **Стандартный**, конечная точка Kafka для пространства имен включается автоматически. Вы можете выполнять потоковую передачу событий из приложений, использующих протокол Kafka, в Центры событий уровня "Стандартный". Выполните пошаговые инструкции в статье [Краткое руководство. Создание концентратора событий с помощью портала Azure](event-hubs-create.md), чтобы создать пространство имен концентраторов событий уровня **Стандартный**. 

> [!NOTE]
> Центры событий Azure для Kafka доступны только на уровнях **Стандартный** и **Выделенный**. Уровень **Базовый** не поддерживает Kafka в Центрах событий.

## <a name="send-and-receive-messages-with-kafka-in-event-hubs"></a>Отправка и получение сообщений с использованием Kafka в Центрах событий

1. Клонируйте [репозиторий Центров событий Azure для Kafka](https://github.com/Azure/azure-event-hubs-for-kafka).

2. Перейдите на страницу `azure-event-hubs-for-kafka/quickstart/java/producer`.

3. Обновите сведения о конфигурации для отправителя в файле по адресу `src/main/resources/producer.config` следующим образом:

    **TLS/SSL:**

    ```xml
    bootstrap.servers=NAMESPACENAME.servicebus.windows.net:9093
    security.protocol=SASL_SSL
    sasl.mechanism=PLAIN
    sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{YOUR.EVENTHUBS.CONNECTION.STRING}";
    ```
    
    > [!IMPORTANT]
    > Замените `{YOUR.EVENTHUBS.CONNECTION.STRING}` строками подключения для вашего пространства имен Центров событий. Инструкции по получению строки подключения см. в статье [Получение строки подключения Центров событий](event-hubs-get-connection-string.md). Пример конфигурации см. здесь: `sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://mynamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXXXXXXXXXXXXXXX";`

    **OAuth:**

    ```xml
    bootstrap.servers=NAMESPACENAME.servicebus.windows.net:9093
    security.protocol=SASL_SSL
    sasl.mechanism=OAUTHBEARER
    sasl.jaas.config=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required;
    sasl.login.callback.handler.class=CustomAuthenticateCallbackHandler;
    ```    

    Исходный код для класса обработчика CustomAuthenticateCallbackHandler из примера см. [в этом репозитории на GitHub](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/oauth/java/appsecret/producer/src/main/java).
4. Выполните код отправителя и потоковую передачу событий в Центры событий.
   
    ```shell
    mvn clean package
    mvn exec:java -Dexec.mainClass="TestProducer"                                    
    ```
    
5. Перейдите на страницу `azure-event-hubs-for-kafka/quickstart/java/consumer`.

6. Обновите для объекта-получателя сведения о конфигурации в файле по адресу `src/main/resources/consumer.config` следующим образом.
   
    **TLS/SSL:**

    ```xml
    bootstrap.servers=NAMESPACENAME.servicebus.windows.net:9093
    security.protocol=SASL_SSL
    sasl.mechanism=PLAIN
    sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{YOUR.EVENTHUBS.CONNECTION.STRING}";
    ```

    > [!IMPORTANT]
    > Замените `{YOUR.EVENTHUBS.CONNECTION.STRING}` строками подключения для вашего пространства имен Центров событий. Инструкции по получению строки подключения см. в статье [Получение строки подключения Центров событий](event-hubs-get-connection-string.md). Пример конфигурации см. здесь: `sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://mynamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXXXXXXXXXXXXXXX";`

    **OAuth:**

    ```xml
    bootstrap.servers=NAMESPACENAME.servicebus.windows.net:9093
    security.protocol=SASL_SSL
    sasl.mechanism=OAUTHBEARER
    sasl.jaas.config=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required;
    sasl.login.callback.handler.class=CustomAuthenticateCallbackHandler;
    ``` 

    Исходный код для класса обработчика CustomAuthenticateCallbackHandler из примера см. [в этом репозитории на GitHub](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/oauth/java/appsecret/consumer/src/main/java).

    Все примеры с OAuth для Центров событий для Kafka [можно найти здесь](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/oauth).
7. Используя клиенты Kafka, запустите код объекта-получателя и выполните обработку из концентраторов событий.

    ```java
    mvn clean package
    mvn exec:java -Dexec.mainClass="TestConsumer"                                    
    ```

Если у кластера Центров событий с поддержкой Kafka появятся события, можно начать получать их от объекта-получателя.

## <a name="next-steps"></a>Дальнейшие действия
Из этой статьи вы узнали, как выполнять потоковую передачу данных в Центры событий без необходимости менять клиенты протоколов или запускать собственные кластеры. См. сведения в [руководстве для разработчиков Apache Kafka по Центрам событий Azure](apache-kafka-developer-guide.md).