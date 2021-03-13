---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/10/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: ce13484aa085cef89ba34151824ee843dd3bd90e
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "103419902"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Синхронизация файлов Azure должны использовать закрытую ссылку](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1d320205-c6a1-4ac6-873d-46224024e8e2) |Создание частной конечной точки для указанного ресурса службы синхронизации хранилища позволяет вам обращаться к ресурсу службы синхронизации хранилища из пространства частных IP-адресов в сети вашей организации, а не через общедоступную конечную точку, доступную через Интернет. При создании частной конечной точки сама сторона не отключает общедоступную конечную точку. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/StorageSync_PrivateEndpoint_AuditIfNotExists.json) |
|[Настройка Синхронизация файлов Azure с частными конечными точками](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fb35dddd9-daf7-423b-8375-5a5b86806d5a) |Для указанного ресурса службы синхронизации хранилища развертывается частная конечная точка. Это позволяет обращаться к ресурсу службы синхронизации хранилища из пространства частных IP-адресов в сети вашей организации, а не через общедоступную конечную точку, доступную через Интернет. Существование одной или нескольких частных конечных точек само по себе не отключает общедоступную конечную точку. |DeployIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/StorageSync_PrivateEndpoint_DeployIfNotExists.json) |
|[Изменить — настроить Синхронизация файлов Azure для отключения доступа к общедоступной сети](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0e07b2e9-6cd9-4c40-9ccb-52817b95133b) |Общедоступная конечная точка с доступом к Интернету для Синхронизация файлов Azure отключена политикой Организации. Вы по-прежнему можете получить доступ к службе синхронизации хранилища через ее частные конечные точки. |Изменение, отключено |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/StorageSync_IncomingTrafficPolicy_Modify.json) |
|[Доступ к общедоступной сети должен быть отключен для Синхронизация файлов Azure](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F21a8cd35-125e-4d13-b82d-2e19b7208bb7) |Отключение общедоступной конечной точки позволяет ограничить доступ к ресурсу службы синхронизации хранилища с запросами, предназначенными для утвержденных частных конечных точек в сети Организации. Тем не менее, нет ничего безопасного, чтобы разрешить запросы к общедоступной конечной точке, однако вы можете отключить ее, чтобы удовлетворить нормативные требования, юридические условия или политики Организации. Вы можете отключить общедоступную конечную точку для службы синхронизации хранилища, задав для Инкомингтраффикполици ресурса значение Алловвиртуалнетворксонли. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Storage/StorageSync_IncomingTrafficPolicy_AuditDeny.json) |
