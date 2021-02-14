---
title: Встроенные определения политик для Центра безопасности Azure
description: Здесь приведены встроенные определения политик в Политике Azure для Центра безопасности Azure. Эти встроенные определения политик предоставляют популярные подходы к управлению ресурсами Azure.
ms.date: 02/09/2021
ms.topic: reference
author: memildin
ms.author: memildin
ms.service: security-center
ms.custom: subject-policy-reference
ms.openlocfilehash: ab6556bc391eb3136fa8c4ba40f5a5d71da956e7
ms.sourcegitcommit: 24f30b1e8bb797e1609b1c8300871d2391a59ac2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/10/2021
ms.locfileid: "100093060"
---
# <a name="azure-policy-built-in-definitions-for-azure-security-center"></a>Встроенные определения в Политике Azure для Центра безопасности Azure

Эта страница является индексом встроенных в [политику Azure](../governance/policy/overview.md) определений политик, связанных с центром безопасности Azure. Доступны следующие группы определений политик:

- В группе " [инициативы](#azure-security-center-initiatives) " перечислены определения инициатив политики Azure в категории "Центр безопасности".
- Группа [инициатив по умолчанию](#azure-security-center-initiatives) перечисляет все определения политик Azure, которые входят в инициативу центра безопасности по умолчанию, производительность [системы безопасности Azure](../security/benchmarks/introduction.md). Это разработанное корпорацией Майкрософт, которое в значительной степени соблюдается в отношении элементов управления от [центра безопасности Интернета (CIS)](https://www.cisecurity.org/benchmark/azure/) и [национального института стандартов и технологий (NIST)](https://www.nist.gov/) с целью сосредоточиться на обеспечении безопасности на основе облака.
- Группа [категорий](#azure-security-center-category) содержит все определения политик Azure в категории "Центр безопасности".

Дополнительные сведения об использовании политик безопасности см. в статье [Использование политик безопасности](./tutorial-security-policy.md). Дополнительные встроенные компоненты Политики Azure для других служб см. в статье [Встроенные определения Политики Azure](../governance/policy/samples/built-in-policies.md).

Имя каждого встроенного определения политики связано с определением политики на портале Azure. Перейдите по ссылке в столбце **Версия**, чтобы просмотреть исходный код в [репозитории GitHub для службы "Политика Azure"](https://github.com/Azure/azure-policy).

## <a name="azure-security-center-initiatives"></a>Инициативы центра безопасности Azure

Дополнительные сведения о встроенных инициативах, отслеживаемых центром безопасности, см. в следующей таблице:

[!INCLUDE [azure-policy-reference-policyset-security-center](../../includes/policy/reference/bycat/policysets-security-center.md)]

## <a name="azure-security-center-default-initiative"></a>Инициатива центра безопасности Azure по умолчанию

Дополнительные сведения о встроенных политиках, отслеживаемых центром безопасности, см. в следующей таблице:

[!INCLUDE [azure-policy-reference-init-asc](../../includes/policy/reference/custom/init-asc.md)]

## <a name="azure-security-center-category"></a>Категория центра безопасности Azure

[!INCLUDE [azure-policy-reference-category-securitycenter](../../includes/policy/reference/bycat/policies-security-center.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали об определениях политик безопасности в Azure в центре безопасности. Дополнительные сведения см. в следующих статьях.

- Ознакомьтесь со встроенными инициативами в [репозитории GitHub для Политики Azure](https://github.com/Azure/azure-policy).
- Изучите статью о [структуре определения Политики Azure](../governance/policy/concepts/definition-structure.md).
- Изучите [сведения о действии политик](../governance/policy/concepts/effects.md).
- [Руководство по планированию и эксплуатации центра безопасности Azure](./security-center-planning-and-operations-guide.md). Узнайте, как планировать и анализировать вопросы проектирования в центре безопасности Azure.
- [Наблюдение за работоспособностью системы безопасности в Центре безопасности Azure](./security-center-monitoring.md) — узнайте, как отслеживать работоспособность ресурсов Azure.
- [Управление оповещениями безопасности в Центре безопасности Azure и реагирование на них](./security-center-managing-and-responding-alerts.md) — узнайте, как управлять оповещениями системы безопасности и реагировать на них.
- [Управление подключенными партнерскими решениями с помощью центра безопасности Azure](./security-center-partner-integration.md) — узнайте, как отслеживать состояние работоспособности решений партнеров.
- [Политика Azure](../governance/policy/overview.md). Узнайте, как проводить аудит и управлять ресурсами Azure.
