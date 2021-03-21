---
title: Справочник по схеме источника данных Sentinel Azure
description: В этой статье перечислены схемы источников данных Azure и сторонних поставщиков, поддерживаемые в Azure Sentinel, со ссылками на справочную документацию.
author: batamig
ms.author: bagol
manager: rkarlin
ms.assetid: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: reference
ms.custom: ''
ms.date: 01/14/2021
ms.openlocfilehash: 0cc1a083e1400b5a3167376e374af6c7bf7ab7c7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101698354"
---
# <a name="data-source-schema-reference"></a>Справочник по схеме источника данных

В этой статье перечислены поддерживаемые схемы источников данных Azure и сторонних поставщиков со ссылками на справочную документацию.

## <a name="azure-data-sources"></a>Источники данных Azure

| Type                             | Источник данных             | Log Analytics TableName | Справочник по схеме |
| -------------------------------- | ---------------------- | ---------------------- | ---------------- |
| **Azure**                            | Azure Active Directory | сигниневентс           | [Свойства входа в отчеты о действиях Azure AD](/graph/api/resources/signin#properties) |
| **Azure**                            | Azure Active Directory | AuditLogs              | [Справочник по Azure Monitor AuditLogs](/azure/azure-monitor/reference/tables/auditlogs) |
| **Azure**                            | Azure Active Directory | AzureActivity          | [Справочник по Azure Monitor AzureActivity](/azure/azure-monitor/reference/tables/azureactivity) |
| **Azure**                            | Office                 | OfficeActivity         | Схемы API действий управления Office 365: <br>- [Общая схема ](/office/office-365-management-api/office-365-management-activity-api-schema#common-schema)   <br>- [Схема администрирования Exchange ](/office/office-365-management-api/office-365-management-activity-api-schema#exchange-admin-schema) <br>- [Схема почтовых ящиков Exchange](/office/office-365-management-api/office-365-management-activity-api-schema#exchange-mailbox-schema)  <br>- [Базовая схема SharePoint](/office/office-365-management-api/office-365-management-activity-api-schema#sharepoint-base-schema)   <br>- [Операции с файлами SharePoint](/office/office-365-management-api/office-365-management-activity-api-schema#sharepoint-file-operations) |
| **Azure**                            | Azure Key Vault         | AzureDiagnostics       | [Справочник по Azure Monitor AzureDiagnostics](/azure/azure-monitor/reference/tables/azurediagnostics) |
| **Узел**                             | Linux                  | Системный журнал                 | [Справочник по Azure Monitor syslog](/azure/azure-monitor/reference/tables/syslog) |
| **Network**                          | Журналы IIS               | W3CIISLog              | [Справочник по Azure Monitor W3CIISLog](/azure/azure-monitor/reference/tables/w3ciislog) |
| **Network**                          | вминсигхтс             | VMConnection           | [Справочник по Azure Monitor Вмконнектион](/azure/azure-monitor/reference/tables/vmconnection) |
| **Network**                          | Решение "данные проводки"     | Данные передачи               | [Справочник по Azure Monitor WireData](/azure/azure-monitor/reference/tables/wiredata) |
| **Network**                          | Журналы потоков NSG          | азуренетворканалитикс  | [Схема и агрегирование данных в Аналитика трафика](../network-watcher/traffic-analytics-schema.md) |
| | | | |

> [!NOTE]
> Дополнительные сведения см. во всей [Azure Monitor ссылке на данные](/azure/azure-monitor/reference/).
>
## <a name="3rd-party-vendor-data-sources"></a>источники данных сторонних поставщиков

В следующей таблице перечислены поддерживаемые сторонние поставщики и их syslog-журналы или CEF. документация по сопоставлению для различных поддерживаемых типов журналов, которые содержат сопоставления полей CEF и примеры журналов для каждого типа категории.

| Type |    Vendor |    Продукт | Log Analytics TableName | Справочник по сопоставлению полей CEF  |
| ----- | ----- | ----- | ----- |----- |
| **Network** | Пало-Альто   | ПАНОРАМИРОВАНИЕ ОС    | CommonSecurityLog |   [Pan-OS 9,0. рекомендации по интеграции формата распространенных событий](https://docs.paloaltonetworks.com/content/dam/techdocs/en_US/pdf/cef/pan-os-90-cef-configuration-guide.pdf) (поиск *форматов журнала в стиле CEF*) |
| **Network** | Check Point  |ALL   | CommonSecurityLog | [Описание полей журнала](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk109795)       |
| **Network** | Fortigate   | ALL   | CommonSecurityLog | [Структура схемы журнала](https://docs.fortinet.com/document/fortigate/6.2.3/fortios-log-message-reference/738142/log-schema-structure)         |
| **Network** | Barracuda | Брандмауэр веб-приложения |  CommonSecurityLog   | [Настройка системного журнала и других журналов](https://campus.barracuda.com/product/webapplicationfirewall/doc/4259935/how-to-configure-syslog-and-other-logs/)  |
| **Network** | Cisco | ASA | CommonSecurityLog | [Сообщения syslog серии Cisco ASA](https://www.cisco.com/c/en/us/td/docs/security/asa/syslog/b_syslog/about.html)    |
| **Network** | Cisco | фиреповер   | CommonSecurityLog | [Сообщения системного журнала защиты от угроз Cisco Фиреповер](https://www.cisco.com/c/en/us/td/docs/security/firepower/Syslogs/b_fptd_syslog_guide.pdf)    |
| **Network** | Cisco   | Символ  | Таблица пользовательских журналов  | [Форматы журналов и управление версиями](https://docs.umbrella.com/deployment-umbrella/docs/log-formats-and-versioning)   |
| **Network**   | Cisco | Meraki    | CommonSecurityLog |   [Типы событий syslog и примеры журналов](https://documentation.meraki.com/zGeneral_Administration/Monitoring_and_Reporting/Syslog_Event_Types_and_Log_Samples)    |
| **Network**   | Zscaler | Служба потоковой передачи Nano (NSS)|   CommonSecurityLog | [Форматирование каналов NSS](https://help.zscaler.com/zia/documentation-knowledgebase/analytics/nss/nss-feeds/formatting-nss-feeds) (только для веб-, брандмауэра, DNS и туннелей) |
| **Network**   |F5 | BigIP LTM|    CommonSecurityLog|  [Сообщения о событиях и типы атак](https://techdocs.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/bigip-external-monitoring-implementations-13-0-0/15.html)  |
| **Network** | F5  | BigIP ASM|    CommonSecurityLog|  [Ведение журнала событий безопасности приложения](https://techdocs.f5.com/kb/en-us/products/big-ip_asm/manuals/product/asm-implementations-13-1-0/14.html)                                                           |
| **Network** | Citrix  |Брандмауэр веб-приложения   | CommonSecurityLog|    [Поддержка ведения журнала общего формата событий (CEF) в брандмауэре приложения](https://support.citrix.com/article/CTX136146) <br>  [Справочник по сообщениям syslog NetScaler 12,0](https://developer-docs.citrix.com/projects/netscaler-syslog-message-reference/en/12.0/)   |
|**Узел** |Symantec | Менеджер Endpoint Protection Symantec (СЕПМ) | CommonSecurityLog|[Параметры внешнего ведения журнала и уровни серьезности событий журнала для менеджер Endpoint Protection](https://support.symantec.com/us/en/article.tech171741.html)|
|**Узел** |Trend Micro |Все |CommonSecurityLog | [Сопоставление содержимого syslog-CEF](https://docs.trendmicro.com/en-us/enterprise/control-manager-70/appendices/syslog-mapping-cef.aspx) |
| | | | | |

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о поддерживаемых соединителях Sentinel Azure, таких как CEF, syslog, Direct, Agent и Custom Connectors:

- [Подключение источников данных](connect-data-sources.md)

- [Azure Sentinel syslog, CEF и другие соединители сторонних поставщиков](https://techcommunity.microsoft.com/t5/azure-sentinel/azure-sentinel-syslog-cef-and-other-3rd-party-connectors-grand/ba-p/803891)