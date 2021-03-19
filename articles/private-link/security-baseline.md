---
title: Базовый канал безопасности Azure для частной связи Azure
description: В базовом плане безопасности частной связи Azure приводятся процедуры и ресурсы для реализации рекомендаций по безопасности, указанных в статье о производительности системы безопасности Azure.
author: msmbaldwin
ms.service: private-link
ms.topic: conceptual
ms.date: 03/16/2021
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 8fcbfa745a78d335606c11dcbd8851a235407b9d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104588534"
---
# <a name="azure-security-baseline-for-azure-private-link"></a>Базовый канал безопасности Azure для частной связи Azure

Этот базовый план безопасности применяет руководство из [теста безопасности Azure с версией 1,0](../security/benchmarks/overview-v1.md) в частную ссылку Azure. Azure Security Benchmark содержит рекомендации по обеспечению безопасности облачных решений в Azure.
Содержимое группируются по **элементам управления безопасностью** , определенным в тестовом производительности системы безопасности Azure, и связанным рекомендациям, применимым к частной ссылке Azure. **Элементы управления** , неприменимые к закрытой ссылке Azure, были исключены.

 
Сведения о том, как частная связь Azure полностью сопоставляется с тестовым показателем безопасности Azure, см. в [полном файле сопоставления базовых показателей безопасности для частного канала Azure](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Безопасность сети

*Дополнительные сведения см. в статье [Azure Security Benchmark: безопасность сети](../security/benchmarks/security-control-network-security.md).*

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11. Использование автоматизированных средств для мониторинга конфигураций сетевых ресурсов и обнаружения изменений

**Руководство**. Использование журнала действий Azure для мониторинга конфигураций ресурсов и обнаружения изменений в сетевых ресурсах, связанных с частной ссылкой. 

Создавайте оповещения в Azure Monitor, которые будут запускаться при изменении критических ресурсов.

- [Как просматривать и извлекать события журнала действий Azure](https://docs.microsoft.com/azure/azure-monitor/essentials/activity-log#view-the-activity-log)

- [Как создать оповещения в службе Azure Monitor](../azure-monitor/alerts/alerts-activity-log.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="logging-and-monitoring"></a>Ведение журналов и мониторинг

*Дополнительные сведения см. в статье [производительность системы безопасности Azure: ведение журнала и мониторинг](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="22-configure-central-security-log-management"></a>2.2. Настройка централизованного управления журналами безопасности

**Руководство**. прием журналов с помощью Azure Monitor для агрегирования данных безопасности, создаваемых сетевыми ресурсами, такими как конечные точки частной связи, виртуальные сети и группы безопасности сети. 

В Azure Monitor используйте рабочие области Log Analytics для запроса и выполнения анализа и используйте учетные записи хранения Azure для долгосрочного и архивного хранения.

Кроме того, включите и подключите данные к Azure Sentinel или сторонним SIEM, основываясь на бизнес-требованиях Организации.

- [Подключение к Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Как получить журналы и метрики платформы с помощью Azure Monitor](../azure-monitor/essentials/diagnostic-settings.md)

- [Ведение журнала и мониторинг для закрытой ссылки](https://docs.microsoft.com/azure/private-link/private-link-overview#logging-and-monitoring)

- [Журнал ведения диагностики для группы безопасности сети](../virtual-network/virtual-network-nsg-manage-log.md)

- [Начало работы с Azure Monitor и интеграция SIEM стороннего производителя](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3. Включение журналов аудита для ресурсов Azure

**Руководство**. Включение журналов действий Azure Monitor, которые записываются в журнал операции с частными ресурсами, такие как, кто запустил операцию, состояние операции и другие полезные данные аудита. 

- [Как получить журналы и метрики платформы с помощью Azure Monitor](../azure-monitor/essentials/diagnostic-settings.md)

- [Как просматривать и извлекать события журнала действий Azure](https://docs.microsoft.com/azure/azure-monitor/essentials/activity-log#view-the-activity-log)

- [Ведение журнала и мониторинг для закрытой ссылки](https://docs.microsoft.com/azure/private-link/private-link-overview#logging-and-monitoring)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="25-configure-security-log-storage-retention"></a>2.5. Настройка хранения журнала безопасности

**Руководство**. для журналов, связанных с частной ссылкой, задайте срок хранения log Analytics рабочей области в соответствии с правилами соответствия вашей организации в Azure Monitor. Используйте учетные записи хранения Azure для долгосрочного и архивного хранения журналов.

- [Изменение срока хранения данных в Log Analytics](https://docs.microsoft.com/azure/azure-monitor/logs/manage-cost-storage#change-the-data-retention-period)

- [Настройка политики хранения для журналов учетных записей хранения Azure](https://docs.microsoft.com/azure/storage/common/manage-storage-analytics-logs#configure-logging)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="26-monitor-and-review-logs"></a>2,6: мониторинг и просмотр журналов

**Руководство**. анализ и мониторинг журналов для аномального поведения и регулярная проверка результатов. Используйте рабочую область Log Analytics Azure Monitor, чтобы просматривать журналы и выполнять запросы к данным журнала.

Другим вариантом является включение и встроенные данные в Azure Sentinel или сторонние SIEM.

- [Подключение к Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Общие сведения о рабочей области Log Analytics](../azure-monitor/logs/log-analytics-tutorial.md)

- [Выполнение пользовательских запросов в Azure Monitor](../azure-monitor/logs/get-started-queries.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="27-enable-alerts-for-anomalous-activities"></a>2,7: Включение оповещений для аномальных действий

**Руководство**. Использование центра безопасности с рабочей областью log Analytics для мониторинга и оповещения о аномальных действиях, обнаруженных в журналах и событиях безопасности.

Включение и встроенные данные в Azure Sentinel или сторонние SIEM на основе бизнес-требований Организации.

- [Подключение к Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Управление оповещениями в центре безопасности Azure](../security-center/security-center-managing-and-responding-alerts.md)

- [Как оповещать данные журнала Log Analytics](../azure-monitor/alerts/tutorial-response.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="identity-and-access-control"></a>Идентификатор и управление доступом

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: идентификация и управление доступом](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1. Инвентаризация учетных записей администраторов

**Руководство**. Использование встроенных ролей администратора Azure Active Directory (Azure AD), которые могут быть явно назначены и запрошены. Запустите модуль Azure AD PowerShell, чтобы выполнить нерегламентированные запросы для обнаружения учетных записей, входящих в группы администраторов.

- [Как получить роль каталога в Azure AD с помощью PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0&amp;preserve-view=true)

- [Как получить членов роли каталога в Azure AD с помощью PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0&amp;preserve-view=true)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="33-use-dedicated-administrative-accounts"></a>3.3. Применение выделенных административных учетных записей

**Руководство**. Создайте стандартные операционные процедуры для использования выделенных административных учетных записей. Используйте функции управления удостоверениями и доступом центра безопасности для наблюдения за количеством административных учетных записей.

Кроме того, включите JIT-доступ или достаточно просто, используя Azure Active Directory (Azure AD) управление привилегированными пользователями привилегированные роли для служб Майкрософт и Azure Resource Manager.

- [Дополнительные сведения о управление привилегированными пользователями](/azure/active-directory/privileged-identity-management/)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="34-use-azure-active-directory-single-sign-on-sso"></a>3,4. Использование единого входа Azure Active Directory (SSO)

**Рекомендации**. когда это возможно, используйте единый вход с Azure Active Directory (Azure AD) вместо настройки отдельных автономных учетных данных для каждой службы.

- [Единый вход в приложения в Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

- [Мониторинг идентификации и доступа в Центре безопасности Azure](../security-center/security-center-identity-access.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3,5: используйте многофакторную проверку подлинности для всех Azure Active Directory доступа

**Руководство**. Включение многофакторной проверки подлинности Azure Active Directory (Azure AD) и следование рекомендациям по управлению удостоверениями и доступом в центре безопасности.

- [Как включить многофакторную проверку подлинности в Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Мониторинг идентификации и доступа в Центре безопасности Azure](../security-center/security-center-identity-access.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="36-use-secure-azure-managed-workstations-for-administrative-tasks"></a>3,6: используйте защищенные рабочие станции, управляемые Azure, для административных задач

**Руководство**. Использование рабочей станции привилегированного доступа (привилегированным доступом) с многофакторной проверкой подлинности, настроенной для входа и настройки сетевых ресурсов Azure.

- [Использование рабочих станций с привилегированным доступом](https://4sysops.com/archives/understand-the-microsoft-privileged-access-workstation-paw-security-model/)

- [Как включить многофакторную проверку подлинности в Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3,7: журналы и оповещения о подозрительных действиях учетных записей администраторов

**Руководство**. Использование функции обнаружения рисков Azure Active Directory (Azure AD) для просмотра предупреждений и отчетов о небезопасном поведении пользователей. 

Получение оповещений центра безопасности об обнаружении рисков в Azure Monitor и настройка пользовательских оповещений и уведомлений с помощью групп действий.

- [Основные сведения об обнаружении рисков в центре безопасности Azure (подозрительные действия)](../active-directory/identity-protection/overview-identity-protection.md)

- [Как интегрировать журналы действий Azure в Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

- [Настройка групп действий для настраиваемых оповещений и уведомлений](../azure-monitor/alerts/action-groups.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8. Управление ресурсами Azure только из утвержденных расположений

**Руководство**. Используйте условный доступ с именованными расположениями, чтобы разрешить доступ только из конкретных логических групп диапазонов IP-адресов или стран и регионов.

- [Настройка именованных расположений в Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="39-use-azure-active-directory"></a>3.9. Использование Azure Active Directory

**Руководство**. Использование Azure Active Directory (Azure AD) в качестве централизованной системы проверки подлинности и авторизации. Azure AD защищает данные с помощью надежного шифрования для неактивных и транзитных данных, а также для использования случайных значений, хэшей и безопасного хранения учетных данных пользователя.  

- [Создание и настройка экземпляра Azure AD](../active-directory-domain-services/tutorial-create-instance.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10. Регулярная проверка и согласование доступа пользователей

**Руководство**. Azure Active Directory (Azure AD) предоставляет журналы для облегчения поиска устаревших учетных записей. Кроме того, используйте проверку доступа удостоверений Azure, чтобы эффективно управлять членством в группах, доступом к корпоративным приложениям и назначениями ролей. Доступ пользователей можно проверить регулярно, чтобы убедиться, что доступ к ним имеют только нужные пользователи.

- [Общие сведения об отчетах Azure AD](/azure/active-directory/reports-monitoring/)

- [Использование проверок доступа для идентификации Azure](../active-directory/governance/access-reviews-overview.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3,11: монитор пытается получить доступ к отключенным учетным данным

**Руководство**. Использование источников журналов событий входа в систему Azure Active Directory (Azure AD), аудита и риска для интеграции с любым средством SIEM/Monitoring.

Оптимизируйте этот процесс, создав параметры диагностики для учетных записей пользователей Azure AD и отправив журналы аудита и журналы входа в рабочую область Log Analytics. Настройте нужные оповещения в Log Analytics рабочей области.

- [Как интегрировать журналы действий Azure в Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3,12: предупреждение об отклонении при входе в учетную запись

**Руководство**. Использование Azure Active Directory (Azure AD) рисков и средств защиты идентификации для настройки автоматических ответов на обнаруженные подозрительные действия, связанные с удостоверениями пользователей для сетевых ресурсов. 

Прием данных в Azure Sentinel для дальнейшего изучения.

- [Просмотр рискованных входов в Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Как настроить и включить политики рисков с помощью защиты идентификации](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Подключение к Azure Sentinel](../sentinel/quickstart-onboard.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="data-protection"></a>Защита данных

*Дополнительные сведения см. в статье [Azure Security Benchmark: защита данных](../security/benchmarks/security-control-data-protection.md).*

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2. Изолирование систем, хранящих или обрабатывающих конфиденциальные данные

**Руководство**. Реализация изоляции с помощью отдельных подписок и групп управления для отдельных доменов безопасности, таких как тип среды и уровень конфиденциальности данных.

Ограничьте уровень доступа к ресурсам Azure с приложениями и корпоративными средами в соответствии с бизнес-требованиями.

Управление доступом к ресурсам Azure с помощью управления доступом на основе ролей Azure (Azure AD).

- [Создание дополнительных подписок Azure](../cost-management-billing/manage/create-subscription.md)

- [Создание групп управления](../governance/management-groups/create-management-group-portal.md)

- [Создание и использование тегов](../azure-resource-manager/management/tag-resources.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4. Шифрование любой конфиденциальной информации при передаче

**Руководство**. шифрование всех конфиденциальных данных во время передачи. Убедитесь, что все клиенты, подключающиеся к ресурсам Azure в виртуальных сетях, могут согласовать TLS 1,2 или более позднюю. Частная ссылка не мешает основным протоколам. Используйте рекомендации для других предложений, используемых клиентами.

Следуйте рекомендациям центра безопасности для шифрования неактивных данных и шифрования при передаче, где это применимо.

- [Общие сведения о шифровании при передаче с помощью Azure](https://docs.microsoft.com/azure/security/fundamentals/encryption-overview#encryption-of-data-in-transit)

**Ответственность**: Совмещаемая блокировка

**Мониторинг центра безопасности Azure**: нет

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6. Контроль доступа к ресурсам с помощью RBAC 

**Руководство**. Использование управления доступом на основе ролей Azure (Azure RBAC) для управления доступом к данным и ресурсам. в противном случае используйте специфические для службы методы управления доступом.

- [Ознакомьтесь с кратким описанием и уникальным ИДЕНТИФИКАТОРом встроенных ролей Azure.](../role-based-access-control/built-in-roles.md)

Чтобы получить список ролей в среде, вызовите PowerShell cmd "Get-Азроледефинитион" или "az Role Definition List".

Просмотрите параметры, чтобы управлять раскрытием службы с помощью параметра "видимость". в закрытой ссылке. Эти параметры видимости решают, может ли потребитель подключаться к вашей службе. 

Сделайте службу частной для использования из других виртуальных сетей (только разрешения RBAC Azure). Ограничьте уязвимость ограниченным набором доверенных подписок или сделайте ее общедоступной, чтобы все подписки Azure могли запрашивать подключения к службе Private Link.

- [Настройка Azure RBAC](../role-based-access-control/role-assignments-portal.md)

- [Свойства для службы закрытых ссылок](private-link-service-overview.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9. Включение в журнал и создание оповещений по изменениям критических ресурсов Azure

**Руководство**. Использование Azure Monitor оповещений журнала действий для создания оповещений о том, когда изменения выполняются с важными ресурсами Azure, такими как службы и конечные точки частной связи. 

- [Журнал ведения диагностики для группы безопасности сети](https://docs.microsoft.com/azure/private-link/private-link-overview#logging-and-monitoring)

- [Создание оповещений для событий журнала действий Azure](../azure-monitor/alerts/alerts-activity-log.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="inventory-and-asset-management"></a>Инвентаризация и управление ресурсами

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: Инвентаризация и управление активами](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6,1. Использование автоматизированного решения для обнаружения ресурсов

**Руководство**. Использование графа ресурсов Azure для запроса и обнаружения всех сетевых ресурсов, таких как службы Private Link и конечные точки в подписках.

Убедитесь, что у вас есть соответствующие разрешения (на чтение) в клиенте и вы можете перечислить все подписки Azure, а также ресурсы в ваших подписках.

- [Как создавать запросы с помощью Azure Graph](../governance/resource-graph/first-query-portal.md)

- [Как просматривать подписки Azure](https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-4.8.0&amp;preserve-view=true)

- [Общие сведения об Azure RBAC](../role-based-access-control/overview.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="62-maintain-asset-metadata"></a>6.2. Ведение метаданных активов

**Руководство**. применение тегов к ресурсам Azure с помощью метаданных для логической организации их в таксономию.

- [Создание и использование тегов](../azure-resource-manager/management/tag-resources.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="63-delete-unauthorized-azure-resources"></a>6.3. Удаление неавторизованных ресурсов Azure

**Руководство**. Использование тегов, групп управления и отдельных подписок (при необходимости) для Организации и отслеживания закрытых ссылок и связанных ресурсов. 

Регулярно сверяйте ресурсы, чтобы своевременно удалять неавторизованные ресурсы из подписки.

- [Создание дополнительных подписок Azure](../cost-management-billing/manage/create-subscription.md)

- [Создание групп управления](../governance/management-groups/create-management-group-portal.md)

- [Создание и использование тегов](../azure-resource-manager/management/tag-resources.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6,4: определение и обслуживание инвентаризации утвержденных ресурсов Azure

**Руководство**. Создание инвентаризации утвержденных ресурсов и программного обеспечения Azure для ресурсов вычислений в соответствии с потребностями Организации.

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5. Отслеживание неутвержденных ресурсов Azure

**Руководство**. Использование политики Azure для ограничения типа ресурсов, которые могут быть созданы в клиентских подписках, с помощью следующих встроенных определений политик:

- Недопустимые типы ресурсов

- Допустимые типы ресурсов

Кроме того, используйте граф ресурсов Azure для запроса или обнаружения ресурсов в подписках. Это может помочь в средах с высоким уровнем безопасности, например с учетными записями хранения.

- [Настройка Политики Azure и управление ею](../governance/policy/tutorials/create-and-manage.md)

- [Как создавать запросы с помощью Azure Graph](../governance/resource-graph/first-query-portal.md)

- [Пример политики Azure "встроенные" для закрытой ссылки](../governance/policy/samples/built-in-policies.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="67-remove-unapproved-azure-resources"></a>6,7: удаление неутвержденных ресурсов Azure

**Руководство**. Клиент может препятствовать созданию или использованию ресурсов политикой Azure согласно требованиям политик компании клиента. Вы можете реализовать собственный процесс для удаления несанкционированных ресурсов.

- [Настройка Политики Azure и управление ею](../governance/policy/tutorials/create-and-manage.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="69-use-only-approved-azure-services"></a>6.9. Использование только утвержденных служб Azure

**Руководство**. Использование политики Azure для ограничения типа ресурсов, которые могут быть созданы в клиентских подписках, с помощью следующих встроенных определений политик:
- Недопустимые типы ресурсов
- Допустимые типы ресурсов

- [Настройка Политики Azure и управление ею](../governance/policy/tutorials/create-and-manage.md)

- [Как отказаться от определенного типа ресурса с помощью Политики Azure](https://docs.microsoft.com/azure/governance/policy/samples/built-in-policies#general)

- [Пример политики Azure "встроенные" для закрытой ссылки](../governance/policy/samples/built-in-policies.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6,11: ограничьте возможность пользователей работать с Azure Resource Manager

**Руководство**. Используйте условный доступ Azure, чтобы ограничить возможность пользователей взаимодействовать с Azure Resource Manager путем настройки "Блокировать доступ" для приложения "Управление Microsoft Azure".

- [Как настроить условный доступ для блокировки доступа к Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="secure-configuration"></a>Настройка безопасности

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: безопасная конфигурация](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1. Установка безопасных конфигураций для всех ресурсов Azure

**Руководство**. Использование псевдонимов политик Azure для создания настраиваемых политик для аудита или принудительного применения конфигурации сетевых ресурсов Azure вместе со встроенными определениями политик Azure.

Azure Resource Manager может экспортировать шаблон в нотация объектов JavaScript (JSON). Этот артефакт следует проверить, чтобы убедиться, что конфигурации соответствуют требованиям безопасности Организации или превышают их.

Реализуйте рекомендации из центра безопасности в качестве защищенного шаблона базовой конфигурации для ресурсов Azure.

- [Просмотр доступных псевдонимов политик Azure](https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-4.8.0&amp;preserve-view=true)

- [Руководство по Создание политик и управление ими для обеспечения соответствия требованиям](../governance/policy/tutorials/create-and-manage.md)

- [Пример политики Azure "встроенные" для закрытой ссылки](../governance/policy/samples/built-in-policies.md)

- [Экспорт одного и нескольких ресурсов в шаблон в портал Azure](../azure-resource-manager/templates/export-template-portal.md)

- [Рекомендации по безопасности — справочное руководство](../security-center/recommendations-reference.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3. Сохранение безопасных конфигураций для ресурсов Azure

**Руководство**. использование шаблонов Azure Resource Manager и политики Azure для безопасной настройки ресурсов Azure, связанных с частными ссылками и связанными ресурсами.  

Azure Resource Manager шаблоны — это файлы на основе нотация объектов JavaScript (JSON), используемые для развертывания ресурсов Azure. в репозитории кода необходимо безопасно хранить и поддерживать все пользовательские шаблоны. 

Используйте политику Azure [отказывать] и [развернуть, если не существует], чтобы обеспечить безопасность параметров в ресурсах Azure.

- [Сведения о создании шаблонов Azure Resource Manager](../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md)

- [Настройка Политики Azure и управление ею](../governance/policy/tutorials/create-and-manage.md)

- [Действия политик Azure](../governance/policy/concepts/effects.md)

- [Примеры шаблонов Azure Resource Manager для частных конечных точек](/azure/templates/microsoft.network/2019-11-01/privateendpoints)

- [Пример политики Azure "встроенные" для закрытой ссылки](../governance/policy/samples/built-in-policies.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5. Безопасное хранение конфигурации ресурсов Azure

**Руководство**. Использование Azure DevOps для безопасного хранения и управления кодом, например пользовательскими политиками Azure, Azure Resource Manager шаблонами и скриптами настройки требуемого состояния.

Предоставьте или запретите разрешения определенным пользователям, встроенным группам безопасности или группам, определенным в Azure Active Directory (Azure AD), если они интегрированы с Azure DevOps для доступа, или Active Directory, если они интегрированы с Team Foundation Server.

- [Как хранить код в Azure DevOps](https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops&amp;preserve-view=true)

- [О разрешениях и группах в Azure DevOps](/azure/devops/organizations/security/about-permissions)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7,7: развертывание средств управления конфигурацией для ресурсов Azure

**Руководство**. Определение и реализация стандартных конфигураций безопасности для ресурсов Azure с помощью политики Azure. 

Используйте псевдонимы политик Azure для создания настраиваемых политик для аудита или принудительного применения конфигурации сети ресурсов Azure. 

Также используйте встроенные определения политик, относящиеся к конкретным ресурсам, управляемым в рамках ваших подписок.

- [Настройка Политики Azure и управление ею](../governance/policy/tutorials/create-and-manage.md)

- [Использование псевдонимов](https://docs.microsoft.com/azure/governance/policy/concepts/definition-structure#aliases)

- [Пример политики Azure "встроенные" для закрытой ссылки](../governance/policy/samples/built-in-policies.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7,9. Реализация автоматического мониторинга конфигурации для ресурсов Azure

**Руководство**. Использование политики Azure для аудита конфигураций ресурсов Azure. Например, политика Azure может использоваться для обнаружения ресурсов, которые не настроены с помощью частной конечной точки.

- [Настройка Политики Azure и управление ею](../governance/policy/tutorials/create-and-manage.md)

- [Пример политики Azure "встроенные" для закрытой ссылки](../governance/policy/samples/built-in-policies.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="data-recovery"></a>Восстановление данных

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: восстановление данных](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9,1: Обеспечьте регулярное автоматическое резервное копирование

**Руководство**. Использование Azure Resource Manager для развертывания служб Private Link, конечных точек и связанных ресурсов. Azure Resource Manager предоставляет возможность экспорта шаблонов, которые можно использовать в качестве резервных копий для восстановления конечных точек частной связи и связанных ресурсов. 

Используйте службу автоматизации Azure для регулярного вызова API-интерфейса экспорта шаблона Azure Resource Manager.

- [Общие сведения об Azure Resource Manager](../azure-resource-manager/management/overview.md)

- [Примеры шаблонов Azure Resource Manager для частных конечных точек](/azure/templates/microsoft.network/2019-11-01/privateendpoints)

- [Экспорт одного и нескольких ресурсов в шаблон в портал Azure](../azure-resource-manager/templates/export-template-portal.md)

- [Группы ресурсов — Экспорт шаблона](/rest/api/resources/resourcegroups/exporttemplate)

- [Введение в службу автоматизации Azure](../automation/automation-intro.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9,2: выполните полное резервное копирование системы и резервное копирование ключей, управляемых клиентом

**Руководство**. Использование Azure Resource Manager для развертывания служб Private Link, конечных точек и связанных ресурсов. Azure Resource Manager предоставляет возможность экспорта шаблонов, которые можно использовать в качестве резервных копий для восстановления конечных точек частной связи и связанных ресурсов.

Используйте службу автоматизации Azure для регулярного вызова API-интерфейса экспорта шаблона Azure Resource Manager.

Резервное копирование ключей, управляемых клиентом, в Azure Key Vault.

- [Общие сведения об Azure Resource Manager](../azure-resource-manager/management/overview.md)

- [Примеры шаблонов Azure Resource Manager для частных конечных точек](/azure/templates/microsoft.network/2019-11-01/privateendpoints)

- [Экспорт одного и нескольких ресурсов в шаблон в портал Azure](../azure-resource-manager/templates/export-template-portal.md)

- [Группы ресурсов — Экспорт шаблона](/rest/api/resources/resourcegroups/exporttemplate)

- [Введение в службу автоматизации Azure](../automation/automation-intro.md)

- [Как создать резервную копию ключей хранилища ключей в Azure](https://docs.microsoft.com/powershell/module/az.keyvault/backup-azkeyvaultkey?view=azps-4.8.0&amp;preserve-view=true)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9,3: Проверьте все резервные копии, включая управляемые клиентом ключи.

**Руководство**. Проверка возможности периодически выполнять развертывание шаблонов Azure Resource Manager в изолированной подписке в соответствии с бизнес-требованиями.

Кроме того, проверьте возможность восстановления резервных копий ключей, управляемых клиентом.

- [Развертывание ресурсов с помощью шаблонов ARM и портал Azure](../azure-resource-manager/templates/deploy-portal.md)

- [Как восстановить резервную копию ключей хранилища ключей в Azure](https://docs.microsoft.com/powershell/module/az.keyvault/restore-azkeyvaultkey?view=azps-4.8.0&amp;preserve-view=true)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9,4: Обеспечьте защиту резервных копий и ключей, управляемых клиентом

**Руководство**. Использование Azure DevOps для безопасного хранения кода, например Azure Resource Manager шаблонов, и управления им.

Предоставьте или запретите разрешения для определенных пользователей, встроенных групп безопасности или групп, определенных в Azure Active Directory (Azure AD), если они интегрированы с Azure DevOps для доступа к этим ресурсам или в Active Directory, если они интегрированы с Team Foundation Server.

- [Как хранить код в Azure DevOps](https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops&amp;preserve-view=true)

- [О разрешениях и группах в Azure DevOps](/azure/devops/organizations/security/about-permissions)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="incident-response"></a>реагирование на инциденты.

*Дополнительные сведения см. в статье [Azure Security Benchmark: реагирование на инциденты](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1. Создание руководства по реагированию на инциденты

**Руководство**. Создайте руководство по реагированию на инциденты для вашей организации. 

Следите за письменными планами реагирования на инциденты, которые определяют все роли персонала и фазы обработки инцидентов или управления от обнаружения до проверки после инцидента.

- [Руководство по созданию собственного процесса реагирования на инциденты безопасности](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Анатомия инцидента Центра Майкрософт по реагированию на угрозы](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)

- [Клиент также может воспользоваться руководством по обработке инцидентов безопасности компьютера NIST, чтобы помочь в создании собственного плана реагирования на инциденты.](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2. Создание процедуры оценки инцидента и определения приоритетов

**Руководство**. Центр безопасности назначает серьезность каждому оповещению, чтобы определить, какие оповещения следует изучить первыми. Серьезность основывается на том, насколько надежным является центр безопасности в поиске или аналитике, используемой для выдаче оповещения, а также об уровне достоверности, который был вредоносным намерением для действия, вызвавшего оповещение.

Четко Пометьте подписки (например, Рабочий, нерабочий) с помощью тегов и создайте систему именования для четкого обнаружения и классификации ресурсов Azure, особенно для обработки конфиденциальных данных.  

Клиент несет ответственность за исправление предупреждений в зависимости от степени важности ресурсов и среды Azure, в которых произошел инцидент.

- [Оповещения безопасности в Центре безопасности Azure](../security-center/security-center-alerts-overview.md)

- [использование тегов для упорядочения ресурсов в Azure](../azure-resource-manager/management/tag-resources.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="103-test-security-response-procedures"></a>10.3. Проверка процедур реагирования на угрозы

**Руководство**. Выполняйте упражнения, чтобы периодически тестировать возможности ваших систем реагировать на угрозы для защиты ресурсов Azure. Выявите слабые точки и пробелов и пересмотрите план по мере необходимости.

- [Публикация NIST — руководство по тестированию, обучению и выполнению программ для ИТ-планов и возможностей](https://csrc.nist.gov/publications/detail/sp/800-84/final)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4. Предоставление контактных сведений и настройка уведомлений по инцидентам безопасности

**Руководство**. Корпорация Майкрософт будет использовать информацию об инциденте безопасности для связи с вами, если центр Microsoft Security Response Center (MSRC) обнаружит, что к вашим данным был получен незаконный или несанкционированный доступ. 

Проверьте инциденты, POST, чтобы убедиться, что проблемы устранены.

- [Как задать контакт безопасности Центра безопасности Azure](../security-center/security-center-provide-security-contact-details.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5. Включение оповещений системы безопасности в систему реагирования на инциденты

**Руководство**. Экспорт оповещений и рекомендаций центра безопасности с помощью функции непрерывного экспорта для выявления рисков ресурсов Azure. Непрерывный экспорт позволяет экспортировать предупреждения и рекомендации как вручную, так и в постоянном, непрерывном режиме. 

Кроме того, используйте соединитель данных центра безопасности для потоковой передачи оповещений в Azure Sentinel, как это необходимо для бизнес-операций.

- [Настройка непрерывного экспорта данных](../security-center/continuous-export.md)

- [Как выполнить потоковую передачу оповещений в Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="106-automate-the-response-to-security-alerts"></a>10.6. Автоматизация реагирования на оповещения системы безопасности

**Руководство**. Использование функции автоматизации рабочих процессов в центре безопасности для автоматического запуска ответов с Logic Apps оповещений системы безопасности и рекомендаций по защите ресурсов Azure.

- [Как настроить автоматизацию рабочего процесса и Logic Apps](../security-center/workflow-automation.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="penetration-tests-and-red-team-exercises"></a>Тесты на проникновение и попытки нарушения безопасности "красной командой"

*Дополнительные сведения см. в статье [тесты производительности системы безопасности Azure: испытания на проникновение и команды красных команд](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11,1. Проведите регулярное тестирование на проникновение ресурсов Azure и обеспечьте исправление всех критических результатов безопасности.

**Руководство**. соблюдение правил участия в корпорации Майкрософт, чтобы убедиться, что тесты на проникновение не нарушают политики Майкрософт. 

Используйте стратегию и исполнение Microsoft, а затем протестировать проникновение в реальном времени для управляемой корпорацией Майкрософт облачной инфраструктуры, служб и приложений.

- [Правила взаимодействия при выполнении тестирования на проникновение](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

- [Привлечение "красных команд для тестирования "Microsoft Cloud](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Ответственность**: Совмещаемая блокировка

**Мониторинг центра безопасности Azure**: нет

## <a name="next-steps"></a>Следующие шаги

- См. [Обзор Azure Security Benchmark версии 2](/azure/security/benchmarks/overview)
- Дополнительные сведения о [базовой конфигурации безопасности Azure](/azure/security/benchmarks/security-baselines-overview).
