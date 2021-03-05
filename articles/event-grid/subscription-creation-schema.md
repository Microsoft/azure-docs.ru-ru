---
title: Схема подписки для службы "Сетка событий Azure"
description: В этой статье описываются свойства подписки на событие с помощью службы "Сетка событий Azure". Схема подписки на сетку событий.
ms.topic: reference
ms.date: 07/07/2020
ms.openlocfilehash: f6e33171cbba65cfeaca49ab6a8954be8bb89acb
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102199957"
---
# <a name="event-grid-subscription-schema"></a>Схема подписки для службы "Сетка событий"

Чтобы создать подписку для службы "Сетка событий", отправьте запрос на выполнение операции по созданию подписки на события. Используйте следующий формат:

```HTTP
PUT /subscriptions/{subscription-id}/resourceGroups/{group-name}/providers/{resource-provider}/{resource-type}/{resource-name}/Microsoft.EventGrid/eventSubscriptions/{event-type-definitions}?api-version=2018-01-01
``` 

Например, чтобы создать подписку на события для учетной записи хранения с именем `examplestorage` в группе ресурсов `examplegroup`, используйте следующий формат:

```HTTP
PUT /subscriptions/{subscription-id}/resourceGroups/examplegroup/providers/Microsoft.Storage/storageaccounts/examplestorage/Microsoft.EventGrid/eventSubscriptions/{event-type-definitions}?api-version=2018-01-01
``` 

Имя подписки на события должны состоять из 3–64 символов и содержать только буквы ( a–z, A–Z), цифры (0–9) и дефис ("-"). В этой статье приведены свойства и схема для основного текста запроса.
 
## <a name="event-subscription-properties"></a>Свойства подписки на события

| Свойство. | Тип | Описание |
| -------- | ---- | ----------- |
| ресурс destination | объект | Объект, который определяет конечную точку. |
| фильтр | объект | Необязательное поле для фильтрации событий по типам. |

### <a name="destination-object"></a>Объект destination

| Свойство. | Тип | Описание |
| -------- | ---- | ----------- |
| endpointType | строка | Тип конечной точки для подписки (веб-перехватчик или HTTP, концентратор событий либо очередь). | 
| endpointUrl | строка | URL-адрес назначения для событий в подписке на событие. | 

### <a name="filter-object"></a>Объект filter

| Свойство. | Тип | Описание |
| -------- | ---- | ----------- |
| includedEventTypes | array | Выполняет сопоставление, если тип события, указанный в сообщении о событии, полностью соответствует одному из этих типов. Вызывает ошибку, если имя события не соответствует зарегистрированному имени типа для источника события. По умолчанию соответствует всем типам событий. |
| subjectBeginsWith | строка | Фильтр соответствия префиксу для поля темы в сообщении о событии. Строка по умолчанию или пустая строка соответствует всем типам. | 
| subjectEndsWith | строка | Фильтр соответствия суффиксу для поля темы в сообщении о событии. Строка по умолчанию или пустая строка соответствует всем типам. |
| isSubjectCaseSensitive | строка | Управляет сопоставлением с учетом регистра в фильтрах. |
| енаблеадванцедфилтерингонаррайс | Логическое | Включает использование массивов для ключей в расширенной фильтрации. Дополнительные сведения см. в разделе [Расширенная фильтрация](event-filtering.md#advanced-filtering). |


## <a name="example-subscription-schema"></a>Пример схемы подписки

```json
{
  "properties": {
    "destination": {
      "endpointType": "webhook",
      "properties": {
          "endpointUrl": "https://example.azurewebsites.net/api/HttpTriggerCSharp1?code=VXbGWce53l48Mt8wuotr0GPmyJ/nDT4hgdFj9DpBiRt38qqnnm5OFg=="
      }
    },
    "filter": {
      "includedEventTypes": [ "Microsoft.Storage.BlobCreated", "Microsoft.Storage.BlobDeleted" ],
      "subjectBeginsWith": "/blobServices/default/containers/mycontainer/log",
      "subjectEndsWith": ".jpg",
      "isSubjectCaseSensitive ": "true"
    }
  }
}
```

## <a name="next-steps"></a>Дальнейшие действия

* См. дополнительные сведения о [службе "Сетка событий Azure"](overview.md).
