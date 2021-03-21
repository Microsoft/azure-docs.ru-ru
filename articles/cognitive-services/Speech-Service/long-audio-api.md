---
title: API длинного звука — служба речи
titleSuffix: Azure Cognitive Services
description: Узнайте, как API длинного аудио-интерфейса предназначен для асинхронного синтеза длинного текста в речь.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 08/11/2020
ms.author: trbye
ms.openlocfilehash: 65c0d80394317c2b2bfbf621d3cc2ad0c2e3448a
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102618412"
---
# <a name="long-audio-api"></a>Длинный аудио API

Длинный аудио API предназначен для асинхронного синтеза текста длинной формы в речь (например, в аудио книгах, новостных статьях и документах). Этот API не возвращает синтезированный звук в режиме реального времени, а предполагает, что вы запрашиваете ответ и используете выходные данные, так как они становятся доступными из службы. В отличие от API преобразования текста в речь, используемого речевым пакетом SDK, длинный аудио API может создавать синтезированный звук дольше 10 минут, что делает его идеальным для издателей и платформ звуковых данных для создания длинного звукового содержимого, такого как звуковые книги в пакете.

Дополнительные преимущества для длинного аудио API:

* Синтезированный речевой ввод, возвращенный службой, использует лучшие нейронные голоса.
* Нет необходимости развертывать голосовую конечную точку, так как она позволяет объединять голоса в режиме без реального времени.

> [!NOTE]
> Длинный аудио API теперь поддерживает как [открытые нейронные голоса](./language-support.md#neural-voices) , так и [пользовательские нейронные голоса](./how-to-custom-voice.md#custom-neural-voices).

## <a name="workflow"></a>Рабочий процесс

Как правило, при использовании длинного аудио API вы отправляете текстовый файл или файлы для синтезирования, опрашиваете состояние, а затем, если состояние прошло успешно, можно загрузить выходные данные звука.

На этой схеме представлен общий обзор рабочего процесса.

![Схема рабочего потока для длинного звука API](media/long-audio-api/long-audio-api-workflow.png)

## <a name="prepare-content-for-synthesis"></a>Подготовка содержимого для синтеза

При подготовке текстового файла убедитесь, что он:

* Является обычным текстом (txt) или SSML Text (. txt)
* Кодируется как [UTF-8 с меткой порядка байтов (BOM)](https://www.w3.org/International/questions/qa-utf8-bom.en#bom)
* Является отдельным файлом, а не ZIP
* Содержит более 400 символов для обычного текста или 400 [оплачиваемых символов](./text-to-speech.md#pricing-note) для текста SSML и менее 10 000 абзацев
  * Для обычного текста каждый абзац отделяется нажатием клавиши **Ввод/возврат** -просмотр. [пример входного текста с открытым текстом](https://github.com/Azure-Samples/Cognitive-Speech-TTS/blob/master/CustomVoice-API-Samples/Java/en-US.txt)
  * Для текста SSML каждая часть SSML считается абзацом. SSML части должны быть разделены с помощью различных абзацев. [Пример текстового ввода](https://github.com/Azure-Samples/Cognitive-Speech-TTS/blob/master/CustomVoice-API-Samples/Java/SSMLTextInputSample.txt) для просмотра SSML

## <a name="sample-code"></a>Пример кода
Оставшаяся часть этой страницы посвящена Python, но пример кода для длинного аудио API доступен на сайте GitHub для следующих языков программирования:

* [Пример кода: Python](https://github.com/Azure-Samples/Cognitive-Speech-TTS/tree/master/CustomVoice-API-Samples/Python)
* [Пример кода: C #](https://github.com/Azure-Samples/Cognitive-Speech-TTS/tree/master/CustomVoice-API-Samples/CSharp)
* [Пример кода: Java](https://github.com/Azure-Samples/Cognitive-Speech-TTS/blob/master/CustomVoice-API-Samples/Java/)

## <a name="python-example"></a>Пример на Python

В этом разделе содержатся примеры Python, демонстрирующие базовое использование длинного аудио API. Создайте проект Python, используя любую IDE или любой текстовый редактор. Затем скопируйте этот фрагмент кода в файл с именем `long_audio_synthesis_client.py` .

```python
import json
import ntpath
import requests
```

Эти библиотеки используются для создания HTTP-запроса и вызова длительного синтеза текста в речь REST API.

### <a name="get-a-list-of-supported-voices"></a>Получение списка поддерживаемых голосов

Чтобы получить список поддерживаемых голосов, отправьте запрос GET в `https://<endpoint>/api/texttospeech/v3.0/longaudiosynthesis/voices` .


Этот код позволяет получить полный список голосов для определенного региона или конечной точки, которые можно использовать.
```python
def get_voices():
    region = '<region>'
    key = '<your_key>'
    url = 'https://{}.customvoice.api.speech.microsoft.com/api/texttospeech/v3.0/longaudiosynthesis/voices'.format(region)
    header = {
        'Ocp-Apim-Subscription-Key': key
    }

    response = requests.get(url, headers=header)
    print(response.text)

get_voices()
```

Измените следующие значения:

* Замените `<your_key>` на ваш ключ подписки службы "Речь". Эти сведения доступны на вкладке **Обзор** ресурса в [портал Azure](https://aka.ms/azureportal).
* Замените на `<region>` регион, в котором был создан ваш речевой ресурс (например: `eastus` или `westus` ). Эти сведения доступны на вкладке **Обзор** ресурса в [портал Azure](https://aka.ms/azureportal).

Вы увидите выходные данные следующего вида:

```console
{
  "values": [
    {
      "locale": "en-US",
      "voiceName": "en-US-AriaNeural",
      "description": "",
      "gender": "Female",
      "createdDateTime": "2020-05-21T05:57:39.123Z",
      "properties": {
        "publicAvailable": true
      }
    },
    {
      "id": "8fafd8cd-5f95-4a27-a0ce-59260f873141"
      "locale": "en-US",
      "voiceName": "my custom neural voice",
      "description": "",
      "gender": "Male",
      "createdDateTime": "2020-05-21T05:25:40.243Z",
      "properties": {
        "publicAvailable": false
      }
    }
  ]
}
```

Если **свойство Properties. публикаваилабле** имеет **значение true**, то речь является общедоступным нейронным голосовым. В противном случае это пользовательский нейронный счет.

### <a name="convert-text-to-speech"></a>Преобразование текста в речь

Подготовьте входной текстовый файл, как обычный текст, так и SSML текст, а затем добавьте следующий код в `long_audio_synthesis_client.py` :

> [!NOTE]
> `concatenateResult` является необязательным параметром. Если этот параметр не задан, выходные данные будут формироваться для каждого абзаца. Вы также можете объединить звуковые данные в 1 выход, задав параметр. 
> `outputFormat` также является необязательным. По умолчанию для вывода звука задано значение Metallica-16khz-16-разрядный-Mono-PCM. Дополнительные сведения о поддерживаемых форматах вывода звука см. в разделе [форматы выходных данных звука](#audio-output-formats).

```python
def submit_synthesis():
    region = '<region>'
    key = '<your_key>'
    input_file_path = '<input_file_path>'
    locale = '<locale>'
    url = 'https://{}.customvoice.api.speech.microsoft.com/api/texttospeech/v3.0/longaudiosynthesis'.format(region)
    header = {
        'Ocp-Apim-Subscription-Key': key
    }

    voice_identities = [
        {
            'voicename': '<voice_name>'
        }
    ]

    payload = {
        'displayname': 'long audio synthesis sample',
        'description': 'sample description',
        'locale': locale,
        'voices': json.dumps(voice_identities),
        'outputformat': 'riff-16khz-16bit-mono-pcm',
        'concatenateresult': True,
    }

    filename = ntpath.basename(input_file_path)
    files = {
        'script': (filename, open(input_file_path, 'rb'), 'text/plain')
    }

    response = requests.post(url, payload, headers=header, files=files)
    print('response.status_code: %d' % response.status_code)
    print(response.headers['Location'])

submit_synthesis()
```

Измените следующие значения:

* Замените `<your_key>` на ваш ключ подписки службы "Речь". Эти сведения доступны на вкладке **Обзор** ресурса в [портал Azure](https://aka.ms/azureportal).
* Замените на `<region>` регион, в котором был создан ваш речевой ресурс (например: `eastus` или `westus` ). Эти сведения доступны на вкладке **Обзор** ресурса в [портал Azure](https://aka.ms/azureportal).
* Замените на `<input_file_path>` путь к текстовому файлу, подготовленному для преобразования текста в речь.
* Замените на `<locale>` нужный языковой стандарт вывода. Дополнительные сведения см. в разделе [Поддержка языков](language-support.md#neural-voices).

Используйте один из голосов, возвращенных предыдущим вызовом к `/voices` конечной точке.

* При использовании общедоступного нейронного голоса замените `<voice_name>` требуемым выходным голосовым.
* Чтобы использовать пользовательский нейронный язык, замените `voice_identities` переменную на следующую и замените `<voice_id>` на `id` Настраиваемый нейронный счет.
```Python
voice_identities = [
    {
        'id': '<voice_id>'
    }
]
```

Вы увидите выходные данные следующего вида:

```console
response.status_code: 202
https://<endpoint>/api/texttospeech/v3.0/longaudiosynthesis/<guid>
```

> [!NOTE]
> При наличии более 1 входных файлов необходимо отправить несколько запросов. Существуют некоторые ограничения, которые необходимо учитывать.
> * Клиент может отправлять до **5** запросов на сервер в секунду для каждой учетной записи подписки Azure. Если оно превышает ограничение, клиент получает код ошибки 429 (слишком много запросов). Сократите количество запросов в секунду.
> * Сервер может запуститься и поставить в очередь до **120** запросов для каждой учетной записи подписки Azure. Если оно превышает ограничение, сервер возвратит код ошибки 429 (слишком много запросов). Подождите и не отправляйте новый запрос, пока не будут завершены некоторые запросы.

URL-адрес в выходных данных можно использовать для получения состояния запроса.

### <a name="get-information-of-a-submitted-request"></a>Получение сведений о отправленном запросе

Чтобы получить состояние отправленного запроса синтеза, просто отправьте запрос GET по URL-адресу, возвращенному на предыдущем шаге.
```Python

def get_synthesis():
    url = '<url>'
    key = '<your_key>'
    header = {
        'Ocp-Apim-Subscription-Key': key
    }
    response = requests.get(url, headers=header)
    print(response.text)

get_synthesis()
```
Выходные данные будут выглядеть следующим образом:
```console
response.status_code: 200
{
  "models": [
    {
      "voiceName": "en-US-AriaNeural"
    }
  ],
  "properties": {
    "outputFormat": "riff-16khz-16bit-mono-pcm",
    "concatenateResult": false,
    "totalDuration": "PT5M57.252S",
    "billableCharacterCount": 3048
  },
  "id": "eb3d7a81-ee3e-4e9a-b725-713383e71677",
  "lastActionDateTime": "2021-01-14T11:12:27.240Z",
  "status": "Succeeded",
  "createdDateTime": "2021-01-14T11:11:02.557Z",
  "locale": "en-US",
  "displayName": "long audio synthesis sample",
  "description": "sample description"
}
```

Из `status` свойства можно прочитать состояние этого запроса. Запрос начнется с `NotStarted` состояния, затем изменится на `Running` , а наконец станет `Succeeded` или `Failed` . Можно использовать цикл для опроса этого API до того, как состояние изменится на `Succeeded` .

### <a name="download-audio-result"></a>Скачать звуковой результат

После выполнения запроса синтеза звука можно скачать звуковой результат, вызвав GET `/files` API.

```python
def get_files():
    id = '<request_id>'
    region = '<region>'
    key = '<your_key>'
    url = 'https://{}.customvoice.api.speech.microsoft.com/api/texttospeech/v3.0/longaudiosynthesis/{}/files'.format(region, id)
    header = {
        'Ocp-Apim-Subscription-Key': key
    }

    response = requests.get(url, headers=header)
    print('response.status_code: %d' % response.status_code)
    print(response.text)

get_files()
```
Замените на `<request_id>` идентификатор запроса, для которого необходимо скачать результат. Его можно найти в ответе на предыдущем шаге.

Выходные данные будут выглядеть следующим образом:
```console
response.status_code: 200
{
  "values": [
    {
      "name": "2779f2aa-4e21-4d13-8afb-6b3104d6661a.txt",
      "kind": "LongAudioSynthesisScript",
      "properties": {
        "size": 4200
      },
      "createdDateTime": "2021-01-14T11:11:02.410Z",
      "links": {
        "contentUrl": "https://customvoice-usw.blob.core.windows.net/artifacts/input.txt?st=2018-02-09T18%3A07%3A00Z&se=2018-02-10T18%3A07%3A00Z&sp=rl&sv=2017-04-17&sr=b&sig=e05d8d56-9675-448b-820c-4318ae64c8d5"
      }
    },
    {
      "name": "voicesynthesis_waves.zip",
      "kind": "LongAudioSynthesisResult",
      "properties": {
        "size": 9290000
      },
      "createdDateTime": "2021-01-14T11:12:27.226Z",
      "links": {
        "contentUrl": "https://customvoice-usw.blob.core.windows.net/artifacts/voicesynthesis_waves.zip?st=2018-02-09T18%3A07%3A00Z&se=2018-02-10T18%3A07%3A00Z&sp=rl&sv=2017-04-17&sr=b&sig=e05d8d56-9675-448b-820c-4318ae64c8d5"
      }
    }
  ]
}
```
Выходные данные содержат сведения о 2 файлах. Объект с `"kind": "LongAudioSynthesisScript"` — отправленный входной скрипт. Второй `"kind": "LongAudioSynthesisResult"` — результат этого запроса.
Результат — ZIP, содержащий созданные выходные файлы вывода вместе с копией входного текста.

Оба файла можно скачать по URL-адресу в `links.contentUrl` свойстве.

### <a name="get-all-synthesis-requests"></a>Получение всех запросов синтеза

Список всех отправленных запросов можно получить с помощью следующего кода:

```python
def get_synthesis():
    region = '<region>'
    key = '<your_key>'
    url = 'https://{}.customvoice.api.speech.microsoft.com/api/texttospeech/v3.0/longaudiosynthesis/'.format(region)    
    header = {
        'Ocp-Apim-Subscription-Key': key
    }

    response = requests.get(url, headers=header)
    print('response.status_code: %d' % response.status_code)
    print(response.text)

get_synthesis()
```

Выходные данные будут выглядеть следующим образом:
```console
response.status_code: 200
{
  "values": [
    {
      "models": [
        {
          "id": "8fafd8cd-5f95-4a27-a0ce-59260f873141",
          "voiceName": "my custom neural voice"
        }
      ],
      "properties": {
        "outputFormat": "riff-16khz-16bit-mono-pcm",
        "concatenateResult": false,
        "totalDuration": "PT1S",
        "billableCharacterCount": 5
      },
      "id": "f9f0bb74-dfa5-423d-95e7-58a5e1479315",
      "lastActionDateTime": "2021-01-05T07:25:42.433Z",
      "status": "Succeeded",
      "createdDateTime": "2021-01-05T07:25:13.600Z",
      "locale": "en-US",
      "displayName": "Long Audio Synthesis",
      "description": "Long audio synthesis sample"
    },
    {
      "models": [
        {
          "voiceName": "en-US-AriaNeural"
        }
      ],
      "properties": {
        "outputFormat": "riff-16khz-16bit-mono-pcm",
        "concatenateResult": false,
        "totalDuration": "PT5M57.252S",
        "billableCharacterCount": 3048
      },
      "id": "eb3d7a81-ee3e-4e9a-b725-713383e71677",
      "lastActionDateTime": "2021-01-14T11:12:27.240Z",
      "status": "Succeeded",
      "createdDateTime": "2021-01-14T11:11:02.557Z",
      "locale": "en-US",
      "displayName": "long audio synthesis sample",
      "description": "sample description"
    }
  ]
}
```

`values` свойство содержит список запросов синтеза. Этот список разбит на страницы с максимальным размером страниц 100. Если количество запросов превышает 100, `"@nextLink"` для получения следующей страницы списка страничных страниц будет указано свойство.

```console
  "@nextLink": "https://<endpoint>/api/texttospeech/v3.0/longaudiosynthesis/?top=100&skip=100"
```

Можно также настроить размер страницы и число пропусков, указав `skip` и `top` в параметре URL-адреса.

### <a name="remove-previous-requests"></a>Удалить предыдущие запросы

Служба будет обслуживать до **20 000** запросов для каждой учетной записи подписки Azure. Если сумма запроса превышает это ограничение, удалите предыдущие запросы перед созданием новых. Если не удалить существующие запросы, вы получите уведомление об ошибке.

В следующем коде показано, как удалить конкретный запрос синтеза.
```python
def delete_synthesis():
    id = '<request_id>'
    region = '<region>'
    key = '<your_key>'
    url = 'https://{}.customvoice.api.speech.microsoft.com/api/texttospeech/v3.0/longaudiosynthesis/{}/'.format(region, id)
    header = {
        'Ocp-Apim-Subscription-Key': key
    }

    response = requests.delete(url, headers=header)
    print('response.status_code: %d' % response.status_code)
```

Если запрос успешно удален, код состояния отклика будет иметь значение HTTP 204 (нет содержимого).

```console
response.status_code: 204
```

> [!NOTE]
> Запросы с состоянием `NotStarted` или `Running` не могут быть удалены или удалены.

Завершенный `long_audio_synthesis_client.py` доступ доступен на [GitHub](https://github.com/Azure-Samples/Cognitive-Speech-TTS/blob/master/CustomVoice-API-Samples/Python/voiceclient.py).

## <a name="http-status-codes"></a>Коды состояния HTTP

В следующей таблице описаны коды и сообщения HTTP-ответов из REST API.

| API | Код состояния HTTP | Описание | Решение |
|-----|------------------|-------------|----------|
| Создать | 400 | Синтез голоса не включен в этом регионе. | Измените ключ подписки на речь с поддерживаемым регионом. |
|        | 400 | Допустима только **Стандартная** подписка на речь для этого региона. | Измените ключ подписки на речь на ценовую категорию "Стандартный". |
|        | 400 | Превышает ограничение в 20 000 запросов для учетной записи Azure. Перед отправкой новых запросов удалите некоторые из них. | Сервер будет иметь до 20 000 запросов для каждой учетной записи Azure. Перед отправкой новых запросов удалите некоторые из них. |
|        | 400 | Эта модель не может использоваться в синтезе речи: {modelID}. | Убедитесь, что состояние {modelID} указано правильно. |
|        | 400 | Регион для запроса не соответствует региону для модели: {modelID}. | Убедитесь, что регион {modelID} соответствует региону запроса. |
|        | 400 | Синтез голоса поддерживает только текстовый файл в кодировке UTF-8 с маркером порядка следования байтов. | Убедитесь, что входные файлы находятся в кодировке UTF-8 с маркером порядка следования байтов. |
|        | 400 | В запросе синтеза голоса допускаются только допустимые входные данные SSML. | Убедитесь, что входные выражения SSML верны. |
|        | 400 | Голосовое имя {Воиценаме} не найдено во входном файле. | Имя входного голоса SSML не соответствует ИДЕНТИФИКАТОРу модели. |
|        | 400 | Число абзацев во входном файле должно быть меньше 10 000. | Убедитесь, что число абзацев в файле меньше 10 000. |
|        | 400 | Длина входного файла должна составлять более 400 символов. | Убедитесь, что входной файл содержит больше 400 символов. |
|        | 404 | Не удается найти модель, объявленную в определении синтеза голоса: {modelID}. | Проверьте правильность {modelID}. |
|        | 429 | Превышает ограничение активного синтеза речи. Дождитесь завершения некоторых запросов. | Сервер может запуститься и поставить в очередь до 120 запросов для каждой учетной записи Azure. Подождите и не отправляйте новые запросы, пока не будут завершены некоторые запросы. |
| Все       | 429 | Слишком много запросов. | Клиент может отправлять до 5 запросов на сервер в секунду для каждой учетной записи Azure. Сократите количество запросов в секунду. |
| Удалить    | 400 | Задача синтеза голоса по-прежнему используется. | Можно удалить только **Завершенные** или **Невыполненные** запросы. |
| жетбид   | 404 | Не удается найти указанную сущность. | Убедитесь, что идентификатор синтеза правильный. |

## <a name="regions-and-endpoints"></a>Регионы и конечные точки

Длинный аудио API доступен в нескольких регионах с уникальными конечными точками.

| Регион | Конечная точка |
|--------|----------|
| Восточная часть США | `https://eastus.customvoice.api.speech.microsoft.com` |
| Центральная Индия | `https://centralindia.customvoice.api.speech.microsoft.com` |
| Юго-Восточная Азия | `https://southeastasia.customvoice.api.speech.microsoft.com` |
| южная часть Соединенного Королевства | `https://uksouth.customvoice.api.speech.microsoft.com` |
| Западная Европа | `https://westeurope.customvoice.api.speech.microsoft.com` |

## <a name="audio-output-formats"></a>Форматы вывода звука

Поддерживаются гибкие форматы вывода звука. Вы можете создавать выходные данные для каждого абзаца или объединять звуковые выходные данные в один выход, задав параметр "Конкатенатересулт". API длинного звука поддерживает следующие форматы вывода звука:

> [!NOTE]
> Формат аудио по умолчанию — Metallica-16khz-16-разрядный-Mono-PCM.

* Metallica-8khz-16-разрядный-Mono-PCM
* Metallica-16khz-16-разрядный-Mono-PCM
* Metallica-24khz-16-разрядный-Mono-PCM
* Metallica-48khz-16-разрядный-Mono-PCM
* Audio-16khz-32kbitrate-Mono-MP3
* Audio-16khz-64kbitrate-Mono-MP3
* Audio-16khz-128kbitrate-Mono-MP3
* Audio-24khz-48kbitrate-Mono-MP3
* Audio-24khz-96kbitrate-Mono-MP3
* Audio-24khz-160kbitrate-Mono-MP3
