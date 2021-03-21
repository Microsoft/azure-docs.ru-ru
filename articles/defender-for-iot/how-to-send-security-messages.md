---
title: Отправка защитника для сообщений безопасности устройств IoT
description: Узнайте, как отправить сообщения безопасности с помощью защитника для Интернета вещей.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: shhazam-ms
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 2/8/2021
ms.author: shhazam
ms.custom: devx-track-js
ms.openlocfilehash: d59121f2dbae208ba045a9c2e6d66245296537a0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99820693"
---
# <a name="send-security-messages-sdk"></a>Пакет SDK для отправки сообщений системы безопасности

В этом пошаговом руководство поясняются возможности службы IoT, если вы решили получать и отсылать сообщения безопасности устройства без использования защитника для агента IoT, а также объясняется, как это сделать.

Из этого руководства вы узнаете, как выполнить следующие задачи:

> [!div class="checklist"]
> * Отправка сообщений безопасности с помощью пакета SDK для Azure IoT C
> * Отправка сообщений безопасности с помощью пакета SDK для C# для Azure IoT
> * Отправка сообщений безопасности с помощью пакета SDK Python для Azure IoT
> * Отправка сообщений безопасности с помощью пакета SDK для Node.js Azure IoT
> * Отправка сообщений безопасности с помощью пакета SDK для Java Azure IoT

## <a name="defender-for-iot-capabilities"></a>Защитник для возможностей IoT

Защитник для Интернета вещей может обрабатывать и анализировать любые данные сообщений безопасности при условии, что данные отправляются в [схему защитника для Интернета вещей](https://aka.ms/iot-security-schemas) , а сообщение задается как сообщение безопасности.

## <a name="security-message"></a>Сообщение безопасности

Защитник для IoT определяет сообщение безопасности, используя следующие критерии:

- Если сообщение было отправлено с помощью пакета SDK для Azure IoT
- Если сообщение соответствует [схеме сообщения безопасности](https://aka.ms/iot-security-schemas)
- Если сообщение было задано как сообщение безопасности перед отправкой

Каждое сообщение безопасности содержит метаданные отправителя, такие как `AgentId` , `AgentVersion` `MessageSchemaVersion` и список событий безопасности.
Схема определяет допустимые и обязательные свойства сообщения безопасности, включая типы событий.

> [!NOTE]
> Отправленные сообщения, которые не соответствуют схеме, игнорируются. Не забудьте проверить схему, прежде чем отправлять данные, так как игнорируемые сообщения в данный момент не хранятся.

> [!NOTE]
> Отправленные сообщения, которые не были заданы в качестве сообщения безопасности с помощью пакета Azure IoT SDK, не будут направляться в защитник для конвейера IoT.

## <a name="valid-message-example"></a>Пример допустимого сообщения

В приведенном ниже примере показан допустимый объект сообщения безопасности. В примере содержатся метаданные сообщения и одно `ProcessCreate` событие безопасности.

После установки в качестве сообщения безопасности и отправки оно будет обработано защитником для Интернета вещей.

```json
"AgentVersion": "0.0.1",
"AgentId": "e89dc5f5-feac-4c3e-87e2-93c16f010c25",
"MessageSchemaVersion": "1.0",
"Events": [
    {
        "EventType": "Security",
        "Category": "Triggered",
        "Name": "ProcessCreate",
        "IsEmpty": false,
        "PayloadSchemaVersion": "1.0",
        "Id": "21a2db0b-44fe-42e9-9cff-bbb2d8fdf874",
        "TimestampLocal": "2019-01-27 15:48:52Z",
        "TimestampUTC": "2019-01-27 13:48:52Z",
        "Payload":
            [
                {
                    "Executable": "/usr/bin/myApp",
                    "ProcessId": 11750,
                    "ParentProcessId": 1593,
                    "UserName": "aUser",
                    "CommandLine": "myApp -a -b"
                }
            ]
    }
]
```

## <a name="send-security-messages"></a>Отправка сообщений системы безопасности

Отправка сообщений безопасности *без* использования защитника для агента IOT с помощью [пакета SDK](https://github.com/Azure/azure-iot-sdk-c/tree/public-preview)для устройств с Azure IOT C, [пакет SDK для устройств C# для Azure IOT](https://github.com/Azure/azure-iot-sdk-csharp/tree/preview), пакет SDK для [Azure IOT Node.js](https://github.com/Azure/azure-iot-sdk-node), пакет SDK для Microsoft Azure [IOT](https://github.com/Azure/azure-iot-sdk-python)или [пакет SDK для Azure IOT Java](https://github.com/Azure/azure-iot-sdk-java).

Чтобы отправить данные устройства с устройств для обработки с помощью защитника для Интернета вещей, используйте один из следующих API-интерфейсов, чтобы пометить сообщения для правильной маршрутизации для конвейера обработки IoT.

Все отправляемые данные, даже если они отмечены правильным заголовком, также должны соответствовать [схеме сообщений защитника для Интернета вещей](https://aka.ms/iot-security-schemas).

### <a name="send-security-message-api"></a>API для отправки сообщений системы безопасности

API **отправки сообщений безопасности** в настоящее время доступен на языках C и C#, Python, Node.js и Java.

#### <a name="c-api"></a>API C

```c
bool SendMessageAsync(IoTHubAdapter* iotHubAdapter, const void* data, size_t dataSize) {

    bool success = true;
    IOTHUB_MESSAGE_HANDLE messageHandle = NULL;

    messageHandle = IoTHubMessage_CreateFromByteArray(data, dataSize);

    if (messageHandle == NULL) {
        success = false;
        goto cleanup;
    }

    if (IoTHubMessage_SetAsSecurityMessage(messageHandle) != IOTHUB_MESSAGE_OK) {
        success = false;
        goto cleanup;
    }

    if (IoTHubModuleClient_SendEventAsync(iotHubAdapter->moduleHandle, messageHandle, SendConfirmCallback, iotHubAdapter) != IOTHUB_CLIENT_OK) {
        success = false;
        goto cleanup;
    }

cleanup:
    if (messageHandle != NULL) {
        IoTHubMessage_Destroy(messageHandle);
    }

    return success;
}

static void SendConfirmCallback(IOTHUB_CLIENT_CONFIRMATION_RESULT result, void* userContextCallback) {
    if (userContextCallback == NULL) {
        //error handling
        return;
    }

    if (result != IOTHUB_CLIENT_CONFIRMATION_OK){
        //error handling
    }
}
```

#### <a name="c-api"></a>API C#

```cs

private static async Task SendSecurityMessageAsync(string messageContent)
{
    ModuleClient client = ModuleClient.CreateFromConnectionString("<connection_string>");
    Message  securityMessage = new Message(Encoding.UTF8.GetBytes(messageContent));
    securityMessage.SetAsSecurityMessage();
    await client.SendEventAsync(securityMessage);
}
```

#### <a name="nodejs-api"></a>API для Node.js

```typescript
var Protocol = require('azure-iot-device-mqtt').Mqtt

function SendSecurityMessage(messageContent)
{
  var client = Client.fromConnectionString(connectionString, Protocol);

  var connectCallback = function (err) {
    if (err) {
      console.error('Could not connect: ' + err.message);
    } else {
      var message = new Message(messageContent);
      message.setAsSecurityMessage();
      client.sendEvent(message);
  
      client.on('error', function (err) {
        console.error(err.message);
      });
  
      client.on('disconnect', function () {
        clearInterval(sendInterval);
        client.removeAllListeners();
        client.open(connectCallback);
      });
    }
  };

  client.open(connectCallback);
}
```

#### <a name="python-api"></a>API Python

Чтобы использовать API Python, необходимо установить пакет [Azure-IOT-Device](https://pypi.org/project/azure-iot-device/).

При использовании API Python можно либо отправить сообщение безопасности через модуль, либо через устройство, используя уникальную строку подключения устройства или модуля. При использовании следующего примера скрипта Python с устройством используйте **иосубдевицеклиент** и модуль, используйте **иосубмодулеклиент**.

```python
from azure.iot.device.aio import IoTHubDeviceClient, IoTHubModuleClient
from azure.iot.device import Message

async def send_security_message_async(message_content):
    conn_str = os.getenv("<connection_string>")
    device_client = IoTHubDeviceClient.create_from_connection_string(conn_str)
    await device_client.connect()
    security_message = Message(message_content)
    security_message.set_as_security_message()
    await device_client.send_message(security_message)
    await device_client.disconnect()
```

#### <a name="java-api"></a>API Java

```java
public void SendSecurityMessage(string message)
{
    ModuleClient client = new ModuleClient("<connection_string>", IotHubClientProtocol.MQTT);
    Message msg = new Message(message);
    msg.setAsSecurityMessage();
    EventCallback callback = new EventCallback();
    string context = "<user_context>";
    client.sendEventAsync(msg, callback, context);
}
```

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с [обзором](overview.md) службы "защитник для Интернета вещей"
- Дополнительные сведения о защитнике для [архитектуры](architecture.md) IOT
- Включите [службу](quickstart-onboard-iot-hub.md).
- Ознакомьтесь с [вопросами и](resources-frequently-asked-questions.md) ответами
- Узнайте, как получить доступ к [необработанным данным безопасности](how-to-security-data-access.md).
- Общие сведения о [рекомендациях](concept-recommendations.md)
- Ознакомьтесь со сведениями об [оповещениях](concept-security-alerts.md).
