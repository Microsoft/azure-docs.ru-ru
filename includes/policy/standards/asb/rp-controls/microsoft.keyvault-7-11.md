---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 04/14/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 8a6d26eb0fb795f0ca047ff4a976a5b31b9493ee
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/14/2021
ms.locfileid: "107511499"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В хранилищах ключей должна быть включена защита от очистки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0b60c0b2-2dc2-4e1c-b5c9-abbed971de53) |Удаление хранилища ключей злоумышленником может привести к необратимой потере данных. Потенциальный злоумышленник в вашей организации может удалить и очистить хранилища ключей. Защита от очистки позволяет оградить вас от атак злоумышленников. Для этого применяется обязательный период хранения данных для хранилищ ключей, которые были обратимо удалены. Ни корпорация Майкрософт, ни пользователи вашей организации не смогут очистить хранилища ключей во время периода хранения при обратимом удалении. |Audit, Deny, Disabled |[1.1.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Key%20Vault/KeyVault_Recoverable_Audit.json) |
