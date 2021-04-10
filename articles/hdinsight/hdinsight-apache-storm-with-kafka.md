---
title: Руководство по Apache Storm с Apache Kafka — Azure HDInsight
description: Узнайте, как создать конвейер потоковой передачи с помощью Apache Storm и Apache Kafka в HDInsight. В этом руководстве используются компоненты KafkaBolt и KafkaSpout, чтобы выполнять потоковую передачу данных из Kafka.
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: tutorial
ms.date: 06/25/2019
ms.openlocfilehash: 2078ba177d68cfaa0a91b79611d92e5c68e4e245
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104868425"
---
# <a name="tutorial-use-apache-storm-with-apache-kafka-on-hdinsight"></a>Руководство по Использование Apache Storm с Apache Kafka в HDInsight

В этом руководстве демонстрируется, как использовать топологию [Apache Storm](https://storm.apache.org/), чтобы считывать и записывать данные с использованием [Apache Kafka](https://kafka.apache.org/) в HDInsight. Здесь также показано, как сохранить данные в совместимое с [Apache Hadoop HDFS](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html) хранилище в кластере Storm.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Использование Storm и Kafka
> * Основные сведения о коде
> * Создание кластеров Kafka и Storm
> * Сборка топологии
> * Настройка топологии
> * Создание раздела Kafka
> * Запуск топологий
> * Остановка топологий
> * Очистка ресурсов

## <a name="prerequisites"></a>Предварительные требования

* Опыт создания разделов Kafka. Дополнительные сведения см. в документе [Приступая к работе с Apache Kafka в HDInsight](./kafka/apache-kafka-get-started.md).

* Опыт создания и развертывания решений Storm (топологий). В частности топологий, в которых используется платформа [Apache Storm Flux](https://storm.apache.org/releases/current/flux.html). Дополнительные сведения см. в статье [Создание топологии Apache Storm на языке Java](./storm/apache-storm-develop-java-topology.md).

* Пакет [Java JDK](https://www.oracle.com/technetwork/pt/java/javase/downloads/jdk8-downloads-2133151.html) 1.8 или более поздней версии. Для HDInsight 3.5 или более поздней версии требуется Java 8.

* [Maven 3.x](https://maven.apache.org/download.cgi)

* SSH-клиент (требуются команды `ssh` и `scp`). Дополнительные сведения см. в разделе [Подключение к HDInsight (Hadoop) с помощью SSH](hdinsight-hadoop-linux-use-ssh-unix.md).

Во время установки Java и JDK на компьютере, где ведется разработка, могут быть установлены следующие переменные среды. Однако следует убедиться, что они существуют и что они содержат правильные значения для вашей системы.

* `JAVA_HOME`. Эта переменная должна указывать на каталог, в который установлен JDK.
* `PATH`. Эта переменная должна содержать следующие пути:
  
    * `JAVA_HOME` или эквивалентный путь.
    * `JAVA_HOME\bin` или эквивалентный путь.
    * Каталог, в который установлено ПО Maven.

> [!IMPORTANT]  
> С помощью описанных здесь шагов будет создана группа ресурсов Azure, которая содержит кластеры Storm в HDInsight и Kafka в HDInsight. Оба этих кластера находятся в виртуальной сети Azure, что позволяет кластеру Storm напрямую обмениваться данными с кластером Kafka.
> 
> Для удобства в этом документе есть ссылка на шаблон, с помощью которого можно создать все необходимые ресурсы Azure. 
>
> Дополнительные сведения об использовании HDInsight в виртуальной сети см. в статье о[планирование виртуальной сети для Azure HDInsight](hdinsight-plan-virtual-network-deployment.md).

## <a name="storm-and-kafka"></a>Использование Storm и Kafka

Apache Storm предоставляет несколько компонентов для работы с Apache Kafka. В этом руководстве используются следующие компоненты:

* `org.apache.storm.kafka.KafkaSpout`: этот компонент считывает данные из Kafka. Он использует следующие компоненты:

    * `org.apache.storm.kafka.SpoutConfig`: обеспечивает конфигурацию для компонента spout.

    * `org.apache.storm.spout.SchemeAsMultiScheme` и `org.apache.storm.kafka.StringScheme`: способ преобразования данных из Kafka в кортеж Storm.

* `org.apache.storm.kafka.bolt.KafkaBolt`: этот компонент записывает данные в Kafka. Он использует следующие компоненты:

    * `org.apache.storm.kafka.bolt.selector.DefaultTopicSelector`: описывает раздел, в который ведется запись;

    * `org.apache.kafka.common.serialization.StringSerializer`: настраивает объект bolt для сериализации данных в виде строкового значения;

    * `org.apache.storm.kafka.bolt.mapper.FieldNameBasedTupleToKafkaMapper`: сопоставляет структуру данных кортежа, используемую в топологии Storm, с полями, хранящимися в Kafka.

Эти компоненты доступны в пакете `org.apache.storm : storm-kafka`. Используйте версию пакета, которая соответствует версии Storm. Для HDInsight 3.6 используется Storm версии 1.1.0.
Вам также необходим пакет `org.apache.kafka : kafka_2.10`, который содержит дополнительные компоненты Kafka. Используйте версию пакета, которая соответствует версии Kafka. Для HDInsight 3.6 используется Kafka версии 1.1.1.

Следующий XML-код является объявлением зависимости в `pom.xml` для проекта [Apache Maven](https://maven.apache.org/):

```xml
<!-- Storm components for talking to Kafka -->
<dependency>
    <groupId>org.apache.storm</groupId>
    <artifactId>storm-kafka</artifactId>
    <version>1.1.0</version>
</dependency>
<!-- needs to be the same Kafka version as used on your cluster -->
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka_2.10</artifactId>
    <version>1.1.1</version>
    <!-- Exclude components that are loaded from the Storm cluster at runtime -->
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
        <exclusion>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
        </exclusion>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

## <a name="understanding-the-code"></a>Основные сведения о коде

Используемый в этом документе код см. по адресу [https://github.com/Azure-Samples/hdinsight-storm-java-kafka](https://github.com/Azure-Samples/hdinsight-storm-java-kafka).

В этом руководстве представлено два варианта топологий:

* Kafka-writer — создает случайные предложения и сохраняет их в Kafka.

* Kafka-reader — считывает данные из Kafka и сохраняет их в совместимое с HDFS хранилище файлов для кластера Storm.

    > [!WARNING]  
    > Чтобы настроить Storm для работы с совместимым с HDFS хранилищем, используемым HDInsight, требуется действие скрипта. Скрипт устанавливает несколько JAR-файлов в пути `extlib` для Storm. Шаблон в этом руководстве автоматически использует этот скрипт во время создания кластера.
    >
    > Если в этом документе не используется шаблон для создания кластера Storm, тогда вам необходимо вручную применить действие скрипта для вашего кластера.
    >
    > Действие скрипта расположено по адресу [https://hdiconfigactions.blob.core.windows.net/linuxstormextlibv01/stormextlib.sh](https://hdiconfigactions.blob.core.windows.net/linuxstormextlibv01/stormextlib.sh). Оно применяется к узлам супервизора и Nimbus кластера Storm. Дополнительные сведения об использовании действий скрипта см. в статье [Настройка кластеров HDInsight под управлением Linux с помощью действий сценариев](hdinsight-hadoop-customize-cluster-linux.md).

Топологии определяются с помощью [Flux](https://storm.apache.org/releases/current/flux.html). Решение Flux, которое впервые было представлено в версии Storm 0.10.x, позволяет отделить конфигурацию топологии от кода. Для топологий, которые используют платформу Flux, топология определяется в YAML-файле. YAML-файл можно включить в топологию или указать его как отдельный файл при отправке топологии. Flux также поддерживает подстановку переменных во время выполнения, которая используется в этом примере.

Во время выполнения этих топологий задаются следующие параметры:

* `${kafka.topic}`: имя раздела Kafka, в котором топология выполняет чтение и запись.

* `${kafka.broker.hosts}`: узлы, на которых работают брокеры Kafka. Сито KafkaBolt использует сведения о брокерах при записи в Kafka.

* `${kafka.zookeeper.hosts}`: узлы кластера Kafka, на которых работает Zookeeper.

* `${hdfs.url}`: URL-адрес файловой системы компонента HDFSBolt. Указывает, куда записываются данные: в учетную запись службы хранилища Azure или в Azure Data Lake Storage.

* `${hdfs.write.dir}`: каталог, в который записываются данные.

Дополнительные сведения о топологиях Flux см. по адресу [https://storm.apache.org/releases/current/flux.html](https://storm.apache.org/releases/current/flux.html).

### <a name="kafka-writer"></a>Kafka-writer

В топологии Kafka-writer компонент Kafka bolt принимает два строковых значения в качестве параметров. Эти параметры указывают, какие поля кортежей объект bolt отправляет в Kafka в виде значений __ключа__ и __сообщения__. Ключ используется для секционирования данных в Kafka. Сообщение — это сохраняемые данные.

В этом примере компонент `com.microsoft.example.SentenceSpout` создает кортеж, который содержит два поля: `key` и `message`. Объект bolt Kafka извлекает эти поля и отправляет данные из них в Kafka.

Полям необязательно использовать имена `key` и `message`. Эти имена используются в этом проекте, чтобы упростить понимание сопоставления.

Следующий YAML является определением для компонента Kafka-reader:

```yaml
# kafka-writer
---

# topology definition
# name to be used when submitting
name: "kafka-writer"

# Components - constructors, property setters, and builder arguments.
# Currently, components must be declared in the order they are referenced
components:
  # Topic selector for KafkaBolt
  - id: "topicSelector"
    className: "org.apache.storm.kafka.bolt.selector.DefaultTopicSelector"
    constructorArgs:
      - "${kafka.topic}"

  # Mapper for KafkaBolt
  - id: "kafkaMapper"
    className: "org.apache.storm.kafka.bolt.mapper.FieldNameBasedTupleToKafkaMapper"
    constructorArgs:
      - "key"
      - "message"

  # Producer properties for KafkaBolt
  - id: "producerProperties"
    className: "java.util.Properties"
    configMethods:
      - name: "put"
        args:
          - "bootstrap.servers"
          - "${kafka.broker.hosts}"
      - name: "put"
        args:
          - "acks"
          - "1"
      - name: "put"
        args:
          - "key.serializer"
          - "org.apache.kafka.common.serialization.StringSerializer"
      - name: "put"
        args:
          - "value.serializer"
          - "org.apache.kafka.common.serialization.StringSerializer"
 

# Topology configuration
config:
  topology.workers: 2

# Spout definitions
spouts:
  - id: "sentence-spout"
    className: "com.microsoft.example.SentenceSpout"
    parallelism: 8

# Bolt definitions
bolts:
  - id: "kafka-bolt"
    className: "org.apache.storm.kafka.bolt.KafkaBolt"
    parallelism: 8
    configMethods:
    - name: "withProducerProperties"
      args: [ref: "producerProperties"]
    - name: "withTopicSelector"
      args: [ref: "topicSelector"]
    - name: "withTupleToKafkaMapper"
      args: [ref: "kafkaMapper"]

# Stream definitions

streams:
  - name: "spout --> kafka" # Streams data from the sentence spout to the Kafka bolt
    from: "sentence-spout"
    to: "kafka-bolt"
    grouping:
      type: SHUFFLE
```

### <a name="kafka-reader"></a>Kafka-reader

В топологии Kafka-reader компонент spout считывает данные из Kafka как строковые значения. Затем данные записываются в журнал Storm с помощью компонента ведения журнала и в совместимую с HDFS файловую систему для кластера Storm с помощью компонента bolt HDFS.

```yaml
# kafka-reader
---

# topology definition
# name to be used when submitting
name: "kafka-reader"

# Components - constructors, property setters, and builder arguments.
# Currently, components must be declared in the order they are referenced
components:
  # Convert data from Kafka into string tuples in storm
  - id: "stringScheme"
    className: "org.apache.storm.kafka.StringScheme"
  - id: "stringMultiScheme"
    className: "org.apache.storm.spout.SchemeAsMultiScheme"
    constructorArgs:
      - ref: "stringScheme"

  - id: "zkHosts"
    className: "org.apache.storm.kafka.ZkHosts"
    constructorArgs:
      - "${kafka.zookeeper.hosts}"

  # Spout configuration
  - id: "spoutConfig"
    className: "org.apache.storm.kafka.SpoutConfig"
    constructorArgs:
      # brokerHosts
      - ref: "zkHosts"
      # topic
      - "${kafka.topic}"
      # zkRoot
      - ""
      # id
      - "readerid"
    properties:
      - name: "scheme"
        ref: "stringMultiScheme"

    # How often to sync files to HDFS; every 1000 tuples.
  - id: "syncPolicy"
    className: "org.apache.storm.hdfs.bolt.sync.CountSyncPolicy"
    constructorArgs:
      - 1

  # Rotate files when they hit 5 MB
  - id: "rotationPolicy"
    className: "org.apache.storm.hdfs.bolt.rotation.FileSizeRotationPolicy"
    constructorArgs:
      - 5
      - "KB"

  # File format; read the directory from filters at run time, and use a .txt extension when writing.
  - id: "fileNameFormat"
    className: "org.apache.storm.hdfs.bolt.format.DefaultFileNameFormat"
    configMethods:
      - name: "withPath"
        args: ["${hdfs.write.dir}"]
      - name: "withExtension"
        args: [".txt"]

  # Internal file format; fields delimited by `|`.
  - id: "recordFormat"
    className: "org.apache.storm.hdfs.bolt.format.DelimitedRecordFormat"
    configMethods:
      - name: "withFieldDelimiter"
        args: ["|"]

# Topology configuration
config:
  topology.workers: 2

# Spout definitions
spouts:
  - id: "kafka-spout"
    className: "org.apache.storm.kafka.KafkaSpout"
    constructorArgs:
      - ref: "spoutConfig"
    # Set to the number of partitions for the topic
    parallelism: 8

# Bolt definitions
bolts:
  - id: "logger-bolt"
    className: "com.microsoft.example.LoggerBolt"
    parallelism: 1
  
  - id: "hdfs-bolt"
    className: "org.apache.storm.hdfs.bolt.HdfsBolt"
    configMethods:
      - name: "withConfigKey"
        args: ["hdfs.config"]
      - name: "withFsUrl"
        args: ["${hdfs.url}"]
      - name: "withFileNameFormat"
        args: [ref: "fileNameFormat"]
      - name: "withRecordFormat"
        args: [ref: "recordFormat"]
      - name: "withRotationPolicy"
        args: [ref: "rotationPolicy"]
      - name: "withSyncPolicy"
        args: [ref: "syncPolicy"]
    parallelism: 1

# Stream definitions

streams:
  # Stream data to log
  - name: "kafka --> log" # name isn't used (placeholder for logging, UI, etc.)
    from: "kafka-spout"
    to: "logger-bolt"
    grouping:
      type: SHUFFLE
  
  # stream data to file
  - name: "kafka --> hdfs"
    from: "kafka-spout"
    to: "hdfs-bolt"
    grouping:
      type: SHUFFLE
```

### <a name="property-substitutions"></a>Подстановки свойств

Проект содержит файл с именем `dev.properties`, в котором передаются параметры, используемые в топологиях. Он определяет следующие свойства:

| Файл dev.properties | Описание |
| --- | --- |
| `kafka.zookeeper.hosts` | Узлы [Apache ZooKeeper](https://zookeeper.apache.org/) для кластера Kafka. |
| `kafka.broker.hosts` | Узлы брокера Kafka (рабочие узлы). |
| `kafka.topic` | Раздел Kafka, используемый топологиями. |
| `hdfs.write.dir` | Каталог, в который топология Kafka-reader выполняет запись. |
| `hdfs.url` | Файловая система, используемая кластером Storm. В учетных записях службы хранилища Azure используйте значение `wasb://`. В учетных записях Azure Data Lake Storage 2-го поколения используйте значение `abfs://`. В учетных записях Azure Data Lake Storage 1-го поколения используйте значение `adl://`. |

## <a name="create-the-clusters"></a>Создание кластеров

Apache Kafka в HDInsight не предоставляет доступ к брокерам Kafka через общедоступный сегмент Интернета. Все ресурсы, которые использует Kafka, должны находиться в одной виртуальной сети Azure. В рамках этого руководства кластеры Kafka и Storm расположены в одной и той же виртуальной сети Azure. 

На следующей схеме показано, как происходит обмен данными между Storm и Kafka:

:::image type="content" source="./media/hdinsight-apache-storm-with-kafka/apache-storm-kafka-vnet.png" alt-text="Схема кластеров Storm и Kafka в виртуальной сети Azure":::

> [!NOTE]  
> Другие службы в кластере, например SSH и [Apache Ambari](https://ambari.apache.org/), могут быть доступны через Интернет. Дополнительные сведения об общих портах, доступных в HDInsight, см. в статье [Порты и универсальные коды ресурсов (URI), используемые кластерами HDInsight](hdinsight-hadoop-port-settings-for-services.md).

Чтобы создать виртуальную сеть Azure, а затем создать кластеры Kafka и Storm в ее пределах, выполните такие действия:

1. Нажмите эту кнопку, чтобы войти в Azure и открыть шаблон на портале Azure.

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fhdinsight-storm-java-kafka%2Fmaster%2Fcreate-kafka-storm-clusters-in-vnet.json" target="_blank"><img src="./media/hdinsight-apache-storm-with-kafka/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>

    Шаблон Azure Resource Manager доступен по адресу **https://github.com/Azure-Samples/hdinsight-storm-java-kafka/blob/master/create-kafka-storm-clusters-in-vnet.json** . Он создает перечисленные ниже ресурсы.

    * Группа ресурсов Azure
    * Виртуальная сеть Azure
    * Учетная запись хранения Azure
    * Kafka в HDInsight версии 3.6 (три рабочих узла)
    * Storm в HDInsight версии 3.6 (три рабочих узла)

   > [!WARNING]  
   > Чтобы обеспечить доступность Kafka в HDInsight, кластер должен содержать не менее трех рабочих узлов. Этот шаблон создает кластер Kafka, содержащий три рабочих узла.

2. Используйте приведенные ниже инструкции для заполнения раздела **Настраиваемое развертывание**.

   1. Заполните раздел **Настроенный шаблон**, используя следующие сведения:

      | Параметр | Значение |
      | --- | --- |
      | Подписка | Ваша подписка Azure. |
      | Группа ресурсов | Группа ресурсов, в которой содержатся ресурсы. |
      | Расположение | Регион Azure, в котором создаются ресурсы. |
      | Kafka Cluster Name (Имя кластера Kafka) | Имя кластера Kafka. |
      | Имя кластера Storm | Имя кластера Storm. |
      | Имя пользователя для входа в кластер | Имя администратора кластеров. |
      | Пароль для входа в кластер | Пароль администратора кластеров. |
      | Имя пользователя SSH | Пользователь SSH, который создается для кластеров. |
      | Пароль SSH | Пароль пользователя SSH. |
   
      :::image type="content" source="./media/hdinsight-apache-storm-with-kafka/storm-kafka-template.png" alt-text="Рисунок с параметрами шаблона":::

3. Прочтите **условия использования** и установите флажок **Я принимаю указанные выше условия**.

4. Установите флажок **Закрепить на панели мониторинга** и нажмите кнопку **Приобрести**.

> [!NOTE]  
> Создание кластеров может занять до 20 минут.

## <a name="build-the-topology"></a>Сборка топологии

1. В среде разработки скачайте проект по адресу [https://github.com/Azure-Samples/hdinsight-storm-java-kafka](https://github.com/Azure-Samples/hdinsight-storm-java-kafka), откройте окно командной строки и перейдите в папку со скачанным проектом.

2. Из каталога **hdinsight-storm-java-kafka** выполните следующую команду, чтобы скомпилировать проект и создать пакет для развертывания.

   ```bash
   mvn clean package
   ```

    Процесс упаковки создаст файл с именем `KafkaTopology-1.0-SNAPSHOT.jar` в каталоге `target`.

3. Воспользуйтесь приведенными ниже командами, чтобы скопировать пакет в кластер Storm в HDInsight. Замените `sshuser` именем пользователя SSH для кластера. Замените `stormclustername` именем кластера __Storm__.

   ```bash
   scp ./target/KafkaTopology-1.0-SNAPSHOT.jar sshuser@stormclustername-ssh.azurehdinsight.net:KafkaTopology-1.0-SNAPSHOT.jar
   ```

    При появлении запроса введите пароль, который использовался при создании кластеров.

## <a name="configure-the-topology"></a>Настройка топологии

1. Используйте один из следующих методов для обнаружения узлов брокера Kafka для **Kafka** в кластере HDInsight.

    ```powershell
    $creds = Get-Credential -UserName "admin" -Message "Enter the HDInsight login"
    $clusterName = Read-Host -Prompt "Enter the Kafka cluster name"
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/KAFKA/components/KAFKA_BROKER" `
        -Credential $creds `
        -UseBasicParsing
    $respObj = ConvertFrom-Json $resp.Content
    $brokerHosts = $respObj.host_components.HostRoles.host_name[0..1]
    ($brokerHosts -join ":9092,") + ":9092"
    ```

    > [!IMPORTANT]  
    > В следующем примере Bash предполагается, что `$CLUSTERNAME` содержит имя кластера __Kafka__. Также предполагается, что установлен обработчик [jq](https://stedolan.github.io/jq/) 1.5 или более поздней версии. При появлении запроса введите пароль, чтобы войти на кластер.

    ```bash
    curl -su admin -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/KAFKA/components/KAFKA_BROKER" | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2
    ```

    Возвращаемое значение аналогично приведенному ниже тексту.

    ```output
    wn0-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:9092,wn1-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:9092
     ```

    > [!IMPORTANT]  
    > Хотя в кластер может содержать более двух узлов брокера, вам необязательно указывать полный список всех узлов на клиентах. Достаточно указать один или два.

2. Используйте один из следующих методов для обнаружения узлов Zookeeper для __Kafka__ в кластере HDInsight.

    ```powershell
    $creds = Get-Credential -UserName "admin" -Message "Enter the HDInsight login"
    $clusterName = Read-Host -Prompt "Enter the Kafka cluster name"
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/ZOOKEEPER/components/ZOOKEEPER_SERVER" `
        -Credential $creds `
        -UseBasicParsing
    $respObj = ConvertFrom-Json $resp.Content
    $zookeeperHosts = $respObj.host_components.HostRoles.host_name[0..1]
    ($zookeeperHosts -join ":2181,") + ":2181"
    ```

    > [!IMPORTANT]  
    > В следующем примере Bash предполагается, что `$CLUSTERNAME` содержит имя кластера __Kafka__. Также предполагается, что [jq](https://stedolan.github.io/jq/) установлен. При появлении запроса введите пароль, чтобы войти на кластер.

    ```bash
    curl -su admin -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/ZOOKEEPER/components/ZOOKEEPER_SERVER" | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2
    ```

    Возвращаемое значение аналогично приведенному ниже тексту.

    ```output
    zk0-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:2181,zk2-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:2181
    ```

    > [!IMPORTANT]  
    > Хотя может быть более двух узлов Zookeeper, вам необязательно указывать полный список всех узлов на клиентах. Достаточно указать один или два.

    Сохраните это значение, так как оно будет использовано позже.

3. Отредактируйте файл `dev.properties` в корневой папке проекта. Добавьте данные узлов брокера и Zookeeper для кластера __Kafka__ в соответствующие строки в этом файле. В следующем примере настройка выполняется с помощью образцов значений, которые использовались на предыдущих шагах:

    ```bash
    kafka.zookeeper.hosts: zk0-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:2181,zk2-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:2181
    kafka.broker.hosts: wn0-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:9092,wn1-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:9092
    kafka.topic: stormtopic
    ```

    > [!IMPORTANT]  
    > Запись `hdfs.url` настроена для кластера, который использует учетную запись службы хранилища Azure. Чтобы использовать эту топологию с кластером Storm, который использует Data Lake Storage, измените это значение с `wasb` на `adl`.

4. Сохраните файл `dev.properties` и выполните следующую команду, чтобы передать его на кластер **Storm**.

     ```bash
    scp dev.properties USERNAME@BASENAME-ssh.azurehdinsight.net:dev.properties
    ```

    Замените **USERNAME** именем пользователя SSH для кластера. Замените **BASENAME** базовым именем, которое использовалось при создании кластера.

## <a name="create-the-kafka-topic"></a>Создание раздела Kafka

Kafka сохраняет данные в _разделе_. Раздел необходимо создать перед запуском топологий Storm. Для создания топологии выполните следующие действия:

1. Подключитесь к кластеру __Kafka__ через SSH с помощью следующей команды. Замените `sshuser` именем пользователя SSH, которое использовалось при создании кластера. Замените `kafkaclustername` именем кластера Kafka:

    ```bash
    ssh sshuser@kafkaclustername-ssh.azurehdinsight.net
    ```

    При появлении запроса введите пароль, который использовался при создании кластеров.
   
    См. дополнительные сведения об [использовании SSH в HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

2. Чтобы создать раздел Kafka, используйте следующую команду. Замените `$KAFKAZKHOSTS` информацией об узле Zookeeper, используемой при настройке топологии:

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic stormtopic --zookeeper $KAFKAZKHOSTS
    ```

    Эта команда подключается к Zookeeper для кластера Kafka и создает раздел под названием `stormtopic`. Этот раздел используется топологиями Storm.

## <a name="start-the-writer"></a>Запуск модуля записи

1. Используйте указанную ниже команду для подключения к кластеру **Storm** с помощью SSH. Замените `sshuser` именем пользователя SSH, которое использовалось при создании кластера. Замените `stormclustername` именем кластера Storm:

    ```bash
    ssh sshuser@stormclustername-ssh.azurehdinsight.net
    ```

    При появлении запроса введите пароль, который использовался при создании кластеров.
   
    См. дополнительные сведения об [использовании SSH в HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

2. Чтобы запустить топологию модуля записи, выполните следующие команды из SSH-подключения к кластеру Storm.

    ```bash
    storm jar KafkaTopology-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --remote -R /writer.yaml --filter dev.properties
    ```

    Параметры, используемые в этой команде:

    * `org.apache.storm.flux.Flux`: Для настройки и запуска этой топологии следует использовать Flux.

    * `--remote`: Отправьте топологию в Nimbus. Топология распределяется между рабочими узлами в кластере.

    * `-R /writer.yaml`: Используйте файл `writer.yaml` для настройки топологии. `-R` указывает, что ресурс включен в JAR-файл. Он включается в корень JAR-файла, поэтому путь к нему выглядит так: `/writer.yaml`.

    * `--filter`: Заполните записи в топологии `writer.yaml`, используя значения из файла `dev.properties`. Например, значение записи `kafka.topic` в файле используется для замены записи `${kafka.topic}` в определении топологии.

## <a name="start-the-reader"></a>Запуск модуля чтения

1. Чтобы запустить топологию модуля чтения, выполните следующие команды из SSH-подключения к кластеру Storm.

   ```bash
   storm jar KafkaTopology-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --remote -R /reader.yaml --filter dev.properties
   ```

2. Подождите минуту, а затем используйте следующую команду для просмотра файлов, созданных топологией чтения:

    ```bash
    hdfs dfs -ls /stormdata
    ```

    Результат будет аналогичен приведенному ниже:

    ```output
    Found 173 items
      -rw-r--r--   1 storm supergroup       5137 2018-04-09 19:00 /stormdata/hdfs-bolt-4-0-1523300453088.txt
      -rw-r--r--   1 storm supergroup       5128 2018-04-09 19:00 /stormdata/hdfs-bolt-4-1-1523300453624.txt
      -rw-r--r--   1 storm supergroup       5131 2018-04-09 19:00 /stormdata/hdfs-bolt-4-10-1523300455170.txt
      ...
    ```

3. Чтобы просмотреть содержимое файла, выполнив следующую команду. Замените `filename.txt` на имя файла:

    ```bash
    hdfs dfs -cat /stormdata/filename.txt
    ```

    Ниже приведен пример содержимого файла:

    > four score and seven years ago (минуло восемьдесят семь лет)
    >
    > snow white and the seven dwarfs (Белоснежка и семь гномов)
    >
    > i am at two with nature (я в дисгармонии с природой)
    >
    > snow white and the seven dwarfs (Белоснежка и семь гномов)
    >
    > i am at two with nature (я в дисгармонии с природой)
    >
    > four score and seven years ago (минуло восемьдесят семь лет)
    >
    > an apple a day keeps the doctor away (кто яблоко в день съедает, у того врач не бывает)

## <a name="stop-the-topologies"></a>Остановка топологий

Чтобы остановить топологии Storm, выполните следующие команды из SSH-подключения к кластеру Storm.

  ```bash
  storm kill kafka-writer
  storm kill kafka-reader
  ```

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы очистить ресурсы, созданные при работе с этим руководством, удалите группу ресурсов. При этом будет удален связанный кластер HDInsight и другие ресурсы, связанные с этой группой ресурсов.

Чтобы удалить группу ресурсов с помощью портала Azure, сделайте следующее:

1. На портале Azure разверните меню слева, чтобы открыть меню служб, а затем выберите __Группы ресурсов__, чтобы просмотреть список групп ресурсов.
2. Найдите группу ресурсов, которую нужно удалить, и щелкните правой кнопкой мыши кнопку __Дополнительно__ (…) справа от списка.
3. Выберите __Удалить группу ресурсов__ и подтвердите выбор.

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как использовать топологию [Apache Storm](https://storm.apache.org/) для записи в систему [Apache Kafka](https://kafka.apache.org/) и считывании из нее в HDInsight. Вы также узнали, как хранить данные в совместимом с [Apache Hadoop HDFS](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html) хранилище, используемом в HDInsight.

> [!div class="nextstepaction"]
> [Руководство. Использование API производителя и потребителя Apache Kafka](kafka/apache-kafka-producer-consumer-api.md)
