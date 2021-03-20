---
title: Настройка входа с помощью Apple (Предварительная версия)
description: Узнайте, как настроить вход с помощью Apple в качестве поставщика удостоверений для службы приложений или приложения функций Azure.
ms.topic: article
ms.date: 11/19/2020
ms.reviewer: mikarmar
ms.openlocfilehash: b77e0613f502d003b5e4651e34be4cadbd4209a9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96603363"
---
# <a name="configure-your-app-service-or-azure-functions-app-to-sign-in-using-a-sign-in-with-apple-provider-preview"></a>Настройка службы приложений или приложения "функции Azure" для входа с использованием входа с помощью поставщика Apple (Предварительная версия)

[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

В этой статье показано, как настроить службу приложений Azure или функцию Azure для использования входа с помощью Apple в качестве поставщика проверки подлинности. 

Чтобы выполнить процедуру, описанную в этой статье, необходимо зарегистрироваться в программе для разработчиков Apple. Чтобы зарегистрироваться в программе Apple Developer, перейдите по адресу [Developer.Apple.com/Programs/Enroll](https://developer.apple.com/programs/enroll/).

> [!CAUTION]
> Включение функции входа с помощью Apple отключит управление функцией проверки подлинности и авторизации в службе приложений с помощью некоторых клиентов, таких как портал Azure, Azure CLI и Azure PowerShell. Эта функция полагается на новую поверхность API, которая, во время предварительной версии, еще не учитывается во всех средствах управления.

[comment]: <Remove this caution block when V2 becomes available for use.> 

## <a name="create-an-application-in-the-apple-developer-portal"></a><a name="createApplication"> </a>Создание приложения на портале разработчика Apple
Вам потребуется создать идентификатор приложения и идентификатор службы на портале разработчика Apple.

1. На портале разработчика Apple выберите **сертификаты, идентификаторы & профили**.
2. На вкладке **идентификаторы** нажмите кнопку **(+)** .
3. На странице **Регистрация нового идентификатора** выберите **Идентификаторы приложений** и нажмите кнопку **продолжить**. (Идентификаторы приложений включают один или несколько идентификаторов служб.) ![Регистрация нового идентификатора приложения на портале разработчика Apple](media/configure-authentication-provider-apple/apple-register-app.jpg)
4. На странице **Регистрация идентификатора приложения** укажите описание и идентификатор пакета, а затем выберите **войти с помощью Apple** из списка возможностей. Затем выберите **Continue** (Продолжить). Запишите **Префикс идентификатора приложения (идентификатор команды)** на этом шаге, он понадобится позже.
![Настройка нового идентификатора приложения на портале разработчика Apple](media/configure-authentication-provider-apple/apple-configure-app.jpg)
5. Проверьте сведения о регистрации приложения и нажмите кнопку **зарегистрировать**.
6. Опять же, на вкладке **идентификаторы** нажмите кнопку **(+)** .
![Создание нового идентификатора службы на портале разработчика Apple](media/configure-authentication-provider-apple/apple-new-app.jpg) 
7. На странице **Регистрация нового идентификатора** выберите **службы идентификаторы** и нажмите кнопку **продолжить**.
![Регистрация нового идентификатора службы на портале разработчика Apple](media/configure-authentication-provider-apple/apple-register-service.jpg)
8. На странице **Регистрация идентификатора служб** укажите описание и идентификатор. Описание будет отображаться для пользователя на экране согласия. Идентификатор будет ИДЕНТИФИКАТОРом клиента, используемым при настройке поставщика Apple в службе приложений. Затем выберите **Настроить**.
![Указание описания и идентификатора](media/configure-authentication-provider-apple/apple-configure-service-1.jpg)
9. Во всплывающем окне укажите в качестве идентификатора основного приложения идентификатор приложения, созданного ранее. Укажите домен приложения в разделе domain. Для URL-адреса возврата используйте URL-адрес `<app-url>/.auth/login/apple/callback` . Например, `https://contoso.azurewebsites.net/.auth/login/apple/callback`. Затем выберите **Добавить** и **сохранить**.
![Указание домена и URL-адреса возврата для регистрации](media/configure-authentication-provider-apple/apple-configure-service-2.jpg)
10. Проверьте сведения о регистрации службы и нажмите кнопку **сохранить**.

## <a name="generate-the-client-secret"></a><a name="generateClientSecret"> </a>Создание секрета клиента
Компании Apple требуется, чтобы разработчики приложений создали и подписывают маркер JWT в качестве секрета клиента. Чтобы создать этот секрет, сначала создайте и скачайте закрытый ключ эллиптической кривой на портале разработчика Apple. Затем используйте этот ключ, чтобы [подписать JWT](#sign-the-client-secret-jwt) с [определенной полезной](#structure-the-client-secret-jwt)нагрузкой.

### <a name="create-and-download-the-private-key"></a>Создание и скачивание закрытого ключа
1. На вкладке **ключи** на портале разработчика Apple выберите **создать ключ** или нажмите кнопку **(+)** .
2. На странице **Регистрация нового ключа** введите имя ключа, установите флажок далее, чтобы **войти с помощью Apple** , и выберите **настроить**.
3. На странице **Настройка ключа** свяжите ключ с идентификатором основного приложения, созданным ранее, и выберите **сохранить**.
4. Завершите создание ключа, подтверждая сведения и выбрав **Continue (продолжить** ), а затем просмотрев информацию и выбрав **Register (зарегистрировать**).
5. На странице **скачивание ключа** Скачайте ключ. Он будет скачан как `.p8` файл (PKCS # 8). Вы будете использовать содержимое файла, чтобы подписать секретный код JWT клиента.

### <a name="structure-the-client-secret-jwt"></a>Структурирование секретного кода JWT клиента
Apple требует, чтобы секрет клиента был Base64-кодированием маркера JWT. Декодированный маркер JWT должен иметь полезные данные, структурированные как в следующем примере:
```json
{
  "alg": "ES256",
  "kid": "URKEYID001",
}.{
  "sub": "com.yourcompany.app1",
  "nbf": 1560203207,
  "exp": 1560289607,
  "iss": "ABC123DEFG",
  "aud": "https://appleid.apple.com"
}.[Signature]
```
- **подсистема**: идентификатор клиента Apple (также идентификатор службы).
- **ISS**: идентификатор группы разработчиков Apple
- **AUD**: Apple получает маркер, поэтому он является аудиторией
- **exp**: не более шести месяцев после **NBF**

Версия приведенных выше полезных данных в кодировке Base64 выглядит следующим образом: ```eyJhbGciOiJFUzI1NiIsImtpZCI6IlVSS0VZSUQwMDEifQ.eyJzdWIiOiJjb20ueW91cmNvbXBhbnkuYXBwMSIsIm5iZiI6MTU2MDIwMzIwNywiZXhwIjoxNTYwMjg5NjA3LCJpc3MiOiJBQkMxMjNERUZHIiwiYXVkIjoiaHR0cHM6Ly9hcHBsZWlkLmFwcGxlLmNvbSJ9.ABSXELWuTbgqfrIUz7bLi6nXvkXAz5O8vt0jB2dSHTQTib1x1DSP4__4UrlKI-pdzNg1sgeocolPNTmDKazO8-BHAZCsdeeTNlgFEzBytIpMKFfVEQbEtGRkam5IeclUK7S9oOva4EK4jV4VmgDrr-LGWWO3TaAxAvy3_ZoKohvFFkVG```

_Примечание. Apple не принимает секрет клиента JWT с датой окончания срока действия более шести месяцев после даты создания (или NBF). Это означает, что необходимо по крайней мере каждые шесть месяцев поворачивать секрет клиента._

Дополнительные сведения о создании и проверке маркеров можно найти в [документации для разработчиков Apple](https://developer.apple.com/documentation/sign_in_with_apple/generate_and_validate_tokens). 

### <a name="sign-the-client-secret-jwt"></a>Подписать секретный код JWT клиента
Вы будете использовать `.p8` скачанный ранее файл, чтобы подписать секрет клиента JWT. Этот файл представляет собой [файл пккс # 8](https://en.wikipedia.org/wiki/PKCS_8) , содержащий закрытый ключ подписывания в формате PEM. Существует множество библиотек, которые могут создавать и подписывать JWT. 

Существует несколько типов библиотек с открытым исходным кодом, доступных в Интернете для создания и подписи маркеров JWT. Дополнительные сведения о создании токенов JWT см. в разделе jwt.io. Например, одним из способов создания секрета клиента является импорт [пакета NuGet Microsoft. IdentityModel. Tokens](https://www.nuget.org/packages/Microsoft.IdentityModel.Tokens/) и запуск небольшого объема кода C#, показанного ниже.

```csharp
using Microsoft.IdentityModel.Tokens;

public static string GetAppleClientSecret(string teamId, string clientId, string keyId, string p8key)
{
    string audience = "https://appleid.apple.com";

    string issuer = teamId;
    string subject = clientId;
    string kid = keyId;

    IList<Claim> claims = new List<Claim> {
        new Claim ("sub", subject)
    };

    CngKey cngKey = CngKey.Import(Convert.FromBase64String(p8key), CngKeyBlobFormat.Pkcs8PrivateBlob);

    SigningCredentials signingCred = new SigningCredentials(
        new ECDsaSecurityKey(new ECDsaCng(cngKey)),
        SecurityAlgorithms.EcdsaSha256
    );

    JwtSecurityToken token = new JwtSecurityToken(
        issuer,
        audience,
        claims,
        DateTime.Now,
        DateTime.Now.AddDays(180),
        signingCred
    );
    token.Header.Add("kid", kid);
    token.Header.Remove("typ");

    JwtSecurityTokenHandler tokenHandler = new JwtSecurityTokenHandler();

    return tokenHandler.WriteToken(token);
}
```
- **теамид**: ваш идентификатор группы разработчиков Apple
- **ClientID**: идентификатор клиента Apple (также идентификатор службы)
- **p8key**: ключ формата PEM. ключ можно получить, открыв его `.p8` в текстовом редакторе и скопировав все между `-----BEGIN PRIVATE KEY-----` и `-----END PRIVATE KEY-----` без разрывов строк.
- **keyId**: идентификатор скачанного ключа.

Этот токен возвращает значение секрета клиента, которое будет использоваться для настройки поставщика Apple.

> [!IMPORTANT]
> Секрет клиента — это важные учетные данные безопасности. Не сообщайте этот секрет никому и не раскрывайте его в клиентском приложении.
>

Добавьте секрет клиента в качестве [параметра приложения](./configure-common.md#configure-app-settings) , используя имя выбранного вами параметра. Запишите это имя для последующего использования.

## <a name="add-provider-information-to-your-application"></a><a name="configure"> </a>Добавление сведений о поставщике в приложение

> [!NOTE]
> Требуемая конфигурация находится в новом формате API, в настоящее время поддерживается только [конфигурацией на основе файлов (Предварительная версия)](.\app-service-authentication-how-to.md#config-file). При использовании такого файла необходимо выполнить приведенные ниже действия.

В этом разделе описано, как обновить конфигурацию, включив в нее новый IDP. Ниже приведен пример конфигурации.

1. В `identityProviders` объекте добавьте `apple` объект, если он еще не существует.
2. Назначьте объект этому ключу с помощью `registration` объекта внутри него и при необходимости `login` объект:
    
    ```json
    "apple" : {
       "registration" : {
            "clientId": "<client id>",
            "clientSecretSettingName": "APP_SETTING_CONTAINING_APPLE_CLIENT_SECRET" 
        },
       "login": {
             "scopes": []
       }
    }
    ```
    а. В `registration` объекте укажите `clientId` идентификатор клиента, который вы собрали.
    
    b. В `registration` объекте укажите `clientSecretSettingName` имя параметра приложения, в котором хранится секрет клиента.
    
    c. В `login` объекте можно задать `scopes` Включение в массив списка областей, используемых при проверке подлинности в Apple, например "имя" и "Электронная почта". Если области настроены, они будут явно запрошены на экране согласия при первом входе пользователей в систему.

После настройки этой конфигурации вы можете использовать поставщик Apple для проверки подлинности в приложении.

Полная конфигурация может выглядеть, как в следующем примере (где APPLE_GENERATED_CLIENT_SECRET параметр указывает на параметр приложения, содержащий созданный JWT):

```json
{
    "platform": {
        "enabled": true
    },
    "globalValidation": {
        "redirectToProvider": "apple",
        "unauthenticatedClientAction": "RedirectToLoginPage"
    },
    "identityProviders": {
        "apple": {
            "registration": {
                "clientId": "com.contoso.example.client",
                "clientSecretSettingName": "APPLE_GENERATED_CLIENT_SECRET"
            },
            "login": {
                "scopes": []
            }
        }
    },
    "login": {
        "tokenStore": {
            "enabled": true
        }
    }     
}
```

## <a name="next-steps"></a><a name="related-content"> </a>Дальнейшие действия

[!INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]
