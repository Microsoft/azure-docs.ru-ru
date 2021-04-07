---
ms.openlocfilehash: fdc7360911e37babdb830b7d67ad7224ea84c774
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92038476"
---
При запуске графа мультимедиа результаты из узла обработчика расширения HTTP отправляются через узел-преемник Центра Интернета вещей в центр Интернета вещей. Эти сообщения, отображаемые в окне **Вывод** содержат разделы body (текст) и applicationProperties (свойства приложения). Дополнительные сведения см. в статье [Создание и чтение сообщений Центра Интернета вещей](../../../../../iot-hub/iot-hub-devguide-messages-construct.md).

В приведенных ниже сообщениях модуль Аналитики видеотрансляций определяет свойства приложения и содержимое раздела body.

### <a name="mediasessionestablished-event"></a>Событие MediaSessionEstablished

После создания экземпляра графа мультимедиа узел источника RTSP пытается подключиться к RTSP-серверу, работающему в контейнере rtspsim-live555. Если соединение установлено, будет выводиться указанное ниже событие. Тип события — `Microsoft.Media.MediaGraph.Diagnostics.MediaSessionEstablished`.

```
[IoTHubMonitor] [9:42:18 AM] Message received from [lvaedgesample/lvaEdge]:
{  "body": {
    "sdp": "SDP:\nv=0\r\no=- 1586450538111534 1 IN IP4 nnn.nn.0.6\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/camera-300s.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.03.06\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-300.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/camera-300s.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=4D0029;sprop-parameter-sets=Z00AKeKQCgC3YC3AQEBpB4kRUA==,aO48gA==\r\na=control:track1\r\n"
  },
  "applicationProperties": {
    "dataVersion": "1.0",
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/mediaservices/hubname",
    "subject": "/graphInstances/GRAPHINSTANCENAMEHERE/sources/rtspSource",
    "eventType": "Microsoft.Media.MediaGraph.Diagnostics.MediaSessionEstablished",
    "eventTime": "2020-04-09T16:42:18.1280000Z"
  }
}
```

В этом сообщении обратите внимание на следующие сведения:
* Сообщение является событием диагностики. `MediaSessionEstablished` указывает, что исходный узел RTSP (параметр subject) установил подключение к симулятору RTSP и начал прием потока данных (имитируемого).
* В `applicationProperties` параметр subject указывает, что сообщение было создано из узла RTSP-источника в графе мультимедиа.
* В `applicationProperties` `eventType` указывает, что это событие является событием диагностики.
* `eventTime` указывает время события.
* Текст содержит данные о событии диагностики. В этом случае данные содержат сведения [протокола SDP](https://en.wikipedia.org/wiki/Session_Description_Protocol).

### <a name="inference-event"></a>Событие вывода

Узел обработчика расширения gRPC получает результаты вывода из модуля lvaExtension. Затем он выдает результаты через узел приемника Центра Интернета вещей в качестве событий вывода.
В этих событиях тип имеет значение сущности, чтобы указать, что это сущность, например легковой автомобиль или грузовик. Значение `eventTime` — время в формате UTC, когда был обнаружен объект.
В следующем примере три автомобиля были обнаружены в одном видеокадре с различными уровнями достоверности.

```
[IoTHubMonitor] [7:52:57 PM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "timestamp": 143802500954716,
    "inferences": [
      {
        "type": "entity",
        "subtype": "",
        "inferenceId": "",
        "relatedInferences": [],
        "entity": {
          "tag": {
            "value": "car",
            "confidence": 0.86707145
          },
          "attributes": [],
          "box": {
            "l": 0.7294476,
            "t": 0.567829,
            "w": 0.031738576,
            "h": 0.027762715
          }
        },
        "extensions": {},
        "valueCase": "entity"
      },
      {
        "type": "entity",
        "subtype": "",
        "inferenceId": "",
        "relatedInferences": [],
        "entity": {
          "tag": {
            "value": "car",
            "confidence": 0.8634398
          },
          "attributes": [],
          "box": {
            "l": 0.4765757,
            "t": 0.59559196,
            "w": 0.05597749,
            "h": 0.048316106
          }
        },
        "extensions": {},
        "valueCase": "entity"
      },
      {
        "type": "entity",
        "subtype": "",
        "inferenceId": "",
        "relatedInferences": [],
        "entity": {
          "tag": {
            "value": "car",
            "confidence": 0.67120105
          },
          "attributes": [],
          "box": {
            "l": 0.7247387,
            "t": 0.49816906,
            "w": 0.032748587,
            "h": 0.030686663
          }
        },
        "extensions": {},
        "valueCase": "entity"
      }
    ]
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/mediaservices/hubname",
    "subject": "/graphInstances/GRAPHINSTANCENAMEHERE/processors/grpcExtension",
    "eventType": "Microsoft.Media.Graph.Analytics.Inference",
    "eventTime": "2020-08-19T02:52:57.174Z",
    "dataVersion": "1.0"
  }
}
```

В сообщениях обратите внимание на следующие сведения:

* В `applicationProperties` параметр subject указывает на узел в топологии графа, из которого было создано сообщение.
* В `applicationProperties` параметр eventType указывает, что это событие является событием аналитики.
* Значение `eventTime` — это время возникновения события.
* Раздел `body` содержит информацию об аналитическом событии. В данном случае событие является событием вывода, поэтому раздел body содержит данные параметра inferences.
* Раздел `inferences` указывает, что этот тип является сущностью. Этот раздел содержит дополнительные данные о сущности.