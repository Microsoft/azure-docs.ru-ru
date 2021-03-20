---
title: Базовый план безопасности Azure для политики Azure
description: В базовом плане безопасности политики Azure содержатся практические рекомендации и ресурсы для реализации рекомендаций по безопасности, указанных в статье о производительности системы безопасности Azure.
author: msmbaldwin
ms.service: azure-policy
ms.topic: conceptual
ms.date: 02/17/2021
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 9916ad6d3b6cb1a63d34004915666226b7836490
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101740361"
---
# <a name="azure-security-baseline-for-azure-policy"></a>Базовый план безопасности Azure для политики Azure

Этот базовый план безопасности применяет рекомендации из [теста безопасности Azure](../../../security/benchmarks/overview.md) к политике Azure. Azure Security Benchmark содержит рекомендации по обеспечению безопасности облачных решений в Azure. Содержимое группируются по **доменам соответствия** и **элементам управления безопасностью** , определенным в производительности системы безопасности Azure, и связанным рекомендациям, применимым к политике Azure. **Элементы управления** , неприменимые к политике Azure, были исключены. Сведения о том, как полностью сопоставить политику Azure с тестовым показателем безопасности Azure, см. в [полном файле сопоставления базовых показателей безопасности политики Azure](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

Сведения о сопоставлении элементов управления производительности Azure с встроенными определениями политик с помощью встроенной инициативы см. в разделе [соответствие нормативным требованиям: производительность системы безопасности Azure](../samples/azure-security-benchmark.md).

Политика Azure использует термин _владение_ вместо _ответственности_. Дополнительные сведения о _владельце_ см. в статье [определения политик](./definition-structure.md#type) и [Общие обязанности политики Azure в облаке](../../../security/fundamentals/shared-responsibility.md).

## <a name="logging-and-monitoring"></a>Ведение журналов и мониторинг

*Дополнительные сведения см. в статье [производительность системы безопасности Azure: ведение журнала и мониторинг](../../../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3. Включение журналов аудита для ресурсов Azure

**Руководство**. Политика Azure использует журналы действий, которые включаются автоматически, для включения источника событий, даты, пользователя, метки времени, исходных адресов, адресов назначения и других полезных элементов.

- [Как получить журналы и метрики платформы с помощью Azure Monitor](/azure/azure-monitor/platform/diagnostic-settings)

- [Общие сведения о ведении журналов и различных типах журналов в Azure](/azure/azure-monitor/platform/platform-logs-overview)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="identity-and-access-control"></a>Идентификатор и управление доступом

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: идентификация и управление доступом](../../../security/benchmarks/security-control-identity-access-control.md).*

### <a name="33-use-dedicated-administrative-accounts"></a>3.3. Применение выделенных административных учетных записей

**Руководство**. Создайте стандартные операционные процедуры для использования выделенных административных учетных записей. Используйте учетную запись Центра безопасности Azure и управление доступом для мониторинга количества административных учетных записей. Вы также можете разрешить JIT-доступ и достаточное для него решение с помощью [Azure Active Directory (Azure AD) Управление привилегированными пользователями](../../../active-directory/privileged-identity-management/pim-configure.md) привилегированных ролей или [Azure Resource Manager](../../../azure-resource-manager/management/overview.md).

**Ответственность**: Customer

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. гуестконфигуратион**:

[!INCLUDE [Resource Policy for Microsoft.GuestConfiguration 3.3](../../../../includes/policy/standards/asb/rp-controls/microsoft.guestconfiguration-3-3.md)]

### <a name="36-use-secure-azure-managed-workstations-for-administrative-tasks"></a>3,6: используйте защищенные рабочие станции, управляемые Azure, для административных задач

**Руководство**. Использование рабочих станций привилегированного доступа (лапы) с многофакторной проверкой подлинности, настроенной для входа и настройки ресурсов Azure.

- [Использование рабочих станций с привилегированным доступом](https://4sysops.com/archives/understand-the-microsoft-privileged-access-workstation-paw-security-model/)

- [Как включить многофакторную проверку подлинности в Azure](../../../active-directory/authentication/howto-mfa-getstarted.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="data-protection"></a>Защита данных

*Дополнительные сведения см. в статье [Azure Security Benchmark: защита данных](../../../security/benchmarks/security-control-data-protection.md).*

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6. Контроль доступа к ресурсам с помощью RBAC 

**Руководство**. Использование управления доступом на основе ролей Azure (Azure RBAC) для управления доступом к политике Azure.

- [Разрешения Azure RBAC в Политике Azure](https://docs.microsoft.com/azure/governance/policy/overview#azure-rbac-permissions-in-azure-policy)

- [Настройка Azure RBAC](../../../role-based-access-control/role-assignments-portal.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9. Включение в журнал и создание оповещений по изменениям критических ресурсов Azure

**Руководство**. Использование Azure Monitor с журналами действий для создания оповещений о том, когда изменения выполняются в политике Azure.

- [Создание оповещений для событий журнала действий Azure](/azure/azure-monitor/platform/alerts-activity-log)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="inventory-and-asset-management"></a>Инвентаризация и управление ресурсами

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: Инвентаризация и управление активами](../../../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="62-maintain-asset-metadata"></a>6.2. Ведение метаданных активов

**Руководство**. Применяйте к ресурсам Azure теги, чтобы логически классифицировать их на основе метаданных. Используйте действие _изменение_ политики Azure для включения и соблюдения соответствия требованиям и управления тегами.

- [Учебник. Создание политик и управление ими](../tutorials/create-and-manage.md)

- [Руководство по Администрирование системы управления тегами](../tutorials/govern-tags.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6,4: определение и обслуживание инвентаризации утвержденных ресурсов Azure

**Руководство**. Создание инвентаризации утвержденных определений политик и назначений политик в соответствии с потребностями Организации.

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5. Отслеживание неутвержденных ресурсов Azure

**Руководство**. Использование политики Azure для ограничения типа ресурсов, которые могут быть созданы в подписках.

- [Настройка Политики Azure и управление ею](../tutorials/create-and-manage.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="next-steps"></a>Следующие шаги

- См. [Обзор Azure Security Benchmark версии 2](/azure/security/benchmarks/overview)
- Дополнительные сведения о [базовой конфигурации безопасности Azure](/azure/security/benchmarks/security-baselines-overview).
