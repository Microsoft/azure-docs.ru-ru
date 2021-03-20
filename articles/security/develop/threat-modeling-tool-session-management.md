---
title: Управление сеансами. Средство моделирования угроз Microsoft Azure | Документация Майкрософт
description: Узнайте о предотвращении управления сеансами для угроз, доступных в Threat Modeling Tool. См. сведения об устранении рисков и просмотр примеров кода.
services: security
documentationcenter: na
author: jegeib
manager: jegeib
editor: jegeib
ms.assetid: na
ms.service: security
ms.subservice: security-develop
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/07/2017
ms.author: jegeib
ms.custom: has-adal-ref, devx-track-js, devx-track-csharp
ms.openlocfilehash: a1f4d4a3bb78da82753d651e1a73cf244096d5df
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94518107"
---
# <a name="security-frame-session-management"></a>Механизм безопасности. Управление сеансами
| Продукт или служба | Статья |
| --------------- | ------- |
| **Azure AD**    | <ul><li>[Реализуйте надлежащее событие выхода с помощью методов ADAL при использовании Azure AD](#logout-adal)</li></ul> |
| **Устройство IoT** | <ul><li>[Используйте токены SaS с ограниченным временем существования](#finite-tokens)</li></ul> |
| **Azure DocumentDB** | <ul><li>[Используйте токены ресурсов с минимальным временем существования](#resource-tokens)</li></ul> |
| **ADFS** | <ul><li>[Реализуйте надлежащее событие выхода с помощью методов WsFederation при использовании ADFS](#wsfederation-logout)</li></ul> |
| **Сервер удостоверений** | <ul><li>[Реализация правильного выхода при использовании сервера удостоверений](#proper-logout)</li></ul> |
| **Веб-приложение** | <ul><li>[Используйте защищенные файлы cookie в приложениях, использующих протокол HTTPS](#https-secure-cookies)</li><li>[Укажите атрибут httpOnly в определении файла cookie всех приложений на основе протокола HTTP](#cookie-definition)</li><li>[Уменьшите риск атак с подделкой межсайтовых запросов на веб-страницах ASP.NET](#csrf-asp)</li><li>[Настройка сеанса на неактивное время](#inactivity-lifetime)</li><li>[Реализация правильного выхода из приложения](#proper-app-logout)</li></ul> |
| **Веб-интерфейс API** | <ul><li>[Уменьшите риск атак с подделкой межсайтовых запросов в веб-интерфейсах API ASP.NET](#csrf-api)</li></ul> |

## <a name="implement-proper-logout-using-adal-methods-when-using-azure-ad"></a><a id="logout-adal"></a>Реализуйте надлежащее событие выхода с помощью методов ADAL при использовании Azure AD

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Azure AD | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | Универсальный шаблон |
| **Атрибуты**              | Недоступно  |
| **Справочные материалы**              | Недоступно  |
| **Шаги** | Если приложение использует маркер доступа, выданный Azure AD, необходимо вызывать обработчик событий выхода. |

### <a name="example"></a>Пример
```csharp
HttpContext.GetOwinContext().Authentication.SignOut(OpenIdConnectAuthenticationDefaults.AuthenticationType, CookieAuthenticationDefaults.AuthenticationType)
```

### <a name="example"></a>Пример
Кроме того, также необходимо уничтожить сеанс пользователя путем вызова метода Session.Abandon(). Ниже приведен метод безопасной реализации события выхода пользователя.
```csharp
    [HttpPost]
        [ValidateAntiForgeryToken]
        public void LogOff()
        {
            string userObjectID = ClaimsPrincipal.Current.FindFirst("http://schemas.microsoft.com/identity/claims/objectidentifier").Value;
            AuthenticationContext authContext = new AuthenticationContext(Authority + TenantId, new NaiveSessionCache(userObjectID));
            authContext.TokenCache.Clear();
            Session.Clear();
            Session.Abandon();
            Response.SetCookie(new HttpCookie("ASP.NET_SessionId", string.Empty));
            HttpContext.GetOwinContext().Authentication.SignOut(
                OpenIdConnectAuthenticationDefaults.AuthenticationType,
                CookieAuthenticationDefaults.AuthenticationType);
        } 
```

## <a name="use-finite-lifetimes-for-generated-sas-tokens"></a><a id="finite-tokens"></a>Используйте токены SaS с ограниченным временем существования

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Устройства Интернета вещей | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | Универсальный шаблон |
| **Атрибуты**              | Недоступно  |
| **Справочные материалы**              | Недоступно  |
| **Шаги** | Токены SaS, созданные для проверки подлинности Центра Интернета вещей Azure, должны иметь ограниченный срок действия. Задайте минимальный срок действия, чтобы ограничить срок повторного воспроизведения токенов SaS в случае их компрометирования.|

## <a name="use-minimum-token-lifetimes-for-generated-resource-tokens"></a><a id="resource-tokens"></a>Используйте токены ресурсов с минимальным временем существования

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Azure DocumentDB | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | Универсальный шаблон |
| **Атрибуты**              | Недоступно  |
| **Справочные материалы**              | Недоступно  |
| **Шаги** | Задайте минимальное время существования токенов ресурсов. Допустимый интервал времени по умолчанию для маркеров ресурсов составляет 1 час.|

## <a name="implement-proper-logout-using-wsfederation-methods-when-using-adfs"></a><a id="wsfederation-logout"></a>Реализуйте надлежащее событие выхода с помощью методов WsFederation при использовании ADFS

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | ADFS | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | Универсальный шаблон |
| **Атрибуты**              | Недоступно  |
| **Справочные материалы**              | Недоступно  |
| **Шаги** | Если приложение использует токен службы токенов безопасности, выданный ADFS, чтобы выйти из системы пользователя, обработчик событий выхода должен вызвать метод WSFederationAuthenticationModule.FederatedSignOut(). Кроме того, необходимо уничтожить текущий сеанс, а также сбросить и обнулить значение токена сеанса.|

### <a name="example"></a>Пример
```csharp
        [HttpPost, ValidateAntiForgeryToken]
        [Authorization]
        public ActionResult SignOut(string redirectUrl)
        {
            if (!this.User.Identity.IsAuthenticated)
            {
                return this.View("LogOff", null);
            }

            // Removes the user profile.
            this.Session.Clear();
            this.Session.Abandon();
            HttpContext.Current.Response.Cookies.Add(new System.Web.HttpCookie("ASP.NET_SessionId", string.Empty)
                {
                    Expires = DateTime.Now.AddDays(-1D),
                    Secure = true,
                    HttpOnly = true
                });

            // Signs out at the specified security token service (STS) by using the WS-Federation protocol.
            Uri signOutUrl = new Uri(FederatedAuthentication.WSFederationAuthenticationModule.Issuer);
            Uri replyUrl = new Uri(FederatedAuthentication.WSFederationAuthenticationModule.Realm);
            if (!string.IsNullOrEmpty(redirectUrl))
            {
                replyUrl = new Uri(FederatedAuthentication.WSFederationAuthenticationModule.Realm + redirectUrl);
            }
           //     Signs out of the current session and raises the appropriate events.
            var authModule = FederatedAuthentication.WSFederationAuthenticationModule;
            authModule.SignOut(false);
        //     Signs out at the specified security token service (STS) by using the WS-Federation
        //     protocol.            
            WSFederationAuthenticationModule.FederatedSignOut(signOutUrl, replyUrl);
            return new RedirectResult(redirectUrl);
        }
```

## <a name="implement-proper-logout-when-using-identity-server"></a><a id="proper-logout"></a>Реализуйте надлежащее событие выхода при использовании сервера удостоверений

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Сервер удостоверений | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | Универсальный шаблон |
| **Атрибуты**              | Недоступно  |
| **Справочные материалы**              | [Сведения о федеративном выходе на платформе IdentityServer3](https://identityserver.github.io/Documentation/docsv2/advanced/federated-signout.html) |
| **Шаги** | IdentityServer поддерживает возможность создавать федерацию с внешними поставщиками удостоверений. Когда пользователь выходит из вышестоящего поставщика удостоверений, в зависимости от используемого протокола можно получить уведомление при выходе пользователя из службы. Она позволяет IdentityServer уведомлять своих клиентов, чтобы они также могли подписывать пользователя. Сведения о реализации см. в документации в разделе "ссылки".|

## <a name="applications-available-over-https-must-use-secure-cookies"></a><a id="https-secure-cookies"></a>Используйте защищенные файлы cookie в приложениях, использующих протокол HTTPS

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Веб-приложение | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | Универсальный шаблон |
| **Атрибуты**              | EnvironmentType: OnPrem |
| **Справочные материалы**              | [Элемент httpCookies (схема параметров ASP.NET)](/previous-versions/dotnet/netframework-4.0/ms228262(v=vs.100)), [Свойство HttpCookie.Secure](/dotnet/api/system.web.httpcookie.secure) |
| **Шаги** | Как правило, файлы cookie доступны только в домене, для которого они были заданы. К сожалению, определение "домен" не включает в себя протокол, и поэтому файлы cookie, созданные по протоколу HTTPS, доступны по протоколу HTTP. Атрибут secure указывает браузеру, что файлы cookie доступны только по протоколу HTTPS. Убедитесь, что во всех этих файлах, заданных по протоколу HTTPS, используется атрибут **secure**. Это требование можно выполнить в файле web.config, задав атрибуту requireSSL значение true. Мы рекомендуем использовать этот подход, так как он позволяет применить атрибут **secure** для всех текущих и будущих файлов cookie, не внося дополнительные изменения в код.|

### <a name="example"></a>Пример
```csharp
<configuration>
  <system.web>
    <httpCookies requireSSL="true"/>
  </system.web>
</configuration>
```
Этот параметр применяется, даже если доступ к приложению осуществляется по протоколу HTTP. Если для доступа к приложению используется протокол HTTP, то этот параметр приводит к сбою приложения, так как из-за атрибута secure браузер не сможет возвращать файлы cookie в приложение.

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Веб-приложение | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | Веб-формы, MVC 5 |
| **Атрибуты**              | EnvironmentType: OnPrem |
| **Справочные материалы**              | Недоступно  |
| **Шаги** | Если веб-приложение выступает проверяющей стороной, а в качестве сервера ADFS используется IdP, атрибут secure в токене FedAuth можно настроить, задав в разделе `system.identityModel.services` файла web.config для параметра requireSSL значение true.|

### <a name="example"></a>Пример
```csharp
  <system.identityModel.services>
    <federationConfiguration>
      <!-- Set requireSsl=true; domain=application domain name used by FedAuth cookies (Ex: .gdinfra.com); -->
      <cookieHandler requireSsl="true" persistentSessionLifetime="0.0:20:0" />
    ....  
    </federationConfiguration>
  </system.identityModel.services>
```

## <a name="all-http-based-application-should-specify-http-only-for-cookie-definition"></a><a id="cookie-definition"></a>Укажите атрибут httpOnly в определении файла cookie всех приложений на основе протокола HTTP

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Веб-приложение | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | Универсальный шаблон |
| **Атрибуты**              | Недоступно  |
| **Справочные материалы**              | [Сведения об атрибуте secure в файлах cookie](https://en.wikipedia.org/wiki/HTTP_cookie#Secure_cookie) |
| **Шаги** | Чтобы устранить угрозу раскрытия информации при атаках с использованием межсайтового сценария, в файлы cookie можно добавить новый атрибут httpOnly, поддерживаемый во всех основных браузерах. Атрибут делает файл cookie недоступным для сценария. С помощью атрибута httpOnly веб-приложение снижает возможность кражи конфиденциальной информации, содержащейся в файле cookie, через сценарий и ее отправки злоумышленнику. |

### <a name="example"></a>Пример
Во всех приложениях на основе HTTP, использующих файлы cookie, необходимо определить атрибут httpOnly в определении файла cookie. Это можно сделать, внедрив в файл web.config следующую конфигурацию:
```XML
<system.web>
.
.
   <httpCookies requireSSL="false" httpOnlyCookies="true"/>
.
.
</system.web>
```

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Веб-приложение | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | веб-формы |
| **Атрибуты**              | Недоступно  |
| **Справочные материалы**              | [Свойство FormsAuthentication.RequireSSL](/dotnet/api/system.web.security.formsauthentication.requiressl) |
| **Шаги** | Значение свойства RequireSSL задается в файле конфигурации web.config для приложения ASP.NET с помощью атрибута requireSSL. Вы можете указать в файле Web.config приложения ASP.NET, требуется ли протокол TLS, ранее известный как SSL (SSL), для возврата файла cookie проверки подлинности форм на сервер путем установки атрибута requireSSL.|

### <a name="example"></a>Пример 
Следующий пример кода задает атрибут requireSSL в файле web.config.
```XML
<authentication mode="Forms">
  <forms loginUrl="member_login.aspx" cookieless="UseCookies" requireSSL="true"/>
</authentication>
```

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Веб-приложение | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | MVC5 |
| **Атрибуты**              | EnvironmentType: OnPrem |
| **Справочные материалы**              | [Запись блога о конфигурации Windows Identity Foundation (WIF). Часть 2](/archive/blogs/alikl/windows-identity-foundation-wif-configuration-part-ii-cookiehandler-chunkedcookiehandler-customcookiehandler) |
| **Шаги** | Чтобы задать атрибут httpOnly для файлов cookie FedAuth, атрибут hideFromCsript должен иметь значение true. |

### <a name="example"></a>Пример
Ниже приведена правильная конфигурация.
```XML
<federatedAuthentication>
<cookieHandler mode="Custom"
                       hideFromScript="true"
                       name="FedAuth"
                       path="/"
                       requireSsl="true"
                       persistentSessionLifetime="25">
</cookieHandler>
</federatedAuthentication>
```

## <a name="mitigate-against-cross-site-request-forgery-csrf-attacks-on-aspnet-web-pages"></a><a id="csrf-asp"></a>Уменьшите риск атак с подделкой межсайтовых запросов на веб-страницах ASP.NET

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Веб-приложение | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | Универсальный шаблон |
| **Атрибуты**              | Недоступно  |
| **Справочные материалы**              | Недоступно  |
| **Шаги** | Подделка межсайтовых запросов (CSRF или XSRF) — это тип атак, при которых злоумышленник может выполнять действия в контексте безопасности разных пользовательских активных сеансов на веб-сайте. Целью является изменение или удаление содержимого, если аутентификация полученных на целевом веб-сайте запросов выполняется исключительно с помощью файлов cookie сеанса. Злоумышленник может воспользоваться этой уязвимостью и внедрить в код сайта, на который выполнил вход пользователь, команду, которая перенаправляет пользователя на другой URL-адрес. Это можно сделать несколькими способами, например, разместив другой веб-сайт, который скачивает ресурсы с уязвимого сервера, или заставив пользователя щелкнуть ссылку. Чтобы предотвратить эту атаку, сервер должен отправлять дополнительный токен клиенту, добавлять его во все последующие запросы клиента, а затем проверять наличие этого токена, относящегося к текущему сеансу, во всех будущих запросах, например с помощью метода AntiForgeryToken или ViewState ASP.NET. |

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Веб-приложение | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | MVC 5, MVC 6 |
| **Атрибуты**              | Недоступно  |
| **Справочные материалы**              | [Предотвращение атак XSRF и CSRF в ASP.NET MVC и на веб-страницах](https://www.asp.net/mvc/overview/security/xsrfcsrf-prevention-in-aspnet-mvc-and-web-pages) |
| **Шаги** | Формы Anti-CSRF и ASP.NET MVC. Используйте в представлениях вспомогательный метод `AntiForgeryToken`. Вставьте `Html.AntiForgeryToken()` в форму.|

### <a name="example"></a>Пример
```csharp
@using (Html.BeginForm("UserProfile", "SubmitUpdate")) { 
    @Html.ValidationSummary(true) 
    @Html.AntiForgeryToken()
    <fieldset> 
```

### <a name="example"></a>Пример
```csharp
<form action="/UserProfile/SubmitUpdate" method="post">
    <input name="__RequestVerificationToken" type="hidden" value="saTFWpkKN0BYazFtN6c4YbZAmsEwG0srqlUqqloi/fVgeV2ciIFVmelvzwRZpArs" />
    <!-- rest of form goes here -->
</form>
```

### <a name="example"></a>Пример
В то же время Html.AntiForgeryToken() предоставляет посетителю файл cookie __RequestVerificationToken с таким же значением, как и случайное скрытое значение, показанное выше. Затем добавьте в целевой метод действия фильтр [ValidateAntiForgeryToken], чтобы проверить входящую форму POST. Пример:
```
[ValidateAntiForgeryToken]
public ViewResult SubmitUpdate()
{
// ... etc.
}
```
Фильтр авторизации, проверяющий, что:
* Входящий запрос имеет файл cookie с именем __RequestVerificationToken.
* Входящий запрос имеет запись `Request.Form` с именем __RequestVerificationToken.
* Эти значения файла cookie и `Request.Form` совпадают. При условии, что все параметры заданы правильно, запрос выполняется в обычном режиме. В противном случае проверка подлинности завершается ошибкой и отображается следующее сообщение: "Обязательный маркер против подделки не был предоставлен или был недопустимым". 

### <a name="example"></a>Пример
Формы Anti-CSRF и AJAX. Токен формы может вызывать проблемы при выполнении запросов AJAX. Это связано с тем, что этот запрос может отправлять данные JSON, а не данные HTML-формы. В этом случае можно отправлять токены в настраиваемый заголовок HTTP. Следующий код создает токены с использованием синтаксиса Razor, а затем добавляет их в запрос AJAX. 
```csharp
<script>
    @functions{
        public string TokenHeaderValue()
        {
            string cookieToken, formToken;
            AntiForgery.GetTokens(null, out cookieToken, out formToken);
            return cookieToken + ":" + formToken;                
        }
    }

    $.ajax("api/values", {
        type: "post",
        contentType: "application/json",
        data: {  }, // JSON data goes here
        dataType: "json",
        headers: {
            'RequestVerificationToken': '@TokenHeaderValue()'
        }
    });
</script>
```

### <a name="example"></a>Пример
При обработке запроса извлеките токены из его заголовка. Затем вызовите метод AntiForgery.Validate для проверки этих токенов. Если токены недопустимые, метод AntiForgery.Validate вызывает исключение.
```csharp
void ValidateRequestHeader(HttpRequestMessage request)
{
    string cookieToken = "";
    string formToken = "";

    IEnumerable<string> tokenHeaders;
    if (request.Headers.TryGetValues("RequestVerificationToken", out tokenHeaders))
    {
        string[] tokens = tokenHeaders.First().Split(':');
        if (tokens.Length == 2)
        {
            cookieToken = tokens[0].Trim();
            formToken = tokens[1].Trim();
        }
    }
    AntiForgery.Validate(cookieToken, formToken);
}
```

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Веб-приложение | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | веб-формы |
| **Атрибуты**              | Недоступно  |
| **Справочные материалы**              | [Сведения о предотвращении сетевых атак с помощью встроенных функций ASP.NET](/previous-versions/dotnet/articles/ms972969(v=msdn.10)#securitybarriers_topic2) |
| **Шаги** | Подделку межсайтовых запросов в приложениях на основе веб-форм можно предотвратить, добавив параметр ViewStateUserKey в случайную строку, уникальную для каждого пользователя, например в строку с идентификатором пользователя, а еще лучше с идентификатором сеанса. По разным техническим и социальным причинам идентификатор сеанса подходит гораздо лучше, так как его нельзя предсказать, время его ожидания истекает и он уникальный для каждого пользователя.|

### <a name="example"></a>Пример
Ниже приведен код, который необходимо добавить на всех страницах.
```csharp
void Page_Init (object sender, EventArgs e) {
   ViewStateUserKey = Session.SessionID;
   :
}
```

## <a name="set-up-session-for-inactivity-lifetime"></a><a id="inactivity-lifetime"></a>Настройте период бездействия сеанса

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Веб-приложение | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | Универсальный шаблон |
| **Атрибуты**              | Недоступно  |
| **Справочные материалы**              | [Свойство HttpSessionState.Timeout](/dotnet/api/system.web.sessionstate.httpsessionstate.timeout) |
| **Шаги** | Время ожидания сеанса представляет событие, возникающее, когда пользователь не выполняет никаких действий на веб-сайте в течение интервала (определенного веб-сервером). Это событие на стороне сервера изменяет состояние сеанса пользователя на "Недопустимый" (например, "больше не используется") и дает указание веб-серверу уничтожить его (удалить все данные сеанса). Пример кода ниже задает для атрибута времени ожидания сеанса значение 15 минут в файле web.config.|

### <a name="example"></a>Пример
```XML 
<configuration>
  <system.web>
    <sessionState mode="InProc" cookieless="true" timeout="15" />
  </system.web>
</configuration>
```

## <a name="enable-threat-detection-on-azure-sql"></a><a id="threat-detection"></a>Включение обнаружения угроз в Azure SQL

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Веб-приложение | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | веб-формы |
| **Атрибуты**              | Недоступно  |
| **Справочные материалы**              | [Элемент Forms для проверки подлинности (схема параметров ASP.NET)](/previous-versions/dotnet/netframework-4.0/1d3t3c61(v=vs.100)) |
| **Шаги** | Задайте для параметра времени ожидания билета проверки подлинности (файла cookie) значение, не превышающее 15 минут.|

### <a name="example"></a>Пример
```XML
<forms  name=".ASPXAUTH" loginUrl="login.aspx"  defaultUrl="default.aspx" protection="All" timeout="15" path="/" requireSSL="true" slidingExpiration="true"/>
</forms>
```

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Веб-приложение | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | Веб-формы, MVC 5 |
| **Атрибуты**              | EnvironmentType: OnPrem |
| **Справочные материалы**              | [asdeqa](https://skf.azurewebsites.net/Mitigations/Details/wefr) |
| **Шаги** | Когда веб-приложение является проверяющей стороной, а ADFS выступает службой токенов безопасности, время существования файлов cookie проверки подлинности (токенов FedAuth) задается с использованием следующей конфигурации в файле web.config:|

### <a name="example"></a>Пример
```XML
  <system.identityModel.services>
    <federationConfiguration>
      <!-- Set requireSsl=true; domain=application domain name used by FedAuth cookies (Ex: .gdinfra.com); -->
      <cookieHandler requireSsl="true" persistentSessionLifetime="0.0:15:0" />
      <!-- Set requireHttps=true; -->
      <wsFederation passiveRedirectEnabled="true" issuer="http://localhost:39529/" realm="https://localhost:44302/" reply="https://localhost:44302/" requireHttps="true"/>
      <!--
      Use the code below to enable encryption-decryption of claims received from ADFS. Thumbprint value varies based on the certificate being used.
      <serviceCertificate>
        <certificateReference findValue="4FBBBA33A1D11A9022A5BF3492FF83320007686A" storeLocation="LocalMachine" storeName="My" x509FindType="FindByThumbprint" />
      </serviceCertificate>
      -->
    </federationConfiguration>
  </system.identityModel.services>
```

### <a name="example"></a>Пример
Время существования токена утверждений, выданного ADFS, также должно составлять 15 минут. Чтобы задать это значение, выполните на сервере ADFS следующий командлет PowerShell:
```csharp
Set-ADFSRelyingPartyTrust -TargetName "<RelyingPartyWebApp>" -ClaimsProviderName @("Active Directory") -TokenLifetime 15 -AlwaysRequireAuthentication $true
```

## <a name="implement-proper-logout-from-the-application"></a><a id="proper-app-logout"></a>Реализуйте правильный выход из приложения

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Веб-приложение | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | Универсальный шаблон |
| **Атрибуты**              | Недоступно  |
| **Справочные материалы**              | Недоступно  |
| **Шаги** | После нажатия кнопки выхода приложение должно правильно завершить этот процесс. То есть оно должно уничтожить сеанс пользователя, а также сбросить и обнулить значение файла cookie сеанса и файла cookie проверки подлинности. Кроме того, когда к идентификатору пользователя привязаны несколько сеансов, они должны завершаться все вместе на стороне сервера во время ожидания или выхода. Наконец, убедитесь, что функция выхода доступна на каждой странице. |

## <a name="mitigate-against-cross-site-request-forgery-csrf-attacks-on-aspnet-web-apis"></a><a id="csrf-api"></a>Уменьшите риск атак с подделкой межсайтовых запросов в веб-интерфейсах API ASP.NET

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Веб-API | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | Универсальный шаблон |
| **Атрибуты**              | Недоступно  |
| **Справочные материалы**              | Недоступно  |
| **Шаги** | Подделка межсайтовых запросов (CSRF или XSRF) — это тип атак, при которых злоумышленник может выполнять действия в контексте безопасности разных пользовательских активных сеансов на веб-сайте. Целью является изменение или удаление содержимого, если аутентификация полученных на целевом веб-сайте запросов выполняется исключительно с помощью файлов cookie сеанса. Злоумышленник может воспользоваться этой уязвимостью и внедрить в код сайта, на который выполнил вход пользователь, команду, которая перенаправляет пользователя на другой URL-адрес. Это можно сделать несколькими способами, например, разместив другой веб-сайт, который скачивает ресурсы с уязвимого сервера, или заставив пользователя щелкнуть ссылку. Чтобы предотвратить эту атаку, сервер должен отправлять дополнительный токен клиенту, добавлять его во все последующие запросы клиента, а затем проверять наличие этого токена, относящегося к текущему сеансу, во всех будущих запросах, например с помощью метода AntiForgeryToken или ViewState ASP.NET. |

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Веб-API | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | MVC 5, MVC 6 |
| **Атрибуты**              | Недоступно  |
| **Справочные материалы**              | [Статья о предотвращении риска атак с подделкой межсайтовых запросов в веб-интерфейсах API ASP.NET](https://www.asp.net/web-api/overview/security/preventing-cross-site-request-forgery-csrf-attacks) |
| **Шаги** | Формы Anti-CSRF и AJAX. Токен формы может вызывать проблемы при выполнении запросов AJAX. Это связано с тем, что этот запрос может отправлять данные JSON, а не данные HTML-формы. В этом случае можно отправлять токены в настраиваемый заголовок HTTP. Следующий код создает токены с использованием синтаксиса Razor, а затем добавляет их в запрос AJAX. |

### <a name="example"></a>Пример
```Javascript
<script>
    @functions{
        public string TokenHeaderValue()
        {
            string cookieToken, formToken;
            AntiForgery.GetTokens(null, out cookieToken, out formToken);
            return cookieToken + ":" + formToken;                
        }
    }
    $.ajax("api/values", {
        type: "post",
        contentType: "application/json",
        data: {  }, // JSON data goes here
        dataType: "json",
        headers: {
            'RequestVerificationToken': '@TokenHeaderValue()'
        }
    });
</script>
```

### <a name="example"></a>Пример
При обработке запроса извлеките токены из его заголовка. Затем вызовите метод AntiForgery.Validate для проверки этих токенов. Если токены недопустимые, метод AntiForgery.Validate вызывает исключение.
```csharp
void ValidateRequestHeader(HttpRequestMessage request)
{
    string cookieToken = "";
    string formToken = "";

    IEnumerable<string> tokenHeaders;
    if (request.Headers.TryGetValues("RequestVerificationToken", out tokenHeaders))
    {
        string[] tokens = tokenHeaders.First().Split(':');
        if (tokens.Length == 2)
        {
            cookieToken = tokens[0].Trim();
            formToken = tokens[1].Trim();
        }
    }
    AntiForgery.Validate(cookieToken, formToken);
}
```

### <a name="example"></a>Пример
Формы Anti-CSRF и ASP.NET MVC. Используйте в представлениях вспомогательный метод AntiForgeryToken. Вставьте Html.AntiForgeryToken() в форму.
```csharp
@using (Html.BeginForm("UserProfile", "SubmitUpdate")) { 
    @Html.ValidationSummary(true) 
    @Html.AntiForgeryToken()
    <fieldset> 
}
```

### <a name="example"></a>Пример
Выходные данные приведенного выше примера будут примерно следующие:
```csharp
<form action="/UserProfile/SubmitUpdate" method="post">
    <input name="__RequestVerificationToken" type="hidden" value="saTFWpkKN0BYazFtN6c4YbZAmsEwG0srqlUqqloi/fVgeV2ciIFVmelvzwRZpArs" />
    <!-- rest of form goes here -->
</form>
```

### <a name="example"></a>Пример
В то же время Html.AntiForgeryToken() предоставляет посетителю файл cookie __RequestVerificationToken с таким же значением, как и случайное скрытое значение, показанное выше. Затем добавьте в целевой метод действия фильтр [ValidateAntiForgeryToken], чтобы проверить входящую форму POST. Пример:
```
[ValidateAntiForgeryToken]
public ViewResult SubmitUpdate()
{
// ... etc.
}
```
Фильтр авторизации, проверяющий, что:
* Входящий запрос имеет файл cookie с именем __RequestVerificationToken.
* Входящий запрос имеет запись `Request.Form` с именем __RequestVerificationToken.
* Эти значения файла cookie и `Request.Form` совпадают. При условии, что все параметры заданы правильно, запрос выполняется в обычном режиме. В противном случае проверка подлинности завершается ошибкой и отображается следующее сообщение: "Обязательный маркер против подделки не был предоставлен или был недопустимым".

| Title                   | Сведения      |
| ----------------------- | ------------ |
| **Компонент**               | Веб-API | 
| **Этап SDL**               | Сборка |  
| **Применимые технологии** | MVC 5, MVC 6 |
| **Атрибуты**              | Поставщик удостоверений — ADFS или Azure AD |
| **Справочные материалы**              | [Статья об обеспечении безопасности веб-интерфейсов API с помощью отдельной учетной записи и локального имени входа в веб-API ASP.NET 2.2](https://www.asp.net/web-api/overview/security/individual-accounts-in-web-api) |
| **Шаги** | Если для защиты веб-интерфейса API используется OAuth 2.0, в заголовке запроса авторизации должен находиться токен носителя. Доступ к веб-приложению предоставляется, только если этот токен в запросе действительный. В отличие от проверки подлинности на основе файлов cookie браузеры не добавляют токены носителя к запросам. Клиент, отправивший запрос, должен явным образом прикрепить токен носителя в заголовке запроса. Таким образом, в веб-интерфейсах API ASP.NET, защищенных с помощью OAuth 2.0, токены носителя используются для защиты от подделки межсайтовых запросов. Обратите внимание, что если сегмент MVC приложения использует проверку подлинности на основе форм (то есть использует файлы cookie), в веб-приложении MVC необходимо использовать токены защиты от подделки. |

### <a name="example"></a>Пример
Веб-API должен использовать только токены носителя, а не файлы cookie. Это можно сделать с помощью следующей настройки в методе `WebApiConfig.Register`.

```csharp
config.SuppressDefaultHostAuthentication();
config.Filters.Add(new HostAuthenticationFilter(OAuthDefaults.AuthenticationType));
```

Метод SuppressDefaultHostAuthentication сообщает веб-API, что необходимо игнорировать любые проверки подлинности, которые происходят прежде, чем запрос достигнет конвейера веб-API, либо с помощью службы IIS, либо с помощью ПО промежуточного слоя OWIN. Таким образом можно ограничить веб-API для проверки подлинности только с помощью токенов носителя.