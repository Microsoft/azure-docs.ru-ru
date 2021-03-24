---
title: Обработка событий из Центров событий с помощью Storm в Azure HDInsight
description: Узнайте, как обрабатывать данные службы Центров событий Azure с использованием топологии C# Storm, созданной в Visual Studio с помощью средств HDInsight для Visual Studio.
ms.service: hdinsight
ms.topic: how-to
ms.date: 01/14/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: ee483872535c58bde521cf0a751058fdc2fecd40
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104871458"
---
# <a name="process-events-from-azure-event-hubs-with-apache-storm-on-hdinsight-c"></a>Обработка событий из Центров событий Azure с помощью Apache Storm в HDInsight (C#)

Узнайте, как работать с концентраторами событий Azure из [Apache Storm](https://storm.apache.org/) в HDInsight. В этом документе для чтения данных из Центров событий и записи в него используется топология Storm на C#.

> [!NOTE]  
> Этот же проект на языке Java рассматривается в статье [Обработка событий из службы Центров событий Azure с помощью Apache Storm в HDInsight (Java)](https://github.com/Azure-Samples/hdinsight-java-storm-eventhub).

## <a name="scpnet"></a>SCP.NET

Действия, описанные в этом документе, используют SCP.NET. Это пакет NuGet, который упрощает создание топологий и компонентов C# для использования со Storm в HDInsight.

В HDInsight 3.4 и более поздних версий использует Mono для запуска топологий C#. Для примера в этом документе используется HDInsight 3.6. Если вы планируете создавать собственные решения .NET для HDInsight, вам следует просмотреть документ о [совместимости Mono](https://www.mono-project.com/docs/about-mono/compatibility/), чтобы определить потенциальные проблемы с совместимостью.

### <a name="cluster-versioning"></a>Управление версиями кластера

Пакет Microsoft.SCP.Net.SDK NuGet, который вы используете для своего проекта, должен соответствовать основному номеру версии Storm, установленной в HDInsight. В HDInsight версий 3.5 и 3.6 используется Storm 1.x, следовательно, для таких кластеров нужно использовать версию SCP.NET 1.0.x.x.

Топологии C# должны быть рассчитаны на использование с .NET 4.5.

## <a name="how-to-work-with-event-hubs"></a>Работа с Центрами событий

Корпорация Майкрософт предоставляет набор компонентов Java для взаимодействия с Центрами событий из топологии Storm. Файл архива Java (JAR), который содержит версию этих компонентов, совместимую с HDInsight 3,6, можно найти по адресу [https://github.com/hdinsight/mvn-repo/raw/master/org/apache/storm/storm-eventhubs/1.1.0.1/storm-eventhubs-1.1.0.1.jar](https://github.com/hdinsight/mvn-repo/raw/master/org/apache/storm/storm-eventhubs/1.1.0.1/storm-eventhubs-1.1.0.1.jar) .

> [!IMPORTANT]  
> Эти компоненты написаны на языке Java, но их можно свободно использовать из топологии C#.

В этом примере используются следующие компоненты:

* __EventHubSpout__: считывает данные из Центров событий Azure;
* __EventHubBolt__: записывает данные в Центры событий;
* __EventHubSpoutConfig__: используется для настройки EventHubSpout;
* __EventHubBoltConfig__: используется для настройки EventHubBolt;

### <a name="example-spout-usage"></a>Пример использования объекта spout

SCP.NET предоставляет методы для добавления EventHubSpout в топологию. С ними добавить объект spout будет проще, чем с использованием универсальных методов для добавления компонента Java. В следующем примере показано, как создается объект spout с использованием методов __SetEventHubSpout__ и **EventHubSpoutConfig**, предоставляемых в SCP.NET:

```csharp
 topologyBuilder.SetEventHubSpout(
    "EventHubSpout",
    new EventHubSpoutConfig(
        ConfigurationManager.AppSettings["EventHubSharedAccessKeyName"],
        ConfigurationManager.AppSettings["EventHubSharedAccessKey"],
        ConfigurationManager.AppSettings["EventHubNamespace"],
        ConfigurationManager.AppSettings["EventHubEntityPath"],
        eventHubPartitions),
    eventHubPartitions);
```

В предыдущем примере создается новый компонент spout с именем __EventHubSpout__ и настраивается его взаимодействие с концентратором событий. Для этого компонента задается указание параллелизма, равное числу разделов в концентраторе событий. Это позволяет Storm создать по экземпляру компонента для каждой секции.

### <a name="example-bolt-usage"></a>Пример использования объекта bolt

Используйте метод **javacomponentconstructor** для создания экземпляра молнии. В следующем примере показано, как создать и настроить новый экземпляр **EventHubBolt**:

```csharp
// Java construcvtor for the Event Hub Bolt
JavaComponentConstructor constructor = JavaComponentConstructor.CreateFromClojureExpr(
    String.Format(@"(org.apache.storm.eventhubs.bolt.EventHubBolt. (org.apache.storm.eventhubs.bolt.EventHubBoltConfig. " +
        @"""{0}"" ""{1}"" ""{2}"" ""{3}"" ""{4}"" {5}))",
        ConfigurationManager.AppSettings["EventHubPolicyName"],
        ConfigurationManager.AppSettings["EventHubPolicyKey"],
        ConfigurationManager.AppSettings["EventHubNamespace"],
        "servicebus.windows.net",
        ConfigurationManager.AppSettings["EventHubName"],
        "true"));

// Set the bolt to subscribe to data from the spout
topologyBuilder.SetJavaBolt(
    "eventhubbolt",
    constructor,
    partitionCount)
        .shuffleGrouping("Spout");
```

> [!NOTE]  
> В этом примере используется выражение Clojure, передаваемое в виде строки, вместо создания объектов **EventHubBoltConfig** с помощью **JavaComponentConstructor**, как было предложено выше в примере с объектом spout. Работает каждый из этих методов. Можно использовать любой из этих методов по своему усмотрению.

## <a name="download-the-completed-project"></a>Скачивание полного примера проекта

Вы можете скачать полную версию проекта, созданного в этой статье, из [GitHub](https://github.com/Azure-Samples/hdinsight-dotnet-java-storm-eventhub). Однако вам по-прежнему необходимо предоставить параметры конфигурации, выполнив действия, описанные в этой статье.

### <a name="prerequisites"></a>Предварительные требования

* Кластер Apache Storm в HDInsight. Ознакомьтесь со статьей [Create Linux-based clusters in HDInsight by using the Azure portal](../hdinsight-hadoop-create-linux-clusters-portal.md) (Создание кластеров под управлением Linux в HDInsight с помощью портала Azure) и выберите **Storm** для параметра **Тип кластера**.

* [Концентратор событий Azure](../../event-hubs/event-hubs-create.md).

* [Пакет SDK для Azure .NET](https://azure.microsoft.com/downloads/).

* [Средства HDInsight для Visual Studio](../hadoop/apache-hadoop-visual-studio-tools-get-started.md).

* Пакет Java JDK 1.8 или более поздней версии, в зависимости от используемой среды разработки. Скачиваемые файлы JDK доступны на сайте [Oracle](/azure/developer/java/fundamentals/java-jdk-long-term-support).

  * Переменная среды **JAVA_HOME** должна указывать на каталог, содержащий Java.
  * Каталог **% JAVA_HOME%** переданного каталога должен находиться в пути.

## <a name="download-the-event-hubs-components"></a>Скачивание компонентов Центров событий

Скачайте компонент spout и молния концентраторов событий из [https://github.com/hdinsight/mvn-repo/raw/master/org/apache/storm/storm-eventhubs/1.1.0.1/storm-eventhubs-1.1.0.1.jar](https://github.com/hdinsight/mvn-repo/raw/master/org/apache/storm/storm-eventhubs/1.1.0.1/storm-eventhubs-1.1.0.1.jar) .

Создайте каталог `eventhubspout` и сохраните в него файл.

## <a name="configure-event-hubs"></a>Настройка Центров событий Azure

Служба "Центры событий" используется в качестве источника данных для этого примера. Используйте сведения в разделе "Создание концентратора событий" блока о [начале работы с Центрами событий](../../event-hubs/event-hubs-create.md).

1. Создав **концентратор событий**, просмотрите его параметры на портале Azure и щелкните **Политики общего доступа**. Выберите **+ Добавить** , чтобы создать следующие политики:

   | Имя | Разрешения |
   | --- | --- |
   | writer |Отправить |
   | reader |Прослушивание |

    :::image type="content" source="./media/apache-storm-develop-csharp-event-hub-topology/share-access-policies.png" alt-text="Снимок экрана окна политик общего доступа" border="true":::

2. Выберите политики **reader** и **writer**. Скопируйте и сохраните значение первичного ключа для обеих политик, так как они будут использоваться позже.

## <a name="configure-the-eventhubwriter"></a>Настройка EventHubWriter

1. Если вы еще не установили последнюю версию средств HDInsight для Visual Studio, см. статью Приступая к [работе с инструментами hdinsight для Visual Studio](../hadoop/apache-hadoop-visual-studio-tools-get-started.md).

2. Скачайте решение из [eventhub-storm-hybrid](https://github.com/Azure-Samples/hdinsight-dotnet-java-storm-eventhub).

3. Откройте **EventHubExample. sln**. В проекте **EventHubWriter** откройте файл **App.config**. Используйте сведения из концентратора событий, настроенного ранее, чтобы указать значения следующих ключей:

   | Ключ | Значение |
   | --- | --- |
   | EventHubPolicyName |writer (если для политики с разрешением на *отправку* указано другое имя, используйте его). |
   | EventHubPolicyKey |Ключ для политики writer. |
   | EventHubNamespace |Пространство имен, содержащее концентратор событий. |
   | EventHubName |Имя концентратора событий. |
   | EventHubPartitionCount |Число разделов в концентраторе событий. |

4. Сохраните и закройте файл **App.config**.

## <a name="configure-the-eventhubreader"></a>Настройка EventHubReader

1. Откройте проект **EventHubReader**.

2. Для **EventHubReader** откройте файл **App.config**. Используйте сведения из концентратора событий, настроенного ранее, чтобы указать значения следующих ключей:

   | Ключ | Значение |
   | --- | --- |
   | EventHubPolicyName |reader (если для политики с разрешением на *прослушивание* указано другое имя, используйте его). |
   | EventHubPolicyKey |Ключ для политики reader. |
   | EventHubNamespace |Пространство имен, содержащее концентратор событий. |
   | EventHubName |Имя концентратора событий. |
   | EventHubPartitionCount |Число разделов в концентраторе событий. |

3. Сохраните и закройте файл **App.config**.

## <a name="deploy-the-topologies"></a>Развертывание топологий

1. В **Обозреватель решений** щелкните правой кнопкой мыши проект **EventHubReader** и выберите пункт **отправить в кластер HDInsight**.

    :::image type="content" source="./media/apache-storm-develop-csharp-event-hub-topology/submit-to-apache-storm.png" alt-text="Снимок экрана с обозревателем решений с выделенной функцией &quot;Отправить в Storm в HDInsight&quot;" border="true":::

2. В диалоговом окне **Submit Topology** (Отправка топологии) выберите **кластер Storm**. Разверните узел **Дополнительные конфигурации**, выберите **Java File Paths** (Пути к файлам Java), щелкните **...** и укажите каталог, содержащий скачанный ранее JAR-файл. Теперь нажмите кнопку **Отправить**.

    :::image type="content" source="./media/apache-storm-develop-csharp-event-hub-topology/submit-storm-topology.png" alt-text="Снимок экрана с диалоговым окном Submit Topology (Отправка топологии)" border="true":::

3. После отправки топологии отобразится **средство просмотра топологий Storm**. Чтобы просмотреть статистику для топологии, выберите топологию **EventHubReader** в левой панели.

    :::image type="content" source="./media/apache-storm-develop-csharp-event-hub-topology/storm-topology-viewer.png" alt-text="Снимок экрана средства просмотра топологий Storm" border="true":::

4. В **Обозреватель решений** щелкните правой кнопкой мыши проект **EventHubWriter** и выберите пункт **отправить в кластер HDInsight**.

5. В диалоговом окне **Submit Topology** (Отправка топологии) выберите **кластер Storm**. Разверните узел **Дополнительные конфигурации**, выберите **Java File Paths** (Пути к файлам Java), щелкните **...** и укажите каталог, содержащий скачанный вами ранее JAR-файл. Теперь нажмите кнопку **Отправить**.

6. После отправки топологии обновите список топологий в **средстве просмотра топологий Storm** , чтобы убедиться, что обе топологии выполняются в кластере.

7. В **средстве просмотра топологий Storm** выберите топологию **EventHubReader**.

8. Чтобы открыть сводку по компонентам для молнии, дважды щелкните компонент **LogBolt** на схеме.

9. В разделе **Executors** (Исполнители) выберите одну из ссылок в столбце **Порт**. Будет выведена информация, зарегистрированная компонентом. Записанные данные аналогичны приведенным ниже:

    ```output
    2017-03-02 14:51:29.255 m.s.p.TaskHost [INFO] Received C# STDOUT: 2017-03-02 14:51:29,255 [1] INFO  EventHubReader_LogBolt [(null)] - Received data: {"deviceValue":1830978598,"deviceId":"8566ccbc-034d-45db-883d-d8a31f34068e"}
    2017-03-02 14:51:29.283 m.s.p.TaskHost [INFO] Received C# STDOUT: 2017-03-02 14:51:29,283 [1] INFO  EventHubReader_LogBolt [(null)] - Received data: {"deviceValue":1756413275,"deviceId":"647a5eff-823d-482f-a8b4-b95b35ae570b"}
    2017-03-02 14:51:29.313 m.s.p.TaskHost [INFO] Received C# STDOUT: 2017-03-02 14:51:29,312 [1] INFO  EventHubReader_LogBolt [(null)] - Received data: {"deviceValue":1108478910,"deviceId":"206a68fa-8264-4d61-9100-bfdb68ee8f0a"}
    ```

## <a name="stop-the-topologies"></a>Остановка топологий

Чтобы остановить топологии, выберите каждую из них в **средстве просмотра топологий Storm**, а затем выберите элемент **Прервать**.

:::image type="content" source="./media/apache-storm-develop-csharp-event-hub-topology/kill-storm-topology1.png" alt-text="Снимок экрана средства просмотра топологии Storm с выделенной кнопкой &quot;Прервать&quot;" border="true":::

## <a name="delete-your-cluster"></a>Удаление кластера

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как использовать концентраторы событий Java spout и молния из топологии C# для работы с данными в концентраторах событий Azure. Дополнительные сведения о создании топологий C# см. в следующих статьях:

* [Разработка топологий для Apache Storm в HDInsight на C# с помощью Visual Studio](apache-storm-develop-csharp-visual-studio-topology.md)
* [Руководство по программированию для SCP](apache-storm-scp-programming-guide.md)
* [Примеры топологий и компонентов Storm для Apache Storm в HDInsight](apache-storm-example-topology.md)