---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/10/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 581b0f6aeae99a16935716e470a0eeec0bbd74e0
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "103422193"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Группа контейнеров экземпляров контейнеров Azure должна быть развернута в виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8af8f826-edcb-4178-b35f-851ea6fea615) |Обеспечьте безопасность обмена данными между контейнерами и виртуальными сетями Azure. При указании виртуальной сети ресурсы в виртуальной сети могут безопасно и частным образом взаимодействовать друг с другом. |Audit, Disabled, Deny |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Container%20Instance/ContainerInstance_VNET_Audit.json) |
|[Группа контейнеров экземпляров контейнеров Azure должна использовать ключ, управляемый клиентом, для шифрования](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0aa61e00-0a01-4a3c-9945-e93cffedf0e6) |Защитите свои контейнеры с большей гибкостью с помощью управляемых клиентом ключей. Указанный ключ CMK используется для защиты доступа к ключу, который шифрует данные, и управления этим доступом. Использование управляемых клиентом ключей предоставляет дополнительные возможности для управления сменой ключей шифрования или криптографического стирания данных. |Audit, Disabled, Deny |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Container%20Instance/ContainerInstance_CMK_Audit.json) |
