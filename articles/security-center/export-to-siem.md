---
title: Потоковая передача оповещений из центра безопасности Azure в системы управления сведениями о безопасности и событиями (SIEM) и другие решения для мониторинга
description: Узнайте, как передавать оповещения системы безопасности в Azure Sentinel, сторонние решения решения Siem, ВЗЛЕТЕЛ или ITSM.
services: security-center
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: how-to
ms.date: 10/06/2020
ms.author: memildin
ms.openlocfilehash: 3ddc385b9d489e0c2ab4abf35a6ade011970342b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100572958"
---
# <a name="stream-alerts-to-a-siem-soar-or-it-service-management-solution"></a>Потоковая передача оповещений в решение SIEM, SOAR или решение для управления ИТ-услугами

Центр безопасности Azure может выполнять потоковую передачу оповещений системы безопасности в наиболее популярные решения по безопасности и управлению событиями (SIEM), автоматизированного реагирования на безопасность (ВЗЛЕТЕЛ) и управления ИТ-службами (ITSM).

Существуют собственные средства Azure, позволяющие просматривать данные оповещений во всех наиболее популярных решениях, используемых сегодня, в том числе:

- **Azure Sentinel**
- **Splunk Enterprise и Splunk Cloud**
- **QRadar IBM**
- **ServiceNow**
- **ArcSight**
- **Power BI**
- **Palo Alto Networks**

## <a name="stream-alerts-to-azure-sentinel"></a>Потоковая передача оповещений в Azure Sentinel 

Центр безопасности изначально интегрируется с Azure Sentinel, облачным решением Azure SIEM и ВЗЛЕТЕЛ. 

Дополнительные [сведения об Azure Sentinel](../sentinel/overview.md).

### <a name="azure-sentinels-connectors-for-security-center"></a>Соединители Azure Sentinel для центра безопасности

Azure Sentinel включает встроенные соединители для центра безопасности Azure на уровнях подписки и клиента:

- [Потоковая передача оповещений на метку Azure на уровне подписки](../sentinel/connect-azure-security-center.md)
- [Подключение всех подписок в клиенте к Azure Sentinel](https://techcommunity.microsoft.com/t5/azure-sentinel/azure-security-center-auto-connect-to-sentinel/ba-p/1387539) 

### <a name="configure-ingestion-of-all-audit-logs-into-azure-sentinel"></a>Настройка приема всех журналов аудита в Azure Sentinel 

Другой альтернативой для изучения оповещений центра безопасности в Azure Sentinel является потоковая передача журналов аудита в Azure Sentinel:
    - [Подключение событий безопасности Windows](../sentinel/connect-windows-security-events.md)
    - [Получение данных из источников на основе Linux с помощью системного журнала](../sentinel/connect-syslog.md)
    - [Подключение данных из журнала действий Azure](../sentinel/connect-azure-activity.md)

> [!TIP]
> Счет за метку Azure оплачивается на основе объема данных, принимаемых для анализа в Azure Sentinel и хранимых в рабочей области Azure Monitor Log Analytics. Azure Sentinel предлагает гибкую и прогнозируемую модель ценообразования. [Дополнительные сведения см. на странице цен на Azure Sentinel](https://azure.microsoft.com/pricing/details/azure-sentinel/).


## <a name="stream-alerts-with-microsoft-graph-security-api"></a>Потоковая передача предупреждений с помощью Microsoft Graph API безопасности

Центр безопасности имеет встроенную интеграцию с API Microsoft Graph безопасности. Настройка не требуется, и нет дополнительных затрат. 

Этот API можно использовать для потоковой передачи предупреждений из **всего клиента** (и данных из многих других продуктов безопасности Майкрософт) в сторонние решения Siem и другие популярные платформы:

- **Splunk Enterprise и Splunk Cloud**  -  [Использование Microsoft Graph API безопасности Add-On для Splunk](https://splunkbase.splunk.com/app/4564/) 
- **Power BI**  -  [Подключение к API Microsoft Graph безопасности в Power BI Desktop](/power-bi/connect-data/desktop-connect-graph-security)
- **ServiceNow**  -  [Следуйте инструкциям по установке и настройке приложения API Microsoft Graph Security из магазина ServiceNow](https://docs.servicenow.com/bundle/orlando-security-management/page/product/secops-integration-sir/secops-integration-ms-graph/task/ms-graph-install.html) .
- **QRadar**  -  [Модуль поддержки устройств IBM для центра безопасности Azure через API Microsoft Graph](https://www.ibm.com/support/knowledgecenter/SS42VS_DSM/com.ibm.dsm.doc/c_dsm_guide_ms_azure_security_center_overview.html) 
- **Palo Alto Networks**, **аномали**, **Lookout**, **Spark** и более [Microsoft Graph API безопасности](https://www.microsoft.com/security/business/graph-security-api#office-MultiFeatureCarousel-09jr2ji)

Дополнительные [сведения об API Microsoft Graph безопасности](https://www.microsoft.com/security/business/graph-security-api).


## <a name="stream-alerts-with-azure-monitor"></a>Потоковая передача предупреждений с Azure Monitor 

Для потоковой передачи предупреждений в **ArcSight**, **Splunk**, **SumoLogic**, Syslog Servers, **логрхисм**, **Logz.IO Cloud** Monitoring Platform и других решениях для мониторинга. подключение к центру безопасности с помощью Azure Monitor через концентраторы событий Azure:

1. Включите [непрерывный экспорт](continuous-export.md) для потоковой передачи оповещений центра безопасности в выделенный концентратор событий Azure на уровне подписки. 
    > [!TIP]
    > Чтобы сделать это на уровне группы управления с помощью политики Azure, см. раздел [Создание конфигураций автоматизации непрерывного экспорта в масштабе](continuous-export.md?tabs=azure-policy#configure-continuous-export-at-scale-using-the-supplied-policies)

1. [Подключите концентратор событий Azure к предпочтительному решению, используя встроенные соединители Azure Monitor](../azure-monitor/essentials/stream-monitoring-data-event-hubs.md#partner-tools-with-azure-monitor-integration).

1. При необходимости можно выполнить потоковую передачу необработанных журналов в концентратор событий Azure и подключиться к предпочтительному решению. Дополнительные сведения см. в статье [мониторинг доступных данных](../azure-monitor/essentials/stream-monitoring-data-event-hubs.md#monitoring-data-available).

> [!TIP]
> Чтобы просмотреть схемы событий экспортируемых типов данных, посетите страницу [схемы событий концентратора событий](https://aka.ms/ASCAutomationSchemas).


## <a name="next-steps"></a>Дальнейшие действия

На этой странице описано, как обеспечить доступность данных оповещений центра безопасности Azure в средстве SIEM, ВЗЛЕТЕЛ или ITSM. Дополнительные материалы:

- [Что собой представляет Azure Sentinel?](../sentinel/overview.md)
- [Проверка предупреждений в центре безопасности Azure](security-center-alert-validation.md) — Проверьте правильность настройки оповещений.
- [Непрерывный экспорт данных Центра безопасности](continuous-export.md)