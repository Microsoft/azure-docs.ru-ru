---
title: Когнитивный навык распознавания языка
titleSuffix: Azure Cognitive Search
description: Вычисляет неструктурированный текст и для каждой записи возвращает идентификатор языка с показателем, указывающим стойкость анализа в конвейере обогащения искусственного интеллекта в Azure Когнитивный поиск.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/17/2020
ms.openlocfilehash: 078a9312a7ee1b3b0eafd000928ed74348a540c3
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102548059"
---
#   <a name="language-detection-cognitive-skill"></a>Когнитивный навык распознавания языка

**Распознавание языка** навык определяет язык вводимого текста и сообщает один код языка для каждого документа, отправленного по запросу. Код языка сопряжен с оценкой, указывающей степень анализа. Этот навык использует модели машинного обучения, предоставляемые функцией [Анализ текста](../cognitive-services/text-analytics/overview.md) в Cognitive Services.

Эта возможность особенно полезна, когда требуется предоставить язык текста в качестве входных данных для других навыков (например, [навыка анализа тональности](cognitive-search-skill-sentiment.md) или [разделения текста](cognitive-search-skill-textsplit.md)).

Определение языка использует библиотеки обработки естественного языка Bing, что превышает число [поддерживаемых языков и регионов](../cognitive-services/text-analytics/language-support.md) , перечисленных для анализ текста. Точный список языков не публикуется, но включает в себя все широко знакомые языки, а также варианты, диалекты и некоторые региональные и культурные языки. Если содержимое представлено на менее часто используемом языке, можно [попробовать распознавание языка API](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/Languages) , чтобы узнать, возвращает ли он код. Для языков, которые не удалась распознать, возвращается ответ `(Unknown)`.

> [!NOTE]
> По мере расширения области путем увеличения частоты обработки и добавления большего количества документов или дополнительных алгоритмов ИИ, вам нужно будет [присоединить оплачиваемый ресурс Cognitive Services](cognitive-search-attach-cognitive-services.md). Плата взимается при вызове API в Cognitive Services и извлечении изображений при распознавании документов в службе "Когнитивный поиск Azure". За извлечение текста из документов плата не взимается.
>
> Плата за выполнение встроенных навыков взимается в рамках существующей [модели оплаты Cognitive Services по мере использования](https://azure.microsoft.com/pricing/details/cognitive-services/). Плата за извлечение изображений указана на [странице с ценами на службу "Когнитивный поиск Azure"](https://azure.microsoft.com/pricing/details/search/).


## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Text.LanguageDetectionSkill

## <a name="data-limits"></a>Ограничения данных
Максимальный размер записи — 50 000 знаков, как определено в [`String.Length`](/dotnet/api/system.string.length). Если необходимо разбить данные перед отправкой в навык определения языка, вы можете использовать [навык разбиения текста](cognitive-search-skill-textsplit.md).

## <a name="skill-parameters"></a>Параметры навыков

Параметры зависят от регистра.

| Входные данные | Описание |
|---------------------|-------------|
| `defaultCountryHint` | Используемых Код страны ISO 3166-1 Alpha-2 2 можно использовать в качестве подсказки для модели определения языка, если не удается устранить неоднозначность языка. Дополнительные сведения см. [в анализ текста документации](../cognitive-services/text-analytics/how-tos/text-analytics-how-to-language-detection.md#ambiguous-content) по этому разделу. В частности, `defaultCountryHint` параметр используется с документами, которые не указывают `countryHint` входные данные явным образом.  |
| `modelVersion`   | Используемых Версия модели, используемая при вызове службы Анализ текста. По умолчанию будет использоваться последняя доступная версия, если она не указана. Мы рекомендуем не указывать это значение, если это не обязательно. Дополнительные сведения см. [в разделе Управление версиями модели в API анализа текста](../cognitive-services/text-analytics/concepts/model-versioning.md) . |

## <a name="skill-inputs"></a>Входные данные навыков

Параметры зависят от регистра.

| Входные данные     | Описание |
|--------------------|-------------|
| `text` | Анализируемый текст.|
| `countryHint` | Код страны ISO 3166-1 Alpha-2 2, используемый в качестве подсказки для модели определения языка, если не может устранить неоднозначность языка. Дополнительные сведения см. [в анализ текста документации](../cognitive-services/text-analytics/how-tos/text-analytics-how-to-language-detection.md#ambiguous-content) по этому разделу. |

## <a name="skill-outputs"></a>Выходные данные навыка

| Имя выходных данных    | Описание |
|--------------------|-------------|
| `languageCode` | Код языка ISO 6391 для распознанного языка. Например, en. |
| `languageName` | Имя языка. Например, английский. |
| `score` | Значение от 0 до 1. Вероятность, что язык правильно распознан. Оценка может быть меньше 1, если в предложении используются разные языки.  |

##  <a name="sample-definition"></a>Пример определения

```json
 {
    "@odata.type": "#Microsoft.Skills.Text.LanguageDetectionSkill",
    "inputs": [
      {
        "name": "text",
        "source": "/document/text"
      },
      {
        "name": "countryHint",
        "source": "/document/countryHint"
      }
    ],
    "outputs": [
      {
        "name": "languageCode",
        "targetName": "myLanguageCode"
      },
      {
        "name": "languageName",
        "targetName": "myLanguageName"
      },
      {
        "name": "score",
        "targetName": "myLanguageScore"
      }

    ]
  }
```

##  <a name="sample-input"></a>Пример ввода

```json
{
    "values": [
      {
        "recordId": "1",
        "data":
           {
             "text": "Glaciers are huge rivers of ice that ooze their way over land, powered by gravity and their own sheer weight. "
           }
      },
      {
        "recordId": "2",
        "data":
           {
             "text": "Estamos muy felices de estar con ustedes."
           }
      },
      {
        "recordId": "3",
        "data":
           {
             "text": "impossible",
             "countryHint": "fr"
           }
      }
    ]
```


##  <a name="sample-output"></a>Пример выходных данных

```json
{
    "values": [
      {
        "recordId": "1",
        "data":
            {
              "languageCode": "en",
              "languageName": "English",
              "score": 1,
            }
      },
      {
        "recordId": "2",
        "data":
            {
              "languageCode": "es",
              "languageName": "Spanish",
              "score": 1,
            }
      },
      {
        "recordId": "3",
        "data":
            {
              "languageCode": "fr",
              "languageName": "French",
              "score": 1,
            }
      }
    ]
}
```

## <a name="see-also"></a>См. также

+ [Встроенные навыки](cognitive-search-predefined-skills.md)
+ [Определение набора навыков](cognitive-search-defining-skillset.md)