---
title: Встроенные определения политик для Центра безопасности Azure
description: Здесь приведены встроенные определения политик в Политике Azure для Центра безопасности Azure. Эти встроенные определения политик предоставляют популярные подходы к управлению ресурсами Azure.
ms.date: 04/14/2021
ms.topic: reference
author: memildin
ms.author: memildin
ms.service: security-center
ms.custom: subject-policy-reference
ms.openlocfilehash: 8023dcc29fe8ea3af853237e911524549e4d2bcb
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107739644"
---
# <a name="azure-policy-built-in-definitions-for-azure-security-center"></a>Встроенные определения в Политике Azure для Центра безопасности Azure

Эта страница представляет собой индекс встроенных определений политик в [Политике Azure](../governance/policy/overview.md), связанных с Центром безопасности Azure. Доступны следующие группы определений политик:

- Группа [инициатив](#azure-security-center-initiatives) содержит определения инициатив Политики Azure в категории "Центр безопасности".
- Группа [инициатив по умолчанию](#azure-security-center-initiatives) содержит все определения Политики Azure, которые входят в инициативу Центра безопасности по умолчанию [Azure Security Benchmark](https://docs.microsoft.com/security/benchmark/azure/introduction). Этот широко используемый разработанный Майкрософт механизм тестирования основан на стандартах [Центра интернет-безопасности (CIS)](https://www.cisecurity.org/benchmark/azure/) и [Национального института стандартов и технологий (NIST)](https://www.nist.gov/) с ориентацией на облачную безопасность.
- Группа [категорий](#azure-security-center-category) содержит все определения Политики Azure в категории "Центр безопасности".

Дополнительные сведения об использовании политик безопасности см. в статье [Использование политик безопасности](./tutorial-security-policy.md). Дополнительные встроенные компоненты Политики Azure для других служб см. в статье [Встроенные определения Политики Azure](../governance/policy/samples/built-in-policies.md).

Имя каждого встроенного определения политики связано с определением политики на портале Azure. Перейдите по ссылке в столбце **Версия**, чтобы просмотреть исходный код в [репозитории GitHub для службы "Политика Azure"](https://github.com/Azure/azure-policy).

## <a name="azure-security-center-initiatives"></a>Инициативы Центра безопасности Azure

В следующей таблице представлены сведения о встроенных инициативах, которые отслеживаются с помощью Центра безопасности:

[!INCLUDE [azure-policy-reference-policyset-security-center](../../includes/policy/reference/bycat/policysets-security-center.md)]

## <a name="security-centers-default-initiative-azure-security-benchmark"></a>Инициатива по умолчанию Центра безопасности (Azure Security Benchmark)

В следующей таблице представлены сведения о встроенных политиках, которые отслеживаются с помощью Центра безопасности:

[!INCLUDE [azure-policy-reference-init-asc](../../includes/policy/reference/custom/init-asc.md)]

## <a name="azure-security-center-category"></a>Категория Центра безопасности Azure

[!INCLUDE [azure-policy-reference-category-securitycenter](../../includes/policy/reference/bycat/policies-security-center.md)]

## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали об определениях политик безопасности службы "Политика Azure" в Центре безопасности. Дополнительные сведения об инициативах, политиках и их связи с рекомендациями Центра безопасности см. в статье [Что собой представляют политики безопасности, а также инициативы и рекомендации по ее обеспечению?](security-policy-concept.md)
