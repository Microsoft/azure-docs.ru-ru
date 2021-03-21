---
title: Обработка внешних событий в устойчивых функциях в Azure
description: Сведения о том, как обрабатывать внешние события в расширении устойчивых функций для Функций Azure.
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: azfuncdf
ms.openlocfilehash: c08306edcea02a9207ab5a15eb62b7fffc2ecb44
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99576335"
---
# <a name="handling-external-events-in-durable-functions-azure-functions"></a>Обработка внешних событий в устойчивых функциях (Функции Azure)

Функции оркестратора могут ожидать передачи внешних событий. Эта возможность [устойчивых функций](durable-functions-overview.md) часто используется для обработки действий человека или других внешних триггеров.

> [!NOTE]
> Внешние события это односторонние асинхронные операции. Они не подходят для ситуаций, где клиенту, отправляющему событие, требуется синхронный ответ функции оркестратора.

## <a name="wait-for-events"></a>Ожидание событий

Методы [ваитфорекстерналевент](/dotnet/api/microsoft.azure.webjobs.durableorchestrationcontextbase.waitforexternalevent?view=azure-dotnet-legacy) (.NET), `waitForExternalEvent` (JavaScript) и `wait_for_external_event` (Python) [привязки триггера оркестрации](durable-functions-bindings.md#orchestration-trigger) позволяют функции Orchestrator асинхронно ждать и прослушивать внешнее событие. Функция оркестратора, ожидающая передачи данных, объявляет *имя* события и *форму данных*, которую ожидает получить.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("BudgetApproval")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    bool approved = await context.WaitForExternalEvent<bool>("Approval");
    if (approved)
    {
        // approval granted - do the approved action
    }
    else
    {
        // approval denied - send a notification
    }
}
```

> [!NOTE]
> Предыдущий код C# предназначен для Устойчивые функции 2. x. Для Устойчивые функции 1. x необходимо использовать `DurableOrchestrationContext` вместо `IDurableOrchestrationContext` . Дополнительные сведения о различиях между версиями см. в статье [устойчивые функции версии](durable-functions-versions.md) .

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const approved = yield context.df.waitForExternalEvent("Approval");
    if (approved) {
        // approval granted - do the approved action
    } else {
        // approval denied - send a notification
    }
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    approved = context.wait_for_external_event('Approval')
    if approved:
        # approval granted - do the approved action
    else:
        # approval denied - send a notification

main = df.Orchestrator.create(orchestrator_function)
```

---

В предыдущем примере ожидается передача определенного одиночного события и предпринимается действие при его получении.

Можно одновременно прослушивать несколько событий, как показано в следующем примере, в котором ожидается одно из трех возможных уведомлений о событиях.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("Select")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var event1 = context.WaitForExternalEvent<float>("Event1");
    var event2 = context.WaitForExternalEvent<bool>("Event2");
    var event3 = context.WaitForExternalEvent<int>("Event3");

    var winner = await Task.WhenAny(event1, event2, event3);
    if (winner == event1)
    {
        // ...
    }
    else if (winner == event2)
    {
        // ...
    }
    else if (winner == event3)
    {
        // ...
    }
}
```

> [!NOTE]
> Предыдущий код C# предназначен для Устойчивые функции 2. x. Для Устойчивые функции 1. x необходимо использовать `DurableOrchestrationContext` вместо `IDurableOrchestrationContext` . Дополнительные сведения о различиях между версиями см. в статье [устойчивые функции версии](durable-functions-versions.md) .

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const event1 = context.df.waitForExternalEvent("Event1");
    const event2 = context.df.waitForExternalEvent("Event2");
    const event3 = context.df.waitForExternalEvent("Event3");

    const winner = yield context.df.Task.any([event1, event2, event3]);
    if (winner === event1) {
        // ...
    } else if (winner === event2) {
        // ...
    } else if (winner === event3) {
        // ...
    }
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    event1 = context.wait_for_external_event('Event1')
    event2 = context.wait_for_external_event('Event2')
    event3 = context.wait_for_external_event('Event3')

    winner = context.task_any([event1, event2, event3])
    if winner == event1:
        # ...
    elif winner == event2:
        # ...
    elif winner == event3:
        # ...

main = df.Orchestrator.create(orchestrator_function)
```

---

В предыдущем примере ожидается *любое* из нескольких событий. Также возможно реализовать ожидание *всех* событий.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("NewBuildingPermit")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    string applicationId = context.GetInput<string>();

    var gate1 = context.WaitForExternalEvent("CityPlanningApproval");
    var gate2 = context.WaitForExternalEvent("FireDeptApproval");
    var gate3 = context.WaitForExternalEvent("BuildingDeptApproval");

    // all three departments must grant approval before a permit can be issued
    await Task.WhenAll(gate1, gate2, gate3);

    await context.CallActivityAsync("IssueBuildingPermit", applicationId);
}
```

> [!NOTE]
> Предыдущий код предназначен для Устойчивые функции 2. x. Для Устойчивые функции 1. x необходимо использовать `DurableOrchestrationContext` вместо `IDurableOrchestrationContext` . Дополнительные сведения о различиях между версиями см. в статье [устойчивые функции версии](durable-functions-versions.md) .

Если полезные данные события невозможно преобразовать в ожидаемый тип `T`, в .NET выдается исключение.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const applicationId = context.df.getInput();

    const gate1 = context.df.waitForExternalEvent("CityPlanningApproval");
    const gate2 = context.df.waitForExternalEvent("FireDeptApproval");
    const gate3 = context.df.waitForExternalEvent("BuildingDeptApproval");

    // all three departments must grant approval before a permit can be issued
    yield context.df.Task.all([gate1, gate2, gate3]);

    yield context.df.callActivity("IssueBuildingPermit", applicationId);
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    application_id = context.get_input()
    
    gate1 = context.wait_for_external_event('CityPlanningApproval')
    gate2 = context.wait_for_external_event('FireDeptApproval')
    gate3 = context.wait_for_external_event('BuildingDeptApproval')

    yield context.task_all([gate1, gate2, gate3])
    yield context.call_activity('IssueBuildingPermit', application_id)

main = df.Orchestrator.create(orchestrator_function)
```

---

`WaitForExternalEvent` Ожидание некоторых входных данных в течение неограниченного времени.  Приложение-функцию можно безопасно выгрузить во время ожидания. Когда событие поступит в этот экземпляр оркестрации, он автоматически активируется и немедленно обработает событие.

> [!NOTE]
> Если приложение-функция использует план потребления, плата за выставление счетов не взимается, пока функция Orchestrator ожидает задачу от `WaitForExternalEvent` (.NET), `waitForExternalEvent` (JavaScript) или `wait_for_external_event` (Python), независимо от того, как долго она ожидает.

## <a name="send-events"></a>Отправка событий

Для отправки внешнего события в оркестрации можно использовать методы [RaiseEventAsync](/dotnet/api/microsoft.azure.webjobs.durableorchestrationclientbase.raiseeventasync?view=azure-dotnet-legacy) (.NET) или `raiseEventAsync` (JavaScript). Эти методы предоставляются привязкой [клиента оркестрации](durable-functions-bindings.md#orchestration-client) . Чтобы отправить внешнее событие в оркестрации, можно также использовать встроенный [API-интерфейс генерации событий](durable-functions-http-api.md#raise-event) .

Вызванное событие включает в себя *идентификатор экземпляра*, *EventName* и *EVENTDATA* в качестве параметров. Функции Orchestrator обрабатывайте эти события с помощью `WaitForExternalEvent` интерфейсов API (.NET) или `waitForExternalEvent` (JavaScript). Для обработки события значение *EventName* должно совпадать с окончанием отправки и получения. Данные события также должны быть сериализуемыми в формат JSON.

На внутреннем уровне механизм "поднять событие" помещает в очередь сообщение, которое выбирается ожидающей функцией Orchestrator. Если экземпляр не ожидает *события с указанным именем*, сообщение о событии добавляется в очередь в памяти. Если экземпляр оркестрации позже начинает ожидать *события с этим именем*, он проверяет сообщения о событиях в очереди.

> [!NOTE]
> Если экземпляр оркестрации с указанным *идентификатором экземпляра* отсутствует, сообщение о событии отменяется.

Ниже приведен пример функции, активируемой с помощью очереди, которая отправляет событие "Approval" в экземпляр функции оркестратора. Идентификатор экземпляра оркестрации поступает из текста сообщения очереди.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("ApprovalQueueProcessor")]
public static async Task Run(
    [QueueTrigger("approval-queue")] string instanceId,
    [DurableClient] IDurableOrchestrationClient client)
{
    await client.RaiseEventAsync(instanceId, "Approval", true);
}
```

> [!NOTE]
> Предыдущий код C# предназначен для Устойчивые функции 2. x. Для Устойчивые функции 1. x необходимо использовать `OrchestrationClient` атрибут, а не `DurableClient` атрибут, и `DurableOrchestrationClient` тип параметра следует использовать вместо `IDurableOrchestrationClient` . Дополнительные сведения о различиях между версиями см. в статье [устойчивые функции версии](durable-functions-versions.md) .

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function(context, instanceId) {
    const client = df.getClient(context);
    await client.raiseEvent(instanceId, "Approval", true);
};
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

async def main(instance_id:str, starter: str) -> func.HttpResponse:
    client = df.DurableOrchestrationClient(starter)
    await client.raise_event(instance_id, 'Approval', True)
```

---

На внутреннем уровне `RaiseEventAsync` (.NET), `raiseEvent` (JavaScript) или `raise_event` (Python) помещает в очередь сообщение, которое забирается ожидающей функцией Orchestrator. Если экземпляр не ожидает *события с указанным именем*, сообщение о событии добавляется в очередь в памяти. Если экземпляр оркестрации позже начинает ожидать *события с этим именем*, он проверяет сообщения о событиях в очереди.

> [!NOTE]
> Если экземпляр оркестрации с указанным *идентификатором экземпляра* отсутствует, сообщение о событии отменяется.

### <a name="http"></a>HTTP

Ниже приведен пример HTTP-запроса, который вызывает событие "утверждение" в экземпляр оркестрации. 

```http
POST /runtime/webhooks/durabletask/instances/MyInstanceId/raiseEvent/Approval&code=XXX
Content-Type: application/json

"true"
```

В этом случае идентификатор экземпляра жестко закодирован как *минстанцеид*.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Узнайте, как реализовать обработку ошибок](durable-functions-error-handling.md)

> [!div class="nextstepaction"]
> [Запуск примера с ожиданием действий человека](durable-functions-phone-verification.md)