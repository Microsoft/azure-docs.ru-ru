---
title: Базовая конфигурация системы безопасности Azure для Key Vault
description: Базовая конфигурация системы безопасности Azure для Key Vault
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 04/10/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: ad47ac7d51de6ab497d7061961c14b5324fab931
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98202459"
---
# <a name="azure-security-baseline-for-key-vault"></a>Базовая конфигурация системы безопасности Azure для Key Vault

Базовый план безопасности Azure для Key Vault содержит рекомендации, которые помогут повысить уровень безопасности развертывания.

Базовая конфигурация безопасности для этой службы взята из [теста производительности системы Azure версии 1.0](../../security/benchmarks/overview.md), содержащего рекомендации по обеспечению безопасности облачных решений в Azure с помощью наших практических рекомендаций.

Дополнительные сведения см. в статье [Обзор базовой конфигурации безопасности Azure](../../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Безопасность сети

*Дополнительные сведения см. в статье [Управление безопасностью: безопасность сети](../../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1. Защита ресурсов в виртуальной сети с помощью групп безопасности сети или Брандмауэра Azure

**Руководство**. Интеграция Azure Key Vault с помощью частной ссылки Azure. 

Служба "Частная связь Azure" позволяет получить доступ к службам Azure (например, Azure Key Vault) и службам клиента или партнера Azure, размещенным в частной конечной точке в виртуальной сети.

Частная конечная точка Azure — это сетевой интерфейс, который защищенно и надежно подключается к службе через Приватный канал Azure. Частная конечная точка использует частный IP-адрес из виртуальной сети, по сути перемещая службу в виртуальную сеть. Весь трафик к службе может маршрутизироваться через частную конечную точку, поэтому шлюзы, устройства преобразования сетевых адресов (NAT), подключения ExpressRoute и VPN, а также общедоступные IP-адреса не требуются. Трафик между виртуальной сетью и службой проходит через магистральную сеть Майкрософт, что позволяет избежать рисков общедоступного Интернета. Вы можете подключиться к экземпляру ресурса Azure, обеспечивая наивысшую степень детализации в управлении доступом.

Как интегрировать Key Vault с помощью частной ссылки Azure:

https://docs.microsoft.com/azure/key-vault/private-link-service


**Мониторинг Центра безопасности Azure**: Да

**Ответственность**: Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2. Мониторинг и запись конфигурации и трафика виртуальных сетей, подсетей и сетевых карт

**Руководство**. Используйте центр безопасности Azure и следуйте рекомендациям по защите сети для защиты ресурсов, настроенных Key Vault в Azure. 

Дополнительные сведения о безопасности сети, предоставляемой центром безопасности Azure: 

https://docs.microsoft.com/azure/security-center/security-center-network-recommendations

**Мониторинг Центра безопасности Azure**: Да

**Ответственность**: Customer

### <a name="13-protect-critical-web-applications"></a>1.3. Защита критических веб-приложений

**Рекомендации**. Неприменимо. Эта рекомендация предназначена для веб-приложений, работающих в службе приложений Azure или в ресурсах вычислений.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4. Запрет взаимодействия с известными опасными IP-адресами

**Руководство**. Включение стандарта защиты Azure от атак DDoS в виртуальных сетях Azure, связанных с экземплярами Key Vault, для защиты от атак типа "отказ в обслуживании". Используйте интегрированную аналитику угроз Центра безопасности Azure, чтобы запретить обмен данными с известными вредоносными или неиспользуемыми IP-адресами Интернета.

 
Управление Azure от атак DDoS Protection Standard с помощью портал Azure: https://docs.microsoft.com/azure/virtual-network/manage-ddos-protection

Обнаружение угроз для уровня служб Azure в центре безопасности Azure: https://docs.microsoft.com/azure/security-center/security-center-alerts-service-layer


**Мониторинг Центра безопасности Azure**: Да

**Ответственность**: Customer

### <a name="15-record-network-packets-and-flow-logs"></a>1.5. Запись сетевых пакетов и журналов потоков

**Руководство**. Azure Key Vault не использует группы безопасности сети (NSG) и журналы потоков для Azure Key Vault не фиксируются. Вместо этого используйте частную ссылку Azure для защиты экземпляров Azure Key Vault и включите параметры диагностики для записи метрик и событий аудита.

Интеграция Key Vault с помощью частной ссылки Azure:

https://docs.microsoft.com/azure/key-vault/private-link-service

Ведение журнала Azure Key Vault: https://docs.microsoft.com/azure/key-vault/key-vault-logging



**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6. Развертывание сетевых систем обнаружения и предотвращения вторжений (IDS/IPS)

**Рекомендации**. это требование можно выполнить, настроив Advanced Threat protection (ATP) для Azure Key Vault. ATP предоставляет дополнительный уровень аналитики безопасности. Это средство обнаруживает потенциально опасные попытки доступа или использования учетных записей Azure Key Vault.

Когда центр безопасности Azure обнаруживает аномальные действия, он отображает оповещения. Он также отправляет администратору подписки сведения о подозрительной активности и рекомендации по исследованию и исправлению обнаруженных угроз.

Настройка расширенной защиты от угроз для Azure Key Vault:

https://docs.microsoft.com/azure/security-center/advanced-threat-protection-key-vault



**Мониторинг Центра безопасности Azure**: Да

**Ответственность**: Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7. Управление трафиком к веб-приложениям

**Рекомендации**. Неприменимо. Эта рекомендация предназначена для веб-приложений, работающих в службе приложений Azure или в ресурсах вычислений.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8. Уменьшение сложности и дополнительных затрат на администрирование в правилах безопасности сети

**Руководство**. для ресурсов, которым требуется доступ к экземплярам Azure Key Vault, используйте теги службы Azure для Azure Key Vault, чтобы определить элементы управления доступом к сети для групп безопасности сети или брандмауэра Azure. Теги служб можно использовать вместо определенных IP-адресов при создании правил безопасности. Указав имя тега службы (например, ApiManagement) в соответствующем исходном поле или поле назначения правила, можно разрешить или запретить трафик для соответствующей службы. Корпорация Майкрософт управляет префиксами адресов, входящих в тег службы, и автоматически обновляет этот тег при изменении адресов.

Общие сведения о тегах служб Azure: https://docs.microsoft.com/azure/virtual-network/service-tags-overview


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9. Поддержание стандартных конфигураций безопасности для сетевых устройств

**Руководство**. Определение и реализация стандартных конфигураций безопасности для сетевых ресурсов, связанных с экземплярами Azure Key Vault, с помощью политики Azure. Используйте псевдонимы политик Azure в пространствах имен Microsoft. KeyVault и Microsoft. Network, чтобы создать настраиваемые политики для аудита или принудительного применения конфигурации сети для экземпляров Azure Key Vault. Вы также можете использовать встроенные определения политик, связанные с Azure Key Vault, например:

Служба Key Vault должна использовать конечную точку службы виртуальной сети.

Руководство. Создание политик для обеспечения соответствия и управление ими:

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Примеры политики Azure:

https://docs.microsoft.com/azure/governance/policy/samples/#networ

Краткое руководство. Определение и назначение схемы на портале.

https://docs.microsoft.com/azure/governance/blueprints/create-blueprint-portal


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10. Документация по правилам конфигурации трафика

**Руководство**. Используйте теги для ресурсов, связанных с сетевой безопасностью и потоком трафика для экземпляров Azure Key Vault, чтобы предоставить метаданные и логическую организацию.

Используйте любое из встроенных определений политик Azure, связанных с тегами, например "требовать тег и его значение", чтобы убедиться, что все ресурсы созданы с помощью тегов и уведомлять вас о существующих ресурсах без тегов.

Вы можете использовать Azure PowerShell или Azure CLI для поиска или выполнения действий с ресурсами на основе их тегов.

Используйте теги для Организации ресурсов Azure:

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11. Использование автоматизированных средств для мониторинга конфигураций сетевых ресурсов и обнаружения изменений

**Руководство**. Использование журнала действий Azure для мониторинга конфигураций сетевых ресурсов и обнаружения изменений сетевых ресурсов, связанных с экземплярами Azure Key Vault. Создавайте оповещения в Azure Monitor, которые будут запускаться при изменении критических сетевых ресурсов.

Просмотр и получение событий журнала действий Azure:

https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-view

Создание, просмотр и Управление оповещениями журнала действий с помощью Azure Monitor.

https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

## <a name="logging-and-monitoring"></a>Ведение журналов и мониторинг

*Дополнительные сведения см. в статье [Управление безопасностью: ведение журналов и мониторинг](../../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1. Использование утвержденных источников синхронизации времени

**Руководство**: неприменимо; Корпорация Майкрософт поддерживает источник времени, используемый для ресурсов Azure, например Azure Key Vault, для меток времени в журналах.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2. Настройка централизованного управления журналами безопасности

**Руководство**. прием журналов с помощью Azure Monitor для агрегирования данных безопасности, создаваемых Azure Key Vault. В Azure Monitor используйте рабочую область Azure Log Analytics для запроса и выполнения анализа и используйте учетные записи хранения Azure для долгосрочного и архивного хранения. Кроме того, вы можете включить и подключить данные к Azure Sentinel или сторонним SIEM. 

Ведение журнала Azure Key Vault:

https://docs.microsoft.com/azure/key-vault/key-vault-logging

Краткое руководство. как настроить метку Azure:

https://docs.microsoft.com/azure/sentinel/quickstart-onboard


**Мониторинг Центра безопасности Azure**: Да

**Ответственность**: Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3. Включение журналов аудита для ресурсов Azure

**Рекомендации**. Включите параметры диагностики для экземпляров Azure Key Vault, чтобы получить доступ к журналам аудита, безопасности и диагностики. Автоматически доступны журналы действий, включающие источник событий, дату, пользователя, метку времени, исходные адреса, адреса назначения и другие полезные элементы.

Ведение журнала Azure Key Vault:

https://docs.microsoft.com/azure/key-vault/key-vault-logging


**Мониторинг Центра безопасности Azure**: Да

**Ответственность**: Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4. Сбор журналов безопасности из операционных систем

**Руководство**. Неприменимо; эта рекомендация предназначена для вычислительных ресурсов.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

### <a name="25-configure-security-log-storage-retention"></a>2.5. Настройка хранения журнала безопасности

**Руководство**. в Azure Monitor для рабочей области log Analytics, используемой для хранения журналов Azure Key Vault, задайте срок хранения согласно нормативным требованиям Организации. Используйте учетные записи хранения Azure для долгосрочного и архивного хранения.

Измените срок хранения данных: https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="26-monitor-and-review-logs"></a>2.6. Мониторинг и просмотр журналов

**Руководство**. анализ и мониторинг журналов для аномального поведения и регулярная проверка результатов для ресурсов, защищенных Azure Key Vault. Используйте рабочую область Log Analytics Azure Monitor, чтобы просматривать журналы и выполнять запросы к данным журнала. Кроме того, вы можете включить и подключить данные к Azure Sentinel или сторонним SIEM. 

Краткое руководство. закрывающий метку Azure:

https://docs.microsoft.com/azure/sentinel/quickstart-onboard

Начало работы с Log Analytics в Azure Monitor:

https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-portal

Начало работы с запросами журналов в Azure Monitor:

https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-queries


**Мониторинг Центра безопасности Azure**: Да

**Ответственность**: Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7. Включение оповещений об аномальных действиях

**Руководство**. в центре безопасности Azure включите Advanced Threat protection (ATP) для Key Vault. Включите параметры диагностики в Azure Key Vault и отправьте журналы в рабочую область Log Analytics. Подключите рабочую область Log Analytics к Azure Sentinel, так как она предоставляет решение для оркестрации событий безопасности и автоматического реагирования (SOAR). Это позволяет создавать и использовать сборники схем (автоматизированные решения) для устранения проблем безопасности.

Краткое руководство. закрывающий метку Azure:

https://docs.microsoft.com/azure/sentinel/quickstart-onboard

Управление оповещениями безопасности в центре безопасности Azure и реагирование на них:

https://docs.microsoft.com/azure/security-center/security-center-managing-and-responding-alerts

Реагирование на события с помощью оповещений Azure Monitor:

https://docs.microsoft.com/azure/azure-monitor/learn/tutorial-response


**Мониторинг Центра безопасности Azure**: Да

**Ответственность**: Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8. Централизованное ведение журнала защиты от вредоносных программ

**Руководство**: неприменимо; Azure Key Vault не обрабатывает и не создает журналы, связанные с защитой от вредоносных программ.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

### <a name="29-enable-dns-query-logging"></a>2.9. Включение ведения журнала запросов DNS

**Руководство**: неприменимо; Azure Key Vault не обрабатывает и не создает журналы, связанные с DNS.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

### <a name="210-enable-command-line-audit-logging"></a>2.10. Включение ведения журнала аудита для командной строки

**Руководство**. Неприменимо; эта рекомендация предназначена для вычислительных ресурсов.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

## <a name="identity-and-access-control"></a>Идентификатор и управление доступом

*Дополнительные сведения см. в статье [Управление безопасностью: идентификаторы и управление доступом](../../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1. Инвентаризация учетных записей администраторов

**Руководство**. обеспечение инвентаризации зарегистрированных Azure Active Directory приложений, а также любых учетных записей пользователей, имеющих доступ к ключам Azure Key Vault, секретам и сертификатам. Для запроса и согласования Key Vault доступа можно использовать либо портал Azure, либо PowerShell. Чтобы просмотреть доступ в PowerShell, используйте следующую команду:

(Get-Азресаурце-ResourceId [Кэйваултресаурцеид]). Properties. AccessPolicies

Регистрация приложения с Azure Active Directory:

https://docs.microsoft.com/azure/key-vault/key-vault-manage-with-cli2#registering-an-application-with-azure-active-directory

Безопасный доступ к хранилищу ключей:

https://docs.microsoft.com/azure/key-vault/key-vault-secure-your-key-vault


**Мониторинг Центра безопасности Azure**: Да

**Ответственность**: Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2. Изменение паролей по умолчанию, где применимо

**Руководство**: неприменимо; Azure Key Vault не имеет концепции паролей по умолчанию, так как проверка подлинности обеспечивается Active Directory и защищена с помощью управления доступом на основе ролей Azure (Azure RBAC).


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

### <a name="33-use-dedicated-administrative-accounts"></a>3.3. Применение выделенных административных учетных записей

**Руководство**. Создание стандартных рабочих процедур, связанных с использованием выделенных административных учетных записей, имеющих доступ к экземплярам Azure Key Vault. Используйте центр безопасности Azure управление удостоверениями и доступом (сейчас в предварительной версии) для отслеживания количества активных административных учетных записей.

Мониторинг удостоверений и доступа (Предварительная версия):

https://docs.microsoft.com/azure/security-center/security-center-identity-access


**Мониторинг Центра безопасности Azure**: Да

**Ответственность**: Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4. Использование единого входа с Azure Active Directory

**Руководство**. Используйте субъект-службу Azure совместно с AppID, TenantID и ClientSecret, чтобы легко проверить подлинность приложения и получить маркер, который будет использоваться для доступа к Azure Key Vault секретам.

Проверка подлинности между службами для Azure Key Vault с помощью .NET:

https://docs.microsoft.com/azure/key-vault/service-to-service-authentication



**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5. Использование многофакторной проверки подлинности для любого доступа на основе Azure Active Directory

**Рекомендации**. Включите многофакторную проверку подлинности Azure Active Directory и следуйте рекомендациям центра безопасности Azure по управлению удостоверениями и доступом (в настоящее время в предварительной версии) для защиты ресурсов, включенных в концентратор событий.

Планирование развертывания с использованием многофакторной идентификации Azure AD в облачной среде:

https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

Мониторинг удостоверений и доступа (Предварительная версия):

https://docs.microsoft.com/azure/security-center/security-center-identity-access


**Мониторинг Центра безопасности Azure**: Да

**Ответственность**: Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6. Использование выделенных компьютеров (рабочих станций с привилегированным доступом) для всех административных задач

**Руководство**. Используйте рабочую станцию привилегированного доступа (привилегированным доступом) с помощью многофакторной идентификации Azure AD (MFA), настроенной для входа в систему и настройки ресурсов, включенных Key Vault. 

Рабочие станции привилегированного доступа: https://4sysops.com/archives/understand-the-microsoft-privileged-access-workstation-paw-security-model/ 

Планирование развертывания с использованием многофакторной идентификации Azure AD в облачной среде: https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted



**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7. Ведение журнала и создание оповещений по подозрительным действиям учетных записей администраторов

**Руководство**. Использование Azure Active Directory (Azure AD) Управление привилегированными пользователями (PIM) для создания журналов и оповещений при возникновении подозрительных или ненадежных действий в среде. Используйте обнаружение рисков Azure AD для просмотра предупреждений и отчетов о поведении опасного пользователя. Для дополнительного ведения журнала Отправьте оповещения об обнаружении рисков центра безопасности Azure в Azure Monitor и настройте пользовательские оповещения и уведомления с помощью групп действий.

Включите расширенную защиту от угроз (ATP) для Azure Key Vault, чтобы создать оповещения для подозрительных действий.

Развертывание Azure AD Privileged Identity Management (PIM): https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-deployment-plan

Настройка расширенной защиты от угроз для Azure Key Vault (Предварительная версия): https://docs.microsoft.com/azure/security-center/advanced-threat-protection-key-vault

Оповещения для Azure Key Vault (Предварительная версия): https://docs.microsoft.com/azure/security-center/alerts-reference#alerts-azurekv

Azure Active Directory обнаружения рисков: https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risk-events

Создание групп действий и управление ими в портал Azure: https://docs.microsoft.com/azure/azure-monitor/platform/action-groups



**Мониторинг Центра безопасности Azure**: Да

**Ответственность**: Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8. Управление ресурсами Azure только из утвержденных расположений

**Руководство**. Настройка условия расположения политики условного доступа и Управление именованными расположениями. С помощью именованных расположений можно создавать логические группы диапазонов IP-адресов или стран и регионов. Вы можете ограничить доступ к конфиденциальным ресурсам, таким как Key Vault секреты, в настроенные именованные расположения.

Каково условие расположения в Azure Active Directory условном доступе? https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations



**Мониторинг Центра безопасности Azure**: в настоящее время недоступен.

**Ответственность**: Customer

### <a name="39-use-azure-active-directory"></a>3.9. Использование Azure Active Directory

**Руководство**. Использование Azure Active Directory (Azure AD) в качестве централизованной системы проверки подлинности и авторизации для ресурсов Azure, таких как key Vault. Это обеспечивает управление доступом на основе ролей Azure (Azure RBAC) для администрирования конфиденциальных ресурсов.

 

Краткое руководство. Создание нового клиента в Azure Active Directory:

https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-access-create-new-tenant


**Мониторинг Центра безопасности Azure**: в настоящее время недоступен.

**Ответственность**: Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10. Регулярная проверка и согласование доступа пользователей

**Руководство**. Ознакомьтесь с журналами Azure Active Directory (Azure AD), чтобы упростить обнаружение устаревших учетных записей с помощью Azure Key Vault административных ролей. Кроме того, используйте проверки доступа Azure AD для эффективного управления членством в группах, доступом к корпоративным приложениям, которые могут использоваться для доступа к Azure Key Vault и назначения ролей. Доступ пользователей следует проверять регулярно, например каждые 90 дней, чтобы убедиться, что доступ к ним имеют только нужные пользователи.

Azure Active Directory отчеты и документация по мониторингу:

https://docs.microsoft.com/azure/active-directory/reports-monitoring/

Что такое проверки доступа Azure AD?

https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview


**Мониторинг Центра безопасности Azure**: Да

**Ответственность**: Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11. Отслеживание попыток доступа к отключенным учетным записям

**Руководство**. Включение параметров диагностики для Azure Key Vault и Azure Active Directory, отправка всех журналов в log Analytics рабочую область. Настройте нужные оповещения (например, попытки доступа к отключенным секретам) в Log Analytics.

Интеграция журналов Azure AD с Azure Monitor журналами: https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics

Миграция из старого решения Key Vault: https://docs.microsoft.com/azure/azure-monitor/insights/azure-key-vault#migrating-from-the-old-key-vault-solution



**Мониторинг Центра безопасности Azure**: Да

**Ответственность**: Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12. Предупреждение при подозрительном входе в учетную запись

**Руководство**. Использование функций защиты идентификации и обнаружения рисков Azure Active Directory для настройки автоматических ответов на обнаруженные подозрительные действия, связанные с защищенными ресурсами Azure Key Vault. Вы должны включить автоматические ответы через метку Azure, чтобы реализовать ответы на безопасность вашей организации. 

Отчет о событиях входа с рисками на портале Azure Active Directory: https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risky-sign-ins 

Как настроить и включить политики риска: https://docs.microsoft.com/azure/active-directory/identity-protection/howto-identity-protection-configure-risk-policies

Как подключить метку Azure:  https://docs.microsoft.com/azure/sentinel/quickstart-onboard


**Мониторинг Центра безопасности Azure**: Да

**Ответственность**: Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13. Предоставление корпорации Майкрософт доступа к соответствующим данным клиентов в рамках сценариев поддержки

**Руководство**: неприменимо; Защищенное хранилище не поддерживается для Azure Key Vault.

Поддерживаемые службы и сценарии в общем доступе: https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview#supported-services-and-scenarios-in-general-availability



**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

## <a name="data-protection"></a>Защита данных

*Дополнительные сведения см. в статье [Управление безопасностью: защита данных](../../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1. Инвентаризация конфиденциальных данных

**Руководство**. Использование тегов для отслеживания ресурсов Azure, в которых хранятся или обрабатываются конфиденциальные сведения о ресурсах с поддержкой Azure Key Vault. 

Используйте теги для Организации ресурсов Azure: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags



**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2. Изолирование систем, хранящих или обрабатывающих конфиденциальные данные

**Руководство**. Вы можете защитить доступ к Azure Key Vault, используя конечные точки службы виртуальной сети, настроенные для ограничения доступа к определенным подсетям.

После того как правила брандмауэра вступают в действие, можно выполнять только Azure Key Vault операции с плоскостью данных, если запрос исходит из разрешенных подсетей или диапазонов IP-адресов. Это также относится к Azure Key Vaultному доступу в портал Azure. Хотя вы можете перейти к хранилищу ключей из портал Azure, вы не сможете перечислить ключи, секреты или сертификаты, если ваш клиентский компьютер не находится в списке разрешенных. Это также влияет на средство выбора Azure Key Vault и другие службы Azure. Вы можете просмотреть списки хранилищ ключей, но не список ключей, если правила брандмауэра не позволяют компьютеру клиента сделать это.

Настройка Azure Key Vault брандмауэров и виртуальных сетей: https://docs.microsoft.com/azure/key-vault/key-vault-network-security

Конечные точки службы виртуальной сети для Azure Key Vault: https://docs.microsoft.com/azure/key-vault/key-vault-overview-vnet-service-endpoints



**Мониторинг Центра безопасности Azure**: в настоящее время недоступен.

**Ответственность**: Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. Мониторинг и блокирование несанкционированной передачи конфиденциальной информации

**Руководство**. все данные, хранящиеся в Azure Key Vault, считаются конфиденциальными. Используйте Azure Key Vault элементы управления доступом к плоскости данных, чтобы управлять доступом к Azure Key Vault секретам. Вы также можете использовать встроенный брандмауэр Key Vault для управления доступом на уровне сети. Чтобы отслеживать доступ к Azure Key Vault, включите Key Vault параметры диагностики и отправьте журналы в учетную запись хранения Azure или Log Analytics рабочую область.

Безопасный доступ к хранилищу ключей: https://docs.microsoft.com/azure/key-vault/key-vault-secure-your-key-vault

Настройка Azure Key Vault брандмауэров и виртуальных сетей: https://docs.microsoft.com/azure/key-vault/key-vault-network-security

Ведение журнала Azure Key Vault: https://docs.microsoft.com/azure/key-vault/key-vault-logging



**Мониторинг Центра безопасности Azure**: Да

**Ответственность**: Customer

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4. Шифрование любой конфиденциальной информации при передаче

**Руководство**. весь трафик, который Azure Key Vault для проверки подлинности, управления и доступа к плоскости данных, шифруется и проходит через HTTPS: порт 443. (Тем не менее, для списка отзыва сертификатов иногда будет использоваться трафик HTTP [порт 80]). Azure Key Vault продолжит принимать данные TLS 1,1 и TLS 1,0. Данные могут быть ограничены TLS 1,2 с помощью конфигурации на стороне клиента.



Доступ к Azure Key Vault за брандмауэром: https://docs.microsoft.com/azure/key-vault/key-vault-access-behind-firewall



**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**. Microsoft

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5. Использование средства активного обнаружения для распознавания конфиденциальных данных

**Руководство**: неприменимо; все данные в Azure Key Vault (секреты, ключи и сертификаты) считаются конфиденциальными.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6. Контроль доступа к ресурсам с помощью RBAC

**Руководство**. Защита доступа к плоскости управления и данных экземпляров Azure Key Vault.

Безопасный доступ к хранилищу ключей:

https://docs.microsoft.com/azure/key-vault/key-vault-secure-your-key-vault


**Мониторинг центра безопасности Azure**: неприменимо

**Ответственность**: Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7. Использование защиты от потери данных на основе узла для обеспечения контроля доступа

**Руководство**. Корпорация Майкрософт управляет базовой инфраструктурой для Azure Key Vault и реализует четкие элементы управления для предотвращения потери или раскрытия данных клиента.

Что такое Azure Key Vault?

https://docs.microsoft.com/azure/key-vault/key-vault-overview

Защита данных клиентов Azure:

https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Microsoft

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8. Шифрование конфиденциальной информации при хранении

**Руководство**. все управляемые объекты (ключ, сертификаты и секреты) шифруются при хранении в Azure Key Vault.

Вспомогательная документация:

- [Модель шифрования и таблица управления ключами](../../security/fundamentals/encryption-atrest.md)


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Microsoft

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9. Включение в журнал и создание оповещений по изменениям критических ресурсов Azure

**Руководство**. использование решения Azure Key Vault Analytics в Azure Monitor для просмотра журналов событий аудита Azure Key Vault.

Решение Azure Key Vault Analytics в Azure Monitor:

https://docs.microsoft.com/azure/azure-monitor/insights/azure-key-vault



**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

## <a name="vulnerability-management"></a>Управление уязвимостями

*Дополнительные сведения см. в статье [Управление безопасностью: управление уязвимостями](../../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1. Выполнение автоматизированных средства анализа уязвимостей

**Руководство**. Корпорация Майкрософт выполняет Управление уязвимостью в базовых системах, поддерживающих Azure Key Vault.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Microsoft

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2. Развертывание автоматизированного решения для управления исправлениями операционной системы

**Руководство**: н/д; Корпорация Майкрософт выполняет управление исправлениями в базовых системах, поддерживающих Key Vault.

**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3: Развертывание автоматизированного решения для управления исправлениями ПО сторонних производителей

**Руководство**. Неприменимо; эта рекомендация предназначена для вычислительных ресурсов.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4: Сравнение проверок смежных уязвимостей

**Руководство**. Корпорация Майкрософт выполняет Управление уязвимостью в базовых системах, поддерживающих Key Vault.

**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Microsoft

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5. Использование процесса оценки рисков для определения приоритета в устранении обнаруженных уязвимостей

**Руководство**. Используйте оценки рисков по умолчанию (Оценка безопасности), предоставляемые центром безопасности Azure.

Улучшение оценки безопасности в центре безопасности Azure:

https://docs.microsoft.com/azure/security-center/security-center-secure-score


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

## <a name="inventory-and-asset-management"></a>Инвентаризация и управление ресурсами

*Дополнительные сведения см. в статье [Управление безопасностью: инвентаризация и управление ресурсами](../../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-azure-asset-discovery"></a>6.1. Использование обнаружения ресурсов Azure

**Руководство**. Использование графа ресурсов Azure для запроса и обнаружения всех ресурсов (включая Azure Key Vault экземпляры) в подписке. Убедитесь, что у вас есть соответствующие разрешения (на чтение) в клиенте и вы можете перечислить все подписки Azure, а также ресурсы в ваших подписках.

Краткое руководство. Запуск первого запроса графа ресурсов с помощью Azure Resource Graph Explorer.

https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

Получение подписок, к которым может получить доступ текущая учетная запись.:

https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0

Что такое управление доступом на основе ролей в Azure (RBAC)?

https://docs.microsoft.com/azure/role-based-access-control/overview

**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="62-maintain-asset-metadata"></a>6.2. Ведение метаданных активов

**Руководство**. применение тегов к Azure Key Vault ресурсам, предоставляя метаданные для логической организации их в таксономию.

Создание и использование тегов:

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3. Удаление неавторизованных ресурсов Azure

**Руководство**. Использование тегов, групп управления и отдельных подписок (при необходимости) для упорядочения и отслеживания Azure Key Vault экземпляров и связанных ресурсов. Регулярно сверяйте ресурсы, чтобы своевременно удалять неавторизованные ресурсы из подписки.

Создайте дополнительную подписку Azure:

https://docs.microsoft.com/azure/billing/billing-create-subscription

Создание групп управления для Организации ресурсов и управления ими:

https://docs.microsoft.com/azure/governance/management-groups/create

Используйте теги для Организации ресурсов Azure: https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="64-maintain-an-inventory-of-approved-azure-resources-and-software-titles"></a>6.4. Ведение каталога утвержденных ресурсов Azure и наименований программного обеспечения

**Руководство**. Определение списка утвержденных ресурсов Azure и утвержденного программного обеспечения для ресурсов вычислений

**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5. Отслеживание неутвержденных ресурсов Azure

**Руководство**. Использование политик Azure для ограничения типа ресурсов, которые могут быть созданы в подписках клиентов, с помощью следующих встроенных определений политик:

- Недопустимые типы ресурсов

- Допустимые типы ресурсов

Кроме того, используйте Azure Resource Graph для запроса или обнаружения ресурсов в подписках.

Руководство. Создание политик для обеспечения соответствия и управление ими: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Краткое руководство. Запуск первого запроса графа ресурсов с помощью Azure Resource Graph Explorer. https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal



**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6. Отслеживание неутвержденных программных приложений в рамках ресурсов вычислений

**Руководство**. Неприменимо; эта рекомендация предназначена для вычислительных ресурсов.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7: Удаление неутвержденных ресурсов Azure и программных приложений

**Руководство**: неприменимо; Эта рекомендация предназначена для Azure как к целому, так и к расчетным ресурсам.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

### <a name="68-use-only-approved-applications"></a>6.8: Использование только утвержденных приложений

**Руководство**. Неприменимо; эта рекомендация предназначена для вычислительных ресурсов.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

### <a name="69-use-only-approved-azure-services"></a>6.9: Использование только утвержденных служб Azure

**Руководство**. Использование политик Azure для ограничения типа ресурсов, которые могут быть созданы в подписках клиентов, с помощью следующих встроенных определений политик:

- Недопустимые типы ресурсов

- Допустимые типы ресурсов

Руководство. Создание политик для обеспечения соответствия и управление ими: https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Примеры политики Azure: https://docs.microsoft.com/azure/governance/policy/samples/not-allowed-resource-types



**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="610-implement-approved-application-list"></a>6.10. Реализация списка утвержденных приложений

**Руководство**. Неприменимо; эта рекомендация предназначена для вычислительных ресурсов.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

### <a name="611-limit-users-ability-to-interact-with-azureresources-manager-via-scripts"></a>6,11: ограничьте возможность пользователей взаимодействовать с Азурересаурцес Manager с помощью сценариев

**Руководство**. Использование условного доступа Azure для ограничения возможности пользователей взаимодействовать с Azure Resource Manager (ARM) путем настройки "блокировать доступ" для приложения "Управление Microsoft Azure". Это может препятствовать созданию и изменению ресурсов в среде с высоким уровнем безопасности, например при использовании конфигурации Key Vault.

Управление доступом к управлению Azure с помощью условного доступа:

https://docs.microsoft.com/azure/role-based-access-control/conditional-access-azure-management


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12. Ограничение возможности пользователей выполнять сценарии в ресурсах вычислений

**Руководство**. Неприменимо; эта рекомендация предназначена для вычислительных ресурсов.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13. Физическое или логическое разделение приложений с высоким риском

**Руководство**. Неприменимо; эта рекомендация предназначена для веб-приложений, работающих в службе приложений Azure или в ресурсах вычислений.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

## <a name="secure-configuration"></a>Настройка безопасности

*Дополнительные сведения см. в статье [Управление безопасностью: безопасная конфигурация](../../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1. Установка безопасных конфигураций для всех ресурсов Azure

**Руководство**. Использование псевдонимов политик Azure в пространстве имен Microsoft. KeyVault для создания настраиваемых политик для аудита или принудительного применения конфигурации экземпляров Azure Key Vault. Вы также можете использовать встроенные определения политик Azure для Azure Key Vault, например:

Объекты Key Vault должны быть восстанавливаемыми

Развертывание параметров диагностики Key Vault в рабочей области Log Analytics

Журналы диагностики в Key Vault должны быть включены

Служба Key Vault должна использовать конечную точку службы виртуальной сети.

Развернуть параметры диагностики для Key Vault в концентраторе событий

Используйте рекомендации из центра безопасности Azure в качестве защищенного шаблона базовой конфигурации для экземпляров Azure Key Vault.

Просмотр доступных псевдонимов политик Azure:

https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0

Руководство. Создание политик для обеспечения соответствия и управление ими:

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage


**Мониторинг Центра безопасности Azure**: Да

**Ответственность**: Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2. Сохранение безопасных конфигураций для операционных систем

**Руководство**. Неприменимо; эта рекомендация предназначена для вычислительных ресурсов.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3: Сохранение безопасных конфигураций для ресурсов Azure

**Руководство**. Использование политики Azure [Deny] и [развертывание если не существует] для обеспечения безопасности параметров в ресурсах с поддержкой Azure Key Vault. 

Руководство. Создание политик для обеспечения соответствия и управление ими:

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage 

  
Общие сведения о влиянии политики Azure: 

https://docs.microsoft.com/azure/governance/policy/concepts/effects


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4. Сохранение безопасных конфигураций для операционных систем

**Руководство**. Неприменимо; эта рекомендация предназначена для вычислительных ресурсов.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5: Безопасное хранение конфигурации ресурсов Azure

**Руководство**. Если вы используете пользовательские определения политики Azure для ресурсов с поддержкой Azure Key Vault, используйте Azure Repos для безопасного хранения кода и управления им.

Хранение кода в Azure DevOps:  

https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops 

Документация по Azure Repos: 

https://docs.microsoft.com/azure/devops/repos/index?view=azure-devops

**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6. Безопасное хранение пользовательских образов операционной системы

**Руководство**. Неприменимо; эта рекомендация предназначена для вычислительных ресурсов.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

### <a name="77-deploy-system-configuration-management-tools"></a>7.7: Развертывание инструментов управления конфигурацией системы

**Руководство**. Использование псевдонимов политик Azure в пространстве имен Microsoft. KeyVault для создания настраиваемых политик для оповещения, аудита и принудительного применения конфигураций системы. Кроме того, разрабатывайте процесс и конвейер для управления исключениями политик.

Настройка Политики SQL Azure и управление ею: 

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8. Развертывание средств управления конфигурацией системы для операционных систем

**Руководство**. Неприменимо; эта рекомендация предназначена для вычислительных ресурсов.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9: Реализация автоматизированного мониторинга конфигурации для служб Azure

**Руководство**. Использование центра безопасности Azure для выполнения проверок базовых данных для ресурсов, защищенных Azure Key Vault 

  

Как исправить рекомендации в центре безопасности Azure: 

https://docs.microsoft.com/azure/security-center/security-center-remediate-recommendations

**Мониторинг Центра безопасности Azure**: Да

**Ответственность**: Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10. Реализация автоматизированного мониторинга конфигурации для операционных систем

**Руководство**: неприменимо; Этот тест производительности предназначен для расчетных ресурсов.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

### <a name="711-manage-azure-secrets-securely"></a>7.11: Безопасное управление секретами Azure

**Рекомендации**. Используйте управляемое удостоверение службы в сочетании с Azure Key Vault, чтобы упростить и защитить управление секретами для облачных приложений. Убедитесь, что включено обратимое удаление Azure Key Vault.

Интеграция с управляемыми удостоверениями Azure: 

https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity

Создание Key Vault: 

https://docs.microsoft.com/azure/key-vault/general/quick-create-portal

Проверка подлинности в Key Vault:

https://docs.microsoft.com/azure/key-vault/general/authentication

Назначение политики доступа Key Vault.

https://docs.microsoft.com/azure/key-vault/general/assign-access-policy-portal

**Мониторинг Центра безопасности Azure**: Да

**Ответственность**: Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12. Безопасное и автоматическое управление удостоверениями

**Рекомендации**. Используйте управляемое удостоверение службы в сочетании с Azure Key Vault, чтобы упростить и защитить управление секретами для облачных приложений. 

  

* [Интеграция с управляемыми удостоверениями Azure](../../azure-app-configuration/howto-integrate-azure-managed-service-identity.md)

* [Создание Key Vault](quick-create-portal.md)

* [Проверка подлинности в Key Vault](authentication.md)

* [Назначение политики доступа Key Vault](assign-access-policy-portal.md)

**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13. Устранение непреднамеренного раскрытия учетных данных

**Руководство**. Реализуйте сканер учетных данных для обнаружения учетных данных в коде. Сканер учетных данных также рекомендует перемещать обнаруженные учетные данные в более безопасные расположения, такие как Azure Key Vault.  
  
 Настройка сканера учетных данных: https://secdevtools.azurewebsites.net/helpcredscan.html

**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

## <a name="malware-defense"></a>Защита от вредоносных программ

*Дополнительные сведения см. в статье [Управление безопасностью: защита от вредоносных программ](../../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1. Использование централизованно управляемого программного обеспечения для защиты от вредоносных программ

**Руководство**. Неприменимо; эта рекомендация предназначена для вычислительных ресурсов. Корпорация Майкрософт обрабатывает Антивредоносные программы для базовой платформы.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2. Предварительная проверка файлов для отправки в ресурсы Azure, не являющиеся вычислительными

**Руководство**. на базовом узле, поддерживающем службы Azure (например, Azure Key Vault), включено антивредоносное по Майкрософт, однако оно не выполняется в содержимом клиента.

Предварительное сканирование любого содержимого, отправляемого или отправленного в Нерасчетные ресурсы Azure, например Azure Key Vault. Корпорация Майкрософт не может получить доступ к данным в этих экземплярах.

Сведения о антивредоносном по Майкрософт для облачных служб и виртуальных машин Azure: https://docs.microsoft.com/azure/security/fundamentals/antimalware

**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3. Своевременное обновление программного обеспечения для защиты от вредоносных программ и подписей

**Рекомендации**. Неприменимо. Эта рекомендация предназначена для вычислительных ресурсов. Корпорация Майкрософт обрабатывает Антивредоносные программы для базовой платформы.


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Неприменимо

## <a name="data-recovery"></a>Восстановление данных

*Дополнительные сведения см. в статье [Управление безопасностью: восстановление данных](../../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1. Обеспечение регулярного автоматического резервного копирования

**Рекомендации**: регулярное автоматическое резервное копирование сертификатов Key Vault, ключей, управляемых учетных записей хранения и секретов с помощью следующих команд PowerShell:

- Backup-AzKeyVaultCertificate

- Backup-AzKeyVaultKey

- Backup-AzKeyVaultManagedStorageAccount

- Backup-AzKeyVaultSecret

При необходимости резервные копии Key Vault можно хранить в Azure Backup.

Резервное копирование Key Vault сертификатов: https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultcertificate

Резервное копирование ключей Key Vault: https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey

Резервное копирование Key Vault управляемых учетных записей хранения: https://docs.microsoft.com/powershell/module/az.keyvault/add-azkeyvaultmanagedstorageaccount

Резервное копирование Key Vault секретов: https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultsecret

Как включить Azure Backup: https://docs.microsoft.com/azure/backup



**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2. Выполнение полного резервного копирования системы и любых ключей, управляемых клиентом

**Руководство**. Резервное копирование сертификатов Key Vault, ключей, управляемых учетных записей хранения и секретов с помощью следующих команд PowerShell:

- Backup-AzKeyVaultCertificate

- Backup-AzKeyVaultKey

- Backup-AzKeyVaultManagedStorageAccount

- Backup-AzKeyVaultSecret

При необходимости резервные копии Key Vault можно хранить в Azure Backup.

Резервное копирование Key Vault сертификатов: https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultcertificate

Резервное копирование ключей Key Vault: https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey

Резервное копирование Key Vault управляемых учетных записей хранения: https://docs.microsoft.com/powershell/module/az.keyvault/add-azkeyvaultmanagedstorageaccount

Резервное копирование Key Vault секретов: https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultsecret

Как включить Azure Backup: https://docs.microsoft.com/azure/backup



**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3. Проверка всех резервных копий, включая управляемые клиентом ключи

**Рекомендации**. периодически выполняется восстановление данных сертификатов Key Vault, ключей, управляемых учетных записей хранения и секретов с помощью следующих команд PowerShell:

- Restore-AzKeyVaultCertificate

- Restore-AzKeyVaultKey

- Restore-AzKeyVaultManagedStorageAccount

- Restore-AzKeyVaultSecret

Восстановление сертификатов Key Vault:  https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultcertificate?view=azurermps-6.13.0

Восстановление Key Vault ключей: https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0 

Как восстановить Key Vault управляемые учетные записи хранения: https://docs.microsoft.com/powershell/module/az.keyvault/backup-azkeyvaultmanagedstorageaccount

Восстановление секретов Key Vault: https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultsecret?view=azurermps-6.13.0


**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4. Обеспечение защиты резервных копий и управляемых клиентом ключей

**Руководство**. Убедитесь, что обратимое удаление включено для Azure Key Vault. Обратимое удаление позволяет восстанавливать удаленные хранилища ключей и объекты хранилища, такие как ключи, секреты и сертификаты. 

Использование обратимого удаления Azure Key Vault: 

https://docs.microsoft.com/azure/key-vault/key-vault-soft-delete-powershell

**Мониторинг Центра безопасности Azure**: в настоящее время недоступен.

**Ответственность**: Customer

## <a name="incident-response"></a>реагирование на инциденты.

*Дополнительные сведения см. в статье [Управление безопасностью: реагирование на инциденты](../../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1. Создание руководства по реагированию на инциденты

**Руководство**. Создайте руководство по реагированию на инциденты для вашей организации. Убедитесь в том, что имеются письменные планы реагирования на инциденты, которые определяют все действия персонала, а также этапы обработки инцидентов и управления ими для проверки после инцидента. Эти процессы должны сосредоточиться на защите конфиденциальных систем, например, с помощью Key Vault секретов.

Настройка автоматизации рабочих процессов в Центре безопасности Azure:  

https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide   

Руководство по созданию собственного процесса реагирования на инциденты безопасности:  

https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/

Анатомия инцидента Microsoft Security Response Center:   

https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process   

Клиент может также использовать руководство по обработке инцидентов безопасности компьютера от NIST, чтобы разработать собственный план реагирования на инциденты:  

https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf

**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2. Создание процедуры оценки инцидента и определения приоритетов

**Руководство**. Центр безопасности назначает каждому оповещению серьезность, которая поможет определить, какие предупреждения следует расследовать первыми. Серьезность основывается на том, насколько уверен Центр безопасности в исследовании или аналитике, используемой для оповещения, а также на уровне достоверности злонамеренности события, приведшего к оповещению. Кроме того, четко помечайте подписки тегами (например, Рабочая, не производственная) и создайте систему именования для четкого обнаружения и классификации ресурсов Azure, особенно для обработки конфиденциальных данных, таких как Azure Key Vault секреты.


**Мониторинг Центра безопасности Azure**: Да

**Ответственность**: Customer

### <a name="103-test-security-response-procedures"></a>10.3. Проверка процедур реагирования на угрозы

**Руководство**. проведение упражнений по тестированию возможностей реагирования на инциденты системы с регулярным интервалом для защиты экземпляров Azure Key Vault и связанных ресурсов. Выявите слабые точки и пробелов и пересмотрите план по мере необходимости.

См. публикацию NIST: Руководство по тестированию, обучению и выполнению программ для ИТ-планов и возможностей:  

https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf

**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4. Предоставление контактных сведений и настройка уведомлений по инцидентам безопасности

**Руководство**. Корпорация Майкрософт будет использовать информацию об инциденте безопасности для связи с вами, если центр Microsoft Security Response Center (MSRC) обнаружит, что к вашим данным был получен незаконный или несанкционированный доступ.  Проверьте инциденты после факта обращения, чтобы убедиться в том, что проблемы устранены.

Как задать контакт безопасности центра безопасности Azure:

https://docs.microsoft.com/azure/security-center/security-center-provide-security-contact-details

**Мониторинг Центра безопасности Azure**: Да

**Ответственность**: Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5. Включение оповещений системы безопасности в систему реагирования на инциденты

**Руководство**. Экспорт оповещений и рекомендаций центра безопасности Azure с помощью функции непрерывного экспорта для выявления рисков для ресурсов с поддержкой Azure Key Vault. Непрерывный экспорт позволяет экспортировать предупреждения и рекомендации как вручную, так и в постоянном, непрерывном режиме.  Вы можете использовать соединитель данных Центра безопасности Azure для потоковой передачи оповещений в Azure Sentinel. 

 

Как настроить непрерывный экспорт: 

https://docs.microsoft.com/azure/security-center/continuous-export 

  

Как выполнить потоковую передачу предупреждений в Azure Sentinel: 

https://docs.microsoft.com/azure/sentinel/connect-azure-security-center

**Мониторинг Центра безопасности Azure**: в настоящее время недоступен.

**Ответственность**: Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6. Автоматизация реагирования на оповещения системы безопасности

**Руководство**. Использование функции автоматизации рабочих процессов в центре безопасности Azure для автоматического запуска ответов с помощью "Logic Apps" в оповещениях системы безопасности и рекомендациях по защите ресурсов, защищенных Azure Key Vault. 

 

Как настроить автоматизацию рабочего процесса и Logic Apps: 

https://docs.microsoft.com/azure/security-center/workflow-automation

**Мониторинг Центра безопасности Azure**: в настоящее время недоступен.

**Ответственность**: Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Тесты на проникновение и попытки нарушения безопасности "красной командой"

*Дополнительные сведения см. в статье [Управление безопасностью: тесты на проникновение и попытки нарушения безопасности "красной командой"](../../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings-within-60-days"></a>11.1. Регулярное тестирование на проникновение ресурсов Azure и отслеживание исправлений всех критических точек безопасности в течение 60 дней

**Руководство**. Вы не выполняете прямое тестирование пера непосредственно в службе Azure Key Vault, однако рекомендуется протестировать ресурсы Azure, использующие Key Vault, чтобы обеспечить безопасность секретов.

Чтобы убедиться, что тесты на проникновение не нарушают политики Майкрософт, необходимо соблюдать правила корпорации Майкрософт.

https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1

Дополнительные сведения о стратегии корпорации Майкрософт и реализации "красной команды", а также о тестировании на основе уязвимости для облачной инфраструктуры, служб и приложений Майкрософт см. здесь:  

https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e

**Мониторинг Центра безопасности Azure**: Неприменимо

**Ответственность**: Совмещаемая блокировка

## <a name="next-steps"></a>Дальнейшие действия

- См. [Тесты производительности системы безопасности Azure](../../security/benchmarks/overview.md).
- Узнайте больше о [Базовой конфигурации безопасности Azure](../../security/benchmarks/security-baselines-overview.md).