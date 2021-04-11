---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/31/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 3d99dcee9030cc482604d60cdfed795c187edccc
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106090935"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Во время шифрования неактивных данных в Azure Data Explorer должен использоваться управляемый клиентом ключ](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F81e74cea-30fd-40d5-802f-d72103c2aaaa) |Включение шифрования неактивных данных с помощью управляемого клиентом ключа в кластере Azure Data Explorer обеспечивает дополнительный контроль над ключом, используемым для шифрования неактивных данных. Эта функция часто применяется для клиентов с особыми нормативными требованиями. Для управления ключами требуется Key Vault. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Azure%20Data%20Explorer/ADX_CMK.json) |
|[В Azure Data Explorer должно быть включено шифрование дисков](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff4b53539-8df9-40e4-86c6-6b607703bd4e) |Включение шифрования дисков Azure помогает защитить данные согласно корпоративным обязательствам по обеспечению безопасности и соответствия. |Audit, Deny, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Azure%20Data%20Explorer/ADX_disk_encrypted.json) |
|[В Azure Data Explorer должно быть включено двойное шифрование](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fec068d99-e9c7-401f-8cef-5bdde4e6ccf1) |Включение двойного шифрования помогает защитить данные согласно корпоративным обязательствам по обеспечению безопасности и соответствия. Если включено двойное шифрование, данные в учетной записи хранения шифруются дважды: один раз на уровне службы и один раз на уровне инфраструктуры с использованием двух разных алгоритмов шифрования и двух разных ключей. |Audit, Deny, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Azure%20Data%20Explorer/ADX_doubleEncryption.json) |
|[Для Azure Data Explorer должно быть включено внедрение в виртуальную сеть](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9ad2fd1f-b25f-47a2-aa01-1a5a779e6413) |Обеспечьте безопасность периметра сети посредством внедрения в виртуальную сеть. Это позволит применять правила группы безопасности сети, подключать локальные ресурсы и защищать источники подключения к данным, используя конечные точки служб. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Azure%20Data%20Explorer/ADX_VNET_configured.json) |
