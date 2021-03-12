---
title: Контекстная фильтрация с помощью метаданных
titleSuffix: Azure Cognitive Services
description: QnA Maker фильтрует QnA пары по метаданным.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 11/09/2020
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 8f65ca9386963824f0cb740f587de83c9dec7f78
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103022143"
---
# <a name="filter-responses-with-metadata"></a>Фильтрация ответов с помощью метаданных

QnA Maker позволяет добавлять метаданные в виде пар "ключ — значение" в свои пары вопросов и ответов. Затем эти сведения можно использовать для фильтрации результатов в пользовательских запросах, а также для хранения дополнительных сведений, которые можно использовать в дальнейших беседах.

<a name="qna-entity"></a>

## <a name="store-questions-and-answers-with-a-qna-entity"></a>Хранение вопросов и ответов с помощью сущности QnA

Важно понимать, как QnA Maker хранит данные вопроса и ответов. На рисунке ниже показана сущность QnA.

![Иллюстрация сущности QnA](../media/qnamaker-how-to-metadata-usage/qna-entity.png)

У каждой сущности QnA имеется уникальный постоянный идентификатор. Идентификатор можно использовать для внесения обновлений в определенную сущность QnA.

## <a name="use-metadata-to-filter-answers-by-custom-metadata-tags"></a>Использование метаданных для фильтрации ответов по тегам пользовательских метаданных

Добавление метаданных позволяет фильтровать ответы по этим тегам метаданных. Добавьте столбец метаданных из меню **Параметры представления** . Добавьте метаданные в базу знаний, щелкнув значок метаданных, **+** чтобы добавить пару метаданных. Эта пара состоит из одного ключа и одного значения.

![Снимок экрана: Добавление метаданных](../media/qnamaker-how-to-metadata-usage/add-metadata.png)

<a name="filter-results-with-strictfilters-for-metadata-tags"></a>

## <a name="filter-results-with-strictfilters-for-metadata-tags"></a>Фильтрация результатов с помощью strictFilters по тегам метаданных

Примите во внимание вопрос пользователя "когда закроется это отель?", где предполагается, что цель предполагается для ресторана "сообразительного".

Поскольку результаты требуются только для ресторана "сообразительного", можно установить фильтр в вызове Женератеансвер для метаданных "имя ресторана". Это показано в следующем примере:

```json
{
    "question": "When does this hotel close?",
    "top": 1,
    "strictFilters": [ { "name": "restaurant", "value": "paradise"}]
}
```

### <a name="logical-and-by-default"></a>Логическое и по умолчанию

Чтобы объединить несколько фильтров метаданных в запросе, добавьте фильтры дополнительных метаданных в массив `strictFilters` Свойства. По умолчанию значения логически объединены (и). Логическое сочетание требует, чтобы все фильтры совпадали с парами QnA, чтобы пара возвращалась в ответе.

Это эквивалентно использованию `strictFiltersCompoundOperationType` свойства со значением `AND` .

### <a name="logical-or-using-strictfilterscompoundoperationtype-property"></a>Логическое или с использованием свойства Стриктфилтерскомпаундоператионтипе

При объединении нескольких фильтров метаданных, если важен только один или несколько соответствующих фильтров, используйте `strictFiltersCompoundOperationType` свойство со значением `OR` .

Это позволяет базе знаний возвращать ответы при совпадении фильтра, но не возвращает ответы, не имеющие метаданных.

```json
{
    "question": "When do facilities in this hotel close?",
    "top": 1,
    "strictFilters": [
      { "name": "type","value": "restaurant"},
      { "name": "type", "value": "bar"},
      { "name": "type", "value": "poolbar"}
    ],
    "strictFiltersCompoundOperationType": "OR"
}
```

### <a name="metadata-examples-in-quickstarts"></a>Примеры метаданных в кратком руководстве

Дополнительные сведения о метаданных см. в кратком руководстве по QnA Maker портале для метаданных.
* [Разработка: добавление метаданных в пару "вопрос-ответ"](../quickstarts/add-question-metadata-portal.md#add-metadata-to-filter-the-answers)
* [Прогнозирование запросов: фильтрация ответов по метаданным](../quickstarts/get-answer-from-knowledge-base-using-url-tool.md)

<a name="keep-context"></a>

## <a name="use-question-and-answer-results-to-keep-conversation-context"></a>Использование результатов вопросов и ответов для сохранения контекста беседы

Ответ на Женератеансвер содержит сведения о метаданных соответствующей пары вопросов и ответов. Эти сведения можно использовать в клиентском приложении для хранения контекста предыдущего диалога, который будет использоваться в последующих диалогах.

```json
{
    "answers": [
        {
            "questions": [
                "What is the closing time?"
            ],
            "answer": "10.30 PM",
            "score": 100,
            "id": 1,
            "source": "Editorial",
            "metadata": [
                {
                    "name": "restaurant",
                    "value": "paradise"
                },
                {
                    "name": "location",
                    "value": "secunderabad"
                }
            ]
        }
    ]
}
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Анализ базы знаний](../How-to/get-analytics-knowledge-base.md)
