---
title: Защита API службы управления API Azure с помощью Azure Active Directory B2C
description: Узнайте, как использовать маркеры доступа, выданные Azure Active Directory B2C для защиты конечной точки API службы управления API Azure.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 07/30/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 60b7bb33dfbf29b7e448887ce992d03009133b2e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94953497"
---
# <a name="secure-an-azure-api-management-api-with-azure-ad-b2c"></a>Защита API службы управления API Azure с помощью Azure AD B2C

Узнайте, как ограничить доступ к API службы управления API Azure (APIM) для клиентов, которые прошли проверку подлинности с помощью Azure Active Directory B2C (Azure AD B2C). Выполните действия, описанные в этой статье, чтобы создать и протестировать политику входящего трафика в APIM, которая предоставляет доступ только к тем запросам, которые включают в себя действительный маркер доступа, выданный Azure AD B2C.

## <a name="prerequisites"></a>Предварительные требования

Для выполнения действий в этой статье необходимы следующие ресурсы.

* [клиент Azure AD B2C](tutorial-create-tenant.md);
* [приложение, зарегистрированное](tutorial-register-applications.md) в клиенте;
* [созданные маршруты пользователей](tutorial-create-user-flows.md) в клиенте.
* [Опубликованный API](../api-management/import-and-publish.md) в службе управления API Azure
* [Postman](https://www.getpostman.com/) для тестирования защищенного доступа (необязательно)

## <a name="get-azure-ad-b2c-application-id"></a>Получение идентификатора приложения Azure AD B2C

При защите API службы управления API Azure с помощью Azure AD B2C требуется несколько значений для [политики входящего трафика](../api-management/api-management-howto-policies.md), создаваемой в APIM. Сначала запишите идентификатор приложения, созданного ранее в клиенте Azure AD B2C. Если вы используете приложение, созданное в предварительных требованиях, используйте идентификатор приложения для *webbapp1*.

Чтобы зарегистрировать приложение в клиенте Azure AD B2C, можно использовать новый унифицированный интерфейс **Регистрации приложений** или устаревший интерфейс **Приложения (прежние версии)** . [См. дополнительные сведения о новом интерфейсе](./app-registrations-training-guide.md).

#### <a name="app-registrations"></a>[Регистрация приложений](#tab/app-reg-ga/)

1. Войдите на [портал Azure](https://portal.azure.com).
1. Выберите фильтр **Каталог и подписка** в верхнем меню, а затем выберите каталог, содержащий клиент Azure AD B2C.
1. В меню слева выберите **Azure AD B2C**. Либо щелкните **Все службы**, а затем найдите и выберите **Azure AD B2C**
1. Выберите **Регистрация приложений**, а затем перейдите на вкладку " **принадлежащие приложения** ".
1. Запишите значение в столбце **ИД приложения (клиент)** для *webapp1* или другого ранее созданного приложения.

#### <a name="applications-legacy"></a>[Приложения (прежняя версия)](#tab/applications-legacy/)

1. Войдите на [портал Azure](https://portal.azure.com).
1. Выберите фильтр **Каталог и подписка** в верхнем меню, а затем выберите каталог, содержащий клиент Azure AD B2C.
1. В меню слева выберите **Azure AD B2C**. Либо щелкните **Все службы**, а затем найдите и выберите **Azure AD B2C**
1. В разделе **Управление** выберите **приложения (прежние версии)**.
1. Запишите значение в столбце **ИД ПРИЛОЖЕНИЯ** для *webapp1* или другого ранее созданного приложения.

* * *

## <a name="get-token-issuer-endpoint"></a>Получение конечной точки издателя маркера

Затем получите известный URL-адрес конфигурации для одного из пользовательских потоков Azure AD B2C. Вам также понадобится URI конечной точки издателя маркера, который вы хотите поддерживать в службе управления API Azure.

1. Перейдите к клиенту Azure AD B2C на [портале Azure](https://portal.azure.com).
1. В разделе **Политики** выберите **Потоки пользователей**.
1. Выберите существующую политику, например *B2C_1_signupsignin1*, а затем нажмите **Запустить пользовательский поток**.
1. Запишите URL-адрес в гиперссылке, которая отображается под заголовком **Запустить пользовательский поток** в верхней части страницы. Этот URL-адрес является известной конечной точкой обнаружения OpenID Connect для пользовательского потока и используется в следующем разделе при настройке политики входящего трафика в службе управления API Azure.

    ![Ссылка на известный URI на странице "Запустить сейчас" на портале Azure](media/secure-apim-with-b2c-token/portal-01-policy-link.png)

1. Нажмите на гиперссылку, чтобы перейти на известную страницу конфигурации OpenID Connect.
1. На странице, которая открывается в браузере, запишите значение `issuer`, например:

    `https://<tenant-name>.b2clogin.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/v2.0/`

    Это значение используется в следующем разделе при настройке API в службе управления API Azure.

У вас должно быть записано два URL-адреса для следующего раздела: URL-адрес известной конечной точки конфигурации OpenID Connect и URI издателя. Пример:

```
https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/B2C_1_signupsignin1/v2.0/.well-known/openid-configuration
https://<tenant-name>.b2clogin.com/99999999-0000-0000-0000-999999999999/v2.0/
```

## <a name="configure-inbound-policy-in-azure-api-management"></a>Настройка политики входящего трафика в службе управления API Azure

Теперь все готово к добавлению политики входящего трафика в службе управления API Azure, которая проверяет вызовы API. Добавив политику [проверки JWT](../api-management/api-management-access-restriction-policies.md#ValidateJWT), которая проверяет аудиторию и издателя в маркере доступа, можно гарантировать, что принимаются только вызовы API с действительным маркером.

1. Перейдите к экземпляру управления API Azure на [портале Azure](https://portal.azure.com).
1. Выберите **Интерфейсы API**.
1. Выберите API, который требуется защитить с помощью Azure AD B2C.
1. Выберите вкладку **Конструктор**.
1. В разделе **Обработка входящего трафика** выберите **\</\>** , чтобы открыть редактор кода политики.
1. Поместите следующий тег `<validate-jwt>` в политику `<inbound>`.

    1. Обновите значение `url` в элементе `<openid-config>` с помощью известного URL-адреса конфигурации политики.
    1. Обновите элемент `<audience>`, указав идентификатор приложения, созданного ранее в клиенте B2C (например, *webapp1*).
    1. Обновите элемент `<issuer>`, указав конечную точку издателя маркера, записанную ранее.

    ```xml
    <policies>
        <inbound>
            <validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
                <openid-config url="https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/B2C_1_signupsignin1/v2.0/.well-known/openid-configuration" />
                <audiences>
                    <audience>44444444-0000-0000-0000-444444444444</audience>
                </audiences>
                <issuers>
                    <issuer>https://<tenant-name>.b2clogin.com/99999999-0000-0000-0000-999999999999/v2.0/</issuer>
                </issuers>
            </validate-jwt>
            <base />
        </inbound>
        <backend> <base /> </backend>
        <outbound> <base /> </outbound>
        <on-error> <base /> </on-error>
    </policies>
    ```

## <a name="validate-secure-api-access"></a>Проверка безопасного доступа к API

Чтобы убедиться, что доступ к API предоставляется только вызывающим объектам, прошедшим проверку подлинности, можно проверить конфигурацию службы управления API Azure, вызвав API с помощью [Postman](https://www.getpostman.com/).

Чтобы вызвать API, требуется маркер доступа, выданный Azure AD B2C, и ключ подписки APIM.

### <a name="get-an-access-token"></a>Получение маркера доступа

Сначала требуется маркер, выданный Azure AD B2C для использования в заголовке `Authorization` в Postman. Вы можете получить его, используя функцию **Запустить сейчас** в потоке регистрации или входа пользователя, который вы создали на этапе предварительных требований.

1. Перейдите к клиенту Azure AD B2C на [портале Azure](https://portal.azure.com).
1. В разделе **Политики** выберите **Потоки пользователей**.
1. Выберите существующий поток регистрации или входа пользователя, например *B2C_1_signupsignin1*.
1. Для параметра **Приложение** выберите *webapp1*.
1. Для параметра **URL-адрес ответа** выберите `https://jwt.ms`.
1. Выберите **Выполнить поток пользователя**.

    ![Страница запуска потока пользователей для потока регистрации или входа пользователей на портале Azure](media/secure-apim-with-b2c-token/portal-03-user-flow.png)

1. Завершите процесс входа в систему. Вы будете перенаправлены на страницу `https://jwt.ms`.
1. Запишите закодированное значение маркера, отображаемое в браузере. Это значение маркера используется для заголовка авторизации в Postman.

    ![Закодированное значение маркера, отображаемое в jwt.ms](media/secure-apim-with-b2c-token/jwt-ms-01-token.png)

### <a name="get-api-subscription-key"></a>Получение ключа подписки на API

Клиентское приложение (в данном случае Postman), которое вызывает опубликованный API, должно содержать допустимый ключ подписки службы управления API в HTTP-запросах к API. Чтобы получить ключ подписки для включения в HTTP-запрос Postman, выполните следующие действия.

1. На [портале Azure](https://portal.azure.com) перейдите к экземпляру службы управления API Azure.
1. Выберите **Подписки**.
1. Щелкните символ многоточия рядом с пунктом **Продукт: без ограничений**, выберите **Показать/скрыть ключи**.
1. Запишите **ПЕРВИЧНЫЙ КЛЮЧ** продукта. Этот ключ используется для заголовка `Ocp-Apim-Subscription-Key` в HTTP-запросе в Postman.

![Страница "Ключ подписки" с выбранным пунктом "Показать/скрыть ключи" на портале Azure](media/secure-apim-with-b2c-token/portal-04-api-subscription-key.png)

### <a name="test-a-secure-api-call"></a>Проверка безопасного вызова API

После регистрации маркера доступа и ключа подписки APIM вы можете проверить, правильно ли настроен безопасный доступ к API.

1. Создайте новый запрос `GET` в [Postman](https://www.getpostman.com/). В поле URL-адреса запроса укажите конечную точку списка докладчиков API, опубликованного в качестве предварительного требования. Пример:

    `https://contosoapim.azure-api.net/conference/speakers`

1. Затем добавьте следующие заголовки.

    | Клавиши | Значение |
    | --- | ----- |
    | `Authorization` | Записанное ранее закодированное значение маркера с префиксом `Bearer ` (после "Bearer" должен быть пробел) |
    | `Ocp-Apim-Subscription-Key` | Записанный ранее ключ подписки APIM |

    URL-адрес и **заголовки** запроса **GET** должны выглядеть следующим образом.

    ![Пользовательский интерфейс Postman с указанием URL-адреса и заголовков запроса GET](media/secure-apim-with-b2c-token/postman-01-headers.png)

1. Нажмите кнопку **Отправить** в Postman, чтобы выполнить запрос. Если все настроено правильно, вы должны получить ответ JSON с коллекцией докладчиков (ниже показана усеченная версия):

    ```json
    {
      "collection": {
        "version": "1.0",
        "href": "https://conferenceapi.azurewebsites.net:443/speakers",
        "links": [],
        "items": [
          {
            "href": "https://conferenceapi.azurewebsites.net/speaker/1",
            "data": [
              {
                "name": "Name",
                "value": "Scott Guthrie"
              }
            ],
            "links": [
              {
                "rel": "http://tavis.net/rels/sessions",
                "href": "https://conferenceapi.azurewebsites.net/speaker/1/sessions"
              }
            ]
          },
    [...]
    ```

### <a name="test-an-insecure-api-call"></a>Проверка небезопасного вызова API

Теперь, когда вы выполнили успешный запрос, проверьте случай сбоя, чтобы убедиться, что вызовы API с *недопустимым* маркером отклоняются, как и ожидалось. Один из способов выполнить тест — добавить или изменить несколько символов в значении маркера, а затем выполнить тот же запрос `GET`, как и раньше.

1. Добавьте несколько символов к значению токена, чтобы имитировать недопустимый токен. Например, добавьте "INVALID" в значение токена:

    ![Раздел заголовков в пользовательском интерфейсе Postman с маркером, к которому добавлено INVALID](media/secure-apim-with-b2c-token/postman-02-invalid-token.png)

1. Нажмите кнопку **Отправить**, чтобы выполнить запрос. При использовании недопустимого маркера ожидаемый результат является кодом состояния несанкционированного доступа `401`:

    ```json
    {
        "statusCode": 401,
        "message": "Unauthorized. Access token is missing or invalid."
    }
    ```

Если вы видите код состояния `401`, это значит, что только вызывающие объекты с действительным маркером доступа, выданным Azure AD B2C, могут успешно выполнять запросы к API службы управления API Azure.

## <a name="support-multiple-applications-and-issuers"></a>Поддержка нескольких приложений и издателей

Несколько приложений обычно взаимодействуют с одним REST API. Чтобы разрешить API принимать маркеры, предназначенные для нескольких приложений, добавьте идентификаторы приложений в элемент `<audiences>` в политике входящего трафика APIM.

```xml
<!-- Accept tokens intended for these recipient applications -->
<audiences>
    <audience>44444444-0000-0000-0000-444444444444</audience>
    <audience>66666666-0000-0000-0000-666666666666</audience>
</audiences>
```

Аналогичным образом для поддержки нескольких издателей маркеров добавьте их URI конечной точки в элемент `<issuers>` в политике входящего трафика APIM.

```xml
<!-- Accept tokens from multiple issuers -->
<issuers>
    <issuer>https://<tenant-name>.b2clogin.com/99999999-0000-0000-0000-999999999999/v2.0/</issuer>
    <issuer>https://login.microsoftonline.com/99999999-0000-0000-0000-999999999999/v2.0/</issuer>
</issuers>
```

## <a name="migrate-to-b2clogincom"></a>Миграция на b2clogin.com

Если у вас есть APIM API, который проверяет маркеры, выданные устаревшей конечной точкой `login.microsoftonline.com`, необходимо перенести API и приложения, вызывающие его, чтобы использовать маркеры, выданные [b2clogin.com](b2clogin.md).

Для выполнения поэтапной миграции можно использовать следующую общую процедуру.

1. Добавьте поддержку в политику входящего трафика APIM для маркеров, выдаваемых как b2clogin.com, так и login.microsoftonline.com.
1. Обновите приложения по одному за раз, чтобы получить маркеры из конечной точки b2clogin.com.
1. Когда все приложения будут правильно получать маркеры от b2clogin.com, удалите поддержку маркеров, выданных login.microsoftonline.com, из API.

В следующем примере политики входящего трафика APIM показано, как принимать маркеры, выданные b2clogin.com и login.microsoftonline.com. Кроме того, она поддерживает запросы API из двух приложений.

```xml
<policies>
    <inbound>
        <validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
            <openid-config url="https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/B2C_1_signupsignin1/v2.0/.well-known/openid-configuration" />
            <audiences>
                <audience>44444444-0000-0000-0000-444444444444</audience>
                <audience>66666666-0000-0000-0000-666666666666</audience>
            </audiences>
            <issuers>
                <issuer>https://login.microsoftonline.com/99999999-0000-0000-0000-999999999999/v2.0/</issuer>
                <issuer>https://<tenant-name>.b2clogin.com/99999999-0000-0000-0000-999999999999/v2.0/</issuer>
            </issuers>
        </validate-jwt>
        <base />
    </inbound>
    <backend> <base /> </backend>
    <outbound> <base /> </outbound>
    <on-error> <base /> </on-error>
</policies>
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о политиках управления API Azure см. в [указателе справочника по политике APIM](../api-management/api-management-policies.md).

Сведения о миграции веб-API на основе OWIN и их приложений в b2clogin.com можно найти в разделе [Миграция веб-API на основе OWIN в b2clogin.com](multiple-token-endpoints.md).