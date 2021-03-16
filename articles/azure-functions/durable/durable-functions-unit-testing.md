---
title: Модульное тестирование устойчивых функций Azure
description: Информация о модульном тестировании устойчивых функций.
ms.topic: conceptual
ms.date: 11/03/2019
ms.openlocfilehash: 89b6419e95b3971b0d272490e19354f300204e1e
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103491050"
---
# <a name="durable-functions-unit-testing"></a>Модульное тестирование устойчивых функций

Модульное тестирование является важной частью современных способов разработки программного обеспечения. Модульные тесты позволяют проверить поведение бизнес-логики и предотвратить внедрение незамеченных критических изменений в будущем. Степень сложности устойчивых функций может быстро увеличиться, поэтому использование модульных тестов поможет предотвращать внесение критических изменений. В следующих разделах объясняется, как выполнить модульное тестирование трех типов функций: клиент оркестрации, Orchestrator и функции действий.

> [!NOTE]
> В этой статье приводятся рекомендации по модульному тестированию для Устойчивые функции приложений, предназначенных для Устойчивые функции 2. x. Дополнительные сведения о различиях между версиями см. в статье [устойчивые функции версии](durable-functions-versions.md) .

## <a name="prerequisites"></a>Предварительные требования

Для выполнения примеров в этой статье нужно ознакомиться со следующими понятиями и платформами.

* Модульное тестирование

* Устойчивые функции

* [xUnit](https://github.com/xunit/xunit) — платформа тестирования.

* [moq](https://github.com/moq/moq4) — платформа имитированной реализации.

## <a name="base-classes-for-mocking"></a>Базовые классы для имитации

Макетирование поддерживается через следующий интерфейс:

* [Идураблеорчестратионклиент](/dotnet/api/microsoft.azure.webjobs.IDurableOrchestrationClient), [идураблинтитиклиент](/dotnet/api/microsoft.azure.webjobs.IDurableEntityClient) и [идураблеклиент](/dotnet/api/microsoft.azure.webjobs.IDurableClient)

* [идураблеорчестратионконтекст](/dotnet/api/microsoft.azure.webjobs.IDurableOrchestrationContext)

* [идураблеактивитиконтекст](/dotnet/api/microsoft.azure.webjobs.IDurableActivityContext)
  
* [идураблинтитиконтекст](/dotnet/api/microsoft.azure.webjobs.IDurableEntityContext)

Эти интерфейсы можно использовать с различными триггерами и привязками, поддерживаемыми Устойчивые функции. При выполнении функций Azure среда выполнения функций будет выполнять код функции с конкретной реализацией этих интерфейсов. Для модульного тестирования можно передать макетную версию этих интерфейсов для тестирования бизнес-логики.

## <a name="unit-testing-trigger-functions"></a>Функции триггеров в модульном тестировании

В этом разделе модульный тест проверяет логику функции триггера HTTP, запускающей новые оркестрации.

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/HttpStart.cs)]

Модульному тесту необходимо проверить заголовок `Retry-After` в полезных данных ответа. Поэтому модульный тест будет макетирование некоторые `IDurableClient` методы, чтобы обеспечить предсказуемое поведение.

Во-первых, в нашем примере используется инфраструктура макетирования ([MOQ](https://github.com/moq/moq4) ) для макетирования `IDurableClient` :

```csharp
// Mock IDurableClient
var durableClientMock = new Mock<IDurableClient>();
```

> [!NOTE]
> Хотя вы можете макетирование интерфейсы, напрямую реализовав интерфейс как класс, макеты инфраструктуры упрощают процесс различными способами. Например, если новый метод добавляется в интерфейс во вспомогательных выпусках, MOQ не потребует каких-либо изменений кода, в отличие от конкретных реализаций.

Затем метод `StartNewAsync` имитируется для возвращения идентификатора известного экземпляра.

```csharp
// Mock StartNewAsync method
durableClientMock.
    Setup(x => x.StartNewAsync(functionName, It.IsAny<object>())).
    ReturnsAsync(instanceId);
```

Кроме того, `CreateCheckStatusResponse` имитируется таким образом, чтобы всегда возвращать пустой ответ HTTP 200.

```csharp
// Mock CreateCheckStatusResponse method
durableClientMock
    // Notice that even though the HttpStart function does not call IDurableClient.CreateCheckStatusResponse() 
    // with the optional parameter returnInternalServerErrorOnFailure, moq requires the method to be set up
    // with each of the optional parameters provided. Simply use It.IsAny<> for each optional parameter
    .Setup(x => x.CreateCheckStatusResponse(It.IsAny<HttpRequestMessage>(), instanceId, returnInternalServerErrorOnFailure: It.IsAny<bool>())
    .Returns(new HttpResponseMessage
    {
        StatusCode = HttpStatusCode.OK,
        Content = new StringContent(string.Empty),
        Headers =
        {
            RetryAfter = new RetryConditionHeaderValue(TimeSpan.FromSeconds(10))
        }
    });
```

`ILogger` также имитируется.

```csharp
// Mock ILogger
var loggerMock = new Mock<ILogger>();
```  

Теперь `Run` метод вызывается из модульного теста.

```csharp
// Call Orchestration trigger function
var result = await HttpStart.Run(
    new HttpRequestMessage()
    {
        Content = new StringContent("{}", Encoding.UTF8, "application/json"),
        RequestUri = new Uri("http://localhost:7071/orchestrators/E1_HelloSequence"),
    },
    durableClientMock.Object,
    functionName,
    loggerMock.Object);
 ```

 На последнем шаге происходит сравнение вывода с ожидаемым значением.

```csharp
// Validate that output is not null
Assert.NotNull(result.Headers.RetryAfter);

// Validate output's Retry-After header value
Assert.Equal(TimeSpan.FromSeconds(10), result.Headers.RetryAfter.Delta);
```

После объединения всех шагов код модульного теста будет выглядеть следующим образом.

[!code-csharp[Main](~/samples-durable-functions/samples/VSSample.Tests/HttpStartTests.cs)]

## <a name="unit-testing-orchestrator-functions"></a>Функции оркестратора для модульного тестирования

Функции оркестратора при модульном тестировании даже более интересны, так как в них обычно гораздо больше бизнес-логики.

В этом разделе модульные тесты проверяют выходные данные функции оркестратора `E1_HelloSequence`.

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/HelloSequence.cs)]

Код модульного теста начинается со строки для создания имитации.

```csharp
var durableOrchestrationContextMock = new Mock<IDurableOrchestrationContext>();
```

Затем имитируются вызовы метода действий.

```csharp
durableOrchestrationContextMock.Setup(x => x.CallActivityAsync<string>("E1_SayHello", "Tokyo")).ReturnsAsync("Hello Tokyo!");
durableOrchestrationContextMock.Setup(x => x.CallActivityAsync<string>("E1_SayHello", "Seattle")).ReturnsAsync("Hello Seattle!");
durableOrchestrationContextMock.Setup(x => x.CallActivityAsync<string>("E1_SayHello", "London")).ReturnsAsync("Hello London!");
```

Далее модульный тест вызывает метод `HelloSequence.Run`.

```csharp
var result = await HelloSequence.Run(durableOrchestrationContextMock.Object);
```

И, наконец, проверяются выходные данные.

```csharp
Assert.Equal(3, result.Count);
Assert.Equal("Hello Tokyo!", result[0]);
Assert.Equal("Hello Seattle!", result[1]);
Assert.Equal("Hello London!", result[2]);
```

После объединения всех шагов код модульного теста будет выглядеть следующим образом.

[!code-csharp[Main](~/samples-durable-functions/samples/VSSample.Tests/HelloSequenceOrchestratorTests.cs)]

## <a name="unit-testing-activity-functions"></a>Функции действий модульного теста

Функции действий можно подвергать модульному тестированию таким же образом, как неустойчивые функции.

В этом разделе модульный тест проверяет поведение функции действия `E1_SayHello`.

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/HelloSequence.cs)]

Кроме того, модульный тест позволяет проверить формат выходных данных. Модульные тесты могут использовать типы параметров напрямую или макетирование `IDurableActivityContext` класса:

[!code-csharp[Main](~/samples-durable-functions/samples/VSSample.Tests/HelloSequenceActivityTests.cs)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Дополнительные сведения о xUnit](https://xunit.net/docs/getting-started/netcore/cmdline)
> 
> [Дополнительные сведения о moq](https://github.com/Moq/moq4/wiki/Quickstart)
