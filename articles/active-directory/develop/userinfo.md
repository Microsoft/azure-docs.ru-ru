---
title: Конечная точка UserInfo платформы удостоверений (Майкрософт) | Службы
titleSuffix: Microsoft identity platform
description: Сведения о конечной точке UserInfo на платформе Microsoft Identity.
services: active-directory
author: hpsin
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 09/21/2020
ms.author: hirsin
ms.reviewer: hirsin
ms.custom: aaddev
ms.openlocfilehash: 3bd0621d607b121f19bc47c717343b8e2e39b04f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98752730"
---
# <a name="microsoft-identity-platform-userinfo-endpoint"></a>Конечная точка UserInfo платформы идентификации Майкрософт

Конечная точка UserInfo входит в состав [OpenID Connect Connect Standard](https://openid.net/specs/openid-connect-core-1_0.html#UserInfo) (OIDC), которая предназначена для возврата утверждений о пользователе, который прошел проверку подлинности.  Для платформы Microsoft Identity конечная точка UserInfo размещается на Microsoft Graph ( https://graph.microsoft.com/oidc/userinfo) . 

## <a name="find-the-well-known-configuration-endpoint"></a>Поиск хорошо известной конечной точки конфигурации

Вы можете программно обнаружить конечную точку UserInfo с помощью документа OpenID Connect Connect Discovery по адресу `https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration` . Он указан в `userinfo_endpoint` поле, и этот шаблон можно использовать в облаках для указания правильной конечной точки.  Мы не рекомендуем жестко кодировать конечную точку UserInfo в приложении — используйте документ OIDC Discovery, чтобы найти эту конечную точку во время выполнения.

В рамках спецификации OpenID Connect Connect конечная точка UserInfo часто автоматически вызывается [библиотеками, совместимыми с OIDC](https://openid.net/developers/certified/)  , для получения сведений о пользователе.  Без размещения такой конечной точки платформа Microsoft Identity не будет соответствовать стандартам, а некоторые библиотеки завершатся бы ошибкой.  В [списке утверждений, определенных в OIDC Standard](https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims) , мы создаем утверждения имен, утверждения субъекта и сообщения электронной почты, если они доступны, и их можно отправить для.  

## <a name="consider-use-an-id-token-instead"></a>Рассмотрим: вместо этого используйте токен идентификатора.

Сведения, доступные в маркере идентификации, который может получить приложение, — это надмножество информации, которую он может получить из конечной точки UserInfo.  Так как вы можете получить маркер идентификации в то же время, когда вы получаете маркер для вызова конечной точки UserInfo, мы рекомендуем использовать этот маркер идентификации для получения сведений о пользователе вместо вызова конечной точки UserInfo.  Использование маркера идентификации приведет к исключению одного или двух сетевых запросов из запуска приложения, что сокращает задержку в приложении.

Если вам нужны дополнительные сведения о пользователе, следует вызвать [ `/user` API Microsoft Graph](/graph/api/user-get) , чтобы получить такие сведения, как номер или должность Office.   Кроме того, можно использовать [необязательные утверждения](active-directory-optional-claims.md) , чтобы включить дополнительные сведения о пользователях в идентификаторы и маркеры доступа.

## <a name="calling-the-userinfo-endpoint"></a>Вызов конечной точки UserInfo

UserInfo — это стандартный интерфейс API токена носителя OAuth, который вызывается как любой другой Microsoft Graph API с использованием маркера доступа, полученного при получении маркера для Microsoft Graph. Он возвращает ответ JSON, содержащий утверждения о пользователе.

### <a name="permissions"></a>Разрешения

Используйте следующие [разрешения OIDC](v2-permissions-and-consent.md#openid-connect-scopes) для вызова API UserInfo. `openid` является обязательным параметром, `profile` а `email` области и гарантируют, что в ответе предоставляются дополнительные сведения.

|Тип разрешения      | Разрешения    |
|:--------------------|:---------------------------------------------------------|
|Делегированная (Рабочая или учебная учетная запись) | OpenID Connect (обязательный), профиль, электронная почта |
|Делегировано (личное учетная запись Майкрософт) | OpenID Connect (обязательный), профиль, электронная почта |
|Приложение | Неприменимо |

> [!TIP]
> Скопируйте этот URL-адрес в браузере, чтобы получить маркер для конечной точки UserInfo, а также [маркер идентификации](id-tokens.md) и замените идентификатор клиента и URI перенаправления собственными. Обратите внимание, что он запрашивает только области для областей OpenID Connect или Graph и ничего другого.  Это необходимо, поскольку нельзя запрашивать разрешения для двух разных ресурсов в одном запросе токена.
>
> `https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=<yourClientID>&response_type=token+id_token&redirect_uri=<YourRedirectUri>&scope=user.read+openid+profile+email&response_mode=fragment&state=12345&nonce=678910`
>
> Этот маркер доступа можно использовать в следующем разделе.

Как и в случае с любым другим маркером Microsoft Graph, полученный здесь токен может не быть JWT. Если вы вошли в учетная запись Майкрософт пользователя, это будет формат зашифрованного маркера. Это обусловлено тем, что Microsoft Graph имеет специальный шаблон выдачи маркера. Это не влияет на возможность использования маркера доступа для вызова конечной точки UserInfo.

### <a name="calling-the-api"></a>Вызов интерфейса API

API UserInfo поддерживает как GET, так и POST, согласно спецификации OIDC.

```http
GET or POST /oidc/userinfo HTTP/1.1
Host: graph.microsoft.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJub25jZSI6Il…
```

### <a name="userinfo-response"></a>Ответ UserInfo

```jsonc
{
    "sub": "OLu859SGc2Sr9ZsqbkG-QbeLgJlb41KcdiPoLYNpSFA",
    "name": "Mikah Ollenburg", // names all require the “profile” scope.
    "family_name": " Ollenburg",
    "given_name": "Mikah",
    "email": "mikoll@contoso.com" //requires the “email” scope.
}
```

Указанные здесь утверждения — это все утверждения, которые может возвращать конечная точка UserInfo.  Это те же значения, которые приложение увидит в [маркере идентификации](id-tokens.md) , выданном приложению.  

## <a name="notes-and-caveats-on-the-userinfo-endpoint"></a>Примечания и предостережения для конечной точки UserInfo

* Если вы хотите вызвать эту конечную точку UserInfo, необходимо использовать конечную точку версии 2.0.  При использовании конечной точки версии 1.0 вы получите маркер для конечной точки UserInfo версии 1.0, размещенной на login.microsoftonline.com.  Рекомендуется, чтобы все приложения и библиотеки, совместимые с OIDC, использовали конечную точку версии 2.0 для обеспечения совместимости.
* Не удается настроить ответ от конечной точки UserInfo.  Если вы хотите настроить утверждения, используйте [сопоставление утверждений]( active-directory-claims-mapping.md) для изменения информации, возвращенной в токенах.
* Ответ от конечной точки UserInfo не может быть добавлен в.  Если вы хотите получить дополнительные утверждения о пользователе, используйте [дополнительные утверждения]( active-directory-optional-claims.md) , чтобы добавить новые утверждения в токены.

## <a name="next-steps"></a>Дальнейшие действия

* [Проверка содержимого токенов ИДЕНТИФИКАТОРов](id-tokens.md)
* [Настройка содержимого маркера идентификации с помощью необязательных утверждений](active-directory-optional-claims.md)
* [Запрос маркера доступа и маркера идентификации с помощью протокола OAuth2](v2-protocols-oidc.md)
