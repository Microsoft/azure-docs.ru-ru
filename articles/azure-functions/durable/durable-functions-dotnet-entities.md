---
title: Рекомендации разработчика по устойчивым сущностям в .NET — функции Azure
description: Как работать с устойчивыми сущностями в .NET с помощью расширения Устойчивые функции для функций Azure.
author: sebastianburckhardt
ms.topic: conceptual
ms.date: 10/06/2019
ms.author: azfuncdf
ms.openlocfilehash: 88d2a23104b67dae8fd480406eb9171e9f3d5652
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92740019"
---
# <a name="developers-guide-to-durable-entities-in-net"></a>Рекомендации разработчика по устойчивым сущностям в .NET

В этой статье мы подробно рассмотрим доступные интерфейсы для разработки устойчивых сущностей с помощью .NET, включая примеры и общие рекомендации. 

Функции сущностей предоставляют разработчикам бессерверных приложений удобный способ организации состояния приложения в виде коллекции детализированных сущностей. Дополнительные сведения об основных понятиях см. в статье о [устойчивых сущностях: основные понятия](durable-functions-entities.md) .

Сейчас мы предлагаем два API для определения сущностей:

- **Синтаксис на основе классов** представляет сущности и операции как классы и методы. Этот синтаксис обеспечивает простой для чтения код и позволяет вызывать операции с помощью интерфейсов с проверкой типа. 

- **Синтаксис на основе функций** является интерфейсом более низкого уровня, который представляет сущности как функции. Он обеспечивает точный контроль над тем, как отправляются операции сущности, и как осуществляется управление состоянием сущности.  

В этой статье основное внимание уделяется синтаксису на основе классов, так как предполагается, что он лучше подходит для большинства приложений. Однако [синтаксис на основе функций](#function-based-syntax) может быть приемлемым для приложений, которые хотят определять собственные абстракции для состояния и операций объекта и управлять ими. Кроме того, он может подойти для реализации библиотек, требующих универсальности, которые в настоящее время не поддерживаются синтаксисом на основе класса. 

> [!NOTE]
> Синтаксис на основе классов — это просто слой поверх синтаксиса на основе функций, поэтому оба варианта могут быть взаимозаменяемы в одном и том же приложении. 
 
## <a name="defining-entity-classes"></a>Определение классов сущностей

В следующем примере реализована `Counter` сущность, которая хранит одно значение типа Integer, и предлагает четыре операции `Add` : `Reset` ,, `Get` и `Delete` .

```csharp
[JsonObject(MemberSerialization.OptIn)]
public class Counter
{
    [JsonProperty("value")]
    public int Value { get; set; }

    public void Add(int amount) 
    {
        this.Value += amount;
    }

    public Task Reset() 
    {
        this.Value = 0;
        return Task.CompletedTask;
    }

    public Task<int> Get() 
    {
        return Task.FromResult(this.Value);
    }

    public void Delete() 
    {
        Entity.Current.DeleteState();
    }

    [FunctionName(nameof(Counter))]
    public static Task Run([EntityTrigger] IDurableEntityContext ctx)
        => ctx.DispatchAsync<Counter>();
}
```

`Run`Функция содержит шаблон, необходимый для использования синтаксиса на основе классов. Это должна быть *статическая* функция Azure. Он выполняется один раз для каждого сообщения операции, обрабатываемого сущностью. Когда `DispatchAsync<T>` вызывается и сущность еще не находится в памяти, она конструирует объект типа `T` и заполняет его поля из последнего сохраненного JSON-файла, найденного в хранилище (если таковой имеется). Затем он вызывает метод с соответствующим именем.

> [!NOTE]
> Состояние сущности на основе класса **создается неявно** перед тем, как сущность обрабатывает операцию, и может быть **удалена явным образом** в операции путем вызова `Entity.Current.DeleteState()` .

### <a name="class-requirements"></a>Требования к классам
 
Классы сущностей — это POCO (обычные старые объекты CLR), для которых не требуются специальные классы, интерфейсы или атрибуты. Но:

- Класс должен быть конструируемым (см. раздел [Конструирование сущностей](#entity-construction)).
- Класс должен быть сериализуемым в формат JSON (см. раздел [Сериализация сущностей](#entity-serialization)).

Кроме того, любой метод, предназначенный для вызова в качестве операции, должен удовлетворять дополнительным требованиям:

- Операция должна иметь не более одного аргумента и не должна иметь никаких перегрузок или аргументов универсального типа.
- Операция, предназначенная для вызова из оркестрации с помощью интерфейса, должна возвращать `Task` или `Task<T>` .
- Аргументы и возвращаемые значения должны быть сериализуемыми значениями или объектами.

### <a name="what-can-operations-do"></a>Что могут делать операции?

Все операции с сущностью могут считывать и обновлять состояние сущности, а изменения состояния автоматически сохраняются в хранилище. Кроме того, операции могут выполнять внешние операции ввода-вывода или другие вычисления в рамках общих ограничений, общих для всех функций Azure.

Операции также имеют доступ к функциональным возможностям, предоставляемым `Entity.Current` контекстом:

* `EntityName`: имя выполняемой в данный момент сущности.
* `EntityKey`— ключ выполняемой в данный момент сущности.
* `EntityId`: идентификатор выполняющейся в данный момент сущности (включает имя и ключ).
* `SignalEntity`: отправляет одностороннее сообщение в сущность.
* `CreateNewOrchestration`: запускает новое согласование.
* `DeleteState`: удаляет состояние этой сущности.

Например, можно изменить сущность счетчика, чтобы она начала оркестрации, когда счетчик достигнет 100 и передаст идентификатор сущности в качестве входного аргумента:

```csharp
    public void Add(int amount) 
    {
        if (this.Value < 100 && this.Value + amount >= 100)
        {
            Entity.Current.StartNewOrchestration("MilestoneReached", Entity.Current.EntityId);
        }
        this.Value += amount;      
    }
```

## <a name="accessing-entities-directly"></a>Прямой доступ к сущностям

К сущностям на основе классов можно обращаться напрямую, используя явные имена строк для сущности и ее операций. Ниже приведены некоторые примеры. более подробное описание базовых понятий (таких как сигналы и вызовы) см. в обсуждении [сущностей Access](durable-functions-entities.md#access-entities). 

> [!NOTE]
> Там, где это возможно, мы советуем [обращаться к сущностям через интерфейсы](#accessing-entities-through-interfaces), так как они обеспечивают дополнительную проверку типов.

### <a name="example-client-signals-entity"></a>Пример: клиент сигнализирует об объекте

Следующая функция HTTP Azure реализует операцию удаления с использованием соглашений RESTFUL. Он отправляет сигнал об удалении в сущность счетчика, ключ которой передается в URL-пути.

```csharp
[FunctionName("DeleteCounter")]
public static async Task<HttpResponseMessage> DeleteCounter(
    [HttpTrigger(AuthorizationLevel.Function, "delete", Route = "Counter/{entityKey}")] HttpRequestMessage req,
    [DurableClient] IDurableEntityClient client,
    string entityKey)
{
    var entityId = new EntityId("Counter", entityKey);
    await client.SignalEntityAsync(entityId, "Delete");    
    return req.CreateResponse(HttpStatusCode.Accepted);
}
```

### <a name="example-client-reads-entity-state"></a>Пример: клиент считывает состояние сущности

Следующая функция HTTP Azure реализует операцию GET с использованием соглашений RESTFUL. Он считывает текущее состояние сущности счетчика, ключ которой передается в URL-пути.

```csharp
[FunctionName("GetCounter")]
public static async Task<HttpResponseMessage> GetCounter(
    [HttpTrigger(AuthorizationLevel.Function, "get", Route = "Counter/{entityKey}")] HttpRequestMessage req,
    [DurableClient] IDurableEntityClient client,
    string entityKey)
{
    var entityId = new EntityId("Counter", entityKey);
    var state = await client.ReadEntityStateAsync<Counter>(entityId); 
    return req.CreateResponse(state);
}
```

> [!NOTE]
> Объект, возвращаемый функцией, `ReadEntityStateAsync` — это только локальная копия, то есть моментальный снимок состояния сущности с более ранней точки во времени. В частности, это может быть устаревшим, и изменение этого объекта не влияет на фактическую сущность. 

### <a name="example-orchestration-first-signals-then-calls-entity"></a>Пример: сначала orchestration передает, а затем вызывает сущность

Следующее согласование сообщает сущности счетчика, чтобы увеличить ее, а затем вызывает ту же сущность для считывания ее последнего значения.

```csharp
[FunctionName("IncrementThenGet")]
public static async Task<int> Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var entityId = new EntityId("Counter", "myCounter");

    // One-way signal to the entity - does not await a response
    context.SignalEntity(entityId, "Add", 1);

    // Two-way call to the entity which returns a value - awaits the response
    int currentValue = await context.CallEntityAsync<int>(entityId, "Get");

    return currentValue;
}
```

## <a name="accessing-entities-through-interfaces"></a>Доступ к сущностям через интерфейсы

Интерфейсы можно использовать для доступа к сущностям через созданные прокси-объекты. Такой подход гарантирует, что имя и тип аргумента операции совпадают с реализуемыми. Мы рекомендуем использовать интерфейсы для доступа к сущностям, когда это возможно.

Например, можно изменить пример счетчика следующим образом:

```csharp
public interface ICounter
{
    void Add(int amount);
    Task Reset();
    Task<int> Get();
    void Delete();
}

public class Counter : ICounter
{
    ...
}
```

Классы сущностей и интерфейсы сущностей похожи на интерфейсы детализации и детализации, которые популярны [Orleans](https://www.microsoft.com/research/project/orleans-virtual-actors/). Дополнительные сведения о сходстве и различиях между устойчивыми сущностями и Orleans см. в статье [Сравнение с виртуальными субъектами](durable-functions-entities.md#comparison-with-virtual-actors).

Помимо обеспечения проверки типов, интерфейсы полезны для более точного разделения проблем в приложении. Например, поскольку сущность может реализовывать несколько интерфейсов, одна сущность может обслуживать несколько ролей. Кроме того, поскольку интерфейс может быть реализован несколькими сущностями, общие шаблоны связи могут быть реализованы в виде многократно используемых библиотек.

### <a name="example-client-signals-entity-through-interface"></a>Пример. Клиент оповещает сущность через интерфейс

Клиентский код может использовать `SignalEntityAsync<TEntityInterface>` для отправки сигналов в сущности, реализующие `TEntityInterface` . Пример:

```csharp
[FunctionName("DeleteCounter")]
public static async Task<HttpResponseMessage> DeleteCounter(
    [HttpTrigger(AuthorizationLevel.Function, "delete", Route = "Counter/{entityKey}")] HttpRequestMessage req,
    [DurableClient] IDurableEntityClient client,
    string entityKey)
{
    var entityId = new EntityId("Counter", entityKey);
    await client.SignalEntityAsync<ICounter>(entityId, proxy => proxy.Delete());    
    return req.CreateResponse(HttpStatusCode.Accepted);
}
```

В этом примере `proxy` параметр является динамически создаваемым экземпляром `ICounter` , который внутренне преобразует вызов в `Delete` сигнал.

> [!NOTE]
> `SignalEntityAsync`API-интерфейсы можно использовать только для односторонних операций. Даже если операция возвращает `Task<T>` , значение `T` параметра всегда будет равно null или `default` , а не фактический результат.
Например, не имеет смысла подать сигнал `Get` операции, так как значение не возвращается. Вместо этого клиенты могут использовать либо `ReadStateAsync` для прямого доступа к состоянию счетчика, либо для запуска функции Orchestrator, которая вызывает `Get` операцию. 

### <a name="example-orchestration-first-signals-then-calls-entity-through-proxy"></a>Пример. процесс оркестрации сначала сигнализирует, затем вызывает сущность через прокси-сервер

Чтобы вызвать или передать объект в рамках оркестрации, `CreateEntityProxy` можно использовать вместе с типом интерфейса, чтобы создать прокси для сущности. Затем этот прокси-сервер можно использовать для вызова или сигнализации операций:

```csharp
[FunctionName("IncrementThenGet")]
public static async Task<int> Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var entityId = new EntityId("Counter", "myCounter");
    var proxy = context.CreateEntityProxy<ICounter>(entityId);

    // One-way signal to the entity - does not await a response
    proxy.Add(1);

    // Two-way call to the entity which returns a value - awaits the response
    int currentValue = await proxy.Get();

    return currentValue;
}
```

Неявным образом возвращаются все возвращаемые операции `void` и любые операции, которые возвращают `Task` или `Task<T>` вызывают. Одно из них может изменить это поведение по умолчанию и операции с сигналами, даже если они возвращают задачу, используя `SignalEntity<IInterfaceType>` метод явным образом.

### <a name="shorter-option-for-specifying-the-target"></a>Более короткий параметр для указания целевого объекта

При вызове или сигнализации сущности с помощью интерфейса в первом аргументе должна быть указана целевая сущность. Целевой объект можно указать либо путем указания идентификатора сущности, либо, в тех случаях, когда существует только один класс, реализующий сущность, а только ключ сущности:

```csharp
context.SignalEntity<ICounter>(new EntityId(nameof(Counter), "myCounter"), ...);
context.SignalEntity<ICounter>("myCounter", ...);
```

Если указан только ключ сущности и не удается найти уникальную реализацию во время выполнения, `InvalidOperationException` возникает исключение. 

### <a name="restrictions-on-entity-interfaces"></a>Ограничения для интерфейсов сущностей

Как правило, все типы параметров и возвращаемых типов должны быть сериализуемыми в формат JSON. В противном случае исключения сериализации создаются во время выполнения.

Также применяются некоторые дополнительные правила:
* Интерфейсы сущности должны определять только методы.
* Интерфейсы сущностей не должны содержать универсальные параметры.
* Методы интерфейса сущности не должны иметь более одного параметра.
* Методы интерфейса сущности должны возвращать `void` , `Task` или `Task<T>` 

Если какое-либо из этих правил нарушается, `InvalidOperationException` во время выполнения создается исключение, когда интерфейс используется в качестве аргумента типа для `SignalEntity` или `CreateProxy` . В сообщении об исключении объясняется, какое правило было разорвано.

> [!NOTE]
> Возвращаемые методы интерфейса `void` могут быть только сигнальными (односторонние), но не вызванными (двусторонними). Методы интерфейса, возвращающие `Task` или, `Task<T>` могут быть либо вызванными, либо сигнальными. При вызове они возвращают результат операции или повторно создают исключения, созданные операцией. Однако при появлению сигнала они не возвращают фактический результат или исключение из операции, а только значение по умолчанию.

## <a name="entity-serialization"></a>Сериализация сущностей

Так как состояние сущности надежно материализованным, класс сущности должен быть сериализуемым. Среда выполнения Устойчивые функции использует библиотеку [JSON.NET](https://www.newtonsoft.com/json) для этой цели, которая поддерживает ряд политик и атрибутов для управления процессом сериализации и десериализации. Наиболее часто используемые типы данных C# (включая массивы и типы коллекций) уже являются сериализуемыми и могут быть легко использованы для определения состояния устойчивых сущностей.

Например, Json.NET может легко сериализовать и десериализовать следующий класс:

```csharp
[JsonObject(MemberSerialization = MemberSerialization.OptIn)]
public class User
{
    [JsonProperty("name")]
    public string Name { get; set; }

    [JsonProperty("yearOfBirth")]
    public int YearOfBirth { get; set; }

    [JsonProperty("timestamp")]
    public DateTime Timestamp { get; set; }

    [JsonProperty("contacts")]
    public Dictionary<Guid, Contact> Contacts { get; set; } = new Dictionary<Guid, Contact>();

    [JsonObject(MemberSerialization = MemberSerialization.OptOut)]
    public struct Contact
    {
        public string Name;
        public string Number;
    }

    ...
}
```

### <a name="serialization-attributes"></a>Атрибуты сериализации

В приведенном выше примере мы решили включить несколько атрибутов, чтобы сделать базовую сериализацию более видимой:
- Мы добавляем аннотацию к классу, `[JsonObject(MemberSerialization.OptIn)]` чтобы напомнить нам, что класс должен быть сериализуемым, а также для сохранения только тех элементов, которые явно помечены как свойства JSON.
-  Мы замечаем поля, которые будут сохраняться в, `[JsonProperty("name")]` чтобы напомнить нам, что поле является частью состояния сохраненной сущности, и указать имя свойства, которое будет использоваться в представлении JSON.

Однако эти атрибуты не требуются. другие соглашения или атрибуты могут быть разрешены при условии, что они работают с Json.NET. Например, один из них может использовать `[DataContract]` атрибуты или вообще не иметь атрибутов:

```csharp
[DataContract]
public class Counter
{
    [DataMember]
    public int Value { get; set; }
    ...
}

public class Counter
{
    public int Value;
    ...
}
```

По умолчанию имя класса *не* сохраняется как часть представления JSON: то есть используется `TypeNameHandling.None` в качестве значения по умолчанию. Это поведение по умолчанию можно переопределить с `JsonObject` помощью `JsonProperty` атрибутов или.

### <a name="making-changes-to-class-definitions"></a>Внесение изменений в определения классов

При внесении изменений в определение класса после запуска приложения необходимо учитывать некоторые меры, поскольку сохраненный объект JSON может больше не соответствовать новому определению класса. Тем не менее часто можно правильно работать с измененными форматами данных, если один из них понимает процесс десериализации, используемый `JsonConvert.PopulateObject` .

Например, ниже приведены некоторые примеры изменений и их воздействие.

1. Если добавляется новое свойство, которое отсутствует в сохраненном JSON-формате, предполагается, что оно имеет значение по умолчанию.
1. Если свойство удалено, которое имеется в сохраненном JSON-формате, то предыдущее содержимое будет утеряно.
1. Если свойство переименовано, то результат будет таким же, как при удалении старого элемента и добавлении нового.
1. Если тип свойства изменяется, поэтому его невозможно десериализовать из хранимого JSON, создается исключение.
1. Если тип свойства изменен, но его все равно можно десериализовать из сохраненного JSON, это будет сделано.

Существует множество параметров, доступных для настройки поведения Json.NET. Например, чтобы принудительно вызвать исключение, если хранимый JSON содержит поле, которое отсутствует в классе, укажите атрибут `JsonObject(MissingMemberHandling = MissingMemberHandling.Error)` . Также можно написать пользовательский код для десериализации, который может читать JSON, хранящийся в произвольных форматах.

## <a name="entity-construction"></a>Создание сущностей

Иногда требуется больший контроль над тем, как создаются объекты сущностей. Теперь мы рассмотрим несколько вариантов изменения поведения по умолчанию при построении объектов сущностей. 

### <a name="custom-initialization-on-first-access"></a>Пользовательская инициализация при первом доступе

Иногда требуется выполнить некоторую специальную инициализацию перед отправкой операции в сущность, к которой никогда не обращались или которые были удалены. Чтобы указать это поведение, можно добавить условие перед `DispatchAsync` :

```csharp
[FunctionName(nameof(Counter))]
public static Task Run([EntityTrigger] IDurableEntityContext ctx)
{
    if (!ctx.HasState)
    {
        ctx.SetState(...);
    }
    return ctx.DispatchAsync<Counter>();
}
```

### <a name="bindings-in-entity-classes"></a>Привязки в классах сущностей

В отличие от обычных функций, методы классов сущностей не имеют прямого доступа к входным и выходным привязкам. Вместо этого данные привязки должны быть записаны в объявлении функции точки входа, а затем переданы в метод `DispatchAsync<T>`. Любые объекты, передаваемые в `DispatchAsync<T>`, будут автоматически переданы в конструктор класса сущностей в виде аргумента.

В следующем примере показано, как можно сделать ссылку `CloudBlobContainer` из [входной привязки BLOB-объекта](../functions-bindings-storage-blob-input.md) доступной для сущности на основе класса.

```csharp
public class BlobBackedEntity
{
    [JsonIgnore]
    private readonly CloudBlobContainer container;

    public BlobBackedEntity(CloudBlobContainer container)
    {
        this.container = container;
    }

    // ... entity methods can use this.container in their implementations ...
    
    [FunctionName(nameof(BlobBackedEntity))]
    public static Task Run(
        [EntityTrigger] IDurableEntityContext context,
        [Blob("my-container", FileAccess.Read)] CloudBlobContainer container)
    {
        // passing the binding object as a parameter makes it available to the
        // entity class constructor
        return context.DispatchAsync<BlobBackedEntity>(container);
    }
}
```

Дополнительные сведения о привязках в Функциях Azure см. в статье[Azure Functions triggers and bindings concepts](../functions-triggers-bindings.md) (Основные понятия триггеров и привязок в Функциях Azure).

### <a name="dependency-injection-in-entity-classes"></a>Внедрение зависимостей в классы сущностей

Классы сущностей [поддерживают внедрение зависимостей Функций Azure](../functions-dotnet-dependency-injection.md). В следующем примере показано, как зарегистрировать службу `IHttpClientFactory` в сущность на основе класса.

```csharp
[assembly: FunctionsStartup(typeof(MyNamespace.Startup))]

namespace MyNamespace
{
    public class Startup : FunctionsStartup
    {
        public override void Configure(IFunctionsHostBuilder builder)
        {
            builder.Services.AddHttpClient();
        }
    }
}
```

В следующем фрагменте кода показано, как встроить внедренную службу в класс сущностей.

```csharp
public class HttpEntity
{
    [JsonIgnore]
    private readonly HttpClient client;

    public HttpEntity(IHttpClientFactory factory)
    {
        this.client = factory.CreateClient();
    }

    public Task<int> GetAsync(string url)
    {
        using (var response = await this.client.GetAsync(url))
        {
            return (int)response.StatusCode;
        }
    }

    [FunctionName(nameof(HttpEntity))]
    public static Task Run([EntityTrigger] IDurableEntityContext ctx)
        => ctx.DispatchAsync<HttpEntity>();
}
```

> [!NOTE]
> Чтобы избежать проблем с сериализацией, не забудьте исключить поля, предназначенные для хранения внедренных значений из сериализации.

> [!NOTE]
> В отличие от внедрения конструктора в обычных Функциях Azure .NET, метод точки входа функций для сущностей на основе класса *необходимо* объявить `static`. Объявление нестатической точки входа функции может привести к конфликтам между обычным инициализатором объектов Функций Azure и инициализатором объектов Устойчивых сущностей.

## <a name="function-based-syntax"></a>Синтаксис на основе функций

До сих пор мы работаем над синтаксисом на основе классов, так как мы предполагаем, что он лучше подходит для большинства приложений. Однако синтаксис на основе функций может быть приемлемым для приложений, которые хотят определять собственные абстракции для состояния и операций объекта и управлять ими. Кроме того, это может быть уместно при реализации библиотек, требующих универсальности, которые в настоящее время не поддерживаются синтаксисом на основе класса. 

При использовании синтаксиса на основе функций функция сущности явным образом обрабатывает диспетчеризацию операции и явно управляет состоянием сущности. Например, в следующем коде показана сущность *счетчика* , реализованная с помощью синтаксиса на основе функций.  

```csharp
[FunctionName("Counter")]
public static void Counter([EntityTrigger] IDurableEntityContext ctx)
{
    switch (ctx.OperationName.ToLowerInvariant())
    {
        case "add":
            ctx.SetState(ctx.GetState<int>() + ctx.GetInput<int>());
            break;
        case "reset":
            ctx.SetState(0);
            break;
        case "get":
            ctx.Return(ctx.GetState<int>());
            break;
        case "delete":
            ctx.DeleteState();
            break;
    }
}
```

### <a name="the-entity-context-object"></a>Объект контекста сущности

Доступ к функциональным возможностям, относящимся к сущностям, можно получить через объект контекста типа `IDurableEntityContext` . Этот объект контекста доступен в качестве параметра функции сущности и через свойство Async-Local `Entity.Current` .

Следующие члены предоставляют сведения о текущей операции и позволяют указать возвращаемое значение. 

* `EntityName`: имя выполняемой в данный момент сущности.
* `EntityKey`— ключ выполняемой в данный момент сущности.
* `EntityId`: идентификатор выполняющейся в данный момент сущности (включает имя и ключ).
* `OperationName`: имя текущей операции.
* `GetInput<TInput>()`: получает входные данные для текущей операции.
* `Return(arg)`: Возвращает значение для оркестрации, вызвавшего операцию.

Следующие элементы управляют состоянием сущности (создание, чтение, обновление, удаление). 

* `HasState`: существует ли сущность, то есть имеет какое-то состояние. 
* `GetState<TState>()`: Возвращает текущее состояние сущности. Если он еще не существует, он будет создан.
* `SetState(arg)`: создает или обновляет состояние сущности.
* `DeleteState()`: удаляет состояние сущности, если она существует. 

Если состояние, возвращенное, `GetState` является объектом, его можно напрямую изменить с помощью кода приложения. Нет необходимости повторно вызываться `SetState` в конце (но также без ущерба). Если `GetState<TState>` вызывается несколько раз, необходимо использовать один и тот же тип.

Наконец, следующие члены используются для сигнализации других сущностей или запуска новых оркестрации:

* `SignalEntity(EntityId, operation, input)`: отправляет одностороннее сообщение в сущность.
* `CreateNewOrchestration(orchestratorFunctionName, input)`: запускает новое согласование.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Сведения о концепциях сущностей](durable-functions-entities.md)
