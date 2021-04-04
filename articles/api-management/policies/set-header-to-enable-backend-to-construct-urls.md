---
title: Пример политики службы управления API Azure. Добавление заголовка перенаправления | Документация Майкрософт
description: Пример политики службы управления API Azure. Способы добавления заголовка перенаправления во входящий запрос для обеспечения создания серверным API правильных URL-адресов.
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
ms.openlocfilehash: df72ae5e9a1471e1387539d2c89a1eca0b09d866
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92078618"
---
# <a name="add-a-forwarded-header"></a>Добавление заголовка перенаправления

В этой статье на примере политики службы управления API Azure показано, как добавить заголовок перенаправления во входящий запрос, чтобы обеспечить создание серверным API правильных URL-адресов. См. дополнительные сведения о [создании и изменении кода политик](../set-edit-policies.md). Также для ознакомления доступны другие [примеры политик](../policy-reference.md).

## <a name="code"></a>Код

Вставьте код в блок **inbound**.

[!code-xml[Main](../../../api-management-policy-samples/examples/Forward gateway hostname to backend for generating correct urls in responses.policy.xml)]

## <a name="next-steps"></a>Дальнейшие действия

Подробнее о политиках службы управления API:

+ [Политики преобразования](../api-management-transformation-policies.md)
+ [Примеры политик](../policy-reference.md).