---
title: Нормализация текста для фильтров, аспектов, сортировки
titleSuffix: Azure Cognitive Search
description: Укажите параметры нормализации для текстовых полей в индексе, чтобы настроить заданное по ключевым словам поведение при фильтрации, аспектах и сортировке.
author: IshanSrivastava
manager: jlembicz
ms.author: ishansri
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 02/23/2021
ms.openlocfilehash: 3e7a33d9213d7af44d2cfc50baa847534618f7e5
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104609525"
---
# <a name="text-normalization-for-case-insensitive-filtering-faceting-and-sorting"></a>Нормализация текста для фильтрации без учета регистра, аспектов и сортировки

 > [!IMPORTANT]
 > Нормализация в общедоступной предварительной версии доступна в **REST API 2020-06-30-Preview**. Предварительные версии функций предлагаются "как есть" в дополнительных условиях использования.

Поиск и извлечение документов из индекса Когнитивный поиск Azure требует, чтобы запрос соответствовал содержимому документа. Содержимое можно проанализировать, чтобы создать маркеры для сопоставления, как в случае `search` с параметром, или же можно использовать "как есть" для параметра с ключевым словом "как есть", как показано в `$filter` , `facets` и `$orderby` . Этот подход полностью или не используется в большинстве сценариев, но оказывается коротким, где простая Предварительная обработка, такая как регистр, Удаление диакритических знаков, asciifolding и т. д., требуется без прохождения всей цепочки анализа.

Рассмотрим следующие примеры:

+ `$filter=City eq 'Las Vegas'` будет возвращать только документы, содержащие точный текст "Лас-деньги", и исключать документы с "Лас-деньги" и "Лас-деньги", которые недостаточны, если вариант использования требует наличия всех документов независимо от регистра.

+ `search=*&facet=City,count:5` будет возвращать "Лас деньги", "Лас деньги" и "Лас деньги" как отдельные значения, несмотря на то, что они совпадают.

+ `search=usa&$orderby=City` Возвращает города в лексикографическом заказе: "Лас-деньги", "Сиэтл", "Лас-деньги", даже если цель — заказать одни и те же города вместе независимо от варианта. 

## <a name="normalizers"></a>Нормализации

Средство *нормализации* является компонентом поисковой системы, ответственным за предварительную обработку текста для сопоставления ключевых слов. Нормализации аналогичны анализаторам, за исключением того, что они не разбивают запрос на запросы. Ниже приведены некоторые преобразования, которые можно достичь с помощью средств нормализации.

+ Преобразование в нижний или верхний регистр.
+ Нормализовать диакритические знаки и диакритические знаки, такие как ц или ensure, в эквивалентные символы ASCII "o" и "e".
+ Сопоставьте символы, такие как `-` и пробелы, с заданным пользователем символом.

Для текстовых полей в индексе могут быть заданы нормализации и применены как при индексировании, так и при выполнении запросов.

## <a name="predefined-and-custom-normalizers"></a>Стандартные и пользовательские нормализации 

Azure Когнитивный поиск поддерживает стандартные средства нормализации для распространенных вариантов использования, а также возможность настройки в соответствии с требованиями.

| Категория | Описание |
|----------|-------------|
| [Предопределенные нормализации](#predefined-normalizers) | Предоставляется готовым и может использоваться без какой-либо настройки. |
|[Пользовательские нормализации](#add-custom-normalizers) | Для расширенных сценариев. Требуется определяемая пользователем конфигурация сочетания существующих элементов, состоящей из фильтров char и Token. <sup>1</sup>|

<sup>(1)</sup> пользовательские нормализации не задают лексемы, так как нормализации всегда создают один маркер.

## <a name="how-to-specify-normalizers"></a>Как указать методы нормализации

В текстовых полях можно указать параметры нормализации для каждого поля ( `Edm.String` и `Collection(Edm.String)` ), имеющие хотя бы одно из `filterable` `sortable` свойств, или, для которых `facetable` задано значение true. Параметр нормализации является необязательным и по `null` умолчанию. Рекомендуется оценить предопределенные нормализации перед настройкой пользовательского, чтобы упростить его использование. Попробуйте другой нормализацией, если результаты не ожидались.

Нормализации можно указывать только при добавлении нового поля в индекс. Рекомендуется заранее оценить потребности нормализации и назначить нормализации на начальных этапах разработки при удалении и повторном создании индексов. Нельзя указывать нормализацию для поля, которое уже было создано.

1. При создании определения поля в [индексе](/rest/api/searchservice/create-index)задайте для свойства "  **нормализация** " одно из следующих действий: [предопределенное нормализация](#predefined-normalizers) , например `lowercase` , или пользовательская нормализация (определенная в той же схеме индекса).  
 
   ```json
   "fields": [
    {
      "name": "Description",
      "type": "Edm.String",
      "retrievable": true,
      "searchable": true,
      "filterable": true,
      "analyzer": "en.microsoft",
      "normalizer": "lowercase"
      ...
    },
   ```

2. Пользовательские нормализации должны быть определены в разделе **[нормализации]** индекса первыми, а затем присваиваться определению поля, как показано на предыдущем шаге. Дополнительные сведения см. в статьях [Создание индекса](/rest/api/searchservice/create-index) , а также [Добавление пользовательских нормализаций](#add-custom-normalizers).


   ```json
   "fields": [
    {
      "name": "Description",
      "type": "Edm.String",
      "retrievable": true,
      "searchable": true,
      "analyzer": null,
      "normalizer": "my_custom_normalizer"
    },
   ```

 
> [!NOTE]
> Чтобы изменить нормализацию существующего поля, необходимо полностью перестроить индекс (нельзя перестроить отдельные поля).

Хорошим решением для производственных индексов, где перестроение индексов является дорогостоящим, является создание нового поля, идентичного старому, но с новым методом нормализации, и его использование вместо старого. Для внедрения нового поля используется запрос [Update Index](/rest/api/searchservice/update-index), а запрос [mergeOrUpload](/rest/api/searchservice/addupdate-or-delete-documents) позволяет заполнить его. Позже в рамках планового обслуживания индекса можно очистить его, чтобы удалить устаревшие поля.

## <a name="add-custom-normalizers"></a>Добавление пользовательских нормализаций

Пользовательские средства нормализации определяются в схеме индекса и могут быть указаны с помощью свойства Field. Определение пользовательской нормализации включает имя, тип, один или несколько фильтров char и фильтры маркеров. Фильтры char и фильтры маркеров являются стандартными блоками для пользовательского нормализации и отвечают за обработку текста. Эти фильтры применяются слева направо.

 `token_filter_name_1`— Это имя фильтра маркера, а `char_filter_name_1` `char_filter_name_2` — имена фильтров типа char (см. раздел [Поддерживаемые фильтры маркеров](#supported-token-filters) и таблицы фильтров char ниже для допустимых значений).

Определение нормализации является частью большего индекса. Дополнительные сведения см. в статье [Create Index (Azure Search Service REST API)](/rest/api/searchservice/create-index) (Создание индексов (REST API службы "Поиск Azure")).

```
"normalizers":(optional)[
   {
      "name":"name of normalizer",
      "@odata.type":"#Microsoft.Azure.Search.CustomNormalizer",
      "charFilters":[
         "char_filter_name_1",
         "char_filter_name_2"
      ],
      "tokenFilters":[
         "token_filter_name_1
      ]
   }
],
"charFilters":(optional)[
   {
      "name":"char_filter_name_1",
      "@odata.type":"#char_filter_type",
      "option1":value1,
      "option2":value2,
      ...
   }
],
"tokenFilters":(optional)[
   {
      "name":"token_filter_name_1",
      "@odata.type":"#token_filter_type",
      "option1":value1,
      "option2":value2,
      ...
   }
]
```

Пользовательские нормализации можно добавить во время создания индекса или позже, обновив существующий. Для добавления пользовательского нормализации к существующему индексу требуется, чтобы в [индексе Update](/rest/api/searchservice/update-index) был указан флаг **алловиндексдовнтиме** , и это приведет к недоступности индекса в течение нескольких секунд.

## <a name="normalizers-reference"></a>Справочник по нормализации

### <a name="predefined-normalizers"></a>Предопределенные нормализации
|**имя**;|**Описание и параметры**|  
|-|-|  
|standard| Текст в нижнем регистре, за которым следует asciifolding.|  
|Нижний регистр| Преобразует символы в нижний регистр.|
|верхний регистр| Преобразует символы в верхний регистр.|
|asciifolding| Преобразует символы, не входящие в основной Латинский символ Юникода, в их эквивалент ASCII, если он существует. Например, изменение продажам на.|
|elision| Удаляет элизии из начала токенов.|

### <a name="supported-char-filters"></a>Поддерживаемые фильтры char
Дополнительные сведения о фильтрах char см. в [справочнике по фильтрам char](index-add-custom-analyzers.md#CharFilter).
+ [Картограф](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/charfilter/MappingCharFilter.html)  
+ [pattern_replace](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/pattern/PatternReplaceCharFilter.html)

### <a name="supported-token-filters"></a>Поддерживаемые фильтры маркеров
В списке ниже показаны фильтры маркеров, поддерживаемые для нормализаций, и является подмножеством общих фильтров маркеров, участвующих в лексическом анализе. Дополнительные сведения о фильтрах см. в [справочнике по фильтрам токенов](index-add-custom-analyzers.md#TokenFilters).

+ [arabic_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ar/ArabicNormalizationFilter.html)
+ [asciifolding](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/ASCIIFoldingFilter.html)
+ [cjk_width](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/cjk/CJKWidthFilter.html)  
+ [elision](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/util/ElisionFilter.html)  
+ [german_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/de/GermanNormalizationFilter.html)
+ [hindi_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/hi/HindiNormalizationFilter.html)  
+ [indic_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/in/IndicNormalizationFilter.html)
+ [persian_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/fa/PersianNormalizationFilter.html)
+ [scandinavian_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/ScandinavianNormalizationFilter.html)  
+ [scandinavian_folding](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/ScandinavianFoldingFilter.html)
+ [sorani_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ckb/SoraniNormalizationFilter.html)  
+ [пишут](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/LowerCaseFilter.html)
+ [верхний регистр](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/UpperCaseFilter.html)


## <a name="custom-normalizer-example"></a>Пользовательский пример нормализации

В приведенном ниже примере показано пользовательское определение нормализации с соответствующими фильтрами char и фильтрами маркеров. Пользовательские параметры для фильтров char и фильтров маркеров задаются отдельно как именованные конструкции, а затем указываются в определении нормализации, как показано ниже.

* Пользовательский метод нормализации "my_custom_normalizer" определен в `normalizers` разделе определения индекса.
* Нормализация состоит из двух фильтров char и трех фильтров маркеров: элизии, строчных и настраиваемых фильтров asciifolding "my_asciifolding".
* Первый символ фильтра "map_dash" заменяет все дефисы символами подчеркивания, а вторая "remove_whitespace" удаляет все пробелы.

```json
  {
     "name":"myindex",
     "fields":[
        {
           "name":"id",
           "type":"Edm.String",
           "key":true,
           "searchable":false,
        },
        {
           "name":"city",
           "type":"Edm.String",
           "filterable": true,
           "facetable": true,
           "normalizer": "my_custom_normalizer"
        }
     ],
     "normalizers":[
        {
           "name":"my_custom_normalizer",
           "@odata.type":"#Microsoft.Azure.Search.CustomNormalizer",
           "charFilters":[
              "map_dash",
              "remove_whitespace"
           ],,
           "tokenFilters":[              
              "my_asciifolding",
              "elision",
              "lowercase",
           ]
        }
     ],
     "charFilters":[
        {
           "name":"map_dash",
           "@odata.type":"#Microsoft.Azure.Search.MappingCharFilter",
           "mappings":["-=>_"]
        },
        {
           "name":"remove_whitespace",
           "@odata.type":"#Microsoft.Azure.Search.MappingCharFilter",
           "mappings":["\\u0020=>"]
        }
     ],
     "tokenFilters":[
        {
           "name":"my_asciifolding",
           "@odata.type":"#Microsoft.Azure.Search.AsciiFoldingTokenFilter",
           "preserveOriginal":true
        }
     ]
  }
```

## <a name="see-also"></a>См. также раздел
+ [Анализаторы для лингвистической и текстовой обработки](search-analyzers.md)

+ [Search Documents (Azure Search Service REST API)](/rest/api/searchservice/search-documents) (Поиск по документам (REST API службы поиска Azure)) 
