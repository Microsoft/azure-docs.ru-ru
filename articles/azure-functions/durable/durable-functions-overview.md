---
title: Обзор устойчивых функций — Azure
description: Общие сведения о расширении устойчивых функций для Функций Azure.
author: cgillum
ms.topic: overview
ms.date: 12/23/2020
ms.author: cgillum
ms.reviewer: azfuncdf
ms.openlocfilehash: d99f1bd97c8199de1bda12f28f3fcb31b697946f
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105031500"
---
# <a name="what-are-durable-functions"></a>Что такое Устойчивые функции?

*Устойчивые функции* — это расширение [Функций Azure](../functions-overview.md), которое позволяет писать функции с отслеживанием состояния в беcсерверной вычислительной среде. Расширение позволяет определять рабочие процессы с отслеживанием состояния, создавая [*функции оркестратора*](durable-functions-orchestrations.md), и сущности с отслеживанием состояния, создавая [*функции сущностей*](durable-functions-entities.md) с помощью модели программирования Функций Azure. В фоновом режиме расширение автоматически управляет состоянием, создает контрольные точки и перезагружается, позволяя вам сосредоточиться на бизнес-логике.

## <a name="supported-languages"></a><a name="language-support"></a>Поддерживаемые языки

Устойчивые функции в настоящее время поддерживают следующие языки.

* **C#**: обе [предварительно скомпилированные библиотеки классов](../functions-dotnet-class-library.md) и [сценарий C#](../functions-reference-csharp.md).
* **JavaScript**: поддерживается только для версии 2.x среды выполнения Функций Azure. Требуется расширение устойчивых функций версии 1.7.0 или более поздней версии. 
* **Python.** Требуется расширение Устойчивых функций 2.3.1 или более поздней версии.
* **F#**: предварительно скомпилированные библиотеки классов и сценарий F#. Сценарий F# (.fsx) поддерживается только в среде выполнения Функций Azure версии 1.x.
* **PowerShell**: поддержка Устойчивых функций в настоящее время предоставляется в общедоступной предварительной версии. Поддерживается только для версии 3.x среды выполнения Функций Azure и PowerShell 7. Требуется расширение Устойчивых функций версии 2.2.2 или более поздней. В настоящее время поддерживаются только следующие шаблоны: [Цепочка функций](#chaining), [Развертывание и объединение](#fan-in-out), [Асинхронные API-интерфейсы HTTP](#async-http).

Для доступа к последним функциям и обновлениям рекомендуется использовать последние версии расширения Устойчивых функций и библиотеки Устойчивых функций для конкретных языков. Узнайте больше о [версиях Устойчивых функций](durable-functions-versions.md).

Целью устойчивых функций является поддержка всех [языков Функций Azure](../supported-languages.md). См. [здесь](https://github.com/Azure/azure-functions-durable-extension/issues) список проблем Устойчивых функций для получения последних сведений о состоянии работы для поддержки дополнительных языков.

Подобно Функциям Azure, доступны шаблоны, которые помогут вам в разработке устойчивых функций с помощью [Visual Studio 2019](durable-functions-create-first-csharp.md), [Visual Studio Code](quickstart-js-vscode.md) и [портала Azure](durable-functions-create-portal.md).

## <a name="application-patterns"></a>Шаблоны приложений

Основной вариант использования Устойчивых функций — это упрощение комплексных требований координации с отслеживанием состояния в безсерверных приложениях. В следующих разделах описываются типичные шаблоны приложений, в которых можно эффективно использовать Устойчивые функции.

* [Цепочка функций](#chaining)
* [Развертывание и объединение](#fan-in-out)
* [Асинхронные API-интерфейсы HTTP](#async-http)
* [Мониторинг](#monitoring)
* [Участие пользователя](#human)
* [Агрегатор (сущности с отслеживанием состояния)](#aggregator)

### <a name="pattern-1-function-chaining"></a>Шаблон 1. Цепочка функций

В шаблоне цепочки функций последовательность функций выполняется в определенном порядке. В этом шаблоне выходные данные одной функции применяются к входным данным другой.

![Схема шаблона цепочки функций](./media/durable-functions-concepts/function-chaining.png)

Устойчивые функции можно использовать для реализации шаблона цепочки функций, как показано в следующем примере.

В этом примере значения `F1`, `F2`, `F3` и `F4` являются именами других функций в том же приложении-функции. Поток управления можно реализовать с помощью обычных принудительных конструкций программирования. Код выполняется сверху вниз. Код может включать в себя имеющуюся семантику языка потока управления, такую ​​как условные обозначения и циклы. Вы можете включить логику обработки ошибок в блоках `try`/`catch`/`finally`.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("Chaining")]
public static async Task<object> Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    try
    {
        var x = await context.CallActivityAsync<object>("F1", null);
        var y = await context.CallActivityAsync<object>("F2", x);
        var z = await context.CallActivityAsync<object>("F3", y);
        return  await context.CallActivityAsync<object>("F4", z);
    }
    catch (Exception)
    {
        // Error handling or compensation goes here.
    }
}
```

Вы можете использовать параметр `context` для вызова других функций по имени, передачи параметров и возврата выходных данных функции. Каждый раз, когда код вызывает `await`, платформа Устойчивых функций создает контрольные точки выполнения текущего экземпляра функции. Если процесс или виртуальная машина перезапускается во время выполнения, экземпляр функции возобновляется из предыдущего вызова `await`. Этот процесс описан в следующем разделе Шаблон 2: развертывание и объединение.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    try {
        const x = yield context.df.callActivity("F1");
        const y = yield context.df.callActivity("F2", x);
        const z = yield context.df.callActivity("F3", y);
        return    yield context.df.callActivity("F4", z);
    } catch (error) {
        // Error handling or compensation goes here.
    }
});
```

Вы можете использовать объект `context.df` для вызова других функций по имени, передачи параметров и возврата выходных данных функции. Каждый раз, когда код вызывает `yield`, платформа Устойчивых функций создает контрольные точки выполнения текущего экземпляра функции. Если процесс или виртуальная машина перезапускается во время выполнения, экземпляр функции возобновляется из предыдущего вызова `yield`. Этот процесс описан в следующем разделе Шаблон 2: развертывание и объединение.

> [!NOTE]
> Объект `context` в JavaScript представляет весь [контекст функции](../functions-reference-node.md#context-object). Доступ к контексту Устойчивых функций с помощью свойства `df` в основном контексте.

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df


def orchestrator_function(context: df.DurableOrchestrationContext):
    x = yield context.call_activity("F1", None)
    y = yield context.call_activity("F2", x)
    z = yield context.call_activity("F3", y)
    result = yield context.call_activity("F4", z)
    return result


main = df.Orchestrator.create(orchestrator_function)
```

Вы можете использовать объект `context` для вызова других функций по имени, передачи параметров и возврата выходных данных функции. Каждый раз, когда код вызывает `yield`, платформа Устойчивых функций создает контрольные точки выполнения текущего экземпляра функции. Если процесс или виртуальная машина перезапускается во время выполнения, экземпляр функции возобновляется из предыдущего вызова `yield`. Этот процесс описан в следующем разделе Шаблон 2: развертывание и объединение.

> [!NOTE]
> Объект `context` в Python представляет контекст оркестрации. Получите доступ к основному контексту Функций Azure, используя свойство `function_context` в контексте оркестрации.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```PowerShell
param($Context)

$X = Invoke-ActivityFunction -FunctionName 'F1'
$Y = Invoke-ActivityFunction -FunctionName 'F2' -Input $X
$Z = Invoke-ActivityFunction -FunctionName 'F3' -Input $Y
Invoke-ActivityFunction -FunctionName 'F4' -Input $Z
```

Вы можете использовать команду `Invoke-ActivityFunction` для вызова других функций по имени, передачи параметров и возврата выходных данных функции. Каждый раз, когда код вызывает `Invoke-ActivityFunction` без параметра `NoWait`, среда Устойчивых функций создает контрольные точки выполнения текущего экземпляра функции. Если процесс или виртуальная машина перезапускается во время выполнения, экземпляр функции возобновляется из предыдущего вызова `Invoke-ActivityFunction`. Этот процесс описан в следующем разделе Шаблон 2: развертывание и объединение.

---

### <a name="pattern-2-fan-outfan-in"></a>Шаблон 2. Развертывание и объединение

В шаблоне развертывания и объединения вы выполняете параллельно несколько функций, а затем ожидаете их завершения. Часто некоторые агрегирования завершаются по результатам, возвращаемым функциями.

![Схема шаблона развертывания и объединения](./media/durable-functions-concepts/fan-out-fan-in.png)

При использовании обычных функций вы можете выполнить развертывание за счет отправки функцией нескольких сообщений в очередь. Войти обратно гораздо сложнее. Чтобы выполнить объединение в обычной функции, напишите код для отслеживания момента, когда функции, активируемые очередью, завершаются, а затем их выходные значения сохраняются.

Расширение "Устойчивые функции" обрабатывает этот шаблон с помощью относительно простого кода:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("FanOutFanIn")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var parallelTasks = new List<Task<int>>();

    // Get a list of N work items to process in parallel.
    object[] workBatch = await context.CallActivityAsync<object[]>("F1", null);
    for (int i = 0; i < workBatch.Length; i++)
    {
        Task<int> task = context.CallActivityAsync<int>("F2", workBatch[i]);
        parallelTasks.Add(task);
    }

    await Task.WhenAll(parallelTasks);

    // Aggregate all N outputs and send the result to F3.
    int sum = parallelTasks.Sum(t => t.Result);
    await context.CallActivityAsync("F3", sum);
}
```

Процесс развертывания распределяется по нескольким экземплярам функции `F2`. И отслеживается с использованием динамического списка задач. `Task.WhenAll` вызывается для ожидания завершения всех вызванных функций. Затем выходные данные функции `F2` агрегируются из списка динамических задач и передаются функции `F3`.

Автоматическое создание контрольных точек, которое происходит при вызове `await` к `Task.WhenAll`, гарантирует, что возможный сбой или перезагрузка во время выполнения не потребуют перезапуска уже завершенной задачи.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const parallelTasks = [];

    // Get a list of N work items to process in parallel.
    const workBatch = yield context.df.callActivity("F1");
    for (let i = 0; i < workBatch.length; i++) {
        parallelTasks.push(context.df.callActivity("F2", workBatch[i]));
    }

    yield context.df.Task.all(parallelTasks);

    // Aggregate all N outputs and send the result to F3.
    const sum = parallelTasks.reduce((prev, curr) => prev + curr, 0);
    yield context.df.callActivity("F3", sum);
});
```

Процесс развертывания распределяется по нескольким экземплярам функции `F2`. И отслеживается с использованием динамического списка задач. `context.df.Task.all` API вызывается для ожидания завершения всех вызванных функций. Затем выходные данные функции `F2` агрегируются из списка динамических задач и передаются функции `F3`.

Автоматическое создание контрольных точек, которое происходит при вызове `yield` к `context.df.Task.all`, гарантирует, что возможный сбой или перезагрузка во время выполнения не потребуют перезапуска уже завершенной задачи.

# <a name="python"></a>[Python](#tab/python)

```python
import azure.durable_functions as df


def orchestrator_function(context: df.DurableOrchestrationContext):
    # Get a list of N work items to process in parallel.
    work_batch = yield context.call_activity("F1", None)

    parallel_tasks = [ context.call_activity("F2", b) for b in work_batch ]
    
    outputs = yield context.task_all(parallel_tasks)

    # Aggregate all N outputs and send the result to F3.
    total = sum(outputs)
    yield context.call_activity("F3", total)


main = df.Orchestrator.create(orchestrator_function)
```

Процесс развертывания распределяется по нескольким экземплярам функции `F2`. И отслеживается с использованием динамического списка задач. `context.task_all` API вызывается для ожидания завершения всех вызванных функций. Затем выходные данные функции `F2` агрегируются из списка динамических задач и передаются функции `F3`.

Автоматическое создание контрольных точек, которое происходит при вызове `yield` к `context.task_all`, гарантирует, что возможный сбой или перезагрузка во время выполнения не потребуют перезапуска уже завершенной задачи.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```PowerShell
param($Context)

# Get a list of work items to process in parallel.
$WorkBatch = Invoke-ActivityFunction -FunctionName 'F1'

$ParallelTasks =
    foreach ($WorkItem in $WorkBatch) {
        Invoke-ActivityFunction -FunctionName 'F2' -Input $WorkItem -NoWait
    }

$Outputs = Wait-ActivityFunction -Task $ParallelTasks

# Aggregate all outputs and send the result to F3.
$Total = ($Outputs | Measure-Object -Sum).Sum
Invoke-ActivityFunction -FunctionName 'F3' -Input $Total
```

Процесс развертывания распределяется по нескольким экземплярам функции `F2`. Обратите внимание, что параметр `NoWait` при вызове функции `F2` позволяет оркестратору продолжать вызов `F2` без завершения действия. И отслеживается с использованием динамического списка задач. Команда `Wait-ActivityFunction` вызывается для ожидания завершения всех вызванных функций. Затем выходные данные функции `F2` агрегируются из списка динамических задач и передаются функции `F3`.

Автоматическое создание контрольных точек, которое происходит при вызове `Wait-ActivityFunction`, гарантирует, что возможный сбой или перезагрузка во время выполнения не потребуют перезапуска уже завершенной задачи.

---

> [!NOTE]
> В редких случаях может произойти сбой в окне после завершения функции действия, но до того, как ее выполнение будет сохранено в журнале оркестрации. В этом случае функция действия будет повторно выполнена с начала после восстановления процесса.

### <a name="pattern-3-async-http-apis"></a>Шаблон 3. Асинхронные API-интерфейсы HTTP

Шаблон асинхронного API HTTP решает проблему координации состояния длительных операций с внешними клиентами. Распространенный способ реализации этого шаблона — наличие триггера конечной точки HTTP для выполнения длительного действия. Затем перенаправьте клиент на конечную точку состояния, которую клиент опрашивает для получения сведений о завершении операции.

![Схема шаблона API HTTP](./media/durable-functions-concepts/async-http-api.png)

Устойчивые функции предоставляют **встроенную поддержку** для этого шаблона, упрощая или даже удаляя код, который вам нужно написать для взаимодействия с длительными выполнениями функций. Например, в примерах из краткого руководства по Устойчивым функциям ([C#](durable-functions-create-first-csharp.md) и [JavaScript](quickstart-js-vscode.md)) показана простая команда REST, которую вы можете использовать для запуска новых экземпляров функции оркестратора. После запуска экземпляра расширение предоставляет API HTTP веб-перехватчика, которые запрашивают состояние функции оркестратора. 

В следующем примере показаны команды REST, которые запускают оркестратор и запрашивают его состояние. Для ясности в примере опущены некоторые данные протокола.

```
> curl -X POST https://myfunc.azurewebsites.net/api/orchestrators/DoWork -H "Content-Length: 0" -i
HTTP/1.1 202 Accepted
Content-Type: application/json
Location: https://myfunc.azurewebsites.net/runtime/webhooks/durabletask/instances/b79baf67f717453ca9e86c5da21e03ec

{"id":"b79baf67f717453ca9e86c5da21e03ec", ...}

> curl https://myfunc.azurewebsites.net/runtime/webhooks/durabletask/instances/b79baf67f717453ca9e86c5da21e03ec -i
HTTP/1.1 202 Accepted
Content-Type: application/json
Location: https://myfunc.azurewebsites.net/runtime/webhooks/durabletask/instances/b79baf67f717453ca9e86c5da21e03ec

{"runtimeStatus":"Running","lastUpdatedTime":"2019-03-16T21:20:47Z", ...}

> curl https://myfunc.azurewebsites.net/runtime/webhooks/durabletask/instances/b79baf67f717453ca9e86c5da21e03ec -i
HTTP/1.1 200 OK
Content-Length: 175
Content-Type: application/json

{"runtimeStatus":"Completed","lastUpdatedTime":"2019-03-16T21:20:57Z", ...}
```

Так как управление состоянием осуществляется средой выполнения Устойчивых функций, вам не нужно реализовывать собственный механизм отслеживания состояния.

Расширение "Устойчивые функции" предоставляет встроенные API HTTP, которые управляют длительными оркестрациями. Вы также можете реализовать этот шаблон самостоятельно, используя собственные триггеры функций (например, HTTP, очередь или Центры событий Azure) и [привязку клиента оркестрации](durable-functions-bindings.md#orchestration-client). Например, для активации действия прекращения можно использовать сообщение очереди. Или можно использовать триггер HTTP, защищенный политикой проверки подлинности Azure Active Directory, вместо встроенных API-интерфейсов HTTP с созданным ключом для проверки подлинности.

Дополнительные сведения см. в статье о [функциях HTTP](durable-functions-http-features.md), в которой объясняется, как можно предоставлять асинхронные длительные процессы по протоколу HTTP с помощью расширения "Устойчивые функции".

### <a name="pattern-4-monitor"></a><a name="monitoring"></a>Шаблон 4. Монитор

Шаблон мониторинга представляет собой гибкий, повторяющийся процесс в рабочем процессе. Например, повторение опроса, пока не будут выполнены определенные условия. Вы можете использовать регулярный [триггер таймера](../functions-bindings-timer.md) для активации основного сценария, например задание периодической очистки. Но интервал является статическим, что усложняет управление временем существования экземпляра. Вы можете использовать Устойчивые функции, чтобы создать гибкие интервалы повторения, управлять временем существования задачи и создавать несколько процессов мониторинга в рамках одной оркестрации.

Примером шаблона мониторинга является измененный сценарий асинхронных API-интерфейсов HTTP, приведенный ранее. Вместо предоставления конечной точки внешнему клиенту для мониторинга длительной операции, мониторинг использует внешнюю конечную точку, а затем ожидает изменение состояния.

![Схема шаблона мониторинга](./media/durable-functions-concepts/monitor.png)

Благодаря нескольким строкам кода можно использовать Устойчивые функции, чтобы создать несколько мониторингов, которые наблюдают за произвольными конечными точками. Работа мониторингов может приостановиться, если выполнено условие, или другая функция может использовать клиент оркестрации для завершения мониторинга. Можно изменить интервал мониторинга `wait` в зависимости от определенного условия (например, экспоненциальной задержки). 

В следующем коде реализуется простой мониторинг:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("MonitorJobStatus")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    int jobId = context.GetInput<int>();
    int pollingInterval = GetPollingInterval();
    DateTime expiryTime = GetExpiryTime();

    while (context.CurrentUtcDateTime < expiryTime)
    {
        var jobStatus = await context.CallActivityAsync<string>("GetJobStatus", jobId);
        if (jobStatus == "Completed")
        {
            // Perform an action when a condition is met.
            await context.CallActivityAsync("SendAlert", machineId);
            break;
        }

        // Orchestration sleeps until this time.
        var nextCheck = context.CurrentUtcDateTime.AddSeconds(pollingInterval);
        await context.CreateTimer(nextCheck, CancellationToken.None);
    }

    // Perform more work here, or let the orchestration end.
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");
const moment = require("moment");

module.exports = df.orchestrator(function*(context) {
    const jobId = context.df.getInput();
    const pollingInterval = getPollingInterval();
    const expiryTime = getExpiryTime();

    while (moment.utc(context.df.currentUtcDateTime).isBefore(expiryTime)) {
        const jobStatus = yield context.df.callActivity("GetJobStatus", jobId);
        if (jobStatus === "Completed") {
            // Perform an action when a condition is met.
            yield context.df.callActivity("SendAlert", machineId);
            break;
        }

        // Orchestration sleeps until this time.
        const nextCheck = moment.utc(context.df.currentUtcDateTime).add(pollingInterval, 's');
        yield context.df.createTimer(nextCheck.toDate());
    }

    // Perform more work here, or let the orchestration end.
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.durable_functions as df
import json
from datetime import timedelta 


def orchestrator_function(context: df.DurableOrchestrationContext):
    job = json.loads(context.get_input())
    job_id = job["jobId"]
    polling_interval = job["pollingInterval"]
    expiry_time = job["expiryTime"]

    while context.current_utc_datetime < expiry_time:
        job_status = yield context.call_activity("GetJobStatus", job_id)
        if job_status == "Completed":
            # Perform an action when a condition is met.
            yield context.call_activity("SendAlert", job_id)
            break

        # Orchestration sleeps until this time.
        next_check = context.current_utc_datetime + timedelta(seconds=polling_interval)
        yield context.create_timer(next_check)

    # Perform more work here, or let the orchestration end.


main = df.Orchestrator.create(orchestrator_function)
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Монитор в настоящее время не поддерживается в PowerShell.

---

При получении запроса создается экземпляр оркестрации для указанного идентификатора события. Экземпляр выполняет опрос состояния, пока не выполнится условие или не произойдет выход из цикла. Для управления интервалом опроса используется устойчивый таймер. После этого можно перейти к другим задачам или завершить оркестрацию. Когда `nextCheck` превышает `expiryTime`, монитор останавливается.

### <a name="pattern-5-human-interaction"></a>Шаблон 5. Участие пользователя

Многие автоматизированные процессы требуют некоторого участия пользователя. Трудность вовлечения людей в автоматизированный процесс заключается в том, что пользователи не так доступны и не так быстро реагируют, как облачные службы. Автоматизированный процесс должен учитывать это. Для этого используется время ожидания и логика компенсации.

Процесс утверждения является примером бизнес-процесса, который предполагает взаимодействие с пользователями. Утверждение от менеджера может потребоваться для отчета о расходах, превышающих определенную денежную сумму. Если менеджер не подтверждает отчет о расходах в течение 72 часов (возможно, он отправился в отпуск), начинается процесс эскалации, чтобы получить одобрение от кого-то другого (возможно, вышестоящего менеджера).

![Схема шаблона участия пользователя](./media/durable-functions-concepts/approval.png)

Этот шаблон в примере можно реализовать с помощью функции оркестратора. Оркестратор использует [устойчивый таймер](durable-functions-timers.md), чтобы запросить одобрение. Оркестратор начинает эскалацию, если истекло время ожидания. Он ожидает [внешнее событие](durable-functions-external-events.md) в виде уведомления, созданного в результате участия пользователя.

В этих примерах создается процесс утверждения для демонстрации шаблона участия пользователя:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("ApprovalWorkflow")]
public static async Task Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    await context.CallActivityAsync("RequestApproval", null);
    using (var timeoutCts = new CancellationTokenSource())
    {
        DateTime dueTime = context.CurrentUtcDateTime.AddHours(72);
        Task durableTimeout = context.CreateTimer(dueTime, timeoutCts.Token);

        Task<bool> approvalEvent = context.WaitForExternalEvent<bool>("ApprovalEvent");
        if (approvalEvent == await Task.WhenAny(approvalEvent, durableTimeout))
        {
            timeoutCts.Cancel();
            await context.CallActivityAsync("ProcessApproval", approvalEvent.Result);
        }
        else
        {
            await context.CallActivityAsync("Escalate", null);
        }
    }
}
```

Для создания устойчивого таймера вызовите `context.CreateTimer`. Уведомление получает `context.WaitForExternalEvent`. Затем `Task.WhenAny` вызывается для того, чтобы решить, следует ли ускорить процесс (сначала истечет время ожидания) или утвердить процесс (утверждение получено до истечения времени ожидания).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");
const moment = require('moment');

module.exports = df.orchestrator(function*(context) {
    yield context.df.callActivity("RequestApproval");

    const dueTime = moment.utc(context.df.currentUtcDateTime).add(72, 'h');
    const durableTimeout = context.df.createTimer(dueTime.toDate());

    const approvalEvent = context.df.waitForExternalEvent("ApprovalEvent");
    if (approvalEvent === yield context.df.Task.any([approvalEvent, durableTimeout])) {
        durableTimeout.cancel();
        yield context.df.callActivity("ProcessApproval", approvalEvent.result);
    } else {
        yield context.df.callActivity("Escalate");
    }
});
```

Для создания устойчивого таймера вызовите `context.df.createTimer`. Уведомление получает `context.df.waitForExternalEvent`. Затем `context.df.Task.any` вызывается для того, чтобы решить, следует ли ускорить процесс (сначала истечет время ожидания) или утвердить процесс (утверждение получено до истечения времени ожидания).

# <a name="python"></a>[Python](#tab/python)

```python
import azure.durable_functions as df
import json
from datetime import timedelta 


def orchestrator_function(context: df.DurableOrchestrationContext):
    yield context.call_activity("RequestApproval", None)

    due_time = context.current_utc_datetime + timedelta(hours=72)
    durable_timeout_task = context.create_timer(due_time)
    approval_event_task = context.wait_for_external_event("ApprovalEvent")

    winning_task = yield context.task_any([approval_event_task, durable_timeout_task])

    if approval_event_task == winning_task:
        durable_timeout_task.cancel()
        yield context.call_activity("ProcessApproval", approval_event_task.result)
    else:
        yield context.call_activity("Escalate", None)


main = df.Orchestrator.create(orchestrator_function)
```

Для создания устойчивого таймера вызовите `context.create_timer`. Уведомление получает `context.wait_for_external_event`. Затем `context.task_any` вызывается для того, чтобы решить, следует ли ускорить процесс (сначала истечет время ожидания) или утвердить процесс (утверждение получено до истечения времени ожидания).

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Участие пользователя в настоящее время не поддерживается в PowerShell.

---

Внешний клиент может доставить уведомление о событии в функцию оркестратора, находящуюся в состоянии ожидания, через [встроенные интерфейсы API HTTP](durable-functions-http-api.md#raise-event):

```bash
curl -d "true" http://localhost:7071/runtime/webhooks/durabletask/instances/{instanceId}/raiseEvent/ApprovalEvent -H "Content-Type: application/json"
```

Событие также можно вызвать с помощью клиента оркестрации устойчивых функций из другой функции в том же приложении-функции:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("RaiseEventToOrchestration")]
public static async Task Run(
    [HttpTrigger] string instanceId,
    [DurableClient] IDurableOrchestrationClient client)
{
    bool isApproved = true;
    await client.RaiseEventAsync(instanceId, "ApprovalEvent", isApproved);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function (context) {
    const client = df.getClient(context);
    const isApproved = true;
    await client.raiseEvent(instanceId, "ApprovalEvent", isApproved);
};
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.durable_functions as df


async def main(client: str):
    durable_client = df.DurableOrchestrationClient(client)
    is_approved = True
    await durable_client.raise_event(instance_id, "ApprovalEvent", is_approved)
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Участие пользователя в настоящее время не поддерживается в PowerShell.

---

### <a name="pattern-6-aggregator-stateful-entities"></a><a name="aggregator"></a>Шаблон 6. Агрегатор (сущности с отслеживанием состояния)

Шестой шаблон заключается в статистической обработке данных событий за определенный период времени в одну доступную для адресации *сущность*. В этом шаблоне данные, для которых выполняется агрегирование, могут поступать из нескольких источников, могут быть доставлены пакетами или разбросаны по длительным временным периодам. Агрегатору может потребоваться выполнить действия над данными событий по мере их поступления, а внешним клиентам может потребоваться запросить агрегированные данные.

![Схема агрегатора](./media/durable-functions-concepts/aggregator.png)

Сложности при реализации этого шаблона с обычными функциями без отслеживания состояния заключаются в том, что управление параллелизмом крайне усложняется. Нужно не только беспокоиться о том, что несколько потоков одновременно изменяют одни и те же данные, также важно и то, чтобы агрегатор одновременно выполнялся только на одной виртуальной машине.

[Устойчивые сущности](durable-functions-entities.md) можно использовать, чтобы легко реализовать этот шаблон как отдельную функцию.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("Counter")]
public static void Counter([EntityTrigger] IDurableEntityContext ctx)
{
    int currentValue = ctx.GetState<int>();
    switch (ctx.OperationName.ToLowerInvariant())
    {
        case "add":
            int amount = ctx.GetInput<int>();
            ctx.SetState(currentValue + amount);
            break;
        case "reset":
            ctx.SetState(0);
            break;
        case "get":
            ctx.Return(currentValue);
            break;
    }
}
```

Устойчивые сущности можно также моделировать как классы в .NET. Эта модель может быть полезной, если список операций является фиксированным и становится большим. Следующий пример представляет собой эквивалентную реализацию сущности `Counter` с помощью классов и методов .NET.

```csharp
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

# <a name="javascript"></a>[JavaScript](#tab/javascript)

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

```python
import logging
import json

import azure.functions as func
import azure.durable_functions as df


def entity_function(context: df.DurableOrchestrationContext):

    current_value = context.get_state(lambda: 0)
    operation = context.operation_name
    if operation == "add":
        amount = context.get_input()
        current_value += amount
        context.set_result(current_value)
    elif operation == "reset":
        current_value = 0
    elif operation == "get":
        context.set_result(current_value)
    
    context.set_state(current_value)

main = df.Entity.create(entity_function)
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Устойчивые сущности в настоящее время не поддерживаются в PowerShell.

---

Клиенты могут поставить в очередь *операции* (называемые также "сигнализацией") для функции сущности, использующей [привязку клиента сущности](durable-functions-bindings.md#entity-client).

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("EventHubTriggerCSharp")]
public static async Task Run(
    [EventHubTrigger("device-sensor-events")] EventData eventData,
    [DurableClient] IDurableEntityClient entityClient)
{
    var metricType = (string)eventData.Properties["metric"];
    var delta = BitConverter.ToInt32(eventData.Body, eventData.Body.Offset);

    // The "Counter/{metricType}" entity is created on-demand.
    var entityId = new EntityId("Counter", metricType);
    await entityClient.SignalEntityAsync(entityId, "add", delta);
}
```

> [!NOTE]
> Динамически создаваемые прокси-серверы также доступны в .NET для сигнализации сущностям типобезопасным способом. Кроме сигнализации, клиенты также могут запрашивать состояние функции сущности с помощью [типобезопасных методов](durable-functions-bindings.md#entity-client-usage) в привязке клиента оркестрации.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function (context) {
    const client = df.getClient(context);
    const entityId = new df.EntityId("Counter", "myCounter");
    await context.df.signalEntity(entityId, "add", 1);
};
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df


async def main(req: func.HttpRequest, starter: str) -> func.HttpResponse:
    client = df.DurableOrchestrationClient(starter)
    entity_id = df.EntityId("Counter", "myCounter")
    instance_id = await client.signal_entity(entity_id, "add", 1)
    return func.HttpResponse("Entity signaled")
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Устойчивые сущности в настоящее время не поддерживаются в PowerShell.

---

Функции сущности доступны в расширении [Устойчивые функции 2.0](durable-functions-versions.md) и более поздних версий для C#, JavaScript и Python.

## <a name="the-technology"></a>Технология

Расширение "Устойчивые функции" создано на [платформе устойчивых задач](https://github.com/Azure/durabletask), библиотеки с открытым кодом на GitHub для создания рабочих процессов в коде. Подобно тому, как Функции Azure являются бессерверным развитием веб-заданий Azure, Устойчивые функции — это бессерверное развитие платформы устойчивых задач. Платформа устойчивых задач широко используется как корпорацией Майкрософт, так и другими организациями для автоматизации критически важных процессов. Это естественный выбор для бессерверной среды функций Azure.

## <a name="code-constraints"></a>Ограничения кода

Чтобы обеспечить надежность и гарантировать длительное выполнение, в функциях оркестратора предусмотрен набор правил кода, которым нужно следовать. Дополнительные сведения см. в статье [Orchestrator function code constraints](durable-functions-code-constraints.md) (Ограничения кода функции оркестратора).

## <a name="billing"></a>Выставление счетов

Счета за устойчивые функции выставляются так же, как и за Функции Azure. Дополнительные сведения см. на странице [цен на Функции Azure](https://azure.microsoft.com/pricing/details/functions/). При выполнении функций оркестратора в рамках [плана потребления](../consumption-plan.md) Функций Azure необходимо учитывать некоторые особенности при выставлении счетов. Дополнительные сведения об этом см. в статье [Durable Functions Billing](durable-functions-billing.md) (Выставление счетов за Устойчивые функции).

## <a name="jump-right-in"></a>Приступайте к работе

Вы можете начать работу с Устойчивыми функциями менее чем за 10 минут, просмотрев одно из следующих кратких руководств по конкретным языкам.

* [Создание устойчивой функции на C#](durable-functions-create-first-csharp.md)
* [Создание устойчивой функции с помощью JavaScript](quickstart-js-vscode.md)
* [Использование Python с Visual Studio Code](quickstart-python-vscode.md)
* [Создание устойчивой функции в PowerShell](quickstart-powershell-vscode.md)

В этих кратких руководствах показано, как локально создать и протестировать устойчивую функцию "Hello world". Затем вы опубликуете код функции в Azure. Функция, которую вы создаете, организовывает и объединяет в цепочку вызовы других функций.

## <a name="learn-more"></a>Дополнительные сведения

В следующем видео показаны преимущества Устойчивых функций.

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Durable-Functions-in-Azure-Functions/player] 

Более подробное обсуждение Устойчивых функций и базовой технологии см. в следующем видео (оно ориентировано на .NET, но понятия также применимы и к другим поддерживаемым языкам):

> [!VIDEO https://channel9.msdn.com/Events/dotnetConf/2018/S204/player]

Так как Устойчивые функции — это дополнительное расширение решения [Функции Azure](../functions-overview.md), оно подходит не для всех приложений. Сравнение с другими технологиями оркестрации Azure см. в разделе [Сравнение служб "Функции Azure" и Azure Logic Apps](../functions-compare-logic-apps-ms-flow-webjobs.md#compare-azure-functions-and-azure-logic-apps).

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Durable Functions types and features (Azure Functions)](durable-functions-types-features-overview.md) (Типы и возможности Устойчивых функций (Функции Azure))
