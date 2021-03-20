---
title: Запуск и завершение узлов кластера
description: Узнайте, как использовать внесение ошибок для тестирования приложения Service Fabric, запуская и останавливая узлы кластера.
author: LMWF
ms.topic: conceptual
ms.date: 6/12/2017
ms.author: lemai
ms.custom: devx-track-csharp
ms.openlocfilehash: 9c31040ec13084f9e4b08bbc9a347e4ad44975bf
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89021261"
---
# <a name="replacing-the-start-node-and-stop-node-apis-with-the-node-transition-api"></a>Замена API-интерфейсов запуска и остановки узла API-интерфейсом перехода узла

## <a name="what-do-the-stop-node-and-start-node-apis-do"></a>Для чего нужны API-интерфейсы запуска и остановки узла?

API остановки узла (управляемый: [StopNodeAsync()][stopnode], PowerShell: [Stop-ServiceFabricNode][stopnodeps]) останавливает узел Service Fabric.  Узел Service Fabric — это процесс, а не компьютер или виртуальная машина, поэтому компьютер или виртуальная машина будут продолжать работу.  В этом документе далее термин "узел" означает узел Service Fabric.  При остановке узел помещается в состояние *Остановлен*, при котором он не является элементом кластера и не может размещать службы, имитируя *отключенный* узел.  Это полезно для внесения ошибок в систему при тестировании приложения.  API запуска узла (управляемый: [StartNodeAsync()][startnode], PowerShell: [Start-ServiceFabricNode][startnodeps]]) обращает API остановки узла, возвращая узел в обычное состояние.

## <a name="why-are-we-replacing-these"></a>Зачем их заменять?

Как было описано ранее, *остановленный* узел Service Fabric — это узел, к которому намеренно применен API остановки узла.  *Отключенный* узел — это узел, который не работает по какой-либо другой причине (например, из-за отключения компьютера или виртуальной машины).  При использовании API остановки узла в системе не отображаются сведения, с помощью которых можно различить *остановленные* и *отключенные* узлы.

Кроме того, некоторые сообщения об ошибках, возвращаемые этими API-интерфейсами, недостаточно содержательны.  Например, при вызове API остановки узла на уже *остановленном* узле возвращается ошибка *InvalidAddress*.  Это поведение можно оптимизировать.

Кроме того, работа узла останавливается на неограниченное время до вызова API запуска узла.  Выяснилось, что это может привести к проблемам и возникновению ошибок.  Например, мы сталкивались с проблемами, когда пользователь вызывал API остановки узла, а потом забывал об этом.  Позже было невозможно установить, что произошло с узлом: был ли он *отключен* или *остановлен*.


## <a name="introducing-the-node-transition-apis"></a>Знакомство с API-интерфейсами перехода узла

Описанные выше проблемы можно решить с помощью нашего нового набора API-интерфейсов.  Новый API перехода узла (управляемый: [StartNodeTransitionAsync()][snt]) может использоваться для перехода узла Service Fabric в *остановленное* состояние или для перехода из *остановленного* состояния в нормальное рабочее состояние.  Обратите внимание на то, что "запуск" в названии API не означает собственно запуск узла.  Это означает начало асинхронной операции, которую выполняет система для перевода узла в *остановленное* или рабочее состояние.

**Использование**

Если при вызове API перехода узла не возвращается исключение, это означает, что система приняла асинхронную операцию и собирается ее выполнить.  Успешный вызов еще не означает, что операция завершена.  Для получения сведений о текущем состоянии операции вызовите API хода выполнения перехода узла (управляемый: [GetNodeTransitionProgressAsync()][gntp]) с помощью идентификатора GUID, используемого при вызове API перехода узла для этой операции.  API хода выполнения перехода узла возвращает объект NodeTransitionProgress.  Это свойство State объекта указывает текущее состояние операции.  Если свойство имеет значение Running, операция выполняется.  Значение Completed указывает на то, что операция завершена без ошибок.  А значение Faulted — на ошибку, возникшую при выполнении операции.  Свойство Exception в свойстве Result указывает на наличие проблемы.  Дополнительные сведения о свойстве State доступны в статье https://docs.microsoft.com/dotnet/api/system.fabric.testcommandprogressstate. Ознакомьтесь также с примерами кода в разделе "Пример использования" ниже.


**Определение остановленного и отключенного узла.** Если узел *остановлен* с помощью API перехода узла, в выходных данных запроса узла (управляемый: [GetNodeListAsync()][nodequery], PowerShell: [Get-ServiceFabricNode][nodequeryps]) свойство *IsStopped* этого узла будет иметь значение true.  Обратите внимание, что оно отличается от значения свойства *NodeStatus*, которое равно *Down*.  Если свойство *NodeStatus* имеет значение *Down*, но свойство *IsStopped* имеет значение false, это значит, что узел не был остановлен с помощью API перехода узла, а *отключен* по какой-то другой причине.  Если свойство *IsStopped* имеет значение true, а свойство *NodeStatus* — значение *Down*, это значит, что узел остановлен с помощью API перехода узла.

Запуск *остановленного* узла с помощью API перехода узла вернет его к работе в качестве обычного элемента кластера.  В выходных данных API запроса узла для свойства *IsStopped* будет отображаться значение false, а для свойства *NodeStatus* — значение, отличное от Down (например, Up).


**Ограниченное время.** При использовании API перехода узла для остановки узла один из обязательных параметров, *stopNodeDurationInSeconds*, отображает время в секундах, в течение которого узел будет оставаться *остановленным*.  Это должно быть значение в разрешенном диапазоне — не менее 600 и не более 14 400.  По истечении этого времени узел автоматически перезапустится и перейдет в рабочее состояние (Up).  Использование этого API показано в примере 1 ниже.

> [!WARNING]
> Не используйте одновременно API-интерфейсы перехода узла и API-интерфейсы запуска и остановки узла.  Рекомендуем использовать только API перехода узла.  Если узел уже остановлен с помощью API остановки узла, его следует запустить с помощью API запуска узла и только потом использовать API перехода узла.

> [!WARNING]
> На одном узле нельзя параллельно выполнить несколько вызовов API перехода узла.  В таком случае API перехода узла выдаст исключение FabricException со значением свойства ErrorCode, равным NodeTransitionInProgress.  После начала перехода на определенном узле следует подождать, пока операция достигнет конечного состояния (Completed, Faulted или ForceCancelled), прежде чем начинать новый переход на том же узле.  Параллельные вызовы перехода на разных узлах разрешены.


#### <a name="sample-usage"></a>Пример использования


**Пример 1.** В этом примере API перехода узла используется для остановки узла.

```csharp
        // Helper function to get information about a node
        static Node GetNodeInfo(FabricClient fc, string node)
        {
            NodeList n = null;
            while (n == null)
            {
                n = fc.QueryManager.GetNodeListAsync(node).GetAwaiter().GetResult();
                Task.Delay(TimeSpan.FromSeconds(1)).GetAwaiter();
            };

            return n.FirstOrDefault();
        }

        static async Task WaitForStateAsync(FabricClient fc, Guid operationId, TestCommandProgressState targetState)
        {
            NodeTransitionProgress progress = null;

            do
            {
                bool exceptionObserved = false;
                try
                {
                    progress = await fc.TestManager.GetNodeTransitionProgressAsync(operationId, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
                }
                catch (OperationCanceledException oce)
                {
                    Console.WriteLine("Caught exception '{0}'", oce);
                    exceptionObserved = true;
                }
                catch (FabricTransientException fte)
                {
                    Console.WriteLine("Caught exception '{0}'", fte);
                    exceptionObserved = true;
                }

                if (!exceptionObserved)
                {
                    Console.WriteLine("Current state of operation '{0}': {1}", operationId, progress.State);

                    if (progress.State == TestCommandProgressState.Faulted)
                    {
                        // Inspect the progress object's Result.Exception.HResult to get the error code.
                        Console.WriteLine("'{0}' failed with: {1}, HResult: {2}", operationId, progress.Result.Exception, progress.Result.Exception.HResult);

                        // ...additional logic as required
                    }

                    if (progress.State == targetState)
                    {
                        Console.WriteLine("Target state '{0}' has been reached", targetState);
                        break;
                    }
                }

                await Task.Delay(TimeSpan.FromSeconds(5)).ConfigureAwait(false);
            }
            while (true);
        }

        static async Task StopNodeAsync(FabricClient fc, string nodeName, int durationInSeconds)
        {
            // Uses the GetNodeListAsync() API to get information about the target node
            Node n = GetNodeInfo(fc, nodeName);

            // Create a Guid
            Guid guid = Guid.NewGuid();

            // Create a NodeStopDescription object, which will be used as a parameter into StartNodeTransition
            NodeStopDescription description = new NodeStopDescription(guid, n.NodeName, n.NodeInstanceId, durationInSeconds);

            bool wasSuccessful = false;

            do
            {
                try
                {
                    // Invoke StartNodeTransitionAsync with the NodeStopDescription from above, which will stop the target node.  Retry transient errors.
                    await fc.TestManager.StartNodeTransitionAsync(description, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
                    wasSuccessful = true;
                }
                catch (OperationCanceledException oce)
                {
                    // This is retryable
                }
                catch (FabricTransientException fte)
                {
                    // This is retryable
                }

                // Backoff
                await Task.Delay(TimeSpan.FromSeconds(5)).ConfigureAwait(false);
            }
            while (!wasSuccessful);

            // Now call StartNodeTransitionProgressAsync() until the desired state is reached.
            await WaitForStateAsync(fc, guid, TestCommandProgressState.Completed).ConfigureAwait(false);
        }
```

**Пример 2.** В этом примере запускается *остановленный* узел.  Здесь используются некоторые вспомогательные методы из первого примера.

```csharp
        static async Task StartNodeAsync(FabricClient fc, string nodeName)
        {
            // Uses the GetNodeListAsync() API to get information about the target node
            Node n = GetNodeInfo(fc, nodeName);

            Guid guid = Guid.NewGuid();
            BigInteger nodeInstanceId = n.NodeInstanceId;

            // Create a NodeStartDescription object, which will be used as a parameter into StartNodeTransition
            NodeStartDescription description = new NodeStartDescription(guid, n.NodeName, nodeInstanceId);

            bool wasSuccessful = false;

            do
            {
                try
                {
                    // Invoke StartNodeTransitionAsync with the NodeStartDescription from above, which will start the target stopped node.  Retry transient errors.
                    await fc.TestManager.StartNodeTransitionAsync(description, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
                    wasSuccessful = true;
                }
                catch (OperationCanceledException oce)
                {
                    Console.WriteLine("Caught exception '{0}'", oce);
                }
                catch (FabricTransientException fte)
                {
                    Console.WriteLine("Caught exception '{0}'", fte);
                }

                await Task.Delay(TimeSpan.FromSeconds(5)).ConfigureAwait(false);

            }
            while (!wasSuccessful);

            // Now call StartNodeTransitionProgressAsync() until the desired state is reached.
            await WaitForStateAsync(fc, guid, TestCommandProgressState.Completed).ConfigureAwait(false);
        }
```

**Пример 3.** В этом примере показано неправильное использование.  Ошибка заключается в том, что для свойства *stopDurationInSeconds* указано значение за пределами допустимого диапазона.  Так как API StartNodeTransitionAsync() прекращает работу с неустранимой ошибкой, операция отклоняется, и API хода выполнения невозможно вызвать.  В этом примере используются некоторые вспомогательные методы из первого примера.

```csharp
        static async Task StopNodeWithOutOfRangeDurationAsync(FabricClient fc, string nodeName)
        {
            Node n = GetNodeInfo(fc, nodeName);

            Guid guid = Guid.NewGuid();

            // Use an out of range value for stopDurationInSeconds to demonstrate error
            NodeStopDescription description = new NodeStopDescription(guid, n.NodeName, n.NodeInstanceId, 99999);

            try
            {
                await fc.TestManager.StartNodeTransitionAsync(description, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
            }

            catch (FabricException e)
            {
                Console.WriteLine("Caught {0}", e);
                Console.WriteLine("ErrorCode {0}", e.ErrorCode);
                // Output:
                // Caught System.Fabric.FabricException: System.Runtime.InteropServices.COMException (-2147017629)
                // StopDurationInSeconds is out of range ---> System.Runtime.InteropServices.COMException: Exception from HRESULT: 0x80071C63
                // << Parts of exception omitted>>
                //
                // ErrorCode InvalidDuration
            }
        }
```

**Пример 4.** В этом примере показаны сведения об ошибке, возвращаемые из API хода выполнения перехода узла, когда операция, запущенная с помощью API перехода узла, принимается, но затем при ее выполнении происходит сбой.  В этом случае сбой происходит из-за того, что API перехода узла пытается запустить несуществующий узел.  В этом примере используются некоторые вспомогательные методы из первого примера.

```csharp
        static async Task StartNodeWithNonexistentNodeAsync(FabricClient fc)
        {
            Guid guid = Guid.NewGuid();
            BigInteger nodeInstanceId = 12345;

            // Intentionally use a nonexistent node
            NodeStartDescription description = new NodeStartDescription(guid, "NonexistentNode", nodeInstanceId);

            bool wasSuccessful = false;

            do
            {
                try
                {
                    // Invoke StartNodeTransitionAsync with the NodeStartDescription from above, which will start the target stopped node.  Retry transient errors.
                    await fc.TestManager.StartNodeTransitionAsync(description, TimeSpan.FromMinutes(1), CancellationToken.None).ConfigureAwait(false);
                    wasSuccessful = true;
                }
                catch (OperationCanceledException oce)
                {
                    Console.WriteLine("Caught exception '{0}'", oce);
                }
                catch (FabricTransientException fte)
                {
                    Console.WriteLine("Caught exception '{0}'", fte);
                }

                await Task.Delay(TimeSpan.FromSeconds(5)).ConfigureAwait(false);

            }
            while (!wasSuccessful);

            // Now call StartNodeTransitionProgressAsync() until the desired state is reached.  In this case, it will end up in the Faulted state since the node does not exist.
            // When StartNodeTransitionProgressAsync()'s returned progress object has a State if Faulted, inspect the progress object's Result.Exception.HResult to get the error code.
            // In this case, it will be NodeNotFound.
            await WaitForStateAsync(fc, guid, TestCommandProgressState.Faulted).ConfigureAwait(false);
        }
```

[stopnode]: /dotnet/api/system.fabric.fabricclient.faultmanagementclient
[stopnodeps]: /previous-versions/azure/mt125982(v=azure.100)
[startnode]: /dotnet/api/system.fabric.fabricclient.faultmanagementclient
[startnodeps]: /previous-versions/azure/mt163520(v=azure.100)
[nodequery]: /dotnet/api/system.fabric.fabricclient.queryclient
[nodequeryps]: /powershell/module/servicefabric/get-servicefabricnode
[snt]: /dotnet/api/system.fabric.fabricclient.testmanagementclient
[gntp]: /dotnet/api/system.fabric.fabricclient.testmanagementclient
