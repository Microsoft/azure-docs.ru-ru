---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/05/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: a4973f9ee064c86fc0f133f9c33e3a18588a86ff
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102435536"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Развертывание параметров диагностики Data Lake Storage 1-го поколения в концентраторе событий](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe8d096bc-85de-4c5f-8cfb-857bd1b9d62d) |Развертывает параметры диагностики Data Lake Storage 1-го поколения для потоковой передачи в региональный концентратор событий при создании или изменении службы Data Lake Storage 1-го поколения, где эти параметры диагностики отсутствуют. |DeployIfNotExists, Disabled |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/DataLakeStorage_DeployDiagnosticLog_Deploy_EventHub.json) |
|[Развертывание параметров диагностики Data Lake Storage 1-го поколения в рабочей области Log Analytics](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F25763a0a-5783-4f14-969e-79d4933eb74b) |Развертывает параметры диагностики Data Lake Storage 1-го поколения для потоковой передачи в региональную рабочую область Log Analytics при создании или изменении службы Data Lake Storage 1-го поколения, где эти параметры диагностики отсутствуют. |DeployIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/DataLakeStorage_DeployDiagnosticLog_Deploy_LogAnalytics.json) |
|[Требовать шифрование учетных записей Data Lake Store](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa7ff3161-0087-490a-9ad9-ad6217f4f43a) |Эта политика активирует шифрование во всех учетных записях Data Lake Storage |deny |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Data%20Lake/DataLakeStoreEncryption_Deny.json) |
|[В Azure Data Lake Storage должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F057ef27e-665e-4328-8ea3-04b3122bd9fb) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для расследования в случае инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Data%20Lake/DataLakeStore_AuditDiagnosticLog_Audit.json) |
