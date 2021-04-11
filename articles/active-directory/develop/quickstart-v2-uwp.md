---
title: Краткое руководство. Реализация входа пользователей и вызова Microsoft Graph в приложении универсальной платформы Windows | Azure
titleSuffix: Microsoft identity platform
description: Из этого краткого руководства вы узнаете, как приложение универсальной платформы Windows (UWP) может получить маркер доступа и вызвать API, защищенный с помощью платформы удостоверений Майкрософт.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 10/07/2020
ms.author: jmprieur
ms.custom: aaddev, identityplatformtop40, scenarios:getting-started, languages:UWP
ms.openlocfilehash: 88fd6260df0039e960ac87b4b83d43ea497a58b0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104658515"
---
# <a name="quickstart-call-the-microsoft-graph-api-from-a-universal-windows-platform-uwp-application"></a>Краткое руководство. Вызов API Microsoft Graph из приложения для универсальной платформы Windows (UWP)

При работе с этим кратким руководством вы скачаете и выполните пример кода. Такой пример кода демонстрирует, как в приложении универсальной платформы Windows (UWP) реализовать вход пользователей и получение маркера доступа для вызова API Microsoft Graph. 

Иллюстрацию см. в разделе [Как работает этот пример](#how-the-sample-works).

> [!div renderon="docs"]
> ## <a name="prerequisites"></a>Предварительные требования
>
> * Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
> * [Visual Studio 2019](https://visualstudio.microsoft.com/vs/)
>
> ## <a name="register-and-download-your-quickstart-app"></a>Регистрация и скачивание приложения, используемого в этом кратком руководстве
> [!div renderon="docs" class="sxs-lookup"]
> У вас есть два варианта запуска приложения, используемого в этом кратком руководстве:
> * [Экспресс-способ] [Вариант 1. Регистрация и автоматическая настройка приложения, а затем скачивание примера кода](#option-1-register-and-auto-configure-your-app-and-then-download-your-code-sample)
> * [Вручную] [Вариант 2. Регистрация и настройка приложения и примера кода вручную](#option-2-register-and-manually-configure-your-application-and-code-sample)
>
> ### <a name="option-1-register-and-auto-configure-your-app-and-then-download-your-code-sample"></a>Вариант 1. Регистрация и автоматическая настройка приложения, а затем скачивание примера кода
>
> 1. Откройте страницу <a href="https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/UwpQuickstartPage/sourceType/docs" target="_blank">регистрации приложений</a> на портале Azure и приступите к работе.
> 1. Введите имя приложения и нажмите кнопку **Зарегистрировать**.
> 1. Следуйте инструкциям для загрузки и автоматической настройки нового приложения одним щелчком мыши.
>
> ### <a name="option-2-register-and-manually-configure-your-application-and-code-sample"></a>Вариант 2. Регистрация и настройка приложения и примера кода вручную
> [!div renderon="docs"]
> #### <a name="step-1-register-your-application"></a>Шаг 1. Регистрация приложения
> Чтобы зарегистрировать приложение и добавить сведения о его регистрации в решение, сделайте следующее:
> 1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>.
> 1. Если у вас есть доступ к нескольким клиентам, в верхнем меню используйте фильтр **Каталог и подписка** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false":::, чтобы выбрать клиент, в котором следует зарегистрировать приложение.
> 1. Найдите и выберите **Azure Active Directory**.
> 1. В разделе **Управление** выберите **Регистрация приложений** > **Создать регистрацию**.
> 1. Введите **имя** приложения, например `UWP-App-calling-MsGraph`. Пользователи приложения могут видеть это имя. Вы можете изменить его позже.
> 1. В разделе **Поддерживаемые типы учетных записей** выберите **Accounts in any organizational directory and personal Microsoft accounts (for example, Skype, Xbox, Outlook.com)** (Учетные записи в любом каталоге организации и личные учетные записи Майкрософт (например, Skype, Xbox, Outlook.com)).
> 1. Выберите пункт **Регистрация**, чтобы создать приложение и запишите **Идентификатор приложения (клиент)** , он понадобится на следующих этапах.
> 1. В разделе **Управление** выберите **Проверка подлинности**.
> 1. Поочередно выберите **Добавить платформу** > **Мобильные и классические приложения**.
> 1. В разделе **URI перенаправления** выберите `https://login.microsoftonline.com/common/oauth2/nativeclient`.
> 1. Нажмите кнопку **Настроить**.

> [!div renderon="portal" class="sxs-lookup"]
> #### <a name="step-1-configure-the-application"></a>Шаг 1. Настройка приложения
> Чтобы пример кода, приведенный в этом кратком руководстве, работал, добавьте **URI перенаправления** `https://login.microsoftonline.com/common/oauth2/nativeclient`.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Внести это изменение для меня]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Уже настроено](media/quickstart-v2-uwp/green-check.png). Ваше приложение настроено с помощью этих атрибутов.

#### <a name="step-2-download-the-visual-studio-project"></a>Шаг 2. Скачайте проект Visual Studio.

> [!div renderon="docs"]
> [Скачайте проект Visual Studio](https://github.com/Azure-Samples/active-directory-dotnet-native-uwp-v2/archive/msal3x.zip)

> [!div class="sxs-lookup" renderon="portal"]
> Запустите проект с помощью Visual Studio 2019.
> [!div class="sxs-lookup" renderon="portal" id="autoupdate" class="nextstepaction"]
> [Скачивание примера кода](https://github.com/Azure-Samples/active-directory-dotnet-native-uwp-v2/archive/msal3x.zip)

[!INCLUDE [active-directory-develop-path-length-tip](../../../includes/active-directory-develop-path-length-tip.md)]

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-your-app-is-configured-and-ready-to-run"></a>Шаг 3. Приложение настроено и готово к запуску
> Мы уже настроили для проекта нужные значения свойств приложения, и его можно запускать.

> [!div class="sxs-lookup" renderon="portal"]
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`

> [!div renderon="docs"]
> #### <a name="step-3-configure-the-visual-studio-project"></a>Шаг 3. Настройка проекта Visual Studio
>
> 1. Извлеките ZIP-архив в локальную папку рядом с корнем диска. Например, в папку **C:\Azure-Samples**.
> 1. Откройте проект в Visual Studio. При наличии запроса установите рабочую нагрузку **Разработка приложений для универсальной платформы Windows** и отдельные компоненты пакета SDK.
> 1. В *MainPage.Xaml.cs* замените значение переменной `ClientId` на **идентификатор приложения (клиент)** , которое вы зарегистрировали ранее.
>
>    ```csharp
>    private const string ClientId = "Enter_the_Application_Id_here";
>    ```
>
>    **Идентификатор приложения (клиент)** можно найти в приложении на панели **Обзор** на портале Azure (**Azure Active Directory** > **Регистрация приложений** >  *{регистрация вашего приложения}* ).
> 1. Создайте и выберите новый самозаверяющий тестовый сертификат для пакета:
>     1. В **Обозревателе решений** дважды щелкните файл *Package.appxmanifest*.
>     1. Выберите **Упаковка** > **Выбор сертификата...**  > **Создать...** .
>     1. Введите пароль и нажмите **OK**.
>     1. Нажмите **Выбрать из файла...** и выберите только что созданный вами файл *Native_UWP_V2_TemporaryKey.pfx*, после чего нажмите **OK**.
>     1. Закройте файл *Package.appxmanifest* (нажмите **OK** при запросе на сохранение файла).
>     1. В **Обозревателе решений** нажмите правой кнопкой мыши проект **Native_UWP_V2** и выберите пункт **Свойства**.
>     1. Выберите **Подпись**,а затем созданный PFX-файл в раскрывающемся списке **Выберите файл ключа строгого имени**.

#### <a name="step-4-run-the-application"></a>Шаг 4. Выполнение приложения

Для запуска примера приложения на локальном компьютере:

1. На панели инструментов Visual Studio выберите нужную платформу (**x64** или **x86**, но не ARM). Значение целевого устройства должно измениться с *Устройство* на *Локальный компьютер*.
1. Выберите **Отладка** > **Запустить без отладки**.
    
    Если вам будет предложено, включите **Режим разработчика**, а затем еще раз нажмите **Запустить без отладки**, чтобы запустить приложение.

Когда откроется окно приложения, можно нажать кнопку **Call Microsoft Graph API** (Вызов API Microsoft Graph), ввести данные для входа и дать согласие на разрешения, запрашиваемые приложением. В случае успешного выполнения приложение отобразит информацию о токенах и данные, полученные в результате вызова API Microsoft Graph.

## <a name="how-the-sample-works"></a>Как работает этот пример

![Схема работы приложения, создаваемого в этом кратком руководстве](media/quickstart-v2-uwp/uwp-intro.svg)

### <a name="msalnet"></a>MSAL.NET

MSAL ([Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client)) — это библиотека, используемая для выполнения входа пользователей и запросов маркеров безопасности. Маркеры безопасности нужны для доступа к API, защищенному платформой удостоверений Майкрософт. MSAL можно установить, выполнив в *консоли диспетчера пакетов* Visual Studio следующую команду.

```powershell
Install-Package Microsoft.Identity.Client
```

### <a name="msal-initialization"></a>Инициализация MSAL

Добавив следующий код, вы можете добавить ссылку на MSAL.

```csharp
using Microsoft.Identity.Client;
```

Затем выполните инициализацию MSAL с помощью следующего кода:

```csharp
public static IPublicClientApplication PublicClientApp;
PublicClientApp = PublicClientApplicationBuilder.Create(ClientId)
                                                .WithRedirectUri("https://login.microsoftonline.com/common/oauth2/nativeclient")
                                                    .Build();
```

Значение `ClientId` является **идентификатором приложения (клиент)** , зарегистрированного на портале Azure. Это значение можно найти на странице приложения **Обзор** на портале Azure.

### <a name="requesting-tokens"></a>Запрос маркеров

В MSAL есть два метода получения маркеров в приложении UWP: `AcquireTokenInteractive` и `AcquireTokenSilent`.

#### <a name="get-a-user-token-interactively"></a>Интерактивное получение маркера пользователя

В некоторых ситуациях необходимо принудительное взаимодействие пользователей с платформой удостоверений Майкрософт через всплывающее окно, чтобы подтвердить учетные данные или предоставить согласие. Некоторые примеры:

- первый вход пользователей в приложение;
- когда пользователям может потребоваться повторно ввести учетные данные, так как истек срок действия пароля;
- когда ваше приложение запрашивает доступ к ресурсу, для чего пользователь должен предоставить согласие;
- когда требуется двухфакторная проверка подлинности.

```csharp
authResult = await App.PublicClientApp.AcquireTokenInteractive(scopes)
                      .ExecuteAsync();
```

Параметр `scopes` содержит запрашиваемые области, такие как `{ "user.read" }` для Microsoft Graph или `{ "api://<Application ID>/access_as_user" }` для пользовательских веб-API.

#### <a name="get-a-user-token-silently"></a>Автоматическое получение маркера пользователя

Используйте метод `AcquireTokenSilent`, чтобы получить маркеры безопасности для доступа к защищенным ресурсам после первоначального метода `AcquireTokenInteractive`. Вы не хотите требовать от пользователя проверки своих учетных данных каждый раз, когда ему необходимо получить доступ к ресурсу. Большую часть времени вы хотите приобретать и обновлять маркеры без какого-либо взаимодействия с пользователем.

```csharp
var accounts = await App.PublicClientApp.GetAccountsAsync();
var firstAccount = accounts.FirstOrDefault();
authResult = await App.PublicClientApp.AcquireTokenSilent(scopes, firstAccount)
                                      .ExecuteAsync();
```

* `scopes` содержит запрашиваемые области, такие как `{ "user.read" }` для Microsoft Graph или `{ "api://<Application ID>/access_as_user" }` для пользовательских веб-API.
* `firstAccount` указывает первую учетную запись в кэше (MSAL поддерживает несколько пользователей в одном приложении).

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

В руководстве по классическому приложению Windows вы найдете пошаговые инструкции по созданию приложений и компонентов, в том числе полное описание того, о чем говорится в этом кратком руководстве.

> [!div class="nextstepaction"]
> [Вызов API Microsoft Graph из приложения для универсальной платформы Windows (XAML)](tutorial-v2-windows-uwp.md)
