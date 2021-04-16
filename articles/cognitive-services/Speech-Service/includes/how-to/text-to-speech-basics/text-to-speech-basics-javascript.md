---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 02/10/2021
ms.author: trbye
ms.custom: devx-track-js
ms.openlocfilehash: 95bc737f8a1b9b0a35b80ca2a80a7245ba407b18
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105104333"
---
Из этого краткого руководства вы узнаете, как работать с распространенными конструктивными шаблонами для синтеза текста в речь, используя пакет SDK службы "Речь". Вы начнете с основных настроек и синтеза, а затем перейдете к более сложным примерам для разработки пользовательских приложений, в том числе к таким задачам:

* получение ответов в виде потоков в памяти;
* настройка частоты выборки и скорости передачи выходных данных;
* отправка запросов синтеза с помощью SSML (язык разметки синтеза речи);
* использование нейронных голосовых моделей.

## <a name="skip-to-samples-on-github"></a>Примеры на GitHub

Если вы хотите сразу перейти к примерам кода, см. [этот репозиторий](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/javascript/node/text-to-speech) на сайте GitHub.

## <a name="prerequisites"></a>Предварительные требования

В этой статье предполагается, что у вас есть учетная запись Azure и ресурс службы "Речь". Если у вас нет учетной записи и ресурса, [попробуйте службу "Речь" бесплатно](../../../overview.md#try-the-speech-service-for-free).

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
import * as sdk from "microsoft-cognitiveservices-speech-sdk";
```

Дополнительные сведения о директиве `import` см. в статье об <a href="https://javascript.info/import-export" target="_blank">экспорте и импорте </a>.

# <a name="require"></a>[require](#tab/require)

```javascript
const sdk = require("microsoft-cognitiveservices-speech-sdk");
```

Дополнительные сведения о функции `require` см. <a href="https://nodejs.org/en/knowledge/getting-started/what-is-require/" target="_blank">здесь </a>.

---


## <a name="create-a-speech-configuration"></a>Создание конфигурации службы "Речь"

Чтобы вызвать службу "Речь" с помощью пакета SDK для службы "Речь", необходимо создать [`SpeechConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig). Этот класс содержит сведения о вашем ресурсе, такие как ключ и связанный регион, конечная точка, узел или маркер авторизации.

> [!NOTE]
> Независимо от того, используете ли вы распознавание речи, синтез речи, перевод или распознавание намерения, вы всегда создаете конфигурацию.

Существует несколько способов инициализации [`SpeechConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig).

* С помощью ресурса: передайте ключ и связанный с ним регион.
* С помощью конечной точки: передайте конечную точку службы "Речь". Ключ или маркер авторизации являются необязательными.
* С помощью узла: передайте адрес узла. Ключ или маркер авторизации являются необязательными.
* С помощью маркера авторизации: передайте маркер авторизации и связанный регион.

В этом примере создается [`SpeechConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig) с использованием ключа и региона ресурса. Чтобы получить эти учетные данные, выполните инструкции из раздела [Бесплатная пробная подписка на службу "Речь"](../../../overview.md#try-the-speech-service-for-free). Для оставшейся части этой статьи также создается стандартный код, который необходимо будет изменить, чтобы внести различные настройки.

```javascript
function synthesizeSpeech() {
    const speechConfig = sdk.SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
}

synthesizeSpeech();
```

## <a name="synthesize-speech-to-a-file"></a>Синтезирование речи в файл

Далее необходимо создать объект [`SpeechSynthesizer`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechsynthesizer), который выполняет преобразование текста в речь и выводит ее на динамики, в файлы или в другие потоки выходных данных. В качестве параметров объект [`SpeechSynthesizer`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechsynthesizer) принимает созданный на предыдущем шаге объект [`SpeechConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig) и объект [`AudioConfig`](/javascript/api/microsoft-cognitiveservices-speech-sdk/audioconfig), определяющий, как должны обрабатываться выходные результаты.

Сначала создайте `AudioConfig` для автоматической записи выходных данных в файл `.wav` с помощью статической функции `fromAudioFileOutput()`.

```javascript
function synthesizeSpeech() {
    const speechConfig = sdk.SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
    const audioConfig = AudioConfig.fromAudioFileOutput("path/to/file.wav");
}
```

Затем создайте экземпляр `SpeechSynthesizer`, передающий объект `speechConfig` и объект `audioConfig` в качестве параметров. После этого синтез речи и запись в файл выполняются так же просто, как запуск `speakTextAsync()` с использованием строки текста. Обратный вызов результата — оптимальное место для вызова `synthesizer.close()`. В действительности этот вызов необходим для правильной работы синтеза.

```javascript
function synthesizeSpeech() {
    const speechConfig = sdk.SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
    const audioConfig = AudioConfig.fromAudioFileOutput("path-to-file.wav");

    const synthesizer = new SpeechSynthesizer(speechConfig, audioConfig);
    synthesizer.speakTextAsync(
        "A simple test to write to a file.",
        result => {
            synthesizer.close();
            if (result) {
                // return result as stream
                return fs.createReadStream("path-to-file.wav");
            }
        },
        error => {
            console.log(error);
            synthesizer.close();
        });
}
```

Запустите программу, после чего синтезированный файл `.wav` будет записан в указанное вами расположение. Это хороший пример наиболее простого способа использования. Далее рассматривается настройка выходных данных и обработка ответа для вывода в виде потока в памяти для работы в пользовательских сценариях.

## <a name="synthesize-to-speaker-output"></a>Синтезирование выходных данных для вывода на динамики

В некоторых случаях может потребоваться напрямую выводить синтезированную речь на динамики. Для этого создайте экземпляр `AudioConfig` с помощью статической функции `fromDefaultSpeakerOutput()`. Это позволит передавать выходные данные на активное устройство вывода.

```javascript
function synthesizeSpeech() {
    const speechConfig = sdk.SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
    const audioConfig = AudioConfig.fromDefaultSpeakerOutput();

    const synthesizer = new SpeechSynthesizer(speechConfig, audioConfig);
    synthesizer.speakTextAsync(
        "Synthesizing directly to speaker output.",
        result => {
            if (result) {
                synthesizer.close();
                return result.audioData;
            }
        },
        error => {
            console.log(error);
            synthesizer.close();
        });
}
```

## <a name="get-result-as-an-in-memory-stream"></a>Получение результата в виде потока в памяти

Во многих сценариях разработки речевых приложений вам, вероятно, понадобится получать звуковые данные в виде потока в памяти, а не записывать данные в файл. Это позволит выполнять различные действия с выходными данными, в частности:

* абстрагировать полученный массив байтов в виде потока с возможностью поиска для настраиваемых нижестоящих служб;
* интегрировать полученные данные с другими API или службами;
* изменять звуковые данные, записывать пользовательские заголовки `.wav` и т. д.

Это изменение легко реализовать в предыдущем примере. Сначала удалите блок `AudioConfig`, так как вы будете вручную управлять выходными данными с целью усиленного контроля над ними. Затем передайте `undefined` для `AudioConfig` в конструкторе `SpeechSynthesizer`.

> [!NOTE]
> Передача значения `undefined` для `AudioConfig` вместо его пропуска, как это сделано в приведенном выше примере с выводом на динамики, не позволит по умолчанию воспроизводить звук на активном устройстве вывода.

В этом случае результат сохраняется в переменной [`SpeechSynthesisResult`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechsynthesisresult). Свойство `SpeechSynthesisResult.audioData` возвращает `ArrayBuffer` выходных данных, тип потока браузера по умолчанию. Для серверного кода преобразуйте arrayBuffer в поток буфера.

Приведенный ниже код работает с кодом на стороне клиента.

```javascript
function synthesizeSpeech() {
    const speechConfig = sdk.SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
    const synthesizer = new sdk.SpeechSynthesizer(speechConfig);

    synthesizer.speakTextAsync(
        "Getting the response as an in-memory stream.",
        result => {
            synthesizer.close();
            return result.audioData;
        },
        error => {
            console.log(error);
            synthesizer.close();
        });
}
```

Теперь можно настроить выполнение любых пользовательских действий, используя полученный объект `ArrayBuffer`. ArrayBuffer — это распространенный тип для получения в браузере и воспроизведения из этого формата.

Для любого серверного кода, если необходимо работать с данными в виде потока, вместо ArrayBuffer необходимо преобразовать в поток объект.

```javascript
function synthesizeSpeech() {
    const speechConfig = sdk.SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
    const synthesizer = new sdk.SpeechSynthesizer(speechConfig);

    synthesizer.speakTextAsync(
        "Getting the response as an in-memory stream.",
        result => {
            const { audioData } = result;

            synthesizer.close();

            // convert arrayBuffer to stream
            // return stream
            const bufferStream = new PassThrough();
            bufferStream.end(Buffer.from(audioData));
            return bufferStream;
        },
        error => {
            console.log(error);
            synthesizer.close();
        });
}
```

## <a name="customize-audio-format"></a>Настройка звукового формата

В этом разделе показано, как настроить атрибуты вывода звука, в том числе:

* тип звукового файла;
* частоту выборки;
* глубину в битах.

Чтобы изменить звуковой формат, используйте свойство `speechSynthesisOutputFormat` для объекта `SpeechConfig`. Оно получает `enum` типа [`SpeechSynthesisOutputFormat`](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechsynthesisoutputformat), который используется для выбора формата выходных данных. Список доступных звуковых форматов см. в [справочной документации](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechsynthesisoutputformat).

Различные типы файлов обеспечивают разные возможности для работы с ними в зависимости от требований. Обратите внимание, что по определению звуковые данные в необработанном формате, таком как `Raw24Khz16BitMonoPcm`, не содержат заголовки. Используйте такие форматы, только если вы уверены, что нижестоящие службы способны декодировать необработанный битовый поток, или если вы планируете создавать заголовки вручную на основе глубины в битах, частоты выборки, количества каналов и т. п.

> [!NOTE]
> Голоса **en-US-AriaRUS** и **en-US-GuyRUS** созданы из примеров с частотой дискретизации `Riff24Khz16BitMonoPcm`.

В этом примере мы укажем формат RIFF высокой точности воспроизведения `Riff24Khz16BitMonoPcm`, задав `speechSynthesisOutputFormat` для объекта `SpeechConfig`. Как и в примере из предыдущего раздела, можно получить данные `ArrayBuffer` для звука и взаимодействовать с ними.

```javascript
function synthesizeSpeech() {
    const speechConfig = SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");

    // Set the output format
    speechConfig.speechSynthesisOutputFormat = SpeechSynthesisOutputFormat.Riff24Khz16BitMonoPcm;

    const synthesizer = new sdk.SpeechSynthesizer(speechConfig, undefined);
    synthesizer.speakTextAsync(
        "Customizing audio output format.",
        result => {
            // Interact with the audio ArrayBuffer data
            const audioData = result.audioData;
            console.log(`Audio data byte size: ${audioData.byteLength}.`)

            synthesizer.close();
        },
        error => {
            console.log(error);
            synthesizer.close();
        });
}
```

При повторном запуске программы будет создан файл `.wav` по указанному пути.

## <a name="use-ssml-to-customize-speech-characteristics"></a>Настройка характеристик речи с помощью SSML

Speech Synthesis Markup Language (язык разметки синтеза речи, SSML) позволяет точно настраивать тон, произношение, скорость речи, громкость и другие параметры выходных данных для преобразования текста в речь, отправляя запросы из схемы XML. В этом разделе показано несколько примеров использования этого языка, но, чтобы получить более полное представление о нем, прочтите [эту статью](../../../speech-synthesis-markup.md).

Чтобы освоить настройку с использованием SSML, внесем простое изменение, которое позволяет менять голос.
Сначала создайте XML-файл для конфигурации SSML в корневом каталоге проекта. В нашем примере это `ssml.xml`. Корневым элементом всегда является `<speak>`, а перенос текста в элемент `<voice>` позволяет менять голос с помощью параметра `name`. В этом примере мы изменим голос, чтобы использовать речь мужчины на (британском) английском. Обратите внимание, что этот голос является **стандартным**, поэтому цена и доступность его использования отличаются от аналогичных параметров голосов, **синтезированных с помощью нейронных сетей**. Ознакомьтесь с [полным списком](../../../language-support.md#standard-voices) поддерживаемых **стандартных** голосов.

```xml
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
  <voice name="en-GB-George-Apollo">
    When you're on the motorway, it's a good idea to use a sat-nav.
  </voice>
</speak>
```

Далее необходимо изменить запрос синтеза речи, чтобы он ссылался на XML-файл. Запрос аналогичен использовавшемуся нами ранее, но вместо функции `speakTextAsync()` содержит `speakSsmlAsync()`. Эта функция принимает строку XML, поэтому сначала создайте функцию для загрузки XML-файла и возвращения его в виде строки.

```javascript
function xmlToString(filePath) {
    const xml = readFileSync(filePath, "utf8");
    return xml;
}
```

Дополнительные сведения о `readFileSync` см. <a href="https://nodejs.org/api/fs.html#fs_fs_readlinksync_path_options" target="_blank">здесь</a>. Теперь мы получим точно такой же объект, как и в предыдущих примерах.

```javascript
function synthesizeSpeech() {
    const speechConfig = sdk.SpeechConfig.fromSubscription("YourSubscriptionKey", "YourServiceRegion");
    const synthesizer = new sdk.SpeechSynthesizer(speechConfig, undefined);

    const ssml = xmlToString("ssml.xml");
    synthesizer.speakSsmlAsync(
        ssml,
        result => {
            if (result.errorDetails) {
                console.error(result.errorDetails);
            } else {
                console.log(JSON.stringify(result));
            }

            synthesizer.close();
        },
        error => {
            console.log(error);
            synthesizer.close();
        });
}
```

Выходные данные поступают, но можно внести несколько простых изменений, чтобы добиться более естественного звучания. Общая скорость речи высокая, поэтому мы добавим тег `<prosody>`, чтобы уменьшить ее до **90 %** от заданной по умолчанию. Кроме того, пауза после запятой в предложении коротковата и неестественна. Чтобы исправить это, добавьте тег `<break>`, что замедлит речь, а параметру времени задайте значение **200 мс**. Повторно запустите синтез, чтобы услышать, как эти настройки повлияли на выходные данные.

```xml
<speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
  <voice name="en-GB-George-Apollo">
    <prosody rate="0.9">
      When you're on the motorway,<break time="200ms"/> it's a good idea to use a sat-nav.
    </prosody>
  </voice>
</speak>
```

## <a name="neural-voices"></a>Синтезирование голоса с помощью нейронных сетей

Синтезированные с помощью нейронных сетей голоса — это модели, созданные с использованием алгоритмов синтеза речи на основе глубоких нейронных сетей. Во время использования нейронных голосовых моделей синтезированная речь почти ничем не отличается от записей человеческого голоса. Эти модели отличаются человекоподобной естественной интонацией и четким произношение слов, что значительно уменьшает усталость от прослушивания при взаимодействии с системами искусственного интеллекта.

Чтобы переключиться на нейронную голосовую модель, замените `name` одним из [доступных вариантов](../../../language-support.md#neural-voices). Затем добавьте пространство имен XML для `mstts` и заключите текст в тег `<mstts:express-as>`. Используйте параметр `style`, чтобы настроить стиль речи. В этом примере используется `cheerful`, но попробуйте заменить его на `customerservice` или `chat`, чтобы оценить различия стиля речи.

> [!IMPORTANT]
> Голоса, синтезированные с помощью нейронных сетей, поддерживают **только** речевые ресурсы, созданные в регионах *Восточная часть США*, *Юго-Восточная Азия* и *Западная Европа*.

```xml
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis"
    xmlns:mstts="https://www.w3.org/2001/mstts" xml:lang="en-US">
  <voice name="en-US-AriaNeural">
    <mstts:express-as style="cheerful">
      This is awesome!
    </mstts:express-as>
  </voice>
</speak>
```

## <a name="get-facial-pose-events"></a>Получение событий расположения лица

Речь может эффективно анимировать эмоциональные выражения лица.
Для представления основных положений в наблюдаемой речи часто используются [виземы](../../../how-to-speech-synthesis-viseme.md), такие как расположение губ, челюстей и языка при создании определенной фонемы.
На событие виземы можно подписаться в пакете SDK для службы "Речь".
Затем события виземы можно применять для анимации лица характера при акустическом воспроизведении речи.
Дополнительные сведения о [получении событий визем](../../../how-to-speech-synthesis-viseme.md#get-viseme-events-with-the-speech-sdk).
