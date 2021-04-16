---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/31/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 4a86ac3ddd639bac7a04d9f2d3bddd40a0774ca2
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106091231"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Шаблоны Конструктора образов виртуальных машин должны использовать приватные каналы](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F2154edb9-244f-4741-9970-660785bccdaa) |Приватный канал Azure позволяет подключить виртуальную сеть к службам Azure без общедоступного IP-адреса в исходном или целевом расположении. Платформа Приватного канала поддерживает подключение между потребителем и службами через магистральную сеть Azure. За счет сопоставления частных конечных точек с ресурсами Конструктора образов виртуальных машин снижается риск утечки данных. Дополнительные сведения о приватных каналах см. по адресу [https://docs.microsoft.com/azure/virtual-machines/linux/image-builder-networking#deploy-using-an-existing-vnet](https://docs.microsoft.com/azure/virtual-machines/linux/image-builder-networking#deploy-using-an-existing-vnet). |Audit, Disabled, Deny |[1.1.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/VM%20Image%20Builder/PrivateLinkEnabled_Audit.json) |
