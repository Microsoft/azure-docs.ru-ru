---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 04/14/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: edca89c21180f92620028cd0b23e026e96e4924d
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/14/2021
ms.locfileid: "107511251"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Учетные записи хранения должны ограничивать доступ к сети.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F34c877ad-507e-4c82-993e-3452a6e0ad3c) |Сетевой доступ к учетным записям хранения должен быть ограниченным. Настройте правила сети так, чтобы учетная запись хранения была доступна только приложениям из разрешенных сетей. Чтобы разрешить подключения от конкретных локальных клиентов и интернет-клиентов, вы можете открыть доступ для трафика из конкретных виртуальных сетей Azure или определенных диапазонов общедоступных IP-адресов. |Audit, Deny, Disabled |[1.1.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/Storage_NetworkAcls_Audit.json) |
|[Учетная запись хранения должна использовать конечную точку службы виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F60d21c4f-21a3-4d94-85f4-b924e6aeeda4) |Эта политика выполняет аудит всех учетных записей хранения, не настроенных на использование конечной точки службы виртуальной сети. |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_StorageAccount_Audit.json) |
