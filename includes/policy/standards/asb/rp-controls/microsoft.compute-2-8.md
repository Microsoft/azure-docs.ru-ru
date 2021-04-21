---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 04/14/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 14ae156cdcf24005f958c17c9b28e7b50d9be2ed
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/14/2021
ms.locfileid: "107512935"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В масштабируемых наборах виртуальных машин должно быть установлено решение Endpoint Protection](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F26a828e1-e88f-464e-bbb3-c134a282b9de) |Аудит наличия и работоспособности решения защиты конечных точек в ваших масштабируемых наборах виртуальных машин для защиты их от угроз и уязвимостей. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_VmssMissingEndpointProtection_Audit.json) |
|[В Microsoft Antimalware для Azure должно быть настроено автоматическое обновление подписей защиты](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc43e4a30-77cb-48ab-a4dd-93f175c63b57) |Эта политика выполняет аудит всех виртуальных машин Windows, на которых не настроено автоматическое обновление подписей защиты Microsoft Antimalware. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/VirtualMachines_AntiMalwareAutoUpdate_AuditIfNotExists.json) |
|[Мониторинг отсутствия Endpoint Protection в Центре безопасности Azure](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Faf6cd1bd-1635-48cb-bde7-5b15693900b9) |Серверы без установленного агента Endpoint Protection будут отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_MissingEndpointProtection_Audit.json) |
