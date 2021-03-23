---
title: Поддержка языков — LUIS
titleSuffix: Azure Cognitive Services
description: Служба LUIS имеет широкий набор функций. Не все компоненты имеют одинаковую языковую функциональную совместимость. Убедитесь, что функции, которые вас интересуют, поддерживаются для используемого языка и региональных параметров. Приложение LUIS учитывает конкретный язык и региональные параметры, которые нельзя изменять после установки.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 12/09/2019
ms.openlocfilehash: 724b94a44ae289063bbb1c6ebf53b38c0a617b88
ms.sourcegitcommit: ba3a4d58a17021a922f763095ddc3cf768b11336
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104799333"
---
# <a name="language-and-region-support-for-luis"></a>Поддержка языков и регионов в LUIS

Служба LUIS имеет широкий набор функций. Не все компоненты имеют одинаковую языковую функциональную совместимость. Убедитесь, что функции, которые вас интересуют, поддерживаются для используемого языка и региональных параметров. Приложение LUIS учитывает конкретный язык и региональные параметры, которые нельзя изменять после установки.

## <a name="multi-language-luis-apps"></a>Многоязыковые приложения LUIS

Если вам требуется многоязыковое клиентское приложение LUIS, такое как чат-бот, у вас есть несколько вариантов. Если LUIS поддерживает все языки, можно разработать приложение LUIS для каждого языка. Каждое приложение LUIS получает уникальный идентификатор приложения и журнал конечной точки. Если необходимо предоставить сведения о языке для языка, LUIS не поддерживает, можно использовать [службу переводчиков](../Translator/translator-info-overview.md) , чтобы перевести utterance на поддерживаемый язык, отправить utterance в конечную точку Luis и получить полученные результаты.

## <a name="languages-supported"></a>Поддерживаемые языки

Служба LUIS распознает фрагменты речи на следующих языках:

| Язык |Locale  |  Предварительно созданная предметная область | Предварительно созданная сущность | Рекомендации по списку фраз | **[Текстовая аналитика](../text-analytics/language-support.md)<br>(Тональность и<br>ключевые слова)|
|--|--|:--:|:--:|:--:|:--:|
| Арабский (Предварительная версия — современные стандартные арабские) |`ar-AR`|-|-|-|-|
| *[китайский](#chinese-support-notes) |`zh-CN` | ✔ | ✔ |✔|-|
| Нидерландский |`nl-NL` |✔|-|-|✔|
| Английский (США) |`en-US` | ✔ | ✔  |✔|✔|
| Французский (Канада) |`fr-CA` |-|-|-|✔|
| Французский (Франция) |`fr-FR` |✔| ✔ |✔ |✔|
| Немецкий |`de-DE` |✔| ✔ |✔ |✔|
| Гуджарати | `gu-IN`|-|-|-|-|
| Hindi | `hi-IN`|-|✔|-|-|
| Итальянский |`it-IT` |✔| ✔ |✔|✔|
| *[японский](#japanese-support-notes) |`ja-JP` |✔| ✔ |✔|Только ключевая фраза|
| Корейский |`ko-KR` |✔|-|-|Только ключевая фраза|
| Маратхи | `mr-IN`|-|-|-|-|
| Португальский (Бразилия) |`pt-BR` |✔| ✔ |✔ |не все вложенные языки и региональные параметры|
| Испанский (Мексика)|`es-MX` |-|✔|✔|✔|
| испанский (Испания) |`es-ES` |✔| ✔ |✔|✔|
| Тамильский | `ta-IN`|-|-|-|-|
| Телугу | `te-IN`|-|-|-|-|
| Турецкий | `tr-TR` |✔|✔|-|Только тональность|




Языковая поддержка зависит от [готовых сущностей](luis-reference-prebuilt-entities.md) и [готовых предметных областей](luis-reference-prebuilt-domains.md).

[!INCLUDE [Chinese language support notes](includes/chinese-language-support-notes.md)]

### <a name="japanese-support-notes"></a>* Примечания о поддержке японского языка

 - Поскольку служба LUIS не обеспечивает синтаксический анализ и не может понять разницу между почтительной и неформальной речью, необходимо включить различные уровни формальности в качестве примеров обучения для ваших приложений.
     - でございます не равно です.
     - です не равно だ.

[!INCLUDE [Text Analytics support notes](includes/text-analytics-support-notes.md)]

### <a name="speech-api-supported-languages"></a>Поддерживаемые языки API речи
Сведения о языках режима диктовки службы речи см. в разделе [Поддерживаемые языки](../speech-service/speech-to-text.md) для службы "Речь".

### <a name="bing-spell-check-supported-languages"></a>Поддерживаемые языки проверки орфографии Bing
Список поддерживаемых языков и состояний см. в разделе [Поддерживаемые языки](../bing-spell-check/language-support.md) проверки орфографии Bing.

## <a name="rare-or-foreign-words-in-an-application"></a>Редкие или иностранные слова в приложении
В языке и региональных параметрах `en-us` служба LUIS обучается для определения большинства английских слов, включая сленг. В языке и региональных параметрах `zh-cn` служба LUIS обучается для определения большинства символов китайского языка. Если при использовании редких слов в `en-us` или символов в `zh-cn` вы понимаете, что служба LUIS не может распознать это слово или символ, их можно добавить в [список фраз](luis-how-to-add-features.md). Например, слова за пределами языка и региональных параметров приложения, то есть иностранные слова, следует добавить в свойство "Список фраз".

<!--This phrase list should be marked non-interchangeable, to indicate that the set of rare words forms a class that LUIS should learn to recognize, but they are not synonyms or interchangeable with each other.-->

### <a name="hybrid-languages"></a>Гибридные языки
Гибридные языки объединяют слова из двух культур, например английского и китайского языка. Эти языки не поддерживаются в LUIS, так как приложение основано на одном языке.

## <a name="tokenization"></a>Выделение лексем
В целях машинного обучения LUIS разбивает фрагменты речи на [маркеры](luis-glossary.md#token), в зависимости от языка и региональных параметров.

|Язык|  каждый пробел и специальный символ | уровень символа|составные слова
|--|:--:|:--:|:--:|
|Арабский|✔|||
|Китайский||✔||
|Нидерландский|✔||✔|
|Английский (en-us)|✔ |||
|Французский (fr-FR)|✔|||
|Французский (fr-CA)|✔|||
|Немецкий|✔||✔|
|Гуджарати|✔|||
|Hindi|✔|||
|Итальянский|✔|||
|Японский|||✔
|Корейский||✔||
|Маратхи|✔|||
|Португальский (Бразилия)|✔|||
|Испанский (es-ES)|✔|||
|Испанский (es-MX)|✔|||
|Тамильский|✔|||
|Телугу|✔|||
|Турецкий|✔|||


### <a name="custom-tokenizer-versions"></a>Пользовательские версии анализатора маркеров

Следующие языки и региональные параметры имеют пользовательские версии маркеров:

|culture|Версия|Назначение|
|--|--|--|
|Немецкий<br>`de-de`|1.0.0|Разделяет слова, разделив их с помощью токена на основе машинного обучения, который пытается разбить составные слова на отдельные компоненты.<br>Если пользователь введет в `Ich fahre einen krankenwagen` качестве utterance, он будет включен `Ich fahre einen kranken wagen` . Разрешение пометки и независимо от того, `kranken` `wagen` как разные сущности.|
|Немецкий<br>`de-de`|1.0.2|Разделяет слова, разбивая их на пробелы.<br> Если пользователь вводит `Ich fahre einen krankenwagen` в качестве utterance, он остается единственным маркером. Таким образом `krankenwagen` , помечается как единая сущность. |
|Нидерландский<br>`nl-nl`|1.0.0|Разделяет слова, разделив их с помощью токена на основе машинного обучения, который пытается разбить составные слова на отдельные компоненты.<br>Если пользователь введет в `Ik ga naar de kleuterschool` качестве utterance, он будет включен `Ik ga naar de kleuter school` . Разрешение пометки и независимо от того, `kleuter` `school` как разные сущности.|
|Нидерландский<br>`nl-nl`|1.0.1|Разделяет слова, разбивая их на пробелы.<br> Если пользователь вводит `Ik ga naar de kleuterschool` в качестве utterance, он остается единственным маркером. Таким образом `kleuterschool` , помечается как единая сущность. |


### <a name="migrating-between-tokenizer-versions"></a>Миграция между версиями маркеров
<!--
Your first choice is to change the tokenizer version in the app file, then import the version. This action changes how the utterances are tokenized but allows you to keep the same app ID.

Tokenizer JSON for 1.0.0. Notice the property value for  `tokenizerVersion`.

```JSON
{
    "luis_schema_version": "3.2.0",
    "versionId": "0.1",
    "name": "german_app_1.0.0",
    "desc": "",
    "culture": "de-de",
    "tokenizerVersion": "1.0.0",
    "intents": [
        {
            "name": "i1"
        },
        {
            "name": "None"
        }
    ],
    "entities": [
        {
            "name": "Fahrzeug",
            "roles": []
        }
    ],
    "composites": [],
    "closedLists": [],
    "patternAnyEntities": [],
    "regex_entities": [],
    "prebuiltEntities": [],
    "model_features": [],
    "regex_features": [],
    "patterns": [],
    "utterances": [
        {
            "text": "ich fahre einen krankenwagen",
            "intent": "i1",
            "entities": [
                {
                    "entity": "Fahrzeug",
                    "startPos": 23,
                    "endPos": 27
                }
            ]
        }
    ],
    "settings": []
}
```

Tokenizer JSON for version 1.0.1. Notice the property value for  `tokenizerVersion`.

```JSON
{
    "luis_schema_version": "3.2.0",
    "versionId": "0.1",
    "name": "german_app_1.0.1",
    "desc": "",
    "culture": "de-de",
    "tokenizerVersion": "1.0.1",
    "intents": [
        {
            "name": "i1"
        },
        {
            "name": "None"
        }
    ],
    "entities": [
        {
            "name": "Fahrzeug",
            "roles": []
        }
    ],
    "composites": [],
    "closedLists": [],
    "patternAnyEntities": [],
    "regex_entities": [],
    "prebuiltEntities": [],
    "model_features": [],
    "regex_features": [],
    "patterns": [],
    "utterances": [
        {
            "text": "ich fahre einen krankenwagen",
            "intent": "i1",
            "entities": [
                {
                    "entity": "Fahrzeug",
                    "startPos": 16,
                    "endPos": 27
                }
            ]
        }
    ],
    "settings": []
}
```
-->

Разметка происходит на уровне приложения. Разметки на уровне версии не поддерживаются.

[Импортируйте файл как новое приложение](luis-how-to-start-new-app.md), а не версию. Это действие означает, что новое приложение имеет другой идентификатор приложения, но использует версию анализатора, указанную в файле.
