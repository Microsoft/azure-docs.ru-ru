---
title: Использование MSAL.js с Azure AD B2C
titleSuffix: Microsoft identity platform
description: Библиотека проверки подлинности Microsoft для JavaScript (MSAL.js) позволяет приложениям работать с Azure AD B2C и получать маркеры для вызова защищенных веб-API. Этими веб-API могут быть Microsoft Graph, другие интерфейсы Microsoft API, веб-API других пользователей или ваш собственный веб-API.
services: active-directory
author: negoe
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 06/05/2020
ms.author: negoe
ms.reviewer: nacanuma
ms.custom: aaddev devx-track-js
ms.openlocfilehash: 53294a30b38bed7ab7516443277cac24e4fef4c1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98063728"
---
# <a name="use-the-microsoft-authentication-library-for-javascript-to-work-with-azure-ad-b2c"></a>Использование библиотеки проверки подлинности Майкрософт для JavaScript для работы с Azure AD B2C

[Библиотека проверки подлинности Microsoft для JavaScript (MSAL.js)](https://github.com/AzureAD/microsoft-authentication-library-for-js) позволяет разработчикам JavaScript проверять подлинность пользователей с помощью социальных сетей и местных удостоверений с помощью [Azure Active Directory B2C](../../active-directory-b2c/overview.md) (Azure AD B2C).

Используя Azure AD B2C в качестве службы управления удостоверениями, можно настроить и контролировать, как ваши клиенты подписываются, входят в систему и управляют своими профилями при использовании ваших приложений. Azure AD B2C также дает возможность фирменной символики и настройки пользовательского интерфейса, отображаемого приложением в процессе проверки подлинности.

В следующих разделах показано, как:

- Защита Node.js веб-API
- Поддержка входа в одностраничное приложение (SPA) *и вызов* защищенного веб-API
- Включить поддержку сброса паролей

## <a name="prerequisites"></a>Предварительные условия

Создайте [клиент Azure AD B2C](../../active-directory-b2c/tutorial-create-tenant.md), если вы этого еще не сделали.

## <a name="nodejs-web-api"></a>Веб-API Node.js

В следующих шагах показано, как **веб-API** может использовать Azure AD B2C для защиты и предоставления доступа к выбранным областям клиентскому приложению.

MSAL.js для Node в настоящее время находится в разработке. Дополнительные сведения см. в статье о [планах](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki#roadmap) на GitHub. Сейчас мы советуем использовать [Passport-Azure-AD](https://github.com/AzureAD/passport-azure-ad), библиотеку проверки подлинности для Node.js, разработанную и поддерживаемую корпорацией Майкрософт.

### <a name="step-1-register-your-application"></a>Шаг 1. Регистрация приложения

Чтобы защитить веб-API с помощью Azure AD B2C, необходимо сначала зарегистрировать его. Подробные шаги описаны в статье [Register your application](../../active-directory-b2c/add-web-api-application.md) (Регистрация вашего приложения).

### <a name="step-2-download-the-sample-application"></a>Шаг 2. скачивание примера приложения

Скачайте пример в виде ZIP-файла или скопируйте его из GitHub:

```console
git clone https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi.git
```

### <a name="step-3-configure-authentication"></a>Шаг 3. Настройка проверки подлинности

1. Откройте файл *config.js* в образце.

2. Настройте пример с учетными данными приложения, полученными ранее при регистрации приложения. Измените следующие строки кода, заменив значения именем клиента, ИДЕНТИФИКАТОРом клиента и именем политики.

    ```json
         "credentials": {
             "tenantName": "<your-tenant-name>",
             "clientID": "<your-webapi-application-ID>"
         },
         "policies": {
             "policyName": "B2C_1_signupsignin1"
         },
         "resource": {
             "scope": ["demo.read"] 
         },
    ```

Дополнительные сведения см. в этом [Node.js примере веб-API B2C](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi).

## <a name="javascript-spa"></a>JavaScript SPA

В следующих шагах показано, как **одностраничное приложение** может использовать Azure AD B2C для регистрации, входа и вызова защищенного веб-API.

### <a name="step-1-register-your-application"></a>Шаг 1. Регистрация приложения

Чтобы выполнить аутентификацию, сначала необходимо зарегистрировать приложение. Подробные шаги описаны в статье [Register your application](../../active-directory-b2c/tutorial-register-applications.md) (Регистрация вашего приложения).

### <a name="step-2-download-the-sample-application"></a>Шаг 2. скачивание примера приложения

Скачайте пример кода [. ZIP-архив](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp/archive/master.zip) или клонировать репозиторий GitHub:

```console
git clone https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp.git
```

### <a name="step-3-configure-authentication"></a>Шаг 3. Настройка проверки подлинности

Настройка приложения состоит из двух моментов:

- Настройка конечной точки API и предоставленных областей
- Настройка параметров проверки подлинности и областей токенов

1. Откройте файл *apiConfig.js* в примере.

2. Настройте пример с параметрами, полученными ранее при регистрации веб-API. Измените следующие строки кода, заменив значения адресом веб-API и предоставляемыми областями.

   ```javascript
    // The current application coordinates were pre-registered in a B2C tenant.
    const apiConfig = {
        b2cScopes: ["https://fabrikamb2c.onmicrosoft.com/helloapi/demo.read"], //API scopes you exposed during api registration
        webApi: "https://fabrikamb2chello.azurewebsites.net/hello"
    };
   ```

1. Откройте файл *authConfig.js* в примере.

1. Настройте пример с параметрами, полученными ранее при регистрации одностраничного приложения. Измените следующие строки кода, заменив значения вашими идентификаторами ClientId, центра сертификации и запросами маркеров.

   ```javascript
    // Config object to be passed to Msal on creation.
    const msalConfig = {
        auth: {
            clientId: "e760cab2-b9a1-4c0d-86fb-ff7084abd902",
            authority: "https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/B2C_1_signupsignin1",
            validateAuthority: false
        },
        cache: {
            cacheLocation: "localStorage", // This configures where your cache will be stored
            storeAuthStateInCookie: false // Set this to "true" to save cache in cookies
        }
    };

    // Add here scopes for id token to be used at the MS Identity Platform endpoint
    const loginRequest = {
        scopes: ["openid", "profile"],
    };
   ```

Дополнительные сведения см. в [примере одностраничного приложения JavaScript B2C](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp).

## <a name="support-password-reset"></a>Поддержка сброса пароля

В этом разделе вы расширяете одностраничное приложение, чтобы использовать пользовательский поток сброса пароля Azure AD B2C. Хотя MSAL.js в настоящее время не поддерживает несколько пользовательских потоков или пользовательских политик, вы можете использовать библиотеку для решения распространенных ситуаций использования, таких как сброс пароля.

Следующие шаги предполагают, что вы уже предоставили действия, описанные в предыдущем разделе [JavaScript Spa](#javascript-spa) .

### <a name="step-1-define-the-authority-string-for-password-reset-user-flow"></a>Шаг 1. определение строки центра для потока пользователя сброса пароля

1. Сначала создайте объект, в котором хранятся URI центра:

    ```javascript
        const b2cPolicies = {
            names: {
                signUpSignIn: "b2c_1_susi",
                forgotPassword: "b2c_1_reset"
            },
            authorities: {
                signUpSignIn: {
                    authority: "https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_susi",
                },
                forgotPassword: {
                    authority: "https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_reset",
                },
            },
        }
    ```

1. Затем инициализируйте объект MSAL с помощью `signInSignUp` политики по умолчанию (см. предыдущий фрагмент кода). Когда пользователь пытается войти в систему, он отображается на следующем экране:

    :::image type="content" source="media/msal-b2c-overview/user-journey-01-signin.png" alt-text="Экран входа, отображаемый Azure AD B2C":::

### <a name="step-2-catch-and-handle-authentication-errors-in-your-login-method"></a>Шаг 2. перехват и обработку ошибок проверки подлинности в методе входа

Когда пользователь выбирает команду " **забыли пароль**", приложение выдает ошибку, которую следует перехватить в коде, а затем обработаемся путем представления соответствующего потока пользователя. В этом случае это `b2c_1_reset` поток сброса пароля.

1. Расширьте метод входа следующим образом:

    ```javascript
    function signIn() {
      myMSALObj.loginPopup(loginRequest)
        .then(loginResponse => {
            console.log("id_token acquired at: " + new Date().toString());

            if (myMSALObj.getAccount()) {
              updateUI();
            }

        }).catch(function (error) {
          console.log(error);

          // error handling
          if (error.errorMessage) {
            // check for forgot password error
            if (error.errorMessage.indexOf("AADB2C90118") > -1) {

              //call login method again with the password reset user flow
              myMSALObj.loginPopup(b2cPolicies.authorities.forgotPassword)
                .then(loginResponse => {
                  console.log(loginResponse);
                  window.alert("Password has been reset successfully. \nPlease sign-in with your new password.");
                })
            }
          }
        });
    }
    ```

1. В предыдущем фрагменте кода показано, как отобразить экран сброса пароля после перехвата ошибки с помощью кода `AADB2C90118` .

    После сброса пароля пользователь возвращается в приложение для повторного входа.

    :::image type="content" source="media/msal-b2c-overview/user-journey-02-password-reset.png" alt-text="Экраны потока сброса пароля, показанные Azure AD B2C" border="false":::

    Дополнительные сведения о кодах ошибок и обработке исключений см. в разделе [MSAL Error and Exception Codes](msal-error-handling-js.md).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об этих Azure AD B2C концепциях:

- [Потоки пользователей](../../active-directory-b2c/tutorial-create-user-flows.md)
- [Пользовательские политики](../../active-directory-b2c/custom-policy-get-started.md)
- [Настройка UX](../../active-directory-b2c/configure-user-input.md)
