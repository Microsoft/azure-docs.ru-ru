---
title: включить файл
description: включить файл
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: include
ms.custom: include file
ms.date: 11/09/2020
ms.openlocfilehash: fa497b69b067d5556f11effdb52505895ecc3bdd
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "94386513"
---
В этом кратком руководстве описывается, как получить ответ из базы знаний с помощью Postman.

## <a name="prerequisites"></a>Предварительные требования

* Последняя версия [**Postman**](https://www.getpostman.com/).
* Вам понадобится:
    * [Служба QnA Maker](../How-To/set-up-qnamaker-service-azure.md).
    * Обученная и опубликованная [база знаний с вопросами и ответами](../Quickstarts/add-question-metadata-portal.md), созданная на основе краткого руководства, с настроенными метаданными и беседой.

> [!NOTE]
> Когда вы будете готовы создать ответ на вопрос из базы знаний, [обучите](../Quickstarts/create-publish-knowledge-base.md#save-and-train) и [опубликуйте](../Quickstarts/create-publish-knowledge-base.md#publish-the-knowledge-base) эту базу знаний. После публикации базы знаний на странице **публикации** отображаются параметры запроса HTTP для создания ответа. На вкладке **Postman** отображаются параметры, необходимые для создания ответа.

## <a name="set-up-postman-for-requests"></a>Настройка получения запросов в Postman

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/v1)

В этом кратком руководстве применяются те же параметры для запроса **POST** через Postman, а затем настраивается отправка JSON в теле запроса POST, где служба-получатель выбирается в зависимости от нужного результата запроса.

Используйте эту процедуру, чтобы настроить Postman, а из следующих разделов вы узнаете, как настроить JSON в теле запроса POST.

1. На странице **Параметры** для базы знаний щелкните вкладку **Postman**, чтобы увидеть конфигурацию, используемую для создания ответа из базы знаний. Скопируйте следующие данные для использования в Postman.

    |Имя|Параметр|Цель и значение|
    |--|--|--|
    |`POST`| `/knowledgebases/replace-with-your-knowledge-base-id/generateAnswer`|Это метод HTTP и маршрут для URL-адреса.|
    |`Host`|`https://YOUR-RESOURCE_NAME.azurewebsites.net/qnamaker`|Это узел URL-адреса. Объедините значения Host и Post, чтобы получить полный URL-адрес generateAnswer.|
    |`Authorization`|`EndpointKey xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`|Значение заголовка для авторизации запроса в Azure. |
    |`Content-type`|`application/json`|Значение заголовка для содержимого.|
    ||`{"question":"<Your question>"}`|Текст запроса POST в формате объекта JSON. Это значение будет изменяться в каждом из следующих разделов в зависимости от того, для чего предназначен очередной запрос.|

1. Откройте Postman и создайте простой запрос **POST** с параметрами опубликованной ранее базы знаний. В следующих разделах JSON в теле POST будет изменяться в соответствии с новыми запросами к базе знаний.

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/v2)

В этом кратком руководстве применяются те же параметры для запроса **POST** через Postman, а затем настраивается отправка JSON в теле запроса POST, где служба-получатель выбирается в зависимости от нужного результата запроса.

Используйте эту процедуру, чтобы настроить Postman, а из следующих разделов вы узнаете, как настроить JSON в теле запроса POST.

1. На странице **Параметры** для базы знаний щелкните вкладку **Postman**, чтобы увидеть конфигурацию, используемую для создания ответа из базы знаний. Скопируйте следующие данные для использования в Postman.

    |Имя|Параметр|Цель и значение|
    |--|--|--|
    |`POST`| `/knowledgebases/replace-with-your-knowledge-base-id/generateAnswer`|Это метод HTTP и маршрут для URL-адреса.|
    |`Host`|`https://YOUR-RESOURCE_NAME.cognitiveservices.azure.com/qnamaker`|Это узел URL-адреса. Объедините значения Host и Post, чтобы получить полный URL-адрес generateAnswer.|
    |`Ocp-Apim-Subscription-Key`|`xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`|Значение заголовка для авторизации запроса. |
    |`Content-type`|`application/json`|Значение заголовка для содержимого.|
    ||`{"question":"<Your question>"}`|Текст запроса POST в формате объекта JSON. Это значение будет изменяться в каждом из следующих разделов в зависимости от того, для чего предназначен очередной запрос.|

1. Откройте Postman и создайте простой запрос **POST** с параметрами опубликованной ранее базы знаний. В следующих разделах JSON в теле POST будет изменяться в соответствии с новыми запросами к базе знаний.
---

## <a name="use-metadata-to-filter-answer"></a>Применение метаданных для фильтрации ответов

При работе с предыдущим кратким руководством мы добавили метаданные в две пары "вопрос — ответ", чтобы различать два разных вопроса. Добавьте метаданные в запрос, чтобы отфильтровать только одну нужную нам пару "вопрос — ответ".

1. В Postman измените только объект JSON в запросе, добавив свойство `strictFilters` с парой "имя — значение" следующего вида: `service:qna_maker`. Теперь текст JSON будет выглядеть так:

    ```json
    {
        'question':'size',
        'strictFilters': [
            {
                'name':'service','value':'qna_maker'
            }
        ]
    }
    ```

    Этот вопрос состоит всего из одного слова (`size`), по которому может возвращаться любая пара из вопроса и ответа. Массив `strictFilters` указывает, что ответ нужно сократить до `qna_maker` ответов.

1. Ответ включает только тот ответ, который соответствует критериям фильтра.

    Следующий ответ был отформатирован для удобства чтения:

    ```JSON
    {
        "answers": [
            {
                "questions": [
                    "How large a knowledge base can I create?",
                    "What is the max size of a knowledge base?",
                    "How many GB of data can a knowledge base hold?"
                ],
                "answer": "The size of the knowledge base depends on the SKU of Azure search you choose when creating the QnA Maker service. Read [here](https://docs.microsoft.com/azure/cognitive-services/qnamaker/tutorials/choosing-capacity-qnamaker-deployment) for more details.",
                "score": 68.76,
                "id": 3,
                "source": "https://docs.microsoft.com/azure/cognitive-services/qnamaker/troubleshooting",
                "metadata": [
                    {
                        "name": "link_in_answer",
                        "value": "true"
                    },
                    {
                        "name": "service",
                        "value": "qna_maker"
                    }
                ],
                "context": {
                    "isContextOnly": false,
                    "prompts": []
                }
            }
        ],
        "debugInfo": null
    }
    ```

    Если существует пара вопроса и ответа, которая не соответствует поисковому запросу, но соответствует фильтру, она не будет возвращена. Вместо этого возвращается общий ответ `No good match found in KB.`.

## <a name="use-debug-query-property"></a>Использование свойства debug в запросе

> [!NOTE]
>Мы не рекомендуем использовать свойство debug для всех зависимостей. Это свойство было добавлено только для того, чтобы помочь разработчикам в устранении неполадок.

Отладочная информация поможет понять, как был определен возвращаемый ответ. Это иногда полезно, но необязательно. Чтобы создать ответ с отладочной информацией, добавьте свойство `debug`:

1. В Postman измените только объект JSON, добавив свойство `debug`. Теперь JSON будет выглядеть так:

    ```json
    {
        'question':'size',
        'Debug': {
            'Enable':true
        }

    }
    ```

1. Ответ включает необходимые сведения об ответе. В следующих выходных данных JSON некоторые отладочные сведения заменены многоточиями.

    ```console
    {
        "answers": [
            {
                "questions": [
                    "How do I share a knowledge base with others?"
                ],
                "answer": "Sharing works at the level of a QnA Maker service, that is, all knowledge bases in the service will be shared. Read [here](https://docs.microsoft.com/azure/cognitive-services/qnamaker/how-to/collaborate-knowledge-base) how to collaborate on a knowledge base.",
                "score": 56.07,
                "id": 5,
                "source": "https://docs.microsoft.com/azure/cognitive-services/qnamaker/troubleshooting",
                "metadata": [],
                "context": {
                    "isContextOnly": false,
                    "prompts": []
                }
            }
        ],
        "debugInfo": {
            "userQuery": {
                "question": "How do I programmatically update my Knowledge Base?",
                "top": 1,
                "userId": null,
                "strictFilters": [],
                "isTest": false,
                "debug": {
                    "enable": true,
                    "recordL1SearchLatency": false,
                    "mockQnaL1Content": null
                },
                "rankerType": 0,
                "context": null,
                "qnaId": 0,
                "scoreThreshold": 0.0
            },
            "rankerInfo": {
                "specialFuzzyQuery": "how do i programmatically~6 update my knowledge base",
                "synonyms": "what s...",
                "rankerLanguage": "English",
                "rankerFileName": "https://qnamakerstore.blob.core.windows.net/qnamakerdata/rankers/ranker-English.ini",
                "rankersDirectory": "D:\\home\\site\\wwwroot\\Data\\QnAMaker\\rd0003ffa60fc45.24.0\\RankerData\\Rankers",
                "allQnAsfeatureValues": {
                    "WordnetSimilarity": {
                        "5": 0.54706300120043716,...
                    },
                    ...
                },
                "rankerVersion": "V2",
                "rankerModelType": "TreeEnsemble",
                "rankerType": 0,
                "indexResultsCount": 25,
                "reRankerResultsCount": 1
            },
            "runtimeVersion": "5.24.0",
            "indexDebugInfo": {
                "indexDefinition": {
                    "name": "064a4112-bd65-42e8-b01d-141c4c9cd09e",
                    "fields": [...
                    ],
                    "scoringProfiles": [],
                    "defaultScoringProfile": null,
                    "corsOptions": null,
                    "suggesters": [],
                    "analyzers": [],
                    "tokenizers": [],
                    "tokenFilters": [],
                    "charFilters": [],
                    "@odata.etag": "\"0x8D7A920EA5EE6FE\""
                },
                "qnaCount": 117,
                "parameters": {},
                "azureSearchResult": {
                    "continuationToken": null,
                    "@odata.count": null,
                    "@search.coverage": null,
                    "@search.facets": null,
                    "@search.nextPageParameters": null,
                    "value": [...],
                    "@odata.nextLink": null
                }
            },
            "l1SearchLatencyInMs": 0,
            "qnaL1Results": {...}
        },
        "activeLearningEnabled": true
    }
    ```

## <a name="use-test-knowledge-base"></a>Использование тестовой базы знаний

Если вы хотите получить ответ от тестовой базы знаний, используйте в тексте свойство `isTest`.

В Postman измените только объект JSON в запросе, добавив свойство `isTest`. Теперь JSON будет выглядеть так:

```json
{
    'question':'size',
    'isTest': true
}
```

В ответе в формате JSON используется та же схема, что и в запросе к опубликованной базе знаний.

> [!NOTE]
> Если тестовая и опубликованная базы знаний полностью совпадают, незначительные вариации могут возникать из-за того, что тестовый индекс является общим для всех баз знаний в ресурсе.

## <a name="query-for-a-chit-chat-answer"></a>Запрос ответа в беседе

1. В Postman измените только объект JSON, включив инструкцию завершения диалога от пользователя. Теперь JSON будет выглядеть так:

    ```json
    {
        'question':'thank you'
    }
    ```

1. Результат содержит оценку и ответ.

    ```json
    {
      "answers": [
          {
              "questions": [
                  "I thank you",
                  "Oh, thank you",
                  "My sincere thanks",
                  "My humblest thanks to you",
                  "Marvelous, thanks",
                  "Marvelous, thank you kindly",
                  "Marvelous, thank you",
                  "Many thanks to you",
                  "Many thanks",
                  "Kthx",
                  "I'm grateful, thanks",
                  "Ahh, thanks",
                  "I'm grateful for that, thank you",
                  "Perfecto, thanks",
                  "I appreciate you",
                  "I appreciate that",
                  "I appreciate it",
                  "I am very thankful for that",
                  "How kind, thank you",
                  "Great, thanks",
                  "Great, thank you",
                  "Gracias",
                  "Gotcha, thanks",
                  "Gotcha, thank you",
                  "Awesome thanks!",
                  "I'm grateful for that, thank you kindly",
                  "thank you pal",
                  "Wonderful, thank you!",
                  "Wonderful, thank you very much",
                  "Why thank you",
                  "Thx",
                  "Thnx",
                  "That's very kind",
                  "That's great, thanks",
                  "That is lovely, thanks",
                  "That is awesome, thanks!",
                  "Thanks bot",
                  "Thanks a lot",
                  "Okay, thanks!",
                  "Thank you so much",
                  "Perfect, thanks",
                  "Thank you my friend",
                  "Thank you kindly",
                  "Thank you for that",
                  "Thank you bot",
                  "Thank you",
                  "Right on, thanks very much",
                  "Right on, thanks a lot",
                  "Radical, thanks",
                  "Rad, thanks",
                  "Rad thank you",
                  "Wonderful, thanks!",
                  "Thanks"
              ],
              "answer": "You're welcome.",
              "score": 100.0,
              "id": 75,
              "source": "qna_chitchat_Professional.tsv",
              "metadata": [
                  {
                      "name": "editorial",
                      "value": "chitchat"
                  }
              ],
              "context": {
                  "isContextOnly": false,
                  "prompts": []
              }
          }
      ],
      "debugInfo": null,
      "activeLearningEnabled": true
    }
    ```

    Так как вопрос `Thank you` точно соответствует вопросу в беседе, достоверность QnA Maker составляет 100 %. QnA Maker также возвращает все связанные вопросы и свойство метаданных, содержащее сведения о теге метаданных беседы.

## <a name="use-threshold-and-default-answer"></a>Использование порога и ответа по умолчанию

Вы можете запросить минимальный порог для возвращаемого ответа. Если указанное пороговое значение не достигается, возвращается ответ по умолчанию.

1. В Postman измените только объект JSON, включив инструкцию завершения диалога от пользователя. Теперь JSON будет выглядеть так:

    ```json
    {
        'question':'size',
        'scoreThreshold':80.00
    }
    ```

    База знаний не должна найти этот ответ, так как оценка вопроса составляет всего 71 %, поэтому вместо него возвращается ответ по умолчанию, указанный при создании базы знаний.

    Ответ JSON, содержащий оценку и ответ, будет выглядеть так:

    ```json
    {
        "answers": [
            {
                "questions": [],
                "answer": "No good match found in KB.",
                "score": 0.0,
                "id": -1,
                "source": null,
                "metadata": []
            }
        ],
        "debugInfo": null,
        "activeLearningEnabled": true
    }
    ```

    QnA Maker возвращает оценку `0`. Это означает, что достоверного ответа нет, но также возвращается ответ по умолчанию.

1. Измените пороговое значение на 60 % и повторите запрос.

    ```json
    {
        'question':'size',
        'scoreThreshold':60.00
    }
    ```

    Теперь JSON возвращает найденный ответ.

    ```json
    {
        "answers": [
            {
                "questions": [
                    "How large a knowledge base can I create?",
                    "What is the max size of a knowledge base?",
                    "How many GB of data can a knowledge base hold?"
                ],
                "answer": "The size of the knowledge base depends on the SKU of Azure search you choose when creating the QnA Maker service. Read [here](https://docs.microsoft.com/azure/cognitive-services/qnamaker/tutorials/choosing-capacity-qnamaker-deployment) for more details.",
                "score": 71.1,
                "id": 3,
                "source": "https://docs.microsoft.com/azure/cognitive-services/qnamaker/troubleshooting",
                "metadata": [
                    {
                        "name": "link_in_answer",
                        "value": "true"
                    },
                    {
                        "name": "server",
                        "value": "qna_maker"
                    }
                ],
                "context": {
                    "isContextOnly": false,
                    "prompts": []
                }
            }
        ],
        "debugInfo": null,
        "activeLearningEnabled": true
    }
    ```