---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/31/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 9d4462e49cac4af564df8ff9d7962acf7512a7f6
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106095550"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Учетные записи HPC Cache должны использовать для шифрования ключ, управляемый клиентом](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F970f84d8-71b6-4091-9979-ace7e3fb6dbb) |Управление шифрованием неактивных данных рабочей HPC Cache с помощью ключей, управляемых клиентом. По умолчанию пользовательские данные шифруются с помощью ключей, управляемых службой, но для соблюдения нормативных требований обычно требуются ключи, управляемые клиентом. Ключи, управляемые клиентом, позволяют шифровать данные с помощью ключа Azure Key Vault, создателем и владельцем которого являетесь вы. Вы полностью контролируете жизненный цикл ключа, включая его смену и управление им. |Audit, Disabled, Deny |[2.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/StorageCache_CMKEnabled.json) |
