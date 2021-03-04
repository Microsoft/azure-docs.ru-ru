---
title: Настройка защитника Azure для SQL
description: Узнайте, как включить дополнительную схему "защитник Azure" в центре безопасности Azure для плана SQL.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: ba46c460-6ba7-48b2-a6a7-ec802dd4eec2
ms.service: security-center
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/11/2021
ms.author: memildin
ms.openlocfilehash: b82f0ca0624fcbd64f1c23f87f8f21f96d8e4d4c
ms.sourcegitcommit: 4b7a53cca4197db8166874831b9f93f716e38e30
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102100582"
---
# <a name="enable-azure-defender-for-sql-servers-on-machines"></a>Включение защитника Azure для серверов SQL Server на компьютерах 

Эта схема защитника Azure обнаруживает аномальные действия, указывающие на необычные и потенциально опасные попытки доступа к базам данных или их использования.

Вы получаете оповещения о подозрительных действиях с базами данных, потенциальных уязвимостях, атаках путем внедрения кода SQL и аномальных закономерностях в доступе к базам данных и шаблонах запросов.

## <a name="availability"></a>Доступность

|Аспект|Сведения|
|----|:----|
|Состояние выпуска:|Общедоступная версия|
|Цены:|Плата **за использование Azure Defender для серверов SQL Server на компьютерах** взимается, как показано на странице [цен на центр безопасности](https://azure.microsoft.com/pricing/details/security-center/) .|
|Защищаемые версии SQL|SQL Server Azure (все версии, охваченные службой поддержки Майкрософт)|
|Облако.|![Да](./media/icons/yes-icon.png) Коммерческие облака<br>![Да](./media/icons/yes-icon.png) US Gov<br>![Нет](./media/icons/no-icon.png) China Gov и другие правительственные облака|
|||

## <a name="set-up-azure-defender-for-sql-servers-on-machines"></a>Настройка защитника Azure для серверов SQL Server на компьютерах

Чтобы включить этот план, сделайте следующее:

[Шаг 1. Подготавливаете агент Log Analytics на узле SQL Server:](#step-1-provision-the-log-analytics-agent-on-your-sql-servers-host)

[Шаг 2. Включите необязательный план на странице цен и настроек центра безопасности:](#step-2-enable-the-optional-plan-in-security-centers-pricing-and-settings-page)


### <a name="step-1-provision-the-log-analytics-agent-on-your-sql-servers-host"></a>Шаг 1. Подготавливаете агент Log Analytics на узле SQL Server:

- **SQL Server на виртуальной машине Azure** . Если компьютер SQL размещается на виртуальной машине Azure, можно [включить автоматическую подготовку log Analytics агента <a name="auto-provision-mma"></a>](security-center-enable-data-collection.md#auto-provision-mma). Кроме того, вы можете выполнить процедуру вручную, чтобы подключить [Azure Stack виртуальные машины](quickstart-onboard-machines.md#onboard-your-azure-stack-vms).
- **SQL Server в службе** "Дуга Azure". если ваша SQL Server управляется серверами с поддержкой [Arc](../azure-arc/index.yml) , вы можете развернуть агент log Analytics с помощью рекомендации центра безопасности "log Analytics должен быть установлен на компьютерах Arc Azure на базе Windows (Предварительная версия)". Кроме того, можно следовать методам установки, описанным в [документации по службе Arc Azure](../azure-arc/servers/manage-vm-extensions.md).

- **SQL Server локального** компьютера. если ваша SQL Server размещена на локальном компьютере Windows без дуги Azure, у вас есть два варианта подключения его к Azure:
    
    - **Развертывание Arc Azure** . Вы можете подключить любой компьютер Windows к центру безопасности. Тем не менее Azure Arc обеспечивает более глубокую интеграцию во *всей* среде Azure. Если вы настроили дугу Azure, вы увидите страницу **SQL Server — Azure Arc** на портале, и ваши оповещения системы безопасности будут отображаться на выделенной вкладке **безопасности** на этой странице. Поэтому первым и рекомендуемым вариантом является [Настройка дуги Azure на узле](../azure-arc/servers/onboard-portal.md#install-and-validate-the-agent-on-windows) и следуйте инструкциям по **SQL Server в Azure**, приведенном выше.
        
    - **Подключение компьютера Windows без дуги Azure** . Если вы решили подключить SQL Server, работающую на компьютере Windows, без использования дуги Azure, следуйте инструкциям в статье [Подключение компьютеров Windows к Azure Monitor](../azure-monitor/agents/agent-windows.md).


### <a name="step-2-enable-the-optional-plan-in-security-centers-pricing-and-settings-page"></a>Шаг 2. Включите необязательный план на странице цен и настроек центра безопасности:

1. В меню центра безопасности откройте страницу **параметры ценообразования &** .

    - Если вы используете **рабочую область по умолчанию центра безопасности Azure** (с именем "defaultWorkspace-[ваш идентификатор подписки]-[регион]"), выберите соответствующую **подписку**.

    - Если вы используете **рабочую область не по умолчанию**, выберите соответствующую **рабочую область** (введите имя рабочей области в фильтре, если это необходимо):

        ![Поиск рабочей области, отличной от используемой по умолчанию, по названию](./media/security-center-advanced-iaas-data/pricing-and-settings-workspaces.png)

1. Установите параметр для **защитника Azure для серверов SQL Server на компьютерах** с планом **на**. 

    :::image type="content" source="./media/security-center-advanced-iaas-data/sql-servers-on-vms-in-pricing-small.png" alt-text="Страница цен на центр безопасности с необязательными планами":::

    Этот план будет включен на всех серверах SQL Server, подключенных к выбранной рабочей области. Защита будет полностью активна после первого перезапуска экземпляра SQL Server.

    >[!TIP] 
    > Чтобы создать новую рабочую область, следуйте инструкциям в разделе [создание log Analytics рабочей области](../azure-monitor/logs/quick-create-workspace.md).


1. При необходимости настройте уведомления по электронной почте для оповещений системы безопасности. 
    Вы можете задать список получателей уведомлений по электронной почте при создании оповещений центра безопасности. Сообщение электронной почты содержит прямую ссылку на оповещение в центре безопасности Azure со всеми релевантными сведениями. Дополнительные сведения см. в разделе [Настройка уведомлений по электронной почте для оповещений системы безопасности](security-center-provide-security-contact-details.md).


## <a name="azure-defender-for-sql-alerts"></a>Оповещения Azure Defender для SQL
Предупреждения создаются необычными и потенциально опасными попытками получить доступ к компьютерам SQL или использовать их. Эти события могут активировать предупреждения, показанные на [странице справки по оповещениям](alerts-reference.md#alerts-sql-db-and-warehouse).

## <a name="explore-and-investigate-security-alerts"></a>Просмотр и исследование оповещений системы безопасности

Оповещения Azure Defender для SQL доступны на странице Оповещения центра безопасности, на вкладке Безопасность ресурса, на [панели мониторинга защитника Azure](azure-defender-dashboard.md)или с помощью прямой ссылки в сообщениях электронной почты с оповещениями.

1. Чтобы просмотреть оповещения, в меню центра безопасности выберите **оповещения системы безопасности** и выберите оповещение.

1. Оповещения предназначены для автономного выполнения с подробными инструкциями по исправлению и анализом информации в каждом из них. Дополнительные сведения см. в более широком обзоре с помощью другого центра безопасности Azure и возможностей Azure Sentinel:

    * Включите функцию аудита SQL Server для дальнейшего исследования. Если вы являетесь пользователем-Sentinel Azure, вы можете отправить журналы аудита SQL из событий журнала безопасности Windows в метку и наслаждаться богатыми возможностями расследования. Дополнительные [сведения об аудите SQL Server](/sql/relational-databases/security/auditing/create-a-server-audit-and-server-audit-specification?preserve-view=true&view=sql-server-ver15).
    * Чтобы повысить уровень безопасности, используйте рекомендации центра безопасности для главного компьютера, указанного в каждом оповещении. Это снизит риски, возникающие в будущих атаках. 

    [Дополнительные сведения об управлении оповещениями и реагировании на них](security-center-managing-and-responding-alerts.md).


## <a name="next-steps"></a>Дальнейшие действия

Связанные материалы см. в следующих статьях.

- [Оповещения системы безопасности для базы данных SQL и Azure синапсе Analytics](alerts-reference.md#alerts-sql-db-and-warehouse)
- [Настройка отправки по электронной почте уведомлений об оповещениях системы безопасности](security-center-provide-security-contact-details.md)
- [Дополнительные сведения об Azure Sentinel](../sentinel/index.yml)