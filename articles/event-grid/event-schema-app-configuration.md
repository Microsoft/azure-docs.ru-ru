---
title: Настройка приложения Azure в качестве источника службы "Сетка событий"
description: В этой статье описывается, как использовать конфигурацию приложения Azure в качестве источника событий службы "Сетка событий". Она содержит схему и ссылки на учебники и практические руководства.
ms.topic: conceptual
ms.date: 02/11/2021
ms.openlocfilehash: a64c6fead5e6d95ba11bc98d7e9a52e3021c3be2
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100366778"
---
# <a name="azure-app-configuration-as-an-event-grid-source"></a>Настройка приложения Azure в качестве источника службы "Сетка событий"
В этой статье содержатся свойства и схема событий конфигурации приложений Azure. Общие сведения о схемах событий см. в статье [Схема событий службы "Сетка событий Azure"](event-schema.md). Здесь также приводится список быстрых запусков и учебников по использованию конфигурации приложений Azure в качестве источника событий.

## <a name="available-event-types"></a>Доступные типы событий

В конфигурации приложений Azure выдаются следующие типы событий:

| Тип события | Описание |
| ---------- | ----------- |
| Microsoft. Аппконфигуратион. Кэйвалуемодифиед | Возникает при создании или замене ключевого значения. |
| Microsoft. Аппконфигуратион. Кэйвалуеделетед | Вызывается при удалении значения ключа. |

## <a name="example-event"></a>Пример события

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)
В следующем примере показана схема события изменения ключа "ключ — значение": 

```json
[{
  "id": "84e17ea4-66db-4b54-8050-df8f7763f87b",
  "topic": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/microsoft.appconfiguration/configurationstores/contoso",
  "subject": "https://contoso.azconfig.io/kv/Foo?label=FizzBuzz",
  "data": {
    "key": "Foo",
    "label": "FizzBuzz",
    "etag": "FnUExLaj2moIi4tJX9AXn9sakm0"
  },
  "eventType": "Microsoft.AppConfiguration.KeyValueModified",
  "eventTime": "2019-05-31T20:05:03Z",
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

Схема для события Deleted-valued имеет следующий вид: 

```json
[{
  "id": "84e17ea4-66db-4b54-8050-df8f7763f87b",
  "topic": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/microsoft.appconfiguration/configurationstores/contoso",
  "subject": "https://contoso.azconfig.io/kv/Foo?label=FizzBuzz",
  "data": {
    "key": "Foo",
    "label": "FizzBuzz",
    "etag": "FnUExLaj2moIi4tJX9AXn9sakm0"
  },
  "eventType": "Microsoft.AppConfiguration.KeyValueDeleted",
  "eventTime": "2019-05-31T20:05:03Z",
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```
# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

В следующем примере показана схема события изменения ключа "ключ — значение": 

```json
[{
  "id": "84e17ea4-66db-4b54-8050-df8f7763f87b",
  "source": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/microsoft.appconfiguration/configurationstores/contoso",
  "subject": "https://contoso.azconfig.io/kv/Foo?label=FizzBuzz",
  "data": {
    "key": "Foo",
    "label": "FizzBuzz",
    "etag": "FnUExLaj2moIi4tJX9AXn9sakm0"
  },
  "type": "Microsoft.AppConfiguration.KeyValueModified",
  "time": "2019-05-31T20:05:03Z",
  "specversion": "1.0"
}]
```

Схема для события Deleted-valued имеет следующий вид: 

```json
[{
  "id": "84e17ea4-66db-4b54-8050-df8f7763f87b",
  "source": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/microsoft.appconfiguration/configurationstores/contoso",
  "subject": "https://contoso.azconfig.io/kv/Foo?label=FizzBuzz",
  "data": {
    "key": "Foo",
    "label": "FizzBuzz",
    "etag": "FnUExLaj2moIi4tJX9AXn9sakm0"
  },
  "type": "Microsoft.AppConfiguration.KeyValueDeleted",
  "time": "2019-05-31T20:05:03Z",
  "specversion": "1.0"
}]
```

---

## <a name="event-properties"></a>Свойства события

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)
Событие содержит следующие высокоуровневые данные:

| Свойство. | Type | Описание |
| -------- | ---- | ----------- |
| `topic` | строка | Полный путь к ресурсу для источника событий. Это поле защищено от записи. Это значение предоставляет служба "Сетка событий". |
| `subject` | строка | Определенный издателем путь к субъекту событий. |
| `eventType` | строка | Один из зарегистрированных типов событий для этого источника событий. |
| `eventTime` | строка | Время создания события с учетом времени поставщика в формате UTC. |
| `id` | строка | Уникальный идентификатор события. |
| `data` | object | Данные события конфигурации приложения. |
| `dataVersion` | строка | Версия схемы для объекта данных. Версию схемы определяет издатель. |
| `metadataVersion` | строка | Версия схемы для метаданных события. Служба "Сетка событий" определяет схему свойств верхнего уровня. Это значение предоставляет служба "Сетка событий". |


# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

Событие содержит следующие высокоуровневые данные:

| Свойство. | Type | Описание |
| -------- | ---- | ----------- |
| `source` | строка | Полный путь к ресурсу для источника событий. Это поле защищено от записи. Это значение предоставляет служба "Сетка событий". |
| `subject` | строка | Определенный издателем путь к субъекту событий. |
| `type` | строка | Один из зарегистрированных типов событий для этого источника событий. |
| `time` | строка | Время создания события с учетом времени поставщика в формате UTC. |
| `id` | строка | Уникальный идентификатор события. |
| `data` | object | Данные события конфигурации приложения. |
| `specversion` | строка | Версия спецификации схемы Клаудевентс. |

---

Объект данных имеет следующие свойства:

| Свойство | Type | Описание |
| -------- | ---- | ----------- |
| `key` | строка | Ключ измененного или удаленного ключевого значения. |
| `label` | строка | Метка, если таковая имеется, для измененного или удаленного ключевого значения. |
| `etag` | строка | Для `KeyValueModified` ETag нового ключевого значения. Для `KeyValueDeleted` ETag удаленного ключевого значения. |


## <a name="tutorials-and-how-tos"></a>Руководства и инструкции

|Заголовок | Описание |
|---------|---------|
| [Реагирование на события настройки приложения Azure с помощью службы "Сетка событий"](../azure-app-configuration/concept-app-configuration-event.md?toc=%2fazure%2fevent-grid%2ftoc.json) | Общие сведения об интеграции конфигурации приложений Azure с сеткой событий. |
| [Уведомление об изменении данных с помощью Сетки событий](../azure-app-configuration/howto-app-configuration-event.md?toc=%2fazure%2fevent-grid%2ftoc.json) | Узнайте, как использовать подписки на события конфигурации приложений Azure для отправки событий изменения значения ключа в веб-конечную точку. |

## <a name="next-steps"></a>Дальнейшие действия

* См. общие сведения о [службе "Сетка событий Azure"](overview.md).
* Дополнительные сведения о создании подписки на Сетку событий Azure см. в статье [Схема подписки для службы "Сетка событий"](subscription-creation-schema.md).
* Общие сведения о работе с событиями конфигурации приложений Azure см. в статье Использование службы " [Сетка событий" для уведомлений об изменении данных](../azure-app-configuration/howto-app-configuration-event.md?toc=%2fazure%2fevent-grid%2ftoc.json). 