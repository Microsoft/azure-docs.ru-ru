---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/19/2020
ms.openlocfilehash: 38a29aa765bde4adb390423a9d180132b57d4fee
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104582804"
---
## <a name="model-id-announcement"></a>Объявление идентификатора модели

Чтобы объявить идентификатор модели, устройство должно включить его в сведения о подключении:

```csharp
DeviceClient.CreateFromConnectionString(
  connectionString,
  TransportType.Mqtt,
  new ClientOptions() { ModelId = modelId })
```

Новая `ClientOptions` перегрузка доступна во всех `DeviceClient` методах, используемых для инициализации соединения.

> [!TIP]
> Для модулей и IoT Edge используйте `ModuleClient` вместо `DeviceClient` .

> [!TIP]
> Это единственный момент, когда устройство может задать идентификатор модели, оно не может быть обновлено после подключения устройства.

## <a name="dps-payload"></a>Полезные данные DPS

Устройства, использующие [службу подготовки устройств (DPS)](../articles/iot-dps/about-iot-dps.md) , могут включать в себя `modelId` для использования в процессе подготовки с помощью следующих полезных данных JSON.

```json
{
    "modelId" : "dtmi:com:example:Thermostat;1"
}
```

## <a name="implement-telemetry-properties-and-commands"></a>Реализация телеметрии, свойств и команд

Как описано в разделе [Общие сведения о компонентах Самонастраивающийся моделей Интернета вещей](../articles/iot-pnp/concepts-modeling-guide.md), построители устройств должны решить, хотят ли они использовать компоненты для описания своих устройств. При использовании компонентов устройства должны соответствовать правилам, описанным в этом разделе.

### <a name="telemetry"></a>Телеметрия

Компонент по умолчанию не требует какого-либо специального свойства.

При использовании вложенных компонентов устройства должны задать свойство Message с именем компонента:

```c#
public async Task SendComponentTelemetryValueAsync(string componentName, string serializedTelemetry)
{
  var message = new Message(Encoding.UTF8.GetBytes(serializedTelemetry));
  message.ComponentName = componentName;
  message.ContentType = "application/json";
  message.ContentEncoding = "utf-8";
  await client.SendEventAsync(message);
}
```

### <a name="read-only-properties"></a>Свойства только для чтения

Для сообщения свойства из компонента по умолчанию не требуется специальная конструкция:

```csharp
TwinCollection reportedProperties = new TwinCollection();
reportedProperties["maxTemperature"] = 38.7;
await client.UpdateReportedPropertiesAsync(reportedProperties);
```

Двойника устройства обновляется со следующим сообщаемым свойством:

```json
{
  "reported": {
      "maxTemperature" : 38.7
  }
}
```

При использовании вложенных компонентов свойства должны быть созданы в имени компонента:

```csharp
TwinCollection reportedProperties = new TwinCollection();
TwinCollection component = new TwinCollection();
component["maxTemperature"] = 38.7;
component["__t"] = "c"; // marker to identify a component
reportedProperties["thermostat1"] = component;
await client.UpdateReportedPropertiesAsync(reportedProperties);
```

Двойника устройства обновляется со следующим сообщаемым свойством:

```json
{
  "reported": {
    "thermostat1" : {  
      "__t" : "c",  
      "maxTemperature" : 38.7
     } 
  }
}
```

### <a name="writable-properties"></a>Доступные для записи свойства

Эти свойства можно задать на устройстве или обновить с помощью решения. Если решение обновляет свойство, клиент получает уведомление в качестве обратного вызова в `DeviceClient` или `ModuleClient` . Чтобы следовать правилам самонастраивающийся IoT, устройство должно информировать службу о том, что свойство было успешно получено.

#### <a name="report-a-writable-property"></a>Сообщить о свойстве, доступном для записи

Когда устройство сообщает о доступном для записи свойстве, оно должно включать `ack` значения, определенные в соглашениях.

Чтобы сообщить свойство, доступное для записи, из компонента по умолчанию:

```csharp
TwinCollection reportedProperties = new TwinCollection();
TwinCollection ackProps = new TwinCollection();
ackProps["value"] = 23.2;
ackProps["ac"] = 200; // using HTTP status codes
ackProps["av"] = 0; // not readed from a desired property
ackProps["ad"] = "reported default value";
reportedProperties["targetTemperature"] = ackProps;
await client.UpdateReportedPropertiesAsync(reportedProperties);
```

Двойника устройства обновляется со следующим сообщаемым свойством:

```json
{
  "reported": {
      "targetTemperature": {
          "value": 23.2,
          "ac": 200,
          "av": 3,
          "ad": "complete"
      }
  }
}
```

Чтобы сообщить свойство, доступное для записи, из вложенного компонента, двойника должен включать маркер:

```csharp
TwinCollection reportedProperties = new TwinCollection();
TwinCollection component = new TwinCollection();
TwinCollection ackProps = new TwinCollection();
component["__t"] = "c"; // marker to identify a component
ackProps["value"] = 23.2;
ackProps["ac"] = 200; // using HTTP status codes
ackProps["av"] = 0; // not read from a desired property
ackProps["ad"] = "reported default value";
component["targetTemperature"] = ackProps;
reportedProperties["thermostat1"] = component;
await client.UpdateReportedPropertiesAsync(reportedProperties);
```

Двойника устройства обновляется со следующим сообщаемым свойством:

```json
{
  "reported": {
    "thermostat1": {
      "__t" : "c",
      "targetTemperature": {
          "value": 23.2,
          "ac": 200,
          "av": 3,
          "ad": "complete"
      }
    }
  }
}
```

#### <a name="subscribe-to-desired-property-updates"></a>Подпишитесь на обновления требуемых свойств

Службы могут обновлять требуемые свойства, которые инициируют уведомление на подключенных устройствах. Это уведомление содержит обновленные требуемые свойства, включая номер версии, определяющий обновление. Устройства должны отвечать с тем же `ack` сообщением, что и сообщаемые свойства.

Компонент по умолчанию видит единственное свойство и создает отчет `ack` с полученной версией:

```csharp
await client.SetDesiredPropertyUpdateCallbackAsync(async (desired, ctx) => 
{
  JValue targetTempJson = desired["targetTemperature"];
  double targetTemperature = targetTempJson.Value<double>();

  TwinCollection reportedProperties = new TwinCollection();
  TwinCollection ackProps = new TwinCollection();
  ackProps["value"] = targetTemperature;
  ackProps["ac"] = 200;
  ackProps["av"] = desired.Version; 
  ackProps["ad"] = "desired property received";
  reportedProperties["targetTemperature"] = ackProps;

  await client.UpdateReportedPropertiesAsync(reportedProperties);
}, null);
```

Двойника устройства показывает свойство в нужных и сообщаемых разделах:

```json
{
  "desired" : {
    "targetTemperature": 23.2,
    "$version" : 3
  },
  "reported": {
      "targetTemperature": {
          "value": 23.2,
          "ac": 200,
          "av": 3,
          "ad": "complete"
      }
  }
}
```

Вложенный компонент получает требуемые свойства, заключенные в оболочку с именем компонента, и должен сообщить `ack` сообщаемое свойство:

```csharp
await client.SetDesiredPropertyUpdateCallbackAsync(async (desired, ctx) =>
{
  JObject thermostatComponent = desired["thermostat1"];
  JToken targetTempProp = thermostatComponent["targetTemperature"];
  double targetTemperature = targetTempProp.Value<double>();

  TwinCollection reportedProperties = new TwinCollection();
  TwinCollection component = new TwinCollection();
  TwinCollection ackProps = new TwinCollection();
  component["__t"] = "c"; // marker to identify a component
  ackProps["value"] = targetTemperature;
  ackProps["ac"] = 200; // using HTTP status codes
  ackProps["av"] = desired.Version; // not readed from a desired property
  ackProps["ad"] = "desired property received";
  component["targetTemperature"] = ackProps;
  reportedProperties["thermostat1"] = component;

  await client.UpdateReportedPropertiesAsync(reportedProperties);
}, null);
```

Двойника устройства для вложенного компонента отображает нужные и указанные разделы следующим образом:

```json
{
  "desired" : {
    "thermostat1" : {
        "__t" : "c",
        "targetTemperature": 23.2,
    }
    "$version" : 3
  },
  "reported": {
    "thermostat1" : {
        "__t" : "c",
      "targetTemperature": {
          "value": 23.2,
          "ac": 200,
          "av": 3,
          "ad": "complete"
      }
    }
  }
}
```

### <a name="commands"></a>Команды

Компонент по умолчанию получает имя команды в том виде, в котором она была вызвана службой.

Вложенный компонент получает имя команды с префиксом имени компонента и `*` разделителя.

```csharp
await client.SetMethodHandlerAsync("themostat*reboot", (MethodRequest req, object ctx) =>
{
  Console.WriteLine("REBOOT");
  return Task.FromResult(new MethodResponse(200));
},
null);
```

#### <a name="request-and-response-payloads"></a>Полезные данные запросов и ответов

Команды используют типы для определения полезных данных запроса и ответа. Устройство должно десериализовать входящий входной параметр и сериализовать ответ. В следующем примере показано, как реализовать команду со сложными типами, определенными в полезных данных:

```json
{
  "@type": "Command",
  "name": "start",
  "request": {
    "name": "startRequest",
    "schema": {
      "@type": "Object",
      "fields": [
        {
          "name": "startPriority",
          "schema": "integer"
        },
        {
          "name": "startMessage",
          "schema" : "string"
        }
      ]
    }
  },
  "response": {
    "name": "startReponse",
    "schema": {
      "@type": "Object",
      "fields": [
        {
            "name": "startupTime",
            "schema": "integer" 
        },
        {
          "name": "startupMessage",
          "schema": "string"
        }
      ]
    }
  }
}
```

В следующих фрагментах кода показано, как устройство реализует это определение команды, включая типы, используемые для включения сериализации и десериализации:

```csharp
class startRequest
{
  public int startPriority { get; set; }
  public string startMessage { get; set; }
}

class startResponse
{
  public int startupTime { get; set; }
  public string startupMessage { get; set; }
}

// ... 

await client.SetMethodHandlerAsync("start", (MethodRequest req, object ctx) =>
{
  var startRequest = JsonConvert.DeserializeObject<startRequest>(req.DataAsJson);
  Console.WriteLine($"Received start command with priority ${startRequest.startPriority} and ${startRequest.startMessage}");

  var startResponse = new startResponse
  {
    startupTime = 123,
    startupMessage = "device started with message " + startRequest.startMessage
  };

  string responsePayload = JsonConvert.SerializeObject(startResponse);
  MethodResponse response = new MethodResponse(Encoding.UTF8.GetBytes(responsePayload), 200);
  return Task.FromResult(response);
},null);
```

> [!Tip]
> Имена запросов и ответов отсутствуют в сериализованных полезных данных, передаваемых по сети.
