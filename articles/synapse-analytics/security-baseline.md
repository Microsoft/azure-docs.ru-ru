---
title: Базовые показатели безопасности Azure для синапсе Analytics
description: Базовый план безопасности синапсе Analytics содержит практические руководства и ресурсы для реализации рекомендаций по безопасности, указанных в статье о производительности системы безопасности Azure.
author: msmbaldwin
ms.service: synapse-analytics
ms.topic: conceptual
ms.date: 03/16/2021
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 323ddfc7d595bd0d2321660e3b4141444db20518
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104586933"
---
# <a name="azure-security-baseline-for-synapse-analytics"></a>Базовые показатели безопасности Azure для синапсе Analytics

Этот базовый план безопасности применяет рекомендации из [контрольного показателя безопасности Azure версии 1,0](../security/benchmarks/overview-v1.md) к синапсе Analytics. Azure Security Benchmark содержит рекомендации по обеспечению безопасности облачных решений в Azure.
Содержимое группируются по **элементам управления безопасностью** , определенным в ходе тестирования системы безопасности Azure, и связанным рекомендациям, применимым к синапсе Analytics. **Элементы управления** , неприменимые к синапсе Analytics, были исключены.

 
Сведения о том, как синапсе Analytics полностью сопоставлены с тестовым показателем безопасности Azure, см. в статье [полный базовый план безопасности синапсе Analytics](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Безопасность сети

*Дополнительные сведения см. в статье [Azure Security Benchmark: безопасность сети](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1,1: защита ресурсов Azure в виртуальных сетях

**Руководство**. Защита SQL Server Azure в виртуальной сети с помощью частной ссылки. Частная ссылка Azure позволяет получить доступ к службам Azure PaaS через закрытую конечную точку в виртуальной сети. Трафик между виртуальной сетью и службой проходит через магистральную сеть Майкрософт.

Кроме того, при подключении к пулу синапсе SQL Ограничьте область исходящего подключения к базе данных SQL с помощью группы безопасности сети. Отключите весь трафик службы Azure к базе данных SQL через общедоступную конечную точку, установив флажок Разрешить службам Azure отключить. Убедитесь, что в правилах брандмауэра не разрешены общедоступные IP-адреса.

- [Общие сведения о частной ссылке Azure](../private-link/private-link-overview.md)

- [Общие сведения о закрытой ссылке для Azure синапсе SQL](../azure-sql/database/private-endpoint-overview.md)

- [Создание виртуальной сети](../virtual-network/quick-create-portal.md)

- [Создание NSG с конфигурацией безопасности](../virtual-network/tutorial-filter-network-traffic.md)

**Ответственность**: Customer

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. SQL**:

[!INCLUDE [Resource Policy for Microsoft.Sql 1.1](../../includes/policy/standards/asb/rp-controls/microsoft.sql-1-1.md)]

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-network-interfaces"></a>1,2. Мониторинг и запись конфигурации и трафика виртуальных сетей, подсетей и сетевых интерфейсов

**Руководство**. при подключении к выделенному пулу SQL и включении журналов потоков для группы безопасности сети (NSG) журналы отправляются в учетную запись хранения Azure для аудита трафика.

Вы также можете отправить журналы потоков для группы безопасности сети в рабочую область Log Analytics и использовать аналитику трафика для получения ценных сведений о потоке трафика в облаке Azure. Некоторые преимущества Аналитики трафика — это возможность визуализировать сетевые активности и определять горячие участки, выявлять угрозы безопасности, анализировать шаблоны потоков трафика и выявлять некорректные сетевые настройки.

- [Как включить журналы потоков NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Общие сведения о безопасности сети, предоставляемой центром безопасности Azure](../security-center/security-center-network-recommendations.md)

- [Как включать и использовать Аналитику трафика](../network-watcher/traffic-analytics.md)

- [Общие сведения о безопасности сети, предоставляемой центром безопасности Azure](../security-center/security-center-network-recommendations.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1,4: запретите обмен данными с известными вредоносными IP-адресами

**Руководство**. Использование расширенной защиты от угроз (ATP) для Azure синапсе SQL. ATP обнаруживает аномальные действия, указывающие на необычные и потенциально опасные попытки доступа к базам данных или их использования, и может запускать различные предупреждения, такие как «потенциальное внедрение кода SQL» и «доступ из необычного расположения». ATP является частью предложения расширенной системы безопасности данных (ADS), доступ к которому можно получить и управлять ими с помощью центрального портала SQL AD.

Включите стандарт защиты от атак DDoS в виртуальных сетях, связанных с Azure синапсе SQL, для защиты от распределенных атак типа "отказ в обслуживании". Используйте интегрированную аналитику угроз Центра безопасности Azure, чтобы запретить обмен данными с известными вредоносными или неиспользуемыми IP-адресами Интернета.

- [Общие сведения о ATP для Azure синапсе SQL](../azure-sql/database/threat-detection-overview.md)

- [Как включить расширенную защиту данных для базы данных SQL Azure](../azure-sql/database/azure-defender-for-sql.md)

- [Общие сведения об ADS](../azure-sql/database/azure-defender-for-sql.md)

- [Настройка защиты от атак DDoS](../ddos-protection/manage-ddos-protection.md)

- [Общие сведения об интегрированной аналитике угроз в Центре безопасности Azure](../security-center/azure-defender.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="15-record-network-packets"></a>1,5: запись сетевых пакетов

**Руководство**. при подключении к выделенному пулу SQL и включении журналов потоков для группы безопасности сети (NSG) отправьте журналы в учетную запись хранения Azure для аудита трафика. Кроме того, можно отправить журналы потоков в Log Analytics рабочую область или передать их в концентраторы событий.  Если требуется для изучения аномальных действий, включите запись пакетов наблюдателя за сетями.

- [Как включить журналы потоков NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Как включить Наблюдатель за сетями](../network-watcher/network-watcher-create.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1,6: развертывание системы обнаружения вторжений на основе сети и предотвращения вторжения (ИДЕНТИФИКАТОРы и IP-адреса)

**Руководство**. Использование расширенной защиты от угроз (ATP) для Azure синапсе SQL. ATP обнаруживает аномальные действия, указывающие на необычные и потенциально опасные попытки доступа к базам данных или их использования, и может запускать различные предупреждения, такие как «потенциальное внедрение кода SQL» и «доступ из необычного расположения». ATP является частью предложения расширенной системы безопасности данных (ADS), доступ к которому можно получить и управлять ими с помощью центрального портала SQL AD. ATP также интегрирует предупреждения с центром безопасности Azure.

- [Общие сведения о ATP для Azure синапсе SQL](../azure-sql/database/threat-detection-overview.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8. Уменьшение сложности и дополнительных затрат на администрирование в правилах безопасности сети

**Руководство**. Использование тегов службы виртуальной сети для определения элементов управления доступом к сети для групп безопасности сети или брандмауэра Azure. Теги служб можно использовать вместо определенных IP-адресов при создании правил безопасности. Указав имя тега службы (например, ApiManagement) в соответствующем исходном поле или поле назначения правила, можно разрешить или запретить трафик для соответствующей службы. Корпорация Майкрософт управляет префиксами адресов, входящих в тег службы, и автоматически обновляет этот тег при изменении адресов.

При использовании конечной точки службы для выделенного пула SQL требуется исходящий трафик к общедоступным IP-адресам базы данных SQL Azure. чтобы разрешить подключение, необходимо открыть группы безопасности сети (группы безопасности сети) для прав доступа к базе данных SQL Azure. Это можно сделать с помощью тегов службы NSG для базы данных SQL Azure.

- [Общие сведения о тегах служб с конечными точками службы для базы данных SQL Azure](https://docs.microsoft.com/azure/azure-sql/database/vnet-service-endpoint-rule-overview#limitations)

- [Общие сведения и использование тегов служб](../virtual-network/service-tags-overview.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9. Поддержание стандартных конфигураций безопасности для сетевых устройств

**Руководство**. Определение и реализация конфигураций сетевой безопасности для ресурсов, связанных с выделенным пулом SQL, с помощью политики Azure. Вы можете использовать пространство имен Microsoft. SQL для определения настраиваемых определений политик или использовать любое из встроенных определений политик, предназначенных для защиты базы данных SQL Azure и сети сервера. Пример применимой встроенной политики безопасности сети для сервера базы данных SQL Azure: "SQL Server должен использовать конечную точку службы виртуальной сети".

Используйте схемы Azure для упрощения крупномасштабных развертываний Azure с помощью ключевых артефактов среды пакетов, таких как шаблоны управления ресурсами Azure, управление доступом на основе ролей Azure (Azure RBAC) и политики, в одном определении схемы. Простое применение схемы к новым подпискам и средам, а также управление точной настройкой и управление с помощью версионирования.

- [Настройка Политики Azure и управление ею](../governance/policy/tutorials/create-and-manage.md)

- [Создание схемы Azure](../governance/blueprints/create-blueprint-portal.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="110-document-traffic-configuration-rules"></a>1.10. Документация по правилам конфигурации трафика

**Руководство**. Использование тегов для групп безопасности сети (NSG) и других ресурсов, относящихся к сетевой безопасности и потоку трафика. Для отдельных правил NSG используйте поле "Описание", чтобы указать бизнес-потребности и длительность (и т. д.) любых правил, которые разрешают трафик в сеть и из нее.

Используйте любое из встроенных определений политик Azure, связанных с тегами, например "требовать тег и его значение", чтобы убедиться, что все ресурсы созданы с помощью тегов и уведомлять вас о существующих ресурсах без тегов.

Вы можете использовать Azure PowerShell или Azure CLI для поиска или выполнения действий с ресурсами на основе их тегов.

- [Создание и использование тегов](../azure-resource-manager/management/tag-resources.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11. Использование автоматизированных средств для мониторинга конфигураций сетевых ресурсов и обнаружения изменений

**Руководство**. Использование журнала действий Azure для мониторинга конфигураций сетевых ресурсов и обнаружения изменений сетевых ресурсов, связанных с выделенным пулом SQL. Создавайте оповещения в Azure Monitor, которые будут запускаться при изменении критических сетевых ресурсов.

- [Как просматривать и извлекать события журнала действий Azure](https://docs.microsoft.com/azure/azure-monitor/essentials/activity-log#view-the-activity-log)

- [Как создать оповещения в службе Azure Monitor](../azure-monitor/alerts/alerts-activity-log.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="logging-and-monitoring"></a>Ведение журналов и мониторинг

*Дополнительные сведения см. в статье [производительность системы безопасности Azure: ведение журнала и мониторинг](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="22-configure-central-security-log-management"></a>2.2. Настройка централизованного управления журналами безопасности

**Руководство**. Политика аудита может быть определена для конкретной базы данных или в качестве политики сервера по умолчанию в Azure (где размещается Azure синапсе). Политика сервера применяется ко всем существующим и новым базам данных на сервере.

Если включен аудит для сервера, он обязательно применяется к базе данных. Аудит базы данных будет выполнен независимо от его параметров.

При включении аудита их можно записать в журнал аудита в учетной записи хранения Azure, Log Analytics рабочей области или в концентраторах событий.

Кроме того, вы можете включить и подключить данные к Azure Sentinel или сторонним SIEM.

- [Настройка аудита для ресурсов SQL Azure](https://docs.microsoft.com/azure/azure-sql/database/auditing-overview#server-vs-database-level)

- [Подключение к Azure Sentinel](../sentinel/quickstart-onboard.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3. Включение журналов аудита для ресурсов Azure

**Руководство**. Включите аудит на уровне сервера Azure SQL для выделенного пула SQL и выберите место хранения журналов аудита (служба хранилища Azure, log Analytics или концентраторы событий).

Аудит может быть включен на уровне базы данных или сервера и предложен для включения только на уровне сервера, если только не требуется настройка отдельного приемника данных или хранения для конкретной базы данных.

- [Включение аудита для базы данных SQL Azure](../azure-sql/database/auditing-overview.md)

- [Включение аудита для сервера](https://docs.microsoft.com/azure/azure-sql/database/auditing-overview#setup-auditing)

- [Различия в политиках аудита уровня сервера и базы данных](https://docs.microsoft.com/azure/azure-sql/database/auditing-overview#server-vs-database-level)

**Ответственность**: Customer

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. SQL**:

[!INCLUDE [Resource Policy for Microsoft.Sql 2.3](../../includes/policy/standards/asb/rp-controls/microsoft.sql-2-3.md)]

### <a name="25-configure-security-log-storage-retention"></a>2.5. Настройка хранения журнала безопасности

**Рекомендации**. при хранении журналов, связанных с выделенным пулом SQL в учетной записи хранения, log Analytics рабочей области или концентраторах событий, задайте срок хранения журнала согласно нормативным требованиям Организации.

- [Управление жизненным циклом хранилища BLOB-объектов Azure (предварительная версия)](../storage/blobs/storage-lifecycle-management-concepts.md)

- [Настройка параметров хранения журнала в Log Analytics рабочей области](https://docs.microsoft.com/azure/azure-monitor/logs/manage-cost-storage#change-the-data-retention-period)

- [Запись потоковых событий в концентраторах событий](../event-hubs/event-hubs-capture-overview.md)

**Ответственность**: Customer

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. SQL**:

[!INCLUDE [Resource Policy for Microsoft.Sql 2.5](../../includes/policy/standards/asb/rp-controls/microsoft.sql-2-5.md)]

### <a name="26-monitor-and-review-logs"></a>2,6: мониторинг и просмотр журналов

**Руководство**. анализ и мониторинг журналов для аномальных поведений и регулярная проверка результатов. Используйте расширенную защиту от угроз для базы данных SQL Azure в сочетании с центром безопасности Azure, чтобы получать оповещения о необычных действиях, связанных с базой данных SQL. Кроме того, можно настроить оповещения на основе значений метрик или записей журнала действий Azure, связанных с базой данных SQL.

Кроме того, вы можете включить и подключить данные к Azure Sentinel или сторонним SIEM.

- [Общие сведения о расширенной защите от угроз и предупреждениях для базы данных SQL Azure](../azure-sql/database/threat-detection-overview.md)

- [Как включить расширенную защиту данных для базы данных SQL Azure](../azure-sql/database/azure-defender-for-sql.md)

- [Настройка пользовательских предупреждений для базы данных SQL Azure](../azure-sql/database/alerts-insights-configure-portal.md)

- [Подключение к Azure Sentinel](../sentinel/quickstart-onboard.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="27-enable-alerts-for-anomalous-activities"></a>2,7: Включение оповещений для аномальных действий

**Руководство**. Использование расширенной защиты от угроз (ATP) для базы данных SQL Azure в сочетании с центром безопасности Azure для мониторинга и оповещения о аномальных действиях. ATP является частью предложения расширенной системы безопасности данных (ADS), и к ним можно получить доступ и управлять ими с помощью центрального БАННЕРа SQL на портале. Реклама включает функции обнаружения и классификации конфиденциальных данных, отображая и устранения потенциальных уязвимостей баз данных, а также обнаружения аномальных действий, которые могут указывать на угрозу для базы данных.

Кроме того, вы можете включить и подключить данные в Azure Sentinel.

- [Общие сведения о расширенной защите от угроз и предупреждениях для базы данных SQL Azure](../azure-sql/database/threat-detection-overview.md)

- [Как включить расширенную защиту данных для базы данных SQL Azure](../azure-sql/database/azure-defender-for-sql.md)

- [Управление оповещениями в центре безопасности Azure](../security-center/security-center-managing-and-responding-alerts.md)

- [Подключение к Azure Sentinel](../sentinel/quickstart-onboard.md)

**Ответственность**: Customer

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. SQL**:

[!INCLUDE [Resource Policy for Microsoft.Sql 2.7](../../includes/policy/standards/asb/rp-controls/microsoft.sql-2-7.md)]

## <a name="identity-and-access-control"></a>Идентификатор и управление доступом

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: идентификация и управление доступом](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1. Инвентаризация учетных записей администраторов

**Руководство**. Проверка подлинности пользователей осуществляется с помощью Azure Active Directory (Azure AD) или SQL.

При первом развертывании SQL Azure укажите имя входа администратора и соответствующий пароль для этого имени входа. Эта административная учетная запись называется администратором сервера. Учетные записи администратора для базы данных можно найти, открыв портал Azure и перейдя на вкладку Свойства сервера или управляемого экземпляра. Вы также можете настроить учетную запись администратора Azure AD с полными правами администратора, если вы хотите включить проверку подлинности Azure AD.

Для операций управления используйте встроенные роли Azure, которые должны быть явно назначены. Используйте модуль Azure AD PowerShell для выполнения специальных запросов для обнаружения учетных записей, входящих в группы администраторов.

- [Проверка подлинности для базы данных SQL](https://docs.microsoft.com/azure/azure-sql/database/security-overview#authentication)

- [Создание учетных записей для пользователей, не имеющих прав администратора](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage#create-accounts-for-non-administrator-users)

- [Использование учетной записи Azure AD для проверки подлинности](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage#create-additional-logins-and-users-having-administrative-permissions)

- [Как получить роль каталога в Azure AD с помощью PowerShell](/powershell/module/azuread/get-azureaddirectoryrole)

- [Как получить членов роли каталога в Azure AD с помощью PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember)

- [Управление существующими именами входа и учетными записями администратора в SQL Azure](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage#existing-logins-and-user-accounts-after-creating-a-new-database)

- [Встроенные роли Azure](../role-based-access-control/built-in-roles.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="32-change-default-passwords-where-applicable"></a>3.2. Изменение паролей по умолчанию, где применимо

**Руководство**. Azure Active Directory (Azure AD) не имеет концепции паролей по умолчанию. При подготовке выделенного пула SQL рекомендуется интегрировать проверку подлинности с Azure AD. При использовании этого метода проверки подлинности пользователь отправляет имя учетной записи пользователя и запрашивает, что служба использует учетные данные, хранящиеся в Azure AD.

- [Настройка аутентификации Azure AD и управление ею с помощью Azure SQL](../azure-sql/database/authentication-aad-configure.md)

- [Общие сведения о проверке подлинности в SQL Azure](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage#existing-logins-and-user-accounts-after-creating-a-new-database)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="33-use-dedicated-administrative-accounts"></a>3.3. Применение выделенных административных учетных записей

**Руководство**. Создание политик и процедур, связанных с использованием выделенных административных учетных записей. Использование управления удостоверениями и доступом центра безопасности Azure для мониторинга количества учетных записей администраторов, которые входят в систему с помощью Azure Active Directory (Azure AD).

Чтобы найти учетные записи администратора для базы данных, откройте портал Azure и перейдите на вкладку Свойства сервера или управляемого экземпляра.

- [Общие сведения об удостоверениях и доступе центра безопасности Azure](../security-center/security-center-identity-access.md)

- [Управление существующими именами входа и учетными записями администратора в SQL Azure](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage#existing-logins-and-user-accounts-after-creating-a-new-database)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="34-use-azure-active-directory-single-sign-on-sso"></a>3,4. Использование единого входа Azure Active Directory (SSO)

**Руководство**. Использование регистрации приложений Azure (субъекта-службы) для получения маркера, который можно использовать для взаимодействия с хранилищем данных на плоскости управления (портал Azure) через вызовы API.

- [Как вызывать интерфейсы API службы Azure](/rest/api/azure/#how-to-call-azure-rest-apis-with-postman)

- [Как зарегистрировать клиентское приложение (субъект-служба) с помощью Azure Active Directory (Azure AD)](/rest/api/azure/#register-your-client-application-with-azure-ad)

- [Сведения о REST API SQL Azure синапсе](sql-data-warehouse/sql-data-warehouse-manage-compute-rest-api.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3,5: используйте многофакторную проверку подлинности для всех Azure Active Directory доступа

**Руководство**. Включение многофакторной проверки подлинности в Azure Active Directory (Azure AD) и следование рекомендациям по управлению удостоверениями и доступом в центре безопасности Azure.

- [Как включить многофакторную проверку подлинности в Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Мониторинг идентификации и доступа в Центре безопасности Azure](../security-center/security-center-identity-access.md)

- [Общие сведения о многофакторной проверке подлинности в SQL Azure](../azure-sql/database/authentication-mfa-ssms-overview.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="36-use-secure-azure-managed-workstations-for-administrative-tasks"></a>3,6: используйте защищенные рабочие станции, управляемые Azure, для административных задач

**Руководство**. Использование рабочей станции привилегированного доступа (привилегированным доступом) с многофакторной проверкой подлинности, настроенной для входа и настройки ресурсов Azure.

- [Использование рабочих станций с привилегированным доступом](https://4sysops.com/archives/understand-the-microsoft-privileged-access-workstation-paw-security-model/)

- [Как включить многофакторную проверку подлинности в Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3,7: журналы и оповещения о подозрительных действиях учетных записей администраторов

**Руководство**. Использование отчетов по безопасности Azure Active Directory (Azure AD) для создания журналов и оповещений при возникновении подозрительных или ненадежных действий в среде.

Используйте расширенную защиту от угроз для базы данных SQL Azure в сочетании с центром безопасности Azure для обнаружения и оповещения о аномальных действиях, указывающих на необычные и потенциально опасные попытки доступа к базам данных или их использования.

Подсистема аудита SQL Server позволяет проводить аудит сервера, который может предусматривать спецификации аудита серверов для событий на уровне сервера и спецификации аудита баз данных для событий на уровне базы данных. События аудита могут записываться в журналы событий или файлы аудита.

- [Как определить пользователей Azure AD, помеченных для события риска](../active-directory/identity-protection/overview-identity-protection.md)

- [Как отслеживать активность удостоверений и доступа пользователей в центре безопасности Azure](../security-center/security-center-identity-access.md)

- [Обзор расширенной защиты от угроз и потенциальных оповещений](https://docs.microsoft.com/azure/azure-sql/database/threat-detection-overview#alerts)

- [Общие сведения об именах входа и учетных записях пользователей в SQL Azure](../azure-sql/database/logins-create-manage.md)

- [Общие сведения об аудите SQL Server](/sql/relational-databases/security/auditing/sql-server-audit-database-engine)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8. Управление ресурсами Azure только из утвержденных расположений

**Руководство**. Используйте условный доступ с именованными расположениями, чтобы разрешить доступ к порталу и управлению ресурсами Azure только из конкретных логических групп диапазонов IP-адресов или стран и регионов.

- [Настройка именованных расположений в Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="39-use-azure-active-directory"></a>3.9. Использование Azure Active Directory

**Руководство**. создание администратора Azure Active Directory (Azure AD) для сервера базы данных SQL Azure в выделенном пуле SQL.

- [Настройка аутентификации Azure AD и управление ею с помощью Azure SQL](../azure-sql/database/authentication-aad-configure.md)

- [Создание и настройка экземпляра Azure AD](../active-directory-domain-services/tutorial-create-instance.md)

**Ответственность**: Customer

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. SQL**:

[!INCLUDE [Resource Policy for Microsoft.Sql 3.9](../../includes/policy/standards/asb/rp-controls/microsoft.sql-3-9.md)]

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10. Регулярная проверка и согласование доступа пользователей

**Руководство**. Azure Active Directory (Azure AD) предоставляет журналы для упрощения обнаружения устаревших учетных записей. Кроме того, с помощью проверок доступа Azure AD можно эффективно управлять членством в группах, доступом к корпоративным приложениям и назначениями ролей. Доступ пользователей можно регулярно проверять, чтобы убедиться, что доступ к ним имеют только нужные пользователи.

При использовании проверки подлинности SQL создайте пользователей автономной базы данных в базе данных. Убедитесь, что один или несколько пользователей базы данных помещаются в пользовательскую роль базы данных с конкретными разрешениями, соответствующими этой группе пользователей.

- [Использование проверок доступа](../active-directory/governance/access-reviews-overview.md)

- [Общие сведения об именах входа и учетных записях пользователей в SQL Azure](../azure-sql/database/logins-create-manage.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3,11: монитор пытается получить доступ к отключенным учетным данным

**Руководство**. Настройка проверки подлинности Azure Active Directory (Azure AD) с помощью SQL Azure и включение параметров диагностики для учетных записей пользователей Azure AD, отправка журналов аудита и журналов входа в рабочую область log Analytics. Настройте необходимые оповещения в Log Analytics.

При использовании проверки подлинности SQL создайте пользователей автономной базы данных в базе данных. Убедитесь, что один или несколько пользователей базы данных помещаются в пользовательскую роль базы данных с конкретными разрешениями, соответствующими этой группе пользователей.

- [Использование проверок доступа](../active-directory/governance/access-reviews-overview.md)

- [Как настроить аутентификацию Azure AD и управлять ею с помощью базы данных SQL Azure](../azure-sql/database/authentication-aad-configure.md)

- [Как интегрировать журналы действий Azure в Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

- [Общие сведения об именах входа и учетных записях пользователей в SQL Azure](../azure-sql/database/logins-create-manage.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3,12: предупреждение об отклонении при входе в учетную запись

**Руководство**. Использование функций защиты идентификации и средств обнаружения рисков Azure Active Directory (Azure AD) для настройки автоматических ответов на обнаруженные подозрительные действия, связанные с удостоверениями пользователей. Кроме того, вы также встроеем и принимаем данные в метку Azure для дальнейшего изучения.

При использовании проверки подлинности SQL создайте пользователей автономной базы данных в базе данных. Убедитесь, что один или несколько пользователей базы данных помещаются в пользовательскую роль базы данных с конкретными разрешениями, соответствующими этой группе пользователей.

- [Просмотр входа в систему с рисками Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Как настроить и включить политики рисков с помощью защиты идентификации](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Подключение к Azure Sentinel](../sentinel/connect-data-sources.md)

- [Общие сведения об именах входа и учетных записях пользователей в SQL Azure](../azure-sql/database/logins-create-manage.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13. Предоставление корпорации Майкрософт доступа к соответствующим данным клиентов в рамках сценариев поддержки

**Руководство**. в сценариях поддержки, в которых Майкрософт требуется доступ к данным, связанным с базой данных SQL Azure, в выделенном пуле SQL Azure защищенное хранилище предоставляет интерфейс для просмотра и утверждения или отклонения запросов на доступ к данным.

- [Общие сведения о защищенное хранилище](../security/fundamentals/customer-lockbox-overview.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="data-protection"></a>Защита данных

*Дополнительные сведения см. в статье [Azure Security Benchmark: защита данных](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1. Инвентаризация конфиденциальных данных

**Руководство**. Используйте теги для пометки ресурсов Azure, в которых хранятся или обрабатываются конфиденциальные данные.

Классификация обнаружения данных &amp; встроена в Azure СИНАПСЕ SQL. Они предоставляют расширенные возможности для обнаружения, классификации и маркировки конфиденциальных данных в вашей базе, а также создания по ним отчетности.

- [Создание и использование тегов](../azure-resource-manager/management/tag-resources.md)

- [Общие сведения о &amp; классификации обнаружения данных](../azure-sql/database/data-discovery-and-classification-overview.md)

**Ответственность**: Customer

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. SQL**:

[!INCLUDE [Resource Policy for Microsoft.Sql 4.1](../../includes/policy/standards/asb/rp-controls/microsoft.sql-4-1.md)]

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2. Изолирование систем, хранящих или обрабатывающих конфиденциальные данные

**Руководство**. Реализуйте отдельные подписки и группы управления для разработки, тестирования и производства. Ресурсы должны быть разделены виртуальной сетью или подсетью, помечены соответствующим образом и защищены в рамках группы безопасности сети или брандмауэра Azure. Ресурсы, которые хранят или обрабатывают конфиденциальные данные, должны быть изолированы. Использовать частную ссылку; развертывание SQL Server Azure в виртуальной сети и безопасное подключение с помощью частной ссылки.

- [Создание дополнительных подписок Azure](../cost-management-billing/manage/create-subscription.md)

- [Создание групп управления](../governance/management-groups/create-management-group-portal.md)

- [Создание и использование тегов](../azure-resource-manager/management/tag-resources.md)

- [Настройка Приватного канала для Базы данных SQL Azure](../azure-sql/database/private-endpoint-overview.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. Мониторинг и блокирование несанкционированной передачи конфиденциальной информации

**Руководство**. для любой базы данных SQL Azure в выделенном пуле SQL, в которой хранятся или обрабатываются конфиденциальные данные, пометьте базу данных и связанные ресурсы как конфиденциальные с помощью тегов. Настройте закрытую ссылку в сочетании с тегами службы группы безопасности сети (NSG) в экземплярах базы данных SQL Azure, чтобы предотвратить утечка конфиденциальной информации.

Кроме того, расширенная защита от угроз для базы данных SQL Azure, Управляемый экземпляр Azure SQL и Azure синапсе обнаруживает аномальные действия, указывающие на необычные и потенциально опасные попытки доступа к базам данных или их использования.

Для базовой платформы, управляемой корпорацией Майкрософт, корпорация Майкрософт считает все содержимое клиента конфиденциальным и предпринимает все возможные усилия для защиты клиентов от потери данных и раскрытия информации. Чтобы обеспечить безопасность данных клиентов в Azure, корпорация Майкрософт реализовала и поддерживает набор надежных элементов управления и возможностей защиты данных.

- [Настройка частных ссылок и группы безопасности сети для предотвращения утечка данных в экземплярах базы данных SQL Azure](../azure-sql/database/private-endpoint-overview.md)

- [Общие сведения о расширенной защите от угроз для базы данных SQL Azure](../azure-sql/database/threat-detection-overview.md)

- [Общие сведения о защите данных клиентов в Azure](../security/fundamentals/protection-customer-data.md)

**Ответственность**: Совмещаемая блокировка

**Мониторинг центра безопасности Azure**: нет

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5. Использование средства активного обнаружения для распознавания конфиденциальных данных

**Руководство**. Использование функции классификации обнаружения данных SQL Azure синапсе &amp; . Классификация обнаружения данных &amp; предоставляет расширенные возможности, встроенные в базу данных SQL Azure, для обнаружения, классификации и добавления меток для &amp; защиты конфиденциальных данных в базах данных.

Классификация обнаружения данных &amp; является частью расширенного предложения по обеспечению безопасности данных, который является единым пакетом для расширенных возможностей обеспечения безопасности SQL. &amp;Доступ к классификации обнаружения данных и управление ими осуществляется с помощью центрального портала SQL AD.

Кроме того, можно настроить политику динамического маскирования данных (DDM) в портал Azure. Обработчик рекомендаций DDM помечает определенные поля базы данных как потенциально конфиденциальные поля, которые могут быть хорошими кандидатами для маскировки.

- [Как использовать обнаружение и классификацию данных для Azure SQL Server](../azure-sql/database/data-discovery-and-classification-overview.md)

- [Общие сведения о динамическом маскировании данных для Azure синапсе SQL](../azure-sql/database/dynamic-data-masking-overview.md)

**Ответственность**: Customer

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. SQL**:

[!INCLUDE [Resource Policy for Microsoft.Sql 4.5](../../includes/policy/standards/asb/rp-controls/microsoft.sql-4-5.md)]

### <a name="46-use-azure-rbac-access-control-to-control-access-to-resources"></a>4,6. использование контроля доступа Azure RBAC для управления доступом к ресурсам 

**Руководство**. Использование управления доступом на основе ролей Azure (Azure RBAC) для управления доступом к базам данных SQL Azure в выделенном пуле SQL.

Авторизация контролируется членством в роли базы данных пользователя и разрешениями уровня объектов. Обычно пользователям рекомендуется предоставлять наименьшие необходимые привилегии.

- [Как интегрировать Azure SQL Server с Azure Active Directory (Azure AD) для проверки подлинности](../azure-sql/database/authentication-aad-overview.md)

- [Управление доступом в Azure SQL Server](../azure-sql/database/logins-create-manage.md)

- [Общие сведения о авторизации и проверке подлинности в SQL Azure](../azure-sql/database/logins-create-manage.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8. Шифрование конфиденциальной информации при хранении

**Руководство**. прозрачное шифрование данных (TDE) помогает защитить Azure синапсе SQL от угрозы вредоносной автономной активности, шифруя неактивных данных. Выполняется шифрование и расшифровка базы данных, связанных резервных копий и неактивных файлов журналов транзакций в реальном времени без необходимости изменения приложения. В Azure значение по умолчанию для TDE заключается в том, что DEK защищен встроенным сертификатом сервера. Кроме того, можно использовать управляемую клиентом TDE, также называемую поддержкой создание собственных ключей (BYOK) для TDE. В этом сценарии средство защиты TDE, которое шифрует DEK, является асимметричным ключом, управляемым клиентом, который хранится в принадлежащем заказчике и управляемом Azure Key Vaultе (облачной системе управления внешними ключами Azure) и никогда не покидает хранилище ключей.

- [Общие сведения о прозрачном шифровании данных, управляемых службой](../azure-sql/database/transparent-data-encryption-tde-overview.md)

- [Понимание управляемого клиентом прозрачного шифрования данных](https://docs.microsoft.com/azure/azure-sql/database/transparent-data-encryption-tde-overview#customer-managed-transparent-data-encryption---bring-your-own-key)

- [Как включить TDE с помощью собственного ключа](../azure-sql/database/transparent-data-encryption-byok-configure.md)

**Ответственность**: Совмещаемая блокировка

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. SQL**:

[!INCLUDE [Resource Policy for Microsoft.Sql 4.8](../../includes/policy/standards/asb/rp-controls/microsoft.sql-4-8.md)]

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9. Включение в журнал и создание оповещений по изменениям критических ресурсов Azure

**Руководство**. Использование Azure Monitor с журналом действий Azure для создания оповещений о том, когда изменения выполняются в рабочих экземплярах пулов SQL синапсе и других критических или связанных ресурсах.

Кроме того, вы можете настроить оповещения для баз данных в пуле синапсе SQL с помощью портал Azure. Оповещения могут отправить электронное сообщение или вызвать веб-перехватчик, когда какая-либо метрика достигает порога (например, размер базы данных или использование ЦП).

- [Создание оповещений для событий журнала действий Azure](../azure-monitor/alerts/alerts-activity-log.md)

- [Создание оповещений для Azure SQL синапсе](../azure-sql/database/alerts-insights-configure-portal.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="vulnerability-management"></a>Управление уязвимостями

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: Управление уязвимостью](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1. Выполнение автоматизированных средства анализа уязвимостей

**Рекомендации**. включите расширенную безопасность данных и следуйте рекомендациям центра безопасности Azure по выполнению оценки уязвимостей в базах данных SQL Azure.

- [Как запустить оценку уязвимостей в базах данных SQL Azure](../azure-sql/database/sql-vulnerability-assessment.md)

- [Включение расширенной защиты данных](../azure-sql/database/azure-defender-for-sql.md)

- [Реализация рекомендаций по оценке уязвимостей в центре безопасности Azure](../security-center/deploy-vulnerability-assessment-vm.md)

**Ответственность**: Customer

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. SQL**:

[!INCLUDE [Resource Policy for Microsoft.Sql 5.1](../../includes/policy/standards/asb/rp-controls/microsoft.sql-5-1.md)]

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4. Сравнение проверок смежных уязвимостей

**Руководство**. Оценка уязвимостей — это служба сканирования, встроенная в Azure синапсе SQL. Служба использует базу знаний правил, которые помечают уязвимости безопасности. Он выделяет отклонения от рекомендаций, таких как непредусмотренные настройки, чрезмерные разрешения и незащищенные конфиденциальные данные.  Можно получить доступ к оценке уязвимостей и управлять ими с помощью центрального портала SQL Advanced Data Security (ADS).

- [Управление сканированием оценки уязвимостей и их экспорт на портале ADS SQL](../azure-sql/database/sql-vulnerability-assessment.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5. Использование процесса оценки рисков для определения приоритета в устранении обнаруженных уязвимостей

**Руководство**. Используйте оценки рисков по умолчанию (Оценка безопасности), предоставляемые центром безопасности Azure.

Классификация обнаружения данных &amp; встроена в Azure СИНАПСЕ SQL. Они предоставляют расширенные возможности для обнаружения, классификации и маркировки конфиденциальных данных в вашей базе, а также создания по ним отчетности.

- [Оценка безопасности центра безопасности Azure](../security-center/secure-score-security-controls.md)

- [Общие сведения о &amp; классификации обнаружения данных](../azure-sql/database/data-discovery-and-classification-overview.md)

**Ответственность**: Customer

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. SQL**:

[!INCLUDE [Resource Policy for Microsoft.Sql 5.5](../../includes/policy/standards/asb/rp-controls/microsoft.sql-5-5.md)]

## <a name="inventory-and-asset-management"></a>Инвентаризация и управление ресурсами

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: Инвентаризация и управление активами](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6,1. Использование автоматизированного решения для обнаружения ресурсов

**Руководство**. Использование графа ресурсов Azure для запроса и обнаружения всех ресурсов, связанных с выделенным пулом SQL в ваших подписках. Убедитесь, что у вас есть соответствующие разрешения (на чтение) в клиенте и вы можете перечислить все подписки Azure, а также ресурсы в ваших подписках.

Хотя классические ресурсы Azure могут быть обнаружены с помощью графа ресурсов Azure, настоятельно рекомендуется создавать и использовать Azure Resource Manager ресурсы, идущие вперед.

- [Как создавать запросы с помощью Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

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

**Руководство**. Использование тегов, групп управления и отдельных подписок (при необходимости) для Организации и мониторинга ресурсов. Регулярно сверяйте ресурсы, чтобы своевременно удалять неавторизованные ресурсы из подписки.

- [Создание дополнительных подписок Azure](../cost-management-billing/manage/create-subscription.md)

- [Создание групп управления](../governance/management-groups/create-management-group-portal.md)

- [Создание и использование тегов](../azure-resource-manager/management/tag-resources.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6,4: определение и обслуживание инвентаризации утвержденных ресурсов Azure

**Руководство**. Определите список утвержденных ресурсов Azure, относящихся к выделенному пулу SQL.

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5. Отслеживание неутвержденных ресурсов Azure

**Руководство**. Использование политики Azure для ограничения типа ресурсов, которые могут быть созданы в клиентских подписках, с помощью следующих встроенных определений политик:

- Недопустимые типы ресурсов

- Допустимые типы ресурсов

Используйте граф ресурсов Azure для запроса или обнаружения ресурсов в ваших подписках. Убедитесь в том, что все ресурсы Azure, представленные в среде, утверждены.

- [Настройка Политики Azure и управление ею](../governance/policy/tutorials/create-and-manage.md)

- [Как создавать запросы с помощью Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="69-use-only-approved-azure-services"></a>6.9. Использование только утвержденных служб Azure

**Руководство**. Использование политики Azure для размещения ограничений на тип ресурсов, которые могут быть созданы в подписках клиента с помощью следующих встроенных определений политик:
- Недопустимые типы ресурсов
- Допустимые типы ресурсов

Используйте граф ресурсов Azure для запроса или обнаружения ресурсов в ваших подписках. Убедитесь в том, что все ресурсы Azure, представленные в среде, утверждены.

- [Настройка Политики Azure и управление ею](../governance/policy/tutorials/create-and-manage.md)

- [Как отказаться от определенного типа ресурса с помощью Политики Azure](https://docs.microsoft.com/azure/governance/policy/samples/built-in-policies#general)

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

**Руководство**. Использование псевдонимов политик Azure в пространстве имен Microsoft. SQL для создания настраиваемых политик для аудита или принудительного применения конфигурации ресурсов, связанных с выделенным пулом SQL. Вы также можете использовать встроенные определения политик для баз данных и серверов Azure, например:

- Развертывание обнаружения угроз на серверах SQL Server
- Служба SQL Server должна использовать конечную точку службы виртуальной сети.

- [Просмотр доступных псевдонимов политик Azure](/powershell/module/az.resources/get-azpolicyalias)

- [Настройка Политики Azure и управление ею](../governance/policy/tutorials/create-and-manage.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3. Сохранение безопасных конфигураций для ресурсов Azure

**Рекомендации**. Используйте Политику Azure [отказывать] и [развернуть, если не существует], чтобы обеспечить безопасность параметров в ресурсах Azure.

- [Настройка Политики Azure и управление ею](../governance/policy/tutorials/create-and-manage.md)

- [Сведения о действии Политик Azure](../governance/policy/concepts/effects.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5. Безопасное хранение конфигурации ресурсов Azure

**Руководство**. Если вы используете пользовательские определения политики Azure, используйте Azure DevOps или Azure Repos для безопасного хранения кода и управления им.

- [Как хранить код в Azure DevOps](/azure/devops/repos/git/gitworkflow)

- [Документация по Azure Repos](/azure/devops/repos/)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7,7: развертывание средств управления конфигурацией для ресурсов Azure

**Руководство**: неприменимо; В Azure синапсе SQL нет настраиваемых параметров безопасности.

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7,9. Реализация автоматического мониторинга конфигурации для ресурсов Azure

**Руководство**. Использование центра безопасности Azure для выполнения проверок базовых показателей для любых ресурсов, связанных с выделенным пулом SQL.

- [Как исправить рекомендации в центре безопасности Azure](../security-center/security-center-remediate-recommendations.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="711-manage-azure-secrets-securely"></a>7.11. Безопасное управление секретами Azure

**Руководство**. ПРОЗРАЧНОЕ шифрование данных (TDE) с ключами, управляемыми клиентом, в Azure Key Vault позволяет шифрование автоматически созданного ключа шифрования базы данных (DEK) с помощью управляемого клиентом асимметричного ключа, который называется предохранителем TDE. Обычно это также называется поддержкой создания собственных ключей (BYOK) для прозрачного шифрования данных. В сценарии BYOK средство защиты TDE хранится в принадлежащих к клиенту и управляемых Azure Key Vault. Кроме того, убедитесь, что в Azure Key Vault включен обратимое удаление.

- [Как включить TDE с управляемым клиентом ключом из Azure Key Vault](../azure-sql/database/transparent-data-encryption-byok-configure.md)

- [Включение обратимого удаления в Azure Key Vault](../key-vault/general/key-vault-recovery.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="712-manage-identities-securely-and-automatically"></a>7.12. Безопасное и автоматическое управление удостоверениями

**Руководство**. Использование управляемых удостоверений для предоставления служб Azure с автоматически управляемым удостоверением в Azure Active Directory (Azure AD). Управляемые удостоверения позволяют проходить проверку подлинности в любой службе, поддерживающей проверку подлинности Azure AD, включая Azure Key Vault без каких бы то ни было учетных данных в коде.

- [Руководство по Использование назначаемого системой управляемого удостоверения на виртуальной машине Windows для доступа к SQL Azure](../active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-sql.md)

- [Настройка управляемых удостоверений](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13. Устранение непреднамеренного раскрытия учетных данных

**Руководство**. Реализация средства проверки учетных данных для указания учетных данных в коде. Сканер учетных данных также рекомендует перемещать обнаруженные учетные данные в более безопасные расположения, такие как Azure Key Vault.

- [Как настроить сканер учетных данных](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="malware-defense"></a>Защита от вредоносных программ

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: защита от вредоносных программ](../security/benchmarks/security-control-malware-defense.md).*

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2. Предварительная проверка файлов для отправки в ресурсы Azure, не являющиеся вычислительными

**Руководство**. Защита от вредоносных программ Майкрософт включена на базовом узле, поддерживающем службы Azure (например, Azure синапсе SQL). Однако он не выполняется в содержимом клиента.

Предварительное сканирование любого содержимого, отправляемого в Нерасчетные ресурсы Azure, например служба приложений, Data Lake Storage, хранилище BLOB-объектов, SQL Server Azure и т. д. Корпорация Майкрософт не может получить доступ к данным в этих экземплярах.

- [Сведения о антивредоносном по Майкрософт для облачных служб и виртуальных машин Azure](../security/fundamentals/antimalware.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="data-recovery"></a>Восстановление данных

*Дополнительные сведения см. в статье о [производительности системы безопасности Azure: восстановление данных](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9,1: Обеспечьте регулярное автоматическое резервное копирование

**Руководство**. моментальные снимки выделенного пула SQL автоматически берутся в течение дня, создавая точки восстановления, доступные в течение семи дней. Этот срок нельзя изменить. Выделенный пул SQL поддерживает целевую точку восстановления в 8 часов (RPO). Вы можете восстановить хранилище данных в основном регионе до любого из моментальных снимков, доступных за последние семь дней. Обратите внимание, что при необходимости можно также активировать моментальные снимки вручную.

- [Резервное копирование и восстановление в выделенном пуле SQL](sql-data-warehouse/backup-and-restore.md)

**Ответственность**: Совмещаемая блокировка

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. SQL**:

[!INCLUDE [Resource Policy for Microsoft.Sql 9.1](../../includes/policy/standards/asb/rp-controls/microsoft.sql-9-1.md)]

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9,2: выполните полное резервное копирование системы и резервное копирование ключей, управляемых клиентом

**Руководство**. моментальные снимки хранилища данных автоматически берутся в течение дня, создавая точки восстановления, доступные в течение семи дней. Этот срок нельзя изменить. Выделенный пул SQL поддерживает целевую точку восстановления в 8 часов (RPO). Вы можете восстановить хранилище данных в основном регионе до любого из моментальных снимков, доступных за последние семь дней. Обратите внимание, что при необходимости можно также активировать моментальные снимки вручную.

Если вы используете ключ, управляемый клиентом, для шифрования ключа шифрования базы данных, убедитесь, что выполняется резервное копирование ключа.

- [Резервное копирование и восстановление в выделенном пуле SQL](sql-data-warehouse/backup-and-restore.md)

- [Резервное копирование ключей Azure Key Vault](/powershell/module/az.keyvault/backup-azkeyvaultkey)

**Ответственность**: Совмещаемая блокировка

**Мониторинг в центре безопасности Azure. производительность** [системы безопасности Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/governance/policy/samples/azure-security-benchmark.md) — это инициатива политики по умолчанию для центра безопасности, которая является основой для [рекомендаций центра безопасности](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/security-center-recommendations.md). Определения политик Azure, связанные с этим элементом управления, автоматически включаются центром безопасности. Для оповещений, связанных с этим элементом управления, может потребоваться план [защитника Azure](/home/mbaldwin/docs/asb/azure-docs-pr/articles/security-center/azure-defender.md) для связанных служб.

**Встроенные определения политики Azure — Microsoft. SQL**:

[!INCLUDE [Resource Policy for Microsoft.Sql 9.2](../../includes/policy/standards/asb/rp-controls/microsoft.sql-9-2.md)]

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9,3: Проверьте все резервные копии, включая управляемые клиентом ключи.

**Руководство**. Периодическая проверка точек восстановления для обеспечения допустимости моментальных снимков. Чтобы восстановить существующий выделенный пул SQL из точки восстановления, можно использовать либо портал Azure, либо PowerShell. Проверка восстановления резервных копий ключей, управляемых клиентом.

- [Восстановление ключей Azure Key Vault](/powershell/module/az.keyvault/restore-azkeyvaultkey)

- [Резервное копирование и восстановление в выделенном пуле SQL](sql-data-warehouse/backup-and-restore.md)

- [Восстановление существующего выделенного пула SQL](sql-data-warehouse/sql-data-warehouse-restore-active-paused-dw.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9,4: Обеспечьте защиту резервных копий и ключей, управляемых клиентом

**Руководство**. в базе данных SQL Azure можно настроить одну или несколько баз данных с помощью политики долгосрочного хранения резервных копий (LTR), чтобы автоматически хранить резервные копии баз данных в отдельных контейнерах хранилища BLOB-объектов Azure до 10 лет. Данные в службе хранилища Azure шифруются и расшифровываются прозрачно с использованием 256-разрядного шифрования AES, одним из наиболее подданных блоков блочных шифров и совместимым с FIPS 140-2.

По умолчанию данные в учетной записи хранения шифруются с помощью ключей, управляемых корпорацией Майкрософт. Вы можете использовать ключи, управляемые корпорацией Майкрософт, для шифрования данных, а также управлять шифрованием с помощью собственных ключей. Если вы управляете собственными ключами с помощью Key Vault, убедитесь, что включено обратимое удаление.

- [Управление долгосрочным хранением резервных копий базы данных SQL Azure](../azure-sql/database/long-term-backup-retention-configure.md)

- [Шифрование службы хранилища Azure для неактивных данных](../storage/common/storage-service-encryption.md)

- [Включение обратимого удаления в Key Vault](../storage/blobs/soft-delete-blob-overview.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="incident-response"></a>реагирование на инциденты.

*Дополнительные сведения см. в статье [Azure Security Benchmark: реагирование на инциденты](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1. Создание руководства по реагированию на инциденты

**Руководство**. Убедитесь, что имеются письменные планы реагирования на инциденты, определяющие роли персонала, а также этапы обработки инцидентов и управления ими.

- [Как настроить автоматизацию рабочих процессов в Центре безопасности Azure](../security-center/security-center-planning-and-operations-guide.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2. Создание процедуры оценки инцидента и определения приоритетов

**Руководство**. Центр безопасности назначает серьезность предупреждениям, чтобы помочь вам определить порядок, в котором вы задаете каждое оповещение, чтобы при компрометации ресурса вы могли сразу перейти к нему. Серьезность основывается на том, насколько надежным является центр безопасности в поиске или метрике, используемой для выдаче оповещения, а также на степень достоверности, которая была вредоносной задачей, вызвавшей оповещение.

- [Оповещения безопасности в Центре безопасности Azure](../security-center/security-center-alerts-overview.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="103-test-security-response-procedures"></a>10.3. Проверка процедур реагирования на угрозы

**Руководство**. проведение упражнений по тестированию возможностей реагирования на инциденты систем на регулярной ритмичности. Выявите слабые точки и пробелов и пересмотрите план по мере необходимости.

- [Вы можете обратиться к публикации NIST: руководство по тестированию, обучению и упражнениям в плане ИТ-планов и возможностей.](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4. Предоставление контактных сведений и настройка уведомлений по инцидентам безопасности

**Руководство**. контактные данные инцидентов безопасности будут использоваться корпорацией Майкрософт для связи с вами, если Microsoft Security Response Center (MSRC) обнаруживает, что доступ к данным был получен незаконные или неавторизованной стороной.

- [Как задать контакт безопасности Центра безопасности Azure](../security-center/security-center-provide-security-contact-details.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5. Включение оповещений системы безопасности в систему реагирования на инциденты

**Рекомендации**. Экспортируйте оповещения и рекомендации центра безопасности Azure с помощью функции непрерывного экспорта. Непрерывный экспорт позволяет экспортировать предупреждения и рекомендации как вручную, так и в постоянном, непрерывном режиме. Вы можете использовать соединитель данных Центра безопасности Azure для потоковой передачи оповещений в Azure Sentinel.

- [Настройка непрерывного экспорта данных](../security-center/continuous-export.md)

- [Как выполнить потоковую передачу оповещений в Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

### <a name="106-automate-the-response-to-security-alerts"></a>10.6. Автоматизация реагирования на оповещения системы безопасности

**Руководство**. Используйте функцию автоматизации рабочих процессов в Центре безопасности Azure для автоматического запуска реагирования с помощью Logic Apps в оповещениях и рекомендациях системы безопасности.

- [Как настроить автоматизацию рабочего процесса и Logic Apps](../security-center/workflow-automation.md)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="penetration-tests-and-red-team-exercises"></a>Тесты на проникновение и попытки нарушения безопасности "красной командой"

*Дополнительные сведения см. в статье [тесты производительности системы безопасности Azure: испытания на проникновение и команды красных команд](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11,1. Проведите регулярное тестирование на проникновение ресурсов Azure и обеспечьте исправление всех критических результатов безопасности.

**Рекомендации**. Следуйте правилам взаимодействия Майкрософт, чтобы убедиться, что тесты на проникновение не нарушают политики Майкрософт: https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1 .

- [Дополнительные сведения о стратегии корпорации Майкрософт и ее выполнении, а также о тестировании на основе уязвимости для управляемой облачной инфраструктуры, служб и приложений Майкрософт см. здесь.](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Ответственность**: Customer

**Мониторинг центра безопасности Azure**: нет

## <a name="next-steps"></a>Следующие шаги

- См. [Обзор Azure Security Benchmark версии 2](/azure/security/benchmarks/overview)
- Дополнительные сведения о [базовой конфигурации безопасности Azure](/azure/security/benchmarks/security-baselines-overview).
