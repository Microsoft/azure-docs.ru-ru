---
title: Базовый план безопасности Azure для Data Lake Analytics
description: Базовый план безопасности Data Lake Analytics содержит практические руководства и ресурсы для реализации рекомендаций по безопасности, указанных в статье о производительности системы безопасности Azure.
author: msmbaldwin
ms.service: data-lake-analytics
ms.topic: conceptual
ms.date: 02/17/2021
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 7569f98b1a721ecc9bd8e41a7d55cfbb80a2bc88
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105564625"
---
# <a name="azure-security-baseline-for-data-lake-analytics"></a>Базовый план безопасности Azure для Data Lake Analytics

Этот базовый план безопасности применяет рекомендации из [теста безопасности Azure версии 1,0](../security/benchmarks/overview-v1.md) к Data Lake Analytics. Azure Security Benchmark содержит рекомендации по обеспечению безопасности облачных решений в Azure.
Содержимое группируются по **элементам управления безопасностью** , определенным в производительности системы безопасности Azure, и связанным рекомендациям, применимым к Data Lake Analytics. **Элементы управления** , неприменимые к Data Lake Analytics, были исключены.

 
Сведения о том, как Data Lake Analytics полностью сопоставлены с тестовым показателем безопасности Azure, см. в разделе [полное Data Lake Analytics файла сопоставления базовых показателей безопасности](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Безопасность сети

*Дополнительные сведения см. в статье [Azure Security Benchmark: безопасность сети](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1,1: защита ресурсов Azure в виртуальных сетях

**Руководство**. Используйте параметры брандмауэра для Data Lake Analytics ограничения внешних диапазонов IP-адресов, чтобы разрешить доступ с локальных клиентов и сторонних служб. Настройка параметров брандмауэра доступна через портал, API-интерфейсы или PowerShell.

- [Общие сведения о правилах брандмауэра](/rest/api/datalakeanalytics/firewallrules) 

- [Управление аналитикой озера данных Azure с помощью Azure PowerShell](data-lake-analytics-manage-use-powershell.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1,4: запретите обмен данными с известными вредоносными IP-адресами

**Руководство**. Используйте параметры брандмауэра для Data Lake Analytics ограничения внешних диапазонов IP-адресов, чтобы разрешить доступ с локальных клиентов и сторонних служб.  Настройка параметров брандмауэра доступна через портал, API-интерфейсы или PowerShell.

- [Общие сведения о правилах брандмауэра](/rest/api/datalakeanalytics/firewallrules) 

- [Управление аналитикой озера данных Azure с помощью Azure PowerShell](data-lake-analytics-manage-use-powershell.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="logging-and-monitoring"></a>Ведение журналов и мониторинг

*Дополнительные сведения см. в статье [производительность системы безопасности Azure: ведение журнала и мониторинг](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="22-configure-central-security-log-management"></a>2.2. Настройка централизованного управления журналами безопасности

**Руководство**. прием журналов с помощью Azure Monitor для агрегирования данных безопасности, таких как Data Lake Analytics "Audit" и "Диагностика запросов". В Azure Monitor используйте рабочую область Log Analytics для запроса и выполнения анализа и используйте учетные записи хранения Azure для долгосрочного и архивного хранения, при необходимости используя такие функции безопасности, как неизменяемое хранилище и принудительное хранение. 

Кроме того, вы можете включить и подключить данные в метку Azure, а также в решение для управления событиями и системами сторонних производителей.

- [Доступ к журналам диагностики для Azure Data Lake Analytics](data-lake-analytics-diagnostic-logs.md)

- [Подключение к Azure Sentinel](../sentinel/quickstart-onboard.md) 

- [Как получить журналы и метрики платформы с помощью Azure Monitor](../azure-monitor/essentials/diagnostic-settings.md) 

- [Как получить журналы внутреннего узла виртуальной машины Azure с помощью Azure Monitor](../azure-monitor/vm/quick-collect-azurevm.md) 

- [Начало работы с Azure Monitor и интеграция SIEM стороннего производителя](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3. Включение журналов аудита для ресурсов Azure

**Руководство**. Включение параметров диагностики для Data Lake Analytics доступа к журналам аудита и запросов. К ним относятся такие данные, как источник событий, Дата, пользователь, отметка времени и другие полезные элементы. 

- [Как получить журналы и метрики платформы с помощью Azure Monitor](../azure-monitor/essentials/diagnostic-settings.md) 

- [Общие сведения о ведении журналов и различных типах журналов в Azure](../azure-monitor/essentials/platform-logs-overview.md)

**Ответственность**: Customer

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/azure/governance/policy/samples/azure-security-benchmark) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/azure/security-center/security-center-recommendations). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/azure/security-center/azure-defender) для связанных служб.

**Встроенные определения политики Azure — Microsoft. Data Lake Analytics**:

[!INCLUDE [Resource Policy for Microsoft.DataLakeAnalytics 2.3](../../includes/policy/standards/asb/rp-controls/microsoft.datalakeanalytics-2-3.md)]

### <a name="25-configure-security-log-storage-retention"></a>2.5. Настройка хранения журнала безопасности

**Рекомендации**. в Azure Monitor задайте срок хранения log Analytics рабочей области в соответствии с нормативными требованиями Организации. Используйте учетные записи хранения Azure для долгосрочного и архивного хранения.

- [Изменение срока хранения данных в Log Analytics](../azure-monitor/logs/manage-cost-storage.md#change-the-data-retention-period) 

- [Настройка политики хранения для журналов учетных записей хранения Azure](../storage/common/manage-storage-analytics-logs.md#configure-logging)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="26-monitor-and-review-logs"></a>2,6: мониторинг и просмотр журналов

**Руководство**. анализ и мониторинг журналов для аномального поведения и регулярная проверка результатов для Data Lake Analyticsных ресурсов. Используйте рабочую область Log Analytics Azure Monitor, чтобы просматривать журналы и выполнять запросы к данным журнала. Кроме того, вы можете включить и подключить данные в метку Azure, а также в решение для управления событиями и системами сторонних производителей.

- [Подключение к Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Дополнительные сведения о рабочей области Log Analytics](../azure-monitor/logs/log-analytics-tutorial.md)

- [Выполнение пользовательских запросов в Azure Monitor](../azure-monitor/logs/get-started-queries.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="27-enable-alerts-for-anomalous-activities"></a>2,7: Включение оповещений для аномальных действий

**Руководство**. Включение параметров диагностики для Data Lake Analytics и отправка журналов в рабочую область log Analytics. Подключите рабочую область Log Analytics к Azure Sentinel, так как она предоставляет решение для оркестрации событий безопасности и автоматического реагирования (SOAR). Это позволяет создавать и использовать сборники схем (автоматизированные решения) для устранения проблем безопасности. 

- [Подключение к Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Как оповещать данные журнала Log Analytics](../azure-monitor/alerts/tutorial-response.md)  

- [Доступ к журналам диагностики для Azure Data Lake Analytics](data-lake-analytics-diagnostic-logs.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="identity-and-access-control"></a>Идентификатор и управление доступом

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: идентификация и управление доступом](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1. Инвентаризация учетных записей администраторов

**Руководство**. Azure Active Directory (Azure AD) содержит встроенные роли, которые должны быть явно назначены и доступны для запросов. Используйте модуль Azure AD PowerShell для выполнения нерегламентированных запросов для обнаружения учетных записей, входящих в группы администраторов.

- [Как получить роль каталога в Azure AD с помощью PowerShell](/powershell/module/azuread/get-azureaddirectoryrole)

- [Как получить членов роли каталога в Azure AD с помощью PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="32-change-default-passwords-where-applicable"></a>3.2. Изменение паролей по умолчанию, где применимо

**Руководство**. Data Lake Analytics не имеет концепции паролей по умолчанию, так как проверка подлинности предоставляется в Azure Active Directory (Azure AD) и защищена с помощью управления доступом на основе ролей Azure (Azure RBAC).

- [Обзор Azure Data Lake Analytics](data-lake-analytics-overview.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="33-use-dedicated-administrative-accounts"></a>3.3. Применение выделенных административных учетных записей

**Руководство**. Создайте стандартные операционные процедуры для использования выделенных административных учетных записей.

Вы также можете включить JIT-доступ с помощью Azure Active Directory (Azure AD) управление привилегированными пользователями и Azure Resource Manager.

- [Дополнительные сведения о управление привилегированными пользователями](../active-directory/privileged-identity-management/index.yml)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="34-use-azure-active-directory-single-sign-on-sso"></a>3,4. Использование единого входа Azure Active Directory (SSO)

**Рекомендации**. везде, где это возможно, используйте Azure Active Directory (Azure AD) SSO вместо настройки отдельных автономных учетных данных для каждой службы. Использование удостоверений центра безопасности Azure и рекомендаций по доступу.

- [Общие сведения об использовании единого входа в Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3,5: используйте многофакторную проверку подлинности для всех Azure Active Directory доступа

**Руководство**. Включение многофакторной проверки подлинности Azure Active Directory (Azure AD) и следование рекомендациям по управлению удостоверениями и доступом в центре безопасности Azure для защиты ресурсов Data Lake Analytics.

- [Как включить многофакторную проверку подлинности в Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Мониторинг идентификации и доступа в Центре безопасности Azure](../security-center/security-center-identity-access.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6. Использование выделенных компьютеров (рабочих станций с привилегированным доступом) для всех административных задач

**Руководство**. используйте защищенную, управляемую Azure рабочую станцию (также называемую рабочей станцией привилегированного доступа или привилегированным доступом) для административных задач, требующих повышенных привилегий.

- [Общие сведения о защищенных рабочих станциях под управлением Azure](https://4sysops.com/archives/understand-the-microsoft-privileged-access-workstation-paw-security-model/)

- [Как включить многофакторную проверку подлинности Azure Active Directory (Azure AD)](/security/compass/privileged-access-devices)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3,7: журналы и оповещения о подозрительных действиях учетных записей администраторов

**Руководство**. Использование отчетов по безопасности Azure Active Directory (Azure AD) для создания журналов и оповещений при возникновении подозрительных или ненадежных действий в среде. Используйте Центр безопасности Azure для мониторинга действий идентификации и доступа.

- [Как определить пользователей Azure AD, помеченных для события риска](../active-directory/identity-protection/overview-identity-protection.md)

- [Мониторинг пользовательских действий идентификации и доступа в Центре безопасности Azure](../security-center/security-center-identity-access.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8. Управление ресурсами Azure только из утвержденных расположений

**Руководство**. Использование Azure Active Directory (Azure AD) с именованными расположениями для разрешения доступа только из конкретных логических групп диапазонов IP-адресов или стран и регионов.

- [Настройка именованных расположений Azure AD](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="39-use-azure-active-directory"></a>3.9. Использование Azure Active Directory

**Руководство**. Использование Azure Active Directory (Azure AD) в качестве централизованной системы проверки подлинности и авторизации. Управление доступом на основе ролей в Azure (Azure RBAC) обеспечивает точный контроль доступа клиента к ресурсам Data Lake Analytics.

 

- [Создание и настройка экземпляра Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10. Регулярная проверка и согласование доступа пользователей

**Руководство**. Azure Active Directory (Azure AD) предоставляет журналы для облегчения поиска устаревших учетных записей. Кроме того, используйте проверки подлинности и доступа Azure AD для эффективного управления членством в группах, доступом к корпоративным приложениям и назначениями ролей. Доступ пользователей можно проверить регулярно, чтобы убедиться, что доступ к ним имеют только нужные пользователи.

- [Общие сведения об отчетах Azure AD](../active-directory/reports-monitoring/index.yml)

- [Использование проверок доступа для идентификации Azure AD](../active-directory/governance/access-reviews-overview.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3,11: монитор пытается получить доступ к отключенным учетным данным

**Руководство**. Включение параметров диагностики для Data Lake Analytics и Azure Active Directory (Azure AD) с последующим отправкой всех журналов в log Analytics рабочую область. Настройте нужные оповещения (например, попытки доступа к отключенным секретам) в Log Analytics.

- [Интеграция журналов Azure AD с Azure Monitor журналами](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3,12: предупреждение об отклонении при входе в учетную запись

**Руководство**. использование возможностей Azure Active Directory (Azure AD) и защиты идентификации для настройки автоматических ответов на обнаруженные подозрительные действия, связанные с ресурсами Data Lake Analytics. Вы должны включить автоматические ответы через метку Azure, чтобы реализовать ответы на безопасность вашей организации.

- [Просмотр рискованных входов в Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Как настроить и включить политики рисков с помощью защиты идентификации](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Подключение к Azure Sentinel](../sentinel/quickstart-onboard.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="data-protection"></a>Защита данных

*Дополнительные сведения см. в статье [Azure Security Benchmark: защита данных](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1. Инвентаризация конфиденциальных данных

**Руководство**. Использование тегов для отслеживания Data Lake Analytics ресурсов, в которых хранятся или обрабатываются конфиденциальные данные. 

- [Создание и использование тегов](../azure-resource-manager/management/tag-resources.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2. Изолирование систем, хранящих или обрабатывающих конфиденциальные данные

**Руководство**. Реализация изоляции с помощью отдельных подписок, групп управления для отдельных доменов безопасности, таких как среда, чувствительность данных. Вы можете ограничить Data Lake Analytics, чтобы управлять уровнем доступа к ресурсам Data Lake Analytics, которые требуются приложениям и корпоративным средам. Когда правила брандмауэра настроены, доступ к ресурсам Data Lake Analytics могут получить только приложения, запрашивающие данные через указанный набор сетей. Вы можете управлять доступом к Azure Data Lake Analytics через Управление доступом на основе ролей Azure (Azure RBAC).

- [Создание дополнительных подписок Azure](../cost-management-billing/manage/create-subscription.md)

- [Создание групп управления](../governance/management-groups/create-management-group-portal.md)

- [Создание и использование тегов](../azure-resource-manager/management/tag-resources.md)

- [Управление управлением доступом на основе ролей в Azure](./data-lake-analytics-manage-use-portal.md#manage-azure-role-based-access-control)

- [Правила брандмауэра](/rest/api/datalakeanalytics/firewallrules)

- [Управление аналитикой озера данных Azure с помощью Azure PowerShell](data-lake-analytics-manage-use-powershell.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. Мониторинг и блокирование несанкционированной передачи конфиденциальной информации

**Руководство**. функции защиты от потери данных еще не доступны для Azure Data Lake Analyticsных ресурсов. Установите сторонние решения, если это необходимо для обеспечения соответствия требованиям.

Для базовой платформы, управляемой корпорацией Майкрософт, корпорация Майкрософт считает все содержимое клиента конфиденциальным и защищенным от потери и доступности данных клиентов. Чтобы обеспечить безопасность данных клиентов в Azure, корпорация Майкрософт реализовала и поддерживает набор надежных элементов управления и возможностей защиты данных.

- [Общие сведения о защите данных клиентов в Azure](../security/fundamentals/protection-customer-data.md)

- [Как защитить службу хранилища Azure](../storage/blobs/security-recommendations.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4. Шифрование любой конфиденциальной информации при передаче

**Руководство**. Microsoft Azure ресурсы будут согласовывать TLS 1,2 по умолчанию. Убедитесь, что все клиенты, подключающиеся к Data Lake Analytics, могут согласовываться с помощью TLS 1,2 или более поздней версии.

- [Пример списка операций](/rest/api/datalakeanalytics/operations/list)

**Ответственность**: Совмещаемая блокировка

**Мониторинг центра безопасности Azure**: нет

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5. Использование средства активного обнаружения для распознавания конфиденциальных данных

**Руководство**. функции идентификации данных еще недоступны для Azure Data Lake Analyticsных ресурсов. Установите сторонние решения, если это необходимо для обеспечения соответствия требованиям. 

- [Общие сведения о защите данных клиентов в Azure](../security/fundamentals/protection-customer-data.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6. Контроль доступа к ресурсам с помощью RBAC 

**Руководство**. Использование управления доступом на основе ролей Azure (Azure RBAC) для управления взаимодействием пользователей со службой.

- [Управление Azure RBAC](./data-lake-analytics-manage-use-portal.md#manage-azure-role-based-access-control)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8. Шифрование конфиденциальной информации при хранении

**Руководство**. данные хранятся в учетной записи Data Lake Storage 1-го поколения по умолчанию.  Data Lake Storage 1-го поколения поддерживает прозрачное шифрование неактивных данных, включенное по умолчанию.

- [Шифрование данных в Azure Data Lake Storage 1-го поколения](../data-lake-store/data-lake-store-encryption.md)

**Ответственность**: Совмещаемая блокировка

**Мониторинг центра безопасности Azure**: нет

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9. Включение в журнал и создание оповещений по изменениям критических ресурсов Azure

**Руководство**. Использование Azure Monitor с журналом действий Azure для создания оповещений о том, когда изменения выполняются в рабочих экземплярах Azure Data Lake Analyticsных ресурсов.

- [Создание оповещений для событий журнала действий Azure](../azure-monitor/alerts/alerts-activity-log.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="vulnerability-management"></a>Управление уязвимостями

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: Управление уязвимостью](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1. Выполнение автоматизированных средства анализа уязвимостей

**Рекомендации**. Следуйте рекомендациям центра безопасности Azure по защите ресурсов Azure Data Lake Analytics.

Корпорация Майкрософт выполняет Управление уязвимостью в базовых системах, поддерживающих Azure Data Lake Analytics.

- [Общие сведения о рекомендациях центра безопасности Azure](../security-center/recommendations-reference.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5. Использование процесса оценки рисков для определения приоритета в устранении обнаруженных уязвимостей

**Руководство**. Используйте общую программу оценки рисков (например, общую систему оценки уязвимостей) или оценки рисков по умолчанию, предоставляемые сторонним средством сканирования.

- [Публикация NIST — общая система оценки уязвимости](https://www.nist.gov/publications/common-vulnerability-scoring-system)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="inventory-and-asset-management"></a>Инвентаризация и управление ресурсами

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: Инвентаризация и управление активами](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6,1. Использование автоматизированного решения для обнаружения ресурсов

**Руководство**. Использование графа ресурсов Azure для запроса и обнаружения всех ресурсов (например, вычислений, хранилища, сети, портов и протоколов и т. д.) в подписках. Обеспечьте соответствующие разрешения (чтение) в клиенте и перечислите все подписки Azure, а также ресурсы в подписках.

Хотя классические ресурсы Azure могут быть обнаружены с помощью обозревателя графа ресурсов Azure, настоятельно рекомендуется создавать и использовать Azure Resource Manager ресурсов.

- [Как создавать запросы с помощью обозревателя Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

- [Просмотр подписок Azure](/powershell/module/az.accounts/get-azsubscription)

- [Общие сведения об Azure RBAC](../role-based-access-control/overview.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="62-maintain-asset-metadata"></a>6.2. Ведение метаданных активов

**Руководство**. Применяйте к ресурсам Azure теги, чтобы логически классифицировать их на основе метаданных.

- [Создание и использование тегов](../azure-resource-manager/management/tag-resources.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="63-delete-unauthorized-azure-resources"></a>6.3. Удаление неавторизованных ресурсов Azure

**Руководство**. Использование тегов, групп управления и отдельных подписок (при необходимости) для Организации и мониторинга Azure Data Lake Analyticsных ресурсов. Регулярно сверяйте ресурсы, чтобы своевременно удалять неавторизованные ресурсы из подписки.

Кроме того, используйте политику Azure, чтобы ограничить тип ресурсов, которые могут быть созданы в клиентских подписках, используя следующие встроенные определения политик:

- Недопустимые типы ресурсов

- Допустимые типы ресурсов

Дополнительные сведения можно найти по ссылкам, на которые имеются ссылки.

- [Создание дополнительных подписок Azure](../cost-management-billing/manage/create-subscription.md)

- [Создание групп управления](../governance/management-groups/create-management-group-portal.md)

- [Создание и использование тегов](../azure-resource-manager/management/tag-resources.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5. Отслеживание неутвержденных ресурсов Azure

**Руководство**. Использование политики Azure для ограничения типа ресурсов, которые могут быть созданы в клиентских подписках, с помощью следующих встроенных определений политик:

- Недопустимые типы ресурсов

- Допустимые типы ресурсов

Кроме того, используйте граф ресурсов Azure для запроса или обнаружения ресурсов в подписках.

- [Настройка Политики Azure и управление ею](../governance/policy/tutorials/create-and-manage.md)

- [Как создавать запросы с помощью Azure Graph](../governance/resource-graph/first-query-portal.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="69-use-only-approved-azure-services"></a>6.9. Использование только утвержденных служб Azure

**Руководство**. Используйте Политику Azure, чтобы ограничить тип ресурсов, которые могут быть созданы в подписках клиентов, используя следующие встроенные определения политик.

- Недопустимые типы ресурсов
- Допустимые типы ресурсов

Дополнительные сведения доступны по ссылкам, на которые имеются ссылки

- [Настройка Политики Azure и управление ею](../governance/policy/tutorials/create-and-manage.md)

- [Как отказаться от определенного типа ресурса с помощью Политики Azure](../governance/policy/samples/built-in-policies.md#general)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6,11: ограничьте возможность пользователей работать с Azure Resource Manager

**Руководство**. Настройте условный доступ Azure, чтобы ограничить возможность пользователей взаимодействовать с Azure Resource Manager путем настройки "Блокировать доступ" для приложения "Управление Microsoft Azure".

- [Как настроить условный доступ для блокировки доступа к Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="secure-configuration"></a>Настройка безопасности

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: безопасная конфигурация](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1. Установка безопасных конфигураций для всех ресурсов Azure

**Руководство**. Использование псевдонимов политик Azure в пространстве имен Microsoft. Data Lake Analytics для создания настраиваемых политик для аудита или принудительного применения конфигурации службы Azure Data Lake Analytics. Вы также можете использовать встроенные определения политик, связанные с Azure Data Lake Analytics, например:

- В Data Lake Analytics должны быть включены журналы диагностики.

Дополнительные сведения доступны по ссылкам, на которые имеются ссылки

- [Просмотр доступных псевдонимов политик Azure](/powershell/module/az.resources/get-azpolicyalias)

- [Настройка Политики Azure и управление ею](../governance/policy/tutorials/create-and-manage.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3. Сохранение безопасных конфигураций для ресурсов Azure

**Рекомендации**. Используйте эффекты политики Azure [Deny] и [развертывание, если не существует] для обеспечения безопасности параметров в ресурсах Azure.

- [Настройка Политики Azure и управление ею](../governance/policy/tutorials/create-and-manage.md)

- [Сведения о действии Политик Azure](../governance/policy/concepts/effects.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5. Безопасное хранение конфигурации ресурсов Azure

**Руководство**. Использование Azure Repos для безопасного хранения и управления кодом, например пользовательскими политиками Azure, Azure Resource Manager шаблонами, сценариями настройки требуемого состояния и т. д. Чтобы получить доступ к ресурсам, которыми вы управляете в Azure DevOps, вы можете предоставить или отклонить разрешения для определенных пользователей, встроенных групп безопасности или групп, определенных в Azure Active Directory (Azure AD), если они интегрированы с Azure DevOps или Azure AD, если они интегрированы с TFS.

- [Как хранить код в Azure DevOps](/azure/devops/repos/git/gitworkflow)

- [О разрешениях и группах в Azure DevOps](/azure/devops/organizations/security/about-permissions)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7,9. Реализация автоматического мониторинга конфигурации для ресурсов Azure

**Руководство**. Использование псевдонимов политик Azure в пространстве имен Microsoft. Data Lake Analytics для создания настраиваемых политик для оповещения, аудита и принудительного применения конфигураций системы. Используйте политики Azure [аудит], [запретить] и [развернуть, если не существует] для автоматического применения конфигураций для ресурсов Azure Data Lake Analytics.

- [Настройка Политики Azure и управление ею](../governance/policy/tutorials/create-and-manage.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13. Устранение непреднамеренного раскрытия учетных данных

**Руководство**. Реализуйте сканер учетных данных для обнаружения учетных данных в коде. Сканер учетных данных также рекомендует перемещать обнаруженные учетные данные в более безопасные расположения, такие как Azure Key Vault.

- [Как настроить сканер учетных данных](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="malware-defense"></a>Защита от вредоносных программ

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: защита от вредоносных программ](../security/benchmarks/security-control-malware-defense.md).*

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2. Предварительная проверка файлов для отправки в ресурсы Azure, не являющиеся вычислительными

**Руководство**. антивредоносное по Майкрософт включено на базовом узле, поддерживающем службы Azure (например, Azure Data Lake Analytics), но не выполняется в содержимом клиента.

Предварительное сканирование любого содержимого, отправляемого в ресурсы Azure, таких как служба приложений, Data Lake Analytics, хранилище BLOB-объектов и т. д. Корпорация Майкрософт не может получить доступ к данным в этих экземплярах.

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="data-recovery"></a>Восстановление данных

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: восстановление данных](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9,1: Обеспечьте регулярное автоматическое резервное копирование

**Руководство**. Data Lake Analytics журналы заданий и выходные данные хранятся в базовой службе Data Lake Storage 1-го поколения.  Вы можете использовать различные методы для копирования данных, включая ADLCopy, Azure PowerShell или фабрику данных Azure.  Службу автоматизации Azure также можно использовать для автоматической архивации данных на регулярной основе.

- [Управление ресурсами Azure Data Lake Storage 1-го поколения с помощью обозревателя службы хранилища](../data-lake-store/data-lake-store-in-storage-explorer.md)

- [Копирование данных из больших двоичных объектов службы хранилища Azure в Azure Data Lake Storage 1-го поколения](../data-lake-store/data-lake-store-copy-data-azure-storage-blob.md)

- [Обзор службы автоматизации Azure](../automation/automation-intro.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9,2: выполните полное резервное копирование системы и резервное копирование ключей, управляемых клиентом

**Руководство**. Data Lake Analytics журналы заданий и выходные данные хранятся в базовой службе Data Lake Storage 1-го поколения.  Вы можете использовать различные методы для копирования данных, включая ADLCopy, Azure PowerShell или фабрику данных Azure.  

- [Управление ресурсами Azure Data Lake Storage 1-го поколения с помощью обозревателя службы хранилища](../data-lake-store/data-lake-store-in-storage-explorer.md)

- [Копирование данных из больших двоичных объектов службы хранилища Azure в Azure Data Lake Storage 1-го поколения](../data-lake-store/data-lake-store-copy-data-azure-storage-blob.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9,3: Проверьте все резервные копии, включая управляемые клиентом ключи.

**Руководство**. периодически выполняется восстановление данных резервной копии для проверки целостности данных.

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9,4: Обеспечьте защиту резервных копий и ключей, управляемых клиентом

**Руководство**. Data Lake Analytics резервные копии, хранящиеся в Data Lake Storage 1-го поколения или в службе хранилища Azure, по умолчанию поддерживают шифрование и не могут быть отключены. Резервные копии следует рассматривать как конфиденциальные данные и применять соответствующие элементы управления доступом и защиты данных в рамках этого базового плана.  

- [Обеспечение безопасности в Azure Data Lake Storage 1-го поколения](../data-lake-store/data-lake-store-secure-data.md)

- [Авторизация доступа к данным в службе хранилища Azure](../storage/common/storage-auth.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="incident-response"></a>реагирование на инциденты.

*Дополнительные сведения см. в статье [Azure Security Benchmark: реагирование на инциденты](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1. Создание руководства по реагированию на инциденты

**Руководство**. Создайте руководство по реагированию на инциденты для вашей организации. Убедитесь в том, что имеются письменные планы реагирования на инциденты, которые определяют все действия персонала, а также этапы обработки инцидентов и управления ими для проверки после инцидента.

- [Руководство по созданию собственного процесса реагирования на инциденты безопасности](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Анатомия инцидента Центра Майкрософт по реагированию на угрозы](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)

- [Руководством по обработке инцидентов компьютерной безопасности NIST](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2. Создание процедуры оценки инцидента и определения приоритетов

**Руководство**. Центр безопасности назначает каждому оповещению серьезность, которая поможет определить, какие предупреждения следует расследовать первыми. Серьезность основывается на том, насколько надежным является центр безопасности в поиске или аналитике, используемой для выдаче оповещения, а также об уровне достоверности, который был вредоносным намерением для действия, вызвавшего оповещение. 

Кроме того, четко помечайте подписки тегами (например, рабочие, нерабочие) и создайте систему именования, чтобы четко определить и классифицировать ресурсы Azure, особенно обрабатывающие конфиденциальные данные.  Вы несете ответственность за назначение приоритета оповещениям, требующим действий по исправлению, в зависимости от важности ресурсов Azure и среды, в которой произошел инцидент.

- [Оповещения безопасности в Центре безопасности Azure](../security-center/security-center-alerts-overview.md)

- [использование тегов для упорядочения ресурсов в Azure](../azure-resource-manager/management/tag-resources.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="103-test-security-response-procedures"></a>10.3. Проверка процедур реагирования на угрозы

**Руководство**. проведение упражнений по тестированию возможностей реагирования на инциденты систем с регулярным интервалом для защиты ресурсов Azure. Выявление слабых точек и пропусков, а затем при необходимости пересмотрите свой план ответа. 

- [Публикация NIST — руководства по тестированию, обучению и выполнению упражнений для ИТ-планов и возможностей](https://csrc.nist.gov/publications/detail/sp/800-84/final)

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

**Рекомендации**. Следуйте правилам тестирования уязвимости Microsoft Cloud, чтобы убедиться, что тесты на проникновение не нарушают политики Майкрософт. Используйте стратегию Майкрософт и рекомендации "красных команд", а затем выполните тест на проникновение в режиме реального времени для управляемых корпорацией Майкрософт облачной инфраструктуры, служб и приложений. 

- [Правила взаимодействия при выполнении тестирования на проникновение](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1) 

- [Привлечение "красных команд для тестирования "Microsoft Cloud](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Ответственность**: Совмещаемая блокировка

**Мониторинг центра безопасности Azure**: нет

## <a name="next-steps"></a>Следующие шаги

- См. [Обзор Azure Security Benchmark версии 2](../security/benchmarks/overview.md)
- Дополнительные сведения о [базовой конфигурации безопасности Azure](../security/benchmarks/security-baselines-overview.md).