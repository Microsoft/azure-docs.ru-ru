---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/10/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 9b4727b34bba563e9f1235771fee34273264726b
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102614264"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Служба управления API должна использовать SKU, поддерживающий виртуальные сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F73ef9241-5d81-4cd4-b483-8443d1730fe5) |Благодаря поддерживаемым SKU управления API, развертывание службы в виртуальной сети разблокирует расширенные функции сети управления API и средства безопасности, что обеспечивает больший контроль над конфигурацией безопасности сети. См. дополнительные сведения: [https://aka.ms/apimvnet](https://aka.ms/apimvnet). |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/API%20Management/ApiManagement_AllowedVNETSkus_AuditDeny.json) |
|[Службы управления API должны использовать виртуальную сеть](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fef619a2c-cc4d-4d03-b2ba-8c94a834d85b) |Развертывание виртуальной сети Azure обеспечивает повышенную безопасность, изоляцию и позволяет разместить службу "Управление API" в сети, доступом к которой будете управлять вы и которая не будет использовать маршрутизацию через Интернет. Затем эти сети можно подключить к локальным сетям с помощью различных технологий VPN, что обеспечивает доступ к внутренним службам в сети или локальной среде. Портал разработчика и шлюз API можно настроить для предоставления доступа как из Интернета, так и только в пределах виртуальной сети. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/API%20Management/ApiManagement_VNETEnabled_Audit.json) |
