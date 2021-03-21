---
title: Базовый план безопасности Azure для Azure Resource Manager
description: Базовый план безопасности Azure Resource Manager содержит практические руководства и ресурсы для реализации рекомендаций по безопасности, указанных в статье о производительности системы безопасности Azure.
author: msmbaldwin
ms.service: azure-resource-manager
ms.topic: conceptual
ms.date: 02/17/2021
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 996159f7014e735c8a4b3e5975c2716ce1ab6910
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101728557"
---
# <a name="azure-security-baseline-for-azure-resource-manager"></a>Базовый план безопасности Azure для Azure Resource Manager

Этот базовый план безопасности применяет рекомендации из [теста безопасности Azure версии 1,0](../../security/benchmarks/overview-v1.md) к Microsoft Azure диспетчер ресурсов. Azure Security Benchmark содержит рекомендации по обеспечению безопасности облачных решений в Azure.
Содержимое группируются по **элементам управления безопасностью** , определенным в производительности системы безопасности Azure, и связанным рекомендациям, применимым к Azure Resource Manager. **Элементы управления** , неприменимые к Azure Resource Manager, были исключены.

 
Сведения о том, как Azure Resource Manager полностью сопоставлены с тестовым показателем безопасности Azure, см. в разделе [полное Azure Resource Manager файла сопоставления базовых показателей безопасности](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="logging-and-monitoring"></a>Ведение журналов и мониторинг

*Дополнительные сведения см. в статье [производительность системы безопасности Azure: ведение журнала и мониторинг](../../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="22-configure-central-security-log-management"></a>2.2. Настройка централизованного управления журналами безопасности

**Руководство**. прием журналов действий политики Azure с помощью Azure Monitor. В Azure Monitor используйте рабочие области Log Analytics, чтобы запрашивать и выполнять анализ, а также использовать учетные записи хранения Azure для долгосрочного или архивного хранилища. Кроме того, вы можете включить и подключить данные к Azure Sentinel или сторонним SIEM.

- [Подключение к Azure Sentinel](../../sentinel/quickstart-onboard.md) 

- [Как получить журналы и метрики платформы с помощью Azure Monitor](/azure/azure-monitor/platform/diagnostic-settings) 

- [Как получить журналы внутреннего узла виртуальной машины Azure с помощью Azure Monitor](/azure/azure-monitor/learn/quick-collect-azurevm) 

- [Начало работы с Azure Monitor и интеграция SIEM стороннего производителя](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

**Ответственность**: Customer

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. Resources**:

[!INCLUDE [Resource Policy for Microsoft.Resources 2.2](../../../includes/policy/standards/asb/rp-controls/microsoft.resources-2-2.md)]

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3. Включение журналов аудита для ресурсов Azure

**Руководство**. Azure Resource Manager использует журналы действий, которые включаются автоматически, для включения источника событий, даты, пользователя, метки времени, исходных адресов, адресов назначения и других полезных элементов.

- [Как получить журналы и метрики платформы с помощью Azure Monitor](/azure/azure-monitor/platform/diagnostic-settings) 

- [Общие сведения о ведении журналов и различных типах журналов в Azure](/azure/azure-monitor/platform/platform-logs-overview)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="26-monitor-and-review-logs"></a>2.6. Мониторинг и просмотр журналов

**Руководство**. анализ и мониторинг журналов для аномального поведения и регулярная проверка результатов. Используйте Azure Monitor и рабочую область Log Analytics для просмотра журналов и выполнения запросов к данным журнала. 

Кроме того, вы можете включить и подключить данные к Azure Sentinel или сторонним SIEM. 

- [Подключение к Azure Sentinel](../../sentinel/quickstart-onboard.md) 

- [Приступая к работе с Log Analytics запросами](/azure/azure-monitor/log-query/log-analytics-tutorial)

- [Выполнение пользовательских запросов в Azure Monitor](/azure/azure-monitor/log-query/get-started-queries)

**Ответственность**: Совмещаемая блокировка

**Мониторинг центра безопасности Azure**: нет

### <a name="27-enable-alerts-for-anomalous-activities"></a>2,7: Включение оповещений для аномальных действий

**Руководство**. Использование центра безопасности Azure с log Analytics для мониторинга и оповещения о аномальных действиях, обнаруженных в журналах действий. Кроме того, вы можете включить и подключить данные в Azure Sentinel. 

- [Подключение к Azure Sentinel](../../sentinel/quickstart-onboard.md) 

- [Управление оповещениями в центре безопасности Azure](../../security-center/security-center-managing-and-responding-alerts.md) 

- [Как оповещать данные журнала Log Analytics](/azure/azure-monitor/learn/tutorial-response)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="identity-and-access-control"></a>Идентификатор и управление доступом

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: идентификация и управление доступом](../../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1. Инвентаризация учетных записей администраторов

**Руководство**. Управление доступом на основе РОЛЕЙ (RBAC) в Azure позволяет управлять доступом к ресурсам Azure с помощью назначений ролей. Эти роли можно назначить пользователям, группам субъектов-служб и управляемым удостоверениям. Существуют предварительно определенные встроенные роли для определенных ресурсов, и эти роли можно выполнять по инвентаризации или запрашивать с помощью таких средств, как Azure CLI, Azure PowerShell или портал Azure.

- [Как получить роль каталога в Azure Active Directory (Azure AD) с помощью PowerShell](/powershell/module/azuread/get-azureaddirectoryrole)

- [Как получить членов роли каталога в Azure AD с помощью PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember)

**Ответственность**: Customer

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. Resources**:

[!INCLUDE [Resource Policy for Microsoft.Resources 3.1](../../../includes/policy/standards/asb/rp-controls/microsoft.resources-3-1.md)]

### <a name="33-use-dedicated-administrative-accounts"></a>3.3. Применение выделенных административных учетных записей

**Руководство**. Создайте стандартные операционные процедуры для использования выделенных административных учетных записей. Используйте учетную запись Центра безопасности Azure и управление доступом для мониторинга количества административных учетных записей.

Кроме того, чтобы помочь вам отследить выделенные административные учетные записи, вы можете использовать рекомендации из центра безопасности Azure или встроенных политик Azure, например:
- Подписке должно быть назначено несколько владельцев.
- Устаревшие учетные записи с разрешениями владельца следует удалять из подписки.
- Внешние учетные записи с разрешениями владельца следует удалять из подписки.

Вы также можете включить JIT-доступ с помощью Azure Active Directory (Azure AD) управление привилегированными пользователями и Azure Resource Manager.

- [Дополнительные сведения о управление привилегированными пользователями](/azure/active-directory/privileged-identity-management/)

- [Использование Политики Azure](../../governance/policy/tutorials/create-and-manage.md)

**Ответственность**: Customer

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. Resources**:

[!INCLUDE [Resource Policy for Microsoft.Resources 3.3](../../../includes/policy/standards/asb/rp-controls/microsoft.resources-3-3.md)]

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4. Использование единого входа с Azure Active Directory

**Рекомендации**. везде, где это возможно, используйте Azure Active Directory (Azure AD) SSO вместо настройки отдельных автономных учетных данных для каждой службы. Использование удостоверений центра безопасности Azure и рекомендаций по доступу.

- [Общие сведения об использовании единого входа в Azure AD](../../active-directory/manage-apps/what-is-single-sign-on.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5. Использование многофакторной проверки подлинности для любого доступа на основе Azure Active Directory

**Руководство**. Включение многофакторной проверки подлинности в Azure Active Directory (Azure AD) и следование рекомендациям по управлению удостоверениями и доступом в центре безопасности Azure.

- [Как включить многофакторную проверку подлинности в Azure](../../active-directory/authentication/howto-mfa-getstarted.md)

- [Мониторинг идентификации и доступа в Центре безопасности Azure](../../security-center/security-center-identity-access.md)

**Ответственность**: Customer

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. Resources**:

[!INCLUDE [Resource Policy for Microsoft.Resources 3.5](../../../includes/policy/standards/asb/rp-controls/microsoft.resources-3-5.md)]

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6. Использование выделенных компьютеров (рабочих станций с привилегированным доступом) для всех административных задач

**Руководство**. используйте защищенную, управляемую Azure рабочую станцию (также называемую рабочей станцией привилегированного доступа или привилегированным доступом) для административных задач, требующих повышенных привилегий.

- [Общие сведения о защищенных рабочих станциях под управлением Azure](https://4sysops.com/archives/understand-the-microsoft-privileged-access-workstation-paw-security-model/)

- [Как включить многофакторную проверку подлинности Azure Active Directory (Azure AD)](../../active-directory/authentication/howto-mfa-getstarted.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3,7: журналы и оповещения о подозрительных действиях учетных записей администраторов

**Руководство**. Использование Azure Active Directory (Azure AD) отчетов о безопасности и мониторинга для обнаружения подозрительных или ненадежных действий в среде. Используйте Центр безопасности Azure для мониторинга действий идентификации и доступа.

- [Как определить пользователей Azure AD, помеченных для события риска](../../active-directory/identity-protection/overview-identity-protection.md)

- [Мониторинг пользовательских действий идентификации и доступа в Центре безопасности Azure](../../security-center/security-center-identity-access.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="38-manage-azure-resources-only-from-approved-locations"></a>3,8: управление ресурсами Azure только из утвержденных расположений

**Руководство**. Использование Azure Active Directory (Azure AD) с именованными расположениями для разрешения доступа только из конкретных логических групп диапазонов IP-адресов или стран и регионов.

- [Настройка именованных расположений Azure AD](../../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="39-use-azure-active-directory"></a>3.9. Использование Azure Active Directory

**Руководство**. Использование Azure Active Directory (Azure AD) с именованными расположениями для разрешения доступа только из конкретных логических групп диапазонов IP-адресов или стран и регионов.

- [Настройка именованных расположений Azure AD](../../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10. Регулярная проверка и согласование доступа пользователей

**Руководство**. Azure Active Directory (Azure AD) предоставляет журналы для облегчения поиска устаревших учетных записей. Кроме того, используйте проверки подлинности и доступа Azure AD для эффективного управления членством в группах, доступом к корпоративным приложениям и назначениями ролей. Доступ пользователей можно проверить регулярно, чтобы убедиться, что доступ к ним имеют только нужные пользователи.

- [Общие сведения об отчетах Azure AD](/azure/active-directory/reports-monitoring/)

- [Использование проверок доступа для идентификации Azure AD](../../active-directory/governance/access-reviews-overview.md)

**Ответственность**: Customer

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. Resources**:

[!INCLUDE [Resource Policy for Microsoft.Resources 3.10](../../../includes/policy/standards/asb/rp-controls/microsoft.resources-3-10.md)]

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3,11: монитор пытается получить доступ к отключенным учетным данным

**Руководство**. у вас есть доступ к источникам журналов событий входа в систему Azure Active Directory (Azure AD), аудита и риску, которые позволяют интегрироваться с любым средством SIEM/Monitoring.

Этот процесс можно упростить, создав параметры диагностики для учетных записей пользователей Azure AD и отправив журналы аудита и журналы входа в рабочую область Log Analytics. Вы можете настроить нужные оповещения в Log Analytics рабочей области.

- [Как интегрировать журналы действий Azure с Azure Monitor](/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12. Предупреждение при подозрительном входе в учетную запись

**Руководство**. Использование функций защиты идентификации Azure Active Directory (Azure AD) для настройки автоматических ответов на обнаруженные подозрительные действия, связанные с удостоверениями пользователей. Вы также можете включить данные в Azure Sentinel для дальнейшего изучения.

- [Просмотр рискованных входов в Azure AD](../../active-directory/identity-protection/overview-identity-protection.md)

- [Как настроить и включить политики рисков с помощью защиты идентификации](../../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Подключение к Azure Sentinel](../../sentinel/quickstart-onboard.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="data-protection"></a>Защита данных

*Дополнительные сведения см. в статье [Azure Security Benchmark: защита данных](../../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1. Инвентаризация конфиденциальных данных

**Руководство**. Используйте теги для пометки ресурсов Azure, в которых хранятся или обрабатываются конфиденциальные данные.

- [Создание и использование тегов](tag-resources.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="46-use-azure-rbac-to-manage-access-to-resources"></a>4,6. Использование Azure RBAC для управления доступом к ресурсам

**Руководство**. Использование Azure Active Directory (Azure AD) RBAC для управления доступом к данным и ресурсам. в противном случае используйте зависящие от службы методы управления доступом.

- [Настройка Azure RBAC](../../role-based-access-control/role-assignments-portal.md)

**Ответственность**: Customer

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. Authorization**:

[!INCLUDE [Resource Policy for Microsoft.Authorization 4.6](../../../includes/policy/standards/asb/rp-controls/microsoft.authorization-4-6.md)]

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8. Шифрование конфиденциальной информации при хранении

**Рекомендации**. для шифрования неактивных на стороне сервера Azure Resource Manager поддерживает ключи, управляемые корпорацией Майкрософт.

- [Общие сведения о защите данных в Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-resource-manager-security-controls#data-protection)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9. Включение в журнал и создание оповещений по изменениям критических ресурсов Azure

**Руководство**. Использование Azure Monitor с журналом действий Azure для создания оповещений при внесении изменений в критические ресурсы Azure.

- [Создание оповещений для событий журнала действий Azure](/azure/azure-monitor/platform/alerts-activity-log)

**Ответственность**: Customer

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. Resources**:

[!INCLUDE [Resource Policy for Microsoft.Resources 4.9](../../../includes/policy/standards/asb/rp-controls/microsoft.resources-4-9.md)]

## <a name="inventory-and-asset-management"></a>Инвентаризация и управление ресурсами

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: Инвентаризация и управление активами](../../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6,1. Использование автоматизированного решения для обнаружения ресурсов

**Руководство**. Использование графа ресурсов Azure для запроса и обнаружения всех ресурсов (таких как вычисление, хранение, сеть, порты, протоколы и т. д.) в подписках. Обеспечьте соответствующие разрешения (чтение) в клиенте и перечислите все подписки Azure, а также ресурсы в подписках.

Несмотря на то, что классические ресурсы Azure могут быть обнаружены через обозреватель графа ресурсов Azure, настоятельно рекомендуется создавать и использовать ресурсы Azure Resource Manager.

- [Как создавать запросы с помощью Azure Resource Graph](../../governance/resource-graph/first-query-portal.md)

- [Просмотр подписок Azure](/powershell/module/az.accounts/get-azsubscription)

- [Общие сведения об Azure RBAC](../../role-based-access-control/overview.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="62-maintain-asset-metadata"></a>6.2. Ведение метаданных активов

**Руководство**. Использование имени политики, описания и категории для логической организации активов в соответствии с классификацией.

- [Дополнительные сведения о разметке ресурсов см. в разделе Определение решения для именования и добавления тегов.](https://docs.microsoft.com/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=/azure/azure-resource-manager/management/toc.json)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="63-delete-unauthorized-azure-resources"></a>6.3. Удаление неавторизованных ресурсов Azure

**Руководство**. При необходимости используйте теги, группы управления и отдельные подписки, чтобы упорядочить и отслеживать ресурсы Azure. Регулярно сверяйте ресурсы, чтобы своевременно удалять неавторизованные ресурсы из подписки.

Кроме того, используйте политику Azure, чтобы ограничить тип ресурсов, которые могут быть созданы в клиентских подписках, используя следующие встроенные определения политик:

- Недопустимые типы ресурсов
- Допустимые типы ресурсов

Дополнительные сведения см. ниже.

- [Создание дополнительных подписок Azure](../../cost-management-billing/manage/create-subscription.md)

- [Создание групп управления](../../governance/management-groups/create-management-group-portal.md)

- [Создание и использование тегов](tag-resources.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="64-define-and-maintain-an-inventory-of-approved-azure-resources"></a>6,4: определение и обслуживание инвентаризации утвержденных ресурсов Azure

**Руководство**. Создание инвентаризации утвержденных ресурсов Azure и утвержденного программного обеспечения для ресурсов вычислений в соответствии с потребностями Организации.

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5. Отслеживание неутвержденных ресурсов Azure

**Руководство**. Использование политики Azure для ограничения типа ресурсов, которые могут быть созданы в подписках.

Используйте Azure Resource Graph для запроса и обнаружения ресурсов в своих подписках.  Убедитесь в том, что все ресурсы Azure, представленные в среде, утверждены.

- [Настройка Политики Azure и управление ею](../../governance/policy/tutorials/create-and-manage.md)

- [Как создавать запросы с помощью Azure Resource Graph](../../governance/resource-graph/first-query-portal.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="69-use-only-approved-azure-services"></a>6.9. Использование только утвержденных служб Azure

**Руководство**. Использование политики Azure для ограничения типа ресурсов, которые могут быть созданы в клиентских подписках, с помощью следующих встроенных определений политик:

- Недопустимые типы ресурсов
- Допустимые типы ресурсов

Дополнительные сведения см. ниже.

- [Настройка Политики Azure и управление ею](../../governance/policy/tutorials/create-and-manage.md)

- [Как отказаться от определенного типа ресурса с помощью Политики Azure](https://docs.microsoft.com/azure/governance/policy/samples/built-in-policies#general)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6,11: ограничьте возможность пользователей работать с Azure Resource Manager

**Руководство**. Использование условного доступа Azure Active Directory (Azure AD) для ограничения возможности пользователей взаимодействовать с диспетчером ресурсов Azure путем настройки "блокировать доступ" для приложения "Управление Microsoft Azure".

- [Настройка условного доступа для блокировки доступа к диспетчеру ресурсов Azure](../../role-based-access-control/conditional-access-azure-management.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="612-limit-users-ability-to-execute-scripts-in-compute-resources"></a>6,12: ограничьте возможность пользователей выполнять сценарии в ресурсах вычислений

**Руководство**. Неприменимо; эта рекомендация предназначена для вычислительных ресурсов.

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="secure-configuration"></a>Настройка безопасности

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: безопасная конфигурация](../../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1. Установка безопасных конфигураций для всех ресурсов Azure

**Руководство**. Использование псевдонимов политик Azure для создания настраиваемых политик для аудита или принудительного применения конфигурации ресурсов Azure. Вы также можете использовать встроенные определения политик Azure.

Azure Resource Manager может экспортировать шаблон в нотация объектов JavaScript (JSON), который следует проверить, чтобы убедиться, что конфигурации соответствуют требованиям безопасности Организации.

Вы также можете использовать рекомендации из центра безопасности Azure в качестве защищенного шаблона базовой конфигурации для ресурсов Azure.

- [Просмотр доступных псевдонимов политик Azure](/powershell/module/az.resources/get-azpolicyalias)

- [Руководство по Создание политик и управление ими для обеспечения соответствия требованиям](../../governance/policy/tutorials/create-and-manage.md)

- [Экспорт одного и нескольких ресурсов в шаблон в портал Azure](../templates/export-template-portal.md)

- [Рекомендации по безопасности — справочное руководство](../../security-center/recommendations-reference.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3. Сохранение безопасных конфигураций для ресурсов Azure

**Рекомендации**. Используйте Политику Azure [отказывать] и [развернуть, если не существует], чтобы обеспечить безопасность параметров в ресурсах Azure.  Кроме того, можно использовать шаблоны Azure Resource Manager для поддержания конфигурации безопасности ресурсов Azure, необходимых вашей организации. 

Кроме того, в качестве администратора может потребоваться заблокировать подписку, группу ресурсов или ресурс, чтобы другие пользователи в Организации не могли случайно удалить или изменить критически важные ресурсы. Можно установить уровень блокировки CanNotDelete или ReadOnly.

- [Общие сведения о влиянии политики Azure](../../governance/policy/concepts/effects.md)

- [Создание политик и управление ими для обеспечения соответствия](../../governance/policy/tutorials/create-and-manage.md)

- [Общие сведения о шаблонах Azure Resource Manager](../templates/overview.md)

- [Блокировка ресурсов для предотвращения непредвиденных изменений](lock-resources.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5. Безопасное хранение конфигурации ресурсов Azure

**Руководство**. Использование Azure DevOps для безопасного хранения и управления кодом, например пользовательскими определениями политик Azure, Azure Resource Manager шаблонами и скриптами настройки требуемого состояния. Чтобы получить доступ к ресурсам, которыми вы управляете в Azure DevOps, вы можете предоставить или отклонить разрешения для определенных пользователей, встроенных групп безопасности или групп, определенных в Azure Active Directory (Azure AD), если они интегрированы с Azure DevOps, или Active Directory, если они интегрированы с TFS.

- [Как хранить код в Azure DevOps](/azure/devops/repos/git/gitworkflow)

- [О разрешениях и группах в Azure DevOps](/azure/devops/organizations/security/about-permissions)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7,7: развертывание средств управления конфигурацией для ресурсов Azure

**Руководство**. Определение и реализация стандартных конфигураций безопасности для ресурсов Azure с помощью политики Azure. Используйте псевдонимы политик Azure для создания настраиваемых политик для аудита или принудительного применения конфигурации сети ресурсов Azure. Можно также использовать встроенные определения политик, связанные с конкретными ресурсами.  Кроме того, можно использовать службу автоматизации Azure для развертывания изменений конфигурации.

- [Настройка Политики Azure и управление ею](../../governance/policy/tutorials/create-and-manage.md)

- [Использование псевдонимов](https://docs.microsoft.com/azure/governance/policy/concepts/definition-structure#aliases)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7,9. Реализация автоматического мониторинга конфигурации для ресурсов Azure

**Руководство**. Использование встроенных определений политик Azure, а также пользовательских политик для оповещения, аудита и принудительного применения конфигураций системы. Используйте политику Azure [аудит], [запретить] и [развернуть, если не существует] для автоматического применения конфигураций для ресурсов Azure.

- [Настройка Политики Azure и управление ею](../../governance/policy/tutorials/create-and-manage.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13. Устранение непреднамеренного раскрытия учетных данных

**Рекомендации**. использование рекомендаций при создании шаблона ARM. Эти рекомендации помогут избежать распространенных проблем при использовании шаблона ARM для развертывания решения.

Реализуйте средство проверки учетных данных для обнаружения учетных данных в коде. Сканер учетных данных также рекомендует перемещать обнаруженные учетные данные в более безопасные расположения, такие как Azure Key Vault. 

- [Реализация рекомендаций по обеспечению безопасности шаблонов](../templates/template-best-practices.md)

- [Как настроить сканер учетных данных](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="data-recovery"></a>Восстановление данных

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: восстановление данных](../../security/benchmarks/security-control-data-recovery.md).*

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9,3: Проверьте все резервные копии, включая управляемые клиентом ключи.

**Руководство**. при необходимости обеспечить возможность регулярного развертывания шаблонов Azure Resource Manager в изолированной подписке.

- [Развертывание ресурсов с помощью шаблонов ARM и портал Azure](../templates/deploy-portal.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9,4: Обеспечьте защиту резервных копий и ключей, управляемых клиентом

**Руководство**. Использование Azure DevOps для безопасного хранения и управления кодом, например пользовательскими определениями политик Azure, Azure Resource Manager шаблонами и скриптами настройки требуемого состояния. Чтобы получить доступ к ресурсам, которыми вы управляете в Azure DevOps, вы можете предоставить или отклонить разрешения для определенных пользователей, встроенных групп безопасности или групп, определенных в Azure Active Directory (Azure AD), если они интегрированы с Azure DevOps, или Active Directory, если они интегрированы с TFS.

- [Как хранить код в Azure DevOps](/azure/devops/repos/git/gitworkflow)

- [О разрешениях и группах в Azure DevOps](/azure/devops/organizations/security/about-permissions)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="incident-response"></a>реагирование на инциденты.

*Дополнительные сведения см. в статье [Azure Security Benchmark: реагирование на инциденты](../../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1. Создание руководства по реагированию на инциденты

**Руководство**. Разработка руководства по реагированию на инциденты для вашей организации. Убедитесь, что имеются письменные планы реагирования на инциденты, которые определяют все роли персонала, а также этапы обработки инцидентов и управления обнаружением до проверки после инцидента. 

- [Руководство по созданию собственного процесса реагирования на инциденты безопасности](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Анатомия инцидента Центра Майкрософт по реагированию на угрозы](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)

- [Воспользуйтесь руководством по обработке инцидентов компьютерной безопасности NIST, чтобы помочь в создании собственного плана реагирования на инциденты.](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="103-test-security-response-procedures"></a>10.3. Проверка процедур реагирования на угрозы

**Руководство**. проведение упражнений по тестированию возможностей реагирования на инциденты систем с регулярным интервалом для защиты ресурсов Azure. Выявление слабых точек и пропусков, а затем при необходимости пересмотрите свой план ответа.

- [Публикация NIST — руководства по тестированию, обучению и выполнению упражнений для ИТ-планов и возможностей](https://csrc.nist.gov/publications/detail/sp/800-84/final)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5. Включение оповещений системы безопасности в систему реагирования на инциденты

**Руководство**. Экспорт оповещений и рекомендаций центра безопасности Azure с помощью функции непрерывного экспорта для выявления рисков ресурсов Azure. Непрерывный экспорт позволяет экспортировать предупреждения и рекомендации как вручную, так и в постоянном, непрерывном режиме. Вы можете использовать соединитель данных центра безопасности Azure для потоковой передачи оповещений в Azure Sentinel.

- [Настройка непрерывного экспорта данных](../../security-center/continuous-export.md)

- [Как выполнить потоковую передачу оповещений в Azure Sentinel](../../sentinel/connect-azure-security-center.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="106-automate-the-response-to-security-alerts"></a>10.6. Автоматизация реагирования на оповещения системы безопасности

**Руководство**. Использование функции автоматизации рабочих процессов центр безопасности Azure для автоматического запуска ответов на оповещения системы безопасности и рекомендации по защите ресурсов Azure.

- [Настройка автоматизации рабочих процессов в центре безопасности](../../security-center/workflow-automation.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="penetration-tests-and-red-team-exercises"></a>Тесты на проникновение и попытки нарушения безопасности "красной командой"

*Дополнительные сведения см. в статье [тесты производительности системы безопасности Azure: испытания на проникновение и команды красных команд](../../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11,1. Проведите регулярное тестирование на проникновение ресурсов Azure и обеспечьте исправление всех критических результатов безопасности.

**Рекомендации**. Следуйте правилам тестирования уязвимости Microsoft Cloud, чтобы убедиться, что тесты на проникновение не нарушают политики Майкрософт. Используйте стратегию Майкрософт и рекомендации "красных команд", а затем выполните тест на проникновение в режиме реального времени для управляемых корпорацией Майкрософт облачной инфраструктуры, служб и приложений.

- [Правила взаимодействия при выполнении тестирования на проникновение](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

- [Привлечение "красных команд для тестирования "Microsoft Cloud](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Ответственность**: Совмещаемая блокировка

**Мониторинг центра безопасности Azure**: нет

## <a name="next-steps"></a>Следующие шаги

- См. [Обзор Azure Security Benchmark версии 2](/azure/security/benchmarks/overview)
- Дополнительные сведения о [базовой конфигурации безопасности Azure](/azure/security/benchmarks/security-baselines-overview).
