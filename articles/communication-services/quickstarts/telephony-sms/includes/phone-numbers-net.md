---
ms.openlocfilehash: 0bfb23977f6553568da24df614621bdf1eb9d06d
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106112964"
---
## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- Последняя версия [клиентской библиотеки NET Core](https://dotnet.microsoft.com/download/dotnet-core) для вашей операционной системы.
- Активный ресурс Служб коммуникации и строка подключения. [Создайте ресурс Служб коммуникации.](../../create-communication-resource.md)

### <a name="prerequisite-check"></a>Проверка предварительных условий

- В окне терминала или команды выполните команду `dotnet`, чтобы проверить, установлена ли клиентская библиотека .NET.

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-c-application"></a>Создание нового приложения C#

В окне консоли (cmd, PowerShell или Bash) выполните команду `dotnet new`, чтобы создать консольное приложение с именем `PhoneNumbersQuickstart`. Эта команда создает простой проект Hello World на языке C# с одним файлом исходного кода: **Program.cs**.

```console
dotnet new console -o PhoneNumbersQuickstart
```

Измените каталог на только что созданную папку приложения и выполните команду `dotnet build`, чтобы скомпилировать приложение.

```console
cd PhoneNumbersQuickstart
dotnet build
```

### <a name="install-the-package"></a>Установка пакета

Оставаясь в каталоге приложения, установите клиентскую библиотеку Служб коммуникации PhoneNumbers для .NET с помощью команды `dotnet add package`.

```console
dotnet add package Azure.Communication.PhoneNumbers --version 1.0.0-beta.6
```

Добавьте директиву `using` в начало файла **Program.cs**, чтобы включить пространства имен.

```csharp
using System;
using System.Linq;
using System.Threading.Tasks;
using Azure.Communication.PhoneNumbers;
```

Обновите сигнатуру функции `Main`, чтобы она стала асинхронной.

```csharp
static async Task Main(string[] args)
{
  ...
}
```

## <a name="authenticate-the-client"></a>Аутентификация клиента

Клиенты номера телефона могут проходить проверку подлинности с помощью строки подключения, полученной из ресурсов Коммуникации Azure на [портале Azure] [azure_portal].

```csharp
// Get a connection string to our Azure Communication resource.
var connectionString = "<connection_string>";
var client = new PhoneNumbersClient(connectionString);
```

Кроме того, клиенты номера телефона также могут пройти проверку подлинности с помощью функции проверки подлинности в Azure Active Directory. В этом случае переменные среды `AZURE_CLIENT_SECRET`, `AZURE_CLIENT_ID` и `AZURE_TENANT_ID` должны быть настроены для проверки подлинности.

```csharp
// Get an endpoint to our Azure Communication resource.
var endpoint = new Uri("<endpoint_url>");
TokenCredential tokenCredential = new DefaultAzureCredential();
client = new PhoneNumbersClient(endpoint, tokenCredential);
```

## <a name="manage-phone-numbers"></a>Управление номерами телефонов

### <a name="search-for-available-phone-numbers"></a>Поиск доступных номеров телефонов

Чтобы приобрести номера телефонов, сначала необходимо найти доступные номера. Для поиска номеров телефонов укажите код города, тип назначения, [возможности номеров телефона](../../../concepts/telephony-sms/plan-solution.md#phone-number-capabilities-in-azure-communication-services), [тип номера телефона](../../../concepts/telephony-sms/plan-solution.md#phone-number-types-in-azure-communication-services) и количество. Обратите внимание, что для типа бесплатного телефонного номера указывать код города необязательно.

```csharp
var capabilities = new PhoneNumberCapabilities(calling:PhoneNumberCapabilityType.None, sms:PhoneNumberCapabilityType.Outbound);
var searchOptions = new PhoneNumberSearchOptions { AreaCode = "833", Quantity = 1 };

var searchOperation = await client.StartSearchAvailablePhoneNumbersAsync("US", PhoneNumberType.TollFree, PhoneNumberAssignmentType.Application, capabilities, searchOptions);
await searchOperation.WaitForCompletionAsync();
```

### <a name="purchase-phone-numbers"></a>Покупка номеров телефонов

Результатом поиска номеров телефона является `PhoneNumberSearchResult`. Он содержит `SearchId`, который можно передать в API для покупки номеров, чтобы купить найденные номера. Обратите внимание, что при вызове API для покупки номеров телефона на учетную запись Azure будет начисляться платеж.

```csharp
var purchaseOperation = await client.StartPurchasePhoneNumbersAsync(searchOperation.Value.SearchId);
await purchaseOperation.WaitForCompletionResponseAsync();
```

### <a name="get-phone-numbers"></a>Получение номеров телефона

После покупки номера его можно получить из клиента.
```csharp
var getPhoneNumberResponse = await client.GetPurchasedPhoneNumberAsync("+14255550123");
Console.WriteLine($"Phone number: {getPhoneNumberResponse.Value.PhoneNumber}, country code: {getPhoneNumberResponse.Value.CountryCode}");
```

Вы также можете получить все приобретенные номера телефонов.
``` csharp
var purchasedPhoneNumbers = client.GetPurchasedPhoneNumbersAsync();
await foreach (var purchasedPhoneNumber in purchasedPhoneNumbers)
{
    Console.WriteLine($"Phone number: {purchasedPhoneNumber.PhoneNumber}, country code: {purchasedPhoneNumber.CountryCode}");
}
```

### <a name="update-phone-number-capabilities"></a>Обновление возможностей номера телефона

Возможности приобретенного номера можно обновить.

````csharp
var updateCapabilitiesOperation = await client.StartUpdateCapabilitiesAsync("+14255550123", calling: PhoneNumberCapabilityType.Outbound, sms: PhoneNumberCapabilityType.InboundOutbound);
await updateCapabilitiesOperation.WaitForCompletionAsync();
````


### <a name="release-phone-number"></a>Освобождение номера телефона

Приобретенный номер телефона можно освободить.

````csharp
var releaseOperation = await client.StartReleasePhoneNumberAsync("+14255550123");
await releaseOperation.WaitForCompletionResponseAsync();
````

## <a name="run-the-code"></a>Выполнение кода

Запустите приложение из каталога приложения с помощью команды `dotnet run`.

```console
dotnet run
```

## <a name="sample-code"></a>Пример кода

Пример приложения можно скачать в репозитории [GitHub](https://github.com/Azure-Samples/communication-services-dotnet-quickstarts/tree/main/PhoneNumbers).
