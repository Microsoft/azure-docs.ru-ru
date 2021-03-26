---
title: Базовый план безопасности Azure для графа ресурсов Azure
description: Базовый план безопасности графа ресурсов Azure содержит рекомендации и ресурсы для реализации рекомендаций по безопасности, указанных в статье о производительности системы безопасности Azure.
author: msmbaldwin
ms.service: resource-graph
ms.topic: conceptual
ms.date: 02/17/2021
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: ad8968fdb6548da29a031f0e44bd3671f67b5553
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105557723"
---
# <a name="azure-security-baseline-for-azure-resource-graph"></a>Базовый план безопасности Azure для графа ресурсов Azure

Этот базовый план безопасности применяет рекомендации из [тестового показателя безопасности Azure версии 1,0](../../../security/benchmarks/overview-v1.md) к графу ресурсов Azure. Azure Security Benchmark содержит рекомендации по обеспечению безопасности облачных решений в Azure.
Содержимое группируются по **элементам управления безопасностью** , определенным в тестовом производительности системы безопасности Azure, и связанным рекомендациям, применимым к графу ресурсов Azure. **Элементы управления** , неприменимые к графу ресурсов Azure, были исключены.

 
Чтобы увидеть, как граф ресурсов Azure полностью сопоставлен с тестовым показателем безопасности Azure, см. [полный базовый файл сопоставления базовых показателей безопасности графа ресурсов Azure](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="identity-and-access-control"></a>Идентификатор и управление доступом

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: идентификация и управление доступом](../../../security/benchmarks/security-control-identity-access-control.md).*

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10. Регулярная проверка и согласование доступа пользователей

**Руководство**. граф ресурсов Azure предоставляет доступ к типам ресурсов и свойствам на основе элементов управления доступом на основе ролей Azure (Azure RBAC). Регулярно проверяйте и просматривайте доступ, предоставленный субъектам безопасности (пользователям, группам и учетным записям служб), чтобы убедиться, что запросы возвращают результаты для соответствующих ресурсов.

- [Разрешения в Azure Resource Graph](../overview.md#permissions-in-azure-resource-graph)

- [Использование проверок доступа для идентификации Azure](../../../active-directory/governance/access-reviews-overview.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="data-protection"></a>Защита данных

*Дополнительные сведения см. в статье [Azure Security Benchmark: защита данных](../../../security/benchmarks/security-control-data-protection.md).*

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6. Контроль доступа к ресурсам с помощью RBAC 

**Руководство**. Использование Azure RBAC для управления доступом к данным и ресурсам. Чтобы использовать граф ресурсов Azure, необходимо иметь соответствующий доступ к ресурсам, к которым необходимо выполнить запрос. Этот доступ должен быть ограничен только чтением и предоставлен только требуемому персоналу.

- [Разрешения в Azure Resource Graph](../overview.md#permissions-in-azure-resource-graph)

- [Как настроить RBAC в Azure](../../../role-based-access-control/role-assignments-rest.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="inventory-and-asset-management"></a>Инвентаризация и управление ресурсами

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: Инвентаризация и управление активами](../../../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6,1. Использование автоматизированного решения для обнаружения ресурсов

**Руководство**. Использование графа ресурсов Azure для запроса и обнаружения всех поддерживаемых ресурсов в подписках, группах управления и клиентах. Убедитесь, что у вас есть соответствующие разрешения в клиенте и вы можете перечислить все подписки Azure, а также ресурсы в ваших подписках.

- [Как создавать запросы с помощью Azure Resource Graph](../first-query-portal.md)

- [Общие сведения об Azure RBAC](../../../role-based-access-control/overview.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6,4: определение и обслуживание инвентаризации утвержденных ресурсов Azure

**Руководство**. Создание инвентаризации утвержденных ресурсов Azure и утвержденного программного обеспечения для ресурсов вычислений в соответствии с потребностями Организации. Используйте граф ресурсов Azure для запроса утвержденных ресурсов Azure и журнала изменений (Предварительная версия), чтобы просмотреть моментальные снимки и просмотреть изменения.

- [Запрос ресурсов Azure с помощью графа ресурсов Azure](../first-query-portal.md)

- [Получение данных об изменении ресурса](../how-to/get-resource-changes.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5. Отслеживание неутвержденных ресурсов Azure

**Руководство**. Использование графа ресурсов Azure для запроса и обнаружения ресурсов в подписках, группах управления и клиентах. Убедитесь, что все ресурсы Azure в окружении утверждены.

- [Запрос ресурсов Azure с помощью графа ресурсов Azure](../first-query-portal.md)

- [Примеры: начальные запросы для графа ресурсов Azure](../samples/starter.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="next-steps"></a>Следующие шаги

- См. [Обзор Azure Security Benchmark версии 2](../../../security/benchmarks/overview.md)
- Дополнительные сведения о [базовой конфигурации безопасности Azure](../../../security/benchmarks/security-baselines-overview.md).