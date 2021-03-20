---
title: Пример топологии Java для Apache Storm в Azure HDInsight
description: Узнайте, как создать топологии Apache Storm на языке Java с помощью создания примера топологии подсчета слов.
ms.service: hdinsight
ms.topic: how-to
ms.custom: H1Hack27Feb2017,hdinsightactive,hdiseo17may2017,seoapr2020,devx-track-java
ms.date: 04/27/2020
ms.openlocfilehash: 620a4e1627b25af22db68173f35924376e26f5f8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98929129"
---
# <a name="create-an-apache-storm-topology-in-java"></a>Создание топологии Apache Storm на языке Java

Узнайте, как создать топологию на основе Java для Apache Storm. Вы создадите топологию Storm, реализующую приложение подсчета слов. Для сборки и упаковки проекта вы будете использовать Apache Maven. Затем вы узнаете, как определить топологию с помощью платформы Apache Storm Flux.

После выполнения действий, описанных в этом документе, вы сможете развернуть топологию в Apache Storm в HDInsight.

> [!NOTE]  
> Завершенная версия примеров топологии, созданных в этом документе, доступна по адресу [https://github.com/Azure-Samples/hdinsight-java-storm-wordcount](https://github.com/Azure-Samples/hdinsight-java-storm-wordcount) .

## <a name="prerequisites"></a>Предварительные условия

* [Java Developer Kit (JDK) версии 8](/azure/developer/java/fundamentals/java-jdk-long-term-support)

* Средство [Apache Maven](https://maven.apache.org/download.cgi), [установленное](https://maven.apache.org/install.html) согласно инструкций Apache.  Maven — система сборки проектов Java.

## <a name="test-environment"></a>Тестовая среда

Среда, используемая для этой статьи, была компьютер под Windows 10.  Команды были выполнены в командной строке, а различные файлы были изменены с помощью блокнота.

В командной строке введите приведенные ниже команды, чтобы создать рабочую среду.

```cmd
mkdir C:\HDI
cd C:\HDI
```

## <a name="create-a-maven-project"></a>Создание проекта Maven

Введите следующую команду, чтобы создать проект Maven с именем **WordCount**:

```cmd
mvn archetype:generate -DarchetypeArtifactId=maven-archetype-quickstart -DgroupId=com.microsoft.example -DartifactId=WordCount -DinteractiveMode=false

cd WordCount
mkdir resources
```

Эта команда создает каталог с именем `WordCount` в текущем расположении, содержащий базовый проект Maven. Вторая команда изменяет текущий рабочий каталог на `WordCount` . Третья команда создает новый каталог, `resources` который будет использоваться позже.  Каталог `WordCount` содержит следующие элементы:

* `pom.xml` содержит параметры для проекта Maven;
* `src\main\java\com\microsoft\example` содержит код приложения;
* `src\test\java\com\microsoft\example` содержит тесты для приложения.  

### <a name="remove-the-generated-example-code"></a>Удаление созданного примера кода

Удалите созданные файлы тестов и приложений `AppTest.java` , а затем `App.java` введите приведенные ниже команды.

```cmd
DEL src\main\java\com\microsoft\example\App.java
DEL src\test\java\com\microsoft\example\AppTest.java
```

## <a name="add-maven-repositories"></a>Добавление репозиториев Maven

Решение HDInsight основано на платформе данных Hortonworks Data Platform (HDP), поэтому мы рекомендуем использовать репозиторий Hortonworks для скачивания зависимостей для проектов Apache Storm.  

Откройте `pom.xml` , введя следующую команду:

```cmd
notepad pom.xml
```

Затем добавьте следующий код XML после `<url>https://maven.apache.org</url>` строки:

```xml
<repositories>
    <repository>
        <releases>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
            <checksumPolicy>warn</checksumPolicy>
        </releases>
        <snapshots>
            <enabled>false</enabled>
            <updatePolicy>never</updatePolicy>
            <checksumPolicy>fail</checksumPolicy>
        </snapshots>
        <id>HDPReleases</id>
        <name>HDP Releases</name>
        <url>https://repo.hortonworks.com/content/repositories/releases/</url>
        <layout>default</layout>
    </repository>
    <repository>
        <releases>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
            <checksumPolicy>warn</checksumPolicy>
        </releases>
        <snapshots>
            <enabled>false</enabled>
            <updatePolicy>never</updatePolicy>
            <checksumPolicy>fail</checksumPolicy>
        </snapshots>
        <id>HDPJetty</id>
        <name>Hadoop Jetty</name>
        <url>https://repo.hortonworks.com/content/repositories/jetty-hadoop/</url>
        <layout>default</layout>
    </repository>
</repositories>
```

## <a name="add-properties"></a>Добавление свойств

Maven позволяет определить значения на уровне проекта, которые называются свойствами. В `pom.xml` добавьте следующий текст после `</repositories>` строки:

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <!--
    This is a version of Storm from the Hortonworks repository that is compatible with HDInsight 3.6.
    -->
    <storm.version>1.1.0.2.6.1.9-1</storm.version>
</properties>
```

Теперь это значение можно использовать в других разделах `pom.xml`. Например, при указании версии компонентов Storm, можно использовать `${storm.version}` вместо жестко запрограммированного значения.

## <a name="add-dependencies"></a>Добавление зависимостей

Добавьте зависимость для компонентов Storm. В `pom.xml` добавьте в раздел следующий текст `<dependencies>` :

```xml
<dependency>
    <groupId>org.apache.storm</groupId>
    <artifactId>storm-core</artifactId>
    <version>${storm.version}</version>
    <!-- keep storm out of the jar-with-dependencies -->
    <scope>provided</scope>
</dependency>
```

Во время компиляции Maven использует эту информацию для поиска `storm-core` в репозитории Maven. Сначала выполняется поиск в репозитории на локальном компьютере. Если файлы отсутствуют, Maven скачает их из общедоступного репозитория Maven и сохранит в локальном репозитории.

> [!NOTE]  
> Обратите внимание на строку `<scope>provided</scope>` в этом разделе. Она указывает Maven, что **storm-core** нужно исключить из всех созданных JAR-файлов, так как этот элемент предоставит система.

## <a name="build-configuration"></a>Конфигурация построения

Подключаемые модули Maven позволяют настроить этапы сборки проекта. Например, способ компиляции проекта или его упаковки в JAR-файл. В `pom.xml` добавьте следующий текст непосредственно над `</project>` строкой.

```xml
<build>
    <plugins>
    </plugins>
    <resources>
    </resources>
</build>
```

Этот раздел используется для добавления подключаемых модулей, ресурсов и других параметров конфигурации сборки. Полную ссылку на `pom.xml` файл см. в разделе [https://maven.apache.org/pom.html](https://maven.apache.org/pom.html) .

### <a name="add-plug-ins"></a>Добавление подключаемых модулей

* **Подключаемый модуль Exec Maven**

    Для топологий Apache Storm, реализованных на языке Java, удобен [подключаемый модуль Exec Maven](https://www.mojohaus.org/exec-maven-plugin/), так как он позволяет легко запускать топологию локально в среде разработки. Добавьте следующий код в раздел `<plugins>` файла `pom.xml`, чтобы включить в него подключаемый модуль Exec Maven.

    ```xml
    <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>exec-maven-plugin</artifactId>
        <version>1.6.0</version>
        <executions>
            <execution>
                <goals>
                    <goal>exec</goal>
                </goals>
            </execution>
        </executions>
        <configuration>
            <executable>java</executable>
            <includeProjectDependencies>true</includeProjectDependencies>
            <includePluginDependencies>false</includePluginDependencies>
            <classpathScope>compile</classpathScope>
            <mainClass>${storm.topology}</mainClass>
            <cleanupDaemonThreads>false</cleanupDaemonThreads>
        </configuration>
    </plugin>
    ```

* **Подключаемый модуль компилятора Apache Maven**

    Еще один полезный подключаемый модуль — [`Apache Maven Compiler Plugin`](https://maven.apache.org/plugins/maven-compiler-plugin/) , который используется для изменения параметров компиляции. Измените версию Java, используемую Maven для исходного и целевого приложения.

  * Для HDInsight __3.4 или более ранней версии__ задайте версию __1.7__ в качестве исходной и целевой версий Java.

  * Для HDInsight __3.5__ в качестве исходной и целевой версий Java задайте версию __1.8__.

  Добавьте следующий код в раздел `<plugins>` файла `pom.xml`, чтобы добавить в него подключаемый модуль Apache Maven Compiler. Этот пример задает версию 1.8, поэтому целевой версией HDInsight является 3.5.

  ```xml
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.1</version>
    <configuration>
            <source>1.8</source>
            <target>1.8</target>
    </configuration>
  </plugin>
  ```

### <a name="configure-resources"></a>Настройка ресурсов

Раздел ресурсов позволяет включать не связанные с кодом ресурсы, такие как файлы конфигурации, необходимые для компонентов топологии. В этом примере добавьте следующий текст в `<resources>` раздел `pom.xml` файла. Сохраните и закройте файл.

```xml
<resource>
    <directory>${basedir}/resources</directory>
    <filtering>false</filtering>
    <includes>
            <include>log4j2.xml</include>
    </includes>
</resource>
```

При этом в корневую папку проекта (`${basedir}`) будет добавлен каталог ресурсов как расположение, содержащее ресурсы и файл `log4j2.xml`. Этот файл используется для настройки того, какая информация записывается топологией.

## <a name="create-the-topology"></a>Создание топологии

Топология Apache Storm на платформе Java состоит из трех компонентов, которые необходимо создать или указать как зависимость.

* **Воронки**— чтение данных из внешних источников и отправка потоков данных в топологию.

* « **Винты**»: обработка потоков, созданных с помощью порождений или других «винтов», и эмиссия одного или нескольких потоков.

* **Топология**— определяет взаимное расположение воронок и сит и предоставляет точку входа для топологии.

### <a name="create-the-spout"></a>Создание «воронки»

Чтобы снизить требования к настройке внешних источников данных, следующая воронка просто выдает случайные предложения. Это измененная версия spout, которая поставляется с примерами для работы с наборами [-начальных](https://github.com/apache/storm/blob/0.10.x-branch/examples/storm-starter/src/jvm/storm/starter)версий.  Хотя в этой топологии используется один spout, другие могут иметь несколько потоков данных из разных источников в топологии.`.`

Введите следующую команду, чтобы создать и открыть новый файл `RandomSentenceSpout.java` :

```cmd
notepad src\main\java\com\microsoft\example\RandomSentenceSpout.java
```

Затем скопируйте и вставьте приведенный ниже код Java в новый файл.  Затем закройте файл.

```java
package com.microsoft.example;

import org.apache.storm.spout.SpoutOutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.topology.base.BaseRichSpout;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Values;
import org.apache.storm.utils.Utils;

import java.util.Map;
import java.util.Random;

//This spout randomly emits sentences
public class RandomSentenceSpout extends BaseRichSpout {
  //Collector used to emit output
  SpoutOutputCollector _collector;
  //Used to generate a random number
  Random _rand;

  //Open is called when an instance of the class is created
  @Override
  public void open(Map conf, TopologyContext context, SpoutOutputCollector collector) {
  //Set the instance collector to the one passed in
    _collector = collector;
    //For randomness
    _rand = new Random();
  }

  //Emit data to the stream
  @Override
  public void nextTuple() {
  //Sleep for a bit
    Utils.sleep(100);
    //The sentences that are randomly emitted
    String[] sentences = new String[]{ "the cow jumped over the moon", "an apple a day keeps the doctor away",
        "four score and seven years ago", "snow white and the seven dwarfs", "i am at two with nature" };
    //Randomly pick a sentence
    String sentence = sentences[_rand.nextInt(sentences.length)];
    //Emit the sentence
    _collector.emit(new Values(sentence));
  }

  //Ack is not implemented since this is a basic example
  @Override
  public void ack(Object id) {
  }

  //Fail is not implemented since this is a basic example
  @Override
  public void fail(Object id) {
  }

  //Declare the output fields. In this case, an sentence
  @Override
  public void declareOutputFields(OutputFieldsDeclarer declarer) {
    declarer.declare(new Fields("sentence"));
  }
}
```

> [!NOTE]  
> Пример воронки, которая считывает информацию из внешнего источника данных, см. в одном из следующих примеров.
>
> * [Твиттерсамплеспаут](https://github.com/apache/storm/blob/0.10.x-branch/examples/storm-starter/src/jvm/storm/starter/spout/TwitterSampleSpout.java). пример spout, считывающий данные из Twitter.
> * [Kafka](https://github.com/apache/storm/tree/0.10.x-branch/external/storm-kafka): spout, считывающий из Kafka.

### <a name="create-the-bolts"></a>Создание «сит»

Сита выполняют обработку данных. Элементы bolt могут выполнять любые операции, например вычисление, сохранение, а также взаимодействие с внешними компонентами. Эта топология включает два сита.

* **SplitSentence** — разделяет предложения, отправленные **RandomSentenceSpout**, на отдельные слова;

* **WordCount**— подсчитывает частоту употребления каждого слова.

#### <a name="splitsentence"></a>SplitSentence

Введите следующую команду, чтобы создать и открыть новый файл `SplitSentence.java` :

```cmd
notepad src\main\java\com\microsoft\example\SplitSentence.java
```

Затем скопируйте и вставьте приведенный ниже код Java в новый файл.  Затем закройте файл.

```java
package com.microsoft.example;

import java.text.BreakIterator;

import org.apache.storm.topology.BasicOutputCollector;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.topology.base.BaseBasicBolt;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Tuple;
import org.apache.storm.tuple.Values;

//There are a variety of bolt types. In this case, use BaseBasicBolt
public class SplitSentence extends BaseBasicBolt {

  //Execute is called to process tuples
  @Override
  public void execute(Tuple tuple, BasicOutputCollector collector) {
    //Get the sentence content from the tuple
    String sentence = tuple.getString(0);
    //An iterator to get each word
    BreakIterator boundary=BreakIterator.getWordInstance();
    //Give the iterator the sentence
    boundary.setText(sentence);
    //Find the beginning first word
    int start=boundary.first();
    //Iterate over each word and emit it to the output stream
    for (int end=boundary.next(); end != BreakIterator.DONE; start=end, end=boundary.next()) {
      //get the word
      String word=sentence.substring(start,end);
      //If a word is whitespace characters, replace it with empty
      word=word.replaceAll("\\s+","");
      //if it's an actual word, emit it
      if (!word.equals("")) {
        collector.emit(new Values(word));
      }
    }
  }

  //Declare that emitted tuples contain a word field
  @Override
  public void declareOutputFields(OutputFieldsDeclarer declarer) {
    declarer.declare(new Fields("word"));
  }
}
```

#### <a name="wordcount"></a>WordCount

Введите следующую команду, чтобы создать и открыть новый файл `WordCount.java` :

```cmd
notepad src\main\java\com\microsoft\example\WordCount.java
```

Затем скопируйте и вставьте приведенный ниже код Java в новый файл.  Затем закройте файл.

```java
package com.microsoft.example;

import java.util.HashMap;
import java.util.Map;
import java.util.Iterator;

import org.apache.storm.Constants;
import org.apache.storm.topology.BasicOutputCollector;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.topology.base.BaseBasicBolt;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Tuple;
import org.apache.storm.tuple.Values;
import org.apache.storm.Config;

// For logging
import org.apache.logging.log4j.Logger;
import org.apache.logging.log4j.LogManager;

//There are a variety of bolt types. In this case, use BaseBasicBolt
public class WordCount extends BaseBasicBolt {
  //Create logger for this class
  private static final Logger logger = LogManager.getLogger(WordCount.class);
  //For holding words and counts
  Map<String, Integer> counts = new HashMap<String, Integer>();
  //How often to emit a count of words
  private Integer emitFrequency;

  // Default constructor
  public WordCount() {
      emitFrequency=5; // Default to 60 seconds
  }

  // Constructor that sets emit frequency
  public WordCount(Integer frequency) {
      emitFrequency=frequency;
  }

  //Configure frequency of tick tuples for this bolt
  //This delivers a 'tick' tuple on a specific interval,
  //which is used to trigger certain actions
  @Override
  public Map<String, Object> getComponentConfiguration() {
      Config conf = new Config();
      conf.put(Config.TOPOLOGY_TICK_TUPLE_FREQ_SECS, emitFrequency);
      return conf;
  }

  //execute is called to process tuples
  @Override
  public void execute(Tuple tuple, BasicOutputCollector collector) {
    //If it's a tick tuple, emit all words and counts
    if(tuple.getSourceComponent().equals(Constants.SYSTEM_COMPONENT_ID)
            && tuple.getSourceStreamId().equals(Constants.SYSTEM_TICK_STREAM_ID)) {
      for(String word : counts.keySet()) {
        Integer count = counts.get(word);
        collector.emit(new Values(word, count));
        logger.info("Emitting a count of " + count + " for word " + word);
      }
    } else {
      //Get the word contents from the tuple
      String word = tuple.getString(0);
      //Have we counted any already?
      Integer count = counts.get(word);
      if (count == null)
        count = 0;
      //Increment the count and store it
      count++;
      counts.put(word, count);
    }
  }

  //Declare that this emits a tuple containing two fields; word and count
  @Override
  public void declareOutputFields(OutputFieldsDeclarer declarer) {
    declarer.declare(new Fields("word", "count"));
  }
}
```

### <a name="define-the-topology"></a>Определение топологии

Топология связывает и объединяет все вместе с графиком. Диаграмма определяет, как потоки данных между компонентами. Она также предоставляет указания по обеспечению параллелизма, которые Storm использует при создании экземпляров компонентов в кластере.

На рисунке ниже приведена базовая диаграмма компонентов этой топологии.

![диаграмма, показывающая упорядочение воронок и сит](./media/apache-storm-develop-java-topology/word-count-topology1.png)

Чтобы реализовать топологию, введите следующую команду, чтобы создать и открыть новый файл `WordCountTopology.java` :

```cmd
notepad src\main\java\com\microsoft\example\WordCountTopology.java
```

Затем скопируйте и вставьте приведенный ниже код Java в новый файл.  Затем закройте файл.

```java
package com.microsoft.example;

import org.apache.storm.Config;
import org.apache.storm.LocalCluster;
import org.apache.storm.StormSubmitter;
import org.apache.storm.topology.TopologyBuilder;
import org.apache.storm.tuple.Fields;

import com.microsoft.example.RandomSentenceSpout;

public class WordCountTopology {

  //Entry point for the topology
  public static void main(String[] args) throws Exception {
  //Used to build the topology
    TopologyBuilder builder = new TopologyBuilder();
    //Add the spout, with a name of 'spout'
    //and parallelism hint of 5 executors
    builder.setSpout("spout", new RandomSentenceSpout(), 5);
    //Add the SplitSentence bolt, with a name of 'split'
    //and parallelism hint of 8 executors
    //shufflegrouping subscribes to the spout, and equally distributes
    //tuples (sentences) across instances of the SplitSentence bolt
    builder.setBolt("split", new SplitSentence(), 8).shuffleGrouping("spout");
    //Add the counter, with a name of 'count'
    //and parallelism hint of 12 executors
    //fieldsgrouping subscribes to the split bolt, and
    //ensures that the same word is sent to the same instance (group by field 'word')
    builder.setBolt("count", new WordCount(), 12).fieldsGrouping("split", new Fields("word"));

    //new configuration
    Config conf = new Config();
    //Set to false to disable debug information when
    // running in production on a cluster
    conf.setDebug(false);

    //If there are arguments, we are running on a cluster
    if (args != null && args.length > 0) {
      //parallelism hint to set the number of workers
      conf.setNumWorkers(3);
      //submit the topology
      StormSubmitter.submitTopology(args[0], conf, builder.createTopology());
    }
    //Otherwise, we are running locally
    else {
      //Cap the maximum number of executors that can be spawned
      //for a component to 3
      conf.setMaxTaskParallelism(3);
      //LocalCluster is used to run locally
      LocalCluster cluster = new LocalCluster();
      //submit the topology
      cluster.submitTopology("word-count", conf, builder.createTopology());
      //sleep
      Thread.sleep(10000);
      //shut down the cluster
      cluster.shutdown();
    }
  }
}
```

### <a name="configure-logging"></a>Настройка журнала

Для записи сведений в журнал используется [Apache log4j 2](https://logging.apache.org/log4j/2.x/) . Если ведение журнала не настроено, топология выдает диагностическую информацию. Чтобы управлять ведением журнала, создайте файл с именем `log4j2.xml` в `resources` каталоге, введя следующую команду:

```cmd
notepad resources\log4j2.xml
```

Затем скопируйте и вставьте приведенный ниже XML-текст в новый файл.  Затем закройте файл.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration>
    <Appenders>
        <Console name="STDOUT" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>
    </Appenders>
    <Loggers>
        <Logger name="com.microsoft.example" level="trace" additivity="false">
            <AppenderRef ref="STDOUT"/>
        </Logger>
        <Root level="error">
            <Appender-Ref ref="STDOUT"/>
        </Root>
    </Loggers>
</Configuration>
```

Это позволит настроить новое средство ведения журнала для класса `com.microsoft.example`, который включает компоненты в этом примере топологии. Для этого средства ведения журнала задается уровень трассировки, таким образом будут протоколироваться все сведения о журнале, генерируемые компонентами данной топологии.

В разделе `<Root level="error">` для корневого уровня ведения журнала (все, что не находится в файле `com.microsoft.example`) настраивается регистрация только сведений об ошибках.

Дополнительные сведения о настройке ведения журнала для log4j 2 см. в разделе [https://logging.apache.org/log4j/2.x/manual/configuration.html](https://logging.apache.org/log4j/2.x/manual/configuration.html) .

> [!NOTE]  
> Storm 0.10.0 и более поздних версий использует Log4j 2.x. В предыдущих версиях Storm использовался выпуск Log4j 1.x, в котором для настройки журнала применялся другой формат. Сведения о более старой конфигурации см. в разделе [https://cwiki.apache.org/confluence/display/LOGGINGLOG4J/Log4jXmlFormat](https://cwiki.apache.org/confluence/display/LOGGINGLOG4J/Log4jXmlFormat) .

## <a name="test-the-topology-locally"></a>Локальная проверка топологии

После сохранения файлов используйте следующую команду для локального тестирования топологии.

```cmd
mvn compile exec:java -Dstorm.topology=com.microsoft.example.WordCountTopology
```

Во время их выполнения топология отображает сведения о запуске. Следующий текст представляет собой пример выходных данных подсчета слов:

```output
17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 56 for word snow
17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 56 for word white
17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 112 for word seven
17:33:27 [Thread-16-count] INFO  com.microsoft.example.WordCount - Emitting a count of 195 for word the
17:33:27 [Thread-30-count] INFO  com.microsoft.example.WordCount - Emitting a count of 113 for word and
17:33:27 [Thread-30-count] INFO  com.microsoft.example.WordCount - Emitting a count of 57 for word dwarfs
17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 57 for word snow
```

Этот пример журнала означает, что слово "и" было отправлено 113 раз. Счетчик продолжит увеличиваться, пока выполняется топология. Это увеличение обусловлено тем, что spout постоянно создает одинаковые предложения.

Существует интервал в 5 секунд между эмиссией слов и счетчиков. Компонент **WordCount** настроен для отправки данных, только когда прибывает временной кортеж. Он запрашивает доставку таких кортежей каждые 5 секунд.

## <a name="convert-the-topology-to-flux"></a>Преобразование топологии для Flux

[Flux](https://storm.apache.org/releases/2.0.0/flux.html) — это новая инфраструктура, доступная в 0.10.0 и более поздних версиях. Flux позволяет отделить конфигурацию от реализации. Компоненты по-прежнему определяются на языке Java, но топология определяется с использованием файла YAML. Вы можете упаковать определение топологии по умолчанию в проекте или же использовать автономный файл при отправке топологии. При отправке топологии в более подсети используйте переменные среды или файлы конфигурации для заполнения значений определения топологии YAML.

Файл YAML определяет компоненты для топологии и поток данных между ними. Файл YAML можно включить в состав JAR-файла. Также можно использовать внешний файл YAML.

Дополнительные сведения о платформе Flux см. [здесь (https://storm.apache.org/releases/current/flux.html)](https://storm.apache.org/releases/current/flux.html).

> [!WARNING]  
> Из-за [ошибки (https://issues.apache.org/jira/browse/STORM-2055)](https://issues.apache.org/jira/browse/STORM-2055)в Storm 1.0.1 может потребоваться установить [среду разработки Storm ](https://storm.apache.org/releases/current/Setting-up-development-environment.html) для локального запуска топологий Flux.

1. Ранее была `WordCountTopology.java` определена топология, но она не требуется для Flux. Удалите файл с помощью следующей команды:

    ```cmd
    DEL src\main\java\com\microsoft\example\WordCountTopology.java
    ```

1. Введите следующую команду, чтобы создать и открыть новый файл `topology.yaml` :

    ```cmd
    notepad resources\topology.yaml
    ```

    Затем скопируйте и вставьте приведенный ниже текст в новый файл.  Затем закройте файл.

    ```yaml
    name: "wordcount"       # friendly name for the topology

    config:                 # Topology configuration
           topology.workers: 1     # Hint for the number of workers to create
  
    spouts:                 # Spout definitions
    - id: "sentence-spout"
           className: "com.microsoft.example.RandomSentenceSpout"
           parallelism: 1      # parallelism hint

    bolts:                  # Bolt definitions
    - id: "splitter-bolt"
           className: "com.microsoft.example.SplitSentence"
           parallelism: 1

    - id: "counter-bolt"
           className: "com.microsoft.example.WordCount"
           constructorArgs:
             - 10
           parallelism: 1

    streams:                # Stream definitions
    - name: "Spout --> Splitter" # name isn't used (placeholder for logging, UI, etc.)
           from: "sentence-spout"       # The stream emitter
           to: "splitter-bolt"          # The stream consumer
           grouping:                    # Grouping type
             type: SHUFFLE

    - name: "Splitter -> Counter"
           from: "splitter-bolt"
           to: "counter-bolt"
           grouping:
             type: FIELDS
             args: ["word"]           # field(s) to group on
    ```

1. Введите следующую команду для открытия `pom.xml` , чтобы внести описанные ниже изменения.

    ```cmd
    notepad pom.xml
    ```

   1. Добавьте следующую новую зависимость в разделе `<dependencies>` :

        ```xml
        <!-- Add a dependency on the Flux framework -->
        <dependency>
            <groupId>org.apache.storm</groupId>
            <artifactId>flux-core</artifactId>
            <version>${storm.version}</version>
        </dependency>
        ```

   1. Добавьте следующий подключаемый модуль в раздел `<plugins>`: Этот подключаемый модуль обрабатывает создание пакета (JAR-файла) для проекта и применяет некоторые преобразования, характерные для Flux, во время создания пакета.

        ```xml
        <!-- build an uber jar -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.2.1</version>
            <configuration>
                <transformers>
                    <!-- Keep us from getting a "can't overwrite file error" -->
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ApacheLicenseResourceTransformer" />
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer" />
                    <!-- We're using Flux, so refer to it as main -->
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>org.apache.storm.flux.Flux</mainClass>
                    </transformer>
                </transformers>
                <!-- Keep us from getting a bad signature error -->
                <filters>
                    <filter>
                        <artifact>*:*</artifact>
                        <excludes>
                            <exclude>META-INF/*.SF</exclude>
                            <exclude>META-INF/*.DSA</exclude>
                            <exclude>META-INF/*.RSA</exclude>
                        </excludes>
                    </filter>
                </filters>
            </configuration>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
        ```

   1. В разделе подключаемый модуль Exec Maven перейдите к `<configuration>`  >  `<mainClass>` и измените `${storm.topology}` на `org.apache.storm.flux.Flux` . Таким образом, Flux сможет обрабатывать выполнение топологии при ее локальном запуске в среде разработки.

   1. В `<resources>` разделе добавьте следующий элемент в `<includes>` . Он включает файл YAML, который определяет топологию как часть проекта.

        ```xml
        <include>topology.yaml</include>
        ```

## <a name="test-the-flux-topology-locally"></a>Локальная проверка топологии Flux

1. Введите следующую команду, чтобы скомпилировать и выполнить топологию Flux с помощью Maven:

    ```cmd
    mvn compile exec:java -Dexec.args="--local -R /topology.yaml"
    ```

    > [!WARNING]  
    > Эта команда не выполняется, если топология использует ресурсы Storm 1.0.1. Эта ошибка вызвана [https://issues.apache.org/jira/browse/STORM-2055](https://issues.apache.org/jira/browse/STORM-2055) . Вместо этого [установите Storm в среде разработки](https://storm.apache.org/releases/current/Setting-up-development-environment.html) и выполните следующие действия:
    >
    > Если вы [установили Storm в среде разработки](https://storm.apache.org/releases/current/Setting-up-development-environment.html), вместо этого можно использовать приведенные ниже команды.
    >
    > ```cmd
    > mvn compile package
    > storm jar target/WordCount-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --local -R /topology.yaml
    > ```

    Параметр `--local` выполняет топологию в локальном режиме в среде разработки. Параметр `-R /topology.yaml` использует файловый ресурс `topology.yaml` из JAR-файла для определения топологии.

    Во время их выполнения топология отображает сведения о запуске. Ниже приведен пример выходных данных.

    ```
    17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 56 for word snow
    17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 56 for word white
    17:33:27 [Thread-12-count] INFO  com.microsoft.example.WordCount - Emitting a count of 112 for word seven
    17:33:27 [Thread-16-count] INFO  com.microsoft.example.WordCount - Emitting a count of 195 for word the
    17:33:27 [Thread-30-count] INFO  com.microsoft.example.WordCount - Emitting a count of 113 for word and
    17:33:27 [Thread-30-count] INFO  com.microsoft.example.WordCount - Emitting a count of 57 for word dwarfs
    ```

    Существует 10-секундная задержка между пакетами записанных в журнал данных.

2. Создайте новую топологию YAML из проекта.

    1. Введите следующую команду, чтобы открыть `topology.xml` :

    ```cmd
    notepad resources\topology.yaml
    ```

    1. Найдите следующий раздел и измените значение `10` на `5` . Это изменит интервал между отправкой пакетов с данными о количестве слов с 10 до 5 секунд.  

    ```yaml
    - id: "counter-bolt"
           className: "com.microsoft.example.WordCount"
           constructorArgs:
             - 5
           parallelism: 1  
    ```

    1. Сохранить файл как `newtopology.yaml` .

3. Чтобы запустить топологию, введите следующую команду:

    ```cmd
    mvn exec:java -Dexec.args="--local resources/newtopology.yaml"
    ```

    Или, если в среде разработки имеется Storm, вы можете выполнить следующую команду.

    ```cmd
    storm jar target/WordCount-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --local resources/newtopology.yaml
    ```

    Эта команда использует в `newtopology.yaml` качестве определения топологии. Так как параметр `compile` не добавлен, Maven использует версию проекта, созданного на предыдущих шагах.

    После запуска топологии следует заметить, что время между выпущенными пакетами изменилось, чтобы отразить значение в `newtopology.yaml` . Таким образом, вы видите, что можно изменить конфигурацию в файле YAML, не выполняя повторную компиляцию топологии.

Дополнительные сведения об этих и других возможностях платформы Flux см. в статье, посвященной [Flux (https://storm.apache.org/releases/current/flux.html)](https://storm.apache.org/releases/current/flux.html).

## <a name="trident"></a>Trident

[Trident](https://storm.apache.org/releases/current/Trident-API-Overview.html) — Это высокоуровневая абстракция, обеспечиваемая степенью. Он поддерживает обработку с отслеживанием состояний. Основное преимущество Trident заключается в том, что гарантируется, что каждое сообщение, которое входит в топологию, обрабатывается только один раз. Без использования Trident топология только гарантирует, что каждое сообщение обрабатывается по крайней мере один раз. Существуют также другие отличия, например встроенные компоненты, которые можно использовать вместо создания сит. Винты заменяются менее универсальными компонентами, такими как фильтры, проекции и функции.

Приложения Trident можно создавать с помощью проектов Maven. Для этого используются те же основные действия, приведенные ранее в этой статье, отличается только код. Trident также не может использоваться с платформой Flux.

Дополнительные сведения о Trident см. в статье [Обзор API Trident](https://storm.apache.org/releases/current/Trident-API-Overview.html) (на английском языке).

## <a name="next-steps"></a>Дальнейшие действия

Вы узнали, как создать топологию Apache Storm с помощью Java. Теперь ознакомьтесь со следующими статьями.

* [Развертывание топологий Apache Storm в HDInsight и управление ими](apache-storm-deploy-monitor-topology-linux.md)

* [Разработка топологий с помощью Python](apache-storm-develop-python-topology.md)

Другие примеры топологий Apache Storm см. в статье [Примеры топологий и компонентов Storm для Apache Storm в HDInsight](apache-storm-example-topology.md).