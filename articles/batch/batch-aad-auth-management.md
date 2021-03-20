---
title: Аутентификация решений по управлению пакетной службой с помощью Azure Active Directory
description: Узнайте больше о выполнении аутентификации в приложениях, применяющих библиотеку .NET для управления пакетной службой, с помощью Azure Active Directory.
ms.topic: how-to
ms.date: 04/27/2017
ms.custom: has-adal-ref, devx-track-csharp
ms.openlocfilehash: ff49d5e88df7c56ed4dee0685f09e45eb372aa5c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88930219"
---
# <a name="authenticate-batch-management-solutions-with-active-directory"></a>Аутентификация решений по управлению пакетной службой с помощью Active Directory

Аутентификация приложений, вызывающих службу управления пакетной службой Azure, выполняется с помощью [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md) (Azure AD). Azure AD — это облачный каталог и служба управления удостоверениями, основанная на нескольких клиентах Майкрософт. Инфраструктура Azure уже использует Azure AD для проверки подлинности клиентов, администраторов служб и пользователей организации.

Библиотека .NET для управления пакетной службой предоставляет типы для работы с учетными записями пакетной службы, ключами учетной записи, приложениями и пакетами приложений. Данная библиотека является клиентом поставщика ресурсов Azure и используется совместно с [Azure Resource Manager](../azure-resource-manager/management/overview.md) для программного управления этими ресурсами. Служба Azure AD необходима для аутентификации запросов, выполненных с помощью любого клиента поставщика ресурсов Azure, а также библиотеки .NET для управления пакетной службой и Azure Resource Manager.

В этой статье рассматривается выполнение аутентификации в приложениях, применяющих библиотеку .NET для управления пакетной службой, с помощью Azure AD. Мы покажем, как использовать Azure AD для аутентификации администратора или соадминистратора подписки с помощью встроенной аутентификации. Чтобы изучить использование Azure AD с библиотекой .NET для управления пакетной службой, мы используем пример проекта [AccountManagement](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/AccountManagement), который доступен на сайте GitHub.

Дополнительные сведения об использовании библиотеки .NET для управления пакетной службой и примера AccountManagement см. в статье [Управление учетными записями и квотами пакетной службы с помощью клиентской библиотеки .NET для управления пакетной службой](batch-management-dotnet.md).

## <a name="register-your-application-with-azure-ad"></a>Регистрация приложения в Azure AD

[Библиотека проверки Подлинности Azure Active Directory](../active-directory/azuread-dev/active-directory-authentication-libraries.md) (ADAL) предоставляет программный интерфейс Azure AD для использования в приложениях. Чтобы вызвать ADAL в приложении, необходимо зарегистрировать приложение в клиенте Azure AD. При регистрации приложения в клиенте Azure AD нужно предоставить Azure AD сведения о приложении, включая его имя. После этого служба Azure AD предоставит идентификатор приложения, позволяющий связать с ней приложение во время выполнения. Дополнительные сведения об идентификаторе приложения см. в статье [Объекты приложения и субъекта-службы в Azure Active Directory](../active-directory/develop/app-objects-and-service-principals.md).

Чтобы зарегистрировать пример приложения AccountManagement, выполните инструкции, приведенные в разделе [Добавление приложения](../active-directory/develop/quickstart-register-app.md) статьи [Интеграция приложений с Azure Active Directory](../active-directory/develop/quickstart-register-app.md). Укажите в качестве типа приложения **собственное клиентское приложение**. `urn:ietf:wg:oauth:2.0:oob` — это стандартный отраслевой универсальный код ресурса (URI) OAuth 2.0, используемый в качестве **URI перенаправления**. Однако можно указать любой допустимый универсальный код ресурса (например, `http://myaccountmanagementsample` ) для **URI перенаправления**, так как он не обязательно должен быть реальной конечной точкой.

![Добавление приложения](./media/batch-aad-auth-management/app-registration-management-plane.png)

После завершения процесса регистрации отобразится указанный для вашего приложения идентификатор приложения и объекта (субъекта-службы).

![Процесс регистрации завершен](./media/batch-aad-auth-management/app-registration-client-id.png)

## <a name="grant-the-azure-resource-manager-api-access-to-your-application"></a>Предоставление доступа API Azure Resource Manager к приложению

Теперь необходимо делегировать доступ к приложению для API Azure Resource Manager. Идентификатором Azure AD для API Resource Manager является **API управления службами Windows Azure**.

На портале Azure сделайте следующее:

1. В области навигации слева на портале Azure выберите **Все службы**, щелкните **Регистрация приложений**, а затем нажмите кнопку **Добавить**.
2. Найдите имя приложения в списке зарегистрированных приложений:

    ![Поиск имени приложения](./media/batch-aad-auth-management/search-app-registration.png)

3. Откройте колонку **Параметры**. В разделе **Доступ через API** выберите **Необходимые разрешения**.
4. Щелкните **Добавить** для добавления нового необходимого разрешения.
5. На шаге 1 введите **API управления службами Windows Azure**, выберите этот API в списке результатов и нажмите кнопку **Выбрать**.
6. На шаге 2 установите флажок рядом с параметром **Access Azure classic deployment model as organization users** (Доступ к классической модели развертывания Azure от имени пользователей организации) и нажмите кнопку **Выбрать**.
7. Нажмите кнопку **Готово**.

Теперь в колонке **Необходимые разрешения** показано, что разрешения на доступ к приложению предоставлены и интерфейсам API Resource Manager, и ADAL. Разрешения ADAL предоставляются по умолчанию во время регистрации приложения в Azure AD.

![Делегирование разрешений для API Azure Resource Manager](./media/batch-aad-auth-management/required-permissions-management-plane.png)

## <a name="azure-ad-endpoints"></a>Конечные точки Azure AD

Чтобы выполнить аутентификацию решений по управлению пакетной службой с помощью Azure AD, необходимо иметь две известные конечные точки.

- **Общая конечная точка Azure AD** предоставляет универсальный интерфейс для сбора учетных данных при отсутствии конкретного клиента (как в случае со встроенной аутентификацией):

    `https://login.microsoftonline.com/common`

- **Конечная точка Azure Resource Manager** используется для получения токена для аутентификации запросов к службе управления пакетной службой:

    `https://management.core.windows.net/`

Пример приложения AccountManagement определяет константы для этих конечных точек. Оставьте их без изменений:

```csharp
// Azure Active Directory "common" endpoint.
private const string AuthorityUri = "https://login.microsoftonline.com/common";
// Azure Resource Manager endpoint
private const string ResourceUri = "https://management.core.windows.net/";
```

## <a name="reference-your-application-id"></a>Ссылка на идентификатор приложения

Клиентское приложение использует идентификатор приложения (также называемый идентификатором клиента) для доступа к Azure AD во время выполнения. После регистрации приложения на портале Azure обновите код, указав в нем идентификатор приложения, предоставленный Azure AD для зарегистрированного приложения. В примере приложения AccountManagement скопируйте идентификатор приложения из портала Azure в соответствующую константу:

```csharp
// Specify the unique identifier (the "Client ID") for your application. This is required so that your
// native client application (i.e. this sample) can access the Microsoft Graph API. For information
// about registering an application in Azure Active Directory, please see "Register an application with the Microsoft identity platform" here:
// https://docs.microsoft.com/azure/active-directory/develop/quickstart-register-app
private const string ClientId = "<application-id>";
```
Скопируйте также URI перенаправления, который вы указали во время процесса регистрации. URI перенаправления, указанный в коде, должен соответствовать URI перенаправления, который был указан при регистрации приложения.

```csharp
// The URI to which Azure AD will redirect in response to an OAuth 2.0 request. This value is
// specified by you when you register an application with AAD (see ClientId comment). It does not
// need to be a real endpoint, but must be a valid URI (e.g. https://accountmgmtsampleapp).
private const string RedirectUri = "http://myaccountmanagementsample";
```

## <a name="acquire-an-azure-ad-authentication-token"></a>Получение маркера проверки подлинности Azure AD

После регистрации приложения AccountManagement в клиенте Azure AD и добавления собственных значений в пример исходного кода этот пример можно использовать для аутентификации с помощью Azure AD. При запуске примера ADAL пытается получить маркер проверки подлинности. На этом этапе предлагается ввести учетные данные Майкрософт:

```csharp
// Obtain an access token using the "common" AAD resource. This allows the application
// to query AAD for information that lies outside the application's tenant (such as for
// querying subscription information in your Azure account).
AuthenticationContext authContext = new AuthenticationContext(AuthorityUri);
AuthenticationResult authResult = authContext.AcquireToken(ResourceUri,
                                                        ClientId,
                                                        new Uri(RedirectUri),
                                                        PromptBehavior.Auto);
```

После указания учетных данных пример приложения может отправлять запросы, прошедшие проверку подлинности, к службе управления пакетами.

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о выполнении [примера приложения AccountManagement](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/AccountManagement) см. в статье [Управление учетными записями и квотами пакетной службы с помощью клиентской библиотеки .NET для управления пакетной службой](batch-management-dotnet.md).
- Дополнительные сведения о службе Azure AD см. в [документации по Azure Active Directory](../active-directory/index.yml).
- Подробные примеры, показывающие, как использовать ADAL, доступны в библиотеке [примеров кода Azure](https://azure.microsoft.com/resources/samples/?service=active-directory).
- Дополнительные сведения об аутентификации приложений пакетной службы с помощью Azure AD см. в статье [Аутентификация решений пакетной службы с помощью Active Directory](batch-aad-auth.md).
