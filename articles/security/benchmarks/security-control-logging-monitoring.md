---
title: Управление безопасностью Azure — ведение журнала и мониторинг
description: Ведение журнала и мониторинг управления безопасностью в Azure
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 04/14/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: ccfbb585ccf50366721925b0b31b17088fd36371
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102612428"
---
# <a name="security-control-logging-and-monitoring"></a>Управление безопасностью: ведение журнала и мониторинг

Журналы безопасности и мониторинг посвящены действиям, связанным с включением, получением и хранением журналов аудита для служб Azure.

## <a name="21-use-approved-time-synchronization-sources"></a>2.1. Использование утвержденных источников синхронизации времени

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 2.1 | 6.1 | пиринг Майкрософт. |

Корпорация Майкрософт поддерживает источники времени для ресурсов Azure. Однако вы можете управлять параметрами синхронизации времени для ресурсов вычислений.

- [Настройка синхронизации времени для ресурсов Windows Azure для вычислений](../../virtual-machines/windows/time-sync.md)

- [Настройка синхронизации времени для ресурсов Azure Linux](../../virtual-machines/linux/time-sync.md)

## <a name="22-configure-central-security-log-management"></a>2.2. Настройка централизованного управления журналами безопасности

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 2.2 | 6,5, 6,6 | Customer |

Прием журналов с помощью Azure Monitor для объединения данных безопасности, создаваемых устройствами конечных точек, сетевыми ресурсами и другими системами безопасности. В Azure Monitor используйте рабочие области Log Analytics для запроса и выполнения анализа, а учетные записи хранения Azure — для долгосрочного и архивного хранения.

Кроме того, вы можете включить и подключить данные к Azure Sentinel или сторонним SIEM. 

- [Подключение к Azure Sentinel](../../sentinel/quickstart-onboard.md)

- [Как получить журналы и метрики платформы с помощью Azure Monitor](../../azure-monitor/essentials/diagnostic-settings.md)

- [Как получить журналы внутреннего узла виртуальной машины Azure с помощью Azure Monitor](../../azure-monitor/vm/quick-collect-azurevm.md)

- [Начало работы с Azure Monitor и интеграция SIEM стороннего производителя](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

## <a name="23-enable-audit-logging-for-azure-resources"></a>2.3. Включение журналов аудита для ресурсов Azure

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 2.3 | 6,2, 6,3 | Customer |

Включите параметры диагностики для ресурсов Azure, чтобы получить доступ к журналам аудита, безопасности и диагностики. Автоматически доступны журналы действий, включающие источник событий, дату, пользователя, метку времени, исходные адреса, адреса назначения и другие полезные элементы.

- [Как получить журналы и метрики платформы с помощью Azure Monitor](../../azure-monitor/essentials/diagnostic-settings.md)

- [Общие сведения о ведении журналов и различных типах журналов в Azure](../../azure-monitor/essentials/platform-logs-overview.md)

## <a name="24-collect-security-logs-from-operating-systems"></a>2.4. Сбор журналов безопасности из операционных систем

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 2.4 | 6,2, 6,3 | Customer |

Если ресурсы для вычислений принадлежат корпорации Майкрософт, корпорация Майкрософт несет ответственность за мониторинг ИТ-ресурсов. Если ресурс-источник принадлежит вашей организации, вы отвечаете за его мониторинг. Для мониторинга операционной системы можно использовать центр безопасности Azure. Данные, собираемые центром безопасности из операционной системы, включают в себя тип ОС и версию, ОПЕРАЦИОНную систему (журналы событий Windows), запущенные процессы, имя компьютера, IP-адреса и вошедший в систему пользователь. Агент Log Analytics также собирает файлы аварийного дампа.

- [Как получить журналы внутреннего узла виртуальной машины Azure с помощью Azure Monitor](../../azure-monitor/vm/quick-collect-azurevm.md)

- [Общие сведения о сборе данных в центре безопасности Azure](../../security-center/security-center-enable-data-collection.md)

## <a name="25-configure-security-log-storage-retention"></a>2.5. Настройка хранения журнала безопасности

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 2.5 | 6.4 | Customer |

В Azure Monitor задайте срок хранения Log Analytics рабочей области согласно нормативным требованиям Организации. Используйте учетные записи хранения Azure для долгосрочного и архивного хранения.

- [Изменение срока хранения данных в Log Analytics](../../azure-monitor/logs/manage-cost-storage.md#change-the-data-retention-period)

- [Настройка политики хранения для журналов учетных записей хранения Azure](../../storage/common/manage-storage-analytics-logs.md#configure-logging)

## <a name="26-monitor-and-review-logs"></a>2.6. Мониторинг и просмотр журналов

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 2.6 | 6.7 | Customer |

Анализируйте и отслеживайте журналы на предмет аномального поведения и регулярно проверяйте результаты. Используйте рабочую область Log Analytics Azure Monitor, чтобы просматривать журналы и выполнять запросы к данным журнала.

Кроме того, вы можете включить и подключить данные к Azure Sentinel или сторонним SIEM. 

- [Подключение к Azure Sentinel](../../sentinel/quickstart-onboard.md)

- [Общие сведения о рабочей области Log Analytics](../../azure-monitor/logs/log-analytics-tutorial.md)

- [Выполнение пользовательских запросов в Azure Monitor](../../azure-monitor/logs/get-started-queries.md)

## <a name="27-enable-alerts-for-anomalous-activities"></a>2,7: Включение оповещений для аномальных действий

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 2.7 | 6,8 | Customer |

Используйте центр безопасности Azure с рабочей областью Log Analytics для мониторинга и оповещения о аномальных действиях, обнаруженных в журналах и событиях безопасности.

Кроме того, вы можете включить и подключить данные в Azure Sentinel.

- [Подключение к Azure Sentinel](../../sentinel/quickstart-onboard.md)

- [Управление оповещениями в центре безопасности Azure](../../security-center/security-center-managing-and-responding-alerts.md)

- [Как оповещать данные журнала Log Analytics](../../azure-monitor/alerts/tutorial-response.md)

## <a name="28-centralize-anti-malware-logging"></a>2.8. Централизованное ведение журнала защиты от вредоносных программ

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 2.8 | 8.6 | Customer |

Включите сбор событий защиты от вредоносных программ для виртуальных машин Azure и облачных служб.

- [Настройка антивредоносного по Майкрософт для виртуальных машин](/powershell/module/servicemanagement/azure.service/set-azurevmmicrosoftantimalwareextension)

- [Настройка антивредоносного по Майкрософт для облачных служб](/powershell/module/servicemanagement/azure.service/set-azureserviceantimalwareextension)

- [Знакомство с антивредоносным по Майкрософт](../fundamentals/antimalware.md)

## <a name="29-enable-dns-query-logging"></a>2.9. Включение ведения журнала запросов DNS

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 2,9 | 8,7 | Customer |

Реализуйте сторонние решения из Azure Marketplace для решения регистрации DNS в соответствии с вашими организациями.  

## <a name="210-enable-command-line-audit-logging"></a>2.10. Включение ведения журнала аудита для командной строки

| Идентификатор Azure | Идентификаторы CIS | Обязательство |
|--|--|--|
| 2.10 | 8,8 | Customer |

Используйте Microsoft Monitoring Agent на всех поддерживаемых виртуальных машинах Windows Azure для регистрации события создания процесса и поля командной строки.   Для поддерживаемых виртуальных машин Linux в Azure можно вручную настроить ведение журнала консоли для каждого узла и использовать syslog для хранения данных.  Кроме того, используйте рабочую область Log Analytics Azure Monitor, чтобы просматривать журналы и выполнять запросы к данным журнала из виртуальных машин Azure. 

- [Сбор данных в центре безопасности Azure](../../security-center/security-center-enable-data-collection.md#data-collection-tier)

- [Выполнение пользовательских запросов в Azure Monitor](../../azure-monitor/logs/get-started-queries.md)

- [Источники данных системного журнала в Azure Monitor](../../azure-monitor/agents/data-sources-syslog.md)


## <a name="next-steps"></a>Дальнейшие действия

- См. следующий контроль безопасности: [Идентификация и управление доступом](security-control-identity-access-control.md) .