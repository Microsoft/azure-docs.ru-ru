---
title: Обработка ошибок в устойчивых функциях — Azure
description: Сведения о том, как обрабатывать ошибки в расширении устойчивых функций для Функций Azure.
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: azfuncdf
ms.openlocfilehash: 023f9dfcc421935c3f7515e847108925d5e5521e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97673653"
---
# <a name="handling-errors-in-durable-functions-azure-functions"></a>Обработка ошибок в устойчивых функциях (Функции Azure)

Согласованность устойчивых функций реализована в коде и может использовать встроенные функции обработки ошибок языка программирования. В действительности нет никаких новых концепций, которые необходимо изучить, чтобы добавить в согласование обработку ошибок и компенсацию. Однако следует помнить о некоторых особенностях поведения.

## <a name="errors-in-activity-functions"></a>Ошибки в функциях действий

Любое исключение, которое вызывается в функции действия, передается обратно в функцию Orchestrator и выдается как `FunctionFailedException` . Вы можете написать код для обработки и компенсации ошибок, соответствующий вашим потребностям в функции оркестратора.

Например, рассмотрим следующую функцию оркестратора, которая перемещает средства с одного счета на другой:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("TransferFunds")]
public static async Task Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var transferDetails = context.GetInput<TransferOperation>();

    await context.CallActivityAsync("DebitAccount",
        new
        {
            Account = transferDetails.SourceAccount,
            Amount = transferDetails.Amount
        });

    try
    {
        await context.CallActivityAsync("CreditAccount",
            new
            {
                Account = transferDetails.DestinationAccount,
                Amount = transferDetails.Amount
            });
    }
    catch (Exception)
    {
        // Refund the source account.
        // Another try/catch could be used here based on the needs of the application.
        await context.CallActivityAsync("CreditAccount",
            new
            {
                Account = transferDetails.SourceAccount,
                Amount = transferDetails.Amount
            });
    }
}
```

> [!NOTE]
> Предыдущие примеры C# предназначены для Устойчивые функции 2. x. Для Устойчивые функции 1. x необходимо использовать `DurableOrchestrationContext` вместо `IDurableOrchestrationContext` . Дополнительные сведения о различиях между версиями см. в статье [устойчивые функции версии](durable-functions-versions.md) .

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const transferDetails = context.df.getInput();

    yield context.df.callActivity("DebitAccount",
        {
            account = transferDetails.sourceAccount,
            amount = transferDetails.amount,
        }
    );

    try {
        yield context.df.callActivity("CreditAccount",
            {
                account = transferDetails.destinationAccount,
                amount = transferDetails.amount,
            }
        );
    }
    catch (error) {
        // Refund the source account.
        // Another try/catch could be used here based on the needs of the application.
        yield context.df.callActivity("CreditAccount",
            {
                account = transferDetails.sourceAccount,
                amount = transferDetails.amount,
            }
        );
    }
});
```
# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    transfer_details = context.get_input()

    yield context.call_activity('DebitAccount', {
         'account': transfer_details['sourceAccount'],
         'amount' : transfer_details['amount']
    })

    try:
        yield context.call_activity('CreditAccount', {
                'account': transfer_details['destinationAccount'],
                'amount': transfer_details['amount'],
            })
    except:
        yield context.call_activity('CreditAccount', {
            'account': transfer_details['sourceAccount'],
            'amount': transfer_details['amount']
        })

main = df.Orchestrator.create(orchestrator_function)
```

---

Если первый вызов функции **кредитаккаунт** завершается неудачно, функция Orchestrator компенсирует денежные средства в исходной учетной записи.

## <a name="automatic-retry-on-failure"></a>Автоматическое повторение попыток при сбое

При вызове функций действий или функций суборкестрации можно указать автоматическую политику повтора. В следующем примере предпринимается попытка вызова функции до трех раз с ожиданием в течение 5 секунд между попытками:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("TimerOrchestratorWithRetry")]
public static async Task Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var retryOptions = new RetryOptions(
        firstRetryInterval: TimeSpan.FromSeconds(5),
        maxNumberOfAttempts: 3);

    await context.CallActivityWithRetryAsync("FlakyFunction", retryOptions, null);

    // ...
}
```

> [!NOTE]
> Предыдущие примеры C# предназначены для Устойчивые функции 2. x. Для Устойчивые функции 1. x необходимо использовать `DurableOrchestrationContext` вместо `IDurableOrchestrationContext` . Дополнительные сведения о различиях между версиями см. в статье [устойчивые функции версии](durable-functions-versions.md) .

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const firstRetryIntervalInMilliseconds = 5000;
    const maxNumberOfAttempts = 3;

    const retryOptions = 
        new df.RetryOptions(firstRetryIntervalInMilliseconds, maxNumberOfAttempts);

    yield context.df.callActivityWithRetry("FlakyFunction", retryOptions);

    // ...
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    first_retry_interval_in_milliseconds = 5000
    max_number_of_attempts = 3

    retry_options = df.RetryOptions(first_retry_interval_in_milliseconds, max_number_of_attempts)

    yield context.call_activity_with_retry('FlakyFunction', retry_options)

main = df.Orchestrator.create(orchestrator_function)
```

---

Вызов функции действия в предыдущем примере принимает параметр для настройки политики автоматической повторной попытки. Существует несколько вариантов настройки политики автоматического повтора.

* **Max number of attempts** (Максимальное число попыток): максимальное число повторных попыток.
* **First retry interval** (Интервал до первого повтора): время ожидания перед первой повторной попыткой.
* **Backoff coefficient** (Коэффициент отсрочки): коэффициент, позволяющий определить степень увеличения отсрочки. По умолчанию равен 1.
* **Max retry interval** (Максимальный интервал повтора): максимальное время ожидания между повторными попытками.
* **Retry timeout** (Время ожидания повтора): максимальное время, отведенное на выполнение повторных попыток. Поведение по умолчанию — бесконечное повторение.
* **Handle**: можно указать определяемый пользователем обратный вызов, чтобы определить, следует ли повторить попытку выполнения функции. 

> [!NOTE]
> Пользовательские обратные вызовы в настоящее время не поддерживаются Устойчивые функции в JavaScript ( `context.df.RetryOptions` ).


## <a name="function-timeouts"></a>Время ожидания функций

Может потребоваться отменить вызов функции в функции Orchestrator, если выполнение займет слишком много времени. Правильный способ сделать это — создать [устойчивый таймер](durable-functions-timers.md) с помощью `context.CreateTimer` (.NET), `context.df.createTimer` (JavaScript) или `context.create_timer` (Python) в сочетании с `Task.WhenAny` (.NET), `context.df.Task.any` (JavaScript) или `context.task_any` (Python), как показано в следующем примере:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("TimerOrchestrator")]
public static async Task<bool> Run([OrchestrationTrigger] IDurableOrchestrationContext context)
{
    TimeSpan timeout = TimeSpan.FromSeconds(30);
    DateTime deadline = context.CurrentUtcDateTime.Add(timeout);

    using (var cts = new CancellationTokenSource())
    {
        Task activityTask = context.CallActivityAsync("FlakyFunction");
        Task timeoutTask = context.CreateTimer(deadline, cts.Token);

        Task winner = await Task.WhenAny(activityTask, timeoutTask);
        if (winner == activityTask)
        {
            // success case
            cts.Cancel();
            return true;
        }
        else
        {
            // timeout case
            return false;
        }
    }
}
```

> [!NOTE]
> Предыдущие примеры C# предназначены для Устойчивые функции 2. x. Для Устойчивые функции 1. x необходимо использовать `DurableOrchestrationContext` вместо `IDurableOrchestrationContext` . Дополнительные сведения о различиях между версиями см. в статье [устойчивые функции версии](durable-functions-versions.md) .

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");
const moment = require("moment");

module.exports = df.orchestrator(function*(context) {
    const deadline = moment.utc(context.df.currentUtcDateTime).add(30, "s");

    const activityTask = context.df.callActivity("FlakyFunction");
    const timeoutTask = context.df.createTimer(deadline.toDate());

    const winner = yield context.df.Task.any([activityTask, timeoutTask]);
    if (winner === activityTask) {
        // success case
        timeoutTask.cancel();
        return true;
    } else {
        // timeout case
        return false;
    }
});
```
# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df
from datetime import datetime, timedelta

def orchestrator_function(context: df.DurableOrchestrationContext):
    deadline = context.current_utc_datetime + timedelta(seconds = 30)
    
    activity_task = context.call_activity('FlakyFunction')
    timeout_task = context.create_timer(deadline)

    winner = yield context.task_any(activity_task, timeout_task)
    if winner == activity_task:
        timeout_task.cancel()
        return True
    else:
        return False

main = df.Orchestrator.create(orchestrator_function)
```

---

> [!NOTE]
> Этот механизм фактически не прекращает текущее выполнение функции действия. Вместо этого он просто позволяет функции оркестратора игнорировать результат и двигаться дальше. Дополнительные сведения см. в документации по [таймерам](durable-functions-timers.md#usage-for-timeout).

## <a name="unhandled-exceptions"></a>необработанных исключений.

Если функция оркестратора завершается сбоем с необработанным исключением, сведения об этом исключении регистрируются в журнале, и экземпляр завершает работу с состоянием `Failed`.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Сведения об оркестрации нескончаемые](durable-functions-eternal-orchestrations.md)

> [!div class="nextstepaction"]
> [Дополнительные сведения для диагностики неполадок](durable-functions-diagnostics.md)
