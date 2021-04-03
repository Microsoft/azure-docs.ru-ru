---
title: Получение токена в приложении Android
titleSuffix: Azure AD B2C
description: Сведения о создании приложения Android, которое использует AppAuth с Azure Active Directory B2C для управления удостоверениями пользователей и проверки подлинности пользователей.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 05/12/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: acbd2918bd311cec1c27018763ad10771d779d85
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94953327"
---
# <a name="sign-in-using-an-android-application-in-azure-active-directory-b2c"></a>Вход с помощью приложения Android в Azure Active Directory B2C

Платформа Microsoft Identity использует открытые стандарты, такие как OAuth2 и OpenID Connect. Эти стандарты позволяют использовать любую библиотеку, которую необходимо интегрировать с Azure Active Directory B2C. Чтобы помочь вам использовать другие библиотеки, мы написали несколько подобных этому пошаговых руководств по настройке сторонних библиотек для подключения к платформе Microsoft Identity. Большинство библиотек, в которых реализована [спецификация RFC6749 OAuth2](https://tools.ietf.org/html/rfc6749), могут подключаться к платформе Microsoft Identity.

> [!WARNING]
> Корпорация Майкрософт не предоставляет исправления для библиотек сторонних производителей и не выполняла их проверку. В этом примере используется библиотека стороннего производителя с именем AppAuth, которая была протестирована в основных сценариях на совместимость со службой Azure AD B2C. Проблемы и запросы возможностей следует отправлять в проекты с открытым кодом библиотеки. Дополнительные сведения см. в [этой статье](../active-directory/develop/reference-v2-libraries.md).
>
>

Если вы еще не работали с OAuth2 или OpenID Connect, вы не поймете большую часть примера конфигурации. Для начала мы рекомендуем просмотреть [общие сведения о протоколе](protocols-overview.md).

## <a name="get-an-azure-ad-b2c-directory"></a>Создание каталога Azure AD B2C

Перед использованием Azure AD B2C необходимо создать каталог или клиент. Каталог — это контейнер для данных всех ваших пользователей, приложений, групп и т. д. Если каталог B2C еще не создан, [создайте его](tutorial-create-tenant.md), прежде чем продолжить.

## <a name="create-an-application"></a>Создание приложения

Затем зарегистрируйте приложение в клиенте Azure AD B2C. Таким образом в Azure AD поступят сведения, необходимые для безопасного взаимодействия с вашим приложением.

[!INCLUDE [active-directory-b2c-appreg-native](../../includes/active-directory-b2c-appreg-native.md)]

Запишите значение параметра **Идентификатор приложения (клиент)** . Оно вам потребуется в дальнейшем.

Кроме того, запишите пользовательский URI перенаправления, который будет использоваться позже. Например, `com.onmicrosoft.contosob2c.exampleapp://oauth/redirect`.

## <a name="create-your-user-flows"></a>Создание потоков пользователей

В Azure AD B2C все действия пользователя регулируются [потоком пользователя](user-flow-overview.md). Это набор политик для управления поведением Azure AD. Для этого приложения требуется пользовательский сценарий входа в систему и регистрации. При создании потока пользователя обязательно сделайте следующее:

* Укажите **отображаемое имя** в качестве атрибута входа для потока пользователя.
* Выберите утверждения приложения **Отображаемое имя** и **Идентификатор объекта** для каждого потока пользователя. Можно также выбрать другие утверждения.
* Скопируйте **имя** каждого созданного потока пользователя. У него должен быть префикс `b2c_1_`.  Оно понадобится вам позже.

Создав потоки пользователей, можно приступать к сборке приложения.

## <a name="download-the-sample-code"></a>Скачивание примера кода

Мы разместили рабочий пример, использующий AppAuth с Azure AD B2C, [на сайте GitHub](https://github.com/Azure-Samples/active-directory-android-native-appauth-b2c). Вы можете скачать код и запустить его. Вы можете быстро приступить к работе с приложением с использованием конфигурации Azure AD B2C, следуя инструкциям в разделе [README.md](https://github.com/Azure-Samples/active-directory-android-native-appauth-b2c/blob/master/README.md).

Пример представляет собой измененную версию примера, предоставляемого [AppAuth](https://openid.github.io/AppAuth-Android/). Посетите нашу страницу для получения дополнительных сведений об AppAuth и его функциях.

## <a name="modifying-your-app-to-use-azure-ad-b2c-with-appauth"></a>Изменение приложения для использования Azure AD B2C с AppAuth

> [!NOTE]
> AppAuth поддерживает Android API версии 16 (Jellybean) и более поздних. Мы советуем использовать API версии 23 и более поздних.
>

### <a name="configuration"></a>Конфигурация

Чтобы настроить взаимодействие с Azure AD B2C, укажите URI обнаружения или URI конечных точек авторизации и токенов. В любом случае вам потребуются следующие сведения:

* идентификатор клиента (например, contoso.onmicrosoft.com);
* имя потока пользователя (например, B2C\_1\_SignUpIn).

Если выбрано автоматическое обнаружение URI конечных точек авторизации и токенов, вам потребуется получить сведения из URI обнаружения. URI обнаружения можно создать, заменив `<tenant-id>` и `<policy-name>` в следующем URL-адресе:

```java
String mDiscoveryURI = "https://<tenant-name>.b2clogin.com/<tenant-id>/<policy-name>/v2.0/.well-known/openid-configuration";
```

Вы можете получить URI конечных точек авторизации и токенов, а также создать объект AuthorizationServiceConfiguration, выполнив следующую команду:

```java
final Uri issuerUri = Uri.parse(mDiscoveryURI);
AuthorizationServiceConfiguration config;

AuthorizationServiceConfiguration.fetchFromIssuer(
    issuerUri,
    new RetrieveConfigurationCallback() {
      @Override public void onFetchConfigurationCompleted(
          @Nullable AuthorizationServiceConfiguration serviceConfiguration,
          @Nullable AuthorizationException ex) {
        if (ex != null) {
            Log.w(TAG, "Failed to retrieve configuration for " + issuerUri, ex);
        } else {
            // service configuration retrieved, proceed to authorization...
        }
      }
  });
```

Вместо того чтобы получать URI конечных точек авторизации и токенов путем обнаружения, их можно также явно определить, заменив `<tenant-id>` и `<policy-name>` в приведенных ниже URL-адресах:

```java
String mAuthEndpoint = "https://<tenant-name>.b2clogin.com/<tenant-id>/<policy-name>/oauth2/v2.0/authorize";

String mTokenEndpoint = "https://<tenant-name>.b2clogin.com/<tenant-id>/<policy-name>/oauth2/v2.0/token";
```

Выполните следующий код для создания объекта AuthorizationServiceConfiguration:

```java
AuthorizationServiceConfiguration config =
        new AuthorizationServiceConfiguration(name, mAuthEndpoint, mTokenEndpoint);

// perform the auth request...
```

### <a name="authorizing"></a>Авторизация

После настройки или извлечения конфигурации службы авторизации можно сформировать запрос авторизации. Для создания запроса вам потребуются следующие сведения:

* Идентификатор клиента (идентификатор приложения), записанный ранее. Например, `00000000-0000-0000-0000-000000000000`.
* Пользовательский URI перенаправления, записанный ранее. Например, `com.onmicrosoft.contosob2c.exampleapp://oauth/redirect`.

Оба элемента нужно сохранить при [регистрации приложения](#create-an-application).

```java
AuthorizationRequest req = new AuthorizationRequest.Builder(
    config,
    clientId,
    ResponseTypeValues.CODE,
    redirectUri)
    .build();
```

Дополнительные сведения о завершении остальной части процесса см. в [руководстве по AppAuth](https://openid.github.io/AppAuth-Android/). Если вам нужно быстро приступить к работе с приложением, ознакомьтесь с [нашим примером](https://github.com/Azure-Samples/active-directory-android-native-appauth-b2c). Следуйте указаниям в файле [README.md](https://github.com/Azure-Samples/active-directory-android-native-appauth-b2c/blob/master/README.md), чтобы ввести собственные значения для настройки Azure AD B2C.