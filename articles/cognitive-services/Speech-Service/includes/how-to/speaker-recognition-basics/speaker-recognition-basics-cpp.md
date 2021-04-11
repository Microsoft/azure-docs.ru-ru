---
author: v-jaswel
ms.service: cognitive-services
ms.topic: include
ms.date: 09/28/2020
ms.author: v-jawe
ms.custom: references_regions
ms.openlocfilehash: 48a4c23fac5dd6b5d18de7f62bd38c4a78a5ce18
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105104582"
---
В этом руководстве показаны базовые шаблоны разработки для Распознавания говорящего с помощью пакета SDK службы Речи. В статье рассматриваются следующие темы:

* Проверка, зависящая от текста и не зависящая от текста.
* Идентификация говорящего для идентификации образца голоса в группе голосов.
* Удаление речевых профилей.

Общие сведения о концепциях распознавания речи см. в статье [Обзор](../../../speaker-recognition-overview.md).

## <a name="skip-to-samples-on-github"></a>Примеры на GitHub

Если вы хотите сразу перейти к примерам кода, см. [этот репозиторий](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/cpp/windows) на сайте GitHub.

## <a name="prerequisites"></a>Предварительные требования

В этой статье предполагается, что у вас есть учетная запись Azure и подписка на службу "Речь". Если у вас нет учетной записи и подписки, [попробуйте службу "Речь" бесплатно](../../../overview.md#try-the-speech-service-for-free).

> [!IMPORTANT]
> Распознавание говорящего в настоящее время поддерживается *только* в ресурсах распознавания речи Azure, созданных в регионе `westus`.

## <a name="install-the-speech-sdk"></a>Установка пакета SDK службы "Речь"

Прежде чем выполнять какие-либо действия, необходимо установить пакет SDK для службы "Речь". В зависимости от используемой платформы выполните следующие действия:

* <a href="/azure/cognitive-services/speech-service/quickstarts/setup-platform?pivots=programming-language-cpp&tabs=linux" target="_blank">Linux </a>
* <a href="/azure/cognitive-services/speech-service/quickstarts/setup-platform?pivots=programming-language-cpp&tabs=macos" target="_blank">macOS </a>
* <a href="/azure/cognitive-services/speech-service/quickstarts/setup-platform?pivots=programming-language-cpp&tabs=windows" target="_blank">Windows </a>

## <a name="import-dependencies"></a>Импорт зависимостей

Чтобы выполнить примеры из этой статьи, добавьте в начале CPP-файла следующие инструкции.

:::code language="cpp" source="~/cognitive-services-quickstart-code/cpp/speech/speaker-recognition.cpp" id="dependencies":::

## <a name="create-a-speech-configuration"></a>Создание конфигурации службы "Речь"

Чтобы вызвать службу "Речь" с помощью пакета SDK для службы "Речь", необходимо создать [`SpeechConfig`](/cpp/cognitive-services/speech/speechconfig). Этот класс содержит сведения о вашей подписке, такие как ключ и связанный регион, конечная точка, узел или маркер авторизации.

:::code language="cpp" source="~/cognitive-services-quickstart-code/cpp/speech/speaker-recognition.cpp" id="get_speech_config":::

## <a name="text-dependent-verification"></a>Проверка, зависящая от текста

Проверка говорящего является подтверждением того, что говорящий соответствует известному или **зарегистрированному** голосу. В первую очередь необходимо **зарегистрировать** речевой профиль, чтобы службе было с чем сравнивать будущие примеры голоса. В этом примере вы регистрируете профиль, используя стратегию **text-dependent**, при применении которой нужно указывать парольную фразу как для регистрации, так и для проверки. Список поддерживаемых парольных фраз см. в [справочной документации](/rest/api/speakerrecognition/).

### <a name="textdependentverification-function"></a>Функция TextDependentVerification

Начните с создания функции `TextDependentVerification`.

:::code language="cpp" source="~/cognitive-services-quickstart-code/cpp/speech/speaker-recognition.cpp" id="text_dependent_verification":::

Эта функция создает объект [VoiceProfile](/cpp/cognitive-services/speech/voiceprofile) с помощью метода [CreateProfileAsync](/cpp/cognitive-services/speech/voiceprofileclient#createprofileasync). Обратите внимание, что существует три [типа](/cpp/cognitive-services/speech/microsoft-cognitiveservices-speech-namespace#enum-voiceprofiletype) `VoiceProfile`:

- TextIndependentIdentification
- TextDependentVerification
- TextIndependentVerification

В этом примере `VoiceProfileType::TextDependentVerification` передается в `CreateProfileAsync`.

Затем нужно вызвать две вспомогательные функции `AddEnrollmentsToTextDependentProfile` и `SpeakerVerify`, которые вы определяете далее. Наконец, вызовите [DeleteProfileAsync](/cpp/cognitive-services/speech/voiceprofileclient#deleteprofileasync), чтобы очистить профиль.

### <a name="addenrollmentstotextdependentprofile-function"></a>Функция AddEnrollmentsToTextDependentProfile

Определите следующую функцию, чтобы зарегистрировать речевой профиль.

:::code language="cpp" source="~/cognitive-services-quickstart-code/cpp/speech/speaker-recognition.cpp" id="add_enrollments_dependent":::

В этой функции вы регистрируете звуковые образцы в цикле `while`, отслеживающем количество оставшихся образцов, которые должны быть зарегистрированы. В каждой итерации [EnrollProfileAsync](/cpp/cognitive-services/speech/voiceprofileclient#enrollprofileasync) предлагает произнести парольную фразу в микрофон и добавлять этот образец в речевой профиль.

### <a name="speakerverify-function"></a>Функция SpeakerVerify

Определите `SpeakerVerify` следующим образом.

:::code language="cpp" source="~/cognitive-services-quickstart-code/cpp/speech/speaker-recognition.cpp" id="speaker_verify":::

В этой функции вы создаете объект [SpeakerVerificationModel](/cpp/cognitive-services/speech/speakerverificationmodel) с помощью метода [SpeakerVerificationModel::FromProfile](/cpp/cognitive-services/speech/speakerverificationmodel#fromprofile), передавая ранее созданный объект [VoiceProfile](/cpp/cognitive-services/speech/voiceprofile).

Затем [SpeechRecognizer::RecognizeOnceAsync](/cpp/cognitive-services/speech/speechrecognizer#recognizeonceasync) предложит снова произнести парольную фразу, но на этот раз она будет проверяться на соответствие речевому профилю, после чего будет возвращаться показатель подобия от 0,0 до 1,0. Объект [SpeakerRecognitionResult](/cpp/cognitive-services/speech/speakerrecognitionresult) также возвращает `Accept` или `Reject` в зависимости от того, соответствует ли парольная фраза.

## <a name="text-independent-verification"></a>Проверка, не зависящая от текста

Проверка, **не зависящая от текста**, отличается от проверки, **зависящей от текста**, в следующих аспектах.

* Определенная парольная фраза не требуется, можно произносить что угодно.
* Не требуются три примера звуковых данных, но *требуется*, чтобы длительность звуковых данных составляла 20 секунд.

### <a name="textindependentverification-function"></a>Функция TextIndependentVerification

Начните с создания функции `TextIndependentVerification`.

:::code language="cpp" source="~/cognitive-services-quickstart-code/cpp/speech/speaker-recognition.cpp" id="text_independent_verification":::

Как и функция `TextDependentVerification`, эта функция создает объект [VoiceProfile](/cpp/cognitive-services/speech/voiceprofile) с помощью метода [CreateProfileAsync](/cpp/cognitive-services/speech/voiceprofileclient#createprofileasync).

В этом примере `VoiceProfileType::TextIndependentVerification` передается в `CreateProfileAsync`.

Затем нужно вызвать две вспомогательные функции: `AddEnrollmentsToTextIndependentProfile` (определяется далее) и `SpeakerVerify` (уже определена). Наконец, вызовите [DeleteProfileAsync](/cpp/cognitive-services/speech/voiceprofileclient#deleteprofileasync), чтобы очистить профиль.

### <a name="addenrollmentstotextindependentprofile"></a>AddEnrollmentsToTextIndependentProfile

Определите следующую функцию, чтобы зарегистрировать речевой профиль.

:::code language="cpp" source="~/cognitive-services-quickstart-code/cpp/speech/speaker-recognition.cpp" id="add_enrollments_independent":::

В этой функции вы регистрируете звуковые образцы в цикле `while`, отслеживающем количество секунд оставшегося звука, которое должно быть зарегистрировано. В каждой итерации [EnrollProfileAsync](/cpp/cognitive-services/speech/voiceprofileclient#enrollprofileasync) предлагает говорить в микрофон и добавлять этот образец в речевой профиль.

## <a name="speaker-identification"></a>Идентификация говорящего

Идентификация говорящего используется для определения, **кто** именно говорит из определенной группы зарегистрированных голосов. Этот процесс очень похож на проверку, **не зависящую от текста**. Основное отличие заключается в том, что можно сравнивать не с одним, а сразу с несколькими речевыми профилями.

### <a name="textindependentidentification-function"></a>Функция TextIndependentIdentification

Начните с создания функции `TextIndependentIdentification`.

:::code language="cpp" source="~/cognitive-services-quickstart-code/cpp/speech/speaker-recognition.cpp" id="text_independent_indentification":::

Как и функции `TextDependentVerification` и `TextIndependentVerification`, эта функция создает объект [VoiceProfile](/cpp/cognitive-services/speech/voiceprofile) с помощью метода [CreateProfileAsync](/cpp/cognitive-services/speech/voiceprofileclient#createprofileasync).

В этом примере `VoiceProfileType::TextIndependentIdentification` передается в `CreateProfileAsync`.

Затем нужно вызвать две вспомогательные функции: уже определенную вами `AddEnrollmentsToTextIndependentProfile` и `SpeakerIdentify`, которую вы определите. Наконец, вызовите [DeleteProfileAsync](/cpp/cognitive-services/speech/voiceprofileclient#deleteprofileasync), чтобы очистить профиль.

### <a name="speakeridentify-function"></a>Функция SpeakerIdentify

Определите функцию `SpeakerIdentify` следующим образом.

:::code language="cpp" source="~/cognitive-services-quickstart-code/cpp/speech/speaker-recognition.cpp" id="speaker_identify":::

В этой функции вы создаете объект [SpeakerIdentificationModel](/cpp/cognitive-services/speech/speakeridentificationmodel) с помощью метода [SpeakerIdentificationModel::FromProfiles](/cpp/cognitive-services/speech/speakeridentificationmodel#fromprofiles). Метод `SpeakerIdentificationModel::FromProfiles` принимает список объектов [VoiceProfile](/cpp/cognitive-services/speech/voiceprofile). В этом случае вы просто передадите ранее созданный объект `VoiceProfile`. Однако при необходимости можно передать несколько объектов `VoiceProfile`, каждый из которых зарегистрирован с помощью звуковых примеров другого голоса.

Затем [SpeechRecognizer::RecognizeOnceAsync](/cpp/cognitive-services/speech/speechrecognizer#recognizeonceasync) предлагает говорить снова. На этот раз он сравнивает ваш голос с зарегистрированными речевыми профилями и возвращает самый похожий речевой профиль.

## <a name="main-function"></a>Функция main

Наконец, определите функцию `main` следующим образом.

:::code language="cpp" source="~/cognitive-services-quickstart-code/cpp/speech/speaker-recognition.cpp" id="main":::

Эта функция просто вызывает ранее определенные функции. Однако сначала она создает объекты [VoiceProfileClient](/cpp/cognitive-services/speech/voiceprofileclient) и [SpeakerRecognizer](/cpp/cognitive-services/speech/speakerrecognizer).

```
auto speech_config = GetSpeechConfig();
auto client = VoiceProfileClient::FromConfig(speech_config);
auto recognizer = SpeakerRecognizer::FromConfig(speech_config, audio_config);
```

`VoiceProfileClient` используется для создания, регистрации и удаления речевых профилей. `SpeakerRecognizer` используется для проверки примеров речи по одному или нескольким зарегистрированным речевым профилям.

## <a name="changing-audio-input-type"></a>Изменение типа входных звуковых данных

В примерах, приведенных в этой статье, для записи образцов звука используется микрофон устройства по умолчанию. Если же вместо ввода с микрофона необходимо использовать звуковые файлы, просто измените следующую строку:

```
auto audio_config = Audio::AudioConfig::FromDefaultMicrophoneInput();
```

на:

```
auto audio_config = Audio::AudioConfig::FromWavFileInput(path/to/your/file.wav);
```

Или замените `audio_config` на [Audio::AudioConfig::FromWavFileInput](/cpp/cognitive-services/speech/audio-audioconfig#fromwavfileinput). Можно также использовать смешанные входные данные, например микрофон для регистрации и файлы для проверки.