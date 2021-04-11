---
title: Настройка издателя OPC — Azure | Документация Майкрософт
description: В этой статье описывается, как настроить OPC Publisher, чтобы указать изменения данных узла OPC UA, события OPC UA для публикации, а также формат данных телеметрии.
author: dominicbetts
ms.author: dobett
ms.date: 06/10/2019
ms.topic: conceptual
ms.service: industrial-iot
services: iot-industrialiot
manager: philmea
ms.custom: devx-track-csharp
ms.openlocfilehash: 405fa04a3b6af7f0c87888aa48aba9bdffeaf282
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105646446"
---
# <a name="configure-opc-publisher"></a>Настройка издателя OPC

> [!IMPORTANT]
> Актуальную информацию по этой теме см. в статье [Промышленный Интернет вещей в Azure](https://azure.github.io/Industrial-IoT/).

Вы можете настроить издатель OPC для указания.

- Изменения данных узла OPC UA для публикации.
- События OPC UA для публикации.
- Формат данных телеметрии.

Вы можете настроить издатель OPC с помощью файлов конфигурации или с помощью вызовов методов.

## <a name="use-configuration-files"></a>Использование файлов конфигурации

В этом разделе описаны параметры для настройки публикации узла OPC UA с помощью файлов конфигурации.

### <a name="use-a-configuration-file-to-configure-publishing-data-changes"></a>Использование файла конфигурации для настройки публикации данных об изменениях

Самый простой способ настройки узлов OPC UA для публикации — с помощью файла конфигурации. Формат файлов конфигурации документирован в [publishednodes.json](https://github.com/Azure/iot-edge-opc-publisher/blob/master/opcpublisher/publishednodes.json) в репозитории.

Синтаксис файла конфигурации со временем изменялся. Издатель OPC по-прежнему считывает старые форматы, но для сохранения конфигурации преобразует их в формат последней версии.

В примере показан формат файла конфигурации.

```json
[
  {
    "EndpointUrl": "opc.tcp://testserver:62541/Quickstarts/ReferenceServer",
    "UseSecurity": true,
    "OpcNodes": [
      {
        "Id": "i=2258",
        "OpcSamplingInterval": 2000,
        "OpcPublishingInterval": 5000,
        "DisplayName": "Current time"
      }
    ]
  }
]
```

### <a name="use-a-configuration-file-to-configure-publishing-events"></a>Используйте файл конфигурации для настройки публикации данных об изменениях

Чтобы опубликовать события OPC UA, используйте один и тот же файл конфигурации, что и для изменения данных.

В следующем примере показано, как настроить публикацию для событий, созданных [сервером SimpleEvents ](https://github.com/OPCFoundation/UA-.NETStandard-Samples/tree/master/Workshop/SimpleEvents/Server). Сервер SimpleEvents находится в [репозитории OPC Foundation](https://github.com/OPCFoundation/UA-.NETStandard-Samples).

```json
[
  {
    "EndpointUrl": "opc.tcp://testserver:62563/Quickstarts/SimpleEventsServer",
    "OpcEvents": [
      {
        "Id": "i=2253",
        "DisplayName": "SimpleEventServerEvents",
        "SelectClauses": [
          {
            "TypeId": "i=2041",
            "BrowsePaths": [
              "EventId"
            ]
          },
          {
            "TypeId": "i=2041",
            "BrowsePaths": [
              "Message"
            ]
          },
          {
            "TypeId": "nsu=http://opcfoundation.org/Quickstarts/SimpleEvents;i=235",
            "BrowsePaths": [
              "/2:CycleId"
            ]
          },
          {
            "TypeId": "nsu=http://opcfoundation.org/Quickstarts/SimpleEvents;i=235",
            "BrowsePaths": [
              "/2:CurrentStep"
            ]
          }
        ],
        "WhereClause": [
          {
            "Operator": "OfType",
            "Operands": [
              {
                "Literal": "ns=2;i=235"
              }
            ]
          }
        ]
      }
    ]
  }
]
```

## <a name="use-method-calls"></a>Использование вызовов методов

В этом разделе описываются вызовы методов, которые можно использовать для настройки издателя OPC.

### <a name="configure-using-opc-ua-method-calls"></a>Настройка с помощью вызовов методов OPC UA

Издатель OPC включает в себя сервер OPC UA, доступ к которому осуществляется через порт 62222. Если имя узла **publisher**, то конечная точка URI — `opc.tcp://publisher:62222/UA/Publisher`.

Эта конечная точка предоставляет следующие четыре метода.

- PublishNode
- UnpublishNode
- GetPublishedNodes
- IoT HubDirectMethod

### <a name="configure-using-iot-hub-direct-method-calls"></a>Настройка с помощью прямых вызовов методов центра Интернета вещей

Издатель OPC реализует следующие прямые вызовы методов центра Интернета вещей.

- PublishNodes
- UnpublishNodes
- UnpublishAllNodes
- GetConfiguredEndpoints
- GetConfiguredNodesOnEndpoint
- GetDiagnosticInfo
- GetDiagnosticLog
- GetDiagnosticStartupLog
- ExitApplication
- GetInfo

Формат полезных данных JSON для запроса метода и ответов определяется в файле [opcpublisher/HubMethodModel.cs](https://github.com/Azure/iot-edge-opc-publisher/tree/master/opcpublisher).

При вызове неизвестного метода в модуле в ответ выдается строка о том, что метод не реализован. Неизвестный метод можно вызвать, чтобы проверить связь с модулем.

### <a name="configure-username-and-password-for-authentication"></a>Настройка имени пользователя и пароля для проверки подлинности

Режим аутентификации можно установить с помощью прямого вызова методов центра Интернета вещей. Полезные данные должны содержать свойство **OpcAuthenticationMode**, имя пользователя и пароль.

```csharp
{
    "EndpointUrl": "<Url of the endpoint to set authentication settings>",
    "OpcAuthenticationMode": "UsernamePassword",
    "Username": "<Username>",
    "Password": "<Password>"
    ...
}
```

Пароль зашифрован с помощью клиента рабочей нагрузки центра Интернета вещей и хранится в конфигурации издателя. Чтобы изменить аутентификацию обратно на анонимную, используйте метод со следующими полезными данными.

```csharp
{
    "EndpointUrl": "<Url of the endpoint to set authentication settings>",
    "OpcAuthenticationMode": "Anonymous"
    ...
}
```

Если свойство **OpcAuthenticationMode** не задано в полезных данных, параметры аутентификации в конфигурации остаются неизменными.

## <a name="configure-telemetry-publishing"></a>Настройка публикации данных телеметрии

Когда издатель OPC получает уведомление об изменении значения в опубликованном узле, он создает форматированное сообщение JSON, которое передается в центр Интернета вещей.

Вы можете настроить содержимое этого форматированного сообщения JSON, с помощью файла конфигурации. Если файл конфигурации не указан с параметром `--tc`, то по умолчанию будет использоваться конфигурация, совместимая с [акселератором решений подключенной фабрики](https://github.com/Azure/azure-iot-connected-factory).

Если издатель OPC настроен на пакетные сообщения, они передаются как допустимый массив JSON.

Данные телеметрии является получаются из следующих источников.

- Конфигурация узла издателя OPC для узла
- Объект **MonitoredItem** стека OPC UA, для которого издатель OPC получил уведомление.
- Передаваемый этому уведомлению аргумент, который предоставляет подробную информацию об изменении значения данных.

Данные телеметрии, поступающие в форматированное сообщение JSON, представляют собой набор важных свойств этих объектов. Если вам требуется больше свойств, необходимо изменить базовый код издателя OPC.

Ниже приведен синтаксис файла конфигурации:

```json
// The configuration settings file consists of two objects:
// 1) The 'Defaults' object, which defines defaults for the telemetry configuration
// 2) An array 'EndpointSpecific' of endpoint specific configuration
// Both objects are optional and if they are not specified, then publisher uses
// its internal default configuration, which generates telemetry messages compatible
// with the Microsoft Connected factory Preconfigured Solution (https://github.com/Azure/azure-iot-connected-factory).

// A JSON telemetry message for Connected factory looks like:
//  {
//      "NodeId": "i=2058",
//      "ApplicationUri": "urn:myopcserver",
//      "DisplayName": "CurrentTime",
//      "Value": {
//          "Value": "10.11.2017 14:03:17",
//          "SourceTimestamp": "2017-11-10T14:03:17Z"
//      }
//  }

// The 'Defaults' object in the sample below, are similar to what publisher is
// using as its internal default telemetry configuration.
{
    "Defaults": {
        // The first two properties ('EndpointUrl' and 'NodeId' are configuring data
        // taken from the OpcPublisher node configuration.
        "EndpointUrl": {

            // The following three properties can be used to configure the 'EndpointUrl'
            // property in the JSON message send by publisher to IoT Hub.

            // Publish controls if the property should be part of the JSON message at all.
            "Publish": false,

            // Pattern is a regular expression, which is applied to the actual value of the
            // property (here 'EndpointUrl').
            // If this key is omitted (which is the default), then no regex matching is done
            // at all, which improves performance.
            // If the key is used you need to define groups in the regular expression.
            // Publisher applies the regular expression and then concatenates all groups
            // found and use the resulting string as the value in the JSON message to
            //sent to IoT Hub.
            // This example mimics the default behaviour and defines a group,
            // which matches the conplete value:
            "Pattern": "(.*)",
            // Here some more exaples for 'Pattern' values and the generated result:
            // "Pattern": "i=(.*)"
            // defined for Defaults.NodeId.Pattern, will generate for the above sample
            // a 'NodeId' value of '2058'to be sent by publisher
            // "Pattern": "(i)=(.*)"
            // defined for Defaults.NodeId.Pattern, will generate for the above sample
            // a 'NodeId' value of 'i2058' to be sent by publisher

            // Name allows you to use a shorter string as property name in the JSON message
            // sent by publisher. By default the property name is unchanged and will be
            // here 'EndpointUrl'.
            // The 'Name' property can only be set in the 'Defaults' object to ensure
            // all messages from publisher sent to IoT Hub have a similar layout.
            "Name": "EndpointUrl"

        },
        "NodeId": {
            "Publish": true,

            // If you set Defaults.NodeId.Name to "ni", then the "NodeId" key/value pair
            // (from the above example) will change to:
            //      "ni": "i=2058",
            "Name": "NodeId"
        },

        // The MonitoredItem object is configuring the data taken from the MonitoredItem
        // OPC UA object for published nodes.
        "MonitoredItem": {

            // If you set the Defaults.MonitoredItem.Flat to 'false', then a
            // 'MonitoredItem' object will appear, which contains 'ApplicationUri'
            // and 'DisplayNode' proerties:
            //      "NodeId": "i=2058",
            //      "MonitoredItem": {
            //          "ApplicationUri": "urn:myopcserver",
            //          "DisplayName": "CurrentTime",
            //      }
            // The 'Flat' property can only be used in the 'MonitoredItem' and
            // 'Value' objects of the 'Defaults' object and will be used
            // for all JSON messages sent by publisher.
            "Flat": true,

            "ApplicationUri": {
                "Publish": true,
                "Name": "ApplicationUri"
            },
            "DisplayName": {
                "Publish": true,
                "Name": "DisplayName"
            }
        },
        // The Value object is configuring the properties taken from the event object
        // the OPC UA stack provided in the value change notification event.
        "Value": {
            // If you set the Defaults.Value.Flat to 'true', then the 'Value'
            // object will disappear completely and the 'Value' and 'SourceTimestamp'
            // members won't be nested:
            //      "DisplayName": "CurrentTime",
            //      "Value": "10.11.2017 14:03:17",
            //      "SourceTimestamp": "2017-11-10T14:03:17Z"
            // The 'Flat' property can only be used for the 'MonitoredItem' and 'Value'
            // objects of the 'Defaults' object and will be used for all
            // messages sent by publisher.
            "Flat": false,

            "Value": {
                "Publish": true,
                "Name": "Value"
            },
            "SourceTimestamp": {
                "Publish": true,
                "Name": "SourceTimestamp"
            },
            // 'StatusCode' is the 32 bit OPC UA status code
            "StatusCode": {
                "Publish": false,
                "Name": "StatusCode"
                // 'Pattern' is ignored for the 'StatusCode' value
            },
            // 'Status' is the symbolic name of 'StatusCode'
            "Status": {
                "Publish": false,
                "Name": "Status"
            }
        }
    },

    // The next object allows to configure 'Publish' and 'Pattern' for specific
    // endpoint URLs. Those will overwrite the ones specified in the 'Defaults' object
    // or the defaults used by publisher.
    // It is not allowed to specify 'Name' and 'Flat' properties in this object.
    "EndpointSpecific": [
        // The following shows how a endpoint specific configuration can look like:
        {
            // 'ForEndpointUrl' allows to configure for which OPC UA server this
            // object applies and is a required property for all objects in the
            // 'EndpointSpecific' array.
            // The value of 'ForEndpointUrl' must be an 'EndpointUrl' configured in
            // the publishednodes.json confguration file.
            "ForEndpointUrl": "opc.tcp://<your_opcua_server>:<your_opcua_server_port>/<your_opcua_server_path>",
            "EndpointUrl": {
                // We overwrite the default behaviour and publish the
                // endpoint URL in this case.
                "Publish": true,
                // We are only interested in the URL part following the 'opc.tcp://' prefix
                // and define a group matching this.
                "Pattern": "opc.tcp://(.*)"
            },
            "NodeId": {
                // We are not interested in the configured 'NodeId' value, 
                // so we do not publish it.
                "Publish": false
                // No 'Pattern' key is specified here, so the 'NodeId' value will be
                // taken as specified in the publishednodes configuration file.
            },
            "MonitoredItem": {
                "ApplicationUri": {
                    // We already publish the endpoint URL, so we do not want
                    // the ApplicationUri of the MonitoredItem to be published.
                    "Publish": false
                },
                "DisplayName": {
                    "Publish": true
                }
            },
            "Value": {
                "Value": {
                    // The value of the node is important for us, everything else we
                    // are not interested in to keep the data ingest as small as possible.
                    "Publish": true
                },
                "SourceTimestamp": {
                    "Publish": false
                },
                "StatusCode": {
                    "Publish": false
                },
                "Status": {
                    "Publish": false
                }
            }
        }
    ]
}
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь вы узнали, как настроить издатель OPC. Мы предлагаем далее ознакомиться со статьей [Запуск издателя OPC](howto-opc-publisher-run.md).
