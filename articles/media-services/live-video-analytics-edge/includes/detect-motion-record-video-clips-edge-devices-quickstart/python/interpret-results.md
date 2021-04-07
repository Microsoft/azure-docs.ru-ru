---
ms.openlocfilehash: 870b5b54c4a5afb39b43063bd00d3cb73e8ee0f9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "88682333"
---
При запуске графа мультимедиа результаты из узла обработчика детектора движения проходят через узел приемника Центра Интернета вещей в Центр Интернета вещей. Сообщения, отображаемые в окне **ВЫХОДНЫЕ ДАННЫЕ** в Visual Studio Code содержат раздел `body` и раздел `applicationProperties`. Дополнительные сведения см. в статье [Создание и чтение сообщений Центра Интернета вещей](../../../../../iot-hub/iot-hub-devguide-messages-construct.md).

В приведенных ниже сообщениях модуль Аналитики видеотрансляций определяет свойства приложения и содержимое раздела body.

### <a name="mediasessionestablished-event"></a>Событие MediaSessionEstablished

После создания экземпляра графа мультимедиа узел источника RTSP пытается подключиться к RTSP-серверу, работающему в контейнере rtspsim-live555. Если соединение установлено, будет выводиться следующее событие.

```
[IoTHubMonitor] [05:37:21 AM] Message received from [lva-sample-device/lvaEdge]:
{  
"body": {
"sdp": "SDP:\nv=0\r\no=- 1586450538111534 1 IN IP4 xxx.xxx.xxx.xxx\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/camera-300s.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.03.06\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-300.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/camera-300s.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=4D0029;sprop-parameter-sets={SPS}\r\na=control:track1\r\n"  
},  
"applicationProperties": {  
    "dataVersion": "1.0",  
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/mediaservices/hubname",  
    "subject": "/graphInstances/Sample-Graph-1/sources/rtspSource",  
    "eventType": "Microsoft.Media.MediaGraph.Diagnostics.MediaSessionEstablished",  
    "eventTime": "2020-05-21T05:37:21.398Z",
    }  
}
```

В приведенных выше выходных данных: 

* Сообщение является событием диагностики `MediaSessionEstablished`. Он указывает, что исходный узел RTSP (субъект) установил подключение к симулятору RTSP и начал получать (имитировать) канал трансляции.
* В `applicationProperties` `subject` — это узел в топологии графа, из которого было создано сообщение. В этом случае сообщение исходит от узла источника RTSP.
* В `applicationProperties` `eventType` указывает, что это событие является событием диагностики.
* Значение `eventTime` — это время возникновения события.
* В разделе `body` содержатся данные о событии диагностики. В этом случае данные содержат сведения [протокола SDP](https://en.wikipedia.org/wiki/Session_Description_Protocol).

### <a name="recordingstarted-event"></a>Событие RecordingStarted

При обнаружении движения активируется узел шлюза обработчика сигнала, а узел приемника файлов в графе мультимедиа начинает записывать MP4-файл. Узел приемника файлов отправляет операционное событие. Для `type` установлено значение `motion`, чтобы указать, что это результат, полученный от обработчика обнаружения движения. Значение `eventTime` — это время в формате UTC, когда произошло движение. Дополнительные сведения об этом процессе см. в разделе [Обзор](#overview) этого краткого руководства.

Ниже приведен пример этого сообщения:

```
[IoTHubMonitor] [05:37:27 AM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "outputType": "filePath",
    "outputLocation": "/var/media/sampleFilesFromEVR-filesinkOutput-20200521T053726Z.mp4"
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/mediaservices/hubname",  
    "subject": "/graphInstances/Sample-Graph-1/sinks/fileSink",
    "eventType": "Microsoft.Media.Graph.Operational.RecordingStarted",
    "eventTime": "2020-05-21T05:37:27.713Z",
    "dataVersion": "1.0"
  }
}
```

В предыдущем сообщении: 

* В `applicationProperties` `subject` — это узел в графе мультимедиа, из которого было создано сообщение. В этом случае сообщение исходит из узла приемника файлов.
* В `applicationProperties` `eventType` указывает, что это событие является операционным.
* Значение `eventTime` — это время возникновения события. Это время составляет от 5 до 6 секунд после `MediaSessionEstablished` и после начала видеотрансляции. Это время соответствует отметке 5–6 секунд, когда [автомобиль начал двигаться](#review-the-sample-video) по стоянке.
* В разделе `body` содержатся данные об операционном событии. В этом случае данные содержат `outputType` и `outputLocation`.
* Переменная `outputType` указывает, что эта информация относится к пути файла
* Значение `outputLocation` — это расположение MP4-файла в пограничном модуле.

### <a name="recordingstopped-and-recordingavailable-events"></a>События RecordingStopped и RecordingAvailable

Если вы посмотрите на свойства узла шлюза обработчика сигнала в [топологии графа](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/evr-motion-files/topology.json), вы увидите, что время активации составляет 5 секунд. Примерно через 5 секунд после получения события `RecordingStarted` вы получаете следующее:

* событие `RecordingStopped`, указывающее, что запись остановлена;
* событие `RecordingAvailable`, указывающее, что теперь MP4-файл можно использовать для просмотра.

Эти два события обычно происходят одно за другим с разницей в несколько секунд.
