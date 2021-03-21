---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 04/13/2020
ms.author: trbye
ms.openlocfilehash: d0cfa7281edfca64616fc62b8af25ef4e5ec598a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102444747"
---
Одной из основных функций речевой службы является возможность распознавания речи и перевода их на другие языки. В этом кратком руководстве вы узнаете, как использовать речевой пакет SDK в приложениях и продуктах для выполнения высококачественного перевода речи. В этом кратком руководстве рассматриваются такие темы, как:

* Преобразование речи в текст
* Преобразование речи на несколько целевых языков
* Выполнение прямого перевода речи в речь

## <a name="skip-to-samples-on-github"></a>Примеры на GitHub

Если вы хотите сразу перейти к примерам кода, см. [этот репозиторий](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/cpp/windows/translate-speech-to-text) на сайте GitHub.

## <a name="prerequisites"></a>Предварительные требования

В этой статье предполагается, что у вас есть учетная запись Azure и подписка на службу "Речь". Если у вас нет учетной записи и подписки, [попробуйте службу "Речь" бесплатно](../../../overview.md#try-the-speech-service-for-free).

## <a name="install-the-speech-sdk"></a>Установка пакета SDK службы "Речь"

Прежде чем выполнять какие-либо действия, необходимо установить пакет SDK для службы "Речь". В зависимости от используемой платформы следуйте инструкциям в разделе <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-sdk#get-the-speech-sdk" target="_blank">Получение речевого пакета SDK</a> статьи _о пакете SDK для распознавания речи_ .

## <a name="import-dependencies"></a>Импорт зависимостей

Чтобы выполнить примеры в этой статье, включите следующие `#include` `using` инструкции и в начало файла кода C++.

```cpp
#include <iostream> // cin, cout
#include <fstream>
#include <string>
#include <stdio.h>
#include <stdlib.h>
#include <speechapi_cxx.h>

using namespace std;
using namespace Microsoft::CognitiveServices::Speech;
using namespace Microsoft::CognitiveServices::Speech::Audio;
using namespace Microsoft::CognitiveServices::Speech::Translation;
```

## <a name="sensitive-data-and-environment-variables"></a>Конфиденциальные данные и переменные среды

Пример исходного кода в этой статье зависит от переменных среды для хранения конфиденциальных данных, таких как ключ и регион подписки на речевые ресурсы. Файл кода C++ содержит два строковых значения, которые назначаются из переменных среды хоста машин, а именно `SPEECH__SUBSCRIPTION__KEY` и `SPEECH__SERVICE__REGION` . Оба эти поля находятся в области класса, делая их доступными в теле метода класса. Дополнительные сведения о переменных среды см. в разделе [переменные среды и конфигурация приложения](../../../../cognitive-services-security.md#environment-variables-and-application-configuration).

```cpp
auto SPEECH__SUBSCRIPTION__KEY = getenv("SPEECH__SUBSCRIPTION__KEY");
auto SPEECH__SERVICE__REGION = getenv("SPEECH__SERVICE__REGION");
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

```cpp
auto SPEECH__SUBSCRIPTION__KEY = getenv("SPEECH__SUBSCRIPTION__KEY");
auto SPEECH__SERVICE__REGION = getenv("SPEECH__SERVICE__REGION");

void translateSpeech() {
    auto config =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);
}

int main(int argc, char** argv) {
    setlocale(LC_ALL, "");
    translateSpeech();
    return 0;
}
```

## <a name="change-source-language"></a>Изменение исходного языка

Одной из распространенных задач перевода речи является указание языка ввода (или исходного кода). Давайте посмотрим, как изменить язык ввода на итальянский. В коде Взаимодействуйте с [`SpeechTranslationConfig`][config] экземпляром, вызвав `SetSpeechRecognitionLanguage` метод.

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    // Source (input) language
    translationConfig->SetSpeechRecognitionLanguage("it-IT");
}
```

Свойство [`SpeechRecognitionLanguage`][recognitionlang] принимает строку формата языкового стандарта. Вы можете указать любое значение в столбце **Языковой стандарт** в списке поддерживаемых [языковых стандартов/языков](../../../language-support.md).

## <a name="add-translation-language"></a>Добавить язык перевода

Другой распространенной задачей перевода речи является указание целевых языков перевода, хотя по крайней мере один является обязательным, но поддерживаются множественные. В следующем фрагменте кода для французского и немецкого языков настраивается цель языка перевода.

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    translationConfig->SetSpeechRecognitionLanguage("it-IT");

    // Translate to languages. See, https://aka.ms/speech/sttt-languages
    translationConfig->AddTargetLanguage("fr");
    translationConfig->AddTargetLanguage("de");
}
```

При каждом вызове метода [`AddTargetLanguage`][addlang] указывается новый целевой язык перевода. Иными словами, когда речь распознается от исходного языка, каждый целевой перевод доступен как часть результирующей операции преобразования.

## <a name="initialize-a-translation-recognizer"></a>Инициализация распознавателя трансляции

После создания [`SpeechTranslationConfig`][config], следующим шагом является инициализация [`TranslationRecognizer`][recognizer]. При инициализации [`TranslationRecognizer`][recognizer]ему необходимо передать `translationConfig`. Объект конфигурации предоставляет учетные данные, необходимые службе распознавания речи для проверки запроса.

Если вы распознаете речь с помощью стандартного микрофона вашего устройства, вот как должен выглядеть [`TranslationRecognizer`][recognizer]:

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    auto fromLanguage = "en-US";
    auto toLanguages = { "it", "fr", "de" };
    translationConfig->SetSpeechRecognitionLanguage(fromLanguage);
    for (auto language : toLanguages) {
        translationConfig->AddTargetLanguage(language);
    }

    auto recognizer = TranslationRecognizer::FromConfig(translationConfig);
}
```

Если вы хотите указать входное аудиоустройство, необходимо создать [`AudioConfig`][audioconfig] и указать параметр `audioConfig` при инициализации [`TranslationRecognizer`][recognizer].

> [!TIP]
> [Узнайте, как получить идентификатор устройства для входного аудиоустройства](../../../how-to-select-audio-input-devices.md).

Во первых, вы будете ссылаться на `AudioConfig` объект следующим образом:

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    auto fromLanguage = "en-US";
    auto toLanguages = { "it", "fr", "de" };
    translationConfig->SetSpeechRecognitionLanguage(fromLanguage);
    for (auto language : toLanguages) {
        translationConfig->AddTargetLanguage(language);
    }

    auto audioConfig = AudioConfig::FromDefaultMicrophoneInput();
    auto recognizer = TranslationRecognizer::FromConfig(translationConfig, audioConfig);
}
```

Если вы хотите предоставить звуковой файл, а не использовать микрофон, вам по-прежнему потребуется предоставить `audioConfig`. Однако при создании [`AudioConfig`][audioconfig] вместо вызова `FromDefaultMicrophoneInput`необходимо вызвать `FromWavFileInput` и передать параметр `filename`.

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    auto fromLanguage = "en-US";
    auto toLanguages = { "it", "fr", "de" };
    translationConfig->SetSpeechRecognitionLanguage(fromLanguage);
    for (auto language : toLanguages) {
        translationConfig->AddTargetLanguage(language);
    }

    auto audioConfig = AudioConfig::FromWavFileInput("YourAudioFile.wav");
    auto recognizer = TranslationRecognizer::FromConfig(translationConfig, audioConfig);
}
```

## <a name="translate-speech"></a>Преобразование текста

Чтобы перевести речь, речевой пакет SDK использует микрофон или входные звуковые файлы. Распознавание речи выполняется перед переводом речи. После инициализации всех объектов вызовите функцию "распознать один раз" и получите результат.

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    string fromLanguage = "en-US";
    string toLanguages[3] = { "it", "fr", "de" };
    translationConfig->SetSpeechRecognitionLanguage(fromLanguage);
    for (auto language : toLanguages) {
        translationConfig->AddTargetLanguage(language);
    }

    auto recognizer = TranslationRecognizer::FromConfig(translationConfig);
    cout << "Say something in '" << fromLanguage << "' and we'll translate...\n";

    auto result = recognizer->RecognizeOnceAsync().get();
    if (result->Reason == ResultReason::TranslatedSpeech)
    {
        cout << "Recognized: \"" << result->Text << "\"" << std::endl;
        for (auto pair : result->Translations)
        {
            auto language = pair.first;
            auto translation = pair.second;
            cout << "Translated into '" << language << "': " << translation << std::endl;
        }
    }
}
```

Дополнительные сведения о распознавании речи в тексте см. [в разделе основы распознавания речи](../../../get-started-speech-to-text.md).

## <a name="synthesize-translations"></a>Синтезирование переводов

После успешного распознавания речи и перевода результат содержит все переводы в словаре. [`Translations`][translations]Ключ словаря — это целевой язык перевода, а значение — переведенный текст. Распознавание речи может быть переведено, а затем синтезировано на другом языке (преобразование речи в речь).

### <a name="event-based-synthesis"></a>Синтез на основе событий

`TranslationRecognizer`Объект предоставляет `Synthesizing` событие. Событие срабатывает несколько раз и предоставляет механизм для получения синтезированного звука из результатов распознавания перевода. Если вы выполняете перевод на несколько языков, см. статью [Ручное синтез](#manual-synthesis). Укажите голосовой голос путем назначения [`SetVoiceName`][voicename] и предоставления обработчика событий для `Synthesizing` события, получения звука. В следующем примере переведенный звук сохраняется в виде *WAV* -файла.

> [!IMPORTANT]
> Синтез на основе событий работает только с одним переводом, не **добавляя** несколько целевых языков преобразования. Кроме того, [`SetVoiceName`][voicename] должен быть тот же язык, что и целевой язык перевода, например; `"de"` может сопоставляться с `"de-DE-Hedda"` .

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    auto fromLanguage = "en-US";
    auto toLanguage = "de";
    translationConfig->SetSpeechRecognitionLanguage(fromLanguage);
    translationConfig->AddTargetLanguage(toLanguage);

    // See: https://aka.ms/speech/sdkregion#standard-and-neural-voices
    translationConfig->SetVoiceName("de-DE-Hedda");

    auto recognizer = TranslationRecognizer::FromConfig(translationConfig);
    recognizer->Synthesizing.Connect([](const TranslationSynthesisEventArgs& e)
        {
            auto audio = e.Result->Audio;
            auto size = audio.size();
            cout << "Audio synthesized: " << size << " byte(s)" << (size == 0 ? "(COMPLETE)" : "") << std::endl;

            if (size > 0) {
                ofstream file("translation.wav", ios::out | ios::binary);
                auto audioData = audio.data();
                file.write((const char*)audioData, sizeof(audio[0]) * size);
                file.close();
            }
        });

    cout << "Say something in '" << fromLanguage << "' and we'll translate...\n";

    auto result = recognizer->RecognizeOnceAsync().get();
    if (result->Reason == ResultReason::TranslatedSpeech)
    {
        cout << "Recognized: \"" << result->Text << "\"" << std::endl;
        for (auto pair : result->Translations)
        {
            auto language = pair.first;
            auto translation = pair.second;
            cout << "Translated into '" << language << "': " << translation << std::endl;
        }
    }
}
```

### <a name="manual-synthesis"></a>Ручное синтез

[`Translations`][translations]Словарь можно использовать для синтезирования звука из текста перевода. Перебирает каждый перевод и синтезированный перевод. При создании `SpeechSynthesizer` экземпляра `SpeechConfig` объект должен иметь [`SetSpeechSynthesisVoiceName`][speechsynthesisvoicename] свойство со значением требуемого голоса. Следующий пример преобразуется в пять языков, и каждый перевод затем синтезирован в звуковой файл на соответствующем языке нейрона.

```cpp
void translateSpeech() {
    auto translationConfig =
        SpeechTranslationConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    auto fromLanguage = "en-US";
    auto toLanguages = { "de", "en", "it", "pt", "zh-Hans" };
    translationConfig->SetSpeechRecognitionLanguage(fromLanguage);
    for (auto language : toLanguages) {
        translationConfig->AddTargetLanguage(language);
    }

    auto recognizer = TranslationRecognizer::FromConfig(translationConfig);

    cout << "Say something in '" << fromLanguage << "' and we'll translate...\n";

    auto result = recognizer->RecognizeOnceAsync().get();
    if (result->Reason == ResultReason::TranslatedSpeech)
    {
        // See: https://aka.ms/speech/sdkregion#standard-and-neural-voices
        map<string, string> languageToVoiceMap;
        languageToVoiceMap["de"] = "de-DE-KatjaNeural";
        languageToVoiceMap["en"] = "en-US-AriaNeural";
        languageToVoiceMap["it"] = "it-IT-ElsaNeural";
        languageToVoiceMap["pt"] = "pt-BR-FranciscaNeural";
        languageToVoiceMap["zh-Hans"] = "zh-CN-XiaoxiaoNeural";

        cout << "Recognized: \"" << result->Text << "\"" << std::endl;
        for (auto pair : result->Translations)
        {
            auto language = pair.first;
            auto translation = pair.second;
            cout << "Translated into '" << language << "': " << translation << std::endl;

            auto speech_config =
                SpeechConfig::FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);
            speech_config->SetSpeechSynthesisVoiceName(languageToVoiceMap[language]);

            auto audio_config = AudioConfig::FromWavFileOutput(language + "-translation.wav");
            auto synthesizer = SpeechSynthesizer::FromConfig(speech_config, audio_config);

            synthesizer->SpeakTextAsync(translation).get();
        }
    }
}
```

Дополнительные сведения о синтезе речи см. [в разделе основы синтеза речи](../../../get-started-text-to-speech.md).

[config]: /cpp/cognitive-services/speech/translation-speechtranslationconfig
[audioconfig]: /cpp/cognitive-services/speech/audio-audioconfig
[recognizer]: /cpp/cognitive-services/speech/translation-translationrecognizer
[recognitionlang]: /cpp/cognitive-services/speech/speechconfig#setspeechrecognitionlanguage
[addlang]: /cpp/cognitive-services/speech/translation-speechtranslationconfig#addtargetlanguage
[translations]: /cpp/cognitive-services/speech/translation-translationrecognitionresult#translations
[voicename]: /cpp/cognitive-services/speech/translation-speechtranslationconfig#setvoicename
[speechsynthesisvoicename]: /cpp/cognitive-services/speech/speechconfig#setspeechsynthesisvoicename