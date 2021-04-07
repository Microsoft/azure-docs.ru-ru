---
ms.openlocfilehash: 766dd13f58268c044435a22fb30c1de816d4d151
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97532131"
---
Чтобы запустить пример кода, выполните следующие действия:

1. В Visual Studio Code откройте вкладку **Расширения** (или нажмите клавиши CTRL+SHIFT+X) и найдите Центр Интернета вещей Azure.
1. Щелкните правой кнопкой мыши и выберите **Параметры расширения**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="../../../media/run-program/extensions-tab.png" alt-text="Параметры расширения":::
1. Найдите и включите параметр "Show Verbose Message" (Показывать подробное сообщение).

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="../../../media/run-program/show-verbose-message.png" alt-text="Show Verbose Message"::: (Показывать подробное сообщение)
1. В Visual Studio Code перейдите к файлу *src/cloud-to-device-console-app/operations.json*.
1. Проверьте, отображается ли в узле **GraphTopologySet** следующее значение:

    `"topologyUrl" : "https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/motion-detection/2.0/topology.json"`
1. Убедитесь, что в узлах **GraphInstanceSet** и **GraphTopologyDelete** значение `topologyName` соответствует значению свойства `name` в топологии графа:

    `"topologyName" : "MotionDetection"`
    
1. Чтобы запустить сеанс отладки, нажмите клавишу F5. В окне **ТЕРМИНАЛ** отобразятся некоторые сообщения.
1. Файл *operations.json* начнет выполнение операций с вызова `GraphTopologyList` и `GraphInstanceList`. Если вы очистили ресурсы после работы с предыдущими краткими руководствами, тогда этот процесс возвратит пустые списки, а затем приостановится. Чтобы продолжить, нажмите клавишу ВВОД.

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
    
    В окне **ТЕРМИНАЛ** показывается следующий набор вызовов прямых методов:
     * Вызов `GraphTopologySet`, который использует использованный ранее `topologyUrl`.
     * Вызов `GraphInstanceSet`, который использует такой код:
         
    ```
    {
      "@apiVersion": "2.0",
      "name": "Sample-Graph",
      "properties": {
        "topologyName": "MotionDetection",
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
1. Вывод данных в окне **ТЕРМИНАЛ** приостановится с появлением предложения `Press Enter to continue`. Не нажимайте клавишу ВВОД на этом этапе. Прокрутите экран вверх, чтобы увидеть полезные данные ответов JSON на вызванные нами прямые методы.
1. Перейдите в окно **ВЫХОДНЫЕ ДАННЫЕ** в Visual Studio Code. Вы увидите сообщения, которые модуль Аналитики видеотрансляций в IoT Edge передает в центр Интернета вещей. Эти сообщения рассматриваются в следующем разделе этого краткого руководства.
1. Граф мультимедиа продолжит работать и выводить результаты. Симулятор RTSP будет продолжать циклический перебор исходного видео. Чтобы остановить граф мультимедиа, вернитесь к окну **ТЕРМИНАЛ** и нажмите клавишу ВВОД. 

    Следующая серия вызовов очищает ресурсы:

    * Вызов `GraphInstanceDeactivate` деактивирует экземпляр графа.
    * Вызов `GraphInstanceDelete` удаляет этот экземпляр.
    * Вызов `GraphTopologyDelete` удаляет топологию.
    * Окончательный вызов `GraphTopologyList` показывает, что список теперь пуст.
