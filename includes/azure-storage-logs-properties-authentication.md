---
author: normesta
ms.service: storage
ms.topic: include
ms.date: 09/28/2020
ms.author: normesta
ms.openlocfilehash: ca8963ed8928745a6d5918c86021199432339c83
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104612164"
---
| Свойство | Описание |
|:--- |:---|
|**identity/type** | Тип аутентификации, используемый для выполнения запроса. Например: `OAuth` ,, `Kerberos` `SAS Key` , `Account Key` или `Anonymous` |
|**identity/tokenHash**|Хэш SHA-256 токена проверки подлинности, используемый в запросе. <br>Если тип проверки подлинности — `Account Key` , то используется формат "key1 \| key2 (хэш SHA256 ключа)". Например, `key1(5RTE343A6FEB12342672AFD40072B70D4A91BGH5CDF797EC56BF82B2C3635CE)`. <br>Если тип проверки подлинности — `SAS Key` , то используется формат "key1 \| key2 (хэш SHA 256), сассигнатуре (хэш SHA 256 маркера SAS)". Например, `key1(0A0XE8AADA354H19722ED12342443F0DC8FAF3E6GF8C8AD805DE6D563E0E5F8A),SasSignature(04D64C2B3A704145C9F1664F201123467A74D72DA72751A9137DDAA732FA03CF)`. Если тип проверки подлинности — `OAuth` , то используется формат "sha 256 hash маркера OAuth". Пример: `B3CC9D5C64B3351573D806751312317FE4E910877E7CBAFA9D95E0BE923DW25C`<br> Для других типов проверки подлинности отсутствует поле tokenHash. |
|**authorization/action** | Действие, назначенное запросу. |
|**authorization/roleAssignmentId** | Идентификатор назначения ролей. Например: `4e2521b7-13be-4363-aeda-111111111111`.|
|**authorization/roleDefinitionId** | Идентификатор определения роли. Например: `ba92f5b4-2d11-453d-a403-111111111111"`.|
|**principals/id** | Идентификатор субъекта безопасности. Например: `a4711f3a-254f-4cfb-8a2d-111111111111`.|
|**principals/type** | Тип субъекта безопасности. Например: `ServicePrincipal`. |
|**requester/appID** | Идентификатор приложения Open Authorization (OAuth), который используется в качестве инициатора запроса. <br> Например: `d3f7d5fe-e64a-4e4e-871d-333333333333`.|
|**requester/audience** | Аудитория OAuth запроса. Например: `https://storage.azure.com`. |
|**requester/objectId** | Идентификатор объекта OAuth инициатора запроса. В случае аутентификации Kerberos предоставляется идентификатор объекта пользователя, который ее прошел. Например: `0e0bf547-55e5-465c-91b7-2873712b249c`. |
|**requester/tenantId** | Идентификатор клиента OAuth удостоверения. Например: `72f988bf-86f1-41af-91ab-222222222222`.|
|**requester/tokenIssuer** | Издатель токенов OAuth. Например: `https://sts.windows.net/72f988bf-86f1-41af-91ab-222222222222/`.|
|**requester/upn** | Имя участника-пользователя (UPN) инициатора запроса. Например: `someone@contoso.com`. |
|**requester/userName** | Это поле зарезервировано для внутреннего использования.|
