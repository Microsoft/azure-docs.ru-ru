---
title: Проверка маркера доступа Azure API для FHIR
description: Пошаговые инструкции по проверке маркера и советы по устранению проблем доступа
services: healthcare-apis
author: caitlinv39
ms.reviewer: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: conceptual
ms.date: 02/19/2019
ms.author: cavoeg
ms.openlocfilehash: 426ec0c2d6cc274aa0b6829eb4a30fd29b9ba8e2
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103020273"
---
# <a name="azure-api-for-fhir-access-token-validation"></a>Проверка маркера доступа Azure API для FHIR

Как Azure API для FHIR проверяет маркер доступа, он будет зависеть от реализации и конфигурации. В этой статье мы рассмотрим шаги проверки, которые могут быть полезны при устранении неполадок доступа.

## <a name="validate-token-has-no-issues-with-identity-provider"></a>У токена проверки нет проблем с поставщиком удостоверений

Первым шагом в проверке маркера является проверка того, что маркер был выдан правильным поставщиком удостоверений и что он не был изменен. Сервер FHIR будет настроен для использования определенного поставщика удостоверений, известного как центр сертификации `Authority` . Сервер FHIR будет получать сведения о поставщике удостоверений из `/.well-known/openid-configuration` конечной точки. При использовании Azure AD полный URL-адрес будет выглядеть следующим образом:

```
GET https://login.microsoftonline.com/<TENANT-ID>/.well-known/openid-configuration
```

где `<TENANT-ID>` — это конкретный клиент Azure AD (идентификатор клиента или доменное имя).

Azure AD вернет на сервер FHIR документ, подобный приведенному ниже.

```json
{
    "authorization_endpoint": "https://login.microsoftonline.com/<TENANT-ID>/oauth2/authorize",
    "token_endpoint": "https://login.microsoftonline.com/<TENANT-ID>/oauth2/token",
    "token_endpoint_auth_methods_supported": [
        "client_secret_post",
        "private_key_jwt",
        "client_secret_basic"
    ],
    "jwks_uri": "https://login.microsoftonline.com/common/discovery/keys",
    "response_modes_supported": [
        "query",
        "fragment",
        "form_post"
    ],
    "subject_types_supported": [
        "pairwise"
    ],
    "id_token_signing_alg_values_supported": [
        "RS256"
    ],
    "http_logout_supported": true,
    "frontchannel_logout_supported": true,
    "end_session_endpoint": "https://login.microsoftonline.com/<TENANT-ID>/oauth2/logout",
    "response_types_supported": [
        "code",
        "id_token",
        "code id_token",
        "token id_token",
        "token"
    ],
    "scopes_supported": [
        "openid"
    ],
    "issuer": "https://sts.windows.net/<TENANT-ID>/",
    "claims_supported": [
        "sub",
        "iss",
        "cloud_instance_name",
        "cloud_instance_host_name",
        "cloud_graph_host_name",
        "msgraph_host",
        "aud",
        "exp",
        "iat",
        "auth_time",
        "acr",
        "amr",
        "nonce",
        "email",
        "given_name",
        "family_name",
        "nickname"
    ],
    "microsoft_multi_refresh_token": true,
    "check_session_iframe": "https://login.microsoftonline.com/<TENANT-ID>/oauth2/checksession",
    "userinfo_endpoint": "https://login.microsoftonline.com/<TENANT-ID>/openid/userinfo",
    "tenant_region_scope": "WW",
    "cloud_instance_name": "microsoftonline.com",
    "cloud_graph_host_name": "graph.windows.net",
    "msgraph_host": "graph.microsoft.com",
    "rbac_url": "https://pas.windows.net"
}
``` 
Важными свойствами сервера FHIR являются `jwks_uri` , что указывает серверу, куда получить ключи шифрования, необходимые для проверки сигнатуры маркера `issuer` , и, который сообщает серверу, что будет в утверждении издателя ( `iss` ) токенов, выданных этим сервером. Сервер FHIR может использовать этот параметр, чтобы проверить, что он получает подлинную лексему.

## <a name="validate-claims-of-the-token"></a>Проверка утверждений токена

После того как сервер проверит подлинность маркера, сервер FHIR продолжит проверку наличия у клиента необходимых утверждений для доступа к маркеру.

При использовании API Azure для FHIR сервер выполняет проверку:

1. Маркер имеет право `Audience` ( `aud` утверждение).
1. Пользователю или участнику, для которого был выдан маркер, разрешен доступ к плоскости данных сервера FHIR. `oid`Утверждение маркера содержит идентификатор объекта Identity, который однозначно определяет пользователя или участника.

Рекомендуется настроить службу FHIR [для использования Azure RBAC](configure-azure-rbac.md) для управления назначениями ролей плоскости данных. Но вы также можете [настроить локальный RBAC](configure-local-rbac.md) , если служба FHIR использует внешний или вторичный клиент Azure Active Directory. 

При использовании OSS Microsoft FHIR Server для Azure сервер выполняет проверку:

1. Маркер имеет право `Audience` ( `aud` утверждение).
1. Маркер имеет роль в `roles` утверждении, которой разрешен доступ к серверу FHIR.

Дополнительные сведения о том, как [определить роли на сервере FHIR](https://github.com/microsoft/fhir-server/blob/master/docs/Roles.md), см. здесь.

Сервер FHIR также может проверить, что маркер доступа имеет области (в утверждении токена `scp` ) для доступа к части API FHIR, к которому пытается получить доступ клиент. В настоящее время API Azure для FHIR и сервер FHIR для Azure не проверяют области токенов.

## <a name="next-steps"></a>Дальнейшие действия
Теперь, когда вы узнали, как пройти проверку маркера, вы можете пройти это руководство, чтобы создать приложение JavaScript и прочесть данные FHIR.

>[!div class="nextstepaction"]
>[Учебник по веб-приложениям](tutorial-web-app-fhir-server.md)