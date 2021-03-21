---
title: Обзор интерфейсов API ретранслятора Azure для платформы .NET Standard | Документация Майкрософт
description: В этой статье перечислены некоторые основные сведения о Azure Relay гибридные подключения .NET Standard API.
ms.topic: article
ms.custom: devx-track-csharp
ms.date: 06/23/2020
ms.openlocfilehash: 724fb1a62b82036b4a0fa8b9f4f3608293f608a9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98625137"
---
# <a name="azure-relay-hybrid-connections-net-standard-api-overview"></a>Обзор API-интерфейсов гибридных подключений ретранслятора Azure для платформы .NET Standard

В этой статье перечислены некоторые ключевые [клиентские API-интерфейсы](/dotnet/api/microsoft.azure.relay) гибридных подключений ретранслятора Azure для платформы .NET Standard.
  
## <a name="relay-connection-string-builder-class"></a>Класс построителя строк подключения ретранслятора

Класс [RelayConnectionStringBuilder][RelayConnectionStringBuilder] форматирует строки подключения, которые относятся к гибридным подключениям ретранслятора. Его можно использовать для проверки формата строки подключения, а также для создания строки подключения с нуля. В качестве примера ниже приведен код.

```csharp
var endpoint = "[Relay namespace]";
var entityPath = "[Name of the Hybrid Connection]";
var sharedAccessKeyName = "[SAS key name]";
var sharedAccessKey = "[SAS key value]";

var connectionStringBuilder = new RelayConnectionStringBuilder()
{
    Endpoint = endpoint,
    EntityPath = entityPath,
    SharedAccessKeyName = sasKeyName,
    SharedAccessKey = sasKeyValue
};
```

Вы также можете передать строку подключения непосредственно в метод `RelayConnectionStringBuilder`. Эта операция позволит убедиться, что строка подключения имеет допустимый формат. Если какой-либо из параметров окажется недоступным, конструктор выдаст `ArgumentException`.

```csharp
var myConnectionString = "[RelayConnectionString]";
// Declare the connectionStringBuilder so that it can be used outside of the loop if needed
RelayConnectionStringBuilder connectionStringBuilder;
try
{
    // Create the connectionStringBuilder using the supplied connection string
    connectionStringBuilder = new RelayConnectionStringBuilder(myConnectionString);
}
catch (ArgumentException ae)
{
    // Perform some error handling
}
```

## <a name="hybrid-connection-stream"></a>Поток гибридного подключения

Класс [HybridConnectionStream][HCStream] является основным объектом, с помощью которого выполняется отправка и получение данных через конечную точку ретранслятора Azure независимо от того, что используется —[HybridConnectionClient][HCClient] или [HybridConnectionListener][HCListener].

### <a name="getting-a-hybrid-connection-stream"></a>Получение потока гибридного подключения

#### <a name="listener"></a>Прослушиватель

С помощью объекта [HybridConnectionListener][HCListener] можно получить объект `HybridConnectionStream`:

```csharp
// Use the RelayConnectionStringBuilder to get a valid connection string
var listener = new HybridConnectionListener(csb.ToString());
// Open a connection to the Relay endpoint
await listener.OpenAsync();
// Get a `HybridConnectionStream`
var hybridConnectionStream = await listener.AcceptConnectionAsync();
```

#### <a name="client"></a>Клиент

С помощью объекта [HybridConnectionClient][HCClient] можно получить объект `HybridConnectionStream`:

```csharp
// Use the RelayConnectionStringBuilder to get a valid connection string
var client = new HybridConnectionClient(csb.ToString());
// Open a connection to the Relay endpoint and get a `HybridConnectionStream`
var hybridConnectionStream = await client.CreateConnectionAsync();
```

### <a name="receiving-data"></a>Получение данных

Класс [HybridConnectionStream][HCStream] можно использовать для двустороннего обмена данными. В большинстве случаев вы будете непрерывно получать данные из потока. Если выполняется чтение текста из потока, вам также может потребоваться использовать объект [StreamReader](/dotnet/api/system.io.streamreader) для упрощения анализа данных. Например, можно считывать данные как текст, а не как `byte[]`.

Следующий код считывает отдельные строки текста из потока, пока не будет запрошена отмена.

```csharp
// Create a CancellationToken, so that we can cancel the while loop
var cancellationToken = new CancellationToken();
// Create a StreamReader from the 'hybridConnectionStream`
var streamReader = new StreamReader(hybridConnectionStream);

while (!cancellationToken.IsCancellationRequested)
{
    // Read a line of input until a newline is encountered
    var line = await streamReader.ReadLineAsync();
    if (string.IsNullOrEmpty(line))
    {
        // If there's no input data, we will signal that 
        // we will no longer send data on this connection
        // and then break out of the processing loop.
        await hybridConnectionStream.ShutdownAsync(cancellationToken);
        break;
    }
}
```

### <a name="sending-data"></a>Отправка данных

Когда подключение будет установлено, можно отправить сообщение на конечную точку ретранслятора. Так как объект подключения наследует [поток](/dotnet/api/system.io.stream), отправляйте данные в виде `byte[]`. В приведенном ниже примере показано, как это сделать.

```csharp
var data = Encoding.UTF8.GetBytes("hello");
await clientConnection.WriteAsync(data, 0, data.Length);
```

Однако если вы хотите отправить текст напрямую, без необходимости кодирования этих строк каждый раз, можно поместить объект `hybridConnectionStream` в объект [StreamWriter](/dotnet/api/system.io.streamwriter).

```csharp
// The StreamWriter object only needs to be created once
var textWriter = new StreamWriter(hybridConnectionStream);
await textWriter.WriteLineAsync("hello");
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о ретрансляторе Azure доступны по следующим ссылкам:

* [Microsoft.Azure.Relay Namespace](/dotnet/api/microsoft.azure.relay) (Пространство имен Microsoft.Azure.Relay)
* [Что такое Azure Relay?](relay-what-is-it.md)
* [Available Relay APIs](relay-api-overview.md) (Доступные API-интерфейсы ретранслятора)

[RelayConnectionStringBuilder]: /dotnet/api/microsoft.azure.relay.relayconnectionstringbuilder
[HCStream]: /dotnet/api/microsoft.azure.relay.hybridconnectionstream
[HCClient]: /dotnet/api/microsoft.azure.relay.hybridconnectionclient
[HCListener]: /dotnet/api/microsoft.azure.relay.hybridconnectionlistener
