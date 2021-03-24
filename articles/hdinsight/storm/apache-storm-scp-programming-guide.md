---
title: Руководство по программированию с использованием SCP.NET для Storm в HDInsight Azure
description: Узнайте, как использовать SCP.NET для создания топологий Storm на основе .NET для использования со Storm под управлением Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-csharp
ms.date: 01/13/2020
ms.openlocfilehash: c993b3f70f609fb79c51ba9be08fa3d5dc7e8317
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104864114"
---
# <a name="scp-programming-guide-for-apache-storm-in-azure-hdinsight"></a>Инструкции по программированию SCP для Apache Storm в Azure HDInsight

SCP — это платформа для создания приложений для обработки данных в реальном времени, надежных, единообразных и высокопроизводительных. Он основан на [Apache Storm](https://storm.incubator.apache.org/), который является системой обработки потоков, разработанной сообществом программного обеспечения с открытым исходным кодом. (Nathan Marz. Он был опубликован в Twitter как открытый исходный код. Оно использует [Apache ZooKeeper](https://zookeeper.apache.org/), который является еще одним проектом Apache, который обеспечивает надежную распределенную координацию и управление состоянием.

Проект SCP был перенесен не только на Windows, но также добавлен в проект расширения и настройку для среды Windows. Расширения включают в себя среду разработки .NET и библиотеки .NET. Настройка включает развертывание на основе Windows.

Благодаря расширениям и настройке вам не нужно развилкировать проекты программного обеспечения с открытым исходным кодом. Можно использовать производные среды, построенные на основе набора.

## <a name="processing-model"></a>Модель обработки данных

Данные в SCP оформляются в виде постоянных потоков кортежей данных. Как правило, кортежи:

1. Потоковая передача в очередь.
1. Выбираются и преобразуются бизнес-логикой, размещенной внутри топологии.
1. Их выходные данные передаются в виде кортежей в другую систему SCP или фиксируются в хранилищах, таких как распределенные файловые системы и базы данных, такие как SQL Server.

:::image type="content" source="./media/apache-storm-scp-programming-guide/queue-feeding-data-to-processing-to-data-store.png" alt-text="Диаграмма очереди, подающее данные для обработки, которые, в свою очередь, формируют хранилище данных." border="false":::

В этом случае топология приложения определяет вычислительную диаграмму. Каждый узел в топологии содержит логику обработки. Связи между узлами указывают на поток данных.

Узлы, которые вставляют входные данные в топологию, называются _выводами_. Их можно использовать для последовательного упорядочения данных. Входные данные могут поступать из таких источников, как журналы файлов, транзакционная база данных или счетчик производительности системы.

Узлы, которые имеют потоки входных и выходных данных, называются « _винтами_». Они выполняют фильтрацию данных, выборку и агрегирование.

Точка подключения службы (SCP) поддерживает рекомендации, как минимум однократно и только после обработки данных.

В распределенном приложении обработки потоков ошибки могут происходить во время обработки данных. Такие ошибки включают в себя сбой сети, сбои компьютера или ошибку в коде. Обработка по крайней мере один раз гарантирует, что все данные обрабатываются по крайней мере один раз, автоматически воспроизводить одни и те же данные при возникновении ошибки.

Обработка по крайней мере один раз является простой и надежной и подходит для многих приложений. Но если приложению требуется точный подсчет, то обработка по крайней мере один раз недостаточна, поскольку одни и те же данные могут быть воспроизведены в топологии приложения. В этом случае, только после обработки гарантируется, что результат будет правильным даже в том случае, если данные воспроизводятся и обрабатываются несколько раз.

SCP позволяет разработчикам .NET создавать приложения для обработки данных в режиме реального времени при использовании виртуальная машина Java (ВИРТУАЛЬНОЙ машины Java) с более высокой выпуском. ВИРТУАЛЬНОЙ машины Java и .NET обмениваются данными через локальные сокеты TCP. Каждый spout/молния является парой процессов .NET/Java, где пользовательская логика выполняется в процессе .NET в качестве подключаемого модуля.

Чтобы создать приложение для обработки данных поверх SCP, выполните следующие действия.

1. Проектирование и реализация реконструирования для извлечения данных из очередей.
1. Разработайте и реализуйте винты, которые обрабатывают входные данные и сохраняют их во внешних хранилищах, таких как база данных.
1. Создайте топологию, а затем отправьте и запустите ее.

Топология определяет вершины и данные, перенаправляемые между ними. SCP принимает спецификацию топологии и развертывает ее в кластере с более кластерами, где каждая вершина выполняется на одном логическом узле. Планировщик задач для работы с графиком отвечает за отработку отказа и масштабирование.

В этой статье приводятся простые примеры создания приложений для обработки данных с помощью SCP.

## <a name="scp-plug-in-interface"></a>Подключаемый интерфейс SCP

Подключаемые модули SCP — это автономные приложения. Они могут запускаться в среде Visual Studio во время разработки и подключаться к конвейеру "область" после развертывания в рабочей среде.

Написание подключаемого модуля SCP аналогично написанию любого другого консольного приложения Windows. Платформа SCP.NET объявляет некоторые интерфейсы для spout/молнии. Код подключаемого модуля реализует эти интерфейсы. Основная цель этой схемы — позволить вам сосредоточиться на бизнес-логике и позволить платформе SCP.NET управлять другими объектами.

Код подключаемого модуля реализует один из следующих интерфейсов. Интерфейс зависит от того, является ли топология транзакционной или нетранзакционной и является ли компонент spout или молнией.

* **ISCPSpout**
* **ISCPBolt**
* **ISCPTxSpout**
* **ISCPBatchBolt**

### <a name="iscpplugin"></a>ISCPPlugin

**ISCPPlugin** — это общий интерфейс для многих подключаемых модулей. В настоящее время это фиктивный интерфейс.

```csharp
public interface ISCPPlugin
{
}
```

### <a name="iscpspout"></a>ISCPSpout

**ISCPSpout** — это интерфейс для нетранзакционного spout.

```csharp
public interface ISCPSpout : ISCPPlugin
{
    void NextTuple(Dictionary<string, Object> parms);
    void Ack(long seqId, Dictionary<string, Object> parms); 
    void Fail(long seqId, Dictionary<string, Object> parms);
}
```

При вызове **NextTuple** код C# может выдавать один или несколько кортежей. Если ничего не выдается, этот метод должен возвращать значение без каких-либо порождений.

Методы **NextTuple**, **ACK** и **Fail** вызываются в строгом цикле в одном потоке процесса C#. Если нет кортежей для выдачи, **NextTuple** спящий режим в течение короткого промежутка времени, например 10 миллисекунд. Этот спящий режим позволяет избежать расхода ресурсов ЦП.

Методы **ACK** и **Fail** вызываются только в том случае, если файл спецификации включает механизм подтверждения. Параметр *seqId* определяет кортеж, который является подтвержденным или неудачным. Если в нетранзакционной топологии включено подтверждение, в spout следует использовать следующую функцию **Emit** :

```csharp
public abstract void Emit(string streamId, List<object> values, long seqId);
```

Если нетранзакционная топология не поддерживает подтверждение, то **Подтверждение** и **сбой** могут быть оставлены пустыми функциями.

Входной параметр *пармс* в этих функциях указывает пустой словарь и зарезервирован для использования в будущем.

### <a name="iscpbolt"></a>ISCPBolt

**ISCPBolt** — это интерфейс для нетранзакционного молнии.

```csharp
public interface ISCPBolt : ISCPPlugin
{
void Execute(SCPTuple tuple);
}
```

Когда доступен новый кортеж, для его обработки вызывается функция **EXECUTE** .

### <a name="iscptxspout"></a>ISCPTxSpout

**ISCPTxSpout** — это интерфейс для транзакционной spout.

```csharp
public interface ISCPTxSpout : ISCPPlugin
{
    void NextTx(out long seqId, Dictionary<string, Object> parms);  
    void Ack(long seqId, Dictionary<string, Object> parms);         
    void Fail(long seqId, Dictionary<string, Object> parms);        
}
```

Как и в случае с нетранзакционными аналогами, **некстткс**, **ACK** и **Fail** вызываются в строгом цикле в одном потоке процесса C#. Если нет кортежей для выдачи, **некстткс** спящий режим в течение короткого промежутка времени, например 10 миллисекунд. Этот спящий режим позволяет избежать расхода ресурсов ЦП.

При вызове **некстткс** для запуска новой транзакции параметр OUTPUT *seqId* идентифицирует транзакцию. Транзакция также используется в **подтверждении** и **сбое**. Метод **некстткс** может выдавать данные на сторону Java. Данные сохраняются в ZooKeeper для возможности провести повторную отсылку. Так как ZooKeeper имеет ограниченную емкость, ваш код должен создавать только метаданные, а не данные в транзакционном spout.

Поскольку множество автоматически воспроизводит сбойную транзакцию, обычно не вызывается **сбой** . Но если SCP может проверить метаданные, порожденные транзакционным spout, можно вызвать **Fail** , если метаданные недопустимы.

Входной параметр *пармс* в этих функциях указывает пустой словарь и зарезервирован для использования в будущем.

### <a name="iscpbatchbolt"></a>ISCPBatchBolt

**ISCPBatchBolt** — это интерфейс для транзакционной.

```csharp
public interface ISCPBatchBolt : ISCPPlugin
{
    void Execute(SCPTuple tuple);
    void FinishBatch(Dictionary<string, Object> parms);  
}
```

Метод **EXECUTE** вызывается при поступлении нового кортежа в молнию. Метод **финишбатч** вызывается при завершении этой транзакции. Входной параметр *пармс* зарезервирован для использования в будущем.

Для транзакционной топологии **стормтксаттемпт** является важным классом. Он имеет два члена: **ИД транзакции** и **аттемптид**. Элемент **ИД транзакции** определяет конкретную транзакцию. Попытка выполнения транзакции может быть выполнена несколько раз, если она завершается сбоем и воспроизводится.

SCP.NET создает новый объект **ISCPBatchBolt** для обработки каждого объекта **стормтксаттемпт** , точно так же, как в Java. Эта цель разработки — поддержка параллельной обработки транзакций. По завершении попытки транзакции уничтожается соответствующий объект **ISCPBatchBolt** и выполняется сбор мусора.

## <a name="object-model"></a>Объектная модель

SCP.NET предоставляет разработчикам простой набор ключевых объектов для программирования. Объекты — **context**, **статесторе** и **SCPRuntime**. Они обсуждаются в этом разделе.

### <a name="context"></a>Контекст

Объект **контекста** предоставляет приложению работающую среду. Каждый экземпляр **ISCPPlugin** **ISCPSpout**, **ISCPBolt**, **ISCPTxSpout** или **ISCPBatchBolt** имеет соответствующий экземпляр **контекста** . Функциональные возможности, предоставляемые **контекстом** , делятся на следующие две части:

* Статическая часть, доступная во всем процессе C#
* Динамическая часть, доступная только для конкретного экземпляра **контекста**

### <a name="static-part"></a>Статическая часть

```csharp
public static ILogger Logger = null;
public static SCPPluginType pluginType;
public static Config Config { get; set; }
public static TopologyContext TopologyContext { get; set; }  
```

Объект **средства ведения** журнала предоставляется в целях ведения журнала.

Объект **плугинтипе** указывает тип подключаемого модуля процесса C#. Если процесс выполняется в локальном тестовом режиме без Java, тип подключаемого модуля — **SCP_NET_LOCAL**.

```csharp
public enum SCPPluginType 
{
    SCP_NET_LOCAL = 0,
    SCP_NET_SPOUT = 1,
    SCP_NET_BOLT = 2,
    SCP_NET_TX_SPOUT = 3,
    SCP_NET_BATCH_BOLT = 4  
    }
```

Свойство **config** получает параметры конфигурации со стороны Java, которая передает их при инициализации подключаемого модуля C#. Параметры **конфигурации** делятся на две части: **стормконф** и **получив pluginconf**.

```csharp
public Dictionary<string, Object> stormConf { get; set; }  
public Dictionary<string, Object> pluginConf { get; set; }  
```

Часть **стормконф** — это параметры, определенные с помощью параметра, а часть **получив pluginconf** — параметры, определенные SCP. Приведем пример:

```csharp
public class Constants
{
    … …

    // constant string for pluginConf
    public static readonly String NONTRANSACTIONAL_ENABLE_ACK = "nontransactional.ack.enabled";  

    // constant string for stormConf
    public static readonly String STORM_ZOOKEEPER_SERVERS = "storm.zookeeper.servers";
    public static readonly String STORM_ZOOKEEPER_PORT = "storm.zookeeper.port";
}
```

Тип **топологиконтекст** получает контекст топологии. Он наиболее удобен для нескольких параллельных компонентов. Приведем пример:

```csharp
//demo how to get TopologyContext info
if (Context.pluginType != SCPPluginType.SCP_NET_LOCAL)
{
    Context.Logger.Info("TopologyContext info:");
    TopologyContext topologyContext = Context.TopologyContext;
    Context.Logger.Info("taskId: {0}", topologyContext.GetThisTaskId());
    taskIndex = topologyContext.GetThisTaskIndex();
    Context.Logger.Info("taskIndex: {0}", taskIndex);
    string componentId = topologyContext.GetThisComponentId();
    Context.Logger.Info("componentId: {0}", componentId);
    List<int> componentTasks = topologyContext.GetComponentTasks(componentId);  
    Context.Logger.Info("taskNum: {0}", componentTasks.Count);
}
```

### <a name="dynamic-part"></a>Динамическая часть

Следующие интерфейсы относятся к определенному экземпляру **контекста** , который создается платформой SCP.NET и передается в код:

```csharp
// Declare the Output and Input Stream Schemas

public void DeclareComponentSchema(ComponentStreamSchema schema);

// Emit tuple to default stream.
public abstract void Emit(List<object> values);

// Emit tuple to the specific stream.
public abstract void Emit(string streamId, List<object> values);  
```

Для нетранзакционного spout, поддерживающего подтверждение, предоставляется следующий метод:

```csharp
// for nontransactional spout that supports ack
public abstract void Emit(string streamId, List<object> values, long seqId);  
```

Нетранзакционное сообщение, которое поддерживает подтверждение, должно явно вызывать **ACK** или **Failed** с полученным кортежем. При выдаче нового кортежа в молнию должны также указываться привязки кортежа. Предоставляются следующие методы.

```csharp
public abstract void Emit(string streamId, IEnumerable<SCPTuple> anchors, List<object> values);
public abstract void Ack(SCPTuple tuple);
public abstract void Fail(SCPTuple tuple);
```

### <a name="statestore"></a>StateStore

Объект **статесторе** предоставляет службы метаданных, создание монотонных последовательностей и координацию без ожидания. Вы можете создавать абстракции распределенного параллелизма более высокого уровня на **статесторе**. Эти абстракции включают распределенные блокировки, распределенные очереди, барьеры и службы транзакций.

Приложения SCP могут использовать объект **State** для сериализации данных в [Apache ZooKeeper](https://zookeeper.apache.org/). Эта возможность особенно полезна для транзакционной топологии. Если транзакционный spout перестает отвечать на запросы и перезапускается, **состояние** может получить необходимые сведения из ZooKeeper и перезапустить конвейер.

Объект **статесторе** имеет следующие основные методы:

```csharp
/// <summary>
/// Static method to retrieve a state store of the given path and connStr 
/// </summary>
/// <param name="storePath">StateStore path</param>
/// <param name="connStr">StateStore address</param>
/// <returns>Instance of StateStore</returns>
public static StateStore Get(string storePath, string connStr);

/// <summary>
/// Create a new state object in this state store instance
/// </summary>
/// <returns>State from StateStore</returns>
public State Create();

/// <summary>
/// Retrieve all states that were previously uncommitted, excluding all exited states
/// </summary>
/// <returns>Uncommitted states</returns>
public IEnumerable<State> GetUnCommitted();

/// <summary>
/// Get all the states in the StateStore
/// </summary>
/// <returns>All the states</returns>
public IEnumerable<State> States();

/// <summary>
/// Get state or registry object
/// </summary>
/// <param name="info">Registry name (registry only)</param>
/// <typeparam name="T">Type, registry or state</typeparam>
/// <returns>Return registry or state</returns>
public T Get<T>(string info = null);

/// <summary>
/// List all the committed states
/// </summary>
/// <returns>Registries containing the committed state </returns>
public IEnumerable<Registry> Committed();

/// <summary>
/// List all the exited states in the StateStore
/// </summary>
/// <returns>Registries containing the exited states</returns>
public IEnumerable<Registry> Aborted();

/// <summary>
/// Retrieve an existing state object from this state store instance 
/// </summary>
/// <returns>State from StateStore</returns>
/// <typeparam name="T">stateId, id of the State</typeparam>
public State GetState(long stateId)
```

Объект **состояния** имеет следующие основные методы:

```csharp
/// <summary>
/// Set the status of the state object to commit
/// </summary>
public void Commit(bool simpleMode = true);

/// <summary>
/// Set the status of the state object to exit
/// </summary>
public void Abort();

/// <summary>
/// Put an attribute value under the given key
/// </summary>
/// <param name="key">Key</param>
/// <param name="attribute">State attribute</param>
    public void PutAttribute<T>(string key, T attribute);

/// <summary>
/// Get the attribute value associated with the given key
/// </summary>
/// <param name="key">Key</param>
/// <returns>State attribute</returns>
    public T GetAttribute<T>(string key);
```

Если **симплемоде** имеет значение **true**, метод **commit** удаляет соответствующий znode будет удален в ZooKeeper. В противном случае метод удаляет текущий Znode будет удален и добавляет новый узел в зафиксированном \_ пути.

### <a name="scpruntime"></a>SCPRuntime

Класс **SCPRuntime** предоставляет следующие два метода:

```csharp
public static void Initialize();

public static void LaunchPlugin(newSCPPlugin createDelegate);  
```

Метод **Initialize** Инициализирует среду выполнения SCP. В этом методе процесс C# подключается к стороне Java для получения параметров конфигурации и контекста топологии.

Метод **лаунчплугин** запускает цикл обработки сообщений. В этом цикле подключаемый модуль C# получает сообщения от стороны Java. Эти сообщения включают кортежи и контрольные сигналы. Затем подключаемый модуль обрабатывает сообщения, возможно, путем вызова метода интерфейса, предоставленного вашим кодом.

Входной параметр для **лаунчплугин** является делегатом. Метод может возвращать объект, реализующий интерфейс **ISCPSpout**, **ISCPBolt**, **ISCPTxSpout** или **ISCPBatchBolt** .

```csharp
public delegate ISCPPlugin newSCPPlugin(Context ctx, Dictionary<string, Object> parms);
```

Для **ISCPBatchBolt** можно получить объект **стормтксаттемпт** из параметра *пармс* и использовать его, чтобы определить, является ли попытка воспроизведенной попыткой. Проверка попыток воспроизведения часто выполняется в процессе фиксации. Эта проверка показана в примере Хелловорлдткс ниже в этой статье.

Подключаемые модули SCP обычно могут работать в двух режимах: в локальном и обычном режиме.

#### <a name="local-test-mode"></a>Режим локального тестирования

В этом режиме подключаемые модули SCP в коде C# выполняются в среде Visual Studio на этапе разработки. В этом режиме можно использовать интерфейс **илокалконтекст** . Интерфейс предоставляет методы для сериализации созданных кортежей в локальные файлы и считывания их обратно в ОЗУ.

```csharp
public interface ILocalContext
{
    List<SCPTuple> RecvFromMsgQueue();
    void WriteMsgQueueToFile(string filepath, bool append = false);  
    void ReadFromFileToMsgQueue(string filepath);
}
```

#### <a name="regular-mode"></a>Обычный режим

В этом режиме процесс Java в процессе выполнения выполняет подключаемые модули SCP. Вот пример:

```csharp
namespace Scp.App.HelloWorld
{
public class Generator : ISCPSpout
{
    … …
    public static Generator Get(Context ctx, Dictionary<string, Object> parms)
    {
    return new Generator(ctx);
    }
}

class HelloWorld
{
    static void Main(string[] args)
    {
    /* Setting the environment variable here can change the log file name */
    System.Environment.SetEnvironmentVariable("microsoft.scp.logPrefix", "HelloWorld");

    SCPRuntime.Initialize();
    SCPRuntime.LaunchPlugin(new newSCPPlugin(Generator.Get));
    }
}
}
```

## <a name="topology-specification-language"></a>Язык спецификации топологии

Спецификация топологии SCP — это доменный язык (DSL) для описания и настройки топологий SCP. Он основан на [CLOJURE DSL](https://storm.incubator.apache.org/documentation/Clojure-DSL.html) и расширяется ТОЧКОЙ подключения службы.

Вы можете отправить спецификации топологии непосредственно в кластер с более точностью для выполнения с помощью команды **runSpec** .

SCP.NET добавил следующие функции для определения топологий транзакций:

| Новая функция | Параметры | Описание |
| --- | --- | --- |
| **tx-topolopy** |*topology-name*<br />*spout-map*<br />*bolt-map* |Определяет топологию транзакций с именем топологии, схемой определения и картой определения "винты". |
| **scp-tx-spout** |*exec-name*<br />*args*<br />*поля* |Определяет транзакционный spout. Функция запускает приложение, указанное *exec-Name* , и использует *аргументы*.<br /><br />Параметр *Fields* указывает выходные поля для spout. |
| **scp-tx-batch-bolt** |*exec-name*<br />*args*<br />*поля* |Определяет транзакционную пакетную публикацию. Функция запускает приложение, указанное *exec-Name* , и использует *аргументы.*<br /><br />Параметр *Fields* указывает поля вывода для молнии. |
| **scp-tx-commit-bolt** |*exec-name*<br />*args*<br />*поля* |Определяет транзакционную фиксацию. Функция запускает приложение, указанное *exec-Name* , и использует *аргументы*.<br /><br />Параметр *Fields* указывает поля вывода для молнии. |
| **нонткс — топология** |*topology-name*<br />*spout-map*<br />*bolt-map* |Определяет нетранзакционную топологию с именем топологии, схемой определения и картой определения "винты". |
| **scp-spout** |*exec-name*<br />*args*<br />*поля*<br />*parameters* |Определяет нетранзакционный spout. Функция запускает приложение, указанное *exec-Name* , и использует *аргументы*.<br /><br />Параметр *Fields* указывает выходные поля для spout.<br /><br />Параметр *Parameters* является необязательным. Используйте его для указания таких параметров, как "нетранзакционный. ACK. Enabled". |
| **scp-bolt** |*exec-name*<br />*args*<br />*поля*<br />*parameters* |Определяет нетранзакционный. Функция запускает приложение, указанное *exec-Name* , и использует *аргументы*.<br /><br />Параметр *Fields* указывает поля вывода для молнии<br /><br />Параметр *Parameters* является необязательным. Используйте его для указания таких параметров, как "нетранзакционный. ACK. Enabled". |

SCP.NET определяет следующие ключевые слова:

| Ключевое слово | Описание |
| --- | --- |
| **: имя** |Имя топологии |
| **: топология** |Топология, использующая функции из предыдущей таблицы и встроенные функции |
| **:p** |Указание параллелизма для каждого spout или молнии |
| **: Конфигурация** |Следует ли настраивать параметры или обновлять существующие |
| **: схема** |Схема потока |

SCP.NET также определяет следующие часто используемые параметры:

| Параметр | Описание |
| --- | --- |
| "plugin.name" |Имя EXE-файла подключаемого модуля C# |
| "plugin.args" |Аргументы подключаемого модуля |
| "output.schema" |Выходная схема |
| "nontransactional.ack.enabled" |Включено ли подтверждение для нетранзакционной топологии |

Команда **runSpec** развертывается вместе с битами. Ниже приведены сведения об использовании команды.

```csharp
.\bin\runSpec.cmd
usage: runSpec [spec-file target-dir [resource-dir] [-cp classpath]]
ex: runSpec examples\HelloWorld\HelloWorld.spec specs examples\HelloWorld\Target
```

Параметр *Resource-dir* является необязательным. Укажите его, если вы хотите подключить приложение C#. Указанный каталог содержит приложение, зависимости и конфигурации.

Параметр *classpath* также необязательный. Он указывает подкаталог классов Java, если файл спецификации содержит Java spout или молния.

## <a name="miscellaneous-features"></a>Прочие функции

### <a name="input-and-output-schema-declarations"></a>Объявления схемы ввода и вывода

Процессы C# могут создавать кортежи. Для этого платформа сериализует кортежи в объекты **Byte []** и передает объекты на сторону Java. А затем передает эти кортежи в целевые объекты.

В нижестоящих компонентах процессы C# получают кортежи обратно со стороны Java и преобразуют их в исходные типы платформы. Все эти операции скрыты платформой.

Для поддержки сериализации и десериализации код должен объявлять схему входных и выходных данных. Схема определена как словарь. Идентификатор потока — это ключ словаря. Значением ключа являются типы столбцов. Компонент может объявлять несколько потоков.

```csharp
public class ComponentStreamSchema
{
    public Dictionary<string, List<Type>> InputStreamSchema { get; set; }
    public Dictionary<string, List<Type>> OutputStreamSchema { get; set; }
    public ComponentStreamSchema(Dictionary<string, List<Type>> input, Dictionary<string, List<Type>> output)
    {
        InputStreamSchema = input;
        OutputStreamSchema = output;
    }
}
```

В объект **контекста** добавляется следующая функция:

```csharp
public void DeclareComponentSchema(ComponentStreamSchema schema)
```

Разработчики должны убедиться, что созданные кортежи подчиняются схеме, определенной для потока. В противном случае система выдаст исключение во время выполнения.

### <a name="multistream-support"></a>Поддержка многопотоковой передачи

SCP позволяет коду выдавать или принимать из нескольких различных потоков одновременно. Объект **контекста** отражает эту поддержку в качестве параметра идентификатора необязательного потока метода **Emit** .

Были добавлены два метода в объекте **контекста** SCP.NET. Они выдают один или несколько кортежей в определенные потоки. Параметр *streamId* является строкой. Его значение должно быть одинаковым в коде C# и в спецификации определения топологии.

```csharp
/* Emit tuple to the specific stream. */
public abstract void Emit(string streamId, List<object> values);

/* for nontransactional spout only */
public abstract void Emit(string streamId, List<object> values, long seqId);
```

Передача в несуществующий поток вызывает исключения среды выполнения.

### <a name="fields-grouping"></a>Группирование полей

Встроенные поля, сгруппированные по, не работают должным образом в SCP.NET. На стороне прокси-сервера Java тип данных всех полей фактически является **Byte []**. Группирование полей использует хэш-код объекта **Byte []** для группирования. Хэш-код — это адрес этого объекта в ОЗУ. Поэтому группирование будет неправильным для многобайтовых объектов, использующих одно и то же содержимое, но не тот же адрес.

SCP.NET добавляет настраиваемый метод группирования и использует содержимое объекта **Byte []** для выполнения группирования. В файле спецификации синтаксис выглядит как в следующем примере:

```csharp
(bolt-spec
    {
        "spout_test" (scp-field-group :non-tx [0,1])
    }
    …
)
```

В предыдущем файле спецификации:

* `scp-field-group` Указывает, что группирование является настраиваемой группировкой полей, реализованной SCP.
* `:tx` или `:non-tx` указывает, является ли топология транзакционной. Эти сведения необходимы, так как начальный индекс отличается между транзакционными и нетранзакционными топологиями.
* `[0,1]` Задает хэш-набор идентификаторов полей, начинающихся с нуля.

### <a name="hybrid-topology"></a>Гибридная топология

Код машинного кода написан на языке Java. SCP.NET имеет Улучшенное расширение, позволяющее писать код C# для обработки бизнес-логики. Но SCP.NET также поддерживает гибридные топологии, которые содержат не только выступающие и обвинты C#, но также и дополнительные выступающие и винты Java.

### <a name="specify-java-spoutbolt-in-a-specification-file"></a>Указание Java spout/молнии в файле спецификации

Вы можете использовать **SCP-spout** и **SCP-молнию** в файле спецификации для указания характеристик и характеристик Java. Приведем пример:

```csharp
(spout-spec 
  (microsoft.scp.example.HybridTopology.Generator.)
  :p 1)
```

Ниже приведено `microsoft.scp.example.HybridTopology.Generator` имя класса spout для Java.

### <a name="specify-the-java-classpath-in-a-runspec-command"></a>Указание подкаталогов Java в команде runSpec

Если вы хотите отправить топологию, содержащую сведения о Java или «винтах», сначала скомпилируйте их для создания JAR-файлов. Затем укажите подкаталог классов Java, который содержит JAR-файлы при отправке топологии. Приведем пример:

```csharp
bin\runSpec.cmd examples\HybridTopology\HybridTopology.spec specs examples\HybridTopology\net\Target -cp examples\HybridTopology\java\target\*
```

Здесь `examples\HybridTopology\java\target\` находится папка, содержащая файл JAR-файла Java spout/.

### <a name="serialization-and-deserialization-between-java-and-c"></a>Сериализация и десериализация между Java и C #

Компонент SCP включает в себя Java и сторону C#. Для взаимодействия с собственными выводами и обобщением Java на стороне Java и C# должны выполняться сериализация и десериализация, как показано на следующей диаграмме:

:::image type="content" source="./media/apache-storm-scp-programming-guide/java-compent-sending-to-scp-component-sending-to-java-component.png" alt-text="Схема компонента Java, отправляющего в компонент SCP, который затем отправляется в другой компонент Java" border="false":::

#### <a name="serialization-in-the-java-side-and-deserialization-in-the-c-side"></a>Сериализация на стороне Java и десериализация на стороне C#

Сначала предоставьте реализацию по умолчанию для сериализации на стороне Java и десериализации на стороне C#.

Укажите метод сериализации на стороне Java в файле спецификации.

```csharp
(scp-bolt
    {
        "plugin.name" "HybridTopology.exe"
        "plugin.args" ["displayer"]
        "output.schema" {}
        "customized.java.serializer" ["microsoft.scp.storm.multilang.CustomizedInteropJSONSerializer"]
    })
```

Укажите метод десериализации на стороне C# в коде C#.

```csharp
Dictionary<string, List<Type>> inputSchema = new Dictionary<string, List<Type>>();
inputSchema.Add("default", new List<Type>() { typeof(Person) });
this.ctx.DeclareComponentSchema(new ComponentStreamSchema(inputSchema, null));
this.ctx.DeclareCustomizedDeserializer(new CustomizedInteropJSONDeserializer());
```  

Если тип данных не слишком сложен, реализация по умолчанию должна работать в большинстве случаев. Ниже приведены случаи, в которых можно подключить собственную реализацию:

* Тип данных слишком сложен для реализации по умолчанию.
* Производительность вашей реализации по умолчанию не соответствует вашим требованиям.

Интерфейс сериализации на стороне Java определяется следующим образом:

```csharp
public interface ICustomizedInteropJavaSerializer {
    public void prepare(String[] args);
    public List<ByteBuffer> serialize(List<Object> objectList);
}
```

Интерфейс десериализации на стороне C# определяется следующим образом:

```csharp
public interface ICustomizedInteropCSharpDeserializer
{
    List<Object> Deserialize(List<byte[]> dataList, List<Type> targetTypes);
}
```

#### <a name="serialization-in-the-c-side-and-deserialization-in-the-java-side"></a>Сериализация на стороне C# и десериализация на стороне Java

Укажите метод сериализации на стороне C# в коде C#.

```csharp
this.ctx.DeclareCustomizedSerializer(new CustomizedInteropJSONSerializer()); 
```

Укажите метод десериализации на стороне Java в файле спецификации.

```csharp
(scp-spout
   {
     "plugin.name" "HybridTopology.exe"
     "plugin.args" ["generator"]
     "output.schema" {"default" ["person"]}
     "customized.java.deserializer" ["microsoft.scp.storm.multilang.CustomizedInteropJSONDeserializer" "microsoft.scp.example.HybridTopology.Person"]
   }
)
```

Здесь `"microsoft.scp.storm.multilang.CustomizedInteropJSONDeserializer"` — имя десериализатора, а `"microsoft.scp.example.HybridTopology.Person"` — целевой класс, в который десериализуется данные.

Можно также подключить собственную реализацию сериализатора C# и десериализатора Java.

Этот код является интерфейсом для сериализатора C#:

```csharp
public interface ICustomizedInteropCSharpSerializer
{
    List<byte[]> Serialize(List<object> dataList);
}
```

Этот код является интерфейсом для десериализатора Java:

```csharp
public interface ICustomizedInteropJavaDeserializer {
    public void prepare(String[] targetClassNames);
    public List<Object> Deserialize(List<ByteBuffer> dataList);
}
```

## <a name="scp-host-mode"></a>Режим узла SCP

В этом режиме можно скомпилировать код как библиотеку DLL и использовать SCPHost.exe, предоставляемые SCP для отправки топологии. Файл спецификации выглядит следующим образом:

```csharp
(scp-spout
  {
    "plugin.name" "SCPHost.exe"
    "plugin.args" ["HelloWorld.dll" "Scp.App.HelloWorld.Generator" "Get"]
    "output.schema" {"default" ["sentence"]}
  })
```

Здесь `"plugin.name"` указывается как `"SCPHost.exe"` , который предоставляется пакетом SDK для SCP. SCPHost.exe принимает три параметра в следующем порядке:

1. Имя библиотеки DLL `"HelloWorld.dll"` в этом примере.
1. Имя класса `"Scp.App.HelloWorld.Generator"` в этом примере.
1. Имя открытого статического метода, который может быть вызван для получения экземпляра **ISCPPlugin**.

В режиме узла Скомпилируйте код как библиотеку DLL для вызова платформой SCP. Так как платформа может получить полный контроль над всей логикой обработки, мы рекомендуем отправить топологию в режиме узла SCP. Это упрощает процесс разработки. Кроме того, она обеспечивает большую гибкость и лучшую обратную совместимость для последующих выпусков.

## <a name="scp-programming-examples"></a>Примеры программирования SCP

### <a name="helloworld"></a>HelloWorld

В следующем простом примере HelloWorld показано, как SCP.NET. Она использует нетранзакционную топологию с spout, именуемым **генератором** , и двумя винтами, именуемыми **Splitter** и **Counter**. **Генератор** spout случайным образом создает предложения и выдает эти предложения в **Разделитель**. **Разделительная** линия разделяет предложения на слова и выдает эти слова в виде **счетчика** . В этом **счетчике** для записи вхождений каждого слова используется словарь.

В этом примере есть два файла спецификации: HelloWorld. spec и HelloWorld \_ енаблеакк. spec. Код C# позволяет узнать, включено ли подтверждение, получая `pluginConf` объект со стороны Java.

```csharp
/* demo how to get pluginConf info */
if (Context.Config.pluginConf.ContainsKey(Constants.NONTRANSACTIONAL_ENABLE_ACK))
{
    enableAck = (bool)(Context.Config.pluginConf[Constants.NONTRANSACTIONAL_ENABLE_ACK]);
}
Context.Logger.Info("enableAck: {0}", enableAck);
```

Если в spout включено подтверждение, словарь кэширует кортежи, которые не были подтверждены. Если `Fail` вызывается метод, неисправный кортеж воспроизводится.

```csharp
public void Fail(long seqId, Dictionary<string, Object> parms)
{
    Context.Logger.Info("Fail, seqId: {0}", seqId);
    if (cachedTuples.ContainsKey(seqId))
    {
        /* get the cached tuple */
        string sentence = cachedTuples[seqId];

        /* replay the failed tuple */
        Context.Logger.Info("Re-Emit: {0}, seqId: {1}", sentence, seqId);
        this.ctx.Emit(Constants.DEFAULT_STREAM_ID, new Values(sentence), seqId);
    }
    else
    {
        Context.Logger.Warn("Fail(), can't find cached tuple for seqId {0}!", seqId);
    }
}
```

### <a name="helloworldtx"></a>HelloWorldTx

В следующем примере Хелловорлдткс показано, как реализовать транзакционную топологию. В этом примере имеется один spout, именуемый **генератором**, пакет, именуемый **частичным подсчетом**, и фиксацию с именем **Count-Sum**. Пример также содержит три существующих текстовых файла: DataSource0.txt, DataSource1.txt и DataSource2.txt.

В каждой транзакции **генератор** spout случайным образом выбирает два файла из существующих трех файлов и выдает два имени файлов в молнию **частичного числа** . Молния **частичного подсчета** :

1. Возвращает имя файла из полученного кортежа.
1. Открывает соответствующий файл.
1. Подсчитывает количество слов в файле.
1. Выдает число слов в молнию **Count-сумм** .

Объект bolt с именем **count-sum** вычисляет итоговое количество.

Чтобы добиться ровно такой семантики, при фиксации **количества** транзакций необходимо определить, является ли она воспроизводимой. В этом примере он имеет следующую статическую переменную члена:

```csharp
public static long lastCommittedTxId = -1; 
```

При создании экземпляра **ISCPBatchBolt** он получает значение `txAttempt` объекта из входных параметров.

```csharp
public static CountSum Get(Context ctx, Dictionary<string, Object> parms)
{
    /* for transactional topology, we can get txAttempt from the input parms */
    if (parms.ContainsKey(Constants.STORM_TX_ATTEMPT))
    {
        StormTxAttempt txAttempt = (StormTxAttempt)parms[Constants.STORM_TX_ATTEMPT];
        return new CountSum(ctx, txAttempt);
    }
    else
    {
        throw new Exception("null txAttempt");
    }
}
```

При `FinishBatch` вызове метод `lastCommittedTxId` обновляется, если он не является воспроизводимой транзакцией.

```csharp
public void FinishBatch(Dictionary<string, Object> parms)
{
    /* judge whether it is a replayed transaction */
    bool replay = (this.txAttempt.TxId <= lastCommittedTxId);

    if (!replay)
    {
        /* If it is not replayed, update the totalCount and lastCommittedTxId value */
        totalCount = totalCount + this.count;
        lastCommittedTxId = this.txAttempt.TxId;
    }
    … …
}
```

### <a name="hybridtopology"></a>Гибридная топология

Эта топология содержит spout Java и C#. Он использует реализацию сериализации и десериализации по умолчанию, предоставляемую платформой SCP. Сведения о файле спецификации см. в файле HybridTopology. Spec в папке examples \\ HybridTopology. См. также SubmitTopology.bat о том, как указать подкаталог классов Java.

### <a name="scphostdemo"></a>SCPHostDemo

Этот пример, по сути, аналогичен HelloWorld. Единственное отличие состоит в том, что код компилируется в виде библиотеки DLL, а топология отправляется с помощью SCPHost.exe. Более подробное описание см. в разделе режим узла SCP.

## <a name="next-steps"></a>Дальнейшие действия

Примеры топологий Apache Storm, созданных с помощью SCP, см. в следующих статьях:

* [Разработка топологий для Apache Storm в HDInsight на C# с помощью Visual Studio](apache-storm-develop-csharp-visual-studio-topology.md)
* [Обработка событий из Центров событий Azure с помощью Apache Storm в HDInsight](apache-storm-develop-csharp-event-hub-topology.md)
* [Обработка данных с датчиков автомобилей из Центров событий с помощью Apache Storm в HDInsight](https://github.com/hdinsight/hdinsight-storm-examples/tree/master/IotExample)
* [Извлечение, преобразование и загрузка (ETL) из концентраторов событий Azure в Apache HBase](https://github.com/hdinsight/hdinsight-storm-examples/blob/master/RealTimeETLExample)
