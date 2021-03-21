---
title: Запрос к конечной точке контейнера текста для речи
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 08/31/2020
ms.author: aahi
ms.openlocfilehash: 9a29244745b154aa81997813fcf4e1457f599270
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103621990"
---
Контейнер предоставляет [интерфейсы API конечной точки на основе интерфейса RESTful](../rest-text-to-speech.md). Существует множество [примеров проектов исходного кода](https://github.com/Azure-Samples/Cognitive-Speech-TTS) для платформ, платформ и языковых вариантов.

Используя стандартные или нейронные контейнеры преобразования текста в речь, вы должны полагаться на языковой стандарт и голоса скачанного тега образа. Например, если вы загрузили `latest` тег, язык по умолчанию — `en-US` и `AriaNeural` Voice. `{VOICE_NAME}`Затем аргумент будет иметь значение [`en-US-AriaNeural`](../language-support.md#neural-voices) . См. пример SSML ниже:

```xml
<speak version="1.0" xml:lang="en-US">
    <voice name="en-US-AriaNeural">
        This text will get converted into synthesized speech.
    </voice>
</speak>
```

Однако для *пользовательского преобразования текста в речь* необходимо получить **голос или модель** с [настраиваемого голосового портала](https://aka.ms/custom-voice-portal). Имя пользовательской модели является синонимом имени голоса. Перейдите на страницу **обучения** и скопируйте **голосовое или модель** для использования в качестве `{VOICE_NAME}` аргумента.
<br><br>
:::image type="content" source="../media/custom-voice/custom-voice-model-voice-name.png" alt-text="Настраиваемая Voice-модель — имя голоса":::

См. пример SSML ниже:

```xml
<speak version="1.0" xml:lang="en-US">
    <voice name="custom-voice-model">
        This text will get converted into synthesized speech.
    </voice>
</speak>
```

Давайте создадим запрос HTTP POST, предоставляя несколько заголовков и полезных данных. Замените `{VOICE_NAME}` заполнитель своим собственным значением.

```curl
curl -s -v -X POST http://localhost:5000/speech/synthesize/cognitiveservices/v1 \
 -H 'Accept: audio/*' \
 -H 'Content-Type: application/ssml+xml' \
 -H 'X-Microsoft-OutputFormat: riff-24khz-16bit-mono-pcm' \
 -d '<speak version="1.0" xml:lang="en-US"><voice name="{VOICE_NAME}">This is a test, only a test.</voice></speak>' > output.wav
```


Команда:

* Конструирует запрос HTTP POST для `speech/synthesize/cognitiveservices/v1` конечной точки.
* Задает `Accept` заголовок `audio/*`
* Задает `Content-Type` заголовок `application/ssml+xml` для более подробной информации см. в разделе [текст запроса](../rest-text-to-speech.md#request-body).
* Задает `X-Microsoft-OutputFormat` заголовок `riff-16khz-16bit-mono-pcm` для дополнительных параметров см. [звуковые выходные данные](../rest-text-to-speech.md#audio-outputs).
* Отправляет запрос на [языке разметки речи (SSML)](../speech-synthesis-markup.md) , заданный в `{VOICE_NAME}` конечной точке.
