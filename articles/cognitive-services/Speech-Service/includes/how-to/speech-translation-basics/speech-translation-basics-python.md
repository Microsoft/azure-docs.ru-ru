---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 04/13/2020
ms.author: trbye
ms.openlocfilehash: 679104a1941e677dd78190fb80964a5d39813e61
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102445579"
---
Одной из основных функций речевой службы является возможность распознавания речи и перевода их на другие языки. В этом кратком руководстве вы узнаете, как использовать речевой пакет SDK в приложениях и продуктах для выполнения высококачественного перевода речи. В этом кратком руководстве рассматриваются такие темы, как:

* Преобразование речи в текст
* Преобразование речи на несколько целевых языков
* Выполнение прямого перевода речи в речь

## <a name="skip-to-samples-on-github"></a>Примеры на GitHub

Если вы хотите сразу перейти к примерам кода, см. [этот репозиторий](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/python) на сайте GitHub.

## <a name="prerequisites"></a>Предварительные требования

В этой статье предполагается, что у вас есть учетная запись Azure и подписка на службу "Речь". Если у вас нет учетной записи и подписки, [попробуйте службу "Речь" бесплатно](../../../overview.md#try-the-speech-service-for-free).

## <a name="install-the-speech-sdk"></a>Установка пакета SDK службы "Речь"

Прежде чем выполнять какие-либо действия, необходимо установить пакет SDK для службы "Речь". В зависимости от используемой платформы следуйте инструкциям в разделе <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-sdk#get-the-speech-sdk" target="_blank">Получение речевого пакета SDK</a> статьи _о пакете SDK для распознавания речи_ .

## <a name="import-dependencies"></a>Импорт зависимостей

Чтобы выполнить примеры в этой статье, включите следующие `import` инструкции в начало файла кода Python.

```python
import os
import azure.cognitiveservices.speech as speechsdk
```

## <a name="sensitive-data-and-environment-variables"></a>Конфиденциальные данные и переменные среды

Пример исходного кода в этой статье зависит от переменных среды для хранения конфиденциальных данных, таких как ключ и регион подписки на речевые ресурсы. Файл кода Python содержит два значения, которые назначаются из переменных среды хоста машин, а именно `SPEECH__SUBSCRIPTION__KEY` и `SPEECH__SERVICE__REGION` . Обе эти переменные находятся в глобальной области видимости, делая их доступными в определении функции файла кода. Дополнительные сведения о переменных среды см. в разделе [переменные среды и конфигурация приложения](../../../../cognitive-services-security.md#environment-variables-and-application-configuration).

```python
speech_key, service_region = os.environ['SPEECH__SUBSCRIPTION__KEY'], os.environ['SPEECH__SERVICE__REGION']
```

## <a name="create-a-speech-translation-configuration"></a>Создание конфигурации перевода речи

Чтобы вызвать службу "Речь" с помощью пакета SDK для службы "Речь", необходимо создать [`SpeechTranslationConfig`][config]. Этот класс содержит сведения о вашей подписке, такие как ключ и связанный регион, конечная точка, узел или маркер авторизации.

> [!TIP]
> Независимо от того, используете ли вы распознавание речи, синтез речи, перевод или распознавание намерения, вы всегда создаете конфигурацию.

Существует несколько способов инициализации [`SpeechTranslationConfig`][config].

* С помощью подписки: передайте ключ и связанный с ним регион.
* С помощью конечной точки: передайте конечную точку службы "Речь". Ключ или маркер авторизации являются необязательными.
* С помощью узла: передайте адрес узла. Ключ или маркер авторизации являются необязательными.
* С помощью маркера авторизации: передайте маркер авторизации и связанный регион.

Давайте посмотрим, как создается [`SpeechTranslationConfig`][config] с помощью ключа и региона. Чтобы получить эти учетные данные, выполните инструкции из раздела [Бесплатная пробная подписка на службу "Речь"](../../../overview.md#try-the-speech-service-for-free).

```python
from_language, to_language = 'en-US', 'de'

def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)
```

## <a name="change-source-language"></a>Изменение исходного языка

Одной из распространенных задач перевода речи является указание языка ввода (или исходного кода). Давайте посмотрим, как изменить язык ввода на итальянский. В коде Взаимодействуйте с [`SpeechTranslationConfig`][config] экземпляром, назначив ему `speech_recognition_language` свойство.

```python
def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    # Source (input) language
    translation_config.speech_recognition_language = from_language
```

Свойство [`speech_recognition_language`][recognitionlang] принимает строку формата языкового стандарта. Вы можете указать любое значение в столбце **Языковой стандарт** в списке поддерживаемых [языковых стандартов/языков](../../../language-support.md).

## <a name="add-translation-language"></a>Добавить язык перевода

Другой распространенной задачей перевода речи является указание целевых языков перевода, хотя по крайней мере один является обязательным, но поддерживаются множественные. В следующем фрагменте кода для французского и немецкого языков настраивается цель языка перевода.

```python
def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    translation_config.speech_recognition_language = "it-IT"

    # Translate to languages. See, https://aka.ms/speech/sttt-languages
    translation_config.add_target_language("fr")
    translation_config.add_target_language("de")
```

При каждом вызове метода [`add_target_language`][addlang] указывается новый целевой язык перевода. Иными словами, когда речь распознается от исходного языка, каждый целевой перевод доступен как часть результирующей операции преобразования.

## <a name="initialize-a-translation-recognizer"></a>Инициализация распознавателя трансляции

После создания [`SpeechTranslationConfig`][config], следующим шагом является инициализация [`TranslationRecognizer`][recognizer]. При инициализации [`TranslationRecognizer`][recognizer]ему необходимо передать `translation_config`. Объект конфигурации предоставляет учетные данные, необходимые службе распознавания речи для проверки запроса.

Если вы распознаете речь с помощью стандартного микрофона вашего устройства, вот как должен выглядеть [`TranslationRecognizer`][recognizer]:

```python
def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    translation_config.speech_recognition_language = from_language
    translation_config.add_target_language(to_language)

    recognizer = speechsdk.translation.TranslationRecognizer(
            translation_config=translation_config)
```

Если вы хотите указать входное аудиоустройство, необходимо создать [`AudioConfig`][audioconfig] и указать параметр `audio_config` при инициализации [`TranslationRecognizer`][recognizer].

> [!TIP]
> [Узнайте, как получить идентификатор устройства для входного аудиоустройства](../../../how-to-select-audio-input-devices.md).

Во первых, вы будете ссылаться на `AudioConfig` объект следующим образом:

```python
def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    translation_config.speech_recognition_language = from_language
    for lang in to_languages:
        translation_config.add_target_language(lang)

    audio_config = speechsdk.audio.AudioConfig(use_default_microphone=True)
    recognizer = speechsdk.translation.TranslationRecognizer(
            translation_config=translation_config, audio_config=audio_config)
```

Если вы хотите предоставить звуковой файл, а не использовать микрофон, вам по-прежнему потребуется предоставить `audioConfig`. Однако при создании [`AudioConfig`][audioconfig] вместо вызова с `use_default_microphone=True` необходимо вызвать метод с `filename="path-to-file.wav"` параметром и указать его `filename` .

```python
def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    translation_config.speech_recognition_language = from_language
    for lang in to_languages:
        translation_config.add_target_language(lang)

    audio_config = speechsdk.audio.AudioConfig(filename="path-to-file.wav")
    recognizer = speechsdk.translation.TranslationRecognizer(
            translation_config=translation_config, audio_config=audio_config)
```

## <a name="translate-speech"></a>Преобразование текста

Чтобы перевести речь, речевой пакет SDK использует микрофон или входные звуковые файлы. Распознавание речи выполняется перед переводом речи. После инициализации всех объектов вызовите функцию "распознать один раз" и получите результат.

```python
import os
import azure.cognitiveservices.speech as speechsdk

speech_key, service_region = os.environ['SPEECH__SERVICE__KEY'], os.environ['SPEECH__SERVICE__REGION']
from_language, to_languages = 'en-US', 'de'

def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    translation_config.speech_recognition_language = from_language
    translation_config.add_target_language(to_language)

    recognizer = speechsdk.translation.TranslationRecognizer(
            translation_config=translation_config)
    
    print('Say something...')
    result = recognizer.recognize_once()
    print(get_result_text(reason=result.reason, result=result))

def get_result_text(reason, result):
    reason_format = {
        speechsdk.ResultReason.TranslatedSpeech:
            f'RECOGNIZED "{from_language}": {result.text}\n' +
            f'TRANSLATED into "{to_language}"": {result.translations[to_language]}',
        speechsdk.ResultReason.RecognizedSpeech: f'Recognized: "{result.text}"',
        speechsdk.ResultReason.NoMatch: f'No speech could be recognized: {result.no_match_details}',
        speechsdk.ResultReason.Canceled: f'Speech Recognition canceled: {result.cancellation_details}'
    }
    return reason_format.get(reason, 'Unable to recognize speech')

translate_speech_to_text()
```

Дополнительные сведения о распознавании речи в тексте см. [в разделе основы распознавания речи](../../../get-started-speech-to-text.md).

## <a name="synthesize-translations"></a>Синтезирование переводов

После успешного распознавания речи и перевода результат содержит все переводы в словаре. [`translations`][translations]Ключ словаря — это целевой язык перевода, а значение — переведенный текст. Распознавание речи может быть переведено, а затем синтезировано на другом языке (преобразование речи в речь).

### <a name="event-based-synthesis"></a>Синтез на основе событий

`TranslationRecognizer`Объект предоставляет `Synthesizing` событие. Событие срабатывает несколько раз и предоставляет механизм для получения синтезированного звука из результатов распознавания перевода. Если вы выполняете перевод на несколько языков, см. статью [Ручное синтез](#manual-synthesis). Укажите голосовой голос путем назначения [`voice_name`][voicename] и предоставления обработчика событий для `Synthesizing` события, получения звука. В следующем примере переведенный звук сохраняется в виде *WAV* -файла.

> [!IMPORTANT]
> Синтез на основе событий работает только с одним переводом, не **добавляя** несколько целевых языков преобразования. Кроме того, [`voice_name`][voicename] должен быть тот же язык, что и целевой язык перевода, например; `"de"` может сопоставляться с `"de-DE-Hedda"` .

```python
import os
import azure.cognitiveservices.speech as speechsdk

speech_key, service_region = os.environ['SPEECH__SERVICE__KEY'], os.environ['SPEECH__SERVICE__REGION']
from_language, to_language = 'en-US', 'de'

def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    translation_config.speech_recognition_language = from_language
    translation_config.add_target_language(to_language)

    # See: https://aka.ms/speech/sdkregion#standard-and-neural-voices
    translation_config.voice_name = "de-DE-Hedda"

    recognizer = speechsdk.translation.TranslationRecognizer(
            translation_config=translation_config)

    def synthesis_callback(evt):
        size = len(evt.result.audio)
        print(f'Audio synthesized: {size} byte(s) {"(COMPLETED)" if size == 0 else ""}')

        if size > 0:
            file = open('translation.wav', 'wb+')
            file.write(evt.result.audio)
            file.close()

    recognizer.synthesizing.connect(synthesis_callback)

    print(f'Say something in "{from_language}" and we\'ll translate into "{to_language}".')

    result = recognizer.recognize_once()
    print(get_result_text(reason=result.reason, result=result))

def get_result_text(reason, result):
    reason_format = {
        speechsdk.ResultReason.TranslatedSpeech:
            f'Recognized "{from_language}": {result.text}\n' +
            f'Translated into "{to_language}"": {result.translations[to_language]}',
        speechsdk.ResultReason.RecognizedSpeech: f'Recognized: "{result.text}"',
        speechsdk.ResultReason.NoMatch: f'No speech could be recognized: {result.no_match_details}',
        speechsdk.ResultReason.Canceled: f'Speech Recognition canceled: {result.cancellation_details}'
    }
    return reason_format.get(reason, 'Unable to recognize speech')

translate_speech_to_text()
```

### <a name="manual-synthesis"></a>Ручное синтез

[`translations`][translations]Словарь можно использовать для синтезирования звука из текста перевода. Перебирает каждый перевод и синтезированный перевод. При создании `SpeechSynthesizer` экземпляра `SpeechConfig` объект должен иметь [`speech_synthesis_voice_name`][speechsynthesisvoicename] свойство со значением требуемого голоса. Следующий пример преобразуется в пять языков, и каждый перевод затем синтезирован в звуковой файл на соответствующем языке нейрона.

```python
import os
import azure.cognitiveservices.speech as speechsdk

speech_key, service_region = os.environ['SPEECH__SERVICE__KEY'], os.environ['SPEECH__SERVICE__REGION']
from_language, to_languages = 'en-US', [ 'de', 'en', 'it', 'pt', 'zh-Hans' ]

def translate_speech_to_text():
    translation_config = speechsdk.translation.SpeechTranslationConfig(
            subscription=speech_key, region=service_region)

    translation_config.speech_recognition_language = from_language
    for lang in to_languages:
        translation_config.add_target_language(lang)

    recognizer = speechsdk.translation.TranslationRecognizer(
            translation_config=translation_config)
    
    print('Say something...')
    result = recognizer.recognize_once()
    synthesize_translations(result=result)

def synthesize_translations(result):
    language_to_voice_map = {
        "de": "de-DE-KatjaNeural",
        "en": "en-US-AriaNeural",
        "it": "it-IT-ElsaNeural",
        "pt": "pt-BR-FranciscaNeural",
        "zh-Hans": "zh-CN-XiaoxiaoNeural"
    }
    print(f'Recognized: "{result.text}"')

    for language in result.translations:
        translation = result.translations[language]
        print(f'Translated into "{language}": {translation}')

        speech_config = speechsdk.SpeechConfig(subscription=speech_key, region=service_region)
        speech_config.speech_synthesis_voice_name = language_to_voice_map.get(language)
        
        audio_config = speechsdk.audio.AudioOutputConfig(filename=f'{language}-translation.wav')
        speech_synthesizer = speechsdk.SpeechSynthesizer(speech_config=speech_config, audio_config=audio_config)
        speech_synthesizer.speak_text_async(translation).get()

translate_speech_to_text()
```

Дополнительные сведения о синтезе речи см. [в разделе основы синтеза речи](../../../get-started-text-to-speech.md).

[config]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.translation.speechtranslationconfig
[audioconfig]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.audio.audioconfig
[recognizer]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.translation.translationrecognizer
[recognitionlang]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig
[addlang]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.translation.speechtranslationconfig#add-target-language-language--str-
[translations]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.translation.translationrecognitionresult#translations
[voicename]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.translation.speechtranslationconfig#voice-name
[speechsynthesisvoicename]: /python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig#speech-synthesis-voice-name