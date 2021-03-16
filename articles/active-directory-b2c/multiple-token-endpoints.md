---
title: Миграция веб-API на основе OWIN в b2clogin.com или пользовательский домен
titleSuffix: Azure AD B2C
description: Узнайте, как включить поддержку маркеров, выдаваемых несколькими поставщиками маркеров, в веб-API .NET при переносе приложений в b2clogin.com.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/15/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 860f167913211ee7c511e515937f29ba5bf954cf
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103491575"
---
# <a name="migrate-an-owin-based-web-api-to-b2clogincom-or-a-custom-domain"></a>Перенос веб-API на основе OWIN в b2clogin.com или пользовательский домен

В этой статье описывается способ включения поддержки нескольких издателей маркеров в веб-API, реализующих [открытый веб-интерфейс для .NET (OWIN)](http://owin.org/). Поддержка нескольких конечных точек маркеров полезна при переносе API Azure Active Directory B2C (Azure AD B2C) и их приложений из одного домена в другой. Например, от *Login.microsoftonline.com* к *b2clogin.com* или к [пользовательскому домену](custom-domain.md).

Добавив поддержку в API для приема маркеров, выпущенных b2clogin.com, login.microsoftonline.com или настраиваемого домена, можно выполнить миграцию веб-приложений в промежуточном виде перед удалением поддержки маркеров, выпущенных login.microsoftonline.com, из API.

В следующих разделах представлен пример включения нескольких издателей в веб-интерфейсе API, который использует компоненты по промежуточного слоя [Microsoft OWIN][katana] (Katana). Хотя примеры кода специфичны для по промежуточного слоя Microsoft OWIN, общий метод должен быть применим к другим библиотекам OWIN.

## <a name="prerequisites"></a>Предварительные требования

Перед продолжением действий, описанных в этой статье, необходимо выполнить следующие Azure AD B2C ресурсы.

* [Потоки пользователей](tutorial-create-user-flows.md) или [пользовательские политики](custom-policy-get-started.md) , созданные в клиенте

## <a name="get-token-issuer-endpoints"></a>Получение конечных точек издателя маркера

Сначала необходимо получить URI конечной точки издателя маркера для каждого издателя, который требуется поддерживать в API. Чтобы получить конечные точки *b2clogin.com* и *Login.microsoftonline.com* , поддерживаемые клиентом Azure AD B2C, выполните следующую процедуру в портал Azure.

Для начала выберите один из существующих потоков пользователя:

1. Перейдите к клиенту Azure AD B2C в [портал Azure](https://portal.azure.com)
1. В разделе **политики** выберите **пользовательские потоки (политики)** .
1. Выберите существующую политику, например *B2C_1_signupsignin1*, а затем выберите пункт **запустить поток пользователя** .
1. Под заголовком **Запуск потока пользователя** в верхней части страницы выберите гиперссылку, чтобы открыть конечную точку обнаружения OpenID Connect Connect для этого потока пользователя.

    ![Ссылка на известный URI на странице "Запустить сейчас" на портале Azure](media/multi-token-endpoints/portal-01-policy-link.png)

1. На странице, которая открывается в браузере, запишите значение `issuer`, например:

    `https://your-b2c-tenant.b2clogin.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/v2.0/`

1. Используйте раскрывающийся список **Выбор домена** , чтобы выбрать другой домен, а затем снова выполните предыдущие два шага и запишите его `issuer` значение.

Теперь у вас должны быть записаны два URI, которые похожи на:

```
https://login.microsoftonline.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/v2.0/
https://your-b2c-tenant.b2clogin.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/v2.0/
```

### <a name="custom-policies"></a>Пользовательские политики

При наличии пользовательских политик вместо потоков пользователей можно использовать аналогичную процедуру для получения URI издателя.

1. Перейдите к вашему клиенту Azure AD B2C
1. Выбор **инфраструктуры процедур идентификации**
1. Выберите одну из политик проверяющей стороны, например *B2C_1A_signup_signin*
1. Используйте раскрывающийся список **Выбор домена** , чтобы выбрать домен, например *yourtenant.b2clogin.com*
1. Выберите гиперссылку, отображаемую в разделе **OpenID Connect подключение конечной точки обнаружения**
1. Запишите `issuer` значение
1. Выполните шаги 4-6 для другого домена, например *Login.microsoftonline.com*

## <a name="get-the-sample-code"></a>Получение примера кода

Теперь, когда у вас есть оба URI конечной точки маркера, необходимо обновить код, чтобы указать, что обе конечные точки являются действительными поставщиками. Чтобы проанализировать пример, скачайте или клонировать пример приложения, а затем обновите пример для поддержки обеих конечных точек в качестве действительных поставщиков.

Скачайте архив: [active-directory-b2c-dotnet-webapp-and-webapi-master.zip][sample-archive]

```
git clone https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi.git
```

## <a name="enable-multiple-issuers-in-web-api"></a>Включение нескольких издателей в веб-API

В этом разделе вы обновите код, чтобы указать, что обе конечные точки поставщика маркеров являются допустимыми.

1. Открытие решения **B2C-WebAPI-DotNet. sln** в Visual Studio
1. В проекте **TaskService** откройте файл *TaskService \\ App_Start * * \\ Startup.auth.CS** * в редакторе.
1. Добавьте следующую директиву `using` в начало файла.

    `using System.Collections.Generic;`
1. Добавьте [`ValidIssuers`][validissuers] свойство в [`TokenValidationParameters`][tokenvalidationparameters] Определение и укажите оба URI, записанные в предыдущем разделе:

    ```csharp
    TokenValidationParameters tvps = new TokenValidationParameters
    {
        // Accept only those tokens where the audience of the token is equal to the client ID of this app
        ValidAudience = ClientId,
        AuthenticationType = Startup.DefaultPolicy,
        ValidIssuers = new List<string> {
            "https://login.microsoftonline.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/v2.0/",
            "https://{your-b2c-tenant}.b2clogin.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/v2.0/"//,
            //"https://your-custom-domain/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/v2.0/"
        }
    };
    ```

`TokenValidationParameters` предоставляется MSAL.NET и используется по промежуточного слоя OWIN в следующем разделе кода в *Startup.auth.CS*. Если указано несколько допустимых издателей, конвейер приложения OWIN учитывает, что обе конечные точки маркеров являются действительными поставщиками.

```csharp
app.UseOAuthBearerAuthentication(new OAuthBearerAuthenticationOptions
{
    // This SecurityTokenProvider fetches the Azure AD B2C metadata &  from the OpenID Connect metadata endpoint
    AccessTokenFormat = new JwtFormat(tvps, new tCachingSecurityTokenProvider(String.Format(AadInstance, ultPolicy)))
});
```

Как упоминалось ранее, другие библиотеки OWIN обычно предоставляют аналогичные средства для поддержки нескольких издателей. Хотя предоставление примеров для каждой библиотеки выходит за рамки этой статьи, для большинства библиотек можно использовать аналогичную методику.

## <a name="switch-endpoints-in-web-app"></a>Переключение конечных точек в веб-приложении

Теперь, когда оба URI поддерживаются веб-API, вам потребуется обновить веб-приложение, чтобы оно получало маркеры от конечной точки b2clogin.com.

Например, можно настроить пример веб-приложения для использования новой конечной точки, изменив `ida:AadInstance` значение в *файле TaskWebApp \\ **Web.config** _ проекта _* TaskWebApp * *.

Измените `ida:AadInstance` значение в *Web.config* TaskWebApp, чтобы оно ссылалось `{your-b2c-tenant-name}.b2clogin.com` вместо `login.microsoftonline.com` .

Перед следующей операцией.

```xml
<!-- Old value -->
<add key="ida:AadInstance" value="https://login.microsoftonline.com/tfp/{0}/{1}" />
```

После (замените `{your-b2c-tenant}` именем своего клиента B2C):

```xml
<!-- New value -->
<add key="ida:AadInstance" value="https://{your-b2c-tenant}.b2clogin.com/tfp/{0}/{1}" />
```

Когда строки конечной точки создаются во время выполнения веб-приложения, конечные точки на основе b2clogin.com используются при запросе токенов.

При использовании пользовательского домена:

```xml
<!-- Custom domain -->
<add key="ida:AadInstance" value="https://custom-domain/{0}/{1}" />
```

## <a name="next-steps"></a>Дальнейшие действия

В этой статье представлен метод настройки веб-API, реализующий по промежуточного слоя Microsoft OWIN (Katana), для приема маркеров из нескольких конечных точек поставщика. Как вы могли заметить, в файлах *Web.Config* и в проектах TaskService и TaskWebApp есть несколько других строк, которые необходимо изменить, если требуется собрать и запустить эти проекты для собственного клиента. Вы можете соответствующим образом изменить проекты, если вы хотите увидеть их в действии, но полное пошаговое руководство выходит за рамки этой статьи.

Дополнительные сведения о различных типах маркеров безопасности, порожденных Azure AD B2C, см. [в разделе Обзор маркеров в Azure Active Directory B2C](tokens-overview.md).

<!-- LINKS - External -->
[sample-archive]: https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi/archive/master.zip
[sample-repo]: https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi

<!-- LINKS - Internal -->
[katana]: /aspnet/aspnet/overview/owin-and-katana/
[validissuers]: /dotnet/api/microsoft.identitymodel.tokens.tokenvalidationparameters.validissuers
[tokenvalidationparameters]: /dotnet/api/microsoft.identitymodel.tokens.tokenvalidationparameters
