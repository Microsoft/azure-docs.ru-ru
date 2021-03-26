---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 01/04/2021
ms.author: trbye
ms.openlocfilehash: 407906727332f3db8d3d0a6840d0c865c6b33ff7
ms.sourcegitcommit: 73d80a95e28618f5dfd719647ff37a8ab157a668
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105609435"
---
Сначала загрузите файл модели ключевых слов с помощью `FromFile()` статической функции, которая возвращает `KeywordRecognitionModel` . Используйте путь к файлу, `.table` скачанному из Speech Studio. Кроме того, вы создаете `AudioConfig` с помощью микрофона по умолчанию, создаете новый экземпляр `KeywordRecognizer` с помощью конфигурации аудио.

```csharp
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;

var keywordModel = KeywordRecognitionModel.FromFile("your/path/to/Activate_device.table");
using var audioConfig = AudioConfig.FromDefaultMicrophoneInput();
using var keywordRecognizer = new KeywordRecognizer(audioConfig);
```

Затем выполнение распознавания ключевых слов выполняется с помощью одного вызова `RecognizeOnceAsync()` , передавая объект модели. Запустится сеанс распознавания ключевых слов, который продолжается до тех пор, пока не будет распознано ключевое слово. Таким образом, этот шаблон разработки обычно используется в многопоточных приложениях или в случаях использования, когда вы можете ожидать пробуждения по неопределенному времени.

```csharp
KeywordRecognitionResult result = await keywordRecognizer.RecognizeOnceAsync(keywordModel);
```

> [!NOTE]
> В приведенном здесь примере используется распознавание локального ключевого слова, так как `SpeechConfig` для контекста проверки подлинности не требуется объект и не выполняется обращение к серверной части. Однако можно выполнить как распознавание ключевых слов, так и проверку, [используя прямое подключение к внутреннему интерфейсу](../../../tutorial-voice-enable-your-bot-speech-sdk.md#view-the-source-code-that-enables-keyword).

### <a name="continuous-recognition"></a>Непрерывное распознавание

Другие классы в пакете SDK для распознавания речи поддерживают непрерывное распознавание (для распознавания речи и намерений) с помощью распознавания ключевых слов. Это позволяет использовать тот же код, который обычно используется для непрерывного распознавания, с возможностью ссылаться на `.table` файл для модели ключевых слов.

Для преобразования речи в текст Используйте тот же шаблон проектирования, что и в [кратком руководстве](../../../get-started-speech-to-text.md?pivots=programming-language-csharp&tabs=script%2cbrowser%2cwindowsinstall#continuous-recognition) , чтобы настроить непрерывное распознавание. Затем замените вызов на `recognizer.StartContinuousRecognitionAsync()` `recognizer.StartKeywordRecognitionAsync(KeywordRecognitionModel)` и передайте `KeywordRecognitionModel` объект. Чтобы предотвратить постоянное распознавание с помощью распознавания ключевых слов, используйте `recognizer.StopKeywordRecognitionAsync()` вместо `recognizer.StopContinuousRecognitionAsync()` .

При распознавании намерения используется идентичный шаблон [`StartKeywordRecognitionAsync`](/dotnet/api/microsoft.cognitiveservices.speech.intent.intentrecognizer.startkeywordrecognitionasync#Microsoft_CognitiveServices_Speech_Intent_IntentRecognizer_StartKeywordRecognitionAsync_Microsoft_CognitiveServices_Speech_KeywordRecognitionModel_) с [`StopKeywordRecognitionAsync`](/dotnet/api/microsoft.cognitiveservices.speech.intent.intentrecognizer.stopkeywordrecognitionasync#Microsoft_CognitiveServices_Speech_Intent_IntentRecognizer_StopKeywordRecognitionAsync) функциями и.
