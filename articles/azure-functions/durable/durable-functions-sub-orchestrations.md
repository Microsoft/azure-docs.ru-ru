---
title: Вложенные оркестрации в устойчивых функциях — Azure
description: Узнайте, как вызывать оркестрации из оркестраций в расширении устойчивых функций для Функций Azure.
ms.topic: conceptual
ms.date: 11/03/2019
ms.author: azfuncdf
ms.openlocfilehash: 5625bc2ddfa4b6f527ca16f19f33d257a1834d4b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85340819"
---
# <a name="sub-orchestrations-in-durable-functions-azure-functions"></a>Вложенные оркестрации в устойчивых функциях (Функции Azure)

Кроме вызова функций действия, функции оркестратора могут вызывать другие функции оркестратора. Например, можно создать более крупное согласование из библиотеки небольших функций Orchestrator. Или можно запустить несколько экземпляров функции оркестратора одновременно.

Функция Orchestrator может вызывать другую функцию Orchestrator `CallSubOrchestratorAsync` , используя методы или в `CallSubOrchestratorWithRetryAsync` .NET или `callSubOrchestrator` `callSubOrchestratorWithRetry` методы или в JavaScript. Дополнительные сведения об автоматических повторных попытках см. в руководстве по [обработке ошибок и компенсации](durable-functions-error-handling.md#automatic-retry-on-failure).

Для вызывающего функции вложенного оркестратора и функции действия ведут себя одинаково. Они возвращают значение, выдают исключение и могут ожидаться родительской функцией оркестратора. 

> [!NOTE]
> В настоящее время подсистемы согласования поддерживаются в .NET и JavaScript.

## <a name="example"></a>Пример

В следующем примере показан сценарий использования Интернета вещей с несколькими устройствами, которые нужно подготовить. Следующая функция представляет рабочий процесс подготовки, который должен выполняться для каждого устройства:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
public static async Task DeviceProvisioningOrchestration(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    string deviceId = context.GetInput<string>();

    // Step 1: Create an installation package in blob storage and return a SAS URL.
    Uri sasUrl = await context.CallActivityAsync<Uri>("CreateInstallationPackage", deviceId);

    // Step 2: Notify the device that the installation package is ready.
    await context.CallActivityAsync("SendPackageUrlToDevice", Tuple.Create(deviceId, sasUrl));

    // Step 3: Wait for the device to acknowledge that it has downloaded the new package.
    await context.WaitForExternalEvent<bool>("DownloadCompletedAck");

    // Step 4: ...
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const deviceId = context.df.getInput();

    // Step 1: Create an installation package in blob storage and return a SAS URL.
    const sasUrl = yield context.df.callActivity("CreateInstallationPackage", deviceId);

    // Step 2: Notify the device that the installation package is ready.
    yield context.df.callActivity("SendPackageUrlToDevice", { id: deviceId, url: sasUrl });

    // Step 3: Wait for the device to acknowledge that it has downloaded the new package.
    yield context.df.waitForExternalEvent("DownloadCompletedAck");

    // Step 4: ...
});
```

---

Эту функцию оркестратора можно использовать как есть — для однократной подготовки устройства — или же как часть более крупной оркестрации. В последнем случае родительская функция Orchestrator может запланировать экземпляры `DeviceProvisioningOrchestration` с помощью `CallSubOrchestratorAsync` API (.NET) или `callSubOrchestrator` (JavaScript).

В примере ниже показано, как реализовать параллельное выполнение нескольких функций оркестратора.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("ProvisionNewDevices")]
public static async Task ProvisionNewDevices(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    string[] deviceIds = await context.CallActivityAsync<string[]>("GetNewDeviceIds");

    // Run multiple device provisioning flows in parallel
    var provisioningTasks = new List<Task>();
    foreach (string deviceId in deviceIds)
    {
        Task provisionTask = context.CallSubOrchestratorAsync("DeviceProvisioningOrchestration", deviceId);
        provisioningTasks.Add(provisionTask);
    }

    await Task.WhenAll(provisioningTasks);

    // ...
}
```

> [!NOTE]
> Предыдущие примеры C# предназначены для Устойчивые функции 2. x. Для Устойчивые функции 1. x необходимо использовать `DurableOrchestrationContext` вместо `IDurableOrchestrationContext` . Дополнительные сведения о различиях между версиями см. в статье [устойчивые функции версии](durable-functions-versions.md) .

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const deviceIds = yield context.df.callActivity("GetNewDeviceIds");

    // Run multiple device provisioning flows in parallel
    const provisioningTasks = [];
    var id = 0;
    for (const deviceId of deviceIds) {
        const child_id = context.df.instanceId+`:${id}`;
        const provisionTask = context.df.callSubOrchestrator("DeviceProvisioningOrchestration", deviceId, child_id);
        provisioningTasks.push(provisionTask);
        id++;
    }

    yield context.df.Task.all(provisioningTasks);

    // ...
});
```

---

> [!NOTE]
> Подсогласования должны быть определены в том же приложении-функции, что и родительское согласование. Если необходимо вызвать и подождать согласования в другом приложении-функции, рассмотрите возможность использования встроенной поддержки API HTTP и шаблона потребителя опроса HTTP 202. Дополнительные сведения см. в разделе [функции HTTP](durable-functions-http-features.md) .

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Узнайте, как задать пользовательское состояние оркестрации](durable-functions-custom-orchestration-status.md)
