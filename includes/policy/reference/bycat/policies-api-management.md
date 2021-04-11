---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/31/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: ca68c0a770023d0362cb9278b3742a6a136c796b
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106090887"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Служба Управления API должна использовать SKU с поддержкой виртуальных сетей](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F73ef9241-5d81-4cd4-b483-8443d1730fe5) |Если для Управления API используются поддерживаемые SKU, при развертывании службы в виртуальной сети вы получаете расширенные функции управления сетью и ее конфигурацией безопасности. См. дополнительные сведения: [https://aka.ms/apimvnet](https://aka.ms/apimvnet). |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/API%20Management/ApiManagement_AllowedVNETSkus_AuditDeny.json) |
|[Службы управления API должны использовать виртуальную сеть](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fef619a2c-cc4d-4d03-b2ba-8c94a834d85b) |Развертывание виртуальной сети Azure обеспечивает повышенную безопасность, изоляцию и позволяет разместить службу "Управление API" в сети, доступом к которой будете управлять вы и которая не будет использовать маршрутизацию через Интернет. Затем эти сети можно подключить к локальным сетям с помощью различных технологий VPN, что обеспечивает доступ к внутренним службам в сети или локальной среде. Портал разработчика и шлюз API можно настроить для предоставления доступа как из Интернета, так и только в пределах виртуальной сети. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/API%20Management/ApiManagement_VNETEnabled_Audit.json) |
