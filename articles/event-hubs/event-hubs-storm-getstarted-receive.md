---
title: Краткое руководство. Получение событий с помощью Apache Storm в Центрах событий Azure
description: Краткое руководство. В этой статье приводятся сведения о том, как получать события от службы "Центры событий Azure" с помощью Apache Storm.
ms.topic: quickstart
ms.date: 06/23/2020
ms.openlocfilehash: b79c2958a10d0193b706c92a28b1792d63c24b82
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107302970"
---
# <a name="quickstart-receive-events-from-event-hubs-using-apache-storm"></a>Краткое руководство. Получение событий из Центров событий с помощью Apache Storm

[Apache Storm](https://storm.incubator.apache.org) — это распределенная система вычислений в реальном времени, упрощающая надежную обработку неограниченных потоков данных. В этом разделе показано использование spout Storm Центров событий Azure для приема событий из Центров событий. С помощью Apache Storm можно разделить события между несколькими процессами, размещенными в разных узлах. Интеграция Центров событий с помощью Storm упрощает использование событий путем прозрачного определения контрольных точек в ходе выполнения с помощью установки Storm Zookeeper, управляя постоянными контрольными точками и одновременно облегчает получение от Центров событий.

Дополнительные сведения о шаблонах получения данных в Центры событий см. в статье [Общие сведения о Центрах событий][Event Hubs overview].

## <a name="prerequisites"></a>Предварительные требования
Вначале следует **создать пространство имен Центров событий и концентратор событий**. Вы можете использовать [портал Azure](https://portal.azure.com) для создания пространства имен типа Центров событий и получать учетные данные для управления, требуемые приложению для взаимодействия с концентратором событий. Чтобы создать пространство имен и концентратор событий, выполните инструкции из [этой статьи](event-hubs-create.md). 

## <a name="create-project-and-add-code"></a>Создание проекта и добавление кода

В данном учебнике используется установка программы [HDInsight Storm][HDInsight Storm] , которая поставляется вместе с уже доступным Spout в Центрах событий.

1. Следуя процедурам, приведенным в разделе [HDInsight Storm — начало работы](../hdinsight/storm/apache-storm-overview.md) , создайте новый кластер HDInsight и подключитесь к нему через удаленный рабочий стол.
2. Скопируйте файл `%STORM_HOME%\examples\eventhubspout\eventhubs-storm-spout-0.9-jar-with-dependencies.jar` в локальную среду разработки. Он содержит пакет events-storm-spout.
3. Для установки пакета в локальный репозиторий Maven выполните следующую команду. Это позволит добавить его в качестве ссылки в проекте Storm позже.

    ```shell
    mvn install:install-file -Dfile=target\eventhubs-storm-spout-0.9-jar-with-dependencies.jar -DgroupId=com.microsoft.eventhubs -DartifactId=eventhubs-storm-spout -Dversion=0.9 -Dpackaging=jar
    ```
4. В Eclipse создайте проект Maven (щелкните **File** (Файл), **New** (Создать), а затем **Project** (Проект)).
   
    ![File (Файл) > New (Создать) > Project (Проект)][12]
5. Выберите параметр **Use default Workspace location** (Использовать расположение рабочей области по умолчанию), а затем нажмите кнопку **Next** (Далее).
6. Выберите архетип **maven-archetype-quickstart** и нажмите кнопку **Next** (Далее).
7. Вставьте параметры **GroupId** и **ArtifactId**, а затем нажмите кнопку **Finish** (Готово).
8. В файле **pom.xml** добавьте следующие зависимости в узел `<dependency>`:

    ```xml  
    <dependency>
        <groupId>org.apache.storm</groupId>
        <artifactId>storm-core</artifactId>
        <version>0.9.2-incubating</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>com.microsoft.eventhubs</groupId>
        <artifactId>eventhubs-storm-spout</artifactId>
        <version>0.9</version>
    </dependency>
    <dependency>
        <groupId>com.netflix.curator</groupId>
        <artifactId>curator-framework</artifactId>
        <version>1.3.3</version>
        <exclusions>
            <exclusion>
                <groupId>log4j</groupId>
                <artifactId>log4j</artifactId>
            </exclusion>
            <exclusion>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-log4j12</artifactId>
            </exclusion>
        </exclusions>
        <scope>provided</scope>
    </dependency>
    ```

9. В папке **src** создайте файл с именем **Config.properties** и скопируйте следующее содержимое, заменив значения `receive rule key` и `event hub name`:

    ```java
    eventhubspout.username = ReceiveRule
    eventhubspout.password = {receive rule key}
    eventhubspout.namespace = ioteventhub-ns
    eventhubspout.entitypath = {event hub name}
    eventhubspout.partitions.count = 16
       
    # if not provided, will use storm's zookeeper settings
    # zookeeper.connectionstring=localhost:2181
       
    eventhubspout.checkpoint.interval = 10
    eventhub.receiver.credits = 10
    ```
    Значение для **eventhub.receiver.credits** определяет, сколько событий являются пакетными перед их выпуском в конвейер Storm. Для простоты в этом примере используется значение 10. В рабочей среде обычно указывается более высокое значение, например, 1024.
10. Создайте новый класс с именем **LoggerBolt** с использованием следующего кода:
    
    ```java
    import java.util.Map;
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import backtype.storm.task.OutputCollector;
    import backtype.storm.task.TopologyContext;
    import backtype.storm.topology.OutputFieldsDeclarer;
    import backtype.storm.topology.base.BaseRichBolt;
    import backtype.storm.tuple.Tuple;
    
    public class LoggerBolt extends BaseRichBolt {
        private OutputCollector collector;
        private static final Logger logger = LoggerFactory
                  .getLogger(LoggerBolt.class);
    
        @Override
        public void execute(Tuple tuple) {
            String value = tuple.getString(0);
            logger.info("Tuple value: " + value);
   
            collector.ack(tuple);
        }
   
        @Override
        public void prepare(Map map, TopologyContext context, OutputCollector collector) {
            this.collector = collector;
            this.count = 0;
        }
        
        @Override
        public void declareOutputFields(OutputFieldsDeclarer declarer) {
            // no output fields
        }
    
    }
    ```
    
    Этот Storm Bolt регистрирует содержимое полученного события. Эго можно легко расширить для хранения кортежей в службе хранилища. В [примере использования HDInsight Storm с концентратором событий] для хранения данных в службе хранилища Azure и Power BI применяется одинаковый подход.
11. Создайте класс с названием **LogTopology** и со следующим кодом:
    
    ```java
    import java.io.FileReader;
    import java.util.Properties;
    import backtype.storm.Config;
    import backtype.storm.LocalCluster;
    import backtype.storm.StormSubmitter;
    import backtype.storm.generated.StormTopology;
    import backtype.storm.topology.TopologyBuilder;
    import com.microsoft.eventhubs.samples.EventCount;
    import com.microsoft.eventhubs.spout.EventHubSpout;
    import com.microsoft.eventhubs.spout.EventHubSpoutConfig;
        
    public class LogTopology {
        protected EventHubSpoutConfig spoutConfig;
        protected int numWorkers;
        
        protected void readEHConfig(String[] args) throws Exception {
            Properties properties = new Properties();
            if (args.length > 1) {
                properties.load(new FileReader(args[1]));
            } else {
                properties.load(EventCount.class.getClassLoader()
                        .getResourceAsStream("Config.properties"));
            }
        
            String username = properties.getProperty("eventhubspout.username");
            String password = properties.getProperty("eventhubspout.password");
            String namespaceName = properties
                    .getProperty("eventhubspout.namespace");
            String entityPath = properties.getProperty("eventhubspout.entitypath");
            String zkEndpointAddress = properties
                    .getProperty("zookeeper.connectionstring"); // opt
            int partitionCount = Integer.parseInt(properties
                    .getProperty("eventhubspout.partitions.count"));
            int checkpointIntervalInSeconds = Integer.parseInt(properties
                    .getProperty("eventhubspout.checkpoint.interval"));
            int receiverCredits = Integer.parseInt(properties
                    .getProperty("eventhub.receiver.credits")); // prefetch count
                                                                // (opt)
            System.out.println("Eventhub spout config: ");
            System.out.println("  partition count: " + partitionCount);
            System.out.println("  checkpoint interval: "
                    + checkpointIntervalInSeconds);
            System.out.println("  receiver credits: " + receiverCredits);
     
            spoutConfig = new EventHubSpoutConfig(username, password,
                    namespaceName, entityPath, partitionCount, zkEndpointAddress,
                    checkpointIntervalInSeconds, receiverCredits);
        
            // set the number of workers to be the same as partition number.
            // the idea is to have a spout and a logger bolt co-exist in one
            // worker to avoid shuffling messages across workers in storm cluster.
            numWorkers = spoutConfig.getPartitionCount();
        
            if (args.length > 0) {
                // set topology name so that sample Trident topology can use it as
                // stream name.
                spoutConfig.setTopologyName(args[0]);
            }
        }
        
        protected StormTopology buildTopology() {
            TopologyBuilder topologyBuilder = new TopologyBuilder();
       
            EventHubSpout eventHubSpout = new EventHubSpout(spoutConfig);
            topologyBuilder.setSpout("EventHubsSpout", eventHubSpout,
                    spoutConfig.getPartitionCount()).setNumTasks(
                    spoutConfig.getPartitionCount());
            topologyBuilder
                    .setBolt("LoggerBolt", new LoggerBolt(),
                            spoutConfig.getPartitionCount())
                    .localOrShuffleGrouping("EventHubsSpout")
                    .setNumTasks(spoutConfig.getPartitionCount());
            return topologyBuilder.createTopology();
        }
        
        protected void runScenario(String[] args) throws Exception {
            boolean runLocal = true;
            readEHConfig(args);
            StormTopology topology = buildTopology();
            Config config = new Config();
            config.setDebug(false);
        
            if (runLocal) {
                config.setMaxTaskParallelism(2);
                LocalCluster localCluster = new LocalCluster();
                localCluster.submitTopology("test", config, topology);
                Thread.sleep(5000000);
                localCluster.shutdown();
            } else {
                config.setNumWorkers(numWorkers);
                StormSubmitter.submitTopology(args[0], config, topology);
            }
        }
        
        public static void main(String[] args) throws Exception {
            LogTopology topology = new LogTopology();
            topology.runScenario(args);
        }
    }
    ```

    Этот класс создает новую воронку Центров событий, используя свойства в файле конфигурации для создания ее экземпляра. Важно отметить, что в данном примере создается такое же количество задач spout, сколько и разделов в концентраторе событий, чтобы обеспечить максимальный параллелизм, допустимый для этого концентратора событий.

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о Центрах событий см. в следующих источниках:

* [Общие сведения о Центрах событий][Event Hubs overview]
* [Создание концентратора событий](event-hubs-create.md)
* [Часто задаваемые вопросы о Центрах событий](event-hubs-faq.yml)

<!-- Links -->
[Event Hubs overview]: ./event-hubs-about.md
[HDInsight Storm]: ../hdinsight/storm/apache-storm-overview.md
[примере использования HDInsight Storm с концентратором событий]: https://github.com/Azure-Samples/hdinsight-java-storm-eventhub

<!-- Images -->

[12]: ./media/event-hubs-get-started-receive-storm/create-storm1.png
