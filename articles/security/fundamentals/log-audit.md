---
title: Ведение журнала и аудит безопасности в Azure | Документация Майкрософт
description: Сведения о журналах, доступных в Azure, и о безопасности, которые вы можете получить.
services: security
documentationcenter: na
author: TerryLanfear
manager: rkarlin
editor: TomSh
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/31/2019
ms.author: terrylan
ms.openlocfilehash: 0d85cf6ae501a7d50f20e48543e361149f4b57d0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100580548"
---
# <a name="azure-security-logging-and-auditing"></a>Ведение журнала и аудит безопасности Azure

Azure предоставляет множество настраиваемых параметров ведения журналов и аудита безопасности, которые помогают выявить недочеты в политиках и механизмах безопасности. Эта статья описывает создание, сбор и анализ журналов безопасности из служб, размещенных в Azure.

> [!Note]
> Выполнение некоторых рекомендаций в этой статье может привести к более интенсивному использованию данных, а также сетевых и вычислительных ресурсов, а значит, и к дополнительным затратам на лицензии или подписки.

## <a name="types-of-logs-in-azure"></a>Типы журналов в Azure

Облачные приложения являются сложными и содержат множество подвижных частей. Данные ведения журнала могут предоставить подробные сведения о приложениях и помочь вам:

- Устранение прошлых проблем или предотвращение возможных неполадок
- Повышение производительности приложения или удобство обслуживания
- Автоматизируйте действия, которые в противном случае требуют вмешательства вручную

Журналы Azure делятся по следующим категориям:
* **Журналы управления и контроля** содержат информацию об операциях CREATE, UPDATE и DELETE в Azure Resource Manager. Дополнительные сведения см. в разделе [Журналы действий Azure](../../azure-monitor/essentials/platform-logs-overview.md).

* **Журналы плоскости данных** предоставляют информацию о событиях, возникших при использовании ресурсов Azure. К этому типу относятся, например, журналы системы событий Windows, журналы безопасности и журналы приложений на виртуальной машине, а также [журналы диагностики](../../azure-monitor/essentials/platform-logs-overview.md), настроенные с помощью Azure Monitor.

* Журналы **обработанных событий** предоставляют сведения о проанализированных событиях и оповещениях, которые были обработаны от вашего имени. К примерам таких событий относятся [оповещения центра безопасности Azure](../../security-center/security-center-managing-and-responding-alerts.md). Обработав и проанализировав вашу подписку, [центр безопасности Azure](../../security-center/security-center-introduction.md) предоставляет лаконичные оповещения безопасности.

В следующей таблице перечислены наиболее важные типы журналов, доступных в Azure:

| Категория журнала | Тип журнала | Использование | Интеграция |
| ------------ | -------- | ------ | ----------- |
|[Журналы действий](../../azure-monitor/essentials/platform-logs-overview.md)|События уровня управления с ресурсами Azure Resource Manager.|    Предоставляет информацию об операциях, которые выполнялись с ресурсами в подписке.|    REST API, [Azure Monitor](../../azure-monitor/essentials/platform-logs-overview.md).|
|[Журналы ресурсов Azure](../../azure-monitor/essentials/platform-logs-overview.md)|Своевременные данные об операциях с ресурсами Azure Resource Manager в подписке.| Информация об операциях, выполняемых самим ресурсом.| Azure Monitor|
|[Отчеты Azure Active Directory](../../active-directory/reports-monitoring/overview-reports.md)|Журналы и отчеты. | Предоставляет сведения о действиях входа пользователей и системных действиях по управлению пользователями и группами.|[API Graph](../../active-directory/develop/microsoft-graph-intro.md)|
|[Виртуальные машины и облачные службы](../../azure-monitor/vm/quick-collect-azurevm.md)|Служба журнала событий Windows и системный журнал Linux| Собирают системные данные и данные журналов, поступающие от виртуальных машин, и передает их в указанную вами учетную запись хранения.|   Windows (с использованием хранилища Диагностики Azure для Windows [[WAD](../../azure-monitor/agents/diagnostics-extension-overview.md)]) и Linux в Azure Monitor|
|[Аналитика Службы хранилища Azure](/rest/api/storageservices/fileservices/storage-analytics)|Обеспечивает ведение журнала хранилища, предоставляет данные метрик для учетной записи хранения.|Предоставляет сведения о трассировке запросов, анализирует тенденции использования и диагностирует проблемы учетной записи хранения.| REST API или [клиентская библиотека](/dotnet/api/overview/azure/storage).|
|[Журналы потоков группы безопасности сети (NSG)](../../network-watcher/network-watcher-nsg-flow-logging-overview.md)|Использует формат JSON, показывает входящие и исходящие потоки на основе правил.|Позволяет просмотреть сведения о входящем и исходящем IP-трафике в группе безопасности сети.|[Наблюдатель за сетями Azure](../../network-watcher/network-watcher-monitoring-overview.md)|
|[Application Insights](../../azure-monitor/app/app-insights-overview.md)|Журналы, исключения и пользовательские системы диагностики.|  Служба наблюдения за производительностью приложений (APM) для веб-разработчиков на нескольких платформах.| REST API, [Power BI](https://powerbi.microsoft.com/documentation/powerbi-azure-and-power-bi/).|
|[Обработка данных/оповещения системы безопасности](../../security-center/security-center-introduction.md)|   Оповещения центра безопасности Azure, оповещения журналов Azure Monitor|    Предоставляет сведения о безопасности и оповещения.|  Интерфейсы REST API, JSON.|

## <a name="log-integration-with-on-premises-siem-systems"></a>Интеграция журналов с локальными системами SIEM
[Интеграция оповещений центра безопасности](../../security-center/security-center-partner-integration.md) рассказывает о том, как синхронизировать оповещения центра безопасности, события безопасности виртуальной машины, собранные журналами диагностики Azure, и журналы аудита Azure с помощью журналов Azure Monitor или решения SIEM.

## <a name="next-steps"></a>Дальнейшие действия

- [Аудит и ведение журнала](management-monitoring-overview.md). Защита данных и обеспечение видимости и быстрого реагирования на своевременные оповещения системы безопасности.

- [Ведение журнала безопасности и сбор журналов аудита в Azure](https://azure.microsoft.com/resources/videos/security-logging-and-audit-log-collection/). Описание параметров, которые нужно применить, чтобы экземпляры Azure собирали нужные журналы безопасности и аудита.

- [Настройка параметров аудита для семейства веб-сайтов](https://support.office.com/article/Configure-audit-settings-for-a-site-collection-A9920C97-38C0-44F2-8BCB-4CF1E2AE22D2?ui=&rs=&ad=US). Являясь администратором семейства веб-сайтов, вы можете получить журнал действий конкретных пользователей, а также журнал действий за определенный диапазон дат.

- [Поиск в журнале аудита в центре безопасности Microsoft 365](/microsoft-365/compliance/search-the-audit-log-in-security-and-compliance): используйте центр безопасности Microsoft 365 для поиска в едином журнале аудита и просмотра действий пользователя и администратора в Организации.