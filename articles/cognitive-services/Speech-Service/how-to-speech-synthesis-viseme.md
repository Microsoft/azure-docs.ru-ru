---
title: Как получить события лица лиц для пакета LIP-Sync
titleSuffix: Azure Cognitive Services
description: Пакет SDK для распознавания речи поддерживает событие висеме в синтезе речи, которое используется для представления ключей в наблюдаемых речевых данных, таких как расположение пакетов LIP, жав и дразнящаяся рожица при создании определенного phoneme.
services: cognitive-services
author: yulin-li
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/03/2021
ms.author: yulili
ms.custom: references_regions
zone_pivot_groups: programming-languages-speech-services-nomore-variant
ms.openlocfilehash: f74a242db2686eb4571ebbea80b88a75dda205d4
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105044073"
---
# <a name="get-facial-pose-events"></a>Получение событий лица, налагаемых лиц

> [!NOTE]
> Сейчас висеме работает только для `en-US-AriaNeural` голоса в регионе "Западная часть США 2 ( `westus2` )".

Висеме — это визуальное описание phoneme в речевом языке.
Он определяет расположение лица и рот при произнесении слова.
В каждом висемее представлено ключевое лиц, относящееся к определенному набору фонемы.
Между висемес и фонемы нет соответствия "один к одному".
Часто несколько фонемы соответствуют одному висеме, так как несколько фонемы выглядят одинаково на лицевой стороне при создании, например `s` и `z` .
См. [таблицу сопоставления между висемес и фонемы](#map-phonemes-to-visemes).

С помощью висемес можно создать более естественный и интеллектуальный помощник по рассылке новостей, более интерактивные игры и мультипликационные символы, а также более интуитивно понятные видеоматериалы по языку. Люди с нарушениями слуха также могут получать звуки визуально и в виде речевого содержимого "LIP-Read", которое показывает, висемес на анимированной грани.

## <a name="get-viseme-events-with-the-speech-sdk"></a>Получение событий висеме с помощью пакета SDK для распознавания речи

Чтобы сделать события висеме, мы преобразуем входной текст в набор phoneme последовательностей и соответствующие последовательности висеме. Мы оцениваем время начала каждого висеме в речевом аудио. События висеме содержат последовательность идентификаторов висеме, каждый из которых имеет смещение в аудио, где отображается висеме. Эти события могут наносить анимации, имитирующие личность пользователя.

| Параметр | Описание |
|-----------|-------------|
| Идентификатор висеме | Целое число, указывающее висеме. На английском языке (США) мы предлагаем 22 разных висемесов, которые описывают фигуры рот для определенного набора фонемы. См. [таблицу сопоставления между ВИСЕМЕ ID и фонемы](#map-phonemes-to-visemes).  |
| Смещение звука | Время начала каждого висеме в тактах (100 наносекунд). |

Чтобы получить события висеме, подпишитесь на `VisemeReceived` событие в пакете SDK для распознавания речи.
В следующих фрагментах кода показано, как подписываться на событие висеме.

::: zone pivot="programming-language-csharp"

```csharp
using (var synthesizer = new SpeechSynthesizer(speechConfig, audioConfig))
{
    // Subscribes to viseme received event
    synthesizer.VisemeReceived += (s, e) =>
    {
        Console.WriteLine($"Viseme event received. Audio offset: " +
            $"{e.AudioOffset / 10000}ms, viseme id: {e.VisemeId}.");
    };

    var result = await synthesizer.SpeakSsmlAsync(ssml));
}

```

::: zone-end

::: zone pivot="programming-language-cpp"

```cpp
auto synthesizer = SpeechSynthesizer::FromConfig(speechConfig, audioConfig);

// Subscribes to viseme received event
synthesizer->VisemeReceived += [](const SpeechSynthesisVisemeEventArgs& e)
{
    cout << "viseme event received. "
        // The unit of e.AudioOffset is tick (1 tick = 100 nanoseconds), divide by 10,000 to convert to milliseconds.
        << "Audio offset: " << e.AudioOffset / 10000 << "ms, "
        << "viseme id: " << e.VisemeId << "." << endl;
};

auto result = synthesizer->SpeakSsmlAsync(ssml).get();
```

::: zone-end

::: zone pivot="programming-language-java"

```java
SpeechSynthesizer synthesizer = new SpeechSynthesizer(speechConfig, audioConfig);

// Subscribes to viseme received event
synthesizer.VisemeReceived.addEventListener((o, e) -> {
    // The unit of e.AudioOffset is tick (1 tick = 100 nanoseconds), divide by 10,000 to convert to milliseconds.
    System.out.print("Viseme event received. Audio offset: " + e.getAudioOffset() / 10000 + "ms, ");
    System.out.println("viseme id: " + e.getVisemeId() + ".");
});

SpeechSynthesisResult result = synthesizer.SpeakSsmlAsync(ssml).get();
```

::: zone-end

::: zone pivot="programming-language-python"

```Python
speech_synthesizer = speechsdk.SpeechSynthesizer(speech_config=speech_config, audio_config=audio_config)

# Subscribes to viseme received event
speech_synthesizer.viseme_received.connect(lambda evt: print(
    "Viseme event received: audio offset: {}ms, viseme id: {}.".format(evt.audio_offset / 10000, evt.viseme_id)))

result = speech_synthesizer.speak_ssml_async(ssml).get()
```

::: zone-end

::: zone pivot="programming-language-javascript"

```Javascript
var synthesizer = new SpeechSDK.SpeechSynthesizer(speechConfig, audioConfig);

// Subscribes to viseme received event
synthesizer.visemeReceived = function (s, e) {
    window.console.log("(Viseme), Audio offset: " + e.audioOffset / 10000 + "ms. Viseme ID: " + e.visemeId);
}

synthesizer.speakSsmlAsync(ssml);
```

::: zone-end

::: zone pivot="programming-language-objectivec"

```Objective-C
SPXSpeechSynthesizer *synthesizer =
    [[SPXSpeechSynthesizer alloc] initWithSpeechConfiguration:speechConfig
                                           audioConfiguration:audioConfig];

// Subscribes to viseme received event
[synthesizer addVisemeReceivedEventHandler: ^ (SPXSpeechSynthesizer *synthesizer, SPXSpeechSynthesisVisemeEventArgs *eventArgs) {
    NSLog(@"Viseme event received. Audio offset: %fms, viseme id: %lu.", eventArgs.audioOffset/10000., eventArgs.visemeId);
}];

[synthesizer speakSsml:ssml];
```

::: zone-end

## <a name="map-phonemes-to-visemes"></a>Map фонемы to висемес

Висемес зависит от языка. Каждый язык имеет набор висемес, соответствующих определенному фонемы. В следующей таблице показано соответствие между международным фонетическим алфавитом (IPA) фонемы и идентификаторами висеме для английского языка (США).

| IPA | Пример | Идентификатор висеме |
|-----|---------|-----------|
| i   | **EA** t   | 6 |
| ɪ   | **i** е | 6 |
| еɪ  | **TE** | 4 |
| ɛ | **e** очень | 4 |
|æ  |   **ктивное**        |1|
|ɑ  |   **o** бстинате     |2|
|ɔ  |   c **Au** SE         |3|
|ʊ  |   б **ОО** k          |4|
|оʊ |   **o** LD           |8|
|u  |   **Личество**          |7|
|ʌ  |   **нкле**         |1|
|аɪ |   **i** CE           |11|
|аʊ |   **подразделение** t           |9|
|ɔɪ |   **Oi** l           |10|
|жу |   **Yu** MA          |[6, 7]|
|ə  |   **Go**           |1|
|ɪɹ |   **год**:          |[6, 13]|
|ɛɹ |   плоскость **воздуха**      |[4, 13]|
|ʊɹ |   c **Мой** электронный          |[4, 13]|
|аɪ (ə) ɹ |   **Немедленно** Land   |[11, 13]|
|аʊ (ə) ɹ |   **час** s     |[9, 13]|
|ɔɹ |   **или** менить        |[3, 13]|
|ɑɹ |   **AR** ТИСТ        |[2, 13]|
|ɝ  |   самое **поое**         |[5, 13]|
|ɚ  |   все GY **ER**       |[1, 13]|
|w  |   **w** i i, s **UE** de   |7|
|j  |   **y** АРД, f **e** w     |6|
|p  |   **p** UT           |21|
|b  |   **б** и           |21|
|t  |   **t** АЛК          |19|
|d  |   и **г**           |19|
|k  |   **c** UT           |20|
|н  |   **g** o            |20|
|m  |   **m****, с, в Ash**    |21|
|n  |   **n** o, s **n**      |19|
|ŋ  |   Li **n** k          |20|
|f  |   **язык для языка f** ORK          |18|
|v  |   **v** ЗНАЧ         |18|
|θ  |   **th** в          |17|
|«  |   **th** EN          |17|
|s  |   **s**           |15|
|z  |   **z** AP           |15|
|ʃ  |   **sh** e           |16|
|ʒ  |   **J** аккуес       |16|
|h  |   **h** ELP          |12|
|тʃ |   **CH** в          |16|
|дʒ |   **j** Oy           |16|
|l  |   **l** ID, g **l** AD     |14|
|ɹ  |   **r** ED **, b.**    |13|


## <a name="next-steps"></a>Дальнейшие действия

* [Справочная документация по пакету SDK для распознавания речи](speech-sdk.md)
