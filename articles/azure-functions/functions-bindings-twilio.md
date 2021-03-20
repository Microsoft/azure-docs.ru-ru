---
title: Привязка Twilio для службы "Функции Azure"
description: Узнайте, как использовать привязки Twilio с функциями Azure.
author: craigshoemaker
ms.topic: reference
ms.date: 07/09/2018
ms.author: cshoe
ms.custom: devx-track-csharp, H1Hack27Feb2017
ms.openlocfilehash: 31d5c528112ccacab47282729b1b5f8a72f19af0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91999285"
---
# <a name="twilio-binding-for-azure-functions"></a>Привязка Twilio для службы "Функции Azure"

В этой статье описано, как отправлять текстовые сообщения с помощью привязок [Twilio](https://www.twilio.com/) в службу "Функции Azure". Служба "Функции Azure" поддерживают выходные привязки для Twilio.

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

## <a name="packages---functions-1x"></a>Пакеты – Функции 1.x

Привязки Twilio доступны в пакете NuGet [Microsoft.Azure.WebJobs.Extensions.Twilio](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Twilio), версия 1.х. Исходный код для пакета находится в репозитории GitHub [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/v2.x/src/WebJobs.Extensions.Twilio/).

[!INCLUDE [functions-package](../../includes/functions-package.md)]

## <a name="packages---functions-2x-and-higher"></a>Packages — функции 2. x и более поздних версий

Привязки Twilio доступны в пакете NuGet [Microsoft.Azure.WebJobs.Extensions.Twilio](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Twilio), версия 3.х. Исходный код для пакета находится в репозитории GitHub [azure-webjobs-sdk](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.Twilio/).

[!INCLUDE [functions-package-v2](../../includes/functions-package-v2.md)]

<a id="example"></a>

## <a name="example---functions-2x-and-higher"></a>Example-functions 2. x и более поздние версии

# <a name="c"></a>[C#](#tab/csharp)

В следующем примере показана [функция C#](functions-dotnet-class-library.md), которая отправляет текстовое сообщение при активации сообщением из очереди.

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json.Linq;
using Twilio.Rest.Api.V2010.Account;
using Twilio.Types;
namespace TwilioQueueOutput
{
    public static class QueueTwilio
    {
        [FunctionName("QueueTwilio")]
        [return: TwilioSms(AccountSidSetting = "TwilioAccountSid", AuthTokenSetting = "TwilioAuthToken", From = "+1425XXXXXXX")]
        public static CreateMessageOptions Run(
        [QueueTrigger("myqueue-items", Connection = "AzureWebJobsStorage")] JObject order,
        ILogger log)
        {
            log.LogInformation($"C# Queue trigger function processed: {order}");

            var message = new CreateMessageOptions(new PhoneNumber(order["mobileNumber"].ToString()))
            {
                Body = $"Hello {order["name"]}, thanks for your order!"
            };

            return message;
        }
    }
}
```

В этом примере используется `TwilioSms` атрибут с помощью возвращаемого значения метода. Альтернативой является использование атрибута с параметрами `out CreateMessageOptions`, `ICollector<CreateMessageOptions>` или `IAsyncCollector<CreateMessageOptions>`.

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В следующем примере показаны выходная привязка Twilio *function.json* и [функция сценария C#](functions-reference-csharp.md), которая использует эту привязку. Эта функция использует параметр `out` для отправки текстового сообщения.

Данные привязки в файле *function.json*:

Пример файла function.json:

```json
{
  "type": "twilioSms",
  "name": "message",
  "accountSidSetting": "TwilioAccountSid",
  "authTokenSetting": "TwilioAuthToken",
  "from": "+1425XXXXXXX",
  "direction": "out",
  "body": "Azure Functions Testing"
}
```

Ниже приведен код сценария C#:

```cs
#r "Newtonsoft.Json"
#r "Twilio"
#r "Microsoft.Azure.WebJobs.Extensions.Twilio"

using System;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;
using Microsoft.Azure.WebJobs.Extensions.Twilio;
using Twilio.Rest.Api.V2010.Account;
using Twilio.Types;

public static void Run(string myQueueItem, out CreateMessageOptions message,  ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem}");

    // In this example the queue item is a JSON string representing an order that contains the name of a
    // customer and a mobile number to send text updates to.
    dynamic order = JsonConvert.DeserializeObject(myQueueItem);
    string msg = "Hello " + order.name + ", thank you for your order.";

    // You must initialize the CreateMessageOptions variable with the "To" phone number.
    message = new CreateMessageOptions(new PhoneNumber("+1704XXXXXXX"));

    // A dynamic message can be set instead of the body in the output binding. In this example, we use
    // the order information to personalize a text message.
    message.Body = msg;
}
```

Вы не можете использовать параметры вывода в асинхронном коде. Ниже приведен пример асинхронного кода сценария C#:

```cs
#r "Newtonsoft.Json"
#r "Twilio"
#r "Microsoft.Azure.WebJobs.Extensions.Twilio"

using System;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;
using Microsoft.Azure.WebJobs.Extensions.Twilio;
using Twilio.Rest.Api.V2010.Account;
using Twilio.Types;

public static async Task Run(string myQueueItem, IAsyncCollector<CreateMessageOptions> message,  ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem}");

    // In this example the queue item is a JSON string representing an order that contains the name of a
    // customer and a mobile number to send text updates to.
    dynamic order = JsonConvert.DeserializeObject(myQueueItem);
    string msg = "Hello " + order.name + ", thank you for your order.";

    // You must initialize the CreateMessageOptions variable with the "To" phone number.
    CreateMessageOptions smsText = new CreateMessageOptions(new PhoneNumber("+1704XXXXXXX"));

    // A dynamic message can be set instead of the body in the output binding. In this example, we use
    // the order information to personalize a text message.
    smsText.Body = msg;

    await message.AddAsync(smsText);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В следующем примере показана выходная привязка Twilio в файле *function.json* и функции [JavaScript](functions-reference-node.md), которая использует привязку.

Данные привязки в файле *function.json*:

Пример файла function.json:

```json
{
  "type": "twilioSms",
  "name": "message",
  "accountSidSetting": "TwilioAccountSid",
  "authTokenSetting": "TwilioAuthToken",
  "from": "+1425XXXXXXX",
  "direction": "out",
  "body": "Azure Functions Testing"
}
```

Ниже показан код JavaScript.

```javascript
module.exports = function (context, myQueueItem) {
    context.log('Node.js queue trigger function processed work item', myQueueItem);

    // In this example the queue item is a JSON string representing an order that contains the name of a
    // customer and a mobile number to send text updates to.
    var msg = "Hello " + myQueueItem.name + ", thank you for your order.";

    // Even if you want to use a hard coded message in the binding, you must at least
    // initialize the message binding.
    context.bindings.message = {};

    // A dynamic message can be set instead of the body in the output binding. The "To" number 
    // must be specified in code. 
    context.bindings.message = {
        body : msg,
        to : myQueueItem.mobileNumber
    };

    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

В следующем примере показано, как отправить SMS сообщения с помощью выходной привязки, как определено в следующем *function.json*.

```json
    {
      "type": "twilioSms",
      "name": "twilioMessage",
      "accountSidSetting": "TwilioAccountSID",
      "authTokenSetting": "TwilioAuthToken",
      "from": "+1XXXXXXXXXX",
      "direction": "out",
      "body": "Azure Functions Testing"
    }
```

Можно передать сериализованный объект JSON в `func.Out` параметр, чтобы отправить SMS-сообщение.

```python
import logging
import json
import azure.functions as func

def main(req: func.HttpRequest, twilioMessage: func.Out[str]) -> func.HttpResponse:

    message = req.params.get('message')
    to = req.params.get('to')

    value = {
      "body": message,
      "to": to
    }

    twilioMessage.set(json.dumps(value))

    return func.HttpResponse(f"Message sent")
```

# <a name="java"></a>[Java](#tab/java)

В следующем примере показано, как использовать заметку [твилиосмсаутпут](/java/api/com.microsoft.azure.functions.annotation.twiliosmsoutput) для отправки сообщения SMS. Значения для `to` , `from` и `body` являются обязательными в определении атрибута, даже если они переопределяются программным способом.

```java
package com.function;

import java.util.*;
import com.microsoft.azure.functions.annotation.*;
import com.microsoft.azure.functions.*;

public class TwilioOutput {

    @FunctionName("TwilioOutput")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req", methods = { HttpMethod.GET, HttpMethod.POST },
                authLevel = AuthorizationLevel.FUNCTION) HttpRequestMessage<Optional<String>> request,
            @TwilioSmsOutput(
                name = "twilioMessage",
                accountSid = "AzureWebJobsTwilioAccountSID",
                authToken = "AzureWebJobsTwilioAuthToken",
                to = "+1XXXXXXXXXX",
                body = "From Azure Functions",
                from = "+1XXXXXXXXXX") OutputBinding<String> twilioMessage,
            final ExecutionContext context) {

        String message = request.getQueryParameters().get("message");
        String to = request.getQueryParameters().get("to");

        StringBuilder builder = new StringBuilder()
            .append("{")
            .append("\"body\": \"%s\",")
            .append("\"to\": \"%s\"")
            .append("}");

        final String body = String.format(builder.toString(), message, to);

        twilioMessage.setValue(body);

        return request.createResponseBuilder(HttpStatus.OK).body("Message sent").build();
    }
}
```

---

## <a name="attributes-and-annotations"></a>Атрибуты и заметки

# <a name="c"></a>[C#](#tab/csharp)

В [библиотеках классов C#](functions-dotnet-class-library.md) используйте атрибут [TwilioSms](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.Twilio/TwilioSMSAttribute.cs).

Дополнительные сведения о настройке свойств атрибутов см. в разделе [Конфигурация](#configuration). Ниже приведен пример атрибута `TwilioSms` в сигнатуре метода:

```csharp
[FunctionName("QueueTwilio")]
[return: TwilioSms(AccountSidSetting = "TwilioAccountSid", AuthTokenSetting = "TwilioAuthToken", From = "+1425XXXXXXX")]
public static CreateMessageOptions Run(
[QueueTrigger("myqueue-items", Connection = "AzureWebJobsStorage")] JObject order, ILogger log)
{
    ...
}
 ```

Полный пример см. в разделе [пример на C#](#example).

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В скрипте C# атрибуты не поддерживаются.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В JavaScript атрибуты не поддерживаются.

# <a name="python"></a>[Python](#tab/python)

В Python атрибуты не поддерживаются.

# <a name="java"></a>[Java](#tab/java)

Поместите заметку [твилиосмсаутпут](/java/api/com.microsoft.azure.functions.annotation.twiliosmsoutput) к [`OutputBinding<T>`](/java/api/com.microsoft.azure.functions.outputbinding) параметру `T` , где может быть любым собственным типом Java `int` , например,, `String` `byte[]` или типом POJO.

---

## <a name="configuration"></a>Конфигурация

В следующей таблице описываются свойства конфигурации привязки, которые задаются в файле *function.json* и атрибуте `TwilioSms`.

| Свойство function.json версии 1 | Свойство function.json версии 2 | Свойство атрибута |Описание|
|---------|---------|---------|----------------------|
|**type**|**type**| Нужно задать значение `twilioSms`.|
|**direction**|**direction**| Нужно задать значение `out`.|
|**name**|**name**| Имя переменной, используемое в коде функции для текстового SMS-сообщения Twilio. |
|**accountSid**|**аккаунтсидсеттинг**| **AccountSidSetting**| Это значение должно быть равно имени параметра приложения, в котором хранится идентификатор безопасности учетной записи Twilio ( `TwilioAccountSid` ). Если значение не задано, имя параметра приложения по умолчанию — AzureWebJobsTwilioAccountSid. |
|**authToken**|**authTokenSetting**|**аустокенсеттинг**| Это значение должно быть равно имени параметра приложения, содержащего токен проверки подлинности Twilio ( `TwilioAccountAuthToken` ). Если значение не задано, имя параметра приложения по умолчанию — AzureWebJobsTwilioAuthToken. |
|**to**| Недоступно — указать в коде | **Чтобы**| Требуемое значение: номер телефона, на который отправляется текст SMS-сообщения.|
|**from**|**from** | **От**| Требуемое значение: номер телефона, с которого отправляется текст SMS-сообщения.|
|**body**|**body** | **Текст**| Это значение можно использовать для жесткого кодирования текстового SMS-сообщения, если его не нужно задавать динамически в коде функции. |  

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Основные понятия триггеров и привязок в Функциях Azure](functions-triggers-bindings.md)
