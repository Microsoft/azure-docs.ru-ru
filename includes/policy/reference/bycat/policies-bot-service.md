---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 04/14/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 18a4ad144870352c22ca7460786fdf3dbcaabeda
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/14/2021
ms.locfileid: "107498942"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Конечная точка Службы Bot должна быть допустимым URI HTTPS](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6164527b-e1ee-4882-8673-572f425f5e0a) |Данные могут быть изменены во время передачи. Существуют протоколы, которые обеспечивают шифрование для избежания проблем неправильного использования и незаконного изменения. Чтобы обеспечить взаимодействие ботов только по зашифрованным каналам, задайте в качестве конечной точки допустимый URI HTTPS. Это гарантирует, что протокол HTTPS используется для шифрования передаваемых данных, и часто является требованием для обеспечения соответствия нормативным или отраслевым стандартам. Перейдите по адресу [https://docs.microsoft.com/azure/bot-service/bot-builder-security-guidelines](https://docs.microsoft.com/azure/bot-service/bot-builder-security-guidelines). |Audit, Deny, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Bot%20Service/BotService_ValidEndpoint_Audit.json) |
|[Служба Bot должна шифроваться с помощью ключа, управляемого клиентом](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F51522a96-0869-4791-82f3-981000c2c67f) |Служба Azure Bot автоматически шифрует ваш ресурс для обеспечения защиты данных и соблюдения требований к безопасности и соответствию в организации. По умолчанию используются ключи шифрования, управляемые корпорацией Майкрософт. Для большей гибкости управления ключами или управления доступом к подписке выберите ключи, управляемые клиентом, которые также называются собственными ключами (BYOK). Подробнее о шифровании в службе Azure Bot: [https://docs.microsoft.com/azure/bot-service/bot-service-encryption](https://docs.microsoft.com/azure/bot-service/bot-service-encryption). |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Bot%20Service/BotService_CMKEnabled_Audit.json) |
