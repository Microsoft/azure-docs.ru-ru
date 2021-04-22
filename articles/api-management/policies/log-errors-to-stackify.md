---
title: Пример политики управления API — отправка ошибок в Stackify для регистрации
titleSuffix: Azure API Management
description: Пример политики службы управления API Azure. Добавление политики регистрации ошибок для отправки ошибок в Stackify и их записи в журнал.
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
ms.openlocfilehash: 89428e9a64c6d4ae78d333c0cf597531588b1638
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92072073"
---
# <a name="send-errors-to-stackify-for-logging"></a>Отправка ошибок Stackify для записи в журнал

В этой статье на примере политики службы управления API Azure показано, как добавить политику регистрации ошибок для отправки ошибок в Stackify и их записи в журнал. См. дополнительные сведения о [создании и изменении кода политик](../set-edit-policies.md). Также для ознакомления доступны другие [примеры политик](../policy-reference.md).

## <a name="policy"></a>Политика

Вставьте код в блок **on-error**.

[!code-xml[Main](../../../api-management-policy-samples/examples/Log errors to Stackify.policy.xml)]

## <a name="next-steps"></a>Дальнейшие действия

Подробнее о политиках службы управления API:

+ [Политики преобразования](../api-management-transformation-policies.md)
+ [Примеры политик](../policy-reference.md).