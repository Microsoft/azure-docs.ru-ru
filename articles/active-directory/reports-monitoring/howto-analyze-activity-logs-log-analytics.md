---
title: Анализ журналов действий с помощью журналов Azure Monitor | Документация Майкрософт
description: Узнайте, как анализировать журналы действий Azure Active Directory с помощью журналов Azure Monitor
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 4535ae65-8591-41ba-9a7d-b7f00c574426
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
ms.openlocfilehash: 905261058c2de0afae18cbc5572c64962bef8834
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100580016"
---
# <a name="analyze-azure-ad-activity-logs-with-azure-monitor-logs"></a>Анализ журналов действий Azure AD с помощью журналов Azure Monitor

После [интеграции журналов действий Azure AD с журналами Azure Monitor](howto-integrate-activity-logs-with-log-analytics.md) вы можете использовать возможности журналов Azure Monitor, чтобы получить полезные сведения о своей среде. Вы также можете установить [представления анализа журналов для журналов действий Azure AD ](howto-install-use-log-analytics-views.md), чтобы получить доступ к предварительно созданным отчетам о событиях аудита и входа в вашу среду.

В этой статье вы узнаете, как анализировать журналы действий Azure AD в рабочей области Log Analytics. 

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="prerequisites"></a>Предварительные требования 

Для работы вам потребуется следующее:

* Рабочая область Log Analytics в подписке Azure. Узнайте, как [создать рабочую область Log Analytics](../../azure-monitor/logs/quick-create-workspace.md).
* Сначала настройте передачу [журналов действий Azure AD в рабочую область Log Analytics](howto-integrate-activity-logs-with-log-analytics.md).
*  [Доступ](../../azure-monitor/logs/manage-access.md#manage-access-using-workspace-permissions) к рабочей области Log Analytics.
* Следующие роли в Azure Active Directory (при доступе к Log Analytics через портал Azure Active Directory)
    - администратор безопасности;
    - Читатель сведений о безопасности
    - Читатель отчетов
    - глобальный администратор.
    
## <a name="navigate-to-the-log-analytics-workspace"></a>Переход к рабочей области Log Analytics

1. Войдите на [портал Azure](https://portal.azure.com). 

2. Выберите **Azure Active Directory**, а затем выберите **Журналы** из раздела **Мониторинг**, чтобы открыть рабочую область Log Analytics. Рабочая область откроется с запросом по умолчанию.

    ![Запрос по умолчанию](./media/howto-analyze-activity-logs-log-analytics/defaultquery.png)


## <a name="view-the-schema-for-azure-ad-activity-logs"></a>Просмотр схемы для журналов действий Azure AD

Журналы отправляются в таблицы **AuditLogs** и **SigninLogs** в рабочей области. Чтобы просмотреть схему для этих таблиц, выполните следующие действия.

1. В представлении запроса по умолчанию в предыдущем разделе выберите **Схему** и разверните рабочую область. 

2. Разверните раздел **Управление журналами**, а затем разверните **AuditLogs** или **SigninLogs**, чтобы просмотреть схему журнала.
    ![Журналы аудита](./media/howto-analyze-activity-logs-log-analytics/auditlogschema.png) ![Журналы входа](./media/howto-analyze-activity-logs-log-analytics/signinlogschema.png)

## <a name="query-the-azure-ad-activity-logs"></a>Запрос журналов действий Azure AD

Теперь, когда у вас есть журналы в вашей рабочей области, вы можете выполнять запросы к ним. Например, чтобы получить основные приложения, используемые на прошлой неделе, замените запрос по умолчанию следующим и выберите **Запустить**

```
SigninLogs 
| where CreatedDateTime >= ago(7d)
| summarize signInCount = count() by AppDisplayName 
| sort by signInCount desc 
```

Чтобы получить события аудита top за последнюю неделю, используйте следующий запрос.

```
AuditLogs 
| where TimeGenerated >= ago(7d)
| summarize auditCount = count() by OperationName 
| sort by auditCount desc 
```
## <a name="alert-on-azure-ad-activity-log-data"></a>Оповещение о данных журнала действий Azure AD

Можно также настроить оповещения по вашему запросу. Например, чтобы настроить оповещение, если на прошлой неделе было использовано более 10 приложений, выполните следующие действия.

1. В рабочей области выберите **Задание оповещения**, чтобы открыть страницу **Создание правила**.

    ![задание оповещения;](./media/howto-analyze-activity-logs-log-analytics/setalert.png)

2. Выберите **Критерии оповещений** по умолчанию, созданные в оповещении, и обновите **Пороговое значение** в метрике по умолчанию до 10.

    ![Критерии оповещений](./media/howto-analyze-activity-logs-log-analytics/alertcriteria.png)

3. Введите имя и описание оповещения и выберите уровень серьезности. В нашем примере мы можем задать значение **Информационный**.

4. Выберите **группу действий**, которая будет получать оповещения при возникновении сигнала. Вы можете уведомлять свою команду по электронной почте или с помощью текстового сообщения, или вы можете автоматизировать действие с помощью веб-перехватчика, Функций Azure или приложений логики. Дополнительные сведения см. в статье [Создание групп действий и управление ими на портале Azure](../../azure-monitor/alerts/action-groups.md).

5. После настройки оповещения, выберите **Создание оповещения**, чтобы включить его. 

## <a name="use-pre-built-workbooks-for-azure-ad-activity-logs"></a>Использование предварительно созданных книг для журналов действий Azure AD

Книги предоставляют несколько отчетов, относящихся к распространенным сценариям, включающим аудит, вход и подготовку событий. Вы также можете создать оповещение о любых данных, представленных в отчетах, выполнив действия, описанные в предыдущем разделе.

* **Анализ подготовки**: в этой [книге](../app-provisioning/application-provisioning-log-analytics.md) отображаются отчеты, связанные с действиями по подготовке аудита, например число подготовленных пользователей и сбоев подготовки, число пользователей, обновленных и неудачных попыток обновления, а также количество пользователей, которые не были подготовлены и соответствующие сбои.    
* **События входа**. в этой книге представлены наиболее релевантные отчеты, связанные с мониторингом активности входа, например, вход в систему по приложениям, пользователю, устройству, а также представление сводки с отслеживанием количества входов в систему с течением времени.
* Сведения о **условном доступе**. [Книга](../conditional-access/howto-conditional-access-insights-reporting.md) условного доступа и отчеты отчетов позволяют понять влияние политик условного доступа в Организации с течением времени. 

## <a name="next-steps"></a>Дальнейшие действия

* [Начало работы с запросами журналов Azure Monitor](../../azure-monitor/logs/get-started-queries.md)
* [Создание групп действий и управление ими на портале Azure](../../azure-monitor/alerts/action-groups.md)
* [Установка и использование представлений Log Analytics для Azure Active Directory](howto-install-use-log-analytics-views.md)