---
title: Ведение журнала ошибок и исключений в MSAL.NET
titleSuffix: Microsoft identity platform
description: Сведения об записи ошибок и исключений в MSAL.NET
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 01/25/2021
ms.author: marsma
ms.reviewer: saeeda, jmprieur
ms.custom: aaddev
ms.openlocfilehash: da36ce0a956e0c3ed369a676960bdb9b5c5b1199
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98954840"
---
# <a name="logging-in-msalnet"></a>Ведение журнала в MSAL.NET

[!INCLUDE [MSAL logging introduction](../../../includes/active-directory-develop-error-logging-introduction.md)]

## <a name="configure-logging-in-msalnet"></a>Настройка ведения журнала в MSAL.NET

В MSAL 3.x ведение журналов настраивается с помощью модификатора построителя `.WithLogging` при создании каждого приложения. Этот метод принимает необязательные параметры:

- `Level` позволяет выбрать необходимый уровень ведения журнала. Если указать Errors, будут регистрироваться только ошибки.
- `PiiLoggingEnabled` позволяет регистрировать личные и организационные данные, если задано значение true. По умолчанию задано значение false, чтобы приложение не регистрировало в журнал персональные данные.
- `LogCallback` задается делегат, который выполняет ведение журнала. Если `PiiLoggingEnabled` имеет значение true, этот метод получит сообщения дважды: один раз с `containsPii` параметром равно false и сообщением без персональных данных, а второй раз с `containsPii` параметром равно true, а сообщение может содержать персональные данные. В некоторых случаях (если сообщение не содержит персональные данные) сообщения будут одинаковыми.
- `DefaultLoggingEnabled` включает ведение журнала по умолчанию для платформы. Значение по умолчанию — false. Если задано значение true, используется трассировка событий в классических приложениях или приложениях UWP, NSLog в iOS и Logcat в Android.

```csharp
class Program
 {
  private static void Log(LogLevel level, string message, bool containsPii)
  {
     if (containsPii)
     {
        Console.ForegroundColor = ConsoleColor.Red;
     }
     Console.WriteLine($"{level} {message}");
     Console.ResetColor();
  }

  static void Main(string[] args)
  {
    var scopes = new string[] { "User.Read" };

    var application = PublicClientApplicationBuilder.Create("<clientID>")
                      .WithLogging(Log, LogLevel.Info, true)
                      .Build();

    AuthenticationResult result = application.AcquireTokenInteractive(scopes)
                                             .ExecuteAsync().Result;
  }
 }
 ```

> [!TIP]
 > Примеры ведения журнала MSAL.NET и многое другое см. на [вики-сайте MSAL.NET](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki) .

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные примеры кода см. в статье [примеры кода платформы идентификации Майкрософт](sample-v2-code.md).