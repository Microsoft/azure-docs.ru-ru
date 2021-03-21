---
title: Аутентификация между службами с помощью потока On-Behalf-Of в OAuth 2.0 | Документация Майкрософт
description: В этой статье описывается, как использовать HTTP-сообщения для проверки подлинности между службами с помощью потока On-Behalf-Of в OAuth 2.0.
services: active-directory
documentationcenter: .net
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: azuread-dev
ms.workload: identity
ms.topic: conceptual
ms.date: 08/5/2020
ms.author: ryanwi
ms.reviewer: hirsin, nacanuma
ms.custom: aaddev
ROBOTS: NOINDEX
ms.openlocfilehash: f746cc654934464d907c6ad669eb7470e4dcaeeb
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88117742"
---
# <a name="service-to-service-calls-that-use-delegated-user-identity-in-the-on-behalf-of-flow"></a>Вызовы между службами с использованием делегированного удостоверения пользователя в потоке On-Behalf-Of

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

Поток On-Behalf-Of (OBO) в OAuth 2.0 позволяет приложению, вызывающему службу или веб-API, поручать проверку подлинности пользователей другой службе или веб-API. Поток OBO распространяет делегированное удостоверение пользователя и разрешения с помощью цепочки запросов. Для того чтобы служба среднего уровня могла выполнять запросы к нижестоящей службе с проверкой подлинности, служба среднего уровня должна защитить токен доступа из Azure Active Directory (Azure AD) от имени пользователя.

> [!IMPORTANT]
> Начиная с мая 2018 г., `id_token` невозможно использовать для потока On-Behalf-Of.  Одностраничные приложения (SPA) должны передавать свои маркеры доступа в конфиденциальный клиент среднего уровня для выполнения потоков OBO. Дополнительные сведения о том, как клиенты могут выполнять вызовы On-Behalf-Of, см. в разделе [Ограничения](#client-limitations).

## <a name="on-behalf-of-flow-diagram"></a>Схема потока On-Behalf-Of

Поток OBO начинается после того, как пользователь пройдет проверку подлинности для приложения с использованием [потока для предоставления кода проверки подлинности в OAuth 2.0](v1-protocols-oauth-code.md). На этом этапе приложение отправляет маркер доступа (токен A) в веб-API среднего уровня (API A), содержащий утверждения и согласие пользователя на доступ к API A. Затем API A может отправить аутентифицированный запрос в подчиненный веб-API (API B).

Поток On-Behalf-Of состоит из следующих этапов: ![Этапы потока OBO в OAuth 2.0](./media/v1-oauth2-on-behalf-of-flow/active-directory-protocols-oauth-on-behalf-of-flow.png)

1. Клиентское приложение отправляет запрос к API A с токеном A.
1. API A выполняет проверку подлинности на конечной точке выдачи токена Azure AD и запрашивает токен доступа к API B.
1. Конечная точка выдачи токена Azure AD проверяет учетные данные API A с помощью токена A и выдает токен доступа для API B (токен B).
1. Запрос к API B содержит токен B в заголовке авторизации.
1. API B возвращает данные из защищенного ресурса.

>[!NOTE]
>Утверждение аудитории в маркере доступа, используемое для запроса токена из нижестоящей службы, должно быть идентификатором службы, совершающей запрос OBO. Токен также должен быть подписан глобальным ключом Azure Active Directory (используемым по умолчанию для приложений, зарегистрированных с помощью страницы **Регистрация приложений** на портале).

## <a name="register-the-application-and-service-in-azure-ad"></a>Регистрация приложения и службы в Azure AD

Как служба среднего уровня, так и клиентское приложение регистрируются в Azure AD.

### <a name="register-the-middle-tier-service"></a>Регистрация службы среднего уровня

1. Войдите на [портал Azure](https://portal.azure.com).
1. На верхней панели выберите учетную запись и в списке **Каталог** выберите клиент Active Directory для вашего приложения.
1. В области слева нажмите **Больше служб** и выберите **Azure Active Directory**.
1. Щелкните **Регистрация приложений** и выберите **Новая регистрация**.
1. Введите понятное имя для приложения и укажите тип приложения.
1. В разделе **Поддерживаемые типы учетных записей** выберите **Accounts in any organizational directory and personal Microsoft accounts** (Учетные записи в любом каталоге организации и личные учетные записи Майкрософт).
1. Введите для URI перенаправления значение базового URL-адреса.
1. Выберите **Зарегистрировать**, чтобы создать приложение.
1. На портале Azure выберите свое приложение и нажмите **Сертификаты и секреты**.
1. Выберите **Новый секрет клиента** и добавьте секрет с длительностью в один или два года.
1. При сохранении этой страницы на портале Azure отображается значение секрета. Скопируйте и сохраните значение секрета в безопасном месте.
1. Создайте область в приложении на странице **предоставление API** для приложения и щелкните "добавить область".  На портале также может потребоваться создать URI идентификатора приложения. 

> [!IMPORTANT]
> Секрет необходим для настройки параметров приложений в вашей реализации. Это значение секрета больше не будет отображаться, и его невозможно получить каким-либо другим способом. Запишите его, как только он появится на портале Azure.

### <a name="register-the-client-application"></a>Регистрация клиентского приложения

1. Войдите на [портал Azure](https://portal.azure.com).
1. На верхней панели выберите учетную запись и в списке **Каталог** выберите клиент Active Directory для вашего приложения.
1. В области слева нажмите **Больше служб** и выберите **Azure Active Directory**.
1. Щелкните **Регистрация приложений** и выберите **Новая регистрация**.
1. Введите понятное имя для приложения и укажите тип приложения.
1. В разделе **Поддерживаемые типы учетных записей** выберите **Accounts in any organizational directory and personal Microsoft accounts** (Учетные записи в любом каталоге организации и личные учетные записи Майкрософт).
1. Введите для URI перенаправления значение базового URL-адреса.
1. Выберите **Зарегистрировать**, чтобы создать приложение.
1. Настройте разрешения для приложения. В разделе **Разрешения API** выберите **Добавить разрешение**, **Мои API**.
1. Введите имя службы среднего уровня в текстовом поле.
1. Щелкните **выбрать разрешения** , а затем выберите область, созданную на последнем шаге регистрации среднего уровня.

### <a name="configure-known-client-applications"></a>Настройка известных клиентских приложений

В этом сценарии службе среднего уровня необходимо получить согласие пользователя на доступ к нижестоящему API без участия пользователя. Согласие на доступ к нижестоящему API необходимо получить заранее при принятии условий во время проверки подлинности.

Выполните указанные ниже действия, чтобы явно привязать регистрацию клиентского приложения в Azure AD к регистрации службы среднего уровня. Эта операция объединяет согласие, необходимое клиенту и среднему уровню, в одном диалоговом окне.

1. Перейдите к регистрации службы среднего уровня и выберите **Манифест**, чтобы открыть редактор манифеста.
1. Найдите массив свойств `knownClientApplications` и добавьте идентификатор клиента для клиентского приложения в качестве одного из элементов массива.
1. Сохраните манифест, нажав кнопку **Сохранить**.

## <a name="service-to-service-access-token-request"></a>Запрос маркера взаимного доступа между службами

Чтобы запросить токен доступа, отправьте запрос HTTP POST к конечной точке Azure AD для конкретного клиента с указанными параметрами.

```
https://login.microsoftonline.com/<tenant>/oauth2/token
```

Клиентское приложение защищается либо общим секретом, либо сертификатом.

### <a name="first-case-access-token-request-with-a-shared-secret"></a>Первый вариант. Запрос маркера доступа с помощью общего секрета

При использовании общего секрета запрос маркера взаимного доступа между службами содержит следующие параметры:

| Параметр | Type | Описание |
| --- | --- | --- |
| grant_type |обязательно | Тип запроса токена. Запрос OBO использует JSON Web Token (JWT), поэтому необходимо задать значение **urn:ietf:params:oauth:grant-type:jwt-bearer**. |
| assertion |обязательно | Значение маркера доступа, используемого в запросе. |
| client_id |обязательно | Идентификатор приложения, назначенный вызывающей службе при регистрации в Azure AD. Чтобы узнать идентификатор приложения на портале Azure, нажмите **Active Directory**, выберите каталог, а затем щелкните имя приложения. |
| client_secret |обязательно | Ключ, зарегистрированный для вызывающей службы в Azure AD. Значение ключа должно было быть записано при регистрации. |
| ресурс |обязательно | URI идентификатора приложения принимающей службы (защищенный ресурс). Чтобы узнать URI идентификатора приложения на портале Azure, нажмите **Active Directory** и выберите каталог. Щелкните имя приложения, выберите **Все параметры**, а затем выберите пункт **Свойства**. |
| requested_token_use |обязательно | Указывает, как должен быть обработан запрос. Для потока On-Behalf-Of это значение должно быть равно **on_behalf_of**. |
| область |обязательно | Список областей для запроса токена, разделенный пробелами. Для OpenID Connect необходимо указать область **openid**.|

#### <a name="example"></a>Пример

Ниже приведен HTTP-запрос POST маркера доступа для веб-API https://graph.microsoft.com. Параметр `client_id` определяет службу, которая запрашивает токен доступа.

```
// line breaks for legibility only

POST /oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Ajwt-bearer
&client_id=625391af-c675-43e5-8e44-edd3e30ceb15
&client_secret=0Y1W%2BY3yYb3d9N8vSjvm8WrGzVZaAaHbHHcGbcgG%2BoI%3D
&resource=https%3A%2F%2Fgraph.microsoft.com
&assertion=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.ewogICJhdWQiOiAiaHR0cHM6Ly9ncmFwaC5taWNyb3NvZnQuY29tIiwKICAiaXNzIjogImh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzI2MDM5Y2NlLTQ4OWQtNDAwMi04MjkzLTViMGM1MTM0ZWFjYi8iLAogICJpYXQiOiAxNDkzNDIzMTY4LAogICJuYmYiOiAxNDkzNDIzMTY4LAogICJleHAiOiAxNDkzNDY2OTUxLAogICJhY3IiOiAiMSIsCiAgImFpbyI6ICJBU1FBMi84REFBQUE1NnZGVmp0WlNjNWdBVWwrY1Z0VFpyM0VvV2NvZEoveWV1S2ZqcTZRdC9NPSIsCiAgImFtciI6IFsKICAgICJwd2QiCiAgXSwKICAiYXBwaWQiOiAiNjI1MzkxYWYtYzY3NS00M2U1LThlNDQtZWRkM2UzMGNlYjE1IiwKICAiYXBwaWRhY3IiOiAiMSIsCiAgImVfZXhwIjogMzAyNjgzLAogICJmYW1pbHlfbmFtZSI6ICJUZXN0IiwKICAiZ2l2ZW5fbmFtZSI6ICJOYXZ5YSIsCiAgImlwYWRkciI6ICIxNjcuMjIwLjEuMTc3IiwKICAibmFtZSI6ICJOYXZ5YSBUZXN0IiwKICAib2lkIjogIjFjZDRiY2FjLWI4MDgtNDIzYS05ZTJmLTgyN2ZiYjFiYjczOSIsCiAgInBsYXRmIjogIjMiLAogICJwdWlkIjogIjEwMDMzRkZGQTEyRUQ3RkUiLAogICJzY3AiOiAiVXNlci5SZWFkIiwKICAic3ViIjogIjNKTUlaSWJlYTc1R2hfWHdDN2ZzX0JDc3kxa1l1ekZKLTUyVm1Zd0JuM3ciLAogICJ0aWQiOiAiMjYwMzljY2UtNDg5ZC00MDAyLTgyOTMtNWIwYzUxMzRlYWNiIiwKICAidW5pcXVlX25hbWUiOiAibmF2eWFAZGRvYmFsaWFub3V0bG9vay5vbm1pY3Jvc29mdC5jb20iLAogICJ1cG4iOiAibmF2eWFAZGRvYmFsaWFub3V0bG9vay5vbm1pY3Jvc29mdC5jb20iLAogICJ1dGkiOiAieEN3ZnpoYS1QMFdKUU9MeENHZ0tBQSIsCiAgInZlciI6ICIxLjAiCn0.cqmUVjfVbqWsxJLUI1Z4FRx1mNQAHP-L0F4EMN09r8FY9bIKeO-0q1eTdP11Nkj_k4BmtaZsTcK_mUygdMqEp9AfyVyA1HYvokcgGCW_Z6DMlVGqlIU4ssEkL9abgl1REHElPhpwBFFBBenOk9iHddD1GddTn6vJbKC3qAaNM5VarjSPu50bVvCrqKNvFixTb5bbdnSz-Qr6n6ACiEimiI1aNOPR2DeKUyWBPaQcU5EAK0ef5IsVJC1yaYDlAcUYIILMDLCD9ebjsy0t9pj_7lvjzUSrbMdSCCdzCqez_MSNxrk1Nu9AecugkBYp3UVUZOIyythVrj6-sVvLZKUutQ
&requested_token_use=on_behalf_of
&scope=openid
```

### <a name="second-case-access-token-request-with-a-certificate"></a>Второй вариант. Запрос маркера доступа с помощью сертификата

Запрос маркера взаимного доступа между службами с помощью сертификата содержит следующие параметры:

| Параметр | Type | Описание |
| --- | --- | --- |
| grant_type |обязательно | Тип запроса токена. Запрос OBO использует маркер доступа JWT, поэтому необходимо задать значение **urn:ietf:params:oauth:grant-type:jwt-bearer**. |
| assertion |обязательно | Значение токена, используемого в запросе. |
| client_id |обязательно | Идентификатор приложения, назначенный вызывающей службе при регистрации в Azure AD. Чтобы узнать идентификатор приложения на портале Azure, нажмите **Active Directory**, выберите каталог, а затем щелкните имя приложения. |
| client_assertion_type |обязательно |Значение должно быть `urn:ietf:params:oauth:client-assertion-type:jwt-bearer`. |
| client_assertion |обязательно | Маркер JSON Web Token, который создается и подписывается с помощью сертификата, зарегистрированного как учетные данные для приложения. Сведения о формате утверждений и о том, как зарегистрировать сертификат, см. в статье [Учетные данные сертификата](../develop/active-directory-certificate-credentials.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).|
| ресурс |обязательно | URI идентификатора приложения принимающей службы (защищенный ресурс). Чтобы узнать URI идентификатора приложения на портале Azure, нажмите **Active Directory** и выберите каталог. Щелкните имя приложения, выберите **Все параметры**, а затем выберите пункт **Свойства**. |
| requested_token_use |обязательно | Указывает, как должен быть обработан запрос. Для потока On-Behalf-Of это значение должно быть равно **on_behalf_of**. |
| область |обязательно | Список областей для запроса токена, разделенный пробелами. Для OpenID Connect необходимо указать область **openid**.|

Эти параметры будут практически такими же, как и для запроса по общему секрету, но параметр `client_secret parameter` заменяется двумя параметрами: `client_assertion_type` и `client_assertion`.

#### <a name="example"></a>Пример

Ниже приведен HTTP-запрос POST маркера доступа для веб-API https://graph.microsoft.com с сертификатом. Параметр `client_id` определяет службу, которая запрашивает токен доступа.

```
// line breaks for legibility only

POST /oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Ajwt-bearer
&client_id=625391af-c675-43e5-8e44-edd3e30ceb15
&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer
&client_assertion=eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJ{a lot of characters here}M8U3bSUKKJDEg
&resource=https%3A%2F%2Fgraph.microsoft.com
&assertion=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.eyJhdWQiOiJodHRwczovL2Rkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tLzE5MjNmODYyLWU2ZGMtNDFhMy04MWRhLTgwMmJhZTAwYWY2ZCIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzI2MDM5Y2NlLTQ4OWQtNDAwMi04MjkzLTViMGM1MTM0ZWFjYi8iLCJpYXQiOjE0OTM0MjMxNTIsIm5iZiI6MTQ5MzQyMzE1MiwiZXhwIjoxNDkzNDY2NjUyLCJhY3IiOiIxIiwiYWlvIjoiWTJaZ1lCRFF2aTlVZEc0LzM0L3dpQndqbjhYeVp4YmR1TFhmVE1QeG8yYlN2elgreHBVQSIsImFtciI6WyJwd2QiXSwiYXBwaWQiOiJiMzE1MDA3OS03YmViLTQxN2YtYTA2YS0zZmRjNzhjMzI1NDUiLCJhcHBpZGFjciI6IjAiLCJlX2V4cCI6MzAyNDAwLCJmYW1pbHlfbmFtZSI6IlRlc3QiLCJnaXZlbl9uYW1lIjoiTmF2eWEiLCJpcGFkZHIiOiIxNjcuMjIwLjEuMTc3IiwibmFtZSI6Ik5hdnlhIFRlc3QiLCJvaWQiOiIxY2Q0YmNhYy1iODA4LTQyM2EtOWUyZi04MjdmYmIxYmI3MzkiLCJwbGF0ZiI6IjMiLCJzY3AiOiJ1c2VyX2ltcGVyc29uYXRpb24iLCJzdWIiOiJEVXpYbkdKMDJIUk0zRW5pbDFxdjZCakxTNUllQy0tQ2ZpbzRxS1MzNEc4IiwidGlkIjoiMjYwMzljY2UtNDg5ZC00MDAyLTgyOTMtNWIwYzUxMzRlYWNiIiwidW5pcXVlX25hbWUiOiJuYXZ5YUBkZG9iYWxpYW5vdXRsb29rLm9ubWljcm9zb2Z0LmNvbSIsInVwbiI6Im5hdnlhQGRkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tIiwidmVyIjoiMS4wIn0.R-Ke-XO7lK0r5uLwxB8g5CrcPAwRln5SccJCfEjU6IUqpqcjWcDzeDdNOySiVPDU_ZU5knJmzRCF8fcjFtPsaA4R7vdIEbDuOur15FXSvE8FvVSjP_49OH6hBYqoSUAslN3FMfbO6Z8YfCIY4tSOB2I6ahQ_x4ZWFWglC3w5mK-_4iX81bqi95eV4RUKefUuHhQDXtWhrSgIEC0YiluMvA4TnaJdLq_tWXIc4_Tq_KfpkvI004ONKgU7EAMEr1wZ4aDcJV2yf22gQ1sCSig6EGSTmmzDuEPsYiyd4NhidRZJP4HiiQh-hePBQsgcSgYGvz9wC6n57ufYKh2wm_Ti3Q
&requested_token_use=on_behalf_of
&scope=openid
```

## <a name="service-to-service-access-token-response"></a>Отклик с маркером доступа между службами

В случае успеха возвращается отклик JSON OAuth 2.0 со следующими параметрами:

| Параметр | Описание |
| --- | --- |
| token_type |Указывает значение типа маркера. Единственный тип, поддерживаемый Azure AD — **носитель**. Дополнительные сведения о маркерах носителей см. в спецификации [OAuth 2.0 Authorization Framework: Bearer Token Usage (RFC 6750)](https://www.rfc-editor.org/rfc/rfc6750.txt) (OAuth2.0 Authorization Framework: использование маркера носителя (RFC 6750)). |
| область |Область доступа, предоставляемая токеном. |
| expires_in |Срок действия доступа для токена (в секундах). |
| expires_on |Время истечения срока действия маркера доступа. Дата представляется как количество секунд с 1970-01-01T0:0:0Z в формате UTC до истечения срока действия. Это значение используется для определения времени существования кэшированных маркеров. |
| ресурс |URI идентификатора приложения принимающей службы (защищенный ресурс). |
| access_token |Запрашиваемый маркер доступа. Вызывающая служба может использовать этот токен для проверки подлинности принимающей службы. |
| id_token |Запрашиваемый токен удостоверения. Вызывающая служба может использовать этот токен для проверки удостоверения пользователя и запуска сеанса пользователя. |
| refresh_token |Токен обновления для запрошенного токена доступа. Вызывающая служба может использовать этот токен для запроса другого токена доступа после того, как срок действия текущего токена доступа истек. |

### <a name="success-response-example"></a>Пример ответа с успешным предоставлением доступа

В следующем примере показано сообщение о предоставлении доступа в ответ на запрос маркера доступа к веб-API https://graph.microsoft.com.

```json
{
    "token_type":"Bearer",
    "scope":"User.Read",
    "expires_in":"43482",
    "ext_expires_in":"302683",
    "expires_on":"1493466951",
    "not_before":"1493423168",
    "resource":"https://graph.microsoft.com",
    "access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.eyJhdWQiOiJodHRwczovL2dyYXBoLndpbmRvd3MubmV0IiwiaXNzIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvMjYwMzljY2UtNDg5ZC00MDAyLTgyOTMtNWIwYzUxMzRlYWNiLyIsImlhdCI6MTQ5MzQyMzE2OCwibmJmIjoxNDkzNDIzMTY4LCJleHAiOjE0OTM0NjY5NTEsImFjciI6IjEiLCJhaW8iOiJBU1FBMi84REFBQUE1NnZGVmp0WlNjNWdBVWwrY1Z0VFpyM0VvV2NvZEoveWV1S2ZqcTZRdC9NPSIsImFtciI6WyJwd2QiXSwiYXBwaWQiOiI2MjUzOTFhZi1jNjc1LTQzZTUtOGU0NC1lZGQzZTMwY2ViMTUiLCJhcHBpZGFjciI6IjEiLCJlX2V4cCI6MzAyNjgzLCJmYW1pbHlfbmFtZSI6IlRlc3QiLCJnaXZlbl9uYW1lIjoiTmF2eWEiLCJpcGFkZHIiOiIxNjcuMjIwLjEuMTc3IiwibmFtZSI6Ik5hdnlhIFRlc3QiLCJvaWQiOiIxY2Q0YmNhYy1iODA4LTQyM2EtOWUyZi04MjdmYmIxYmI3MzkiLCJwbGF0ZiI6IjMiLCJwdWlkIjoiMTAwMzNGRkZBMTJFRDdGRSIsInNjcCI6IlVzZXIuUmVhZCIsInN1YiI6IjNKTUlaSWJlYTc1R2hfWHdDN2ZzX0JDc3kxa1l1ekZKLTUyVm1Zd0JuM3ciLCJ0aWQiOiIyNjAzOWNjZS00ODlkLTQwMDItODI5My01YjBjNTEzNGVhY2IiLCJ1bmlxdWVfbmFtZSI6Im5hdnlhQGRkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tIiwidXBuIjoibmF2eWFAZGRvYmFsaWFub3V0bG9vay5vbm1pY3Jvc29mdC5jb20iLCJ1dGkiOiJ4Q3dmemhhLVAwV0pRT0x4Q0dnS0FBIiwidmVyIjoiMS4wIn0.cqmUVjfVbqWsxJLUI1Z4FRx1mNQAHP-L0F4EMN09r8FY9bIKeO-0q1eTdP11Nkj_k4BmtaZsTcK_mUygdMqEp9AfyVyA1HYvokcgGCW_Z6DMlVGqlIU4ssEkL9abgl1REHElPhpwBFFBBenOk9iHddD1GddTn6vJbKC3qAaNM5VarjSPu50bVvCrqKNvFixTb5bbdnSz-Qr6n6ACiEimiI1aNOPR2DeKUyWBPaQcU5EAK0ef5IsVJC1yaYDlAcUYIILMDLCD9ebjsy0t9pj_7lvjzUSrbMdSCCdzCqez_MSNxrk1Nu9AecugkBYp3UVUZOIyythVrj6-sVvLZKUutQ",
    "refresh_token":"AQABAAAAAABnfiG-mA6NTae7CdWW7QfdjKGu9-t1scy_TDEmLi4eLQMjJGt_nAoVu6A4oSu1KsRiz8XyQIPKQxSGfbf2FoSK-hm2K8TYzbJuswYusQpJaHUQnSqEvdaCeFuqXHBv84wjFhuanzF9dQZB_Ng5za9xKlUENrNtlq9XuLNVKzxEyeUM7JyxzdY7JiEphWImwgOYf6II316d0Z6-H3oYsFezf4Xsjz-MOBYEov0P64UaB5nJMvDyApV-NWpgklLASfNoSPGb67Bc02aFRZrm4kLk-xTl6eKE6hSo0XU2z2t70stFJDxvNQobnvNHrAmBaHWPAcC3FGwFnBOojpZB2tzG1gLEbmdROVDp8kHEYAwnRK947Py12fJNKExUdN0njmXrKxNZ_fEM33LHW1Tf4kMX_GvNmbWHtBnIyG0w5emb-b54ef5AwV5_tGUeivTCCysgucEc-S7G8Cz0xNJ_BOiM_4bAv9iFmrm9STkltpz0-Tftg8WKmaJiC0xXj6uTf4ZkX79mJJIuuM7XP4ARIcLpkktyg2Iym9jcZqymRkGH2Rm9sxBwC4eeZXM7M5a7TJ-5CqOdfuE3sBPq40RdEWMFLcrAzFvP0VDR8NKHIrPR1AcUruat9DETmTNJukdlJN3O41nWdZOVoJM-uKN3uz2wQ2Ld1z0Mb9_6YfMox9KTJNzRzcL52r4V_y3kB6ekaOZ9wQ3HxGBQ4zFt-2U0mSszIAA",
    "id_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiI2MjUzOTFhZi1jNjc1LTQzZTUtOGU0NC1lZGQzZTMwY2ViMTUiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC8yNjAzOWNjZS00ODlkLTQwMDItODI5My01YjBjNTEzNGVhY2IvIiwiaWF0IjoxNDkzNDIzMTY4LCJuYmYiOjE0OTM0MjMxNjgsImV4cCI6MTQ5MzQ2Njk1MSwiYW1yIjpbInB3ZCJdLCJmYW1pbHlfbmFtZSI6IlRlc3QiLCJnaXZlbl9uYW1lIjoiTmF2eWEiLCJpcGFkZHIiOiIxNjcuMjIwLjEuMTc3IiwibmFtZSI6Ik5hdnlhIFRlc3QiLCJvaWQiOiIxY2Q0YmNhYy1iODA4LTQyM2EtOWUyZi04MjdmYmIxYmI3MzkiLCJwbGF0ZiI6IjMiLCJzdWIiOiJEVXpYbkdKMDJIUk0zRW5pbDFxdjZCakxTNUllQy0tQ2ZpbzRxS1MzNEc4IiwidGlkIjoiMjYwMzljY2UtNDg5ZC00MDAyLTgyOTMtNWIwYzUxMzRlYWNiIiwidW5pcXVlX25hbWUiOiJuYXZ5YUBkZG9iYWxpYW5vdXRsb29rLm9ubWljcm9zb2Z0LmNvbSIsInVwbiI6Im5hdnlhQGRkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tIiwidXRpIjoieEN3ZnpoYS1QMFdKUU9MeENHZ0tBQSIsInZlciI6IjEuMCJ9."
}
```

### <a name="error-response-example"></a>Пример ответа с сообщением об ошибке

Конечная точка токена Azure AD возвращает ответ с ошибкой при попытке получить маркер доступа для нижестоящего API, настроенного с помощью политики условного доступа (например, многофакторной проверки подлинности). Служба среднего уровня должна передать эту ошибку в клиентское приложение, чтобы пользователь мог выполнить необходимые действия для соблюдения политики условного доступа.

```json
{
    "error":"interaction_required",
    "error_description":"AADSTS50079: Due to a configuration change made by your administrator, or because you moved to a new location, you must enroll in multi-factor authentication to access 'bf8d80f9-9098-4972-b203-500f535113b1'.\r\nTrace ID: b72a68c3-0926-4b8e-bc35-3150069c2800\r\nCorrelation ID: 73d656cf-54b1-4eb2-b429-26d8165a52d7\r\nTimestamp: 2017-05-01 22:43:20Z",
    "error_codes":[50079],
    "timestamp":"2017-05-01 22:43:20Z",
    "trace_id":"b72a68c3-0926-4b8e-bc35-3150069c2800",
    "correlation_id":"73d656cf-54b1-4eb2-b429-26d8165a52d7",
    "claims":"{\"access_token\":{\"polids\":{\"essential\":true,\"values\":[\"9ab03e19-ed42-4168-b6b7-7001fb3e933a\"]}}}"
}
```

## <a name="use-the-access-token-to-access-the-secured-resource"></a>Использование токена доступа для доступа к защищенному ресурсу

Служба среднего уровня может использовать полученный маркер доступа для запросов к нижестоящему веб-API с проверкой подлинности путем установки токена в заголовке `Authorization`.

### <a name="example"></a>Пример

```
GET /me?api-version=2013-11-08 HTTP/1.1
Host: graph.microsoft.com
Authorization: Bearer eyJ0eXAiO ... 0X2tnSQLEANnSPHY0gKcgw
```

## <a name="saml-assertions-obtained-with-an-oauth20-obo-flow"></a>Утверждения SAML, полученные с помощью потока OBO в OAuth2.0

Некоторые веб-службы на основе OAuth должны получать доступ к другим API веб-служб, получающим утверждения SAML в неинтерактивных потоках. Azure Active Directory может предоставить утверждение SAML в ответ на поток On-Behalf-Of с использованием веб-службы на основе SAML в качестве целевого ресурса.

>[!NOTE]
>Это нестандартное расширение для потока On-Behalf-Of в OAuth 2.0, позволяющее приложению на основе OAuth2 получать доступ к конечным точкам API веб-служб, использующим токены SAML.

> [!TIP]
> Чтобы вызвать защищенную с помощью SAML веб-службу из веб-приложения переднего плана, достаточно вызвать API и инициировать обычный интерактивный поток проверки подлинности в имеющемся сеансе пользователя. Если для предоставления контекста пользователя вызову между службами необходим токен SAML, то достаточно использовать поток OBO.

### <a name="obtain-a-saml-token-by-using-an-obo-request-with-a-shared-secret"></a>Получение токена SAML при помощи запроса OBO с общим секретом

Запрос утверждения SAML между службами содержит следующие параметры:

| Параметр | Type | Описание |
| --- | --- | --- |
| grant_type |обязательно | Тип запроса токена. Для запроса с использованием JWT это значение должно быть равно **urn:ietf:params:oauth:grant-type:jwt-bearer**. |
| assertion |обязательно | Значение маркера доступа, используемого в запросе.|
| client_id |обязательно | Идентификатор приложения, назначенный вызывающей службе при регистрации в Azure AD. Чтобы узнать идентификатор приложения на портале Azure, нажмите **Active Directory**, выберите каталог, а затем щелкните имя приложения. |
| client_secret |обязательно | Ключ, зарегистрированный для вызывающей службы в Azure AD. Значение ключа должно было быть записано при регистрации. |
| ресурс |обязательно | URI идентификатора приложения принимающей службы (защищенный ресурс). Это ресурс, который будет аудиторией токена SAML. Чтобы узнать URI идентификатора приложения на портале Azure, нажмите **Active Directory** и выберите каталог. Щелкните имя приложения, выберите **Все параметры**, а затем выберите пункт **Свойства**. |
| requested_token_use |обязательно | Указывает, как должен быть обработан запрос. Для потока On-Behalf-Of это значение должно быть равно **on_behalf_of**. |
| requested_token_type | обязательно | Задает тип запрашиваемого токена. Можно задать значение **urn:ietf:params:oauth:token-type:saml2** или **urn:ietf:params:oauth:token-type:saml1** в зависимости от требований запрашиваемого ресурса. |

Отклик содержит токен SAML с кодировками UTF8 и Base64url.

- **SubjectConfirmationData для утверждения SAML, полученного из вызова OBO.** Если целевому приложению необходимо значение получателя в параметре **SubjectConfirmationData**, то значением должен быть URL-адрес ответа без подстановочных знаков в конфигурации приложения-ресурса.
- **Узел SubjectConfirmationData.** Узел не может содержать атрибут **InResponseTo**, так как он не входит в состав отклика SAML. Приложение, получающее токен SAML, должно иметь возможность принимать утверждение SAML без атрибута **InResponseTo**.

- **Согласие.** Должно быть предоставлено согласие на получение токена SAML, содержащего данные пользователей в потоке OAuth. Сведения о разрешениях и получении согласия администратора см. в статье [Разрешения и согласие в конечной точке Azure Active Directory версии 1.0](./v1-permissions-consent.md).

### <a name="response-with-saml-assertion"></a>Отклик с утверждением SAML

| Параметр | Описание |
| --- | --- |
| token_type |Указывает значение типа маркера. Единственный тип, поддерживаемый Azure AD — **носитель**. Дополнительные сведения о маркерах носителей см. в спецификации [OAuth 2.0 Authorization Framework: Bearer Token Usage (RFC 6750)](https://www.rfc-editor.org/rfc/rfc6750.txt) (OAuth2.0 Authorization Framework: использование маркера носителя (RFC 6750)). |
| область |Область доступа, предоставляемая токеном. |
| expires_in |Срок действия доступа для токена (в секундах). |
| expires_on |Время истечения срока действия маркера доступа. Дата представляется как количество секунд с 1970-01-01T0:0:0Z в формате UTC до истечения срока действия. Это значение используется для определения времени существования кэшированных маркеров. |
| ресурс |URI идентификатора приложения принимающей службы (защищенный ресурс). |
| access_token |Параметр, возвращающий утверждение SAML. |
| refresh_token |Маркер обновления. Вызывающая служба может использовать этот токен для запроса другого токена доступа по истечении срока действия текущего утверждения SAML. |

- token_type Носитель
- expires_in: 3296.
- ext_expires_in: 0
- expires_on: 1529627844.
- resource: `https://api.contoso.com`
- access_token: \<SAML assertion\>
- issued_token_type: urn:ietf:params:oauth:token-type:saml2
- refresh_token: \<Refresh token\>

## <a name="client-limitations"></a>Ограничения клиентов

Общедоступные клиенты с URL-адресами ответа с подстановочными знаками не могут использовать `id_token` для потоков OBO. Тем не менее конфиденциальный клиент может активировать маркеры **доступа**, полученные с помощью потока неявного предоставления, даже если для общедоступного клиента зарегистрирован универсальный код ресурса (URI) перенаправления с подстановочными знаками.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о протоколе OAuth 2.0 и другом способе проверки подлинности между службами с использованием учетных данных клиента:

* [Проверка подлинности между службами с использованием учетных данных клиента OAuth 2.0 в Azure AD](v1-oauth2-client-creds-grant-flow.md)
* [OAuth 2.0 в Azure AD](v1-protocols-oauth-code.md)