---
title: Использование Анализ текста для работоспособности
titleSuffix: Azure Cognitive Services
description: Узнайте, как извлекать и помечать медицинские сведения из неструктурированного текста клинической практике с Анализ текста для работоспособности.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 03/11/2021
ms.author: aahi
ms.custom: references_regions
ms.openlocfilehash: 80a943d235783852f57832363b5af8048f010575
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104599441"
---
# <a name="how-to-use-text-analytics-for-health-preview"></a>Как использовать Анализ текста для работоспособности (Предварительная версия)

> [!IMPORTANT] 
> Анализ текста для работоспособности — это возможность предварительной версии, предоставленная "как есть" и "со всеми ОШИБКАми". Таким образом, **анализ текста для работоспособности (Предварительная версия) не следует реализовывать и развертывать в любом рабочем использовании.** Анализ текста для обеспечения работоспособности не предназначены для использования в качестве медицинских устройств, поддержки клинической практике, диагностического средства или других технологий, предназначенных для диагностики, получения, устранения, обработки или предотвращения болезни или других условий, и ни одна из лицензий или прав не предоставляется корпорацией Майкрософт для использования этой возможности в таких целях. Эта возможность не предназначена для реализации или развертывания в качестве замены для профессиональных медицинских консультаций или здравоохранения, диагностики, лечения или клинической практике, которые являются нежелательными специалистами здравоохранения и не должны использоваться. Клиент несет ответственность за использование Анализ текста для работоспособности. Корпорация Майкрософт не гарантирует, что Анализ текста для обеспечения работоспособности или каких бы то ни было материалов, предоставляемых в связи с этой возможностью, будет достаточно для любых медицинских целей или иным образом соответствовать требованиям или медицинских нужд любого человека. 


Анализ текста для работоспособности — это функция службы API анализа текста, которая извлекает и помечает соответствующие медицинские сведения из неструктурированных текстов, таких как заметки Doctor, сводки разрядов, документы клинической практике и электронные записи о работоспособности.  Использовать эту службу можно двумя способами: 

* [Веб-API (асинхронный)](#structure-the-api-request-for-the-hosted-asynchronous-web-api)
* [Контейнер DOCKER (синхронный);](#hosted-asynchronous-web-api-response)   

> [!VIDEO https://channel9.msdn.com/Shows/AI-Show/Introducing-Text-Analytics-for-Health/player]

## <a name="features"></a>Компоненты

Анализ текста для проверки работоспособности: распознавание сущностей (NER), извлечение отношений, отрицание сущностей и связывание сущностей на английском языке для получения аналитических сведений в неструктурированном клинической практике и специальном тексте.

### <a name="named-entity-recognition"></a>[Распознавание именованных сущностей](#tab/ner)

Распознавание именованных сущностей обнаруживает слова и фразы, упомянутые в неструктурированном тексте, которые могут быть связаны с одним или несколькими семантическими типами, такими как диагностика, имя лечения, симптом, подпись или возраст.

> [!div class="mx-imgBorder"]
> ![NER работоспособности](../media/ta-for-health/health-named-entity-recognition.png)

### <a name="relation-extraction"></a>[Извлечение связей](#tab/relation-extraction)

Извлечение связей определяет осмысленные соединения между понятиями, упомянутыми в тексте. Например, отношение "время выполнения" обнаруживается путем сопоставления имени условия со временем или между сокращением и полным описанием.  

> [!div class="mx-imgBorder"]
> ![Повтор работоспособности](../media/ta-for-health/health-relation-extraction.png)


### <a name="entity-linking"></a>[Связывание сущностей](#tab/entity-linking)

Сущность, связывающая сущности различения, сопоставляя именованные сущности, упомянутые в тексте, с концепциями, найденными в предопределенной базе данных концепций, включая унифицированную систему медицинских языков (УМЛС). Медицинские понятия также назначают предпочтительное именование в качестве дополнительной формы нормализации.

> [!div class="mx-imgBorder"]
> ![Работоспособность EL](../media/ta-for-health/health-entity-linking.png)

Анализ текста для работоспособности поддерживает связывание с работоспособностью и биомедицинских словари, найденных в источнике знаний об метасесаурусе единой системы обработки медицинских языков ([умлс](https://www.nlm.nih.gov/research/umls/sourcereleasedocs/index.html)).

### <a name="assertion-detection"></a>[Обнаружение утверждения](#tab/assertion-detection) 

Значение медицинских материалов сильно зависит от модификаторов, например отрицательных или условных утверждений, которые могут иметь критические последствия, если они не представлены. Анализ текста для работоспособности поддерживает три категории обнаружения утверждения для сущностей в тексте: 

* уверен
* условные
* связь

> [!div class="mx-imgBorder"]
> ![РАСХОД работоспособности](../media/ta-for-health/assertions.png)

---

Полный список поддерживаемых сущностей см. в статье [категории сущностей](../named-entity-types.md?tabs=health) , возвращенных анализ текста. Сведения о рейтинге достоверности см. в разделе [анализ текстаная заметка о прозрачности](/legal/cognitive-services/text-analytics/transparency-note#general-guidelines-to-understand-and-improve-performance?context=/azure/cognitive-services/text-analytics/context/context). 

### <a name="supported-languages-and-regions"></a>Поддерживаемые языки и регионы

Анализ текста для работоспособности поддерживает только документы на английском языке. 

Анализ текста для веб-API, размещенного в состоянии работоспособности, сейчас доступны только в следующих регионах: Западная часть США 2, Восточная часть США 2, Центральная часть США, Северная Европа и Западная Европа.

## <a name="request-access-to-the-public-preview"></a>Запрос доступа к общедоступной предварительной версии

Заполните и отправьте [форму запроса Cognitive Services](https://aka.ms/csgate) , чтобы запросить доступ к анализ текста для общедоступной предварительной версии работоспособности. Вам не будет выставлен счет за Анализ текста для использования работоспособности. 

В форме нужно указать сведения о себе, компании и пользовательском сценарии, для которого будет использоваться контейнер. После отправки формы команда Azure Cognitive Services будет просматривать ее и отправлять вам решение.

> [!IMPORTANT]
> * В форме необходимо использовать адрес электронной почты, связанный с ИДЕНТИФИКАТОРом подписки Azure.
> * Используемый ресурс Azure должен быть создан с утвержденным ИДЕНТИФИКАТОРом подписки Azure. 
> * Проверьте электронную почту (папки "Входящие" и "спам") на наличие обновлений для приложения от корпорации Майкрософт.

## <a name="using-the-docker-container"></a>Использование контейнера DOCKER 

Чтобы запустить Анализ текста для контейнера работоспособности в своей среде, выполните следующие [инструкции, чтобы скачать и установить контейнер](../how-tos/text-analytics-how-to-install-containers.md?tabs=healthcare).

## <a name="using-the-client-library"></a>Использование клиентской библиотеки

Последняя Предварительная версия клиентской библиотеки Анализ текста позволяет вызывать Анализ текста для работоспособности с помощью клиентского объекта. См. справочную документацию и примеры на сайте GitHub:
* [C#](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/textanalytics/Azure.AI.TextAnalytics)
* [Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/textanalytics/azure-ai-textanalytics/)
* [Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/textanalytics/azure-ai-textanalytics)



## <a name="sending-a-rest-api-request"></a>Отправка запроса к REST API 

### <a name="preparation"></a>Подготовка

Анализ текста для работоспособности позволяет получить более качественный результат, если вы придаете ему меньшее количество текста для работы. Это противоположено некоторым другим Анализ текстаным функциям, таким как извлечение ключевых фраз, которые лучше подходят для больших блоков текста. Чтобы получить наилучшие результаты из этих операций, рассмотрите возможность реструктуризации входных данных соответствующим образом.

Необходимы документы JSON в следующем формате: ID, текст и язык. 

Размер документа должен быть менее 5120 символов. Максимальное число документов, которые могут содержаться в коллекции, см. в статье об [ограничениях данных](../concepts/data-limits.md?tabs=version-3) в разделе "Основные понятия". Коллекция передается в тексте запроса.

### <a name="structure-the-api-request-for-the-hosted-asynchronous-web-api"></a>Структурирование запроса API для размещенного асинхронного веб-API

Для контейнера и размещенного веб-API необходимо создать запрос POST. Чтобы быстро создать и отправить запрос POST в размещенный веб-API в нужном регионе, можно [воспользоваться командой POST](text-analytics-how-to-call-api.md), фигурой или **консолью тестирования API** в [анализ текста](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-3/operations/Health) . 

> [!NOTE]
> Асинхронные `/analyze` и `/health` конечные точки доступны только в следующих регионах: Западная часть США 2, Восточная часть США 2, Центральная часть США, Северная Европа и Западная Европа.  Для успешного выполнения запросов к этим конечным точкам убедитесь, что ресурс создан в одном из этих регионов.

Ниже приведен пример JSON-файла, присоединенного к Анализ текста для текста POST запроса API работоспособности:

```json
example.json

{
  "documents": [
    {
      "language": "en",
      "id": "1",
      "text": "Subject was administered 100mg remdesivir intravenously over a period of 120 min"
    }
  ]
}
```

### <a name="hosted-asynchronous-web-api-response"></a>Размещенный асинхронный ответ веб-API 

Так как этот запрос POST используется для отправки задания для асинхронной операции, в объекте Response нет текста.  Однако для выполнения запроса GET для проверки состояния задания и выходных данных требуется значение ключа Operation-Location в заголовках ответа.  Ниже приведен пример значения ключа Location операции в заголовке ответа запроса POST:

`https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.1-preview.4/entities/health/jobs/<jobID>`

Чтобы проверить состояние задания, выполните запрос GET к URL-адресу в заголовке ключа операции запроса POST.  Для отражения состояния задания используются следующие состояния: `NotStarted` , `running` ,,,, `succeeded` `failed` `rejected` `cancelling` и `cancelled` .  

Можно отменить задание с `NotStarted` состоянием или, `running` используя вызов HTTP DELETE с тем же URL-адресом, что и у запроса GET.  Дополнительные сведения о вызове DELETE доступны в [анализ текста для Справочника по API, размещенного в исправности](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-3/operations/CancelHealthJob).

Ниже приведен пример ответа на запрос GET.  Выходные данные доступны для получения до тех пор, пока не `expirationDateTime` пройдет время (24 часа со времени создания задания), после чего выходные данные будут очищены.

```json
{
    "jobId": "be437134-a76b-4e45-829e-9b37dcd209bf",
    "lastUpdateDateTime": "2021-03-11T05:43:37Z",
    "createdDateTime": "2021-03-11T05:42:32Z",
    "expirationDateTime": "2021-03-12T05:42:32Z",
    "status": "succeeded",
    "errors": [],
    "results": {
        "documents": [
            {
                "id": "1",
                "entities": [
                    {
                        "offset": 25,
                        "length": 5,
                        "text": "100mg",
                        "category": "Dosage",
                        "confidenceScore": 1.0
                    },
                    {
                        "offset": 31,
                        "length": 10,
                        "text": "remdesivir",
                        "category": "MedicationName",
                        "confidenceScore": 1.0,
                        "name": "remdesivir",
                        "links": [
                            {
                                "dataSource": "UMLS",
                                "id": "C4726677"
                            },
                            {
                                "dataSource": "DRUGBANK",
                                "id": "DB14761"
                            },
                            {
                                "dataSource": "GS",
                                "id": "6192"
                            },
                            {
                                "dataSource": "MEDCIN",
                                "id": "398132"
                            },
                            {
                                "dataSource": "MMSL",
                                "id": "d09540"
                            },
                            {
                                "dataSource": "MSH",
                                "id": "C000606551"
                            },
                            {
                                "dataSource": "MTHSPL",
                                "id": "3QKI37EEHE"
                            },
                            {
                                "dataSource": "NCI",
                                "id": "C152185"
                            },
                            {
                                "dataSource": "NCI_FDA",
                                "id": "3QKI37EEHE"
                            },
                            {
                                "dataSource": "NDDF",
                                "id": "018308"
                            },
                            {
                                "dataSource": "RXNORM",
                                "id": "2284718"
                            },
                            {
                                "dataSource": "SNOMEDCT_US",
                                "id": "870592005"
                            },
                            {
                                "dataSource": "VANDF",
                                "id": "4039395"
                            }
                        ]
                    },
                    {
                        "offset": 42,
                        "length": 13,
                        "text": "intravenously",
                        "category": "MedicationRoute",
                        "confidenceScore": 1.0
                    },
                    {
                        "offset": 73,
                        "length": 7,
                        "text": "120 min",
                        "category": "Time",
                        "confidenceScore": 0.94
                    }
                ],
                "relations": [
                    {
                        "relationType": "DosageOfMedication",
                        "entities": [
                            {
                                "ref": "#/results/documents/0/entities/0",
                                "role": "Dosage"
                            },
                            {
                                "ref": "#/results/documents/0/entities/1",
                                "role": "Medication"
                            }
                        ]
                    },
                    {
                        "relationType": "RouteOfMedication",
                        "entities": [
                            {
                                "ref": "#/results/documents/0/entities/1",
                                "role": "Medication"
                            },
                            {
                                "ref": "#/results/documents/0/entities/2",
                                "role": "Route"
                            }
                        ]
                    },
                    {
                        "relationType": "TimeOfMedication",
                        "entities": [
                            {
                                "ref": "#/results/documents/0/entities/1",
                                "role": "Medication"
                            },
                            {
                                "ref": "#/results/documents/0/entities/3",
                                "role": "Time"
                            }
                        ]
                    }
                ],
                "warnings": []
            }
        ],
        "errors": [],
        "modelVersion": "2021-03-01"
    }
}
```


### <a name="structure-the-api-request-for-the-container"></a>Структурирование запроса API для контейнера

Чтобы отправить запрос в развернутый контейнер, можно [использовать POST](text-analytics-how-to-call-api.md) или пример запроса с фигурными скобками, заменив `serverURL` переменную соответствующим значением.  Обратите внимание, что версия API в URL-адресе для контейнера отличается от размещенного API.

```bash
curl -X POST 'http://<serverURL>:5000/text/analytics/v3.2-preview.1/entities/health' --header 'Content-Type: application/json' --header 'accept: application/json' --data-binary @example.json

```

Ниже приведен пример JSON-файла, присоединенного к Анализ текста в теле запроса API работоспособности:

```json
example.json

{
  "documents": [
    {
      "language": "en",
      "id": "1",
      "text": "Patient reported itchy sores after swimming in the lake."
    },
    {
      "language": "en",
      "id": "2",
      "text": "Prescribed 50mg benadryl, taken twice daily."
    }
  ]
}
```

### <a name="container-response-body"></a>Тело ответа контейнера

Следующий код JSON является примером Анализ текста для текста ответа API работоспособности из контейнерного синхронного вызова:

```json
{
    "documents": [
        {
            "id": "1",
            "entities": [
                {
                    "offset": 25,
                    "length": 5,
                    "text": "100mg",
                    "category": "Dosage",
                    "confidenceScore": 1.0
                },
                {
                    "offset": 31,
                    "length": 10,
                    "text": "remdesivir",
                    "category": "MedicationName",
                    "confidenceScore": 1.0,
                    "name": "remdesivir",
                    "links": [
                        {
                            "dataSource": "UMLS",
                            "id": "C4726677"
                        },
                        {
                            "dataSource": "DRUGBANK",
                            "id": "DB14761"
                        },
                        {
                            "dataSource": "GS",
                            "id": "6192"
                        },
                        {
                            "dataSource": "MEDCIN",
                            "id": "398132"
                        },
                        {
                            "dataSource": "MMSL",
                            "id": "d09540"
                        },
                        {
                            "dataSource": "MSH",
                            "id": "C000606551"
                        },
                        {
                            "dataSource": "MTHSPL",
                            "id": "3QKI37EEHE"
                        },
                        {
                            "dataSource": "NCI",
                            "id": "C152185"
                        },
                        {
                            "dataSource": "NCI_FDA",
                            "id": "3QKI37EEHE"
                        },
                        {
                            "dataSource": "NDDF",
                            "id": "018308"
                        },
                        {
                            "dataSource": "RXNORM",
                            "id": "2284718"
                        },
                        {
                            "dataSource": "SNOMEDCT_US",
                            "id": "870592005"
                        },
                        {
                            "dataSource": "VANDF",
                            "id": "4039395"
                        }
                    ]
                },
                {
                    "offset": 42,
                    "length": 13,
                    "text": "intravenously",
                    "category": "MedicationRoute",
                    "confidenceScore": 1.0
                },
                {
                    "offset": 73,
                    "length": 7,
                    "text": "120 min",
                    "category": "Time",
                    "confidenceScore": 0.94
                }
            ],
            "relations": [
                {
                    "relationType": "DosageOfMedication",
                    "entities": [
                        {
                            "ref": "#/documents/0/entities/0",
                            "role": "Dosage"
                        },
                        {
                            "ref": "#/documents/0/entities/1",
                            "role": "Medication"
                        }
                    ]
                },
                {
                    "relationType": "RouteOfMedication",
                    "entities": [
                        {
                            "ref": "#/documents/0/entities/1",
                            "role": "Medication"
                        },
                        {
                            "ref": "#/documents/0/entities/2",
                            "role": "Route"
                        }
                    ]
                },
                {
                    "relationType": "TimeOfMedication",
                    "entities": [
                        {
                            "ref": "#/documents/0/entities/1",
                            "role": "Medication"
                        },
                        {
                            "ref": "#/documents/0/entities/3",
                            "role": "Time"
                        }
                    ]
                }
            ],
            "warnings": []
        }
    ],
    "errors": [],
    "modelVersion": "2021-03-01"
}
```

### <a name="assertion-output"></a>Выходные данные утверждения

Анализ текста для проверки работоспособности возвращает модификаторы утверждения, которые представляют собой информативные атрибуты, назначенные медицинских концепциям, которые предоставляют более глубокое представление о контексте концепций в тексте. Эти модификаторы делятся на три категории, каждое из которых сосредоточено на различных аспектах и содержат набор взаимоисключающих значений. Каждой сущности присваивается только одно значение каждой категории. Наиболее распространенным значением для каждой категории является значение по умолчанию. Выходной ответ службы содержит только модификаторы утверждения, отличные от значений по умолчанию.

**Защита**  . предоставляет сведения о присутствии (в настоящее время и отсутствии) концепции, а также о том, насколько текст относится к его присутствию (определенному и возможному).
*   **Положительный** [по умолчанию]: концепция существует или произошла.
* **Отрицательное**: концепция не существует или никогда не выполняется.
* **Positive_Possible**: вероятно, концепция существует, но есть какая-то неопределенность.
* **Negative_Possible**: существование концепции маловероятно, но есть какая-то неопределенность.
* **Neutral_Possible**: концепция может быть или не существовать без тенденции в обе стороны.

**Условность** — предоставляет сведения о том, зависит ли существование концепции от определенных условий. 
*   **None** [default]: концепция является фактом и не является гипотетической и не зависит от определенных условий.
*   **Гипотетический**: концепция может разрабатываться или происходить в будущем.
*   **Условное**: концепция существует или имеет место только при определенных условиях.

**Ассоциация** — описывает, связана ли концепция с темой текста или другим пользователем.
*   **Subject** [по умолчанию]: концепция связана с темой текста, обычно с пациентами.
*   **Someone_Else**: концепция связана с пользователем, который не является темой текста.


Функция обнаружения утверждений представляет сущности с отрицанием в виде отрицательного значения для категории «уверенность», например:

```json
{
                        "offset": 381,
                        "length": 3,
                        "text": "SOB",
                        "category": "SymptomOrSign",
                        "confidenceScore": 0.98,
                        "assertion": {
                            "certainty": "negative"
                        },
                        "name": "Dyspnea",
                        "links": [
                            {
                                "dataSource": "UMLS",
                                "id": "C0013404"
                            },
                            {
                                "dataSource": "AOD",
                                "id": "0000005442"
                            },
    ...
```

### <a name="relation-extraction-output"></a>Выход извлечения связи

Анализ текста для работоспособности распознает отношения между разными концепциями, включая связи между атрибутом и сущностью (например, направление структуры текста, досаже лечения) и между сущностями (например, определение сокращения).

**СОКРАЩЕНИЕ**

**DIRECTION_OF_BODY_STRUCTURE**

**DIRECTION_OF_CONDITION**

**DIRECTION_OF_EXAMINATION**

**DIRECTION_OF_TREATMENT**

**DOSAGE_OF_MEDICATION**

**FORM_OF_MEDICATION**

**FREQUENCY_OF_MEDICATION**

**FREQUENCY_OF_TREATMENT**

**QUALIFIER_OF_CONDITION**

**RELATION_OF_EXAMINATION**

**ROUTE_OF_MEDICATION** 

**TIME_OF_CONDITION**

**TIME_OF_EVENT**

**TIME_OF_EXAMINATION**

**TIME_OF_MEDICATION**

**TIME_OF_TREATMENT**

**UNIT_OF_CONDITION**

**UNIT_OF_EXAMINATION**

**VALUE_OF_CONDITION**  

**VALUE_OF_EXAMINATION**

> [!NOTE]
> * Связи, ссылающиеся на условие, могут ссылаться либо на тип сущности ДИАГНОЗа, либо на тип сущности SYMPTOM_OR_SIGN.
> * Связи, которые ссылаются на лечения, могут ссылаться либо на MEDICATION_NAME тип сущности, либо на MEDICATION_CLASS тип сущности.
> * Связи, которые ссылаются на время, могут ссылаться либо на тип сущности времени, либо на тип сущности DATE.

Выход извлечения связи содержит ссылки URI и назначенные роли сущностей типа связи. Пример:

```json
                "relations": [
                    {
                        "relationType": "DosageOfMedication",
                        "entities": [
                            {
                                "ref": "#/results/documents/0/entities/0",
                                "role": "Dosage"
                            },
                            {
                                "ref": "#/results/documents/0/entities/1",
                                "role": "Medication"
                            }
                        ]
                    },
                    {
                        "relationType": "RouteOfMedication",
                        "entities": [
                            {
                                "ref": "#/results/documents/0/entities/1",
                                "role": "Medication"
                            },
                            {
                                "ref": "#/results/documents/0/entities/2",
                                "role": "Route"
                            }
                        ]
...
]
```

## <a name="see-also"></a>См. также раздел

* [Text Analytics overview](../overview.md) (Общие сведения об анализе текста)
* [Категории именованных сущностей](../named-entity-types.md)
* [Новые возможности](../whats-new.md)
