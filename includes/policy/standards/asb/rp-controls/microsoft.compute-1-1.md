---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/31/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 7345c6078767b0fa807c546d6bcec42a10cb5573
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "107285046"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[На виртуальных машинах с выходом в Интернет должны применяться рекомендации по адаптивной защите сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F08e6af2d-db70-460a-bfe9-d5bd474ba9d6) |Центр безопасности Azure анализирует шаблоны трафика виртуальных машин, доступных через Интернет, и предоставляет рекомендации по правилам группы безопасности сети, которые уменьшают потенциальную область атаки. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_AdaptiveNetworkHardenings_Audit.json) |
|[Виртуальные машины с выходом в Интернет должны быть защищены с помощью групп безопасности сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff6de0be7-9a8a-4b8a-b349-43cf02d22f7c) |Защитите виртуальные машины от потенциальных угроз, ограничив доступ к ним с помощью групп безопасности сети (NSG). Дополнительные сведения об управлении трафиком с помощью групп безопасности сети: [https://aka.ms/nsg-doc](https://aka.ms/nsg-doc). |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_NetworkSecurityGroupsOnInternetFacingVirtualMachines_Audit.json) |
|[На виртуальной машине должна быть отключена IP-переадресация](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fbd352bd5-2853-4985-bf0d-73806b4a5744) |Включение IP-переадресации на сетевом адаптере виртуальной машины позволяет компьютеру принимать трафик, адресованный другим получателям. IP-переадресация редко требуется (например, при использовании виртуальной машины в качестве сетевого виртуального модуля), поэтому она должна быть проверена командой безопасности сети. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_IPForwardingOnVirtualMachines_Audit.json) |
|[Порты управления виртуальных машин должны быть защищены с помощью JIT-управления доступом к сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb0f33259-77d7-4c9e-aac6-3aabcfae693c) |Возможный JIT-доступ к сети будет отслеживаться центром безопасности Azure для предоставления рекомендаций. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_JITNetworkAccess_Audit.json) |
|[Порты управления на виртуальных машинах должны быть закрыты](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22730e10-96f6-4aac-ad84-9383d35b5917) |Открытые порты удаленного управления создают для вашей виртуальной машины высокий уровень риска атак из Интернета. Эти атаки используют метод подбора учетных данных для получения доступа к компьютеру от имени администратора. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_OpenManagementPortsOnVirtualMachines_Audit.json) |
