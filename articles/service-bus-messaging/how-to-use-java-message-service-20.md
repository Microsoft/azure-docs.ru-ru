---
title: Использование API службы сообщений Java 2,0 с помощью служебной шины Azure Premium
description: Как использовать службу сообщений Java (JMS) с служебной шиной Azure
ms.topic: article
ms.date: 07/17/2020
ms.custom: seo-java-july2019, seo-java-august2019, seo-java-september2019
ms.openlocfilehash: 894821444f74248b73578595df943cb3a0025360
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101698405"
---
# <a name="use-java-message-service-20-api-with-azure-service-bus-premium"></a>Использование API службы сообщений Java 2,0 с помощью служебной шины Azure Premium

В этой статье объясняется, как использовать популярный API **2,0 службы сообщений Java (JMS)** для взаимодействия с служебной шиной Azure по протоколу расширенного протокола очередей сообщений (AMQP 1,0).

> [!NOTE]
> Поддержка API службы сообщений Java (JMS) 2,0 доступна только на **уровне Premium служебной шины Azure**.
>

## <a name="pre-requisites"></a>Предварительные требования

### <a name="get-started-with-service-bus"></a>Приступая к работе со служебной шиной

В этом учебнике предполагается, что у вас уже есть пространство имен служебной шины. Если этого не сделать, то можно [создать пространство имен и очередь](service-bus-create-namespace-portal.md) с помощью [портал Azure](https://portal.azure.com). 

Дополнительные сведения о создании пространств имен и очередей служебной шины см. в разделе Начало [работы с очередями служебной шины с помощью портал Azure](service-bus-quickstart-portal.md).

### <a name="set-up-a-java-development-environment"></a>Настройка среды разработки Java

Для разработки приложений Java необходимо настроить соответствующую среду разработки. 
   * Установлен либо JDK (пакет средств разработки Java), либо JRE (среда выполнения Java).
   * JDK или JRE добавляется в путь сборки и соответствующие системные переменные.
   * Для использования JDK или JRE устанавливается Java IDE. Например, Eclipse или IntelliJ.

Чтобы узнать больше о том, как подготовить среду разработки для Java в Azure, используйте [это руководство](/azure/developer/java/fundamentals/).

## <a name="what-jms-features-are-supported"></a>Какие функции JMS поддерживаются?

[!INCLUDE [service-bus-jms-features-list](../../includes/service-bus-jms-feature-list.md)]

## <a name="downloading-the-java-message-service-jms-client-library"></a>Загрузка клиентской библиотеки службы сообщений Java (JMS)

Чтобы использовать все функции, доступные на уровне Premium служебной шины Azure, в путь сборки проекта необходимо добавить приведенную ниже библиотеку.

[Azure-servicebus-JMS](https://search.maven.org/artifact/com.microsoft.azure/azure-servicebus-jms)

> [!NOTE]
> Чтобы добавить [Azure-servicebus-JMS](https://search.maven.org/artifact/com.microsoft.azure/azure-servicebus-jms) в путь сборки, используйте предпочтительное средство управления зависимостями для вашего проекта, например [Maven](https://maven.apache.org/) или [Gradle](https://gradle.org/).
>

## <a name="coding-java-applications"></a>Создание приложений Java

После импорта зависимостей приложения Java могут быть написаны независимым образом с помощью поставщика JMS.

### <a name="connecting-to-azure-service-bus-using-jms"></a>Подключение к служебной шине Azure с помощью JMS

Для подключения к служебной шине Azure с помощью клиентов JMS необходимо, чтобы **строка подключения** , доступная в разделе "политики общего доступа", в [портал Azure](https://portal.azure.com) в **строке первичного подключения**.

1. Создайте экземпляр `ServiceBusJmsConnectionFactorySettings`

    ```java
    ServiceBusJmsConnectionFactorySettings connFactorySettings = new ServiceBusJmsConnectionFactorySettings();
    connFactorySettings.setConnectionIdleTimeoutMS(20000);
    ```
2. Создайте экземпляр `ServiceBusJmsConnectionFactory` с соответствующим `ServiceBusConnectionString` .

    ```java
    String ServiceBusConnectionString = "<SERVICE_BUS_CONNECTION_STRING_WITH_MANAGE_PERMISSIONS>";
    ConnectionFactory factory = new ServiceBusJmsConnectionFactory(ServiceBusConnectionString, connFactorySettings);
    ```

3. Используйте `ConnectionFactory` для создания, `Connection` а затем `Session` 

    ```java
    Connection connection = factory.createConnection();
    Session session = connection.createSession();
    ```
    или `JMSContext` (для клиентов JMS 2,0)

    ```java
    JMSContext jmsContext = factory.createContext();
    ```

    >[!IMPORTANT]
    > Несмотря на то, что аналогично именованию, сеанс JMS и служебная шина "сеанс" полностью независимы друг от друга.
    >
    > В JMS 1,1 сеанс — это важный Стандартный блок API, который позволяет создавать MessageProducer, MessageConsumer и само сообщение. Дополнительные сведения см. в [модели программирования JMS API](https://docs.oracle.com/javaee/6/tutorial/doc/bnceh.html) .
    >
    > В служебной шине [сеансы](message-sessions.md) — это клиентские конструкции служб и на стороне клиента, позволяющие выполнять обработку FIFO в очередях и подписках.
    >

### <a name="write-the-jms-application"></a>Написание приложения JMS

После `Session` `JMSContext` создания экземпляра или, приложение может использовать знакомые API-интерфейсы JMS для выполнения операций управления и работы с данными.

Сведения о поддерживаемых интерфейсах API см. в списке [поддерживаемых функций JMS](how-to-use-java-message-service-20.md#what-jms-features-are-supported) .

Ниже приведены примеры фрагментов кода, которые помогут приступить к работе с JMS-

#### <a name="sending-messages-to-a-queue-and-topic"></a>Отправка сообщений в очередь и раздел

```java
// Create the queue and topic
Queue queue = jmsContext.createQueue("basicQueue");
Topic topic = jmsContext.createTopic("basicTopic");
// Create the message
Message msg = jmsContext.createMessage();

// Create the JMS message producer
JMSProducer producer = jmsContext.createProducer();

// send the message to the queue
producer.send(queue, msg);
// send the message to the topic
producer.send(topic, msg);
```

#### <a name="receiving-messages-from-a-queue"></a>Получение сообщений из очереди

```java
// Create the queue
Queue queue = jmsContext.createQueue("basicQueue");

// Create the message consumer
JMSConsumer consumer = jmsContext.createConsumer(queue);

// Receive the message
Message msg = (Message) consumer.receive();
```

#### <a name="receiving-messages-from-a-shared-durable-subscription-on-a-topic"></a>Получение сообщений из общей устойчивой подписки на раздел

```java
// Create the topic
Topic topic = jmsContext.createTopic("basicTopic");

// Create a shared durable subscriber on the topic
JMSConsumer sharedDurableConsumer = jmsContext.createSharedDurableConsumer(topic, "sharedDurableConsumer");

// Receive the message
Message msg = (Message) sharedDurableConsumer.receive();
```

## <a name="summary"></a>Итоги

В этом руководство показано, как клиентские приложения Java, использующие службу сообщений Java (JMS) через AMQP 1,0, могут взаимодействовать с служебной шиной Azure.

Протокол AMQP 1.0 Service Bus можно также использовать из других языков, в числе которых .NET, C, Python и PHP. Компоненты, созданные с помощью этих языков, могут надежно и точно обмениваться сообщениями, используя AMQP 1.0 в Service Bus.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о служебной шине Azure и сведения о сущностях службы сообщений Java (JMS) см. по ссылкам ниже. 
* [Служебная шина — очереди, разделы и подписки](service-bus-queues-topics-subscriptions.md)
* [Служебная шина — сущности службы сообщений Java](service-bus-queues-topics-subscriptions.md#java-message-service-jms-20-entities)
* [Поддержка AMQP 1.0 в служебной шине](service-bus-amqp-overview.md)
* [Инструкции для разработчиков служебной шины AMQP 1,0](service-bus-amqp-dotnet.md)
* [Начало работы с очередями служебной шины](service-bus-dotnet-get-started-with-queues.md)
* [API службы сообщений Java (внешний документ Oracle)](https://docs.oracle.com/javaee/7/api/javax/jms/package-summary.html)
* [Узнайте, как выполнить миграцию с Активемк на служебную шину.](migrate-jms-activemq-to-servicebus.md)