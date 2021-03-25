---
title: Базовый план безопасности Azure для Azure Monitor
description: Базовый план безопасности Azure Monitor содержит практические руководства и ресурсы для реализации рекомендаций по безопасности, указанных в статье о производительности системы безопасности Azure.
author: msmbaldwin
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 02/17/2021
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 86d660ef6133231172684f18ad3250392df5f509
ms.sourcegitcommit: a8ff4f9f69332eef9c75093fd56a9aae2fe65122
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105026127"
---
# <a name="azure-security-baseline-for-azure-monitor"></a>Базовый план безопасности Azure для Azure Monitor

Этот базовый план безопасности применяет рекомендации из [теста безопасности Azure версии 1,0](../security/benchmarks/overview-v1.md) к Azure Monitor. Azure Security Benchmark содержит рекомендации по обеспечению безопасности облачных решений в Azure.
Содержимое группируются по **элементам управления безопасностью** , определенным в производительности системы безопасности Azure, и связанным рекомендациям, применимым к Azure Monitor. **Элементы управления** , неприменимые к Azure Monitor, были исключены. 

 
Сведения о том, как Azure Monitor полностью сопоставлены с тестовым показателем безопасности Azure, см. в разделе [полное Azure Monitor файла сопоставления базовых показателей безопасности](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Безопасность сети

*Дополнительные сведения см. в статье [Azure Security Benchmark: безопасность сети](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1,1: защита ресурсов Azure в виртуальных сетях

**Руководство**. включение частного канала Azure для разрешения доступа к службам Azure SaaS (например, Azure Monitor) и размещенным в Azure клиентским или партнерским службам через частную конечную точку в виртуальной сети. Трафик между виртуальной сетью и службой проходит через магистральную сеть Майкрософт, что позволяет избежать рисков общедоступного Интернета.

Чтобы разрешить передачу трафика Azure Monitor, используйте теги службы "Азуремонитор", чтобы разрешить входящий и исходящий трафик через группы безопасности сети. Чтобы разрешить доступ к тестовому трафику мониторинга доступности Azure Monitor, используйте тег службы "Аппликатионинсигхтсаваилабилити" для всего входящего трафика через группы безопасности сети. Чтобы разрешить уведомление о достижении конечных точек клиента, используйте тег службы "ActionGroup", чтобы разрешить входящий трафик через группы безопасности сети.

Правила виртуальной сети позволяют Azure Monitor принимать только подключения, отправленные из выбранных подсетей в виртуальной сети.

Используйте шлюз Log Analytics для отправки данных в рабочую область Log Analytics в Azure Monitor от имени компьютеров, которые не могут напрямую подключаться к Интернету, предотвращая подключение компьютеров к Интернету. 

- [Настройка частного канала для Azure Monitor](./logs/private-link-security.md)

- [Подключение компьютеров без доступа к Интернету с помощью шлюза Log Analytics в Azure Monitor](./agents/gateway.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-network-interfaces"></a>1,2. Мониторинг и запись конфигурации и трафика виртуальных сетей, подсетей и сетевых интерфейсов

**Руководство**. Azure Monitor является основной службой и не поддерживает развертывание непосредственно в виртуальной сети, ее базовая инфраструктура обрабатывается корпорацией Майкрософт. Однако для ресурсов, которые делают сетевые подключения к предложению Azure Monitor, обеспечьте безопасность сети с помощью группы безопасности сети. Включите журналы потоков для групп безопасности сети и отправьте журналы в учетную запись хранения для аудита трафика. Вы также можете отправить журналы потоков в рабочую область Log Analytics и использовать Аналитика трафика для получения сведений о потоке трафика в облаке Azure. Некоторые преимущества Аналитики трафика — это возможность визуализировать сетевые активности и определять горячие участки, выявлять угрозы безопасности, анализировать шаблоны потоков трафика и выявлять некорректные сетевые настройки.

При использовании Azure Monitor с закрытой ссылкой вы получаете доступ к сетевому журналу, например "данные, обработанные закрытой конечной точкой (в/с)".

- [Требования к сети для агентов Azure Monitor](./agents/log-analytics-agent.md#network-requirements)

- [Подключение компьютеров без доступа к Интернету с помощью шлюза Log Analytics в Azure Monitor](./agents/gateway.md)

- [Включение журналов потоков для групп безопасности сети](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Как включить и использовать Аналитика трафика](../network-watcher/traffic-analytics.md)

- [Общие сведения о безопасности сети, предоставляемой центром безопасности Azure](../security-center/security-center-network-recommendations.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8. Уменьшение сложности и дополнительных затрат на администрирование в правилах безопасности сети

**Рекомендации**. чтобы разрешить трафик к Azure Monitor, используйте теги службы "азуремонитор", чтобы разрешить входящий и исходящий трафик через группы безопасности сети. Чтобы разрешить доступ к тестовому трафику мониторинга доступности Azure Monitor, используйте тег службы "Аппликатионинсигхтсаваилабилити" для всего входящего трафика через группы безопасности сети. Корпорация Майкрософт управляет префиксами адресов, входящих в тег службы, и автоматически обновляет этот тег при изменении адресов.

- [Общие сведения и использование тегов служб](../virtual-network/service-tags-overview.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="110-document-traffic-configuration-rules"></a>1.10. Документация по правилам конфигурации трафика

**Руководство**. Azure Monitor входит в состав служб ядра Azure и не может быть развернуто как услуга отдельно. Azure Monitor компоненты, включая агент Azure Monitor и пакет SDK для Application Insights, могут быть развернуты вместе с ресурсами, что может повлиять на безопасность этих ресурсов.

- [Требования к сети для агентов Azure Monitor](./agents/log-analytics-agent.md#network-requirements)

- [Подключение компьютеров без доступа к Интернету с помощью шлюза Log Analytics в Azure Monitor](./agents/gateway.md) 

- [См. раздел Приступая к работе с Application Insights](./app/app-insights-overview.md#get-started)

- [Настройка веб-тестов доступности](app/monitor-web-app-availability.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11. Использование автоматизированных средств для мониторинга конфигураций сетевых ресурсов и обнаружения изменений

**Руководство**. Использование журнала действий Azure для мониторинга конфигураций ресурсов и обнаружения изменений в сетевых ресурсах, связанных с Azure Monitor. Создавайте оповещения в Azure Monitor, которые будут запускаться при внесении изменений в эти критические сетевые ресурсы.

- [Как просматривать и извлекать события журнала действий Azure](./essentials/activity-log.md#view-the-activity-log)

- [Как создать оповещения в службе Azure Monitor](./alerts/alerts-activity-log.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="logging-and-monitoring"></a>Ведение журналов и мониторинг

*Дополнительные сведения см. в статье [производительность системы безопасности Azure: ведение журнала и мониторинг](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="22-configure-central-security-log-management"></a>2.2. Настройка централизованного управления журналами безопасности

**Руководство**. Azure Monitor использует журналы действий для записи изменений в ресурсы. Эти журналы можно экспортировать в службу хранилища Azure, концентратор событий или рабочую область Log Analytics. Прием журналов с помощью Azure Monitor для объединения данных безопасности, создаваемых устройствами конечных точек, сетевыми ресурсами и другими системами безопасности. В Azure Monitor можно запрашивать и выполнять анализ данных, использовать учетные записи хранения Azure для долгосрочного и архивного хранения журналов.

Кроме того, вы можете включить и подключить данные к Azure Sentinel или сторонним SIEM.

- [Как получить журналы и метрики платформы с помощью Azure Monitor](./essentials/diagnostic-settings.md)

- [Как получить журналы внутреннего узла виртуальной машины Azure с помощью Azure Monitor](./vm/quick-collect-azurevm.md)

- [Подключение к Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Начало работы с Azure Monitor и интеграция SIEM стороннего производителя](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

**Ответственность**: Customer

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. Insights**:

[!INCLUDE [Resource Policy for microsoft.insights 2.2](../../includes/policy/standards/asb/rp-controls/microsoft.insights-2-2.md)]

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3. Включение журналов аудита для ресурсов Azure

**Руководство**. Azure Monitor использует журналы действий, журнал действий автоматически включается и записывает в журнал операции, выполняемые с Azure Monitorными ресурсами, например: кто начал операцию, когда произошла операция, состояние операции и другие полезные сведения об аудите. 

- [Как получить журналы и метрики платформы с помощью Azure Monitor](./essentials/diagnostic-settings.md)

- [Общие сведения о ведении журналов и различных типах журналов в Azure](./essentials/platform-logs-overview.md)

**Ответственность**: Customer

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. Insights**:

[!INCLUDE [Resource Policy for microsoft.insights 2.3](../../includes/policy/standards/asb/rp-controls/microsoft.insights-2-3.md)]

### <a name="25-configure-security-log-storage-retention"></a>2.5. Настройка хранения журнала безопасности

**Рекомендации**. в Azure Monitor задайте срок хранения log Analytics рабочей области в соответствии с нормативными требованиями Организации. Используйте учетные записи хранения Azure для всех долгосрочных и архивных хранилищ журналов.

- [Изменение срока хранения данных в Log Analytics](./logs/manage-cost-storage.md#change-the-data-retention-period)

- [Настройка политики хранения для журналов учетных записей хранения Azure](../storage/common/manage-storage-analytics-logs.md#configure-logging)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="26-monitor-and-review-logs"></a>2,6: мониторинг и просмотр журналов

**Руководство**. анализ и мониторинг журналов для аномального поведения и регулярная проверка результатов. Используйте Azure Monitor и рабочую область Log Analytics для просмотра журналов и выполнения запросов к данным журнала.

Кроме того, вы можете включить и подключить данные к Azure Sentinel или сторонним SIEM.

- [Подключение к Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Приступая к работе с Log Analytics запросами](./logs/log-analytics-tutorial.md)

- [Выполнение пользовательских запросов в Azure Monitor](./logs/get-started-queries.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="27-enable-alerts-for-anomalous-activities"></a>2,7: Включение оповещений для аномальных действий

**Руководство**. Использование центра безопасности Azure с рабочей областью log Analytics для мониторинга и оповещения о аномальных действиях, обнаруженных в журналах и событиях безопасности. Кроме того, вы можете включить и подключить данные в Azure Sentinel.

- [Подключение к Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Управление оповещениями в центре безопасности Azure](../security-center/security-center-managing-and-responding-alerts.md)

- [Как оповещать данные журнала Log Analytics](./alerts/tutorial-response.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="identity-and-access-control"></a>Идентификатор и управление доступом

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: идентификация и управление доступом](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1. Инвентаризация учетных записей администраторов

**Руководство**. Управление доступом на основе ролей в Azure (Azure RBAC) позволяет управлять доступом к ресурсам Azure с помощью назначений ролей. Эти роли можно назначить пользователям, группам субъектов-служб и управляемым удостоверениям. Для некоторых ресурсов существуют предварительно определенные встроенные роли. Эти роли могут быть инвентаризированы или запрошены с помощью таких средств, как Azure CLI, Azure PowerShell или портал Azure.

- [Как получить роль каталога в Azure Active Directory (Azure AD) с помощью PowerShell](/powershell/module/azuread/get-azureaddirectoryrole?amp;preserve-view=true&view=azureadps-2.0)

- [Как получить членов роли каталога в Azure AD с помощью PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember?amp;preserve-view=true&view=azureadps-2.0)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="33-use-dedicated-administrative-accounts"></a>3.3. Применение выделенных административных учетных записей

**Руководство**. Создайте стандартные операционные процедуры для использования выделенных административных учетных записей. Используйте учетную запись Центра безопасности Azure и управление доступом для мониторинга количества административных учетных записей.

Вы также можете включить JIT-доступ с помощью Azure Active Directory (Azure AD) управление привилегированными пользователями привилегированных ролей для служб Майкрософт и Azure Resource Manager.

- [Обзор Azure AD Privileged Identity Management](../active-directory/privileged-identity-management/pim-configure.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="34-use-azure-active-directory-single-sign-on-sso"></a>3,4. Использование единого входа Azure Active Directory (SSO)

**Рекомендации**. везде, где это возможно, используйте Azure Active Directory (Azure AD) SSO вместо настройки отдельных автономных учетных данных для каждой службы. Используйте рекомендации по управлению удостоверениями и доступом в центре безопасности Azure.

- [Общие сведения об использовании единого входа в Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3,5: используйте многофакторную проверку подлинности для всех Azure Active Directory доступа

**Руководство**. Включение многофакторной проверки подлинности Azure Active Directory (Azure AD) и соблюдение рекомендаций по удостоверениям и доступу центра безопасности Azure.

- [Как включить многофакторную проверку подлинности в Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Мониторинг идентификации и доступа в Центре безопасности Azure](../security-center/security-center-identity-access.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6. Использование выделенных компьютеров (рабочих станций с привилегированным доступом) для всех административных задач

**Руководство**. используйте защищенную, управляемую Azure рабочую станцию (также называемую рабочей станцией привилегированного доступа или привилегированным доступом) для административных задач, требующих повышенных привилегий.

- [Общие сведения о защищенных рабочих станциях под управлением Azure](https://4sysops.com/archives/understand-the-microsoft-privileged-access-workstation-paw-security-model/)

- [Как включить многофакторную проверку подлинности Azure Active Directory (Azure AD)](../active-directory/authentication/howto-mfa-getstarted.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3,7: журналы и оповещения о подозрительных действиях учетных записей администраторов

**Руководство**. Использование Azure Active Directory (Azure AD) отчетов о безопасности и мониторинга для обнаружения подозрительных или ненадежных действий в среде. Используйте Центр безопасности Azure для мониторинга действий идентификации и доступа.

- [Как определить пользователей Azure AD, помеченных для события риска](../active-directory/identity-protection/overview-identity-protection.md)

- [Мониторинг пользовательских действий идентификации и доступа в Центре безопасности Azure](../security-center/security-center-identity-access.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8. Управление ресурсами Azure только из утвержденных расположений

**Руководство**. Используйте именованные расположения с условным доступом, чтобы разрешить доступ только из конкретных логических групп диапазонов IP-адресов или стран и регионов.

- [Как настроить именованные расположения](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="39-use-azure-active-directory"></a>3.9. Использование Azure Active Directory

**Руководство**. Использование Azure Active Directory (Azure AD) в качестве централизованной системы проверки подлинности и авторизации. Azure AD защищает данные с помощью надежного шифрования для хранимых и транзитных данных. Кроме того, в Azure AD используются salt-записи, хэши и безопасное хранение учетных данных пользователей.

- [Создание и настройка экземпляра Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10. Регулярная проверка и согласование доступа пользователей

**Руководство**. Azure Active Directory (Azure AD) предоставляет журналы для облегчения поиска устаревших учетных записей. Кроме того, используйте проверку доступа удостоверений Azure, чтобы эффективно управлять членством в группах, доступом к корпоративным приложениям и назначениями ролей. Доступ пользователей можно проверять на регулярной основе, чтобы только у авторизованных пользователей был постоянный доступ.

- [Общие сведения об отчетах Azure AD](../active-directory/reports-monitoring/index.yml)

- [Использование проверок доступа для идентификации Azure](../active-directory/governance/access-reviews-overview.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3,11: монитор пытается получить доступ к отключенным учетным данным

**Руководство**. у вас есть доступ к источникам журнала событий входа Azure Active Directory (Azure AD), аудита и риску, которые позволяют интегрироваться с любым средством SIEM/Monitoring. Этот процесс можно упростить, создав параметры диагностики для учетных записей пользователей Azure AD и отправив журналы аудита и журналы входа в рабочую область Log Analytics. Вы можете настроить необходимые оповещения в рабочей области Log Analytics.

- [Как интегрировать журналы действий Azure в Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3,12: предупреждение об отклонении при входе в учетную запись

**Руководство**. Использование Azure Active Directory (Azure AD) рисков и средств защиты идентификации для настройки автоматических ответов на обнаруженные подозрительные действия, связанные с удостоверениями пользователей. Вы также можете включить данные в Azure Sentinel для дальнейшего изучения.

- [Просмотр рискованных входов в Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Как настроить и включить политики рисков с помощью защиты идентификации](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Подключение к Azure Sentinel](../sentinel/quickstart-onboard.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="data-protection"></a>Защита данных

*Дополнительные сведения см. в статье [Azure Security Benchmark: защита данных](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1. Инвентаризация конфиденциальных данных

**Рекомендации**. Используйте теги, если это возможно для отслеживания Azure Monitorных ресурсов, которые хранят или обрабатывают конфиденциальную информацию, например log Analytics рабочих областей.

- [Создание и использование тегов](../azure-resource-manager/management/tag-resources.md)

- [Управление доступом к данным журнала и рабочим областям в Azure Monitor](./logs/manage-access.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2. Изолирование систем, хранящих или обрабатывающих конфиденциальные данные

**Руководство**. Реализация изоляции с помощью отдельных подписок и групп управления для отдельных доменов безопасности, таких как тип среды и уровень конфиденциальности данных. Вы можете ограничить уровень доступа к Azure Monitor и связанным ресурсам, которые требуются приложениям и корпоративным средам. Вы можете управлять доступом к Azure Monitor через Управление доступом на основе ролей Azure (Azure RBAC).

- [Создание дополнительных подписок Azure](../cost-management-billing/manage/create-subscription.md)

- [Создание групп управления](../governance/management-groups/create-management-group-portal.md)

- [Создание и использование тегов](../azure-resource-manager/management/tag-resources.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4. Шифрование любой конфиденциальной информации при передаче

**Руководство**. Azure Monitor по умолчанию СОГЛАСОВЫВАЕТ протокол TLS 1,2. Убедитесь, что все клиенты, подключающиеся к ресурсам Azure, могут согласовать TLS 1,2 или более поздней версии. 

Application Insights и Log Analytics оба продолжают принимать данные TLS 1,1 и TLS 1,0 для приема данных. Данные могут быть ограничены TLS 1,2 путем настройки на стороне клиента.

- [Как безопасно отправить данные с помощью TLS 1,2](./logs/data-security.md#sending-data-securely-using-tls-12)

**Ответственность**: Совмещаемая блокировка

**Мониторинг центра безопасности Azure**: нет

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5. Использование средства активного обнаружения для распознавания конфиденциальных данных

**Руководство**. функции для идентификации данных, классификации и предотвращения потерь еще не доступны для Azure Monitor. Установите сторонние решения, если это необходимо для обеспечения соответствия требованиям.
Для базовой платформы, управляемой корпорацией Майкрософт, корпорация Майкрософт считает все содержимое клиента конфиденциальным и предпринимает все возможные усилия для защиты клиентов от потери данных и раскрытия информации. Чтобы обеспечить безопасность данных клиентов в Azure, корпорация Майкрософт реализовала и поддерживает набор надежных элементов управления и возможностей защиты данных.

- [Общие сведения о защите данных клиентов в Azure](../security/fundamentals/protection-customer-data.md)

**Ответственность**: Совмещаемая блокировка

**Мониторинг центра безопасности Azure**: нет

### <a name="46-use-role-based-access-control-to-control-access-to-resources"></a>4,6. Использование управления доступом на основе ролей для управления доступом к ресурсам

**Руководство**. Использование управления доступом на основе РОЛЕЙ (RBAC) Azure для управления доступом к Azure Monitor.

- [Роли, разрешения и безопасность в Azure Monitor](./roles-permissions-security.md)

- [Как настроить RBAC в Azure](../role-based-access-control/role-assignments-portal.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8. Шифрование конфиденциальной информации при хранении

**Рекомендации**. Azure Monitor гарантирует, что все данные и сохраненные запросы шифруются при хранении с помощью ключей, управляемых корпорацией Майкрософт (ММК). Azure Monitor также предоставляет возможность шифрования с помощью собственного ключа, хранящегося в Azure Key Vault и доступного для хранилища, с помощью управляемой системой идентификации с проверкой подлинности. Этим ключом, управляемым клиентом (CMK), может быть программное или аппаратное обеспечение с защитой HSM.

- [Azure Monitor ключи, управляемые клиентом](./logs/customer-managed-keys.md)

- [Безопасность данных Log Analytics](./logs/data-security.md)

- [Сбор и хранение данных в Application Insights](app/data-retention-privacy.md)

- [Общие сведения о шифровании неактивных данных в Azure](../security/fundamentals/encryption-atrest.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9. Включение в журнал и создание оповещений по изменениям критических ресурсов Azure

**Руководство**. Использование Azure Monitor с журналом действий Azure для создания оповещений о том, когда изменения происходят в Azure Monitor и связанных ресурсах.

- [Создание оповещений для событий журнала действий Azure](./alerts/alerts-activity-log.md)

**Ответственность**: Customer

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. Insights**:

[!INCLUDE [Resource Policy for microsoft.insights 4.9](../../includes/policy/standards/asb/rp-controls/microsoft.insights-4-9.md)]

## <a name="vulnerability-management"></a>Управление уязвимостями

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: Управление уязвимостью](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5. Использование процесса оценки рисков для определения приоритета в устранении обнаруженных уязвимостей

**Руководство**. Используйте общую программу оценки рисков (например, общую систему оценки уязвимостей) или оценки рисков по умолчанию, предоставляемые сторонним средством сканирования.

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="inventory-and-asset-management"></a>Инвентаризация и управление ресурсами

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: Инвентаризация и управление активами](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6,1. Использование автоматизированного решения для обнаружения ресурсов

**Руководство**. Использование Azure CLI для запроса и обнаружения Azure Monitorных ресурсов в подписках. Убедитесь в том, что в вашем клиенте есть соответствующие разрешения (на чтение) и вы можете перечислить все подписки Azure, а также ресурсы в ваших подписках.

- [Azure Monitor CLI](/cli/azure/monitor)

- [Как просматривать подписки Azure](/powershell/module/az.accounts/get-azsubscription?preserve-view=true&view=azps-4.8.0)

- [Общие сведения об Azure RBAC](../role-based-access-control/overview.md)

- [Роли, разрешения и безопасность в Azure Monitor](./roles-permissions-security.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="62-maintain-asset-metadata"></a>6.2. Ведение метаданных активов

**Руководство**. применение тегов к Azure Monitor ресурсам, предоставляя метаданные для логической организации их в таксономию.

- [Создание и использование тегов](../azure-resource-manager/management/tag-resources.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="63-delete-unauthorized-azure-resources"></a>6.3. Удаление неавторизованных ресурсов Azure

**Руководство**. Использование тегов, групп управления и отдельных подписок (при необходимости) для упорядочения и отслеживания Azure Monitor связанных ресурсов. Регулярно сверяйте ресурсы, чтобы своевременно удалять неавторизованные ресурсы из подписки.

- [Создание дополнительных подписок Azure](../cost-management-billing/manage/create-subscription.md)

- [Создание групп управления](../governance/management-groups/create-management-group-portal.md)

- [Создание и использование тегов](../azure-resource-manager/management/tag-resources.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6,4: определение и обслуживание инвентаризации утвержденных ресурсов Azure

**Руководство**. Создание инвентаризации утвержденных ресурсов Azure и утвержденного программного обеспечения для ресурсов вычислений в соответствии с потребностями Организации.

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5. Отслеживание неутвержденных ресурсов Azure

**Руководство**. Использование политики Azure для ограничения типа ресурсов, которые могут быть созданы в подписках.

Используйте Azure Resource Graph для запроса и обнаружения ресурсов в своих подписках.  Убедитесь в том, что все ресурсы Azure, представленные в среде, утверждены.

- [Настройка Политики Azure и управление ею](../governance/policy/tutorials/create-and-manage.md)

- [Как создавать запросы с помощью обозревателя Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7. Удаление неутвержденных ресурсов Azure и программных приложений

**Руководство**. Выверка запасов на регулярной основе и обеспечение своевременного удаления неавторизованных Azure Monitor связанных ресурсов.  

- [Удаление рабочей области Azure Log Analytics](./logs/delete-workspace.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="69-use-only-approved-azure-services"></a>6.9. Использование только утвержденных служб Azure

**Руководство**. Использование политики Azure для ограничения Azure Monitor связанных ресурсов, которые можно подготавливать в вашей среде.

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

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1. Установка безопасных конфигураций для всех ресурсов Azure

**Руководство**. Использование настраиваемых определений политики Azure для аудита или принудительного применения конфигурации связанных ресурсов Azure Monitor. Вы также можете использовать встроенные определения политик Azure в качестве доступных.

Кроме того, Azure Resource Manager может экспортировать шаблон в нотация объектов JavaScript (JSON), который следует проверить, чтобы убедиться, что конфигурации соответствуют требованиям безопасности Организации или превышают их.

Вы также можете использовать рекомендации из центра безопасности Azure в качестве защищенного шаблона базовой конфигурации для ресурсов Azure.

При использовании возможностей динамической потоковой передачи данных необходимо обеспечить безопасность канала с помощью секретного ключа API в дополнение к ключу инструментирования.

- [Защита Live Metrics Stream APM](./app/live-stream.md#secure-the-control-channel)

- [Просмотр доступных псевдонимов политик Azure](/powershell/module/az.resources/get-azpolicyalias?amp;preserve-view=true&view=azps-4.8.0)

- [Руководство по Создание политик и управление ими для обеспечения соответствия требованиям](../governance/policy/tutorials/create-and-manage.md)

- [Экспорт одного и нескольких ресурсов в шаблон в портал Azure](../azure-resource-manager/templates/export-template-portal.md)

- [Рекомендации по безопасности — справочное руководство](../security-center/recommendations-reference.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3. Сохранение безопасных конфигураций для ресурсов Azure

**Руководство**. Использование политики Azure [Deny] и [развертывание если не существует] для обеспечения безопасности параметров в связанных с Azure Monitor ресурсах.  Кроме того, вы можете использовать шаблоны Azure Resource Manager, чтобы сохранить конфигурацию безопасности Azure Monitor связанных ресурсов, необходимых для Организации.

- [Общие сведения о влиянии политики Azure](../governance/policy/concepts/effects.md)

- [Создание политик и управление ими для обеспечения соответствия](../governance/policy/tutorials/create-and-manage.md)

- [Общие сведения о шаблонах Azure Resource Manager](../azure-resource-manager/templates/overview.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5. Безопасное хранение конфигурации ресурсов Azure

**Руководство**. Использование Azure DevOps для безопасного хранения и управления кодом, например пользовательскими политиками Azure и шаблонами Azure Resource Manager. Чтобы получить доступ к ресурсам, которыми вы управляете в Azure DevOps, вы можете предоставить или отклонить разрешения для определенных пользователей, встроенных групп безопасности или групп, определенных в Azure Active Directory (Azure AD), если они интегрированы с Azure DevOps, или Active Directory, если они интегрированы с TFS.

- [Как хранить код в Azure DevOps](/azure/devops/repos/git/gitworkflow?amp;preserve-view=true&view=azure-devops)

- [О разрешениях и группах в Azure DevOps](/azure/devops/organizations/security/about-permissions)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7,7: развертывание средств управления конфигурацией для ресурсов Azure

**Руководство**. Определение и реализация стандартных конфигураций безопасности для Azure Monitor связанных ресурсов с помощью политики Azure. Используйте пользовательские определения политики Azure для аудита или принудительного применения конфигурации безопасности Azure Monitor связанных ресурсов. Вы также можете использовать встроенные определения политик, связанные с конкретными ресурсами.

- [Настройка Политики Azure и управление ею](../governance/policy/tutorials/create-and-manage.md)

- [Псевдонимы политик Azure](../governance/policy/concepts/definition-structure.md#aliases)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7,9. Реализация автоматического мониторинга конфигурации для ресурсов Azure

**Руководство**. Использование центра безопасности Azure для выполнения проверок базовых данных для Azure Monitor связанных ресурсов.  Кроме того, используйте политику Azure для оповещения и аудита конфигураций ресурсов Azure.

- [Как исправить рекомендации в центре безопасности Azure](../security-center/security-center-remediate-recommendations.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="711-manage-azure-secrets-securely"></a>7.11. Безопасное управление секретами Azure

**Руководство**. Используйте управляемое удостоверение службы в сочетании с Azure Key Vault, чтобы упростить и защитить управление секретами для поддерживаемых ресурсов, связанных с Azure Monitor.

- [Интеграция с управляемыми удостоверениями Azure](../azure-app-configuration/howto-integrate-azure-managed-service-identity.md)

- [Службы с поддержкой управляемых удостоверений для ресурсов Azure](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md)

- [Создание Key Vault](../key-vault/secrets/quick-create-portal.md)

- [Как обеспечить проверку подлинности Key Vault с помощью управляемого удостоверения](/azure/key-vault/general/assign-access=policy-portal)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="712-manage-identities-securely-and-automatically"></a>7.12. Безопасное и автоматическое управление удостоверениями

**Руководство**. Использование управляемых удостоверений для предоставления служб Azure с автоматически управляемым удостоверением в Azure Active Directory (Azure AD). Управляемые удостоверения позволяют проходить проверку подлинности в любой службе, поддерживающей проверку подлинности Azure AD, включая Azure Monitorные ресурсы без каких бы то ни было учетных данных в коде.

- [Настройка управляемых удостоверений для ресурсов Azure](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13. Устранение непреднамеренного раскрытия учетных данных

**Руководство**. Реализуйте сканер учетных данных для обнаружения учетных данных в коде. Сканер учетных данных также рекомендует перемещать обнаруженные учетные данные в более безопасные расположения, такие как Azure Key Vault.

- [Настройка средства проверки учетных данных](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="malware-defense"></a>Защита от вредоносных программ

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: защита от вредоносных программ](../security/benchmarks/security-control-malware-defense.md).*

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2. Предварительная проверка файлов для отправки в ресурсы Azure, не являющиеся вычислительными

**Руководство**. Защита от вредоносных программ Майкрософт включена на базовом узле, поддерживающем службы Azure (например, Azure Monitor связанные ресурсы), но не выполняется в вашем содержимом. 

Предварительно проверять все файлы, отправляемые в соответствующие Azure Monitor связанные ресурсы, такие как Log Analytics рабочей области.

Используйте средство обнаружения угроз для служб данных в центре безопасности Azure, чтобы обнаружить вредоносные программы, отправленные в учетные записи хранения. 

- [Сведения об антивредоносном по Майкрософт для облачных служб и виртуальных машин Azure](../security/fundamentals/antimalware.md)

- [Сведения об обнаружении угроз для служб данных в центре безопасности Azure](../security-center/azure-defender.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="data-recovery"></a>Восстановление данных

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: восстановление данных](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9,1: Обеспечьте регулярное автоматическое резервное копирование

**Руководство**. Использование Azure Resource Manager для экспорта Azure Monitor и связанных ресурсов в шаблон НОТАЦИЯ объектов JavaScript (JSON), который можно использовать в качестве резервного копирования для Azure Monitor и связанных конфигураций.  Используйте службу автоматизации Azure для автоматического запуска сценариев резервного копирования. 

- [Управление рабочей областью Log Analytics с помощью шаблонов Azure Resource Manager](./logs/resource-manager-workspace.md)

- [Экспорт одного и нескольких ресурсов в шаблон в портал Azure](../azure-resource-manager/templates/export-template-portal.md)

- [Сведения о службе автоматизации Azure](../automation/automation-intro.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9,2: выполните полное резервное копирование системы и резервное копирование ключей, управляемых клиентом

**Руководство**. Использование Azure Resource Manager для экспорта Azure Monitor и связанных ресурсов в шаблон НОТАЦИЯ объектов JavaScript (JSON), который можно использовать в качестве резервного копирования для Azure Monitor и связанных конфигураций. Резервное копирование управляемых клиентом ключей в Azure Key Vault если Azure Monitor связанные ресурсы используют ключи, управляемые клиентом,

- [Управление рабочей областью Log Analytics с помощью шаблонов Azure Resource Manager](./logs/resource-manager-workspace.md)

- [Экспорт одного и нескольких ресурсов в шаблон в портал Azure](../azure-resource-manager/templates/export-template-portal.md)

- [Как создать резервную копию ключей хранилища ключей в Azure](/powershell/module/az.keyvault/backup-azkeyvaultkey?amp;preserve-view=true&view=azps-4.8.0)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9,3: Проверьте все резервные копии, включая управляемые клиентом ключи.

**Руководство**. обеспечение возможности регулярного выполнения восстановления с помощью Azure Resource Manager файлов шаблонов. Проверка восстановления резервных копий ключей, управляемых клиентом.

- [Управление рабочей областью Log Analytics с помощью шаблонов Azure Resource Manager](./logs/resource-manager-workspace.md)

- [Как восстановить резервную копию ключей хранилища ключей в Azure](/powershell/module/az.keyvault/restore-azkeyvaultkey?amp;preserve-view=true&view=azps-4.8.0)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9,4: Обеспечьте защиту резервных копий и ключей, управляемых клиентом

**Руководство**. Использование Azure DevOps для безопасного хранения и управления кодом, например пользовательскими политиками Azure, Azure Resource Manager шаблонами. Чтобы защитить ресурсы, которыми вы управляете в Azure DevOps, вы можете предоставить или отклонить разрешения для определенных пользователей, встроенных групп безопасности или групп, определенных в Azure Active Directory (Azure AD), если они интегрированы с Azure DevOps, или Active Directory, если они интегрированы с TFS.  Используйте управление доступом на основе ролей Azure (Azure RBAC) для защиты ключей, управляемых клиентом.

Кроме того, включите Soft-Delete и очистите защиту в Key Vault, чтобы защитить ключи от случайного или вредоносного удаления. Если служба хранилища Azure используется для хранения резервных копий шаблонов Azure Resource Manager, включите обратимое удаление, чтобы сохранять и восстанавливать данные при удалении больших двоичных объектов или моментальных снимков больших двоичных объектов.

- [Как хранить код в Azure DevOps](/azure/devops/repos/git/gitworkflow?amp;preserve-view=true&view=azure-devops)

- [О разрешениях и группах в Azure DevOps](/azure/devops/organizations/security/about-permissions)

- [Включение Soft-Delete и очистка защиты в Key Vault](../storage/blobs/soft-delete-blob-overview.md)

- [Soft delete for Azure Storage blobs](../storage/blobs/soft-delete-blob-overview.md) (Обратимое удаление больших двоичных объектов службы хранилища Azure)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="incident-response"></a>реагирование на инциденты.

*Дополнительные сведения см. в статье [Azure Security Benchmark: реагирование на инциденты](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1. Создание руководства по реагированию на инциденты

**Руководство**. Создайте руководство по реагированию на инциденты для вашей организации. Убедитесь в том, что имеются письменные планы реагирования на инциденты, которые определяют все действия персонала, а также этапы обработки инцидентов и управления ими для проверки после инцидента.

- [Руководство по созданию собственного процесса реагирования на инциденты безопасности](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Анатомия инцидента Центра Майкрософт по реагированию на угрозы](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)

- [Использование руководства по обработке инцидентов безопасности компьютера от NIST для создания собственного плана реагирования на инциденты](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2. Создание процедуры оценки инцидента и определения приоритетов

**Руководство**. Центр безопасности назначает каждому оповещению серьезность, которая поможет определить, какие предупреждения следует расследовать первыми. Серьезность основывается на том, насколько надежным является центр безопасности в поиске или аналитике, используемой для выдаче оповещения, а также об уровне достоверности, который был вредоносным намерением для действия, вызвавшего оповещение.

Кроме того, четко помечайте подписки тегами (например, рабочие, нерабочие) и создайте систему именования, чтобы четко определить и классифицировать ресурсы Azure, особенно обрабатывающие конфиденциальные данные.  Вы несете ответственность за назначение приоритета оповещениям, требующим действий по исправлению, в зависимости от важности ресурсов Azure и среды, в которой произошел инцидент.

- [Оповещения безопасности в Центре безопасности Azure](../security-center/security-center-alerts-overview.md)

- [Создание и использование тегов](../azure-resource-manager/management/tag-resources.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="103-test-security-response-procedures"></a>10.3. Проверка процедур реагирования на угрозы

**Руководство**. Выполняйте упражнения, чтобы периодически тестировать возможности ваших систем реагировать на угрозы для защиты ресурсов Azure. Выявите слабые точки и пробелов и пересмотрите план по мере необходимости.

- [Публикация NIST — руководство по тестированию, обучению и выполнению программ для ИТ-планов и возможностей](https://csrc.nist.gov/publications/detail/sp/800-84/final)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4. Предоставление контактных сведений и настройка уведомлений по инцидентам безопасности

**Руководство**. Корпорация Майкрософт будет использовать информацию об инциденте безопасности для связи с вами, если центр Microsoft Security Response Center (MSRC) обнаружит, что к вашим данным был получен незаконный или несанкционированный доступ. Проверьте инциденты после факта обращения, чтобы убедиться в том, что проблемы устранены.

- [Как задать контакт безопасности Центра безопасности Azure](../security-center/security-center-provide-security-contact-details.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5. Включение оповещений системы безопасности в систему реагирования на инциденты

**Руководство**. Экспортируйте оповещения и рекомендации Центра безопасности Azure с помощью функции непрерывного экспорта с целью выявления рисков для ресурсов Azure. Непрерывный экспорт позволяет экспортировать предупреждения и рекомендации как вручную, так и в постоянном, непрерывном режиме. Вы можете использовать соединитель данных Центра безопасности Azure для потоковой передачи оповещений в Azure Sentinel. 

- [Настройка непрерывного экспорта данных](../security-center/continuous-export.md) 

- [Как выполнить потоковую передачу оповещений в Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="106-automate-the-response-to-security-alerts"></a>10.6. Автоматизация реагирования на оповещения системы безопасности

**Руководство**. Использование функции автоматизации рабочих процессов в центре безопасности Azure для автоматического запуска ответов с помощью "Logic Apps" в оповещениях системы безопасности и рекомендациях по защите ресурсов Azure.

- [Как настроить автоматизацию рабочего процесса и Logic Apps](../security-center/workflow-automation.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="penetration-tests-and-red-team-exercises"></a>Тесты на проникновение и попытки нарушения безопасности "красной командой"

*Дополнительные сведения см. в статье [тесты производительности системы безопасности Azure: испытания на проникновение и команды красных команд](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11,1. Проведите регулярное тестирование на проникновение ресурсов Azure и обеспечьте исправление всех критических результатов безопасности.

**Руководство**. соблюдение правил корпорации Майкрософт, чтобы убедиться, что тесты на проникновение не нарушают политики Майкрософт. Используйте стратегию и исполнение Microsoft, а затем протестировать проникновение в реальном времени для управляемой корпорацией Майкрософт облачной инфраструктуры, служб и приложений.

- [Правила взаимодействия при выполнении тестирования на проникновение](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

- [Привлечение "красных команд для тестирования "Microsoft Cloud](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Ответственность**: Совмещаемая блокировка

**Мониторинг центра безопасности Azure**: нет

## <a name="next-steps"></a>Следующие шаги

- См. [Обзор Azure Security Benchmark версии 2](../security/benchmarks/overview.md)
- Дополнительные сведения о [базовой конфигурации безопасности Azure](../security/benchmarks/security-baselines-overview.md).