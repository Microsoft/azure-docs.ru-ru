---
title: Как использовать транскрипции пакетной службы — речь
titleSuffix: Azure Cognitive Services
description: Пакетное транскрибирование идеально подходит, если вам нужно расшифровать большой объем аудиоматериала в хранилище, например в хранилище BLOB-объектов Azure. Используя выделенный REST API, вы можете указывать аудиофайлы с помощью URI подписанного URL-адреса (SAS) и асинхронно получать транскрипции.
services: cognitive-services
author: wolfma61
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 12/23/2020
ms.author: wolfma
ms.custom: devx-track-csharp
ms.openlocfilehash: e48fead4d4364fd84f178388dbfb9158296e687b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98659977"
---
# <a name="how-to-use-batch-transcription"></a>Использование записи пакетов

Запись пакетов — это набор REST API операций, позволяющих транскрипция большой объем аудио в хранилище. Вы можете указывать на звуковые файлы, используя стандартный универсальный код ресурса (URI) или URI [подписанного URL-доступа (SAS)](../../storage/common/storage-sas-overview.md) , и асинхронно получать результаты транскрипции. С помощью API версии 3.0 можно транскрипция один или несколько звуковых файлов или обработать весь контейнер хранилища.

Для вызова следующих методов можно использовать API-интерфейсы RESTFUL для записи пакетов.

|    Операция записи пакетов                                             |    Метод    |    Вызов REST API                                   |
|------------------------------------------------------------------------------|--------------|----------------------------------------------------|
|    Создает новую транскрипцию.                                              |    POST      |    спичтотекст/v 3.0/транскрипции            |
|    Извлекает список транскрипций для подписки, прошедшей проверку подлинности.    |    GET       |    спичтотекст/v 3.0/транскрипции            |
|    Возвращает список поддерживаемых языковых стандартов для автономных транскрипций.              |    GET       |    спичтотекст/v 3.0/транскрипции/языковые стандарты    |
|    Обновляет изменяемые сведения о транскрипции, определяемые его ИДЕНТИФИКАТОРом.    |    PATCH     |    спичтотекст/v 3.0/транскрипции/{ID}       |
|    Удаляет указанную задачу транскрипции.                                 |    DELETE    |    спичтотекст/v 3.0/транскрипции/{ID}       |
|    Возвращает транскрипцию, определяемую заданным ИДЕНТИФИКАТОРом.                        |    GET       |    спичтотекст/v 3.0/транскрипции/{ID}       |
|    Возвращает файлы результатов транскрипции, идентифицируемые по заданному ИДЕНТИФИКАТОРу.    |    GET       |    спичтотекст/v 3.0/транскрипции/{ID}/файлов |

Вы можете просматривать и тестировать подробный API, который доступен в виде [документа Swagger](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0).

Задания записи пакетов планируются в соответствии с оптимальными усилиями.
Вы не можете оценить, когда задание изменится в состояние выполнения, но оно должно происходить в течение нескольких минут после обычной загрузки системы. В состоянии выполнения транскрипция выполняется быстрее, чем скорость воспроизведения в среде выполнения аудио.

## <a name="prerequisites"></a>Предварительные условия

Как и для всех функций службы "Речь", вам необходимо создать ключ подписки на [портале Azure](https://portal.azure.com), следуя [руководству по началу работы](overview.md#try-the-speech-service-for-free).

>[!NOTE]
> Для использования записи пакетов требуется стандартная подписка (S0) для службы речи. Ключи бесплатной подписки (F0) не подходят. Дополнительные сведения см. в разделе [цены и ограничения](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).

Если вы планируете настроить модели, выполните действия, описанные в разделе [Настройка акустической настройки](./how-to-custom-speech-train-model.md) и [Настройка языка](./how-to-custom-speech-train-model.md). Для использования созданных моделей в записи пакетной службы требуется расположение их модели. Расположение модели можно получить при проверке сведений о модели ( `self` свойство). Развернутая Пользовательская конечная точка *не требуется* для службы транскрипции пакетов.

>[!NOTE]
> В составе REST API запись пакетной записи содержит набор [квот и ограничений](speech-services-quotas-and-limits.md#batch-transcription), которые мы рекомендуем изучить. Чтобы воспользоваться всеми преимуществами функции записи пакетов для эффективного транскрипция большого количества звуковых файлов, рекомендуется всегда отправлять несколько файлов на запрос или указывать контейнер хранилища BLOB-объектов с аудиофайлами для транскрипция. Служба будет транскрипция файлы одновременно, уменьшая время истечения времени. Использование нескольких файлов в одном запросе очень просто и очень просто. см. раздел [конфигурации](#configuration) . 

## <a name="batch-transcription-api"></a>API пакетного транскрибирования

API-интерфейс для транскрипции пакетов поддерживает следующие форматы:

| Формат | Кодек | Бит на выборку | Частота выборки             |
|--------|-------|---------|---------------------------------|
| WAV    | PCM   | 16-разрядный  | 8 кГц или 16 кГц, моно или стерео |
| MP3    | PCM   | 16-разрядный  | 8 кГц или 16 кГц, моно или стерео |
| OGG    | OPUS  | 16-разрядный  | 8 кГц или 16 кГц, моно или стерео |

Для звуковых потоков стереозвука левый и правый каналы разбиваются во время транскрипции. Файл результатов JSON создается для каждого канала.
Чтобы создать упорядоченную конечную запись, используйте метки времени, созданные для каждого utterance.

### <a name="configuration"></a>Конфигурация

Параметры конфигурации предоставляются в виде JSON. 

**Фотографировать один или несколько отдельных файлов.** При наличии нескольких файлов для транскрипция рекомендуется отправлять несколько файлов в одном запросе. В приведенном ниже примере используются три файла:

```json
{
  "contentUrls": [
    "<URL to an audio file 1 to transcribe>",
    "<URL to an audio file 2 to transcribe>",
    "<URL to an audio file 3 to transcribe>"
  ],
  "properties": {
    "wordLevelTimestampsEnabled": true
  },
  "locale": "en-US",
  "displayName": "Transcription of file using default model for en-US"
}
```

**Обработка всего контейнера хранилища.** [SAS](../../storage/common/storage-sas-overview.md) -контейнер должен содержать `r` разрешения (чтение) и `l` (List):

```json
{
  "contentContainerUrl": "<SAS URL to the Azure blob container to transcribe>",
  "properties": {
    "wordLevelTimestampsEnabled": true
  },
  "locale": "en-US",
  "displayName": "Transcription of container using default model for en-US"
}
```

**Используйте пользовательскую обученную модель в транскрипции пакета.** В примере используются три файла:

```json
{
  "contentUrls": [
    "<URL to an audio file 1 to transcribe>",
    "<URL to an audio file 2 to transcribe>",
    "<URL to an audio file 3 to transcribe>"
  ],
  "properties": {
    "wordLevelTimestampsEnabled": true
  },
  "locale": "en-US",
  "model": {
    "self": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/models/{id}"
  },
  "displayName": "Transcription of file using default model for en-US"
}
```


### <a name="configuration-properties"></a>Свойства конфигурации

Используйте эти необязательные свойства для настройки транскрипции:

:::row:::
   :::column span="1":::
      **Параметр**
   :::column-end:::
   :::column span="2":::
      **Описание**
:::row-end:::
:::row:::
   :::column span="1":::
      `profanityFilterMode`
   :::column-end:::
   :::column span="2":::
      Необязательный, по умолчанию — `Masked` . Указывает, как обрабатывать ненормативную лексику в результатах распознавания. Допустимые значения: `None` отключить фильтрацию ненормативных слов, заменить ненормативную лексику `Masked` звездочками, `Removed` удалить всю ненормативную лексику из результата или `Tags` Добавить теги "Ненормативная лексика".
:::row-end:::
:::row:::
   :::column span="1":::
      `punctuationMode`
   :::column-end:::
   :::column span="2":::
      Необязательный, по умолчанию — `DictatedAndAutomatic` . Указывает, как обрабатывать знаки препинания в результатах распознавания. Допустимые значения — `None` Отключить пунктуацию, `Dictated` чтобы указать явный (произнесенный) знак пунктуации, `Automatic` Разрешить декодеру использовать знаки препинания или `DictatedAndAutomatic` задиктовать и автоматически применять пунктуацию.
:::row-end:::
:::row:::
   :::column span="1":::
      `wordLevelTimestampsEnabled`
   :::column-end:::
   :::column span="2":::
      Необязательный `false` параметр по умолчанию. Указывает, следует ли добавлять к выводимым данным метки времени на уровне слова.
:::row-end:::
:::row:::
   :::column span="1":::
      `diarizationEnabled`
   :::column-end:::
   :::column span="2":::
      Необязательный `false` параметр по умолчанию. Указывает, что анализ диаризатион должен выполняться во входных данных, который должен быть каналом Mono, содержащим два голоса. Примечание. необходимо `wordLevelTimestampsEnabled` задать значение `true` .
:::row-end:::
:::row:::
   :::column span="1":::
      `channels`
   :::column-end:::
   :::column span="2":::
      Необязательный `0` `1` параметр и расшифрованной по умолчанию. Массив номеров каналов для обработки. Здесь можно указать подмножество доступных каналов в звуковом файле (например, `0` только).
:::row-end:::
:::row:::
   :::column span="1":::
      `timeToLive`
   :::column-end:::
   :::column span="2":::
      Необязательно, по умолчанию нет удаления. Длительность автоматического удаления записей после выполнения транскрипции. Параметр `timeToLive` полезен при обходе массовой обработки, чтобы убедиться, что они будут удаляться в конечном итоге (например, в `PT12H` течение 12 часов).
:::row-end:::
:::row:::
   :::column span="1":::
      `destinationContainerUrl`
   :::column-end:::
   :::column span="2":::
      Необязательный URL-адрес с [нерегламентированным SAS](../../storage/common/storage-sas-overview.md) в контейнер, доступный для записи в Azure. Результат сохраняется в этом контейнере. SAS с хранимой политикой доступа **не** поддерживается. Если этот параметр не указан, корпорация Майкрософт сохраняет результаты в контейнере хранилища, который управляется корпорацией Майкрософт. При удалении транскрипции путем вызова функции [удаления](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/DeleteTranscription)записей результирующие данные также будут удалены.
:::row-end:::

### <a name="storage"></a>Служба хранилища

Запись пакетов может читать звук из общедоступного URI Интернета, а также читать звук или записывать записи с помощью URI SAS с [хранилищем BLOB-объектов Azure](../../storage/blobs/storage-blobs-overview.md).

## <a name="batch-transcription-result"></a>Результат транскрипции пакета

Для каждого входного звука создается один файл результатов для транскрипции.
Операция [получения файлов транскрипций](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/GetTranscriptionFiles) возвращает список файлов результатов для этого транскрипции. Чтобы найти файл транскрипции для определенного входного файла, отфильтруйте все возвращенные файлы с помощью `kind`  ==  `Transcription` и `name`  ==  `{originalInputName.suffix}.json` .

Каждый файл результатов транскрипции имеет следующий формат:

```json
{
  "source": "...",                      // sas url of a given contentUrl or the path relative to the root of a given container
  "timestamp": "2020-06-16T09:30:21Z",  // creation time of the transcription, ISO 8601 encoded timestamp, combined date and time
  "durationInTicks": 41200000,          // total audio duration in ticks (1 tick is 100 nanoseconds)
  "duration": "PT4.12S",                // total audio duration, ISO 8601 encoded duration
  "combinedRecognizedPhrases": [        // concatenated results for simple access in single string for each channel
    {
      "channel": 0,                     // channel number of the concatenated results
      "lexical": "hello world",
      "itn": "hello world",
      "maskedITN": "hello world",
      "display": "Hello world."
    }
  ],
  "recognizedPhrases": [                // results for each phrase and each channel individually
    {
      "recognitionStatus": "Success",   // recognition state, e.g. "Success", "Failure"          
      "speaker": 1,                     // if `diarizationEnabled` is `true`, this is the identified speaker (1 or 2), otherwise this property is not present
      "channel": 0,                     // channel number of the result
      "offset": "PT0.07S",              // offset in audio of this phrase, ISO 8601 encoded duration 
      "duration": "PT1.59S",            // audio duration of this phrase, ISO 8601 encoded duration
      "offsetInTicks": 700000.0,        // offset in audio of this phrase in ticks (1 tick is 100 nanoseconds)
      "durationInTicks": 15900000.0,    // audio duration of this phrase in ticks (1 tick is 100 nanoseconds)

      // possible transcriptions of the current phrase with confidences
      "nBest": [
        {
          "confidence": 0.898652852,    // confidence value for the recognition of the whole phrase
          "lexical": "hello world",
          "itn": "hello world",
          "maskedITN": "hello world",
          "display": "Hello world.",

          // if wordLevelTimestampsEnabled is `true`, there will be a result for each word of the phrase, otherwise this property is not present
          "words": [
            {
              "word": "hello",
              "offset": "PT0.09S",
              "duration": "PT0.48S",
              "offsetInTicks": 900000.0,
              "durationInTicks": 4800000.0,
              "confidence": 0.987572
            },
            {
              "word": "world",
              "offset": "PT0.59S",
              "duration": "PT0.16S",
              "offsetInTicks": 5900000.0,
              "durationInTicks": 1600000.0,
              "confidence": 0.906032
            }
          ]
        }
      ]
    }
  ]
}
```

Результат содержит следующие поля:

:::row:::
   :::column span="1":::
      **Поле**
   :::column-end:::
   :::column span="2":::
      **Содержимое**
:::row-end:::
:::row:::
   :::column span="1":::
      `lexical`
   :::column-end:::
   :::column span="2":::
      Фактические слова распознаны.
:::row-end:::
:::row:::
   :::column span="1":::
      `itn`
   :::column-end:::
   :::column span="2":::
      Обратная текстовая Нормализованная форма распознанного текста. Аббревиатуры ("врач Смит" в "Dr Smith"), Номера телефонов и другие преобразования применяются.
:::row-end:::
:::row:::
   :::column span="1":::
      `maskedITN`
   :::column-end:::
   :::column span="2":::
      Форма восстановление с применением маскировки ненормативной лексики.
:::row-end:::
:::row:::
   :::column span="1":::
      `display`
   :::column-end:::
   :::column span="2":::
      Отображаемая форма распознанного текста. Добавлены знаки препинания и прописные буквы.
:::row-end:::

## <a name="speaker-separation-diarization"></a>Разделение докладчика (диаризатион)

Диаризатион — это процесс отделения динамиков от звука. Конвейер пакетной обработки поддерживает диаризатион и может распознать два динамика в записях каналов Mono. Эта функция недоступна в стерео записях.

Выходные данные транскрипции с включенным диаризатион содержат `Speaker` запись для каждой расшифрованной фразы. Если диаризатион не используется, свойство отсутствует `Speaker` в выходных данных JSON. Для диаризатион мы поддерживаем два голоса, поэтому динамики обозначены как `1` или `2` .

Чтобы запросить диаризатион, добавьте `diarizationEnabled` свойство в значение `true` Like HTTP-запрос, показанный ниже.

 ```json
{
  "contentUrls": [
    "<URL to an audio file to transcribe>",
  ],
  "properties": {
    "diarizationEnabled": true,
    "wordLevelTimestampsEnabled": true,
    "punctuationMode": "DictatedAndAutomatic",
    "profanityFilterMode": "Masked"
  },
  "locale": "en-US",
  "displayName": "Transcription of file using default model for en-US"
}
```

Метки времени на уровне слов должны быть включены, так как параметры в приведенном выше запросе указывают.

## <a name="best-practices"></a>Рекомендации

Служба транскрипции пакетов может работать с большим количеством отправленных транскрипций. Вы можете запросить состояние транскрипций с помощью [получения](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/GetTranscriptions)сведений о них.
Регулярно вызывайте функцию [транскрипции удаления](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/DeleteTranscription) из службы после получения результатов. Кроме того `timeToLive` , задайте свойство, чтобы обеспечить окончательное удаление результатов.

## <a name="sample-code"></a>Пример кода

Полные примеры доступны в [репозитории примера GitHub](https://aka.ms/csspeech/samples) внутри `samples/batch` подкаталога.

Обновите пример кода, указав сведения о подписке, регион службы, URI, указывающий на звуковой файл, в транскрипция и расположение модели, если вы используете пользовательскую модель.

[!code-csharp[Configuration variables for batch transcription](~/samples-cognitive-services-speech-sdk/samples/batch/csharp/program.cs#transcriptiondefinition)]

В примере кода настраивается клиент и отправляется запрос на расшифровку. Затем он опрашивает сведения о состоянии и выводит сведения о ходе процесса транскрипции.

```csharp
// get the status of our transcriptions periodically and log results
int completed = 0, running = 0, notStarted = 0;
while (completed < 1)
{
    completed = 0; running = 0; notStarted = 0;

    // get all transcriptions for the user
    paginatedTranscriptions = null;
    do
    {
        // <transcriptionstatus>
        if (paginatedTranscriptions == null)
        {
            paginatedTranscriptions = await client.GetTranscriptionsAsync().ConfigureAwait(false);
        }
        else
        {
            paginatedTranscriptions = await client.GetTranscriptionsAsync(paginatedTranscriptions.NextLink).ConfigureAwait(false);
        }

        // delete all pre-existing completed transcriptions. If transcriptions are still running or not started, they will not be deleted
        foreach (var transcription in paginatedTranscriptions.Values)
        {
            switch (transcription.Status)
            {
                case "Failed":
                case "Succeeded":
                    // we check to see if it was one of the transcriptions we created from this client.
                    if (!createdTranscriptions.Contains(transcription.Self))
                    {
                        // not created form here, continue
                        continue;
                    }

                    completed++;

                    // if the transcription was successful, check the results
                    if (transcription.Status == "Succeeded")
                    {
                        var paginatedfiles = await client.GetTranscriptionFilesAsync(transcription.Links.Files).ConfigureAwait(false);

                        var resultFile = paginatedfiles.Values.FirstOrDefault(f => f.Kind == ArtifactKind.Transcription);
                        var result = await client.GetTranscriptionResultAsync(new Uri(resultFile.Links.ContentUrl)).ConfigureAwait(false);
                        Console.WriteLine("Transcription succeeded. Results: ");
                        Console.WriteLine(JsonConvert.SerializeObject(result, SpeechJsonContractResolver.WriterSettings));
                    }
                    else
                    {
                        Console.WriteLine("Transcription failed. Status: {0}", transcription.Properties.Error.Message);
                    }

                    break;

                case "Running":
                    running++;
                    break;

                case "NotStarted":
                    notStarted++;
                    break;
            }
        }

        // for each transcription in the list we check the status
        Console.WriteLine(string.Format("Transcriptions status: {0} completed, {1} running, {2} not started yet", completed, running, notStarted));
    }
    while (paginatedTranscriptions.NextLink != null);

    // </transcriptionstatus>
    // check again after 1 minute
    await Task.Delay(TimeSpan.FromMinutes(1)).ConfigureAwait(false);
}
```

Подробные сведения о предыдущих вызовах см. в [документации по Swagger](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0). Полная версия показанного здесь примера находится на [GitHub](https://aka.ms/csspeech/samples) в подкаталоге `samples/batch`.

В этом примере используется асинхронная Настройка для отправки аудио и получения сведений о состоянии транскрипции.
`PostTranscriptions`Метод отправляет сведения о звуковом файле, и `GetTranscriptions` метод получает состояния.
`PostTranscriptions` возвращает дескриптор, который `GetTranscriptions` использует для создания дескриптора, чтобы получить состояние транскрибирования.

В этом примере кода не указана пользовательская модель. Служба использует базовую модель для фотографировать файлов или файлов. Чтобы указать модель, можно передать в тот же метод ссылку на модель для настраиваемой модели.

> [!NOTE]
> Для транскрипций базовых показателей не нужно объявлять идентификатор для базовой модели.

## <a name="next-steps"></a>Дальнейшие действия

- [Справочник по API для преобразования речи в текст v3](https://centralus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/CopyModelToSubscription)
