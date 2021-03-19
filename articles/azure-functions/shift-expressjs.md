---
title: Сдвиг от Express.js к функциям Azure
description: Сведения о реструктуризации Express.js конечных точек в функции Azure.
author: craigshoemaker
ms.topic: conceptual
ms.date: 07/31/2020
ms.author: cshoe
ms.openlocfilehash: 266df5371ff5f47526fa9d6567c62e31d51ebb05
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87810230"
---
# <a name="shifting-from-expressjs-to-azure-functions"></a>Сдвиг от Express.js к функциям Azure

Express.js является одной из самых популярных Node.js платформ для веб-разработчиков и остается отличным выбором для создания приложений, обслуживающих конечные точки API.

При переносе кода в бессерверную архитектуру рефакторинг Express.js конечных точек влияет на следующие области:

- По **промежуточного слоя**: Express.js обеспечивает надежную коллекцию по промежуточного слоя. Многие модули по промежуточного слоя больше не требуются для функций Azure и [управления API Azure](../api-management/api-management-key-concepts.md) . Перед миграцией конечных точек убедитесь, что вы можете реплицировать или заменить любую логику, обработанную по промежуточного слоя.

- **Различные API-интерфейсы**. API-интерфейс, используемый для обработки запросов и ответов, отличается в разных функциях и Express.js Azure. В следующем примере подробно описаны необходимые изменения.

- **Маршрут по умолчанию**: по умолчанию конечные точки функций Azure предоставляются по `api` маршруту. Правила маршрутизации можно настроить с помощью [ `routePrefix` в _host.js_ файле](./functions-bindings-http-webhook-output.md#hostjson-settings).

- **Конфигурация и соглашения**. приложение-функция использует _function.jsв_ файле для определения глаголов HTTP, определения политик безопасности и настройки [входных и выходных данных](./functions-triggers-bindings.md)функции. По умолчанию имя папки, которая содержит файлы функции, определяет имя конечной точки, но имя можно изменить с помощью `route` свойства в [function.js](./functions-bindings-http-webhook-trigger.md#customize-the-http-endpoint) в файле.

> [!TIP]
> Дополнительные сведения см. в интерактивном учебнике [рефакторинг Node.js и Express API для бессерверных интерфейсов API с помощью функций Azure](/learn/modules/shift-nodejs-express-apis-serverless/).

## <a name="example"></a>Пример

### <a name="expressjs"></a>Express.js

В следующем примере показана типичная `GET` Конечная точка Express.js.

```javascript
// server.js
app.get('/hello', (req, res) => {
  try {
    res.send("Success!");
  } catch(error) {
    const err = JSON.stringify(error);
    res.status(500).send(`Request error. ${err}`);
  }
});
```

При `GET` отправке запроса в `/hello` `HTTP 200` возвращается ответ, содержащий `Success` . Если в конечной точке возникает ошибка, то ответ содержит `HTTP 500` сведения об ошибке.

### <a name="azure-functions"></a>Функции Azure

Функции Azure упорядочивают файлы конфигурации и кода в одну папку для каждой функции. По умолчанию имя папки определяется именем функции.

Например, функция с именем `hello` имеет папку со следующими файлами.

``` files
| - hello
|  - function.json
|  - index.js
```

В следующем примере реализуется тот же результат, что и в указанной выше Express.js конечной точке, но с помощью функций Azure.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
// hello/index.js
module.exports = async function (context, req) {
  try {
    context.res = { body: "Success!" };
  } catch(error) {
    const err = JSON.stringify(error);
    context.res = {
      status: 500,
      body: `Request error. ${err}`
    };
  }
};
```

# <a name="typescript"></a>[TypeScript](#tab/typescript)

```typescript
// hello/index.ts
import { AzureFunction, Context, HttpRequest } from "@azure/functions";

const httpTrigger: AzureFunction = async function (context: Context, req: HttpRequest): Promise<void> {
  try {
    context.res = { body: "Success!" };
  } catch (error) {
    const err = JSON.stringify(error);
    context.res = {
      status: 500,
      body: `Request error. ${err}`,
    };
  }
};

export default httpTrigger;
```

---

При переходе к функциям вносятся следующие изменения:

- **Модуль:** Код функции реализуется как модуль JavaScript.

- **Объект контекста и ответа**: [`context`](./functions-reference-node.md#context-object) позволяет взаимодействовать со средой выполнения функции. Из контекста можно считывать данные запроса и задавать ответ функции. Синхронный код требует вызова `context.done()` для завершения выполнения, в то время как `asyc` функции разрешают запрос неявно.

- **Соглашение об именовании**. имя папки, используемое для хранения файлов функций Azure, по умолчанию используется в качестве имени конечной точки (его можно переопределить в [function.json](./functions-bindings-http-webhook-trigger.md#customize-the-http-endpoint)).

- **Конфигурация**. вы определяете глаголы HTTP в [function.js](./functions-bindings-http-webhook-trigger.md#customize-the-http-endpoint) в файле, например `POST` или `PUT` .

В следующем _function.js_ файла содержатся сведения о конфигурации для функции.

```json
{
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": ["get"]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ]
}
```

Определяя `get` в `methods` массиве, функция доступна для HTTP- `GET` запросов. Если вы хотите, чтобы ваш API принимал запросы на поддержку `POST` , можно `post` также добавить в массив.

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения см. в интерактивном учебнике [рефакторинг Node.js и Express API для бессерверных интерфейсов API с помощью функций Azure](/learn/modules/shift-nodejs-express-apis-serverless/) .