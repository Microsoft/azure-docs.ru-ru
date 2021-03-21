---
title: Использование речевого пакета SDK для оценки произношения
titleSuffix: Azure Cognitive Services
description: Пакет SDK для речевых функций поддерживает оценку произношения, которая оценивает качество речевого ввода с помощью индикаторов точности, владение, полноты и т. д.
services: cognitive-services
author: yulin-li
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 01/12/2021
ms.author: yulili
ms.custom: references_regions
zone_pivot_groups: programming-languages-speech-services-nomore-variant
ms.openlocfilehash: dc1ab8bd1a851f7fafd5c001ac73e66973e1b64c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102051894"
---
# <a name="pronunciation-assessment"></a>Оценка произношения

Оценка произношения проявляется произношение речи и дает отзыв о точности и владение речевого звука.
С помощью оценки произношения языки могут попрактиковаться, получать мгновенные Отзывы и улучшать их произношение, чтобы они могли говорить и представляться уверенно.
Преподаватели могут использовать возможности для вычисления произношения нескольких докладчиков в режиме реального времени.

В этой статье вы узнаете, как настроить `PronunciationAssessmentConfig` и извлечь `PronunciationAssessmentResult` с помощью РЕЧЕВОГО пакета SDK.

> [!NOTE]
> В настоящее время функция оценки произношения поддерживает только язык `en-US` .

## <a name="pronunciation-assessment-with-the-speech-sdk"></a>Оценка произношения с помощью пакета SDK для распознавания речи

В приведенных ниже примерах вы создадите `PronunciationAssessmentConfig` , а затем примените его к `SpeechRecognizer` .

В следующих фрагментах кода показано, как использовать автоматическое определение языка в приложениях.

::: zone pivot="programming-language-csharp"

```csharp
var pronunciationAssessmentConfig = new PronunciationAssessmentConfig(
    "reference text", GradingSystem.HundredMark, Granularity.Phoneme);

using (var recognizer = new SpeechRecognizer(
    speechConfig,
    audioConfig))
{
    // apply the pronunciation assessment configuration to the speech recognizer
    pronunciationAssessmentConfig.ApplyTo(recognizer);
    var speechRecognitionResult = await recognizer.RecognizeOnceAsync();
    var pronunciationAssessmentResult =
        PronunciationAssessmentResult.FromResult(speechRecognitionResult);
    var pronunciationScore = pronunciationAssessmentResult.PronunciationScore;
}
```

::: zone-end

::: zone pivot="programming-language-cpp"

```cpp
auto pronunciationAssessmentConfig =
    PronunciationAssessmentConfig::Create("reference text",
        PronunciationAssessmentGradingSystem::HundredMark,
        PronunciationAssessmentGranularity::Phoneme);

auto recognizer = SpeechRecognizer::FromConfig(
    speechConfig,
    audioConfig);

// apply the pronunciation assessment configuration to the speech recognizer
pronunciationAssessmentConfig->ApplyTo(recognizer);
speechRecognitionResult = recognizer->RecognizeOnceAsync().get();
auto pronunciationAssessmentResult =
    PronunciationAssessmentResult::FromResult(speechRecognitionResult);
auto pronunciationScore = pronunciationAssessmentResult->PronunciationScore;
```

::: zone-end

::: zone pivot="programming-language-java"

```java
PronunciationAssessmentConfig pronunciationAssessmentConfig =
    new PronunciationAssessmentConfig("reference text", 
        PronunciationAssessmentGradingSystem.HundredMark,
        PronunciationAssessmentGranularity.Phoneme);

SpeechRecognizer recognizer = new SpeechRecognizer(
    speechConfig,
    audioConfig);

// apply the pronunciation assessment configuration to the speech recognizer
pronunciationAssessmentConfig.applyTo(recognizer);
Future<SpeechRecognitionResult> future = recognizer.recognizeOnceAsync();
SpeechRecognitionResult result = future.get(30, TimeUnit.SECONDS);
PronunciationAssessmentResult pronunciationAssessmentResult =
    PronunciationAssessmentResult.fromResult(result);
Double pronunciationScore = pronunciationAssessmentResult.getPronunciationScore();

recognizer.close();
speechConfig.close();
audioConfig.close();
pronunciationAssessmentConfig.close();
result.close();
```

::: zone-end

::: zone pivot="programming-language-python"

```Python
pronunciation_assessment_config = \
        speechsdk.PronunciationAssessmentConfig(reference_text='reference text',
                grading_system=msspeech.PronunciationAssessmentGradingSystem.HundredMark,
                granularity=msspeech.PronunciationAssessmentGranularity.Phoneme)
speech_recognizer = speechsdk.SpeechRecognizer(
        speech_config=speech_config, \
        audio_config=audio_config)

# apply the pronunciation assessment configuration to the speech recognizer
pronunciation_assessment_config.apply_to(speech_recognizer)
result = speech_recognizer.recognize_once()
pronunciation_assessment_result = speechsdk.PronunciationAssessmentResult(result)
pronunciation_score = pronunciation_assessment_result.pronunciation_score
```

::: zone-end

::: zone pivot="programming-language-javascript"

```Javascript
var pronunciationAssessmentConfig = new SpeechSDK.PronunciationAssessmentConfig("reference text",
    PronunciationAssessmentGradingSystem.HundredMark,
    PronunciationAssessmentGranularity.Word, true);
var speechRecognizer = SpeechSDK.SpeechRecognizer.FromConfig(speechConfig, audioConfig);
// apply the pronunciation assessment configuration to the speech recognizer
pronunciationAssessmentConfig.applyTo(speechRecognizer);

speechRecognizer.recognizeOnceAsync((result: SpeechSDK.SpeechRecognitionResult) => {
        var pronunciationAssessmentResult = SpeechSDK.PronunciationAssessmentResult.fromResult(result);
        var pronunciationScore = pronunciationAssessmentResult.pronunciationScore;
        var wordLevelResult = pronunciationAssessmentResult.detailResult.Words;
},
{});
```

::: zone-end

::: zone pivot="programming-language-objectivec"

```Objective-C
SPXPronunciationAssessmentConfiguration* pronunciationAssessmentConfig =
    [[SPXPronunciationAssessmentConfiguration alloc]init:@"reference text"
                                           gradingSystem:SPXPronunciationAssessmentGradingSystem_HundredMark
                                             granularity:SPXPronunciationAssessmentGranularity_Phoneme];

SPXSpeechRecognizer* speechRecognizer = \
        [[SPXSpeechRecognizer alloc] initWithSpeechConfiguration:speechConfig
                                              audioConfiguration:audioConfig];

// apply the pronunciation assessment configuration to the speech recognizer
[pronunciationAssessmentConfig applyToRecognizer:speechRecognizer];

SPXSpeechRecognitionResult *result = [speechRecognizer recognizeOnce];
SPXPronunciationAssessmentResult* pronunciationAssessmentResult = [[SPXPronunciationAssessmentResult alloc] init:result];
double pronunciationScore = pronunciationAssessmentResult.pronunciationScore;
```

::: zone-end

### <a name="pronunciation-assessment-configuration-parameters"></a>Параметры конфигурации оценки произношения

В этой таблице перечислены параметры конфигурации для оценки произношения.

| Параметр | Описание | Необходим? |
|-----------|-------------|---------------------|
| референцетекст | Текст, по которому будет оцениваться произношение. | Обязательно |
| градингсистем | Система точек для калибровки оценок. `FivePoint`Система дает 0-5 оценку с плавающей запятой и `HundredMark` предоставляет 0-100 с плавающей запятой. Значение по умолчанию: `FivePoint`. | Необязательно |
| Степень детализации данных | Гранулярность оценки. Допустимые значения: `Phoneme` , которые показывают оценку для полного текста, Word и phoneme Level, `Word` который показывает оценку на полном тексте и на уровне слов, то `FullText` есть только на уровне Full Text. Значение по умолчанию: `Phoneme`. | Необязательно |
| енаблемискуе | Включает вычисление мискуе. Если этот параметр включен, то произведение слов будет сравниваться с текстом ссылки и будет отмечено с помощью пропуска или вставки на основе сравнения. Допустимые значения: `False` и `True`. Значение по умолчанию: `False`. | Необязательно |
| сценариоид | Идентификатор GUID, указывающий настроенную систему точек. | Необязательно |

### <a name="pronunciation-assessment-result-parameters"></a>Параметры результатов оценки произношения

В этой таблице перечислены параметры результатов оценки произношения.

| Параметр | Описание |
|-----------|-------------|
| `AccuracyScore` | Точность произношения речи. Точность указывает, насколько близко фонемы соответствует произношению собственного динамика. Оценка точности слов и полнотекстового индекса вычисляется из оценки точности на уровне phoneme. |
| `FluencyScore` | Владение заданной речи. Владение указывает, насколько близко речь соответствует использованию собственного динамика для автоматического разрыва слов. |
| `CompletenessScore` | Полнота речи, определяемая путем вычисления соотношения слов со словами для ссылки на ввод текста. |
| `PronunciationScore` | Общая оценка, показывающая качество произношения заданной речи. Это статистическое вычисление `AccuracyScore` из `FluencyScore` и `CompletenessScore` с весом. |
| `ErrorType` | Это значение указывает, пропущено ли слово, вставлено или неверно, по сравнению с `ReferenceText` . Возможными значениями являются `None` (это означает отсутствие ошибок в этом слове), `Omission` `Insertion` и `Mispronunciation` . |

## <a name="next-steps"></a>Дальнейшие действия

<!-- TODO: update JavaScript sample links after release -->

::: zone pivot="programming-language-csharp"
* См. [пример кода](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/csharp/sharedcontent/console/speech_recognition_samples.cs#L949) на сайте GitHub для оценки произношения.
::: zone-end

::: zone pivot="programming-language-cpp"
* См. [пример кода](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/cpp/windows/console/samples/speech_recognition_samples.cpp#L633) на сайте GitHub для оценки произношения.
::: zone-end

::: zone pivot="programming-language-java"
* См. [пример кода](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/java/jre/console/src/com/microsoft/cognitiveservices/speech/samples/console/SpeechRecognitionSamples.java#L697) на сайте GitHub для оценки произношения.
::: zone-end

::: zone pivot="programming-language-python"
* См. [пример кода](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/python/console/speech_sample.py#L576) на сайте GitHub для оценки произношения.
::: zone-end

::: zone pivot="programming-language-objectivec"
* См. [пример кода](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/objective-c/ios/speech-samples/speech-samples/ViewController.m#L642) на сайте GitHub для оценки произношения.
::: zone-end

* [Справочная документация по пакету SDK для распознавания речи](speech-sdk.md)
