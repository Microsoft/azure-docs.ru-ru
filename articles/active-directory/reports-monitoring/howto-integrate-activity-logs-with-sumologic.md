---
title: Потоковая передача журналов в SumoLogic с помощью Azure Monitor | Документация Майкрософт
description: Узнайте, как интегрировать журналы Azure Active Directory с SumoLogic с помощью Azure Monitor.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 2c3db9a8-50fa-475a-97d8-f31082af6593
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 04/18/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 51e1f45c787c319c32358e7f310108131647d60e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91335839"
---
# <a name="integrate-azure-active-directory-logs-with-sumologic-using-azure-monitor"></a>Интеграция журналов Azure Active Directory с SumoLogic с помощью Azure Monitor

Из этой статьи вы узнаете, как интегрировать журналы Azure Active Directory (Azure AD) с SumoLogic с помощью Azure Monitor. Сначала вы направите журналы в концентратор событий Azure, а затем интегрируете этот концентратор событий со SumoLogic.

## <a name="prerequisites"></a>Предварительные условия

Для использования этой функции необходимо иметь следующее.
* Концентратор событий Azure, содержащий журналы действий Azure AD. Узнайте, как [настроить потоковую передачу журналов действий в концентратор событий](./tutorial-azure-monitor-stream-logs-to-event-hub.md). 
* Подписка с поддержкой единого входа SumoLogic.

## <a name="steps-to-integrate-azure-ad-logs-with-sumologic"></a>Шаги по интеграции журналов Azure AD с SumoLogic 

1. Сначала [выполните потоковую передачу журналов Azure AD в концентратор событий Azure](./tutorial-azure-monitor-stream-logs-to-event-hub.md).
2. Настройте имеющийся экземпляр SumoLogic, чтобы [собирать журналы для Azure Active Directory](https://help.sumologic.com/Send-Data/Applications-and-Other-Data-Sources/Azure_Active_Directory/Collect_Logs_for_Azure_Active_Directory).
3. [Установите приложение Azure AD SumoLogic](https://help.sumologic.com/Send-Data/Applications-and-Other-Data-Sources/Azure_Active_Directory/Install_the_Azure_Active_Directory_App_and_View_the_Dashboards), чтобы использовать предварительно настроенные панели мониторинга, которые предоставляют анализ среды в реальном времени.

   ![Панель мониторинга](./media/howto-integrate-activity-logs-with-sumologic/overview-dashboard.png)

## <a name="next-steps"></a>Дальнейшие действия

* [Interpret the Azure AD audit logs schema in Azure Monitor (preview)](reference-azure-monitor-audit-log-schema.md) (Интерпретация схемы журналов аудита Azure Active Directory в Azure Monitor (предварительная версия))
* [Interpret the Azure AD sign-in logs schema in Azure Monitor (preview)](reference-azure-monitor-sign-ins-log-schema.md) (Интерпретация схемы журналов входа Azure Active Directory в Azure Monitor (предварительная версия))
* [Часто задаваемые вопросы и известные проблемы](concept-activity-logs-azure-monitor.md#frequently-asked-questions)