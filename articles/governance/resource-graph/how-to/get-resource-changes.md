---
title: Получение данных об изменении ресурса
description: Сведения о том, как узнать об изменении ресурса, как получить список измененных свойств и оценить различия.
ms.date: 01/27/2021
ms.topic: how-to
ms.openlocfilehash: 5ad86ec2598cd7f24b8e0cd2208889bb7a088568
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100594649"
---
# <a name="get-resource-changes"></a>Получение данных об изменении ресурса

Ресурсы постоянно изменяются в ходе их ежедневного использования, при перенастройке или повторном развертывании.
Изменение может быть вызвано человеком или автоматическим процессом. Большая часть изменений вносится специально, но иногда это не так. Предоставляя вам журнал изменений за последние 14 дней, Azure Resource Graph позволяет сделать следующее:

- узнать время, когда было обнаружено изменение свойства в Azure Resource Manager;
- получить подробные сведения по каждому изменению ресурса;
- увидеть полное сравнение состояний ресурса до и после обнаруженного изменения.

Обнаружение изменений и сведения о них полезны в таких сценариях:

- Изучение _потенциально_ важных изменений в процессе управления инцидентами. Получение событий изменения за определенный период времени и оценка информации об этих изменениях.
- Поддержка базы данных для управления конфигурацией в актуальном состоянии. Вы сможете получать только изменившиеся данные, а не обновлять все ресурсы с полным набором свойств.
- Изучение того, какие еще свойства могли быть изменены, когда для ресурса изменилось состояние соответствия. Оценка этих дополнительных свойств может предоставить важные сведения о других свойствах, которые следует включить в управление через определение Политики Azure.

В этой статье показано, как собирать эти сведения через пакет SDK для Resource Graph. Чтобы просмотреть эти сведения на портале Azure, воспользуйтесь [журналом изменений Политики Azure](../../policy/how-to/determine-non-compliance.md#change-history) или [журналом изменений для журнала действий Azure](../../../azure-monitor/essentials/activity-log.md#view-the-activity-log). Дополнительные сведения об изменениях в приложениях на уровне инфраструктуры вплоть до развертывания см. в статье [Использование анализа изменений в приложениях (предварительная версия) в Azure Monitor](../../../azure-monitor/app/change-analysis.md).

> [!NOTE]
> Сведения в Resource Graph приводятся для свойств Resource Manager. Сведения об отслеживании изменений в виртуальной машине см. в статьях [Общие сведения об отслеживании изменений и инвентаризации](../../../automation/change-tracking/overview.md) для службы автоматизации Azure и [Общие сведения о гостевой конфигурации для виртуальных машин](../../policy/concepts/guest-configuration.md).

> [!IMPORTANT]
> История изменений в Azure Resource Graph предоставляется в режиме предварительной версии.

## <a name="find-detected-change-events-and-view-change-details"></a>Поиск событий обнаружения изменений и просмотр сведений об изменении

Чтобы просмотреть изменения в ресурсе, прежде всего нужно найти события изменений, связанные с этим ресурсом за определенный период времени. Каждое событие изменения содержит некоторые сведения о том, что изменилось в ресурсе. Этот шаг выполняется через конечную точку REST **resourceChanges**.

Конечная точка **resourceChanges** принимает в тексте запроса следующие параметры:

- **resourceId** \[обязательный\]: ресурс Azure, для которого мы ищем изменения.
- **interval** \[обязательный\]: свойство с датами _start_ (начало) и _end_ (конец), которые определяют период для поиска изменений по часовому поясу **Zulu Time Zone (Z)** .
- **fetchPropertyChanges** (необязательный): логическое свойство, которое указывает, нужно ли включать изменения свойств в объект ответа.

Пример тела запроса:

```json
{
    "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/MyResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
    "interval": {
        "start": "2019-09-28T00:00:00.000Z",
        "end": "2019-09-29T00:00:00.000Z"
    },
    "fetchPropertyChanges": true
}
```

Для приведенного выше текста запроса применяется следующий универсальный код ресурса (URI) REST API для конечной точки **resourceChanges**:

```http
POST https://management.azure.com/providers/Microsoft.ResourceGraph/resourceChanges?api-version=2018-09-01-preview
```

Ответ выглядит примерно так:

```json
{
    "changes": [
        {
            "changeId": "{\"beforeId\":\"3262e382-9f73-4866-a2e9-9d9dbee6a796\",\"beforeTime\":\"2019-09-28T00:45:35.012Z\",\"afterId\":\"6178968e-981e-4dac-ac37-340ee73eb577\",\"afterTime\":\"2019-09-28T00:52:53.371Z\"}",
            "beforeSnapshot": {
                "snapshotId": "3262e382-9f73-4866-a2e9-9d9dbee6a796",
                "timestamp": "2019-09-28T00:45:35.012Z"
            },
            "afterSnapshot": {
                "snapshotId": "6178968e-981e-4dac-ac37-340ee73eb577",
                "timestamp": "2019-09-28T00:52:53.371Z"
            },
            "changeType": "Create"
        },
        {
            "changeId": "{\"beforeId\":\"a00f5dac-86a1-4d86-a1c5-a9f7c8147b7c\",\"beforeTime\":\"2019-09-28T00:43:38.366Z\",\"afterId\":\"3262e382-9f73-4866-a2e9-9d9dbee6a796\",\"afterTime\":\"2019-09-28T00:45:35.012Z\"}",
            "beforeSnapshot": {
                "snapshotId": "a00f5dac-86a1-4d86-a1c5-a9f7c8147b7c",
                "timestamp": "2019-09-28T00:43:38.366Z"
            },
            "afterSnapshot": {
                "snapshotId": "3262e382-9f73-4866-a2e9-9d9dbee6a796",
                "timestamp": "2019-09-28T00:45:35.012Z"
            },
            "changeType": "Delete"
        },
        {
            "changeId": "{\"beforeId\":\"b37a90d1-7ebf-41cd-8766-eb95e7ee4f1c\",\"beforeTime\":\"2019-09-28T00:43:15.518Z\",\"afterId\":\"a00f5dac-86a1-4d86-a1c5-a9f7c8147b7c\",\"afterTime\":\"2019-09-28T00:43:38.366Z\"}",
            "beforeSnapshot": {
                "snapshotId": "b37a90d1-7ebf-41cd-8766-eb95e7ee4f1c",
                "timestamp": "2019-09-28T00:43:15.518Z"
            },
            "afterSnapshot": {
                "snapshotId": "a00f5dac-86a1-4d86-a1c5-a9f7c8147b7c",
                "timestamp": "2019-09-28T00:43:38.366Z"
            },
            "propertyChanges": [
                {
                    "propertyName": "tags.org",
                    "afterValue": "compute",
                    "changeCategory": "User",
                    "changeType": "Insert"
                },
                {
                    "propertyName": "tags.team",
                    "afterValue": "ARG",
                    "changeCategory": "User",
                    "changeType": "Insert"
                }
            ],
            "changeType": "Update"
        },
        {
            "changeId": "{\"beforeId\":\"19d12ab1-6ac6-4cd7-a2fe-d453a8e5b268\",\"beforeTime\":\"2019-09-28T00:42:46.839Z\",\"afterId\":\"b37a90d1-7ebf-41cd-8766-eb95e7ee4f1c\",\"afterTime\":\"2019-09-28T00:43:15.518Z\"}",
            "beforeSnapshot": {
                "snapshotId": "19d12ab1-6ac6-4cd7-a2fe-d453a8e5b268",
                "timestamp": "2019-09-28T00:42:46.839Z"
            },
            "afterSnapshot": {
                "snapshotId": "b37a90d1-7ebf-41cd-8766-eb95e7ee4f1c",
                "timestamp": "2019-09-28T00:43:15.518Z"
            },
            "propertyChanges": [{
                "propertyName": "tags.cgtest",
                "afterValue": "hello",
                "changeCategory": "User",
                "changeType": "Insert"
            }],
            "changeType": "Update"
        }
    ]
}
```

Каждое обнаруженное событие изменения для ресурса **resourceId** имеет следующие свойства:

- **changeId** — уникальный идентификатор ресурса. Впрочем, строка **changeId** может содержать и другие свойства, гарантируется только ее уникальность.
- **beforeSnapshot** — содержит значения **snapshotId** и **timestamp** для моментального снимка этого ресурса, который был создан до обнаруженного изменения.
- **afterSnapshot** — содержит значения **snapshotId** и **timestamp** для моментального снимка этого ресурса, который был создан после обнаруженного изменения.
- **changeType** — описывает тип обнаруженных изменений за весь период между **beforeSnapshot** и **afterSnapshot**. Возможны следующие значения. _Create_ (Создание), _Update_ (Обновление) или _Delete_ (Удаление). Массив свойств **propertyChanges** включается только в том случае, если **changeType** имеет значение _Update_ (Обновление).
- **propertyChanges** — это массив сведений о всех свойствах ресурса, которые изменились за период между **beforeSnapshot** и **afterSnapshot**:
  - **propertyName** — имя свойства ресурса, которое изменилось.
  - **changeCategory** — описывает источник или причину изменений. Возможны следующие значения. _System_ (Система) и _User_ (Пользователь).
  - **changeType** — описывает тип изменений, обнаруженных для отдельного свойства ресурса.
    Возможны следующие значения. _Insert_ (Вставка), _Update_ (Обновление), _Remove_ (Удаление).
  - **beforeValue** — это значение свойства ресурса, указанное в **beforeSnapshot**. Не отображается, если **changeType** имеет значение _Insert_ (Вставка).
  - **afterValue** — это значение свойства ресурса, указанное в **afterSnapshot**. Не отображается, если **changeType** имеет значение _Remove_ (Удаление).

## <a name="compare-resource-changes"></a>Сравнение изменений ресурса

Используя **changeId**, полученный от конечной точки **resourceChanges**, конечная точка REST **resourceChangeDetails** возвращает моментальные снимки ресурса по состоянию до и после изменения.

Конечная точка **resourceChangeDetails** принимает в тексте запроса два обязательных параметра:

- **resourceId**: ресурс Azure, для которого мы ищем изменения.
- **changeId**: уникальное событие изменения для ресурса **resourceId**, полученное от конечной точки **resourceChanges**.

Пример тела запроса:

```json
{
    "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/MyResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
    "changeId": "{\"beforeId\":\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\",\"beforeTime\":'2019-05-09T00:00:00.000Z\",\"afterId\":\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\",\"afterTime\":'2019-05-10T00:00:00.000Z\"}"
}
```

Для приведенного выше текста запроса применяется следующий универсальный код ресурса (URI) REST API для конечной точки **resourceChangeDetails**:

```http
POST https://management.azure.com/providers/Microsoft.ResourceGraph/resourceChangeDetails?api-version=2018-09-01-preview
```

Ответ выглядит примерно так:

```json
{
    "changeId": "{\"beforeId\":\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\",\"beforeTime\":'2019-05-09T00:00:00.000Z\",\"afterId\":\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\",\"beforeTime\":'2019-05-10T00:00:00.000Z\"}",
    "beforeSnapshot": {
        "timestamp": "2019-03-29T01:32:05.993Z",
        "content": {
            "sku": {
                "name": "Standard_LRS",
                "tier": "Standard"
            },
            "kind": "Storage",
            "id": "/subscriptions/{subscriptionId}/resourceGroups/MyResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
            "name": "mystorageaccount",
            "type": "Microsoft.Storage/storageAccounts",
            "location": "westus",
            "tags": {},
            "properties": {
                "networkAcls": {
                    "bypass": "AzureServices",
                    "virtualNetworkRules": [],
                    "ipRules": [],
                    "defaultAction": "Allow"
                },
                "supportsHttpsTrafficOnly": false,
                "encryption": {
                    "services": {
                        "file": {
                            "enabled": true,
                            "lastEnabledTime": "2018-07-27T18:37:21.8333895Z"
                        },
                        "blob": {
                            "enabled": true,
                            "lastEnabledTime": "2018-07-27T18:37:21.8333895Z"
                        }
                    },
                    "keySource": "Microsoft.Storage"
                },
                "provisioningState": "Succeeded",
                "creationTime": "2018-07-27T18:37:21.7708872Z",
                "primaryEndpoints": {
                    "blob": "https://mystorageaccount.blob.core.windows.net/",
                    "queue": "https://mystorageaccount.queue.core.windows.net/",
                    "table": "https://mystorageaccount.table.core.windows.net/",
                    "file": "https://mystorageaccount.file.core.windows.net/"
                },
                "primaryLocation": "westus",
                "statusOfPrimary": "available"
            }
        }
    },
    "afterSnapshot": {
        "timestamp": "2019-03-29T01:54:24.42Z",
        "content": {
            "sku": {
                "name": "Standard_LRS",
                "tier": "Standard"
            },
            "kind": "Storage",
            "id": "/subscriptions/{subscriptionId}/resourceGroups/MyResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
            "name": "mystorageaccount",
            "type": "Microsoft.Storage/storageAccounts",
            "location": "westus",
            "tags": {},
            "properties": {
                "networkAcls": {
                    "bypass": "AzureServices",
                    "virtualNetworkRules": [],
                    "ipRules": [],
                    "defaultAction": "Allow"
                },
                "supportsHttpsTrafficOnly": true,
                "encryption": {
                    "services": {
                        "file": {
                            "enabled": true,
                            "lastEnabledTime": "2018-07-27T18:37:21.8333895Z"
                        },
                        "blob": {
                            "enabled": true,
                            "lastEnabledTime": "2018-07-27T18:37:21.8333895Z"
                        }
                    },
                    "keySource": "Microsoft.Storage"
                },
                "provisioningState": "Succeeded",
                "creationTime": "2018-07-27T18:37:21.7708872Z",
                "primaryEndpoints": {
                    "blob": "https://mystorageaccount.blob.core.windows.net/",
                    "queue": "https://mystorageaccount.queue.core.windows.net/",
                    "table": "https://mystorageaccount.table.core.windows.net/",
                    "file": "https://mystorageaccount.file.core.windows.net/"
                },
                "primaryLocation": "westus",
                "statusOfPrimary": "available"
            }
        }
    }
}
```

Каждый из моментальных снимков **beforeSnapshot** и **afterSnapshot** содержит время своего создания и все свойства по состоянию на этот момент. Известно, что изменение произошло в некоторый момент между этими снимками. Взглянув на предыдущий пример, можно увидеть, что измененное свойство было **supportsHttpsTrafficOnly**.

Чтобы сравнить результаты, используйте свойство **changes** в ответе **resourceChanges** или **содержимое** обоих моментальных снимков в ответе **resourceChangeDetails**. При сравнении моментальных снимков для значения **timestamp** всегда отображается наличие различий, хотя это и ожидаемо.

## <a name="next-steps"></a>Дальнейшие действия

- См. описание используемого языка в разделе [Запросы для начинающих](../samples/starter.md).
- См. описание расширенных вариантов использования в разделе [Расширенные запросы](../samples/advanced.md).
- Узнайте больше о том, как [изучать ресурсы](../concepts/explore-resources.md).
- Рекомендации по работе с запросами с высокой частотой см. в статье [Руководство по регулируемым запросам](../concepts/guidance-for-throttled-requests.md).
