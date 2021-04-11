---
ms.openlocfilehash: 8a9149119bc754ff0715f2841925da01301faecd
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101750478"
---
### <a name="examine-and-edit-the-sample-files"></a>Изучение и изменение образцов файлов

Согласно предварительным требованиям вы загрузили в папку пример кода. Выполните указания ниже, чтобы изучить и изменить файлы примеров.

1. В Visual Studio Code перейдите в папку *src/edge*. Здесь находится созданный нами *ENV*-файл и несколько файлов шаблона развертывания.

    Шаблон развертывания deployment.grpcyolov3icpu.template.json содержит манифест развертывания для пограничного устройства. Он содержит некоторые значения-заполнители. В ENV-файле указаны значения таких переменных.
1. Теперь перейдите в папку *src/cloud-to-device-console-app*. Здесь вы увидите файл *appSettings.json* и несколько других файлов:
    
    * c2d-console-app.csproj — файл проекта Visual Studio Code.
    * operations.json — список операций, которые должна запускать программа.
    * Program.cs — пример кода программы. Этот код выполняет следующие действия:

        * загружает параметры приложения;
        * вызывает прямые методы, которые представляются модулем Аналитики видеотрансляций в IoT Edge (с помощью модуля можно анализировать потоки видеотрансляции, вызывая его прямые методы);
        * приостанавливает выполнение, позволяя проанализировать выходные данные программы в окне **ТЕРМИНАЛ** и проверить сгенерированные модулем события в окне **ВЫХОДНЫЕ ДАННЫЕ**;
        * вызывает прямые методы для очистки ресурсов.
1. Внесите правки в файл *operations.json*:
 
    * Измените ссылку на топологию графа:
    * `"topologyUrl"` : `"https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/motion-with-grpcExtension/2.0/topology.json"`
    * В разделе GraphInstanceSet измените имя топологии графа, чтобы оно совпадало со значением в предыдущей ссылке:
    * `"topologyName"` : `"EVROnMotionPlusGrpcExtension"`
    * В разделе GraphTopologyDelete измените имя:
    * `"name"` : `"EVROnMotionPlusGrpcExtension"`

> [!NOTE]
> <p>
> <details>
> <summary>Разверните это примечание и узнайте, как в топологии реализуется узел MediaGraphGrpcExtension.</summary>
> <pre><code>
> {
>   "@type": "#Microsoft.Media.MediaGraphGrpcExtension",
>   "name": "grpcExtension",
>   "endpoint": {
>       "@type": "#Microsoft.Media.MediaGraphUnsecuredEndpoint",
>       "url": "${grpcExtensionAddress}",
>       "credentials": {
>           "@type": "#Microsoft.Media.MediaGraphUsernamePasswordCredentials",
>           "username": "${grpcExtensionUserName}",
>           "password": "${grpcExtensionPassword}"
>       }
>   },
>   "dataTransfer": {
>       "mode": "sharedMemory",
>       "SharedMemorySizeMiB": "256"
>   },
>   "image": {
>       "scale": {
>           "mode": "${imageScaleMode}",
>           "width": "${frameWidth}",
>           "height": "${frameHeight}"
>       },
>       "format": {
>           "@type": "#Microsoft.Media.MediaGraphImageFormatEncoded",
>           "encoding": "${imageEncoding}",
>           "quality": "${imageQuality}"
>       }
>   },
>   "inputs": [
>       {
>           "nodeName": "motionDetection"
>       }
>   ]
> }          
> </code></pre>
> </details>    
> </p>

### <a name="generate-and-deploy-the-iot-edge-deployment-manifest"></a>Создание и развертывание манифеста развертывания IoT Edge

1. Щелкните правой кнопкой мыши файл *src/edge/* *deployment.grpcyolov3icpu.template.json* и выберите **Generate IoT Edge Deployment Manifest** (Создать манифест развертывания IoT Edge).

    ![Создание манифеста развертывания IoT Edge](../../../media/quickstarts/generate-iot-edge-deployment-manifest-grpc.png)

    Файл манифеста *deployment.grpcyolov3icpu.amd64.json* создается в папке *src/edge/config*.
1. Если вы прошли краткое руководство [Создание событий при обнаружении движения](../../../detect-motion-emit-events-quickstart.md), пропустите этот шаг.

    В противном случае рядом с областью **Центр Интернета вещей Azure** в левом нижнем углу щелкните значок **Дополнительные действия**, а затем выберите **Установка строки подключения Центра Интернета вещей**. Строку можно скопировать из файла *appsettings.json*. Чтобы обеспечить правильность настройки центра Интернета вещей в Visual Studio Code, используйте команду [Выбрать центр Интернета вещей](https://github.com/Microsoft/vscode-azure-iot-toolkit/wiki/Select-IoT-Hub).

    ![Строка подключения к Центру Интернета вещей](../../../media/quickstarts/iot-hub-connection-string-grpc.png)

    > [!NOTE]
    > Возможно, вам будет предложено ввести сведения о встроенной конечной точке для Центра Интернета вещей. Чтобы получить эти сведения, на портале Azure перейдите в Центр Интернета вещей и найдите пункт **Встроенные конечные точки** в области навигации слева. Щелкните этот пункт и найдите значение **конечной точки, совместимой с центрами событий,** в разделе **Конечная точка, совместимая с концентратором событий**. В поле скопируйте текст и используйте его. Значение конечной точки будет приблизительно таким:  
        ```
        Endpoint=sb://iothub-ns-xxx.servicebus.windows.net/;SharedAccessKeyName=iothubowner;SharedAccessKey=XXX;EntityPath=<IoT Hub name>
        ```

1. Щелкните правой кнопкой мыши файл *src/edge/config/* *deployment.grpcyolov3icpu.amd64.json* и выберите **Create Deployment for Single Device** (Создать развертывание для одного устройства).

    ![Создание развертывания для одного устройства](../../../media/quickstarts/create-deployment-single-device-grpc.png)
1. Когда появится запрос на выбор устройства Центра Интернета вещей, выберите **lva-sample-device**.
1. Примерно через 30 секунд обновите содержимое Центра Интернета вещей в левом нижнем углу окна. Теперь в пограничном устройстве должны отображаться следующие развернутые модули:

    * Модуль Аналитики видеотрансляций **lvaEdge**.
    * Модуль **rtspsim**, имитирующий RTSP-сервер, который выступает в качестве источника веб-канала видеотрансляции.

        > [!NOTE]
        > В приведенных выше шагах предполагается, что вы используете виртуальную машину, созданную с помощью скрипта настройки. Если вы используете собственное пограничное устройство, перейдите на это устройство и выполните следующие команды с **правами администратора**, чтобы извлечь и сохранить пример видеофайла, используемый для этого краткого руководства.  

        ```
        mkdir /home/lvaedgeuser/samples
        mkdir /home/lvaedgeuser/samples/input    
        curl https://lvamedia.blob.core.windows.net/public/camera-300s.mkv > /home/lvaedgeuser/samples/input/camera-300s.mkv  
        chown -R lvalvaedgeuser:localusergroup /home/lvaedgeuser/samples/  
        ```
    * Модуль **lvaExtension**, являющийся моделью обнаружения объектов YOLOv3, которая использует gRPC для обмена данными, применяет компьютерное зрение к изображениям и возвращает несколько классов типов объектов.
    
    ![Расширение lva](../../../media/quickstarts/lvaextension-grpc.png)

### <a name="prepare-to-monitor-events"></a>Подготовка к мониторингу событий

1. В Visual Studio Code откройте вкладку **Расширения** (или нажмите клавиши CTRL+SHIFT+X) и найдите центр Интернета вещей Azure.
1. Щелкните правой кнопкой мыши и выберите **Параметры расширения**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="../../../media/run-program/extensions-tab.png" alt-text="Параметры расширения":::
1. Найдите и включите параметр "Show Verbose Message" (Показывать подробное сообщение).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="../../../media/run-program/show-verbose-message.png" alt-text="Show Verbose Message"::: (Показывать подробное сообщение)
1. Щелкните правой кнопкой мыши устройство Аналитики видеотрансляции и выберите **Запуск мониторинга встроенной конечной точки события**. Этот шаг необходим для мониторинга событий Центра Интернета вещей в окне **ВЫХОДНЫЕ ДАННЫЕ** в Visual Studio Code.

   ![Запуск мониторинга](../../../media/quickstarts/start-monitoring-built-event-endpoint-grpc.png)

    > [!NOTE]
    > Возможно, вам будет предложено ввести сведения о встроенной конечной точке для Центра Интернета вещей. Чтобы получить эти сведения, на портале Azure перейдите в Центр Интернета вещей и найдите пункт **Встроенные конечные точки** в области навигации слева. Щелкните этот пункт и найдите значение **конечной точки, совместимой с центрами событий,** в разделе **Конечная точка, совместимая с концентратором событий**. В поле скопируйте текст и используйте его. Значение конечной точки будет приблизительно таким:  
        ```
        Endpoint=sb://iothub-ns-xxx.servicebus.windows.net/;SharedAccessKeyName=iothubowner;SharedAccessKey=XXX;EntityPath=<IoT Hub name>
        ```
### <a name="run-the-sample-program"></a>Запуск примера программы

1. Чтобы начать сеанс отладки, нажмите клавишу F5. В окне ТЕРМИНАЛ отображаются выводимые сообщения.
1. Код *operations.json* начинается с вызовов прямых методов `GraphTopologyList` и `GraphInstanceList`. Если вы очистили ресурсы после работы с предыдущими краткими руководствами, тогда этот процесс возвратит пустые списки, а затем приостановится. Чтобы продолжить, нажмите клавишу ВВОД.
    
    ```
    -------------------------------Executing operation GraphTopologyList-----------------------  
    Request: GraphTopologyList  --------------------------------------------------
    {
    "@apiVersion": "2.0"
    }
    ---------------  
    Response: GraphTopologyList - Status: 200  ---------------
    {
    "value": []
    }
    --------------------------------------------------------------------------
    Executing operation WaitForInput
    Press Enter to continue
    ```
    
    В окне **ТЕРМИНАЛ** показывается следующий набор вызовов прямых методов:
    
    * Вызов к `GraphTopologySet`, который использует предыдущее значение topologyUrl.
    * Вызов `GraphInstanceSet`, который использует такой код:
    
    ```
    {
      "@apiVersion": "2.0",
      "name": "Sample-Graph-1",
      "properties": {
        "topologyName": "EVROnMotionPlusGrpcExtension",
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
    
    * Вызов к `GraphInstanceActivate`, который активирует экземпляр графа и поток видео.
    * Второй вызов к `GraphInstanceList`, который показывает, что экземпляр графа находится в состоянии выполнения.
1. Вывод данных в окне **Терминал** будет приостановлен с предложением нажать клавишу ВВОД для продолжения. Не нажимайте клавишу ВВОД на этом этапе. Прокрутите экран вверх, чтобы увидеть полезные данные ответов JSON на вызванные нами прямые методы.
1. Перейдите в окно **ВЫХОДНЫЕ ДАННЫЕ** в Visual Studio Code. Вы увидите сообщения, которые модуль Аналитики видеотрансляций в IoT Edge передает в центр Интернета вещей. Эти сообщения рассматриваются в следующем разделе этого краткого руководства.
1. Граф мультимедиа продолжит работать и выводить результаты. Симулятор RTSP будет продолжать циклический перебор исходного видео. Чтобы остановить граф мультимедиа, вернитесь к окну **ТЕРМИНАЛ** и нажмите клавишу ВВОД.
1. Следующая серия вызовов очищает ресурсы:
    
    * Вызов `GraphInstanceDeactivate` деактивирует экземпляр графа.
    * Вызов `GraphInstanceDelete` удаляет этот экземпляр.
    * Вызов `GraphTopologyDelete` удаляет топологию.
    * Окончательный вызов `GraphTopologyList` показывает, что список теперь пуст.

