---
title: Мониторинг в Azure Active Directory | Документы Майкрософт
description: Общие сведения о мониторинге в Azure Active Directory.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: e2b3d8ce-708a-46e4-b474-123792f35526
ms.service: active-directory
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 04/18/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 763e628183e5f6ad7b7bdbb8ee7ce6db572f44ad
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100577780"
---
# <a name="what-is-azure-active-directory-monitoring"></a>Мониторинг в Azure Active Directory

Теперь функция мониторинга Azure Active Directory (Azure AD) позволяет направлять журналы действий в различные конечные точки. Эти журналы затем можно сохранить для долговременного использования или интегрировать со сторонними средствами для управления сведениями и событиями безопасности (SIEM), чтобы получать аналитические сведения о своей среде.

Сейчас журналы можно направить в следующие конечные точки:

- Учетная запись хранения Azure.
- Концентратор событий Azure для интеграции с экземплярами Splunk и Sumologic.
- Рабочая область Azure Log Analytics, где вы можете анализировать данные, а также создавать информационные панели и оповещения для определенных событий.

**Требуемая роль**: глобальный администратор.

> [!VIDEO https://www.youtube.com/embed/syT-9KNfug8]

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="licensing-and-prerequisites-for-azure-ad-reporting-and-monitoring"></a>Лицензирование и предварительные требования для отчетов и мониторинга Azure AD

Для доступа к журналам входа в Azure AD вам потребуется лицензия Azure AD Premium.

Подробные сведения о компонентах и лицензировании см. на странице с [ценами на Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).

Для развертывания мониторинга и отчетов Azure AD вам потребуется пользователь с правами глобального администратора или администратора безопасности для арендатора Azure AD.

В зависимости от конечного назначения данных журналов вам потребуется одно из следующего:

* Учетная запись хранения Azure, для которой имеется разрешение ListKeys. Рекомендуется использовать общую учетную запись хранения вместо учетной записи хранилища BLOB-объектов. Информацию о ценах на хранилище см. [здесь](https://azure.microsoft.com/pricing/calculator/?service=storage).

* Пространство имен Центров событий Azure для интеграции со сторонними решениями SIEM.

* Рабочая область Azure Log Analytics для отправки данных в журналы Azure Monitor.

## <a name="diagnostic-settings-configuration"></a>Настройка параметров диагностики

Чтобы настроить параметры мониторинга для журналов действий Azure AD, сначала войдите на [портал Azure](https://portal.azure.com), а затем выберите **Azure Active Directory**. Отсюда вы можете получить доступ к странице параметров диагностики двумя способами:

* В разделе **Мониторинг** выберите **Параметры диагностики**.

    ![Параметры диагностики](./media/overview-monitoring/diagnostic-settings.png)
    
* Выберите **Журналы аудита** или **События входа**, а затем — **Экспорт параметров**. 

    ![Параметры экспорта](./media/overview-monitoring/export-settings.png)


## <a name="route-logs-to-storage-account"></a>Передача журналов в учетную запись хранения

Направляя журналы в учетную запись хранения Azure, вы можете хранить их в дольше, чем определено в наших [политиках хранения](reference-reports-data-retention.md) по умолчанию. См. сведения о том, как [направлять данные в учетную запись хранения](quickstart-azure-monitor-route-logs-to-storage-account.md).

## <a name="stream-logs-to-event-hub"></a>Потоковая передача журналов в концентратор событий

Направляя журналы в концентратор событий Azure, вы можете выполнить интеграцию со сторонними средствами, например Sumologic и Splunk. Такая интеграция позволяет объединить данные из журнала действий Azure AD с другими данными, управляемыми средствами SIEM, что позволяет получать более подробные аналитические сведения о среде. См. сведения о том, как [осуществлять потоковую передачу журналов в концентратор событий](tutorial-azure-monitor-stream-logs-to-event-hub.md).

## <a name="send-logs-to-azure-monitor-logs"></a>Отправка данных в журналы Azure Monitor

[Журналы Azure Monitor](../../azure-monitor/logs/log-query-overview.md) — это решение, которое консолидирует данные мониторинга из различных источников, а также предоставляет язык запросов и аналитическую систему для получения сведений о работе приложений и ресурсов. Отправляя журналы действий Azure AD в Azure Monitor Logs вы можете быстро извлекать и отслеживать собранные данные, а также создавать на их основе оповещения. См. дополнительные сведения об [отправке данных в журналы Azure Monitor](howto-integrate-activity-logs-with-log-analytics.md).

Вы также можете установить готовые представления для журналов действий Azure AD, чтобы отслеживать распространенные сценарии, связанные со входом в систему и событиями аудита. См. сведения об [установке и использовании предоставлений анализа журналов для журналов действий Azure AD](howto-install-use-log-analytics-views.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Журналы действий в Azure Monitor](concept-activity-logs-azure-monitor.md)
* [Потоковая передача журналов в концентратор событий](tutorial-azure-monitor-stream-logs-to-event-hub.md)
* [Send logs to Azure Monitor logs](howto-integrate-activity-logs-with-log-analytics.md) (Отправка данных в журналы Azure Monitor)
