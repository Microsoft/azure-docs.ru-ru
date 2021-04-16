---
title: включить файл
description: включить файл
services: azure-communication-services
author: peiliu
manager: rejooyan
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 03/11/2021
ms.topic: include
ms.custom: include file
ms.author: peiliu
ms.openlocfilehash: 5fd209c612f90e3912e244daf60d20edf30a08c6
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106113399"
---
Начало работы со Службами коммуникации Azure с помощью пакета SDK для SMS Служб коммуникации Azure для C# для отправки SMS-сообщений.

Выполнение этого краткого руководства предполагает небольшую дополнительную плату в несколько центов США в учетной записи Azure.

<!--**TODO: update all these reference links as the resources go live**

[API reference documentation](../../../references/overview.md) | [Library source code](https://github.com/Azure/azure-sdk-for-net-pr/tree/feature/communication/sdk/communication/Azure.Communication.Sms#todo-update-to-public) | [Package (NuGet)](#todo-nuget) | [Samples](#todo-samples)-->

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- Последняя версия [пакета SDK для .NET Core](https://dotnet.microsoft.com/download/dotnet-core) для вашей операционной системы.
- Активный ресурс Служб коммуникации и строка подключения. [Создайте ресурс Служб коммуникации.](../../create-communication-resource.md)
- Номер телефона с поддержкой SMS-сообщений. [Получите номер телефона.](../get-phone-number.md)

### <a name="prerequisite-check"></a>Проверка предварительных условий

- В окне терминала или командном окне выполните команду `dotnet`, чтобы проверить, установлен ли пакет SDK для .NET.
- Чтобы просмотреть номера телефонов, связанные с ресурсом Служб коммуникации, войдите на [портал Azure](https://portal.azure.com/), перейдите к ресурсу Служб коммуникации и откройте вкладку с **номерами телефонов** в области навигации слева.

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-c-application"></a>Создание нового приложения C#

В окне консоли (cmd, PowerShell или Bash) выполните команду `dotnet new`, чтобы создать консольное приложение с именем `SmsQuickstart`. Эта команда создает простой проект Hello World на языке C# с одним файлом исходного кода: **Program.cs**.

```console
dotnet new console -o SmsQuickstart
```

Измените каталог на только что созданную папку приложения и выполните команду `dotnet build`, чтобы скомпилировать приложение.

```console
cd SmsQuickstart
dotnet build
```

### <a name="install-the-package"></a>Установка пакета

Оставаясь в каталоге приложения, установите пакет SDK для SMS Служб коммуникации для пакета .NET с помощью команды `dotnet add package`.

```console
dotnet add package Azure.Communication.Sms --version 1.0.0
```

Добавьте директиву `using` в начало **Program.cs**, чтобы включить пространство имен `Azure.Communication`.

```csharp

using System;
using System.Collections.Generic;

using Azure;
using Azure.Communication;
using Azure.Communication.Sms;

```

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы реализуют некоторые основные функции пакета SDK для SMS Служб коммуникации для C#.

| Имя                                       | Описание                                                                                                                                                       |
| ------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| SmsClient     | Этот класс требуется для реализации всех функций обмена текстовыми сообщениями. Его экземпляр можно создать на основе сведений о подписке и использовать для отправки SMS.                           |
| SmsSendOptions | Этот класс предоставляет параметры для настройки отчетов о доставке. Если enable_delivery_report имеет значение True, при успешной доставке будет создано событие. |
| SmsSendResult               | Этот класс содержит результат, полученный от службы SMS.                                          |

## <a name="authenticate-the-client"></a>Аутентификация клиента

 Откройте **Program.cs** в текстовом редакторе и замените текст метода `Main` кодом, чтобы инициализировать `SmsClient` с помощью строки подключения. Приведенный ниже код извлекает строку подключения для ресурса из переменной среды с именем `COMMUNICATION_SERVICES_CONNECTION_STRING`. См. сведения о том, как [управлять строкой подключения ресурса](../../create-communication-resource.md#store-your-connection-string).


```csharp
// This code demonstrates how to fetch your connection string
// from an environment variable.
string connectionString = Environment.GetEnvironmentVariable("COMMUNICATION_SERVICES_CONNECTION_STRING");

SmsClient smsClient = new SmsClient(connectionString);
```

## <a name="send-a-11-sms-message"></a>Отправка личного текстового сообщения

Чтобы отправить текстовое сообщение одному получателю, вызовите функцию `Send` или `SendAsync` из SmsClient. Добавьте следующий код в конец метода `Main` в **Program.cs**:

```csharp
SmsSendResult sendResult = smsClient.Send(
    from: "<from-phone-number>",
    to: "<to-phone-number>",
    message: "Hello World via SMS"
);

Console.WriteLine($"Sms id: {sendResult.MessageId}");
```
`<from-phone-number>` необходимо заменить номером телефона с поддержкой SMS, связанным с ресурсом Служб коммуникации, а `<to-phone-number>` — номером телефона, на который вы хотите отправить сообщение.

> [!WARNING]
> Обратите внимание, что номера телефонов должны быть указаны в формате международного стандарта E.164, (например, +14255550123).

## <a name="send-a-1n-sms-message-with-options"></a>Отправка группового текстового сообщения с параметрами
Чтобы отправить текстовое сообщение списку получателей, вызовите функцию `Send` или `SendAsync` из SmsClient со списком номеров телефонов получателей. Кроме того, вы можете передать необязательные параметры для указания того, должен ли быть включен отчет о доставке, а также задать пользовательские теги.

```csharp
Response<IReadOnlyList<SmsSendResult>> response = smsClient.Send(
    from: "<from-phone-number>",
    to: new string[] { "<to-phone-number-1>", "<to-phone-number-2>" },
    message: "Weekly Promotion!",
    options: new SmsSendOptions(enableDeliveryReport: true) // OPTIONAL
    {
        Tag = "marketing", // custom tags
    });

IEnumerable<SmsSendResult> results = response.Value;
foreach (SmsSendResult result in results)
{
    Console.WriteLine($"Sms id: {result.MessageId}");
    Console.WriteLine($"Send Result Successful: {result.Successful}");
}
```

`<from-phone-number>` необходимо заменить номером телефона с поддержкой SMS, связанным с ресурсом Служб коммуникации, а `<to-phone-number-1>` и `<to-phone-number-2>` — номерами телефона, на которые вы хотите отправить сообщение.

> [!WARNING]
> Обратите внимание, что номера телефонов должны быть указаны в формате международного стандарта E.164, (например, +14255550123).

Параметр `enableDeliveryReport` является необязательным. Его можно использовать для настройки отчетов о доставке. Это полезно, если вы хотите, чтобы при доставке SMS-сообщений создавались события. Сведения о настройке отчетов о доставке SMS-сообщений см. в кратком руководстве [Обработка событий SMS-сообщений](../handle-sms-events.md).

Для применения тега к отчету о доставке используется `Tag`

## <a name="run-the-code"></a>Выполнение кода

Запустите приложение из каталога приложения с помощью команды `dotnet run`.

```console
dotnet run
```

## <a name="sample-code"></a>Пример кода

Пример приложения можно скачать в репозитории [GitHub](https://github.com/Azure-Samples/communication-services-dotnet-quickstarts/tree/main/SendSMS).
