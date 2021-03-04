---
title: Установка и запуск контейнеров DOCKER для API-интерфейсов речевой службы
titleSuffix: Azure Cognitive Services
description: Используйте контейнеры DOCKER для службы распознавания речи, чтобы выполнять распознавание речи, транскрипцию, создание и другие действия в локальной среде.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/02/2021
ms.author: aahi
ms.custom: cog-serv-seo-aug-2020
keywords: локальная среда, Docker, контейнер
ms.openlocfilehash: 4970b33d51ed7ef54727c1c15e2482ff10d70506
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102032949"
---
# <a name="install-and-run-docker-containers-for-the-speech-service-apis"></a>Установка и запуск контейнеров DOCKER для API-интерфейсов речевой службы 

Контейнеры позволяют запускать некоторые API-интерфейсы службы "Речь" в собственной среде. Контейнеры соответствуют конкретным требованиям к безопасности и управлению данными. В этой статье содержатся сведения о скачивании, установке и запуске контейнера службы "Речь".

Речевые контейнеры позволяют клиентам создавать архитектуру приложений для распознавания речи, оптимизированную как для надежных облачных решений, так и для пограничных локальных сред. Доступно несколько контейнеров, которые используют те же [цены](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/) , что и облачные службы распознавания речи Azure.


> [!IMPORTANT]
> Теперь общедоступны следующие контейнеры речи:
> * стандартное преобразование речи в текст;
> * пользовательское преобразование речи в текст;
> * стандартное преобразование текста в речь;
> * нейронное преобразование текста в речь;
>
> Следующие контейнеры распознавания речи находятся в предварительной версии.
> * Пользовательский текст в речь
> * Речь распознавание языка 
>
> Чтобы использовать контейнеры распознавания речи, необходимо отправить онлайн-запрос и утвердить его. Дополнительные сведения см. в разделе **Утверждение запроса для запуска контейнера** ниже.

| Контейнер | Компоненты | Последняя версия |
|--|--|--|
| Преобразование речи в текст | Анализирует тональности и расшифровывает непрерывную голосовую или пакетную звукозапись в реальном времени с промежуточными результатами.  | 2.9.0 |
| пользовательское преобразование речи в текст; | Используя настраиваемую модель на [портале пользовательское распознавание речи](https://speech.microsoft.com/customspeech), расшифровывает непрерывную голосовую или пакетную звукозапись в режиме реального времени в текст с промежуточными результатами. | 2.9.0 |
| Преобразование текста в речь | Преобразует текст в голосовую речь с помощью обычного текстового ввода или языка разметки речи (SSML). | 1.11.0 |
| Пользовательский текст в речь | С помощью настраиваемой модели [пользовательского голосового портала](https://aka.ms/custom-voice-portal)преобразует текст в голосовую речь с помощью обычного текстового ввода или языка разметки речи (SSML). | 1.11.0 |
| Речь распознавание языка | Определение языка, произнесенного в звуковых файлах. | 1,0 |
| нейронное преобразование текста в речь; | Преобразует текст в голосовую речь с помощью технологии глубокой нейронной сети, обеспечивая более естественную синтезированную речь. | 1.3.0 |

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Перед использованием речевых контейнеров выполните следующие предварительные требования.

| Обязательно | Назначение |
|--|--|
| Модуль Docker | На [главном компьютере](#the-host-computer) должен быть установлен модуль Docker. Docker предоставляет пакеты, которые настраивают среду Docker в ОС [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) и [Linux](https://docs.docker.com/engine/installation/#supported-platforms). Ознакомьтесь с [общими сведениями о Docker и контейнерах](https://docs.docker.com/engine/docker-overview/).<br><br> Docker нужно настроить таким образом, чтобы контейнеры могли подключать и отправлять данные о выставлении счетов в Azure. <br><br> **В ОС Windows** для Docker нужно также настроить поддержку контейнеров Linux.<br><br> |
| Опыт работы с Docker | Требуется базовое представление о понятиях Docker, включая реестры, репозитории, контейнеры и образы контейнеров, а также знание основных команд `docker`. |
| Речевой ресурс | Для использования контейнеров необходимо следующее:<br><br>Ресурс _речи_ Azure для получения связанного ключа API и URI конечной точки. Оба значения доступны на страницах "Обзор **речи** " и "ключи" портал Azure. Они необходимы для запуска контейнера.<br><br>**{API_KEY}**: один из двух доступных ключей ресурсов на странице " **ключи** "<br><br>**{ENDPOINT_URI}**: конечная точка, указанная на странице **обзора** |

[!INCLUDE [Gathering required parameters](../containers/includes/container-gathering-required-parameters.md)]

## <a name="the-host-computer"></a>Главный компьютер

[!INCLUDE [Host Computer requirements](../../../includes/cognitive-services-containers-host-computer.md)]

### <a name="advanced-vector-extension-support"></a>Поддержка расширенного векторного расширения

**Главным** является компьютер, на котором выполняется контейнер Docker. Узел *должен поддерживать* [Расширенные векторные расширения](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#CPUs_with_AVX2) (AVX2). Вы можете проверить поддержку AVX2 на узлах Linux с помощью следующей команды:

```console
grep -q avx2 /proc/cpuinfo && echo AVX2 supported || echo No AVX2 support detected
```
> [!WARNING]
> Главный компьютер *необходим* для поддержки AVX2. Контейнер *не будет* работать правильно без поддержки AVX2.

### <a name="container-requirements-and-recommendations"></a>Требования к контейнеру и рекомендации

В следующей таблице описаны минимальное и рекомендуемое выделение ресурсов для каждого контейнера речи.

| Контейнер | Минимальные | Рекомендуемая |
|-----------|---------|-------------|
| Преобразование речи в текст | 2 ядра, 2 ГБ памяти | 4 ядра, 4 ГБ памяти |
| пользовательское преобразование речи в текст; | 2 ядра, 2 ГБ памяти | 4 ядра, 4 ГБ памяти |
| Преобразование текста в речь | 1 ядро, 2 ГБ памяти | 2 ядра, 3 ГБ памяти |
| Пользовательский текст в речь | 1 ядро, 2 ГБ памяти | 2 ядра, 3 ГБ памяти |
| Речь распознавание языка | 1 ядро, 1 ГБ памяти | 1 ядро, 1 ГБ памяти |
| нейронное преобразование текста в речь; | 6 ядер, 12 ГБ памяти | 8 ядер, 16 ГБ памяти |

* Частота каждого ядра должна быть минимум 2,6 ГГц.

Ядро и память соответствуют параметрам `--cpus` и `--memory`, которые используются как часть команды `docker run`.

> [!NOTE]
> Минимальное и рекомендуемое значение основано на ограничениях DOCKER, а *не* на ресурсах компьютера размещения. Например, контейнеры преобразования речи в текст отображают части модели больших языков, и *рекомендуется* , чтобы весь файл поместился в память, что является дополнительным 4-6 ГБ. Кроме того, первый запуск любого контейнера может занять больше времени, так как модели разбиваются на страницы в памяти.

## <a name="request-approval-to-the-run-the-container"></a>Запросить утверждение для запуска контейнера

Заполните [форму запроса](https://aka.ms/csgate) и отправьте ее, чтобы запросить доступ к контейнеру. 

[!INCLUDE [Request access to public preview](../../../includes/cognitive-services-containers-request-access.md)]


## <a name="get-the-container-image-with-docker-pull"></a>Получение образа контейнера с помощью `docker pull`

Образы контейнеров для речи доступны в следующем реестре контейнеров.

# <a name="speech-to-text"></a>[Преобразование речи в текст](#tab/stt)

| Контейнер | Хранилище |
|-----------|------------|
| Преобразование речи в текст | `mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text:latest` |

# <a name="custom-speech-to-text"></a>[пользовательское преобразование речи в текст;](#tab/cstt)

| Контейнер | Хранилище |
|-----------|------------|
| пользовательское преобразование речи в текст; | `mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text:latest` |

# <a name="text-to-speech"></a>[Преобразование текста в речь](#tab/tts)

| Контейнер | Хранилище |
|-----------|------------|
| Преобразование текста в речь | `mcr.microsoft.com/azure-cognitive-services/speechservices/text-to-speech:latest` |

# <a name="neural-text-to-speech"></a>[нейронное преобразование текста в речь;](#tab/ntts)

| Контейнер | Хранилище |
|-----------|------------|
| нейронное преобразование текста в речь; | `mcr.microsoft.com/azure-cognitive-services/speechservices/neural-text-to-speech:latest` |

# <a name="custom-text-to-speech"></a>[Пользовательский текст в речь](#tab/ctts)

| Контейнер | Хранилище |
|-----------|------------|
| Пользовательский текст в речь | `mcr.microsoft.com/azure-cognitive-services/speechservices/custom-text-to-speech:latest` |

# <a name="speech-language-detection"></a>[Речь распознавание языка](#tab/lid)

> [!TIP]
> Чтобы получить наиболее полезные результаты, мы рекомендуем использовать контейнер обнаружения языка распознавания речи с текстовыми или пользовательскими контейнерами преобразования речи в текст. 

| Контейнер | Хранилище |
|-----------|------------|
| Речь распознавание языка | `mcr.microsoft.com/azure-cognitive-services/speechservices/language-detection:latest` |

***

[!INCLUDE [Tip for using docker list](../../../includes/cognitive-services-containers-docker-list-tip.md)]

### <a name="docker-pull-for-the-speech-containers"></a>Извлечение DOCKER для контейнеров распознавания речи

# <a name="speech-to-text"></a>[Преобразование речи в текст](#tab/stt)

#### <a name="docker-pull-for-the-speech-to-text-container"></a>Опрашивающий запрос DOCKER для контейнера преобразования речи в текст

Используйте команду [DOCKER Pull](https://docs.docker.com/engine/reference/commandline/pull/) , чтобы скачать образ контейнера из реестра контейнеров Майкрософт.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text:latest
```

> [!IMPORTANT]
> `latest`Тег извлекает `en-US` языковой стандарт. Дополнительные языковые стандарты см. [в разделе языки и речь](#speech-to-text-locales).

#### <a name="speech-to-text-locales"></a>Языки перевода речи в текст

Все теги, кроме, `latest` имеют следующий формат и учитывают регистр:

```
<major>.<minor>.<patch>-<platform>-<locale>-<prerelease>
```

Следующий тег является примером формата:

```
2.6.0-amd64-en-us
```

Сведения о всех поддерживаемых языковых стандартах контейнера **для преобразования речи** в текст см. в статье [теги изображений](../containers/container-image-tags.md#speech-to-text).

# <a name="custom-speech-to-text"></a>[пользовательское преобразование речи в текст;](#tab/cstt)

#### <a name="docker-pull-for-the-custom-speech-to-text-container"></a>Опрашивающий запрос DOCKER для контейнера Пользовательское распознавание речи в текст

Используйте команду [DOCKER Pull](https://docs.docker.com/engine/reference/commandline/pull/) , чтобы скачать образ контейнера из реестра контейнеров Майкрософт.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text:latest
```

> [!NOTE]
> `locale` `voice` Пользовательские контейнеры и для пользовательских речевых контейнеров определяются настраиваемой моделью, принимаемой контейнером.

# <a name="text-to-speech"></a>[Преобразование текста в речь](#tab/tts)

#### <a name="docker-pull-for-the-text-to-speech-container"></a>Извлечение DOCKER для контейнера текста в речь

Используйте команду [DOCKER Pull](https://docs.docker.com/engine/reference/commandline/pull/) , чтобы скачать образ контейнера из реестра контейнеров Майкрософт.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/text-to-speech:latest
```

> [!IMPORTANT]
> `latest`Тег извлекает `en-US` языковой стандарт и `ariarus` голосовое значение. Дополнительные языковые стандарты см. [в разделе Языки перевода текста в речь](#text-to-speech-locales).

#### <a name="text-to-speech-locales"></a>Языки перевода текста в речь

Все теги, кроме, `latest` имеют следующий формат и учитывают регистр:

```
<major>.<minor>.<patch>-<platform>-<locale>-<voice>-<prerelease>
```

Следующий тег является примером формата:

```
1.8.0-amd64-en-us-ariarus
```

Все поддерживаемые языковые стандарты и соответствующие **голоса контейнера преобразования текста в речь см** . в разделе [теги изображений текста в речь](../containers/container-image-tags.md#text-to-speech).

> [!IMPORTANT]
> При создании HTTP-запроса POST *для преобразования текста в речь* в сообщении о [языке разметки речи (SSML)](speech-synthesis-markup.md) требуется `voice` элемент с `name` атрибутом. Значение представляет собой соответствующий языковой стандарт контейнера и голоса, также называемый ["коротким именем"](language-support.md#standard-voices). Например, `latest` тег будет иметь название Voice `en-US-AriaRUS` .

# <a name="neural-text-to-speech"></a>[нейронное преобразование текста в речь;](#tab/ntts)

#### <a name="docker-pull-for-the-neural-text-to-speech-container"></a>Извлечение DOCKER для контейнера нейронного текста в речь

Используйте команду [DOCKER Pull](https://docs.docker.com/engine/reference/commandline/pull/) , чтобы скачать образ контейнера из реестра контейнеров Майкрософт.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/neural-text-to-speech:latest
```

> [!IMPORTANT]
> `latest`Тег извлекает `en-US` языковой стандарт и `arianeural` голосовое значение. Дополнительные сведения о дополнительных языковых стандартах см. [в статье языки нейронных текстов в речь](#neural-text-to-speech-locales).

#### <a name="neural-text-to-speech-locales"></a>Языковые стандарты нейронного текста в речь

Все теги, кроме, `latest` имеют следующий формат и учитывают регистр:

```
<major>.<minor>.<patch>-<platform>-<locale>-<voice>
```

Следующий тег является примером формата:

```
1.3.0-amd64-en-us-arianeural
```

Дополнительные сведения о поддерживаемых языковых стандартах и соответствующих голосовах для контейнера преобразования **текста в речь** см. в разделе [теги изображений нейронного](../containers/container-image-tags.md#neural-text-to-speech)текста.

> [!IMPORTANT]
> При создании HTTP-запроса POST *текста в формате нейронной речи для* сообщения [SSML (язык разметки речи)](speech-synthesis-markup.md) требуется `voice` элемент с `name` атрибутом. Значение представляет собой соответствующий языковой стандарт контейнера и голоса, также называемый ["коротким именем"](language-support.md#neural-voices). Например, `latest` тег будет иметь название Voice `en-US-AriaNeural` .

# <a name="custom-text-to-speech"></a>[Пользовательский текст в речь](#tab/ctts)

#### <a name="docker-pull-for-the-custom-text-to-speech-container"></a>Опрашивающий запрос DOCKER для пользовательского контейнера преобразования текста в речь

Используйте команду [DOCKER Pull](https://docs.docker.com/engine/reference/commandline/pull/) , чтобы скачать образ контейнера из реестра контейнеров Майкрософт.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/custom-text-to-speech:latest
```

> [!NOTE]
> `locale` `voice` Пользовательские контейнеры и для пользовательских речевых контейнеров определяются настраиваемой моделью, принимаемой контейнером.

# <a name="speech-language-detection"></a>[Речь распознавание языка](#tab/lid)

#### <a name="docker-pull-for-the-speech-language-detection-container"></a>Извлечение DOCKER для контейнера распознавание языка речи

Используйте команду [DOCKER Pull](https://docs.docker.com/engine/reference/commandline/pull/) , чтобы скачать образ контейнера из реестра контейнеров Майкрософт.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/language-detection:latest
```

***

## <a name="how-to-use-the-container"></a>Использование контейнера

После размещения контейнера на [главном компьютере](#the-host-computer) воспользуйтесь следующей процедурой для работы с ним.

1. [Запустите контейнер](#run-the-container-with-docker-run) с необходимыми настройками выставления счетов. Доступны дополнительные [примеры](speech-container-configuration.md#example-docker-run-commands) команды `docker run`.
1. [Запросите конечную точку прогнозирования контейнера](#query-the-containers-prediction-endpoint).

## <a name="run-the-container-with-docker-run"></a>Запуск контейнера с помощью команды `docker run`

Воспользуйтесь командой [docker run](https://docs.docker.com/engine/reference/commandline/run/) для запуска контейнера. Дополнительные сведения о том, как получить значения и, см. в разделе [сбор обязательных параметров](#gathering-required-parameters) `{Endpoint_URI}` `{API_Key}` . Также [](speech-container-configuration.md#example-docker-run-commands) доступны дополнительные примеры `docker run` команды.

# <a name="speech-to-text"></a>[Преобразование речи в текст](#tab/stt)

Чтобы запустить стандартный контейнер преобразования *речи в текст* , выполните следующую `docker run` команду.

```bash
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Команда:

* Запускает контейнер преобразования *речи в текст* из образа контейнера.
* Выделяет 4 ядра ЦП и 4 гигабайта (ГБ) памяти.
* предоставляет TCP-порт 5000 и выделяет псевдотелетайп для контейнера;
* автоматически удаляет контейнер после завершения его работы. Образ контейнера остается доступным на главном компьютере.

> [!NOTE]
> Контейнеры поддерживают сжатый речевой ввод в пакет SDK для распознавания речи с помощью Гстреамер.
> Чтобы установить Гстреамер в контейнере, следуйте инструкциям Linux по [использованию сжатого звукового потока с пакетом SDK](how-to-use-codec-compressed-audio-input-streams.md)для гстреамер.

#### <a name="diarization-on-the-speech-to-text-output"></a>Диаризатион для вывода речи в текст
Диаризатион включен по умолчанию. чтобы получить диаризатион в ответе, используйте `diarize_speech_config.set_service_property` .

1. Задайте для параметра формат вывода фразу значение `Detailed` .
2. Установите режим диаризатион. Поддерживаемые режимы: `Identity` и `Anonymous` .
```python
diarize_speech_config.set_service_property(
    name='speechcontext-PhraseOutput.Format',
    value='Detailed',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)

diarize_speech_config.set_service_property(
    name='speechcontext-phraseDetection.speakerDiarization.mode',
    value='Identity',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
```
> [!NOTE]
> Режим "Identity" возвращает `"SpeakerId": "Customer"` или `"SpeakerId": "Agent"` .
> "Anonymous" режим возвращает `"SpeakerId": "Speaker 1"` или `"SpeakerId": "Speaker 2"`


#### <a name="analyze-sentiment-on-the-speech-to-text-output"></a>Анализ тональности на выходе из речи в текст 
Начиная с версии v 2.6.0 контейнера "речь-Text", вместо предварительной версии следует использовать конечную точку API Текстаналитикс 3,0. Например:
* `https://westus2.api.cognitive.microsoft.com/text/analytics/v3.0/sentiment`
* `https://localhost:5000/text/analytics/v3.0/sentiment`

> [!NOTE]
> Анализ текста `v3.0` API не имеет обратной совместимости с анализ текста `v3.0-preview.1` . Чтобы получить последнюю поддержку функций тональности, используйте `v2.6.0` образ контейнера для преобразования речи в текст и анализ текста `v3.0` .

Начиная с версии v 2.2.0 контейнера для преобразования речи в текст можно вызвать [API тональности Analysis v3](../text-analytics/how-tos/text-analytics-how-to-sentiment-analysis.md) на выходе. Для вызова анализа тональности требуется конечная точка ресурса API анализа текста. Пример. 
* `https://westus2.api.cognitive.microsoft.com/text/analytics/v3.0-preview.1/sentiment`
* `https://localhost:5000/text/analytics/v3.0-preview.1/sentiment`

Если вы обращаетесь к конечной точке текстовой аналитики в облаке, потребуется ключ. Если вы работаете Анализ текста локально, вам может не потребоваться его указывать.

Ключ и конечная точка передаются в контейнер речи в качестве аргументов, как показано в следующем примере.

```bash
docker run -it --rm -p 5000:5000 \
mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text:latest \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
CloudAI:SentimentAnalysisSettings:TextAnalyticsHost={TEXT_ANALYTICS_HOST} \
CloudAI:SentimentAnalysisSettings:SentimentAnalysisApiKey={SENTIMENT_APIKEY}
```

Команда:

* Выполняет те же действия, что и приведенная выше команда.
* Хранит API анализа текстаную конечную точку и ключ для отправки запросов анализа тональности. 

#### <a name="phraselist-v2-on-the-speech-to-text-output"></a>Фраселист v2 для вывода речи в текст 

Начиная с версии v 2.6.0 контейнера "речь-Text", вы можете получить выходные данные с помощью собственных фраз — либо всего предложения, либо фраз в середине. Например, вы обладаете *высокой назначением человека* в следующем предложении:

* « **Это предложение — это еще** одно предложение».

Чтобы настроить список фраз, необходимо добавить собственные фразы при выполнении вызова. Пример.

```python
    phrase="the tall man"
    recognizer = speechsdk.SpeechRecognizer(
        speech_config=dict_speech_config,
        audio_config=audio_config)
    phrase_list_grammer = speechsdk.PhraseListGrammar.from_recognizer(recognizer)
    phrase_list_grammer.addPhrase(phrase)

```

Если нужно добавить несколько фраз, вызовите `.addPhrase()` каждую фразу, чтобы добавить ее в список фраз. 


# <a name="custom-speech-to-text"></a>[пользовательское преобразование речи в текст;](#tab/cstt)

Пользовательское распознавание речи контейнер полагается на пользовательскую модель речевого *ввода* . Пользовательская модель должна быть [обучена](how-to-custom-speech-train-model.md) с помощью [пользовательского портала речевого](https://speech.microsoft.com/customspeech)ввода.

Для запуска контейнера требуется **идентификатор пользовательской модели** распознавания речи. Его можно найти на странице " **обучение** " пользовательского речевого портала. На настраиваемом портале речевого ввода перейдите на страницу **обучения** и выберите модель.
<br>

![Страница обучения пользовательской речи](media/custom-speech/custom-speech-model-training.png)

Получите **идентификатор модели** , который будет использоваться в качестве аргумента для `ModelId` параметра `docker run` команды.
<br>

![Сведения о пользовательской модели речи](media/custom-speech/custom-speech-model-details.png)

В следующей таблице представлены различные `docker run` Параметры и соответствующие им описания.

| Параметр | Описание |
|---------|---------|
| `{VOLUME_MOUNT}` | Узел [тома](https://docs.docker.com/storage/volumes/)главного компьютера, который DOCKER использует для сохранения настраиваемой модели. Например, *к:\кустомспич* , где *диск C* находится на хост-компьютере. |
| `{MODEL_ID}` | **Идентификатор модели** пользовательское распознавание речи на странице **обучения** пользовательского речевого портала. |
| `{ENDPOINT_URI}` | Для оценки и выставления счетов требуется конечная точка. Дополнительные сведения см. в разделе [сбор обязательных параметров](#gathering-required-parameters). |
| `{API_KEY}` | Требуется ключ API. Дополнительные сведения см. в разделе [сбор обязательных параметров](#gathering-required-parameters). |

Чтобы запустить контейнер *пользовательское распознавание речи в текст* , выполните следующую `docker run` команду:

```bash
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
-v {VOLUME_MOUNT}:/usr/local/models \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Команда:

* Выполняет *пользовательское распознавание речиный* контейнер из образа контейнера.
* Выделяет 4 ядра ЦП и 4 гигабайта (ГБ) памяти.
* Загружает *пользовательское распознавание речиную* модель из подключения ввода тома, например *к:\кустомспич*.
* предоставляет TCP-порт 5000 и выделяет псевдотелетайп для контейнера;
* Загружает модель, заданную `ModelId` (если она не найдена в подключении тома).
* Если пользовательская модель была ранее скачана, параметр `ModelId` игнорируется.
* автоматически удаляет контейнер после завершения его работы. Образ контейнера остается доступным на главном компьютере.


#### <a name="base-model-download-on-the-custom-speech-to-text-container"></a>Загрузка базовой модели в пользовательском контейнере преобразования речи в текст  
Начиная с версии v 2.6.0 контейнера пользовательских речевых функций можно получить доступную базовую модель с помощью параметра Option `BaseModelLocale=<locale>` . Этот параметр позволяет получить список доступных базовых моделей для этого языкового стандарта в учетной записи выставления счетов. Пример.

```bash
docker run --rm -it \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text \
BaseModelLocale={LOCALE} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Команда:

* Выполняет *пользовательское распознавание речиный* контейнер из образа контейнера.
* Проверьте и верните доступные базовые модели для целевого языкового стандарта.

Выходные данные предоставляют список базовых моделей с региональными стандартами данных, ИДЕНТИФИКАТОРом модели и датой создания. Вы можете использовать идентификатор модели для загрузки и использования конкретной базовой модели, которую вы предпочитаете. Пример.
```
Checking available base model for en-us
2020/10/30 21:54:20 [Info] Searching available base models for en-us
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2016-11-04T08:23:42Z, Id: a3d8aab9-6f36-44cd-9904-b37389ce2bfa
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2016-11-04T12:01:02Z, Id: cc7826ac-5355-471d-9bc6-a54673d06e45
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2017-08-17T12:00:00Z, Id: a1f8db59-40ff-4f0e-b011-37629c3a1a53
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2018-04-16T11:55:00Z, Id: c7a69da3-27de-4a4b-ab75-b6716f6321e5
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2018-09-21T15:18:43Z, Id: da494a53-0dad-4158-b15f-8f9daca7a412
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2018-10-19T11:28:54Z, Id: 84ec130b-d047-44bf-a46d-58c1ac292ca7
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2018-11-26T07:59:09Z, Id: ee5c100f-152f-4ae5-9e9d-014af3c01c56
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2018-11-26T09:21:55Z, Id: d04959a6-71da-4913-9997-836793e3c115
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2019-01-11T10:04:19Z, Id: 488e5f23-8bc5-46f8-9ad8-ea9a49a8efda
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2019-02-18T14:37:57Z, Id: 0207b3e6-92a8-4363-8c0e-361114cdd719
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2019-03-03T17:34:10Z, Id: 198d9b79-2950-4609-b6ec-f52254074a05
2020/10/30 21:54:21 [Fatal] Please run this tool again and assign --modelId '<one above base model id>'. If no model id listed above, it means currently there is no available base model for en-us
```

#### <a name="custom-pronunciation-on-the-custom-speech-to-text-container"></a>Пользовательское произношение по пользовательскому контейнеру преобразования речи в текст 
Начиная с версии v 2.5.0 контейнера пользовательского преобразования "речь-к тексту" вы можете получить результат выполнения пользовательской произношения. Все, что вам нужно сделать, — это настроить пользовательские правила произношения в пользовательской модели и подключить модель к контейнеру пользовательских речевых преобразований в текст.


# <a name="text-to-speech"></a>[Преобразование текста в речь](#tab/tts)

Чтобы запустить стандартный контейнер преобразования *текста в речь* , выполните следующую `docker run` команду.

```bash
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
mcr.microsoft.com/azure-cognitive-services/speechservices/text-to-speech \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Команда:

* Выполняет стандартный контейнер преобразования *текста в речь* из образа контейнера.
* Выделяется 1 ядро ЦП и 2 гигабайта (ГБ) памяти.
* предоставляет TCP-порт 5000 и выделяет псевдотелетайп для контейнера;
* автоматически удаляет контейнер после завершения его работы. Образ контейнера остается доступным на главном компьютере.

# <a name="neural-text-to-speech"></a>[нейронное преобразование текста в речь;](#tab/ntts)

Чтобы запустить контейнер *нейронного текста в речь* , выполните следующую `docker run` команду.

```bash
docker run --rm -it -p 5000:5000 --memory 12g --cpus 6 \
mcr.microsoft.com/azure-cognitive-services/speechservices/neural-text-to-speech \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Команда:

* Запускает контейнер *нейронного текста в речь* из образа контейнера.
* Выделяет 6 ядер ЦП и 12 гигабайт (ГБ) памяти.
* предоставляет TCP-порт 5000 и выделяет псевдотелетайп для контейнера;
* автоматически удаляет контейнер после завершения его работы. Образ контейнера остается доступным на главном компьютере.

# <a name="custom-text-to-speech"></a>[Пользовательский текст в речь](#tab/ctts)

Пользовательский контейнер преобразования *текста в речь* использует пользовательскую голосовую модель. Пользовательская модель должна быть [обучена](how-to-custom-voice-create-voice.md) с помощью [пользовательского Voice](https://aka.ms/custom-voice-portal). Для запуска контейнера требуется идентификатор настраиваемой **модели** голоса. Его можно найти на странице **обучение** на настраиваемом голосовом портале. На портале настраиваемого голоса перейдите на страницу **обучения** и выберите модель.
<br>

![Страница настраиваемого речевого обучения](media/custom-voice/custom-voice-model-training.png)

Получите **идентификатор модели** , который будет использоваться в качестве аргумента `ModelId` параметра команды DOCKER Run.
<br>

![Сведения о настраиваемой модели голоса](media/custom-voice/custom-voice-model-details.png)

В следующей таблице представлены различные `docker run` Параметры и соответствующие им описания.

| Параметр | Описание |
|---------|---------|
| `{VOLUME_MOUNT}` | Узел [тома](https://docs.docker.com/storage/volumes/)главного компьютера, который DOCKER использует для сохранения настраиваемой модели. Например, *к:\кустомспич* , где *диск C* находится на хост-компьютере. |
| `{MODEL_ID}` | **Идентификатор модели** пользовательское распознавание речи на странице " **обучение** " настраиваемого голоса Portal. |
| `{ENDPOINT_URI}` | Для оценки и выставления счетов требуется конечная точка. Дополнительные сведения см. в разделе [сбор обязательных параметров](#gathering-required-parameters). |
| `{API_KEY}` | Требуется ключ API. Дополнительные сведения см. в разделе [сбор обязательных параметров](#gathering-required-parameters). |

Чтобы запустить пользовательский контейнер преобразования *текста в речь* , выполните следующую `docker run` команду:

```bash
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
-v {VOLUME_MOUNT}:/usr/local/models \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-text-to-speech \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Команда:

* Выполняет пользовательский контейнер преобразования *текста в речь* из образа контейнера.
* Выделяется 1 ядро ЦП и 2 гигабайта (ГБ) памяти.
* Загружает *настраиваемую модель преобразования текста в речь* из подключения ввода тома, например *к:\кустомвоице*.
* предоставляет TCP-порт 5000 и выделяет псевдотелетайп для контейнера;
* Загружает модель, заданную `ModelId` (если она не найдена в подключении тома).
* Если пользовательская модель была ранее скачана, параметр `ModelId` игнорируется.
* автоматически удаляет контейнер после завершения его работы. Образ контейнера остается доступным на главном компьютере.

# <a name="speech-language-detection"></a>[Речь распознавание языка](#tab/lid)

Чтобы запустить контейнер *распознавание языка речи* , выполните следующую `docker run` команду.

```bash
docker run --rm -it -p 5003:5003 --memory 1g --cpus 1 \
mcr.microsoft.com/azure-cognitive-services/speechservices/language-detection \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Команда: 

* Запускает контейнер обнаружения языка распознавания речи из образа контейнера. В настоящее время вам не придется начисляться на выполнение этого образа.
* Выделяет 1 ядро ЦП и 1 гигабайт (ГБ) памяти.
* Предоставляет TCP-порт 5003 и выделяет псевдо-TTY для контейнера.
* автоматически удаляет контейнер после завершения его работы. Образ контейнера остается доступным на главном компьютере.

Если вы отправляете только голосовые распознавание языка запросы, необходимо задать для речевого клиента `phraseDetection` значение `None` .  

```python
speech_config.set_service_property(
      name='speechcontext-phraseDetection.Mode',
      value='None',
      channel=speechsdk.ServicePropertyChannel.UriQueryParameter
   )
```

Если вы хотите запустить этот контейнер с контейнером преобразования речи в текст, можно использовать этот [образ DOCKER](https://hub.docker.com/r/antsu/on-prem-client). После запуска обоих контейнеров используйте команду DOCKER Run для выполнения `speech-to-text-with-languagedetection-client` .

```Docker
docker run --rm -v ${HOME}:/root -ti antsu/on-prem-client:latest ./speech-to-text-with-languagedetection-client ./audio/LanguageDetection_en-us.wav --host localhost --lport 5003 --sport 5000
```

> [!NOTE]
> Увеличение числа одновременных вызовов может повлиять на надежность и задержку. Для определения языка рекомендуется не более 4 одновременных вызовов, использующих 1 ЦП, а также память объемом ОЗУ. Для узлов с двумя процессорами и 2 ГБ памяти рекомендуется не более 6 одновременных вызовов.

***

> [!IMPORTANT]
> Для запуска контейнера необходимо указать параметры `Eula`, `Billing` и `ApiKey`. В противном случае контейнер не запустится.  Дополнительные сведения см. в [разделе о выставлении счетов](#billing).

## <a name="query-the-containers-prediction-endpoint"></a>Запрос конечной точки прогнозирования контейнера

> [!NOTE]
> Если вы используете несколько контейнеров, используйте уникальный номер порта.

| Контейнеры | URL-адрес узла SDK | Протокол |
|--|--|--|
| Стандартные преобразования речи в текст и Пользовательское распознавание речи в текст | `ws://localhost:5000` | WS |
| Преобразование текста в речь (включая стандартные, пользовательские и нейронные), определение языка распознавания речи | `http://localhost:5000` | HTTP |

Дополнительные сведения об использовании протоколов WSS и HTTPS см. в разделе [Безопасность контейнера](../cognitive-services-container-support.md#azure-cognitive-services-container-security).

### <a name="speech-to-text-standard-and-custom"></a>Преобразование речи в текст (стандартные и пользовательские)

[!INCLUDE [Query Speech-to-text container endpoint](includes/speech-to-text-container-query-endpoint.md)]

#### <a name="analyze-sentiment"></a>Анализ тональности

Если вы указали учетные данные API анализа текста в [контейнере](#analyze-sentiment-on-the-speech-to-text-output), вы можете использовать РЕЧЕВОЙ пакет SDK для отправки запросов распознавания речи с помощью анализа тональности. Можно настроить Ответы API для использования *простого* или *подробного* формата.
> [!NOTE]
> v 1.13 пакета SDK Python для службы распознавания речи обнаружил определенную ошибку при анализе тональности. Используйте v 1.12. x или более раннюю версию, если вы используете тональности Analysis в пакете SDK Python для службы распознавания речи.

# <a name="simple-format"></a>[Простой формат](#tab/simple-format)

Чтобы настроить речевой клиент для использования простого формата, добавьте `"Sentiment"` в качестве значения `Simple.Extensions` . Если вы хотите выбрать конкретную версию модели Анализ текста, замените `'latest'` в `speechcontext-phraseDetection.sentimentAnalysis.modelversion` конфигурации свойства.

```python
speech_config.set_service_property(
    name='speechcontext-PhraseOutput.Simple.Extensions',
    value='["Sentiment"]',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
speech_config.set_service_property(
    name='speechcontext-phraseDetection.sentimentAnalysis.modelversion',
    value='latest',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
```

`Simple.Extensions` вернет результат тональности в корневом слое ответа.

```json
{
   "DisplayText":"What's the weather like?",
   "Duration":13000000,
   "Id":"6098574b79434bd4849fee7e0a50f22e",
   "Offset":4700000,
   "RecognitionStatus":"Success",
   "Sentiment":{
      "Negative":0.03,
      "Neutral":0.79,
      "Positive":0.18
   }
}
```

# <a name="detailed-format"></a>[Подробный формат](#tab/detailed-format)

Чтобы настроить речевой клиент для использования подробного формата, добавьте `"Sentiment"` в качестве значения для параметров `Detailed.Extensions` , `Detailed.Options` или. Если вы хотите выбрать конкретную версию модели Анализ текста, замените `'latest'` в `speechcontext-phraseDetection.sentimentAnalysis.modelversion` конфигурации свойства.

```python
speech_config.set_service_property(
    name='speechcontext-PhraseOutput.Detailed.Options',
    value='["Sentiment"]',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
speech_config.set_service_property(
    name='speechcontext-PhraseOutput.Detailed.Extensions',
    value='["Sentiment"]',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
speech_config.set_service_property(
    name='speechcontext-phraseDetection.sentimentAnalysis.modelversion',
    value='latest',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
```

`Detailed.Extensions` предоставляет тональности результат в корневом слое ответа. `Detailed.Options` предоставляет результат в `NBest` слое ответа. Их можно использовать отдельно или вместе.

```json
{
   "DisplayText":"What's the weather like?",
   "Duration":13000000,
   "Id":"6a2aac009b9743d8a47794f3e81f7963",
   "NBest":[
      {
         "Confidence":0.973695,
         "Display":"What's the weather like?",
         "ITN":"what's the weather like",
         "Lexical":"what's the weather like",
         "MaskedITN":"What's the weather like",
         "Sentiment":{
            "Negative":0.03,
            "Neutral":0.79,
            "Positive":0.18
         }
      },
      {
         "Confidence":0.9164971,
         "Display":"What is the weather like?",
         "ITN":"what is the weather like",
         "Lexical":"what is the weather like",
         "MaskedITN":"What is the weather like",
         "Sentiment":{
            "Negative":0.02,
            "Neutral":0.88,
            "Positive":0.1
         }
      }
   ],
   "Offset":4700000,
   "RecognitionStatus":"Success",
   "Sentiment":{
      "Negative":0.03,
      "Neutral":0.79,
      "Positive":0.18
   }
}
```

---

Если вы хотите полностью отключить анализ тональности, добавьте `false` значение в `sentimentanalysis.enabled` .

```python
speech_config.set_service_property(
    name='speechcontext-phraseDetection.sentimentanalysis.enabled',
    value='false',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
```

### <a name="text-to-speech-standard-neural-and-custom"></a>Преобразование текста в речь (стандартный, нейронный и пользовательский)

[!INCLUDE [Query Text-to-speech container endpoint](includes/text-to-speech-container-query-endpoint.md)]

### <a name="run-multiple-containers-on-the-same-host"></a>Запуск нескольких контейнеров на одном узле

Если вы планируете запускать несколько контейнеров при открытых портах, обязательно назначьте каждому контейнеру отдельный открытый порт. Например, запускайте первый контейнер на порте 5000, а второй — на порте 5001.

Этот контейнер и другой контейнер Azure Cognitive Services могут одновременно работать на одном и том же узле. Также одному запущенному контейнеру Cognitive Services могут соответствовать сразу несколько контейнеров.

[!INCLUDE [Validate container is running - Container's API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]

## <a name="stop-the-container"></a>Остановка контейнера

[!INCLUDE [How to stop the container](../../../includes/cognitive-services-containers-stop.md)]

## <a name="troubleshooting"></a>Устранение неполадок

При запуске или запуске контейнера могут возникнуть проблемы. Используйте [Подключение](speech-container-configuration.md#mount-settings) к выходным данным и включите ведение журнала. Это позволит контейнеру создавать файлы журналов, которые полезны при устранении неполадок.

[!INCLUDE [Cognitive Services FAQ note](../containers/includes/cognitive-services-faq-note.md)]

## <a name="billing"></a>Выставление счетов

Контейнеры распознавания речи отправляют сведения о выставлении счетов в Azure с помощью *речевого* ресурса в учетной записи Azure.

[!INCLUDE [Container's Billing Settings](../../../includes/cognitive-services-containers-how-to-billing-info.md)]

Дополнительные сведения об этих параметрах см. в статье [Настройка контейнеров](speech-container-configuration.md).

## <a name="summary"></a>Итоги

В этой статье вы узнали основные понятия и рабочий процесс по скачиванию, установке и запуску речевых контейнеров. В разделе "Сводка" сделайте следующее.

* Речь предоставляет четыре контейнера Linux для DOCKER, включая различные возможности:
  * *Преобразование речи в текст*
  * *пользовательское преобразование речи в текст;*
  * *Преобразование текста в речь*
  * *Пользовательский текст в речь*
  * *нейронное преобразование текста в речь;*
  * *Речь распознавание языка*
* Образы контейнеров загружаются из реестра контейнеров в Azure.
* Образы контейнеров выполняются в Docker.
* Независимо от того, используете ли вы REST API (только преобразование текста в речь) или пакет SDK (преобразование речи в текст или преобразование текста в речь), укажите URI узла контейнера. 
* При создании экземпляра контейнера необходимо указать сведения о выставлении счетов.

> [!IMPORTANT]
>  Контейнеры Cognitive Services не лицензируются для запуска без подключения к Azure для отслеживания использования. Клиенты должны разрешить контейнерам непрерывную передачу данных для выставления счетов в службу контроля потребления. Контейнеры Cognitive Services не отправляют в корпорацию Майкрософт данные клиента (например, анализируемые изображения или тексты).

## <a name="next-steps"></a>Дальнейшие действия

* Проверьте настройки [контейнеров](speech-container-configuration.md) на наличие параметров конфигурации.
* Узнайте, как [использовать контейнеры службы речи с Kubernetes и Helm](speech-container-howto-on-premises.md)
* Использование большего числа [контейнеров Cognitive Services](../cognitive-services-container-support.md)
