---
title: Получение API операций в Azure Marketplace
description: API для получения всех операций в предложении или для получения определенной операции для указанного элемента с идентификатором.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
author: mingshen-ms
ms.author: mingshen
ms.date: 07/14/2020
ms.openlocfilehash: cb44d977407a7e854603e6bbacf3591752b109c2
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87271949"
---
# <a name="retrieve-operations"></a>Получение операций

> [!NOTE]
> Портал Cloud Partner API интегрированы с и будут продолжать работать в центре партнеров. Переход содержит небольшие изменения. Ознакомьтесь с изменениями, приведенными в [справочнике по портал Cloud Partner API](./cloud-partner-portal-api-overview.md) , чтобы убедиться, что код продолжит работать после перехода в центр партнеров. API-интерфейсы CPP следует использовать только для существующих продуктов, которые уже были интегрированы до перехода в центр партнеров. новые продукты должны использовать API-интерфейсы отправки центра партнеров.

Получает все операции над предложением или чтобы получить определенную операцию для указанного operationId. Клиент может использовать параметры запроса для фильтрации выполнений операций.

``` https

  GET https://cloudpartner.azure.com/api/publishers/<publisherId>/offers/<offerId>/submissions/?api-version=2017-10-31&status=<filteredStatus>

  GET https://cloudpartner.azure.com/api/publishers/<publisherId>/offers/<offerId>/operations/<operationId>?api-version=2017-10-31

```

## <a name="uri-parameters"></a>Параметры универсального кода ресурса (URI)

|  **имя**;          |      **Описание**                                                                                           | **Data type** |
|  ----------------  |     --------------------------------------------------------------------------------------------------------   |  -----------  |
|  publisherid       |  Идентификатор издателя, например `Contoso`                                                                   |  Строка       |
|  offerId           |  Идентификатор предложения                                                                                              |  Строка       |
|  operationId       |  Глобальный уникальный идентификатор (GUID) идентифицирует предложение. OperationId можно получить с помощью этого API, а также она возвращается в заголовок ответа HTTP для любой долго выполняющиеся операции, например, [публикации предложения](./cloud-partner-portal-api-publish-offer.md) API.  |   Guid   |
|  api-version       | API последней версии |    Дата      |
|  |  |  |

## <a name="header"></a>Header

|  **имя**;          |  **Значение**           |
|  ---------------   | -------------------- |
|  Content-Type      | `application/json`   |
|  Авторизация     | `Bearer YOUR_TOKEN`  |
|  |  |

## <a name="body-example"></a>Пример текста запроса

### <a name="response"></a>Ответ

#### <a name="get-operations"></a>Операции GET

``` json
    [
        {
            "id": "5a63deb5-925b-4ee0-938b-7c86fbf287c5",
            "offerId": "56615b67-2185-49fe-80d2-c4ddf77bb2e8",
            "offerVersion": 1,
            "offerTypeId": "microsoft-azure-virtualmachines",
            "publisherId": "contoso",
            "submissionType": "publish",
            "submissionState": "running",
            "publishingVersion": 2,
            "slot": "staging",
            "version": 636576975611768314,
            "definition": {
                "metadata": {
                    "emails": "jdoe@contoso.com"
                }
            },
            "changedTime": "2018-03-26T21:46:01.179948Z"
        }
    ]
```

#### <a name="get-operation"></a>Операция GET

``` json
    [
        {
            "status" : "running",
            "messages" : [],
            "publishingVersion" : 2,
            "offerVersion" : 1,
            "cancellationRequestState": "canCancel",
            "steps": [
                        {
                            "estimatedTimeFrame": "< 15 min",
                            "id": "displaydummycertify",
                            "stepName": "Validate Pre-Requisites",
                            "description": "Offer settings provided are validated",
                            "status": "complete",
                            "messages": 
                            [
                                {
                                    "messageHtml": "Step completed.",
                                    "level": "information",
                                    "timestamp": "2017-03-28T19:50:36.500052Z"
                                }
                            ],
                            "progressPercentage": 100
                        },
                        {
                            "estimatedTimeFrame": "< 5 day",
                            "id": "displaycertify",
                            "stepName": "Certification",
                            "description": "Your offer is analyzed by our certification systems for issues.",
                            "status": "blocked",
                            "messages": 
                            [
                                {
                                    "messageHtml": "No virtual machine image was found for the plan contoso.",
                                    "level": "error",
                                    "timestamp": "2017-03-28T19:50:39.5506018Z"
                                },
                                {
                                    "messageHtml": "This step has not started yet.",
                                    "level": "information",
                                    "timestamp": "2017-03-28T19:50:39.5506018Z"
                                }
                            ],
                            "progressPercentage": 0
                        },
                        {
                            "estimatedTimeFrame": "< 1 day",
                            "id": "displayprovision",
                            "stepName": "Provisioning",
                            "description": "Your virtual machine is being replicated in our production systems.",
                            "status": "notStarted",
                            "messages": [],
                            "progressPercentage": 0
                        },
                        {
                            "estimatedTimeFrame": "< 1 hour",
                            "id": "displaypackage",
                            "stepName": "Packaging and Lead Generation Registration",
                            "description": "Your virtual machine is packaged for being shown to your customers. Additionally, we hookup our lead generation systems to send leads for your offer.",
                            "status": "notStarted",
                            "messages": [],
                            "progressPercentage": 0
                        },
                        {
                            "id": "publisher-signoff",
                            "stepName": "Publisher signoff",
                            "description": "Offer is available to preview. Ensure that everything looks good before making your offer live.",
                            "status": "notStarted",
                            "messages": [],
                            "progressPercentage": 0
                        },
                        {
                            "estimatedTimeFrame": "~2-5 days",
                            "id": "live",
                            "stepName": "Live",
                            "description": "Offer is publicly visible and is available for purchase.",
                            "status": "notStarted",
                            "messages": [],
                            "progressPercentage": 0
                        }
                    ],
                "previewLinks": [],
                "liveLinks": [],
            }
        }
    ]
```

### <a name="response-body-properties"></a>Свойства текста ответа

|  **имя**;                    |  **Описание**                                                                                  |
|  --------------------        |  ------------------------------------------------------------------------------------------------ |
|  идентификатор                          | Глобальный уникальный идентификатор идентифицирующий операцию.                                                       |
|  submissionType              | Определяет тип операции, которая возвращается на предложения, например `Publish/GoLive`.      |
|  createdDateTime             | Дата и время создания операции в формате UTC.                                                       |
|  lastActionDateTime          | Дата и время выполнения последнего обновления операции в формате UTC.                                       |
|  status                      | Состояние операции: `not started` \| `running` \| `failed` \| `completed` . Только одна операция может иметь состояние `running` за раз. |
|  Ошибка                       | Сообщение об ошибке, появившееся при сбое операций                                                               |
|  |  |

### <a name="response-step-properties"></a>Свойства шага ответа

|  **имя**;                    |  **Описание**                                                                                  |
|  --------------------        |  ------------------------------------------------------------------------------------------------ |
| estimatedTimeFrame | Предполагаемая длительность операции |
| идентификатор | Уникальный идентификатор процесса шага |
| description; | Описание шага |
| stepName | Понятное имя для шага |
| status | Состояние `notStarted` \| `running` шага \| : `failed` \|`completed` |
| отправляемых из облака на устройство | Все уведомления или предупреждения, возникшие на шаге. Массив строк |
| progressPercentage | Целое число от 0 до 100, указывающее ход выполнения шага |
| | |

### <a name="response-status-codes"></a>Коды состояния ответа

| **Код**  |   **Описание**                                                                                  |
|  -------- |   -------------------------------------------------------------------------------------------------|
|  200      | `OK` — запрос успешно обработан и синхронное выполнение операции отменяется.        |
|  400      | `Bad/Malformed request` — текст ответа ошибки может содержать дополнительные сведения.                    |
|  403      | `Forbidden` — клиент не имеет доступ к указанным пространствам имен.                          |
|  404      | `Not found` — указанная сущность не существует.                                                 |
|  |  |
