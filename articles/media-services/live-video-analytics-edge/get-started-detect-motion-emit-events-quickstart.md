---
title: Начало работы с Аналитикой видеотрансляции в IoT Edge — Azure
description: В этом кратком руководстве показано, как приступить к работе с Аналитикой видеотрансляций в IoT Edge. Узнайте, как обнаружить движение в потоке видеотрансляции.
ms.topic: quickstart
ms.date: 04/27/2020
ms.openlocfilehash: 57edf1721249f839f5c781756b3e09bf59888dab
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101730292"
---
# <a name="quickstart-get-started---live-video-analytics-on-iot-edge"></a>Краткое руководство. Начало работы. Общие сведения об Аналитике видеотрансляции в IoT Edge

В этом руководстве содержатся пошаговые инструкции по началу работы с Аналитикой видеотрансляции в IoT Edge. Здесь в качестве устройства IoT Edge используется виртуальная машина Azure. Здесь также используется имитация потока видеотрансляции. 

После завершения процедуры настройки вы сможете запустить имитацию видеотрансляции с помощью графа мультимедиа, который обнаруживает движения в этой трансляции и сообщает о них. На следующей схеме показан граф мультимедиа.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/analyze-live-video/motion-detection.svg" alt-text="Аналитика видеотрансляции на основе обнаружения движения":::

Просмотрите следующее видео с подробными инструкциями о том, как приступить к работе со службой "Аналитика видеотрансляций" в IoT Edge.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4Hcax]

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись Azure с активной подпиской. Если у вас еще нет учетной записи, [ создайте ее бесплатно](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

  > [!NOTE]
  > Вам потребуется подписка Azure с разрешениями на создание субъектов-служб (такие разрешения предоставляет роль **Владелец**). Если у вас нет нужных разрешений, обратитесь к администратору учетной записи с просьбой предоставить вам нужные разрешения.  

* [Visual Studio Code](https://code.visualstudio.com/) на компьютере для разработки. Убедитесь, что у вас есть [расширение Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).
* Убедитесь, что в сети, к которой подключен компьютер разработки, разрешен исходящий трафик по протоколу управления очередью сообщений (AMQP) через порт 5671. Эта настройка позволяет Azure IoT Tools взаимодействовать с Центром Интернета вещей Azure.

> [!TIP]
> Во время установки расширения Azure IoT Tools может появиться запрос на установку Docker. Вы можете проигнорировать его.

## <a name="set-up-azure-resources"></a>Настройка ресурсов Azure

Для работы с этим учебником требуются следующие ресурсы Azure:

* Центр Интернета вещей
* Учетная запись хранения
* Учетная запись Служб мультимедиа Azure
* Виртуальная машина Linux в Azure с установленной [средой выполнения IoT Edge](../../iot-edge/how-to-install-iot-edge.md)

В целях этого краткого руководства мы рекомендуем использовать [сценарий подготовки ресурсов Аналитики видеотрансляций](https://github.com/Azure/live-video-analytics/tree/master/edge/setup) для развертывания необходимых ресурсов в подписке Azure. Для этого выполните следующие действия.

1. Перейдите на [портал Azure](https://portal.azure.com) и выберите значок Cloud Shell.
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/quickstarts/cloud-shell.png" alt-text="Cloud Shell":::
1. Если вы используете Cloud Shell впервые, вам будет предложено выбрать подписку для создания учетной записи хранения и общей папки для службы "Файлы Microsoft Azure". Выберите **Create storage** (Создать хранилище), чтобы создать учетную запись для хранения сведений о сеансах Cloud Shell. Эта учетная запись хранения отличается от учетной записи, которая будет создана сценарием для использования с учетной записью Служб мультимедиа Azure.
1. В раскрывающемся меню в левой части окна Cloud Shell выберите в качестве среды **bash**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/quickstarts/env-selector.png" alt-text="Выбор среды":::
1. Выполните следующую команду.

    ```
    bash -c "$(curl -sL https://aka.ms/lva-edge/setup-resources-for-samples)"
    ```
    
    Если скрипт завершит работу успешно, вы увидите все необходимые ресурсы в своей подписке. Этот скрипт позволяет настроить в общей сложности 12 ресурсов.
    1. **Конечная точка потоковой передачи** полезна для воспроизведения записанного ресурса AMS.
    1. **Виртуальная машина** будет использоваться в качестве пограничного устройства.
    1. **Диск** хранилища подключается к виртуальной машине для хранения мультимедиа и артефактов.
    1. **Группа безопасности сети** применяется для фильтрации входящего и исходящего трафика ресурсов Azure в виртуальной сети Azure.
    1. **Сетевой интерфейс** позволяет виртуальной машине Azure взаимодействовать с Интернетом, Azure и другими ресурсами.
    1. **Подключение бастиона** позволяет подключаться к виртуальной машине из браузера или с портала Azure.
    1. **Общедоступный IP-адрес** предоставляет ресурсам Azure возможность взаимодействовать с Интернетом и общедоступными службами Azure.
    1. **Виртуальная сеть Azure** позволяет ресурсам Azure различных типов (в том числе виртуальным машинам Azure) обмениваться данными друг с другом, с локальными сетями и Интернетом. Ознакомьтесь с дополнительными сведениями о [виртуальных сетях](../../virtual-network/virtual-networks-overview.md).
    1. **Центр Интернета вещей** выполняет роль централизованного концентратора сообщений для двусторонней связи между приложением Интернета вещей, модулями IoT Edge и управляемыми устройствами.
    1. **Учетная запись Служб мультимедиа** помогает управлять содержимым мультимедиа в Azure и выполнять его потоковую передачу.
    1. **Учетная запись хранения** сопоставляется с учетной записью Служб мультимедиа. Необходимо иметь одну основную учетную запись хранения. Допускается любое количество дополнительных учетных записей хранения. Дополнительные сведения см. в статье [Учетные записи хранения Azure с учетными записями Служб мультимедиа Azure](../latest/storage-account-concept.md).
    1. **Реестр контейнеров** помогает хранить и администрировать частные образы контейнеров Docker и связанные с ними артефакты.

В выходных данных скрипта содержится таблица ресурсов с именем Центра Интернета вещей. Найдите тип ресурса **`Microsoft.Devices/IotHubs`** и запишите его имя. Это имя потребуется на следующем шаге.  

> [!NOTE]
> Скрипт также создает несколько файлов конфигурации в каталоге ***~/clouddrive/lva-sample/*** . Эти файлы понадобятся вам позднее в этом кратком руководстве.

> [!TIP]
> Если вы столкнулись с проблемами с созданными ресурсами Azure, ознакомьтесь с нашим **[руководством по устранению неполадок](troubleshoot-how-to.md#common-error-resolutions)** , чтобы решить некоторые часто встречающиеся проблемы.

## <a name="deploy-modules-on-your-edge-device"></a>Развертывание модулей на пограничном устройстве

Выполните следующую команду в Cloud Shell.

```
az iot edge set-modules --hub-name <iot-hub-name> --device-id lva-sample-device --content ~/clouddrive/lva-sample/edge-deployment/deployment.amd64.json
```

Эта команда развертывает следующие модули на пограничном устройстве, которое в этом случае является виртуальной машиной Linux.

* Аналитика видеотрансляций в IoT Edge (модуль `lvaEdge`).
* Симулятор протокола RTSP (модуль `rtspsim`)

Модуль симулятора RTSP имитирует видеотрансляцию с применением видеофайла, который был скопирован на пограничное устройство при запуске [сценария подготовки ресурсов Аналитики видеотрансляций](https://github.com/Azure/live-video-analytics/tree/master/edge/setup). 

Теперь модули развернуты, но графы мультимедиа неактивны.

## <a name="configure-the-azure-iot-tools-extension"></a>Настройка расширения Azure IoT Tools

Выполните эти инструкции, чтобы подключиться к Центру Интернета вещей с помощью расширения Azure IoT Tools.

1. В Visual Studio Code откройте вкладку **Расширения** (или нажмите клавиши CTRL+SHIFT+X) и найдите Центр Интернета вещей Azure.
1. Щелкните правой кнопкой мыши и выберите **Параметры расширения**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/extensions-tab.png" alt-text="Параметры расширения":::
1. Найдите и включите параметр "Show Verbose Message" (Показывать подробное сообщение).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/show-verbose-message.png" alt-text="Show Verbose Message"::: (Показывать подробное сообщение)
1. Выберите **Вид** > **Проводник** или нажмите клавиши CTRL+SHIFT+E.
1. В левом нижнем углу вкладки **Explorer** выберите **Центр Интернета вещей Azure**.
1. Щелкните значок **Дополнительные параметры**, чтобы открыть контекстное меню. Затем выберите **Установка строки подключения Центра Интернета вещей**.
1. Когда появится поле ввода, введите строку подключения для Центра Интернета вещей. В Cloud Shell вы можете получить строку подключения из файла *~/clouddrive/lva-sample/appsettings.json*.

> [!NOTE]
> Возможно, вам будет предложено ввести сведения о встроенной конечной точке для Центра Интернета вещей. Чтобы получить эти сведения, на портале Azure перейдите в Центр Интернета вещей и найдите пункт **Встроенные конечные точки** в области навигации слева. Щелкните этот пункт и найдите значение **конечной точки, совместимой с центрами событий,** в разделе **Конечная точка, совместимая с концентратором событий**. В поле скопируйте текст и используйте его. Значение конечной точки будет приблизительно таким:  
    ```
    Endpoint=sb://iothub-ns-xxx.servicebus.windows.net/;SharedAccessKeyName=iothubowner;SharedAccessKey=XXX;EntityPath=<IoT Hub name>
    ```

Если подключение будет установлено, отобразится список пограничных устройств. Среди них должно быть по крайней мере одно устройство с именем **lva-sample-device**. Теперь вы можете управлять устройствами IoT Edge и взаимодействовать с Центром Интернета вещей Azure с помощью контекстного меню. Чтобы просмотреть модули, развернутые на пограничном устройстве, разверните узел **Modules** в разделе **lva-sample-device**.

![Узел lva-sample-device](./media/quickstarts/lva-sample-device-node.png)

> [!TIP]
> Если вы самостоятельно [вручную развернули Аналитику видеотрансляций в IoT Edge](deploy-iot-edge-device.md) на пограничном устройстве (например, на устройстве ARM64), модуль отобразится под таким устройством в Центре Интернета вещей Azure. Вы можете выбрать такой модуль и выполнить остальные шаги (приведены ниже).

## <a name="use-direct-method-calls"></a>Использование вызовов прямых методов

С помощью модуля можно анализировать потоки видеотрансляции, вызывая его прямые методы. Дополнительные сведения см. в статье [Прямые методы для Аналитики видеотрансляций в IoT Edge](direct-methods.md). 

### <a name="invoke-graphtopologylist"></a>Вызов GraphTopologyList

Чтобы перечислить все [топологии графов](media-graph-concept.md#media-graph-topologies-and-instances) в модуле:

1. В Visual Studio Code щелкните правой кнопкой мыши модуль **lvaEdge** и выберите **Вызывать прямой метод модуля**.
1. В появившемся поле введите *GraphTopologyList*.
1. Скопируйте следующие полезные данные JSON и вставьте их в поле. Затем нажмите клавишу ВВОД.

    ```
    {
        "@apiVersion" : "2.0"
    }
    ```

    Через несколько секунд в окне **ВЫХОДНЫЕ ДАННЫЕ** отобразится следующий ответ.

    ```
    [DirectMethod] Invoking Direct Method [GraphTopologyList] to [lva-sample-device/lvaEdge] ...
    [DirectMethod] Response from [lva-sample-device/lvaEdge]:
    {
      "status": 200,
      "payload": {
        "value": []
      }
    }
    ```
    
    Должен появится ответ, так как топологии графов не были созданы.
    

### <a name="invoke-graphtopologyset"></a>Вызов GraphTopologySet

Как и раньше, вы можете вызвать `GraphTopologySet` и задать [топологию графа](media-graph-concept.md#media-graph-topologies-and-instances). Используйте в качестве полезных данных следующий код JSON.

```
{
    "@apiVersion": "2.0",
    "name": "MotionDetection",
    "properties": {
        "description": "Analyzing live video to detect motion and emit events",
        "parameters": [
            {
                "name": "rtspUserName",
                "type": "String",
                "description": "rtsp source user name.",
                "default": "dummyUserName"
            },
            {
                "name": "rtspPassword",
                "type": "String",
                "description": "rtsp source password.",
                "default": "dummyPassword"
            },
            {
                "name": "rtspUrl",
                "type": "String",
                "description": "rtsp Url"
            }
        ],
        "sources": [
            {
                "@type": "#Microsoft.Media.MediaGraphRtspSource",
                "name": "rtspSource",
                "endpoint": {
                    "@type": "#Microsoft.Media.MediaGraphUnsecuredEndpoint",
                    "url": "${rtspUrl}",
                    "credentials": {
                        "@type": "#Microsoft.Media.MediaGraphUsernamePasswordCredentials",
                        "username": "${rtspUserName}",
                        "password": "${rtspPassword}"
                    }
                }
            }
        ],
        "processors": [
            {
                "@type": "#Microsoft.Media.MediaGraphMotionDetectionProcessor",
                "name": "motionDetection",
                "sensitivity": "medium",
                "inputs": [
                    {
                        "nodeName": "rtspSource"
                    }
                ]
            }
        ],
        "sinks": [
            {
                "@type": "#Microsoft.Media.MediaGraphIoTHubMessageSink",
                "name": "hubSink",
                "hubOutputName": "inferenceOutput",
                "inputs": [
                    {
                        "nodeName": "motionDetection"
                    }
                ]
            }
        ]
    }
}

```

Эта полезная нагрузка JSON создает топологию графа, которая определяет три параметра. Два из этих параметров содержат значения по умолчанию. Топология содержит один узел источника (RTSP Source), один узел обработчика (обработчик обнаружения движения) и один узел-приемник (приемник Центра Интернета вещей).

Через несколько секунд появится окно **ВЫХОДНЫЕ ДАННЫЕ** со следующим ответом:

```
[DirectMethod] Invoking Direct Method [GraphTopologySet] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 201,
  "payload": {
    "systemData": {
      "createdAt": "2020-05-19T07:41:34.507Z",
      "lastModifiedAt": "2020-05-19T07:41:34.507Z"
    },
    "name": "MotionDetection",
    "properties": {
      "description": "Analyzing live video to detect motion and emit events",
      "parameters": [
        {
          "name": "rtspUserName",
          "type": "String",
          "description": "rtsp source user name.",
          "default": "dummyUserName"
        },
        {
          "name": "rtspPassword",
          "type": "String",
          "description": "rtsp source password.",
          "default": "dummyPassword"
        },
        {
          "name": "rtspUrl",
          "type": "String",
          "description": "rtsp Url"
        }
      ],
      "sources": [
        {
          "@type": "#Microsoft.Media.MediaGraphRtspSource",
          "name": "rtspSource",
          "transport": "Tcp",
          "endpoint": {
            "@type": "#Microsoft.Media.MediaGraphUnsecuredEndpoint",
            "url": "${rtspUrl}",
            "credentials": {
              "@type": "#Microsoft.Media.MediaGraphUsernamePasswordCredentials",
              "username": "${rtspUserName}"
            }
          }
        }
      ],
      "processors": [
        {
          "@type": "#Microsoft.Media.MediaGraphMotionDetectionProcessor",
          "sensitivity": "medium",
          "name": "motionDetection",
          "inputs": [
            {
              "nodeName": "rtspSource",
              "outputSelectors": []
            }
          ]
        }
      ],
      "sinks": [
        {
          "@type": "#Microsoft.Media.MediaGraphIoTHubMessageSink",
          "hubOutputName": "inferenceOutput",
          "name": "hubSink",
          "inputs": [
            {
              "nodeName": "motionDetection",
              "outputSelectors": []
            }
          ]
        }
      ]
    }
  }
}
```

Возвращенное состояние — 201. Это состояние указывает, что была создана новая топология. 

Далее выполните такие действия:

1. Вызовите `GraphTopologySet` еще раз. Возвращенный код состояния — 200. Этот код указывает, что существующая топология успешно обновлена.
1. Вызовите `GraphTopologySet` еще раз, но измените строку описания. Возвращенный код состояния — 200, а описание обновлено новым значением.
1. Вызовите `GraphTopologyList`, как описано в предыдущем разделе. Теперь можно увидеть топологию `MotionDetection` в возвращенных полезных данных.

### <a name="invoke-graphtopologyget"></a>Вызов GraphTopologyGet

Вызовите `GraphTopologyGet`, используя следующие полезные данные.

```
{
    "@apiVersion" : "2.0",
    "name" : "MotionDetection"
}
```

Через несколько секунд появится окно **ВЫХОДНЫЕ ДАННЫЕ** со следующим ответом:

```
[DirectMethod] Invoking Direct Method [GraphTopologyGet] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 200,
  "payload": {
    "systemData": {
      "createdAt": "2020-05-19T07:41:34.507Z",
      "lastModifiedAt": "2020-05-19T07:41:34.507Z"
    },
    "name": "MotionDetection",
    "properties": {
      "description": "Analyzing live video to detect motion and emit events",
      "parameters": [
        {
          "name": "rtspUserName",
          "type": "String",
          "description": "rtsp source user name.",
          "default": "dummyUserName"
        },
        {
          "name": "rtspPassword",
          "type": "String",
          "description": "rtsp source password.",
          "default": "dummyPassword"
        },
        {
          "name": "rtspUrl",
          "type": "String",
          "description": "rtsp Url"
        }
      ],
      "sources": [
        {
          "@type": "#Microsoft.Media.MediaGraphRtspSource",
          "name": "rtspSource",
          "transport": "Tcp",
          "endpoint": {
            "@type": "#Microsoft.Media.MediaGraphUnsecuredEndpoint",
            "url": "${rtspUrl}",
            "credentials": {
              "@type": "#Microsoft.Media.MediaGraphUsernamePasswordCredentials",
              "username": "${rtspUserName}"
            }
          }
        }
      ],
      "processors": [
        {
          "@type": "#Microsoft.Media.MediaGraphMotionDetectionProcessor",
          "sensitivity": "medium",
          "name": "motionDetection",
          "inputs": [
            {
              "nodeName": "rtspSource",
              "outputSelectors": []
            }
          ]
        }
      ],
      "sinks": [
        {
          "@type": "#Microsoft.Media.MediaGraphIoTHubMessageSink",
          "hubOutputName": "inferenceOutput",
          "name": "hubSink",
          "inputs": [
            {
              "nodeName": "motionDetection",
              "outputSelectors": []
            }
          ]
        }
      ]
    }
  }
}
```

В полезных данных ответа обратите внимание на следующие сведения:

* Код состояния — 200, что означает успешное выполнение.
* Полезная нагрузка содержит метку времени `created` и метку времени `lastModified`.

### <a name="invoke-graphinstanceset"></a>Вызов GraphInstanceSet

Создайте экземпляр графа, который ссылается на указанную выше топологию графа. Экземпляры графа позволяют анализировать видеопотоки с нескольких камер с одинаковой топологией графа. Дополнительные сведения см. в разделе [Топологии и экземпляры графа мультимедиа](media-graph-concept.md#media-graph-topologies-and-instances).

Вызовите прямой метод `GraphInstanceSet`, используя следующие полезные данные.

```
{
    "@apiVersion" : "2.0",
    "name" : "Sample-Graph-1",
    "properties" : {
        "topologyName" : "MotionDetection",
        "description" : "Sample graph description",
        "parameters" : [
            { "name" : "rtspUrl", "value" : "rtsp://rtspsim:554/media/camera-300s.mkv" }
        ]
    }
}
```

Обратите внимание, что эти полезные данные:

* Указывает имя топологии (`MotionDetection`), для которой необходимо создать экземпляр.
* Содержит значение параметра для `rtspUrl`, для которого в полезных данных топологии графа отсутствует значение по умолчанию. Это значение является ссылкой на приведенный ниже пример видео:
    > [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4LTY4]
Через несколько секунд появится окно **ВЫХОДНЫЕ ДАННЫЕ** со следующим ответом:

```
[DirectMethod] Invoking Direct Method [GraphInstanceSet] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 201,
  "payload": {
    "name": "Sample-Graph-1",
    "properties": {
      "created": "2020-05-19T07:44:33.868Z",
      "lastModified": "2020-05-19T07:44:33.868Z",
      "state": "Inactive",
      "description": "Sample graph description",
      "topologyName": "MotionDetection",
      "parameters": [
        {
          "name": "rtspUrl",
          "value": "rtsp://rtspsim:554/media/camera-300s.mkv"
        }
      ]
    }
  }
}
```

В полезных данных ответа обратите внимание на следующее:

* Код состояния — 201, т. е. был создан новый экземпляр.
* Состояние — `Inactive`, т. е экземпляр графа был создан, но не активирован. Дополнительные сведения см. в разделе с описанием [состояний графа мультимедиа](media-graph-concept.md).

Далее выполните такие действия:

1. Снова вызовите `GraphInstanceSet`, используя те же полезные данные. Обратите внимание, что возвращенный код состояния — 200.
1. Вызовите `GraphInstanceSet` еще раз, но используйте другое описание. Обратите внимание на обновленное описание в полезных данных ответа, указывающее на то, что экземпляр графа успешно обновлен.
1. Вызовите `GraphInstanceSet`, но измените имя на `Sample-Graph-2`. В полезных данных ответа обратите внимание на созданный экземпляр графа (т. е. код состояния 201).

### <a name="invoke-graphinstanceactivate"></a>Вызов GraphInstanceActivate

Теперь активируйте экземпляр графа, чтобы запустить поток видеотрансляции через модуль. Вызовите прямой метод `GraphInstanceActivate`, используя следующие полезные данные.

```
{
    "@apiVersion" : "2.0",
    "name" : "Sample-Graph-1"
}
```

Через несколько секунд появится окно **ВЫХОДНЫЕ ДАННЫЕ** со следующим ответом:

```
[DirectMethod] Invoking Direct Method [GraphInstanceActivate] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 200,
  "payload": null
}
```

Код состояния 200 указывает на то, что экземпляр графа успешно активирован.

### <a name="invoke-graphinstanceget"></a>Вызов GraphInstanceGet

Теперь вызовите прямой метод `GraphInstanceGet`, используя следующие полезные данные.

```
 {
     "@apiVersion" : "2.0",
     "name" : "Sample-Graph-1"
 }
 ```

Через несколько секунд появится окно **ВЫХОДНЫЕ ДАННЫЕ** со следующим ответом:

```
[DirectMethod] Invoking Direct Method [GraphInstanceGet] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 200,
  "payload": {
    "name": "Sample-Graph-1",
    "properties": {
      "created": "2020-05-19T07:44:33.868Z",
      "lastModified": "2020-05-19T07:44:33.868Z",
      "state": "Active",
      "description": "graph description",
      "topologyName": "MotionDetection",
      "parameters": [
        {
          "name": "rtspUrl",
          "value": "rtsp://rtspsim:554/media/camera-300s.mkv"
        }
      ]
    }
  }
}
```

В полезных данных ответа обратите внимание на следующие сведения:

* Код состояния — 200, что означает успешное выполнение.
* Код состояния `Active` означает, что экземпляр графа теперь активен.

## <a name="observe-results"></a>Просмотр результатов

Созданный и активированный экземпляр графа использует узел обработчика обнаружения движения для обнаружения движения во входящем потоке видеотрансляции. Он отправляет события в узел приемника Центра Интернета вещей. Эти события ретранслируются в Центр IoT Edge. 

Чтобы просмотреть результаты, выполните следующие действия.

1. Откройте в Visual Studio Code панель **Explorer**. В левом нижнем углу найдите **Центр Интернета вещей Azure**.
2. Разверните узел **Devices** (Устройства).
3. Щелкните правой кнопкой мыши **lva-sample-device** а затем выберите **Запуск мониторинга встроенной конечной точки события**.

    ![Запуск мониторинга событий в Центре Интернета вещей](./media/quickstarts/start-monitoring-iothub-events.png)

    > [!NOTE]
    > Возможно, вам будет предложено ввести сведения о встроенной конечной точке для Центра Интернета вещей. Чтобы получить эти сведения, на портале Azure перейдите в Центр Интернета вещей и найдите пункт **Встроенные конечные точки** в области навигации слева. Щелкните этот пункт и найдите значение **конечной точки, совместимой с центрами событий,** в разделе **Конечная точка, совместимая с концентратором событий**. В поле скопируйте текст и используйте его. Значение конечной точки будет приблизительно таким:  
        ```
        Endpoint=sb://iothub-ns-xxx.servicebus.windows.net/;SharedAccessKeyName=iothubowner;SharedAccessKey=XXX;EntityPath=<IoT Hub name>
        ```
    
В окне **ВЫХОДНЫЕ ДАННЫЕ** отображается следующее сообщение:

```
[IoTHubMonitor] [7:44:33 AM] Message received from [lva-sample-device/lvaEdge]:
{
    "body": {
    "timestamp": 143005362606360,
    "inferences": [
        {
        "type": "motion",
        "motion": {
            "box": {
            "l": 0.828452,
            "t": 0.455224,
            "w": 0.1,
            "h": 0.088889
            }
        }
        },
        {
        "type": "motion",
        "motion": {
            "box": {
            "l": 0.661088,
            "t": 0.597015,
            "w": 0.0625,
            "h": 0.051852
            }
        }
        }
    ]
    }
}
```

Обратите внимание на следующие сведения:

* Сообщение содержит раздел `body` и раздел `applicationProperties`. Дополнительные сведения см. в статье [Создание и чтение сообщений Центра Интернета вещей](../../iot-hub/iot-hub-devguide-messages-construct.md).
* В `applicationProperties` `subject` — это узел в `MediaGraph`, из которого было создано сообщение. В данном случае сообщение исходит от обработчика обнаружения движения.
* В `applicationProperties` `eventType` указывает, что это событие является событием аналитики.
* Значение `eventTime` — это время возникновения события.
* Раздел `body` содержит информацию об аналитическом событии. В этом случае событие является событием вывода, поэтому текст содержит данные `timestamp` и `inferences`.
* Раздел `inferences` указывает, что `type` — это `motion`. Он предоставляет дополнительные данные о событии `motion`.

Если вы разрешите выполнение графа мультимедиа в течение определенного времени, в окне **ВЫХОДНЫЕ ДАННЫЕ** появится следующее сообщение.

```
[IoTHubMonitor] [7:47:45 AM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "sdp": "SDP:\nv=0\r\no=- 1588948185746703 1 IN IP4 172.xx.xx.xx\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/camera-300s.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.04.12\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-300.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/camera-300s.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=4D0029;sprop-parameter-sets={SPS}\r\na=control:track1\r\n"
  },
  "applicationProperties": {
    "dataVersion": "1.0",
    "topic": "/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/<my-resource-group>/providers/microsoft.media/mediaservices/<ams-account-name>",
    "subject": "/graphInstances/Sample-Graph-1/sources/rtspSource",
    "eventType": "Microsoft.Media.Graph.Diagnostics.MediaSessionEstablished",
    "eventTime": "2020-05-19T07:47:45.747Z"
  }
}
```

В этом сообщении обратите внимание на следующие сведения:

* В `applicationProperties` `subject` указывает, что сообщение было создано из узла RTSP-источника в графе мультимедиа.
* В `applicationProperties` `eventType` указывает, что это событие является событием диагностики.
* Раздел `body` содержит данные о диагностическом событии. В этом случае сообщение содержит текст, так как это событие `MediaSessionEstablished`.

## <a name="invoke-additional-direct-methods-to-clean-up"></a>Вызов дополнительных прямых методов для очистки

Вызовите прямые методы, чтобы сначала отключить экземпляр графа, а затем удалить его.

### <a name="invoke-graphinstancedeactivate"></a>Вызов GraphInstanceDeactivate

Вызовите прямой метод `GraphInstanceDeactivate`, используя следующие полезные данные.

```
{
    "@apiVersion" : "2.0",
    "name" : "Sample-Graph-1"
}
```

Через несколько секунд появится окно **ВЫХОДНЫЕ ДАННЫЕ** со следующим ответом:

```
[DirectMethod] Invoking Direct Method [GraphInstanceDeactivate] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 200,
  "payload": null
}
```

Код состояния 200 указывает на то, что экземпляр графа успешно деактивирован.

Затем попробуйте вызвать `GraphInstanceGet`, как показано ранее в этой статье. Обратите внимание на значение `state`.

### <a name="invoke-graphinstancedelete"></a>Вызов GraphInstanceDelete

Вызовите прямой метод `GraphInstanceDelete`, используя следующие полезные данные.

```
{
    "@apiVersion" : "2.0",
    "name" : "Sample-Graph-1"
}
```

Через несколько секунд появится окно **ВЫХОДНЫЕ ДАННЫЕ** со следующим ответом:

```
[DirectMethod] Invoking Direct Method [GraphInstanceDelete] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 200,
  "payload": null
}
```

Код состояния 200 указывает на то, что экземпляр графа успешно удален.

### <a name="invoke-graphtopologydelete"></a>Вызов GraphTopologyDelete

Вызовите прямой метод `GraphTopologyDelete`, используя следующие полезные данные.

```
{
    "@apiVersion" : "2.0",
    "name" : "MotionDetection"
}
```

Через несколько секунд появится окно **ВЫХОДНЫЕ ДАННЫЕ** со следующим ответом:

```
[DirectMethod] Invoking Direct Method [GraphTopologyDelete] to [lva-sample-device/lvaEdge] ...
[DirectMethod] Response from [lva-sample-device/lvaEdge]:
{
  "status": 200,
  "payload": null
}
```

Код состояния 200 означает, что топология графа успешно удалена.

Далее выполните такие действия:

1. Вызовите `GraphTopologyList` и убедитесь, что модуль не содержит топологий графов.
1. Вызовите `GraphInstanceList`, используя те же полезные данные, что и `GraphTopologyList`. Обратите внимание, что экземпляры графа не перечисляются.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не собираетесь использовать это приложение в дальнейшем, удалите ресурсы, созданные в этом кратком руководстве.

## <a name="next-steps"></a>Дальнейшие действия

* Узнайте, как [записывать видео с помощью Аналитики видеотрансляций в IoT Edge](continuous-video-recording-tutorial.md).
* Узнайте больше о [диагностических сообщениях](monitoring-logging.md).