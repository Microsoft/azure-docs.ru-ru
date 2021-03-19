---
title: Настройка поставщика OpenID Connect Connect (Предварительная версия)
description: Узнайте, как настроить поставщик OpenID Connect Connect в качестве поставщика удостоверений для службы приложений или приложения функций Azure.
ms.topic: article
ms.date: 07/08/2020
ms.reviewer: mahender
ms.openlocfilehash: e8112f2dc20175e81cfa8388440b2d9aef6a419c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90983864"
---
# <a name="configure-your-app-service-or-azure-functions-app-to-login-using-an-openid-connect-provider-preview"></a>Настройка Службы приложений или службы "Функции Azure" для входа с помощью поставщика OpenID Connect (предварительная версия)

[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

В этой статье показано, как настроить службу приложений Azure или функцию Azure для использования пользовательского поставщика проверки подлинности, который соответствует [спецификации OpenID Connect Connect](https://openid.net/connect/). OpenID Connect Connect (OIDC) — это промышленный стандарт, используемый многими поставщиками удостоверений (поставщиков удостоверений). Вам не нужно понимать детали спецификации, чтобы настроить приложение для использования IDP адхерент.

Вы можете настроить приложение для использования одного или нескольких поставщиков OIDC. Каждому из них должно быть присвоено уникальное имя в конфигурации, и только одно из них может выступать в качестве цели перенаправления по умолчанию.

> [!CAUTION]
> Включение поставщика OpenID Connect Connect отключит управление функцией проверки подлинности и авторизации службы приложений с помощью некоторых клиентов, таких как портал Azure, Azure CLI и Azure PowerShell. Эта функция полагается на новую поверхность API, которая, во время предварительной версии, еще не учитывается во всех средствах управления.

## <a name="register-your-application-with-the-identity-provider"></a><a name="register"> </a>Регистрация приложения с помощью поставщика удостоверений

Поставщик потребует зарегистрировать сведения о приложении. Ознакомьтесь с инструкциями, относящимися к этому поставщику. Необходимо будет получить **идентификатор клиента** и **секрет клиента** для приложения.

> [!IMPORTANT]
> Секрет приложения — это важные учетные данные безопасности. Не сообщайте этот секрет никому и не раскрывайте его в клиентском приложении.
>

> [!NOTE]
> Некоторые поставщики могут потребовать дополнительных действий для их настройки и использования предоставленных ими значений. Например, компания Apple предоставляет закрытый ключ, который не используется в качестве секрета клиента OIDC, и его необходимо использовать для создания JWT, который рассматривается как секрет, который вы задаете в конфигурации приложения (см. раздел "Создание секрета клиента" статьи [Вход с помощью документации Apple](https://developer.apple.com/documentation/sign_in_with_apple/generate_and_validate_tokens)).
>

Добавьте секрет клиента в качестве [параметра приложения](./configure-common.md#configure-app-settings) , используя имя выбранного вами параметра. Запишите это имя для последующего использования.

Кроме того, вам понадобятся метаданные OpenID Connect Connect для поставщика. Это часто предоставляется с помощью [документа метаданных конфигурации](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig), который является URL-адресом издателя поставщика с суффиксом `/.well-known/openid-configuration` . Соберите этот URL-адрес конфигурации.

Если не удается использовать документ метаданных конфигурации, необходимо отдельно собрать следующие значения:

- URL-адрес издателя (иногда отображается как `issuer` )
- [Конечная точка авторизации OAuth 2,0](https://tools.ietf.org/html/rfc6749#section-3.1) (иногда отображается как `authorization_endpoint` );
- [Конечная точка токена OAuth 2,0](https://tools.ietf.org/html/rfc6749#section-3.2) (иногда она показана как `token_endpoint` )
- URL-адрес документа [набора веб-ключей JSON OAuth 2,0](https://tools.ietf.org/html/rfc8414#section-2) (иногда отображается как `jwks_uri` )

## <a name="add-provider-information-to-your-application"></a><a name="configure"> </a>Добавление сведений о поставщике в приложение

> [!NOTE]
> Требуемая конфигурация находится в новом формате API, в настоящее время поддерживается только [конфигурацией на основе файлов (Предварительная версия)](.\app-service-authentication-how-to.md#config-file). При использовании такого файла необходимо выполнить приведенные ниже действия.

В этом разделе описано, как обновить конфигурацию, включив в нее новый IDP. Ниже приведен пример конфигурации.

1. В `identityProviders` объекте добавьте `openIdConnectProviders` объект, если он еще не существует.
1. В `openIdConnectProviders` объекте добавьте ключ для нового поставщика. Это понятное имя, используемое для ссылки на поставщик в оставшейся части конфигурации. Например, если требуется, чтобы все запросы проходили проверку подлинности с помощью этого поставщика, необходимо установить значение `globalValidation.unauthenticatedClientAction` "редиректтологинпаже" и задать `redirectToProvider` это же понятное имя.
1. Назначьте объект этому ключу с помощью `registration` объекта внутри него и при необходимости `login` объект:
    
    ```json
    "myCustomIDP" : {
       "registration" : {},
       "login": {
             "nameClaimType": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name",
             "scope": [],
             "loginParameterNames": [],
       }
    }
    ```

1. В объекте регистрации присвойте параметру `clientId` идентификатор клиента, который вы собрали, `clientCredential.secretSettingName` и укажите имя параметра приложения, в котором хранится секрет клиента, и создайте `openIdConnectConfiguration` объект:

    ```json
    "registration": {
      "clientId": "bd96cf8a-3f2b-4806-b180-d3c5fd11a2be",
      "clientCredential": {
         "secretSettingName": "IDP_CLIENT_SECRET"
      },
      "openIdConnectConfiguration" : {}
    }
    ```

1. В `openIdConnectConfiguration` объекте укажите метаданные OpenID Connect Connect, собранные ранее. В зависимости от собираемой информации можно получить два варианта.

    - Присвойте `wellKnownOpenIdConfiguration` свойству URL-адрес метаданных конфигурации, который вы собирали ранее.
    - Кроме того, можно задать четыре отдельных значения, собранные следующим образом:
        - Укажите `issuer` URL-адрес издателя
        - Задать `authorizationEndpoint` конечную точку авторизации
        - Задать `tokenEndpoint` конечную точку токена
        - Задайте `certificationUri` URL-адрес документа набора веб-ключей JSON.

    Эти два параметра являются взаимоисключающими.

После настройки этой конфигурации вы можете использовать поставщик OpenID Connect Connect для проверки подлинности в приложении.

Пример конфигурации может выглядеть следующим образом (для этого используется вход с Apple в качестве примера, где APPLE_GENERATED_CLIENT_SECRET параметр указывает на созданный JWT в соответствии с [документацией Apple](https://developer.apple.com/documentation/sign_in_with_apple/generate_and_validate_tokens)):

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
        "openIdConnectProviders": {
            "apple": {
                "registration": {
                    "clientId": "com.contoso.example.client",
                    "clientCredential": {
                        "secretSettingName": "APPLE_GENERATED_CLIENT_SECRET"
                    },
                    "openIdConnectConfiguration": {
                        "wellKnownOpenIdConfiguration": "https://appleid.apple.com/.well-known/openid-configuration"
                    }
                },
                "login": {
                    "nameClaimType": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name",
                    "scope": [],
                    "loginParameterNames": []
                }
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
