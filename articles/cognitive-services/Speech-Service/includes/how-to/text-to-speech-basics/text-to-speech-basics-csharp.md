---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/25/2020
ms.author: trbye
ms.custom: devx-track-csharp
ms.openlocfilehash: dca444e4590508f74585f4774c3bbda2ca0835c6
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/28/2021
ms.locfileid: "98947146"
---
Из этого краткого руководства вы узнаете, как работать с распространенными конструктивными шаблонами для синтеза текста в речь, используя пакет SDK службы "Речь". Вы начнете с основных настроек и синтеза, а затем перейдете к более сложным примерам для разработки пользовательских приложений, в том числе к таким задачам:

* получение ответов в виде потоков в памяти;
* настройка частоты выборки и скорости передачи выходных данных;
* отправка запросов синтеза с помощью SSML (язык разметки синтеза речи);
* использование нейронных голосовых моделей.

## <a name="skip-to-samples-on-github"></a>Примеры на GitHub

Если вы хотите сразу перейти к примерам кода, см. [этот репозиторий](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/csharp/dotnet/text-to-speech) на сайте GitHub.

## <a name="prerequisites"></a>Предварительные требования

В этой статье предполагается, что у вас есть учетная запись Azure и подписка на службу "Речь". Если у вас нет учетной записи и подписки, [попробуйте службу "Речь" бесплатно](../../../overview.md#try-the-speech-service-for-free).

## <a name="install-the-speech-sdk"></a>Установка пакета SDK службы "Речь"

Прежде чем выполнять какие-либо действия, необходимо установить пакет SDK для службы "Речь". В зависимости от используемой платформы выполните следующие действия:

* <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/setup-platform?tabs=dotnet&pivots=programming-language-csharp" target="_blank">.NET Framework <span class="docon docon-navigate-external x-hidden-focus"></span></a>
* <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/setup-platform?tabs=dotnetcore&pivots=programming-language-csharp" target="_blank">.NET Core <span class="docon docon-navigate-external x-hidden-focus"></span></a>
* <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/setup-platform?tabs=unity&pivots=programming-language-csharp" target="_blank">Unity <span class="docon docon-navigate-external x-hidden-focus"></span></a>
* <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/setup-platform?tabs=uwps&pivots=programming-language-csharp" target="_blank">UWP <span class="docon docon-navigate-external x-hidden-focus"></span></a>
* <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/setup-platform?tabs=xaml&pivots=programming-language-csharp" target="_blank">Xamarin <span class="docon docon-navigate-external x-hidden-focus"></span></a>

## <a name="import-dependencies"></a>Импорт зависимостей

Чтобы выполнить примеры из этой статьи, добавьте в начале скрипта следующие инструкции `using`:

```csharp
using System;
using System.IO;
using System.Text;
using System.Threading.Tasks;
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;
```

## <a name="create-a-speech-configuration"></a>Создание конфигурации службы "Речь"

Чтобы вызвать службу "Речь" с помощью пакета SDK для службы "Речь", необходимо создать [`SpeechConfig`](/dotnet/api/microsoft.cognitiveservices.speech.speechconfig). Этот класс содержит сведения о вашей подписке, такие как ключ и связанный регион, конечная точка, узел или маркер авторизации.

> [!NOTE]
> Независимо от того, используете ли вы распознавание речи, синтез речи, перевод или распознавание намерения, вы всегда создаете конфигурацию.

Существует несколько способов инициализации [`SpeechConfig`](/dotnet/api/microsoft.cognitiveservices.speech.speechconfig).

* С помощью подписки: передайте ключ и связанный с ним регион.
* С помощью конечной точки: передайте конечную точку службы "Речь". Ключ или маркер авторизации являются необязательными.
* С помощью узла: передайте адрес узла. Ключ или маркер авторизации являются необязательными.
* С помощью маркера авторизации: передайте маркер авторизации и связанный регион.

В этом примере создается [`SpeechConfig`](/dotnet/api/microsoft.cognitiveservices.speech.speechconfig) с использованием ключа и региона подписки. Чтобы получить эти учетные данные, выполните инструкции из раздела [Бесплатная пробная подписка на службу "Речь"](../../../overview.md#try-the-speech-service-for-free). Для оставшейся части этой статьи также создается стандартный код, который необходимо будет изменить, чтобы внести различные настройки.

```csharp
public class Program 
{
    static async Task Main()
    {
        await SynthesizeAudioAsync();
    }

    static async Task SynthesizeAudioAsync() 
    {
        var config = SpeechConfig.FromSubscription("YourSubscriptionKey", "YourServiceRegion");
    }
}
```

## <a name="synthesize-speech-to-a-file"></a>Синтезирование речи в файл

Далее необходимо создать объект [`SpeechSynthesizer`](/dotnet/api/microsoft.cognitiveservices.speech.speechsynthesizer), который выполняет преобразование текста в речь и выводит ее на динамики, в файлы или в другие потоки выходных данных. [`SpeechSynthesizer`](/dotnet/api/microsoft.cognitiveservices.speech.speechsynthesizer) принимает в качестве параметров объект [`SpeechConfig`](/dotnet/api/microsoft.cognitiveservices.speech.speechconfig), созданный на предыдущем шаге, и объект [`AudioConfig`](/dotnet/api/microsoft.cognitiveservices.speech.audio.audioconfig), определяющий, как должны обрабатываться выходные результаты.

Сначала создайте `AudioConfig` для автоматической записи выходных данных в файл `.wav` с помощью инструкции `FromWavFileOutput()`, а затем создайте экземпляр с использованием инструкции `using`. Инструкция `using` в этом контексте автоматически удаляет неуправляемые ресурсы и выводит объект из области после его удаления.

```csharp
static async Task SynthesizeAudioAsync() 
{
    var config = SpeechConfig.FromSubscription("YourSubscriptionKey", "YourServiceRegion");
    using var audioConfig = AudioConfig.FromWavFileOutput("path/to/write/file.wav");
}
```

Теперь создайте экземпляр `SpeechSynthesizer`, используя еще одну инструкцию `using`. Передайте объекты `config` и `audioConfig` в качестве параметров. После этого синтез речи и запись в файл выполняются так же просто, как запуск `SpeakTextAsync()` с использованием строки текста.

```csharp
static async Task SynthesizeAudioAsync() 
{
    var config = SpeechConfig.FromSubscription("YourSubscriptionKey", "YourServiceRegion");
    using var audioConfig = AudioConfig.FromWavFileOutput("path/to/write/file.wav");
    using var synthesizer = new SpeechSynthesizer(config, audioConfig);
    await synthesizer.SpeakTextAsync("A simple test to write to a file.");
}
```

Запустите программу, после чего синтезированный файл `.wav` будет записан в указанное вами расположение. Это хороший пример наиболее простого способа использования. Далее рассматривается настройка выходных данных и обработка ответа для вывода в виде потока в памяти для работы в пользовательских сценариях.

## <a name="synthesize-to-speaker-output"></a>Синтезирование выходных данных для вывода на динамики

В некоторых случаях может потребоваться напрямую выводить синтезированную речь на динамики. Для этого просто опустите параметр `AudioConfig` при создании `SpeechSynthesizer` в приведенном выше примере. Это позволит передавать выходные данные на текущее активное устройство вывода.

```csharp
static async Task SynthesizeAudioAsync() 
{
    var config = SpeechConfig.FromSubscription("YourSubscriptionKey", "YourServiceRegion");
    using var synthesizer = new SpeechSynthesizer(config);
    await synthesizer.SpeakTextAsync("Synthesizing directly to speaker output.");
}
```

## <a name="get-result-as-an-in-memory-stream"></a>Получение результата в виде потока в памяти

Во многих сценариях разработки речевых приложений вам, вероятно, понадобится получать звуковые данные в виде потока в памяти, а не записывать данные в файл. Это позволит выполнять различные действия с выходными данными, в частности:

* абстрагировать полученный массив байтов в виде потока с возможностью поиска для настраиваемых нижестоящих служб;
* интегрировать полученные данные с другими API или службами;
* изменять звуковые данные, записывать пользовательские заголовки `.wav` и т. д.

Это изменение легко реализовать в предыдущем примере. Сначала удалите блок `AudioConfig`, так как вы будете вручную управлять выходными данными с целью усиленного контроля над ними. Затем передайте `null` для `AudioConfig` в конструкторе `SpeechSynthesizer`. 

> [!NOTE]
> Передача значения `null` для `AudioConfig` вместо его пропуска, как это сделано в приведенном выше примере с выводом на динамики, не позволит по умолчанию воспроизводить звук на активном устройстве вывода.

В этом случае результат сохраняется в переменной [`SpeechSynthesisResult`](/dotnet/api/microsoft.cognitiveservices.speech.speechsynthesisresult). Свойство `AudioData` содержит `byte []` выходных данных. Вы можете работать с `byte []` вручную или использовать класс [`AudioDataStream`](/dotnet/api/microsoft.cognitiveservices.speech.audiodatastream) для управления потоком в памяти. В этом примере используется статическая функция `AudioDataStream.FromResult()` для получения потока из результата.

```csharp
static async Task SynthesizeAudioAsync() 
{
    var config = SpeechConfig.FromSubscription("YourSubscriptionKey", "YourServiceRegion");
    using var synthesizer = new SpeechSynthesizer(config, null);
    
    var result = await synthesizer.SpeakTextAsync("Getting the response as an in-memory stream.");
    using var stream = AudioDataStream.FromResult(result);
}
```

Теперь можно реализовать любое пользовательское поведение, используя полученный объект `stream`.

## <a name="customize-audio-format"></a>Настройка звукового формата

В этом разделе показано, как настроить атрибуты вывода звука, в том числе:

* тип звукового файла;
* частоту выборки;
* глубину в битах.

Чтобы изменить звуковой формат, используйте функцию `SetSpeechSynthesisOutputFormat()` для объекта `SpeechConfig`. Эта функция получает `enum` типа [`SpeechSynthesisOutputFormat`](/dotnet/api/microsoft.cognitiveservices.speech.speechsynthesisoutputformat) для выбора формата выходных данных. Список доступных звуковых форматов см. в [справочной документации](/dotnet/api/microsoft.cognitiveservices.speech.speechsynthesisoutputformat).

Различные типы файлов обеспечивают разные возможности для работы с ними в зависимости от требований. Обратите внимание, что по определению звуковые данные в необработанном формате, таком как `Raw24Khz16BitMonoPcm`, не содержат заголовки. Используйте такие форматы, только если вы уверены, что нижестоящие службы способны декодировать необработанный битовый поток, или если вы планируете создавать заголовки вручную на основе глубины в битах, частоты выборки, количества каналов и т. п.

> [!NOTE]
> Голоса **en-US-AriaRUS** и **en-US-GuyRUS** созданы из примеров с частотой дискретизации `Riff24Khz16BitMonoPcm`.

В этом примере мы укажем формат RIFF высокой точности воспроизведения `Riff24Khz16BitMonoPcm`, задав `SpeechSynthesisOutputFormat` для объекта `SpeechConfig`. Как и в примере из предыдущего раздела, мы будем использовать [`AudioDataStream`](/dotnet/api/microsoft.cognitiveservices.speech.audiodatastream) для получения результата в виде потока в памяти, а затем записывать его в файл.

```csharp
static async Task SynthesizeAudioAsync() 
{
    var config = SpeechConfig.FromSubscription("YourSubscriptionKey", "YourServiceRegion");
    config.SetSpeechSynthesisOutputFormat(SpeechSynthesisOutputFormat.Riff24Khz16BitMonoPcm);

    using var synthesizer = new SpeechSynthesizer(config, null);
    var result = await synthesizer.SpeakTextAsync("Customizing audio output format.");

    using var stream = AudioDataStream.FromResult(result);
    await stream.SaveToWaveFileAsync("path/to/write/file.wav");
}
```

При повторном запуске программы будет создан файл `.wav` по указанному пути.

## <a name="use-ssml-to-customize-speech-characteristics"></a>Настройка характеристик речи с использованием SSML

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

Далее необходимо изменить запрос синтеза речи, чтобы он ссылался на XML-файл. Запрос аналогичен использовавшемуся нами ранее, но вместо функции `SpeakTextAsync()` содержит `SpeakSsmlAsync()`. Эта функция ожидает строку XML, поэтому сначала нужно загрузить конфигурацию SSML в виде строки, используя `File.ReadAllText()` Теперь мы получим точно такой же объект, как и в предыдущих примерах.

> [!NOTE]
> Если вы используете Visual Studio, то конфигурация сборки вероятнее всего не найдет XML-файл по умолчанию. Чтобы устранить эту проблему, правой кнопкой мыши щелкните XML-файл и выберите пункт **Свойства**. Измените **Действие построения** на *Содержимое*, а **Копировать в выходной каталог** на *Всегда копировать*.

```csharp
public static async Task SynthesizeAudioAsync() 
{
    var config = SpeechConfig.FromSubscription("YourSubscriptionKey", "YourServiceRegion");
    using var synthesizer = new SpeechSynthesizer(config, null);
    
    var ssml = File.ReadAllText("./ssml.xml");
    var result = await synthesizer.SpeakSsmlAsync(ssml);

    using var stream = AudioDataStream.FromResult(result);
    await stream.SaveToWaveFileAsync("path/to/write/file.wav");
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
<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xmlns:mstts="https://www.w3.org/2001/mstts" xml:lang="en-US">
  <voice name="en-US-AriaNeural">
    <mstts:express-as style="cheerful">
      This is awesome!
    </mstts:express-as>
  </voice>
</speak>
```
