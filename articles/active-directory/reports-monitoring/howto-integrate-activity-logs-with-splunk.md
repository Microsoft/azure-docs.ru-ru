---
title: Интеграция Splunk с помощью Azure Monitor | Документация Майкрософт
description: Узнайте, как интегрировать журналы Azure Active Directory с Splunk с помощью Azure Monitor.
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
ms.date: 03/10/2021
ms.author: markvi
ms.reviewer: besiler
ms.collection: M365-identity-device-management
ms.openlocfilehash: afb6a597d4fd58646f56e271cb6027fb46db1e26
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102634232"
---
# <a name="how-to-integrate-azure-active-directory-logs-with-splunk-using-azure-monitor"></a>Инструкции. Интеграция журналов Azure Active Directory с Splunk с помощью Azure Monitor

В этой статье вы узнаете, как интегрировать журналы Azure Active Directory (Azure AD) со Splunk с помощью Azure Monitor. Сначала вы направите журналы в концентратор событий Azure, а затем интегрируете этот концентратор событий со Splunk.

## <a name="prerequisites"></a>Предварительные требования

Для использования этой функции необходимо иметь следующее.

- Концентратор событий Azure, содержащий журналы действий Azure AD. Узнайте, как [настроить потоковую передачу журналов действий в концентратор событий](./tutorial-azure-monitor-stream-logs-to-event-hub.md). 

-  [Надстройка Splunk для служб Microsoft Cloud](https://splunkbase.splunk.com/app/3110/#/details). 

## <a name="integrate-azure-active-directory-logs"></a>Интеграция журналов Azure Active Directory 

1. Откройте свой экземпляр Splunk и выберите **Data Summary** (Сводка данных).

    ![Кнопка "Data Summary" (Сводка данных)](./media/howto-integrate-activity-logs-with-splunk/DataSummary.png)

2. Выберите вкладку **Sourcetypes** (Типы источников), а затем — **amal: aadal:audit**.

    ![Вкладка "Sourcetypes" (Типы источников) в диалоговом окне сводки данных](./media/howto-integrate-activity-logs-with-splunk/sourcetypeaadal.png)

    Отобразятся журналы действий Azure AD, как показано на рисунке ниже.

    ![Журналы действий](./media/howto-integrate-activity-logs-with-splunk/activitylogs.png)

> [!NOTE]
> Если вам не удается установить надстройку в экземпляре Splunk (например, если вы используете прокси-сервер или работаете со Splunk Cloud), можно передать эти события в сборщик событий HTTP Splunk с помощью этой [функции Azure](https://github.com/Microsoft/AzureFunctionforSplunkVS), активируемой при поступлении новых сообщений в концентратор событий. 
>

## <a name="next-steps"></a>Следующие шаги

* [Interpret the Azure AD audit logs schema in Azure Monitor (preview)](reference-azure-monitor-audit-log-schema.md) (Интерпретация схемы журналов аудита Azure Active Directory в Azure Monitor (предварительная версия))
* [Interpret the Azure AD sign-in logs schema in Azure Monitor (preview)](reference-azure-monitor-sign-ins-log-schema.md) (Интерпретация схемы журналов входа Azure Active Directory в Azure Monitor (предварительная версия))
* [Часто задаваемые вопросы и известные проблемы](concept-activity-logs-azure-monitor.md#frequently-asked-questions)