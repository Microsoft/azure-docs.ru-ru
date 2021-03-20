---
title: Когнитивный навык формирователя
titleSuffix: Azure Cognitive Search
description: Извлеките метаданные и структурированную информацию из неструктурированных данных и соберите их как сложный тип в конвейере обогащения искусственного интеллекта в Azure Когнитивный поиск.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 81eb0e60befc544a6c3bee8f04e901b6a5e472bc
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85560814"
---
# <a name="shaper-cognitive-skill"></a>Когнитивный навык формирователя

Навык работы с **фигурой** объединяет несколько входов в [сложный тип](search-howto-complex-data-types.md) , на который можно ссылаться позже в конвейере обогащения. Навык « **фигура** » позволяет создать структуру, определить имя членов этой структуры и присвоить значения каждому элементу. Примеры консолидированных полей, полезных для сценариев поиска, включают объединение имени и фамилии в одну структуру, город и штат в одну структуру, а также имя и дату рождения в одну структуру для установления уникального удостоверения.

Кроме того, **в** [сценарии 3](#nested-complex-types) во входных данных добавляется необязательное свойство *саурцеконтекст* . Свойства *Source* и *саурцеконтекст* являются взаимоисключающими. Если входные данные находятся в контексте навыка, просто используйте *Source*. Если входные данные находятся в *контексте,* отличном от контекста навыка, используйте *саурцеконтекст*. *Саурцеконтекст* требует определить вложенные входные данные с определенным элементом, адресованным в качестве источника. 

Имя выхода всегда равно «Output». На внутреннем уровне конвейер может сопоставлять другое имя, например "Анализедтекст", как показано в приведенных ниже примерах, но сам по **себе навык возвращает** "Output" в ответе. Это может оказаться важным при обнаружении несоответствия имен во время отладки обогащенных документов или при создании пользовательского навыка с самостоятельным структурированием отклика.

> [!NOTE]
> Навык для **формы** не привязан к Cognitive Services API, и вы не платите за его использование. Но вам следует по-прежнему [подключать ресурс Cognitive Services](cognitive-search-attach-cognitive-services.md), чтобы переопределить **бесплатную** категорию ресурса, ограниченную малым количеством обогащений в день.

## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Util.ShaperSkill

## <a name="scenario-1-complex-types"></a>Сценарий 1. сложные типы

Рассмотрим сценарий, где требуется создать структуру *analyzedText* с двумя членами: *text* и *sentiment*, соответственно. В индексе многокомпонентное поле с возможностью поиска называется *сложным типом* и часто создается, когда исходные данные имеют соответствующую сложную структуру, сопоставленную с ней.

Однако еще один подход к созданию сложных типов — это с помощью навыка **Shape** . Благодаря включению этого навыка в набор навыков операции в памяти во время обработки набора навыков могут выводить фигуры данных с вложенными структурами, которые затем могут быть сопоставлены со сложным типом в индексе. 

В следующем примере определение навыка содержит имена членов в качестве входных данных. 


```json
{
  "@odata.type": "#Microsoft.Skills.Util.ShaperSkill",
  "context": "/document/content/phrases/*",
  "inputs": [
    {
      "name": "text",
      "source": "/document/content/phrases/*"
    },
    {
      "name": "sentiment",
      "source": "/document/content/phrases/*/sentiment"
    }
  ],
  "outputs": [
    {
      "name": "output",
      "targetName": "analyzedText"
    }
  ]
}
```

### <a name="sample-index"></a>Пример индекса

Набор навыков вызывается индексатором, а для индексатора требуется индекс. Представление сложного поля в индексе может выглядеть, как в следующем примере. 

```json

    "name": "my-index",
    "fields": [
        {   "name": "myId", "type": "Edm.String", "key": true, "filterable": true   },
        {   "name": "analyzedText", "type": "Edm.ComplexType",
            "fields": [{
                    "name": "text",
                    "type": "Edm.String",
                    "filterable": false,
                    "sortable": false,
                    "facetable": false,
                    "searchable": true  },
          {
                    "name": "sentiment",
                    "type": "Edm.Double",
                    "searchable": true,
                    "filterable": true,
                    "sortable": true,
                    "facetable": true
                },
```

### <a name="skill-input"></a>Ввод навыка

Входящий документ JSON, предоставляющий пригодные для использования входные данные **для этого** навыка, может быть следующим:

```json
{
    "values": [
        {
            "recordId": "1",
            "data": {
                "text": "this movie is awesome",
                "sentiment": 0.9
            }
        }
    ]
}
```


### <a name="skill-output"></a>Выходные данные навыка

Навык создания **фигуры** создает новый элемент с именем *анализедтекст* с объединенными элементами *Text* и *тональности*. Эти выходные данные соответствует схеме индекса. Он будет импортирован и проиндексирован в индексе Azure Когнитивный поиск.

```json
{
    "values": [
      {
        "recordId": "1",
        "data":
           {
            "analyzedText": 
              {
                "text": "this movie is awesome" ,
                "sentiment": 0.9
              }
           }
      }
    ]
}
```

## <a name="scenario-2-input-consolidation"></a>Сценарий 2. входная консолидация

В другом примере предположим, что на различных этапах обработки конвейера вы извлекли название книги и заголовки глав на разных страницах книги. Теперь вы можете создать единую структуру, состоящую из этих разнородных входных данных.

Определение **навыка** для этого сценария может выглядеть, как в следующем примере:

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ShaperSkill",
    "context": "/document",
    "inputs": [
        {
            "name": "title",
            "source": "/document/content/title"
        },
        {
            "name": "chapterTitles",
            "source": "/document/content/pages/*/chapterTitles/*/title"
        }
    ],
    "outputs": [
        {
            "name": "output",
            "targetName": "titlesAndChapters"
        }
    ]
}
```

### <a name="skill-output"></a>Выходные данные навыка
В этом случае **форма** выполняет сведение всех заголовков глав для создания одного массива. 

```json
{
    "values": [
        {
            "recordId": "1",
            "data": {
                "titlesAndChapters": {
                    "title": "How to be happy",
                    "chapterTitles": [
                        "Start young",
                        "Laugh often",
                        "Eat, sleep and exercise"
                    ]
                }
            }
        }
    ]
}
```

<a name="nested-complex-types"></a>

## <a name="scenario-3-input-consolidation-from-nested-contexts"></a>Сценарий 3. входная консолидация из вложенных контекстов

Представьте, что у вас есть заголовок, главы и содержимое книги, и вы запустили функцию распознавания сущностей и ключевые фразы на содержании и теперь должны выполнять статистическую обработку результатов из различных навыков в одну фигуру с названием главы, сущностями и ключевыми фразами.

Определение **навыка** для этого сценария может выглядеть, как в следующем примере:

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ShaperSkill",
    "context": "/document",
    "inputs": [
        {
            "name": "title",
            "source": "/document/content/title"
        },
        {
            "name": "chapterTitles",
            "sourceContext": "/document/content/pages/*/chapterTitles/*",
            "inputs": [
              {
                  "name": "title",
                  "source": "/document/content/pages/*/chapterTitles/*/title"
              },
              {
                  "name": "number",
                  "source": "/document/content/pages/*/chapterTitles/*/number"
              }
            ]
        }

    ],
    "outputs": [
        {
            "name": "output",
            "targetName": "titlesAndChapters"
        }
    ]
}
```

### <a name="skill-output"></a>Выходные данные навыка
В этом случае **форма** создает сложный тип. Эта структура существует в памяти. Если вы хотите сохранить его в [хранилище знаний](knowledge-store-concept-intro.md), в наборе навыков необходимо создать проекцию, определяющую характеристики хранилища.

```json
{
    "values": [
        {
            "recordId": "1",
            "data": {
                "titlesAndChapters": {
                    "title": "How to be happy",
                    "chapterTitles": [
                      { "title": "Start young", "number": 1},
                      { "title": "Laugh often", "number": 2},
                      { "title": "Eat, sleep and exercise", "number: 3}
                    ]
                }
            }
        }
    ]
}
```

## <a name="see-also"></a>См. также раздел

+ [Встроенные навыки](cognitive-search-predefined-skills.md)
+ [Определение набора навыков](cognitive-search-defining-skillset.md)
+ [Использование сложных типов](search-howto-complex-data-types.md)
+ [Хранилище знаний](knowledge-store-concept-intro.md)
+ [Создание хранилища знаний в REST](knowledge-store-create-rest.md)