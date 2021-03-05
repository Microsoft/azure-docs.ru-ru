---
title: включить файл
description: включить файл
services: azure-communication-services
author: tomaschladek
manager: nmurav
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 08/20/2020
ms.topic: include
ms.custom: include file
ms.author: tchladek
ms.openlocfilehash: 2213da7b9c6e4776a0e463e6a43d0cc645a1e911
ms.sourcegitcommit: c27a20b278f2ac758447418ea4c8c61e27927d6a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/03/2021
ms.locfileid: "101750182"
---
## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно. 
- Последняя версия [клиентской библиотеки NET Core](https://dotnet.microsoft.com/download/dotnet-core) для вашей операционной системы.
- Активный ресурс Служб коммуникации и строка подключения. [Создайте ресурс Служб коммуникации.](../create-communication-resource.md)

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-c-application"></a>Создание нового приложения C#

В окне консоли (cmd, PowerShell или Bash) выполните команду `dotnet new`, чтобы создать консольное приложение с именем `AccessTokensQuickstart`. Эта команда создает простой проект Hello World на языке C# с одним файлом исходного кода: **Program.cs**.

```console
dotnet new console -o AccessTokensQuickstart
```

Измените каталог на только что созданную папку приложения и выполните команду `dotnet build`, чтобы скомпилировать приложение.

```console
cd AccessTokensQuickstart
dotnet build
```

### <a name="install-the-package"></a>Установка пакета

Оставаясь в каталоге приложения, установите пакет библиотеки удостоверений Служб коммуникации Azure для .NET с помощью команды `dotnet add package`.

```console
dotnet add package Azure.Communication.Identity
```

### <a name="set-up-the-app-framework"></a>Настройка платформы приложения

Из каталога проекта:

1. Откройте файл **Program.cs** в текстовом редакторе.
1. Добавьте директиву `using` для включения пространства имен `Azure.Communication.Identity`.
1. Обновите объявление метода `Main` для поддержки асинхронного кода.

Используйте следующий код:

```csharp
using System;
using Azure.Communication;
using Azure.Communication.Identity;

namespace AccessTokensQuickstart
{
    class Program
    {
        static async System.Threading.Tasks.Task Main(string[] args)
        {
            Console.WriteLine("Azure Communication Services - Access Tokens Quickstart");

            // Quickstart code goes here
        }
    }
}
```
## <a name="authenticate-the-client"></a>Аутентификация клиента

Инициализируйте экземпляр `CommunicationIdentityClient` с использованием строки подключения. Приведенный ниже код извлекает строку подключения для ресурса из переменной среды с именем `COMMUNICATION_SERVICES_CONNECTION_STRING`. См. сведения о том, как [управлять строкой подключения ресурса](../create-communication-resource.md#store-your-connection-string).

Добавьте следующий код в метод `Main`:

```csharp
// This code demonstrates how to fetch your connection string
// from an environment variable.
string connectionString = Environment.GetEnvironmentVariable("COMMUNICATION_SERVICES_CONNECTION_STRING");
var client = new CommunicationIdentityClient(connectionString);
```

## <a name="create-an-identity"></a>Создание удостоверения

Службы коммуникации Azure позволяют использовать упрощенный каталог удостоверений. Используйте метод `createUser`, чтобы создать новую запись в каталоге с уникальным значением `Id`. Магазин получил удостоверение с сопоставлением с пользователями приложения. Например, сохраните их в базе данных сервера приложений. Удостоверение потребуется позже для выдачи маркеров доступа.

```csharp
var identityResponse = await client.CreateUserAsync();
var identity = identityResponse.Value;
Console.WriteLine($"\nCreated an identity with ID: {identity.Id}");
```

## <a name="issue-identity-access-tokens"></a>Выдача маркеров доступа идентификаторов

Чтобы выдать маркер доступа для имеющегося удостоверения Служб коммуникации, используйте метод `issueToken`. Параметр `scopes` определяет набор базовых функций, которые будут авторизовать этот маркер доступа. Ознакомьтесь со [списком поддерживаемых действий](../../concepts/authentication.md). Новый экземпляр параметра `communicationUser` можно создать на основе строкового представления удостоверения Служб коммуникации Azure.

```csharp
// Issue an access token with the "voip" scope for an identity
var tokenResponse = await client.IssueTokenAsync(identity, scopes: new [] { CommunicationTokenScope.VoIP });
var token =  tokenResponse.Value.Token;
var expiresOn = tokenResponse.Value.ExpiresOn;
Console.WriteLine($"\nIssued an access token with 'voip' scope that expires at {expiresOn}:");
Console.WriteLine(token);
```

Маркеры доступа — это недолговечные учетные данные, которые необходимо выдавать повторно. Если этого не сделать, это может привести к нарушению работы пользователей приложения. Свойство ответа `expiresOn` указывает время существования маркера доступа. 

## <a name="create-an-identity-and-issue-an-access-token-within-the-same-request"></a>Создание удостоверения и выдача маркера доступа в рамках одного запроса

Чтобы создать удостоверение Служб коммуникации и выдать для него маркер доступа, используйте метод `createUserWithToken`. Параметр `scopes` определяет набор базовых функций, которые будут авторизовать этот маркер доступа. Ознакомьтесь со [списком поддерживаемых действий](../../concepts/authentication.md).

```csharp  
// Issue an identity and an access token with the "voip" scope for the new identity
var identityWithTokenResponse = await client.CreateUserWithTokenAsync(scopes: new[] { CommunicationTokenScope.VoIP });
var identity = identityWithTokenResponse.Value.user.Id;
var token = identityWithTokenResponse.Value.token.Token;
var expiresOn = identityWithTokenResponse.Value.token.ExpiresOn;
```

## <a name="refresh-access-tokens"></a>Обновление токенов доступа

Чтобы обновить маркер доступа, передайте экземпляр объекта `CommunicationUser` в метод `IssueTokenAsync`. Если вы сохранили значение `Id` и вам нужно создать новый объект `CommunicationUser`, для этого можно передать сохраненное значение `Id` в конструктор `CommunicationUser` следующим образом:

```csharp  
// In this example, userId is a string containing the Id property of a previously-created CommunicationUser
identityToRefresh = new CommunicationUser(userId);
tokenResponse = await client.IssueTokenAsync(identityToRefresh, scopes: new [] { CommunicationTokenScope.VoIP });
```

## <a name="revoke-access-tokens"></a>Отмена маркеров доступа

В некоторых случаях вы можете явно отменить маркеры доступа. Например, когда пользователь приложения меняет пароль, который он использует для проверки подлинности в службе. Метод `RevokeTokensAsync` аннулирует все активные маркеры доступа, выданные удостоверению.

```csharp  
await client.RevokeTokensAsync(identity);
Console.WriteLine($"\nSuccessfully revoked all access tokens for identity with ID: {identity.Id}");
```

## <a name="delete-an-identity"></a>Удаление удостоверения

При удалении идентификатора отменяются все активные маркеры доступа и запрещается выдача последующих маркеров для идентификатора. Вместе с этим удаляется и все хранимое содержимое, связанное с удостоверением.

```csharp
await client.DeleteUserAsync(identity);
Console.WriteLine($"\nDeleted the identity with ID: {identity.Id}");
```

## <a name="run-the-code"></a>Выполнение кода

Запустите приложение из каталога приложения с помощью команды `dotnet run`.

```console
dotnet run
```
