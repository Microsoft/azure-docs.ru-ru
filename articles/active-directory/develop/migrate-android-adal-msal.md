---
title: Руководством по миграции ADAL в MSAL для Android | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как перенести приложение "Библиотека проверки подлинности Azure Active Directory" (ADAL) в библиотеку проверки подлинности Майкрософт (MSAL).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.tgt_pltfrm: Android
ms.workload: identity
ms.date: 10/14/2020
ms.author: marsma
ms.reviewer: shoatman
ms.custom: aaddev
ms.openlocfilehash: ba639bc023affc7c2e6b2b675cdedc1229636893
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99581046"
---
# <a name="adal-to-msal-migration-guide-for-android"></a>Руководством по миграции ADAL в MSAL для Android

В этой статье описываются изменения, которые необходимо внести, чтобы перенести приложение, Azure Active Directory использующее библиотеку проверки подлинности (MSAL), с помощью библиотеки аутентификации Майкрософт (ADAL).

## <a name="difference-highlights"></a>Особенности разницы

ADAL работает с конечной точкой Azure Active Directory v 1.0. Библиотека проверки подлинности Microsoft (MSAL) работает с платформой идентификации Майкрософт, которая ранее называлась конечной точкой Azure Active Directory v 2.0. Платформа Microsoft Identity отличается от Azure Active Directory v 1.0:

Поддерживаются следующие возможности:
  - Удостоверение Организации (Azure Active Directory)
  - Удостоверения, отличные от Организации, такие как Outlook.com, Xbox Live и т. д.
  - (Только Azure AD B2C) Федеративная учетная запись с Google, Facebook, Twitter и Amazon

- Совместимы со стандартами:
  - OAuth 2.0
  - OpenID Connect Connect (OIDC)

Общедоступный API MSAL содержит важные изменения, в том числе:

- Новая модель для доступа к маркерам:
  - ADAL предоставляет доступ к маркерам через объект `AuthenticationContext` , который представляет сервер. MSAL предоставляет доступ к маркерам через объект `PublicClientApplication` , который представляет клиент. Разработчикам клиентов не нужно создавать новый `PublicClientApplication` экземпляр для каждого центра, с которым им нужно взаимодействовать. Требуется только одна `PublicClientApplication` Конфигурация.
  - Поддержка запроса маркеров доступа с использованием областей в дополнение к идентификаторам ресурсов.
  - Поддержка добавочного согласия. Разработчики могут запрашивать области по мере того, как пользователь обращается к большей и более функциональной функциональности в приложении, включая те, которые не были добавлены во время регистрации приложения.
  - Центры больше не проверяются во время выполнения. Вместо этого разработчик объявляет список известных центров во время разработки.
- Изменения API-интерфейса маркеров:
  - В ADAL `AcquireToken()` сначала выполняет автоматический запрос. В таком случае он выполняет интерактивный запрос. Это привело к тому, что некоторые разработчики полагается только на `AcquireToken` , что привело к непредвиденному получению учетных данных у пользователя. MSAL требует, чтобы разработчики предпринимали намерение о том, когда пользователь получит запрос пользовательского интерфейса.
    - `AcquireTokenSilent` всегда приводит к автоматическому запросу, который либо завершается успешно, либо завершается ошибкой.
    - `AcquireToken` всегда приводит к запросу, запрашивающему пользователя через пользовательский интерфейс.
- MSAL поддерживает вход из браузера по умолчанию или встроенного веб-представления:
  - По умолчанию используется браузер на устройстве по умолчанию. Это позволяет MSAL использовать состояние проверки подлинности (файлы cookie), которое уже может присутствовать в одной или нескольких учетных записях для входа. Если состояние проверки подлинности отсутствует, проверка подлинности во время авторизации через MSAL приводит к созданию состояния проверки подлинности (файлов cookie) для преимуществ других веб-приложений, которые будут использоваться в том же браузере.
- Новая модель исключений:
  - Исключения более четко определяют тип возникшей ошибки и действия, которые разработчик должен выполнить для ее устранения.
- MSAL поддерживает объекты параметров для `AcquireToken` `AcquireTokenSilent` вызовов и.
- MSAL поддерживает декларативную конфигурацию для:
  - Идентификатор клиента, URI перенаправления.
  - Браузер Embedded VS по умолчанию
  - Органы
  - Параметры HTTP, такие как время ожидания чтения и подключения

## <a name="your-app-registration-and-migration-to-msal"></a>Регистрация и миграция приложения в MSAL

Вам не нужно изменять существующую регистрацию приложения для использования MSAL. Если вы хотите использовать преимущества добавочного или прогрессивного согласия, может потребоваться проверить регистрацию, чтобы определить конкретные области, для которых требуется добавочный запрос. Дополнительные сведения об областях и последовательном согласии приведены ниже.

На портале регистрации приложения вы увидите вкладку **разрешения API** . Это предоставляет список интерфейсов API и разрешений (областей), в которых приложение настроено для запроса доступа к. Также отображается список имен областей, связанных с каждым разрешениями API.

### <a name="user-consent"></a>предоставление согласия пользователем.

При использовании ADAL и конечной точки Azure AD версии 1 согласие пользователя на ресурсы, которыми они владеют, было предоставлено при первом использовании. При использовании MSAL и платформы идентификации Майкрософт разрешение можно запросить постепенно. Добавочное согласие полезно для разрешений, которые пользователь может считать высокой привилегией, или, в противном случае, если не предоставлено четкое объяснение того, почему требуется разрешение. В ADAL эти разрешения могли привести к тому, что пользователь отказался от входа в приложение.

> [!TIP]
> Используйте добавочное согласие, чтобы предоставить пользователям дополнительный контекст о том, почему приложению требуется разрешение.

### <a name="admin-consent"></a>Согласие администратора

Администраторы организации могут дать согласие на разрешения, которые требуются вашему приложению от имени всех членов Организации. Некоторые организации разрешают только администраторам согласие на приложения. Для предоставления согласия администратора необходимо включить все разрешения API и области, используемые приложением при регистрации приложения.

> [!TIP]
> Несмотря на то, что можно запросить область с помощью MSAL для чего-либо, не включенного в регистрацию приложения, рекомендуется обновить регистрацию приложения, включив в нее все ресурсы и области, на которые пользователь может предоставить разрешение.

## <a name="migrating-from-resource-ids-to-scopes"></a>Миграция с идентификаторов ресурсов в области

### <a name="authenticate-and-request-authorization-for-all-permissions-on-first-use"></a>Проверка подлинности и запрос авторизации для всех разрешений при первом использовании

Если вы используете ADAL и не хотите использовать добавочное согласие, самый простой способ начать использовать MSAL — создать `acquireToken` запрос с помощью нового `AcquireTokenParameter` объекта и задать значение идентификатора ресурса.

> [!CAUTION]
> Невозможно задать обе области и идентификатор ресурса. Попытка задать оба значения приведет к вызову `IllegalArgumentException` .

Это приведет к тому же поведению версии v1, которое вы используете. Все разрешения, запрошенные в регистрации приложения, запрашиваются от пользователя во время первого взаимодействия.

### <a name="authenticate-and-request-permissions-only-as-needed"></a>Проверка подлинности и запрос разрешений только по мере необходимости

Чтобы воспользоваться преимуществами добавочного согласия, создайте список разрешений (областей), которые приложение использует при регистрации приложения, и упорядочите их в два списка на основе:

- Какие области необходимо запросить во время первого взаимодействия пользователя с приложением во время входа.
- Разрешения, связанные с важной функцией приложения, которые также необходимо объяснить пользователю.

После того как вы организовали области, упорядочите каждый список, используя ресурс (API), для которого требуется запросить маркер. А также любые другие области, которые пользователь должен авторизовать в то же время.

Объект параметров, используемый для выполнения запроса к MSAL, поддерживает:

- `Scope`— Список областей, для которых требуется запросить авторизацию и получить маркер доступа.
- `ExtraScopesToConsent`— Дополнительный список областей, для которых требуется запросить авторизацию при запросе маркера доступа для другого ресурса. Этот список областей позволяет максимально сокращать количество раз, необходимое для запроса авторизации пользователя. Это означает меньше запросов на авторизацию или согласие пользователя.

## <a name="migrate-from-authenticationcontext-to-publicclientapplications"></a>Миграция с AuthenticationContext на Публикклиентаппликатионс

### <a name="constructing-publicclientapplication"></a>Создание Публикклиентаппликатион

При использовании MSAL создается экземпляр `PublicClientApplication` . Этот объект моделирует удостоверение приложения и используется для выполнения запросов к одному или нескольким центрам. С помощью этого объекта вы настроите удостоверение клиента, URI перенаправления, центр по умолчанию, следует ли использовать браузер устройств и встроенное веб-представление, уровень ведения журнала и многое другое.

Этот объект можно декларативно настроить с помощью JSON, который вы либо предоставляете в качестве файла, либо сохраняете в качестве ресурса в APK.

Хотя этот объект не является Singleton-классом, он использует Shared `Executors` как для интерактивных, так и для автоматических запросов.

### <a name="business-to-business"></a>Бизнес-Бизнес

В ADAL каждой организации, для которой запрашиваются маркеры доступа, требуется отдельный экземпляр `AuthenticationContext` . В MSAL это больше не требуется. Можно указать центр, из которого требуется запросить маркер в рамках интерактивного запроса.

### <a name="migrate-from-authority-validation-to-known-authorities"></a>Переход от проверки центра сертификации к известным центрам

MSAL не имеет флага для включения или отключения проверки центра. Проверка центра — это функция ADAL, а также в ранних выпусках MSAL, которая не позволяет коду запрашивать маркеры от потенциально вредоносного центра. MSAL теперь получает список органов, известных корпорации Майкрософт, и объединяет этот список с центрами, указанными в конфигурации.

> [!TIP]
> Если вы являетесь пользователем Azure Business to consumer (B2C), это означает, что вам больше не нужно отключать проверку полномочий. Вместо этого включите все поддерживаемые политики Azure AD B2C как центры в конфигурацию MSAL.

Если вы попытаетесь использовать центр, который не известен корпорации Майкрософт и не входит в конфигурацию, вы получите `UnknownAuthorityException` .

### <a name="logging"></a>Ведение журнала
Теперь можно декларативно настроить ведение журнала как часть конфигурации, например:

```json
"logging": {
  "pii_enabled": false,
  "log_level": "WARNING",
  "logcat_enabled": true
}
```

## <a name="migrate-from-userinfo-to-account"></a>Миграция из UserInfo в учетную запись

В ADAL `AuthenticationResult` предоставляет `UserInfo` объект, используемый для получения сведений об учетной записи, прошедшей проверку подлинности. Термин «пользователь», предназначенный для человека или агента программного обеспечения, был применен таким образом, что он затрудняет связь с тем, что некоторые приложения поддерживают одного пользователя (или агента программного обеспечения) с несколькими учетными записями.

Рассмотрим банковский счет. У вас может быть более одной учетной записи в нескольких финансовых учреждениях. При открытии учетной записи вы (пользователь) выдаете учетные данные, такие как ATM-карты & закрепления, которые используются для доступа к балансу, оплате счетов и т. д. для каждой учетной записи. Эти учетные данные можно использовать только в финансовом учреждении, в котором они были выпущены.

По аналогии, как и учетные записи в финансовом учреждении, доступ к учетным записям на платформе Microsoft Identity осуществляется с помощью учетных данных. Эти учетные данные либо регистрируются в, либо выдаются корпорацией Майкрософт. Или корпорацией Майкрософт от имени Организации.

Если платформа Microsoft Identity отличается от финансового учреждения, в этом аналоговом виде платформа Microsoft Identity предоставляет платформу, которая позволяет пользователю использовать одну учетную запись и связанные с ней учетные данные для доступа к ресурсам, принадлежащим нескольким сотрудникам и организациям. Это похоже на возможность использования карточки, выданной одним банком, но еще одного финансового учреждения. Это работает потому, что все рассматриваемые организации используют платформу Microsoft Identity, что позволяет использовать одну учетную запись в нескольких организациях. Приведем пример:

Сэм работает для Contoso.com, но управляет виртуальными машинами Azure, принадлежащими Fabrikam.com. Чтобы администратор SAM управлял виртуальными машинами Fabrikam, ему необходимо иметь права на доступ к ним. Этот доступ можно предоставить, добавив учетную запись SAM в Fabrikam.com и предоставив своей учетной записи роль, которая позволит ему работать с виртуальными машинами. Это можно сделать с помощью портал Azure.

Добавление учетной записи Contoso.com SAM в качестве члена Fabrikam.com приведет к созданию новой записи в Azure Active Directory Fabrikam. com для Sam. Запись SAM в Azure Active Directory называется объектом пользователя. В этом случае этот объект пользователя будет указывать на объект пользователя SAM в Contoso.com. Объект пользователя Fabrikam компании SAM — это локальное представление SAM, которое будет использоваться для хранения сведений об учетной записи, связанной с SAM, в контексте Fabrikam.com. В Contoso.com названием Сэм является старший консультант DevOps. В Fabrikam название Сэм — Contractor-Virtual машины. В Contoso.com Сэм не несет ответственности за управление виртуальными машинами, а также не является его авторизацией. В Fabrikam.com это единственная функция работы. Но у Sam по-прежнему есть только один набор учетных данных для отслеживания, то есть учетные данные, выданные Contoso.com.

После успешного `acquireToken` вызова вы увидите ссылку на `IAccount` объект, который можно использовать в последующих `acquireTokenSilent` запросах.

### <a name="imultitenantaccount"></a>имултитенантаккаунт

Если у вас есть приложение, обращающееся к утверждениям учетной записи из каждого клиента, в котором представлена учетная запись, можно привести `IAccount` объекты в `IMultiTenantAccount` . Этот интерфейс предоставляет карту, с `ITenantProfiles` ключом по идентификатору клиента, которая позволяет получить доступ к утверждениям, принадлежащим учетной записи в каждом клиенте, от которого запрашивается маркер, относительно текущей учетной записи.

Утверждения в корне `IAccount` и `IMultiTenantAccount` всегда содержат утверждения от домашнего клиента. Если вы еще не сделали запрос на маркер в домашнем клиенте, эта коллекция будет пустой.

## <a name="other-changes"></a>Другие изменения

### <a name="use-the-new-authenticationcallback"></a>Использование нового AuthenticationCallback

```java
// Existing ADAL Interface
public interface AuthenticationCallback<T> {

    /**
     * This will have the token info.
     *
     * @param result returns <T>
     */
    void onSuccess(T result);

    /**
     * Sends error information. This can be user related error or server error.
     * Cancellation error is AuthenticationCancelError.
     *
     * @param exc return {@link Exception}
     */
    void onError(Exception exc);
}
```

```java
// New Interface for Interactive AcquireToken
public interface AuthenticationCallback {

    /**
     * Authentication finishes successfully.
     *
     * @param authenticationResult {@link IAuthenticationResult} that contains the success response.
     */
    void onSuccess(final IAuthenticationResult authenticationResult);

    /**
     * Error occurs during the authentication.
     *
     * @param exception The {@link MsalException} contains the error code, error message and cause if applicable. The exception
     *                  returned in the callback could be {@link MsalClientException}, {@link MsalServiceException}
     */
    void onError(final MsalException exception);

    /**
     * Will be called if user cancels the flow.
     */
    void onCancel();
}

// New Interface for Silent AcquireToken
public interface SilentAuthenticationCallback {

    /**
     * Authentication finishes successfully.
     *
     * @param authenticationResult {@link IAuthenticationResult} that contains the success response.
     */
    void onSuccess(final IAuthenticationResult authenticationResult);

    /**
     * Error occurs during the authentication.
     *
     * @param exception The {@link MsalException} contains the error code, error message and cause if applicable. The exception
     *                  returned in the callback could be {@link MsalClientException}, {@link MsalServiceException} or
     *                  {@link MsalUiRequiredException}.
     */
    void onError(final MsalException exception);
}
```

## <a name="migrate-to-the-new-exceptions"></a>Миграция на новые исключения

В ADAL существует один тип исключения, `AuthenticationException` который включает метод для получения `ADALError` значения перечисления.
В MSAL есть иерархия исключений, каждая из которых имеет собственный набор связанных с ними кодов ошибок.

| Исключение                                        | Описание                                                         |
|--------------------------------------------------|---------------------------------------------------------------------|
| `MsalArgumentException`                          | Вызывается, если один или несколько входных аргументов являются недопустимыми.                 |
| `MsalClientException`                            | Вызывается, если ошибка является клиентской стороны.                                 |
| `MsalDeclinedScopeException`                     | Возникает, если одна или несколько запрошенных областей были отклонены сервером. |
| `MsalException`                                  | Исключение, установленное по умолчанию, вызвано MSAL.                           |
| `MsalIntuneAppProtectionPolicyRequiredException` | Создается, если для ресурса включена политика защиты МАМКА.         |
| `MsalServiceException`                           | Вызывается, если ошибка является серверной стороной.                                 |
| `MsalUiRequiredException`                        | Создается, если маркер не может быть обновлен автоматически.                    |
| `MsalUserCancelException`                        | Создается, если пользователь отменил поток проверки подлинности.                |

### <a name="adalerror-to-msalexception-translation"></a>Преобразование Адалеррор в Мсалексцептион

| Если вы перехватите эти ошибки в ADAL...  | ... Перехватите следующие исключения MSAL:                                                         |
|--------------------------------------------------|---------------------------------------------------------------------|
| *Нет эквивалентного Адалеррор* | `MsalArgumentException`                          |
| <ul><li>`ADALError.ANDROIDKEYSTORE_FAILED`<li>`ADALError.AUTH_FAILED_USER_MISMATCH`<li>`ADALError.DECRYPTION_FAILED`<li>`ADALError.DEVELOPER_AUTHORITY_CAN_NOT_BE_VALIDED`<li>`ADALError.EVELOPER_AUTHORITY_IS_NOT_VALID_INSTANCE`<li>`ADALError.DEVELOPER_AUTHORITY_IS_NOT_VALID_URL`<li>`ADALError.DEVICE_CONNECTION_IS_NOT_AVAILABLE`<li>`ADALError.DEVICE_NO_SUCH_ALGORITHM`<li>`ADALError.ENCODING_IS_NOT_SUPPORTED`<li>`ADALError.ENCRYPTION_ERROR`<li>`ADALError.IO_EXCEPTION`<li>`ADALError.JSON_PARSE_ERROR`<li>`ADALError.NO_NETWORK_CONNECTION_POWER_OPTIMIZATION`<li>`ADALError.SOCKET_TIMEOUT_EXCEPTION`</ul> | `MsalClientException`                            |
| *Нет эквивалентного Адалеррор* | `MsalDeclinedScopeException`                     |
| <ul><li>`ADALError.APP_PACKAGE_NAME_NOT_FOUND`<li>`ADALError.BROKER_APP_VERIFICATION_FAILED`<li>`ADALError.PACKAGE_NAME_NOT_FOUND`</ul> | `MsalException`                                  |
| *Нет эквивалентного Адалеррор* | `MsalIntuneAppProtectionPolicyRequiredException` |
| <ul><li>`ADALError.SERVER_ERROR`<li>`ADALError.SERVER_INVALID_REQUEST`</ul> | `MsalServiceException`                           |
| <ul><li>`ADALError.AUTH_REFRESH_FAILED_PROMPT_NOT_ALLOWED` | `MsalUiRequiredException`</ul>                        |
| *Нет эквивалентного Адалеррор* | `MsalUserCancelException`                        |

### <a name="adal-logging-to-msal-logging"></a>Ведение журнала ADAL для ведения журнала MSAL

```java
// Legacy Interface
    StringBuilder logs = new StringBuilder();
    Logger.getInstance().setExternalLogger(new ILogger() {
            @Override
            public void Log(String tag, String message, String additionalMessage, LogLevel logLevel, ADALError errorCode) {
                logs.append(message).append('\n');
            }
        });
```

```java
// New interface
  StringBuilder logs = new StringBuilder();
  Logger.getInstance().setExternalLogger(new ILoggerCallback() {
      @Override
      public void log(String tag, Logger.LogLevel logLevel, String message, boolean containsPII) {
          logs.append(message).append('\n');
      }
  });

// New Log Levels:
public enum LogLevel
{
    /**
     * Error level logging.
     */
    ERROR,
    /**
     * Warning level logging.
     */
    WARNING,
    /**
     * Info level logging.
     */
    INFO,
    /**
     * Verbose level logging.
     */
    VERBOSE
}
```
