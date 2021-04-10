---
title: Руководство по Создание приложения Windows Presentation Foundation (WPF), использующего платформу удостоверений Майкрософт для проверки подлинности | Azure
titleSuffix: Microsoft identity platform
description: В этом руководстве показано, как создать приложение WPF, которое использует платформу удостоверений Майкрософт для реализации входа пользователей и как получить маркер доступа для вызова API Microsoft Graph от имени пользователей.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 12/12/2019
ms.author: jmprieur
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: 60a29efc4d2daa9d1bc90f00e71094da382a83b9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101686892"
---
# <a name="tutorial-call-the-microsoft-graph-api-from-a-windows-desktop-app"></a>Руководство по Вызов API Microsoft Graph из классического приложения для Windows

В этом руководстве показано, как создать нативное классическое приложение Windows .NET (XAML), которое поддерживает вход пользователей и может получить маркер доступа для вызова API Microsoft Graph. 

Когда вы завершите работу с руководством, ваше приложение сможет вызывать защищенный API, использующий личные учетные записи (в том числе outlook.com, live.com и другие). Приложение будет также использовать рабочие и учебные учетные записи из любой компании или организации, использующей Azure Active Directory.

В этом руководстве рассматриваются следующие темы:

> [!div class="checklist"]
> * создание проекта *Windows Presentation Foundation (WPF)* в Visual Studio;
> * установка библиотеки аутентификации Майкрософт (MSAL) для .NET;
> * регистрация приложения на портале Azure;
> * добавление кода для поддержки входа и выхода пользователей;
> * добавление кода для вызова API Microsoft Graph;
> * Тестирование приложения

## <a name="prerequisites"></a>Предварительные требования

* [Visual Studio 2019](https://visualstudio.microsoft.com/vs/)

## <a name="how-the-sample-app-generated-by-this-guide-works"></a>Как работает пример приложения, созданный в этом руководстве

![Схема работы примера приложения, создаваемого в этом кратком руководстве](./media/active-directory-develop-guidedsetup-windesktop-intro/windesktophowitworks.svg)

Пример приложения, создаваемый с помощью этого руководства, позволяет классическому приложению для Windows выполнять запрос к API Microsoft Graph или веб-API, принимающему токены от конечной точки платформы удостоверений Майкрософт. В этом сценарии маркер добавляется в запросы HTTP с помощью заголовка авторизации. Получение маркера и его обновление выполняется с помощью библиотеки проверки подлинности Майкрософт (MSAL).

## <a name="handling-token-acquisition-for-accessing-protected-web-apis"></a>Обработка получения токена для доступа к защищенным веб-API

Когда пользователь пройдет аутентификацию, пример приложения получит токен, который может использоваться для запрашивания API Microsoft Graph или веб-API, защищенного с помощью платформы удостоверений Майкрософт.

Программным интерфейсам, таким как Microsoft Graph, для доступа к определенным ресурсам требуется маркер. Например, маркер необходим для чтения профиля пользователя, доступа к календарю пользователя и отправки электронной почты. Приложение может запросить маркер доступа с помощью MSAL, чтобы получить доступ к этим ресурсам, указав определенные области API. Затем этот маркер доступа добавляется в заголовок авторизации HTTP для каждого вызова к защищенному ресурсу.

MSAL управляет кэшированием и обновлением маркеров доступа, поэтому вашему приложению не нужно этого делать.

## <a name="nuget-packages"></a>Пакеты NuGet

В этом руководстве используются следующие пакеты NuGet:

|Библиотека|Описание|
|---|---|
|[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client)|Библиотека аутентификации Майкрософт (MSAL)|

## <a name="set-up-your-project"></a>Настройка проекта

В этом разделе показано, как создать проект и интегрировать классическое приложение для Windows.NET (XAML) с возможностью *входа с учетной записью Майкрософт*, чтобы приложение могло выполнять запрос к веб-API, требующим токен.

В приложении, которое будет создано при работе с этим руководством, отображается кнопка, используемая для вызова графа, области для отображения результатов на экране, и кнопка выхода.

> [!NOTE]
> Предпочитаете скачать этот пример проекта Visual Studio? [Скачайте проект](https://github.com/Azure-Samples/active-directory-dotnet-desktop-msgraph-v2/archive/msal3x.zip) и перейдите к [настройке](#register-your-application), чтобы настроить пример кода перед его выполнением.
>

Для создания приложения выполните следующие действия.

1. В Visual Studio выберите **Файл** > **Создать** > **Проект**.
2. В разделе **Шаблоны** выберите **Visual C#** .
3. Выберите **WPF App** (.NET Framework) в зависимости от используемой версии Visual Studio.

## <a name="add-msal-to-your-project"></a>Добавление MSAL в проект

1. В Visual Studio выберите **Сервис** > **Диспетчер пакетов NuGet**> **Консоль диспетчера пакетов**.
2. В окне консоли диспетчера пакетов вставьте следующую команду Azure PowerShell:

    ```powershell
    Install-Package Microsoft.Identity.Client -Pre
    ```

    > [!NOTE]
    > Эта команда устанавливает библиотеку проверки подлинности Майкрософт. MSAL обрабатывает получение, кэширование и обновление маркеров пользователей, которые используются для доступа к программным интерфейсам, защищенным Azure Active Directory версии 2.0
    >

## <a name="register-your-application"></a>Регистрация приложения

Приложение можно зарегистрировать одним из двух способов.

### <a name="option-1-express-mode"></a>Вариант 1. Экспресс-режим

Чтобы быстро зарегистрировать приложение, сделайте следующее:
1. Откройте страницу <a href="https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/WinDesktopQuickstartPage/sourceType/docs" target="_blank">регистрации приложений</a> на портале Azure и приступите к работе.
1. Введите имя приложения и нажмите кнопку **Зарегистрировать**.
1. Следуйте инструкциям, чтобы быстро скачать и автоматически настроить новое приложение.

### <a name="option-2-advanced-mode"></a>Вариант 2. Расширенный режим

Чтобы зарегистрировать приложение и добавить сведения о его регистрации в решение, сделайте следующее:
1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>.
1. Если у вас есть доступ к нескольким клиентам, в верхнем меню используйте фильтр **Каталог и подписка** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false":::, чтобы выбрать клиент, в котором следует зарегистрировать приложение.
1. Найдите и выберите **Azure Active Directory**.
1. В разделе **Управление** выберите **Регистрация приложений** > **Создать регистрацию**.
1. Введите **имя** приложения, например `Win-App-calling-MsGraph`. Пользователи приложения могут видеть это имя. Вы можете изменить его позже.
1. В разделе **Поддерживаемые типы учетных записей** выберите вариант **Учетные записи в любом каталоге организации (любой каталог Azure AD — мультитенантный) и персональные учетные записи Майкрософт (например, Skype, Xbox)** .
1. Выберите **Зарегистрировать**.
1. В разделе **Управление** выберите **Проверка подлинности** > **Добавить платформу**.
1. Выберите вариант **Мобильные и классические приложения**.
1. В разделе **URI перенаправления** выберите **https://login.microsoftonline.com/common/oauth2/nativeclient** .
1. Нажмите кнопку **Настроить**.
1. Перейдите в Visual Studio, откройте файл *App.xaml.cs*, а затем замените `Enter_the_Application_Id_here` в представленном ниже фрагменте кода только что зарегистрированным и скопированным идентификатором приложения.

    ```csharp
    private static string ClientId = "Enter_the_Application_Id_here";
    ```

## <a name="add-the-code-to-initialize-msal"></a>Добавление кода для инициализации MSAL

На этом шаге вы создадите класс для обработки взаимодействия с MSAL, например обработки маркеров.

1. Откройте файл *App.xaml.cs*, а затем добавьте ссылку для MSAL в класс:

    ```csharp
    using Microsoft.Identity.Client;
    ```
   <!-- Workaround for Docs conversion bug -->

2. Обновите класс app следующим образом:

    ```csharp
    public partial class App : Application
    {
        static App()
        {
            _clientApp = PublicClientApplicationBuilder.Create(ClientId)
                .WithAuthority(AzureCloudInstance.AzurePublic, Tenant)
                .WithDefaultRedirectUri()
                .Build();
        }

        // Below are the clientId (Application Id) of your app registration and the tenant information.
        // You have to replace:
        // - the content of ClientID with the Application Id for your app registration
        // - the content of Tenant by the information about the accounts allowed to sign-in in your application:
        //   - For Work or School account in your org, use your tenant ID, or domain
        //   - for any Work or School accounts, use `organizations`
        //   - for any Work or School accounts, or Microsoft personal account, use `common`
        //   - for Microsoft Personal account, use consumers
        private static string ClientId = "0b8b0665-bc13-4fdc-bd72-e0227b9fc011";

        private static string Tenant = "common";

        private static IPublicClientApplication _clientApp ;

        public static IPublicClientApplication PublicClientApp { get { return _clientApp; } }
    }
    ```

## <a name="create-the-application-ui"></a>Создание пользовательского интерфейса приложения

В этом разделе описывается, как приложение может запрашивать защищенный внутренний сервер, например Microsoft Graph.

Файл *MainWindow.xaml* должен создаваться автоматически как часть шаблона проекта. Откройте этот файл и замените узел *\<Grid>* вашего приложения следующим кодом:

```xml
<Grid>
    <StackPanel Background="Azure">
        <StackPanel Orientation="Horizontal" HorizontalAlignment="Right">
            <Button x:Name="CallGraphButton" Content="Call Microsoft Graph API" HorizontalAlignment="Right" Padding="5" Click="CallGraphButton_Click" Margin="5" FontFamily="Segoe Ui"/>
            <Button x:Name="SignOutButton" Content="Sign-Out" HorizontalAlignment="Right" Padding="5" Click="SignOutButton_Click" Margin="5" Visibility="Collapsed" FontFamily="Segoe Ui"/>
        </StackPanel>
        <Label Content="API Call Results" Margin="0,0,0,-5" FontFamily="Segoe Ui" />
        <TextBox x:Name="ResultText" TextWrapping="Wrap" MinHeight="120" Margin="5" FontFamily="Segoe Ui"/>
        <Label Content="Token Info" Margin="0,0,0,-5" FontFamily="Segoe Ui" />
        <TextBox x:Name="TokenInfoText" TextWrapping="Wrap" MinHeight="70" Margin="5" FontFamily="Segoe Ui"/>
    </StackPanel>
</Grid>
```

## <a name="use-msal-to-get-a-token-for-the-microsoft-graph-api"></a>Получение маркера для API Microsoft Graph с помощью MSAL

В этом разделе вы получите маркер для API Microsoft Graph с помощью MSAL.

1. В файле *MainWindow.xaml.cs* добавьте ссылку для MSAL в класс:

    ```csharp
    using Microsoft.Identity.Client;
    ```

2. Замените код класса `MainWindow` следующим:

    ```csharp
    public partial class MainWindow : Window
    {
        //Set the API Endpoint to Graph 'me' endpoint
        string graphAPIEndpoint = "https://graph.microsoft.com/v1.0/me";

        //Set the scope for API call to user.read
        string[] scopes = new string[] { "user.read" };


        public MainWindow()
        {
            InitializeComponent();
        }

      /// <summary>
        /// Call AcquireToken - to acquire a token requiring user to sign-in
        /// </summary>
        private async void CallGraphButton_Click(object sender, RoutedEventArgs e)
        {
            AuthenticationResult authResult = null;
            var app = App.PublicClientApp;
            ResultText.Text = string.Empty;
            TokenInfoText.Text = string.Empty;

            var accounts = await app.GetAccountsAsync();
            var firstAccount = accounts.FirstOrDefault();

            try
            {
                authResult = await app.AcquireTokenSilent(scopes, firstAccount)
                    .ExecuteAsync();
            }
            catch (MsalUiRequiredException ex)
            {
                // A MsalUiRequiredException happened on AcquireTokenSilent.
                // This indicates you need to call AcquireTokenInteractive to acquire a token
                System.Diagnostics.Debug.WriteLine($"MsalUiRequiredException: {ex.Message}");

                try
                {
                    authResult = await app.AcquireTokenInteractive(scopes)
                        .WithAccount(accounts.FirstOrDefault())
                        .WithPrompt(Prompt.SelectAccount)
                        .ExecuteAsync();
                }
                catch (MsalException msalex)
                {
                    ResultText.Text = $"Error Acquiring Token:{System.Environment.NewLine}{msalex}";
                }
            }
            catch (Exception ex)
            {
                ResultText.Text = $"Error Acquiring Token Silently:{System.Environment.NewLine}{ex}";
                return;
            }

            if (authResult != null)
            {
                ResultText.Text = await GetHttpContentWithToken(graphAPIEndpoint, authResult.AccessToken);
                DisplayBasicTokenInfo(authResult);
                this.SignOutButton.Visibility = Visibility.Visible;
            }
        }
        }
    ```

### <a name="more-information"></a>Дополнительные сведения

#### <a name="get-a-user-token-interactively"></a>Интерактивное получение маркера пользователя

При вызове метода `AcquireTokenInteractive` появится окно, в котором пользователю предлагается выполнить вход. Когда пользователь впервые запрашивает доступ к защищенному ресурсу, приложение обычно требует выполнить интерактивный вход. Пользователям также может потребоваться выполнить вход при сбое автоматической операции получения маркера (например, по истечении срока действия пароля пользователя).

#### <a name="get-a-user-token-silently"></a>Автоматическое получение маркера пользователя

Метод `AcquireTokenSilent` обрабатывает получение и обновление маркера без участия пользователя. После первого выполнения метода `AcquireTokenInteractive` обычно используется метод `AcquireTokenSilent`, чтобы получить маркеры для доступа к защищенным ресурсам для последующих вызовов, так как вызовы для запроса или обновления маркеров выполняются автоматически.

Иногда метод `AcquireTokenSilent` завершается ошибкой. Причина может заключаться в том, что пользователь вышел из системы или изменил пароль на другом устройстве. Когда MSAL обнаруживает, что эту проблему можно решить, запросив интерактивное действие, возникает исключение `MsalUiRequiredException`. Приложение может обработать это исключение двумя способами:

* Может немедленно вызвать `AcquireTokenInteractive`. Этот вызов приводит к появлению запроса на вход пользователя. Этот шаблон обычно используется в интерактивных приложениях, где пользователю недоступно автономное содержимое. Этот шаблон используется в примере, созданном в ходе настройки с помощью этих инструкций. Вы увидите его в действии при первом запуске примера.

* Так как приложение еще не использовалось, `PublicClientApp.Users.FirstOrDefault()` будет содержать значение NULL и будет выдано исключение `MsalUiRequiredException`.

* Код в примере обработает исключение, вызвав `AcquireTokenInteractive`, в результате чего пользователю будет предложено войти.

* Также приложение может визуально уведомить пользователей, что требуется интерактивный вход, чтобы они могли выбрать подходящее время для входа. Либо приложение может попытаться повторно выполнить метод `AcquireTokenSilent` позже. Этот шаблон часто применяется, когда пользователи могут использовать другие функциональные возможности приложения без прерывания работы — например, если в приложении доступно автономное содержимое. В этом случае пользователи могут решать, что им нужно сделать после входа — получить доступ к защищенному ресурсу или обновить устаревшие данные. Кроме того, приложение может попытаться повторно выполнить `AcquireTokenSilent` при восстановлении сети после временной недоступности.

## <a name="call-the-microsoft-graph-api-by-using-the-token-you-just-obtained"></a>Вызов API Microsoft Graph с помощью полученного маркера

Добавьте в `MainWindow.xaml.cs` следующий новый метод. Этот метод используется для выполнения запроса `GET` к API Graph с помощью заголовка авторизации "Authorize":

```csharp
/// <summary>
/// Perform an HTTP GET request to a URL using an HTTP Authorization header
/// </summary>
/// <param name="url">The URL</param>
/// <param name="token">The token</param>
/// <returns>String containing the results of the GET operation</returns>
public async Task<string> GetHttpContentWithToken(string url, string token)
{
    var httpClient = new System.Net.Http.HttpClient();
    System.Net.Http.HttpResponseMessage response;
    try
    {
        var request = new System.Net.Http.HttpRequestMessage(System.Net.Http.HttpMethod.Get, url);
        //Add the token in Authorization header
        request.Headers.Authorization = new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", token);
        response = await httpClient.SendAsync(request);
        var content = await response.Content.ReadAsStringAsync();
        return content;
    }
    catch (Exception ex)
    {
        return ex.ToString();
    }
}
```

### <a name="more-information-about-making-a-rest-call-against-a-protected-api"></a>Дополнительные сведения о вызове REST через защищенный API

В этом примере приложения используется метод `GetHttpContentWithToken` для выполнения HTTP-запроса `GET` к защищенному ресурсу, требующему маркер, а затем возвращает содержимое вызывающему объекту. Этот метод добавляет полученный маркер в заголовок авторизации HTTP. В этом примере ресурс — это конечная точка *me* API Microsoft Graph, которая отображает сведения о профиле пользователя.

## <a name="add-a-method-to-sign-out-a-user"></a>Добавление метода для выхода пользователя

Добавьте следующий метод в файл `MainWindow.xaml.cs` для выхода пользователя:

```csharp
/// <summary>
/// Sign out the current user
/// </summary>
private async void SignOutButton_Click(object sender, RoutedEventArgs e)
{
    var accounts = await App.PublicClientApp.GetAccountsAsync();

    if (accounts.Any())
    {
        try
        {
            await App.PublicClientApp.RemoveAsync(accounts.FirstOrDefault());
            this.ResultText.Text = "User has signed-out";
            this.CallGraphButton.Visibility = Visibility.Visible;
            this.SignOutButton.Visibility = Visibility.Collapsed;
        }
        catch (MsalException ex)
        {
            ResultText.Text = $"Error signing-out user: {ex.Message}";
        }
    }
}
```

### <a name="more-information-about-user-sign-out"></a>Дополнительные сведения о выходе пользователя

Метод `SignOutButton_Click` удаляет пользователей из кэша пользователей MSAL. Фактически это приводит к удалению данных текущего пользователя из MSAL, поэтому в дальнейшем успешно выполняться будут только интерактивные запросы на получение маркеров.

Хотя приложение в этом примере поддерживает одного пользователя, MSAL поддерживает сценарии, в которых несколько пользователей могут войти в систему одновременно. Пример — приложение электронной почты, в котором у пользователя есть несколько учетных записей.

## <a name="display-basic-token-information"></a>Отображение основных сведений о маркере

Для отображения основных сведений о маркере добавьте следующий метод в файл *MainWindow.xaml.cs*:

```csharp
/// <summary>
/// Display basic information contained in the token
/// </summary>
private void DisplayBasicTokenInfo(AuthenticationResult authResult)
{
    TokenInfoText.Text = "";
    if (authResult != null)
    {
        TokenInfoText.Text += $"Username: {authResult.Account.Username}" + Environment.NewLine;
        TokenInfoText.Text += $"Token Expires: {authResult.ExpiresOn.ToLocalTime()}" + Environment.NewLine;
    }
}
```

### <a name="more-information"></a>Дополнительные сведения

После входа пользователя MSAL получает идентификатор маркера в дополнение к маркеру доступа, который используется для вызова Microsoft Graph API. Этот маркер содержит небольшое подмножество сведений, имеющие отношение к пользователям. Метод `DisplayBasicTokenInfo` отображает основные сведения, содержащиеся в маркере. Например, он показывает отображаемое имя и идентификатор пользователя, а также дату истечения срока действия маркера и строку, представляющую сам маркер доступа. Нажав кнопку *вызова API Microsoft Graph* несколько раз, вы увидите, что для последующих запросов повторно используется тот же маркер. Вы также можете увидеть, что библиотека MSAL продлила срок действия.

[!INCLUDE [5. Test and Validate](../../../includes/active-directory-develop-guidedsetup-windesktop-test.md)]

## <a name="next-steps"></a>Дальнейшие действия

Из нашей серии сценариев узнайте, как создавать классические приложения, вызывающие защищенные веб-API.

> [!div class="nextstepaction"]
> [Сценарий: Классическое приложение, которое вызывает веб-API](scenario-desktop-overview.md)
