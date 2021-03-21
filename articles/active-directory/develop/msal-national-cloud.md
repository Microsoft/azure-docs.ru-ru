---
title: Использование MSAL в приложении национального облака | Службы
titleSuffix: Microsoft identity platform
description: Библиотека проверки подлинности Майкрософт (MSAL) позволяет разработчикам приложений получать маркеры для вызова защищенных веб-API. Эти веб-API можно Microsoft Graph, других API Майкрософт, партнерских веб-API или собственного веб-API. MSAL поддерживает несколько архитектур и платформ приложений.
services: active-directory
author: negoe
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 11/22/2019
ms.author: negoe
ms.reviewer: marsma, nacanuma
ms.custom: aaddev
ms.openlocfilehash: 09c4dadd7a6560bd5163d623dd8a7f247b57860e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100102501"
---
# <a name="use-msal-in-a-national-cloud-environment"></a>Использование MSAL в национальной облачной среде

[Местные облака](authentication-national-cloud.md), также известные как независимых облака, являются физически изолированными экземплярами Azure. Эти области справки Azure обеспечивают соблюдение требований к местонахождениеам данных, независимости и соответствию в географических границах.

Помимо Microsoft Worldwide, Библиотека проверки подлинности Майкрософт (MSAL) позволяет разработчикам приложений в национальных облаках получать маркеры для проверки подлинности и вызова защищенных веб-API. Эти веб-API можно Microsoft Graph или других интерфейсов API Майкрософт.

Включая глобальное облако, Azure Active Directory (Azure AD) развертывается в следующих национальных облаках:  

- Azure для государственных организаций
- Azure China 21Vianet
- Azure для Германии

В этом руководстве показано, как войти в рабочие и учебные учетные записи, получить маркер доступа и вызвать API Microsoft Graph в [облачной среде Azure для государственных организаций](https://azure.microsoft.com/global-infrastructure/government/) .

## <a name="prerequisites"></a>Предварительные условия

Прежде чем начать, убедитесь, что выполнены все необходимые условия.

### <a name="choose-the-appropriate-identities"></a>Выберите соответствующие удостоверения

Приложения [Azure для государственных организаций](../../azure-government/index.yml) могут использовать удостоверения государственных организаций Azure AD и общедоступные удостоверения Azure AD для проверки подлинности пользователей. Так как вы можете использовать любое из этих удостоверений, решите, какую конечную точку следует выбрать для своего сценария:

- Общедоступная служба Azure AD. обычно используется, если в Организации уже есть общедоступный клиент Azure AD для поддержки Microsoft 365 (Public или GCC) или другого приложения.
- Azure AD для государственных организаций. обычно используется, если в Организации уже есть клиент Azure AD для государственных организаций, поддерживающий Office 365 (GCC High или DoD) или создающий новый клиент в Azure AD для государственных организаций.

После принятия решения обратите особое внимание на то, где выполняется регистрация приложения. Если вы выбрали общедоступные удостоверения Azure AD для приложения Azure для государственных организаций, необходимо зарегистрировать приложение в общедоступном клиенте Azure AD.

### <a name="get-an-azure-government-subscription"></a>Получение подписки Azure для государственных организаций

Чтобы получить подписку Azure для государственных организаций, см. статью [Управление подпиской и подключение к ней в Azure](../../azure-government/compare-azure-government-global-azure.md)для государственных организаций.

Если у вас нет подписки Azure для государственных организаций, создайте [бесплатную учетную запись](https://azure.microsoft.com/global-infrastructure/government/request/) , прежде чем начинать работу.

Для получения сведений об использовании национального облака с определенным языком программирования выберите вкладку, соответствующую вашему языку:

## <a name="net"></a>[.NET](#tab/dotnet)

Вы можете использовать MSAL.NET для входа пользователей, получения маркеров и вызова API Microsoft Graph в национальных облаках.

В следующих руководствах показано, как создать веб-приложение MVC для .NET Core 2,2. Приложение использует OpenID Connect Connect для входа пользователей с рабочей и учебной учетной записью в Организации, которая принадлежит национальной облаку.

- Чтобы войти в систему пользователей и получить маркеры, следуйте указаниям в этом руководстве: [Создание пользователей ASP.NET Core для входа в веб-приложение в облаках независимых с платформой Microsoft Identity](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/1-WebApp-OIDC/1-4-Sovereign#build-an-aspnet-core-web-app-signing-in-users-in-sovereign-clouds-with-the-microsoft-identity-platform).
- Чтобы вызвать API Microsoft Graph, следуйте указаниям в этом руководстве: [использование платформы Microsoft Identity для вызова api Microsoft Graph из веб-приложения ASP.NET Core 2. x от имени входа пользователя с помощью рабочей и учебной учетной записи в Microsoft National Cloud](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/2-WebApp-graph-user/2-4-Sovereign-Call-MSGraph#using-the-microsoft-identity-platform-to-call-the-microsoft-graph-api-from-an-an-aspnet-core-2x-web-app-on-behalf-of-a-user-signing-in-using-their-work-and-school-account-in-microsoft-national-cloud).

## <a name="javascript"></a>[JavaScript](#tab/javascript)

Чтобы включить MSAL.jsное приложение для облаков независимых:

### <a name="step-1-register-your-application"></a>Шаг 1. Регистрация приложения

1. Войдите на <a href="https://portal.azure.us/" target="_blank">портал Azure</a>.

   Чтобы найти портал Azure конечные точки для других национальных облаков, см. раздел [конечные точки регистрации приложений](authentication-national-cloud.md#app-registration-endpoints).

1. Если у вас есть доступ к нескольким клиентам, в верхнем меню используйте фильтр **Каталог и подписка** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false":::, чтобы выбрать клиент, в котором следует зарегистрировать приложение.
1. Найдите и выберите **Azure Active Directory**.
1. В разделе **Управление** выберите **Регистрация приложений** > **Создать регистрацию**.
1. Введите значение **Name** (Имя) для приложения. Пользователи приложения могут видеть это имя. Вы можете изменить его позже.
1. В разделе **Поддерживаемые типы учетных записей** выберите **Учетные записи в любом каталоге организации**.
1. В разделе **URI перенаправления** выберите **веб-** платформу и задайте в качестве значения URL-адрес приложения, основанный на веб-сервере. Инструкции по установке и получению URL-адреса перенаправления в Visual Studio и узле см. в следующих разделах.
1. Выберите **Зарегистрировать**.
1. На странице **Обзор** запишите значение **идентификатора приложения (клиента)** для последующего использования.
    Для работы с этим руководством необходимо включить [неявный поток предоставления](v2-oauth2-implicit-grant-flow.md). 
1. В разделе **Управление** выберите **Проверка подлинности**.
1. В разделе **неявное предоставление и гибридные потоки** выберите **токены идентификации** и **маркеры доступа**. Маркеры идентификации и маркеры доступа являются обязательными, так как это приложение должно входить в систему пользователей и вызывать API.
1. Щелкните **Сохранить**.

### <a name="step-2--set-up-your-web-server-or-project"></a>Шаг 2. Настройка веб-сервера или проекта

- [Скачайте файлы проекта](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/archive/quickstart.zip) для локального веб-сервера, например node.

  или диспетчер конфигурации служб

- [Скачайте проект Visual Studio](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/archive/vsquickstart.zip).

Затем перейдите к [настройке JavaScript Spa](#step-4-configure-your-javascript-spa) , чтобы настроить пример кода перед его запуском.

### <a name="step-3-use-the-microsoft-authentication-library-to-sign-in-the-user"></a>Шаг 3. Использование библиотеки проверки подлинности Майкрософт для входа пользователя

Выполните действия, описанные в [руководстве по JavaScript](tutorial-v2-javascript-spa.md#create-your-project) , чтобы создать проект и интегрировать его с MSAL, чтобы войти в систему.

### <a name="step-4-configure-your-javascript-spa"></a>Шаг 4. Настройка JavaScript SPA

Внесите сведения о регистрации приложения в файл `index.html`, созданный во время настройки проекта. В начале вашего файла `index.html` между тегами `<script></script>` вставьте следующий код:

```javascript
const msalConfig = {
    auth:{
        clientId: "Enter_the_Application_Id_here",
        authority: "https://login.microsoftonline.us/Enter_the_Tenant_Info_Here",
        }
}

const graphConfig = {
        graphEndpoint: "https://graph.microsoft.us",
        graphScopes: ["user.read"],
}

// create UserAgentApplication instance
const myMSALObj = new UserAgentApplication(msalConfig);
```

В этом коде:

- `Enter_the_Application_Id_here` — Это значение **идентификатора приложения (клиента)** для зарегистрированного приложения.
- `Enter_the_Tenant_Info_Here` может иметь несколько значений:
    - Если приложение поддерживает **учетные записи в этом каталоге Организации**, замените это значение на идентификатор клиента или имя клиента (например, contoso.Microsoft.com).
    - Если приложение поддерживает вариант **Учетные записи в любом каталоге организации**, замените это значение на `organizations`.

    Чтобы найти конечные точки проверки подлинности для всех национальных облаков, см. раздел [конечные точки аутентификации Azure AD](./authentication-national-cloud.md#azure-ad-authentication-endpoints).

    > [!NOTE]
    > Личные учетные записи Майкрософт не поддерживаются в национальных облаках.

- `graphEndpoint` является Microsoft Graph конечной точкой для Microsoft Cloud для государственных организаций США.

   Чтобы найти Microsoft Graph конечных точек для всех национальных облаков, см. раздел [Microsoft Graph конечных точек в национальных облаках](/graph/deployments#microsoft-graph-and-graph-explorer-service-root-endpoints).

## <a name="python"></a>[Python](#tab/python)

Чтобы включить приложение MSAL Python для облаков независимых, выполните следующие действия.

- Зарегистрируйте приложение на определенном портале в зависимости от облака. Дополнительные сведения о выборе портала см. в разделе [конечные точки регистрации приложений](authentication-national-cloud.md#app-registration-endpoints) .
- Используйте любой из [примеров](https://github.com/AzureAD/microsoft-authentication-library-for-python/tree/dev/sample) из репозитория с внесением нескольких изменений в конфигурацию, в зависимости от облака, которое упоминается далее.
- Используйте конкретный центр в зависимости от облака, в котором вы зарегистрировали приложение. Дополнительные сведения о центрах для различных облаков см. в разделе [Аутентификация Azure AD конечные точки](authentication-national-cloud.md#azure-ad-authentication-endpoints).

    Вот пример центра:

    ```json
    "authority": "https://login.microsoftonline.us/Enter_the_Tenant_Info_Here"
    ```

- Для вызова API Microsoft Graph требуется URL-адрес конечной точки, относящийся к используемому облаку. Чтобы найти Microsoft Graph конечных точек для всех национальных облаков, см. статью [Microsoft Graph и корневые конечные точки службы Graph Explorer](/graph/deployments#microsoft-graph-and-graph-explorer-service-root-endpoints).

    Ниже приведен пример конечной точки Microsoft Graph с областью действия:

    ```json
    "endpoint" : "https://graph.microsoft.us/v1.0/me"
    "scope": "User.Read"
    ```

## <a name="java"></a>[Java](#tab/java)

Чтобы включить приложение MSAL для Java в облаках независимых, выполните следующие действия.

- Зарегистрируйте приложение на определенном портале в зависимости от облака. Дополнительные сведения о выборе портала см. в разделе [конечные точки регистрации приложений](authentication-national-cloud.md#app-registration-endpoints) .
- Используйте любой из [примеров](https://github.com/AzureAD/microsoft-authentication-library-for-java/tree/dev/src/samples) из репозитория с небольшими изменениями в конфигурации, в зависимости от облака, которые упоминались далее.
- Используйте конкретный центр в зависимости от облака, в котором вы зарегистрировали приложение. Дополнительные сведения о центрах для различных облаков см. в разделе [Аутентификация Azure AD конечные точки](authentication-national-cloud.md#azure-ad-authentication-endpoints).

Вот пример центра:

```json
"authority": "https://login.microsoftonline.us/Enter_the_Tenant_Info_Here"
```

- Для вызова API Microsoft Graph требуется URL-адрес конечной точки, относящийся к используемому облаку. Чтобы найти Microsoft Graph конечных точек для всех национальных облаков, см. статью [Microsoft Graph и корневые конечные точки службы Graph Explorer](/graph/deployments#microsoft-graph-and-graph-explorer-service-root-endpoints).

Ниже приведен пример конечной точки графа с областью действия:

```json
"endpoint" : "https://graph.microsoft.us/v1.0/me"
"scope": "User.Read"
```

## <a name="objective-c"></a>[Objective-C](#tab/objc)

MSAL для iOS и macOS можно использовать для получения маркеров в национальных облаках, но при создании требуется дополнительная настройка `MSALPublicClientApplication` .

Например, если вы хотите, чтобы приложение было многопользовательским приложением в национальном облаке (здесь правительство США), можно написать:

```objc
MSALAADAuthority *aadAuthority =
                [[MSALAADAuthority alloc] initWithCloudInstance:MSALAzureUsGovernmentCloudInstance
                                                   audienceType:MSALAzureADMultipleOrgsAudience
                                                      rawTenant:nil
                                                          error:nil];

MSALPublicClientApplicationConfig *config =
                [[MSALPublicClientApplicationConfig alloc] initWithClientId:@"<your-client-id-here>"
                                                                redirectUri:@"<your-redirect-uri-here>"
                                                                  authority:aadAuthority];

NSError *applicationError = nil;
MSALPublicClientApplication *application =
                [[MSALPublicClientApplication alloc] initWithConfiguration:config error:&applicationError];
```

## <a name="swift"></a>[Swift](#tab/swift)

MSAL для iOS и macOS можно использовать для получения маркеров в национальных облаках, но при создании требуется дополнительная настройка `MSALPublicClientApplication` .

Например, если вы хотите, чтобы приложение было многопользовательским приложением в национальном облаке (здесь правительство США), можно написать:

```swift
let authority = try? MSALAADAuthority(cloudInstance: .usGovernmentCloudInstance, audienceType: .azureADMultipleOrgsAudience, rawTenant: nil)

let config = MSALPublicClientApplicationConfig(clientId: "<your-client-id-here>", redirectUri: "<your-redirect-uri-here>", authority: authority)
if let application = try? MSALPublicClientApplication(configuration: config) { /* Use application */}
```

---

## <a name="next-steps"></a>Дальнейшие действия

Список URL-адресов портал Azure и конечных точек токенов для каждого облака см. в разделе [местные облачные конечные точки аутентификации](authentication-national-cloud.md) .

Документация по национальной облаку:

- [Azure для государственных организаций](../../azure-government/index.yml)
- [Azure China 21Vianet](/azure/china/)
- [Azure для Германии](../../germany/index.yml)