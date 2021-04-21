---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 04/14/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 0478e606cd5eb6652b992b647d5f69b347753341
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/14/2021
ms.locfileid: "107504311"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Синхронизация файлов Azure должна использовать приватный канал](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1d320205-c6a1-4ac6-873d-46224024e8e2) |Создав частную конечную точку для указанного ресурса службы синхронизации хранилища, можно обращаться к этому ресурсу из пространства частных IP-адресов в сети организации, а не через общедоступную конечную точку в Интернете. Создание частной конечной точки само по себе не приводит к отключению общедоступной конечной точки. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/StorageSync_PrivateEndpoint_AuditIfNotExists.json) |
|[Настройка частных конечных точек для Синхронизации файлов Azure](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb35dddd9-daf7-423b-8375-5a5b86806d5a) |Частная конечная точка развертывается для указанного ресурса службы синхронизации хранилища. Это позволяет обращаться к ресурсу службы синхронизации хранилища из пространства частных IP-адресов в сети организации, а не через общедоступную конечную точку в Интернете. Существование одной или нескольких частных конечных точек само по себе не приводит к отключению общедоступной конечной точки. |DeployIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/StorageSync_PrivateEndpoint_DeployIfNotExists.json) |
|[Изменение — настройка Синхронизации файлов Azure для отключения доступа к общедоступной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0e07b2e9-6cd9-4c40-9ccb-52817b95133b) |Общедоступная конечная точка в Интернете для Синхронизации файлов Azure отключена политикой организации. Вы по-прежнему можете получить доступ к службе синхронизации хранилища через ее частные конечные точки. |Modify, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/StorageSync_IncomingTrafficPolicy_Modify.json) |
|[Доступ к общедоступной сети для Синхронизации файлов Azure должен быть отключен](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F21a8cd35-125e-4d13-b82d-2e19b7208bb7) |Отключение общедоступной конечной точки позволяет ограничить доступ к ресурсу службы синхронизации хранилища запросами, предназначенными для утвержденных частных конечных точек в сети организации. Нет ничего принципиально опасного в разрешении запросов к общедоступной конечной точке, но вы можете отключить эту функцию, чтобы обеспечить соблюдение нормативных, юридических или организационных требований к политикам. Вы можете отключить общедоступную конечную точку для службы синхронизации хранилища, задав для incomingTrafficPolicy ресурса значение AllowVirtualNetworksOnly. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/StorageSync_IncomingTrafficPolicy_AuditDeny.json) |
