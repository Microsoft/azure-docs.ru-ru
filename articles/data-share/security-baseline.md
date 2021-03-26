---
title: Базовый план безопасности Azure для общей папки данных Azure
description: Базовый план безопасности общего ресурса данных Azure содержит руководство по использованию процедур и ресурсы для реализации рекомендаций по безопасности, указанных в статье о производительности системы безопасности Azure.
author: msmbaldwin
ms.service: data-share
ms.topic: conceptual
ms.date: 02/18/2021
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 6a804b6d6840b25993ad6e249305f531a818be32
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105559457"
---
# <a name="azure-security-baseline-for-azure-data-share"></a>Базовый план безопасности Azure для общей папки данных Azure

Этот базовый план безопасности применяет рекомендации из [теста безопасности Azure версии 1,0](../security/benchmarks/overview-v1.md) к Microsoft Azure общему ресурсу данных. Azure Security Benchmark содержит рекомендации по обеспечению безопасности облачных решений в Azure.
Содержимое группируются по **элементам управления безопасностью** , определенным в ходе тестирования системы безопасности Azure, и связанным рекомендациям, применимым к общему ресурсу данных Azure. **Элементы управления** , неприменимые к общему ресурсу данных Azure, были исключены.

 
Сведения о том, как общая папка данных Azure полностью сопоставлена с тестовым уровнем безопасности Azure, см. в [полном файле сопоставления базовых показателей безопасности общего ресурса Azure](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="logging-and-monitoring"></a>Ведение журналов и мониторинг

*Дополнительные сведения см. в статье [производительность системы безопасности Azure: ведение журнала и мониторинг](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="22-configure-central-security-log-management"></a>2.2. Настройка централизованного управления журналами безопасности

**Рекомендации**. прием журналов, связанных с общим ресурсом данных Azure, с помощью Azure Monitor для объединения данных безопасности, созданных устройствами конечных точек, сетевыми ресурсами и другими системами безопасности. В Azure Monitor используйте рабочие области Log Analytics для запроса и выполнения анализа и используйте учетные записи хранения Azure для долгосрочного и архивного хранения.

Кроме того, вы можете включить эти данные в Azure Sentinel или SIEM стороннего производителя.

- [Подключение к Azure Sentinel](../sentinel/quickstart-onboard.md) 

- [Как получить журналы и метрики платформы с помощью Azure Monitor](../azure-monitor/essentials/diagnostic-settings.md) 

- [Начало работы с Azure Monitor и интеграция SIEM стороннего производителя](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3. Включение журналов аудита для ресурсов Azure

**Руководство**. журналы действий, которые автоматически доступны, содержат все операции записи (размещение, публикация, удаление) для ресурсов общего доступа к данным Azure, за исключением операций чтения (Get). Журналы действий можно использовать для поиска ошибок при устранении неполадок или для мониторинга того, как пользователь в вашей организации изменил ресурс.

Включите журналы диагностики для общей папки данных Azure, в частности журналы диагностики для Микрософтдаташаресентшареснапшотслог &amp; микрософтдаташаререцеиведшареснапшотслог. Эти журналы позволяют записывать ключевые сведения, такие как время начала синхронизации, время окончания, состояние и другие сведения. Эти журналы могут быть критически важными для дальнейшего изучения инцидентов безопасности и проведения судебных упражнений.

- [Как получить журналы и метрики платформы с помощью Azure Monitor](../azure-monitor/essentials/diagnostic-settings.md) 

- [Общие сведения о ведении журналов и различных типах журналов в Azure](../azure-monitor/essentials/platform-logs-overview.md)

- [Настройка параметров диагностики для журнала действий Azure](../azure-monitor/essentials/activity-log.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="25-configure-security-log-storage-retention"></a>2.5. Настройка хранения журнала безопасности

**Руководство**. Убедитесь, что для всех учетных записей хранения log Analytics или рабочих областей, используемых для хранения журналов общего доступа к данным Azure, задан срок хранения журнала в соответствии с нормативными требованиями вашей организации.

- [Настройка срока хранения Log Analytics рабочей области](../azure-monitor/logs/manage-cost-storage.md)

- [Хранение журналов ресурсов в учетной записи хранения Azure](../azure-monitor/essentials/resource-logs.md#send-to-azure-storage)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="26-monitor-and-review-logs"></a>2.6. Мониторинг и просмотр журналов

**Руководство**. анализ и мониторинг журналов, связанных с общими ресурсами в данных Azure, для аномального поведения и регулярной проверки результатов. Используйте Azure Monitor и рабочую область Log Analytics для просмотра журналов и выполнения запросов к данным журнала.

Кроме того, вы можете включить и подключить данные к Azure Sentinel или сторонним SIEM.

- [Подключение к Azure Sentinel](../sentinel/quickstart-onboard.md) 

- [Приступая к работе с Log Analytics запросами](../azure-monitor/logs/log-analytics-tutorial.md)

- [Выполнение пользовательских запросов в Azure Monitor](../azure-monitor/logs/get-started-queries.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="27-enable-alerts-for-anomalous-activities"></a>2,7: Включение оповещений для аномальных действий

**Руководство**. Использование центра безопасности Azure с рабочей областью log Analytics для мониторинга и оповещения о аномальных действиях, обнаруженных в журналах и событиях безопасности. Кроме того, вы можете включить и подключить данные в Azure Sentinel.

- [Подключение к Azure Sentinel](../sentinel/quickstart-onboard.md) 

- [Управление оповещениями в центре безопасности Azure](../security-center/security-center-managing-and-responding-alerts.md) 

- [Как оповещать данные журнала Log Analytics](../azure-monitor/alerts/tutorial-response.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="identity-and-access-control"></a>Идентификатор и управление доступом

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: идентификация и управление доступом](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4. Использование единого входа с Azure Active Directory

**Руководство**. общий ресурс данных Azure поддерживает проверку подлинности SSO с помощью Azure Active Directory (Azure AD). Сократите количество удостоверений и учетных данных, которыми должен управлять пользователь, включив единый вход для службы с существующими удостоверениями вашей организации.

- [Общие сведения об использовании единого входа в Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5. Использование многофакторной проверки подлинности для любого доступа на основе Azure Active Directory

**Руководство**. Включение многофакторной проверки подлинности Azure AD и соблюдение рекомендаций по удостоверениям и доступу центра безопасности Azure.

- [Как включить многофакторную проверку подлинности в Azure](../active-directory/authentication/howto-mfa-getstarted.md) 

- [Мониторинг идентификации и доступа в Центре безопасности Azure](../security-center/security-center-identity-access.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6. Использование выделенных компьютеров (рабочих станций с привилегированным доступом) для всех административных задач

**Руководство**. используйте защищенную, управляемую Azure рабочую станцию (также называемую рабочей станцией привилегированного доступа или привилегированным доступом) для административных задач, требующих повышенных привилегий.

- [Общие сведения о защищенных рабочих станциях под управлением Azure](https://4sysops.com/archives/understand-the-microsoft-privileged-access-workstation-paw-security-model/)

- [Как включить многофакторную проверку подлинности Azure AD](../active-directory/authentication/howto-mfa-getstarted.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="39-use-azure-active-directory"></a>3.9. Использование Azure Active Directory

**Руководство**. Использование Azure Active Directory (Azure AD) в качестве централизованной системы проверки подлинности и авторизации. Azure AD защищает данные с помощью надежного шифрования для хранимых и транзитных данных. Кроме того, в Azure AD используются salt-записи, хэши и безопасное хранение учетных данных пользователей.

- [Создание и настройка экземпляра Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md) 

- [Общий ресурс Azure Data Share работает с общими встроенными ролями Azure. ](../role-based-access-control/built-in-roles.md#general)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10. Регулярная проверка и согласование доступа пользователей

**Руководство**. Azure AD предоставляет журналы для облегчения поиска устаревших учетных записей. Кроме того, используйте проверки подлинности и доступа Azure AD для эффективного управления членством в группах, доступом к корпоративным приложениям и назначениями ролей. Доступ пользователей можно проверить регулярно, чтобы убедиться, что доступ к ним имеют только нужные пользователи.

- [Общие сведения об отчетах Azure AD](../active-directory/reports-monitoring/index.yml) 

- [Использование проверок доступа для идентификации Azure AD](../active-directory/governance/access-reviews-overview.md) 

- [Общий ресурс Azure Data Share работает с общими встроенными ролями Azure. ](../role-based-access-control/built-in-roles.md#general)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3,11: монитор пытается получить доступ к отключенным учетным данным

**Руководство**. у вас есть доступ к источникам журналов событий входа в систему Azure Active Directory (Azure AD), аудита и риску, которые позволяют интегрироваться с любым средством SIEM/Monitoring.

Этот процесс можно упростить, создав параметры диагностики для учетных записей пользователей Azure AD и отправив журналы аудита и журналы входа в рабочую область Log Analytics. Вы можете настроить нужные оповещения в Log Analytics рабочей области.

- [Как интегрировать журналы действий Azure с Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12. Предупреждение при подозрительном входе в учетную запись

**Руководство**. Использование функций защиты идентификации Azure Active Directory (Azure AD) для настройки автоматических ответов на обнаруженные подозрительные действия, связанные с удостоверениями пользователей. Вы также можете включить данные в Azure Sentinel для дальнейшего изучения.

- [Просмотр рискованных входов в Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Как настроить и включить политики рисков с помощью защиты идентификации](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Подключение к Azure Sentinel](../sentinel/quickstart-onboard.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="data-protection"></a>Защита данных

*Дополнительные сведения см. в статье [Azure Security Benchmark: защита данных](../security/benchmarks/security-control-data-protection.md).*

### <a name="46-use-azure-rbac-to-manage-access-to-resources"></a>4,6. Использование Azure RBAC для управления доступом к ресурсам

**Руководство**. Использование управления доступом на основе ролей Azure (Azure RBAC) для управления доступом к данным и ресурсам, связанным с общими ресурсами данных Azure. в противном случае используйте специфические для службы методы управления доступом.

- [Настройка RBAC в Azure](../role-based-access-control/role-assignments-portal.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9. Включение в журнал и создание оповещений по изменениям критических ресурсов Azure

**Руководство**. Использование Azure Monitor с журналом действий Azure для создания Azure Monitor оповещений о том, когда изменения выполняются с важными ресурсами Azure.

- [Создание оповещений для событий журнала действий Azure](../azure-monitor/alerts/alerts-activity-log.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="vulnerability-management"></a>Управление уязвимостями

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: Управление уязвимостью](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1. Выполнение автоматизированных средства анализа уязвимостей

**Руководство**. общий ресурс данных Azure не разрешает установку средств проверки уязвимостей для своих ресурсов.

Как правило, следуйте рекомендациям центра безопасности Azure, чтобы выполнять оценку уязвимостей на виртуальных машинах Azure, в образах контейнеров и серверах SQL Server.

Используйте стороннее решение для проведения оценки уязвимостей на сетевых устройствах и веб-приложениях. При проведении удаленной проверки не используйте одну, бессрочную учетную запись администратора. Рассмотрите возможность реализации методологии JIT-подготовки для учетной записи проверки. Учетные данные для учетной записи проверки должны быть защищены, отслеживаться и использоваться только для поиска уязвимостей.

- [Реализация рекомендаций по оценке уязвимостей в центре безопасности Azure](../security-center/deploy-vulnerability-assessment-vm.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="inventory-and-asset-management"></a>Инвентаризация и управление ресурсами

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: Инвентаризация и управление активами](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6,1. Использование автоматизированного решения для обнаружения ресурсов

**Руководство**. вы применяете теги к ресурсам Azure, группам ресурсов и подпискам для логической организации их в таксономию. Каждый тег состоит из пары "имя — значение". Например, имя Environment и значение Production можно применить ко всем ресурсам в рабочей среде.

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="62-maintain-asset-metadata"></a>6.2. Ведение метаданных активов

**Руководство**. применение тегов к ресурсам Azure, группам ресурсов и подпискам для логической организации их в таксономию. Каждый тег состоит из пары "имя — значение". Например, имя Environment и значение Production можно применить ко всем ресурсам в рабочей среде.

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="63-delete-unauthorized-azure-resources"></a>6.3. Удаление неавторизованных ресурсов Azure

**Рекомендации**. Используйте теги, группы управления и отдельные подписки, если это необходимо, для Организации и мониторинга ресурсов. Регулярно сверяйте ресурсы, чтобы своевременно удалять неавторизованные ресурсы из подписки.

- [Создание дополнительных подписок Azure](../cost-management-billing/manage/create-subscription.md)

- [Создание групп управления](../governance/management-groups/create-management-group-portal.md)

- [Создание и использование тегов](../azure-resource-manager/management/tag-resources.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="64-define-and-maintain-an-inventory-of-approved-azure-resources"></a>6,4: определение и обслуживание инвентаризации утвержденных ресурсов Azure

**Руководство**. Создание инвентаризации утвержденных ресурсов Azure и утвержденного программного обеспечения для ресурсов вычислений в соответствии с потребностями Организации.

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5. Отслеживание неутвержденных ресурсов Azure

**Руководство**. Использование политики Azure для ограничения типа ресурсов, которые могут быть созданы в подписках.
Используйте граф ресурсов Azure для запроса и обнаружения ресурсов в своих подписках. Убедитесь в том, что все ресурсы Azure, представленные в среде, утверждены.

- [Настройка Политики Azure и управление ею](../governance/policy/tutorials/create-and-manage.md) 

- [Как создавать запросы с помощью Azure Graph](../governance/resource-graph/first-query-portal.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7. Удаление неутвержденных ресурсов Azure и программных приложений

**Руководство**. Удаление ресурсов Azure, если они больше не нужны, это можно сделать с помощью портал Azure, POWERSHELL или CLI.

- [Удаление группы ресурсов и ресурсов Azure](../azure-resource-manager/management/delete-resource-group.md?tabs=azure-powershell)

Общая папка данных Azure не предоставляет ОС или не позволяет устанавливать сторонние программные приложения для своих ресурсов.

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="69-use-only-approved-azure-services"></a>6.9. Использование только утвержденных служб Azure

**Руководство**. Использование политики Azure для ограничения служб, которые можно подготавливать в вашей среде.

- [Настройка Политики Azure и управление ею](../governance/policy/tutorials/create-and-manage.md) 

- [Как отказаться от определенного типа ресурса с помощью Политики Azure](../governance/policy/samples/built-in-policies.md#general)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6,11: ограничьте возможность пользователей работать с Azure Resource Manager

**Руководство**. Использование условного доступа Azure для ограничения возможности пользователей взаимодействовать с диспетчером ресурсов Azure путем настройки "блокировать доступ" для приложения "Microsoft Azure управления".

- [Настройка условного доступа для блокировки доступа к диспетчеру ресурсов Azure](../role-based-access-control/conditional-access-azure-management.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="secure-configuration"></a>Настройка безопасности

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: безопасная конфигурация](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5. Безопасное хранение конфигурации ресурсов Azure

**Руководство**. Использование Azure DevOps для безопасного хранения и управления кодом, например пользовательскими определениями политик Azure, Azure Resource Manager шаблонами и скриптами настройки требуемого состояния. Чтобы получить доступ к ресурсам, которыми вы управляете в Azure DevOps, вы можете предоставить или отклонить разрешения для определенных пользователей, встроенных групп безопасности или групп, определенных в Azure Active Directory (Azure AD), если они интегрированы с Azure DevOps, или Active Directory, если они интегрированы с TFS.

- [Как хранить код в Azure DevOps](/azure/devops/repos/git/gitworkflow)

- [О разрешениях и группах в Azure DevOps](/azure/devops/organizations/security/about-permissions)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7,7: развертывание средств управления конфигурацией для ресурсов Azure

**Руководство**. Использование псевдонимов политик Azure в пространстве имен "Share" для создания настраиваемых политик для оповещения, аудита и принудительного применения конфигураций системы. Кроме того, разрабатывайте процесс и конвейер для управления исключениями политик.

- [Настройка Политики Azure и управление ею](../governance/policy/tutorials/create-and-manage.md) 

- [Использование псевдонимов](../governance/policy/concepts/definition-structure.md#aliases)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="712-manage-identities-securely-and-automatically"></a>7.12. Безопасное и автоматическое управление удостоверениями

**Руководство**. Использование управляемых удостоверений для предоставления служб Azure с автоматически управляемым удостоверением в Azure Active Directory (Azure AD). Управляемые удостоверения позволяют проходить проверку подлинности в любой службе, поддерживающей проверку подлинности Azure AD, включая Key Vault без каких бы то ни было учетных данных в коде.

- [Настройка управляемых удостоверений для ресурсов Azure](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="next-steps"></a>Следующие шаги

- См. [Обзор Azure Security Benchmark версии 2](../security/benchmarks/overview.md)
- Дополнительные сведения о [базовой конфигурации безопасности Azure](../security/benchmarks/security-baselines-overview.md).