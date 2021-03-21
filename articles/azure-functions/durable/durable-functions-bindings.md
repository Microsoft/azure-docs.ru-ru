---
title: Привязки для устойчивых функций — Azure
description: Инструкции по использованию триггеров и привязок в расширении Устойчивых функций для Функций Azure.
ms.topic: conceptual
ms.date: 12/17/2019
ms.author: azfuncdf
ms.openlocfilehash: fe64945aeb71011e769ba4016ae36cb557f41f33
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102424263"
---
# <a name="bindings-for-durable-functions-azure-functions"></a>Привязки для устойчивых функций (Функции Azure)

Расширение [устойчивых функций](durable-functions-overview.md) содержит две новые привязки триггера, которые управляют выполнением функций действий и оркестратора. Он также содержит привязку к выходным данным, которая действует как клиент для среды выполнения устойчивых функций.

## <a name="orchestration-trigger"></a>Триггер оркестрации

Триггер оркестрации позволяет создавать [устойчивые функции Orchestrator](durable-functions-types-features-overview.md#orchestrator-functions). Этот триггер поддерживает запуск новых экземпляров функции оркестратора и возобновление существующих экземпляров функции оркестратора, которые "ожидают" задачу.

При использовании средств Visual Studio для Функций Azure триггер оркестрации настраивается с помощью атрибута .NET [OrchestrationTriggerAttribute](/dotnet/api/microsoft.azure.webjobs.extensions.durabletask.orchestrationtriggerattribute).

При написании функции оркестратора на языках сценариев (например,на JavaScript или C#) триггер оркестрации определяется следующим объектом JSON в массиве `bindings` файла *function.json*:

```json
{
    "name": "<Name of input parameter in function signature>",
    "orchestration": "<Optional - name of the orchestration>",
    "type": "orchestrationTrigger",
    "direction": "in"
}
```

* `orchestration` имя оркестрации, которое клиенты должны использовать, когда им нужно запустить новые экземпляры этой функции Orchestrator. Это необязательное свойство. Если не указан, используется имя функции.

Внутри эта привязка триггера опрашивает серии очередей в учетной записи хранения по умолчанию приложения-функции. Эти очереди представляют собой сведения о внутренней реализации расширения, поэтому они не настраиваются явно в свойствах привязки.

### <a name="trigger-behavior"></a>Поведение триггера

Ниже приведены сведения о триггере оркестрации:

* **Однопоточная обработка**. Один поток диспетчера используется для выполнения всех функций оркестратора на одном экземпляре узла. По этой причине убедитесь, что код функции оркестратора эффективен и не выполняет никаких операций ввода-вывода. Также важно убедиться, что этот поток не выполняет никакой асинхронной работы, кроме тех случаев, когда ожидаются типы задач, специфичные для устойчивых функций.
* **Обработка сообщений о сбое.** В триггерах оркестрации не поддерживаются сообщения о сбое.
* **Видимость сообщений**. Сообщения триггеров оркестрации удаляются из очереди и сохраняются невидимыми в течение настраиваемого промежутка времени. Видимость этих сообщений обновляется автоматически, пока приложение-функция выполняется и работоспособно.
* **Возвращаемые значения**. Возвращаемые значения сериализуются в JSON и сохраняются в таблице с журналом оркестрации в хранилище таблиц Azure. Эти возвращаемые значения может запрашивать привязка клиента оркестрации, описанная ниже.

> [!WARNING]
> Функции оркестратора никогда не должны использовать никаких входных и выходных привязок, отличных от привязки триггера оркестрации. Это может вызвать проблемы с расширением устойчивых функций, так как эти привязки могут не подчиняться правилам однопоточности и ввода-вывода. Если вы хотите использовать другие привязки, добавьте их в функцию действия, вызываемую из функции Orchestrator.

> [!WARNING]
> Функции оркестратора JavaScript никогда не должны быть объявлены как `async`.

### <a name="trigger-usage-net"></a>Использование триггера (.NET)

Привязка триггера оркестрации поддерживает входные и выходные данные. Ниже приведены некоторые сведения об обработке входных и выходных данных.

* **входные данные** . функции оркестрации .NET поддерживают только `DurableOrchestrationContext` в качестве типа параметра. Входные данные десериализации непосредственно в сигнатуре функции не поддерживаются. Код должен использовать `GetInput<T>` метод (.NET) или `getInput` (JavaScript) для выборки входных данных функции Orchestrator. Эти входные данные должны быть типа, который сериализуется в JSON.
* **Выходные данные**. Триггеры оркестрации поддерживают выходные значения, а также входные данные. Возвращаемое значение функции используется для назначения выходного значения и должно быть типа, который сериализуется в JSON. Если функция .NET возвращает `Task` или `void`, значение `null` будет сохранено как выходное.

### <a name="trigger-sample"></a>Пример триггера

В следующем примере кода показано, как может выглядеть самая простая функция Orchestrator "Hello World":

#### <a name="c"></a>C#

```csharp
[FunctionName("HelloWorld")]
public static string Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    string name = context.GetInput<string>();
    return $"Hello {name}!";
}
```
> [!NOTE]
> Предыдущий код предназначен для Устойчивые функции 2. x. Для Устойчивые функции 1. x необходимо использовать `DurableOrchestrationContext` вместо `IDurableOrchestrationContext` . Дополнительные сведения о различиях между версиями см. в статье [устойчивые функции версии](durable-functions-versions.md) .

#### <a name="javascript-functions-20-only"></a>JavaScript (только Функции 2.0)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const name = context.df.getInput();
    return `Hello ${name}!`;
});
```

> [!NOTE]
> `context`Объект в JavaScript не представляет DurableOrchestrationContext, а [контекст функции в целом](../functions-reference-node.md#context-object). Получить доступ к методам оркестрации можно через свойство `df` объекта `context`.

> [!NOTE]
> Для оркестраторов JavaScript нужно использовать `return`. Библиотека `durable-functions` отвечает за вызов метода `context.done`.

Большинство функций оркестратора вызывают функции действий. Пример "Hello World" демонстрирует, как вызвать функцию действия:

#### <a name="c"></a>C#

```csharp
[FunctionName("HelloWorld")]
public static async Task<string> Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    string name = context.GetInput<string>();
    string result = await context.CallActivityAsync<string>("SayHello", name);
    return result;
}
```

> [!NOTE]
> Предыдущий код предназначен для Устойчивые функции 2. x. Для Устойчивые функции 1. x необходимо использовать `DurableOrchestrationContext` вместо `IDurableOrchestrationContext` . Дополнительные сведения о различиях между версиями см. в статье [устойчивые функции версии](durable-functions-versions.md) .

#### <a name="javascript-functions-20-only"></a>JavaScript (только Функции 2.0)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const name = context.df.getInput();
    const result = yield context.df.callActivity("SayHello", name);
    return result;
});
```

## <a name="activity-trigger"></a>Триггер действия

Триггер действия позволяет создавать функции, которые вызываются функциями Orchestrator, называемыми [функциями действий](durable-functions-types-features-overview.md#activity-functions).

Если вы используете Visual Studio, триггер действия настраивается с помощью `ActivityTriggerAttribute` атрибута .NET.

Если вы используете VS Code или портал Azure для разработки приложений, триггер действия определяется следующим объектом JSON в массиве `bindings` файла *function.json*:

```json
{
    "name": "<Name of input parameter in function signature>",
    "activity": "<Optional - name of the activity>",
    "type": "activityTrigger",
    "direction": "in"
}
```

* `activity` является именем действия. Это значение представляет собой имя, которое функции Orchestrator используют для вызова этой функции действия. Это необязательное свойство. Если не указан, используется имя функции.

Внутри эта привязка триггера опрашивает очередь в учетной записи хранения по умолчанию приложения-функции. Эта очередь представляет собой сведения о внутренней реализации расширения, поэтому она не настраивается явно в свойствах привязки.

### <a name="trigger-behavior"></a>Поведение триггера

Ниже приведены сведения о триггере действия:

* **Работа с потоками**. В отличие от триггера оркестрации, триггеры действий не имеют каких-либо ограничений при работе с потоками или операциями ввода-вывода. Они могут рассматриваться как обычные функции.
* **Обработка сообщений о сбое**. В триггерах действий не поддерживаются сообщения о сбое.
* **Видимость сообщений**. Сообщения триггеров действий удаляются из очереди и сохраняются невидимыми в течение настраиваемого промежутка времени. Видимость этих сообщений обновляется автоматически, пока приложение-функция выполняется и работоспособно.
* **Возвращаемые значения**. Возвращаемые значения сериализуются в JSON и сохраняются в таблице с журналом оркестрации в хранилище таблиц Azure.

> [!WARNING]
> Внутренняя часть хранилища функций действий является частью реализации, и код пользователя не должен взаимодействовать с этими сущностями хранилища напрямую.

### <a name="trigger-usage-net"></a>Использование триггера (.NET)

Привязка триггера действий поддерживает входные и выходные данные, как и триггер оркестрации. Ниже приведены некоторые сведения об обработке входных и выходных данных.

* **входные данные** . функции действий .NET изначально используются `DurableActivityContext` в качестве типа параметра. Кроме того, функция действия может быть объявлена с любым типом параметра, который сериализуется в JSON. При использовании `DurableActivityContext` можно вызывать `GetInput<T>` для выборки и десериализации входных данных функции действия.
* **Выходные данные.** Функции действий поддерживают выходные значения, а также входные данные. Возвращаемое значение функции используется для назначения выходного значения и должно быть типа, который сериализуется в JSON. Если функция .NET возвращает `Task` или `void`, значение `null` будет сохранено как выходное.
* **Метаданные**. Функции действий .NET можно привязать к параметру `string instanceId`, чтобы получить идентификатор экземпляра родительской оркестрации.

### <a name="trigger-sample"></a>Пример триггера

В следующем примере кода показано, как может выглядеть простая функция действия "Hello World":

#### <a name="c"></a>C#

```csharp
[FunctionName("SayHello")]
public static string SayHello([ActivityTrigger] IDurableActivityContext helloContext)
{
    string name = helloContext.GetInput<string>();
    return $"Hello {name}!";
}
```

> [!NOTE]
> Предыдущий код предназначен для Устойчивые функции 2. x. Для Устойчивые функции 1. x необходимо использовать `DurableActivityContext` вместо `IDurableActivityContext` . Дополнительные сведения о различиях между версиями см. в статье [устойчивые функции версии](durable-functions-versions.md) .

Типом параметра по умолчанию для привязки `ActivityTriggerAttribute` .NET является `IDurableActivityContext`. Но триггеры действий .NET также поддерживают привязку непосредственно к JSON-сериализуемым типам (включая примитивные типы), поэтому эту же функцию можно упростить следующим образом:

```csharp
[FunctionName("SayHello")]
public static string SayHello([ActivityTrigger] string name)
{
    return $"Hello {name}!";
}
```

#### <a name="javascript-functions-20-only"></a>JavaScript (только Функции 2.0)

```javascript
module.exports = async function(context) {
    return `Hello ${context.bindings.name}!`;
};
```

Привязки JavaScript также можно передать в качестве дополнительных параметров, поэтому одну и ту же функцию можно упростить следующим образом:

```javascript
module.exports = async function(context, name) {
    return `Hello ${name}!`;
};
```


### <a name="using-input-and-output-bindings"></a>Использование входных и выходных привязок

В дополнение к привязке триггера действия можно использовать обычные входные и выходные привязки. Например, можно принять входные данные для привязки действия и отправить сообщение EventHub с помощью выходной привязки EventHub:

```json
{
  "bindings": [
    {
      "name": "message",
      "type": "activityTrigger",
      "direction": "in"
    },
    {
      "type": "eventHub",
      "name": "outputEventHubMessage",
      "connection": "EventhubConnectionSetting",
      "eventHubName": "eh_messages",
      "direction": "out"
  }
  ]
}
```

```javascript
module.exports = async function (context) {
    context.bindings.outputEventHubMessage = context.bindings.message;
};
```

## <a name="orchestration-client"></a>Клиент оркестрации

Привязка клиента оркестрации позволяет создавать функции, взаимодействующие с функциями Orchestrator. Эти функции иногда называют [клиентскими функциями](durable-functions-types-features-overview.md#client-functions). Например, с экземплярами оркестрации можно взаимодействовать следующим образом:

* Запустите их.
* Запросите их состояние.
* Завершите их.
* Отправляйте им события во время их работы.
* Очистите журнал экземпляра.

Если вы используете Visual Studio, можно выполнить привязку к клиенту оркестрации с помощью `OrchestrationClientAttribute` атрибута .NET для Устойчивые функции 1,0. Начиная с Устойчивые функции 2,0 можно выполнить привязку к клиенту оркестрации с помощью `DurableClientAttribute` атрибута .NET.

Если вы используете языки сценариев (например, файлы *. CSX* или *. js* ) для разработки, триггер ОРКЕСТРАЦИИ определяется следующим объектом JSON в `bindings` массиве *function.js*:

```json
{
    "name": "<Name of input parameter in function signature>",
    "taskHub": "<Optional - name of the task hub>",
    "connectionName": "<Optional - name of the connection string app setting>",
    "type": "orchestrationClient",
    "direction": "in"
}
```

* `taskHub` используется в ситуациях, когда несколько приложений-функций совместно используют одну и ту же учетную запись хранения, но должны быть изолированы друг от друга. Если значение не задано, по умолчанию используется значение из `host.json`. Оно должно соответствовать значению, которое используется целевыми функциями оркестратора.
* `connectionName` — имя параметра приложения, содержащего строку подключения к учетной записи хранения. Учетная запись хранения, представленная этой строкой подключения, должна быть той же, которая используется целевыми функциями оркестратора. Если она не указана, для приложения-функции используется строка подключения к учетной записи хранения по умолчанию.

> [!NOTE]
> В большинстве случаев рекомендуется исключить эти свойства и полагаться на поведение по умолчанию.

### <a name="client-usage"></a>Использование клиента

В функциях .NET обычно выполняется привязка к `IDurableOrchestrationClient` , которая предоставляет полный доступ ко всем API клиентов оркестрации, поддерживаемым устойчивые функции. В более старых выпусках Устойчивые функции 2. x вместо этого необходимо привязать к `DurableOrchestrationClient` классу. В JavaScript одни и те же API предоставляются объектом, возвращаемым из `getClient` . API-интерфейсы в клиентском объекте включают в себя:

* `StartNewAsync`
* `GetStatusAsync`
* `TerminateAsync`
* `RaiseEventAsync`
* `PurgeInstanceHistoryAsync`
* `CreateCheckStatusResponse`
* `CreateHttpManagementPayload`

Кроме того, функции .NET можно привязать к, `IAsyncCollector<T>` где `T` — `StartOrchestrationArgs` или `JObject` .

Дополнительные сведения об этих операциях см. в `IDurableOrchestrationClient` документации по API.

### <a name="client-sample-visual-studio-development"></a>Пример клиента (разработка в Visual Studio)

Ниже приведен пример функции, активируемой с помощью очереди, которая запускает оркестрацию HelloWorld.

```csharp
[FunctionName("QueueStart")]
public static Task Run(
    [QueueTrigger("durable-function-trigger")] string input,
    [DurableClient] IDurableOrchestrationClient starter)
{
    // Orchestration input comes from the queue message content.
    return starter.StartNewAsync("HelloWorld", input);
}
```

> [!NOTE]
> Предыдущий код C# предназначен для Устойчивые функции 2. x. Для Устойчивые функции 1. x необходимо использовать `OrchestrationClient` атрибут, а не `DurableClient` атрибут, и `DurableOrchestrationClient` тип параметра следует использовать вместо `IDurableOrchestrationClient` . Дополнительные сведения о различиях между версиями см. в статье [устойчивые функции версии](durable-functions-versions.md) .

### <a name="client-sample-not-visual-studio"></a>Пример клиента (не Visual Studio)

Если для разработки вы не используете Visual Studio, можно создать указанный ниже файл *function.json*. В этом примере показано, как настроить функцию, активируемую с помощью очереди, которая использует привязку клиента устойчивой оркестрации:

```json
{
  "bindings": [
    {
      "name": "input",
      "type": "queueTrigger",
      "queueName": "durable-function-trigger",
      "direction": "in"
    },
    {
      "name": "starter",
      "type": "durableClient",
      "direction": "in"
    }
  ]
}
```

> [!NOTE]
> Предыдущий формат JSON предназначен для Устойчивые функции 2. x. Для Устойчивые функции 1. x необходимо использовать `orchestrationClient` вместо в `durableClient` качестве типа триггера. Дополнительные сведения о различиях между версиями см. в статье [устойчивые функции версии](durable-functions-versions.md) .

Ниже приведены примеры конкретно для языка, которые запускают новые экземпляры функции оркестратора.

#### <a name="c-script-sample"></a>Пример скрипта C#

В следующем примере показано, как использовать привязку клиента долговременного оркестрации для запуска нового экземпляра функции из функции C#, активируемой с помощью очереди:

```csharp
#r "Microsoft.Azure.WebJobs.Extensions.DurableTask"

using Microsoft.Azure.WebJobs.Extensions.DurableTask;

public static Task Run(string input, IDurableOrchestrationClient starter)
{
    return starter.StartNewAsync("HelloWorld", input);
}
```

> [!NOTE]
> Предыдущий код предназначен для Устойчивые функции 2. x. Для Устойчивые функции 1. x необходимо использовать `DurableOrchestrationClient` тип параметра вместо `IDurableOrchestrationClient` . Дополнительные сведения о различиях между версиями см. в статье [устойчивые функции версии](durable-functions-versions.md) .

#### <a name="javascript-sample"></a>Пример на языке JavaScript

В следующем примере показано, как использовать привязку клиента устойчивой оркестрации для запуска нового экземпляра функции из функции JavaScript:

```javascript
const df = require("durable-functions");

module.exports = async function (context) {
    const client = df.getClient(context);
    return instanceId = await client.startNew("HelloWorld", undefined, context.bindings.input);
};
```

Дополнительные сведения о запуске экземпляров см. в статье [Управление экземплярами в устойчивых функциях (Функции Azure)](durable-functions-instance-management.md).

## <a name="entity-trigger"></a>Триггер сущности

Триггеры сущностей позволяют создавать [функции сущностей](durable-functions-entities.md). Этот триггер поддерживает обработку событий для конкретного экземпляра сущности.

При использовании средств Visual Studio для функций Azure триггер сущности настраивается с помощью `EntityTriggerAttribute` атрибута .NET.

> [!NOTE]
> Триггеры сущностей доступны начиная с Устойчивые функции 2. x.

Внутри эта привязка триггера опрашивает серии очередей в учетной записи хранения по умолчанию приложения-функции. Эти очереди представляют собой сведения о внутренней реализации расширения, поэтому они не настраиваются явно в свойствах привязки.

### <a name="trigger-behavior"></a>Поведение триггера

Ниже приведены некоторые замечания о триггере сущности.

* С **одним потоком**: один поток Dispatcher используется для обработки операций с определенной сущностью. Если несколько сообщений отправляются в одну сущность одновременно, операции будут обрабатываться один за раз.
* **Обработка подозрительных сообщений** — в триггерах сущностей нет поддержки подозрительных сообщений.
* **Видимость сообщений** — сообщения триггера сущности удаляются из очереди и остаются невидимыми в течение настраиваемого периода времени. Видимость этих сообщений обновляется автоматически, пока приложение-функция выполняется и работоспособно.
* **Возвращаемые значения** — функции сущностей не поддерживают возвращаемые значения. Существуют определенные API, которые можно использовать для сохранения состояния или передачи значений обратно в оркестрации.

Любые изменения состояния, внесенные в сущность во время ее выполнения, будут автоматически сохраняться после завершения выполнения.

### <a name="trigger-usage-net"></a>Использование триггера (.NET)

Каждая функция сущности имеет тип параметра `IDurableEntityContext` , который имеет следующие члены:

* **Имя_объекта**: имя выполняемой в данный момент сущности.
* **EntityKey**— ключ выполняемой в данный момент сущности.
* **EntityId**: идентификатор выполняющейся в данный момент сущности.
* **Имя_операции**: имя текущей операции.
* **Хасстате**: существует ли сущность, то есть имеет какое-то состояние. 
* **State \<TState> ()**: Возвращает текущее состояние сущности. Если он еще не существует, он создается и инициализируется в `default<TState>` . `TState`Параметр должен быть примитивным или сериализуемым типом JSON. 
* **State \<TState> (инитфунктион)**: Возвращает текущее состояние сущности. Если он еще не существует, он создается путем вызова предоставленного `initfunction` параметра. `TState`Параметр должен быть примитивным или сериализуемым типом JSON. 
* **Setstate (ARG)**: создает или обновляет состояние сущности. `arg`Параметр должен быть сериализуемым объектом или примитивом JSON.
* **Делетестате ()**: удаляет состояние сущности. 
* **Input \<TInput> ()**: получает входные данные для текущей операции. `TInput`Параметр типа должен быть примитивным или сериализуемым типом JSON.
* **Return (ARG)**: Возвращает значение для оркестрации, вызвавшего операцию. `arg`Параметр должен быть примитивным или сериализуемым объектом JSON.
* **Сигналентити (EntityId, счедуледтимеутк, Operation, input)**: отправляет одностороннее сообщение в сущность. `operation`Параметр должен быть строкой, отличной от NULL, `scheduledTimeUtc` которая должна быть значением времени в формате UTC, в которое вызывается операция, а `input` параметр должен быть примитивным или сериализуемым объектом JSON.
* **Креатеневорчестратион (орчестраторфунктионнаме, input)**: запускает новое согласование. `input`Параметр должен быть примитивным или сериализуемым объектом JSON.

`IDurableEntityContext`Доступ к объекту, переданному в функцию сущности, можно получить с помощью `Entity.Current` свойства Async-Local. Этот подход удобен при использовании модели программирования на основе классов.

### <a name="trigger-sample-c-function-based-syntax"></a>Пример триггера (синтаксис на основе функций C#)

Приведенный ниже код является примером простой сущности *Counter*, реализованной в формате устойчивой функции. Эта функция определяет три операции: `add`, `reset` и `get`, каждая из которых изменяет целочисленное значение состояния.

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
            ctx.Return(ctx.GetState<int>()));
            break;
    }
}
```

Дополнительные сведения о синтаксисе на основе функций и способах его использования см. в разделе [синтаксис на основе функций](durable-functions-dotnet-entities.md#function-based-syntax).

### <a name="trigger-sample-c-class-based-syntax"></a>Пример триггера (синтаксис на основе классов C#)

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

> [!NOTE]
> Метод точки входа функции `[FunctionName]` с атрибутом *должен* быть объявлен `static` при использовании классов сущностей. Нестатические методы точек входа могут привести к инициализации нескольких объектов и возможно других неопределенных поведений.

Классы сущностей имеют специальные механизмы для взаимодействия с привязками и внедрением зависимостей .NET. Дополнительные сведения см. в разделе [Конструирование сущностей](durable-functions-dotnet-entities.md#entity-construction).

### <a name="trigger-sample-javascript"></a>Пример триггера (JavaScript)

Следующий код является примером простой сущности *счетчика* , реализованной в виде устойчивой функции, написанной на JavaScript. Эта функция определяет три операции: `add`, `reset` и `get`, каждая из которых изменяет целочисленное значение состояния.

**function.json**
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

**index.js**
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

> [!NOTE]
> Устойчивые сущности доступны в JavaScript начиная с версии **1.3.0** пакета npm `durable-functions`.

## <a name="entity-client"></a>Клиент сущности

Привязка клиента сущности позволяет асинхронно активировать [функции сущностей](#entity-trigger). Эти функции иногда называют [клиентскими функциями](durable-functions-types-features-overview.md#client-functions).

Если вы используете Visual Studio, можно выполнить привязку к клиенту сущности с помощью `DurableClientAttribute` атрибута .NET.

> [!NOTE]
> `[DurableClientAttribute]`Также можно использовать для привязки к [клиенту оркестрации](#orchestration-client).

Если вы используете языки сценариев (например, файлы *CSX* или *JS* ) для разработки, триггер СУЩНОСТИ определяется следующим объектом JSON в `bindings` массиве *function.js*:

```json
{
    "name": "<Name of input parameter in function signature>",
    "taskHub": "<Optional - name of the task hub>",
    "connectionName": "<Optional - name of the connection string app setting>",
    "type": "durableClient",
    "direction": "in"
}
```

* `taskHub` используется в ситуациях, когда несколько приложений-функций совместно используют одну и ту же учетную запись хранения, но должны быть изолированы друг от друга. Если значение не задано, по умолчанию используется значение из `host.json`. Это значение должно соответствовать значению, используемому целевыми функциями сущности.
* `connectionName` — имя параметра приложения, содержащего строку подключения к учетной записи хранения. Учетная запись хранения, представленная этой строкой подключения, должна быть той же, которая используется целевыми функциями сущности. Если она не указана, для приложения-функции используется строка подключения к учетной записи хранения по умолчанию.

> [!NOTE]
> В большинстве случаев рекомендуется опустить необязательные свойства и полагаться на поведение по умолчанию.

### <a name="entity-client-usage"></a>Использование клиента сущности

В функциях .NET обычно выполняется привязка к `IDurableEntityClient` , которая предоставляет полный доступ ко всем клиентским API, поддерживаемым устойчивыми сущностями. Также можно выполнить привязку к `IDurableOrchestrationClient` интерфейсу, который предоставляет доступ к клиентским API как для сущностей, так и для оркестрации. API-интерфейсы в клиентском объекте включают в себя:

* **Реадентитистатеасинк \<T>**: считывает состояние сущности. Он возвращает ответ, указывающий, существует ли целевая сущность, и, если да, то что ее состояние —.
* **Сигналентитясинк**: отправляет одностороннее сообщение в сущность и ожидает его постановки в очередь.
* **Листентитиесасинк**: запросы состояния нескольких сущностей. К сущностям можно обращаться по *имени* и *времени последней операции*.

Нет необходимости создавать целевую сущность перед отправкой сигнала. состояние сущности можно создать в функции сущности, которая обрабатывает сигнал.

> [!NOTE]
> Важно понимать, что "сигналы", отправленные клиентом, просто помещаются в очередь, чтобы обрабатываться асинхронно позже. В частности, `SignalEntityAsync` обычно возвращает значение до того, как сущность запустит операцию, и возврат возвращаемого значения или наблюдение за исключениями невозможен. Если требуются более надежные гарантии (например, для рабочих процессов), следует использовать *функции Orchestrator* , которые могут ожидать завершения операций сущностей и могут обрабатывать возвращаемые значения и наблюдать за исключениями.

### <a name="example-client-signals-entity-directly---c"></a>Пример. Клиент сообщает сущности напрямую — C #

Ниже приведен пример функции, активируемой в очереди, которая вызывает сущность "Counter".

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

### <a name="example-client-signals-entity-via-interface---c"></a>Пример: клиент сигнализирует о сущности через интерфейс — C #

По возможности рекомендуется [обращаться к сущностям через интерфейсы](durable-functions-dotnet-entities.md#accessing-entities-through-interfaces) , так как она обеспечивает дополнительную проверку типов. Например, предположим, что `Counter` упомянутая ранее сущность реализовала `ICounter` интерфейс, определенную следующим образом:

```csharp
public interface ICounter
{
    void Add(int amount);
    void Reset();
    Task<int> Get();
}

public class Counter : ICounter
{
    // ...
}
```

Затем клиентский код может использовать `SignalEntityAsync<ICounter>` для создания строго типизированного прокси-сервера:

```csharp
[FunctionName("UserDeleteAvailable")]
public static async Task AddValueClient(
    [QueueTrigger("my-queue")] string message,
    [DurableClient] IDurableEntityClient client)
{
    var target = new EntityId(nameof(Counter), "myCounter");
    int amount = int.Parse(message);
    await client.SignalEntityAsync<ICounter>(target, proxy => proxy.Add(amount));
}
```

`proxy`Параметр является динамически создаваемым экземпляром `ICounter` , который внутренне преобразует вызов в `Add` эквивалентный (нетипизированный) вызов `SignalEntityAsync` .

> [!NOTE]
> `SignalEntityAsync`API представляют односторонние операции. При возвращении интерфейсов сущностей `Task<T>` значение `T` параметра всегда будет равно null или `default` .

В частности, не имеет смысла подать сигнал `Get` операции, так как значение не возвращается. Вместо этого клиенты могут использовать либо `ReadStateAsync` для прямого доступа к состоянию счетчика, либо для запуска функции Orchestrator, которая вызывает `Get` операцию.

### <a name="example-client-signals-entity---javascript"></a>Пример: Клиент сообщает сущность — JavaScript

Ниже приведен пример функции, активируемой в очереди, которая сигнализирует сущности "Counter" в JavaScript.

**function.json**
```json
{
    "bindings": [
      {
        "name": "input",
        "type": "queueTrigger",
        "queueName": "durable-entity-trigger",
        "direction": "in",
      },
      {
        "name": "starter",
        "type": "durableClient",
        "direction": "in"
      }
    ],
    "disabled": false
  }
```

**index.js**
```javascript
const df = require("durable-functions");

module.exports = async function (context) {
    const client = df.getClient(context);
    const entityId = new df.EntityId("Counter", "myCounter");
    await context.df.signalEntity(entityId, "add", 1);
};
```

> [!NOTE]
> Устойчивые сущности доступны в JavaScript начиная с версии **1.3.0** пакета npm `durable-functions`.

<a name="host-json"></a>
## <a name="hostjson-settings"></a>Параметры файла host.json

[!INCLUDE [durabletask](../../../includes/functions-host-json-durabletask.md)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Встроенная Справочник по HTTP API для управления экземплярами](durable-functions-http-api.md)
