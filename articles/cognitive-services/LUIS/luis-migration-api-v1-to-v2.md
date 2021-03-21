---
title: Переход с API версии 1 на API версии 2
titleSuffix: Azure Cognitive Services
description: Конечная точка версии 1 и API-интерфейсы Распознавание речи разработки являются устаревшими. Используйте это руководство, чтобы перейти на API конечных точек и разработки версии 2.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 04/02/2019
ms.openlocfilehash: 867ae2cc7567077786bb0840cd11c47b786be423
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95018758"
---
# <a name="api-v1-to-v2-migration-guide-for-luis-apps"></a>Руководство по переходу с API версии 1 на API версии 2 для приложений на основе LUIS
[Конечная точка](https://aka.ms/v1-endpoint-api-docs) версии 1 и API-интерфейсы [разработки](https://aka.ms/v1-authoring-api-docs) являются устаревшими. В этом руководство вы узнаете, как выполнить миграцию на [конечную точку](https://go.microsoft.com/fwlink/?linkid=2092356) версии 2 и на API-интерфейсы [разработки](https://go.microsoft.com/fwlink/?linkid=2092087) .

## <a name="new-azure-regions"></a>Новые регионы Azure
В LUIS были добавлены новые [регионы](./luis-reference-regions.md) для API LUIS. LUIS предоставляет другой портал для групп регионов. Приложение должно быть разработано в том же регионе, в который вы собираетесь отправлять запросы. Приложения не переносятся между регионами автоматически. Чтобы приложение стало доступно в новом регионе, необходимо экспортировать приложение из одного региона и импортировать его в другом регионе.

## <a name="authoring-route-changes"></a>Изменения в маршруте разработки
Маршрут API разработки был изменен с **prog** на **api**.


| version | route |
|--|--|
|1|/luis/v1.0/**prog**/apps|
|2|/luis/**api**/v2.0/apps|


## <a name="endpoint-route-changes"></a>Изменения в маршруте конечных точек
API конечной точки содержит новые параметры строки запроса, а также другой ответ. Если флаг verbose установлен, то наряду с максимальной оценкой topScoringIntent возвращаются все намерения независимо от оценок (в массиве intents).

| version | Маршрут GET |
|--|--|
|1|/luis/v1/application?ID={идентификатор_приложения}&q={запрос}|
|2|/luis/v2.0/apps/{идентификатор_приложения}?q={запрос}[&timezoneOffset][&verbose][&spellCheck][&staging][&bing-spell-check-subscription-key][&log]|


Успешный ответ конечной точки версии 1:
```json
{
  "odata.metadata":"https://dialogice.cloudapp.net/odata/$metadata#domain","value":[
    {
      "id":"bccb84ee-4bd6-4460-a340-0595b12db294","q":"turn on the camera","response":"[{\"intent\":\"OpenCamera\",\"score\":0.976928055},{\"intent\":\"None\",\"score\":0.0230718572}]"
    }
  ]
}
```

Успешный ответ конечной точки версии 2:
```json
{
  "query": "forward to frank 30 dollars through HSBC",
  "topScoringIntent": {
    "intent": "give",
    "score": 0.3964121
  },
  "entities": [
    {
      "entity": "30",
      "type": "builtin.number",
      "startIndex": 17,
      "endIndex": 18,
      "resolution": {
        "value": "30"
      }
    },
    {
      "entity": "frank",
      "type": "frank",
      "startIndex": 11,
      "endIndex": 15,
      "score": 0.935219169
    },
    {
      "entity": "30 dollars",
      "type": "builtin.currency",
      "startIndex": 17,
      "endIndex": 26,
      "resolution": {
        "unit": "Dollar",
        "value": "30"
      }
    },
    {
      "entity": "hsbc",
      "type": "Bank",
      "startIndex": 36,
      "endIndex": 39,
      "resolution": {
        "values": [
          "BankeName"
        ]
      }
    }
  ]
}
```

## <a name="key-management-no-longer-in-api"></a>Управление ключами больше не осуществляется не в API
API ключей конечных точек подписок устарели, возвращая 410 GONE.

| version | route |
|--|--|
|1|/luis/v1.0/prog/subscriptions|
|1|/luis/v1.0/prog/subscriptions/{ключ_подписки}|

[Ключи конечных точек](luis-how-to-azure-subscription.md) Azure создаются на портале Azure. Вы назначаете ключ приложению LUIS на странице **[Публикация](luis-how-to-azure-subscription.md)**. Фактическое значение ключа знать не требуется. Для назначения LUIS использует имя подписки.

## <a name="new-versioning-route"></a>Новый маршрут для управления версиями
Модель версии 2 теперь содержится в параметре [version](luis-how-to-manage-versions.md). Длина имени версии в маршруте — 10 символов. Версия по умолчанию — "0.1".

| version | route |
|--|--|
|1|/luis/v1.0/**prog**/apps/{идентификатор_приложения}/entities|
|2|/luis/**api**/v2.0/apps/{идентификатор_приложения}/**versions**/{идентификатор_версии}/entities|

## <a name="metadata-renamed"></a>Переименование метаданных
Несколько API, возвращающих метаданные LUIS, были переименованы.

| Имя маршрута версии 1 | Имя маршрута версии 2 |
|--|--|
|PersonalAssistantApps |assistants|
|applicationcultures|cultures|
|applicationdomains|domains|
|applicationusagescenarios|usagescenarios|


## <a name="sample-renamed-to-suggest"></a>"Пример" переименован в "предложение"
LUIS предлагает высказывания из существующих [высказываний конечных точек](luis-how-to-review-endpoint-utterances.md), которые могут улучшить модель. В предыдущей версии предложения назывались **примерами**. В новой версии "примеры" называются **предложениями**. На веб-сайте LUIS это называется **[Просмотрами высказываний конечных точек](luis-how-to-review-endpoint-utterances.md)**.

| version | route |
|--|--|
|1|/luis/v1.0/**prog**/apps/{идентификатор_приложения}/entities/{идентификатор_сущности}/**sample**|
|1|/luis/v1.0/**prog**/apps/{идентификатор_приложения}/intents/{идентификатор_намерения}/**sample**|
|2|/luis/**api**/v2.0/apps/{идентификатор_приложения}/**versions**/{идентификатор_версии}/entities/{идентификатор_сущности}/**suggest**|
|2|/luis/**api**/v2.0/apps/{идентификатор_приложения}/**versions**/{идентификатор_версии}/entities/intents/{идентификатор_намерения}/**suggest**|


## <a name="create-app-from-prebuilt-domains"></a>Создание приложения на основе предварительно созданных доменов
[Предварительно созданные домены](./howto-add-prebuilt-models.md) предоставляют предварительно определенную модель предметной области. Предварительно созданные домены позволяют быстро разработать приложение LUIS для распространенных доменов. Этот API позволяет создать новое приложение на основе предварительно созданного домена. В качестве ответа возвращается идентификатор нового приложения.

|Маршрут версии 2|Команда|
|--|--|
|/luis/api/v2.0/apps/customprebuiltdomains  |get, post|
|/luis/api/v2.0/apps/customprebuiltdomains/{язык_и_региональные_параметры}  |get|

## <a name="importing-1x-app-into-2x"></a>Импорт приложений версии 1.x в версию 2.x
В экспортированном коде JSON приложения версии 1.x есть некоторые фрагменты, которые необходимо изменить перед импортом в [LUIS][LUIS] 2.0.

### <a name="prebuilt-entities"></a>Предварительно созданные сущности
[Предварительно созданные сущности](./howto-add-prebuilt-models.md) были изменены. Убедитесь, что вы используете предварительно созданные сущности версии 2. К ним относится [datetimeV2](luis-reference-prebuilt-datetimev2.md) вместо datetime.

### <a name="actions"></a>Действия
Свойство actions больше не используется. Оно должно быть пустым.

### <a name="labeled-utterances"></a>Помеченные высказывания
В высказываниях версии 1 можно было включать пробелы в начале или в конце слова или фразы. Пробелы удалены.

## <a name="common-reasons-for-http-response-status-codes"></a>Наиболее распространенные причины для кодов ответа HTTP
См. раздел [Коды ответов API LUIS](luis-reference-response-codes.md).

## <a name="next-steps"></a>Дальнейшие действия

Используйте документацию по API версии 2, чтобы обновить существующие вызовы REST для API [конечных точек](https://go.microsoft.com/fwlink/?linkid=2092356) и [разработки](https://go.microsoft.com/fwlink/?linkid=2092087) LUIS.

[LUIS]: ./luis-reference-regions.md