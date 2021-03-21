---
title: Доступ к сведениям о пользователе в Статических веб-приложениях Azure
description: Сведения о чтении данных пользователя, возвращенных поставщиком авторизации.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: conceptual
ms.date: 05/08/2020
ms.author: cshoe
ms.custom: devx-track-js
ms.openlocfilehash: d5a1d810c357aa83b8069023b00d76352da124df
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94844801"
---
# <a name="accessing-user-information-in-azure-static-web-apps-preview"></a>Доступ к сведениям о пользователе в предварительной версии службы "Статические веб-приложения Azure"

Служба "Статические веб-приложения Azure" предоставляет связанные с проверкой подлинности сведения о пользователе с помощью [конечной точки прямого доступа](#direct-access-endpoint) и передает их [функциям API](#api-functions).

Многие пользовательские интерфейсы в значительной степени опираются на данные проверки подлинности пользователя. Конечная точка прямого доступа — это служебный API-интерфейс, который предоставляет сведения о пользователе без необходимости реализовывать пользовательскую функцию. Конечная точка прямого доступа не только удобна, но и не испытывает задержек при холодном запуске, которые характерны для бессерверной архитектуры.

## <a name="client-principal-data"></a>Данные субъекта клиента

Объект данных субъекта клиента предоставляет приложению сведения, позволяющие идентифицировать пользователя. В объекте субъекта клиента представлены приведенные ниже свойства.

| Свойство  | Описание |
|-----------|---------|
| `identityProvider` | Имя [поставщика удостоверений](authentication-authorization.md). |
| `userId` | Специальный уникальный идентификатор пользователя для Статических веб-приложений Azure. <ul><li>Значение уникально для каждого приложения. Например, для одного и того же пользователя для разных ресурсов Статических веб-приложений возвращаются разные значения `userId`.<li>Значение сохраняется в течение времени существования пользователя. При удалении и добавлении одного и того же пользователя обратно в приложение создается новое значение `userId`.</ul>|
| `userDetails` | Имя или адрес электронной почты пользователя. Одни поставщики возвращают [адрес электронной почты пользователя](authentication-authorization.md), а другие — [его маркер](authentication-authorization.md). |
| `userRoles`     | Массив [назначенных пользователю ролей](authentication-authorization.md). |

Ниже приведен пример объекта субъекта клиента.

```json
{
  "identityProvider": "facebook",
  "userId": "d75b260a64504067bfc5b2905e3b8182",
  "userDetails": "user@example.com",
  "userRoles": [ "anonymous", "authenticated" ]
}
```

## <a name="direct-access-endpoint"></a>Конечная точка прямого доступа

Вы можете отправить запрос `GET` по маршруту `/.auth/me` и получить прямой доступ к данным субъекта клиента. Если состояние представления зависит от данных авторизации, используйте этот подход, чтобы улучшить производительность.

Для пользователей, вошедших в систему, ответ содержит объект JSON субъекта клиента. Для запросов от пользователей, не прошедших проверку подлинности, возвращается значение `null`.

С помощью API [fetch](https://developer.mozilla.org/docs/Web/API/Fetch_API/Using_Fetch)<sup>1</sup> можно получить доступ к данным субъекта клиента, используя следующий синтаксис.

```javascript
async function getUserInfo() {
  const response = await fetch("/.auth/me");
  const payload = await response.json();
  const { clientPrincipal } = payload;
  return clientPrincipal;
}

console.log(getUserInfo());
```

## <a name="api-functions"></a>Функции API

Функции API, доступные в статических веб-приложениях через серверную часть функций Azure, имеют доступ к тем же данным пользователя, что и клиентское приложение. Хотя API получает сведения, определяемые пользователем, он не выполняет собственных проверок, если пользователь прошел проверку подлинности или соответствует требуемой роли. Правила управления доступом определяются в [`routes.json`](routes.md) файле.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Данные субъекта клиента передаются в функции API в заголовке запроса `x-ms-client-principal`. Данные субъекта клиента отправляются в виде строки в кодировке [Base64](https://www.wikipedia.org/wiki/Base64), содержащей сериализованный объект JSON.

В следующем примере функции показано, как считывать и возвращать сведения о пользователе.

```javascript
module.exports = async function (context, req) {
  const header = req.headers["x-ms-client-principal"];
  const encoded = Buffer.from(header, "base64");
  const decoded = encoded.toString("ascii");

  context.res = {
    body: {
      clientPrincipal: JSON.parse(decoded)
    }
  };
};
```

Если приведенная выше функция называется `user`, можно использовать API браузера [fetch](https://developer.mozilla.org/docs/Web/API/Fetch_API/Using_Fetch)<sup>1</sup> для доступа к ответу API с помощью следующего синтаксиса.

```javascript
async function getUser() {
  const response = await fetch("/api/user");
  const payload = await response.json();
  const { clientPrincipal } = payload;
  return clientPrincipal;
}

console.log(await getUser());
```

# <a name="c"></a>[C#](#tab/csharp)

В функции C# сведения о пользователе доступны из `x-ms-client-principal` заголовка, который может быть десериализован в `ClaimsPrincipal` объект, или пользовательского типа. В следующем коде показано, как распаковать заголовок в промежуточный тип, `ClientPrincipal` который затем превращается в `ClaimsPrincipal` экземпляр.

```csharp
  public static class StaticWebAppsAuth
  {
    private class ClientPrincipal
    {
        public string IdentityProvider { get; set; }
        public string UserId { get; set; }
        public string UserDetails { get; set; }
        public IEnumerable<string> UserRoles { get; set; }
    }

    public static ClaimsPrincipal Parse(HttpRequest req)
    {
        var principal = new ClientPrincipal();

        if (req.Headers.TryGetValue("x-ms-client-principal", out var header))
        {
            var data = header[0];
            var decoded = Convert.FromBase64String(data);
            var json = Encoding.ASCII.GetString(decoded);
            principal = JsonSerializer.Deserialize<ClientPrincipal>(json, new JsonSerializerOptions { PropertyNameCaseInsensitive = true });
        }

        principal.UserRoles = principal.UserRoles?.Except(new string[] { "anonymous" }, StringComparer.CurrentCultureIgnoreCase);

        if (!principal.UserRoles?.Any() ?? true)
        {
            return new ClaimsPrincipal();
        }

        var identity = new ClaimsIdentity(principal.IdentityProvider);
        identity.AddClaim(new Claim(ClaimTypes.NameIdentifier, principal.UserId));
        identity.AddClaim(new Claim(ClaimTypes.Name, principal.UserDetails));
        identity.AddClaims(principal.UserRoles.Select(r => new Claim(ClaimTypes.Role, r)));

        return new ClaimsPrincipal(identity);
    }
  }
```

---

<sup>1</sup> API-интерфейс [fetch](https://caniuse.com/#feat=fetch) и оператор [await](https://caniuse.com/#feat=mdn-javascript_operators_await) не поддерживаются в Internet Explorer.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Настройка параметров приложения](application-settings.md)
