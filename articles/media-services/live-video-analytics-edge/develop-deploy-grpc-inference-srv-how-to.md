---
title: Разработка и развертывание сервера вывода gRPC — Azure
description: В этой статье содержатся рекомендации по разработке и развертыванию сервера вывода gRPC.
ms.topic: how-to
ms.date: 12/02/2020
ms.openlocfilehash: 6184a369e73c26d3a8a716f9daf1c0420a5239fe
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98881658"
---
# <a name="how-to-guide--develop-and-deploy-a-grpc-inference-server"></a>Руководство по разработке и развертыванию сервера вывода gRPC

## <a name="overview"></a>Обзор

В этой статье показано, как можно заключить модели ИСКУССТВЕНного выбора на сервер вывода gRPC, чтобы его можно было интегрировать с помощью модуля Live Video Analytics (лва) через расширение Graph. 

## <a name="suggested-pre-reading"></a>Рекомендуемые материалы для предварительного ознакомления

* [Расширения графа мультимедиа](media-graph-extension-concept.md)
* [Протокол расширения gRPC](grpc-extension-protocol.md)
* [Схема метаданных вывода](inference-metadata-schema.md)
* [Введение в gRPC](https://www.grpc.io/docs/what-is-grpc/introduction/)
* [proto3 язык](https://developers.google.com/protocol-buffers/docs/proto3)

## <a name="prerequisites"></a>Предварительные условия

* Устройство с архитектурой x86-64 или ARM64, работающее под управлением одной из [поддерживаемых операционных систем Linux](../../iot-edge/support.md#operating-systems) или компьютера Windows.
* [Установите DOCKER](https://docs.docker.com/desktop/#download-and-install) на компьютере.
* Установите [среду выполнения IOT Edge](../../iot-edge/how-to-install-iot-edge.md?tabs=linux).

## <a name="grpc-implementation-steps"></a>шаги реализации gRPC

Чтобы создать сервер вывода gRPC и реализовать его как расширение с помощью Live Video Analytics, необходимо выполнить следующие действия:

### <a name="setup"></a>Настройка:

Выполните действия, необходимые для [развертывания и работы модуля Live Video Analytics на IOT Edge устройстве](deploy-iot-edge-device.md).

### <a name="high-level-implementation-steps"></a>Этапы реализации высокого уровня:

1. Выберите один из множества языков, поддерживаемых gRPC: C#, C++, DART, Go, Java, Node, цели-C, PHP, Python, Ruby.
1. Реализуйте сервер gRPC, который будет взаимодействовать с видео-аналитикой в реальном времени с помощью [файлов proto3](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc).

    :::image type="content" source="./media/develop-deploy-grpc-inference-srv-how-to/inference-srv-container-process.png" alt-text="Сервер gRPC, который будет обмениваться данными с Video Analytics с помощью файлов proto3":::

    В этой службе:
    1. Обработайте обмен сообщениями описания сеанса между сервером и клиентом.
    1. Обрабатывайте [примеры сообщений мультимедиа](https://github.com/Azure/live-video-analytics/blob/master/contracts/grpc/extension.proto) и возвращают результаты.

        1. Вызов обработчика вызова, который использует обученную модель для формирования вывода входящих сообщений.
        1. Получение результатов из ядра, упаковка их обратно в качестве примера мультимедиа и обратная передача в Live Video Analytics с помощью файла с [расширением](https://github.com/Azure/live-video-analytics/blob/master/contracts/grpc/inferencing.proto) имени.

            Кроме того, можно вызвать любую функцию преобразования мультимедиа для примера мультимедиа.
1. Развертывание реализации сервера gRPC. Это делается двумя способами:

    1. Развертывание в виде модуля IoT, размещенного с помощью модуля Live Video Analytics
    1. Разверните как модуль IoT в доступном для сети узле (локально или в облаке), который может обмениваться данными с модулем Live Video Analytics.
1. Настройте топологию Graph в реальном времени с помощью модуля Live Video Analytics и наведите его на сервер gRPC.

### <a name="recommendation"></a>Рекомендация.

При совместном размещении на том же узле можно использовать общую память для лучшей производительности. Для этого необходимо использовать возможности общей памяти Linux, предоставляемые языком программирования или средой.

1. Откройте маркер общей памяти Linux.
1. При получении кадра получите доступ к смещению адреса в общей памяти.
1. Подтвердите завершение обработки кадра, чтобы его память можно было освободить в службе Live Video Analytics.

## <a name="create-a-grpc-inference-server"></a>Создание сервера вывода gRPC

Теперь вы создадите модуль IoT Edge (внешний AI), который принимает видеоматериалы из Live Video Analytics с помощью сообщений [protobuf](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc) через общую память, классификация кадров как "темного" или "светлого" и возврат результатов вывода в приемник сообщений центра Интернета вещей в службе Live Video Analytics с помощью [схемы метаданных вывода](inference-metadata-schema.md).

:::image type="content" source="./media/develop-deploy-grpc-inference-srv-how-to/external-ai.png" alt-text="Создание модуля IoT Edge (External AI)":::

Этот сервер вывода gRPC — это консольное приложение .NET Core, которое обрабатывает сообщения [protobuf](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc) , передаваемые между Live Video Analytics и настраиваемым AI. Ниже приведен поток сообщений между динамическим видео-аналитикой и сервером вывода gRPC.

1. Live Video Analytics отправляет дескриптор потока мультимедиа (см [. extension.](https://github.com/Azure/live-video-analytics/blob/master/contracts/grpc/extension.proto)идентификатор), который определяет сведения о потоке мультимедиа, которые будут отправлены, за которыми следуют кадры видео, на сервер в виде сообщения [protobuf](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc) в сеансе потока gRPC. 
1. Сервер проверяет и подтверждает дескриптор потока и настраивает требуемый метод передачи данных.
1. После этого интерактивная аналитика видео начнет отправлять файлы Медиасампле, содержащие кадры видео.
1. Сервер анализирует видеоматериалы по мере их получения и начинает их обработку с помощью определенного обработчика изображений.
1. Затем сервер возвращает результаты вывода как [protobuf](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc) сообщения, как только они станут доступны. 

    :::image type="content" source="./media/develop-deploy-grpc-inference-srv-how-to/grpc-external-srv.png" alt-text="Создание сервера вывода gRPC":::

Видеокадры можно передавать с помощью [общей памяти](https://en.wikipedia.org/wiki/Shared_memory#:~:text=In%20computer%20science%2C%20shared%20memory,of%20passing%20data%20between%20programs.) , или они могут быть внедрены в сообщение protobuf. Режим передачи данных можно настроить в топологии графа лва, чтобы определить, как будут передаваться кадры. Это достигается путем настройки элемента **данных** свойства медиаграфгрпцекстенсион, как показано ниже.

Внедренное.

```
"dataTransfer": {
              "mode": "Embedded"
            }
```

Общая память:

```
"dataTransfer": {
              "mode": "sharedMemory",
              "SharedMemorySizeMiB": "20"
            }
```

> [!NOTE]
> При обмене данными через общую память значение Ипкмоде должно быть **совместно** используемым, а в модуле сервера gRPC установите значение ипкмоде в **container: {CONTAINER_NAME}**. Эти параметры должны быть сделаны в файле манифеста развертывания, который используется для развертывания модулей в центре Интернета вещей Azure. Ниже приведен пример параметров контейнера, используемых при настройке модулей IoT Edge.

Модуль Live Video Analytics:

```
{
    "HostConfig": {
        "LogConfig": {
            "Config": {
                "max-size": "10m",
                "max-file": "10"
            }
        },
        "IpcMode": "shareable"
    }
}
```

Модуль расширения gRPC:

```
{
    "HostConfig": {
        "LogConfig": {
            "Config": {
                "max-size": "10m",
                "max-file": "10"
            }
        },
        "IpcMode": "container:lvaEdge"
    }
}
```

> [!NOTE]
> Убедитесь, что вы можете получить доступ к области общей памяти "Container: Лваедже" в Грпцекстенсион.

## <a name="sample-grpc-server"></a>Пример сервера gRPC

Чтобы понять, как разрабатывается сервер gRPC, давайте проверим наш пример кода.

1. Клонировать репозиторий из ссылки GitHub [https://github.com/Azure-Samples/live-video-analytics-iot-edge-csharp](https://github.com/Azure-Samples/live-video-analytics-iot-edge-csharp) .
1. Запустите VSCode и перейдите в папку/СРК/Едже/модулес/грпцекстенсион.
1. Давайте кратко пошаговым руководством по файлам:

    1. **Program. CS**: это точка входа приложения. Он отвечает за инициализацию и управление сервером gRPC, который будет использоваться в качестве узла. В нашем примере порт для прослушивания входящих сообщений gRPC от клиента gRPC (например, Live Video Analytics) on задается элементом конфигурации Грпкбиндингс в AppConfig.js.
    
        ```json    
        {
          "grpcBinding": "tcp://0.0.0.0:5001"
        }
        ```
    1. **Сервицес\медиаграфекстенсионсервице.КС**: Этот класс отвечает за обработку сообщений [protobuf](https://github.com/Azure/live-video-analytics/tree/master/contracts/grpc) . Он считывает кадр в сообщении, вызывает Имажепроцессор и записывает результаты вывода.
Теперь, когда мы настроили и инициализировал подключения к портам сервера gRPC, давайте посмотрим, как можно обрабатывать входящие сообщения gRPC.

        * После установки сеанса gRPC первое сообщение о том, что сервер gRPC получит от клиента (Live Video Analytics), является Медиастреамдескриптор, который определен в файле [Extension.](https://github.com/Azure/live-video-analytics/blob/master/contracts/grpc/extension.proto) имя. 

            ```
            message MediaStreamDescriptor {
              oneof stream_identifier {
                GraphIdentifier graph_identifier = 1;     // Media Stream graph identifier
                string extension_identifier = 2;          // Media Stream extension identifier
              }
              MediaDescriptor media_descriptor = 5;       // Session media information.
              // Additional data transfer properties. If none is set, it is assumed
              // that all content will be transferred through messages (embedded transfer).
              oneof data_transfer_properties {
                SharedMemoryBufferTransferProperties shared_memory_buffer_transfer_properties = 10;
              }
            }
            ```
        * В нашей серверной реализации метод `ProcessMediaStreamDescriptor` будет проверять свойство Медиастреамдескриптор медиадескриптор для видеофайла, а затем настраивает режим пересылки данных (который либо использует общую память, либо использовать режим пересылки встроенных кадров) в зависимости от того, что указано в топологии и используемый файл шаблона развертывания. 
        * После получения сообщения и успешной настройки режима передачи данных сервер gRPC возвращает клиенту сообщение о Медиастреамдескриптор, а значит, устанавливает соединение между сервером и клиентом.    
        * После того, как Live Video Analytics получит подтверждение, оно начнет передачу потока мультимедиа на сервер gRPC. В нашей реализации сервера метод `ProcessMediaStream` будет обрабатывать входящий медиастреаммессаже. Медиастреаммессаже также определяется в [расширении. имени](https://github.com/Azure/live-video-analytics/blob/master/contracts/grpc/extension.proto).

            ```
            message MediaStreamMessage {
              uint64 sequence_number = 1;       // Monotonically increasing directional message identifier starting from 1 when the gRPC connection is created
              uint64 ack_sequence_number = 2;   // 0 if this message is not referencing any sent message.
            
            
              // Possible payloads are strongly defined by the contract below
              oneof payload {
                MediaStreamDescriptor media_stream_descriptor = 5;
                MediaSample media_sample = 6;
              }
            }
            ```
        * В зависимости от значения batchSize в Appconfig.js, наш сервер продолжает получать сообщения и сохранит видеокадры в списке. После достижения предела batchSize функция вызывает функцию или файл, который будет обрабатывать изображение. В нашем случае метод вызывает файл с именем Батчимажепроцессор. cs.
    1. **Процессорс\батчимажепроцессор.КС**: Этот класс отвечает за обработку изображений. В этом примере использовалась модель классификации изображений. Для каждого изображения, которое будет обработано, используется следующий алгоритм:

        1. Преобразуйте изображение в массив байтов для обработки. См. метод: `GetBytes(Bitmap image)`
        
            Образец процессора, который мы используем, поддерживает только фрейм изображения в кодировке JPG и нет в формате пикселей. Если пользовательский процессор поддерживает другую кодировку или формат, обновите `IsMediaFormatSupported` метод класса процессора.
        1. С помощью [класса КолорМатрикс](/dotnet/api/system.drawing.imaging.colormatrix?preserve-view=true&view=dotnet-plat-ext-3.1)преобразуйте изображение в серый масштаб. См. метод: `ToGrayScale(Image source)` .
        1. После получения изображения серого масштаба вычисляется среднее значение серого масштаба в байтах.
        1. Если среднее значение < 127, то изображение классифицируется как "темное". в противном случае мы будем классифицировать их как "светлое", а значение достоверности — 1,0. См. метод: `ProcessImage(List<Image> images)` .

    Вы можете добавить собственную логику процессора, изменив этот класс или добавив новый класс и реализовав метод:

    ```
    IEnumerable<Inference> ProcessImage(List<Image> images) 
    ```

    После добавления нового класса необходимо обновить Медиаграфекстенсионсервице. cs, чтобы он создавал экземпляр класса и вызывал для него метод Процессимаже для выполнения логики обработки. 

## <a name="connect-with-live-video-analytics-module"></a>Подключение с помощью модуля Live Video Analytics

Теперь, когда вы создали модуль расширения gRPC, мы создадим и развернем топологию Media Graph.

1. С помощью Visual Studio Code выполните [эти инструкции](../../iot-edge/tutorial-develop-for-linux.md#build-and-push-your-solution), чтобы войти в Docker.
1. В Visual Studio Code перейдите в папку src/edge. Здесь находится созданный нами ENV-файл и несколько файлов шаблона развертывания.

    Шаблон развертывания содержит манифест развертывания для пограничного устройства. Он содержит некоторые значения-заполнители. В ENV-файле указаны значения таких переменных.
    
    Потом выберите Создание и отправка решения IoT Edge. Для этого шага используйте src/ребр/deployment.grpc.template.js.
        
    :::image type="content" source="./media/develop-deploy-grpc-inference-srv-how-to/build-push-iot-edge-solution.png" alt-text="Подключение с помощью модуля Live Video Analytics":::
    
    Это действие создает серверный модуль GRPC и отправляет образ в реестр контейнеров Azure.
    Убедитесь, что в файле .env определены переменные среды CONTAINER_REGISTRY_USERNAME_myacr и CONTAINER_REGISTRY_PASSWORD_myacr.
1. Теперь перейдите в папку src/cloud-to-device-console-app. Здесь вы увидите файл appSettings.json и несколько других файлов:

    * c2d-console-app.csproj — файл проекта Visual Studio Code.
    * operations.json — список операций, которые должна запускать программа.
    * Program.cs — пример кода программы. Этот код выполняет следующие действия:

        * загружает параметры приложения;
        * вызывает прямые методы, которые представляются модулем Аналитики видеотрансляций в IoT Edge (с помощью модуля можно анализировать потоки видеотрансляции, вызывая его [прямые методы](direct-methods.md));
        * приостанавливает выполнение, позволяя проанализировать выходные данные программы в окне ТЕРМИНАЛ и проверить сгенерированные модулем события в окне ВЫХОДНЫЕ ДАННЫЕ;
        * вызывает прямые методы для очистки ресурсов.
1. Внесите правки в файл operations.json:

    * Измените ссылку на топологию графа:

        * `"topologyUrl" : https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/grpcExtension/topology.json`
        * В разделе GraphInstanceSet измените имя топологии графа, чтобы оно совпадало со значением в предыдущей ссылке:<br/>`"topologyName": "InferencingWithGrpcExtension"`
        * В разделе GraphTopologyDelete измените имя:<br/>`"name": "InferencingWithGrpcExtension"`

            Топология (например, `https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/grpcExtension/topology.json` ) должна определять адрес расширения:
    * Параметр адреса расширения

        ```
        {
            "name": "grpcExtensionAddress",
            "type": "String",
            "description": "gRPC LVA Extension Address",
            "default": "https://<REPLACE-WITH-IP-OR-CONTAINER-NAME>/score"
        },
        ```
    * Конфигурация

        ```
        {
            "@apiVersion": "1.0",
            "name": "TopologyName",
            "properties": {
            "processors": [
              {
                "@type": "#Microsoft.Media.MediaGraphGrpcExtension",
                "name": "grpcExtension",
                "endpoint": {
                  "@type": "#Microsoft.Media.MediaGraphUnsecuredEndpoint",
                  "url": "${grpcExtensionAddress}",
                  "credentials": {
                    "@type": "#Microsoft.Media.MediaGraphUsernamePasswordCredentials",
                    "username": "${grpcExtensionUserName}",
                    "password": "${grpcExtensionPassword}"
                  }
                },
                "dataTransfer": {
                        "mode": "sharedMemory",
                        "SharedMemorySizeMiB": "5"
                },
                "image": {
                  "scale": {
                    "mode": "${imageScaleMode}",
                    "width": "${frameWidth}",
                    "height": "${frameHeight}"
                  },
                  "format": {
                    "@type": "#Microsoft.Media.MediaGraphImageFormatEncoded",
                    "encoding": "${imageEncoding}",
                    "quality": "${imageQuality}"
                  }
                }
            ]
          }
        }
        ```
    
## <a name="generate-and-deploy-the-iot-edge-deployment-manifest"></a>Создание и развертывание манифеста развертывания IoT Edge

Манифест развертывания определяет, какие именно модули развертываются на пограничном устройстве, а также содержит параметры конфигурации для этих модулей. Выполните перечисленные ниже действия, чтобы создать манифест на базе файла шаблона, а затем развернуть его на пограничном устройстве.
На этом шаге создается манифест развертывания IoT Edge по адресу src/ребр/config/deployment.grpc.amd64.jsв. Щелкните этот файл правой кнопкой мыши и выберите пункт **Создать развертывание для одного устройства**.

:::image type="content" source="./media/develop-deploy-grpc-inference-srv-how-to/create-deployment-single-device.png" alt-text="Создание и развертывание манифеста развертывания IoT Edge":::

Далее Visual Studio Code предложит выбрать устройство Центра Интернета вещей. Выберите свое устройство IoT Edge, которое должно быть устройством lva-sample-device.
На этом этапе начинается развертывание пограничных модулей на устройстве IoT Edge. В течении 30 секунд обновите центр Интернета вещей Azure в левом нижнем углу раздела Visual Studio Code. Вы увидите, что новый модуль был развернут с именем Лваекстенсион.

:::image type="content" source="./media/develop-deploy-grpc-inference-srv-how-to/devices.png" alt-text="Новый модуль был развернут с именем Лваекстенсион":::

## <a name="next-steps"></a>Дальнейшие действия

Выполните действия, описанные в статье **Подготовка к наблюдению за событиями** , приведенными в кратком руководстве [Анализ видео с помощью модели](use-your-model-quickstart.md) , чтобы запустить пример и интерпретировать результаты. Кроме того, ознакомьтесь с примерами топологий gRPC: [грпцекстенсион](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/grpcExtension/topology.json), [Кврвисгрпцекстенсион](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/cvr-with-grpcExtension/topology.json), [евртоассетсбигрпцекстенсион и [евронмотионплусгрпцекстенсион](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/motion-with-grpcExtension/topology.json).