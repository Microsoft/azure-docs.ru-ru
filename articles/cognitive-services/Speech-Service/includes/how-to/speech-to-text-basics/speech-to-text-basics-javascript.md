---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/04/2021
ms.author: trbye
ms.custom: devx-track-js
ms.openlocfilehash: dd92cf24cf007418e52cb5091eb390b46d7a5571
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104988275"
---
Одной из основных функций службы "Речь" является распознавание и преобразование человеческой речи (часто это называется преобразованием речи в текст). Из этого краткого руководство вы узнаете, как использовать пакет SDK для службы "Речь" в приложениях и продуктах для выполнения высококачественного преобразования речи в текст.

## <a name="skip-to-samples-on-github"></a>Примеры на GitHub

Если вы хотите сразу перейти к примерам кода, см. [этот репозиторий](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/javascript/node) на сайте GitHub.

Сведения о том, как использовать пакет SDK службы "Речь" в среде на основе браузера, также см. на странице [примера React](https://github.com/Azure-Samples/AzureSpeechReactSample).

## <a name="prerequisites"></a>Предварительные требования

В этой статье предполагается, что у вас есть учетная запись Azure и подписка на службу "Речь". Если у вас нет учетной записи и подписки, [попробуйте службу "Речь" бесплатно](../../../overview.md#try-the-speech-service-for-free).

## <a name="install-the-speech-sdk"></a>Установка пакета SDK службы "Речь"

Прежде чем выполнять какие-либо действия, установите пакет SDK службы "Речь" для Node.js. Если вам только нужно имя пакета для установки, выполните команду `npm install microsoft-cognitiveservices-speech-sdk`. Пошаговые инструкции по установке см. в статье по [началу работы](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/setup-platform?tabs=dotnet%2Clinux%2Cjre%2Cnodejs&pivots=programming-language-javascript).

Воспользуйтесь следующим оператором `require`, чтобы импортировать пакет SDK.

```javascript
const sdk = require("microsoft-cognitiveservices-speech-sdk");
```

Дополнительные сведения об операторе `require` см. в [документации по require](https://nodejs.org/en/knowledge/getting-started/what-is-require/).

## <a name="create-a-speech-configuration"></a>Создание конфигурации службы "Речь"

Чтобы вызвать службу "Речь" с помощью пакета SDK для службы "Речь", необходимо создать [`SpeechConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig). Этот класс содержит сведения о вашей подписке, такие как ключ и связанный регион, конечная точка, узел или маркер авторизации. Создайте [`SpeechConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig) с помощью ключа и региона. Сведения о том, как найти пару "ключ-регион", см. в разделе [Поиск ключей и региона](../../../overview.md#find-keys-and-region).

```javascript
const speechConfig = sdk.SpeechConfig.fromSubscription("<paste-your-subscription-key>", "<paste-your-region>");
```

Существует несколько других способов инициализации [`SpeechConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig):

* С помощью конечной точки: передайте конечную точку службы "Речь". Ключ или маркер авторизации являются необязательными.
* С помощью узла: передайте адрес узла. Ключ или маркер авторизации являются необязательными.
* С помощью маркера авторизации: передайте маркер авторизации и связанный регион.

> [!NOTE]
> Независимо от того, используете ли вы распознавание речи, синтез речи, перевод или распознавание намерения, вы всегда создаете конфигурацию.

## <a name="recognize-from-microphone-browser-only"></a>Распознавание речи с микрофона (только с помощью браузера)

Распознавание речи с микрофона **не поддерживается в Node.js** и поддерживается только в среде JavaScript на основе браузера. [Реализацию преобразования речи с микрофона в текст](https://github.com/Azure-Samples/AzureSpeechReactSample/blob/main/src/App.js#L29) см. в [примере React](https://github.com/Azure-Samples/AzureSpeechReactSample) на сайте GitHub.

> [!NOTE]
> Если необходимо использовать *конкретное* входное аудиоустройство, необходимо указать код устройства в `AudioConfig`. Узнайте, [как получить код устройства](../../../how-to-select-audio-input-devices.md) для входного аудиоустройства.

## <a name="recognize-from-file"></a>Распознавание речи из файла 

Чтобы распознать речь из аудиофайла, создайте с помощью `fromWavFileInput()` объект `AudioConfig`, который принимает объект `Buffer`. Затем инициализируйте [`SpeechRecognizer`](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer?view=azure-node-latest), передав `audioConfig` и `speechConfig`.

```javascript
const fs = require('fs');
const sdk = require("microsoft-cognitiveservices-speech-sdk");
const speechConfig = sdk.SpeechConfig.fromSubscription("<paste-your-subscription-key>", "<paste-your-region>");

function fromFile() {
    let audioConfig = sdk.AudioConfig.fromWavFileInput(fs.readFileSync("YourAudioFile.wav"));
    let recognizer = new sdk.SpeechRecognizer(speechConfig, audioConfig);

    recognizer.recognizeOnceAsync(result => {
        console.log(`RECOGNIZED: Text=${result.text}`);
        recognizer.close();
    });
}
fromFile();
```

## <a name="recognize-from-in-memory-stream"></a>Распознавание из потока в памяти

Во многих вариантах использования аудиоданные, скорее всего, поступают из Хранилища BLOB-объектов или уже находятся в памяти в виде [`ArrayBuffer`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer) или аналогичной структуры необработанных данных. В приведенном ниже коде

* создается поток push-передачи с помощью `createPushStream()`;
* считывается файл `.wav` с помощью `fs.createReadStream` в демонстрационных целях, но если у вас уже есть аудиоданные в `ArrayBuffer`, вы можете сразу перейти к записи содержимого во входной поток;
* создается аудио-конфигурация с помощью потока push-передачи.

```javascript
const fs = require('fs');
const sdk = require("microsoft-cognitiveservices-speech-sdk");
const speechConfig = sdk.SpeechConfig.fromSubscription("<paste-your-subscription-key>", "<paste-your-region>");

function fromStream() {
    let pushStream = sdk.AudioInputStream.createPushStream();

    fs.createReadStream("YourAudioFile.wav").on('data', function(arrayBuffer) {
        pushStream.write(arrayBuffer.slice());
    }).on('end', function() {
        pushStream.close();
    });
 
    let audioConfig = sdk.AudioConfig.fromStreamInput(pushStream);
    let recognizer = new sdk.SpeechRecognizer(speechConfig, audioConfig);
    recognizer.recognizeOnceAsync(result => {
        console.log(`RECOGNIZED: Text=${result.text}`);
        recognizer.close();
    });
}
fromStream();
```

При использовании потока push-передачи в качестве входных данных предполагается, что звуковые данные являются необработанным PCM (пропуская все заголовки).
API будет по-прежнему работать в некоторых случаях, если заголовок не пропущен, но для получения наилучших результатов рекомендуется реализовать логику для чтения заголовков, чтобы `fs` начинался с *начала звуковых данных*.

## <a name="error-handling"></a>Обработка ошибок

В предыдущих примерах мы просто получили распознанный текст из `result.text`, но чтобы обработать ошибки и другие ответы, необходимо написать код для обработки результата. Следующий код вычисляет свойство [`result.reason`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognitionresult#reason) и:

* Выводит результат распознавания: `ResultReason.RecognizedSpeech`
* Если совпадений нет, сообщает об этом пользователю: `ResultReason.NoMatch`
* В случае ошибки выводит сообщение об ошибке: `ResultReason.Canceled`

```javascript
switch (result.reason) {
    case ResultReason.RecognizedSpeech:
        console.log(`RECOGNIZED: Text=${result.text}`);
        break;
    case ResultReason.NoMatch:
        console.log("NOMATCH: Speech could not be recognized.");
        break;
    case ResultReason.Canceled:
        const cancellation = CancellationDetails.fromResult(result);
        console.log(`CANCELED: Reason=${cancellation.reason}`);

        if (cancellation.reason == CancellationReason.Error) {
            console.log(`CANCELED: ErrorCode=${cancellation.ErrorCode}`);
            console.log(`CANCELED: ErrorDetails=${cancellation.errorDetails}`);
            console.log("CANCELED: Did you update the subscription info?");
        }
        break;
    }
```

## <a name="continuous-recognition"></a>Непрерывное распознавание

В предыдущих примерах используется одноразовое распознавание, которое распознает один речевой фрагмент. Конец одного речевого фрагмента определяется путем прослушивания до тишины в конце, или пока не будет обработано максимум 15 секунд аудио.

Непрерывное распознавание функционирует иначе. Оно используется, если требуется **контролировать**, когда следует остановить распознавание. Для получения результатов распознавания необходимо подписаться на события `Recognizing`, `Recognized`и `Canceled`. Чтобы отключить распознавание, необходимо вызвать [`stopContinuousRecognitionAsync`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer#stopcontinuousrecognitionasync). Ниже приведен пример того, как выполняется непрерывное распознавание входного аудиофайла.

Начните с определения входных данных и инициализации [`SpeechRecognizer`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer):

```javascript
const recognizer = new sdk.SpeechRecognizer(speechConfig, audioConfig);
```

Теперь подпишитесь на события, отправленные из [`SpeechRecognizer`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer).

* [`recognizing`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer#recognizing): Сигнал для событий, содержащих промежуточные результаты распознавания.
* [`recognized`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer#recognized): Сигнал для событий, содержащих окончательные результаты распознавания (указывает на успешность попытки распознавания).
* [`sessionStopped`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer#sessionstopped): Сигнал для событий, указывающих на окончание распознавания (операция).
* [`canceled`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer#canceled): Сигнал для событий, содержащих отмененные результаты распознавания (показывающий попытку распознавания, которая была отменена в результате ошибки распознавания или при прямом запросе об отмене, в случае сбоя транспортировки или протоколирования).

```javascript
recognizer.recognizing = (s, e) => {
    console.log(`RECOGNIZING: Text=${e.result.text}`);
};

recognizer.recognized = (s, e) => {
    if (e.result.reason == ResultReason.RecognizedSpeech) {
        console.log(`RECOGNIZED: Text=${e.result.text}`);
    }
    else if (e.result.reason == ResultReason.NoMatch) {
        console.log("NOMATCH: Speech could not be recognized.");
    }
};

recognizer.canceled = (s, e) => {
    console.log(`CANCELED: Reason=${e.reason}`);

    if (e.reason == CancellationReason.Error) {
        console.log(`"CANCELED: ErrorCode=${e.errorCode}`);
        console.log(`"CANCELED: ErrorDetails=${e.errorDetails}`);
        console.log("CANCELED: Did you update the subscription info?");
    }

    recognizer.stopContinuousRecognitionAsync();
};

recognizer.sessionStopped = (s, e) => {
    console.log("\n    Session stopped event.");
    recognizer.stopContinuousRecognitionAsync();
};
```

Завершив настройку, вызовите [`startContinuousRecognitionAsync`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechrecognizer#startcontinuousrecognitionasync), чтобы начать распознавание.

```javascript
recognizer.startContinuousRecognitionAsync();

// make the following call at some point to stop recognition.
// recognizer.StopContinuousRecognitionAsync();
```

### <a name="dictation-mode"></a>Режим диктовки

При использовании непрерывного распознавания вы можете включить обработку диктовки с помощью соответствующей функции "Включить диктовку". Этот режим приведет к тому, что экземпляр конфигурации речи будет интерпретировать словесные описания структуры предложения, например, пунктуацию. Например, речевой фрагмент "Живете ли вы в городе вопросительный знак" будет интерпретироваться как текст "Живете ли вы в городе?".

Чтобы включить режим диктовки, используйте метод [`enableDictation`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig#enabledictation--) в [`SpeechConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig).

```javascript
speechConfig.enableDictation();
```

## <a name="change-source-language"></a>Изменение исходного языка

Распространенной задачей распознавания речи является указание языка ввода (или исходного языка). Давайте посмотрим, как изменить язык ввода на итальянский. В коде найдите [`SpeechConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig), а затем добавьте следующую строку непосредственно под ней.

```javascript
speechConfig.speechRecognitionLanguage = "it-IT";
```

Свойство [`speechRecognitionLanguage`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig#speechrecognitionlanguage) принимает строку формата языкового стандарта. Вы можете указать любое значение в столбце **Языковой стандарт** в списке поддерживаемых [языковых стандартов/языков](../../../language-support.md).

## <a name="improve-recognition-accuracy"></a>Улучшение точности распознавания

Списки фраз используются для определения известных фраз в звуковых данных, например имени пользователя или определенного местоположения. Предоставив список фраз, вы улучшите точность распознавания речи.

Например, если у вас есть команда "Move to" и возможное произносимое назначение "Ward", вы можете добавить запись "Move to Ward". Добавление фразы увеличит вероятность того, что аудио будет распознано как "Move to Ward", а не "Move toward".

В список фраз можно добавить отдельные слова или полные фразы. В ходе распознавания запись в списке фраз используется для ускоренного распознавания слов и фраз в списке, даже если записи появляются в середине речевого фрагмента. 

> [!IMPORTANT]
> Функция списка фраз доступна для следующих языков: en-US, de-DE, en-AU, en-CA, en-GB, es-ES, es-MX, fr-CA, fr-FR, it-IT, ja-JP, ko-KR, pt-BR, zh-CN.

Чтобы использовать список фраз, сначала создайте объект [`PhraseListGrammar`](/javascript/api/microsoft-cognitiveservices-speech-sdk/phraselistgrammar), а затем добавьте определенные слова и фразы с помощью [`addPhrase`](/javascript/api/microsoft-cognitiveservices-speech-sdk/phraselistgrammar#addphrase-string-).

Любые изменения [`PhraseListGrammar`](/javascript/api/microsoft-cognitiveservices-speech-sdk/phraselistgrammar) вступают в силу при следующем распознавании или после повторного подключения к службе "Речь".

```javascript
const phraseList = sdk.PhraseListGrammar.fromRecognizer(recognizer);
phraseList.addPhrase("Supercalifragilisticexpialidocious");
```

Если необходимо очистить список фраз, выполните следующие действия.

```javascript
phraseList.clear();
```

### <a name="other-options-to-improve-recognition-accuracy"></a>Другие опции для повышения точности распознавания

Списки фраз — это только один вариант для повышения точности распознавания. Кроме того, вы можете сделать следующее: 

* [Повышение точности с помощью пользовательского распознавания речи](../../../custom-speech-overview.md)
* [Повышение точности с помощью моделей клиентов](../../../tutorial-tenant-model.md)
