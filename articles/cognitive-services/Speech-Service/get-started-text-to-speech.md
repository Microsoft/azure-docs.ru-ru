---
title: Краткое руководство по преобразованию текста в речь. Служба "Речь"
titleSuffix: Azure Cognitive Services
description: Узнайте, как использовать пакет SDK службы "Речь" для преобразования текста в речь. Из этого краткого руководства вы узнаете о создании объектов и конструктивных шаблонах, поддерживаемых форматах выходного аудиопотока, CLI службы "Речь" и пользовательских параметрах конфигурации для синтеза речи.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: quickstart
ms.date: 10/01/2020
ms.author: trbye
ms.custom: devx-track-python, devx-track-js, devx-track-csharp, cog-serv-seo-aug-2020
zone_pivot_groups: programming-languages-set-twenty-four
keywords: Преобразование текста в речь
ms.openlocfilehash: 7a41c4d9c1074b376da3de556caf63ced0bc84ec
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102428242"
---
# <a name="get-started-with-text-to-speech"></a>Начало работы с преобразованием текста в речь

::: zone pivot="programming-language-csharp"
[!INCLUDE [C# Basics include](includes/how-to/text-to-speech-basics/text-to-speech-basics-csharp.md)]
::: zone-end

::: zone pivot="programming-language-cpp"
[!INCLUDE [C++ Basics include](includes/how-to/text-to-speech-basics/text-to-speech-basics-cpp.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [Java Basics include](includes/how-to/text-to-speech-basics/text-to-speech-basics-java.md)]
::: zone-end

::: zone pivot="programming-language-javascript"
[!INCLUDE [JavaScript Basics include](includes/how-to/text-to-speech-basics/text-to-speech-basics-javascript.md)]
::: zone-end

::: zone pivot="programming-languages-objectivec-swift"
[!INCLUDE [Objective-C and Swift Basics include](includes/how-to/text-to-speech-basics/text-to-speech-basics-objectivec-swift.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Python Basics include](includes/how-to/text-to-speech-basics/text-to-speech-basics-python.md)]
::: zone-end

::: zone pivot="programming-language-curl"
[!INCLUDE [REST include](includes/how-to/text-to-speech-basics/text-to-speech-basics-curl.md)]
::: zone-end

::: zone pivot="programmer-tool-spx"
[!INCLUDE [CLI Basics include](includes/how-to/text-to-speech-basics/text-to-speech-basics-cli.md)]
::: zone-end

## <a name="get-position-information"></a>Получение сведений о расположении

Иногда в проекте важно иметь информацию о том, когда произносится конкретное слово службой преобразования речи в текст, чтобы выполнять определенные действия с учетом времени этого события. Например, если вы хотите выделять произносимые слова, вам нужны данные для определения выделяемого слова, времени начала и завершения выделения.

Для этого вы можете применить событие `WordBoundary`, предоставляемое в `SpeechSynthesizer`. Это событие создается в начале произнесения каждого нового слова и содержит значения смещений по времени в пределах речевого потока и по длине текста в строке входных данных.

* `AudioOffset` передает значение времени, прошедшего в выходном аудиопотоке с момента начала синтеза до начала произнесения следующего слова. Этот параметр измеряется в единицах HNS (сотни наносекунд), то есть значение 10 000 HNS соответствует 1 миллисекунде.
* `WordOffset` передает положение символа в строке входных данных (исходного текста или [SSML](speech-synthesis-markup.md)), расположенного непосредственно перед произносимым словом.

> [!NOTE]
> События `WordBoundary` создаются в момент, когда становятся доступными исходящие аудиоданные, что происходит раньше начала реального воспроизведения на устройстве вывода. Обеспечение синхронизации исходящего потока с "реальным временем" возлагается на вызывающую сторону.

Применение `WordBoundary` можно изучить в [примерах преобразования текста в речь](https://aka.ms/csspeech/samples) на сайте GitHub.

## <a name="next-steps"></a>Дальнейшие действия

* [Начало работы с набором средств "Пользовательский голос"](how-to-custom-voice.md)
* [Улучшение синтеза с помощью SSML](speech-synthesis-markup.md)
* Узнайте, как использовать [API длинных аудиоматериалов](long-audio-api.md) для примеров текста большого размера, например для книг или новостных статей.
* См. [примеры для краткого руководства](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart) на сайте GitHub.
