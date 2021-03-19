---
title: Вопросы и ответы по центру безопасности Azure — вопросы о разрешениях
description: В этой статье содержатся ответы на вопросы о разрешениях в центре безопасности Azure — продукте, который помогает предотвращать, обнаруживать угрозы и реагировать на них.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: be2ab6d5-72a8-411f-878e-98dac21bc5cb
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/25/2020
ms.author: memildin
ms.openlocfilehash: 88d6207bf44d1ec04fde4f74ac9295412ea88fbc
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87824619"
---
# <a name="permissions"></a>Разрешения

## <a name="how-do-permissions-work-in-azure-security-center"></a>Как работают разрешения в центре безопасности Azure?

Центр безопасности Azure использует [управление доступом на основе ролей (RBAC)](../role-based-access-control/role-assignments-portal.md) с несколькими [встроенными ролями](../role-based-access-control/built-in-roles.md), которые можно назначать пользователям, группам и службам.

Центр безопасности оценивает конфигурацию ресурсов, чтобы выявить проблемы безопасности и уязвимости. В центре безопасности отображается только информация, связанная с ресурсами, для которых вам назначена роль владельца, участника или читателя в рамках подписки или группы ресурсов, к которым принадлежит ресурс.

Подробные сведения о ролях и разрешенных действиях в центре безопасности см. в статье [Permissions in Azure Security Center](security-center-permissions.md) (Разрешения в центре безопасности Azure).



## <a name="who-can-modify-a-security-policy"></a>Кто может изменять политику безопасности?

Чтобы изменить политику безопасности, необходимо быть администратором безопасности или владельцем или участником этой подписки.

Дополнительные сведения о настройке политики безопасности см. в статье [Настройка политик безопасности в Центре безопасности Azure](tutorial-security-policy.md).