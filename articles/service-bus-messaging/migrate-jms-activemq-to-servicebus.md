---
title: Перенос приложений службы сообщений Java (JMS) из Apache Активемк в служебную шину Azure | Документация Майкрософт
description: В этой статье объясняется, как перенести существующие приложения JMS, взаимодействующие с Apache Активемк, для взаимодействия с служебной шиной Azure.
services: service-bus-messaging
documentationcenter: ''
author: axisc
manager: timlt
editor: spelluru
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/07/2020
ms.author: aschhab
ms.custom: devx-track-java
ms.openlocfilehash: b8408dde86d1902cf5b4899c4783c9dd185449ee
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92515752"
---
# <a name="migrate-existing-java-message-service-jms-20-applications-from-apache-activemq-to-azure-service-bus"></a>Перенос существующих приложений службы сообщений Java (JMS) 2,0 из Apache Активемк в служебную шину Azure

В этой статье описывается, как изменить существующее приложение службы сообщений Java (JMS) 2,0, которое взаимодействует с брокером JMS для взаимодействия с служебной шиной Azure. В частности, в статье рассматривается миграция с Apache Активемк или Amazon MQ.

Служебная шина Azure поддерживает платформу Java 2, выпуск Enterprise Edition и пружинные рабочие нагрузки, использующие API JMS 2,0 через расширенный протокол очередей сообщений (AMQP).

## <a name="before-you-start"></a>Перед началом работы

### <a name="differences-between-azure-service-bus-and-apache-activemq"></a>Различия между служебной шиной Azure и Apache Активемк

Служебная шина Azure и Apache Активемк — это оба брокера сообщений, которые работают как поставщики JMS для клиентских приложений для отправки сообщений и получения сообщений от. Они включают семантику "точка-точка" с очередями и семантику публикации и подписки с разделами и подписками. 

Несмотря на это, существуют некоторые различия между ними, как показано в следующей таблице.

| Category | ActiveMQ | Azure Service Bus |
| --- | --- | --- |
| Распределение по уровням приложений | Кластеризованный Монолит | Двухуровневая <br> (шлюз + серверная части) |
| Поддержка протоколов | <ul> <li>AMQP</li> <li> стомп </li> <li> опенвире </li> </ul> | AMQP |
| Режим подготовки | <ul> <li> Инфраструктура как услуга (IaaS) в локальной среде </li> <li> Amazon MQ (управляемая платформа как услуга) </li> | Управляемая платформа как услуга (PaaS) |
| Размер сообщения | Настраиваемый пользователем | 1 МБ (уровень Premium) |
| Высокий уровень доступности | Организуется пользователем | Управляемая платформа |
| Аварийное восстановление | Организуется пользователем | Управляемая платформа | 

### <a name="current-supported-and-unsupported-features"></a>Текущие поддерживаемые и неподдерживаемые функции

[!INCLUDE [service-bus-jms-features-list](../../includes/service-bus-jms-feature-list.md)]

### <a name="considerations"></a>Рекомендации

Двухэтапный уровень служебной шины Azure предоставляет различные возможности обеспечения непрерывности бизнес-процессов (высокий уровень доступности и аварийное восстановление). Однако при использовании функций JMS необходимо учитывать некоторые моменты.

#### <a name="service-upgrades"></a>Обновления службы

В случае обновления и перезапуска служебной шины временные очереди или разделы удаляются. Если ваше приложение чувствительно к утере данных во временных очередях или разделах, не используйте временные очереди или разделы. Вместо этого используйте устойчивые очереди, разделы и подписки.

#### <a name="data-migration"></a>Перенос данных

В рамках миграции и изменения клиентских приложений для взаимодействия с служебной шиной Azure данные, хранящиеся в Активемк, не переносятся в служебную шину. Может потребоваться пользовательское приложение для очистки очередей, разделов и подписок Активемк, а также воспроизведения сообщений в очередях, разделах и подписках служебной шины.

#### <a name="authentication-and-authorization"></a>Аутентификация и авторизация

Контроль доступа на основе ролей Azure (Azure RBAC), поддерживаемый Azure Active Directory, является предпочтительным механизмом проверки подлинности для служебной шины. Так как Azure RBAC или аутентификация на основе утверждений в настоящее время не поддерживается в Apache QPID JMS, для проверки подлинности следует использовать ключи SAS.

## <a name="pre-migration"></a>Подготовка к миграции

### <a name="version-check"></a>Проверка версии

При написании приложений JMS используются следующие компоненты и версии: 

| Компонент | Версия |
|---|---|
| API-интерфейс службы сообщений Java (JMS) | 1,1 или выше |
| Протокол AMQP | 1.0 |

### <a name="ensure-that-amqp-ports-are-open"></a>Убедитесь, что порты AMQP открыты

Служебная шина поддерживает обмен данными по протоколу AMQP. Для этой цели необходимо разрешить обмен данными через порты 5671 (AMQP) и 443 (TCP). В зависимости от того, где размещаются клиентские приложения, может потребоваться обращение в службу поддержки, чтобы разрешить обмен данными через эти порты.

> [!IMPORTANT]
> Служебная шина поддерживает только протокол AMQP 1,0.

### <a name="set-up-enterprise-configurations"></a>Настройка корпоративных конфигураций

Служебная шина обеспечивает различные возможности корпоративной безопасности и обеспечения высокого уровня доступности. Дополнительные сведения см. в разделе: 

  * [Конечные точки службы для виртуальной сети](service-bus-service-endpoints.md)
  * [Брандмауэр](service-bus-ip-filtering.md)
  * [Шифрование на стороне службы с помощью управляемого клиентом ключа (BYOK)](configure-customer-managed-key.md)
  * [Частные конечные точки](private-link-service.md)
  * [Аутентификация и авторизация](service-bus-authentication-and-authorization.md)

### <a name="monitoring-alerts-and-tracing"></a>Мониторинг, оповещения и трассировка

Для каждого пространства имен служебной шины метрики публикуются на Azure Monitor. Эти метрики можно использовать для оповещения и динамического масштабирования ресурсов, выделенных пространству имен.

Дополнительные сведения о различных метриках и о настройке оповещений см. [в разделе метрики служебной шины в Azure Monitor](service-bus-metrics-azure-monitor.md). Кроме того, вы можете получить дополнительные сведения о [трассировке на стороне клиента для операций с данными](service-bus-end-to-end-tracing.md) , а также [оперативного и диагностического ведения журнала для операций управления](service-bus-diagnostic-logs.md).

### <a name="metrics---new-relic"></a>Метрики — новые Relic

Вы можете сопоставить метрики из Активемк Map с метриками в служебной шине Azure. См. следующие сведения с нового веб-сайта Relic:

  * [Активемк/Amazon MQ новые метрики Relic](https://docs.newrelic.com/docs/integrations/amazon-integrations/aws-integrations-list/aws-mq-integration)
  * [Новые метрики Relic в служебной шине Azure](https://docs.newrelic.com/docs/integrations/microsoft-azure-integrations/azure-integrations-list/azure-service-bus-monitoring-integration)

> [!NOTE]
> Сейчас у новых Relic нет прямой, простой интеграции с Активемк, но у них есть метрики, доступные для Amazon MQ. Поскольку Amazon MQ является производным от Активемк, в следующей таблице сопоставлены новые метрики Relic из Amazon MQ в служебную шину Azure.
>

|Группирование метрик| Метрика Amazon MQ/Активемк | Метрика служебной шины Azure |
|------------|---------------------------|--------------------------|
|Broker|`CpuUtilization`|`CPUXNS`|
|Broker|`MemoryUsage`|`WSXNS`|
|Broker|`CurrentConnectionsCount`|`activeConnections`|
|Broker|`EstablishedConnectionsCount`|`activeConnections` + `connectionsClosed`|
|Broker|`InactiveDurableTopicSubscribersCount`|Использование метрик подписки|
|Broker|`TotalMessageCount`|Использование уровня очереди, раздела или подписки `activeMessages`|
|Очередь или раздел|`EnqueueCount`|`incomingMessages`|
|Очередь или раздел|`DequeueCount`|`outgoingMessages`|
|Очередь|`QueueSize`|`sizeBytes`|



## <a name="migration"></a>Миграция

Чтобы перенести существующее приложение JMS 2,0 для взаимодействия с служебной шиной, выполните действия, описанные в следующих разделах.

### <a name="export-the-topology-from-activemq-and-create-the-entities-in-service-bus-optional"></a>Экспорт топологии из Активемк и создание сущностей в служебной шине (необязательно)

Чтобы обеспечить возможность беспрепятственного подключения клиентских приложений к служебной шине, перенесите топологию (включая очереди, разделы и подписки) из Apache Активемк в служебную шину.

> [!NOTE]
> Для приложений JMS в качестве операции времени выполнения создаются очереди, разделы и подписки. Большинство поставщиков JMS (брокеров сообщений) дают возможность создавать их во время выполнения. Вот почему этот шаг экспорта считается необязательным. Чтобы убедиться, что приложение имеет разрешения на создание топологии во время выполнения, используйте строку подключения с `Manage` разрешениями SAS.

Для этого выполните следующие действия.

1. Чтобы экспортировать топологию, используйте [программы командной строки активемк](https://activemq.apache.org/activemq-command-line-tools-reference) .
1. Повторно создайте ту же топологию с помощью [шаблона Azure Resource Manager](../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).
1. Запустите шаблон Azure Resource Manager.


### <a name="import-the-maven-dependency-for-service-bus-jms-implementation"></a>Импорт зависимости Maven для реализации JMS служебной шины

Чтобы обеспечить бесперебойное подключение к служебной шине, добавьте `azure-servicebus-jms` пакет в качестве зависимости в `pom.xml` файл Maven следующим образом:

```xml
<dependencies>
...
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-servicebus-jms</artifactId>
    </dependency>
...
</dependencies>
```

### <a name="application-server-configuration-changes"></a>Изменения конфигурации сервера приложений

Эта часть настроена для сервера приложений, на котором размещены клиентские приложения, подключающиеся к Активемк.

#### <a name="tomcat"></a>Tomcat

Здесь вы начинаете с конфигурации, относящейся к Активемк, как показано в `/META-INF/context.xml` файле.

```XML
<Context antiJARLocking="true">
    <Resource
        name="jms/ConnectionFactory"
        auth="Container"
        type="org.apache.activemq.ActiveMQConnectionFactory"
        description="JMS Connection Factory"
        factory="org.apache.activemq.jndi.JNDIReferenceFactory"
        brokerURL="tcp://localhost:61616"
        brokerName="LocalActiveMQBroker"
        useEmbeddedBroker="false"/>

    <Resource name="jms/topic/MyTopic"
        auth="Container"
        type="org.apache.activemq.command.ActiveMQTopic"
        factory="org.apache.activemq.jndi.JNDIReferenceFactory"
        physicalName="MY.TEST.FOO"/>
    <Resource name="jms/queue/MyQueue"
        auth="Container"
        type="org.apache.activemq.command.ActiveMQQueue"
        factory="org.apache.activemq.jndi.JNDIReferenceFactory"
        physicalName="MY.TEST.FOO.QUEUE"/>
</Context>
```

Это можно адаптировать для указания служебной шины следующим образом:

```xml
<Context antiJARLocking="true">
    <Resource
        name="jms/ConnectionFactory"
        auth="Container"
        type="com.microsoft.azure.servicebus.jms.ServiceBusJmsConnectionFactory"
        description="JMS Connection Factory"
        factory="org.apache.qpid.jms.jndi.JNDIReferenceFactory"
        connectionString="<INSERT YOUR SERVICE BUS CONNECTION STRING HERE>"/>

    <Resource name="jms/topic/MyTopic"
        auth="Container"
        type="org.apache.qpid.jms.JmsTopic"
        factory="org.apache.qpid.jms.jndi.JNDIReferenceFactory"
        physicalName="MY.TEST.FOO"/>
    <Resource name="jms/queue/MyQueue"
        auth="Container"
        type="org.apache.qpid.jms.JmsQueue"
        factory="org.apache.qpid.jms.jndi.JNDIReferenceFactory"
        physicalName="MY.TEST.FOO.QUEUE"/>
</Context>
```

#### <a name="spring-applications"></a>Приложения Spring

##### <a name="update-the-applicationproperties-file"></a>Обновление `application.properties` файла

Если вы используете приложение с пружинной загрузкой для подключения к Активемк, необходимо удалить свойства, относящиеся к Активемк, из `application.properties` файла.

```properties
spring.activemq.broker-url=<ACTIVEMQ BROKER URL>
spring.activemq.user=<ACTIVEMQ USERNAME>
spring.activemq.password=<ACTIVEMQ PASSWORD>
```

Затем добавьте в файл свойства, относящиеся к служебной шине `application.properties` .

```properties
azure.servicebus.connection-string=Endpoint=myEndpoint;SharedAccessKeyName=mySharedAccessKeyName;SharedAccessKey=mySharedAccessKey
```

##### <a name="replace-activemqconnectionfactory-with-servicebusjmsconnectionfactory"></a>Замените `ActiveMQConnectionFactory` на `ServiceBusJmsConnectionFactory`:

Следующим шагом является замена экземпляра на `ActiveMQConnectionFactory` `ServiceBusJmsConnectionFactory` .

> [!NOTE] 
> Фактические изменения кода зависят от приложения и от управления зависимостями, но в следующем примере приведены рекомендации по изменению.
>

Ранее вы могли создать экземпляр объекта `ActiveMQConnectionFactory` , как показано ниже.

```java

String BROKER_URL = "<URL of the hosted ActiveMQ broker>";
ConnectionFactory factory = new ActiveMQConnectionFactory(BROKER_URL);

Connection connection = factory.createConnection();
connection.start();

```

Теперь вы изменяете его для создания экземпляра объекта `ServiceBusJmsConnectionFactory` , как показано ниже.

```java

ServiceBusJmsConnectionFactorySettings settings = new ServiceBusJmsConnectionFactorySettings();
String SERVICE_BUS_CONNECTION_STRING = "<Service Bus Connection string>";

ConnectionFactory factory = new ServiceBusJmsConnectionFactory(SERVICE_BUS_CONNECTION_STRING, settings);

Connection connection = factory.createConnection();
connection.start();

```

## <a name="post-migration"></a>После миграции

Теперь, когда вы изменили приложение, чтобы начать отправку и получение сообщений из служебной шины, следует убедиться, что она работает правильно. По завершении вы можете продолжить и модернизировать стека приложений.

## <a name="next-steps"></a>Дальнейшие действия

Используйте [стартовую начальную загрузку для JMS служебной шины Azure](/azure/developer/java/spring-framework/configure-spring-boot-starter-java-app-with-azure-service-bus) для эффективной интеграции с служебной шиной.

Дополнительные сведения об обмене сообщениями служебной шины и JMS см. в следующих статьях:

* [JMS служебной шины](service-bus-java-how-to-use-jms-api-amqp.md)
* [Очереди, разделы и подписки служебной шины](service-bus-queues-topics-subscriptions.md)
* [Начало работы с очередями служебной шины](service-bus-dotnet-get-started-with-queues.md)