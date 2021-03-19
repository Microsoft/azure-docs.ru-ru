---
title: Инструкции по миграции для Python ADAL в MSAL | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как перенести приложение Python для библиотеки проверки подлинности Azure Active Directory (ADAL) в библиотеку проверки подлинности Майкрософт (MSAL) для Python.
services: active-directory
author: rayluo
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.tgt_pltfrm: Python
ms.workload: identity
ms.date: 11/11/2019
ms.author: rayluo
ms.reviewer: marsma, rayluo, nacanuma
ms.custom: aaddev, devx-track-python
ms.openlocfilehash: 60d6e40b568c1189cdc01ef4239612d3717063b7
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104578844"
---
# <a name="adal-to-msal-migration-guide-for-python"></a>Руководством по миграции ADAL в MSAL для Python

В этой статье описываются изменения, которые необходимо внести, чтобы перенести приложение, Azure Active Directory использующее библиотеку проверки подлинности (MSAL), с помощью библиотеки аутентификации Майкрософт (ADAL).

## <a name="difference-highlights"></a>Особенности разницы

ADAL работает с конечной точкой Azure Active Directory (Azure AD) v 1.0. Библиотека проверки подлинности Microsoft (MSAL) работает с платформой идентификации Майкрософт, которая ранее называлась конечной точкой Azure Active Directory v 2.0. Платформа Microsoft Identity отличается от Azure AD версии 1.0 тем, что:

Поддерживаются следующие возможности:
  - пользователи с рабочими и учебными учетными записями (учетные записи, подготовленные в Azure AD);
  - пользователи с личными учетными записями (например, Outlook.com или Hotmail.com);
  - клиенты, которые используют собственные адреса электронной почты или удостоверения социальных сетей (например, LinkedIn, Facebook, Google) через предложение Azure AD B2C.

- Совместимы со стандартами:
  - OAuth 2.0
  - OpenID Connect Connect (OIDC)

Дополнительные сведения см. [в разделе чем отличаются платформы Microsoft Identity](../azuread-dev/azure-ad-endpoint-comparison.md) .

### <a name="scopes-not-resources"></a>Области вместо ресурсов

ADAL Python получает маркеры для ресурсов, но MSAL Python получает маркеры для областей. У поверхности API в MSAL Python больше нет параметра ресурса. Необходимо указать области в виде списка строк, которые объявляют требуемые разрешения и ресурсы. Некоторые примеры областей см. в разделе [области Microsoft Graph](/graph/permissions-reference).

Вы можете добавить `/.default` суффикс области к ресурсу, чтобы упростить перенос приложений из конечной точки версии 1.0 (ADAL) на платформу Microsoft Identity (MSAL). Например, для значения ресурса `https://graph.microsoft.com` в качестве эквивалентного значения области является `https://graph.microsoft.com/.default` .  Если ресурс отсутствует в форме URL-адреса, но идентификатор ресурса формы `XXXXXXXX-XXXX-XXXX-XXXXXXXXXXXX` , можно по-прежнему использовать значение области в качестве `XXXXXXXX-XXXX-XXXX-XXXXXXXXXXXX/.default` .

Дополнительные сведения о различных типах областей см. в статьях [разрешения и согласие на платформе Microsoft Identity](./v2-permissions-and-consent.md) , а также [области для веб-API, принимающего статьи о токенах версии 1.0](./msal-v1-app-scopes.md) .

### <a name="error-handling"></a>Обработка ошибок

Azure Active Directory Библиотека проверки подлинности (ADAL) для Python использует исключение `AdalError` , чтобы указать, что возникла проблема. MSAL для Python обычно использует коды ошибок. Дополнительные сведения см. в разделе [MSAL for Python Error обработка ошибок](msal-error-handling-python.md).

### <a name="api-changes"></a>Изменения API

В следующей таблице приведен интерфейс API в ADAL для Python, который можно использовать в своем месте в MSAL для Python:

| ADAL для API Python  | MSAL для API Python |
| ------------------- | ---------------------------------- |
| [AuthenticationContext](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext)  | [Публикклиентаппликатион или Конфидентиалклиентаппликатион](https://msal-python.readthedocs.io/en/latest/#msal.ClientApplication.__init__)  |
| Н/Д  | [get_authorization_request_url ()](https://msal-python.readthedocs.io/en/latest/#msal.ClientApplication.get_authorization_request_url)  |
| [acquire_token_with_authorization_code ()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_token_with_authorization_code) | [acquire_token_by_authorization_code ()](https://msal-python.readthedocs.io/en/latest/#msal.ClientApplication.acquire_token_by_authorization_code) |
| [acquire_token ()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_token) | [acquire_token_silent ()](https://msal-python.readthedocs.io/en/latest/#msal.ClientApplication.acquire_token_silent) |
| [acquire_token_with_refresh_token ()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_token_with_refresh_token) | Н/Д |
| [acquire_user_code ()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_user_code) | [initiate_device_flow ()](https://msal-python.readthedocs.io/en/latest/#msal.PublicClientApplication.initiate_device_flow) |
| [acquire_token_with_device_code ()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_token_with_device_code) и [cancel_request_to_get_token_with_device_code ()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.cancel_request_to_get_token_with_device_code) | [acquire_token_by_device_flow ()](https://msal-python.readthedocs.io/en/latest/#msal.PublicClientApplication.acquire_token_by_device_flow) |
| [acquire_token_with_username_password ()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_token_with_username_password) | [acquire_token_by_username_password ()](https://msal-python.readthedocs.io/en/latest/#msal.PublicClientApplication.acquire_token_by_username_password) |
| [acquire_token_with_client_credentials ()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_token_with_client_credentials) и [acquire_token_with_client_certificate ()](https://adal-python.readthedocs.io/en/latest/#adal.AuthenticationContext.acquire_token_with_client_certificate) | [acquire_token_for_client ()](https://msal-python.readthedocs.io/en/latest/#msal.ConfidentialClientApplication.acquire_token_for_client) |
| Н/Д | [acquire_token_on_behalf_of ()](https://msal-python.readthedocs.io/en/latest/#msal.ConfidentialClientApplication.acquire_token_on_behalf_of) |
| [TokenCache ()](https://adal-python.readthedocs.io/en/latest/#adal.TokenCache) | [Сериализаблетокенкаче ()](https://msal-python.readthedocs.io/en/latest/#msal.SerializableTokenCache) |
| Н/Д | Кэширование с сохранением, доступное из [расширений MSAL](https://github.com/marstr/original-microsoft-authentication-extensions-for-python) |

## <a name="migrate-existing-refresh-tokens-for-msal-python"></a>Перенос существующих маркеров обновления для MSAL Python

Библиотека проверки подлинности Майкрософт (MSAL) абстрагирует концепцию маркеров обновления. MSAL Python по умолчанию предоставляет кэш маркеров в памяти, поэтому вам не нужно хранить, искать или обновлять маркеры обновления. Пользователи также увидят меньше запросов на вход, так как маркеры обновления обычно обновляются без вмешательства пользователя. Дополнительные сведения о кэше маркеров см. в разделе [сериализация кэша пользовательской лексемы в MSAL для Python](msal-python-token-cache-serialization.md).

Следующий код поможет перенести маркеры обновления, управляемые другой библиотекой OAuth2 (включая, но не ограниченные ADAL Python), под управление MSAL для Python. Одна из причин переноса этих маркеров обновления — предотвращение повторного входа существующих пользователей при переносе приложения в MSAL для Python.

Метод переноса маркера обновления — использование MSAL для Python для получения нового маркера доступа с помощью предыдущего маркера обновления. При возвращении нового маркера обновления MSAL для Python сохранит его в кэше.
Так как MSAL Python 1.3.0, мы предоставляем API в MSAL для этой цели.
См. следующий фрагмент кода, заключенный в кавычки из [завершенного примера миграции маркеров обновления с помощью MSAL Python](https://github.com/AzureAD/microsoft-authentication-library-for-python/blob/1.3.0/sample/migrate_rt.py#L28-L67) .

```python
import msal
def get_preexisting_rt_and_their_scopes_from_elsewhere():
    # Maybe you have an ADAL-powered app like this
    #   https://github.com/AzureAD/azure-activedirectory-library-for-python/blob/1.2.3/sample/device_code_sample.py#L72
    # which uses a resource rather than a scope,
    # you need to convert your v1 resource into v2 scopes
    # See https://docs.microsoft.com/azure/active-directory/azuread-dev/azure-ad-endpoint-comparison#scopes-not-resources
    # You may be able to append "/.default" to your v1 resource to form a scope
    # See https://docs.microsoft.com/azure/active-directory/develop/v2-permissions-and-consent#the-default-scope

    # Or maybe you have an app already talking to the Microsoft identity platform,
    # powered by some 3rd-party auth library, and persist its tokens somehow.

    # Either way, you need to extract RTs from there, and return them like this.
    return [
        ("old_rt_1", ["scope1", "scope2"]),
        ("old_rt_2", ["scope3", "scope4"]),
        ]


# We will migrate all the old RTs into a new app powered by MSAL
app = msal.PublicClientApplication(
    "client_id", authority="...",
    # token_cache=...  # Default cache is in memory only.
                       # You can learn how to use SerializableTokenCache from
                       # https://msal-python.readthedocs.io/en/latest/#msal.SerializableTokenCache
    )

# We choose a migration strategy of migrating all RTs in one loop
for old_rt, scopes in get_preexisting_rt_and_their_scopes_from_elsewhere():
    result = app.acquire_token_by_refresh_token(old_rt, scopes)
    if "error" in result:
        print("Discarding unsuccessful RT. Error: ", json.dumps(result, indent=2))

print("Migration completed")
```


## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о сравнении версий 1.0 и 2.0 см. [здесь](../azuread-dev/azure-ad-endpoint-comparison.md).
