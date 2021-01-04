---
title: Протокол расширения gRPC (Azure)
description: Из этой статьи вы узнаете об использовании протокола расширения gRPC для отправки сообщений между модулем службы "Аналитика видеотрансляций" и пользовательским расширением для искусственного интеллекта или компьютерного зрения.
ms.topic: overview
ms.date: 09/14/2020
ms.openlocfilehash: 7f21ff358b8dd5ac540de8c39c37c52e98977e59
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/14/2020
ms.locfileid: "97401633"
---
# <a name="grpc-extension-protocol"></a>Протокол расширения gRPC

Аналитика видеотрансляций в IoT Edge позволяет расширить возможности обработки графов мультимедиа с помощью [узла расширения графа](https://review.docs.microsoft.com/en-us/azure/media-services/live-video-analytics-edge/media-graph-extension-concept?branch=release-lva-dec-update). Если в качестве узла расширения используется обработчик расширений gRPC, обмен данными между модулем Аналитики видеотрансляций и модулем искусственного интеллекта или компьютерного зрения осуществляется по высокопроизводительному структурированному протоколу на базе gRPC.

Из этой статьи вы узнаете об использовании протокола расширения gRPC для отправки сообщений между модулем службы "Аналитика видеотрансляций" и пользовательским расширением для искусственного интеллекта или компьютерного зрения.

gRPC — это современная высокопроизводительная платформа RPC с открытым исходным кодом, которая работает в любой среде и поддерживает обмен данными для любых платформ и языков. Служба транспорта gRPC использует двунаправленную потоковую передачу HTTP/2 между:

* клиентом gRPC (службой "Аналитика видеотрансляций" в модуле IoT Edge) и 
* сервером gRPC (настраиваемым расширением).

Сеанс gRPC — это одно подключение клиента gRPC к серверу gRPC через порт TCP/TLS. 

В рамках одного сеанса происходит следующее. Клиент отправляет на сервер дескриптор потока мультимедиа, а затем видеокадры. Отправка осуществляется в виде сообщения [protobuf](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc) в сеансе потоковой передачи gRPC. Сервер проверяет дескриптор потока, анализирует видеокадр и возвращает результирующие выводы в виде сообщения protobuf. 

Настоятельно рекомендуется использовать для возврата ответов допустимые документы JSON, соответствующие предварительно установленной схеме, определенной согласно [объектной модели схемы метаданных выводов](https://review.docs.microsoft.com/en-us/azure/media-services/live-video-analytics-edge/inference-metadata-schema?branch=release-lva-dec-update). Это улучшит совместимость с другими компонентами и возможностями, которые могут быть добавлены в модуль Аналитики видеотрансляций в будущем.

![Контракт для расширения gRPC](./media/grpc-extension-protocol/grpc.png)

## <a name="implementing-grpc-protocol"></a>Реализация протокола gRPC

### <a name="creating-a-grpc-connection"></a>Создание подключения gRPC

Пользовательское расширение должно реализовать следующую службу gRPC:

```
service MediaGraphExtension
    {
        rpc ProcessMediaStream(stream MediaStreamMessage) returns (stream MediaStreamMessage);
    }
```

При вызове она будет открывать двунаправленный поток для обмена сообщениями между расширением gRPC и графом Аналитики видеотрансляций. Первое сообщение, отправленное в этом потоке каждой стороной, будет содержать дескриптор MediaStreamDescriptor, который определяет, какие сведения будут отправляться в следующих сообщениях MediaSample.

Например, расширение графа может отправить сообщение (представленное здесь в формате JSON) для информирования о том, что оно будет передавать пользовательскому расширению кадры 416x416 в кодировке Rgb24, внедренные в сообщения gRPC.

```
 {
    "sequence_number": 1,
    "ack_sequence_number": 0,
    "media_stream_descriptor": 
    {
        "graph_identifier": 
        {
            "media_services_arm_id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/resourceGroupName/providers/microsoft.media/mediaservices/mediaAccountName",
            "graph_instance_name": "mediaGraphName",
            "graph_node_name": "grpcExtension"
        },
        "media_descriptor": 
        {
            "timescale": 90000,
            "video_frame_sample_format": 
            {
                "encoding": "RAW",
                "pixel_format": "RGB24",
                "dimensions": 
                {
                    "width": 416,
                    "height": 416
                },
                "stride_bytes": 1248
            }
        }
    }
}
```

Пользовательское расширение в ответе отправит следующее сообщение о том, что оно возвращает только выводы.

```
{
    "sequence_number": 1,
    "ack_sequence_number": 1,
    "media_stream_descriptor": 
    {
        "extension_identifier": "customExtensionName"    
    }
}
```

Теперь, когда обе стороны обменялись дескрипторами мультимедиа, служба "Аналитика видеотрансляций" начнет передачу кадров в расширение.

> [!NOTE]
> Реализацию gRPC на стороне сервера можно выполнить на любом языке программирования.
### <a name="sequence-numbers"></a>Порядковые номера

Как узел расширения gRPC, так и пользовательское расширение сохраняют отдельные наборы порядковых номеров, назначенных их сообщениям. Эти порядковые номера должны единообразно увеличиваться, начиная с 1. `ack_sequence_number` можно пропустить при отсутствии подтвержденных сообщений, что возможно при отправке первого сообщения.

Запрос должен быть подтвержден после полной обработки соответствующего сообщения. В случае передачи через общую память это подтверждение означает, что отправитель может освободить общую память, а у получателя больше не будет к ней доступа. Отправитель должен хранить любое содержимое общей памяти, пока не будет получено подтверждение.

### <a name="reading-embedded-content"></a>Чтение внедренного содержимого

Внедренное содержимое может считываться непосредственно из сообщения `MediaSample` с помощью поля `content_byte`s. Данные будут кодироваться в соответствии с `MediaDescriptor`.

### <a name="reading-content-from-shared-memory"></a>Чтение содержимого из общей памяти

В случае передачи содержимого через общую память отправитель будет включать `SharedMemoryBufferTransferProperties` в свой дескриптор `MediaStreamDescriptor` при первом установлении сеанса. Это может выглядеть следующим образом (представленное в формате JSON).

```
{
  "handle_name": "inference_client_share_memory_2146989006636459346"
  "length_bytes": 20971520
}
```

Затем получатель откроет файл `/dev/shm/inference_client_share_memory_2146989006636459346`. Отправитель отправит сообщение `MediaSample`, содержащее элемент `ContentReference`, ссылающийся на определенное место в этом файле.

```
{
    "timestamp": 143598615750000,
    "content_reference": 
    {
        "address_offset": 519168,
        "length_bytes": 173056
    }
}
```

После этого получатель считает данные из этого расположения в файле.

Чтобы контейнер Аналитики видеотрансляций мог обмениваться данными через общую память, для него необходимо правильно настроить режим IPC. Это можно сделать множеством способов. Некоторые из рекомендуемых конфигураций приведены ниже.

* При обмене данными с подсистемой формирования выводов gRPC, запущенной на устройстве узла, в качестве режима IPC должен быть задан host.
* При обмене данными с сервером gRPC, запущенным в другом модуле IoT Edge, в качестве режима IPC следует задать shareable для модуля Аналитики видеотрансляций и `container:liveVideoAnalytics` для пользовательского расширения, где `liveVideoAnalytics` — имя модуля Аналитики видеотрансляций.

Ниже показано, как это может выглядеть на двойнике устройства при использовании первого из приведенных выше вариантов.

```
"liveVideoAnalytics": 
{
  "version": "1.0",
  "type": "docker",
  "status": "running",
  "restartPolicy": "always",
  "settings": 
  {
    "image": "mcr.microsoft.com/media/live-video-analytics:1",
    "createOptions": 
      "HostConfig": 
      {
        "IpcMode": "host"
      }
  }
}
```

Дополнительные сведения о режимах IPC см. по ссылке https://docs.docker.com/engine/reference/run/#ipc-settings---ipc.

## <a name="mediagraph-grpc-extension-contract-definitions"></a>Определения контракта расширения gRPC для графа мультимедиа

В этом разделе рассказывается о том, как определить контракт gRPC, задающий поток данных.

### <a name="protocol-messages"></a>Сообщения протокола

![Сообщения протокола](./media/grpc-extension-protocol/grpc2.png)
 
### <a name="client-authentication"></a>Проверка подлинности клиента

Пользователи, внедрившие пользовательские расширения, могут проверить подлинность входящих подключений gRPC и убедиться, что они поступают с узла расширения gRPC. Узел предоставит запись для проверки в заголовке запроса.

Для выполнения проверки можно использовать учетные данные (имя пользователя и пароль). При создании узла расширения gRPC учетные данные предоставляются примерно так, как показано ниже.

```
{
  "@type": "#Microsoft.Media.MediaGraphGrpcExtension",
  "name": "{moduleIdentifier}",
  "endpoint": 
  {
    "@type": "#Microsoft.Media.MediaGraphUnsecuredEndpoint",
    "url": "tcp://customExtension:8081",
    "credentials": 
    {
      "@type": "#Microsoft.Media.MediaGraphUsernamePasswordCredentials",
      "username": "username",
      "password": "password"
    }
  }
  // Other fields omitted
}
```

При отправке запроса gRPC в его метаданные включается приведенный ниже заголовок. Это напоминает обычную проверку подлинности HTTP.

`x-ms-authentication: Basic (Base64 Encoded username:password)`


## <a name="configuring-inference-server-for-each-mediagraph-over-grpc-extension"></a>Настройка сервера вывода для каждого графа мультимедиа с помощью расширения gRPC
При настройке сервера вывода не нужно предоставлять узел для каждой модели ИИ, упакованной на сервере вывода. Вместо этого для экземпляра графа можно использовать свойство `extensionConfiguration` узла `MediaGraphGrpcExtension` и определить способ выбора моделей ИИ. Во время выполнения LVA передаст эту строку серверу вывода, который может использовать ее для вызова требуемой модели ИИ. Это свойство `extensionConfiguration` является необязательным и зависит от сервера. Свойство можно использовать следующим образом:
```
{
  "@type": "#Microsoft.Media.MediaGraphGrpcExtension",
  "name": "{moduleIdentifier}",
  "endpoint": 
  {
    "@type": "#Microsoft.Media.MediaGraphUnsecuredEndpoint",
    "url": "${grpcExtensionAddress}",
    "credentials": 
    {
      "@type": "#Microsoft.Media.MediaGraphUsernamePasswordCredentials",
      "username": "${grpcExtensionUserName}",
      "password": "${grpcExtensionPassword}"
    }
  },
    // Optional server configuration string. This is server specific 
  "extensionConfiguration": "{Optional extension specific string}",
  "dataTransfer": 
  {
    "mode": "sharedMemory",
    "SharedMemorySizeMiB": "5"
  }
    //Other fields omitted
}
```

## <a name="using-grpc-over-tls"></a>Использование gRPC через TLS

Подключение gRPC, используемое для формирования выводов, может быть защищено с помощью протокола TLS. Это полезно в ситуациях, когда нет возможности гарантировать безопасность сети между службой "Аналитика видеотрансляций" и подсистемой формирования выводов. TLS шифрует все содержимое, встроенное в сообщения gRPC, что приводит к дополнительной нагрузке на ЦП при высокой скорости передачи кадров.

Параметры проверки IgnoreHostname и IgnoreSignature не поддерживаются gRPC, поэтому сертификат сервера, который представляет подсистема формирования выводов, должен содержать общее имя, совпадающее с IP-адресом или именем узла в URL-адресе конечной точки узла расширения gRPC.

## <a name="next-steps"></a>Дальнейшие шаги

[Ознакомьтесь со схемой метаданных выводов](inference-metadata-schema.md)
