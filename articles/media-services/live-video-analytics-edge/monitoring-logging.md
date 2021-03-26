---
title: Мониторинг и ведение журналов — Azure
description: В этой статье приводятся общие сведения о мониторинге и ведении журналов в службе Live Video Analytics на IoT Edge.
ms.topic: reference
ms.date: 04/27/2020
ms.openlocfilehash: 08b2f5cce80581d71ce73e97ab30900aa8957c77
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105564489"
---
# <a name="monitoring-and-logging"></a>Мониторинг и ведение журнала

В этой статье вы узнаете, как получать события для удаленного мониторинга из интерактивной аналитики видео в модуле IoT Edge. 

Вы также узнаете, как управлять журналами, создаваемыми модулем.

## <a name="taxonomy-of-events"></a>Классификация событий

Интерактивная аналитика видео на IoT Edge выдает события или данные телеметрии в соответствии со следующей классификацией:

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/telemetry-schema/taxonomy.png" alt-text="Схема, показывающая таксономию событий.":::

* Операционные: события, создаваемые действиями пользователя или во время выполнения [графа мультимедиа](media-graph-concept.md) .
   
   * Том: ожидается низкий (несколько раз в минуту или даже меньше)
   * Примеры:

      - Запись начата (показано в следующем примере)
      - Запись остановлена
      
      ```
      {
        "body": {
          "outputType": "assetName",
          "outputLocation": "sampleAssetFromEVR-LVAEdge-20200512T233309Z"
        },
        "applicationProperties": {
          "topic": "/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/<my-resource-group>/providers/microsoft.media/mediaservices/<ams-account-name>",
          "subject": "/graphInstances/Sample-Graph-2/sinks/assetSink",
          "eventType": "Microsoft.Media.Graph.Operational.RecordingStarted",
          "eventTime": "2020-05-12T23:33:10.392Z",
          "dataVersion": "1.0"
        }
      }
      ```
* Диагностика: события, помогающие диагностировать проблемы с производительностью

   * Том: может быть высокой (несколько раз в минуту)
   * Примеры:
   
      - Сведения о протоколе RTSP для [распространения](https://en.wikipedia.org/wiki/Session_Description_Protocol) данных (показаны в следующем примере) 
      - Пропуски во входящем веб-канале видео

      ```
      {
        "body": {
          "sdp": "SDP:\nv=0\r\no=- 1589326384077235 1 IN IP4 XXX.XX.XX.XXX\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/lots_015.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.04.12\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-73.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/lots_015.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=640028;sprop-parameter-sets=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\r\na=control:track1\r\n"
        },
        "applicationProperties": {
          "topic": "/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/<my-resource-group>/providers/microsoft.media/mediaservices/<ams-account-name>",
          "subject": "/graphInstances/Sample-Graph-2/sources/rtspSource",
          "eventType": "Microsoft.Media.Graph.Diagnostics.MediaSessionEstablished",
          "eventTime": "2020-05-12T23:33:04.077Z",
          "dataVersion": "1.0"
        }
      }
      ```
* Аналитика: события, созданные как часть анализа видео

   * Том: может быть высокой (несколько раз в минуту или более)
   * Примеры:
      
      - Обнаружено перемещение (показано в следующем примере) 
      - Результат вывода

   ```      
   {
     "body": {
       "timestamp": 143039375044290,
       "inferences": [
         {
           "type": "motion",
           "motion": {
             "box": {
               "l": 0.48954,
               "t": 0.140741,
               "w": 0.075,
               "h": 0.058824
             }
           }
         }
       ]
     },
     "applicationProperties": {
       "topic": "/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/<my-resource-group>/providers/microsoft.media/mediaservices/<ams-account-name>",
       "subject": "/graphInstances/Sample-Graph-2/processors/md",
       "eventType": "Microsoft.Media.Graph.Analytics.Inference",
       "eventTime": "2020-05-12T23:33:09.381Z",
       "dataVersion": "1.0"
     }
   }
   ```

События, порожденные модулем, отправляются в [центр IOT Edge](../../iot-edge/iot-edge-runtime.md#iot-edge-hub). Их можно направлять в другие места назначения. 

### <a name="timestamps-in-analytic-events"></a>Метки времени в событиях аналитики

Как было сказано ранее, события, созданные как часть анализа видео, имеют связанные с ними метки времени. Если вы [записали в реальном времени видео](video-recording-concept.md) в качестве части топологии графа, они помогут определить, где в записанном видео произошло определенное событие. Ниже приведены рекомендации по сопоставлению метки времени в событии аналитики с временной шкалой видео, записанном в [ресурс служб мультимедиа Azure](terminology.md#asset).

Сначала извлеките `eventTime` значение. Используйте это значение в [фильтре диапазона времени](playback-recordings-how-to.md#time-range-filters) , чтобы получить подходящую часть записи. Например, может потребоваться получить видео, которое начинается через 30 секунд до `eventTime` и заканчивается 30 секунд после него. Для предыдущего примера, где `eventTime` — 2020-05-12T23:33:09.381 z, запрос манифеста HLS в течение 30 секунд до и после `eventTime` будет выглядеть следующим образом:

```
https://{hostname-here}/{locatorGUID}/content.ism/manifest(format=m3u8-aapl,startTime=2020-05-12T23:32:39Z,endTime=2020-05-12T23:33:39Z).m3u8
```

Предыдущий URL-адрес вернет [главный список воспроизведения](https://developer.apple.com/documentation/http_live_streaming/example_playlists_for_http_live_streaming) , содержащий URL-адреса для списков воспроизведения мультимедиа. Список воспроизведения мультимедиа будет содержать такие записи:

```
...
#EXTINF:3.103011,no-desc
Fragments(video=143039375031270,format=m3u8-aapl)
...
```
Предыдущая запись сообщает о наличии фрагмента видео, который начинается со `timestamp` значения `143039375031270` . `timestamp`Значение в событии аналитики использует ту же шкалу времени, что и список воспроизведения мультимедиа. Его можно использовать для обнаружения соответствующего фрагмента видео и поиска в нужном кадре.

Дополнительные сведения см. в этих [статьях по поиску с точностью](https://www.bing.com/search?q=frame+accurate+seeking+in+HLS) в HLS.

## <a name="controlling-events"></a>Управление событиями

Вы можете использовать следующие свойства модуля двойника, чтобы управлять операционными и диагностическими событиями, опубликованными в реальном видео Analytics, в модуле IoT Edge. Эти свойства описаны в [схеме JSON модуля двойника](module-twin-configuration-schema.md).

- `diagnosticsEventsOutputName`: Для получения диагностических событий из модуля включите это свойство и укажите для него любое значение. Удалите его или оставьте пустым, чтобы модуль не подводил публикацию событий диагностики.
   
- `operationalEventsOutputName`: Чтобы получить операционные события из модуля, включите это свойство и укажите для него любое значение. Удалите его или оставьте пустым, чтобы модуль не подводил публикацию операционных событий.
   
События аналитики создаются такими узлами, как процессор обнаружения движения или обработчик расширения HTTP. Приемник центра Интернета вещей используется для отправки их в центр IoT Edge. 

Можно управлять [маршрутизацией всех предшествующих событий](../../iot-edge/module-composition.md#declare-routes) , используя `desired` свойство `$edgeHub` модуля двойника в манифесте развертывания:

```
 "$edgeHub": {
   "properties.desired": {
     "schemaVersion": "1.0",
     "routes": {
       "moduleToHub": "FROM /messages/modules/lvaEdge/outputs/* INTO $upstream"
     },
     "storeAndForwardConfiguration": {
       "timeToLiveSecs": 7200
     }
   }
 }
```

В предыдущем примере JSON `lvaEdge` — это имя службы анализа видео в реальном времени в модуле IOT Edge. Правило маршрутизации соответствует схеме, определенной в [Declare Routes](../../iot-edge/module-composition.md#declare-routes).

> [!NOTE]
> Чтобы обеспечить доступность событий аналитики в центре IoT Edge, необходимо, чтобы узел приемника центра Интернета вещей был расположен на любом узле процессора для обнаружения движения и (или) любом узле обработчика HTTP-расширений.

## <a name="event-schema"></a>Схема событий

События происходят на пограничном устройстве, и их можно использовать на границе или в облаке. События, создаваемые функцией Live Video Analytics на IoT Edge соответствуют [шаблону обмена сообщениями потоковой передачи](../../iot-hub/iot-hub-devguide-messages-construct.md) , установленному центром Интернета вещей Azure. Шаблон состоит из системных свойств, свойств приложений и текста.

### <a name="summary"></a>Итоги

Каждое событие, наблюдаемое через центр Интернета вещей, имеет набор общих свойств:

|Свойство   |Тип свойства| Тип данных   |Описание|
|---|---|---|---|
|`message-id`   |система |guid|  Уникальный идентификатор события.|
|`topic`|   аппликатионпроперти |строка|    Azure Resource Manager путь к учетной записи служб мультимедиа Azure.|
|`subject`| аппликатионпроперти |строка|    Вложенный путь сущности, порожденной событием.|
|`eventTime`|   аппликатионпроперти|    строка| Время создания события.|
|`eventType`|   аппликатионпроперти |строка|    Идентификатор типа события. (См. следующий раздел.)|
|`body`|текст    |объект|    Данные определенного события.|
|`dataVersion`  |аппликатионпроперти|   строка  |{Major}. Дополнительный|

### <a name="properties"></a>Свойства

#### <a name="message-id"></a>message-id

Глобальный уникальный идентификатор (GUID) для события.

#### <a name="topic"></a>Раздел

Представляет учетную запись служб мультимедиа Azure, связанную с графом.

`/subscriptions/{subId}/resourceGroups/{rgName}/providers/Microsoft.Media/mediaServices/{accountName}`

#### <a name="subject"></a>subject

Сущность, которая порождает событие:

`/graphInstances/{graphInstanceName}`<br/>
`/graphInstances/{graphInstanceName}/sources/{sourceName}`<br/>
`/graphInstances/{graphInstanceName}/processors/{processorName}`<br/>
`/graphInstances/{graphInstanceName}/sinks/{sinkName}`

`subject`Свойство позволяет сопоставлять универсальные события с создаваемым модулем. Например, для недопустимого имени пользователя или пароля RTSP созданное событие будет находиться `Microsoft.Media.Graph.Diagnostics.ProtocolError` на `/graphInstances/myGraph/sources/myRtspSource` узле.

#### <a name="event-types"></a>Типы событий

Типы событий назначаются пространству имен в соответствии со схемой:

`Microsoft.Media.Graph.{EventClass}.{EventType}`

#### <a name="event-classes"></a>Классы событий

|Имя класса|Описание|
|---|---|
|Аналитика  |События, создаваемые как часть анализа содержимого.|
|Диагностика    |События, помогающие в диагностике проблем и производительности.|
|Операционный    |События, создаваемые как часть операции с ресурсами.|

Типы событий относятся к каждому классу событий.

Примеры:

* `Microsoft.Media.Graph.Analytics.Inference`
* `Microsoft.Media.Graph.Diagnostics.AuthorizationError`
* `Microsoft.Media.Graph.Operational.GraphInstanceStarted`

### <a name="event-time"></a>Время события

Время события отформатировано в строке ISO 8601. Он представляет время, когда произошло событие.

### <a name="azure-monitor-collection-via-telegraf"></a>Сбор Azure Monitor с помощью Telegraf

Эти метрики будут переданы из службы "интерактивная аналитика видео" в модуле IoT Edge:  

|Имя метрики|Тип|Метка|Описание|
|-----------|----|-----|-----------|
|lva_active_graph_instances|Датчик|iothub, edge_device, module_name, graph_topology|Общее число активных графиков на топологию.|
|lva_received_bytes_total|Счетчик|iothub, edge_device, module_name, graph_topology, graph_instance, graph_node|Общее число байтов, полученных узлом. Поддерживается только для источников RTSP.|
|lva_data_dropped_total|Счетчик|iothub, edge_device, module_name, graph_topology, graph_instance, graph_node, data_kind|Счетчик любых удаленных данных (событий, носителей и т. д.).|

> [!NOTE]
> [Конечная точка Prometheus](https://prometheus.io/docs/practices/naming/) предоставляется по порту 9600 контейнера. Если вы направите свое имя в Live Video Analytics на IoT Edge модуле "Лваедже", они смогут получить доступ к метрикам, отправив запрос GET в http://lvaEdge:9600/metrics .   

Выполните следующие действия, чтобы включить сбор метрик из интерактивной аналитики видео в модуле IoT Edge:

1. Создайте папку на компьютере разработчика и перейдите в эту папку.

1. В папке создайте `telegraf.toml` файл, содержащий следующие конфигурации.
    ```
    [agent]
        interval = "30s"
        omit_hostname = true

    [[inputs.prometheus]]
      metric_version = 2
      urls = ["http://edgeHub:9600/metrics", "http://edgeAgent:9600/metrics", "http://{LVA_EDGE_MODULE_NAME}:9600/metrics"]

    [[outputs.azure_monitor]]
      namespace_prefix = "lvaEdge"
      region = "westus"
      resource_id = "/subscriptions/{SUBSCRIPTON_ID}/resourceGroups/{RESOURCE_GROUP}/providers/Microsoft.Devices/IotHubs/{IOT_HUB_NAME}"
    ```
    > [!IMPORTANT]
    > Не забудьте заменить переменные в файле. томл. Переменные обозначаются фигурными скобками ( `{}` ).

1. В той же папке создайте Dockerfile, содержащую следующие команды:
    ```
        FROM telegraf:1.15.3-alpine
        COPY telegraf.toml /etc/telegraf/telegraf.conf
    ```

1. Используйте команды DOCKER CLI, чтобы создать файл DOCKER и опубликовать образ в реестре контейнеров Azure.
    
   Дополнительные сведения об использовании DOCKER CLI для отправки в реестр контейнеров см. в разделе [Отправка и извлечение образов DOCKER](../../container-registry/container-registry-get-started-docker-cli.md). Дополнительные сведения о реестре контейнеров Azure см. в [документации](../../container-registry/index.yml).


1. После завершения отправки в реестр контейнеров Azure добавьте следующий узел в файл манифеста развертывания:
    ```
    "telegraf": 
    {
      "settings": 
        {
            "image": "{AZURE_CONTAINER_REGISTRY_LINK_TO_YOUR_TELEGRAF_IMAGE}"
        },
      "type": "docker",
      "version": "1.0",
      "status": "running",
      "restartPolicy": "always",
      "env": 
        {
            "AZURE_TENANT_ID": { "value": "{YOUR_TENANT_ID}" },
            "AZURE_CLIENT_ID": { "value": "{YOUR CLIENT_ID}" },
            "AZURE_CLIENT_SECRET": { "value": "{YOUR_CLIENT_SECRET}" }
        }
    ``` 
    > [!IMPORTANT]
    > Не забудьте заменить переменные в файле манифеста. Переменные обозначаются фигурными скобками ( `{}` ).


   [Проверку подлинности](https://github.com/influxdata/telegraf/blob/master/plugins/outputs/azure_monitor/README.md#azure-authentication)Azure Monitor можно выполнить с помощью субъекта-службы.
        
   Подключаемый модуль Azure Monitor Telegraf предоставляет [несколько методов проверки подлинности](https://github.com/influxdata/telegraf/blob/master/plugins/outputs/azure_monitor/README.md#azure-authentication). 

  1. Чтобы использовать проверку подлинности субъекта-службы, задайте следующие переменные среды:  
     `AZURE_TENANT_ID`: Указывает клиента для проверки подлинности.  
     `AZURE_CLIENT_ID`: Указывает используемый идентификатор клиента приложения.  
     `AZURE_CLIENT_SECRET`: Указывает секрет приложения для использования.  
     
     >[!TIP]
     > Субъекту-службе можно назначить роль **издателя метрики мониторинга** . Выполните действия, описанные в разделе **[Создание субъекта-службы](../../azure-arc/data/upload-metrics-and-logs-to-azure-monitor.md?pivots=client-operating-system-macos-and-linux#create-service-principal)** , чтобы создать субъект-службу и назначить роль.

1. После развертывания модулей в Azure Monitor в одном пространстве имен будут отображаться метрики. Имена метрик будут совпадать с именами, порожденными Prometheus. 

   В этом случае в портал Azure перейдите в центр Интернета вещей и выберите **метрики** в левой области. Вы должны увидеть метрики там.

### <a name="log-analytics-metrics-collection"></a>Коллекция метрик Log Analytics
Используя [конечную точку Prometheus](https://prometheus.io/docs/practices/naming/) вместе с [log Analytics](../../azure-monitor/logs/log-analytics-tutorial.md), можно создавать и [отслеживать такие метрики](../../azure-monitor/essentials/metrics-supported.md) , как используемые cpupercent присутствуют, меморюседперцент и т. д.   

> [!NOTE]
> Приведенная ниже конфигурация не выполняет сбора журналов, а **только метрик**. Можно расширить модуль сборщика для сбора и отправки журналов.

[![Схема, на которой показана коллекция метрик с использованием log Analytics.](./media/telemetry-schema/log-analytics.png)](./media/telemetry-schema/log-analytics.png#lightbox)

1. Узнайте, как выполнять [Получение метрик](https://github.com/Azure/iotedge/tree/master/edge-modules/MetricsCollector)
1. Используйте команды DOCKER CLI, чтобы создать [файл DOCKER](https://github.com/Azure/iotedge/tree/master/edge-modules/MetricsCollector/docker/linux) и опубликовать образ в реестре контейнеров Azure.
    
   Дополнительные сведения об использовании DOCKER CLI для отправки в реестр контейнеров см. в разделе [Отправка и извлечение образов DOCKER](../../container-registry/container-registry-get-started-docker-cli.md). Дополнительные сведения о реестре контейнеров Azure см. в [документации](../../container-registry/index.yml).

1. После завершения отправки в реестр контейнеров Azure в манифест развертывания вставляется следующее:
    ```json
    "azmAgent": {
      "settings": {
        "image": "{AZURE_CONTAINER_REGISTRY_LINK_TO_YOUR_METRICS_COLLECTOR}"
      },
      "type": "docker",
      "version": "1.0",
      "status": "running",
      "restartPolicy": "always",
      "env": {
        "LogAnalyticsWorkspaceId": { "value": "{YOUR_LOG_ANALYTICS_WORKSPACE_ID}" },
        "LogAnalyticsSharedKey": { "value": "{YOUR_LOG_ANALYTICS_WORKSPACE_SECRET}" },
        "LogAnalyticsLogType": { "value": "IoTEdgeMetrics" },
        "MetricsEndpointsCSV": { "value": "http://edgeHub:9600/metrics,http://edgeAgent:9600/metrics,http://lvaEdge:9600/metrics" },
        "ScrapeFrequencyInSecs": { "value": "30 " },
        "UploadTarget": { "value": "AzureLogAnalytics" }
      }
    }
    ```
    > [!NOTE]
    > Модули `edgeHub` `edgeAgent` и — это `lvaEdge` имена модулей, определенных в файле манифеста развертывания. Убедитесь, что имена модулей совпадают.   

    Чтобы получить `LogAnalyticsWorkspaceId` `LogAnalyticsSharedKey` значения и, выполните следующие действия.
    1. Перейдите на портал Azure.
    1. Поиск рабочих областей Log Analytics
    1. Найдя рабочую область Log Analytics, перейдите к `Agents management` параметру в левой области навигации.
    1. Вы увидите идентификатор рабочей области и секретные ключи, которые можно использовать.

1. Затем создайте книгу, щелкнув `Workbooks` вкладку в левой области навигации.
1. С помощью языка запросов Kusto можно создавать запросы, как показано ниже, и получать процент загрузки ЦП, используемый модулями IoT Edge.
    ```kusto
    let cpu_metrics = IoTEdgeMetrics_CL
    | where Name_s == "edgeAgent_used_cpu_percent"
    | extend dimensions = parse_json(Tags_s)
    | extend module_name = tostring(dimensions.module_name)
    | where module_name in ("lvaEdge","yolov3","tinyyolov3")
    | summarize cpu_percent = avg(Value_d) by bin(TimeGenerated, 5s), module_name;
    cpu_metrics
    | summarize cpu_percent = sum(cpu_percent) by TimeGenerated
    | extend module_name = "Total"
    | union cpu_metrics
    ```

    [![Схема, на которой показаны метрики с помощью запроса Kusto.](./media/telemetry-schema/metrics.png)](./media/telemetry-schema/metrics.png#lightbox)
## <a name="logging"></a>Ведение журнала

Как и в случае с другими IoT Edgeными модулями, вы также можете [изучить журналы контейнеров](../../iot-edge/troubleshoot.md#check-container-logs-for-issues) на пограничном устройстве. Сведения, записываемые в журналы, можно настроить с помощью [следующих свойств модуля двойника](module-twin-configuration-schema.md) :

* `logLevel`

   * Допустимые значения: `Verbose`, `Information`, `Warning`, `Error` и `None`.
   * Значение по умолчанию — `Information`. Журналы будут содержать ошибки, предупреждения и информационные сообщения.
   * Если задать значение `Warning` , журналы будут содержать сообщения об ошибках и предупреждения.
   * Если задать значение `Error` , журналы будут содержать только сообщения об ошибках.
   * Если задано значение `None` , журналы не создаются. (Мы не рекомендуем использовать эту конфигурацию.)
   * Используйте, `Verbose` только если вам необходимо предоставить общий доступ к журналам с помощью службы поддержки Azure для диагностики проблемы.

* `logCategories`

   * Список с разделителями-запятыми одного или нескольких следующих значений: `Application` , `Events` , `MediaPipeline` .
   * Значение по умолчанию — `Application, Events`.
   * `Application`: Сведения высокого уровня из модуля, например сообщения о запуске модуля, ошибки среды и вызовы прямых методов.
   * `Events`: Все события, описанные ранее в этой статье.
   * `MediaPipeline`: Журналы низкого уровня, которые могут предоставлять аналитические сведения при устранении неполадок, например проблемы с подключением к камере с поддержкой RTSP.
   
### <a name="generating-debug-logs"></a>Создание журналов отладки

В некоторых случаях, чтобы помочь службе поддержки Azure разрешить проблему, может потребоваться создать более подробные журналы, чем описано выше. Чтобы создать эти журналы, выполните следующие действия.

1. [Свяжите хранилище модулей с хранилищем устройств с](../../iot-edge/how-to-access-host-storage-from-module.md#link-module-storage-to-device-storage) помощью `createOptions` . Если взглянуть на [шаблон манифеста развертывания](https://github.com/Azure-Samples/live-video-analytics-iot-edge-csharp/blob/master/src/edge/deployment.template.json) из кратких руководств, вы увидите следующий код:

   ```
   "createOptions": {
     …
     "Binds": [
       "/var/local/mediaservices/:/var/lib/azuremediaservices/"
     ]
    }
   ```

   Этот код позволяет модулю пограничных записей записывать журналы в путь к хранилищу устройства `/var/local/mediaservices/` . 

 1. Добавьте в `desired` модуль следующее свойство:

    `"debugLogsDirectory": "/var/lib/azuremediaservices/debuglogs/",`

Теперь модуль будет записывать журналы отладки в двоичном формате в путь к хранилищу устройства `/var/local/mediaservices/debuglogs/` . Вы можете поделиться этими журналами с помощью службы поддержки Azure.

## <a name="faq"></a>ВОПРОСЫ И ОТВЕТЫ

Если у вас есть вопросы, см. [вопросы и ответы по мониторингу и метрикам](faq.md#monitoring-and-metrics).

## <a name="next-steps"></a>Дальнейшие действия

[Непрерывная запись видео](continuous-video-recording-tutorial.md)