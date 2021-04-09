---
title: Подписывание HTTP-запроса с помощью C#
description: В этом учебнике описана версия подписания HTTP-запроса на языке C# с подписью HMAC для Служб коммуникации Azure.
author: alexandra142
manager: soricos
services: azure-communication-services
ms.author: apistrak
ms.date: 03/10/2021
ms.topic: include
ms.service: azure-communication-services
ms.openlocfilehash: 34c7df2b0e61536c0b5f0bc1e4a97d58d0d9c6a4
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103490520"
---
## <a name="prerequisites"></a>Предварительные требования

Перед началом работы нужно сделать следующее:

- Создайте учетную запись Azure с активной подпиской. Дополнительные сведения см. на странице [Создайте бесплатную учетную запись Azure уже сегодня](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Установить [Visual Studio](https://visualstudio.microsoft.com/downloads/).
- Создайте ресурс Служб коммуникации Azure. Дополнительные сведения см. в [кратком руководстве по созданию ресурсов Служб коммуникации и управлению ими](../../quickstarts/create-communication-resource.md). Для работы с данным учебником необходимо записать **resourceEndpoint** и **resourceAccessKey**.

## <a name="sign-an-http-request-with-c"></a>Подписывание HTTP-запроса с помощью C#

Проверка подлинности на основе ключа доступа использует для создания подписей HMAC для всех HTTP-запросов общий секретный ключ. Эта сигнатура создается с использованием алгоритма SHA256 и отправляется в заголовок `Authorization` с помощью схемы `HMAC-SHA256`. Например:

```
Authorization: "HMAC-SHA256 SignedHeaders=date;host;x-ms-content-sha256&Signature=<hmac-sha256-signature>"
```

В состав `hmac-sha256-signature` входит следующее:

- HTTP-команда (например, `GET` или `PUT`)
- Путь HTTP-запроса
- Date
- Узел
- x-ms-content-sha256

## <a name="setup"></a>Настройка

Ниже описано, как создать заголовок авторизации.

### <a name="create-a-new-c-application"></a>Создание нового приложения C#

В окне консоли (cmd, PowerShell или Bash) выполните команду `dotnet new`, чтобы создать консольное приложение с именем `SignHmacTutorial`. Эта команда создает простой проект Hello World на языке C# с одним файлом исходного кода: **Program.cs**.

```console
dotnet new console -o SignHmacTutorial
```

Измените каталог на созданную папку приложения. Используйте команду`dotnet build` для компиляции приложения.

```console
cd SignHmacTutorial
dotnet build
```

## <a name="install-the-package"></a>Установка пакета

Установите пакет `Newtonsoft.Json`, который используется для сериализации текстовой части.

```console
dotnet add package Newtonsoft.Json
```

Обновите объявление метода `Main`, чтобы он поддерживал асинхронный код. Используйте следующий код.

```csharp
using System;
using System.Globalization;
using System.Net.Http;
using System.Security.Cryptography;
using System.Text;
using System.Threading.Tasks;
using Newtonsoft.Json;
namespace SignHmacTutorial
{
    class Program
    {
        static async Task Main(string[] args)
        {
            Console.WriteLine("Azure Communication Services - Sign an HTTP request Tutorial");
            // Tutorial code goes here.
        }
    }
}

```

## <a name="create-a-request-message"></a>Создание сообщения запроса

В этом примере мы подпишем запрос на создание нового удостоверения с помощью API проверки подлинности Служб коммуникации Azure (версии `2021-03-07`).

Добавьте в метод `Main` следующий код.

```csharp
string resourceEndpoint = "resourceEndpoint";
//Create a uri you are going to call.
var requestUri = new Uri($"{resourceEndpoint}/identities?api-version=2021-03-07");
//Endpoint identities?api-version=2021-03-07 accepts list of scopes as a body
var body = new[] { "chat" }; 
var serializedBody = JsonConvert.SerializeObject(body);
var requestMessage = new HttpRequestMessage(HttpMethod.Post, requestUri)
{
    Content = new StringContent(serializedBody, Encoding.UTF8)
};
```

Замените `resourceEndpoint` фактическим значением конечной точки ресурса.

## <a name="create-a-content-hash"></a>Создание хэша содержимого

Хэш содержимого является частью сигнатуры HMAC. Используйте следующий код, чтобы вычислить хэш содержимого. Этот метод можно добавить в `Progam.cs`, используя метод `Main`.

```csharp
static string ComputeContentHash(string content)
{
    using (var sha256 = SHA256.Create())
    {
        byte[] hashedBytes = sha256.ComputeHash(Encoding.UTF8.GetBytes(content));
        return Convert.ToBase64String(hashedBytes);
    }
}
```

## <a name="compute-a-signature"></a>Вычисление сигнатуры

Используйте следующий код, чтобы создать метод для вычисления сигнатуры HMAC.

```csharp
 static string ComputeSignature(string stringToSign)
{
    string secret = "resourceAccessKey";
    using (var hmacsha256 = new HMACSHA256(Convert.FromBase64String(secret)))
    {
        var bytes = Encoding.ASCII.GetBytes(stringToSign);
        var hashedBytes = hmacsha256.ComputeHash(bytes);
        return Convert.ToBase64String(hashedBytes);
    }
}
```

Замените `resourceAccessKey` ключом доступа к реальному ресурсу Служб коммуникации.

## <a name="create-an-authorization-header-string"></a>Создание строки заголовка авторизации

Далее мы создадим строку, которая будет добавлена в заголовок авторизации.

1. Вычислите хэш содержимого.
1. Укажите метку времени в формате UTC.
1. Подготовьте строку к подписыванию.
1. Вычислите сигнатуру.
1. Объедините строку, которая будет использована в заголовке авторизации.
 
Добавьте в метод `Main` следующий код.

```csharp
// Compute a content hash.
var contentHash = ComputeContentHash(serializedBody);
//Specify the Coordinated Universal Time (UTC) timestamp.
var date = DateTimeOffset.UtcNow.ToString("r", CultureInfo.InvariantCulture);
//Prepare a string to sign.
var stringToSign = $"POST\n{requestUri.PathAndQuery}\n{date};{requestUri.Authority};{contentHash}";
//Compute the signature.
var signature = ComputeSignature(stringToSign);
//Concatenate the string, which will be used in the authorization header.
var authorizationHeader = $"HMAC-SHA256 SignedHeaders=date;host;x-ms-content-sha256&Signature={signature}";
```

## <a name="add-headers-to-requestmessage"></a>Добавление заголовков в requestMessage

Используйте следующий код, чтобы добавить необходимые заголовки в `requestMessage`.

```csharp
//Add a content hash header.
requestMessage.Headers.Add("x-ms-content-sha256", contentHash);
//Add a date header.
requestMessage.Headers.Add("Date", date);
//Add an authorization header.
requestMessage.Headers.Add("Authorization", authorizationHeader);
```

## <a name="test-the-client"></a>Тестирование клиента

Вызовите конечную точку с помощью `HttpClient` и проверьте ответ.

```csharp
HttpClient httpClient = new HttpClient
{
    BaseAddress = requestUri
};
var response = await httpClient.SendAsync(requestMessage);
var responseString = await response.Content.ReadAsStringAsync();
Console.WriteLine(responseString);
```
