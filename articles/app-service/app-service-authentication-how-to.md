---
title: Расширенное использование AuthN/AuthZ
description: Научитесь настраивать функцию проверки подлинности и авторизации в службе приложений для различных сценариев, а также получать утверждения пользователей и различные токены.
ms.topic: article
ms.date: 07/08/2020
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: fc2916cbccc21262467533b0b497b14f4f4b941c
ms.sourcegitcommit: bb330af42e70e8419996d3cba4acff49d398b399
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105034883"
---
# <a name="advanced-usage-of-authentication-and-authorization-in-azure-app-service"></a>Расширенное использование проверки подлинности и авторизации в Службе приложений Azure

В этой статье показано, как настроить встроенные [проверку подлинности и авторизацию в службе приложений](overview-authentication-authorization.md), а также управлять удостоверениями, используя приложение. 

Чтобы быстро приступить к работе, ознакомьтесь с одним из следующих руководств:

* [Руководство. Сквозная проверка подлинности и авторизация в Службе приложений Azure](tutorial-auth-aad.md)
* [Настройка приложения службы приложений для использования службы входа Azure Active Directory](configure-authentication-provider-aad.md)
* [Как настроить приложение службы приложений для использования имени для входа Facebook](configure-authentication-provider-facebook.md)
* [Настройка приложения для использования имени входа Google](configure-authentication-provider-google.md)
* [Настройка приложения для использования входа по учетной записи Майкрософт](configure-authentication-provider-microsoft.md)
* [Настройка приложения для использования имени входа Twitter](configure-authentication-provider-twitter.md)
* [Настройка приложения для входа с помощью поставщика OpenID Connect Connect (Предварительная версия)](configure-authentication-provider-openid-connect.md)
* [Настройка приложения для входа с использованием входа с помощью Apple (Предварительная версия)](configure-authentication-provider-apple.md)

## <a name="use-multiple-sign-in-providers"></a>Использование нескольких поставщиков входа

На портале нельзя напрямую настроить несколько поставщиков входа для пользователей (например, через Facebook и Twitter). Однако эту функцию можно легко добавить к функциональным возможностям вашего приложения. Для этого необходимо сделать следующее:

Во-первых, на странице **Authentication / Authorization** (Проверка подлинности и авторизация) на портале Azure настройте все поставщики удостоверений, которые нужно включить.

В раскрывающемся списке **Action to take when request is not authenticated** (Предпринимаемое действие, если проверка подлинности для запроса не выполнена) выберите **Разрешить анонимные запросы (нет действия)**.

На странице входа, на панели навигации или в любом другом расположении приложения добавьте ссылку входа для каждого включенного поставщика (`/.auth/login/<provider>`). Пример:

```html
<a href="/.auth/login/aad">Log in with Azure AD</a>
<a href="/.auth/login/microsoftaccount">Log in with Microsoft Account</a>
<a href="/.auth/login/facebook">Log in with Facebook</a>
<a href="/.auth/login/google">Log in with Google</a>
<a href="/.auth/login/twitter">Log in with Twitter</a>
<a href="/.auth/login/apple">Log in with Apple</a>
```

Когда пользователь щелкнет одну из этих ссылок, откроется соответствующая страница для входа.

Чтобы перенаправить пользователя после входа по пользовательскому URL-адресу, используйте параметр строки запроса `post_login_redirect_url` (не следует путать с URI перенаправления в конфигурации поставщика удостоверений). Например, чтобы перенаправить пользователя к `/Home/Index` после входа в систему, используйте следующий код HTML:

```html
<a href="/.auth/login/<provider>?post_login_redirect_url=/Home/Index">Log in</a>
```

## <a name="validate-tokens-from-providers"></a>Проверка токенов от поставщиков

При входе с помощью клиента приложение входит в систему поставщика вручную, а затем отправляет токен проверки подлинности службе приложений для проверки (см. [Поток проверки подлинности](overview-authentication-authorization.md#authentication-flow)). Эта проверка сама по себе не предоставляет вам доступ к требуемым ресурсам приложения, но успешная проверка даст вам токен сеанса, который вы можете использовать для доступа к ресурсам приложений. 

Чтобы проверить токен поставщика, для приложения службы приложений сначала нужно настроить требуемый поставщик. Получив токен проверки подлинности у своего поставщика, во время выполнения отправьте токен по адресу `/.auth/login/<provider>` для проверки. Пример: 

```
POST https://<appname>.azurewebsites.net/.auth/login/aad HTTP/1.1
Content-Type: application/json

{"id_token":"<token>","access_token":"<token>"}
```

Формат токена незначительно отличается в соответствии с поставщиком. Подробности см. в следующей таблице:

| Значение поставщика | Требуется в тексте запроса | Комментарии |
|-|-|-|
| `aad` | `{"access_token":"<access_token>"}` | |
| `microsoftaccount` | `{"access_token":"<token>"}` | Свойство `expires_in` необязательное. <br/>При запросе маркера из служб Live всегда запрашиваются области `wl.basic`. |
| `google` | `{"id_token":"<id_token>"}` | Свойство `authorization_code` необязательное. Если указано, при необходимости оно может сопровождаться свойством `redirect_uri`. |
| `facebook`| `{"access_token":"<user_access_token>"}` | Используйте допустимый [токен доступа пользователя](https://developers.facebook.com/docs/facebook-login/access-tokens) из Facebook. |
| `twitter` | `{"access_token":"<access_token>", "access_token_secret":"<acces_token_secret>"}` | |
| | | |

При успешной проверке токена поставщика API возвращается с `authenticationToken` в тексте ответа, который является вашим токеном сеанса. 

```json
{
    "authenticationToken": "...",
    "user": {
        "userId": "sid:..."
    }
}
```

Получив этот токен сеанса, вы можете получить доступ к защищенным ресурсам приложений, добавив заголовок `X-ZUMO-AUTH` к HTTP-запросам. Пример: 

```
GET https://<appname>.azurewebsites.net/api/products/1
X-ZUMO-AUTH: <authenticationToken_value>
```

## <a name="sign-out-of-a-session"></a>Выход из сеанса

Пользователи могут сделать выход, отправив запрос `GET` в конечную точку `/.auth/logout` приложения. Запрос `GET` выполняет следующие действия:

- Очищает файлы cookie проверки подлинности в текущем сеансе.
- Удаляет текущие маркеры пользователя из хранилища маркеров.
- Выполняет выход в поставщике удостоверений на стороне сервера для Azure Active Directory и Google.

Здесь представлена ссылка для простого выхода на веб-странице:

```html
<a href="/.auth/logout">Sign out</a>
```

После успешного выхода клиент по умолчанию перенаправляется на URL-адрес `/.auth/logout/done`. Можно изменить страницу перенаправления после выхода, добавив параметр запроса `post_logout_redirect_uri`. Пример:

```
GET /.auth/logout?post_logout_redirect_uri=/index.html
```

Рекомендуется [закодировать](https://wikipedia.org/wiki/Percent-encoding) значение `post_logout_redirect_uri`.

При использовании полного URL-адреса он должен размещаться в одном и том же домене или быть настроенным в качестве разрешенного URL-адреса внешнего перенаправления для приложения. В следующем примере показано перенаправление на адрес `https://myexternalurl.com`, который не размещен в одном и том же домене:

```
GET /.auth/logout?post_logout_redirect_uri=https%3A%2F%2Fmyexternalurl.com
```

Выполните следующую команду в [Azure Cloud Shell](../cloud-shell/quickstart.md):

```azurecli-interactive
az webapp auth update --name <app_name> --resource-group <group_name> --allowed-external-redirect-urls "https://myexternalurl.com"
```

## <a name="preserve-url-fragments"></a>Сохранение фрагментов URL-адреса

После входа в приложение пользователи обычно желают быть перенаправленными в один и тот же раздел той же страницы, например `/wiki/Main_Page#SectionZ`. Но так как [фрагменты URL-адреса](https://wikipedia.org/wiki/Fragment_identifier) (например, `#SectionZ`) никогда не отправляются на сервер, они не сохраняются по умолчанию после завершения входа OAuth и перенаправления обратно в приложение. Это неудобно для пользователей, когда им снова нужно перейти в требуемую закладку. Это ограничение распространяется на все решения аутентификации на стороне сервера.

При использовании аутентификации службы приложений можно сохранять фрагменты URL-адресов во время входа OAuth. Чтобы сделать это, установите для параметра приложения `WEBSITE_AUTH_PRESERVE_URL_FRAGMENT` значение `true`. Это можно сделать на [портале Azure](https://portal.azure.com) или просто выполнив следующую команду в [Azure Cloud Shell](../cloud-shell/quickstart.md):

```azurecli-interactive
az webapp config appsettings set --name <app_name> --resource-group <group_name> --settings WEBSITE_AUTH_PRESERVE_URL_FRAGMENT="true"
```

## <a name="access-user-claims"></a>Доступ к утверждениям пользователей

Служба приложений передает утверждения пользователей в приложение с помощью специальных заголовков. Эти заголовки нельзя настроить с помощью внешних запросов. Они присутствуют, только если это установлено службой приложений. Некоторые примеры заголовков включают:

* X-MS-CLIENT-PRINCIPAL-NAME
* X-MS-CLIENT-PRINCIPAL-ID

Сведения из этих заголовков можно получить с помощью кода, написанного на любом языке или в любой платформе. Для приложений ASP.NET 4.6 автоматически настраивается класс **ClaimsPrincipal** с соответствующими значениями. Однако ASP.NET Core не предоставляет по промежуточного слоя для проверки подлинности, которое интегрируется с утверждениями пользователя службы приложений. Обходной путь см. в разделе [максимерауиллер. Azure. AppService. еасяус](https://github.com/MaximRouiller/MaximeRouiller.Azure.AppService.EasyAuth).

Если [хранилище токенов](overview-authentication-authorization.md#token-store) включено для приложения, можно также получить дополнительные сведения о прошедшем проверку подлинности пользователе, вызвав метод `/.auth/me` . Пакеты SDK для серверной части мобильных приложений предоставляют вспомогательные методы для работы с этими данными  Дополнительные сведения см. Дополнительные сведения см. в разделах [Практическое руководство. Использование утверждений аутентификации для таблиц](/previous-versions/azure/app-service-mobile/app-service-mobile-node-backend-how-to-use-server-sdk#howto-tables-getidentity) и [Практическое руководство. Получение сведений о пользователе, прошедшем проверку подлинности](/previous-versions/azure/app-service-mobile/app-service-mobile-dotnet-backend-how-to-use-server-sdk#user-info).

## <a name="retrieve-tokens-in-app-code"></a>Извлечение токенов в коде приложения

Токены, предоставляющиеся поставщиком, вводятся в заголовке запроса из кода сервера, что позволяет легко получить к ним доступ. В следующей таблице показаны возможные заголовки токена.

| Поставщик | Имена заголовков |
|-|-|
| Azure Active Directory | `X-MS-TOKEN-AAD-ID-TOKEN` <br/> `X-MS-TOKEN-AAD-ACCESS-TOKEN` <br/> `X-MS-TOKEN-AAD-EXPIRES-ON`  <br/> `X-MS-TOKEN-AAD-REFRESH-TOKEN` |
| Токен Facebook | `X-MS-TOKEN-FACEBOOK-ACCESS-TOKEN` <br/> `X-MS-TOKEN-FACEBOOK-EXPIRES-ON` |
| Google | `X-MS-TOKEN-GOOGLE-ID-TOKEN` <br/> `X-MS-TOKEN-GOOGLE-ACCESS-TOKEN` <br/> `X-MS-TOKEN-GOOGLE-EXPIRES-ON` <br/> `X-MS-TOKEN-GOOGLE-REFRESH-TOKEN` |
| Учетная запись Майкрософт | `X-MS-TOKEN-MICROSOFTACCOUNT-ACCESS-TOKEN` <br/> `X-MS-TOKEN-MICROSOFTACCOUNT-EXPIRES-ON` <br/> `X-MS-TOKEN-MICROSOFTACCOUNT-AUTHENTICATION-TOKEN` <br/> `X-MS-TOKEN-MICROSOFTACCOUNT-REFRESH-TOKEN` |
| Twitter | `X-MS-TOKEN-TWITTER-ACCESS-TOKEN` <br/> `X-MS-TOKEN-TWITTER-ACCESS-TOKEN-SECRET` |
|||

В коде клиента (например, в мобильном приложении или в браузере JavaScript) отправьте `GET` запрос HTTP `/.auth/me` ([хранилище токенов](overview-authentication-authorization.md#token-store) должно быть включено). В возвращаемом JSON-файле будут содержаться предоставляемые поставщиком токены.

> [!NOTE]
> Токены доступа предназначены для получения доступа к ресурсам поставщика, поэтому они заданы, только если настройка поставщика осуществляется с помощью секрета клиента. Дополнительные сведения о том, как получить токены обновления, см. в разделе об обновлении маркеров доступа.

## <a name="refresh-identity-provider-tokens"></a>Обновление маркеров поставщика удостоверений

По истечении срока действия маркера доступа вашего провайдера (а не [токена сеанса](#extend-session-token-expiration-grace-period)) необходимо повторно подтвердить подлинность пользователя, прежде чем снова использовать этот маркер. Истечения срока действия токена можно избежать, выполнив вызов `GET` к конечной точке приложения `/.auth/refresh`. При вызове служба приложений автоматически обновляет маркеры доступа в [хранилище токенов](overview-authentication-authorization.md#token-store) для пользователя, прошедшего проверку подлинности. В результате последующих запросов на токены с помощью кода приложения будут возвращаться обновленные токены. Однако для правильного обновления токена в хранилище токенов должны содержаться [токены обновления](https://auth0.com/learn/refresh-tokens/) для поставщика. Способ получения токенов обновления документируется каждым поставщиком, но ниже приведено краткое описание.

- **Google**. Добавьте параметр строки запроса `access_type=offline` к вызову API `/.auth/login/google`. Если используется пакет SDK для мобильных служб, можно добавить параметр к одной из перегрузок `LogicAsync` (см. в разделе о [токенах обновления Google](https://developers.google.com/identity/protocols/OpenIDConnect#refresh-tokens)).
- **Facebook**. Не предоставляет токены обновления. Срок действия токенов с долгим временем существования истекает через 60 дней (см. раздел об [истечении и продлении срока действия токенов доступа Facebook](https://developers.facebook.com/docs/facebook-login/access-tokens/expiration-and-extension)).
- **Twitter**. Срок действия токенов доступа не истекает (см. раздел о [часто задаваемых вопросах о Twitter OAuth](https://developer.twitter.com/en/docs/authentication/faq)).
- **Учетная запись Майкрософт**. [Настраивая параметры проверки подлинности учетной записи Майкрософт](configure-authentication-provider-microsoft.md), выберите область `wl.offline_access`.
- **Azure Active Directory**. В [https://resources.azure.com](https://resources.azure.com) сделайте следующее:
    1. В верхней части страницы выберите **Read/Write** (Чтение и запись).
    2. В левом браузере перейдите к разделу **подписки** > * *_\<subscription\_name_** > **resourceGroups** > * *_* * \<resource\_group\_name> _**поставщики**>  >  **Microsoft. Web**  >  **Sites** > * *_ \<app\_name> _ * * > **config**  >  **authsettings**. 
    3. Нажмите кнопку **Изменить**.
    4. Измените следующее свойство. Замените _\<app\_id>_ Azure Active Directory идентификатором приложения службы, к которой требуется получить доступ.

        ```json
        "additionalLoginParams": ["response_type=code id_token", "resource=<app_id>"]
        ```

    5. Щелкните **Put**. 

После настройки поставщика можно [найти токен обновления и срок действия для токена доступа](#retrieve-tokens-in-app-code) в хранилище токенов. 

Чтобы обновить маркер доступа в любое время, просто вызовите `/.auth/refresh` его на любом языке. В следующем фрагменте кода jQuery используется для обновления токенов доступа из клиента JavaScript.

```javascript
function refreshTokens() {
  let refreshUrl = "/.auth/refresh";
  $.ajax(refreshUrl) .done(function() {
    console.log("Token refresh completed successfully.");
  }) .fail(function() {
    console.log("Token refresh failed. See application logs for details.");
  });
}
```

Если пользователь отменяет разрешения, предоставленные приложению, вызов `/.auth/me` может завершиться ошибкой `403 Forbidden`. Чтобы выполнить диагностику ошибок, проверьте дополнительные сведения в журналах приложений.

## <a name="extend-session-token-expiration-grace-period"></a>Продление льготного периода срока действия токена сеанса

Сеанс проверки подлинности истекает через 8 часов. По истечении срока действия авторизованного сеанса по умолчанию предоставляется 72-часовой льготный период. В течение этого льготного периода токен сеанса можно обновить с помощью Службы приложений без повторной проверки подлинности пользователя. Если токен сеанса становится недействительным, вы можете просто вызвать `/.auth/refresh`, и вам не нужно самостоятельно отслеживать истечение срока действия токена. После завершения 72-часового льготного периода пользователь должен выполнить вход в систему, чтобы получить действительный токен сеанса.

Если 72 часов не достаточно, можно продлить этот льготный период. Значительное продление этого периода может повлиять на уровень безопасности (например, токен проверки подлинности может быть утерян или украден). В связи с этим следует оставить значение по умолчанию (72 часа) или установить для периода продления наименьшее значение.

Чтобы продлить льготный период истечения срока действия по умолчанию, выполните следующую команду в [Cloud Shell](../cloud-shell/overview.md).

```azurecli-interactive
az webapp auth update --resource-group <group_name> --name <app_name> --token-refresh-extension-hours <hours>
```

> [!NOTE]
> Льготный период применяется только к сеансу, проверка подлинности для которого осуществлялась с помощью службы приложений, а не токенов, предоставляемых поставщиками удостоверений. Льготный период на предоставленные поставщиком токены, срок действия которых истек, не распространяется. 
>

## <a name="limit-the-domain-of-sign-in-accounts"></a>Ограничение домена учетных записей для входа

Учетные записи Майкрософт и Azure Active Directory позволяют выполнять вход из нескольких доменов. Например, учетная запись Майкрософт позволяет выполнять вход с помощью учетных записей _outlook.com_, _live.com_ и _hotmail.com_. Azure AD позволяет любому числу пользовательских доменов для учетных записей входа. Тем не менее, вам может потребоваться ускорить работу пользователей с собственной фирменной страницей входа в Azure AD (например, `contoso.com` ). Чтобы предложить доменное имя учетных записей входа, выполните следующие действия.

В [https://resources.azure.com](https://resources.azure.com) перейдите к разделу **подписки** > * *_\<subscription\_name_** > **resourceGroups** > * *_* * \<resource\_group\_name> _**поставщики**>  >  **Microsoft. Web**  >  **Sites** > * *_ \<app\_name> _ * * > **config**  >  **authsettings**. 

Щелкните **Edit** (Изменить), измените следующее свойство и выберите **Put**. Не забудьте заменить _\<domain\_name>_ требуемым доменом.

```json
"additionalLoginParams": ["domain_hint=<domain_name>"]
```

Этот параметр добавляет `domain_hint` параметр строки запроса к URL-адресу перенаправления имени входа. 

> [!IMPORTANT]
> Клиент может удалить `domain_hint` параметр после получения URL-адреса перенаправления, а затем войти в другой домен. Поэтому хотя эта функция удобна, она не является функцией безопасности.
>

## <a name="authorize-or-deny-users"></a>Авторизовать или запретить пользователей

Хотя служба приложений отвечает за простейший вариант авторизации (т. е. отклонять запросы без проверки подлинности), приложение может потребовать более детального поведения авторизации, например ограничение доступа только к определенной группе пользователей. В некоторых случаях необходимо написать пользовательский код приложения, чтобы разрешить или запретить доступ для пользователя, выполнившего вход. В других случаях служба приложений или поставщик удостоверений могут помочь без внесения изменений в код.

- [Уровень сервера](#server-level-windows-apps-only)
- [Уровень поставщика удостоверений](#identity-provider-level)
- [Уровень приложения](#application-level)

### <a name="server-level-windows-apps-only"></a>Уровень сервера (только для приложений Windows)

Для любого приложения Windows можно определить поведение авторизации веб-сервера IIS, изменив файл *Web.config* . Приложения Linux не используют IIS и не могут быть настроены с помощью *Web.config*.

1. Перейдите на страницу `https://<app-name>.scm.azurewebsites.net/DebugConsole`.

1. В обозревателе браузера файлов службы приложений перейдите на *сайт Site/wwwroot*. Если *Web.config* не существует, создайте его, выбрав **+**  >  **создать файл**. 

1. Выберите карандаш для *Web.config* , чтобы изменить его. Добавьте следующий код конфигурации и нажмите кнопку **сохранить**. Если *Web.config* уже существует, просто добавьте `<authorization>` элемент со всеми элементами. Добавьте учетные записи, которые требуется разрешить, в `<allow>` элементе.

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
       <system.web>
          <authorization>
            <allow users="user1@contoso.com,user2@contoso.com"/>
            <deny users="*"/>
          </authorization>
       </system.web>
    </configuration>
    ```

### <a name="identity-provider-level"></a>Уровень поставщика удостоверений

Поставщик удостоверений может предоставить некоторую проверку подлинности с помощью ключа. Пример:

- Для [службы приложений Azure](configure-authentication-provider-aad.md)вы можете [управлять доступом на уровне предприятия](../active-directory/manage-apps/what-is-access-management.md) непосредственно в Azure AD. Инструкции см. [в разделе Удаление доступа пользователя к приложению](../active-directory/manage-apps/methods-for-removing-user-access.md).
- Для [Google](configure-authentication-provider-google.md), проектов Google API, входящих в [организацию](https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy#organizations) , можно настроить разрешение доступа только для пользователей в вашей организации (см. [страницу поддержки **OAuth 2,0** в Google](https://support.google.com/cloud/answer/6158849?hl=en)).

### <a name="application-level"></a>На уровне приложения

Если любой из других уровней не обеспечивает необходимую авторизацию или если ваша платформа или поставщик удостоверений не поддерживается, необходимо написать пользовательский код для авторизации пользователей на основе [утверждений пользователей](#access-user-claims).

## <a name="updating-the-configuration-version-preview"></a>Обновление версии конфигурации (Предварительная версия)

Существует две версии API управления для функции проверки подлинности и авторизации. Для работы функции проверки подлинности (Предварительная версия) в портал Azure требуется версия предварительной версии 2. Приложение, уже использующее API V1, может выполнить обновление до версии v2 после внесения нескольких изменений. В частности, конфигурация секрета должна быть перемещена в параметры приложения, прикрепленные к гнезде. В настоящее время Настройка поставщика учетных записей Майкрософт также не поддерживается в версии 2.

> [!WARNING]
> Миграция в предварительную версию v2 приведет к отключению управления функцией проверки подлинности и авторизации службы приложений с помощью некоторых клиентов, таких как существующий интерфейс в портал Azure, Azure CLI и Azure PowerShell. Это не может быть отменено. Во время предварительной версии перенос рабочих нагрузок не рекомендуется или не поддерживается. Для тестирования приложений следует выполнить действия, описанные в этом разделе.

### <a name="moving-secrets-to-application-settings"></a>Перемещение секретов в параметры приложения

1. Получите существующую конфигурацию с помощью API V1:

   ```azurecli
   # For Web Apps
   az webapp auth show -g <group_name> -n <site_name>

   # For Azure Functions
   az functionapp auth show -g <group_name> -n <site_name>
   ```

   В результирующих полезных данных JSON запишите значение секрета, используемое для каждого настроенного поставщика:

   * AAD `clientSecret`
   * Ими `googleClientSecret`
   * Facebook `facebookAppSecret`
   * Сайте `twitterConsumerSecret`
   * Учетная запись Майкрософт: `microsoftAccountClientSecret`

   > [!IMPORTANT]
   > Секретные значения являются важными учетными данными безопасности и должны обрабатываться аккуратно. Не предоставляйте эти значения или не сохраним их на локальном компьютере.

1. Создание параметров приложения с прикрепленными слотами для каждого значения секрета. Вы можете выбрать имя каждого параметра приложения. Это значение должно соответствовать значению, полученному на предыдущем шаге, или [ссылке на Key Vault секрет](./app-service-key-vault-references.md?toc=/azure/azure-functions/toc.json) , созданный с помощью этого значения.

   Чтобы создать параметр, можно использовать портал Azure или выполнить один из следующих вариантов для каждого поставщика:

   ```azurecli
   # For Web Apps, Google example    
   az webapp config appsettings set -g <group_name> -n <site_name> --slot-settings GOOGLE_PROVIDER_AUTHENTICATION_SECRET=<value_from_previous_step>

   # For Azure Functions, Twitter example
   az functionapp config appsettings set -g <group_name> -n <site_name> --slot-settings TWITTER_PROVIDER_AUTHENTICATION_SECRET=<value_from_previous_step>
   ```

   > [!NOTE]
   > Параметры приложения для этой конфигурации должны быть отмечены как слоты, что означает, что они не будут перемещаться между средами во время [операции переключения слотов](./deploy-staging-slots.md). Это связано с тем, что сама конфигурация проверки подлинности привязана к среде. 

1. Создайте новый JSON-файл с именем `authsettings.json` . Примите ранее полученные выходные данные и удалите из него все значения секрета. Запишите оставшиеся выходные данные в файл, убедившись, что секрет не включен. В некоторых случаях в конфигурации могут быть массивы, содержащие пустые строки. Убедитесь, что это не `microsoftAccountOAuthScopes` так, и если это так, переключите это значение на `null` .

1. Добавьте свойство, `authsettings.json` которое указывает на имя параметра приложения, созданное ранее для каждого поставщика:
 
   * AAD `clientSecretSettingName`
   * Ими `googleClientSecretSettingName`
   * Facebook `facebookAppSecretSettingName`
   * Сайте `twitterConsumerSecretSettingName`
   * Учетная запись Майкрософт: `microsoftAccountClientSecretSettingName`

   Пример файла после этой операции может выглядеть примерно так, как показано ниже, в данном случае только для AAD:

   ```json
   {
       "id": "/subscriptions/00d563f8-5b89-4c6a-bcec-c1b9f6d607e0/resourceGroups/myresourcegroup/providers/Microsoft.Web/sites/mywebapp/config/authsettings",
       "name": "authsettings",
       "type": "Microsoft.Web/sites/config",
       "location": "Central US",
       "properties": {
           "enabled": true,
           "runtimeVersion": "~1",
           "unauthenticatedClientAction": "AllowAnonymous",
           "tokenStoreEnabled": true,
           "allowedExternalRedirectUrls": null,
           "defaultProvider": "AzureActiveDirectory",
           "clientId": "3197c8ed-2470-480a-8fae-58c25558ac9b",
           "clientSecret": "",
           "clientSecretSettingName": "MICROSOFT_IDENTITY_AUTHENTICATION_SECRET",
           "clientSecretCertificateThumbprint": null,
           "issuer": "https://sts.windows.net/0b2ef922-672a-4707-9643-9a5726eec524/",
           "allowedAudiences": [
               "https://mywebapp.azurewebsites.net"
           ],
           "additionalLoginParams": null,
           "isAadAutoProvisioned": true,
           "aadClaimsAuthorization": null,
           "googleClientId": null,
           "googleClientSecret": null,
           "googleClientSecretSettingName": null,
           "googleOAuthScopes": null,
           "facebookAppId": null,
           "facebookAppSecret": null,
           "facebookAppSecretSettingName": null,
           "facebookOAuthScopes": null,
           "gitHubClientId": null,
           "gitHubClientSecret": null,
           "gitHubClientSecretSettingName": null,
           "gitHubOAuthScopes": null,
           "twitterConsumerKey": null,
           "twitterConsumerSecret": null,
           "twitterConsumerSecretSettingName": null,
           "microsoftAccountClientId": null,
           "microsoftAccountClientSecret": null,
           "microsoftAccountClientSecretSettingName": null,
           "microsoftAccountOAuthScopes": null,
           "isAuthFromFile": "false"
       }   
   }
   ```

1. Отправьте этот файл в качестве новой конфигурации проверки подлинности или авторизации для приложения:

   ```azurecli
   az rest --method PUT --url "/subscriptions/<subscription_id>/resourceGroups/<group_name>/providers/Microsoft.Web/sites/<site_name>/config/authsettings?api-version=2020-06-01" --body @./authsettings.json
   ```

1. Проверьте, что приложение по-прежнему работает, как ожидалось после этого жеста.

1. Удалите файл, который использовался на предыдущих шагах.

Теперь вы выполнили миграцию приложения для хранения секретов поставщика удостоверений в качестве параметров приложения.

### <a name="support-for-microsoft-account-registrations"></a>Поддержка регистраций учетная запись Майкрософт

В настоящее время API v2 не поддерживает учетную запись Майкрософт в качестве отдельного поставщика. Вместо этого он использует объединенную [платформу Microsoft Identity](../active-directory/develop/v2-overview.md) для входа пользователей с личными учетными записями Майкрософт. При переключении на API v2 конфигурация Azure Active Directory v1 используется для настройки поставщика платформы удостоверений Майкрософт.

Если существующая конфигурация содержит поставщик учетных записей Майкрософт и не содержит поставщика Azure Active Directory, можно переключить конфигурацию на поставщик Azure Active Directory, а затем выполнить миграцию. Для этого выполните следующие действия.

1. Перейдите к [**Регистрация приложений**](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) в портал Azure и найдите регистрацию, связанную с поставщиком учетной записи Майкрософт. Он может находиться в заголовке "приложения из личной учетной записи".
1. Перейдите на страницу проверки подлинности для регистрации. В разделе "URI перенаправления" должна отобразиться запись, которая заканчивается на `/.auth/login/microsoftaccount/callback` . Скопируйте этот URI.
1. Добавьте новый универсальный код ресурса (URI), совпадающий с только что скопированным, за исключением того, что он имеет значение End `/.auth/login/aad/callback` . Это позволит использовать регистрацию в конфигурации проверки подлинности и авторизации службы приложений.
1. Перейдите к конфигурации проверки подлинности и авторизации в службе приложений для приложения.
1. Собирайте конфигурацию для поставщика учетных записей Майкрософт.
1. Настройте поставщик Azure Active Directory с помощью режима управления "Дополнительно", указав идентификатор клиента и значения секрета клиента, собранные на предыдущем шаге. Для URL-адреса издателя используйте `<authentication-endpoint>/<tenant-id>/v2.0` и замените *\<authentication-endpoint>* [конечной точкой проверки подлинности для облачной среды](../active-directory/develop/authentication-national-cloud.md#azure-ad-authentication-endpoints) (например, " https://login.microsoftonline.com " для глобального Azure), а также замените *\<tenant-id>* **идентификатором каталога (клиента)**.
1. После сохранения конфигурации протестируйте поток входа, перейдя в браузер к `/.auth/login/aad` конечной точке на сайте и завершая процесс входа.
1. На этом этапе вы успешно скопировали конфигурацию, но существующая конфигурация поставщика учетных записей Майкрософт остается. Перед удалением убедитесь, что все части приложения ссылаются на поставщика Azure Active Directory по ссылкам для входа и т. д. Убедитесь, что все части приложения работают должным образом.
1. После проверки того, что все работает с поставщиком Azure Active Directory AAD, можно удалить конфигурацию поставщика учетных записей Майкрософт.

Некоторые приложения уже могут иметь отдельные регистрации для Azure Active Directory и учетной записи Майкрософт. В настоящее время невозможно выполнить миграцию этих приложений. 

> [!WARNING]
> Можно выполнить схождение двух регистраций, изменив [Поддерживаемые типы учетных записей](../active-directory/develop/supported-accounts-validation.md) для регистрации приложения AAD. Тем не менее это приведет к принудительному появлении нового запроса согласия для пользователей учетных записей Майкрософт, а в структуре утверждения удостоверений этих пользователей могут отличаться, в `sub` особенности изменяя значения с момента использования нового идентификатора приложения. Этот подход не рекомендуется использовать, если не тщательно обпонимается. Вместо этого следует дождаться поддержки двух регистраций в области API v2.

### <a name="switching-to-v2"></a>Переключение на v2

После выполнения описанных выше действий перейдите к приложению в портал Azure. Выберите раздел "Проверка подлинности (Предварительная версия)". 

Кроме того, вы можете выполнить запрос на размещение для `config/authsettingsv2` ресурса в ресурсе сайта. Схема полезных данных аналогична схеме, захваченной в разделе [Настройка с использованием файла](#config-file) .

## <a name="configure-using-a-file-preview"></a><a name="config-file"> </a>Настройка с помощью файла (Предварительная версия)

При необходимости можно настроить параметры проверки подлинности с помощью файла, предоставленного вашим развертыванием. Это может потребоваться для некоторых возможностей предварительной версии проверки подлинности и авторизации службы приложений.

> [!IMPORTANT]
> Помните, что полезные данные приложения и, следовательно, могут перемещаться между средами, как и с [слотами](./deploy-staging-slots.md). Вполне вероятно, что для каждого слота будет закреплена другая регистрация приложения, и в таких случаях следует продолжить использовать стандартный метод настройки вместо использования файла конфигурации.

### <a name="enabling-file-based-configuration"></a>Включение конфигурации на основе файлов

> [!CAUTION]
> Во время предварительной версии включение настройки на основе файлов приведет к отключению управления функцией проверки подлинности и авторизации службы приложений с помощью некоторых клиентов, таких как портал Azure, Azure CLI и Azure PowerShell.

1. Создайте новый JSON-файл для конфигурации в корне проекта (развернутый в D:\home\site\wwwroot в приложении для Интернета или функции). Укажите требуемую конфигурацию в соответствии с [файловой ссылкой на файловую конфигурацию](#configuration-file-reference). При изменении существующей конфигурации Azure Resource Manager убедитесь, что свойства, захваченные в коллекции, переведены в `authsettings` файл конфигурации.

2. Измените существующую конфигурацию, которая захватывается в [Azure Resource Manager](../azure-resource-manager/management/overview.md) API в разделе `Microsoft.Web/sites/<siteName>/config/authsettings` . Чтобы изменить это, можно использовать [шаблон Azure Resource Manager](../azure-resource-manager/templates/overview.md) или средство, например [Обозреватель ресурсов Azure](https://resources.azure.com/). В коллекции authsettings необходимо задать три свойства (и, возможно, удалить другие):

    1.  Значение `enabled` "true"
    2.  Значение `isAuthFromFile` "true"
    3.  Задайте `authFilePath` в качестве имени файла (например, "auth.js").

> [!NOTE]
> Формат `authFilePath` различается для разных платформ. В Windows поддерживаются как относительные, так и абсолютные пути. Рекомендуется использовать относительный вариант. В Linux в настоящее время поддерживаются только абсолютные пути, поэтому значение параметра должно быть "/Хоме/Сите/ввврут/auth.json" или аналогичным образом.

После внесения этого обновления конфигурации содержимое файла будет использоваться для определения поведения проверки подлинности и авторизации службы приложений для этого сайта. Если вы хотите вернуться к Azure Resource Managerной конфигурации, это можно сделать, задав значение `isAuthFromFile` "false".

### <a name="configuration-file-reference"></a>Ссылка на файл конфигурации

Все секреты, на которые будет ссылаться файл конфигурации, должны храниться в виде [параметров приложения](./configure-common.md#configure-app-settings). Вы можете назвать все нужные параметры. Просто убедитесь, что ссылки из файла конфигурации используют одни и те же ключи.

Ниже перечислены возможные варианты конфигурации в файле.

```json
{
    "platform": {
        "enabled": <true|false>
    },
    "globalValidation": {
        "unauthenticatedClientAction": "RedirectToLoginPage|AllowAnonymous|Return401|Return403",
        "redirectToProvider": "<default provider alias>",
        "excludedPaths": [
            "/path1",
            "/path2"
        ]
    },
    "httpSettings": {
        "requireHttps": <true|false>,
        "routes": {
            "apiPrefix": "<api prefix>"
        },
        "forwardProxy": {
            "convention": "NoProxy|Standard|Custom",
            "customHostHeaderName": "<host header value>",
            "customProtoHeaderName": "<proto header value>"
        }
    },
    "login": {
        "routes": {
            "logoutEndpoint": "<logout endpoint>"
        },
        "tokenStore": {
            "enabled": <true|false>,
            "tokenRefreshExtensionHours": "<double>",
            "fileSystem": {
                "directory": "<directory to store the tokens in if using a file system token store (default)>"
            },
            "azureBlobStorage": {
                "sasUrlSettingName": "<app setting name containing the sas url for the Azure Blob Storage if opting to use that for a token store>"
            }
        },
        "preserveUrlFragmentsForLogins": <true|false>,
        "allowedExternalRedirectUrls": [
            "https://uri1.azurewebsites.net/",
            "https://uri2.azurewebsites.net/",
            "url_scheme_of_your_app://easyauth.callback"
        ],
        "cookieExpiration": {
            "convention": "FixedTime|IdentityDerived",
            "timeToExpiration": "<timespan>"
        },
        "nonce": {
            "validateNonce": <true|false>,
            "nonceExpirationInterval": "<timespan>"
        }
    },
    "identityProviders": {
        "azureActiveDirectory": {
            "enabled": <true|false>,
            "registration": {
                "openIdIssuer": "<issuer url>",
                "clientId": "<app id>",
                "clientSecretSettingName": "APP_SETTING_CONTAINING_AAD_SECRET",
            },
            "login": {
                "loginParameters": [
                    "paramName1=value1",
                    "paramName2=value2"
                ]
            },
            "validation": {
                "allowedAudiences": [
                    "audience1",
                    "audience2"
                ]
            }
        },
        "facebook": {
            "enabled": <true|false>,
            "registration": {
                "appId": "<app id>",
                "appSecretSettingName": "APP_SETTING_CONTAINING_FACEBOOK_SECRET"
            },
            "graphApiVersion": "v3.3",
            "login": {
                "scopes": [
                    "public_profile",
                    "email"
                ]
            },
        },
        "gitHub": {
            "enabled": <true|false>,
            "registration": {
                "clientId": "<client id>",
                "clientSecretSettingName": "APP_SETTING_CONTAINING_GITHUB_SECRET"
            },
            "login": {
                "scopes": [
                    "profile",
                    "email"
                ]
            }
        },
        "google": {
            "enabled": true,
            "registration": {
                "clientId": "<client id>",
                "clientSecretSettingName": "APP_SETTING_CONTAINING_GOOGLE_SECRET"
            },
            "login": {
                "scopes": [
                    "profile",
                    "email"
                ]
            },
            "validation": {
                "allowedAudiences": [
                    "audience1",
                    "audience2"
                ]
            }
        },
        "twitter": {
            "enabled": <true|false>,
            "registration": {
                "consumerKey": "<consumer key>",
                "consumerSecretSettingName": "APP_SETTING_CONTAINING TWITTER_CONSUMER_SECRET"
            }
        },
        "apple": {
            "enabled": <true|false>,
            "registration": {
                "clientId": "<client id>",
                "clientSecretSettingName": "APP_SETTING_CONTAINING_APPLE_SECRET"
            },
            "login": {
                "scopes": [
                    "profile",
                    "email"
                ]
            }
        },
        "openIdConnectProviders": {
            "<providerName>": {
                "enabled": <true|false>,
                "registration": {
                    "clientId": "<client id>",
                    "clientCredential": {
                        "clientSecretSettingName": "<name of app setting containing client secret>"
                    },
                    "openIdConnectConfiguration": {
                        "authorizationEndpoint": "<url specifying authorization endpoint>",
                        "tokenEndpoint": "<url specifying token endpoint>",
                        "issuer": "<url specifying issuer>",
                        "certificationUri": "<url specifying jwks endpoint>",
                        "wellKnownOpenIdConfiguration": "<url specifying .well-known/open-id-configuration endpoint - if this property is set, the other properties of this object are ignored, and authorizationEndpoint, tokenEndpoint, issuer, and certificationUri are set to the corresponding values listed at this endpoint>"
                    }
                },
                "login": {
                    "nameClaimType": "<name of claim containing name>",
                    "scopes": [
                        "openid",
                        "profile",
                        "email"
                    ],
                    "loginParameterNames": [
                        "paramName1=value1",
                        "paramName2=value2"
                    ],
                }
            },
            //...
        }
    }
}
```

## <a name="pin-your-app-to-a-specific-authentication-runtime-version"></a>Закрепление приложения для конкретной версии среды выполнения проверки подлинности

При включении проверки подлинности и авторизации по промежуточного слоя платформы внедряется в конвейер HTTP-запросов, как описано в [обзоре функций](overview-authentication-authorization.md#how-it-works). Это по промежуточного слоя платформы периодически обновляется новыми функциями и улучшениями в рамках регулярных обновлений платформы. По умолчанию приложение веб-приложения или функции будет выполняться в последней версии этого по промежуточного слоя платформы. Эти автоматические обновления всегда имеют обратную совместимость. Однако в редких случаях, когда это автоматическое обновление вводит ошибку во время выполнения для веб-приложения или функции, можно временно вернуться к предыдущей версии по промежуточного слоя. В этой статье объясняется, как временно закрепить приложение для определенной версии по промежуточного слоя проверки подлинности.

### <a name="automatic-and-manual-version-updates"></a>Обновление версий автоматически и вручную 

Вы можете закрепить приложение на конкретной версии по промежуточного слоя платформы, задав `runtimeVersion` параметр для приложения. Приложение всегда работает в последней версии, если вы не решили явно закрепить его к определенной версии. В каждый момент времени будет поддерживаться несколько версий. Если вы закрепляетесь на недопустимую версию, которая больше не поддерживается, приложение будет использовать последнюю версию. Чтобы всегда запускать последнюю версию, задайте значение `runtimeVersion` ~ 1. 

### <a name="view-and-update-the-current-runtime-version"></a>Просмотр и обновление текущей версии среды выполнения

Вы можете изменить версию среды выполнения, используемую приложением. Новая версия среды выполнения должна вступить в силу после перезапуска приложения. 

#### <a name="view-the-current-runtime-version"></a>Просмотр текущей версии среды выполнения

Текущую версию промежуточного слоя проверки подлинности платформы можно просмотреть либо с помощью Azure CLI, либо через одну из встроенных конечных точек HTTP версии в приложении.

##### <a name="from-the-azure-cli"></a>Использование Azure CLI

С помощью Azure CLI просмотрите текущую версию по промежуточного слоя с помощью команды [AZ webapp auth Показать](/cli/azure/webapp/auth#az-webapp-auth-show) .

```azurecli-interactive
az webapp auth show --name <my_app_name> \
--resource-group <my_resource_group>
```

В этом коде замените `<my_app_name>` именем своего приложения. Также замените `<my_resource_group>` именем группы ресурсов для приложения.

Вы увидите `runtimeVersion` поле в выходных данных CLI. Он будет похож на следующий пример выходных данных, который был усечен для ясности: 
```output
{
  "additionalLoginParams": null,
  "allowedAudiences": null,
    ...
  "runtimeVersion": "1.3.2",
    ...
}
```

##### <a name="from-the-version-endpoint"></a>Из конечной точки версии

Также можно нажать конечную точку/.АУС/версион в приложении, чтобы просмотреть текущую версию по промежуточного слоя, в которой выполняется приложение. Он будет выглядеть примерно следующим образом:
```output
{
"version": "1.3.2"
}
```

#### <a name="update-the-current-runtime-version"></a>Обновление текущей версии среды выполнения

С помощью Azure CLI можно обновить `runtimeVersion` параметр в приложении с помощью команды [AZ webapp auth Update](/cli/azure/webapp/auth#az-webapp-auth-update) .

```azurecli-interactive
az webapp auth update --name <my_app_name> \
--resource-group <my_resource_group> \
--runtime-version <version>
```

Замените `<my_app_name>` именем своего приложения. Также замените `<my_resource_group>` именем группы ресурсов для приложения. Кроме того, замените на `<version>` допустимую версию среды выполнения 1. x или `~1` для последней версии. Заметки о выпуске можно найти в разных версиях среды выполнения [здесь] ( https://github.com/Azure/app-service-announcements) чтобы определить версию для закрепления.

Эту команду можно выполнить в [Azure Cloud Shell](../cloud-shell/overview.md), выбрав **Попробовать** в предыдущем примере кода. Также можно использовать [Azure CLI локально](/cli/azure/install-azure-cli) для выполнения этой команды после выполнения команды [az login](/cli/azure/reference-index#az-login) для входа.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Руководство. Сквозная аутентификация и авторизация пользователей](tutorial-auth-aad.md)
