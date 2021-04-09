---
title: Руководство по API производителя и потребителя Apache Kafka в Azure HDInsight
description: Узнайте, как использовать API производителя и потребителя Apache Kafka для Kafka в HDInsight. В этом руководстве вы узнаете, как использовать эти API с Kafka в HDInsight из приложения Java.
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: tutorial
ms.date: 05/19/2020
ms.openlocfilehash: e5a635a8837aadaf423c6f3a0925dbac4080e60f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98945177"
---
# <a name="tutorial-use-the-apache-kafka-producer-and-consumer-apis"></a>Руководство по Использование API производителя и потребителя Apache Kafka

Узнайте, как использовать API производителя и потребителя Apache Kafka для Kafka в HDInsight.

API производителя Kafka позволяет приложениям отправлять потоки данных в кластер Kafka. API потребителя Kafka позволяет приложениям считывать потоки данных из кластера.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Предварительные требования
> * Изучение кода
> * Создание и развертывание приложения.
> * Запуск приложения в кластере

Дополнительные сведения об интерфейсах API см. в документации [по API производителя](https://kafka.apache.org/documentation/#producerapi) и [API потребителя](https://kafka.apache.org/documentation/#consumerapi).

## <a name="prerequisites"></a>Предварительные требования

* Apache Kafka в кластере HDInsight. Чтобы узнать, как создать кластер, ознакомьтесь со статьей [Краткое руководство. Создание кластера Apache Kafka в Azure HDInsight с помощью портала Azure](apache-kafka-get-started.md).
* Пакет [Java Developer Kit (JDK) версии 8](/azure/developer/java/fundamentals/java-jdk-long-term-support) или аналогичный пакет, например OpenJDK.
* Средство [Apache Maven](https://maven.apache.org/download.cgi), [установленное](https://maven.apache.org/install.html) согласно инструкций Apache.  Maven — система сборки проектов Java.
* Клиент SSH, например, Putty. Дополнительные сведения см. в руководстве по [подключению к HDInsight (Apache Hadoop) с помощью SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="understand-the-code"></a>Изучение кода

Пример приложения расположен в подкаталоге `Producer-Consumer` по адресу [https://github.com/Azure-Samples/hdinsight-kafka-java-get-started](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started). Если вы используете кластер Kafka с включенным **Корпоративным пакетом безопасности (ESP)** , следует использовать версию приложения, расположенную в подкаталоге `DomainJoined-Producer-Consumer`.

Приложение состоит в основном из четырех файлов:
* `pom.xml`: этот файл определяет зависимости проекта, версию Java и методы упаковки.
* `Producer.java`: этот файл отправляет случайные предложения в Kafka, используя API производителя.
* `Consumer.java`: этот файл использует API потребителя для чтения данных из Kafka и передачи их в STDOUT.
* `AdminClientWrapper.java`: Этот файл использует API администратора для создания, описания и удаления разделов Kafka.
* `Run.java`: интерфейс командной строки используется для выполнения кода потребителя и производителя.

### <a name="pomxml"></a>Pom.xml

В файле `pom.xml` важны следующие элементы:

* Зависимости. Этот проект использует API производителя и потребителя Kafka, предоставленные в пакете `kafka-clients`. Приведенный ниже код XML определяет эту зависимость:

    ```xml
    <!-- Kafka client for producer/consumer operations -->
    <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>${kafka.version}</version>
    </dependency>
    ```

    Запись `${kafka.version}` объявлена в разделе `<properties>..</properties>` файла `pom.xml`. Она настроена для версии Kafka кластера HDInsight.

* Подключаемые модули. Подключаемые модули Maven предоставляют различные возможности. В этом проекте используются следующие подключаемые модули:

    * `maven-compiler-plugin`: с помощью этого модуля можно задать для проекта Java версии 8. Это версия Java, используемая в HDInsight 3.6.
    * `maven-shade-plugin`: используется для создания файла типа uber jar, содержащего это приложение, а также любые зависимости. Он также используется для установки точки входа приложения, с помощью которой вы сможете напрямую запускать JAR-файл, не указывая основной класс.

### <a name="producerjava"></a>Producer.java

Производитель взаимодействует с узлами брокера Kafka (рабочие узлы) и отправляет данные в разделы Kafka. Следующий фрагмент кода взят из файла [Producer.java](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/blob/master/Producer-Consumer/src/main/java/com/microsoft/example/Producer.java) в [репозитории GitHub](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started). Этот фрагмент демонстрирует, как задавать свойства производителя. Для кластеров с поддержкой корпоративной безопасности необходимо добавить дополнительное свойство properties.setProperty(CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, "SASL_PLAINTEXT");

```java
Properties properties = new Properties();
// Set the brokers (bootstrap servers)
properties.setProperty("bootstrap.servers", brokers);
// Set how to serialize key/value pairs
properties.setProperty("key.serializer","org.apache.kafka.common.serialization.StringSerializer");
properties.setProperty("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
KafkaProducer<String, String> producer = new KafkaProducer<>(properties);
```

### <a name="consumerjava"></a>Consumer.java

Потребитель взаимодействует с узлами брокера Kafka (рабочие узлы) и считывает записи в цикле. В следующем фрагменте кода из файла [Consumer.java](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/blob/master/Producer-Consumer/src/main/java/com/microsoft/example/Consumer.java) задаются свойства потребителя. Для кластеров с поддержкой корпоративной безопасности необходимо добавить дополнительное свойство properties.setProperty(CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, "SASL_PLAINTEXT");

```java
KafkaConsumer<String, String> consumer;
// Configure the consumer
Properties properties = new Properties();
// Point it to the brokers
properties.setProperty("bootstrap.servers", brokers);
// Set the consumer group (all consumers must belong to a group).
properties.setProperty("group.id", groupId);
// Set how to serialize key/value pairs
properties.setProperty("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
properties.setProperty("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
// When a group is first created, it has no offset stored to start reading from. This tells it to start
// with the earliest record in the stream.
properties.setProperty("auto.offset.reset","earliest");

consumer = new KafkaConsumer<>(properties);
```

В этом коде потребитель настроен для чтения с самого начала раздела (`auto.offset.reset` имеет значение `earliest`).

### <a name="runjava"></a>Run.java

Файл [Run.java](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/blob/master/Producer-Consumer/src/main/java/com/microsoft/example/Run.java) предоставляет интерфейс командной строки, который выполняет код производителя или потребителя. Нужно указать сведения об узле брокера Kafka в качестве параметра. При необходимости можно указать значение идентификатора группы, которое используется процессом потребителя. При создании нескольких экземпляров потребителя с одним и тем же идентификатором группы между ними будет распределяться нагрузка по операциям чтения из раздела.

## <a name="build-and-deploy-the-example"></a>Создание и развертывание примера

### <a name="use-pre-built-jar-files"></a>Использование предварительно созданных JAR-файлов

Скачайте JAR-файлы из [примера Azure по началу работы с Kafka](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/tree/master/Prebuilt-Jars). Если кластер имеет **Корпоративный пакет безопасности (ESP)** , используйте файл kafka-producer-consumer-esp.jar. Выполните приведенную ниже команду, чтобы скопировать JAR-файлы в кластер.

```cmd
scp kafka-producer-consumer*.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.net:kafka-producer-consumer.jar
```

### <a name="build-the-jar-files-from-code"></a>Создание JAR-файлов из кода

Если вы хотите пропустить этот шаг, предварительно созданные JAR можно скачать из подкаталога `Prebuilt-Jars`. Скачайте файл kafka-producer-consumer.jar. Если кластер имеет **Корпоративный пакет безопасности (ESP)** , используйте файл kafka-producer-consumer-esp.jar. Выполните шаг 3, чтобы скопировать JAR-файл в кластер HDInsight.

1. Скачайте и извлеките примеры по ссылке: [https://github.com/Azure-Samples/hdinsight-kafka-java-get-started](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started).

2. Укажите для текущего каталога расположение каталога `hdinsight-kafka-java-get-started\Producer-Consumer`. Если вы используете кластер Kafka с включенным **Корпоративным пакетом безопасности (ESP)** , установите расположением подкаталог `DomainJoined-Producer-Consumer`. Запустите сборку приложения с помощью следующей команды.

    ```cmd
    mvn clean package
    ```

    Эта команда создает каталог с именем `target`, который содержит файл с именем `kafka-producer-consumer-1.0-SNAPSHOT.jar`. Для кластеров с корпоративным пакетом безопасности будет использоваться файл `kafka-producer-consumer-esp-1.0-SNAPSHOT.jar`.

3. Замените `sshuser` именем пользователя SSH для кластера, а `CLUSTERNAME` — именем кластера. Используя следующую команду, скопируйте файл `kafka-producer-consumer-1.0-SNAPSHOT.jar` в свой кластер HDInsight. При появлении запроса введите пароль пользователя SSH.

    ```cmd
    scp ./target/kafka-producer-consumer*.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.net:kafka-producer-consumer.jar
    ```

## <a name="run-the-example"></a><a id="run"></a>Выполнение примера

1. Замените `sshuser` именем пользователя SSH для кластера, а `CLUSTERNAME` — именем кластера. Откройте SSH-подключение к кластеру, выполнив следующую команду. При появлении запроса введите пароль для учетной записи пользователя SSH.

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Чтобы получить узлы брокера Kafka, замените значения для `<clustername>` и `<password>` в следующей команде и выполните ее. Используйте тот же регистр для `<clustername>`, как показано на портале Azure. Замените `<password>` паролем для входа в кластер, а затем выполните:

    ```bash
    sudo apt -y install jq
    export clusterName='<clustername>'
    export password='<password>'
    export KAFKABROKERS=$(curl -sS -u admin:$password -G https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2);
    ```

    > [!Note]  
    > Для этой команды требуется доступ к Ambari. Если кластер находится за пределами NSG, выполните эту команду на компьютере с доступом к Ambari.

1. Создайте раздел Kafka, `myTest`, использовав следующую команду.

    ```bash
    java -jar kafka-producer-consumer.jar create myTest $KAFKABROKERS
    ```

1. Чтобы запустить производитель и сохранить данные в разделе, выполните следующую команду:

    ```bash
    java -jar kafka-producer-consumer.jar producer myTest $KAFKABROKERS
    ```

1. Когда процесс производителя завершится, выполните следующую команду для считывания данных из раздела:

    ```bash
    java -jar kafka-producer-consumer.jar consumer myTest $KAFKABROKERS
    scp ./target/kafka-producer-consumer*.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.net:kafka-producer-consumer.jar
    ```

    Отобразятся считанные записи, а также их количество.

1. Нажмите клавиши __Ctrl+C__, чтобы закрыть потребитель.

### <a name="multiple-consumers"></a>Несколько потребителей

При считывании записей объект-получатель Kafka использует группу объектов-получателей. Использование одной группы с несколькими потребителями приведет к считыванию записей с балансировкой нагрузки из раздела. Каждый потребитель в группе получает часть записей.

Приложение-потребитель принимает параметр, который используется как идентификатор группы. Например, следующая команда предназначена для запуска нового потребителя с идентификатором группы `myGroup`.

```bash
java -jar kafka-producer-consumer.jar consumer myTest $KAFKABROKERS myGroup
```

Нажмите клавиши __Ctrl+C__, чтобы закрыть потребитель.

Чтобы увидеть этот процесс в действии, выполните следующую команду:

```bash
tmux new-session 'java -jar kafka-producer-consumer.jar consumer myTest $KAFKABROKERS myGroup' \
\; split-window -h 'java -jar kafka-producer-consumer.jar consumer myTest $KAFKABROKERS myGroup' \
\; attach
```

Эта команда использует `tmux`, чтобы разделить терминал на два столбца. Потребитель запускается в каждом столбце с тем же значением идентификатора группы. Когда потребители закончат чтение, обратите внимание, что каждый из них считал только часть записей. Дважды нажмите клавиши __CTLR+C__, чтобы выйти из `tmux`.

Потребление по клиентам, входящих в одну группу, обрабатывается по секциям раздела. В этом примере кода созданный ранее раздел `test` имеет восемь секций. Если запущено восемь потребителей, каждый из них считывает записи из одной секции раздела.

> [!IMPORTANT]  
> Число экземпляров потребителя, входящих в группу потребителей, должно соответствовать числу секций. В этом примере одна группа объектов-получателей может содержать не больше восьми объектов-получателей, так как это число секций в разделе. Но у вас может быть несколько групп объектов-получателей, в каждую из которых входит не более восьми из них.

Записи хранятся в Kafka в том порядке, в котором они поступают в секцию. Чтобы обеспечить упорядоченную доставку записей *в секцию*, нужно создать группу потребителей. При этом число входящих в нее экземпляров потребителя должно соответствовать числу секций. Чтобы обеспечить упорядоченную доставку записей *в раздел*, нужно создать группу потребителей, которая содержит только один экземпляр потребителя.

## <a name="common-issues-faced"></a>Распространенные проблемы

1. **Сбой при создании раздела.** Если в кластере включен корпоративный пакет безопасности, используйте [предварительно созданные JAR-файлы для производителя и потребителя](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/blob/master/Prebuilt-Jars/kafka-producer-consumer-esp.jar). JAR-файл с корпоративным пакетом безопасности можно создать с помощью кода в [подкаталоге `DomainJoined-Producer-Consumer`](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/tree/master/DomainJoined-Producer-Consumer). В свойствах производителя и потребителя предусмотрено дополнительное свойство `CommonClientConfigs.SECURITY_PROTOCOL_CONFIG` для кластеров с корпоративным пакетом безопасности.

2. **Проблемы с кластерами с корпоративным пакетом безопасности.** Если произошел сбой в операциях создания и потребления и вы используете кластер с корпоративным пакетом безопасности, убедитесь, что пользователь `kafka` присутствует во всех политиках Ranger. Если нет, добавьте его во все политики Ranger.

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы очистить ресурсы, созданные при работе с этим руководством, удалите группу ресурсов. При этом будет удален связанный кластер HDInsight и другие ресурсы, связанные с этой группой ресурсов.

Чтобы удалить группу ресурсов с помощью портала Azure, сделайте следующее:

1. На портале Azure разверните меню слева, чтобы открыть меню служб, а затем выберите __Группы ресурсов__, чтобы просмотреть список групп ресурсов.
2. Найдите группу ресурсов, которую нужно удалить, и щелкните правой кнопкой мыши кнопку __Дополнительно__ (…) справа от списка.
3. Выберите __Удалить группу ресурсов__ и подтвердите выбор.

## <a name="next-steps"></a>Дальнейшие действия

Из этого документа вы узнали, как использовать API производителя и потребителя Apache Kafka для Kafka в HDInsight. Дополнительные сведения о работе с Kafka см. в следующих материалах.

* [Использование прокси-сервера REST для Kafka](rest-proxy.md)
* [Анализ журналов для Apache Kafka в HDInsight](apache-kafka-log-analytics-operations-management.md)