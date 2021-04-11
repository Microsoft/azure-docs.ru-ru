---
title: Анализ видеотрансляций с помощью Компьютерного зрения для Пространственного анализа — Azure
description: В этом руководстве показано, как использовать функцию "Аналитика видеотрансляций" вместе с функцией ИИ "Пространственный анализ с помощью Компьютерного зрения" от Azure Cognitive Services для анализа веб-канала видеотрансляции с IP-камеры (имитация).
ms.topic: tutorial
ms.date: 09/08/2020
ms.openlocfilehash: 1c3fd411e60cabb504f01bece62d497ddeea31d8
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105561021"
---
# <a name="analyze-live-video-with-computer-vision-for-spatial-analysis-preview"></a>Анализ видеотрансляций с помощью Компьютерного зрения для Пространственного анализа (предварительная версия)

В этом руководстве показано, как использовать функцию "Аналитика видеотрансляций" со службой ИИ [Пространственный анализ с помощью Компьютерного зрения](https://azure.microsoft.com/services/cognitive-services/computer-vision/) от Azure Cognitive Services для анализа веб-канала видеотрансляции с IP-камеры (имитация). Вы увидите, как использовать этот сервер вывода, чтобы проанализировать видеопоток и понять пространственные отношения между людьми и движением в физическом пространстве.  На этот сервер вывода отправляют подмножество кадров в видеоканале, а в центр IoT Edge — результаты, и, если некоторые условия выполняются, видеоролики записываются и сохраняются в виде ресурсов Служб мультимедиа Azure.

Изучив данный учебник, вы научитесь:

> [!div class="checklist"]
> * Настраивать ресурсы.
> * Изучите код.
> * Запустите пример кода.
> * Отслеживать события.
 
[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]
  > [!NOTE]
  > Вам потребуется подписка Azure с разрешениями на создание субъектов-служб (такие разрешения предоставляет роль **Владелец**). Если у вас нет нужных разрешений, обратитесь к администратору учетной записи с просьбой предоставить вам нужные разрешения. 
## <a name="suggested-pre-reading"></a>Рекомендуемые материалы для предварительного ознакомления

Перед началом ознакомьтесь со следующими статьями:

* [Аналитика видеотрансляций в IoT Edge: обзор](overview.md)
* [Аналитика видеотрансляции в IoT Edge: терминология](terminology.md)
* [Граф мультимедиа: основные понятия](media-graph-concept.md)
* [Запись видео на основе событий](event-based-video-recording-concept.md)
* [Руководство. Разработка модуля IoT Edge](../../iot-edge/tutorial-develop-for-linux.md)
* [Развертывание Аналитики видеотрансляций в Azure Stack Edge](deploy-azure-stack-edge-how-to.md) 

## <a name="prerequisites"></a>Prerequisites

Ниже приведены предварительные требования для подключения модуля пространственного анализа к модулю Аналитики видеотрансляций.

* [Visual Studio Code](https://code.visualstudio.com/) на компьютере для разработки. Убедитесь, что у вас есть [расширение Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).
    * Убедитесь, что в сети, к которой подключен компьютер разработки, разрешен расширенный протокол управления очередью сообщений через порт 5671. Эта настройка позволяет Azure IoT Tools взаимодействовать с Центром Интернета вещей Azure.
* [Azure Stack Edge](https://azure.microsoft.com/products/azure-stack/edge/) с ускорением GPU.  
    Мы рекомендуем использовать Azure Stack Edge с ускорением GPU, однако при использовании [GPU NVIDIA Tesla T4](https://www.nvidia.com/en-us/data-center/tesla-t4/) контейнер также будет работать на любом другом устройстве. 
* [Контейнер Компьютерного зрения Azure Cognitive Service](https://azure.microsoft.com/services/cognitive-services/computer-vision/) для пространственного анализа.  
    Чтобы использовать этот контейнер, у вас должен быть ресурс Компьютерного зрения, с помощью которого можно получить связанный **ключ API** и **URI конечной точки**. Ключ API можно найти на страницах обзора и ключей Компьютерного зрения на портале Azure. Ключ и конечная точка необходимы, чтобы запустить контейнер.

## <a name="overview"></a>Обзор

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/spatial-analysis-tutorial/overview.png" alt-text="Обзор Пространственного анализа":::
 
На схеме показан порядок передачи сигналов в этом руководстве. [Пограничный модуль](https://github.com/Azure/live-video-analytics/tree/master/utilities/rtspsim-live555) имитирует IP-камеру, на которой находится RTSP-сервер. Узел [RTSP-источника](media-graph-concept.md#rtsp-source) получает видеосигнал с этого сервера и передает видеокадры на узел обработчика `MediaGraphCognitiveServicesVisionExtension`.

Узел MediaGraphCognitiveServicesVisionExtension выступает прокси-сервером. Он преобразует видеокадры в изображения указанного типа. Затем посредством **общей памяти** он передает изображение на другой пограничный модуль, выполняющий операции искусственного интеллекта на конечной точке gRPC. В этом примере такой пограничный модуль является модулем пространственного анализа. Узел процессора MediaGraphCognitiveServicesVisionExtension выполняет два действия:

* Собирает результаты и публикует события в узле [приемника центра Интернета вещей](media-graph-concept.md#iot-hub-message-sink). Затем узел отправляет эти события в [центр IoT Edge](../../iot-fundamentals/iot-glossary.md#iot-edge-hub). 
* Он также захватывает 30-секундный видеоролик из источника RTSP с помощью [процессора шлюза сигнала](media-graph-concept.md#signal-gate-processor) и сохраняет его как ресурс Служб мультимедиа.

## <a name="create-the-computer-vision-resource"></a>Создание ресурса Компьютерного зрения

Вам понадобится создать ресурс Azure для Компьютерного зрения либо на [портале Azure](../../iot-edge/how-to-deploy-modules-portal.md), либо с помощью Azure CLI. Такой ресурс можно создать после утверждения запроса на доступ к контейнеру и регистрации идентификатора подписки Azure. Перейдите по адресу https://aka.ms/csgate, чтобы предоставить свой вариант использования и идентификатор подписки Azure.  Ресурс Azure следует создавать, используя подписку Azure, которая была предоставлена в форме запроса на доступ.

### <a name="gathering-required-parameters"></a>Сбор обязательных параметров

Для всех контейнеров Cognitive Services, включая контейнер пространственного анализа, существует три основных необходимых параметра. Вам следует принять Лицензионное соглашение с конечным пользователем (EULA). Кроме того, вам понадобятся универсальный код ресурса (URI) конечной точки и ключ API.

### <a name="keys-and-endpoint-uri"></a>Ключи и URI конечной точки

Ключ используется для запуска контейнера пространственного анализа. Его можно найти на странице `Keys and Endpoint` соответствующего ресурса Cognitive Service на портале Azure. Перейдите на эту страницу и найдите ключи и URI конечной точки.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/spatial-analysis-tutorial/keys-endpoint.png" alt-text="URI конечной точки":::

## <a name="set-up-azure-stack-edge"></a>Настройка Azure Stack Edge

Выполните [эти шаги](../../databox-online/azure-stack-edge-gpu-deploy-prep.md), чтобы настроить Azure Stack Edge, и продолжайте следовать описанным далее действиям, чтобы развернуть модули Аналитики видеотрансляций и пространственного анализа.

## <a name="set-up-your-development-environment"></a>Настройка среды разработки

1. Клонируйте репозиторий из этого расположения: https://github.com/Azure-Samples/live-video-analytics-iot-edge-csharp.
1. В Visual Studio Code откройте папку, в которую вы скачали репозиторий.
1. В Visual Studio Code перейдите в папку src/cloud-to-device-console-app. Создайте там файл и назовите его appsettings.json. Этот файл содержит параметры, необходимые для выполнения программы.
1. Получите IotHubConnectionString из Azure Stack Edge, выполнив следующие действия.

    * Перейдите в центр Интернета вещей на портале Azure и щелкните `Shared access policies` в левой области навигации.
    * Щелкните `iothubowner`, чтобы получить ключи общего доступа.
    * Скопируйте значение `Connection String – primary key` и вставьте его в поле ввода в VSCode.
    
        Строка подключения будет выглядеть следующим образом: <br/>`HostName=xxx.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=xxx`
1. Скопируйте приведенное ниже содержимое в файл. Обязательно замените переменные.
    
    ```json
    {
        "IoThubConnectionString" : "HostName=<IoTHubName>.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=<SharedAccessKey>",
        "deviceId" : "<your Azure Stack Edge name>",
        "moduleId" : "lvaEdge"
    } 
    ```
1. Перейдите в папку src/edge и создайте файл с именем .env.
1. Скопируйте содержимое файла /clouddrive/lva-sample/edge-deployment/.env. Текст должен выглядеть как следующий код:

    ```env
    SUBSCRIPTION_ID="<Subscription ID>"  
    RESOURCE_GROUP="<Resource Group>"  
    AMS_ACCOUNT="<AMS Account ID>"  
    IOTHUB_CONNECTION_STRING="HostName=xxx.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=xxx"  
    AAD_TENANT_ID="<AAD Tenant ID>"  
    AAD_SERVICE_PRINCIPAL_ID="<AAD SERVICE_PRINCIPAL ID>"  
    AAD_SERVICE_PRINCIPAL_SECRET="<AAD SERVICE_PRINCIPAL ID>"  
    VIDEO_INPUT_FOLDER_ON_DEVICE="/home/lvaedgeuser/samples/input"  
    VIDEO_OUTPUT_FOLDER_ON_DEVICE="/var/media"
    APPDATA_FOLDER_ON_DEVICE="/var/local/mediaservices"
    CONTAINER_REGISTRY_USERNAME_myacr="<your container registry username>"  
    CONTAINER_REGISTRY_PASSWORD_myacr="<your container registry password>"   
    ```
    
## <a name="set-up-deployment-template"></a>Настройка шаблона развертывания  

Найдите файл шаблона развертывания /src/edge/deployment.spatialAnalysis.template.json. В шаблоне есть модуль lvaEdge, модуль rtspsim и наш модуль пространственного анализа.

В файле шаблона развертывания следует уделить внимание нескольким моментам.

1. Задайте привязки портов.
    
    ```
    "PortBindings": {
        "50051/tcp": [
            {
                "HostPort": "50051"
            }
        ]
    },
    ```
1. `IpcMode` в lvaEdge и модуль пространственного анализа createOptions должны быть одинаковыми и иметь значение "host".
1. Убедитесь, что вы настроили границы тома, поскольку это влияет на работу симулятора RTSP. Дополнительные сведения см. в разделе [Настройка подключений тома Docker](deploy-azure-stack-edge-how-to.md#optional-setup-docker-volume-mounts).

    1. [Подключитесь к общему ресурсу SMB](../../databox-online/azure-stack-edge-deploy-add-shares.md#connect-to-an-smb-share) и скопируйте [пример видеофайла bulldozer.mkv](https://lvamedia.blob.core.windows.net/public/bulldozer.mkv) в локальную общую папку.  
        > [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4Mesi]  
    1. Проверьте, содержит ли модуль rtspsim следующую конфигурацию:
        ```
        "createOptions": {
                            "HostConfig": {
                              "Mounts": [
                                {
                                  "Target": "/live/mediaServer/media",
                                  "Source": "<your Local Docker Volume Mount name>",
                                  "Type": "volume"
                                }
                              ],
                              "PortBindings": {
                                "554/tcp": [
                                  {
                                    "HostPort": "554"
                                  }
                                ]
                              }
                            }
                          }
        ```
        

## <a name="generate-and-deploy-the-deployment-manifest"></a>Создание и развертывание манифеста развертывания

Манифест развертывания определяет, какие именно модули развертываются на пограничном устройстве. Кроме того, он определяет параметры конфигурации для этих модулей.

Выполните перечисленные ниже действия, чтобы создать манифест на базе файла шаблона, а затем развернуть его на пограничном устройстве.

1. Откройте Visual Studio Code.
1. Рядом с областью ЦЕНТР ИНТЕРНЕТА ВЕЩЕЙ AZURE выберите значок Дополнительные действия, чтобы задать строку подключения к Центру Интернета вещей. Строку можно скопировать из файла `src/cloud-to-device-console-app/appsettings.json`.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/spatial-analysis-tutorial/connection-string.png" alt-text="Пространственный анализ: строка подключения":::
1. Щелкните правой кнопкой мыши `src/edge/deployment.spatialAnalysis.template.json` и выберите "Generate IoT Edge Deployment Manifest" (Создать манифест развертывания IoT Edge).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/spatial-analysis-tutorial/deployment-template-json.png" alt-text="Пространственный анализ: deployment.amd64.json":::
    
    В результате этого действия в папке src/edge/config должен быть создан файл манифеста под названием deployment.amd64.json.
1. Щелкните правой кнопкой мыши `src/edge/config/deployment.spatialAnalysis.amd64.json`, а затем выберите "Create Deployment for Single Device" (Создать развертывание для одного устройства). Выберите имя пограничного устройства.
    
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/spatial-analysis-tutorial/deployment-amd64-json.png" alt-text="Пространственный анализ: шаблон развертывания json":::   
1. Вам будет предложено выбрать устройство Центра Интернета вещей. Выберите имя вашего устройства Azure Stack Edge из раскрывающегося меню.
1. Примерно через 30 секунд обновите содержимое Центра Интернета вещей в левом нижнем углу окна. Теперь в пограничном устройстве должны отображаться следующие развернутые модули:
    
    * Аналитика видеотрансляций в IoT Edge (модуль lvaEdge).
    * Симулятор протокола RTSP (модуль rtspsim).
    * Пространственный анализ (модуль spatialAnalysis).
    
При успешном развертывании в выводе будет содержаться следующее сообщение:

```
[Edge] Start deployment to device [<Azure Stack Edge name>]
[Edge] Deployment succeeded.
```

После этого вы сможете найти модули `lvaEdge`, `rtspsim`, `spatialAnalysis` и `rtspsim` в разделе Устройства/Модули. Их состояние должно быть "Выполняется".

## <a name="prepare-to-monitor-events"></a>Подготовка к мониторингу событий

Чтобы просмотреть эти события, выполните указанные ниже действия.

1. В Visual Studio Code откройте вкладку **Расширения** (или нажмите клавиши CTRL+SHIFT+X) и найдите центр Интернета вещей Azure.
1. Щелкните правой кнопкой мыши и выберите **Параметры расширения**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/extensions-tab.png" alt-text="Параметры расширения":::
1. Найдите и включите параметр "Show Verbose Message" (Показывать подробное сообщение).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/show-verbose-message.png" alt-text="Show Verbose Message"::: (Показывать подробное сообщение)
1. Откройте область обозревателя и найдите Центр Интернета вещей Azure в левом нижнем углу.
1. Разверните узел Devices ("Устройства").
1. Щелкните правой кнопкой мыши Azure Stack Edge и выберите "Запуск мониторинга встроенной конечной точки события".
    
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/spatial-analysis-tutorial/start-monitoring.png" alt-text="Пространственный анализ: запуск мониторинга":::
     
## <a name="run-the-program"></a>Запуск программы

Код примера программы program.cs будет вызывать прямые методы в файле src/cloud-to-device-console-app/operations.json. Нам нужно настроить operations.json и предоставить топологию для использования в графе мультимедиа.  

В operations.json:

* Задайте топологию следующим образом:

```json
{
    "opName": "GraphTopologySet",
    "opParams": {
        "topologyUrl": "https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/lva-spatial-analysis/2.0/topology.json"
    }
},
```

* Создайте экземпляр graph, как показано ниже, задайте параметры в топологии:

```json
{
    "opName": "GraphInstanceSet",
    "opParams": {
        "name": "Sample-Graph-1",
        "properties": {
            "topologyName": "InferencingWithCVExtension",
            "description": "Sample graph description",
            "parameters": [
                {
                    "name": "rtspUrl",
                    "value": " rtsp://rtspsim:554/media/bulldozer.mkv"
                },
                {
                    "name": "rtspUserName",
                    "value": "testuser"
                },
                {
                    "name": "rtspPassword",
                    "value": "testpassword"
                }
            ]
        }
    }
},
```

>[!Note]
Проверьте использование MediaGraphRealTimeComputerVisionExtension для подключения модуля пространственного анализа. Задайте для ${grpcUrl} значение **tcp://spatialAnalysis:<номер_порта>** , т. е. tcp://spatialAnalysis:50051

```json
{
    "@type": "#Microsoft.Media.MediaGraphCognitiveServicesVisionExtension",
    "name": "computerVisionExtension",
    "endpoint": {
        "@type": "#Microsoft.Media.MediaGraphUnsecuredEndpoint",
        "url": "${grpcUrl}",
        "credentials": {
            "@type": "#Microsoft.Media.MediaGraphUsernamePasswordCredentials",
            "username": "${spatialanalysisusername}",
            "password": "${spatialanalysispassword}"
        }
    },
    "image": {
        "scale": {
            "mode": "pad",
            "width": "1408",
            "height": "786"
        },
        "format": {
            "@type": "#Microsoft.Media.MediaGraphImageFormatRaw",
            "pixelFormat": "bgr24"
        }
    },
    "samplingOptions": {
        "skipSamplesWithoutAnnotation": "false",
        "maximumSamplesPerSecond": "20"
    },
    "inputs": [
        {
            "nodeName": "rtspSource",
            "outputSelectors": [
                {
                    "property": "mediaType",
                    "operator": "is",
                    "value": "video"
                }
            ]
        }
    ]
}
```

Запустите сеанс отладки и следуйте инструкциям в окне **Терминал**. В этом сеансе будет задана топология, установлен и активирован экземпляр, а также, в конечном итоге, будут удалены ресурсы.

## <a name="interpret-results"></a>Интерпретация результатов

При создании экземпляра графа мультимедиа вы увидите событие "MediaSessionEstablished", здесь [пример события MediaSessionEstablished](detect-motion-emit-events-quickstart.md#mediasessionestablished-event).

Модуль пространственного анализа также будет отправлять события Аналитики ИИ в службу "Аналитика видеотрансляций", а затем в Центр Интернета вещей. Эти события также будут содержатся в окне **Вывод**. Значение ENTITY — это объекты обнаружения, а EVENT — события spaceanalytics. Эти выходные данные будут переданы в Аналитику видеотрансляций.

Пример выходных данных для personZoneEvent (из cognitiveservices.vision.spatialanalysis-personcrossingpolygon.livevideoanalytics operation):

```
{
  "body": {
    "timestamp": 143810610210090,
    "inferences": [
      {
        "type": "entity",
        "subtype": "",
        "inferenceId": "a895c86824db41a898f776da1876d230",
        "relatedInferences": [],
        "entity": {
          "tag": {
            "value": "person",
            "confidence": 0.66026187
          },
          "attributes": [],
          "box": {
            "l": 0.26559368,
            "t": 0.17887735,
            "w": 0.49247605,
            "h": 0.76629657
          }
        },
        "extensions": {},
        "valueCase": "entity"
      },
      {
        "type": "event",
        "subtype": "",
        "inferenceId": "995fe4206847467f8dfde83a15187d76",
        "relatedInferences": [
          "a895c86824db41a898f776da1876d230"
        ],
        "event": {
          "name": "personZoneEvent",
          "properties": {
            "status": "Enter",
            "metadataVersion": "1.0",
            "zone": "polygon0",
            "trackingId": "a895c86824db41a898f776da1876d230"
          }
        },
        "extensions": {},
        "valueCase": "event"
      }
    ]
  },
  "applicationProperties": {
    "topic": "/subscriptions/81631a63-f0bd-4f35-8344-5bc541b36bfc/resourceGroups/lva-sample-resources/providers/microsoft.media/mediaservices/lvasamplea4bylcwoqqypi",
    "subject": "/graphInstances/Sample-Graph-1/processors/spatialanalysisExtension",
    "eventType": "Microsoft.Media.Graph.Analytics.Inference",
    "eventTime": "2020-08-20T03:54:29.001Z",
    "dataVersion": "1.0"
  }
}
```

## <a name="next-steps"></a>Следующие шаги

Попробуйте выполнить различные операции, доступные для `spatialAnalysis`, например **personCount** и **personDistance**, установив флаг "enabled" (включено) в узле графа файла манифеста развертывания.
>[!Tip]
> Используйте [пример видеофайла](https://lvamedia.blob.core.windows.net/public/2018-03-07.16-50-00.16-55-00.school.G421.mkv), в котором присутствует несколько человек.

> [!NOTE]
> Вы можете выполнять только одну операцию за раз. Поэтому убедитесь, что значение **true** принимает только один флаг (остальные флаги должны принимать значение **false**).