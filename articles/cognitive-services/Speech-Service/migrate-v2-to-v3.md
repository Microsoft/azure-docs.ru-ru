---
title: Миграция с версии v2 на v3 REST API-Speech Service
titleSuffix: Azure Cognitive Services
description: Этот документ помогает разработчикам перенести код с v2 на v3 в речевых REST API речевых служб.
services: cognitive-services
author: bexxx
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 02/12/2020
ms.author: rbeckers
ms.custom: devx-track-csharp
ms.openlocfilehash: 9c8016b566db8be1b7f5c5ddb8d92123d6673db5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98569850"
---
# <a name="migrate-code-from-v20-to-v30-of-the-rest-api"></a>Перенос кода с версии 2.0 на версию 3.0 REST API

По сравнению с v2 версия версии 3 речевых служб REST API для преобразования речи в текст более надежна, проще в использовании и более согласуется с интерфейсами API для схожих служб. Большинство команд могут переходить с версии v2 на v3 в один день или две.

## <a name="forward-compatibility"></a>Прямая совместимость

Все сущности из v2 также можно найти в API V3 с тем же идентификатором. Где изменилась схема результата (например, транскрипции), результат GET в версии v3 API использует схему v3. Результат получения в версии 2 API использует ту же схему v2. Вновь созданные сущности на **v3 недоступны**   в ответах от API v2. 

## <a name="migration-steps"></a>Этапы миграции

Это сводный список элементов, которые необходимо учитывать при подготовке к миграции. Сведения содержатся в отдельных ссылках. В зависимости от текущего использования API не все описанные здесь действия могут быть применены. В вызывающем коде требуется внести нетривиальные изменения. Для большинства изменений требуется только изменение имен элементов. 

Общие изменения: 

1. [Изменение имени узла](#host-name-changes)

1. [Переименование идентификатора свойства в собственный код клиента](#identity-of-an-entity) 

1. [Изменение кода для итерации коллекций сущностей](#working-with-collections-of-entities)

1. [Переименуйте имя свойства в displayName в клиентском коде.](#name-of-an-entity)

1. [Настройка извлечения метаданных упоминаемых сущностей](#accessing-referenced-entities)

1. При использовании записи пакетов: 

    * [Настройка кода для создания транскрипций пакетной службы](#creating-transcriptions) 

    * [Адаптация кода к новой схеме результатов транскрипции](#format-of-v3-transcription-results)

    * [Настройка кода для получения результатов](#getting-the-content-of-entities-and-the-results)

1. Если вы используете пользовательские API для обучения и тестирования модели: 

    * [Применение изменений для обучения пользовательской модели](#customizing-models)

    * [Изменение порядка извлечения базовых и пользовательских моделей](#retrieving-base-and-custom-models)

    * [Переименование сегмента пути аккурацитестс в оценку в коде клиента](#accuracy-tests)

1. При использовании API конечных точек:

    * [Изменение порядка извлечения журналов конечных точек](#retrieving-endpoint-logs)

1. Другие незначительные изменения: 

    * [Передайте все пользовательские свойства как customProperties вместо свойств в запросах POST.](#using-custom-properties)

    * [Чтение расположения из расположения заголовка ответа вместо "Operation-Location"](#response-headers)

## <a name="breaking-changes"></a>Критические изменения

### <a name="host-name-changes"></a>Изменения имени узла

Имена узлов конечных точек изменились с `{region}.cris.ai` на `{region}.api.cognitive.microsoft.com` . Пути к новым конечным точкам больше не содержат `api/` , так как они являются частью имени узла. В [документе Swagger](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0) перечисляются допустимые регионы и пути.
>[!IMPORTANT]
>Измените имя узла с `{region}.cris.ai` на `{region}.api.cognitive.microsoft.com` регион, где находится регион вашей подписки на речь. Также удалите `api/` из любого пути в коде клиента.

### <a name="identity-of-an-entity"></a>Удостоверение сущности

Свойство `id` теперь имеет значение `self` . В версии 2 пользователь API должен был понять, как создаются наши пути в API. Это было нерасширяемым и не требует от пользователя ненужной работы. Свойство `id` (UUID) заменяется на `self` (String), которое является расположением сущности (URL-адреса). Значение по-прежнему уникально для всех сущностей. Если `id` хранится в коде в виде строки, то для поддержки новой схемы достаточно переименования. Теперь содержимое можно использовать в `self` качестве URL-адреса для `GET` `PATCH` вызовов функций, и `DELETE` RESTful для сущности.

Если сущность имеет дополнительные функциональные возможности, доступные через другие пути, они перечислены в разделе `links` . В следующем примере для транскрипции показан отдельный метод для `GET` содержимого транскрипции:
>[!IMPORTANT]
>Переименуйте свойство `id` в `self` в коде клиента. При необходимости измените тип с `uuid` на `string` . 

**Транскрипция v2:**

```json
{
    "id": "9891c965-bb32-4880-b14b-6d44efb158f3",
    "createdDateTime": "2019-01-07T11:34:12Z",
    "lastActionDateTime": "2019-01-07T11:36:07Z",
    "status": "Succeeded",
    "locale": "en-US",
    "name": "Transcription using locale en-US"
}
```

**v3 запись транскрипции:**

```json
{
    "self": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/transcriptions/9891c965-bb32-4880-b14b-6d44efb158f3",
    "createdDateTime": "2019-01-07T11:34:12Z",
    "lastActionDateTime": "2019-01-07T11:36:07Z",
    "status": "Succeeded",
    "locale": "en-US", 
    "displayName": "Transcription using locale en-US",
    "links": {
      "files": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/transcriptions/9891c965-bb32-4880-b14b-6d44efb158f3/files"
    }
}
```

В зависимости от реализации кода может быть недостаточно для переименования свойства. Мы рекомендуем использовать возвращаемые `self` значения и в `links` качестве целевых URL-адресов вызовов RESTful вместо создания путей в клиенте. Используя возвращенные URL-адреса, можно убедиться, что будущие изменения в путях не нарушат код клиента.

### <a name="working-with-collections-of-entities"></a>Работа с коллекциями сущностей

Ранее API v2 вернул все доступные сущности в результате. Чтобы обеспечить более детализированный контроль над ожидаемым размером ответа в v3, все результаты коллекции будут разбиты на страницы. Вы можете управлять количеством возвращаемых сущностей и начальным смещением страницы. Такое поведение упрощает прогнозирование среды выполнения обработчика ответа.

Основная форма ответа одинакова для всех коллекций:

```json
{
    "values": [
        {     
        }
    ],
    "@nextLink": "https://{region}.api.cognitive.microsoft.com/speechtotext/v3.0/{collection}?skip=100&top=100"
}
```

`values`Свойство содержит подмножество доступных сущностей коллекции. Количество и смещение можно контролировать с помощью `skip` `top` параметров запроса и. Если `@nextLink` параметр не имеет значение `null` , доступны дополнительные данные, и можно получить следующий пакет данных, выполнив команду Get On `$.@nextLink` .

Это изменение требует вызова `GET` для коллекции в цикле, пока не будут возвращены все элементы.

>[!IMPORTANT]
>Если ответ GET `speechtotext/v3.0/{collection}` содержит значение в `$.@nextLink` , продолжайте выполнять процедуру, `GETs` `$.@nextLink` пока `$.@nextLink` не будет задано получение всех элементов этой коллекции.

### <a name="creating-transcriptions"></a>Создание транскрипций

Подробное описание создания пакетов транскрипции можно найти в разделе [практические руководства](./batch-transcription.md)по записи пакетов.

API для транскрипции v3 позволяет явно задать конкретные параметры для транскрипции. Все свойства конфигурации (необязательно) теперь можно задать в `properties` свойстве.
Версия V3 также поддерживает несколько входных файлов, поэтому для них требуется список URL-адресов, а не один URL-адрес в версии 2. Теперь имя свойства v2 `recordingsUrl` находится `contentUrls` в версии 3. Функция анализа тональности в транскрипциях была удалена в версии 3. См. раздел [анализ текста](https://azure.microsoft.com/en-us/services/cognitive-services/text-analytics/) службы Microsoft для анализа тональности.

Новое свойство `timeToLive` в разделе `properties` может помочь очистить существующие завершенные сущности. `timeToLive`Указывает время, после которого Завершенная сущность будет автоматически удалена. Задайте для него большое значение (например, `PT12H` ), если сущности постоянно отправляются, потребляются и удаляются, и, следовательно, обычно обрабатываются в течение 12 часов.

**текст запроса записи транскрипции v2:**

```json
{
  "locale": "en-US",
  "name": "Transcription using locale en-US",
  "recordingsUrl": "https://contoso.com/mystoragelocation",
  "properties": {
    "AddDiarization": "False",
    "AddWordLevelTimestamps": "False",
    "PunctuationMode": "DictatedAndAutomatic",
    "ProfanityFilterMode": "Masked"
  }
}
```

**текст запроса POST для транскрипции V3:**

```json
{
  "locale": "en-US",
  "displayName": "Transcription using locale en-US",
  "contentUrls": [
    "https://contoso.com/mystoragelocation",
    "https://contoso.com/myotherstoragelocation"
  ],
  "properties": {
    "diarizationEnabled": false,
    "wordLevelTimestampsEnabled": false,
    "punctuationMode": "DictatedAndAutomatic",
    "profanityFilterMode": "Masked"
  }
}
```
>[!IMPORTANT]
>Переименуйте свойство `recordingsUrl` в `contentUrls` и передайте массив URL-адресов вместо одного URL-адреса. Передайте параметры `diarizationEnabled` для `wordLevelTimestampsEnabled` или `bool` вместо `string` .

### <a name="format-of-v3-transcription-results"></a>Формат результатов транскрипции v3

Схема результатов транскрипции немного изменилась, чтобы согласовать с транскрипциями, созданными конечными точками в режиме реального времени. Подробное описание нового формата см. в разделе [практические советы](./batch-transcription.md)по записи пакетов. Схема результата публикуется в [репозитории примера GitHub](https://aka.ms/csspeech/samples) в разделе `samples/batch/transcriptionresult_v3.schema.json` .

Имена свойств теперь имеют стиль Camel, а значения для `channel` и `speaker` теперь используют целочисленные типы. Формат длительности теперь использует структуру, описанную в стандарте ISO 8601, которая соответствует формату длительности, используемому в других API-интерфейсах Azure.

Пример результата транскрипции v3. Различия описаны в комментариях.

```json
{
  "source": "...",                      // (new in v3) was AudioFileName / AudioFileUrl
  "timestamp": "2020-06-16T09:30:21Z",  // (new in v3)
  "durationInTicks": 41200000,          // (new in v3) was AudioLengthInSeconds
  "duration": "PT4.12S",                // (new in v3)
  "combinedRecognizedPhrases": [        // (new in v3) was CombinedResults
    {
      "channel": 0,                     // (new in v3) was ChannelNumber
      "lexical": "hello world",
      "itn": "hello world",
      "maskedITN": "hello world",
      "display": "Hello world."
    }
  ],
  "recognizedPhrases": [                // (new in v3) was SegmentResults
    {
      "recognitionStatus": "Success",   // 
      "channel": 0,                     // (new in v3) was ChannelNumber
      "offset": "PT0.07S",              // (new in v3) new format, was OffsetInSeconds
      "duration": "PT1.59S",            // (new in v3) new format, was DurationInSeconds
      "offsetInTicks": 700000.0,        // (new in v3) was Offset
      "durationInTicks": 15900000.0,    // (new in v3) was Duration

      // possible transcriptions of the current phrase with confidences
      "nBest": [
        {
          "confidence": 0.898652852,phrase
          "speaker": 1,
          "lexical": "hello world",
          "itn": "hello world",
          "maskedITN": "hello world",
          "display": "Hello world.",

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
>[!IMPORTANT]
>Десериализация результатов транскрипции в новый тип, как показано выше. Вместо одного файла для каждого звукового канала следует отличать каналы путем проверки значения свойства `channel` для каждого элемента в `recognizedPhrases` . Теперь для каждого входного файла имеется один файл результатов.


### <a name="getting-the-content-of-entities-and-the-results"></a>Получение содержимого сущностей и результатов

В версии 2 ссылки на входные и результирующие файлы были встроены вместе с остальными метаданными сущности. В качестве улучшения в версии 3 существует четкое разделение метаданных сущности (возвращаемое функцией GET On `$.self` ), а также сведения и учетные данные для доступа к результирующим файлам. Это разделение помогает защитить данные клиентов и позволяет точно контролировать длительность срока действия учетных данных.

В версии 3 `links` включите подсвойство, которое вызывается, `files` Если сущность предоставляет данные (наборы данных, транскрипции, конечные точки или оценки). При вызове GET `$.links.files` возвращается список файлов и URL-адрес SAS для доступа к содержимому каждого файла. Для управления сроком действия URL-адресов SAS можно использовать параметр запроса `sasValidityInSeconds` для указания времени существования.

**Транскрипция v2:**

```json
{
  "id": "9891c965-bb32-4880-b14b-6d44efb158f3",
  "status": "Succeeded",
  "reportFileUrl": "https://contoso.com/report.txt?st=2018-02-09T18%3A07%3A00Z&se=2018-02-10T18%3A07%3A00Z&sp=rl&sv=2017-04-17&sr=b&sig=6c044930-3926-4be4-be76-f728327c53b5",
  "resultsUrls": {
    "channel_0": "https://contoso.com/audiofile1.wav?st=2018-02-09T18%3A07%3A00Z&se=2018-02-10T18%3A07%3A00Z&sp=rl&sv=2017-04-17&sr=b&sig=6c044930-3926-4be4-be76-f72832e6600c",
    "channel_1": "https://contoso.com/audiofile2.wav?st=2018-02-09T18%3A07%3A00Z&se=2018-02-10T18%3A07%3A00Z&sp=rl&sv=2017-04-17&sr=b&sig=3e0163f1-0029-4d4a-988d-3fba7d7c53b5"
  }
}
```

**v3 запись транскрипции:**

```json
{
    "self": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/transcriptions/9891c965-bb32-4880-b14b-6d44efb158f3",
    "links": {
      "files": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/transcriptions/9891c965-bb32-4880-b14b-6d44efb158f3/files"
    } 
}
```

**В результате ПОЛУЧАЕТся `$.links.files` следующее:**

```json
{
  "values": [
    {
      "self": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/transcriptions/9891c965-bb32-4880-b14b-6d44efb158f3/files/f23e54f5-ed74-4c31-9730-2f1a3ef83ce8",
      "name": "Name",
      "kind": "Transcription",
      "properties": {
        "size": 200
      },
      "createdDateTime": "2020-01-13T08:00:00Z",
      "links": {
        "contentUrl": "https://customspeech-usw.blob.core.windows.net/artifacts/mywavefile1.wav.json?st=2018-02-09T18%3A07%3A00Z&se=2018-02-10T18%3A07%3A00Z&sp=rl&sv=2017-04-17&sr=b&sig=e05d8d56-9675-448b-820c-4318ae64c8d5"
      }
    },
    {
      "self": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/transcriptions/9891c965-bb32-4880-b14b-6d44efb158f3/files/28bc946b-c251-4a86-84f6-ea0f0a2373ef",
      "name": "Name",
      "kind": "TranscriptionReport",
      "properties": {
        "size": 200
      },
      "createdDateTime": "2020-01-13T08:00:00Z",
      "links": {
        "contentUrl": "https://customspeech-usw.blob.core.windows.net/artifacts/report.json?st=2018-02-09T18%3A07%3A00Z&se=2018-02-10T18%3A07%3A00Z&sp=rl&sv=2017-04-17&sr=b&sig=e05d8d56-9675-448b-820c-4318ae64c8d5"
      }
    }
  ],
  "@nextLink": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/transcriptions/9891c965-bb32-4880-b14b-6d44efb158f3/files?skip=2&top=2"
}
```

`kind`Свойство указывает формат содержимого файла. Для транскрипций файлы типа `TranscriptionReport` представляют собой сводку по заданию, а файлы типа `Transcription` являются результатом самого задания.

>[!IMPORTANT]
>Чтобы получить результаты операций, используйте `GET` On `/speechtotext/v3.0/{collection}/{id}/files` , которые больше не содержатся в ответах `GET` на `/speechtotext/v3.0/{collection}/{id}` или `/speechtotext/v3.0/{collection}` .

### <a name="customizing-models"></a>Настройка моделей

До версии 3 существовало различие между _акустической моделью_ и _языковой моделью_ при обучении модели. Это различие привело к тому, что при создании конечных точек или транскрипций необходимо указать несколько моделей. Чтобы упростить этот процесс для вызывающего, мы удалили различия и внесли все, что зависит от содержимого наборов данных, используемых для обучения модели. После этого изменения модель создания модели теперь поддерживает смешанные наборы данных (данные языка и акустические данные). Для конечных точек и транскрипций теперь требуется только одна модель.

В результате этого изменения потребность `kind` в в `POST` операции была удалена, и `datasets[]` Теперь массив может содержать несколько наборов данных одного и того же или смешанного типа.

Чтобы улучшить результаты обученной модели, акустические данные автоматически используются во время обучения на языке. Как правило, модели, созданные с помощью API V3, предоставляют более точные результаты, чем модели, созданные с помощью API v2.

>[!IMPORTANT]
>Чтобы настроить компонент "модель акустического и языкового", передайте все необходимые языковые и акустические наборы данных в `datasets[]` записи в `/speechtotext/v3.0/models` . В результате будет создана одна модель с настройками обеих частей.

### <a name="retrieving-base-and-custom-models"></a>Получение базовых и пользовательских моделей

Чтобы упростить получение доступных моделей, v3 разделяет коллекции базовых моделей от клиента «настроенные модели». Два маршрута теперь `GET /speechtotext/v3.0/models/base` и `GET /speechtotext/v3.0/models/` .

В версии 2 все модели были возвращены вместе в одном ответе.

>[!IMPORTANT]
>Чтобы получить список предоставленных базовых моделей для настройки, используйте `GET` On `/speechtotext/v3.0/models/base` . Вы можете найти собственные настраиваемые модели с помощью `GET` `/speechtotext/v3.0/models` .

### <a name="name-of-an-entity"></a>Имя сущности

`name`Свойство теперь имеет значение `displayName` . Это согласуется с другими API-интерфейсами Azure, чтобы не указывать свойства идентификации. Значение этого свойства не должно быть уникальным и может быть изменено после создания сущности с помощью `PATCH` операции.

**Транскрипция v2:**

```json
{
    "name": "Transcription using locale en-US"
}
```

**v3 запись транскрипции:**

```json
{
    "displayName": "Transcription using locale en-US"
}
```

>[!IMPORTANT]
>Переименуйте свойство `name` в `displayName` в коде клиента.

### <a name="accessing-referenced-entities"></a>Доступ к упоминаемым сущностям

В версии 2 упоминаемые сущности были всегда встроены, например используемые модели конечной точки. Вложение сущностей привело к большим откликам, и потребители редко потребляют вложенное содержимое. Чтобы уменьшить размер ответа и повысить производительность, упоминаемые сущности больше не встроены в ответ. Вместо этого появляется ссылка на другую сущность, и ее можно использовать для последующего `GET` (это URL-адрес), следуя тому же шаблону, что и `self` ссылка.

**Транскрипция v2:**

```json
{
  "id": "9891c965-bb32-4880-b14b-6d44efb158f3",
  "models": [
    {
      "id": "827712a5-f942-4997-91c3-7c6cde35600b",
      "modelKind": "Language",
      "lastActionDateTime": "2019-01-07T11:36:07Z",
      "status": "Running",
      "createdDateTime": "2019-01-07T11:34:12Z",
      "locale": "en-US",
      "name": "Acoustic model",
      "description": "Example for an acoustic model",
      "datasets": [
        {
          "id": "702d913a-8ba6-4f66-ad5c-897400b081fb",
          "dataImportKind": "Language",
          "lastActionDateTime": "2019-01-07T11:36:07Z",
          "status": "Succeeded",
          "createdDateTime": "2019-01-07T11:34:12Z",
          "locale": "en-US",
          "name": "Language dataset",
        }
      ]
    },
  ]
}
```

**v3 запись транскрипции:**

```json
{
  "self": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/transcriptions/9891c965-bb32-4880-b14b-6d44efb158f3",
  "model": {
    "self": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/models/021a72d0-54c4-43d3-8254-27336ead9037"
  }
}
```

Если необходимо использовать сведения о упоминаемой модели, как показано в приведенном выше примере, просто выдайте инструкцию GET On `$.model.self` .

>[!IMPORTANT]
>Чтобы получить метаданные упоминаемых сущностей, выдайте инструкцию GET On `$.{referencedEntity}.self` , например, чтобы получить модель транскрипции `GET` `$.model.self` .


### <a name="retrieving-endpoint-logs"></a>Получение журналов конечной точки

Версия V2 поддерживаемых службой результатов регистрации конечных точек. Чтобы получить результаты работы конечной точки с v2, необходимо создать "экспорт данных", который представляет моментальный снимок результатов, определенных диапазоном времени. Процесс экспорта пакетов данных был негибким. API V3 предоставляет доступ к каждому отдельному файлу и позволяет выполнять итерацию по ним.

**Конечная точка v3 успешно запущена:**

```json
{
  "self": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/endpoints/afa0669c-a01e-4693-ae3a-93baf40f26d6",
  "links": {
    "logs": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/endpoints/afa0669c-a01e-4693-ae3a-93baf40f26d6/files/logs" 
  }
}
```

**Ответ на получение `$.links.logs` :**

```json
{
  "values": [
    {
      "self": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/endpoints/6d72ad7e-f286-4a6f-b81b-a0532ca6bcaa/files/logs/2019-09-20_080000_3b5f4628-e225-439d-bd27-8804f9eed13f.wav",
      "name": "2019-09-20_080000_3b5f4628-e225-439d-bd27-8804f9eed13f.wav",
      "kind": "Audio",
      "properties": {
        "size": 12345
      },
      "createdDateTime": "2020-01-13T08:00:00Z",
      "links": {
        "contentUrl": "https://customspeech-usw.blob.core.windows.net/artifacts/2019-09-20_080000_3b5f4628-e225-439d-bd27-8804f9eed13f.wav?st=2018-02-09T18%3A07%3A00Z&se=2018-02-10T18%3A07%3A00Z&sp=rl&sv=2017-04-17&sr=b&sig=e05d8d56-9675-448b-820c-4318ae64c8d5"
      }
    }    
  ],
  "@nextLink": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/endpoints/afa0669c-a01e-4693-ae3a-93baf40f26d6/files/logs?top=2&SkipToken=2!188!MDAwMDk1ITZhMjhiMDllLTg0MDYtNDViMi1hMGRkLWFlNzRlOGRhZWJkNi8yMDIwLTA0LTAxLzEyNDY0M182MzI5NGRkMi1mZGYzLTRhZmEtOTA0NC1mODU5ZTcxOWJiYzYud2F2ITAwMDAyOCE5OTk5LTEyLTMxVDIzOjU5OjU5Ljk5OTk5OTlaIQ--"
}
```

Разбиение на страницы для журналов конечных точек работает аналогично всем другим коллекциям, за исключением того, что смещение не может быть указано. Из-за большого объема доступных данных разбиение на страницы определяется сервером.

В версии 3 журнал каждого из конечных точек можно удалить по отдельности, выполнив `DELETE` операцию с `self` файлом или с помощью `DELETE` On `$.links.logs` . Чтобы указать дату окончания, в `endDate` запрос можно добавить параметр запроса.

>[!IMPORTANT]
>Вместо создания экспортов журналов при `/api/speechtotext/v2.0/endpoints/{id}/data` использовании `/v3.0/endpoints/{id}/files/logs/` для доступа к файлам журнала по отдельности. 

### <a name="using-custom-properties"></a>Использование пользовательских свойств

Чтобы отделить пользовательские свойства от необязательных свойств конфигурации, все явно именованные свойства теперь находятся в `properties` свойстве, а все свойства, определенные вызывающими объектами, теперь находятся в `customProperties` свойстве.

**сущность транскрипции v2:**

```json
{
  "properties": {
    "customerDefinedKey": "value",
    "diarizationEnabled": "False",
    "wordLevelTimestampsEnabled": "False"
  }
}
```

**сущность транскрипции V3:**

```json
{
  "properties": {
    "diarizationEnabled": false,
    "wordLevelTimestampsEnabled": false 
  },
  "customProperties": {
    "customerDefinedKey": "value"
  }
}
```

Это изменение также позволяет использовать правильные типы для всех явно именованных свойств в разделе `properties` (например, Boolean вместо String).

>[!IMPORTANT]
>Передайте все пользовательские свойства `customProperties` в качестве, а не `properties` в `POST` запросах.

### <a name="response-headers"></a>Заголовки ответов

v3 больше не возвращает `Operation-Location` заголовок в дополнение к `Location` заголовку в `POST` запросах. Значения обоих заголовков в версии 2 совпадают. Теперь `Location` возвращается только.

Так как новая версия API теперь управляется службой управления API Azure (APIM), заголовки, связанные с регулированием `X-RateLimit-Limit` , `X-RateLimit-Remaining` и `X-RateLimit-Reset` не содержащиеся в заголовках ответа.

>[!IMPORTANT]
>Считывает расположение из заголовка ответа `Location` вместо `Operation-Location` . В случае с кодом ответа 429 считайте `Retry-After` значение заголовка вместо `X-RateLimit-Limit` , `X-RateLimit-Remaining` или `X-RateLimit-Reset` .


### <a name="accuracy-tests"></a>Проверки точности

Тесты точности были переименованы в оценки, так как новое имя описывает более подходящее представление. Новые пути: `https://{region}.api.cognitive.microsoft.com/speechtotext/v3.0/evaluations` .

>[!IMPORTANT]
>Переименуйте сегмент пути `accuracytests` в `evaluations` код клиента.


## <a name="next-steps"></a>Дальнейшие действия

Изучите все возможности этих часто используемых API-интерфейсов RESTFUL, предоставляемых речевыми службами:

* [REST API преобразования речи в текст;](rest-speech-to-text.md)
* [Документ Swagger](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0) для версии 3 REST API
* Пример кода для выполнения пакетной записи можно просмотреть в [репозитории примера GitHub](https://aka.ms/csspeech/samples) в `samples/batch` подкаталоге.
