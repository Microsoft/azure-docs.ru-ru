---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/10/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 293a32a52419d281409dcdfc7b234d8cdb5e6942
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102616673"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Служба Azure SignalR должна использовать приватный канал](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F53503636-bcc9-4748-9663-5348217f160f) |Приватный канал Azure позволяет подключить виртуальную сеть к службам Azure без общедоступного IP-адреса в исходном или целевом расположении. Платформа Приватного канала поддерживает подключение между потребителем и службами через магистральную сеть Azure. Сопоставление частных конечных точек с ресурсами SignalR вместо всей службы обеспечивает защиту от утечки данных. См. дополнительные сведения: [https://aka.ms/asrs/privatelink](https://aka.ms/asrs/privatelink). |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SignalR/SignalR_PrivateEndpointEnabled_Audit.json) |
