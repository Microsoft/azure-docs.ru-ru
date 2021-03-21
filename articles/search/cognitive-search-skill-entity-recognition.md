---
title: Когнитивный навык распознавания сущностей
titleSuffix: Azure Cognitive Search
description: Извлечение различных типов сущностей из текста в конвейере обогащения в Azure Когнитивный поиск.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/17/2020
ms.openlocfilehash: 3300d68c6356cb7cdd56a9ad79eb0ac011942eb2
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102547906"
---
#    <a name="entity-recognition-cognitive-skill"></a>Когнитивный навык распознавания сущностей

Навык **распознавания сущностей** извлекает сущности различных типов из текста. Этот навык использует модели машинного обучения, предоставляемые функцией [Анализ текста](../cognitive-services/text-analytics/overview.md) в Cognitive Services.

> [!NOTE]
> По мере расширения области путем увеличения частоты обработки и добавления большего количества документов или дополнительных алгоритмов ИИ, вам нужно будет [присоединить оплачиваемый ресурс Cognitive Services](cognitive-search-attach-cognitive-services.md). Плата взимается при вызове API в Cognitive Services и извлечении изображений при распознавании документов в службе "Когнитивный поиск Azure". За извлечение текста из документов плата не взимается.
>
> Плата за выполнение встроенных навыков взимается в рамках существующей [модели оплаты Cognitive Services по мере использования](https://azure.microsoft.com/pricing/details/cognitive-services/). Плата за извлечение изображений указана на [странице с ценами на службу "Когнитивный поиск Azure"](https://azure.microsoft.com/pricing/details/search/).


## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Text.EntityRecognitionSkill

## <a name="data-limits"></a>Ограничения данных
Максимальный размер записи — 50 000 знаков, как определено в [`String.Length`](/dotnet/api/system.string.length). Если вам нужно разбить данные перед отправкой для извлечения ключевой фразы, можно воспользоваться [навыком разделения текста](cognitive-search-skill-textsplit.md).

## <a name="skill-parameters"></a>Параметры навыков

Все параметры чувствительны к регистру и являются необязательными.

| Имя параметра     | Описание |
|--------------------|-------------|
| `categories`    | Массив категорий, который следует извлекать.  Возможные типы категорий: `"Person"`, `"Location"`, `"Organization"`, `"Quantity"`, `"Datetime"`, `"URL"`, `"Email"`. Если категория не указана, возвращаются все типы.|
| `defaultLanguageCode` |    Код языка вводимого текста. Поддерживаются следующие языки: `ar, cs, da, de, en, es, fi, fr, hu, it, ja, ko, nl, no, pl, pt-BR, pt-PT, ru, sv, tr, zh-hans` . Не все категории сущностей поддерживаются для всех языков. см. Примечание ниже.|
| `minimumPrecision` | Значение от 0 до 1. Если показатель достоверности (в `namedEntities` выходных данных) меньше этого значения, сущность не возвращается. Значение по умолчанию равно 0. |
| `includeTypelessEntities` | Задайте значение, `true` Если нужно распознать хорошо известные сущности, не соответствующие текущим категориям. Распознанные сущности возвращаются в `entities` составном поле вывода. Например, "Windows 10" является известной сущностью (продуктом), но так как "продукты" не являются поддерживаемой категорией, эта сущность будет включена в поле выходных данных сущностей. Значение по умолчанию — `false`. |


## <a name="skill-inputs"></a>Входные данные навыков

| Ввод имени      | Описание                   |
|---------------|-------------------------------|
| `languageCode`    | Необязательный параметр. По умолчанию — `"en"`.  |
| `text`          | Текст для анализа.          |

## <a name="skill-outputs"></a>Выходные данные навыка

> [!NOTE]
> Для некоторых языков категории сущностей не поддерживаются. `"Person"` `"Location"` `"Organization"` Типы категорий сущностей, и поддерживаются для полного списка языков, описанных выше. При извлечении типов, , и поддерживаются только de, EN, ES, _fr_ и _zh-Ханс_ `"Quantity"` `"Datetime"` `"URL"` `"Email"` . Дополнительные сведения см. [в разделе Поддержка языков и регионов для API анализа текста](../cognitive-services/text-analytics/language-support.md).  

| Имя вывода      | Описание                   |
|---------------|-------------------------------|
| `persons`       | Массив строк, где каждая строка представляет имя человека. |
| `locations`  | Массив строк, где каждая строка представляет расположение. |
| `organizations`  | Массив строк, где каждая строка представляет организацию. |
| `quantities`  | Массив строк, где каждая строка представляет количество. |
| `dateTimes`  | Массив строк, где каждая строка представляет собой значение даты и времени (как оно отображается в тексте). |
| `urls` | Массив строк, где каждая строка представляет собой URL-адрес. |
| `emails` | Массив строк, где каждая строка представляет собой адрес электронной почты. |
| `namedEntities` | Массив сложных типов, содержащий следующие поля: <ul><li>категория</li> <li>значение (фактическое имя сущности)</li><li>смещение (расположение, где оно было найдено в тексте);</li><li>достоверность (более высокое значение означает, что она больше, чем фактическая сущность)</li></ul> |
| `entities` | Массив сложных типов, содержащий подробные сведения о сущностях, извлеченных из текста, со следующими полями. <ul><li> name (Фактическое имя сущности. Представляет собой "нормализованную" форму);</li><li> wikipediaId;</li><li>wikipediaLanguage;</li><li>wikipediaUrl (ссылка на страницу Википедии для сущности);</li><li>bingId</li><li>type (категория распознанной сущности);</li><li>subType (доступно только для определенных категорий, что дает дополнительное представление о типе сущности);</li><li> matches (имеющаяся составная коллекция);<ul><li>text (необработанный текст для сущности);</li><li>offset (расположение, где было найдено смещение);</li><li>length (длина необработанного текста сущности).</li></ul></li></ul> |

##    <a name="sample-definition"></a>Пример определения

```json
  {
    "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
    "categories": [ "Person", "Email"],
    "defaultLanguageCode": "en",
    "includeTypelessEntities": true,
    "minimumPrecision": 0.5,
    "inputs": [
      {
        "name": "text",
        "source": "/document/content"
      }
    ],
    "outputs": [
      {
        "name": "persons",
        "targetName": "people"
      },
      {
        "name": "emails",
        "targetName": "contact"
      },
      {
        "name": "entities"
      }
    ]
  }
```
##    <a name="sample-input"></a>Пример ввода

```json
{
    "values": [
      {
        "recordId": "1",
        "data":
           {
             "text": "Contoso corporation was founded by John Smith. They can be reached at contact@contoso.com",
             "languageCode": "en"
           }
      }
    ]
}
```

##    <a name="sample-output"></a>Пример выходных данных

```json
{
  "values": [
    {
      "recordId": "1",
      "data" : 
      {
        "persons": [ "John Smith"],
        "emails":["contact@contoso.com"],
        "namedEntities": 
        [
          {
            "category":"Person",
            "value": "John Smith",
            "offset": 35,
            "confidence": 0.98
          }
        ],
        "entities":  
        [
          {
            "name":"John Smith",
            "wikipediaId": null,
            "wikipediaLanguage": null,
            "wikipediaUrl": null,
            "bingId": null,
            "type": "Person",
            "subType": null,
            "matches": [{
                "text": "John Smith",
                "offset": 35,
                "length": 10
            }]
          },
          {
            "name": "contact@contoso.com",
            "wikipediaId": null,
            "wikipediaLanguage": null,
            "wikipediaUrl": null,
            "bingId": null,
            "type": "Email",
            "subType": null,
            "matches": [
            {
                "text": "contact@contoso.com",
                "offset": 70,
                "length": 19
            }]
          },
          {
            "name": "Contoso",
            "wikipediaId": "Contoso",
            "wikipediaLanguage": "en",
            "wikipediaUrl": "https://en.wikipedia.org/wiki/Contoso",
            "bingId": "349f014e-7a37-e619-0374-787ebb288113",
            "type": null,
            "subType": null,
            "matches": [
            {
                "text": "Contoso",
                "offset": 0,
                "length": 7
            }]
          }
        ]
      }
    }
  ]
}
```

Обратите внимание, что смещения, возвращаемые для сущностей в выходных данных этого навыка, возвращаются непосредственно из [API анализа текста](../cognitive-services/text-analytics/overview.md). Это означает, что если вы используете их для индексации в исходной строке, следует использовать класс [StringInfo](/dotnet/api/system.globalization.stringinfo) в .NET для извлечения правильного содержимого.  [Дополнительные сведения можно найти здесь.](../cognitive-services/text-analytics/concepts/text-offsets.md)

## <a name="warning-cases"></a>Случаи предупреждений
Если код языка для документа не поддерживается, возвращается предупреждение, и сущности не извлекаются.

## <a name="see-also"></a>См. также раздел

+ [Встроенные навыки](cognitive-search-predefined-skills.md)
+ [Определение набора навыков](cognitive-search-defining-skillset.md)