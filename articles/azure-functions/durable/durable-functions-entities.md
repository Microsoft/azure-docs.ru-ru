---
title: Устойчивые сущности — Функции Azure
description: Узнайте, что такое устойчивые сущности и как их использовать в расширении "Устойчивые функции" для Функций Azure.
author: cgillum
ms.topic: overview
ms.date: 12/17/2019
ms.author: azfuncdf
ms.openlocfilehash: c898444659c2ce071163e9ab774a4534f8c51a9c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102632056"
---
# <a name="entity-functions"></a>Функции сущностей

Функции сущностей определяют операции чтения и обновления мелких частей состояния, известных как *устойчивые сущности*. Как и функции оркестратора, функции сущностей — это функции с особым типом триггера, *триггером сущности*. В отличие от функций оркестрации, функции сущностей управляют состоянием сущности явным образом, а не отображают состояние с помощью потока управления.
Сущности предоставляют средства для масштабирования приложений путем распределения задач между несколькими сущностями сравнительно небольших размеров.

> [!NOTE]
> Функции сущностей и связанные функции доступны только в расширении [Устойчивые функции 2.0](durable-functions-versions.md#migrate-from-1x-to-2x) и более поздних версий. В настоящее время они поддерживаются в .NET, JavaScript и Python.

## <a name="general-concepts"></a>Общие концепции

Сущности похожи на небольшие службы, которые взаимодействуют через сообщения. Каждая сущность имеет уникальный идентификатор и внутреннее состояние (если оно существует). Подобно службам или объектам, сущности выполняют операции в ответ на соответствующие запросы. Во время выполнения операция может изменять внутреннее состояние сущности. Кроме того, она может вызывать внешние службы и ожидать ответа. Сущности взаимодействуют с другими сущностями, средствами оркестрации и клиентами, используя сообщения, которые неявно отправляются через надежные очереди. 

Во избежание конфликтов все операции с одной сущностью гарантированно выполняются последовательно, то есть одна за другой.

> [!NOTE]
> При вызове сущность полностью обрабатывает свои полезные данные, а затем планирует новое выполнение для активации при поступлении входных данных в будущем. В результате журналы выполнения сущности могут содержать сведения о дополнительном выполнении после каждого вызова сущности. Так и должно быть.

### <a name="entity-id"></a>Идентификатор сущности
Доступ к сущностям осуществляется с помощью уникального *идентификатора сущности*. Идентификатор сущности — это просто пара строк, с помощью которых уникально идентифицируется экземпляр сущности. В ее состав входит:

* **Имя сущности** — это имя, идентифицирующее тип сущности, например, "счетчик". Это имя должно совпадать с именем функции сущности, которая реализует эту сущность. Оно не чувствительно к регистру.
* **Ключ сущности** — это строка, однозначно идентифицирующая сущность среди всех других сущностей с тем же именем, например GUID.

Например, функция сущности `Counter` может использоваться для отслеживания оценки в интернет-игре. Каждый экземпляр игры имеет уникальный идентификатор сущности, например `@Counter@Game1` и `@Counter@Game2`. Для всех операций, предназначенных для определенной сущности, необходимо указать идентификатор в качестве параметра.

### <a name="entity-operations"></a>Операции с сущностями ###

Чтобы вызвать операцию с сущностью, нужно указать следующее:

* **Идентификатор целевой сущности**.
* **Имя выполняемой операции** в строковом формате. Например, сущность `Counter` может поддерживать операции `add`, `get` или `reset`.
* **Входные данные для операции** (необязательно), которые передаются в вызываемую операцию. Например, операция add может принимать в качестве входных данных целое число.
* **Запланированное время** — необязательный параметр, позволяющий указать время доставки для операции. Например, для операции можно надежно запланировать выполнение на протяжении нескольких дней в будущем.

Операции могут возвращать результирующее значение или ошибку (например, ошибку JavaScript или исключение .NET). Этот результат или ошибку можно наблюдать из средств оркестрации, которые вызвали эту операцию.

Операция с сущностью также может создавать, читать, обновлять и удалять состояние сущности. Состояние сущности всегда надежно сохраняется в хранилище.

## <a name="define-entities"></a>Определение сущностей

Сейчас доступны два разных интерфейса API для определения сущностей:

**Синтаксис на основе функций**, в котором сущности представлены в виде функций, а операции явным образом объявляются приложением. Этот синтаксис подходит для сущностей с простыми состояниями, небольшим количеством операций или динамическим набором операций (как, например, в исполняющей среде). Но настройка такого синтаксиса может быть утомительной, так как в нем невозможен перехват ошибок типа во время компиляции.

**Синтаксис на основе классов (только для .NET)** , в котором сущности и операции представлены классами и методами, соответственно. Этот синтаксис позволяет получить более удобочитаемый код и вызывать операции типобезопасным способом. Синтаксис на основе классов представляет собой тонкий слой, реализованный на базе синтаксиса на основе функций, и вы можете применять оба варианта одновременно в одном приложении.

# <a name="c"></a>[C#](#tab/csharp)

### <a name="example-function-based-syntax---c"></a>Пример Синтаксис на основе функций: C#

Приведенный ниже код является примером простой сущности `Counter`, реализованной в формате устойчивой функции. Эта функция определяет три операции: `add`, `reset` и `get`, каждая из которых изменяет целочисленное значение состояния.

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
    }
}
```

Дополнительные сведения о синтаксисе на основе функций и его использовании см. в [этой статье](durable-functions-dotnet-entities.md#function-based-syntax).

### <a name="example-class-based-syntax---c"></a>Пример Синтаксис на основе классов: C#

Следующий пример представляет собой эквивалентную реализацию сущности `Counter` с помощью классов и методов.

```csharp
[JsonObject(MemberSerialization.OptIn)]
public class Counter
{
    [JsonProperty("value")]
    public int CurrentValue { get; set; }

    public void Add(int amount) => this.CurrentValue += amount;

    public void Reset() => this.CurrentValue = 0;

    public int Get() => this.CurrentValue;

    [FunctionName(nameof(Counter))]
    public static Task Run([EntityTrigger] IDurableEntityContext ctx)
        => ctx.DispatchAsync<Counter>();
}
```

Состояние этой сущности представлено объектом типа `Counter`, который содержит поле с текущим значением счетчика. Чтобы сохранить этот объект в хранилище, он сериализуется и десериализуется библиотекой [Json.NET](https://www.newtonsoft.com/json). 

Дополнительные сведения о синтаксисе на основе классов и его использовании см. в [этой статье](durable-functions-dotnet-entities.md#defining-entity-classes).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

### <a name="example-javascript-entity"></a>Пример Сущность JavaScript

Устойчивые сущности доступны в JavaScript начиная с версии **1.3.0** пакета npm `durable-functions`. Приведенный ниже код является примером сущности `Counter`, реализованной в виде устойчивой функции на JavaScript.

**Counter/function.json**
```json
{
  "bindings": [
    {
      "name": "context",
      "type": "entityTrigger",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

**Counter/index.js**
```javascript
const df = require("durable-functions");

module.exports = df.entity(function(context) {
    const currentValue = context.df.getState(() => 0);
    switch (context.df.operationName) {
        case "add":
            const amount = context.df.getInput();
            context.df.setState(currentValue + amount);
            break;
        case "reset":
            context.df.setState(0);
            break;
        case "get":
            context.df.return(currentValue);
            break;
    }
});
```
# <a name="python"></a>[Python](#tab/python)

### <a name="example-python-entity"></a>Пример: сущность Python

Ниже приведен код примера сущности `Counter`, реализованной в виде устойчивой функции на Python.

**Counter/function.json**
```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "context",
      "type": "entityTrigger",
      "direction": "in"
    }
  ]
}
```

**Counter/__init__.py**
```Python
import azure.functions as func
import azure.durable_functions as df


def entity_function(context: df.DurableEntityContext):
    current_value = context.get_state(lambda: 0)
    operation = context.operation_name
    if operation == "add":
        amount = context.get_input()
        current_value += amount
    elif operation == "reset":
        current_value = 0
    elif operation == "get":
        context.set_result(current_value)
    context.set_state(current_value)


main = df.Entity.create(entity_function)
```
---

## <a name="access-entities"></a>Доступ к сущностям

К сущностям можно обращаться с использованием односторонней или двусторонней связи. Чтобы различать их, используются следующие термины: 

* **Вызов** сущности — использование двустороннего взаимодействия (туда и обратно). Вы отправляете сообщение с операцией сущности, а затем ожидаете ответного сообщения перед тем, как продолжить работу. Ответное сообщение может содержать результирующее значение или ошибку (например, ошибку JavaScript или исключение .NET). Этот результат или ошибка наблюдается вызывающей стороной.
* **Сигнал** для сущности — использование одностороннего взаимодействия (отправил и забыл). Вы отправляете сообщение с операцией и не ожидаете ответа. Доставка сообщений гарантируется в конечном счете, но у отправителя не будет информации о времени доставки и результатах выполнения операции (или ошибках).

К сущностям можно обращаться из клиентских функций, функций оркестрации и функций сущности. Не все формы обмена данными поддерживаются для всех контекстов.

* Из клиентов можно отправлять сигналы и считывать состояние сущности.
* Из средств оркестрации можно отправлять сигналы и вызовы сущности.
* Из сущностей можно только отправлять сигналы.

Ниже представлены несколько примеров с разными способами доступа к сущностям.

### <a name="example-client-signals-an-entity"></a>Пример сигнал к сущности из клиента

Для доступа к сущностям из обычной функции Azure, также известной как клиентская функция, используйте [привязку клиента сущности](durable-functions-bindings.md#entity-client). В следующем примере показана функция, активируемая очередью, которая сигнализирует сущностям, использующим эту привязку.

# <a name="c"></a>[C#](#tab/csharp)

> [!NOTE]
> Для простоты в этих примерах используется слабо типизированный синтаксис для обращений к сущностям. Как правило, мы рекомендуем [обращаться к сущностям через интерфейсы](durable-functions-dotnet-entities.md#accessing-entities-through-interfaces), так как они обеспечивают дополнительную проверку типов.

```csharp
[FunctionName("AddFromQueue")]
public static Task Run(
    [QueueTrigger("durable-function-trigger")] string input,
    [DurableClient] IDurableEntityClient client)
{
    // Entity operation input comes from the queue message content.
    var entityId = new EntityId(nameof(Counter), "myCounter");
    int amount = int.Parse(input);
    return client.SignalEntityAsync(entityId, "Add", amount);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function (context) {
    const client = df.getClient(context);
    const entityId = new df.EntityId("Counter", "myCounter");
    await client.signalEntity(entityId, "add", 1);
};
```

# <a name="python"></a>[Python](#tab/python)

```Python
from azure.durable_functions import DurableOrchestrationClient
import azure.functions as func


async def main(req: func.HttpRequest, starter: str, message):
    client = DurableOrchestrationClient(starter)
    entityId = df.EntityId("Counter", "myCounter")
    await client.signal_entity(entityId, "add", 1)
```

---

*Сигнализация* означает, что вызов API сущности является односторонним и асинхронным. Клиентская функция не может получить информацию о том, когда сущность обработала операцию. Также клиентская функция не может получить результирующие значения и (или) исключения. 

### <a name="example-client-reads-an-entity-state"></a>Пример клиент считывает состояние сущности

Клиентские функции могут также запрашивать состояние сущности, как показано в примере ниже:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("QueryCounter")]
public static async Task<HttpResponseMessage> Run(
    [HttpTrigger(AuthorizationLevel.Function)] HttpRequestMessage req,
    [DurableClient] IDurableEntityClient client)
{
    var entityId = new EntityId(nameof(Counter), "myCounter");
    EntityStateResponse<JObject> stateResponse = await client.ReadEntityStateAsync<JObject>(entityId);
    return req.CreateResponse(HttpStatusCode.OK, stateResponse.EntityState);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function (context) {
    const client = df.getClient(context);
    const entityId = new df.EntityId("Counter", "myCounter");
    const stateResponse = await client.readEntityState(entityId);
    return stateResponse.entityState;
};
```

# <a name="python"></a>[Python](#tab/python)

> [!NOTE]
> В настоящее время Python не поддерживает чтение состояний сущностей из клиента. Вместо этого используйте метод `callEntity` оркестратора.

---

Запросы о состоянии объектов отправляются в хранилище отслеживания Устойчивых сущностей и возвращают последнее сохраненное состояние сущности. Это состояние всегда является "фиксированным", то есть не может быть временным состоянием на период выполнения операции. Но возвращаемое состояние может быть устаревшим по сравнению с текущим состоянием сущности, сохраненным в памяти. Только оркестрации могут считывать состояние сущности в памяти, как описано в разделе ниже.

### <a name="example-orchestration-signals-and-calls-an-entity"></a>Пример сигнал и вызов сущности из оркестрации

Функции оркестрации могут обращаться к сущностям с помощью API-интерфейсов в [привязке триггера оркестрации](durable-functions-bindings.md#orchestration-trigger). В следующем примере кода показан вызов функции оркестрации и сигнализация сущности `Counter`.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("CounterOrchestration")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var entityId = new EntityId(nameof(Counter), "myCounter");

    // Two-way call to the entity which returns a value - awaits the response
    int currentValue = await context.CallEntityAsync<int>(entityId, "Get");
    if (currentValue < 10)
    {
        // One-way signal to the entity which updates the value - does not await a response
        context.SignalEntity(entityId, "Add", 1);
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context){
    const entityId = new df.EntityId("Counter", "myCounter");

    // Two-way call to the entity which returns a value - awaits the response
    currentValue = yield context.df.callEntity(entityId, "get");
});
```

> [!NOTE]
> JavaScript в настоящее время не поддерживает передачу сигнала сущности из оркестратора. Используйте вместо этого `callEntity`.

# <a name="python"></a>[Python](#tab/python)

```Python
import azure.functions as func
import azure.durable_functions as df


def orchestrator_function(context: df.DurableOrchestrationContext):
    entityId = df.EntityId("Counter", "myCounter")
    current_value = yield context.call_entity(entityId, "get")
    if current_value < 10:
        context.signal_entity(entityId, "add", 1)
    return state
```

---

Только оркестрации могут вызывать сущности и получать ответ, который может быть либо возвращаемым значением, либо исключением. Клиентские функции, использующие [клиентскую привязку](durable-functions-bindings.md#entity-client), могут сигнализировать только сущностям.

> [!NOTE]
> Вызов сущности из функции оркестрации аналогичен вызову [функции действия](durable-functions-types-features-overview.md#activity-functions) из функции оркестрации. Основное отличие заключается в том, что функции сущностей являются устойчивыми объектами с адресом (идентификатор сущности). Функции сущностей поддерживают указание имени операции. Функции действий, с другой стороны, не имеют состояния и концепции использования системы.

### <a name="example-entity-signals-an-entity"></a>Пример сигнал к сущности из сущности

Функция сущности может отправлять сигналы сущностям (в том числе сама себе) при выполнении любой операции.
Например, мы можем изменить представленный выше пример сущности `Counter` так, чтобы она отправляла некоторой сущности-наблюдателю сигнал о достижении заданной отметки, когда значение счетчика будет равно 100.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
   case "add":
        var currentValue = ctx.GetState<int>();
        var amount = ctx.GetInput<int>();
        if (currentValue < 100 && currentValue + amount >= 100)
        {
            ctx.SignalEntity(new EntityId("MonitorEntity", ""), "milestone-reached", ctx.EntityKey);
        }

        ctx.SetState(currentValue + amount);
        break;
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
    case "add":
        const amount = context.df.getInput();
        if (currentValue < 100 && currentValue + amount >= 100) {
            const entityId = new df.EntityId("MonitorEntity", "");
            context.df.signalEntity(entityId, "milestone-reached", context.df.instanceId);
        }
        context.df.setState(currentValue + amount);
        break;
```

# <a name="python"></a>[Python](#tab/python)

> [!NOTE]
> Python пока не поддерживает передачу сигналов между сущностями. Вместо этого для передачи сигналов в сущности используйте оркестратор.

---

## <a name="entity-coordination-currently-net-only"></a><a name="entity-coordination"></a>Координация сущностей (сейчас только для .NET)

Иногда приходится координировать операции между несколькими сущностями. Например, в приложении для банковских операций могут быть сущности, представляющие отдельные банковские счета. При передаче средств из одной учетной записи в другую необходимо убедиться, что на исходном счете имеется достаточно средств, а также что как исходный счет, так и целевой обновляются в транзакционно согласованном виде.

### <a name="example-transfer-funds-c"></a>Пример передача средств (C#)

Следующий пример кода перемещает средства между двумя сущностями счетов с помощью функции оркестрации. Для координации обновлений сущностей необходимо использовать метод `LockAsync`, чтобы создать _критическую секцию_ в оркестрации:

> [!NOTE]
> Для простоты в этом примере повторно используется сущность `Counter`, определенная ранее. В реальных приложениях лучше определить более подробную сущность `BankAccount`.

```csharp
// This is a method called by an orchestrator function
public static async Task<bool> TransferFundsAsync(
    string sourceId,
    string destinationId,
    int transferAmount,
    IDurableOrchestrationContext context)
{
    var sourceEntity = new EntityId(nameof(Counter), sourceId);
    var destinationEntity = new EntityId(nameof(Counter), destinationId);

    // Create a critical section to avoid race conditions.
    // No operations can be performed on either the source or
    // destination accounts until the locks are released.
    using (await context.LockAsync(sourceEntity, destinationEntity))
    {
        ICounter sourceProxy = 
            context.CreateEntityProxy<ICounter>(sourceEntity);
        ICounter destinationProxy =
            context.CreateEntityProxy<ICounter>(destinationEntity);

        int sourceBalance = await sourceProxy.Get();

        if (sourceBalance >= transferAmount)
        {
            await sourceProxy.Add(-transferAmount);
            await destinationProxy.Add(transferAmount);

            // the transfer succeeded
            return true;
        }
        else
        {
            // the transfer failed due to insufficient funds
            return false;
        }
    }
}
```

В .NET `LockAsync` возвращает `IDisposable`, который завершает критическую секцию при удалении. Этот результат `IDisposable` можно использовать вместе с блоком `using`, чтобы получить синтаксическое представление критической секции.

В предыдущем примере функция оркестрации передала средства из исходной сущности в целевую. Метод `LockAsync` заблокировал сущности исходного и целевого счета. Такая блокировка гарантирует, что никакой другой клиент не сможет запросить или изменить состояние любого счета, пока логика оркестрации не выйдет из критической секции в конце инструкции `using`. Такое поведение эффективно препятствует возможности перерасхода с исходного счета.

> [!NOTE] 
> Когда оркестрация завершается (нормально или с ошибкой), все критические секции процесса завершаются неявным образом, а все блокировки освобождаются.

### <a name="critical-section-behavior"></a>Поведение критической секции

Метод `LockAsync` создает критическую секцию в оркестрации. Эти критические секции предотвращают внесение перекрывающихся изменений в указанный набор сущностей. На внутреннем уровне API `LockAsync` отправляет операции блокировки в сущности и возвращает, когда получает ответное сообщение о получении блокировки от каждой из этих сущностей. Блокировка и разблокировка являются встроенными операциями, поддерживаемыми всеми сущностями.

Никакие операции от других клиентов не разрешены для сущности, находящейся в заблокированном состоянии. Такое поведение гарантирует, что только один экземпляр оркестрации может блокировать сущность за один раз. Если вызывающий объект пытается вызвать операцию для сущности, пока она заблокирована оркестрацией, такая операция будет помещена в очередь ожидающих операций. Ни одна из ожидающих операций не будет обработана до тех пор, пока оркестрация не снимет блокировку.

> [!NOTE] 
> Это поведение несколько отличается от примитивов синхронизации, используемых в большинстве языков программирования, таких как инструкция `lock` в C#. Например, в C# инструкция `lock` должна использоваться всеми потоками, чтобы обеспечить надлежащую синхронизацию в нескольких потоках. Однако для сущностей не требуется, чтобы все вызывающие объекты явно блокировали сущность. Если любой вызывающий объект блокирует сущность, все остальные операции с этой сущностью будут заблокированы и помещены в очередь после этой блокировки.

Блокировки сущностей являются устойчивыми, поэтому они сохраняются даже при повторном запуске процесса. Блокировки внутренне сохраняются как часть устойчивого состояния сущности.

В отличие от транзакций, критические секции не откатывают изменения автоматически в случае ошибок. Вместо этого любая обработка ошибок (откат или повтор) должна быть включена в код явным образом, например через отслеживание ошибок или исключений. Такое поведение реализовано намеренно. Автоматический откат всех результатов работы средств оркестрации в общем случае крайне сложен, а иногда и невозможен, поскольку средства оркестрации могут выполнять действия и вызовы внешних служб, не поддерживающих откат. Кроме того, попытка отката может завершиться сбоем, из-за чего потребуется дополнительная обработка.

### <a name="critical-section-rules"></a>Правила критической секции

В отличие от примитивов с низкоуровневой блокировкой, которые используются в большинстве языков программирования, критические секции *гарантированно не взаимоблокируются*. Чтобы избежать взаимоблокировок, мы налагаем следующие ограничения. 

* Критические секции не могут быть вложенными.
* Критические секции не могут создавать подоркестрации.
* Критические разделы могут вызывать только те сущности, которые они заблокировали.
* Критические секции не могут вызывать одну и ту же сущность с помощью нескольких параллельных вызовов.
* Критические разделы могут сигнализировать только тем сущностям, которые не заблокированы.

Любые нарушения этих правил порождают ошибку в среде выполнения (например, `LockingRulesViolationException` в .NET) с сообщением о том, какое именно правило было нарушено.

## <a name="comparison-with-virtual-actors"></a>Сравнение с виртуальными субъектами

Многие из функций Устойчивых сущностей представляют собой [модель субъектов](https://en.wikipedia.org/wiki/Actor_model). Если вы уже знакомы с субъектами, вам могут быть знакомы многие из концепций, описанных в этой статье. Устойчивые сущности очень похожи на [виртуальные субъекты](https://research.microsoft.com/projects/orleans/) (зерна), которые известны по [проекту Orleans](http://dotnet.github.io/orleans/). Пример:

* К устойчивым сущностям можно обращаться с помощью идентификатора сущности.
* Операции с устойчивыми сущностями выполняются последовательно, по очереди, чтобы предотвратить состояние гонки.
* Устойчивые сущности создаются неявным образом при их вызове или получении сигнала для них.
* Если операции не выполняются, устойчивые сущности выгружаются из памяти без вывода сообщений.

Следует отметить несколько важных отличий.

* Для устойчивых сущностей устойчивость важнее, чем задержка. Таким образом они могут не подходить для приложений с требованиями к длительной задержке.
* В устойчивых сущностях не применяется встроенное время ожидания для сообщений. В проекте Orleans срок действия всех сообщений истекает после определенного настраиваемого периода. По умолчанию это 30 секунд.
* Сообщения, передаваемые между сущностями, доставляются надежно и по порядку. В проекте Orleans поддерживается гарантированная или упорядоченная доставка для содержимого, отправляемого через потоки, но не для всех сообщений между зернами.
* Шаблоны запросов и ответов в сущностях ограничены оркестрацией. Из сущностей допускается только односторонний обмен сообщениями (отправка "сигналов"), как в исходной субъектной модели. Это отличает их от зерен в проекте Orleans. 
* Устойчивые сущности не поддерживают взаимоблокировку. В проекте Orleans может происходить взаимоблокировка (и она не будет устранена до истечения времени ожидания).
* Устойчивые сущности можно использовать совместно с устойчивой оркестрацией вместе с поддержкой механизмов распределенной блокировки. 

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Изучить руководство для разработчиков по устойчивым сущностям в .NET](durable-functions-dotnet-entities.md)

> [!div class="nextstepaction"]
> [Task hubs in Durable Functions (Azure Functions)](durable-functions-task-hubs.md) (Центры задач в устойчивых функциях (Функции Azure))
