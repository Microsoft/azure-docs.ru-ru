---
title: Извлечение ключевой фразы с помощью REST API Анализа текста
titleSuffix: Azure Cognitive Services
description: Узнайте, как извлечь ключевые фразы с помощью REST API Анализа текста из Azure Cognitive Services.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: article
ms.date: 12/17/2020
ms.author: aahi
ms.openlocfilehash: 91e10c25d2c3bef9c1ca20e3e5737a326d45997c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97654784"
---
# <a name="example-how-to-extract-key-phrases-using-text-analytics"></a>Пример. Извлечение ключевых фраз с помощью Анализ текста

[API извлечения ключевых фраз](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/KeyPhrases) оценивает неструктурированный текст и для каждого документа JSON возвращает список ключевых фраз.

Эта возможность полезна, если необходимо быстро определить основные тезисы в коллекции документов. Например, данный входной текст "Еда была вкусной, и были замечательные сотрудники", служба вернет основные тезисы в записи: "еда" и "замечательные сотрудники".

Дополнительные сведения см. в [списке поддерживаемых языков](../language-support.md).

> [!TIP]
> * API анализа текста также предоставляет образ контейнера Docker под управлением Linux для извлечения ключевых фраз, поэтому вы можете [установить и запустить контейнер API анализа текста](text-analytics-how-to-install-containers.md) в непосредственной близости к своим данным.
> * Эту функцию также можно использовать [асинхронно](text-analytics-how-to-call-api.md) с помощью `/analyze` конечной точки.

## <a name="preparation"></a>Подготовка

Извлечение ключевых фраз лучше всего работает с большими фрагментами текста, в отличие от анализа тональности, который эффективнее работает на маленьких блоках текста. Для получения наилучших результатов обеих операций советуем реструктуризировать входные данные соответствующим образом.

Необходимо иметь документы JSON в следующем формате: ID, Text, Language.

Документ должен содержать менее 5120 символов и иметь до 1000 элементов (идентификаторов) в коллекции. Коллекция передается в тексте запроса. Ниже приведен пример иллюстрации содержимого, которое можно отправлять для извлечения ключевых фраз. 

Дополнительные сведения об объектах запросов и ответов см. в разделе [вызов API анализа текста](text-analytics-how-to-call-api.md) .  

### <a name="example-synchronous-request-object"></a>Пример синхронного объекта запроса


```json
    {
        "documents": [
            {
                "language": "en",
                "id": "1",
                "text": "We love this trail and make the trip every year. The views are breathtaking and well worth the hike!"
            },
            {
                "language": "en",
                "id": "2",
                "text": "Poorly marked trails! I thought we were goners. Worst hike ever."
            },
            {
                "language": "en",
                "id": "3",
                "text": "Everyone in my family liked the trail but thought it was too challenging for the less athletic among us. Not necessarily recommended for small children."
            },
            {
                "language": "en",
                "id": "4",
                "text": "It was foggy so we missed the spectacular views, but the trail was ok. Worth checking out if you are in the area."
            },
            {
                "language": "en",
                "id": "5",
                "text": "This is my favorite trail. It has beautiful views and many places to stop and rest"
            }
        ]
    }
```

### <a name="example-asynchronous-request-object"></a>Пример объекта асинхронного запроса

Начиная `v3.1-preview.3` с, запросы NER можно отправить асинхронно с помощью `/analyze` конечной точки.


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
        "keyPhraseExtractionTasks": [{
            "parameters": {
                "model-version": "latest"
            }
        }],
    }
}
```

## <a name="step-1-structure-the-request"></a>Шаг 1. Структурирование запроса

Дополнительные сведения об определении запроса см. в статье [How to call the Text Analytics API](text-analytics-how-to-call-api.md) (Как вызвать REST API службы "Анализ текста"). Для удобства повторим следующие моменты.

+ Создайте запрос **POST** . Ознакомьтесь с документацией по API для этого запроса: [Ключевые фразы API](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/KeyPhrases).

+ Задайте конечную точку HTTP для извлечения ключевых фраз с помощью ресурса Анализа текста в Azure или экземпляра [контейнера Анализа текста](text-analytics-how-to-install-containers.md). Если вы используете API в синхронном режиме, необходимо включить `/text/analytics/v3.0/keyPhrases` в URL-адрес. Например: `https://<your-custom-subdomain>.api.cognitiveservices.azure.com/text/analytics/v3.0/keyPhrases`.

+ Задайте заголовок запроса, чтобы включить [ключ доступа](../../cognitive-services-apis-create-account.md#get-the-keys-for-your-resource) для операций Анализа текста.

+ В тексте запроса укажите набор документов JSON, которые подготовлены для этого анализа.

> [!Tip]
> Используйте [Postman](text-analytics-how-to-call-api.md) или откройте **консоль тестирования API** в [документации](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/KeyPhrases), чтобы структурировать запрос и передать его (используя метод POST) в службу.

## <a name="step-2-post-the-request"></a>Шаг 2. Передача запроса

Анализ выполняется при получении запроса. Сведения о размере и количестве запросов, которые можно отправить в минуту или в секунду, см. в разделе [ограничения данных](../overview.md#data-limits) в обзоре.

Не забывайте, что эта служба работает без отслеживания состояния. Данные не хранятся в учетной записи. Результаты возвращаются немедленно в ответе.

## <a name="step-3-view-results"></a>Шаг 3. Просмотр результатов

Все запросы POST вернут ответ в формате JSON с идентификаторами и обнаруженными свойствами. Порядок возвращаемых ключевых фраз определяется в модели внутренним образом.

Вывод возвращается немедленно. Можно выполнить потоковую передачу результатов в приложение, которое принимает JSON, или сохранить выходные данные в файле локальной системы, а затем импортировать их в приложение, которое дает возможность сортировки, поиска данных и управления ими.

Ниже показан пример выходных данных для извлечения ключевых фраз из конечной точки v 3.1-Preview. 2:

### <a name="synchronous-result"></a>Синхронный результат

```json
    {
       "documents":[
          {
             "id":"1",
             "keyPhrases":[
                "year",
                "trail",
                "trip",
                "views",
                "hike"
             ],
             "warnings":[]
          },
          {
             "id":"2",
             "keyPhrases":[
                "marked trails",
                "Worst hike",
                "goners"
             ],
             "warnings":[]
          },
          {
             "id":"3",
             "keyPhrases":[
                "trail",
                "small children",
                "family"
             ],
             "warnings":[]
          },
          {
             "id":"4",
             "keyPhrases":[
                "spectacular views",
                "trail",
                "Worth",
                "area"
             ],
             "warnings":[]
          },
          {
             "id":"5",
             "keyPhrases":[
                "places",
                "beautiful views",
                "favorite trail",
                "rest"
             ],
             "warnings":[]
          }
       ],
       "errors":[],
       "modelVersion":"2020-07-01"
    }

```
Как уже отмечалось, анализатор находит и отбрасывает слова, которые не представляют смысла и сохраняет одиночные термины или фразы, которые могут быть темой или объектом предложения.

### <a name="asynchronous-result"></a>Асинхронный результат

При использовании `/analyze` конечной точки для асинхронной операции вы получите ответ, содержащий задачи, отправленные в API.

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


## <a name="summary"></a>Итоги

В этой статье рассматриваются основные понятия и рабочий процесс извлечения ключевой фразы с помощью Анализа текста в Cognitive Services. В разделе "Сводка" сделайте следующее.

+ [API извлечения ключевых фраз](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/KeyPhrases) доступен для некоторых языков.
+ В тексте запроса передаются документы JSON, которые содержат идентификатор, текст и код языка.
+ Запрос POST к `/keyphrases` `/analyze` конечной точке или, используя персонализированный [ключ доступа и конечную точку](../../cognitive-services-apis-create-account.md#get-the-keys-for-your-resource) , допустимую для вашей подписки.
+ Выходные данные ответа, состоящие из ключевых слов и фраз для каждого идентификатора документа, могут передаваться в любое приложение, поддерживающее формат JSON, например Microsoft Office Excel и Power BI.

## <a name="see-also"></a>См. также раздел

 API "Анализ текста": [общие сведения](../overview.md) и [вопросы и ответы](../text-analytics-resource-faq.md)</br>
 [Анализ текста](//go.microsoft.com/fwlink/?LinkID=759712)

## <a name="next-steps"></a>Дальнейшие действия

* [Text Analytics overview](../overview.md) (Общие сведения об анализе текста)
* [Использование клиентской библиотеки Анализа текста](../quickstarts/client-libraries-rest-api.md)
* [Новые возможности](../whats-new.md)
