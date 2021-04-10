---
title: Руководство по Создание приложения универсальной платформы Windows (UWP), которое использует платформу удостоверений Майкрософт для аутентификации | Azure
titleSuffix: Microsoft identity platform
description: В этом учебнике показано, как создать приложение UWP, которое использует платформу удостоверений Майкрософт для реализации входа пользователей, и как получить маркер доступа для вызова API Microsoft Graph от имени пользователей.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 12/13/2019
ms.author: jmprieur
ms.custom: devx-track-csharp, aaddev, identityplatformtop40
ms.openlocfilehash: 6383f63d2118d8618f07bf3cb6cd08a0b16140f3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100102654"
---
# <a name="tutorial-call-the-microsoft-graph-api-from-a-universal-windows-platform-uwp-application"></a>Руководство по Вызов API Microsoft Graph из приложения для универсальной платформы Windows (UWP)

В этом руководстве показано, как создать нативное приложение универсальной платформы Windows (UWP), которое поддерживает вход пользователей и может получить маркер доступа для вызова API Microsoft Graph. 

В конце этого руководства приложение вызывает защищенный API-интерфейс с использованием личных учетных записей, таких как outlook.com, live.com и другие. Приложение также вызывает рабочие и учебные учетные записи из любой компании или организации, использующей Azure Active Directory (Azure AD).

В этом руководстве рассматриваются следующие темы:

> [!div class="checklist"]
> * создание проекта *универсальной платформы Windows (UWP)* в Visual Studio;
> * регистрация приложения на портале Azure;
> * добавление кода для поддержки входа и выхода пользователей;
> * добавление кода для вызова API Microsoft Graph;
> * Тестирование приложения

## <a name="prerequisites"></a>Предварительные требования

* [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) с установленной рабочей нагрузкой [Разработка приложений для универсальной платформы Windows](/windows/uwp/get-started/get-set-up).

## <a name="how-this-guide-works"></a>Принцип работы с руководством

![Схема работы примера приложения, создаваемого в этом кратком руководстве](./media/tutorial-v2-windows-uwp/uwp-intro.svg)

В рамках этого руководства создается приложение UWP, которое обращается к API Microsoft Graph. В этом сценарии маркер добавляется в запросы HTTP в заголовке "Authorization". Получение и возобновление действия маркеров выполняет библиотека проверки подлинности Майкрософт.

## <a name="nuget-packages"></a>Пакеты NuGet

Для работы с руководством необходимы следующие пакеты NuGet:

|Библиотека|Описание|
|---|---|
|[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client)| Библиотека проверки подлинности Майкрософт|
|[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph)|Клиентская библиотека Microsoft Graph|

## <a name="set-up-your-project"></a>Настройка проекта

В этом разделе даны пошаговые инструкции по интеграции классического приложения .NET для Windows (XAML) с платформой входа корпорации Майкрософт. После этого приложение сможет выполнять запросы к интерфейсам веб-API, которым требуется маркер, таким как API Microsoft Graph.

В этом руководстве показано, как создать приложение с кнопкой для запроса к API Graph и кнопкой выхода. В приложении также есть текстовые поля, содержащие результаты вызовов.

> [!Tip]
> Чтобы просмотреть полную версию проекта, созданного в этом руководстве, можете скачать его на сайте [GitHub](https://github.com/Azure-Samples/active-directory-dotnet-native-uwp-v2/archive/msal3x.zip).

### <a name="create-your-application"></a>Создание приложения

1. Откройте Visual Studio и выберите элемент **Создать проект**.
1. В окне **Создание проекта** выберите шаблон проекта **Пустое приложение (универсальное приложение Windows)** для C# и щелкните **Далее**.
1. В окне **Настроить новый проект** присвойте проекту имя и щелкните **Создать**.
1. Когда появится соответствующий запрос, в поле **Новый проект приложения для универсальной платформы Windows** выберите произвольные **целевую** и **минимальную** версии и щелкните **ОК**.

   ![Минимальная и целевая версии](./media/tutorial-v2-windows-uwp/select-uwp-target-minimum.png)

### <a name="add-the-microsoft-authentication-library-to-your-project"></a>Добавление библиотеки проверки подлинности Майкрософт в проект

1. В Visual Studio выберите **Сервис** > **Диспетчер пакетов NuGet** > **Консоль диспетчера пакетов**.
1. Скопируйте и вставьте следующую команду в окно **консоли диспетчера пакетов**:

    ```powershell
    Install-Package Microsoft.Identity.Client
    Install-Package Microsoft.Graph
    ```

   > [!NOTE]
   > Первая команда позволяет установить [библиотеку проверки подлинности Майкрософт (MSAL.NET)](https://aka.ms/msal-net). MSAL.NET получает, кэширует и обновляет маркеры пользователей для доступа к API-интерфейсам, которые защищены с помощью платформы удостоверений Майкрософт. Вторая позволяет установить [клиентскую библиотеку Microsoft Graph для .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet), чтобы обрабатывать проверку подлинности запросов к Microsoft Graph и вызовы этой службы.

### <a name="create-your-applications-ui"></a>Создание пользовательского интерфейса приложения

Visual Studio автоматически создает файл *MainPage.xaml* как часть шаблона проекта. Откройте этот файл и замените узел **Grid** для своего приложения следующим кодом:

```xml
<Grid>
    <StackPanel Background="Azure">
        <StackPanel Orientation="Horizontal" HorizontalAlignment="Right">
            <Button x:Name="CallGraphButton" Content="Call Microsoft Graph API" HorizontalAlignment="Right" Padding="5" Click="CallGraphButton_Click" Margin="5" FontFamily="Segoe Ui"/>
            <Button x:Name="SignOutButton" Content="Sign-Out" HorizontalAlignment="Right" Padding="5" Click="SignOutButton_Click" Margin="5" Visibility="Collapsed" FontFamily="Segoe Ui"/>
        </StackPanel>
        <TextBlock Text="API Call Results" Margin="2,0,0,-5" FontFamily="Segoe Ui" />
        <TextBox x:Name="ResultText" TextWrapping="Wrap" MinHeight="120" Margin="5" FontFamily="Segoe Ui"/>
        <TextBlock Text="Token Info" Margin="2,0,0,-5" FontFamily="Segoe Ui" />
        <TextBox x:Name="TokenInfoText" TextWrapping="Wrap" MinHeight="70" Margin="5" FontFamily="Segoe Ui"/>
    </StackPanel>
</Grid>
```

### <a name="use-the-microsoft-authentication-library-to-get-a-token-for-the-microsoft-graph-api"></a>Получение маркера для API Microsoft Graph с помощью библиотеки проверки подлинности Майкрософт

В этом разделе показано, как получить маркер для API Microsoft Graph с помощью библиотеки проверки подлинности Майкрософт. Внесите изменения в файл *MainPage.xaml.cs*.

1. Добавьте в *MainPage.xaml.cs* следующие ссылки.

    ```csharp
    using Microsoft.Identity.Client;
    using Microsoft.Graph;
    using System.Diagnostics;
    using System.Threading.Tasks;
    ```

1. Замените класс `MainPage` следующим кодом.

    ```csharp
    public sealed partial class MainPage : Page
    {

        //Set the scope for API call to user.read
        private string[] scopes = new string[] { "user.read" };

        // Below are the clientId (Application Id) of your app registration and the tenant information.
        // You have to replace:
        // - the content of ClientID with the Application Id for your app registration
        private const string ClientId = "[Application Id pasted from the application registration portal]";

        private const string Tenant = "common"; // Alternatively "[Enter your tenant, as obtained from the Azure portal, e.g. kko365.onmicrosoft.com]"
        private const string Authority = "https://login.microsoftonline.com/" + Tenant;

        // The MSAL Public client app
        private static IPublicClientApplication PublicClientApp;

        private static string MSGraphURL = "https://graph.microsoft.com/v1.0/";
        private static AuthenticationResult authResult;

        public MainPage()
        {
            this.InitializeComponent();
        }

        /// <summary>
        /// Call AcquireTokenAsync - to acquire a token requiring user to sign in
        /// </summary>
        private async void CallGraphButton_Click(object sender, RoutedEventArgs e)
        {
            try
            {
                // Sign in user using MSAL and obtain an access token for Microsoft Graph
                GraphServiceClient graphClient = await SignInAndInitializeGraphServiceClient(scopes);

                // Call the /me endpoint of Graph
                User graphUser = await graphClient.Me.Request().GetAsync();

                // Go back to the UI thread to make changes to the UI
                await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, () =>
                {
                    ResultText.Text = "Display Name: " + graphUser.DisplayName + "\nBusiness Phone: " + graphUser.BusinessPhones.FirstOrDefault()
                                      + "\nGiven Name: " + graphUser.GivenName + "\nid: " + graphUser.Id
                                      + "\nUser Principal Name: " + graphUser.UserPrincipalName;
                    DisplayBasicTokenInfo(authResult);
                    this.SignOutButton.Visibility = Visibility.Visible;
                });
            }
            catch (MsalException msalEx)
            {
                await DisplayMessageAsync($"Error Acquiring Token:{System.Environment.NewLine}{msalEx}");
            }
            catch (Exception ex)
            {
                await DisplayMessageAsync($"Error Acquiring Token Silently:{System.Environment.NewLine}{ex}");
                return;
            }
        }
                /// <summary>
        /// Signs in the user and obtains an access token for Microsoft Graph
        /// </summary>
        /// <param name="scopes"></param>
        /// <returns> Access Token</returns>
        private static async Task<string> SignInUserAndGetTokenUsingMSAL(string[] scopes)
        {
            // Initialize the MSAL library by building a public client application
            PublicClientApp = PublicClientApplicationBuilder.Create(ClientId)
                .WithAuthority(Authority)
                .WithUseCorporateNetwork(false)
                .WithRedirectUri("https://login.microsoftonline.com/common/oauth2/nativeclient")
                 .WithLogging((level, message, containsPii) =>
                 {
                     Debug.WriteLine($"MSAL: {level} {message} ");
                 }, LogLevel.Warning, enablePiiLogging: false, enableDefaultPlatformLogging: true)
                .Build();

            // It's good practice to not do work on the UI thread, so use ConfigureAwait(false) whenever possible.
            IEnumerable<IAccount> accounts = await PublicClientApp.GetAccountsAsync().ConfigureAwait(false);
            IAccount firstAccount = accounts.FirstOrDefault();

            try
            {
                authResult = await PublicClientApp.AcquireTokenSilent(scopes, firstAccount)
                                                  .ExecuteAsync();
            }
            catch (MsalUiRequiredException ex)
            {
                // A MsalUiRequiredException happened on AcquireTokenSilentAsync. This indicates you need to call AcquireTokenAsync to acquire a token
                Debug.WriteLine($"MsalUiRequiredException: {ex.Message}");

                authResult = await PublicClientApp.AcquireTokenInteractive(scopes)
                                                  .ExecuteAsync()
                                                  .ConfigureAwait(false);

            }
            return authResult.AccessToken;
        }
    }
    ```

#### <a name="get-a-user-token-interactively"></a>Интерактивное получение маркера<a name="more-information"></a>

Метод `AcquireTokenInteractive` отображает окно, в котором пользователю предлагается выполнить вход. Когда пользователь впервые запрашивает доступ к защищенному ресурсу, приложение обычно требует выполнить интерактивный вход. Пользователям также может потребоваться выполнить вход при сбое автоматической операции получения маркера, например, если истек срок действия пароля пользователя.

#### <a name="get-a-user-token-silently"></a>Автоматическое получение маркера пользователя

Метод `AcquireTokenSilent` обрабатывает получение и обновление маркера без участия пользователя. После первого запуска `AcquireTokenInteractive` и запроса учетных данных пользователя выполните метод `AcquireTokenSilent`, чтобы запросить маркеры для последующих вызовов. Этот метод принимает маркеры без дополнительного взаимодействия. Кэширование и возобновление действия маркеров выполняет библиотека проверки подлинности Майкрософт.

Иногда метод `AcquireTokenSilent` завершается ошибкой. Например, если пользователь вышел из системы или изменил пароль на другом устройстве. Когда библиотека проверки подлинности Майкрософт обнаруживает, что для решения проблемы требуется интерактивное действие, она создает исключение `MsalUiRequiredException`. Приложение может обработать это исключение двумя способами:

* Приложение немедленно вызывает `AcquireTokenInteractive`. Этот вызов приводит к появлению запроса на вход пользователя. Этот подход обычно используется в интернет-приложениях, где пользователю не предоставляется автономное содержимое. Пример, созданный в ходе пошаговой настройки, следует этому шаблону. Вы увидите его в действии при первом запуске примера.

   Так как приложение еще не использовалось, `accounts.FirstOrDefault()` содержит значение NULL, поэтому создается исключение `MsalUiRequiredException`.

   Затем код в примере обрабатывает исключение, вызывая `AcquireTokenInteractive`. Этот вызов приводит к появлению запроса на вход пользователя.

* Приложение демонстрирует пользователям визуальное оповещение о том, что требуется вход. После этого пользователи могут выбрать нужное время для входа. Приложение может повторить вызов `AcquireTokenSilent` позже. Такой подход применяется, когда пользователи без проблем могут использовать другие функциональные возможности приложения. Например, когда в приложении доступно автономное содержимое. В этом случае пользователи могут самостоятельно решить, когда они хотят войти в систему. Приложение может повторно вызвать `AcquireTokenSilent` после временной недоступности сетевого подключения.

### <a name="instantiate-the-microsoft-graph-service-client-by-obtaining-the-token-from-the-signinuserandgettokenusingmsal-method"></a>Создание экземпляра клиента службы Microsoft Graph путем получения маркера с помощью метода SignInUserAndGetTokenUsingMSAL

Добавьте приведенный ниже новый метод в файл *MainPage.xaml.cs*.

```csharp
      /// <summary>
     /// Sign in user using MSAL and obtain a token for Microsoft Graph
     /// </summary>
     /// <returns>GraphServiceClient</returns>
     private async static Task<GraphServiceClient> SignInAndInitializeGraphServiceClient(string[] scopes)
     {
         GraphServiceClient graphClient = new GraphServiceClient(MSGraphURL,
             new DelegateAuthenticationProvider(async (requestMessage) =>
             {
                 requestMessage.Headers.Authorization = new AuthenticationHeaderValue("bearer", await SignInUserAndGetTokenUsingMSAL(scopes));
             }));

         return await Task.FromResult(graphClient);
     }
```

#### <a name="more-information-on-making-a-rest-call-against-a-protected-api"></a>Дополнительные сведения о вызове REST через защищенный API

В этом примере приложения с помощью метода `GetGraphServiceClient` создается экземпляр `GraphServiceClient` с применением маркера доступа. Затем используется `GraphServiceClient` для получения сведений о профиле пользователя из конечной точки **me**.

### <a name="add-a-method-to-sign-out-the-user"></a>Добавление метода для выхода пользователя

Для выхода пользователя добавьте следующий метод в файл *MainPage.xaml.cs*:

```csharp
/// <summary>
/// Sign out the current user
/// </summary>
private async void SignOutButton_Click(object sender, RoutedEventArgs e)
{
    IEnumerable<IAccount> accounts = await PublicClientApp.GetAccountsAsync().ConfigureAwait(false);
    IAccount firstAccount = accounts.FirstOrDefault();

    try
    {
        await PublicClientApp.RemoveAsync(firstAccount).ConfigureAwait(false);
        await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, () =>
        {
            ResultText.Text = "User has signed out";
            this.CallGraphButton.Visibility = Visibility.Visible;
                this.SignOutButton.Visibility = Visibility.Collapsed;
            });
        }
        catch (MsalException ex)
        {
            ResultText.Text = $"Error signing out user: {ex.Message}";
        }
    }
```

MSAL.NET использует асинхронные методы для получения маркеров или управления учетными записями. Поэтому нужно поддерживать действия пользовательского интерфейса в потоке пользовательского интерфейса. Именно поэтому нужно вызвать `Dispatcher.RunAsync` и соблюдать меры предосторожности при вызове `ConfigureAwait(false)`.

#### <a name="more-information-about-signing-out"></a>Дополнительные сведения о выходе<a name="more-information-on-sign-out"></a>

Метод `SignOutButton_Click` позволяет удалить пользователя из кэша пользователей библиотеки проверки подлинности Майкрософт. Этот метод фактически информирует библиотеку проверки подлинности Майкрософт, что сведения о текущем пользователе не нужно сохранять. Последующий запрос получения маркера завершается успешно только в том случае, если он выполняется интерактивно.

Приложение в этом примере поддерживает одиночного пользователя. Библиотека проверки подлинности Майкрософт поддерживает сценарии одновременного входа пользователя с несколькими учетными записями. Пример — приложение электронной почты, в котором у пользователя есть несколько учетных записей.

### <a name="display-basic-token-information"></a>Отображение основных сведений о маркере

Добавьте следующий метод в файл *MainPage.xaml.cs* для отображения основных сведений о маркере:

```csharp
/// <summary>
/// Display basic information contained in the token. Needs to be called from the UI thread.
/// </summary>
private void DisplayBasicTokenInfo(AuthenticationResult authResult)
{
    TokenInfoText.Text = "";
    if (authResult != null)
    {
        TokenInfoText.Text += $"User Name: {authResult.Account.Username}" + Environment.NewLine;
        TokenInfoText.Text += $"Token Expires: {authResult.ExpiresOn.ToLocalTime()}" + Environment.NewLine;
    }
}
```

#### <a name="more-information"></a>Дополнительные сведения<a name="more-information-1"></a>

Маркеры идентификаторов, полученные с использованием **OpenID Connect**, также содержат небольшой набор данных, относящихся к пользователю. `DisplayBasicTokenInfo` отображает базовые сведения, содержащиеся в маркере, Эти сведения включают отображаемое имя и идентификатор пользователя. В них также есть сведения о сроке действия маркера и строка, представляющая сам маркер доступа. Если вы несколько раз нажмете кнопку **Call Microsoft Graph API** (Вызвать API Microsoft Graph), то увидите, что для последующих запросов повторно используется тот же маркер. Также можно заметить, что библиотека проверки подлинности Майкрософт продлила срок действия маркера.

### <a name="display-message"></a>Отображение сообщения

Добавьте приведенный ниже новый метод в файл *MainPage.xaml.cs*.

```csharp
/// <summary>
/// Displays a message in the ResultText. Can be called from any thread.
/// </summary>
private async Task DisplayMessageAsync(string message)
{
     await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal,
         () =>
         {
             ResultText.Text = message;
         });
     }
```

## <a name="register-your-application"></a>Регистрация приложения

Теперь зарегистрируйте свое приложение:

1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>.
1. Если у вас есть доступ к нескольким клиентам, в верхнем меню используйте фильтр **Каталог и подписка** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false":::, чтобы выбрать клиент, в котором следует зарегистрировать приложение.
1. Найдите и выберите **Azure Active Directory**.
1. В разделе **Управление** выберите **Регистрация приложений** > **Создать регистрацию**.
1. Введите **имя** приложения, например `UWP-App-calling-MSGraph`. Пользователи приложения могут видеть это имя. Вы можете изменить его позже.
1. В поле **Поддерживаемые типы учетных записей** выберите вариант **Учетные записи в любом каталоге организации (любой каталог Azure AD — мультитенантный) и персональные учетные записи Майкрософт (например, Skype, Xbox)** . 
1. Выберите **Зарегистрировать**.
1. На странице общих сведений найдите **идентификатор приложения (клиента)** и скопируйте его. Вернитесь в Visual Studio, откройте файл *MainPage.xaml.cs* и замените значение `ClientId` полученным значением.

Настройте проверку подлинности для приложения.

1. Вернитесь на <a href="https://portal.azure.com/" target="_blank">портал Azure</a> и в разделе **Управление** щелкните **Проверка подлинности** > **Добавить платформу**, а затем выберите **Мобильные и классические приложения**.
1. В разделе **URI перенаправления** введите `https://login.microsoftonline.com/common/oauth2/nativeclient`.
1. Нажмите кнопку **Настроить**.

Настройте разрешения API для приложения.

1. В разделе **Управление** выберите **Разрешения API** > **Добавить разрешение**.
1. Выберите **Microsoft Graph**.
1. Выберите **Делегированные разрешения**, выполните поиск строки *User.Read* и убедитесь, что установлен параметр **User.Read**.
1. Если вы внесли изменения, щелкните **Добавить разрешения**, чтобы сохранить эти изменения.

## <a name="enable-integrated-authentication-on-federated-domains-optional"></a>Включение встроенной проверки подлинности в федеративных доменах (необязательно)

Чтобы включить Встроенную проверку подлинности Windows при работе с федеративным доменом Azure AD, необходимо включить дополнительные возможности в манифесте приложения. Вернитесь к приложению, открытому в Visual Studio.

1. Откройте файл *Package.appxmanifest*.
1. Выберите элемент **Возможности** и включите следующие параметры:

   * **Корпоративная проверка подлинности**
   * **Частные сети (клиент и сервер)**
   * **Общие сертификаты пользователей**

> [!IMPORTANT]
> [Встроенная проверка подлинности Windows](https://aka.ms/msal-net-iwa) не настроена по умолчанию для этого примера. Приложениям, которые запрашивают возможности `Enterprise Authentication` или `Shared User Certificates`, требуется более высокий уровень проверки в Windows Store. Кроме того, не все разработчики желают выполнять проверку более высокого уровня. Этот параметр следует включать, только если Встроенная проверка подлинности Windows должна работать с федеративным доменом Azure AD.

## <a name="alternate-approach-to-using-withdefaultredirecturi"></a>Другой подход к использованию WithDefaultRedirectURI()

В нашем примере используется метод `WithRedirectUri("https://login.microsoftonline.com/common/oauth2/nativeclient")`. Чтобы применить `WithDefaultRedirectURI()`, выполните следующие действия:

1. В *MainPage.XAML.cs* замените `WithRedirectUri` на `WithDefaultRedirectUri`.

   **Прежний код**

   ```csharp

   PublicClientApp = PublicClientApplicationBuilder.Create(ClientId)
       .WithAuthority(Authority)
       .WithUseCorporateNetwork(false)
       .WithRedirectUri("https://login.microsoftonline.com/common/oauth2/nativeclient")
       .WithLogging((level, message, containsPii) =>
        {
            Debug.WriteLine($"MSAL: {level} {message} ");
        }, LogLevel.Warning, enablePiiLogging: false, enableDefaultPlatformLogging: true)
       .Build();

   ```

   **Обновленный код**

   ```csharp

   PublicClientApp = PublicClientApplicationBuilder.Create(ClientId)
       .WithAuthority("https://login.microsoftonline.com/common")
       .WithUseCorporateNetwork(false)
       .WithDefaultRedirectUri()
       .WithLogging((level, message, containsPii) =>
        {
            Debug.WriteLine($"MSAL: {level} {message} ");
        }, LogLevel.Warning, enablePiiLogging: false, enableDefaultPlatformLogging: true)
       .Build();
   ```

2.  Найдите правильный URI обратного вызова для приложения, добавив поле `redirectURI` в файл *MainPage.xaml.cs* и установив на нем точку останова:

    ```csharp

    public sealed partial class MainPage : Page
    {
            ...

            string redirectURI = Windows.Security.Authentication.Web.WebAuthenticationBroker
                                .GetCurrentApplicationCallbackUri().ToString();
            public MainPage()
            {
                ...
            }
           ...
    }

    ```

    Запустите это приложение и скопируйте значение `redirectUri`, когда сработает точка останова. Это значение должно выглядеть примерно так: `ms-app://s-1-15-2-1352796503-54529114-405753024-3540103335-3203256200-511895534-1429095407/`

    После этого добавленную строку кода можно удалить, так как она нужна только для получения этого значения.

3. На портале регистрации приложений добавьте полученное значение в поле **RedirectUri** на панели **Проверка подлинности**.

## <a name="test-your-code"></a>Тестирование кода

Чтобы протестировать приложение, запустите проект в Visual Studio с помощью клавиши **F5**. Откроется главное окно:

![Пользовательский интерфейс приложения](./media/tutorial-v2-windows-uwp/testapp-ui-vs2019.png)

Когда будете готовы выполнить тестирование, выберите **Call Microsoft Graph API** (вызвать API Graph Microsoft). Затем используйте рабочую или учебную учетную запись Azure AD либо учетную запись Майкрософт (live.com, outlook.com) для входа. При первом запуске теста приложение отображает окно, где пользователю предлагается войти в систему.

### <a name="consent"></a>Согласие на предоставление разрешений

При первом входе в приложение появится экран предоставления согласия, как на представленном ниже изображении. Выберите **Да**, чтобы дать явное согласие на доступ:

![Экран согласия на доступ](./media/tutorial-v2-windows-uwp/consentscreen-vs2019.png)

### <a name="expected-results"></a>Ожидаемые результаты

На экране **API Call Results** (Результаты вызова API) должны отобразиться сведения о профиле пользователя, возвращенные вызовом API Microsoft Graph:

![Экран результатов вызова API](./media/tutorial-v2-windows-uwp/uwp-results-screen-vs2019.png)

Вы также увидите основные сведения о маркере, полученные с помощью `AcquireTokenInteractive` или `AcquireTokenSilent`, в окне **Token Info** (Сведения о маркере):

|Свойство  |Формат  |Описание |
|---------|---------|---------|
|`Username` |`user@domain.com` |Имя пользователя, которое позволяет его идентифицировать.|
|`Token Expires` |`DateTime` |Это время, когда истекает срок действия маркера. Библиотека проверки подлинности Майкрософт продлевает срок действия, возобновляя действие маркера по мере необходимости.|

### <a name="more-information-about-scopes-and-delegated-permissions"></a>Дополнительные сведения об областях и делегированных разрешениях

Для чтения профиля пользователя API Microsoft Graph требуется область `user.read`. По умолчанию эта область добавляется в каждое приложение, зарегистрированное на портале регистрации приложений. Для других API Microsoft Graph, а также для пользовательских API вашего внутреннего сервера могут потребоваться дополнительные области. Например, API Microsoft Graph нужна область `Calendars.Read` для отображения календарей пользователя.

Чтобы из контекста приложения получать доступ к календарям пользователя, добавьте делегированное разрешение `Calendars.Read` в сведения о регистрации приложения. Затем добавьте область `Calendars.Read` в вызов `acquireTokenSilent`.

При увеличении количества областей от пользователя могут потребоваться дополнительные согласия.

## <a name="known-issues"></a>Известные проблемы

### <a name="issue-1"></a>Проблема 1

При входе в приложение в федеративном домене Azure AD появляется одно из следующих сообщений об ошибке:

* "No valid client certificate found in the request" (В запросе отсутствует действительный клиентский сертификат).
* "No valid certificates found in the user's certificate store" (В хранилище сертификатов пользователя отсутствуют действительные сертификаты).
* "Try again choosing a different authentication method" (Попробуйте еще раз, выбрав другой метод проверки подлинности).

**Причина.** Не включены возможности корпоративного уровня и сертификатов.

**Решение**. Выполните инструкции из раздела [Включение встроенной проверки подлинности в федеративных доменах (необязательно)](#enable-integrated-authentication-on-federated-domains-optional).

### <a name="issue-2"></a>Проблема 2

Вы включили [встроенную проверку подлинности в федеративных доменах](#enable-integrated-authentication-on-federated-domains-optional) и пытаетесь использовать Windows Hello на компьютере под управлением Windows 10 для входа в среду с настроенной многофакторной проверкой подлинности. Появится список сертификатов. Если вы решили использовать собственный ПИН-код, окно ПИН-кода не отображается.

**Причина.** Эта проблема вызвана известным ограничением веб-брокера аутентификации в приложениях UWP, запускаемых на рабочем столе Windows 10. В Windows 10 Mobile он работает нормально.

**Решение:** Выберите **Sign in with other options** (Вход с другими параметрами). Затем выберите **Sign in with a username and password** (Вход с использованием имени пользователя и пароля). Выберите **Provide your password** (Ввести пароль). После этого пройдите процесс проверки подлинности по телефону.

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

Подробнее об использовании библиотеки аутентификации Майкрософт (MSAL) для авторизации и аутентификации в приложениях .NET:

> [!div class="nextstepaction"]
> [Обзор библиотеки проверки подлинности Майкрософт (MSAL)](msal-overview.md)
