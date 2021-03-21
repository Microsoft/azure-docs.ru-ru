---
title: Проверка областей и ролей приложений в защищенном веб-API | Azure
titleSuffix: Microsoft identity platform
description: Убедитесь, что API вызывается только приложениями от имени пользователей, которые имеют правильные области действия и управляющие приложения, имеющие правильные роли приложений.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/15/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 6d737f107e7a9b1476fdf86ac0320931ea137671
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94442911"
---
# <a name="protected-web-api-verify-scopes-and-app-roles"></a>Защищенный веб-API: проверка областей и ролей приложений

В этой статье описывается, как можно добавить авторизацию в веб-API. Эта защита гарантирует, что API будет вызываться только:

- приложениями от имени пользователей, имеющих нужные области;
- управляющими приложениями, имеющими правильные роли приложений.

> [!NOTE]
> Фрагменты кода в этой статье извлекаются из следующих примеров кода на сайте GitHub:
>
> - [Пошаговое руководство по ASP.NET Core веб-API](https://github.com/Azure-Samples/active-directory-dotnet-native-aspnetcore-v2/blob/master/1.%20Desktop%20app%20calls%20Web%20API/TodoListService/Controllers/TodoListController.cs)
> - [Пример веб-API ASP.NET](https://github.com/Azure-Samples/ms-identity-aspnet-webapi-onbehalfof/blob/master/TodoListService/Controllers/TodoListController.cs)

Чтобы защитить веб-API ASP.NET или ASP.NET Core, необходимо добавить атрибут `[Authorize]` в один из следующих элементов:

- сам контроллер, если требуется защитить все действия контроллера;
- действие отдельного контроллера для API.

```csharp
    [Authorize]
    public class TodoListController : Controller
    {
     ...
    }
```

Но этой защиты недостаточно. Она гарантирует только то, что ASP.NET и ASP.NET Core проверят маркер. API необходимо проверить, что маркер, используемый для вызова API, запрашивается с ожидаемыми утверждениями. В частности, требуют проверки следующие утверждения.

- *Области*, если API вызывается от имени пользователя.
- *Роли приложения*, если API можно вызывать из управляющего приложения.

## <a name="verify-scopes-in-apis-called-on-behalf-of-users"></a>Проверка областей в интерфейсах API, вызываемых от имени пользователей

Если клиентское приложение вызывает API от имени пользователя, API должен запросить токен носителя с конкретными областями для API. Дополнительные сведения см. в [Конфигурация кода | Токен носителя](scenario-protected-web-api-app-configuration.md#bearer-token).

### <a name="net-core"></a>.NET Core

#### <a name="verify-the-scopes-on-each-controller-action"></a>Проверка областей действия каждого контроллера

```csharp
[Authorize]
public class TodoListController : Controller
{
    /// <summary>
    /// The web API will accept only tokens 1) for users, 2) that have the `access_as_user` scope for
    /// this API.
    /// </summary>
    static readonly string[] scopeRequiredByApi = new string[] { "access_as_user" };

    // GET: api/values
    [HttpGet]
    public IEnumerable<TodoItem> Get()
    {
         HttpContext.VerifyUserHasAnyAcceptedScope(scopeRequiredByApi);
        // Do the work and return the result.
        // ...
    }
...
}
```

Метод `VerifyUserHasAnyAcceptedScope` делает нечто вроде следующего.

- Проверьте, существует ли утверждение с именем `http://schemas.microsoft.com/identity/claims/scope` или `scp`.
- Проверьте, что утверждение имеет значение, которое содержит область, ожидаемую API.


#### <a name="verify-the-scopes-more-globally"></a>Проверка областей на глобальном уровне

Рекомендуемым подходом является определение детализированных областей для веб-API и проверка областей в каждом действии контроллера. Однако можно также проверить области на уровне приложения или контроллера с помощью ASP.NET Core. Дополнительные сведения см. в разделе [авторизация на основе утверждений](/aspnet/core/security/authorization/claims) в документации по ASP.NET Core.

### <a name="net-mvc"></a>ПЛАТФОРМА .NET MVC

Для ASP.NET просто замените `HttpContext.User` на `ClaimsPrincipal.Current` и замените тип утверждения `"http://schemas.microsoft.com/identity/claims/scope"` на `"scp"`. См. также фрагмент кода ниже в этой статье.

## <a name="verify-app-roles-in-apis-called-by-daemon-apps"></a>Проверка ролей приложения в интерфейсах API, которые вызываются приложениями управляющей программы

Если веб-API вызывается [приложением управляющей программы](scenario-daemon-overview.md), это приложение должно требовать разрешения приложения для веб-API. Как показано в разделе [Предоставление разрешений приложения (роли приложений)](./scenario-protected-web-api-app-registration.md#exposing-application-permissions-app-roles), API предоставляет такие разрешения. Одним из примеров является роль приложения `access_as_application`.

Теперь необходимо, чтобы ваш API проверял, что полученный маркер содержит утверждение `roles` и что это утверждение имеет ожидаемое значение. Код проверки аналогичен коду, который проверяет делегированные разрешения, за исключением того, что действие контроллера проверяет роли, а не области.

### <a name="aspnet-core"></a>ASP.NET Core

```csharp
[Authorize]
public class TodoListController : ApiController
{
    public IEnumerable<TodoItem> Get()
    {
        HttpContext.ValidateAppRole("access_as_application");
        ...
    }
```

`ValidateAppRole`Метод определен в Microsoft. Identity. Web в [ролесрекуиредхттпконтекстекстенсионс. CS](https://github.com/AzureAD/microsoft-identity-web/blob/d2ad0f5f830391a34175d48621a2c56011a45082/src/Microsoft.Identity.Web/Resource/RolesRequiredHttpContextExtensions.cs#L28).

### <a name="aspnet-mvc"></a>ASP.NET MVC 3

```csharp
private void ValidateAppRole(string appRole)
{
    //
    // The `role` claim tells you what permissions the client application has in the service.
    // In this case, we look for a `role` value of `access_as_application`.
    //
    Claim roleClaim = ClaimsPrincipal.Current.FindFirst("roles");
    if (roleClaim == null || !roleClaim.Value.Split(' ').Contains(appRole))
    {
        throw new HttpResponseException(new HttpResponseMessage
        { StatusCode = HttpStatusCode.Unauthorized,
            ReasonPhrase = $"The 'roles' claim does not contain '{appRole}' or was not found"
        });
    }
}
}
```

### <a name="accepting-app-only-tokens-if-the-web-api-should-be-called-only-by-daemon-apps"></a>Принятие маркеров только для приложений, если веб-API должен вызываться только управляющими приложениями

Пользователи также могут использовать утверждения ролей в шаблонах назначения пользователей, как показано в [Практическом руководстве. Добавление ролей приложения в приложение, зарегистрированное в Azure Active Directory, и их получение в токене](howto-add-app-roles-in-azure-ad-apps.md). Если роли могут быть назначены обоим, проверка ролей позволит приложениям входить в систему в качестве пользователей, а пользователям входить в качестве приложений. Для предотвращения этой путаницы рекомендуется объявлять разные роли для пользователей и приложений.

Если вы хотите, чтобы только управляющие приложения вызывали веб-API, добавьте условие, согласно которому маркер является маркером только для приложения при проверке роли приложения.

```csharp
string oid = ClaimsPrincipal.Current.FindFirst("oid")?.Value;
string sub = ClaimsPrincipal.Current.FindFirst("sub")?.Value;
bool isAppOnlyToken = oid == sub;
```

Проверка обратного условия позволяет только приложениям, которые входят в систему в качестве пользователя, вызывать ваш API.

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к следующей статье в этом сценарии, [перейдите в рабочую среду](scenario-protected-web-api-production.md).
