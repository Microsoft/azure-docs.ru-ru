---
title: Разработка топологий Apache Storm с помощью Visual Studio и C# в Azure HDInsight
description: Сведения о создании топологий Storm в C#. Создайте топологию подсчета слов в Visual Studio с помощью средств Hadoop для Visual Studio.
ROBOTS: NOINDEX
ms.service: hdinsight
ms.topic: how-to
ms.date: 12/31/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: a81f2b21545a5362168482f3f0a65fbbbf381c10
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98929155"
---
# <a name="develop-c-topologies-for-apache-storm-by-using-the-data-lake-tools-for-visual-studio"></a>Разработка топологий для Apache Storm на C# с помощью средств Data Lake для Visual Studio

Сведения о создании топологии Apache Storm на языке C# с помощью средств Azure Data Lake (Apache Hadoop) для Visual Studio. В этом документе приведены пошаговые инструкции по созданию проекта Storm в Visual Studio, его локальному тестированию и развертыванию в Apache Storm в кластере Azure HDInsight.

Вы также узнаете о том, как создавать гибридные топологии, использующие компоненты C# и Java.

Топологии C# используют .NET 4,5 и используют Mono для запуска в кластере HDInsight. Сведения о возможных несовместимости см. в разделе [Совместимость с Mono](https://www.mono-project.com/docs/about-mono/compatibility/). Чтобы использовать топологию C#, необходимо обновить `Microsoft.SCP.Net.SDK` пакет NuGet, используемый в проекте, до версии 0.10.0.6 или более поздней. Версия пакета также должна соответствовать основной версии Storm, установленной на HDInsight.

| Версия HDInsight | Версия Apache Storm | Версия SCP.NET | Версия Mono по умолчанию |
|:-----------------:|:-------------:|:---------------:|:--------------------:|
| 3.4 | 0.10.0.x | 0.10.0.x | 3.2.8 |
| 3,5 | 1.0.2.x | 1.0.0.x | 4.2.1 |
| 3,6 | 1.1.0.x | 1.0.0.x | 4.2.8 |

## <a name="prerequisite"></a>Предварительное требование

Кластер Apache Storm в HDInsight. Ознакомьтесь со статьей [Create Linux-based clusters in HDInsight by using the Azure portal](../hdinsight-hadoop-create-linux-clusters-portal.md) (Создание кластеров под управлением Linux в HDInsight с помощью портала Azure) и выберите **Storm** для параметра **Тип кластера**.

## <a name="install-visual-studio"></a>Установка Visual Studio

Вы можете разрабатывать топологии C# с помощью SCP.NET, используя [Visual Studio](https://visualstudio.microsoft.com/downloads/). В приведенных здесь инструкциях используется Visual Studio 2019, но вы также можете использовать более ранние версии Visual Studio.

## <a name="install-data-lake-tools-for-visual-studio"></a>Установка средств Data Lake для Visual Studio

Инструкции по установке средств Data Lake для Visual Studio см. в статье [Подключение к Azure HDInsight и выполнение запросов Hive с помощью средств Data Lake для Visual Studio](../hadoop/apache-hadoop-visual-studio-tools-get-started.md#install-data-lake-tools-for-visual-studio).

## <a name="install-java"></a>Установка Java

При отправке топологии Storm из Visual Studio SCP.NET создает ZIP-файл, содержащий топологию и зависимости. Эти ZIP-файлы создаются с помощью Java, так как при этом используется формат, который более совместим с кластерами под управлением Linux.

1. Установите Java Developer Kit (JDK) 7 или более поздней версии в среде разработки. Получить пакет Oracle JDK можно на сайте [Oracle](https://openjdk.java.net/). Вы также можете использовать [другие дистрибутивы Java](/java/azure/jdk/).

2. Задайте `JAVA_HOME` для переменной среды каталог, содержащий Java.

3. Задайте `PATH` переменную среды, чтобы включить `%JAVA_HOME%\bin` каталог.

Вы можете создать и запустить следующее консольное приложение C#, чтобы убедиться, что Java и JDK установлены правильно:

```csharp
using System;
using System.IO;
namespace ConsoleApplication2
{
   class Program
   {
       static void Main(string[] args)
       {
           string javaHome = Environment.GetEnvironmentVariable("JAVA_HOME");
           if (!string.IsNullOrEmpty(javaHome))
           {
               string jarExe = Path.Combine(javaHome + @"\bin", "jar.exe");
               if (File.Exists(jarExe))
               {
                   Console.WriteLine("JAVA Is Installed properly");
                    return;
               }
               else
               {
                   Console.WriteLine("A valid JAVA JDK is not found. Looks like JRE is installed instead of JDK.");
               }
           }
           else
           {
             Console.WriteLine("A valid JAVA JDK is not found. JAVA_HOME environment variable is not set.");
           }
       }  
   }
}
```

## <a name="apache-storm-templates"></a>Шаблоны Apache Storm

Средства Data Lake для Visual Studio предоставляют следующие шаблоны:

| Тип проекта | Что демонстрирует |
| --- | --- |
| Приложение Storm |Пустой проект топологии Storm |
| Пример модуля записи Storm Azure SQL |Запись в базу данных SQL Azure. |
| Пример модуля чтения Storm Azure Cosmos DB |Чтение из базы данных Azure Cosmos DB. |
| Пример модуля записи Storm Azure Cosmos DB |Запись в базу данных Azure Cosmos DB. |
| Пример модуля чтения концентратора событий Storm |Чтение из Центров событий Azure. |
| Пример модуля записи концентратора событий Storm |Запись в Центры событий Azure. |
| Пример модуля чтения Storm HBase |Чтение из HBase в кластерах HDInsight. |
| Пример модуля записи Storm HBase |Запись в HBase в кластерах HDInsight. |
| Пример Storm Hybrid |Использование компонентов Java. |
| Пример Storm |Базовая топология подсчета слов. |

> [!WARNING]  
> Не все шаблоны работают с HDInsight под управлением Linux. Пакеты NuGet, используемые в шаблонах, могут оказаться несовместимыми с Mono. Чтобы определить потенциальные проблемы, см. раздел [Совместимость с Mono](https://www.mono-project.com/docs/about-mono/compatibility/) и использование [анализатора переносимости .NET](../hdinsight-hadoop-migrate-dotnet-to-linux.md#automated-portability-analysis).

На данном этапе в этом документе используется базовый тип проекта приложения Storm для создания топологии.

### <a name="apache-hbase-templates"></a>Шаблоны Apache HBase

Для взаимодействия с HBase в кластере HDInsight шаблоны модулей чтения и записи HBase используют REST API HBase, а не API Java HBase.

### <a name="eventhub-templates"></a>Шаблоны EventHub

> [!IMPORTANT]  
> Компонент spout EventHub на основе Java, входящий в состав шаблона EventHub Reader, не работает со Storm в HDInsight версии 3.5 или более поздней. Обновленная версия этого компонента доступна в [GitHub](https://github.com/hdinsight/hdinsight-storm-examples/tree/master/HDI3.5/lib).

Пример топологии, которая использует этот компонент и работает со Storm в HDInsight 3.5, см. в [GitHub](https://github.com/Azure-Samples/hdinsight-dotnet-java-storm-eventhub).

## <a name="create-a-c-topology"></a>Создание топологии на C#

Чтобы создать проект топологии C# в Visual Studio, выполните следующие действия.

1. Запустите Visual Studio.

1. В окне **Начало работы** выберите **Создать проект**.

1. В окне **Создание нового проекта** перейдите к приложению и выберите **приложение**, а затем нажмите кнопку **Далее**.

1. В окне **Настройка нового проекта** введите **имя проекта** *WordCount*, перейдите на страницу или создайте путь к каталогу **расположения** для проекта, а затем нажмите кнопку **создать**.

    ![Приложение для работы с проектами, диалоговое окно "Настройка нового проекта", Visual Studio](./media/apache-storm-develop-csharp-visual-studio-topology/apache-storm-new-project.png)

После создания проекта у вас должны быть следующие файлы:

* *Program. CS*: Определение топологии для проекта. Обратите внимание, что по умолчанию создается топология, состоящая из одного объекта spout и одного объекта bolt.

* *Spout.cs*— пример воронки, выдающей случайные числа.

* *Bolt.cs*— пример сита, которое подсчитывает числа, созданные воронкой.

При создании проекта NuGet скачивает последнюю версию [пакета SCP.NET](https://www.nuget.org/packages/Microsoft.SCP.Net.SDK/).

### <a name="implement-the-spout"></a>Реализация воронки

Затем добавьте код для spout, который используется для чтения данных в топологии из внешнего источника. Эта воронка случайным образом отправляет предложения в топологию.

1. Откройте файл *Spout.cs*. Ниже перечислены основные компоненты воронки.

   * `NextTuple`: Вызывается методом, если spout разрешается создавать новые кортежи.

   * `Ack` (только для транзакционной топологии): обрабатывает подтверждения, запущенные другими компонентами в топологии для кортежей, отправленных из spout. Подтверждение кортежа позволяет воронке определить, что он был успешно обработан нижестоящими компонентами.

   * `Fail` (только для транзакционной топологии): обрабатывает кортежи, которые не проходят обработку других компонентов в топологии. Реализация `Fail` метода позволяет повторно выдавать кортеж, чтобы его можно было обработать снова.

2. Замените содержимое `Spout` класса следующим текстом:

    ```csharp
    private Context ctx;
    private Random r = new Random();
    string[] sentences = new string[] {
        "the cow jumped over the moon",
        "an apple a day keeps the doctor away",
        "four score and seven years ago",
        "snow white and the seven dwarfs",
        "i am at two with nature"
    };

    public Spout(Context ctx)
    {
        // Set the instance context
        this.ctx = ctx;

        Context.Logger.Info("Generator constructor called");

        // Declare Output schema
        Dictionary<string, List<Type>> outputSchema = new Dictionary<string, List<Type>>();
        // The schema for the default output stream is
        // a tuple that contains a string field
        outputSchema.Add("default", new List<Type>() { typeof(string) });
        this.ctx.DeclareComponentSchema(new ComponentStreamSchema(null, outputSchema));
    }

    // Get an instance of the spout
    public static Spout Get(Context ctx, Dictionary<string, Object> parms)
    {
        return new Spout(ctx);
    }

    public void NextTuple(Dictionary<string, Object> parms)
    {
        Context.Logger.Info("NextTuple enter");
        // The sentence to be emitted
        string sentence;

        // Get a random sentence
        sentence = sentences[r.Next(0, sentences.Length - 1)];
        Context.Logger.Info("Emit: {0}", sentence);
        // Emit it
        this.ctx.Emit(new Values(sentence));

        Context.Logger.Info("NextTuple exit");
    }

    public void Ack(long seqId, Dictionary<string, Object> parms)
    {
        // Only used for transactional topologies
    }

    public void Fail(long seqId, Dictionary<string, Object> parms)
    {
        // Only used for transactional topologies
    }
    ```

### <a name="implement-the-bolts"></a>Реализация сит

Теперь создайте два винта в этом примере:

1. Удалите существующий файл *Bolt.cs* из проекта.

2. В **Обозреватель решений** щелкните правой кнопкой мыши проект и выберите команду **Добавить**  >  **новый элемент**. Выберите в списке **Storm Bolt** (Сито воронки) и введите имя *Splitter.cs*. В коде нового файла измените имя пространства имен на `WordCount` . Затем повторите эту процедуру, чтобы создать вторую молнию с именем *Counter. CS*.

   * *Splitter. CS*: реализует молнию, которая разделяет предложения на отдельные слова и создает новый поток слов.

   * *Counter. CS*: реализуется, который подсчитывает каждое слово и создает новый поток слов и число для каждого слова.

     > [!NOTE]  
     > Хотя эти сита считывают данные и записывают их в потоки, их можно также использовать для взаимодействия с базой данных, службой и т. д.

3. Откройте файл *Splitter.cs*. По умолчанию у него есть только один метод: `Execute` . `Execute`Метод вызывается, когда молния получает кортеж для обработки. Здесь можно считывать и обрабатывать входящие кортежи и создавать исходящие кортежи.

4. Замените содержимое класса `Splitter` следующим кодом:

    ```csharp
    private Context ctx;

    // Constructor
    public Splitter(Context ctx)
    {
        Context.Logger.Info("Splitter constructor called");
        this.ctx = ctx;

        // Declare Input and Output schemas
        Dictionary<string, List<Type>> inputSchema = new Dictionary<string, List<Type>>();
        // Input contains a tuple with a string field (the sentence)
        inputSchema.Add("default", new List<Type>() { typeof(string) });
        Dictionary<string, List<Type>> outputSchema = new Dictionary<string, List<Type>>();
        // Outbound contains a tuple with a string field (the word)
        outputSchema.Add("default", new List<Type>() { typeof(string) });
        this.ctx.DeclareComponentSchema(new ComponentStreamSchema(inputSchema, outputSchema));
    }

    // Get a new instance of the bolt
    public static Splitter Get(Context ctx, Dictionary<string, Object> parms)
    {
        return new Splitter(ctx);
    }

    // Called when a new tuple is available
    public void Execute(SCPTuple tuple)
    {
        Context.Logger.Info("Execute enter");

        // Get the sentence from the tuple
        string sentence = tuple.GetString(0);
        // Split at space characters
        foreach (string word in sentence.Split(' '))
        {
            Context.Logger.Info("Emit: {0}", word);
            //Emit each word
            this.ctx.Emit(new Values(word));
        }

        Context.Logger.Info("Execute exit");
    }
    ```

5. Откройте файл *Counter.cs* и замените содержимое класса следующим кодом:

    ```csharp
    private Context ctx;

    // Dictionary for holding words and counts
    private Dictionary<string, int> counts = new Dictionary<string, int>();

    // Constructor
    public Counter(Context ctx)
    {
        Context.Logger.Info("Counter constructor called");
        // Set instance context
        this.ctx = ctx;

        // Declare Input and Output schemas
        Dictionary<string, List<Type>> inputSchema = new Dictionary<string, List<Type>>();
        // A tuple containing a string field - the word
        inputSchema.Add("default", new List<Type>() { typeof(string) });

        Dictionary<string, List<Type>> outputSchema = new Dictionary<string, List<Type>>();
        // A tuple containing a string and integer field - the word and the word count
        outputSchema.Add("default", new List<Type>() { typeof(string), typeof(int) });
        this.ctx.DeclareComponentSchema(new ComponentStreamSchema(inputSchema, outputSchema));
    }

    // Get a new instance
    public static Counter Get(Context ctx, Dictionary<string, Object> parms)
    {
        return new Counter(ctx);
    }

    // Called when a new tuple is available
    public void Execute(SCPTuple tuple)
    {
        Context.Logger.Info("Execute enter");

        // Get the word from the tuple
        string word = tuple.GetString(0);
        // Do we already have an entry for the word in the dictionary?
        // If no, create one with a count of 0
        int count = counts.ContainsKey(word) ? counts[word] : 0;
        // Increment the count
        count++;
        // Update the count in the dictionary
        counts[word] = count;

        Context.Logger.Info("Emit: {0}, count: {1}", word, count);
        // Emit the word and count information
        this.ctx.Emit(Constants.DEFAULT_STREAM_ID, new List<SCPTuple> { tuple }, new Values(word, count));
        Context.Logger.Info("Execute exit");
    }
    ```

### <a name="define-the-topology"></a>Определение топологии

Потоки данных между компонентами могут быть представлены в виде диаграммы воронок и сит. Граф для этой топологии выглядит следующим образом.

![Схема размещения компонентов spout и молнии, топология с графикой](./media/apache-storm-develop-csharp-visual-studio-topology/word-count-topology1.png)

Spout выдает предложения, которые распространяются на экземпляры разделителя. Сито Splitter разбивает предложения на слова, которые распределяются среди экземпляров сита Counter.

Так как экземпляр счетчика содержит число слов в локальной среде, необходимо убедиться, что определенные слова поступают в один и тот же экземпляр счетчика молнии. Каждое конкретное слово отслеживается только одним экземпляром. Так как сито Splitter не поддерживает состояние, на самом деле неважно, какое предложение получает то или иное сито.

Откройте файл *Program.cs*. Важным методом является `GetTopologyBuilder` , который используется для определения топологии, которая отправляется в самое большое. Замените содержимое `GetTopologyBuilder` следующим кодом, чтобы реализовать топологию, описанную ранее.

```csharp
// Create a new topology named 'WordCount'
TopologyBuilder topologyBuilder = new TopologyBuilder(
    "WordCount" + DateTime.Now.ToString("yyyyMMddHHmmss"));

// Add the spout to the topology.
// Name the component 'sentences'
// Name the field that is emitted as 'sentence'
topologyBuilder.SetSpout(
    "sentences",
    Spout.Get,
    new Dictionary<string, List<string>>()
    {
        {Constants.DEFAULT_STREAM_ID, new List<string>(){"sentence"}}
    },
    1);
// Add the splitter bolt to the topology.
// Name the component 'splitter'
// Name the field that is emitted 'word'
// Use suffleGrouping to distribute incoming tuples
//   from the 'sentences' spout across instances
//   of the splitter
topologyBuilder.SetBolt(
    "splitter",
    Splitter.Get,
    new Dictionary<string, List<string>>()
    {
        {Constants.DEFAULT_STREAM_ID, new List<string>(){"word"}}
    },
    1).shuffleGrouping("sentences");

// Add the counter bolt to the topology.
// Name the component 'counter'
// Name the fields that are emitted 'word' and 'count'
// Use fieldsGrouping to ensure that tuples are routed
//   to counter instances based on the contents of field
//   position 0 (the word). This could also have been
//   List<string>(){"word"}.
//   This ensures that the word 'jumped', for example, will always
//   go to the same instance
topologyBuilder.SetBolt(
    "counter",
    Counter.Get,
    new Dictionary<string, List<string>>()
    {
        {Constants.DEFAULT_STREAM_ID, new List<string>(){"word", "count"}}
    },
    1).fieldsGrouping("splitter", new List<int>() { 0 });

// Add topology config
topologyBuilder.SetTopologyConfig(new Dictionary<string, string>()
{
    {"topology.kryo.register","[\"[B\"]"}
});

return topologyBuilder;
```

## <a name="submit-the-topology"></a>Отправка топологии

Теперь вы готовы отправить топологию в кластер HDInsight.

1. Перейдите к **просмотру**  >  **Обозреватель сервера**.

1. Щелкните правой кнопкой мыши **Azure**, выберите **подключиться к Microsoft Azure подписке...** и завершите процесс входа.

1. В **Обозреватель решений** щелкните правой кнопкой мыши проект и выберите **отправить в кластер HDInsight**.

1. В диалоговом окне **Отправка топологии** в раскрывающемся списке **кластер кластера** выберите свой вариант в кластере HDInsight, а затем нажмите кнопку **Отправить**. Можно проверить успешность отправки, просмотрев область **вывода** .

    После успешной отправки топологии появится окно **представления топологии** для кластера. Выберите топологию **WordCount** из списка, чтобы просмотреть сведения о работающей топологии.

    ![Окно представления топологии "дерево", кластер HDInsight, Visual Studio](./media/apache-storm-develop-csharp-visual-studio-topology/storm-topology-view.png)

    > [!NOTE]  
    > **Топологии Storm** можно также просмотреть в **обозревателе сервера**. Разверните **Azure**  >  **hdinsight**, щелкните правой кнопкой мыши множество в кластере HDInsight, а затем выберите пункт **Просмотр топологий**.

    Чтобы просмотреть сведения о компонентах в топологии, выберите компонент на схеме.

1. В разделе " **Сводка топологии** " выберите **Kill** , чтобы остановить топологию.

    > [!NOTE]  
    > Топологии Storm выполняются, пока не будут удалены или пока не будет удален кластер.

## <a name="transactional-topology"></a>Транзакционная топология

Предыдущая топология является нетранзакционной. Компоненты в топологии не реализуют функции для воспроизведения сообщений. Чтобы получить пример транзакционной топологии, создайте проект и выберите в качестве типа проекта **Пример Storm**.

Транзакционные топологии реализуют следующие возможности, обеспечивающие воспроизведение данных:

* **Кэширование метаданных**. spout должен хранить метаданные о выдаваемых данных, чтобы данные могли быть извлечены и переданы снова при возникновении сбоя. Так как в этом примере отправляются данные небольшого размера, необработанные данные каждого кортежа хранятся в словаре для воспроизведения.

* **Подтверждение** — каждый элемент bolt в топологии может вызвать `this.ctx.Ack(tuple)`, чтобы подтвердить успешную обработку кортежа. Когда все винты подтверждают кортеж, `Ack` вызывается метод spout. Метод `Ack` позволяет воронке удалить кэшированные данные для воспроизведения.

* **Fail** (Сбой) — каждое сито может вызвать `this.ctx.Fail(tuple)`, указав тем самым на сбой при обработке кортежа. Информация о сбое передается в метод `Fail` воронки, в которой кортеж может быть воспроизведен с помощью кэшированных метаданных.

* **Sequence ID**(Идентификатор последовательности) — при отправке кортежа может быть указан уникальный идентификатор последовательности. Это значение определяет кортеж для обработки воспроизведения ( `Ack` и `Fail` ). Например, при выдаче данных в **примере** проекта spout используется следующий вызов метода:

  `this.ctx.Emit(Constants.DEFAULT_STREAM_ID, new Values(sentence), lastSeqId);`

  Этот код создает кортеж, содержащий предложение в поток по умолчанию, со значением идентификатора последовательности, содержащимся в `lastSeqId` . В этом примере `lastSeqId` увеличивается на единицу для каждого порожденного кортежа.

Как показано в проекте **Пример Storm**, транзакционность компонента можно задать во время выполнения в соответствии с конфигурацией.

## <a name="hybrid-topology-with-c-and-java"></a>Гибридная топология с помощью C# и Java

Средства Data Lake для Visual Studio можно также использовать для создания гибридных топологий, в которых одни компоненты написаны на C#, а другие — на Java.

Чтобы получить пример гибридной топологии, создайте проект и выберите **Пример гибридного решения Storm**. В примере демонстрируются следующие понятия:

* **Java spout** и **C# молния**: определяются в `HybridTopology_javaSpout_csharpBolt` классе.

  Транзакционная версия определяется в `HybridTopologyTx_javaSpout_csharpBolt` классе.

* **C# spout** и **Java**: определены в `HybridTopology_csharpSpout_javaBolt` классе.

  Транзакционная версия определяется в `HybridTopologyTx_csharpSpout_javaBolt` классе.

  > [!NOTE]  
  > В этой версии также демонстрируется использование кода Clojure из текстового файла в качестве компонента Java.

Чтобы изменить топологию, используемую при отправке проекта, перед отправкой в кластер переместите оператор `[Active(true)]` в топологию, которую необходимо использовать.

> [!NOTE]  
> Все необходимые файлы Java предоставляются в составе этого проекта и находятся в папке *JavaDependency* .

При создании и отправке гибридной топологии учитывайте следующее.

* Используйте `JavaComponentConstructor` для создания экземпляра класса Java для spout или молнии.

* Используется `microsoft.scp.storm.multilang.CustomizedInteropJSONSerializer` для сериализации данных в компоненты Java или из них из объектов Java в JSON.

* Отправляя топологию на сервер, необходимо использовать параметр **Дополнительные конфигурации**, чтобы задать **пути к файлам Java**. Указанный путь должен быть каталогом с JAR-файлами, содержащими классы Java.

### <a name="azure-event-hubs"></a>Центры событий Azure

В SCP.NET версии 0.9.4.203 появились новые класс и метод, предназначенные специально для работы с компонентом spout концентратора событий (компонентом spout Java, который считывает данные из Центров событий). При создании топологии, в которой используется spout концентратора событий (например, с помощью шаблона " **средство чтения EventHub** " в наборе), используйте следующие API:

* `EventHubSpoutConfig` класс: создает объект, который содержит конфигурацию для компонента spout.

* `TopologyBuilder.SetEventHubSpout` метод: добавляет компонент spout концентратора событий в топологию.

> [!NOTE]  
> По-прежнему необходимо использовать `CustomizedInteropJSONSerializer` для сериализации данных, создаваемых spout.

## <a name="use-configurationmanager"></a>Использование ConfigurationManager

Не используйте **ConfigurationManager** для получения значений конфигурации из компонентов молнии и spout. Это вызовет исключение пустого указателя. Вместо этого передайте конфигурацию проекта в топологию, в качестве пары ключей и значений в контексте топологии. Каждому компоненту, зависящему от значений конфигурации, необходимо получить их из контекста во время инициализации.

В приведенном ниже коде показано, как получить эти значения.

```csharp
public class MyComponent : ISCPBolt
{
    // To hold configuration information loaded from context
    Configuration configuration;
    ...
    public MyComponent(Context ctx, Dictionary<string, Object> parms)
    {
        // Save a copy of the context for this component instance
        this.ctx = ctx;
        // If it exists, load the configuration for the component
        if(parms.ContainsKey(Constants.USER_CONFIG))
        {
            this.configuration = parms[Constants.USER_CONFIG] 
                as System.Configuration.Configuration;
        }
        // Retrieve the value of "Foo" from configuration
        var foo = this.configuration.AppSettings.Settings["Foo"].Value;
    }
    ...
}
```

Если используется метод `Get`, чтобы вернуть экземпляр компонента, он должен передавать конструктору параметры `Context` и `Dictionary<string, Object>`. В примере ниже показан базовый метод `Get`, передающий значения должным образом.

```csharp
public static MyComponent Get(Context ctx, Dictionary<string, Object> parms)
{
    return new MyComponent(ctx, parms);
}
```

## <a name="how-to-update-scpnet"></a>Обновление SCP.NET

Последние версии SCP.NET поддерживают обновление пакета через NuGet. Если доступно новое обновление, вы получите уведомление об обновлении. Чтобы вручную проверить обновления, выполните указанные ниже действия.

1. В **Обозреватель решений** щелкните правой кнопкой мыши проект и выберите **Управление пакетами NuGet**.

2. Из диспетчера пакетов выберите **Обновления**. Если обновление для пакета поддержки SCP.NET доступно, оно отображается в списке. Выберите **Обновить** для пакета, а затем в диалоговом окне **Предварительный просмотр изменений** нажмите кнопку **ОК** , чтобы установить его.

> [!IMPORTANT]  
> Если проект был создан с помощью предыдущей версии SCP.NET, в которой не использовалось средство NuGet, выполните следующие действия для обновления до новой версии.
>
> 1. В **Обозреватель решений** щелкните правой кнопкой мыши проект и выберите **Управление пакетами NuGet**.
> 2. С помощью поля **поиска** найдите и добавьте `Microsoft.SCP.Net.SDK` в проект.

## <a name="troubleshoot-common-issues-with-topologies"></a>Устранение распространенных неполадок с топологиями

### <a name="null-pointer-exceptions"></a>Исключение пустого указателя

Если вы используете топологию C# с кластером HDInsight под управлением Linux, компоненты spout, использующие **ConfigurationManager** для чтения параметров конфигурации во время выполнения, могут возвращать исключения пустых указателей.

Конфигурация проекта передается в топологию Storm как пара "ключ —значение" в контексте топологии. Его можно получить из объекта Dictionary, который передается в компоненты при инициализации.

Дополнительные сведения см. в разделе [use ConfigurationManager](#use-configurationmanager) этого документа.

### <a name="systemtypeloadexception"></a>System.TypeLoadException

Если вы используете топологию C# с кластером HDInsight под управлением Linux, может возникнуть следующая ошибка:

`System.TypeLoadException: Failure has occurred while loading a type.`

Эта ошибка возникает при использовании двоичного файла, который не совместим с версией .NET, поддерживаемой Mono.

Для кластеров HDInsight под управлением Linux в проекте должны использоваться двоичные файлы, скомпилированные для .NET 4.5.

### <a name="test-a-topology-locally"></a>Локальная проверка топологии

Хотя топологию можно легко развернуть в кластере, в некоторых случаях может потребоваться локальное тестирование топологии. Выполните следующие действия, чтобы выполнить и проверить пример топологии в этой статье локально в вашей среде разработки.

> [!WARNING]  
> Локальное тестирование работает только для базовых топологий на C#. Нельзя использовать локальное тестирование для гибридных топологий или топологий, использующих несколько потоков.

1. В **обозревателе решений** щелкните правой кнопкой мыши проект и выберите **Свойства**. В свойствах проекта. Затем измените **тип выходных данных** на **консольное приложение**.

   ![Приложение для HDInsight, свойства проекта, тип вывода](./media/apache-storm-develop-csharp-visual-studio-topology/hdi-output-type-window.png)

   > [!NOTE]
   > Перед развертыванием топологии в кластере не забудьте вернуть для параметра **Тип выходных данных** значение **Библиотеки классов**.

1. В **обозревателе решений** щелкните проект правой кнопкой мыши и выберите пункты **Добавить** > **Новый элемент**. Выберите **класс** и введите *localtest следующий код. CS* в качестве имени класса. Наконец, выберите **Добавить**.

1. Откройте *localtest следующий код. CS* и добавьте в начало следующую `using` инструкцию:

    ```csharp
    using Microsoft.SCP;
    ```

1. Используйте следующий код в качестве содержимого `LocalTest` класса:

    ```csharp
    // Drives the topology components
    public void RunTestCase()
    {
        // An empty dictionary for use when creating components
        Dictionary<string, Object> emptyDictionary = new Dictionary<string, object>();

        #region Test the spout
        {
            Console.WriteLine("Starting spout");
            // LocalContext is a local-mode context that can be used to initialize
            // components in the development environment.
            LocalContext spoutCtx = LocalContext.Get();
            // Get a new instance of the spout, using the local context
            Spout sentences = Spout.Get(spoutCtx, emptyDictionary);

            // Emit 10 tuples
            for (int i = 0; i < 10; i++)
            {
                sentences.NextTuple(emptyDictionary);
            }
            // Use LocalContext to persist the data stream to file
            spoutCtx.WriteMsgQueueToFile("sentences.txt");
            Console.WriteLine("Spout finished");
        }
        #endregion

        #region Test the splitter bolt
        {
            Console.WriteLine("Starting splitter bolt");
            // LocalContext is a local-mode context that can be used to initialize
            // components in the development environment.
            LocalContext splitterCtx = LocalContext.Get();
            // Get a new instance of the bolt
            Splitter splitter = Splitter.Get(splitterCtx, emptyDictionary);

            // Set the data stream to the data created by the spout
            splitterCtx.ReadFromFileToMsgQueue("sentences.txt");
            // Get a batch of tuples from the stream
            List<SCPTuple> batch = splitterCtx.RecvFromMsgQueue();
            // Process each tuple in the batch
            foreach (SCPTuple tuple in batch)
            {
                splitter.Execute(tuple);
            }
            // Use LocalContext to persist the data stream to file
            splitterCtx.WriteMsgQueueToFile("splitter.txt");
            Console.WriteLine("Splitter bolt finished");
        }
        #endregion

        #region Test the counter bolt
        {
            Console.WriteLine("Starting counter bolt");
            // LocalContext is a local-mode context that can be used to initialize
            // components in the development environment.
            LocalContext counterCtx = LocalContext.Get();
            // Get a new instance of the bolt
            Counter counter = Counter.Get(counterCtx, emptyDictionary);

            // Set the data stream to the data created by splitter bolt
            counterCtx.ReadFromFileToMsgQueue("splitter.txt");
            // Get a batch of tuples from the stream
            List<SCPTuple> batch = counterCtx.RecvFromMsgQueue();
            // Process each tuple in the batch
            foreach (SCPTuple tuple in batch)
            {
                counter.Execute(tuple);
            }
            // Use LocalContext to persist the data stream to file
            counterCtx.WriteMsgQueueToFile("counter.txt");
            Console.WriteLine("Counter bolt finished");
        }
        #endregion
    }
    ```

    Ознакомьтесь с комментариями в коде. Этот код использует `LocalContext` для запуска компонентов в среде разработки. Он сохраняет поток данных между компонентами в текстовые файлы на локальном диске.

1. Откройте *программу Program. CS* и добавьте в метод следующий код `Main` :

    ```csharp
    Console.WriteLine("Starting tests");
    System.Environment.SetEnvironmentVariable("microsoft.scp.logPrefix", "WordCount-LocalTest");
    // Initialize the runtime
    SCPRuntime.Initialize();

    //If we are not running under the local context, throw an error
    if (Context.pluginType != SCPPluginType.SCP_NET_LOCAL)
    {
        throw new Exception(string.Format("unexpected pluginType: {0}", Context.pluginType));
    }
    // Create test instance
    LocalTest tests = new LocalTest();
    // Run tests
    tests.RunTestCase();
    Console.WriteLine("Tests finished");
    Console.ReadKey();
    ```

1. Сохраните изменения, а затем нажмите **клавишу F5** или выберите **Отладка**  >  **начать отладку** , чтобы запустить проект. Должно появится окно консоли, а также данные о состоянии журнала в ходе выполнения тестов. Когда `Tests finished` отобразится, выберите любой ключ, чтобы закрыть окно.

1. Используйте **проводник** для поиска каталога, который содержит проект, (Например, *C: \\ Users \\ \<your_user_name> \\ \\ репозиториев \\ \\* WordCount WordCount.) Затем в этом каталоге откройте *bin* и выберите *Отладка*. Вы должны увидеть текстовые файлы, созданные при выполнении тестов: *sentences.txt*, *counter.txt* и *splitter.txt*. Откройте каждый текстовый файл и проверьте данные.

   > [!NOTE]  
   > Строковые данные сохраняются в этих файлах как массив значений типа decimal. Например, `[[97,103,111]]` в файле **splitter.txt** представляет слово « *назад*».

> [!NOTE]  
> Перед развертыванием в кластере HDInsight не забудьте вернуть **тип проекта** к **библиотеке классов** в свойствах проекта.

### <a name="log-information"></a>Сведения о журнале

С помощью `Context.Logger`можно легко записывать информацию из компонентов топологии в журнал. Например, приведенная ниже команда создает информационную запись журнала:

`Context.Logger.Info("Component started");`

Записанную информацию можно просмотреть в **журнале службы Hadoop**, который можно найти в **обозревателе сервера**. Разверните запись для Storm в кластере HDInsight, а затем — **Журнал службы Hadoop**. Выберите файла журнала для просмотра.

> [!NOTE]  
> Журналы хранятся в используемой в кластере учетной записи хранения Azure. Для просмотра журналов в Visual Studio необходимо войти в подписку Azure, содержащую эту учетную запись хранения.

### <a name="view-error-information"></a>Просмотр информации об ошибке

Чтобы просмотреть ошибки, произошедшие в работающей топологии, сделайте следующее.

1. В **Обозреватель сервера** щелкните его в кластере HDInsight правой кнопкой мыши и выберите пункт **Просмотр топологий**.

   Для элементов **spout** и **bolt** столбец **Последняя ошибка** содержит сведения о последней ошибке.

2. Выберите **идентификатор spout** или **идентификатор молнии** для компонента, в котором содержится ошибка. На странице сведений отображаются дополнительные сведения об ошибке в разделе **ошибки** в нижней части страницы.

3. Чтобы получить дополнительные сведения, выберите **порт** в разделе **исполнители** страницы, чтобы просмотреть журнал рабочей роли для более последних минут.

### <a name="errors-submitting-topologies"></a>Ошибки при отправке топологии

Если при отправке топологии в HDInsight возникают ошибки, можно найти журналы серверных компонентов, которые обрабатывали передачу топологии в кластере HDInsight. Чтобы скачать эти журналы, используйте следующую команду из командной строки:

```cmd
scp sshuser@clustername-ssh.azurehdinsight.net:/var/log/hdinsight-scpwebapi/hdinsight-scpwebapi.out .
```

Замените *sshuser* именем учетной записи пользователя SSH для кластера. Замените *имя_кластера* именем кластера HDInsight. Дополнительные сведения об использовании `scp` и `ssh` с HDInsight см. в статье [Подключение к HDInsight (Hadoop) с помощью SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

Отправка может завершиться ошибкой по нескольким причинам:

* JDK не установлен или не находится в пути.
* Необходимые зависимости Java не включаются в отправку.
* Зависимости несовместимы.
* Имена топологий дублируются.

Если файл журнала *хдинсигхт-скпвебапи. out* содержит `FileNotFoundException` , то исключение может быть вызвано следующими причинами.

* JDK не находится в пути в среде разработки. Убедитесь в том, что пакет JDK установлен в среде разработки, а `%JAVA_HOME%/bin` находится по требуемому пути.
* У вас отсутствует зависимость Java. Убедитесь, что вы включаете все необходимые JAR-файлы в состав отправки.

## <a name="next-steps"></a>Дальнейшие действия

Пример обработки данных из Центров событий см. в статье [Обработка событий из Центров событий Azure с помощью Storm в HDInsight](apache-storm-develop-csharp-event-hub-topology.md).

Пример топологии C#, которая разбивает поток данных на несколько потоков, см. на странице с [примером C# Storm](https://github.com/Blackmist/csharp-storm-example).

Дополнительные сведения о создании топологий C# см. в [GitHub](https://github.com/hdinsight/hdinsight-storm-examples/blob/master/SCPNet-GettingStarted.md).

Другие методы работы с HDInsight и дополнительные примеры Storm в HDInsight см. в следующих статьях.

**Microsoft SCP.NET**

* [Инструкции по программированию SCP для Apache Storm в Azure HDInsight](apache-storm-scp-programming-guide.md)

**Apache Storm в HDInsight**

* [Deploy and manage Apache Storm topologies on Azure HDInsight](apache-storm-deploy-monitor-topology-linux.md) (Развертывание топологий Apache Storm и управление ими в Azure HDInsight)
* [Примеры топологий Apache Storm в Azure HDInsight](apache-storm-example-topology.md)

**Apache Hadoop в HDInsight**

* [Обзор Apache Hive и HiveQL в Azure HDInsight](../hadoop/hdinsight-use-hive.md)
* [Использование MapReduce в Apache Hadoop в HDInsight](../hadoop/hdinsight-use-mapreduce.md)

**Apache HBase в HDInsight**

* [использованию Apache HBase в Azure HDInsight](../hbase/apache-hbase-tutorial-get-started-linux.md)
