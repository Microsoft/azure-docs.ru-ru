---
title: Использование журналов Azure Monitor с мультитенантным приложением
description: Настройка и использование журналов Azure Monitor с мультитенантным приложением SaaS для Базы данных SQL Azure
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/25/2019
ms.openlocfilehash: 029e3ba799e5f239bde0ef049316dd268ebe4c8f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100588776"
---
# <a name="set-up-and-use-azure-monitor-logs-with-a-multitenant-azure-sql-database-saas-app"></a>Настройка и использование журналов Azure Monitor с мультитенантным приложением SaaS для Базы данных SQL Azure
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

В этом учебнике показано, как настроить и использовать [журналы Azure Monitor](../../azure-monitor/logs/log-query-overview.md) для мониторинга эластичных пулов и баз данных. Данный документ основан на руководстве [Мониторинг производительности баз данных и пулов SQL Azure в мультитенантном приложении SaaS и управление ею](saas-dbpertenant-performance-monitoring.md). Здесь показано, как использовать журналы Azure Monitor для расширения функций мониторинга и оповещения на портале Azure. Журналы Azure Monitor позволяют выполнять мониторинг тысяч эластичных пулов и сотен тысяч баз данных. Журналы Azure Monitor также предлагают единое решение для мониторинга, с помощью которого можно отслеживать различные приложения и службы Azure в нескольких подписках Azure.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

Из этого руководства вы узнаете, как выполнить следующие задачи:

> [!div class="checklist"]
> * установка и настройка журналов Azure Monitor;
> * использование журналов Azure Monitor для мониторинга пулов и баз данных.

Для работы с этим руководством выполните следующие предварительные требования:

* Должно быть развернуто приложение SaaS Wingtip Tickets, использующее одну базу данных на клиент. Изучите инструкции из руководства по быстрому [развертыванию приложения SaaS Wingtip Tickets, использующего одну базу данных на клиент](./saas-dbpertenant-get-started-deploy.md).
* Установите Azure PowerShell. Дополнительные сведения см. в статье [Начало работы с Azure PowerShell](/powershell/azure/get-started-azureps).

Обсуждение сценариев и шаблонов SaaS и их влияния на требования к решениям для мониторинга можно найти в статье [Мониторинг производительности баз данных и пулов SQL Azure в мультитенантном приложении SaaS и управление ею](saas-dbpertenant-performance-monitoring.md).

## <a name="monitor-and-manage-database-and-elastic-pool-performance-with-azure-monitor-logs"></a>Мониторинг производительности базы данных и эластичного пула и управление ею с помощью журналов Azure Monitor

Для Базы данных SQL средства мониторинга и создания оповещений для баз данных и пулов доступны на портале Azure. Эти встроенные функции мониторинга и оповещения удобны, но зависят от ресурсов. Это означает, что они не так хорошо подходят для отслеживания крупных систем или предоставления унифицированного представления ресурсов и подписок.

В сценариях с большим количеством операций для мониторинга и создания оповещений можно использовать журналы Azure Monitor. Azure Monitor — это отдельная служба Azure, которая позволяет выполнять анализ журналов, собранных в рабочей области из множества служб. Журналы Azure Monitor предоставляют встроенный язык запросов и инструменты визуализации данных для анализа операционных данных. Решение "Аналитика SQL Azure" предоставляет несколько предопределенных представлений и запросов мониторинга и оповещений эластичных пулов и баз данных. Журналы Azure Monitor также предоставляют конструктор пользовательского представления.

Рабочие области OMS теперь называются рабочими областями Log Analytics. Решения на основе аналитики и рабочих областей Log Analytics можно запустить на портале Azure. Портал Azure является более новой точкой доступа, однако может уступать порталу Operations Management Suite в некоторых областях.

### <a name="create-performance-diagnostic-data-by-simulating-a-workload-on-your-tenants"></a>Создание диагностических данных о производительности путем имитации рабочей нагрузки в клиентах 

1. В интегрированной среде сценариев PowerShell откройте файл *..\\WingtipTicketsSaaS-MultiTenantDb-master\\Learning Modules\\Performance Monitoring and Management\\Demo-PerformanceMonitoringAndManagement.ps1*. Не закрывайте этот сценарий, так как при работе с данным руководством может потребоваться запустить несколько сценариев генератора нагрузки.
1. Если вы еще не сделали этого, подготовьте пакет клиентов, чтобы предоставить более содержательный контекст мониторинга. Этот процесс займет несколько минут.

   a. Чтобы выбрать сценарий **подготовки пакета клиентов**, задайте для параметра _$DemoScenario_ значение 1.

   b. Чтобы запустить сценарий или развернуть 17 дополнительных клиентов, нажмите клавишу F5.

1. Теперь запустите генератор нагрузки, чтобы выполнить имитацию нагрузки на всех клиентах.

    a. Чтобы **создать обычную нагрузку (около 30 DTU)** , задайте для параметра _$DemoScenario_ значение 2.

    b. Нажмите клавишу F5 для запуска сценария.

## <a name="get-the-wingtip-tickets-saas-database-per-tenant-application-scripts"></a>Получение сценариев для приложения SaaS Wingtip Tickets, использующего одну базу данных на клиент

Сценарии и исходный код приложения SaaS Wingtip Tickets мультитенантной базы данных размещены в репозитории GitHub [WingtipTicketsSaaS-DbPerTenant](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant). Инструкции по скачиванию и разблокированию сценариев PowerShell приложения Wingtip Tickets приведены [здесь](saas-tenancy-wingtip-app-guidance-tips.md).

## <a name="install-and-configure-log-analytics-workspace-and-the-azure-sql-analytics-solution"></a>Установка и настройка рабочей области Log Analytics и решения Аналитики SQL Azure

Azure Monitor — это отдельная служба, которую нужно настроить. Журналы Azure Monitor собирают данные журналов, телеметрию и метрики в рабочей области Log Analytics. Рабочую область Log Analytics нужно создать, как и другие ресурсы в Azure. Рабочую область не обязательно создавать в одной группе ресурсов с приложением, которое она отслеживает. Однако зачастую это самый оптимальный вариант. Использование одной группы ресурсов с приложением Wingtip Tickets гарантирует, что рабочая область удаляется вместе с приложением.

1. В интегрированной среде сценариев PowerShell откройте файл *..\\WingtipTicketsSaaS-MultiTenantDb-master\\Learning Modules\\Performance Monitoring and Management\\Log Analytics\\Demo-LogAnalytics.ps1*.
1. Нажмите клавишу F5 для запуска сценария.

Теперь вы можете открыть журналы Azure Monitor на портале Azure. Для сбора данных телеметрии и их отображения в рабочей области Log Analytics требуется несколько минут. Чем дольше выполняется сбор системных данных диагностики, тем содержательнее результат. 

## <a name="use-log-analytics-workspace-and-the-sql-analytics-solution-to-monitor-pools-and-databases"></a>Использование рабочей области Log Analytics и решения Аналитики SQL для мониторинга пулов и баз данных


Для этого упражнения откройте рабочую область Log Analytics на портале Azure для просмотра данных телеметрии, собранных для баз данных и пулов.

1. Перейдите на [портал Azure](https://portal.azure.com). Выберите **Все службы**, чтобы открыть рабочую область Log Analytics. Найдите Log Analytics.

   ![Открытие рабочей области Log Analytics](./media/saas-dbpertenant-log-analytics/log-analytics-open.png)

1. Выберите рабочую область _wtploganalytics-&lt;пользователь&gt;_.

1. Щелкните **Обзор**, чтобы открыть решение анализа журналов на портале Azure.

   ![Обзор](./media/saas-dbpertenant-log-analytics/click-overview.png)

    > [!IMPORTANT]
    > Активация решения может занять несколько минут. 

1. Щелкните элемент **Аналитика SQL Azure**, чтобы открыть его.

    ![Элемент обзора](./media/saas-dbpertenant-log-analytics/overview.png)

1. Используйте внутреннюю полосу прокрутки внизу, чтобы просмотреть страницы решений в представлении. При необходимости обновите страницу.

1. На странице сводки можно щелкнуть нужные элементы или отдельную базу данных, чтобы открыть обозреватель с детализированным данными.

    ![Панель мониторинга Log Analytics](./media/saas-dbpertenant-log-analytics/log-analytics-overview.png)

1. Измените параметр фильтра, чтобы указать диапазон времени. В рамках этого руководства выберите **Последний час**.

    ![Фильтр времени](./media/saas-dbpertenant-log-analytics/log-analytics-time-filter.png)

1. Выберите отдельную базу данных, чтобы просмотреть данные об использовании запросов и метрики для нее.

    ![Аналитика базы данных](./media/saas-dbpertenant-log-analytics/log-analytics-database.png)

1. Чтобы просмотреть метрики использования, прокрутите страницу аналитики вправо.
 
     ![Метрики базы данных](./media/saas-dbpertenant-log-analytics/log-analytics-database-metrics.png)

1. Прокрутите страницу аналитики влево и щелкните элемент сервера в списке **Сведения о ресурсе**.  

    ![Список "Сведения о ресурсе"](./media/saas-dbpertenant-log-analytics/log-analytics-resource-info.png)

    Откроется страница со списком пулов и баз данных на сервере.

    ![Сервер с пулами и базами данных](./media/saas-dbpertenant-log-analytics/log-analytics-server.png)

1. Выберите пул. На открывшейся странице пула прокрутите вправо, чтобы просмотреть метрики пула. 

    ![Метрики пула](./media/saas-dbpertenant-log-analytics/log-analytics-pool-metrics.png)


1. В рабочей области Log Analytics выберите **Портал OMS**, чтобы открыть на нем рабочую область.

    ![Рабочая область Log Analytics](./media/saas-dbpertenant-log-analytics/log-analytics-workspace-oms-portal.png)

В рабочей области Log Analytics можно подробнее исследовать данные журнала и метрик. 

В отличие от функции оповещения, определенной для каждого ресурса на портале Azure, функции мониторинга и оповещения в журналах Azure Monitor основаны на запросах к данным в рабочей области. Создавая оповещения на основе запросов, вместо того чтобы определять одно оповещение для каждой базы данных, можно определить одно общее оповещение для всех баз данных. Запросы ограничиваются только данными, доступными в рабочей области.

Сведения об использовании журналов Azure Monitor для создания запросов и настройки оповещений см. в статье [Создание, просмотр оповещений метрик и управление ими с помощью Azure Monitor](../../azure-monitor/alerts/alerts-metric.md).

За использование журналов Azure Monitor для Базы данных SQL будет взиматься плата на основе потребляемого объема данных в рабочей области. В этом руководстве вы создали бесплатную рабочую область с ограничением в 500 МБ в день. По достижении этого ограничения данные больше не будут добавляться в рабочую область.


## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как выполнять такие задачи:

> [!div class="checklist"]
> * установка и настройка журналов Azure Monitor;
> * использование журналов Azure Monitor для мониторинга пулов и баз данных.

Ознакомьтесь со статьей [Настройка и использование Log Analytics (OMS) с мультитенантным приложением SaaS для базы данных SQL Azure](saas-dbpertenant-log-analytics.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Дополнительные руководства на основе приложения SaaS Wingtip Tickets, использующего одну базу данных на клиент](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
* [Журналы Azure Monitor](../../azure-monitor/insights/azure-sql.md)