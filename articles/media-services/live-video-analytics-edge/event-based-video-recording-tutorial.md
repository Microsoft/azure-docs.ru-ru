---
title: Руководство по записи видео в облако на основе событий и его воспроизведение — Azure
description: Из этого руководства можно узнать, как с помощью службы Аналитики видеотрансляций на платформе Azure IoT Edge записывать видео в облако на основе событий и воспроизводить его.
ms.topic: tutorial
ms.date: 05/27/2020
ms.openlocfilehash: ea98b4c8981be9fffe7911e4c8402a8f522976f9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101702323"
---
# <a name="tutorial-event-based-video-recording-to-the-cloud-and-playback-from-the-cloud"></a>Руководство. Запись видео в облако на основе событий и его воспроизведение

Из этого руководства вы узнаете, как с помощью службы Аналитики видеотрансляций на платформе IoT Edge осуществлять выборочную запись фрагментов видеотрансляции из источника в Службах мультимедиа Azure в облаке. Такой сценарий в этом руководстве называется [записью видео на основе событий](event-based-video-recording-concept.md). Для записи фрагментов видеотрансляции будет использоваться модель обнаружения объектов на базе ИИ для поиска объектов в видеопотоке и запуска видеозаписи только в случае обнаружения объекта некоторого типа. Вы также узнаете, как воспроизводить видеозаписи с помощью Служб мультимедиа. Эта возможность полезна в различных ситуациях, когда необходимо вести архив определенных видеозаписей. 

Выполняя данное руководство, вы сделаете следующее:

> [!div class="checklist"]
> * Настроите необходимые ресурсы.
> * Проверите код, который выполняет запись видео на основе событий.
> * Запустите пример кода.
> * Проверите результаты и просмотрите видео.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="suggested-pre-reading"></a>Рекомендуемые материалы для предварительного ознакомления  

Перед началом ознакомьтесь со следующими статьями:

* [Аналитика видеотрансляций в IoT Edge: обзор](overview.md)
* [Аналитика видеотрансляции в IoT Edge: терминология](terminology.md)
* [Граф мультимедиа: основные понятия](media-graph-concept.md) 
* [Запись видео на основе событий](event-based-video-recording-concept.md)
* [Руководство. Разработка модуля IoT Edge](../../iot-edge/tutorial-develop-for-linux.md)
* [Редактирование файла deployment.*.template.json](https://github.com/microsoft/vscode-azure-iot-edge/wiki/How-to-edit-deployment.*.template.json)
* Раздел, посвященный [объявлению маршрутов в манифесте развертывания IoT Edge](../../iot-edge/module-composition.md#declare-routes)

## <a name="prerequisites"></a>Предварительные требования

Обязательные компоненты для работы с этим руководством:

* [Visual Studio Code](https://code.visualstudio.com/) на компьютере для разработки с расширениями [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) и [C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp).

    > [!TIP]
    > Может появиться запрос на установку Docker. Проигнорируйте его.
* Установите [SDK .NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/thank-you/sdk-3.1.201-windows-x64-installer) на компьютере, на котором ведется разработка.
* Выполните [скрипт подготовки ресурсов аналитики видеотрансляции](https://github.com/Azure/live-video-analytics/tree/master/edge/setup) и [процедуру подготовки среды](./detect-motion-emit-events-quickstart.md?pivots=programming-language-csharp#set-up-your-development-environment).

После выполнения этой процедуры вы получите соответствующие ресурсы Azure, доступные в вашей подписке Azure:

* Центр Интернета вещей Azure
* Учетная запись хранения Azure
* Учетная запись Служб мультимедиа Azure
* Виртуальная машина Linux в Azure с установленной [средой выполнения IoT Edge](../../iot-edge/how-to-install-iot-edge.md)

> [!TIP]
> Если вы столкнулись с проблемами с созданными ресурсами Azure, ознакомьтесь с нашим **[руководством по устранению неполадок](troubleshoot-how-to.md#common-error-resolutions)** , чтобы решить некоторые часто встречающиеся проблемы.

## <a name="concepts"></a>Основные понятия

Запись видео на основе событий — это процесс записи видео, инициируемый определенным событием. Это событие может быть создано из:
- Обработка видеосигнала, например, при обнаружении перемещающегося объекта в видео.
- Независимый источник, например открытие двери. 

Кроме того, запись может активироваться при обнаружении определенного события службой логического вывода. В этом руководстве мы используем видео автомобилей, проезжающих по трассе, и будем записывать видео каждый раз при появлении грузовика.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/event-based-video-recording-tutorial/overview.svg" alt-text="Граф мультимедиа":::

Схема представляет собой изображение [графа мультимедиа](media-graph-concept.md) и дополнительных модулей, которые позволяют реализовать описанный сценарий. Используются четыре модуля IoT Edge:

* Модуль аналитики видеотрансляции в IoT Edge.
* Модуль Edge с моделью ИИ, расположенный за конечной точкой HTTP. В этом модуле ИИ используется модель [YOLOv3](https://github.com/Azure/live-video-analytics/tree/master/utilities/video-analysis/yolov3-onnx), которая может распознавать объекты различных видов.
* Пользовательский модуль для подсчета и фильтрации объектов, который на схеме назван "Object Counter". Вы создадите "Object Counter" и развернете его в этом руководстве.
* [Модуль симулятора RTSP](https://github.com/Azure/live-video-analytics/tree/master/utilities/rtspsim-live555), эмулирующий работу RTSP-камеры.
    
Как видно на схеме, мы используем узел [RTSP-источника](media-graph-concept.md#rtsp-source) в графе мультимедиа для захвата условного видеопотока движения автомобилей по шоссе и отправляем это видео по двум маршрутам:

* Первый маршрут — узел расширения HTTP. Узел создает выборку видеокадров на основе значения, которое вы задали с помощью поля `samplingOptions`, а затем передает кадры как изображения в модуль ИИ YOLOv3, который выполняет функцию детектора объектов. Узел получает результаты, которые являются объектами (автомобилями в потоке), обнаруженными моделью. Затем узел расширения HTTP публикует результаты через узел приемника сообщений Центра Интернета вещей в концентраторе IoT Edge.
* Модуль objectCounter настроен для приема сообщений от концентратора IoT Edge, которые включают результаты обнаружения объектов (автомобилей в потоке). Модуль проверяет эти сообщения на наличие объектов определенного типа, которые были заданы в настройках. При обнаружении такого объекта модуль отправляет сообщение в центр IoT Edge. Эти сообщения об обнаружении объекта затем перенаправляются в узел источника Центра Интернета вещей графа мультимедиа. Получив сообщение, узел источника Центра Интернета вещей в графе мультимедиа активирует узел [обработчика шлюза сигналов](media-graph-concept.md#signal-gate-processor). Затем узел обработчика сигнала шлюза откроется на заданный промежуток времени. В течение этого периода видео поступает через шлюз на узел приемника актива. Соответствующий фрагмент транслируемого видеопотока записывается через узел [приемника ресурса](media-graph-concept.md#asset-sink) в [ресурс](terminology.md#asset) в учетной записи Служб мультимедиа Azure.

## <a name="set-up-your-development-environment"></a>Настройка среды разработки

Прежде чем приступать к работе, убедитесь, что вы выполнили 3-й пункт из раздела [Предварительные требования](#prerequisites). После того, как скрипт подготовки ресурсов завершит свою работу, выберите фигурные скобки, чтобы раскрыть структуру папок. В каталоге ~/clouddrive/lva-sample будет создано несколько файлов.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/quickstarts/clouddrive.png" alt-text="Параметры приложения":::

В этом руководстве нас интересуют нижеперечисленные файлы:

* **~/clouddrive/lva-sample/edge-deployment/.env**: Содержит свойства, которые Visual Studio Code использует для развертывания модулей на пограничном устройстве.
* **~/clouddrive/lva-sample/appsetting.json**: Используется Visual Studio Code для запуска примера кода.

Для выполнения этих действий потребуются файлы.

1. Клонируйте репозиторий из ссылки GitHub https://github.com/Azure-Samples/live-video-analytics-iot-edge-csharp.
1. Запустите Visual Studio Code и откройте папку, в которую загружен репозиторий.
1. В Visual Studio Code перейдите в папку src/cloud-to-device-console-app и создайте файл **appsettings.json**. Этот файл содержит параметры, необходимые для выполнения программы.
1. Скопируйте содержимое файла ~/clouddrive/lva-sample/appsettings.json. Текст должен выглядеть так:

    ```
    {  
        "IoThubConnectionString" : "HostName=xxx.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=XXX",  
        "deviceId" : "lva-sample-device",  
        "moduleId" : "lvaEdge"  
    }
    ```

    Строка подключения к Центру Интернета вещей позволяет использовать Visual Studio Code для отправки команд модулям Edge через Центр Интернета вещей Azure.
    
1. Затем перейдите в папку src/edge и создайте файл с именем **.env**.
1. Скопируйте содержимое файла ~/clouddrive/lva-sample/edge-deployment/.env. Текст должен выглядеть так:

    ```
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
    CONTAINER_REGISTRY_PASSWORD_myacr="<your container registry username>"      
    ```

## <a name="examine-the-template-file"></a>Изучение примеров файлов 

На предыдущем этапе вы запустили Visual Studio Code и открыли папку, содержащую пример программного кода.

В Visual Studio Code перейдите в папку src/edge. Здесь находится созданный нами ENV-файл и несколько файлов шаблона развертывания. Этот шаблон определяет, какие модули Edge мы будем развертывать на пограничном устройстве (виртуальной машине Azure Linux). Файл .env содержит значения для переменных, которые используются в этих шаблонах, например учетных данных Служб мультимедиа.

Откройте файл src/edge/deployment.objectCounter.template.json. В разделе **модули** есть четыре записи — по одной для каждого из перечисленных в разделе "Основные понятия" объектов:

* **lvaEdge**: Это модуль аналитики видеотрансляции в IoT Edge.
* **yolov3**: Это модуль ИИ, который создан используя модель YOLO v3
* **rtspsim**: Это симулятор RTSP.
* **objectCounter**: Это модуль, который ищет определенные объекты в результатах, полученных от модуля yolov3.

Сведения о модуле objectCounter см. в строке (${MODULES.objectCounter}), используемой для значения "изображение". Это основано на [руководстве](../../iot-edge/tutorial-develop-for-linux.md) о разработке модуля IoT Edge. Visual Studio Code автоматически находит код модуля objectCounter в разделе src/edge/modules/objectCounter. 

Прочитайте [этот раздел](../../iot-edge/module-composition.md#declare-routes) посвященный объявлению маршрутов в манифесте развертывания IoT Edge. Затем проверьте маршруты в файле JSON шаблона. Обратите внимание на следующие моменты:

* LVAToObjectCounter используется для отправки определенных событий на определенную конечную точку в модуле objectCounter.
* ObjectCounterToLVA используется для отправки события-триггера на определенную конечную точку (которая должна быть узлом источника Центра Интернета вещей) в модуле lvaEdge.
* objectCounterToIoTHub используется в качестве средства отладки и помогает просматривать выходные данные модуля objectCounter при запуске кода из этого руководства.

> [!NOTE]
> Проверьте необходимые свойства модуля objectCounter, которые настроены для поиска объектов, помеченных как грузовики, с уровнем достоверности не ниже 50 %.

## <a name="generate-and-deploy-the-iot-edge-deployment-manifest"></a>Создание и развертывание манифеста развертывания IoT Edge 

Манифест развертывания определяет, какие именно модули развертываются на пограничном устройстве, а также содержит параметры конфигурации для этих модулей. Выполните перечисленные ниже действия, чтобы создать манифест на базе файла шаблона, а затем развернуть его на пограничном устройстве.

С помощью Visual Studio Code выполните [эти инструкции](../../iot-edge/tutorial-develop-for-linux.md#build-and-push-your-solution), чтобы войти в Docker. Потом выберите **Создание и отправка решения IoT Edge**. Используйте файл src/edge/deployment.objectCounter.template.json для этого шага.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/event-based-video-recording-tutorial/build-push.png" alt-text="Создание и отправка решения IoT Edge":::

Это действие создает модуль objectCounter для подсчета объектов и отправки изображения в Реестр контейнеров Azure.

* Убедитесь, что в файле .env определены переменные среды CONTAINER_REGISTRY_USERNAME_myacr и CONTAINER_REGISTRY_PASSWORD_myacr.

Этот шаг создает манифест развертывания IoT Edge в файле src/edge/config/deployment.objectCounter.amd64.json. Щелкните этот файл правой кнопкой и выберите **Create Deployment for Single Device** (Создание развертывания для одного устройства).

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/quickstarts/create-deployment-single-device.png" alt-text="Создание развертывания для одного устройства":::

Если вы раньше не работали с руководством по аналитике видеотрансляций в IoT Edge, среда Visual Studio Code предложит вам ввести строку подключения к Центру Интернета вещей. Ее можно скопировать из файла appsettings.json.

> [!NOTE]
> Возможно, вам будет предложено ввести сведения о встроенной конечной точке для Центра Интернета вещей. Чтобы получить эти сведения, на портале Azure перейдите в Центр Интернета вещей и найдите пункт **Встроенные конечные точки** в области навигации слева. Щелкните этот пункт и найдите значение **конечной точки, совместимой с центрами событий,** в разделе **Конечная точка, совместимая с концентратором событий**. В поле скопируйте текст и используйте его. Значение конечной точки будет приблизительно таким:  
    ```
    Endpoint=sb://iothub-ns-xxx.servicebus.windows.net/;SharedAccessKeyName=iothubowner;SharedAccessKey=XXX;EntityPath=<IoT Hub name>
    ```

Далее Visual Studio Code предложит выбрать устройство Центра Интернета вещей. Выберите свое устройство IoT Edge, которое должно быть устройством lva-sample-device.

На этом этапе начинается развертывание пограничных модулей на устройстве IoT Edge.
В течении 30 секунд обновите центр Интернета вещей Azure в левом нижнем углу раздела Visual Studio Code. Вы увидите, что разворачиваются четыре модуля с именами lvaEdge, rtspsim, yolov3, and objectCounter.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/event-based-video-recording-tutorial/iot-hub.png" alt-text="Четыре развернутых модуля":::

## <a name="prepare-for-monitoring-events"></a>Подготовка к мониторингу событий

Чтобы посмотреть события, поступающие от модуля objectCounter и от модуля аналитики видеотрансляции в IoT Edge, выполните указанные ниже действия:

1. Откройте область обозревателя в Visual Studio Code и найдите **Центр Интернета вещей Azure** в левом нижнем углу.
1. Разверните узел **Devices** (Устройства).
1. Щелкните правой кнопкой мыши на файл lva-sample-device и выберите **Start Monitoring Built-in Event Endpoint** (Запуск мониторинга встроенной конечной точки события).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/quickstarts/start-monitoring-iothub-events.png" alt-text="Запуск мониторинга встроенной конечной точки события":::

    > [!NOTE]
    > Возможно, вам будет предложено ввести сведения о встроенной конечной точке для Центра Интернета вещей. Чтобы получить эти сведения, на портале Azure перейдите в Центр Интернета вещей и найдите пункт **Встроенные конечные точки** в области навигации слева. Щелкните этот пункт и найдите значение **конечной точки, совместимой с центрами событий,** в разделе **Конечная точка, совместимая с концентратором событий**. В поле скопируйте текст и используйте его. Значение конечной точки будет приблизительно таким:  
        ```
        Endpoint=sb://iothub-ns-xxx.servicebus.windows.net/;SharedAccessKeyName=iothubowner;SharedAccessKey=XXX;EntityPath=<IoT Hub name>
        ```
    
## <a name="run-the-program"></a>Запуск программы

1. В Visual Studio Code откройте вкладку **Расширения** (или нажмите клавиши CTRL+SHIFT+X) и найдите центр Интернета вещей Azure.
1. Щелкните правой кнопкой мыши и выберите **Параметры расширения**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/extensions-tab.png" alt-text="Параметры расширения":::
1. Найдите и включите параметр "Show Verbose Message" (Показывать подробное сообщение).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/show-verbose-message.png" alt-text="Show Verbose Message"::: (Показывать подробное сообщение)
1. <!--In Visual Studio Code, go-->Перейдите к файлу src/cloud-to-device-console-app/operations.json.
1. В узле **GraphTopologySet** внесите следующие изменения:

    `"topologyUrl" : "https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/evr-hubMessage-assets/2.0/topology.json"`
    
1. Затем внесите изменения в узлах **GraphInstanceSet** и **GraphTopologyDelete**:

    `"topologyName" : "EVRtoAssetsOnObjDetect"`
1. Чтобы запустить сеанс отладки, нажмите клавишу F5. В окне **ТЕРМИНАЛ** начнут появляться сообщения.
1. Файл operations.json начнет выполнение операций с вызова GraphTopologyList и GraphInstanceList. Если вы очистили ресурсы после выполнения действий, описанных в предыдущих кратких руководствах или учебниках, это действие возвратит пустые списки, а затем выполнение будет приостановлено, пока вы не нажмете клавишу **ВВОД**, как показано ниже:

    ```
    --------------------------------------------------------------------------
    Executing operation GraphTopologyList
    -----------------------  Request: GraphTopologyList  --------------------------------------------------
    {
      "@apiVersion": "2.0"
    }
    ---------------  Response: GraphTopologyList - Status: 200  ---------------
    {
      "value": []
    }
    --------------------------------------------------------------------------
    Executing operation WaitForInput
    Press Enter to continue
    ```
1. После нажатия клавиши **ВВОД** в окне **ТЕРМИНАЛ** будет вызвана следующая серия прямых методов:
   * Вызов GraphTopologySet с использованием предыдущего topologyUrl
   * Вызов GraphInstanceSet с использованием приведенного ниже кода
     
        ```
        {
          "@apiVersion": "2.0",
          "name": "Sample-Graph-1",
          "properties": {
            "topologyName": "EVRtoAssetsOnObjDetect",
            "description": "Sample graph description",
            "parameters": [
              {
                "name": "rtspUrl",
                "value": "rtsp://rtspsim:554/media/camera-300s.mkv"
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
    
   * Вызов GraphInstanceActivate для запуска экземпляра графа и видеопотока
   * Второй вызов GraphInstanceList должен указывать, что экземпляр графа находится в состоянии выполнения
     
1. Вывод данных в окне **ТЕРМИНАЛ** будет приостановлен с предложением **Нажать клавишу ВВОД для продолжения**. Не нажимайте **ВВОД** на этом этапе. Прокрутите экран вверх, чтобы увидеть полезные данные ответов JSON на вызванные нами прямые методы.
1. Если теперь переключиться в окно **ВЫХОДНЫЕ ДАННЫЕ** в Visual Studio Code, вы увидите сообщения, отправляемые в Центр Интернета вещей модулем Аналитики видеотрансляции в IoT Edge.

   Эти сообщения рассматриваются в следующем разделе.
1. Экземпляр графы продолжит работу и запись видео. Симулятор RTSP будет продолжать циклический перебор исходного видео. Просмотрите сообщения как написано в следующем разделе. Затем, чтобы закрыть экземпляр, вернитесь в окно **ТЕРМИНАЛ** и выберите **ВВОД**. Следующая серия вызовов производится для очистки ресурсов используя:

   * Вызов GraphInstanceDeactivate для деактивации экземпляра графа.
   * Вызов GraphInstanceDelete для удаления экземпляра.
   * Вызов GraphTopologyDelete для удаления топологии.
   * Окончательный вызов GraphTopologyList, чтобы продемонстрировать, что список пуст.

## <a name="interpret-the-results"></a>Интерпретация результатов 

После запуска графа мультимедиа модуль аналитики видеотрансляции в IoT Edge отправляет в концентратор IoT Edge определенные диагностические и операционные события. Эти события являются сообщениями, отображаемыми в окне **ВЫХОДНЫЕ ДАННЫЕ** Visual Studio Code. Они содержат разделы body и applicationProperties. Сведения о назначении этих разделов см. в статье [Создание и чтение сообщений Центра Интернета вещей](../../iot-hub/iot-hub-devguide-messages-construct.md).

В приведенных сообщениях свойства приложений и содержимое раздела body определяются модулем Аналитики видеотрансляции.

## <a name="diagnostics-events"></a>События диагностики

### <a name="mediasessionestablished-event"></a>Событие MediaSessionEstablished 

После создания экземпляра графа узел источника RTSP пытается подключиться к серверу TRSP, работающему в контейнере симулятора RTSP. В случае успеха, это событие выводится на печать. Тип этого события — **Microsoft.Media.MediaGraph.Diagnostics.MediaSessionEstablished**.

```
[IoTHubMonitor] [5:53:17 PM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "sdp&quot;: &quot;SDP:\nv=0\r\no=- 1586450538111534 1 IN IP4 XXX.XX.XX.XX\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/camera-300s.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.03.06\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-300.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/camera-300s.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=4D0029;sprop-parameter-sets=XXXXXXXXXXXXXXXXXXXXXX\r\na=control:track1\r\n"
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{resource-group-name}/providers/microsoft.media/mediaservices/{ams-account-name}",
    "subject": "/graphInstances/Sample-Graph-1/sources/rtspSource",
    "eventType": "Microsoft.Media.Graph.Diagnostics.MediaSessionEstablished",
    "eventTime": "2020-05-17T17:53:16.981Z",
    "dataVersion": "1.0"
  }
}
```


* Сообщение является Событием диагностики (MediaSessionEstablished). Он указывает, что исходный узел RTSP (субъект) установил подключение к симулятору RTSP и начал получать (имитировать) динамический канал.
* Раздел субъекта в разделе свойств приложения applicationProperties обозначает узел в топологии графа, сгенерировавший сообщение. В этом случае сообщение исходит от узла источника RTSP.
* Раздел eventType в applicationProperties указывает, что событие является диагностическим.
* Раздел eventTime указывает время события. Это время, когда видеозапись дорожного движения (MKV-файл) начала поступать в модуль в режиме трансляции.
* Раздел body содержит информацию о диагностическом событии: в этом случае это данные [SDP](https://en.wikipedia.org/wiki/Session_Description_Protocol).


## <a name="operational-events"></a>Операционные события

Через некоторое время после запуска графа мультимедиа, поступит событие от модуля objectCounter. 

```
[IoTHubMonitor] [5:53:44 PM] Message received from [lva-sample-device/objectCounter]:
{
  "body": {
    "count": 2
  },
  "applicationProperties": {
    "eventTime&quot;: &quot;2020-05-17T17:53:44.062Z"
  }
}
```

Раздел applicationProperties содержит время события. Это время, когда модуль objectCounter наблюдает, что результаты из модуля yolov3 содержат объекты, представляющие интерес (грузовики).

По мере обнаружения на видео новых грузовиков будут возникать новые события этого типа.

### <a name="recordingstarted-event"></a>Событие RecordingStarted

Практически сразу после того, как счетчик объектов отправит события, мы увидим событие типа **Microsoft.Media.Graph.Operational.RecordingStarted**:

```
[IoTHubMonitor] [5:53:46 PM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "outputType": "assetName",
    "outputLocation&quot;: &quot;sampleAssetFromEVR-LVAEdge-20200517T175346Z"
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{resource-group-name}/providers/microsoft.media/mediaservices/{ams-account-name}",
    "subject": "/graphInstances/Sample-Graph-1/sinks/assetSink",
    "eventType": "Microsoft.Media.Graph.Operational.RecordingStarted",
    "eventTime": " 2020-05-17T17:53:46.132Z",
    "dataVersion&quot;: &quot;1.0"
  }
}
```

Раздел subject в разделе свойств приложения applicationProperties обозначает узел приемника ресурса в графе, сгенерировавший это сообщение. В разделе body содержатся сведения о расположении выходных данных. В этом случае, видео записывается в имя ресурса служб мультимедиа Azure. Запишите это значение.

### <a name="recordingavailable-event"></a>Событие RecordingAvailable

После того как узел приемника ресурса загружает видео в ресурс, он генерирует событие типа **Microsoft.Media.Graph.Operational.RecordingAvailable**:

```
[IoTHubMonitor] [5:54:15 PM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "outputType": "assetName",
    "outputLocation&quot;: &quot;sampleAssetFromEVR-LVAEdge-20200517T175346Z"
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{resource-group-name}/providers/microsoft.media/mediaservices/{ams-account-name}",
    "subject": "/graphInstances/Sample-Graph-1/sinks/assetSink",
    "eventType": "Microsoft.Media.Graph.Operational.RecordingAvailable",
    "eventTime": "2020-05-17T17:54:15.808Z",
    "dataVersion&quot;: &quot;1.0"
  }
}
```

Это событие означает, что в ресурс записано уже достаточно данных для того, чтобы на проигрывателях или клиентах можно было начать воспроизведение видео. Раздел subject в разделе свойств приложения applicationProperties определяет узел AssetSink в графе, сгенерировавший это сообщение. В разделе body содержатся сведения о расположении выходных данных. В этом случае, видео записывается в имя ресурса служб мультимедиа Azure.

### <a name="recordingstopped-event"></a>Событие RecordingStopped

Изучив параметры активации (maximumActivationTime) узла обработчика шлюза сигналов в [топологии](https://github.com/Azure/live-video-analytics/tree/master/MediaGraph/topologies/evr-hubMessage-assets/topology.json), мы увидим, что шлюз закрывается, пропустив через себя видео продолжительностью 30 секунд. Таким образом, примерно через 30 секунд после события RecordingStarted вы должны увидеть событие типа **Microsoft.Media.Graph.Operational.RecordingStopped**. Это событие означает, что узел приемника ресурса прекратил запись видео на ресурс.

```
[IoTHubMonitor] [5:54:15 PM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "outputType": "assetName",
    "outputLocation&quot;: &quot;sampleAssetFromEVR-LVAEdge-20200517T175346Z"
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{resource-group-name}/providers/microsoft.media/mediaservices/{ams-account-name}",
    "subject": "/graphInstances/Sample-Graph-1/sinks/assetSink",
    "eventType": "Microsoft.Media.Graph.Operational.RecordingStopped",
    "eventTime": "2020-05-17T17:54:15.040Z",
    "dataVersion&quot;: &quot;1.0"
  }
}
```

Это событие означает, что запись прекращена. Раздел subject в разделе свойств приложения applicationProperties определяет узел AssetSink в графе, сгенерировавший это сообщение. В разделе body содержатся сведения о расположении выходных данных. В этом случае, видео записывается в имя ресурса служб мультимедиа Azure.

## <a name="media-services-asset"></a>Актив Служб мультимедиа  

Чтобы проверить ресурс Служб мультимедиа, созданный графом, войдите на портал Azure и просмотрите видео.

1. В браузере откройте [портал Azure](https://portal.azure.com/). Введите свои учетные данные для входа на портал. Панель мониторинга службы является представлением по умолчанию.
1. Найдите свою учетную запись Служб мультимедиа среди ресурсов в составе вашей подписки. Откройте панель учетной записи.
1. Выберите **ресурсы** в списке **Служб мультимедиа**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/continuous-video-recording-tutorial/assets.png" alt-text="Непрерывная запись видео":::
1. Вы найдете ресурс с именем sampleAssetFromEVR-LVAEdge-{DateTime}. Это имя указанное в свойстве outputLocation события RecordingStarted. Алгоритм создания этого имени определяется свойством assetNamePattern топологии.
1. Выберите ресурс.
1. На странице подробных сведений о ресурсе выберите команду **Создать** под текстовым полем **URL-адрес потоковой передачи**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/continuous-video-recording-tutorial/new-asset.png" alt-text="Новый актив":::
1. В открывшемся мастере примите параметры по умолчанию и выберите **Добавить**. Дополнительные сведения см. в статье о [воспроизведении видео](video-playback-concept.md).

    > [!TIP]
    > Убедитесь, что [конечная точка потоковой передачи запущена](../latest/streaming-endpoint-concept.md).
1. Проигрыватель должен загрузить видео. Выберите **Воспроизвести**, чтобы просмотреть его.

> [!NOTE]
> Поскольку источник видео представлял собой контейнер, имитирующий поток данных с камеры, метки времени в видео обозначают моменты активации и деактивации экземпляра графа. Если вы используете элементы управления воспроизведением, созданные с помощью руководства по [Воспроизведению записей за несколько дней](playback-multi-day-recordings-tutorial.md), в видео на экране будут отображаться метки времени.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы планируете поработать и с другими руководствами, созданные ресурсы следует сохранить. В противном случае зайдите на портал Azure, откройте список групп ресурсов, выберите группу, в которой работали с этим руководством, и удалите ее.

## <a name="next-steps"></a>Дальнейшие действия

* Вместо симулятора RTSP используйте [IP-камеру](https://en.wikipedia.org/wiki/IP_camera) с поддержкой RTSP. Подобрать RTSP-совместимую IP-камеру можно на странице [продуктов, соответствующих спецификации ONVIF](https://www.onvif.org/conformant-products/): ищите устройства, совместимые с профилем G, S или T.
* Используйте устройство AMD64 или X64 на базе Linux (вместо виртуальной машины Linux в Azure). Устройство должно находиться в той же сети, что и IP-камера. Выполните инструкции из статьи [Install Azure IoT Edge runtime on Linux](../../iot-edge/how-to-install-iot-edge.md) (Установка среды выполнения Azure IoT Edge в системах Linux на основе Debian). Потом следуйте инструкциям из краткого руководства [Развертывание первого модуля IoT Edge на виртуальном устройстве Linux](../../iot-edge/quickstart-linux.md), чтобы зарегистрировать устройство в Центре Интернета вещей Azure.