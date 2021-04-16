---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/31/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 289a2fe7fecc3e28d646e98692c1d5ada4ea7fa8
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106097744"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для серверов SQL Server с аудитом в назначении учетной записи хранения следует настроить срок хранения длительностью 90 дней или более](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F89099bee-89e0-4b26-a5f4-165451757743) |Для исследования инцидентов мы рекомендуем задать срок хранения данных аудита SQL Server в назначении учетной записи хранения длительностью как минимум 90 дней. Убедитесь, что соблюдаются необходимые правила хранения для регионов, в которых вы работаете. Иногда это необходимо для обеспечения соответствия нормативным стандартам. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServerAuditingRetentionDays_Audit.json) |
