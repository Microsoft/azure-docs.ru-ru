---
title: Встроенные определения политик для Центра безопасности Azure
description: Здесь приведены встроенные определения политик в Политике Azure для Центра безопасности Azure. Эти встроенные определения политик предоставляют популярные подходы к управлению ресурсами Azure.
ms.date: 03/24/2021
ms.topic: reference
author: memildin
ms.author: memildin
ms.service: security-center
ms.custom: subject-policy-reference
ms.openlocfilehash: a96dcc3b6f4abafd6cf72c11b2f52480fe3780ec
ms.sourcegitcommit: bb330af42e70e8419996d3cba4acff49d398b399
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105036715"
---
# <a name="azure-policy-built-in-definitions-for-azure-security-center"></a>Встроенные определения в Политике Azure для Центра безопасности Azure

Эта страница является индексом встроенных в [политику Azure](../governance/policy/overview.md) определений политик, связанных с центром безопасности Azure. Доступны следующие группы определений политик:

- В группе [инициативы](#azure-security-center-initiatives) перечислены определения инициатив политики Azure в категории "Центр безопасности".
- Группа [инициатив по умолчанию](#azure-security-center-initiatives) перечисляет все определения политик Azure, которые входят в инициативу центра безопасности по умолчанию, производительность [системы безопасности Azure](../security/benchmarks/introduction.md). Это разработанное корпорацией Майкрософт, которое в значительной степени соблюдается в отношении элементов управления от [центра безопасности Интернета (CIS)](https://www.cisecurity.org/benchmark/azure/) и [национального института стандартов и технологий (NIST)](https://www.nist.gov/) с целью сосредоточиться на обеспечении безопасности на основе облака.
- Группа [категорий](#azure-security-center-category) содержит все определения политик Azure в категории "Центр безопасности".

Дополнительные сведения об использовании политик безопасности см. в статье [Использование политик безопасности](./tutorial-security-policy.md). Дополнительные встроенные компоненты Политики Azure для других служб см. в статье [Встроенные определения Политики Azure](../governance/policy/samples/built-in-policies.md).

Имя каждого встроенного определения политики связано с определением политики на портале Azure. Перейдите по ссылке в столбце **Версия**, чтобы просмотреть исходный код в [репозитории GitHub для службы "Политика Azure"](https://github.com/Azure/azure-policy).

## <a name="azure-security-center-initiatives"></a>Инициативы центра безопасности Azure

Дополнительные сведения о встроенных инициативах, отслеживаемых центром безопасности, см. в следующей таблице:

[!INCLUDE [azure-policy-reference-policyset-security-center](../../includes/policy/reference/bycat/policysets-security-center.md)]

## <a name="security-centers-default-initiative-azure-security-benchmark"></a>Инициатива по умолчанию для центра безопасности (контрольный тест системы безопасности Azure)

Дополнительные сведения о встроенных политиках, отслеживаемых центром безопасности, см. в следующей таблице:

[!INCLUDE [azure-policy-reference-init-asc](../../includes/policy/reference/custom/init-asc.md)]

## <a name="azure-security-center-category"></a>Категория центра безопасности Azure

[!INCLUDE [azure-policy-reference-category-securitycenter](../../includes/policy/reference/bycat/policies-security-center.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали об определениях политик безопасности в Azure в центре безопасности. Дополнительные сведения о инициативах, политиках и способах их связи с рекомендациями центра безопасности см. в статье [что такое политики безопасности, инициативы и рекомендации?](security-policy-concept.md).
