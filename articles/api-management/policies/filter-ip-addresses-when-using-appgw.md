---
title: Пример политики управления API — фильтрация по IP-адресу при использовании шлюза приложений
titleSuffix: Azure API Management
description: Пример политики службы управления API Azure. здесь показано, как выполнить фильтрацию по IP-адресу запроса при использовании шлюза приложений.
services: api-management
documentationcenter: ''
author: jftl6y
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 01/13/2020
ms.author: joscot
ms.custom: fasttrack-new
ms.openlocfilehash: 8249cc543c6334841c8e5152d5d1ceb84d4097dc
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92076119"
---
# <a name="filter-on-request-ip-address-when-using-an-application-gateway"></a>Фильтровать по IP-адресу запроса при использовании шлюза приложений

В этой статье показан пример политики службы управления API Azure, демонстрирующий фильтрацию по IP-адресу запроса при доступе к экземпляру управления API через шлюз приложений или другую промежуточную подпрограмму. См. дополнительные сведения о [создании и изменении кода политик](../set-edit-policies.md). Также для ознакомления доступны другие [примеры политик](../policy-reference.md).

## <a name="policy"></a>Политика

Вставьте код в блок **inbound**.

[!code-xml[Main](../../../api-management-policy-samples/examples/Filter on IP Address when using Application Gateway.policy.xml)]

## <a name="next-steps"></a>Дальнейшие действия

Подробнее о политиках службы управления API:

+ [Политики ограничения доступа в службе управления API](../api-management-access-restriction-policies.md)
+ [Примеры политик](../policy-reference.md).