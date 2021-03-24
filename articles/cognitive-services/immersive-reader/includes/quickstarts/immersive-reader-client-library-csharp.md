---
title: 'Клиентская библиотека C# иммерсивного средства чтения: краткое руководство'
titleSuffix: Azure Cognitive Services
description: В рамках этого краткого руководства вы создадите веб-приложения с нуля и добавите функции API "Иммерсивное средство чтения".
services: cognitive-services
author: nitinme
manager: nitinme
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: include
ms.date: 09/14/2020
ms.author: nitinme
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 442f51759b136200fd61706864568e580265e572
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102619261"
---
[Иммерсивное средство чтения](https://www.onenote.com/learningtools) — это инклюзивное решение, в котором реализованы проверенные методы, улучшающие понимание текста при чтении у людей, которые учатся читать или изучают язык, а также у людей, которые испытывают определенные трудности при обучении, например, из-за дислексии. Вы можете использовать иммерсивное средство чтения в своих приложениях, чтобы изолировать текст для фокусировки, отображать рисунки, связанные с часто используемыми словами, выделять части речи, читать вслух выделенный текст, переводить слова и фразы в реальном времени и многое другое.

В этом кратком руководстве показано, как создать веб-приложение с нуля и интегрировать иммерсивное средство чтения с помощью соответствующей клиентской библиотеки. Полностью рабочий пример этого краткого руководства доступен [на GitHub](https://github.com/microsoft/immersive-reader-sdk/tree/master/js/samples/quickstart-csharp).

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services).
* [Visual Studio 2019](https://visualstudio.microsoft.com/downloads)
* Ресурс "Иммерсивное средство чтения", настроенный для проверки подлинности Azure Active Directory. Инструкции по настройке см. [здесь](../../how-to-create-immersive-reader.md). Некоторые из этих созданных значений потребуются вам при настройке свойств примера проекта. Сохраните результаты своего сеанса в текстовом файле для использования в будущем.

## <a name="create-a-web-app-project"></a>Создание проекта веб-приложения

В Visual Studio создайте проект с использованием шаблона веб-приложений ASP.NET Core со встроенным MVC и ASP.NET Core 2.1. Назовите проект "QuickstartSampleWebApp".

![Новый проект — C#](../../media/quickstart-csharp/1-createproject.png)

![Настройка нового проекта — C#](../../media/quickstart-csharp/2-configureproject.png)

![Новое веб-приложение ASP.NET Core — C#](../../media/quickstart-csharp/3-createmvc.png)

## <a name="set-up-authentication"></a>Настройка проверки подлинности

### <a name="configure-authentication-values"></a>Настройка значений проверки подлинности

Щелкните правой кнопкой мыши проект в _обозревателе решений_, а затем выберите **Управление секретами пользователей**. Откроется файл с именем _secrets.json_. Этот файл не возвращен в систему управления версиями. Дополнительные сведения см. [здесь](/aspnet/core/security/app-secrets?tabs=windows). Замените содержимое _secrets.json_ следующим параметром, указав значения, заданные при создании ресурса "Иммерсивное средство чтения".

```json
{
  "TenantId": "YOUR_TENANT_ID",
  "ClientId": "YOUR_CLIENT_ID",
  "ClientSecret": "YOUR_CLIENT_SECRET",
  "Subdomain": "YOUR_SUBDOMAIN"
}
```

### <a name="install-active-directory-nuget-package"></a>Установка пакета NuGet Active Directory

В приведенном далее коде используется пакет NuGet **Microsoft.IdentityModel.Clients.ActiveDirectory**, поэтому необходимо добавить в проект ссылку на этот пакет.

Откройте консоль диспетчера пакетов NuGet, выбрав **Средства -> Диспетчер пакетов NuGet -> Консоль диспетчера пакетов**, и запустите приведенную ниже команду.

```powershell
    Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -Version 5.2.0
```

### <a name="update-the-controller-to-acquire-the-token"></a>Обновление контроллера для получения токена 

Откройте _Controllers\HomeController.cs_, а затем добавьте следующий код после операторов _using_ в верхней части файла.

```csharp
using Microsoft.IdentityModel.Clients.ActiveDirectory;
```

Теперь настроим контроллер для получения значений Azure AD из _secrets.json_. В верхней части класса _HomeController_ добавьте следующий код после ```public class HomeController : Controller {```.

```csharp
private readonly string TenantId;     // Azure subscription TenantId
private readonly string ClientId;     // Azure AD ApplicationId
private readonly string ClientSecret; // Azure AD Application Service Principal password
private readonly string Subdomain;    // Immersive Reader resource subdomain (resource 'Name' if the resource was created in the Azure portal, or 'CustomSubDomain' option if the resource was created with Azure CLI Powershell. Check the Azure portal for the subdomain on the Endpoint in the resource Overview page, for example, 'https://[SUBDOMAIN].cognitiveservices.azure.com/')

public HomeController(Microsoft.Extensions.Configuration.IConfiguration configuration)
{
    TenantId = configuration["TenantId"];
    ClientId = configuration["ClientId"];
    ClientSecret = configuration["ClientSecret"];
    Subdomain = configuration["Subdomain"];

    if (string.IsNullOrWhiteSpace(TenantId))
    {
        throw new ArgumentNullException("TenantId is null! Did you add that info to secrets.json?");
    }

    if (string.IsNullOrWhiteSpace(ClientId))
    {
        throw new ArgumentNullException("ClientId is null! Did you add that info to secrets.json?");
    }

    if (string.IsNullOrWhiteSpace(ClientSecret))
    {
        throw new ArgumentNullException("ClientSecret is null! Did you add that info to secrets.json?");
    }

    if (string.IsNullOrWhiteSpace(Subdomain))
    {
        throw new ArgumentNullException("Subdomain is null! Did you add that info to secrets.json?");
    }
}

/// <summary>
/// Get an Azure AD authentication token
/// </summary>
private async Task<string> GetTokenAsync()
{
    string authority = $"https://login.windows.net/{TenantId}";
    const string resource = "https://cognitiveservices.azure.com/";

    AuthenticationContext authContext = new AuthenticationContext(authority);
    ClientCredential clientCredential = new ClientCredential(ClientId, ClientSecret);

    AuthenticationResult authResult = await authContext.AcquireTokenAsync(resource, clientCredential);

    return authResult.AccessToken;
}

[HttpGet]
public async Task<JsonResult> GetTokenAndSubdomain()
{
    try
    {
        string tokenResult = await GetTokenAsync();

        return new JsonResult(new { token = tokenResult, subdomain = Subdomain });
    }
    catch (Exception e)
    {
        string message = "Unable to acquire Azure AD token. Check the debugger for more information.";
        Debug.WriteLine(message, e);
        return new JsonResult(new { error = message });
    }
}
```

## <a name="add-sample-content"></a>Добавление примеров содержимого
Сначала откройте _Views\Shared\Layout.cshtml_. Перед строкой ```</head>```добавьте следующие строки:

```html
@RenderSection("Styles", required: false)
```

Теперь мы добавим пример содержимого для этого веб-приложения. Откройте файл _Views\Home\Index.cshtml_ и замените автоматически созданный код приведенным ниже.

```html
@{
    ViewData["Title"] = "Immersive Reader C# Quickstart";
}

@section Styles {
    <style type="text/css">
        .immersive-reader-button {
            background-color: white;
            margin-top: 5px;
            border: 1px solid black;
            float: right;
        }
    </style>
}

<div class="container">
    <button class="immersive-reader-button" data-button-style="iconAndText" data-locale="en"></button>

    <h1 id="ir-title">About Immersive Reader</h1>
    <div id="ir-content" lang="en-us">
        <p>
            Immersive Reader is a tool that implements proven techniques to improve reading comprehension for emerging readers, language learners, and people with learning differences.
            The Immersive Reader is designed to make reading more accessible for everyone. The Immersive Reader
            <ul>
                <li>
                    Shows content in a minimal reading view
                </li>
                <li>
                    Displays pictures of commonly used words
                </li>
                <li>
                    Highlights nouns, verbs, adjectives, and adverbs
                </li>
                <li>
                    Reads your content out loud to you
                </li>
                <li>
                    Translates your content into another language
                </li>
                <li>
                    Breaks down words into syllables
                </li>
            </ul>
        </p>
        <h3>
            The Immersive Reader is available in many languages.
        </h3>
        <p lang="es-es">
            El Lector inmersivo está disponible en varios idiomas.
        </p>
        <p lang="zh-cn">
            沉浸式阅读器支持许多语言
        </p>
        <p lang="de-de">
            Der plastische Reader ist in vielen Sprachen verfügbar.
        </p>
        <p lang="ar-eg" dir="rtl" style="text-align:right">
            يتوفر \"القارئ الشامل\" في العديد من اللغات.
        </p>
    </div>
</div>
```

Обратите внимание, что весь текст имеет атрибут **lang**, который описывает языки текста. Этот атрибут помогает Иммерсивному средству чтения предоставить соответствующие функции языка и грамматики.

## <a name="add-javascript-to-handle-launching-immersive-reader"></a>Добавление JavaScript для обработки запуска иммерсивного средства чтения

Иммерсивное средство чтения предоставляет такие функциональные возможности, как запуск Иммерсивного средства чтения и рендеринг кнопок Иммерсивного средства чтения. Дополнительные сведения см. [здесь](../../reference.md).

В нижней части _Views\Home\Index.cshtml_ добавьте следующий код:

```html
@section Scripts
{
    <script src="https://contentstorage.onenote.office.net/onenoteltir/immersivereadersdk/immersive-reader-sdk.1.0.0.js"></script>
    <script>
        function getTokenAndSubdomainAsync() {
            return new Promise(function (resolve, reject) {
                $.ajax({
                    url: "@Url.Action("GetTokenAndSubdomain", "Home")",
                    type: "GET",
                    success: function (data) {
                        if (data.error) {
                            reject(data.error);
                        } else {
                            resolve(data);
                        }
                    },
                    error: function (err) {
                        reject(err);
                    }
                });
            });
        }
    
        $(".immersive-reader-button").click(function () {
            handleLaunchImmersiveReader();
        });
    
        function handleLaunchImmersiveReader() {
            getTokenAndSubdomainAsync()
                .then(function (response) {
                    const token = response["token"];
                    const subdomain = response["subdomain"];
    
                    // Learn more about chunk usage and supported MIME types https://docs.microsoft.com/en-us/azure/cognitive-services/immersive-reader/reference#chunk
                    const data = {
                        title: $("#ir-title").text(),
                        chunks: [{
                            content: $("#ir-content").html(),
                            mimeType: "text/html"
                        }]
                    };
    
                    // Learn more about options https://docs.microsoft.com/en-us/azure/cognitive-services/immersive-reader/reference#options
                    const options = {
                        "onExit": exitCallback,
                        "uiZIndex": 2000
                    };
    
                    ImmersiveReader.launchAsync(token, subdomain, data, options)
                        .catch(function (error) {
                            alert("Error in launching the Immersive Reader. Check the console.");
                            console.log(error);
                        });
                })
                .catch(function (error) {
                    alert("Error in getting the Immersive Reader token and subdomain. Check the console.");
                    console.log(error);
                });
        }
    
        function exitCallback() {
            console.log("This is the callback function. It is executed when the Immersive Reader closes.");
        }
    </script>
}
```

## <a name="build-and-run-the-app"></a>Создание и запуск приложения

В строке меню выберите **Отладка > Начать отладку** или нажмите клавишу **F5**, чтобы запустить приложение.

В браузере вы уведите:

![Пример приложения — C#](../../media/quickstart-csharp/4-buildapp.png)

## <a name="launch-the-immersive-reader"></a>Запуск Иммерсивного средства чтения

При нажатии кнопки "иммерсивное средство чтения" появится запущенное иммерсивное средство чтения с содержимым на странице.

![Иммерсивное средство чтения — C#](../../media/quickstart-csharp/5-viewimmersivereader.png)

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Создание ресурса и настройка AAD](../../how-to-create-immersive-reader.md)