---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 02/04/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 366f617a30417ab19568d374a2b04d674313fbf7
ms.sourcegitcommit: f82e290076298b25a85e979a101753f9f16b720c
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/04/2021
ms.locfileid: "99556304"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Определение приложения для управляемого приложения должно использовать учетную запись хранения, предоставленную клиентом](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F9db7917b-1607-4e7d-a689-bca978dd0633) |Используйте собственную учетную запись хранения для управления данными определения приложения, если это нормативное требование или требование к обеспечению соответствия. Вы можете сохранить определение управляемого приложения в учетной записи хранения, которую вы указали во время создания. Это позволит полностью контролировать расположение приложения и доступ к нему в соответствии с требованиями к обеспечению соответствия. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Managed%20Application/ApplicationDefinition_Missing_StorageAccount_Deny.json) |
|[Развертывание связываний для управляемого приложения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F17763ad9-70c0-4794-9397-53d765932634) |Развертывает ресурс для связывания выбранных типов ресурсов с указанным управляемым приложением.  Эта политика не поддерживает вложенные типы ресурсов. |deployIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Managed%20Application/AssociationForManagedApplication_Deploy.json) |
