---
title: Анализ видеотрансляций с помощью OpenVINO™ Model Server (расширение ИИ от Intel)
description: В этом учебнике вы будете использовать сервер модели искусственного интеллекта от Intel для анализа веб-канала видеотрансляций с помощью IP-камеры (имитированной).
ms.topic: tutorial
ms.date: 09/08/2020
titleSuffix: Azure
ms.openlocfilehash: 5751184493fffeeaf647507e9e9b00834f63ab5e
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105557264"
---
# <a name="tutorial-analyze-live-video-by-using-openvino-model-server--ai-extension-from-intel"></a>Руководство по Анализ видеотрансляций с помощью OpenVINO™ Model Server (расширение ИИ от Intel) 

В этом учебнике показано, как использовать OpenVINO™ Model Server (расширение ИИ от Intel) для анализа веб-канала видеотрансляций с помощью IP-камеры (имитированной). Вы увидите, как этот сервер вывода предоставляет доступ к моделям для обнаружения объектов (человека, автомобиля или велосипеда) и модели для классификации транспортных средств. Подмножество кадров в веб-канале видеотрансляций отправляется на этот сервер вывода, а результаты отправляются в центр IoT Edge.

В этом учебнике используется имитация видеотрансляции, а в качестве устройства IoT Edge — виртуальная машина Azure. Оно основывается на образце кода, написанном на C# для краткого руководства [Создание событий при обнаружении движения](detect-motion-emit-events-quickstart.md).

> [!NOTE]
> При работе с руководством требуется использование компьютера x86-64 в качестве пограничного устройства.

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись Azure с активной подпиской. Если у вас еще нет учетной записи, [ создайте ее бесплатно](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
  > [!NOTE]
  > Вам потребуется подписка Azure с разрешениями на создание субъектов-служб (такие разрешения предоставляет роль **Владелец**). Если у вас нет нужных разрешений, обратитесь к администратору учетной записи с просьбой предоставить вам нужные разрешения. 
* [Visual Studio Code](https://code.visualstudio.com/) со следующими расширениями:
    * [Средства Интернета вещей Azure](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
    * [C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
* [Пакет SDK для .NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/3.1).
* Если вы еще не изучали краткое руководство [Создание событий при обнаружении движения](detect-motion-emit-events-quickstart.md), обязательно [настройте ресурсы Azure](detect-motion-emit-events-quickstart.md#set-up-azure-resources), выполнив указанные в нем шаги.

> [!TIP]
> Во время установки расширения Azure IoT Tools может появиться запрос на установку Docker. На него можно не обращать внимания.

## <a name="review-the-sample-video"></a>Просмотр примера видео

При настройке ресурсов Azure на виртуальную машину Linux в Azure, используемую в качестве устройства IoT Edge, копируется короткое видео с изображением автостоянки. В этом кратком руководстве для имитации потока в реальном времени используется видеофайл.

Откройте приложение, например [проигрыватель мультимедиа VLC](https://www.videolan.org/vlc/). Нажмите сочетание клавиш CTRL+N, а затем вставьте ссылку на [видео](https://lvamedia.blob.core.windows.net/public/lots_015.mkv), чтобы начать воспроизведение. Вы увидите видеоматериал с автомобилями на автостоянке, большинство из которых припарковано, а один движется.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4LUbN]

В этом кратком руководстве Аналитика видеотрансляций в IoT Edge используется вместе с OpenVINO™ Model Server (расширением ИИ от Intel) для обнаружения таких объектов, как транспортные средства, и их классификации. Полученные события вывода будут опубликованы в центре IoT Edge.

## <a name="overview"></a>Обзор

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/use-intel-openvino-tutorial/http-extension-with-vino.svg" alt-text="Обзор":::

На схеме показан порядок передачи сигналов в этом кратком руководстве. [Пограничный модуль](https://github.com/Azure/live-video-analytics/tree/master/utilities/rtspsim-live555) имитирует IP-камеру, на которой находится RTSP-сервер. Узел [RTSP-источника](media-graph-concept.md#rtsp-source) получает видеосигнал с этого сервера и передает видеокадры на узел [обработчика расширений HTTP](media-graph-concept.md#http-extension-processor). 

Узел расширения HTTP играет роль прокси-сервера. Он выбирает входящие видеокадры, заданные в поле `samplingOptions`, а также преобразует видеокадры в изображения указанного типа. Затем с использованием REST он передает изображение на другой пограничный модуль, выполняющий модели искусственного интеллекта на конечной точке HTTP. В этом примере этим пограничным модулем является OpenVINO™ Model Server (расширение ИИ от Intel). Узел обработчика расширений HTTP собирает результаты обнаружения и публикует события в узле [приемника Центра Интернета вещей](media-graph-concept.md#iot-hub-message-sink). Затем узел отправляет эти события в [центр IoT Edge](../../iot-fundamentals/iot-glossary.md#iot-edge-hub).

Изучив данный учебник, вы научитесь:

1. Создание и развертывание графа мультимедиа и его изменение.
1. Интерпретация результатов.
1. Очистка ресурсов.

## <a name="about-openvino-model-server--ai-extension-from-intel"></a>Сведения о OpenVINO™ Model Server (расширении ИИ от Intel)

[Intel® Distribution of OpenVINO™ toolkit](https://software.intel.com/content/www/us/en/develop/tools/openvino-toolkit.html) (открытая оптимизация вывода визуальных объектов и нейронной сети) — это бесплатный комплект программного обеспечения, который помогает разработчикам и специалистам по обработке данных ускорить рабочие нагрузки компьютерного зрения, оптимизировать вывод глубокого обучения и развертывание, а также упростить разнородное выполнение на платформах Intel® (от пограничных к облачным). Решение включает в себя Intel® Deep Learning Deployment Toolkit с оптимизатором моделей и подсистемой вывода, а также репозиторий [Open Model Zoo](https://github.com/openvinotoolkit/open_model_zoo) с более 40 оптимизированными предварительно обученными моделями.

Чтобы создавать сложные высокопроизводительные решения для анализа видеотрансляций, Аналитику видеотрансляций в модуле IoT Edge следует связать с мощной подсистемой вывода, которая может использовать масштабирование в пограничной среде. В рамках этого учебника запросы на вывод отправляются в [OpenVINO™ Model Server (расширение ИИ от Inte)](https://aka.ms/lva-intel-ovms), модуль Edge, предназначенный для работы с Аналитикой видеотрансляций в IoT Edge. Этот серверный модуль вывода содержит OpenVINO™ Model Server (OVMS), сервер вывода на базе набора средств OpenVINO™, который значительно оптимизирован для рабочих нагрузок компьютерного зрения и разработан для архитектур Intel®. В OVMS добавлено расширение для простого обмена видеокадрами и результатами вывода между сервером вывода и службой "Аналитика видеотрансляций" в модуле IoT Edge, что дает вам возможность запускать любую модель, поддерживаемую набором средств OpenVINO™ (вы можете настроить модуль сервера вывода, изменив [код](https://github.com/openvinotoolkit/model_server/tree/master/extras/ams_wrapper)). Вы можете дополнительно выбрать один из множества механизмов ускорения, обеспечиваемых оборудованием Intel®. К ним относятся процессоры (Atom, Core, Xeon), ППВМ, VPU.

В первоначальном выпуске этого сервера вывода можно использовать указанные ниже [модели](https://github.com/openvinotoolkit/model_server/tree/master/extras/ams_models).

- Обнаружение автомобиля (URL-адрес вывода: http://{module-name}:4000/vehicleDetection).
- Обнаружение лица/автомобиля/велосипеда (URL-адрес вывода: http://{module-name}:4000/personVehicleBikeDetection).
- Классификация автомобилей (URL-адрес вывода: http://{module-name}:4000/vehicleClassification).
- Распознавание лиц (URL-адрес вывода: http://{module-name}:4000/faceDetection).

> [!NOTE]
> Скачав и используя модуль Edge, OpenVINO™ Model Server (расширение ИИ от Intel) и прилагаемое программное обеспечение, вы соглашаетесь с условиями [Лицензионного соглашения](https://www.intel.com/content/www/us/en/legal/terms-of-use.html).
> Intel обязуется уважать права человека и избегать соучастия в нарушениях этих прав. Дополнительные сведения см. на странице [Глобальные принципы прав человека в Intel](https://www.intel.com/content/www/us/en/policy/policy-human-rights.html). Продукты и программное обеспечение Intel предназначены только для приложений, использование которых не приводит к нарушению международно признанных прав человека.

## <a name="create-and-deploy-the-media-graph"></a>Создание и развертывание графа мультимедиа

### <a name="examine-and-edit-the-sample-files"></a>Изучение и изменение образцов файлов

Согласно предварительным требованиям вы загрузили в папку пример кода. Выполните указания ниже, чтобы изучить и изменить файлы примеров.

1. В Visual Studio Code перейдите в папку *src/edge*. Здесь находится созданный нами *ENV*-файл и несколько файлов шаблона развертывания.

    Шаблон развертывания содержит манифест развертывания для пограничного устройства. Он содержит некоторые значения-заполнители. В *ENV*-файле указаны значения таких переменных.

1. Теперь перейдите в папку *src/cloud-to-device-console-app*. Здесь вы увидите файл *appSettings.json* и несколько других файлов:

    * ***c2d-console-app.csproj*** — файл проекта Visual Studio Code.
    * ***operations.json*** — список операций, которые должна запускать программа.
    * ***Program.cs*** — пример кода программы. Этот код выполняет следующие действия:

        * загружает параметры приложения;
        * вызывает прямые методы, которые представляются модулем Аналитики видеотрансляций в IoT Edge (с помощью модуля можно анализировать потоки видеотрансляции, вызывая его [прямые методы](direct-methods.md));
        * приостанавливает выполнение, позволяя проанализировать выходные данные программы в окне **ТЕРМИНАЛ** и проверить сгенерированные модулем события в окне **ВЫХОДНЫЕ ДАННЫЕ**;
        * вызывает прямые методы для очистки ресурсов.


1. Внесите правки в файл *operations.json*:
    * Измените ссылку на топологию графа:

        `"topologyUrl" : "https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/httpExtensionOpenVINO/2.0/topology.json"`

    * В разделе `GraphInstanceSet` измените имя топологии графа, чтобы оно совпадало со значением в приведенной выше ссылке:

      `"topologyName" : "InferencingWithOpenVINO"`

    * В разделе `GraphTopologyDelete` измените имя:

      `"name": "InferencingWithOpenVINO"`

### <a name="generate-and-deploy-the-iot-edge-deployment-manifest"></a>Создание и развертывание манифеста развертывания IoT Edge

1. Щелкните правой кнопкой мыши файл *src/edge/deployment.openvino.template.json* и выберите **Generate IoT Edge Deployment Manifest** (Создать манифест развертывания IoT Edge).

    ![Создание файла манифеста развертывания IoT Edge](./media/use-intel-openvino-tutorial/generate-deployment-manifest.png)  

    Файл манифеста *deployment.openvino.amd64.json* создается в папке *src/edge/config*.

1. Если вы прошли краткое руководство [Создание событий при обнаружении движения](detect-motion-emit-events-quickstart.md), пропустите этот шаг. 

    В противном случае рядом с областью **Центр Интернета вещей Azure** в левом нижнем углу щелкните значок **Дополнительные действия**, а затем выберите **Установка строки подключения Центра Интернета вещей**. Строку можно скопировать из файла *appsettings.json*. Чтобы обеспечить правильность настройки центра Интернета вещей в Visual Studio Code, используйте команду [Выбрать центр Интернета вещей](https://github.com/Microsoft/vscode-azure-iot-toolkit/wiki/Select-IoT-Hub).
    
    ![Установка строки подключения Центра Интернета вещей](./media/quickstarts/set-iotconnection-string.png)

> [!NOTE]
> Возможно, вам будет предложено ввести сведения о встроенной конечной точке для Центра Интернета вещей. Чтобы получить эти сведения, на портале Azure перейдите в Центр Интернета вещей и найдите пункт **Встроенные конечные точки** в области навигации слева. Щелкните этот пункт и найдите значение **конечной точки, совместимой с центрами событий,** в разделе **Конечная точка, совместимая с концентратором событий**. В поле скопируйте текст и используйте его. Значение конечной точки будет приблизительно таким:  
    ```
    Endpoint=sb://iothub-ns-xxx.servicebus.windows.net/;SharedAccessKeyName=iothubowner;SharedAccessKey=XXX;EntityPath=<IoT Hub name>
    ```

1. Щелкните правой кнопкой мыши файл *src/edge/config/deployment.openvino.amd64.json* и выберите **Create Deployment for Single Device** (Создать развертывание для одного устройства). 

    ![Создание развертывания для одного устройства](./media/use-intel-openvino-tutorial/deploy-manifest.png)

1. Когда появится запрос на выбор устройства Центра Интернета вещей, выберите **lva-sample-device**.
1. Примерно через 30 секунд обновите содержимое Центра Интернета вещей в левом нижнем углу окна. Теперь в пограничном устройстве должны отображаться следующие развернутые модули:

    * Модуль Аналитики видеотрансляций **lvaEdge**.
    * Модуль **rtspsim**, имитирующий RTSP-сервер, выступающий в качестве источника веб-канала видеотрансляции.
    * Модуль **openvino** (модуль OpenVINO™ Model Server (расширения ИИ от Intel)). 

### <a name="prepare-to-monitor-events"></a>Подготовка к мониторингу событий

Щелкните правой кнопкой мыши устройство Аналитики видеотрансляции и выберите **Запуск мониторинга встроенной конечной точки события**. Этот шаг необходим для мониторинга событий Центра Интернета вещей в окне **ВЫХОДНЫЕ ДАННЫЕ** в Visual Studio Code. 

![Запуск мониторинга](./media/quickstarts/start-monitoring-iothub-events.png) 

### <a name="run-the-sample-program-to-detect-vehicles"></a>Запуск примера программы для обнаружения транспортных средств
Если вы откроете [топологию графа](https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/httpExtensionOpenVINO/topology.json) для этого учебника в браузере, вы увидите, что для параметра `inferencingUrl` установлено значение `http://openvino:4000/vehicleDetection`. Это означает, что сервер вывода будет возвращать результаты после обнаружения транспортных средств (при их наличии) в видеотрансляции.

1. В Visual Studio Code откройте вкладку **Расширения** (или нажмите клавиши CTRL+SHIFT+X) и найдите центр Интернета вещей Azure.
1. Щелкните правой кнопкой мыши и выберите **Параметры расширения**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/extensions-tab.png" alt-text="Параметры расширения":::
1. Найдите и включите параметр "Show Verbose Message" (Показывать подробное сообщение).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/show-verbose-message.png" alt-text="Show Verbose Message"::: (Показывать подробное сообщение)
1. Чтобы начать сеанс отладки, нажмите клавишу F5. В окне **ТЕРМИНАЛ** отображаются выводимые сообщения.
1. Код *operations.json* начинается с вызовов прямых методов `GraphTopologyList` и `GraphInstanceList`. Если вы очистили ресурсы после работы с предыдущими краткими руководствами, тогда этот процесс возвратит пустые списки, а затем приостановится. Чтобы продолжить, нажмите клавишу ВВОД.

    В окне **ТЕРМИНАЛ** показывается следующий набор вызовов прямых методов:

     * Вызов `GraphTopologySet`, который использует использованный ранее `topologyUrl`.
     * Вызов `GraphInstanceSet`, который использует такой код:

         ```
         {
           "@apiVersion": "2.0",
           "name": "Sample-Graph-1",
           "properties": {
             "topologyName": "InferencingWithOpenVINO",
             "description": "Sample graph description",
             "parameters": [
               {
                 "name": "rtspUrl",
                 "value": "rtsp://rtspsim:554/media/lots_015.mkv"
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
         ```

     * Вызов `GraphInstanceActivate`, который активирует экземпляр графа и поток видео.
     * Второй вызов к `GraphInstanceList`, который показывает, что экземпляр графа находится в состоянии выполнения
1. Вывод данных в окне **ТЕРМИНАЛ** приостановится с появлением предложения `Press Enter to continue`. Не нажимайте клавишу ВВОД на этом этапе. Прокрутите экран вверх, чтобы увидеть полезные данные ответов JSON на вызванные нами прямые методы.
1. Перейдите в окно **ВЫХОДНЫЕ ДАННЫЕ** в Visual Studio Code. Вы увидите сообщения, которые модуль Аналитики видеотрансляций в IoT Edge передает в центр Интернета вещей. Эти сообщения рассматриваются в следующем разделе этого краткого руководства.
1. Граф мультимедиа продолжит работать и выводить результаты. Симулятор RTSP будет продолжать циклический перебор исходного видео. Чтобы остановить граф мультимедиа, вернитесь к окну **ТЕРМИНАЛ** и нажмите клавишу ВВОД. 

    Следующая серия вызовов очищает ресурсы:
      * Вызов `GraphInstanceDeactivate` деактивирует экземпляр графа.
      * Вызов `GraphInstanceDelete` удаляет этот экземпляр.
      * Вызов `GraphTopologyDelete` удаляет топологию.
      * Окончательный вызов `GraphTopologyList` показывает, что список теперь пуст.

## <a name="interpret-results"></a>Интерпретация результатов

При запуске графа мультимедиа результаты из узла обработчика расширения HTTP отправляются через узел-преемник Центра Интернета вещей в центр Интернета вещей. Сообщения, отображаемые в окне **ВЫХОДНЫЕ ДАННЫЕ**, содержат раздел `body` и раздел `applicationProperties`. Дополнительные сведения см. в статье [Создание и чтение сообщений Центра Интернета вещей](../../iot-hub/iot-hub-devguide-messages-construct.md).

В приведенных ниже сообщениях модуль Аналитики видеотрансляций определяет свойства приложения и содержимое раздела body. 

### <a name="mediasessionestablished-event"></a>Событие MediaSessionEstablished

После создания экземпляра графа мультимедиа узел источника RTSP пытается подключиться к RTSP-серверу, работающему в контейнере rtspsim-live555. Если соединение установлено, будет выводиться указанное ниже событие. Тип этого события — **Microsoft.Media.MediaGraph.Diagnostics.MediaSessionEstablished**.

```
[IoTHubMonitor] [9:42:18 AM] Message received from [lvaedgesample/lvaEdge]:
{
  "body": {
    "sdp&quot;: &quot;SDP:\nv=0\r\no=- 1586450538111534 1 IN IP4 nnn.nn.0.6\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/lots_015.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.03.06\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-300.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/lots_015.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=4D0029;sprop-parameter-sets=Z00AKeKQCgC3YC3AQEBpB4kRUA==,aO48gA==\r\na=control:track1\r\n"
  },
  "applicationProperties": {
    "dataVersion": "1.0",
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/mediaservices/hubname",
    "subject": "/graphInstances/GRAPHINSTANCENAMEHERE/sources/rtspSource",
    "eventType": "Microsoft.Media.MediaGraph.Diagnostics.MediaSessionEstablished",
    "eventTime": "2020-07-24T16:42:18.1280000Z"
  }
}
```

В этом сообщении обратите внимание на следующие сведения:

* Сообщение является событием диагностики. `MediaSessionEstablished` указывает, что исходный узел RTSP (параметр subject) установил подключение к симулятору RTSP и начал прием потока данных (имитируемого).
* В `applicationProperties` `subject` указывает, что сообщение было создано из узла RTSP-источника в графе мультимедиа.
* В `applicationProperties` `eventType` указывает, что это событие является событием диагностики.
* `eventTime` указывает время события.
* `body` содержит данные о событии диагностики. В этом случае данные содержат сведения [протокола SDP](https://en.wikipedia.org/wiki/Session_Description_Protocol).

### <a name="inference-event"></a>Событие вывода

Узел процессора расширений HTTP получает результаты вывода из модуля OpenVINO™ Model Server (расширения ИИ). Затем он выдает результаты через узел приемника Центра Интернета вещей в качестве событий вывода. 

В этих событиях тип имеет значение `entity`, чтобы указать, что это сущность, например легковой автомобиль или грузовик. Значение `eventTime` — время в формате UTC, когда был обнаружен объект. 

В следующем примере обнаружены два транспортных средства со значениями доверительного уровня выше 0,9.

```
[IoTHubMonitor] [9:43:18 AM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "inferences": [
      {
        "type": "entity",
        "subtype": "vehicleDetection",
        "entity": {
          "tag": {
            "value": "vehicle",
            "confidence": 0.9951713681221008
          },
          "box": {
            "l": 0.042635321617126465,
            "t": 0.4004564881324768,
            "w": 0.10961548984050751,
            "h": 0.07942074537277222
          }
        }
      },
      {
        "type": "entity",
        "subtype": "vehicleDetection",
        "entity": {
          "tag": {
            "value": "vehicle",
            "confidence": 0.928486168384552
          },
          "box": {
            "l": 0.2506900727748871,
            "t": 0.07512682676315308,
            "w": 0.05470699071884155,
            "h": 0.07408371567726135
          }
        }
      }
    ]
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/mediaservices/hubname",
    "subject": "/graphInstances/GRAPHINSTANCENAMEHERE/processors/inferenceClient",
    "eventType": "Microsoft.Media.Graph.Analytics.Inference",
    "eventTime&quot;: &quot;2020-07-24T16:43:18.1280000Z"
  }
}
```

В сообщениях обратите внимание на следующие сведения:

* В `applicationProperties` `subject` — это узел в топологии графа, из которого было создано сообщение. 
* В `applicationProperties` `eventType` указывает, что это событие является событием аналитики.
* Значение `eventTime` — это время возникновения события.
* Раздел `body` содержит информацию об аналитическом событии. В этом случае событие является событием вывода, поэтому текст содержит данные `inferences`.
* Раздел `inferences` указывает, что `type` — это `entity`. Этот раздел содержит дополнительные данные о сущности.

## <a name="run-the-sample-program-to-detect-persons-or-vehicles-or-bikes"></a>Запуск примера программы для распознавания лиц или обнаружения автомобилей или велосипедов
Чтобы использовать другую модель, необходимо изменить топологию графа, а также файл `operations.json`.

Скопируйте [топологию графа](https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/httpExtensionOpenVINO/topology.json) в локальный файл, например `C:\TEMP\topology.json`. Откройте эту копию и измените значение `inferencingUrl` на `http://openvino:4000/personVehicleBikeDetection`.

Затем в Visual Studio Code перейдите в папку *src/cloud-to-device-console-app* и откройте файл `operations.json`. Измените строку с `topologyUrl` следующим образом:

```
      "topologyFile" : "C:\\TEMP\\topology.json" 
```
Теперь можно повторить приведенные выше шаги, чтобы снова запустить пример программы с новой топологией. Результаты вывода будут похожи (в схеме) на результаты модели обнаружения автомобилей, только для `subtype` будет установлено значение `personVehicleBikeDetection`.

## <a name="run-the-sample-program-to-classify-vehicles"></a>Запуск примера программы для классификации транспортных средств
В Visual Studio Code откройте локальную копию `topology.json` из предыдущего шага и измените значение `inferencingUrl` на `http://openvino:4000/vehicleClassification`. Если вы выполнили предыдущий пример распознавания лиц или обнаружения автомобилей и велосипедов, вам не нужно снова изменять файл `operations.json`.

Теперь можно повторить приведенные выше шаги, чтобы снова запустить пример программы с новой топологией. Пример результата классификации выглядит следующим образом:

```
[IoTHubMonitor] [9:44:18 AM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "inferences": [
      {
        "type": "classification",
        "subtype": "color",
        "classification": {
          "tag": {
            "value": "black",
            "confidence": 0.9179772138595581
          }
        }
      },
      {
        "type": "classification",
        "subtype": "type",
        "classification": {
          "tag": {
            "value": "truck",
            "confidence": 1
          }
        }
      }
    ]
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/mediaservices/hubname",
    "subject": "/graphInstances/GRAPHINSTANCENAMEHERE/processors/inferenceClient",
    "eventType": "Microsoft.Media.Graph.Analytics.Inference",
    "eventTime&quot;: &quot;2020-07-24T16:44:18.1280000Z"
  }
}
```

## <a name="run-the-sample-program-to-detect-faces"></a>Запуск примера программы для распознавания лиц
В Visual Studio Code откройте локальную копию `topology.json` из предыдущего шага и измените значение `inferencingUrl` на `http://openvino:4000/faceDetection`. Если вы выполнили предыдущий пример распознавания лиц или обнаружения автомобилей и велосипедов, вам не нужно снова изменять файл `operations.json`.

Теперь можно повторить приведенные выше шаги, чтобы снова запустить пример программы с новой топологией. Пример результата распознавания выглядит следующим образом (примечание: видео автостоянки, использованное выше, не содержит ни одного лица для распознавания, поэтому для этой модели следует использовать другое видео).

```
[IoTHubMonitor] [9:54:18 AM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "inferences": [
      {
        "type": "entity",
        "subtype": "faceDetection",
        "entity": {
          "tag": {
            "value": "face",
            "confidence": 0.9997053742408752
          },
          "box": {
            "l": 0.2559490501880646,
            "t": 0.03403960168361664,
            "w": 0.17685115337371826,
            "h": 0.45835764706134796
          }
        }
      }
    ]
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/mediaservices/hubname",
    "subject": "/graphInstances/GRAPHINSTANCENAMEHERE/processors/inferenceClient",
    "eventType": "Microsoft.Media.Graph.Analytics.Inference",
    "eventTime&quot;: &quot;2020-07-24T16:54:18.1280000Z"
  }
}
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы планируете изучить другие краткие руководства или учебники, сохраните созданные ресурсы. В противном случае на портале Azure перейдите к группам ресурсов, выберите группу ресурсов, с которой вы работали в этом учебнике, и удалите все ресурсы.

## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь с дополнительными задачами для опытных пользователей.

* Вместо симулятора RTSP используйте [IP-камеру](https://en.wikipedia.org/wiki/IP_camera) с поддержкой RTSP. IP-камеры, поддерживающие протокол RTSP, можно найти на странице [продуктов, соответствующих ONVIF](https://www.onvif.org/conformant-products/). Ищите устройства, которые соответствуют профилям G, S или T.
* Используйте устройство AMD64 или x64 на базе Linux вместо виртуальной машины Linux в Azure. Устройство должно находиться в той же сети, что и IP-камера. Вы можете выполнить инструкции из статьи [Install Azure IoT Edge runtime on Linux](../../iot-edge/how-to-install-iot-edge.md) (Установка среды выполнения Azure IoT Edge в системах Linux на основе Debian). Чтобы зарегистрировать устройство в Центре Интернета вещей Azure, следуйте инструкциям краткого руководства [Развертывание первого модуля IoT Edge на виртуальном устройстве Linux](../../iot-edge/quickstart-linux.md).