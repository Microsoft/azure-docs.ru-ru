---
title: Управление ресурсами с помощью Microsoft Graph
titleSuffix: Azure AD B2C
description: Управление ресурсами в клиенте Azure AD B2C путем вызова API Microsoft Graph и использования удостоверения приложения для автоматизации процесса.
services: B2C
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 01/28/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 41336d59d51685d5daf78a1809ce6c0df2cd6124
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104781319"
---
# <a name="manage-azure-ad-b2c-with-microsoft-graph"></a>Управление Azure AD B2C с помощью Microsoft Graph

Microsoft Graph позволяет управлять ресурсами в каталоге Azure AD B2C. Следующие операции API Microsoft Graph поддерживаются для управления Azure AD B2C ресурсами, включая пользователей, поставщиков удостоверений, потоки пользователей, пользовательские политики и ключи политики. Каждая ссылка в следующих разделах предназначена для соответствующей страницы в справочнике по Microsoft Graph API для этой операции. 

## <a name="prerequisites"></a>Предварительные требования

Чтобы использовать MS API Graph и взаимодействовать с ресурсами в клиенте Azure AD B2C, необходима регистрация приложения, предоставляющая разрешения на это. Выполните действия, описанные в статье [управление Azure AD B2C с Microsoft Graph](microsoft-graph-get-started.md) , чтобы создать регистрацию приложения, которую может использовать приложение управления. 

## <a name="user-management"></a>управление пользователями;

- [Список пользователей](/graph/api/user-list)
- [Создание пользователя-получателя](/graph/api/user-post-users)
- [Получение пользователя](/graph/api/user-get)
- [Обновление пользователя](/graph/api/user-update)
- [Удаление пользователя](/graph/api/user-delete)

## <a name="user-phone-number-management-beta"></a>Управление номером телефона пользователя (бета-версия)

Номер телефона, который может использоваться пользователем для входа с помощью [SMS или голосовой связи](identity-provider-local.md#phone-sign-in-preview)или [многофакторной проверки подлинности](multi-factor-authentication.md). Дополнительные сведения см. в статье [API методов проверки подлинности Azure AD](/graph/api/resources/phoneauthenticationmethod).

- [Добавление](/graph/api/authentication-post-phonemethods)
- [Список](/graph/api/authentication-list-phonemethods)
- [Получить](/graph/api/phoneauthenticationmethod-get)
- [Обновление](/graph/api/phoneauthenticationmethod-update)
- [Удалить](/graph/api/phoneauthenticationmethod-delete)

Обратите внимание, что операция [List](/graph/api/authentication-list-phonemethods) возвращает только включенные номера телефонов. Следующий номер телефона должен быть включен для использования с операциями списка. 

![Включить вход с помощью телефона](./media/microsoft-graph-operations/enable-phone-sign-in.png)

## <a name="self-service-password-reset-email-address-beta"></a>Самостоятельный адрес электронной почты для сброса пароля (бета-версия)

Адрес электронной почты, который может использоваться [учетной записью входа пользователя](identity-provider-local.md#username-sign-in) для сброса пароля. Дополнительные сведения см. в статье [API методов проверки подлинности Azure AD](/graph/api/resources/emailauthenticationmethod).

- [Добавление](/graph/api/emailauthenticationmethod-post)
- [Список](/graph/api/emailauthenticationmethod-list)
- [Получить](/graph/api/emailauthenticationmethod-get)
- [Обновление](/graph/api/emailauthenticationmethod-update)
- [Удалить](/graph/api/emailauthenticationmethod-delete)

## <a name="identity-providers"></a>Поставщики удостоверений

Управление [поставщиками удостоверений](add-identity-provider.md) , доступными для пользователя, в клиенте Azure AD B2C.

- [Список поставщиков удостоверений, зарегистрированных в клиенте Azure AD B2C](/graph/api/identityprovider-list)
- [Создание поставщика удостоверений](/graph/api/identityprovider-post-identityproviders)
- [Получение поставщика удостоверений](/graph/api/identityprovider-get)
- [Обновление поставщика удостоверений](/graph/api/identityprovider-update)
- [Удаление поставщика удостоверений](/graph/api/identityprovider-delete)

## <a name="user-flow"></a>Поток пользователя

Настройте предварительно созданные политики для регистрации, входа, объединения и входа в систему, сброса пароля и обновления профиля.

- [Перечисление потоков пользователей](/graph/api/identitycontainer-list-b2cuserflows)
- [Создание потока пользователей](/graph/api/identitycontainer-post-b2cuserflows)
- [Получение потока пользователя](/graph/api/b2cidentityuserflow-get)
- [Удаление потока пользователя](/graph/api/b2cidentityuserflow-delete)

## <a name="user-flow-authentication-methods-beta"></a>Методы проверки подлинности потока пользователей (бета-версия)

Выберите механизм, позволяющий регистрировать пользователей с помощью локальных учетных записей. Локальные учетные записи — это учетные записи, в которых Azure AD выполняет утверждение удостоверения. Дополнительные сведения см. в разделе [тип ресурса b2cAuthenticationMethodsPolicy](/graph/api/resources/b2cauthenticationmethodspolicy).

- [Получить](/graph/api/b2cauthenticationmethodspolicy-get)
- [Обновление](/graph/api/b2cauthenticationmethodspolicy-update)

## <a name="custom-policies"></a>Пользовательские политики

Следующие операции позволяют управлять политиками инфраструктуры доверия Azure AD B2C, называемыми [пользовательскими политиками](custom-policy-overview.md).

- [Вывод списка всех политик инфраструктуры доверия, настроенных в клиенте](/graph/api/trustframework-list-trustframeworkpolicies)
- [Создание политики инфраструктуры доверия](/graph/api/trustframework-post-trustframeworkpolicy)
- [Чтение свойств существующей политики инфраструктуры доверия](/graph/api/trustframeworkpolicy-get)
- [Обновите или создайте политику инфраструктуры доверия.](/graph/api/trustframework-put-trustframeworkpolicy)
- [Удаление существующей политики инфраструктуры доверия](/graph/api/trustframeworkpolicy-delete)

## <a name="policy-keys"></a>Ключи политики

Инфраструктура процедур идентификации хранит секреты, на которые ссылается пользовательская политика, для установления отношений доверия между компонентами. Эти секреты могут быть симметричными или асимметричными ключами или значениями. В портал Azure эти сущности отображаются в виде **ключей политики**.

Ресурс верхнего уровня для ключей политики в Microsoft Graph API — это [набор ключей доверенной платформы](/graph/api/resources/trustframeworkkeyset). Каждый **набор ключей** содержит по крайней мере один **ключ**. Чтобы создать ключ, сначала создайте пустой набор ключей, а затем создайте ключ в наборе ключей. Можно создать секрет вручную, отправить сертификат или PKCS12 ключ. Ключом может быть созданный секрет, строка (например, секрет приложения Facebook) или отправленный сертификат. Если набор ключей содержит несколько ключей, то активен только один из них.

### <a name="trust-framework-policy-keyset"></a>Набор ключей политики доверенной инфраструктуры

- [Перечисление инфраструктуры доверия формируются](/graph/api/trustframework-list-keysets)
- [Создание инфраструктуры доверия формируются](/graph/api/trustframework-post-keysets)
- [Получение набора ключей](/graph/api/trustframeworkkeyset-get)
- [Обновление инфраструктуры доверия формируются](/graph/api/trustframeworkkeyset-update)
- [Удаление инфраструктуры доверия формируются](/graph/api/trustframeworkkeyset-delete)

### <a name="trust-framework-policy-key"></a>Ключ политики инфраструктуры доверия

- [Получить активный в данный момент ключ в наборе ключей](/graph/api/trustframeworkkeyset-getactivekey)
- [Создание ключа в наборе ключей](/graph/api/trustframeworkkeyset-generatekey)
- [Отправка секрета на основе строки](/graph/api/trustframeworkkeyset-uploadsecret)
- [Отправка сертификата X. 509](/graph/api/trustframeworkkeyset-uploadcertificate)
- [Отправка сертификата формата PKCS12](/graph/api/trustframeworkkeyset-uploadpkcs12)

## <a name="applications"></a>Приложения

- [Список приложений](/graph/api/application-list)
- [Создание приложения](/graph/api/resources/application)
- [Update application](/graph/api/application-update)
- [Создание servicePrincipal](/graph/api/resources/serviceprincipal)
- [Создание oauth2Permission Grant](/graph/api/resources/oauth2permissiongrant)
- [Удаление приложения](/graph/api/application-delete)

## <a name="application-extension-properties"></a>Свойства расширения приложения

- [Список свойств расширения](/graph/api/application-list-extensionproperty)

Azure AD B2C предоставляет каталог, который может содержать 100 настраиваемых атрибутов для каждого пользователя. Для потоков пользователей эти свойства расширения [управляются с помощью портал Azure](user-flow-custom-attributes.md). Для настраиваемых политик Azure AD B2C создает свойство для вас в первый раз, когда политика записывает значение в свойство расширения.

## <a name="audit-logs"></a>Журналы аудита

- [Список журналов аудита](/graph/api/directoryaudit-list)

Дополнительные сведения о доступе к журналам аудита Azure AD B2C см. в разделе [доступ к журналам аудита Azure AD B2C](view-audit-logs.md).

## <a name="conditional-access"></a>Условный доступ

- [Вывод списка всех политик условного доступа](/graph/api/conditionalaccessroot-list-policies?view=graph-rest-beta&tabs=http)
- [Чтение свойств и отношений политики условного доступа](/graph/api/conditionalaccesspolicy-get)
- [Создание политики условного доступа](/graph/api/resources/application)
- [Обновление политики условного доступа](/graph/api/conditionalaccesspolicy-update)
- [Удаление политики условного доступа](/graph/api/conditionalaccesspolicy-delete)

## <a name="code-sample-how-to-programmatically-manage-user-accounts"></a>Пример кода: программное управление учетными записями пользователей

Этот пример кода является консольным приложением .NET Core, которое использует [пакет SDK для Microsoft Graph](/graph/sdks/sdks-overview) для взаимодействия с Microsoft Graph API. В его коде показано, как вызывать API для программного управления пользователями в клиенте Azure AD B2C.
Вы можете [скачать пример архива](https://github.com/Azure-Samples/ms-identity-dotnetcore-b2c-account-management/archive/master.zip) (*. zip), [просмотреть репозиторий](https://github.com/Azure-Samples/ms-identity-dotnetcore-b2c-account-management) на сайте GitHub или клонировать репозиторий:

```cmd
git clone https://github.com/Azure-Samples/ms-identity-dotnetcore-b2c-account-management.git
```

После получения примера кода настройте его для своей среды, а затем выполните сборку проекта:

1. Откройте проект в [Visual Studio](https://visualstudio.microsoft.com) или [Visual Studio Code](https://code.visualstudio.com).
1. Откройте среду `src/appsettings.json`.
1. В `appSettings` разделе замените `your-b2c-tenant` именем своего клиента и `Application (client) ID` `Client secret` значениями для регистрации приложения управления. Дополнительные сведения см. [в разделе Регистрация приложения Microsoft Graph](microsoft-graph-get-started.md).
1. Откройте окно консоли в локальном клоне репозитория, перейдите в `src` каталог, а затем выполните сборку проекта:

    ```console
    cd src
    dotnet build
    ```
    
1. Запустите приложение, выполнив команду `dotnet`:

    ```console
    dotnet bin/Debug/netcoreapp3.1/b2c-ms-graph.dll
    ```

В приложении отображается список команд, которые можно выполнить. Например, можно получить всех пользователей, получить одного пользователя, удалить пользователя, обновить пароль пользователя и выполнить групповое импортирование.

### <a name="code-discussion"></a>Обсуждение кода

В примере кода используется [пакет SDK Microsoft Graph](/graph/sdks/sdks-overview), предназначенный для упрощения создания высококачественных, эффективных и устойчивых приложений, обращающихся к Microsoft Graph.

Для любого запроса к Microsoft Graph API требуется маркер доступа для проверки подлинности. Решение использует пакет NuGet [Microsoft. Graph. auth](https://www.nuget.org/packages/Microsoft.Graph.Auth/) , который предоставляет оболочку на основе сценария проверки подлинности Майкрософт (MSAL) для использования с пакетом SDK для Microsoft Graph.

`RunAsync`Метод в файле _Program. CS_ :

1. Чтение параметров приложения из _appsettings.js_ файла
1. Инициализирует поставщик проверки подлинности с помощью потока [предоставления учетных данных клиента OAuth 2,0](../active-directory/develop/v2-oauth2-client-creds-grant-flow.md) . Используя поток предоставления учетных данных клиента, приложение может получить маркер доступа для вызова API Microsoft Graph.
1. Настраивает Клиент службы Microsoft Graph с помощью поставщика проверки подлинности:

    ```csharp
    // Read application settings from appsettings.json (tenant ID, app ID, client secret, etc.)
    AppSettings config = AppSettingsFile.ReadFromJsonFile();

    // Initialize the client credential auth provider
    IConfidentialClientApplication confidentialClientApplication = ConfidentialClientApplicationBuilder
        .Create(config.AppId)
        .WithTenantId(config.TenantId)
        .WithClientSecret(config.ClientSecret)
        .Build();
    ClientCredentialProvider authProvider = new ClientCredentialProvider(confidentialClientApplication);

    // Set up the Microsoft Graph service client with client credentials
    GraphServiceClient graphClient = new GraphServiceClient(authProvider);
    ```

Затем инициализированный *графсервицеклиент* используется в _UserService. CS_ для выполнения операций управления пользователями. Например, получение списка учетных записей пользователей в клиенте:

```csharp
public static async Task ListUsers(GraphServiceClient graphClient)
{
    Console.WriteLine("Getting list of users...");

    // Get all users (one page)
    var result = await graphClient.Users
        .Request()
        .Select(e => new
        {
            e.DisplayName,
            e.Id,
            e.Identities
        })
        .GetAsync();

    foreach (var user in result.CurrentPage)
    {
        Console.WriteLine(JsonConvert.SerializeObject(user));
    }
}
```

[Выполнение вызовов API с помощью Microsoft Graph пакетов SDK](/graph/sdks/create-requests) включает сведения о том, как считывать и записывать данные из Microsoft Graph, использовать `$select` для управления возвращаемыми свойствами, предоставлять настраиваемые параметры запроса и использовать `$filter` `$orderBy` Параметры запроса и.

<!-- LINK -->

[graph-objectIdentity]: /graph/api/resources/objectidentity
[graph-user]: (https://docs.microsoft.com/graph/api/resources/user)
