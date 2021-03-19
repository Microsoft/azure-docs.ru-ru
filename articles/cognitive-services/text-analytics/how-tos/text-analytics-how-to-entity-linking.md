---
title: Использование распознавания сущностей с помощью API анализа текста
titleSuffix: Azure Cognitive Services
description: Узнайте, как идентифицировать и устранить неоднозначность удостоверения сущности, найденной в тексте с помощью REST API Анализ текста.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: article
ms.date: 03/15/2021
ms.author: aahi
ms.openlocfilehash: 0876dd00933203c943417d87978567cf555a3e4f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104599006"
---
# <a name="how-to-use-named-entity-recognition-in-text-analytics"></a>Как использовать распознавание именованных сущностей в Анализ текста

API анализа текста позволяет получить неструктурированный текст и возвращает список неоднозначного количества сущностей со ссылками на дополнительные сведения в Интернете. API поддерживает как распознавание имен сущностей (NER) для нескольких категорий сущностей, так и связывание сущностей.

## <a name="entity-linking"></a>Связывание сущностей

Связывание сущностей — это возможность определить и устранить неоднозначность идентификации сущности, найденной в тексте (например, определить, относится ли вхождение слова «Mars» к планетам или Roman божественные War). Для этого процесса требуется наличие базы знаний на соответствующем языке, чтобы связать распознанные сущности в тексте. При связывании сущностей в качестве этой базы знаний используется [Википедии](https://www.wikipedia.org/) .

## <a name="named-entity-recognition-ner"></a>Распознавание именованных сущностей (NER)

Распознавание имен сущностей (NER) — это возможность определения различных сущностей в тексте и категоризация их по заранее определенным классам или типам, таким как "пользователь", "расположение", "событие", "продукт" и "Организация".  

## <a name="personally-identifiable-information-pii"></a>личные сведения (PII).

Функция PII является частью NER и может находить и изменять конфиденциальные сущности в тексте, связанном с отдельным пользователем, например: номер телефона, адрес электронной почты, почтовый адрес, номер паспорта.

## <a name="named-entity-recognition-features-and-versions"></a>Функции и версии распознавания сущностей

| Компонент                                                         | NER версии 3.0 | NER версии 3.1 – Preview. 4 |
|-----------------------------------------------------------------|--------|----------|
| Методы для одиночных и пакетных запросов                          | X      | X        |
| Расширенное распознавание сущностей в нескольких категориях           | X      | X        |
| Отдельные конечные точки для отправки запросов на связывание сущностей и NER. | X      | X        |
| Распознавание личных `PII` объектов () и сведений о работоспособности ( `PHI` )        |        | X        |
| Исправление `PII`        |        | X        |

Дополнительные сведения см. в разделе [Поддержка языков](../language-support.md) .

Распознавание именованных сущностей v3 обеспечивает Расширенное обнаружение для нескольких типов. В настоящее время NER v 3.0 может распознавать сущности в [категории общих сущностей](../named-entity-types.md).

Распознавание имен сущностей версии 3.1 – Preview. 4 включает возможности обнаружения версий 3.0 и: 
* Возможность обнаружения персональных данных ( `PII` ) с помощью `v3.1-preview.4/entities/recognition/pii` конечной точки. 
* Необязательный `domain=phi` параметр для обнаружения конфиденциальных сведений о работоспособности ( `PHI` ).
* [Асинхронная операция](text-analytics-how-to-call-api.md) с использованием `/analyze` конечной точки.

Дополнительные сведения см. в разделе статьи о [категориях сущностей](../named-entity-types.md) и [запрос конечных точек](#request-endpoints) ниже. Дополнительные сведения о рейтинге достоверности см. в разделе [анализ текстаная заметка о прозрачности](/legal/cognitive-services/text-analytics/transparency-note?context=/azure/cognitive-services/text-analytics/context/context). 

## <a name="sending-a-rest-api-request"></a>Отправка запроса к REST API

### <a name="preparation"></a>Подготовка

Необходимо иметь документы JSON в следующем формате: ID, Text, Language.

Длина каждого документа не должна превышать 5 120 символов. в каждой коллекции может быть до 1 000 элементов (идентификаторов). Коллекция передается в тексте запроса.

### <a name="structure-the-request"></a>Структурирование запроса

Создайте запрос POST. Вы можете [использовать POST](text-analytics-how-to-call-api.md) или **консоль тестирования API** в следующих ссылках, чтобы быстро структурировать и отправить ее. 

> [!NOTE]
> Ключ и конечную точку для ресурса Анализа текста можно найти на портале Azure. Они находятся на странице ресурса **Быстрый запуск** в разделе **Управление ресурсами**. 


### <a name="request-endpoints"></a>Конечные точки запроса

#### <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-preview)

При распознавании именованных сущностей `v3.1-preview.4` используются отдельные конечные точки для NER, PII и запросов на связывание сущностей. Используйте следующий формат URL-адреса в зависимости от вашего запроса.

**Связывание сущностей**
* `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.1-preview.4/entities/linking`

[Распознавание именованных сущностей версии 3,1 — Предварительная версия справочника для `Linking`](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-Preview-4/operations/EntitiesLinking)

**Распознавание именованных сущностей**
* Общие сущности — `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.1-preview.4/entities/recognition/general`

[Распознавание именованных сущностей версии 3,1 — Предварительная версия справочника для `General`](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-Preview-4/operations/EntitiesRecognitionGeneral)

**личные сведения (PII).**
* Личные `PII` сведения () — `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.1-preview.4/entities/recognition/pii`

Кроме того, можно использовать необязательный `domain=phi` параметр для обнаружения `PHI` сведений о работоспособности () в тексте. 

`https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.1-preview.4/entities/recognition/pii?domain=phi`

Начиная с `v3.1-preview.4` , в ответе JSON содержится `redactedText` свойство, содержащее измененный входной текст, в котором обнаруженные персональные сущности заменяются на `*` для каждого символа в сущностях.

[Распознавание именованных сущностей версии 3,1 — Предварительная версия справочника для `PII`](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-Preview-4/operations/EntitiesRecognitionPii)

API будет пытаться обнаружить [указанные категории сущностей](../named-entity-types.md?tabs=personal) для данного языка документа. Если необходимо указать, какие сущности будут обнаружены и возвращены, используйте необязательный параметр PII-Categories с соответствующими категориями сущностей. Этот параметр также позволяет обнаруживать сущности, которые не включены по умолчанию для языка документа. Например, номер водительского номера драйвера для французского языка, который может возникнуть в тексте на английском языке.

`https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.1-preview.4/entities/recognition/pii?piiCategories=[FRDriversLicenseNumber]`

**Асинхронная операция**

Начиная `v3.1-preview.4` с, запросы на связывание NER и сущности можно передавать асинхронно с помощью `/analyze` конечной точки.

* Асинхронная операция — `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.1-preview.4/analyze`

Сведения об отправке асинхронных запросов см. в разделе [вызов API анализа текста](text-analytics-how-to-call-api.md) .

#### <a name="version-30"></a>[Версия 3.0](#tab/version-3)

В примере распознавания сущностей v3 используются отдельные конечные точки для запросов NER и связывания сущностей. Используйте следующий формат URL-адреса в зависимости от вашего запроса:

**Связывание сущностей**
* `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/entities/linking`

[Справочник по именованным сущностям версии 3,0 для `Linking`](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/EntitiesRecognitionGeneral)

**Распознавание именованных сущностей**
* `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/entities/recognition/general`

[Справочник по именованным сущностям версии 3,0 для `General`](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/EntitiesRecognitionGeneral)

---

Включите ключ API Анализа текста в заголовок запроса. В тексте запроса укажите подготовленные документы JSON.

## <a name="example-requests"></a>Примеры запросов

#### <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-preview)

### <a name="example-synchronous-ner-request"></a>Пример синхронного запроса NER 

Следующий код JSON является примером содержимого, которое можно отправить в API. Формат запроса одинаков для обеих версий API.

```json
{
  "documents": [
    {
        "id": "1",
        "language": "en",
        "text": "Our tour guide took us up the Space Needle during our trip to Seattle last week."
    }
  ]
}
```

### <a name="example-asynchronous-ner-request"></a>Пример асинхронного запроса NER

При использовании `/analyze` конечной точки для [асинхронной операции](text-analytics-how-to-call-api.md)вы получите ответ, содержащий задачи, отправленные в API.

```json
{
    "displayName": "My Job",
    "analysisInput": {
        "documents": [
            {
                "id": "doc1",
                "text": "It's incredibly sunny outside! I'm so happy"
            },
            {
                "id": "doc2",
                "text": "Pike place market is my favorite Seattle attraction."
            }
        ]
    },
    "tasks": {
        "entityRecognitionTasks": [
            {
                "parameters": {
                    "model-version": "latest",
                    "stringIndexType": "TextElements_v8"
                }
            }
        ],
        "entityRecognitionPiiTasks": [{
            "parameters": {
                "model-version": "latest"
            }
        }]
    }
}
```

#### <a name="version-30"></a>[Версия 3.0](#tab/version-3)

### <a name="example-synchronous-ner-request"></a>Пример синхронного запроса NER 

Версия 3,0 включает только синхронные операции. Следующий код JSON является примером содержимого, которое можно отправить в API. Формат запроса одинаков для обеих версий API.

```json
{
  "documents": [
    {
        "id": "1",
        "language": "en",
        "text": "Our tour guide took us up the Space Needle during our trip to Seattle last week."
    }
  ]
}
```

---

## <a name="post-the-request"></a>Передача запроса

Анализ выполняется при получении запроса. Сведения о размере и числе запросов, которые можно отправлять в минуту и секунду, см. в разделе об [ограничениях данных](../overview.md#data-limits).

API Анализа текста не учитывает состояние. Никакие данные в учетной записи не сохраняются, а все результаты немедленно возвращаются в ответе.

## <a name="view-results"></a>Просмотр результатов

Все запросы POST возвращают отформатированный ответ JSON с идентификаторами и обнаруженными свойствами сущности.

Вывод возвращается немедленно. Можно выполнить потоковую передачу результатов в приложение, которое принимает JSON, или сохранить выходные данные в файле локальной системы, а затем импортировать их в приложение, которое дает возможность сортировки, поиска данных и управления ими. В связи с поддержкой многоязычности и эмодзи, ответ может содержать смещения текста. Дополнительные сведения см. [в разделе Обработка смещения текста](../concepts/text-offsets.md).

### <a name="example-responses"></a>Примеры ответов

Версия 3 предоставляет отдельные конечные точки для общих NER, PII и связывания сущностей. Версия 3,1-паревиев включает режим асинхронного анализа. Ниже приведены ответы на эти операции. 

#### <a name="version-31-preview"></a>[Версия 3.1 (предварительная версия)](#tab/version-3-preview)

### <a name="synchronous-example-results"></a>Синхронные примеры результатов

Пример общего ответа NER:

```json
{
  "documents": [
    {
      "id": "1",
      "entities": [
        {
          "text": "tour guide",
          "category": "PersonType",
          "offset": 4,
          "length": 10,
          "confidenceScore": 0.45
        },
        {
          "text": "Space Needle",
          "category": "Location",
          "offset": 30,
          "length": 12,
          "confidenceScore": 0.38
        },
        {
          "text": "trip",
          "category": "Event",
          "offset": 54,
          "length": 4,
          "confidenceScore": 0.78
        },
        {
          "text": "Seattle",
          "category": "Location",
          "subcategory": "GPE",
          "offset": 62,
          "length": 7,
          "confidenceScore": 0.78
        },
        {
          "text": "last week",
          "category": "DateTime",
          "subcategory": "DateRange",
          "offset": 70,
          "length": 9,
          "confidenceScore": 0.8
        }
      ],
      "warnings": []
    }
  ],
  "errors": [],
  "modelVersion": "2020-04-01"
}
```

Пример ответа с PII-откликом:

```json
{
  "documents": [
    {
    "redactedText": "You can even pre-order from their online menu at *************************, call ************ or send email to ***************************!",
    "id": "0",
    "entities": [
        {
        "text": "www.contososteakhouse.com",
        "category": "URL",
        "offset": 49,
        "length": 25,
        "confidenceScore": 0.8
        }, 
        {
        "text": "312-555-0176",
        "category": "Phone Number",
        "offset": 81,
        "length": 12,
        "confidenceScore": 0.8
        }, 
        {
        "text": "order@contososteakhouse.com",
        "category": "Email",
        "offset": 111,
        "length": 27,
        "confidenceScore": 0.8
        }
      ],
    "warnings": []
    }
  ],
  "errors": [],
  "modelVersion": "2020-07-01"
}
```

Пример ответа на связывание сущности:

```json
{
  "documents": [
    {
      "id": "1",
      "entities": [
        {
          "bingId": "f8dd5b08-206d-2554-6e4a-893f51f4de7e", 
          "name": "Space Needle",
          "matches": [
            {
              "text": "Space Needle",
              "offset": 30,
              "length": 12,
              "confidenceScore": 0.4
            }
          ],
          "language": "en",
          "id": "Space Needle",
          "url": "https://en.wikipedia.org/wiki/Space_Needle",
          "dataSource": "Wikipedia"
        },
        {
          "bingId": "5fbba6b8-85e1-4d41-9444-d9055436e473",
          "name": "Seattle",
          "matches": [
            {
              "text": "Seattle",
              "offset": 62,
              "length": 7,
              "confidenceScore": 0.25
            }
          ],
          "language": "en",
          "id": "Seattle",
          "url": "https://en.wikipedia.org/wiki/Seattle",
          "dataSource": "Wikipedia"
        }
      ],
      "warnings": []
    }
  ],
  "errors": [],
  "modelVersion": "2020-02-01"
}
```

### <a name="example-asynchronous-result"></a>Пример асинхронного результата

```json
{
  "displayName": "My Analyze Job",
  "jobId": "dbec96a8-ea22-4ad1-8c99-280b211eb59e_637408224000000000",
  "lastUpdateDateTime": "2020-11-13T04:01:14Z",
  "createdDateTime": "2020-11-13T04:01:13Z",
  "expirationDateTime": "2020-11-14T04:01:13Z",
  "status": "running",
  "errors": [],
  "tasks": {
      "details": {
          "name": "My Analyze Job",
          "lastUpdateDateTime": "2020-11-13T04:01:14Z"
      },
      "completed": 1,
      "failed": 0,
      "inProgress": 2,
      "total": 3,
      "keyPhraseExtractionTasks": [
          {
              "name": "My Analyze Job",
              "lastUpdateDateTime": "2020-11-13T04:01:14.3763516Z",
              "results": {
                  "inTerminalState": true,
                  "documents": [
                      {
                          "id": "doc1",
                          "keyPhrases": [
                              "sunny outside"
                          ],
                          "warnings": []
                      },
                      {
                          "id": "doc2",
                          "keyPhrases": [
                              "favorite Seattle attraction",
                              "Pike place market"
                          ],
                          "warnings": []
                      }
                  ],
                  "errors": [],
                  "modelVersion": "2020-07-01"
              }
          }
      ]
  }
}
```


#### <a name="version-30"></a>[Версия 3.0](#tab/version-3)

Пример общего ответа NER:
```json
{
  "documents": [
    {
      "id": "1",
      "entities": [
        {
          "text": "tour guide",
          "category": "PersonType",
          "offset": 4,
          "length": 10,
          "confidenceScore": 0.45
        },
        {
          "text": "Space Needle",
          "category": "Location",
          "offset": 30,
          "length": 12,
          "confidenceScore": 0.38
        },
        {
          "text": "trip",
          "category": "Event",
          "offset": 54,
          "length": 4,
          "confidenceScore": 0.78
        },
        {
          "text": "Seattle",
          "category": "Location",
          "subcategory": "GPE",
          "offset": 62,
          "length": 7,
          "confidenceScore": 0.78
        },
        {
          "text": "last week",
          "category": "DateTime",
          "subcategory": "DateRange",
          "offset": 70,
          "length": 9,
          "confidenceScore": 0.8
        }
      ],
      "warnings": []
    }
  ],
  "errors": [],
  "modelVersion": "2020-04-01"
}
```

---


## <a name="summary"></a>Итоги

В этой статье рассматриваются основные понятия и рабочий процесс связывания сущностей в службе Текстовой аналитики Cognitive Services. В разделе "Сводка" сделайте следующее.

* В тексте запроса передаются документы JSON, которые содержат идентификатор, текст и код языка.
* Запросы POST отправляются на одну или несколько конечных точек, используя персонализированный [ключ доступа и конечную точку](../../cognitive-services-apis-create-account.md#get-the-keys-for-your-resource) , допустимую для вашей подписки.
* Ответ будет содержать выходные данные, содержащие связанные сущности (включая оценки уверенности, смещения и веб-ссылки для каждого идентификатора документа), которые можно использовать в любом приложении

## <a name="next-steps"></a>Дальнейшие действия

* [Text Analytics overview](../overview.md) (Общие сведения об анализе текста)
* [Использование клиентской библиотеки Анализа текста](../quickstarts/client-libraries-rest-api.md)
* [Новые возможности](../whats-new.md)
