---
title: Использование AMQP с API службы сообщений Java и служебной шиной Azure
description: Используйте службу сообщений Java (JMS) с служебной шиной Azure и Расширенный протокол управления очередью сообщений (AMQP) 1,0.
ms.topic: article
ms.date: 06/23/2020
ms.custom: seo-java-july2019, seo-java-august2019, seo-java-september2019, devx-track-java
ms.openlocfilehash: 97031abaedaa3e5595e290fa0292646feb744d47
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90086697"
---
# <a name="use-the-java-message-service-with-azure-service-bus-and-amqp-10"></a>Использование службы сообщений Java с служебной шиной Azure и AMQP 1,0

> [!WARNING]
> Эта статья посвящена *ограничению поддержки* API 1,1 службы сообщений Java (JMS) и существует только для уровня "Стандартный" служебной шины Azure.
>
> Полная поддержка API 2,0 службы сообщений Java доступна только на [уровне Premium служебной шины Azure в предварительной версии](how-to-use-java-message-service-20.md). Мы рекомендуем использовать этот уровень.
>

В этой статье объясняется, как использовать функции обмена сообщениями служебной шины из приложений Java с помощью популярного стандарта API JMS. Эти функции обмена сообщениями включают очереди и публикацию или подписку на разделы. В [сопровождающей статье](service-bus-amqp-dotnet.md) объясняется, как выполнять те же действия с помощью API .NET служебной шины Azure. Вы можете использовать эти две статьи вместе, чтобы узнать об обмене сообщениями между различными платформами с помощью Расширенный протокол управления очередью сообщений (AMQP) 1,0.

AMQP 1,0 — это эффективный и надежный протокол обмена сообщениями на уровне сети, который можно использовать для создания надежных, кросс-платформенных приложений обмена сообщениями.

Поддержка AMQP 1,0 в служебной шине означает, что вы можете использовать функции очереди и публикации или подписки через посредника из различных платформ с помощью эффективного двоичного протокола. Кроме того, можно создавать приложения, состоящие из компонентов, созданных с помощью различных языков, платформ и операционных систем.

## <a name="get-started-with-service-bus"></a>Приступая к работе со служебной шиной

В этой статье предполагается, что у вас уже есть пространство имен служебной шины, содержащее очередь с именем `basicqueue` . В противном случае можно [создать пространство имен и очередь](service-bus-create-namespace-portal.md) с помощью [портал Azure](https://portal.azure.com). Дополнительные сведения о создании пространства имен и очередей служебной шины см. в статье [Приступая к работе с очередями служебной шины](service-bus-dotnet-get-started-with-queues.md).

> [!NOTE]
> Секционированные очереди и разделы также поддерживают AMQP. Дополнительные сведения см. в статьях [Секционированные сущности обмена сообщениями](service-bus-partitioning.md) и [Поддержка AMQP 1.0 для секционированных очередей и разделов служебной шины](./service-bus-amqp-protocol-guide.md).
> 
> 

## <a name="download-the-amqp-10-jms-client-library"></a>Загрузка клиентской библиотеки AMQP 1,0 JMS

Сведения о том, где загрузить последнюю версию клиентской библиотеки Apache Qpid JMS AMQP 1,0, см. на [сайте загрузки Apache Qpid](https://qpid.apache.org/download.html).

При создании и запуске приложений JMS с помощью служебной шины необходимо добавить следующие JAR-файлы из архивного архива Apache Qpid JMS AMQP 1,0 в переменную среды подкаталога Java.

* geronimo-jms\_1.1\_spec-1.0.jar
* qpid-jms-client-[version].jar

> [!NOTE]
> Имена и версии JAR-файлов JMS могли измениться. Дополнительные сведения см. в статье [QPID JMS AMQP 1,0](https://qpid.apache.org/maven.html#qpid-jms-amqp-10).

## <a name="code-java-applications"></a>Код приложений Java

### <a name="java-naming-and-directory-interface"></a>Интерфейс именования и каталогов Java

JMS использует интерфейс JNDI для разделения логических и физических имен. Два типа объектов JMS разрешаются с помощью JNDI: **ConnectionFactory** и **Destination**. JNDI использует модель поставщика, к которой можно подключить различные службы каталогов для обработки заданий разрешения имен. В библиотеку Apache Qpid JMS AMQP 1,0 входит простой поставщик JNDI на основе файлов, который настраивается с помощью файла свойств в следующем формате:

```TEXT
# servicebus.properties - sample JNDI configuration

# Register a ConnectionFactory in JNDI using the form:
# connectionfactory.[jndi_name] = [ConnectionURL]
connectionfactory.SBCF = amqps://[SASPolicyName]:[SASPolicyKey]@[namespace].servicebus.windows.net

# Register some queues in JNDI using the form
# queue.[jndi_name] = [physical_name]
# topic.[jndi_name] = [physical_name]
queue.QUEUE = queue1
```

#### <a name="set-up-jndi-context-and-configure-the-connectionfactory-object"></a>Настройка контекста JNDI и настройка объекта ConnectionFactory

Строка подключения, на которую указывает ссылка, доступна в политиках общего доступа в [портал Azure](https://portal.azure.com) в разделе **Основная строка подключения**.

```java
// The connection string builder is the only part of the azure-servicebus SDK library
// we use in this JMS sample and for the purpose of robustly parsing the Service Bus 
// connection string. 
ConnectionStringBuilder csb = new ConnectionStringBuilder(connectionString);
        
// Set up JNDI context
Hashtable<String, String> hashtable = new Hashtable<>();
hashtable.put("connectionfactory.SBCF", "amqps://" + csb.getEndpoint().getHost() + "?amqp.idleTimeout=120000&amqp.traceFrames=true");
hashtable.put("queue.QUEUE", "BasicQueue");
hashtable.put(Context.INITIAL_CONTEXT_FACTORY, "org.apache.qpid.jms.jndi.JmsInitialContextFactory");
Context context = new InitialContext(hashtable);

ConnectionFactory cf = (ConnectionFactory) context.lookup("SBCF");

// Look up queue
Destination queue = (Destination) context.lookup("QUEUE");
```

#### <a name="configure-producer-and-consumer-destination-queues"></a>Настройка очередей назначения для производителя и потребителя

Запись, используемая для определения назначения в поставщике JNDI файла свойств Qpid, имеет следующий формат.

Чтобы создать очередь назначения для производителя, сделайте следующее:
```java
String queueName = "queueName";
Destination queue = (Destination) queueName;

ConnectionFactory cf = (ConnectionFactory) context.lookup("SBCF");
Connection connection - cf.createConnection(csb.getSasKeyName(), csb.getSasKey());

Session session = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);

// Create producer
MessageProducer producer = session.createProducer(queue);
```

Чтобы создать очередь назначения для потребителя, сделайте следующее:
```java
String queueName = "queueName";
Destination queue = (Destination) queueName;

ConnectionFactory cf = (ConnectionFactory) context.lookup("SBCF");
Connection connection - cf.createConnection(csb.getSasKeyName(), csb.getSasKey());

Session session = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);

// Create consumer
MessageConsumer consumer = session.createConsumer(queue);
```

### <a name="write-the-jms-application"></a>Написание приложения JMS

При использовании JMS с служебной шиной специальные API или параметры не требуются. Существует несколько ограничений, которые будут рассмотрены позже. Как и в любом приложении JMS, первое, что требуется, — это настройка среды JNDI для разрешения объекта **ConnectionFactory** и назначений.

#### <a name="configure-the-jndi-initialcontext-object"></a>Настройка объекта Инитиалконтекст JNDI

Среда JNDI настраивается путем передачи хэш-таблицы сведений о конфигурации в конструктор класса javax.naming.IniТиалконтекст. Два обязательных элемента в хэш-таблице — это имя класса начальной фабрики контекста и URL-адрес поставщика. В следующем примере кода показано, как настроить среду JNDI для использования свойства Qpid на основе файлового поставщика JNDI с файлом свойств с именем **servicebus. Properties**.

```java
// Set up JNDI context
Hashtable<String, String> hashtable = new Hashtable<>();
hashtable.put("connectionfactory.SBCF", "amqps://" + csb.getEndpoint().getHost() + \
"?amqp.idleTimeout=120000&amqp.traceFrames=true");
hashtable.put("queue.QUEUE", "BasicQueue");
hashtable.put(Context.INITIAL_CONTEXT_FACTORY, "org.apache.qpid.jms.jndi.JmsInitialContextFactory");
Context context = new InitialContext(hashtable);
``` 

### <a name="a-simple-jms-application-that-uses-a-service-bus-queue"></a>Простое приложение JMS, использующее очередь служебной шины

В следующем примере программа отправляет текстовые сообщения JMS в очередь служебной шины с логическим именем JNDI в очереди и получает их обратно.

Вы можете получить доступ ко всем исходным коду и сведениям о конфигурации из [краткого руководства по использованию очереди JMS образцов служебной шины Azure](https://github.com/Azure/azure-service-bus/tree/master/samples/Java/qpid-jms-client/JmsQueueQuickstart).

```java
// Copyright (c) Microsoft. All rights reserved.
// Licensed under the MIT license. See LICENSE file in the project root for full license information.

package com.microsoft.azure.servicebus.samples.jmsqueuequickstart;

import com.microsoft.azure.servicebus.primitives.ConnectionStringBuilder;
import org.apache.commons.cli.*;
import org.apache.log4j.*;

import javax.jms.*;
import javax.naming.Context;
import javax.naming.InitialContext;
import java.util.Hashtable;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.function.Function;

/**
 * This sample demonstrates how to send messages from a JMS queue producer into
 * an Azure Service Bus queue and receive them with a JMS message consumer.
 * JMS queue. 
 */
public class JmsQueueQuickstart {

    // Number of messages to send
    private static int totalSend = 10;
    //Tracking counter for how many messages have been received; used as termination condition
    private static AtomicInteger totalReceived = new AtomicInteger(0);
    // log4j logger 
    private static Logger logger = Logger.getRootLogger();

    public void run(String connectionString) throws Exception {

        // The connection string builder is the only part of the azure-servicebus SDK library
        // we use in this JMS sample and for the purpose of robustly parsing the Service Bus 
        // connection string. 
        ConnectionStringBuilder csb = new ConnectionStringBuilder(connectionString);
        
        // Set up JNDI context
        Hashtable<String, String> hashtable = new Hashtable<>();
        hashtable.put("connectionfactory.SBCF", "amqps://" + csb.getEndpoint().getHost() + "?amqp.idleTimeout=120000&amqp.traceFrames=true");
        hashtable.put("queue.QUEUE", "BasicQueue");
        hashtable.put(Context.INITIAL_CONTEXT_FACTORY, "org.apache.qpid.jms.jndi.JmsInitialContextFactory");
        Context context = new InitialContext(hashtable);
        ConnectionFactory cf = (ConnectionFactory) context.lookup("SBCF");
        
        // Look up queue
        Destination queue = (Destination) context.lookup("QUEUE");

        // We create a scope here so we can use the same set of local variables cleanly 
        // again to show the receive side separately with minimal clutter.
        {
            // Create connection
            Connection connection = cf.createConnection(csb.getSasKeyName(), csb.getSasKey());
            // Create session, no transaction, client ack
            Session session = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);

            // Create producer
            MessageProducer producer = session.createProducer(queue);

            // Send messages
            for (int i = 0; i < totalSend; i++) {
                BytesMessage message = session.createBytesMessage();
                message.writeBytes(String.valueOf(i).getBytes());
                producer.send(message);
                System.out.printf("Sent message %d.\n", i + 1);
            }

            producer.close();
            session.close();
            connection.stop();
            connection.close();
        }

        {
            // Create connection
            Connection connection = cf.createConnection(csb.getSasKeyName(), csb.getSasKey());
            connection.start();
            // Create session, no transaction, client ack
            Session session = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);
            // Create consumer
            MessageConsumer consumer = session.createConsumer(queue);
            // Create a listener callback to receive the messages
            consumer.setMessageListener(message -> {
                try {
                    // Received message is passed to callback
                    System.out.printf("Received message %d with sq#: %s\n",
                            totalReceived.incrementAndGet(), // increments the tracking counter
                            message.getJMSMessageID());
                    message.acknowledge();
                } catch (Exception e) {
                    logger.error(e);
                }
            });

            // Wait on the main thread until all sent messages have been received
            while (totalReceived.get() < totalSend) {
                Thread.sleep(1000);
            }
            consumer.close();
            session.close();
            connection.stop();
            connection.close();
        }

        System.out.printf("Received all messages, exiting the sample.\n");
        System.out.printf("Closing queue client.\n");
    }

    public static void main(String[] args) {

        System.exit(runApp(args, (connectionString) -> {
            JmsQueueQuickstart app = new JmsQueueQuickstart();
            try {
                app.run(connectionString);
                return 0;
            } catch (Exception e) {
                System.out.printf("%s", e.toString());
                return 1;
            }
        }));
    }

    static final String SB_SAMPLES_CONNECTIONSTRING = "SB_SAMPLES_CONNECTIONSTRING";

    public static int runApp(String[] args, Function<String, Integer> run) {
        try {

            String connectionString = null;

            // Parse connection string from command line
            Options options = new Options();
            options.addOption(new Option("c", true, "Connection string"));
            CommandLineParser clp = new DefaultParser();
            CommandLine cl = clp.parse(options, args);
            if (cl.getOptionValue("c") != null) {
                connectionString = cl.getOptionValue("c");
            }

            // Get overrides from the environment
            String env = System.getenv(SB_SAMPLES_CONNECTIONSTRING);
            if (env != null) {
                connectionString = env;
            }

            if (connectionString == null) {
                HelpFormatter formatter = new HelpFormatter();
                formatter.printHelp("run jar with", "", options, "", true);
                return 2;
            }
            return run.apply(connectionString);
        } catch (Exception e) {
            System.out.printf("%s", e.toString());
            return 3;
        }
    }
}
```

### <a name="run-the-application"></a>Выполнение приложения

Передайте **Строку подключения** из политики совместного доступа для запуска приложения.
Следующие выходные данные имеют форму запуска приложения:

```Output
> mvn clean package
>java -jar ./target/jmsqueuequickstart-1.0.0-jar-with-dependencies.jar -c "<CONNECTION_STRING>"

Sent message 1.
Sent message 2.
Sent message 3.
Sent message 4.
Sent message 5.
Sent message 6.
Sent message 7.
Sent message 8.
Sent message 9.
Sent message 10.
Received message 1 with sq#: ID:7f6a7659-bcdf-4af6-afc1-4011e2ddcb3c:1:1:1-1
Received message 2 with sq#: ID:7f6a7659-bcdf-4af6-afc1-4011e2ddcb3c:1:1:1-2
Received message 3 with sq#: ID:7f6a7659-bcdf-4af6-afc1-4011e2ddcb3c:1:1:1-3
Received message 4 with sq#: ID:7f6a7659-bcdf-4af6-afc1-4011e2ddcb3c:1:1:1-4
Received message 5 with sq#: ID:7f6a7659-bcdf-4af6-afc1-4011e2ddcb3c:1:1:1-5
Received message 6 with sq#: ID:7f6a7659-bcdf-4af6-afc1-4011e2ddcb3c:1:1:1-6
Received message 7 with sq#: ID:7f6a7659-bcdf-4af6-afc1-4011e2ddcb3c:1:1:1-7
Received message 8 with sq#: ID:7f6a7659-bcdf-4af6-afc1-4011e2ddcb3c:1:1:1-8
Received message 9 with sq#: ID:7f6a7659-bcdf-4af6-afc1-4011e2ddcb3c:1:1:1-9
Received message 10 with sq#: ID:7f6a7659-bcdf-4af6-afc1-4011e2ddcb3c:1:1:1-10
Received all messages, exiting the sample.
Closing queue client.

```

## <a name="amqp-disposition-and-service-bus-operation-mapping"></a>Расположение AMQP и сопоставление операции служебной шины

Ниже показано, как AMQPное расположение преобразуется в операцию служебной шины.

```Output
ACCEPTED = 1; -> Complete()
REJECTED = 2; -> DeadLetter()
RELEASED = 3; (just unlock the message in service bus, will then get redelivered)
MODIFIED_FAILED = 4; -> Abandon() which increases delivery count
MODIFIED_FAILED_UNDELIVERABLE = 5; -> Defer()
```

## <a name="jms-topics-vs-service-bus-topics"></a>Статьи о JMS и разделы служебной шины

Использование разделов и подписок служебной шины с помощью API JMS обеспечивает базовые возможности отправки и получения. Это удобный вариант при переносе приложений из других брокеров сообщений в интерфейсы API, совместимые с JMS, даже если разделы служебной шины отличаются от подразделов JMS и требует нескольких корректировок.

Разделы служебной шины направляют сообщения в именованные, общие и устойчивые подписки, управляемые через интерфейс управления ресурсами Azure, программы командной строки Azure или портал Azure. Каждая подписка позволяет использовать до 2 000 правил выбора, каждое из которых может иметь условие фильтра, а для фильтров SQL — также действие преобразования метаданных. Каждое соответствие условию фильтра выбирает входное сообщение, которое необходимо скопировать в подписку.  

Получение сообщений из подписок идентично приему сообщений из очередей. Каждая подписка имеет связанную очередь недоставленных сообщений и возможность автоматически пересылать сообщения в другую очередь или темы.

Темы JMS позволяют клиентам динамически создавать неустойчивые и устойчивые подписчики, которые при необходимости позволяют фильтровать сообщения с помощью селекторов сообщений. Эти несовместно используемые сущности не поддерживаются служебной шиной. Синтаксис правил фильтрации SQL для служебной шины аналогичен синтаксису селектора сообщений, поддерживаемому JMS.

Сторона издателя раздела JMS совместима с служебной шиной, как показано в этом примере, но динамические подписчики нет. В служебной шине не поддерживаются следующие API-интерфейсы JMS, связанные с топологией.

## <a name="unsupported-features-and-restrictions"></a>Неподдерживаемые возможности и ограничения

При использовании JMS через AMQP 1,0 с служебной шиной можно использовать следующие ограничения:

* В каждом сеансе допускается только один объект **MessageProducer** или **MessageConsumer** . Если в приложении необходимо создать несколько объектов **MessageProducer** или **MessageConsumer** , создайте выделенный сеанс для каждого из них.
* Подписки с временными разделами в настоящее время не поддерживаются.
* Объекты **мессажеселектор** в настоящее время не поддерживаются.
* Распределенные транзакции не поддерживаются, но поддерживаются сеансы транзакций.

Служебная шина разделяет плоскость управления с плоскости данных, поэтому она не поддерживает несколько функций динамической топологии JMS.

| Неподдерживаемый метод          | Заменить на                                                                             |
|-----------------------------|------------------------------------------------------------------------------------------|
| createDurableSubscriber     | Создайте подписку раздела, которая будет напортовать селектор сообщений.                                |
| createDurableConsumer       | Создайте подписку раздела, которая будет напортовать селектор сообщений.                                |
| createSharedConsumer        | Разделы служебной шины всегда являются общими. См. раздел "темы JMS и разделы служебной шины".                                    |
| createSharedDurableConsumer | Разделы служебной шины всегда являются общими. См. раздел "темы JMS и разделы служебной шины".                                      |
| createTemporaryTopic        | Создайте раздел с помощью API управления, инструментов или портала с *AutoDeleteOnIdle* , для которого задан срок действия. |
| createTopic                 | Создайте раздел с помощью API управления, инструментов или портала.                                         |
| unsubscribe                 | Удалите API управления разделами, инструменты или портал.                                            |
| createBrowser               | Не поддерживается. Используйте функцию Peek () API служебной шины.                         |
| createQueue                 | Создайте очередь с помощью API управления, инструментов или портала.                                           | 
| createTemporaryQueue        | Создайте очередь с помощью API управления, инструментов или портала с *AutoDeleteOnIdle* , для которого задан срок действия. |
| рецеивеноваит               | Используйте метод Receive (), предоставленный пакетом SDK служебной шины, и укажите очень низкое или нулевое время ожидания. |

## <a name="summary"></a>Итоги

В этой статье показано, как использовать функции обмена сообщениями через брокер служебной шины, такие как очереди и разделы публикации или подписки, из Java с помощью популярных API JMS и AMQP 1,0.

Вы также можете использовать служебную шину AMQP 1,0 на других языках, таких как .NET, C, Python и PHP. Компоненты, созданные с помощью этих разных языков, могут надежно обмениваться сообщениями с помощью поддержки AMQP 1,0 в служебной шине.

## <a name="next-steps"></a>Дальнейшие действия

* [Поддержка AMQP 1.0 в служебной шине](service-bus-amqp-overview.md)
* [Использование AMQP 1,0 с API .NET служебной шины](./service-bus-amqp-dotnet.md)
* [Руководство разработчика AMQP 1.0 для служебной шины](service-bus-amqp-dotnet.md)
* [Начало работы с очередями служебной шины](service-bus-dotnet-get-started-with-queues.md)
* [Центр разработчиков Java](https://azure.microsoft.com/develop/java/)