---
title: Исправление слов с ошибками — LUIS
titleSuffix: Azure Cognitive Services
description: Включите исправление слов с ошибками во фрагментах речи, добавив API проверки орфографии Bing версии 7 в запросы конечной точки LUIS.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 01/12/2021
ms.openlocfilehash: 509d1dc0b94bdfa9be5185df0bad793f7702eb26
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101731040"
---
# <a name="correct-misspelled-words-with-bing-resource"></a>Исправление слов с ошибками в ресурсе Bing

API прогнозирования версии 3 теперь поддерживает [API Bing для проверки орфографии](/bing/search-apis/bing-spell-check/overview). Добавьте проверку орфографии в приложение, включив ключ к ресурсу поиска Bing в заголовке запросов. Вы можете использовать существующий ресурс Bing, если вы уже являетесь его владельцем, или [создать новый](https://portal.azure.com/#create/Microsoft.BingSearch) , чтобы использовать эту функцию. 

Пример результата прогнозирования для запроса с ошибками:

```json
{
  "query": "bouk me a fliht to kayro",
  "prediction": {
    "alteredQuery": "book me a flight to cairo",
    "topIntent": "book a flight",
    "intents": {
      "book a flight": {
        "score": 0.9480589
      }
      "None": {
        "score": 0.0332136229
      }
    },
    "entities": {}
  }
}
```

Исправления для проверки орфографии выполняются до прогнозирования utterance пользователя LUIS. В ответе можно увидеть любые изменения в исходном utterance, включая проверку орфографии.

## <a name="create-bing-search-resource"></a>Создание Поиск Bing ресурса

Чтобы создать Поиск Bing ресурс в портал Azure, выполните следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com).

2. В верхнем левом углу щелкните **Создать ресурс**.

3. В поле поиска введите `Bing Search V7` и выберите службу.

4. Справа появится информационная панель, содержащая сведения, включая юридическое уведомление. Выберите **Создать**, чтобы начать процесс создания подписки.

> [!div class="mx-imgBorder"]
> ![Ресурс API Bing для проверки орфографии версии 7](./media/luis-tutorial-bing-spellcheck/bing-search-resource-portal.png)

5. На следующей панели введите параметры службы. Дождитесь завершения процесса создания службы.

6. После создания ресурса перейдите в колонку **ключи и конечная точка** слева. 

7. Скопируйте один из ключей, которые будут добавлены в заголовок прогнозирующего запроса. Вам потребуется только один из двух ключей.

<!--
## Using the key in LUIS test panel
There are two places in LUIS to use the key. The first is in the [test panel](luis-interactive-test.md#view-bing-spell-check-corrections-in-test-panel). The key isn't saved into LUIS but instead is a session variable. You need to set the key every time you want the test panel to apply the Bing Spell Check API v7 service to the utterance. See [instructions](luis-interactive-test.md#view-bing-spell-check-corrections-in-test-panel) in the test panel for setting the key.
-->
## <a name="enable-spell-check-from-ui"></a>Включить проверку орфографии в пользовательском интерфейсе 
Вы можете включить функцию проверкой орфографии для примера запроса с помощью [портала Luis](https://www.luis.ai). Выберите **Управление** в верхней части экрана, а **ресурсы Azure** — в левой области навигации. После привязки к приложению прогнозирующего ресурса можно выбрать **параметр изменить параметры запроса** в нижней части страницы и вставить ключ ресурса в поле **включить проверку орфографии** .
    
   > [!div class="mx-imgBorder"]
   > ![Включить проверку орфографии](./media/luis-tutorial-bing-spellcheck/spellcheck-query-params.png)


## <a name="adding-the-key-to-the-endpoint-url"></a>Добавление ключа в URL-адрес конечной точки
Для каждого запроса, для которого необходимо применить исправление орфографии, запросу конечной точки требуется ключ ресурса Bing проверки орфографии, переданный в параметре заголовка запроса. Можно настроить чат-бот, вызывающий LUIS, или вызывать API конечной точки LUIS напрямую. Независимо от того, как вызывается конечная точка, каждый и каждый вызов должен включать в запрос заголовка необходимые сведения для правильной работы исправлений правописания. Значение ключа необходимо задать с помощью **МКТ-Bing-правописания-Check-Key** .

|Ключ заголовка|Значение заголовка|
|--|--|
|`mkt-bing-spell-check-key`|Ключи, найденные в колонке " **ключи и конечная точка** " вашего ресурса|

## <a name="send-misspelled-utterance-to-luis"></a>Отправка фрагментов речи с орфографическими ошибками в LUIS
1. Добавьте в прогнозирующий запрос с ошибкой utterance, который будет отправляться, например "насколько далеко маунтаинн?". В английском языке правильное написание `mountain` — с одной буквой `n`.

2. Служба LUIS передаст ответ с результатом JSON для `How far is the mountain?`. Если API проверки орфографии Bing версии 7 обнаружит орфографическую ошибку, в поле `query` ответа JSON приложения LUIS будет указан исходный запрос, а в поле `alteredQuery` — исправленный запрос, отправленный в LUIS.

```json
{
  "query": "How far is the mountainn?",
  "alteredQuery": "How far is the mountain?",
  "topScoringIntent": {
    "intent": "Concierge",
    "score": 0.183866
  },
  "entities": []
}
```

## <a name="ignore-spelling-mistakes"></a>Пропуск орфографических ошибок

Если вы не хотите использовать службу Поиск Bing API версии 7, необходимо добавить правильную и неправильную орфографию.

Существуют два решения.

* Пример метки фразы продолжительностью, имеющий все различные слова, чтобы LUIS мог изучить правильную орфографию и опечатки. Этот способ требует больше усилий, чем применение проверки орфографии.
* Создайте список фраз со всеми вариациями слова. В этом решении не нужно помечать варианты слов в примере фразы продолжительностью.


> [!div class="nextstepaction"]
> [Подробнее о примерах фрагментов речи](./luis-how-to-add-entities.md)