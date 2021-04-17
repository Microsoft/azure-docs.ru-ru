---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/31/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: f3d63180219caa99fed22a5f8249de7e8f1a3eea
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106098010"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[В хранилищах ключей должна быть включена защита от очистки](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0b60c0b2-2dc2-4e1c-b5c9-abbed971de53) |Удаление хранилища ключей злоумышленником может привести к необратимой потере данных. Потенциальный злоумышленник в вашей организации может удалить и очистить хранилища ключей. Защита от очистки позволяет оградить вас от атак злоумышленников. Для этого применяется обязательный период хранения данных для хранилищ ключей, которые были обратимо удалены. Ни корпорация Майкрософт, ни пользователи вашей организации не смогут очистить хранилища ключей во время периода хранения при обратимом удалении. |Audit, Deny, Disabled |[1.1.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Key%20Vault/KeyVault_Recoverable_Audit.json) |
