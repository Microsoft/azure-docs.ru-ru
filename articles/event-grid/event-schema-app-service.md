---
title: Служба приложений Azure в качестве источника Сетки событий
description: В этой статье описывается, как использовать Службу приложений Azure в качестве источника событий Сетки событий. Она содержит схему и ссылки на учебники и практические руководства.
author: jasonfreeberg
ms.topic: conceptual
ms.date: 03/06/2021
ms.author: jafreebe
ms.openlocfilehash: 404fac634a628da49dee72b10b52785820fe1bf6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102443783"
---
# <a name="azure-app-service-as-an-event-grid-source"></a>Служба приложений Azure в качестве источника Сетки событий

В этой статье описаны свойства и схема для событий Службы приложений Azure. Общие сведения о схемах событий см. в статье [Схема событий службы "Сетка событий Azure"](event-schema.md). Здесь также приводится список кратких руководств и учебников по использованию Службы приложений Azure в качестве источника событий.

## <a name="available-event-types"></a>Доступные типы событий

Служба приложений Azure выдает приведенные ниже типы событий.

|    Тип события                                             |    Описание                                                     |
|-----------------------------------------------------------|--------------------------------------------------------------------|
|    Microsoft.Web/sites.BackupOperationStarted             |    Активируется при запуске резервного копирования.                             |
|    Microsoft.Web/sites.BackupOperationCompleted           |    Активируется при завершении резервного копирования.                           |
|    Microsoft.Web/sites.BackupOperationFailed              |    Активируется при сбое резервного копирования.                              |
|    Microsoft.Web/sites.RestoreOperationStarted            |    Активируется при запуске восстановления из резервной копии.        |
|    Microsoft.Web/sites.RestoreOperationCompleted          |    Активируется при завершении восстановления из резервной копии.      |
|    Microsoft.Web/sites.RestoreOperationFailed             |    Активируется при сбое восстановления из резервной копии.         |
|    Microsoft.Web/sites.SlotSwapStarted                    |    Активируется при запуске переключения слотов.                          |
|    Microsoft.Web/sites.SlotSwapCompleted                  |    Активируется при завершении переключения слотов.                      |
|    Microsoft.Web/sites.SlotSwapFailed                     |    Активируется при сбое переключения слотов.                           |
|    Microsoft.Web/sites.SlotSwapWithPreviewStarted         |    Активируется при запуске переключения слотов с предварительным просмотром.           |
|    Microsoft.Web/sites.SlotSwapWithPreviewCancelled       |    Активируется при отмене переключения слотов с предварительным просмотром.    |
|    Microsoft.Web/sites.AppUpdated.Restarted               |    Активируется при перезапуске сайта.                      |
|    Microsoft.Web/sites.AppUpdated.Stopped                 |    Активируется при остановке сайта.                          |
|    Microsoft.Web/sites.AppUpdated.ChangedAppSettings      |    Активируется при изменении параметров приложения сайта.             |
|    Microsoft.Web/serverfarms.AppServicePlanUpdated        |    Активируется при обновлении плана службы приложений.                 |

## <a name="properties-common-to-all-events"></a>Свойства, общие для всех событий

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)
При активации события служба Сетки событий отправляет данные о нем на подписанную конечную точку.
В этом разделе приведен пример того, как будут выглядеть данные для каждого события. Каждое событие содержит приведенные ниже данные верхнего уровня.

|     Свойство          |     Type     |     Описание                                                                                                                                |
|-----------------------|--------------|------------------------------------------------------------------------------------------------------------------------------------------------|
|    `topic`              |    строка    |    Полный путь к ресурсу источника событий. Это поле защищено от записи. Это значение предоставляет служба "Сетка событий".                                      |
|    `subject`            |    строка    |    Определенный издателем путь к субъекту события.                                                                                              |
|    `eventType`          |    строка    |    Один из зарегистрированных типов событий для этого источника событий.                                                                                  |
|    `eventTime`          |    строка    |    Время создания события с учетом времени поставщика в формате UTC.                                                                         |
|    `id`                 |    строка    |    Уникальный идентификатор события.                                                                                                            |
|    `data`               |    объект    |    Данные события хранилища BLOB-объектов.                                                                                                                    |
|    `dataVersion`        |    строка    |    Версия схемы для объекта данных. Версию схемы определяет издатель.                                                          |
|    `metadataVersion`    |    строка    |    Версия схемы для метаданных события. Служба "Сетка событий" определяет схему свойств верхнего уровня. Это значение предоставляет Сетка событий.    |

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

При активации события служба Сетки событий отправляет данные о нем на подписанную конечную точку.
В этом разделе приведен пример того, как будут выглядеть данные для каждого события. Каждое событие содержит приведенные ниже данные верхнего уровня.

|     Свойство          |     Type     |     Описание                                                                                                                                |
|-----------------------|--------------|------------------------------------------------------------------------------------------------------------------------------------------------|
|    `source`              |    строка    |    Полный путь к ресурсу источника событий. Это поле защищено от записи. Это значение предоставляет служба "Сетка событий".                                      |
|    `subject`            |    строка    |    Определенный издателем путь к субъекту события.                                                                                              |
|    `type`          |    строка    |    Один из зарегистрированных типов событий для этого источника событий.                                                                                  |
|    `time`          |    строка    |    Время создания события с учетом времени поставщика в формате UTC.                                                                         |
|    `id`                 |    строка    |    Уникальный идентификатор события.                                                                                                            |
|    `data`               |    объект    |    Данные события хранилища BLOB-объектов.                                                                                                                    |
| `specversion` | строка | Версия спецификации схемы Клаудевентс. |

---

## <a name="example-events"></a>Примеры событий

### <a name="backupoperationstarted-backupoperationcompleted-backupoperationfailed"></a>BackupOperationStarted, BackupOperationCompleted, BackupOperationFailed

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

```json
{
    "id": "7c5d6de5-eb70-4de2-b788-c52a544e68b8",
    "topic": "/subscriptions/<id>/resourceGroups/<rg>/providers/Microsoft.Web/sites/<site-name>",
    "subject": "/Microsoft.Web/sites/<site-name>",
    "eventType": "Microsoft.Web.BackupOperationStarted",
    "eventTime": "2020-01-28T18:26:51.7194887Z",
    "data": {
        "appEventTypeDetail": {
            "action": "Started"
        },
        "name": "<site-name>",
        "clientRequestId": "None",
        "correlationRequestId": "None",
        "requestId": "292f499d-04ee-4066-994d-c2df57b99198",
        "address": "None",
        "verb": "None"
    },
    "dataVersion": "1",
    "metaDataVersion": "1"
}
```
# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

```json
{
    "id": "7c5d6de5-eb70-4de2-b788-c52a544e68b8",
    "source": "/subscriptions/<id>/resourceGroups/<rg>/providers/Microsoft.Web/sites/<site-name>",
    "subject": "/Microsoft.Web/sites/<site-name>",
    "type": "Microsoft.Web.BackupOperationStarted",
    "time": "2020-01-28T18:26:51.7194887Z",
    "data": {
        "appEventTypeDetail": {
            "action": "Started"
        },
        "name": "<site-name>",
        "clientRequestId": "None",
        "correlationRequestId": "None",
        "requestId": "292f499d-04ee-4066-994d-c2df57b99198",
        "address": "None",
        "verb": "None"
    },
    "specversion": "1.0"
}
```

---

Объект данных содержит следующие свойства.

|    Свойство                |    Type      |    Описание                                                                                                       |
|----------------------------|--------------|----------------------------------------------------------------------------------------------------------------------|
|    `appEventTypeDetail`      |    объект    |    Сведения о действии в приложении.                                                                                       |
|    `action`                  |    строка    |    Тип действия операции.                                                                                   |
|    `name`                    |    строка    |    Имя веб-сайта с этим событием.                                                                          |
|    `clientRequestId`         |    строка    |    Идентификатор запроса клиента, созданный службой приложений для операции API сайта, вызвавшей это событие         |
|    `correlationRequestId`    |    строка    |    Идентификатор запроса корреляции, созданный службой приложений для операции API сайта, вызвавшей это событие    |
|    `requestId`               |    строка    |    Идентификатор запроса, сформированный службой приложений для операции API сайта, вызвавшей это событие                |
|    `address`                 |    строка    |    URL-адрес HTTP-запроса для этой операции.                                                                                |
|    `verb`                    |    строка    |    HTTP-команда для этой операции.                                                                                       |

### <a name="restoreoperationstarted-restoreoperationcompleted-restoreoperationfailed"></a>RestoreOperationStarted, RestoreOperationCompleted, RestoreOperationFailed

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

```json
{
    "id": "7c5d6de5-eb70-4de2-b788-c52a544e68b8",
    "topic": "/subscriptions/<id>/resourceGroups/<rg>/providers/Microsoft.Web/sites/<site-name>",
    "subject": "/Microsoft.Web/sites/<site-name>",
    "eventType": "Microsoft.Web.RestoreOperationStarted",
    "eventTime": "2020-01-28T18:26:51.7194887Z",
    "data": {
        "appEventTypeDetail": {
            "action": "Started"
        },
        "name": "<site-name>",
        "clientRequestId": "None",
        "correlationRequestId": "None",
        "requestId": "292f499d-04ee-4066-994d-c2df57b99198",
        "address": "None",
        "verb": "POST"
    },
    "dataVersion": "1",
    "metaDataVersion": "1"
}
```

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

```json
{
    "id": "7c5d6de5-eb70-4de2-b788-c52a544e68b8",
    "source": "/subscriptions/<id>/resourceGroups/<rg>/providers/Microsoft.Web/sites/<site-name>",
    "subject": "/Microsoft.Web/sites/<site-name>",
    "type": "Microsoft.Web.RestoreOperationStarted",
    "time": "2020-01-28T18:26:51.7194887Z",
    "data": {
        "appEventTypeDetail": {
            "action": "Started"
        },
        "name": "<site-name>",
        "clientRequestId": "None",
        "correlationRequestId": "None",
        "requestId": "292f499d-04ee-4066-994d-c2df57b99198",
        "address": "None",
        "verb": "POST"
    },
    "specversion": "1.0"
}
```

---

Объект данных содержит следующие свойства.

|    Свойство                |    Type      |    Описание                                                                                                       |
|----------------------------|--------------|----------------------------------------------------------------------------------------------------------------------|
|    `appEventTypeDetail`      |    объект    |    Сведения о действии в приложении.                                                                                       |
|    `action`                  |    строка    |    Тип действия операции.                                                                                   |
|    `name`                    |    строка    |    Имя веб-сайта с этим событием.                                                                          |
|    `clientRequestId`         |    строка    |    Идентификатор запроса клиента, созданный службой приложений для операции API сайта, вызвавшей это событие         |
|    `correlationRequestId`    |    строка    |    Идентификатор запроса корреляции, созданный службой приложений для операции API сайта, вызвавшей это событие    |
|    `requestId`               |    строка    |    Идентификатор запроса, сформированный службой приложений для операции API сайта, вызвавшей это событие                |
|    `address`                 |    строка    |    URL-адрес HTTP-запроса для этой операции.                                                                                |
|    `verb`                    |    строка    |    HTTP-команда для этой операции.                                                                                       |

### <a name="slotswapstarted-slotswapcompleted-slotswapfailed"></a>SlotSwapStarted, SlotSwapCompleted, SlotSwapFailed

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

```json
{
    "id": "7c5d6de5-eb70-4de2-b788-c52a544e68b8",
    "topic": "/subscriptions/<id>/resourceGroups/<rg>/providers/Microsoft.Web/sites/<site-name>",
    "subject": "/Microsoft.Web/sites/<site-name>",
    "eventType": "Microsoft.Web.SlotSwapStarted",
    "eventTime": "2020-01-28T18:26:51.7194887Z",
    "data": {
        "appEventTypeDetail": null,
        "name": "<site-name>",
        "clientRequestId": "922f4841-20d9-4dd6-8c5b-23f0d85e5592",
        "correlationRequestId": "9ac46505-2b8a-4e06-834c-05ffbe2e8c3a",
        "requestId": "765117aa-eaf8-4bd2-a644-1dbf69c7b0fd",
        "address": "/websystems/WebSites/web/subscriptions/<id>/webspaces/<webspace>/sites/<site-name>/slots?Command=SWAP&targetSlot=production",
        "verb": "POST",
        "sourceSlot": "staging",
        "targetSlot": "production"
    },
    "dataVersion": "1",
    "metaDataVersion": "1"
}
```

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

```json
{
    "id": "7c5d6de5-eb70-4de2-b788-c52a544e68b8",
    "source": "/subscriptions/<id>/resourceGroups/<rg>/providers/Microsoft.Web/sites/<site-name>",
    "subject": "/Microsoft.Web/sites/<site-name>",
    "type": "Microsoft.Web.SlotSwapStarted",
    "time": "2020-01-28T18:26:51.7194887Z",
    "data": {
        "appEventTypeDetail": null,
        "name": "<site-name>",
        "clientRequestId": "922f4841-20d9-4dd6-8c5b-23f0d85e5592",
        "correlationRequestId": "9ac46505-2b8a-4e06-834c-05ffbe2e8c3a",
        "requestId": "765117aa-eaf8-4bd2-a644-1dbf69c7b0fd",
        "address": "/websystems/WebSites/web/subscriptions/<id>/webspaces/<webspace>/sites/<site-name>/slots?Command=SWAP&targetSlot=production",
        "verb": "POST",
        "sourceSlot": "staging",
        "targetSlot": "production"
    },
    "specversion": "1.0"
}
```

---

Объект данных содержит следующие свойства.

|    Свойство                |    Type      |    Описание                                                                                                       |
|----------------------------|--------------|----------------------------------------------------------------------------------------------------------------------|
|    `appEventTypeDetail`      |    объект    |    Сведения о действии в приложении.                                                                                       |
|    `action`                 |    строка    |    Тип действия операции.                                                                                   |
|    `name`                    |    строка    |    Имя веб-сайта с этим событием.                                                                          |
|    `clientRequestId`         |    строка    |    Идентификатор запроса клиента, созданный службой приложений для операции API сайта, вызвавшей это событие         |
|    `correlationRequestId`    |    строка    |    Идентификатор запроса корреляции, созданный службой приложений для операции API сайта, вызвавшей это событие    |
|   `requestId`               |    строка    |    Идентификатор запроса, сформированный службой приложений для операции API сайта, вызвавшей это событие                |
|    `address`                 |    строка    |    URL-адрес HTTP-запроса для этой операции.                                                                                |
|    `verb`                    |    строка    |    HTTP-команда для этой операции.                                                                                       |
|    `sourceSlot`              |    строка    |    Исходный слот для переключения.                                                                                       |

### <a name="slotswapwithpreviewstarted-slotswapwithpreviewcancelled"></a>SlotSwapWithPreviewStarted, SlotSwapWithPreviewCancelled

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

```json
{
    "id": "7c5d6de5-eb70-4de2-b788-c52a544e68b8",
    "topic": "/subscriptions/<id>/resourceGroups/<rg>/providers/Microsoft.Web/sites/<site-name>",
    "subject": "/Microsoft.Web/sites/<site-name>",
    "eventType": "Microsoft.Web.SlotSwapWithPreviewStarted",
    "eventTime": "2020-01-28T18:26:51.7194887Z",
    "data": {
        "appEventTypeDetail": null,
        "name": "<site-name>",
        "clientRequestId": "922f4841-20d9-4dd6-8c5b-23f0d85e5592",
        "correlationRequestId": "9ac46505-2b8a-4e06-834c-05ffbe2e8c3a",
        "requestId": "765117aa-eaf8-4bd2-a644-1dbf69c7b0fd",
        "address": "/websystems/WebSites/web/subscriptions/<id>/webspaces/<webspace>/sites/<site-name>/slots?Command=SWAP&targetSlot=production",
        "verb": "POST",
        "sourceSlot": "staging",
        "targetSlot": "production"
    },
    "dataVersion": "1",
    "metaDataVersion": "1"
}
```

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

```json
{
    "id": "7c5d6de5-eb70-4de2-b788-c52a544e68b8",
    "source": "/subscriptions/<id>/resourceGroups/<rg>/providers/Microsoft.Web/sites/<site-name>",
    "subject": "/Microsoft.Web/sites/<site-name>",
    "type": "Microsoft.Web.SlotSwapWithPreviewStarted",
    "time": "2020-01-28T18:26:51.7194887Z",
    "data": {
        "appEventTypeDetail": null,
        "name": "<site-name>",
        "clientRequestId": "922f4841-20d9-4dd6-8c5b-23f0d85e5592",
        "correlationRequestId": "9ac46505-2b8a-4e06-834c-05ffbe2e8c3a",
        "requestId": "765117aa-eaf8-4bd2-a644-1dbf69c7b0fd",
        "address": "/websystems/WebSites/web/subscriptions/<id>/webspaces/<webspace>/sites/<site-name>/slots?Command=SWAP&targetSlot=production",
        "verb": "POST",
        "sourceSlot": "staging",
        "targetSlot": "production"
    },
    "specversion": "1.0"
}
```

---

Объект данных содержит следующие свойства.

|    Свойство                |    Type      |    Описание                                                                                                       |
|----------------------------|--------------|----------------------------------------------------------------------------------------------------------------------|
|    `appEventTypeDetail`      |    объект    |    Сведения о действии в приложении.                                                                                       |
|    `action`                 |    строка    |    Тип действия операции.                                                                                   |
|    `name`                    |    строка    |    Имя веб-сайта с этим событием.                                                                          |
|    `clientRequestId`         |    строка    |    Идентификатор запроса клиента, созданный службой приложений для операции API сайта, вызвавшей это событие         |
|    `correlationRequestId`    |    строка    |    Идентификатор запроса корреляции, созданный службой приложений для операции API сайта, вызвавшей это событие    |
|    `requestId`               |    строка    |    Идентификатор запроса, сформированный службой приложений для операции API сайта, вызвавшей это событие                |
|    `address`                 |    строка    |    URL-адрес HTTP-запроса для этой операции.                                                                                |
|    `verb`                    |    строка    |    HTTP-команда для этой операции.                                                                                       |

### <a name="appupdatedrestarted-appupdatedstopped-appupdatedchangedappsettings"></a>AppUpdated.Restarted, AppUpdated.Stopped, AppUpdated.ChangedAppSettings

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

```json
{
    "id": "b74ea56b-2a3f-4de5-a5d7-38e60c81cf23",
    "topic": "/subscriptions/<id>/resourceGroups/<group>/providers/Microsoft.Web/sites/<site-name>",
    "subject": "/Microsoft.Web/sites/<site-name>",
    "eventType": "Microsoft.Web.AppUpdated",
    "eventTime": "2020-01-28T18:22:30.2760952Z",
    "data": {
        "appEventTypeDetail": {
            "action": "Stopped"
        },
        "name": "<site-name>",
        "clientRequestId": "64a5e0aa-7cee-4ff1-9093-b9197b820014",
        "correlationRequestId": "25bb36a5-8f6c-4f04-b615-e9a0ee045756",
        "requestId": "f2e8eb3f-b190-42de-b99e-6acefe587374",
        "address": "/websystems/WebSites/web/subscriptions/<id>/webspaces/<webspace>/sites/<site-name>/stop",
        "verb": "POST"
    },
    "dataVersion": "1'",
    "metaDataVersion": "1"
}
```

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

```json
{
    "id": "b74ea56b-2a3f-4de5-a5d7-38e60c81cf23",
    "source": "/subscriptions/<id>/resourceGroups/<group>/providers/Microsoft.Web/sites/<site-name>",
    "subject": "/Microsoft.Web/sites/<site-name>",
    "type": "Microsoft.Web.AppUpdated",
    "time": "2020-01-28T18:22:30.2760952Z",
    "data": {
        "appEventTypeDetail": {
            "action": "Stopped"
        },
        "name": "<site-name>",
        "clientRequestId": "64a5e0aa-7cee-4ff1-9093-b9197b820014",
        "correlationRequestId": "25bb36a5-8f6c-4f04-b615-e9a0ee045756",
        "requestId": "f2e8eb3f-b190-42de-b99e-6acefe587374",
        "address": "/websystems/WebSites/web/subscriptions/<id>/webspaces/<webspace>/sites/<site-name>/stop",
        "verb": "POST"
    },
    "specversion": "1.0"
}
```

---

Объект данных имеет следующие свойства:

|    Свойство                |    Type      |    Описание                                                                                                       |
|----------------------------|--------------|----------------------------------------------------------------------------------------------------------------------|
|    `appEventTypeDetail`      |    объект    |    Сведения о действии в приложении.                                                                                       |
|    `action`                  |    строка    |    Тип действия операции.                                                                                   |
|    `name`                    |    строка    |    Имя веб-сайта с этим событием.                                                                          |
|    `clientRequestId`         |    строка    |    Идентификатор запроса клиента, созданный службой приложений для операции API сайта, вызвавшей это событие         |
|    `correlationRequestId`    |    строка    |    Идентификатор запроса корреляции, созданный службой приложений для операции API сайта, вызвавшей это событие    |
|    `requestId`               |    строка    |    Идентификатор запроса, сформированный службой приложений для операции API сайта, вызвавшей это событие                |
|    `address`                 |    строка    |    URL-адрес HTTP-запроса для этой операции.                                                                                |
|    `verb`                    |    строка    |    HTTP-команда для этой операции.                                                                                       |

### <a name="serverfarmsappserviceplanupdated"></a>Serverfarms.AppServicePlanUpdated

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

```json
{
    "id": "56501672-9150-40e1-893a-18420c7fdbf7",
    "topic": "/subscriptions/<id>/resourceGroups/<rg>/providers/Microsoft.Web/serverfarms/<serverfarm-name>",
    "subject": "/Microsoft.Web/serverfarms/<plan-name>",
    "eventType": "Microsoft.Web.AppServicePlanUpdated",
    "eventTime": "2020-01-28T18:22:23.5516004Z",
    "data": {
        "serverFarmEventTypeDetail": {
            "stampKind": "Public",
            "action": "Updated",
            "status": "Started"
        },
        "serverFarmId": "0",
        "sku": {
            "name": "P1v2",
            "tier": "PremiumV2",
            "size": "P1v2",
            "family": "Pv2",
            "capacity": 1
        },
        "clientRequestId": "8f880321-a991-45c7-b743-6ff63fe4c004",
        "correlationRequestId": "1995c3be-ba7f-4ccf-94af-516df637ec8a",
        "requestId": "b973a8e6-6949-4783-b44c-ac778be831bb",
        "address": "/websystems/WebSites/serverfarms/subscriptions/<id>/webspaces/<webspace-id>/serverfarms/<plan-name>/async",
        "verb": "PUT"
    },
    "dataVersion": "1",
    "metaDataVersion": "1"
}
```

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

```json
{
    "id": "56501672-9150-40e1-893a-18420c7fdbf7",
    "source": "/subscriptions/<id>/resourceGroups/<rg>/providers/Microsoft.Web/serverfarms/<serverfarm-name>",
    "subject": "/Microsoft.Web/serverfarms/<plan-name>",
    "type": "Microsoft.Web.AppServicePlanUpdated",
    "time": "2020-01-28T18:22:23.5516004Z",
    "data": {
        "serverFarmEventTypeDetail": {
            "stampKind": "Public",
            "action": "Updated",
            "status": "Started"
        },
        "serverFarmId": "0",
        "sku": {
            "name": "P1v2",
            "tier": "PremiumV2",
            "size": "P1v2",
            "family": "Pv2",
            "capacity": 1
        },
        "clientRequestId": "8f880321-a991-45c7-b743-6ff63fe4c004",
        "correlationRequestId": "1995c3be-ba7f-4ccf-94af-516df637ec8a",
        "requestId": "b973a8e6-6949-4783-b44c-ac778be831bb",
        "address": "/websystems/WebSites/serverfarms/subscriptions/<id>/webspaces/<webspace-id>/serverfarms/<plan-name>/async",
        "verb": "PUT"
    },
    "specversion": "1.0"
}
```

---

Объект данных имеет следующие свойства:

|    Свойство                         |    Type      |    Описание                                                                                                       |
|-------------------------------------|--------------|----------------------------------------------------------------------------------------------------------------------|
|    `appServicePlanEventTypeDetail`    |    объект    |    Сведения о действии для плана службы приложений.                                                                          |
|    `stampKind`                        |    строка    |    Тип среды, в которой находится план службы приложений.                                                                     |
|    `action`                           |    строка    |    Тип действия для плана службы приложений.                                                                            |
|    `status`                           |    строка    |    Состояние операции для плана службы приложений.                                                                   |
|    `sku`                              |    объект    |    Номер SKU плана службы приложений.                                                                                       |
|    `name`                             |    строка    |    Имя плана службы приложений.                                                                                      |
|    `Tier`                             |    строка    |    Уровень плана службы приложений.                                                                                      |
|    `Size`                             |    строка    |    Размер плана службы приложений.                                                                                      |
|    `Family`                           |    строка    |    Семейство плана службы приложений.                                                                                        |
|    `Capacity`                         |    строка    |    Емкость плана службы приложений.                                                                                      |
|    `action`                           |    строка    |    Тип действия операции.                                                                                   |
|    `name`                             |    строка    |    Имя веб-сайта с этим событием.                                                                          |
|    `clientRequestId`                  |    строка    |    Идентификатор запроса клиента, созданный службой приложений для операции API сайта, вызвавшей это событие         |
|    `correlationRequestId`             |    строка    |    Идентификатор запроса корреляции, созданный службой приложений для операции API сайта, вызвавшей это событие    |
|    `requestId`                        |    строка    |    Идентификатор запроса, сформированный службой приложений для операции API сайта, вызвавшей это событие                |
|    `address`                         |    строка    |    URL-адрес HTTP-запроса для этой операции.                                                                                |
|    `verb`                             |    строка    |    HTTP-команда для этой операции.                                                                                       |

## <a name="next-steps"></a>Дальнейшие действия

* См. общие сведения о [службе "Сетка событий Azure"](overview.md).
* Дополнительные сведения о создании подписки на службу "Сетка событий Azure" см. в разделе [Схема подписки службы "Сетка событий](subscription-creation-schema.md) ".