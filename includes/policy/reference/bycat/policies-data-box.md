---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/10/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 194b4ad507a48f1010dc884326c16b95d6513081
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102608673"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В заданиях Azure Data Box должно быть включено двойное шифрование для неактивных данных на устройстве](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc349d81b-9985-44ae-a8da-ff98d108ede8) |Включите второй уровень программного шифрования для неактивных данных на устройстве. Устройство уже защищено с помощью 256-разрядного шифрования AES для неактивных данных. Этот параметр добавляет второй уровень шифрования данных. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Data%20Box/DataBox_DoubleEncryption_Audit.json) |
|[В заданиях Azure Data Box для шифрования пароля разблокировки устройства должен использоваться ключ, управляемый клиентом](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F86efb160-8de7-451d-bc08-5d475b0aadae) |Для управления шифрованием пароля разблокировки устройства для Azure Data Box следует использовать ключ, управляемый клиентом. Ключи, управляемые клиентом, также позволяют управлять доступом службы Data Box к паролю разблокировки устройства, чтобы подготовить устройство и автоматически копировать данные. Данные на самом устройстве шифруются при хранении с помощью 256-разрядного шифрования AES, а пароль разблокировки устройства шифруется по умолчанию с помощью ключа, управляемого Майкрософт. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Data%20Box/DataBox_CMK_Audit.json) |
