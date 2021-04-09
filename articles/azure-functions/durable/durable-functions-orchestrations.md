---
title: Устойчивые оркестрации — Функции Azure
description: Общие сведения о функции оркестрации для Устойчивых функций Azure.
author: cgillum
ms.topic: overview
ms.date: 09/08/2019
ms.author: azfuncdf
ms.openlocfilehash: ba314963058389e171601407ff00411049eecd45
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97845424"
---
# <a name="durable-orchestrations"></a>Устойчивые оркестрации

Устойчивые функции — это расширение [Функций Azure](../functions-overview.md). *Функцию оркестратора* можно использовать для оркестрации выполнения других Устойчивых функций в приложении-функции. Функции оркестратора обладают следующими характеристиками.

* Функции оркестратора определяют рабочий процесс функции, используя процедурный код. Не требуются декларативные схемы или конструкторы.
* Функции оркестратора могут вызывать другие устойчивые функции синхронно и асинхронно. Выходные данные вызванной функции могут быть надежно сохранены в локальных переменных.
* Функции оркестратора являются устойчивыми и надежными. Ход выполнения автоматически фиксируется, когда функция ожидает или получает результат. Локальное состояние не теряется при перезапуске процесса или при перезагрузке виртуальной машины.
* Функции оркестратора могут быть длительными. Общее время существования *экземпляра оркестрации* может измеряться в секундах, днях, месяцах или никогда не заканчиваться.

В этой статье представлен обзор функций оркестратора и того, как они могут помочь в решении различных задач по разработке приложений. Если вы еще не знакомы с типами функций, доступных в приложении "Устойчивые функции", сначала прочитайте статью о [типах Устойчивых функций](durable-functions-types-features-overview.md).

## <a name="orchestration-identity"></a>Идентификатор оркестрации

Каждый *экземпляр* оркестрации обладает идентификатором экземпляра (также известный как *ИД экземпляра*). По умолчанию каждый идентификатор экземпляра является автоматически созданным глобальным уникальным идентификатором (GUID). Однако идентификаторы экземпляров также могут быть любыми строковыми значениями, созданными пользователем. Идентификатор каждого экземпляра оркестрации должен быть уникальным в пределах [центра задач](durable-functions-task-hubs.md).

Ниже приведены некоторые правила для идентификаторов экземпляров.

* Идентификаторы экземпляров должны содержать от 1 до 256 символов.
* Идентификаторы экземпляров не должны начинаться со знака `@`.
* Идентификаторы экземпляров не должны содержать символы `/`, `\`, `#` или `?`.
* Идентификаторы экземпляров не должны содержать управляющие символы.

> [!NOTE]
> Обычно рекомендуется по мере возможности использовать автоматически создаваемые идентификаторы экземпляров. Созданные пользователем идентификаторы экземпляров предназначены для сценариев, в которых существует сопоставление "один к одному" между экземпляром оркестрации и некоторой внешней сущностью приложения, такой как заказ на покупку или документ.

Идентификатор экземпляра оркестрации является обязательным параметром для большинства [операций управления экземпляром](durable-functions-instance-management.md). Они также важны для диагностики, например [для поиска в данных отслеживания оркестрации](durable-functions-diagnostics.md#application-insights) в Application Insights для устранения неполадок или аналитики. По этой причине рекомендуется сохранять созданные идентификаторы экземпляров за пределами текущей среды (например, в базе данных или в журналах приложений), где на них можно будет легко ссылаться позже.

## <a name="reliability"></a>Надежность

Функции оркестратора надежно сохраняют свое состояние выполнения с помощью конструктивного шаблона [Источник событий](/azure/architecture/patterns/event-sourcing). Вместо непосредственного сохранения текущего состояния оркестрации платформа устойчивых задач использует инкрементируемое хранилище для записи полной последовательности действий, взятых из функции оркестрации. Инкрементируемое хранилище обладает множеством преимуществ по сравнению с выполнением дампа полного состояния среды выполнения. К этим преимуществам относятся: повышение производительности, масштабируемость и скорость реагирования. Вы также получаете итоговую согласованность данных о транзакциях и все аудиторские следы и журналы. Аудиторские следы поддерживают надежные компенсирующие действия.

Устойчивые функции прозрачно используют источник событий. По сути оператор `await` (C#) или `yield` (JavaScript или Python) в функции оркестратора передает управление потоком оркестратора обратно диспетчеру платформы устойчивых задач. Затем диспетчер совершает любые новые действия, запланированные функцией оркестратора (например, вызов одной или нескольких дочерних функций или планирование устойчивого таймера) в хранилище. Это прозрачное действие фиксации добавляется в журнал выполнения экземпляра оркестрации. Журнал хранится в таблице хранилища. Затем действие фиксации добавляет сообщения в очередь для планирования фактической работы. На этом этапе функцию оркестрации можно выгрузить из памяти.

После того, как функции оркестрации получают больше работы (например, получено ответное сообщение или истек срок действия устойчивого таймера), оркестратор активируется и повторно запускает всю функцию с самого начала, чтобы перестроить локальное состояние. Если во время этого воспроизведения код пытается вызвать функцию (или выполнить любую другую асинхронную работу), платформа устойчивых задач просматривает журнал выполнения текущей оркестрации. Если он обнаруживает, что [функция действия](durable-functions-types-features-overview.md#activity-functions) уже выполнена и получен результат, он воспроизводит результат этой функции, а код оркестратора продолжает выполняться. Воспроизведение продолжится до тех пор, пока не завершится код функции или пока не будет запланирована новуя асинхронная работа.

> [!NOTE]
> Чтобы шаблон воспроизведения работал правильно и надежно, код функции оркестратора должен быть *детерминированным*. Дополнительные сведения об ограничениях кода для функций оркестратора см. в статье [Orchestrator function code constraints](durable-functions-code-constraints.md) (Ограничения кода для функций оркестратора).

> [!NOTE]
> Если функция оркестратора отправляет сообщения журнала, поведение при воспроизведении может привести к отправке дублированных сообщений журнала. Ознакомьтесь с разделом [Ведение журнала](durable-functions-diagnostics.md#app-logging), чтобы узнать больше о том, почему происходит такое поведение и как его устранить.

## <a name="orchestration-history"></a>Журнал оркестраций

Поведение источника событий в платформе устойчивых задач тесно связано с написанным кодом функции оркестратора. Предположим, у вас есть функция связывания действий оркестратора, например, следующая функция:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("E1_HelloSequence")]
public static async Task<List<string>> Run(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var outputs = new List<string>();

    outputs.Add(await context.CallActivityAsync<string>("E1_SayHello", "Tokyo"));
    outputs.Add(await context.CallActivityAsync<string>("E1_SayHello", "Seattle"));
    outputs.Add(await context.CallActivityAsync<string>("E1_SayHello", "London"));

    // returns ["Hello Tokyo!", "Hello Seattle!", "Hello London!"]
    return outputs;
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const output = [];
    output.push(yield context.df.callActivity("E1_SayHello", "Tokyo"));
    output.push(yield context.df.callActivity("E1_SayHello", "Seattle"));
    output.push(yield context.df.callActivity("E1_SayHello", "London"));

    // returns ["Hello Tokyo!", "Hello Seattle!", "Hello London!"]
    return output;
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    result1 = yield context.call_activity('SayHello', "Tokyo")
    result2 = yield context.call_activity('SayHello', "Seattle")
    result3 = yield context.call_activity('SayHello', "London")
    return [result1, result2, result3]

main = df.Orchestrator.create(orchestrator_function)
```
---

В каждой инструкции `await` (C#) или `yield` (JavaScript или Python) платформа устойчивых задач создает контрольные точки состояния выполнения функции в некоторых устойчивых внутренних частях хранилища (обычно это хранилище таблиц Azure). Это состояние также называется *журналом оркестрации*.

### <a name="history-table"></a>Таблица журнала

Как правило, платформа устойчивых задач выполняет следующие действия в каждой контрольной точке:

1. Сохраняет журнал выполнения в таблицах службы хранилища Azure.
2. Помещает в очередь сообщения для функций, которые хочет вызвать оркестратор.
3. Помещает в очередь сообщения для самого оркестратора, например сообщения устойчивого таймера.

После создания контрольной точки функцию оркестратора можно удалить из памяти, пока для нее не появится работа.

> [!NOTE]
> Служба хранилища Azure не предоставляет никаких гарантий выполнения транзакций между сохранением данных в хранилище таблиц и очередях. Для обработки ошибок поставщик хранилища устойчивых функций использует шаблоны *итоговой согласованности*. Эти шаблоны позволяют избежать потери данных в случае аварийного завершения или потери возможности подключения в середине контрольной точки.

По завершении приведенный выше журнал функций будет выглядеть примерно как в следующей таблице в хранилище таблиц Azure (следующий пример приведен в сокращенном виде):

| PartitionKey (InstanceId)                     | EventType             | Отметка времени               | Входные данные | Имя             | Результат                                                    | Состояние |
|----------------------------------|-----------------------|----------|--------------------------|-------|------------------|-----------------------------------------------------------|
| eaee885b | ExecutionStarted      | 2017-05-05T18:45:28.852Z | null  | E1_HelloSequence |                                                           |                     |
| eaee885b | OrchestratorStarted   | 2017-05-05T18:45:32.362Z |       |                  |                                                           |                     |
| eaee885b | TaskScheduled         | 2017-05-05T18:45:32.670Z |       | E1_SayHello      |                                                           |                     |
| eaee885b | OrchestratorCompleted | 2017-05-05T18:45:32.670Z |       |                  |                                                           |                     |
| eaee885b | TaskCompleted         | 2017-05-05T18:45:34.201Z |       |                  | """Hello Tokyo!"""                                        |                     |
| eaee885b | OrchestratorStarted   | 2017-05-05T18:45:34.232Z |       |                  |                                                           |                     |
| eaee885b | TaskScheduled         | 2017-05-05T18:45:34.435Z |       | E1_SayHello      |                                                           |                     |
| eaee885b | OrchestratorCompleted | 2017-05-05T18:45:34.435Z |       |                  |                                                           |                     |
| eaee885b | TaskCompleted         | 2017-05-05T18:45:34.763Z |       |                  | """Hello Seattle!"""                                      |                     |
| eaee885b | OrchestratorStarted   | 2017-05-05T18:45:34.857Z |       |                  |                                                           |                     |
| eaee885b | TaskScheduled         | 2017-05-05T18:45:34.857Z |       | E1_SayHello      |                                                           |                     |
| eaee885b | OrchestratorCompleted | 2017-05-05T18:45:34.857Z |       |                  |                                                           |                     |
| eaee885b | TaskCompleted         | 2017-05-05T18:45:34.919Z |       |                  | """Hello London!"""                                       |                     |
| eaee885b | OrchestratorStarted   | 2017-05-05T18:45:35.032Z |       |                  |                                                           |                     |
| eaee885b | OrchestratorCompleted | 2017-05-05T18:45:35.044Z |       |                  |                                                           |                     |
| eaee885b | ExecutionCompleted    | 2017-05-05T18:45:35.044Z |       |                  | "[""Hello Tokyo!"",""Hello Seattle!"",""Hello London!""]" | Завершено           |

Некоторые сведения о значениях столбцов:

* **PartitionKey.** Содержит идентификатор экземпляра оркестрации.
* **EventType.** Предоставляет тип события. Принимается один из следующих типов:
  * **OrchestrationStarted.** Функция оркестратора, которая возобновлена после состояния ожидания или выполняется впервые. Столбец `Timestamp` используется, чтобы заполнить детерминированное значение для интерфейсов API `CurrentUtcDateTime` (.NET), `currentUtcDateTime` (JavaScript) и `current_utc_datetime` (Python).
  * **ExecutionStarted.** Функция оркестратора, которая начала выполнение впервые. Это событие также содержит входные данные функции в столбце `Input`.
  * **TaskScheduled.** Функция действия была запланирована. Имя функции действия сохраняется в столбце `Name`.
  * **TaskCompleted.** Функция действия выполнена. Результаты функции находятся в столбце `Result`.
  * **TimerCreated.** Устойчивый таймер создан. Столбец `FireAt` содержит запланированное время в формате UTC, когда истекает срок действия таймера.
  * **TimerFired.** Устойчивый таймер активирован.
  * **EventRaised.** Внешнее событие отправлено в экземпляр оркестрации. Столбец `Name` содержит имя события, а столбец `Input` — полезные данные события.
  * **OrchestratorCompleted.** Функция оркестратора находится в состоянии ожидания.
  * **ContinueAsNew.** Функция оркестратора выполнена и автоматически перезапущена с новым состоянием. Столбец `Result` содержит значение, которое используется в качестве входных данных в перезапущенном экземпляре.
  * **ExecutionCompleted.** Функция оркестратора выполнена (или завершилась сбоем). Выходные данные функции или сведения об ошибке хранятся в столбце `Result`.
* **Метка времени**: Метка времени события журнала в формате UTC.
* **Name** (Имя). Имя вызванной функции.
* **Входные данные** Входные данные функции в формате JSON.
* **Result.** Выходные данные функции (то есть ее возвращаемое значение).

> [!WARNING]
> Хотя это удобно использовать в качестве средства отладки, не используйте в таблице никакие зависимости. Они могут измениться при развитии расширения устойчивых функций.

Каждый раз, когда функция возобновляется из состояния `await` (C#) или `yield` (JavaScript или Python), платформа устойчивых задач повторно выполняет функцию оркестратора с нуля. При каждом повторном выполнении она учитывает журнал выполнения, чтобы определить, выполнялась ли асинхронная операция.  Если операция выполнялась, платформа немедленно воспроизводит выходные данные этой операции и переходит к следующему объекту с состоянием `await` (C#) или `yield` (JavaScript или Python). Этот процесс продолжается, пока не будет воспроизведен весь журнал. После воспроизведения текущего журнала локальные переменные будут восстановлены до своих прежних значений.

## <a name="features-and-patterns"></a>Возможности и шаблоны

В следующих разделах описываются функции и шаблоны функций оркестратора.

### <a name="sub-orchestrations"></a>Вложенные оркестрации

Функции оркестратора могут вызывать функции действия, а также другие функции оркестратора. Например, вы можете построить создать крупную оркестрацию из библиотеки функций оркестратора. Или можно запустить несколько экземпляров функции оркестратора одновременно.

Дополнительные сведения и примеры см. в статье [Sub-orchestrations in Durable Functions (Azure Functions)](durable-functions-sub-orchestrations.md) (Вложенные оркестрации в устойчивых функциях (Функции Azure)).

### <a name="durable-timers"></a>Устойчивые таймеры

Оркестрации могут планировать *устойчивые таймеры* для реализации задержек или для настройки обработки времени ожидания при асинхронных действиях. Используйте устойчивые таймеры в функциях оркестратора вместо `Thread.Sleep` и `Task.Delay` (C#), `setTimeout()` и `setInterval()` (JavaScript) или `time.sleep()` (Python).

Дополнительные сведения и примеры см. в статье [Timers in Durable Functions (Azure Functions)](durable-functions-timers.md) (Таймеры в устойчивых функциях (Функции Azure)).

### <a name="external-events"></a>Внешние события

Функции оркестратора могут ожидать передачи внешних событий для обновления экземпляра оркестрации. Эта возможность устойчивых функций часто используется для обработки действий человека или других внешних обратных вызовов.

Дополнительные сведения и примеры см. в статье [Handling external events in Durable Functions (Azure Functions)](durable-functions-external-events.md) (Обработка внешних событий в устойчивых функциях (Функции Azure)).

### <a name="error-handling"></a>Обработка ошибок

Функции оркестратора могут использовать функции обработки ошибок языка программирования. Существующие шаблоны, такие как `try`/`catch`, поддерживаются в коде оркестрации.

Функции оркестратора также могут добавлять политики повтора к действиям или функциям вложенных оркестраций, которые они вызывают. Если действие или функция вложенной оркестрации завершается сбоем с исключением, указанная политика повтора может автоматически задерживать и повторять выполнение исключения до указанного числа раз.

> [!NOTE]
> Если в функции оркестратора есть необработанное исключение, экземпляр оркестрации завершится в состоянии `Failed`. Экземпляр оркестрации нельзя повторить после сбоя.

Дополнительные сведения и примеры см. в статье [Handling errors in Durable Functions (Azure Functions)](durable-functions-error-handling.md) (Обработка ошибок в устойчивых функциях (Функции Azure)).

### <a name="critical-sections-durable-functions-2x-currently-net-only"></a>Критические разделы (Устойчивые функции 2.x сейчас поддерживается только .NET)

Экземпляры оркестрации являются однопотоковыми, поэтому нет необходимости беспокоиться о состоянии гонки *внутри* оркестрации. Однако условия гонки возможны, когда оркестрации взаимодействуют с внешними системами. Чтобы устранить состояние гонки при взаимодействии с внешними системами, функции оркестратора могут определять *критические секции* с помощью метода `LockAsync` в .NET.

В следующем примере кода показана функция оркестратора, определяющая критическую секцию. Она входит в критическую секцию, используя метод `LockAsync`. Этот метод требует передачи одной или нескольких ссылок на [устойчивую сущность](durable-functions-entities.md), которая длительно управляет состоянием блокировки. Только один экземпляр этой оркестрации может одновременно выполнять код в критической секции.

```csharp
[FunctionName("Synchronize")]
public static async Task Synchronize(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    var lockId = new EntityId("LockEntity", "MyLockIdentifier");
    using (await context.LockAsync(lockId))
    {
        // critical section - only one orchestration can enter at a time
    }
}
```

Объект `LockAsync` приобретает устойчивые замки и возвращает объект `IDisposable`, который завершает критическую секцию при удалении. Этот результат `IDisposable` можно использовать вместе с блоком `using`, чтобы получить синтаксическое представление критической секции. Когда функция оркестратора входит в критическую секцию, только один экземпляр может выполнить этот блок кода. Любые другие экземпляры, которые пытаются войти в критическую секцию, будут заблокированы, пока предыдущий экземпляр не выйдет из критической секции.

Функция критической секции также полезна для согласования изменений в устойчивых сущностях. Дополнительные сведения о критических секциях см. в статье [Entity functions (preview)](durable-functions-entities.md#entity-coordination) (Функции сущностей (предварительная версия)).

> [!NOTE]
> Критические секции доступны в Устойчивых функциях версии 2.0 и выше. В настоящее время только оркестрации .NET реализуют эту функцию.

### <a name="calling-http-endpoints-durable-functions-2x"></a>Вызов конечных точек HTTP (Устойчивые функции 2.x)

Функциям оркестратора не разрешено выполнять операции ввода-вывода, как описано в [ограничениях кода функции оркестратора](durable-functions-code-constraints.md). Типичным обходным решением для этого ограничения является перенос любого кода, который должен выполнять операции ввода-вывода, в функцию действия. Оркестрации, которые взаимодействуют с внешними системами, часто используют функции действий для выполнения вызовов HTTP и возврата результата в оркестрацию.

# <a name="c"></a>[C#](#tab/csharp)

Чтобы упростить этот распространенный шаблон, функции оркестратора могут использовать метод `CallHttpAsync` для вызова API-интерфейсов HTTP напрямую.

```csharp
[FunctionName("CheckSiteAvailable")]
public static async Task CheckSiteAvailable(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    Uri url = context.GetInput<Uri>();

    // Makes an HTTP GET request to the specified endpoint
    DurableHttpResponse response = 
        await context.CallHttpAsync(HttpMethod.Get, url);

    if (response.StatusCode >= 400)
    {
        // handling of error codes goes here
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const url = context.df.getInput();
    var res = yield context.df.callHttp("GET", url);
    if (res.statusCode >= 400) {
        // handling of error codes goes here
    }
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    url = context.get_input()
    res = yield context.call_http('GET', url)
    if res.status_code >= 400:
        # handing of error code goes here
```
---

Помимо поддержки базовых шаблонов запросов/ответов, метод поддерживает автоматическую обработку общих асинхронных шаблонов опроса HTTP 202, а также проверку подлинности с помощью внешних служб с использованием [Управляемых удостоверений](../../active-directory/managed-identities-azure-resources/overview.md).

Дополнительные сведения и подробные примеры см. в статье [HTTP Features](durable-functions-http-features.md) (Функции HTTP).

> [!NOTE]
> Вызов конечных точек HTTP напрямую из функций оркестратора доступен в Устойчивых функциях версии 2.0 и выше.

### <a name="passing-multiple-parameters"></a>Передача нескольких параметров

Передать несколько параметров непосредственно в функцию действия нельзя. Мы рекомендуем передать массив объектов или составные объектов.

# <a name="c"></a>[C#](#tab/csharp)

В .NET также можно использовать объекты [ValueTuple](/dotnet/csharp/tuples). В следующем примере используются новые функции [ValueTuple](/dotnet/csharp/tuples), добавленные в [C# 7](/dotnet/csharp/whats-new/csharp-7#tuples):

```csharp
[FunctionName("GetCourseRecommendations")]
public static async Task<object> RunOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    string major = "ComputerScience";
    int universityYear = context.GetInput<int>();

    object courseRecommendations = await context.CallActivityAsync<object>(
        "CourseRecommendations",
        (major, universityYear));
    return courseRecommendations;
}

[FunctionName("CourseRecommendations")]
public static async Task<object> Mapper([ActivityTrigger] IDurableActivityContext inputs)
{
    // parse input for student's major and year in university
    (string Major, int UniversityYear) studentInfo = inputs.GetInput<(string, int)>();

    // retrieve and return course recommendations by major and university year
    return new
    {
        major = studentInfo.Major,
        universityYear = studentInfo.UniversityYear,
        recommendedCourses = new []
        {
            "Introduction to .NET Programming",
            "Introduction to Linux",
            "Becoming an Entrepreneur"
        }
    };
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

#### <a name="orchestrator"></a>Оркестратор:

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const location = {
        city: "Seattle",
        state: "WA"
    };
    const weather = yield context.df.callActivity("GetWeather", location);

    // ...
};
```

#### <a name="getweather-activity"></a>`GetWeather` Действие

```javascript
module.exports = async function (context, location) {
    const {city, state} = location; // destructure properties into variables

    // ...
};
```

# <a name="python"></a>[Python](#tab/python)

#### <a name="orchestrator"></a>Оркестратор:

```python
from collections import namedtuple
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    Location = namedtuple('Location', ['city', 'state'])
    location = Location(city='Seattle', state= 'WA')

    weather = yield context.call_activity("GetWeather", location)

    # ...

```
#### <a name="getweather-activity"></a>`GetWeather` Действие

```python
from collections import namedtuple

Location = namedtuple('Location', ['city', 'state'])

def main(location: Location) -> str:
    city, state = location
    return f"Hello {city}, {state}!"
```

---

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Ограничения кода оркестратора](durable-functions-code-constraints.md)
