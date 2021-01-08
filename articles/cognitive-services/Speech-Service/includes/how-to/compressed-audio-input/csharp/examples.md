---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/09/2020
ms.author: trbye
ms.custom: devx-track-csharp
ms.openlocfilehash: 754b9c5215299a6f7488633c0871bceb61cbedba
ms.sourcegitcommit: 9514d24118135b6f753d8fc312f4b702a2957780
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/07/2021
ms.locfileid: "97978880"
---
Чтобы настроить речевой пакет SDK для приема сжатого звукового ввода, создайте `PullAudioInputStream` или `PushAudioInputStream` . Затем создайте объект `AudioConfig` из экземпляра класса Stream, указав формат сжатия потока.

Предположим, что у вас есть класс входного потока `pushStream` с именем и используется опус/OGG. Код может выглядеть следующим образом:

```csharp
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;

// ... omitted for brevity

var speechConfig =
    SpeechConfig.FromSubscription(
        "YourSubscriptionKey",
        "YourServiceRegion");

// Create an audio config specifying the compressed
// audio format and the instance of your input stream class.
var audioFormat =
    AudioStreamFormat.GetCompressedFormat(
        AudioStreamContainerFormat.OGG_OPUS);
var audioConfig =
    AudioConfig.FromStreamInput(
        pushStream,
        audioFormat);

using var recognizer = new SpeechRecognizer(speechConfig, audioConfig);
var result = await recognizer.RecognizeOnceAsync();

var text = result.Text;
```
