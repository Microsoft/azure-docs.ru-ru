---
title: Подключение журналов Dynamics 365 к Azure Sentinel | Документация Майкрософт
description: Узнайте, как использовать соединитель действий Dynamics 365 Common Data Service (CD) для получения сведений о текущих действиях администратора, пользователя и поддержки.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/13/2020
ms.author: yelevin
ms.openlocfilehash: 018368b6284cf39edec01f0a9a943b8ea15c85d0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98104198"
---
# <a name="connect-dynamics-365-activity-logs-to-azure-sentinel"></a>Подключение журналов действий Dynamics 365 к Azure Sentinel

Соединитель действий [Dynamics 365](/office365/servicedescriptions/microsoft-dynamics-365-online-service-description) Common Data Service (CD) предоставляет информацию о действиях администратора, пользователя и поддержки, а также о событиях ведения журнала в социальных сетях Майкрософт. Подключив журналы Dynamics 365 CRM к Azure Sentinel, вы сможете просматривать эти данные в книгах, использовать их для создания пользовательских оповещений и использовать их для улучшения процесса расследования. Этот новый соединитель Sentinel Azure собирает данные из пакета API управления Office для компакт-дисков Dynamics. Дополнительные сведения о действиях компакт-дисков Dynamics, прошедшие аудит в Power Platform, см. в статье [Включение и использование ведения журнала действий](/power-platform/admin/enable-use-comprehensive-auditing).

> [!IMPORTANT]
>
> Соединитель действий Dynamics 365 Common Data Service (CD) в настоящее время доступен в **предварительной версии**. Ознакомьтесь с дополнительными [условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для дополнительных юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не выпущены в общедоступную версию.

## <a name="prerequisites"></a>Предварительные условия

- У вас должны быть разрешения на чтение и запись в рабочей области Sentinel Azure.

- Необходима [Рабочая лицензия Microsoft Dynamics 365](/office365/servicedescriptions/microsoft-dynamics-365-online-service-description). Этот соединитель недоступен для изолированных сред.
    - Для ведения журнала действий требуется подписка Microsoft 365 корпоративный [E3 или](/power-platform/admin/enable-use-comprehensive-auditing#requirements) в журнале.

- Извлечение данных из API управления Office:
    - Вы должны быть глобальным администратором вашего клиента.

    - [Ведение журнала аудита Office](/office365/servicedescriptions/office-365-platform-service-description/office-365-securitycompliance-center) должно быть включено в [центре безопасности и соответствия требованиям Office](/microsoft-365/compliance/search-the-audit-log-in-security-and-compliance).

## <a name="enable-the-dynamics-365-activities-data-connector"></a>Включение соединителя данных действий Dynamics 365

1. В меню навигации меток Azure выберите **соединители данных**.

1. В коллекции **соединителей данных** выберите **Dynamics 365 (Предварительная версия)**, а затем на панели просмотра выберите **открыть страницу соединителя** .

1. На вкладке **инструкции** в разделе **Конфигурация** нажмите кнопку **подключить**. 

    После активации соединителя займет около 30 минут, пока вы не сможете увидеть данные, поступающие в граф. 

    В соответствии с [журналом аудита Office в центре соответствия](/microsoft-365/compliance/search-the-audit-log-in-security-and-compliance#requirements-to-search-the-audit-log)может пройти до 30 минут или до 24 часов после наступления события, чтобы соответствующая запись журнала аудита возвращалась в результатах.

1. Журналы аудита Microsoft Dynamics можно найти в `Dynamics365Activity` таблице. См. Справочник по [схемам](/azure/azure-monitor/reference/tables/dynamics365activity)таблицы.

## <a name="querying-the-data"></a>Запрос данных

1. В меню навигации меток Azure выберите **журналы**.

1. Выполните следующий запрос, чтобы убедиться, что журналы прибывают:

    ```kusto
    Dynamics365Activity
    | take 10
    ```


## <a name="next-steps"></a>Дальнейшие действия
В этом документе вы узнали, как подключить данные действий Dynamics 365 к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.
- Узнайте, как [получить представление о данных и потенциальных угрозах](quickstart-get-visibility.md).
- Приступите к обнаружению угроз с помощью Azure Sentinel, используя [встроенные](tutorial-detect-threats-built-in.md) или [Настраиваемые](tutorial-detect-threats-custom.md) правила.
