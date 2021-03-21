---
title: Подключение источников данных к Azure Sentinel | Документация Майкрософт
description: Узнайте, как подключать такие источники данных, как Microsoft 365 Defender (ранее — Microsoft Threat protection), Microsoft 365 и Office 365, Azure AD, ATP и Cloud App Security в Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/01/2020
ms.author: yelevin
ms.openlocfilehash: 3df78d6b53f8e8739307e9b870aa03d76bfd6771
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101718613"
---
# <a name="connect-data-sources"></a>Подключение источников данных

Когда вы включили метку Azure, первым делом нужно подключить источники данных. В состав Azure Sentinel входит ряд соединителей для решений Майкрософт, которые доступны в режиме реального времени и обеспечивают интеграцию с Microsoft 365 Defender (ранее — решения Майкрософт для защиты от угроз), Microsoft 365 источники (включая Office 365), Azure AD, защитник Майкрософт для идентификации (ранее — Azure ATP), Microsoft Cloud App Security и многое другое. Кроме того, доступны встроенные соединители для более широкой экосистемы безопасности для решений сторонних разработчиков. Чтобы подключить источники данных к Azure Sentinel, вы можете использовать Common Event Format (CEF), Syslog или REST API.

1. В меню выберите **Data connectors** (Соединители данных). На этой странице приведен полный список соединителей Azure Sentinel и показано состояние каждого из них. Выберите нужный соединитель и щелкните **Open connector page** (Открыть страницу соединителя). 

   ![Коллекция соединителей данных](./media/collect-data/collect-data-page.png)

1. На странице соединителя проверьте, выполнены ли все предварительные требования, и следуйте инструкциям, чтобы подключить данные к Azure Sentinel. Синхронизация журналов с Azure Sentinel может начаться не сразу. После подключения вы увидите сводку данных в графе **Data received** (Полученные данные) и состояние подключения для типов данных.

   ![Настройка соединителей данных](./media/collect-data/opened-connector-page.png)
  
1. Перейдите на вкладку **Next steps** (Дальнейшие действия), чтобы посмотреть список готового содержимого, предоставляемого Azure Sentinel для определенного типа данных.

   ![Дальнейшие действия для соединителей](./media/collect-data/data-insights.png)
 

## <a name="data-connection-methods"></a>Методы подключения к данным

Azure Sentinel поддерживает следующие методы подключения к данным:

- **Интеграция между службами**.<br> Некоторые службы подключены в собственном коде, например службы AWS и Майкрософт. Они используют платформу Azure для готовой интеграции. Следующие решения можно подключить несколькими щелчками мыши:
    - [Amazon Web Services — CloudTrail](connect-aws.md);
    - [Azure Active Directory](connect-azure-active-directory.md) — журналы аудита и журналы входа
    - [Действия Azure](connect-azure-activity.md)
    - [Защита идентификации Azure AD](connect-azure-ad-Identity-protection.md)
    - [Защита от атак DDoS Azure](connect-azure-ddos-protection.md)
    - [Защитник Azure для Интернета вещей](connect-asc-iot.md) (прежнее название — центр безопасности Azure для IOT)
    - [Azure Information Protection](connect-azure-information-protection.md)
    - [Брандмауэр Azure](connect-azure-firewall.md)
    - [Центр безопасности Azure](connect-azure-security-center.md) — оповещения от решений защитника Azure
    - [Брандмауэр веб-приложения Azure (WAF)](connect-azure-waf.md) (прежнее название — Microsoft WAF)
    - [Cloud App Security](connect-cloud-app-security.md)
    - [Сервер доменных имен](connect-dns.md)
    - [Защитник Microsoft 365](connect-microsoft-365-defender.md) — включает инциденты M365D и необработанные данные MDE.
    - [Защитник Майкрософт для конечной точки](connect-microsoft-defender-advanced-threat-protection.md) (прежнее название — Advanced Threat Protection в защитнике Майкрософт)
    - [Защитник Майкрософт для идентификации](connect-azure-atp.md) (ранее — Azure Advanced Threat protection)
    - [Защитник Microsoft для office 365](connect-office-365-advanced-threat-protection.md) (прежнее название — Office 365 Advanced Threat protection)
    - [Office 365](connect-office-365.md) (теперь с командами!)
    - [Брандмауэр Windows](connect-windows-firewall.md)
    - [События безопасности Windows](connect-windows-security-events.md)

- **Внешние решения через API**. Некоторые источники данных подключены с помощью интерфейсов API, предоставляемых подключенным источником данных. Как правило, большинство технологий безопасности предоставляют набор API, с помощью которых можно получить журналы событий. API подключаются к Azure Sentinel и собирают определенные типы данных, а затем отправляют их в Azure Log Analytics. Устройства, подключенные с помощью API:
    
    - [Agari Phishing Defense and Brand Protection](connect-agari-phishing-defense.md)
    - [Alcide kAudit](connect-alcide-kaudit.md)
    - [Barracuda WAF](connect-barracuda.md)
    - [Barracuda CloudGen Firewall](connect-barracuda-cloudgen-firewall.md)
    - [BETTER Mobile Threat Defense](connect-better-mtd.md)
    - [Beyond Security beSECURE](connect-besecure.md)
    - [Cisco Umbrella](connect-cisco-umbrella.md)
    - [Citrix Analytics (модуль безопасности)](connect-citrix-analytics.md)
    - [F5 BIG-IP](connect-f5-big-ip.md)
    - [Forcepoint DLP](connect-forcepoint-dlp.md)
    - [Google Workspace (ранее G Suite)](connect-google-workspace.md)
    - [Журналы DNS Нкслог (Windows)](connect-nxlog-dns.md)
    - [NXLog LinuxAudit](connect-nxlog-linuxaudit.md)
    - [Okta SSO](connect-okta-single-sign-on.md)
    - [Orca Security](connect-orca-security-alerts.md)
    - [Журналы Perimeter 81](connect-perimeter-81-logs.md)
    - [Безопасность электронной почты proofpoint On Demand по запросу (POD)](connect-proofpoint-pod.md)
    - [Proofpoint TAP](connect-proofpoint-tap.md)
    - [Qualys VM](connect-qualys-vm.md)
    - [Salesforce Service Cloud](connect-salesforce-service-cloud.md)
    - [Sophos Cloud Optix](connect-sophos-cloud-optix.md)
    - [Squadra Technologies secRMM](connect-squadra-secrmm.md)
    - [Symantec ICDX](connect-symantec.md)
    - [VMware Carbon Black Cloud Endpoint Standard](connect-vmware-carbon-black.md)
    - [Zimperium](connect-zimperium-mtd.md)


- **Внешние решения через агент**. Sentinel можно подключить с помощью агента к любому другому источнику данных, который может выполнять потоковую передачу журналов в реальном времени с помощью протокола syslog.

    Большинство устройств использует протокол Syslog для отправки сообщений о событиях, которые включают сам журнал и данные о нем. Формат журналов меняется, но большинство устройств поддерживают форматирование данных журнала на основе CEF. 

    Агент Sentinel Azure, который на самом деле является агентом Log Analytics, преобразует журналы в формате CEF в формат, который может быть получен Log Analytics. В зависимости от типа устройства агент устанавливается либо непосредственно на устройстве, либо на выделенном сервере пересылки журналов на основе Linux. Агент для Linux получает события из управляющей программы Syslog по протоколу UDP. Если компьютер Linux должен собирать большое количество событий Syslog, отправка производится по протоколу TCP из управляющей программы Syslog в агент, а затем — в Log Analytics.

    - **Брандмауэры, прокси-серверы и конечные точки — CEF:**
        - [Vectra AI Detect](connect-ai-vectra-detect.md)
        - [Akamai Security Events](connect-akamai-security-events.md)
        - [Aruba ClearPass](connect-aruba-clearpass.md)
        - [Broadcom Symantec DLP](connect-broadcom-symantec-dlp.md)
        - [Check Point](connect-checkpoint.md)
        - [Cisco ASA](connect-cisco.md)
        - [Citrix WAF](connect-citrix-waf.md)
        - [CyberArk Enterprise Password Vault](connect-cyberark.md)
        - [ExtraHop Reveal(x)](connect-extrahop.md)
        - [F5 ASM](connect-f5.md)
        - [Продукты Forcepoint](connect-forcepoint-casb-ngfw.md)
        - [Fortinet](connect-fortinet.md)
        - [Illusive Networks AMS](connect-illusive-attack-management-system.md)
        - [Imperva WAF Gateway](connect-imperva-waf-gateway.md)
        - [One Identity Safeguard](connect-one-identity.md)
        - [Palo Alto Networks](connect-paloalto.md)
        - [Thycotic Secret Server](connect-thycotic-secret-server.md)
        - [Trend Micro Deep Security](connect-trend-micro.md)
        - [Trend Micro TippingPoint](connect-trend-micro-tippingpoint.md)
        - [Платформа сетевого судебного Вирекса](connect-wirex-systems.md)
        - [Zscaler](connect-zscaler.md)
        - [Другие устройства на основе CEF](connect-common-event-format.md)
    - **Брандмауэры, прокси-серверы и конечные точки — syslog:**
        - [Alsid for Active Directory](connect-alsid-active-directory.md)
        - [Cisco Meraki](connect-cisco-meraki.md)
        - [Унифицированная вычислительная система Cisco (UCS)](connect-cisco-ucs.md)
        - [Infoblox NIOS](connect-infoblox.md)
        - [Juniper SRX](connect-juniper-srx.md)
        - [Pulse Connect Secure](connect-pulse-connect-secure.md)
        - [Sophos XG](connect-sophos-xg-firewall.md)
        - [Squid Proxy](connect-squid-proxy.md)
        - [Symantec Proxy SG](connect-symantec-proxy-sg.md)
        - [Symantec VIP](connect-symantec-vip.md)
        - [VMware ESXi](connect-vmware-esxi.md)
        - [Другие устройства на основе системного журнала](connect-syslog.md)
    - [HTTP-сервер Apache](connect-apache-http-server.md)
    - Решения DLP
    - [Поставщики Threat Intelligence](connect-threat-intelligence.md)
    - [Компьютеры DNS](connect-dns.md) — агент установлен непосредственно на компьютере DNS
    - [Виртуальные машины Azure Stack](connect-azure-stack.md)
    - Серверы Linux
    - Другие облака
    
## <a name="agent-connection-options"></a>Параметры подключения агента<a name="agent-options"></a>

Чтобы подключить внешнее устройство к Azure Sentinel, агент должен быть развернут на выделенном компьютере (виртуальной машине или локальном компьютере) для поддержки связи между устройством и Azure Sentinel. Вы можете развернуть агент автоматически или вручную. Автоматическое развертывание доступно только в том случае, если выделенный компьютер является новой созданной в Azure виртуальной машиной. 

![CEF в Azure](./media/connect-cef/cef-syslog-azure.png)

Вместо этого вы можете развернуть агент вручную на существующей виртуальной машине Azure, на виртуальной машине в другом облаке или на локальном компьютере.

![CEF в локальной среде](./media/connect-cef/cef-syslog-onprem.png)

## <a name="map-data-types-with-azure-sentinel-connection-options"></a>Преобразование типов данных с помощью параметров подключения к Sentinel Azure


| **Data type** | **Инструкции по подключению** | **Соединитель данных?** | **Комментарии** |
|------|---------|-------------|------|
| AWSCloudTrail | [Подключение AWS](connect-aws.md) | &#10003; | |
| AzureActivity | [Подключение действий Azure](connect-azure-activity.md) и [обзора журналов действий](../azure-monitor/essentials/platform-logs-overview.md)| &#10003; | |
| AuditLogs | [Подключение Azure AD](connect-azure-active-directory.md)  | &#10003; | |
| SigninLogs | [Подключение Azure AD](connect-azure-active-directory.md)  | &#10003; | |
| AzureFirewall |[Система диагностики Azure](../firewall/firewall-diagnostics.md) | &#10003; | |
| InformationProtectionLogs_CL  | [Отчеты Azure Information Protection](/azure/information-protection/reports-aip)<br>[Подключение Azure Information Protection](connect-azure-information-protection.md)  | &#10003; | Обычно в этом случае в дополнение к типу данных используется функция **InformationProtectionEvents**. Дополнительные сведения см. статью [Как изменять отчеты и создавать пользовательские запросы](/azure/information-protection/reports-aip#how-to-modify-the-reports-and-create-custom-queries)|
| AzureNetworkAnalytics_CL  | [Схема аналитики трафика](../network-watcher/traffic-analytics.md) [Аналитика трафика](../network-watcher/traffic-analytics.md)  | | |
| CommonSecurityLog  | [Подключение CEF](connect-common-event-format.md)  | &#10003; | |
| OfficeActivity | [Подключение Office 365](connect-office-365.md) | &#10003; | |
| SecurityEvents | [Подключение событий безопасности Windows](connect-windows-security-events.md)  | &#10003; | Книги небезопасных протоколов см. в разделе [Настройка книг небезопасных протоколов](./quickstart-get-visibility.md#use-built-in-workbooks)  |
| Системный журнал | [Подключение системного журнала](connect-syslog.md) | &#10003; | |
| Брандмауэр веб-приложений Майкрософт (WAF) – (AzureDiagnostics) |[Подключение брандмауэра веб-приложений Майкрософт](./connect-azure-waf.md) | &#10003; | |
| SymantecICDx_CL | [Подключение Symantec](connect-symantec.md) | &#10003; | |
| ThreatIntelligenceIndicator  | [Подключение аналитики угроз](connect-threat-intelligence.md)  | &#10003; | |
| VMConnection <br> ServiceMapComputer_CL<br> ServiceMapProcess_CL|  [Схема службы Azure Monitor](../azure-monitor/vm/service-map.md)<br>[Подключение аналитических сведений для виртуальных машин Azure Monitor](../azure-monitor/vm/vminsights-enable-overview.md) <br> [Включение аналитических сведений для виртуальных машин Azure Monitor](../azure-monitor/vm/vminsights-enable-overview.md) <br> [Подключение одиночной виртуальной машины](../azure-monitor/vm/vminsights-enable-portal.md)<br>  [Использование подключения через политику](../azure-monitor/vm/vminsights-enable-policy.md)| &#10007; | Книга аналитики виртуальных машин  |
| DnsEvents | [Подключение DNS](connect-dns.md) | &#10003; | |
| W3CIISLog | [Подключение журналов IIS](../azure-monitor/agents/data-sources-iis-logs.md)  | &#10007; | |
| Данные передачи | [Подключение передаваемых данных](../azure-monitor/insights/wire-data.md) | &#10007; | |
| WindowsFirewall | [Подключение брандмауэра Windows](connect-windows-firewall.md) | &#10003; | |
| SecurityAlert службы "Защита идентификации Azure AD"  | [Подключения Защиты идентификации Azure AD](connect-azure-ad-identity-protection.md)  | &#10003; | |
| AATP SecurityAlert  | [Подключение защитника Майкрософт для идентификации](connect-azure-atp.md) (ранее — Azure ATP) | &#10003; | |
| SecurityAlert сертификатов службы приложений  | [Подключение оповещений защитника Azure](connect-azure-security-center.md) из центра безопасности Azure  | &#10003; | |
| MCAS SecurityAlert  | [Подключение Microsoft Cloud App Security](connect-cloud-app-security.md)  | &#10003; | |
| SecurityAlert | | | |
| Sysmon (событие) | [Подключение Sysmon](https://azure.microsoft.com/blog/detecting-in-memory-attacks-with-sysmon-and-azure-security-center)<br> [Подключение событий Windows](../azure-monitor/agents/data-sources-windows-events.md) <br> [Получение средства синтаксического анализа Sysmon](https://github.com/Azure/Azure-Sentinel/blob/master/Parsers/Sysmon/Sysmon-v10.42-Parser.txt)| &#10007; | По умолчанию на виртуальных машинах не установлена коллекция Sysmon. Дополнительные сведения об установке агента Sysmon см. в разделе [Sysmon](/sysinternals/downloads/sysmon). |
| ConfigurationData  | [Автоматизация учета виртуальной машины](../automation/change-tracking/overview.md)| &#10007; | |
| ConfigurationChange  | [Автоматизация отслеживания виртуальной машины](../automation/change-tracking/overview.md) | &#10007; | |
| F5 BIG-IP | [Подключение F5 BIG-IP](https://devcentral.f5.com/s/articles/Integrating-the-F5-BIGIP-with-Azure-Sentinel)  | &#10007; | |
| McasShadowItReporting  |  | &#10007; | |
| Barracuda_CL | [Подключение Barracuda](connect-barracuda.md) | &#10003; | |


## <a name="next-steps"></a>Дальнейшие действия

- Чтобы начать работу с Azure Sentinel, вам нужна подписка на Microsoft Azure. Если у вас нет подписки, вы можете зарегистрироваться для получения [бесплатной пробной версии](https://azure.microsoft.com/free/).
- Узнайте, как [подключить данные к Azure Sentinel](quickstart-onboard.md), и [отслеживайте свои данные и потенциальные угрозы](quickstart-get-visibility.md).