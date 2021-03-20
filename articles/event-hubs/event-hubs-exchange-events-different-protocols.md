---
title: Концентраторы событий Azure — обмен событиями с помощью разных протоколов
description: В этой статье показано, как через Центры событий Azure организовать обмен событиями между потребителями и производителями, которые используют разные протоколы (AMQP, Apache Kafka и HTTPS).
ms.topic: article
ms.date: 06/23/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: cbc6999e3ede73b948ce034769966922b4b0f282
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89010326"
---
# <a name="exchange-events-between-consumers-and-producers-that-use-different-protocols-amqp-kafka-and-https"></a>Обмен событиями через Центры событий Azure между потребителями и производителями, использующими разные протоколы (AMQP, Apache Kafka и HTTPS).
Центры событий Azure поддерживают для потребителей и производителей три протокола: AMQP, Kafka и HTTPS. В каждом из этих протоколов описан собственный способ представления сообщений, что приводит нас к логичному вопросу: как будут выглядеть разные компоненты и значения события при поступлении к потребителю, если в Центр событий они отправлялись по одному протоколу, а потребитель использовал для получения другой протокол? В этой статье обсуждаются современные рекомендации для производителей и потребителей, которые позволяют обеспечить правильную обработку потребителем всех значений в событии.

Приведенные здесь рекомендации применимы к перечисленным ниже клиентам, с указанием конкретных номеров версий, которые использовались при разработке фрагментов кода:

* клиент Kafka Java (версия 1.1.1 от https://www.mvnrepository.com/artifact/org.apache.kafka/kafka-clients);
* клиент Центров событий Microsoft Azure для Java (версия 1.1.0 от https://github.com/Azure/azure-event-hubs-java);
* клиент Центров событий Microsoft Azure для .NET (версия 2.1.0 от https://github.com/Azure/azure-event-hubs-dotnet);
* служебная шина Microsoft Azure (версия 5.0.0 от https://www.nuget.org/packages/WindowsAzure.ServiceBus).
* HTTPS (поддерживается только для поставщиков)

Поведение других клиентов AMQP может немного отличаться. AMQP использует четко определенную систему типов, но оставляет клиентам реализацию сериализации в эти типы из типов конкретного языка и обратно, а также предоставление доступа к компонентам сообщения AMQP.

## <a name="event-body"></a>Текст события
Все клиенты Microsoft AMQP представляют текст события в виде набора байтов без какой-либо обработки. Приложение-производитель передает клиенту последовательность байтов, а приложение-потребитель получает от клиента ту же последовательность в неизменном виде. Вся обработка последовательности байтов происходит только в коде приложений.

При отправке события через HTTPS текстом события считается все содержимое текста запроса, и обработка к нему не применяется. Аналогичное поведение можно легко реализовать для потребителя или производителя Kafka, используя предоставленные объекты ByteArraySerializer и ByteArrayDeserializer, как показано в следующем коде:

### <a name="kafka-byte-producer"></a>Производитель Kafka byte[]

```java
final Properties properties = new Properties();
// add other properties
properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, ByteArraySerializer.class.getName());

final KafkaProducer<Long, byte[]> producer = new KafkaProducer<Long, byte[]>(properties);

final byte[] eventBody = new byte[] { 0x01, 0x02, 0x03, 0x04 };
ProducerRecord<Long, byte[]> pr =
    new ProducerRecord<Long, byte[]>(myTopic, myPartitionId, myTimeStamp, eventBody);
```

### <a name="kafka-byte-consumer"></a>Потребитель Kafka byte[]
```java
final Properties properties = new Properties();
// add other properties
properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, ByteArrayDeserializer.class.getName());

final KafkaConsumer<Long, byte[]> consumer = new KafkaConsumer<Long, byte[]>(properties);

ConsumerRecord<Long, byte[]> cr = /* receive event */
// cr.value() is a byte[] with values { 0x01, 0x02, 0x03, 0x04 }
```

Этот код создает прозрачный конвейер байтов между двумя частями приложения и позволяет разработчику приложения вручную сериализовать и десериализовать данные так, как он того пожалает, в том числе принимать решения по десериализации во время выполнения, например с учетом типа данных или сведений об отправителе в пользовательских свойствах события.

Приложения, которые используют только один фиксированный тип текста события, могут применить другие сериализаторы и десериализаторы Kafka для прозрачного преобразования данных. В качестве примера давайте рассмотрим приложение с данными в формате JSON. Формирование и обработка строки JSON происходит на уровне приложения. На уровне Центров событий текст события всегда считается строкой, то есть последовательностью байтов с символами в кодировке UTF-8. В этом случае потребитель и (или) производитель Kafka смогут воспользоваться преимуществами предоставленных StringSerializer или StringDeserializer, как показано в следующем коде:

### <a name="kafka-utf-8-string-producer"></a>Производитель Kafka для строк в UTF-8
```java
final Properties properties = new Properties();
// add other properties
properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

final KafkaProducer<Long, String> producer = new KafkaProducer<Long, String>(properties);

final String exampleJson = "{\"name\":\"John\", \"number\":9001}";
ProducerRecord<Long, String> pr =
    new ProducerRecord<Long, String>(myTopic, myPartitionId, myTimeStamp, exampleJson);
```

### <a name="kafka-utf-8-string-consumer"></a>Потребитель Kafka для строк в UTF-8
```java
final Properties properties = new Properties();
// add other properties
properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

final KafkaConsumer<Long, String> consumer = new KafkaConsumer<Long, String>(properties);

ConsumerRecord<Long, Bytes> cr = /* receive event */
final String receivedJson = cr.value();
```

Для протокола AMQP платформы Java и .NET предоставляют встроенные способы преобразования строк в последовательность байтов UTF-8 и обратно. Клиенты Microsoft AMQP представляют события как класс с именем EventData. В следующих примерах вы увидите, как сериализовать строку UTF-8 в текст события EventData в производителе AMQP и затем десериализовать тело события EventData в строку UTF-8 в потребителе AMQP.

### <a name="java-amqp-utf-8-string-producer"></a>Производитель Java AMQP для строк в UTF-8
```java
final String exampleJson = "{\"name\":\"John\", \"number\":9001}";
final EventData ed = EventData.create(exampleJson.getBytes(StandardCharsets.UTF_8));
```

### <a name="java-amqp-utf-8-string-consumer"></a>Потребитель Java AMQP для строк в UTF-8
```java
EventData ed = /* receive event */
String receivedJson = new String(ed.getBytes(), StandardCharsets.UTF_8);
```

### <a name="c-net-utf-8-string-producer"></a>Производитель C# .NET для строк в UTF-8
```csharp
string exampleJson = "{\"name\":\"John\", \"number\":9001}";
EventData working = new EventData(Encoding.UTF8.GetBytes(exampleJson));
```

### <a name="c-net-utf-8-string-consumer"></a>Потребитель C# .NET для строк в UTF-8
```csharp
EventData ed = /* receive event */

// getting the event body bytes depends on which .NET client is used
byte[] bodyBytes = ed.Body.Array;  // Microsoft Azure Event Hubs Client for .NET
// byte[] bodyBytes = ed.GetBytes(); // Microsoft Azure Service Bus

string receivedJson = Encoding.UTF8.GetString(bodyBytes);
```

Так как Kafka имеет открытый исходный код, разработчик приложения может изучить реализацию любого сериализатора или десериализатора и создать код, который создает или обрабатывает совместимую с ним последовательность байтов на стороне AMQP.

## <a name="event-user-properties"></a>Пользовательские свойства события

Пользовательские свойства можно задавать и получать в клиентах AMQP (в клиентах Microsoft AMQP они называются свойствами) и Kafka (здесь они именуются заголовками). Отправители HTTPS могут задавать для события свойства, предоставляя их в операции POST в виде заголовков HTTP. Но при этом Kafka обрабатывает значения текстов и заголовков события как последовательность байтов. Клиенты AMQP, с другой стороны, используют для значений свойств типы и передают их путем кодирования значений свойств в систему типов AMQP.

В HTTPS используется особая схема работы. На момент отправки все значения свойств имеют формат текста в кодировке UTF-8. Служба Центров событий выполняет некоторую обработку для преобразования значений соответствующих свойств в кодировку AMQP для 32-разрядных и 64-разрядных целых чисел со знаком, 64-разрядные чисел с плавающей запятой и логических значений. Любое значение, которое не соответствует ни одному из этих типов, рассматривается как строка.

Сочетание этих подходов к типам свойств приводит к тому, что потребитель Kafka получит необработанную последовательность байтов в кодировке AMQP, включая информацию о типах AMQP. С другой стороны, потребитель AMQP получит нетипизированную последовательность байтов, отправленную производителем Kafka, и обработка данных полностью возлагается на приложение.

Для потребителей Kafka, которые получают свойства от производителей AMQP или HTTPS, используйте класс AmqpDeserializer, поведение которого соответствует другим десериализаторам экосистемы Kafka. Он обрабатывает сведения о типах, содержащиеся в последовательности байтов в кодировке AMQP, чтобы десериализировать байты данных в тип Java.

Мы рекомендуем включать специальное свойство в сообщения, отправленные через AMQP или HTTPS. Потребитель Kafka сможет с его помощью определить, нужно ли применять десериализацию AMQP для значений заголовков. Значение этого свойства не имеет значения. Достаточно присвоить ему хорошо известное имя, чтобы потребитель Kafka смог найти его в списке заголовков и изменить свое поведение соответствующим образом.

### <a name="amqp-to-kafka-part-1-create-and-send-an-event-in-c-net-with-properties"></a>Преобразование AMQP в Kafka, часть 1. Создание и отправка события на C# (.NET) с помощью свойств
```csharp
// Create an event with properties "MyStringProperty" and "MyIntegerProperty"
EventData working = new EventData(Encoding.UTF8.GetBytes("an event body"));
working.Properties.Add("MyStringProperty", "hello");
working.Properties.Add("MyIntegerProperty", 1234);

// BEST PRACTICE: include a property which indicates that properties will need AMQP deserialization
working.Properties.Add("AMQPheaders", 0);
```

### <a name="amqp-to-kafka-part-2-use-amqpdeserializer-to-deserialize-those-properties-in-a-kafka-consumer"></a>Преобразование AMQP в Kafka, часть 2. Десериализация этих свойств в потребителе Kafka с помощью AmqpDeserializer
```java
final AmqpDeserializer amqpDeser = new AmqpDeserializer();

ConsumerRecord<Long, Bytes> cr = /* receive event */
final Header[] headers = cr.headers().toArray();

final Header headerNamedMyStringProperty = /* find header with key "MyStringProperty" */
final Header headerNamedMyIntegerProperty = /* find header with key "MyIntegerProperty" */
final Header headerNamedAMQPheaders = /* find header with key "AMQPheaders", or null if not found */

// BEST PRACTICE: detect whether AMQP deserialization is needed
if (headerNamedAMQPheaders != null) {
    // The deserialize() method requires no prior knowledge of a property's type.
    // It returns Object and the application can check the type and perform a cast later.
    Object propertyOfUnknownType = amqpDeser.deserialize("topicname", headerNamedMyStringProperty.value());
    if (propertyOfUnknownType instanceof String) {
        final String propertyString = (String)propertyOfUnknownType;
        // do work here
    }
    propertyOfUnknownType = amqpDeser.deserialize("topicname", headerNamedMyIntegerProperty.value());
    if (propertyOfUnknownType instanceof Integer) {
        final Integer propertyInt = (Integer)propertyOfUnknownType;
        // do work here
    }
} else {
    /* event sent via Kafka, interpret header values the Kafka way */
}
```

Если приложение знает ожидаемый тип свойства, можно использовать методы десериализации, которые не требуют последующего приведения, но создают исключения в случае несовпадения ожидаемого типа для свойства.

### <a name="amqp-to-kafka-part-3-a-different-way-of-using-amqpdeserializer-in-a-kafka-consumer"></a>Преобразование AMQP в Kafka, часть 3. Другой способ применения AmqpDeserializer в потребителе Kafka
```java
// BEST PRACTICE: detect whether AMQP deserialization is needed
if (headerNamedAMQPheaders != null) {
    // Property "MyStringProperty" is expected to be of type string.
    try {
        final String propertyString = amqpDeser.deserializeString(headerNamedMyStringProperty.value());
        // do work here
    }
    catch (IllegalArgumentException e) {
        // property was not a string
    }

    // Property "MyIntegerProperty" is expected to be a signed integer type.
    // The method returns long because long can represent the value range of all AMQP signed integer types.
    try {
        final long propertyLong = amqpDeser.deserializeSignedInteger(headerNamedMyIntegerProperty.value());
        // do work here
    }
    catch (IllegalArgumentException e) {
        // property was not a signed integer
    }
} else {
    /* event sent via Kafka, interpret header values the Kafka way */
}
```

Обратное преобразование требует больше усилий, поскольку заголовки производителя Kafka всегда получаются потребителем AMQP как необработанные байты (тип org.apache.qpid.proton.amqp.Binary для клиента Центров событий Microsoft Azure для Java или тип `System.Byte[]` для клиентов AMQP Microsoft .NET). Самый простой способ — применить один из стандартных десериализаторов Kafka, чтобы получить байты для значений заголовков на стороне производителя Kafka, а на стороне потребителя AMQP создать свой совместимый код десериализации.

Как и для преобразования AMQP в Kafka, мы рекомендуем включать специальное свойство во все сообщения, отправляемые через Kafka. Потребитель AMQP сможет с помощью этого свойства определить, нужно ли применять десериализацию для значений заголовков. Значение этого свойства не имеет значения. Достаточно присвоить ему хорошо известное имя, чтобы потребитель AMQP смог найти его в списке заголовков и изменить свое поведение соответствующим образом. Если производитель Kafka нельзя изменить, остается возможность проверять в приложении-потребителе, имеет ли значение свойства двоичный или байтовый тип, а затем выполнять десериализацию в зависимости от этого типа.

### <a name="kafka-to-amqp-part-1-create-and-send-an-event-from-kafka-with-properties"></a>Преобразование Kafka в AMQP, часть 1. Создание и отправка события из Kafka с помощью свойств
```java
final String topicName = /* topic name */
final ProducerRecord<Long, String> pr = new ProducerRecord<Long, String>(topicName, /* other arguments */);
final Headers h = pr.headers();

// Set headers using Kafka serializers
IntegerSerializer intSer = new IntegerSerializer();
h.add("MyIntegerProperty", intSer.serialize(topicName, 1234));

LongSerializer longSer = new LongSerializer();
h.add("MyLongProperty", longSer.serialize(topicName, 5555555555L));

ShortSerializer shortSer = new ShortSerializer();
h.add("MyShortProperty", shortSer.serialize(topicName, (short)22222));

FloatSerializer floatSer = new FloatSerializer();
h.add("MyFloatProperty", floatSer.serialize(topicName, 1.125F));

DoubleSerializer doubleSer = new DoubleSerializer();
h.add("MyDoubleProperty", doubleSer.serialize(topicName, Double.MAX_VALUE));

StringSerializer stringSer = new StringSerializer();
h.add("MyStringProperty", stringSer.serialize(topicName, "hello world"));

// BEST PRACTICE: include a property which indicates that properties will need deserialization
h.add("RawHeaders", intSer.serialize(0));
```

### <a name="kafka-to-amqp-part-2-manually-deserialize-those-properties-in-c-net"></a>Преобразование Kafka в AMQP, часть 2. Десериализация этих свойств вручную на C# (.NET)
```csharp
EventData ed = /* receive event */

// BEST PRACTICE: detect whether manual deserialization is needed
if (ed.Properties.ContainsKey("RawHeaders"))
{
    // Kafka serializers send bytes in big-endian order, whereas .NET on x86/x64 is little-endian.
    // Therefore it is frequently necessary to reverse the bytes before further deserialization.

    byte[] rawbytes = ed.Properties["MyIntegerProperty"] as System.Byte[];
    if (BitConverter.IsLittleEndian)
    {
            Array.Reverse(rawbytes);
    }
    int myIntegerProperty = BitConverter.ToInt32(rawbytes, 0);

    rawbytes = ed.Properties["MyLongProperty"] as System.Byte[];
    if (BitConverter.IsLittleEndian)
    {
            Array.Reverse(rawbytes);
    }
    long myLongProperty = BitConverter.ToInt64(rawbytes, 0);

    rawbytes = ed.Properties["MyShortProperty"] as System.Byte[];
    if (BitConverter.IsLittleEndian)
    {
            Array.Reverse(rawbytes);
    }
    short myShortProperty = BitConverter.ToInt16(rawbytes, 0);

    rawbytes = ed.Properties["MyFloatProperty"] as System.Byte[];
    if (BitConverter.IsLittleEndian)
    {
            Array.Reverse(rawbytes);
    }
    float myFloatProperty = BitConverter.ToSingle(rawbytes, 0);

    rawbytes = ed.Properties["MyDoubleProperty"] as System.Byte[];
    if (BitConverter.IsLittleEndian)
    {
            Array.Reverse(rawbytes);
    }
    double myDoubleProperty = BitConverter.ToDouble(rawbytes, 0);

    rawbytes = ed.Properties["MyStringProperty"] as System.Byte[];
string myStringProperty = Encoding.UTF8.GetString(rawbytes);
}
```

### <a name="kafka-to-amqp-part-3-manually-deserialize-those-properties-in-java"></a>Преобразование Kafka в AMQP, часть 3. Десериализация этих свойств вручную на Java
```java
final EventData ed = /* receive event */

// BEST PRACTICE: detect whether manual deserialization is needed
if (ed.getProperties().containsKey("RawHeaders")) {
    byte[] rawbytes =
        ((org.apache.qpid.proton.amqp.Binary)ed.getProperties().get("MyIntegerProperty")).getArray();
    int myIntegerProperty = 0;
    for (byte b : rawbytes) {
        myIntegerProperty <<= 8;
        myIntegerProperty |= ((int)b & 0x00FF);
    }

    rawbytes = ((org.apache.qpid.proton.amqp.Binary)ed.getProperties().get("MyLongProperty")).getArray();
    long myLongProperty = 0;
    for (byte b : rawbytes) {
        myLongProperty <<= 8;
        myLongProperty |= ((long)b & 0x00FF);
    }

    rawbytes = ((org.apache.qpid.proton.amqp.Binary)ed.getProperties().get("MyShortProperty")).getArray();
    short myShortProperty = (short)rawbytes[0];
    myShortProperty <<= 8;
    myShortProperty |= ((short)rawbytes[1] & 0x00FF);

    rawbytes = ((org.apache.qpid.proton.amqp.Binary)ed.getProperties().get("MyFloatProperty")).getArray();
    int intbits = 0;
    for (byte b : rawbytes) {
        intbits <<= 8;
        intbits |= ((int)b & 0x00FF);
    }
    float myFloatProperty = Float.intBitsToFloat(intbits);

    rawbytes = ((org.apache.qpid.proton.amqp.Binary)ed.getProperties().get("MyDoubleProperty")).getArray();
    long longbits = 0;
    for (byte b : rawbytes) {
        longbits <<= 8;
        longbits |= ((long)b & 0x00FF);
    }
    double myDoubleProperty = Double.longBitsToDouble(longbits);

    rawbytes = ((org.apache.qpid.proton.amqp.Binary)ed.getProperties().get("MyStringProperty")).getArray();
String myStringProperty = new String(rawbytes, StandardCharsets.UTF_8);
}
```

## <a name="next-steps"></a>Дальнейшие действия
Из этой статьи вы узнали, как выполнять потоковую передачу данных в Центры событий без необходимости менять клиенты протоколов или запускать собственные кластеры. Дополнительные сведения о Центрах событий и Центрах событий для Kafka см. в следующих статьях:  

* [Сведения о Центрах событий](./event-hubs-about.md)
* [Azure Event Hubs for Apache Kafka (preview)](event-hubs-for-kafka-ecosystem-overview.md) (Центры событий Azure для Apache Kafka (предварительный просмотр))
* [Дополнительные примеры для службы "Центры событий" для Kafka на сайте GitHub](https://github.com/Azure/azure-event-hubs-for-kafka)
* Используйте [MirrorMaker](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=27846330) для [потоковой передачи событий из Kafka в локальной среде в концентраторы событий в облаке.](event-hubs-kafka-mirror-maker-tutorial.md)
* Узнайте, как выполнять потоковую передачу в концентраторы событий с помощью [собственных приложений Kafka](event-hubs-quickstart-kafka-enabled-event-hubs.md), [Apache Флинк](event-hubs-kafka-flink-tutorial.md)или [потоков Akka Streams](event-hubs-kafka-akka-streams-tutorial.md) .
