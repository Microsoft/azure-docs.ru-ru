---
title: Пример политики управления API Azure. Использование OAuth2 для авторизации между шлюзом и серверной частью
titleSuffix: Azure API Management
description: Пример политики службы управления API Azure. Использование OAuth2 для авторизации между шлюзом и серверной частью. В статье показано, как получить маркер доступа из AAD и перенаправить его в серверную часть.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 10/13/2017
ms.author: apimpm
ms.openlocfilehash: 0f5a10eb2ebc3b3a7c527dd718e37faf03c927bc
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92076102"
---
# <a name="use-oauth2-for-authorization-between-the-gateway-and-a-backend"></a>Использование OAuth2 для авторизации между шлюзом и серверной частью

В этой статье на примере политики службы управления API Azure показано, как использовать OAuth2 для авторизации между шлюзом и серверной частью. В статье показано, как получить маркер доступа из AAD и перенаправить его в серверную часть. 

См. дополнительные сведения о [создании и изменении кода политик](../set-edit-policies.md). Также для ознакомления доступны другие [примеры политик](../policy-reference.md).

Следующий сценарий использует свойства, которые отображаются в {{property}}. Чтобы узнать о свойствах и их использовании в политиках управления API, ознакомьтесь с [этим](../api-management-howto-properties.md) разделом.
 
## <a name="policy"></a>Политика

Вставьте код в блок **inbound**.

[!code-xml[Main](../../../api-management-policy-samples/examples/Get OAuth2 access token from AAD and forward it to the backend.policy.xml)]
  
## <a name="next-steps"></a>Дальнейшие действия

Подробнее о политиках службы управления API:

+ [Политики преобразования](../api-management-transformation-policies.md)
+ [Примеры политик](../policy-reference.md).