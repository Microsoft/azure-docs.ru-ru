---
title: Добавление пользовательских анализаторов в строковые поля
titleSuffix: Azure Cognitive Search
description: Настройка маркеров текста и фильтров символов для выполнения анализа строк во время индексирования и запросов.
author: HeidiSteen
manager: nitinme
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 03/17/2021
ms.openlocfilehash: 831e57a68c79c245b96baec0fc3d062c4c9112c5
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104604446"
---
# <a name="add-custom-analyzers-to-string-fields-in-an-azure-cognitive-search-index"></a>Добавление пользовательских анализаторов в строковые поля в индексе Azure Когнитивный поиск

*Пользовательский анализатор* — это сочетание маркера, одного или нескольких фильтров маркеров и одного или нескольких фильтров символов, определяемых в индексе поиска, а затем ссылки на определения полей, для которых требуется пользовательский анализ. Токенизатор отвечает за разбивку текста на токены, а фильтры — за изменение токенов, созданных токенизатором. Фильтры символов подготавливают входной текст перед его обработкой с помощью анализатора. 

Пользовательское средство анализа позволяет управлять процессом преобразования текста в маркеры, поддерживающие индексирование и возможность поиска, позволяя выбирать типы анализа или фильтрации, которые будут вызываться, и порядок их возникновения. Если вы хотите использовать встроенный анализатор с пользовательскими параметрами, такими как изменение Макстокенленгс в стандарте, создайте пользовательский анализатор с именем, определяемым пользователем, чтобы задать эти параметры.

Ситуации, в которых могут быть полезны пользовательские анализаторы, включают:

- Использование фильтров символов для удаления разметки HTML перед разметкой текстовых входов или заменой определенных символов или символов.

- Фонетический поиск. Добавьте фонетический фильтр, чтобы искать слова на основе их звучания, а не написания.  

- Отключение лексического анализа. Используйте анализатор ключевых слов, чтобы создать поля с возможностью поиска, которые не анализируются.  

- Быстрый поиск по префиксам или суффиксам. Добавьте фильтр маркеров, создающий N-граммы с определенной стороны, чтобы индексировать префиксы слов для быстрого сопоставления. Объедините его с обратным фильтром маркеров, чтобы выполнить сопоставление суффиксов.  

- Пользовательская разметка. Например, с помощью создателя маркеров пробелов можно разделить предложения на маркеры, используя пробел как разделитель.  

- Приведение к ASCII. Добавьте стандартный фильтр приведения к ASCII для нормализации диакритических знаков, таких как ö или ê, в условиях поиска.  

Чтобы создать пользовательский анализатор, укажите его в разделе "Анализаторы" индекса во время разработки, а затем сослаться на него в полях с возможностью поиска, в строках EDM с помощью свойства "анализатор" или пары "Индексанализер" и "Сеарчанализер".

> [!NOTE]  
> Пользовательские анализаторы, которые вы создаете, не отображаются на портале Azure. Единственный способ добавления пользовательского анализатора — это код, определяющий индекс. 

## <a name="create-a-custom-analyzer"></a>Добавление пользовательского анализатора

Определение анализатора включает имя, тип, один или несколько фильтров символов, максимум один маркер, а также один или несколько фильтров маркеров для обработки после разметки. Фильтры символов применяются до разметки. Фильтры маркеров и символьные фильтры применяются слева направо.

- Имена в пользовательском анализаторе должны быть уникальными и не могут совпадать с любыми встроенными анализаторами, маркерами, фильтрами маркеров и символами. Название должно содержать только буквы, цифры, тире или знаки подчеркивания. Оно может начинаться и заканчиваться только буквенно-цифровыми знаками, и его длина не должна превышать 128 знаков. 

- Тип должен быть #Microsoft. Azure. Search. Кустоманализер.

- "Чарфилтерс" может быть одним или несколькими фильтрами из [символьных фильтров](#CharFilter), обработанных до разметки, в указанном порядке. Некоторые фильтры символов имеют параметры, которые можно задать в разделе "Чарфилтер". Фильтры символов необязательны.

- "лексический анализатор" — это [ровно один токен](#tokenizers). Требуется значение. Если вам нужно больше токенизаторов, создайте несколько пользовательских анализаторов и в схеме индекса назначьте каждому полю по одному анализатору.

- "Токенфилтерс" может быть одним или несколькими фильтрами из [фильтров маркеров](#TokenFilters), обработанных после разметки, в указанном порядке. Для фильтров маркеров, имеющих параметры, добавьте раздел "Токенфилтер" для указания конфигурации. Фильтры маркеров необязательны.

Анализаторы не должны создавать маркеры длиннее 300 символов, иначе индексирование завершится ошибкой. Чтобы обрезать длинный токен или исключить их, используйте **трункатетокенфилтер** и **ленгстокенфилтер** соответственно. См. раздел [**фильтры маркеров**](#TokenFilters) для справки.

```json
"analyzers":(optional)[
   {
      "name":"name of analyzer",
      "@odata.type":"#Microsoft.Azure.Search.CustomAnalyzer",
      "charFilters":[
         "char_filter_name_1",
         "char_filter_name_2"
      ],
      "tokenizer":"tokenizer_name",
      "tokenFilters":[
         "token_filter_name_1",
         "token_filter_name_2"
      ]
   },
   {
      "name":"name of analyzer",
      "@odata.type":"#analyzer_type",
      "option1":value1,
      "option2":value2,
      ...
   }
],
"charFilters":(optional)[
   {
      "name":"char_filter_name",
      "@odata.type":"#char_filter_type",
      "option1":value1,
      "option2":value2,
      ...
   }
],
"tokenizers":(optional)[
   {
      "name":"tokenizer_name",
      "@odata.type":"#tokenizer_type",
      "option1":value1,
      "option2":value2,
      ...
   }
],
"tokenFilters":(optional)[
   {
      "name":"token_filter_name",
      "@odata.type":"#token_filter_type",
      "option1":value1,
      "option2":value2,
      ...
   }
]
```

В определении индекса этот раздел можно разместить в любом месте текста запроса на создание индекса, но он обычно находится в конце.  

```json
{
  "name": "name_of_index",
  "fields": [ ],
  "suggesters": [ ],
  "scoringProfiles": [ ],
  "defaultScoringProfile": (optional) "...",
  "corsOptions": (optional) { },
  "analyzers":(optional)[ ],
  "charFilters":(optional)[ ],
  "tokenizers":(optional)[ ],
  "tokenFilters":(optional)[ ]
}
```

Определение анализатора является частью большого индекса. Определения для фильтров знаков, создателей маркеров и фильтров маркеров добавляются в индекс только в том случае, если заданы пользовательские параметры. Чтобы использовать существующий фильтр или создатель маркеров "как есть", укажите его по имени в определении анализатора. Дополнительные сведения см. в разделе [Создание индекса (остальное)](/rest/api/searchservice/create-index). Дополнительные примеры см. [в разделе Добавление анализаторов в Azure когнитивный Поиск](search-analyzers.md#examples).

## <a name="test-custom-analyzers"></a>Проверка пользовательских анализаторов

С помощью [анализатора тестов (RESTful)](/rest/api/searchservice/test-analyzer) можно увидеть, как анализатор разбивает текст на маркеры.

**Запрос**

```http
  POST https://[search service name].search.windows.net/indexes/[index name]/analyze?api-version=[api-version]
    Content-Type: application/json
    api-key: [admin key]

  {
     "analyzer":"my_analyzer",
     "text": "Vis-à-vis means Opposite"
  }
```

**Ответ**

```http
  {
    "tokens": [
      {
        "token": "vis_a_vis",
        "startOffset": 0,
        "endOffset": 9,
        "position": 0
      },
      {
        "token": "vis_à_vis",
        "startOffset": 0,
        "endOffset": 9,
        "position": 0
      },
      {
        "token": "means",
        "startOffset": 10,
        "endOffset": 15,
        "position": 1
      },
      {
        "token": "opposite",
        "startOffset": 16,
        "endOffset": 24,
        "position": 2
      }
    ]
  }
```

## <a name="update-custom-analyzers"></a>Обновление пользовательских анализаторов

После определения анализатора, маркера, фильтра маркеров или фильтра символов его нельзя изменить. Добавить новые определения в существующий индекс можно только в том случае, если в запросе на обновление индекса для флага `allowIndexDowntime` установлено значение true.

```http
PUT https://[search service name].search.windows.net/indexes/[index name]?api-version=[api-version]&allowIndexDowntime=true
```

Эта операция переведет индекс в автономный режим на несколько секунд, что приведет к сбоям при выполнении индексирования и запросов. После обновления индекса его производительность и доступность для записи может быть снижена на несколько минут или дольше, если индекс очень большой. Эти последствия временны и проходят сами по себе.

<a name="built-in-analyzers"></a>

## <a name="built-in-analyzers"></a>Встроенные анализаторы

Если вы хотите использовать встроенный анализатор с пользовательскими параметрами, создание пользовательского анализатора — это механизм, с помощью которого можно указать эти параметры. В отличие от этого, для использования встроенного анализатора "как есть" необходимо просто [сослаться на него по имени](search-analyzers.md#how-to-specify-analyzers) в определении поля.

|**Название анализатора**|**Тип анализатора**  <sup>1</sup>|**Описание и параметры**|  
|-----------------|-------------------------------|---------------------------|  
|[This](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/KeywordAnalyzer.html)| (тип применяется только при наличии параметров) |Обрабатывает все содержимое поля как один маркер. Это полезно для данных некоторых типов, таких как почтовые индексы, идентификаторы и названия продуктов.|  
|[pattern](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/miscellaneous/PatternAnalyzer.html)|PatternAnalyzer|Гибко разделяет текст на термины с помощью шаблона регулярного выражения. </br></br>**Параметры** </br></br>lowercase (тип: логическое значение) — определяет, являются ли термины строчными. Значение по умолчанию — true. </br></br>[pattern](https://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html?is-external=true) (тип: строка) — шаблон регулярного выражения для сопоставления разделителей маркеров. Значение по умолчанию — `\W+` , которое соответствует символам, отличным от слов. </br></br>[flags](https://docs.oracle.com/javase/6/docs/api/java/util/regex/Pattern.html#field_summary) (тип: строка) — флаги регулярных выражений. Значение по умолчанию - пустая строка. Допустимые значения: CANON_EQ, CASE_INSENSITIVE, COMMENTs, ДОТАЛЛ, LITERAL, MULTILINE, UNICODE_CASE, UNIX_LINES </br></br>stopwords (тип: массив строк) — список стоп-слов. Значение по умолчанию — пустой список.|  
|[простого](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/SimpleAnalyzer.html)|(тип применяется только при наличии параметров) |Разбивает текст по небуквенным знакам и преобразует его в нижний регистр. |  
|[Standard](https://lucene.apache.org/core/6_6_1/core/org/apache/lucene/analysis/standard/StandardAnalyzer.html) </br>(также называется standard.lucene)|StandardAnalyzer|Стандартный анализатор Lucene, состоящий из стандартного создателя маркеров, фильтра, преобразующего знаки в нижний регистр, и фильтра стоп-слов. </br></br>**Параметры** </br></br>maxTokenLength (тип: целое число) — максимальная длина маркера. Значение по умолчанию — 255. Маркеры, размер которых превышает максимальную длину, разделяются. Максимальная допустимая длина маркера — 300 знаков. </br></br>stopwords (тип: массив строк) — список стоп-слов. Значение по умолчанию — пустой список.|  
|standardasciifolding.lucene|(тип применяется только при наличии параметров) |Стандартный анализатор с фильтром приведения к ASCII. |  
|[stop](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/StopAnalyzer.html)|StopAnalyzer|Разбивает текст по небуквенным знакам, применяет фильтр преобразования в знаки нижнего регистра и фильтр стоп-слов. </br></br>**Параметры** </br></br>stopwords (тип: массив строк) — список стоп-слов. Значение по умолчанию представляет собой стандартный список для английского языка. |  
|[Бель](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/WhitespaceAnalyzer.html)|(тип применяется только при наличии параметров) |Анализатор, использующий создатель маркеров пробелов. Маркеры длиной более 255 знаков разделяются.|  

 <sup>1</sup> Типы анализаторов всегда начинаются в коде с префиксов "#Microsoft.Azure.Search". Например, "PatternAnalyzer" фактически будет указан как "#Microsoft.Azure.Search.PatternAnalyzer". Мы удалили префикс для краткости, но он требуется в коде.

Обратите внимание, что параметр analyzer_type предоставляется только для анализаторов, которые можно настроить. Если параметров нет, как в случае с анализатором ключевых слов, связанный тип #Microsoft.Azure.Search будет отсутствовать.

<a name="CharFilter"></a>

## <a name="character-filters"></a>Фильтры символов

В таблице ниже для фильтров знаков, реализованных с помощью Apache Lucene, приведены ссылки на документацию по API Lucene.

|**Имя фильтра знаков**|**Тип фильтра знаков** <sup>1</sup>|**Описание и параметры**|  
|--------------------|---------------------------------|---------------------------|
|[html_strip](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/charfilter/HTMLStripCharFilter.html)|(тип применяется только при наличии параметров)  |Фильтр знаков, который пытается удалить HTML-конструкции.|  
|[Картограф](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/charfilter/MappingCharFilter.html)|MappingCharFilter|Фильтр знаков, который применяет сопоставления, определенные в параметре сопоставлений. Сопоставление является каскадным (самое длинное сопоставление шаблона в заданной точке имеет приоритет). Замена может быть пустой строкой.  </br></br>**Параметры**  </br></br> mappings (тип: массив строк) — список сопоставлений в следующем формате: "a=>b" (все вхождения знака "a" будут заменены знаком "b"). Обязательный.|  
|[pattern_replace](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/pattern/PatternReplaceCharFilter.html)|PatternReplaceCharFilter|Фильтр знаков, который заменяет знаки во входной строке. Он использует регулярное выражение, чтобы определить последовательности знаков, которые нужно сохранить, и шаблон замены, чтобы определить знаки для замены. Например, входной текст = "aa bb aa bb", шаблон = "(aa)\\\s+(bb)", замена = "$1#$2", результат = "aa#bb aa#bb".  </br></br>**Параметры**  </br></br>pattern (тип: строка) — обязательный параметр.  </br></br>replacement (тип: строка) — обязательный параметр.|  

 <sup>1</sup> Типы фильтров знаков всегда начинаются в коде с префиксов "#Microsoft.Azure.Search". Например, "MappingCharFilter" фактически будет указан как "#Microsoft.Azure.Search.MappingCharFilter". Мы удалили префикс, чтобы уменьшить ширину таблицы, но не забудьте включить его в код. Обратите внимание, что char_filter_type предоставляется только для фильтров, которые могут быть настроены. Если параметров нет, как в случае с html_strip, связанный тип #Microsoft.Azure.Search будет отсутствовать.

<a name="tokenizers"></a>

## <a name="tokenizers"></a>Токенизаторы

Токенизатор разбивает непрерывный текст на последовательность токенов, например делит предложение на слова. В таблице ниже для создателей маркеров, реализованных с помощью Apache Lucene, приведены ссылки на документацию по API Lucene.

|**Имя создателя маркеров**|**Тип создателя маркеров** <sup>1</sup>|**Описание и параметры**|  
|------------------|-------------------------------|---------------------------|  
|[Классические](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/standard/ClassicTokenizer.html)|ClassicTokenizer|Создатель маркеров на основе грамматики, который подходит для обработки документов на большинстве европейских языков.  </br></br>**Параметры**  </br></br>maxTokenLength (тип: целое число) — максимальная длина маркера. Значение по умолчанию: 255, максимум: 300. Маркеры, размер которых превышает максимальную длину, разделяются.|  
|[edgeNGram](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ngram/EdgeNGramTokenizer.html)|EdgeNGramTokenizer|Размечает входные данные с указанной стороны на N-граммы заданного размера.  </br></br> **Параметры**  </br></br>Минграм (тип: int) — по умолчанию: 1, максимум: 300.  </br></br>Максграм (тип: int) — по умолчанию: 2, максимум: 300. Значение должно превышать minGram.  </br></br>tokenChars (тип: массив строк) — классы знаков, которые нужно оставить в маркерах. Допустимые значения: </br>"letter", "digit", "whitespace", "punctuation", "symbol". По умолчанию используется пустой массив — все знаки сохраняются. |  
|[keyword_v2](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/KeywordTokenizer.html)|KeywordTokenizerV2|Выдает все входные данные в виде одного маркера.  </br></br>**Параметры**  </br></br>maxTokenLength (тип: целое число) — максимальная длина маркера. Значение по умолчанию: 256, максимум: 300. Маркеры, размер которых превышает максимальную длину, разделяются.|  
|[письм](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/LetterTokenizer.html)|(тип применяется только при наличии параметров)  |Разбивает текст по небуквенным знакам. Маркеры длиной более 255 знаков разделяются.|  
|[пишут](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/LowerCaseTokenizer.html)|(тип применяется только при наличии параметров)  |Разбивает текст по небуквенным знакам и преобразует его в нижний регистр. Маркеры длиной более 255 знаков разделяются.|  
| microsoft_language_tokenizer| MicrosoftLanguageTokenizer| Разбивает текст на основе правил определенного языка.  </br></br>**Параметры**  </br></br>Макстокенленгс (тип: int) — максимальная длина маркера, по умолчанию — 255, максимум: 300. Маркеры, размер которых превышает максимальную длину, разделяются. Маркеры длиной более 300 знаков сначала разбиваются на маркеры длиной 300 знаков, а затем каждый из них разбивается на основе параметра maxTokenLength.  </br></br>isSearchTokenizer (тип: логическое значение) — задайте значение true, если используется как создатель маркеров поиска, или false, если используется в качестве создателя маркеров индексации. </br></br>language (тип: строка) — используемый язык. Значение по умолчанию: "english". Допустимые значения: </br>"bangla", "bulgarian", "catalan", "chineseSimplified",  "chineseTraditional", "croatian", "czech", "danish", "dutch", "english",  "french", "german", "greek", "gujarati", "hindi", "icelandic", "indonesian", "italian", "japanese", "kannada", "korean", "malay", "malayalam", "marathi", "norwegianBokmaal", "polish", "portuguese", "portugueseBrazilian", "punjabi", "romanian", "russian", "serbianCyrillic", "serbianLatin", "slovenian", "spanish", "swedish", "tamil", "telugu", "thai", "ukrainian", "urdu", "vietnamese". |
| microsoft_language_stemming_tokenizer | MicrosoftLanguageStemmingTokenizer| Делит текст на основе правил определенного языка и сокращает слова до их базовых форм. </br></br>**Параметры** </br></br>Макстокенленгс (тип: int) — максимальная длина маркера, по умолчанию — 255, максимум: 300. Маркеры, размер которых превышает максимальную длину, разделяются. Маркеры длиной более 300 знаков сначала разбиваются на маркеры длиной 300 знаков, а затем каждый из них разбивается на основе параметра maxTokenLength. </br></br> isSearchTokenizer (тип: логическое значение) — задайте значение true, если используется как создатель маркеров поиска, или false, если используется в качестве создателя маркеров индексации. </br></br>language (тип: строка) — используемый язык. Значение по умолчанию: "english". Допустимые значения: </br>"arabic", "bangla", "bulgarian", "catalan", "croatian", "czech", "danish", "dutch", "english", "estonian", "finnish", "french", "german", "greek", "gujarati", "hebrew", "hindi", "hungarian", "icelandic", "indonesian", "italian", "kannada", "latvian", "lithuanian", "malay", "malayalam", "marathi", "norwegianBokmaal", "polish", "portuguese", "portugueseBrazilian", "punjabi", "romanian", "russian", "serbianCyrillic", "serbianLatin", "slovak", "slovenian", "spanish", "swedish", "tamil", "telugu", "turkish", "ukrainian", "urdu". |
|[nGram](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ngram/NGramTokenizer.html)|NGramTokenizer|Размечает входные данные на N-граммы заданного размера. </br></br>**Параметры** </br></br>Минграм (тип: int) — по умолчанию: 1, максимум: 300. </br></br>Максграм (тип: int) — по умолчанию: 2, максимум: 300. Значение должно превышать minGram. </br></br>tokenChars (тип: массив строк) — классы знаков, которые нужно оставить в маркерах. Допустимые значения: "letter", "digit", "whitespace", "punctuation" и "symbol". По умолчанию используется пустой массив — все знаки сохраняются. |  
|[path_hierarchy_v2](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/path/PathHierarchyTokenizer.html)|PathHierarchyTokenizerV2|Создатель маркеров для иерархий в виде пути. **Параметры** </br></br>delimiter (тип: строка). Значение по умолчанию: '/. </br></br>replacement (тип: строка) — если задано, заменяет знак-разделитель. Значение по умолчанию соответствует значению параметра delimiter. </br></br>maxTokenLength (тип: целое число) — максимальная длина маркера. Значение по умолчанию: 300, максимум: 300. Пути длиннее, чем значение maxTokenLength, игнорируются. </br></br>reverse (тип: логическое значение) — если задано значение true, создает маркер в обратном порядке. По умолчанию: false. </br></br>skip (тип: логическое значение) — начальные маркеры, которые следует пропустить. Значение по умолчанию равно 0.|  
|[pattern](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/pattern/PatternTokenizer.html)|PatternTokenizer|Этот создатель маркеров использует сопоставление шаблонов регулярных выражений для создания различных маркеров. </br></br>**Параметры** </br></br> [pattern](https://docs.oracle.com/javase/6/docs/api/java/util/regex/Pattern.html) (тип: строка) — шаблон регулярного выражения для сопоставления разделителей токенов. Значение по умолчанию — `\W+` , которое соответствует символам, отличным от слов. </br></br>[flags](https://docs.oracle.com/javase/6/docs/api/java/util/regex/Pattern.html#field_summary) (тип: строка) — флаги регулярных выражений. Значение по умолчанию - пустая строка. Допустимые значения: CANON_EQ, CASE_INSENSITIVE, COMMENTs, ДОТАЛЛ, LITERAL, MULTILINE, UNICODE_CASE, UNIX_LINES </br></br>group (тип: целое число) — указывает группу, которую нужно извлекать в маркеры. Значение по умолчанию: "-1" (разделение).|
|[standard_v2](https://lucene.apache.org/core/6_6_1/core/org/apache/lucene/analysis/standard/StandardTokenizer.html)|StandardTokenizerV2|Разбивает текст по правилам [сегментации текста в формате Юникод](https://unicode.org/reports/tr29/). </br></br>**Параметры** </br></br>maxTokenLength (тип: целое число) — максимальная длина маркера. Значение по умолчанию: 255, максимум: 300. Маркеры, размер которых превышает максимальную длину, разделяются.|  
|[uax_url_email](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/standard/UAX29URLEmailTokenizer.html)|UaxUrlEmailTokenizer|Размечает URL-адреса и сообщения электронной почты как один маркер. </br></br>**Параметры** </br></br> maxTokenLength (тип: целое число) — максимальная длина маркера. Значение по умолчанию: 255, максимум: 300. Маркеры, размер которых превышает максимальную длину, разделяются.|  
|[Бель](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/WhitespaceTokenizer.html)|(тип применяется только при наличии параметров) |Разбивает текст по пробелам. Маркеры длиной более 255 знаков разделяются.|  

 <sup>1</sup> Типы создателей маркеров всегда начинаются в коде с префиксов "#Microsoft.Azure.Search". Например, "ClassicTokenizer" фактически будет указан как "#Microsoft.Azure.Search.ClassicTokenizer". Мы удалили префикс, чтобы уменьшить ширину таблицы, но не забудьте включить его в код. Обратите внимание, что tokenizer_type предоставляется только для поданализаторов, которые могут быть настроены. Если параметров нет, как в случае с создателем буквенных маркеров, связанный тип #Microsoft.Azure.Search будет отсутствовать.

<a name="TokenFilters"></a>

## <a name="token-filters"></a>Фильтры токенов

Фильтр токенов используется для фильтрации или изменения токенов, созданных токенизатором. Например, вы можете указать специальный фильтр, который преобразует все символы в нижний регистр. Пользовательский анализатор может содержать несколько фильтров токенов. Фильтры токенов применяются в том порядке, в котором они указаны. 

В таблице ниже для фильтров маркеров, реализованных с помощью Apache Lucene, приведены ссылки на документацию по API Lucene.

|**Имя фильтра маркеров**|**Тип фильтра маркеров** <sup>1</sup>|**Описание и параметры**|  
|-|-|-|  
|[arabic_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ar/ArabicNormalizationFilter.html)|(тип применяется только при наличии параметров)  |Фильтр маркеров, применяющий нормализатор арабского языка для нормализации орфографии.|  
|[apostrophe](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/tr/ApostropheFilter.html)|(тип применяется только при наличии параметров)  |Удаляет все знаки после апострофа (включая сам апостроф). |  
|[asciifolding](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/ASCIIFoldingFilter.html)|AsciiFoldingTokenFilter|Преобразует алфавитные, числовые и символьные знаки формата Юникод, которых нет в первых 127 знаках ASCII (блок Юникода "основная латиница"), в их эквиваленты ASCII, если таковые существуют.<br /><br /> **Параметры**<br /><br /> preserveOriginal (тип: логическое значение) — если значение равно true, сохраняется исходный маркер. Значение по умолчанию – false.|  
|[cjk_bigram](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/cjk/CJKBigramFilter.html)|CjkBigramTokenFilter|Формирует биграммы терминов ККЯ, создаваемых с помощью StandardTokenizer.<br /><br /> **Параметры**<br /><br /> ignoreScripts (тип: массив строк) — сценарий, который следует игнорировать. Допустимые значения: "han", "hiragana", "katakana" и "hangul". Значение по умолчанию — пустой список.<br /><br /> outputUnigrams (тип: логическое значение) — задайте значение true, если нужно всегда выводить и униграммы, и биграммы. Значение по умолчанию – false.|  
|[cjk_width](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/cjk/CJKWidthFilter.html)|(тип применяется только при наличии параметров)  |Нормализует различия в ширине ККЯ. Преобразовывает полноширинные варианты ASCII в эквивалентные знаки основной латиницы, а варианты полуширинной катаканы в эквивалентные знаки обычной японской азбуки. |  
|[Классические](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/standard/ClassicFilter.html)|(тип применяется только при наличии параметров)  |Удаляет английские притяжательные местоимения и точки из аббревиатур. |  
|[common_grams](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/commongrams/CommonGramsFilter.html)|CommonGramTokenFilter|Создает биграммы для часто встречающихся терминов при индексировании. Отдельные термины также индексируются с наложением биграмм.<br /><br /> **Параметры**<br /><br /> commonWords (тип: массив строк) — набор распространенных слов. Значение по умолчанию — пустой список. Обязательный.<br /><br /> ignoreCase (тип: логическое значение) — если значение равно true, сопоставление выполняется без учета регистра. Значение по умолчанию – false.<br /><br /> queryMode (тип: логическое значение) — создает биграммы, а затем удаляет распространенные слова и отдельные термины, за которыми следует распространенное слово. Значение по умолчанию – false.|  
|[dictionary_decompounder](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/compound/DictionaryCompoundWordTokenFilter.html)|DictionaryDecompounderTokenFilter|Разбивает составные слова, распространенные во многих германских языках.<br /><br /> **Параметры**<br /><br /> wordList (тип: массив строк) — список слов для сопоставления. Значение по умолчанию — пустой список. Обязательный.<br /><br /> minWordSize (тип: целое число) — обрабатываются только слова длиннее этого значения. Значение по умолчанию — 5.<br /><br /> minSubwordSize (тип: целое число) — выводятся только подслова длиннее этого ограничения. Значение по умолчанию — 2<br /><br /> maxSubwordSize (тип: целое число) — выводятся только подслова короче этого ограничения. Значение по умолчанию — 15.<br /><br /> onlyLongestMatch (тип: логическое значение) — в выходные данные добавляется только самое длинное соответствующее подслово. Значение по умолчанию – false.|  
|[edgeNGram_v2](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ngram/EdgeNGramTokenFilter.html)|EdgeNGramTokenFilterV2|Генерирует N-граммы заданного размера, начиная с начала или конца входного маркера.<br /><br /> **Параметры**<br /><br /> Минграм (тип: int) — по умолчанию: 1, максимум: 300.<br /><br /> Максграм (тип: int) — по умолчанию: 2, максимальное 300. Значение должно превышать minGram.<br /><br /> side (тип: строка) — указывает, с какой стороны входных данных следует создавать N-грамму. Допустимые значения: "front", "back". |  
|[elision](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/util/ElisionFilter.html)|ElisionTokenFilter|Удаляет элизии. Например "l'avion" ("самолет"; с артиклем) преобразуется в "avion" ("самолет"; без артикля).<br /><br /> **Параметры**<br /><br /> articles (тип: массив строк) — набор статей, которые следует удалить. Значение по умолчанию — пустой список. Если список статей не указан, по умолчанию удаляются все статьи на французском языке.|  
|[german_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/de/GermanNormalizationFilter.html)|(тип применяется только при наличии параметров)  |Нормализует немецкие знаки в соответствии с эвристикой морфологического алгоритма [German2 snowball](https://snowballstem.org/algorithms/german2/stemmer.html).|  
|[hindi_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/hi/HindiNormalizationFilter.html)|(тип применяется только при наличии параметров)  |Нормализует текст на хинди, чтобы удалить некоторые различия в орфографических вариациях. |  
|[indic_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/in/IndicNormalizationFilter.html)|IndicNormalizationTokenFilter|Нормализует представление текста в Юникоде на индийских языках.
|[отслеживать](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/KeepWordFilter.html)|KeepTokenFilter|Фильтр маркеров, который сохраняет только маркеры с текстом, содержащимся в указанном списке слов.<br /><br /> **Параметры**<br /><br /> keepWords (тип: массив строк) — список слов, которые следует оставить. Значение по умолчанию — пустой список. Обязательный.<br /><br /> keepWordsCase (тип: логическое значение) — если значение равно true, сначала все слова преобразуются в нижний регистр. Значение по умолчанию – false.|  
|[keyword_marker](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/KeywordMarkerFilter.html)|KeywordMarkerTokenFilter|Помечает термины как ключевые слова.<br /><br /> **Параметры**<br /><br /> keywords (тип: массив строк) — список слов, которые следует пометить как ключевые. Значение по умолчанию — пустой список. Обязательный.<br /><br /> ignoreCase (тип: логическое значение) — если значение равно true, сначала все слова преобразуются в нижний регистр. Значение по умолчанию – false.|  
|[keyword_repeat](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/KeywordRepeatFilter.html)|(тип применяется только при наличии параметров)  |Выдает каждый входящий маркер дважды: сначала в качестве ключевого слова, а затем — не ключевого. |  
|[kstem](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/en/KStemFilter.html)|(тип применяется только при наличии параметров)  |Высокопроизводительный фильтр kstem для английского языка. |  
|[length](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/LengthFilter.html)|LengthTokenFilter|Удаляет слишком длинные или слишком короткие слова.<br /><br /> **Параметры**<br /><br /> min (тип: целое число) — минимальное число. Значение по умолчанию: 0, максимум: 300.<br /><br /> max (тип: целое число) — максимальное число. Значение по умолчанию: 300, максимум: 300.|  
|[limit](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/LimitTokenCountFilter.html)|Microsoft.Azure.Search.LimitTokenFilter|Ограничивает количество маркеров при индексировании.<br /><br /> **Параметры**<br /><br /> maxTokenCount (тип: целое число) — максимальное количество маркеров, которые следует создать. Значение по умолчанию — 1.<br /><br /> consumeAllTokens (тип: логическое значение) — нужно ли использовать все маркеры из входных данных, даже если достигнуто значение maxTokenCount. Значение по умолчанию – false.|  
|[пишут](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/LowerCaseFilter.html)|(тип применяется только при наличии параметров)  |Нормализует текст в маркере в нижний регистр. |  
|[nGram_v2](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ngram/NGramTokenFilter.html)|NGramTokenFilterV2|Создает N-граммы заданного размера.<br /><br /> **Параметры**<br /><br /> Минграм (тип: int) — по умолчанию: 1, максимум: 300.<br /><br /> Максграм (тип: int) — по умолчанию: 2, максимальное 300. Значение должно превышать minGram.|  
|[pattern_capture](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/pattern/PatternCaptureGroupTokenFilter.html)|PatternCaptureTokenFilter|Использует регулярные выражения Java для создания нескольких маркеров: по одному для каждой группы записи в одном или нескольких шаблонах.<br /><br /> **Параметры**<br /><br /> patterns (тип: массив строк) — список шаблонов для сопоставления с каждым маркеров. Обязательный.<br /><br /> preserveOriginal (тип: логическое значение) — задайте значение true, чтобы вернуть исходный маркер, даже если один из шаблонов совпадает (значение по умолчанию: true). |  
|[pattern_replace](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/pattern/PatternReplaceFilter.html)|PatternReplaceTokenFilter|Фильтр маркеров, который применяет шаблон к каждому маркеру в потоке, заменяя соответствующие вхождения указанной строкой замены.<br /><br /> **Параметры**<br /><br /> pattern (тип: строка) — обязательный параметр.<br /><br /> replacement (тип: строка) — обязательный параметр.|  
|[persian_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/fa/PersianNormalizationFilter.html)|(тип применяется только при наличии параметров) |Применяет нормализацию для персидского языка. |  
|[phonetic](https://lucene.apache.org/core/6_6_1/analyzers-phonetic/org/apache/lucene/analysis/phonetic/package-tree.html)|PhoneticTokenFilter|Создает маркеры для фонетических совпадений.<br /><br /> **Параметры**<br /><br /> encoder (тип: строка) — фонетический кодировщик, который следует использовать. Допустимые значения: "metaphone", "doubleMetaphone", "soundex", "refinedSoundex", "caverphone1", "caverphone2", "cologne", "nysiis", "koelnerPhonetik", "haasePhonetik" и "beiderMorse". Значение по умолчанию — "metaphone".<br /><br /> Дополнительные сведения см. [здесь](https://commons.apache.org/proper/commons-codec/archives/1.10/apidocs/org/apache/commons/codec/language/package-frame.html).<br /><br /> replace (тип: логическое значение) — true, если закодированные маркеры должны заменить оригинальные маркеры, false, если они должны быть добавлены как синонимы. Значение по умолчанию — true.|  
|[porter_stem](https://lucene.apache.org/core/6_6_1/analyzers-common/org/tartarus/snowball/ext/PorterStemmer.html)|(тип применяется только при наличии параметров)  |Преобразует поток маркеров в соответствии с [алгоритмом стемминга (поиска основы слова) Портера](https://tartarus.org/~martin/PorterStemmer/). |  
|[Отмена](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/reverse/ReverseStringFilter.html)|(тип применяется только при наличии параметров)  |Обращает порядок строки маркера. |  
|[scandinavian_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/ScandinavianNormalizationFilter.html)|(тип применяется только при наличии параметров)  |Нормализует использование взаимозаменяемых скандинавских знаков. |  
|[scandinavian_folding](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/ScandinavianFoldingFilter.html)|(тип применяется только при наличии параметров)  |Свертывает скандинавские знаки åÅäæÄÆ->a и öÖøØ->o. Он также предотвращает использование двойных гласных aa, ae, ao, oe и oo, оставляя только первую. |  
|[shingle](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/shingle/ShingleFilter.html)|ShingleTokenFilter|Создает сочетания маркеров в виде одного маркера.<br /><br /> **Параметры**<br /><br /> maxShingleSize (тип: целое число). Значение по умолчанию: 2.<br /><br /> minShingleSize (тип: целое число). Значение по умолчанию: 2.<br /><br /> outputUnigrams (тип: логическое значение) — если значение равно true, выходной поток содержит входные маркеры (униграммы), а также объединения. Значение по умолчанию — true.<br /><br /> outputUnigramsIfNoShingles (тип: логическое значение) — если значение равно true, переопределяет поведение outputUnigrams. Значение false следует использовать, когда объединения недоступны. Значение по умолчанию – false.<br /><br /> tokenSeparator (тип: строка) — строка, используемая при соединении соседних маркеров для формирования объединения. Значение по умолчанию — " ".<br /><br /> filterToken (тип: строка) — строка для вставки в каждую позицию, в которой нет маркера. Значение по умолчанию — "_".|  
|[snowball](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/snowball/SnowballFilter.html)|SnowballTokenFilter|Фильтр маркера Snowball.<br /><br /> **Параметры**<br /><br /> language (тип: строка). Допустимые значения: "armenian", "basque", "catalan", "danish", "dutch", "english", "finnish", "french", "german", "german2", "hungarian", "italian", "kp", "lovins", "norwegian", "porter", "portuguese", "romanian", "russian", "spanish", "swedish" и "turkish".|  
|[sorani_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ckb/SoraniNormalizationFilter.html)|SoraniNormalizationTokenFilter|Нормализует представление текста в Юникоде на языке сорани.<br /><br /> **Параметры**<br /><br /> Отсутствует.|  
|stemmer|StemmerTokenFilter|Языковой стемминговый фильтр.<br /><br /> **Параметры**<br /><br /> language (тип: строка). Допустимые значения: <br /> -   [Арабской](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ar/ArabicStemmer.html)<br />-   [языка](https://snowballstem.org/algorithms/armenian/stemmer.html)<br />-   [Баск](https://snowballstem.org/algorithms/basque/stemmer.html)<br />-   [португальского](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/br/BrazilianStemmer.html)<br />-"Болгарский"<br />-   [Каталанский](https://snowballstem.org/algorithms/catalan/stemmer.html)<br />-   [Czech](https://portal.acm.org/citation.cfm?id=1598600)<br />-   [язык](https://snowballstem.org/algorithms/danish/stemmer.html)<br />-   [Нидерландский](https://snowballstem.org/algorithms/dutch/stemmer.html)<br />-   ["dutchKp"](https://snowballstem.org/algorithms/kraaij_pohlmann/stemmer.html)<br />-   [English](https://snowballstem.org/algorithms/porter/stemmer.html)<br />-   ["lightEnglish"](https://ciir.cs.umass.edu/pubfiles/ir-35.pdf)<br />-   ["minimalEnglish"](https://www.researchgate.net/publication/220433848_How_effective_is_suffixing)<br />-   ["possessiveEnglish"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/en/EnglishPossessiveFilter.html)<br />-   ["porter2"](https://snowballstem.org/algorithms/english/stemmer.html)<br />-   ["lovins"](https://snowballstem.org/algorithms/lovins/stemmer.html)<br />-   [Финляндия](https://snowballstem.org/algorithms/finnish/stemmer.html)<br />-"Лигхтфинниш"<br />-   [французский](https://snowballstem.org/algorithms/french/stemmer.html)<br />-   ["lightFrench"](https://dl.acm.org/citation.cfm?id=1141523)<br />-   ["minimalFrench"](https://dl.acm.org/citation.cfm?id=318984)<br />-"галисийский"<br />-"МинималгалиЦиан"<br />-   [языка](https://snowballstem.org/algorithms/german/stemmer.html)<br />-   ["german2"](https://snowballstem.org/algorithms/german2/stemmer.html)<br />-   ["lightGerman"](https://dl.acm.org/citation.cfm?id=1141523)<br />-"Минималжерман"<br />-   [Greek](https://sais.se/mthprize/2007/ntais2007.pdf)<br />-"Хинди"<br />-   [Hungarian](https://snowballstem.org/algorithms/hungarian/stemmer.html)<br />-   ["lightHungarian"](https://dl.acm.org/citation.cfm?id=1141523&dl=ACM&coll=DL&CFID=179095584&CFTOKEN=80067181)<br />-   [Индонезийский](https://eprints.illc.uva.nl/741/2/MoL-2003-03.text.pdf)<br />-   [Ирландский](https://snowballstem.org/otherapps/oregan/)<br />-   [Итальянский](https://snowballstem.org/algorithms/italian/stemmer.html)<br />-   ["lightItalian"](https://www.ercim.eu/publication/ws-proceedings/CLEF2/savoy.pdf)<br />-   ["sorani"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ckb/SoraniStemmer.html)<br />-   [Latvian](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/lv/LatvianStemmer.html)<br />-   [Норвегии](https://snowballstem.org/algorithms/norwegian/stemmer.html)<br />-   ["lightNorwegian"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/no/NorwegianLightStemmer.html)<br />-   ["minimalNorwegian"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/no/NorwegianMinimalStemmer.html)<br />-   ["lightNynorsk"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/no/NorwegianLightStemmer.html)<br />-   ["minimalNynorsk"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/no/NorwegianMinimalStemmer.html)<br />-   [Португальский](https://snowballstem.org/algorithms/portuguese/stemmer.html)<br />-   ["lightPortuguese"](https://dl.acm.org/citation.cfm?id=1141523&dl=ACM&coll=DL&CFID=179095584&CFTOKEN=80067181)<br />-   ["minimalPortuguese"](https://www.inf.ufrgs.br/~buriol/papers/Orengo_CLEF07.pdf)<br />-   ["portugueseRslp"](https://www.inf.ufrgs.br//~viviane/rslp/index.htm)<br />-   [Румынская](https://snowballstem.org/otherapps/romanian/)<br />-   [Московское](https://snowballstem.org/algorithms/russian/stemmer.html)<br />-   ["lightRussian"](https://doc.rero.ch/lm.php?url=1000%2C43%2C4%2C20091209094227-CA%2FDolamic_Ljiljana_-_Indexing_and_Searching_Strategies_for_the_Russian_20091209.pdf)<br />-   [языках](https://snowballstem.org/algorithms/spanish/stemmer.html)<br />-   ["lightSpanish"](https://www.ercim.eu/publication/ws-proceedings/CLEF2/savoy.pdf)<br />-   [Swedish](https://snowballstem.org/algorithms/swedish/stemmer.html)<br />-"Лигхтсведиш"<br />-   [Turkish](https://snowballstem.org/algorithms/turkish/stemmer.html)|  
|[stemmer_override](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/StemmerOverrideFilter.html)|StemmerOverrideTokenFilter|Любые словарные однокоренные термины помечаются как ключевые слова, что предотвращает морфологический поиск по цепочке. Необходимо поместить перед всеми стемминговыми фильтрами.<br /><br /> **Параметры**<br /><br /> rules (тип: массив строк) — правила поиска корней в следующем формате "слово => корень", например "бегать => бежать". Значение по умолчанию — пустой список.  Обязательный.|  
|[стоп-слова](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/StopFilter.html)|StopwordsTokenFilter|Удаляет стоп-слова из потока маркеров. По умолчанию фильтр использует стандартный список стоп-слов для английского языка.<br /><br /> **Параметры**<br /><br /> stopwords (тип: массив строк) — список стоп-слов. Не указывается, если указан параметр stopwordsList.<br /><br /> stopwordsList (тип: строка) — стандартный список стоп-слов. Не указывается, если заданы стоп-слова. Допустимые значения: "arabic", "armenian", "basque", "brazilian", "bulgarian", "catalan", "czech", "danish", "dutch", "english", "finnish", "french", "galician", "german", "greek", "hindi", "hungarian", "indonesian", "irish", "italian", "latvian", "norwegian", "persian", "portuguese", "romanian", "russian", "sorani", "spanish", "swedish", "thai" и "turkish". Значение по умолчанию: "english". Не указывается, если заданы стоп-слова. <br /><br /> ignoreCase (тип: логическое значение) — если значение равно true, сначала слова преобразуются в нижний регистр. Значение по умолчанию – false.<br /><br /> removeTrailing (тип: логическое значение) — если значение равно true, последний поисковый термин игнорируется, если это стоп-слово. Значение по умолчанию — true.
|[термин](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/synonym/SynonymFilter.html)|SynonymTokenFilter|Сопоставляет одно-или многословные синонимы в потоке маркеров.<br /><br /> **Параметры**<br /><br /> synonyms (тип: массив строк) — обязательный параметр. Список синонимов в одном из следующих двух форматов.<br /><br /> "incredible, unbelievable, fabulous => amazing" — все термины c левой стороны знака => заменяются всеми терминами с правой стороны.<br /><br /> "incredible, unbelievable, fabulous, amazing" — список эквивалентных слов, разделенных запятыми. Установите параметр expand, чтобы изменить способ интерпретации этого списка.<br /><br /> ignoreCase (тип: логическое значение) — меняет регистр входных данных для сопоставления. Значение по умолчанию – false.<br /><br /> expand (тип: логическое значение) — если значение равно true, то все слова в списке синонимов (если нотация =>не используется) сопоставляются друг с другом. <br />Список "incredible, unbelievable, fabulous, amazing" равен списку "incredible, unbelievable, fabulous, amazing => incredible, unbelievable, fabulous, amazing".<br /><br />Если значение равно false, список "incredible, unbelievable, fabulous, amazing" равен списку "incredible, unbelievable, fabulous, amazing => incredible".|  
|[trim](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/TrimFilter.html)|(тип применяется только при наличии параметров)  |Усекает пробел в начале и конце маркеров. |  
|[truncate](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/TruncateTokenFilter.html)|TruncateTokenFilter|Усекает термины до определенной длины.<br /><br /> **Параметры**<br /><br /> Length (тип: int) — по умолчанию: 300, максимум: 300. Обязательный.|  
|[unique](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/RemoveDuplicatesTokenFilter.html)|UniqueTokenFilter|Отфильтровывает маркеры с тем же текстом, что и в предыдущем маркере.<br /><br /> **Параметры**<br /><br /> onlyOnSamePosition (тип: логическое значение) — если указано, удаляет дубликаты только в той же позиции. Значение по умолчанию — true.|  
|[верхний регистр](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/UpperCaseFilter.html)|(тип применяется только при наличии параметров)  |Нормализует текст в маркере в верхний регистр. |  
|[word_delimiter](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/WordDelimiterFilter.html)|WordDelimiterTokenFilter|Разделяет слова на подслова и выполняет необязательные преобразования в группах подслов.<br /><br /> **Параметры**<br /><br /> generateWordParts (тип: логическое значение) — разбивает слова на части, например, "AzureSearch" становится "Azure" "Search". Значение по умолчанию — true.<br /><br /> generateNumberParts (тип: логическое значение) — создает определенное число подслов. Значение по умолчанию — true.<br /><br /> catenateWords (тип: логическое значение) — объединяет максимальное количество частей слов, например "Azure-Search" стает "AzureSearch". Значение по умолчанию – false.<br /><br /> catenateNumbers (тип: логическое значение) — объединяет максимальное количество частей чисел, например "1-2" стает "12". Значение по умолчанию – false.<br /><br /> catenateAll (тип: логическое значение) — объединяет все части подслов, например "Azure-Search-1" стает "AzureSearch1". Значение по умолчанию – false.<br /><br /> splitOnCaseChange (тип: логическое значение) — если значение равно true, делит слова по знаку изменения регистра, например "AzureSearch" становится "Azure" "Search". Значение по умолчанию — true.<br /><br /> preserveOriginal — сохраняет изначальные слова и добавляет их в список подслов. Значение по умолчанию – false.<br /><br /> splitOnNumerics (тип: логическое значение) — если значение равно true, разделяет по цифрам, например "Azure1Search" становится "Azure" "1" "Search". Значение по умолчанию — true.<br /><br /> stemEnglishPossessive (тип: логическое значение) — удаляет "'s" в конце каждого подслова. Значение по умолчанию — true.<br /><br /> protectedWords (тип: массив строк) — маркеры, которые следует защитить от разделения. Значение по умолчанию — пустой список.|  

 <sup>1</sup> Типы фильтров маркеров всегда начинаются в коде с префиксов "#Microsoft.Azure.Search". Например, "ArabicNormalizationTokenFilter" фактически будет указан как "#Microsoft.Azure.Search.ArabicNormalizationTokenFilter".  Мы удалили префикс, чтобы уменьшить ширину таблицы, но не забудьте включить его в код.  

## <a name="see-also"></a>См. также раздел

- [Azure Когнитивный поиск API-интерфейсы службы других интерфейсов](/rest/api/searchservice/)
- [Анализаторы в Когнитивный поиск Azure (примеры)](search-analyzers.md#examples)
- [Создание индексов — REST](/rest/api/searchservice/create-index)