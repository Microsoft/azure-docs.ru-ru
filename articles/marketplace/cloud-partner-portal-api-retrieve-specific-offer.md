---
title: Получение API-интерфейса для конкретного предложения в Azure Marketplace
description: API для получения указанного предложения в пространстве имен издателя.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
author: mingshen-ms
ms.author: mingshen
ms.date: 07/14/2020
ms.openlocfilehash: 9f3ba6b2f13b9f2bb1d538db84723e3a9baaef12
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87271847"
---
# <a name="retrieve-a-specific-offer"></a>Получение определенного предложения

> [!NOTE]
> Портал Cloud Partner API интегрированы с и будут продолжать работать в центре партнеров. Переход содержит небольшие изменения. Ознакомьтесь с изменениями, приведенными в [справочнике по портал Cloud Partner API](./cloud-partner-portal-api-overview.md) , чтобы убедиться, что код продолжит работать после перехода в центр партнеров. API-интерфейсы CPP следует использовать только для существующих продуктов, которые уже были интегрированы до перехода в центр партнеров. новые продукты должны использовать API-интерфейсы отправки центра партнеров.

Получает определенное предложение в пространстве имен издателя.  

Можно также получить конкретную версию предложения или предложение в черновике, представление или рабочие слоты. Если слот не задан, по умолчанию используется `draft`. Попытка получить предложение, которое не было опубликовано или не выходило в предварительной версии, приведет к ошибке `404 Not Found`.

> [!WARNING]
> Эти секретные значения для полей секретного типа не будут получены этим API.

``` http
    GET https://cloudpartner.azure.com/api/publishers/<publisherId>/offers/<offerId>?api-version=2017-10-31

    GET https://cloudpartner.azure.com/api/publishers/<publisherId>/offers/<offerId>/versions/<version>?api-version=2017-10-31

    GET https://cloudpartner.azure.com/api/publishers/<publisherId>/offers/<offerId>/slot/<slotId>?api-version=2017-10-31

```

## <a name="uri-parameters"></a>Параметры универсального кода ресурса (URI)

| **имя**;    | **Описание**                                                                          | **Data type** |
|-------------|------------------------------------------------------------------------------------------|---------------|
| publisherid | publisherId Например, Contoso                                                        | Строка        |
| offerId     | Глобальный уникальный идентификатор уникально идентифицирующий предложение.                                                 | Строка        |
| version     | Версия извлекаемого предложения. По умолчанию извлекается последняя версия предложения. | Целое число       |
| slotId      | Слот, из которого извлекается предложение, может принимать одно из значений:      <br/>  - `Draft` (по умолчанию) извлекает версию предложения, которое в настоящее время находится в черновике.  <br/>  -  `Preview` извлекает версию предложения, которое в настоящее время находится в предварительной версии.     <br/>  -  `Production` извлекает версию предложения, которое в настоящее время находится в разработке.          |      enum |
| api-version | API последней версии                                                                    | Дата          |
|  |  |  |

## <a name="header"></a>Header

|  **имя**;          |   **Значение**            |
|  ---------------   |  --------------        |
|  Content-Type      | `application/json`     |
|  Авторизация     | `Bearer YOUR_TOKEN`    |
|  |  |

## <a name="body-example"></a>Пример текста запроса

### <a name="response"></a>Ответ

``` json
{
    "offerTypeId": "microsoft-azure-virtualmachines",
    "publisherId": "contoso",
    "status": "failed",
    "id": "059afc24-07de-4126-b004-4e42a51816fe",
    "version": 5,
    "definition": {
        "displayText": "Contoso Virtual Machine Offer",
        "offer": {
            "microsoft-azure-marketplace-testdrive.enabled": false,
            "microsoft-azure-marketplace-testdrive.videos": [],
            "microsoft-azure-marketplace.title": "Contoso App",
            "microsoft-azure-marketplace.summary": "Contoso App makes dev ops a breeze",
            "microsoft-azure-marketplace.longSummary": "Contoso App makes dev ops a breeze",
            "microsoft-azure-marketplace.description": "Contoso App makes dev ops a breeze",
            "microsoft-azure-marketplace.offerMarketingUrlIdentifier": "contosoapp",
            "microsoft-azure-marketplace.allowedSubscriptions": [
                "59160c40-2e25-4dcf-a2fd-6514cb08bf08"
            ],
            "microsoft-azure-marketplace.usefulLinks": [
            {
                "linkTitle": "Contoso App for Azure",
                "linkUrl": "https://azuremarketplace.microsoft.com"
            }
            ],
                "microsoft-azure-marketplace.categoryMap": [
                {
                "categoryL1": "analytics",
                "categoryL2-analytics": [
                "visualization-and-reporting"
                ]
                },
                {
                "categoryL1": "ai-plus-machine-learning",
                "categoryL2-ai-plus-machine-learning": [
                "bot-services",
                "cognitive-services",
                "other"
                ]
                }
            ],
            "microsoft-azure-marketplace.smallLogo": "https://publishingapistore.blob.core.windows.net/testcontent/D6191_publishers_contoso/contosovirtualmachine/6218c455-9cbc-450c-9920-f2e7a69ee132.png?sv=2014-02-14&sr=b&sig=6O8MM9dgiJ48VK0MwddkyVbprRAnBszyhVkVHGShhkI%3D&se=2019-03-28T19%3A46%3A50Z&sp=r",
            "microsoft-azure-marketplace.mediumLogo": "https://publishingapistore.blob.core.windows.net/testcontent/D6191_publishers_contoso/contosovirtualmachine/557e714b-2f31-4e12-b0cc-e48dd840edf4.png?sv=2014-02-14&sr=b&sig=NwL67NTQf9Gc9VScmZehtbHXpYmxhwZc2foy3o4xavs%3D&se=2019-03-28T19%3A46%3A49Z&sp=r",
            "microsoft-azure-marketplace.largeLogo": "https://publishingapistore.blob.core.windows.net/testcontent/D6191_publishers_contoso/contosovirtualmachine/142485da-784c-44cb-9523-d4f396446258.png?sv=2014-02-14&sr=b&sig=xaMxhwx%2FlKYfz33mJGIg8UBdVpsOwVvqhjTJ883o0iY%3D&se=2019-03-28T19%3A46%3A49Z&sp=r",
            "microsoft-azure-marketplace.wideLogo": "https://publishingapistore.blob.core.windows.net/testcontent/D6191_publishers_contoso/contosovirtualmachine/48af9013-1df7-4c94-8da8-4626e5039ce0.png?sv=2014-02-14&sr=b&sig=%2BnN7f2tprkrqb45ID6JlT01zXcy1PMTkWXtLKD6nfoE%3D&se=2019-03-28T19%3A46%3A49Z&sp=r",
            "microsoft-azure-marketplace.screenshots": [],
            "microsoft-azure-marketplace.videos": [],
            "microsoft-azure-marketplace.leadDestination": "None",
            "microsoft-azure-marketplace.tableLeadConfiguration": {},
            "microsoft-azure-marketplace.blobLeadConfiguration": {},
            "microsoft-azure-marketplace.salesForceLeadConfiguration": {},
            "microsoft-azure-marketplace.crmLeadConfiguration": {},
            "microsoft-azure-marketplace.httpsEndpointLeadConfiguration": {},
            "microsoft-azure-marketplace.marketoLeadConfiguration": {},
            "microsoft-azure-marketplace.privacyURL": "https://azuremarketplace.microsoft.com",
            "microsoft-azure-marketplace.termsOfUse": "Terms of use",
            "microsoft-azure-marketplace.engineeringContactName": "Jon Doe",
            "microsoft-azure-marketplace.engineeringContactEmail": "jondoe@outlook.com",
            "microsoft-azure-marketplace.engineeringContactPhone": "555-555-5555",
            "microsoft-azure-marketplace.supportContactName": "Jon Doe",
            "microsoft-azure-marketplace.supportContactEmail": "jondoe@outlook.com",
            "microsoft-azure-marketplace.supportContactPhone": "555-555-5555",
            "microsoft-azure-marketplace.publicAzureSupportUrl": "",
            "microsoft-azure-marketplace.fairfaxSupportUrl": ""
            },
            "plans": [
            {
                "planId": "contososkuidentifier",
                "microsoft-azure-virtualmachines.skuTitle": "Contoso App",
                "microsoft-azure-virtualmachines.skuSummary": "Contoso App makes dev ops a breeze.",
                "microsoft-azure-virtualmachines.skuDescription": "This is a description for the Contoso App that makes dev ops a breeze.",
                "microsoft-azure-virtualmachines.hideSKUForSolutionTemplate": false,
                "microsoft-azure-virtualmachines.cloudAvailability": [
                    "PublicAzure"
                ],
                "microsoft-azure-virtualmachines.certificationsFairfax": [],
                "virtualMachinePricing": {
                    "isByol": true,
                    "freeTrialDurationInMonths": 0
                },
                "microsoft-azure-virtualmachines.operatingSystemFamily": "Windows",
                "microsoft-azure-virtualmachines.windowsOSType": "Other",
                "microsoft-azure-virtualmachines.operationSystem": "Contoso App",
                "microsoft-azure-virtualmachines.recommendedVMSizes": [
                    "a0-basic",
                    "a0-standard",
                    "a1-basic",
                    "a1-standard",
                    "a2-basic",
                    "a2-standard"
                ],
                "microsoft-azure-virtualmachines.openPorts": [],
                "microsoft-azure-virtualmachines.vmImages": {
                    "1.0.1": {
                    "osVhdUrl": "http://contosoteststorage.blob.core.windows.net/test/contosoVM.vhd?sv=2014-02-14&sig=WlDo6Q4xwYH%2B5QEJbItPUVdgHhBcrVxPBmntZ2vU96w%3D&st=2016-06-25T18%3A30%3A00Z&se=2017-06-25T18%3A30%3A00Z&sp=rl",
                    "lunVhdDetails": []
                    }
                },
                "regions": [
                    "DZ",
                    "AR"
                ]
            }
            ]
        },
        "changedTime": "2017-06-07T06:15:39.7349221Z"
    }
}
```

### <a name="response-body-properties"></a>Свойства текста ответа

|  **имя**;       |   **Описание**                                                                                                               |
|  -------------  |   -----------------------------------------------------------------------------------------------------                         |
|  offerTypeId    | Определяет тип предложения                                                                                                    |
|  publisherid    | Уникальный идентификатор издателя                                                                                              |
|  status         | Состояние предложения. Список возможных значений, см. ниже в разделе [Состояние предложения](#offer-status).                                  |
|  Идентификатор             | Глобальный уникальный идентификатор уникально идентифицирующий предложение                                                                                         |
|  version        | Текущая версия предложения. Клиент не может изменить свойство версии. Оно увеличивается после каждой публикации.    |
|  Определение     | Фактическое определение рабочей нагрузки                                                                                               |
|  changedTime    | Дата и время последнего изменения предложения                                                                                   |
|  |  |

### <a name="response-status-codes"></a>Коды состояния ответа

| **Код**  | **Описание**                                                                                                                 |
|  ------   | ------------------------------------------------------------------------------------------------------------------------------- |
|  200      | `OK` — запрос успешно обработан, и все предложения в разделе издателя были возвращены клиенту.               |
|  400      | `Bad/Malformed request` — текст ответа ошибки может содержать дополнительные сведения.                                                 |
|  403      | `Forbidden` — клиент не имеет доступ к указанным пространствам имен.                                                        |
|  404      | `Not found` — указанная сущность не существует. Клиент должен проверять publisherId, offerId и версию (если указано).      |
|  |  |

### <a name="offer-status"></a>Состояние предложения

|  **имя**;                   |   **Описание**                             |
| --------------------------- |  -------------------------------------------- |
|  NeverPublished             | Предложение не было опубликовано.               |
|  NotStarted                 | Предложения — новое, но не запущенное.              |
|  WaitingForPublisherReview  | Предложение ожидает утверждения издателя.      |
|  Запущен                    | Отправка предложения обрабатывается.          |
|  Выполнено                  | Отправка предложения прекратила обработку.    |
|  Отменено                   | Отправка предложения была отменена.                |
|  Сбой                     | Не удалось отправить предложение.                      |
|  |  |
