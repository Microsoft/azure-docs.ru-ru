---
title: Преобразование входных данных в поля вывода
titleSuffix: Azure Cognitive Search
description: Извлеките и дополните поля исходных данных и сопоставьте их с полями вывода в индексе Azure Когнитивный поиск.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 58bb87d5af785d3cffd96f3bd02477f97ed967a9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96001309"
---
# <a name="how-to-map-ai-enriched-fields-to-a-searchable-index"></a>Как сопоставлять ИСКУССТВЕНные поля с индексом с возможностью поиска

![Этапы индексатора](./media/cognitive-search-output-field-mapping/indexer-stages-output-field-mapping.png "этапы индексатора")

В этой статье вы узнаете, как сопоставлять обогащенные поля входных данных с полями выходных данных в индексе, поддерживающем поиск. После [определения набора навыков](cognitive-search-defining-skillset.md) вы должны сопоставить поля выходных данных любого навыка, который напрямую вносит значения в заданное поле в ваш индекс поиска.

Сопоставления полей вывода необходимы для перемещения содержимого из обогащенных документов в индекс.  Обогащенный документ на самом деле представляет собой дерево информации, и несмотря на то, что в индексе имеется поддержка сложных типов, иногда может потребоваться преобразовать данные из расширенного дерева в более простой тип (например, массив строк). Сопоставления полей вывода позволяют выполнять преобразования фигур данных, образуя данные. Сопоставления полей вывода всегда происходят после выполнения набора навыков, хотя этот этап может выполняться, даже если набор умений не определен.

Примеры сопоставлений полей вывода:

* В рамках набора навыков вы извлекли названия организаций, упомянутых на каждой из страниц документа. Теперь необходимо сопоставлять каждое из этих имен организации с полем в индексе типа EDM. Collection (EDM. String).

* В рамках набора навыков вы создавали новый узел с именем "Document/translated_text". Вы хотите сопоставлять сведения об этом узле с конкретным полем в индексе.

* У вас нет набора навыков, но выполняется индексирование сложного типа из базы данных Cosmos DB. Вы хотите перейти к узлу этого сложного типа и преобразовать его в поле в индексе.

> [!NOTE]
> Недавно мы включили функции сопоставления для сопоставления полей вывода. Дополнительные сведения о функциях сопоставления см. в разделе [функции сопоставления полей](./search-indexer-field-mappings.md#field-mapping-functions) .

## <a name="use-outputfieldmappings"></a>Использование outputFieldMappings

Чтобы сопоставить поля, добавьте `outputFieldMappings` в определение индексатора, как показано ниже.

```http
PUT https://[servicename].search.windows.net/indexers/[indexer name]?api-version=2020-06-30
api-key: [admin key]
Content-Type: application/json
```

Текст запроса имеет следующую структуру:

```json
{
    "name": "myIndexer",
    "dataSourceName": "myDataSource",
    "targetIndexName": "myIndex",
    "skillsetName": "myFirstSkillSet",
    "fieldMappings": [
        {
            "sourceFieldName": "metadata_storage_path",
            "targetFieldName": "id",
            "mappingFunction": {
                "name": "base64Encode"
            }
        }
    ],
    "outputFieldMappings": [
        {
            "sourceFieldName": "/document/content/organizations/*/description",
            "targetFieldName": "descriptions",
            "mappingFunction": {
                "name": "base64Decode"
            }
        },
        {
            "sourceFieldName": "/document/content/organizations",
            "targetFieldName": "orgNames"
        },
        {
            "sourceFieldName": "/document/content/sentiment",
            "targetFieldName": "sentiment"
        }
    ]
}
```

Для каждого сопоставления полей вывода задайте расположение данных в расширенном дереве документов (Саурцефиелднаме) и имя поля, на которое указывает ссылка в индексе (Таржетфиелднаме).

## <a name="flattening-information-from-complex-types"></a>Сведение информации из сложных типов 

Путь в sourceFieldName может представлять один элемент или несколько элементов. В приведенном выше примере ```/document/content/sentiment``` представляет одно числовое значение, а ```/document/content/organizations/*/description``` представляет несколько описаний организации. 

В случаях с несколькими элементами они преобразуются в массив, содержащий все элементы. 

Более конкретно, данные для примера ```/document/content/organizations/*/description``` в поле *описаний* будут выглядеть как плоский массив описаний до его индексации:

```
 ["Microsoft is a company in Seattle","LinkedIn's office is in San Francisco"]
```

Это важный принцип, поэтому мы будем предоставлять другой пример. Представьте себе, что у вас есть массив сложных типов в составе дерева обогащения. Предположим, что существует член с именем Кустоментитиес, который имеет массив сложных типов, как показано ниже.

```json
"document/customEntities": 
[
    {
        "name": "heart failure",
        "matches": [
            {
                "text": "heart failure",
                "offset": 10,
                "length": 12,
                "matchDistance": 0.0
            }
        ]
    },
    {
        "name": "morquio",
        "matches": [
            {
                "text": "morquio",
                "offset": 25,
                "length": 7,
                "matchDistance": 0.0
            }
        ]
    }
    //...
]
```

Предположим, что у индекса есть поле с именем "заболеваниях" типа Collection (EDM. String), где вы хотите сохранить каждое из имен сущностей. 

Это можно легко сделать с помощью \* символа "" следующим образом:

```json
    "outputFieldMappings": [
        {
            "sourceFieldName": "/document/customEntities/*/name",
            "targetFieldName": "diseases"
        }
    ]
```

Эта операция просто "выполняет сведение" всех имен элементов Кустоментитиес в один массив строк следующего вида:

```json
  "diseases" : ["heart failure","morquio"]
```

## <a name="next-steps"></a>Дальнейшие действия
После сопоставления обогащенных полей с полями, поддерживающими поиск, для каждого поля, поддерживающего поиск, можно задать атрибуты поля [в качестве части определения индекса](search-what-is-an-index.md).

Дополнительные сведения о сопоставлении полей см. [в разделе сопоставления полей в Azure когнитивный Поиск индексаторы](search-indexer-field-mappings.md).