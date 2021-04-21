---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 04/14/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 3e4004eac01d223ff7b5894c410b233ba6885aa6
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/14/2021
ms.locfileid: "107513100"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Весь интернет-трафик должен направляться через развернутую службу "Брандмауэр Azure"](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffc5e4038-4584-4632-8c85-c0448d374b2c) |В Центре безопасности Azure обнаружено, что некоторые подсети не защищены брандмауэром следующего поколения. Защитите подсети от возможных угроз, запретив доступ к этим сетям с помощью Брандмауэра Azure или поддерживаемого брандмауэра следующего поколения. |AuditIfNotExists, Disabled |[3.0.0 (предварительная версия)](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/ASC_All_Internet_traffic_should_be_routed_via_Azure_Firewall.json) |
|[Защита от атак DDoS Azure уровня "Стандартный" должна быть включена](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa7aca53f-2ed4-4466-a25e-0b45ade68efd) |Стандарт защиты от DDoS должен быть включен для всех виртуальных сетей с подсетью, которая является частью шлюза приложения с общедоступным IP-адресом. |AuditIfNotExists, Disabled |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableDDoSProtection_Audit.json) |
