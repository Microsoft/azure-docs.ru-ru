---
title: Производительность системы безопасности Azure v2 — ведение журналов и обнаружение угроз
description: Ведение журнала и обнаружение угроз для системы безопасности Azure версии 2
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 02/22/2021
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: a274ec1aab3f530700f89ef810fa667fdf4d08e6
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102042986"
---
# <a name="security-control-v2-logging-and-threat-detection"></a>Управление безопасностью v2: ведение журнала и обнаружение угроз

Ведение журналов и обнаружение угроз охватывает элементы управления для обнаружения угроз в Azure и включения, сбора и хранения журналов аудита для служб Azure. Сюда входят процессы обнаружения, исследования и исправления с помощью элементов управления для создания высококачественных оповещений с помощью встроенного обнаружения угроз в службах Azure. Кроме того, он включает сбор журналов с Azure Monitor, централизованный анализ безопасности с помощью маркеров Azure, синхронизации времени и срока хранения журнала.

Чтобы просмотреть подходящую встроенную политику Azure, ознакомьтесь с [подробными сведениями о встроенной инициативе по обеспечению безопасности в Azure: ведение журнала и обнаружение угроз](../../governance/policy/samples/azure-security-benchmark.md#logging-and-threat-detection)

## <a name="lt-1-enable-threat-detection-for-azure-resources"></a>LT-1: Включение обнаружения угроз для ресурсов Azure

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| LT-1 | 6.7 | AU-3, AU-6, AU-12, SI-4 |

Убедитесь, что вы отслеживаете различные типы ресурсов Azure для потенциальных угроз и аномалий. Сосредоточьтесь на получении высококачественных оповещений, чтобы сократить число ложных срабатываний аналитиков для сортировки. Источником предупреждений могут быть данные журнала, агенты или другие данные.

Используйте встроенную функцию обнаружения угроз в центре безопасности Azure, которая основана на мониторинге телеметрии службы Azure и анализе журналов служб. Сбор данных выполняется с помощью агента Log Analytics, который считывает различные конфигурации, связанные с безопасностью, и журналы событий из системы и копирует их в рабочую область для анализа. 

Кроме того, используйте метку Azure для создания правил аналитики, которые позволяют найти угрозы, соответствующие определенному критерию в среде. Правила создают инциденты при совпадении критериев, чтобы можно было исследовать каждый инцидент. Azure Sentinel также может импортировать аналитику угроз сторонних производителей, чтобы улучшить возможности обнаружения угроз. 

- [Защита от угроз с помощью Центра безопасности Azure](../../security-center/azure-defender.md)

- [Справочное руководство по оповещениям системы безопасности в центре безопасности Azure](../../security-center/alerts-reference.md)

- [Создание настраиваемых правил аналитики для обнаружения угроз](../../sentinel/tutorial-detect-threats-custom.md)

- [Кибератакная аналитика угроз с помощью Sentinel Azure](/azure/architecture/example-scenario/data/sentinel-threat-intelligence)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Безопасность инфраструктуры и конечных точек](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

- [Операции безопасности](/azure/cloud-adoption-framework/organize/cloud-security-operations-center)

- [Управление состоянием](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)

- [Безопасность приложений и DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops) 

- [Аналитика угроз](/azure/cloud-adoption-framework/organize/cloud-security-threat-intelligence)

## <a name="lt-2-enable-threat-detection-for-azure-identity-and-access-management"></a>LT-2: включение обнаружения угроз для управления удостоверениями и доступом Azure

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| LT-2 | 6,8 | AU-3, AU-6, AU-12, SI-4 |

Azure AD предоставляет следующие журналы пользователей, которые можно просмотреть в службе отчетов Azure AD или интегрировать с Azure Monitor, Sentinel или другими средствами SIEM и мониторинга для более сложных вариантов использования в целях мониторинга и анализа. 
-   Входы. Отчет о входах содержит информацию об использовании управляемых приложений и входах пользователей.

-   Журналы аудита — возможность отслеживания с помощью журналов всех изменений, внесенных при использовании разных функций в Azure AD. Примеры журналов аудита включают изменения, внесенные в такие ресурсы в Azure AD, как добавление или удаление пользователей, приложений, групп, ролей и политик.

-   Входы, представляющие риск. Вход, представляющий риск, означает, что в систему пытался войти пользователь, который не является законным владельцем учетной записи.

-   Пользователи, находящиеся в группе риска. Такая пометка означает, что конфиденциальность учетной записи пользователя, возможно, нарушена.

Центр безопасности Azure также может оповещать о некоторых подозрительных действиях, например о чрезмерном числе неудачных попыток проверки подлинности, а также об устаревших учетных записях в подписке. Помимо базового уровня безопасности санации, модуль защиты от угроз в центре безопасности Azure также может получать более подробные оповещения системы безопасности от отдельных ресурсов вычислений Azure (например, виртуальных машин, контейнеров, службы приложений), ресурсов данных (таких как база данных SQL и хранилище) и уровней обслуживания Azure. Эта возможность позволяет просматривать аномалии учетной записи внутри отдельных ресурсов.

- [Отчеты о действиях аудита в Azure AD](../../active-directory/reports-monitoring/concept-audit-logs.md)

- [Включение Защиты идентификации Azure](../../active-directory/identity-protection/overview-identity-protection.md)

- [Защита от угроз с помощью Центра безопасности Azure](../../security-center/azure-defender.md)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Безопасность инфраструктуры и конечных точек](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

- [Операции безопасности](/azure/cloud-adoption-framework/organize/cloud-security-operations-center)

- [Управление состоянием](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)

- [Безопасность приложений и DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Аналитика угроз](/azure/cloud-adoption-framework/organize/cloud-security-threat-intelligence)

## <a name="lt-3-enable-logging-for-azure-network-activities"></a>LT-3: включение ведения журнала для сетевых операций Azure

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| LT-3 | 9,3, 12,2, 12,5, 12,8 | AU-3, AU-6, AU-12, SI-4 |

Включение и получение журналов ресурсов группы безопасности сети (NSG), журналов потоков NSG, журналов брандмауэра Azure и брандмауэра веб-приложения (WAF) для анализа безопасности с целью поддержки исследований инцидентов, обнаружения угроз и создания предупреждений системы безопасности. Журналы потоков можно отправить в рабочую область Azure Monitor Log Analytics, а затем использовать Аналитика трафика для получения ценных сведений.

Убедитесь, что вы собираете журналы запросов DNS для помощи в корреляции других сетевых данных.

- [Включение журналов потоков для групп безопасности сети](../../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Журналы и метрики Брандмауэра Azure](../../firewall/logs-and-metrics.md)

- [Как включить и использовать Аналитика трафика](../../network-watcher/traffic-analytics.md)

- [Мониторинг с помощью наблюдателя за сетями](../../network-watcher/network-watcher-monitoring-overview.md)

- [Решения для мониторинга сетей Azure в Azure Monitor](../../azure-monitor/insights/azure-networking-analytics.md)

- [Сбор подробных сведений о инфраструктуре DNS с помощью решения Аналитика DNS](../../azure-monitor/insights/dns-analytics.md)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Безопасность инфраструктуры и конечных точек](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

- [Операции безопасности](/azure/cloud-adoption-framework/organize/cloud-security-operations-center)

- [Управление состоянием](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)

- [Безопасность приложений и DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops) 

- [Аналитика угроз](/azure/cloud-adoption-framework/organize/cloud-security-threat-intelligence)

## <a name="lt-4-enable-logging-for-azure-resources"></a>LT-4: включение ведения журнала для ресурсов Azure

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| LT-4 | 6,2, 6,3, 8,8 | AU-3, AU-12 |

Включите ведение журнала для ресурсов Azure в соответствии с требованиями к соответствию, обнаружению угроз, поиску и исследованию инцидентов. 

Вы можете использовать центр безопасности Azure и политику Azure, чтобы включить журналы ресурсов и сбор данных журналов в ресурсах Azure для доступа к журналам аудита, безопасности и ресурсов. Автоматически доступны журналы действий, включающие источник событий, дату, пользователя, метку времени, исходные адреса, адреса назначения и другие полезные элементы.

- [Общие сведения о ведении журналов и различных типах журналов в Azure](../../azure-monitor/essentials/platform-logs-overview.md)

- [Общие сведения о сборе данных в центре безопасности Azure](../../security-center/security-center-enable-data-collection.md)

**Ответственность**: Совмещаемая блокировка

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Операции безопасности](/azure/cloud-adoption-framework/organize/cloud-security-operations-center)

Безопасность инфраструктуры и конечных точек 

- [Безопасность приложений и DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Аналитика угроз](/azure/cloud-adoption-framework/organize/cloud-security-threat-intelligence)

## <a name="lt-5-centralize-security-log-management-and-analysis"></a>LT-5: централизованные управление журналом безопасности и анализ

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| LT-5 | 6,5, 6,6 | AU-3, SI-4 |

Централизованное хранение журналов и анализ для включения корреляции. Для каждого источника журнала убедитесь, что вы назначили владельца данных, руководство по доступу, место хранения, какие средства используются для обработки данных и доступа к ним, а также требования к хранению данных.

Убедитесь, что вы интегрируете журналы действий Azure в центральное ведение журнала. Прием журналов с помощью Azure Monitor для объединения данных безопасности, создаваемых устройствами конечных точек, сетевыми ресурсами и другими системами безопасности. В Azure Monitor используйте рабочие области Log Analytics для запроса и выполнения анализа и используйте учетные записи хранения Azure для долгосрочного и архивного хранения.

Кроме того, включите и подключите данные к Azure Sentinel или сторонним SIEM.

Многие организации предпочитают использовать метку Azure для "горячих" данных, которые часто используются, а служба хранилища Azure — для "холодного" данных, которые используются реже.

- [Как получить журналы и метрики платформы с помощью Azure Monitor](../../azure-monitor/essentials/diagnostic-settings.md)

- [Подключение к Azure Sentinel](../../sentinel/quickstart-onboard.md)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Архитектура безопасности](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

- [Безопасность приложений и DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops) 

- [Безопасность инфраструктуры и конечных точек](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

## <a name="lt-6-configure-log-storage-retention"></a>LT-6: Настройка хранения журналов

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| LT – 6 | 6.4 | AU-3, AU-11 |

Настройте срок хранения журнала в соответствии с требованиями, нормативным требованиям и бизнес-потребностями. 

В Azure Monitor можно задать срок хранения Log Analytics рабочей области в соответствии с нормативными требованиями Организации. Используйте службы хранилища Azure, Data Lake или Log Analytics учетные записи рабочих областей для долгосрочного и архивного хранения.

- [Изменение срока хранения данных в Log Analytics](../../azure-monitor/logs/manage-cost-storage.md#change-the-data-retention-period)

- [Настройка политики хранения для журналов учетных записей хранения Azure](../../storage/common/manage-storage-analytics-logs.md#configure-logging)

- [Экспорт оповещений и рекомендаций центра безопасности Azure](../../security-center/continuous-export.md)

**Ответственность**: Customer

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Архитектура безопасности](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

- [Безопасность приложений и DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Операции безопасности](/azure/cloud-adoption-framework/organize/cloud-security-operations-center)

- [Управление соответствием требованиям безопасности](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

## <a name="lt-7-use-approved-time-synchronization-sources"></a>LT-7: использование источников синхронизации утвержденного времени

| Идентификатор Azure | ИДЕНТИФИКАТОРы элементов управления CIS v 7.1 | Директива NIST SP 800-53 R4 ID (s) |
|--|--|--|--|
| LT – 7 | 6.1 | AU-8 |

Корпорация Майкрософт поддерживает источники времени для большинства служб Azure PaaS и SaaS. Для виртуальных машин используйте Microsoft по умолчанию NTP-сервер для синхронизации по времени, если нет конкретных требований. Если необходимо установить собственный сервер NTP, убедитесь, что вы защищаете порт службы UDP 123.

Все журналы, созданные ресурсами в Azure, содержат отметки времени с часовым поясом, указанным по умолчанию.

- [Настройка синхронизации времени для ресурсов Windows Azure для вычислений](../../virtual-machines/windows/time-sync.md)

- [Настройка синхронизации времени для ресурсов Azure Linux](../../virtual-machines/linux/time-sync.md)

- [Как отключить входящий трафик UDP для служб Azure](https://support.microsoft.com/help/4558520/how-to-disable-inbound-udp-for-azure-services)

**Ответственность**: Совмещаемая блокировка

**Заинтересованные лица по безопасности клиентов** (дополнительные [сведения](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)):

- [Политика и стандарты](/azure/cloud-adoption-framework/organize/cloud-security-policy-standards)

- [Безопасность приложений и DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops) 

- [Безопасность инфраструктуры и конечных точек](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)