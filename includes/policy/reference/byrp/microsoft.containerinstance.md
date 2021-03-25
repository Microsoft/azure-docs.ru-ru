---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/24/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 116b1ca47c9fa7c5f2448499c14b6da56a8b02c9
ms.sourcegitcommit: bb330af42e70e8419996d3cba4acff49d398b399
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105035356"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Группа контейнеров экземпляров контейнеров Azure должна быть развернута в виртуальной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F8af8f826-edcb-4178-b35f-851ea6fea615) |Обеспечение безопасного обмена данными между контейнерами и виртуальными сетями Azure. При указании виртуальной сети ресурсы в виртуальной сети могут безопасно и частным образом обмениваться данными. |Audit, Disabled, Deny |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Container%20Instance/ContainerInstance_VNET_Audit.json) |
|[Группа контейнеров экземпляров контейнеров Azure должна использовать для шифрования ключ, управляемый клиентом](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0aa61e00-0a01-4a3c-9945-e93cffedf0e6) |Защита контейнеров благодаря увеличению гибкости с помощью ключей, управляемых клиентом. Указанный ключ CMK используется для защиты доступа к ключу, который шифрует данные, и управления этим доступом. Использование управляемых клиентом ключей предоставляет дополнительные возможности для управления сменой ключей шифрования или криптографического стирания данных. |Audit, Disabled, Deny |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Container%20Instance/ContainerInstance_CMK_Audit.json) |
