---
title: Функции HTTP в Устойчивые функции — функции Azure
description: Сведения об интегрированных функциях HTTP в расширении Устойчивые функции для функций Azure.
author: cgillum
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: azfuncdf
ms.openlocfilehash: 64d40de50f21811a56318971de1836abc8fbf8c9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93027267"
---
# <a name="http-features"></a>Функции HTTP

Устойчивые функции имеет несколько функций, которые упрощают внедрение устойчивых оркестрации и сущностей в рабочие процессы HTTP. В этой статье подробно рассматриваются некоторые из этих функций.

## <a name="exposing-http-apis"></a>Предоставление API-интерфейсов HTTP

Оркестрации и сущности могут вызываться и управляться с помощью HTTP-запросов. Расширение Устойчивые функции предоставляет встроенные API HTTP. Он также предоставляет интерфейсы API для взаимодействия с оркестрации и сущностями из функций, активируемых HTTP.

### <a name="built-in-http-apis"></a>Встроенные API HTTP

Расширение Устойчивые функции автоматически добавляет набор API-интерфейсов HTTP к узлу функций Azure. С помощью этих API-интерфейсов можно взаимодействовать и управлять согласованиями и сущностями без написания кода.

Поддерживаются следующие встроенные API HTTP.

* [Начать новое согласование](durable-functions-http-api.md#start-orchestration)
* [Экземпляр оркестрации запросов](durable-functions-http-api.md#get-instance-status)
* [Завершить экземпляр оркестрации](durable-functions-http-api.md#terminate-instance)
* [Отправка внешнего события в оркестрации](durable-functions-http-api.md#raise-event)
* [Очистка журнала оркестрации](durable-functions-http-api.md#purge-single-instance-history)
* [Отправка события операции в сущность](durable-functions-http-api.md#signal-entity)
* [Получение состояния сущности](durable-functions-http-api.md#get-entity)
* [Запрос списка сущностей](durable-functions-http-api.md#list-entities)

Полное описание всех встроенных API HTTP, предоставляемых расширением Устойчивые функции, см. в [статье API-интерфейсы HTTP](durable-functions-http-api.md) .

### <a name="http-api-url-discovery"></a>Обнаружение URL-адреса API HTTP

[Привязка клиента оркестрации](durable-functions-bindings.md#orchestration-client) предоставляет интерфейсы API, которые могут создавать удобные полезные данные ответа HTTP. Например, он может создать ответ, содержащий ссылки на API управления для конкретного экземпляра оркестрации. В следующих примерах показана функция HTTP-триггера, которая демонстрирует использование этого API для нового экземпляра оркестрации:

# <a name="c"></a>[C#](#tab/csharp)

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/HttpStart.cs)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

**index.js**

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/HttpStart/index.js)]

**function.json**

[!code-json[Main](~/samples-durable-functions/samples/javascript/HttpStart/function.json)]

# <a name="python"></a>[Python](#tab/python)

**__init__. Корректировка**

```python
import logging
import azure.functions as func
import azure.durable_functions as df

async def main(req: func.HttpRequest, starter: str) -> func.HttpResponse:
    client = df.DurableOrchestrationClient(starter)
    function_name = req.route_params['functionName']
    event_data = req.get_body()

    instance_id = await client.start_new(function_name, instance_id, event_data)
    
    logging.info(f"Started orchestration with ID = '{instance_id}'.")
    return client.create_check_status_response(req, instance_id)
```

**function.json**

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "authLevel": "function",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "route": "orchestrators/{functionName}",
      "methods": [
        "post",
        "get"
      ]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "name": "starter",
      "type": "orchestrationClient",
      "direction": "in"
    }
  ]
}
```

---

Запуск функции Orchestrator с помощью функций HTTP-триггеров, показанных выше, можно выполнить с помощью любого клиента HTTP. Следующая фигурная команда запускает функцию Orchestrator с именем `DoWork` :

```bash
curl -X POST https://localhost:7071/orchestrators/DoWork -H "Content-Length: 0" -i
```

Далее приведен пример ответа для оркестрации, имеющего `abc123` идентификатор. Некоторые сведения были удалены для ясности.

```http
HTTP/1.1 202 Accepted
Content-Type: application/json; charset=utf-8
Location: http://localhost:7071/runtime/webhooks/durabletask/instances/abc123?code=XXX
Retry-After: 10

{
    "id": "abc123",
    "purgeHistoryDeleteUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123?code=XXX",
    "sendEventPostUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123/raiseEvent/{eventName}?code=XXX",
    "statusQueryGetUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123?code=XXX",
    "terminatePostUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123/terminate?reason={text}&code=XXX"
}
```

В предыдущем примере каждое из полей, оканчивающихся на, `Uri` соответствует встроенному API HTTP. Эти API можно использовать для управления целевым экземпляром оркестрации.

> [!NOTE]
> Формат URL-адресов веб перехватчика зависит от версии используемого узла функций Azure. Предыдущий пример относится к узлу функции Azure 2,0.

Описание всех встроенных API HTTP см. в [справочнике по API HTTP](durable-functions-http-api.md).

### <a name="async-operation-tracking"></a>Отслеживание асинхронных операций

Упомянутый ранее HTTP-ответ предназначен для помощи в реализации долго выполняющихся асинхронных API-интерфейсов HTTP с устойчивыми функциями. Этот шаблон иногда называют *шаблоном объекта-получателя опроса*. Поток клиента или сервера работает следующим образом:

1. Клиент отправляет запрос HTTP для запуска длительного процесса, например, функции Orchestrator.
1. Целевой триггер HTTP возвращает ответ HTTP 202 с заголовком Location со значением "statusQueryGetUri".
1. Клиент опрашивает URL-адрес в заголовке Location. Клиент продолжит видеть ответы HTTP 202 с заголовком Location.
1. Когда экземпляр завершается или завершается ошибкой, конечная точка в заголовке Location возвращает HTTP 200.

Этот протокол обеспечивает координацию длительно выполняющихся процессов с внешними клиентами или службами, которые могут опрашивать конечную точку HTTP и следовать заголовку Location. Серверные и клиентские реализации этого шаблона встроены в API-интерфейсы Устойчивые функции HTTP.

> [!NOTE]
> По умолчанию все действия на основе HTTP, предоставляемые [Azure Logic Apps](https://azure.microsoft.com/services/logic-apps/), поддерживают стандартную модель асинхронных операций. Эта возможность позволяет внедрять долго выполняющиеся устойчивые функции в рамках рабочего процесса Logic Apps. Дополнительные сведения о Logic Apps поддержки для асинхронных шаблонов HTTP см. в [документации по действиям и триггерам рабочего процесса Azure Logic Apps](../../logic-apps/logic-apps-workflow-actions-triggers.md).

> [!NOTE]
> Взаимодействие с оркестрации можно выполнять из любого типа функции, а не только с помощью функций, активируемых HTTP.

Дополнительные сведения об управлении согласованиями и сущностями с помощью клиентских API см. в [статье Управление экземплярами](durable-functions-instance-management.md).

## <a name="consuming-http-apis"></a>Использование API-интерфейсов HTTP

Как описано в статье [ограничения кода функции Orchestrator](durable-functions-code-constraints.md), функции Orchestrator не могут выполнять операции ввода-вывода напрямую. Вместо этого они обычно вызывают [функции действий](durable-functions-types-features-overview.md#activity-functions) , которые выполняют операции ввода-вывода.

Начиная с Устойчивые функции 2,0, оркестрации могут использовать API-интерфейсы HTTP с помощью [привязки триггера оркестрации](durable-functions-bindings.md#orchestration-trigger).

В следующем примере кода показана функция Orchestrator, выполняющая исходящий HTTP-запрос:

# <a name="c"></a>[C#](#tab/csharp)

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

module.exports = df.orchestrator(function*(context){
    const url = context.df.getInput();
    const response = yield context.df.callHttp("GET", url)

    if (response.statusCode >= 400) {
        // handling of error codes goes here
    }
});
```
# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df
from datetime import datetime, timedelta

def orchestrator_function(context: df.DurableOrchestrationContext):
    url = context.get_input()
    response = yield context.call_http('GET', url)
    
    if response["statusCode"] >= 400:
        # handling of error codes goes here

main = df.Orchestrator.create(orchestrator_function)
```

---

С помощью действия Call HTTP можно выполнять следующие действия в функциях Orchestrator:

* Вызывайте API HTTP непосредственно из функций оркестрации с некоторыми ограничениями, упомянутыми позже.
* Автоматически поддерживает шаблоны опроса состояния HTTP 202 на стороне клиента.
* Используйте [управляемые удостоверения Azure](../../active-directory/managed-identities-azure-resources/overview.md) для выполнения санкционированных вызовов HTTP к другим конечным точкам Azure.

Возможность использования API-интерфейсов HTTP непосредственно из функций Orchestrator предназначена для удобства работы с определенным набором распространенных сценариев. Все эти функции можно реализовать самостоятельно с помощью функций действий. Во многих случаях функции действий могут обеспечить большую гибкость.

### <a name="http-202-handling"></a>Обработка HTTP 202

API "Call HTTP" может автоматически реализовать клиентскую сторону шаблона опрашивающего потребителя. Если вызываемый API возвращает ответ HTTP 202 с заголовком Location, функция Orchestrator автоматически опрашивает ресурс расположения, пока не получит ответ, отличный от 202. Ответ будет возвращен в код функции Orchestrator.

> [!NOTE]
> Функции Orchestrator также изначально поддерживают клиентский шаблон опроса на стороне сервера, как описано в разделе [Отслеживание асинхронных операций](#async-operation-tracking). Такая поддержка означает, что согласование в одном приложении функции может легко координировать функции Orchestrator в других приложениях функций. Это похоже на концепцию [подсистемы оркестрации](durable-functions-sub-orchestrations.md) , но с поддержкой взаимодействия между приложениями. Эта поддержка особенно полезна при разработке приложений в стиле микрослужб.

### <a name="managed-identities"></a>Управляемые удостоверения

Устойчивые функции изначально поддерживает вызовы API, которые принимают маркеры Azure Active Directory (Azure AD) для авторизации. Эта поддержка использует [управляемые удостоверения Azure](../../active-directory/managed-identities-azure-resources/overview.md) для получения этих маркеров.

Следующий код является примером функции .NET Orchestrator. Функция выполняет проверку подлинности вызовов для перезапуска виртуальной машины с помощью Azure Resource Manager [виртуальных машин REST API](/rest/api/compute/virtualmachines).

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("RestartVm")]
public static async Task RunOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    string subscriptionId = "mySubId";
    string resourceGroup = "myRG";
    string vmName = "myVM";
    string apiVersion = "2019-03-01";
    
    // Automatically fetches an Azure AD token for resource = https://management.core.windows.net/.default
    // and attaches it to the outgoing Azure Resource Manager API call.
    var restartRequest = new DurableHttpRequest(
        HttpMethod.Post, 
        new Uri($"https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Compute/virtualMachines/{vmName}/restart?api-version={apiVersion}"),
        tokenSource: new ManagedIdentityTokenSource("https://management.core.windows.net/.default"));
    DurableHttpResponse restartResponse = await context.CallHttpAsync(restartRequest);
    if (restartResponse.StatusCode != HttpStatusCode.OK)
    {
        throw new ArgumentException($"Failed to restart VM: {restartResponse.StatusCode}: {restartResponse.Content}");
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const subscriptionId = "mySubId";
    const resourceGroup = "myRG";
    const vmName = "myVM";
    const apiVersion = "2019-03-01";
    const tokenSource = new df.ManagedIdentityTokenSource("https://management.core.windows.net/.default");

    // get a list of the Azure subscriptions that I have access to
    const restartResponse = yield context.df.callHttp(
        "POST",
        `https://management.azure.com/subscriptions/${subscriptionId}/resourceGroups/${resourceGroup}/providers/Microsoft.Compute/virtualMachines/${vmName}/restart?api-version=${apiVersion}`,
        undefined, // no request content
        undefined, // no request headers (besides auth which is handled by the token source)
        tokenSource);

    return restartResponse;
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    subscription_id = "mySubId"
    resource_group = "myRg"
    vm_name = "myVM"
    api_version = "2019-03-01"
    token_source = df.ManagedIdentityTokenSource("https://management.core.windows.net/.default")

    # get a list of the Azure subscriptions that I have access to
    restart_response = yield context.call_http("POST", 
        f"https://management.azure.com/subscriptions/{subscription_id}/resourceGroups/{resource_group}/providers/Microsoft.Compute/virtualMachines/{vm_name}/restart?api-version={api_version}",
        None,
        None,
        token_source)
    return restart_response

main = df.Orchestrator.create(orchestrator_function)
```

---

В предыдущем примере `tokenSource` параметр настроен для получения маркеров Azure AD для [Azure Resource Manager](../../azure-resource-manager/management/overview.md). Токены идентифицируются с помощью URI ресурса `https://management.core.windows.net/.default` . В этом примере предполагается, что текущее приложение-функция выполняется локально или развернуто как приложение-функция с управляемым удостоверением. Предполагается, что локальным удостоверением или управляемым удостоверением предоставлено разрешение на управление виртуальными машинами в указанной группе ресурсов `myRG` .

Во время выполнения настроенный источник токена автоматически возвращает маркер доступа OAuth 2,0. Затем источник добавляет маркер в качестве токена носителя в заголовок авторизации исходящего запроса. Эта модель является улучшением по сравнению с добавлением заголовков авторизации вручную в HTTP-запросы по следующим причинам.

* Обновление токена обрабатывается автоматически. Вам не нужно беспокоиться о токенах с истекшим сроком действия.
* Токены никогда не хранятся в состоянии устойчивого оркестрации.
* Вам не нужно писать код для управления получением маркера.

Более полный пример можно найти в [предварительно скомпилированном примере C# рестартвмс](https://github.com/Azure/azure-functions-durable-extension/blob/dev/samples/precompiled/RestartVMs.cs).

Управляемые удостоверения не ограничиваются управлением ресурсами Azure. Управляемые удостоверения можно использовать для доступа к любому API, который принимает токены носителя Azure AD, включая службы Azure из Майкрософт и веб-приложения от партнеров. Веб-приложение партнера может даже быть другим приложением-функцией. Список служб Azure, которые поддерживают проверку подлинности в Azure AD, см. в статье [службы Azure, поддерживающие аутентификацию Azure AD](../../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication).

### <a name="limitations"></a>Ограничения

Встроенная поддержка вызова API-интерфейсов HTTP является удобной функцией. Это не подходит для всех сценариев.

HTTP-запросы, отправленные функциями Orchestrator и их ответами, сериализуются и сохраняются как сообщения очереди. Такое поведение очередей гарантирует надежность и безопасность HTTP-вызовов при [воспроизведении оркестрации](durable-functions-orchestrations.md#reliability). Однако поведение очереди также имеет ограничения.

* Каждый HTTP-запрос требует дополнительной задержки по сравнению с собственным HTTP-клиентом.
* Большие сообщения запроса или ответа, которые не могут поместиться в очередь, могут значительно снизить производительность оркестрации. Накладные расходы на разгрузку полезных данных сообщений в хранилище BLOB-объектов могут привести к снижению производительности.
* Потоковая передача, фрагментированность и двоичные данные не поддерживаются.
* Возможность настройки поведения клиента HTTP ограничена.

Если какое-либо из этих ограничений может повлиять на ваш вариант использования, вместо этого следует использовать функции действий и клиентские библиотеки HTTP, зависящие от языка, для выполнения исходящих вызовов HTTP.

> [!NOTE]
> Если вы являетесь разработчиком .NET, то можете спросить, почему эта функция использует типы **дураблехттпрекуест** и **дураблехттпреспонсе** вместо встроенных типов .NET **HttpRequestMessage** и **HttpResponseMessage** .
>
> Такое поведение реализовано намеренно. Основная причина заключается в том, что пользовательские типы помогают убедиться в том, что пользователи не делают неправильные предположения о поддерживаемых поведениях внутреннего HTTP-клиента. Типы, характерные для Устойчивые функции, позволяют упростить разработку API. Они также могут упростить доступ к специальным функциям, таким как [Интеграция управляемой идентификации](#managed-identities) и [шаблон опрашивающего потребителя](#http-202-handling). 

### <a name="extensibility-net-only"></a>Расширяемость (только .NET)

Настройка поведения внутреннего HTTP-клиента оркестрации возможна с помощью [внедрения зависимостей .NET для функций Azure](../functions-dotnet-dependency-injection.md). Эта возможность может быть полезной для внесения небольших изменений в поведение. Он также может быть полезен для модульного тестирования HTTP-клиента путем внедрения макетов объектов.

В следующем примере демонстрируется использование внедрения зависимостей для отключения проверки сертификата TLS/SSL для функций Orchestrator, вызывающих внешние конечные точки HTTP.

```csharp
public class Startup : FunctionsStartup
{
    public override void Configure(IFunctionsHostBuilder builder)
    {
        // Register own factory
        builder.Services.AddSingleton<
            IDurableHttpMessageHandlerFactory,
            MyDurableHttpMessageHandlerFactory>();
    }
}

public class MyDurableHttpMessageHandlerFactory : IDurableHttpMessageHandlerFactory
{
    public HttpMessageHandler CreateHttpMessageHandler()
    {
        // Disable TLS/SSL certificate validation (not recommended in production!)
        return new HttpClientHandler
        {
            ServerCertificateCustomValidationCallback =
                HttpClientHandler.DangerousAcceptAnyServerCertificateValidator,
        };
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Сведения о устойчивых сущностях](durable-functions-entities.md)
