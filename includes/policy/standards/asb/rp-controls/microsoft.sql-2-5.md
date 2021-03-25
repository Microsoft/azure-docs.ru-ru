---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/24/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 0e6aaab50b549f0bf5c971c0e05c83b84a8b379b
ms.sourcegitcommit: bb330af42e70e8419996d3cba4acff49d398b399
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105104022"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для серверов SQL Server с аудитом в качестве назначения учетной записи хранения следует настроить срок хранения в 90 дней или выше.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F89099bee-89e0-4b26-a5f4-165451757743) |При исследовании инцидентов рекомендуется задать для параметра срок хранения данных SQL Server "аудит для учетной записи хранения не менее 90 дней. Убедитесь, что вы отвечаете требованиям к хранению для регионов, в которых вы используете. Иногда это необходимо для обеспечения соответствия нормативным стандартам. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServerAuditingRetentionDays_Audit.json) |
