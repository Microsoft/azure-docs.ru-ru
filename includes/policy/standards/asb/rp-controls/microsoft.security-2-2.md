---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/31/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 355a74ded8b8d01632aff70b35f96f2502c970b7
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "107285864"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В подписке должна быть включена автоматическая подготовка агента Log Analytics](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F475aae12-b88a-4572-8b36-9b712b2b3a17) |Чтобы отслеживать уязвимости и угрозы безопасности, Центр безопасности Azure собирает данные из виртуальных машин Azure. Для сбора данных используется агент Log Analytics (ранее — Microsoft Monitoring Agent, или MMA), который считывает разные конфигурации, связанные с безопасностью, и журналы событий с компьютера, а также копирует данные в рабочую область Log Analytics для анализа. Мы рекомендуем включить автоматическую подготовку для автоматического развертывания агента на всех поддерживаемых виртуальных машинах Azure, включая те, которые созданы недавно. |AuditIfNotExists, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_Automatic_provisioning_log_analytics_monitoring_agent.json) |
