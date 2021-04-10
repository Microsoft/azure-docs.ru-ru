---
title: Основные сведения об Apache Storm в Azure HDInsight
description: Apache Storm позволяет обрабатывать потоки данных в режиме реального времени. Azure HDInsight позволяет легко создавать кластеры Storm в облаке Azure. С помощью Visual Studio можно создать решения Storm, используя C#, а затем развернуть их в кластерах HDInsight Storm.
ms.service: hdinsight
ms.topic: overview
ms.custom: hdinsightactive,hdiseo17may2017,seoapr2020
ms.date: 04/20/2020
ms.openlocfilehash: 867f042332457ebc5fdd6b1f10ce7fb636309ba8
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104865338"
---
# <a name="what-is-apache-storm-on-azure-hdinsight"></a>Основные сведения об Apache Storm в Azure HDInsight

[Apache Storm](https://storm.apache.org/) — это распределенная отказоустойчивая вычислительная система с открытым исходным кодом. Ее можно использовать для обработки потоков данных в реальном времени с помощью [Apache Hadoop](../hadoop/apache-hadoop-introduction.md). Решения Storm могут также обеспечить гарантированную обработку данных и возможность воспроизвести те данные, которые не прошли удачную обработку в первый раз.

## <a name="why-use-apache-storm-on-hdinsight"></a>Преимущества использования Apache Storm в HDInsight

Использование Storm в HDInsight обеспечивает следующие преимущества:

* __Соглашение об уровне обслуживания (SLA) 99 % в отношении бесперебойной работы Storm__. Для кластеров Storm в HDInsight действует полная и постоянная поддержка. Кроме того, для кластеров Storm в HDInsight заявлена гарантированная доступность в течение 99,9 % времени. Это означает, что корпорация Майкрософт гарантирует возможность подключения к кластеру Storm извне в течение как минимум 99,9 % времени. Дополнительные сведения см. на странице [службы поддержки Azure](https://azure.microsoft.com/support/options/). См. также [сведения о соглашении об уровне обслуживания для HDInsight](https://azure.microsoft.com/support/legal/sla/hdinsight/v1_0/).

* Простая настройка с помощью скриптов во время или после создания кластера Storm. Дополнительные сведения см. в статье [Настройка кластеров HDInsight под управлением Linux с помощью действия сценария](../hdinsight-hadoop-customize-cluster-linux.md).

* **Создание решений на нескольких языках**. Вы можете написать компоненты Storm на удобном для вас языке, например Java, C# и Python.

    * Интеграция Visual Studio с HDInsight для разработки, администрирования и отслеживания топологий C#. Дополнительные сведения см. в статье [Разработка топологий для Apache Storm в HDInsight на C# с помощью средств Hadoop для Visual Studio](apache-storm-develop-csharp-visual-studio-topology.md).

    * Поддержка Java-интерфейса Trident. Вы можете создавать топологии Storm, поддерживающие обработку сообщений в рамках подхода "только один раз", сохраняемость "транзакционных" хранилищ данных, а также набор распространенных операций Stream Analytics.

* **Динамическое масштабирование**. Вы можете добавить или удалить рабочие узлы, не влияя на выполняющиеся топологии Storm. Чтобы использовать новые узлы, добавленные с помощью операций масштабирования, деактивируйте и повторно активируйте выполняющиеся топологии.

* **Создание конвейеров потоковой передачи с помощью нескольких служб Azure**. Storm в HDInsight интегрируется с другими службами Azure, например Центрами событий, Базой данных SQL, службой хранилища Azure и Azure Data Lake Storage. Пример приложения, которое интегрируется со службами Azure, см. в руководстве по [обработке событий из Центров событий Azure с помощью Apache Storm в HDInsight](https://github.com/Azure-Samples/hdinsight-java-storm-eventhub).

Список компаний, использующих Apache Storm в качестве решения для анализа данных в реальном времени, см. на [этой странице](https://storm.apache.org/Powered-By.html).

Чтобы начать работу с помощью Storm, см. статью [Учебник: создание и мониторинг топологии Apache Storm в Azure HDInsight](apache-storm-quickstart.md).

## <a name="how-does-apache-storm-work"></a>Использование Apache Storm

Storm работает с топологиями, а не заданиями [Apache Hadoop MapReduce](https://hadoop.apache.org/docs/r1.2.1/mapred_tutorial.html), которые вы могли использовать ранее. Топологии Storm состоят из нескольких компонентов, которые расположены в виде направленного ациклического графа (DAG). На указанной ниже схеме показаны потоки данных между компонентами. Каждый компонент использует один или несколько потоков данных и может генерировать дополнительные потоки. На следующей схеме показан поток данных между компонентами в базовой топологии подсчета слов:

:::image type="content" source="./media/apache-storm-overview/example-apache-storm-topology-diagram.png" alt-text="Пример расположения компонентов в топологии Storm" border="false":::

* Компоненты spout переносят данные в топологию. Они генерируют один или несколько потоков в топологии.

* Компоненты bolt используют потоки, генерируемые элементами spout или другими элементами bolt. Элементы bolt могут дополнительно генерировать потоки в топологии. Они также отвечают за запись данных во внешние службы или хранилище, например HDFS, Kafka или HBase.

## <a name="reliability"></a>Надежность

Система Apache Storm гарантирует, что каждое входящее сообщение будет полностью обработано даже в том случае, когда анализ данных распределен между сотнями узлов.

Узел Nimbus предоставляет функции аналогичные Apache Hadoop JobTracker. Nimbus назначает задачи другим узлам в кластере с помощью Apache ZooKeeper. Узлы Zookeeper обеспечивают координацию кластера и способствуют взаимодействию между узлом Nimbus и процессом контролера на рабочих узлах. Если один узел обработки выходит из строя, узел Nimbus узнает об этом и назначает задачу и связанные данные другому узлу.

По умолчанию для кластеров Apache Storm используется только один узел Nimbus, но кластер Storm в HDInsight имеет два. При сбое основного узла кластер Storm переключится на дополнительный на время восстановления основного узла. На схеме ниже показана конфигурация потока задач для Storm в HDInsight.

:::image type="content" source="./media/apache-storm-overview/storm-diagram-nimbus.png" alt-text="Схема с контролерами и компонентами Nimbus и Zookeeper" border="false":::

## <a name="ease-of-use"></a>Простота использования

|Использование |Описание |
|---|---|
|Подключение Secure Shell (SSH)|Головные узлы кластера Storm доступны через Интернет по протоколу SSH. С помощью SSH вы можете выполнять команды непосредственно в кластере. Дополнительные сведения см. в статье [Использование SSH с Hadoop на основе Linux в HDInsight из Linux, Unix или OS X](../hdinsight-hadoop-linux-use-ssh-unix.md).|
|Веб-подключение|Все кластеры HDInsight предоставляют веб-интерфейс Ambari. Он позволяет легко отслеживать, настраивать и администрировать службы в кластере. Кластеры Storm также предоставляют пользовательский интерфейс Storm. С его помощью вы можете отслеживать и контролировать выполнение топологий Storm из браузера. Дополнительные сведения см. в руководствах по [управлению кластерами HDInsight с помощью веб-интерфейса Apache Ambari](../hdinsight-hadoop-manage-ambari.md) и [мониторингу и администрированию пользовательского интерфейса Apache Storm](apache-storm-deploy-monitor-topology-linux.md#monitor-and-manage-a-topology-using-the-storm-ui).|
|Azure PowerShell и Azure CLI|Azure PowerShell и Azure CLI предоставляют служебные программы командной строки для работы с HDInsight и другими службами Azure из клиентской системы.|
|Интеграция Visual Studio|Средства Azure Data Lake для Visual Studio включают шаблоны проектов для создания топологий C# Storm с помощью платформы SCP.NET. Средства Data Lake также предоставляют средства для развертывания, отслеживания и администрирования решений с помощью Storm в HDInsight. Дополнительные сведения см. в статье [Разработка топологий для Apache Storm в HDInsight на C# с помощью средств Hadoop для Visual Studio](apache-storm-develop-csharp-visual-studio-topology.md).|

## <a name="integration-with-other-azure-services"></a>Интеграция с другими службами Azure

* __Хранилище Azure Data Lake Storage__. См. статью [Использование Azure Data Lake Storage с Apache Storm в HDInsight](apache-storm-write-data-lake-store.md).

* __Центры событий__. Примеры использования Центров событий с кластером Storm

    * [Обработка событий из Центров событий Azure с помощью Apache Storm в HDInsight (Java)](https://github.com/Azure-Samples/hdinsight-java-storm-eventhub)

    * [Обработка событий из Центров событий Azure с помощью Apache Storm в HDInsight (C#)](apache-storm-develop-csharp-event-hub-topology.md)

* __База данных SQL__, __Cosmos DB__, __Центры событий__ и __HBase__. Примеры шаблонов для работы с ними включены в средства Data Lake для Visual Studio. Дополнительные сведения см. в руководстве по [разработке топологии C# для Apache Storm в HDInsight](apache-storm-develop-csharp-visual-studio-topology.md).

## <a name="apache-storm-use-cases"></a>Варианты использования Apache Storm

Ниже приведены несколько распространенных ситуаций, в которых вам может помочь кластер Storm в HDInsight.

* Интернет вещей.
* Обнаружение мошенничества.
* Социальная аналитика.
* Извлечение, преобразование и загрузка.
* Мониторинг сетей.
* Поиск
* Службы мобильного взаимодействия

См. реальные сценарии того, [как компании используют Apache Storm](https://storm.apache.org/Powered-By.html).

## <a name="development"></a>Разработка

Средства Data Lake для Visual Studio позволяют разработчикам .NET проектировать и реализовывать топологии на языке C#. Вы также можете создавать гибридные топологии, в которых используются компоненты Java и C#. Дополнительные сведения см. в статье [Разработка топологий для Apache Storm в HDInsight на C# с помощью средств Hadoop для Visual Studio](apache-storm-develop-csharp-visual-studio-topology.md).

Можно также разрабатывать решения Java с помощью интегрированной среды разработки по своему усмотрению. Дополнительные сведения см. в руководстве по [разработке топологии Java для Apache Storm в HDInsight](apache-storm-develop-java-topology.md).

Для разработки компонентов Storm также может использоваться Python. Дополнительные сведения см. в руководстве по [разработке топологии Python для Apache Storm в HDInsight](apache-storm-develop-python-topology.md).

## <a name="common-development-patterns"></a>Общие шаблоны разработки

### <a name="guaranteed-message-processing"></a>Гарантированная обработка сообщений

Apache Storm может обеспечить различные уровни гарантированной обработки сообщений. Например, простое приложение Storm гарантирует как минимум одну обработку, в то время как Trident может гарантировать только одну обработку. См.сведения о [гарантиях на обработку данных](https://storm.apache.org/about/guarantees-data-processing.html) на веб-сайте apache.org.

### <a name="ibasicbolt"></a>IBasicBolt

Типичным является шаблон чтения входного кортежа, выдающий значение 0 или больше, а затем подтверждающий входной кортеж непосредственно в конце метода execute. Storm обеспечивает интерфейс [IBasicBolt](https://storm.apache.org/releases/current/javadocs/org/apache/storm/topology/IBasicBolt.html) для автоматизации этого шаблона.

### <a name="joins"></a>Соединения

Объединение потоков данных отличается в разных приложениях. Например, вы можете объединять все кортежи с нескольких потоков в один новый поток или объединять только пакеты кортежей для отдельного окна. В любом случае объединение можно выполнить с помощью [fieldsGrouping](https://storm.apache.org/releases/current/javadocs/org/apache/storm/topology/InputDeclarer.html#fieldsGrouping-java.lang.String-org.apache.storm.tuple.Fields-). Группирование полей — это способ определения того, как кортежи перенаправляются к элементам bolts.

В приведенном ниже примере Java используется fieldsGrouping для перенаправления кортежей из компонентов "1", "2" и "3" в элемент bolt MyJoiner.

```java
builder.setBolt("join", new MyJoiner(), parallelism) .fieldsGrouping("1", new Fields("joinfield1", "joinfield2")) .fieldsGrouping("2", new Fields("joinfield1", "joinfield2")) .fieldsGrouping("3", new Fields("joinfield1", "joinfield2"));
```

### <a name="batches"></a>Пакеты

Apache Storm предоставляет внутренний временной механизм (временный кортеж). Вы можете задать частоту использования временного кортежа в топологии.

Пример использования временного кортежа из компонента на C# см. здесь: [PartialBoltCount.cs](https://github.com/hdinsight/hdinsight-storm-examples/blob/3b2c960549cac122e8874931df4801f0934fffa7/EventCountExample/EventCountTopology/src/main/java/com/microsoft/hdinsight/storm/examples/PartialCountBolt.java).

### <a name="caches"></a>Кэши

Для ускорения обработки часто используется кэширование в памяти, при котором в памяти сохраняются часто используемые ресурсы. Так как топология распределяется между несколькими узлами и несколькими процессами в пределах одного узла, вы можете использовать [fieldsGrouping](https://storm.apache.org/releases/current/javadocs/org/apache/storm/topology/InputDeclarer.html#fieldsGrouping-java.lang.String-org.apache.storm.tuple.Fields-). `fieldsGrouping` гарантирует, что кортежи, содержащие поля, используемые для поиска в кэше, всегда перенаправляются к одному и тому же процессу. Эта функция группирования избавит от дублирования записей кэша в процессах.

### <a name="stream-top-n"></a>Передача значения "Первые N"

Когда ваша топология зависит от расчета значения "Первые N", это значение нужно рассчитывать параллельно. Затем объедините результат расчетов в глобальное значение. Эту операцию можно выполнить с помощью [fieldsGrouping](https://storm.apache.org/releases/current/javadocs/org/apache/storm/topology/InputDeclarer.html#fieldsGrouping-java.lang.String-org.apache.storm.tuple.Fields-), направив отдельные поля в параллельную обработку, а затем — в элемент bolt, который определяет глобальное значение "Топ N".

Для расчета значения "Первые N" см. пример [RollingTopWords](https://github.com/apache/storm/blob/master/examples/storm-starter/src/jvm/org/apache/storm/starter/RollingTopWords.java).

## <a name="logging"></a>Logging

Storm использует Apache Log4j 2 для записи информации в журнал. По умолчанию в журнал записывается большой объем данных, и разобраться в этой информации может быть трудно. Чтобы управлять ведением журналов, включите файл конфигурации ведения журналов в топологию Storm.

Пример топологии, в котором показано, как настроить ведение журнала, см. в примере Storm в HDInsight в статье [Разработка топологий на основе Java для базовых приложений подсчета слов с помощью Apache Storm и Maven в HDInsight](apache-storm-develop-java-topology.md).

## <a name="next-steps"></a>Дальнейшие действия

Ниже приведены статьи с описанием решений для анализа данных в реальном времени с помощью Apache Storm в HDInsight.

* [Учебник: создание и мониторинг топологии Apache Storm в Azure HDInsight](apache-storm-quickstart.md)
* [Примеры топологий и компонентов Storm для Apache Storm в HDInsight](apache-storm-example-topology.md)
