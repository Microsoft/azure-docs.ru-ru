---
author: v-jaswel
ms.service: cognitive-services
ms.topic: include
ms.date: 10/07/2020
ms.author: v-jawe
ms.custom: references_regions
ms.openlocfilehash: 3953e7182d90cb1737a2083e2315612b07f2eb84
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105103954"
---
В этом руководстве показаны базовые шаблоны разработки для Распознавания говорящего с помощью пакета SDK службы Речи. В статье рассматриваются следующие темы:

* Проверка, зависящая от текста и не зависящая от текста.
* Идентификация говорящего для идентификации образца голоса в группе голосов.
* Удаление речевых профилей.

Общие сведения о концепциях распознавания речи см. в статье [Обзор](../../../speaker-recognition-overview.md).

## <a name="skip-to-samples-on-github"></a>Примеры на GitHub

Если вы хотите сразу перейти к примерам кода, см. [этот репозиторий](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/fa6428a0837779cbeae172688e0286625e340942/quickstart/javascript/node/speaker-recognition) на сайте GitHub.

## <a name="prerequisites"></a>Предварительные требования

В этой статье предполагается, что у вас есть учетная запись Azure и подписка на службу "Речь". Если у вас нет учетной записи и подписки, [попробуйте службу "Речь" бесплатно](../../../overview.md#try-the-speech-service-for-free).

> [!IMPORTANT]
> Распознавание говорящего в настоящее время поддерживается *только* в ресурсах распознавания речи Azure, созданных в регионе `westus`.

## <a name="install-the-speech-sdk"></a>Установка пакета SDK службы "Речь"

Сначала необходимо установить <a href="https://www.npmjs.com/package/microsoft-cognitiveservices-speech-sdk" target="_blank">пакет SDK службы "Речь" для JavaScript </a>. В зависимости от используемой платформы выполните следующие действия:

- <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-sdk?tabs=nodejs#get-the-speech-sdk" target="_blank">Node.js <span 
class="docon docon-navigate-external x-hidden-focus"></span></a>
- <a href="/azure/cognitive-services/speech-service/speech-sdk?tabs=browser#get-the-speech-sdk" target="_blank">Веб-браузер </a>

Кроме того, в зависимости от целевой среды используйте один из следующих параметров:

# <a name="script"></a>[script](#tab/script)

Скачайте пакет <a href="https://aka.ms/csspeech/jsbrowserpackage" target="_blank">SDK службы "Речь" для JavaScript</a> , извлеките из него файл *microsoft.cognitiveservices.speech.sdk.bundle.js* и поместите его в папку, доступную для HTML-файла.

```html
<script src="microsoft.cognitiveservices.speech.sdk.bundle.js"></script>;
```

> [!TIP]
> Если в качестве платформы вы используете веб-браузер и применяете тег `<script>`, использовать префикс `sdk` не нужно. Префикс `sdk` является псевдонимом для присвоения имени модулю `require`.

# <a name="import"></a>[import](#tab/import)

```javascript
import * from "microsoft-cognitiveservices-speech-sdk";
```

Дополнительные сведения о директиве `import` см. в статье об <a href="https://javascript.info/import-export" target="_blank">экспорте и импорте </a>.

# <a name="require"></a>[require](#tab/require)

```javascript
const sdk = require("microsoft-cognitiveservices-speech-sdk");
```

Дополнительные сведения о функции `require` см. <a href="https://nodejs.org/en/knowledge/getting-started/what-is-require/" target="_blank">здесь </a>.

---

## <a name="import-dependencies"></a>Импорт зависимостей

Чтобы выполнить примеры из этой статьи, добавьте в начале JS-файла следующие инструкции.

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/speech/speaker-recognition.js" id="dependencies":::

Эти операторы импортируют необходимые библиотеки и получают ключ к подписке службы "Речь" и сведения о регионе из ваших переменных среды. Они также указывают путь к аудиофайлам, которые будут использоваться в следующих задачах.

## <a name="create-helper-function"></a>Создание вспомогательной функции

Добавьте следующую вспомогательную функцию для чтения аудиофайлов в потоках, используемых службой "Речь".

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/speech/speaker-recognition.js" id="helpers":::

В этой функции для создания объекта [AudioConfig](/javascript/api/microsoft-cognitiveservices-speech-sdk/audioconfig) используются методы [AudioInputStream.createPushStream](/javascript/api/microsoft-cognitiveservices-speech-sdk/audioinputstream#createpushstream-audiostreamformat-) и [AudioConfig.fromStreamInput](/javascript/api/microsoft-cognitiveservices-speech-sdk/audioconfig#fromstreaminput-audioinputstream---pullaudioinputstreamcallback-). Этот объект `AudioConfig` представляет звуковой поток. Некоторые из этих объектов `AudioConfig` понадобятся в следующих задачах.

## <a name="text-dependent-verification"></a>Проверка, зависящая от текста

Проверка говорящего является подтверждением того, что говорящий соответствует известному или **зарегистрированному** голосу. В первую очередь необходимо **зарегистрировать** речевой профиль, чтобы службе было с чем сравнивать будущие примеры голоса. В этом примере вы регистрируете профиль, используя стратегию **text-dependent**, при применении которой нужно указывать парольную фразу как для регистрации, так и для проверки. Список поддерживаемых парольных фраз см. в [справочной документации](/rest/api/speakerrecognition/).

### <a name="textdependentverification-function"></a>Функция TextDependentVerification

Начните с создания функции `TextDependentVerification`.

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/speech/speaker-recognition.js" id="text_dependent_verification":::

Эта функция создает объект [VoiceProfile](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofile) с помощью метода [VoiceProfileClient.createProfileAsync](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofileclient#createprofileasync-voiceprofiletype--string---e--voiceprofile-----void---e--string-----void-). Обратите внимание, что существует три [типа](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofiletype) `VoiceProfile`:

- TextIndependentIdentification
- TextDependentVerification
- TextIndependentVerification

В этом примере `VoiceProfileType.TextDependentVerification` передается в `VoiceProfileClient.createProfileAsync`.

Затем нужно вызвать две вспомогательные функции `AddEnrollmentsToTextDependentProfile` и `SpeakerVerify`, которые вы определяете далее. Наконец, для удаления профиля вызовите [VoiceProfileClient.deleteProfileAsync](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofileclient#deleteprofileasync-voiceprofile---response--voiceprofileresult-----void---e--string-----void-).

### <a name="addenrollmentstotextdependentprofile-function"></a>Функция AddEnrollmentsToTextDependentProfile

Определите следующую функцию, чтобы зарегистрировать речевой профиль.

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/speech/speaker-recognition.js" id="add_enrollments_dependent":::

В этой функции вы вызываете функцию `GetAudioConfigFromFile`, определенную ранее, для создания объектов `AudioConfig` из примеров звуковых данных. Эти примеры звуковых данных содержат парольную фразу, например My voice is my passport, verify me (Мой голос — мой паспорт, проверь меня). Затем вы зарегистрируете эти примеры звуковых данных, используя метод [VoiceProfileClient.enrollProfileAsync](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofileclient#enrollprofileasync-voiceprofile--audioconfig---e--voiceprofileenrollmentresult-----void---e--string-----void-).

### <a name="speakerverify-function"></a>Функция SpeakerVerify

Определите `SpeakerVerify` следующим образом.

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/speech/speaker-recognition.js" id="speaker_verify":::

В этой функции вы создаете объект [SpeakerVerificationModel](/javascript/api/microsoft-cognitiveservices-speech-sdk/speakerverificationmodel) с помощью метода [SpeakerVerificationModel.FromProfile](/javascript/api/microsoft-cognitiveservices-speech-sdk/speakerverificationmodel#fromprofile-voiceprofile-), передавая ранее созданный объект [VoiceProfile](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofile).

Далее вызовите метод [SpeechRecognizer.recognizeOnceAsync](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer#recognizeonceasync--e--speechrecognitionresult-----void---e--string-----void-), чтобы проверить пример звуковых данных, который содержит ту же парольную фразу, что и в ранее зарегистрированном примере звуковых данных. `SpeechRecognizer.recognizeOnceAsync` возвращает объект [SpeakerRecognitionResult](/javascript/api/microsoft-cognitiveservices-speech-sdk/speakerrecognitionresult), свойство `score` которого содержит оценку схожести в диапазоне 0.0–1.0. Объект `SpeakerRecognitionResult` также содержит свойство `reason` типа [ResultReason](/javascript/api/microsoft-cognitiveservices-speech-sdk/resultreason). Если проверка прошла успешно, тогда свойство `reason` должно иметь значение `RecognizedSpeaker`.

## <a name="text-independent-verification"></a>Проверка, не зависящая от текста

Проверка, **не зависящая от текста**, отличается от проверки, **зависящей от текста**, в следующих аспектах.

* Определенная парольная фраза не требуется, можно произносить что угодно.
* Не требуются три примера звуковых данных, но *требуется*, чтобы длительность звуковых данных составляла 20 секунд.

### <a name="textindependentverification-function"></a>Функция TextIndependentVerification

Начните с создания функции `TextIndependentVerification`.

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/speech/speaker-recognition.js" id="text_independent_verification":::

Как и функция `TextDependentVerification`, эта функция создает объект [VoiceProfile](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofile) с помощью метода [VoiceProfileClient.createProfileAsync](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofileclient#createprofileasync-voiceprofiletype--string---e--voiceprofile-----void---e--string-----void-).

В этом примере `VoiceProfileType.TextIndependentVerification` передается в `createProfileAsync`.

Затем нужно вызвать две вспомогательные функции: `AddEnrollmentsToTextIndependentProfile` (определяется далее) и `SpeakerVerify` (уже определена). Наконец, для удаления профиля вызовите [VoiceProfileClient.deleteProfileAsync](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofileclient#deleteprofileasync-voiceprofile---response--voiceprofileresult-----void---e--string-----void-).

### <a name="addenrollmentstotextindependentprofile"></a>AddEnrollmentsToTextIndependentProfile

Определите следующую функцию, чтобы зарегистрировать речевой профиль.

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/speech/speaker-recognition.js" id="add_enrollments_independent":::

В этой функции вы вызываете функцию `GetAudioConfigFromFile`, определенную ранее, для создания объектов `AudioConfig` из примеров звуковых данных. Затем вы зарегистрируете эти примеры звуковых данных, используя метод [VoiceProfileClient.enrollProfileAsync](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofileclient#enrollprofileasync-voiceprofile--audioconfig---e--voiceprofileenrollmentresult-----void---e--string-----void-).

## <a name="speaker-identification"></a>Идентификация говорящего

Идентификация говорящего используется для определения, **кто** именно говорит из определенной группы зарегистрированных голосов. Этот процесс похож на проверку, **не зависящую от текста**. Основное отличие заключается в том, что можно сравнивать не с одним, а сразу с несколькими речевыми профилями.

### <a name="textindependentidentification-function"></a>Функция TextIndependentIdentification

Начните с создания функции `TextIndependentIdentification`.

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/speech/speaker-recognition.js" id="text_independent_indentification":::

Как и функции `TextDependentVerification` и `TextIndependentVerification`, эта функция создает объект [VoiceProfile](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofile) с помощью метода [VoiceProfileClient.createProfileAsync](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofileclient#createprofileasync-voiceprofiletype--string---e--voiceprofile-----void---e--string-----void-).

В этом примере `VoiceProfileType.TextIndependentIdentification` передается в `VoiceProfileClient.createProfileAsync`.

Затем нужно вызвать две вспомогательные функции: уже определенную вами `AddEnrollmentsToTextIndependentProfile` и `SpeakerIdentify`, которую вы определите. Наконец, для удаления профиля вызовите [VoiceProfileClient.deleteProfileAsync](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofileclient#deleteprofileasync-voiceprofile---response--voiceprofileresult-----void---e--string-----void-).

### <a name="speakeridentify-function"></a>Функция SpeakerIdentify

Определите функцию `SpeakerIdentify` следующим образом.

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/speech/speaker-recognition.js" id="speaker_identify":::

В этой функции вы создаете объект [SpeakerIdentificationModel](/javascript/api/microsoft-cognitiveservices-speech-sdk/speakeridentificationmodel) с помощью метода [SpeakerIdentificationModel.fromProfiles](/javascript/api/microsoft-cognitiveservices-speech-sdk/speakeridentificationmodel#fromprofiles-voiceprofile---), передавая ранее созданный объект [VoiceProfile](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofile).

Затем вызовите метод [SpeechRecognizer.recognizeOnceAsync](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer#recognizeonceasync--e--speechrecognitionresult-----void---e--string-----void-) и передайте пример звуковых данных.
Метод `SpeechRecognizer.recognizeOnceAsync` пытается определить речь для этого примера звуковых данных на основе объектов `VoiceProfile`, использованных для создания `SpeakerIdentificationModel`. Он возвращает объект [SpeakerRecognitionResult](/javascript/api/microsoft-cognitiveservices-speech-sdk/speakerrecognitionresult), свойство `profileId` которого определяет совпадение с `VoiceProfile`, если таковое имеется, а свойство `score` содержит оценку схожести в диапазоне 0.0–1.0.

## <a name="main-function"></a>Функция main

Наконец, определите функцию `main` следующим образом.

:::code language="javascript" source="~/cognitive-services-quickstart-code/javascript/speech/speaker-recognition.js" id="main":::

Эта функция создает объект [VoiceProfileClient](/javascript/api/microsoft-cognitiveservices-speech-sdk/voiceprofileclient), который используется при создании, регистрации и удалении голосовых профилей. Затем она вызывает функции, определенные ранее.