---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 04/14/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 3a9221b40f01d8b01f9b343865f4e9d0276aea9c
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/14/2021
ms.locfileid: "107499088"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Среда службы интеграции Logic Apps должна быть зашифрована с помощью ключей, управляемых клиентом](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1fafeaf6-7927-4059-a50a-8eb2a7a6f2b5) |Выполните развертывание в среде службы интеграции для управления шифрованием неактивных данных Logic Apps с помощью ключей, управляемых клиентом. По умолчанию пользовательские данные шифруются с помощью ключей, управляемых службой, но для соблюдения нормативных требований обычно требуются ключи, управляемые клиентом. Ключи, управляемые клиентом, позволяют шифровать данные с помощью ключа Azure Key Vault, создателем и владельцем которого являетесь вы. Вы полностью контролируете жизненный цикл ключа, включая его смену и управление им. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Logic%20Apps/LogicApps_ISEWithCustomerManagedKey_AuditDeny.json) |
|[Logic Apps следует развертывать в среде службы интеграции](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fdc595cb1-1cde-45f6-8faf-f88874e1c0e1) |Развертывание Logic Apps в среде службы интеграции в виртуальной сети включает расширенные возможности сетевого взаимодействия и защиты Logic Apps, а также обеспечивает больший контроль над конфигурацией сети. См. дополнительные сведения: [https://aka.ms/integration-service-environment](https://aka.ms/integration-service-environment). Развертывание в среде службы интеграции также позволяет выполнять шифрование с помощью ключей, управляемых клиентом, обеспечивая усиленную защиту данных благодаря возможности управления ключами шифрования. Это часто необходимо для удовлетворения требований по обеспечению соответствия. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Logic%20Apps/LogicApps_LogicAppsInISE_AuditDeny.json) |
|[В Logic Apps должны быть включены журналы ресурсов](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F34f95f76-5386-4de7-b824-0d8478470c9d) |Выполняйте аудит для включения журналов ресурсов. Это позволит воссоздать действия для расследования в случае инцидентов безопасности или компрометации сети. |AuditIfNotExists, Disabled |[4.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Logic%20Apps/LogicApps_AuditDiagnosticLog_Audit.json) |
