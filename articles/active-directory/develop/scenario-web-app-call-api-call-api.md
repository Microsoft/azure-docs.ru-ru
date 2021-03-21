---
title: Вызов веб-API из веб-приложения | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как создать веб-приложение, вызывающее веб-API (вызов защищенного веб-API).
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/25/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: b294a56a523adaa2629a5d1e72a7ccef532956e0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98753284"
---
# <a name="a-web-app-that-calls-web-apis-call-a-web-api"></a>Веб-приложение, вызывающее веб-API: вызов веб-API

Теперь, когда у вас есть маркер, можно вызвать защищенный веб-API. Обычно подчиненный API вызывается из контроллера или страниц веб-приложения.

## <a name="call-a-protected-web-api"></a>Вызов защищенного веб-API

Вызов защищенного веб-API зависит от выбранного языка и платформы.

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

При использовании *Microsoft. Identity. Web* у вас есть три варианта использования для вызова API:

- [Вариант 1. вызов Microsoft Graph с помощью пакета SDK для Microsoft Graph](#option-1-call-microsoft-graph-with-the-sdk)
- [Вариант 2. вызов подчиненного веб-API с помощью вспомогательного класса](#option-2-call-a-downstream-web-api-with-the-helper-class)
- [Вариант 3. вызов подчиненного веб-интерфейса API без вспомогательного класса](#option-3-call-a-downstream-web-api-without-the-helper-class)

#### <a name="option-1-call-microsoft-graph-with-the-sdk"></a>Вариант 1. вызов Microsoft Graph с помощью пакета SDK

Необходимо вызвать Microsoft Graph. В этом сценарии вы добавили `AddMicrosoftGraph` в *Startup. CS* , как указано в [конфигурации кода](scenario-web-app-call-api-app-configuration.md#option-1-call-microsoft-graph), и можете напрямую внедрить `GraphServiceClient` в контроллер или конструктор страницы для использования в действиях. В следующем примере на странице Razor отображается фотография пользователя, выполнившего вход.

```CSharp
[Authorize]
[AuthorizeForScopes(Scopes = new[] { "user.read" })]
public class IndexModel : PageModel
{
 private readonly GraphServiceClient _graphServiceClient;

 public IndexModel(GraphServiceClient graphServiceClient)
 {
    _graphServiceClient = graphServiceClient;
 }

 public async Task OnGet()
 {
  var user = await _graphServiceClient.Me.Request().GetAsync();
  try
  {
   using (var photoStream = await _graphServiceClient.Me.Photo.Content.Request().GetAsync())
   {
    byte[] photoByte = ((MemoryStream)photoStream).ToArray();
    ViewData["photo"] = Convert.ToBase64String(photoByte);
   }
   ViewData["name"] = user.DisplayName;
  }
  catch (Exception)
  {
   ViewData["photo"] = null;
  }
 }
}
```

#### <a name="option-2-call-a-downstream-web-api-with-the-helper-class"></a>Вариант 2. вызов подчиненного веб-API с помощью вспомогательного класса

Вы хотите вызвать веб-API, отличный от Microsoft Graph. В этом случае вы добавили `AddDownstreamWebApi` в *Startup. CS* , как указано в [конфигурации кода](scenario-web-app-call-api-app-configuration.md#option-2-call-a-downstream-web-api-other-than-microsoft-graph), и можете напрямую внедрить `IDownstreamWebApi` службу в контроллер или конструктор страниц и использовать ее в действиях:

```CSharp
[Authorize]
[AuthorizeForScopes(ScopeKeySection = "TodoList:Scopes")]
public class TodoListController : Controller
{
  private IDownstreamWebApi _downstreamWebApi;
  private const string ServiceName = "TodoList";

  public TodoListController(IDownstreamWebApi downstreamWebApi)
  {
    _downstreamWebApi = downstreamWebApi;
  }

  public async Task<ActionResult> Details(int id)
  {
    var value = await _downstreamWebApi.CallWebApiForUserAsync(
      ServiceName,
      options =>
      {
        options.RelativePath = $"me";
      });
      return View(value);
  }
}
```

`CallWebApiForUserAsync`Также имеет строго типизированные универсальные переопределения, позволяющие напрямую получить объект. Например, следующий метод получает `Todo` экземпляр, который является строго типизированным представлением JSON, возвращенного веб-API.

```CSharp
    // GET: TodoList/Details/5
    public async Task<ActionResult> Details(int id)
    {
        var value = await _downstreamWebApi.CallWebApiForUserAsync<object, Todo>(
            ServiceName,
            null,
            options =>
            {
                options.HttpMethod = HttpMethod.Get;
                options.RelativePath = $"api/todolist/{id}";
            });
        return View(value);
    }
   ```

#### <a name="option-3-call-a-downstream-web-api-without-the-helper-class"></a>Вариант 3. вызов подчиненного веб-интерфейса API без вспомогательного класса

Вы решили получить маркер вручную с помощью `ITokenAcquisition` службы, и теперь необходимо использовать маркер. В этом случае следующий код продолжит пример кода, показанного в [веб-приложении, которое вызывает веб-API: получить маркер для приложения](scenario-web-app-call-api-acquire-token.md). Код вызывается в действиях контроллеров веб-приложений.

После получения маркера используйте его в качестве токена носителя для вызова подчиненного API, в данном случае Microsoft Graph.

 ```csharp
public async Task<IActionResult> Profile()
{
 // Acquire the access token.
 string[] scopes = new string[]{"user.read"};
 string accessToken = await tokenAcquisition.GetAccessTokenForUserAsync(scopes);

 // Use the access token to call a protected web API.
 HttpClient client = new HttpClient();
 client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);

  var response = await httpClient.GetAsync($"{webOptions.GraphApiUrl}/beta/me");

  if (response.StatusCode == HttpStatusCode.OK)
  {
   var content = await response.Content.ReadAsStringAsync();

   dynamic me = JsonConvert.DeserializeObject(content);
   ViewData["Me"] = me;
  }

  return View();
}
```
> [!NOTE]
> Вы можете использовать тот же принцип для вызова любого веб-API.
>
> Большинство веб-API Azure предоставляют пакет SDK, который упрощает вызов API, как в случае Microsoft Graph. См., например, [Создание веб-приложения, которое разрешает доступ к хранилищу BLOB-объектов с помощью Azure AD](../../storage/common/storage-auth-aad-app.md?tabs=dotnet&toc=%2fazure%2fstorage%2fblobs%2ftoc.json) , например, с помощью Microsoft. Identity. Web и пакета SDK службы хранилища Azure.

# <a name="java"></a>[Java](#tab/java)

```Java
private String getUserInfoFromGraph(String accessToken) throws Exception {
    // Microsoft Graph user endpoint
    URL url = new URL("https://graph.microsoft.com/v1.0/me");

    HttpURLConnection conn = (HttpURLConnection) url.openConnection();

    // Set the appropriate header fields in the request header.
    conn.setRequestProperty("Authorization", "Bearer " + accessToken);
    conn.setRequestProperty("Accept", "application/json");

    String response = HttpClientHelper.getResponseStringFromConn(conn);

    int responseCode = conn.getResponseCode();
    if(responseCode != HttpURLConnection.HTTP_OK) {
        throw new IOException(response);
    }

    JSONObject responseObject = HttpClientHelper.processResponse(responseCode, response);
    return responseObject.toString();
}

```

# <a name="python"></a>[Python](#tab/python)

```Python
@app.route("/graphcall")
def graphcall():
    token = _get_token_from_cache(app_config.SCOPE)
    if not token:
        return redirect(url_for("login"))
    graph_data = requests.get(  # Use token to call downstream service.
        app_config.ENDPOINT,
        headers={'Authorization': 'Bearer ' + token['access_token']},
        ).json()
    return render_template('display.html', result=graph_data)
```

---

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к следующей статье в этом сценарии, [перейдите в рабочую среду](scenario-web-app-call-api-production.md).