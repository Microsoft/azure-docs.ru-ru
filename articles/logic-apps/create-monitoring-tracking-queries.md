---
title: Просмотр и создание запросов для приложений логики в журналах Azure Monitor
description: Просмотр и создание запросов в журналах Azure Monitor для Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: divswa, logicappspm
ms.topic: article
ms.date: 01/30/2020
ms.openlocfilehash: 5de795811923c965a1473f8d91e3fd654aeef266
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100574544"
---
# <a name="view-and-create-queries-for-monitoring-and-tracking-in-azure-monitor-logs-for-azure-logic-apps"></a>Просмотр и создание запросов для мониторинга и отслеживания в журналах Azure Monitor для Azure Logic Apps

Можно просматривать базовые запросы, которые формируют результаты из [журналов Azure Monitor](../azure-monitor/logs/log-query-overview.md) , и создавать запросы, которые фильтруют результаты по конкретным критериям. Например, можно найти сообщения с определенным контрольным номером обмена. Запросы используют [язык запросов Kusto](/azure/data-explorer/kusto/query/), который можно изменить, если нужно просмотреть различные результаты. Дополнительные сведения см. в разделе [Azure Monitor запросы журналов](/azure/data-explorer/kusto/query/).

## <a name="prerequisites"></a>Предварительные требования

* Рабочая область Log Analytics. См. дополнительные сведения о [создании рабочей области Log Analytics](../azure-monitor/logs/quick-create-workspace.md).

* Приложение логики, которое настроено с Azure Monitor ведением журнала и отправляет эту информацию в Log Analytics рабочую область. Узнайте [, как настроить журналы Azure Monitor для приложения логики](../logic-apps/monitor-logic-apps.md).

* Если вы используете учетную запись интеграции, убедитесь, что вы настроили учетную запись с Azure Monitor ведением журнала, чтобы отправить эту информацию в Log Analytics рабочую область. Узнайте, как [настроить ведение журнала Azure Monitor для учетной записи интеграции](../logic-apps/monitor-b2b-messages-log-analytics.md).

## <a name="view-queries-behind-results"></a>Просмотр запросов за результатами

Чтобы просмотреть или изменить запрос, который формирует результаты в сводке рабочей области, выполните следующие действия.

1. На любой странице результатов в нижней части выберите **Показать все**.

   ![Просмотреть все результаты](./media/create-monitoring-tracking-queries/logic-app-see-all.png)

   Откроется страница журналы, на которой будет показан запрос, выходящий за предыдущую страницу результатов.

   ![Страница "журналы" — представление запроса](./media/create-monitoring-tracking-queries/view-query-behind-results.png)

1. На странице **журналы** можно выбрать следующие параметры.

   * Чтобы просмотреть результаты запроса в виде таблицы, в редакторе запросов выберите **Таблица**.

   * Чтобы изменить запрос, обновите строку запроса и выберите **выполнить** , чтобы просмотреть результаты в таблице.

## <a name="create-your-own-query"></a>Создание собственного запроса

Чтобы найти или отфильтровать результаты на основе конкретных свойств или значений, можно создать собственный запрос, начав с пустого запроса или используя существующий запрос. Дополнительные сведения см. [в разделе Приступая к работе с запросами журналов в Azure Monitor](../azure-monitor/logs/get-started-queries.md).

1. В [портал Azure](https://portal.azure.com)найдите и выберите свою рабочую область log Analytics.

1. В меню Рабочая область в разделе **Общие** выберите **журналы**.

1. Начните с пустого запроса или любых доступных существующих запросов.

   * Чтобы проверить, доступны ли какие либо существующие запросы, на панели инструментов запрос выберите пункт **выборка**  >  **журнала** запросов, где отображаются запросы из предыдущих запусков запросов, или выберите **Обозреватель запросов**, в котором отображаются готовые запросы.

     Например, Logic Apps B2B решение предоставляет следующие готовые запросы:

     ![Начните с предварительно подготовленных запросов решения "Logic Apps B2B"](./media/create-monitoring-tracking-queries/b2b-prebuilt-queries.png)

   * Чтобы начать с пустого запроса, в редакторе запросов начните вводить [язык запросов Kusto](/azure/data-explorer/kusto/query/) для запроса.

     ![Начать с пустого запроса](./media/create-monitoring-tracking-queries/create-query-from-blank.png)

## <a name="next-steps"></a>Дальнейшие действия

* [Схемы отслеживания AS2](../logic-apps/logic-apps-track-integration-account-as2-tracking-schemas.md)
* [Схемы отслеживания X12](../logic-apps/logic-apps-track-integration-account-x12-tracking-schema.md)
* [Настраиваемые схемы отслеживания](../logic-apps/logic-apps-track-integration-account-custom-tracking-schema.md)