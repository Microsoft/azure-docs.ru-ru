---
title: Анализ видеотрансляций с помощью Аналитики видеотрансляций в IoT Edge и Пользовательского визуального распознавания Azure
description: Сведения о том, как с помощью Пользовательского визуального распознавания Azure создать контейнерную модель, которая сможет определить игрушечный грузовик, а также применить расширяемость ИИ в Аналитике видеотрансляций Azure на основе Azure IoT Edge, чтобы развернуть на пограничном устройстве модель для обнаружения игрушечных грузовиков во время потоковой передачи видео.
ms.topic: tutorial
ms.date: 09/08/2020
zone_pivot_groups: ams-lva-edge-programming-languages
ms.openlocfilehash: 1abf123883a89bb41909e8aa67aedfadffc3d37e
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105561210"
---
# <a name="tutorial-analyze-live-video-with-live-video-analytics-on-iot-edge-and-azure-custom-vision"></a>Руководство по анализу видеотрансляций с помощью Аналитики видеотрансляций в IoT Edge и Пользовательского визуального распознавания Azure

В этом руководстве показано, как с помощью [Пользовательского визуального распознавания](https://azure.microsoft.com/services/cognitive-services/custom-vision-service/) Azure создать контейнерную модель, которая сможет определить игрушечный грузовик, а также применить [расширяемость ИИ](analyze-live-video-concept.md#analyzing-video-using-a-custom-vision-model) в Аналитике видеотрансляций Azure на основе Azure IoT Edge, чтобы развернуть на пограничном устройстве модель для обнаружения игрушечных грузовиков во время потоковой передачи видео.

Мы покажем, как разные возможности Пользовательского визуального распознавания объединяются для создания и обучения модели компьютерного зрения путем отправки и маркировки нескольких изображений. Знания в области обработки и анализа данных, машинного обучения и искусственного интеллекта не требуются. Вы также узнаете, как с помощью Аналитики видеотрансляций можно без усилий развернуть пользовательскую модель в качестве контейнера на пограничном устройстве для анализа имитируемой видеотрансляции.

::: zone pivot="programming-language-csharp"
[!INCLUDE [header](includes/custom-vision-tutorial/csharp/header.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [header](includes/custom-vision-tutorial/python/header.md)]
::: zone-end

В этом руководстве описаны следующие процедуры.

> [!div class="checklist"]
> * Настроите необходимые ресурсы.
> * создание в облаке и развертывание на пограничном устройстве модели Пользовательского визуального распознавания, которая обнаруживает игрушечные грузовики;
> * создание и развертывание графа мультимедиа с расширением HTTP в модель Пользовательского визуального распознавания;
> * Запустите пример кода.
> * Изучите и интерпретируйте результаты.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="suggested-pre-reading"></a>Рекомендуемые материалы для предварительного ознакомления  

Перед началом работы ознакомьтесь со следующими статьями:

* [Аналитика видеотрансляций в IoT Edge: обзор](overview.md)
* [Обзор Пользовательского визуального распознавания Azure](../../cognitive-services/custom-vision-service/overview.md)
* [Аналитика видеотрансляции в IoT Edge: терминология](terminology.md)
* [Граф мультимедиа: основные понятия](media-graph-concept.md)
* [Аналитика видеотрансляции без записи видео](analyze-live-video-concept.md)
* [Аналитика видеотрансляции с помощью собственной модели](use-your-model-quickstart.md)
* [Руководство. Разработка модуля IoT Edge](../../iot-edge/tutorial-develop-for-linux.md)
* [Редактирование файла deployment.*.template.json](https://github.com/microsoft/vscode-azure-iot-edge/wiki/How-to-edit-deployment.*.template.json)

## <a name="prerequisites"></a>Предварительные требования

::: zone pivot="programming-language-csharp"
[!INCLUDE [prerequisites](includes/custom-vision-tutorial/csharp/prerequisites.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [prerequisites](includes/custom-vision-tutorial/python/prerequisites.md)]
::: zone-end

> [!IMPORTANT]
> Этот модуль Пользовательского визуального распознавания поддерживает только архитектуры **Intel x86 и AMD64**. Прежде чем продолжить, проверьте архитектуру своего пограничного устройства.

## <a name="review-the-sample-video"></a>Просмотр примера видео

В этом руководстве для моделирования прямой трансляции используется файл [видео вывода игрушечного автомобиля](https://lvamedia.blob.core.windows.net/public/t2.mkv). Видео можно просмотреть с помощью таких приложений, как [проигрыватель мультимедиа VLC](https://www.videolan.org/vlc/). Нажмите клавиши **CTRL+N**, а затем вставьте ссылку на [видео вывода игрушечного автомобиля](https://lvamedia.blob.core.windows.net/public/t2.mkv), чтобы начать воспроизведение. Просматривая видео, обратите внимание на то, что игрушечный грузовик появляется на отметке времени, которая соответствует 36 секунде. Пользовательская модель обучена так, чтобы обнаруживать этот конкретный игрушечный грузовик. 

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4LPwK]

В этом руководстве показано, как использовать Аналитику видеотрансляций в IoT Edge, чтобы обнаруживать игрушечные грузовики и публиковать соответствующие события вывода в центре IoT Edge.

## <a name="overview"></a>Обзор

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/custom-vision-tutorial/topology-custom-vision.svg" alt-text="Схема: обзор Пользовательского визуального распознавания":::

На схеме показан порядок передачи сигналов в этом руководстве. [Пограничный модуль](https://github.com/Azure/live-video-analytics/tree/master/utilities/rtspsim-live555) имитирует IP-камеру, на которой находится RTSP-сервер. Узел [RTSP-источника](media-graph-concept.md#rtsp-source) получает видеосигнал с этого сервера и передает видеокадры на узел [обработчика расширений HTTP](media-graph-concept.md#http-extension-processor).

Узел расширения HTTP играет роль прокси-сервера.  Он выбирает входящие видеокадры, заданные с помощью поля `samplingOptions`, а также преобразует видеокадры в изображения указанного типа. Затем с использованием REST он передает изображение на другой пограничный модуль, выполняющий модель искусственного интеллекта на конечной точке HTTP. В этом примере пограничный модуль является моделью средства обнаружения игрушечного грузовика, созданной с помощью Пользовательского визуального распознавания. Узел обработчика расширений HTTP собирает результаты обнаружения и публикует события в узле [приемника Центра Интернета вещей Azure](media-graph-concept.md#iot-hub-message-sink). Затем этот узел отправляет события в [центр IoT Edge](../../iot-fundamentals/iot-glossary.md#iot-edge-hub).

## <a name="build-and-deploy-a-custom-vision-toy-detection-model"></a>Создание и развертывание модели обнаружения игрушки Пользовательского визуального распознавания 

Как нам подсказывает имя службы, Пользовательское визуальное распознавание можно использовать для создания собственного облачного средства для обнаружения объектов или их классификации. Служба предоставляет простой и понятный интерфейс для создания моделей Пользовательского визуального распознавания, которые можно развернуть в облаке или на пограничном устройстве с помощью контейнеров.

Чтобы создать средство обнаружения игрушечных грузовиков, выполните действия, описанные в кратком руководстве [ Создание средства обнаружения объектов с помощью веб-сайта Пользовательского визуального распознавания](../../cognitive-services/custom-vision-service/get-started-build-detector.md).

> [!IMPORTANT]
> Этот модуль Пользовательского визуального распознавания поддерживает только архитектуры **Intel x86 и AMD64**. Прежде чем продолжить, проверьте архитектуру своего пограничного устройства.

Дополнительные замечания
 
* Для работы с этим руководством нам не нужны примеры изображений, предоставленные в разделе [Предварительные требования](../../cognitive-services/custom-vision-service/get-started-build-detector.md#prerequisites). Вместо них мы использовали определенный набор изображений для создания модели Пользовательского визуального распознавания для обнаружения игрушек. Используйте [эти изображения](https://lvamedia.blob.core.windows.net/public/ToyCarTrainingImages.zip), когда в кратком руководстве будет предложено [выбрать изображения для обучения](../../cognitive-services/custom-vision-service/get-started-build-detector.md#choose-training-images).
* В разделе краткого руководства, посвященном добавлению тегов, обязательно добавьте тег delivery truck для изображения игрушечного грузовика.

Завершив этот процесс, вы можете экспортировать модель в контейнер Docker с помощью кнопки **Экспорт** на вкладке **Производительность**. Убедитесь, что в качестве типа платформы для контейнера выбрана платформа Linux. Это платформа, на которой будет выполняться контейнер. Компьютер, на который вы загружаете контейнер, может работать как под управлением Windows, так и Linux. Приведенные ниже инструкции основаны на файле контейнера, загруженном на компьютер Windows.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/custom-vision-tutorial/docker-file.png" alt-text="Снимок экрана: выбранный файл Dockerfile":::
 
1. На локальный компьютер следует загрузить ZIP-файл с именем `<projectname>.DockerFile.Linux.zip`. 
1. Проверьте, установлено ли средство Docker. Если нет, установите [Docker](https://docs.docker.com/get-docker/) на локальный компьютер Windows.
1. Распакуйте загруженный файл в выбранном расположении. В командной строке перейдите к каталогу распакованных папок.
    
    Выполните следующие команды:
    
    1. `docker build -t cvtruck` 
    
        Эта команда скачивает много пакетов, создает образ Docker и присваивает ему тег `cvtruck:latest`.
    
        > [!NOTE]
        > В случае успешного выполнения отобразятся сообщения `Successfully built <docker image id>` и `Successfully tagged cvtruck:latest`. Если команда сборки завершится неудачно, повторите попытку. Иногда пакеты зависимостей не скачиваются с первой попытки.
    1. `docker  image ls`

        С помощью этой команды можно проверить, находится ли новый образ в локальном реестре.
    1. `docker run -p 127.0.0.1:80:80 -d cvtruck`
    
        С помощью этой команды можно опубликовать открытый порт Docker (80) под номером порта локального компьютера (80).
    1. `docker container ls`
    
        С помощью этой команды можно проверить сопоставление портов и убедиться в том, что контейнер Docker успешно работает на вашем компьютере. Результат должен выглядеть следующим образом.

        ```
        CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
        8b7505398367        cvtruck             "/bin/sh -c 'python …"   13 hours ago        Up 25 seconds       127.0.0.1:80->80/tcp   practical_cohen
        ```
      1. `curl -X POST http://127.0.0.1:80/image -F imageData=@<path to any image file that has the toy delivery truck in it>`
            
            Эта команда проверяет контейнер на локальном компьютере. Если изображение содержит тот же грузовой автомобиль, с использованием которого мы обучали модель, выходные данные должны выглядеть примерно так, как показано в следующем примере. Здесь мы видим, что грузовой автомобиль был обнаружен с вероятностью 90,12 %.
    
            ```
            {"created":"2020-03-20T07:10:47.827673","id":"","iteration":"","predictions":[{"boundingBox":{"height":0.66167289,"left":-0.03923762,"top":0.12781593,"width":0.70003178},"probability":0.90128148,"tagId":0,"tagName":"delivery truck"},{"boundingBox":{"height":0.63733053,"left":0.25220079,"top":0.0876643,"width":0.53331227},"probability":0.59745145,"tagId":0,"tagName":"delivery truck"}],"project":""}
            ```

<!--## Create and deploy a media graph with http extension to custom vision model-->

## <a name="examine-the-sample-files"></a>Изучение примеров файлов

::: zone pivot="programming-language-csharp"
[!INCLUDE [examine-sample-files](includes/custom-vision-tutorial/csharp/examine-sample-files.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [examine-sample-files](includes/custom-vision-tutorial/python/examine-sample-files.md)]
::: zone-end

## <a name="generate-and-deploy-the-deployment-manifest"></a>Создание и развертывание манифеста развертывания

1. В Visual Studio Code перейдите к файлу src/cloud-to-device-console-app/operations.json.

1. В разделе `GraphTopologySet` убедитесь, что соблюдены следующие условия:<br/>`"topologyUrl" : "https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/httpExtension/2.0/topology.json"`
1. В разделе `GraphInstanceSet` проверьте следующее.
    1. `"topologyName" : "InferencingWithHttpExtension"`
    1. Добавьте следующий код в начало массива параметров `{"name": "inferencingUrl","value": "http://cv:80/image"},`.
    1. Измените значение параметра `rtspUrl` на `"rtsp://rtspsim:554/media/t2.mkv"`.
1. В разделе `GraphTopologyDelete` проверьте `"name": "InferencingWithHttpExtension"`.
1. Щелкните правой кнопкой мыши файл src/edge/deployment.customvision.template.json и выберите **Создать манифест развертывания IoT Edge**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/custom-vision-tutorial/deployment-template-json.png" alt-text="Снимок экрана: команда &quot;Создать манифеста развертывания IoT Edge&quot;":::
  
    В результате этого действия в папке src/edge/config будет создан файл манифеста с именем deployment.customvision.amd64.json.
1. Откройте файл src/edge/deployment.customvision.template.json и найдите JSON-блок `registryCredentials`. В этом блоке вы увидите адрес реестра, имя пользователя и пароль для контейнера Azure.
1. Отправьте локальный контейнер Пользовательского визуального распознавания в Реестр контейнеров Azure, выполнив в командной строке следующие шаги:

    1. Войдите в реестр, выполнив следующую команду:
    
        `docker login <address>`
    
        При появлении запроса на проверку подлинности введите имя пользователя и пароль.
        
        > [!NOTE]
        > Пароль не отображается в командной строке.
    1. Присвойте изображению тег с помощью следующей команды: <br/>`docker tag cvtruck   <address>/cvtruck`.
    1. Отправьте это изображение с помощью следующей команды: <br/>`docker push <address>/cvtruck`.

        В случае успешного выполнения вы должны увидеть в командной строке сообщение `Pushed`, а также значение SHA для изображения.
    1. Вы также можете проверить результат выполнения в Реестре контейнеров Azure с помощью портала Azure. Вы должны увидеть имя репозитория и присвоенный ему тег.
1. Задайте строку подключения к Центру Интернета вещей, щелкнув значок **Дополнительные действия** рядом с областью **ЦЕНТР ИНТЕРНЕТА ВЕЩЕЙ AZURE** в левом нижнем углу. Строку можно скопировать из файла appsettings.json. (Есть еще один рекомендуемый подход, позволяющий обеспечивать правильность настройки Центра Интернета вещей в Visual Studio Code. Он предполагает использование команды [Задать строку подключения Центра Интернета вещей](https://github.com/Microsoft/vscode-azure-iot-toolkit/wiki/Select-IoT-Hub).)

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/custom-vision-tutorial/connection-string.png" alt-text="Снимок экрана: команда &quot;Задать строку подключения Центра Интернета вещей&quot;":::
1. Затем щелкните правой кнопкой мыши файл src/edge/config/deployment.customvision.amd64.json и выберите **Создать развертывание для одного устройства**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/custom-vision-tutorial/deployment-amd64-json.png" alt-text="Снимок экрана: команда &quot;Создать развертывание для одного устройства&quot;":::
1. Вам будет предложено выбрать устройство Центра Интернета вещей. Выберите устройство **lva-sample-device** в раскрывающемся списке.
1. Примерно через 30 секунд обновите сведения о центре Интернета вещей Azure в разделе внизу слева. Вы увидите пограничное устройство, на котором развернуты следующие модули:

    * модуль Аналитики видеотрансляций в IoT Edge с именем `lvaEdge`;
    * модуль с именем `rtspsim`, который имитирует сервер RTSP и создает сигнала видеотрансляции;
    * модуль с именем `cv`, который в соответствии с этим именем является моделью Пользовательского визуального распознавания для обнаружения игрушечных грузовиков, которая применяет Пользовательское визуальное распознавание к изображениям и возвращает несколько типов тегов. (Наша модель обучена только по тегу delivery truck.)

## <a name="prepare-for-monitoring-events"></a>Подготовка к мониторингу событий

Щелкните правой кнопкой мыши устройство Аналитики видеотрансляции и выберите **Запуск мониторинга встроенной конечной точки события**. Этот шаг необходим для мониторинга событий Центра Интернета вещей в окне **ВЫХОДНЫЕ ДАННЫЕ** в Visual Studio Code.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/custom-vision-tutorial/start-monitoring.png" alt-text="Снимок экрана: команда &quot;Запуск мониторинга встроенной конечной точки событий&quot;":::

## <a name="run-the-sample-program"></a>Запуск примера программы

Если вы откроете в браузере топологию графа для этого руководства, вы увидите, что для параметра `inferencingUrl` установлено значение `http://cv:80/image`. Это означает, что сервер вывода будет возвращать результаты после обнаружения в видеотрансляции игрушечных грузовиков (если они есть).

1. В Visual Studio Code откройте вкладку **Расширения** (или нажмите клавиши **CTRL+SHIFT+X**) и найдите Центр Интернета вещей Azure.
1. Щелкните правой кнопкой мыши и выберите **Параметры расширения**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/extensions-tab.png" alt-text="Снимок экрана: раздел &quot;Параметры расширения&quot;":::
1. Найдите и включите параметр **Показывать подробное сообщение**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/run-program/show-verbose-message.png" alt-text="Снимок экрана: параметр &quot;Показывать подробное сообщение&quot;":::
1. Чтобы начать сеанс отладки, нажмите клавишу **F5**. В окне **ТЕРМИНАЛ** отображаются выводимые сообщения.
1. Код operations.json начинается с вызовов прямых методов `GraphTopologyList` и `GraphInstanceList`. Если вы очистили ресурсы после работы с предыдущими краткими руководствами, тогда этот процесс возвратит пустые списки и приостановит работу. Чтобы продолжить, нажмите клавишу **ВВОД**.
    
   В окне **ТЕРМИНАЛ** показывается следующий набор вызовов прямых методов:
    
   * Вызов `GraphTopologySet`, который использует указанный выше `topologyUrl`.
   * Вызов `GraphInstanceSet`, который использует такой код:
        
   ```
        {
          "@apiVersion": "2.0",
          "name": "Sample-Graph-1",
          "properties": {
            "topologyName": "CustomVisionWithHttpExtension",
            "description": "Sample graph description",
            "parameters": [
              { 
                "name": "inferencingUrl",
                "value": "http://cv:80/image"
              },
              {
                "name": "rtspUrl",
                "value": "rtsp://rtspsim:554/media/t2.mkv"
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
    
   * Вызов к `GraphInstanceActivate`, который активирует экземпляр графа и поток видео.
   * Второй вызов к `GraphInstanceList`, который показывает, что экземпляр графа находится в состоянии выполнения.
    
1. Вывод данных в окне **Терминал** будет приостановлен, и появится запрос **Нажмите клавишу "Ввод", чтобы продолжить**. Пока не нажимайте клавишу **ВВОД**. Прокрутите экран вверх, чтобы увидеть полезные данные ответов JSON на вызванные нами прямые методы.
1. Перейдите в окно **ВЫХОДНЫЕ ДАННЫЕ** в Visual Studio Code. Вы увидите сообщения, которые модуль Аналитики видеотрансляций в IoT Edge передает в центр Интернета вещей. Эти сообщения рассматриваются в следующем разделе этого учебника.
1. Граф мультимедиа продолжит работать и выводить результаты. Симулятор RTSP будет продолжать циклический перебор исходного видео. Чтобы остановить граф мультимедиа, вернитесь в окно **Терминал** и нажмите клавишу **ВВОД**.
Следующая серия вызовов очищает ресурсы:
    
   * Вызов `GraphInstanceDeactivate` деактивирует экземпляр графа.
   * Вызов `GraphInstanceDelete` удаляет этот экземпляр.
   * Вызов `GraphTopologyDelete` удаляет топологию.
   * Окончательный вызов `GraphTopologyList` показывает, что список теперь пуст.
    
## <a name="interpret-the-results"></a>Интерпретация результатов

При запуске графа мультимедиа результаты из узла обработчика расширения HTTP отправляются через узел-преемник Центра Интернета вещей в центр Интернета вещей. Сообщения, отображаемые в окне **Выходные данные**, содержат разделы body и `applicationProperties`. Дополнительные сведения см. в статье [Создание и чтение сообщений Центра Интернета вещей](../../iot-hub/iot-hub-devguide-messages-construct.md).

В приведенных ниже сообщениях модуль Аналитики видеотрансляций определяет свойства приложения и содержимое раздела body.

### <a name="mediasessionestablished-event"></a>Событие MediaSessionEstablished

После создания экземпляра графа мультимедиа узел источника RTSP пытается подключиться к RTSP-серверу, работающему в контейнере rtspsim-live555. Если соединение установлено, будет выводиться следующее событие. Тип этого события — **Microsoft.Media.MediaGraph.Diagnostics.MediaSessionEstablished**.

```
{
  "body": {
    "sdp": "SDP:\nv=0\r\no=- 1599412849822466 1 IN IP4 172.18.0.3\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/t2.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.04.12\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-78.357\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/t2.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=42C01F;sprop-parameter-sets=Z0LAH9kAUAW6EAAAAwAQAAADAwDxgySA,aMuBcsg=\r\na=control:track1\r\nm=audio 0 RTP/AVP 97\r\nc=IN IP4 0.0.0.0\r\nb=AS:96\r\na=rtpmap:97 MPEG4-GENERIC/44100/2\r\na=fmtp:97 streamtype=5;profile-level-id=1;mode=AAC-hbr;sizelength=13;indexlength=3;indexdeltalength=3;config=121056E500\r\na=control:track2\r\n"
  },
  "applicationProperties": {
    "topic": "/subscriptions/35c2594a-23da-4fce-b59c-f6fb9513eeeb/resourceGroups/lvaavi_0827/providers/microsoft.media/mediaservices/lvasampleulf2rv2x5msy2",
    "subject": "/graphInstances/ GRAPHINSTANCENAMEHERE /sources/rtspSource",
    "eventType": "Microsoft.Media.Graph.Diagnostics.MediaSessionEstablished",
    "eventTime": "2020-09-06T17:20:49.823Z",
    "dataVersion": "1.0"
  }
```

В этом сообщении обратите внимание на следующие сведения:

* Сообщение является событием диагностики. `MediaSessionEstablished` указывает, что исходный узел RTSP (subject) установил подключение к симулятору RTSP и начал прием имитируемого потока данных.
* В `applicationProperties` `subject` указывает, что сообщение было создано из узла RTSP-источника в графе мультимедиа.
* Тип события в `applicationProperties` указывает, что это событие диагностики.
* Параметр времени указывает время возникновения этого события.
* Текст содержит данные о событии диагностики. В этом случае данные содержат сведения [протокола SDP](https://en.wikipedia.org/wiki/Session_Description_Protocol).

### <a name="inference-event"></a>Событие вывода

Узел обработчика расширений HTTP получает результаты вывода из контейнера Пользовательского визуального распознавания и выдает результаты через узел приемника Центра Интернета вещей как события вывода.

```
{
  "body": {
    "created": "2020-05-18T01:08:34.483Z",
    "id": "",
    "iteration": "",
    "predictions": [
      {
        "boundingBox": {
          "height": 0.06219418,
          "left": 0.14977954,
          "top": 0.65847444,
          "width": 0.01879117
        },
        "probability": 0.69806677,
        "tagId": 0,
        "tagName": "delivery truck"
      },
      {
        "boundingBox": {
          "height": 0.1148454,
          "left": 0.77430711,
          "top": 0.54488315,
          "width": 0.11315252
        },
        "probability": 0.29394844,
        "tagId": 0,
        "tagName": "delivery truck"
      },
      {
        "boundingBox": {
          "height": 0.03187029,
          "left": 0.62644471,
          "top": 0.59640052,
          "width": 0.01582896
        },
        "probability": 0.14435098,
        "tagId": 0,
        "tagName": "delivery truck"
      },
      {
        "boundingBox": {
          "height": 0.11421723,
          "left": 0.72737214,
          "top": 0.55907888,
          "width": 0.12627538
        },
        "probability": 0.1141128,
        "tagId": 0,
        "tagName": "delivery truck"
      },
      {
        "boundingBox": {
          "height": 0.04717135,
          "left": 0.66516746,
          "top": 0.34617995,
          "width": 0.03802613
        },
        "probability": 0.10461298,
        "tagId": 0,
        "tagName": "delivery truck"
      },
      {
        "boundingBox": {
          "height": 0.20650843,
          "left": 0.56349564,
          "top": 0.51482571,
          "width": 0.35045707
        },
        "probability": 0.10056595,
        "tagId": 0,
        "tagName": "delivery truck"
      }
    ],
    "project": ""
  },
  "applicationProperties": {
    "topic": "/subscriptions/35c2594a-23da-4fce-b59c-f6fb9513eeeb/resourceGroups/lsravi/providers/microsoft.media/mediaservices/lvasamplerc3he6a7uhire",
    "subject": "/graphInstances/GRAPHINSTANCENAMEHERE/processors/httpExtension",
    "eventType": "Microsoft.Media.Graph.Analytics.Inference",
    "eventTime": "2020-05-18T01:08:34.038Z"
  }
}
```

В представленных выше сообщениях обратите внимание на следующие сведения:

* subject в `applicationProperties` указывает на узел в графе мультимедиа MediaGraph, из которого было создано сообщение. В нашем примере сообщение исходит от обработчика расширений HTTP.
* Тип события в `applicationProperties` указывает, что это событие вывода аналитики.
* Параметр времени указывает время возникновения этого события.
* Текст сообщения содержит информацию об аналитическом событии. В нашем примере представлено событие вывода, поэтому текст содержит массив выводов (predictions).
* Раздел predictions содержит список предположений о наличии игрушечного грузовика в кадре (тег delivery truck). Как вы помните, delivery truck — это пользовательский тег, который вы предоставили для игрушечного грузовика в пользовательской обученной модели. Модель выводит и обнаруживает игрушечный грузовик в поступающей трансляции с разными показателями достоверности.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы планируете работать с другими руководствами или материалами, сохраните созданные ресурсы. В противном случае перейдите на портал Azure, откройте список групп ресурсов, выберите группу, которую вы использовали для этого руководства, и удалите все ресурсы.

## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь с дополнительными задачами для опытных пользователей.

* Вместо симулятора RTSP используйте [IP-камеру](https://en.wikipedia.org/wiki/IP_camera) с поддержкой RTSP. IP-камеры, поддерживающие протокол RTSP, можно найти на странице [продуктов, соответствующих ONVIF](https://www.onvif.org/conformant-products/). Ищите устройства, которые соответствуют профилям G, S или T.
* Используйте устройство AMD64 или x64 на базе Linux вместо виртуальной машины Linux в Azure. Устройство должно находиться в той же сети, что и IP-камера. Вы можете выполнить инструкции из статьи [Install Azure IoT Edge runtime on Linux](../../iot-edge/how-to-install-iot-edge.md) (Установка среды выполнения Azure IoT Edge в системах Linux на основе Debian).

Чтобы зарегистрировать устройство в Центре Интернета вещей Azure, следуйте инструкциям краткого руководства [Развертывание первого модуля IoT Edge на виртуальном устройстве Linux](../../iot-edge/quickstart-linux.md).