---
title: Краткое руководство. Получение маркера и вызов Microsoft Graph в консольном приложении | Azure
titleSuffix: Microsoft identity platform
description: В этом кратком руководстве демонстрируется, как пример приложения .NET Core может использовать поток учетных данных клиента для получения маркера и вызова Microsoft Graph.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 10/05/2020
ms.author: jmprieur
ms.reviewer: marsma
ms.custom: devx-track-csharp, aaddev, identityplatformtop40, scenarios:getting-started, languages:aspnet-core
ms.openlocfilehash: 1b539c168deab7c1893f071a2453be28310fc132
ms.sourcegitcommit: a8ff4f9f69332eef9c75093fd56a9aae2fe65122
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105022931"
---
# <a name="quickstart-get-a-token-and-call-the-microsoft-graph-api-by-using-a-console-apps-identity"></a>Краткое руководство. Получение маркера безопасности и вызов API Microsoft Graph из консольного приложения с помощью удостоверения приложения

При работе с этим кратким руководством вы скачаете и выполните пример кода. Такой пример кода демонстрирует, как в консольном приложении .NET Core реализовать получение маркера доступа для вызова API Microsoft Graph и отобразить [список пользователей](/graph/api/user-list) в каталоге, а также как задание или служба Windows могут выполняться с удостоверением приложения, а не пользователя. Пример консольного приложения в этом кратком руководстве представляет собой также управляющее приложение, которое является конфиденциальным клиентским приложением.

> [!div renderon="docs"]
> На следующей схеме показано, как работает пример приложения.
>
> ![Схема: работа примера приложения, созданного при работе с этим кратким руководством.](media/quickstart-v2-netcore-daemon/netcore-daemon-intro.svg)
>

## <a name="prerequisites"></a>Предварительные требования

Для выполнения инструкций этого краткого руководства требуется [.NET Core 3.1](https://www.microsoft.com/net/download/dotnet-core), но можно также использовать .NET Core 5.0.

> [!div renderon="docs"]
> ## <a name="register-and-download-the-app"></a>Регистрация и скачивание приложения

> [!div renderon="docs" class="sxs-lookup"]
>
> Есть два варианта по созданию приложения: автоматическая или ручная настройка.
>
> ### <a name="automatic-configuration"></a>Автоматическая настройка
>
> Если вы хотите зарегистрировать и автоматически настроить приложение, а затем скачать пример кода, выполните следующие действия.
>
> 1. Перейдите на <a href="https://portal.azure.com/?Microsoft_AAD_RegisteredApps=true#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/DotNetCoreDaemonQuickstartPage/sourceType/docs" target="_blank">страницу портала Azure для регистрации приложения</a>.
> 1. Введите имя приложения и нажмите кнопку **Зарегистрировать**.
> 1. Следуйте инструкциям, чтобы скачать и автоматически настроить новое приложение одним щелчком мыши.
>
> ### <a name="manual-configuration"></a>Настройка вручную
>
> Если вы хотите вручную настроить приложение и пример кода, воспользуйтесь следующими процедурами.
>
> [!div renderon="docs"]
> #### <a name="step-1-register-your-application"></a>Шаг 1. Регистрация приложения
> Чтобы зарегистрировать приложение и добавить сведения о его регистрации в решение вручную, сделайте следующее:
>
> 1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</span></a>.
> 1. Если у вас есть доступ к нескольким клиентам, в верхнем меню используйте фильтр **Каталог и подписка** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false":::, чтобы выбрать клиент, в котором следует зарегистрировать приложение.
> 1. Найдите и выберите **Azure Active Directory**.
> 1. В разделе **Управление** выберите **Регистрация приложений** > **Создать регистрацию**.
> 1. В поле **Имя** введите имя для своего приложения. Например, введите **Daemon-console**. Это имя будут видеть пользователи приложения. Вы сможете изменить его позже.
> 1. Выберите **Зарегистрировать**, чтобы создать приложение.
> 1. В разделе **Управление** выберите **Сертификаты и секреты**.
> 1. В разделе **Секреты клиента** выберите **Новый секрет клиента**, введите имя, а затем нажмите кнопку **Добавить**. Сохраните значение секрета в надежном месте, так как это значение потребуется вам в дальнейшем.
> 1. В разделе **Управление** выберите **Разрешения API** > **Добавить разрешение**. Выберите **Microsoft Graph**.
> 1. Выберите **Разрешения приложений**.
> 1. В узле **Пользователь** выберите **User.Read.All**, а затем щелкните **Добавить разрешения**.

> [!div class="sxs-lookup" renderon="portal"]
> ### <a name="download-and-configure-your-quickstart-app"></a>Скачивание и настройка приложения, используемого в этом кратком руководстве
>
> #### <a name="step-1-configure-your-application-in-the-azure-portal"></a>Шаг 1. Настройка приложения на портале Azure
> Для работы примера кода, приведенного в этом кратком руководстве, необходимо создать секрет клиента и добавить разрешение приложения API Graph **User.Read.All**.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Внести эти изменения для меня]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Уже настроено](media/quickstart-v2-netcore-daemon/green-check.png). Ваше приложение настроено с помощью этих атрибутов.

#### <a name="step-2-download-your-visual-studio-project"></a>Шаг 2. Скачивание проекта Visual Studio

> [!div renderon="docs"]
> [Скачайте проект Visual Studio](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/archive/master.zip)
>
> Вы можете работать с этим проектом в Visual Studio или Visual Studio для Mac.


> [!div class="sxs-lookup" renderon="portal"]
> Запустите проект с помощью Visual Studio 2019.
> [!div class="sxs-lookup" renderon="portal" id="autoupdate" class="nextstepaction"]
> [Скачивание примера кода](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/archive/master.zip)

[!INCLUDE [active-directory-develop-path-length-tip](../../../includes/active-directory-develop-path-length-tip.md)]

> [!div class="sxs-lookup" renderon="portal"]
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`

> [!div renderon="docs"]
> #### <a name="step-3-configure-your-visual-studio-project"></a>Шаг 3. Настройка проекта Visual Studio
>
> 1. Извлеките ZIP-файл в локальный каталог рядом с корневым каталогом диска. Например, извлеките в *C:\Azure-Samples*.
>
>    Рекомендуется извлечь архив в каталог рядом с корнем диска, чтобы избежать ошибок, вызванных ограничениями длины пути в Windows.
>
> 1. Откройте решение в Visual Studio: *1-Call-MSGraph\daemon-console.sln* (необязательно).
> 1. В *appsettings.json* замените значения `Tenant`, `ClientId`и `ClientSecret`:
>
>    ```json
>    "Tenant": "Enter_the_Tenant_Id_Here",
>    "ClientId": "Enter_the_Application_Id_Here",
>    "ClientSecret": "Enter_the_Client_Secret_Here"
>    ```
>    В этом коде:
>    - `Enter_the_Application_Id_Here` — это идентификатор приложения (клиента) для зарегистрированного приложения.
        Чтобы найти значения идентификатора приложения (клиента) и каталога (клиента), на портале Azure перейдите на страницу приложения **Обзор**.
>    - Замените `Enter_the_Tenant_Id_Here` идентификатором или именем клиента (например, `contoso.microsoft.com`).
>    - Замените `Enter_the_Client_Secret_Here` на секрет клиента, созданный на шаге 1.
    Чтобы создать ключ, перейдите на страницу **Сертификаты и секреты**.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-admin-consent"></a>Шаг 3. Согласие администратора

> [!div renderon="docs"]
> #### <a name="step-4-admin-consent"></a>Шаг 4. Согласие администратора

Если вы попытаетесь запустить приложение на этом этапе, отобразится сообщение об ошибке *HTTP 403. Запрещено*: "Недостаточно привилегий для выполнения операции". Эта ошибка возникает, так как разрешение "только для приложения" предоставляется с согласия глобального администратора каталога. Выберите один из следующих вариантов с учетом своей роли.

##### <a name="global-tenant-administrator"></a>Глобальный администратор клиента

> [!div renderon="docs"]
> Если вы являетесь глобальным администратором клиента, на портале Azure перейдите на страницу **Корпоративные приложения**. Выберите регистрацию приложения и на панели слева в разделе **Безопасность** выберите **Разрешения**. Затем нажмите большую кнопку **Предоставить согласие администратора для {Tenant Name}** (где **{Tenant Name}**  — это название вашего каталога).

> [!div renderon="portal" class="sxs-lookup"]
> Если вы являетесь глобальным администратором, перейдите на страницу **Разрешения API** и выберите **Предоставить согласие администратора для имя_арендатор**.
> > [!div id="apipermissionspage"]
> > [Переход на страницу "Разрешения API"]()

##### <a name="standard-user"></a>Обычный пользователь

Если вы являетесь обычным пользователем арендатора, попросите глобального администратора предоставить согласие администратора для вашего приложения. Чтобы сделать это, предоставьте следующий URL-адрес администратору:

```url
https://login.microsoftonline.com/Enter_the_Tenant_Id_Here/adminconsent?client_id=Enter_the_Application_Id_Here
```

> [!div renderon="docs"]
> В этом URL-адресе:
> * Замените `Enter_the_Tenant_Id_Here` идентификатором или именем клиента (например, `contoso.microsoft.com`).
> * `Enter_the_Application_Id_Here` — это идентификатор приложения (клиента) для зарегистрированного приложения.

После предоставления согласия для приложения с использованием предыдущего URL-адреса может отобразиться сообщение об ошибке "AADSTS50011: для приложения не зарегистрирован адрес ответа". Такая ошибка происходит, если у этого приложения и URL-адреса нет URI перенаправления. Его можно пропустить.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-4-run-the-application"></a>Шаг 4. Выполнение приложения

> [!div renderon="docs"]
> #### <a name="step-5-run-the-application"></a>Шаг 5. Выполнение приложения

Если используется Visual Studio или Visual Studio для Mac, нажмите клавишу **F5** для запуска приложения. Приложение можно также запустить с помощью командной строки, консоли или терминала:

```dotnetcli
cd {ProjectFolder}\1-Call-MSGraph\daemon-console
dotnet run
```
В этом коде:
* `{ProjectFolder}` — это папка, куда вы извлекли ZIP-файл. Например, `C:\Azure-Samples\active-directory-dotnetcore-daemon-v2`.

В результате вы увидите список пользователей в Azure Active Directory.

В рамках этого краткого руководства приложение использует секрет клиента для собственной идентификации в качестве конфиденциального клиента. Секрет клиента добавляется в файлы проекта как обычный текстовый файл. По соображениям безопасности вместо секрета клиента рекомендуется применить сертификат, прежде чем использовать приложение в качестве рабочего. См. дополнительные сведения о том, как [использовать сертификат](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/#variation-daemon-application-using-client-credentials-with-certificates) в репозитории GitHub.

## <a name="more-information"></a>Дополнительные сведения
В этом разделе представлен код, используемый для выполнения входа пользователей. Это может быть полезно для рассмотрения принципов работы кода и основных аргументов. Вы также поймете, нужно ли добавлять функцию входа в существующее консольное приложение .NET Core.

> [!div class="sxs-lookup" renderon="portal"]
> ### <a name="how-the-sample-works"></a>Как работает этот пример
>
> ![Схема: работа примера приложения, созданного при работе с этим кратким руководством.](media/quickstart-v2-netcore-daemon/netcore-daemon-intro.svg)

### <a name="msalnet"></a>MSAL.NET

Библиотека проверки подлинности Майкрософт (MSAL в пакете [Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client)) — это библиотека, которая используется для входа пользователей и запроса маркеров для доступа к интерфейсу API, защищенному платформой Microsoft Identity. В этом кратком руководстве маркеры запрашиваются с использованием собственного удостоверения приложения вместо делегированных разрешений. Поток проверки подлинности, используемый в данном случае, называется [потоком OAuth для учетных данных клиента](v2-oauth2-client-creds-grant-flow.md). Подробные сведения об использовании MSAL.NET с потоком учетных данных клиента см. в [этой статье](https://aka.ms/msal-net-client-credentials).

 MSAL.NET можно установить, выполнив в консоли диспетчера пакетов Visual Studio следующую команду:

```dotnetcli
dotnet add package Microsoft.Identity.Client
```

### <a name="msal-initialization"></a>Инициализация MSAL

Добавив следующий код, вы можете добавить ссылку на MSAL.

```csharp
using Microsoft.Identity.Client;
```

Затем выполните инициализацию MSAL с помощью следующего кода:

```csharp
IConfidentialClientApplication app;
app = ConfidentialClientApplicationBuilder.Create(config.ClientId)
                                          .WithClientSecret(config.ClientSecret)
                                          .WithAuthority(new Uri(config.Authority))
                                          .Build();
```

 | Элемент | Описание |
 |---------|---------|
 | `config.ClientSecret` | Секрет клиента, созданный для приложения на портале Azure. |
 | `config.ClientId` | Идентификатор приложения (клиента), зарегистрированного на портале Azure. Это значение можно найти на портале Azure на странице приложения **Обзор**. |
 | `config.Authority`    | (Необязательно.) Конечная точка службы токенов безопасности для проверки подлинности пользователей. Обычно `https://login.microsoftonline.com/{tenant}` для общедоступного облака, где `{tenant}` — имя или идентификатор вашего клиента.|

Дополнительные сведения см. в [справочной документации по `ConfidentialClientApplication`](/dotnet/api/microsoft.identity.client.iconfidentialclientapplication).

### <a name="requesting-tokens"></a>Запрос маркеров

Чтобы запросить маркер с помощью удостоверения приложения, используйте метод `AcquireTokenForClient`.

```csharp
result = await app.AcquireTokenForClient(scopes)
                  .ExecuteAsync();
```

|Элемент| Описание |
|---------|---------|
| `scopes` | Содержит запрошенные области. Для конфиденциальных клиентов для этого значения следует использовать формат, аналогичный `{Application ID URI}/.default`. Этот формат указывает, что запрошенные области статически определены в наборе объектов приложения, заданном на портале Azure. Для Microsoft Graph `{Application ID URI}` указывает на `https://graph.microsoft.com`. Для пользовательских веб-API значение `{Application ID URI}` определяется на портале Azure в разделе **Регистрация приложения (предварительная версия)**  > **Предоставление API**. |

Дополнительные сведения см. в [справочной документации по `AcquireTokenForClient`](/dotnet/api/microsoft.identity.client.confidentialclientapplication.acquiretokenforclient).

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об управляющих программах см. следующий обзор сценария:

> [!div class="nextstepaction"]
> [Создание управляющей программы, которая вызывает веб-API](scenario-daemon-overview.md)
