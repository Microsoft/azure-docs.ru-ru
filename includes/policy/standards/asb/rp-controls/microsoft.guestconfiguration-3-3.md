---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/17/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: ffe2dc8ac0290ffce8d7ca816a72ea89edd16ca9
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104612974"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Аудит компьютеров с Windows без любого из указанных участников в группе администраторов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F30f71ea1-ac77-4f26-9fc5-2d926bbd4ba7) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры считаются несоответствующими, если в локальной группе администраторов отсутствует один или несколько участников, указанных в параметре политики. |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_AdministratorsGroupMembersToInclude_AINE.json) |
|[Аудит компьютеров Windows с дополнительными учетными записями в группе администраторов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3d2a3320-2a72-4c67-ac5f-caa40fbee2b2) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры считаются несоответствующими, если локальная группа администраторов содержит участников, не указанных в параметре политики. |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_AdministratorsGroupMembers_AINE.json) |
|[Аудит компьютеров Windows с указанными участниками в группе администраторов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F69bf4abd-ca1e-4cf6-8b5a-762d42e61d4f) |Требует развертывания необходимых компонентов в области назначения политики. Дополнительные сведения см. на сайте [https://aka.ms/gcpol](https://aka.ms/gcpol). Компьютеры считаются несоответствующими, если локальная группа администраторов содержит одного или нескольких участников, указанных в параметре политики. |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Guest%20Configuration/GuestConfiguration_AdministratorsGroupMembersToExclude_AINE.json) |
