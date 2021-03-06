---
title: Синонимы для расширения запроса по индексу поиска
titleSuffix: Azure Cognitive Search
description: Создайте карту синонимов, чтобы расширить область поискового запроса по индексу Azure Когнитивный поиск. В область поиска включаются эквивалентные термины, указанные в списке.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 12/18/2020
ms.openlocfilehash: 5e608d38ff70d51b569088629a6d80cb08e74ed4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98251630"
---
# <a name="synonyms-in-azure-cognitive-search"></a>Синонимы в Azure Когнитивный поиск

С помощью карт синонимов можно связать эквивалентные термины, расширяя область запроса, не выполняя фактического предоставления этого термина пользователю. Например, при условии, что "собака", "собака" и "щенок" являются синонимами, запрос на "собака" будет соответствовать документу, содержащему "Dog".

## <a name="create-synonyms"></a>Создание синонимов

Схема синонимов — это ресурс-контейнер, который можно создать один раз и использовать во многих индексах. [Уровень служб](search-limits-quotas-capacity.md#synonym-limits) определяет, сколько карт синонимов можно создать, от трех карт синонимов для уровней Free и Basic до 20 для уровней "Стандартный". 

Вы можете создать несколько сопоставлений синонимов для разных языков, например английской и французской версий, или лексиконов, если содержимое включает в себя техническую или скрытую терминологию. Несмотря на то, что в службе поиска можно создать несколько сопоставлений синонимов, поле может использовать только один из них.

Схема синонимов состоит из имени, формата и правил, которые работают как записи сопоставлений синонимов. Единственным поддерживаемым форматом является `solr` , а `solr` Формат определяет конструкцию правила.

```http
POST /synonymmaps?api-version=2020-06-30
{
    "name": "geo-synonyms",
    "format": "solr",
    "synonyms": "
        USA, United States, United States of America\n
        Washington, Wash., WA => WA\n"
}
```

Чтобы создать карту синонимов, используйте [карту создания синонимов (REST API)](/rest/api/searchservice/create-synonym-map) или пакет Azure SDK. Для разработчиков на C# мы рекомендуем начать с [добавления синонимов в функции поиска в Azure с помощью c#](search-synonyms-tutorial-sdk.md).

## <a name="define-rules"></a>Определение правил

Правила сопоставления соответствуют спецификации фильтра синонимов с открытым исходным кодом в Apache Solr, описанном в этом документе: [синонимфилтер](https://cwiki.apache.org/confluence/display/solr/Filter+Descriptions#FilterDescriptions-SynonymFilter). `solr` Формат поддерживает два вида правил:

+ эквивалентность (где условия совпадают с заменяющими в запросе)

+ явные сопоставления (где условия сопоставляются с одним явным термином до запроса)

Каждое правило должно быть разделено символом новой строки ( `\n` ). Вы можете определить до 5 000 правил на карту синонимов в бесплатной службе и правила 20 000 на карте на других уровнях. Каждое правило может иметь до 20 расширений (или элементов в правиле). Дополнительные сведения см. в разделе [ограничения синонимов](search-limits-quotas-capacity.md#synonym-limits).

Средства синтаксического анализа запросов будут иметь меньший регистр любых прописных или строчных букв, но если вы хотите сохранить в строке специальные символы, такие как запятая или тире, добавьте соответствующие escape-символы при создании сопоставлений синонимов.

### <a name="equivalency-rules"></a>Правила эквивалентности

Правила для эквивалентных терминов разделяются запятыми в рамках одного правила. В первом примере запрос к `USA` будет расширен до `USA` или `"United States"` или `"United States of America"` . Обратите внимание, что если требуется совпадение по фразе, сам запрос должен быть запросом фразы, заключенными в кавычки.

В случае эквивалентности запрос для `dog` развернет запрос, который также будет включать `puppy` и `canine` .

```json
{
"format": "solr",
"synonyms": "
    USA, United States, United States of America\n
    dog, puppy, canine\n
    coffee, latte, cup of joe, java\n"
}
```

### <a name="explicit-mapping"></a>Явное сопоставление

Правила для явного сопоставления обозначаются стрелкой `=>` . Если этот параметр указан, последовательность терминов поискового запроса, которая соответствует левой части, `=>` будет заменена альтернативными вариантами в правой части запроса.

В явном регистре запрос для `Washington` `Wash.` или `WA` будет перезаписан как `WA` , а механизм запросов будет искать совпадения только в термине `WA` . Явное сопоставление применяется только в указанном направлении и не переписывает запрос в `WA` `Washington` в этом случае.

```json
{
"format": "solr",
"synonyms": "
    Washington, Wash., WA => WA\n
    California, Calif., CA => CA\n"
}
```

### <a name="escaping-special-characters"></a>Экранирование специальных знаков

Синонимы анализируются во время обработки запроса. Если необходимо определить синонимы, содержащие запятые или другие специальные символы, их можно отформатировать с помощью обратной косой черты, как в следующем примере:

```json
{
"format": "solr",
"synonyms": "WA\, USA, WA, Washington\n"
}
```

Так как обратная косая черта является специальным символом в других языках, таких как JSON и C#, вам, вероятно, потребуется дважды его escape-последовательность. Например, формат JSON, отправленный в REST API для приведенной выше карте синонимов, будет выглядеть следующим образом:

```json
{
"format":"solr",
"synonyms": "WA\\, USA, WA, Washington"
}
```

## <a name="upload-and-manage-synonym-maps"></a>Отправка и управление сопоставлений синонимов

Как упоминалось ранее, можно создать или обновить карту синонимов, не нарушая рабочие нагрузки запросов и индексирования. Сопоставленный синоним — это автономный объект (например, индексы или источники данных), и пока ни одно из полей не использует его, обновления не приведут к сбою индексирования или запросов. Однако после добавления сопоставлений синонимов в определение поля, если затем удалить карту синонимов, любой запрос, содержащий рассматриваемые поля, завершится ошибкой 404.

Создание, обновление и удаление сопоставлений синонимов всегда является операцией с целым документом, то есть вы не можете постепенно обновлять или удалять части схемы синонимов. Для обновления даже одного правила требуется перезагрузка.

## <a name="assign-synonyms-to-fields"></a>Назначение синонимов полям

После отправки на карту синонимов можно включить синонимы для полей типа `Edm.String` или `Collection(Edm.String)` , в полях, имеющих значение `"searchable":true` . Как уже отмечалось, определение поля может использовать только одну карту синонимов.

```http
POST /indexes?api-version=2020-06-30
{
    "name":"hotels-sample-index",
    "fields":[
        {
            "name":"description",
            "type":"Edm.String",
            "searchable":true,
            "synonymMaps":[
            "en-synonyms"
            ]
        },
        {
            "name":"description_fr",
            "type":"Edm.String",
            "searchable":true,
            "analyzer":"fr.microsoft",
            "synonymMaps":[
            "fr-synonyms"
            ]
        }
    ]
}
```

## <a name="query-on-equivalent-or-mapped-fields"></a>Запрос к эквивалентному или сопоставленному полю

Добавление синонимов не накладывает новых требований на построение запроса. Запросы терминов и фраз можно выдавать так же, как и перед добавлением синонимов. Единственное отличие состоит в том, что если термин запроса существует на карте синонимов, обработчик запросов развернет или перезапишет термин или фразу в зависимости от правила.

## <a name="how-synonyms-are-used-during-query-execution"></a>Использование синонимов во время выполнения запроса

Синонимы — это метод расширения запросов, который дополняет содержимое индекса эквивалентными условиями, но только для полей, которые имеют присвоение синонимов. Если запрос в области полей *исключает* поле с поддержкой синонимов, то совпадения с картой синонимов отображаться не будут.

Для полей с поддержкой синонимов синонимы подчиняются тому же текстовому анализу, что и связанное поле. Например, если поле анализируется с помощью стандартного анализатора Lucene, термины-синонимы также будут подвергаться стандартному анализатору Lucene во время выполнения запроса. Если вы хотите сохранить знаки препинания (например, точки или тире) в термине синонима, примените к этому полю анализатор сохранения содержимого.

На внутреннем уровне функция синонимов переписывает исходный запрос с синонимами с помощью оператора или. По этой причине функция выделения совпадений и профиль повышения считают исходный термин и его синонимы эквивалентными.

Синонимы применяются только к текстовым запросам в свободной форме и не поддерживаются для фильтров, аспектов, автозаполнения и предложений. Автозаполнение и предложения основаны только на исходном термине. совпадения синонимов не отображаются в ответе.

Расширения синонимов не применяются к поисковым терминам с подстановочными знаками. Это также относится к терминам, содержащим префикс, нечеткое условие и регулярное выражение.

Если вам нужно сделать один запрос, который применяет расширение синонимов, подстановочные знаки, регулярные выражения или нечеткий поиск, можно объединить запросы с использованием синтаксиса OR. Например, чтобы объединить синонимы с подстановочными знаками для обеспечения простого синтаксиса запросов, термин будет выглядеть так: `<query> | <query>*`.

Если у вас есть индекс в среде разработки (не в рабочей среде), поэкспериментируйте с небольшим словарем, чтобы узнать, как добавление синонимов изменяет результаты поиска, включая влияние на профили повышения, выделение совпадений и предложения.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Создание сопоставлений синонимов (REST API)](/rest/api/searchservice/create-synonym-map)