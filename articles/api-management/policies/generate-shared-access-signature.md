---
title: Пример политики управления API — Создание подписи общего доступа
titleSuffix: Azure API Management
description: Пример политики службы управления API Azure. Создание подписанного URL-адреса с помощью выражений и перенаправление запроса в службу хранилища Azure с использованием политики rewrite-uri.
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
ms.openlocfilehash: f90c1e492d6da4f85fe86e4c5c76b89bdfaa4797
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92078431"
---
# <a name="generate-shared-access-signature"></a>Создание подписанного URL-адреса

В этой статье на примере политики службы управления API Azure показано, как создать [подписанный URL-адрес](../../storage/common/storage-sas-overview.md) с помощью выражений и перенаправить запрос в службу хранилища Azure с использованием политики rewrite-uri. См. дополнительные сведения о [создании и изменении кода политик](../set-edit-policies.md). Также для ознакомления доступны другие [примеры политик](../policy-reference.md).

## <a name="policy"></a>Политика

Вставьте код в блок **inbound**.

[!code-xml[Main](../../../api-management-policy-samples/examples/Generate Shared Access Signature and forward request to Azure storage.policy.xml)]

## <a name="next-steps"></a>Дальнейшие действия

Подробнее о политиках службы управления API:

+ [Политики преобразования](../api-management-transformation-policies.md)
+ [Примеры политик](../policy-reference.md).