---
title: Пример политики службы управления API Azure. Реализация шаблона X-CSRF | Документация Майкрософт
description: Пример политики службы управления API Azure. Реализация шаблона X-CSRF, который используется во многих API. Этот пример относится только к шлюзу SAP.
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
ms.openlocfilehash: bc6f60397c2d432dd6547a8be34280fcf3acd05d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92078414"
---
# <a name="implement-x-csrf-pattern"></a>Реализация шаблона X-CSRF

В этой статье на примере политики службы управления API Azure показано, как реализовать шаблон X-CSRF, который используется во многих интерфейсах API. Этот пример относится только к шлюзу SAP. См. дополнительные сведения о [создании и изменении кода политик](../set-edit-policies.md). Также для ознакомления доступны другие [примеры политик](../policy-reference.md).

## <a name="policy"></a>Политика

Вставьте код в блок **inbound**.

[!code-xml[Main](../../../api-management-policy-samples/examples/Get X-CSRF token from SAP gateway using send request.policy.xml)]

## <a name="next-steps"></a>Дальнейшие действия

Подробнее о политиках службы управления API:

+ [Политики преобразования](../api-management-transformation-policies.md)
+ [Примеры политик](../policy-reference.md).