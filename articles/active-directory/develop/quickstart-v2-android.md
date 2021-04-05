---
title: Краткое руководство. Добавление возможности входа в приложение Android с помощью учетной записи Майкрософт | Azure
titleSuffix: Microsoft identity platform
description: Из этого краткого руководства вы узнаете, как приложения Android могут вызывать API, которому требуются маркеры доступа, выданные платформой удостоверений Майкрософт.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 10/15/2019
ms.author: marsma
ms.custom: aaddev, identityplatformtop40, scenarios:getting-started, languages:Android
ms.openlocfilehash: 49c69a79b4841f13623c9e6ab3daea72dd7fde26
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99583660"
---
# <a name="quickstart-sign-in-users-and-call-the-microsoft-graph-api-from-an-android-app"></a>Краткое руководство. Вход пользователей и вызов API Microsoft Graph из приложения Android

Из этого краткого руководства вы узнаете, как скачать и выполнить пример кода. В примере кода показано, как в приложении Android реализовать вход пользователей и получение маркера доступа для вызова API Microsoft Graph. 

Иллюстрацию см. в разделе [Как работает этот пример](#how-the-sample-works).

Чтобы приложение получало маркеры от платформы удостоверений Майкрософт, оно должно быть представлено объектом приложения в Azure Active Directory.

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
* Android Studio
* Android 16+

> [!div class="sxs-lookup" renderon="portal"]
> ### <a name="step-1-configure-your-application-in-the-azure-portal"></a>Шаг 1. Настройка приложения на портале Azure
> Для работы примера кода в этом кратком руководстве необходимо добавить **URI перенаправления**, совместимый с брокером авторизации.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Внести эти изменения для меня]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Уже настроено](media/quickstart-v2-android/green-check.png). Ваше приложение настроено с помощью этих атрибутов
>
> ### <a name="step-2-download-the-project"></a>Шаг 2. Скачивание проекта
> [!div class="sxs-lookup" renderon="portal"]
> Выполните проект с помощью Android Studio.
> [!div class="sxs-lookup" renderon="portal" id="autoupdate" class="nextstepaction"]
> [Скачивание примера кода](https://github.com/Azure-Samples/ms-identity-android-java/archive/master.zip)
>
> [!div class="sxs-lookup" renderon="portal"]
> ### <a name="step-3-your-app-is-configured-and-ready-to-run"></a>Шаг 3. Приложение настроено и готово к запуску
> Мы уже настроили для проекта нужные значения свойств приложения, и его можно запускать.
> Пример приложения запускается на экране **Single Account Mode** (Режим единой учетной записи). Область по умолчанию, **user.read**, предоставляется по умолчанию и используется при чтении данных собственного профиля во время вызова API Microsoft Graph. URL-адрес для вызова Microsoft Graph API предоставляется по умолчанию. При необходимости их можно изменить.
>
> ![Пример приложения MSAL с использованием одной и нескольких учетных записей](./media/quickstart-v2-android/quickstart-sample-app.png)
>
> Используйте меню приложения для переключения между режимами одной и нескольких учетных записей.
>
> В режиме одной учетной записи войдите с помощью рабочей или домашней учетной записи:
>
> 1. Выберите **Get graph data interactively** (Получить данные графов в интерактивном режиме), чтобы запросить у пользователя учетные данные. Вы увидите выходные данные вызова API Microsoft Graph в нижней части экрана.
> 2. После входа в систему выберите **Get graph data silently** (Получить данные графов без уведомления), чтобы вызвать Microsoft Graph API, не запрашивая у пользователя учетные данные снова. Вы увидите выходные данные вызова API Microsoft Graph в нижней части экрана.
>
> В режиме нескольких учетных записей можно повторить те же действия.  Кроме того, можно удалить учетную запись для входа. При этом кэшированные токены для этой учетной записи также удаляются.

> [!div class="sxs-lookup" renderon="portal"]
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`

> [!div renderon="docs"]
> ## <a name="step-1-get-the-sample-app"></a>Шаг 1. Получение примера приложения
>
> [Скачайте код](https://github.com/Azure-Samples/ms-identity-android-java/archive/master.zip).
>
> ## <a name="step-2-run-the-sample-app"></a>Шаг 2. Запуск примера приложения
>
> Выберите эмулятор или физическое устройство из раскрывающегося списка **доступных устройств** Android Studio и запустите приложение.
>
> Пример приложения запускается на экране **Single Account Mode** (Режим единой учетной записи). Область по умолчанию, **user.read**, предоставляется по умолчанию и используется при чтении данных собственного профиля во время вызова API Microsoft Graph. URL-адрес для вызова Microsoft Graph API предоставляется по умолчанию. При необходимости их можно изменить.
>
> ![Пример приложения MSAL с использованием одной и нескольких учетных записей](./media/quickstart-v2-android/quickstart-sample-app.png)
>
> Используйте меню приложения для переключения между режимами одной и нескольких учетных записей.
>
> В режиме одной учетной записи войдите с помощью рабочей или домашней учетной записи:
>
> 1. Выберите **Get graph data interactively** (Получить данные графов в интерактивном режиме), чтобы запросить у пользователя учетные данные. Вы увидите выходные данные вызова API Microsoft Graph в нижней части экрана.
> 2. После входа в систему выберите **Get graph data silently** (Получить данные графов без уведомления), чтобы вызвать Microsoft Graph API, не запрашивая у пользователя учетные данные снова. Вы увидите выходные данные вызова API Microsoft Graph в нижней части экрана.
>
> В режиме нескольких учетных записей можно повторить те же действия.  Кроме того, можно удалить учетную запись для входа. При этом кэшированные токены для этой учетной записи также удаляются.

## <a name="how-the-sample-works"></a>Как работает этот пример
![Снимок экрана примера приложения](media/quickstart-v2-android/android-intro.svg)


Код разделен на фрагменты, демонстрирующие, как записывать приложение MSAL для одной и нескольких учетных записей. Файлы кода организованы следующим образом:

| Файл  | Что демонстрирует  |
|---------|---------|
| MainActivity | Управляет пользовательским интерфейсом |
| MSGraphRequestWrapper  | Вызывает API Microsoft Graph, используя токен, предоставленный MSAL |
| MultipleAccountModeFragment  | Инициализирует приложение с несколькими учетными записями, загружает учетную запись пользователя и получает токен для вызова API Microsoft Graph |
| SingleAccountModeFragment | Инициализирует приложение с одной учетной записью, загружает учетную запись пользователя и получает токен для вызова API Microsoft Graph |
| res/auth_config_multiple_account.json  | Файл конфигурации нескольких учетных записей |
| res/auth_config_single_account.json  | Файл конфигурации одной учетной записи |
| Скрипты Gradle/build.grade (модуль: app) | Здесь добавляются зависимости библиотеки MSAL |

Теперь мы рассмотрим эти файлы подробнее и вызовем код, относящийся к MSAL, в каждом из них

### <a name="adding-msal-to-the-app"></a>Добавление MSAL в приложение

MSAL ([com.microsoft.identity.client](https://javadoc.io/doc/com.microsoft.identity.client/msal)) — это библиотека, используемая для выполнения входа пользователей и запросов маркеров, которые нужны для доступа к API, защищенному платформой удостоверений Майкрософт. Gradle 3.0+ устанавливает библиотеку при добавлении в файл **Скрипты Gradle** > **build.gradle (модуль: app)** в разделе **Зависимости** следующего фрагмента кода:

```gradle
implementation 'com.microsoft.identity.client:msal:2.+'
```

Следующий фрагмент кода можно увидеть в примере проекта в build.gradle (модуль: app):

```java
dependencies {
    ...
    implementation 'com.microsoft.identity.client:msal:2.+'
    ...
}
```

Он указывает Gradle загрузить и создать MSAL из Maven Central.

### <a name="msal-imports"></a>Операции импорта MSAL

Операции импорта, относящиеся к библиотеке MSAL: `com.microsoft.identity.client.*`.  Например, вы увидите `import com.microsoft.identity.client.PublicClientApplication;`, что является пространством имен для класса `PublicClientApplication`, который представляет общедоступное клиентское приложение.

### <a name="singleaccountmodefragmentjava"></a>SingleAccountModeFragment.java

В этом файле показано, как создать приложение MSAL для одной учетной записи и вызвать API Microsoft Graph.

Приложения с одной учетной записью используются только одним пользователем.  Например, у вас может быть только одна учетная запись для входа в приложение сопоставления.

#### <a name="single-account-msal-initialization"></a>Инициализация MSAL одной учетной записи

В файле `auth_config_single_account.json` в `onCreateView()` создается одна учетная запись `PublicClientApplication` с использованием сведений о конфигурации, хранящихся в файле `auth_config_single_account.json`.  Таким образом вы инициализируете библиотеку MSAL для использования в приложении MSAL с одной учетной записью:

```java
...
// Creates a PublicClientApplication object with res/raw/auth_config_single_account.json
PublicClientApplication.createSingleAccountPublicClientApplication(getContext(),
        R.raw.auth_config_single_account,
        new IPublicClientApplication.ISingleAccountApplicationCreatedListener() {
            @Override
            public void onCreated(ISingleAccountPublicClientApplication application) {
                /**
                 * This test app assumes that the app is only going to support one account.
                 * This requires "account_mode" : "SINGLE" in the config json file.
                 **/
                mSingleAccountApp = application;
                loadAccount();
            }

            @Override
            public void onError(MsalException exception) {
                displayError(exception);
            }
        });
```

#### <a name="sign-in-a-user"></a>Вход пользователя

В файле `SingleAccountModeFragment.java` код для входа пользователя находится в `initializeUI()` в обработчике щелчков `signInButton`.

Прежде чем пытаться получить токены, вызовите `signIn()`. `signIn()` ведет себя так, как будто вызывается `acquireToken()`, в результате чего пользователь получает интерактивный запрос на вход.

Вход пользователя является асинхронной операцией. Передается метод для обратного вызова, который вызывает API Microsoft Graph и обновляет пользовательский интерфейс после входа пользователя:

```java
mSingleAccountApp.signIn(getActivity(), null, getScopes(), getAuthInteractiveCallback());
```

#### <a name="sign-out-a-user"></a>Выход пользователя

В файле `SingleAccountModeFragment.java` код для выхода пользователя находится в `initializeUI()` в обработчике щелчков `signOutButton`.  Выход пользователя является асинхронной операцией. При выходе пользователя также очищается кеш токена для этой учетной записи. Для обновления пользовательского интерфейса после выхода из учетной записи пользователя создается обратный вызов:

```java
mSingleAccountApp.signOut(new ISingleAccountPublicClientApplication.SignOutCallback() {
    @Override
    public void onSignOut() {
        updateUI(null);
        performOperationOnSignOut();
    }

    @Override
    public void onError(@NonNull MsalException exception) {
        displayError(exception);
    }
});
```

#### <a name="get-a-token-interactively-or-silently"></a>Получение токена в интерактивном режиме или автоматически

Чтобы пользователь получил минимальное количество запросов, токен обычно получается автоматически. Если возникает ошибка, попытайтесь перейти к токену в интерактивном режиме. Когда приложение вызывает `signIn()` впервые, оно фактически действует как вызов `acquireToken()`, который запрашивает учетные данные пользователя.

Ниже представлены некоторые ситуации, когда пользователю может быть предложено выбрать учетную запись, ввести учетные данные или дать согласие на разрешения, запрошенные вашим приложением:

* Первый вход пользователя в приложение.
* Если пользователь сбрасывает пароль, то потребуется вводить свои учетные данные.
* Если отменяется согласие.
* Приложение явно требует согласия.
* Когда ваше приложение впервые запрашивает доступ к ресурсу.
* Когда требуются политики условного доступа или MFA.

Код для интерактивного получения токена (то есть с помощью пользовательского интерфейса) находится в файле `SingleAccountModeFragment.java` в `initializeUI()` в обработчике щелчков `callGraphApiInteractiveButton`:

```java
/**
 * If acquireTokenSilent() returns an error that requires an interaction (MsalUiRequiredException),
 * invoke acquireToken() to have the user resolve the interrupt interactively.
 *
 * Some example scenarios are
 *  - password change
 *  - the resource you're acquiring a token for has a stricter set of requirement than your Single Sign-On refresh token.
 *  - you're introducing a new scope which the user has never consented for.
 **/
mSingleAccountApp.acquireToken(getActivity(), getScopes(), getAuthInteractiveCallback());
```

Если пользователь уже выполнил вход, `acquireTokenSilentAsync()` позволяет приложениям запрашивать токены автоматически, как показано в `initializeUI()`, в обработчике щелчков `callGraphApiSilentButton`:

```java
/**
 * Once you've signed the user in,
 * you can perform acquireTokenSilent to obtain resources without interrupting the user.
 **/
  mSingleAccountApp.acquireTokenSilentAsync(getScopes(), AUTHORITY, getAuthSilentCallback());
```

#### <a name="load-an-account"></a>Загрузка учетной записи

Код для загрузки учетной записи находится в файле `SingleAccountModeFragment.java` в `loadAccount()`.  Загрузка учетной записи пользователя является асинхронной операцией, поэтому обратные вызовы для обработки при загрузке учетной записи, изменении или возникновении ошибки передаются в MSAL.  Следующий код также обрабатывает `onAccountChanged()`, что происходит при удалении учетной записи, переходе пользователя на другую учетную запись и т. д.

```java
private void loadAccount() {
    ...

    mSingleAccountApp.getCurrentAccountAsync(new ISingleAccountPublicClientApplication.CurrentAccountCallback() {
        @Override
        public void onAccountLoaded(@Nullable IAccount activeAccount) {
            // You can use the account data to update your UI or your app database.
            updateUI(activeAccount);
        }

        @Override
        public void onAccountChanged(@Nullable IAccount priorAccount, @Nullable IAccount currentAccount) {
            if (currentAccount == null) {
                // Perform a cleanup task as the signed-in account changed.
                performOperationOnSignOut();
            }
        }

        @Override
        public void onError(@NonNull MsalException exception) {
            displayError(exception);
        }
    });
```

#### <a name="call-microsoft-graph"></a>Вызов Microsoft Graph

Когда пользователь выполнил вход, вызов Microsoft Graph выполняется через HTTP-запрос с помощью `callGraphAPI()`, определенного в файле `SingleAccountModeFragment.java`. Эта функция представляет собой программу-оболочку, которая упрощает пример, выполняя некоторые задачи, такие как получение маркера доступа из `authenticationResult`, упаковка вызова MSGraphRequestWrapper и отображение результатов вызова.

```java
private void callGraphAPI(final IAuthenticationResult authenticationResult) {
    MSGraphRequestWrapper.callGraphAPIUsingVolley(
            getContext(),
            graphResourceTextView.getText().toString(),
            authenticationResult.getAccessToken(),
            new Response.Listener<JSONObject>() {
                @Override
                public void onResponse(JSONObject response) {
                    /* Successfully called graph, process data and send to UI */
                    ...
                }
            },
            new Response.ErrorListener() {
                @Override
                public void onErrorResponse(VolleyError error) {
                    ...
                }
            });
}
```

### <a name="auth_config_single_accountjson"></a>auth_config_single_account.json

Это файл конфигурации для приложения MSAL, использующего одну учетную запись.

Описание этих полей см. в разделе [Android Microsoft Authentication Library (MSAL) configuration file](msal-configuration.md) (Файл конфигурации библиотеки аутентификации Microsoft для Android (MSAL)).

Обратите внимание на присутствие `"account_mode" : "SINGLE"`, который настраивает это приложение для использования одной учетной записи.

Параметр `"client_id"` предварительно настроен для использования регистрации объекта приложения, которую обслуживает корпорация Майкрософт.
Параметр `"redirect_uri"` предварительно настроен для использования ключа подписывания, предоставленного в примере кода.

```json
{
  "client_id" : "0984a7b6-bc13-4141-8b0d-8f767e136bb7",
  "authorization_user_agent" : "DEFAULT",
  "redirect_uri" : "msauth://com.azuresamples.msalandroidapp/1wIqXSqBj7w%2Bh11ZifsnqwgyKrY%3D",
  "account_mode" : "SINGLE",
  "broker_redirect_uri_registered": true,
  "authorities" : [
    {
      "type": "AAD",
      "audience": {
        "type": "AzureADandPersonalMicrosoftAccount",
        "tenant_id": "common"
      }
    }
  ]
}
```

### <a name="multipleaccountmodefragmentjava"></a>MultipleAccountModeFragment.java

В этом файле показано, как создать приложение MSAL с несколькими учетными записями и вызвать API Microsoft Graph.

Примером приложения с несколькими учетными записями является почтовое приложение, которое позволяет работать с несколькими учетными записями пользователей, такими как рабочая и личная учетная запись.

#### <a name="multiple-account-msal-initialization"></a>Инициализация MSAL нескольких учетных записей

В файле `MultipleAccountModeFragment.java` в `onCreateView()` объект приложения с несколькими учетными записями (`IMultipleAccountPublicClientApplication`) создается с использованием сведений о конфигурации, хранящихся в `auth_config_multiple_account.json file`:

```java
// Creates a PublicClientApplication object with res/raw/auth_config_multiple_account.json
PublicClientApplication.createMultipleAccountPublicClientApplication(getContext(),
        R.raw.auth_config_multiple_account,
        new IPublicClientApplication.IMultipleAccountApplicationCreatedListener() {
            @Override
            public void onCreated(IMultipleAccountPublicClientApplication application) {
                mMultipleAccountApp = application;
                loadAccounts();
            }

            @Override
            public void onError(MsalException exception) {
                ...
            }
        });
```

Созданный объект `MultipleAccountPublicClientApplication` хранится в переменной члена класса, чтобы его можно было использовать для взаимодействия с библиотекой MSAL для получения токенов, загрузки и удаления учетной записи пользователя.

#### <a name="load-an-account"></a>Загрузка учетной записи

Приложения с несколькими учетными записями обычно вызывают `getAccounts()`, чтобы выбрать учетную запись для использования в операциях MSAL. Код для загрузки учетной записи находится в файле `MultipleAccountModeFragment.java` в `loadAccounts()`.  Загрузка учетной записи пользователя является асинхронной операцией. Поэтому обратный вызов обрабатывает ситуации, когда учетная запись загружается, изменяется или возникает ошибка.

```java
/**
 * Load currently signed-in accounts, if there's any.
 **/
private void loadAccounts() {
    if (mMultipleAccountApp == null) {
        return;
    }

    mMultipleAccountApp.getAccounts(new IPublicClientApplication.LoadAccountsCallback() {
        @Override
        public void onTaskCompleted(final List<IAccount> result) {
            // You can use the account data to update your UI or your app database.
            accountList = result;
            updateUI(accountList);
        }

        @Override
        public void onError(MsalException exception) {
            displayError(exception);
        }
    });
}
```

#### <a name="get-a-token-interactively-or-silently"></a>Получение токена в интерактивном режиме или автоматически

Ниже представлены некоторые ситуации, когда пользователю может быть предложено выбрать учетную запись, ввести учетные данные или дать согласие на разрешения, запрошенные вашим приложением:

* первый вход пользователей в приложение;
* Если пользователь сбрасывает пароль, то потребуется вводить свои учетные данные.
* Если отменяется согласие.
* Приложение явно требует согласия.
* Когда ваше приложение впервые запрашивает доступ к ресурсу.
* Когда требуются политики условного доступа или MFA.

Приложения с несколькими учетными записями обычно должны получать токены в интерактивном режиме, то есть в пользовательском интерфейсе, с вызовом `acquireToken()`.  Код для интерактивного получения токена находится в файле `MultipleAccountModeFragment.java` в `initializeUI()` в обработчике щелчков `callGraphApiInteractiveButton`:

```java
/**
 * Acquire token interactively. It will also create an account object for the silent call as a result (to be obtained by getAccount()).
 *
 * If acquireTokenSilent() returns an error that requires an interaction,
 * invoke acquireToken() to have the user resolve the interrupt interactively.
 *
 * Some example scenarios are
 *  - password change
 *  - the resource you're acquiring a token for has a stricter set of requirement than your SSO refresh token.
 *  - you're introducing a new scope which the user has never consented for.
 **/
mMultipleAccountApp.acquireToken(getActivity(), getScopes(), getAuthInteractiveCallback());
```

Приложения не требуют, чтобы пользователи выполняли вход каждый раз при запросе токена. Если пользователь уже выполнил вход, `acquireTokenSilentAsync()` позволяет приложениям запрашивать токены без запроса к пользователю, как показано в файле `MultipleAccountModeFragment.java` в `initializeUI()` в обработчике щелчков `callGraphApiSilentButton`:

```java
/**
 * Performs acquireToken without interrupting the user.
 *
 * This requires an account object of the account you're obtaining a token for.
 * (can be obtained via getAccount()).
 */
mMultipleAccountApp.acquireTokenSilentAsync(getScopes(),
    accountList.get(accountListSpinner.getSelectedItemPosition()),
    AUTHORITY,
    getAuthSilentCallback());
```

#### <a name="remove-an-account"></a>Удаление учетной записи

Код для удаления учетной записи и всех кэшированных токенов для учетной записи находится в файле `MultipleAccountModeFragment.java` в `initializeUI()` в обработчике кнопки удаления учетной записи. Прежде чем вы сможете удалить учетную запись, вам нужен объект учетной записи, который вы получаете из методов MSAL, таких как `getAccounts()` и `acquireToken()`. Так как удаление учетной записи является асинхронной операцией, обратный вызов `onRemoved` предоставляется для обновления пользовательского интерфейса.

```java
/**
 * Removes the selected account and cached tokens from this app (or device, if the device is in shared mode).
 **/
mMultipleAccountApp.removeAccount(accountList.get(accountListSpinner.getSelectedItemPosition()),
        new IMultipleAccountPublicClientApplication.RemoveAccountCallback() {
            @Override
            public void onRemoved() {
                ...
                /* Reload account asynchronously to get the up-to-date list. */
                loadAccounts();
            }

            @Override
            public void onError(@NonNull MsalException exception) {
                displayError(exception);
            }
        });
```

### <a name="auth_config_multiple_accountjson"></a>auth_config_multiple_account.json

Это файл конфигурации для приложения MSAL, которое использует несколько учетных записей.

Описание разных полей см. в статье [Android Microsoft Authentication Library (MSAL) configuration file](msal-configuration.md) (Файл конфигурации библиотеки аутентификации Microsoft для Android (MSAL)).

В отличие от файла конфигурации [auth_config_single_account.json](#auth_config_single_accountjson), в этом файле конфигурации указано `"account_mode" : "MULTIPLE"` вместо `"account_mode" : "SINGLE"`, так как это приложение с несколькими учетными записями.

Параметр `"client_id"` предварительно настроен для использования регистрации объекта приложения, которую обслуживает корпорация Майкрософт.
Параметр `"redirect_uri"` предварительно настроен для использования ключа подписывания, предоставленного в примере кода.

```json
{
  "client_id" : "0984a7b6-bc13-4141-8b0d-8f767e136bb7",
  "authorization_user_agent" : "DEFAULT",
  "redirect_uri" : "msauth://com.azuresamples.msalandroidapp/1wIqXSqBj7w%2Bh11ZifsnqwgyKrY%3D",
  "account_mode" : "MULTIPLE",
  "broker_redirect_uri_registered": true,
  "authorities" : [
    {
      "type": "AAD",
      "audience": {
        "type": "AzureADandPersonalMicrosoftAccount",
        "tenant_id": "common"
      }
    }
  ]
}
```

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к учебнику по Android, с помощью которого вы создадите приложение Android, получающее маркер доступа от платформы удостоверений Майкрософт и использующее его для вызова API Microsoft Graph.

> [!div class="nextstepaction"]
> [Учебник. Вход пользователей и вызов Microsoft Graph из приложения Android](tutorial-v2-android.md)
